# 13-GitLab — 06 GitLab Runners

> Production-oriented notes on GitLab Runner architecture, installation, registration, executors, Docker and Kubernetes runners, runner tags, protected runners, autoscaling, security, AWS/EKS integration, troubleshooting, performance, observability, and senior DevOps interview scenarios.

---

## 1. What Is GitLab Runner?

GitLab Runner is the execution agent that runs GitLab CI/CD jobs.

Architecture:

```text
GitLab
   ↓
Pipeline
   ↓
Job
   ↓
GitLab Runner
   ↓
Commands execute
```

GitLab coordinates the pipeline.

The Runner performs the actual work.

---

## 2. Why Runners Are Required

A GitLab pipeline defines:

```text
What should happen
```

The Runner provides:

```text
Where and how it happens
```

Example:

```yaml
test:
  script:
    - pytest
```

GitLab schedules the job.

A Runner executes:

```bash
pytest
```

---

## 3. Runner Architecture

Simplified:

```text
Developer
    ↓
GitLab Repository
    ↓
.gitlab-ci.yml
    ↓
Pipeline
    ↓
Job Queue
    ↓
Runner
    ↓
Executor
    ↓
Environment
    ↓
Commands
```

The executor determines how the job environment is created.

---

## 4. Runner Lifecycle

Typical lifecycle:

```text
Install Runner
      ↓
Register/Authenticate
      ↓
Configure Executor
      ↓
Runner becomes available
      ↓
Receives eligible jobs
      ↓
Executes jobs
      ↓
Reports status
```

---

## 5. Runner Scope

Runners can be associated at different organizational scopes depending on GitLab configuration/licensing:

- instance-level
- group-level
- project-level

Use the narrowest scope that satisfies the requirement.

---

## 6. Project Runner

A project-specific Runner is useful when:

```text
Project A
   ↓
Dedicated execution environment
```

Advantages:

- isolation
- predictable dependencies
- project-specific permissions

Disadvantage:

- potentially lower utilization

---

## 7. Group Runner

A group Runner can serve multiple projects:

```text
Group
 ├── Project A
 ├── Project B
 └── Project C
        ↓
     Runner Pool
```

Useful for shared engineering workloads.

Security boundaries must be considered carefully.

---

## 8. Instance-Level Runner

An instance-wide Runner can potentially serve many projects.

This can maximize utilization.

But:

```text
More projects
   ↓
Larger blast radius
```

Use strict isolation and least privilege.

---

## 9. Runner Registration

A Runner must be associated with GitLab so it can receive jobs.

Modern GitLab setups use Runner authentication mechanisms rather than relying on old registration-token workflows.

General process:

```text
Create Runner in GitLab
      ↓
Obtain authentication information
      ↓
Install Runner
      ↓
Authenticate/configure
      ↓
Verify
```

Follow the GitLab version's supported registration method.

---

## 10. Runner Authentication

The Runner needs credentials that allow it to communicate with GitLab.

Treat Runner authentication material as sensitive.

Do not:

- commit it to Git
- place it in public repositories
- paste it into logs
- share it unnecessarily

---

## 11. Runner Installation on Linux

For a Linux VM, the conceptual process is:

```text
Provision VM
 ↓
Install GitLab Runner
 ↓
Configure authentication
 ↓
Start service
 ↓
Verify runner
```

The exact installation commands depend on distribution and GitLab Runner version.

---

## 12. Runner Service

On Linux, Runner commonly operates as a system service.

Conceptual commands:

```bash
systemctl status gitlab-runner
systemctl start gitlab-runner
systemctl restart gitlab-runner
```

For production, monitor service state and restart behavior.

---

## 13. Runner Configuration

A Runner stores configuration describing:

- GitLab endpoint
- authentication
- executor
- tags
- concurrency
- environment settings
- cache configuration
- resource behavior

Configuration location depends on installation and operating system.

---

## 14. Runner Executor

The executor determines how a CI job runs.

Common executor types include:

- Shell
- Docker
- Kubernetes
- SSH
- Docker Machine in older/deprecated architectures
- other supported executors depending on Runner version

For modern DevOps environments, Docker and Kubernetes executors are particularly common.

---

## 15. Shell Executor

With Shell executor:

```text
GitLab Job
   ↓
Runner host
   ↓
Host shell
   ↓
Commands
```

Example:

```bash
mvn clean package
```

runs directly on the Runner host.

---

## 16. Shell Executor Advantages

Advantages:

- simple
- fast startup
- easy access to host tools
- straightforward troubleshooting

Useful for:

- controlled internal servers
- special host-level workloads
- environments where container isolation is unnecessary

---

## 17. Shell Executor Risks

Major risk:

```text
Job
 ↓
Host
```

The job can potentially access the Runner host.

If untrusted code executes:

```text
Untrusted CI
 ↓
Runner host
 ↓
Other workloads
```

the blast radius can be significant.

---

## 18. Shell Executor Production Rule

Do not use a highly privileged shared Shell Runner for untrusted repositories.

If Shell executor is required:

- isolate the host
- restrict project access
- minimize credentials
- patch the OS
- monitor the host
- avoid running unrelated workloads

---

## 19. Docker Executor

Architecture:

```text
Runner Host
    ↓
Docker
    ↓
Job Container
    ↓
Commands
```

Example:

```yaml
image: python:3.12
```

The job runs inside the selected container image.

---

## 20. Docker Executor Advantages

Advantages:

- isolation from host process space compared with Shell
- reproducible environments
- easy dependency management
- clean job environments
- different images per job

Example:

```text
Python job → Python image
Terraform job → Terraform image
Node job → Node image
```

---

## 21. Docker Executor Limitations

Consider:

- container escape risk
- Docker daemon access
- privileged mode
- image supply chain
- network access
- storage
- build strategy

Containerization reduces risk but is not equivalent to perfect isolation.

---

## 22. Docker-in-Docker

Docker-in-Docker is commonly abbreviated:

```text
DinD
```

Concept:

```text
Runner
  ↓
Job container
  ↓
Docker daemon
  ↓
Build container/image
```

It can be useful but introduces additional complexity.

---

## 23. Docker Socket Mount

Another pattern:

```text
Job container
      ↓
/var/run/docker.sock
      ↓
Host Docker daemon
```

This can provide extensive control over the host Docker environment.

Treat it as a high-risk privilege boundary.

---

## 24. DinD vs Docker Socket

### DinD

```text
Job
 ↓
Dedicated Docker daemon
```

### Socket

```text
Job
 ↓
Host Docker daemon
```

Socket access can expose host-level capabilities.

The correct architecture depends on security and build requirements.

---

## 25. Kaniko / Build Tools

Container images can also be built without mounting the host Docker socket using suitable rootless/build-oriented tooling.

Possible options vary by organization and platform.

The important principle:

> Avoid granting more host privilege than the build actually requires.

---

## 26. Kubernetes Executor

Architecture:

```text
GitLab
  ↓
Runner
  ↓
Kubernetes API
  ↓
Pod
  ↓
Job
```

Each CI job can execute in a Kubernetes Pod.

This is highly useful for Kubernetes-based organizations.

---

## 27. Kubernetes Executor Advantages

Advantages:

- ephemeral job environments
- elastic scheduling
- Kubernetes-native resource controls
- clean job lifecycle
- strong integration with EKS/Kubernetes

Example:

```text
Job 1 → Pod
Job 2 → Pod
Job 3 → Pod
```

---

## 28. Kubernetes Executor Job Lifecycle

Conceptually:

```text
Job queued
   ↓
Runner receives job
   ↓
Runner requests Pod
   ↓
Pod scheduled
   ↓
Container starts
   ↓
CI commands execute
   ↓
Logs returned
   ↓
Pod cleaned up
```

---

## 29. GitLab Runner on EKS

A common AWS architecture:

```text
GitLab
   ↓
GitLab Runner
   ↓
EKS
   ↓
Ephemeral CI Pods
   ↓
AWS services
```

The Runner itself can operate inside the EKS cluster.

---

## 30. Runner Namespace

Create a dedicated namespace conceptually:

```text
gitlab-runner
```

Example:

```bash
kubectl create namespace gitlab-runner
```

This separates Runner resources from application workloads.

---

## 31. Runner Service Account

The Runner needs Kubernetes permissions.

Do not automatically grant:

```text
cluster-admin
```

Use the minimum permissions needed.

---

## 32. Runner RBAC

Conceptual:

```text
GitLab Runner
    ↓
ServiceAccount
    ↓
Role / ClusterRole
    ↓
Kubernetes API
```

Permissions should cover only required operations.

---

## 33. Runner Permissions vs Job Permissions

Important distinction:

```text
Runner control plane permissions
```

are not necessarily the same as:

```text
CI job application permissions
```

Avoid giving every job the Runner's maximum privileges.

---

## 34. Protected Runner

A protected Runner can be restricted to protected branches/tags according to GitLab configuration.

Useful for:

```text
production
release
trusted pipelines
```

Example:

```text
Untrusted MR
   ↓
Unprotected runner

Production
   ↓
Protected runner
```

---

## 35. Runner Tags

Tags help route jobs to appropriate Runners.

Example:

```yaml
deploy_production:
  tags:
    - production
```

Runner:

```text
tags:
  production
```

Only eligible Runners can execute the job.

---

## 36. Tag Strategy

Example:

```text
docker
kubernetes
terraform
production
gpu
arm64
```

Use tags to represent execution capabilities or trust boundaries.

Do not create hundreds of unnecessary tags.

---

## 37. Tags Are Not a Complete Security Boundary

A job requesting:

```yaml
tags:
  - production
```

does not automatically mean production is secure.

Combine:

```text
Tags
+
Protected runner
+
Protected environment
+
Protected variables
+
Branch protection
```

---

## 38. Runner Concurrency

A Runner can execute a configured number of jobs concurrently.

Concept:

```text
Runner
 ├── Job A
 ├── Job B
 └── Job C
```

Too low:

```text
Long queues
```

Too high:

```text
CPU/memory/network contention
```

Tune based on workload.

---

## 39. Global vs Runner Concurrency

There can be limits at different levels.

Concept:

```text
Global runner capacity
      ↓
Individual runner
      ↓
Executor capacity
      ↓
Job resources
```

Always determine where the bottleneck is.

---

## 40. Runner Queue

When jobs are pending:

```text
GitLab
 ↓
Job Queue
 ↓
Eligible Runner?
```

If no Runner matches:

```text
Job remains pending
```

Common reasons:

- tag mismatch
- Runner offline
- protected mismatch
- no capacity
- Runner not assigned to project/group

---

## 41. Pending Job Troubleshooting

Check:

```text
Job tags
Runner tags
Runner status
Runner scope
Protected status
Concurrency
Runner capacity
```

This should be your first diagnostic sequence.

---

## 42. Runner Offline

Check on Linux:

```bash
systemctl status gitlab-runner
```

Then:

```bash
journalctl -u gitlab-runner
```

Look for:

- authentication errors
- network errors
- configuration errors
- service crashes

---

## 43. Runner Connectivity

Runner must reach GitLab.

Check:

```text
DNS
HTTPS
Firewall
Proxy
TLS
GitLab URL
```

If the Runner cannot communicate with GitLab, jobs may remain pending.

---

## 44. Runner Registration Failure

Possible causes:

- incorrect authentication
- wrong GitLab URL
- expired/revoked credentials
- network restriction
- invalid configuration
- unsupported Runner version

Validate the Runner configuration rather than repeatedly re-registering it.

---

## 45. Runner Version

Keep Runner versions supported and compatible with the GitLab environment.

A large version mismatch can introduce:

- feature differences
- executor problems
- unexpected behavior
- security exposure

Use a controlled upgrade strategy.

---

## 46. Runner Upgrade Strategy

Production approach:

```text
Test Runner
 ↓
Canary project
 ↓
Observe
 ↓
Upgrade pool gradually
 ↓
Monitor
```

Avoid upgrading every production Runner simultaneously.

---

## 47. Runner Image Updates

For Docker/Kubernetes executors, job images also need lifecycle management.

Example:

```text
python:3.12
```

may change over time if not pinned precisely.

For reproducibility, use controlled image versions/digests where appropriate.

---

## 48. Runner Host Patching

Shell/Docker Runner hosts require:

- OS patching
- Docker/container runtime updates
- certificate updates
- security monitoring
- disk cleanup
- capacity monitoring

A Runner is production infrastructure.

---

## 49. Disk Usage on Runner

Common causes:

```text
Docker images
Docker layers
Build workspaces
Artifacts
Caches
Logs
```

Symptoms:

```text
No space left on device
```

Check:

```bash
df -h
du -sh /var/lib/docker/*
```

Use carefully in production.

---

## 50. Docker Cleanup

Before cleanup:

```text
Identify unused resources
 ↓
Check running jobs
 ↓
Check required images
 ↓
Remove safely
```

Do not blindly execute destructive Docker cleanup on a busy shared Runner.

---

## 51. Runner CPU Saturation

Symptoms:

```text
Jobs slow
Load high
Queue grows
```

Check:

```bash
top
```

or equivalent monitoring.

Potential fixes:

- reduce concurrency
- increase CPU
- autoscale
- optimize jobs
- separate heavy workloads

---

## 52. Runner Memory Pressure

Symptoms:

```text
OOM
job killed
container evicted
host swap pressure
```

Check:

```bash
free -h
```

For Kubernetes:

```bash
kubectl top nodes
kubectl top pods
```

if metrics are available.

---

## 53. Kubernetes Runner Pod Pending

Check:

```bash
kubectl get pods -n gitlab-runner
kubectl describe pod <pod> -n gitlab-runner
```

Look for:

- insufficient CPU
- insufficient memory
- node selectors
- taints/tolerations
- image pull errors
- RBAC
- PVC issues

---

## 54. Kubernetes Runner Pod Evicted

Possible causes:

```text
Memory pressure
Disk pressure
Ephemeral storage pressure
Node pressure
```

Check:

```bash
kubectl describe node <node>
```

and:

```bash
kubectl get events -n gitlab-runner
```

---

## 55. Kubernetes Runner Resource Requests

Job Pods should have appropriate resources.

Concept:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

Exact settings depend on workload.

---

## 56. Why Requests Matter

Requests influence scheduling.

```text
Pod request
   ↓
Kubernetes scheduler
   ↓
Suitable node
```

Without appropriate requests, CI workloads can cause poor cluster utilization.

---

## 57. Why Limits Matter

Limits prevent a job from consuming unlimited resources.

But overly low limits cause:

```text
OOMKilled
CPU throttling
failed builds
```

Tune based on actual job requirements.

---

## 58. Runner Namespace ResourceQuota

A shared EKS Runner namespace can use quotas to control aggregate usage.

Concept:

```text
gitlab-runner namespace
       ↓
ResourceQuota
       ↓
Maximum CI consumption
```

This protects the cluster from runaway CI workloads.

---

## 59. LimitRange

A Kubernetes `LimitRange` can provide default/min/max resource behavior for namespace workloads.

This can reduce accidental resource abuse.

---

## 60. Node Pool Separation

For production EKS:

```text
Application Nodes
        +
CI Runner Nodes
```

may be separated.

Example:

```text
EKS
├── application node group
└── CI node group
```

Benefits:

- workload isolation
- predictable capacity
- easier cost analysis
- reduced CI impact on applications

---

## 61. Taints and Tolerations

CI nodes can be tainted:

```text
ci=true:NoSchedule
```

Runner Pods can tolerate the taint.

Concept:

```text
CI Node
   ↑
Toleration
   ↑
Runner Pod
```

This keeps general workloads away from dedicated CI capacity.

---

## 62. Node Selectors

Runner jobs can target specific node pools.

Concept:

```yaml
nodeSelector:
  workload: ci
```

Use labels that represent stable infrastructure intent.

---

## 63. Affinity

Node affinity can provide more flexible placement.

Useful for:

- architecture
- instance type
- workload class
- dedicated CI nodes

Avoid overly restrictive affinity that causes Pods to remain pending.

---

## 64. ARM and x86 Runners

If your application builds for:

```text
amd64
arm64
```

you may need architecture-specific Runners.

Example tags:

```text
amd64
arm64
```

Container builds must match target architecture.

---

## 65. Autoscaling Runners

A fixed Runner pool:

```text
10 runners
```

may be inefficient during low traffic.

Autoscaling:

```text
Low demand → few workers
High demand → more workers
```

reduces idle infrastructure cost.

---

## 66. Runner Autoscaling Architecture

Conceptually:

```text
GitLab
 ↓
Runner Manager
 ↓
Autoscaling mechanism
 ↓
Ephemeral workers
 ↓
Jobs
```

Exact autoscaling implementation depends on Runner version and infrastructure.

---

## 67. Ephemeral Runners

An ephemeral worker exists for limited job execution.

Concept:

```text
Create worker
 ↓
Run job
 ↓
Collect result
 ↓
Destroy worker
```

Benefits:

- reduced persistence
- cleaner environments
- lower cross-job contamination
- easier recovery

---

## 68. Persistent vs Ephemeral

### Persistent

```text
Runner
 ↓
Job A
 ↓
Job B
 ↓
Job C
```

Risk:

```text
State can remain
```

### Ephemeral

```text
Worker A → Job A → destroy
Worker B → Job B → destroy
```

Cleaner trust boundary.

---

## 69. Workspace Contamination

A persistent Runner can accidentally retain:

```text
source files
credentials
build outputs
dependency caches
temporary files
```

Ephemeral execution reduces this risk.

---

## 70. Shared Runner Security

A shared Runner should be treated as a high-value asset.

Potential threats:

```text
Malicious repository
 ↓
CI command execution
 ↓
Runner compromise
 ↓
Credential theft
 ↓
Lateral movement
```

Isolation is essential.

---

## 71. Runner Least Privilege

Runner infrastructure should have only the permissions required.

Example AWS:

```text
ECR push role
```

instead of:

```text
AdministratorAccess
```

Example Kubernetes:

```text
specific namespace permissions
```

instead of:

```text
cluster-admin
```

---

## 72. GitLab Runner and AWS IAM

Recommended architecture:

```text
GitLab CI
    ↓
OIDC
    ↓
AWS STS
    ↓
IAM Role
    ↓
Required AWS APIs
```

For ECR-only build jobs, the role should have only the required ECR permissions.

---

## 73. GitLab Runner and EKS IAM

If Runner Pods need AWS access:

```text
Runner Pod
   ↓
Kubernetes/AWS identity mechanism
   ↓
IAM Role
```

Prefer workload identity mechanisms supported by the EKS environment rather than distributing static access keys.

---

## 74. Runner and Kubernetes RBAC

The Runner itself may need to create/manage job Pods.

But the application deployment job should not automatically inherit:

```text
cluster-admin
```

Separate:

```text
Runner control permissions
+
Deployment permissions
```

where architecture permits.

---

## 75. Protected Runner + Production

A strong pattern:

```text
Protected main
      ↓
Protected Runner
      ↓
Protected variables
      ↓
Protected environment
      ↓
Production
```

Each layer reduces the chance of unauthorized deployment.

---

## 76. Runner Tags + Environment

Example:

```yaml
deploy_production:
  tags:
    - production
  environment:
    name: production
```

Then configure:

```text
production Runner
+
protected environment
```

This is stronger than tags alone.

---

## 77. Runner Network Segmentation

Runner nodes should not have unrestricted network access.

Allow only required destinations:

```text
GitLab
Container registry
AWS APIs
Package repositories
Required deployment endpoints
```

Network restrictions reduce lateral movement.

---

## 78. Proxy Environment

Enterprise environments may require:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

Incorrect proxy configuration can cause:

- package installation failures
- GitLab connectivity issues
- Docker pull failures
- AWS API errors

Document the required network path.

---

## 79. DNS Troubleshooting

If jobs cannot reach:

```text
registry
GitLab
AWS endpoint
package repository
```

check:

```bash
nslookup <hostname>
```

or:

```bash
dig <hostname>
```

depending on installed tools.

---

## 80. TLS Troubleshooting

Check:

```text
Certificate validity
CA trust
Proxy interception
Hostname
System time
TLS version
```

Do not disable TLS verification as a permanent fix.

Avoid:

```bash
curl -k
```

in production workflows unless explicitly justified for controlled diagnostics.

---

## 81. Runner Network Security

A production Runner should have:

```text
Outbound access
  → only what is required

Inbound access
  → minimal/none where possible
```

The Runner generally initiates communication to GitLab.

---

## 82. Runner Credentials

Credentials can include:

- GitLab Runner authentication
- cloud identity
- registry credentials
- package credentials
- deployment identity

Protect each independently.

Do not place all credentials in one universal Runner configuration.

---

## 83. Credential Blast Radius

Bad:

```text
One Runner
 ↓
Administrator AWS credentials
 ↓
All projects
```

Better:

```text
Project/job
 ↓
specific identity
 ↓
specific permissions
```

---

## 84. Runner Shell Escape Risk

With Shell executor:

```text
Job script
 ↓
Host shell
```

A malicious script can potentially access:

```text
host filesystem
processes
network
credentials
```

This is why Shell runners require strong trust isolation.

---

## 85. Privileged Containers

A privileged Docker/Kubernetes job may have powerful host capabilities.

Use only when necessary.

Before enabling privileged mode ask:

```text
Why is it required?
Can rootless tooling work?
Can a dedicated build service be used?
Can the workload be isolated?
```

---

## 86. Docker Socket Risk

Mounting:

```text
/var/run/docker.sock
```

can provide powerful access to the host Docker daemon.

Treat it as sensitive host-level privilege.

---

## 87. Runner Supply Chain

The Runner itself is software.

Secure:

```text
Runner binary
Runner image
Base OS
Container runtime
Helm chart if used
Configuration
Dependencies
```

Keep them patched and verify trusted sources.

---

## 88. Runner Helm Deployment

A Kubernetes-based Runner can commonly be installed through a Helm-based workflow.

Concept:

```text
Helm
 ↓
GitLab Runner chart
 ↓
EKS
 ↓
Runner deployment
```

Pin approved chart/app versions in production.

---

## 89. Runner Helm Values

Configuration may control:

- GitLab URL
- authentication
- RBAC
- executor
- concurrency
- node selectors
- tolerations
- resources
- metrics
- security context

Keep sensitive values out of Git when they are secret material.

---

## 90. Runner Security Context

For Kubernetes Runner Pods, configure security context appropriately.

Consider:

```text
runAsNonRoot
readOnlyRootFilesystem
capabilities
seccomp
privilege escalation
```

Exact settings depend on the job executor and build requirements.

---

## 91. Runner Pod Security

Kubernetes Pod Security standards/policies may restrict Runner jobs.

Potential conflicts:

```text
Security policy
     ↓
Runner requires privileged behavior
     ↓
Pod rejected
```

Solve the architectural conflict rather than disabling cluster security globally.

---

## 92. Runner and Container Builds on EKS

If CI needs Docker image builds inside EKS, choose a build strategy deliberately.

Options may include:

```text
Docker-in-Docker
BuildKit
rootless builders
Kaniko
specialized build services
```

Evaluate:

- security
- performance
- cache
- reproducibility
- operational complexity

---

## 93. Build Cache on Kubernetes Runners

Ephemeral Pods can lose local cache after completion.

Possible strategies:

```text
Registry cache
Object storage
Persistent volume
Remote cache
```

Use a cache that fits the build system.

Do not introduce persistent storage merely to save a few seconds unless justified.

---

## 94. Kubernetes Runner Storage

Builds may need:

```text
workspace
temporary files
Docker layers
artifact data
```

Monitor:

```text
ephemeral-storage
```

A job can fail even when CPU and memory are available if disk/ephemeral storage is exhausted.

---

## 95. Runner Job Timeout

Kubernetes executor jobs may be affected by:

- image pull time
- Pod scheduling
- startup time
- command duration
- artifact upload

Separate:

```text
queue delay
+
Pod startup
+
execution
```

when troubleshooting.

---

## 96. Runner Pod Startup Failure

Check:

```bash
kubectl describe pod <runner-job-pod> -n gitlab-runner
```

Look for:

```text
FailedScheduling
ErrImagePull
ImagePullBackOff
CreateContainerError
AdmissionDenied
RBAC
```

---

## 97. ImagePullBackOff on Runner Pod

Check:

```text
Image name
Registry
Credentials
Network
Architecture
Image availability
```

If using a private registry, verify the Pod's image-pull authentication.

---

## 98. Runner Pod Permission Denied

Potential causes:

```text
RBAC
Pod Security
SecurityContext
Filesystem permissions
ServiceAccount
```

Do not solve every permission problem by making the Pod privileged.

---

## 99. Runner Pod Cannot Create Resources

If Runner cannot create job Pods:

```text
Check ServiceAccount
 ↓
Check Role/ClusterRole
 ↓
Check RoleBinding
 ↓
Check namespace
 ↓
Check API errors
```

Use least privilege.

---

## 100. Runner Job Cannot Access AWS

Check:

```text
Pod identity/workload identity
IAM role
Trust policy
AWS account
Region
AWS SDK/CLI
Network
```

First verify identity:

```bash
aws sts get-caller-identity
```

Then verify authorization.

---

## 101. Runner Job Cannot Push to ECR

Troubleshoot:

```text
AWS identity
 ↓
ECR repository
 ↓
Region
 ↓
Authentication
 ↓
Push permissions
 ↓
Image architecture/tag
```

Do not grant broad AWS permissions without identifying the missing API action.

---

## 102. Runner Job Cannot Access Kubernetes

If direct Kubernetes access is intentionally required:

```text
kubectl
 ↓
cluster endpoint
 ↓
authentication
 ↓
RBAC
```

For production GitOps, prefer:

```text
GitLab
 ↓
GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

instead of granting CI direct cluster-admin access.

---

## 103. Runner Job Cannot Access GitLab

Check:

```text
DNS
TLS
Proxy
GitLab URL
Runner authentication
Network
Firewall
```

For Git operations against another repository, use an appropriately scoped identity.

---

## 104. Runner Cache Troubleshooting

If cache is not restored:

```text
Cache key
 ↓
Cache availability
 ↓
Runner configuration
 ↓
Storage backend
 ↓
Permissions
```

A cache miss should generally cause slower execution, not production correctness failure.

---

## 105. Runner Artifact Upload Failure

Possible causes:

- network
- GitLab availability
- storage
- artifact size
- timeout
- permissions

If artifact is critical, investigate whether the job should be considered successful without it.

---

## 106. Runner Disk Full

Sequence:

```bash
df -h
```

Then identify large areas:

```bash
du -xh /var/lib/docker | sort -h | tail
```

or use appropriate system tools.

Before deleting:

```text
Check active jobs
Check running containers
Check required caches
```

---

## 107. Runner Log Analysis

Linux:

```bash
journalctl -u gitlab-runner
```

Kubernetes:

```bash
kubectl logs deployment/<runner> -n gitlab-runner
```

Also inspect:

```bash
kubectl get events -n gitlab-runner
```

---

## 108. Runner Health Monitoring

Monitor:

```text
Runner online/offline
Job queue
Job duration
Failure rate
CPU
Memory
Disk
Network
Pod scheduling
```

A Runner platform is part of the delivery system's reliability boundary.

---

## 109. Runner Metrics

If metrics are enabled in your architecture, collect useful indicators such as:

```text
job execution duration
concurrency
request latency
worker utilization
error rate
```

Send operational metrics to your monitoring stack where appropriate.

---

## 110. ELK for Runner Logs

Architecture:

```text
Runner
 ↓
Logs
 ↓
Log collection
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Useful searches:

```text
runner offline
authentication error
image pull
disk full
job timeout
```

Do not ingest secrets.

---

## 111. Prometheus + Grafana for Runners

Architecture:

```text
Runner metrics
 ↓
Prometheus
 ↓
Grafana
```

Useful dashboards:

```text
Runner availability
Queue time
Job duration
CPU
Memory
Failure rate
```

---

## 112. Runner Alerting

Useful alerts:

```text
Runner unavailable
Queue time high
Disk > threshold
Memory pressure
CPU saturation
Job failure spike
Kubernetes Runner Pods pending
```

Alerts should identify actionable problems.

---

## 113. Runner Capacity Planning

Estimate:

```text
Average jobs/minute
+
Average job duration
+
Peak workload
```

Then determine:

```text
Required concurrent workers
```

Leave enough headroom for bursts.

---

## 114. Noisy Neighbor Problem

A heavy Docker build can consume:

```text
CPU
Memory
Disk I/O
Network
```

and slow other jobs.

Solutions:

- dedicated runners
- Kubernetes resource limits
- separate node pools
- workload-specific tags
- concurrency control

---

## 115. Runner Pool Design

Example:

```text
Runner Pool
├── general
├── docker-build
├── terraform
├── security
└── production
```

Each pool can have different:

- permissions
- capacity
- executor
- node placement

---

## 116. Production Runner Separation

A strong pattern:

```text
Untrusted CI
   ↓
General runners

Trusted release
   ↓
Protected runners

Production deployment
   ↓
Protected deployment environment
```

This reduces blast radius.

---

## 117. Security Runner

Security scanning may require special tools or resources.

Example:

```text
Trivy
SonarQube
Veracode
```

Use dedicated images and appropriate Runner capacity rather than installing everything onto every host.

---

## 118. Terraform Runner

Terraform jobs may require:

```text
Terraform binary
Cloud identity
Network access
Backend access
```

Use a controlled Runner and scoped IAM role.

---

## 119. Docker Build Runner

Docker builds may require:

```text
CPU
Memory
Disk
registry access
build cache
```

Dedicated build runners can improve predictability.

---

## 120. Production Deployment Runner

If direct deployment is required:

```text
Protected Runner
+
Protected environment
+
Restricted IAM
+
Restricted Kubernetes RBAC
```

But in your GitOps architecture:

```text
GitLab CI
 → GitOps update

ArgoCD
 → EKS
```

is preferred.

---

## 121. Runner and GitOps Security

The CI job should generally have:

```text
GitOps repository write access
```

rather than:

```text
Kubernetes cluster-admin
```

This preserves the GitOps control boundary.

---

## 122. Runner and ArgoCD

Recommended:

```text
GitLab Runner
   ↓
Update GitOps repository
   ↓
ArgoCD detects commit
   ↓
ArgoCD syncs EKS
```

This avoids making the CI Runner the Kubernetes production control plane.

---

## 123. Runner Disaster Recovery

Plan for:

```text
Runner host failure
Runner configuration loss
Credential compromise
EKS Runner node failure
GitLab connectivity loss
```

Use infrastructure as code where practical.

---

## 124. Runner as Code

Instead of manually configuring production Runners:

```text
Terraform
+
Helm
+
GitLab configuration
```

can make Runner infrastructure reproducible.

Example:

```text
Terraform
 ↓
EKS
 ↓
Helm
 ↓
GitLab Runner
```

---

## 125. Runner Upgrade as Code

A controlled process:

```text
Update version
 ↓
Terraform/Helm diff
 ↓
MR
 ↓
Test
 ↓
Canary
 ↓
Rollout
```

This aligns Runner management with DevOps practices.

---

## 126. Runner Backup

Back up what is actually required.

Potentially important:

- configuration
- infrastructure definitions
- Helm values
- authentication/bootstrap material according to security policy

Do not create insecure copies of secrets merely for backup.

---

## 127. Runner Replacement

Ephemeral infrastructure should make replacement easy:

```text
Old Runner
   ↓
Drain/stop
   ↓
New Runner
   ↓
Validate
   ↓
Remove old
```

Avoid making a single Runner irreplaceable.

---

## 128. Runner Drain

Before maintenance:

```text
Stop accepting new jobs
 ↓
Wait for running jobs
 ↓
Upgrade/replace
```

Exact GitLab Runner behavior depends on configuration and operational method.

---

## 129. Runner Security Checklist

```text
[ ] Supported Runner version
[ ] Trusted installation source
[ ] Authentication protected
[ ] Minimal project scope
[ ] Protected runners for trusted workloads
[ ] No unnecessary privileged mode
[ ] No unnecessary Docker socket access
[ ] Least-privilege IAM
[ ] Least-privilege Kubernetes RBAC
[ ] Network restrictions
[ ] Patched host
[ ] Disk monitoring
[ ] CPU/memory monitoring
[ ] Logs monitored
[ ] Secrets protected
[ ] Ephemeral execution where appropriate
```

---

## 130. Senior Interview — What Is GitLab Runner?

> GitLab Runner is the execution agent that receives eligible GitLab CI jobs and runs them using an executor such as Shell, Docker, or Kubernetes. GitLab orchestrates the pipeline while Runner provides the execution environment.

---

## 131. Senior Interview — Shell vs Docker Executor

> Shell runs commands directly on the Runner host, so it is simple and fast but has a larger host-isolation risk. Docker runs jobs inside containers, giving cleaner environments and dependency isolation. For shared or untrusted workloads, I prefer stronger isolation rather than a shared privileged Shell Runner.

---

## 132. Senior Interview — Why Kubernetes Executor?

> Kubernetes executor creates CI job Pods dynamically. It provides ephemeral environments, elastic scheduling, Kubernetes resource controls, and good integration with EKS. It is especially useful when CI workloads vary significantly.

---

## 133. Senior Interview — How Would You Run GitLab Runner on EKS?

> I would deploy the Runner into a dedicated namespace using an approved Helm-based configuration, configure the Kubernetes executor, use least-privilege RBAC, dedicate or isolate CI capacity where needed, configure resource requests/limits, and monitor Runner Pods. For AWS access I would use workload identity/OIDC-style short-lived credentials rather than static keys.

---

## 134. Senior Interview — Why Not Give Runner Cluster-Admin?

> Because Runner compromise would then become cluster compromise. The Runner should have only the Kubernetes permissions required to create and manage its CI workload. Application deployment permissions should be separated where possible, and GitOps should be preferred for production.

---

## 135. Senior Interview — How Do You Secure Shared Runners?

> I separate trusted and untrusted workloads, use protected runners for protected branches/environments, minimize credentials, avoid privileged execution, isolate projects where necessary, restrict network access, patch the Runner, monitor it, and use ephemeral workers where practical.

---

## 136. Senior Interview — Job Is Stuck in Pending

> I first check Runner availability, tags, project/group assignment, protected status, concurrency, and executor capacity. A common cause is a job requesting a tag that no available Runner has.

---

## 137. Senior Interview — Runner Is Offline

> I check the Runner service, logs, GitLab connectivity, DNS/TLS/proxy configuration, authentication, host health, and version compatibility. I restore the Runner or replace it if the infrastructure is unhealthy.

---

## 138. Senior Interview — Kubernetes Runner Pod Is Pending

> I inspect `kubectl describe pod` and events, then check CPU/memory requests, node capacity, taints/tolerations, node selectors, image availability, admission policies, and RBAC.

---

## 139. Senior Interview — Runner Host Disk Is Full

> I first identify active jobs and determine whether Docker layers, workspaces, caches, artifacts, or logs consume the disk. I avoid destructive cleanup during active builds, clean safe unused data, and then implement retention/cleanup and monitoring to prevent recurrence.

---

## 140. Senior Interview — CI Needs Docker Builds on EKS

> I would choose the build architecture based on security and requirements. I would avoid exposing the host Docker socket unless justified and evaluate rootless BuildKit, Kaniko, DinD, or another approved builder. The priority is least privilege and reproducibility.

---

## 141. Senior Interview — Why Ephemeral Runners?

> Ephemeral runners reduce state leakage between jobs, limit persistence after compromise, provide clean execution environments, and scale with workload. They are especially useful for shared CI infrastructure.

---

## 142. Senior Interview — How Do You Scale Runners?

> I measure queue time and concurrency, then scale the Runner pool horizontally or use autoscaling. In Kubernetes, I can use additional worker capacity and ephemeral job Pods. I also separate heavy workloads so one workload class does not starve others.

---

## 143. Senior Interview — How Do You Prevent CI From Affecting EKS Applications?

> I separate CI capacity from application capacity where appropriate, use resource requests and limits, node selectors/taints for dedicated CI nodes, namespace quotas, and concurrency controls. I monitor both Runner and application workloads.

---

## 144. Senior Interview — Why Use Protected Runners?

> Protected runners provide an execution boundary for trusted branches/tags. They can help prevent untrusted pipelines from reaching sensitive execution environments, but they must be combined with protected variables, environments, branch protection, and least-privilege credentials.

---

## 145. Senior Interview — Production Deployment in GitOps

> I would avoid giving the Runner direct cluster-admin access. GitLab CI builds and scans the image, publishes it to ECR, updates the GitOps repository with the approved image identity, and ArgoCD reconciles that desired state into EKS.

---

## 146. Senior Interview — Runner Compromise

> I would isolate the Runner, stop affected jobs, rotate credentials, inspect GitLab and AWS audit logs, identify affected projects/artifacts, rebuild the Runner from a trusted source, and review the privilege and isolation model that allowed the compromise.

---

## 147. Production Runner Architecture

Recommended architecture for your stack:

```text
                       GitLab
                          │
                          ▼
                  Pipeline / Job Queue
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
        General Runners        Protected Runners
              │                       │
              ▼                       ▼
        Validation/Build        Trusted Release
              │                       │
              └───────────┬───────────┘
                          ▼
                       ECR
                          │
                          ▼
                   GitOps Repository
                          │
                          ▼
                        ArgoCD
                          │
                          ▼
                         EKS
```

---

## 148. Recommended EKS Runner Architecture

```text
EKS Cluster
│
├── Application Node Group
│    ├── User service
│    ├── Cart service
│    ├── Order service
│    └── Other workloads
│
└── CI Node Group
     ├── Runner job Pod
     ├── Runner job Pod
     └── Runner job Pod
```

Use:

```text
taints
+
tolerations
+
node selectors
+
resource limits
```

where appropriate.

---

## 149. Production Runner Flow

```text
Developer
    ↓
GitLab
    ↓
MR / Push
    ↓
Pipeline
    ↓
Eligible Runner
    ↓
Ephemeral Job Environment
    ↓
Build/Test/Security
    ↓
Immutable Artifact
    ↓
GitOps
    ↓
ArgoCD
    ↓
EKS
```

---

## 150. Final Runner Checklist

```text
[ ] Runner scope understood
[ ] Correct executor
[ ] Runner authentication protected
[ ] Tags designed
[ ] Protected Runner configured where required
[ ] Shared Runner risks assessed
[ ] Shell executor avoided for untrusted code
[ ] Docker privilege minimized
[ ] Kubernetes RBAC least privilege
[ ] AWS identity least privilege
[ ] OIDC/workload identity preferred
[ ] Dedicated CI namespace
[ ] Resource requests/limits
[ ] Node pool strategy
[ ] Taints/tolerations where required
[ ] Autoscaling strategy
[ ] Disk cleanup
[ ] CPU/memory monitoring
[ ] Network restrictions
[ ] Logs monitored
[ ] Runner upgrades controlled
[ ] Disaster recovery
[ ] Runner replacement process
[ ] Production GitOps boundary
```

---

## 151. Final Mental Model

```text
                 GitLab
                    │
                    ▼
              CI/CD Pipeline
                    │
                    ▼
              Runner Selection
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Shell      Docker   Kubernetes
                              │
                              ▼
                         Ephemeral Pod
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                  Build      Test    Security
                    │
                    ▼
                   ECR
                    │
                    ▼
              GitOps Repository
                    │
                    ▼
                  ArgoCD
                    │
                    ▼
                   EKS
```

> **GitLab Runner is part of the production delivery platform, not just a utility that executes scripts. Runner choice, isolation, permissions, capacity, networking, and lifecycle directly affect CI reliability and the security blast radius of every pipeline.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md ✓
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
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `07-GitLab-Variables-Secrets-and-Environments.md`**
