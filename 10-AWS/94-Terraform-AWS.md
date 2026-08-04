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

