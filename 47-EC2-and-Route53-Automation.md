# EC2 and Route53 Automation

## Introduction

One of the most common DevOps activities is:

```text
Provision Infrastructure
Configure DNS
Deploy Applications
```

Instead of manually creating servers from the AWS Console, we can automate the entire process using:

```text
AWS CLI
Shell Scripts
Route53
```

---

# Infrastructure Automation

Manual Process:

```text
Login AWS Console
      │
      ▼
Create EC2 Instance
      │
      ▼
Wait for Instance
      │
      ▼
Copy IP Address
      │
      ▼
Update DNS Record
      │
      ▼
Verify Access
```

Automation:

```text
Run Script
      │
      ▼
Everything Happens Automatically
```

---

# What is EC2?

EC2 stands for:

```text
Elastic Compute Cloud
```

It is AWS's virtual machine service.

---

# Components Required for EC2 Creation

Before creating an instance, we need:

```text
AMI
Security Group
Instance Type
Key Pair (Optional)
Storage
Subnet
```

---

# AMI

AMI stands for:

```text
Amazon Machine Image
```

It contains:

```text
Operating System
Required Software
Configuration
```

Example:

```text
Amazon Linux
Ubuntu
RedHat
CentOS
```

---

# Example AMI

```text
ami-09c813fb71547fc4f
```

---

# Security Group

Security Group acts as:

```text
Virtual Firewall
```

Controls:

```text
Inbound Traffic
Outbound Traffic
```

---

# Example Security Group

```text
sg-07c8acf3fa6b923fa
```

---

# Instance Type

Determines:

```text
CPU
Memory
Network Capacity
```

---

# Example

```text
t3.micro
```

Common Free Tier instance.

---

# Storage

Storage attached to EC2.

Example:

```text
20 GB
30 GB
50 GB
```

---

# Authentication Before Automation

Before creating resources:

```text
Validate AWS Access
```

---

# Example

```bash
aws s3 ls
```

If successful:

```text
Proceed
```

Otherwise:

```text
Stop Script
```

---

# EC2 Creation Command

Example:

```bash
aws ec2 run-instances \
--image-id ami-09c813fb71547fc4f \
--instance-type t3.micro \
--security-group-ids sg-07c8acf3fa6b923fa
```

---

# What Happens?

AWS creates:

```text
EC2 Instance
```

and returns:

```json
Instance Details
```

---

# Getting Instance ID

Every EC2 instance receives a unique ID.

Example:

```text
i-0682bd1dbd92ce16c
```

---

# Extract Instance ID

```bash
aws ec2 run-instances \
--image-id ami-09c813fb71547fc4f \
--instance-type t3.micro \
--security-group-ids sg-07c8acf3fa6b923fa \
--query 'Instances[0].InstanceId' \
--output text
```

---

# Store Instance ID

```bash
INSTANCE_ID=$(aws ec2 run-instances \
--image-id ami-09c813fb71547fc4f \
--instance-type t3.micro \
--security-group-ids sg-07c8acf3fa6b923fa \
--query 'Instances[0].InstanceId' \
--output text)
```

---

# Why Store Instance ID?

Because later we need:

```text
Public IP
Private IP
Tags
Status
```

All are retrieved using Instance ID.

---

# Querying Instance Information

Command:

```bash
aws ec2 describe-instances
```

Returns detailed instance information.

---

# Get Public IP

```bash
aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query 'Reservations[0].Instances[0].PublicIpAddress' \
--output text
```

---

# Store Public IP

```bash
PUBLIC_IP=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query 'Reservations[0].Instances[0].PublicIpAddress' \
--output text)
```

---

# Get Private IP

```bash
aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query 'Reservations[0].Instances[0].PrivateIpAddress' \
--output text
```

---

# Store Private IP

```bash
PRIVATE_IP=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query 'Reservations[0].Instances[0].PrivateIpAddress' \
--output text)
```

---

# Why Public and Private IPs?

Public IP:

```text
Internet Accessible
```

Examples:

```text
Frontend
Bastion Server
Public Applications
```

---

Private IP:

```text
Internal Communication
```

Examples:

```text
MySQL
Redis
MongoDB
Backend Services
```

---

# Real DevOps Example

Roboshop Components:

```text
frontend
catalogue
user
cart
shipping
payment
mysql
mongodb
redis
rabbitmq
```

---

# Frontend Requirement

Frontend must be accessible from internet.

Use:

```text
Public IP
```

---

# Backend Components

Examples:

```text
MySQL
Redis
MongoDB
RabbitMQ
Catalogue
User
Cart
```

Use:

```text
Private IP
```

---

# Dynamic Logic

Algorithm:

```text
If Component = Frontend

    Use Public IP

Else

    Use Private IP
```

---

# Pseudo Code

```bash
if [ "$COMPONENT" = "frontend" ]
then
    IP=$PUBLIC_IP
else
    IP=$PRIVATE_IP
fi
```

---

# Route53

Route53 is AWS DNS service.

Responsible for:

```text
Domain Management
DNS Records
Routing Traffic
```

---

# Why Route53?

Instead of:

```text
54.88.122.243
```

Use:

```text
frontend.example.com
```

Humans remember names more easily.

---

# Common DNS Records

## A Record

Maps:

```text
Domain → IPv4 Address
```

Example:

```text
frontend.example.com
      │
      ▼
54.88.122.243
```

---

# Updating Route53

AWS CLI command:

```bash
aws route53 change-resource-record-sets
```

---

# Hosted Zone

Every domain belongs to a Hosted Zone.

Example:

```text
Z0948150OFPSYTNVYZOY
```

---

# UPSERT Operation

UPSERT means:

```text
Create If Not Exists

Update If Exists
```

---

# Example

```bash
aws route53 change-resource-record-sets \
--hosted-zone-id Z0948150OFPSYTNVYZOY \
--change-batch '{
  "Changes":[{
      "Action":"UPSERT",
      "ResourceRecordSet":{
         "Name":"frontend.example.com",
         "Type":"A",
         "TTL":1,
         "ResourceRecords":[
            {
              "Value":"54.88.122.243"
            }
         ]
      }
  }]
}'
```

---

# Dynamic Variables

Instead of hardcoding:

```text
frontend.example.com
54.88.122.243
```

Use:

```bash
RECORD_NAME=$COMPONENT.example.com

IP=$PUBLIC_IP
```

---

# Dynamic DNS Automation

Flow:

```text
Create Instance
      │
      ▼
Get Instance ID
      │
      ▼
Get IP Address
      │
      ▼
Create DNS Record
      │
      ▼
Application Ready
```

---

# Multiple Instance Creation

Suppose:

```text
frontend
backend
mysql
redis
mongodb
rabbitmq
```

Need to be created.

---

Without Automation:

```text
Create One by One
```

---

With Automation:

```text
Loop Through Components
```

Create everything automatically.

---

# Production Workflow

```text
Authenticate AWS
        │
        ▼
Create Instance
        │
        ▼
Get Instance ID
        │
        ▼
Get IP Address
        │
        ▼
Update Route53
        │
        ▼
Verify DNS
```

---

# Common Mistakes

## Wrong AMI

Instance creation fails.

---

## Wrong Security Group

Application inaccessible.

---

## Wrong Hosted Zone ID

DNS update fails.

---

## Using Public IP for Internal Services

Security issue.

---

## Hardcoding Values

Makes automation difficult to reuse.

---

# Best Practices

## Use Variables

```bash
AMI_ID
SG_ID
INSTANCE_TYPE
HOSTED_ZONE_ID
```

---

## Use Dynamic Logic

Choose public/private IP automatically.

---

## Validate AWS Authentication

Before resource creation.

---

## Use UPSERT

Avoid duplicate record errors.

---

## Automate Everything

Avoid manual infrastructure creation.

---

# Frequently Asked Interview Questions

## What is AMI?

Amazon Machine Image used to create EC2 instances.

---

## What is a Security Group?

Virtual firewall controlling network access.

---

## What is Instance Type?

Defines CPU and memory resources.

---

## Why Use Public IP?

For internet-facing applications.

---

## Why Use Private IP?

For internal communication.

---

## What is Route53?

AWS DNS service.

---

## What Does UPSERT Mean?

Create if missing, update if existing.

---

## Why Store Instance ID?

To retrieve instance details later.

---

## How Do You Retrieve Public IP?

Using:

```bash
aws ec2 describe-instances
```

with query filters.

---

# Key Takeaways

- EC2 instances can be created using AWS CLI.
- AMI, Security Group, and Instance Type are required for instance creation.
- Instance IDs are used to retrieve instance information.
- Public IPs are used for internet-facing applications.
- Private IPs are used for internal services.
- Route53 manages DNS records.
- UPSERT simplifies DNS automation.
- Dynamic infrastructure automation is a core DevOps skill.
- AWS CLI and Shell Scripting together enable complete infrastructure provisioning.