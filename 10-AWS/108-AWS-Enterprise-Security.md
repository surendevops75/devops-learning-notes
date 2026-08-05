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

# Chapter 5 - Multi-Factor Authentication (MFA), Password Policies & Root Account Security

Identity is the first line of defense in AWS.

However,

a username and password alone are no longer sufficient to protect cloud environments.

Most successful cloud attacks occur because

- Weak Passwords
- Stolen Credentials
- Phishing Attacks
- Credential Reuse
- Compromised Root Accounts

AWS provides multiple identity protection mechanisms such as

- Multi-Factor Authentication (MFA)
- Password Policies
- Root User Protection
- Credential Rotation
- Temporary Credentials

These significantly reduce the risk of unauthorized access.

---

# Why MFA is Important

Consider this scenario

```text
Username

↓

Password

↓

AWS Console
```

If the password is stolen,

the attacker gains access.

With MFA

```text
Username

↓

Password

↓

MFA Code

↓

AWS Console
```

Even if the password is compromised,

the attacker still cannot log in.

---

# What is Multi-Factor Authentication?

Multi-Factor Authentication (MFA) requires

two or more authentication factors.

```text
Something You Know

↓

Password

+

Something You Have

↓

Authenticator App

OR

Hardware Token
```

AWS strongly recommends enabling MFA for every privileged account.

---

# Authentication Factors

Authentication generally falls into three categories.

```text
Knowledge

↓

Password

────────────────

Possession

↓

Phone

↓

Security Key

────────────────

Biometric

↓

Fingerprint

↓

Face Recognition
```

MFA combines multiple factors.

---

# AWS MFA Workflow

```text
User

↓

Username

↓

Password

↓

MFA Challenge

↓

AWS Console
```

Only after successful verification is access granted.

---

# MFA Devices Supported by AWS

AWS supports

- Virtual MFA Applications
- Hardware MFA Devices
- FIDO2 Security Keys
- Passkeys

Common virtual MFA apps include

- Google Authenticator
- Microsoft Authenticator
- Authy

---

# Virtual MFA

Example

```text
AWS Console

↓

Scan QR Code

↓

Authenticator App

↓

6-Digit Code

↓

Login
```

The code changes every 30 seconds.

---

# Hardware MFA

Hardware tokens generate authentication codes without relying on a mobile device.

Architecture

```text
User

↓

Hardware Token

↓

Authentication Code

↓

AWS
```

Suitable for highly secure environments.

---

# FIDO2 Security Keys

Modern enterprises increasingly use

- YubiKey
- Feitian Keys

Authentication

```text
User

↓

Security Key

↓

AWS Login
```

These provide strong phishing resistance.

---

# Passkeys

AWS also supports passkeys for compatible devices.

Example

```text
User

↓

Device Authentication

↓

AWS Console
```

Passkeys eliminate traditional passwords for supported workflows.

---

# MFA for Root User

AWS strongly recommends

```text
Root User

↓

MFA Enabled
```

The root account should never exist without MFA.

---

# Why Protect the Root Account?

The root user can

- Close AWS Account
- Delete Resources
- Modify Billing
- Delete IAM Users
- Disable Security Controls

Compromising the root account compromises the entire AWS account.

---

# Root User Best Practice

```text
Create AWS Account

↓

Enable MFA

↓

Create IAM Administrator

↓

Never Use Root Again
```

Root should only be used for tasks that specifically require it.

---

# Root User Tasks

Examples requiring the root user

- Change Support Plan
- Close AWS Account
- Modify Root Email
- Restore Certain IAM Permissions

Daily operations should use IAM identities instead.

---

# Password Policy

AWS IAM supports password policies.

Organizations can enforce

- Minimum Length
- Complexity
- Expiration
- Password Reuse Prevention
- Account Password Rotation

---

# Strong Password Policy

Example

```text
Minimum Length

16 Characters

↓

Uppercase

↓

Lowercase

↓

Numbers

↓

Special Characters
```

Long passwords are more resistant to attacks.

---

# Password Expiration

Example

```text
Password

↓

90 Days

↓

Change Required
```

Regular rotation reduces long-term credential exposure.

---

# Password Reuse Prevention

Prevent users from reusing previous passwords.

Example

```text
Last 10 Passwords

↓

Cannot Reuse
```

This strengthens account security.

---

# Login Protection

Authentication process

```text
User

↓

Password

↓

MFA

↓

IAM Policy

↓

AWS Resource
```

Multiple layers protect access.

---

# Temporary Credentials vs Passwords

Instead of

```text
Long-Term Credentials
```

AWS recommends

```text
IAM Role

↓

STS

↓

Temporary Credentials
```

Temporary credentials reduce attack exposure.

---

# Credential Rotation

Long-lived credentials should be rotated regularly.

Example

```text
Access Key

↓

90 Days

↓

Rotate

↓

Delete Old Key
```

For IAM Roles,

AWS handles rotation automatically.

---

# Detecting Credential Misuse

Monitor

- Failed Login Attempts
- Root Account Usage
- MFA Disabled Events
- Console Logins
- API Calls

CloudTrail records all authentication events.

---

# Enterprise Authentication Architecture

```text
Employee

↓

Corporate Identity

↓

IAM Identity Center

↓

MFA

↓

AWS STS

↓

Temporary Credentials

↓

AWS Resources
```

Modern enterprises rarely create individual IAM Users.

---

# Enterprise Banking Example

```text
Employee

↓

Microsoft Entra ID

↓

IAM Identity Center

↓

MFA

↓

IAM Role

↓

AWS STS

↓

Production AWS
```

Every login requires

- Corporate Authentication
- MFA
- Temporary Credentials

---

# Best Practices

- Enable MFA for all users.
- Always enable MFA for the root account.
- Never share AWS accounts.
- Use long, complex passwords.
- Rotate long-lived credentials regularly.
- Prefer IAM Roles over IAM Users.
- Monitor login activity with CloudTrail.
- Disable unused credentials immediately.

---

# Common Mistakes

- Leaving the root account without MFA.
- Sharing root credentials.
- Using weak passwords.
- Reusing passwords.
- Never rotating access keys.
- Storing passwords in documentation.
- Creating IAM users for applications instead of IAM Roles.

---

# Interview Questions

## Basic

- What is Multi-Factor Authentication (MFA)?
- Why should the AWS root account always have MFA enabled?
- What is the difference between authentication and authorization?

## Intermediate

- Virtual MFA vs Hardware MFA.
- Why are IAM Roles preferred over long-lived credentials?
- Explain AWS password policy best practices.

## Advanced

- Design a secure enterprise authentication architecture using Microsoft Entra ID, IAM Identity Center, IAM Roles, AWS STS, and MFA.
- Explain how MFA, password policies, IAM Roles, CloudTrail, and temporary credentials work together to secure AWS accounts.
- A company has hundreds of AWS users still using IAM usernames and passwords without MFA. Design a migration strategy to modernize authentication while minimizing operational disruption.

---

# Chapter 6 - AWS IAM Identity Center (AWS SSO), Identity Federation & Enterprise Authentication

As organizations grow,

managing hundreds or thousands of IAM Users becomes difficult.

Imagine an enterprise with

- 5,000 Employees
- 300 AWS Accounts
- 50 Development Teams
- Multiple Cloud Platforms

Creating IAM Users for every employee in every AWS account would become impossible to manage.

Instead, enterprises use centralized identity management through

- AWS IAM Identity Center
- Identity Federation
- Corporate Identity Providers
- Single Sign-On (SSO)

This enables users to log in once and securely access multiple AWS accounts and applications.

---

# What is AWS IAM Identity Center?

AWS IAM Identity Center (formerly AWS Single Sign-On) is a centralized identity management service.

It provides

- Single Sign-On (SSO)
- Centralized User Management
- Multi-Account Access
- Application Access
- Permission Management
- Identity Federation

Users authenticate once and access multiple AWS resources securely.

---

# IAM Identity Center Architecture

```text
Employee

↓

IAM Identity Center

↓

AWS Accounts

↓

Applications
```

Authentication is centralized.

---

# Why IAM Identity Center?

Without IAM Identity Center

```text
Developer

↓

AWS Account A

↓

IAM User

────────────

AWS Account B

↓

IAM User

────────────

AWS Account C

↓

IAM User
```

Users require separate credentials.

With IAM Identity Center

```text
Developer

↓

IAM Identity Center

↓

AWS Account A

↓

AWS Account B

↓

AWS Account C
```

One login provides access to multiple accounts.

---

# What is Single Sign-On (SSO)?

Single Sign-On allows users to

authenticate once

and access multiple systems without repeated logins.

Example

```text
Employee

↓

Login Once

↓

AWS

↓

GitHub

↓

Jira

↓

Slack
```

---

# Enterprise Authentication Flow

```text
Employee

↓

Corporate Identity

↓

IAM Identity Center

↓

AWS STS

↓

Temporary Credentials

↓

AWS Resources
```

Permanent AWS credentials are never required.

---

# Identity Sources

IAM Identity Center supports

- Internal Identity Store
- Microsoft Entra ID
- Active Directory
- Okta
- Ping Identity
- OneLogin
- Other SAML 2.0 Providers

Organizations continue using existing corporate identities.

---

# Internal Identity Store

Small organizations may use

```text
IAM Identity Center

↓

Internal Users

↓

AWS Accounts
```

No external identity provider is required.

---

# Microsoft Entra ID Integration

Architecture

```text
Employee

↓

Microsoft Entra ID

↓

IAM Identity Center

↓

AWS
```

Users log in using corporate credentials.

---

# Active Directory Integration

Many enterprises use Microsoft Active Directory.

```text
Employee

↓

Active Directory

↓

IAM Identity Center

↓

AWS Accounts
```

Existing user accounts remain unchanged.

---

# Okta Integration

Example

```text
Employee

↓

Okta

↓

IAM Identity Center

↓

AWS Console
```

Authentication remains centralized.

---

# What is Identity Federation?

Identity Federation allows external identity providers to authenticate AWS users.

Instead of AWS storing passwords,

authentication is delegated.

Architecture

```text
Corporate Identity

↓

Federation

↓

AWS
```

---

# SAML Authentication

Most enterprise identity providers use

Security Assertion Markup Language (SAML).

Workflow

```text
Employee

↓

Identity Provider

↓

SAML Assertion

↓

IAM Identity Center

↓

AWS Access
```

---

# OpenID Connect (OIDC)

Modern cloud applications commonly use

OpenID Connect.

Architecture

```text
Application

↓

OIDC Provider

↓

Authentication

↓

AWS
```

OIDC is commonly used for web applications.

---

# AWS Organizations Integration

IAM Identity Center integrates directly with AWS Organizations.

```text
AWS Organizations

├── Production

├── Development

├── Security

├── Shared Services

↓

IAM Identity Center
```

Permissions are centrally managed.

---

# Permission Sets

IAM Identity Center uses

Permission Sets

instead of directly assigning IAM policies.

Example

```text
Developer

↓

Developer Permission Set

↓

Development Account
```

Permission Sets are automatically provisioned.

---

# Permission Set Architecture

```text
Permission Set

↓

IAM Role

↓

AWS Account

↓

User Access
```

AWS automatically creates the required IAM Roles.

---

# Enterprise Example

```text
Security Team

↓

Permission Sets

↓

Developers

↓

ReadOnly

↓

Production

────────────

Developers

↓

PowerUser

↓

Development
```

Different accounts receive different permissions.

---

# Temporary Credentials

IAM Identity Center uses AWS STS.

Workflow

```text
User

↓

Authentication

↓

STS

↓

Temporary Credentials

↓

AWS Console
```

No permanent credentials are stored.

---

# Multi-Account Login

Example

```text
Developer

↓

IAM Identity Center

↓

Production

↓

Development

↓

Testing

↓

Security
```

Users choose the account they need.

---

# Application Access

IAM Identity Center also supports

- Salesforce
- Slack
- GitHub Enterprise
- Microsoft 365
- ServiceNow

One authentication provides access to multiple enterprise applications.

---

# Authentication Lifecycle

```text
User Login

↓

Identity Provider

↓

IAM Identity Center

↓

STS

↓

Temporary Credentials

↓

AWS Access

↓

Session Expires

↓

Reauthenticate
```

Sessions automatically expire.

---

# Enterprise Banking Example

```text
Employee

↓

Microsoft Entra ID

↓

IAM Identity Center

↓

Permission Set

↓

Production Account

↓

IAM Role

↓

Temporary Credentials

↓

Banking Platform
```

Employees never receive permanent AWS credentials.

---

# Benefits

- Centralized Authentication
- Single Sign-On
- Temporary Credentials
- Improved Security
- Simplified User Management
- Multi-Account Access
- Better Compliance
- Reduced Credential Management

---

# Best Practices

- Integrate with a corporate identity provider.
- Use IAM Identity Center instead of creating IAM Users.
- Assign Permission Sets rather than individual IAM policies.
- Enable MFA through the identity provider.
- Use temporary credentials.
- Review Permission Sets regularly.
- Integrate with AWS Organizations.
- Audit login activity using CloudTrail.

---

# Common Mistakes

- Creating IAM Users for every employee.
- Sharing AWS credentials.
- Using permanent access keys for workforce users.
- Assigning Administrator permissions to everyone.
- Ignoring MFA.
- Managing permissions individually instead of using Permission Sets.
- Not integrating with AWS Organizations.

---

# Interview Questions

## Basic

- What is AWS IAM Identity Center?
- What is Single Sign-On (SSO)?
- What is Identity Federation?

## Intermediate

- IAM Users vs IAM Identity Center.
- Permission Sets vs IAM Policies.
- Explain SAML authentication.
- How does IAM Identity Center use AWS STS?

## Advanced

- Design an enterprise authentication solution for a company with 5,000 employees and 300 AWS accounts.
- Explain how IAM Identity Center, AWS Organizations, Microsoft Entra ID, AWS STS, Permission Sets, and IAM Roles work together in a multi-account AWS environment.
- Your organization currently manages thousands of IAM Users across multiple AWS accounts. Design a migration strategy to AWS IAM Identity Center that minimizes operational disruption while improving security, scalability, and compliance.

---

# Chapter 7 - AWS Organizations, Multi-Account Strategy & Governance (Deep Dive)

As cloud environments grow,

managing everything inside a single AWS account becomes difficult and risky.

Imagine an enterprise with

- 500 Developers
- 150 Applications
- 30 Business Units
- Multiple Environments
- Separate Security Teams

Managing everything inside one AWS account leads to

- Poor Security
- Permission Conflicts
- Billing Challenges
- Compliance Issues
- Operational Complexity

AWS Organizations solves these problems by enabling centralized management of multiple AWS accounts.

---

# What is AWS Organizations?

AWS Organizations is a service that allows you to centrally manage multiple AWS accounts.

It provides

- Centralized Account Management
- Centralized Billing
- Governance
- Security Policies
- Account Automation
- Permission Delegation

Large enterprises rarely operate using a single AWS account.

---

# AWS Organizations Architecture

```text
AWS Organization

│

├── Management Account

│

├── Security Account

├── Logging Account

├── Shared Services Account

├── Networking Account

├── Production Account

├── Development Account

├── Testing Account

└── Sandbox Account
```

Each account serves a dedicated purpose.

---

# Why Multiple AWS Accounts?

Instead of

```text
Everything

↓

One AWS Account
```

Use

```text
Separate Accounts

↓

Better Isolation

↓

Better Security
```

Benefits

- Fault Isolation
- Security Isolation
- Independent Billing
- Compliance
- Easier Permission Management

---

# Management Account

Every AWS Organization has one

Management Account.

Responsibilities

- Create Accounts
- Consolidated Billing
- Organization Policies
- Account Management

This account should not host production workloads.

---

# Member Accounts

Member Accounts contain workloads.

Example

```text
Production

↓

Applications

────────────

Development

↓

Applications

────────────

Testing

↓

Applications
```

Applications remain isolated.

---

# Organizational Units (OUs)

Accounts are grouped into

Organizational Units.

Example

```text
Root

│

├── Production OU

│    ├── Prod-1

│    └── Prod-2

│

├── Development OU

│    ├── Dev-1

│    └── Dev-2

│

└── Security OU

     ├── Audit

     └── Logging
```

Policies can be applied at the OU level.

---

# Benefits of Organizational Units

- Easier Governance
- Central Policy Management
- Environment Separation
- Delegated Administration
- Better Security

---

# Consolidated Billing

Instead of

```text
10 Accounts

↓

10 Bills
```

AWS Organizations provides

```text
One Invoice

↓

Entire Organization
```

This simplifies financial management.

---

# Cost Benefits

Organizations also share

- Reserved Instances
- Savings Plans

Across eligible accounts.

This reduces overall infrastructure costs.

---

# Service Control Policies (SCPs)

Service Control Policies are one of the most powerful AWS governance features.

SCPs define

the **maximum permissions** available within an account or Organizational Unit.

---

# SCP Architecture

```text
AWS Organization

↓

Organizational Unit

↓

Service Control Policy

↓

AWS Account

↓

IAM Users / Roles
```

Even administrators cannot exceed SCP restrictions.

---

# Example SCP

Suppose

Security Team wants to block

Amazon EC2 deletion.

```text
SCP

↓

Deny

↓

Terminate EC2
```

Even if an IAM Role has AdministratorAccess,

termination remains blocked.

---

# SCP vs IAM Policy

| IAM Policy | SCP |
|------------|-----|
| Grants Permissions | Limits Maximum Permissions |
| Applied to Users/Roles | Applied to Accounts/OUs |
| Identity Level | Organization Level |
| Cannot Override SCP | Highest Governance Control |

---

# Multi-Account Strategy

A common enterprise structure

```text
AWS Organization

│

├── Networking

├── Shared Services

├── Security

├── Production

├── Development

├── QA

└── Sandbox
```

Each workload is isolated.

---

# Security Account

A dedicated Security Account usually contains

- GuardDuty
- Security Hub
- IAM Access Analyzer
- Inspector
- Audit Logs

Security remains centralized.

---

# Logging Account

Organizations centralize logs.

```text
CloudTrail

↓

Logging Account

↓

Amazon S3

↓

Long-Term Storage
```

Attackers cannot easily delete logs from workload accounts.

---

# Shared Services Account

Contains shared enterprise services

```text
Active Directory

↓

DNS

↓

Jenkins

↓

GitHub Runners

↓

Monitoring

↓

Artifact Repository
```

Other accounts consume these services.

---

# Networking Account

Central networking resources

```text
Transit Gateway

↓

Direct Connect

↓

VPN

↓

Shared Networking
```

All application accounts connect through the networking account.

---

# Account Provisioning

New accounts can be created automatically.

Workflow

```text
AWS Organizations

↓

Create Account

↓

Apply SCP

↓

Configure IAM

↓

Ready
```

Automation ensures consistency.

---

# Delegated Administration

Organizations can delegate management.

Example

```text
Management Account

↓

Delegate

↓

Security Account

↓

Security Hub
```

Reduces dependency on the Management Account.

---

# Enterprise Example

```text
AWS Organization

│

├── Security

│    ├── GuardDuty

│    ├── Security Hub

│    └── IAM Access Analyzer

│

├── Networking

│    ├── Transit Gateway

│    └── Direct Connect

│

├── Shared Services

│    ├── Active Directory

│    └── Jenkins

│

├── Production

│

└── Development
```

Each team works independently while governance remains centralized.

---

# Best Practices

- Use multiple AWS accounts.
- Separate production and development.
- Apply SCPs to Organizational Units.
- Keep the Management Account dedicated to administration.
- Centralize logging and security.
- Use AWS Organizations with IAM Identity Center.
- Automate account creation.
- Regularly review SCPs.

---

# Common Mistakes

- Running all workloads in one AWS account.
- Using the Management Account for production applications.
- Granting unrestricted AdministratorAccess without SCPs.
- Mixing production and development resources.
- Storing audit logs in workload accounts.
- Ignoring account-level governance.

---

# Interview Questions

## Basic

- What is AWS Organizations?
- What is an Organizational Unit (OU)?
- What is Consolidated Billing?

## Intermediate

- IAM Policy vs Service Control Policy (SCP).
- Why do enterprises use multiple AWS accounts?
- Explain the purpose of the Management Account.

## Advanced

- Design a secure AWS Organization for a multinational enterprise with Production, Development, Security, Networking, and Shared Services accounts.
- Explain how AWS Organizations, Organizational Units, Service Control Policies, IAM Identity Center, and IAM Roles work together to implement enterprise governance.
- Your company currently operates everything in a single AWS account and wants to migrate to a multi-account architecture. Design the complete migration strategy, account structure, governance model, and security controls while minimizing operational disruption.

---

# Chapter 8 - AWS Network Security (VPC, Security Groups, NACLs & Network Firewall)

Securing identities alone is not enough.

Even if users are properly authenticated,

applications can still be compromised through

- Open Ports
- Misconfigured Firewalls
- Public Subnets
- Internet Exposure
- Unauthorized Network Access

AWS provides multiple layers of network security to protect workloads.

Enterprise network security is built using

- Amazon VPC
- Security Groups
- Network ACLs
- AWS Network Firewall
- AWS WAF
- AWS Shield
- Private Subnets
- VPC Endpoints

Each layer protects a different part of the network.

---

# Enterprise Network Security Architecture

```text
Internet

↓

AWS Shield

↓

AWS WAF

↓

Application Load Balancer

↓

Security Group

↓

Private Subnet

↓

Application

↓

Security Group

↓

Database

↓

Network ACL

↓

VPC
```

Multiple security layers protect every request.

---

# Amazon VPC

Amazon VPC provides

logical network isolation.

Architecture

```text
AWS Region

↓

Amazon VPC

↓

Private Network

↓

AWS Resources
```

Resources inside one VPC are isolated from others unless explicitly connected.

---

# Public vs Private Subnets

Public Subnet

```text
Internet

↓

Public Subnet

↓

ALB

↓

Bastion Host
```

Private Subnet

```text
Private Subnet

↓

Application

↓

Database
```

Production workloads should generally run in private subnets.

---

# VPC Security Layers

```text
Internet

↓

Route Table

↓

Network ACL

↓

Subnet

↓

Security Group

↓

EC2
```

Traffic passes through multiple security controls.

---

# What is a Security Group?

A Security Group is a

**stateful virtual firewall**

attached to AWS resources.

It controls

- Inbound Traffic
- Outbound Traffic

Example

```text
Internet

↓

443

↓

Application Load Balancer
```

Only HTTPS traffic is allowed.

---

# Security Group Architecture

```text
Client

↓

Security Group

↓

EC2
```

If traffic matches the rules,

it is allowed.

Otherwise,

it is denied.

---

# Stateful Firewall

Security Groups are

stateful.

Example

```text
Inbound

↓

Allow

↓

Response

↓

Automatically Allowed
```

No outbound rule is required for return traffic.

---

# Example Security Group

Application Server

Inbound

```text
443

HTTPS

From ALB
```

Outbound

```text
3306

MySQL

To Database
```

Everything else is denied.

---

# Database Security Group

```text
Application

↓

3306

↓

Database
```

Database access is allowed only from the application layer.

No Internet access exists.

---

# Security Group Referencing

Instead of using IP addresses,

Security Groups can reference other Security Groups.

Example

```text
ALB SG

↓

Application SG

↓

Database SG
```

This improves scalability.

---

# What is a Network ACL?

A Network ACL (NACL) is a

**stateless subnet-level firewall.**

Architecture

```text
Internet

↓

Network ACL

↓

Subnet

↓

EC2
```

It controls traffic entering and leaving the subnet.

---

# Stateless Firewall

Unlike Security Groups,

Network ACLs are stateless.

Example

```text
Inbound

↓

Allow

↓

Outbound

↓

Must Also Allow
```

Both directions require explicit rules.

---

# Security Group vs Network ACL

| Security Group | Network ACL |
|---------------|-------------|
| Stateful | Stateless |
| Instance Level | Subnet Level |
| Supports Allow Rules Only | Supports Allow & Deny Rules |
| Automatically Allows Return Traffic | Return Traffic Must Be Explicitly Allowed |

---

# Rule Evaluation

Security Groups

```text
Default

↓

Deny

↓

Allow Matching Rules
```

Network ACLs

```text
Rule Number

↓

Lowest Number First

↓

First Match Wins
```

Order matters for Network ACLs.

---

# Multi-Tier Application Security

```text
Internet

↓

ALB SG

↓

Web Tier SG

↓

Application Tier SG

↓

Database SG
```

Each tier communicates only with the required layer.

---

# Bastion Host

Administrators should not SSH directly into application servers.

Instead

```text
Administrator

↓

Bastion Host

↓

Private EC2
```

Modern architectures increasingly replace bastion hosts with AWS Systems Manager Session Manager.

---

# NAT Gateway

Private instances often require Internet access.

Architecture

```text
Private EC2

↓

NAT Gateway

↓

Internet
```

The Internet cannot initiate connections back to the private instances.

---

# VPC Endpoints

Instead of accessing AWS services over the Internet,

use VPC Endpoints.

Example

```text
EC2

↓

VPC Endpoint

↓

Amazon S3
```

Traffic never leaves the AWS network.

---

# AWS Network Firewall

AWS Network Firewall provides

advanced packet inspection.

Architecture

```text
Internet

↓

AWS Network Firewall

↓

VPC

↓

Applications
```

Capabilities

- Stateful Inspection
- Intrusion Detection
- Domain Filtering
- Deep Packet Inspection

---

# AWS WAF

AWS WAF protects

Layer 7 (HTTP/HTTPS) applications.

Architecture

```text
Users

↓

AWS WAF

↓

ALB

↓

Application
```

Protects against

- SQL Injection
- Cross-Site Scripting (XSS)
- HTTP Floods
- Malicious Bots

---

# AWS Shield

AWS Shield protects against

Distributed Denial of Service (DDoS) attacks.

Architecture

```text
Internet

↓

DDoS Attack

↓

AWS Shield

↓

Application
```

AWS Shield Standard is enabled automatically for supported AWS services.

---

# Enterprise Network Architecture

```text
Internet

↓

AWS Shield

↓

AWS WAF

↓

CloudFront

↓

Application Load Balancer

↓

Security Group

↓

Private EC2

↓

Database Security Group

↓

Amazon RDS
```

Every layer contributes to security.

---

# Banking Example

```text
Customers

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

Private EKS

↓

Aurora

↓

Private Subnets
```

No production workloads are directly exposed to the Internet.

---

# Best Practices

- Use private subnets for production workloads.
- Follow least-privilege Security Group rules.
- Use Security Group references instead of IP addresses where possible.
- Deploy one NAT Gateway per Availability Zone.
- Use VPC Endpoints for AWS service access.
- Enable AWS WAF for Internet-facing applications.
- Use AWS Shield for DDoS protection.
- Implement AWS Network Firewall for advanced traffic inspection.

---

# Common Mistakes

- Allowing SSH (22) from 0.0.0.0/0.
- Opening database ports to the Internet.
- Using public subnets for databases.
- Overusing 0.0.0.0/0 in Security Groups.
- Forgetting outbound Network ACL rules.
- Not enabling AWS WAF on public applications.
- Accessing S3 through the Internet instead of VPC Endpoints.

---

# Interview Questions

## Basic

- What is a Security Group?
- What is a Network ACL?
- Security Group vs Network ACL.

## Intermediate

- What is a VPC Endpoint?
- Why are Security Groups stateful?
- Why are Network ACLs stateless?
- Explain AWS WAF and AWS Shield.

## Advanced

- Design a secure three-tier banking application using VPC, private subnets, Security Groups, Network ACLs, AWS WAF, AWS Shield, VPC Endpoints, and AWS Network Firewall.
- Explain how network traffic flows from the Internet to an Amazon RDS database, describing how each AWS network security service protects the application.
- Your security team discovers that production databases are accessible from the Internet. Explain how you would redesign the VPC architecture, subnet layout, Security Groups, Network ACLs, and routing to eliminate unnecessary exposure while maintaining application functionality.

---

# Chapter 9 - AWS Data Security (Encryption, KMS, Secrets Manager & Certificate Management)

Data is one of the most valuable assets in any organization.

Even if your infrastructure is secure,

a data breach can expose

- Customer Information
- Financial Records
- API Keys
- Passwords
- Intellectual Property
- Healthcare Records

AWS provides multiple services to secure data throughout its lifecycle.

Enterprise data security focuses on

- Encryption
- Key Management
- Secret Management
- Certificate Management
- Secure Data Access

---

# Data Security Lifecycle

```text
Create

↓

Store

↓

Access

↓

Transfer

↓

Archive

↓

Delete
```

Every stage must be protected.

---

# Types of Data

AWS classifies data into two major categories.

```text
Data at Rest

↓

Stored Data

────────────────

Data in Transit

↓

Moving Data
```

Each requires different protection mechanisms.

---

# Data at Rest

Data stored on

- Amazon S3
- Amazon EBS
- Amazon RDS
- DynamoDB
- EFS
- FSx

is called

```text
Data at Rest
```

This data should always be encrypted.

---

# Data in Transit

Data moving between systems.

Example

```text
User

↓

HTTPS

↓

Application

↓

TLS

↓

Database
```

Encryption protects traffic during transmission.

---

# Encryption

Encryption converts readable information into unreadable data.

```text
Plain Text

↓

Encryption

↓

Cipher Text

↓

Decryption

↓

Plain Text
```

Without the encryption key,

the data cannot be read.

---

# Symmetric Encryption

Uses one key.

```text
Encrypt

↓

Key

↓

Decrypt

↓

Same Key
```

Examples

- AES-256
- AWS KMS Keys

---

# Asymmetric Encryption

Uses two keys.

```text
Public Key

↓

Encrypt

↓

Private Key

↓

Decrypt
```

Commonly used for

- TLS
- SSL
- Certificates

---

# AWS Key Management Service (KMS)

AWS KMS manages encryption keys.

It provides

- Key Creation
- Key Rotation
- Key Policies
- Auditing
- Secure Storage

Applications use KMS without managing encryption infrastructure.

---

# KMS Architecture

```text
Application

↓

AWS KMS

↓

Encryption Key

↓

Encrypted Data
```

Keys remain protected inside AWS.

---

# Customer Managed Keys (CMKs)

Organizations create and manage their own keys.

Benefits

- Rotation Control
- IAM Policies
- Auditing
- Compliance

Suitable for enterprise workloads.

---

# AWS Managed Keys

AWS automatically creates and manages keys.

Example

```text
Amazon RDS

↓

AWS Managed Key
```

Simpler,

but offers less administrative control.

---

# Envelope Encryption

AWS commonly uses envelope encryption.

```text
Data

↓

Data Key

↓

KMS Key

↓

Encrypted Data Key
```

This provides high performance and strong security.

---

# Amazon S3 Encryption

Amazon S3 supports multiple encryption options.

- SSE-S3
- SSE-KMS
- SSE-C
- Client-Side Encryption

---

# SSE-S3

AWS manages encryption automatically.

```text
Object

↓

Amazon S3

↓

Encrypted
```

Easy to enable.

---

# SSE-KMS

Uses AWS KMS.

```text
Object

↓

AWS KMS

↓

Encrypted Object
```

Provides auditing and fine-grained access control.

---

# Client-Side Encryption

Data is encrypted before reaching AWS.

```text
Application

↓

Encrypt

↓

Amazon S3
```

AWS never sees plaintext.

---

# Amazon EBS Encryption

Example

```text
EC2

↓

Encrypted EBS Volume

↓

AWS KMS
```

Snapshots remain encrypted.

---

# Amazon RDS Encryption

RDS supports encryption using AWS KMS.

```text
Application

↓

Amazon RDS

↓

Encrypted Storage
```

Backups and snapshots are also encrypted.

---

# DynamoDB Encryption

DynamoDB encrypts data automatically.

Architecture

```text
Application

↓

DynamoDB

↓

AWS KMS
```

No application changes are required.

---

# What is AWS Secrets Manager?

Applications need

- Database Passwords
- API Keys
- Tokens
- Certificates

Hardcoding these values is insecure.

Secrets Manager securely stores and rotates secrets.

---

# Secrets Manager Architecture

```text
Application

↓

Secrets Manager

↓

Retrieve Secret

↓

Database
```

Applications request secrets at runtime.

---

# Benefits

- Automatic Rotation
- Encryption
- Audit Logging
- IAM Integration
- Centralized Secret Storage

---

# Systems Manager Parameter Store

Parameter Store also stores

- Configuration
- Secrets
- Environment Variables

Architecture

```text
Application

↓

Parameter Store

↓

Configuration
```

Often used for application settings.

---

# Secrets Manager vs Parameter Store

| Secrets Manager | Parameter Store |
|-----------------|-----------------|
| Secret Rotation | Manual or Basic |
| Database Integration | Limited |
| Designed for Secrets | General Configuration |
| Higher Cost | Lower Cost |

---

# Certificate Management

Applications using HTTPS require SSL/TLS certificates.

AWS Certificate Manager (ACM) manages

- Certificate Issuance
- Renewal
- Deployment

---

# AWS Certificate Manager

Architecture

```text
Users

↓

HTTPS

↓

ACM Certificate

↓

Application Load Balancer
```

Certificates renew automatically.

---

# TLS Communication

```text
Client

↓

HTTPS

↓

ALB

↓

Application
```

Traffic remains encrypted.

---

# Enterprise Banking Example

```text
Customers

↓

HTTPS

↓

AWS WAF

↓

ALB

↓

Amazon EKS

↓

Secrets Manager

↓

Aurora

↓

AWS KMS Encryption
```

Sensitive data remains protected throughout the application.

---

# Enterprise Encryption Architecture

```text
Application

↓

TLS

↓

ALB

↓

Amazon EKS

↓

Secrets Manager

↓

Amazon RDS

↓

AWS KMS

↓

Encrypted Storage
```

Every layer implements encryption.

---

# Best Practices

- Encrypt all sensitive data at rest.
- Encrypt all traffic using TLS.
- Use AWS KMS for enterprise encryption.
- Store secrets in AWS Secrets Manager.
- Rotate secrets automatically.
- Enable encryption for EBS, RDS, and S3.
- Use ACM for certificate management.
- Audit KMS usage with CloudTrail.

---

# Common Mistakes

- Hardcoding database passwords.
- Storing secrets in Git repositories.
- Disabling encryption for databases.
- Using HTTP instead of HTTPS.
- Sharing encryption keys.
- Never rotating secrets.
- Using the same key for every workload.

---

# Interview Questions

## Basic

- What is AWS KMS?
- Data at Rest vs Data in Transit.
- What is AWS Secrets Manager?

## Intermediate

- SSE-S3 vs SSE-KMS.
- Secrets Manager vs Parameter Store.
- Symmetric vs Asymmetric Encryption.
- What is Envelope Encryption?

## Advanced

- Design a secure banking application using AWS KMS, Secrets Manager, ACM, Amazon RDS encryption, and TLS.
- Explain how encryption works from the user's browser to an encrypted Amazon RDS database.
- Design an enterprise key management strategy for a multi-account AWS organization, including KMS key policies, Secrets Manager, certificate management, automatic key rotation, and auditing with CloudTrail.

---

