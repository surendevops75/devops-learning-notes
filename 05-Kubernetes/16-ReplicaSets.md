# ReplicaSets

## Introduction

Applications should always be available.

Suppose you have:

```text
Frontend Pod
```

running successfully.

---

What happens if the Pod crashes?

```text
Application Becomes Unavailable
```

---

Manually creating Pods every time is not practical.

---

Kubernetes solves this using:

```text
ReplicaSets
```

---

# What is a ReplicaSet?

A ReplicaSet ensures:

```text
Desired Number Of Pods
```

are always running.

---

Think of ReplicaSet as:

```text
Pod Manager
```

---

Its primary responsibility is:

```text
Maintain Desired Pod Count
```

---

# Why Do We Need ReplicaSets?

Suppose:

```text
Frontend Application
```

needs:

```text
3 Pods
```

---

Current State

```text
Pod-1

Pod-2

Pod-3
```

---

One Pod crashes.

Current State

```text
Pod-1

Pod-2
```

---

Desired State

```text
3 Pods
```

---

ReplicaSet detects:

```text
Mismatch
```

---

Creates:

```text
New Pod
```

---

Result

```text
3 Pods Running Again
```

---

# Desired State Concept

Kubernetes always works with:

```text
Desired State
```

---

Example

```yaml
replicas: 3
```

---

Meaning

```text
Always Keep

3 Pods Running
```

---

Even if Pods fail.

---

# ReplicaSet Architecture

```text
ReplicaSet
      ↓

3 Pods
```

---

Example

```text
Frontend ReplicaSet
          ↓

Frontend Pod-1

Frontend Pod-2

Frontend Pod-3
```

---

# How ReplicaSet Works

Step 1

```text
User Creates ReplicaSet
```

---

Step 2

```text
ReplicaSet Creates Pods
```

---

Step 3

```text
One Pod Crashes
```

---

Step 4

```text
ReplicaSet Detects Failure
```

---

Step 5

```text
ReplicaSet Creates New Pod
```

---

Application remains available.

---

# ReplicaSet YAML

Example

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: frontend-rs

spec:

  replicas: 3

  selector:
    matchLabels:
      app: frontend

  template:

    metadata:
      labels:
        app: frontend

    spec:

      containers:

      - name: frontend

        image: nginx
```

---

Create ReplicaSet

```bash
kubectl apply -f replicaset.yaml
```

---

Verify

```bash
kubectl get rs
```

---

Output

```text
frontend-rs
```

---

# Understanding ReplicaSet YAML

Main Components

```text
replicas

selector

template
```

---

# Replicas

Defines:

```text
Number Of Pods
```

to maintain.

---

Example

```yaml
replicas: 3
```

---

Result

```text
3 Pods Running
```

---

# Selector

Defines:

```text
Which Pods
ReplicaSet Manages
```

---

Example

```yaml
selector:
  matchLabels:
    app: frontend
```

---

ReplicaSet searches for:

```text
app=frontend
```

Pods.

---

# Template

Template defines:

```text
How New Pods
Should Be Created
```

---

Example

```yaml
template:
```

contains:

```text
Labels

Containers

Images
```

---

# Template Example

```yaml
template:

  metadata:
    labels:
      app: frontend

  spec:

    containers:

    - name: frontend

      image: nginx
```

---

ReplicaSet uses template whenever:

```text
New Pod Needed
```

---

# Viewing ReplicaSets

List ReplicaSets

```bash
kubectl get rs
```

---

Output

```text
NAME          DESIRED

frontend-rs   3
```

---

Detailed Information

```bash
kubectl describe rs frontend-rs
```

---

Shows

```text
Desired Pods

Current Pods

Events
```

---

# Viewing Pods

```bash
kubectl get pods
```

---

Output

```text
frontend-abc

frontend-def

frontend-xyz
```

---

All created by ReplicaSet.

---

# Self-Healing

One of the biggest advantages.

---

Delete Pod

```bash
kubectl delete pod pod-name
```

---

Example

```bash
kubectl delete pod frontend-abc
```

---

Immediately

```text
ReplicaSet Creates New Pod
```

---

Verify

```bash
kubectl get pods
```

---

New Pod appears.

---

# Scaling ReplicaSets

Increase replicas.

---

Example

```yaml
replicas: 5
```

---

Apply

```bash
kubectl apply -f replicaset.yaml
```

---

Result

```text
5 Pods Running
```

---

Reduce replicas.

---

Example

```yaml
replicas: 2
```

---

Result

```text
Extra Pods Removed
```

---

# Scaling Using kubectl

Command

```bash
kubectl scale rs frontend-rs \
--replicas=5
```

---

Verify

```bash
kubectl get rs
```

---

# Label Matching

ReplicaSet heavily depends on labels.

---

Selector

```yaml
selector:
  matchLabels:
    app: frontend
```

---

Template

```yaml
labels:
  app: frontend
```

---

Must match.

---

Otherwise

```text
ReplicaSet Fails
```

---

# Common Mistake

Wrong Selector

```yaml
selector:
  matchLabels:
    app: frontend
```

---

Template

```yaml
labels:
  app: nginx
```

---

Result

```text
ReplicaSet Cannot Manage Pods
```

---

# ReplicaSet vs Pod

| Pod | ReplicaSet |
|------|------|
| Single Pod | Multiple Pods |
| No Self-Healing | Self-Healing |
| Manual Recovery | Automatic Recovery |
| No Scaling | Scaling Supported |

---

# ReplicaSet vs Deployment

ReplicaSet

```text
Creates Pods

Maintains Replicas
```

---

Deployment

```text
Manages ReplicaSets

Supports Rolling Updates

Supports Rollbacks
```

---

Production environments typically use:

```text
Deployments
```

not ReplicaSets directly.

---

# Real Production Example

Frontend Application

Requirement

```text
3 Replicas
```

---

ReplicaSet

```yaml
replicas: 3
```

---

If one Pod crashes:

```text
New Pod Created Automatically
```

---

Benefits

```text
High Availability

Self Healing

Scalability
```

---

# ReplicaSet Flow

```text
ReplicaSet
      ↓

Desired Replicas = 3
      ↓

Pod-1

Pod-2

Pod-3
```

---

One Pod Crashes

```text
ReplicaSet
      ↓

Creates New Pod
```

---

Desired State Restored.

---

# Troubleshooting ReplicaSets

Check ReplicaSets

```bash
kubectl get rs
```

---

Describe ReplicaSet

```bash
kubectl describe rs frontend-rs
```

---

Check Pods

```bash
kubectl get pods
```

---

Verify Labels

```bash
kubectl get pods --show-labels
```

---

Common Issues

```text
Label Mismatch

ImagePullBackOff

Insufficient Resources
```

---

# Common Interview Questions

## What is a ReplicaSet?

### Short Answer

A ReplicaSet ensures the desired number of Pod replicas are always running.

### Detailed Explanation

ReplicaSet continuously monitors Pods and recreates missing Pods to maintain availability.

### Production Example

Maintaining three frontend Pods.

### Follow-Up Questions

- What is self-healing?
- How does ReplicaSet find Pods?

---

## What is Self-Healing?

### Short Answer

ReplicaSet automatically recreates failed Pods.

### Detailed Explanation

If a Pod crashes or is deleted, ReplicaSet creates a replacement Pod.

### Production Example

Frontend Pod crash automatically recovered.

### Follow-Up Questions

- Which Kubernetes component performs self-healing?
- Does ReplicaSet restart containers?

---

## What is the Purpose of Selector?

### Short Answer

Selector identifies Pods managed by ReplicaSet.

### Example

```yaml
selector:
  matchLabels:
    app: frontend
```

### Follow-Up Questions

- What happens if labels don't match?
- Why are selectors important?

---

## Why Do We Use Deployments Instead Of ReplicaSets?

### Short Answer

Deployments provide rolling updates and rollback capabilities.

### Detailed Explanation

Deployments manage ReplicaSets and add advanced deployment features.

### Production Example

Upgrading frontend:v1 to frontend:v2.

### Follow-Up Questions

- Does Deployment create ReplicaSets?
- Can ReplicaSets perform rolling updates?

---

# Key Takeaways

```text
ReplicaSet maintains the desired number of Pods.

ReplicaSet provides self-healing.

ReplicaSet supports scaling.

ReplicaSet uses labels and selectors.

ReplicaSet automatically recreates failed Pods.

Template defines how Pods are created.

Replicas define how many Pods should run.

ReplicaSets are rarely created directly in production.

Deployments manage ReplicaSets and are preferred in real-world environments.

ReplicaSet is a core building block of Kubernetes workloads.
```