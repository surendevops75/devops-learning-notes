# AWS Architecture Diagrams

---

# Introduction

Architecture diagrams help visualize how AWS services interact to build scalable, secure, and highly available applications.

These diagrams represent common enterprise architectures used in production environments.

---

# Diagram 1

# Basic AWS Infrastructure

```text
                Internet

                   │

               Route53

                   │

             CloudFront

                   │

        Application Load Balancer

                   │

          Auto Scaling Group

        ┌─────────┴─────────┐

        │                   │

     EC2 Instance       EC2 Instance

        │                   │

        └─────────┬─────────┘

                  │

           Amazon RDS Multi-AZ
```

Purpose

- Highly Available Web Application
- Auto Scaling
- Load Balancing

---

# Diagram 2

# Three-Tier Architecture

```text
               Users

                 │

           Application LB

                 │

      Presentation Layer

        (Web Servers)

                 │

      Business Logic Layer

      (Application Servers)

                 │

         Database Layer

       Amazon RDS / Aurora
```

Purpose

- Enterprise Applications
- Banking
- ERP
- Healthcare

---

# Diagram 3

# Public and Private Subnets

```text
              AWS VPC

──────────────────────────────────

Public Subnet

├── ALB

├── NAT Gateway

└── Bastion Host

──────────────────────────────────

Private Subnet

├── EC2

├── ECS

├── EKS

└── Lambda (VPC)

──────────────────────────────────

Database Subnet

├── RDS

├── Aurora

└── ElastiCache
```

Purpose

- Secure Network Segmentation
- Private Databases
- Internet Isolation

---

# Diagram 4

# VPC Networking

```text
Internet

↓

Internet Gateway

↓

Public Route Table

↓

Public Subnet

↓

ALB

↓

Private Route Table

↓

Private Subnet

↓

EC2

↓

RDS
```

Purpose

- Secure VPC Design
- Internet Access
- Private Backend

---

# Diagram 5

# High Availability

```text
Users

↓

Route53

↓

Application Load Balancer

↓

───────────────

AZ-A

EC2

↓

───────────────

AZ-B

EC2

↓

───────────────

Amazon RDS Multi-AZ
```

Purpose

- Fault Tolerance
- Multi-AZ Deployment

---

# Diagram 6

# Auto Scaling Architecture

```text
Users

↓

ALB

↓

Auto Scaling Group

↓

EC2

EC2

EC2

↓

CloudWatch

↓

Scaling Policy
```

Purpose

- Automatic Scaling
- High Availability

---

# Diagram 7

# Static Website Hosting

```text
Users

↓

Route53

↓

CloudFront

↓

Amazon S3

↓

Static Website
```

Purpose

- Static Websites
- Documentation
- Portfolio Sites

---

# Diagram 8

# Dynamic Web Application

```text
Users

↓

CloudFront

↓

AWS WAF

↓

Application Load Balancer

↓

EC2 Auto Scaling

↓

Amazon RDS

↓

Amazon S3
```

Purpose

- Enterprise Web Applications

---

# Diagram 9

# Multi-AZ Database

```text
Application

↓

Primary RDS

↓

Synchronous Replication

↓

Standby RDS

↓

Automatic Failover
```

Purpose

- High Availability
- Disaster Recovery

---

# Diagram 10

# Read Replica Architecture

```text
Application

↓

Primary Database

↓

Read Replica 1

↓

Read Replica 2

↓

Analytics
```

Purpose

- Read Scaling
- Reporting
- Analytics

---

# Diagram 11

# Storage Architecture

```text
Application

↓

Amazon S3

↓

Lifecycle Policy

↓

Glacier

↓

Deep Archive
```

Purpose

- Backup
- Archiving
- Cost Optimization

---

# Diagram 12

# Hybrid Connectivity

```text
On-Premises

↓

VPN / Direct Connect

↓

Transit Gateway

↓

AWS VPC

↓

Applications
```

Purpose

- Hybrid Cloud
- Enterprise Connectivity

---

# Diagram 13

# VPC Peering

```text
VPC-A

↓

VPC Peering

↓

VPC-B
```

Purpose

- Private Communication
- Cross-VPC Connectivity

---

# Diagram 14

# Transit Gateway

```text
Transit Gateway

──────────────

│

├── VPC-A

├── VPC-B

├── VPC-C

├── VPN

└── Direct Connect
```

Purpose

- Centralized Networking

---

# Diagram 15

# Route53 Failover

```text
Users

↓

Route53

↓

Health Check

↓

Primary Region

↓

Failure

↓

Secondary Region
```

Purpose

- Disaster Recovery
- High Availability

---

# Best Practices

- Use Multi-AZ deployments.
- Keep databases in private subnets.
- Place ALBs in public subnets.
- Enable Auto Scaling.
- Use Route53 health checks.
- Use CloudFront for global content delivery.
- Separate workloads across Availability Zones.
- Secure VPCs with Security Groups and NACLs.

---

# Summary

These foundational AWS infrastructure diagrams demonstrate common production architectures including VPC design, subnet segmentation, Auto Scaling, high availability, storage lifecycle, hybrid connectivity, and DNS failover. These patterns are widely used in enterprise cloud environments.

---

# AWS Architecture Diagrams

---

# Introduction

Architecture diagrams help visualize how AWS services interact to build scalable, secure, and highly available applications.

These diagrams represent common enterprise architectures used in production environments.

---

# Diagram 1

# Basic AWS Infrastructure

```text
                Internet

                   │

               Route53

                   │

             CloudFront

                   │

        Application Load Balancer

                   │

          Auto Scaling Group

        ┌─────────┴─────────┐

        │                   │

     EC2 Instance       EC2 Instance

        │                   │

        └─────────┬─────────┘

                  │

           Amazon RDS Multi-AZ
```

Purpose

- Highly Available Web Application
- Auto Scaling
- Load Balancing

---

# Diagram 2

# Three-Tier Architecture

```text
               Users

                 │

           Application LB

                 │

      Presentation Layer

        (Web Servers)

                 │

      Business Logic Layer

      (Application Servers)

                 │

         Database Layer

       Amazon RDS / Aurora
```

Purpose

- Enterprise Applications
- Banking
- ERP
- Healthcare

---

# Diagram 3

# Public and Private Subnets

```text
              AWS VPC

──────────────────────────────────

Public Subnet

├── ALB

├── NAT Gateway

└── Bastion Host

──────────────────────────────────

Private Subnet

├── EC2

├── ECS

├── EKS

└── Lambda (VPC)

──────────────────────────────────

Database Subnet

├── RDS

├── Aurora

└── ElastiCache
```

Purpose

- Secure Network Segmentation
- Private Databases
- Internet Isolation

---

# Diagram 4

# VPC Networking

```text
Internet

↓

Internet Gateway

↓

Public Route Table

↓

Public Subnet

↓

ALB

↓

Private Route Table

↓

Private Subnet

↓

EC2

↓

RDS
```

Purpose

- Secure VPC Design
- Internet Access
- Private Backend

---

# Diagram 5

# High Availability

```text
Users

↓

Route53

↓

Application Load Balancer

↓

───────────────

AZ-A

EC2

↓

───────────────

AZ-B

EC2

↓

───────────────

Amazon RDS Multi-AZ
```

Purpose

- Fault Tolerance
- Multi-AZ Deployment

---

# Diagram 6

# Auto Scaling Architecture

```text
Users

↓

ALB

↓

Auto Scaling Group

↓

EC2

EC2

EC2

↓

CloudWatch

↓

Scaling Policy
```

Purpose

- Automatic Scaling
- High Availability

---

# Diagram 7

# Static Website Hosting

```text
Users

↓

Route53

↓

CloudFront

↓

Amazon S3

↓

Static Website
```

Purpose

- Static Websites
- Documentation
- Portfolio Sites

---

# Diagram 8

# Dynamic Web Application

```text
Users

↓

CloudFront

↓

AWS WAF

↓

Application Load Balancer

↓

EC2 Auto Scaling

↓

Amazon RDS

↓

Amazon S3
```

Purpose

- Enterprise Web Applications

---

# Diagram 9

# Multi-AZ Database

```text
Application

↓

Primary RDS

↓

Synchronous Replication

↓

Standby RDS

↓

Automatic Failover
```

Purpose

- High Availability
- Disaster Recovery

---

# Diagram 10

# Read Replica Architecture

```text
Application

↓

Primary Database

↓

Read Replica 1

↓

Read Replica 2

↓

Analytics
```

Purpose

- Read Scaling
- Reporting
- Analytics

---

# Diagram 11

# Storage Architecture

```text
Application

↓

Amazon S3

↓

Lifecycle Policy

↓

Glacier

↓

Deep Archive
```

Purpose

- Backup
- Archiving
- Cost Optimization

---

# Diagram 12

# Hybrid Connectivity

```text
On-Premises

↓

VPN / Direct Connect

↓

Transit Gateway

↓

AWS VPC

↓

Applications
```

Purpose

- Hybrid Cloud
- Enterprise Connectivity

---

# Diagram 13

# VPC Peering

```text
VPC-A

↓

VPC Peering

↓

VPC-B
```

Purpose

- Private Communication
- Cross-VPC Connectivity

---

# Diagram 14

# Transit Gateway

```text
Transit Gateway

──────────────

│

├── VPC-A

├── VPC-B

├── VPC-C

├── VPN

└── Direct Connect
```

Purpose

- Centralized Networking

---

# Diagram 15

# Route53 Failover

```text
Users

↓

Route53

↓

Health Check

↓

Primary Region

↓

Failure

↓

Secondary Region
```

Purpose

- Disaster Recovery
- High Availability

---

# Best Practices

- Use Multi-AZ deployments.
- Keep databases in private subnets.
- Place ALBs in public subnets.
- Enable Auto Scaling.
- Use Route53 health checks.
- Use CloudFront for global content delivery.
- Separate workloads across Availability Zones.
- Secure VPCs with Security Groups and NACLs.

---

# Summary

These foundational AWS infrastructure diagrams demonstrate common production architectures including VPC design, subnet segmentation, Auto Scaling, high availability, storage lifecycle, hybrid connectivity, and DNS failover. These patterns are widely used in enterprise cloud environments.

