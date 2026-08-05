# AWS Enterprise Security

# Chapter 1 - Enterprise Security Fundamentals & AWS Shared Responsibility Model

Security is the foundation of every AWS architecture.

Whether you're deploying

- EC2
- Kubernetes (EKS)
- ECS/Fargate
- Lambda
- RDS
- S3
- OpenSearch

security must be considered from the very beginning—not added later.

AWS follows a **Defense in Depth** approach, where multiple layers of security work together to protect applications, infrastructure, identities, and data.

---

# What is Enterprise Security?

Enterprise Security is the practice of protecting

- People
- Applications
- Infrastructure
- Networks
- Data
- Identities
- Cloud Resources

from

- Unauthorized Access
- Data Breaches
- Malware
- Insider Threats
- Misconfigurations
- Cyber Attacks

Security is a continuous process rather than a one-time configuration.

---

# Security Goals

Enterprise security is built around three primary goals.

```text
Confidentiality

↓

Integrity

↓

Availability
```

These three principles are known as the **CIA Triad**.

---

# CIA Triad

```text
          Security

             │

    ┌────────┼────────┐

Confidentiality Integrity Availability
```

Every security control supports one or more of these objectives.

---

# Confidentiality

Confidentiality ensures that

only authorized users can access data.

Examples

- IAM Policies
- Encryption
- Multi-Factor Authentication
- Secrets Manager

Example

```text
Employee

↓

IAM Authentication

↓

Sensitive Database
```

Unauthorized users are denied access.

---

# Integrity

Integrity ensures that

data cannot be modified without authorization.

Examples

- Hashing
- Digital Signatures
- Versioning
- Audit Logs

Example

```text
Original File

↓

SHA-256 Hash

↓

Verification

↓

File Unchanged
```

---

# Availability

Availability ensures systems remain operational.

AWS achieves availability using

- Multi-AZ
- Auto Scaling
- Load Balancers
- Disaster Recovery
- Monitoring

Example

```text
Users

↓

ALB

↓

Multiple EC2

↓

Application Available
```

---

# Security Principles

AWS recommends following these principles.

- Least Privilege
- Defense in Depth
- Zero Trust
- Automation
- Continuous Monitoring
- Encryption Everywhere

---

# Defense in Depth

Instead of relying on one security control,

multiple layers protect workloads.

```text
Users

↓

IAM

↓

MFA

↓

VPC

↓

Security Groups

↓

Application

↓

Database

↓

Encryption
```

Even if one layer is compromised,

others continue protecting resources.

---

# Zero Trust

Zero Trust assumes

```text
Never Trust

Always Verify
```

Every request is authenticated,

authorized,

and validated.

Location alone does not grant trust.

---

# AWS Shared Responsibility Model

One of the most important AWS interview topics.

AWS and customers share security responsibilities.

```text
AWS

↓

Security OF the Cloud

────────────────────────

Customer

↓

Security IN the Cloud
```

---

# AWS Responsibilities

AWS manages

- Physical Data Centers
- Hardware
- Networking Infrastructure
- Global Backbone
- Hypervisor
- Power
- Cooling
- Physical Security

Customers never manage these components.

---

# Customer Responsibilities

Customers manage

- IAM
- Security Groups
- Applications
- Operating System (EC2)
- Data
- Encryption
- Network Configuration
- Patching (EC2)
- Container Images

The customer remains responsible for workload security.

---

# Example - Amazon EC2

AWS manages

```text
Physical Server

↓

Hypervisor

↓

Networking
```

Customer manages

```text
Operating System

↓

Applications

↓

Users

↓

Data
```

---

# Example - Amazon RDS

AWS manages

- Database Software Installation
- Operating System
- Infrastructure
- Backups (if enabled)

Customer manages

- Database Users
- Database Permissions
- Encryption Choices
- Data

---

# Example - AWS Fargate

AWS manages

- Servers
- Container Runtime
- Host Operating System
- Infrastructure

Customer manages

- Container Image
- Application
- IAM Roles
- Secrets
- Networking Configuration

---

# Example - Amazon S3

AWS manages

- Storage Infrastructure
- Hardware
- Availability

Customer manages

- Bucket Policies
- IAM
- Object Permissions
- Encryption
- Public Access Settings

Misconfigured permissions remain the customer's responsibility.

---

# Security Domains

Enterprise AWS security consists of multiple domains.

```text
Identity Security

↓

Network Security

↓

Application Security

↓

Data Security

↓

Infrastructure Security

↓

Monitoring

↓

Compliance
```

Each domain contributes to overall security.

---

# Security Lifecycle

Security is continuous.

```text
Plan

↓

Build

↓

Deploy

↓

Monitor

↓

Audit

↓

Improve
```

There is no final "secure" state.

---

# Security by Design

Security should be integrated into architecture from the beginning.

Instead of

```text
Application

↓

Deploy

↓

Add Security
```

Use

```text
Design

↓

Security

↓

Deployment
```

Security becomes part of the architecture.

---

# Enterprise Example

A financial organization deploys

```text
Users

↓

IAM + MFA

↓

Application Load Balancer

↓

Private Subnet

↓

Amazon EKS

↓

Amazon RDS

↓

AWS KMS Encryption

↓

CloudTrail Logging

↓

CloudWatch Monitoring
```

Multiple security controls protect every layer.

---

# Benefits

- Reduced Attack Surface
- Better Compliance
- Stronger Identity Protection
- Secure Data
- Continuous Monitoring
- Business Continuity

---

# Best Practices

- Follow the Shared Responsibility Model.
- Implement least privilege.
- Enable Multi-Factor Authentication.
- Encrypt sensitive data.
- Automate security checks.
- Continuously monitor resources.
- Design security from the beginning.
- Regularly review permissions.

---

# Common Mistakes

- Assuming AWS secures customer applications.
- Granting AdministratorAccess to all users.
- Ignoring encryption.
- Storing secrets in source code.
- Disabling logging.
- Making S3 buckets public accidentally.
- Treating security as a one-time task.

---

# Interview Questions

## Basic

- What is the AWS Shared Responsibility Model?
- What is the CIA Triad?
- What is Defense in Depth?

## Intermediate

- AWS responsibility vs Customer responsibility.
- Explain Zero Trust Architecture.
- Why is least privilege important?

## Advanced

- Design a secure AWS architecture for an enterprise banking application.
- Explain how the Shared Responsibility Model changes when using EC2, RDS, and Fargate.
- Design a multi-layer security architecture using IAM, VPC, Security Groups, KMS, CloudTrail, CloudWatch, and Multi-AZ deployment.

---

