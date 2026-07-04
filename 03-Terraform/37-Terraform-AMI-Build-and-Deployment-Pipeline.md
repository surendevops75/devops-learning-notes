# Terraform AMI Build and Deployment Pipeline

## Introduction

In production environments, applications should be deployed in a repeatable and consistent manner.

Instead of:

```text
Create Server

Login Manually

Install Software

Configure Application

Repeat Again
```

we automate the entire process.

---

# Deployment Goal

We want:

```text
Identical Servers

Automated Deployments

Easy Scaling

Easy Rollbacks
```

---

# Complete Deployment Pipeline

```text
Terraform
    ↓
Create Instance
    ↓
remote-exec
    ↓
Shell Script
    ↓
Ansible
    ↓
Configure Application
    ↓
Stop Instance
    ↓
Create AMI
    ↓
Launch Template
    ↓
Auto Scaling Group
    ↓
Target Group
    ↓
ALB Rule
    ↓
Production Traffic
```

---

# Step 1 - Create EC2 Instance

Terraform creates a temporary server.

Purpose:

```text
Build Server

Golden Image Server

AMI Build Server
```

---

# Why Create a Temporary Instance?

We need a server where:

```text
Application

Dependencies

Configuration
```

can be installed.

---

# Step 2 - remote-exec Provisioner

Terraform uses:

```text
remote-exec
```

to connect to the newly created server.

---

# What is remote-exec?

remote-exec allows Terraform to:

```text
Connect To Server

Run Commands

Execute Scripts
```

inside the instance.

---

# Example Flow

```text
Terraform
      ↓
SSH Connection
      ↓
EC2 Instance
      ↓
Execute Commands
```

---

# Step 3 - Copy Deployment Script

Terraform copies a shell script.

Purpose:

```text
Automation

Configuration

Application Setup
```

---

# Example

```text
deploy.sh
```

may contain:

```text
Clone Repository

Install Packages

Run Ansible
```

---

# Step 4 - Clone Ansible Roles

Deployment script clones:

```text
Ansible Roles Repository
```

---

# Why Clone Roles?

Roles contain:

```text
Application Configuration

Database Configuration

Monitoring Setup

Reusable Automation
```

---

# Production Example

Clone:

```text
roboshop-ansible
```

repository.

---

# Step 5 - Pass Arguments

Terraform passes arguments.

Example:

```text
dev

catalogue
```

---

# Flow

```text
Terraform
      ↓
Shell Script
      ↓
Ansible
```

---

# Example

Terraform:

```text
Component = catalogue

Environment = dev
```

Shell:

```bash
$1 = catalogue

$2 = dev
```

Ansible receives:

```text
catalogue

dev
```

---

# Step 6 - Configure Instance Using Ansible

Ansible performs:

```text
Package Installation

Application Deployment

Service Configuration

Health Validation
```

---

# Production Example

Catalogue Server:

```text
Install Java

Deploy Catalogue Service

Configure Systemd

Start Application
```

---

# Step 7 - Validate Application

Verify:

```text
Application Running

Health Check Passing
```

---

# Example

```bash
curl http://localhost:8080/health
```

Expected:

```text
Healthy Response
```

---

# Step 8 - Stop Instance

After configuration:

```text
Stop EC2 Instance
```

---

# Why Stop Instance?

AWS recommends:

```text
Create Stable AMI
```

from a stopped server.

---

# Step 9 - Create AMI

AMI stands for:

```text
Amazon Machine Image
```

---

# AMI Contains

```text
Operating System

Application

Packages

Configuration
```

---

# Example

```text
catalogue-v1
```

---

# Why Create AMI?

Benefits:

```text
Consistency

Repeatability

Fast Deployment

Easy Rollback
```

---

# Step 10 - Create Target Group

Target Group contains:

```text
Application Servers
```

Example:

```text
Catalogue Target Group
```

---

# Target Group Responsibilities

```text
Health Checks

Traffic Distribution

Backend Registration
```

---

# Step 11 - Create Launch Template

Launch Template acts as:

```text
Server Blueprint
```

---

# Launch Template Contains

```text
AMI

Instance Type

Subnet

Security Group

Storage

IAM Role
```

---

# Example

```text
AMI = catalogue-v1

Instance Type = t3.micro

Security Group = catalogue-sg
```

---

# Real World Analogy

Creating a Job Description before recruitment.

Example:

```text
Skills

Experience

Location

Responsibilities
```

Similarly:

```text
Launch Template
```

defines server requirements.

---

# Step 12 - Create Auto Scaling Group

ASG automatically creates servers.

---

# Responsibilities

```text
Launch Instances

Terminate Instances

Replace Failed Instances

Maintain Capacity
```

---

# Example

Desired:

```text
3 Catalogue Servers
```

If one fails:

```text
ASG Creates Replacement
```

---

# Step 13 - Create Auto Scaling Policy

Policy determines:

```text
When To Add Servers

When To Remove Servers
```

---

# Scale Out

Condition:

```text
Average CPU > 75%
```

Action:

```text
Launch New Instance
```

---

# Scale In

Condition:

```text
Average CPU < 30%
```

Action:

```text
Terminate Instance
```

---

# Step 14 - Create Load Balancer Rule

Traffic reaches:

```text
Application Load Balancer
```

---

# Rule Example

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue Target Group
```

---

# Complete Request Flow

```text
User
   ↓
ALB
   ↓
Listener
   ↓
Rule
   ↓
Target Group
   ↓
Catalogue Instances
```

---

# Production Architecture

```text
Terraform
      ↓
Create Build Server
      ↓
remote-exec
      ↓
Shell Script
      ↓
Ansible
      ↓
Configure Catalogue
      ↓
Create AMI
      ↓
Launch Template
      ↓
Auto Scaling Group
      ↓
Target Group
      ↓
ALB Listener Rule
      ↓
Production Traffic
```

---

# Why This Architecture?

Benefits:

```text
Immutable Infrastructure

Repeatable Deployments

Easy Scaling

Easy Rollback

High Availability
```

---

# Common Interview Questions

## What is remote-exec Provisioner?

### Short Answer

remote-exec allows Terraform to execute commands inside a remote server.

### Detailed Explanation

Terraform establishes an SSH connection to the target instance and executes commands or scripts after resource creation.

### Production Example

Terraform connects to a Catalogue build server and executes deployment scripts.

### Follow-Up Questions

- How does Terraform connect to the server?
- What are alternatives to remote-exec?

---

## Why Create an AMI After Configuration?

### Short Answer

To create reusable and identical server images.

### Detailed Explanation

Once the application is configured successfully, the server is converted into an AMI. Future servers can be launched directly from this image.

### Production Example

Catalogue-v1 AMI created after configuring Catalogue service.

### Follow-Up Questions

- Why are AMI-based deployments preferred?
- What is immutable infrastructure?

---

## What is the Role of a Launch Template?

### Short Answer

A Launch Template defines how new EC2 instances should be created.

### Detailed Explanation

It stores AMI, instance type, security groups, subnet configuration, and other settings required to launch servers.

### Production Example

Catalogue Launch Template referencing catalogue-v1 AMI.

### Follow-Up Questions

- Can Launch Templates be versioned?
- Can ASGs share Launch Templates?

---

## Explain the Complete Deployment Pipeline.

### Short Answer

Create → Configure → AMI → Launch Template → ASG → Target Group → ALB Rule.

### Detailed Explanation

A temporary server is created and configured using Ansible. An AMI is generated and used in a Launch Template. Auto Scaling launches instances from the Launch Template, and traffic is routed through ALB and Target Groups.

### Production Example

Catalogue microservice deployment in AWS.

### Follow-Up Questions

- Where does Ansible fit into the deployment process?
- Why is this architecture scalable?

---

# Key Takeaways

```text
Terraform creates the build server.

remote-exec executes commands on remote instances.

Shell scripts act as a bridge between Terraform and Ansible.

Ansible configures applications and dependencies.

Configured instances are converted into AMIs.

AMIs provide immutable and repeatable deployments.

Launch Templates define instance creation settings.

Auto Scaling Groups manage server capacity automatically.

Target Groups distribute traffic to healthy instances.

ALB Listener Rules route requests to the correct application.

This deployment model is widely used in production environments.
```