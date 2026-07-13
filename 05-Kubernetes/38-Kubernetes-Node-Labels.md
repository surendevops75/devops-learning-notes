# Kubernetes Node Labels

## Introduction

Earlier we learned:

```text
NodeSelector

Node Affinity

Pod Affinity

Pod Anti-Affinity

Taints and Tolerations
```

---

Most scheduling decisions in Kubernetes are based on:

```text
Labels
```

---

Labels help Kubernetes identify:

```text
Nodes

Pods

Deployments

Services
```

---

In this section we focus on:

```text
Node Labels
```

---

# What are Node Labels?

Node labels are:

```text
Key Value Pairs

Attached To Nodes
```

---

Example

```text
project=roboshop

env=prod

hardware=gpu
```

---

Scheduler uses these labels for:

```text
Node Selection

Affinity Rules

Workload Placement
```

---

# Why Do We Need Node Labels?

Suppose cluster contains:

```text
Node-1

Node-2

Node-3
```

---

Some nodes may be:

```text
Production Nodes

GPU Nodes

Database Nodes

Spot Instances
```

---

Labels help identify them.

---

Example

```text
project=roboshop

environment=prod

hardware=gpu
```

---

# Node Label Architecture

```text
Node-1

project=roboshop

env=prod
```

---

```text
Node-2

project=amazon

env=dev
```

---

```text
Node-3

hardware=gpu
```

---

Scheduler reads labels and decides:

```text
Where Pods Should Run
```

---

# Viewing Node Labels

Command

```bash
kubectl get nodes --show-labels
```

---

Example

```text
NAME

worker-1

worker-2
```

---

Labels

```text
project=roboshop

env=prod

hardware=gpu
```

---

# Built-In Kubernetes Labels

Kubernetes automatically adds labels.

---

# Operating System

```text
kubernetes.io/os=linux
```

---

Used for:

```text
Linux Scheduling
```

---

# CPU Architecture

```text
kubernetes.io/arch=amd64
```

---

Examples

```text
amd64

arm64
```

---

# Hostname

```text
kubernetes.io/hostname
```

---

Example

```text
kubernetes.io/hostname=ip-192-168-11-226.ec2.internal
```

---

Represents:

```text
Node Name
```

---

# Region

```text
topology.kubernetes.io/region
```

---

Example

```text
topology.kubernetes.io/region=us-east-1
```

---

# Availability Zone

```text
topology.kubernetes.io/zone
```

---

Example

```text
topology.kubernetes.io/zone=us-east-1b
```

---

Useful for:

```text
Storage

High Availability

Disaster Recovery
```

---

# Instance Type

```text
node.kubernetes.io/instance-type
```

---

Example

```text
node.kubernetes.io/instance-type=c3.large
```

---

Useful for:

```text
Scheduling High Resource Workloads
```

---

# EKS Labels

Amazon EKS automatically adds labels.

---

# Capacity Type

```text
eks.amazonaws.com/capacityType=SPOT
```

---

or

```text
eks.amazonaws.com/capacityType=ON_DEMAND
```

---

Useful for:

```text
Cost Optimization
```

---

# Node Group

```text
eks.amazonaws.com/nodegroup
```

---

Example

```text
eks.amazonaws.com/nodegroup=roboshop-dev
```

---

Identifies:

```text
Managed Node Group
```

---

# Cluster Name

```text
alpha.eksctl.io/cluster-name
```

---

Example

```text
alpha.eksctl.io/cluster-name=roboshop-dev
```

---

Useful for:

```text
Multi Cluster Administration
```

---

# Viewing Specific Node Labels

Command

```bash
kubectl describe node node-name
```

---

Example

```bash
kubectl describe node ip-192-168-11-226.ec2.internal
```

---

Shows

```text
Labels

Taints

Capacity

Allocatable Resources
```

---

# Creating Custom Labels

Command

```bash
kubectl label nodes node-name project=roboshop
```

---

Example

```bash
kubectl label nodes ip-192-168-11-226.ec2.internal project=roboshop
```

---

Verify

```bash
kubectl get nodes --show-labels
```

---

# Another Example

```bash
kubectl label nodes ip-192-168-21-7.ec2.internal project=amazon
```

---

Result

```text
Node-1

project=roboshop
```

---

```text
Node-2

project=amazon
```

---

# Updating Labels

Existing

```text
project=roboshop
```

---

Update

```bash
kubectl label nodes node-name project=ecommerce --overwrite
```

---

Verify

```bash
kubectl get nodes --show-labels
```

---

# Removing Labels

Command

```bash
kubectl label nodes node-name project-
```

---

Notice

```text
Trailing Hyphen
```

---

Removes label.

---

# Using Labels With NodeSelector

Node

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

Result

```text
Pod Runs

On Matching Node
```

---

# Using Labels With Node Affinity

Example

```yaml
matchExpressions:

- key: project

  operator: In

  values:

  - roboshop
```

---

Scheduler checks:

```text
Node Labels
```

---

# Production Example

Cluster

```text
Production Nodes

project=roboshop
```

---

```text
Development Nodes

project=dev
```

---

Frontend Deployment

```yaml
nodeSelector:

  project: roboshop
```

---

Result

```text
Frontend Pods

Only On Production Nodes
```

---

# Production Example - Spot Instances

Label

```text
eks.amazonaws.com/capacityType=SPOT
```

---

Use Cases

```text
CI/CD

Testing

Batch Jobs
```

---

Avoid For

```text
Critical Databases
```

---

# Common Problems

## Pod Pending

Cause

```text
Node Label Missing
```

---

Check

```bash
kubectl get nodes --show-labels
```

---

## Wrong Node Selected

Cause

```text
Incorrect Label
```

---

Verify

```bash
kubectl describe node
```

---

## Affinity Not Working

Cause

```text
Label Key Mismatch
```

---

Verify

```text
Label Name

Label Value
```

---

# Troubleshooting

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

Check Scheduling

```bash
kubectl describe pod pod-name
```

---

View Events

```bash
kubectl get events
```

---

# Common Interview Questions

## What are Node Labels?

### Short Answer

Node Labels are key-value pairs attached to Kubernetes nodes used for identification and scheduling.

### Detailed Explanation

Labels provide metadata about nodes and allow Kubernetes Scheduler to place workloads on appropriate nodes.

### Production Example

Production nodes are labeled:

```text
project=roboshop
```

and application Pods use NodeSelector to run only on those nodes.

### Follow-Up Questions

- Are labels unique?
- Can labels be modified?
- Which component uses labels?

---

## How Do You View Node Labels?

### Short Answer

Using:

```bash
kubectl get nodes --show-labels
```

### Detailed Explanation

This command displays all labels attached to every node in the cluster.

### Production Example

DevOps engineers verify labels before applying Node Affinity rules.

### Follow-Up Questions

- How do you view labels for a single node?
- Can labels be filtered?
- How do you describe a node?

---

## What are Common Built-In Node Labels?

### Short Answer

Common labels include OS, architecture, hostname, region, zone, and instance type.

### Detailed Explanation

Kubernetes and cloud providers automatically attach labels that can be used for scheduling decisions.

### Production Example

Applications requiring Linux nodes use:

```text
kubernetes.io/os=linux
```

### Follow-Up Questions

- Which label identifies Availability Zone?
- Which label identifies CPU architecture?
- Which label identifies instance type?

---

## How Do You Add a Label to a Node?

### Short Answer

Using:

```bash
kubectl label nodes node-name key=value
```

### Detailed Explanation

Labels can be added dynamically and are immediately available for scheduling decisions.

### Production Example

A node is labeled:

```text
project=roboshop
```

for production application workloads.

### Follow-Up Questions

- How do you remove labels?
- How do you update labels?
- Can labels be overwritten?

---

## Why Are Node Labels Important?

### Short Answer

They enable intelligent workload scheduling.

### Detailed Explanation

NodeSelector, Affinity, Anti-Affinity, and many Kubernetes scheduling features rely on labels.

### Production Example

Database workloads run only on nodes labeled:

```text
database=true
```

### Follow-Up Questions

- Do Taints use labels?
- Does NodeSelector use labels?
- Does Affinity use labels?

---

# Key Takeaways

```text
Node Labels are key-value pairs attached to nodes.

Labels help Kubernetes identify node characteristics.

Kubernetes automatically adds built-in labels.

Cloud providers add cloud-specific labels.

Custom labels can be created by administrators.

NodeSelector uses labels.

Node Affinity uses labels.

Labels play a major role in workload scheduling.

Understanding node labels is essential for Kubernetes scheduling.
```