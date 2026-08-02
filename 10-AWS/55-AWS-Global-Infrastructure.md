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

# Region Selection Strategy

Choosing the appropriate AWS Region is one of the most important architectural decisions.

Consider

- Customer Location
- Latency Requirements
- Service Availability
- Regulatory Compliance
- Disaster Recovery
- Pricing

Example

```text
Indian Customers

↓

Mumbai Region

↓

Lower Latency
```

---

# Multi-Region Architecture

Multi-Region architectures improve availability and disaster recovery.

Architecture

```text
Users

↓

Route 53

↓

Mumbai Region

↓

Singapore Region

↓

Automatic Failover
```

Benefits

- Disaster Recovery
- Global Availability
- Regional Isolation
- Business Continuity

---

# Disaster Recovery Strategies

Common strategies

- Backup and Restore
- Pilot Light
- Warm Standby
- Multi-Site Active/Active

Each strategy balances cost, recovery time, and complexity.

---

# Backup and Restore

Architecture

```text
Production

↓

Backup

↓

Amazon S3

↓

Restore During Disaster
```

Characteristics

- Lowest Cost
- Highest Recovery Time

---

# Pilot Light

A minimal version of the production environment runs continuously.

Workflow

```text
Primary Region

↓

Replication

↓

Minimal Environment

↓

Scale During Disaster
```

Suitable for moderate Recovery Time Objectives (RTOs).

---

# Warm Standby

A scaled-down production environment remains running in another Region.

Benefits

- Faster Recovery
- Moderate Cost
- Easier Failover

---

# Multi-Site Active/Active

Applications run simultaneously in multiple Regions.

Architecture

```text
Users

↓

Route 53

↓

Mumbai

↓

Virginia

↓

Both Regions Active
```

Provides the highest availability but also the highest cost.

---

# Global Services vs Regional Services

Global Services

- IAM
- Route 53
- AWS Organizations
- AWS WAF (Global for CloudFront)
- CloudFront

Regional Services

- EC2
- RDS
- EKS
- ECS
- Lambda
- VPC
- S3 (Buckets are region-specific, although the service is global)

Understanding the scope of each service is important for architecture design.

---

# Route 53 Integration

Amazon Route 53 routes users to the best AWS Region.

Routing Policies

- Simple
- Weighted
- Latency-Based
- Geolocation
- Geoproximity
- Failover
- Multi-Value Answer

Example

```text
User

↓

Route 53

↓

Nearest AWS Region
```

---

# CloudFront Integration

CloudFront uses Edge Locations to deliver cached content.

Workflow

```text
User

↓

Edge Location

↓

Cached Content

↓

Origin (AWS Region)
```

Benefits

- Lower Latency
- Reduced Origin Load
- Faster Content Delivery

---

# AWS Direct Connect Integration

Direct Connect provides private connectivity to AWS Regions.

Architecture

```text
Corporate Data Center

↓

Direct Connect

↓

AWS Region
```

Suitable for

- Hybrid Cloud
- Large Data Transfers
- Enterprise Applications

---

# High Availability Architecture

```text
Internet

↓

Route 53

↓

Application Load Balancer

↓

Availability Zone A

↓

Availability Zone B

↓

Amazon RDS Multi-AZ
```

Ensures application availability even during infrastructure failures.

---

# Global Load Distribution

Global traffic can be distributed using Route 53.

Example

```text
Asia Users

↓

Mumbai

------------

Europe Users

↓

Frankfurt

------------

US Users

↓

Virginia
```

Improves user experience by reducing latency.

---

# AWS CLI

List Regions

```bash
aws ec2 describe-regions
```

Describe Availability Zones

```bash
aws ec2 describe-availability-zones
```

---

# Terraform

```hcl
provider "aws" {

  region = "ap-south-1"

}
```

Multi-Region Example

```hcl
provider "aws" {

  alias  = "mumbai"

  region = "ap-south-1"

}

provider "aws" {

  alias  = "virginia"

  region = "us-east-1"

}
```

---

# CloudFormation

```yaml
Resources:

  EC2Instance:

    Type: AWS::EC2::Instance
```

CloudFormation stacks are deployed per Region.

---

# Python (Boto3)

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_regions()

print(response["Regions"])
```

---

# Enterprise Production Architecture

```text
                  Global Users

                       │

                  Amazon Route 53

                       │

        ┌──────────────┼──────────────┐

        │                             │

 Mumbai Region                 Virginia Region

        │                             │

   ALB • EC2 • RDS             ALB • EC2 • RDS

        │                             │

        └──────────────┼──────────────┘

                       │

               AWS Global Backbone

                       │

     CloudFront • Direct Connect • Edge Locations
```

---

# Best Practices

- Deploy production workloads across multiple Availability Zones
- Use multiple Regions for disaster recovery
- Choose Regions close to end users
- Use Route 53 health checks and failover routing
- Enable CloudFront for global content delivery
- Use Direct Connect for hybrid workloads
- Replicate critical data across Regions
- Test disaster recovery procedures regularly
- Understand global vs regional services
- Monitor regional service health
- Plan for compliance and data residency requirements
- Design for fault isolation

---

# Common Mistakes

- Deploying production workloads in a single Availability Zone
- Confusing Regions with Availability Zones
- No disaster recovery strategy
- Ignoring latency requirements
- Not testing failover
- Choosing Regions based only on price
- Assuming all AWS services are global
- No cross-region backups
- Ignoring compliance requirements
- Overlooking network latency

---

# Troubleshooting

## High Application Latency

Check

- Region Selection
- CloudFront Configuration
- Route 53 Routing Policy
- Network Path

---

## Failover Not Working

Verify

- Route 53 Health Checks
- Secondary Region
- DNS Configuration
- Application Health

---

## Cross-Region Replication Failed

Check

- IAM Permissions
- Destination Region
- Replication Rules
- Service Configuration

---

## Direct Connect Connectivity Issues

Verify

- Virtual Interface
- BGP Status
- Physical Connection
- Routing Configuration

---

## Region Not Available

Check

- AWS Service Availability
- Region Enablement
- IAM Permissions

---

# Interview Questions

## Basic

1. What is AWS Global Infrastructure?
2. What is an AWS Region?
3. What is an Availability Zone?
4. What are Edge Locations?
5. What are Local Zones?
6. What are Wavelength Zones?
7. What is the AWS Global Backbone?
8. Why are Regions isolated?
9. What is Multi-AZ deployment?
10. What is Multi-Region deployment?

---

## Intermediate

11. Explain Region selection strategy.
12. Explain disaster recovery patterns.
13. Explain Local Zones.
14. Explain Wavelength Zones.
15. Explain Regional Edge Caches.
16. Explain CloudFront integration.
17. Explain Route 53 integration.
18. Explain Direct Connect integration.
19. Explain Multi-Region architecture.
20. Explain global vs regional services.

---

## Advanced

21. Design a globally available AWS architecture.
22. How would you build a disaster recovery solution for a banking application?
23. Explain Multi-AZ vs Multi-Region.
24. Design a low-latency global application.
25. Explain Route 53 latency-based routing.
26. Design enterprise disaster recovery architecture.
27. Explain Active/Active vs Active/Passive.
28. Design hybrid connectivity using Direct Connect.
29. Explain AWS Global Infrastructure best practices.
30. Best practices for global AWS deployments.

---

# Production Scenarios

### Scenario 1

Your production application is deployed only in one Availability Zone.

How would you redesign it for high availability?

---

### Scenario 2

Customers in Europe experience high latency when accessing an application hosted in Mumbai.

How would AWS Global Infrastructure improve performance?

---

### Scenario 3

A natural disaster causes an entire AWS Region to become unavailable.

Which disaster recovery strategy would provide the fastest recovery?

---

### Scenario 4

A multinational company must keep European customer data within Europe.

How would Region selection help satisfy regulatory requirements?

---

### Scenario 5

A media streaming platform serves users worldwide.

How would CloudFront and Edge Locations improve content delivery?

---

### Scenario 6

An enterprise requires private, low-latency connectivity between its on-premises data center and AWS.

How would AWS Direct Connect support this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Region | Geographic AWS Location |
| Availability Zone | Isolated Data Centers |
| Edge Location | Content Delivery |
| Regional Edge Cache | Larger Content Cache |
| Local Zone | Low-Latency Local Infrastructure |
| Wavelength Zone | 5G Edge Computing |
| Direct Connect | Private AWS Connectivity |
| Global Backbone | AWS Private Network |
| Multi-AZ | High Availability |
| Multi-Region | Disaster Recovery & Global Availability |

---

# Summary

AWS Global Infrastructure is the worldwide foundation of AWS cloud services, consisting of Regions, Availability Zones, Edge Locations, Regional Edge Caches, Local Zones, Wavelength Zones, Direct Connect locations, and the AWS Global Backbone. By combining Multi-AZ deployments, Multi-Region architectures, Route 53 routing, CloudFront caching, and Direct Connect connectivity, organizations can build highly available, fault-tolerant, low-latency, and globally scalable applications that meet enterprise reliability and compliance requirements.