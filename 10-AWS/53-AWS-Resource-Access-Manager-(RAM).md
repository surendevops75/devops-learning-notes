# AWS Resource Access Manager (RAM)

---

# Introduction

AWS Resource Access Manager (AWS RAM) is a fully managed service that enables organizations to securely share supported AWS resources across multiple AWS accounts, Organizational Units (OUs), or AWS Organizations without creating duplicate resources.

In enterprise environments, teams often need to share networking resources, licenses, or infrastructure components between AWS accounts. Creating duplicate resources increases operational complexity, costs, and management overhead.

AWS RAM allows centralized resource sharing while maintaining account isolation and governance.

AWS RAM integrates with

- AWS Organizations
- Amazon VPC
- AWS Transit Gateway
- Route 53 Resolver
- AWS License Manager
- AWS Resource Groups
- AWS IAM
- AWS CloudTrail
- AWS Config

AWS RAM simplifies multi-account architectures by enabling secure resource sharing.

---

# What is AWS RAM?

AWS RAM enables sharing supported AWS resources across AWS accounts.

It helps organizations

- Reduce Resource Duplication
- Simplify Multi-Account Architectures
- Centralize Shared Infrastructure
- Improve Governance
- Reduce Costs

Workflow

```text
Shared Resource

↓

AWS RAM

↓

Resource Share

↓

Member Accounts

↓

Resource Access
```

---

# Why AWS RAM?

Without AWS RAM

```text
Multiple AWS Accounts

↓

Duplicate Resources

↓

Higher Cost

↓

Operational Complexity
```

Problems

- Resource Duplication
- Higher Costs
- Inconsistent Configuration
- Difficult Management

With AWS RAM

```text
Shared Resource

↓

AWS RAM

↓

Multiple Accounts

↓

Centralized Management
```

---

# Real World Problem Statement

An enterprise manages

- 400 AWS Accounts
- Shared Networking
- Shared Transit Gateway
- Shared Route 53 Resolver
- Shared License Configurations

Requirements

- Central Resource Management
- Secure Sharing
- Reduced Costs
- Simplified Administration

AWS RAM provides centralized resource sharing.

---

# Enterprise Architecture

```text
Management Account

        │

Shared Resources

Transit Gateway

Resolver Rules

License Configurations

        │

        ▼

AWS RAM

        │

 ┌────────┼──────────┐

 │        │          │

Prod    Dev      Test Accounts
```

---

# Core Components

AWS RAM consists of

- Resource Shares
- Shared Resources
- Principals
- Resource Associations
- Invitations
- AWS Organizations Integration
- Permission Management
- Managed Permissions
- Customer Managed Permissions
- Share Monitoring

---

# Resource Share

A Resource Share groups one or more AWS resources that will be shared.

Example

```text
Resource Share

↓

Transit Gateway

Route 53 Resolver Rules

↓

Shared Accounts
```

---

# Supported Principals

Resources can be shared with

- AWS Account
- Organizational Unit (OU)
- Entire AWS Organization
- IAM Roles (for supported integrations)

---

# Shared Resources

Examples of resources supported by AWS RAM

- AWS Transit Gateway
- Route 53 Resolver Rules
- License Manager Configurations
- EC2 Capacity Reservations
- VPC Prefix Lists
- Subnets (supported scenarios)
- AWS Resource Groups

Support varies by AWS service.

---

# Resource Associations

Resource Associations connect resources to a Resource Share.

Example

```text
Transit Gateway

↓

Resource Share

↓

AWS Accounts
```

---

# Principal Associations

Principal Associations specify who receives access.

Example

```text
Production OU

↓

Resource Share

↓

Transit Gateway
```

---

# Invitations

If AWS Organizations sharing is not enabled, recipient accounts receive invitations.

Workflow

```text
Create Share

↓

Invitation Sent

↓

Recipient Accepts

↓

Access Granted
```

---

# AWS Organizations Integration

AWS RAM integrates with AWS Organizations.

Benefits

- Automatic Sharing
- No Manual Invitations
- Organization-Wide Governance
- Simplified Administration

---

# Managed Permissions

Managed Permissions define what actions recipients can perform.

Examples

- Read Only
- Full Use
- Service-Specific Permissions

Provides consistent access control.

---

# Customer Managed Permissions

Organizations can create custom permission sets for supported resources.

Supports granular governance.

---

# Monitoring

AWS RAM integrates with

- AWS CloudTrail
- AWS Config

Monitor

- Resource Shares
- Access Changes
- Invitations
- Permission Updates

---

# CloudTrail Integration

CloudTrail records

- Share Creation
- Invitation Acceptance
- Permission Changes
- Resource Associations

Useful for auditing.

---

# AWS Config Integration

AWS Config tracks

- Shared Resources
- Configuration Changes
- Compliance

Supports governance.

---

# AWS CLI

Create Resource Share

```bash
aws ram create-resource-share \
--name Shared-TransitGateway
```

List Resource Shares

```bash
aws ram get-resource-shares
```

List Shared Resources

```bash
aws ram list-resources
```

Accept Invitation

```bash
aws ram accept-resource-share-invitation
```

---

# Terraform

```hcl
resource "aws_ram_resource_share" "network" {

  name = "shared-network"

  allow_external_principals = false

}
```

Associate Resource

```hcl
resource "aws_ram_resource_association" "tgw" {

  resource_arn       = aws_ec2_transit_gateway.main.arn

  resource_share_arn = aws_ram_resource_share.network.arn

}
```

---

# CloudFormation

```yaml
Resources:

  ResourceShare:

    Type: AWS::RAM::ResourceShare

    Properties:

      Name: SharedNetwork
```

---

# Python (Boto3)

```python
import boto3

ram = boto3.client("ram")

response = ram.get_resource_shares()

print(response)
```

---

# Enterprise Production Architecture

```text
              AWS Organization

                     │

             Shared Services Account

                     │

 Transit Gateway • Resolver Rules

 License Configurations • Prefix Lists

                     │

                 AWS RAM

                     │

 ┌───────────┼────────────┐

 │           │            │

Production Development Testing

                     │

 CloudTrail • Config • IAM
```

---

# Best Practices

- Use AWS Organizations integration
- Disable external principals unless required
- Share resources from dedicated shared-services accounts
- Apply least-privilege permissions
- Review resource shares regularly
- Monitor CloudTrail logs
- Enable AWS Config
- Use Organizational Units for sharing
- Document shared resource ownership
- Periodically review permissions
- Remove unused resource shares
- Centralize networking resources

---

# Common Mistakes

- Sharing with external accounts unnecessarily
- Overly permissive managed permissions
- Duplicating resources instead of sharing
- Ignoring CloudTrail logs
- Missing AWS Config monitoring
- Poor documentation
- Sharing production resources with development
- No ownership model
- Manual invitation management
- Weak governance

---

# Troubleshooting

## Resource Not Visible

Check

- Resource Share
- Principal Association
- Invitation Status
- Supported Resource Type

---

## Invitation Not Received

Verify

- AWS Organizations Integration
- Recipient Account
- Invitation Status

---

## Access Denied

Check

- Managed Permissions
- IAM Permissions
- Resource Share Configuration

---

## Resource Association Failed

Verify

- Supported Resource
- Resource ARN
- AWS Region

---

## External Sharing Blocked

Check

- allow_external_principals Setting
- Organization Configuration

---

# Interview Questions

## Basic

1. What is AWS RAM?
2. Why use AWS RAM?
3. What is a Resource Share?
4. What is a Principal?
5. What resources can AWS RAM share?
6. What is a Resource Association?
7. What is a Principal Association?
8. What are Managed Permissions?
9. How does AWS Organizations integrate with RAM?
10. What happens if Organizations sharing is disabled?

---

## Intermediate

11. Explain Resource Shares.
12. Explain Managed Permissions.
13. Explain Customer Managed Permissions.
14. Explain AWS Config integration.
15. Explain CloudTrail integration.
16. Explain invitation workflow.
17. Explain OU-based sharing.
18. Explain shared networking architecture.
19. Explain governance using AWS RAM.
20. Explain monitoring shared resources.

---

## Advanced

21. Design enterprise shared networking using AWS RAM.
22. How would you share a Transit Gateway across 500 AWS accounts?
23. Explain AWS RAM vs VPC Peering.
24. Design centralized shared-services architecture.
25. Explain governance using AWS RAM.
26. Design secure cross-account resource sharing.
27. Explain RAM operational best practices.
28. Design multi-account Route 53 Resolver sharing.
29. Explain resource ownership strategies.
30. Best practices for enterprise AWS RAM deployments.

---

# Production Scenarios

### Scenario 1

Your networking team manages a central Transit Gateway used by hundreds of AWS accounts.

How would AWS RAM simplify this architecture?

---

### Scenario 2

A company wants every development account to use the same Route 53 Resolver rules.

How would AWS RAM implement this?

---

### Scenario 3

Your organization uses centralized Microsoft SQL Server license configurations.

How would License Manager and AWS RAM work together?

---

### Scenario 4

An external AWS account needs temporary access to a shared resource.

How would invitations and permissions control access?

---

### Scenario 5

Security teams need to audit all resource-sharing activity.

How would CloudTrail support this requirement?

---

### Scenario 6

An enterprise adopts a Shared Services account model.

Which AWS RAM features simplify resource reuse across production, development, and testing accounts?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Resource Share | Group of Shared Resources |
| Principal | AWS Account, OU, or Organization |
| Resource Association | Link Resource to Share |
| Principal Association | Grant Access to Principal |
| Managed Permissions | Predefined Access Rules |
| Customer Managed Permissions | Custom Access Rules |
| AWS Organizations | Automatic Sharing |
| CloudTrail | Audit Resource Sharing |
| AWS Config | Compliance Monitoring |
| Transit Gateway | Common Shared Resource |

---

# Summary

AWS Resource Access Manager (AWS RAM) enables secure sharing of supported AWS resources across AWS accounts, Organizational Units, and AWS Organizations. Features such as Resource Shares, Principal Associations, Managed Permissions, AWS Organizations integration, CloudTrail auditing, AWS Config monitoring, and centralized resource governance help enterprises reduce duplication, simplify multi-account architectures, and securely share networking and infrastructure resources at scale.