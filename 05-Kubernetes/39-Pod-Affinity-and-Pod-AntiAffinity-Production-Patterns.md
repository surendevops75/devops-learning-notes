# Pod Affinity and Pod Anti-Affinity Production Patterns

## Introduction

Earlier we learned:

```text
NodeSelector

Node Affinity

Node Anti-Affinity

Taints and Tolerations
```

---

These scheduling techniques focus on:

```text
Nodes
```

---

Sometimes we need scheduling decisions based on:

```text
Other Pods
```

instead of nodes.

---

Examples

```text
Application Pod Near Redis

Frontend Near Backend

Replica Separation

Latency Optimization
```

---

Kubernetes provides:

```text
Pod Affinity

Pod Anti-Affinity
```

---

# Why Pod Affinity?

Suppose:

```text
Application Pod
```

runs in:

```text
us-east-1a
```

---

Redis Pod runs in:

```text
us-east-1b
```

---

Every request travels between:

```text
Application

↓

Redis
```

---

Result

```text
Higher Network Latency
```

---

Solution

```text
Pod Affinity
```

---

# What is Pod Affinity?

Pod Affinity tells Scheduler:

```text
Schedule My Pod

Near Another Pod
```

---

Goal

```text
Reduce Network Latency

Improve Performance
```

---

# Pod Affinity Architecture

```text
Node-1

Redis
```

---

New Pod

```text
Application
```

---

Affinity Rule

```text
Run Near Redis
```

---

Result

```text
Application

Redis

Same Node
```

or nearby topology.

---

# Real Production Example

Application

```text
User Service
```

---

Dependency

```text
Redis Cache
```

---

Without Affinity

```text
User Service

→ us-east-1a

Redis

→ us-east-1b
```

---

Every cache call crosses AZ.

---

Result

```text
More Latency
```

---

With Pod Affinity

```text
User Service

Redis

Same Node
```

---

Result

```text
Faster Response Time
```

---

# Pod Affinity YAML

```yaml
affinity:

  podAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

    - labelSelector:

        matchExpressions:

        - key: app

          operator: In

          values:

          - redis

      topologyKey: kubernetes.io/hostname
```

---

Meaning

```text
Schedule Pod

Near Redis Pods
```

---

# Understanding topologyKey

Determines:

```text
How Close
```

Pods should be.

---

Example

```text
kubernetes.io/hostname
```

---

Means

```text
Same Node
```

---

Example

```text
topology.kubernetes.io/zone
```

---

Means

```text
Same Availability Zone
```

---

# Pod Anti-Affinity

Opposite of Pod Affinity.

---

Meaning

```text
Do Not Schedule

Near Another Pod
```

---

Goal

```text
High Availability
```

---

# Why Anti-Affinity?

Suppose:

```text
Redis-1

Redis-2
```

run on:

```text
Node-1
```

---

Node crashes.

---

Result

```text
Redis Cluster Down
```

---

Solution

```text
Spread Replicas
```

---

# Redis Example

Node-1

```text
Redis-1
```

---

Node-2

```text
Redis-2
```

---

Node-3

```text
Redis-3
```

---

Benefits

```text
Fault Tolerance

High Availability
```

---

# Pod Anti-Affinity YAML

```yaml
affinity:

  podAntiAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

    - labelSelector:

        matchExpressions:

        - key: app

          operator: In

          values:

          - redis

      topologyKey: kubernetes.io/hostname
```

---

Meaning

```text
Do Not Place

Redis Pods

On Same Node
```

---

# Web and Redis Architecture

Trainer Scenario

---

Nodes

```text
Server-1

Server-2

Server-3
```

---

Redis

```text
Redis-1

Redis-2

Redis-3
```

---

Web

```text
Web-1

Web-2

Web-3
```

---

Desired Behavior

```text
Web-1 Near Redis-1

Web-2 Near Redis-2

Web-3 Near Redis-3
```

---

At same time

```text
Web Replicas

Must Not Share Nodes
```

---

Uses

```text
Affinity

+

Anti-Affinity
```

together.

---

# Advanced Production Pattern

## Cache Affinity

```text
Application

↓

Redis
```

---

Keep together.

---

Benefits

```text
Lower Latency

Reduced Network Cost
```

---

# Database Affinity

```text
Application

↓

Database
```

---

Keep in same AZ.

---

Benefits

```text
Faster Queries
```

---

# Replica Anti-Affinity

```text
Frontend-1

Frontend-2

Frontend-3
```

---

Spread across:

```text
Different Nodes
```

---

Benefits

```text
High Availability
```

---

# Affinity vs Anti-Affinity

| Feature | Affinity | Anti-Affinity |
|----------|-----------|-----------|
| Goal | Bring Pods Together | Keep Pods Apart |
| Main Benefit | Performance | Availability |
| Example | App + Redis | Redis Replicas |
| Latency | Reduced | No Impact |
| Fault Tolerance | Limited | High |

---

# Required vs Preferred

## Required

```text
Hard Rule
```

---

Must satisfy condition.

---

Otherwise

```text
Pod Pending
```

---

## Preferred

```text
Soft Rule
```

---

Scheduler tries.

---

If not possible

```text
Choose Another Node
```

---

# Production Example

E-Commerce Platform

---

Frontend

```text
Web Pods
```

---

Cache

```text
Redis Cluster
```

---

Requirements

```text
Web Near Redis

Redis Replicas Separate
```

---

Solution

```text
Pod Affinity

+

Pod Anti-Affinity
```

---

Benefits

```text
Low Latency

High Availability
```

---

# Common Problems

## Pod Pending

Cause

```text
Affinity Rules Too Strict
```

---

Check

```bash
kubectl describe pod pod-name
```

---

## Anti-Affinity Failure

Cause

```text
Insufficient Nodes
```

---

Example

```text
3 Replicas

1 Node
```

---

Scheduler cannot satisfy rule.

---

## Unexpected Placement

Cause

```text
Preferred Rule
```

instead of:

```text
Required Rule
```

---

# Troubleshooting

Check Pod

```bash
kubectl describe pod pod-name
```

---

View Events

```bash
kubectl get events
```

---

Check Labels

```bash
kubectl get pods --show-labels
```

---

Check Scheduling Errors

```text
Affinity Rules Not Satisfied
```

---

# Common Interview Questions

## What is Pod Affinity?

### Short Answer

Pod Affinity schedules Pods near other Pods.

### Detailed Explanation

Scheduler places Pods close to matching Pods to reduce latency and improve communication efficiency.

### Production Example

User Service runs near Redis Cache for faster cache lookups.

### Follow-Up Questions

- Does Pod Affinity use node labels?
- What is topologyKey?
- What problem does Affinity solve?

---

## What is Pod Anti-Affinity?

### Short Answer

Pod Anti-Affinity prevents Pods from running together.

### Detailed Explanation

Scheduler spreads replicas across nodes or zones to improve fault tolerance and availability.

### Production Example

Redis replicas are distributed across multiple worker nodes.

### Follow-Up Questions

- Why is Anti-Affinity important?
- What happens during node failure?
- Can Anti-Affinity be preferred?

---

## What is topologyKey?

### Short Answer

topologyKey defines the boundary used for Affinity or Anti-Affinity decisions.

### Detailed Explanation

Scheduler uses topologyKey to determine whether Pods should be placed on the same node, zone, or region.

### Production Example

Using:

```text
kubernetes.io/hostname
```

keeps Pods on the same node.

### Follow-Up Questions

- Can topologyKey use zones?
- What is hostname topology?
- Why is topologyKey required?

---

## When Should You Use Pod Affinity?

### Short Answer

When applications frequently communicate with each other.

### Detailed Explanation

Keeping related services close together reduces network latency and improves performance.

### Production Example

Web application and Redis cache run in the same node.

### Follow-Up Questions

- Does it improve latency?
- Does it improve availability?
- Is it suitable for databases?

---

## When Should You Use Pod Anti-Affinity?

### Short Answer

When replicas should be distributed for high availability.

### Detailed Explanation

Anti-Affinity prevents multiple replicas from sharing the same failure domain.

### Production Example

Redis replicas run on different worker nodes.

### Follow-Up Questions

- Why spread replicas?
- What is a failure domain?
- Can Anti-Affinity use zones?

---

# Key Takeaways

```text
Pod Affinity places Pods together.

Pod Anti-Affinity separates Pods.

Affinity improves performance.

Anti-Affinity improves availability.

Redis and Application Pods commonly use Affinity.

Redis Replicas commonly use Anti-Affinity.

topologyKey controls placement boundaries.

Required rules are mandatory.

Preferred rules are optional.

Affinity and Anti-Affinity are common production scheduling techniques.
```