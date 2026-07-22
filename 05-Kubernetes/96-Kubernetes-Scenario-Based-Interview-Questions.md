# Kubernetes Scenario-Based Interview Questions

## Introduction

This document contains production-oriented Kubernetes scenario-based interview questions commonly asked for DevOps, Platform Engineer, Cloud Engineer, and Site Reliability Engineer (SRE) roles.

These scenarios are based on real production incidents rather than theoretical concepts.

For every scenario, always follow a structured troubleshooting methodology:

- Understand the problem
- Verify the symptoms
- Collect logs and events
- Identify the root cause
- Implement the fix
- Prevent recurrence

Always troubleshoot using evidence instead of assumptions.

---

# Section 1 - Pod Troubleshooting Scenarios

---

## Scenario 1

### Interview Question

A production application suddenly becomes unavailable because every Pod enters **CrashLoopBackOff**. How would you troubleshoot it?

### Production-Level Answer

CrashLoopBackOff means the container starts successfully but repeatedly crashes after execution.

The Kubernetes cluster is functioning correctly—the application inside the container is failing.

Never restart Pods immediately.

Identify why the application is exiting.

### Approach

Check Pod status.

```bash
kubectl get pods
```

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Review current logs.

```bash
kubectl logs <pod-name>
```

Review previous logs.

```bash
kubectl logs <pod-name> --previous
```

Check Events.

Look for:

- Failed mounts
- Probe failures
- OOMKilled
- Configuration errors

Verify

- Environment Variables
- ConfigMaps
- Secrets
- Database connectivity
- External APIs

### Common Root Causes

- Application exception
- Missing ConfigMap
- Missing Secret
- Wrong environment variable
- Database unavailable
- Port already in use
- Invalid startup command
- OOMKilled

### Best Practices

- Always check previous logs
- Configure proper health probes
- Validate configurations before deployment
- Implement startup validation
- Monitor restart counts

---

## Scenario 2

### Interview Question

Pods remain in **Pending** state after deployment. How would you investigate?

### Production-Level Answer

Pending means Kubernetes has accepted the Pod but cannot schedule it onto any Worker Node.

This is usually a scheduling issue rather than an application issue.

### Approach

Check Pod status.

```bash
kubectl get pods
```

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Review Events.

Check

- Node availability
- CPU requests
- Memory requests
- Taints
- Tolerations
- Node Selectors
- Affinity Rules
- Persistent Volume Claims

Check nodes.

```bash
kubectl get nodes
```

### Common Root Causes

- Insufficient CPU
- Insufficient Memory
- Node NotReady
- Taints
- Node Selector mismatch
- PVC Pending
- Resource Quotas exceeded

### Best Practices

- Right-size resource requests
- Monitor cluster capacity
- Enable Cluster Autoscaler
- Avoid unnecessary node selectors

---

## Scenario 3

### Interview Question

Your Pods enter **ImagePullBackOff** after deployment. How would you troubleshoot it?

### Production-Level Answer

ImagePullBackOff indicates Kubernetes cannot download the container image from the registry.

The problem usually exists between Kubernetes and the container registry.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Check Events.

Verify

- Image name
- Image tag
- Registry
- Authentication
- ImagePullSecrets

If using ECR verify

- IAM permissions
- IRSA
- Repository existence

Attempt pulling the image manually.

### Common Root Causes

- Wrong image tag
- Image not pushed
- Repository deleted
- Authentication failure
- ImagePullSecret missing
- Registry unavailable

### Best Practices

- Use immutable image tags
- Validate images before deployment
- Avoid latest tag
- Automate registry authentication

---

## Scenario 4

### Interview Question

Pods show **CreateContainerConfigError**. What would you check?

### Production-Level Answer

CreateContainerConfigError indicates Kubernetes cannot create the container because required configuration is missing or invalid.

The container never starts.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Verify

- ConfigMaps
- Secrets
- Environment Variables
- Volume mounts
- Container configuration

Review Events.

### Common Root Causes

- ConfigMap missing
- Secret missing
- Invalid environment variable
- Wrong volume mount
- Incorrect configuration

### Best Practices

- Validate configuration before deployment
- Use GitOps
- Store configurations in version control
- Verify manifests during CI

---

## Scenario 5

### Interview Question

Pods show **OOMKilled** after deployment. How would you troubleshoot it?

### Production-Level Answer

OOMKilled means the container exceeded its configured memory limit and Kubernetes terminated it.

The application is consuming more memory than allocated.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Check resource usage.

```bash
kubectl top pod
```

Review logs.

Check

- Memory limits
- Memory requests
- Application heap
- Memory leaks

### Common Root Causes

- Memory leak
- Low memory limit
- Large data processing
- JVM heap too large
- Infinite cache growth

### Best Practices

- Configure realistic limits
- Monitor memory usage
- Tune JVM
- Perform load testing

---

## Scenario 6

### Interview Question

A Pod is running, but the application is not receiving traffic. How would you investigate?

### Production-Level Answer

A Running Pod does not guarantee the application is healthy. Verify whether the Pod is Ready and whether it is registered behind the Service.

### Approach

Check Pod readiness.

```bash
kubectl get pods
```

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Verify

- Readiness Probe
- Service
- Endpoints
- Application logs
- Listening port

### Common Root Causes

- Readiness Probe failure
- Application startup incomplete
- Wrong container port
- Service selector mismatch

### Best Practices

- Configure readiness probes
- Test endpoints after deployment
- Monitor Pod health continuously

---

## Scenario 7

### Interview Question

A Pod continuously restarts even though there are no application errors. What would you investigate?

### Production-Level Answer

If the application logs show no errors, investigate Kubernetes health checks, resource limits, and infrastructure events before blaming the application.

### Approach

Check:

- Restart count
- Liveness Probe
- Readiness Probe
- Startup Probe
- Events
- Node status
- Resource utilization

Review kubelet events.

### Common Root Causes

- Liveness Probe failure
- OOMKilled
- Node reboot
- Disk pressure
- Incorrect probe configuration

### Best Practices

- Configure Startup Probes
- Tune health checks
- Monitor restart counts
- Validate probe timings

---

## Scenario 8

### Interview Question

One Pod in a Deployment is healthy, while another identical Pod continuously fails. How would you troubleshoot it?

### Production-Level Answer

Since the Deployment configuration is identical, compare the unhealthy Pod with a healthy one to identify infrastructure or runtime differences.

### Approach

Compare:

- Node placement
- Events
- Logs
- Environment variables
- Mounted volumes
- Resource usage
- Restart history

Identify what differs between the two Pods.

### Common Root Causes

- Node issue
- Corrupted volume
- Resource exhaustion
- Application race condition
- Configuration drift

### Best Practices

- Compare healthy vs unhealthy Pods
- Enable centralized logging
- Monitor node health
- Use anti-affinity when appropriate

---

## Scenario 9

### Interview Question

A Pod is stuck in the **Terminating** state for several minutes. How would you investigate?

### Production-Level Answer

A Pod remaining in the Terminating state usually indicates that Kubernetes cannot complete the graceful shutdown process or remove attached resources.

### Approach

Check:

```bash
kubectl describe pod <pod-name>
```

Verify:

- Finalizers
- Mounted volumes
- PreStop hooks
- Active connections
- Node status
- Container runtime

Review Pod events before forcing deletion.

### Common Root Causes

- Finalizer blocking deletion
- Stuck PreStop hook
- CSI volume detach issue
- Node unavailable
- Container runtime problem

### Best Practices

- Use graceful shutdown
- Configure reasonable terminationGracePeriodSeconds
- Avoid force deletion unless necessary
- Monitor storage detach operations

---

## Scenario 10

### Interview Question

Your application starts successfully but fails every readiness check. How would you troubleshoot it?

### Production-Level Answer

A failing Readiness Probe means Kubernetes considers the application unavailable for serving traffic, even if the container is running.

Focus on why the readiness endpoint is failing rather than restarting the Pod.

### Approach

Check:

- Readiness Probe configuration
- Application health endpoint
- Container port
- Startup time
- Application logs
- Service endpoints

Run:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Test the readiness endpoint from inside the container if possible.

### Common Root Causes

- Incorrect readiness path
- Wrong container port
- Slow application startup
- Backend dependency unavailable
- Application initialization incomplete

### Best Practices

- Use dedicated readiness endpoints
- Configure realistic probe timings
- Separate readiness and liveness checks
- Validate probes during deployment testing

---

## Scenario 11

### Interview Question

Your Pods continuously fail the **Liveness Probe** and keep restarting. How would you troubleshoot it?

### Production-Level Answer

A failing Liveness Probe causes Kubernetes to restart the container because it assumes the application is unhealthy.

The application may actually be healthy, but an incorrectly configured probe can create an infinite restart loop.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Review Events.

Check:

- Liveness Probe path
- Container port
- Probe timeout
- Initial delay
- Failure threshold
- Application logs

Verify the health endpoint manually.

### Common Root Causes

- Wrong endpoint
- Wrong port
- Timeout too small
- Slow application
- Backend dependency failure
- Incorrect HTTP response

### Best Practices

- Keep liveness checks simple
- Don't perform database calls
- Configure realistic timeout values
- Test probes before production

---

## Scenario 12

### Interview Question

A newly deployed application never becomes Ready because the **Startup Probe** keeps failing. What would you investigate?

### Production-Level Answer

Startup Probes are designed for slow-starting applications.

If Startup Probes fail repeatedly, Kubernetes kills the container before initialization completes.

### Approach

Check

```bash
kubectl describe pod <pod-name>
```

Verify

- Startup Probe
- Application startup time
- JVM initialization
- Database connection
- Logs

Measure actual startup duration.

### Common Root Causes

- Startup timeout too small
- Heavy application initialization
- Database unavailable
- External API dependency
- JVM warm-up

### Best Practices

- Use Startup Probe for slow applications
- Increase failure threshold
- Measure startup time
- Separate startup and liveness probes

---

## Scenario 13

### Interview Question

A Pod remains in **Init:0/1** state for several minutes. How would you troubleshoot it?

### Production-Level Answer

A Pod cannot start its main container until every Init Container completes successfully.

Focus on the Init Container rather than the application container.

### Approach

Check Pod status.

```bash
kubectl get pods
```

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Review Init Container logs.

```bash
kubectl logs <pod-name> -c <init-container>
```

Verify

- Scripts
- Database connectivity
- Volume mounts
- Network
- Permissions

### Common Root Causes

- Database unavailable
- Script failure
- Wrong image
- Missing Secret
- Permission denied

### Best Practices

- Keep Init Containers lightweight
- Log every initialization step
- Avoid long-running tasks
- Validate dependencies before startup

---

## Scenario 14

### Interview Question

One container inside a **multi-container Pod** crashes while the other continues running. How would you investigate?

### Production-Level Answer

Each container has its own lifecycle.

A single failing container may impact the application even if the Pod remains Running.

Investigate the failing container independently.

### Approach

List containers.

```bash
kubectl get pod <pod-name> -o yaml
```

View logs.

```bash
kubectl logs <pod-name> -c <container-name>
```

Compare

- Environment variables
- Mounted volumes
- Resources
- Commands

### Common Root Causes

- Wrong configuration
- Missing dependency
- Resource limit exceeded
- Application exception

### Best Practices

- Monitor every container separately
- Allocate resources individually
- Separate application and helper containers

---

## Scenario 15

### Interview Question

Your Sidecar container is healthy, but the main application container keeps failing. How would you troubleshoot it?

### Production-Level Answer

A healthy Sidecar only confirms that supporting services are functioning.

The main application container must be investigated independently.

### Approach

Check

- Application logs
- Sidecar logs
- Shared volumes
- Shared configuration
- Resource limits

Verify communication between containers.

### Common Root Causes

- Shared volume issue
- Configuration mismatch
- Application bug
- Sidecar dependency unavailable

### Best Practices

- Monitor containers separately
- Avoid unnecessary container dependencies
- Share only required resources

---

## Scenario 16

### Interview Question

How would you debug a production Pod without modifying the running container?

### Production-Level Answer

Use Ephemeral Containers for production debugging.

They allow temporary debugging without rebuilding or restarting the application Pod.

### Approach

Launch an Ephemeral Container.

```bash
kubectl debug -it <pod-name> --image=busybox
```

Investigate

- Network
- DNS
- File system
- Running processes
- Mounted volumes

Exit after debugging.

### Common Root Causes Found

- DNS issue
- Missing files
- Network connectivity
- Configuration problems
- Storage issues

### Best Practices

- Use Ephemeral Containers only for debugging
- Avoid modifying production containers
- Remove debugging sessions after investigation

---

## Scenario 17

### Interview Question

A Pod shows **Completed** status, but users report the application is down. Why?

### Production-Level Answer

Completed means the container finished execution successfully.

If the workload is intended to run continuously, the wrong Kubernetes resource may have been used.

### Approach

Verify

- Deployment
- Job
- CronJob
- Pod manifest
- Restart Policy

Determine whether the workload should be long-running.

### Common Root Causes

- Job used instead of Deployment
- RestartPolicy Never
- Batch workload completed
- Incorrect manifest

### Best Practices

- Use Deployments for applications
- Use Jobs for batch workloads
- Review Restart Policies
- Validate manifests

---

## Scenario 18

### Interview Question

Your Pod reports **CreateContainerError** immediately after scheduling. How would you troubleshoot it?

### Production-Level Answer

CreateContainerError occurs after scheduling but before container startup.

The container runtime cannot create the container.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Check

- Image
- Commands
- Arguments
- Mounted volumes
- Container runtime logs

Review Events.

### Common Root Causes

- Invalid command
- Missing executable
- Volume mount failure
- Permission issue
- Runtime error

### Best Practices

- Validate startup commands
- Test images locally
- Review manifests carefully
- Keep container images minimal

---

## Scenario 19

### Interview Question

Several Pods are suddenly marked as **Evicted**. How would you investigate?

### Production-Level Answer

Evicted Pods indicate the Kubernetes scheduler removed workloads because the Node was under resource pressure.

Investigate the Worker Node rather than the application.

### Approach

Check

```bash
kubectl describe pod <pod-name>
```

Review

- Node conditions
- Memory Pressure
- Disk Pressure
- PID Pressure

Check Node.

```bash
kubectl describe node <node-name>
```

### Common Root Causes

- Memory exhaustion
- Disk full
- Ephemeral storage exhausted
- Node pressure
- Large log files

### Best Practices

- Monitor node resources
- Configure requests and limits
- Clean temporary files
- Enable Cluster Autoscaler

---

## Scenario 20

### Interview Question

A production Pod is failing, but its logs disappear after every restart. How would you troubleshoot it?

### Production-Level Answer

When a container restarts, the latest logs may not contain the original failure.

Always retrieve logs from the previous container instance before investigating further.

### Approach

View current logs.

```bash
kubectl logs <pod-name>
```

View previous logs.

```bash
kubectl logs <pod-name> --previous
```

Review

- Events
- Restart count
- Probe failures
- Exit code
- Application errors

Correlate logs with Kubernetes Events.

### Common Root Causes

- Application crash
- OOMKilled
- Probe failure
- Missing configuration
- Dependency unavailable

### Best Practices

- Always check `--previous` logs
- Enable centralized logging
- Retain application logs
- Correlate logs with monitoring alerts

---

# Section 2 - Scheduling Scenarios

---

## Scenario 21

### Interview Question

A Pod remains in the **Pending** state, and the event says **0/5 nodes are available: Insufficient CPU**. How would you troubleshoot it?

### Production-Level Answer

This indicates that Kubernetes cannot find any Worker Node with enough available CPU to satisfy the Pod's resource request.

The application is healthy, but the cluster lacks scheduling capacity.

### Approach

Check Pod events.

```bash
kubectl describe pod <pod-name>
```

Check node resource utilization.

```bash
kubectl top nodes
```

Review Pod resource requests.

```bash
kubectl describe pod <pod-name>
```

Verify

- CPU Requests
- CPU Limits
- Cluster capacity
- Running workloads

### Common Root Causes

- CPU requests too high
- Cluster fully utilized
- Cluster Autoscaler not working
- Resource fragmentation
- Too many workloads

### Best Practices

- Right-size CPU requests
- Enable Cluster Autoscaler
- Monitor node utilization
- Avoid over-provisioning

---

## Scenario 22

### Interview Question

A Pod cannot be scheduled because of **Insufficient Memory**. How would you investigate?

### Production-Level Answer

The scheduler cannot place the Pod because every Worker Node lacks sufficient allocatable memory.

### Approach

Check events.

```bash
kubectl describe pod <pod-name>
```

Review node memory.

```bash
kubectl top nodes
```

Verify

- Memory requests
- Memory limits
- Existing workloads
- Node capacity

### Common Root Causes

- Large memory requests
- Memory fragmentation
- Cluster fully utilized
- Memory leak in workloads
- Autoscaler failure

### Best Practices

- Monitor memory usage
- Allocate realistic requests
- Enable node auto scaling
- Avoid oversized Pods

---

## Scenario 23

### Interview Question

A Pod cannot be scheduled because of a **Node Selector** mismatch. How would you troubleshoot it?

### Production-Level Answer

A Node Selector restricts scheduling to specific Worker Nodes.

If no matching node exists, the Pod remains Pending indefinitely.

### Approach

View Pod specification.

```bash
kubectl get pod <pod-name> -o yaml
```

Check node labels.

```bash
kubectl get nodes --show-labels
```

Compare

- Required labels
- Available labels

### Common Root Causes

- Incorrect label
- Label removed
- Typographical error
- Wrong environment

### Best Practices

- Standardize node labels
- Validate selectors
- Automate node labeling
- Document scheduling rules

---

## Scenario 24

### Interview Question

Pods remain Pending because of **Taints and Tolerations**. What would you check?

### Production-Level Answer

Taints prevent Pods from being scheduled onto specific nodes unless the Pod has a matching Toleration.

### Approach

Check node taints.

```bash
kubectl describe node <node-name>
```

Review Pod tolerations.

```bash
kubectl get pod <pod-name> -o yaml
```

Compare

- Taints
- Tolerations

### Common Root Causes

- Missing toleration
- Incorrect taint key
- Wrong effect
- New node tainted

### Best Practices

- Use taints carefully
- Document node roles
- Validate tolerations before deployment
- Avoid unnecessary taints

---

## Scenario 25

### Interview Question

A Deployment using **Node Affinity** suddenly stops scheduling new Pods. How would you investigate?

### Production-Level Answer

Node Affinity provides advanced scheduling rules. If no nodes satisfy the affinity conditions, Pods remain Pending.

### Approach

Review Pod manifest.

```bash
kubectl get deployment <deployment-name> -o yaml
```

Check node labels.

```bash
kubectl get nodes --show-labels
```

Verify

- Required affinity
- Preferred affinity
- Available nodes

### Common Root Causes

- Missing labels
- Label removed
- Incorrect affinity rule
- Cluster topology changed

### Best Practices

- Prefer preferredDuringScheduling where possible
- Review affinity rules
- Avoid overly restrictive scheduling

---

## Scenario 26

### Interview Question

Pods using **Pod Anti-Affinity** cannot be scheduled. Why?

### Production-Level Answer

Pod Anti-Affinity prevents similar Pods from running on the same node.

If the cluster has too few nodes, Kubernetes cannot satisfy the rule.

### Approach

Check

- Pod Anti-Affinity rules
- Available nodes
- Existing Pods

Run

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Too few nodes
- Strict anti-affinity
- Node unavailable
- Cluster capacity insufficient

### Best Practices

- Use preferred anti-affinity where appropriate
- Scale Worker Nodes
- Review scheduling constraints

---

## Scenario 27

### Interview Question

A Pod with **requiredDuringSchedulingIgnoredDuringExecution** never gets scheduled. How would you troubleshoot it?

### Production-Level Answer

This affinity rule is mandatory.

If Kubernetes cannot satisfy it, the Pod will never be scheduled.

### Approach

Review

- Affinity rules
- Node labels
- Scheduler events

Check

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Missing node labels
- Incorrect affinity rule
- Infrastructure changes
- Wrong environment

### Best Practices

- Use mandatory affinity only when necessary
- Validate labels regularly
- Monitor scheduler events

---

## Scenario 28

### Interview Question

A Pod remains Pending because of **ResourceQuota** restrictions. How would you investigate?

### Production-Level Answer

ResourceQuota limits how much CPU, memory, storage, or object count a namespace can consume.

Once the quota is exhausted, new workloads cannot be scheduled.

### Approach

Check ResourceQuota.

```bash
kubectl get resourcequota
```

Describe it.

```bash
kubectl describe resourcequota
```

Compare

- Requested resources
- Available quota

### Common Root Causes

- Namespace quota exceeded
- New deployment too large
- Multiple teams sharing namespace
- Resource requests increased

### Best Practices

- Monitor quotas
- Allocate namespace budgets
- Review requests periodically
- Alert before exhaustion

---

## Scenario 29

### Interview Question

Pods fail admission because of **LimitRange** policies. How would you troubleshoot it?

### Production-Level Answer

LimitRanges enforce default and maximum resource values inside a namespace.

Pods violating these limits are rejected before scheduling.

### Approach

Check

```bash
kubectl get limitrange
```

Describe it.

```bash
kubectl describe limitrange
```

Compare

- CPU requests
- Memory requests
- Limits

### Common Root Causes

- Missing resource requests
- Exceeding maximum limits
- Invalid configuration
- Namespace policy changes

### Best Practices

- Define requests and limits
- Review namespace policies
- Validate manifests during CI

---

## Scenario 30

### Interview Question

A Pod remains Pending even though every Worker Node appears healthy. How would you troubleshoot it?

### Production-Level Answer

When nodes appear healthy but Pods are still Pending, investigate scheduler constraints rather than infrastructure health.

The scheduler only places Pods when **all** scheduling conditions are satisfied.

### Approach

Review

- Scheduler Events
- Node Selectors
- Affinity
- Anti-Affinity
- Taints
- Tolerations
- PVC status
- ResourceQuota
- LimitRange
- Resource requests

Run

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Scheduler constraints
- PVC Pending
- Affinity conflict
- ResourceQuota exceeded
- Taints
- LimitRange rejection
- Unsatisfied node selector

### Best Practices

- Always begin with `kubectl describe pod`
- Read scheduler events before changing manifests
- Keep scheduling policies simple
- Monitor scheduler metrics
- Avoid overly restrictive scheduling rules

---

