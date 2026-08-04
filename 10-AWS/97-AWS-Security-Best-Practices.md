# AWS Security Best Practices

---

# Introduction

AWS Security Best Practices help organizations protect cloud workloads by implementing secure architectures, least-privilege access, encryption, monitoring, governance, and continuous compliance.

Security Goals

- Confidentiality
- Integrity
- Availability
- Accountability
- Compliance
- Resilience

---

# AWS Shared Responsibility Model

## AWS Responsibility

AWS manages

- Physical Security
- Data Centers
- Hardware
- Networking
- Hypervisor
- Global Infrastructure

---

## Customer Responsibility

Customers manage

- IAM
- Applications
- Operating Systems
- Data
- Network Configuration
- Encryption
- Security Groups
- Compliance Configuration

---

# Security Pillars

```text
Identity

↓

Network

↓

Data

↓

Application

↓

Infrastructure

↓

Monitoring

↓

Incident Response
```

---

# Identity and Access Management (IAM)

Purpose

- Authentication
- Authorization
- Access Control

---

# IAM Components

- Users
- Groups
- Roles
- Policies
- Identity Providers
- Permission Boundaries

---

# IAM Best Practice

Never use the root user for daily operations.

Use

```text
Root Account

↓

MFA Enabled

↓

Create Administrator Role

↓

Use IAM Identity Center / IAM Users
```

---

# Root Account Protection

Enable

- MFA
- Strong Password
- No Access Keys
- Hardware MFA (Recommended)

---

# Multi-Factor Authentication (MFA)

Enable MFA for

- Root User
- Administrators
- Developers
- Console Users

Supported Methods

- Authenticator Apps
- FIDO2 Security Keys
- Hardware Tokens

---

# Least Privilege Principle

Grant only permissions required to perform specific tasks.

Example

Instead of

```text
AdministratorAccess
```

Use

```text
AmazonS3ReadOnlyAccess
```

or a custom least-privilege policy.

---

# IAM Users

Best Practices

- Avoid long-term users when possible.
- Rotate credentials regularly.
- Enable MFA.
- Remove inactive users.
- Avoid sharing accounts.

---

# IAM Groups

Examples

```text
Developers

DevOps

Security

DBA

ReadOnly
```

Assign permissions to groups rather than individual users whenever possible.

---

# IAM Roles

Use IAM Roles for

- EC2
- Lambda
- ECS
- EKS
- Cross-Account Access
- Applications

Avoid embedding AWS credentials in code.

---

# Temporary Credentials

Preferred Workflow

```text
IAM Role

↓

STS

↓

Temporary Credentials

↓

Access AWS Services
```

---

# AWS STS

Benefits

- Temporary credentials
- Cross-account access
- Federation
- Improved security

---

# IAM Policies

Types

- AWS Managed Policies
- Customer Managed Policies
- Inline Policies

Preferred

```text
Customer Managed Policies
```

---

# Policy Structure

Contains

- Version
- Statement
- Effect
- Action
- Resource
- Condition

---

# Policy Evaluation

```text
Authentication

↓

Policy Evaluation

↓

Explicit Deny

↓

Allow

↓

Implicit Deny
```

---

# Explicit Deny

Highest Priority

Example

```text
Deny

↓

Overrides Allow
```

---

# IAM Conditions

Common Examples

- Source IP
- Time
- Region
- MFA Present
- VPC Endpoint
- Principal Tag

---

# Permission Boundaries

Purpose

Limit the maximum permissions that an IAM user or role can receive.

Useful for delegated administration.

---

# IAM Access Analyzer

Detects

- Public access
- Cross-account access
- External resource sharing
- Unused access

---

# Credential Report

Review

- Password age
- MFA status
- Active access keys
- Last login
- Credential rotation

---

# Access Advisor

Shows

- Services accessed
- Last accessed date
- Unused permissions

---

# Access Key Best Practices

Avoid

- Long-lived keys
- Hardcoded credentials
- Sharing keys

Rotate regularly if access keys are required.

---

# IAM Password Policy

Recommended

- Minimum 14 characters
- Uppercase
- Lowercase
- Numbers
- Symbols
- Password expiration
- Prevent password reuse

---

# Cross-Account Access

Recommended Workflow

```text
Account A

↓

Assume Role

↓

STS

↓

Temporary Credentials

↓

Account B
```

---

# Service Roles

Examples

- EC2 Instance Profile
- Lambda Execution Role
- ECS Task Role
- EKS IRSA
- CodeBuild Role

---

# Identity Federation

Supported Providers

- IAM Identity Center
- Active Directory
- SAML 2.0
- OpenID Connect (OIDC)

---

# IAM Identity Center

Benefits

- Single Sign-On
- Centralized user management
- Multi-account access
- MFA enforcement
- Permission sets

---

# Identity Security Checklist

- Root MFA enabled
- No root access keys
- Least privilege enforced
- IAM roles used instead of access keys
- Temporary credentials preferred
- Password policy configured
- Credential report reviewed
- Access Analyzer enabled
- MFA for privileged users
- Remove inactive identities

---

# Common IAM Mistakes

- Using the root account daily
- Sharing IAM users
- Overly permissive policies
- Using AdministratorAccess unnecessarily
- Hardcoding AWS credentials
- Ignoring unused permissions
- Missing MFA
- Long-lived access keys
- No credential rotation
- No policy reviews

---

# Best Practices

- Use IAM Roles instead of IAM Users wherever possible.
- Enable MFA for all privileged identities.
- Apply the principle of least privilege.
- Rotate credentials regularly.
- Use temporary credentials with AWS STS.
- Review IAM policies periodically.
- Enable IAM Access Analyzer.
- Remove unused users and permissions.
- Monitor credential reports regularly.
- Use IAM Identity Center for centralized access management.

---

# Summary

This section covered the AWS Shared Responsibility Model, IAM fundamentals, authentication, authorization, IAM users, groups, roles, policies, STS, Identity Center, MFA, credential management, and identity security best practices. These principles establish the foundation for securing AWS identities and access across enterprise environments.

---

# AWS Organizations

---

# Introduction

AWS Organizations helps centrally manage multiple AWS accounts using a single management account.

Benefits

- Centralized governance
- Consolidated billing
- Security controls
- Policy management
- Multi-account architecture

---

# Organization Structure

```text
Management Account

↓

Organizational Units (OUs)

↓

Member Accounts
```

---

# AWS Organizations Components

- Management Account
- Member Accounts
- Organizational Units (OUs)
- Service Control Policies (SCPs)
- Tag Policies
- Backup Policies
- AI Services Opt-Out Policies

---

# Multi-Account Strategy

Recommended Structure

```text
Management

↓

Security

↓

Log Archive

↓

Shared Services

↓

Development

↓

Testing

↓

Staging

↓

Production
```

---

# Why Multiple Accounts?

Benefits

- Security isolation
- Separate billing
- Resource quotas
- Blast radius reduction
- Independent deployments
- Compliance

---

# Organizational Units (OUs)

Purpose

Group AWS accounts based on business or security requirements.

Example

```text
Production OU

Development OU

Security OU

Infrastructure OU

Sandbox OU
```

---

# Nested OUs

Example

```text
Production

↓

Finance

↓

Payments

↓

Core Banking
```

---

# Service Control Policies (SCPs)

Purpose

Define the maximum permissions available to accounts within an Organization.

Important

SCPs **do not grant permissions**.

They only restrict permissions.

---

# SCP Evaluation

```text
IAM Policy

↓

SCP Check

↓

Allowed

↓

AWS API
```

If denied by an SCP, access is denied even if IAM allows it.

---

# Common SCP Examples

Restrict

- Root account usage
- Region usage
- IAM changes
- Security service deletion
- Public resource creation

---

# Deny Root User Actions

Example

```text
Root User

↓

Sensitive API

↓

Denied
```

---

# Restrict AWS Regions

Example

Allow

```text
ap-south-1

us-east-1
```

Deny

```text
All Other Regions
```

---

# Protect Security Services

Prevent disabling

- CloudTrail
- AWS Config
- GuardDuty
- Security Hub
- Inspector

---

# Restrict Public S3 Buckets

Prevent

```text
Public Bucket Policy

↓

Denied
```

---

# Restrict IAM Changes

Protect

- IAM Roles
- IAM Policies
- Administrator Accounts

---

# Deny Resource Deletion

Protect

- KMS Keys
- CloudTrail Trails
- Config Recorders
- Production Resources

---

# Delegated Administrator

Allows member accounts to manage AWS services without using the management account.

Examples

- GuardDuty
- Security Hub
- Inspector
- Firewall Manager

---

# AWS Control Tower

Purpose

Automates secure multi-account AWS environments.

Features

- Landing Zone
- Guardrails
- Account Factory
- Centralized governance

---

# Landing Zone

Provides

- Secure account structure
- Logging
- Identity
- Networking
- Governance

---

# Account Factory

Automatically creates new AWS accounts with predefined security settings.

Workflow

```text
Request

↓

Provision

↓

Apply Guardrails

↓

Account Ready
```

---

# Preventive Guardrails

Prevent actions before they occur.

Examples

- Block public S3 buckets
- Restrict Regions
- Deny root usage

---

# Detective Guardrails

Monitor and report policy violations.

Examples

- Unencrypted resources
- Disabled logging
- Public access

---

# Mandatory Guardrails

Always enforced.

Examples

- CloudTrail enabled
- Central logging
- Security monitoring

---

# Optional Guardrails

Applied based on organizational requirements.

---

# Centralized Logging

Recommended

```text
All Accounts

↓

CloudTrail

↓

Log Archive Account

↓

Amazon S3
```

---

# Centralized Security

Architecture

```text
GuardDuty

↓

Security Hub

↓

Inspector

↓

Security Account
```

---

# Account Isolation

Separate

- Production
- Development
- Security
- Networking
- Shared Services

Never mix production and development workloads.

---

# Permission Management

Recommended

```text
IAM Identity Center

↓

Permission Sets

↓

Multiple Accounts
```

---

# Enterprise Governance Workflow

```text
Create Account

↓

Assign OU

↓

Apply SCP

↓

Enable Logging

↓

Enable Monitoring

↓

Enable Security Services

↓

Production Ready
```

---

# Governance Checklist

Every account should have

- CloudTrail
- AWS Config
- GuardDuty
- Security Hub
- IAM Identity Center
- Budget
- Cost Allocation Tags
- Backup Policies

---

# Common Governance Mistakes

- Running everything in one AWS account
- No SCPs
- No centralized logging
- Shared production and development accounts
- Root account usage
- Missing guardrails
- No account ownership
- Disabled CloudTrail
- Weak IAM policies
- No governance reviews

---

# Enterprise Security Architecture

```text
Management Account

↓

Security Account

↓

Logging Account

↓

Shared Services

↓

Production OU

↓

Development OU

↓

Testing OU
```

---

# Governance Review Schedule

Daily

- Security alerts
- GuardDuty findings
- Failed log delivery

---

Weekly

- SCP changes
- New accounts
- IAM reviews
- Guardrail violations

---

Monthly

- OU review
- Account inventory
- Security audit
- Governance assessment

---

Quarterly

- Landing Zone review
- SCP optimization
- Compliance assessment
- Architecture review

---

# Best Practices

- Use multiple AWS accounts instead of a single large account.
- Organize accounts using Organizational Units (OUs).
- Apply SCPs to enforce security guardrails.
- Use AWS Control Tower for enterprise deployments.
- Separate production, development, and security accounts.
- Centralize logging in a dedicated Log Archive account.
- Centralize GuardDuty, Security Hub, and Inspector findings.
- Use IAM Identity Center with permission sets.
- Protect critical security services using SCPs.
- Regularly review governance policies and organizational structure.

---

# Summary

This section covered AWS Organizations, Organizational Units (OUs), Service Control Policies (SCPs), delegated administration, AWS Control Tower, Landing Zones, Account Factory, guardrails, centralized logging, enterprise account structures, and governance best practices. These capabilities provide the foundation for secure, scalable, and well-governed multi-account AWS environments.

---

# Data Protection

---

# Introduction

Data protection in AWS focuses on securing data throughout its lifecycle using encryption, key management, secrets management, certificates, and secure communication.

Goals

- Confidentiality
- Integrity
- Availability
- Compliance
- Secure Key Management

---

# Data States

Protect data

- At Rest
- In Transit
- In Use

---

# Encryption at Rest

Examples

- Amazon S3
- Amazon EBS
- Amazon RDS
- Amazon EFS
- Amazon DynamoDB
- Amazon OpenSearch

Recommended

```text
AWS KMS
```

---

# Encryption in Transit

Use

- HTTPS
- TLS 1.2+
- SSH
- VPN
- Direct Connect MACsec (where supported)

---

# AWS Key Management Service (KMS)

Purpose

Centralized encryption key management.

Supports

- Symmetric Keys
- Asymmetric Keys
- HMAC Keys

---

# KMS Components

- Customer Managed Keys (CMKs)
- AWS Managed Keys
- AWS Owned Keys
- Key Policies
- Grants
- Aliases

---

# Customer Managed Keys (CMKs)

Benefits

- Full control
- Key rotation
- Key policies
- Audit logging
- Cross-account access

---

# AWS Managed Keys

Characteristics

- Automatically created
- Managed by AWS
- Used by AWS services
- Limited customization

---

# AWS Owned Keys

Used internally by AWS services.

Not visible or manageable by customers.

---

# KMS Workflow

```text
Application

↓

AWS KMS

↓

Encrypt Data Key

↓

Encrypt Data

↓

Store Ciphertext
```

---

# Envelope Encryption

Workflow

```text
Plaintext

↓

Generate Data Key

↓

Encrypt Data

↓

Encrypt Data Key

↓

Store Both
```

Benefits

- High performance
- Secure key management
- Scalable encryption

---

# Data Key

Purpose

Encrypt application data.

Protected using a KMS Key.

---

# Key Policies

Control

- Who can use keys
- Who can administer keys
- Cross-account permissions

---

# IAM vs Key Policy

IAM Policy

- Controls IAM permissions

Key Policy

- Controls KMS key usage

Both may be required for access.

---

# Key Rotation

Recommended

Enable automatic rotation for Customer Managed symmetric keys.

Benefits

- Improved security
- Compliance
- Reduced key exposure

---

# Key Aliases

Example

```text
alias/production

alias/database

alias/payment
```

Avoid using Key IDs directly in applications.

---

# Key Deletion

Workflow

```text
Schedule Deletion

↓

Waiting Period

↓

Delete Key
```

Default waiting period

```text
7–30 Days
```

---

# Cross-Account KMS

Requirements

- Key Policy
- IAM Policy
- Resource Permissions

---

# AWS Secrets Manager

Purpose

Securely store and rotate secrets.

Examples

- Database Passwords
- API Keys
- OAuth Tokens
- Application Secrets

---

# Secrets Manager Workflow

```text
Application

↓

Secrets Manager

↓

Retrieve Secret

↓

Temporary Use

↓

Application
```

---

# Automatic Rotation

Supported

- Amazon RDS
- Aurora
- Custom Lambda Rotation

---

# Secret Types

Store

- Passwords
- Tokens
- Certificates
- SSH Keys
- API Credentials

---

# Secrets Manager Best Practices

- Enable automatic rotation.
- Use IAM Roles.
- Never hardcode secrets.
- Audit secret access.

---

# Systems Manager Parameter Store

Purpose

Store

- Configuration
- Secure Parameters
- Application Settings

---

# Parameter Types

- String
- StringList
- SecureString

---

# SecureString

Uses AWS KMS for encryption.

Ideal for

- Passwords
- Tokens
- Sensitive configuration

---

# Parameter Store vs Secrets Manager

| Feature | Parameter Store | Secrets Manager |
|----------|-----------------|-----------------|
| Configuration | ✅ | Limited |
| Secret Rotation | No | Yes |
| Cost | Lower | Higher |
| API Keys | Yes | Yes |
| Database Credentials | Basic | Recommended |

---

# AWS CloudHSM

Purpose

Dedicated Hardware Security Module (HSM).

Use Cases

- Regulatory compliance
- Customer-controlled cryptography
- PKI
- Certificate authorities

---

# AWS Certificate Manager (ACM)

Purpose

Manage SSL/TLS certificates.

Supports

- Public Certificates
- Private Certificates (with ACM PCA)

---

# ACM Benefits

- Automatic renewal
- AWS integration
- No certificate management overhead

---

# AWS Private CA

Use Cases

- Internal services
- Mutual TLS (mTLS)
- Enterprise PKI

---

# Secure Communication

Recommended Protocols

- HTTPS
- TLS 1.2+
- TLS 1.3
- SSH
- IPSec VPN

Avoid

- HTTP
- FTP
- Telnet
- SSLv3

---

# S3 Encryption

Options

- SSE-S3
- SSE-KMS
- SSE-C
- Client-Side Encryption

Production Recommendation

```text
SSE-KMS
```

---

# EBS Encryption

Encrypt

- Volumes
- Snapshots
- AMIs

Use KMS.

---

# RDS Encryption

Encrypt

- Database
- Snapshots
- Read Replicas

---

# DynamoDB Encryption

Enabled by default.

Supports AWS KMS.

---

# EFS Encryption

Enable

- Encryption at Rest
- Encryption in Transit

---

# Lambda Environment Variables

Store sensitive values using

- KMS encryption
- Secrets Manager
- Parameter Store

Avoid plaintext environment variables.

---

# Encryption Audit

Review

- Unencrypted storage
- Disabled key rotation
- Public certificates
- Expiring certificates
- Secret rotation status

---

# Data Protection Checklist

- Encryption enabled
- KMS key rotation enabled
- Secrets stored securely
- TLS enforced
- Certificates managed
- Key policies reviewed
- Secrets rotated
- SecureString used
- CloudTrail logging enabled
- Access monitored

---

# Common Mistakes

- Hardcoding secrets
- Sharing KMS keys unnecessarily
- Disabling key rotation
- Using HTTP
- Unencrypted S3 buckets
- Public secrets
- Plaintext passwords
- Poor key policies
- Ignoring certificate expiration
- Reusing secrets indefinitely

---

# Best Practices

- Encrypt all sensitive data at rest using AWS KMS.
- Enforce TLS for all network communications.
- Use Customer Managed Keys for production workloads.
- Enable automatic key rotation where supported.
- Store secrets in AWS Secrets Manager.
- Store application configuration in Parameter Store.
- Avoid hardcoding secrets in code or CI/CD pipelines.
- Use ACM for certificate lifecycle management.
- Audit key usage and secret access regularly.
- Review encryption posture as part of security assessments.

---

# Summary

This section covered AWS KMS, Customer Managed Keys, AWS Managed Keys, envelope encryption, Secrets Manager, Systems Manager Parameter Store, CloudHSM, ACM, encryption at rest, encryption in transit, and enterprise data protection best practices. These capabilities provide the foundation for protecting sensitive data and cryptographic assets across AWS environments.

---

# Network Security

---

# Introduction

Network security in AWS focuses on protecting workloads using layered controls, segmentation, private networking, traffic inspection, and edge protection.

Goals

- Minimize attack surface
- Restrict unauthorized access
- Secure east-west traffic
- Secure north-south traffic
- Enable Zero Trust networking

---

# Amazon VPC

Purpose

Provides logically isolated virtual networks.

Components

- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- VPC Endpoints

---

# VPC Design

Recommended

```text
Internet

↓

ALB

↓

Public Subnets

↓

Private Application Subnets

↓

Private Database Subnets
```

---

# Public vs Private Subnets

Public

- ALB
- NAT Gateway
- Bastion Host (if required)

Private

- EC2
- ECS
- EKS
- RDS
- ElastiCache

---

# Network Segmentation

Separate

```text
Production

↓

Staging

↓

Development

↓

Management
```

Never mix production workloads with development resources.

---

# Security Groups

Purpose

Stateful virtual firewall.

Characteristics

- Instance level
- Allow rules only
- Stateful
- Default deny inbound

---

# Security Group Best Practices

- Allow minimum required ports.
- Restrict SSH access.
- Use Security Group references.
- Avoid 0.0.0.0/0 where unnecessary.
- Review rules regularly.

---

# Example Security Groups

Web Tier

```text
HTTP

HTTPS
```

---

Application Tier

```text
Only ALB Security Group
```

---

Database Tier

```text
Only Application Security Group
```

---

# Network ACLs

Purpose

Subnet-level firewall.

Characteristics

- Stateless
- Allow and Deny rules
- Ordered evaluation

---

# Security Groups vs NACLs

| Feature | Security Group | Network ACL |
|----------|----------------|-------------|
| Level | Instance | Subnet |
| Stateful | Yes | No |
| Allow Rules | Yes | Yes |
| Deny Rules | No | Yes |
| Rule Order | No | Yes |

---

# Route Tables

Best Practices

- Separate public/private routing.
- Avoid unnecessary internet access.
- Use Transit Gateway for large environments.

---

# Internet Gateway

Purpose

Provides internet connectivity for public subnets.

Best Practice

Only attach to public route tables.

---

# NAT Gateway

Purpose

Allows private subnets to access the internet without accepting inbound internet traffic.

Best Practices

- Deploy in public subnets.
- Use one per Availability Zone for production.
- Reduce traffic using VPC Endpoints.

---

# VPC Endpoints

Purpose

Access AWS services privately without traversing the public internet.

Benefits

- Improved security
- Lower NAT Gateway costs
- Private connectivity

---

# Gateway Endpoints

Supports

- Amazon S3
- Amazon DynamoDB

---

# Interface Endpoints (AWS PrivateLink)

Supports

- Systems Manager
- Secrets Manager
- CloudWatch
- ECR
- KMS
- Hundreds of AWS and partner services

---

# AWS PrivateLink

Purpose

Secure private connectivity between VPCs, AWS services, and SaaS providers without exposing traffic to the public internet.

---

# Bastion Host

Purpose

Secure administrative access to private EC2 instances.

Recommended Architecture

```text
Internet

↓

Bastion Host

↓

Private EC2
```

---

# AWS Systems Manager Session Manager

Recommended Alternative

```text
Administrator

↓

IAM Authentication

↓

Session Manager

↓

Private EC2
```

Benefits

- No SSH port required
- No Bastion Host
- Full audit logging
- IAM-based access

---

# AWS Network Firewall

Purpose

Managed network firewall for VPC traffic inspection.

Features

- Stateful inspection
- Stateless inspection
- Domain filtering
- Threat prevention

---

# Firewall Policies

Examples

- Block malicious domains
- Restrict outbound internet
- Allow approved applications only

---

# AWS WAF

Purpose

Protects web applications from common web attacks.

Protects

- ALB
- API Gateway
- CloudFront
- App Runner

---

# WAF Managed Rules

Examples

- SQL Injection
- Cross-Site Scripting (XSS)
- Known Bad Inputs
- Anonymous IP Lists
- Bot Protection

---

# Rate Limiting

Example

```text
1000 Requests

↓

5 Minutes

↓

Block Client
```

---

# AWS Shield

## Shield Standard

Included automatically.

Protects against common DDoS attacks.

---

## Shield Advanced

Provides

- Advanced DDoS protection
- 24×7 DDoS Response Team (DRT)
- Cost protection
- Advanced visibility

Recommended for internet-facing production workloads.

---

# AWS Firewall Manager

Purpose

Centralized security policy management across AWS Organizations.

Manage

- WAF
- Shield
- Security Groups
- Network Firewall

---

# AWS Transit Gateway

Purpose

Simplifies connectivity between

- VPCs
- VPNs
- Direct Connect

Benefits

- Centralized routing
- Simplified architecture
- Better scalability

---

# VPN Security

Options

- Site-to-Site VPN
- Client VPN

Best Practices

- Strong encryption
- Certificate authentication
- MFA for Client VPN

---

# Zero Trust Networking

Principles

- Never trust
- Always verify
- Least privilege
- Continuous validation
- Micro-segmentation

---

# Zero Trust Workflow

```text
User

↓

Identity Verification

↓

MFA

↓

Authorization

↓

Resource Access

↓

Continuous Monitoring
```

---

# Secure Administrative Access

Recommended

```text
IAM Identity Center

↓

MFA

↓

Session Manager

↓

Private EC2
```

Avoid exposing SSH (22) and RDP (3389) to the internet.

---

# Network Monitoring

Monitor

- VPC Flow Logs
- CloudTrail
- GuardDuty
- Security Hub
- Network Firewall Logs

---

# VPC Flow Logs

Capture

- Accepted traffic
- Rejected traffic
- Network troubleshooting
- Security investigations

---

# Enterprise Network Architecture

```text
CloudFront

↓

AWS WAF

↓

Shield Advanced

↓

Application Load Balancer

↓

Private Application Subnets

↓

Private Database Subnets

↓

VPC Endpoints
```

---

# Network Security Checklist

- Private subnets for workloads
- Least-privilege Security Groups
- NACLs configured where required
- WAF protecting internet-facing applications
- Shield enabled
- Session Manager for administration
- VPC Flow Logs enabled
- Network Firewall deployed (where required)
- VPC Endpoints for AWS services
- Zero Trust principles implemented

---

# Common Network Security Mistakes

- SSH open to 0.0.0.0/0
- Public databases
- Overly permissive Security Groups
- Missing WAF
- No Flow Logs
- No VPC Endpoints
- Shared Security Groups
- Internet access from private workloads
- Ignoring east-west traffic
- No network segmentation

---

# Best Practices

- Deploy workloads in private subnets whenever possible.
- Use Security Groups with least-privilege rules.
- Use Session Manager instead of Bastion Hosts where possible.
- Protect internet-facing applications with AWS WAF.
- Enable Shield Advanced for critical production applications.
- Use VPC Endpoints to eliminate unnecessary internet traffic.
- Enable VPC Flow Logs for monitoring and investigations.
- Use AWS Firewall Manager for centralized policy enforcement.
- Segment environments using separate VPCs or subnets.
- Apply Zero Trust principles across all network access.

---

# Summary

This section covered VPC security, Security Groups, Network ACLs, VPC Endpoints, AWS PrivateLink, NAT Gateways, Bastion Hosts, Session Manager, AWS Network Firewall, AWS WAF, AWS Shield, Firewall Manager, Transit Gateway, VPN security, Zero Trust networking, VPC Flow Logs, and enterprise network security best practices. These controls form the foundation of a secure AWS networking architecture.

---

# Continuous Security Monitoring

---

# Introduction

Continuous security monitoring enables organizations to detect threats, investigate suspicious activity, automate responses, and maintain compliance across AWS environments.

Objectives

- Detect threats
- Monitor configuration changes
- Investigate incidents
- Automate response
- Improve security posture

---

# Security Monitoring Workflow

```text
AWS Resources

↓

Logs & Events

↓

Threat Detection

↓

Security Findings

↓

Investigation

↓

Response

↓

Recovery
```

---

# Amazon GuardDuty

Purpose

Intelligent threat detection service using machine learning, anomaly detection, and AWS threat intelligence.

---

# GuardDuty Data Sources

Analyzes

- CloudTrail Events
- VPC Flow Logs
- DNS Logs
- Kubernetes Audit Logs
- EKS Runtime Monitoring
- S3 Data Events
- RDS Login Activity

---

# Common GuardDuty Findings

- Cryptocurrency mining
- IAM credential compromise
- Port scanning
- SSH brute force
- EC2 malware
- Suspicious API activity
- DNS exfiltration
- S3 data exposure

---

# GuardDuty Workflow

```text
AWS Activity

↓

GuardDuty

↓

Threat Detection

↓

Security Finding

↓

Security Team
```

---

# Multi-Account GuardDuty

Recommended

```text
Management Account

↓

Security Account

↓

GuardDuty Administrator

↓

Member Accounts
```

---

# Amazon Inspector

Purpose

Automatically identifies software vulnerabilities and unintended network exposure.

---

# Inspector Supports

- Amazon EC2
- Amazon ECR
- AWS Lambda

---

# Inspector Scans

- CVEs
- Operating Systems
- Installed Packages
- Container Images
- Lambda Packages
- Network Exposure

---

# Vulnerability Workflow

```text
Resource

↓

Inspector Scan

↓

Vulnerability

↓

Prioritize

↓

Patch

↓

Rescan
```

---

# Amazon Security Hub

Purpose

Centralizes security findings from AWS services and third-party security tools.

---

# Security Hub Integrations

- GuardDuty
- Inspector
- IAM Access Analyzer
- AWS Config
- Firewall Manager
- Macie
- Partner Security Products

---

# Security Standards

Supports

- CIS AWS Foundations Benchmark
- AWS Foundational Security Best Practices
- PCI DSS
- NIST

---

# Security Hub Workflow

```text
Security Findings

↓

Security Hub

↓

Central Dashboard

↓

Investigation

↓

Remediation
```

---

# Amazon Detective

Purpose

Investigates security incidents using relationships between AWS resources and activities.

---

# Detective Analysis

Investigate

- IAM users
- EC2 instances
- API activity
- Network traffic
- Resource relationships

---

# Investigation Workflow

```text
GuardDuty Finding

↓

Detective

↓

Root Cause Analysis

↓

Affected Resources

↓

Incident Response
```

---

# AWS Config

Purpose

Tracks AWS resource configuration and evaluates compliance.

---

# AWS Config Components

- Configuration Recorder
- Delivery Channel
- Config Rules
- Conformance Packs

---

# Config Rules

Examples

- Encrypted EBS volumes
- Public S3 buckets
- Root MFA enabled
- Security Group compliance
- Required tags

---

# Conformance Packs

Examples

- CIS Benchmark
- PCI DSS
- NIST
- Operational Best Practices

---

# AWS CloudTrail

Purpose

Records AWS API activity.

---

# CloudTrail Records

- Console Logins
- CLI Commands
- SDK Calls
- IAM Changes
- Resource Changes

---

# CloudTrail Best Practices

- Enable organization trails.
- Enable log file validation.
- Store logs in dedicated logging accounts.
- Encrypt logs.
- Enable multi-region trails.

---

# CloudTrail Workflow

```text
AWS API

↓

CloudTrail

↓

Amazon S3

↓

Security Analysis
```

---

# CloudWatch Security Monitoring

Monitor

- Unauthorized API calls
- Failed logins
- IAM changes
- Network changes
- Resource deletion

---

# EventBridge Security Automation

Workflow

```text
Security Finding

↓

EventBridge

↓

Lambda

↓

SNS

↓

Security Team
```

---

# Automated Remediation

Examples

- Remove public S3 access
- Quarantine EC2 instance
- Disable IAM user
- Revoke access keys
- Block malicious IP

---

# Security Findings Lifecycle

```text
Detection

↓

Classification

↓

Investigation

↓

Containment

↓

Remediation

↓

Recovery

↓

Lessons Learned
```

---

# Centralized Logging

Collect

- CloudTrail
- VPC Flow Logs
- DNS Logs
- Application Logs
- WAF Logs
- Firewall Logs

Store in

```text
Dedicated Log Archive Account
```

---

# Security Dashboards

Display

- Critical findings
- Vulnerabilities
- Compliance score
- IAM risks
- Network threats
- Patch status
- Open incidents

---

# Security Operations Center (SOC)

Workflow

```text
Alert

↓

SOC Analyst

↓

Investigation

↓

Incident

↓

Containment

↓

Recovery
```

---

# Severity Classification

Critical

- Credential compromise
- Active malware
- Data exfiltration

---

High

- Public exposure
- Privilege escalation
- Critical vulnerabilities

---

Medium

- Configuration drift
- Missing encryption
- Policy violations

---

Low

- Informational findings
- Best practice recommendations

---

# Security Metrics

Track

- Open findings
- Critical findings
- Mean Time to Detect (MTTD)
- Mean Time to Respond (MTTR)
- Patch compliance
- Vulnerability count
- Compliance score
- Incident count

---

# Daily Security Review

Review

- GuardDuty findings
- Security Hub findings
- CloudTrail events
- Failed logins
- IAM changes
- Public resources

---

# Weekly Security Review

Review

- Inspector vulnerabilities
- Config compliance
- WAF activity
- Firewall logs
- Network exposure

---

# Monthly Security Review

Review

- Compliance reports
- Incident trends
- Threat intelligence
- Security posture
- Policy updates

---

# Common Monitoring Mistakes

- CloudTrail disabled
- No centralized logging
- Ignoring GuardDuty findings
- Delayed vulnerability patching
- No Config Rules
- Missing Security Hub
- No automated remediation
- Unreviewed IAM changes
- Missing log retention
- No incident metrics

---

# Best Practices

- Enable GuardDuty organization-wide.
- Centralize findings using Security Hub.
- Continuously scan workloads using Amazon Inspector.
- Enable AWS Config with Conformance Packs.
- Enable organization-wide CloudTrail.
- Store logs in a dedicated Log Archive account.
- Automate remediation using EventBridge and Lambda.
- Review security findings daily.
- Track security KPIs and incident response metrics.
- Integrate security monitoring into enterprise SOC processes.

---

# Summary

This section covered Amazon GuardDuty, Amazon Inspector, AWS Security Hub, Amazon Detective, AWS Config, AWS CloudTrail, CloudWatch security monitoring, EventBridge automation, centralized logging, vulnerability management, and continuous security monitoring. Together, these services provide comprehensive threat detection, compliance monitoring, and enterprise security operations across AWS environments.

---

# DevSecOps

---

# Introduction

DevSecOps integrates security into every phase of the Software Development Lifecycle (SDLC), ensuring security is automated and continuously enforced throughout development, testing, deployment, and operations.

Goals

- Shift security left
- Automate security testing
- Secure software supply chain
- Continuous compliance
- Rapid vulnerability remediation

---

# DevSecOps Lifecycle

```text
Plan

↓

Code

↓

Build

↓

Test

↓

Scan

↓

Deploy

↓

Monitor

↓

Respond
```

---

# Secure SDLC

Security Activities

- Threat Modeling
- Secure Coding
- Code Review
- Dependency Scanning
- Container Scanning
- Infrastructure Scanning
- Runtime Monitoring

---

# CI/CD Security Pipeline

```text
Developer

↓

Git Push

↓

Build

↓

SAST

↓

Dependency Scan

↓

Container Scan

↓

Infrastructure Scan

↓

Approval

↓

Deploy

↓

Runtime Monitoring
```

---

# Source Code Security

Best Practices

- Branch protection
- Mandatory pull requests
- Code reviews
- Signed commits
- Secret scanning
- Repository permissions

---

# Jenkins Security

Best Practices

- Enable authentication
- Use Role-Based Authorization
- Disable anonymous access
- Secure agent communication
- Store credentials securely
- Update plugins regularly

---

# Jenkins Credentials

Store

- AWS Credentials
- Git Credentials
- SSH Keys
- API Tokens
- Database Passwords

Never hardcode credentials inside Jenkinsfiles.

---

# GitHub Actions Security

Best Practices

- Use GitHub Secrets
- Pin action versions
- Restrict repository permissions
- Protect environments
- Require approvals for production deployments

---

# GitLab CI/CD Security

Best Practices

- Protected branches
- Protected variables
- Secret masking
- Environment approvals
- Pipeline permissions

---

# Secrets Management

Preferred Sources

```text
AWS Secrets Manager

↓

Systems Manager Parameter Store

↓

IAM Roles
```

Avoid

- Hardcoded passwords
- Plaintext configuration files
- Secrets stored in Git repositories

---

# Static Application Security Testing (SAST)

Purpose

Analyze source code for vulnerabilities before deployment.

Examples

- SQL Injection
- Command Injection
- Hardcoded Secrets
- Insecure APIs

---

# Software Composition Analysis (SCA)

Purpose

Identify vulnerable third-party libraries and dependencies.

Checks

- Known CVEs
- License compliance
- Outdated packages

---

# Dynamic Application Security Testing (DAST)

Purpose

Test running applications for security vulnerabilities.

Examples

- Authentication flaws
- Session issues
- Input validation
- Cross-Site Scripting

---

# Infrastructure as Code (IaC) Security

Scan

- CloudFormation
- Terraform
- Kubernetes Manifests

Checks

- Public resources
- Missing encryption
- Weak IAM policies
- Security Group exposure

---

# Amazon Elastic Container Registry (ECR)

Security Features

- Private repositories
- Image scanning
- IAM integration
- Encryption
- Lifecycle policies

---

# Container Image Scanning

Scan for

- Operating System vulnerabilities
- Application vulnerabilities
- Known CVEs
- Malware indicators

Workflow

```text
Build Image

↓

Push to ECR

↓

Image Scan

↓

Review Findings

↓

Deploy
```

---

# Container Image Best Practices

- Use minimal base images.
- Keep images updated.
- Remove unnecessary packages.
- Avoid running as root.
- Sign container images where applicable.

---

# Kubernetes (Amazon EKS) Security

Security Areas

- Cluster
- Nodes
- Pods
- Networking
- IAM
- Secrets

---

# IAM Roles for Service Accounts (IRSA)

Purpose

Assign IAM permissions directly to Kubernetes Service Accounts.

Benefits

- Least privilege
- No static AWS credentials
- Pod-level permissions

Workflow

```text
Pod

↓

Service Account

↓

IAM Role

↓

AWS API
```

---

# Kubernetes RBAC

Use Role-Based Access Control to restrict

- Users
- Groups
- Service Accounts

Follow least-privilege principles.

---

# Pod Security

Recommendations

- Non-root containers
- Read-only root filesystem
- Drop unnecessary Linux capabilities
- Resource limits
- Liveness and readiness probes

---

# Kubernetes Network Policies

Purpose

Control communication between Pods.

Benefits

- Micro-segmentation
- East-west traffic control
- Reduced attack surface

---

# Admission Control

Examples

- Image validation
- Security policy enforcement
- Namespace restrictions
- Label validation

---

# Runtime Security

Monitor

- Unexpected processes
- Privilege escalation
- File modifications
- Network anomalies
- Container escapes

---

# Supply Chain Security

Protect

- Source code
- Build pipeline
- Dependencies
- Container images
- Deployment artifacts

---

# Artifact Security

Store securely

- Container Images
- Build Artifacts
- Packages
- Helm Charts

Control access using IAM and repository permissions.

---

# Secure Deployment Workflow

```text
Code

↓

SAST

↓

Dependency Scan

↓

Container Scan

↓

IaC Scan

↓

Approval

↓

Production
```

---

# Security Gates

Require successful completion of

- Unit Tests
- SAST
- Dependency Scan
- Container Scan
- IaC Scan
- Policy Validation

before deployment.

---

# Runtime Monitoring

Monitor

- GuardDuty
- Security Hub
- Inspector
- CloudTrail
- CloudWatch
- EKS Audit Logs

---

# Common DevSecOps Mistakes

- Hardcoded credentials
- Skipping vulnerability scans
- Running containers as root
- Unscanned container images
- Excessive IAM permissions
- Public container registries
- Ignoring CVEs
- Unprotected CI/CD pipelines
- No security approvals
- Missing runtime monitoring

---

# DevSecOps Checklist

## Source Code

- Branch protection
- Pull request reviews
- Secret scanning

---

## CI/CD

- Secure credentials
- Signed artifacts
- Security gates
- Environment approvals

---

## Containers

- Image scanning
- Minimal images
- Non-root containers
- Image lifecycle policies

---

## Kubernetes

- IRSA
- RBAC
- Network Policies
- Pod Security
- Audit Logging

---

## Monitoring

- GuardDuty
- Inspector
- Security Hub
- CloudTrail
- CloudWatch

---

# Best Practices

- Shift security left by integrating security checks early in the SDLC.
- Automate SAST, SCA, DAST, and IaC scanning in CI/CD pipelines.
- Store secrets in AWS Secrets Manager or Parameter Store.
- Use IAM Roles for Service Accounts (IRSA) in Amazon EKS.
- Scan every container image before deployment.
- Enforce Kubernetes RBAC and Network Policies.
- Protect CI/CD pipelines with approvals and least-privilege access.
- Continuously monitor runtime environments for suspicious activity.
- Secure the software supply chain from source code to production.
- Treat security as a continuous process rather than a final deployment step.

---

# Summary

This section covered DevSecOps principles, secure SDLC, CI/CD security, Jenkins, GitHub Actions, GitLab CI/CD, secrets management, SAST, SCA, DAST, Infrastructure as Code security, Amazon ECR, container image scanning, Kubernetes security, IAM Roles for Service Accounts (IRSA), runtime security, supply chain protection, and production DevSecOps best practices.

---

