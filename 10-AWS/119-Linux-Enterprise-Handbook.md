# Linux Enterprise Handbook

# Chapter 1 - Linux Fundamentals & Enterprise Architecture

Linux is the foundation of modern IT infrastructure. Almost every modern DevOps platform runs on Linux, making it one of the most important technologies for DevOps Engineers, Cloud Engineers, SREs, and Platform Engineers.

Today, Linux powers:

- Cloud Platforms
- Kubernetes Clusters
- Docker Containers
- CI/CD Servers
- Databases
- Web Servers
- Monitoring Platforms
- Automation Servers
- Enterprise Applications

More than **90% of cloud workloads** run on Linux.

A DevOps Engineer must have a strong understanding of:

- Linux architecture
- System administration
- Process management
- Networking
- Storage
- Security
- Performance tuning
- Production troubleshooting

---

# Why Linux?

Linux has become the industry standard because it is:

- Open Source
- Stable
- Secure
- Highly Performant
- Scalable
- Portable
- Enterprise Ready

Almost every major cloud platform uses Linux, including:

- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)

Most DevOps tools also run on Linux:

- Docker
- Kubernetes
- Jenkins
- GitHub Actions Runners
- GitLab Runner
- Prometheus
- Grafana
- ELK Stack
- Nginx
- Apache

---

# Linux in Enterprise Infrastructure

A modern enterprise infrastructure typically looks like this:

```text
Applications
      │
      ▼
Docker Containers
      │
      ▼
Kubernetes
      │
      ▼
Linux Operating System
      │
      ▼
Virtual Machine / Hypervisor
      │
      ▼
Physical Server
      │
      ▼
Storage & Network
```

Regardless of how modern the infrastructure is, everything ultimately depends on the Linux operating system.

---

# What is Linux?

Linux is an operating system that manages computer hardware and provides services for applications.

Its primary responsibilities include:

- Managing CPU resources
- Managing memory
- Managing storage
- Managing networking
- Managing devices
- Managing users
- Managing processes
- Enforcing security

Applications do not communicate directly with hardware. Instead, every application interacts with the Linux kernel.

---

# What is the Linux Kernel?

The **Kernel** is the core component of the Linux operating system.

It acts as an interface between applications and hardware.

The kernel is responsible for:

- Process Management
- Memory Management
- Device Management
- File System Management
- Network Stack
- Security
- Scheduling

Without the kernel, the operating system cannot function.

---

# Linux Architecture

Linux consists of multiple layers.

```text
+----------------------------+
|     User Applications      |
+----------------------------+
|         Shell              |
+----------------------------+
|      System Calls          |
+----------------------------+
|      Linux Kernel          |
+----------------------------+
|        Hardware            |
+----------------------------+
```

Each layer has a specific responsibility.

---

# User Space vs Kernel Space

Linux separates execution into two areas.

## User Space

User Space contains applications such as:

- Chrome
- Nginx
- Docker CLI
- kubectl
- Jenkins
- Python
- Bash

Applications running here **cannot directly access hardware**.

---

## Kernel Space

Kernel Space contains:

- CPU Scheduler
- Memory Manager
- Device Drivers
- Network Stack
- File System

Only the kernel can communicate directly with hardware.

Applications request services through **System Calls**.

---

# What are System Calls?

A System Call is the mechanism through which an application requests services from the Linux kernel.

Example:

```text
Application

↓

System Call

↓

Kernel

↓

Hardware
```

Examples of system calls include:

- Reading a file
- Writing to disk
- Opening a network socket
- Creating a process
- Allocating memory

---

# Linux Boot Process

Every Linux server follows the same boot sequence.

```text
Power On

↓

BIOS / UEFI

↓

Boot Loader (GRUB)

↓

Linux Kernel

↓

initramfs

↓

systemd

↓

System Services

↓

Login Prompt
```

If any stage fails, the operating system may fail to boot.

---

# BIOS / UEFI

The BIOS (or UEFI on modern systems) is firmware stored on the motherboard.

Its responsibilities include:

- Detect hardware
- Perform POST (Power-On Self-Test)
- Identify boot devices
- Start the boot loader

---

# Boot Loader (GRUB)

The boot loader is responsible for loading the Linux kernel into memory.

Common boot loaders include:

- GRUB2
- systemd-boot

The boot loader also passes boot parameters to the kernel.

---

# initramfs

**initramfs** (Initial RAM File System) is a temporary file system loaded into memory during boot.

It performs tasks such as:

- Loading required drivers
- Detecting storage devices
- Mounting the root filesystem
- Preparing the operating system

---

# systemd

systemd is the default initialization system for most modern Linux distributions.

Its responsibilities include:

- Starting services
- Managing dependencies
- Managing targets
- Restarting failed services
- Collecting logs
- Managing system startup

Most enterprise Linux servers use **systemd**.

---

# Linux Filesystem Hierarchy

Linux stores everything under a single root directory.

```text
/

├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

Understanding this hierarchy is essential for Linux administration.

---

# Important Directories

| Directory | Purpose |
|-----------|---------|
| `/etc` | System configuration files |
| `/var` | Logs, cache, spool files |
| `/home` | User home directories |
| `/root` | Root user's home directory |
| `/boot` | Kernel and boot files |
| `/tmp` | Temporary files |
| `/opt` | Third-party software |
| `/usr` | User applications and libraries |
| `/proc` | Kernel and process information |
| `/dev` | Device files |

---

# Linux File Types

Linux treats everything as a file.

Common file types include:

- Regular File
- Directory
- Symbolic Link
- Block Device
- Character Device
- Socket
- Named Pipe (FIFO)

Each file type serves a different purpose within the operating system.

---

# Linux Permissions Overview

Every file has three permission groups:

- Owner
- Group
- Others

Each group can have:

- Read (r)
- Write (w)
- Execute (x)

Permissions are one of the most important security mechanisms in Linux.

---

# Linux in a DevOps Environment

A typical production DevOps workflow looks like this:

```text
Developer

↓

GitHub

↓

Jenkins

↓

Docker

↓

Amazon EKS

↓

Linux Worker Nodes

↓

Application Pods
```

Although developers interact with Kubernetes, every Kubernetes node is ultimately a Linux server.

---

# Real Production Example

Consider an online banking application.

```text
Customer

↓

Application Load Balancer

↓

Amazon EKS

↓

Linux Worker Nodes

↓

Payment Service

↓

Database
```

Even though customers interact with the application through a browser, every request is processed by applications running on Linux.

---

# Enterprise Best Practices

- Build a strong Linux foundation before learning Kubernetes.
- Understand the complete Linux boot process.
- Learn the Linux filesystem hierarchy thoroughly.
- Understand how the kernel manages system resources.
- Use Bash as your primary shell.
- Learn systemd service management.
- Practice on enterprise Linux distributions such as RHEL, Rocky Linux, Ubuntu Server, or Amazon Linux.
- Understand Linux before automating it with Ansible or Shell scripting.

---

# Common Mistakes

- Memorizing commands without understanding Linux internals.
- Ignoring the Linux boot process.
- Confusing User Space with Kernel Space.
- Editing critical system files without backups.
- Misunderstanding filesystem locations.
- Ignoring Linux security fundamentals.
- Learning Kubernetes before mastering Linux.

---

# Interview Questions

## Basic

1. What is Linux?
2. Why is Linux widely used in DevOps?
3. What is the Linux kernel?
4. Explain Linux architecture.
5. What is the difference between User Space and Kernel Space?

## Intermediate

1. Explain the Linux boot process.
2. What is initramfs?
3. What is systemd?
4. Explain the Linux filesystem hierarchy.
5. Why is everything treated as a file in Linux?

## Advanced

1. Design a highly available Linux platform for hosting Kubernetes worker nodes in Amazon EKS.
2. Explain how Linux forms the foundation for Docker, Kubernetes, Jenkins, GitHub Actions, and cloud infrastructure.
3. A financial organization is deploying mission-critical applications on Amazon EKS. Explain how Linux architecture, the kernel, systemd, and the filesystem hierarchy contribute to building a secure, scalable, and reliable production platform.

---

# Chapter 2 - Linux File System, Files, Directories & Navigation

The Linux File System is one of the most important topics for every DevOps Engineer.

Whether you are working with:

- Docker
- Kubernetes
- Jenkins
- Terraform
- Ansible
- AWS EC2

you interact with the Linux file system every day.

A strong understanding of the Linux file system is essential for managing production servers efficiently.

---

# What is a File System?

A file system is the method Linux uses to organize and store data on storage devices.

It provides:

- File Storage
- Directory Structure
- Permissions
- Metadata
- Access Control

Without a file system, Linux cannot store or retrieve data.

---

# Linux File System Architecture

```text
Application

↓

System Calls

↓

Virtual File System (VFS)

↓

File System

↓

Storage Device
```

The Virtual File System (VFS) allows Linux to support multiple file system types through a common interface.

---

# Everything is a File

One of the core principles of Linux is:

> **Everything is a file.**

Examples include:

- Regular files
- Directories
- Hard disks
- USB devices
- Network sockets
- Processes
- Pipes
- Terminals

This design simplifies administration and scripting.

---

# File System Hierarchy Standard (FHS)

Linux follows the **Filesystem Hierarchy Standard (FHS)**, which defines the purpose of each directory.

```text
/

├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

Every Linux administrator should know this hierarchy.

---

# Root Directory (/)

The root directory (`/`) is the top-level directory in Linux.

Every file and directory exists beneath it.

Example:

```text
/

├── etc

├── home

├── var

├── usr
```

There is only **one root directory**.

---

# Home Directory

Each user has a personal home directory.

Example:

```text
/home/surendra

/home/admin

/home/devops
```

User-specific files are stored here.

---

# Root User Home

Do not confuse:

```text
/

with

/root
```

`/root`

is the home directory

for the **root user**.

---

# Important Directories

| Directory | Purpose |
|-----------|---------|
| `/etc` | System configuration files |
| `/var` | Logs, cache, mail, spool |
| `/usr` | User applications and libraries |
| `/tmp` | Temporary files |
| `/opt` | Third-party software |
| `/boot` | Kernel and boot files |
| `/dev` | Device files |
| `/proc` | Process and kernel information |
| `/sys` | Kernel device information |
| `/run` | Runtime system information |

---

# Linux File Types

Linux supports multiple file types.

| File Type | Description |
|-----------|-------------|
| Regular File | Stores data |
| Directory | Stores files and directories |
| Symbolic Link | Shortcut to another file |
| Character Device | Keyboard, terminal |
| Block Device | Hard disks, SSDs |
| Socket | Network communication |
| FIFO (Named Pipe) | Process communication |

---

# Absolute Path

An absolute path starts from the root directory.

Example:

```text
/home/devops/project/app.py
```

It always begins with:

```text
/
```

---

# Relative Path

A relative path starts from the current working directory.

Example:

```text
project/app.py
```

Relative paths are shorter and commonly used in scripts.

---

# Present Working Directory (pwd)

Displays the current directory.

Example:

```bash
pwd
```

Output:

```text
/home/devops
```

Useful when navigating large directory structures.

---

# List Files (ls)

Displays files and directories.

Basic command:

```bash
ls
```

Common options:

```bash
ls -l
```

Long listing format.

```bash
ls -a
```

Show hidden files.

```bash
ls -lh
```

Human-readable file sizes.

```bash
ls -ltr
```

Sort by modification time.

---

# Change Directory (cd)

Move between directories.

Examples:

```bash
cd /etc

cd /var/log

cd ~

cd ..

cd -
```

Understanding navigation is essential for system administration.

---

# Create Directories

Create a directory:

```bash
mkdir project
```

Create nested directories:

```bash
mkdir -p project/dev/backend
```

The `-p` option creates parent directories if they do not exist.

---

# Remove Directories

Remove an empty directory:

```bash
rmdir project
```

Remove recursively:

```bash
rm -r project
```

Force deletion:

```bash
rm -rf project
```

⚠️ Use `rm -rf` carefully in production.

---

# Create Files

Create an empty file:

```bash
touch file.txt
```

Create multiple files:

```bash
touch app.py config.yaml Dockerfile
```

---

# Copy Files

Copy a file:

```bash
cp source.txt destination.txt
```

Copy directories:

```bash
cp -r source destination
```

Useful during backups and deployments.

---

# Move Files

Rename a file:

```bash
mv old.txt new.txt
```

Move a file:

```bash
mv app.py /opt/apps/
```

The same command performs both move and rename operations.

---

# Delete Files

Delete a file:

```bash
rm file.txt
```

Delete multiple files:

```bash
rm file1.txt file2.txt
```

Always verify the path before deletion.

---

# View File Contents

Display entire file:

```bash
cat file.txt
```

View large files:

```bash
less file.txt
```

Display first 10 lines:

```bash
head file.txt
```

Display last 10 lines:

```bash
tail file.txt
```

Monitor logs continuously:

```bash
tail -f /var/log/messages
```

This is widely used in production troubleshooting.

---

# Hidden Files

Files beginning with a dot (`.`) are hidden.

Examples:

```text
.bashrc

.profile

.gitignore

.ssh
```

Display them using:

```bash
ls -a
```

---

# File Metadata

Each file stores metadata such as:

- Owner
- Group
- Permissions
- Size
- Last Modified Time
- Inode Number

This information is useful for troubleshooting.

---

# Inodes

Every file has an inode.

An inode stores:

- Ownership
- Permissions
- File Size
- Timestamps
- Block Locations

The filename itself is **not** stored inside the inode.

---

# Enterprise Example

Consider a Jenkins server.

```text
/var/lib/jenkins

↓

Jobs

↓

Workspace

↓

Build Artifacts

↓

Logs
```

Understanding the filesystem helps locate build failures quickly.

---

# Kubernetes Example

A Kubernetes worker node stores data in locations such as:

```text
/var/lib/kubelet

↓

Pods

↓

Volumes

↓

Container Data
```

A DevOps Engineer frequently inspects these directories during troubleshooting.

---

# Enterprise Best Practices

- Learn the purpose of every major Linux directory.
- Always use absolute paths in production scripts.
- Avoid using `rm -rf` without verification.
- Store application data in appropriate directories.
- Monitor log files under `/var/log`.
- Use `less` instead of `cat` for large files.
- Understand inode usage for troubleshooting.
- Keep temporary files under `/tmp`.

---

# Common Mistakes

- Confusing `/` with `/root`.
- Deleting files using incorrect paths.
- Using relative paths in production automation.
- Ignoring hidden configuration files.
- Editing files directly without backups.
- Storing application data in system directories.
- Forgetting that `rm` permanently deletes files.

---

# Interview Questions

## Basic

1. What is the Linux File System?
2. What does "Everything is a file" mean?
3. What is the difference between an absolute path and a relative path?
4. Explain the purpose of `/etc`, `/var`, and `/usr`.
5. What are hidden files?

## Intermediate

1. What is an inode?
2. Explain the File System Hierarchy Standard (FHS).
3. Why is `/var/log` important?
4. Difference between `cp` and `mv`.
5. Explain `tail -f` and where you use it in production.

## Advanced

1. Design a Linux filesystem layout for a production Kubernetes worker node and explain where logs, containers, volumes, and application data should reside.
2. A production Jenkins server is running out of disk space. Explain how you would investigate the filesystem, identify large directories, locate build artifacts, and safely reclaim space without affecting running jobs.
3. Explain how knowledge of the Linux filesystem hierarchy helps troubleshoot Docker, Kubernetes, Jenkins, and application issues in enterprise environments.

---

# Chapter 3 - Linux Users, Groups, Permissions & Access Control

Linux is a **multi-user operating system**, meaning multiple users can access the same system simultaneously.

To ensure security and proper resource management, Linux provides:

- User Management
- Group Management
- File Permissions
- Ownership
- Access Control Lists (ACLs)
- Privilege Management

Understanding these concepts is essential for managing production Linux servers securely.

---

# Multi-User Architecture

Linux allows multiple users to work on the same system while maintaining isolation.

```text
User 1
    │
User 2
    │
User 3
    │
    ▼
Linux Operating System
    │
    ▼
Files, Processes, Resources
```

Each user has independent permissions and access rights.

---

# Types of Users

Linux supports three main types of users.

| User Type | Description |
|-----------|-------------|
| Root User | Full administrative privileges |
| System User | Used by services and applications |
| Regular User | Used by human users |

---

# Root User

The **root** user is the superuser.

Capabilities include:

- Install software
- Manage users
- Modify system files
- Start and stop services
- Change permissions
- Access all files

Example:

```text
Username: root

Home Directory: /root

UID: 0
```

Avoid using the root account for daily activities.

---

# Regular Users

Regular users perform day-to-day tasks.

Example:

```text
surendra

devops

developer

admin
```

Their home directories are located under:

```text
/home
```

---

# System Users

System users run background services.

Examples:

```text
nginx

mysql

postgres

jenkins

docker
```

These users usually do not have login access.

---

# User Identification (UID)

Each user has a unique User ID (UID).

Examples:

| UID | Purpose |
|-----|----------|
| 0 | Root User |
| 1-999 | System Users (varies by distribution) |
| 1000+ | Regular Users |

Linux uses the UID internally rather than the username.

---

# Groups

Groups simplify permission management.

Instead of assigning permissions to individual users, permissions are assigned to groups.

Example:

```text
Developers

↓

Alice

Bob

Charlie
```

All members inherit the group's permissions.

---

# Group Identification (GID)

Every group has a unique Group ID (GID).

Example:

```text
developers

↓

GID 1001
```

---

# Primary Group

Each user belongs to one primary group.

Example:

```text
User

↓

surendra

↓

Primary Group

↓

developers
```

New files are generally assigned to the user's primary group.

---

# Secondary Groups

A user can belong to multiple secondary groups.

Example:

```text
surendra

↓

developers

docker

sudo

kubernetes
```

This allows flexible access control.

---

# User Management Commands

Create a user:

```bash
useradd devops
```

Create a user with a home directory:

```bash
useradd -m devops
```

Set a password:

```bash
passwd devops
```

Delete a user:

```bash
userdel devops
```

Delete a user along with the home directory:

```bash
userdel -r devops
```

---

# Group Management Commands

Create a group:

```bash
groupadd developers
```

Delete a group:

```bash
groupdel developers
```

Add a user to a group:

```bash
usermod -aG docker devops
```

View group membership:

```bash
groups devops
```

---

# Linux File Ownership

Every file has:

- Owner
- Group

Example:

```text
surendra developers app.py
```

Owner and group determine who can access the file.

---

# View File Ownership

Display ownership information:

```bash
ls -l
```

Example output:

```text
-rw-r--r-- 1 surendra developers 1200 app.py
```

---

# Change File Owner

Change owner:

```bash
chown devops file.txt
```

Change owner and group:

```bash
chown devops:developers file.txt
```

---

# Change Group Ownership

```bash
chgrp developers file.txt
```

Useful when sharing files among teams.

---

# Linux Permissions

Linux permissions are divided into three categories.

```text
Owner

Group

Others
```

Each category can have:

- Read (r)
- Write (w)
- Execute (x)

---

# Permission Representation

Example:

```text
-rwxr-xr--
```

Breakdown:

```text
Owner  → rwx

Group  → r-x

Others → r--
```

---

# Permission Values

| Permission | Value |
|------------|------:|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

These values are used in numeric permission notation.

---

# Numeric Permissions

Examples:

| Permission | Meaning |
|------------|---------|
| 777 | Full access to everyone |
| 755 | Owner full, others read & execute |
| 700 | Owner only |
| 644 | Owner read/write, others read only |
| 600 | Owner read/write only |

---

# Change Permissions

Grant execute permission:

```bash
chmod +x script.sh
```

Numeric format:

```bash
chmod 755 script.sh
```

Restrict access:

```bash
chmod 600 secret.txt
```

---

# Symbolic Mode

Examples:

```bash
chmod u+x script.sh

chmod g+w file.txt

chmod o-r file.txt
```

Where:

- u = User
- g = Group
- o = Others
- a = All

---

# Default Permissions

When files are created, Linux assigns default permissions.

Typical defaults:

| Resource | Default Permission |
|----------|--------------------|
| File | 644 |
| Directory | 755 |

These defaults are influenced by **umask**.

---

# umask

The **umask** defines which permissions are removed from newly created files and directories.

View current umask:

```bash
umask
```

Example:

```text
022
```

A common enterprise setting is:

```text
022
```

---

# Special Permissions

Linux provides three special permissions.

| Permission | Purpose |
|------------|---------|
| SUID | Run as file owner |
| SGID | Inherit group ownership |
| Sticky Bit | Restrict file deletion in shared directories |

---

# SUID

When SUID is set,

a program runs with the permissions of the file owner.

Example:

```text
passwd
```

The `passwd` command needs temporary root privileges to update password files.

---

# SGID

When applied to directories,

new files inherit the directory's group.

Useful for shared project folders.

---

# Sticky Bit

Commonly used on:

```text
/tmp
```

Only the file owner can delete their own files inside the directory.

---

# Access Control Lists (ACL)

Standard Linux permissions provide only:

- Owner
- Group
- Others

ACLs allow assigning permissions to specific users or groups.

Example:

```text
Project Directory

↓

Owner

↓

developers

↓

tester
```

Different users can have different permissions.

---

# sudo

Instead of logging in as root,

users are granted administrative privileges through **sudo**.

Example:

```bash
sudo systemctl restart nginx
```

This provides accountability and improves security.

---

# Enterprise Example

A Jenkins server may use:

```text
jenkins

↓

docker

↓

developers
```

The Jenkins user is added to the **docker** group to build container images without using the root account.

---

# Kubernetes Example

On Kubernetes worker nodes:

```text
kubelet

↓

containerd

↓

root
```

System services run under dedicated service accounts with controlled permissions.

---

# Enterprise Best Practices

- Use regular users for daily work.
- Avoid direct root logins.
- Grant administrative access using `sudo`.
- Follow the principle of least privilege.
- Assign users to groups instead of managing permissions individually.
- Use ACLs when standard permissions are insufficient.
- Regularly review file ownership and permissions.
- Protect sensitive files with restrictive permissions.

---

# Common Mistakes

- Running daily tasks as the root user.
- Assigning `777` permissions to files or directories.
- Giving unnecessary sudo access.
- Ignoring group-based permission management.
- Forgetting to set execute permission on scripts.
- Misconfiguring shared directories.
- Leaving sensitive files world-readable.

---

# Interview Questions

## Basic

1. What is the difference between the root user and a regular user?
2. What is a UID and GID?
3. Explain Linux file permissions.
4. What is the difference between `chmod`, `chown`, and `chgrp`?
5. What is `sudo`?

## Intermediate

1. Explain numeric permissions such as `755`, `644`, and `700`.
2. What is `umask`?
3. Explain SUID, SGID, and Sticky Bit.
4. What are ACLs, and when would you use them?
5. Why is group-based access control preferred in enterprise environments?

## Advanced

1. Design a secure user and permission model for a production Jenkins server where developers can build Docker images without granting full root access.
2. Explain how Linux users, groups, permissions, ACLs, and `sudo` work together to implement the principle of least privilege in enterprise environments.
3. A financial organization hosts hundreds of applications on Linux servers. Design a secure access control strategy covering user management, service accounts, shared directories, file permissions, administrative access, and auditability.

---

# Chapter 4 - Linux Process Management, Services & systemd

Every application running on Linux is executed as a **process**.

Whether you are running:

- Nginx
- Docker
- Kubernetes
- Jenkins
- MySQL
- Redis

they all run as Linux processes.

Understanding process management is one of the most important skills for every DevOps Engineer because most production incidents involve process or service failures.

---

# What is a Process?

A process is a **running instance of a program**.

For example:

```text
Program

↓

Execution

↓

Process
```

Examples:

- Running Nginx
- Running Docker
- Running Jenkins
- Running PostgreSQL

Each running application creates one or more processes.

---

# Process Lifecycle

A process moves through several states during its lifetime.

```text
New

↓

Ready

↓

Running

↓

Waiting

↓

Completed
```

The Linux kernel manages every transition.

---

# Process States

Common process states include:

| State | Description |
|--------|-------------|
| Running (R) | Currently executing |
| Sleeping (S) | Waiting for an event |
| Disk Sleep (D) | Waiting for disk I/O |
| Stopped (T) | Suspended by user or debugger |
| Zombie (Z) | Process finished but not cleaned up |

Understanding these states helps during production troubleshooting.

---

# Process Identifier (PID)

Every process has a unique **Process ID (PID)**.

Example:

```text
PID 1

↓

systemd
```

Every new process receives a unique PID from the kernel.

---

# Parent and Child Processes

Linux processes are created in a hierarchy.

```text
systemd (PID 1)

↓

Jenkins

↓

Java

↓

Shell Script

↓

Docker Build
```

Every process except `PID 1` has a parent process.

---

# PID 1

The first process started by the Linux kernel is:

```text
systemd
```

It is responsible for:

- Starting services
- Managing child processes
- System initialization
- Service recovery

If PID 1 fails, the system becomes unstable.

---

# Viewing Running Processes

Display running processes:

```bash
ps
```

Detailed process list:

```bash
ps -ef
```

Show processes for current user:

```bash
ps -u username
```

---

# top Command

Monitor system activity in real time.

```bash
top
```

Displays:

- CPU usage
- Memory usage
- Running processes
- Load average
- Process states

Widely used during production incidents.

---

# htop

An improved interactive version of `top`.

Features:

- Better interface
- Process search
- Process tree
- Interactive management

```bash
htop
```

---

# Process Tree

Display parent-child relationships.

```bash
pstree
```

Example:

```text
systemd

├── sshd

├── nginx

├── docker

└── jenkins
```

Useful for identifying dependent processes.

---

# Finding Processes

Search for a process by name:

```bash
pgrep nginx
```

Or:

```bash
ps -ef | grep nginx
```

Commonly used during troubleshooting.

---

# Foreground vs Background Processes

Foreground process:

```bash
python app.py
```

The terminal waits until execution completes.

Background process:

```bash
python app.py &
```

The terminal remains available.

---

# Job Control

View background jobs:

```bash
jobs
```

Bring a job to the foreground:

```bash
fg
```

Move a process to the background:

```bash
bg
```

Useful during administration tasks.

---

# Killing Processes

Terminate a process gracefully:

```bash
kill PID
```

Force termination:

```bash
kill -9 PID
```

Terminate by name:

```bash
pkill nginx
```

Use force termination only when necessary.

---

# Process Priority

Linux schedules processes using priorities.

Nice values range from:

```text
-20

↓

Highest Priority

↓

0

↓

19

↓

Lowest Priority
```

Lower values receive more CPU time.

---

# nice Command

Start a process with a specific priority.

Example:

```bash
nice -n 10 python app.py
```

Higher nice values reduce process priority.

---

# renice

Change the priority of a running process.

Example:

```bash
renice 5 -p 1234
```

Useful for balancing system resources.

---

# Services in Linux

A **service** is a long-running background process managed by the operating system.

Examples:

- nginx
- docker
- sshd
- kubelet
- containerd
- jenkins

Services usually start automatically during boot.

---

# systemd

Most modern Linux distributions use **systemd** to manage services.

Responsibilities include:

- Starting services
- Stopping services
- Restarting services
- Monitoring failures
- Managing dependencies

---

# systemctl

`systemctl` is the primary command used to manage services.

Check service status:

```bash
systemctl status nginx
```

Start a service:

```bash
systemctl start nginx
```

Stop a service:

```bash
systemctl stop nginx
```

Restart a service:

```bash
systemctl restart nginx
```

Reload configuration:

```bash
systemctl reload nginx
```

---

# Enable and Disable Services

Start automatically during boot:

```bash
systemctl enable nginx
```

Disable automatic startup:

```bash
systemctl disable nginx
```

---

# Check Boot Status

View failed services:

```bash
systemctl --failed
```

List all services:

```bash
systemctl list-units --type=service
```

---

# Service Unit Files

systemd stores service definitions as unit files.

Common location:

```text
/etc/systemd/system/
```

Example:

```text
jenkins.service

docker.service

nginx.service
```

---

# Example Service File

A typical service file contains:

- Service description
- Startup command
- Restart policy
- Dependencies
- User account

These files define how services behave.

---

# Journald

systemd includes its own logging system called **journald**.

View logs:

```bash
journalctl
```

Logs for a specific service:

```bash
journalctl -u nginx
```

Follow logs:

```bash
journalctl -fu nginx
```

Useful during troubleshooting.

---

# Enterprise Example

A Jenkins deployment might follow this sequence:

```text
systemd

↓

jenkins.service

↓

Java Process

↓

Build Agent

↓

Shell Script

↓

Docker Build
```

Understanding the process tree helps identify failures quickly.

---

# Kubernetes Example

A Kubernetes worker node runs several critical services.

```text
systemd

├── kubelet

├── containerd

├── sshd

└── chronyd
```

If `kubelet` stops, workloads cannot be scheduled.

---

# Enterprise Best Practices

- Monitor long-running services regularly.
- Use `systemctl` instead of legacy service commands.
- Avoid using `kill -9` unless absolutely necessary.
- Review service logs using `journalctl`.
- Configure services to restart automatically after failures.
- Monitor CPU and memory usage during incidents.
- Understand parent-child process relationships.
- Verify service status after every deployment.

---

# Common Mistakes

- Killing critical system processes.
- Using `kill -9` without investigation.
- Forgetting to enable services after installation.
- Ignoring failed systemd services.
- Restarting services without checking logs.
- Running production workloads in the foreground.
- Ignoring zombie processes.

---

# Interview Questions

## Basic

1. What is a Linux process?
2. What is a PID?
3. Explain process states.
4. What is `systemd`?
5. What is the difference between a process and a service?

## Intermediate

1. Explain parent and child processes.
2. Difference between `kill` and `kill -9`.
3. What is the purpose of `nice` and `renice`?
4. How does `systemctl` manage services?
5. How do you troubleshoot a failed service using `journalctl`?

## Advanced

1. Design a process monitoring strategy for a production Linux server running Jenkins, Docker, and Kubernetes components.
2. A production application becomes unresponsive after deployment. Explain how you would investigate processes, service status, logs, CPU usage, memory usage, and process hierarchy to identify the root cause.
3. A Kubernetes worker node suddenly stops scheduling pods. Describe your step-by-step troubleshooting approach using `systemctl`, `journalctl`, process inspection, and service dependency analysis.

---

# Chapter 5 - Linux Memory Management, CPU Scheduling & Performance Monitoring

Performance is one of the most critical aspects of Linux system administration.

Whether you are running:

- Kubernetes Worker Nodes
- Docker Hosts
- Jenkins Servers
- Database Servers
- Application Servers

understanding how Linux manages CPU and memory is essential for maintaining a stable production environment.

Most production incidents are caused by:

- High CPU Utilization
- Memory Exhaustion
- Swap Usage
- Load Spikes
- Resource Contention
- Memory Leaks

---

# Linux Resource Management

Linux efficiently manages system resources through the kernel.

The kernel is responsible for:

- CPU Scheduling
- Memory Allocation
- Process Scheduling
- Disk I/O
- Network Resources

```text
Applications
      │
      ▼
Linux Kernel
      │
      ▼
CPU | Memory | Disk | Network
```

---

# Physical Memory (RAM)

RAM is the primary working memory used by running applications.

Applications load into RAM for execution.

Examples:

- Java Applications
- Nginx
- Docker Containers
- Kubernetes Pods
- Databases

More available RAM generally improves system performance.

---

# Virtual Memory

Linux uses **Virtual Memory** to allow processes to operate independently of the actual physical memory available.

Benefits include:

- Process Isolation
- Efficient Memory Allocation
- Better Stability
- Address Space Protection

Each process believes it has its own dedicated memory space.

---

# Swap Memory

Swap is disk space used as an extension of RAM.

```text
RAM Full

↓

Swap Space

↓

Disk
```

Swap helps prevent immediate application crashes but is much slower than RAM.

Excessive swap usage usually indicates memory pressure.

---

# Check Memory Usage

Display memory statistics:

```bash
free -h
```

Example output:

```text
              total   used   free   shared   buff/cache   available
Mem:           16G     8G     3G       1G        5G          7G
Swap:           4G     0G     4G
```

The `-h` option displays values in a human-readable format.

---

# Understanding Memory Output

Important fields:

| Field | Description |
|--------|-------------|
| Total | Total installed memory |
| Used | Memory currently in use |
| Free | Completely unused memory |
| Buff/Cache | Memory used for caching |
| Available | Memory available for new applications |

Do not panic if **free memory is low**. Linux aggressively uses free memory for caching.

---

# Memory Cache

Linux uses unused RAM as a filesystem cache.

Benefits:

- Faster file access
- Improved application performance
- Reduced disk I/O

Cached memory is automatically released when applications require additional memory.

---

# CPU Scheduling

The Linux kernel decides which process gets CPU time.

```text
Ready Queue

↓

CPU Scheduler

↓

CPU Core
```

The scheduler ensures fair CPU allocation among processes.

---

# CPU Cores

Modern servers contain multiple CPU cores.

Example:

```text
CPU

↓

Core 1

Core 2

Core 3

Core 4
```

Linux distributes workloads across available cores.

---

# Check CPU Information

View CPU details:

```bash
lscpu
```

Useful information includes:

- CPU Architecture
- Number of CPUs
- Number of Cores
- Threads per Core
- CPU Model

---

# View System Uptime

Display system uptime:

```bash
uptime
```

Example:

```text
10:20:45 up 25 days, 4 users, load average: 0.82, 0.90, 1.01
```

---

# Understanding Load Average

Linux reports three load averages.

```text
1 Minute

5 Minutes

15 Minutes
```

Interpretation depends on CPU count.

Example:

Server with 4 CPU cores

| Load Average | Status |
|--------------|---------|
| 2 | Healthy |
| 4 | Fully Utilized |
| 8 | Overloaded |

Always compare load average with the number of CPU cores.

---

# CPU Monitoring

Real-time CPU monitoring:

```bash
top
```

or

```bash
htop
```

Useful metrics include:

- CPU Utilization
- Running Processes
- Memory Usage
- Load Average

---

# vmstat

Display virtual memory statistics:

```bash
vmstat 2
```

Updates every two seconds.

Useful metrics:

- CPU Usage
- Memory
- Swap
- Disk I/O
- Context Switches

---

# iostat

Monitor disk I/O performance:

```bash
iostat -x 2
```

Useful for identifying storage bottlenecks.

Metrics include:

- Read/Write Speed
- Disk Utilization
- Queue Length
- I/O Wait

---

# sar

The `sar` command provides historical performance statistics.

Examples:

```bash
sar -u
```

CPU history.

```bash
sar -r
```

Memory history.

```bash
sar -n DEV
```

Network statistics.

Very useful during production investigations.

---

# Load vs CPU Usage

Load Average and CPU Usage are different.

| CPU Usage | Load Average |
|------------|--------------|
| CPU currently busy | Processes waiting for CPU or I/O |
| Percentage | Queue length |

A server can have:

- Low CPU usage
- High Load Average

if processes are waiting for disk or network operations.

---

# Out Of Memory (OOM)

When Linux runs out of available memory,

the **OOM Killer** terminates one or more processes.

```text
Memory Exhausted

↓

OOM Killer

↓

Terminate Process

↓

System Stabilized
```

OOM events are common causes of application outages.

---

# OOM Killer

The OOM Killer selects a process based on several factors, including memory usage and priority.

Typical victims:

- Java Applications
- Large Databases
- Memory-intensive Services

Investigate OOM events immediately.

---

# Check OOM Events

View kernel messages:

```bash
dmesg | grep -i oom
```

or

```bash
journalctl -k
```

These commands help identify processes terminated by the OOM Killer.

---

# Memory Leak

A memory leak occurs when an application continuously allocates memory without releasing it.

Symptoms:

- Increasing memory usage
- Growing swap usage
- OOM events
- Slow application performance

Memory leaks require application-level investigation.

---

# Enterprise Example

A Jenkins server may experience increasing memory usage during multiple concurrent builds.

```text
Jenkins

↓

Java Process

↓

Memory Growth

↓

High RAM Usage

↓

OOM Risk
```

Continuous monitoring helps detect the issue before service interruption.

---

# Kubernetes Example

A Kubernetes worker node runs multiple pods.

```text
Linux Worker Node

↓

Container Runtime

↓

Pods

↓

Memory Consumption
```

If total memory exceeds node capacity, Kubernetes may evict pods or Linux may invoke the OOM Killer.

---

# Enterprise Best Practices

- Monitor CPU and memory continuously.
- Compare load average with CPU core count.
- Investigate increasing swap usage.
- Review OOM events immediately.
- Monitor application memory growth.
- Use `top`, `htop`, `vmstat`, `iostat`, and `sar` regularly.
- Allocate appropriate resources for applications.
- Establish performance baselines for production systems.

---

# Common Mistakes

- Assuming low free memory indicates a problem.
- Ignoring swap usage.
- Confusing CPU usage with load average.
- Ignoring OOM Killer events.
- Restarting applications without identifying memory leaks.
- Monitoring only CPU while ignoring disk I/O.
- Not establishing baseline performance metrics.

---

# Interview Questions

## Basic

1. What is virtual memory?
2. What is swap memory?
3. What is the difference between RAM and swap?
4. What does the `free -h` command show?
5. What is load average?

## Intermediate

1. Explain Linux CPU scheduling.
2. What is the OOM Killer?
3. Difference between CPU utilization and load average.
4. Explain memory caching in Linux.
5. How do you monitor CPU and memory usage in production?

## Advanced

1. A production Linux server suddenly becomes slow even though CPU utilization is only 30%. Explain how you would investigate load average, disk I/O, memory usage, swap, and process behavior to identify the root cause.
2. A Kubernetes worker node begins terminating pods unexpectedly. Explain how you would investigate memory pressure, OOM Killer events, container resource consumption, and node performance.
3. Design a performance monitoring strategy for enterprise Linux servers hosting Jenkins, Docker, Kubernetes, and database workloads, including CPU, memory, disk, and historical performance metrics.

---

# Chapter 6 - Linux Disk Management, File Systems, LVM & Storage Administration

Storage is one of the most critical components of every Linux server.

Whether you are managing:

- Kubernetes Worker Nodes
- Docker Hosts
- Jenkins Servers
- Database Servers
- Application Servers

understanding Linux storage is essential for ensuring high availability, performance, and reliability.

Many production incidents occur because of:

- Disk Full
- Incorrect Mount Points
- File System Corruption
- LVM Issues
- Storage Failures
- Inode Exhaustion

---

# Linux Storage Architecture

A typical Linux storage stack looks like this:

```text
Application

↓

File System

↓

Logical Volume (LVM)

↓

Volume Group

↓

Physical Volume

↓

Disk

↓

Storage Controller
```

Each layer provides flexibility and simplifies storage management.

---

# Block Devices

Linux represents storage devices as **block devices**.

Common examples:

```text
/dev/sda

/dev/sdb

/dev/nvme0n1
```

These devices store data in fixed-size blocks.

---

# View Storage Devices

Display block devices:

```bash
lsblk
```

Example output:

```text
NAME        SIZE TYPE MOUNTPOINT

sda         100G disk

├── sda1      1G part /boot

├── sda2     50G part /

└── sda3     49G part
```

This command provides a clear overview of disks and partitions.

---

# View Disk Information

Display partition details:

```bash
fdisk -l
```

Useful for identifying:

- Disk Size
- Partition Table
- Partition Types

---

# Partitions

A disk can be divided into multiple partitions.

Example:

```text
Disk

↓

Partition 1

Partition 2

Partition 3
```

Each partition can contain its own file system.

---

# File Systems

A file system organizes data stored on a partition.

Common Linux file systems:

| File System | Description |
|-------------|-------------|
| ext4 | Most common Linux file system |
| XFS | Enterprise-grade, default on RHEL |
| Btrfs | Advanced snapshots and features |
| FAT32 | Cross-platform compatibility |
| NTFS | Windows compatibility |

---

# Check Mounted File Systems

Display mounted file systems:

```bash
df -h
```

Example:

```text
Filesystem      Size Used Avail Use%

/dev/sda2       50G  18G   30G   38%
```

The `-h` option shows sizes in a human-readable format.

---

# Disk Usage

Check directory size:

```bash
du -sh /var/log
```

Display all directories:

```bash
du -h --max-depth=1
```

Useful for identifying large directories.

---

# Finding Large Files

Locate files larger than 1 GB:

```bash
find / -type f -size +1G
```

This command is frequently used during disk space investigations.

---

# Mount Points

A mount point is a directory where a file system is attached.

Example:

```text
Disk

↓

Partition

↓

Mounted at

↓

/data
```

Applications access storage through mount points.

---

# View Mounted File Systems

```bash
mount
```

Or

```bash
findmnt
```

These commands display all active mounts.

---

# Mount a File System

Example:

```bash
mount /dev/sdb1 /data
```

The storage becomes accessible through:

```text
/data
```

---

# Unmount a File System

Safely remove a mounted file system:

```bash
umount /data
```

Always ensure no application is using the mount before unmounting.

---

# Persistent Mounts

Linux stores permanent mount configurations in:

```text
/etc/fstab
```

Example:

```text
UUID=xxxxx

↓

/

UUID=yyyyy

↓

/data
```

The system mounts these file systems automatically during boot.

---

# UUID

Every file system has a unique identifier called a UUID.

View UUIDs:

```bash
blkid
```

Using UUIDs is preferred over device names because device names can change after reboot.

---

# Logical Volume Manager (LVM)

LVM provides flexible storage management.

Benefits include:

- Online Expansion
- Snapshot Support
- Flexible Disk Allocation
- Easier Storage Management

Enterprise Linux servers commonly use LVM.

---

# LVM Architecture

```text
Disk

↓

Physical Volume (PV)

↓

Volume Group (VG)

↓

Logical Volume (LV)

↓

File System

↓

Mount Point
```

This abstraction simplifies storage expansion.

---

# Physical Volume (PV)

A Physical Volume is the underlying storage device used by LVM.

Example:

```text
/dev/sdb
```

Multiple physical volumes can be combined.

---

# Volume Group (VG)

A Volume Group combines one or more physical volumes into a storage pool.

```text
PV1

↓

PV2

↓

Volume Group
```

Storage is allocated from the volume group.

---

# Logical Volume (LV)

A Logical Volume is created from the Volume Group.

Applications interact with Logical Volumes rather than physical disks.

Example:

```text
Volume Group

↓

Logical Volume

↓

File System
```

---

# LVM Benefits

Compared to traditional partitions, LVM provides:

- Flexible resizing
- Better storage utilization
- Simplified expansion
- Snapshot capability

It is widely used in enterprise environments.

---

# Swap Partition

Swap can be configured as:

- Dedicated Partition
- Logical Volume
- Swap File

View swap usage:

```bash
swapon --show
```

---

# File System Check

Check a file system:

```bash
fsck /dev/sdb1
```

This command detects and repairs file system errors.

Run `fsck` only on unmounted file systems unless using specialized recovery procedures.

---

# Inodes

Every file consumes an inode.

A disk may have:

- Available storage
- No available inodes

Check inode usage:

```bash
df -i
```

Running out of inodes prevents new files from being created.

---

# Enterprise Example

A Jenkins server stores:

```text
/var/lib/jenkins

↓

Workspaces

↓

Artifacts

↓

Logs
```

If disk usage reaches 100%, builds fail and Jenkins becomes unstable.

---

# Kubernetes Example

On a Kubernetes worker node:

```text
/var/lib/kubelet

↓

Pods

↓

Volumes

↓

Persistent Storage
```

Storage issues may prevent pods from starting or mounting volumes.

---

# Storage Monitoring

Regularly monitor:

- Disk Utilization
- Free Space
- Inode Usage
- Mount Status
- I/O Performance

Common commands:

```bash
df -h

du -sh

lsblk

iostat

findmnt
```

---

# Enterprise Best Practices

- Monitor disk usage continuously.
- Use UUIDs in `/etc/fstab`.
- Prefer LVM for enterprise servers.
- Monitor inode utilization.
- Separate application data from the root filesystem.
- Regularly clean unnecessary log files.
- Verify mounts after reboot.
- Back up critical data before modifying partitions.

---

# Common Mistakes

- Filling the root filesystem.
- Editing `/etc/fstab` incorrectly.
- Mounting the wrong partition.
- Ignoring inode exhaustion.
- Running `fsck` on mounted production filesystems.
- Forgetting to back up before resizing storage.
- Using device names instead of UUIDs.

---

# Interview Questions

## Basic

1. What is a Linux file system?
2. What is a partition?
3. What is a mount point?
4. What is LVM?
5. What is the purpose of `/etc/fstab`?

## Intermediate

1. Explain the difference between a Physical Volume, Volume Group, and Logical Volume.
2. Why is UUID preferred over device names?
3. What is inode exhaustion?
4. Difference between `df` and `du`.
5. How do you investigate disk space issues?

## Advanced

1. Design a storage architecture for a production Kubernetes cluster using LVM, multiple file systems, and persistent storage while ensuring scalability and high availability.
2. A Jenkins server suddenly stops building because the disk is full. Explain your step-by-step approach to identify large files, inode exhaustion, log growth, mount issues, and recovery procedures.
3. Explain how Linux storage management, file systems, LVM, mount points, UUIDs, and monitoring contribute to reliable enterprise infrastructure.

---

