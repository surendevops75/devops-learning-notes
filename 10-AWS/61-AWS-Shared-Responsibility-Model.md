# AWS Shared Responsibility Model

---

# Introduction

The AWS Shared Responsibility Model defines the security responsibilities shared between AWS and the customer. AWS is responsible for securing the underlying cloud infrastructure, while customers are responsible for securing everything they deploy and configure within the cloud.

Understanding this model is critical because many security incidents occur due to customer misconfigurations rather than failures of AWS infrastructure.

The Shared Responsibility Model applies to every AWS service, although the customer’s responsibilities vary depending on whether the service is Infrastructure as a Service (IaaS), Platform as a Service (PaaS), or Software as a Service (SaaS).

---

# What is the AWS Shared Responsibility Model?

The Shared Responsibility Model divides security responsibilities between AWS and customers.

Workflow

```text
AWS Cloud

↓

AWS Responsibilities

+

Customer Responsibilities

↓

Secure Workload
```

---

# Why is it Important?

Without understanding the model

```text
Customer

↓

Assumes AWS Secures Everything

↓

Misconfigured Resources

↓

Security Incident
```

With proper understanding

```text
AWS Responsibilities

+

Customer Responsibilities

↓

Secure Infrastructure
```

---

# AWS Responsibility

AWS is responsible for **Security OF the Cloud**.

AWS manages

- Physical Data Centers
- Networking Infrastructure
- Hardware
- Storage Devices
- Hypervisor
- Global Infrastructure
- Availability Zones
- AWS Managed Services Infrastructure

Architecture

```text
Physical Security

↓

Servers

↓

Networking

↓

Hypervisor

↓

AWS Services
```

---

# Customer Responsibility

Customers are responsible for **Security IN the Cloud**.

Customers manage

- IAM Users and Roles
- Data
- Encryption Keys
- Operating Systems (EC2)
- Guest OS Patching
- Security Groups
- Network ACLs
- Application Security
- Database Configuration
- Backups
- Monitoring
- Logging

---

# Shared Responsibility Diagram

```text
                AWS

--------------------------------

Data Centers

Networking

Hardware

Hypervisor

Global Infrastructure

--------------------------------

             Customer

IAM

Applications

Operating System

Firewall Rules

Data

Encryption

Monitoring

Backups
```

---

# Responsibility by Service Type

| Service Type | AWS Responsibility | Customer Responsibility |
|--------------|-------------------|--------------------------|
| EC2 | Infrastructure | OS, Applications, Data |
| RDS | Database Engine Infrastructure | Database Users, Data |
| Lambda | Runtime Infrastructure | Function Code |
| S3 | Storage Infrastructure | Bucket Policies, Objects |
| DynamoDB | Service Infrastructure | Data, IAM Policies |

---

# Amazon EC2 Responsibilities

AWS manages

- Physical Servers
- Hypervisor
- Networking
- Data Center

Customer manages

- Guest Operating System
- OS Updates
- Applications
- Security Groups
- EBS Encryption
- IAM Roles
- SSH Access

---

# Amazon RDS Responsibilities

AWS manages

- Database Software Installation
- OS Patching
- Database Engine Updates
- High Availability Infrastructure
- Automated Failover

Customer manages

- Database Users
- Database Permissions
- Data
- Parameter Configuration
- Backup Retention Settings

---

# Amazon S3 Responsibilities

AWS manages

- Storage Infrastructure
- Disk Hardware
- Availability
- Durability

Customer manages

- Bucket Policies
- IAM Permissions
- Object Encryption
- Versioning
- Lifecycle Policies
- Public Access Settings

---

# AWS Lambda Responsibilities

AWS manages

- Runtime
- Operating System
- Infrastructure
- Scaling

Customer manages

- Function Code
- IAM Execution Role
- Environment Variables
- Secrets
- Dependencies

---

# IAM Responsibilities

AWS provides

- IAM Service
- Authentication Infrastructure

Customer manages

- Users
- Groups
- Roles
- Policies
- MFA
- Password Policies

---

# Encryption Responsibilities

AWS provides

- AWS KMS
- Encryption Services
- Hardware Security Modules (HSMs)

Customer manages

- Encryption Keys
- Key Rotation Policies
- Data Encryption Choices
- Access Control

---

# Logging Responsibilities

AWS provides

- CloudTrail Service
- CloudWatch Infrastructure

Customer manages

- Enable Logging
- Retention Policies
- Alarm Configuration
- Log Analysis

---

# AWS Services and Responsibility

| Service | AWS Manages | Customer Manages |
|----------|-------------|------------------|
| EC2 | Infrastructure | OS & Applications |
| RDS | Database Platform | Database Configuration |
| S3 | Storage Platform | Data Access |
| Lambda | Runtime | Code |
| EKS | Control Plane | Worker Nodes & Applications |
| ECS | Cluster Infrastructure | Containers |
| DynamoDB | Database Infrastructure | Data & Access |
| CloudFront | CDN Infrastructure | Distribution Configuration |

---

# Common Security Misconfigurations

Examples

- Public S3 Buckets
- Open Security Groups
- Weak IAM Policies
- Disabled MFA
- Unencrypted Storage
- Hardcoded Secrets
- Missing Logging
- Public Databases

Most are customer responsibilities.

---

# AWS Security Services Supporting the Model

Examples

- AWS IAM
- AWS KMS
- AWS CloudTrail
- AWS Config
- AWS GuardDuty
- AWS Security Hub
- AWS Shield
- AWS WAF
- Amazon Inspector
- AWS Macie

These services help customers fulfill their security responsibilities.

---

# AWS CLI

List IAM Users

```bash
aws iam list-users
```

Describe Security Groups

```bash
aws ec2 describe-security-groups
```

List S3 Buckets

```bash
aws s3 ls
```

---

# Terraform

Example

```hcl
resource "aws_security_group" "web" {

  name = "web-sg"

}
```

Terraform helps customers automate their security configurations.

---

# CloudFormation

```yaml
Resources:

  SecurityGroup:

    Type: AWS::EC2::SecurityGroup
```

---

# Python (Boto3)

```python
import boto3

iam = boto3.client("iam")

response = iam.list_users()

print(response["Users"])
```

---

# Enterprise Production Architecture

```text
             AWS

Data Centers • Hardware • Network

             │

             ▼

      AWS Cloud Services

             │

             ▼

         Customer

IAM • EC2 • Applications

Encryption • Monitoring

Logging • Security Groups
```

---

# Best Practices

- Understand Security OF vs Security IN the Cloud
- Enable MFA for all privileged users
- Follow least-privilege IAM policies
- Encrypt data at rest and in transit
- Enable CloudTrail and CloudWatch
- Regularly patch EC2 instances
- Monitor AWS Config compliance
- Use Security Hub and GuardDuty
- Rotate credentials regularly
- Review IAM permissions periodically
- Avoid public S3 buckets unless required
- Document security responsibilities

---

# Common Mistakes

- Assuming AWS patches EC2 operating systems
- Leaving S3 buckets public
- Weak IAM permissions
- Not enabling MFA
- Ignoring CloudTrail
- Storing secrets in code
- Not patching guest operating systems
- Overly permissive security groups
- Ignoring encryption
- Poor monitoring

---

# Troubleshooting

## EC2 Compromised

Check

- OS Patches
- IAM Roles
- Security Groups
- CloudTrail Logs

---

## Public S3 Bucket

Verify

- Bucket Policy
- Block Public Access
- IAM Permissions
- ACLs

---

## Database Exposed

Check

- Security Groups
- RDS Public Access
- IAM Authentication
- Network ACLs

---

## IAM Access Issues

Verify

- IAM Policies
- Role Trust Policy
- Permission Boundaries
- SCPs

---

## Missing Logs

Check

- CloudTrail Enabled
- CloudWatch Configuration
- IAM Permissions
- Log Retention

---

# Interview Questions

## Basic

1. What is the AWS Shared Responsibility Model?
2. What is Security OF the Cloud?
3. What is Security IN the Cloud?
4. Who patches the EC2 operating system?
5. Who secures the AWS data center?
6. Who manages S3 bucket policies?
7. Who manages IAM users?
8. Who configures Security Groups?
9. Who manages Lambda function code?
10. Why is the Shared Responsibility Model important?

---

## Intermediate

11. Explain EC2 responsibilities.
12. Explain RDS responsibilities.
13. Explain S3 responsibilities.
14. Explain Lambda responsibilities.
15. Explain IAM responsibilities.
16. Explain encryption responsibilities.
17. Explain monitoring responsibilities.
18. Explain customer security controls.
19. Explain AWS security services.
20. Explain common customer mistakes.

---

## Advanced

21. Design enterprise security using the Shared Responsibility Model.
22. Explain responsibility differences between EC2 and Lambda.
23. Design secure multi-account governance.
24. Explain customer responsibilities for Kubernetes on EKS.
25. Explain how AWS Security Hub supports customer responsibilities.
26. Design operational security processes.
27. Explain compliance under the Shared Responsibility Model.
28. Design enterprise IAM governance.
29. Explain security best practices.
30. Best practices for implementing the AWS Shared Responsibility Model.

---

# Production Scenarios

### Scenario 1

An EC2 instance is compromised because it was not patched for several months.

Who is responsible, AWS or the customer?

---

### Scenario 2

A production S3 bucket accidentally becomes publicly accessible.

Which configuration should be reviewed first?

---

### Scenario 3

A company enables Amazon RDS but creates weak database passwords.

Which responsibilities belong to AWS, and which belong to the customer?

---

### Scenario 4

An application running on AWS Lambda exposes sensitive data because secrets are hardcoded.

How does the Shared Responsibility Model apply?

---

### Scenario 5

An auditor requests evidence of encryption, IAM controls, and logging.

Which responsibilities must the customer demonstrate?

---

### Scenario 6

A company migrates to Amazon EKS.

Which security responsibilities remain with AWS, and which must the organization manage?

---

# Cheat Sheet

| AWS Manages | Customer Manages |
|-------------|------------------|
| Data Centers | IAM |
| Hardware | Applications |
| Networking | Data |
| Hypervisor | OS (EC2) |
| Managed Service Infrastructure | Security Groups |
| Global Infrastructure | Encryption |
| Availability | Monitoring |
| Physical Security | Logging Configuration |

---

# Summary

The AWS Shared Responsibility Model defines how security responsibilities are divided between AWS and its customers. AWS is responsible for **Security OF the Cloud**, including physical infrastructure, networking, and managed service infrastructure, while customers are responsible for **Security IN the Cloud**, including IAM, operating systems (where applicable), applications, data, encryption, monitoring, and access controls. Understanding this model is essential for building secure, compliant, and resilient AWS environments.