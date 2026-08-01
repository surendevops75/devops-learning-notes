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

---

---

# Integration with AWS Services

Amazon EFS integrates seamlessly with multiple AWS services.

| AWS Service | Integration | Purpose |
|-------------|-------------|----------|
| EC2 | Native | Shared File Storage |
| EKS | CSI Driver | Persistent Volumes |
| ECS | Task Storage | Shared Containers |
| Lambda | Mount EFS | Persistent Storage |
| AWS Backup | Native | Automated Backup |
| CloudWatch | Native | Monitoring |
| IAM | Native | Authentication |
| KMS | Native | Encryption |
| CloudTrail | API Calls | Auditing |

---

# AWS Console Walkthrough

## Step 1

Open

```
AWS Console

↓

Elastic File System (EFS)
```

---

## Step 2

Click

```
Create File System
```

---

## Step 3

Configure

- Name
- VPC
- Availability Zones
- Performance Mode
- Throughput Mode
- Encryption

---

## Step 4

Create Mount Targets

AWS automatically creates one mount target per selected Availability Zone.

---

## Step 5

Configure Security Groups

Allow

```
TCP 2049

From

Application Servers
```

Never allow

```
0.0.0.0/0
```

---

## Step 6

Review

↓

Create File System

---

# Mounting EFS on Linux

Install NFS utilities.

Amazon Linux

```bash
sudo yum install amazon-efs-utils -y
```

Ubuntu

```bash
sudo apt install nfs-common -y
```

Create mount directory.

```bash
sudo mkdir /efs
```

Mount.

```bash
sudo mount -t efs fs-12345678:/ /efs
```

Verify.

```bash
df -h
```

Permanent mount.

```bash
sudo vim /etc/fstab
```

Example

```text
fs-12345678:/ /efs efs defaults,_netdev 0 0
```

---

# AWS CLI

## Create File System

```bash
aws efs create-file-system \
--creation-token production-efs
```

---

## Describe File Systems

```bash
aws efs describe-file-systems
```

---

## Create Mount Target

```bash
aws efs create-mount-target \
--file-system-id fs-12345678 \
--subnet-id subnet-12345 \
--security-groups sg-12345
```

---

## Describe Mount Targets

```bash
aws efs describe-mount-targets \
--file-system-id fs-12345678
```

---

## Delete Mount Target

```bash
aws efs delete-mount-target \
--mount-target-id fsmt-123456
```

---

## Delete File System

```bash
aws efs delete-file-system \
--file-system-id fs-12345678
```

---

## Describe Lifecycle Configuration

```bash
aws efs describe-lifecycle-configuration \
--file-system-id fs-12345678
```

---

# Terraform Example

```hcl
resource "aws_efs_file_system" "efs" {

  creation_token = "production"

  encrypted = true

  performance_mode = "generalPurpose"

  throughput_mode = "elastic"

  tags = {

    Name = "production-efs"

  }

}
```

---

Create Mount Target

```hcl
resource "aws_efs_mount_target" "private" {

  file_system_id = aws_efs_file_system.efs.id

  subnet_id = aws_subnet.private.id

  security_groups = [

    aws_security_group.efs.id

  ]

}
```

---

Security Group

```hcl
resource "aws_security_group" "efs" {

  ingress {

    from_port = 2049

    to_port = 2049

    protocol = "tcp"

    security_groups = [

      aws_security_group.application.id

    ]

  }

}
```

---

# CloudFormation

```yaml
Resources:

  EFS:

    Type: AWS::EFS::FileSystem

    Properties:

      Encrypted: true

      PerformanceMode: generalPurpose

      ThroughputMode: elastic
```

---

# Python (Boto3)

Create File System

```python
import boto3

efs = boto3.client("efs")

response = efs.create_file_system(

    CreationToken="production"

)

print(response)
```

---

List File Systems

```python
response = efs.describe_file_systems()

print(response)
```

---

Delete File System

```python
efs.delete_file_system(

    FileSystemId="fs-123456"

)
```

---

# Kubernetes Integration

Amazon EFS is widely used in Kubernetes because multiple Pods can read and write the same storage simultaneously.

Unlike EBS:

- EBS → ReadWriteOnce
- EFS → ReadWriteMany (RWX)

This makes EFS ideal for:

- WordPress
- Jenkins
- Shared uploads
- Shared configuration
- ML workloads
- Media applications

---

# EFS CSI Driver

Architecture

```text
Pod

↓

Persistent Volume Claim

↓

Persistent Volume

↓

EFS CSI Driver

↓

Amazon EFS
```

---

Persistent Volume Example

```yaml
apiVersion: v1

kind: PersistentVolume

metadata:

  name: efs-pv

spec:

  capacity:

    storage: 100Gi

  accessModes:

  - ReadWriteMany

  csi:

    driver: efs.csi.aws.com

    volumeHandle: fs-12345678
```

---

Persistent Volume Claim

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

      storage: 100Gi
```

---

# Enterprise Production Architecture

```text
                         Internet
                             │
                      Amazon Route53
                             │
                   Application Load Balancer
                             │
              ┌──────────────┴──────────────┐
              │                             │
        EC2 / EKS (AZ-A)              EC2 / EKS (AZ-B)
              │                             │
              ├──────────── NFS ────────────┤
              │                             │
              └──────────────┬──────────────┘
                             │
                    Amazon EFS File System
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
   Mount Target A     Mount Target B     Mount Target C
          │                  │                  │
      AWS KMS          AWS Backup         CloudWatch
```

---

# Enterprise Use Cases

Amazon EFS is commonly used for:

- Kubernetes Persistent Volumes
- Jenkins Home Directory
- Shared Maven Repository
- GitLab Shared Storage
- WordPress Uploads
- Shared Configuration Files
- AI/ML Training Data
- Apache Spark
- Video Rendering
- Genomics
- Enterprise File Sharing

---

# Cost Optimization

Best Practices

- Use Lifecycle Policies
- Enable IA Storage
- Archive old files
- Delete unused file systems
- Monitor throughput
- Prefer Elastic Throughput
- Monitor CloudWatch metrics
- Use AWS Backup lifecycle policies

---

# Performance Optimization

- Use General Purpose mode for low latency
- Use Max I/O only for massive parallel workloads
- Mount locally within the same AZ
- Monitor Percent I/O Limit
- Avoid unnecessary metadata operations
- Use Elastic Throughput for variable workloads

---

# Best Practices

✅ Create Mount Targets in every Availability Zone

✅ Enable encryption

✅ Restrict Security Groups

✅ Use Lifecycle Policies

✅ Enable AWS Backup

✅ Monitor CloudWatch

✅ Mount using DNS

✅ Use IAM authentication where possible

✅ Tag all file systems

✅ Test disaster recovery regularly

---

# Common Mistakes

❌ Allowing NFS from the internet

❌ Using EFS for database storage

❌ Forgetting lifecycle policies

❌ Mounting through another AZ unnecessarily

❌ Disabling encryption

❌ Ignoring CloudWatch metrics

❌ Using Max I/O without need

❌ No backup strategy

---

# Troubleshooting

## Mount Failed

Check:

- Security Group
- Port 2049
- DNS
- Mount Target
- Subnet
- Route Table

---

## Slow Performance

Check:

- Performance Mode
- Throughput Mode
- CloudWatch
- Percent I/O Limit
- Network Latency

---

## Permission Denied

Verify:

- POSIX permissions
- File ownership
- IAM policy
- NFS configuration

---

## Cannot Connect

Verify:

- Mount Target
- Security Groups
- VPC
- NACL
- DNS Resolution

---

# Interview Questions

1. What is Amazon EFS?
2. Difference between EFS and EBS?
3. Difference between EFS and S3?
4. What protocol does EFS use?
5. What port does EFS use?
6. What are Mount Targets?
7. Explain Performance Modes.
8. Explain Throughput Modes.
9. What is Elastic Throughput?
10. What is ReadWriteMany?
11. How does EFS scale?
12. Can EFS be encrypted?
13. What is Lifecycle Management?
14. How does EFS integrate with EKS?
15. How does AWS Backup work with EFS?
16. What are Storage Classes?
17. How do you troubleshoot mount failures?
18. Why is EFS regional?
19. Can Windows use EFS?
20. When would you choose EFS over EBS?

---

# Production Scenarios

### Scenario 1

Pods cannot mount EFS.

Investigation:

- CSI Driver
- PVC
- PV
- Security Groups
- Mount Target

---

### Scenario 2

Application uploads fail.

Check:

- Mount Status
- Permissions
- Disk Usage
- Security Groups

---

### Scenario 3

High latency.

Review:

- Throughput Mode
- Percent I/O Limit
- CloudWatch Metrics

---

### Scenario 4

Cross-AZ traffic cost increases.

Verify:

- Mount Targets
- Node Placement
- Application Architecture

---

### Scenario 5

Backup restore fails.

Check:

- AWS Backup Vault
- Recovery Point
- IAM Permissions

---

# Cheat Sheet

| Item | Value |
|------|-------|
| Protocol | NFS v4.1 |
| Port | TCP 2049 |
| Shared Storage | Yes |
| Multiple EC2 | Yes |
| Auto Scaling | Yes |
| Multi-AZ | Yes |
| Encryption | KMS |
| Monitoring | CloudWatch |
| Backup | AWS Backup |
| Kubernetes | RWX |

---

# Summary

Amazon EFS is a fully managed, elastic, regional file storage service that provides shared access to multiple Linux-based workloads using the NFS protocol. It is ideal for applications requiring concurrent access from multiple EC2 instances or Kubernetes Pods.

For production environments, use Mount Targets in every Availability Zone, enable encryption, automate backups, monitor CloudWatch metrics, apply lifecycle policies for cost optimization, and secure access with Security Groups and least-privilege IAM controls.

---

# Key Takeaways

- Fully managed NFS file system
- Regional, Multi-AZ architecture
- Automatically scales storage
- Supports ReadWriteMany (RWX)
- Native integration with EC2, EKS, ECS, and Lambda
- KMS encryption and AWS Backup integration
- Lifecycle policies reduce storage costs
- Best suited for shared application data, CI/CD, and Kubernetes workloads