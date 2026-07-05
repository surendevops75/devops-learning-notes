# Terraform Import and Existing Infrastructure Management

## Introduction

Terraform is primarily used to create and manage infrastructure.

Normally the workflow is:

```text
Write Terraform Code
        ↓
terraform apply
        ↓
Infrastructure Created
```

---

However, in many real-world environments:

```text
Infrastructure Already Exists
```

before Terraform is introduced.

Examples:

```text
EC2 Instances

VPCs

Security Groups

S3 Buckets

Load Balancers
```

created manually through:

```text
AWS Console

CLI

Other Teams
```

---

# The Problem

Suppose an EC2 instance already exists:

```text
Catalogue Server
```

---

Terraform Code:

```text
aws_instance.catalogue
```

---

If you run:

```bash
terraform apply
```

Terraform assumes:

```text
Resource Does Not Exist
```

---

Result:

```text
New EC2 Created
```

instead of managing the existing one.

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
Start Managing Existing Resources
```

without recreating them.

---

# Import Concept

Before Import:

```text
AWS Resource Exists

Terraform Does Not Know About It
```

---

After Import:

```text
AWS Resource Exists

Terraform State Knows About It
```

---

# Important Point

Import does NOT create resources.

Import does NOT modify resources.

Import only updates:

```text
Terraform State
```

---

# Terraform Import Flow

```text
Existing Resource
       ↓
terraform import
       ↓
Terraform State
       ↓
Terraform Management
```

---

# Example

Existing EC2:

```text
i-0a6e339f43ee8b372
```

---

Terraform Resource:

```hcl
resource "aws_instance" "catalogue" {
}
```

---

Import Command

```bash
terraform import aws_instance.catalogue i-0a6e339f43ee8b372
```

---

# What Happens Internally?

Terraform:

```text
Reads AWS Resource
```

↓

```text
Stores Information In State File
```

↓

```text
Begins Managing Resource
```

---

# Resource Address

Import syntax requires:

```text
Terraform Resource Address
```

Example:

```text
aws_instance.catalogue
```

---

and:

```text
Actual AWS Resource ID
```

Example:

```text
i-0a6e339f43ee8b372
```

---

# Before Import

AWS:

```text
EC2 Exists
```

---

Terraform:

```text
No Knowledge
```

---

# After Import

AWS:

```text
EC2 Exists
```

---

Terraform:

```text
Managing EC2
```

---

# Common Resources Imported

```text
EC2

VPC

Subnets

Security Groups

Route53 Records

Load Balancers

Target Groups

IAM Roles

S3 Buckets
```

---

# Real World Scenario

Company has:

```text
5 Years Old AWS Environment
```

---

Resources created manually.

---

Management decides:

```text
Move To Infrastructure As Code
```

---

Deleting everything is impossible.

---

Solution:

```text
Import Existing Infrastructure
```

into Terraform.

---

# Production Migration Strategy

Step 1

Create Terraform Code

---

Step 2

Identify Existing Resources

---

Step 3

Import Resources

---

Step 4

Verify Terraform State

---

Step 5

Run Terraform Plan

---

# Why Verify After Import?

Imported resource may contain:

```text
Configuration Differences
```

between:

```text
Actual AWS Resource

Terraform Code
```

---

# Example

AWS Security Group:

```text
Port 22

Port 80

Port 443
```

---

Terraform Code:

```text
Port 22

Port 80
```

---

Terraform Plan Shows:

```text
Drift
```

---

# What is Drift?

Difference between:

```text
Actual Infrastructure
```

and:

```text
Terraform Code
```

---

# Production Best Practice

After Import:

```text
terraform plan
```

must show:

```text
No Changes
```

---

before managing the resource.

---

# Importing ALB Example

Existing ALB:

```text
frontend-alb
```

---

Terraform Resource:

```text
aws_lb.frontend
```

---

Import:

```bash
terraform import aws_lb.frontend arn:aws:elasticloadbalancing:...
```

---

# Importing Security Group Example

Existing Security Group:

```text
sg-12345678
```

---

Command:

```bash
terraform import aws_security_group.frontend sg-12345678
```

---

# Importing VPC Example

Existing VPC:

```text
vpc-12345678
```

---

Command:

```bash
terraform import aws_vpc.main vpc-12345678
```

---

# Terraform Import Limitations

Import only updates:

```text
State File
```

---

Import does NOT generate:

```text
Terraform Code
```

---

You must still write:

```text
Resource Definitions
```

manually.

---

# Example

Import:

```bash
terraform import aws_instance.catalogue i-123456
```

---

Terraform State Updated.

---

But Terraform will NOT automatically create:

```hcl
resource "aws_instance" "catalogue" {
}
```

---

Engineer must write it.

---

# Production Example

Suppose:

```text
Frontend ALB
```

was created manually.

---

Company wants:

```text
Infrastructure As Code
```

---

Process:

```text
Write Terraform Code
        ↓
Import ALB
        ↓
Verify State
        ↓
Run Plan
        ↓
Manage Using Terraform
```

---

# Import and Multi-Account Environments

Organizations often have:

```text
DEV Account

QA Account

PROD Account
```

---

Resources may exist in all accounts.

---

Terraform Import helps:

```text
Bring Existing Resources Under Management
```

without rebuilding them.

---

# Common Interview Questions

## What is Terraform Import?

### Short Answer

Terraform Import allows Terraform to start managing existing infrastructure resources.

### Detailed Explanation

Import adds existing resources into the Terraform state file without recreating them.

### Production Example

Importing an existing ALB into Terraform.

### Follow-Up Questions

- Does import create resources?
- Does import generate code?

---

## What Does Terraform Import Actually Update?

### Short Answer

Terraform State.

### Detailed Explanation

Import associates existing infrastructure with Terraform resources in the state file.

### Production Example

Importing an EC2 instance into Terraform state.

### Follow-Up Questions

- What is stored in state?
- Why is state important?

---

## Does Terraform Import Generate Terraform Code?

### Short Answer

No.

### Detailed Explanation

Import updates state only. Engineers must still write the resource definitions manually.

### Production Example

Importing a Security Group while manually maintaining its Terraform code.

### Follow-Up Questions

- Why doesn't Terraform generate code?
- How do you validate imported resources?

---

## What Should You Do After Import?

### Short Answer

Run terraform plan.

### Detailed Explanation

Plan verifies whether Terraform code matches the actual infrastructure.

### Production Example

Checking for drift after importing a production ALB.

### Follow-Up Questions

- What is drift?
- How do you resolve drift?

---

# Key Takeaways

```text
Terraform Import allows Terraform to manage existing resources.

Import updates the Terraform state file.

Import does not create resources.

Import does not generate Terraform code.

Resource definitions must still be written manually.

Terraform Plan should be run after import.

Drift occurs when Terraform code differs from actual infrastructure.

Import is commonly used during Infrastructure as Code adoption.

Many organizations use Import when migrating manually managed AWS environments to Terraform.
```