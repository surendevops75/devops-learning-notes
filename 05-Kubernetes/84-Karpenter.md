# Karpenter

## Introduction

Cluster Autoscaler solved one major problem:

```text
Pods Pending

↓

Add More Nodes
```

---

However it introduced another problem:

```text
Node Group Dependency
```

---

Karpenter was created to solve this limitation.

---

# Problem Before Karpenter

Suppose we have:

```text
Managed Node Group

Instance Type

t3.medium
```

---

Current Capacity

```text
3 Nodes
```

---

Application Suddenly Requires

```text
16 CPU

64 GB RAM
```

---

Cluster Autoscaler Can Only Do

```text
Add More t3.medium Nodes
```

---

Problem

```text
Too Many Nodes

Wasted Resources

Higher Cost
```

---

# Real Production Example

Application

```text
Machine Learning Service
```

Requires

```text
16 CPU

64 GB RAM
```

---

Current Node Group

```text
t3.medium

2 CPU

4 GB RAM
```

---

Cluster Autoscaler

```text
Adds More t3.medium
```

---

Still Cannot Run Workload.

---

Why?

```text
Pod Needs Large Instance

Node Group Has Small Instances
```

---

# What Is Karpenter?

Karpenter is:

```text
Kubernetes Node Provisioner
```

for AWS.

---

Instead Of Scaling:

```text
Node Groups
```

Karpenter Creates:

```text
EC2 Instances Directly
```

---

# Main Goal

Move From

```text
Node Group Based Scaling
```

To

```text
Workload Based Scaling
```

---

# Traditional Cluster Autoscaler

```text
Pending Pod

↓

Cluster Autoscaler

↓

Increase Node Group Size

↓

New Node
```

---

# Karpenter Flow

```text
Pending Pod

↓

Analyze Requirements

↓

Choose Best EC2 Type

↓

Launch EC2

↓

Register Node

↓

Schedule Pod
```

---

# Architecture

```text
Pod

↓

Scheduler

↓

Unschedulable

↓

Karpenter

↓

EC2 API

↓

Launch Instance

↓

Node Joins Cluster

↓

Pod Scheduled
```

---

# Example

Pod Requires

```yaml
resources:

  requests:

    cpu: "8"

    memory: "32Gi"
```

---

Karpenter Evaluates

```text
m5.2xlarge

m5.4xlarge

r5.2xlarge
```

---

Chooses Best Match.

---

# Why Karpenter Is Better

Cluster Autoscaler

```text
Uses Existing Node Groups
```

---

Karpenter

```text
Chooses Best EC2 Automatically
```

---

Benefits

```text
Faster

Cheaper

Smarter
```

---

# Real World Example

E-Commerce Platform

Normal Traffic

```text
3 Nodes
```

---

Black Friday

```text
500 Pods
```

---

Karpenter

```text
Launches Multiple EC2 Instances

Automatically
```

---

Traffic Drops

---

Karpenter

```text
Terminates Unused Nodes
```

---

# Installation Prerequisites

## OIDC Provider

Required For

```text
IRSA
```

---

## IAM Role

Karpenter Needs Permission To

```text
Launch EC2

Terminate EC2

Read AWS Resources
```

---

## SQS Queue

Required For

```text
Interruption Handling
```

---

# NodePool

In Modern Karpenter

```text
Provisioner

↓

NodePool
```

---

NodePool Defines

```text
Node Requirements

Capacity Type

Instance Family
```

---

# Example NodePool

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool

metadata:
  name: default

spec:

  template:

    spec:

      requirements:

      - key: kubernetes.io/arch

        operator: In

        values:

        - amd64

      - key: karpenter.k8s.aws/instance-category

        operator: In

        values:

        - m

        - c

        - r
```

---

# What Does This Mean?

Allow

```text
Compute Optimized

Memory Optimized

General Purpose
```

instances.

---

# Spot Instances

One Huge Advantage.

---

NodePool

```yaml
requirements:

- key: karpenter.sh/capacity-type

  operator: In

  values:

  - spot
```

---

Result

```text
Much Lower AWS Cost
```

---

# Production Example

Batch Processing

```text
1000 Jobs
```

---

Can Run On

```text
Spot Instances
```

---

Cost Reduced Significantly.

---

# Consolidation

One Powerful Feature.

---

Suppose

```text
10 Small Nodes
```

Running.

---

Traffic Drops.

---

Karpenter Can Consolidate

```text
10 Nodes

↓

2 Nodes
```

---

Automatically.

---

# Scale-Up Process

Current

```text
3 Nodes
```

---

New Deployment

```text
100 Pods
```

---

Result

```text
Pods Pending
```

---

Karpenter Detects

```text
CPU

Memory

Architecture

Capacity Type
```

---

Creates Nodes.

---

Pods Become Running.

---

# Scale-Down Process

Traffic Drops.

---

Unused Nodes Detected.

---

Karpenter

```text
Cordon

↓

Drain

↓

Terminate EC2
```

---

Automatically.

---

# Karpenter vs Cluster Autoscaler

## Cluster Autoscaler

```text
Scales Node Groups
```

---

Needs

```text
ASG

Managed Node Group
```

---

Only Adds

```text
Existing Instance Types
```

---

## Karpenter

```text
Scales EC2 Directly
```

---

Does Not Need

```text
Large Number Of Node Groups
```

---

Chooses

```text
Best Instance Type
```

Automatically.

---

# Example Comparison

Workload Requires

```text
16 CPU

64 GB RAM
```

---

Cluster Autoscaler

```text
Adds More Small Nodes
```

---

Karpenter

```text
Launches

m5.4xlarge
```

---

Better Solution.

---

# Spot + On Demand Example

Critical Applications

```text
On Demand
```

---

Batch Jobs

```text
Spot
```

---

NodePool Can Control Both.

---

# Production Architecture

```text
Deployment

↓

Pods

↓

Pending

↓

Karpenter

↓

EC2 API

↓

Launch Instance

↓

Node Registered

↓

Pods Scheduled
```

---

# Common Production Problems

## Problem 1

NodePool Too Restrictive

Example

```text
Only m5.large Allowed
```

---

AWS Has No Capacity.

---

Pods Remain Pending.

---

## Problem 2

Insufficient IAM Permissions

---

Karpenter Cannot Launch EC2.

---

## Problem 3

Wrong Resource Requests

---

Pod Requests

```text
128 CPU
```

---

No Suitable Instance Found.

---

# Troubleshooting Commands

Pods

```bash
kubectl get pods
```

---

Nodes

```bash
kubectl get nodes
```

---

NodePools

```bash
kubectl get nodepools
```

---

Describe NodePool

```bash
kubectl describe nodepool
```

---

Karpenter Logs

```bash
kubectl logs \
-n karpenter \
deployment/karpenter
```

---

# Common Interview Questions

## Why Was Karpenter Created?

### Short Answer

Karpenter was created to overcome Cluster Autoscaler's dependency on predefined node groups.

### Detailed Explanation

Cluster Autoscaler can only scale existing node groups, while Karpenter dynamically chooses and launches the most suitable EC2 instance type.

### Production Example

```text
Workload Needs

16 CPU

64 GB RAM

↓

Karpenter

↓

Launch Appropriate EC2
```

### Follow-Up Questions

- Why is this cheaper?
- Why is this faster?
- What AWS APIs are used?

---

## Difference Between Karpenter And Cluster Autoscaler?

### Short Answer

Cluster Autoscaler scales node groups while Karpenter provisions EC2 instances directly.

### Production Example

```text
Cluster Autoscaler

Pending Pods

↓

Increase Node Group
```

```text
Karpenter

Pending Pods

↓

Launch Best EC2
```

### Follow-Up Questions

- Which is preferred in modern EKS?
- Can both run together?
- What are the migration challenges?

---

## How Does Karpenter Reduce Cost?

### Short Answer

By selecting optimal instance types, consolidating nodes, and supporting Spot instances.

### Production Example

```text
10 Nodes

↓

2 Nodes

After Consolidation
```

### Follow-Up Questions

- What is consolidation?
- How do Spot instances work?
- How do you handle Spot interruptions?

---

# Key Takeaways

```text
Karpenter is a modern Kubernetes node provisioning solution for AWS.

It solves Cluster Autoscaler's node group limitations.

It launches EC2 instances directly.

It selects the best instance type based on workload requirements.

It supports Spot and On-Demand instances.

It performs automatic consolidation to reduce costs.

It provides faster and more efficient scaling than Cluster Autoscaler.

Many modern EKS clusters are replacing Cluster Autoscaler with Karpenter.
```