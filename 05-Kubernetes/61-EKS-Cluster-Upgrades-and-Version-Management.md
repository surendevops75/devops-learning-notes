# EKS Cluster Upgrades and Version Management

## Introduction

Kubernetes is evolving continuously.

---

New releases provide:

```text
Bug Fixes

Security Fixes

Performance Improvements

New Features
```

---

To take advantage of these improvements:

```text
Clusters Must Be Upgraded
```

---

In production environments:

```text
Cluster Upgrades

Are Planned Activities
```

---

Not emergency tasks.

---

# What is a Cluster Upgrade?

A cluster upgrade is the process of moving:

```text
Current Version

↓

New Version
```

---

Example

```text
1.32

↓

1.33
```

---

The upgrade affects:

```text
Control Plane

Worker Nodes

Addons
```

---

# Why Are Upgrades Required?

Without upgrades:

```text
Unsupported Versions

Security Risks

Missing Features

Compatibility Problems
```

---

Organizations upgrade to:

```text
Stay Supported

Improve Security

Use New Features

Fix Bugs
```

---

# Real Production Example

Current Version

```text
1.32
```

---

AWS announces:

```text
1.32 Support Ending Soon
```

---

Platform Team plans:

```text
Upgrade To 1.33
```

---

# Types of Upgrades

There are multiple types of upgrades.

---

## Platform Upgrade

Example

```text
EKS 1.32

↓

EKS 1.33
```

---

Focus

```text
Cluster Infrastructure
```

---

## Application Upgrade

Example

```text
Catalogue v4

↓

Catalogue v5
```

---

Focus

```text
Business Application
```

---

## Programming Language Upgrade

Example

```text
Java 17

↓

Java 21
```

---

Or

```text
Python 3.10

↓

Python 3.12
```

---

Focus

```text
Application Runtime
```

---

## Tool Upgrade

Examples

```text
Terraform

Helm

Jenkins

ArgoCD
```

---

Focus

```text
DevOps Toolchain
```

---

# Kubernetes Versioning

Kubernetes versions follow:

```text
Major.Minor.Patch
```

---

Example

```text
1.33.1
```

---

Components

```text
1 = Major

33 = Minor

1 = Patch
```

---

# Major Version

Represents:

```text
Large Architectural Changes
```

---

Rare in Kubernetes.

---

# Minor Version

Represents:

```text
New Features

Enhancements
```

---

Example

```text
1.32

↓

1.33
```

---

Most common upgrade type.

---

# Patch Version

Represents:

```text
Bug Fixes

Security Fixes
```

---

Example

```text
1.33.1

↓

1.33.2
```

---

Generally low risk.

---

# EKS Version Lifecycle

AWS supports:

```text
Specific Kubernetes Versions
```

---

Older versions eventually become:

```text
Deprecated
```

---

Then

```text
Unsupported
```

---

Running unsupported versions is risky.

---

# Risks of Staying on Old Versions

## Security Risks

Missing patches.

---

## Compatibility Issues

New tools may fail.

---

## Vendor Support Issues

AWS support becomes limited.

---

## Missing Features

Cannot use latest capabilities.

---

# Components Involved in Upgrades

## Control Plane

Managed by AWS.

---

Contains

```text
API Server

Scheduler

Controller Manager

etcd
```

---

## Worker Nodes

Managed by customer.

---

Run:

```text
Pods

Containers
```

---

## Addons

Examples

```text
CoreDNS

kube-proxy

VPC CNI
```

---

# Upgrade Order

Production best practice:

```text
Control Plane

↓

Addons

↓

Worker Nodes
```

---

Never reverse the order.

---

# Why Upgrade Control Plane First?

Worker nodes communicate with:

```text
API Server
```

---

API Server must be upgraded first.

---

# Why Upgrade Addons Next?

Addons interact with:

```text
Cluster Networking

DNS

Service Discovery
```

---

Need compatibility.

---

# Why Upgrade Worker Nodes Last?

Workloads run on:

```text
Worker Nodes
```

---

Upgrade should happen after:

```text
Control Plane Stability
```

---

# Version Skew Policy

Question

```text
Can Worker Nodes

Be Different Versions?
```

---

Answer

```text
Yes
```

---

But within supported limits.

---

Example

```text
Control Plane 1.33

Node 1.32
```

---

Usually supported temporarily.

---

# Compatibility Checks

Before upgrade verify:

```text
Applications

Addons

Controllers

Helm Charts
```

---

Examples

```text
AWS Load Balancer Controller

EBS CSI Driver

EFS CSI Driver
```

---

# Production Example

Current Cluster

```text
EKS 1.32
```

---

Before upgrade verify:

```text
ALB Controller Compatibility

CSI Driver Compatibility

ArgoCD Compatibility
```

---

# Pre-Upgrade Checklist

## Verify Cluster Access

```bash
aws sts get-caller-identity
```

---

Purpose

```text
Verify AWS Identity
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
Node List
```

---

## Verify Workloads

```bash
kubectl get pods -A
```

---

Ensure

```text
No Critical Failures
```

---

## Verify Storage

```bash
kubectl get pvc -A
```

---

Check

```text
PVC Bound
```

---

## Verify Ingress

```bash
kubectl get ingress -A
```

---

Check

```text
Traffic Working
```

---

# Upgrade Planning

Production upgrades require:

```text
Maintenance Window
```

---

Example

```text
12 AM

↓

6 AM
```

---

Why?

```text
Lower User Traffic
```

---

# Stakeholders

Upgrade involves:

```text
Platform Team

Network Team

Security Team

Application Teams
```

---

# Communication

Platform Team sends:

```text
Upgrade Date

Upgrade Time

Expected Impact

Rollback Plan
```

---

# Upgrade Validation

After upgrade verify:

```text
Nodes

Pods

Services

Ingress

Storage
```

---

# Node Validation

```bash
kubectl get nodes
```

---

Verify

```text
All Nodes Ready
```

---

# Pod Validation

```bash
kubectl get pods -A
```

---

Verify

```text
Running

Ready
```

---

# Service Validation

```bash
kubectl get svc -A
```

---

Verify

```text
Endpoints Available
```

---

# Storage Validation

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
kubectl get ingress -A
```

---

Verify

```text
ALB Accessible
```

---

# Common Upgrade Risks

## Application Failure

After upgrade:

```text
Pods Crash
```

---

Cause

```text
Compatibility Problems
```

---

# Addon Failure

Example

```text
Old CoreDNS
```

---

May fail with:

```text
New Kubernetes Version
```

---

# Controller Failure

Examples

```text
ALB Controller

CSI Drivers
```

---

May become incompatible.

---

# Why Test First?

Production upgrades should never be tested first in production.

---

Use:

```text
DEV

QA

UAT
```

---

Before:

```text
PROD
```

---

# Upgrade Environments

Order

```text
DEV

↓

QA

↓

UAT

↓

PROD
```

---

# Production Example

Roboshop Environment

```text
roboshop-dev

↓

roboshop-qa

↓

roboshop-prod
```

---

Upgrade lower environments first.

---

# Common Interview Questions

## Why Do We Upgrade Kubernetes Clusters?

### Short Answer

To receive security fixes, bug fixes, and new features.

### Detailed Explanation

Older versions eventually become unsupported and may introduce security and compatibility risks.

### Production Example

Upgrading EKS from 1.32 to 1.33 before support ends.

### Follow-Up Questions

- What risks exist if you don't upgrade?
- Who plans upgrades?
- How often should upgrades occur?

---

## What Components Are Upgraded in EKS?

### Short Answer

Control Plane, Addons, and Worker Nodes.

### Detailed Explanation

The EKS upgrade process includes upgrading Kubernetes control plane components, cluster addons, and worker nodes.

### Production Example

Control Plane → CoreDNS → VPC CNI → Worker Nodes.

### Follow-Up Questions

- Which is upgraded first?
- Why not upgrade nodes first?
- What is an addon?

---

## What Is Version Skew Policy?

### Short Answer

Control Plane and Nodes can temporarily run different supported versions.

### Detailed Explanation

Kubernetes allows a limited version difference between control plane and worker nodes during upgrades.

### Production Example

Control Plane 1.33 while nodes remain 1.32 during migration.

### Follow-Up Questions

- Is long-term skew recommended?
- What are the limits?
- Why does skew exist?

---

## What Checks Do You Perform Before Upgrading EKS?

### Short Answer

Verify cluster health, workloads, addons, storage, ingress, and compatibility.

### Detailed Explanation

Production upgrades require health checks and compatibility validation before execution.

### Production Example

Validate ALB Controller, CSI Drivers, and application workloads before upgrading.

### Follow-Up Questions

- Which commands do you run?
- How do you validate addons?
- How do you prepare rollback?

---

# Key Takeaways

```text
Cluster upgrades are planned production activities.

Upgrades improve security, stability, and features.

EKS upgrades involve Control Plane, Addons, and Worker Nodes.

Recommended order:
Control Plane → Addons → Nodes.

Always verify compatibility before upgrading.

Perform upgrades first in lower environments.

Validate workloads before and after upgrades.

Maintenance windows and communication are critical.

Version skew is supported temporarily during upgrades.

Understanding upgrade planning is essential for DevOps and Platform Engineers.
```