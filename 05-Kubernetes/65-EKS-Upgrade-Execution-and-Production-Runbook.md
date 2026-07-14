# EKS Upgrade Execution and Production Runbook

## Introduction

In previous chapters we learned:

```text
Cluster Upgrades

Control Plane Upgrades

Addon Upgrades

Node Group Upgrades

Blue-Green Strategy
```

---

Now let's see:

```text
What Actually Happens

During A Real Production Upgrade
```

---

This chapter is based on:

```text
Real Platform Team Activities

During Maintenance Windows
```

---

# Production Scenario

Current Cluster

```text
roboshop-dev

Kubernetes 1.32
```

---

Target Version

```text
Kubernetes 1.33
```

---

Upgrade Window

```text
16 DEC

12:00 AM

To

06:00 AM
```

---

# Why Maintenance Window?

Upgrades may cause:

```text
API Delays

Pod Rescheduling

Temporary Service Issues
```

---

To minimize impact:

```text
Perform Upgrades

During Low Traffic Hours
```

---

# Teams Involved

Production Upgrade Usually Involves

```text
Platform Team

Network Team

Security Team

Application Teams

Management
```

---

# Upgrade Lifecycle

```text
Planning

↓

Communication

↓

Preparation

↓

Upgrade

↓

Validation

↓

Completion
```

---

# Phase 1 - Planning

Platform Team Creates:

```text
Upgrade Plan

Rollback Plan

Validation Checklist
```

---

Before Upgrade Verify:

```text
Version Compatibility

Addon Compatibility

Controller Compatibility
```

---

Examples

```text
ALB Controller

EBS CSI Driver

EFS CSI Driver

ArgoCD
```

---

# Phase 2 - Communication

Usually Sent

```text
2 Months

1 Month

1 Week

1 Day
```

before upgrade.

---

Example Mail

```text
Subject:

EKS Upgrade Notification

Cluster:
roboshop-dev

Current:
1.32

Target:
1.33

Date:
16 DEC

Time:
12AM-6AM
```

---

# Why Multiple Communications?

To ensure:

```text
No Surprises

No Deployments

No Production Changes
```

during upgrade.

---

# Phase 3 - Access Freeze

Network Team may temporarily block:

```text
Application Team Access
```

---

Reason

```text
Prevent Unexpected Changes
```

---

Examples

```text
Jenkins Access

GitHub Actions

Developer VPN Access
```

---

may be restricted.

---

# Phase 4 - Pre-Upgrade Validation

Verify AWS Identity

```bash
aws sts get-caller-identity
```

---

Verify Cluster Access

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

Verify Nodes

```bash
kubectl get nodes
```

---

Verify Pods

```bash
kubectl get pods -A
```

---

Verify Storage

```bash
kubectl get pvc -A
```

---

Verify Ingress

```bash
kubectl get ingress -A
```

---

# Phase 5 - Create Green Node Group

Current Environment

```text
Blue NG

5 Nodes

Version 1.32
```

---

Create

```text
Green NG

5 Nodes

Version 1.33
```

---

Goal

```text
Same Capacity

Same Configuration
```

---

# Architecture

Before

```text
Blue NG

Node-1

Node-2

Node-3

Node-4

Node-5
```

---

After

```text
Blue NG

+

Green NG
```

---

# Phase 6 - Taint Green Nodes

Purpose

```text
Prevent Scheduling
```

until validation.

---

Command

```bash
kubectl taint nodes node-name \
upgrade=green:NoSchedule
```

---

Result

```text
Pods Cannot Run

On Green Nodes
```

---

# Why Taint First?

Platform Team Wants To Verify:

```text
Node Health

Version

Networking
```

before workloads move.

---

# Phase 7 - Upgrade Control Plane

Upgrade

```text
1.32

↓

1.33
```

---

AWS Upgrades

```text
API Server

Scheduler

Controller Manager

etcd
```

---

Verify

```bash
kubectl version
```

---

Expected

```text
Server Version 1.33
```

---

# Phase 8 - Upgrade Addons

Upgrade

```text
CoreDNS

kube-proxy

VPC CNI
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
Running
```

---

# Phase 9 - Verify Green Nodes

Check

```bash
kubectl get nodes
```

---

Expected

```text
Version 1.33
```

---

Verify

```text
Networking

Storage

Node Readiness
```

---

# Phase 10 - Remove Taints

Command

```bash
kubectl taint nodes node-name \
upgrade=green:NoSchedule-
```

---

Result

```text
Scheduler Can Use Green Nodes
```

---

# Phase 11 - Cordon Blue Nodes

Purpose

```text
Stop New Scheduling
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

# What Does Cordon Do?

Cordon:

```text
Blocks New Pods
```

---

Does Not

```text
Remove Existing Pods
```

---

# Phase 12 - Drain Blue Nodes

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

# Workload Migration Flow

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

# What Happens During Drain?

Kubernetes:

```text
Evicts Pods

Finds New Nodes

Schedules Pods
```

---

Applications continue running.

---

# Phase 13 - Validate Applications

Platform Team Checks

```text
Nodes

Pods

Services

Ingress

Storage
```

---

Application Teams Check

```text
Catalogue

Cart

User

Shipping

Payment
```

---

# Application Validation

Verify

```text
Login

API Calls

Checkout

Payments
```

---

# Storage Validation

Verify

```bash
kubectl get pvc -A
```

---

Expected

```text
Bound
```

---

# Ingress Validation

Verify

```bash
kubectl get ingress
```

---

Check

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

# Phase 14 - Delete Blue Node Group

After Validation

```text
Delete Blue NG
```

---

Result

```text
Only Green NG Exists
```

---

Cluster State

```text
1.33

Healthy
```

---

# Phase 15 - Restore Access

Network Team Restores

```text
Jenkins

GitHub Actions

Developer Access
```

---

Deployments Allowed Again.

---

# Phase 16 - Completion Communication

Example Mail

```text
Subject:

EKS Upgrade Completed

Cluster:
roboshop-dev

Version:
1.33

Status:
Healthy
```

---

# Rollback Scenario

Suppose

```text
Addon Failure

Networking Failure

Application Failure
```

---

Do Not Delete

```text
Blue NG
```

---

Move Workloads Back.

---

Investigate.

---

# Emergency Rollback Flow

```text
Green Problem

↓

Cordon Green

↓

Drain Green

↓

Uncordon Blue

↓

Workloads Return

↓

Investigate
```

---

# Common Production Issues

## Pods Stuck Pending

Cause

```text
Green Nodes Still Tainted
```

---

Verify

```bash
kubectl describe node
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

# Application Not Reachable

Cause

```text
Ingress

ALB

DNS
```

issues.

---

Verify

```bash
kubectl get ingress
```

---

# Storage Issues

Cause

```text
CSI Driver Failure
```

---

Verify

```text
EBS CSI Driver

EFS CSI Driver
```

---

# Validation Checklist

Before Completion Verify:

```text
All Nodes Ready

All Pods Running

PVCs Bound

ALB Healthy

Applications Reachable

Monitoring Working

Logs Available
```

---

# Troubleshooting Commands

Nodes

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

Ingress

```bash
kubectl get ingress
```

---

PVC

```bash
kubectl get pvc -A
```

---

PDB

```bash
kubectl get pdb -A
```

---

# Common Interview Questions

## How Do You Perform an EKS Upgrade?

### Short Answer

Create Green Node Group, upgrade Control Plane and Addons, migrate workloads, validate, and remove old nodes.

### Detailed Explanation

Production upgrades follow a planned runbook involving communication, access freeze, node migration, validation, and rollback planning.

### Production Example

roboshop-dev upgraded from Kubernetes 1.32 to 1.33 using Blue-Green node groups.

### Follow-Up Questions

- Why taint Green nodes?
- Why cordon Blue nodes?
- Why drain nodes?
- How do you rollback?

---

## Why Do We Cordon Nodes?

### Short Answer

To stop new pod scheduling.

### Detailed Explanation

Cordon marks nodes as unschedulable while existing workloads continue running.

### Production Example

Blue nodes cordoned before workload migration.

### Follow-Up Questions

- Does cordon remove pods?
- What happens after cordon?
- Why not drain directly?

---

## Why Do We Drain Nodes?

### Short Answer

To safely move workloads.

### Detailed Explanation

Drain evicts workloads and allows Kubernetes to recreate them elsewhere.

### Production Example

Pods moved from Blue Node Group to Green Node Group.

### Follow-Up Questions

- What is pod eviction?
- What is a PDB?
- What if drain fails?

---

## How Do You Rollback an EKS Upgrade?

### Short Answer

Keep Blue environment available and move workloads back.

### Detailed Explanation

Blue-Green architecture enables fast rollback by retaining the previous environment until validation completes.

### Production Example

Application issues discovered after migration resulted in workloads moving back to Blue Node Group.

### Follow-Up Questions

- Why not delete Blue immediately?
- What validations are required?
- What triggers rollback?

---

# Key Takeaways

```text
Production upgrades follow a structured runbook.

Communication is critical.

Access is usually frozen during maintenance.

Green Node Groups are created before migration.

Control Plane is upgraded first.

Addons are upgraded next.

Blue nodes are cordoned and drained.

Applications must be validated before completion.

Rollback planning is mandatory.

Blue-Green upgrades are the safest approach for EKS production environments.
```