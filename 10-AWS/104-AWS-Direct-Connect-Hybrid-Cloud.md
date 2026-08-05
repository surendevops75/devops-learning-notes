# AWS Direct Connect & Hybrid Cloud

# Chapter 1 - Hybrid Cloud Fundamentals

## What is Hybrid Cloud?

A Hybrid Cloud is an architecture where an organization's on-premises infrastructure works together with public cloud services such as AWS.

Instead of moving every application to AWS immediately, organizations run some workloads in their own data center while others run in AWS.

Both environments communicate securely over private or encrypted network connections.

---

## Hybrid Cloud Architecture

```text
                    Users

                      │

              Corporate Network

                      │

          ┌───────────┴───────────┐

          │                       │

    On-Premises DC           AWS Cloud

          │                       │

      VMware                Amazon EC2

      Oracle DB             Amazon EKS

      File Server           Amazon RDS

      Active Directory      Amazon S3

          │                       │

          └───────────┬───────────┘

              VPN / Direct Connect
```

---

## Why Do Companies Use Hybrid Cloud?

Most enterprises have invested millions of dollars in their data centers.

Migrating everything overnight is

- Expensive
- Risky
- Time consuming

Instead,

organizations migrate gradually.

Typical reasons include

- Legacy applications
- Regulatory compliance
- Data residency
- Low latency access
- Existing infrastructure investment
- Disaster recovery
- Business continuity

---

# Common Hybrid Cloud Use Cases

## Database On-Premises

```text
Amazon EKS

↓

Transit Gateway

↓

Direct Connect

↓

Oracle Database
```

---

## Active Directory

```text
AWS

↓

VPN

↓

Active Directory
```

---

## File Servers

```text
EC2

↓

VPN

↓

Windows File Server
```

---

## Disaster Recovery

```text
Primary DC

↓

Replication

↓

AWS
```

---

## Cloud Bursting

During peak traffic

```text
Normal

↓

On-Premises

Peak

↓

AWS Auto Scaling
```

---

# Benefits

- Gradual cloud adoption
- Lower migration risk
- Business continuity
- Disaster recovery
- Better scalability
- Cost optimization
- Compliance support

---

# Challenges

- Network latency
- Security
- DNS integration
- Authentication
- Monitoring
- Routing complexity
- Data synchronization

---

# Hybrid Connectivity Options

AWS supports

| Service | Purpose |
|----------|----------|
| Site-to-Site VPN | Secure IPSec tunnel |
| AWS Direct Connect | Dedicated private connection |
| Transit Gateway | Central networking hub |
| Direct Connect Gateway | Connect DX to multiple VPCs |
| Route53 Resolver | Hybrid DNS |

---

# Typical Enterprise Architecture

```text
                 AWS Organization

                      │

              Transit Gateway

          ┌───────────┼───────────┐

       Production    Shared Services

              │

     Direct Connect Gateway

              │

        AWS Direct Connect

              │

      Corporate Data Center
```

---

# Best Practices

- Design hybrid networking before migration.
- Avoid overlapping CIDRs.
- Deploy redundant connectivity.
- Enable centralized logging.
- Monitor network health.
- Test failover regularly.

---

# Interview Questions

### Basic

- What is Hybrid Cloud?
- Why do companies use Hybrid Cloud?

### Intermediate

- Hybrid Cloud vs Public Cloud
- Benefits of Hybrid Architecture

### Advanced

- Design Hybrid Cloud for a banking application.
- Explain migration from on-premises to AWS.

---

# Chapter 2 - AWS Site-to-Site VPN

## What is Site-to-Site VPN?

AWS Site-to-Site VPN is a fully managed service that creates an encrypted IPSec tunnel between an on-premises network and AWS.

Traffic travels securely over the public internet.

---

# Architecture

```text
On-Premises Router

        │

 IPSec Tunnel 1

        │

 IPSec Tunnel 2

        │

 Virtual Private Gateway

        │

 Amazon VPC
```

AWS automatically provisions two VPN tunnels for high availability.

---

# Components

| Component | Purpose |
|-----------|---------|
| Customer Gateway (CGW) | On-premises VPN device |
| Virtual Private Gateway (VGW) | AWS VPN endpoint |
| VPN Connection | Secure IPSec tunnel |
| BGP | Dynamic routing |
| Static Routes | Manual routing |

---

# Customer Gateway (CGW)

The Customer Gateway represents the on-premises VPN device.

Examples

- Cisco
- Fortinet
- Palo Alto
- Juniper
- Check Point

AWS stores

- Public IP
- BGP ASN
- Routing information

---

# Virtual Private Gateway (VGW)

The Virtual Private Gateway is the AWS-side VPN endpoint.

Architecture

```text
Customer Gateway

↓

Internet

↓

Virtual Private Gateway

↓

Amazon VPC
```

VGW attaches directly to a VPC.

---

# VPN Tunnel Architecture

AWS creates two tunnels.

```text
Customer Gateway

↓

Tunnel 1

↓

Virtual Private Gateway

↑

Tunnel 2
```

If one tunnel fails,

traffic automatically switches to the second tunnel.

---

# Dynamic Routing (BGP)

Recommended for enterprise deployments.

Benefits

- Automatic route exchange
- Automatic failover
- Reduced administration
- Better scalability

---

# Static Routing

Administrator manually configures routes.

Suitable for

- Small environments
- Simple VPN deployments

Not recommended for enterprise architectures.

---

# VPN Packet Flow

```text
Application

↓

Amazon VPC

↓

Virtual Private Gateway

↓

Encrypted Tunnel

↓

Customer Gateway

↓

On-Premises Server
```

---

# VPN Advantages

- Fast deployment
- Low cost
- Secure IPSec encryption
- Highly available
- AWS managed

---

# VPN Limitations

- Uses public internet
- Higher latency
- Bandwidth depends on ISP
- Performance varies

---

# Common Use Cases

- Hybrid Cloud
- Disaster Recovery
- Branch Office Connectivity
- Data Center Extension
- Temporary Migration

---

# Best Practices

- Always use BGP.
- Configure both VPN tunnels.
- Monitor tunnel health.
- Enable CloudWatch alarms.
- Test automatic failover.

---

# Interview Questions

### Basic

- What is Site-to-Site VPN?
- Why does AWS create two VPN tunnels?

### Intermediate

- Customer Gateway vs Virtual Private Gateway
- BGP vs Static Routing

### Advanced

- Design highly available VPN architecture.
- Explain VPN failover process.

---

# Chapter 3 - Customer Gateway (CGW)

## What is a Customer Gateway?

A Customer Gateway (CGW) represents the physical or virtual VPN device located in the customer's on-premises environment.

It is the entry point into the organization's network.

---

# Customer Gateway Architecture

```text
Corporate Network

↓

Firewall

↓

Customer Gateway

↓

Internet

↓

AWS
```

---

# Supported Devices

Examples

- Cisco ISR
- Palo Alto
- Fortinet
- Juniper
- Check Point
- VyOS

---

# Customer Gateway Requirements

- Public IP Address
- BGP ASN (for dynamic routing)
- IPSec Support
- Internet Connectivity

---

# Best Practices

- Deploy redundant CGWs.
- Use BGP.
- Monitor device health.
- Enable logging.
- Secure device access.

---

# Chapter 4 - Virtual Private Gateway (VGW)

## What is a Virtual Private Gateway?

A Virtual Private Gateway (VGW) is the AWS-managed VPN concentrator that terminates Site-to-Site VPN connections.

It attaches directly to a VPC.

---

# Architecture

```text
Customer Gateway

↓

Internet

↓

Virtual Private Gateway

↓

Amazon VPC
```

---

# Responsibilities

VGW

- Terminates VPN tunnels
- Exchanges routes
- Encrypts traffic
- Connects on-premises to VPC
- Supports BGP

---

# VGW vs Transit Gateway VPN

| VGW | Transit Gateway |
|------|-----------------|
| Single VPC | Multiple VPCs |
| Small deployments | Enterprise |
| Limited scalability | Highly scalable |
| Simple routing | Central routing |

---

# Production Example

A company has

- One Data Center
- One Production VPC

Architecture

```text
On-Premises

↓

Customer Gateway

↓

VPN

↓

Virtual Private Gateway

↓

Production VPC
```

Simple,

low-cost,

easy to manage.

---

# Chapter 5 - AWS Direct Connect

## What is AWS Direct Connect?

AWS Direct Connect (DX) is a dedicated private network connection between an organization's on-premises data center and AWS.

Unlike Site-to-Site VPN, Direct Connect does **not** use the public internet.

Traffic flows through a dedicated physical circuit provided by an AWS Direct Connect Partner or at an AWS Direct Connect Location.

---

# Why Direct Connect?

Consider a financial organization hosting:

- Core Banking System
- Oracle Databases
- SAP
- VMware Infrastructure

These systems require:

- Low latency
- High bandwidth
- Predictable performance
- Private connectivity

Using the public internet is not ideal.

Direct Connect provides a dedicated private path.

---

# Direct Connect Architecture

```text
             On-Premises Data Center

                      │

                Corporate Router

                      │

            Dedicated Fiber Circuit

                      │

          AWS Direct Connect Location

                      │

             AWS Direct Connect

                      │

        Direct Connect Gateway

                      │

            Transit Gateway / VGW

                      │

                Amazon VPC
```

---

# Direct Connect Components

| Component | Purpose |
|-----------|---------|
| Customer Router | Enterprise edge router |
| Direct Connect Location | AWS partner facility |
| Dedicated Connection | Physical fiber connection |
| Virtual Interface (VIF) | Logical network connection |
| Direct Connect Gateway | Connects DX to multiple VPCs |
| Transit Gateway | Enterprise routing |

---

# How Direct Connect Works

Packet flow

```text
Application

↓

Amazon EC2

↓

Transit Gateway

↓

Direct Connect Gateway

↓

Direct Connect

↓

Customer Router

↓

On-Premises Server
```

Traffic never traverses the public internet.

---

# Direct Connect Speeds

AWS supports multiple connection speeds.

Common options

- 1 Gbps
- 10 Gbps
- 100 Gbps

Hosted connections may offer smaller bandwidths such as:

- 50 Mbps
- 100 Mbps
- 200 Mbps
- 500 Mbps

Choose based on workload requirements.

---

# Direct Connect Connection Types

## Dedicated Connection

Provisioned directly by AWS.

Characteristics

- High bandwidth
- Single customer
- Enterprise workloads
- Consistent performance

---

## Hosted Connection

Provisioned through a Direct Connect Partner.

Characteristics

- Flexible bandwidth
- Faster provisioning
- Lower entry cost
- Suitable for smaller environments

---

# Virtual Interfaces (VIF)

A Virtual Interface (VIF) is a logical connection over the physical Direct Connect link.

Types

| VIF | Purpose |
|------|---------|
| Private VIF | Access private VPC resources |
| Public VIF | Access AWS public services |
| Transit VIF | Connect to Transit Gateway through Direct Connect Gateway |

---

## Private VIF

Used to access private resources.

```text
On-Premises

↓

Private VIF

↓

VPC

↓

EC2

↓

RDS
```

Traffic stays private.

---

## Public VIF

Provides access to AWS public services.

Example

```text
On-Premises

↓

Public VIF

↓

Amazon S3

↓

DynamoDB

↓

CloudFront
```

Although these services have public endpoints, traffic uses AWS's private backbone after entering AWS.

---

## Transit VIF

Used with Direct Connect Gateway and Transit Gateway.

Architecture

```text
On-Premises

↓

Transit VIF

↓

Direct Connect Gateway

↓

Transit Gateway

↓

Multiple VPCs
```

This is the recommended architecture for enterprise environments.

---

# Direct Connect Gateway (DXGW)

## What is Direct Connect Gateway?

A Direct Connect Gateway enables a single Direct Connect connection to access multiple VPCs, even across different AWS Regions (supported scenarios).

Without DXGW

```text
Direct Connect

↓

VPC-A
```

Only one VPC.

---

With DXGW

```text
Direct Connect

↓

Direct Connect Gateway

↓

Transit Gateway

↓

Production

↓

Development

↓

Shared Services
```

One connection.

Multiple VPCs.

---

# Direct Connect + Transit Gateway

Modern enterprise architecture

```text
               On-Premises

                     │

             Direct Connect

                     │

        Direct Connect Gateway

                     │

            Transit Gateway

      ┌─────────┼─────────┐

    Dev       QA      Production

          Shared Services
```

Advantages

- Central routing
- Hybrid connectivity
- Easy expansion
- Reduced operational complexity

---

# Direct Connect vs Site-to-Site VPN

| Feature | Direct Connect | Site-to-Site VPN |
|----------|----------------|------------------|
| Network | Dedicated | Public Internet |
| Encryption | Optional (MACsec/IPsec depending on design) | IPSec |
| Latency | Lower and more consistent | Higher and variable |
| Bandwidth | High | Internet dependent |
| Reliability | High | ISP dependent |
| Cost | Higher | Lower |
| Enterprise Workloads | Excellent | Good |

---

# Common Production Use Cases

## Database Access

```text
Oracle Database

↓

Direct Connect

↓

Amazon EKS
```

---

## VMware Migration

```text
VMware

↓

Direct Connect

↓

Amazon EC2
```

---

## SAP

```text
SAP

↓

Direct Connect

↓

AWS
```

---

## Backup

```text
AWS Backup

↓

Direct Connect

↓

Data Center
```

---

# Best Practices

- Deploy redundant Direct Connect circuits.
- Use Transit Gateway for enterprise routing.
- Configure VPN as backup.
- Monitor BGP sessions.
- Monitor bandwidth utilization.
- Test failover regularly.
- Avoid overlapping CIDRs.

---

# Common Mistakes

- No backup VPN.
- Single Direct Connect circuit.
- Ignoring BGP monitoring.
- Using VGW for large enterprise environments instead of TGW where centralized routing is needed.
- Poor CIDR planning.

---

# Interview Questions

### Basic

- What is AWS Direct Connect?
- Why use Direct Connect instead of VPN?
- What is a Virtual Interface?

### Intermediate

- Private VIF vs Public VIF vs Transit VIF.
- What is a Direct Connect Gateway?
- Explain Direct Connect architecture.

### Advanced

- Design enterprise hybrid networking using Direct Connect.
- Explain packet flow from AWS to an on-premises database.
- Design highly available Direct Connect architecture.

---

# Chapter 6 - Direct Connect Gateway (DXGW)

## Why Direct Connect Gateway?

Without Direct Connect Gateway,

every VPC requires separate connectivity.

Example

```text
Direct Connect

↓

Production VPC

Another Connection

↓

Development VPC
```

Not scalable.

---

With Direct Connect Gateway

```text
                Direct Connect

                      │

        Direct Connect Gateway

                      │

             Transit Gateway

      ┌─────────┼─────────┐

   Production   Development

        │

  Shared Services
```

One enterprise connection.

---

# Benefits

- Centralized connectivity
- Multiple VPC access
- Better scalability
- Simplified routing
- Enterprise architecture

---

# Packet Flow

```text
Application

↓

Transit Gateway

↓

Direct Connect Gateway

↓

Direct Connect

↓

Corporate Router

↓

Application Server
```

---

# Best Practices

- Pair DXGW with Transit Gateway.
- Use redundant Direct Connect links.
- Monitor BGP.
- Use AWS RAM for centralized networking.
- Test disaster recovery procedures.

---

