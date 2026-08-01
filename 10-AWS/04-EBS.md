# Amazon Elastic Block Store (EBS)

---

# Introduction

Amazon Elastic Block Store (Amazon EBS) is a high-performance, persistent block storage service designed for use with Amazon EC2 instances.

Unlike instance store volumes, EBS volumes retain data even after an EC2 instance is stopped or restarted. EBS is ideal for applications requiring durable, low-latency storage such as databases, enterprise applications, Kubernetes worker nodes, CI/CD servers, and file systems.

---

# What is EBS?

Amazon EBS is a block-level storage device that can be attached to an EC2 instance.

Think of EBS as a virtual hard disk.

It provides:

- Persistent Storage
- High Availability
- Low Latency
- Encryption
- Snapshots
- Dynamic Resizing

---

# EBS Architecture

```text
          Amazon EC2 Instance

                 │

          Amazon EBS Volume

                 │

          Amazon S3 Snapshots
```

---

# Why EBS?

EBS provides:

- Persistent storage
- High durability
- Automatic replication within an Availability Zone
- High IOPS
- Encryption
- Backup using Snapshots
- Dynamic resizing
- Easy recovery

---

# Characteristics

- Block Storage
- Persistent
- AZ Specific
- Network Attached Storage
- Highly Available
- Low Latency
- Encrypted
- Scalable

---

# EBS Volume Lifecycle

```text
Create Volume

↓

Attach Volume

↓

Read / Write Data

↓

Snapshot

↓

Resize

↓

Detach

↓

Delete
```

---

# Types of EBS Volumes

Amazon EBS provides several volume types.

### SSD Volumes

- gp3
- gp2
- io1
- io2

### HDD Volumes

- st1
- sc1

---

# General Purpose SSD (gp3)

Recommended for most workloads.

Features:

- Independent IOPS
- Independent Throughput
- Lower cost than gp2
- Better performance

Common Uses:

- Web Servers
- Kubernetes Nodes
- Jenkins
- GitLab
- Application Servers

---

# General Purpose SSD (gp2)

Older generation SSD.

Performance depends on volume size.

Mostly replaced by gp3.

---

# Provisioned IOPS SSD (io1 / io2)

Designed for mission-critical workloads.

Suitable for:

- Oracle Database
- SQL Server
- SAP
- Financial Systems

Provides:

- Very high IOPS
- Consistent latency
- High durability

---

# Throughput Optimized HDD (st1)

Designed for:

- Big Data
- Log Processing
- Hadoop
- Streaming

Not suitable for boot volumes.

---

# Cold HDD (sc1)

Low-cost storage.

Used for:

- Archives
- Backups
- Rarely accessed data

---

# Volume Comparison

| Type | Storage | Performance | Use Case |
|-------|----------|-------------|----------|
| gp3 | SSD | High | General workloads |
| gp2 | SSD | Medium | Legacy workloads |
| io1/io2 | SSD | Very High | Databases |
| st1 | HDD | High Throughput | Big Data |
| sc1 | HDD | Low Cost | Archive |

---

# Root Volume

Every EC2 instance contains a Root Volume.

Stores:

- Operating System
- Boot Loader
- System Files

Usually:

- gp3
- gp2

---

# Additional Volumes

You can attach multiple EBS volumes.

Example:

```text
EC2

├── Root Volume

├── Database Volume

├── Log Volume

└── Backup Volume
```

---

# EBS Attachment

```text
Volume

↓

Attach

↓

EC2

↓

Mount

↓

Read / Write
```

---

# Multi-Attach

Supported only on io1/io2 volumes.

Allows one volume to be attached to multiple EC2 instances simultaneously.

Common Use:

- Clustered applications

---

# Encryption

EBS supports encryption using AWS KMS.

Encrypts:

- Data at rest
- Data in transit
- Snapshots
- Restored volumes

No application changes required.

---

# Snapshots

Snapshots are incremental backups stored in Amazon S3.

Benefits:

- Backup
- Disaster Recovery
- Migration
- Restore
- Volume cloning

---

# Snapshot Workflow

```text
EBS Volume

↓

Snapshot

↓

Amazon S3

↓

Restore

↓

New EBS Volume
```

---

# Snapshot Characteristics

- Incremental
- Durable
- Encrypted
- Region-specific
- Can be copied across Regions

---

# Volume Expansion

Amazon EBS supports online resizing.

You can increase:

- Storage Size
- IOPS
- Throughput

without recreating the volume.

---

# Volume Resize Process

```text
Modify Volume

↓

Increase Size

↓

Extend File System

↓

Application Continues
```

---

# IOPS

IOPS = Input Output Operations Per Second.

Higher IOPS means:

- Faster databases
- Better application performance
- Lower latency

---

# Throughput

Throughput measures:

MB/sec transferred.

Higher throughput benefits:

- Analytics
- Log processing
- Streaming

---

# Availability

EBS volumes are automatically replicated within the same Availability Zone.

This protects against hardware failures.

---

# High Availability

For Production:

```text
Application

↓

Auto Scaling

↓

Multiple EC2

↓

Multiple EBS

↓

Snapshots

↓

Cross-Region Backup
```

---

# EBS Monitoring

Monitor using CloudWatch.

Important Metrics:

- Read Ops
- Write Ops
- Read Bytes
- Write Bytes
- Queue Length
- Burst Balance
- Volume Idle Time

---

# Performance Optimization

Use:

- gp3 for most workloads
- io2 for databases
- Enable EBS optimization
- Monitor Queue Length
- Separate Logs and Database volumes
- Use RAID when required

---

# Production Example

A Jenkins Server:

```text
EC2

├── Root Volume

├── Jenkins Home

├── Docker Storage

├── Build Artifacts

↓

Snapshots

↓

Amazon S3
```

---

# Backup Strategy

Daily:

- Snapshot

Weekly:

- Copy Snapshot

Monthly:

- Cross Region Backup

Yearly:

- Archive

---

# Best Practices

- Prefer gp3 over gp2
- Enable encryption
- Schedule snapshots
- Monitor CloudWatch metrics
- Separate application and log volumes
- Tag all volumes
- Delete unused snapshots
- Enable EBS Optimization
- Test backup restoration
- Use lifecycle policies

---

# Common Mistakes

- Forgetting backups
- Not encrypting volumes
- Using gp2 for new workloads
- Leaving unattached volumes
- Ignoring CloudWatch metrics
- Deleting snapshots accidentally
- Using HDD for databases
- Not extending filesystem after resizing

---

# Troubleshooting Checklist

If EBS performance is poor:

Check:

- IOPS
- Throughput
- Queue Length
- Burst Balance
- CloudWatch Metrics
- Filesystem usage
- Mount status
- Volume attachment
- Encryption status
- AWS Health Dashboard

---

# Interview Questions

1. What is Amazon EBS?
2. Difference between EBS and Instance Store?
3. What is gp3?
4. Difference between gp2 and gp3?
5. What are io1 and io2 volumes?
6. What is Multi-Attach?
7. What are Snapshots?
8. Where are Snapshots stored?
9. Can EBS volumes be resized?
10. Explain IOPS.
11. Explain Throughput.
12. Can EBS volumes be encrypted?
13. How do you migrate an EBS volume?
14. How do you back up an EC2 instance?
15. How do you troubleshoot poor EBS performance?

---

# Key Takeaways

- Amazon EBS provides persistent block storage for EC2 instances.
- gp3 is the recommended SSD volume type for most workloads.
- io2 is designed for mission-critical, high-performance applications.
- Snapshots are incremental backups stored in Amazon S3.
- EBS volumes can be resized online without recreating them.
- Enable encryption using AWS KMS to protect data at rest.
- Monitor IOPS, throughput, and queue length with CloudWatch.
- Regular snapshots and lifecycle policies are essential for backup and disaster recovery.
- Use EBS optimization and appropriate volume types to achieve consistent production performance.