# Persistent Volumes and Persistent Volume Claims

## Introduction

Earlier we learned:

```text
emptyDir

hostPath
```

---

These are:

```text
Ephemeral Storage
```

---

Problem

```text
Pod Deleted
      ↓

Data Deleted
```

---

This is acceptable for:

```text
Logs

Cache

Temporary Files
```

---

But not for:

```text
Databases

Business Data

User Uploads

Application Data
```

---

We need:

```text
Persistent Storage
```

---

# What is Persistent Storage?

Persistent Storage means:

```text
Data Survives

Pod Restart

Pod Deletion

Pod Recreation
```

---

Storage lifecycle becomes:

```text
Independent Of Pod
```

---

Example

```text
Pod Deleted
      ↓

Storage Remains
```

---

# Why Persistent Storage?

Suppose:

```text
MongoDB Running In Kubernetes
```

---

Stores

```text
Products

Orders

Users
```

---

Pod crashes.

---

New Pod created.

---

Without persistent storage:

```text
Database Lost
```

---

Business Impact

```text
Data Loss
```

---

Solution

```text
Persistent Volumes
```

---

# Kubernetes Storage Abstraction

Kubernetes does not directly expose disks to Pods.

---

Instead Kubernetes provides:

```text
Persistent Volume (PV)

Persistent Volume Claim (PVC)
```

---

Architecture

```text
Pod
 ↓

PVC
 ↓

PV
 ↓

Disk
```

---

Think of:

```text
Pod
```

as:

```text
Application
```

---

PVC as:

```text
Request
```

---

PV as:

```text
Storage Resource
```

---

Disk as:

```text
Actual Storage
```

---

# Understanding the Relationship

Example

```text
Pod
 ↓

PVC
 ↓

PV
 ↓

EBS Disk
```

---

Application never talks directly to disk.

---

Kubernetes manages storage access.

---

# Real World Analogy

```text
Pod
```

is like:

```text
Son
```

---

```text
PVC
```

is like:

```text
Mother
```

---

```text
PV
```

is like:

```text
Father
```

---

```text
Disk
```

is like:

```text
Wallet
```

---

Flow

```text
Son
 ↓

Mother
 ↓

Father
 ↓

Wallet
```

---

Kubernetes Flow

```text
Pod
 ↓

PVC
 ↓

PV
 ↓

Disk
```

---

# What is a Persistent Volume?

A Persistent Volume (PV) is:

```text
Kubernetes Object

Representing Storage
```

---

Think of PV as:

```text
Storage Resource
Inside Kubernetes
```

---

PV represents:

```text
EBS

EFS

NFS

Other Storage
```

---

# PV Architecture

```text
PV
      ↓

Represents

Disk
```

---

Pod never mounts disk directly.

---

Pod uses PVC.

---

PVC uses PV.

---

PV uses Disk.

---

# Persistent Volume Example

```yaml
apiVersion: v1

kind: PersistentVolume

metadata:
  name: mongodb-pv

spec:

  capacity:

    storage: 10Gi

  accessModes:

  - ReadWriteOnce

  hostPath:

    path: /data
```

---

Create

```bash
kubectl apply -f pv.yaml
```

---

Verify

```bash
kubectl get pv
```

---

# What is PVC?

PVC means:

```text
Persistent Volume Claim
```

---

PVC is a request for storage.

---

Example

```text
Application Needs

10Gi Storage
```

---

PVC requests:

```text
10Gi
```

---

Kubernetes finds matching PV.

---

# PVC Architecture

```text
Pod
 ↓

PVC
```

---

PVC requests storage.

---

Kubernetes binds PVC to PV.

---

# PVC Example

```yaml
apiVersion: v1

kind: PersistentVolumeClaim

metadata:
  name: mongodb-pvc

spec:

  accessModes:

  - ReadWriteOnce

  resources:

    requests:

      storage: 10Gi
```

---

Create

```bash
kubectl apply -f pvc.yaml
```

---

Verify

```bash
kubectl get pvc
```

---

# PVC Binding Process

PVC Created

---

Kubernetes checks:

```text
Available PVs
```

---

Matching PV found.

---

PVC becomes:

```text
Bound
```

---

Architecture

```text
PVC
 ↓

PV
 ↓

Disk
```

---

# Using PVC in Pod

Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: mongodb

spec:

  containers:

  - name: mongodb

    image: mongo

    volumeMounts:

    - name: mongodb-storage

      mountPath: /data/db

  volumes:

  - name: mongodb-storage

    persistentVolumeClaim:

      claimName: mongodb-pvc
```

---

Result

```text
MongoDB Data

Stored On Persistent Storage
```

---

# Access Modes

Access mode defines:

```text
How Storage Can Be Mounted
```

---

Common Modes

```text
ReadWriteOnce (RWO)

ReadOnlyMany (ROX)

ReadWriteMany (RWX)
```

---

# ReadWriteOnce (RWO)

Meaning

```text
One Node

Read + Write
```

---

Common For

```text
EBS
```

---

# ReadOnlyMany (ROX)

Meaning

```text
Many Nodes

Read Only
```

---

Less common.

---

# ReadWriteMany (RWX)

Meaning

```text
Multiple Nodes

Read + Write
```

---

Common For

```text
EFS

NFS
```

---

# EBS Storage

Amazon EBS

```text
Elastic Block Storage
```

---

Characteristics

```text
High Performance

Low Latency

Block Storage
```

---

Can be attached to:

```text
One Node At A Time
```

---

Best For

```text
Databases

Operating Systems

High Performance Workloads
```

---

Examples

```text
MongoDB

MySQL

PostgreSQL
```

---

# EFS Storage

Amazon EFS

```text
Elastic File System
```

---

Characteristics

```text
Shared File Storage

Network File System

Multi Node Access
```

---

Can be mounted by:

```text
Multiple Nodes
```

simultaneously.

---

Best For

```text
Shared Files

Reports

User Uploads

Application Assets
```

---

# EBS vs EFS

| Feature | EBS | EFS |
|----------|----------|----------|
| Type | Block Storage | File Storage |
| Performance | Faster | Slower |
| Node Access | Single Node | Multiple Nodes |
| Database Usage | Yes | No |
| Shared Files | No | Yes |
| Access Mode | RWO | RWX |

---

# Static Provisioning

Traditional approach.

---

Storage Team

```text
Creates Disk
```

---

Kubernetes Admin

```text
Creates PV
```

---

DevOps Engineer

```text
Creates PVC
```

---

Application

```text
Uses PVC
```

---

Flow

```text
Disk Created
      ↓

PV Created
      ↓

PVC Created
      ↓

Pod Uses PVC
```

---

# Team Responsibilities

Storage Team

```text
Create Storage

Expand Storage

Backup Storage

Restore Storage
```

---

Kubernetes Admin

```text
Create PVs

Manage Storage Resources
```

---

DevOps Engineer

```text
Create PVC

Mount Storage

Configure Applications
```

---

# Common Production Example

Catalogue Service

```text
Stateless
```

---

Uses

```text
ClusterIP
```

only.

---

MongoDB

```text
Stateful
```

---

Requires

```text
Persistent Storage
```

---

Architecture

```text
MongoDB Pod
      ↓

PVC
      ↓

PV
      ↓

EBS Volume
```

---

# Troubleshooting

View PV

```bash
kubectl get pv
```

---

View PVC

```bash
kubectl get pvc
```

---

Describe PVC

```bash
kubectl describe pvc mongodb-pvc
```

---

Check Status

```text
Pending

Bound
```

---

Expected

```text
Bound
```

---

# Common Problems

## PVC Pending

Cause

```text
No Matching PV
```

---

Check

```bash
kubectl get pv
```

---

Verify

```text
Size

Access Mode
```

---

## Pod Cannot Mount Storage

Check

```bash
kubectl describe pod pod-name
```

---

Verify

```text
PVC Exists

PVC Bound
```

---

# Common Interview Questions

## What is a Persistent Volume?

### Short Answer

A Persistent Volume is a Kubernetes storage resource that represents underlying storage.

### Production Example

An EBS volume represented as a PV.

---

## What is a PVC?

### Short Answer

A Persistent Volume Claim is a request for storage made by an application.

### Production Example

MongoDB requesting 10Gi storage.

---

## Explain Pod → PVC → PV → Disk

### Short Answer

Pod uses PVC, PVC binds to PV, PV represents the actual storage.

### Production Example

MongoDB storing data in an EBS volume.

---

## What is the Difference Between PV and PVC?

### PV

```text
Storage Resource
```

### PVC

```text
Storage Request
```

---

## Difference Between EBS and EFS?

### EBS

```text
Single Node

High Performance

Database Storage
```

### EFS

```text
Multiple Nodes

Shared Storage

File Sharing
```

---

# Key Takeaways

```text
Persistent storage survives Pod deletion.

PV represents storage inside Kubernetes.

PVC is a request for storage.

Pods use PVCs, not PVs directly.

PV maps to actual storage such as EBS or EFS.

EBS is ideal for databases.

EFS is ideal for shared file storage.

Static provisioning requires manual storage creation.

PVC status should become Bound before Pods can use storage.

Persistent storage is critical for stateful applications.
```