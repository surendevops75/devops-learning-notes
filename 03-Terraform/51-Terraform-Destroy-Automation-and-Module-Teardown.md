# Terraform Destroy Automation and Module Teardown

## Introduction

Terraform is used to:

```text
Create Infrastructure

Update Infrastructure

Delete Infrastructure
```

Most engineers focus on:

```text
terraform apply
```

but in real projects, understanding:

```text
terraform destroy
```

is equally important.

---

# Why Destroy Infrastructure?

Common reasons:

```text
Environment Cleanup

Cost Optimization

Testing

Rebuilding Infrastructure

Removing Unused Resources
```

---

# Example

Development Environment:

```text
DEV
```

is no longer required.

Instead of manually deleting resources:

```text
EC2

ALB

Databases

Security Groups

VPC
```

Terraform can remove everything automatically.

---

# Terraform Destroy

Command:

```bash
terraform destroy
```

Purpose:

```text
Delete All Resources Managed By Terraform
```

---

# Auto Approval

Command:

```bash
terraform destroy -auto-approve
```

Terraform skips:

```text
Do you really want to destroy?
```

and proceeds automatically.

---

# Multi-Module Projects

Large Terraform projects are split into modules.

Example:

```text
00-vpc

10-sg

20-bastion

30-sg-rules

40-databases

50-backend-alb

70-acm

80-frontend-alb

90-components
```

---

# Important Rule

Infrastructure must be destroyed in:

```text
Reverse Dependency Order
```

---

# Why?

Because resources depend on each other.

---

# Example

```text
VPC
   ↓
Subnets
   ↓
EC2
```

---

You cannot delete:

```text
VPC
```

while EC2 instances still exist.

AWS will return:

```text
Dependency Violation
```

---

# Correct Approach

Delete:

```text
EC2
```

first.

---

Then:

```text
Subnets
```

---

Finally:

```text
VPC
```

---

# Trainer Example

Destroy Script:

```bash
for i in 90-components/ 40-databases/; do
    cd $i
    terraform destroy -auto-approve
    cd ..
done
```

---

# Understanding the Script

## Step 1

Loop Through Modules

```bash
for i in directories
```

---

Variable:

```text
i
```

contains:

```text
90-components/

40-databases/
```

---

# Step 2

Move Into Module

```bash
cd $i
```

---

Example:

```bash
cd 90-components/
```

---

# Step 3

Destroy Resources

```bash
terraform destroy -auto-approve
```

---

Terraform removes:

```text
EC2 Instances

Target Groups

ASGs

Launch Templates
```

managed by that module.

---

# Step 4

Return To Root

```bash
cd ..
```

---

# Step 5

Repeat

Process continues for the next module.

---

# Why Components First?

Roboshop Architecture:

```text
Components
    ↓
Databases
```

---

Application Components:

```text
Catalogue

User

Cart

Shipping

Payment
```

depend on:

```text
MongoDB

Redis

MySQL

RabbitMQ
```

---

# Wrong Order

Delete:

```text
MongoDB
```

first.

---

Applications:

```text
Still Running
```

---

Result:

```text
Application Failures
```

---

# Correct Order

Delete:

```text
Application Components
```

first.

---

Then:

```text
Databases
```

---

# Dependency Flow

Creation:

```text
Databases
      ↓
Components
```

---

Destruction:

```text
Components
      ↓
Databases
```

---

# Real Roboshop Example

Creation Order:

```text
40-databases
      ↓
90-components
```

---

Destroy Order:

```text
90-components
      ↓
40-databases
```

---

# Infrastructure Lifecycle

Terraform manages:

```text
Create

Read

Update

Delete
```

---

Destroy operation uses:

```text
Terraform State File
```

to identify resources that must be removed.

---

# What Happens Internally?

Terraform reads:

```text
terraform.tfstate
```

---

Finds:

```text
Managed Resources
```

---

Calls AWS APIs:

```text
Delete EC2

Delete ASG

Delete ALB

Delete Databases
```

---

Updates:

```text
State File
```

after successful deletion.

---

# Production Considerations

Never run:

```bash
terraform destroy
```

directly in production without approval.

---

Best Practice:

```text
Review Plan

Validate Dependencies

Take Backups

Approve Change
```

---

# Common Mistakes

## Mistake 1

Destroying Parent Resources First

Example:

```text
Delete VPC
```

before:

```text
EC2
```

---

Result:

```text
Dependency Errors
```

---

## Mistake 2

Ignoring Data Backups

Destroying:

```text
Database
```

without backup.

---

Result:

```text
Permanent Data Loss
```

---

## Mistake 3

Running Destroy in Wrong Environment

Example:

```text
PROD Instead Of DEV
```

---

Result:

```text
Major Outage
```

---

# Production Example

Developer requests:

```text
Remove DEV Environment
```

---

Process:

```text
Destroy Components
      ↓
Destroy Databases
      ↓
Destroy ALBs
      ↓
Destroy Security Groups
      ↓
Destroy VPC
```

---

Result:

```text
Clean Environment Removal
```

---

# Complete Destroy Flow

```text
terraform destroy
       ↓
Read State
       ↓
Identify Resources
       ↓
Delete Dependencies
       ↓
Delete Parent Resources
       ↓
Update State
```

---

# Common Interview Questions

## What Does terraform destroy Do?

### Short Answer

It removes all resources managed by Terraform.

### Detailed Explanation

Terraform reads the state file, identifies managed resources, and deletes them using provider APIs.

### Production Example

Removing an entire DEV environment.

### Follow-Up Questions

- Does Terraform use the state file during destroy?
- Can destroy be targeted?

---

## Why Must Resources Be Destroyed in Reverse Order?

### Short Answer

Because resources have dependencies.

### Detailed Explanation

Parent resources cannot be removed while dependent resources still exist.

### Production Example

Deleting application servers before deleting databases and VPC resources.

### Follow-Up Questions

- What is a dependency violation?
- How does Terraform handle dependencies?

---

## Why Use a Bash Loop for Destroy Operations?

### Short Answer

To automate deletion of multiple Terraform modules.

### Detailed Explanation

The loop executes destroy operations module by module in the correct dependency order.

### Production Example

Destroying Roboshop components before databases.

### Follow-Up Questions

- Can the same pattern be used for apply?
- How can failures be handled?

---

## What Precautions Should Be Taken Before Destroying Infrastructure?

### Short Answer

Review dependencies, verify environment, and take backups.

### Detailed Explanation

Destroy operations can permanently remove resources and data. Proper validation is essential.

### Production Example

Backing up MySQL databases before destroying a QA environment.

### Follow-Up Questions

- Would you run destroy directly in production?
- How do you prevent accidental deletions?

---

# Key Takeaways

```text
Terraform destroy removes infrastructure managed by Terraform.

Destroy operations rely on the Terraform state file.

Resources must be deleted in reverse dependency order.

Application components should be destroyed before databases.

Bash loops help automate multi-module destruction.

Dependency violations occur when parent resources are deleted first.

Destroy operations should be carefully reviewed before execution.

Production environments require backups and approvals before teardown.

Understanding destroy workflows is important for real-world DevOps operations.
```