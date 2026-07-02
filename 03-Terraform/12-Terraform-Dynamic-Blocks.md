# Terraform Dynamic Blocks

## Introduction

So far we have learned multiple ways to create resources dynamically:

```text
count

for_each

locals
```

These work well when creating:

```text
EC2 Instances

S3 Buckets

Route53 Records
```

However, some Terraform resources contain:

```text
Nested Blocks
```

Examples:

```text
Security Group Ingress Rules

Security Group Egress Rules

Load Balancer Listeners

EBS Volumes

Network Interfaces
```

For such cases, Terraform provides:

```text
Dynamic Blocks
```

---

# Why Do We Need Dynamic Blocks?

Suppose we want a Security Group with:

```text
22

80

443

8080

3306

27017

6379

5672
```

ports open.

Without Dynamic Blocks:

```hcl
ingress {

  from_port = 22

  to_port = 22

  protocol = "tcp"

}
```

Repeated multiple times.

This leads to:

```text
Code Duplication

Poor Maintainability

Large Terraform Files
```

---

# What is a Dynamic Block?

A Dynamic Block allows Terraform to generate:

```text
Nested Blocks Dynamically
```

using loops.

Think of it as:

```text
for_each

Inside A Resource Block
```

---

# Dynamic Block Syntax

```hcl
dynamic "block_name" {

  for_each = collection

  content {

    block_configuration

  }

}
```

---

# Understanding the Structure

```hcl
dynamic "ingress"
```

means:

```text
Generate Multiple Ingress Blocks
```

---

```hcl
for_each
```

controls:

```text
How Many Blocks To Generate
```

---

```hcl
content {}
```

contains:

```text
Actual Block Configuration
```

---

# Basic Example

Variable:

```hcl
variable "ports" {

  default = [

    22,

    80,

    443

  ]

}
```

---

Security Group:

```hcl
resource "aws_security_group" "allow_ports" {

  name = "allow-ports"

  dynamic "ingress" {

    for_each = var.ports

    content {

      from_port = ingress.value

      to_port = ingress.value

      protocol = "tcp"

      cidr_blocks = ["0.0.0.0/0"]

    }

  }

}
```

---

# Result

Terraform generates:

```hcl
ingress {

  from_port = 22

  to_port = 22

}
```

---

```hcl
ingress {

  from_port = 80

  to_port = 80

}
```

---

```hcl
ingress {

  from_port = 443

  to_port = 443

}
```

Automatically.

---

# How ingress.value Works

Terraform automatically creates:

```hcl
ingress.value
```

for the current iteration.

Example:

```text
Iteration 1 → 22

Iteration 2 → 80

Iteration 3 → 443
```

---

# Real Security Group Example

Variable:

```hcl
variable "allowed_ports" {

  default = [

    22,

    80,

    3306,

    6379,

    8080

  ]

}
```

---

Security Group:

```hcl
resource "aws_security_group" "roboshop" {

  name = "roboshop"

  dynamic "ingress" {

    for_each = var.allowed_ports

    content {

      from_port = ingress.value

      to_port = ingress.value

      protocol = "tcp"

      cidr_blocks = ["10.0.0.0/16"]

    }

  }

}
```

---

Generated Rules

```text
22

80

3306

6379

8080
```

Each port receives its own ingress rule.

---

# Using Dynamic Blocks with for_each

Dynamic blocks internally use:

```text
for_each
```

to iterate over collections.

Examples:

```text
List

Set

Map
```

---

# Dynamic Block with Map

Variable:

```hcl
variable "ports" {

  default = {

    ssh   = 22

    http  = 80

    https = 443

  }

}
```

---

Security Group:

```hcl
dynamic "ingress" {

  for_each = var.ports

  content {

    from_port = ingress.value

    to_port = ingress.value

    protocol = "tcp"

  }

}
```

---

# Dynamic Block vs Count

## Count

Used for:

```text
Multiple Resources
```

Example:

```text
EC2 Instances
```

---

## Dynamic Blocks

Used for:

```text
Multiple Nested Blocks
```

Example:

```text
Ingress Rules
```

---

# Dynamic Block vs for_each

## for_each

Creates:

```text
Multiple Resources
```

---

## Dynamic Block

Creates:

```text
Multiple Child Blocks
```

inside a resource.

---

# Real Production Example

Suppose a microservices platform requires:

```text
SSH     → 22

HTTP    → 80

HTTPS   → 443

MySQL   → 3306

MongoDB → 27017

Redis   → 6379

RabbitMQ → 5672
```

Without dynamic blocks:

```text
7 Separate Ingress Blocks
```

With dynamic blocks:

```text
Single Reusable Configuration
```

---

# Roboshop Example

Variable:

```hcl
variable "allowed_ports" {

  default = [

    22,

    80,

    3306,

    27017,

    6379,

    5672

  ]

}
```

---

Security Group:

```hcl
resource "aws_security_group" "allow_all" {

  dynamic "ingress" {

    for_each = var.allowed_ports

    content {

      from_port = ingress.value

      to_port = ingress.value

      protocol = "tcp"

      cidr_blocks = ["10.0.0.0/16"]

    }

  }

}
```

---

Benefits:

```text
Reusable

Scalable

Maintainable
```

---

# Common Mistakes

## Using Count Instead of Dynamic Blocks

Count creates resources.

Dynamic blocks create nested blocks.

---

## Hardcoding Multiple Ingress Rules

Bad:

```text
Many Repeated Ingress Blocks
```

---

## Forgetting ingress.value

Bad:

```hcl
from_port = port
```

Correct:

```hcl
from_port = ingress.value
```

---

## Complex Nested Dynamic Blocks

Can reduce readability.

Use carefully.

---

# Best Practices

## Use Dynamic Blocks for Repeated Nested Blocks

---

## Store Values in Variables

Avoid hardcoded ports.

---

## Keep Dynamic Blocks Simple

Readable code is maintainable code.

---

## Use Lists for Port Definitions

Example:

```hcl
variable "allowed_ports"
```

---

# Frequently Asked Interview Questions

## What is a Dynamic Block?

### Short Answer

A Dynamic Block is used to generate repeated nested blocks inside a Terraform resource.

### Detailed Explanation

Dynamic blocks allow Terraform to create child blocks dynamically using loops. They are commonly used when resources contain repeated nested configurations such as ingress rules, listener rules, or volume definitions.

### Production Example

Creating multiple Security Group ingress rules from a list of allowed ports.

### Follow-Up Questions

- Why not use count?
- How does ingress.value work?
- When should dynamic blocks be used?

---

## Why Do We Need Dynamic Blocks?

### Short Answer

To avoid repeating nested configuration blocks.

### Detailed Explanation

Many Terraform resources contain child blocks that must be repeated multiple times. Dynamic blocks automate this process and make configurations easier to maintain.

### Production Example

Generating dozens of ingress rules from a list of ports.

### Follow-Up Questions

- Can dynamic blocks use variables?
- Can they iterate over maps?

---

## Difference Between Count and Dynamic Blocks?

### Short Answer

Count creates resources, while Dynamic Blocks create nested blocks.

### Detailed Explanation

Count is used when multiple resources need to be created. Dynamic blocks are used when multiple child configurations need to be generated within a single resource.

### Production Example

Count for EC2 instances and Dynamic Blocks for Security Group rules.

### Follow-Up Questions

- Can both be used together?
- Which is better for Security Groups?

---

## What is ingress.value?

### Short Answer

ingress.value represents the current value being processed inside a dynamic ingress block.

### Detailed Explanation

Terraform automatically creates this variable during iteration. It allows access to the current port or configuration value being processed.

### Production Example

If iterating over:

```text
22

80

443
```

then ingress.value becomes:

```text
22 → 80 → 443
```

during each iteration.

### Follow-Up Questions

- Is ingress.value available outside the dynamic block?
- How does it differ from each.value?

---

## Where Are Dynamic Blocks Commonly Used?

### Short Answer

In resources containing repeated nested blocks.

### Detailed Explanation

Dynamic blocks are frequently used for Security Groups, Load Balancer listeners, EBS configurations, and network-related resources.

### Production Example

Generating ingress rules for all application ports in a microservices platform.

### Follow-Up Questions

- Can dynamic blocks use maps?
- Can dynamic blocks be nested?

---

# Key Takeaways

- Dynamic Blocks generate repeated nested blocks.
- They are commonly used for Security Group rules.
- Dynamic Blocks use for_each internally.
- ingress.value provides access to the current iteration value.
- Dynamic Blocks reduce duplication and improve maintainability.
- Count creates resources, Dynamic Blocks create child blocks.
- Dynamic Blocks are widely used in production Terraform projects.