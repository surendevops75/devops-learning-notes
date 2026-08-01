# Amazon Virtual Private Cloud (Amazon VPC)

---

# Introduction

Amazon Virtual Private Cloud (Amazon VPC) is the networking foundation of AWS. It enables you to create a logically isolated network where AWS resources such as EC2, RDS, EKS, ECS, Lambda, ElastiCache, and Load Balancers can securely communicate.

Think of a VPC as your organization's private data center in AWS. Just as an on-premises data center contains routers, switches, firewalls, servers, and IP addressing, a VPC provides these networking capabilities in a fully managed cloud environment.

Every production application deployed on AWS depends on a well-designed VPC.

---

# What is Amazon VPC?

Amazon VPC is a virtual network that allows you to define your own IP address range, create subnets, configure routing, apply security controls, and securely connect AWS resources.

A VPC gives you complete control over:

- IP Addressing
- Network Segmentation
- Internet Connectivity
- Private Connectivity
- Firewalls
- Routing
- Hybrid Networking
- DNS Resolution

Unlike a traditional physical network, AWS manages the underlying infrastructure while you control the logical network configuration.

---

# Why Do We Need Amazon VPC?

Without a VPC, every resource would exist on a shared public network.

Problems include:

- No network isolation
- No IP planning
- No subnet separation
- No routing control
- No firewall management
- Difficult compliance
- Increased security risks

A VPC solves these problems by creating an isolated network where only authorized traffic is allowed.

---

# Real-World Problem

A financial organization needs to deploy a banking application.

Requirements:

- Public internet access only to the Load Balancer
- Application servers must remain private
- Databases must never have public IP addresses
- Developers should access servers through a Bastion Host
- High availability across multiple Availability Zones
- Secure communication between application tiers

Amazon VPC enables this architecture through subnets, route tables, gateways, and security controls.

---

# Enterprise Architecture

```text
                               Internet
                                   │
                            Internet Gateway
                                   │
        ┌──────────────────────────────────────────────────┐
        │                    Amazon VPC                    │
        │                                                  │
        │  Public Subnet (AZ-A)      Public Subnet (AZ-B)  │
        │  ────────────────────      ────────────────────  │
        │   ALB        Bastion         ALB        NAT GW    │
        │      │                           │               │
        │──────────────────────────────────────────────────│
        │ Private Subnet (AZ-A)     Private Subnet (AZ-B)  │
        │──────────────────────     ────────────────────── │
        │ EC2     EKS Nodes         EC2      EKS Nodes     │
        │        │                         │               │
        │──────────────────────────────────────────────────│
        │ Database Subnet          Database Subnet         │
        │────────────────────      ─────────────────────   │
        │ Amazon RDS              Amazon RDS (Standby)     │
        └──────────────────────────────────────────────────┘
```

---

# VPC Components

A production VPC typically contains:

- CIDR Block
- Public Subnets
- Private Subnets
- Database Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Elastic IP
- Security Groups
- Network ACLs
- DHCP Option Set
- DNS Support
- DNS Hostnames

Each component serves a specific purpose in networking and security.

---

# CIDR Block

Every VPC starts with a CIDR block.

Example:

```
10.0.0.0/16
```

CIDR (Classless Inter-Domain Routing) defines the IP address range available for your VPC.

Examples:

| CIDR | Total IP Addresses |
|------|-------------------:|
| /16 | 65,536 |
| /17 | 32,768 |
| /18 | 16,384 |
| /20 | 4,096 |
| /22 | 1,024 |
| /24 | 256 |

### Enterprise Note

Choose a CIDR range that does **not overlap** with your on-premises network or other VPCs if future VPC Peering or VPN connectivity is planned.

---

# IPv4 vs IPv6

AWS supports both IPv4 and IPv6.

| IPv4 | IPv6 |
|------|------|
| 32-bit | 128-bit |
| Limited addresses | Vast address space |
| Commonly used | Increasing adoption |
| NAT often required | Direct addressing possible |

Many enterprises deploy dual-stack environments to support both protocols.

---

# Public IP vs Private IP

Every EC2 instance has a private IP.

Public IPs are optional.

| Private IP | Public IP |
|------------|-----------|
| Used inside VPC | Internet accessible |
| Never changes within the instance lifecycle | Can change unless using Elastic IP |
| Internal communication | External communication |

**Production Tip:** Keep application and database servers on private IPs. Expose only load balancers or bastion hosts to the internet.

---

# Subnets

A subnet is a logical subdivision of a VPC.

Instead of placing every resource into one large network, subnets separate workloads based on their purpose.

Example:

```text
VPC (10.0.0.0/16)

├── Public Subnet A (10.0.1.0/24)

├── Public Subnet B (10.0.2.0/24)

├── Private Subnet A (10.0.11.0/24)

├── Private Subnet B (10.0.12.0/24)

├── DB Subnet A (10.0.21.0/24)

└── DB Subnet B (10.0.22.0/24)
```

---

# Public Subnet

A subnet becomes public when:

- It has a route to an Internet Gateway.
- Resources have public IPs (when required).

Typical resources:

- Application Load Balancer
- NAT Gateway
- Bastion Host
- Reverse Proxy

---

# Private Subnet

Private subnets cannot receive traffic directly from the internet.

Typical resources:

- Application Servers
- Kubernetes Worker Nodes
- Internal APIs
- ECS Tasks

Private resources access the internet through a NAT Gateway for updates and package downloads.

---

# Database Subnet

Database subnets are isolated from both the internet and application users.

Typical resources:

- Amazon RDS
- Amazon Aurora
- ElastiCache

Only application servers should communicate with databases.

---

# Multi-AZ Design

A production VPC should span multiple Availability Zones.

Benefits:

- High Availability
- Fault Tolerance
- Disaster Recovery
- Zero Single Point of Failure

Example:

```text
AZ-A

Public
Private
Database

↓

AZ-B

Public
Private
Database
```

---

# Route Tables

Every subnet is associated with a Route Table.

A Route Table determines where packets should be forwarded.

Example:

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | Local |
| 0.0.0.0/0 | Internet Gateway |

### How AWS Selects a Route

AWS uses **Longest Prefix Match**.

Example:

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | Local |
| 10.0.10.0/24 | NAT Gateway |
| 0.0.0.0/0 | Internet Gateway |

Traffic to `10.0.10.15` matches `/24`, not `/16`, because it is the more specific route.

This is a common interview question.

---

# Internet Gateway (IGW)

An Internet Gateway connects your VPC to the public internet.

Functions:

- Enables inbound internet traffic
- Enables outbound internet traffic
- Performs one-to-one NAT for public IPv4 addresses

Without an Internet Gateway, even instances with public IP addresses cannot communicate with the internet.

Traffic flow:

```text
Internet

↓

Internet Gateway

↓

Route Table

↓

Public Subnet

↓

EC2
```

---

# NAT Gateway

A NAT Gateway allows instances in private subnets to initiate outbound internet connections while preventing inbound internet access.

Typical use cases:

- OS updates
- Docker image pulls
- Package installations
- Accessing AWS APIs

Traffic flow:

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

# NAT Gateway vs NAT Instance

| NAT Gateway | NAT Instance |
|-------------|--------------|
| Fully managed | Self-managed EC2 |
| Highly available (per AZ) | Manual HA |
| Auto scaling bandwidth | Limited by instance type |
| No OS maintenance | Requires patching |
| Recommended | Legacy approach |

AWS recommends NAT Gateway for production.

---

# Elastic IP

An Elastic IP is a static public IPv4 address allocated to your AWS account.

Common uses:

- NAT Gateway
- Bastion Host
- Public-facing services

Unlike a standard public IP, an Elastic IP remains allocated until you release it.

---

# Security Groups

Security Groups act as virtual firewalls at the instance or ENI level.

Characteristics:

- Stateful
- Allow rules only
- Evaluated before traffic reaches the instance

Common rules:

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 2049 | TCP | Amazon EFS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |

**Production Tip:** Reference Security Groups instead of IP addresses whenever possible. For example, allow MySQL access from the Application Security Group rather than from an entire subnet.

---

# Network ACL (NACL)

A Network ACL provides subnet-level firewall protection.

Characteristics:

- Stateless
- Supports Allow and Deny rules
- Evaluated in rule-number order
- Applies to all resources within a subnet

Example:

| Rule | Action | CIDR | Port |
|------|--------|------|------|
| 100 | Allow | 10.0.0.0/16 | All |
| 200 | Allow | 0.0.0.0/0 | 443 |
| * | Deny | All | All |

---

# Security Groups vs Network ACLs

| Security Group | Network ACL |
|----------------|-------------|
| Stateful | Stateless |
| Instance level | Subnet level |
| Allow only | Allow & Deny |
| Default allow outbound | Explicit inbound/outbound rules |
| References Security Groups | Uses CIDR ranges |

### Interview Tip

A common question is:

**"Which is evaluated first?"**

Traffic enters the subnet:

1. Network ACL
2. Security Group
3. EC2 Instance

If the NACL blocks the traffic, it never reaches the Security Group.

---

---

# DHCP Option Sets

Every VPC automatically receives DHCP configuration.

DHCP (Dynamic Host Configuration Protocol) provides configuration information to EC2 instances automatically.

It supplies:

- Domain Name
- DNS Server
- NTP Server
- NetBIOS Server (Legacy)
- NetBIOS Node Type

Default Workflow

```text
EC2 Starts

↓

DHCP Request

↓

AWS DHCP Server

↓

IP Address

↓

DNS Server

↓

Hostname

↓

Network Ready
```

You can create a custom DHCP Option Set if your organization uses:

- On-premises DNS
- Active Directory
- Hybrid Cloud

---

# DNS Support

Amazon VPC provides an internal DNS service.

Options

```
DNS Resolution

Enabled

DNS Hostnames

Enabled
```

Recommended for almost every production VPC.

Example

```
Private IP

10.0.1.25

↓

DNS Name

ip-10-0-1-25.ec2.internal
```

---

# Amazon Provided DNS

AWS automatically provides a DNS Resolver inside every VPC.

Example

```
VPC CIDR

10.0.0.0/16

↓

DNS Resolver

10.0.0.2
```

Applications use this resolver for internal name resolution.

---

# VPC Endpoints

Normally, when an EC2 instance accesses Amazon S3:

```text
Private EC2

↓

NAT Gateway

↓

Internet Gateway

↓

Amazon S3
```

This uses public AWS endpoints.

A VPC Endpoint allows private communication without traversing the internet.

```text
Private EC2

↓

VPC Endpoint

↓

Amazon S3
```

Benefits

- Private Communication
- Better Security
- Lower Latency
- No NAT Gateway Required (Gateway Endpoints)
- Reduced Data Transfer Costs

---

# Types of VPC Endpoints

AWS supports two major endpoint types.

| Endpoint | Supports |
|-----------|----------|
| Gateway Endpoint | S3, DynamoDB |
| Interface Endpoint | Most AWS Services |

---

# Gateway Endpoint

Supported Services

- Amazon S3
- Amazon DynamoDB

Architecture

```text
Private EC2

↓

Route Table

↓

Gateway Endpoint

↓

Amazon S3
```

No NAT Gateway required.

No Internet Gateway required.

No Public IP required.

---

# Interface Endpoint

Uses AWS PrivateLink.

Creates an Elastic Network Interface (ENI) inside your subnet.

Example

```text
EC2

↓

Private ENI

↓

PrivateLink

↓

Secrets Manager
```

Supports services such as:

- Secrets Manager
- CloudWatch
- Systems Manager
- ECR
- SNS
- SQS
- KMS

---

# AWS PrivateLink

AWS PrivateLink enables private connectivity to AWS services and supported third-party services.

Benefits

- Private IP communication
- No Internet exposure
- No NAT Gateway
- Secure service access

Example

```text
Application

↓

Private Endpoint

↓

AWS PrivateLink

↓

Amazon ECR
```

---

# VPC Peering

VPC Peering connects two VPCs.

Example

```text
Production VPC

10.0.0.0/16

↓

VPC Peering

↓

Shared Services VPC

172.16.0.0/16
```

Requirements

- Non-overlapping CIDR ranges
- Route Table updates
- Security Group rules

Limitations

- No transitive routing
- Manual configuration
- Point-to-point connectivity

---

# Transit Gateway

Transit Gateway acts as a central networking hub.

Architecture

```text
VPC A

      │

VPC B

      │

Transit Gateway

      │

VPC C

      │

VPN

      │

Direct Connect
```

Benefits

- Centralized routing
- Simplified architecture
- Scalable
- Supports hundreds of VPCs

Best for enterprise environments.

---

# Site-to-Site VPN

AWS Site-to-Site VPN securely connects an on-premises network to AWS.

Architecture

```text
Corporate Data Center

↓

VPN Device

↓

Encrypted Tunnel

↓

Virtual Private Gateway

↓

Amazon VPC
```

Traffic remains encrypted while traveling across the public internet.

---

# AWS Direct Connect

AWS Direct Connect provides a dedicated private network connection between your data center and AWS.

Benefits

- Lower latency
- Consistent bandwidth
- Private connectivity
- Reduced internet dependency

Architecture

```text
Corporate Data Center

↓

Dedicated Fiber Connection

↓

AWS Direct Connect

↓

Amazon VPC
```

Suitable for:

- Banking
- Healthcare
- Government
- Large Enterprises

---

# VPC Flow Logs

Flow Logs capture network traffic information.

Useful for:

- Troubleshooting
- Security investigations
- Compliance
- Traffic analysis

Flow Logs record:

- Source IP
- Destination IP
- Source Port
- Destination Port
- Protocol
- Action (Accept / Reject)
- Bytes
- Packets

Flow Logs do NOT capture packet payloads.

---

# Traffic Flow Example

Internet Request

```text
User

↓

Internet Gateway

↓

Route Table

↓

Security Group

↓

Application Load Balancer

↓

EC2

↓

Application

↓

Amazon RDS
```

Private AWS Service Request

```text
EC2

↓

Interface Endpoint

↓

Secrets Manager
```

---

# AWS Console Walkthrough

1. Open **VPC Dashboard**
2. Create a VPC
3. Specify CIDR block
4. Create Public and Private Subnets
5. Attach Internet Gateway
6. Create Route Tables
7. Associate Route Tables
8. Create NAT Gateway
9. Allocate Elastic IP
10. Configure Security Groups
11. Configure NACLs
12. Launch EC2 instances
13. Test Connectivity

---

# AWS CLI

Create VPC

```bash
aws ec2 create-vpc \
--cidr-block 10.0.0.0/16
```

Create Subnet

```bash
aws ec2 create-subnet \
--vpc-id vpc-12345 \
--cidr-block 10.0.1.0/24
```

Create Internet Gateway

```bash
aws ec2 create-internet-gateway
```

Attach Internet Gateway

```bash
aws ec2 attach-internet-gateway \
--internet-gateway-id igw-123 \
--vpc-id vpc-123
```

Create Route Table

```bash
aws ec2 create-route-table \
--vpc-id vpc-123
```

Create Route

```bash
aws ec2 create-route \
--route-table-id rtb-123 \
--destination-cidr-block 0.0.0.0/0 \
--gateway-id igw-123
```

---

# Terraform

```hcl
resource "aws_vpc" "main" {

  cidr_block = "10.0.0.0/16"

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "production-vpc"

  }

}
```

Create Public Subnet

```hcl
resource "aws_subnet" "public" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.1.0/24"

  availability_zone = "ap-south-1a"

}
```

Internet Gateway

```hcl
resource "aws_internet_gateway" "igw" {

  vpc_id = aws_vpc.main.id

}
```

---

# CloudFormation

```yaml
Resources:

  VPC:

    Type: AWS::EC2::VPC

    Properties:

      CidrBlock: 10.0.0.0/16

      EnableDnsSupport: true

      EnableDnsHostnames: true
```

---

# Python (Boto3)

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.create_vpc(
    CidrBlock="10.0.0.0/16"
)

print(response)
```

---

# Amazon EKS Networking

Amazon EKS uses the Amazon VPC CNI plugin.

Each Pod receives its own VPC IP address.

Architecture

```text
Amazon VPC

↓

Private Subnet

↓

Worker Node

↓

Pod

↓

Elastic Network Interface

↓

Application
```

Benefits

- Native VPC networking
- Security Groups for Pods (supported scenarios)
- Direct communication
- No overlay networking required

---

# Enterprise Use Cases

Amazon VPC is used for:

- Banking Platforms
- Healthcare Applications
- E-commerce Systems
- Kubernetes Clusters
- Hybrid Cloud
- Multi-Account Networking
- Shared Services
- Disaster Recovery
- Secure API Platforms
- Enterprise Data Lakes

---

# Best Practices

- Design CIDR blocks carefully
- Use Multi-AZ architecture
- Keep databases private
- Use NAT Gateway instead of NAT Instance
- Enable DNS support
- Use VPC Endpoints where possible
- Enable Flow Logs
- Apply least-privilege Security Group rules
- Use Infrastructure as Code
- Tag all networking resources

---

# Common Mistakes

- Overlapping CIDR ranges
- Large open Security Groups (0.0.0.0/0)
- Public databases
- Missing Flow Logs
- Single NAT Gateway for critical workloads
- Poor subnet planning
- Ignoring route table associations
- Using public subnets unnecessarily

---

# Troubleshooting

## EC2 Cannot Access Internet

Check:

- Internet Gateway
- Route Table
- Security Group
- Public IP
- NACL

---

## Private EC2 Cannot Download Packages

Verify:

- NAT Gateway
- Route Table
- Elastic IP
- Security Group

---

## VPC Peering Not Working

Check:

- CIDR overlap
- Route Tables
- Security Groups
- Peering status

---

## Cannot Reach RDS

Verify:

- Security Group rules
- Database subnet
- Route Tables
- NACL

---

## DNS Resolution Fails

Check:

- EnableDnsSupport
- EnableDnsHostnames
- DHCP Option Set
- Route 53 Resolver

---

# Interview Questions

### Basic

1. What is Amazon VPC?
2. Why do we use a VPC?
3. What is a CIDR block?
4. Difference between Public and Private Subnets?
5. What is an Internet Gateway?
6. What is a Route Table?
7. What is an Elastic IP?
8. What is a NAT Gateway?
9. What is a Security Group?
10. What is a Network ACL?

### Intermediate

11. Difference between Security Group and NACL?
12. What is Longest Prefix Match?
13. Why are five IP addresses reserved in every subnet?
14. What is a Gateway Endpoint?
15. What is an Interface Endpoint?
16. Explain AWS PrivateLink.
17. What is VPC Peering?
18. What is Transit Gateway?
19. What are VPC Flow Logs?
20. Explain DHCP Option Sets.

### Advanced

21. Difference between NAT Gateway and NAT Instance?
22. How does Route Table evaluation work?
23. Explain hybrid networking.
24. When would you choose Direct Connect over VPN?
25. Explain Multi-AZ networking.
26. How does EKS networking work?
27. What happens if the NAT Gateway fails?
28. How do you troubleshoot internet connectivity issues?
29. How do VPC Endpoints reduce cost?
30. Design a secure production VPC.

---

# Scenario-Based Questions

### Scenario 1

Your EC2 instance has a public IP but cannot access the internet.

How would you troubleshoot?

---

### Scenario 2

Private instances cannot download operating system updates.

What would you verify?

---

### Scenario 3

Application servers cannot connect to Amazon RDS.

What networking components would you inspect?

---

### Scenario 4

Two VPCs cannot communicate after creating a VPC Peering connection.

How would you investigate?

---

### Scenario 5

A company with 40 VPCs needs full connectivity.

Would you choose VPC Peering or Transit Gateway? Why?

---

### Scenario 6

Security reports that developers opened SSH to `0.0.0.0/0`.

What risks does this introduce and how would you fix it?

---

### Scenario 7

Your AWS bill increased significantly because of NAT Gateway charges.

How would you optimize the architecture?

---

### Scenario 8

A private EC2 instance needs to access Amazon S3 without using the internet.

What AWS feature would you implement?

---

### Cheat Sheet

| Component | Purpose |
|----------|---------|
| VPC | Isolated Network |
| Subnet | Network Segment |
| Route Table | Traffic Routing |
| Internet Gateway | Internet Access |
| NAT Gateway | Outbound Internet for Private Subnets |
| Security Group | Stateful Firewall |
| NACL | Stateless Firewall |
| VPC Endpoint | Private AWS Service Access |
| Transit Gateway | Centralized Routing |
| Flow Logs | Network Monitoring |

---

# Summary

Amazon VPC is the networking foundation of AWS. A well-designed VPC combines proper CIDR planning, subnet segmentation, routing, security controls, and private connectivity to build secure, scalable, and highly available cloud environments.

Production architectures typically use Multi-AZ deployments, public subnets only for internet-facing components, private subnets for application workloads, database subnets for managed databases, VPC Endpoints for private AWS service access, Flow Logs for visibility, and Infrastructure as Code for repeatable deployments. Mastering VPC concepts is essential for designing, operating, and troubleshooting enterprise AWS environments.