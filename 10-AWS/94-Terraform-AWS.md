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

# Amazon VPC

---

# Create VPC

```hcl
resource "aws_vpc" "main" {

  cidr_block = "10.0.0.0/16"

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "production-vpc"

  }

}
```

---

# DHCP Options

```hcl
resource "aws_vpc_dhcp_options" "main" {

  domain_name = "ec2.internal"

  domain_name_servers = [

    "AmazonProvidedDNS"

  ]

}
```

---

# Associate DHCP Options

```hcl
resource "aws_vpc_dhcp_options_association" "main" {

  vpc_id = aws_vpc.main.id

  dhcp_options_id = aws_vpc_dhcp_options.main.id

}
```

---

# Internet Gateway

```hcl
resource "aws_internet_gateway" "igw" {

  vpc_id = aws_vpc.main.id

  tags = {

    Name = "production-igw"

  }

}
```

---

# Public Subnet

```hcl
resource "aws_subnet" "public" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.1.0/24"

  availability_zone = "ap-south-1a"

  map_public_ip_on_launch = true

  tags = {

    Name = "public-subnet"

  }

}
```

---

# Private Subnet

```hcl
resource "aws_subnet" "private" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.2.0/24"

  availability_zone = "ap-south-1a"

  tags = {

    Name = "private-subnet"

  }

}
```

---

# Multiple Public Subnets

```hcl
resource "aws_subnet" "public" {

  count = 3

  vpc_id = aws_vpc.main.id

  cidr_block = cidrsubnet(

    aws_vpc.main.cidr_block,

    8,

    count.index

  )

  availability_zone = element(

    [

      "ap-south-1a",

      "ap-south-1b",

      "ap-south-1c"

    ],

    count.index

  )

  map_public_ip_on_launch = true

}
```

---

# Route Table

```hcl
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id

}
```

---

# Internet Route

```hcl
resource "aws_route" "internet" {

  route_table_id = aws_route_table.public.id

  destination_cidr_block = "0.0.0.0/0"

  gateway_id = aws_internet_gateway.igw.id

}
```

---

# Route Table Association

```hcl
resource "aws_route_table_association" "public" {

  subnet_id = aws_subnet.public.id

  route_table_id = aws_route_table.public.id

}
```

---

# Elastic IP

```hcl
resource "aws_eip" "nat" {

  domain = "vpc"

}
```

---

# NAT Gateway

```hcl
resource "aws_nat_gateway" "nat" {

  allocation_id = aws_eip.nat.id

  subnet_id = aws_subnet.public.id

  depends_on = [

    aws_internet_gateway.igw

  ]

}
```

---

# Private Route Table

```hcl
resource "aws_route_table" "private" {

  vpc_id = aws_vpc.main.id

}
```

---

# NAT Route

```hcl
resource "aws_route" "nat" {

  route_table_id = aws_route_table.private.id

  destination_cidr_block = "0.0.0.0/0"

  nat_gateway_id = aws_nat_gateway.nat.id

}
```

---

# Associate Private Route Table

```hcl
resource "aws_route_table_association" "private" {

  subnet_id = aws_subnet.private.id

  route_table_id = aws_route_table.private.id

}
```

---

# Security Group

```hcl
resource "aws_security_group" "web" {

  name = "web"

  vpc_id = aws_vpc.main.id

}
```

---

# SSH Rule

```hcl
resource "aws_vpc_security_group_ingress_rule" "ssh" {

  security_group_id = aws_security_group.web.id

  cidr_ipv4 = "0.0.0.0/0"

  from_port = 22

  to_port = 22

  ip_protocol = "tcp"

}
```

---

# HTTP Rule

```hcl
resource "aws_vpc_security_group_ingress_rule" "http" {

  security_group_id = aws_security_group.web.id

  cidr_ipv4 = "0.0.0.0/0"

  from_port = 80

  to_port = 80

  ip_protocol = "tcp"

}
```

---

# HTTPS Rule

```hcl
resource "aws_vpc_security_group_ingress_rule" "https" {

  security_group_id = aws_security_group.web.id

  cidr_ipv4 = "0.0.0.0/0"

  from_port = 443

  to_port = 443

  ip_protocol = "tcp"

}
```

---

# Egress Rule

```hcl
resource "aws_vpc_security_group_egress_rule" "all" {

  security_group_id = aws_security_group.web.id

  cidr_ipv4 = "0.0.0.0/0"

  ip_protocol = "-1"

}
```

---

# Network ACL

```hcl
resource "aws_network_acl" "public" {

  vpc_id = aws_vpc.main.id

}
```

---

# NACL Inbound Rule

```hcl
resource "aws_network_acl_rule" "http" {

  network_acl_id = aws_network_acl.public.id

  rule_number = 100

  egress = false

  protocol = "tcp"

  rule_action = "allow"

  cidr_block = "0.0.0.0/0"

  from_port = 80

  to_port = 80

}
```

---

# NACL Outbound Rule

```hcl
resource "aws_network_acl_rule" "egress" {

  network_acl_id = aws_network_acl.public.id

  rule_number = 100

  egress = true

  protocol = "-1"

  rule_action = "allow"

  cidr_block = "0.0.0.0/0"

  from_port = 0

  to_port = 0

}
```

---

# VPC Peering

```hcl
resource "aws_vpc_peering_connection" "peer" {

  vpc_id = aws_vpc.main.id

  peer_vpc_id = "vpc-xxxxxxxx"

  auto_accept = true

}
```

---

# Transit Gateway

```hcl
resource "aws_ec2_transit_gateway" "main" {

  description = "Production Transit Gateway"

}
```

---

# Transit Gateway Attachment

```hcl
resource "aws_ec2_transit_gateway_vpc_attachment" "main" {

  subnet_ids = [

    aws_subnet.private.id

  ]

  transit_gateway_id = aws_ec2_transit_gateway.main.id

  vpc_id = aws_vpc.main.id

}
```

---

# Elastic Network Interface

```hcl
resource "aws_network_interface" "eni" {

  subnet_id = aws_subnet.private.id

}
```

---

# Attach ENI

```hcl
resource "aws_network_interface_attachment" "attach" {

  instance_id = aws_instance.web.id

  network_interface_id = aws_network_interface.eni.id

  device_index = 1

}
```

---

# VPC Endpoint (Gateway)

```hcl
resource "aws_vpc_endpoint" "s3" {

  vpc_id = aws_vpc.main.id

  service_name = "com.amazonaws.ap-south-1.s3"

  vpc_endpoint_type = "Gateway"

  route_table_ids = [

    aws_route_table.private.id

  ]

}
```

---

# Interface Endpoint

```hcl
resource "aws_vpc_endpoint" "ssm" {

  vpc_id = aws_vpc.main.id

  service_name = "com.amazonaws.ap-south-1.ssm"

  vpc_endpoint_type = "Interface"

  subnet_ids = [

    aws_subnet.private.id

  ]

  security_group_ids = [

    aws_security_group.web.id

  ]

}
```

---

# Route53 Private Hosted Zone

```hcl
resource "aws_route53_zone" "private" {

  name = "internal.local"

  vpc {

    vpc_id = aws_vpc.main.id

  }

}
```

---

# Route53 Record

```hcl
resource "aws_route53_record" "app" {

  zone_id = aws_route53_zone.private.zone_id

  name = "app"

  type = "A"

  ttl = 300

  records = [

    aws_instance.web.private_ip

  ]

}
```

---

# Default Security Group

```hcl
resource "aws_default_security_group" "default" {

  vpc_id = aws_vpc.main.id

}
```

---

# Default Route Table

```hcl
resource "aws_default_route_table" "default" {

  default_route_table_id = aws_vpc.main.default_route_table_id

}
```

---

# Default Network ACL

```hcl
resource "aws_default_network_acl" "default" {

  default_network_acl_id = aws_vpc.main.default_network_acl_id

}
```

---

# Networking Outputs

```hcl
output "vpc_id" {

  value = aws_vpc.main.id

}

output "public_subnet" {

  value = aws_subnet.public.id

}

output "private_subnet" {

  value = aws_subnet.private.id

}
```

---

# Best Practices

- Use separate public and private subnets.
- Deploy resources across multiple Availability Zones.
- Use NAT Gateway only for private subnet internet access.
- Prefer Interface/Gateway Endpoints over NAT Gateway where applicable.
- Keep Security Groups restrictive.
- Use Network ACLs only when subnet-level filtering is required.
- Enable DNS hostnames and DNS support.
- Tag all networking resources consistently.
- Use Route53 private hosted zones for internal services.
- Avoid using the default VPC in production.

---

# Summary

This section covered Terraform examples for Amazon VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, Network ACLs, VPC Peering, Transit Gateway, ENIs, VPC Endpoints, Route53 integration, and networking outputs. These examples provide production-ready patterns for building secure, scalable AWS networking infrastructure.

---

# AWS IAM

---

# IAM User

```hcl
resource "aws_iam_user" "developer" {

  name = "developer"

  path = "/"

  tags = {

    Team = "DevOps"

  }

}
```

---

# Multiple IAM Users

```hcl
resource "aws_iam_user" "users" {

  for_each = toset([

    "alice",

    "bob",

    "charlie"

  ])

  name = each.key

}
```

---

# IAM Group

```hcl
resource "aws_iam_group" "devops" {

  name = "DevOps"

}
```

---

# Group Membership

```hcl
resource "aws_iam_group_membership" "members" {

  name = "devops-members"

  users = [

    aws_iam_user.developer.name

  ]

  group = aws_iam_group.devops.name

}
```

---

# IAM Policy

```hcl
resource "aws_iam_policy" "s3_readonly" {

  name = "S3ReadOnly"

  policy = jsonencode({

    Version = "2012-10-17"

    Statement = [

      {

        Effect = "Allow"

        Action = [

          "s3:GetObject",

          "s3:ListBucket"

        ]

        Resource = "*"

      }

    ]

  })

}
```

---

# Attach Policy to User

```hcl
resource "aws_iam_user_policy_attachment" "user" {

  user = aws_iam_user.developer.name

  policy_arn = aws_iam_policy.s3_readonly.arn

}
```

---

# Attach Policy to Group

```hcl
resource "aws_iam_group_policy_attachment" "group" {

  group = aws_iam_group.devops.name

  policy_arn = aws_iam_policy.s3_readonly.arn

}
```

---

# IAM Role

```hcl
resource "aws_iam_role" "ec2" {

  name = "EC2Role"

  assume_role_policy = jsonencode({

    Version = "2012-10-17"

    Statement = [

      {

        Effect = "Allow"

        Principal = {

          Service = "ec2.amazonaws.com"

        }

        Action = "sts:AssumeRole"

      }

    ]

  })

}
```

---

# Attach Managed Policy

```hcl
resource "aws_iam_role_policy_attachment" "ssm" {

  role = aws_iam_role.ec2.name

  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"

}
```

---

# Inline Role Policy

```hcl
resource "aws_iam_role_policy" "inline" {

  name = "InlinePolicy"

  role = aws_iam_role.ec2.id

  policy = jsonencode({

    Version = "2012-10-17"

    Statement = [

      {

        Effect = "Allow"

        Action = [

          "s3:*"

        ]

        Resource = "*"

      }

    ]

  })

}
```

---

# Instance Profile

```hcl
resource "aws_iam_instance_profile" "ec2" {

  name = "EC2Profile"

  role = aws_iam_role.ec2.name

}
```

---

# IAM Access Key

```hcl
resource "aws_iam_access_key" "developer" {

  user = aws_iam_user.developer.name

}
```

---

# Password Policy

```hcl
resource "aws_iam_account_password_policy" "policy" {

  minimum_password_length = 14

  require_uppercase_characters = true

  require_lowercase_characters = true

  require_numbers = true

  require_symbols = true

}
```

---

# Customer Managed KMS Key

```hcl
resource "aws_kms_key" "main" {

  description = "Production Key"

  deletion_window_in_days = 30

  enable_key_rotation = true

}
```

---

# KMS Alias

```hcl
resource "aws_kms_alias" "main" {

  name = "alias/production"

  target_key_id = aws_kms_key.main.key_id

}
```

---

# Secrets Manager Secret

```hcl
resource "aws_secretsmanager_secret" "db" {

  name = "database-password"

}
```

---

# Secret Version

```hcl
resource "aws_secretsmanager_secret_version" "db" {

  secret_id = aws_secretsmanager_secret.db.id

  secret_string = jsonencode({

    username = "admin"

    password = "Password@123"

  })

}
```

---

# Systems Manager Parameter

```hcl
resource "aws_ssm_parameter" "db" {

  name = "/prod/db/password"

  type = "SecureString"

  value = "Password@123"

}
```

---

# ACM Certificate

```hcl
resource "aws_acm_certificate" "web" {

  domain_name = "example.com"

  validation_method = "DNS"

}
```

---

# ACM Validation

```hcl
resource "aws_acm_certificate_validation" "web" {

  certificate_arn = aws_acm_certificate.web.arn

}
```

---

# IAM Identity Center Instance

```hcl
data "aws_ssoadmin_instances" "main" {}
```

---

# Organizations

---

# Organization

```hcl
resource "aws_organizations_organization" "main" {

  feature_set = "ALL"

}
```

---

# Organizational Unit

```hcl
resource "aws_organizations_organizational_unit" "dev" {

  name = "Development"

  parent_id = aws_organizations_organization.main.roots[0].id

}
```

---

# AWS Account

```hcl
resource "aws_organizations_account" "dev" {

  name = "Development"

  email = "dev@example.com"

}
```

---

# Service Control Policy

```hcl
resource "aws_organizations_policy" "scp" {

  name = "DenyRoot"

  type = "SERVICE_CONTROL_POLICY"

  content = jsonencode({

    Version = "2012-10-17"

    Statement = []

  })

}
```

---

# Attach SCP

```hcl
resource "aws_organizations_policy_attachment" "attach" {

  policy_id = aws_organizations_policy.scp.id

  target_id = aws_organizations_organizational_unit.dev.id

}
```

---

# GuardDuty Detector

```hcl
resource "aws_guardduty_detector" "main" {

  enable = true

}
```

---

# Security Hub

```hcl
resource "aws_securityhub_account" "main" {}
```

---

# Security Hub Standards

```hcl
resource "aws_securityhub_standards_subscription" "cis" {

  standards_arn = "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0"

}
```

---

# AWS Inspector

```hcl
resource "aws_inspector2_enabler" "main" {

  account_ids = [

    data.aws_caller_identity.current.account_id

  ]

  resource_types = [

    "EC2",

    "ECR"

  ]

}
```

---

# Macie

```hcl
resource "aws_macie2_account" "main" {

  status = "ENABLED"

}
```

---

# Detective

```hcl
resource "aws_detective_graph" "main" {

  auto_enable_members = true

}
```

---

# AWS WAF

```hcl
resource "aws_wafv2_web_acl" "web" {

  name = "production-acl"

  scope = "REGIONAL"

  default_action {

    allow {}

  }

  visibility_config {

    cloudwatch_metrics_enabled = true

    metric_name = "ProductionACL"

    sampled_requests_enabled = true

  }

}
```

---

# AWS Shield Advanced Protection

```hcl
resource "aws_shield_protection" "alb" {

  name = "production-alb"

  resource_arn = aws_lb.app.arn

}
```

---

# AWS CloudTrail

```hcl
resource "aws_cloudtrail" "main" {

  name = "organization-trail"

  s3_bucket_name = aws_s3_bucket.logs.id

}
```

---

# AWS Config Recorder

```hcl
resource "aws_config_configuration_recorder" "main" {

  name = "config"

  role_arn = aws_iam_role.config.arn

}
```

---

# Current Account

```hcl
data "aws_caller_identity" "current" {}
```

---

# Current Region

```hcl
data "aws_region" "current" {}
```

---

# Current Partition

```hcl
data "aws_partition" "current" {}
```

---

# Outputs

```hcl
output "kms_key_arn" {

  value = aws_kms_key.main.arn

}

output "role_arn" {

  value = aws_iam_role.ec2.arn

}
```

---

# Best Practices

- Prefer IAM Roles over IAM Users for AWS workloads.
- Apply the principle of least privilege to all IAM policies.
- Enable automatic KMS key rotation.
- Store secrets in AWS Secrets Manager or Parameter Store.
- Enforce strong account password policies.
- Use Service Control Policies (SCPs) in AWS Organizations.
- Enable GuardDuty, Security Hub, Inspector, Macie, and Detective in every production account.
- Use ACM for certificate lifecycle management.
- Never hardcode secrets in Terraform code; use variables or external secret stores.
- Tag all IAM and security resources for governance.

---

# Summary

This section covered Terraform examples for IAM Users, Groups, Roles, Policies, Instance Profiles, KMS, Secrets Manager, Systems Manager Parameter Store, ACM, IAM Identity Center data sources, AWS Organizations, Service Control Policies, GuardDuty, Security Hub, Inspector, Macie, Detective, WAF, CloudTrail, Config, and identity-related data sources. These examples provide production-ready patterns for building secure AWS environments with Terraform.

---

# Amazon S3

---

# Create S3 Bucket

```hcl
resource "aws_s3_bucket" "main" {

  bucket = "production-app-storage"

  tags = {

    Environment = "Production"

  }

}
```

---

# Enable Versioning

```hcl
resource "aws_s3_bucket_versioning" "main" {

  bucket = aws_s3_bucket.main.id

  versioning_configuration {

    status = "Enabled"

  }

}
```

---

# Enable Server-Side Encryption

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "main" {

  bucket = aws_s3_bucket.main.id

  rule {

    apply_server_side_encryption_by_default {

      sse_algorithm = "AES256"

    }

  }

}
```

---

# SSE-KMS Encryption

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "kms" {

  bucket = aws_s3_bucket.main.id

  rule {

    apply_server_side_encryption_by_default {

      kms_master_key_id = aws_kms_key.main.arn

      sse_algorithm = "aws:kms"

    }

  }

}
```

---

# Block Public Access

```hcl
resource "aws_s3_bucket_public_access_block" "main" {

  bucket = aws_s3_bucket.main.id

  block_public_acls = true

  block_public_policy = true

  ignore_public_acls = true

  restrict_public_buckets = true

}
```

---

# Bucket Lifecycle

```hcl
resource "aws_s3_bucket_lifecycle_configuration" "main" {

  bucket = aws_s3_bucket.main.id

  rule {

    id = "archive"

    status = "Enabled"

    transition {

      days = 30

      storage_class = "GLACIER"

    }

  }

}
```

---

# Bucket Logging

```hcl
resource "aws_s3_bucket_logging" "main" {

  bucket = aws_s3_bucket.main.id

  target_bucket = aws_s3_bucket.logs.id

  target_prefix = "logs/"

}
```

---

# Bucket Policy

```hcl
resource "aws_s3_bucket_policy" "main" {

  bucket = aws_s3_bucket.main.id

  policy = data.aws_iam_policy_document.bucket.json

}
```

---

# Bucket Notification

```hcl
resource "aws_s3_bucket_notification" "main" {

  bucket = aws_s3_bucket.main.id

}
```

---

# Bucket Ownership Controls

```hcl
resource "aws_s3_bucket_ownership_controls" "main" {

  bucket = aws_s3_bucket.main.id

  rule {

    object_ownership = "BucketOwnerPreferred"

  }

}
```

---

# Amazon EFS

---

# File System

```hcl
resource "aws_efs_file_system" "main" {

  encrypted = true

  performance_mode = "generalPurpose"

  throughput_mode = "bursting"

}
```

---

# Mount Target

```hcl
resource "aws_efs_mount_target" "private" {

  file_system_id = aws_efs_file_system.main.id

  subnet_id = aws_subnet.private.id

  security_groups = [

    aws_security_group.efs.id

  ]

}
```

---

# Access Point

```hcl
resource "aws_efs_access_point" "app" {

  file_system_id = aws_efs_file_system.main.id

}
```

---

# Amazon FSx

---

# FSx Windows

```hcl
resource "aws_fsx_windows_file_system" "main" {

  storage_capacity = 32

  subnet_ids = [

    aws_subnet.private.id

  ]

  throughput_capacity = 8

}
```

---

# FSx Lustre

```hcl
resource "aws_fsx_lustre_file_system" "main" {

  storage_capacity = 1200

  subnet_ids = [

    aws_subnet.private.id

  ]

}
```

---

# Amazon RDS

---

# MySQL Instance

```hcl
resource "aws_db_instance" "mysql" {

  identifier = "production-db"

  engine = "mysql"

  instance_class = "db.t3.micro"

  allocated_storage = 20

  username = "admin"

  password = var.db_password

  skip_final_snapshot = true

}
```

---

# PostgreSQL Instance

```hcl
resource "aws_db_instance" "postgres" {

  identifier = "postgres-db"

  engine = "postgres"

  instance_class = "db.t3.small"

  allocated_storage = 50

  username = "postgres"

  password = var.db_password

}
```

---

# Parameter Group

```hcl
resource "aws_db_parameter_group" "mysql" {

  family = "mysql8.0"

  name = "mysql-parameters"

}
```

---

# Option Group

```hcl
resource "aws_db_option_group" "mysql" {

  engine_name = "mysql"

  major_engine_version = "8.0"

}
```

---

# DB Subnet Group

```hcl
resource "aws_db_subnet_group" "main" {

  subnet_ids = [

    aws_subnet.private.id

  ]

}
```

---

# Read Replica

```hcl
resource "aws_db_instance" "replica" {

  replicate_source_db = aws_db_instance.mysql.identifier

  instance_class = "db.t3.micro"

}
```

---

# Aurora Cluster

```hcl
resource "aws_rds_cluster" "aurora" {

  cluster_identifier = "aurora-prod"

  engine = "aurora-mysql"

  master_username = "admin"

  master_password = var.db_password

}
```

---

# Aurora Instance

```hcl
resource "aws_rds_cluster_instance" "writer" {

  cluster_identifier = aws_rds_cluster.aurora.id

  instance_class = "db.r6g.large"

  engine = aws_rds_cluster.aurora.engine

}
```

---

# Amazon DynamoDB

---

# DynamoDB Table

```hcl
resource "aws_dynamodb_table" "users" {

  name = "Users"

  billing_mode = "PAY_PER_REQUEST"

  hash_key = "UserId"

  attribute {

    name = "UserId"

    type = "S"

  }

}
```

---

# Global Secondary Index

```hcl
global_secondary_index {

  name = "EmailIndex"

  hash_key = "Email"

  projection_type = "ALL"

}
```

---

# TTL

```hcl
ttl {

  attribute_name = "Expiry"

  enabled = true

}
```

---

# Point-in-Time Recovery

```hcl
point_in_time_recovery {

  enabled = true

}
```

---

# Amazon ElastiCache

---

# Redis Cluster

```hcl
resource "aws_elasticache_cluster" "redis" {

  cluster_id = "redis-prod"

  engine = "redis"

  node_type = "cache.t3.micro"

  num_cache_nodes = 1

}
```

---

# Redis Replication Group

```hcl
resource "aws_elasticache_replication_group" "redis" {

  replication_group_id = "redis"

  engine = "redis"

  node_type = "cache.t3.small"

  automatic_failover_enabled = true

}
```

---

# Memcached Cluster

```hcl
resource "aws_elasticache_cluster" "memcached" {

  cluster_id = "memcached"

  engine = "memcached"

  node_type = "cache.t3.micro"

  num_cache_nodes = 2

}
```

---

# AWS Backup

---

# Backup Vault

```hcl
resource "aws_backup_vault" "main" {

  name = "ProductionVault"

}
```

---

# Backup Plan

```hcl
resource "aws_backup_plan" "daily" {

  name = "DailyBackup"

  rule {

    rule_name = "Daily"

    target_vault_name = aws_backup_vault.main.name

    schedule = "cron(0 2 * * ? *)"

  }

}
```

---

# Backup Selection

```hcl
resource "aws_backup_selection" "ec2" {

  iam_role_arn = aws_iam_role.backup.arn

  name = "EC2Selection"

  plan_id = aws_backup_plan.daily.id

  resources = [

    aws_instance.web.arn

  ]

}
```

---

# AWS DataSync

---

# DataSync Location (S3)

```hcl
resource "aws_datasync_location_s3" "source" {

  s3_bucket_arn = aws_s3_bucket.main.arn

  subdirectory = "/"

}
```

---

# DataSync Task

```hcl
resource "aws_datasync_task" "migration" {

  source_location_arn = aws_datasync_location_s3.source.arn

  destination_location_arn = aws_datasync_location_s3.destination.arn

}
```

---

# AWS Storage Gateway

---

# File Gateway

```hcl
resource "aws_storagegateway_gateway" "main" {

  gateway_name = "FileGateway"

  gateway_timezone = "GMT"

  gateway_type = "FILE_S3"

}
```

---

# SMB File Share

```hcl
resource "aws_storagegateway_smb_file_share" "share" {

  gateway_arn = aws_storagegateway_gateway.main.arn

  location_arn = aws_s3_bucket.main.arn

}
```

---

# AWS Snow Family

---

# Snowball Job

```hcl
resource "aws_snowball_job" "import" {

  job_type = "IMPORT"

  snowball_type = "EDGE"

}
```

---

# Data Sources

---

# Latest RDS Engine

```hcl
data "aws_rds_engine_version" "mysql" {

  engine = "mysql"

}
```

---

# Current Caller Identity

```hcl
data "aws_caller_identity" "current" {}
```

---

# Outputs

```hcl
output "bucket_name" {

  value = aws_s3_bucket.main.bucket

}

output "database_endpoint" {

  value = aws_db_instance.mysql.endpoint

}

output "efs_id" {

  value = aws_efs_file_system.main.id

}
```

---

# Best Practices

- Enable S3 Versioning and default encryption.
- Block public access for production buckets.
- Store databases in private subnets.
- Enable automated backups and Multi-AZ for production RDS instances.
- Enable Point-in-Time Recovery for DynamoDB tables.
- Encrypt EFS, FSx, RDS, and ElastiCache using KMS.
- Use AWS Backup to centralize backup policies.
- Protect sensitive variables (such as database passwords) using Terraform variables and secret managers.
- Tag all storage and database resources consistently.

---

# Summary

This section covered Terraform examples for Amazon S3, EFS, FSx, RDS, Aurora, DynamoDB, ElastiCache, AWS Backup, DataSync, Storage Gateway, and Snow Family. These examples provide production-ready infrastructure patterns for AWS storage, databases, backup, and migration services.

---

# Amazon Elastic Container Registry (ECR)

---

# Create ECR Repository

```hcl
resource "aws_ecr_repository" "app" {

  name = "production-app"

  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {

    scan_on_push = true

  }

}
```

---

# Repository Policy

```hcl
resource "aws_ecr_repository_policy" "app" {

  repository = aws_ecr_repository.app.name

  policy = data.aws_iam_policy_document.ecr.json

}
```

---

# Lifecycle Policy

```hcl
resource "aws_ecr_lifecycle_policy" "app" {

  repository = aws_ecr_repository.app.name

  policy = jsonencode({

    rules = [

      {

        rulePriority = 1

        description = "Keep last 20 images"

        selection = {

          tagStatus = "any"

          countType = "imageCountMoreThan"

          countNumber = 20

        }

        action = {

          type = "expire"

        }

      }

    ]

  })

}
```

---

# Amazon ECS

---

# ECS Cluster

```hcl
resource "aws_ecs_cluster" "main" {

  name = "production"

}
```

---

# Capacity Providers

```hcl
resource "aws_ecs_cluster_capacity_providers" "main" {

  cluster_name = aws_ecs_cluster.main.name

  capacity_providers = [

    "FARGATE",

    "FARGATE_SPOT"

  ]

}
```

---

# Task Definition

```hcl
resource "aws_ecs_task_definition" "web" {

  family = "web"

  network_mode = "awsvpc"

  requires_compatibilities = [

    "FARGATE"

  ]

  cpu = 512

  memory = 1024

  execution_role_arn = aws_iam_role.ecs_execution.arn

  container_definitions = file("task-definition.json")

}
```

---

# ECS Service

```hcl
resource "aws_ecs_service" "web" {

  name = "web"

  cluster = aws_ecs_cluster.main.id

  task_definition = aws_ecs_task_definition.web.arn

  desired_count = 2

  launch_type = "FARGATE"

}
```

---

# Auto Scaling Target

```hcl
resource "aws_appautoscaling_target" "ecs" {

  service_namespace = "ecs"

  resource_id = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.web.name}"

  scalable_dimension = "ecs:service:DesiredCount"

  min_capacity = 2

  max_capacity = 10

}
```

---

# Auto Scaling Policy

```hcl
resource "aws_appautoscaling_policy" "cpu" {

  name = "cpu-scaling"

  service_namespace = "ecs"

  resource_id = aws_appautoscaling_target.ecs.resource_id

  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension

  policy_type = "TargetTrackingScaling"

}
```

---

# Amazon EKS

---

# EKS Cluster

```hcl
resource "aws_eks_cluster" "main" {

  name = "production"

  role_arn = aws_iam_role.eks.arn

  version = "1.31"

  vpc_config {

    subnet_ids = [

      aws_subnet.private.id

    ]

  }

}
```

---

# Managed Node Group

```hcl
resource "aws_eks_node_group" "workers" {

  cluster_name = aws_eks_cluster.main.name

  node_group_name = "workers"

  node_role_arn = aws_iam_role.worker.arn

  subnet_ids = [

    aws_subnet.private.id

  ]

  scaling_config {

    desired_size = 2

    min_size = 2

    max_size = 6

  }

}
```

---

# EKS Add-on

```hcl
resource "aws_eks_addon" "coredns" {

  cluster_name = aws_eks_cluster.main.name

  addon_name = "coredns"

}
```

---

# OIDC Provider

```hcl
resource "aws_iam_openid_connect_provider" "eks" {

  url = aws_eks_cluster.main.identity[0].oidc[0].issuer

  client_id_list = [

    "sts.amazonaws.com"

  ]

}
```

---

# Fargate Profile

```hcl
resource "aws_eks_fargate_profile" "default" {

  cluster_name = aws_eks_cluster.main.name

  fargate_profile_name = "default"

  pod_execution_role_arn = aws_iam_role.fargate.arn

  subnet_ids = [

    aws_subnet.private.id

  ]

}
```

---

# Kubernetes Provider

```hcl
provider "kubernetes" {

  host = aws_eks_cluster.main.endpoint

}
```

---

# Namespace

```hcl
resource "kubernetes_namespace" "app" {

  metadata {

    name = "production"

  }

}
```

---

# ConfigMap

```hcl
resource "kubernetes_config_map" "app" {

  metadata {

    name = "application-config"

    namespace = "production"

  }

}
```

---

# Secret

```hcl
resource "kubernetes_secret" "db" {

  metadata {

    name = "db-secret"

  }

}
```

---

# Deployment

```hcl
resource "kubernetes_deployment" "web" {

  metadata {

    name = "web"

  }

}
```

---

# Service

```hcl
resource "kubernetes_service" "web" {

  metadata {

    name = "web"

  }

}
```

---

# Ingress

```hcl
resource "kubernetes_ingress_v1" "web" {

  metadata {

    name = "web"

  }

}
```

---

# AWS Lambda

---

# Lambda Function

```hcl
resource "aws_lambda_function" "processor" {

  function_name = "processor"

  runtime = "python3.12"

  handler = "lambda_function.lambda_handler"

  filename = "lambda.zip"

  role = aws_iam_role.lambda.arn

}
```

---

# Lambda Permission

```hcl
resource "aws_lambda_permission" "apigw" {

  statement_id = "AllowAPIGateway"

  action = "lambda:InvokeFunction"

  function_name = aws_lambda_function.processor.function_name

  principal = "apigateway.amazonaws.com"

}
```

---

# Lambda Function URL

```hcl
resource "aws_lambda_function_url" "main" {

  function_name = aws_lambda_function.processor.function_name

  authorization_type = "NONE"

}
```

---

# API Gateway REST API

```hcl
resource "aws_api_gateway_rest_api" "main" {

  name = "OrdersAPI"

}
```

---

# API Gateway HTTP API

```hcl
resource "aws_apigatewayv2_api" "http" {

  name = "OrdersHTTP"

  protocol_type = "HTTP"

}
```

---

# App Runner

```hcl
resource "aws_apprunner_service" "app" {

  service_name = "production-app"

}
```

---

# EventBridge Bus

```hcl
resource "aws_cloudwatch_event_bus" "main" {

  name = "application-events"

}
```

---

# EventBridge Rule

```hcl
resource "aws_cloudwatch_event_rule" "schedule" {

  name = "daily-job"

  schedule_expression = "rate(1 day)"

}
```

---

# EventBridge Target

```hcl
resource "aws_cloudwatch_event_target" "lambda" {

  rule = aws_cloudwatch_event_rule.schedule.name

  arn = aws_lambda_function.processor.arn

}
```

---

# SNS Topic

```hcl
resource "aws_sns_topic" "alerts" {

  name = "alerts"

}
```

---

# Email Subscription

```hcl
resource "aws_sns_topic_subscription" "email" {

  topic_arn = aws_sns_topic.alerts.arn

  protocol = "email"

  endpoint = "admin@example.com"

}
```

---

# SQS Queue

```hcl
resource "aws_sqs_queue" "orders" {

  name = "orders"

  visibility_timeout_seconds = 30

}
```

---

# Dead Letter Queue

```hcl
resource "aws_sqs_queue" "dlq" {

  name = "orders-dlq"

}
```

---

# Queue Redrive Policy

```hcl
resource "aws_sqs_queue_redrive_policy" "orders" {

  queue_url = aws_sqs_queue.orders.id

  redrive_policy = jsonencode({

    deadLetterTargetArn = aws_sqs_queue.dlq.arn

    maxReceiveCount = 5

  })

}
```

---

# Step Functions

```hcl
resource "aws_sfn_state_machine" "workflow" {

  name = "OrderWorkflow"

  role_arn = aws_iam_role.stepfunctions.arn

  definition = file("workflow.json")

}
```

---

# Cloud Map Namespace

```hcl
resource "aws_service_discovery_private_dns_namespace" "main" {

  name = "internal.local"

  vpc = aws_vpc.main.id

}
```

---

# Cloud Map Service

```hcl
resource "aws_service_discovery_service" "web" {

  name = "web"

  dns_config {

    namespace_id = aws_service_discovery_private_dns_namespace.main.id

  }

}
```

---

# Outputs

```hcl
output "eks_cluster" {

  value = aws_eks_cluster.main.name

}

output "ecs_cluster" {

  value = aws_ecs_cluster.main.name

}

output "lambda_name" {

  value = aws_lambda_function.processor.function_name

}
```

---

# Best Practices

- Enable image scanning and immutable tags in Amazon ECR.
- Use Fargate for serverless container workloads when appropriate.
- Enable EKS managed add-ons and OIDC for IAM Roles for Service Accounts (IRSA).
- Store Kubernetes manifests separately from infrastructure code when using GitOps.
- Package Lambda functions with minimal dependencies.
- Configure DLQs for SQS and Lambda error handling.
- Use EventBridge for event-driven architectures instead of polling.
- Protect API Gateway endpoints with authentication and authorization.
- Tag all container and serverless resources consistently.

---

# Summary

This section covered Terraform examples for Amazon ECR, ECS, EKS, Kubernetes resources, Lambda, API Gateway, App Runner, EventBridge, SNS, SQS, Step Functions, and AWS Cloud Map. These examples provide production-ready infrastructure patterns for containers, Kubernetes, serverless applications, messaging, and event-driven architectures.

---

# Amazon CloudWatch

---

# CloudWatch Dashboard

```hcl
resource "aws_cloudwatch_dashboard" "main" {

  dashboard_name = "ProductionDashboard"

  dashboard_body = file("dashboard.json")

}
```

---

# Metric Alarm

```hcl
resource "aws_cloudwatch_metric_alarm" "cpu" {

  alarm_name = "HighCPU"

  namespace = "AWS/EC2"

  metric_name = "CPUUtilization"

  statistic = "Average"

  threshold = 80

  comparison_operator = "GreaterThanThreshold"

  evaluation_periods = 2

  period = 300

}
```

---

# Composite Alarm

```hcl
resource "aws_cloudwatch_composite_alarm" "critical" {

  alarm_name = "CriticalAlarm"

  alarm_rule = "ALARM(HighCPU)"

}
```

---

# Log Group

```hcl
resource "aws_cloudwatch_log_group" "application" {

  name = "/application/logs"

  retention_in_days = 30

}
```

---

# Log Stream

```hcl
resource "aws_cloudwatch_log_stream" "main" {

  name = "production"

  log_group_name = aws_cloudwatch_log_group.application.name

}
```

---

# Log Resource Policy

```hcl
resource "aws_cloudwatch_log_resource_policy" "logs" {

  policy_name = "CloudWatchLogs"

  policy_document = file("logs-policy.json")

}
```

---

# EventBridge Bus

```hcl
resource "aws_cloudwatch_event_bus" "operations" {

  name = "operations"

}
```

---

# EventBridge Rule

```hcl
resource "aws_cloudwatch_event_rule" "ec2" {

  name = "EC2StateChange"

  event_pattern = file("ec2-events.json")

}
```

---

# EventBridge Target

```hcl
resource "aws_cloudwatch_event_target" "lambda" {

  rule = aws_cloudwatch_event_rule.ec2.name

  arn = aws_lambda_function.processor.arn

}
```

---

# CloudTrail

---

# Organization Trail

```hcl
resource "aws_cloudtrail" "organization" {

  name = "OrganizationTrail"

  s3_bucket_name = aws_s3_bucket.logs.id

  is_organization_trail = true

}
```

---

# CloudTrail Event Selector

```hcl
event_selector {

  read_write_type = "All"

  include_management_events = true

}
```

---

# AWS Config

---

# Configuration Recorder

```hcl
resource "aws_config_configuration_recorder" "main" {

  name = "config"

  role_arn = aws_iam_role.config.arn

}
```

---

# Delivery Channel

```hcl
resource "aws_config_delivery_channel" "main" {

  s3_bucket_name = aws_s3_bucket.logs.bucket

}
```

---

# Config Rule

```hcl
resource "aws_config_config_rule" "encrypted_volumes" {

  name = "encrypted-volumes"

  source {

    owner = "AWS"

    source_identifier = "ENCRYPTED_VOLUMES"

  }

}
```

---

# Conformance Pack

```hcl
resource "aws_config_conformance_pack" "security" {

  name = "OperationalBestPractices"

  template_body = file("conformance-pack.yaml")

}
```

---

# AWS CodeCommit

---

# Repository

```hcl
resource "aws_codecommit_repository" "repo" {

  repository_name = "devops-repository"

  description = "Infrastructure Repository"

}
```

---

# Approval Rule Template

```hcl
resource "aws_codecommit_approval_rule_template" "main" {

  approval_rule_template_name = "MandatoryReview"

  approval_rule_template_content = file("approval.json")

}
```

---

# AWS CodeBuild

---

# Build Project

```hcl
resource "aws_codebuild_project" "build" {

  name = "ApplicationBuild"

  service_role = aws_iam_role.codebuild.arn

}
```

---

# Build Webhook

```hcl
resource "aws_codebuild_webhook" "main" {

  project_name = aws_codebuild_project.build.name

}
```

---

# Source Credential

```hcl
resource "aws_codebuild_source_credential" "github" {

  auth_type = "PERSONAL_ACCESS_TOKEN"

  server_type = "GITHUB"

  token = var.github_token

}
```

---

# AWS CodeDeploy

---

# Application

```hcl
resource "aws_codedeploy_app" "main" {

  name = "WebApplication"

}
```

---

# Deployment Group

```hcl
resource "aws_codedeploy_deployment_group" "production" {

  app_name = aws_codedeploy_app.main.name

  deployment_group_name = "Production"

  service_role_arn = aws_iam_role.codedeploy.arn

}
```

---

# AWS CodePipeline

---

# Pipeline

```hcl
resource "aws_codepipeline" "pipeline" {

  name = "ProductionPipeline"

  role_arn = aws_iam_role.pipeline.arn

}
```

---

# AWS CodeArtifact

---

# Domain

```hcl
resource "aws_codeartifact_domain" "main" {

  domain = "company"

}
```

---

# Repository

```hcl
resource "aws_codeartifact_repository" "maven" {

  repository = "maven"

  domain = aws_codeartifact_domain.main.domain

}
```

---

# AWS Systems Manager

---

# SSM Parameter

```hcl
resource "aws_ssm_parameter" "api" {

  name = "/prod/api/url"

  type = "String"

  value = "https://api.example.com"

}
```

---

# Patch Baseline

```hcl
resource "aws_ssm_patch_baseline" "linux" {

  name = "LinuxBaseline"

  operating_system = "AMAZON_LINUX_2"

}
```

---

# Maintenance Window

```hcl
resource "aws_ssm_maintenance_window" "weekly" {

  name = "WeeklyMaintenance"

  schedule = "cron(0 2 ? * SUN *)"

  duration = 3

  cutoff = 1

}
```

---

# Maintenance Window Target

```hcl
resource "aws_ssm_maintenance_window_target" "instances" {

  window_id = aws_ssm_maintenance_window.weekly.id

  resource_type = "INSTANCE"

}
```

---

# Maintenance Window Task

```hcl
resource "aws_ssm_maintenance_window_task" "patch" {

  window_id = aws_ssm_maintenance_window.weekly.id

  task_type = "RUN_COMMAND"

  task_arn = "AWS-RunPatchBaseline"

}
```

---

# Association

```hcl
resource "aws_ssm_association" "inventory" {

  name = "AWS-GatherSoftwareInventory"

}
```

---

# Document

```hcl
resource "aws_ssm_document" "restart" {

  name = "RestartService"

  document_type = "Command"

  content = file("restart.json")

}
```

---

# OpsCenter

---

# OpsItem

```hcl
resource "aws_ssm_opsitem" "incident" {

  title = "High CPU Usage"

  source = "CloudWatch"

}
```

---

# Explorer

```hcl
resource "aws_ssm_resource_data_sync" "inventory" {

  name = "InventorySync"

  s3_destination {

    bucket_name = aws_s3_bucket.logs.bucket

  }

}
```

---

# CloudFormation Stack

```hcl
resource "aws_cloudformation_stack" "network" {

  name = "LegacyNetwork"

  template_body = file("network.yaml")

}
```

---

# CloudFormation StackSet

```hcl
resource "aws_cloudformation_stack_set" "organization" {

  name = "SecurityBaseline"

  permission_model = "SERVICE_MANAGED"

  template_body = file("baseline.yaml")

}
```

---

# CloudFormation StackSet Instance

```hcl
resource "aws_cloudformation_stack_set_instance" "prod" {

  stack_set_name = aws_cloudformation_stack_set.organization.name

  deployment_targets {

    organizational_unit_ids = [

      "ou-xxxxxxxx"

    ]

  }

  region = "ap-south-1"

}
```

---

# Data Sources

---

# Current Region

```hcl
data "aws_region" "current" {}
```

---

# Current Caller Identity

```hcl
data "aws_caller_identity" "current" {}
```

---

# Outputs

```hcl
output "pipeline_name" {

  value = aws_codepipeline.pipeline.name

}

output "codebuild_project" {

  value = aws_codebuild_project.build.name

}

output "log_group" {

  value = aws_cloudwatch_log_group.application.name

}
```

---

# Best Practices

- Create CloudWatch dashboards for critical workloads.
- Configure CloudWatch alarms with SNS notifications.
- Enable CloudTrail organization-wide with log file validation.
- Record all supported resources using AWS Config.
- Store configuration values in Systems Manager Parameter Store.
- Use CodePipeline for automated deployments.
- Protect CodeBuild credentials using Secrets Manager.
- Configure regular patching with Systems Manager Maintenance Windows.
- Use CloudFormation StackSets for multi-account deployments.
- Tag monitoring and CI/CD resources consistently.

---

# Summary

This section covered Terraform examples for Amazon CloudWatch, CloudWatch Logs, EventBridge, CloudTrail, AWS Config, CodeCommit, CodeBuild, CodeDeploy, CodePipeline, CodeArtifact, Systems Manager (SSM), OpsCenter, CloudFormation Stacks, and StackSets. These examples provide production-ready patterns for monitoring, CI/CD automation, configuration management, and operational governance.

---

# Amazon Route 53

---

# Public Hosted Zone

```hcl
resource "aws_route53_zone" "public" {

  name = "example.com"

}
```

---

# Private Hosted Zone

```hcl
resource "aws_route53_zone" "private" {

  name = "internal.local"

  vpc {

    vpc_id = aws_vpc.main.id

  }

}
```

---

# A Record

```hcl
resource "aws_route53_record" "app" {

  zone_id = aws_route53_zone.public.zone_id

  name = "app"

  type = "A"

  ttl = 300

  records = [

    aws_instance.web.public_ip

  ]

}
```

---

# Alias Record

```hcl
resource "aws_route53_record" "alb" {

  zone_id = aws_route53_zone.public.zone_id

  name = "www"

  type = "A"

  alias {

    name = aws_lb.app.dns_name

    zone_id = aws_lb.app.zone_id

    evaluate_target_health = true

  }

}
```

---

# CNAME Record

```hcl
resource "aws_route53_record" "api" {

  zone_id = aws_route53_zone.public.zone_id

  name = "api"

  type = "CNAME"

  ttl = 300

  records = [

    aws_lb.app.dns_name

  ]

}
```

---

# MX Record

```hcl
resource "aws_route53_record" "mail" {

  zone_id = aws_route53_zone.public.zone_id

  name = "example.com"

  type = "MX"

  ttl = 300

  records = [

    "10 mail.example.com"

  ]

}
```

---

# TXT Record

```hcl
resource "aws_route53_record" "spf" {

  zone_id = aws_route53_zone.public.zone_id

  name = "example.com"

  type = "TXT"

  ttl = 300

  records = [

    "\"v=spf1 include:_spf.google.com ~all\""

  ]

}
```

---

# Health Check

```hcl
resource "aws_route53_health_check" "app" {

  fqdn = "app.example.com"

  port = 443

  type = "HTTPS"

}
```

---

# Failover Record

```hcl
resource "aws_route53_record" "primary" {

  zone_id = aws_route53_zone.public.zone_id

  name = "app"

  type = "A"

  set_identifier = "primary"

  failover_routing_policy {

    type = "PRIMARY"

  }

}
```

---

# Amazon CloudFront

---

# Distribution

```hcl
resource "aws_cloudfront_distribution" "cdn" {

  enabled = true

  default_root_object = "index.html"

}
```

---

# Origin Access Control

```hcl
resource "aws_cloudfront_origin_access_control" "oac" {

  name = "s3-origin"

  origin_access_control_origin_type = "s3"

  signing_behavior = "always"

  signing_protocol = "sigv4"

}
```

---

# Cache Policy

```hcl
resource "aws_cloudfront_cache_policy" "default" {

  name = "ApplicationCache"

}
```

---

# Response Headers Policy

```hcl
resource "aws_cloudfront_response_headers_policy" "security" {

  name = "SecurityHeaders"

}
```

---

# Origin Request Policy

```hcl
resource "aws_cloudfront_origin_request_policy" "main" {

  name = "OriginPolicy"

}
```

---

# AWS WAF

---

# Web ACL

```hcl
resource "aws_wafv2_web_acl" "main" {

  name = "ProductionACL"

  scope = "REGIONAL"

  default_action {

    allow {}

  }

  visibility_config {

    cloudwatch_metrics_enabled = true

    metric_name = "ProductionACL"

    sampled_requests_enabled = true

  }

}
```

---

# Managed Rule Group

```hcl
rule {

  name = "AWSManagedRulesCommonRuleSet"

  priority = 1

}
```

---

# Associate WAF with ALB

```hcl
resource "aws_wafv2_web_acl_association" "alb" {

  resource_arn = aws_lb.app.arn

  web_acl_arn = aws_wafv2_web_acl.main.arn

}
```

---

# AWS Shield Advanced

---

# Protection

```hcl
resource "aws_shield_protection" "alb" {

  name = "ALBProtection"

  resource_arn = aws_lb.app.arn

}
```

---

# Protection Group

```hcl
resource "aws_shield_protection_group" "main" {

  protection_group_id = "production"

  aggregation = "SUM"

}
```

---

# AWS Global Accelerator

---

# Accelerator

```hcl
resource "aws_globalaccelerator_accelerator" "main" {

  name = "production"

  enabled = true

}
```

---

# Listener

```hcl
resource "aws_globalaccelerator_listener" "https" {

  accelerator_arn = aws_globalaccelerator_accelerator.main.id

  protocol = "TCP"

}
```

---

# Endpoint Group

```hcl
resource "aws_globalaccelerator_endpoint_group" "alb" {

  listener_arn = aws_globalaccelerator_listener.https.id

  endpoint_group_region = "ap-south-1"

}
```

---

# AWS Direct Connect

---

# Connection

```hcl
resource "aws_dx_connection" "main" {

  name = "PrimaryDX"

  bandwidth = "1Gbps"

  location = "EqDC2"

}
```

---

# Private Virtual Interface

```hcl
resource "aws_dx_private_virtual_interface" "private" {

  connection_id = aws_dx_connection.main.id

  vlan = 101

}
```

---

# Gateway

```hcl
resource "aws_dx_gateway" "main" {

  name = "ProductionGateway"

}
```

---

# Site-to-Site VPN

---

# Customer Gateway

```hcl
resource "aws_customer_gateway" "main" {

  bgp_asn = 65000

  ip_address = "203.0.113.10"

  type = "ipsec.1"

}
```

---

# VPN Gateway

```hcl
resource "aws_vpn_gateway" "main" {

  vpc_id = aws_vpc.main.id

}
```

---

# VPN Connection

```hcl
resource "aws_vpn_connection" "main" {

  vpn_gateway_id = aws_vpn_gateway.main.id

  customer_gateway_id = aws_customer_gateway.main.id

  type = "ipsec.1"

}
```

---

# VPN Gateway Attachment

```hcl
resource "aws_vpn_gateway_attachment" "main" {

  vpn_gateway_id = aws_vpn_gateway.main.id

  vpc_id = aws_vpc.main.id

}
```

---

# AWS Client VPN

---

# Client VPN Endpoint

```hcl
resource "aws_ec2_client_vpn_endpoint" "main" {

  description = "Remote Access"

  server_certificate_arn = aws_acm_certificate.vpn.arn

  client_cidr_block = "172.16.0.0/22"

}
```

---

# Client VPN Network Association

```hcl
resource "aws_ec2_client_vpn_network_association" "private" {

  client_vpn_endpoint_id = aws_ec2_client_vpn_endpoint.main.id

  subnet_id = aws_subnet.private.id

}
```

---

# Authorization Rule

```hcl
resource "aws_ec2_client_vpn_authorization_rule" "main" {

  client_vpn_endpoint_id = aws_ec2_client_vpn_endpoint.main.id

  target_network_cidr = aws_vpc.main.cidr_block

  authorize_all_groups = true

}
```

---

# Transit Gateway Route Table

```hcl
resource "aws_ec2_transit_gateway_route_table" "main" {

  transit_gateway_id = aws_ec2_transit_gateway.main.id

}
```

---

# Transit Gateway Route

```hcl
resource "aws_ec2_transit_gateway_route" "private" {

  destination_cidr_block = "10.1.0.0/16"

  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.main.id

}
```

---

# AWS Network Firewall

---

# Firewall Policy

```hcl
resource "aws_networkfirewall_firewall_policy" "main" {

  name = "ProductionPolicy"

}
```

---

# Firewall

```hcl
resource "aws_networkfirewall_firewall" "main" {

  name = "ProductionFirewall"

  vpc_id = aws_vpc.main.id

}
```

---

# Rule Group

```hcl
resource "aws_networkfirewall_rule_group" "stateful" {

  capacity = 100

  name = "StatefulRules"

  type = "STATEFUL"

}
```

---

# AWS Private CA

---

# Certificate Authority

```hcl
resource "aws_acmpca_certificate_authority" "main" {

  type = "ROOT"

}
```

---

# CA Certificate

```hcl
resource "aws_acmpca_certificate" "root" {

  certificate_authority_arn = aws_acmpca_certificate_authority.main.arn

}
```

---

# RAM Resource Share

```hcl
resource "aws_ram_resource_share" "network" {

  name = "SharedNetworking"

  allow_external_principals = false

}
```

---

# Share Transit Gateway

```hcl
resource "aws_ram_resource_association" "tgw" {

  resource_arn = aws_ec2_transit_gateway.main.arn

  resource_share_arn = aws_ram_resource_share.network.arn

}
```

---

# Outputs

```hcl
output "cloudfront_domain" {

  value = aws_cloudfront_distribution.cdn.domain_name

}

output "hosted_zone_id" {

  value = aws_route53_zone.public.zone_id

}

output "global_accelerator_dns" {

  value = aws_globalaccelerator_accelerator.main.dns_name

}
```

---

# Best Practices

- Use Route53 Alias records for AWS resources.
- Enable Route53 health checks for failover.
- Protect CloudFront and ALBs using AWS WAF.
- Use AWS Shield Advanced for internet-facing production workloads.
- Enable Origin Access Control (OAC) for CloudFront S3 origins.
- Use Global Accelerator for globally distributed applications.
- Prefer Direct Connect for consistent hybrid connectivity.
- Use Site-to-Site VPN as a backup to Direct Connect.
- Use AWS Network Firewall for centralized traffic inspection.
- Protect internal PKI with AWS Private CA.

---

# Summary

This section covered Terraform examples for Amazon Route53, CloudFront, AWS WAF, Shield Advanced, Global Accelerator, Direct Connect, Site-to-Site VPN, Client VPN, Transit Gateway routing, AWS Network Firewall, Private CA, and AWS RAM resource sharing. These examples provide production-ready patterns for enterprise networking, hybrid connectivity, CDN, and edge security.

---

