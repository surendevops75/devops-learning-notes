# Terraform Project - VPC and Subnet Design

## Introduction

After understanding:

```text
IP Addressing

CIDR

Subnetting

Network and Host Portions
```

the next step is to design a production-ready network in AWS.

In the Roboshop project, Terraform will create:

```text
VPC

Subnets

Route Tables

Internet Gateway

NAT Gateway

Elastic IP

Bastion Host

VPN Connectivity

NACLs
```

Understanding these concepts is important before writing Terraform code.

---

# What is a VPC?

VPC stands for:

```text
Virtual Private Cloud
```

A VPC is an isolated network inside AWS where we deploy our resources.

Examples:

```text
EC2

RDS

EKS

Load Balancers

NAT Gateway
```

---

# Why Do We Need a VPC?

Without VPC:

```text
No Network Isolation

No Traffic Control

No Security Boundaries
```

With VPC:

```text
Private Network

Custom IP Ranges

Controlled Access

Traffic Segregation
```

---

# Real World Analogy

Consider your home address:

```text
PINCODE-STREET-HOUSE
```

Example:

```text
534673-33-25
```

Similarly in AWS:

```text
VPC → City/Village

Subnet → Street

EC2 → House
```

---

# Village Analogy

```text
VPC               → Village

Subnet            → Street

Route Table       → Road

Internet Gateway  → Main Gate

EC2               → House

NAT Gateway       → Shared Internet Connection
```

---

# Roboshop VPC Design

VPC CIDR:

```text
10.0.0.0/16
```

This provides:

```text
65,536 IP Addresses
```

---

# Subnet Design

Public Subnets:

```text
10.0.1.0/24

10.0.2.0/24
```

Private Subnets:

```text
10.0.11.0/24

10.0.12.0/24
```

---

# Why Multiple Subnets?

Benefits:

```text
High Availability

Security

Traffic Isolation

Maintenance
```

---

# Public Subnets

Resources exposed to the internet.

Examples:

```text
Application Load Balancer

NAT Gateway

Bastion Host
```

---

# Public Subnet Example

```text
10.0.1.0/24

10.0.2.0/24
```

---

# Private Subnets

Resources not directly accessible from the internet.

Examples:

```text
Application Servers

Database Servers

Cache Servers

Message Queues
```

---

# Private Subnet Example

```text
10.0.11.0/24

10.0.12.0/24
```

---

# Availability Zones

For High Availability:

```text
Public-1A

Public-1B

Private-1A

Private-1B
```

should be distributed across multiple AZs.

Example:

```text
us-east-1a

us-east-1b
```

---

# Route Tables

A route table controls:

```text
Where Traffic Goes
```

Think of it as:

```text
Road Map
```

for subnet traffic.

---

# Route Table Example

```text
Destination → Target
```

Example:

```text
0.0.0.0/0 → Internet Gateway
```

---

# Public Route Table

Routes:

```text
10.0.0.0/16 → Local

0.0.0.0/0 → IGW
```

Result:

```text
Internet Access Enabled
```

---

# Private Route Table

Routes:

```text
10.0.0.0/16 → Local

0.0.0.0/0 → NAT Gateway
```

Result:

```text
Outbound Internet Access Only
```

---

# Internet Gateway (IGW)

IGW stands for:

```text
Internet Gateway
```

Purpose:

```text
Connect VPC To Internet
```

---

# Why IGW?

Without IGW:

```text
Public Subnets Cannot Access Internet
```

---

# Traffic Flow

```text
User
   ↓
Internet
   ↓
Internet Gateway
   ↓
Public Subnet
```

---

# NAT Gateway

Question:

```text
How Can Private Servers Download Packages?
```

Example:

```bash
dnf install mysql-server
```

or

```bash
yum update -y
```

These require internet access.

---

# Problem

Private Servers:

```text
Should Not Be Publicly Accessible
```

but still need:

```text
Software Updates

Package Downloads

OS Patches
```

---

# Solution

```text
NAT Gateway
```

---

# What is NAT Gateway?

NAT stands for:

```text
Network Address Translation
```

Purpose:

```text
Provide Outbound Internet Access

To Private Subnets
```

---

# NAT Traffic Flow

```text
Private EC2
      ↓
Private Route Table
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

---

# NAT Gateway Placement

NAT Gateway must be deployed in:

```text
Public Subnet
```

because it needs internet connectivity.

---

# Elastic IP (EIP)

NAT Gateway requires:

```text
Elastic IP
```

---

# What is Elastic IP?

Elastic IP is:

```text
Static Public IP Address
```

Example:

```text
44.201.100.50
```

---

# Why NAT Needs EIP?

Internet responses must know:

```text
Where To Return Traffic
```

Static public IP solves this.

---

# Public vs Private Subnets

## Public Subnet

Route:

```text
0.0.0.0/0 → IGW
```

Resources:

```text
ALB

NAT

Bastion
```

Internet Access:

```text
Yes
```

---

## Private Subnet

Route:

```text
0.0.0.0/0 → NAT
```

Resources:

```text
Applications

Databases
```

Internet Access:

```text
Outbound Only
```

---

# Bastion Host

Bastion Host is:

```text
Jump Server
```

used to access private infrastructure.

---

# Why Bastion?

Private EC2 instances:

```text
Do Not Have Public IPs
```

Direct SSH access is not possible.

---

# Bastion Architecture

```text
Engineer Laptop
        ↓
Bastion Host
        ↓
Private EC2
```

---

# Bastion Location

Deploy in:

```text
Public Subnet
```

because engineers connect from the internet.

---

# VPN Access

Instead of Bastion:

```text
Laptop
   ↓
VPN
   ↓
Private Network
```

---

# Benefits

```text
More Secure

No Public SSH

Corporate Access Model
```

---

# Production Architecture

```text
Internet
    ↓
ALB
    ↓
Application Servers
    ↓
Database Servers
```

Access Flow:

```text
Users
  ↓
ALB (Public)
  ↓
App Servers (Private)
  ↓
DB Servers (Private)
```

---

# NACL

NACL stands for:

```text
Network Access Control List
```

---

# Purpose

Provides:

```text
Subnet Level Security
```

---

# NACL Attached To

```text
Subnet
```

Not individual instances.

---

# Example

Private Subnet:

```text
10.0.11.0/24
```

Bastion Subnet:

```text
10.0.1.0/24
```

Allow:

```text
SSH

Source: 10.0.1.0/24
```

---

# Security Layers

```text
NACL → Subnet Level

Security Group → Instance Level
```

---

# Production Roboshop Architecture

```text
VPC (10.0.0.0/16)

├── Public-1A (10.0.1.0/24)
│     ├── ALB
│     ├── NAT Gateway
│     └── Bastion
│
├── Public-1B (10.0.2.0/24)
│
├── Private-1A (10.0.11.0/24)
│     ├── Catalogue
│     ├── User
│     ├── Cart
│
└── Private-1B (10.0.12.0/24)
      ├── MongoDB
      ├── Redis
      ├── MySQL
      └── RabbitMQ
```

---

# Common Interview Questions

## What is a VPC?

### Short Answer

A VPC is an isolated virtual network in AWS where resources are deployed securely.

### Detailed Explanation

A VPC provides complete control over IP addressing, subnetting, routing, and security boundaries.

### Production Example

Creating a dedicated VPC for Roboshop infrastructure.

---

## What is the Difference Between Public and Private Subnet?

### Short Answer

Public subnets have internet access through an IGW, while private subnets access the internet through a NAT Gateway.

### Detailed Explanation

Public resources can receive internet traffic. Private resources cannot receive direct internet traffic but can initiate outbound connections.

### Production Example

ALB in public subnet and application servers in private subnets.

---

## Why Do We Need a NAT Gateway?

### Short Answer

To provide outbound internet access to private servers.

### Detailed Explanation

Private servers require package downloads, OS updates, and external connectivity without exposing them to inbound internet traffic.

### Production Example

Application servers downloading software updates.

---

## Why Is NAT Gateway Placed in Public Subnet?

### Short Answer

Because NAT Gateway itself requires internet connectivity through an Internet Gateway.

### Detailed Explanation

A NAT Gateway acts as an intermediary between private resources and the internet.

### Production Example

Private application servers using NAT Gateway for updates.

---

## What is a Bastion Host?

### Short Answer

A Bastion Host is a jump server used to access private infrastructure.

### Detailed Explanation

Engineers first connect to the Bastion Host and then connect to private servers.

### Production Example

Accessing application servers located in private subnets.

---

# Key Takeaways

```text
VPC provides isolated networking.

Subnets divide the VPC.

Public Subnets use Internet Gateway.

Private Subnets use NAT Gateway.

Elastic IP provides static public IP.

Route Tables control traffic flow.

Bastion Hosts provide administrative access.

VPN provides secure private access.

NACLs provide subnet-level security.

These components form the foundation of production-grade AWS infrastructure.
```