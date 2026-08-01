# AWS Fundamentals

---

# Introduction

Amazon Web Services (AWS) is the world's leading cloud computing platform, providing more than **250+ fully managed cloud services** that help organizations build, deploy, secure, and scale applications without managing physical infrastructure.

AWS enables businesses to provision resources on demand, pay only for what they use, and scale globally within minutes.

Today, startups, enterprises, government organizations, and Fortune 500 companies use AWS for hosting applications, databases, machine learning, analytics, DevOps, IoT, and much more.

---

# What is Cloud Computing?

Cloud Computing is the on-demand delivery of computing resources such as servers, storage, networking, databases, software, and security over the internet with pay-as-you-go pricing.

Instead of purchasing and maintaining physical hardware, organizations can rent resources whenever needed.

---

# Traditional Infrastructure (On-Premises)

```text
Application

↓

Operating System

↓

Virtualization

↓

Servers

↓

Storage

↓

Networking

↓

Datacenter
```

The organization is responsible for purchasing, configuring, securing, maintaining, and replacing every component.

---

# Cloud Infrastructure

```text
Application

↓

Operating System

↓

AWS Services

↓

AWS Global Infrastructure
```

AWS manages the physical infrastructure while customers consume cloud resources on demand.

---

# Why Cloud Computing?

Cloud computing solves many limitations of traditional data centers.

Benefits include:

- No upfront hardware investment
- Pay only for resources used
- High availability
- Global deployment
- Elastic scalability
- Faster application delivery
- Improved disaster recovery
- Better security
- Automated infrastructure
- Reduced operational overhead

---

# Characteristics of Cloud Computing

AWS cloud provides the following characteristics:

- On-demand self-service
- Broad network access
- Resource pooling
- Rapid elasticity
- Measured service
- High availability
- Fault tolerance
- Global accessibility

---

# Why AWS?

AWS is the most widely adopted cloud platform because it offers:

- Largest global infrastructure
- Highly secure services
- Mature ecosystem
- Excellent DevOps integration
- Massive service portfolio
- Continuous innovation
- Enterprise-grade reliability
- Strong compliance support
- Extensive documentation
- Large community support

---

# Major AWS Categories

AWS services are grouped into multiple categories.

```text
Compute

Storage

Networking

Database

Security

Containers

Monitoring

Analytics

AI & ML

Developer Tools

Management

Migration

Serverless

Integration

IoT
```

---

# AWS Global Infrastructure

AWS operates one of the largest cloud infrastructures in the world.

It consists of:

- Regions
- Availability Zones
- Edge Locations
- Local Zones
- Wavelength Zones

---

# AWS Global Infrastructure Diagram

```text
AWS Global Network

├── Region

│      ├── Availability Zone A

│      ├── Availability Zone B

│      └── Availability Zone C

│
├── Region

│      ├── AZ A

│      ├── AZ B

│      └── AZ C

│
└── Edge Locations
```

---

# AWS Region

A Region is a geographical area where AWS hosts multiple data centers.

Examples:

- Mumbai
- Hyderabad (Local Zone)
- Singapore
- Frankfurt
- Tokyo
- Ohio
- Virginia
- Sydney

Each Region operates independently.

---

# Availability Zone (AZ)

An Availability Zone is one or more physically separate data centers within a Region.

Characteristics:

- Independent power supply
- Independent networking
- Independent cooling
- High-speed private connections between AZs

Applications should always be deployed across multiple Availability Zones for high availability.

---

# Region vs Availability Zone

| Region | Availability Zone |
|---------|-------------------|
| Geographic location | Physical data center |
| Contains multiple AZs | Exists inside a Region |
| Independent from other Regions | Connected using low-latency network |
| Used for disaster recovery | Used for high availability |

---

# Edge Location

Edge Locations are used for content delivery and low-latency services.

Common services:

- CloudFront
- Route53
- AWS Shield
- AWS WAF

They cache content closer to users.

---

# Local Zone

Local Zones extend AWS infrastructure closer to metropolitan areas for applications requiring ultra-low latency.

Example:

Gaming

Media Rendering

Video Editing

Machine Learning

---

# Wavelength Zone

Wavelength Zones bring AWS infrastructure into 5G networks.

Used for:

- Autonomous vehicles
- Smart manufacturing
- AR/VR
- IoT
- Real-time analytics

---

# AWS Backbone Network

AWS Regions are connected through AWS's private global fiber network.

Benefits:

- Low latency
- High bandwidth
- Secure communication
- Reliable connectivity

Customer traffic does not traverse the public internet between AWS Regions.

---

# AWS Console

The AWS Management Console is a web-based interface used to manage AWS resources.

Capabilities:

- Launch EC2 instances
- Create VPCs
- Manage IAM users
- Configure S3 buckets
- Monitor CloudWatch
- Deploy EKS clusters

---

# AWS CLI

AWS Command Line Interface allows engineers to manage AWS resources using commands.

Example:

```bash
aws ec2 describe-instances

aws s3 ls

aws eks list-clusters
```

Benefits:

- Automation
- Scripting
- CI/CD integration
- Infrastructure management

---

# AWS SDK

AWS SDK enables developers to interact with AWS services programmatically.

Supported languages include:

- Python (Boto3)
- Java
- Go
- Node.js
- .NET
- C++
- PHP
- Ruby

---

# Production Example

A microservices application deployed on AWS may use:

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Amazon EKS

↓

Microservices

↓

Amazon RDS

↓

Amazon ElastiCache

↓

Amazon S3

↓

CloudWatch
```

---

# Best Practices

- Use multiple Availability Zones.
- Design applications for high availability.
- Automate infrastructure using Terraform or CloudFormation.
- Enable monitoring with CloudWatch.
- Follow the Principle of Least Privilege.
- Use tagging for all resources.
- Regularly review costs using AWS Cost Explorer.
- Secure accounts with MFA.

---

# Common Mistakes

- Deploying workloads in a single AZ.
- Using the root account for daily operations.
- Not enabling MFA.
- Ignoring cost optimization.
- Hardcoding AWS credentials.
- Not backing up critical resources.
- Creating overly permissive IAM policies.
- Failing to monitor infrastructure.

---

# Troubleshooting Checklist

When an AWS service has issues, verify:

- Region selection
- IAM permissions
- Service quotas
- Security Groups
- Network ACLs
- Route Tables
- CloudWatch metrics
- CloudTrail logs
- AWS Health Dashboard

---

# Interview Questions

1. What is Cloud Computing?
2. Why AWS over other cloud providers?
3. What is an AWS Region?
4. What is an Availability Zone?
5. Difference between Region and AZ?
6. What are Edge Locations?
7. What is a Local Zone?
8. What is a Wavelength Zone?
9. What is the AWS Global Backbone Network?
10. What are the advantages of AWS?

---

# Key Takeaways

- AWS is the leading cloud platform with 250+ managed services.
- Regions contain multiple independent Availability Zones.
- High availability is achieved by deploying across multiple AZs.
- Edge Locations improve global content delivery.
- AWS CLI and SDK enable automation.
- Always follow security, scalability, and cost optimization best practices.
- Understanding AWS global infrastructure is the foundation for learning all AWS services.