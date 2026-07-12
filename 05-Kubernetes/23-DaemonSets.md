# DaemonSets

## Introduction

So far we have learned:

```text
Pods

ReplicaSets

Deployments
```

---

Deployments focus on:

```text
Application Workloads
```

---

Examples

```text
Frontend

Catalogue

User

Cart

Shipping
```

---

Question

```text
What If We Need

One Pod On Every Node?
```

---

Examples

```text
Log Collection

Node Monitoring

Security Agents

Backup Agents
```

---

Kubernetes provides:

```text
DaemonSets
```

for these use cases.

---

# What is a DaemonSet?

A DaemonSet ensures:

```text
One Pod Runs

On Every Node
```

in the cluster.

---

Architecture

```text
Node-1
   ↓
DaemonSet Pod

Node-2
   ↓
DaemonSet Pod

Node-3
   ↓
DaemonSet Pod
```

---

Think of DaemonSet as:

```text
One Pod Per Node
```

---

# Why Do We Need DaemonSets?

Suppose we want to collect logs from:

```text
Node-1

Node-2

Node-3
```

---

Deployments cannot guarantee:

```text
One Pod Per Node
```

---

Example

Deployment

```text
3 Replicas
```

might schedule:

```text
Node-1 → 2 Pods

Node-2 → 1 Pod

Node-3 → 0 Pods
```

---

Not suitable.

---

Need

```text
Exactly One Pod
Per Node
```

---

Solution

```text
DaemonSet
```

---

# DaemonSet Architecture

```text
Cluster
 │
 ├── Node-1
 │      │
 │      └── DaemonSet Pod
 │
 ├── Node-2
 │      │
 │      └── DaemonSet Pod
 │
 └── Node-3
        │
        └── DaemonSet Pod
```

---

Every node receives:

```text
One Pod
```

automatically.

---

# DaemonSet Behavior

Suppose cluster contains:

```text
3 Nodes
```

---

DaemonSet creates:

```text
3 Pods
```

---

New Node Added

```text
Node-4
```

---

DaemonSet automatically creates:

```text
Pod On Node-4
```

---

Node Removed

```text
Node-2 Deleted
```

---

DaemonSet automatically removes:

```text
Corresponding Pod
```

---

# DaemonSet YAML

Example

```yaml
apiVersion: apps/v1

kind: DaemonSet

metadata:
  name: fluentbit

spec:

  selector:

    matchLabels:
      app: fluentbit

  template:

    metadata:

      labels:
        app: fluentbit

    spec:

      containers:

      - name: fluentbit

        image: fluent/fluent-bit
```

---

Create

```bash
kubectl apply -f daemonset.yaml
```

---

Verify

```bash
kubectl get daemonsets
```

---

# Viewing DaemonSets

List DaemonSets

```bash
kubectl get ds
```

---

Output

```text
fluentbit
```

---

Describe

```bash
kubectl describe ds fluentbit
```

---

Shows

```text
Desired Pods

Current Pods

Node Information
```

---

# DaemonSet Scheduling

Deployment

```text
Based On Replica Count
```

---

DaemonSet

```text
Based On Node Count
```

---

Example

Cluster

```text
5 Nodes
```

---

DaemonSet

```text
5 Pods
```

---

Cluster

```text
10 Nodes
```

---

DaemonSet

```text
10 Pods
```

---

# Common DaemonSet Use Cases

```text
Log Collection

Monitoring

Security

Storage

Networking
```

---

# Log Collection

Most common DaemonSet use case.

---

Every node generates:

```text
Container Logs

System Logs

Application Logs
```

---

Need to collect:

```text
All Logs
```

---

DaemonSet runs on every node.

---

Reads logs.

---

Sends logs to:

```text
ELK

Splunk

Cloud Logging
```

---

# Fluent Bit Architecture

```text
Node Logs
      ↓

hostPath
      ↓

Fluent Bit DaemonSet
      ↓

Elasticsearch
```

---

Every node runs:

```text
Fluent Bit Pod
```

---

# Filebeat Architecture

```text
Node Logs
      ↓

hostPath
      ↓

Filebeat DaemonSet
      ↓

ELK
```

---

Another common implementation.

---

# Monitoring Nodes

Nodes generate:

```text
CPU Metrics

Memory Metrics

Disk Metrics

Network Metrics
```

---

Need monitoring.

---

DaemonSet deployed.

---

Collects metrics from every node.

---

# Node Exporter

Common monitoring DaemonSet.

---

Purpose

```text
Collect Node Metrics
```

---

Architecture

```text
Node
      ↓

Node Exporter
      ↓

Prometheus
```

---

Metrics

```text
CPU

Memory

Disk

Network
```

---

# Security Agents

Security tools often run as DaemonSets.

---

Examples

```text
Falco

Security Scanners

Runtime Protection Agents
```

---

Need visibility into:

```text
Every Node
```

---

DaemonSet is ideal.

---

# DaemonSet and hostPath

DaemonSets frequently use:

```text
hostPath
```

---

Reason

```text
Need Access
To Node Files
```

---

Example

```yaml
volumes:

- name: host-logs

  hostPath:

    path: /var/log
```

---

Container Mount

```yaml
volumeMounts:

- name: host-logs

  mountPath: /host-logs

  readOnly: true
```

---

Result

```text
Read Node Logs
```

---

# Why ReadOnly?

Good Practice

```yaml
readOnly: true
```

---

Benefits

```text
Improved Security

Prevent Accidental Changes

Safer Access
```

---

# Complete DaemonSet Example

```yaml
apiVersion: apps/v1

kind: DaemonSet

metadata:
  name: node-logger

spec:

  selector:

    matchLabels:
      app: node-logger

  template:

    metadata:

      labels:
        app: node-logger

    spec:

      containers:

      - name: logger

        image: busybox

        volumeMounts:

        - name: logs

          mountPath: /host-logs

          readOnly: true

      volumes:

      - name: logs

        hostPath:

          path: /var/log
```

---

# DaemonSet vs Deployment

| Feature | Deployment | DaemonSet |
|----------|------------|------------|
| Purpose | Application Deployment | Node-Level Services |
| Pod Count | Based On Replicas | Based On Nodes |
| Scaling | Manual/HPA | Automatic Per Node |
| One Pod Per Node | No | Yes |
| Log Collection | Not Ideal | Excellent |
| Monitoring | Not Ideal | Excellent |

---

# Real Production Example

EKS Cluster

```text
5 Worker Nodes
```

---

DaemonSet

```text
Fluent Bit
```

---

Result

```text
5 Fluent Bit Pods
```

---

Each Pod

```text
Reads Local Logs
```

---

Ships Logs To

```text
ELK Stack
```

---

# DaemonSet Lifecycle

```text
DaemonSet Created
         ↓

One Pod Per Node
         ↓

New Node Added
         ↓

New Pod Created
         ↓

Node Removed
         ↓

Pod Removed
```

---

# Troubleshooting

Check DaemonSets

```bash
kubectl get ds
```

---

Describe

```bash
kubectl describe ds daemonset-name
```

---

Check Pods

```bash
kubectl get pods -o wide
```

---

Verify

```text
One Pod Per Node
```

---

Check Nodes

```bash
kubectl get nodes
```

---

# Common Problems

## Missing Pods

Expected

```text
5 Nodes

5 Pods
```

---

Actual

```text
5 Nodes

3 Pods
```

---

Check

```bash
kubectl describe ds
```

---

Possible Causes

```text
Node Taints

Scheduling Restrictions

Resource Constraints
```

---

## Cannot Read Logs

Check

```text
hostPath Mount

Permissions

ReadOnly Settings
```

---

# Common Interview Questions

## What is a DaemonSet?

### Short Answer

A DaemonSet ensures one Pod runs on every node in a cluster.

### Detailed Explanation

DaemonSets are used for node-level services such as monitoring, logging, and security.

### Production Example

Fluent Bit collecting logs from all nodes.

---

## Why Do We Use DaemonSets?

### Short Answer

To run a specific Pod on every node.

### Production Example

Node monitoring using Node Exporter.

### Follow-Up Questions

- Why not use Deployments?
- What happens when a new node is added?

---

## What Happens When a New Node Joins the Cluster?

### Short Answer

DaemonSet automatically schedules a Pod on the new node.

### Follow-Up Questions

- Does manual scaling require?
- How many Pods are created?

---

## Why Are DaemonSets Commonly Used With hostPath?

### Short Answer

Because they need access to node files and logs.

### Production Example

Reading /var/log and sending logs to ELK.

### Follow-Up Questions

- Why should hostPath be read-only?
- What are the security concerns?

---

# Key Takeaways

```text
DaemonSets ensure one Pod runs on every node.

DaemonSets automatically adjust when nodes are added or removed.

DaemonSets are commonly used for logging, monitoring, and security.

Fluent Bit and Filebeat are common logging DaemonSets.

Node Exporter is a common monitoring DaemonSet.

DaemonSets frequently use hostPath to access node files.

hostPath should be mounted as read-only whenever possible.

DaemonSets are node-focused workloads, unlike Deployments which are application-focused.
```