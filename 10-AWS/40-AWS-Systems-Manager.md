# AWS Systems Manager (SSM)

---

# Introduction

AWS Systems Manager (SSM) is a fully managed operations management service that helps organizations securely manage, automate, patch, configure, monitor, and operate AWS resources at scale.

In enterprise environments, administrators often manage hundreds or thousands of EC2 instances, hybrid servers, virtual machines, and cloud resources. Performing software installation, patching, configuration updates, inventory collection, and remote access manually becomes complex and error-prone.

AWS Systems Manager centralizes these operational tasks through a unified management platform.

AWS Systems Manager integrates with:

- Amazon EC2
- Amazon EKS
- Amazon ECS
- AWS Lambda
- AWS IAM
- AWS CloudTrail
- Amazon CloudWatch
- Amazon EventBridge
- AWS Secrets Manager
- AWS KMS
- Amazon S3
- AWS Organizations
- Hybrid Servers

Systems Manager is one of the most important services for enterprise operations and automation.

---

# What is AWS Systems Manager?

AWS Systems Manager is a centralized operations management service.

It allows administrators to

- Manage Servers
- Execute Commands
- Patch Systems
- Collect Inventory
- Automate Tasks
- Store Parameters
- Securely Access Instances

Workflow

```text
Administrator

↓

AWS Systems Manager

↓

Managed Nodes

↓

Operations Executed
```

---

# Why AWS Systems Manager?

Without Systems Manager

```text
Administrator

↓

SSH

↓

Manual Commands

↓

Individual Servers

↓

Configuration Drift
```

Problems

- Manual Administration
- SSH Dependency
- Inconsistent Configurations
- Slow Patching
- Difficult Auditing

With Systems Manager

```text
Administrator

↓

Systems Manager

↓

Automation

↓

Managed Infrastructure
```

---

# Real World Problem Statement

A company manages

- 1,200 EC2 Instances
- 200 On-Premises Servers
- Multiple AWS Accounts
- Kubernetes Worker Nodes

Requirements

- Remote Administration
- Patch Management
- Inventory Collection
- Configuration Management
- Secure Access
- Automation

AWS Systems Manager provides centralized management.

---

# Enterprise Architecture

```text
Administrator

↓

AWS Systems Manager

↓

SSM Agent

↓

EC2

Hybrid Servers

EKS Nodes

↓

Automation
```

---

# Core Components

AWS Systems Manager consists of

- Managed Nodes
- SSM Agent
- Session Manager
- Run Command
- Automation
- Patch Manager
- State Manager
- Inventory
- Parameter Store
- Maintenance Windows

---

# Managed Node

A Managed Node is any resource managed by Systems Manager.

Examples

- EC2 Instance
- On-Premises Server
- Virtual Machine
- Edge Device

---

# SSM Agent

SSM Agent runs on managed nodes.

Responsibilities

- Receive Commands
- Execute Tasks
- Report Status
- Upload Logs

Workflow

```text
Systems Manager

↓

SSM Agent

↓

Server

↓

Execution
```

---

# IAM Role

Managed instances require an IAM role.

Common Policy

```
AmazonSSMManagedInstanceCore
```

Allows Systems Manager communication.

---

# Session Manager

Session Manager provides secure shell access.

Benefits

- No SSH Keys
- No Bastion Host
- No Open Port 22
- Fully Audited

Workflow

```text
Administrator

↓

Session Manager

↓

Managed Instance
```

---

# Session Logging

Sessions can be logged to

- Amazon CloudWatch Logs
- Amazon S3

Useful for compliance.

---

# Run Command

Run Command executes commands remotely.

Example

```text
Administrator

↓

Run Command

↓

500 EC2 Instances

↓

Execute Script
```

No SSH required.

---

# Run Command Use Cases

Common tasks

- Install Software
- Restart Services
- Collect Logs
- Update Configuration
- Execute Scripts

---

# Documents (SSM Documents)

Systems Manager uses Documents to define operations.

Document Types

- Command
- Automation
- Session
- Policy

AWS provides many prebuilt documents.

---

# Command Document

Example

```text
AWS-RunShellScript
```

Runs shell commands on Linux.

Windows equivalent

```text
AWS-RunPowerShellScript
```

---

# Automation

Automation executes multi-step operational workflows.

Example

```text
Stop EC2

↓

Create Snapshot

↓

Patch

↓

Restart EC2
```

Supports approvals and rollbacks.

---

# Automation Runbooks

Runbooks define

- Steps
- Conditions
- Parameters
- Outputs

Useful for repeatable operations.

---

# State Manager

State Manager maintains desired configurations.

Example

```text
Install CloudWatch Agent

↓

Every Server

↓

Always Installed
```

Prevents configuration drift.

---

# Associations

State Manager uses Associations.

Association

↓

Document

↓

Target Instances

↓

Continuous Compliance

---

# Inventory

Inventory collects information about managed nodes.

Examples

- Installed Software
- Operating System
- Network Configuration
- Running Services
- Applications

Useful for audits and compliance.

---

# Inventory Data

Collected information includes

- OS Version
- Installed Packages
- Windows Updates
- Applications
- AWS Components

Automatically updated.

---

# Compliance

Systems Manager tracks compliance.

Examples

- Patch Compliance
- Configuration Compliance
- Inventory Compliance

Results appear in the Systems Manager dashboard.

---

# Summary

AWS Systems Manager is a centralized operations management service that simplifies server administration, automation, remote access, inventory collection, and configuration management across AWS and hybrid environments. Components such as SSM Agent, Session Manager, Run Command, Automation, State Manager, Inventory, and Compliance enable secure, scalable, and automated infrastructure management.

---

