# K9s - Kubernetes CLI Dashboard

## Introduction

Managing Kubernetes clusters using:

```bash
kubectl
```

is powerful.

---

However in large environments:

```text
Many Pods

Many Namespaces

Many Deployments

Many Services
```

---

Managing resources becomes difficult.

---

To simplify cluster management we use:

```text
K9s
```

---

# What is K9s?

K9s is a:

```text
Terminal Based Kubernetes Dashboard
```

---

It provides:

```text
Cluster Visibility

Resource Navigation

Logs Access

Pod Access

Troubleshooting
```

---

without opening a browser.

---

# Why Use K9s?

Instead of running:

```bash
kubectl get pods

kubectl get svc

kubectl get deploy

kubectl logs

kubectl describe
```

---

multiple times,

---

K9s provides:

```text
Single Interactive Interface
```

---

# K9s Architecture

```text
User
  ↓

K9s
  ↓

kubectl Context
  ↓

Kubernetes API Server
```

---

K9s uses:

```text
kubectl Configuration
```

already present on the workstation.

---

# Install K9s

Trainer Command

```bash
curl -sS https://webinstall.dev/k9s | bash
```

---

Verify Installation

```bash
k9s version
```

---

# Starting K9s

Launch

```bash
k9s
```

---

K9s automatically connects to:

```text
Current Kubernetes Context
```

---

Example

```bash
kubectl config current-context
```

---

# Main Dashboard

After opening K9s:

```text
Pods

Deployments

Services

Namespaces

Nodes

StatefulSets
```

can be viewed.

---

# Viewing Pods

Command inside K9s

```text
:pods
```

or

```text
:po
```

---

Shows

```text
Pod Name

Namespace

Status

CPU

Memory
```

---

# Viewing Deployments

Command

```text
:deploy
```

or

```text
:dp
```

---

Displays

```text
Deployments

Replicas

Status
```

---

# Viewing Services

Command

```text
:svc
```

---

Displays

```text
ClusterIP

NodePort

LoadBalancer
```

---

# Viewing StatefulSets

Command

```text
:sts
```

---

Displays

```text
StatefulSet Name

Replicas

Status
```

---

# Viewing Nodes

Command

```text
:nodes
```

or

```text
:no
```

---

Displays

```text
Node Status

CPU Usage

Memory Usage
```

---

Useful for:

```text
Cluster Monitoring
```

---

# Viewing Namespaces

Command

```text
:ns
```

---

Switch Namespace

```text
Select Namespace
```

---

Press

```text
Enter
```

---

# Viewing Logs

Select Pod

---

Press

```text
l
```

---

Shows

```text
Container Logs
```

---

Equivalent

```bash
kubectl logs pod-name
```

---

# Follow Logs

Inside Logs Screen

Press

```text
f
```

---

Equivalent

```bash
kubectl logs -f pod-name
```

---

# Describe Resources

Select Resource

---

Press

```text
d
```

---

Equivalent

```bash
kubectl describe
```

---

Useful for:

```text
Events

Errors

Scheduling Issues
```

---

# Execute Into Pod

Select Pod

---

Press

```text
s
```

---

Opens Shell

---

Equivalent

```bash
kubectl exec -it pod-name -- bash
```

---

# Delete Resource

Select Resource

---

Press

```text
Ctrl + D
```

---

Be careful in:

```text
Production
```

---

# Resource Filtering

Press

```text
/
```

---

Search

```text
Pod Name

Deployment Name

Namespace
```

---

# Viewing YAML

Select Resource

---

Press

```text
y
```

---

Displays

```text
Complete YAML
```

---

Useful during troubleshooting.

---

# Viewing Events

Command

```text
:events
```

---

Shows

```text
Scheduling Failures

Mount Errors

CrashLoopBackOff

ImagePullBackOff
```

---

# Common Production Usage

## CrashLoopBackOff

Open

```text
Pods
```

---

Select Pod

---

Press

```text
l
```

---

View logs immediately.

---

## ImagePullBackOff

Open

```text
Events
```

---

Check

```text
Image Pull Errors
```

---

## Pod Pending

Open

```text
Describe
```

---

Check

```text
Node Scheduling

PVC Issues
```

---

# K9s vs Kubectl

| Feature | kubectl | K9s |
|----------|----------|----------|
| Resource Listing | Yes | Yes |
| Interactive UI | No | Yes |
| Logs | Yes | Yes |
| Exec | Yes | Yes |
| Events | Yes | Yes |
| YAML Viewing | Yes | Yes |
| Ease Of Use | Medium | High |

---

# Common Interview Questions

## What is K9s?

### Short Answer

K9s is a terminal-based Kubernetes dashboard used for cluster management and troubleshooting.

### Detailed Explanation

K9s provides an interactive interface for viewing and managing Kubernetes resources without repeatedly typing kubectl commands.

### Production Example

During a production incident, engineers use K9s to quickly access logs, events, and pod details.

### Follow-Up Questions

- Does K9s replace kubectl?
- How does K9s connect to Kubernetes?
- What troubleshooting features does K9s provide?

---

## Why Do DevOps Engineers Use K9s?

### Short Answer

To improve productivity while managing Kubernetes clusters.

### Detailed Explanation

K9s centralizes resource viewing, log analysis, event monitoring, and pod access into a single interface.

### Production Example

A DevOps engineer uses K9s to troubleshoot CrashLoopBackOff pods across multiple namespaces.

### Follow-Up Questions

- Is K9s GUI based?
- Can K9s view logs?
- Can K9s access multiple clusters?

---

## Can K9s Replace Kubectl?

### Short Answer

No.

### Detailed Explanation

K9s uses the existing kubectl configuration and complements kubectl rather than replacing it.

### Production Example

Infrastructure provisioning still uses kubectl and YAML manifests, while K9s helps with operations and troubleshooting.

### Follow-Up Questions

- Does K9s require kubectl?
- Does K9s use kubeconfig?
- Can K9s create resources?

---

# Key Takeaways

```text
K9s is a terminal-based Kubernetes dashboard.

K9s uses existing kubectl configuration.

K9s simplifies cluster navigation.

K9s provides quick access to logs.

K9s helps troubleshoot Kubernetes issues.

K9s supports pods, services, deployments, and StatefulSets.

K9s is widely used by platform and DevOps engineers.
```