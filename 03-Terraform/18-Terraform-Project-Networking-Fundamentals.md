# Terraform Project - Networking Fundamentals

## Why Are We Learning Networking?

Before creating infrastructure using Terraform, we must understand networking concepts.

In the Roboshop project, we are going to create:

```text
VPC

Subnets

Route Tables

Internet Gateway

NAT Gateway

Security Groups

EC2 Instances

Load Balancers
```

All of these depend on networking fundamentals.

---

# Real Project Context

Suppose we are building Roboshop.

Services:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment

MongoDB

Redis

MySQL

RabbitMQ
```

Questions:

```text
How do services communicate?

How do users access the application?

How do private servers download packages?

How do databases remain secure?
```

To answer these questions, we need networking knowledge.

---

# Domain Registration and DNS

Suppose we buy a domain from:

```text
Hostinger
```

Example:

```text
daws86s.fun
```

But we want DNS management in:

```text
AWS Route53
```

---

# Steps

## Buy Domain

```text
Hostinger
```

---

## Create Hosted Zone

In:

```text
AWS Route53
```

AWS generates:

```text
NS Records
```

Example:

```text
ns-100.awsdns.com

ns-200.awsdns.net

ns-300.awsdns.org

ns-400.awsdns.co.uk
```

---

## Update NS Records

Login to:

```text
Hostinger
```

Replace existing NS records with Route53 NS records.

---

## Result

Route53 becomes:

```text
Authoritative DNS Server
```

for the domain.

---

# DNS Resolution Flow

When a user opens:

```text
www.daws86s.fun
```

DNS resolution happens as follows:

```text
Browser Cache
        ↓
Operating System Cache
        ↓
ISP DNS Resolver
        ↓
Root DNS Servers
        ↓
TLD Servers (.com/.fun/.org)
        ↓
Authoritative DNS Server
        ↓
IP Address
```

---

# High Availability (HA)

Goal:

```text
Application Should Continue Working
```

even if some infrastructure fails.

---

## Example

Region:

```text
us-east-1
```

Availability Zones:

```text
us-east-1a

us-east-1b
```

Deploying resources in multiple AZs provides:

```text
High Availability
```

---

# Disaster Recovery (DR)

Goal:

```text
Recover From Region Failure
```

---

Example

Primary:

```text
us-east-1
```

Secondary:

```text
us-west-2
```

If an entire region fails:

```text
Traffic Can Be Shifted
```

to another region.

---

# High Availability vs Disaster Recovery

## High Availability

```text
Same Region

Multiple AZs
```

Example:

```text
us-east-1a

us-east-1b
```

---

## Disaster Recovery

```text
Multiple Regions
```

Example:

```text
us-east-1

us-west-2
```

---

# Multi Cloud

Using multiple cloud providers.

Example:

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

# IP Address

Every device connected to a network requires an IP address.

Example:

```text
192.168.1.10
```

IPv4 consists of:

```text
32 Bits
```

---

# Structure of IPv4

Example:

```text
192.168.1.10
```

Contains:

```text
4 Octets

8 Bits Per Octet

4 × 8 = 32 Bits
```

---

# Binary Basics

Computers understand:

```text
0

1
```

called:

```text
Binary
```

---

# Decimal vs Binary

Decimal:

```text
0 1 2 3 4 5 6 7 8 9
```

Binary:

```text
0 1
```

---

# Number of Possible Values

Formula:

```text
2^n
```

---

Examples

```text
1 Bit = 2 Values

2 Bits = 4 Values

3 Bits = 8 Values

8 Bits = 256 Values
```

---

# Binary to Decimal Example

Binary:

```text
10010101
```

Positions:

```text
7 6 5 4 3 2 1 0
```

Calculation:

```text
128 + 16 + 4 + 1
```

Result:

```text
149
```

---

# 8-Bit Range

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

# IPv4 Address Space

IPv4 contains:

```text
32 Bits
```

Total addresses:

```text
2^32
```

Approximately:

```text
4.29 Billion Addresses
```

---

# Network and Host

Every IP address contains:

```text
Network Portion

Host Portion
```

---

Example

```text
10.0.0.0/16
```

Meaning:

```text
First 16 Bits → Network

Last 16 Bits → Host
```

---

# CIDR

CIDR stands for:

```text
Classless Inter-Domain Routing
```

CIDR determines:

```text
Network Size

Host Capacity

Subnet Range
```

---

# Example: /16

Network:

```text
10.0.0.0/16
```

Range:

```text
10.0.0.0

to

10.0.255.255
```

Hosts:

```text
2^16

65,536 Addresses
```

---

# Example: /24

Network:

```text
10.0.1.0/24
```

Range:

```text
10.0.1.0

to

10.0.1.255
```

Hosts:

```text
256 Addresses
```

---

# Why Subnetting?

Suppose we have:

```text
10.0.0.0/16
```

Using one huge network is difficult.

Problems:

```text
Security

Maintenance

Traffic Management
```

Solution:

```text
Subnetting
```

---

# Example Subnets

```text
10.0.1.0/24

10.0.2.0/24

10.0.3.0/24

10.0.4.0/24
```

---

# Understanding /24

Network:

```text
10.0.1.0/24
```

Available Range:

```text
10.0.1.0

to

10.0.1.255
```

Total Addresses:

```text
256
```

---

# Common Interview Questions

## What is CIDR?

### Short Answer

CIDR stands for Classless Inter-Domain Routing and defines network size and host capacity.

### Detailed Explanation

CIDR specifies how many bits belong to the network portion and how many belong to the host portion. It helps in subnet planning and IP allocation.

### Example

```text
10.0.0.0/16
```

16 bits for network and 16 bits for hosts.

---

## What is an IPv4 Address?

### Short Answer

An IPv4 address is a 32-bit address used to identify devices on a network.

### Detailed Explanation

IPv4 consists of four octets, each containing 8 bits, giving a total of 32 bits.

### Example

```text
192.168.1.10
```

---

## Difference Between High Availability and Disaster Recovery?

### Short Answer

High Availability protects against AZ failures, while Disaster Recovery protects against region failures.

### Detailed Explanation

HA uses multiple Availability Zones in the same region. DR uses multiple regions to recover from complete regional outages.

### Example

```text
HA → us-east-1a + us-east-1b

DR → us-east-1 + us-west-2
```

---

## Why Do We Need Subnetting?

### Short Answer

Subnetting improves security, maintenance, and traffic management.

### Detailed Explanation

Instead of placing all resources in a single network, subnetting creates logical network boundaries for better organization and control.

### Example

Separate subnets for frontend, application, and database servers.

---

# Key Takeaways

```text
Networking is the foundation of cloud infrastructure.

DNS converts domain names into IP addresses.

High Availability uses multiple AZs.

Disaster Recovery uses multiple regions.

IPv4 contains 32 bits.

CIDR defines network size.

IP addresses contain Network and Host portions.

Subnetting improves security and manageability.

Understanding these concepts is essential before building VPC infrastructure with Terraform.
```