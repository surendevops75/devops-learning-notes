# AWS Interview Questions

---

# Introduction

AWS interviews evaluate both theoretical knowledge and practical production experience. Interviewers expect candidates to explain concepts, troubleshoot issues, design architectures, and justify design decisions.

This guide is organized into

- Basic Questions
- Intermediate Questions
- Advanced Questions
- Production Scenarios
- Architecture Questions
- Troubleshooting Questions

---

# AWS Fundamentals

---

## Question 1

### What is AWS?

**Answer**

Amazon Web Services (AWS) is a cloud computing platform that provides on-demand services such as compute, storage, databases, networking, security, analytics, AI/ML, and DevOps tools. It enables organizations to build scalable, secure, and highly available applications using a pay-as-you-go pricing model.

---

## Question 2

### What are the advantages of AWS?

**Answer**

- Pay-as-you-go pricing
- Global infrastructure
- High availability
- Elastic scalability
- Security and compliance
- Managed services
- Disaster recovery capabilities
- Automation support

---

## Question 3

### Explain the AWS Global Infrastructure.

**Answer**

AWS Global Infrastructure consists of

- Regions
- Availability Zones (AZs)
- Local Zones
- Edge Locations

Example

```text
Region

↓

Availability Zones

↓

VPC

↓

Resources
```

---

## Question 4

### What is an AWS Region?

**Answer**

A Region is a geographical location containing multiple isolated Availability Zones. Resources deployed in different Regions are physically separated.

Examples

- ap-south-1
- us-east-1
- eu-west-1

---

## Question 5

### What is an Availability Zone?

**Answer**

An Availability Zone is an isolated data center (or group of data centers) within an AWS Region.

Benefits

- High Availability
- Fault Isolation
- Disaster Recovery

---

## Question 6

### Difference between Region and Availability Zone?

| Region | Availability Zone |
|---------|-------------------|
| Geographic Location | Data Center |
| Multiple AZs | Inside Region |
| Disaster Isolation | High Availability |

---

# EC2

---

## Question 7

### What is Amazon EC2?

**Answer**

Amazon EC2 (Elastic Compute Cloud) is a virtual machine service that provides scalable compute capacity in the cloud.

Features

- Virtual Servers
- Auto Scaling
- Multiple Instance Types
- Security Groups
- Elastic IPs

---

## Question 8

### What are EC2 Instance Types?

**Answer**

Examples

- General Purpose (T, M)
- Compute Optimized (C)
- Memory Optimized (R, X)
- Storage Optimized (I, D)
- Accelerated Computing (P, G)

---

## Question 9

### Difference between Security Groups and NACLs?

| Security Group | Network ACL |
|----------------|------------|
| Stateful | Stateless |
| Instance Level | Subnet Level |
| Allow Rules Only | Allow & Deny Rules |

---

## Question 10

### What is an AMI?

**Answer**

An Amazon Machine Image (AMI) is a template containing

- Operating System
- Software
- Configuration
- Application Packages

Used to launch EC2 instances.

---

## Question 11

### What is an Elastic IP?

**Answer**

An Elastic IP is a static public IPv4 address that can be associated with an EC2 instance and remapped to another instance if required.

---

## Question 12

### Difference between EBS and Instance Store?

| EBS | Instance Store |
|-----|---------------|
| Persistent | Temporary |
| Network Attached | Local Storage |
| Supports Snapshots | No Snapshots |

---

# Auto Scaling

---

## Question 13

### What is Auto Scaling?

**Answer**

Auto Scaling automatically adjusts the number of EC2 instances based on demand.

Benefits

- High Availability
- Cost Optimization
- Automatic Scaling

---

## Question 14

### What are the components of an Auto Scaling Group?

**Answer**

- Launch Template
- Desired Capacity
- Minimum Capacity
- Maximum Capacity
- Scaling Policies

---

# Elastic Load Balancer

---

## Question 15

### What are the types of Elastic Load Balancers?

**Answer**

- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- Gateway Load Balancer (GWLB)
- Classic Load Balancer (Legacy)

---

## Question 16

### Difference between ALB and NLB?

| ALB | NLB |
|-----|-----|
| Layer 7 | Layer 4 |
| HTTP/HTTPS | TCP/UDP/TLS |
| Path Routing | High Performance |

---

# VPC

---

## Question 17

### What is Amazon VPC?

**Answer**

Amazon VPC (Virtual Private Cloud) provides an isolated virtual network where AWS resources can be securely deployed.

---

## Question 18

### Difference between Public and Private Subnets?

| Public | Private |
|---------|----------|
| Internet Access | No Direct Internet |
| Uses Internet Gateway | Uses NAT Gateway |

---

## Question 19

### What is a NAT Gateway?

**Answer**

A NAT Gateway enables instances in private subnets to access the internet without exposing them to inbound internet traffic.

---

## Question 20

### What is an Internet Gateway?

**Answer**

An Internet Gateway enables communication between resources in a VPC and the public internet.

---

# S3

---

## Question 21

### What is Amazon S3?

**Answer**

Amazon S3 is an object storage service used for storing and retrieving unlimited amounts of data.

---

## Question 22

### What are S3 Storage Classes?

**Answer**

- Standard
- Intelligent-Tiering
- Standard-IA
- One Zone-IA
- Glacier Instant Retrieval
- Glacier Flexible Retrieval
- Glacier Deep Archive

---

## Question 23

### What is Versioning?

**Answer**

Versioning preserves multiple versions of an object, protecting against accidental deletion and overwrites.

---

## Question 24

### What is Lifecycle Management?

**Answer**

Lifecycle policies automatically transition or delete objects based on predefined rules.

---

# IAM

---

## Question 25

### What is IAM?

**Answer**

AWS Identity and Access Management (IAM) enables secure authentication and authorization for AWS resources.

---

## Question 26

### Difference between IAM User, Group and Role?

| User | Group | Role |
|------|-------|------|
| Individual Identity | Collection of Users | Temporary Identity |

---

## Question 27

### What is the Principle of Least Privilege?

**Answer**

Grant only the permissions required to perform a task and nothing more.

---

## Question 28

### What is an IAM Policy?

**Answer**

An IAM Policy is a JSON document defining permissions for AWS resources.

---

# RDS

---

## Question 29

### What is Amazon RDS?

**Answer**

Amazon RDS is a managed relational database service supporting engines such as MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

---

## Question 30

### What is Multi-AZ?

**Answer**

Multi-AZ deploys a standby database in another Availability Zone to improve high availability and automatic failover.

---

# Basic Rapid Fire

31. What is AWS?
32. What is EC2?
33. What is S3?
34. What is IAM?
35. What is VPC?
36. What is Auto Scaling?
37. What is ALB?
38. What is Route53?
39. What is CloudFront?
40. What is RDS?
41. What is EBS?
42. What is EFS?
43. What is Lambda?
44. What is DynamoDB?
45. What is CloudWatch?
46. What is CloudTrail?
47. What is SNS?
48. What is SQS?
49. What is KMS?
50. What is Secrets Manager?

---

# Interview Tips

- Always explain the service first.
- Mention at least one real-world use case.
- Explain advantages and limitations.
- If applicable, compare it with similar AWS services.
- Use architecture diagrams or production examples to strengthen your answer.

---

# Summary

This section covered foundational AWS interview questions related to Global Infrastructure, EC2, VPC, IAM, S3, RDS, Auto Scaling, and Elastic Load Balancing. These concepts form the basis of most AWS interviews and are commonly asked in entry-level to intermediate DevOps and Cloud Engineer roles.