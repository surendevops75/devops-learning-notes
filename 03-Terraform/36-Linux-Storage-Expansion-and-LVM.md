# Linux Storage Expansion and LVM

## Introduction

In production environments, applications continuously generate:

```text
Logs

Application Data

Reports

Backups

Temporary Files
```

Over time, disk space may become insufficient.

Common symptoms:

```text
Disk Full Errors

Application Failures

Database Issues

Log Generation Stops

Server Performance Problems
```

To solve this problem, storage must be expanded.

---

# Production Scenario

Suppose:

```text
/home
```

filesystem is almost full.

Current Size:

```text
50 GB
```

Requirement:

```text
Increase To 80 GB
```

Additional:

```text
30 GB
```

must be allocated.

---

# Linux Storage Architecture

Understanding storage layers is important.

```text
EBS Volume
     ↓
Partition
     ↓
Physical Volume (PV)
     ↓
Volume Group (VG)
     ↓
Logical Volume (LV)
     ↓
Filesystem
     ↓
Mount Point
```

Example:

```text
AWS EBS
     ↓
/dev/nvme0n1
     ↓
Partition 4
     ↓
RootVG
     ↓
homeVol
     ↓
XFS
     ↓
/home
```

---

# Real World Analogy

Consider a building.

```text
Land
   ↓
Building
   ↓
Floor
   ↓
Room
```

Linux Storage:

```text
Disk
   ↓
Partition
   ↓
VG
   ↓
LV
   ↓
Filesystem
```

---

# Step 1 - Check Existing Storage

Command:

```bash
sudo lsblk
```

---

# What is lsblk?

lsblk means:

```text
List Block Devices
```

Purpose:

```text
Display Disks

Display Partitions

Display Mount Points

Display LVM Volumes
```

---

# Example Output

```text
nvme0n1
├─nvme0n1p1
├─nvme0n1p2
├─nvme0n1p3
└─nvme0n1p4
```

---

# Why Run lsblk?

To verify:

```text
Available Disks

Partition Layout

Filesystem Mapping
```

---

# Production Example

Before making any changes:

```bash
sudo lsblk
```

Always verify current layout.

---

# Step 2 - Expand EBS Volume

In AWS:

```text
EC2
   ↓
Volumes
   ↓
Modify Volume
   ↓
Increase Size
```

Example:

```text
50 GB → 80 GB
```

AWS increases disk size.

Linux does not automatically use the extra space.

---

# Step 3 - Expand Partition

Command:

```bash
sudo growpart /dev/nvme0n1 4
```

---

# Understanding the Command

Disk:

```text
/dev/nvme0n1
```

Partition:

```text
4
```

Meaning:

```text
Expand Partition 4
```

to consume newly available disk space.

---

# What Does growpart Do?

Before:

```text
Disk = 80 GB

Partition = 50 GB
```

After:

```text
Disk = 80 GB

Partition = 80 GB
```

---

# Verify Expansion

Command:

```bash
sudo lsblk
```

Verify partition size increased.

---

# Step 4 - Understanding LVM

Most production Linux servers use:

```text
LVM
```

Meaning:

```text
Logical Volume Manager
```

---

# Why LVM?

Without LVM:

```text
Fixed Partitions
```

With LVM:

```text
Flexible Storage

Easy Expansion

Easy Management
```

---

# LVM Components

## Physical Volume (PV)

Actual disk storage.

Example:

```text
/dev/nvme0n1p4
```

---

## Volume Group (VG)

Collection of storage.

Example:

```text
RootVG
```

---

## Logical Volume (LV)

Virtual partition.

Example:

```text
homeVol
```

---

# Storage Hierarchy

```text
Disk
  ↓
PV
  ↓
VG
  ↓
LV
  ↓
Filesystem
```

---

# Step 5 - Expand Logical Volume

Command:

```bash
sudo lvextend -L +30G /dev/mapper/RootVG-homeVol
```

---

# Command Breakdown

```text
-L
```

means:

```text
Size
```

---

```text
+30G
```

means:

```text
Add 30 GB
```

---

Logical Volume:

```text
/dev/mapper/RootVG-homeVol
```

---

# Result

Before:

```text
50 GB
```

After:

```text
80 GB
```

Logical volume is expanded.

---

# Important Point

At this stage:

```text
Filesystem Still 50 GB
```

Only logical volume increased.

Filesystem must also be expanded.

---

# Step 6 - Expand Filesystem

Command:

```bash
sudo xfs_growfs /home
```

---

# Why xfs_growfs?

Most modern Linux servers use:

```text
XFS Filesystem
```

Filesystem expansion requires:

```text
xfs_growfs
```

---

# What Happens?

Before:

```text
Filesystem = 50 GB
```

After:

```text
Filesystem = 80 GB
```

---

# Final Storage Expansion Workflow

```text
Increase EBS Volume
          ↓
growpart
          ↓
Partition Expanded
          ↓
lvextend
          ↓
Logical Volume Expanded
          ↓
xfs_growfs
          ↓
Filesystem Expanded
```

---

# Complete Production Procedure

Step 1

```bash
sudo lsblk
```

---

Step 2

Increase EBS Volume from AWS Console.

---

Step 3

```bash
sudo growpart /dev/nvme0n1 4
```

---

Step 4

```bash
sudo lsblk
```

---

Step 5

```bash
sudo lvextend -L +30G /dev/mapper/RootVG-homeVol
```

---

Step 6

```bash
sudo xfs_growfs /home
```

---

Step 7

Verify space:

```bash
df -h
```

---

# Production Example

Problem:

```text
Application Logs Filled /home
```

Available:

```text
2 GB Remaining
```

Solution:

```text
Increase EBS Volume

Expand Partition

Expand LVM

Expand Filesystem
```

No reboot required.

---

# Common Interview Questions

## What is lsblk?

### Short Answer

lsblk displays block devices, partitions, and mount points.

### Detailed Explanation

It helps administrators understand the storage layout of Linux systems.

### Production Example

Checking partitions before extending storage.

### Follow-Up Questions

- What is the difference between lsblk and df -h?
- Can lsblk show LVM volumes?

---

## What is LVM?

### Short Answer

LVM stands for Logical Volume Manager and provides flexible storage management.

### Detailed Explanation

LVM abstracts physical disks into logical volumes, making expansion and management easier.

### Production Example

Expanding /home from 50 GB to 80 GB without rebuilding partitions.

### Follow-Up Questions

- What are PV, VG, and LV?
- Why is LVM preferred in production?

---

## What Does growpart Do?

### Short Answer

growpart expands an existing partition.

### Detailed Explanation

After increasing disk size, growpart allows the partition to consume newly available space.

### Production Example

Expanding partition 4 after increasing EBS volume size.

### Follow-Up Questions

- Why is growpart required?
- Does growpart expand filesystems?

---

## What Does lvextend Do?

### Short Answer

lvextend increases the size of a logical volume.

### Detailed Explanation

It allocates additional storage from the Volume Group to the Logical Volume.

### Production Example

Adding 30 GB to homeVol.

### Follow-Up Questions

- Does lvextend expand the filesystem?
- What happens if the VG has insufficient space?

---

## What Does xfs_growfs Do?

### Short Answer

xfs_growfs expands an XFS filesystem.

### Detailed Explanation

After increasing the logical volume, the filesystem must also be expanded to use the additional space.

### Production Example

Expanding /home filesystem from 50 GB to 80 GB.

### Follow-Up Questions

- Why is xfs_growfs needed?
- What filesystem uses resize2fs instead?

---

# Key Takeaways

```text
Production servers frequently require storage expansion.

Linux storage consists of Disk → Partition → PV → VG → LV → Filesystem.

lsblk displays storage layout.

growpart expands partitions.

LVM provides flexible storage management.

lvextend expands logical volumes.

xfs_growfs expands XFS filesystems.

Storage expansion can often be performed without rebooting.

Always verify changes using lsblk and df -h.

Understanding LVM is a common Linux and DevOps interview topic.
```