# StatefulSets

## Introduction

So far we have learned:

```text
Pods

ReplicaSets

Deployments

Persistent Volumes

Persistent Volume Claims

EBS

EFS
```

---

Deployments work very well for:

```text
Frontend

Catalogue

User

Cart

Shipping
```

---

These applications are usually:

```text
Stateless
```

---

However some applications require:

```text
Persistent Identity

Persistent Storage

Ordered Deployment

Ordered Scaling
```

---

Examples

```text
MongoDB

MySQL

PostgreSQL

Redis

Timeseries Databases
```

---

For such workloads Kubernetes provides:

```text
StatefulSets
```

---

# What is a Stateful Application?

A Stateful Application stores:

```text
Application State

Database Records

User Data

Persistent Data
```

---

Examples

```text
MongoDB

MySQL

Redis

Cassandra
```

---

If Pod changes:

```text
Data Must Survive
```

---

# What is a Stateless Application?

A Stateless Application stores:

```text
Only Code
```

---

Examples

```text
Frontend

Catalogue

User

Cart
```

---

If Pod is recreated:

```text
No Data Loss
```

---

Because data is stored elsewhere.

---

# Why Deployments Are Not Enough?

Deployment creates Pods like:

```text
frontend-abc123

frontend-xyz789

frontend-def456
```

---

If Pod is recreated:

```text
New Name

New Identity
```

---

For databases:

```text
Identity Matters
```

---

Need

```text
Stable Pod Name

Stable Storage

Stable Network Identity
```

---

Solution

```text
StatefulSet
```

---

# What is a StatefulSet?

A StatefulSet is a Kubernetes workload resource used for:

```text
Stateful Applications
```

---

It provides:

```text
Stable Pod Names

Stable Storage

Ordered Deployment

Ordered Scaling
```

---

# StatefulSet Architecture

```text
StatefulSet
      │

      ├── mongodb-0
      │      │
      │      └── PVC-0
      │
      ├── mongodb-1
      │      │
      │      └── PVC-1
      │
      └── mongodb-2
             │
             └── PVC-2
```

---

Each Pod gets:

```text
Unique Identity

Dedicated Storage
```

---

# Stable Pod Names

Deployment

```text
mongodb-abc123

mongodb-def456
```

---

Names change.

---

StatefulSet

```text
mongodb-0

mongodb-1

mongodb-2
```

---

Names remain stable.

---

Even after restart.

---

# Why Stable Names Matter?

Database clusters often communicate using:

```text
Hostnames
```

---

Example

```text
mongodb-0

mongodb-1

mongodb-2
```

---

Cluster members always know:

```text
Who Is Who
```

---

# Stable Storage

Each StatefulSet Pod gets:

```text
Dedicated PVC
```

---

Example

```text
mongodb-0
      ↓
mongodb-pvc-0

mongodb-1
      ↓
mongodb-pvc-1

mongodb-2
      ↓
mongodb-pvc-2
```

---

Even if Pod restarts:

```text
Same Storage
```

is attached.

---

# Ordered Deployment

Deployments create Pods:

```text
Parallel
```

---

StatefulSets create Pods:

```text
Sequentially
```

---

Example

```text
mongodb-0
      ↓

mongodb-1
      ↓

mongodb-2
```

---

Next Pod starts only after previous Pod becomes healthy.

---

# Ordered Scaling

Scale Up

```text
mongodb-0
      ↓

mongodb-1
      ↓

mongodb-2
```

---

Scale Down

```text
mongodb-2
      ↓

mongodb-1
      ↓

mongodb-0
```

---

Controlled and predictable.

---

# Headless Service

StatefulSets require:

```text
Headless Service
```

---

Why?

```text
Stable DNS Names
```

---

Example

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mongodb

spec:

  clusterIP: None

  selector:

    app: mongodb
```

---

Notice

```text
clusterIP: None
```

---

This creates:

```text
Headless Service
```

---

# DNS Resolution

Pods become reachable as:

```text
mongodb-0.mongodb

mongodb-1.mongodb

mongodb-2.mongodb
```

---

Useful for:

```text
Database Replication

Cluster Communication
```

---

# StatefulSet Example

```yaml
apiVersion: apps/v1

kind: StatefulSet

metadata:
  name: mongodb

spec:

  serviceName: mongodb

  replicas: 3

  selector:

    matchLabels:

      app: mongodb

  template:

    metadata:

      labels:

        app: mongodb

    spec:

      containers:

      - name: mongodb

        image: mongo
```

---

Create

```bash
kubectl apply -f statefulset.yaml
```

---

Verify

```bash
kubectl get statefulsets
```

---

# Viewing StatefulSets

List

```bash
kubectl get sts
```

---

Output

```text
mongodb
```

---

Describe

```bash
kubectl describe sts mongodb
```

---

Shows

```text
Replicas

Storage

Events
```

---

# Volume Claim Templates

StatefulSets automatically create PVCs.

---

Example

```yaml
volumeClaimTemplates:

- metadata:

    name: mongodb-storage

  spec:

    accessModes:

    - ReadWriteOnce

    resources:

      requests:

        storage: 20Gi
```

---

Result

```text
PVC Created Automatically
```

for every Pod.

---

# StatefulSet Storage

Example

```text
mongodb-0
      ↓
PVC-0
      ↓
EBS Volume

mongodb-1
      ↓
PVC-1
      ↓
EBS Volume
```

---

Storage remains even if Pod is deleted.

---

# StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|----------|------------|-------------|
| Workload Type | Stateless | Stateful |
| Stable Pod Name | No | Yes |
| Stable Storage | No | Yes |
| Ordered Deployment | No | Yes |
| Ordered Scaling | No | Yes |
| Dedicated Storage | No | Yes |
| Database Workloads | Not Ideal | Recommended |

---

# Common Stateful Workloads

## MongoDB

Needs

```text
Persistent Data

Stable Identity
```

---

Use

```text
StatefulSet
```

---

## MySQL

Needs

```text
Persistent Storage
```

---

Use

```text
StatefulSet
```

---

## Redis

Needs

```text
Persistent Data

Replication
```

---

Use

```text
StatefulSet
```

---

## Timeseries Databases

Examples

```text
InfluxDB

VictoriaMetrics
```

---

Need

```text
Persistent Storage
```

---

Use

```text
StatefulSet
```

---

# Production Example

Roboshop MongoDB

Requirements

```text
Persistent Database

Stable Hostname

Dedicated Storage
```

---

Architecture

```text
MongoDB StatefulSet
         ↓

mongodb-0
         ↓

PVC
         ↓

EBS Volume
```

---

Benefits

```text
Data Persistence

Reliable Recovery

Predictable Scaling
```

---

# Common Problems

## PVC Not Created

Check

```bash
kubectl get pvc
```

---

Verify

```text
StorageClass

VolumeClaimTemplates
```

---

## Pod Pending

Cause

```text
Storage Not Available
```

---

Check

```bash
kubectl describe pod
```

---

## DNS Not Working

Cause

```text
Headless Service Missing
```

---

Verify

```yaml
clusterIP: None
```

---

# Troubleshooting

Check StatefulSets

```bash
kubectl get sts
```

---

Check Pods

```bash
kubectl get pods
```

---

Check PVCs

```bash
kubectl get pvc
```

---

Describe StatefulSet

```bash
kubectl describe sts mongodb
```

---

Check DNS

```bash
nslookup mongodb-0.mongodb
```

---

# Common Interview Questions

## What is a StatefulSet?

### Short Answer

A StatefulSet is a Kubernetes workload resource used for stateful applications requiring stable identity and storage.

### Detailed Explanation

StatefulSets provide predictable pod names, dedicated storage, ordered deployment, and ordered scaling, making them ideal for databases and clustered applications.

### Production Example

MongoDB deployed as a StatefulSet in EKS with dedicated EBS volumes for each replica.

### Follow-Up Questions

- Why not use a Deployment?
- What makes an application stateful?
- What storage is commonly used with StatefulSets?

---

## What are the Advantages of StatefulSets?

### Short Answer

Stable identity, stable storage, and ordered operations.

### Detailed Explanation

Each Pod receives a predictable hostname and dedicated storage. Pods are created and terminated in a controlled sequence.

### Production Example

MongoDB replicas use stable hostnames such as mongodb-0 and mongodb-1 for cluster communication.

### Follow-Up Questions

- What is ordered deployment?
- What is ordered scaling?
- Why do databases need stable identities?

---

## Why Do StatefulSets Need Headless Services?

### Short Answer

To provide stable DNS names for Pods.

### Detailed Explanation

Headless Services allow each StatefulSet Pod to be reachable through a predictable hostname.

### Production Example

MongoDB nodes communicate using DNS names like mongodb-0.mongodb.

### Follow-Up Questions

- What is a Headless Service?
- What does clusterIP: None mean?
- How does DNS work in StatefulSets?

---

## StatefulSet vs Deployment?

### Short Answer

Deployments are for stateless applications, StatefulSets are for stateful applications.

### Detailed Explanation

Deployments focus on scaling and availability, while StatefulSets add stable identity and storage guarantees.

### Production Example

Frontend uses Deployment, MongoDB uses StatefulSet.

### Follow-Up Questions

- Can StatefulSets use PVCs?
- Can Deployments use PVCs?
- Which one is recommended for databases?

---

## What Happens When a StatefulSet Pod Restarts?

### Short Answer

It gets the same name and same storage.

### Detailed Explanation

StatefulSets preserve pod identity and reattach the same PVC to the recreated Pod.

### Production Example

mongodb-0 restarts and reconnects to the same EBS-backed PVC.

### Follow-Up Questions

- Does the hostname change?
- Does the PVC change?
- Why is this important for databases?

---

# Key Takeaways

```text
StatefulSets are designed for stateful applications.

Stateful applications require persistent storage.

StatefulSets provide stable pod identities.

Each Pod receives dedicated storage.

StatefulSets create and terminate Pods in order.

Headless Services provide stable DNS names.

MongoDB, MySQL, Redis, and Timeseries databases commonly use StatefulSets.

StatefulSets are the preferred Kubernetes resource for database workloads.
```