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

# Patch Manager

AWS Systems Manager Patch Manager automates operating system patching.

Supported operating systems

- Amazon Linux
- Ubuntu
- Red Hat Enterprise Linux
- CentOS
- SUSE Linux
- Windows Server

Benefits

- Automated Patching
- Compliance Reporting
- Maintenance Scheduling

---

# Patch Baselines

Patch Baselines define

- Approved Patches
- Rejected Patches
- Patch Classification
- Auto Approval Rules

Example

```text
Critical

↓

Approve Immediately

------------

Security

↓

Approve After 7 Days
```

---

# Patch Groups

Patch Groups organize servers.

Example

```text
Production

↓

Production Baseline

------------

Development

↓

Development Baseline
```

Different environments can use different patch schedules.

---

# Parameter Store

Parameter Store securely stores configuration values.

Examples

- Database URLs
- API Endpoints
- Feature Flags
- Application Configuration
- Environment Variables

Supports

- Plain Text
- Secure Strings

---

# SecureString Parameters

Sensitive parameters are encrypted using AWS KMS.

Workflow

```text
Application

↓

Parameter Store

↓

AWS KMS

↓

Encrypted Value
```

---

# Parameter Hierarchy

Example

```text
/production/database/host

/production/database/user

/production/database/password

/development/api/url
```

Hierarchical naming simplifies management.

---

# Maintenance Windows

Maintenance Windows schedule administrative tasks.

Examples

- Patch Servers
- Restart Services
- Execute Scripts
- Backup Databases

Workflow

```text
Maintenance Window

↓

Run Command

↓

Automation

↓

Completion
```

---

# Distributor

Distributor deploys software packages to managed nodes.

Examples

- Monitoring Agents
- Security Agents
- Internal Applications
- Utilities

Supports version-controlled package deployment.

---

# OpsCenter

OpsCenter centralizes operational issues.

Example

```text
CloudWatch Alarm

↓

OpsItem

↓

Engineer

↓

Resolution
```

Benefits

- Incident Tracking
- Operational Visibility
- Central Dashboard

---

# OpsItems

OpsItems contain

- Issue Description
- Severity
- Resource
- Investigation Notes
- Resolution Status

Useful for operational workflows.

---

# Explorer

Explorer provides a centralized operational dashboard.

Displays

- Patch Compliance
- Inventory
- OpsItems
- Automation Status
- Resource Health

Supports multi-account visibility.

---

# Change Manager

Change Manager standardizes infrastructure changes.

Supports

- Approval Workflows
- Change Templates
- Notifications
- Auditing

Useful for controlled production changes.

---

# Change Templates

Templates define

- Required Approvals
- Execution Steps
- Rollback Plans
- Notifications

Improves governance.

---

# Incident Manager

Incident Manager helps respond to operational incidents.

Features

- Incident Response Plans
- Contacts
- Escalation
- Runbooks
- Timeline

Supports enterprise incident management.

---

# Response Plans

Response Plans define

- Contacts
- Runbooks
- Escalation Paths
- Communication Channels

Automates incident response.

---

# Fleet Manager

Fleet Manager provides graphical server management.

Capabilities

- File Browser
- Windows Registry
- Process Viewer
- Performance Metrics
- Event Logs

No RDP or SSH required.

---

# Hybrid Activations

Systems Manager supports hybrid environments.

Supported

- On-Premises Servers
- VMware
- Edge Devices
- Other Cloud Providers

Architecture

```text
Hybrid Server

↓

SSM Agent

↓

AWS Systems Manager
```

---

# AWS Organizations Integration

Systems Manager supports centralized management.

Benefits

- Organization-wide Automation
- Shared Patch Policies
- Centralized Compliance
- Multi-Account Operations

---

# CloudWatch Integration

Monitor

- Automation Executions
- Patch Compliance
- Run Command Status
- Session Activity
- SSM Agent Health

---

# EventBridge Integration

Operational events trigger automation.

Example

```text
Patch Failed

↓

EventBridge

↓

Lambda

↓

SNS

↓

Operations Team
```

---

# AWS CLI

List Managed Instances

```bash
aws ssm describe-instance-information
```

Send Run Command

```bash
aws ssm send-command \
--document-name AWS-RunShellScript
```

Start Session

```bash
aws ssm start-session \
--target i-1234567890abcdef0
```

Get Parameter

```bash
aws ssm get-parameter \
--name "/production/database/password" \
--with-decryption
```

---

# Terraform

```hcl
resource "aws_ssm_parameter" "db_password" {

  name  = "/production/database/password"

  type  = "SecureString"

  value = "ChangeMe123"

}
```

Maintenance Window

```hcl
resource "aws_ssm_maintenance_window" "weekly" {

  name     = "weekly-maintenance"

  schedule = "cron(0 2 ? * SUN *)"

}
```

---

# CloudFormation

```yaml
Resources:

  DatabasePassword:

    Type: AWS::SSM::Parameter

    Properties:

      Name: /production/database/password

      Type: SecureString

      Value: ChangeMe123
```

---

# Python (Boto3)

```python
import boto3

ssm = boto3.client("ssm")

response = ssm.get_parameter(

    Name="/production/database/password",

    WithDecryption=True

)

print(response["Parameter"]["Value"])
```

---

# Enterprise Production Architecture

```text
          Administrators

                │

        AWS Systems Manager

                │

 ┌──────────────┼──────────────┐

 │              │              │

Run Command  Session Manager Patch Manager

 │              │              │

Automation  Parameter Store Inventory

 │              │              │

 EC2   Hybrid Servers   EKS Nodes

                │

 CloudWatch • EventBridge • OpsCenter
```

---

# Best Practices

- Install and update SSM Agent regularly
- Use Session Manager instead of SSH
- Enable CloudTrail logging
- Encrypt SecureString parameters with KMS
- Configure Patch Baselines
- Use Maintenance Windows
- Automate repetitive tasks
- Monitor SSM Agent health
- Enable centralized management with Organizations
- Apply least-privilege IAM permissions
- Log Session Manager sessions
- Review compliance reports regularly

---

# Common Mistakes

- Opening SSH port 22 unnecessarily
- Outdated SSM Agent
- Hardcoding application configuration
- Ignoring patch compliance
- No maintenance windows
- Missing IAM permissions
- Storing passwords in plain text
- No automation
- No session logging
- Ignoring OpsItems

---

# Troubleshooting

## Managed Instance Not Appearing

Check

- SSM Agent Running
- IAM Role
- Network Connectivity
- VPC Endpoints

---

## Session Manager Connection Failed

Verify

- IAM Permissions
- SSM Agent
- Systems Manager Endpoints
- Instance Status

---

## Run Command Failed

Check

- Document Name
- IAM Role
- Instance Status
- Command Logs

---

## Patch Manager Failed

Verify

- Patch Baseline
- Maintenance Window
- SSM Agent
- Repository Access

---

## Parameter Access Denied

Check

- IAM Policy
- KMS Key Policy
- Parameter Name
- SecureString Permissions

---

# Interview Questions

## Basic

1. What is AWS Systems Manager?
2. What is the SSM Agent?
3. What is a Managed Node?
4. What is Session Manager?
5. What is Run Command?
6. What is Patch Manager?
7. What is Parameter Store?
8. What are Maintenance Windows?
9. What is State Manager?
10. What is Inventory?

---

## Intermediate

11. Explain Automation Runbooks.
12. Explain Patch Baselines.
13. Explain Patch Groups.
14. Explain SecureString parameters.
15. Explain OpsCenter.
16. Explain Explorer.
17. Explain Fleet Manager.
18. Explain Hybrid Activations.
19. Explain EventBridge integration.
20. Explain CloudWatch integration.

---

## Advanced

21. Design enterprise server management architecture.
22. How would you replace SSH using Systems Manager?
23. Design automated patch management.
24. Explain Systems Manager vs Ansible.
25. Explain centralized multi-account operations.
26. Design secure configuration management.
27. Explain Parameter Store vs Secrets Manager.
28. Design enterprise automation workflows.
29. Explain incident management with Systems Manager.
30. Best practices for production AWS Systems Manager deployments.

---

# Production Scenarios

### Scenario 1

Your organization wants to disable SSH access to all production EC2 instances.

How would Session Manager replace SSH?

---

### Scenario 2

A critical Linux security patch must be deployed to 2,000 EC2 instances.

How would Patch Manager automate the process?

---

### Scenario 3

Developers need application configuration values without hardcoding them.

How would Parameter Store provide a secure solution?

---

### Scenario 4

An enterprise manages AWS and on-premises servers.

How would Hybrid Activations simplify administration?

---

### Scenario 5

Operations teams require approval before running production automation.

Which Systems Manager feature would implement this process?

---

### Scenario 6

Auditors request evidence showing all administrator sessions.

Which Systems Manager capabilities provide this information?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| SSM Agent | Execute Management Tasks |
| Session Manager | Secure Remote Access |
| Run Command | Remote Command Execution |
| Automation | Multi-Step Workflows |
| Patch Manager | OS Patching |
| State Manager | Configuration Management |
| Inventory | Resource Information |
| Parameter Store | Configuration Storage |
| Maintenance Window | Scheduled Operations |
| OpsCenter | Operational Issues |
| Fleet Manager | Server Management |
| Incident Manager | Incident Response |

---

# Summary

AWS Systems Manager is a comprehensive operations management service that enables organizations to securely manage infrastructure at scale. Features such as Session Manager, Run Command, Patch Manager, State Manager, Parameter Store, Maintenance Windows, Automation, Inventory, OpsCenter, Fleet Manager, and Incident Manager simplify server administration, eliminate the need for direct SSH access, automate operational tasks, and improve compliance. When integrated with IAM, CloudWatch, EventBridge, KMS, AWS Organizations, and hybrid infrastructure, Systems Manager becomes a foundational service for enterprise cloud operations and DevOps automation.