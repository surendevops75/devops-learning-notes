# EBS vs EFS

## Introduction

When deploying stateful applications in Kubernetes, choosing the correct storage solution is critical.

---

AWS provides two commonly used storage services:

```text
EBS

EFS
```

---

Both provide persistent storage.

---

However they solve different problems.

---

Choosing the wrong storage may lead to:

```text
Performance Issues

Scaling Problems

Application Failures

Higher Costs
```

---

# What is EBS?

EBS stands for:

```text
Elastic Block Store
```

---

EBS is:

```text
Block Storage
```

---

Think of EBS as:

```text
Virtual Hard Disk
Attached To Server
```

---

Common Uses

```text
Operating Systems

Databases

Application Storage
```

---

Examples

```text
MongoDB

MySQL

PostgreSQL

Redis

Timeseries Databases
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
Network File Storage
```

---

Think of EFS as:

```text
Shared Network Drive
```

---

Common Uses

```text
Shared Files

Uploads

Reports

Application Assets

Shared Configuration
```

---

# EBS Architecture

```text
Application
      ↓

PVC
      ↓

PV
      ↓

EBS Volume
      ↓

Single Node
```

---

Storage is attached to:

```text
One Node
```

at a time.

---

# EFS Architecture

```text
Application-1
       ↓

Application-2
       ↓

Application-3
       ↓

PVC
       ↓

PV
       ↓

EFS
```

---

Multiple nodes access:

```text
Same Storage
```

simultaneously.

---

# Availability Zone Requirement

## EBS

EBS volumes are:

```text
Availability Zone Specific
```

---

Example

```text
EBS Volume

us-east-1b
```

---

Can only be attached within:

```text
us-east-1b
```

---

Cannot be mounted from:

```text
us-east-1a

us-east-1c
```

---

## EFS

EFS is:

```text
Network Based Storage
```

---

Can be accessed from:

```text
Multiple Availability Zones
```

---

No AZ restriction.

---

# Access Pattern

## EBS

Supports

```text
Single Server Access
```

---

Common Access Mode

```text
ReadWriteOnce (RWO)
```

---

Example

```text
One Database Server
```

using one EBS volume.

---

## EFS

Supports

```text
Multiple Server Access
```

---

Common Access Mode

```text
ReadWriteMany (RWX)
```

---

Example

```text
Multiple Pods

Same Storage
```

---

# Performance

## EBS

Performance

```text
Very Fast
```

---

Reason

```text
Block Storage
```

---

Low Latency

High Throughput

---

Ideal For

```text
Databases

Operating Systems

Transactional Workloads
```

---

## EFS

Performance

```text
Slower Than EBS
```

---

Reason

```text
Network File System
```

---

Ideal For

```text
Shared Files

Documents

Reports
```

---

# Storage Size

## EBS

Size is:

```text
Allocated Explicitly
```

---

Example

```text
100 GB

500 GB

1 TB
```

---

Expansion possible.

---

However size is:

```text
Manually Managed
```

---

## EFS

Size is:

```text
Elastic
```

---

Automatically grows.

---

Can scale up to:

```text
Petabytes
```

---

No manual disk expansion required.

---

# Filesystem

## EBS

Supports

```text
XFS

EXT4

Other Linux Filesystems
```

---

Filesystem chosen during formatting.

---

## EFS

Uses

```text
NFS
```

---

Filesystem type fixed.

---

Applications access storage through:

```text
NFS Protocol
```

---

# Security Groups

## EBS

No dedicated security group.

---

Storage attached directly to:

```text
EC2 Instance
```

---

No network communication required.

---

## EFS

Requires:

```text
Security Group
```

---

Reason

```text
Network Storage
```

---

Traffic flows through network.

---

# EFS Port

NFS uses:

```text
Port 2049
```

---

Security Group must allow:

```text
TCP 2049
```

---

Example

```text
EFS Security Group

Allow TCP 2049

From EC2 Node Security Group
```

---

Otherwise:

```text
Mount Fails
```

---

# Database Storage

## EBS

Recommended

```text
Yes
```

---

Examples

```text
MongoDB

MySQL

PostgreSQL

Redis
```

---

Reason

```text
High Performance

Low Latency
```

---

## EFS

Recommended

```text
No
```

---

Reason

```text
Higher Latency

Network Storage
```

---

Not ideal for database workloads.

---

# Shared File Storage

## EBS

Recommended

```text
No
```

---

Single Node Limitation.

---

## EFS

Recommended

```text
Yes
```

---

Examples

```text
User Uploads

Reports

Shared Documents

Images
```

---

# Kubernetes Access Modes

## EBS

Typically

```text
ReadWriteOnce
```

---

Meaning

```text
One Node

Read + Write
```

---

## EFS

Typically

```text
ReadWriteMany
```

---

Meaning

```text
Multiple Nodes

Read + Write
```

---

# Real Production Examples

## MongoDB

Storage

```text
EBS
```

---

Reason

```text
High IOPS

Low Latency
```

---

## MySQL

Storage

```text
EBS
```

---

Reason

```text
Database Workload
```

---

## User Uploads

Storage

```text
EFS
```

---

Reason

```text
Multiple Applications
Need Same Files
```

---

## Shared Reports

Storage

```text
EFS
```

---

Reason

```text
Multi Node Access
```

---

# EBS vs EFS Comparison

| Feature | EBS | EFS |
|----------|----------|----------|
| Storage Type | Block Storage | File Storage |
| Access | Single Node | Multiple Nodes |
| Performance | Faster | Slower |
| Latency | Low | Higher |
| Database Usage | Excellent | Not Recommended |
| Shared Storage | No | Yes |
| Access Mode | RWO | RWX |
| AZ Restriction | Yes | No |
| Security Group Required | No | Yes |
| Port Requirement | No | 2049 |
| Filesystem | XFS/EXT4 | NFS |
| Scaling | Manual | Automatic |
| Maximum Size | Fixed Allocation | Petabyte Scale |

---

# Storage Selection Guide

Question

```text
Running Database?
```

---

Use

```text
EBS
```

---

Question

```text
Need Shared Storage?
```

---

Use

```text
EFS
```

---

Question

```text
Need Multiple Pods
To Access Same Data?
```

---

Use

```text
EFS
```

---

Question

```text
Need Highest Performance?
```

---

Use

```text
EBS
```

---

# Common Problems

## EBS Volume Mount Failure

Cause

```text
Wrong Availability Zone
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

## EFS Mount Failure

Cause

```text
Port 2049 Blocked
```

---

Check

```text
Security Groups
```

---

## EFS Access Failure

Cause

```text
EFS CSI Driver Missing
```

---

Verify

```bash
kubectl get pods -n kube-system
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

Verify Node AZ

```bash
kubectl get nodes -o wide
```

---

Check EFS Security Group

```text
Allow TCP 2049
```

---

Verify CSI Drivers

```bash
kubectl get pods -n kube-system
```

---

# Common Interview Questions

## What is the Difference Between EBS and EFS?

### Short Answer

EBS is block storage designed for single-node access, while EFS is file storage designed for multi-node access.

### Detailed Explanation

EBS provides high-performance block storage attached to a single node. EFS provides shared network storage that multiple nodes can access simultaneously.

### Production Example

MongoDB uses EBS for database storage, while application uploads are stored in EFS for shared access across multiple pods.

### Follow-Up Questions

- Which one supports RWX?
- Which one is faster?
- Which one is suitable for databases?

---

## Why is EBS Preferred for Databases?

### Short Answer

Because EBS provides low latency and high performance.

### Detailed Explanation

Databases perform frequent read and write operations and require fast storage. EBS provides block-level storage optimized for such workloads.

### Production Example

MongoDB in EKS stores its data on GP3 EBS volumes.

### Follow-Up Questions

- Can databases run on EFS?
- What storage type does EBS provide?
- What is IOPS?

---

## Why Does EFS Require Port 2049?

### Short Answer

Because EFS uses the NFS protocol.

### Detailed Explanation

NFS communicates over TCP port 2049. EFS security groups must allow this traffic from worker nodes.

### Production Example

An EKS cluster mounts EFS successfully only after allowing TCP 2049 from the node security group.

### Follow-Up Questions

- What is NFS?
- Does EBS require port 2049?
- Why does EFS need security groups?

---

## Can EBS Be Accessed By Multiple Nodes?

### Short Answer

Typically no.

### Detailed Explanation

EBS volumes are generally mounted to a single node and commonly use ReadWriteOnce access mode.

### Production Example

A MongoDB pod uses a single EBS volume attached to one worker node.

### Follow-Up Questions

- What access mode does EBS commonly use?
- Which AWS storage supports RWX?
- Why is EFS better for shared storage?

---

## When Should You Choose EFS?

### Short Answer

When multiple applications or nodes need shared access to the same files.

### Detailed Explanation

EFS provides network-based shared storage accessible from multiple nodes simultaneously.

### Production Example

Multiple frontend pods access the same reports and uploaded files stored in EFS.

### Follow-Up Questions

- Does EFS support RWX?
- Can EFS scale automatically?
- Is EFS suitable for databases?

---

# Key Takeaways

```text
EBS is block storage.

EFS is file storage.

EBS is faster than EFS.

EBS is ideal for databases.

EFS is ideal for shared files.

EBS is Availability Zone specific.

EFS can be accessed from multiple Availability Zones.

EFS requires TCP port 2049.

EBS commonly uses RWO.

EFS commonly uses RWX.

Understanding EBS and EFS is essential for Kubernetes storage design.
```