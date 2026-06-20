# Linux and AWS EC2 Fundamentals

## Account Creation Rules

### Prerequisites

1. Private bank debit cards with RuPay may not work.
2. Canara Bank and Union Bank cards are commonly supported.
3. SBI cards also work if international transactions are enabled.
4. Maintain a minimum account balance of ₹500.
5. Use the same address as registered with the bank card.
6. PAN Card is mandatory.

### Important Notes

- Do not create multiple accounts using the same card.
- Use a different email address and different card for another account.
- Wait for at least 6 months before creating another account.

### Credits and Refunds

- Initial Credit: $100
- Additional Credit: $100 after usage begins
- Total Value: Approximately ₹17,000

### Refund Policy

- Below ₹1000: Avoid requesting refunds.
- Above ₹1000: Refund requests can be raised.
- Maximum refund attempts: 3

---

# What is a Computer?

A computer is an **IP-enabled device** capable of processing data and communicating over a network.

## Communication Between Computers

The exchange of data between computers is called **Information Technology (IT)**.

### Examples of Computers

- Server
- Laptop
- Desktop
- Mobile Phone
- Smart TV
- Smart AC

Although they have different names, all are computers designed for different purposes.

---

## Purpose-Based Classification

### Server

Used for:

- Hosting applications
- Hosting websites
- Storing data
- Providing services to clients

### Personal Computer

Used for:

- Internet browsing
- Watching videos
- Listening to music
- Online banking
- General-purpose computing

### Mobile Phone

Used for:

- Calling
- Messaging
- Social media
- Internet access

---

## Basic Components of a Computer

### Hardware

- CPU (Processor)
- RAM (Memory)
- Storage (Disk)

### Software

- Operating System (OS)

---

# Operating System (OS)

An Operating System acts as a bridge between hardware and software.

```text
Application
     ↓
Operating System
     ↓
Hardware
```

---

# Linux

Linux is one of the most widely used operating systems for servers.

---

# Why Linux is Better than Windows for Servers?

## Windows

### Advantages

- User-friendly graphical interface

### Disadvantages

- Expensive licensing
- Heavy graphics consumption
- Higher resource usage
- Slower performance
- Frequent restarts
- Antivirus installation is mandatory
- OS upgrades can be complex

---

## Linux Server Edition

### Advantages

- Open Source
- Free to use
- No graphical interface (CLI-based)
- Extremely fast
- Minimal resource consumption
- Can run for years without rebooting
- Strong built-in security
- Easy upgrades
- Ideal for servers and cloud environments

### Disadvantage

- Learning curve for beginners

---

# Linux Learning Path

1. Create a Linux Server
2. Connect to the Server
3. Execute Basic Linux Commands
4. Learn Administration Tasks

---

# AWS Basics

## Region and Availability Zone

### Region

A Region is a geographical location containing AWS data centers.

### Example

```text
us-east-1
```

### North Virginia

- First AWS Region
- One of the cheapest AWS regions
- Commonly used for learning and testing

---

## Availability Zone (AZ)

Each Region contains multiple Availability Zones.

### Benefits

- High Availability
- Fault Tolerance
- Disaster Recovery

### Example

```text
us-east-1a
us-east-1b
us-east-1c
```

AWS recommends using at least two Availability Zones.

---

# EC2 (Elastic Compute Cloud)

EC2 is a virtual server service provided by AWS.

### Uses

- Hosting Applications
- Running Linux Servers
- Web Servers
- Databases

---

# AMI (Amazon Machine Image)

An AMI is a template used to launch an EC2 instance.

### Contains

- Operating System
- Pre-installed Software
- Configuration Settings

### Example

- Amazon Linux
- Ubuntu
- Red Hat Linux

---

# Free Tier Instances

### Common Types

#### t2.micro

- Eligible for AWS Free Tier

#### t3.micro

- Eligible in some accounts
- Better performance

### Example Specifications

```text
t3.micro
---------
2 vCPU
1 GB RAM
```

---

# Security in AWS

Security is controlled through:

- Authentication
- Security Groups (Firewall)

---

# Authentication

Authentication verifies the identity of a user.

---

## What You Know

Knowledge-Based Authentication

### Examples

- Username
- Password

---

## What You Have

Possession-Based Authentication

### Examples

- OTP
- RSA Token
- Google Authenticator
- Microsoft Authenticator

---

## What You Are

Biometric Authentication

### Examples

- Fingerprint
- Retina Scan
- Palm Scan
- Face Recognition

---

# SSH Keys

SSH uses Public Key Cryptography.

### Components

- Public Key
- Private Key

---

# SSH (Secure Shell)

SSH is used to securely connect to remote Linux servers.

### Default Port

```text
22
```

---

# Client-Server Architecture

### Server

A system that provides services.

### Client

A system that consumes services.

---

## Real-Life Example

### Lawyer

```text
Lawyer → Server
Public → Client
```

The lawyer provides services.

---

## IT Example

```text
facebook.com → Server
Browser → Client
```

The browser accesses services provided by Facebook.

---

# Linux Server Clients

Tools used to connect to Linux servers:

- PuTTY
- MobaXterm
- Windows CMD
- Mac Terminal
- Git Bash

---

# Git Bash

Git Bash provides:

- SSH Client
- Git Client
- Mini Linux Environment on Windows

### Default Location

```text
C:\Users\<username>
```

Example:

```text
C:\Users\siva
```

---

# Windows vs Linux Paths

## Windows

```text
\
```

Example:

```text
C:\Users\siva
```

---

## Linux

```text
/
```

Example:

```text
/home/ec2-user
```

---

# Basic Linux Commands

## pwd

Present Working Directory

```bash
pwd
```

Example Output:

```text
/c/devops/daws-86s
```

---

## cd

Change Directory

```bash
cd /c/devops/daws-86s
```

---

# Generating SSH Keys

```bash
ssh-keygen -f daws-86s
```

### Purpose

Creates:

- Private Key
- Public Key

Used for secure SSH authentication.

---

# AWS Security Groups

Security Groups act as a virtual firewall.

---

## Traffic Types

### Inbound Rules

Incoming traffic to the server.

### Outbound Rules

Outgoing traffic from the server.

---

# SSH Access

### Default Port

```text
22
```

### Example Public IP

```text
98.80.251.7
```

### Default User

```text
ec2-user
```

---

# Connect to Linux Server

```bash
ssh -i <private-key> ec2-user@98.80.251.7
```

### Example

```bash
ssh -i daws-86s ec2-user@98.80.251.7
```

Where:

- `ssh` → Secure Shell Command
- `-i` → Identity File (Private Key)
- `ec2-user` → Username
- `98.80.251.7` → Server Public IP

---

# List Files

## ls -l

Display files and directories in long listing format.

```bash
ls -l
```

### Information Displayed

- Permissions
- Owner
- Group
- File Size
- Modification Date
- File Name

---

# Key Interview Questions

## What is Linux?

Linux is an open-source operating system widely used for servers, cloud computing, and enterprise applications.

---

## What is EC2?

EC2 (Elastic Compute Cloud) is AWS's virtual machine service used to run applications and servers in the cloud.

---

## What is an AMI?

AMI (Amazon Machine Image) is a preconfigured template used to launch EC2 instances.

---

## What is SSH?

SSH (Secure Shell) is a secure protocol used to connect and manage remote Linux servers.

---

## What is a Security Group?

A Security Group is a virtual firewall that controls inbound and outbound traffic to AWS resources.

---

## What is the Difference Between Authentication and Authorization?

### Authentication

Verifies who you are.

Example:

```text
Username + Password
```

### Authorization

Determines what you can access after authentication.

Example:

```text
Admin → Full Access
User → Limited Access
```