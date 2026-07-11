# Pods and Multi-Container Pods

## Introduction

Containers are the fundamental runtime units.

However Kubernetes does not deploy containers directly.

Kubernetes deploys:

```text
Pods
```

---

Pod is the smallest deployable unit in Kubernetes.

Every application running in Kubernetes runs inside a Pod.

---

# What is a Pod?

A Pod is a wrapper around one or more containers.

---

Architecture

```text
Pod
 │
 └── Container
```

---

Example

```text
Frontend Pod
      │
      └── Frontend Container
```

---

Most Pods contain:

```text
One Container
```

---

But Kubernetes supports:

```text
Multiple Containers
```

inside the same Pod.

---

# Why Do We Need Pods?

Question:

```text
Why Not Run Containers Directly?
```

---

Kubernetes requires additional features:

```text
Networking

Storage

Lifecycle Management

Health Checks

Scheduling
```

---

Pods provide these capabilities.

---

Think of:

```text
Container = Application

Pod = Kubernetes Deployment Unit
```

---

# Pod Characteristics

Every Pod gets:

```text
IP Address

Hostname

Storage

Lifecycle
```

---

Containers inside a Pod share:

```text
Network

Storage
```

---

# Single Container Pod

Most common pattern.

---

Example

```text
Catalogue Pod
       │
       └── Catalogue Container
```

---

Architecture

```text
Pod
 │
 └── Container
```

---

Benefits

```text
Simple

Easy To Manage

Most Common Pattern
```

---

# Pod YAML

Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
  - name: nginx
    image: nginx
```

---

Create Pod

```bash
kubectl apply -f pod.yaml
```

---

Verify

```bash
kubectl get pods
```

---

Output

```text
nginx
```

---

# Pod Lifecycle

Pod goes through multiple phases.

---

Architecture

```text
Pending
   ↓

Running
   ↓

Succeeded

Failed

Unknown
```

---

# Pending

Pod accepted by Kubernetes.

---

But:

```text
Container Not Started
```

yet.

---

Reasons

```text
Image Download

Scheduling

Volume Mounting
```

---

# Running

Pod created successfully.

---

Container running.

---

Example

```bash
kubectl get pods
```

Output

```text
Running
```

---

# Succeeded

Container completed successfully.

---

Usually seen in:

```text
Jobs

Batch Processing
```

---

# Failed

Container terminated unexpectedly.

---

Reasons

```text
Application Error

Configuration Error

Missing Dependencies
```

---

# Unknown

Kubernetes cannot determine Pod state.

---

Usually caused by:

```text
Node Issues

Network Problems
```

---

# Pod Networking

Every Pod gets:

```text
Unique IP Address
```

---

Example

```text
Frontend Pod

10.244.1.5
```

---

```text
Catalogue Pod

10.244.2.10
```

---

Pods communicate using:

```text
Pod IP

Service
```

---

# Shared Network

Containers inside same Pod share:

```text
Same IP Address

Same Network Namespace
```

---

Example

```text
Pod
 │
 ├── App Container
 │
 └── Logging Container
```

---

Both use:

```text
localhost
```

for communication.

---

# Shared Storage

Containers inside a Pod can share volumes.

---

Architecture

```text
Pod
 │
 ├── App Container
 │
 ├── Sidecar Container
 │
 └── Shared Volume
```

---

Useful for:

```text
Logs

Temporary Data

Configuration Files
```

---

# Multi-Container Pods

A Pod can contain:

```text
Two

Three

Multiple Containers
```

---

Example

```text
Pod
 │
 ├── Application
 │
 └── Sidecar
```

---

Containers work together.

---

# Why Multi-Container Pods?

Useful when containers must:

```text
Share Network

Share Storage

Share Lifecycle
```

---

Common Patterns

```text
Sidecar

Ambassador

Adapter
```

---

# Sidecar Pattern

Most popular pattern.

---

Architecture

```text
Pod
 │
 ├── Main Application
 │
 └── Sidecar Container
```

---

Example

```text
Application

Fluent Bit
```

---

Application generates logs.

---

Sidecar collects logs.

---

Benefits

```text
Separation Of Responsibilities

Easy Monitoring

Reusable Components
```

---

# Sidecar Example

```text
Catalogue Application
         │
         └── Log Collector
```

---

Application focuses on business logic.

---

Sidecar handles logging.

---

# Ambassador Pattern

Acts as a proxy.

---

Architecture

```text
Pod
 │
 ├── Application
 │
 └── Proxy Container
```

---

Example

```text
Application
      ↓

Proxy
      ↓

Database
```

---

Benefits

```text
Connection Management

Security

Abstraction
```

---

# Adapter Pattern

Transforms data formats.

---

Architecture

```text
Application
      ↓

Adapter
      ↓

External System
```

---

Example

```text
Metrics Conversion

Log Transformation
```

---

Useful for integrations.

---

# Init Containers

Special containers that run:

```text
Before Main Containers
```

---

Flow

```text
Init Container
       ↓

Complete
       ↓

Application Container
```

---

Use Cases

```text
Database Checks

Configuration Validation

File Downloads
```

---

# Viewing Pods

List Pods

```bash
kubectl get pods
```

---

Detailed Output

```bash
kubectl get pods -o wide
```

---

Shows

```text
IP Address

Node

Status
```

---

# Describe Pod

Command

```bash
kubectl describe pod pod-name
```

---

Shows

```text
Events

Container Status

Image Information

Scheduling Details
```

---

Useful for troubleshooting.

---

# Pod Logs

View Logs

```bash
kubectl logs pod-name
```

---

Example

```bash
kubectl logs catalogue
```

---

Useful for:

```text
Application Troubleshooting
```

---

# Access Pod

Open Shell

```bash
kubectl exec -it pod-name -- bash
```

---

Example

```bash
kubectl exec -it catalogue -- bash
```

---

Useful for:

```text
Debugging

Configuration Validation

Network Testing
```

---

# ImagePullBackOff

## What is ImagePullBackOff?

Kubernetes unable to pull image.

---

Example

```text
ImagePullBackOff
```

---

Common Reasons

```text
Wrong Image Name

Wrong Tag

Private Registry Authentication Failure
```

---

Example

```yaml
image: nginxh
```

---

Image does not exist.

---

Troubleshoot

```bash
kubectl describe pod pod-name
```

---

Check:

```text
Events Section
```

---

# CrashLoopBackOff

## What is CrashLoopBackOff?

Container starts.

---

Then crashes repeatedly.

---

Kubernetes continuously tries:

```text
Restart

Restart

Restart
```

---

Result

```text
CrashLoopBackOff
```

---

Common Reasons

```text
Wrong Configuration

Application Error

Database Connectivity Failure

Missing Environment Variables
```

---

Troubleshoot

```bash
kubectl logs pod-name
```

---

And

```bash
kubectl describe pod pod-name
```

---

# Pod Troubleshooting Flow

```text
Pod Issue
    ↓

kubectl get pods
    ↓

Check Status
    ↓

kubectl describe pod
    ↓

Check Events
    ↓

kubectl logs
    ↓

kubectl exec
    ↓

Find Root Cause
```

---

# Production Example

Roboshop

```text
Catalogue Pod

User Pod

Cart Pod

Shipping Pod
```

---

Each service runs inside its own Pod.

---

Benefits

```text
Independent Scaling

Independent Updates

Fault Isolation
```

---

# Common Interview Questions

## What is a Pod?

### Short Answer

Pod is the smallest deployable unit in Kubernetes.

### Detailed Explanation

A Pod contains one or more containers that share networking and storage.

### Production Example

Catalogue application running inside a Pod.

### Follow-Up Questions

- Why do we need Pods?
- Can a Pod contain multiple containers?

---

## Why Do Containers Inside a Pod Share Network?

### Short Answer

To communicate efficiently using localhost.

### Detailed Explanation

Containers inside a Pod belong to the same network namespace.

### Production Example

Application container communicating with a logging sidecar.

### Follow-Up Questions

- Do containers get separate IPs?
- What resources are shared?

---

## What is ImagePullBackOff?

### Short Answer

Kubernetes cannot pull the container image.

### Detailed Explanation

Usually caused by wrong image names, wrong tags, or registry authentication problems.

### Production Example

Incorrect ECR image URL in deployment manifest.

### Follow-Up Questions

- How do you troubleshoot it?
- Which command shows events?

---

## What is CrashLoopBackOff?

### Short Answer

Container repeatedly crashes after starting.

### Detailed Explanation

Kubernetes continuously restarts the container because it cannot stay running.

### Production Example

Application failing due to incorrect database configuration.

### Follow-Up Questions

- How do you check logs?
- What are common causes?

---

# Key Takeaways

```text
Pod is the smallest deployable unit in Kubernetes.

Pods contain one or more containers.

Containers inside a Pod share network and storage.

Most Pods contain a single container.

Multi-container Pods are used for Sidecar, Ambassador, and Adapter patterns.

Init Containers run before application containers.

ImagePullBackOff indicates image download failure.

CrashLoopBackOff indicates repeated container crashes.

kubectl logs and kubectl describe are the most important troubleshooting commands.

Pods are the foundation of all Kubernetes workloads.
```