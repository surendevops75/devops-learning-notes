# Terraform Project Structure and Module Strategy

## Introduction

After building the VPC and networking layer, the next step is organizing Terraform code properly.

A typical Roboshop Terraform repository may look like:

```text
00-vpc

10-sg

20-ec2
```

Each folder is responsible for a specific layer of infrastructure.

---

# Why Split Terraform Code?

Instead of placing everything in one file:

```text
VPC

Subnets

Security Groups

EC2

Route53

Load Balancers
```

we separate infrastructure into logical layers.

Benefits:

```text
Easy Maintenance

Reusable Components

Clear Dependencies

Better Troubleshooting
```

---

# Typical Roboshop Structure

## 00-vpc

Creates:

```text
VPC

Subnets

Route Tables

Internet Gateway

NAT Gateway

VPC Peering
```

---

## 10-sg

Creates:

```text
Security Groups
```

Examples:

```text
ALB SG

Frontend SG

Catalogue SG

MongoDB SG
```

---

## 20-ec2

Creates:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment

MongoDB

Redis

RabbitMQ

MySQL
```

---

# Understanding Route Table Logic

Suppose an EC2 instance sends traffic to:

```text
10.0.5.25
```

This belongs to:

```text
10.0.0.0/16
```

same VPC.

Route Table:

```text
10.0.0.0/16 → local
```

Meaning:

```text
Keep Traffic Inside VPC
```

---

# Production Example

Source:

```text
Catalogue Server

10.0.11.25
```

Destination:

```text
MongoDB Server

10.0.21.15
```

Route:

```text
10.0.0.0/16 → local
```

Traffic never leaves AWS networking and directly reaches MongoDB through the VPC local route.

---

# Internet Route Example

Destination:

```text
52.87.172.93
```

This IP is not part of:

```text
10.0.0.0/16
```

Route Table checks:

```text
0.0.0.0/0
```

Meaning:

```text
Unknown Network

Send Outside
```

Traffic goes to:

```text
Internet Gateway

or

NAT Gateway
```

depending on subnet type.

---

# Production Example

A private EC2 instance executes:

```bash
dnf install nginx -y
```

Package repository IP:

```text
52.87.172.93
```

Since it is outside:

```text
10.0.0.0/16
```

Route:

```text
0.0.0.0/0 → NAT Gateway
```

Traffic Flow:

```text
Private EC2
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

---

# VPC Peering Example

Roboshop VPC:

```text
10.0.0.0/16
```

Default VPC:

```text
172.31.0.0/16
```

Destination:

```text
172.31.25.237
```

Matches:

```text
172.31.0.0/16
```

Route Table sends traffic through:

```text
VPC Peering Connection
```

instead of Internet Gateway.

---

# Production Example

Roboshop VPC contains:

```text
Application Servers
```

Default VPC contains:

```text
Monitoring Server

172.31.25.237
```

Traffic Flow:

```text
Catalogue
10.0.11.25
      ↓
VPC Peering
      ↓
Monitoring
172.31.25.237
```

Communication happens through AWS private network.

---

# Infrastructure Creation Approaches

Terraform provides three common approaches.

---

# 1. Custom Modules

Developed internally.

Example:

```text
terraform-aws-vpc

terraform-aws-ec2

terraform-aws-rds
```

created by your organization.

---

## Advantages

```text
Full Control

Custom Standards

Custom Security

Company Governance
```

---

## Disadvantages

```text
Development Effort

Maintenance Responsibility
```

---

## Production Example

Platform Team creates:

```text
terraform-aws-vpc
```

All application teams use the same module.

When security standards change:

```text
Update Module Once
```

All projects automatically inherit the changes.

---

# 2. Open Source Modules

Provided by community.

Example:

```text
Terraform Registry Modules
```

---

## Advantages

```text
Fast Development

Battle Tested

No Need To Build
```

---

## Disadvantages

```text
Limited Control

External Dependency

Version Compatibility Issues
```

---

## Production Example

Instead of developing an EKS module from scratch:

```text
terraform-aws-modules/eks/aws
```

can be directly used.

This reduces development effort significantly.

---

# 3. Direct Resource Creation

Example:

```hcl
resource "aws_instance" "catalogue" {

}
```

No modules involved.

---

## Advantages

```text
Simple

Easy To Learn
```

---

## Disadvantages

```text
Code Duplication

Difficult Maintenance

No Reusability
```

---

## Production Example

Creating:

```text
Frontend

Catalogue

User

Cart

Shipping
```

using separate EC2 resources.

If tagging standards change:

```text
Every Resource Must Be Updated Manually
```

---

# Which Approach is Preferred?

Small Learning Projects:

```text
Direct Resources
```

---

Medium Projects:

```text
Custom Modules
```

---

Enterprise Environments:

```text
Custom Modules

or

Approved Open Source Modules
```

---

# Resource Naming Standards

A common naming convention:

```text
project-environment-component
```

---

Examples

```text
roboshop-dev-catalogue

roboshop-dev-mongodb

roboshop-prod-catalogue

roboshop-prod-mongodb
```

---

# Why Naming Standards Matter?

Benefits:

```text
Easy Identification

Cost Tracking

Automation

Troubleshooting
```

---

# Production Example

Engineer receives an alert:

```text
roboshop-prod-mongodb
```

Immediately understands:

```text
Project     → Roboshop

Environment → Production

Component   → MongoDB
```

No investigation required.

---

# Common Interview Questions

## What Does the Local Route Mean?

### Short Answer

The local route allows communication within the same VPC.

### Detailed Explanation

AWS automatically creates a local route for the VPC CIDR. Resources inside the VPC use this route to communicate privately without leaving the AWS network.

### Production Example

Catalogue server communicates with MongoDB server inside:

```text
10.0.0.0/16
```

using the automatically created local route.

### Follow-Up Questions

- Who creates the local route?
- Can we delete the local route?
- Is local route required for subnet communication?

---

## Why Do We Need 0.0.0.0/0?

### Short Answer

It represents all external networks.

### Detailed Explanation

Traffic that does not match a more specific route uses 0.0.0.0/0 and is forwarded to an Internet Gateway or NAT Gateway.

### Production Example

Private EC2 instance downloading OS updates through NAT Gateway.

### Follow-Up Questions

- What is the default route?
- Why is NAT used with private subnets?

---

## What Are the Different Ways to Use Terraform?

### Short Answer

Custom modules, open-source modules, and direct resource creation.

### Detailed Explanation

Each approach provides a different balance between control, development effort, and maintainability.

### Production Example

Enterprise platform teams typically create reusable VPC, EC2, and Security Group modules that application teams consume.

### Follow-Up Questions

- Which approach is most common in enterprises?
- Why are modules preferred?

---

## Why Use Naming Conventions?

### Short Answer

To make infrastructure easier to identify and manage.

### Detailed Explanation

Naming conventions improve visibility, governance, automation, and operational efficiency.

### Production Example

Resource name:

```text
roboshop-prod-mongodb
```

instantly identifies project, environment, and component.

### Follow-Up Questions

- What naming convention do you use?
- How do naming standards help operations teams?

---

# Key Takeaways

```text
Infrastructure should be organized into logical layers.

00-vpc creates networking resources.

10-sg creates security groups.

20-ec2 creates compute resources.

Local routes keep traffic inside the VPC.

0.0.0.0/0 handles external traffic.

VPC peering enables communication between VPCs.

Terraform supports custom modules, open-source modules, and direct resources.

Consistent naming standards are critical in production environments.

Modules are the preferred approach in enterprise Terraform projects.
```