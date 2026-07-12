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

A StorageClass is a Kubernetes resource that defines how persistent storage should be dynamically provisioned.

### Detailed Explanation

StorageClass acts as a template for storage creation. It specifies the provisioner, parameters, reclaim policy, and volume binding mode used when a PVC requests storage.

Instead of manually creating disks and Persistent Volumes, Kubernetes can automatically create storage using the StorageClass configuration.

### Production Example

In EKS, a StorageClass named:

```yaml
storageClassName: gp3
```

uses the AWS EBS CSI Driver to automatically create GP3 EBS volumes whenever a PVC requests storage.

### Follow-Up Questions

- Why do we need StorageClasses?
- What happens if no StorageClass exists?
- Can multiple StorageClasses exist in a cluster?
- What is a default StorageClass?

---

## Why Do We Need StorageClasses?

### Short Answer

StorageClasses automate storage provisioning and eliminate manual PV creation.

### Detailed Explanation

Without StorageClasses:

```text
Create Disk
 ↓
Create PV
 ↓
Create PVC
```

must be done manually.

With StorageClasses:

```text
PVC
 ↓
StorageClass
 ↓
Disk Created Automatically
```

This reduces operational effort and improves scalability.

### Production Example

In a cluster hosting 100 microservices, manually creating 100 PVs is difficult. StorageClasses allow developers to request storage through PVCs while Kubernetes provisions disks automatically.

### Follow-Up Questions

- What problems do StorageClasses solve?
- How do they support self-service infrastructure?
- Are StorageClasses mandatory for dynamic provisioning?

---

## What is a Provisioner?

### Short Answer

A provisioner is the component responsible for creating storage resources.

### Detailed Explanation

When a PVC requests storage through a StorageClass, the provisioner communicates with the storage platform and creates the required storage resource.

Examples:

```text
ebs.csi.aws.com

efs.csi.aws.com

disk.csi.azure.com
```

### Production Example

In EKS:

```text
ebs.csi.aws.com
```

creates EBS volumes automatically.

### Follow-Up Questions

- What is the difference between an in-tree and CSI provisioner?
- Can a StorageClass exist without a provisioner?
- How does the provisioner communicate with AWS?

---

## What is the AWS EBS CSI Driver?

### Short Answer

The AWS EBS CSI Driver enables Kubernetes to dynamically provision and manage EBS volumes.

### Detailed Explanation

CSI (Container Storage Interface) is a standard used by Kubernetes to integrate with storage systems.

The EBS CSI Driver:

```text
Creates EBS Volumes

Attaches Volumes

Mounts Volumes

Deletes Volumes
```

based on PVC requests.

### Production Example

MongoDB PVC requests:

```text
20Gi Storage
```

The EBS CSI Driver creates a 20Gi EBS volume automatically.

### Follow-Up Questions

- What does CSI stand for?
- Why did Kubernetes move to CSI?
- What happens if the CSI driver is missing?

---

## What is a Default StorageClass?

### Short Answer

A default StorageClass is automatically used when a PVC does not specify a StorageClass.

### Detailed Explanation

Clusters can designate one StorageClass as default.

When a PVC is created without:

```yaml
storageClassName:
```

Kubernetes automatically uses the default StorageClass.

### Production Example

EKS often has:

```text
gp3 (default)
```

configured.

A PVC requesting storage automatically gets a GP3 EBS volume.

### Follow-Up Questions

- How do you identify the default StorageClass?
- Can multiple default StorageClasses exist?
- How do you change the default StorageClass?

---

## What is Reclaim Policy?

### Short Answer

Reclaim Policy determines what happens to storage after a PVC is deleted.

### Detailed Explanation

Common policies:

```text
Delete

Retain
```

Delete:

```text
PVC Deleted
 ↓
PV Deleted
 ↓
Disk Deleted
```

Retain:

```text
PVC Deleted
 ↓
PV Remains
 ↓
Disk Remains
```

### Production Example

Production database volumes commonly use:

```text
Retain
```

to prevent accidental data loss.

### Follow-Up Questions

- What is the default reclaim policy?
- Which policy is safer for production databases?
- Can reclaim policies be modified later?

---

## What is WaitForFirstConsumer?

### Short Answer

WaitForFirstConsumer delays storage provisioning until a Pod is scheduled.

### Detailed Explanation

Instead of creating storage immediately after PVC creation, Kubernetes waits until the Pod is assigned to a node.

This helps place storage in the correct Availability Zone.

### Production Example

In EKS:

```yaml
volumeBindingMode: WaitForFirstConsumer
```

ensures the EBS volume is created in the same AZ where the Pod will run.

### Follow-Up Questions

- Why is this important for EBS?
- What happens with Immediate mode?
- Which mode is recommended in EKS?

---

## Explain the Complete Dynamic Provisioning Flow

### Short Answer

PVC triggers StorageClass, which triggers the provisioner to create storage automatically.

### Detailed Explanation

Flow:

```text
PVC Created
      ↓

StorageClass Selected
      ↓

Provisioner Triggered
      ↓

EBS Volume Created
      ↓

PV Created
      ↓

PVC Bound
      ↓

Pod Mounted
```

### Production Example

Roboshop MongoDB deployment:

```text
MongoDB Pod
      ↓

PVC
      ↓

StorageClass (gp3)
      ↓

EBS CSI Driver
      ↓

20Gi EBS Volume
```

### Follow-Up Questions

- At what stage is the PV created?
- Who creates the EBS volume?
- What happens if provisioning fails?

---

## StorageClass vs Persistent Volume

### Short Answer

StorageClass creates storage; PV represents storage.

### Detailed Explanation

StorageClass:

```text
Blueprint / Template
```

PV:

```text
Actual Storage Resource
```

StorageClass is used for dynamic provisioning, while PV represents the provisioned storage.

### Production Example

GP3 StorageClass provisions an EBS volume and Kubernetes creates a corresponding PV.

### Follow-Up Questions

- Can PV exist without StorageClass?
- Can StorageClass exist without PV?
- Which resource is consumed by a PVC?

---