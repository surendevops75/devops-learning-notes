# Terraform Tagging Strategies

## Introduction

Tagging is one of the most important practices in cloud infrastructure management.

A tag is simply:

```text
Key = Value
```

metadata attached to cloud resources.

Examples:

```text
EC2 Instances

VPC

Subnets

Security Groups

Load Balancers

RDS

S3 Buckets
```

---

# Why Are Tags Important?

Without tags:

```text
Difficult Resource Identification

Poor Cost Tracking

Difficult Ownership Tracking

Operational Challenges
```

With tags:

```text
Easy Resource Management

Cost Allocation

Automation

Governance
```

---

# What is a Tag?

Example:

```hcl
tags = {

  Name = "catalogue"

}
```

---

AWS Resource

```text
EC2 Instance
```

Tag:

```text
Name = catalogue
```

---

# Tag Structure

```text
Key = Value
```

Examples:

```text
Project = Roboshop

Environment = Dev

Owner = DevOps

Application = Catalogue
```

---

# Common Tags

Most organizations maintain a set of standard tags.

Example:

```hcl
{
  Project = "Roboshop"

  Terraform = "true"
}
```

---

Purpose

```text
Identify Project

Identify Terraform Managed Resources
```

---

# Environment-Specific Tags

Different environments require different tags.

DEV:

```hcl
{
  Environment = "dev"
}
```

---

PROD:

```hcl
{
  Environment = "prod"
}
```

---

# Component Tags

Application-specific information.

Example:

```hcl
{
  Component = "catalogue"
}
```

---

Examples

```text
catalogue

user

cart

shipping

payment

frontend
```

---

# Typical Production Tag Structure

```hcl
tags = {

  Project = "Roboshop"

  Terraform = "true"

  Environment = "dev"

  Component = "catalogue"

}
```

---

# Why Separate Common and Specific Tags?

Suppose:

Every resource needs:

```text
Project

Terraform
```

tags.

But only some values change:

```text
Environment

Component
```

Repeating common tags everywhere creates duplication.

---

# Common Tags Strategy

Create reusable common tags.

Example:

```hcl
locals {

  common_tags = {

    Project = "Roboshop"

    Terraform = "true"

  }

}
```

---

# Environment Tags

```hcl
{
  Environment = "dev"
}
```

---

# Component Tags

```hcl
{
  Component = "catalogue"
}
```

---

# Using merge()

Terraform provides:

```hcl
merge()
```

to combine maps.

---

# Example

```hcl
tags = merge(

  local.common_tags,

  {

    Environment = "dev"

    Component = "catalogue"

  }

)
```

---

# Final Result

```hcl
{
  Project     = "Roboshop"

  Terraform   = "true"

  Environment = "dev"

  Component   = "catalogue"
}
```

---

# Production Example

Variables:

```hcl
variable "environment" {

  default = "dev"

}
```

---

```hcl
variable "component" {

  default = "catalogue"

}
```

---

Locals:

```hcl
locals {

  common_tags = {

    Project = "Roboshop"

    Terraform = "true"

  }

}
```

---

Resource:

```hcl
resource "aws_instance" "catalogue" {

  ami = "ami-xxxxxxxx"

  instance_type = "t3.micro"

  tags = merge(

    local.common_tags,

    {

      Environment = var.environment

      Component = var.component

    }

  )

}
```

---

# Output

```hcl
{
  Project     = "Roboshop"

  Terraform   = "true"

  Environment = "dev"

  Component   = "catalogue"
}
```

---

# Using Tags with Multiple Environments

DEV

```hcl
Environment = "dev"
```

---

QA

```hcl
Environment = "qa"
```

---

PROD

```hcl
Environment = "prod"
```

---

Same codebase.

Different values.

---

# Workspace-Based Tags

Terraform provides:

```hcl
terraform.workspace
```

---

Example

```hcl
tags = merge(

  local.common_tags,

  {

    Environment = terraform.workspace

  }

)
```

---

Workspace:

```text
dev
```

Output:

```text
Environment = dev
```

---

Workspace:

```text
prod
```

Output:

```text
Environment = prod
```

---

# Cost Allocation Tags

Organizations often track cloud costs using tags.

Examples:

```text
Project

Department

CostCenter

BusinessUnit
```

---

Example

```hcl
{
  CostCenter = "Engineering"

  Department = "Platform"
}
```

---

# Ownership Tags

Useful for support and operations.

Example:

```hcl
{
  Owner = "DevOps Team"

  Team = "Platform Engineering"
}
```

---

# Compliance Tags

Many enterprises enforce:

```text
Owner

Environment

CostCenter

Application
```

as mandatory tags.

---

# Resource Naming vs Tags

Resource Name:

```text
catalogue-dev
```

---

Tags:

```text
Project = Roboshop

Environment = dev

Component = catalogue
```

---

Both are important.

---

# Real Roboshop Example

Catalogue Service:

```hcl
{
  Project = "Roboshop"

  Terraform = "true"

  Environment = "dev"

  Component = "catalogue"
}
```

---

User Service:

```hcl
{
  Project = "Roboshop"

  Terraform = "true"

  Environment = "dev"

  Component = "user"
}
```

---

Only:

```text
Component
```

changes.

---

# Common Mistakes

## No Tagging Strategy

Results in:

```text
Unmanaged Resources

Cost Tracking Issues
```

---

## Hardcoding Tags

Bad:

```hcl
Project = "Roboshop"
```

repeated everywhere.

Use:

```hcl
local.common_tags
```

---

## Missing Environment Tags

Makes troubleshooting difficult.

---

## Inconsistent Naming

Bad:

```text
Dev

DEV

development
```

Choose one standard.

---

# Best Practices

## Define Common Tags in Locals

---

## Use merge() for Tag Composition

---

## Tag Every Resource

---

## Include Environment Tag

---

## Include Ownership Tag

---

## Include Cost Allocation Tag

---

## Maintain Consistent Standards

---

# Frequently Asked Interview Questions

## Why Are Tags Important in AWS?

### Short Answer

Tags help identify, organize, manage, and track cloud resources.

### Detailed Explanation

Tags provide metadata that improves resource visibility, governance, cost allocation, automation, and operational management. Most enterprises enforce tagging standards across all infrastructure.

### Production Example

Using tags to identify all resources belonging to the Roboshop application and calculate their monthly AWS costs.

### Follow-Up Questions

- What tags do you commonly use?
- How do tags help with cost optimization?

---

## What Are Common Tags?

### Short Answer

Common tags are reusable tags applied to most or all resources.

### Detailed Explanation

Organizations typically define standard tags such as Project, Environment, Owner, and Terraform. These tags are merged with resource-specific tags to maintain consistency.

### Production Example

Applying Project=Roboshop and Terraform=true to all AWS resources.

### Follow-Up Questions

- Where do you store common tags?
- Why use locals for common tags?

---

## Why Use merge() for Tags?

### Short Answer

merge() combines common tags and resource-specific tags into a single tag map.

### Detailed Explanation

Instead of repeating standard tags across resources, merge() allows centralized tag management while still supporting environment and component-specific values.

### Production Example

Combining Project, Terraform, Environment, and Component tags for an EC2 instance.

### Follow-Up Questions

- What happens if duplicate keys exist?
- Why is merge() useful?

---

## How Do You Manage Tags Across Environments?

### Short Answer

Using variables, tfvars files, or Terraform workspaces.

### Detailed Explanation

Environment-specific tags can be generated dynamically using variables or the current Terraform workspace, ensuring consistency across DEV, QA, and PROD environments.

### Production Example

Using `terraform.workspace` to automatically set the Environment tag.

### Follow-Up Questions

- What is terraform.workspace?
- How do you manage environment-specific values?

---

## What Tags Do Enterprises Commonly Enforce?

### Short Answer

Project, Environment, Owner, Application, and CostCenter.

### Detailed Explanation

These tags help organizations manage ownership, compliance, billing, governance, and operational responsibilities across large cloud environments.

### Production Example

Using CostCenter tags to allocate AWS spending to individual business units.

### Follow-Up Questions

- How do tags help FinOps?
- What happens when resources are not tagged?

---

# Key Takeaways

- Tags are metadata attached to cloud resources.
- Common tags should be centralized and reused.
- `merge()` is commonly used for combining tags.
- Environment and Component tags are typically resource-specific.
- Tags improve governance, cost allocation, and resource management.
- Every production resource should have a tagging strategy.
- Tagging is a common Terraform and AWS interview topic.