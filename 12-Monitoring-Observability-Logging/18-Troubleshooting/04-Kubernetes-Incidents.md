# 04 - Kubernetes Incidents

> Production Kubernetes Incident Troubleshooting — Pod Failures, Node Failures, Scheduling, Networking, DNS, Storage, ConfigMaps, Secrets, Ingress, Deployments, StatefulSets, HPA, Cluster Autoscaling, Resource Pressure, Observability, EKS, Incident Response, Root Cause Analysis and DevOps Interview Preparation

---

# 1. Kubernetes Incident Fundamentals

A Kubernetes incident occurs when a workload, cluster component, dependency, or platform behavior prevents the system from meeting its expected availability, performance, or reliability requirements.

Typical production flow:

    User
      |
      v
    Route53 / DNS
      |
      v
    ALB
      |
      v
    Ingress
      |
      v
    Kubernetes Service
      |
      v
    Pod
      |
      +---- Database
      |
      +---- Cache
      |
      +---- External API

A Kubernetes incident can happen at any layer.

The troubleshooting goal is:

> Identify the first failure, understand its impact, restore service safely, and prevent recurrence.

---

# 2. Kubernetes Incident Mindset

Do not immediately restart the pod.

First determine:

    What is failing?
    Who is affected?
    When did it start?
    What changed?
    Is the problem application, pod, node, cluster, network, storage, or dependency related?

Use:

    Observe
       |
       v
    Scope
       |
       v
    Isolate
       |
       v
    Mitigate
       |
       v
    Validate
       |
       v
    Root Cause
       |
       v
    Prevent

---

# 3. Define Incident Scope

Start by determining whether the incident affects:

    One container
    One pod
    One deployment
    One service
    One namespace
    One node
    Multiple nodes
    One availability zone
    Entire cluster
    External dependency

Examples:

    One Pod failing
        |
        v
    Workload-level problem

    All pods on one node failing
        |
        v
    Node-level problem

    All services unreachable
        |
        v
    Network / ingress / cluster-level problem

Scope prevents random troubleshooting.

---

# 4. First Kubernetes Commands

Start with:

    kubectl get pods -A

Then:

    kubectl get nodes

Then:

    kubectl get events -A --sort-by=.lastTimestamp

Then inspect the affected workload:

    kubectl describe pod <pod> -n <namespace>

And logs:

    kubectl logs <pod> -n <namespace>

For previous container:

    kubectl logs <pod> --previous -n <namespace>

These commands establish the initial state.

---

# 5. Kubernetes Incident Evidence

Before changing anything, collect:

- Pod status
- Container status
- Pod events
- Application logs
- Previous logs
- Node status
- Node events
- Deployment status
- ReplicaSet status
- Service endpoints
- Ingress status
- Recent deployments
- Resource usage
- Network errors
- Storage status

Evidence can disappear after restarts.

---

# 6. Kubernetes Events

Events are often the fastest way to understand scheduling and lifecycle problems.

Use:

    kubectl get events -A --sort-by=.lastTimestamp

Look for:

    FailedScheduling
    FailedMount
    FailedAttachVolume
    BackOff
    Unhealthy
    OOMKilling
    Evicted
    FailedCreatePodSandBox
    ImagePullBackOff
    FailedPullImage

Events provide clues about what Kubernetes attempted and why it failed.

---

# 7. Pod Incident Classification

Common states:

    Pending
    Running
    Succeeded
    Failed
    Unknown

But pod phase alone is not enough.

A pod may be:

    Running

while the container is:

    CrashLoopBackOff

or the application is:

    unhealthy

Always inspect:

    kubectl describe pod

and container status.

---

# 8. Pending Pod Incident

If a pod remains:

    Pending

it usually has not been scheduled successfully or is waiting on resources/conditions.

Check:

    kubectl describe pod <pod> -n <namespace>

Look at:

    Events

Common causes:

- Insufficient CPU
- Insufficient memory
- Node selector mismatch
- Taints
- Affinity rules
- PVC unavailable
- Scheduling constraints
- Resource quota

---

# 9. Pending — Insufficient CPU

Event:

    Insufficient cpu

Meaning:

    Scheduler cannot find a suitable node.

Check:

    kubectl get nodes
    kubectl describe nodes

Review:

    Requests
    Allocatable CPU
    Current workload

Do not immediately increase limits.

First determine whether the cluster lacks capacity or requests are unnecessarily large.

---

# 10. Pending — Insufficient Memory

Event:

    Insufficient memory

Possible causes:

- Cluster genuinely full
- Memory requests too high
- Pods concentrated on nodes
- Autoscaler not adding nodes

Check:

    kubectl describe nodes

Then:

    kubectl top nodes

where Metrics Server is available.

Compare:

    Allocatable
    Requests
    Actual usage

---

# 11. Pending — Node Selector

Pod:

    nodeSelector:
      workload: frontend

Nodes:

    node-a: workload=backend
    node-b: workload=database

No matching node exists.

Result:

    Pod Pending

Check:

    kubectl get nodes --show-labels

Verify the labels actually exist.

---

# 12. Pending — Node Affinity

Affinity may require:

    zone = us-east-1a

If no node satisfies the requirement:

    Pod cannot schedule.

Check:

- Required affinity
- Preferred affinity
- Node labels
- Availability zone distribution

A complex affinity rule can unintentionally make workloads unschedulable.

---

# 13. Pending — Taints and Tolerations

Node:

    taint:
    dedicated=database:NoSchedule

Pod has no matching toleration.

Scheduler rejects the node.

Check:

    kubectl describe node <node>

Look for:

    Taints

Then compare pod tolerations.

---

# 14. Pending — ResourceQuota

Namespace may have:

    CPU quota
    Memory quota
    Object count quota

A pod can fail to create or schedule because the namespace quota is exhausted.

Check:

    kubectl get resourcequota -n <namespace>

Then:

    kubectl describe resourcequota <name> -n <namespace>

---

# 15. Pending — LimitRange

A namespace may have default resource values or constraints.

Check:

    kubectl get limitrange -n <namespace>

A new pod may receive defaults that make scheduling difficult.

Always understand namespace-level resource policies.

---

# 16. ImagePullBackOff

Symptoms:

    ImagePullBackOff
    ErrImagePull

Check:

    kubectl describe pod <pod> -n <namespace>

Look at:

    Events

Common causes:

- Wrong image name
- Wrong tag
- Image does not exist
- Registry authentication
- Network connectivity
- ECR permission issue
- Registry rate limiting

---

# 17. ECR Image Pull Troubleshooting

In EKS:

    Pod
      |
      v
    Node
      |
      v
    ECR
      |
      v
    Image

Check:

- Repository exists
- Tag exists
- Node/pod identity has required permissions
- Network path works
- ECR endpoint reachable

Do not assume the image exists because the repository exists.

Verify the exact tag.

---

# 18. Image Tag Incident

Deployment:

    image: orders:v2.4

ECR contains:

    orders:v2.3

Result:

    ImagePullBackOff

Check exact:

    Repository
    Image
    Tag
    Registry
    Architecture

A typo in the tag is one of the simplest causes of image pull failures.

---

# 19. ImagePullBackOff — Authentication

Possible causes:

- Incorrect imagePullSecret
- Expired credentials
- Missing IAM permissions
- Registry authentication issue

For private registries, verify the authentication mechanism used by the cluster.

Do not store registry passwords directly in application manifests.

---

# 20. CrashLoopBackOff

This means the container starts and repeatedly exits.

Check:

    kubectl logs <pod> -n <namespace>

Then:

    kubectl logs <pod> --previous -n <namespace>

Then:

    kubectl describe pod <pod> -n <namespace>

Look for:

- Application exception
- Missing configuration
- Dependency failure
- Incorrect command
- Permission issue
- OOMKilled
- Probe failure

---

# 21. CrashLoopBackOff — Application Error

Example:

    Connection refused to database

The pod may restart repeatedly.

The Kubernetes symptom is:

    CrashLoopBackOff

but the root cause is:

    Database unavailable

Always follow the application error rather than treating CrashLoopBackOff as the root cause.

---

# 22. CrashLoopBackOff — Wrong Command

Deployment:

    command:
      - /app/start.sh

But image contains:

    /app/startup.sh

Container exits immediately.

Check:

    kubectl describe pod

and image/application configuration.

---

# 23. CrashLoopBackOff — Missing Environment Variable

Application requires:

    DATABASE_URL

but it is missing.

Startup:

    Application
       |
       X
    Missing config
       |
       v
    Exit
       |
       v
    Restart
       |
       v
    CrashLoopBackOff

Check:

    kubectl describe pod

and relevant ConfigMap/Secret references.

---

# 24. OOMKilled

A container may show:

    Reason: OOMKilled

This means the container exceeded its memory limit or was terminated due to memory pressure depending on the exact context.

Check:

    kubectl describe pod <pod>

Then:

    kubectl top pod <pod>

Investigate:

- Memory limit
- Actual usage
- Application memory behavior
- Heap configuration
- Traffic
- Recent release

---

# 25. OOMKilled — Java Application

Example:

    Container memory limit = 1Gi

Java process may have:

    JVM heap
    + native memory
    + threads
    + metaspace
    + libraries

If heap is configured too aggressively:

    JVM memory
       |
       v
    Container limit
       |
       v
    OOMKilled

Do not allocate the entire container memory limit to Java heap.

---

# 26. OOMKilled — Node.js

Possible causes:

- Memory leak
- Large objects
- Traffic increase
- Heap growth
- Large payloads

Check application metrics and restart history.

Do not simply increase memory limits without identifying the growth pattern.

---

# 27. OOMKilled — Python

Investigate:

- Memory-intensive workloads
- Worker count
- Large datasets
- Caching
- Memory leaks
- Request concurrency

Compare memory behavior across healthy and affected pods.

---

# 28. OOMKilled vs Evicted

Important distinction.

## OOMKilled

Usually:

    Container memory limit exceeded

## Evicted

Usually:

    Node resource pressure caused Kubernetes to evict a pod.

For eviction, inspect:

    kubectl describe pod

and:

    kubectl describe node

Do not treat them as the same problem.

---

# 29. Pod Eviction

Common causes:

- Memory pressure
- Disk pressure
- Ephemeral storage pressure
- PID pressure

Check node conditions:

    kubectl describe node <node>

Look for:

    MemoryPressure
    DiskPressure
    PIDPressure

---

# 30. DiskPressure

Node condition:

    DiskPressure=True

Possible causes:

- Container images
- Container logs
- Temporary files
- Ephemeral storage
- Application files

Check:

    df -h

Then:

    du -sh

on the node where appropriate.

---

# 31. Ephemeral Storage Pressure

Applications can consume local storage through:

- Logs
- Temporary files
- EmptyDir
- Container writable layer

If storage grows unexpectedly:

    Pod
       |
       v
    Node disk
       |
       v
    DiskPressure
       |
       v
    Eviction

Set appropriate ephemeral-storage requests/limits where applicable.

---

# 32. Node NotReady

Check:

    kubectl get nodes

If:

    NotReady

inspect:

    kubectl describe node <node>

Potential causes:

- Kubelet failure
- Network problem
- Container runtime issue
- Disk pressure
- Memory pressure
- Node termination
- AWS infrastructure issue

---

# 33. Kubelet Troubleshooting

On the node, where access is available:

    systemctl status kubelet

Check logs:

    journalctl -u kubelet

Look for:

- Runtime errors
- Certificate issues
- API server connectivity
- Disk problems
- CNI errors

In managed EKS environments, the exact access model differs, so use the supported node diagnostics available to your organization.

---

# 34. Container Runtime Problems

If the runtime fails:

    containerd
       |
       X
    Containers cannot start

Symptoms:

- Pods stuck
- Container creation failures
- Node NotReady
- Runtime errors

Check node/runtime health and relevant events.

---

# 35. Node Network Failure

Symptoms:

- Node NotReady
- Pod communication failures
- Service connectivity failures

Check:

- Node network
- CNI
- Routes
- Security Groups
- DNS
- NetworkPolicy

A network issue can look like an application issue.

---

# 36. EKS CNI Troubleshooting

AWS VPC CNI provides pod networking.

If CNI has issues:

    Pod
       |
       X
    Network
       |
       v
    Service communication fails

Check:

    kubectl get pods -n kube-system

Look for CNI pods and their health.

Also inspect:

- IP availability
- ENIs
- Subnets
- IAM permissions
- CNI logs

---

# 37. Pod IP Exhaustion

In VPC-native EKS networking, available IP addresses can become a constraint.

Symptoms:

    Pods Pending
    Network-related scheduling/create failures

Investigate:

- Subnet available IPs
- ENI capacity
- CNI configuration
- Instance type limits

Cluster scaling alone may not solve a subnet IP shortage.

---

# 38. Kubernetes DNS Incident

Symptoms:

    Application cannot resolve:
    database.internal

Check:

    kubectl get pods -n kube-system

Look for CoreDNS.

Then:

    kubectl logs <coredns-pod> -n kube-system

Check DNS from an affected pod using an appropriate diagnostic image/tool.

---

# 39. CoreDNS Failure

If CoreDNS is unhealthy:

    Pod
      |
      v
    DNS Query
      |
      X
    CoreDNS
      |
      v
    Service unreachable by name

Check:

- CoreDNS pods
- Deployment replicas
- CPU/memory
- ConfigMap
- Network connectivity

---

# 40. Kubernetes Service Incident

A Service may exist:

    kubectl get svc

but have:

    No endpoints

Check:

    kubectl get endpoints <service> -n <namespace>

or EndpointSlices:

    kubectl get endpointslice -n <namespace>

If there are no endpoints, traffic has nowhere to go.

---

# 41. Service Selector Problem

Service:

    selector:
      app: orders

Pod label:

    app: order

Mismatch.

Result:

    Service
       |
       X
    No matching pods

Check:

    kubectl get pods --show-labels

Compare Service selectors with pod labels.

---

# 42. Readiness and Service Endpoints

A pod may be:

    Running

but:

    Ready=False

The Service may not send traffic to it.

This is why:

> Running does not mean serving traffic.

Check:

    kubectl get pods

and:

    kubectl describe pod

---

# 43. Readiness Probe Failure

Common causes:

- Application startup slow
- Wrong endpoint
- Wrong port
- Dependency unavailable
- Authentication issue
- Probe timeout too short

Example:

    readinessProbe:
      httpGet:
        path: /health
        port: 8080

If application listens on:

    8081

the probe fails.

---

# 44. Liveness Probe Failure

A liveness failure causes Kubernetes to restart the container.

Possible causes:

- Wrong probe
- Application deadlock
- Probe timeout
- Temporary overload
- Dependency check incorrectly placed in liveness

Do not put every dependency check into liveness.

A temporary database outage should not necessarily cause every application container to restart.

---

# 45. Startup Probe

Startup probes help slow-starting applications.

Flow:

    Container starts
       |
       v
    Startup probe
       |
       v
    Application initializes
       |
       v
    Startup succeeds
       |
       v
    Liveness/readiness become active

Without a startup probe, liveness may kill an application before initialization completes.

---

# 46. Probe Troubleshooting

Check:

    Path
    Port
    Protocol
    Initial delay
    Timeout
    Period
    Failure threshold

Then test the endpoint from the pod where appropriate.

Do not increase probe thresholds blindly.

---

# 47. Service Unreachable — Internal

Example:

    orders -> payment

Check:

    DNS
       |
       v
    Service
       |
       v
    Endpoint
       |
       v
    Pod
       |
       v
    Application port

Verify each layer.

---

# 48. Service Unreachable — Port Mismatch

Pod listens:

    8080

Service:

    targetPort: 8081

Traffic reaches the pod IP but wrong port.

Check:

    kubectl get svc <service> -o yaml

and:

    kubectl get pod <pod> -o yaml

---

# 49. NetworkPolicy Incident

A NetworkPolicy may block:

    orders -> payment

Symptoms:

    Connection timeout

Check:

- Namespace policies
- Pod selectors
- Ingress rules
- Egress rules
- Ports

Compare with a known working communication path.

---

# 50. Ingress Incident

Symptoms:

    Internal Service works
    External URL fails

Likely layers:

    DNS
      |
      v
    ALB
      |
      v
    Ingress
      |
      v
    Service
      |
      v
    Pod

Test from inside the cluster first.

If internal service works but external traffic fails, focus on ingress/ALB/DNS.

---

# 51. ALB Ingress Troubleshooting in EKS

Check:

    kubectl get ingress -A

Then:

    kubectl describe ingress <name> -n <namespace>

Look for:

- ALB provisioning errors
- Target registration
- Health checks
- Listener configuration
- Security Groups
- Subnets
- Annotations

---

# 52. ALB Target Health

If ALB target health is unhealthy:

    ALB
      |
      X
    Pod target

Possible causes:

- Wrong health check path
- Wrong port
- Security Group
- Pod not Ready
- Application not listening
- Network policy

The ALB can exist while all targets are unhealthy.

---

# 53. DNS Incident

External request:

    app.example.com
          |
          X
        DNS
          |
          v
        ALB

Check:

- Route53 record
- DNS resolution
- ALB hostname
- TTL/caching
- Record type

If DNS resolves correctly, move to ALB.

---

# 54. TLS Certificate Incident

Symptoms:

    HTTPS errors
    Certificate mismatch
    Expired certificate

Check:

- ACM certificate
- Ingress annotations
- Listener configuration
- Hostname
- Certificate status

TLS failures can occur before Kubernetes receives the request.

---

# 55. Deployment Incident

Check:

    kubectl rollout status deployment/<deployment> -n <namespace>

Then:

    kubectl rollout history deployment/<deployment> -n <namespace>

If a new version is failing:

    kubectl rollout undo deployment/<deployment> -n <namespace>

Rollback is a mitigation, not the root cause.

---

# 56. Deployment Stuck

Possible causes:

- Image pull failure
- Readiness failure
- Insufficient resources
- Scheduling failure
- Invalid configuration
- Pod crash

Check:

    kubectl describe deployment
    kubectl get rs
    kubectl get pods
    kubectl get events

Follow the rollout chain.

---

# 57. ReplicaSet Troubleshooting

A Deployment manages ReplicaSets.

Architecture:

    Deployment
       |
       v
    ReplicaSet
       |
       v
    Pods

If Deployment appears healthy but pods are wrong, inspect ReplicaSets.

Check:

    kubectl get rs -n <namespace>

---

# 58. Rolling Update Incident

During deployment:

    Old Pods
       +
    New Pods

If new pods fail readiness:

    Old Pods remain
    New Pods fail

Depending on strategy and availability settings, rollout may stall.

Check:

- maxUnavailable
- maxSurge
- readiness
- pod health

---

# 59. Deployment Rollback

If production impact is severe:

    kubectl rollout undo deployment/<name> -n <namespace>

Then validate:

    kubectl rollout status deployment/<name> -n <namespace>

After recovery:

    Identify why new version failed.

Do not repeatedly roll forward/back without understanding the state.

---

# 60. StatefulSet Incident

StatefulSets have:

- Stable identities
- Ordered deployment
- Persistent storage

Troubleshooting must consider:

    Pod identity
    PVC
    Volume
    Ordering
    Readiness

A StatefulSet issue may be very different from a stateless Deployment issue.

---

# 61. PVC Pending

Check:

    kubectl get pvc -n <namespace>

If:

    Pending

inspect:

    kubectl describe pvc <pvc> -n <namespace>

Possible causes:

- StorageClass issue
- No provisioner
- Capacity unavailable
- Zone mismatch
- Access mode mismatch

---

# 62. PVC Bound but Pod Cannot Mount

Check pod events:

    FailedMount
    FailedAttachVolume

Investigate:

- CSI driver
- Volume attachment
- Node availability
- Access mode
- Storage backend
- Zone placement

A Bound PVC does not guarantee successful mounting.

---

# 63. EBS CSI Troubleshooting in EKS

For EBS-backed volumes, check CSI components in the appropriate system namespace.

Potential issues:

- CSI controller
- Node plugin
- IAM permissions
- Volume attachment
- Availability Zone constraints

If a pod cannot attach a volume, inspect both Kubernetes events and CSI component logs.

---

# 64. Volume Zone Mismatch

EBS volumes are AZ-specific.

Example:

    Volume -> AZ-a

Pod scheduled:

    AZ-b

Depending on storage and scheduling configuration, the pod may become unschedulable or fail to attach the volume.

Use topology-aware storage configuration.

---

# 65. Read-Only Filesystem Incident

Application expects:

    /tmp writable

but filesystem is read-only.

Symptoms:

    Permission denied
    Read-only filesystem

Check:

- SecurityContext
- Volume mount
- Container filesystem
- Application assumptions

Do not disable security settings blindly.

---

# 66. ConfigMap Incident

If application configuration is wrong:

Check:

    kubectl get configmap -n <namespace>

Then:

    kubectl describe configmap <name> -n <namespace>

Potential problems:

- Wrong key
- Wrong value
- Missing key
- Application expects another path
- Config changed but pod not restarted where required

---

# 67. Secret Incident

Symptoms:

- Authentication failures
- TLS errors
- Database connection failures

Check:

    kubectl get secret -n <namespace>

Do not print secret values unnecessarily.

Verify:

- Secret exists
- Correct key
- Correct namespace
- Pod references it
- Secret updated
- Application received the updated value

---

# 68. Secret Rotation Incident

A secret may be updated but application continues using old credentials.

Depending on how the secret is consumed:

    Environment variable
       |
       v
    Usually requires pod restart

or:

    Mounted volume
       |
       v
    May update on disk according to Kubernetes behavior

The application may still need to reload the value.

Understand the consumption model.

---

# 69. ServiceAccount Incident

A pod may fail because its ServiceAccount lacks permissions.

Check:

    kubectl get serviceaccount -n <namespace>

Then inspect:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

Use:

    kubectl auth can-i

to test permissions.

---

# 70. RBAC Troubleshooting

Example:

    Pod
      |
      v
    Kubernetes API
      |
      X
    Forbidden

Check:

    kubectl auth can-i get pods \
      --as=system:serviceaccount:<namespace>:<serviceaccount> \
      -n <namespace>

This helps determine whether the identity has the required permission.

---

# 71. API Server Incident

If Kubernetes API access is degraded:

Symptoms:

- kubectl commands slow/fail
- Controllers stop reconciling
- Deployments stall
- Operators fail

In managed EKS, investigate the control-plane/service health using available AWS and Kubernetes diagnostics.

---

# 72. Controller Manager Symptoms

Controllers reconcile desired state.

If controller behavior is degraded:

    Desired State
       |
       X
    Reconciliation
       |
       v
    Actual State

Symptoms may include:

- Replica counts incorrect
- Services not updated
- Jobs not progressing

In managed Kubernetes, control-plane component access differs from self-managed clusters.

---

# 73. Scheduler Incident

Symptoms:

    Pods remain Pending

even though nodes appear healthy.

Investigate:

- Scheduler events
- Resource requests
- Taints
- Affinity
- Topology constraints
- Quotas

Do not assume scheduler failure until scheduling constraints are ruled out.

---

# 74. Kubernetes Control Plane vs Data Plane

Control plane:

    API Server
    Scheduler
    Controllers
    etcd

Data plane:

    Nodes
    Kubelet
    Container Runtime
    CNI
    Pods

In EKS, AWS manages the Kubernetes control plane.

During incidents, first determine whether the failure is:

    Control plane

or:

    Data plane

---

# 75. Cluster Autoscaler Incident

If pods remain Pending because capacity is insufficient:

    Pending Pods
       |
       v
    Autoscaler
       |
       v
    Node Group
       |
       v
    New Node

If no node is added, check:

- Autoscaler logs
- Node group limits
- IAM
- Scheduling constraints
- Instance availability
- Subnet capacity

---

# 76. EKS Managed Node Group Scaling

Potential constraints:

- Minimum size
- Maximum size
- Desired size
- Instance capacity
- AZ availability
- Subnet IP capacity

A node group may be allowed to scale but still fail to launch usable nodes.

---

# 77. Cluster Autoscaling vs HPA

HPA:

    Changes pod replicas.

Cluster Autoscaler:

    Changes node capacity.

Example:

    Traffic ↑
       |
       v
    HPA
       |
       v
    More Pods
       |
       v
    Nodes full
       |
       v
    Cluster Autoscaler
       |
       v
    More Nodes

Both systems may need to work together.

---

# 78. HPA Incident

If HPA does not scale:

Check:

    kubectl get hpa -n <namespace>

Then:

    kubectl describe hpa <name> -n <namespace>

Investigate:

- Metrics availability
- Target utilization
- Min/max replicas
- Resource requests
- Application behavior

---

# 79. HPA No Metrics

If HPA shows:

    unknown

or cannot calculate utilization:

Check:

    Metrics Server

and:

    kubectl top pods

If Metrics Server cannot provide metrics, HPA may not scale correctly.

---

# 80. HPA Scaling Too Aggressively

Symptoms:

    Pods rapidly increase

Possible causes:

- Incorrect target
- CPU spike
- Memory target
- Traffic pattern
- Application startup behavior

Review:

- Metrics
- Scaling policies
- Stabilization windows
- Application startup

---

# 81. HPA Scaling Too Slowly

Possible causes:

- Metrics delay
- Target too high
- Scaling policies
- Insufficient max replicas
- Cluster capacity shortage

Separate:

    HPA decision

from:

    Scheduler capacity

HPA may request more pods while the scheduler cannot place them.

---

# 82. PodDisruptionBudget Incident

A PDB protects availability during voluntary disruptions.

If too restrictive:

    Node drain
       |
       X
    Pod eviction blocked

This can interfere with:

- Node upgrades
- Cluster maintenance
- Scaling operations

Review:

    kubectl get pdb -A

---

# 83. Node Drain Incident

Command concept:

    kubectl drain <node>

If drain fails:

Check:

- PDB
- DaemonSets
- Local storage
- Unmanaged pods
- Force requirements

Do not use force flags casually in production.

---

# 84. Node Upgrade Incident

During node replacement:

    Old Node
       |
       v
    Drain
       |
       v
    Pods rescheduled
       |
       v
    New Node

Potential failures:

- PDB blocks drain
- New node lacks capacity
- Volume cannot attach
- Pod affinity blocks scheduling
- Network issue
- Image pull issue

---

# 85. DaemonSet Incident

DaemonSets run workloads on nodes.

Examples:

- Logging agent
- Monitoring agent
- CNI components

If a DaemonSet is unhealthy:

    Entire node population may lose functionality.

Check:

    kubectl get daemonset -A

---

# 86. DaemonSet Rollout Incident

A new DaemonSet version may fail on every node.

Result:

    Logging agent down
    Monitoring agent down

This is a high-impact rollout.

Use:

- Staging
- Controlled rollout
- Health validation
- Rollback plan

---

# 87. Namespace Incident

If only one namespace is affected:

Check:

- ResourceQuota
- LimitRange
- NetworkPolicy
- ServiceAccounts
- Secrets
- ConfigMaps
- Namespace labels
- Admission policies

Namespace boundaries can help isolate the problem.

---

# 88. Admission Webhook Incident

Admission webhooks can validate or mutate resources.

If webhook is unavailable:

    kubectl apply
       |
       v
    API Server
       |
       X
    Admission Webhook
       |
       v
    Resource rejected/delayed

Symptoms may affect many deployments.

Check:

- Webhook pods
- Service
- TLS
- Endpoints
- Failure policy

---

# 89. CRD / Operator Incident

Operators extend Kubernetes behavior.

If an operator fails:

    Custom Resource
       |
       v
    Operator
       X
    Reconciliation

Symptoms:

- Resource stuck
- Status not updated
- Dependent resources not created

Check operator logs and custom resource status.

---

# 90. Job Incident

Jobs may fail because:

- Image failure
- Application failure
- Secret issue
- Resource limits
- Dependency failure

Check:

    kubectl get jobs -n <namespace>

Then:

    kubectl describe job <job>

and pod logs.

---

# 91. CronJob Incident

If scheduled work did not run:

Check:

    kubectl get cronjob -n <namespace>

Then:

    kubectl describe cronjob <name> -n <namespace>

Investigate:

- Schedule
- Suspend flag
- Job history
- Concurrency policy
- Starting deadline
- Resource availability

---

# 92. Kubernetes Networking Incident Framework

Use:

    DNS
      |
      v
    Service
      |
      v
    Endpoint
      |
      v
    Pod IP
      |
      v
    Container Port
      |
      v
    Application

For external traffic:

    DNS
      |
      v
    ALB
      |
      v
    Ingress
      |
      v
    Service
      |
      v
    Pod

Test one layer at a time.

---

# 93. DNS vs Service vs Application

If:

    DNS resolves

but:

    Connection refused

DNS is probably not the primary problem.

If:

    DNS fails

do not troubleshoot application port first.

This layer-by-layer approach prevents wasted time.

---

# 94. Connection Refused vs Timeout

## Connection Refused

Usually indicates:

    Host reachable
    Port not accepting connection

Possible causes:

- Wrong port
- Application not listening
- Service targetPort wrong

## Timeout

Possible causes:

- NetworkPolicy
- Security Group
- Route
- Firewall
- Dropped packets
- Application overloaded

Use the error type as a clue.

---

# 95. Kubernetes Network Debugging

Useful tools:

    kubectl exec
    curl
    wget
    nslookup
    dig
    nc
    ss

Example:

    kubectl exec -it <pod> -n <namespace> -- sh

Then test:

    nslookup payment
    curl http://payment:8080/health
    nc -vz payment 8080

Use a diagnostic container if the application image lacks troubleshooting tools.

---

# 96. Service Endpoint Debugging

Check:

    kubectl get endpoints <service> -n <namespace>

and:

    kubectl get endpointslice -n <namespace>

If endpoints are missing:

    Check pod labels
    Check readiness
    Check Service selector

If endpoints exist but traffic fails:

    Check port/network/application.

---

# 97. Pod-to-Pod Network Incident

Test:

    Pod A
      |
      v
    Pod B IP

Then:

    Pod A
      |
      v
    Service B

If Pod B IP works but Service B fails:

    Service/DNS issue.

If Pod B IP also fails:

    Network/CNI/NetworkPolicy/application issue.

---

# 98. Kubernetes Storage Incident Framework

Use:

    Pod
      |
      v
    PVC
      |
      v
    PV
      |
      v
    StorageClass
      |
      v
    CSI Driver
      |
      v
    Cloud Storage

Check from top to bottom.

---

# 99. Kubernetes Resource Troubleshooting

Always distinguish:

    Requests
    Limits
    Actual usage
    Node allocatable

Requests influence scheduling.

Limits constrain runtime resources.

Actual usage shows behavior.

Do not use one as a substitute for another.

---

# 100. CPU Throttling Incident

A container may have enough CPU request but hit its CPU limit.

Symptoms:

- Increased latency
- Slow application
- High CPU throttling

Investigate:

- CPU usage
- CPU limits
- Application latency
- Workload patterns

A CPU limit can sometimes become a performance bottleneck.

---

# 101. Memory Pressure Incident

Memory pressure can occur at:

    Container
    Pod
    Node

Different symptoms:

    Container -> OOMKilled

    Node -> MemoryPressure / eviction

Determine which level is failing.

---

# 102. Node Resource Fragmentation

A cluster may have:

    Total free CPU = 8 cores

but no single node has enough free CPU for:

    Pod request = 6 cores

The pod can remain Pending.

This is resource fragmentation.

Cluster-wide totals are not enough; scheduling is node-specific.

---

# 103. Topology Spread Incident

Topology constraints may require pods across:

    Zones
    Nodes
    Hostnames

If the cluster lacks capacity in one topology domain:

    Scheduling may fail.

Check:

    topologySpreadConstraints

and available nodes.

---

# 104. Priority and Preemption

High-priority pods may preempt lower-priority workloads.

If an important workload is Pending:

Check:

- PriorityClass
- Preemption events
- Node capacity

Understand whether the scheduler can preempt lower-priority pods.

---

# 105. Kubernetes Security Incident

Potential causes:

- RBAC change
- NetworkPolicy
- Pod Security restrictions
- Secret issue
- ServiceAccount permissions
- Admission policy

Security configuration can cause what appears to be an availability incident.

---

# 106. Pod Security / SecurityContext Incident

Possible failures:

- Running as non-root
- Read-only filesystem
- Dropping capabilities
- File permission mismatch
- User ID mismatch

Example:

    Application expects /app writable

SecurityContext:

    readOnlyRootFilesystem=true

Result:

    Application startup failure

The secure setting may be correct; the application may need to be adapted.

---

# 107. Kubernetes API Authentication Incident

If automation receives:

    Forbidden

check:

- ServiceAccount
- Role
- RoleBinding
- Namespace
- Resource
- Verb

Do not grant cluster-admin simply to make the error disappear.

---

# 108. Kubernetes API Rate Limiting

Controllers and clients may make excessive API calls.

Symptoms:

- API errors
- Slow controllers
- Reconciliation delays

Investigate:

- Client request rate
- Controller behavior
- API server metrics where available
- Recent configuration changes

---

# 109. EKS IAM Incident

Applications may use AWS IAM through mechanisms such as pod/service identity integrations depending on the platform configuration.

If AWS API calls fail:

    Pod
      |
      v
    AWS identity
      |
      X
    AWS API

Check:

- Identity configuration
- Policy
- Trust relationship
- Region
- Credentials
- Endpoint/network access

---

# 110. AWS Dependency Incident

Example:

    Application
       |
       v
    S3
       X
    Failure

Kubernetes may be completely healthy.

Do not blame Kubernetes when an external dependency is failing.

Use:

    Application logs
    Traces
    AWS service health
    Network checks

to identify the true dependency.

---

# 111. Database Dependency Incident

Example:

    Pod = healthy
    Service = healthy
    Database = unavailable

Application may return:

    500
    Timeout
    Connection refused

Kubernetes symptom may be:

    Restart loop

but root cause:

    Database

Always inspect dependency health.

---

# 112. Redis / Cache Dependency Incident

If cache is unavailable:

- Latency may increase
- Database load may increase
- Application errors may increase

A dependency failure can create a cascading Kubernetes incident.

Monitor:

    Application
       |
       v
    Cache
       |
       v
    Database

---

# 113. Cascading Failure

Example:

    Payment latency ↑
       |
       v
    Application retries ↑
       |
       v
    Pod CPU ↑
       |
       v
    HPA scales
       |
       v
    Nodes fill
       |
       v
    Pods Pending
       |
       v
    Request failures ↑

This is why Kubernetes incidents must be analyzed as systems, not isolated pods.

---

# 114. Retry Storm

Retries can amplify an outage.

Example:

    Dependency fails
       |
       v
    Request retries
       |
       v
    Dependency receives more requests
       |
       v
    Dependency becomes more overloaded
       |
       v
    More retries

Use controlled retry policies and backoff.

---

# 115. Kubernetes Incident Timeline

Create a timeline:

    10:00 Deployment
    10:02 Error rate increases
    10:03 Pod restarts
    10:04 HPA scales
    10:05 Nodes become pressured
    10:06 Pods Pending
    10:07 Availability drops

Then ask:

> What was the first abnormal event?

The first abnormal event is often closer to the root cause.

---

# 116. Recent Change Analysis

Always check:

- Deployment
- ConfigMap
- Secret
- Helm release
- ArgoCD sync
- Terraform change
- Node upgrade
- NetworkPolicy
- Ingress change
- Scaling event

Most production incidents have a change, dependency failure, capacity issue, or latent bug behind them.

---

# 117. Kubernetes Incident Severity

Classify impact.

Example:

## Critical

    Entire production unavailable

## High

    Major service unavailable

## Medium

    Partial functionality affected

## Low

    Limited/non-critical impact

Severity determines response and communication.

---

# 118. Incident Mitigation

Possible safe mitigations:

- Roll back deployment
- Scale healthy replicas
- Restore capacity
- Remove unhealthy workload
- Correct configuration
- Restore network path
- Fail over dependency

Choose the smallest change that restores service.

---

# 119. Rollback as First Mitigation

If incident clearly began after deployment:

    New version
       |
       X
    Failure

Rollback may be appropriate.

But after recovery:

    Compare versions
    Inspect code/config
    Reproduce
    Fix
    Test

Rollback restores service; it does not explain the incident.

---

# 120. Scaling as Mitigation

If traffic unexpectedly increases:

    Pods CPU ↑
       |
       v
    HPA
       |
       v
    More Pods

If cluster has capacity:

    Scale successfully.

If not:

    Cluster Autoscaler / node capacity

must be considered.

Scaling is useful only when the bottleneck is actually capacity.

---

# 121. Incident Validation

After mitigation verify:

    Pods healthy
    Ready replicas correct
    Services have endpoints
    ALB targets healthy
    Error rate normal
    Latency normal
    Logs flowing
    Metrics normal
    Traces healthy
    Dependencies healthy

Do not declare recovery based only on:

    kubectl get pods

---

# 122. Post-Incident Monitoring

After recovery watch:

    5 minutes
    15 minutes
    30 minutes
    Longer according to incident severity

Look for:

- Error recurrence
- Memory growth
- Queue growth
- Restarts
- Latency
- Scaling behavior

A system can appear recovered and fail again shortly afterward.

---

# 123. Kubernetes Incident RCA

RCA should include:

    Incident
    Impact
    Detection
    Timeline
    Root Cause
    Contributing Factors
    Mitigation
    Recovery
    Corrective Actions
    Preventive Actions

Example:

    Root cause:
    Incorrect readiness probe after deployment.

    Impact:
    New pods were removed from Service endpoints.

    Mitigation:
    Rolled back deployment.

    Prevention:
    Add readiness validation to deployment tests.

---

# 124. Five Whys — CrashLoopBackOff

Why did the pod restart?

    Application exited.

Why?

    Database connection failed.

Why?

    Database endpoint configuration was incorrect.

Why?

    ConfigMap changed during deployment.

Why?

    Configuration was not validated in staging.

Prevention:

    Configuration validation before production rollout.

---

# 125. Five Whys — Pending Pods

Why are pods Pending?

    No suitable nodes.

Why?

    Nodes lack available memory.

Why?

    Workload memory requests are high.

Why?

    Requests were increased during a deployment.

Why?

    Capacity impact was not reviewed.

Prevention:

    Resource review + capacity testing.

---

# 126. Five Whys — Service Unavailable

Why is the service unavailable?

    Service has no endpoints.

Why?

    Pods are not Ready.

Why?

    Readiness probe fails.

Why?

    Probe points to wrong port.

Why?

    Application port changed without updating deployment configuration.

Prevention:

    Deployment validation and automated smoke tests.

---

# 127. Five Whys — Node NotReady

Why is node NotReady?

    Kubelet cannot maintain healthy communication.

Why?

    Node has severe disk pressure.

Why?

    Container logs consumed disk.

Why?

    Logging volume increased significantly.

Why?

    Application entered an exception loop.

Prevention:

    Application error controls + node disk alerts + log volume monitoring.

---

# 128. Kubernetes Observability During Incidents

Use all three:

    Metrics
       |
       v
    Logs
       |
       v
    Traces

Metrics:

    What is abnormal?

Logs:

    What happened?

Traces:

    Where did the request spend time/fail?

Together they reduce troubleshooting time.

---

# 129. Kubernetes Metrics During Incidents

Monitor:

- Pod CPU
- Pod memory
- Node CPU
- Node memory
- Restarts
- Network traffic
- HPA replicas
- Request rate
- Error rate
- Latency

Use Prometheus/Grafana where available.

---

# 130. Kubernetes Logs During Incidents

Check:

    Application logs
    Previous container logs
    Kubernetes events
    Collector logs
    Ingress/controller logs
    CNI logs
    CoreDNS logs

Logs provide detailed failure context.

---

# 131. Kubernetes Traces During Incidents

If tracing is available:

    User request
       |
       v
    API
       |
       v
    Orders
       |
       v
    Payment
       |
       v
    Database

A trace can reveal the slow or failing dependency quickly.

---

# 132. Incident Correlation Example

Suppose:

    Request latency ↑
    Error rate ↑
    Pod restarts ↑

Metrics show:

    Memory ↑

Logs show:

    OutOfMemoryError

Kubernetes shows:

    OOMKilled

Trace shows:

    Large request path

Combined evidence:

    Memory-related application failure.

This is stronger than relying on one signal.

---

# 133. Production Scenario — Entire Application Down

Start:

    DNS
      |
      v
    ALB
      |
      v
    Ingress
      |
      v
    Service
      |
      v
    Endpoints
      |
      v
    Pods
      |
      v
    Application
      |
      v
    Dependencies

Check from outside to inside.

---

# 134. Production Scenario — 502 From ALB

Possible layers:

    ALB
      |
      v
    Target
      |
      v
    Service
      |
      v
    Pod
      |
      v
    Application

Check:

- ALB target health
- Ingress
- Service endpoints
- Target port
- Application listener
- Readiness

Do not assume ALB is the root cause because it returns 502.

---

# 135. Production Scenario — 503 From Ingress

Possible causes:

- No healthy backend
- No service endpoints
- Pod not Ready
- Service selector mismatch
- Ingress configuration

Check:

    Ingress
       |
       v
    Service
       |
       v
    Endpoints
       |
       v
    Ready Pods

---

# 136. Production Scenario — Pods Healthy but Requests Fail

If:

    Pods = Running
    Ready = True

but traffic fails:

Check:

- Service
- Endpoints
- TargetPort
- NetworkPolicy
- DNS
- Ingress
- ALB
- Application listener

Pod health alone is not sufficient.

---

# 137. Production Scenario — One AZ Has Problems

If workloads in one availability zone fail:

Check:

- Nodes in that AZ
- Subnets
- EBS volumes
- ALB targets
- CNI/IP capacity
- AWS infrastructure
- Pod topology

A Kubernetes cluster can be healthy overall while one AZ is degraded.

---

# 138. Production Scenario — All Pods Restart After Node Event

Check:

    Node
       |
       v
    Node event
       |
       v
    Pod eviction/restart
       |
       v
    Rescheduling

Investigate:

- Node pressure
- Node termination
- Upgrade
- Runtime failure
- Network event

---

# 139. Production Scenario — Pods Pending After Scale-Out

Timeline:

    HPA ↑
       |
       v
    Pods Pending
       |
       v
    Autoscaler
       |
       v
    Nodes not added

Investigate:

- Autoscaler
- Node group max
- Subnet IPs
- Instance capacity
- Scheduling constraints

---

# 140. Production Scenario — New Nodes Join but Pods Still Pending

Possible causes:

- Node labels missing
- Taints
- Affinity
- Resource requests too large
- Topology constraints
- Pod capacity
- IP exhaustion

More nodes do not automatically solve every scheduling problem.

---

# 141. Production Scenario — Application Works After Manual Restart

This is a warning, not a resolution.

Possible causes:

- Memory leak
- Dependency recovery
- Stale connection
- Initialization race
- Resource exhaustion
- Bad readiness behavior

Record the evidence before restarting.

Then investigate why restart restored service.

---

# 142. Production Scenario — Pod Works Locally but Not Through Service

Test:

    Pod localhost
       |
       v
    Pod IP
       |
       v
    Service DNS
       |
       v
    Ingress

If localhost works but Service fails:

    Network/service layer

If Pod IP works but Service fails:

    Service/DNS

If Service works but ingress fails:

    Ingress/ALB/DNS

---

# 143. Production Scenario — New Pods Receive No Traffic

Check:

    Pod Ready?
       |
       v
    Service endpoint?
       |
       v
    Target registered?
       |
       v
    Target healthy?
       |
       v
    Ingress routing?

A pod can be running but excluded from traffic because readiness failed.

---

# 144. Production Scenario — Logs Stop but Application Works

This is an observability incident.

Check:

    Application logs
       |
       v
    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch
       |
       v
    Kibana

Do not restart application pods if application functionality is normal and only logging is broken.

---

# 145. Production Scenario — Metrics Stop but Application Works

Check:

    Prometheus
       |
       v
    Service discovery
       |
       v
    Target
       |
       v
    /metrics

Possible causes:

- Target unavailable
- ServiceMonitor issue
- Network
- Prometheus overload
- Scrape configuration

Observability failures should be separated from application failures.

---

# 146. Production Scenario — Traces Stop but Logs Work

Check:

    Instrumentation
       |
       v
    Exporter
       |
       v
    OTel Collector
       |
       v
    Trace Backend

Logs being healthy does not prove tracing is healthy.

---

# 147. Kubernetes Incident Runbook

A good runbook should contain:

    Symptoms
    Impact
    Initial checks
    Commands
    Decision tree
    Mitigation
    Validation
    Escalation
    Rollback
    RCA requirements

Avoid runbooks containing only commands without decision logic.

---

# 148. Incident Communication

During a major production incident communicate:

    What is affected?
    When did it start?
    Current impact?
    What is being investigated?
    What mitigation is underway?
    Has service recovered?

Avoid speculative root-cause claims before evidence exists.

---

# 149. Incident Escalation

Escalate when:

- Impact is increasing
- Control-plane issue suspected
- Security incident suspected
- Data loss possible
- Dependency requires another team
- Recovery exceeds expected window

Provide evidence when escalating.

Example:

    Production orders unavailable.
    Started at 10:04 UTC.
    Pods are Ready.
    Service has endpoints.
    ALB targets are unhealthy.
    Recent deployment at 10:02 UTC.

This is far more useful than:

    "Kubernetes is broken."

---

# 150. Avoiding the Wrong Fix

Bad:

    Pod failing
       |
       v
    Delete pod

If Deployment recreates it:

    Same problem returns.

Better:

    Pod failing
       |
       v
    Identify reason
       |
       v
    Fix source
       |
       v
    Validate

Restart is a tool, not a troubleshooting methodology.

---

# 151. Production Safety Rules

Do not casually:

- Delete production pods
- Delete PVCs
- Delete namespaces
- Delete deployments
- Change security groups
- Grant cluster-admin
- Disable NetworkPolicies
- Disable TLS
- Force node drains
- Delete logs
- Modify production data

Use controlled changes and approvals appropriate to the environment.

---

# 152. Kubernetes Incident Recovery Validation

After recovery:

    [ ] Error rate normal
    [ ] Latency normal
    [ ] Pods Ready
    [ ] Replica count correct
    [ ] Restarts stable
    [ ] Nodes healthy
    [ ] Services have endpoints
    [ ] ALB targets healthy
    [ ] Logs flowing
    [ ] Metrics flowing
    [ ] Traces flowing
    [ ] Dependencies healthy

---

# 153. Kubernetes Incident Postmortem

Include:

## Summary

What happened?

## Impact

Who/what was affected?

## Timeline

What happened in order?

## Root Cause

What caused it?

## Contributing Factors

What made it worse?

## Detection

How was it discovered?

## Mitigation

What restored service?

## Corrective Actions

What must be fixed?

## Preventive Actions

What monitoring/process changes are needed?

---

# 154. Production Incident — Full Example

Scenario:

    Orders API returning 503.

Investigation:

    ALB = healthy
       |
       v
    Ingress = healthy
       |
       v
    Service = exists
       |
       v
    Endpoints = empty
       |
       v
    Pods = Running
       |
       v
    Ready = False
       |
       v
    Readiness probe = failing
       |
       v
    Probe endpoint = /health
       |
       v
    Application health endpoint moved to /ready

Root cause:

    Readiness configuration mismatch after deployment.

Mitigation:

    Rollback.

Prevention:

    Automated readiness smoke test.

---

# 155. Production Incident — Full Example

Scenario:

    Payments pods repeatedly restart.

Investigation:

    CrashLoopBackOff
       |
       v
    Previous logs
       |
       v
    Database timeout
       |
       v
    Database connection count maxed
       |
       v
    New application version increased connection usage

Root cause:

    Connection pool configuration.

Kubernetes was only showing the symptom.

---

# 156. Production Incident — Full Example

Scenario:

    Many pods Pending.

Investigation:

    Events:
    Insufficient memory

    Nodes:
    High memory requests

    HPA:
    Replicas increased

    Cluster Autoscaler:
    At max node group size

Root cause:

    Workload demand exceeded configured cluster capacity.

Mitigation:

    Increase capacity within approved limits.

Prevention:

    Capacity planning + autoscaler limits review.

---

# 157. Production Incident — Full Example

Scenario:

    Entire logging platform stops receiving Kubernetes logs.

Investigation:

    Application stdout = healthy
    Collector DaemonSet = unhealthy
    New collector rollout = failed
    Config error = invalid parser

Root cause:

    Collector configuration deployment.

Mitigation:

    Roll back collector DaemonSet.

Prevention:

    Configuration validation + staged rollout.

---

# 158. Production Incident — Full Example

Scenario:

    Pods cannot communicate across namespaces.

Investigation:

    DNS = works
    Pod IP connectivity = blocked
    NetworkPolicy = newly deployed
    Policy = missing allowed ingress

Root cause:

    NetworkPolicy configuration.

Mitigation:

    Correct policy.

Prevention:

    Network policy integration tests.

---

# 159. Senior DevOps Kubernetes Incident Framework

Use five layers:

    1. APPLICATION
    2. POD
    3. NODE
    4. CLUSTER
    5. CLOUD / DEPENDENCY

Example:

    Application error
       |
       v
    Pod restart
       |
       v
    Node pressure
       |
       v
    Cluster capacity
       |
       v
    AWS infrastructure

Always identify the lowest/root layer supported by evidence.

---

# 160. Kubernetes Incident Interview Framework

When asked:

> How would you troubleshoot this production issue?

Answer in this order:

    1. Confirm impact
    2. Define scope
    3. Check recent changes
    4. Check application symptoms
    5. Check pod state
    6. Check events
    7. Check nodes/resources
    8. Check networking
    9. Check dependencies
    10. Mitigate
    11. Validate
    12. RCA

This demonstrates production thinking rather than memorized commands.

---

# 161. Interview Question — Pod Is CrashLoopBackOff. What Do You Do?

Strong answer:

> I first check the current and previous container logs because the previous container often contains the actual startup failure. Then I use `kubectl describe pod` to inspect events, exit codes, probe failures and OOMKilled status. I verify configuration, secrets, dependencies, commands and resource limits. I identify the application-level root cause before restarting or changing the pod.

---

# 162. Interview Question — Pod Is Pending. What Do You Check?

Strong answer:

> I check `kubectl describe pod` and inspect scheduler events. I look for insufficient CPU or memory, taints and tolerations, node selectors, affinity, topology constraints, quotas and storage requirements. Then I compare the pod's resource requests with node allocatable capacity and determine whether the issue is scheduling configuration or actual cluster capacity.

---

# 163. Interview Question — Pods Are Running but Application Is Not Reachable. What Do You Check?

Strong answer:

> I verify readiness first, then check the Service and its endpoints. After that I validate the target port and application listener, test Service DNS and connectivity from inside the cluster, and finally inspect NetworkPolicies, Ingress and ALB target health if external traffic is involved.

---

# 164. Interview Question — How Do You Troubleshoot an EKS Networking Problem?

Strong answer:

> I first determine whether DNS, Service routing, pod-to-pod networking or external connectivity is failing. Then I check CoreDNS, Services and EndpointSlices, NetworkPolicies, the AWS VPC CNI, subnet IP availability, security groups and routing. I test connectivity layer by layer rather than changing network settings blindly.

---

# 165. Interview Question — How Do You Troubleshoot OOMKilled?

Strong answer:

> I confirm the container was OOMKilled, then compare memory requests and limits with actual usage. I investigate application memory behavior, JVM or runtime configuration, traffic changes and recent releases. I determine whether the problem is a container limit or node memory pressure before choosing mitigation.

---

# 166. Interview Question — What Is the Difference Between Readiness and Liveness?

Strong answer:

> Readiness determines whether the pod should receive traffic. Liveness determines whether Kubernetes should restart the container. A readiness failure should normally remove the pod from traffic without restarting it, while a liveness failure triggers a restart. I avoid putting dependency failures into liveness unless restarting the application is actually the desired recovery behavior.

---

# 167. Interview Question — How Do You Troubleshoot 502 From ALB?

Strong answer:

> I start with ALB target health, then inspect the Kubernetes Ingress, Service endpoints, target port, pod readiness and application listener. If the application works directly through the Service but ALB returns 502, I focus on the ingress/ALB target path rather than the application itself.

---

# 168. Interview Question — How Do You Troubleshoot Pods Pending After HPA Scaling?

Strong answer:

> I check HPA desired replicas and then inspect the Pending pods for scheduler events. If capacity is insufficient, I check Cluster Autoscaler and node group limits. I also verify that scheduling constraints, taints, affinity, topology and subnet IP capacity are not preventing new pods or nodes from being placed.

---

# 169. Interview Question — How Do You Troubleshoot a Node NotReady?

Strong answer:

> I inspect node conditions and events first. I check kubelet and container runtime health where node access is available, then investigate disk, memory, PID pressure, networking and CNI health. In EKS I also consider node group state, AWS networking and infrastructure events. I determine whether workloads need to be drained or rescheduled based on the impact.

---

# 170. Interview Question — How Do You Handle a Production Kubernetes Incident?

Strong answer:

> I first establish impact and scope, then identify the first abnormal signal and check recent changes. I trace the problem through application, pod, node, cluster and external dependencies. I apply the smallest safe mitigation that restores service, validate the full request path and observability, then perform RCA with corrective and preventive actions.

---

# 171. Final Kubernetes Incident Decision Tree

                    INCIDENT
                       |
                       v
                 DEFINE IMPACT
                       |
                       v
                  CHECK PODS
                       |
          +------------+------------+
          |                         |
       Pods fail                 Pods healthy
          |                         |
          v                         v
      Events                    Service?
          |                         |
          v                         v
    App/Resource               Endpoints?
          |                         |
          v                         v
      Node?                    Network?
          |                         |
          v                         v
     Cluster?                   Ingress?
          |                         |
          +------------+------------+
                       |
                       v
                 DEPENDENCIES
                       |
                       v
                    MITIGATE
                       |
                       v
                    VALIDATE
                       |
                       v
                      RCA

---

# 172. Final Kubernetes Troubleshooting Matrix

| Symptom | First Check | Common Root Cause |
|---|---|---|
| Pending | `describe pod` | Scheduling/resource issue |
| CrashLoopBackOff | `logs --previous` | Application/config/dependency |
| ImagePullBackOff | Pod events | Image/tag/auth |
| OOMKilled | Container state | Memory limit/application |
| Evicted | Node conditions | Resource pressure |
| NotReady node | Node conditions/events | Kubelet/runtime/network |
| No Service traffic | Endpoints | Selector/readiness |
| 502 ALB | Target health | Port/health check/backend |
| 503 Ingress | Endpoints/readiness | No healthy backend |
| DNS failure | CoreDNS | DNS/service issue |
| PVC Pending | PVC events | StorageClass/provisioner |
| FailedMount | Pod events | CSI/volume/permission |
| Pods cannot communicate | NetworkPolicy/CNI | Network restriction |
| HPA not scaling | HPA status/metrics | Metrics/config/capacity |
| Pods Pending after HPA | Scheduler/autoscaler | Node capacity |
| Logs missing | Collector pipeline | Observability failure |
| Traces missing | Instrumentation/export | Tracing failure |

---

# 173. Final Production Kubernetes Incident Checklist

## Before Mitigation

    [ ] Confirm impact
    [ ] Define scope
    [ ] Capture timeline
    [ ] Check recent changes
    [ ] Preserve logs/events
    [ ] Identify affected workloads

## During Investigation

    [ ] Pod state
    [ ] Container logs
    [ ] Previous logs
    [ ] Events
    [ ] Resources
    [ ] Nodes
    [ ] Services
    [ ] Endpoints
    [ ] Networking
    [ ] Storage
    [ ] Ingress
    [ ] Dependencies
    [ ] Observability

## During Mitigation

    [ ] Choose smallest safe change
    [ ] Record action
    [ ] Validate immediately
    [ ] Monitor recovery

## After Recovery

    [ ] Verify application
    [ ] Verify traffic
    [ ] Verify pods
    [ ] Verify nodes
    [ ] Verify logs
    [ ] Verify metrics
    [ ] Verify traces
    [ ] Perform RCA
    [ ] Create preventive actions

---

# 174. Final Production Principles

Remember:

> Kubernetes status is a symptom, not always the root cause.

> CrashLoopBackOff tells you the container keeps failing; application logs tell you why.

> Pending means the workload has not found a valid placement; scheduler events explain why.

> Running does not mean Ready.

> Ready does not automatically mean the application is externally reachable.

> A Service with no endpoints cannot route traffic.

> A healthy pod can still depend on an unhealthy database.

> HPA creates pods; it does not create node capacity by itself.

> Cluster Autoscaler adds capacity, but scheduling constraints can still block workloads.

> Node pressure can cause application symptoms far away from the original resource problem.

> Restarting a pod is mitigation, not root-cause analysis.

> Always investigate recent deployments, configuration changes and infrastructure changes.

> Use metrics, logs and traces together.

> In EKS, distinguish Kubernetes-level failures from AWS networking, storage, IAM and dependency failures.

> Preserve evidence before destructive actions.

> Restore service first when necessary, then perform a complete RCA.

The production Kubernetes incident mindset is:

    OBSERVE
       +
    SCOPE
       +
    ISOLATE
       +
    MITIGATE
       +
    VALIDATE
       +
    RCA
       +
    PREVENT

That is the troubleshooting approach expected from a production DevOps / DevSecOps engineer.
