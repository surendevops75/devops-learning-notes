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

# AWS CLI

List Managed Instances

```bash
aws ssm describe-instance-information
```

Start Session

```bash
aws ssm start-session \
--target i-0123456789abcdef0
```

Run Command

```bash
aws ssm send-command \
--document-name "AWS-RunShellScript" \
--targets "Key=InstanceIds,Values=i-0123456789abcdef0" \
--parameters commands="sudo systemctl restart nginx"
```

Check Command Status

```bash
aws ssm list-command-invocations
```

Get Parameter

```bash
aws ssm get-parameter \
--name database-host
```

Get Secure Parameter

```bash
aws ssm get-parameter \
--name database-password \
--with-decryption
```

Put Parameter

```bash
aws ssm put-parameter \
--name application-url \
--value https://api.company.com \
--type String
```

Delete Parameter

```bash
aws ssm delete-parameter \
--name application-url
```

---

# Terraform

Create Parameter

```hcl
resource "aws_ssm_parameter" "database_host" {

  name  = "/production/database/host"

  type  = "String"

  value = "database.company.internal"

}
```

Secure Parameter

```hcl
resource "aws_ssm_parameter" "database_password" {

  name  = "/production/database/password"

  type  = "SecureString"

  value = "Password@123"

}
```

Maintenance Window

```hcl
resource "aws_ssm_maintenance_window" "weekly_patch" {

  name = "weekly-patching"

  schedule = "cron(0 2 ? * SUN *)"

  duration = 4

}
```

---

# CloudFormation

```yaml
Resources:

  DatabaseHost:

    Type: AWS::SSM::Parameter

    Properties:

      Name: /production/database/host

      Type: String

      Value: database.company.internal
```

---

# Python (Boto3)

Retrieve Parameter

```python
import boto3

ssm = boto3.client("ssm")

response = ssm.get_parameter(
    Name="/production/database/host"
)

print(response)
```

Retrieve Secure Parameter

```python
response = ssm.get_parameter(
    Name="/production/database/password",
    WithDecryption=True
)

print(response)
```

Execute Run Command

```python
response = ssm.send_command(
    InstanceIds=["i-0123456789abcdef0"],
    DocumentName="AWS-RunShellScript",
    Parameters={
        "commands": [
            "sudo systemctl restart nginx"
        ]
    }
)
```

---

# Enterprise Production Architecture

```text
                    DevOps Engineer

                           │

                    IAM Authentication

                           │

                AWS Systems Manager

     ┌──────────────┬──────────────┬──────────────┐

     │              │              │

Session Manager  Run Command  Automation

     │              │              │

     └──────────────┼──────────────┘

                    │

               SSM Agent

      ┌─────────────┼─────────────┐

      │             │             │

    EC2        EKS Worker     On-Prem

      │             │             │

      └─────────────┼─────────────┘

           CloudWatch Logs

                  │

             Amazon S3

                  │

         Compliance Reports
```

---

# Production Use Cases

## Centralized Linux Administration

```text
Administrator

↓

Systems Manager

↓

500 Linux Servers

↓

Execute Commands
```

No SSH access required.

---

## Patch Management

```text
Maintenance Window

↓

Patch Manager

↓

Linux Updates

↓

Compliance Report
```

---

## CloudWatch Agent Installation

```text
Automation

↓

Run Command

↓

Install Agent

↓

CloudWatch Metrics
```

---

## Emergency Service Restart

```text
Run Command

↓

systemctl restart nginx

↓

500 Servers
```

Completed simultaneously.

---

## Configuration Distribution

```text
Parameter Store

↓

EC2

↓

Read Configuration

↓

Application
```

Applications always retrieve the latest configuration.

---

# Security Best Practices

### Use Session Manager Instead of SSH

Avoid exposing:

- Port 22
- SSH Keys
- Bastion Hosts

---

### Encrypt Secure Parameters

Always use

```
SecureString
```

with

```
AWS KMS
```

for passwords.

---

### Least Privilege IAM

Allow only required actions.

Example

```text
Developer

↓

Read Parameters

----------------

Operations

↓

Run Command

----------------

Security

↓

Patch Manager
```

---

### Enable CloudTrail

Audit every operation.

Track

- Session Start
- Run Command
- Parameter Updates
- Automation Executions

---

### Enable Session Logging

Store sessions in

- CloudWatch Logs
- Amazon S3

Useful for compliance.

---

### Use Maintenance Windows

Avoid patching production during business hours.

Schedule

- Sunday Morning
- Midnight
- Planned Downtime

---

### Use Parameter Hierarchy

Example

```text
/production/database/host

/production/database/password

/staging/database/host

/development/database/host
```

Improves organization.

---

### Enable Inventory

Track

- Installed Packages
- Running Services
- OS Version
- Security Software

---

### Use Automation Documents

Instead of manual operations.

Examples

- Restart Services
- Install Packages
- Rotate Logs
- Create Snapshots

---

# Common Mistakes

- Using SSH instead of Session Manager
- Not installing SSM Agent
- Missing IAM Role
- Storing passwords in String parameters
- No CloudWatch logging
- Manual server patching
- Using AdministratorAccess for SSM
- Ignoring failed Run Commands
- Running outdated SSM Agent
- Not using Automation Documents

---

# Troubleshooting

## Instance Not Visible

Check

- SSM Agent installed
- Agent running
- IAM Role attached
- Internet/NAT/VPC Endpoint
- Correct Region

---

## Session Manager Fails

Verify

- IAM permissions
- Agent status
- SSM endpoint connectivity
- Session Manager Plugin
- CloudWatch logs

---

## Run Command Failed

Check

- Document exists
- Command syntax
- Target instance
- IAM permissions
- Agent logs

---

## Secure Parameter Access Denied

Verify

- IAM Policy
- KMS Key Policy
- Parameter ARN
- AWS Region

---

## Patch Manager Failed

Check

- Patch Baseline
- Maintenance Window
- Instance Registration
- IAM Role
- Agent Version

---

# Interview Questions

## Basic

1. What is AWS Systems Manager?
2. What is SSM Agent?
3. What is a Managed Instance?
4. What is Session Manager?
5. What is Run Command?
6. What is Patch Manager?
7. What is State Manager?
8. What is Parameter Store?
9. What is Inventory?
10. What are Maintenance Windows?

---

## Intermediate

11. Session Manager vs SSH?
12. Parameter Store vs Secrets Manager?
13. Explain Automation.
14. Explain Distributor.
15. Explain Fleet Manager.
16. Explain Hybrid Activations.
17. Explain Patch Baselines.
18. How does State Manager work?
19. How does Run Command work internally?
20. Explain SSM architecture.

---

## Advanced

21. How would you eliminate SSH from production?
22. Explain Session Manager internals.
23. Design centralized server management.
24. How would you patch 5,000 EC2 instances?
25. How would you manage hybrid infrastructure?
26. Explain SSM Agent communication.
27. How would you troubleshoot disconnected instances?
28. Explain Parameter Store hierarchy.
29. How would you secure Parameter Store?
30. Explain Systems Manager integration with CloudWatch and CloudTrail.

---

# Production Scenarios

### Scenario 1

Your organization decides to disable SSH across all EC2 instances.

How would Systems Manager replace SSH?

---

### Scenario 2

A new CloudWatch Agent must be installed on 2,000 servers.

Which Systems Manager features would you use?

---

### Scenario 3

A critical vulnerability requires immediate patching.

How would Patch Manager help?

---

### Scenario 4

A production EC2 instance is missing from Systems Manager.

Describe your troubleshooting steps.

---

### Scenario 5

Database connection strings change across all environments.

How would Parameter Store simplify configuration management?

---

### Scenario 6

Operations teams need proof of every administrative session.

Which AWS services would you enable?

---

### Scenario 7

You need to restart Nginx on 1,000 servers simultaneously.

Which Systems Manager capability would you use?

---

### Scenario 8

Your company manages AWS, VMware, and Azure virtual machines.

How can Systems Manager provide centralized administration?

---

# Cheat Sheet

| Feature | Purpose |
|----------|---------|
| Managed Instance | Server managed by SSM |
| SSM Agent | Communication agent |
| Session Manager | Secure shell without SSH |
| Run Command | Execute remote commands |
| Automation | Workflow execution |
| Patch Manager | OS patch management |
| State Manager | Maintain desired configuration |
| Inventory | Collect server metadata |
| Parameter Store | Configuration management |
| SecureString | Encrypted parameters |
| Maintenance Window | Scheduled operations |
| Fleet Manager | GUI server management |
| Distributor | Software deployment |
| Hybrid Activation | Manage on-premises servers |

---

# Summary

AWS Systems Manager is a comprehensive operations management service that centralizes administration, automation, configuration management, patching, inventory collection, and secure remote access for AWS and hybrid environments. By leveraging Session Manager, Run Command, Patch Manager, State Manager, Parameter Store, and Automation, organizations can eliminate SSH-based administration, improve security, automate repetitive operational tasks, and efficiently manage thousands of servers from a single platform. Systems Manager is a foundational service for enterprise DevOps, Platform Engineering, and Cloud Operations teams.