# AWS CloudFormation

---

# Introduction

AWS CloudFormation is an Infrastructure as Code (IaC) service that provisions and manages AWS resources using declarative templates.

Templates can be written in:

- YAML (Recommended)
- JSON

CloudFormation automatically manages:

- Resource creation
- Updates
- Rollbacks
- Dependency ordering
- Stack deletion

---

# Benefits

- Infrastructure as Code
- Repeatable deployments
- Version controlled infrastructure
- Automatic dependency resolution
- Rollback on failures
- Native AWS service

---

# CloudFormation Workflow

```text
Write Template
        ↓
Validate Template
        ↓
Create Stack
        ↓
Review Events
        ↓
Update Stack
        ↓
Delete Stack
```

---

# Template Structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: Sample Template

Metadata:

Parameters:

Mappings:

Conditions:

Resources:

Outputs:
```

---

# Minimal Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: Hello CloudFormation

Resources: {}
```

---

# Create Stack

```bash
aws cloudformation create-stack \
--stack-name production \
--template-body file://template.yaml
```

---

# Update Stack

```bash
aws cloudformation update-stack \
--stack-name production \
--template-body file://template.yaml
```

---

# Delete Stack

```bash
aws cloudformation delete-stack \
--stack-name production
```

---

# Validate Template

```bash
aws cloudformation validate-template \
--template-body file://template.yaml
```

---

# List Stacks

```bash
aws cloudformation list-stacks
```

---

# Describe Stack

```bash
aws cloudformation describe-stacks \
--stack-name production
```

---

# Describe Stack Events

```bash
aws cloudformation describe-stack-events \
--stack-name production
```

---

# List Stack Resources

```bash
aws cloudformation list-stack-resources \
--stack-name production
```

---

# Parameters

```yaml
Parameters:

  Environment:

    Type: String

    Default: dev
```

---

# Number Parameter

```yaml
Parameters:

  InstanceCount:

    Type: Number

    Default: 2
```

---

# CommaDelimitedList

```yaml
Parameters:

  Subnets:

    Type: CommaDelimitedList
```

---

# Allowed Values

```yaml
Parameters:

  Environment:

    Type: String

    AllowedValues:

      - dev

      - qa

      - prod
```

---

# Parameter Constraints

```yaml
Parameters:

  InstanceType:

    Type: String

    AllowedPattern: "^t3.*"
```

---

# NoEcho

```yaml
Parameters:

  DBPassword:

    Type: String

    NoEcho: true
```

---

# Metadata

```yaml
Metadata:

  AWS::CloudFormation::Interface:

    ParameterGroups:
      - Label:

          default: Environment
```

---

# Mappings

```yaml
Mappings:

  RegionMap:

    ap-south-1:

      AMI: ami-xxxxxxxx
```

---

# FindInMap

```yaml
ImageId:

  Fn::FindInMap:

    - RegionMap

    - !Ref AWS::Region

    - AMI
```

---

# Conditions

```yaml
Conditions:

  IsProduction:

    !Equals

      - !Ref Environment

      - prod
```

---

# Use Condition

```yaml
Condition: IsProduction
```

---

# Outputs

```yaml
Outputs:

  VpcId:

    Value: !Ref VPC
```

---

# Export Output

```yaml
Outputs:

  VpcId:

    Value: !Ref VPC

    Export:

      Name: Production-VPC
```

---

# Pseudo Parameters

```yaml
!Ref AWS::Region
```

---

```yaml
!Ref AWS::AccountId
```

---

```yaml
!Ref AWS::StackName
```

---

```yaml
!Ref AWS::Partition
```

---

```yaml
!Ref AWS::URLSuffix
```

---

# Resource Example

```yaml
Resources:

  VPC:

    Type: AWS::EC2::VPC

    Properties:

      CidrBlock: 10.0.0.0/16
```

---

# Resource Tags

```yaml
Tags:

  - Key: Name

    Value: Production
```

---

# DependsOn

```yaml
DependsOn:

  - InternetGateway
```

---

# Deletion Policy

```yaml
DeletionPolicy: Retain
```

---

# Update Replace Policy

```yaml
UpdateReplacePolicy: Retain
```

---

# Stack Policy

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*"
    }
  ]
}
```

---

# Wait Condition

```yaml
Type: AWS::CloudFormation::WaitCondition
```

---

# Wait Condition Handle

```yaml
Type: AWS::CloudFormation::WaitConditionHandle
```

---

# Resource Attributes

```yaml
!GetAtt VPC.CidrBlock
```

---

```yaml
!GetAtt Bucket.DomainName
```

---

# Resource Reference

```yaml
!Ref VPC
```

---

# Intrinsic Function

Join

```yaml
!Join

  - "-"

  - [dev, app]
```

---

Split

```yaml
!Split

  - ","

  - "a,b,c"
```

---

Select

```yaml
!Select

  - 0

  - !Ref Subnets
```

---

Sub

```yaml
!Sub

  arn:aws:s3:::${Bucket}
```

---

Base64

```yaml
!Base64 |
  #!/bin/bash
  yum update -y
```

---

# AWS-Specific Parameter

```yaml
Parameters:

  KeyPair:

    Type: AWS::EC2::KeyPair::KeyName
```

---

# VPC Parameter

```yaml
Type: AWS::EC2::VPC::Id
```

---

# Subnet Parameter

```yaml
Type: AWS::EC2::Subnet::Id
```

---

# Security Group Parameter

```yaml
Type: AWS::EC2::SecurityGroup::Id
```

---

# Best Practices

- Prefer YAML over JSON for readability.
- Use Parameters instead of hardcoded values.
- Export shared resources using Outputs.
- Use Mappings for region-specific values.
- Keep templates modular.
- Use Conditions to avoid duplicate templates.
- Validate templates before deployment.
- Tag all resources consistently.
- Protect critical resources using `DeletionPolicy: Retain`.

---

# Summary

This section covered CloudFormation fundamentals, template structure, parameters, mappings, conditions, outputs, intrinsic functions, pseudo parameters, stack operations, resource attributes, deletion policies, and best practices. These concepts form the foundation for building reusable and production-ready CloudFormation templates.

---

