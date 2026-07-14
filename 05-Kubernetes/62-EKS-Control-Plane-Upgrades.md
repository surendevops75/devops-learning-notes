# EKS Control Plane Upgrades

## Introduction

An EKS cluster consists of:

```text
Control Plane

Worker Nodes
```

---

Applications run on:

```text
Worker Nodes
```

---

But the entire cluster is managed by:

```text
Control Plane
```

---

Without Control Plane:

```text
No Scheduling

No API Access

No Cluster Operations
```

---

# What is the Control Plane?

The Control Plane is the brain of Kubernetes.

---

It contains:

```text
API Server

Scheduler

Controller Manager

etcd
```

---

In EKS:

```text
Control Plane

Is Managed By AWS
```

---

You cannot SSH into it.

---

You cannot directly manage it.

---

AWS manages:

```text
Availability

Patching

Scaling

Upgrades
```

---

# Control Plane Components

## API Server

Entry point to Kubernetes.

---

Every command:

```bash
kubectl get pods
```

goes to:

```text
API Server
```

---

## Scheduler

Responsible for:

```text
Selecting Worker Nodes

For Pods
```

---

Example

```text
Pod Created

↓

Scheduler

↓

Node Selected
```

---

## Controller Manager

Maintains desired state.

---

Example

```text
Deployment

Replicas = 3
```

---

One pod crashes.

---

Controller Manager creates:

```text
Replacement Pod
```

---

## etcd

Kubernetes database.

---

Stores:

```text
Pods

Services

Secrets

ConfigMaps

Deployments
```

---

Everything in Kubernetes.

---

# What is a Control Plane Upgrade?

Example

```text
1.32

↓

1.33
```

---

AWS upgrades:

```text
API Server

Scheduler

Controller Manager

etcd
```

---

Worker Nodes remain unchanged initially.

---

# Why Upgrade Control Plane?

Reasons

```text
Security Fixes

Bug Fixes

Performance Improvements

New Features

AWS Support
```

---

# Production Example

Current Version

```text
1.32
```

---

AWS Support Ends Soon.

---

Platform Team Plans

```text
Upgrade To 1.33
```

---

# Important Rule

Always Upgrade:

```text
Control Plane First
```

---

Then

```text
Addons

Worker Nodes
```

---

Never reverse this order.

---

# Why Upgrade Control Plane First?

Worker Nodes communicate with:

```text
API Server
```

---

API Server must support:

```text
New Version Features
```

---

Before nodes upgrade.

---

# Version Skew Policy

Question

```text
Can Control Plane

And Nodes

Run Different Versions?
```

---

Answer

```text
Yes
```

---

Example

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

Supported temporarily.

---

# Control Plane Upgrade Flow

```text
Current Version

1.32

↓

Compatibility Check

↓

Upgrade Control Plane

↓

Validation

↓

Upgrade Addons

↓

Upgrade Nodes
```

---

# Pre-Upgrade Checks

## Verify AWS Access

```bash
aws sts get-caller-identity
```

---

Purpose

```text
Verify AWS Authentication
```

---

## Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

Purpose

```text
Connect kubectl To Cluster
```

---

## Verify Cluster Access

```bash
kubectl get nodes
```

---

Expected

```text
All Nodes Ready
```

---

## Verify Cluster Health

```bash
kubectl get pods -A
```

---

Check

```text
CrashLoopBackOff

Pending Pods

Failed Pods
```

---

# Verify Addons

Check:

```text
CoreDNS

kube-proxy

VPC CNI
```

---

Command

```bash
kubectl get pods -n kube-system
```

---

# Verify Controllers

Examples

```text
AWS Load Balancer Controller

EBS CSI Driver

EFS CSI Driver

ArgoCD
```

---

# Upgrade Planning

Platform Team schedules:

```text
Maintenance Window
```

---

Example

```text
12:00 AM

↓

06:00 AM
```

---

Reason

```text
Low User Traffic
```

---

# Stakeholders

Upgrade involves:

```text
Platform Team

Application Teams

Network Team

Security Team
```

---

# Upgrade Methods

## AWS Console

Navigate

```text
EKS

↓

Cluster

↓

Update Version
```

---

## AWS CLI

Example

```bash
aws eks update-cluster-version \
--name roboshop-dev \
--kubernetes-version 1.33
```

---

# What Happens During Upgrade?

AWS upgrades:

```text
API Server

Scheduler

Controller Manager

etcd
```

---

AWS handles:

```text
High Availability
```

automatically.

---

# Does Control Plane Have Downtime?

Generally:

```text
No
```

---

AWS performs rolling upgrades.

---

API requests may experience:

```text
Brief Intermittent Delays
```

---

Usually applications remain unaffected.

---

# Monitoring Upgrade

Check Status

```bash
aws eks describe-update \
--name roboshop-dev \
--update-id <update-id>
```

---

Status

```text
InProgress

Successful

Failed
```

---

# Verify Version After Upgrade

Command

```bash
kubectl version
```

---

Expected

```text
Server Version

1.33
```

---

# Verify Node Versions

Command

```bash
kubectl get nodes
```

---

Example

```text
Control Plane 1.33

Nodes 1.32
```

---

Temporary state.

---

# Post Upgrade Validation

Verify:

```text
API Server

Pods

Services

Ingress

Storage
```

---

## Nodes

```bash
kubectl get nodes
```

---

Expected

```text
Ready
```

---

## Pods

```bash
kubectl get pods -A
```

---

Expected

```text
Running
```

---

## Services

```bash
kubectl get svc -A
```

---

Expected

```text
Healthy Endpoints
```

---

## Ingress

```bash
kubectl get ingress -A
```

---

Verify

```text
ALB Accessible
```

---

# Common Production Problems

## Upgrade Blocked

Cause

```text
Unsupported Version Jump
```

---

Example

```text
1.30

↓

1.33
```

---

Not supported directly.

---

# Addon Compatibility Issue

Example

```text
Old CoreDNS
```

---

May not support:

```text
New Kubernetes Version
```

---

# Controller Compatibility Issue

Examples

```text
AWS Load Balancer Controller

CSI Drivers
```

---

Need validation before upgrade.

---

# API Server Access Problems

Symptoms

```text
kubectl Timeout
```

---

Verify

```bash
kubectl cluster-info
```

---

# Troubleshooting Commands

Cluster Version

```bash
aws eks describe-cluster \
--name roboshop-dev
```

---

Node Status

```bash
kubectl get nodes
```

---

Pods

```bash
kubectl get pods -A
```

---

Events

```bash
kubectl get events -A
```

---

Cluster Info

```bash
kubectl cluster-info
```

---

# Production Example

Current State

```text
Control Plane 1.32

Worker Nodes 1.32
```

---

Upgrade

```text
Control Plane

1.32

↓

1.33
```

---

Result

```text
Control Plane 1.33

Worker Nodes 1.32
```

---

Validation Completed.

---

Next Step

```text
Upgrade Addons

Upgrade Nodes
```

---

# Common Interview Questions

## What is the Kubernetes Control Plane?

### Short Answer

The Control Plane is the brain of Kubernetes responsible for cluster management.

### Detailed Explanation

It consists of API Server, Scheduler, Controller Manager, and etcd.

### Production Example

Every kubectl request goes through the API Server.

### Follow-Up Questions

- What is etcd?
- What does Scheduler do?
- What is Controller Manager?

---

## What Happens During an EKS Control Plane Upgrade?

### Short Answer

AWS upgrades API Server, Scheduler, Controller Manager, and etcd.

### Detailed Explanation

The upgrade is managed by AWS and typically occurs with minimal disruption.

### Production Example

Control Plane upgraded from 1.32 to 1.33 while worker nodes remained on 1.32 temporarily.

### Follow-Up Questions

- Is there downtime?
- What is version skew?
- Who manages the upgrade?

---

## Why Upgrade Control Plane First?

### Short Answer

Worker nodes depend on the API Server.

### Detailed Explanation

The Control Plane must be upgraded before addons and worker nodes to maintain compatibility.

### Production Example

Control Plane upgraded first, followed by CoreDNS, kube-proxy, and worker nodes.

### Follow-Up Questions

- What happens if nodes are upgraded first?
- Is version skew allowed?
- What is the upgrade sequence?

---

## Have You Performed Control Plane Upgrades?

### Short Answer

Yes, during planned EKS maintenance windows.

### Detailed Explanation

We validated compatibility, upgraded the control plane, verified cluster health, and then proceeded with addon and node upgrades.

### Production Example

Upgraded roboshop-dev from Kubernetes 1.32 to 1.33 with post-upgrade validation of workloads and ingress traffic.

### Follow-Up Questions

- How did you validate success?
- What commands did you use?
- How did you verify application health?

---

# Key Takeaways

```text
The Control Plane is the brain of Kubernetes.

EKS Control Plane is managed by AWS.

Control Plane contains API Server, Scheduler, Controller Manager, and etcd.

Always upgrade Control Plane first.

Version skew between Control Plane and Nodes is supported temporarily.

Validate addons and controllers before upgrading.

Perform health checks before and after upgrades.

Control Plane upgrades are common production operations for Platform Engineers.

Understanding Control Plane upgrades is a frequently asked interview topic.
```