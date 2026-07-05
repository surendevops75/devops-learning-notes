# Terraform Outputs and Module Outputs

## Introduction

When Terraform creates infrastructure, it generates useful information.

Examples:

```text
VPC ID

Subnet ID

EC2 Private IP

Load Balancer DNS

Database Endpoint
```

---

These values are often required by:

```text
Engineers

Other Terraform Modules

CI/CD Pipelines

Applications
```

---

Terraform provides:

```text
Outputs
```

to expose this information.

---

# What is an Output?

An Output is a value that Terraform displays after:

```text
terraform apply
```

or

```text
terraform output
```

---

# Example

Terraform creates:

```text
VPC
```

---

AWS returns:

```text
vpc-0abc123456789xyz
```

---

Instead of searching AWS Console:

Terraform can display:

```text
VPC ID
```

using outputs.

---

# Basic Output Syntax

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

---

# Example

Resource:

```hcl
resource "aws_vpc" "main" {

}
```

---

Output:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

---

# Apply Result

Command:

```bash
terraform apply
```

---

Output:

```text
Outputs:

vpc_id = vpc-0abc123456789xyz
```

---

# Why Outputs?

Without Outputs:

```text
Open AWS Console

Search Resource

Copy Value
```

---

With Outputs:

```text
Terraform Displays Value
```

automatically.

---

# Accessing Outputs

Command:

```bash
terraform output
```

---

Example

```text
vpc_id = vpc-0abc123456789xyz
```

---

# Specific Output

Command:

```bash
terraform output vpc_id
```

---

Output:

```text
vpc-0abc123456789xyz
```

---

# Common Output Examples

## VPC ID

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

---

## Security Group ID

```hcl
output "sg_id" {
  value = aws_security_group.main.id
}
```

---

## EC2 Private IP

```hcl
output "private_ip" {
  value = aws_instance.catalogue.private_ip
}
```

---

## ALB DNS Name

```hcl
output "alb_dns_name" {
  value = aws_lb.frontend.dns_name
}
```

---

# Real Production Example

Terraform creates:

```text
Frontend ALB
```

---

Need DNS:

```text
frontend-alb.amazonaws.com
```

---

Output:

```hcl
output "alb_dns_name" {
  value = aws_lb.frontend.dns_name
}
```

---

Used later in:

```text
Route53
```

configuration.

---

# Sensitive Outputs

Some values should not be displayed.

Examples:

```text
Passwords

API Keys

Tokens
```

---

Terraform supports:

```text
sensitive = true
```

---

Example

```hcl
output "db_password" {
  value     = aws_db_instance.mysql.password
  sensitive = true
}
```

---

Result

```text
Value Hidden
```

during output display.

---

# Output Architecture

```text
Terraform Resource
         ↓
Output Block
         ↓
Display Value
```

---

# What are Module Outputs?

Modules are reusable Terraform components.

---

Example:

```text
VPC Module

Security Group Module

ALB Module
```

---

Resources inside modules are isolated.

---

Parent modules cannot directly access:

```text
Internal Resources
```

---

# Problem

Main Module:

```text
Needs VPC ID
```

---

VPC created inside:

```text
VPC Module
```

---

Direct access:

```text
Not Possible
```

---

# Solution

Module Outputs

---

# VPC Module Example

Inside:

```text
modules/vpc
```

---

Resource:

```hcl
resource "aws_vpc" "main" {

}
```

---

Output:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

---

# Module Output Flow

```text
Resource
      ↓
Module Output
      ↓
Parent Module
```

---

# Using Module Outputs

Module:

```hcl
module "vpc" {
  source = "./modules/vpc"
}
```

---

Access Output:

```hcl
module.vpc.vpc_id
```

---

# Example

Create Security Group

Need:

```text
VPC ID
```

---

Configuration:

```hcl
resource "aws_security_group" "catalogue" {

  vpc_id = module.vpc.vpc_id

}
```

---

# Complete Flow

```text
VPC Module
      ↓
Creates VPC
      ↓
Exports VPC ID
      ↓
Security Group Uses Output
```

---

# Multi-Module Architecture

```text
VPC Module
      ↓
VPC ID Output
      ↓
SG Module
      ↓
Security Group
```

---

# Roboshop Example

Modules:

```text
00-vpc

10-sg

20-bastion

50-backend-alb
```

---

VPC Module Output:

```text
vpc_id
```

---

Used By:

```text
SG Module

ALB Module

Bastion Module
```

---

# Why Module Outputs?

Benefits:

```text
Loose Coupling

Reusability

Modularity

Clean Design
```

---

# Output vs Variable

Variable:

```text
Input To Module
```

---

Output:

```text
Output From Module
```

---

# Flow

```text
Variable
      ↓
Module
      ↓
Output
```

---

# Production Example

VPC Module Creates:

```text
VPC
```

---

Exports:

```text
vpc_id
```

---

ALB Module Uses:

```text
module.vpc.vpc_id
```

---

Result:

```text
Modules Communicate Safely
```

---

# Common Interview Questions

## What is a Terraform Output?

### Short Answer

An Output exposes resource information created by Terraform.

### Detailed Explanation

Outputs allow Terraform to display useful values such as IDs, IPs, DNS names, and endpoints.

### Production Example

Displaying ALB DNS name after deployment.

### Follow-Up Questions

- How do you view outputs?
- Can outputs be sensitive?

---

## What is a Module Output?

### Short Answer

A Module Output exposes values from a child module to a parent module.

### Detailed Explanation

Module outputs allow modules to share resource information while maintaining isolation.

### Production Example

Exposing VPC ID from a VPC module.

### Follow-Up Questions

- How do you reference module outputs?
- Why are outputs needed?

---

## Difference Between Variable and Output?

### Short Answer

Variables are inputs, outputs are results.

### Detailed Explanation

Variables pass values into modules, while outputs expose values from modules.

### Production Example

Passing CIDR as a variable and returning VPC ID as an output.

### Follow-Up Questions

- Can outputs be used as inputs?
- How do modules communicate?

---

## What is a Sensitive Output?

### Short Answer

A sensitive output hides confidential values from Terraform output.

### Detailed Explanation

Terraform masks sensitive information such as passwords and secrets.

### Production Example

Database passwords and API tokens.

### Follow-Up Questions

- Does Terraform store sensitive values in state?
- Should secrets be exposed as outputs?

---

# Key Takeaways

```text
Outputs expose useful Terraform resource information.

terraform output displays output values.

Specific outputs can be queried individually.

Sensitive outputs hide confidential information.

Module outputs allow communication between modules.

Parent modules access outputs using module.<name>.<output>.

Variables are inputs, outputs are results.

Outputs improve modularity and reusability.

Module outputs are heavily used in production Terraform projects.

Understanding outputs is essential for Terraform module design.
```