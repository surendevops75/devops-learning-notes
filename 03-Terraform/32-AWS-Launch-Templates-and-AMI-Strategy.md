# AWS Launch Templates and AMI Strategy

## Introduction

In production environments, creating and configuring servers manually is not practical.

Wrong Approach:

```text
Create EC2

Login To EC2

Install Software

Configure Application

Repeat For Every New Server
```

Problems:

```text
Time Consuming

Manual Errors

Inconsistent Configuration

Difficult Scaling
```

---

# Better Approach

```text
Create One Server

Configure Completely

Create AMI

Use AMI To Launch New Servers
```

This ensures:

```text
Consistency

Automation

Fast Scaling
```

---

# What is an AMI?

AMI stands for:

```text
Amazon Machine Image
```

An AMI is a template used to launch EC2 instances.

---

# AMI Contains

```text
Operating System

Installed Packages

Application Code

Configurations

Custom Settings
```

---

# Production Example

Create:

```text
Catalogue Server
```

Install:

```text
Java

Catalogue Application

Dependencies
```

After configuration:

```text
Create AMI
```

Now every new server launched from this AMI will be identical.

---

# Traditional Deployment

```text
Create EC2
      ↓
Install Packages
      ↓
Configure Application
      ↓
Application Ready
```

Every new server repeats the same process.

---

# AMI Based Deployment

```text
Create EC2
      ↓
Configure Using Ansible
      ↓
Stop Instance
      ↓
Create AMI
      ↓
Launch New Servers From AMI
```

Configuration happens only once.

---

# Production Workflow

Step 1

Create EC2 Instance.

```text
Catalogue EC2
```

---

Step 2

Configure using Ansible.

```text
Install Java

Deploy Application

Configure Service
```

---

Step 3

Validate Application.

```text
Application Working
```

---

Step 4

Stop Server.

```text
EC2 Stopped
```

---

Step 5

Create AMI.

```text
catalogue-v1
```

---

Step 6

Launch new servers from AMI.

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

All servers become identical.

---

# What is a Launch Template?

A Launch Template is a blueprint for launching EC2 instances.

It defines:

```text
How New Instances Should Be Created
```

---

# Launch Template Contains

```text
AMI

Instance Type

Security Group

Subnet

User Data

IAM Role
```

---

# Real World Analogy

Suppose HR wants to recruit employees.

HR requires:

```text
Job Description

Experience

Skills

Location
```

This information is used repeatedly.

Similarly:

```text
Launch Template
```

stores server creation details.

---

# Production Example

Launch Template:

```text
AMI = catalogue-v1

Instance Type = t3.micro

Security Group = catalogue-sg

IAM Role = catalogue-role
```

Whenever AWS needs a new server:

```text
Use Launch Template
```

---

# Why Use Launch Templates?

Without Launch Template:

```text
Manually Select

AMI

Instance Type

Security Group

IAM Role
```

every time.

---

With Launch Template:

```text
Reusable Configuration
```

---

# Launch Template Workflow

```text
Launch Template
       ↓
Create EC2
       ↓
Ready Server
```

---

# Production Scenario

Catalogue Application receives high traffic.

Auto Scaling needs:

```text
3 New Servers
```

AWS creates:

```text
Catalogue-11

Catalogue-12

Catalogue-13
```

using Launch Template.

No manual work required.

---

# Launch Template Components

## AMI

Defines:

```text
Operating System

Application

Installed Packages
```

---

## Instance Type

Defines:

```text
CPU

Memory

Performance
```

Example:

```text
t3.micro

t3.small

t3.medium
```

---

## Security Group

Defines:

```text
Network Access Rules
```

Example:

```text
catalogue-sg
```

---

## IAM Role

Defines:

```text
AWS Permissions
```

Example:

```text
catalogue-role
```

---

## User Data

Runs commands during boot.

Example:

```bash
#!/bin/bash

echo "Server Started"
```

---

# AMI Versioning Strategy

Version Example:

```text
catalogue-v1

catalogue-v2

catalogue-v3
```

Benefits:

```text
Easy Rollback

Controlled Releases

Version Tracking
```

---

# Production Example

Current Production:

```text
catalogue-v1
```

New Release:

```text
catalogue-v2
```

If issue occurs:

```text
Rollback To catalogue-v1
```

---

# AMI vs Launch Template

## AMI

Contains:

```text
Operating System

Application

Configuration
```

---

## Launch Template

Contains:

```text
AMI

Instance Type

Security Group

IAM Role

User Data
```

---

# Relationship

```text
Launch Template
       ↓
Uses
       ↓
AMI
```

---

# Production Architecture

```text
Create Catalogue Server
            ↓
Configure Using Ansible
            ↓
Create AMI
            ↓
Create Launch Template
            ↓
Auto Scaling Uses Launch Template
            ↓
New Servers Created
```

---

# Common Interview Questions

## What is an AMI?

### Short Answer

An AMI is a template used to launch EC2 instances.

### Detailed Explanation

AMI contains the operating system, installed software, application code, and configuration required to create identical servers.

### Production Example

A configured Catalogue server is converted into catalogue-v1 AMI and used for launching multiple instances.

### Follow-Up Questions

- What does an AMI contain?
- Why create custom AMIs?

---

## What is a Launch Template?

### Short Answer

A Launch Template is a reusable blueprint for launching EC2 instances.

### Detailed Explanation

It stores instance configuration such as AMI, instance type, security groups, IAM roles, and user data.

### Production Example

Auto Scaling Group uses a Catalogue Launch Template to create new servers automatically.

### Follow-Up Questions

- What settings are stored in a Launch Template?
- Can Launch Templates use custom AMIs?

---

## Why Create an AMI After Configuration?

### Short Answer

To avoid configuring every new server repeatedly.

### Detailed Explanation

Once a server is fully configured, an AMI captures its state and allows identical servers to be launched quickly.

### Production Example

Catalogue application installed once and reused through AMI creation.

### Follow-Up Questions

- What are the advantages of AMI-based deployments?
- Why is this better than manual configuration?

---

## Difference Between AMI and Launch Template?

### Short Answer

AMI stores the server image, Launch Template stores instance launch configuration.

### Detailed Explanation

AMI contains software and configuration, while Launch Template defines how AWS should create new instances using that AMI.

### Production Example

Catalogue-v2 AMI is referenced inside a Catalogue Launch Template.

### Follow-Up Questions

- Can multiple Launch Templates use the same AMI?
- Can a Launch Template work without an AMI?

---

# Key Takeaways

```text
AMI stands for Amazon Machine Image.

AMI contains OS, application, and configuration.

Launch Templates define how EC2 instances are created.

Launch Templates contain AMI, instance type, security groups, and IAM roles.

Configured servers can be converted into reusable AMIs.

AMI-based deployments improve consistency and scalability.

Launch Templates are heavily used by Auto Scaling Groups.

Versioned AMIs simplify upgrades and rollbacks.

Auto Scaling uses Launch Templates to launch identical servers automatically.
```