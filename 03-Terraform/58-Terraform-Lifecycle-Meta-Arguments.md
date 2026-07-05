# Terraform Lifecycle Meta Arguments

## Introduction

Terraform normally follows a simple lifecycle:

```text
Create Resource
      ↓
Update Resource
      ↓
Destroy Resource
```

---

However, some resources have dependencies.

Example:

```text
EC2 Instance
      ↓
Depends On
      ↓
Security Group
```

---

In such cases, default Terraform behavior may cause:

```text
Downtime

Resource Conflicts

Unexpected Changes
```

---

To control resource behavior, Terraform provides:

```text
Lifecycle Meta Arguments
```

---

# What is Lifecycle?

Lifecycle allows Terraform engineers to control:

```text
Resource Creation

Resource Replacement

Resource Deletion

Change Detection
```

---

# Common Lifecycle Arguments

```text
create_before_destroy

prevent_destroy

ignore_changes
```

---

# Why Lifecycle is Important?

Consider:

```text
Security Group
```

attached to:

```text
EC2 Instance
```

---

Architecture:

```text
EC2
 ↓
Security Group
```

---

Suppose a Security Group change requires:

```text
Resource Replacement
```

---

Default Terraform Behavior

```text
Destroy Old SG
      ↓
Create New SG
```

---

Problem:

```text
EC2 Still Uses Old SG
```

---

AWS will not allow:

```text
Security Group Deletion
```

because it is attached.

---

Result:

```text
Terraform Failure
```

---

# create_before_destroy

This lifecycle rule changes the order.

---

Instead of:

```text
Destroy Old
      ↓
Create New
```

---

Terraform performs:

```text
Create New
      ↓
Switch References
      ↓
Destroy Old
```

---

# Example

Current State:

```text
EC2
 ↓
SG-OLD
```

---

New Security Group Required:

```text
SG-NEW
```

---

Flow

```text
Create SG-NEW
      ↓
Attach SG-NEW To EC2
      ↓
Detach SG-OLD
      ↓
Delete SG-OLD
```

---

# Benefit

```text
Zero Downtime

Safe Replacement

Dependency Handling
```

---

# Example Configuration

```hcl
resource "aws_security_group" "main" {

  lifecycle {
    create_before_destroy = true
  }

}
```

---

# Production Example

Application Server:

```text
Catalogue
```

---

Security Group requires replacement.

---

Without Lifecycle:

```text
Downtime Possible
```

---

With:

```text
create_before_destroy
```

Terraform safely replaces the Security Group.

---

# prevent_destroy

Sometimes resources should never be deleted accidentally.

Examples:

```text
Production Database

Production VPC

Critical S3 Bucket
```

---

Terraform provides:

```text
prevent_destroy
```

---

# Configuration

```hcl
resource "aws_db_instance" "prod" {

  lifecycle {
    prevent_destroy = true
  }

}
```

---

# Result

Even if someone runs:

```bash
terraform destroy
```

Terraform blocks deletion.

---

Error Example

```text
Resource Cannot Be Destroyed
```

---

# Production Use Case

Protect:

```text
Production MySQL

Production PostgreSQL

Production S3 Backups
```

---

# ignore_changes

Some resources are modified outside Terraform.

---

Examples:

```text
Auto Scaling Groups

Tags

Cloud Services

Managed Services
```

---

Terraform may detect:

```text
Unexpected Changes
```

---

and attempt to revert them.

---

Sometimes this behavior is undesirable.

---

# Example

Auto Scaling Group

Configuration:

```text
min=1

desired=1

max=10
```

---

# Current State

Application Traffic Increases.

---

Auto Scaling Creates:

```text
5 Instances
```

---

ASG Updates:

```text
desired_capacity = 5
```

---

Terraform Code Still Contains:

```text
desired_capacity = 1
```

---

# Problem

Next Terraform Apply:

```text
Terraform Tries To Revert ASG
```

---

Result:

```text
Scaling Interrupted
```

---

# Solution

Ignore Capacity Changes.

---

Example

```hcl
lifecycle {
  ignore_changes = [
    min_size,
    max_size,
    desired_capacity,
  ]
}
```

---

# What Happens?

Terraform ignores:

```text
min_size

max_size

desired_capacity
```

changes.

---

Even if AWS changes them.

---

# Production Example

ASG Scales:

```text
1 → 5 Instances
```

---

Terraform Apply:

```text
No Changes
```

because ignored attributes are skipped.

---

# Complete ASG Example

```hcl
resource "aws_autoscaling_group" "catalogue" {

  lifecycle {
    ignore_changes = [
      min_size,
      max_size,
      desired_capacity,
    ]
  }

}
```

---

# Lifecycle Decision Flow

Without Lifecycle

```text
Resource Change
      ↓
Terraform Action
      ↓
Apply Change
```

---

With Lifecycle

```text
Resource Change
      ↓
Lifecycle Rules
      ↓
Controlled Behavior
```

---

# Real Production Scenario

Suppose:

```text
Catalogue ASG
```

---

Traffic Surge:

```text
1 Instance
      ↓
5 Instances
```

---

ASG Updates:

```text
desired_capacity
```

---

Terraform Code:

```text
desired_capacity = 1
```

---

Without:

```text
ignore_changes
```

Terraform tries:

```text
5 → 1
```

---

Result:

```text
Application Capacity Reduced
```

---

With:

```text
ignore_changes
```

Terraform allows ASG to manage scaling independently.

---

# Common Interview Questions

## What is Terraform Lifecycle?

### Short Answer

Lifecycle is a Terraform meta argument used to control resource creation, replacement, and deletion behavior.

### Detailed Explanation

Lifecycle provides additional control over how Terraform handles resources during apply and destroy operations.

### Production Example

Using lifecycle settings for Security Groups and Auto Scaling Groups.

### Follow-Up Questions

- What lifecycle arguments are available?
- Why is lifecycle useful?

---

## What is create_before_destroy?

### Short Answer

Terraform creates the replacement resource before deleting the existing resource.

### Detailed Explanation

This prevents dependency issues and reduces downtime during resource replacement.

### Production Example

Replacing a Security Group attached to EC2 instances.

### Follow-Up Questions

- Why is it needed?
- Which resources commonly use it?

---

## What is prevent_destroy?

### Short Answer

Prevents accidental deletion of critical resources.

### Detailed Explanation

Terraform blocks destroy operations for resources protected by prevent_destroy.

### Production Example

Protecting production databases.

### Follow-Up Questions

- Can terraform destroy bypass it?
- Which resources should use it?

---

## What is ignore_changes?

### Short Answer

Tells Terraform to ignore changes to specified attributes.

### Detailed Explanation

Useful when external systems or AWS services modify resource attributes automatically.

### Production Example

Ignoring ASG capacity changes caused by Auto Scaling.

### Follow-Up Questions

- Why is it useful for ASGs?
- Can multiple attributes be ignored?

---

# Key Takeaways

```text
Lifecycle meta arguments provide additional control over resource behavior.

create_before_destroy helps avoid downtime and dependency issues.

prevent_destroy protects critical resources from accidental deletion.

ignore_changes prevents Terraform from reverting external changes.

Auto Scaling Groups commonly use ignore_changes.

Security Groups often use create_before_destroy.

Lifecycle rules are heavily used in production Terraform deployments.

Proper lifecycle configuration improves reliability and safety.
```