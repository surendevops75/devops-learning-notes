# Taints and Tolerations

## Introduction

By default Kubernetes Scheduler places Pods on:

```text
Any Suitable Node
```

based on:

```text
CPU

Memory

Available Resources
```

---

Sometimes we want to:

```text
Protect Nodes

Reserve Nodes

Create Dedicated Nodes
```

for specific workloads.

---

Examples

```text
GPU Nodes

Database Nodes

High Priority Workloads

Dedicated Project Nodes
```

---

For this Kubernetes provides:

```text
Taints

Tolerations
```

---

# What is a Taint?

Think of a taint as:

```text
Paint On A Node
```

---

The paint says:

```text
Do Not Schedule Pods Here
```

unless they explicitly accept it.

---

A taint is applied on:

```text
Node
```

---

Purpose

```text
Restrict Scheduling
```

---

# What is a Toleration?

A toleration is:

```text
Acceptance Of A Taint
```

---

Applied on:

```text
Pod
```

---

Purpose

```text
Allow Pod To Be Scheduled
```

on tainted nodes.

---

# Taint and Toleration Relationship

```text
Node
 │
 └── Taint
         │

         ▼

      Pod
         │
         └── Toleration
```

---

Without toleration

```text
Pod Rejected
```

---

With toleration

```text
Pod Allowed
```

---

# Why Do We Need Taints?

Suppose a cluster has:

```text
10 Worker Nodes
```

---

One node contains:

```text
GPU Hardware
```

---

We don't want:

```text
Frontend

Nginx

Utility Pods
```

using expensive GPU nodes.

---

We reserve the node using:

```text
Taints
```

---

# Common Use Cases

## GPU Nodes

```text
AI

Machine Learning

Deep Learning
```

---

Only GPU applications should run.

---

## Database Nodes

Databases may accept traffic only from:

```text
Specific IP Addresses
```

---

Dedicated nodes simplify firewall rules.

---

## High Priority Nodes

```text
Production Workloads
```

only.

---

## Dedicated Project Nodes

Example

```text
Roboshop
```

workloads only.

---

# Taint Syntax

```bash
kubectl taint nodes node-name key=value:effect
```

---

Example

```bash
kubectl taint nodes node-1 project=roboshop:NoSchedule
```

---

Meaning

```text
Do Not Schedule New Pods
```

on node-1.

---

# View Taints

```bash
kubectl describe node node-1
```

---

Look for:

```text
Taints
```

section.

---

# Toleration Example

```yaml
tolerations:

- key: "project"

  operator: "Equal"

  value: "roboshop"

  effect: "NoSchedule"
```

---

Meaning

```text
Pod Accepts

project=roboshop
```

taint.

---

# Important Concept

Many people misunderstand tolerations.

---

Toleration means:

```text
Pod Is Allowed
```

---

It does NOT mean:

```text
Pod Is Guaranteed
```

to run on that node.

---

Scheduler may still choose:

```text
Another Suitable Node
```

---

# Scheduler Decision Process

Without Toleration

```text
Scheduler
     ↓

Tainted Node

Rejected
```

---

With Toleration

```text
Scheduler
     ↓

Tainted Node

Allowed
```

---

Still scheduler decides based on:

```text
Resources

Policies

Availability
```

---

# Taint Effects

Kubernetes supports:

```text
NoSchedule

PreferNoSchedule

NoExecute
```

---

# NoSchedule

Most commonly used.

---

Meaning

```text
Do Not Schedule
New Pods
```

---

Existing Pods

```text
Remain Running
```

---

Example

Node already contains:

```text
Pod-A

Pod-B

Pod-C
```

---

Apply

```text
project=roboshop:NoSchedule
```

---

Result

```text
Existing Pods Stay

New Pods Blocked
```

---

# NoExecute

Strictest effect.

---

Meaning

```text
Do Not Schedule New Pods

Evict Existing Pods
```

---

Example

Node contains:

```text
Pod-A

Pod-B

Pod-C
```

---

Apply

```text
project=roboshop:NoExecute
```

---

Result

```text
Existing Pods Removed

New Pods Blocked
```

---

# PreferNoSchedule

Soft rule.

---

Meaning

```text
Avoid Scheduling
```

if possible.

---

Scheduler preference:

```text
Try Other Nodes First
```

---

Example

Cluster

```text
Node-1 Tainted

Node-2 Available

Node-3 Available
```

---

Scheduler chooses:

```text
Node-2

or

Node-3
```

---

If resources unavailable:

```text
May Use Tainted Node
```

---

# Example Cluster

```text
Node-1

project=roboshop:NoSchedule
```

---

```text
Node-2

No Taint
```

---

```text
Node-3

No Taint
```

---

Pod Without Toleration

```text
Node-1 Rejected
```

---

Pod With Toleration

```text
Node-1 Allowed
```

---

# Creating a Taint

```bash
kubectl taint nodes worker-1 project=roboshop:NoSchedule
```

---

Verify

```bash
kubectl describe node worker-1
```

---

# Removing a Taint

```bash
kubectl taint nodes worker-1 project=roboshop:NoSchedule-
```

---

Notice

```text
Trailing Hyphen
```

---

Removes the taint.

---

# Complete Example

## Taint

```bash
kubectl taint nodes worker-1 project=roboshop:NoSchedule
```

---

## Pod

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx

spec:

  tolerations:

  - key: "project"

    operator: "Equal"

    value: "roboshop"

    effect: "NoSchedule"

  containers:

  - name: nginx

    image: nginx
```

---

Result

```text
Pod Allowed
```

on worker-1.

---

# Production Example

Dedicated Database Nodes

---

Node

```text
database=true:NoSchedule
```

---

Only

```text
MongoDB

MySQL

PostgreSQL
```

Pods receive matching tolerations.

---

Benefits

```text
Predictable Placement

Security

Isolation
```

---

# Taints vs Node Selector

## Node Selector

```text
Attracts Pods
```

---

Example

```yaml
nodeSelector:

  project: roboshop
```

---

## Taints

```text
Repels Pods
```

---

Example

```text
project=roboshop:NoSchedule
```

---

Easy interview answer:

```text
NodeSelector Pulls

Taints Push Away
```

---

# Taints and Scheduler

Scheduler checks:

```text
Untainted Nodes
```

first.

---

If Pod contains:

```text
Matching Toleration
```

---

Scheduler also considers:

```text
Tainted Nodes
```

---

Then resource evaluation occurs.

---

# Common Problems

## Pod Pending

Cause

```text
No Matching Toleration
```

---

Check

```bash
kubectl describe pod pod-name
```

---

Look for:

```text
Taint Errors
```

---

## Pod Not Running On Tainted Node

Cause

```text
Toleration Present
```

but

```text
Scheduler Selected
Another Node
```

---

This is expected behavior.

---

## Pods Suddenly Removed

Cause

```text
NoExecute Taint
```

---

Verify

```bash
kubectl describe node node-name
```

---

# Troubleshooting

Check Node Taints

```bash
kubectl describe node node-name
```

---

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

Check Scheduling Errors

```text
Untolerated Taint
```

---

# Common Interview Questions

## What is a Taint?

### Short Answer

A taint is a restriction applied to a node that prevents Pods from being scheduled there.

### Detailed Explanation

Taints are applied to nodes and instruct the Scheduler to avoid placing Pods unless they explicitly tolerate the taint.

### Production Example

GPU nodes are tainted so only AI workloads can run on them.

### Follow-Up Questions

- Where is a taint applied?
- Can a taint block all Pods?
- How do you view taints?

---

## What is a Toleration?

### Short Answer

A toleration allows a Pod to be scheduled onto a tainted node.

### Detailed Explanation

A toleration does not force scheduling. It simply removes the restriction imposed by the taint.

### Production Example

MongoDB Pods tolerate database node taints.

### Follow-Up Questions

- Does toleration guarantee scheduling?
- Where is toleration defined?
- Can a Pod have multiple tolerations?

---

## Difference Between NoSchedule and NoExecute?

### Short Answer

NoSchedule blocks new Pods, while NoExecute blocks new Pods and removes existing Pods.

### Detailed Explanation

NoSchedule affects future scheduling. NoExecute affects both future and currently running Pods.

### Production Example

A maintenance node receives NoExecute and existing workloads are evicted.

### Follow-Up Questions

- What happens to existing Pods with NoSchedule?
- Which effect is the strictest?
- When is NoExecute useful?

---

## What is PreferNoSchedule?

### Short Answer

A soft scheduling rule that asks the Scheduler to avoid the node if possible.

### Detailed Explanation

Scheduler prefers other nodes but may still schedule Pods on the tainted node when necessary.

### Production Example

A backup node is marked PreferNoSchedule and used only when other nodes are full.

### Follow-Up Questions

- Is PreferNoSchedule mandatory?
- Can Scheduler ignore it?
- When should it be used?

---

## Difference Between Node Selector and Taints?

### Short Answer

Node Selector attracts Pods to nodes, while Taints repel Pods from nodes.

### Detailed Explanation

Node Selector defines where Pods should go. Taints define where Pods should not go unless tolerated.

### Production Example

Frontend Pods use Node Selector for application nodes, while database nodes are protected using taints.

### Follow-Up Questions

- Can both be used together?
- Which is applied to nodes?
- Which is applied to Pods?

---

# Key Takeaways

```text
Taints are applied to nodes.

Tolerations are applied to Pods.

Taints restrict scheduling.

Tolerations allow scheduling.

Toleration does not guarantee placement.

NoSchedule blocks future Pods.

NoExecute blocks future Pods and removes existing Pods.

PreferNoSchedule is a soft restriction.

Taints are commonly used for GPU, database, and dedicated nodes.

Node Selector attracts Pods, Taints repel Pods.
```