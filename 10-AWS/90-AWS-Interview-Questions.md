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

---

# VPC & Networking

---

## Question 51

### Explain the components of a VPC.

**Answer**

A VPC consists of

- CIDR Block
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- Elastic IPs

Architecture

```text
VPC

↓

Subnets

↓

Route Tables

↓

Internet / NAT Gateway

↓

EC2
```

---

## Question 52

### What is CIDR?

**Answer**

CIDR (Classless Inter-Domain Routing) defines the IP address range for a VPC or subnet.

Example

```
10.0.0.0/16
```

Supports

65,536 IP addresses.

---

## Question 53

### Difference between Internet Gateway and NAT Gateway?

| Internet Gateway | NAT Gateway |
|------------------|-------------|
| Public Subnet | Private Subnet |
| Inbound + Outbound | Outbound Only |
| Internet Access | Secure Internet Access |

---

## Question 54

### What is VPC Peering?

**Answer**

VPC Peering connects two VPCs privately using AWS's internal network.

Characteristics

- Private Communication
- No Internet
- Same or Different Regions

---

## Question 55

### What is AWS Transit Gateway?

**Answer**

Transit Gateway acts as a central networking hub connecting

- VPCs
- VPNs
- Direct Connect

Architecture

```text
Transit Gateway

↓

VPC A

VPC B

VPN

Direct Connect
```

---

## Question 56

### What is AWS Direct Connect?

**Answer**

AWS Direct Connect provides a dedicated private network connection between an organization's data center and AWS.

Benefits

- Lower Latency
- Consistent Performance
- Private Connectivity

---

## Question 57

### Difference between Direct Connect and VPN?

| Direct Connect | VPN |
|---------------|-----|
| Dedicated Connection | Internet-Based |
| Higher Bandwidth | Lower Bandwidth |
| Lower Latency | Higher Latency |
| Higher Cost | Lower Cost |

---

## Question 58

### What is Site-to-Site VPN?

**Answer**

Site-to-Site VPN securely connects an on-premises network to an AWS VPC using IPSec tunnels over the internet.

---

## Question 59

### What is AWS Client VPN?

**Answer**

AWS Client VPN allows individual users to securely connect to AWS resources using an encrypted VPN connection.

---

## Question 60

### Explain Route Tables.

**Answer**

Route Tables determine where network traffic is routed.

Example

```
Destination

↓

Target

↓

Internet Gateway

NAT Gateway

VPC Peering
```

---

# Route 53

---

## Question 61

### What is Amazon Route 53?

**Answer**

Amazon Route 53 is a scalable DNS and domain registration service.

Capabilities

- Domain Registration
- DNS Management
- Health Checks
- Traffic Routing

---

## Question 62

### What Routing Policies does Route 53 support?

**Answer**

- Simple
- Weighted
- Latency
- Failover
- Geolocation
- Geoproximity
- Multi-Value Answer

---

## Question 63

### What is a Health Check?

**Answer**

A Health Check monitors endpoint availability and automatically redirects traffic if an endpoint becomes unhealthy.

---

## Question 64

### Explain Failover Routing.

**Answer**

Failover Routing directs traffic to a secondary endpoint when the primary endpoint fails health checks.

---

# CloudFront

---

## Question 65

### What is Amazon CloudFront?

**Answer**

Amazon CloudFront is a Content Delivery Network (CDN) that delivers content from Edge Locations close to users.

Benefits

- Lower Latency
- Faster Content Delivery
- DDoS Protection
- Global Reach

---

## Question 66

### What are Edge Locations?

**Answer**

Edge Locations cache content closer to users, reducing latency and improving performance.

---

## Question 67

### Difference between CloudFront and S3?

| CloudFront | Amazon S3 |
|------------|-----------|
| CDN | Object Storage |
| Content Caching | Data Storage |
| Edge Locations | Regional Storage |

---

# EBS

---

## Question 68

### What is Amazon EBS?

**Answer**

Amazon Elastic Block Store (EBS) provides persistent block-level storage for EC2 instances.

Features

- Persistent Storage
- Snapshots
- Encryption
- High Performance

---

## Question 69

### What EBS Volume Types are available?

**Answer**

- gp3
- gp2
- io2
- io1
- st1
- sc1

---

## Question 70

### What are EBS Snapshots?

**Answer**

Snapshots are incremental backups of EBS volumes stored in Amazon S3.

---

# EFS

---

## Question 71

### What is Amazon EFS?

**Answer**

Amazon Elastic File System (EFS) is a managed NFS-based shared file system.

Supports

- Multiple EC2 Instances
- Linux Workloads
- Auto Scaling

---

## Question 72

### Difference between EBS and EFS?

| EBS | EFS |
|-----|-----|
| Block Storage | File Storage |
| Single EC2 (typically) | Multiple EC2 Instances |
| High Performance | Shared Storage |

---

# FSx

---

## Question 73

### What is Amazon FSx?

**Answer**

Amazon FSx provides managed file systems for specialized workloads.

Examples

- FSx for Windows File Server
- FSx for Lustre
- FSx for NetApp ONTAP
- FSx for OpenZFS

---

## Question 74

### When would you choose FSx over EFS?

**Answer**

Use FSx when specialized file systems or Windows-native file sharing are required.

---

# Storage Gateway

---

## Question 75

### What is AWS Storage Gateway?

**Answer**

Storage Gateway connects on-premises storage with AWS cloud storage.

Gateway Types

- File Gateway
- Volume Gateway
- Tape Gateway

---

## Question 76

### What is AWS DataSync?

**Answer**

AWS DataSync securely transfers large amounts of data between on-premises storage and AWS storage services.

---

## Question 77

### What is AWS Snow Family?

**Answer**

AWS Snow Family devices transfer large datasets when network connectivity is limited or insufficient.

Services

- Snowcone
- Snowball Edge
- Snowmobile

---

# Global Networking

---

## Question 78

### What is AWS Global Accelerator?

**Answer**

AWS Global Accelerator improves application availability and performance using the AWS global network.

---

## Question 79

### Difference between Global Accelerator and CloudFront?

| Global Accelerator | CloudFront |
|-------------------|------------|
| TCP/UDP Applications | HTTP/HTTPS Content |
| Improves Network Path | Content Caching |
| Static Anycast IP | Edge Caching |

---

## Question 80

### What is AWS PrivateLink?

**Answer**

AWS PrivateLink enables private access to supported AWS services and applications without traversing the public internet.

---

# Rapid Fire

81. What is CIDR?
82. What is VPC Peering?
83. What is Transit Gateway?
84. What is Direct Connect?
85. What is Site-to-Site VPN?
86. What is Client VPN?
87. What is Route Table?
88. What is Internet Gateway?
89. What is NAT Gateway?
90. What is CloudFront?
91. What is Route 53?
92. What is EBS?
93. What is EFS?
94. What is FSx?
95. What is Storage Gateway?
96. What is DataSync?
97. What is Snow Family?
98. What is Global Accelerator?
99. What is AWS PrivateLink?
100. What is an Edge Location?

---

# Interview Tips

- Draw network architecture diagrams whenever possible.
- Clearly explain traffic flow through VPC components.
- Compare similar services (e.g., NAT Gateway vs Internet Gateway, Direct Connect vs VPN).
- Relate networking concepts to production scenarios.
- Mention security considerations such as Security Groups, NACLs, and private connectivity.

---

# Summary

This section covered AWS networking and storage interview topics including VPC, Route Tables, Internet Gateway, NAT Gateway, Route 53, CloudFront, EBS, EFS, FSx, Storage Gateway, DataSync, Snow Family, Global Accelerator, and PrivateLink. These services are fundamental for designing secure, scalable, and highly available AWS architectures.

---

# IAM (Identity and Access Management)

---

## Question 101

### What is IAM?

**Answer**

AWS Identity and Access Management (IAM) is a global AWS service used to securely manage authentication and authorization for AWS resources.

IAM enables you to create

- Users
- Groups
- Roles
- Policies

---

## Question 102

### Difference between IAM User, Group and Role?

**Answer**

| IAM User | IAM Group | IAM Role |
|----------|-----------|----------|
| Individual Identity | Collection of Users | Temporary Identity |
| Long-term Credentials | Permission Management | Assumed by Users/Services |
| Password & Access Keys | No Credentials | Temporary Credentials |

---

## Question 103

### What is an IAM Policy?

**Answer**

An IAM Policy is a JSON document that defines permissions.

Example

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"
}
```

---

## Question 104

### What are IAM Roles?

**Answer**

IAM Roles provide temporary credentials to AWS services or users.

Common Uses

- EC2
- Lambda
- ECS
- EKS
- Cross-Account Access

---

## Question 105

### What is the Principle of Least Privilege?

**Answer**

Grant only the minimum permissions required to perform a task.

---

## Question 106

### What is a Trust Policy?

**Answer**

A Trust Policy defines who or what can assume an IAM Role using `sts:AssumeRole`.

---

## Question 107

### What is Cross-Account Access?

**Answer**

Cross-account access allows users or services in one AWS account to access resources in another AWS account by assuming an IAM Role.

---

## Question 108

### What is an IAM Permission Boundary?

**Answer**

A Permission Boundary sets the maximum permissions an IAM User or Role can receive.

---

## Question 109

### What are Access Keys?

**Answer**

Access Keys consist of

- Access Key ID
- Secret Access Key

Used for programmatic access through AWS CLI and SDKs.

---

## Question 110

### What is AWS STS?

**Answer**

AWS Security Token Service (STS) provides temporary security credentials for IAM Roles and federated users.

---

# AWS KMS

---

## Question 111

### What is AWS KMS?

**Answer**

AWS Key Management Service (KMS) is a managed encryption service used to create and manage cryptographic keys.

---

## Question 112

### Difference between AWS Managed Key and Customer Managed Key (CMK)?

| AWS Managed Key | Customer Managed Key |
|-----------------|----------------------|
| Managed by AWS | Managed by Customer |
| Limited Control | Full Control |
| Automatic Creation | Manual Creation |

---

## Question 113

### What is Envelope Encryption?

**Answer**

Envelope Encryption encrypts data using a Data Key, while the Data Key itself is encrypted using a KMS Key.

---

## Question 114

### Difference between Data Key and CMK?

| Data Key | CMK |
|-----------|-----|
| Encrypts Data | Encrypts Data Keys |
| Temporary | Long-lived |

---

# Secrets Manager

---

## Question 115

### What is AWS Secrets Manager?

**Answer**

Secrets Manager securely stores and manages sensitive information such as

- Database Passwords
- API Keys
- Tokens
- Credentials

Supports automatic secret rotation.

---

## Question 116

### Difference between Secrets Manager and Parameter Store?

| Secrets Manager | Parameter Store |
|----------------|-----------------|
| Automatic Rotation | No Native Rotation |
| Designed for Secrets | General Configuration |
| Additional Cost | Standard Tier Available |

---

# CloudTrail

---

## Question 117

### What is AWS CloudTrail?

**Answer**

CloudTrail records AWS API activity for governance, auditing, and troubleshooting.

Captures

- Console Actions
- CLI Commands
- SDK Calls
- Service Events

---

## Question 118

### Difference between CloudTrail and CloudWatch?

| CloudTrail | CloudWatch |
|------------|------------|
| API Auditing | Monitoring & Metrics |
| Governance | Performance Monitoring |
| Event History | Logs & Alarms |

---

# GuardDuty

---

## Question 119

### What is Amazon GuardDuty?

**Answer**

Amazon GuardDuty is an intelligent threat detection service that continuously monitors AWS accounts for suspicious activity using machine learning and threat intelligence.

---

## Question 120

### What data sources does GuardDuty analyze?

**Answer**

- CloudTrail
- VPC Flow Logs
- DNS Logs
- EKS Audit Logs (optional)
- S3 Protection (optional)
- Malware Protection (supported integrations)

---

# Security Hub

---

## Question 121

### What is AWS Security Hub?

**Answer**

AWS Security Hub centralizes security findings from AWS services and partner tools into a single dashboard.

---

## Question 122

### Difference between GuardDuty and Security Hub?

| GuardDuty | Security Hub |
|------------|--------------|
| Threat Detection | Security Dashboard |
| Generates Findings | Aggregates Findings |
| Continuous Monitoring | Compliance Reporting |

---

# AWS Inspector

---

## Question 123

### What is Amazon Inspector?

**Answer**

Amazon Inspector automatically scans AWS workloads for software vulnerabilities and unintended network exposure.

---

## Question 124

### What resources can Inspector scan?

**Answer**

- EC2
- Amazon ECR Container Images
- AWS Lambda Functions

---

# AWS Macie

---

## Question 125

### What is Amazon Macie?

**Answer**

Amazon Macie uses machine learning to discover, classify, and protect sensitive data stored in Amazon S3.

---

## Question 126

### What type of information can Macie detect?

**Answer**

Examples

- Personally Identifiable Information (PII)
- Financial Data
- Credentials
- Personal Records

---

# AWS Detective

---

## Question 127

### What is Amazon Detective?

**Answer**

Amazon Detective helps investigate security incidents by analyzing relationships between AWS resources, users, and activities.

---

## Question 128

### Difference between GuardDuty and Detective?

| GuardDuty | Detective |
|------------|-----------|
| Detects Threats | Investigates Threats |
| Generates Findings | Root Cause Analysis |

---

# AWS WAF

---

## Question 129

### What is AWS WAF?

**Answer**

AWS Web Application Firewall (WAF) protects web applications from common web attacks.

Examples

- SQL Injection
- Cross-Site Scripting (XSS)
- IP Blocking
- Rate Limiting

---

## Question 130

### Where can AWS WAF be attached?

**Answer**

- Application Load Balancer
- Amazon CloudFront
- Amazon API Gateway
- AWS AppSync

---

# AWS Shield

---

## Question 131

### What is AWS Shield?

**Answer**

AWS Shield provides managed protection against Distributed Denial of Service (DDoS) attacks.

---

## Question 132

### Difference between Shield Standard and Shield Advanced?

| Shield Standard | Shield Advanced |
|-----------------|-----------------|
| Free | Paid |
| Basic DDoS Protection | Advanced DDoS Protection |
| Automatic | Additional Detection & Support |

---

# AWS Certificate Manager

---

## Question 133

### What is AWS Certificate Manager (ACM)?

**Answer**

AWS Certificate Manager provisions, manages, and renews SSL/TLS certificates for AWS resources.

---

## Question 134

### Which AWS services integrate with ACM?

**Answer**

- Application Load Balancer
- CloudFront
- API Gateway

---

# AWS Organizations

---

## Question 135

### What is AWS Organizations?

**Answer**

AWS Organizations enables centralized management of multiple AWS accounts.

---

## Question 136

### What are Organizational Units (OUs)?

**Answer**

Organizational Units (OUs) are logical containers used to organize AWS accounts within AWS Organizations.

---

## Question 137

### What are Service Control Policies (SCPs)?

**Answer**

SCPs define the maximum permissions available to AWS accounts within an AWS Organization.

---

# AWS Control Tower

---

## Question 138

### What is AWS Control Tower?

**Answer**

AWS Control Tower automates the setup and governance of secure multi-account AWS environments using AWS Organizations.

---

# IAM Identity Center

---

## Question 139

### What is IAM Identity Center?

**Answer**

IAM Identity Center (formerly AWS Single Sign-On) provides centralized identity management and single sign-on access to multiple AWS accounts and applications.

---

# AWS Directory Service

---

## Question 140

### What is AWS Directory Service?

**Answer**

AWS Directory Service enables AWS resources to use Microsoft Active Directory and other directory services for authentication and identity management.

---

# Rapid Fire

141. What is IAM?
142. What is an IAM Role?
143. What is STS?
144. What is a Trust Policy?
145. What is KMS?
146. What is Envelope Encryption?
147. What is Secrets Manager?
148. What is CloudTrail?
149. What is GuardDuty?
150. What is Security Hub?
151. What is Inspector?
152. What is Macie?
153. What is Detective?
154. What is AWS WAF?
155. What is AWS Shield?
156. What is ACM?
157. What is AWS Organizations?
158. What is an SCP?
159. What is Control Tower?
160. What is IAM Identity Center?

---

# Interview Tips

- Always explain the difference between authentication (who you are) and authorization (what you can do).
- Mention the Principle of Least Privilege whenever discussing IAM.
- Compare security services (e.g., GuardDuty vs Security Hub vs Inspector).
- Explain real-world use cases such as using Secrets Manager for database credentials or KMS for encrypting EBS volumes.
- For senior interviews, discuss multi-account governance with AWS Organizations and SCPs.

---

# Summary

This section covered AWS security and identity services including IAM, KMS, Secrets Manager, CloudTrail, GuardDuty, Security Hub, Inspector, Macie, Detective, WAF, Shield, ACM, AWS Organizations, Control Tower, IAM Identity Center, and Directory Service. These services form the foundation of securing AWS environments and are frequently discussed in DevOps, Cloud Engineer, and Solutions Architect interviews.

---

