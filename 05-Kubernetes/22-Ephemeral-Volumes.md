# Ephemeral Volumes

## Introduction

Applications often need storage for:

```text
Logs

Temporary Files

Cache

Shared Data Between Containers
```

---

Not all data needs to be stored permanently.

---

Examples

```text
Application Logs

Temporary Reports

Cache Files

Session Data
```

---

Kubernetes provides:

```text
Ephemeral Volumes
```

for temporary storage.

---

# What is Ephemeral Storage?

Ephemeral means:

```text
Temporary
```

---

Data exists only while:

```text
Pod Exists
```

---

When Pod is deleted:

```text
Data Is Deleted
```

---

Think of Ephemeral Storage as:

```text
Temporary Workspace
For Pods
```

---

# Characteristics of Ephemeral Storage

```text
Temporary

Fast

Simple

Pod Dependent
```

---

Not suitable for:

```text
Databases

Business Data

Critical Files
```

---

Suitable for:

```text
Logs

Caching

Scratch Space

Container Communication
```

---

# Types of Ephemeral Volumes

Most commonly used:

```text
emptyDir

hostPath
```

---

# emptyDir

## What is emptyDir?

emptyDir is the default temporary storage mechanism.

---

Created when:

```text
Pod Starts
```

---

Deleted when:

```text
Pod Deleted
```

---

Lifecycle

```text
Pod Created
      ↓

emptyDir Created
      ↓

Containers Use Storage
      ↓

Pod Deleted
      ↓

Storage Deleted
```

---

# Why Use emptyDir?

Useful when:

```text
Multiple Containers
Need Shared Storage
```

---

Architecture

```text
Pod
 │
 ├── App Container
 │
 ├── Sidecar Container
 │
 └── emptyDir Volume
```

---

Both containers can access:

```text
Same Files
```

---

# emptyDir Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: shared-storage

spec:

  containers:

  - name: app

    image: nginx

    volumeMounts:

    - name: shared-data

      mountPath: /data

  volumes:

  - name: shared-data

    emptyDir: {}
```

---

Create Pod

```bash
kubectl apply -f pod.yaml
```

---

# Multi Container Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: sidecar-demo

spec:

  containers:

  - name: application

    image: nginx

    volumeMounts:

    - name: shared-logs

      mountPath: /logs

  - name: log-reader

    image: busybox

    volumeMounts:

    - name: shared-logs

      mountPath: /logs

  volumes:

  - name: shared-logs

    emptyDir: {}
```

---

Result

```text
Both Containers

Read And Write

Same Storage
```

---

# Sidecar Pattern

One of the most common uses of emptyDir.

---

Architecture

```text
Application Container
          │
          │ Writes Logs
          ▼

      emptyDir

          ▲
          │ Reads Logs
          │

Sidecar Container
```

---

Application

```text
Generates Logs
```

---

Sidecar

```text
Collects Logs
```

and sends them to:

```text
ELK

Splunk

Cloud Logging
```

---

# Real Example

Application Container

```text
Writes

/app/logs/application.log
```

---

Sidecar Container

```text
Reads Same File
```

from shared volume.

---

Ships logs to:

```text
Elasticsearch
```

---

# Benefits of emptyDir

```text
Simple

Fast

Shared Storage

No External Dependency
```

---

# Limitations of emptyDir

```text
Data Lost On Pod Deletion

Not Persistent

Not Suitable For Databases
```

---

# hostPath

## What is hostPath?

hostPath allows Pods to access:

```text
Node Filesystem
```

directly.

---

Architecture

```text
Node
 │
 ├── /var/log
 │
 └── Pod
        │
        ▼
     hostPath
```

---

Container accesses:

```text
Actual Files
From Node
```

---

# hostPath Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: hostpath-demo

spec:

  containers:

  - name: app

    image: nginx

    volumeMounts:

    - name: host-storage

      mountPath: /host-logs

  volumes:

  - name: host-storage

    hostPath:

      path: /var/log

      type: Directory
```

---

Result

```text
Container Can Access

Node Logs
```

---

# Why Use hostPath?

Useful when applications need:

```text
Node Logs

Node Metrics

Host Files
```

---

Common Examples

```text
Log Collectors

Monitoring Agents

Security Agents
```

---

# hostPath and DaemonSets

Very common combination.

---

Example

```text
Fluent Bit

Filebeat

Node Monitoring Agents
```

---

Architecture

```text
Node Logs
      ↓

hostPath
      ↓

DaemonSet Pod
      ↓

ELK
```

---

# Log Shipping Example

Node stores logs in:

```text
/var/log
```

---

DaemonSet Pod

```text
Reads Logs
```

using:

```text
hostPath
```

---

Ships logs to:

```text
ELK Stack
```

---

# hostPath Persistence

Unlike emptyDir:

```text
Data Survives Pod Restart
```

---

As long as:

```text
Node Exists
```

---

However:

```text
Node Failure
      ↓

Data Lost
```

---

Still considered:

```text
Ephemeral Storage
```

---

# Security Risks of hostPath

hostPath gives access to:

```text
Underlying Node
```

---

Potential Risks

```text
Sensitive Files

System Files

Node Compromise
```

---

Example

```yaml
hostPath:
  path: /
```

---

Extremely dangerous.

---

# Best Practice

Use:

```text
Read Only Access
```

whenever possible.

---

Example

```yaml
volumeMounts:

- name: host-storage

  mountPath: /host-logs

  readOnly: true
```

---

Recommended for:

```text
Log Collection

Monitoring
```

---

# emptyDir vs hostPath

| Feature | emptyDir | hostPath |
|----------|----------|----------|
| Scope | Pod | Node |
| Shared Between Containers | Yes | Yes |
| Access Node Files | No | Yes |
| Security Risk | Low | High |
| Data Survives Pod Restart | No | Yes |
| Data Survives Node Failure | No | No |

---

# Production Example

Logging Architecture

```text
Application
      ↓

Node Logs
      ↓

hostPath
      ↓

Fluent Bit DaemonSet
      ↓

ELK Stack
```

---

Monitoring Architecture

```text
Node Metrics
      ↓

hostPath
      ↓

Monitoring Agent
      ↓

Prometheus
```

---

# Troubleshooting

View Pod

```bash
kubectl describe pod pod-name
```

---

Verify Volumes

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

Check Mounted Files

```bash
ls -l
```

---

Verify Host Files

```bash
cat /host-logs/file.log
```

---

# Common Interview Questions

## What is emptyDir?

### Short Answer

A temporary volume created when a Pod starts and deleted when the Pod is removed.

### Production Example

Sharing logs between application and sidecar containers.

### Follow-Up Questions

- Does emptyDir survive Pod deletion?
- Can multiple containers use emptyDir?

---

## What is hostPath?

### Short Answer

A volume that mounts files or directories from the Kubernetes node.

### Production Example

Reading node logs using Fluent Bit.

### Follow-Up Questions

- Why is hostPath risky?
- When should hostPath be used?

---

## What is the Sidecar Pattern?

### Short Answer

A secondary container running alongside the main application container.

### Detailed Explanation

Both containers share a volume. The sidecar performs supporting tasks such as log shipping or monitoring.

### Production Example

Sending logs to ELK using a sidecar container.

### Follow-Up Questions

- Why is emptyDir commonly used with sidecars?
- What are common sidecar use cases?

---

## Why is hostPath considered insecure?

### Short Answer

It exposes node filesystems to containers.

### Detailed Explanation

Containers may gain access to sensitive host files if permissions are not controlled properly.

### Follow-Up Questions

- How can hostPath be secured?
- When should readOnly mounts be used?

---

# Key Takeaways

```text
Ephemeral storage is temporary storage.

emptyDir exists only for the lifetime of a Pod.

emptyDir is commonly used for container-to-container data sharing.

hostPath allows Pods to access node filesystems.

hostPath is commonly used by monitoring and logging solutions.

hostPath should be mounted as read-only whenever possible.

Sidecar patterns commonly use shared volumes.

Neither emptyDir nor hostPath is suitable for database storage.

Understanding ephemeral storage is important for production Kubernetes environments.
```