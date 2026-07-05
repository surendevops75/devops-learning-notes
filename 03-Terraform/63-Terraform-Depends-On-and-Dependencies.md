# Terraform Depends_On and Dependencies

## Introduction

Terraform creates infrastructure by understanding:

```text
Resource Relationships

Dependencies

Creation Order
```

---

When resources depend on each other, Terraform automatically determines:

```text
What To Create First

What To Create Next

What To Destroy Last
```

---

This process is called:

```text
Dependency Management
```

---

# What is a Dependency?

A dependency exists when:

```text
One Resource Requires Another Resource
```

before it can be created.

---

# Example

Security Group:

```text
Needs VPC
```

---

Architecture

```text
VPC
 ↓
Security Group
```

---

Terraform must create:

```text
VPC First
```

---

Then:

```text
Security Group
```

---

# Why Dependencies Matter?

Without dependencies:

```text
Terraform Might Create Resources
In The Wrong Order
```

---

Result:

```text
Creation Failures

API Errors

Broken Infrastructure
```

---

# Implicit Dependencies

Terraform automatically detects many dependencies.

---

Example

```hcl
resource "aws_vpc" "main" {

}

resource "aws_security_group" "catalogue" {

  vpc_id = aws_vpc.main.id

}
```

---

Terraform notices:

```text
aws_security_group.catalogue
```

uses:

```text
aws_vpc.main.id
```

---

Dependency Created Automatically.

---

# Dependency Graph

Terraform internally builds:

```text
Dependency Graph
```

---

Example

```text
VPC
 ↓
Subnet
 ↓
EC2
```

---

Terraform Creation Order

```text
VPC
 ↓
Subnet
 ↓
EC2
```

---

Terraform Destroy Order

```text
EC2
 ↓
Subnet
 ↓
VPC
```

---

# What is depends_on?

Sometimes Terraform cannot automatically identify dependencies.

---

In such situations:

```text
depends_on
```

is used.

---

Purpose:

```text
Create Explicit Dependency
```

---

# Syntax

```hcl
depends_on = [
  RESOURCE
]
```

---

# Example

Internet Gateway

```hcl
resource "aws_internet_gateway" "main" {

}
```

---

NAT Gateway

```hcl
resource "aws_nat_gateway" "nat" {

  depends_on = [
    aws_internet_gateway.main
  ]

}
```

---

# Why?

Even if NAT does not directly reference:

```text
Internet Gateway
```

---

Operationally:

```text
IGW Must Exist First
```

---

# Flow

```text
Internet Gateway
       ↓
NAT Gateway
```

---

# Common AWS Example

VPC

↓

Internet Gateway

↓

Elastic IP

↓

NAT Gateway

---

Creation Order:

```text
VPC
 ↓
IGW
 ↓
EIP
 ↓
NAT
```

---

# Example from Production

```hcl
resource "aws_nat_gateway" "nat" {

  allocation_id = aws_eip.nat.id

  depends_on = [
    aws_internet_gateway.main
  ]

}
```

---

Terraform ensures:

```text
IGW Exists
```

before creating:

```text
NAT Gateway
```

---

# When Terraform Fails Without depends_on

Suppose:

```text
Terraform Starts Creating NAT
```

---

But:

```text
IGW Not Ready Yet
```

---

Result:

```text
AWS API Error
```

---

# Explicit Dependency Architecture

```text
Resource A
      ↓
depends_on
      ↓
Resource B
```

---

Terraform creates:

```text
Resource B
```

first.

---

# Module Dependencies

depends_on can also be used with:

```text
Modules
```

---

Example

```hcl
module "sg" {

  depends_on = [
    module.vpc
  ]

}
```

---

Result:

```text
VPC Module
```

completed before:

```text
Security Group Module
```

---

# Multi Module Example

```text
00-vpc
    ↓
10-sg
    ↓
20-bastion
```

---

Terraform Order:

```text
VPC
 ↓
Security Groups
 ↓
Bastion
```

---

# Data Source Dependencies

Sometimes:

```text
Data Sources
```

need dependencies.

---

Example

```text
Create Resource
      ↓
Read Resource
```

---

depends_on ensures:

```text
Read Happens After Create
```

---

# Resource Dependency Types

## Implicit Dependency

Terraform Detects Automatically

Example:

```hcl
vpc_id = aws_vpc.main.id
```

---

## Explicit Dependency

Engineer Specifies Manually

Example:

```hcl
depends_on = [
  aws_internet_gateway.main
]
```

---

# Best Practice

Use:

```text
Implicit Dependencies
```

whenever possible.

---

Use:

```text
depends_on
```

only when Terraform cannot infer relationships.

---

# Why Avoid Excessive depends_on?

Too many explicit dependencies can:

```text
Slow Terraform

Create Complex Graphs

Reduce Parallelism
```

---

# Production Example

Infrastructure:

```text
VPC
 ↓
Subnets
 ↓
ALB
 ↓
ASG
```

---

Terraform automatically identifies most dependencies.

---

Additional:

```text
depends_on
```

used only where operational requirements exist.

---

# Example Architecture

```text
VPC
 ↓
Internet Gateway
 ↓
NAT Gateway
 ↓
Private Subnets
 ↓
EC2
```

---

Terraform Dependency Graph

```text
VPC
 ↓
IGW
 ↓
NAT
 ↓
Private Route
 ↓
EC2
```

---

# Common Interview Questions

## What is a Dependency in Terraform?

### Short Answer

A dependency is a relationship where one resource requires another resource to exist first.

### Detailed Explanation

Terraform uses dependencies to determine resource creation and destruction order.

### Production Example

Security Groups requiring a VPC.

### Follow-Up Questions

- How does Terraform identify dependencies?
- What is a dependency graph?

---

## What is depends_on?

### Short Answer

depends_on creates an explicit dependency between resources.

### Detailed Explanation

It instructs Terraform to create or process one resource only after another resource is completed.

### Production Example

NAT Gateway depending on Internet Gateway.

### Follow-Up Questions

- When should depends_on be used?
- Can modules use depends_on?

---

## What is the Difference Between Implicit and Explicit Dependencies?

### Short Answer

Implicit dependencies are automatically detected, while explicit dependencies are manually defined.

### Detailed Explanation

Terraform infers dependencies through references. depends_on is used when references are unavailable.

### Production Example

vpc_id creates an implicit dependency, while NAT → IGW uses explicit dependency.

### Follow-Up Questions

- Which approach is preferred?
- Why avoid excessive depends_on?

---

## Why Does Terraform Build a Dependency Graph?

### Short Answer

To determine correct resource creation and destruction order.

### Detailed Explanation

The graph helps Terraform create resources efficiently and safely.

### Production Example

Creating VPC before Security Groups and EC2 instances.

### Follow-Up Questions

- How does Terraform parallelize resources?
- What happens if dependencies are incorrect?

---

# Key Takeaways

```text
Dependencies determine resource creation order.

Terraform automatically detects most dependencies.

Implicit dependencies are created through resource references.

depends_on creates explicit dependencies.

depends_on should be used only when Terraform cannot infer relationships.

Terraform builds a dependency graph before execution.

Dependencies affect both creation and destruction order.

Modules can also use depends_on.

Proper dependency management prevents infrastructure deployment failures.

Understanding dependencies is essential for Terraform design and troubleshooting.
```