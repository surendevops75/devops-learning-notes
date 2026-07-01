# Terraform Variables and TFVARS

## Introduction

One of the most important principles in software development and DevOps is:

```text
DRY

Don't Repeat Yourself
```

Instead of hardcoding values multiple times, Terraform provides:

```text
Variables
```

Variables make Terraform code:

```text
Reusable

Flexible

Maintainable

Environment Independent
```

---

# Why Do We Need Variables?

Without variables:

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = "t3.micro"

}
```

If we need:

```text
DEV

QA

PROD
```

environments, we must manually edit the code every time.

This creates:

```text
Duplication

Human Errors

Maintenance Problems
```

---

# What is a Variable?

A variable is a placeholder that stores a value.

Example:

```hcl
variable "instance_type" {

  default = "t3.micro"

}
```

Instead of hardcoding:

```hcl
instance_type = "t3.micro"
```

we can use:

```hcl
instance_type = var.instance_type
```

---

# Variable Syntax

```hcl
variable "variable_name" {

  default = "value"

}
```

---

# Example

```hcl
variable "environment" {

  default = "dev"

}
```

Usage:

```hcl
tags = {
  Environment = var.environment
}
```

---

# Accessing Variables

Terraform uses:

```hcl
var.variable_name
```

Examples:

```hcl
var.instance_type

var.environment

var.domain_name
```

---

# Complete Example

variables.tf

```hcl
variable "instance_type" {

  default = "t3.micro"

}
```

main.tf

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = var.instance_type

}
```

---

# Variable Types

Terraform supports several data types.

---

## String

```hcl
variable "environment" {

  default = "dev"

}
```

---

## Number

```hcl
variable "instance_count" {

  default = 3

}
```

---

## Boolean

```hcl
variable "create_instance" {

  default = true

}
```

---

## List

```hcl
variable "instances" {

  default = [
    "frontend",
    "catalogue",
    "user"
  ]

}
```

---

## Map

```hcl
variable "instance_types" {

  default = {

    dev  = "t3.micro"

    prod = "t3.large"

  }

}
```

---

## Object

```hcl
variable "instance" {

  default = {

    name = "frontend"

    type = "t3.micro"

  }

}
```

---

# Project Structure

Terraform projects typically use:

```text
main.tf

variables.tf

terraform.tfvars
```

---

# variables.tf

Contains variable declarations.

Example:

```hcl
variable "environment" {}

variable "instance_type" {}

variable "domain_name" {}
```

---

# terraform.tfvars

Contains actual values.

Example:

```hcl
environment = "dev"

instance_type = "t3.micro"

domain_name = "daws86s.fun"
```

---

# Why Use terraform.tfvars?

Benefits:

```text
Separation of Code and Configuration

Environment Specific Values

Cleaner Code

Easy Maintenance
```

---

# Variable Value Precedence

Terraform follows a precedence order.

---

# 1. Command Line Variables

Highest priority.

Example:

```bash
terraform apply \
-var="instance_type=t3.large"
```

Overrides all other values.

---

# 2. terraform.tfvars

Example:

```hcl
instance_type = "t3.medium"
```

Terraform automatically loads:

```text
terraform.tfvars
```

---

# 3. Environment Variables

Terraform recognizes variables prefixed with:

```text
TF_VAR_
```

Example:

```bash
export TF_VAR_instance_type=t3.small
```

Terraform automatically maps:

```text
TF_VAR_instance_type
```

to:

```hcl
variable "instance_type"
```

---

# 4. Default Value

Example:

```hcl
variable "instance_type" {

  default = "t3.micro"

}
```

Used when no higher-priority value exists.

---

# 5. Interactive Prompt

Example:

```hcl
variable "instance_type" {}
```

Running:

```bash
terraform apply
```

Terraform asks:

```text
Enter a value:
```

if no value is supplied.

---

# Variable Precedence Flow

```text
Command Line Variables
          ↓
terraform.tfvars
          ↓
Environment Variables
          ↓
Default Values
          ↓
Prompt User
```

---

# Environment Specific Files

Production projects usually maintain:

```text
dev.tfvars

qa.tfvars

prod.tfvars
```

---

## dev.tfvars

```hcl
environment = "dev"

instance_type = "t3.micro"
```

---

## prod.tfvars

```hcl
environment = "prod"

instance_type = "t3.large"
```

---

# Using Specific TFVARS Files

DEV:

```bash
terraform apply \
-var-file=dev.tfvars
```

---

PROD:

```bash
terraform apply \
-var-file=prod.tfvars
```

---

# Real Roboshop Example

variables.tf

```hcl
variable "environment" {}

variable "domain_name" {}

variable "instance_type" {}
```

---

terraform.tfvars

```hcl
environment   = "dev"

domain_name   = "daws86s.fun"

instance_type = "t3.micro"
```

---

Usage:

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = var.instance_type

}
```

---

# Sensitive Variables

Terraform supports:

```hcl
variable "db_password" {

  sensitive = true

}
```

Purpose:

```text
Hide Sensitive Values

Avoid Accidental Exposure
```

---

# Common Mistakes

## Hardcoding Values

Bad:

```hcl
instance_type = "t3.micro"
```

everywhere.

Use variables.

---

## Storing Passwords in TFVARS

Bad:

```hcl
db_password = "DevOps123"
```

Use:

```text
AWS Secrets Manager

HashiCorp Vault
```

---

## No Environment Separation

Avoid using one tfvars file for all environments.

---

## Using Meaningless Variable Names

Bad:

```hcl
variable "x" {}
```

Good:

```hcl
variable "instance_type" {}
```

---

# Best Practices

## Keep Variables in variables.tf

---

## Keep Values in TFVARS Files

---

## Use Separate Files Per Environment

```text
dev.tfvars

qa.tfvars

prod.tfvars
```

---

## Mark Sensitive Values

```hcl
sensitive = true
```

---

## Use Descriptive Names

Example:

```hcl
variable "domain_name"
```

instead of:

```hcl
variable "d"
```

---

# Frequently Asked Interview Questions

## Why Do We Use Variables?

To improve code reusability, flexibility, and maintainability.

---

## How Do You Access a Variable?

```hcl
var.variable_name
```

Example:

```hcl
var.instance_type
```

---

## What is terraform.tfvars?

A file used to store values assigned to Terraform variables.

---

## What is Variable Precedence?

Terraform follows:

```text
1. Command Line Variables

2. terraform.tfvars

3. Environment Variables

4. Default Values

5. Prompt User
```

---

## How Do Environment Variables Work?

Example:

```bash
export TF_VAR_environment=dev
```

Terraform automatically reads it.

---

## Why Use Separate TFVARS Files?

To manage:

```text
DEV

QA

UAT

PROD
```

independently.

---

## How Do You Pass Variables at Runtime?

```bash
terraform apply \
-var="environment=prod"
```

---

# Key Takeaways

- Variables implement the DRY principle.
- Variables make Terraform code reusable and flexible.
- `var.variable_name` is used to access variables.
- `terraform.tfvars` stores variable values.
- Command-line variables have the highest precedence.
- Environment variables use the `TF_VAR_` prefix.
- Separate TFVARS files should be maintained for different environments.
- Variables are heavily used in production Terraform projects.