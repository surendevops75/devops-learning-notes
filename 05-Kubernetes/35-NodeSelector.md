# Node Selector

## Introduction

When a Pod is created in Kubernetes:

```bash
kubectl apply -f pod.yaml
```

---

The Pod is not immediately placed on a node.

---

Kubernetes Scheduler decides:

```text
Which Node

Should Run

The Pod
```

---

By default Scheduler selects:

```text
Any Suitable Node
```

based on:

```text
CPU

Memory

Resource Availability
```

---

Sometimes we want:

```text
Specific Pods

On Specific Nodes
```

---

Examples

```text
Frontend Pods

On Application Nodes

AI Workloads

On GPU Nodes

Database Pods

On Dedicated Nodes
```

---

For this Kubernetes provides:

```text
Node Selector
```

---

# Kubernetes Scheduling Flow

When we apply a manifest:

```bash
kubectl apply -f pod.yaml
```

---

Flow

```text
kubectl
    ↓

API Server
    ↓

Scheduler
    ↓

Select Suitable Node
    ↓

Kubelet
    ↓

Pod Running
```

---

Scheduler is responsible for:

```text
Node Selection
```

---

# What is Node Selector?

Node Selector is the simplest scheduling mechanism in Kubernetes.

---

It allows us to:

```text
Schedule Pods

On Specific Nodes

Using Labels
```

---

Think of it as:

```text
Pod Requirement
        ↓

Node Labels
        ↓

Scheduler Match
```

---

# Why Do We Need Node Selector?

Suppose a cluster has:

```text
Node-1

Node-2

Node-3
```

---

All nodes are healthy.

---

By default Scheduler can place Pods on:

```text
Any Node
```

---

But suppose:

```text
Node-1

Has SSD Storage
```

---

We want:

```text
Database Pods

Only On SSD Nodes
```

---

Node Selector solves this.

---

# Node Labels

Node Selector works using:

```text
Node Labels
```

---

Example Label

```text
project=roboshop
```

---

Or

```text
node-type=database
```

---

Or

```text
hardware=gpu
```

---

# View Node Labels

Command

```bash
kubectl get nodes --show-labels
```

---

Output

```text
node-1

project=roboshop

hardware=gpu
```

---

# Add Label To Node

Example

```bash
kubectl label nodes node-1 project=roboshop
```

---

Verify

```bash
kubectl get nodes --show-labels
```

---

# Node Selector Architecture

```text
Node Labels
      │

      ├── node-1
      │      project=roboshop
      │
      ├── node-2
      │      project=test
      │
      └── node-3
             project=dev


Pod
 │
 └── nodeSelector:
        project=roboshop
```

---

Scheduler checks:

```text
Matching Labels
```

---

Pod gets scheduled on:

```text
node-1
```

---

# Node Selector Example

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  nodeSelector:

    project: roboshop

  containers:

  - name: nginx

    image: nginx
```

---

Meaning

```text
Schedule Pod

Only On Nodes

Having Label

project=roboshop
```

---

# Scheduler Decision Process

Scheduler receives:

```text
Pod Request
```

---

Checks

```text
Available Nodes
```

---

Looks for

```text
Matching Labels
```

---

Selects

```text
Matching Node
```

---

Schedules Pod.

---

# Example

Cluster

```text
node-1

project=roboshop
```

---

```text
node-2

project=test
```

---

Pod

```yaml
nodeSelector:

  project: roboshop
```

---

Result

```text
Pod Scheduled

On node-1
```

---

# What If No Matching Node Exists?

Example

```yaml
nodeSelector:

  project: production
```

---

Cluster

```text
No Node

Has Label

project=production
```

---

Result

```text
Pod Remains Pending
```

---

Scheduler cannot place Pod.

---

# Common Use Cases

## GPU Workloads

Node Label

```text
hardware=gpu
```

---

Pod

```yaml
nodeSelector:

  hardware: gpu
```

---

Used for:

```text
AI

ML

Deep Learning
```

---

## Database Nodes

Node Label

```text
node-type=database
```

---

Pod

```yaml
nodeSelector:

  node-type: database
```

---

Used for:

```text
MongoDB

MySQL

PostgreSQL
```

---

## Application Nodes

Node Label

```text
project=roboshop
```

---

Pod

```yaml
nodeSelector:

  project: roboshop
```

---

# Node Selector Limitations

Node Selector supports:

```text
Exact Match Only
```

---

Cannot do:

```text
OR Logic

AND Logic

Complex Rules
```

---

Example Not Possible

```text
project=roboshop

OR

project=ecommerce
```

---

For advanced scheduling Kubernetes provides:

```text
Node Affinity
```

---

# Production Example

Cluster

```text
Node-1

project=roboshop
```

---

```text
Node-2

project=dev
```

---

MongoDB Pod

```yaml
nodeSelector:

  project: roboshop
```

---

Result

```text
MongoDB Runs

Only On Roboshop Nodes
```

---

# Common Problems

## Pod Pending

Cause

```text
No Matching Label
```

---

Check

```bash
kubectl get nodes --show-labels
```

---

Verify

```yaml
nodeSelector
```

---

## Wrong Node Selection

Cause

```text
Incorrect Labels
```

---

Verify

```bash
kubectl describe node node-name
```

---

# Troubleshooting

View Pod

```bash
kubectl get pods -o wide
```

---

Check Node

```bash
kubectl describe pod pod-name
```

---

View Labels

```bash
kubectl get nodes --show-labels
```

---

Describe Node

```bash
kubectl describe node node-name
```

---

# Common Interview Questions

## What is Node Selector?

### Short Answer

Node Selector is a Kubernetes scheduling feature that places Pods on nodes with matching labels.

### Detailed Explanation

Node Selector allows Pods to request specific nodes using label-based matching. The Scheduler only considers nodes that contain the specified labels.

### Production Example

MongoDB Pods are scheduled only on nodes labeled:

```text
project=roboshop
```

### Follow-Up Questions

- Does Node Selector use labels?
- What happens if no node matches?
- Is Node Selector mandatory?

---

## How Does Node Selector Work?

### Short Answer

Scheduler compares Pod nodeSelector values with node labels.

### Detailed Explanation

When a Pod is created, the Scheduler searches for nodes whose labels match the requested nodeSelector values.

### Production Example

A Pod requests:

```yaml
project: roboshop
```

and gets scheduled on a node with the same label.

### Follow-Up Questions

- Which component performs scheduling?
- Where are labels stored?
- How can labels be viewed?

---

## What Happens If No Node Matches?

### Short Answer

The Pod remains in Pending state.

### Detailed Explanation

Scheduler cannot find a suitable node and continuously retries scheduling until a matching node becomes available.

### Production Example

A Pod requesting:

```text
project=production
```

remains Pending because no node has that label.

### Follow-Up Questions

- How do you troubleshoot Pending Pods?
- Which command shows node labels?
- Can Scheduler ignore nodeSelector?

---

## What Are The Limitations Of Node Selector?

### Short Answer

Node Selector supports only exact label matching.

### Detailed Explanation

Node Selector cannot perform complex scheduling logic such as AND, OR, preference-based scheduling, or anti-affinity rules.

### Production Example

A workload needing GPU OR High-Memory nodes cannot be expressed using only Node Selector.

### Follow-Up Questions

- What solves this limitation?
- What is Node Affinity?
- What is Pod Affinity?

---

## Difference Between Node Selector and Node Affinity?

### Short Answer

Node Selector supports simple matching, while Node Affinity supports advanced scheduling rules.

### Detailed Explanation

Node Affinity adds support for operators, preferences, AND/OR logic, and advanced scheduling constraints.

### Production Example

Node Selector schedules Pods to GPU nodes, while Node Affinity can prefer GPU nodes but allow fallback options.

### Follow-Up Questions

- Which is more flexible?
- Which one supports OR logic?
- Which one is recommended for production?

---

# Key Takeaways

```text
Node Selector is the simplest Kubernetes scheduling mechanism.

Node Selector uses node labels.

Scheduler matches Pod requirements with node labels.

Pods are scheduled only on matching nodes.

No matching node results in Pending Pods.

Node Selector supports only exact matches.

Node Selector is commonly used for GPU, database, and dedicated nodes.

Advanced scheduling requirements use Node Affinity.
```