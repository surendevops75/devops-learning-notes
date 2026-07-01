# Terraform Conditional Expressions

## Introduction

In real-world projects, infrastructure requirements differ based on:

```text
Environment

Application

Region

Business Requirements
```

Examples:

```text
DEV → t3.micro

QA → t3.small

PROD → t3.large
```

Instead of maintaining multiple code versions, Terraform provides:

```text
Conditional Expressions
```

---

# Why Do We Need Conditions?

Suppose:

DEV:

```text
t3.micro
```

PROD:

```text
t3.large
```

Without conditions:

```text
Separate Terraform Files

Code Duplication

Maintenance Issues
```

With conditions:

```text
Single Reusable Code Base
```

---

# Traditional If Else

Most programming languages use:

```text
if (condition) {

  do this

}

else {

  do that

}
```

Example:

```text
if environment == prod

use t3.large

else

use t3.micro
```

---

# Terraform Conditional Syntax

Terraform uses:

```hcl
condition ? true_value : false_value
```

---

# Structure

```hcl
expression ? value_if_true : value_if_false
```

---

# Example

```hcl
var.environment == "prod" ? "t3.large" : "t3.micro"
```

Meaning:

```text
If Environment Is PROD

Use t3.large

Otherwise

Use t3.micro
```

---

# Visual Representation

```text
Condition
    │
    ▼
TRUE ?
 │     │
 ▼     ▼
Yes    No
 │     │
 ▼     ▼
Value1 Value2
```

---

# Example 1

Variable:

```hcl
variable "environment" {

  default = "dev"

}
```

Resource:

```hcl
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
```

---

Result

If:

```text
environment = prod
```

Output:

```text
t3.large
```

---

If:

```text
environment = dev
```

Output:

```text
t3.micro
```

---

# Example 2

Storage Size

```hcl
root_block_device {

  volume_size = var.environment == "prod" ? 100 : 20

}
```

---

Result

PROD:

```text
100 GB
```

DEV:

```text
20 GB
```

---

# Example 3

Enable Monitoring

```hcl
monitoring = var.environment == "prod" ? true : false
```

---

Output

PROD:

```text
true
```

---

DEV:

```text
false
```

---

# Example 4

Tags

```hcl
tags = {

  Backup = var.environment == "prod" ? "Enabled" : "Disabled"

}
```

---

# Multiple Conditions

Terraform supports nested conditions.

Example:

```hcl
var.environment == "prod"
?
"t3.large"
:
var.environment == "qa"
?
"t3.small"
:
"t3.micro"
```

---

Output

```text
PROD → t3.large

QA → t3.small

DEV → t3.micro
```

---

# Better Approach

Instead of many nested conditions:

Use maps.

Example:

```hcl
variable "instance_types" {

  default = {

    dev  = "t3.micro"

    qa   = "t3.small"

    prod = "t3.large"

  }

}
```

Usage:

```hcl
instance_type = var.instance_types[var.environment]
```

More readable.

---

# Conditional Resource Creation

One of the most common production use cases.

Example:

```hcl
count = var.environment == "prod" ? 1 : 0
```

Meaning:

```text
Create Resource In PROD

Do Not Create In DEV
```

---

# Example

```hcl
resource "aws_instance" "bastion" {

  count = var.environment == "prod" ? 1 : 0

}
```

---

Output

DEV:

```text
0 Bastion Servers
```

PROD:

```text
1 Bastion Server
```

---

# Real Production Example

Create Monitoring Server Only In PROD.

```hcl
resource "aws_instance" "monitoring" {

  count = var.environment == "prod" ? 1 : 0

}
```

---

Benefits:

```text
Cost Optimization

Cleaner Infrastructure

Environment Specific Resources
```

---

# Route53 Example

```hcl
ttl = var.environment == "prod" ? 300 : 60
```

---

Output

PROD:

```text
300 Seconds
```

DEV:

```text
60 Seconds
```

---

# Security Group Example

```hcl
cidr_blocks = var.environment == "prod"

? ["10.0.0.0/16"]

: ["0.0.0.0/0"]
```

---

# Interpolation With Conditions

Example:

```hcl
Name = "${var.environment}-frontend"
```

Output:

```text
dev-frontend

prod-frontend
```

---

Combined Example

```hcl
Name = "${var.environment == "prod" ? "prod" : "dev"}-frontend"
```

---

# Real Roboshop Example

Variable:

```hcl
variable "environment" {

  default = "dev"

}
```

---

Resource:

```hcl
resource "aws_instance" "frontend" {

  instance_type = var.environment == "prod"

  ? "t3.large"

  : "t3.micro"

}
```

---

Output

DEV:

```text
t3.micro
```

---

PROD:

```text
t3.large
```

---

# Common Mistakes

## Overusing Nested Conditions

Bad:

```hcl
a ? b : c ? d : e ? f : g
```

Hard to read.

---

Prefer:

```text
Maps

Locals
```

for complex logic.

---

## Using Conditions Everywhere

Use only when needed.

Keep code readable.

---

## Forgetting Data Types

Bad:

```hcl
var.env == "prod" ? "1" : 0
```

Different data types.

---

Good:

```hcl
var.env == "prod" ? 1 : 0
```

---

# Best Practices

## Use Conditions For Environment Differences

Examples:

```text
Instance Types

Storage Sizes

Monitoring

Backups

Tags
```

---

## Prefer Maps For Large Logic

Avoid deeply nested conditions.

---

## Keep Expressions Readable

Readable code is maintainable code.

---

## Use Conditional Resource Creation Carefully

Example:

```hcl
count = condition ? 1 : 0
```

Very common in production.

---

# Frequently Asked Interview Questions

## What Is the Terraform Conditional Expression Syntax?

```hcl
condition ? true_value : false_value
```

---

## Why Are Conditional Expressions Used?

To create environment-specific behavior without duplicating code.

---

## Give a Real Example.

```hcl
instance_type = var.environment == "prod"

? "t3.large"

: "t3.micro"
```

---

## How Do You Create a Resource Only in PROD?

```hcl
count = var.environment == "prod" ? 1 : 0
```

---

## What Is Better Than Multiple Nested Conditions?

Using:

```text
Maps

Locals
```

for cleaner code.

---

## Can Conditional Expressions Return Different Data Types?

They should return compatible data types to avoid errors.

---

# Key Takeaways

- Terraform uses conditional expressions instead of traditional if/else blocks.
- Syntax:
  
  ```hcl
  condition ? true_value : false_value
  ```

- Conditions help create reusable infrastructure.
- Common use cases include instance sizing, monitoring, backups, and resource creation.
- `count = condition ? 1 : 0` is a very common production pattern.
- Avoid deeply nested conditions.
- Prefer maps and locals for complex logic.
- Conditional expressions are frequently asked in Terraform interviews.