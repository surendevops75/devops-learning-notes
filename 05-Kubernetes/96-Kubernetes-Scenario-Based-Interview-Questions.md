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

