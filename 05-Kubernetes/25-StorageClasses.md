# StorageClasses

## Introduction

In the previous section, we learned:

```text
PV

PVC

Persistent Storage
```

---

Traditional storage provisioning works like this:

```text
Create Disk
      ↓

Create PV
      ↓

Create PVC
      ↓

Use Storage
```

---

Problem

```text
Too Much Manual Work
```

---

For every application:

```text
Storage Team Creates Disk

Kubernetes Admin Creates PV

DevOps Creates PVC
```

---

Not scalable.

---

Kubernetes solves this using:

```text
StorageClasses
```

---

# What is a StorageClass?

A StorageClass is:

```text
A Kubernetes Resource

Used To Define

How Storage Should Be Created
```

---

Think of StorageClass as:

```text
Storage Template
```

---

It tells Kubernetes:

```text
Which Storage To Create

How To Create It

What Type To Create
```

---

# Why Do We Need StorageClasses?

Without StorageClass

```text
Disk Creation Manual
```

---

With StorageClass

```text
Disk Creation Automated
```

---

Benefits

```text
Less Manual Work

Self Service Storage

Automation

Scalability
```

---

# Storage Evolution

Traditional

```text
Disk
 ↓

PV
 ↓

PVC
 ↓

Pod
```

---

StorageClass

```text
StorageClass
        ↓

PVC
        ↓

PV Created Automatically
        ↓

Disk Created Automatically
        ↓

Pod
```

---

# StorageClass Architecture

```text
Pod
 ↓

PVC
 ↓

StorageClass
 ↓

PV
 ↓

Disk
```

---

StorageClass acts as:

```text
Storage Provisioning Blueprint
```

---

# StorageClass Components

A StorageClass typically defines:

```text
Provisioner

Parameters

Reclaim Policy

Volume Binding Mode
```

---

# What is a Provisioner?

Provisioner is responsible for:

```text
Creating Storage
```

---

Examples

```text
AWS EBS CSI Driver

AWS EFS CSI Driver

NFS Provisioner

Azure Disk CSI

GCP Persistent Disk CSI
```

---

# AWS EBS Provisioner

Example

```text
ebs.csi.aws.com
```

---

Meaning

```text
Create EBS Volumes
```

automatically.

---

# StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:
  name: gp3-storage

provisioner: ebs.csi.aws.com

parameters:

  type: gp3
```

---

Create

```bash
kubectl apply -f storageclass.yaml
```

---

Verify

```bash
kubectl get storageclass
```

---

Output

```text
gp3-storage
```

---

# Understanding Parameters

Parameters control:

```text
Disk Type

Performance

Storage Behavior
```

---

Example

```yaml
parameters:

  type: gp3
```

---

Creates

```text
AWS GP3 EBS Volumes
```

---

Other Examples

```text
gp2

gp3

io1

io2
```

---

# Default StorageClass

Most clusters have:

```text
Default StorageClass
```

---

Check

```bash
kubectl get storageclass
```

---

Output

```text
gp3 (default)
```

---

Meaning

```text
PVC Uses This StorageClass

Automatically
```

if none specified.

---

# Using StorageClass in PVC

Example

```yaml
apiVersion: v1

kind: PersistentVolumeClaim

metadata:
  name: mongodb-pvc

spec:

  storageClassName: gp3-storage

  accessModes:

  - ReadWriteOnce

  resources:

    requests:

      storage: 20Gi
```

---

PVC Requests

```text
20Gi Storage
```

---

StorageClass Handles

```text
Disk Creation
```

---

# StorageClass Workflow

Step 1

```text
StorageClass Exists
```

---

Step 2

```text
PVC Created
```

---

Step 3

```text
StorageClass Triggered
```

---

Step 4

```text
Disk Created
```

---

Step 5

```text
PV Created
```

---

Step 6

```text
PVC Bound
```

---

Step 7

```text
Pod Uses Storage
```

---

# Reclaim Policy

Defines:

```text
What Happens To Storage

After PVC Deleted
```

---

Common Policies

```text
Delete

Retain
```

---

# Delete Policy

Example

```text
PVC Deleted
      ↓

PV Deleted
      ↓

Disk Deleted
```

---

Useful For

```text
Temporary Environments
```

---

# Retain Policy

Example

```text
PVC Deleted
      ↓

PV Remains
      ↓

Disk Remains
```

---

Useful For

```text
Production Data
```

---

# Volume Binding Mode

Controls:

```text
When Storage Is Created
```

---

Common Modes

```text
Immediate

WaitForFirstConsumer
```

---

# Immediate

Creates Storage

```text
Immediately
```

after PVC creation.

---

# WaitForFirstConsumer

Creates Storage

```text
When Pod Is Scheduled
```

---

Commonly used in EKS.

---

Benefits

```text
Better Availability Zone Placement
```

---

# AWS EKS Example

StorageClass

```yaml
apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:
  name: gp3

provisioner: ebs.csi.aws.com

volumeBindingMode: WaitForFirstConsumer

parameters:

  type: gp3
```

---

Used By

```text
MongoDB

MySQL

PostgreSQL
```

---

# EBS CSI Driver

Modern EKS clusters use:

```text
AWS EBS CSI Driver
```

---

Purpose

```text
Provision EBS Volumes
```

for Kubernetes.

---

Architecture

```text
PVC
 ↓

StorageClass
 ↓

EBS CSI Driver
 ↓

AWS API
 ↓

EBS Volume
```

---

# Multiple StorageClasses

Cluster may contain:

```text
gp3-storage

efs-storage

premium-storage
```

---

Applications choose:

```yaml
storageClassName:
```

---

Example

Database

```text
gp3-storage
```

---

Shared Files

```text
efs-storage
```

---

# Production Example

MongoDB Deployment

Needs

```text
Persistent Storage
```

---

PVC

```text
Requests 20Gi
```

---

StorageClass

```text
gp3-storage
```

---

Automatically Creates

```text
20Gi EBS Volume
```

---

Result

```text
No Manual PV Creation
```

---

# StorageClass vs PV

| Feature | PV | StorageClass |
|----------|----------|----------|
| Represents Storage | Yes | No |
| Creates Storage | No | Yes |
| Manual Process | Usually | Automated |
| Provisioning | Static | Dynamic |
| Used By PVC | Yes | Yes |

---

# Common Problems

## No StorageClass Found

PVC

```text
Pending
```

---

Check

```bash
kubectl get storageclass
```

---

Verify

```text
Correct StorageClass Name
```

---

## EBS Volume Not Created

Check

```bash
kubectl describe pvc
```

---

Verify

```text
CSI Driver Installed
```

---

## PVC Stuck Pending

Possible Causes

```text
Wrong StorageClass

Provisioner Issue

CSI Driver Issue
```

---

# Troubleshooting

List StorageClasses

```bash
kubectl get storageclass
```

---

Describe

```bash
kubectl describe storageclass gp3-storage
```

---

Check PVC

```bash
kubectl get pvc
```

---

Describe PVC

```bash
kubectl describe pvc pvc-name
```

---

Check Events

```text
Provisioning

Binding

Errors
```

---

# Common Interview Questions

## What is a StorageClass?

### Short Answer

A StorageClass defines how storage should be dynamically provisioned.

### Production Example

Creating GP3 EBS volumes automatically.

---

## Why Do We Need StorageClasses?

### Short Answer

To automate storage provisioning.

### Detailed Explanation

StorageClasses eliminate manual disk and PV creation.

---

## What is a Provisioner?

### Short Answer

A component responsible for creating storage resources.

### Example

```text
ebs.csi.aws.com
```

---

## What is the Default StorageClass?

### Short Answer

The StorageClass automatically used when a PVC doesn't specify one.

---

## What is Reclaim Policy?

### Short Answer

Defines what happens to storage after a PVC is deleted.

### Options

```text
Delete

Retain
```

---

## What is WaitForFirstConsumer?

### Short Answer

Storage is provisioned only after a Pod is scheduled.

### Benefit

Improves availability zone placement.

---

# Key Takeaways

```text
StorageClass automates storage provisioning.

StorageClass acts as a storage template.

Provisioners create actual storage resources.

AWS EKS commonly uses the EBS CSI Driver.

StorageClasses eliminate manual PV creation.

PVCs can reference specific StorageClasses.

Default StorageClasses simplify storage requests.

Reclaim policies control storage cleanup behavior.

StorageClasses are the foundation of dynamic provisioning.
```