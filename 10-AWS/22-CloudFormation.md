# AWS CloudFormation

---

# Introduction

AWS CloudFormation is an Infrastructure as Code (IaC) service that enables you to provision, manage, update, and delete AWS infrastructure using code instead of manual operations.

Rather than creating resources one by one through the AWS Management Console, CloudFormation allows you to define your entire infrastructure in JSON or YAML templates.

Everything becomes:

- Version Controlled
- Repeatable
- Automated
- Auditable
- Consistent

CloudFormation is one of the core DevOps services on AWS and is widely used for automating infrastructure deployment.

It integrates with almost every AWS service including:

- Amazon EC2
- Amazon VPC
- IAM
- Amazon S3
- Amazon RDS
- Amazon EKS
- Amazon ECS
- Lambda
- API Gateway
- CloudFront
- Route53
- AWS Secrets Manager
- AWS Systems Manager

---

# What is CloudFormation?

CloudFormation is AWS's native Infrastructure as Code service.

Instead of manually creating infrastructure,

you define it as code.

Example

Traditional Deployment

```text
Console

↓

Create VPC

↓

Create Subnets

↓

Create EC2

↓

Create Security Groups

↓

Create Load Balancer
```

CloudFormation

```text
Template

↓

CloudFormation

↓

Entire Infrastructure
```

---

# Why CloudFormation?

Imagine deploying the same application across

- Development
- Testing
- Staging
- Production

Creating resources manually causes

- Configuration Drift
- Human Errors
- Inconsistent Infrastructure
- Slow Deployments

CloudFormation solves these issues.

---

# Real World Problem

A DevOps team needs to deploy

- VPC
- Public Subnets
- Private Subnets
- NAT Gateway
- Internet Gateway
- Route Tables
- EC2
- EKS
- RDS
- ALB
- IAM Roles

Every week.

Instead of manual creation,

one CloudFormation template deploys everything automatically.

---

# Enterprise Architecture

```text
Git Repository

↓

CloudFormation Template

↓

CloudFormation

↓

AWS Resources

↓

Running Infrastructure
```

---

# Infrastructure as Code (IaC)

Infrastructure becomes source code.

Benefits

- Version Control
- Code Review
- Reusability
- Automation
- Disaster Recovery

---

# Template

A Template is the blueprint of infrastructure.

Supported formats

- YAML
- JSON

Most organizations prefer YAML because it is easier to read.

---

# Stack

A Stack is a deployed CloudFormation template.

Example

```text
Template

↓

Stack

↓

AWS Resources
```

Deleting the stack removes associated resources unless protected.

---

# Stack Lifecycle

```text
Create Stack

↓

Update Stack

↓

Delete Stack
```

CloudFormation manages the complete lifecycle.

---

# Template Structure

A CloudFormation template typically contains

- AWSTemplateFormatVersion
- Description
- Parameters
- Mappings
- Conditions
- Resources
- Outputs

---

# Resources

Resources define AWS infrastructure.

Example

```yaml
Resources:

  EC2Instance:

    Type: AWS::EC2::Instance
```

Every AWS service has a corresponding resource type.

---

# Parameters

Parameters allow reusable templates.

Example

```text
Environment

↓

Development

Production

Testing
```

Instead of modifying templates,

users provide values during deployment.

---

# Parameter Types

Examples

- String
- Number
- List
- AWS Resource Types

Example

```yaml
Parameters:

  InstanceType

  KeyPair

  VpcId
```

---

# Mappings

Mappings define fixed values.

Example

```text
Region

↓

AMI ID
```

Different Regions can use different AMIs.

---

# Conditions

Conditions create resources only when required.

Example

```text
Environment

↓

Production

↓

Create NAT Gateway

Development

↓

Skip NAT Gateway
```

Reduces infrastructure cost.

---

# Outputs

Outputs return useful information.

Examples

- VPC ID
- Subnet IDs
- Load Balancer DNS
- Security Group ID
- Instance ID

Applications and other stacks can consume these values.

---

# Pseudo Parameters

CloudFormation provides built-in variables.

Examples

```text
AWS::Region

AWS::AccountId

AWS::StackName

AWS::Partition
```

Useful for reusable templates.

---

# Intrinsic Functions

CloudFormation provides built-in functions.

Common Functions

- Ref
- Fn::GetAtt
- Fn::Sub
- Fn::Join
- Fn::ImportValue
- Fn::If
- Fn::FindInMap
- Fn::Select

These functions create dynamic templates.

---

# Ref

Returns

- Resource ID
- Parameter Value

Example

```yaml
Ref: VpcId
```

---

# GetAtt

Returns resource attributes.

Example

```yaml
Fn::GetAtt

↓

LoadBalancer

↓

DNSName
```

---

# Sub

Performs string substitution.

Example

```yaml
arn:aws:s3:::${BucketName}
```

Useful for ARNs and resource names.

---

# Join

Combines multiple strings.

Example

```text
Production

+

Application

↓

production-application
```

---

# ImportValue

Imports outputs from another stack.

Useful for

- Shared VPC
- Shared IAM
- Shared Networking

---

# Nested Stacks

Large infrastructures are divided into multiple templates.

Example

```text
Main Stack

├── VPC Stack

├── Security Stack

├── Compute Stack

├── Database Stack

└── Monitoring Stack
```

Benefits

- Better Organization
- Reusability
- Easier Maintenance

---

# Change Sets

Change Sets preview infrastructure changes before deployment.

Workflow

```text
Modify Template

↓

Create Change Set

↓

Review Changes

↓

Execute
```

Helps avoid accidental changes.

---

# Drift Detection

CloudFormation detects manual changes.

Example

```text
CloudFormation

↓

Expected

↓

EC2 Type = t3.medium

--------------------

Console

↓

Changed

↓

t3.large

↓

Drift Detected
```

Useful for governance.

---

# Stack Policies

Stack Policies protect critical resources.

Example

Prevent accidental deletion of

- Production RDS
- Production VPC
- IAM Roles

---

# Rollback

If deployment fails,

CloudFormation automatically rolls back changes.

Example

```text
Deployment

↓

Failure

↓

Rollback

↓

Previous Working State
```

Improves deployment reliability.

---

# Summary

AWS CloudFormation enables Infrastructure as Code by allowing organizations to define, provision, update, and manage AWS resources using declarative templates. Concepts such as Templates, Stacks, Parameters, Resources, Outputs, Intrinsic Functions, Nested Stacks, Change Sets, Drift Detection, and Rollback provide the foundation for building repeatable, automated, and production-ready cloud infrastructure.

---

# AWS CLI

Validate Template

```bash
aws cloudformation validate-template \
--template-body file://template.yaml
```

Create Stack

```bash
aws cloudformation create-stack \
--stack-name production-stack \
--template-body file://template.yaml
```

Update Stack

```bash
aws cloudformation update-stack \
--stack-name production-stack \
--template-body file://template.yaml
```

Delete Stack

```bash
aws cloudformation delete-stack \
--stack-name production-stack
```

Describe Stack

```bash
aws cloudformation describe-stacks \
--stack-name production-stack
```

List Stacks

```bash
aws cloudformation list-stacks
```

Detect Stack Drift

```bash
aws cloudformation detect-stack-drift \
--stack-name production-stack
```

---

# StackSets

CloudFormation StackSets allow administrators to deploy the same CloudFormation stack across multiple AWS Accounts and multiple AWS Regions from a single template.

Instead of deploying the template manually into every account, StackSets automatically distribute the stack.

Architecture

```text
Management Account

        │

CloudFormation StackSet

        │

 ┌──────┼───────────────┐

 │      │               │

Account A      Account B      Account C

 │              │               │

VPC           IAM Roles      CloudTrail

EKS           Config         GuardDuty

S3            Security       CloudWatch
```

Advantages

- Centralized deployments
- Multi-account management
- Multi-region deployment
- Consistent infrastructure
- Organization-wide governance

Common Use Cases

- IAM Roles
- AWS Config
- CloudTrail
- Security Hub
- GuardDuty
- Organization SCP Resources

---

# Nested Stacks

Large CloudFormation templates become difficult to maintain.

Nested Stacks divide infrastructure into reusable templates.

Example

```text
Main Stack

├── Network Stack

├── Security Stack

├── Compute Stack

├── Database Stack

├── Monitoring Stack

└── Application Stack
```

Benefits

- Better organization
- Easier maintenance
- Faster deployments
- Team collaboration
- Reusable infrastructure

---

# Cross Stack References

CloudFormation allows stacks to share resources.

Example

Network Stack

Outputs

```text
VPC ID

Subnet IDs

Security Groups
```

Application Stack

Imports

```text
VPC ID

Subnet IDs

Security Groups
```

Workflow

```text
Stack A

↓

Outputs

↓

Export

↓

Stack B

↓

ImportValue
```

---

# Modules

CloudFormation Modules package commonly used infrastructure.

Instead of repeatedly writing

- EC2
- IAM
- Security Groups
- CloudWatch

they become reusable modules.

Example

```text
Web Server Module

↓

EC2

IAM

Security Group

CloudWatch

↓

Reusable Component
```

Benefits

- Reusability
- Standardization
- Reduced Development Time

---

# CloudFormation Registry

Registry extends CloudFormation functionality.

Supports

- Third-party Resources
- Custom Resources
- Modules
- Extensions

Useful when AWS doesn't natively support a particular resource.

---

# Custom Resources

Sometimes CloudFormation cannot perform a required operation.

Custom Resources solve this using Lambda.

Architecture

```text
CloudFormation

↓

Lambda Function

↓

Custom Logic

↓

AWS Resource
```

Example

Automatically creating users inside an application after deployment.

---

# CloudFormation Macros

Macros preprocess templates before deployment.

Workflow

```text
CloudFormation Template

↓

Macro

↓

Modified Template

↓

Deployment
```

Macros help

- Reduce repetition
- Generate dynamic resources
- Simplify templates

---

# CloudFormation Hooks

Hooks validate resources before creation.

Example

```text
Create S3 Bucket

↓

Hook

↓

Encryption Enabled?

↓

YES

↓

Create Resource
```

If validation fails,

deployment stops.

Useful for governance.

---

# CloudFormation Guard

CloudFormation Guard validates templates against security rules.

Example Rules

- S3 Bucket must be encrypted
- EC2 cannot have public IP
- Root volume must be encrypted
- IAM policies cannot use "*"

Workflow

```text
Template

↓

CloudFormation Guard

↓

Validation

↓

Deploy
```

Excellent for DevSecOps pipelines.

---

# Change Sets

Change Sets preview modifications before deployment.

Workflow

```text
Existing Stack

↓

Updated Template

↓

Change Set

↓

Review

↓

Execute
```

Displays

- Resources Added
- Resources Modified
- Resources Deleted

Always review Change Sets before production deployments.

---

# Drift Detection

Infrastructure sometimes changes outside CloudFormation.

Example

Administrator manually changes

```text
EC2 Instance Type

↓

t3.medium

↓

t3.large
```

CloudFormation compares

Expected State

vs

Actual State

Result

```
DRIFT DETECTED
```

Benefits

- Governance
- Compliance
- Configuration Management

---

# Rollback

If deployment fails,

CloudFormation automatically restores infrastructure.

Workflow

```text
Deploy Stack

↓

Failure

↓

Rollback

↓

Previous State
```

Benefits

- Safe Deployments
- Automatic Recovery
- Reduced Downtime

---

# Deletion Policies

Deletion Policies protect important resources.

Supported Values

```text
Delete

Retain

Snapshot
```

Example

Production RDS

↓

Deletion Policy

↓

Snapshot

↓

Database Protected

---

# DependsOn

Sometimes resources must be created in sequence.

Example

```text
Internet Gateway

↓

VPC Attachment

↓

Route Table

↓

EC2 Instance
```

DependsOn enforces creation order.

---

# Wait Conditions

Wait Conditions pause deployment until external confirmation.

Useful for

- Software Installation
- Application Bootstrap
- Configuration Completion

---

# CloudFormation Designer

Provides graphical visualization.

Capabilities

- View Architecture
- Edit Templates
- Export Templates

Helpful for documentation.

---

# CloudFormation vs Terraform

| CloudFormation | Terraform |
|----------------|-----------|
| AWS Native | Multi Cloud |
| YAML / JSON | HCL |
| Stack Based | State File Based |
| AWS Managed | HashiCorp |
| Automatic Rollback | Manual Recovery |
| Deep AWS Integration | Multi Provider Support |

---

# CloudFormation vs AWS CDK

| CloudFormation | AWS CDK |
|----------------|---------|
| Declarative | Imperative |
| YAML | Python, TypeScript, Java, C#, Go |
| Manual Templates | Generated Templates |
| Resource Definitions | High-Level Constructs |

AWS CDK generates CloudFormation templates internally.

---

# Python (Boto3)

Create Stack

```python
import boto3

cf = boto3.client("cloudformation")

response = cf.create_stack(
    StackName="production-stack",
    TemplateBody=open("template.yaml").read()
)
```

Describe Stack

```python
response = cf.describe_stacks(
    StackName="production-stack"
)

print(response)
```

Delete Stack

```python
cf.delete_stack(
    StackName="production-stack"
)
```

---

# Enterprise Production Architecture

```text
                    GitHub

                      │

           CloudFormation Templates

                      │

          GitHub Actions / Jenkins

                      │

              Validate Template

                      │

            CloudFormation Guard

                      │

               Change Set Review

                      │

           CloudFormation Stack

 ┌─────────────────────────────────────────┐

 │              AWS Resources              │

 └─────────────────────────────────────────┘

 VPC

 IAM

 EC2

 ALB

 EKS

 RDS

 CloudWatch

 Route53

 S3

 CloudTrail
```

---

# Best Practices

- Store templates in Git
- Use YAML instead of JSON
- Keep templates modular
- Use Nested Stacks
- Parameterize reusable values
- Enable Rollback
- Review Change Sets
- Enable Drift Detection
- Protect production resources with Stack Policies
- Use Outputs for cross-stack communication
- Use StackSets for multi-account deployments
- Validate templates before deployment

---

# Common Mistakes

- Editing resources manually
- Ignoring Drift Detection
- Hardcoding AMI IDs
- Large monolithic templates
- Missing Parameters
- Not using Outputs
- Skipping Change Set review
- No Rollback strategy
- Not protecting production resources
- Ignoring validation errors

---

# Troubleshooting

## Stack Creation Failed

Check

- IAM Permissions
- Invalid Template
- Resource Quotas
- Existing Resource Names
- Dependencies

---

## UPDATE_ROLLBACK_FAILED

Verify

- Failed Resource
- Manual Changes
- Resource Dependencies
- Continue Rollback

---

## Stack Drift Detected

Check

- Manual Console Changes
- CLI Changes
- External Automation

---

## Resource Creation Timeout

Verify

- IAM Roles
- Network Configuration
- Dependencies
- Service Quotas

---

## Stack Cannot Be Deleted

Check

- S3 Objects
- Retain Policies
- Resource Dependencies
- Delete Protection

---

# Interview Questions

## Basic

1. What is AWS CloudFormation?
2. What is Infrastructure as Code?
3. What is a Stack?
4. What is a Template?
5. What are Parameters?
6. What are Outputs?
7. What are Resources?
8. What is Rollback?
9. What is Drift Detection?
10. What are Change Sets?

---

## Intermediate

11. Explain Nested Stacks.
12. What are StackSets?
13. Explain CloudFormation Guard.
14. Explain CloudFormation Hooks.
15. What are Modules?
16. Explain Custom Resources.
17. Explain CloudFormation Registry.
18. CloudFormation vs Terraform?
19. CloudFormation vs AWS CDK?
20. Explain Cross Stack References.

---

## Advanced

21. How would you deploy infrastructure across 100 AWS accounts?
22. Explain StackSets architecture.
23. How would you modularize enterprise infrastructure?
24. Explain Change Set workflow.
25. How would you protect production RDS from deletion?
26. Explain Drift Detection internals.
27. Design reusable CloudFormation modules.
28. Explain Rollback behavior.
29. Explain CloudFormation governance.
30. How would you integrate CloudFormation into CI/CD?

---

# Production Scenarios

### Scenario 1

A developer manually changes a Security Group in production.

How would CloudFormation identify the issue?

---

### Scenario 2

Your company has 80 AWS accounts.

How would StackSets simplify infrastructure deployment?

---

### Scenario 3

A deployment fails halfway through.

How does CloudFormation recover?

---

### Scenario 4

Security requires every S3 bucket to be encrypted before deployment.

How would CloudFormation Guard enforce this?

---

### Scenario 5

A production database must never be accidentally deleted.

How would you configure CloudFormation?

---

### Scenario 6

Your infrastructure templates exceed 8,000 lines.

How would Nested Stacks improve maintainability?

---

### Scenario 7

A stack cannot be deleted because an S3 bucket still contains objects.

How would you resolve the issue?

---

### Scenario 8

A networking stack exports a VPC ID that must be consumed by an application stack.

How would you implement this?

---

# Cheat Sheet

| Feature | Purpose |
|----------|---------|
| Template | Infrastructure Blueprint |
| Stack | Deployed Infrastructure |
| Parameters | User Inputs |
| Resources | AWS Resources |
| Outputs | Export Values |
| Nested Stack | Modular Templates |
| StackSet | Multi-Account Deployment |
| Change Set | Preview Changes |
| Drift Detection | Detect Manual Changes |
| Rollback | Automatic Recovery |
| CloudFormation Guard | Policy Validation |
| CloudFormation Hooks | Governance |
| Custom Resource | Lambda-Based Extension |
| Registry | Third-Party Resources |

---

# Summary

AWS CloudFormation is AWS's native Infrastructure as Code service that enables automated, repeatable, and consistent infrastructure provisioning through declarative templates. Enterprise features such as Nested Stacks, StackSets, Change Sets, Drift Detection, CloudFormation Guard, Hooks, Rollback, and Custom Resources make it suitable for large-scale cloud deployments. When integrated with version control, CI/CD pipelines, and governance policies, CloudFormation provides a secure, standardized, and highly maintainable approach to managing AWS infrastructure.