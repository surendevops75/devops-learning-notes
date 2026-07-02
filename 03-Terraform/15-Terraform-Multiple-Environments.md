# Terraform Multiple Environments

## Introduction

One of the most common requirements in Terraform projects is managing multiple environments.

Examples:

```text
DEV

QA

UAT

PRE-PROD

PROD
```

Each environment may require:

```text
Different Instance Types

Different Resource Sizes

Different Tags

Different DNS Records
```

Terraform provides multiple approaches to manage environments efficiently.

---

# Why Do We Need Multiple Environments?

Development teams usually deploy infrastructure through several stages.

Example:

```text
Developer
     ↓
DEV
     ↓
QA
     ↓
UAT
     ↓
PROD
```

Every environment has different requirements.

---

# Example

DEV:

```text
t3.micro

Small Database

Minimal Resources
```

---

PROD:

```text
t3.large

High Availability

Monitoring

Backups
```

---

# Common Approaches

Terraform environments are typically managed using:

```text
1. TFVARS Files

2. Terraform Workspaces

3. Separate Repositories
```

---

# Approach 1: Using TFVARS Files

This is one of the most popular approaches.

---

# Variables

variables.tf

```hcl
variable "environment" {}

variable "instance_type" {}
```

---

# DEV Configuration

dev.tfvars

```hcl
environment = "dev"

instance_type = "t3.micro"
```

---

# PROD Configuration

prod.tfvars

```hcl
environment = "prod"

instance_type = "t3.large"
```

---

# Deploy DEV

```bash
terraform apply \
-var-file=dev.tfvars
```

---

# Deploy PROD

```bash
terraform apply \
-var-file=prod.tfvars
```

---

# Advantages

```text
Simple

Easy To Understand

Most Common Method
```

---

# Disadvantages

```text
Need Separate State Files

Manual Environment Selection
```

---

# Approach 2: Terraform Workspaces

Terraform supports:

```text
Workspaces
```

to maintain multiple state files automatically.

---

# Default Workspace

Terraform starts with:

```text
default
```

workspace.

Check current workspace:

```bash
terraform workspace show
```

---

# Create Workspace

```bash
terraform workspace new dev
```

---

Create another:

```bash
terraform workspace new prod
```

---

# List Workspaces

```bash
terraform workspace list
```

Output:

```text
default

dev

prod
```

---

# Switch Workspace

```bash
terraform workspace select dev
```

---

Switch to PROD:

```bash
terraform workspace select prod
```

---

# Special Variable

Terraform provides:

```hcl
terraform.workspace
```

---

Example

```hcl
locals {

  environment = terraform.workspace

}
```

---

Current Workspace:

```text
dev
```

Output:

```text
dev
```

---

# Environment-Based Instance Types

Example:

```hcl
locals {

  instance_type = lookup(

    {

      dev  = "t3.micro"

      qa   = "t3.small"

      prod = "t3.large"

    },

    terraform.workspace,

    "t3.micro"

  )

}
```

---

# Understanding lookup()

Syntax:

```hcl
lookup(map, key, default)
```

---

Example:

```hcl
lookup(

  {

    dev  = "t3.micro"

    prod = "t3.large"

  },

  "prod",

  "t3.micro"

)
```

Output:

```text
t3.large
```

---

# What Happens If Key Does Not Exist?

Example:

```hcl
lookup(

  {

    dev = "t3.micro"

  },

  "uat",

  "t3.small"

)
```

Output:

```text
t3.small
```

Default value is returned.

---

# Workspace-Based Resource Naming

Example:

```hcl
tags = {

  Name = "catalogue-${terraform.workspace}"

}
```

Output:

DEV:

```text
catalogue-dev
```

---

PROD:

```text
catalogue-prod
```

---

# Advantages of Workspaces

```text
Single Code Base

Separate State Files

Easy Environment Switching
```

---

# Disadvantages of Workspaces

```text
Can Become Difficult To Manage

Less Isolation Between Environments
```

for large organizations.

---

# Approach 3: Separate Repositories

Large organizations often maintain:

```text
roboshop-infra-dev

roboshop-infra-qa

roboshop-infra-prod
```

---

# Repository Structure

```text
Git Repository 1

DEV
```

---

```text
Git Repository 2

PROD
```

---

Each environment has:

```text
Independent Code

Independent State

Independent Pipelines
```

---

# Advantages

```text
Strong Isolation

Reduced Risk

Independent Release Cycles
```

---

# Disadvantages

```text
Code Duplication

Maintenance Overhead
```

---

# Which Approach Is Best?

Small Teams:

```text
TFVARS
```

---

Medium Teams:

```text
Workspaces
```

---

Large Enterprises:

```text
Separate Repositories
```

or

```text
Separate Terraform Projects
```

---

# Real Roboshop Example

DEV:

```text
catalogue-dev

user-dev

cart-dev
```

---

PROD:

```text
catalogue-prod

user-prod

cart-prod
```

---

Using Workspaces:

```hcl
Name = "${var.component}-${terraform.workspace}"
```

---

Output Automatically Changes:

```text
catalogue-dev

catalogue-prod
```

based on selected workspace.

---

# Common Mistakes

## Hardcoding Environment Names

Bad:

```hcl
Environment = "dev"
```

Use:

```hcl
terraform.workspace
```

or

```hcl
var.environment
```

---

## Sharing Same State Across Environments

Can cause:

```text
Resource Conflicts

Accidental Changes
```

---

## Using One TFVARS File Everywhere

Creates maintenance problems.

---

# Best Practices

## Separate State Per Environment

---

## Separate CI/CD Pipelines

---

## Use Consistent Naming Standards

---

## Use Tags Extensively

---

## Protect Production Environment

---

# Frequently Asked Interview Questions

## How Do You Create Multiple Environments in Terraform?

### Short Answer

Terraform environments can be managed using TFVARS files, Workspaces, or separate repositories.

### Detailed Explanation

Each approach provides a different level of isolation and operational complexity. The choice depends on team size, compliance requirements, and infrastructure scale.

### Production Example

Using dev.tfvars and prod.tfvars to deploy different EC2 instance types for development and production environments.

### Follow-Up Questions

- Which approach do you prefer?
- How do you separate state files?

---

## What is a Terraform Workspace?

### Short Answer

A Workspace is a Terraform feature that maintains separate state files for different environments.

### Detailed Explanation

Workspaces allow the same Terraform code to manage multiple environments by maintaining independent state files under different workspace names.

### Production Example

Using dev and prod workspaces with identical Terraform code but separate infrastructure states.

### Follow-Up Questions

- How do you create a workspace?
- What is terraform.workspace?

---

## What is terraform.workspace?

### Short Answer

terraform.workspace returns the name of the currently selected workspace.

### Detailed Explanation

It allows Terraform configurations to behave differently depending on the active workspace.

### Production Example

Assigning different instance sizes based on the selected environment.

### Follow-Up Questions

- Can terraform.workspace be used in locals?
- How is it used with lookup()?

---

## When Would You Use Separate Repositories?

### Short Answer

For large environments that require strong isolation.

### Detailed Explanation

Separate repositories provide independent codebases, pipelines, permissions, and state management for each environment.

### Production Example

Maintaining separate repositories for production and development infrastructure.

### Follow-Up Questions

- What are the disadvantages?
- How do enterprises manage environment separation?

---

# Key Takeaways

- Terraform commonly manages DEV, QA, UAT, and PROD environments.
- Three popular approaches are:
  
  ```text
  TFVARS

  Workspaces

  Separate Repositories
  ```

- Workspaces provide environment-specific state files.
- terraform.workspace returns the active workspace.
- lookup() is commonly used with workspaces.
- Environment isolation becomes more important as infrastructure grows.
- Multiple environment strategies are frequently discussed in Terraform interviews.