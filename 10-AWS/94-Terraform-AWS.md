# Terraform AWS Cookbook

---

# Introduction

Terraform is an Infrastructure as Code (IaC) tool developed by HashiCorp that enables you to provision, manage, and version cloud infrastructure using declarative configuration files.

Common Use Cases

- AWS Infrastructure Provisioning
- Infrastructure Automation
- CI/CD Integration
- Multi-Cloud Deployments
- GitOps Infrastructure
- Disaster Recovery
- Immutable Infrastructure

---

# Install Terraform

Linux

```bash
wget https://releases.hashicorp.com/terraform/
```

macOS

```bash
brew install terraform
```

Windows

Download Terraform from HashiCorp and add it to the PATH.

---

# Verify Installation

```bash
terraform version
```

---

# Terraform Workflow

```text
Write Code
      ↓
terraform init
      ↓
terraform validate
      ↓
terraform plan
      ↓
terraform apply
      ↓
terraform destroy
```

---

# Project Structure

```text
terraform-project/

├── provider.tf
├── variables.tf
├── outputs.tf
├── main.tf
├── terraform.tfvars
├── backend.tf
├── versions.tf
└── modules/
```

---

# Configure AWS Provider

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

---

# Specify Provider Version

```hcl
terraform {

  required_providers {

    aws = {

      source  = "hashicorp/aws"
      version = "~> 5.0"

    }

  }

}
```

---

# Configure AWS Profile

```hcl
provider "aws" {

  profile = "production"

  region  = "ap-south-1"

}
```

---

# Configure Multiple Providers

```hcl
provider "aws" {

  alias  = "mumbai"

  region = "ap-south-1"

}

provider "aws" {

  alias  = "singapore"

  region = "ap-southeast-1"

}
```

---

# Terraform Block

```hcl
terraform {

  required_version = ">= 1.7"

}
```

---

# Local Values

```hcl
locals {

  environment = "production"

  owner = "DevOps"

}
```

---

# Variables

```hcl
variable "instance_type" {

  default = "t3.micro"

}
```

---

# terraform.tfvars

```hcl
instance_type = "t3.small"
```

---

# Outputs

```hcl
output "instance_id" {

  value = aws_instance.web.id

}
```

---

# Create EC2 Instance

```hcl
resource "aws_instance" "web" {

  ami           = "ami-xxxxxxxx"

  instance_type = "t3.micro"

}
```

---

# Add Tags

```hcl
resource "aws_instance" "web" {

  ami           = "ami-xxxxxxxx"

  instance_type = "t3.micro"

  tags = {

    Name = "Production"

    Environment = "Prod"

  }

}
```

---

# EC2 with Key Pair

```hcl
resource "aws_instance" "web" {

  ami           = "ami-xxxxxxxx"

  instance_type = "t3.micro"

  key_name = "devops-key"

}
```

---

# Attach Security Group

```hcl
vpc_security_group_ids = [

  aws_security_group.web.id

]
```

---

# Attach IAM Role

```hcl
iam_instance_profile = aws_iam_instance_profile.ec2.name
```

---

# User Data

```hcl
user_data = file("userdata.sh")
```

---

# Root Volume

```hcl
root_block_device {

  volume_size = 30

  volume_type = "gp3"

  encrypted = true

}
```

---

# Additional EBS Volume

```hcl
resource "aws_ebs_volume" "data" {

  availability_zone = "ap-south-1a"

  size = 100

}
```

---

# Attach EBS Volume

```hcl
resource "aws_volume_attachment" "attach" {

  device_name = "/dev/xvdf"

  volume_id = aws_ebs_volume.data.id

  instance_id = aws_instance.web.id

}
```

---

# Elastic IP

```hcl
resource "aws_eip" "web" {

  instance = aws_instance.web.id

}
```

---

# Data Source

Latest Amazon Linux

```hcl
data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = ["amazon"]

  filter {

    name = "name"

    values = [

      "al2023-ami-*"

    ]

  }

}
```

---

# Use Data Source

```hcl
ami = data.aws_ami.amazon_linux.id
```

---

# Multiple EC2 Instances

```hcl
resource "aws_instance" "servers" {

  count = 3

  ami = data.aws_ami.amazon_linux.id

  instance_type = "t3.micro"

}
```

---

# for_each Example

```hcl
resource "aws_instance" "servers" {

  for_each = toset([

    "app",

    "api",

    "worker"

  ])

  ami = data.aws_ami.amazon_linux.id

  instance_type = "t3.micro"

  tags = {

    Name = each.key

  }

}
```

---

# Conditional Expression

```hcl
instance_type = var.environment == "prod"

? "t3.large"

: "t3.micro"
```

---

# Dependencies

```hcl
depends_on = [

  aws_security_group.web

]
```

---

# Lifecycle Rules

```hcl
lifecycle {

  create_before_destroy = true

}
```

---

# Ignore Changes

```hcl
lifecycle {

  ignore_changes = [

    tags

  ]

}
```

---

# Prevent Destroy

```hcl
lifecycle {

  prevent_destroy = true

}
```

---

# Common Terraform Commands

Initialize

```bash
terraform init
```

Validate

```bash
terraform validate
```

Format

```bash
terraform fmt
```

Plan

```bash
terraform plan
```

Apply

```bash
terraform apply
```

Destroy

```bash
terraform destroy
```

---

# State Commands

Show State

```bash
terraform show
```

List Resources

```bash
terraform state list
```

Show Resource

```bash
terraform state show aws_instance.web
```

Remove State

```bash
terraform state rm aws_instance.web
```

Import Resource

```bash
terraform import aws_instance.web i-0123456789abcdef0
```

---

# Best Practices

- Pin provider versions.
- Store state remotely.
- Use variables instead of hardcoding values.
- Tag every resource.
- Use data sources whenever possible.
- Keep reusable resources in modules.
- Separate environments using workspaces or separate state files.
- Always review `terraform plan` before applying changes.

---

# Summary

This section covered Terraform fundamentals, providers, variables, outputs, EC2 provisioning, EBS volumes, Elastic IPs, data sources, lifecycle rules, state management, and essential Terraform commands. These concepts form the foundation for building production-ready AWS infrastructure with Terraform.

---

