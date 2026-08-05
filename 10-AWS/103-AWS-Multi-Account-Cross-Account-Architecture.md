# AWS Multi-Account & Cross-Account Architecture

# Chapter 1 - Multi-Account Fundamentals

## Why Multiple AWS Accounts?

Small companies often start with a single AWS account.

Example

```text
AWS Account

├── Development
├── QA
└── Production
```

This works initially but becomes difficult as the organization grows.

Large enterprises use multiple AWS accounts to improve:

- Security
- Governance
- Billing
- Compliance
- Operational management
- Team ownership

Instead of one account, they organize workloads by business function or environment.

---

## Enterprise Example

```text
AWS Organization

├── Network Account
├── Security Account
├── Logging Account
├── Shared Services Account
├── Development Account
├── QA Account
├── Production Account
└── Sandbox Account
```

Each account has a specific purpose.

---

# Problems with a Single AWS Account

Imagine a company has

- 300 Developers
- 40 Teams
- 250 Applications

Using one AWS account creates problems:

- Everyone has access to everything
- Billing becomes difficult
- Security risks increase
- Resource limits are shared
- Accidental production changes
- Difficult auditing

---

# Benefits of Multi-Account Architecture

## Security

Each workload is isolated.

```text
Production

↓

Separate AWS Account

↓

Development

↓

Separate AWS Account
```

Compromising one account doesn't automatically compromise others.

---

## Billing

Separate billing for

- Development
- QA
- Production
- Business Units

Example

```text
Finance

↓

AWS Account

Marketing

↓

AWS Account

Engineering

↓

AWS Account
```

Each department tracks its own costs.

---

## Service Quotas

AWS limits resources per account.

Using multiple accounts increases scalability.

Example

Instead of

```
One EC2 quota
```

Each account receives its own quota.

---

## Compliance

Industries like banking and healthcare require strict separation.

Examples

- PCI DSS
- HIPAA
- ISO 27001

Separate accounts simplify compliance.

---

# AWS Organizations

AWS Organizations is the service used to centrally manage multiple AWS accounts.

It provides

- Central governance
- Central billing
- Account creation
- Policy enforcement
- Security controls

Architecture

```text
               AWS Organization

                      │

             Management Account

                      │

      ┌───────────────┼───────────────┐

   Production      Development      Security
```

The Management Account controls the organization.

---

# Components of AWS Organizations

| Component | Purpose |
|-----------|---------|
| Management Account | Controls organization |
| Member Account | Individual AWS Account |
| Organizational Unit (OU) | Groups accounts |
| Service Control Policy | Restricts permissions |
| Consolidated Billing | Central billing |

---

# Organization Hierarchy

```text
Management Account

│

├── Security OU

│      ├── Security Account

│      └── Logging Account

│

├── Infrastructure OU

│      ├── Network Account

│      └── Shared Services

│

└── Workloads OU

       ├── Development

       ├── QA

       └── Production
```

---

# Management Account

The Management Account

- Creates accounts
- Applies SCPs
- Manages billing
- Invites existing accounts
- Enables AWS RAM sharing

Best Practice

Never deploy applications inside the Management Account.

Use it only for administration.

---

# Member Accounts

Each workload belongs to a Member Account.

Example

```text
Production Account

↓

Amazon EKS

↓

RDS

↓

EC2
```

Development Account

```text
Amazon EKS

↓

Lambda

↓

S3
```

Isolation improves security.

---

# Organizational Units (OU)

An OU is a logical grouping of AWS accounts.

Example

```text
Production OU

├── Banking

├── Payments

└── Insurance
```

Development OU

```text
Development

QA

Sandbox
```

Policies can be applied at the OU level.

---

# Best Practices

- Separate Production from Non-Production.
- Create dedicated Security and Logging accounts.
- Keep networking centralized.
- Enable consolidated billing.
- Use Infrastructure as Code.
- Apply least privilege.

---

# Interview Questions

## Basic

- What is AWS Organizations?
- Why use multiple AWS accounts?
- What is a Management Account?

## Intermediate

- Explain Organizational Units.
- What are the benefits of consolidated billing?
- Why shouldn't applications run in the Management Account?

## Advanced

- Design AWS Organizations for a company with 150 teams.
- Explain account isolation strategy.
- Design a Landing Zone for an enterprise.

---

# Chapter 2 - Organizational Units (OU)

## What is an Organizational Unit?

An Organizational Unit (OU) is a logical container inside AWS Organizations.

It groups AWS accounts that require similar governance.

Example

```text
Production OU

├── Banking

├── Payments

└── Insurance
```

Instead of applying policies individually,

AWS applies them to the entire OU.

---

# OU Hierarchy

```text
Root

│

├── Security OU

├── Infrastructure OU

├── Production OU

├── Development OU

└── Sandbox OU
```

Every AWS Account belongs to one OU.

---

# Benefits of OUs

- Central policy management
- Easier administration
- Better governance
- Simplified compliance
- Logical organization

---

# Typical Enterprise OU Structure

```text
Root

│

├── Infrastructure

│      ├── Network

│      ├── Shared Services

│      └── DNS

│

├── Security

│      ├── GuardDuty

│      ├── Logging

│      └── Audit

│

├── Production

│

├── Non-Production

│

└── Sandbox
```

---

# Best Practices

- Separate Production and Development.
- Create dedicated Security OU.
- Create Infrastructure OU.
- Keep Sandbox isolated.
- Apply policies at OU level.

---

# Chapter 3 - Service Control Policies (SCP)

## What is an SCP?

A Service Control Policy (SCP) defines the maximum permissions available to AWS accounts inside an Organization.

Important

SCP does **not** grant permissions.

It only limits them.

IAM still determines what users can actually do.

---

# How SCP Works

```text
IAM Policy

↓

Allowed

↓

SCP

↓

Final Decision
```

If SCP denies an action,

IAM cannot override it.

---

# Example

Developer IAM Policy

```text
Allow

EC2:TerminateInstances
```

SCP

```text
Deny

EC2:TerminateInstances
```

Result

```text
Denied
```

SCP always wins.

---

# Typical Enterprise SCPs

- Deny deleting CloudTrail
- Deny disabling GuardDuty
- Deny deleting Config
- Restrict Region usage
- Restrict root account usage
- Prevent IAM privilege escalation

---

# Example Architecture

```text
Production OU

↓

SCP

↓

No EC2 Termination

↓

Production Accounts
```

Every production account automatically inherits the restriction.

---

# SCP vs IAM

| IAM Policy | SCP |
|------------|-----|
| Grants Permissions | Limits Permissions |
| User/Role Level | Organization Level |
| Applied Inside Account | Applied Across Accounts |
| Can Allow | Cannot Grant Access |

---

# Best Practices

- Apply SCPs at the OU level.
- Keep policies simple.
- Test in Development first.
- Protect security services.
- Restrict Production carefully.

---

# Interview Questions

### Basic

- What is an SCP?
- Does SCP grant permissions?

### Intermediate

- SCP vs IAM Policy
- Can IAM override SCP?

### Advanced

- Design SCPs for a banking organization.
- Restrict Production account modifications.
- Prevent developers from deleting CloudTrail.

---

# Chapter 4 - Cross-Account IAM

## What is Cross-Account IAM?

Cross-Account IAM allows users, applications, or AWS services in one AWS account to securely access resources in another AWS account without creating duplicate IAM users.

Instead of sharing passwords or long-term access keys, AWS uses **IAM Roles** and **AWS Security Token Service (STS)** to provide temporary credentials.

Example

```text
Development Account

↓

IAM Role

↓

Production Account

↓

Amazon EKS
```

The user never logs in directly to the Production account.

---

# Why Cross-Account Access?

Large organizations separate workloads into multiple AWS accounts.

Example

```text
Development Account

QA Account

Production Account

Security Account

Logging Account
```

However, teams still need controlled access.

Examples

- Jenkins deploys to Production.
- Security team audits every account.
- Backup account accesses Production S3.
- Developers read CloudWatch logs from another account.

Cross-Account IAM enables this securely.

---

# Cross-Account Architecture

```text
Account A

Developer

↓

Assume Role

↓

STS

↓

Temporary Credentials

↓

Account B

↓

AWS Resources
```

---

# Components

| Component | Purpose |
|-----------|---------|
| IAM User | Identity requesting access |
| IAM Role | Identity being assumed |
| Trust Policy | Defines who can assume the role |
| IAM Policy | Defines allowed actions |
| STS | Generates temporary credentials |

---

# How Cross-Account IAM Works

Step 1

Developer authenticates in Account A.

↓

Step 2

Developer requests AssumeRole.

↓

Step 3

STS validates permissions.

↓

Step 4

Temporary credentials are generated.

↓

Step 5

Developer accesses resources in Account B.

---

# Example

```text
Account A

↓

Developer

↓

AssumeRole

↓

Account B

↓

EC2

↓

S3

↓

RDS
```

Developer never becomes a permanent user in Account B.

---

# Benefits

- No password sharing
- Temporary credentials
- Better security
- Least privilege
- Centralized identity
- Easy auditing

---

# Trust Policy

A Trust Policy defines **who can assume the role**.

Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This allows Account A to assume the role in Account B.

---

# IAM Permission Policy

The role also needs permissions.

Example

```text
Allow

EC2

S3

CloudWatch
```

Remember

Trust Policy answers

**Who can assume the role?**

IAM Policy answers

**What can the role do?**

---

# Cross-Account Deployment Example

A Jenkins server exists inside the Shared Services Account.

Architecture

```text
Shared Services

↓

Jenkins

↓

AssumeRole

↓

Production Account

↓

Amazon EKS
```

Jenkins receives temporary credentials and deploys applications.

---

# Best Practices

- Use IAM Roles instead of IAM Users.
- Grant least privilege.
- Use temporary credentials.
- Enable CloudTrail.
- Rotate roles regularly.
- Restrict AssumeRole permissions.

---

# Interview Questions

### Basic

- What is Cross-Account IAM?
- Why use IAM Roles?

### Intermediate

- Explain Trust Policy.
- Explain IAM Permission Policy.
- Why use temporary credentials?

### Advanced

- Design secure Jenkins deployment across AWS accounts.
- Explain how STS works.
- Design access for a Security Team across 200 AWS accounts.

---

# Chapter 5 - AWS Security Token Service (STS)

## What is AWS STS?

AWS Security Token Service (STS) provides temporary security credentials for AWS resources.

Unlike IAM Users,

STS credentials expire automatically.

This significantly improves security.

---

# Why STS?

Without STS

```text
IAM User

↓

Permanent Access Keys
```

Problems

- Long-term credentials
- Difficult rotation
- Security risk

---

With STS

```text
IAM User

↓

AssumeRole

↓

STS

↓

Temporary Credentials
```

Credentials automatically expire.

---

# Temporary Credentials

STS generates

- Access Key ID
- Secret Access Key
- Session Token

Example

```text
Developer

↓

AssumeRole

↓

STS

↓

1 Hour Credentials
```

After expiration,

new credentials must be requested.

---

# Benefits

- Temporary access
- Better security
- No permanent credentials
- MFA integration
- Cross-account access
- Federation support

---

# Common STS APIs

| API | Purpose |
|------|---------|
| AssumeRole | Cross-account access |
| AssumeRoleWithSAML | Enterprise SSO |
| AssumeRoleWithWebIdentity | OIDC providers |
| GetCallerIdentity | Identify current identity |
| GetSessionToken | Temporary credentials |

---

# STS Authentication Flow

```text
IAM User

↓

AssumeRole

↓

STS

↓

Temporary Credentials

↓

AWS Service
```

---

# GetCallerIdentity

One of the most useful APIs.

Returns

- AWS Account ID
- IAM User
- IAM Role
- ARN

Useful for

- Debugging
- Automation
- CI/CD
- Terraform

---

# Production Example

GitHub Actions

↓

OIDC

↓

STS

↓

Temporary Credentials

↓

AWS

No AWS Access Keys stored in GitHub.

---

# STS vs IAM User

| IAM User | STS |
|-----------|-----|
| Permanent Credentials | Temporary Credentials |
| Manual Rotation | Automatic Expiry |
| Higher Risk | Lower Risk |
| Static Access | Session Based |

---

# Best Practices

- Prefer STS over IAM Users.
- Enable MFA.
- Keep session duration short.
- Avoid long-term access keys.
- Monitor AssumeRole events using CloudTrail.

---

# Chapter 6 - AssumeRole

## What is AssumeRole?

AssumeRole is an AWS STS operation that allows one identity to temporarily become another IAM Role.

Example

```text
Developer

↓

AssumeRole

↓

Production Role

↓

Deploy Application
```

---

# AssumeRole Flow

```text
IAM User

↓

AssumeRole

↓

STS

↓

Temporary Credentials

↓

IAM Role

↓

AWS Resources
```

---

# Production Example

CI/CD Pipeline

```text
GitHub Actions

↓

OIDC

↓

AssumeRole

↓

Production Role

↓

Amazon EKS
```

No static AWS credentials are required.

---

# Advantages

- Temporary access
- Secure deployments
- Cross-account support
- Easy auditing
- Least privilege

---

# Common Use Cases

- Cross-account deployments
- Terraform automation
- GitHub Actions
- Jenkins
- Lambda
- Security auditing
- Centralized administration

---

# Best Practices

- Grant only required permissions.
- Enable MFA for human users.
- Use OIDC where possible.
- Monitor AssumeRole activity.
- Avoid IAM Users for automation.

---

# Interview Questions

### Basic

- What is STS?
- What is AssumeRole?
- Why are temporary credentials better?

### Intermediate

- STS vs IAM User
- AssumeRole vs IAM User
- Explain Cross-Account authentication flow.

### Advanced

- Design GitHub Actions authentication using OIDC.
- Explain Cross-Account deployments.
- How would you secure CI/CD pipelines without AWS access keys?

---

# Chapter 7 - Cross-Account Resource Access

Enterprise AWS environments frequently require one AWS account to access resources that belong to another AWS account.

Examples

- Production reads container images from a Shared ECR.
- Security Account audits CloudTrail logs from all accounts.
- Backup Account stores snapshots from Production.
- Logging Account collects logs from every workload account.

Cross-account resource sharing enables this securely.

---

# Cross-Account Amazon S3

One of the most common scenarios.

Architecture

```text
Development Account

↓

IAM Role

↓

Production Account

↓

Amazon S3
```

Instead of copying data,

the application securely accesses the bucket.

---

## How It Works

Authentication

↓

AssumeRole

↓

STS

↓

Temporary Credentials

↓

Bucket Policy

↓

Amazon S3

Both IAM permissions and Bucket Policy must allow access.

---

## Required Components

- IAM Role
- Trust Policy
- IAM Permission Policy
- Bucket Policy

If any one is incorrect,

access is denied.

---

# Cross-Account Amazon ECR

Many enterprises maintain one centralized ECR.

Example

```text
Shared Services Account

↓

Amazon ECR

↓

Development

↓

QA

↓

Production
```

Instead of maintaining three repositories,

all environments pull images from one repository.

---

## Benefits

- Single Image Repository
- Consistent Deployments
- Easier Version Management
- Reduced Storage
- Better Security

---

# Cross-Account EKS Deployment

A common interview scenario.

Architecture

```text
GitHub Actions

↓

Shared Services Account

↓

AssumeRole

↓

Production Account

↓

Amazon EKS
```

Pipeline exists in one account.

Cluster exists in another.

Authentication uses STS.

---

# Cross-Account Lambda

Lambda functions may access resources in another account.

Example

```text
Lambda

↓

AssumeRole

↓

DynamoDB

↓

Different Account
```

---

# Cross-Account CloudWatch

Security teams often monitor applications across multiple accounts.

Architecture

```text
CloudWatch

↓

Monitoring Account

↓

Application Accounts
```

Benefits

- Central dashboards
- Central alarms
- Enterprise monitoring

---

# Cross-Account CloudTrail

Large organizations centralize audit logs.

```text
All Accounts

↓

CloudTrail

↓

Logging Account

↓

Amazon S3
```

Advantages

- Immutable audit trail
- Compliance
- Easier investigations

---

# Cross-Account KMS

Encrypted resources may need access across accounts.

Example

```text
Production Account

↓

Encrypted S3

↓

KMS Key

↓

Security Account
```

Key Policies must explicitly allow access.

---

# Cross-Account Secrets Manager

Shared secrets

Example

```text
Shared Services

↓

Secrets Manager

↓

Production

↓

Development
```

Only authorized roles retrieve secrets.

---

# Best Practices

- Use IAM Roles.
- Avoid IAM Users.
- Use temporary credentials.
- Restrict resource policies.
- Enable CloudTrail.
- Rotate permissions regularly.

---

# Chapter 8 - Shared Services Account

## What is a Shared Services Account?

Instead of installing common tools in every AWS account,

enterprises deploy them once inside a Shared Services Account.

Applications consume these services securely.

---

# Architecture

```text
                 Shared Services

         ┌──────────┼──────────┐

      Jenkins    Artifactory   GitLab

           │           │           │

      Prometheus   Grafana    OpenSearch

           │

       Transit Gateway

           │

 Application Accounts
```

---

# Why Shared Services?

Without Shared Services

```text
Development

↓

Own Jenkins

QA

↓

Own Jenkins

Production

↓

Own Jenkins
```

Problems

- Duplicate infrastructure
- Higher cost
- Multiple upgrades
- Operational overhead

---

With Shared Services

```text
Shared Jenkins

↓

Development

↓

QA

↓

Production
```

One installation.

Many consumers.

---

# Typical Shared Services

CI/CD

- Jenkins
- GitHub Enterprise
- GitLab
- Argo CD

Artifact Management

- Artifactory
- Harbor
- Amazon ECR

Monitoring

- Prometheus
- Grafana
- OpenSearch
- ELK

Security

- Vault
- Secrets Manager
- IAM Identity Center

Networking

- DNS
- Bastion Hosts
- Route53 Resolver

---

# Benefits

- Central management
- Standardization
- Easier upgrades
- Lower cost
- Better governance

---

# Best Practices

- Separate Shared Services from Production.
- Use Transit Gateway.
- Restrict access using IAM Roles.
- Monitor service availability.
- Backup shared platforms.

---

# Chapter 9 - Logging & Security Accounts

Enterprise AWS environments separate operational responsibilities.

Typical architecture

```text
AWS Organization

├── Network
├── Security
├── Logging
├── Shared Services
├── Production
├── Development
└── QA
```

---

# Logging Account

Purpose

Collect logs from every account.

Sources

- CloudTrail
- VPC Flow Logs
- CloudWatch Logs
- AWS Config
- Load Balancer Logs

Architecture

```text
All Accounts

↓

CloudTrail

↓

Logging Account

↓

Amazon S3

↓

OpenSearch
```

---

# Security Account

Purpose

Centralize security operations.

Services

- GuardDuty
- Security Hub
- IAM Identity Center
- AWS Config
- Inspector
- Detective

Architecture

```text
Application Accounts

↓

Security Hub

↓

Security Account
```

---

# Benefits

- Central monitoring
- Easier auditing
- Compliance
- Incident response
- Threat detection

---

# Landing Zone Architecture

A Landing Zone is a pre-configured multi-account AWS environment following AWS best practices.

Typical enterprise layout

```text
                     Root

                      │

              Management Account

                      │

     ┌────────┬────────┬────────┬────────┐

 Network   Security   Logging   Shared

                      │

             Workloads OU

       ┌────────┬────────┬────────┐

     Development   QA   Production
```

Every new AWS account follows the same standards.

---

# Production Case Study

A global fintech company operates

- 220 AWS Accounts
- 50 Development Teams
- 4 AWS Regions
- 35 Amazon EKS Clusters

Architecture

```text
AWS Organizations

↓

Management Account

↓

Infrastructure OU

↓

Network Account

↓

Transit Gateway

↓

AWS RAM

↓

Production Accounts

↓

Shared Services

↓

Logging

↓

Security
```

Authentication

- IAM Roles
- STS
- AssumeRole
- IAM Identity Center

Networking

- Transit Gateway
- Direct Connect
- PrivateLink

CI/CD

- GitHub Actions
- Jenkins
- Argo CD

Monitoring

- Prometheus
- Grafana
- OpenSearch

This architecture provides

- Enterprise security
- Central governance
- Cross-account deployments
- Shared infrastructure
- Centralized monitoring
- Simplified operations

---

# Interview Questions

## Basic

- Why use multiple AWS accounts?
- What is AWS Organizations?
- What is an Organizational Unit?
- What is AWS STS?
- What is AssumeRole?

---

## Intermediate

- SCP vs IAM Policy
- Trust Policy vs Permission Policy
- Cross-account ECR
- Cross-account S3
- Shared Services Account
- Logging Account
- Security Account

---

## Advanced

- Design AWS Organizations for 500 AWS accounts.
- Design a secure multi-account Landing Zone.
- Explain cross-account GitHub Actions deployment.
- Design centralized logging architecture.
- Explain authentication flow between Jenkins and Production.
- Design networking for a global enterprise.

---

## FAANG / Architect Questions

1. Design AWS Organizations for a multinational bank.

2. How would you isolate Production while allowing centralized deployments?

3. Design a Landing Zone for 1,000 AWS accounts.

4. Explain how STS, AssumeRole, IAM Roles, and SCP work together.

5. Design centralized logging and security for multiple AWS accounts.

6. Explain how a Shared Services Account integrates with Transit Gateway and AWS RAM.

---

# Quick Revision Cheat Sheet

| Requirement | AWS Service |
|-------------|-------------|
| Manage multiple AWS accounts | AWS Organizations |
| Group accounts | Organizational Units (OU) |
| Restrict permissions | Service Control Policy (SCP) |
| Temporary credentials | AWS STS |
| Cross-account access | AssumeRole |
| Authenticate between accounts | IAM Role |
| Share networking | AWS RAM |
| Shared CI/CD | Shared Services Account |
| Central logging | Logging Account |
| Central security | Security Account |
| Enterprise networking | Transit Gateway |
| Hybrid connectivity | Direct Connect + VPN |