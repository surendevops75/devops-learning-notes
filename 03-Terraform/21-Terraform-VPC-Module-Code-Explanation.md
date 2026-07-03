# Terraform VPC Module Code Explanation

## Introduction

In the previous files, we learned:

```text
Networking Fundamentals

CIDR

Subnets

VPC Design

Route Tables

Internet Gateway

NAT Gateway

VPC Peering
```

Now let's understand the actual Terraform code used to create the Roboshop VPC infrastructure.

This module creates:

```text
VPC

Internet Gateway

Public Subnets

Private Subnets

Database Subnets

Route Tables

Routes

Elastic IP

NAT Gateway

Route Table Associations
```

---

# Naming Convention

The module follows:

```text
project_name-environment-resource_name
```

Example:

```text
roboshop-dev-vpc

roboshop-dev-public-us-east-1a

roboshop-dev-private-us-east-1a

roboshop-dev-database-us-east-1a
```

---

# High Level Architecture

```text
VPC (10.0.0.0/16)

├── Public Subnet 1A
│     ├── ALB
│     ├── NAT Gateway
│     └── Bastion
│
├── Public Subnet 1B
│
├── Private Subnet 1A
│     ├── Catalogue
│     ├── User
│     └── Cart
│
├── Private Subnet 1B
│     ├── Shipping
│     └── Payment
│
├── Database Subnet 1A
│     ├── MongoDB
│     └── Redis
│
└── Database Subnet 1B
      ├── MySQL
      └── RabbitMQ
```

---

# VPC Resource

Terraform Resource:

```hcl
resource "aws_vpc" "main"
```

Purpose:

```text
Creates the main network boundary for the project.
```

Configuration:

```hcl
cidr_block = var.vpc_cidr
```

Example:

```text
10.0.0.0/16
```

---

# enable_dns_hostnames

```hcl
enable_dns_hostnames = true
```

Purpose:

```text
Allows AWS to assign DNS hostnames to EC2 instances.
```

Example:

```text
ip-10-0-11-25.ec2.internal
```

---

# VPC Tags

```hcl
tags = merge(
  var.vpc_tags,
  local.common_tags,
  {
    Name = local.common_name_suffix
  }
)
```

Purpose:

```text
Combine common tags and resource-specific tags.
```

Terraform Concept:

```text
merge()
```

---

# Internet Gateway

Terraform Resource:

```hcl
resource "aws_internet_gateway" "main"
```

Purpose:

```text
Connects the VPC to the Internet.
```

Without IGW:

```text
Public Subnets Cannot Access Internet.
```

---

# Public Subnets

Terraform Resource:

```hcl
resource "aws_subnet" "public"
```

Purpose:

```text
Create public-facing subnets.
```

Examples:

```text
10.0.1.0/24

10.0.2.0/24
```

Used for:

```text
ALB

NAT Gateway

Bastion Host
```

---

# Count Loop

```hcl
count = length(var.public_subnet_cidrs)
```

Purpose:

```text
Create multiple subnets automatically.
```

Example:

```text
public_subnet_cidrs =

[
  "10.0.1.0/24",
  "10.0.2.0/24"
]
```

Terraform creates:

```text
Public Subnet 1A

Public Subnet 1B
```

---

# count.index

```hcl
cidr_block = var.public_subnet_cidrs[count.index]
```

Purpose:

```text
Fetch current subnet CIDR during iteration.
```

Iteration:

```text
count.index = 0 → 10.0.1.0/24

count.index = 1 → 10.0.2.0/24
```

---

# Availability Zone Assignment

```hcl
availability_zone =
local.az_names[count.index]
```

Purpose:

```text
Distribute subnets across Availability Zones.
```

Example:

```text
us-east-1a

us-east-1b
```

---

# map_public_ip_on_launch

```hcl
map_public_ip_on_launch = true
```

Purpose:

```text
Automatically assign public IPs to instances launched in public subnets.
```

Without this:

```text
Instances Won't Receive Public IPs Automatically.
```

---

# Public Subnet Naming

```hcl
Name =
"${local.common_name_suffix}-public-${local.az_names[count.index]}"
```

Example:

```text
roboshop-dev-public-us-east-1a

roboshop-dev-public-us-east-1b
```

---

# Private Subnets

Terraform Resource:

```hcl
resource "aws_subnet" "private"
```

Purpose:

```text
Host application servers.
```

Examples:

```text
Catalogue

User

Cart

Shipping

Payment
```

CIDRs:

```text
10.0.11.0/24

10.0.12.0/24
```

---

# Why Private Subnets?

Benefits:

```text
No Direct Internet Access

Better Security

Application Isolation
```

---

# Database Subnets

Terraform Resource:

```hcl
resource "aws_subnet" "database"
```

Purpose:

```text
Host data layer services.
```

Examples:

```text
MongoDB

Redis

MySQL

RabbitMQ
```

---

# Why Separate Database Subnets?

Benefits:

```text
Additional Security

Traffic Isolation

Compliance Requirements
```

---

# Public Route Table

Terraform Resource:

```hcl
resource "aws_route_table" "public"
```

Purpose:

```text
Control routing for public subnets.
```

---

# Public Route

Terraform Resource:

```hcl
resource "aws_route" "public"
```

Configuration:

```hcl
destination_cidr_block = "0.0.0.0/0"

gateway_id = aws_internet_gateway.main.id
```

Meaning:

```text
Any Internet Traffic

→ Internet Gateway
```

Result:

```text
Internet Access Enabled
```

---

# Private Route Table

Terraform Resource:

```hcl
resource "aws_route_table" "private"
```

Used by:

```text
Application Subnets
```

---

# Database Route Table

Terraform Resource:

```hcl
resource "aws_route_table" "database"
```

Used by:

```text
Database Subnets
```

---

# Elastic IP

Terraform Resource:

```hcl
resource "aws_eip" "nat"
```

Purpose:

```text
Provide a Static Public IP.
```

Used by:

```text
NAT Gateway
```

---

# What is Elastic IP?

Elastic IP:

```text
Static Public IP Address
```

Example:

```text
44.210.100.50
```

---

# NAT Gateway

Terraform Resource:

```hcl
resource "aws_nat_gateway" "nat"
```

Purpose:

```text
Provide Internet Access

To Private Resources
```

---

# Why NAT Gateway?

Private servers need:

```text
yum update

dnf install

OS patches

Package Downloads
```

But should not be publicly accessible.

---

# NAT Gateway Placement

```hcl
subnet_id = aws_subnet.public[0].id
```

Meaning:

```text
Deploy NAT In Public Subnet
```

---

# Why Public Subnet?

NAT Gateway itself needs:

```text
Internet Connectivity
```

through:

```text
Internet Gateway
```

---

# depends_on

```hcl
depends_on = [
  aws_internet_gateway.main
]
```

Purpose:

```text
Ensure Internet Gateway Is Created First.
```

---

# Interview Question

Why use depends_on?

Answer:

```text
Terraform normally determines dependencies automatically.

When explicit ordering is required, depends_on is used.
```

---

# Private Route Through NAT

Terraform Resource:

```hcl
resource "aws_route" "private"
```

Configuration:

```hcl
0.0.0.0/0

→ NAT Gateway
```

Purpose:

```text
Allow outbound internet access from private subnets.
```

---

# Database Route Through NAT

Terraform Resource:

```hcl
resource "aws_route" "database"
```

Configuration:

```hcl
0.0.0.0/0

→ NAT Gateway
```

Purpose:

```text
Allow database servers to download updates.
```

---

# Route Table Associations

Purpose:

```text
Attach Route Tables To Subnets.
```

Without association:

```text
Subnet Doesn't Know Which Route Table To Use.
```

---

# Public Association

Terraform Resource:

```hcl
resource "aws_route_table_association" "public"
```

Result:

```text
Public Subnets

↓

Public Route Table
```

---

# Private Association

Terraform Resource:

```hcl
resource "aws_route_table_association" "private"
```

Result:

```text
Private Subnets

↓

Private Route Table
```

---

# Database Association

Terraform Resource:

```hcl
resource "aws_route_table_association" "database"
```

Result:

```text
Database Subnets

↓

Database Route Table
```

---

# Terraform Concepts Used

## count

```hcl
count = length(...)
```

Used for:

```text
Multiple Resource Creation
```

---

## count.index

```hcl
count.index
```

Used for:

```text
Current Iteration
```

---

## locals

```hcl
local.common_name_suffix
```

Used for:

```text
Centralized Naming
```

---

## merge()

```hcl
merge(...)
```

Used for:

```text
Combining Tags
```

---

## depends_on

```hcl
depends_on = [...]
```

Used for:

```text
Explicit Resource Ordering
```

---

# Common Interview Questions

## Why Create Separate Public, Private and Database Subnets?

### Short Answer

To improve security and isolate workloads.

### Detailed Explanation

Public subnets host internet-facing resources, private subnets host application servers, and database subnets host data services. This layered architecture reduces attack surface and improves security.

---

## Why Is NAT Gateway Created In Public Subnet?

### Short Answer

Because NAT Gateway requires internet connectivity.

### Detailed Explanation

Private servers send traffic to NAT Gateway, NAT Gateway forwards traffic through the Internet Gateway, and responses return through the NAT Gateway.

---

## Why Use Route Table Associations?

### Short Answer

To attach route tables to subnets.

### Detailed Explanation

Without route table associations, subnets cannot determine how traffic should be routed.

---

## Why Use map_public_ip_on_launch?

### Short Answer

To automatically assign public IPs to instances launched in public subnets.

### Detailed Explanation

Public-facing resources need public IP addresses for internet communication.

---

# Key Takeaways

```text
This module creates a complete production-ready VPC.

Public subnets host internet-facing resources.

Private subnets host application servers.

Database subnets host data services.

Internet Gateway enables public internet access.

NAT Gateway provides outbound internet access for private resources.

Route tables control traffic flow.

Associations connect route tables to subnets.

The module demonstrates real-world Terraform practices such as count, locals, merge, and depends_on.
```