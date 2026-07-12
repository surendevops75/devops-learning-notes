# Kubernetes Volumes

## Introduction

Containers are designed to be:

```text
Lightweight

Portable

Ephemeral
```

---

Applications often need:

```text
Storage

Logs

Configuration Files

Application Data
```

---

Question

```text
Where Does A Container Store Data?
```

---

By default:

```text
Container Filesystem
```

---

Problem

```text
Container Deleted
      ↓

Data Deleted
```

---

Kubernetes solves this using:

```text
Volumes
```

---

# Why Do We Need Volumes?

Suppose an application writes:

```text
Logs

Uploaded Files

Reports

Cache Data
```

inside a container.

---

Application Running

```text
Pod
  ↓

Container
  ↓

Data Created
```

---

Pod Deleted

```text
Pod Removed
      ↓

Container Removed
      ↓

Data Lost
```

---

Need

```text
External Storage
```

---

Solution

```text
Volumes
```

---

# What is a Volume?

A Volume is:

```text
Storage Attached
To A Pod
```

---

Volumes allow:

```text
Containers To Read Data

Containers To Write Data

Data Sharing

Data Persistence
```

---

Think of Volume as:

```text
Additional Storage

Mounted Into Container
```

---

# Volume Architecture

```text
Pod
 │
 ├── Container-1
 │
 └── Volume
```

---

Container accesses:

```text
Mounted Volume
```

instead of storing everything inside its filesystem.

---

# Why Storage Matters

Applications generate:

```text
Logs

Configuration

Temporary Files

Business Data
```

---

Without volumes:

```text
Data Loss
```

occurs whenever Pods disappear.

---

# Pods Are Ephemeral

Pods are temporary.

---

Example

```text
Pod Running
      ↓

Node Failure
      ↓

Pod Recreated
```

---

New Pod

```text
New Container

New Filesystem
```

---

Old Data

```text
Lost
```

---

# Nodes Are Also Ephemeral

In cloud environments:

```text
EC2 Instances

Worker Nodes
```

can also be replaced.

---

Examples

```text
Auto Scaling

Spot Termination

Node Upgrade

Node Failure
```

---

Therefore:

```text
Pod Storage

Node Storage
```

cannot always be trusted.

---

# Types of Storage

Kubernetes storage can be grouped into:

```text
Ephemeral Storage

Persistent Storage
```

---

# Ephemeral Storage

Lifetime

```text
Until Pod Exists
```

---

When Pod Deleted

```text
Data Deleted
```

---

Examples

```text
emptyDir

hostPath
```

---

Good For

```text
Temporary Data

Caching

Sharing Data
Between Containers
```

---

# Persistent Storage

Lifetime

```text
Independent Of Pod
```

---

Pod Deleted

```text
Data Remains
```

---

Examples

```text
EBS

EFS

Persistent Volumes
```

---

Good For

```text
Databases

Uploads

Reports

Application Data
```

---

# Storage Classification

```text
Volumes
   │
   ├── Ephemeral
   │      │
   │      ├── emptyDir
   │      └── hostPath
   │
   └── Persistent
          │
          ├── PV
          ├── PVC
          └── StorageClass
```

---

# Volume Lifecycle

Volume created.

---

Pod starts.

---

Volume mounted.

---

Container reads and writes data.

---

Depending on storage type:

```text
Data Removed

OR

Data Preserved
```

---

# Volume Mounting

Volumes become available through:

```yaml
volumeMounts:
```

---

Example

```yaml
volumeMounts:

- name: app-storage

  mountPath: /data
```

---

Application can access:

```text
/data
```

inside container.

---

# Volumes and Containers

Example

```text
Pod
 │
 ├── Container
 │
 └── Volume
```

---

Container writes:

```text
/data/report.txt
```

---

File stored in volume.

---

# Multi-Container Pods

Volumes allow:

```text
Container Sharing
```

---

Example

```text
Container-1
      ↓

Shared Volume
      ↑

Container-2
```

---

Both containers access same files.

---

Very useful for:

```text
Sidecar Pattern
```

---

# Sidecar Pattern Overview

Example

```text
Application Container
```

writes logs.

---

Sidecar Container

```text
Reads Logs
```

and sends them to:

```text
ELK

Splunk

External Logging Systems
```

---

Both containers share same volume.

---

Architecture

```text
Application
      ↓

Shared Volume
      ↑

Log Shipper
```

---

# Common Kubernetes Volume Types

```text
emptyDir

hostPath

Persistent Volumes

ConfigMap Volumes

Secret Volumes
```

---

We will study each separately.

---

# Production Example

Frontend Application

Needs:

```text
Temporary Files

Cache

Logs
```

---

Volume attached.

---

Application writes data.

---

Volume provides:

```text
Shared Storage

Temporary Storage

Configuration Storage
```

---

# Benefits of Volumes

```text
Data Sharing

Application Flexibility

Storage Abstraction

Persistence Support

Configuration Management
```

---

# Common Mistakes

## Storing Everything In Container Filesystem

Problem

```text
Data Lost
```

when Pod recreated.

---

Better

```text
Use Volumes
```

---

## Ignoring Pod Lifecycle

Pods are temporary.

---

Always evaluate:

```text
Should Data Survive Pod Restart?
```

---

# Storage Decision Guide

Question

```text
Data Temporary?
```

---

Yes

```text
Use Ephemeral Volume
```

---

No

```text
Use Persistent Storage
```

---

Question

```text
Database Data?
```

---

Use

```text
Persistent Storage
```

---

Question

```text
Shared Logs?
```

---

Use

```text
Volume Sharing
```

---

# Troubleshooting

Check Pod

```bash
kubectl describe pod pod-name
```

---

Verify

```text
Volumes

Volume Mounts
```

---

Access Container

```bash
kubectl exec -it pod-name -- bash
```

---

Verify Mount

```bash
df -h
```

---

View Files

```bash
ls -l /data
```

---

# Common Interview Questions

## What is a Volume?

### Short Answer

A Volume is storage attached to a Pod.

### Detailed Explanation

Volumes allow containers to store and access data independently from the container filesystem.

### Follow-Up Questions

- Why do Pods need volumes?
- How are volumes mounted?

---

## Why Are Volumes Required?

### Short Answer

Containers are ephemeral and data may be lost when Pods are recreated.

### Detailed Explanation

Volumes provide shared or persistent storage for applications.

### Production Example

Application logs and uploaded files.

### Follow-Up Questions

- What is ephemeral storage?
- What is persistent storage?

---

## What Are The Types Of Kubernetes Storage?

### Short Answer

Ephemeral and Persistent Storage.

### Examples

```text
emptyDir

hostPath

Persistent Volumes

Storage Classes
```

### Follow-Up Questions

- Which storage survives Pod deletion?
- Which storage is temporary?

---

## Why Are Pods Considered Ephemeral?

### Short Answer

Pods can be recreated at any time.

### Detailed Explanation

Failures, upgrades, scaling, and scheduling events can replace Pods.

### Follow-Up Questions

- What happens to local Pod data?
- How do volumes solve this problem?

---

# Key Takeaways

```text
Pods are ephemeral.

Nodes can also be ephemeral.

Volumes provide storage to Pods.

Volumes allow data sharing between containers.

Storage can be ephemeral or persistent.

Volumes are mounted inside containers.

Sidecar patterns commonly use shared volumes.

Persistent storage is required for databases and business data.

Understanding Kubernetes storage is critical for production workloads.
```