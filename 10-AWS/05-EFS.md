# Amazon Elastic File System (Amazon EFS)


## Introduction

Amazon Elastic File System (Amazon EFS) is a fully managed, elastic, highly available, and scalable Network File System (NFS) provided by AWS.

Unlike Amazon EBS, which can usually be attached to a single EC2 instance, Amazon EFS allows **multiple EC2 instances** to mount and access the same file system simultaneously.

EFS automatically scales as files are added or removed. There is no need to provision storage capacity in advance.

It is widely used for:

- Enterprise applications
- Kubernetes Persistent Volumes
- Web applications
- Content Management Systems
- Shared application storage
- Machine Learning workloads
- CI/CD shared storage
- Media rendering
- Analytics
- Home directories

---

# What is Amazon EFS?

Amazon EFS is a fully managed file storage service that supports the **Network File System (NFS)** protocol.

Applications connect to EFS just like they connect to a traditional Linux file server.

Because it uses NFS, applications usually require **no code changes**.

Think of Amazon EFS as a shared network drive available to multiple Linux servers simultaneously.

---

# Why Amazon EFS?

Consider an enterprise web application.

```text
EC2-1

↓

Uploads Image

↓

Local Disk
```

Now another server receives the request.

```text
EC2-2

↓

Image Missing

↓

Application Error
```

Every server has its own local storage.

Files are not shared.

Amazon EFS solves this problem.

```text
           EC2-1

             │

             │

        Amazon EFS

             │

             │

           EC2-2
```

Both servers access exactly the same files.

---

# Real-World Problem Statement

Imagine an e-commerce company.

The architecture contains:

- 20 EC2 Web Servers
- Auto Scaling
- Application Load Balancer

Customers upload:

- Product Images
- Invoices
- PDFs
- Videos

Without shared storage:

- Files become inconsistent
- Auto Scaling becomes difficult
- Data synchronization is required

Using Amazon EFS:

Every server immediately sees every uploaded file.

No synchronization jobs.

No rsync.

No shared FTP server.

---

# Complete Architecture Diagram

```text
                               Internet

                                   │

                             Amazon Route53

                                   │

                        Application Load Balancer

                                   │

          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
     EC2 Instance-1          EC2 Instance-2          EC2 Instance-3
          │                        │                        │
          ├─────────────── NFS v4 (Port 2049) ──────────────┤
          │                        │                        │
          └────────────────────────┼────────────────────────┘
                                   │
                         Amazon Elastic File System
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
  Mount Target (AZ-A)      Mount Target (AZ-B)      Mount Target (AZ-C)
         │                         │                         │
     Security Group          Security Group          Security Group
         │                         │                         │
      AWS KMS                 AWS Backup              CloudWatch
         │                         │                         │
 Encryption at Rest        Scheduled Snapshots      Monitoring & Alerts
```

---

# Internal Working

Unlike EBS, EFS is **not attached directly** to an EC2 instance.

Instead:

```text
Application

↓

Linux File System

↓

NFS Client

↓

TCP 2049

↓

Mount Target

↓

Amazon EFS

↓

Distributed Storage Layer
```

The Linux kernel communicates using the NFS protocol.

AWS manages:

- Storage
- Replication
- Durability
- Capacity
- Availability

The user only mounts the file system.

---

# Core Components

Amazon EFS consists of:

- File System
- Mount Targets
- NFS Protocol
- Security Groups
- Lifecycle Policies
- Performance Mode
- Throughput Mode
- Storage Classes
- Backup Integration
- Monitoring

---

# File System

The File System is the primary storage resource.

It stores:

- Files
- Directories
- Metadata
- Permissions
- Symbolic Links

Supports standard Linux file operations.

---

# Mount Target

A Mount Target acts as the network endpoint for EFS.

Each Availability Zone should have its own Mount Target.

Example:

```text
VPC

├── AZ-A

│     └── Mount Target

├── AZ-B

│     └── Mount Target

└── AZ-C

      └── Mount Target
```

Instances always connect to the Mount Target in their own Availability Zone.

---

# Network File System (NFS)

Amazon EFS uses:

```
NFS Version 4.1

or

NFS Version 4.0
```

Advantages:

- Shared Storage
- File Locking
- POSIX Permissions
- Multiple Clients
- Standard Linux Support

---

# Features

Amazon EFS provides:

- Fully Managed
- Elastic Capacity
- Automatic Scaling
- High Availability
- Multi-AZ
- Shared Storage
- Encryption
- Lifecycle Policies
- Backup Integration
- CloudWatch Monitoring

---

# Automatic Scaling

Unlike EBS:

```
EBS

100 GB

↓

Need Resize

↓

Modify Volume
```

Amazon EFS:

```
10 GB

↓

50 GB

↓

200 GB

↓

2 TB

↓

50 TB

↓

Automatically
```

No manual resizing.

---

# High Availability

EFS automatically replicates data across multiple Availability Zones.

```
AZ-A

↓

Replication

↓

AZ-B

↓

Replication

↓

AZ-C
```

If one Availability Zone fails, the file system remains available.

---

# Durability

Amazon EFS is designed for very high durability.

AWS automatically:

- Replicates data
- Detects hardware failures
- Replaces failed disks
- Performs integrity checks

No RAID configuration is required.

---

# Performance Modes

Amazon EFS supports two performance modes.

## 1. General Purpose

Best for:

- Web Servers
- CMS
- CI/CD
- Kubernetes
- Git Repositories

Provides:

- Low latency
- High throughput
- General workloads

---

## 2. Max I/O

Designed for:

- Big Data
- Analytics
- Machine Learning
- Massive Parallel Processing

Higher throughput

Slightly higher latency

---

# Throughput Modes

Amazon EFS supports:

### Elastic Throughput

Automatically adjusts throughput.

Recommended for most workloads.

---

### Provisioned Throughput

You specify throughput manually.

Useful when workload is predictable.

---

### Bursting Throughput

Throughput depends on file system size.

Smaller file systems receive burst credits.

---

# Storage Classes

Amazon EFS supports:

## Standard

Frequently accessed files.

Highest performance.

---

## Infrequent Access (IA)

Automatically moves inactive files.

Lower storage cost.

Higher access cost.

---

## Archive

Very low-cost storage.

For long-term retention.

Higher retrieval latency.

---

# Lifecycle Management

Lifecycle policies automatically move files.

Example:

```
30 Days

↓

Standard

↓

IA

↓

90 Days

↓

Archive
```

This reduces storage costs automatically.

---

# Security

Amazon EFS security consists of multiple layers:

- IAM
- Security Groups
- POSIX Permissions
- NFS Permissions
- KMS Encryption
- TLS Encryption

---

# Security Groups

Only allow:

```
TCP 2049

From

Application Servers
```

Never expose EFS publicly.

---

# Encryption

Amazon EFS supports:

### Encryption At Rest

Uses:

AWS KMS

---

### Encryption In Transit

Uses:

TLS

Protects data between EC2 and EFS.

---

# Monitoring

Amazon CloudWatch monitors:

- Storage Usage
- Throughput
- IOPS
- Client Connections
- Burst Credits
- Percent I/O Limit

Create alarms for:

- High Throughput
- Low Burst Credits
- Client Errors

---

# Backup

Amazon EFS integrates with:

AWS Backup

Backup Strategy:

Daily

↓

Weekly

↓

Monthly

↓

Cross-Region Copy

↓

Disaster Recovery