# Terraform Modules

## Introduction

As Terraform projects grow, infrastructure code becomes larger and more difficult to maintain.

Suppose an organization has:

```text
Savings Banking

Current Banking

Mobile Banking

WhatsApp Banking

Loans

Demat
```

Each project creates:

```text
EC2 Instances

Security Groups

Route53 Records

Load Balancers

IAM Roles
```

If every team writes its own Terraform code:

```text
Code Duplication

No Standards

Maintenance Problems

Governance Issues
```

Terraform solves this problem using:

```text
Modules
```

---

# What is a Module?

A Module is a reusable Terraform component that encapsulates infrastructure logic.

Think of a module as:

```text
Reusable Infrastructure Package
```

or

```text
Function In Programming
```

---

# Why Do We Need Modules?

Without modules:

```text
Team A Writes EC2 Code

Team B Writes EC2 Code

Team C Writes EC2 Code
```

Result:

```text
Duplicate Code

Different Standards

Maintenance Issues
```

---

# Real-World Problem

Suppose all projects create EC2 instances.

Security team mandates:

```text
New AMI Version

Mandatory Tags

Updated Security Standards
```

Without modules:

```text
Update Every Project

Update Every Repository

High Risk Of Missing Projects
```

---

# What Happens Without Modules?

Example:

```text
Savings Banking

Current Banking

Mobile Banking

WhatsApp Banking
```

Each project contains:

```text
Separate EC2 Code
```

If instance standards change:

```text
Update All Projects Individually
```

---

# Problems

```text
No Single Point Of Control

Difficult Maintenance

Difficult Auditing

Inconsistent Standards
```

---

# Module Advantages

## Centralized Updates

Changes made inside a module automatically affect all projects using it.

Example:

```text
Update EC2 Standard Once

All Projects Benefit
```

---

## Governance and Compliance

Organizations can enforce:

```text
Tagging Standards

Security Standards

Naming Standards

Compliance Rules
```

inside modules.

All teams automatically follow those standards.

---

## Code Reusability

Write Once.

Reuse Everywhere.

---

## Easier Maintenance

Instead of:

```text
10 Projects To Update
```

Update:

```text
1 Module
```

---

# Module Roles

Terraform modules usually involve:

```text
Module Developers

Module Users
```

---

# Module Developers

Responsible for:

```text
Creating Modules

Implementing Standards

Maintaining Modules
```

---

# Module Users

Responsible for:

```text
Using Existing Modules

Providing Inputs

Consuming Outputs
```

---

# Module Structure

Example:

```text
terraform-aws-ec2-module/

├── main.tf

├── variables.tf

├── outputs.tf

└── README.md
```

---

# Module Usage

Syntax:

```hcl
module "module_name" {

  source = ""

}
```

---

# Example

```hcl
module "ec2" {

  source = "./modules/ec2"

}
```

---

# Understanding Source

Source tells Terraform:

```text
Where Module Exists
```

---

# Local Module

```hcl
source = "./modules/ec2"
```

---

# Git Module

```hcl
source = "git::https://github.com/org/ec2-module.git"
```

---

# Terraform Registry Module

```hcl
source = "terraform-aws-modules/ec2-instance/aws"
```

---

# Module Inputs

Modules accept variables.

Example Module:

variables.tf

```hcl
variable "instance_type" {}

variable "component" {}
```

---

# Module Call

```hcl
module "catalogue" {

  source = "./modules/ec2"

  instance_type = "t3.micro"

  component = "catalogue"

}
```

---

# Mandatory Variables

If a variable has no default value:

```hcl
variable "component" {}
```

Terraform requires:

```hcl
component = "catalogue"
```

while calling the module.

---

# Module Outputs

Modules can expose information.

Example:

outputs.tf

```hcl
output "private_ip" {

  value = aws_instance.this.private_ip

}
```

---

Module Usage:

```hcl
module.catalogue.private_ip
```

---

# Real Roboshop Example

Instead of:

```text
catalogue.tf

user.tf

cart.tf

shipping.tf
```

containing duplicate EC2 code,

create:

```text
EC2 Module
```

---

Module Call

```hcl
module "catalogue" {

  source = "./modules/ec2"

  component = "catalogue"

}
```

---

```hcl
module "user" {

  source = "./modules/ec2"

  component = "user"

}
```

---

```hcl
module "cart" {

  source = "./modules/ec2"

  component = "cart"

}
```

---

Benefits:

```text
Single Code Base

Reusable

Easy Maintenance
```

---

# Naming Conventions

Terraform follows specific naming conventions.

---

## Human Readable Names

Use:

```text
sivakumar-reddy-m
```

Reason:

```text
Humans Prefer Hyphens
```

---

## Terraform Resource Names

Use:

```text
catalogue

frontend

user
```

instead of:

```text
catalogue-instance

frontend-instance
```

---

Bad:

```hcl
resource "aws_instance" "catalogue-instance"
```

Reason:

```text
Resource Type Already Specifies Instance
```

---

Good:

```hcl
resource "aws_instance" "catalogue"
```

---

# Variable Naming Standards

Single Value:

```hcl
id
```

Type:

```text
String
```

---

Multiple Values:

```hcl
ids
```

Type:

```text
List
```

---

Examples

```hcl
subnet_id
```

Single subnet.

---

```hcl
subnet_ids
```

Multiple subnets.

---

# Production Module Architecture

```text
Root Project
       ↓

EC2 Module

Security Group Module

Route53 Module

RDS Module
```

---

# Example

```text
Roboshop
      ↓

EC2 Module
      ↓

catalogue

user

cart

shipping
```

All use the same module.

---

# Common Mistakes

## Copy-Pasting Terraform Code

Results in:

```text
Code Duplication

Maintenance Problems
```

---

## Hardcoding Values

Bad:

```hcl
instance_type = "t3.micro"
```

Use variables.

---

## No Module Standards

Leads to:

```text
Different Naming Standards

Different Security Standards
```

---

## Large Monolithic Modules

Avoid creating one module that manages everything.

---

# Best Practices

## Build Reusable Modules

---

## Follow Naming Standards

---

## Keep Modules Small

One module should solve one problem.

---

## Use Variables for Inputs

---

## Use Outputs for Exposing Information

---

## Enforce Standards Through Modules

---

# Frequently Asked Interview Questions

## What is a Terraform Module?

### Short Answer

A Terraform Module is a reusable collection of Terraform resources that can be used across multiple projects.

### Detailed Explanation

Modules help reduce duplication, improve maintainability, and enforce infrastructure standards. They allow organizations to centralize infrastructure logic and reuse it consistently.

### Production Example

Creating a reusable EC2 module used by catalogue, user, cart, and shipping services.

### Follow-Up Questions

- What are module inputs?
- What are module outputs?
- Why use modules?

---

## Why Do We Need Modules?

### Short Answer

To improve reusability, governance, and maintainability.

### Detailed Explanation

Without modules, teams duplicate infrastructure code across projects. Modules centralize infrastructure logic and make updates easier.

### Production Example

Updating EC2 standards in one module instead of modifying dozens of projects.

### Follow-Up Questions

- What problems occur without modules?
- How do modules help compliance?

---

## What is the Difference Between Module Developer and Module User?

### Short Answer

Module developers build modules, while module users consume them.

### Detailed Explanation

A platform team often develops modules that application teams use to provision infrastructure consistently.

### Production Example

Platform team creates an EC2 module, and application teams use it for deploying services.

### Follow-Up Questions

- What responsibilities belong to module developers?
- What inputs should modules expose?

---

## What is Source in a Module?

### Short Answer

Source tells Terraform where to find the module.

### Detailed Explanation

Terraform supports local modules, Git-based modules, and modules from the Terraform Registry.

### Production Example

```hcl
source = "./modules/ec2"
```

### Follow-Up Questions

- Can modules be stored in Git?
- Can modules come from Terraform Registry?

---

## What Are Module Inputs and Outputs?

### Short Answer

Inputs are variables passed into modules, while outputs expose values from modules.

### Detailed Explanation

Inputs make modules configurable, while outputs allow other resources and modules to consume generated values.

### Production Example

Passing component names as inputs and exposing private IPs as outputs.

### Follow-Up Questions

- How are outputs referenced?
- Why are outputs important?

---

# Key Takeaways

- Modules are reusable Terraform components.
- Modules reduce code duplication.
- Modules provide centralized governance and standards.
- Modules support inputs through variables.
- Modules expose information through outputs.
- Modules are heavily used in enterprise Terraform projects.
- A good module should be reusable, maintainable, and focused on a single responsibility.