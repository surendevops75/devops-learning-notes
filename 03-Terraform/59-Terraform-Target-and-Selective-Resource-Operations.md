# Terraform Target and Selective Resource Operations

## Introduction

Normally, Terraform manages infrastructure as a complete system.

Example:

```text
VPC

Subnets

Security Groups

Load Balancers

EC2 Instances

Databases
```

---

When executing:

```bash
terraform apply
```

Terraform evaluates:

```text
Entire Infrastructure
```

---

and applies all required changes.

---

# The Problem

Suppose your Terraform project contains:

```text
10 Resources
```

---

Example:

```text
VPC

Subnets

Security Groups

Bastion

ALB

Target Groups

Launch Templates

ASG

Route53

Database
```

---

You only want to create:

```text
One Resource
```

---

Instead of applying everything.

---

# Solution

Terraform provides:

```text
terraform target
```

---

# What is terraform target?

terraform target allows Terraform to:

```text
Focus On Specific Resources
```

during:

```text
Apply

Plan

Destroy
```

operations.

---

# Default Terraform Behavior

```text
Terraform Code
      ↓
Analyze Entire Dependency Graph
      ↓
Apply Changes
```

---

# Targeted Behavior

```text
Terraform Code
      ↓
Specific Resource
      ↓
Apply Changes
```

---

# Syntax

```bash
terraform apply -target=RESOURCE
```

---

Example:

```bash
terraform apply -target=aws_instance.catalogue
```

---

Result:

```text
Only Catalogue Instance
```

is processed.

---

# Resource Address

Terraform Target requires:

```text
Resource Address
```

---

Examples:

```text
aws_instance.catalogue

aws_vpc.main

aws_security_group.frontend

aws_lb.backend
```

---

# Example Scenario

Infrastructure:

```text
10 Resources
```

---

Need:

```text
Only ALB
```

---

Command:

```bash
terraform apply -target=aws_lb.frontend
```

---

Terraform focuses on:

```text
Frontend ALB
```

and required dependencies.

---

# Dependency Handling

Terraform still respects:

```text
Dependencies
```

---

Example:

```text
ALB
 ↓
Subnets
 ↓
VPC
```

---

If ALB requires:

```text
Subnets

VPC
```

Terraform may create:

```text
Required Dependencies
```

---

# Target During Planning

Syntax:

```bash
terraform plan -target=RESOURCE
```

---

Example:

```bash
terraform plan -target=aws_instance.catalogue
```

---

Purpose:

```text
Preview Changes
```

for a specific resource.

---

# Target During Destroy

Syntax:

```bash
terraform destroy -target=RESOURCE
```

---

Example:

```bash
terraform destroy -target=aws_instance.catalogue
```

---

Result:

```text
Only Catalogue Instance Destroyed
```

---

# Production Use Case 1

Suppose:

```text
Catalogue Server Failed
```

---

Need to recreate only:

```text
Catalogue Instance
```

---

Instead of:

```text
Entire Infrastructure
```

---

Use:

```bash
terraform apply -target=aws_instance.catalogue
```

---

# Production Use Case 2

Need to update:

```text
Route53 Record
```

---

No need to touch:

```text
VPC

ALB

Database
```

---

Use:

```bash
terraform apply -target=aws_route53_record.main
```

---

# Production Use Case 3

Testing a new module.

---

Example:

```text
Backend ALB
```

---

Apply only:

```text
Backend ALB Resources
```

before deploying the rest of the infrastructure.

---

# Advantages

```text
Faster Execution

Focused Changes

Troubleshooting

Testing

Partial Deployments
```

---

# Risks of terraform target

Terraform is designed to manage:

```text
Entire Infrastructure
```

---

Using target repeatedly may lead to:

```text
State Drift

Incomplete Changes

Dependency Issues
```

---

# Why Terraform Warns About Target?

Terraform documentation considers:

```text
-target
```

a special-purpose option.

---

Because it bypasses the normal:

```text
Infrastructure As A Whole
```

approach.

---

# Best Practice

Use target for:

```text
Troubleshooting

Recovery

Testing

Exceptional Situations
```

---

Avoid using it as:

```text
Daily Deployment Strategy
```

---

# Example

Suppose:

```text
VPC

Subnets

ALB

ASG

Route53
```

already exist.

---

Need only:

```text
New Security Group
```

---

Command:

```bash
terraform apply -target=aws_security_group.catalogue
```

---

Result:

```text
Focused Deployment
```

---

# Difference Between Normal Apply and Target

Normal Apply:

```text
All Resources Evaluated
```

---

Target Apply:

```text
Specific Resource Evaluated
```

---

# Architecture Example

Normal Apply

```text
Terraform
    ↓
Entire Infrastructure
```

---

Target Apply

```text
Terraform
    ↓
Specific Resource
```

---

# Real Production Scenario

Roboshop Environment:

```text
VPC

ALB

Catalogue

User

Cart

Shipping

Database
```

---

Need to update:

```text
Only Route53
```

---

Use:

```bash
terraform apply -target=aws_route53_record.roboshop
```

---

Faster than:

```bash
terraform apply
```

across the entire environment.

---

# Common Interview Questions

## What is terraform target?

### Short Answer

terraform target allows Terraform to apply, plan, or destroy specific resources.

### Detailed Explanation

It focuses Terraform operations on selected resources instead of processing the entire infrastructure.

### Production Example

Applying only a Route53 record or EC2 instance.

### Follow-Up Questions

- Does Terraform still process dependencies?
- Should target be used regularly?

---

## Why Use terraform target?

### Short Answer

To perform focused infrastructure operations.

### Detailed Explanation

Useful for troubleshooting, testing, recovery, and selective deployments.

### Production Example

Recreating a failed EC2 instance without modifying the rest of the environment.

### Follow-Up Questions

- What are the benefits?
- What are the risks?

---

## Can terraform target Be Used With Destroy?

### Short Answer

Yes.

### Detailed Explanation

Terraform can destroy specific resources using the -target option.

### Production Example

Destroying only a Catalogue EC2 instance.

### Follow-Up Questions

- Does it affect dependencies?
- Is it safe for production?

---

## Why Does Terraform Recommend Limited Use of target?

### Short Answer

Because Terraform is designed to manage infrastructure as a complete system.

### Detailed Explanation

Overusing target may cause state inconsistencies and incomplete infrastructure updates.

### Production Example

Applying changes to only one resource while missing dependent updates elsewhere.

### Follow-Up Questions

- What is state drift?
- When is target appropriate?

---

# Key Takeaways

```text
terraform target allows selective resource operations.

It can be used with apply, plan, and destroy.

Target operations focus on specific resources.

Terraform still evaluates required dependencies.

Target is useful for troubleshooting and recovery.

It should not replace normal terraform apply workflows.

Overusing target can lead to state drift and incomplete changes.

Target is best used for exceptional situations rather than routine deployments.

Understanding terraform target is important for Terraform troubleshooting and interviews.
```