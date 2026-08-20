# GitLab Production Troubleshooting

> Production-oriented troubleshooting playbook for GitLab, GitLab CI/CD, runners, APIs, repositories, artifacts, container registry, AWS, Kubernetes/EKS, ArgoCD, Prometheus, Grafana and ELK. The focus is systematic diagnosis, evidence collection, safe mitigation, root-cause analysis, recovery and senior DevOps interview scenarios.

---

## 1. Production Troubleshooting Philosophy

Production troubleshooting should follow:

```text
Detect
 ↓
Assess impact
 ↓
Collect evidence
 ↓
Form hypothesis
 ↓
Test safely
 ↓
Mitigate
 ↓
Verify recovery
 ↓
Find root cause
 ↓
Prevent recurrence
```

---

## 2. Do Not Guess First

Start with observable evidence:

```text
metrics
logs
events
recent changes
system state
```

---

## 3. Incident Priority

Classify the incident by:

```text
user impact
deployment impact
security impact
data impact
scope
```

---

## 4. Production Safety

Avoid experiments that can:

```text
delete data
restart critical services
change production state
```

unless the action is understood and authorized.

---

## 5. Recent Change Principle

Check recent:

```text
deployments
configuration changes
GitLab upgrades
runner upgrades
AWS changes
Kubernetes changes
```

---

## 6. Establish Timeline

Create:

```text
last known good
first symptom
alert time
recent change
mitigation
recovery
```

---

## 7. Scope the Incident

Determine whether the issue affects:

```text
one user
one project
one runner
one environment
one region
entire GitLab
```

---

## 8. Symptoms vs Causes

Example:

```text
Pipeline failed
```

is a symptom.

Possible causes:

```text
runner unavailable
credentials expired
build failure
registry outage
```

---

## 9. First Five Questions

Ask:

```text
What is broken?
When did it start?
Who is affected?
What changed?
Is the problem getting worse?
```

---

## 10. GitLab Is Unavailable

Check:

```text
DNS
load balancer
reverse proxy
GitLab health
application processes
database
cache
host resources
```

---

## 11. GitLab HTTP 5xx

Check:

```text
request logs
application logs
backend health
database
cache
resource saturation
```

---

## 12. GitLab HTTP 4xx

Determine whether the issue is:

```text
authentication
authorization
invalid request
missing resource
```

---

## 13. GitLab 401

Common causes:

```text
expired token
invalid token
wrong authentication header
credential rotation
```

---

## 14. GitLab 403

Common causes:

```text
insufficient permission
protected resource
protected branch
environment restriction
```

---

## 15. GitLab 404

Possible causes:

```text
wrong project ID
wrong namespace
resource removed
insufficient visibility
incorrect URL
```

---

## 16. GitLab API Slow

Check:

```text
API latency
database latency
application load
cache
network
request volume
```

---

## 17. GitLab API 429

Investigate:

```text
automation volume
polling frequency
parallel workers
rate-limit configuration
```

---

## 18. API Retry Storm

A badly designed client can amplify an outage:

```text
API fails
 ↓
client retries
 ↓
more traffic
 ↓
API becomes slower
 ↓
more retries
```

---

## 19. Stop Retry Storm

Use:

```text
exponential backoff
jitter
bounded retries
circuit breaker
```

---

## 20. GitLab UI Slow

Compare:

```text
UI
API
database
host
network
```

If API is also slow, the UI may simply be exposing backend latency.

---

## 21. GitLab Git Clone Slow

Check:

```text
repository size
network
GitLab storage
CPU
disk I/O
large files
```

---

## 22. Git Clone Fails

Collect:

```text
exact command
error
repository
branch
user
timestamp
```

---

## 23. Git Push Fails

Check:

```text
authentication
authorization
branch protection
repository storage
pre-receive hooks
network
```

---

## 24. Push Rejected by Protected Branch

Verify:

```text
branch protection
user role
MR policy
allowed push rules
```

Do not bypass protection merely to make a deployment work.

---

## 25. Repository Corruption Concern

Treat repository integrity issues as high severity.

Avoid destructive Git operations until backups and repository state are understood.

---

## 26. Large Repository

Symptoms:

```text
clone slow
push slow
CI checkout slow
storage growth
```

Review:

```text
Git history
large binaries
LFS
artifacts
```

---

## 27. Git LFS Problem

Check:

```text
LFS configuration
object availability
credentials
storage
```

---

## 28. Merge Request Cannot Be Created

Check:

```text
source branch
target branch
permissions
existing MR
project state
API response
```

---

## 29. MR Pipeline Missing

Check:

```text
workflow rules
pipeline source
branch rules
.gitlab-ci.yml
project settings
```

---

## 30. Duplicate Pipelines

Check:

```text
workflow rules
push pipeline
MR pipeline
trigger pipeline
```

---

## 31. Pipeline Not Created

Possible causes:

```text
workflow rules
invalid CI configuration
commit not matching rules
pipeline source excluded
```

---

## 32. CI YAML Syntax Failure

Validate the configuration before changing production automation.

---

## 33. Job Not Included

Check job-level:

```text
rules
only/except where applicable
needs
stage
workflow
```

---

## 34. Job Stuck Pending

This is commonly a runner problem.

Check:

```text
runner availability
tags
protected runner
runner scope
concurrency
```

---

## 35. Job Stuck Because of Tags

Example:

```text
job tag = production
```

but no available runner has that tag.

---

## 36. Protected Runner Problem

A protected runner may only execute jobs from protected refs according to configuration.

---

## 37. Runner Offline

Check:

```text
runner process
network
registration
host
Kubernetes
```

---

## 38. Runner Online but No Jobs

Check:

```text
tags
paused state
protected setting
concurrency
job requirements
```

---

## 39. Runner Authentication Failure

Verify:

```text
runner authentication
registration configuration
network
GitLab endpoint
```

Use current supported runner registration/authentication mechanisms.

---

## 40. Runner Version Mismatch

Check:

```text
GitLab version
runner version
executor
job image
```

---

## 41. Runner CPU Saturation

Symptoms:

```text
jobs slow
queue grows
job startup delayed
```

Scale capacity or optimize workload.

---

## 42. Runner Memory Pressure

Check:

```text
host memory
container memory
job workload
parallel jobs
```

---

## 43. Kubernetes Runner Pending

If using Kubernetes executor:

```text
kubectl get pods
kubectl describe pod
kubectl get events
```

---

## 44. Runner Pod Cannot Schedule

Check:

```text
node capacity
taints
tolerations
resource requests
affinity
quotas
```

---

## 45. Runner Pod OOMKilled

Check:

```text
memory request
memory limit
actual usage
build workload
```

---

## 46. Runner Pod ImagePullBackOff

Check:

```text
image name
registry
credentials
network
image tag
```

---

## 47. Docker Executor Failure

Check:

```text
Docker daemon
socket
permissions
disk
image
```

---

## 48. Docker-in-Docker Failure

Check:

```text
DinD service
TLS configuration
network
privileged mode requirements
resource limits
```

Use the least-privileged architecture practical.

---

## 49. Shell Executor Failure

Check:

```text
host packages
permissions
PATH
disk
shell
user
```

---

## 50. Job Image Failure

Check:

```text
image
tag
registry
architecture
entrypoint
```

---

## 51. Job Starts Then Immediately Fails

Check:

```text
entrypoint
command
environment variables
working directory
permissions
```

---

## 52. Job Timeout

Separate:

```text
queue timeout
script timeout
external command timeout
deployment timeout
```

---

## 53. Long Build

Measure:

```text
checkout
dependency download
compile
tests
image build
push
```

---

## 54. Slow Dependency Download

Check:

```text
cache
registry
package repository
network
DNS
```

---

## 55. Cache Not Restored

Check:

```text
cache key
runner configuration
cache backend
permissions
branch rules
```

---

## 56. Cache Corruption

Symptoms:

```text
random build failures
unexpected dependency versions
```

Clear or rotate cache keys safely.

---

## 57. Artifact Missing

Check:

```text
artifact path
job success
artifacts configuration
retention
dependencies
needs
```

---

## 58. Artifact Expired

Check:

```text
retention policy
pipeline age
job configuration
```

---

## 59. Artifact Download Failure

Check:

```text
permissions
storage
network
artifact existence
```

---

## 60. Pipeline Dependency Failure

If a job cannot access another job's artifact:

```text
check needs
check dependencies
check artifact configuration
```

---

## 61. DAG Failure

A DAG may fail because a required dependency is:

```text
missing
failed
not included
```

---

## 62. `needs` Troubleshooting

Review the actual dependency graph rather than assuming stage ordering.

---

## 63. Pipeline Runs Sequentially

Check whether unnecessary:

```text
needs
dependencies
stage boundaries
```

are forcing serialization.

---

## 64. Pipeline Suddenly Slower

Compare:

```text
before
after
```

for:

```text
job duration
queue time
runner capacity
dependency downloads
```

---

## 65. Security Scan Failure

Determine whether failure is:

```text
vulnerability
scanner
network
configuration
policy
```

---

## 66. Trivy Failure

Check:

```text
scanner database
network
image
credentials
resource limits
```

---

## 67. SonarQube Failure

Check:

```text
server availability
token
project configuration
scanner version
quality gate
```

---

## 68. Veracode Failure

Check:

```text
credentials
upload
scan status
policy
API availability
```

---

## 69. False Security Failure

Do not bypass the gate immediately.

Confirm:

```text
finding
severity
policy
false positive
exception process
```

---

## 70. Security Scanner Timeout

Check:

```text
application size
network
scanner capacity
server health
job timeout
```

---

## 71. Docker Build Failure

Collect:

```text
Dockerfile
base image
build context
error line
dependency
```

---

## 72. Docker Build Context Too Large

Check:

```text
.dockerignore
repository
artifacts
node_modules
build outputs
```

---

## 73. Docker Build Cache Missing

Check:

```text
cache source
cache permissions
builder configuration
Dockerfile changes
```

---

## 74. Image Push Failure

Check:

```text
registry authentication
repository
network
image size
permissions
```

---

## 75. ECR Push Failure

Check:

```text
AWS identity
STS
ECR permissions
region
repository
Docker login
```

---

## 76. ECR Login Failure

Verify the job's AWS identity before troubleshooting Docker.

---

## 77. AWS OIDC Failure

Check:

```text
OIDC token
trust policy
audience
subject claims
role ARN
region
```

---

## 78. AWS AccessDenied

Identify:

```text
caller identity
requested action
resource
IAM policy
trust relationship
```

---

## 79. AWS Credentials Expired

Prefer short-lived OIDC credentials and verify their validity during the job.

---

## 80. Terraform Plan Failure

Check:

```text
credentials
backend
provider
variables
state
resource configuration
```

---

## 81. Terraform State Lock

Check:

```text
active Terraform operation
state backend
lock status
```

Do not force-unlock without verifying that no valid operation is running.

---

## 82. Terraform Backend Failure

Check:

```text
S3
permissions
bucket
region
network
state key
```

---

## 83. Terraform Apply Failed

Separate:

```text
Terraform configuration
AWS API
permissions
quota
resource conflict
```

---

## 84. Terraform Partial Apply

Terraform may have created some resources before failing.

Run:

```text
inspect state
inspect cloud resources
plan again
```

before retrying blindly.

---

## 85. Kubernetes Deployment Failed

Check:

```bash
kubectl get deployment -n <namespace>
kubectl rollout status deployment/<name> -n <namespace>
```

---

## 86. Pod Not Ready

Check:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
```

---

## 87. CrashLoopBackOff

Investigate:

```text
current logs
previous logs
events
exit code
environment
probes
resources
```

---

## 88. OOMKilled

Check:

```text
memory usage
limit
request
application behavior
traffic
```

---

## 89. ImagePullBackOff

Check:

```text
image
tag
registry
secret
IAM
network
```

---

## 90. Pending Pod

Check:

```text
resources
taints
tolerations
affinity
PVC
quotas
```

---

## 91. Readiness Probe Failure

A readiness failure means the Pod should generally stop receiving traffic.

Check:

```text
endpoint
port
startup time
dependency
configuration
```

---

## 92. Liveness Probe Failure

A liveness failure can cause restarts.

Do not use overly aggressive liveness probes.

---

## 93. Startup Probe

For slow-starting applications, a startup probe can prevent premature liveness failures.

---

## 94. Service Has No Endpoints

Check:

```text
selector
Pod labels
Pod readiness
namespace
```

---

## 95. ALB Ingress Failure

Check:

```text
Ingress
target health
security groups
subnets
listener
service
Pod readiness
```

---

## 96. ALB 5xx

Separate:

```text
ALB-generated error
application-generated error
```

then inspect target health and application logs.

---

## 97. DNS Failure

Check:

```bash
dig <hostname>
nslookup <hostname>
```

from an appropriate network location.

---

## 98. TLS Failure

Check:

```text
certificate
hostname
chain
expiry
listener
```

---

## 99. Kubernetes Node NotReady

Check:

```bash
kubectl describe node <node>
```

Then inspect:

```text
conditions
events
kubelet
resources
network
disk
```

---

## 100. Node DiskPressure

Check:

```text
disk usage
container images
logs
ephemeral storage
```

---

## 101. Node MemoryPressure

Check:

```text
memory usage
evictions
Pod requests
system processes
```

---

## 102. Node Network Problem

Check:

```text
CNI
routes
security groups
DNS
kube-proxy
```

---

## 103. EKS API Access Problem

Check:

```text
AWS identity
cluster endpoint
network
IAM
authentication
```

---

## 104. EKS Authentication Failure

Verify:

```text
caller identity
cluster access configuration
IAM permissions
kubeconfig
```

---

## 105. ArgoCD Application OutOfSync

Check:

```text
Git revision
desired manifest
live state
sync status
```

---

## 106. ArgoCD Sync Failed

Check:

```text
manifest rendering
RBAC
Kubernetes API
resource validation
hooks
```

---

## 107. ArgoCD Degraded

Check:

```text
application health
Pod health
service
ingress
dependency
```

---

## 108. ArgoCD Cannot Access Git

Check:

```text
repository credentials
network
repository URL
revision
```

---

## 109. GitOps Drift

If the cluster differs from Git:

```text
check manual changes
check ArgoCD sync
check ignore rules
```

---

## 110. Manual Kubernetes Change

Do not assume the manual change is the correct permanent fix.

Update Git desired state if the change should remain.

---

## 111. ArgoCD Self-Heal

If enabled, ArgoCD may revert manual changes.

Understand this before making emergency changes.

---

## 112. GitOps Emergency Change

If an emergency direct change is required:

```text
mitigate
document
update Git
reconcile
```

---

## 113. Application 5xx

Start with:

```text
error rate
deployment
Pod health
application logs
dependencies
```

---

## 114. Application High Latency

Check:

```text
CPU
memory
database
network
external APIs
recent deployment
```

---

## 115. Application Memory Leak

Indicators:

```text
memory steadily rises
restarts
GC pressure
OOMKilled
```

---

## 116. Application CPU Spike

Correlate:

```text
traffic
deployment
query behavior
loop
resource limits
```

---

## 117. High Traffic Incident

Check:

```text
load balancer
HPA
Pods
nodes
database
queue
```

---

## 118. HPA Not Scaling

Check:

```text
metrics
requests
target
HPA status
resource availability
```

---

## 119. HPA Scales Too Slowly

Investigate:

```text
metric collection
cooldown behavior
Pod startup
node capacity
```

---

## 120. HPA Scaling Loop

Repeated scale up/down can indicate:

```text
unstable workload
bad target
startup behavior
insufficient stabilization
```

---

## 121. Database Dependency Failure

Application symptoms may include:

```text
timeouts
5xx
connection errors
latency
```

---

## 122. Database Connection Exhaustion

Check:

```text
connection count
pool configuration
long-running queries
application replicas
```

---

## 123. Redis Dependency Failure

Check:

```text
availability
latency
connections
memory
evictions
```

---

## 124. RabbitMQ Dependency Failure

Check:

```text
queue depth
consumer count
message rate
connections
memory
```

---

## 125. Asynchronous Queue Backlog

A growing queue can indicate:

```text
consumer failure
traffic spike
downstream dependency
```

---

## 126. API Automation Failure

Check:

```text
token
endpoint
status code
rate limit
payload
```

---

## 127. Webhook Not Delivered

Check:

```text
webhook configuration
receiver health
secret
network
event
```

---

## 128. Webhook Delivered but Not Processed

Check:

```text
receiver logs
queue
worker
deduplication
payload validation
```

---

## 129. GitLab API Rate Limit Incident

Reduce:

```text
polling
parallel requests
duplicate API calls
```

and use backoff.

---

## 130. Pipeline Trigger Not Working

Check:

```text
token
project ID
ref
variables
permissions
rules
```

---

## 131. Pipeline Triggered Twice

Check:

```text
webhook
push
MR
schedule
trigger
workflow rules
```

---

## 132. Scheduled Pipeline Missing

Check:

```text
schedule enabled
owner
ref
permissions
workflow rules
```

---

## 133. Manual Job Missing

Check:

```text
rules
environment
protected branch
pipeline source
```

---

## 134. Production Deployment Job Missing

Check:

```text
protected environment
rules
branch/tag
variables
pipeline source
```

---

## 135. Production Deployment Blocked

Determine whether it is intentionally blocked by:

```text
approval
environment protection
security gate
branch protection
```

Do not bypass controls without authorization.

---

## 136. Production Deployment Failed

Collect:

```text
pipeline
job
commit
image digest
environment
logs
ArgoCD state
Kubernetes state
```

---

## 137. Deployment Failed Before Kubernetes

If the CI job failed before GitOps update:

```text
investigate GitLab job
```

---

## 138. Deployment Failed After GitOps Update

Check:

```text
GitOps commit
ArgoCD
Kubernetes
```

---

## 139. Wrong Image Deployed

Verify:

```text
Git commit
manifest
image tag
image digest
ECR
ArgoCD revision
```

---

## 140. Mutable Tag Problem

If the same tag points to different images, trace deployment by digest.

Prefer immutable artifact references.

---

## 141. Rollback

Preferred GitOps rollback:

```text
revert desired state
 ↓
ArgoCD
 ↓
Kubernetes
```

---

## 142. Rollback Verification

Check:

```text
Pod version
image digest
health
errors
latency
```

---

## 143. Rollback Failed

Possible causes:

```text
previous image unavailable
database migration incompatible
configuration mismatch
dependency changed
```

---

## 144. Database Migration Incident

Do not blindly roll back application code if schema changes are incompatible.

Assess:

```text
schema state
migration history
application compatibility
```

---

## 145. Forward-Compatible Migration

Prefer migration strategies that allow old and new application versions to coexist during rolling deployments.

---

## 146. Secret Rotation Incident

If a rotated secret breaks production:

```text
identify changed secret
restore valid credential if safe
update consumers
verify
```

---

## 147. ConfigMap Change Incident

Compare:

```text
Git
live ConfigMap
deployment revision
```

---

## 148. Secret Not Available to Pod

Check:

```text
Secret existence
namespace
key
RBAC
volume/env reference
```

---

## 149. Environment Variable Missing

Check:

```text
Deployment
ConfigMap
Secret
Helm values
GitOps rendered manifest
```

---

## 150. Helm Rendering Failure

Run rendering in a controlled environment and inspect:

```text
values
templates
chart dependencies
Kubernetes API versions
```

---

## 151. Helm Deployment Failure

Separate:

```text
rendering
validation
Kubernetes apply
runtime health
```

---

## 152. Terraform and Kubernetes Conflict

Determine ownership.

Avoid having:

```text
Terraform
+
ArgoCD
```

manage the same Kubernetes resource unintentionally.

---

## 153. Ownership Conflict

One system should be the authoritative owner for a resource.

---

## 154. Infrastructure Drift

For Terraform:

```text
terraform plan
```

For GitOps:

```text
ArgoCD sync/diff
```

---

## 155. AWS Resource Drift

Check:

```text
Terraform state
AWS actual state
Git desired state
```

---

## 156. Security Group Issue

If application connectivity breaks:

```text
source
destination
port
protocol
security group
NACL
route
```

---

## 157. NAT Gateway Problem

Private subnet workloads may fail to access external services.

Check:

```text
route table
NAT gateway
subnet
network ACL
```

---

## 158. ECR Connectivity Problem

Check:

```text
DNS
VPC endpoints/NAT
security groups
IAM
region
```

---

## 159. S3 Access Failure

Check:

```text
IAM
bucket policy
region
endpoint/NAT
encryption
```

---

## 160. RDS Connectivity Failure

Check:

```text
security groups
subnet
route
DNS
credentials
database status
```

---

## 161. Route53 Failure

Check:

```text
record
hosted zone
health check
TTL
DNS resolution
```

---

## 162. ALB Target Unhealthy

Check:

```text
health path
port
security groups
Pod readiness
service
```

---

## 163. ALB 502

Often investigate:

```text
target connection
application port
target health
```

---

## 164. ALB 503

Investigate:

```text
no healthy targets
listener
target group
```

---

## 165. ALB 504

Investigate:

```text
backend timeout
application latency
network
```

---

## 166. Incident During High Load

Prioritize:

```text
availability
capacity
critical paths
```

Avoid unrelated maintenance.

---

## 167. Load Shedding

If necessary, protect critical services by reducing non-critical work.

---

## 168. Autoscaling Failure

Check:

```text
metrics
IAM
capacity
quotas
scheduler
```

---

## 169. AWS Quota Problem

Check service quotas when resources fail unexpectedly.

---

## 170. Kubernetes Quota Problem

Check:

```bash
kubectl get resourcequota -n <namespace>
```

---

## 171. Pod Security Policy Failure

Modern Kubernetes security may involve:

```text
Pod Security Standards
RBAC
admission controllers
```

Check the exact cluster policy.

---

## 172. RBAC Failure

Identify:

```text
user/service account
verb
resource
namespace
role
role binding
```

---

## 173. ServiceAccount Token Problem

Check:

```text
ServiceAccount
RBAC
token behavior
workload identity
```

---

## 174. EKS IAM Role for Service Account

Check:

```text
IAM role
trust policy
service account annotation/configuration
OIDC provider
```

---

## 175. Node IAM Problem

Separate:

```text
node role
Pod workload identity
```

Do not give node roles unnecessary application permissions.

---

## 176. Kubernetes DNS Failure

Check:

```text
CoreDNS
Service
NetworkPolicy
node network
```

---

## 177. CoreDNS Failure

Symptoms:

```text
service names fail
external DNS fails
applications timeout
```

---

## 178. NetworkPolicy Problem

Check whether traffic is allowed between:

```text
source
destination
port
namespace
```

---

## 179. Pod-to-Pod Connectivity

Test from an appropriate debugging Pod.

---

## 180. Service-to-Service Failure

Check:

```text
DNS
Service
Endpoints
NetworkPolicy
application port
```

---

## 181. Ingress-to-Service Failure

Check:

```text
Ingress
target port
Service port
Endpoints
```

---

## 182. EKS Node Autoscaling Failure

Check:

```text
cluster autoscaler/Karpenter configuration
subnets
IAM
capacity
quotas
```

Use the scaling mechanism actually deployed.

---

## 183. Pending Pods During Scale-Out

Investigate:

```text
node provisioning
instance capacity
subnet IPs
taints
resource requests
```

---

## 184. IP Exhaustion

EKS networking can run out of available IPs.

Monitor:

```text
subnet free IPs
ENIs
Pod IP usage
```

---

## 185. Container Runtime Problem

Check:

```text
runtime
disk
images
container logs
node health
```

---

## 186. Container Image Disk Growth

Clean old images carefully and use appropriate node image/runtime policies.

---

## 187. Kubernetes Log Growth

Large container logs can consume node disk.

Use centralized logging and retention controls.

---

## 188. ELK Log Pipeline Failure

Trace:

```text
application
 ↓
collector
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

---

## 189. Elasticsearch Red Cluster

Treat red cluster health as urgent.

Check:

```text
unassigned shards
disk
nodes
allocation
```

---

## 190. Elasticsearch Yellow Cluster

Yellow often indicates replica allocation problems while primary shards remain available.

Investigate capacity and allocation.

---

## 191. Elasticsearch Unassigned Shards

Check:

```text
disk
node availability
allocation rules
replica requirements
```

---

## 192. Logstash Backpressure

Check:

```text
queue
CPU
memory
Elasticsearch
network
```

---

## 193. Prometheus Target Down

Check:

```text
target
network
endpoint
service discovery
TLS/auth
```

---

## 194. Prometheus Storage Full

Check:

```text
retention
disk
high cardinality
remote storage
```

---

## 195. Prometheus High Memory

Investigate:

```text
cardinality
scrape interval
label design
queries
retention
```

---

## 196. Grafana Dashboard Empty

Check:

```text
data source
time range
query
labels
Prometheus
```

---

## 197. Grafana Alert Missing

Check:

```text
alert rule
evaluation
data source
routing
notification
```

---

## 198. Monitoring Stack Failure

If monitoring is unavailable:

```text
use direct system checks
use logs
use cloud metrics
```

until observability is restored.

---

## 199. Break-Glass Troubleshooting

Emergency access should be:

```text
temporary
audited
minimal
```

---

## 200. Break-Glass Credential

Do not use emergency credentials as normal daily credentials.

---

## 201. Incident Command

For major incidents establish:

```text
incident commander
technical lead
communications
scribe
```

where organizational process supports these roles.

---

## 202. Communication

Communicate:

```text
impact
current status
mitigation
next update
```

without exposing sensitive information.

---

## 203. Evidence Collection

Preserve:

```text
logs
metrics
events
pipeline IDs
commit SHA
deployment revision
timestamps
```

---

## 204. Do Not Destroy Evidence

Avoid clearing:

```text
logs
Pods
resources
```

before collecting enough evidence unless required for immediate mitigation.

---

## 205. Incident Commands

Useful commands include:

```bash
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

---

## 206. Node Commands

On Linux hosts:

```bash
uptime
free -m
df -h
df -i
top
ps -ef
ss -lntp
```

Use appropriate permissions.

---

## 207. Network Commands

Useful tools:

```bash
curl
dig
nslookup
ss
traceroute
```

---

## 208. AWS Identity Check

Before debugging AWS permissions:

```bash
aws sts get-caller-identity
```

This identifies the active AWS identity.

---

## 209. Kubernetes Context Check

Before running cluster commands:

```bash
kubectl config current-context
```

---

## 210. Namespace Check

Avoid accidental commands against the wrong namespace.

```bash
kubectl config set-context --current --namespace=<namespace>
```

Use carefully.

---

## 211. Git Context Check

Before changing repositories:

```bash
git branch --show-current
git status
git remote -v
```

---

## 212. ArgoCD Context

Verify:

```text
application
project
revision
cluster
namespace
```

before sync or rollback actions.

---

## 213. Production Command Safety

Prefer read-only commands first:

```text
get
describe
status
logs
diff
plan
```

before:

```text
apply
delete
restart
destroy
```

---

## 214. Troubleshooting Decision Tree

```text
Is service unavailable?
        │
       Yes
        │
        ▼
Check recent change
        │
        ▼
Check metrics
        │
        ▼
Check logs
        │
        ▼
Check infrastructure
        │
        ▼
Check dependencies
        │
        ▼
Mitigate
        │
        ▼
Verify
```

---

## 215. Pipeline Troubleshooting Tree

```text
Pipeline issue
 │
 ├── Not created?
 │      └── workflow/rules/config
 │
 ├── Pending?
 │      └── runner/tags/capacity
 │
 ├── Job failed?
 │      └── logs/script/dependency
 │
 ├── Artifact missing?
 │      └── artifacts/needs/retention
 │
 └── Deployment failed?
        └── GitOps/ArgoCD/Kubernetes
```

---

## 216. Deployment Troubleshooting Tree

```text
Deployment
 │
 ├── CI failed
 │    └── GitLab job
 │
 ├── GitOps not updated
 │    └── repository/MR
 │
 ├── ArgoCD failed
 │    └── sync/manifest/RBAC
 │
 └── Kubernetes unhealthy
      └── Pods/service/ingress/resources
```

---

## 217. Runner Troubleshooting Tree

```text
Job pending
 │
 ├── runner online?
 │      └── no → runner host/service
 │
 ├── tags match?
 │      └── no → correct tags
 │
 ├── protected?
 │      └── check ref/environment
 │
 └── capacity?
        └── scale runners
```

---

## 218. API Troubleshooting Tree

```text
API failure
 │
 ├── 4xx → request/auth/permission
 │
 ├── 429 → rate limit
 │
 ├── 5xx → GitLab/backend
 │
 └── timeout → network/load/dependency
```

---

## 219. AWS Troubleshooting Tree

```text
AWS failure
 │
 ├── identity?
 ├── permission?
 ├── region?
 ├── resource exists?
 ├── quota?
 └── network?
```

---

## 220. Kubernetes Troubleshooting Tree

```text
Workload failure
 │
 ├── Pod pending?
 ├── Pod crash?
 ├── Pod not ready?
 ├── service no endpoints?
 ├── ingress unhealthy?
 └── node issue?
```

---

## 221. ArgoCD Troubleshooting Tree

```text
ArgoCD issue
 │
 ├── Git access?
 ├── revision?
 ├── render?
 ├── sync?
 ├── RBAC?
 └── application health?
```

---

## 222. Troubleshooting by Layer

```text
Layer 1 → DNS/network
Layer 2 → load balancer
Layer 3 → GitLab/API
Layer 4 → runner/CI
Layer 5 → GitOps
Layer 6 → Kubernetes
Layer 7 → application
Layer 8 → dependency
```

---

## 223. Root Cause Analysis

After mitigation ask:

```text
Why did it happen?
Why was it not detected earlier?
Why did safeguards fail?
```

---

## 224. Five Whys

Example:

```text
Deployment failed
 ↓
Pod unhealthy
 ↓
wrong configuration
 ↓
GitOps validation missed it
 ↓
test environment lacked production configuration
```

---

## 225. Contributing Factors

Root cause may not be a single error.

Consider:

```text
process
technology
configuration
capacity
human factors
```

---

## 226. Corrective Action

Fix the direct technical problem.

---

## 227. Preventive Action

Improve:

```text
automation
testing
monitoring
security
documentation
```

---

## 228. Post-Incident Review

Review:

```text
detection
response
mitigation
communication
recovery
prevention
```

---

## 229. Blameless Review

Focus on:

```text
system behavior
process gaps
technical controls
```

rather than individual blame.

---

## 230. Runbook Improvement

If troubleshooting required undocumented steps:

```text
update runbook
```

---

## 231. Alert Improvement

If detection was late:

```text
add/tune signal
```

---

## 232. Test Improvement

If deployment escaped validation:

```text
add test
```

---

## 233. Architecture Improvement

If recurring incidents indicate structural weakness:

```text
redesign
```

rather than repeatedly applying manual fixes.

---

## 234. Production Troubleshooting Checklist

```text
[ ] Identify impact
[ ] Establish timeline
[ ] Check recent changes
[ ] Check metrics
[ ] Check logs
[ ] Check events
[ ] Check dependencies
[ ] Check runners
[ ] Check GitLab API
[ ] Check AWS
[ ] Check Kubernetes
[ ] Check ArgoCD
[ ] Mitigate safely
[ ] Verify recovery
[ ] Preserve evidence
[ ] Root cause
[ ] Corrective action
[ ] Preventive action
[ ] Update runbook
```

---

## 235. Senior Interview — How Do You Troubleshoot a Production Pipeline Failure?

> I first identify whether the pipeline was not created, is pending, or actually failed. If pending, I investigate runners, tags and capacity. If failed, I inspect the job logs and dependency chain. If deployment is involved, I continue into GitOps, ArgoCD and Kubernetes rather than treating the CI failure in isolation.

---

## 236. Senior Interview — How Do You Troubleshoot CrashLoopBackOff?

> I check Pod events, current logs and `--previous` logs, then inspect exit codes, resource limits, environment variables, ConfigMaps, Secrets and probes. I determine whether it is an application failure, configuration issue, dependency problem or OOMKilled condition.

---

## 237. Senior Interview — How Do You Troubleshoot ImagePullBackOff?

> I verify the exact image and tag, registry availability, credentials, IAM permissions, network connectivity and whether the image exists. In EKS I also verify the workload identity or node permissions used for registry access.

---

## 238. Senior Interview — How Do You Troubleshoot a Pending Pod?

> I inspect `describe pod` and events, then check resource requests, node capacity, taints/tolerations, affinity, quotas, PVCs and scheduling constraints.

---

## 239. Senior Interview — How Do You Troubleshoot ArgoCD OutOfSync?

> I compare Git desired state with live state, inspect the application diff and sync status, check whether a manual change caused drift, and verify whether ignore rules or resource ownership are affecting reconciliation.

---

## 240. Senior Interview — How Do You Handle a Production Deployment Failure?

> I stop further promotion, determine whether the failure is in CI, GitOps or Kubernetes, identify the exact commit and image digest, assess customer impact, mitigate with rollback or fix-forward when appropriate, verify recovery and then perform root-cause analysis.

---

## 241. Senior Interview — How Do You Troubleshoot a Slow GitLab Pipeline?

> I separate queue time from execution time. High queue time points toward runner capacity or matching problems. High execution time requires inspecting dependency downloads, caching, tests, Docker builds and the pipeline dependency graph.

---

## 242. Senior Interview — How Do You Troubleshoot AWS AccessDenied?

> I first identify the caller with `aws sts get-caller-identity`, then identify the denied action and resource. I inspect IAM policies, trust relationships, resource policies and the identity actually used by the GitLab job.

---

## 243. Senior Interview — How Do You Troubleshoot ALB 503?

> I check whether the target group has healthy targets, then inspect Kubernetes Service endpoints, Pod readiness, target ports, security groups and the application listener.

---

## 244. Senior Interview — How Do You Troubleshoot Kubernetes 5xx?

> I identify whether the error is generated by ALB/Ingress or the application. Then I correlate request metrics, deployment timestamps, Pod health, application logs, dependencies and resource saturation.

---

## 245. Senior Interview — How Do You Troubleshoot Prometheus Missing Metrics?

> I check the target's health, service discovery, scrape endpoint, authentication/TLS, network connectivity and the PromQL query. I also verify whether the metric exists under the expected name and labels.

---

## 246. Senior Interview — How Do You Troubleshoot ELK Missing Logs?

> I trace the entire path from application output to collector, Logstash, Elasticsearch and Kibana. I check ingestion errors, queues, index availability, time range and whether the application generated the expected logs.

---

## 247. Senior Interview — How Do You Decide Between Rollback and Fix Forward?

> I consider customer impact, rollback safety, previous-version availability, database compatibility and root-cause confidence. If the previous version is known-good and rollback is safe, rollback is often fastest. If rollback is unsafe or the fix is small and well understood, fix-forward may be preferable.

---

## 248. Senior Interview — What Is Your Production Troubleshooting Method?

> My method is evidence-driven: establish impact and timeline, identify recent changes, inspect metrics and logs, isolate the failing layer, test the smallest safe hypothesis, mitigate, verify recovery and then perform root-cause analysis with corrective and preventive actions.

---

## 249. Final Mental Model

```text
                    PRODUCTION INCIDENT

                         ALERT
                           │
                           ▼
                    ASSESS IMPACT
                           │
                           ▼
                    BUILD TIMELINE
                           │
                           ▼
                    CHECK CHANGES
                           │
                           ▼
                    METRICS + LOGS
                           │
                           ▼
                  ISOLATE FAILURE LAYER
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
      GitLab             CI/CD             AWS/EKS
        │                  │                  │
        ▼                  ▼                  ▼
       API              Runner            Kubernetes
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                         ArgoCD
                           │
                           ▼
                      APPLICATION
                           │
                           ▼
                      MITIGATE
                           │
                           ▼
                       VERIFY
                           │
                           ▼
                      ROOT CAUSE
                           │
                           ▼
                    PREVENT RECURRENCE
```

> **Core principle:** Production troubleshooting is not about knowing the most commands. It is about reducing uncertainty systematically. Start with impact and evidence, isolate the failing layer, make the smallest safe change, verify recovery, and convert every important incident into stronger automation, monitoring, testing or architecture.

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md ✓
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `22-GitLab-Production-Architecture.md`**
