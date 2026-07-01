# Terraform For_Each Loops

## Introduction

In the previous section, we learned about:

```text
count

count.index
```

Count works very well when creating multiple similar resources using a list.

However, count has limitations:

```text
Index Based

Less Readable

Resource Reordering Issues
```

Terraform provides another looping mechanism:

```text
for_each
```

which is commonly used in production projects.

---

# Why Do We Need for_each?

Suppose we need to create:

```text
frontend

catalogue

user
```

servers.

Using count:

```hcl
count = 3
```

Terraform identifies resources using:

```text
0

1

2
```

instead of meaningful names.

---

# What is for_each?

for_each creates resources using:

```text
Map

Set

Collection
```

instead of numeric indexes.

---

# Count vs for_each

## Count

Works with:

```text
Numbers

Lists
```

Uses:

```hcl
count.index
```

---

## for_each

Works with:

```text
Maps

Sets
```

Uses:

```hcl
each.key

each.value
```

---

# Basic Syntax

```hcl
resource "resource_type" "name" {

  for_each = collection

}
```

---

# Using for_each with a Set

Example:

```hcl
for_each = toset([
  "frontend",
  "catalogue",
  "user"
])
```

---

# What is toset()?

Terraform cannot directly use a list with for_each.

Convert list to set:

```hcl
toset(list)
```

---

# Example

```hcl
resource "aws_s3_bucket" "buckets" {

  for_each = toset([
    "bucket-alpha",
    "bucket-beta",
    "bucket-gamma"
  ])

  bucket = each.value

}
```

---

Terraform Creates

```text
bucket-alpha

bucket-beta

bucket-gamma
```

---

# What is each.value?

When iterating through:

```hcl
toset([
  "frontend",
  "catalogue",
  "user"
])
```

Terraform provides:

```hcl
each.value
```

Values:

```text
frontend

catalogue

user
```

---

# List Iteration Using for_each

Variable:

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

Resource:

```hcl
resource "aws_instance" "servers" {

  for_each = toset(var.instances)

  ami = "ami-09c813fb71547fc4f"

  instance_type = "t3.micro"

  tags = {

    Name = each.value

  }

}
```

---

# What is a Map?

Map contains:

```text
Key

Value
```

Example:

```hcl
{
  frontend = "10.0.1.10"

  catalogue = "10.0.1.20"

  user = "10.0.1.30"
}
```

---

# Iterating Through Maps

Example:

```hcl
variable "instances" {

  default = {

    frontend = "10.0.1.10"

    catalogue = "10.0.1.20"

    user = "10.0.1.30"
  }

}
```

---

Resource:

```hcl
resource "aws_route53_record" "records" {

  for_each = var.instances

  name = each.key

  records = [each.value]

}
```

---

# each.key and each.value

Map:

```hcl
{
  frontend = "10.0.1.10"
}
```

Terraform provides:

```text
each.key   = frontend

each.value = 10.0.1.10
```

---

# Production Example

Roboshop DNS Records:

```hcl
{
  frontend  = "10.0.1.10"

  catalogue = "10.0.1.20"

  user      = "10.0.1.30"
}
```

---

Route53:

```hcl
resource "aws_route53_record" "records" {

  for_each = var.instances

  name = each.key

  records = [each.value]

}
```

Creates:

```text
frontend.daws86s.fun

catalogue.daws86s.fun

user.daws86s.fun
```

---

# Count vs For_Each

## Count

Uses:

```hcl
count.index
```

Example:

```text
0

1

2
```

---

## For_Each

Uses:

```hcl
each.key

each.value
```

Example:

```text
frontend

catalogue

user
```

More readable.

---

# When Should You Use Count?

Use count when:

```text
Resources Are Identical

Only Number Matters
```

Example:

```text
3 EC2 Instances
```

---

# When Should You Use For_Each?

Use for_each when:

```text
Resources Have Unique Names

Resources Have Different Values
```

Example:

```text
DNS Records

S3 Buckets

IAM Users
```

---

# Frequently Asked Interview Questions

## What is for_each?

### Short Answer

for_each is a Terraform meta-argument used to create resources from maps or sets.

### Detailed Explanation

Unlike count, which uses numeric indexes, for_each creates resources using meaningful keys and values. This makes infrastructure easier to understand and manage.

### Production Example

Creating Route53 records from a map of application names and IP addresses.

### Follow-Up Questions

- What is each.key?
- What is each.value?
- Difference between count and for_each?

---

## Can for_each Iterate Over a List?

### Short Answer

Not directly.

### Detailed Explanation

Terraform requires a set or map for for_each. Lists are typically converted using:

```hcl
toset(list)
```

### Production Example

Creating multiple S3 buckets from a list of names.

---

## Difference Between Count and For_Each?

### Short Answer

Count uses indexes, while for_each uses keys and values.

### Detailed Explanation

Count is better for identical resources. for_each is better for uniquely identified resources.

### Production Example

Count for EC2 instances, for_each for Route53 records.

---

# Key Takeaways

- for_each works with maps and sets.
- each.key and each.value are available during iteration.
- Lists can be converted using toset().
- for_each provides better readability than count.
- Commonly used for Route53, IAM users, S3 buckets, and DNS records.
- Understanding for_each is important for Terraform interviews and production projects.
