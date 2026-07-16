# Cluster Autoscaler

## Introduction

One of the most common production problems in Kubernetes is:

```text
Pods Cannot Be Scheduled
```

---

Example

Current Cluster

```text
3 Worker Nodes
```

---

Application Scales

```text
10 Pods

↓

100 Pods
```

---

Result

```text
80 Pods Pending
```

---

Reason

```text
No Available CPU

No Available Memory

No Available Nodes
```

---

# Problem Before Cluster Autoscaler

Suppose an EKS cluster contains:

```text
3 Nodes
```

---

Each Node Can Run

```text
20 Pods
```

---

Total Capacity

```text
60 Pods
```

---

Suddenly

```text
Black Friday Sale
```

Traffic Increases.

---

Application Scales

```text
60 Pods

↓

150 Pods
```

---

Scheduler Tries To Schedule Pods.

---

Result

```text
90 Pods Pending
```

---

Because

```text
No Free Resources
```

---

# Traditional Solution

Operations Team Receives Alert.

---

Engineer Logs Into AWS.

---

Manually Updates

```text
Node Group
```

---

Example

```text
Desired Capacity

3

↓

10
```

---

Wait For New Nodes.

---

Pods Finally Start.

---

Problems

```text
Manual

Slow

Human Errors

Not Scalable
```

---

# What Is Cluster Autoscaler?

Cluster Autoscaler is a Kubernetes component that automatically:

```text
Adds Nodes

Removes Nodes
```

based on workload demand.

---

# Main Goal

Convert:

```text
Manual Node Scaling
```

into

```text
Automatic Node Scaling
```

---

# How Cluster Autoscaler Works

It continuously watches:

```text
Pending Pods
```

---

When It Finds:

```text
Unschedulable Pods
```

It Checks:

```text
Can A New Node Solve This?
```

---

If Yes

```text
Increase Node Group Size
```

---

# Architecture

```text
Pending Pod

↓

Scheduler Cannot Place Pod

↓

Cluster Autoscaler Detects

↓

AWS ASG / Managed Node Group

↓

New EC2 Created

↓

Node Joins Cluster

↓

Pod Scheduled
```

---

# Example

Current State

```text
3 Nodes

60 Pods Capacity
```

---

Need

```text
120 Pods
```

---

Result

```text
60 Pods Pending
```

---

Cluster Autoscaler

```text
3 Nodes

↓

6 Nodes
```

---

Pods Scheduled Successfully.

---

# Scale-Up Flow

```text
Pod Created

↓

Pending

↓

Scheduler Fails

↓

Cluster Autoscaler Detects

↓

Node Group Expanded

↓

New EC2 Instance

↓

Node Registered

↓

Pod Scheduled
```

---

# Scale-Down Flow

Traffic Drops.

---

Current

```text
10 Nodes
```

---

Only Need

```text
3 Nodes
```

---

Cluster Autoscaler Detects:

```text
Underutilized Nodes
```

---

Removes Nodes.

---

Flow

```text
Low Utilization

↓

Drain Node

↓

Move Pods

↓

Terminate Node
```

---

# EKS Architecture

```text
EKS Cluster

├── Control Plane

├── Cluster Autoscaler

└── Managed Node Groups
```

---

Cluster Autoscaler Communicates With

```text
AWS Auto Scaling Groups
```

or

```text
Managed Node Groups
```

---

# Installation Prerequisites

## OIDC Provider

Required For

```text
IRSA
```

---

## IAM Policy

Cluster Autoscaler Needs Permission To

```text
Read ASG

Update ASG

Describe Instances
```

---

## Service Account

Uses

```text
IRSA
```

for AWS Access.

---

# Production Example

Node Group

```text
Minimum = 3

Desired = 3

Maximum = 10
```

---

Traffic Spike

---

Cluster Autoscaler Can Scale

```text
3

↓

10
```

---

But Never Beyond

```text
10
```

# Cluster Autoscaler Deployment

## Service Account Using IRSA

Cluster Autoscaler requires AWS permissions to:

```text
Read ASG

Update ASG

Describe Instances

Modify Desired Capacity
```

---

Create Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=roboshop-dev \
  --namespace=kube-system \
  --name=cluster-autoscaler \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/ClusterAutoscalerPolicy \
  --approve
```

---

# Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: cluster-autoscaler
  namespace: kube-system

spec:
  replicas: 1

  selector:
    matchLabels:
      app: cluster-autoscaler

  template:

    metadata:
      labels:
        app: cluster-autoscaler

    spec:

      serviceAccountName: cluster-autoscaler

      containers:

      - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.32.0

        name: cluster-autoscaler

        command:

        - ./cluster-autoscaler

        - --cloud-provider=aws

        - --skip-nodes-with-local-storage=false

        - --balance-similar-node-groups

        - --expander=least-waste

        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled

        resources:

          requests:
            cpu: 100m
            memory: 300Mi

          limits:
            cpu: 500m
            memory: 600Mi
```

---

# How Auto Discovery Works

Cluster Autoscaler discovers Node Groups using tags.

---

Example ASG Tags

```text
k8s.io/cluster-autoscaler/enabled=true

k8s.io/cluster-autoscaler/roboshop-dev=owned
```

---

Flow

```text
Cluster Autoscaler

↓

Find ASG Tags

↓

Discover Node Groups

↓

Manage Scaling
```

---

# Managed Node Group Example

Terraform

```hcl
desired_size = 3

min_size = 3

max_size = 10
```

---

Current State

```text
3 Nodes
```

---

Traffic Spike

```text
Pods Pending
```

---

Cluster Autoscaler

```text
desired_size

3

↓

8
```

---

AWS Creates

```text
5 Additional EC2 Instances
```

---

# Real Production Example

Node Group

```text
Min = 3

Desired = 3

Max = 10
```

---

Traffic

```text
5 Pods

↓

200 Pods
```

---

HPA Creates

```text
200 Pods
```

---

Cluster Capacity

```text
60 Pods
```

only.

---

Result

```text
140 Pods Pending
```

---

Cluster Autoscaler

```text
3 Nodes

↓

10 Nodes
```

---

Pods Become Running.

---

# Verifying Cluster Autoscaler

Check Deployment

```bash
kubectl get deployment \
-n kube-system cluster-autoscaler
```

---

Check Logs

```bash
kubectl logs \
-n kube-system \
deployment/cluster-autoscaler
```

---

Typical Log

```text
Scale-up triggered

Node group: roboshop-dev

Current size: 3

New size: 5
```
---

# Important Configuration

## Min Nodes

```text
Always Running
```

Example

```text
3
```

---

## Max Nodes

```text
Maximum Allowed
```

Example

```text
10
```

---

## Desired Nodes

```text
Current Running Nodes
```

Example

```text
5
```

---

# Production Example

Roboshop

Services

```text
Catalogue

User

Cart

Shipping

Payment
```

---

Normal Traffic

```text
3 Nodes
```

---

Sale Event

```text
8 Nodes
```

---

Night Time

```text
3 Nodes
```

---

Automatically Managed.

---

# Interaction With HPA

Many Engineers Confuse These.

---

# HPA

Scales

```text
Pods
```

---

Example

```text
5 Pods

↓

20 Pods
```

---

# Cluster Autoscaler

Scales

```text
Nodes
```

---

Example

```text
3 Nodes

↓

6 Nodes
```

---

# Combined Flow

```text
Traffic Increases

↓

CPU High

↓

HPA Creates More Pods

↓

Pods Pending

↓

Cluster Autoscaler Adds Nodes

↓

Pods Scheduled
```

---

# Common Production Problems

## Problem 1

Pods Still Pending.

---

Reason

```text
Node Group Max Reached
```

---

Example

```text
Maximum = 10
```

Already Running

```text
10 Nodes
```

---

Cannot Scale Further.

---

## Problem 2

Wrong Instance Type

---

Need

```text
32 GB RAM
```

---

Node Type

```text
t3.medium
```

---

Pod Never Schedules.

---

## Problem 3

Node Affinity Restrictions

---

Pod Requires

```text
Specific Node Label
```

---

Autoscaler Adds Wrong Node.

---

Pod Still Pending.

---

# Major Limitation Of Cluster Autoscaler

Cluster Autoscaler Can Only Scale:

```text
Existing Node Groups
```

---

Example

Node Group

```text
t3.medium
```

---

Autoscaler Adds More

```text
t3.medium
```

Only.

---

Cannot Dynamically Choose

```text
m5.large

m5.xlarge

r6g.large
```

---

# Why Karpenter Was Created

To Solve This Limitation.

---

Instead Of

```text
Scaling Node Groups
```

Karpenter

```text
Creates Best EC2 Instance
```

Based On Pod Requirements.

---

# Troubleshooting Commands

Check Pending Pods

```bash
kubectl get pods
```

---

Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

Check Events

```bash
kubectl get events
```

---

Cluster Autoscaler Logs

```bash
kubectl logs \
-n kube-system \
deployment/cluster-autoscaler
```

---

Nodes

```bash
kubectl get nodes
```

---

# Common Interview Questions

## What Problem Does Cluster Autoscaler Solve?

### Short Answer

Cluster Autoscaler automatically adds or removes worker nodes based on workload demand.

### Detailed Explanation

When pods cannot be scheduled due to insufficient resources, Cluster Autoscaler increases node group capacity automatically.

### Production Example

```text
Pods Pending

↓

Cluster Autoscaler

↓

New Nodes

↓

Pods Scheduled
```

### Follow-Up Questions

- Does it scale Pods or Nodes?
- What triggers scale-up?
- What triggers scale-down?
- Does it work with HPA?

---

## Difference Between HPA And Cluster Autoscaler?

### Short Answer

HPA scales Pods while Cluster Autoscaler scales Nodes.

### Production Example

```text
HPA

5 Pods → 20 Pods

Cluster Autoscaler

3 Nodes → 6 Nodes
```

### Follow-Up Questions

- Which one acts first?
- Can HPA work without Cluster Autoscaler?
- Can Cluster Autoscaler work without HPA?

---

## How Is Cluster Autoscaler Configured In EKS?

### Short Answer

Cluster Autoscaler runs as a Deployment inside the cluster and uses IRSA to access AWS APIs. It discovers and manages Auto Scaling Groups or Managed Node Groups using AWS tags.

### Detailed Explanation

Cluster Autoscaler continuously monitors unschedulable pods. When it detects pods that cannot be placed due to insufficient resources, it communicates with AWS Auto Scaling Groups or Managed Node Groups and increases the desired capacity.

### Production Example

```text
Pending Pods

↓

Cluster Autoscaler

↓

Increase Desired Capacity

↓

AWS Creates EC2

↓

Node Joins Cluster

↓

Pods Scheduled
```

### Follow-Up Questions

- How does Cluster Autoscaler authenticate with AWS?
- What AWS permissions are required?
- How does auto-discovery work?
- How does it identify node groups?
- Can it scale beyond max_size?
- What happens if max_size is reached?

---

## What Is The Biggest Limitation Of Cluster Autoscaler?

### Short Answer

It can only scale existing node groups.

### Detailed Explanation

It cannot dynamically choose optimal EC2 instance types based on workload requirements.

### Production Example

```text
Node Group = t3.medium

Autoscaler Can Add

More t3.medium Nodes

Only
```

### Follow-Up Questions

- How does Karpenter solve this?
- Why is this inefficient?
- What happens with large workloads?

---

# Key Takeaways

```text
Cluster Autoscaler runs as a Deployment inside the cluster.

IRSA is used to securely access AWS APIs.

Node Groups are discovered using AWS tags.

Cluster Autoscaler increases desired capacity when pods are pending.

AWS launches new EC2 instances automatically.

New nodes join the cluster and pending pods get scheduled.

Cluster Autoscaler works together with HPA.

It can only scale existing node groups and cannot dynamically select instance types.

Cluster Autoscaler solves the problem of insufficient worker nodes.

It automatically scales node groups based on pending pods.

It works together with HPA.

It supports automatic scale-up and scale-down.

It requires AWS Auto Scaling Groups or Managed Node Groups.

It can only scale existing node groups.

Karpenter was introduced to overcome this limitation.

Cluster Autoscaler is still widely used in production EKS environments.
```