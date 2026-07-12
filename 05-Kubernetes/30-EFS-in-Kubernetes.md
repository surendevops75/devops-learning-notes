# EFS in Kubernetes

## Introduction

Earlier we learned:

```text
EBS

Persistent Volumes

Persistent Volume Claims

Storage Classes
```

---

EBS is excellent for:

```text
Databases

Operating Systems

High Performance Storage
```

---

However EBS has limitations:

```text
Single Node Access

Availability Zone Restrictions
```

---

Many applications require:

```text
Shared Storage

Multiple Pod Access

Cross AZ Access
```

---

Kubernetes solves this using:

```text
EFS
```

---

# What is EFS?

EFS stands for:

```text
Elastic File System
```

---

EFS is:

```text
Managed NFS Service
Provided By AWS
```

---

Think of EFS as:

```text
Shared Network Drive
```

for cloud applications.

---

# Why Do We Need EFS?

Suppose:

```text
5 Frontend Pods
```

need access to:

```text
Uploaded Files

Images

Reports

Documents
```

---

Using EBS

```text
Only One Node
Can Access Storage
```

---

Problem

```text
Files Not Shared
```

---

Solution

```text
EFS
```

---

# EFS Architecture

```text
Frontend Pod-1
       │

Frontend Pod-2
       │

Frontend Pod-3
       │

Frontend Pod-4
       │

Frontend Pod-5
       │

       ▼

       PVC
        ↓

        PV
        ↓

       EFS
```

---

All Pods access:

```text
Same Files
```

simultaneously.

---

# Characteristics of EFS

```text
Shared Storage

Elastic Storage

Managed Service

Multi AZ Access

RWX Support
```

---

# EFS Storage Growth

Unlike EBS:

```text
No Fixed Size
```

---

EFS automatically grows.

---

Can scale to:

```text
Petabytes
```

---

No manual resizing required.

---

# Access Modes

EFS commonly supports:

```text
ReadWriteMany (RWX)
```

---

Meaning

```text
Multiple Nodes

Read + Write
```

at the same time.

---

# Real World Use Cases

```text
User Uploads

Shared Reports

Application Assets

CMS Files

Shared Application Data
```

---

# EFS Requirements

Before Kubernetes can use EFS:

```text
Create EFS

Install EFS CSI Driver

Configure IAM Permissions

Configure Security Groups
```

---

# Create EFS

AWS Administrator creates:

```text
Elastic File System
```

---

Example

```text
fs-1234567890abcdef
```

---

This becomes backend storage.

---

# Security Groups

Unlike EBS:

```text
EFS Uses Network
```

---

Therefore EFS requires:

```text
Security Group
```

---

# NFS Port

EFS uses:

```text
TCP 2049
```

---

Security Group must allow:

```text
TCP 2049
```

from:

```text
EKS Node Security Group
```

---

Example

```text
EC2 Node SG
        ↓

Allow TCP 2049
        ↓

EFS SG
```

---

Without this:

```text
Mount Fails
```

---

# IAM Permissions

Worker Nodes require permissions to:

```text
Mount EFS

Describe EFS

Access Mount Targets
```

---

Without permissions:

```text
PVC Binding May Fail
```

---

# EFS CSI Driver

Kubernetes communicates with AWS using:

```text
AWS EFS CSI Driver
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

Mount EFS

Manage Access

Provision Storage
```

---

# Install EFS CSI Driver

Trainer Command

```bash
kubectl kustomize \
"github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-2.1" \
> public-ecr-driver.yaml
```

---

Apply

```bash
kubectl apply -f public-ecr-driver.yaml
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
efs-csi-controller

efs-csi-node
```

---

# Static Provisioning With EFS

Flow

```text
Create EFS
       ↓

Create PV
       ↓

Create PVC
       ↓

Mount Into Pod
```

---

# EFS Persistent Volume Example

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: efs-pv

spec:

  capacity:
    storage: 5Gi

  accessModes:
  - ReadWriteMany

  csi:

    driver: efs.csi.aws.com

    volumeHandle: fs-1234567890abcdef
```

---

Create

```bash
kubectl apply -f pv.yaml
```

---

# EFS PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: efs-pvc

spec:

  accessModes:
  - ReadWriteMany

  resources:

    requests:
      storage: 5Gi
```

---

Create

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

- name: shared-storage

  persistentVolumeClaim:

    claimName: efs-pvc
```

---

Mount

```yaml
volumeMounts:

- name: shared-storage

  mountPath: /shared-data
```

---

Result

```text
Multiple Pods
Use Same Storage
```

---

# Multi Pod Example

Pod-1

```text
Creates report.pdf
```

---

Pod-2

```text
Reads report.pdf
```

---

Pod-3

```text
Updates report.pdf
```

---

All access:

```text
Same EFS Storage
```

---

# Dynamic Provisioning With EFS

StorageClass can be used.

---

Flow

```text
StorageClass
       ↓

PVC
       ↓

PV Created
       ↓

EFS Mounted
```

---

Common for:

```text
Large Kubernetes Environments
```

---

# Production Example

Frontend Deployment

Replicas

```text
5 Pods
```

---

Requirements

```text
Shared Images

Shared Uploads

Shared Reports
```

---

Solution

```text
EFS
```

---

All pods access:

```text
Same Files
```

---

# EFS vs EBS Revisited

EFS

```text
Shared Storage

RWX

Network Based
```

---

EBS

```text
Single Node

RWO

Block Storage
```

---

# Common Problems

## Mount Failure

Cause

```text
Port 2049 Blocked
```

---

Verify

```text
Security Groups
```

---

## PVC Pending

Cause

```text
PV Missing

Driver Missing
```

---

Check

```bash
kubectl get pv
```

---

## Driver Not Running

Check

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
efs-csi-controller

efs-csi-node
```

---

## Permission Denied

Cause

```text
IAM Role Missing
```

---

Verify

```text
Node IAM Permissions
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
kubectl describe pvc efs-pvc
```

---

Verify CSI Driver

```bash
kubectl get pods -n kube-system
```

---

Verify Security Group

```text
Allow TCP 2049
```

---

Verify Mount Targets

```text
Available In Required AZs
```

---

# Common Interview Questions

## What is EFS?

### Short Answer

EFS is AWS managed network file storage that can be accessed by multiple servers simultaneously.

### Detailed Explanation

Elastic File System (EFS) provides shared, scalable, NFS-based storage that multiple Kubernetes nodes and pods can access at the same time.

### Production Example

Five frontend pods share uploaded files stored in EFS.

### Follow-Up Questions

- Is EFS block storage or file storage?
- Does EFS support RWX?
- Can EFS be accessed by multiple nodes?

---

## Why Do We Use EFS?

### Short Answer

To provide shared storage across multiple pods and nodes.

### Detailed Explanation

Applications often require common access to files, reports, and uploaded content. EFS enables this through network-based shared storage.

### Production Example

A reporting application stores PDFs in EFS and multiple frontend pods access the same files.

### Follow-Up Questions

- Can EBS provide shared access?
- What workloads benefit from EFS?
- Why is EFS commonly used with RWX?

---

## Why Does EFS Require Port 2049?

### Short Answer

Because EFS uses the NFS protocol.

### Detailed Explanation

NFS communication occurs over TCP port 2049. Worker nodes must be allowed to communicate with EFS using this port.

### Production Example

An EKS cluster could not mount EFS until TCP 2049 was opened between the node security group and the EFS security group.

### Follow-Up Questions

- What protocol does EFS use?
- Does EBS require network ports?
- Where should port 2049 be opened?

---

## What is the EFS CSI Driver?

### Short Answer

The EFS CSI Driver enables Kubernetes to mount and manage EFS storage.

### Detailed Explanation

The driver integrates Kubernetes with AWS EFS and handles mounting, access management, and storage provisioning.

### Production Example

A PVC requests shared storage and the EFS CSI Driver mounts the EFS filesystem into application pods.

### Follow-Up Questions

- What does CSI stand for?
- Is the CSI Driver mandatory?
- How do you verify the driver installation?

---

## Why is EFS Commonly Used With RWX?

### Short Answer

Because multiple pods can read and write the same storage simultaneously.

### Detailed Explanation

EFS is designed for shared file access and supports ReadWriteMany access mode, making it ideal for distributed applications.

### Production Example

Five frontend pods store and retrieve uploaded images from the same EFS filesystem.

### Follow-Up Questions

- What does RWX mean?
- Which AWS storage commonly uses RWO?
- Can databases use RWX storage?

---

# Key Takeaways

```text
EFS is AWS managed file storage.

EFS supports multiple node access.

EFS commonly uses RWX access mode.

EFS requires the EFS CSI Driver.

EFS requires TCP port 2049.

EFS requires Security Group configuration.

EFS automatically scales storage capacity.

EFS is ideal for shared files and uploads.

EFS is not typically used for high-performance databases.

EFS is one of the most common shared storage solutions in Kubernetes.
```