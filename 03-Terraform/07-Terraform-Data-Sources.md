# Terraform Data Sources

## Introduction

So far, we have used Terraform to:

```text
Create Resources

Update Resources

Delete Resources
```

using resource blocks.

Example:

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = "t3.micro"

}
```

This creates new infrastructure.

---

# What is a Data Source?

A Data Source is used to:

```text
Read Existing Information

Query Existing Resources

Fetch Provider Data
```

without creating anything.

---

# Resource vs Data Source

## Resource

Used for:

```text
Create

Update

Delete
```

infrastructure.

Example:

```hcl
resource "aws_instance" "frontend" {

}
```

Creates a new EC2 instance.

---

## Data Source

Used for:

```text
Read Existing Information
```

Example:

```hcl
data "aws_ami" "example" {

}
```

Fetches information about an existing AMI.

---

# Why Do We Need Data Sources?

In real projects, not everything is created by Terraform.

Many resources already exist:

```text
VPC

Subnets

Security Groups

AMIs

Route53 Zones

IAM Roles
```

Instead of hardcoding values, we can query them dynamically.

---

# Data Source Syntax

```hcl
data "resource_type" "name" {

}
```

---

# Example

```hcl
data "aws_ami" "joindevops" {

}
```

---

# Real-World Scenario

Suppose:

```text
DevOps Team Creates Golden AMI

Terraform Should Use Latest AMI
```

Hardcoding:

```hcl
ami = "ami-123456"
```

is not a good practice.

Because:

```text
AMI Changes

Infrastructure Breaks

Manual Updates Required
```

---

# Query Existing AMI

Example:

```hcl
data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = ["amazon"]

}
```

---

# Using Data Source Output

Example:

```hcl
resource "aws_instance" "frontend" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = "t3.micro"

}
```

---

# Flow

```text
Terraform
      ↓
Query AWS
      ↓
Get Latest AMI
      ↓
Launch EC2 Instance
```

---

# Understanding Data Source References

Syntax:

```hcl
data.resource_type.resource_name.attribute
```

Example:

```hcl
data.aws_ami.amazon_linux.id
```

Meaning:

```text
data

aws_ami

amazon_linux

id
```

---

# Query Existing VPC

Example:

```hcl
data "aws_vpc" "default" {

  default = true

}
```

---

# Using VPC Information

```hcl
data.aws_vpc.default.id
```

Returns:

```text
vpc-xxxxxxxx
```

---

# Query Existing Route53 Zone

Example:

```hcl
data "aws_route53_zone" "main" {

  name = "daws86s.fun"

}
```

---

# Use Hosted Zone ID

```hcl
data.aws_route53_zone.main.zone_id
```

---

# Query Existing Security Group

Example:

```hcl
data "aws_security_group" "allow_all" {

  name = "allow-all"

}
```

---

# Query Existing Subnet

Example:

```hcl
data "aws_subnet" "public" {

  id = "subnet-123456"
}
```

---

# Commonly Used Data Sources

```text
aws_ami

aws_vpc

aws_subnet

aws_security_group

aws_route53_zone

aws_caller_identity

aws_region
```

---

# AWS Caller Identity

Useful for obtaining account information.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

---

# Usage

```hcl
data.aws_caller_identity.current.account_id
```

Output:

```text
123456789012
```

---

# AWS Region

Example:

```hcl
data "aws_region" "current" {}
```

---

Usage:

```hcl
data.aws_region.current.name
```

Output:

```text
us-east-1
```

---

# Production Example

Suppose Terraform creates:

```text
EC2

Route53 Records

Security Groups
```

Instead of hardcoding:

```hcl
zone_id = "Z0123456789"
```

Use:

```hcl
data.aws_route53_zone.main.zone_id
```

Benefits:

```text
Reusable

Environment Independent

Less Maintenance
```

---

# Roboshop Example

Query Existing Hosted Zone:

```hcl
data "aws_route53_zone" "main" {

  name = "daws86s.fun"

}
```

---

Create Record:

```hcl
resource "aws_route53_record" "frontend" {

  zone_id = data.aws_route53_zone.main.zone_id

  name    = "frontend"

}
```

---

# Data Source Dependency

Terraform automatically understands:

```text
Resource Depends On Data Source
```

Example:

```hcl
resource "aws_instance" "frontend" {

  ami = data.aws_ami.amazon_linux.id

}
```

Terraform fetches AMI information first.

---

# Common Mistakes

## Hardcoding IDs

Bad:

```hcl
subnet_id = "subnet-123456"
```

Prefer:

```hcl
data.aws_subnet.public.id
```

---

## Assuming Resources Exist

If queried resources do not exist:

```text
Terraform Fails
```

---

## Wrong Filters

Incorrect filters may return:

```text
No Results

Multiple Results
```

---

# Best Practices

## Use Data Sources for Existing Resources

---

## Avoid Hardcoding IDs

---

## Query Latest AMIs Dynamically

---

## Use Route53 Data Sources

Instead of hardcoded Hosted Zone IDs.

---

## Validate Filters Carefully

Ensure correct resources are returned.

---

# Frequently Asked Interview Questions

## What is a Data Source?

### Short Answer

A Data Source is used to query and retrieve information about existing resources without creating them.

### Detailed Explanation

Terraform data sources allow infrastructure code to consume information from resources that already exist in the provider. They help avoid hardcoding IDs, names, and configuration details, making Terraform code more reusable and maintainable.

### Production Example

Fetching the latest Amazon Linux AMI dynamically before launching EC2 instances.

### Follow-Up Questions

- Difference between resource and data source?
- Can data sources create resources?
- Why are data sources important?

---

## Difference Between Resource and Data Source?

### Short Answer

Resources create infrastructure, while data sources read existing infrastructure.

### Detailed Explanation

A resource block manages the lifecycle of infrastructure. A data source only retrieves information from existing infrastructure and does not create or modify anything.

### Production Example

Using a data source to retrieve a Hosted Zone ID and a resource block to create Route53 records.

### Follow-Up Questions

- Can a resource use data source output?
- Can data sources depend on resources?

---

## Why Use Data Sources?

### Short Answer

To avoid hardcoding values and dynamically fetch existing infrastructure information.

### Detailed Explanation

Data sources improve reusability and maintainability by allowing Terraform to query resources such as AMIs, VPCs, subnets, and Route53 zones directly from the provider.

### Production Example

Fetching the latest approved AMI automatically during EC2 creation.

### Follow-Up Questions

- Which data sources do you use frequently?
- What happens if the resource doesn't exist?

---

## How Do You Reference a Data Source?

### Short Answer

Using:

```hcl
data.resource_type.resource_name.attribute
```

### Detailed Explanation

Terraform exposes attributes from queried resources. These attributes can be consumed by resource blocks and outputs.

### Production Example

```hcl
data.aws_route53_zone.main.zone_id
```

### Follow-Up Questions

- What attributes are available?
- How do you discover them?

---

## Which Data Sources Are Commonly Used in AWS?

### Short Answer

```text
aws_ami

aws_vpc

aws_subnet

aws_route53_zone

aws_security_group

aws_caller_identity
```

### Detailed Explanation

These data sources help retrieve infrastructure information dynamically and are frequently used in enterprise Terraform projects.

### Production Example

Querying VPC, subnet, and AMI information before creating EC2 instances.

---

# Key Takeaways

- Data sources query existing infrastructure.
- They do not create resources.
- Syntax:

```hcl
data "resource_type" "name" {}
```

- Data sources help eliminate hardcoded values.
- Common examples include AMIs, VPCs, Route53 zones, and subnets.
- Resources and data sources are often used together.
- Data sources are heavily used in production Terraform environments.