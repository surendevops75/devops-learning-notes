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

