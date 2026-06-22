# Linux and Operating System Fundamentals

## Introduction

Linux is one of the most popular operating systems used in:

- Cloud Computing
- DevOps
- Containers
- Kubernetes
- Enterprise Applications
- Super Computers

Most servers running on AWS, Azure, and GCP use Linux-based operating systems.

---

# What is an Operating System?

An Operating System (OS) acts as a bridge between Hardware and Software.

Without an OS:

- Applications cannot interact with hardware
- Memory cannot be managed
- Processes cannot be executed
- Devices cannot communicate

## Operating System Architecture

```text
Applications
     │
     ▼
Operating System
     │
     ▼
Hardware
```

Examples:

- Linux
- Windows
- macOS
- Android
- Unix

---

# Linux: OS or Kernel?

This is a common interview question.

## Kernel

A Kernel is the core component of an operating system.

Responsibilities:

- Process Management
- Memory Management
- Device Management
- CPU Scheduling
- File System Management
- Network Management

Architecture:

```text
Applications
      │
      ▼
Kernel
      │
      ▼
Hardware
```

The kernel directly interacts with hardware.

---

## Operating System

Operating System consists of:

```text
Operating System = Kernel + Utilities + Applications
```

Examples of utilities:

```text
ls
cp
mv
cat
grep
awk
systemctl
```

Therefore:

```text
Linux = Kernel

Ubuntu = Operating System

RedHat Enterprise Linux = Operating System

CentOS = Operating System
```

---

# History of Linux

Before Linux, Unix was the dominant operating system for servers.

Architecture:

```text
Hardware + Unix OS
```

Unix was powerful but expensive.

---

# Linus Torvalds

Linux was created by:

```text
Linus Torvalds
```

He developed:

- Linux Kernel
- Git Version Control System

Programming Language Used:

```text
C Language
```

---

# GNU/Linux

Linux itself is only a kernel.

A complete operating system requires:

```text
Linux Kernel
+
GNU Utilities
=
GNU/Linux
```

Examples:

- Ubuntu
- Fedora
- CentOS
- AlmaLinux
- Rocky Linux
- RHEL

---

# Why Linux Dominates Servers

## Open Source

Source code is publicly available.

Anyone can:

- Study
- Modify
- Improve

---

## Stability

Linux servers can run for months or years without rebooting.

---

## Security

Linux provides:

- User Management
- Group Management
- File Permissions
- SSH Authentication
- Firewall Support

---

## Performance

Linux consumes fewer resources than many desktop operating systems.

---

## Flexibility

Linux runs on:

- Servers
- Desktops
- Mobile Devices
- Cloud Platforms
- Embedded Devices

---

# Linux Distributions

A Linux Distribution is:

```text
Linux Kernel
+
Utilities
+
Package Manager
+
Applications
```

---

# RedHat Family

The RedHat ecosystem consists of several Linux distributions that share a common foundation.

Major distributions include:

- Fedora
- Red Hat Enterprise Linux (RHEL)
- CentOS
- AlmaLinux
- Rocky Linux
- Amazon Linux

---

## Fedora

Community-driven Linux distribution.

Features are introduced here first.

---

## RHEL (Red Hat Enterprise Linux)

Enterprise Linux distribution.

Provides:

```text
Open Source Software
+
Enterprise Support
```

---

## CentOS

Community Linux distribution derived from RedHat technologies.

Historically used as a free alternative to RHEL.

---

## AlmaLinux

Free and community-supported RHEL-compatible distribution.

---

## Rocky Linux

Community enterprise Linux distribution designed to be compatible with RHEL.

---

## Amazon Linux

AWS optimized Linux distribution.

Default AWS User:

```text
ec2-user
```

---

# RedHat Ecosystem

```text
Fedora
   │
   ▼
RHEL
   │
   ├── CentOS
   ├── AlmaLinux
   ├── Rocky Linux
   └── Amazon Linux
```

---

# Open Source vs Enterprise

## Open Source

Characteristics:

- Free
- Community Driven
- Public Source Code

Examples:

- Fedora
- Ubuntu
- AlmaLinux

### Limitation

Bug fixes depend on community contributions.

---

## Enterprise

Characteristics:

- Paid Support
- Service Level Agreements (SLAs)
- Vendor Assistance

Example:

```text
Enterprise = Open Source + Support
```

Example:

```text
RHEL
```

---

# What is an Image?

An Image is a pre-configured template used to create servers or virtual machines.

An image typically contains:

- Operating System
- Required Software
- Configuration Files
- Dependencies

Formula:

```text
Image = Operating System + Required Software + Configuration
```

Examples:

- Amazon Linux AMI
- Ubuntu AMI
- RedHat AMI

---

# Amazon Machine Image (AMI)

AMI stands for:

```text
Amazon Machine Image
```

Used to launch EC2 Instances.

Contains:

- Operating System
- Installed Software
- Security Configuration
- Application Dependencies

---

## Example

```text
devops-practice
```

AMI may contain:

```text
Amazon Linux
+
NodeJS
+
MySQL Client
+
Application Dependencies
```

---

# EC2 Login Example

Default AWS User:

```text
ec2-user
```

Connect:

```bash
ssh ec2-user@34.228.199.227
```

---

# Server Concepts

## What is a Server?

A Server is a machine that provides services to other machines.

Examples:

- Web Server
- Application Server
- Database Server
- DNS Server

---

## Server Terminology

```text
Server
=
Node
=
Host
=
Machine
=
Instance (Cloud)
```

---

# Hardware and Operating Systems

## Apple Ecosystem

```text
Hardware + macOS
```

Apple controls both hardware and software.

---

## Dell / PC Ecosystem

```text
Hardware + Operating System
```

Examples:

```text
Dell + Windows

Dell + Ubuntu

Dell + RedHat

Dell + Fedora
```

Users have freedom to choose their operating system.

---

# User Space vs Kernel Space

## User Space

Applications execute here.

Examples:

- Browser
- VS Code
- Nginx
- MySQL
- Java Applications

---

## Kernel Space

Kernel operates here.

Responsibilities:

- Hardware Access
- Device Drivers
- CPU Scheduling
- Memory Management

---

# Linux Boot Process (High Level)

```text
Power On
   │
   ▼
BIOS / UEFI
   │
   ▼
GRUB Bootloader
   │
   ▼
Linux Kernel
   │
   ▼
systemd
   │
   ▼
User Space
```

---

# Process Example

When you run:

```bash
ls -l
```

Linux creates a process:

```text
Command
   │
   ▼
Process
   │
   ├── PID
   ├── Memory
   └── CPU Resources
```

The kernel manages these resources.

---

# Modern Application Architecture

A common production architecture follows:

```text
User
   │
   ▼
Load Balancer
   │
   ▼
Web Server
   │
   ▼
Application Server
   │
   ▼
Database
```

Each layer has a specific responsibility and can be scaled independently.

---

# Frequently Asked Interview Questions

## Is Linux an OS or Kernel?

Linux is technically a Kernel.

Ubuntu, RHEL, Fedora, AlmaLinux, Rocky Linux, and Amazon Linux are Operating Systems.

---

## Who Created Linux?

```text
Linus Torvalds
```

---

## Which Language is Linux Written In?

```text
C Language
```

---

## Difference Between Kernel and Operating System

| Kernel | Operating System |
|----------|----------|
| Core Component | Complete Environment |
| Manages Hardware | Provides User Interface & Utilities |
| Linux Kernel | Ubuntu, RHEL, Fedora |

---

## What is an AMI?

Amazon Machine Image used to launch EC2 instances.

---

## Difference Between Open Source and Enterprise Linux

| Open Source | Enterprise |
|------------|------------|
| Free | Paid Support |
| Community Driven | Vendor Support |
| No SLA | SLA Available |

---

## Why Linux is Preferred for Servers?

- Stable
- Secure
- Lightweight
- High Performance
- Cost Effective

---

# Key Takeaways

- Linux is a Kernel, not a complete Operating System.
- Operating System = Kernel + Utilities + Applications.
- Linux was created by Linus Torvalds.
- Git was also created by Linus Torvalds.
- Unix inspired modern Linux systems.
- Fedora, RHEL, CentOS, AlmaLinux, Rocky Linux, and Amazon Linux belong to the RedHat ecosystem.
- Open Source software is community-driven.
- Enterprise software provides vendor support.
- AMIs are used to launch EC2 instances.
- Linux dominates cloud, DevOps, and server environments.
- Understanding Linux fundamentals is essential for DevOps Engineers.