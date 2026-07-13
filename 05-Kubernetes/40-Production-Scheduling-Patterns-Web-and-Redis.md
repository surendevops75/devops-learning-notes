# Production Scheduling Patterns - Web and Redis

## Introduction

In production environments, applications rarely run alone.

---

Most applications depend on:

```text
Cache

Database

Message Queues

Search Engines
```

---

Example

```text
Web Application
       ↓

Redis Cache
```

---

Every request may need:

```text
Cache Read

Cache Write

Session Lookup
```

---

Improper scheduling can cause:

```text
High Latency

Network Overhead

Poor Performance

Single Point Of Failure
```

---

Kubernetes provides:

```text
Pod Affinity

Pod Anti-Affinity
```

to solve these problems.

---

# Production Scenario

E-Commerce Platform

---

Components

```text
Frontend Web Application

Redis Cache Cluster
```

---

Requirements

```text
Web Pods Near Redis Pods

Redis Replicas Separate

Web Replicas Separate
```

---

Goals

```text
Reduce Latency

Improve Performance

Increase Availability

Avoid Single Point Of Failure
```

---

# Bad Architecture

Suppose all Pods run on the same node.

```text
Node-1

Redis-1
Redis-2
Redis-3

Web-1
Web-2
Web-3
```

---

Problem

```text
Node Failure

↓

Entire Application Down
```

---

This design is not production ready.

---

# Better Architecture

```text
Node-1

Redis-1
Web-1
```

---

```text
Node-2

Redis-2
Web-2
```

---

```text
Node-3

Redis-3
Web-3
```

---

Benefits

```text
Low Latency

High Availability

Fault Tolerance
```

---

# Why Pod Affinity?

Suppose

```text
Web-1
```

runs in

```text
us-east-1a
```

---

Redis runs in

```text
us-east-1b
```

---

Flow

```text
User
 ↓

Web
 ↓

Redis
```

---

Every request crosses:

```text
Node Boundary

Or

Availability Zone Boundary
```

---

Result

```text
Higher Latency
```

---

Solution

```text
Pod Affinity
```

---

# Why Pod Anti-Affinity?

Suppose

```text
Redis-1

Redis-2

Redis-3
```

all run on:

```text
Node-1
```

---

Node crashes.

---

Result

```text
Entire Redis Cluster Down
```

---

Solution

```text
Pod Anti-Affinity
```

---

# Final Design

Requirements

```text
Web Pods Should Run Near Redis Pods

Redis Replicas Must Be Separate

Web Replicas Must Be Separate
```

---

Solution

```text
Pod Affinity

+

Pod Anti-Affinity
```

---

# Step 1 - Redis StatefulSet

## Goal

```text
Distribute Redis Replicas

Across Multiple Nodes
```

---

YAML

```yaml
apiVersion: apps/v1

kind: StatefulSet

metadata:
  name: redis

spec:

  serviceName: redis

  replicas: 3

  selector:
    matchLabels:
      app: redis

  template:

    metadata:
      labels:
        app: redis

    spec:

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

      containers:

      - name: redis
        image: redis
```

---

# Redis Anti-Affinity Explanation

Rule

```text
Do Not Schedule

Redis Pods

On Same Node
```

---

Result

```text
Node-1

Redis-1
```

---

```text
Node-2

Redis-2
```

---

```text
Node-3

Redis-3
```

---

Benefits

```text
High Availability

Fault Tolerance
```

---

# Step 2 - Web Deployment

## Goal

```text
Place Web Pods

Near Redis Pods
```

---

YAML

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: web

spec:

  replicas: 3

  selector:
    matchLabels:
      app: web

  template:

    metadata:
      labels:
        app: web

    spec:

      affinity:

        podAffinity:

          preferredDuringSchedulingIgnoredDuringExecution:

          - weight: 100

            podAffinityTerm:

              labelSelector:

                matchExpressions:

                - key: app
                  operator: In
                  values:
                  - redis

              topologyKey: kubernetes.io/hostname

      containers:

      - name: web
        image: nginx
```

---

# Web Affinity Explanation

Scheduler preference

```text
Place Web Pods

Near Redis Pods
```

---

Meaning

```text
Web-1 Near Redis-1

Web-2 Near Redis-2

Web-3 Near Redis-3
```

---

Benefits

```text
Reduced Latency

Reduced Network Traffic

Faster Cache Access
```

---

# Step 3 - Add Web Anti-Affinity

Requirement

```text
Do Not Place

Multiple Web Replicas

On Same Node
```

---

Updated Affinity

```yaml
affinity:

  podAffinity:

    preferredDuringSchedulingIgnoredDuringExecution:

    - weight: 100

      podAffinityTerm:

        labelSelector:

          matchExpressions:

          - key: app
            operator: In
            values:
            - redis

        topologyKey: kubernetes.io/hostname

  podAntiAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

    - labelSelector:

        matchExpressions:

        - key: app
          operator: In
          values:
          - web

      topologyKey: kubernetes.io/hostname
```

---

# Web Anti-Affinity Explanation

Rule

```text
Web Pods

Must Not Share Nodes
```

---

Result

```text
Web-1 → Node-1

Web-2 → Node-2

Web-3 → Node-3
```

---

Benefits

```text
High Availability

Replica Distribution

Fault Isolation
```

---

# Final Production Layout

```text
Node-1
--------
Redis-1
Web-1
```

---

```text
Node-2
--------
Redis-2
Web-2
```

---

```text
Node-3
--------
Redis-3
Web-3
```

---

# Request Flow

```text
User
  ↓

Web-1
  ↓

Redis-1
```

---

Traffic stays local.

---

Result

```text
Low Latency

Better Performance
```

---

# Node Failure Scenario

Suppose

```text
Node-2
```

fails.

---

Affected

```text
Redis-2

Web-2
```

---

Still Available

```text
Redis-1

Redis-3

Web-1

Web-3
```

---

Application continues serving traffic.

---

# Why This Design Is Popular?

Because it provides:

```text
Performance

Availability

Scalability
```

at the same time.

---

# Affinity Benefit

```text
Web Near Redis

Lower Latency

Faster Response Time
```

---

# Anti-Affinity Benefit

```text
Replica Separation

Node Failure Protection

High Availability
```

---

# Combined Benefit

```text
Performance

+

High Availability
```

---

# Common Problems

## Pods Remain Pending

Cause

```text
Affinity Rules Too Strict
```

---

Example

```text
3 Replicas

Only 1 Node
```

---

Scheduler cannot satisfy requirements.

---

## Pods Not Running Together

Cause

```text
Anti-Affinity Rules
```

---

Expected behavior.

---

## Affinity Not Working

Cause

```text
Incorrect Labels
```

---

Verify

```bash
kubectl get pods --show-labels
```

---

# Troubleshooting

Check Pod Placement

```bash
kubectl get pods -o wide
```

---

Describe Pod

```bash
kubectl describe pod pod-name
```

---

Check Events

```bash
kubectl get events
```

---

Verify Labels

```bash
kubectl get pods --show-labels
```

---

# Common Interview Questions

## Why Use Pod Affinity For Redis?

### Short Answer

To reduce latency between Web Pods and Redis Pods.

### Detailed Explanation

Scheduler places application Pods close to Redis Pods so cache requests do not travel across nodes or availability zones.

### Production Example

Web Pods run near Redis Pods using Pod Affinity.

### Follow-Up Questions

- What is topologyKey?
- Why use preferred affinity?
- Can affinity improve performance?

---

## Why Use Pod Anti-Affinity For Redis?

### Short Answer

To distribute Redis replicas across different nodes.

### Detailed Explanation

Anti-Affinity prevents multiple Redis replicas from running on the same node, improving availability.

### Production Example

Redis-1, Redis-2, and Redis-3 run on separate worker nodes.

### Follow-Up Questions

- What happens during node failure?
- Why spread replicas?
- What is a failure domain?

---

## Why Use Pod Anti-Affinity For Web Pods?

### Short Answer

To prevent all replicas from running on a single node.

### Detailed Explanation

Spreading replicas improves fault tolerance and application availability.

### Production Example

Web-1, Web-2, and Web-3 run on separate worker nodes.

### Follow-Up Questions

- Is Anti-Affinity mandatory?
- What happens if only one node exists?
- Can preferred Anti-Affinity be used?

---

## Why Combine Affinity And Anti-Affinity?

### Short Answer

Affinity improves performance while Anti-Affinity improves availability.

### Detailed Explanation

Production systems often need both low latency and high availability. Affinity keeps related applications together, while Anti-Affinity spreads replicas apart.

### Production Example

Web Pods run near Redis Pods while Redis and Web replicas are distributed across separate nodes.

### Follow-Up Questions

- Which improves latency?
- Which improves fault tolerance?
- Is this a common production pattern?

---

# Key Takeaways

```text
Pod Affinity places related Pods together.

Pod Anti-Affinity separates replicas.

Redis and Web applications commonly use Affinity.

Redis replicas commonly use Anti-Affinity.

Web replicas commonly use Anti-Affinity.

Affinity improves performance.

Anti-Affinity improves availability.

Combining both is a common production design pattern.

This design reduces latency and improves fault tolerance.
```