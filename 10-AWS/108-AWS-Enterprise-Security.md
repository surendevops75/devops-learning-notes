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

# Chapter 3 - IAM Roles, AWS STS & Cross-Account Access (Deep Dive)

As AWS environments grow,

organizations quickly realize that creating IAM users for every application, server, or AWS account is not scalable.

Instead, AWS recommends using **IAM Roles** with **AWS Security Token Service (STS)**.

This approach provides

- Temporary Credentials
- Better Security
- Automatic Credential Rotation
- Cross-Account Access
- Federated Authentication

IAM Roles are one of the most important AWS security concepts and are used extensively in enterprise environments.

---

# Why IAM Roles?

Suppose an EC2 instance needs to read objects from Amazon S3.

One approach is

```text
EC2

↓

Access Keys

↓

Amazon S3
```

Problems

- Keys stored on server
- Manual rotation
- Risk of credential leakage
- Operational overhead

Instead

```text
EC2

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

Amazon S3
```

No permanent credentials exist.

---

# What is an IAM Role?

An IAM Role is an AWS identity that

- Has permissions
- Has no permanent credentials
- Can be assumed temporarily
- Is designed for users, applications, or AWS services

Roles improve both security and manageability.

---

# IAM Role Architecture

```text
Application

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

AWS Service
```

The application never stores credentials.

---

# IAM Role Components

Every IAM Role contains

- Trust Policy
- Permission Policy
- Temporary Credentials
- Session Duration

These components determine

- Who can assume the role
- What permissions the role has
- How long access lasts

---

# Trust Policy

A Trust Policy defines

**Who can assume the role.**

Example

```text
EC2

↓

Assume Role

↓

IAM Role
```

If EC2 is trusted,

AWS allows the request.

---

# Permission Policy

A Permission Policy defines

**What the role can do.**

Example

```text
IAM Role

↓

Read Objects

↓

Amazon S3
```

Even after assuming the role,

permissions remain limited.

---

# AWS Security Token Service (STS)

AWS STS issues

temporary security credentials.

Architecture

```text
User

↓

AWS STS

↓

Temporary Credentials

↓

AWS Resources
```

Temporary credentials automatically expire.

---

# Benefits of STS

- Short-lived Credentials
- Automatic Rotation
- Reduced Risk
- Better Auditability
- No Credential Storage

---

# Temporary Credentials

STS generates

```text
Access Key

↓

Secret Key

↓

Session Token
```

These credentials remain valid only for a limited time.

---

# Credential Lifecycle

```text
Assume Role

↓

STS Generates Credentials

↓

Application Uses Credentials

↓

Credentials Expire

↓

New Credentials Generated
```

No manual rotation is required.

---

# EC2 with IAM Role

Example

```text
Amazon EC2

↓

IAM Role

↓

Amazon S3

↓

Read Objects
```

The application running on EC2 automatically receives temporary credentials.

---

# Lambda with IAM Role

Architecture

```text
Lambda Function

↓

IAM Role

↓

DynamoDB
```

Lambda securely accesses DynamoDB without Access Keys.

---

# ECS Task Role

Amazon ECS tasks receive permissions through IAM Roles.

```text
Amazon ECS Task

↓

IAM Task Role

↓

Amazon SQS

↓

Receive Messages
```

Each task receives its own credentials.

---

# EKS IAM Role

Kubernetes workloads use

IAM Roles for Service Accounts (IRSA).

Architecture

```text
Kubernetes Pod

↓

Service Account

↓

IAM Role

↓

Amazon S3
```

Each Pod gets only the permissions it requires.

---

# Fargate Task Role

Amazon Fargate uses Task Roles.

```text
Fargate Task

↓

IAM Role

↓

Amazon Secrets Manager
```

Applications securely retrieve secrets.

---

# Cross-Account Access

Large organizations often use multiple AWS accounts.

Example

```text
Development Account

↓

Assume Role

↓

Production Account
```

Developers never receive permanent production credentials.

---

# Cross-Account Architecture

```text
Account A

↓

IAM User

↓

Assume Role

↓

AWS STS

↓

Temporary Credentials

↓

Account B
```

This is the recommended enterprise architecture.

---

# Cross-Account S3 Access

Example

```text
Application

↓

IAM Role

↓

Account B

↓

Amazon S3 Bucket
```

No credential sharing is required.

---

# Cross-Account DevOps Example

A CI/CD pipeline

```text
GitHub Actions

↓

IAM Role

↓

Production Account

↓

Deploy ECS
```

GitHub assumes an IAM Role to deploy infrastructure securely.

---

# Identity Federation

Employees often authenticate using corporate identity providers.

Example

```text
Employee

↓

Microsoft Entra ID

↓

IAM Identity Center

↓

AWS STS

↓

AWS Console
```

Employees use existing corporate credentials.

---

# AssumeRole API

Applications use the

```text
AssumeRole
```

API to request temporary credentials.

Workflow

```text
Application

↓

AssumeRole

↓

STS

↓

Temporary Credentials

↓

AWS Resources
```

---

# Session Duration

Roles have configurable session durations.

Example

```text
1 Hour

↓

Credentials Expire

↓

New Session Required
```

Short sessions improve security.

---

# Role Chaining

A role can assume another role.

Example

```text
Developer

↓

Role A

↓

Role B

↓

Production
```

AWS limits role chaining duration to improve security.

---

# Enterprise Architecture

```text
Developer

↓

IAM Identity Center

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

Production Account

↓

Amazon EKS

↓

Amazon S3

↓

CloudWatch
```

Every request uses temporary credentials.

---

# Security Benefits

Using IAM Roles provides

- No Hardcoded Credentials
- Automatic Credential Rotation
- Least Privilege
- Better Auditing
- Secure Cross-Account Access
- Temporary Authentication

---

# Best Practices

- Use IAM Roles instead of Access Keys.
- Use temporary credentials whenever possible.
- Enable IAM Roles for EC2, ECS, Lambda, and EKS.
- Use IAM Roles for cross-account access.
- Limit session duration.
- Follow least privilege.
- Audit AssumeRole activity using CloudTrail.
- Use IAM Identity Center for workforce access.

---

# Common Mistakes

- Hardcoding Access Keys in applications.
- Sharing AWS credentials between accounts.
- Using long-lived IAM Users for automation.
- Granting excessive permissions to roles.
- Ignoring Trust Policies.
- Never auditing role usage.
- Using the root account instead of IAM Roles.

---

# Interview Questions

## Basic

- What is an IAM Role?
- What is AWS STS?
- Why are IAM Roles more secure than Access Keys?

## Intermediate

- IAM User vs IAM Role.
- Explain AssumeRole.
- What is Cross-Account Access?
- What are temporary credentials?

## Advanced

- Design a secure multi-account AWS environment where developers deploy applications into production without using permanent credentials.
- Explain how IAM Roles, AWS STS, Trust Policies, and Permission Policies work together during a cross-account deployment.
- Design an Amazon EKS platform where each Kubernetes microservice securely accesses different AWS services using IAM Roles for Service Accounts (IRSA), ensuring least-privilege access and complete auditability.

---

# Chapter 4 - IAM Policies, Permission Boundaries & Policy Evaluation Logic (Deep Dive)

AWS IAM determines whether every API request should be allowed or denied.

Every action such as

- Launching an EC2 instance
- Reading an S3 object
- Creating an RDS database
- Updating an IAM Role
- Accessing Secrets Manager

goes through AWS's **Policy Evaluation Engine**.

Understanding how IAM policies are evaluated is one of the most important AWS security topics for Solutions Architect, DevOps Engineer, and Security Engineer interviews.

---

# What is an IAM Policy?

An IAM Policy is a JSON document that defines

- Who
- Can perform
- Which actions
- On which resources
- Under what conditions

Policies determine permissions.

---

# IAM Policy Architecture

```text
User / Role

↓

IAM Policy

↓

AWS Policy Engine

↓

Allow

OR

Deny
```

Every AWS API request is evaluated.

---

# IAM Policy Components

Every policy consists of

- Version
- Statement
- Effect
- Action
- Resource
- Condition

These fields define access behavior.

---

# Policy Structure

Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-bucket/*"
    }
  ]
}
```

---

# Version

Example

```text
2012-10-17
```

This specifies the IAM policy language version.

---

# Statement

A policy can contain multiple statements.

Example

```text
Policy

├── Statement 1

├── Statement 2

└── Statement 3
```

Each statement is evaluated independently.

---

# Effect

Effect determines

```text
Allow

OR

Deny
```

AWS supports only two effects.

---

# Action

Action specifies

what operations are allowed.

Examples

```text
ec2:RunInstances

s3:GetObject

lambda:InvokeFunction

rds:CreateDBInstance
```

---

# Resource

Resource specifies

where permissions apply.

Example

```text
Amazon S3 Bucket

↓

company-data
```

Instead of

```text
*
```

Always restrict permissions to required resources.

---

# Condition

Conditions make permissions more secure.

Example

```text
Allow

↓

EC2 Start

↓

Only

↓

Business Hours
```

Conditions support

- IP Address
- Time
- MFA
- Tags
- Region
- VPC Endpoint

---

# IAM Policy Flow

```text
Request

↓

Authenticate

↓

Collect Policies

↓

Evaluate Policies

↓

Decision

↓

Allow

OR

Deny
```

AWS evaluates every request.

---

# Identity-Based Policies

Attached to

- Users
- Groups
- Roles

Example

```text
Developer Role

↓

Amazon S3 Read
```

Most IAM policies are identity-based.

---

# Resource-Based Policies

Attached directly to AWS resources.

Examples

- Amazon S3 Bucket Policy
- SQS Queue Policy
- SNS Topic Policy
- KMS Key Policy

Architecture

```text
User

↓

S3 Bucket Policy

↓

Amazon S3
```

---

# Managed Policies

Reusable policies.

Types

```text
AWS Managed

↓

Customer Managed
```

Reusable across multiple identities.

---

# Inline Policies

Attached directly to one identity.

```text
Developer

↓

Inline Policy
```

Cannot be shared.

---

# Explicit Allow

Example

```text
Allow

↓

s3:GetObject
```

If no Deny exists,

access is granted.

---

# Explicit Deny

Example

```text
Deny

↓

Delete S3 Bucket
```

Explicit Deny always wins.

---

# Implicit Deny

By default

everything is denied.

Only explicitly allowed actions become accessible.

```text
No Policy

↓

Access Denied
```

---

# Policy Evaluation Logic

AWS follows this sequence.

```text
Request

↓

Authenticate

↓

Explicit Deny?

↓

Yes

↓

Denied

────────────

No

↓

Explicit Allow?

↓

Yes

↓

Allowed

────────────

No

↓

Implicit Deny
```

This evaluation order is critical.

---

# Example

Suppose

Developer Policy

```text
Allow

↓

Read S3
```

Another Policy

```text
Deny

↓

Delete Bucket
```

Results

```text
Read

↓

Allowed

Delete

↓

Denied
```

The Deny overrides the Allow.

---

# Permission Boundaries

Permission Boundaries define

the **maximum permissions** an IAM User or Role can receive.

Architecture

```text
IAM Role

↓

Permissions

↓

Permission Boundary

↓

Maximum Allowed
```

Even if another policy grants additional permissions,

the boundary limits access.

---

# Example

Developer Role

```text
Allow

↓

AdministratorAccess
```

Permission Boundary

```text
Only EC2

Only CloudWatch
```

Effective permissions become

```text
EC2

CloudWatch
```

Administrator permissions are restricted.

---

# Why Permission Boundaries?

Large enterprises delegate IAM management.

Example

Security Team

↓

Creates Permission Boundary

↓

Development Team

↓

Creates Roles

↓

Cannot Exceed Boundary
```

This enables safe delegation.

---

# IAM Policy Conditions

Policies can enforce additional requirements.

Examples

```text
Require MFA

↓

Allow Delete

──────────────

Corporate IP Only

↓

AWS Console Access

──────────────

Specific AWS Region

↓

Launch EC2
```

Conditions significantly improve security.

---

# Tag-Based Access Control (ABAC)

Instead of assigning permissions by user,

AWS can evaluate resource tags.

Example

```text
Developer

↓

Project=A

↓

Access

↓

EC2

Project=A
```

Resources with different tags remain inaccessible.

---

# Least Privilege Example

Instead of

```text
AmazonS3FullAccess
```

Grant

```text
GetObject

PutObject

Specific Bucket
```

Permissions remain limited.

---

# Enterprise Architecture

```text
Developer

↓

IAM Role

↓

Permission Boundary

↓

IAM Policy

↓

Amazon S3

↓

CloudWatch

↓

Amazon ECR
```

Every permission passes through multiple controls.

---

# Best Practices

- Follow least privilege.
- Prefer Customer Managed Policies.
- Avoid wildcard (*) resources whenever possible.
- Use Permission Boundaries for delegated administration.
- Use Conditions to strengthen policies.
- Audit permissions regularly.
- Remove unused permissions.
- Use AWS IAM Access Analyzer to identify overly permissive access.

---

# Common Mistakes

- Using AdministratorAccess unnecessarily.
- Granting Resource "*".
- Ignoring Explicit Deny.
- Creating duplicate policies.
- Never reviewing permissions.
- Not using Permission Boundaries in large organizations.
- Ignoring policy conditions.

---

# Interview Questions

## Basic

- What is an IAM Policy?
- What are the components of an IAM Policy?
- What is the difference between Allow and Deny?

## Intermediate

- Identity-Based Policy vs Resource-Based Policy.
- Managed Policy vs Inline Policy.
- Explain IAM Policy Evaluation Logic.
- What are Permission Boundaries?

## Advanced

- Design IAM policies for an enterprise where developers can manage EC2 resources but cannot modify IAM, billing, or production databases.
- Explain how Explicit Deny, Permission Boundaries, IAM Policies, and Resource Policies work together during permission evaluation.
- Design a secure tag-based access control (ABAC) model for a multinational organization with multiple departments, AWS accounts, and production environments while enforcing least privilege.

---

