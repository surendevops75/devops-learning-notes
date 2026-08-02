# AWS Global Infrastructure

---

# Introduction

AWS Global Infrastructure is the worldwide network of Regions, Availability Zones (AZs), Edge Locations, Regional Edge Caches, and Local Zones that enables AWS customers to build highly available, fault-tolerant, low-latency, and globally distributed applications.

Organizations serving customers across multiple countries require infrastructure that provides high availability, disaster recovery, low latency, and regulatory compliance. AWS Global Infrastructure delivers these capabilities through a globally distributed cloud platform.

AWS Global Infrastructure consists of

- AWS Regions
- Availability Zones (AZs)
- Edge Locations
- Regional Edge Caches
- Local Zones
- Wavelength Zones
- Direct Connect Locations
- Global Backbone Network

It forms the foundation for every AWS service.

---

# What is AWS Global Infrastructure?

AWS Global Infrastructure is the physical and network infrastructure that powers AWS services worldwide.

It enables organizations to

- Deploy Applications Globally
- Improve Availability
- Reduce Latency
- Support Disaster Recovery
- Meet Compliance Requirements

Workflow

```text
Users

↓

AWS Global Network

↓

Region

↓

Availability Zone

↓

AWS Services
```

---

# Why AWS Global Infrastructure?

Without Global Infrastructure

```text
Single Datacenter

↓

Hardware Failure

↓

Application Down
```

Problems

- Single Point of Failure
- High Latency
- Poor Disaster Recovery
- Limited Scalability

With AWS Global Infrastructure

```text
Multiple Regions

↓

Multiple AZs

↓

Fault Isolation

↓

High Availability
```

---

# Real World Problem Statement

A multinational company serves customers in

- India
- United States
- Europe
- Australia

Requirements

- Low Latency
- Global Availability
- Disaster Recovery
- Regulatory Compliance

AWS Global Infrastructure provides worldwide deployment options.

---

# Core Components

AWS Global Infrastructure consists of

- Regions
- Availability Zones
- Edge Locations
- Regional Edge Caches
- Local Zones
- Wavelength Zones
- Direct Connect Locations
- Global Backbone

---

# AWS Region

A Region is a geographical area containing multiple isolated Availability Zones.

Examples

- Mumbai
- Singapore
- Frankfurt
- Virginia
- Sydney
- Tokyo

Each Region is independent.

---

# Characteristics of a Region

Each Region has

- Multiple Availability Zones
- Independent Power
- Independent Cooling
- Independent Networking
- Low-Latency Connectivity

Regions provide fault isolation.

---

# Availability Zone (AZ)

An Availability Zone is one or more physically separate data centers within a Region.

Example

```text
Mumbai Region

│

├── ap-south-1a

├── ap-south-1b

└── ap-south-1c
```

AZs are connected by high-speed private fiber.

---

# Benefits of Multiple AZs

Deploying resources across multiple AZs provides

- High Availability
- Fault Tolerance
- Automatic Failover
- Business Continuity

---

# Multi-AZ Architecture

```text
Internet

↓

Application Load Balancer

↓

AZ-A        AZ-B

│            │

EC2          EC2

↓

Amazon RDS Multi-AZ
```

Applications continue operating even if one AZ becomes unavailable.

---

# Region Isolation

Regions are isolated from each other.

Benefits

- Fault Isolation
- Compliance
- Independent Failures
- Separate Disaster Recovery

Example

```text
Mumbai

↓

Independent

↓

Singapore
```

---

# Edge Locations

Edge Locations are globally distributed sites used by AWS content delivery and networking services.

Used by

- Amazon CloudFront
- AWS Shield
- AWS WAF
- Route 53

Benefits

- Low Latency
- Faster Content Delivery
- Improved User Experience

---

# Regional Edge Caches

Regional Edge Caches sit between Edge Locations and AWS Regions.

Benefits

- Larger Cache Capacity
- Improved Cache Hit Ratio
- Reduced Origin Requests

Architecture

```text
User

↓

Edge Location

↓

Regional Edge Cache

↓

Origin Server
```

---

# Local Zones

Local Zones extend AWS infrastructure closer to large population centers.

Benefits

- Low Latency
- Local Compute
- Local Storage

Use Cases

- Gaming
- Media Rendering
- Machine Learning
- Real-Time Analytics

---

# Wavelength Zones

Wavelength Zones bring AWS services into 5G networks.

Benefits

- Ultra-Low Latency
- Mobile Applications
- Edge Computing

Use Cases

- Autonomous Vehicles
- AR/VR
- IoT
- Smart Manufacturing

---

# Direct Connect Locations

Direct Connect Locations provide dedicated private connectivity between customer data centers and AWS.

Benefits

- Consistent Performance
- Lower Latency
- Increased Security
- Reduced Internet Dependency

---

# AWS Global Backbone

AWS operates a private global fiber network connecting Regions.

Benefits

- Secure Traffic
- Low Latency
- High Bandwidth
- Global Connectivity

Customer traffic remains on the AWS network whenever possible.

---

# Summary

AWS Global Infrastructure is the worldwide foundation of AWS services, consisting of Regions, Availability Zones, Edge Locations, Regional Edge Caches, Local Zones, Wavelength Zones, Direct Connect Locations, and the AWS Global Backbone. Together, these components provide high availability, fault tolerance, low latency, disaster recovery, and global scalability for enterprise cloud applications.

---

