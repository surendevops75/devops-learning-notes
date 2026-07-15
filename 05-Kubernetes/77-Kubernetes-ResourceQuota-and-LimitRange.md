# Kubernetes ResourceQuota and LimitRange

## Introduction

In a shared Kubernetes cluster, multiple teams deploy applications into the same cluster.

Without resource governance:

```text
One Team

Can Consume

All CPU

All Memory

All Storage
```

---

Result:

```text
Resource Starvation

Pod Scheduling Failures

Cluster Instability
```

---

Kubernetes provides:

```text
ResourceQuota

LimitRange
```

to control resource consumption.

---

# Real Production Problem

Namespace

```text
roboshop-dev
```

---

Developer Deploys

```text
50 Pods
```

---

Each Pod Requests

```text
4 CPU

8 GB RAM
```

---

Result

```text
Cluster Resources Exhausted
```

---

Other Applications

```text
Cannot Schedule Pods
```

---

Solution

```text
ResourceQuota

+

LimitRange
```

---

# What Is ResourceQuota?

ResourceQuota controls:

```text
Total Resources

Allowed

Inside A Namespace
```

---

Think Of It As:

```text
Namespace Budget
```

---

ResourceQuota Limits

```text
CPU

Memory

Storage

Pods

PVCs

Services

Secrets

ConfigMaps
```

---

# ResourceQuota Architecture

```text
Namespace

↓

ResourceQuota

↓

Allowed Resources

↓

Applications
```

---

# Example

Namespace

```text
roboshop-dev
```

---

Quota

```text
20 Pods

10 CPU

20 GB Memory
```

---

Meaning

Applications Cannot Exceed:

```text
20 Pods

10 CPU

20 GB Memory
```

---

# ResourceQuota Example YAML

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: roboshop-quota

spec:

  hard:

    pods: "20"

    requests.cpu: "10"

    requests.memory: 20Gi

    limits.cpu: "20"

    limits.memory: 40Gi
```

---

# What Happens When Limit Exceeds?

Developer Deploys

```text
25 Pods
```

---

Quota

```text
20 Pods
```

---

Result

```text
Deployment Rejected
```

---

Error

```text
Exceeded ResourceQuota
```

---

# ResourceQuota Example

Namespace Budget

```text
Pods        20

CPU         10

Memory      20Gi
```

---

Current Usage

```text
Pods        18

CPU         8

Memory      15Gi
```

---

Available

```text
Pods        2

CPU         2

Memory      5Gi
```

---

# Verify ResourceQuota

```bash
kubectl get resourcequota
```

---

Describe

```bash
kubectl describe resourcequota
```

---

Output

```text
Used

Hard
```

---

# What Is LimitRange?

ResourceQuota Controls:

```text
Namespace Total Resources
```

---

LimitRange Controls:

```text
Individual Pod Resources
```

---

Think Of It As:

```text
Per Pod Rules
```

---

# Why LimitRange Is Needed?

Without LimitRange

Developer Creates

```yaml
resources:
```

Nothing Specified.

---

Result

```text
Unlimited Resource Usage
```

---

Risk

```text
OOM Issues

Node Starvation

Unpredictable Scheduling
```

---

# LimitRange Functions

```text
Default CPU

Default Memory

Minimum CPU

Maximum CPU

Minimum Memory

Maximum Memory
```

---

# Example

Policy

```text
Every Pod Must Request

At Least

100m CPU

128Mi Memory
```

---

And

```text
Cannot Exceed

2 CPU

4Gi Memory
```

---

# LimitRange Example YAML

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: application-limits

spec:

  limits:

  - default:

      cpu: "500m"

      memory: "512Mi"

    defaultRequest:

      cpu: "250m"

      memory: "256Mi"

    max:

      cpu: "2"

      memory: "4Gi"

    min:

      cpu: "100m"

      memory: "128Mi"

    type: Container
```

---

# How LimitRange Works

Developer Creates

```yaml
container:
```

without resources.

---

Kubernetes Automatically Applies

```text
Default Requests

Default Limits
```

---

Result

```text
Consistent Resource Allocation
```

---

# ResourceQuota vs LimitRange

## ResourceQuota

Controls

```text
Namespace Total Resources
```

---

Example

```text
20 Pods

10 CPU

20Gi Memory
```

---

## LimitRange

Controls

```text
Single Pod Resources
```

---

Example

```text
Max

2 CPU

4Gi Memory
```

---

# Production Example

Namespace

```text
roboshop-dev
```

---

ResourceQuota

```text
20 Pods

10 CPU

20Gi Memory
```

---

LimitRange

```text
Min CPU

100m
```

---

```text
Max CPU

2
```

---

Result

```text
Namespace Protected

Pods Standardized
```

---

# ResourceQuota Flow

Developer Deploys

↓

Deployment

↓

API Server

↓

ResourceQuota Validation

↓

Allowed?

↓

Create Resource

---

If Exceeded

```text
Rejected
```

---

# LimitRange Flow

Developer Creates Pod

↓

No Resources Specified

↓

LimitRange Applies Defaults

↓

Pod Created

---

# Multi-Team Production Example

Cluster

```text
Dev Team

QA Team

Platform Team
```

---

Each Namespace Gets

```text
ResourceQuota
```

---

Dev

```text
5 CPU
```

---

QA

```text
5 CPU
```

---

Platform

```text
20 CPU
```

---

Result

```text
Fair Resource Usage
```

---

# Common Production Problems

## ResourceQuota Exceeded

Symptoms

```text
Deployment Failed
```

---

Error

```text
Exceeded Quota
```

---

Verify

```bash
kubectl describe resourcequota
```

---

# Missing Resource Requests

Symptoms

```text
Scheduling Problems
```

---

Cause

```text
No LimitRange
```

---

# OOMKilled Containers

Cause

```text
Memory Limits Too Low
```

---

Verify

```bash
kubectl describe pod
```

---

# Overly Restrictive Limits

Symptoms

```text
Application Crash
```

---

Cause

```text
Insufficient CPU

Insufficient Memory
```

---

# Troubleshooting Commands

Resource Quotas

```bash
kubectl get resourcequota
```

---

Describe Quota

```bash
kubectl describe resourcequota
```

---

LimitRanges

```bash
kubectl get limitrange
```

---

Describe LimitRange

```bash
kubectl describe limitrange
```

---

Pod Resources

```bash
kubectl describe pod <pod-name>
```

---

Namespace Resources

```bash
kubectl top pods
```

---

# Production Architecture

```text
Namespace

↓

ResourceQuota

↓

Total Resource Control

↓

LimitRange

↓

Per Pod Resource Control

↓

Application Pods
```

---

# Common Interview Questions

## What Is ResourceQuota?

### Short Answer

ResourceQuota limits the total amount of resources that can be consumed within a namespace.

### Detailed Explanation

ResourceQuota prevents teams or applications from consuming excessive cluster resources by enforcing namespace-level limits.

### Production Example

```text
Namespace

Max 20 Pods

Max 10 CPU

Max 20Gi Memory
```

### Follow-Up Questions

- Does ResourceQuota work cluster-wide?
- What resources can be limited?
- What happens when quota is exceeded?
- How do you verify quota usage?

---

## What Is LimitRange?

### Short Answer

LimitRange defines minimum, maximum, and default resource values for containers within a namespace.

### Detailed Explanation

LimitRange ensures consistent resource allocation and prevents workloads from running without proper requests and limits.

### Production Example

```text
Default CPU

250m

Default Memory

256Mi
```

### Follow-Up Questions

- What is defaultRequest?
- What is default limit?
- Why use LimitRange?
- What happens without it?

---

## Difference Between ResourceQuota And LimitRange?

### Short Answer

ResourceQuota controls namespace resources, while LimitRange controls individual container resources.

### Detailed Explanation

ResourceQuota defines total namespace budgets, whereas LimitRange defines per-container constraints and defaults.

### Production Example

ResourceQuota

```text
20 Pods
```

LimitRange

```text
Max 2 CPU Per Container
```

### Follow-Up Questions

- Can both be used together?
- Which one applies first?
- What happens during validation?
- How do they improve multi-tenancy?

---

## Why Are ResourceQuota And LimitRange Important?

### Short Answer

They prevent resource abuse and ensure fair cluster utilization.

### Detailed Explanation

These controls help maintain cluster stability by enforcing namespace-level budgets and per-container limits.

### Production Example

```text
Dev Team

Cannot Consume

Entire Cluster
```

### Follow-Up Questions

- How do they support multi-team clusters?
- What happens when limits are too strict?
- How do you tune quotas?
- How do you monitor resource usage?

---

# Key Takeaways

```text
ResourceQuota controls total namespace resources.

LimitRange controls per-container resources.

ResourceQuota prevents resource exhaustion.

LimitRange enforces resource standards.

ResourceQuota validates resource creation requests.

LimitRange applies default requests and limits.

Both are essential in multi-team Kubernetes environments.

ResourceQuota improves fairness across namespaces.

LimitRange improves scheduling predictability.

Production clusters commonly use both together.
```