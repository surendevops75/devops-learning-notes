# AWS VPC Networking Fundamentals

## Introduction

Before deploying applications in AWS, we need a network where resources can run securely.

AWS provides:

```text
VPC (Virtual Private Cloud)
```

A VPC is an isolated network inside AWS where we can deploy:

```text
EC2 Instances

RDS Databases

Load Balancers

EKS Clusters

NAT Gateways
```

---

# What is a VPC?

VPC stands for:

```text
Virtual Private Cloud
```

Think of it as your own private data center inside AWS.

AWS manages:

```text
Physical Infrastructure
```

You manage:

```text
Network Design

Security

Routing

IP Addressing
```

---

# Real World Analogy

Imagine an address:

```text
PINCODE-STREET-HOUSE-FLOOR-FLAT

534673-33-25-03-302
```

Similarly in AWS:

```text
VPC

Subnet

Server
```

---

# Village Analogy

```text
VPC          → Village

Subnet       → Street

Route Table  → Roads

Internet Gateway → Modem

EC2 Instance → House
```

---

# Why Do We Need VPC?

Benefits:

```text
Network Isolation

Security

Traffic Control

Custom IP Addressing

High Availability
```

---

# High Availability vs Disaster Recovery

## High Availability (HA)

```text
Same Region

Multiple Availability Zones
```

Example:

```text
us-east-1a

us-east-1b
```

If one AZ fails:

```text
Application Continues Running
```

---

## Disaster Recovery (DR)

```text
Multiple Regions
```

Example:

```text
us-east-1

us-west-2
```

If an entire region fails:

```text
Traffic Can Be Shifted
```

---

# Multi Cloud

Instead of one cloud provider:

```text
AWS

Azure

GCP
```

Benefits:

```text
Vendor Independence

Disaster Recovery

Business Continuity
```

---

# DNS Resolution Flow

When a user opens:

```text
www.example.com
```

Resolution happens in this order:

```text
Browser Cache
        ↓
OS Cache
        ↓
ISP DNS Resolver
        ↓
Root Servers
        ↓
TLD Servers
        ↓
Authoritative Name Servers
        ↓
IP Address
```

---

# Domain Delegation Example

Buy Domain:

```text
Hostinger
```

Manage DNS:

```text
Route53
```

Steps:

```text
Buy Domain

Create Hosted Zone

Get Route53 NS Records

Update NS Records In Hostinger

Route53 Becomes DNS Authority
```

---

# IP Address Basics

Example:

```text
192.168.0.1
```

IPv4 contains:

```text
32 Bits
```

---

# IPv4 Structure

```text
4 Octets

8 Bits Per Octet

4 × 8 = 32 Bits
```

Example:

```text
192.168.0.1
```

---

# Binary Basics

Possible Binary Values:

```text
0

1
```

---

# Number of Combinations

Formula:

```text
2^n
```

Examples:

```text
1 Bit = 2 Values

2 Bits = 4 Values

8 Bits = 256 Values
```

---

# 8 Bit Range

Minimum:

```text
00000000 = 0
```

Maximum:

```text
11111111 = 255
```

Therefore:

```text
0 - 255
```

---

# IPv4 Range

```text
32 Bits
```

Total Addresses:

```text
2^32
```

Approximately:

```text
4.29 Billion Addresses
```

---

# Network and Host Portion

IP Address consists of:

```text
Network Portion

Host Portion
```

Example:

```text
10.0.0.0/16
```

Meaning:

```text
16 Bits Network

16 Bits Host
```

---

# CIDR

CIDR stands for:

```text
Classless Inter-Domain Routing
```

CIDR tells us:

```text
Network Size

Available Hosts

Subnet Range
```

---

# Example: /16

```text
10.0.0.0/16
```

Range:

```text
10.0.0.0

to

10.0.255.255
```

Available Addresses:

```text
2^16

65,536 Addresses
```

---

# Example: /24

```text
10.0.1.0/24
```

Range:

```text
10.0.1.0

to

10.0.1.255
```

Available Addresses:

```text
256 Addresses
```

---

# Subnets

A subnet is a partition of a VPC.

Purpose:

```text
Security

Maintenance

Traffic Segregation
```

---

# Example VPC

```text
10.0.0.0/16
```

Subnets:

```text
10.0.1.0/24

10.0.2.0/24

10.0.11.0/24

10.0.12.0/24
```

---

# Public Subnets

Example:

```text
10.0.1.0/24

10.0.2.0/24
```

Usually contain:

```text
Load Balancers

NAT Gateways

Bastion Hosts
```

---

# Private Subnets

Example:

```text
10.0.11.0/24

10.0.12.0/24
```

Usually contain:

```text
Application Servers

Databases

Internal Services
```

---

# Internet Gateway (IGW)

Internet Gateway enables:

```text
Internet Access
```

for public subnets.

Without IGW:

```text
No Internet Connectivity
```

---

# Route Tables

Route tables determine:

```text
Where Traffic Goes
```

Example:

```text
Destination → Target
```

---

# Default Route

```text
0.0.0.0/0
```

Meaning:

```text
Any Destination
```

---

# Public Route

```text
0.0.0.0/0 → Internet Gateway
```

Result:

```text
Internet Access Enabled
```

---

# Private Servers

Private servers:

```text
Do Not Accept Direct Internet Traffic
```

Examples:

```text
Database Servers

Backend Servers
```

---

# Ingress and Egress

## Ingress

Traffic entering a resource.

Example:

```text
Application → Database
```

Database receives traffic.

---

## Egress

Traffic leaving a resource.

Example:

```bash
dnf install mysql-server
```

Server downloads packages.

Traffic originates from server.

This is:

```text
Egress Traffic
```

---

# NAT Gateway

Question:

```text
How Can Private Servers Access Internet?
```

Answer:

```text
NAT Gateway
```

---

# Purpose of NAT Gateway

Allows:

```text
Outgoing Internet Access
```

for private servers.

Examples:

```text
Software Installation

Package Updates

OS Patching

Downloading Dependencies
```

---

# NAT Gateway Placement

NAT Gateway must be placed in:

```text
Public Subnet
```

because it requires internet connectivity.

---

# Elastic IP (EIP)

NAT Gateway requires:

```text
Elastic IP
```

Elastic IP means:

```text
Static Public IP
```

---

# Typical Production VPC Architecture

```text
VPC
│
├── Public Subnet 1A
│
├── Public Subnet 1B
│
├── Private Subnet 1A
│
└── Private Subnet 1B
```

---

# Components of Production VPC

```text
VPC

Subnets

Internet Gateway

Route Tables

Elastic IP

NAT Gateway
```

---

# Bastion Host

A Bastion Host is:

```text
Jump Server
```

used to access private resources.

Architecture:

```text
Engineer
    ↓
Bastion Host
    ↓
Private EC2
```

---

# VPN Access

Alternative approach:

```text
Laptop
   ↓
VPN
   ↓
Private Network
```

Benefits:

```text
Secure Access

No Public Exposure
```

---

# NACL (Network ACL)

NACL stands for:

```text
Network Access Control List
```

Attached at:

```text
Subnet Level
```

Used to:

```text
Allow Traffic

Deny Traffic
```

---

# Example

Private Subnet contains:

```text
Database Server
```

Bastion Host subnet:

```text
10.0.1.0/24
```

NACL must allow:

```text
10.0.1.0/24
```

to access the database subnet.

---

# Frequently Asked Interview Questions

## What is a VPC?

### Short Answer

A VPC is an isolated virtual network in AWS where resources can be deployed securely.

### Detailed Explanation

A VPC provides complete control over IP addressing, routing, subnet design, internet connectivity, and network security. It acts as a logical data center inside AWS.

### Production Example

Deploying EKS, EC2, RDS, and Load Balancers inside a VPC.

---

## What is a Subnet?

### Short Answer

A subnet is a logical partition of a VPC.

### Detailed Explanation

Subnets divide a VPC into smaller networks for security, availability, and workload isolation.

### Production Example

Using public subnets for ALBs and private subnets for application servers.

---

## What is a NAT Gateway?

### Short Answer

A NAT Gateway provides internet access to private subnet resources without exposing them to inbound internet traffic.

### Detailed Explanation

Private servers often need software updates and package downloads. NAT allows outbound internet access while keeping servers private.

### Production Example

Application servers in private subnets downloading updates through a NAT Gateway.

---

## Difference Between Public and Private Subnet?

### Short Answer

Public subnets have routes to an Internet Gateway, while private subnets do not.

### Detailed Explanation

Resources in public subnets can communicate directly with the internet. Private subnet resources typically access the internet through a NAT Gateway.

### Production Example

ALB in public subnet and application servers in private subnet.

---

## What is 0.0.0.0/0?

### Short Answer

0.0.0.0/0 represents all IPv4 addresses.

### Detailed Explanation

It is commonly used as the default route in route tables to direct traffic to an Internet Gateway or NAT Gateway.

### Production Example

```text
0.0.0.0/0 → IGW
```

---

# Key Takeaways

- VPC is an isolated AWS network.
- Subnets partition a VPC.
- CIDR defines network size.
- Public subnets use Internet Gateway.
- Private subnets use NAT Gateway for outbound internet access.
- Route tables control traffic flow.
- NACLs provide subnet-level security.
- Bastion Hosts and VPNs provide secure access to private resources.
- Understanding VPC fundamentals is essential for AWS and Terraform interviews.