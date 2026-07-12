# EBS Static Provisioning in Kubernetes

## Introduction

Earlier we learned:

```text
Persistent Volumes (PV)

Persistent Volume Claims (PVC)

StorageClasses

Static Provisioning

Dynamic Provisioning
```

---

In this section we will focus on:

```text
AWS EBS

Static Provisioning

Manual Disk Creation

Manual PV Creation
```

---

This approach is commonly used when:

```text
Specific Storage Is Required

Existing Disk Must Be Reused

Compliance Requirements Exist

Migration Projects
```

---

# What is EBS?

EBS stands for:

```text
Elastic Block Store
```

---

It is AWS block storage used by:

```text
EC2 Instances

Databases

Operating Systems

Stateful Applications
```

---

Think of EBS as:

```text
Virtual Hard Disk
```

for cloud servers.

---

# Why Use EBS?

Characteristics

```text
High Performance

Low Latency

Persistent Storage

Block Storage
```

---

Common Use Cases

```text
MongoDB

MySQL

PostgreSQL

Redis

Timeseries Databases
```

---

# Static Provisioning Overview

Storage is created manually.

---

Flow

```text
Storage Team
      ↓

Create EBS Volume
      ↓

Kubernetes Admin
      ↓

Create PV
      ↓

DevOps Engineer
      ↓

Create PVC
      ↓

Pod Uses Storage
```

---

Everything is manually managed.

---

# Real Production Workflow

Suppose Roboshop MongoDB requires:

```text
100 GB Storage
```

---

Step 1

DevOps Engineer raises request.

---

Example

```text
Need 100GB EBS Volume
For MongoDB Database
```

---

Step 2

Delivery Manager approves request.

---

Step 3

Storage Team Lead approves request.

---

Step 4

Storage Team creates EBS volume.

---

Step 5

Kubernetes Administrator creates PV.

---

Step 6

DevOps Engineer creates PVC.

---

Step 7

MongoDB Pod mounts storage.

---

# Sample Storage Request Email

Subject:

```text
Request For 100GB EBS Volume
For Roboshop MongoDB
```

---

Body

```text
Hi Storage Team,

Please provision a 100GB EBS volume
for the Roboshop MongoDB workload
in the EKS production environment.

Required Capacity: 100GB
Environment: Production
Purpose: Database Storage

Kindly proceed after approval.

Thanks,
DevOps Team
```

---

# EBS Architecture

```text
MongoDB Pod
       ↓

PVC
       ↓

PV
       ↓

EBS Volume
```

---

Application never directly accesses:

```text
EBS Volume
```

---

Kubernetes manages storage access.

---

# EBS Availability Zone Requirement

EBS volumes are:

```text
AZ Specific
```

---

Example

```text
EBS Volume

us-east-1b
```

---

Can be attached only within:

```text
us-east-1b
```

---

Example

```text
Volume

vol-08a1628c9a820c0e0

AZ

1b
```

---

Important Rule

```text
Pod And EBS Volume
Must Be In Same AZ
```

---

# Why Same AZ?

EBS is:

```text
Block Storage
```

---

AWS does not allow:

```text
Cross AZ Attachment
```

---

Otherwise mount fails.

---

# EKS Requirements For EBS

Before mounting EBS volumes:

```text
Install EBS CSI Driver

Configure IAM Permissions

Create EBS Volume

Create PV
```

---

# EBS CSI Driver

Kubernetes communicates with AWS using:

```text
EBS CSI Driver
```

---

CSI Means

```text
Container Storage Interface
```

---

Responsibilities

```text
Create Volumes

Attach Volumes

Mount Volumes

Delete Volumes
```

---

# Install EBS CSI Driver

Command

```bash
kubectl apply -k \
"github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.53"
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
ebs-csi-controller

ebs-csi-node
```

---

# IAM Permissions

Worker Nodes require permissions to:

```text
Create Volumes

Attach Volumes

Detach Volumes

Describe Volumes
```

---

Without IAM permissions:

```text
Volume Mount Fails
```

---

# Creating EBS Volume

Storage Team creates:

```text
100 GB EBS Volume
```

---

Example

```text
vol-08a1628c9a820c0e0
```

---

This becomes the backend storage.

---

# Persistent Volume

PV represents:

```text
Physical EBS Volume
```

inside Kubernetes.

---

# PV Example

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mongodb-pv

spec:

  capacity:
    storage: 100Gi

  accessModes:
  - ReadWriteOnce

  csi:

    driver: ebs.csi.aws.com

    volumeHandle: vol-08a1628c9a820c0e0
```

---

Create PV

```bash
kubectl apply -f pv.yaml
```

---

Verify

```bash
kubectl get pv
```

---

# Persistent Volume Claim

PVC requests storage.

---

Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mongodb-pvc

spec:

  accessModes:
  - ReadWriteOnce

  resources:

    requests:
      storage: 100Gi
```

---

Create PVC

```bash
kubectl apply -f pvc.yaml
```

---

Verify

```bash
kubectl get pvc
```

---

Expected

```text
Bound
```

---

# Mount PVC Into Pod

Example

```yaml
volumes:

- name: mongodb-storage

  persistentVolumeClaim:

    claimName: mongodb-pvc
```

---

Mount

```yaml
volumeMounts:

- name: mongodb-storage

  mountPath: /data/db
```

---

Result

```text
MongoDB Uses EBS Storage
```

---

# Complete Static Provisioning Flow

```text
Raise Storage Request
         ↓

Approval
         ↓

Storage Team Creates EBS
         ↓

Kubernetes Admin Creates PV
         ↓

DevOps Creates PVC
         ↓

PVC Binds To PV
         ↓

Pod Mounts Storage
```

---

# Advantages

```text
Full Control

Predictable Storage

Existing Volume Reuse

Compliance Friendly
```

---

# Disadvantages

```text
Manual Process

Slow Provisioning

Administrative Overhead

Not Scalable
```

---

# Production Example

Roboshop MongoDB

Requirements

```text
100GB Storage

Persistent Database

Production Environment
```

---

Workflow

```text
EBS Volume Created
        ↓

PV Created
        ↓

PVC Created
        ↓

MongoDB Mounted
```

---

Data survives:

```text
Pod Restart

Pod Recreation
```

---

# Troubleshooting

Check PV

```bash
kubectl get pv
```

---

Check PVC

```bash
kubectl get pvc
```

---

Describe PVC

```bash
kubectl describe pvc mongodb-pvc
```

---

Check Events

```bash
kubectl describe pv mongodb-pv
```

---

Verify CSI Driver

```bash
kubectl get pods -n kube-system
```

---

# Common Problems

## PVC Pending

Cause

```text
No Matching PV
```

---

Check

```bash
kubectl get pv
```

---

## Volume Not Mounted

Cause

```text
EBS CSI Driver Missing
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

## AttachVolume Failed

Cause

```text
Volume In Different AZ
```

---

Verify

```text
Node AZ

Volume AZ
```

---

Must match.

---

## Permission Denied

Cause

```text
IAM Role Missing
```

---

Verify

```text
EC2 IAM Permissions
```

---

# Common Interview Questions

## What is EBS?

### Short Answer

EBS is AWS block storage used for EC2 instances and stateful applications.

### Detailed Explanation

Elastic Block Store (EBS) provides persistent block storage with high performance and low latency. It is commonly used for databases and operating systems.

### Production Example

MongoDB running in EKS stores its database files on a 100GB EBS volume.

### Follow-Up Questions

- Is EBS block storage or file storage?
- Can EBS be mounted to multiple nodes?
- Is EBS suitable for databases?

---

## What is Static Provisioning?

### Short Answer

Static provisioning is a storage provisioning method where disks and PVs are created manually before applications request them.

### Detailed Explanation

Storage teams create disks, Kubernetes administrators create PVs, and DevOps engineers create PVCs that bind to those PVs.

### Production Example

Storage team creates a 100GB EBS volume, Kubernetes admin creates a PV, and MongoDB uses a PVC to access it.

### Follow-Up Questions

- What are the steps involved?
- Who creates the PV?
- How does it differ from dynamic provisioning?

---

## Why Must EBS Be In The Same AZ?

### Short Answer

EBS volumes can only be attached within the same Availability Zone.

### Detailed Explanation

AWS does not allow EBS volumes to be attached across Availability Zones.

### Production Example

A MongoDB pod running in us-east-1b can only mount an EBS volume created in us-east-1b.

### Follow-Up Questions

- What happens if AZs differ?
- Does EFS have this limitation?
- Why is this important in EKS?

---

## What is the EBS CSI Driver?

### Short Answer

The EBS CSI Driver allows Kubernetes to manage EBS volumes.

### Detailed Explanation

The driver communicates with AWS APIs to create, attach, mount, and delete EBS volumes.

### Production Example

An EKS cluster uses the EBS CSI Driver to attach a MongoDB volume to worker nodes.

### Follow-Up Questions

- What does CSI stand for?
- Is the CSI Driver mandatory?
- What happens if the driver is unavailable?

---

# Key Takeaways

```text
EBS is AWS block storage.

EBS is commonly used for databases.

EBS volumes are Availability Zone specific.

Static provisioning requires manual disk creation.

PV represents the EBS volume inside Kubernetes.

PVC is used by applications to consume storage.

The EBS CSI Driver is required for volume operations.

Worker nodes need IAM permissions for EBS access.

Static provisioning provides control but requires manual effort.
```