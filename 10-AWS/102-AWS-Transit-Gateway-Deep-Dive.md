# AWS Transit Gateway (Deep Dive)

# Chapter 1 - AWS Transit Gateway Fundamentals

## What is AWS Transit Gateway?

AWS Transit Gateway (TGW) is a fully managed regional network transit hub that simplifies connectivity between multiple Amazon VPCs, AWS accounts, VPN connections, and AWS Direct Connect connections.

Instead of creating individual VPC Peering connections between every VPC, all networks connect to a single Transit Gateway, making routing centralized, scalable, and easier to manage.

Think of Transit Gateway as the **core router** of an enterprise AWS network.

---

## Why Was Transit Gateway Introduced?

Before Transit Gateway, enterprises relied on VPC Peering.

Example

```text
            VPC-A
           /     \
       VPC-B     VPC-C
        /  \      /  \
    VPC-D  VPC-E VPC-F
```

Problems

- Large number of peering connections
- Difficult route management
- No transitive routing
- Difficult troubleshooting
- Hard to scale

For N VPCs,

Number of peerings required

```
N × (N - 1)
/──────────
     2
```

Example

| Number of VPCs | Required Peerings |
|---------------:|-----------------:|
| 5 | 10 |
| 10 | 45 |
| 20 | 190 |
| 50 | 1225 |
| 100 | 4950 |

Clearly, this architecture does not scale.

---

## Transit Gateway Solution

Instead of connecting VPCs together,

every VPC connects only once.

```text
                  Transit Gateway

         ┌──────────┼──────────┐

      VPC-A      VPC-B      VPC-C

         │           │           │

      VPC-D      VPC-E      VPC-F
```

Benefits

- One attachment per VPC
- Central routing
- Easier operations
- Better scalability

---

# Enterprise Example

A banking company operates

- 180 AWS Accounts
- 320 VPCs
- 40 Amazon EKS Clusters
- 3 AWS Regions
- 2 Data Centers

Instead of maintaining thousands of peering connections,

the company deploys

```text
                  AWS Organization

                        │

                 Network Account

                        │

                 Transit Gateway

      ┌─────────┬─────────┬─────────┐

     Dev       QA      Production

        │         │          │

   Shared Services     Security

                        │

                 Direct Connect

                        │

                On-Premises DC
```

This design provides centralized networking and significantly reduces operational complexity.

---

# Key Features

AWS Transit Gateway provides

- Centralized routing
- Transitive routing
- Multi-account connectivity
- Hybrid networking
- High availability
- Route segmentation
- AWS-managed infrastructure
- Multi-Region connectivity

---

# Transit Gateway Components

| Component | Description |
|------------|-------------|
| Transit Gateway | Central router |
| Attachment | Connects resources to TGW |
| Route Table | Determines packet forwarding |
| Route Propagation | Automatically advertises routes |
| Route Association | Associates attachments with route tables |
| TGW Peering | Connects Transit Gateways across Regions |
| Connect Attachment | SD-WAN connectivity |

---

# Supported Attachments

Transit Gateway supports multiple attachment types.

```text
                    Transit Gateway

        ┌──────────┼──────────┬──────────┐

      VPC       VPN       Direct Connect

                                    │

                             On-Premises

                     TGW Peering

                     Connect Attachment
```

Supported resources

- Amazon VPC
- Site-to-Site VPN
- Direct Connect Gateway
- Transit Gateway
- SD-WAN Appliances

---

# How Transit Gateway Works

Suppose

Application Server

```
10.10.1.15
```

needs to access

Database

```
10.30.10.20
```

Packet flow

```text
Application

↓

VPC Route Table

↓

Transit Gateway Attachment

↓

Transit Gateway

↓

TGW Route Table

↓

Production Attachment

↓

Destination VPC

↓

Database
```

Notice

The Transit Gateway never replaces the VPC Route Table.

Both routing layers work together.

---

# Transit Gateway Architecture

```text
                    Transit Gateway

        ┌──────────┼──────────┬──────────┐

      Dev        QA        Production

        │          │            │

      EKS        EC2          RDS

        │

 Shared Services

        │

   Direct Connect

        │

 On-Premises DC
```

---

# Transit Gateway vs Traditional Router

| Traditional Router | Transit Gateway |
|--------------------|-----------------|
| Physical appliance | Fully managed |
| Hardware upgrades | AWS managed |
| Limited interfaces | Multiple attachments |
| Manual redundancy | Built-in High Availability |
| Limited scalability | Automatically scales |

---

# Transit Gateway vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|----------|-----------------|-------------|
| Centralized Routing | ✅ | ❌ |
| Transitive Routing | ✅ | ❌ |
| Enterprise Scale | Excellent | Poor |
| Hybrid Cloud | Excellent | Limited |
| Operational Complexity | Low | High |
| Supports Hundreds of VPCs | ✅ | ❌ |

---

# When Should You Use Transit Gateway?

Transit Gateway is recommended when

- More than 10–15 VPCs
- Multiple AWS accounts
- Enterprise Landing Zone
- Shared Services architecture
- Hybrid Cloud
- Multiple EKS clusters
- Disaster Recovery
- Centralized networking

Avoid using Transit Gateway for only two VPCs.

In that case,

VPC Peering is simpler and cheaper.

---

# Advantages

- Simplifies enterprise networking
- Supports transitive routing
- Reduces VPC Peering complexity
- Central route management
- Multi-account support
- Hybrid cloud integration
- Easy expansion
- AWS-managed service

---

# Limitations

- Regional service
- Does not support overlapping CIDRs
- Adds additional cost compared to small-scale peering
- Requires careful route table planning
- Route limits apply depending on AWS quotas

---

# Best Practices

- Deploy TGW in a dedicated Network Account.
- Share using AWS RAM.
- Plan CIDR ranges before deployment.
- Use dedicated Transit Subnets.
- Separate Development and Production route tables.
- Enable monitoring.
- Tag every attachment.
- Use Infrastructure as Code (Terraform or CloudFormation).

---

# Common Mistakes

- Continuing to create VPC Peerings after TGW deployment.
- Using one TGW Route Table for every environment.
- Forgetting VPC Route Table updates.
- Creating overlapping CIDR ranges.
- Not planning future expansion.

---

# Interview Questions

## Basic

- What is AWS Transit Gateway?
- Why was Transit Gateway introduced?
- How is Transit Gateway different from VPC Peering?
- What resources can connect to Transit Gateway?

## Intermediate

- Explain packet flow through Transit Gateway.
- What is transitive routing?
- Why do enterprises prefer Transit Gateway?
- Explain supported attachment types.

## Advanced

- Design networking for 250 VPCs.
- How would you migrate from VPC Peering to Transit Gateway?
- Explain centralized enterprise networking using TGW.

---

# Chapter 2 - Transit Gateway Attachments

## What is a Transit Gateway Attachment?

A Transit Gateway Attachment is a logical connection between the Transit Gateway and another AWS networking resource.

Without an attachment,

the Transit Gateway cannot exchange traffic with that resource.

Think of it as a network cable connecting a device to a physical router.

---

## Types of Transit Gateway Attachments

| Attachment | Purpose |
|------------|---------|
| VPC Attachment | Connect VPCs |
| VPN Attachment | Connect Site-to-Site VPN |
| Direct Connect Attachment | Connect Direct Connect Gateway |
| Transit Gateway Peering | Connect TGWs in different Regions |
| Connect Attachment | SD-WAN Connectivity |

---

## VPC Attachment

The most common attachment type.

Architecture

```text
              Transit Gateway

                     │

             VPC Attachment

                     │

      ┌──────────────┴──────────────┐

   Application                 Database
```

Each VPC connects only once to the Transit Gateway.

Unlike VPC Peering,

no direct VPC-to-VPC connections are required.

---

## Creating a VPC Attachment

AWS requires

- Transit Gateway
- Target VPC
- One subnet from each Availability Zone

Example

```text
VPC

├── AZ-A

│      Transit Subnet

├── AZ-B

│      Transit Subnet

└── AZ-C

       Transit Subnet
```

AWS creates ENIs inside these subnets.

Traffic enters and exits through those ENIs.

---

## Why Dedicated Transit Subnets?

Instead of using application subnets,

enterprises usually create

```text
Transit Subnet

Application Subnet

Database Subnet
```

Benefits

- Better organization
- Easier troubleshooting
- Improved security
- Predictable routing

---

## VPN Attachment

Transit Gateway can terminate multiple Site-to-Site VPNs.

```text
On-Premises

↓

IPSec VPN

↓

Transit Gateway

↓

Application VPCs
```

Instead of creating one VPN per VPC,

a single VPN attachment provides connectivity to multiple VPCs through the Transit Gateway.

---

## Direct Connect Attachment

For enterprise hybrid environments,

Direct Connect integrates with Transit Gateway through a Direct Connect Gateway.

```text
On-Premises

↓

Direct Connect

↓

Direct Connect Gateway

↓

Transit Gateway

↓

VPCs
```

Benefits

- High bandwidth
- Low latency
- Centralized hybrid networking

---

