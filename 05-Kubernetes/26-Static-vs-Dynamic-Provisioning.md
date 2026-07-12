# Static vs Dynamic Provisioning

## Introduction

Applications need storage for:

```text
Databases

Uploads

Reports

Business Data
```

---

In Kubernetes, storage can be provisioned in two ways:

```text
Static Provisioning

Dynamic Provisioning
```

---

Understanding both approaches is important for:

```text
Production Environments

Storage Management

Interview Preparation
```

---

# What is Provisioning?

Provisioning means:

```text
Creating Storage

Making Storage Available

Attaching Storage To Applications
```

---

Example

```text
Create EBS Volume
      ↓

Use It In Kubernetes
```

---

This process is called:

```text
Storage Provisioning
```

---

# Two Provisioning Models

```text
Static Provisioning

Dynamic Provisioning
```

---

# Static Provisioning

Traditional Kubernetes storage approach.

---

Storage is created:

```text
Before Application Requests It
```

---

Flow

```text
Create Disk
      ↓

Create PV
      ↓

Create PVC
      ↓

Pod Uses PVC
```

---

Everything is created manually.

---

# Static Provisioning Architecture

```text
Storage Team
      ↓

Disk
      ↓

Kubernetes Admin
      ↓

PV
      ↓

DevOps Engineer
      ↓

PVC
      ↓

Pod
```

---

Storage already exists before PVC is created.

---

# Static Provisioning Workflow

Step 1

```text
Storage Team Creates Disk
```

Example

```text
10Gi EBS Volume
```

---

Step 2

```text
Kubernetes Admin Creates PV
```

---

Step 3

```text
DevOps Creates PVC
```

---

Step 4

```text
PVC Binds To PV
```

---

Step 5

```text
Pod Uses PVC
```

---

Application gets storage.

---

# Static Provisioning Prerequisites

Before static provisioning can work:

```text
Install EBS/EFS CSI Drivers

Configure IAM Permissions

Create Storage

Configure Network Access
```

---

## EBS Requirements

```text
Create EBS Volume

Install EBS CSI Driver

Worker Nodes Need IAM Permissions

Volume Must Be In Same Availability Zone As Worker Node
```

---

Example

```text
Node → us-east-1a

EBS Volume → us-east-1a
```

---

Otherwise

```text
Volume Mount Fails
```

---

## EFS Requirements

```text
Create EFS

Install EFS CSI Driver

Worker Nodes Need IAM Permissions

Allow TCP 2049
```

---

Security Group Rule

```text
EFS Security Group

Allow TCP 2049

From Worker Node Security Group
```

---

Otherwise

```text
Mount Fails
```
---

# Static Provisioning Example

Create Disk

```text
AWS EBS Volume
```

---

Create PV

```yaml
apiVersion: v1

kind: PersistentVolume

metadata:
  name: mongodb-pv

spec:

  capacity:
    storage: 20Gi

  accessModes:
  - ReadWriteOnce
```

---

Create PVC

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

      storage: 20Gi
```

---

Result

```text
PVC Binds To PV
```

---

# Advantages of Static Provisioning

```text
Full Control

Predictable

Simple To Understand
```

---

Useful for:

```text
Legacy Environments

Special Storage Requirements
```

---

# Disadvantages of Static Provisioning

```text
Manual Work

Slow Provisioning

Scaling Challenges

Administrative Overhead
```

---

Large environments become difficult to manage.

---

# Dynamic Provisioning

Modern Kubernetes approach.

---

Storage is created:

```text
Automatically
```

when required.

---

No manual PV creation required.

---

# Dynamic Provisioning Architecture

```text
Pod
 ↓

PVC
 ↓

StorageClass
 ↓

PV Created Automatically
 ↓

Disk Created Automatically
```

---

Storage appears on demand.

---

# Dynamic Provisioning Workflow

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
Disk Created Automatically
```

---

Step 5

```text
PV Created Automatically
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

# Dynamic Provisioning Prerequisites

Before dynamic provisioning can work:

```text
Install CSI Drivers

Configure IAM Permissions

Configure Network Access

Create StorageClass
```

---

## EBS Dynamic Provisioning

Requirements

```text
EBS CSI Driver

IAM Permissions

StorageClass
```

---

Workflow

```text
PVC Created
      ↓

StorageClass Invoked
      ↓

EBS Volume Created
      ↓

PV Created
      ↓

PVC Bound
```

---

## EFS Dynamic Provisioning

Requirements

```text
EFS CSI Driver

IAM Permissions

TCP 2049 Open

StorageClass
```

---

Workflow

```text
PVC Created
      ↓

StorageClass Invoked
      ↓

PV Created
      ↓

EFS Mounted
```

---

# Dynamic Provisioning Example

StorageClass

```yaml
apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:
  name: gp3-storage

provisioner: ebs.csi.aws.com
```

---

PVC

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

Result

```text
EBS Volume Created

PV Created

PVC Bound
```

Automatically.

---

# Real World Analogy

Static Provisioning

```text
Need Money
      ↓

Mother
      ↓

Father
      ↓

Wallet Already Exists
```

---

Dynamic Provisioning

```text
Need Money
      ↓

Mother
      ↓

PhonePe
      ↓

Money Created And Transferred
```

---

Kubernetes Equivalent

```text
Pod
 ↓

PVC
 ↓

StorageClass
 ↓

Disk Created Automatically
```

---

# Team Responsibilities

## Static Provisioning

Storage Team

```text
Create Disk

Expand Disk

Maintain Disk
```

---

Kubernetes Admin

```text
Create PV

Manage PV
```

---

DevOps Engineer

```text
Create PVC

Mount Storage
```

---

# Dynamic Provisioning

Storage Team

```text
Define Storage Standards
```

---

Kubernetes Admin

```text
Create StorageClass
```

---

DevOps Engineer

```text
Create PVC Only
```

---

Kubernetes handles the rest.

---

# Static vs Dynamic Provisioning

| Feature | Static | Dynamic |
|----------|---------|---------|
| Disk Creation | Manual | Automatic |
| PV Creation | Manual | Automatic |
| PVC Creation | Manual | Manual |
| StorageClass Required | No | Yes |
| Scalability | Limited | High |
| Operational Effort | High | Low |
| Production Usage | Less Common | Very Common |

---

# AWS EKS Example

MongoDB Deployment

Requires

```text
20Gi Storage
```

---

Static

```text
Create EBS
 ↓
Create PV
 ↓
Create PVC
```

---

Dynamic

```text
Create PVC
 ↓
StorageClass
 ↓
EBS Created Automatically
```

---

Most EKS environments use:

```text
Dynamic Provisioning
```

---

# Why Dynamic Provisioning Became Popular?

Suppose

```text
100 Applications
```

need storage.

---

Static Provisioning

```text
100 Disks

100 PVs

100 PVCs
```

---

Dynamic Provisioning

```text
StorageClass
      ↓

Applications Create PVCs
```

---

Far easier to manage.

---

# Common Problems

## Static Provisioning

PVC Stuck

```text
Pending
```

---

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

# Dynamic Provisioning

PVC Stuck

```text
Pending
```

---

Cause

```text
StorageClass Missing

Provisioner Issue

CSI Driver Issue
```

---

Check

```bash
kubectl get storageclass
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

Check StorageClasses

```bash
kubectl get storageclass
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

# Production Recommendation

For modern environments:

```text
Dynamic Provisioning
```

is preferred.

---

Benefits

```text
Automation

Scalability

Reduced Manual Effort

Faster Deployments
```

---

Static provisioning is mainly used when:

```text
Specific Existing Storage

Special Compliance Requirements

Legacy Systems
```

are involved.

---

# Common Interview Questions

## What is Static Provisioning?

### Short Answer

Storage resources are created manually before applications request them.

### Example

Create EBS → Create PV → Create PVC.

---

## What is Dynamic Provisioning?

### Short Answer

Storage is automatically created when a PVC requests it.

### Example

PVC → StorageClass → EBS Created Automatically.

---

## Which Provisioning Method Is Preferred?

### Short Answer

Dynamic Provisioning.

### Reason

```text
Automation

Scalability

Less Administration
```

---

## What Resource Enables Dynamic Provisioning?

### Short Answer

StorageClass.

---

## Does Dynamic Provisioning Create PVs?

### Short Answer

Yes.

### Process

```text
PVC
 ↓

StorageClass
 ↓

PV Created Automatically
```

---

# Key Takeaways

```text
Provisioning means creating and assigning storage.

Static provisioning requires manual disk and PV creation.

Dynamic provisioning automates storage creation.

StorageClass is the foundation of dynamic provisioning.

Dynamic provisioning is the preferred approach in modern Kubernetes environments.

EKS commonly uses dynamic provisioning with the EBS CSI Driver.

Dynamic provisioning reduces operational overhead and improves scalability.

Understanding both models is important for production support and interviews.
```