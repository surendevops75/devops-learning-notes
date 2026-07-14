# EKS NodeGroup and Addon Upgrades

## Introduction

After upgrading:

```text
Control Plane
```

the next step is upgrading:

```text
Addons

Worker Nodes
```

---

This is a critical production activity because:

```text
Applications

Run On Worker Nodes
```

---

Control Plane manages the cluster.

---

Worker Nodes run:

```text
Pods

Containers

Applications
```

---

# EKS Upgrade Sequence

Correct Order

```text
Control Plane

↓

Addons

↓

Node Groups
```

---

Never perform:

```text
Node Upgrade

Before

Control Plane Upgrade
```

---

# What Are Addons?

Addons extend Kubernetes functionality.

---

Common EKS Addons

```text
CoreDNS

kube-proxy

VPC CNI
```

---

Additional Addons

```text
EBS CSI Driver

EFS CSI Driver

AWS Load Balancer Controller
```

---

# CoreDNS

Purpose

```text
DNS Resolution
```

---

Example

```text
catalogue

↓

catalogue.default.svc.cluster.local
```

---

Without CoreDNS

```text
Service Discovery

Fails
```

---

# kube-proxy

Purpose

```text
Service Networking
```

---

Responsible for:

```text
ClusterIP

NodePort

LoadBalancer Routing
```

---

Without kube-proxy

```text
Service Traffic

Fails
```

---

# VPC CNI

Purpose

```text
Pod Networking
```

---

Provides:

```text
Pod IP Addresses
```

---

Without VPC CNI

```text
Pods Cannot Communicate
```

---

# Why Upgrade Addons?

Control Plane Version

```text
1.33
```

---

Old Addons

```text
May Not Support

1.33
```

---

Result

```text
Unexpected Issues
```

---

# Production Example

Current

```text
EKS 1.32

CoreDNS 1.11

kube-proxy 1.32

VPC CNI 1.18
```

---

Upgrade

```text
Control Plane

1.33
```

---

Need Compatible Addons.

---

# Verify Addon Versions

AWS Console

```text
EKS

↓

Addons
```

---

AWS CLI

```bash
aws eks describe-addon \
--cluster-name roboshop-dev \
--addon-name coredns
```

---

# List Addons

```bash
aws eks list-addons \
--cluster-name roboshop-dev
```

---

Expected

```text
coredns

kube-proxy

vpc-cni
```

---

# Upgrade CoreDNS

Example

```bash
aws eks update-addon \
--cluster-name roboshop-dev \
--addon-name coredns
```

---

# Upgrade kube-proxy

```bash
aws eks update-addon \
--cluster-name roboshop-dev \
--addon-name kube-proxy
```

---

# Upgrade VPC CNI

```bash
aws eks update-addon \
--cluster-name roboshop-dev \
--addon-name vpc-cni
```

---

# Verify Addon Status

```bash
aws eks describe-addon \
--cluster-name roboshop-dev \
--addon-name coredns
```

---

Status

```text
ACTIVE
```

---

Expected.

---

# Validate Addons

Check Pods

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
Running
```

---

Examples

```text
coredns

aws-node

kube-proxy
```

---

# What Are Node Groups?

Node Groups are:

```text
Collections

Of Worker Nodes
```

---

Example

```text
roboshop-dev-ng
```

---

Contains

```text
Node-1

Node-2

Node-3
```

---

# Why Upgrade Node Groups?

Current Version

```text
1.32
```

---

Control Plane

```text
1.33
```

---

Need Consistency.

---

Need Latest Security Patches.

---

Need Latest Features.

---

# Managed Node Groups

Recommended in EKS.

---

AWS manages:

```text
Node Lifecycle

Node Replacement

Upgrade Process
```

---

# Production Scenario

Current

```text
Control Plane

1.33
```

---

Worker Nodes

```text
1.32
```

---

Need Upgrade

```text
1.32

↓

1.33
```

---

# Upgrade Approaches

## In-Place Upgrade

AWS replaces nodes gradually.

---

Simple approach.

---

## Blue-Green Upgrade

Create new node group.

---

Move workloads.

---

Delete old node group.

---

Most production teams prefer:

```text
Blue-Green
```

---

# Blue Node Group

Current

```text
1.32
```

---

Running workloads.

---

# Green Node Group

New

```text
1.33
```

---

Initially empty.

---

# Create Green Node Group

Example

```text
Blue NG

5 Nodes
```

---

Create

```text
Green NG

5 Nodes
```

---

Same Capacity.

---

# Taint Green Nodes

Purpose

```text
Prevent Scheduling
```

---

Command

```bash
kubectl taint nodes node-name \
upgrade=green:NoSchedule
```

---

Result

```text
Pods Cannot Schedule
```

---

# Verify Green Nodes

```bash
kubectl get nodes
```

---

Check

```text
Version

1.33
```

---

# Untaint Green Nodes

Command

```bash
kubectl taint nodes node-name \
upgrade=green:NoSchedule-
```

---

Scheduler can now use:

```text
Green Nodes
```

---

# Cordon Blue Nodes

Purpose

```text
Disable Scheduling
```

---

Command

```bash
kubectl cordon node-name
```

---

Result

```text
SchedulingDisabled
```

---

Existing Pods

```text
Continue Running
```

---

# Drain Blue Nodes

Purpose

```text
Move Workloads
```

---

Command

```bash
kubectl drain node-name \
--ignore-daemonsets
```

---

Result

```text
Pods Evicted

Pods Recreated

On Green Nodes
```

---

# Workload Migration

Before

```text
Blue Nodes

Running Pods
```

---

After

```text
Green Nodes

Running Pods
```

---

# Validate Migration

Check Nodes

```bash
kubectl get nodes
```

---

Check Pods

```bash
kubectl get pods -A -o wide
```

---

Verify

```text
Pods Running

On Green Nodes
```

---

# Delete Blue Node Group

After validation:

```text
Remove Blue NG
```

---

Result

```text
Only Green NG Exists
```

---

# Production Validation

Verify

```text
Applications

Ingress

Storage

Services

Monitoring
```

---

# Application Validation

Examples

```text
Catalogue

Cart

User

Shipping

Payment
```

---

All applications must be tested.

---

# Storage Validation

Check

```bash
kubectl get pvc -A
```

---

Verify

```text
Bound
```

---

# Ingress Validation

```bash
kubectl get ingress
```

---

Verify

```text
ALB Reachable
```

---

# Monitoring Validation

Verify

```text
Prometheus

Grafana

ELK
```

---

Working correctly.

---

# Common Production Problems

## Pods Stuck Pending

Cause

```text
Taints

Node Selector

Affinity Rules
```

---

Verify

```bash
kubectl describe pod pod-name
```

---

# Addon Upgrade Failure

Cause

```text
Version Incompatibility
```

---

Verify

```bash
aws eks describe-addon
```

---

# Drain Failure

Cause

```text
Pod Disruption Budget
```

---

Verify

```bash
kubectl get pdb -A
```

---

# Workloads Not Moving

Cause

```text
Green Nodes

Still Tainted
```

---

Verify

```bash
kubectl describe node
```

---

# Storage Mount Failure

Cause

```text
CSI Driver Issues
```

---

Check

```text
EBS CSI Driver

EFS CSI Driver
```

---

# Troubleshooting Commands

List Nodes

```bash
kubectl get nodes
```

---

Node Versions

```bash
kubectl get nodes -o wide
```

---

Check Pods

```bash
kubectl get pods -A
```

---

Check Addons

```bash
aws eks list-addons \
--cluster-name roboshop-dev
```

---

Describe Addon

```bash
aws eks describe-addon \
--cluster-name roboshop-dev \
--addon-name coredns
```

---

Node Details

```bash
kubectl describe node node-name
```

---

# Production Example

Current

```text
Control Plane 1.33

Blue NG 1.32
```

---

Create

```text
Green NG 1.33
```

---

Upgrade Addons

```text
CoreDNS

kube-proxy

VPC CNI
```

---

Untaint Green

---

Cordon Blue

---

Drain Blue

---

Validate

---

Delete Blue.

---

Upgrade Complete.

---

# Common Interview Questions

## What Are EKS Addons?

### Short Answer

Addons extend Kubernetes functionality.

### Detailed Explanation

CoreDNS, kube-proxy, and VPC CNI provide DNS, networking, and service routing capabilities.

### Production Example

Upgraded CoreDNS and VPC CNI after upgrading EKS Control Plane to 1.33.

### Follow-Up Questions

- Why upgrade addons?
- Which addon provides Pod IPs?
- What does CoreDNS do?

---

## Why Upgrade Addons After Control Plane?

### Short Answer

Addons must be compatible with the new Kubernetes version.

### Detailed Explanation

Older addon versions may not support newer Kubernetes releases.

### Production Example

CoreDNS upgraded after moving Control Plane from 1.32 to 1.33.

### Follow-Up Questions

- What happens if addons aren't upgraded?
- Which addons are critical?
- How do you verify versions?

---

## What Is a Node Group?

### Short Answer

A Node Group is a collection of worker nodes managed together.

### Detailed Explanation

Node Groups simplify scaling, upgrades, and lifecycle management of worker nodes.

### Production Example

roboshop-dev-ng containing five worker nodes.

### Follow-Up Questions

- What is a Managed Node Group?
- How do you upgrade nodes?
- What is Blue-Green Node Group migration?

---

## How Do You Upgrade Worker Nodes?

### Short Answer

Create a new node group, migrate workloads, and remove the old node group.

### Detailed Explanation

Production environments commonly use a Blue-Green strategy for node upgrades.

### Production Example

Blue NG 1.32 → Green NG 1.33 → Migrate Workloads → Delete Blue.

### Follow-Up Questions

- Why cordon nodes?
- Why drain nodes?
- Why use Blue-Green?

---

## What Is The Difference Between Cordon and Drain?

### Short Answer

Cordon prevents new pods from scheduling. Drain moves existing pods away.

### Detailed Explanation

Cordon marks a node unschedulable while drain safely evicts workloads.

### Production Example

Blue nodes cordoned and drained during an EKS node upgrade.

### Follow-Up Questions

- Does cordon remove pods?
- What happens during drain?
- Can a drained node be reused?

---

# Key Takeaways

```text
Control Plane is upgraded before addons and nodes.

CoreDNS, kube-proxy, and VPC CNI are critical EKS addons.

Addons must be compatible with the Kubernetes version.

Node Groups contain worker nodes.

Blue-Green is a common strategy for node upgrades.

Cordon stops new scheduling.

Drain moves existing workloads.

Validation is mandatory before deleting old node groups.

Understanding addon and node upgrades is a critical production and interview topic.
```