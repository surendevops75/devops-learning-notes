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

# Chapter 2 - AWS Identity & Access Management (IAM) Deep Dive

Identity is the foundation of AWS security.

Before securing networks, applications, or data, AWS must answer two questions:

1. **Who is making the request?**
2. **What are they allowed to do?**

AWS Identity and Access Management (IAM) is the service responsible for authentication and authorization across AWS.

Every AWS service integrates with IAM.

---

# What is IAM?

AWS Identity and Access Management (IAM) is a global AWS service that controls

- Authentication
- Authorization
- Access Management
- Permissions
- Temporary Credentials

IAM allows you to securely control access to AWS resources.

---

# IAM Architecture

```text
User

↓

Authenticate

↓

IAM

↓

Authorization

↓

AWS Resources
```

Every request passes through IAM before reaching AWS services.

---

# Core IAM Components

IAM consists of

- Users
- Groups
- Roles
- Policies
- Identity Providers
- MFA
- Access Keys

Each component has a specific purpose.

---

# IAM Users

An IAM User represents a person or application requiring AWS access.

Example

```text
Developer

↓

IAM User

↓

AWS Console
```

Each user has unique credentials.

---

# IAM Groups

Groups simplify permission management.

Instead of assigning permissions individually,

users inherit permissions from their group.

Example

```text
Developers

↓

IAM Group

↓

Amazon EC2 Access
```

Adding a new developer automatically grants the required permissions.

---

# IAM Roles

IAM Roles provide temporary permissions.

Unlike users,

roles do not have permanent credentials.

Example

```text
EC2 Instance

↓

IAM Role

↓

Amazon S3
```

The EC2 instance receives temporary credentials automatically.

---

# IAM Policies

Policies define permissions.

Example

```text
Allow

↓

Amazon S3

↓

Read Objects
```

Policies answer

```text
What actions are allowed?
```

---

# Authentication vs Authorization

Authentication

```text
Who are you?
```

Authorization

```text
What can you do?
```

IAM performs both functions.

---

# IAM Policy Structure

A policy contains

- Effect
- Action
- Resource
- Condition

Example

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"
}
```

---

# IAM Policy Flow

```text
User

↓

IAM Policy

↓

Evaluation

↓

Allow

OR

Deny
```

Every AWS API request is evaluated against IAM policies.

---

# Types of IAM Policies

AWS supports

- Identity-Based Policies
- Resource-Based Policies
- Inline Policies
- Managed Policies

Each serves different use cases.

---

# AWS Managed Policies

AWS provides predefined policies.

Examples

- AmazonS3ReadOnlyAccess
- AmazonEC2ReadOnlyAccess
- AmazonRDSFullAccess

Advantages

- Easy to use
- Maintained by AWS
- Frequently updated

---

# Customer Managed Policies

Organizations create their own reusable policies.

Example

```text
Company

↓

Custom Policy

↓

Read Production Logs
```

These policies follow organizational requirements.

---

# Inline Policies

Inline policies are attached directly to

- One User
- One Group
- One Role

They cannot be reused.

Suitable for

special-case permissions.

---

# IAM Permission Evaluation

AWS evaluates permissions in the following order.

```text
Request

↓

Authentication

↓

Policy Evaluation

↓

Explicit Deny?

↓

Yes

↓

Access Denied

────────────

No

↓

Explicit Allow?

↓

Yes

↓

Access Granted
```

An Explicit Deny always overrides Allow.

---

# Principle of Least Privilege

One of AWS's most important security principles.

Users receive

only the permissions required to perform their jobs.

Example

Instead of

```text
AdministratorAccess
```

Use

```text
Read S3

Write CloudWatch Logs

Describe EC2
```

---

# Multi-Factor Authentication (MFA)

Passwords alone are insufficient.

Enable MFA.

Architecture

```text
User

↓

Password

↓

MFA Code

↓

AWS Console
```

Even if the password is stolen,

attackers cannot log in without the second factor.

---

# Access Keys

Applications may access AWS using

- Access Key ID
- Secret Access Key

Example

```text
Application

↓

Access Keys

↓

AWS API
```

AWS recommends replacing long-lived keys with IAM Roles whenever possible.

---

# Temporary Credentials

IAM Roles use temporary credentials issued by AWS STS.

Architecture

```text
Application

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

AWS Services
```

Benefits

- Automatic Rotation
- Improved Security
- No Hardcoded Secrets

---

# Cross-Account Access

Organizations commonly allow one AWS account to access another.

Architecture

```text
Account A

↓

AssumeRole

↓

AWS STS

↓

Temporary Credentials

↓

Account B
```

No permanent credential sharing is required.

---

# Identity Federation

Employees often log in using corporate identities.

Example

```text
Employee

↓

Azure AD

↓

IAM Identity Center

↓

AWS
```

Users avoid separate AWS passwords.

---

# IAM Best Practices

- Use IAM Roles instead of Access Keys.
- Enable MFA for all privileged users.
- Follow Least Privilege.
- Rotate credentials regularly.
- Use Groups instead of assigning permissions individually.
- Avoid using the root account.
- Review IAM permissions periodically.
- Enable CloudTrail auditing.

---

# Common Mistakes

- Sharing IAM Users.
- Hardcoding Access Keys.
- Granting AdministratorAccess unnecessarily.
- Disabling MFA.
- Using the Root User for daily work.
- Never reviewing permissions.
- Creating duplicate policies.

---

# Enterprise Example

A production environment

```text
Developer

↓

IAM Group

↓

Read-Only Production

──────────────

EC2

↓

IAM Role

↓

Amazon S3

──────────────

Lambda

↓

IAM Role

↓

DynamoDB

──────────────

CloudTrail

↓

Audit Logs
```

Every workload follows least privilege.

---

# Interview Questions

## Basic

- What is IAM?
- IAM User vs IAM Role.
- What is an IAM Policy?

## Intermediate

- IAM Role vs Access Keys.
- AWS Managed Policy vs Customer Managed Policy.
- Authentication vs Authorization.
- Explain the Principle of Least Privilege.

## Advanced

- Design IAM permissions for an enterprise with Developers, DevOps Engineers, Security Engineers, and Auditors.
- Explain how IAM Roles improve security compared to Access Keys.
- Design a cross-account AWS architecture where applications in one account securely access S3 buckets and DynamoDB tables in another account without sharing credentials.

---

