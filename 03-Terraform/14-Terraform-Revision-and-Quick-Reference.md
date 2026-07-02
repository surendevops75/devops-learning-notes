# Terraform Revision and Quick Reference

## Terraform Overview

Terraform is an:

```text
Infrastructure as Code (IaC) Tool
```

used to:

```text
Create Infrastructure

Update Infrastructure

Delete Infrastructure
```

using code instead of manual cloud console operations.

Terraform follows:

```text
Declarative Approach
```

Meaning:

```text
We Declare Desired Infrastructure

Terraform Figures Out How To Achieve It
```

---

# Terraform Workflow

```text
Write Terraform Code
        ↓
terraform init
        ↓
terraform plan
        ↓
terraform apply
        ↓
Infrastructure Created
        ↓
terraform destroy
```

---

# Core Terraform Concepts

```text
Setup

Variables

Data Types

Conditions

Loops

Functions

Data Sources

Locals

State

Remote State

Provisioners
```

---

# Terraform Setup

Install:

```text
Terraform

AWS CLI v2
```

Configure:

```bash
aws configure
```

Verify:

```bash
terraform --version

aws --version
```

---

# Important Terraform Commands

## Initialize Terraform

```bash
terraform init
```

Purpose:

```text
Download Providers

Initialize Working Directory
```

---

## Generate Execution Plan

```bash
terraform plan
```

Purpose:

```text
Shows What Terraform Will Do
```

---

## Create Infrastructure

```bash
terraform apply
```

---

## Delete Infrastructure

```bash
terraform destroy
```

---

# Resource Block

Terraform creates resources using:

```hcl
resource "resource_type" "resource_name" {

  key = value

}
```

Example:

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-xxxxxxxx"

  instance_type = "t3.micro"

}
```

---

# Variables

Variables help implement:

```text
DRY Principle

(Don't Repeat Yourself)
```

---

## Variable Syntax

```hcl
variable "var_name" {

  default = "default_value"

}
```

---

## Access Variable

```hcl
var.var_name
```

---

## Interpolation

```hcl
"${var.var_name}"
```

---

Example:

```hcl
Name = "${var.environment}-frontend"
```

Output:

```text
dev-frontend
```

---

# Variable Precedence

Terraform follows:

```text
1. Command Line Variables

2. terraform.tfvars

3. Environment Variables

4. Default Values

5. Prompt User
```

---

# Data Types

Terraform supports:

```text
String

Number

Boolean

List

Map

Object
```

---

## String

```hcl
"dev"
```

---

## Number

```hcl
10
```

---

## Boolean

```hcl
true

false
```

---

## List

```hcl
[
  "frontend",
  "catalogue",
  "user"
]
```

---

## Map

```hcl
{
  frontend = "10.0.1.10"

  user = "10.0.1.20"
}
```

---

# Conditions

Terraform uses:

```hcl
condition ? true_value : false_value
```

instead of traditional:

```text
if

else
```

---

## Example

```hcl
var.environment == "prod"

? "t3.large"

: "t3.micro"
```

---

Output:

```text
PROD → t3.large

DEV → t3.micro
```

---

# Loops

Terraform supports:

```text
Count

For_Each

Dynamic Blocks
```

---

# Count

Used for:

```text
List Iteration

Multiple Resource Creation
```

---

Example:

```hcl
count = 3
```

Terraform creates:

```text
Resource[0]

Resource[1]

Resource[2]
```

---

# count.index

Special variable:

```hcl
count.index
```

Values:

```text
0

1

2
```

---

# Count with List

```hcl
var.instances[count.index]
```

---

# For_Each

Works with:

```text
Map

Set
```

---

Example:

```hcl
for_each = toset([
  "frontend",
  "catalogue",
  "user"
])
```

---

# each.value

Represents:

```text
Current Value
```

Example:

```hcl
each.value
```

Output:

```text
frontend

catalogue

user
```

---

# Map Iteration

Map:

```hcl
{
  frontend = "10.0.1.10"

  user = "10.0.1.20"
}
```

---

Terraform Provides:

```hcl
each.key

each.value
```

---

Output:

```text
each.key   → frontend

each.value → 10.0.1.10
```

---

# Dynamic Blocks

Used for:

```text
Repeated Nested Blocks
```

Example:

```text
Security Group Ingress Rules
```

---

Syntax:

```hcl
dynamic "ingress" {

  for_each = var.ports

  content {

    from_port = ingress.value

  }

}
```

---

# Functions

Terraform provides many built-in functions.

Common examples:

```text
length()

merge()

lookup()

join()

split()

upper()

lower()

toset()
```

---

# Example: merge()

```hcl
merge(
  {a="b", c="d"},
  {e="f", c="z"}
)
```

Output:

```hcl
{
  a = "b"

  c = "z"

  e = "f"
}
```

---

# Example: lookup()

```hcl
lookup(map, key, default)
```

Returns:

```text
Map Value

Or Default Value
```

---

# Data Sources

Used to:

```text
Query Existing Information

From Provider
```

---

Syntax:

```hcl
data "resource_type" "name" {

}
```

---

Example

```hcl
data "aws_ami" "latest" {

}
```

---

Purpose:

```text
Fetch Existing AMI

Fetch Existing VPC

Fetch Existing Route53 Zone
```

---

# Locals

Locals are reusable values inside Terraform.

---

Syntax

```hcl
locals {

  name = value

}
```

---

Access:

```hcl
local.name
```

---

# Advantages of Locals

```text
Cannot Be Overridden

Can Use Variables

Can Store Functions

Can Store Expressions
```

---

Example

```hcl
locals {

  resource_name = "roboshop-${var.environment}-catalogue"

}
```

---

Output:

```text
roboshop-dev-catalogue
```

---

# State

Terraform uses state to compare:

```text
Desired Infrastructure

Actual Infrastructure

State File
```

---

State File:

```text
terraform.tfstate
```

---

Purpose:

```text
Track Resources

Detect Changes

Perform CRUD Operations
```

---

# State Comparison

```text
Desired Infrastructure
         ↓

Terraform State
         ↓

Actual Infrastructure
```

---

# Infrastructure Drift

Occurs when:

```text
Actual Infrastructure

≠

Desired Infrastructure
```

Usually caused by:

```text
Manual AWS Console Changes
```

---

# Remote State

State should not remain on a developer's laptop in team environments.

---

Problems with Local State

```text
No Collaboration

Duplicate Resource Creation

State Loss

Security Risks
```

---

Solution

```text
Remote State
```

---

Benefits

```text
Centralized State

Collaboration

State Locking

Security
```

---

# State Locking

Prevents:

```text
Multiple Terraform Operations

At The Same Time
```

---

Example

Developer A:

```bash
terraform apply
```

Developer B:

```bash
terraform apply
```

Without locking:

```text
State Corruption
```

---

# Provisioners

Provisioners execute:

```text
Scripts

Commands

Programs
```

during resource creation.

---

# local-exec

Runs commands on:

```text
Terraform Machine
```

---

Syntax

```hcl
provisioner "local-exec" {

  command = ""

}
```

---

Advanced Example

```hcl
provisioner "local-exec" {

  command = ""

  on_failure = continue

  when = destroy

}
```

---

# local-exec Use Cases

```text
Generate Inventory

Call APIs

Run Ansible

Trigger Automation
```

---

# remote-exec

Runs commands on:

```text
Remote Server
```

after creation.

---

Example

```hcl
provisioner "remote-exec" {

  inline = [

    "sudo dnf install nginx -y",

    "sudo systemctl start nginx"

  ]

}
```

---

Purpose

```text
Configure Server

Install Packages

Execute Commands
```

---

# Terraform + Ansible

Typical Workflow

```text
Terraform
      ↓
Create EC2
      ↓
Generate Inventory
      ↓
Run Ansible
      ↓
Configure Servers
```

---

Example

```hcl
provisioner "local-exec" {

  command = "ansible-playbook -i inventory playbook.yaml"

}
```

---

# Frequently Asked Interview Questions

## What is Terraform?

### Short Answer

Terraform is an Infrastructure as Code tool used to provision and manage cloud infrastructure using declarative configuration files.

### Detailed Explanation

Terraform enables teams to define infrastructure in code, version it using Git, and deploy it consistently across environments. It supports multiple cloud providers and automatically determines the changes required to achieve the desired state.

### Production Example

Using Terraform to create VPCs, EC2 instances, Security Groups, Route53 records, and EKS clusters in AWS.

### Follow-Up Questions

- What is Infrastructure as Code?
- Why is Terraform declarative?
- What providers have you used?

---

## What is Terraform State?

### Short Answer

Terraform State is Terraform's memory that tracks resources it manages.

### Detailed Explanation

State stores infrastructure metadata and allows Terraform to compare desired infrastructure with actual infrastructure to determine required changes.

### Production Example

Tracking EC2 instance IDs and Security Group IDs to manage future updates.

### Follow-Up Questions

- What is terraform.tfstate?
- What is remote state?
- Why is state important?

---

## Difference Between Count and For_Each?

### Short Answer

Count uses numeric indexes, while for_each uses keys and values.

### Detailed Explanation

Count is ideal for identical resources, while for_each is better for uniquely identified resources such as DNS records or IAM users.

### Production Example

Using count for multiple EC2 instances and for_each for Route53 records.

### Follow-Up Questions

- What is count.index?
- What are each.key and each.value?

---

# Quick Revision Cheat Sheet

```text
Variables        → Reusable Inputs

Locals           → Reusable Internal Logic

Count            → Loop Using Numbers

For_Each         → Loop Using Maps/Sets

Dynamic Block    → Loop Inside Nested Blocks

Functions        → Data Manipulation

Data Sources     → Query Existing Resources

State            → Terraform Memory

Remote State     → Shared State Storage

Provisioners     → Run Commands/Scripts
```