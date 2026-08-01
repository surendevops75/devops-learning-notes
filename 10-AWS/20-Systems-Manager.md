# AWS Systems Manager (AWS SSM)

---

# Introduction

AWS Systems Manager (SSM) is a fully managed management service that enables organizations to securely manage AWS infrastructure, on-premises servers, virtual machines, and hybrid cloud environments from a single centralized platform.

Instead of logging into every EC2 instance individually using SSH or RDP, administrators can manage servers remotely, automate operational tasks, patch operating systems, execute commands, maintain inventory, and securely store configuration parameters.

AWS Systems Manager is one of the most important services for DevOps, Cloud Engineers, and Platform Engineers because it greatly reduces operational complexity while improving security and compliance.

AWS Systems Manager integrates with:

- Amazon EC2
- Amazon EKS Worker Nodes
- Amazon ECS EC2 Instances
- AWS IAM
- AWS CloudWatch
- AWS KMS
- Amazon S3
- AWS Secrets Manager
- AWS Organizations

---

# What is AWS Systems Manager?

AWS Systems Manager is a centralized operations management service.

It helps manage:

- EC2 Instances
- Hybrid Servers
- On-Premises Servers
- Virtual Machines
- Edge Devices

It provides a secure way to perform administrative operations without directly accessing servers.

---

# Why AWS Systems Manager?

Traditional Server Management

```text
Administrator

↓

SSH

↓

Server

↓

Manual Commands
```

Problems

- SSH Key Management
- Bastion Hosts
- Security Risks
- Manual Operations
- Difficult Automation

Using Systems Manager

```text
Administrator

↓

AWS Console / CLI

↓

Systems Manager

↓

Managed Instance
```

No SSH required.

---

# Real World Problem

A company manages

- 800 EC2 Instances
- 120 Kubernetes Worker Nodes
- 40 On-Prem Servers

Requirements

- Patch Management
- Centralized Administration
- Secure Remote Access
- Inventory Collection
- Automation
- Compliance

AWS Systems Manager solves these challenges.

---

# Enterprise Architecture

```text
Administrator

↓

AWS Systems Manager

↓

IAM Authentication

↓

SSM Agent

↓

Managed Instances

↓

CloudWatch

↓

Amazon S3
```

---

# Core Components

AWS Systems Manager includes

- Managed Instances
- SSM Agent
- Session Manager
- Run Command
- Automation
- Patch Manager
- State Manager
- Inventory
- Parameter Store
- Maintenance Windows
- Distributor
- Fleet Manager
- Documents

---

# Managed Instance

A Managed Instance is any server registered with Systems Manager.

Examples

- EC2
- VMware VM
- On-Prem Linux
- Windows Server

---

# SSM Agent

The SSM Agent runs on managed instances.

Responsibilities

- Receive Commands
- Execute Tasks
- Send Logs
- Report Status

Architecture

```text
Systems Manager

↓

SSM Agent

↓

Linux Server
```

Amazon Linux and modern Windows AMIs include the agent by default.

---

# IAM Role

EC2 instances require an IAM Role.

Example

```text
EC2

↓

IAM Role

↓

AmazonSSMManagedInstanceCore
```

Without this role,

the instance cannot communicate with Systems Manager.

---

# Session Manager

Session Manager provides secure shell access without SSH.

Traditional

```text
Laptop

↓

SSH

↓

EC2
```

Session Manager

```text
Laptop

↓

AWS Console

↓

Systems Manager

↓

EC2
```

Benefits

- No SSH Keys
- No Bastion Host
- No Open Port 22
- Audit Logging

---

# Session Manager Workflow

```text
Administrator

↓

IAM Authentication

↓

Systems Manager

↓

SSM Agent

↓

EC2 Shell
```

Connections are encrypted.

---

# Run Command

Run Command executes commands across multiple servers simultaneously.

Example

```text
100 Servers

↓

Run Command

↓

systemctl restart nginx
```

No SSH required.

---

# Common Run Command Tasks

Examples

- Restart Services
- Install Packages
- Collect Logs
- Update Applications
- Change Configuration
- Execute Scripts

---

# Systems Manager Documents

Documents define automation instructions.

Types

- Command Documents
- Automation Documents
- Session Documents
- Policy Documents

Example

```text
Document

↓

Install Docker

↓

Execute
```

---

# Automation

Automation executes predefined workflows.

Example

```text
Automation

↓

Create Snapshot

↓

Stop Instance

↓

Patch Server

↓

Start Instance
```

Useful for repetitive operational tasks.

---

# Patch Manager

Patch Manager automates operating system updates.

Supports

- Linux
- Windows

Workflow

```text
Patch Baseline

↓

Maintenance Window

↓

Install Updates

↓

Compliance Report
```

---

# Patch Baselines

Patch Baselines define

- Approved Patches
- Rejected Patches
- Auto Approval Rules
- Compliance Levels

---

# State Manager

State Manager maintains desired server configuration.

Example

Desired State

```text
CloudWatch Agent Installed
```

If removed,

State Manager reinstalls it automatically.

---

# Inventory

Inventory collects server information.

Examples

- Installed Packages
- Applications
- OS Version
- Running Services
- Network Configuration

Useful for audits.

---

# Parameter Store

Parameter Store securely stores configuration values.

Examples

- Database Host
- API URL
- Environment Variables
- Feature Flags

Supports

- Plain Text
- Secure Strings

Secure Strings are encrypted using AWS KMS.

---

# Parameter Store vs Secrets Manager

| Parameter Store | Secrets Manager |
|----------------|----------------|
| Configuration | Sensitive Secrets |
| Optional Encryption | Always Encrypted |
| No Rotation | Automatic Rotation |
| Lower Cost | Premium Features |

---

# Maintenance Windows

Maintenance Windows schedule operational tasks.

Examples

- Sunday Patching
- Database Restart
- Agent Updates
- Security Scans

Workflow

```text
Schedule

↓

Run Command

↓

Automation

↓

Report
```

---

# Distributor

Distributor installs software packages.

Examples

- CloudWatch Agent
- Security Agent
- Internal Software

---

# Fleet Manager

Fleet Manager provides graphical management.

Supports

- File System
- Performance
- Processes
- Services
- Event Logs

Useful for Windows and Linux administration.

---

# Hybrid Activations

Systems Manager also manages

- Physical Servers
- VMware
- Azure VMs
- GCP VMs

Single management platform.

---

# Security Benefits

Systems Manager eliminates

- SSH Keys
- Bastion Hosts
- Public SSH Access
- Open Port 22
- Password Sharing

This significantly improves security.

---

# Summary

AWS Systems Manager centralizes infrastructure management by providing secure remote access, automation, patch management, inventory collection, configuration management, and parameter storage. Using Session Manager, Run Command, Automation, Patch Manager, State Manager, and Parameter Store, organizations can manage thousands of servers without relying on SSH or RDP while improving security, compliance, and operational efficiency.

---