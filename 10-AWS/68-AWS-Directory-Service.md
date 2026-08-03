# AWS Directory Service

---

# Introduction

AWS Directory Service is a managed service that enables organizations to integrate AWS resources with Microsoft Active Directory (AD) or use a managed directory in the AWS Cloud.

Many enterprise applications depend on Active Directory for authentication, authorization, user management, and Group Policy. AWS Directory Service eliminates the operational burden of deploying and maintaining directory infrastructure while allowing seamless integration with AWS services and on-premises environments.

AWS Directory Service integrates with

- Amazon EC2
- Amazon WorkSpaces
- Amazon WorkDocs
- Amazon WorkMail
- Amazon FSx for Windows File Server
- AWS IAM Identity Center
- AWS Organizations
- AWS Client VPN
- Amazon RDS for SQL Server

It provides secure, highly available directory services for enterprise workloads.

---

# What is AWS Directory Service?

AWS Directory Service provides managed directory services for authentication and identity management.

It helps organizations

- Centralize Authentication
- Manage Users and Groups
- Integrate with Active Directory
- Simplify Identity Management
- Support Hybrid Environments

Workflow

```text
Users

↓

AWS Directory Service

↓

Authentication

↓

AWS Resources

↓

Access Granted
```

---

# Why AWS Directory Service?

Without Directory Service

```text
Deploy Active Directory

↓

Manage Domain Controllers

↓

Handle Patching

↓

High Operational Overhead
```

Problems

- Manual AD Management
- High Availability Challenges
- Patching
- Backup Management
- Infrastructure Maintenance

With AWS Directory Service

```text
Managed Directory

↓

Automatic Availability

↓

Integrated Authentication

↓

Simplified Operations
```

---

# Real World Problem Statement

A multinational company has

- 12,000 Employees
- Windows File Servers
- Amazon WorkSpaces
- Hybrid Infrastructure
- Active Directory Authentication

Requirements

- Single Sign-On
- Central User Management
- Hybrid Authentication
- High Availability

AWS Directory Service provides managed directory infrastructure.

---

# Enterprise Architecture

```text
Employees

        │

Authentication

        │

        ▼

AWS Directory Service

        │

────────┼─────────────

│        │            │

EC2   WorkSpaces   FSx Windows

        │

On-Premises AD (Optional)
```

---

# Directory Types

AWS Directory Service supports

- AWS Managed Microsoft AD
- AD Connector
- Simple AD

Each option serves different enterprise requirements.

---

# AWS Managed Microsoft AD

A fully managed Microsoft Active Directory.

Features

- Domain Controllers Managed by AWS
- Multi-AZ Deployment
- Group Policy Support
- Kerberos Authentication
- LDAP Support
- Trust Relationships

Suitable for

- Enterprise Workloads
- Windows Applications
- Hybrid Environments

---

# AD Connector

AD Connector acts as a proxy between AWS and an existing on-premises Active Directory.

Workflow

```text
Users

↓

AWS Resources

↓

AD Connector

↓

On-Premises Active Directory
```

Benefits

- No Directory Replication
- Existing Credentials
- Hybrid Authentication

---

# Simple AD

Simple AD is a lightweight directory based on Samba.

Suitable for

- Small Businesses
- Development Environments
- Basic Authentication

Limitations

- Fewer Enterprise Features
- No Advanced AD Capabilities

---

# Multi-AZ Deployment

AWS Directory Service automatically deploys directory infrastructure across multiple Availability Zones.

Benefits

- High Availability
- Fault Tolerance
- Automatic Recovery

---

# Trust Relationships

Managed Microsoft AD supports trust relationships.

Examples

```text
AWS Managed AD

↓

Trust

↓

Corporate Active Directory
```

Benefits

- Cross-Domain Authentication
- Resource Sharing
- Hybrid Identity

---

# Group Policy

Managed Microsoft AD supports Microsoft Group Policy Objects (GPOs).

Examples

- Password Policies
- Desktop Configuration
- Security Policies
- Login Scripts

---

# LDAP Support

Directory Service supports LDAP for authentication and directory queries.

Used by

- Enterprise Applications
- Linux Systems
- Identity Solutions

---

# Kerberos Authentication

Managed Microsoft AD supports Kerberos authentication.

Benefits

- Secure Authentication
- Mutual Authentication
- Enterprise Integration

---

# AWS Service Integrations

Directory Service integrates with

- Amazon WorkSpaces
- Amazon WorkDocs
- Amazon WorkMail
- Amazon FSx
- Amazon EC2
- AWS IAM Identity Center
- Client VPN

---

# Hybrid Identity

Architecture

```text
Corporate Users

↓

On-Premises Active Directory

↓

AD Connector

↓

AWS Applications
```

Provides seamless hybrid authentication.

---

# Security Features

Directory Service supports

- Multi-AZ Deployment
- Encryption
- IAM Integration
- CloudTrail Logging
- Security Groups
- Automated Monitoring

---

# AWS CLI

List Directories

```bash
aws ds describe-directories
```

Create Directory

```bash
aws ds create-microsoft-ad
```

Describe Trusts

```bash
aws ds describe-trusts
```

---

# Terraform

```hcl
resource "aws_directory_service_directory" "corp" {

  name     = "corp.example.com"

  password = "ExamplePassword123!"

  type     = "MicrosoftAD"

}
```

---

# CloudFormation

```yaml
Resources:

  Directory:

    Type: AWS::DirectoryService::MicrosoftAD
```

---

# Python (Boto3)

```python
import boto3

ds = boto3.client("ds")

response = ds.describe_directories()

print(response)
```

---

# Enterprise Production Architecture

```text
           Corporate Users

                 │

        AWS Directory Service

                 │

 Managed Microsoft AD

                 │

 ┌──────────────┼──────────────┐

 │              │              │

EC2        Amazon FSx     WorkSpaces

                 │

       IAM Identity Center

                 │

       Hybrid Authentication
```

---

# Best Practices

- Use Managed Microsoft AD for enterprise environments
- Deploy directories across multiple Availability Zones
- Integrate with IAM Identity Center
- Use AD Connector for hybrid authentication
- Enable CloudTrail auditing
- Follow least-privilege IAM permissions
- Monitor directory health
- Regularly back up critical configurations
- Secure directory access with Security Groups
- Enable MFA where applicable
- Review trust relationships regularly
- Patch connected Windows systems

---

# Common Mistakes

- Choosing Simple AD for enterprise workloads
- Deploying only one domain controller
- Weak administrator passwords
- No directory monitoring
- Missing CloudTrail logging
- Poor trust relationship management
- Overly permissive Security Groups
- Ignoring backup requirements
- No disaster recovery planning
- Using local accounts instead of centralized authentication

---

# Troubleshooting

## Users Cannot Authenticate

Check

- Directory Status
- Network Connectivity
- DNS Configuration
- User Credentials

---

## Trust Relationship Failed

Verify

- DNS Resolution
- Network Connectivity
- Trust Configuration
- Firewall Rules

---

## WorkSpaces Login Failed

Check

- Directory Registration
- User Assignment
- Group Membership
- Directory Health

---

## AD Connector Cannot Reach On-Premises AD

Verify

- VPN or Direct Connect
- DNS Resolution
- Firewall Rules
- Domain Controller Availability

---

## Directory Unavailable

Check

- AWS Region
- Multi-AZ Health
- Security Groups
- Service Status

---

# Interview Questions

## Basic

1. What is AWS Directory Service?
2. Why use Directory Service?
3. What are the three directory types?
4. What is AWS Managed Microsoft AD?
5. What is AD Connector?
6. What is Simple AD?
7. What is LDAP?
8. What is Kerberos?
9. What are Group Policies?
10. Which AWS services integrate with Directory Service?

---

## Intermediate

11. Explain Managed Microsoft AD architecture.
12. Explain AD Connector.
13. Explain trust relationships.
14. Explain Multi-AZ deployment.
15. Explain LDAP authentication.
16. Explain Kerberos authentication.
17. Explain hybrid identity.
18. Explain WorkSpaces integration.
19. Explain FSx integration.
20. Explain enterprise directory management.

---

## Advanced

21. Design enterprise authentication using AWS Directory Service.
22. Explain Managed Microsoft AD vs AD Connector.
23. Design hybrid Active Directory architecture.
24. Explain trust relationships across organizations.
25. Design secure directory services.
26. Explain operational best practices.
27. Design high availability directory infrastructure.
28. Explain identity governance.
29. Design enterprise hybrid authentication.
30. Best practices for AWS Directory Service.

---

# Production Scenarios

### Scenario 1

A company wants to migrate Windows applications to AWS while continuing to use its existing on-premises Active Directory.

Which Directory Service option would you recommend and why?

---

### Scenario 2

An enterprise deploys Amazon WorkSpaces for 5,000 employees.

How does AWS Directory Service simplify authentication?

---

### Scenario 3

A financial institution requires a fully managed Microsoft Active Directory with Group Policy support.

Which AWS directory option best meets this requirement?

---

### Scenario 4

Your organization needs users to authenticate across both on-premises and AWS environments.

How would AD Connector support this architecture?

---

### Scenario 5

A company requires secure authentication between two Active Directory forests.

How do trust relationships help satisfy this requirement?

---

### Scenario 6

An organization wants centralized authentication for Amazon EC2, Amazon FSx, Amazon WorkSpaces, and AWS Client VPN.

How would AWS Directory Service integrate with these services?

---

# Cheat Sheet

| Directory Type | Purpose |
|----------------|---------|
| AWS Managed Microsoft AD | Fully Managed Enterprise Active Directory |
| AD Connector | Proxy to On-Premises Active Directory |
| Simple AD | Lightweight Directory |
| LDAP | Directory Queries |
| Kerberos | Secure Authentication |
| Group Policy | Centralized Policy Management |
| Trust Relationship | Cross-Domain Authentication |
| IAM Identity Center | Enterprise SSO |
| Amazon WorkSpaces | Desktop Authentication |
| Amazon FSx | Windows File Authentication |

---

# Summary

AWS Directory Service is a managed identity service that enables organizations to deploy Microsoft Active Directory in AWS or integrate existing on-premises Active Directory environments with AWS resources. Through AWS Managed Microsoft AD, AD Connector, and Simple AD, organizations can provide centralized authentication, Group Policy management, LDAP and Kerberos support, hybrid identity, and seamless integration with services such as Amazon WorkSpaces, Amazon FSx, AWS IAM Identity Center, and AWS Client VPN, while reducing the operational burden of managing directory infrastructure.