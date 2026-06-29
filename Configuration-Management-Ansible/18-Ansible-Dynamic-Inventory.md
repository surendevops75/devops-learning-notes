# Ansible Dynamic Inventory

## Introduction

In the beginning of learning Ansible, we usually create a static inventory.

Example:

```ini
[frontend]
172.31.10.10

[mongodb]
172.31.10.20
```

This works fine for small environments.

However, in production environments, infrastructure changes frequently.

Examples:

```text
Autoscaling

Instance Replacement

Disaster Recovery

Blue-Green Deployments

Temporary Servers
```

In such environments:

```text
Static Inventory Becomes Difficult To Maintain
```

To solve this problem, Ansible provides:

```text
Dynamic Inventory
```

---

# What is Dynamic Inventory?

Dynamic Inventory allows Ansible to:

```text
Connect To Cloud Providers

Fetch Server Details Dynamically

Build Inventory Automatically
```

instead of manually maintaining inventory files.

---

# Static Inventory Problem

Suppose inventory.ini contains:

```ini
[frontend]

172.31.10.10
```

---

Today:

```text
Frontend Server

172.31.10.10
```

---

Tomorrow:

```text
Frontend Server Terminated

New Server Created

172.31.50.25
```

---

Problem:

```text
Inventory File Is Wrong
```

Playbook fails.

---

# Autoscaling Challenge

Modern applications often use:

```text
Auto Scaling Groups
```

---

Example:

```text
Traffic Low
      │
      ▼
2 Servers

Traffic High
      │
      ▼
10 Servers

Traffic Very High
      │
      ▼
20 Servers
```

---

IP addresses continuously change.

Maintaining static inventory becomes impossible.

---

# Dynamic Inventory Solution

Instead of:

```text
Store IP Addresses
```

we:

```text
Query AWS

Fetch Current Instances

Generate Inventory Automatically
```

---

# Architecture

```text
Ansible Controller
         │
         ▼
AWS API
         │
         ▼
Fetch EC2 Information
         │
         ▼
Generate Inventory
         │
         ▼
Execute Tasks
```

---

# How Dynamic Inventory Works

Workflow:

```text
Playbook Starts
       │
       ▼
Inventory Plugin Executes
       │
       ▼
Query AWS
       │
       ▼
Fetch Running Instances
       │
       ▼
Create Inventory
       │
       ▼
Execute Tasks
```

---

# Inventory Plugins

Ansible uses:

```text
Inventory Plugins
```

to communicate with external systems.

Examples:

```text
AWS

Azure

GCP

VMware

Kubernetes
```

---

# AWS Dynamic Inventory Plugin

Ansible provides:

```text
amazon.aws.aws_ec2
```

plugin.

This plugin:

```text
Connects To AWS

Queries EC2

Returns Hosts Dynamically
```

---
# Production Dynamic Inventory Example

In real-world environments, Dynamic Inventory is commonly used with AWS Auto Scaling Groups and EC2 instances.

Instead of manually maintaining inventory files, Ansible automatically discovers servers using AWS APIs and EC2 tags.

---

## Inventory Plugin File

File Name:

```text
inventory.aws_ec2.yml
```

---

## Complete Production Configuration

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

filters:
  instance-state-name: running
  tag:Project: roboshop
  tag:Environment: prod

hostnames:
  - private-ip-address

keyed_groups:
  - key: tags.Name
    prefix: ""

  - key: tags.Environment
    prefix: ""

compose:
  ansible_host: private_ip_address
```

---

# Architecture

```text
Ansible Controller
        │
        ▼
AWS Dynamic Inventory Plugin
        │
        ▼
AWS API
        │
        ▼
Fetch Running EC2 Instances
        │
        ▼
Generate Inventory
        │
        ▼
Execute Playbooks
```

---

# Understanding the Configuration

## Plugin

```yaml
plugin: amazon.aws.aws_ec2
```

Uses the AWS EC2 Inventory Plugin provided by the Amazon AWS collection.

---

## Regions

```yaml
regions:
  - us-east-1
```

Specifies which AWS region should be queried.

Multiple regions are supported.

Example:

```yaml
regions:
  - us-east-1
  - us-west-2
```

---

## Filters

```yaml
filters:
  instance-state-name: running
  tag:Project: roboshop
  tag:Environment: prod
```

Returns only:

- Running instances
- Roboshop resources
- Production environment resources

This prevents accidentally targeting:

- Development servers
- QA servers
- Other projects

---

## Hostnames

```yaml
hostnames:
  - private-ip-address
```

Inventory hosts will appear as:

```text
172.31.10.20
172.31.10.21
172.31.10.22
```

Alternative options:

```yaml
hostnames:
  - public-ip-address
```

or

```yaml
hostnames:
  - dns-name
```

---

## Dynamic Group Creation

```yaml
keyed_groups:
  - key: tags.Name
    prefix: ""
```

AWS Tags:

```text
Name=frontend
Name=catalogue
Name=user
```

Generated Groups:

```text
frontend
catalogue
user
```

---

Environment Groups:

```yaml
- key: tags.Environment
  prefix: ""
```

Generated Groups:

```text
prod
dev
qa
```

---

## Compose

```yaml
compose:
  ansible_host: private_ip_address
```

Defines which IP address Ansible uses for SSH connectivity.

---

# Recommended AWS Tagging Strategy

Production environments should use consistent tagging.

Example:

```text
Name=frontend

Environment=prod

Project=roboshop

Owner=devops
```

Benefits:

- Easier filtering
- Better organization
- Cost allocation
- Dynamic grouping

---

# Roboshop Production Example

Instances:

```text
frontend
catalogue
user
cart
shipping
payment
mongodb
redis
mysql
rabbitmq
```

Tags:

```text
Name=frontend
Environment=prod

Name=catalogue
Environment=prod

Name=user
Environment=prod
```

Dynamic Inventory automatically generates groups without manually maintaining inventory.ini.

---

# Generated Inventory Example

Equivalent Static Inventory:

```ini
[frontend]
172.31.10.20

[catalogue]
172.31.10.21

[user]
172.31.10.22

[cart]
172.31.10.23

[shipping]
172.31.10.24
```

No manual updates are required.

---

# Testing Dynamic Inventory

View inventory structure:

```bash
ansible-inventory -i inventory.aws_ec2.yml --graph
```

Example Output:

```text
@all
 ├── @frontend
 │    └── 172.31.10.20

 ├── @catalogue
 │    └── 172.31.10.21

 ├── @user
 │    └── 172.31.10.22
```

---

View complete inventory:

```bash
ansible-inventory -i inventory.aws_ec2.yml --list
```

---

# Running Playbooks

Execute a playbook using Dynamic Inventory:

```bash
ansible-playbook \
-i inventory.aws_ec2.yml \
frontend.yaml
```

Target a specific group:

```bash
ansible frontend \
-i inventory.aws_ec2.yml \
-m ping
```

---

# Production Benefits

## Supports Autoscaling

```text
2 Frontend Servers
        ↓
10 Frontend Servers
```

New instances automatically appear in inventory.

---

## Eliminates Manual IP Management

No need to update:

```ini
inventory.ini
```

every time servers change.

---

## Cloud Native

Works perfectly with:

- Auto Scaling Groups
- Spot Instances
- Blue-Green Deployments
- Disaster Recovery

---

# Enterprise Workflow

```text
Terraform
      │
      ▼
Creates Infrastructure
      │
      ▼
AWS Tags Resources
      │
      ▼
Dynamic Inventory
      │
      ▼
Ansible Configuration Management
      │
      ▼
Application Deployment
```

---

# Interview Question

## What Is the Most Common Production Use Case for Dynamic Inventory?

Dynamic Inventory is heavily used in cloud environments where infrastructure changes frequently.

Examples:

- Auto Scaling Groups
- Blue-Green Deployments
- Spot Instances
- Disaster Recovery

Instead of maintaining IP addresses manually, Ansible discovers hosts dynamically using cloud APIs and resource tags.
# What is a Plugin?

Understanding the difference is important.

---

# Tasks

Tasks:

```text
Execute Something
```

on remote servers.

Examples:

```yaml
Install Package

Create User

Restart Service
```

---

# Plugins

Plugins:

```text
Extend Ansible Functionality
```

Examples:

```text
Inventory Plugins

Connection Plugins

Lookup Plugins

Filter Plugins
```

---

# Inventory Plugin Role

Inventory plugin:

```text
Connect To AWS

Fetch Instance Details

Build Inventory
```

---

# AWS Query Example

Instead of specifying:

```text
172.31.10.10

172.31.10.20
```

we can ask AWS:

```text
Give Me All Running Frontend Servers
```

---

# Query Example

Requirements:

```text
Region = us-east-1

Service = EC2

State = Running

Name Tag = frontend
```

---

AWS returns:

```text
frontend-1

frontend-2

frontend-3
```

---

# Dynamic Inventory File

Convention:

```text
*.aws_ec2.yml

or

*.aws_ec2.yaml
```

---

Example

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1
```

---

# What Happens?

Ansible:

```text
Reads Inventory Plugin

Connects To AWS

Discovers Instances
```

automatically.

---

# Filtering Instances

Suppose we want only:

```text
Running Instances
```

---

Example:

```yaml
filters:

  instance-state-name: running
```

---

# Filtering by Name

Example:

```yaml
filters:

  tag:Name: frontend
```

---

Result:

```text
Only Frontend Servers
```

are returned.

---

# Grouping Hosts

Dynamic inventory can automatically create groups.

Example:

```text
frontend

catalogue

user

cart

shipping
```

based on EC2 tags.

---

# Example

AWS Tags:

```text
Name = frontend

Environment = prod
```

---

Ansible can automatically create:

```text
frontend

prod
```

groups.

---

# Real Production Example

Environment:

```text
Auto Scaling Group
```

---

Traffic Increases:

```text
2 Servers
```

become

```text
10 Servers
```

---

Static Inventory:

```text
Must Be Updated Manually
```

---

Dynamic Inventory:

```text
Automatically Detects New Servers
```

---

# Roboshop Example

Suppose:

```text
frontend

catalogue

user

cart
```

servers are recreated.

Instead of:

```ini
[frontend]
54.12.10.20
```

we fetch hosts dynamically using AWS tags.

---

# Benefits

## No Manual Updates

Inventory always stays current.

---

## Works With Autoscaling

New instances automatically appear.

---

## Better Scalability

Suitable for:

```text
10 Servers

100 Servers

1000 Servers
```

---

## Less Human Error

No manual IP management.

---

# Static vs Dynamic Inventory

| Feature | Static Inventory | Dynamic Inventory |
|----------|----------------|------------------|
| Manual Maintenance | Required | Not Required |
| Autoscaling Support | Poor | Excellent |
| Cloud Integration | Limited | Native |
| Accuracy | Can Become Outdated | Always Current |
| Scalability | Limited | High |

---

# Common Mistakes

## Using Static Inventory in Autoscaling Environments

Bad Practice.

Servers frequently change.

---

## Hardcoding IP Addresses

Avoid:

```ini
54.12.10.20
```

when instances are dynamic.

---

## Missing AWS Permissions

The plugin requires:

```text
DescribeInstances
```

permissions.

---

## Wrong Region

Example:

```yaml
regions:
  - us-east-1
```

must match actual deployment region.

---

# Best Practices

## Use Tags Properly

Example:

```text
Name=frontend

Environment=prod
```

---

## Group Instances by Tags

Makes playbooks cleaner.

---

## Use Dynamic Inventory for Cloud Environments

Especially:

```text
AWS

Azure

GCP
```

---

## Test Inventory Before Running Playbooks

Example:

```bash
ansible-inventory \
-i inventory.aws_ec2.yml \
--graph
```

---

# Frequently Asked Interview Questions

## What is Dynamic Inventory?

Dynamic Inventory is a mechanism where Ansible fetches host information automatically from external systems such as AWS, Azure, or GCP instead of using manually maintained inventory files.

---

## Why Do We Need Dynamic Inventory?

Dynamic Inventory solves problems caused by changing infrastructure.

Examples:

- Autoscaling
- Instance Recreation
- Cloud Environments

It ensures inventory is always current.

---

## Which AWS Plugin Is Used for Dynamic Inventory?

```yaml
amazon.aws.aws_ec2
```

This plugin queries AWS and dynamically builds inventory.

---

## What Is the Difference Between Tasks and Plugins?

Tasks perform actions on target servers.

Examples:

```text
Install Package

Restart Service
```

Plugins extend Ansible functionality.

Examples:

```text
Inventory Plugins

Filter Plugins

Lookup Plugins
```

---

## Why Is Dynamic Inventory Important in Autoscaling?

Autoscaling continuously creates and terminates instances.

Static inventories quickly become outdated.

Dynamic inventory automatically discovers new instances and removes terminated ones.

---

## How Can Dynamic Inventory Discover Frontend Servers?

Using filters such as:

```yaml
filters:
  tag:Name: frontend
```

Ansible queries AWS and returns only frontend instances.

---

## How Do You Validate Dynamic Inventory?

Use:

```bash
ansible-inventory \
-i inventory.aws_ec2.yml \
--list
```

or

```bash
ansible-inventory \
-i inventory.aws_ec2.yml \
--graph
```

to verify discovered hosts.

---

# Key Takeaways

- Dynamic Inventory automatically discovers hosts.
- It is essential in cloud and autoscaling environments.
- AWS dynamic inventory uses the `amazon.aws.aws_ec2` plugin.
- Inventory plugins connect to external systems and build inventories dynamically.
- Tags are commonly used to organize and discover instances.
- Dynamic inventory eliminates manual IP management.
- It is a common production-level Ansible feature and a frequent interview topic.