# Terraform Import Deep Dive and State Reconstruction

## Introduction

In many organizations, infrastructure already exists before Terraform is introduced.

Examples:

```text
VPCs

Security Groups

EC2 Instances

Load Balancers

Route53 Records
```

These resources may have been created through:

```text
AWS Console

AWS CLI

CloudFormation

Manual Processes
```

---

# The Challenge

Terraform manages infrastructure through:

```text
Terraform Code
       ↓
Resources
       ↓
State File
```

---

When Terraform creates resources:

```text
Terraform Code
       ↓
terraform apply
       ↓
AWS Resources
       ↓
Terraform State
```

---

However, for existing resources:

```text
AWS Resources Exist

Terraform State Empty
```

---

Terraform has no knowledge of those resources.

---

# Solution

Terraform provides:

```text
terraform import
```

---

# What is Terraform Import?

Terraform Import allows Terraform to:

```text
Bring Existing Resources Under Management
```

without recreating them.

---

# Important Concept

Terraform Import does NOT:

```text
Create Resources

Modify Resources

Delete Resources
```

---

Terraform Import only:

```text
Updates State File
```

---

# Import Architecture

Before Import:

```text
AWS Resource
      ↓
Terraform State
      ↓
No Relationship
```

---

After Import:

```text
AWS Resource
      ↓
Terraform State
      ↓
Terraform Management
```

---

# Understanding the Import Process

Terraform needs:

```text
Resource Block

Unique Resource Identifier
```

---

# Import Workflow

Step 1

Create Skeleton Resource

---

Step 2

Run Import Command

---

Step 3

Terraform Fetches Resource Information

---

Step 4

Information Stored In State

---

Step 5

Complete Resource Definition

---

# Step 1 - Create Skeleton Resource

Example EC2:

```hcl
resource "aws_instance" "catalogue" {

}
```

---

No parameters required initially.

---

Purpose:

```text
Provide Resource Address
```

---

# Step 2 - Run Import

Syntax:

```bash
terraform import RESOURCE_ADDRESS RESOURCE_ID
```

---

Example:

```bash
terraform import aws_instance.catalogue i-0a6e339f43ee8b372
```

---

# Resource Address

```text
aws_instance.catalogue
```

---

Resource ID

```text
i-0a6e339f43ee8b372
```

---

# Step 3 - Terraform Reads AWS

Terraform connects to AWS.

---

Retrieves:

```text
Instance Details

Tags

Network Configuration

Attributes
```

---

# Step 4 - Store Information

Terraform stores information in:

```text
terraform.tfstate
```

---

Now Terraform understands:

```text
This EC2 Exists
```

---

# Step 5 - Complete Resource Definition

State contains:

```text
Actual Resource Configuration
```

---

Engineer reviews state information and updates Terraform code.

---

Example:

```hcl
resource "aws_instance" "catalogue" {
  ami           = "ami-xxxxx"
  instance_type = "t3.micro"
}
```

---

# Resource Identification

Every AWS resource has a unique identifier.

---

# VPC

Unique Identifier:

```text
vpc_id
```

---

Example:

```text
vpc-12345678
```

---

Import:

```bash
terraform import aws_vpc.main vpc-12345678
```

---

# Security Group

Identifiers:

```text
sg_id

Name
```

---

Example:

```text
sg-12345678
```

---

Import:

```bash
terraform import aws_security_group.catalogue sg-12345678
```

---

# EC2

Identifier:

```text
instance_id
```

---

Example:

```text
i-0a6e339f43ee8b372
```

---

Import:

```bash
terraform import aws_instance.catalogue i-0a6e339f43ee8b372
```

---

# Load Balancer

Identifier:

```text
ARN
```

---

Example:

```text
arn:aws:elasticloadbalancing:...
```

---

Import:

```bash
terraform import aws_lb.frontend arn:aws:elasticloadbalancing:...
```

---

# Route53 Records

Identifiers may include:

```text
Hosted Zone ID

Record Name
```

---

# Import Flow Diagram

```text
Create Resource Block
        ↓
Run Import
        ↓
Terraform Reads AWS
        ↓
State File Updated
        ↓
Review State
        ↓
Complete Terraform Code
```

---

# Why Not Directly Write Terraform Code?

Suppose:

```text
Production VPC
```

contains:

```text
50 Subnets

30 Route Tables

Hundreds Of Rules
```

---

Manually writing everything is difficult.

---

Import helps discover:

```text
Existing Configuration
```

---

# Migration to Infrastructure as Code

Organizations often follow:

```text
Manual Infrastructure
        ↓
Terraform Import
        ↓
Infrastructure As Code
```

---

# Route53 Migration Example

Old Infrastructure:

```text
Route53
      ↓
Old ALB
```

---

New Infrastructure:

```text
Route53
      ↓
Frontend ALB
      ↓
Backend ALB
      ↓
Components
      ↓
Databases
```

---

Instead of recreating Route53 records:

```text
Import Existing Records
```

and manage them through Terraform.

---

# Common Mistake

Import Resource

↓

Forget To Update Terraform Code

↓

Run Terraform Apply

---

Result:

```text
Unexpected Changes
```

---

# Best Practice

After Import:

```bash
terraform plan
```

---

Goal:

```text
No Changes Required
```

---

# Production Example

Existing Environment:

```text
VPC

ALB

EC2

Security Groups
```

---

Terraform Adoption:

```text
Create Skeleton Resources
        ↓
Import Resources
        ↓
Review State
        ↓
Complete Code
        ↓
Run Plan
        ↓
Manage Through Terraform
```

---

# Common Interview Questions

## What is Terraform Import?

### Short Answer

Terraform Import allows existing resources to be brought under Terraform management.

### Detailed Explanation

Import associates existing infrastructure with Terraform state without recreating resources.

### Production Example

Importing a manually created ALB into Terraform.

### Follow-Up Questions

- Does import create resources?
- Does import modify resources?

---

## What Does Terraform Import Actually Update?

### Short Answer

Terraform State.

### Detailed Explanation

Terraform stores resource metadata and associations in the state file.

### Production Example

Importing an EC2 instance into terraform.tfstate.

### Follow-Up Questions

- Why is state important?
- What happens if state is missing?

---

## Why Create a Skeleton Resource Before Import?

### Short Answer

Terraform requires a resource address during import.

### Detailed Explanation

A minimal resource block provides the address Terraform uses to map the imported resource.

### Production Example

Creating aws_instance.catalogue before importing an EC2 instance.

### Follow-Up Questions

- Can import work without a resource block?
- What is a resource address?

---

## What Should Be Done After Import?

### Short Answer

Review state and run terraform plan.

### Detailed Explanation

The Terraform code should be updated to match the actual infrastructure configuration.

### Production Example

Importing a production VPC and validating that no changes are required.

### Follow-Up Questions

- What is drift?
- How do you resolve drift?

---

# Key Takeaways

```text
Terraform Import brings existing resources under Terraform management.

Import updates the state file, not the infrastructure.

Every imported resource requires a resource block.

Unique resource identifiers are used during import.

Skeleton resource blocks are commonly created before import.

State information helps reconstruct Terraform definitions.

terraform plan should be executed after import.

Import is heavily used when adopting Infrastructure as Code.

Existing production resources can be safely migrated to Terraform using import.
```