# AWS ARN (Amazon Resource Name)

## Introduction

Every AWS resource must be uniquely identifiable.

Examples:

```text
EC2 Instance

S3 Bucket

IAM Role

Lambda Function

Load Balancer

Target Group
```

AWS uses:

```text
ARN
```

to uniquely identify resources.

---

# What is ARN?

ARN stands for:

```text
Amazon Resource Name
```

An ARN uniquely identifies an AWS resource.

Think of it as:

```text
Aadhaar Number

Passport Number

Employee ID
```

for AWS resources.

---

# Why Do We Need ARN?

Suppose AWS contains:

```text
Millions Of Resources
```

Examples:

```text
EC2 Instances

S3 Buckets

IAM Roles

Lambda Functions
```

AWS must uniquely identify each resource.

This is achieved through:

```text
ARN
```

---

# ARN Structure

General Format:

```text
arn:partition:service:region:account-id:resource
```

---

# Example ARN

```text
arn:aws:ec2:us-east-1:160885265516:instance/i-0a6e339f43ee8b372
```

---

# ARN Breakdown

```text
arn
 ↓
aws
 ↓
ec2
 ↓
us-east-1
 ↓
160885265516
 ↓
instance
 ↓
i-0a6e339f43ee8b372
```

---

# Component 1 - ARN Prefix

```text
arn
```

Indicates:

```text
Amazon Resource Name
```

Every ARN starts with:

```text
arn
```

---

# Component 2 - Partition

Example:

```text
aws
```

Represents:

```text
AWS Commercial Cloud
```

---

# Common Partitions

```text
aws

aws-cn

aws-us-gov
```

---

# Component 3 - Service

Example:

```text
ec2
```

Represents:

```text
Elastic Compute Cloud
```

---

# More Examples

```text
s3

iam

lambda

elasticloadbalancing

route53
```

---

# Component 4 - Region

Example:

```text
us-east-1
```

Represents:

```text
AWS Region
```

---

# Examples

```text
us-east-1

us-west-2

ap-south-1

eu-west-1
```

---

# Why Region Matters?

AWS resources often exist in specific regions.

Example:

```text
Mumbai

Virginia

Ohio
```

AWS must know:

```text
Where Resource Exists
```

---

# Component 5 - Account ID

Example:

```text
160885265516
```

Represents:

```text
AWS Account Number
```

---

# Why Account ID?

Different AWS accounts may have:

```text
Similar Resources
```

Account ID ensures uniqueness.

---

# Component 6 - Resource Type

Example:

```text
instance
```

Represents:

```text
EC2 Instance Resource Type
```

---

# Other Examples

```text
volume

snapshot

role

bucket

loadbalancer
```

---

# Component 7 - Resource ID

Example:

```text
i-0a6e339f43ee8b372
```

Represents:

```text
Actual Resource Identifier
```

---

# Complete Example

```text
arn:aws:ec2:us-east-1:160885265516:instance/i-0a6e339f43ee8b372
```

Meaning:

```text
AWS Commercial Cloud

↓

EC2 Service

↓

Virginia Region

↓

Account 160885265516

↓

EC2 Instance

↓

Instance ID i-0a6e339f43ee8b372
```

---

# ARN for EC2 Instance

Example:

```text
arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef
```

---

# ARN for IAM Role

Example:

```text
arn:aws:iam::123456789012:role/catalogue-role
```

---

# ARN for Lambda

Example:

```text
arn:aws:lambda:us-east-1:123456789012:function:catalogue
```

---

# ARN for Target Group

Example:

```text
arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/catalogue-tg/abc123
```

---

# ARN for Load Balancer

Example:

```text
arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/backend-alb/xyz123
```

---

# Where Are ARNs Used?

Common Uses:

```text
IAM Policies

Resource Permissions

Terraform

CloudFormation

Lambda Permissions

SNS

SQS
```

---

# IAM Policy Example

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances"
  ],
  "Resource": "*"
}
```

---

# Resource Specific Permission

Example:

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:StartInstances"
  ],
  "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef"
}
```

---

# Why Not Use Resource Names?

Names may:

```text
Change

Duplicate

Be Ambiguous
```

ARNs are:

```text
Unique

Reliable

Permanent Identifiers
```

---

# Terraform Example

Terraform often requires:

```text
Target Group ARN

Load Balancer ARN

IAM Role ARN
```

instead of names.

---

# Production Example

Suppose:

```text
Catalogue Target Group
```

must be attached to:

```text
Application Load Balancer
```

AWS internally uses:

```text
ARNs
```

to establish the relationship.

---

# Real Production Scenario

An IAM Role needs access to:

```text
Specific S3 Bucket
```

Instead of:

```text
Bucket Name
```

the IAM Policy references:

```text
Bucket ARN
```

for precise authorization.

---

# Common Interview Questions

## What is ARN?

### Short Answer

ARN stands for Amazon Resource Name and uniquely identifies AWS resources.

### Detailed Explanation

ARN provides a globally unique identifier for AWS resources across accounts, services, and regions.

### Production Example

An IAM policy granting access to a specific EC2 instance using its ARN.

### Follow-Up Questions

- Why does AWS use ARNs?
- Are ARNs globally unique?

---

## What Information Does an ARN Contain?

### Short Answer

ARN contains partition, service, region, account ID, resource type, and resource identifier.

### Detailed Explanation

These fields help AWS uniquely identify and locate resources.

### Production Example

EC2 Instance ARN specifying service, region, account, and instance ID.

### Follow-Up Questions

- What is the purpose of Account ID in ARN?
- Do all ARNs contain regions?

---

## Where Are ARNs Used?

### Short Answer

ARNs are used in IAM policies, Terraform, CloudFormation, and service integrations.

### Detailed Explanation

AWS services reference resources using ARNs rather than friendly names.

### Production Example

Terraform attaching a Target Group ARN to an ALB listener rule.

### Follow-Up Questions

- Can IAM policies use resource names instead of ARNs?
- Why are ARNs preferred?

---

## Difference Between Resource Name and ARN?

### Short Answer

Resource names are human-friendly; ARNs uniquely identify resources.

### Detailed Explanation

Names can change or duplicate, while ARNs remain unique and are used internally by AWS.

### Production Example

Two AWS accounts may have resources with similar names, but their ARNs are different.

### Follow-Up Questions

- Which one is used in IAM policies?
- Which one is used by AWS APIs?

---

# Key Takeaways

```text
ARN stands for Amazon Resource Name.

ARN uniquely identifies AWS resources.

Every ARN contains service, region, account ID, and resource information.

ARNs are widely used in IAM policies.

Terraform frequently uses ARNs to reference AWS resources.

Resource names are human-friendly.

ARNs are AWS-friendly unique identifiers.

Understanding ARN structure is a common AWS interview topic.

Many AWS service integrations rely on ARNs rather than names.
```