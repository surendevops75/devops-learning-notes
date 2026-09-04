# 34-Complete-Terraform-Repository

# # 1 Purpose

This chapter defines a realistic production Terraform repository for the RoboShop DevOps/DevSecOps platform.

The Terraform repository is responsible for AWS infrastructure and platform prerequisites.

It is intentionally separate from the GitOps repository.

```text
Terraform
    |
    +--> AWS VPC
    +--> IAM
    +--> EKS
    +--> ECR
    +--> KMS
    +--> CloudWatch
    +--> ALB prerequisites
    +--> Security controls
    |
    v
AWS Infrastructure

GitOps
    |
    +--> Kubernetes desired state
    |
    v
EKS workloads
```

The core separation is:

```text
Terraform = infrastructure
GitOps    = Kubernetes/application desired state
```

---

 # 2 Why Terraform

Terraform provides:

- Infrastructure as Code
- repeatability
- version control
- reviewable changes
- dependency management
- reusable modules
- drift visibility
- environment consistency
- automated provisioning
- disaster-recovery capability

Instead of manually creating:

```text
VPC
subnets
route tables
IAM roles
EKS
ECR
security resources
```

Terraform defines them declaratively.

---

 # 3 Production Architecture

```text
                    Git
                     |
                     v
             Terraform Repository
                     |
                     v
              CI Terraform Pipeline
                     |
          +----------+----------+
          |                     |
        plan                  policy
          |                     |
          +----------+----------+
                     |
                 approval
                     |
                     v
               terraform apply
                     |
                     v
                   AWS
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
      VPC           EKS           ECR
       |             |
       |             v
       |        Kubernetes
       |             |
       |             v
       |          Argo CD
       |             |
       |             v
       |        GitOps workloads
       |
       +--> ALB
       +--> IAM
       +--> KMS
```

---

 # 4 Repository Responsibilities

Terraform should manage infrastructure such as:

```text
AWS Organizations/accounts where applicable
VPC
subnets
route tables
NAT gateways
VPC endpoints
security groups
IAM roles
EKS cluster
EKS node groups
ECR repositories
KMS keys
CloudWatch configuration
S3 backend
DynamoDB locking where applicable
AWS Load Balancer prerequisites
IRSA / Pod Identity prerequisites
```

Terraform should not normally manage every application deployment.

For example:

```text
catalogue Deployment → GitOps
frontend Service → GitOps
PrometheusRule → GitOps
Argo CD Application → GitOps
```

unless a specific bootstrap dependency requires Terraform to install or configure it.

---

 # 5 Recommended Repository

```text
roboshop-infrastructure/
├── README.md
├── CODEOWNERS
├── .gitignore
├── .terraform.lock.hcl
│
├── bootstrap/
│   ├── backend/
│   └── state/
│
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── ecr/
│   ├── iam/
│   ├── kms/
│   ├── security-groups/
│   ├── vpc-endpoints/
│   └── cloudwatch/
│
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── outputs.tf
│   │   ├── providers.tf
│   │   └── backend.tf
│   │
│   ├── qa/
│   │   └── ...
│   │
│   └── prod/
│       └── ...
│
├── policies/
│   ├── terraform/
│   └── security/
│
└── scripts/
    ├── fmt-check.sh
    ├── validate.sh
    ├── plan.sh
    └── security-scan.sh
```

---

 # 6 Environment Separation

Production should not share state with development.

Use separate state:

```text
dev state
qa state
prod state
```

A practical model:

```text
s3://terraform-state/org/roboshop/dev/terraform.tfstate
s3://terraform-state/org/roboshop/qa/terraform.tfstate
s3://terraform-state/org/roboshop/prod/terraform.tfstate
```

The exact bucket naming convention is organization-specific.

---

 # 7 Remote State

Never depend on a developer laptop for production state.

Use a remote backend.

A common AWS approach is:

```text
S3
+
state locking mechanism supported by the chosen Terraform version/backend
```

State must be protected because it may contain sensitive infrastructure information.

---

 # 8 Backend Example

```hcl
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "roboshop/prod/terraform.tfstate"
    region = "ap-south-1"

    encrypt = true
  }
}
```

Use the organization's approved backend locking strategy.

Do not hard-code sensitive credentials.

---

 # 9 Backend Security

The state bucket should have:

```text
encryption
versioning
restricted IAM
public access blocked
logging where required
retention controls
backup/recovery
```

The Terraform state itself should be treated as sensitive.

---

 # 10 Provider

Example:

```hcl
terraform {
  required_version = ">= 1.6.0, < 2.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = local.common_tags
  }
}
```

In an actual project, pin to an organization-approved provider version.

---

 # 11 Provider Authentication

Do not put:

```hcl
access_key = "..."
secret_key = "..."
```

in Terraform code.

Use:

```text
AWS IAM role
OIDC federation
AWS profile for local development
CI workload identity
```

Production CI should use short-lived credentials wherever possible.

---

 # 12 Provider Aliases

For multi-region architectures:

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  alias  = "dr"
  region = "ap-southeast-1"
}
```

Modules can then receive the appropriate provider.

---

 # 13 Naming Convention

Use consistent names:

```text
roboshop-prod-vpc
roboshop-prod-eks
roboshop-prod-ecr
roboshop-prod-kms
```

Tags:

```text
Project
Environment
Owner
CostCenter
ManagedBy
```

---

 # 14 Common Tags

```hcl
locals {
  common_tags = {
    Project     = "roboshop"
    Environment = var.environment
    ManagedBy   = "terraform"
    Owner       = "platform"
    CostCenter  = "engineering"
  }
}
```

Tags improve:

- cost allocation
- inventory
- operations
- ownership
- governance

---

 # 15 VPC Architecture

Production VPC:

```text
                    Internet
                       |
                Internet Gateway
                       |
          +------------+------------+
          |                         |
      Public AZ-a                Public AZ-b
          |                         |
       ALB/NAT                   ALB/NAT
          |                         |
     Private AZ-a              Private AZ-b
          |                         |
        EKS nodes               EKS nodes
```

Use multiple Availability Zones.

---

 # 16 Subnet Strategy

Example:

```text
Public:
10.10.0.0/20
10.10.16.0/20
10.10.32.0/20

Private:
10.10.64.0/20
10.10.80.0/20
10.10.96.0/20

Database:
10.10.128.0/20
10.10.144.0/20
10.10.160.0/20
```

CIDRs are examples and must be designed around the organization's address plan.

---

 # 17 VPC Module Interface

```hcl
variable "name" {
  type = string
}

variable "cidr" {
  type = string
}

variable "availability_zones" {
  type = list(string)
}

variable "private_subnets" {
  type = list(string)
}

variable "public_subnets" {
  type = list(string)
}

variable "enable_nat_gateway" {
  type    = bool
  default = true
}
```

---

 # 18 VPC Module Example

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = "${var.project}-${var.environment}"

  cidr = var.vpc_cidr

  availability_zones = var.availability_zones

  private_subnets = var.private_subnets
  public_subnets  = var.public_subnets

  enable_nat_gateway = true
}
```

A production implementation may use an approved upstream VPC module rather than maintaining all networking resources internally.

---

 # 19 NAT Gateway Strategy

Options:

```text
one NAT gateway
one NAT per AZ
centralized egress
VPC endpoints
```

For production availability, NAT per AZ reduces dependency on a single AZ.

Trade-off:

```text
higher cost
+
better AZ resilience
```

---

 # 20 VPC Endpoints

Use VPC endpoints where appropriate for AWS services such as:

```text
S3
ECR API
ECR DKR
CloudWatch
STS
Secrets Manager
SSM
```

This can reduce NAT dependency and improve network security.

Exact endpoint requirements depend on workloads.

---

 # 21 EKS Architecture

```text
AWS VPC
 |
 +--> EKS control plane
 |
 +--> managed node group AZ-a
 |
 +--> managed node group AZ-b
 |
 +--> managed node group AZ-c
```

Use managed node groups or another approved compute strategy.

---

 # 22 EKS Terraform Example

```hcl
module "eks" {
  source = "../../modules/eks"

  cluster_name    = "${var.project}-${var.environment}"
  cluster_version = var.eks_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids

  endpoint_public_access  = false
  endpoint_private_access = true

  enabled_log_types = [
    "api",
    "audit",
    "authenticator",
    "controllerManager",
    "scheduler"
  ]
}
```

A real module should expose only required options and use approved defaults.

---

 # 23 EKS API Endpoint

Production should evaluate:

```text
private endpoint
public endpoint restricted by CIDR
or controlled combination
```

A private endpoint reduces internet exposure.

If public access is required, restrict allowed CIDRs.

---

 # 24 EKS Control Plane Logging

Enable relevant control plane logs:

```text
api
audit
authenticator
controllerManager
scheduler
```

Centralized logs improve:

- troubleshooting
- security investigations
- auditability

AWS charges apply.

---

 # 25 EKS Encryption

Use KMS for Kubernetes secrets encryption where supported by the selected EKS design.

Example:

```hcl
resource "aws_kms_key" "eks" {
  description             = "KMS key for RoboShop EKS secrets"
  deletion_window_in_days = 30

  enable_key_rotation = true
}
```

Key policies must be carefully restricted.

---

 # 26 EKS Access Management

Avoid broad:

```text
cluster-admin
```

access.

Use:

```text
EKS access entries
Kubernetes RBAC
IAM roles
least privilege
```

The exact EKS access mechanism should match the cluster version and organizational standard.

---

 # 27 EKS Node Groups

Example:

```hcl
module "system_nodes" {
  source = "../../modules/eks"

  node_group_name = "system"

  instance_types = [
    "m6i.large"
  ]

  desired_size = 3
  min_size     = 3
  max_size     = 6

  subnet_ids = module.vpc.private_subnet_ids
}
```

For a real implementation, the EKS module would normally separate cluster and node-group resources/interfaces.

---

 # 28 Workload Node Groups

Consider separate groups:

```text
system
general
memory-optimized
compute-optimized
```

Example:

```text
system nodes
  |
  +--> CoreDNS
  +--> controllers
  +--> monitoring agents

application nodes
  |
  +--> RoboShop services
```

Use taints/tolerations only when there is a real scheduling requirement.

---

 # 29 EKS Autoscaling

Infrastructure autoscaling may include:

```text
managed node group scaling
Karpenter where approved
```

Application scaling is handled separately:

```text
HPA
```

Do not confuse:

```text
HPA = pod scaling
node autoscaling = node capacity scaling
```

---

 # 30 ECR

Create separate repositories:

```text
roboshop/frontend
roboshop/catalogue
roboshop/cart
roboshop/user
roboshop/payment
roboshop/shipping
roboshop/checkout
```

---

 # 31 ECR Terraform

```hcl
resource "aws_ecr_repository" "catalogue" {
  name                 = "roboshop/catalogue"
  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }

  encryption_configuration {
    encryption_type = "AES256"
  }

  tags = local.common_tags
}
```

KMS-backed ECR encryption may be selected if required by organizational policy.

---

 # 32 ECR Lifecycle Policy

```hcl
resource "aws_ecr_lifecycle_policy" "catalogue" {
  repository = aws_ecr_repository.catalogue.name

  policy = jsonencode({
    rules = [
      {
        rulePriority = 1
        description  = "Retain recent production images"

        selection = {
          tagStatus   = "tagged"
          tagPrefixList = ["release"]
          countType   = "imageCountMoreThan"
          countNumber = 50
        }

        action = {
          type = "expire"
        }
      }
    ]
  })
}
```

Test lifecycle rules carefully before enabling aggressive cleanup.

---

 # 33 ECR Repository Policy

Only required principals should be allowed to:

```text
push
pull
delete
```

Application nodes normally need pull permissions, not repository administration.

---

 # 34 IAM Architecture

Separate roles:

```text
Terraform CI role
Argo CD role
EKS node role
AWS Load Balancer Controller role
External Secrets role
application roles
developer read-only role
break-glass role
```

---

 # 35 Terraform CI Role

The CI role should have only permissions required for:

```text
terraform plan
terraform apply
state access
specific AWS resources
```

Do not automatically use:

```text
AdministratorAccess
```

for routine CI.

---

 # 36 IAM Role for Service Account

A workload may require AWS access.

Prefer workload identity:

```text
EKS Pod Identity
or
IRSA
```

rather than static access keys.

Example conceptual mapping:

```text
catalogue Pod
    |
    v
ServiceAccount
    |
    v
IAM Role
    |
    v
AWS API
```

---

 # 37 Security Group Strategy

Use purpose-specific security groups.

Example:

```text
eks-cluster-sg
eks-node-sg
alb-sg
database-sg
```

Avoid:

```text
allow all from 0.0.0.0/0
```

unless explicitly required.

---

 # 38 ALB Security Group

Internet-facing ALB typically allows:

```text
TCP 443 from approved internet ranges
```

and redirects or handles HTTP according to security requirements.

Node/application security groups should not expose application ports directly to the internet.

---

 # 39 Database Security Group

Example conceptual rule:

```text
application SG
    |
    v
database SG
port 27017
```

The database should not be:

```text
0.0.0.0/0
```

---

 # 40 Network ACLs

NACLs operate at subnet boundaries.

Use them carefully.

Security groups are generally the primary stateful workload firewall.

Do not create overly complex NACL rules without operational justification.

---

 # 41 KMS

Potential keys:

```text
EKS secrets
S3 Terraform state
CloudWatch logs where required
application data
database backups
```

Key management should include:

```text
rotation
least privilege
key policies
deletion protection considerations
```

---

 # 42 S3 State Bucket

Example:

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = var.state_bucket_name

  tags = local.common_tags
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

Production bucket configuration should also address:

```text
public access block
encryption
lifecycle
access logging where required
```

---

 # 43 S3 Public Access Block

```hcl
resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

---

 # 44 S3 Encryption

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

Use KMS if required by the security standard.

---

 # 45 Terraform State Locking

State locking prevents concurrent operations.

Without locking:

```text
Engineer A → apply
Engineer B → apply
```

can produce conflicts.

The CI pipeline should serialize production applies.

---

 # 46 State Isolation

Never use:

```text
same state file for dev + QA + prod
```

Prefer:

```text
separate state
separate IAM
separate approval
```

---

 # 47 Terraform Modules

Good modules should have:

```text
main.tf
variables.tf
outputs.tf
versions.tf
README.md
```

Example:

```text
modules/vpc/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
└── README.md
```

---

 # 48 Module Inputs

Avoid huge modules with hundreds of options.

Expose meaningful inputs:

```text
name
cidr
AZs
subnets
NAT strategy
tags
```

Defaults should be safe.

---

 # 49 Module Outputs

Example:

```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}
```

Outputs become dependencies for other modules.

---

 # 50 Root Module

Production environment:

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = local.name
  cidr = var.vpc_cidr

  availability_zones = var.availability_zones
  private_subnets    = var.private_subnets
  public_subnets     = var.public_subnets
}

module "eks" {
  source = "../../modules/eks"

  cluster_name = local.name

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}

module "ecr" {
  source = "../../modules/ecr"

  repositories = var.ecr_repositories
}
```

---

 # 51 Variables

```hcl
variable "aws_region" {
  type        = string
  description = "AWS region"
}

variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "qa", "prod"], var.environment)
    error_message = "Environment must be dev, qa, or prod."
  }
}

variable "eks_version" {
  type = string
}
```

---

 # 52 Variable Validation

Good validation catches mistakes before AWS changes.

Example:

```hcl
variable "vpc_cidr" {
  type = string

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "vpc_cidr must be a valid IPv4 CIDR."
  }
}
```

---

 # 53 Locals

```hcl
locals {
  name = "${var.project}-${var.environment}"

  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

Keep locals understandable.

---

 # 54 Outputs

Production outputs may include:

```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "eks_cluster_name" {
  value = module.eks.cluster_name
}

output "eks_cluster_endpoint" {
  value     = module.eks.cluster_endpoint
  sensitive = true
}

output "ecr_repository_urls" {
  value = module.ecr.repository_urls
}
```

Do not expose unnecessary sensitive information.

---

 # 55 Environment tfvars

Example:

```hcl
project     = "roboshop"
environment = "prod"

aws_region = "ap-south-1"

vpc_cidr = "10.10.0.0/16"

availability_zones = [
  "ap-south-1a",
  "ap-south-1b",
  "ap-south-1c"
]

eks_version = "1.33"
```

Versions must be selected according to current AWS/EKS support and organizational standards.

---

 # 56 Do Not Put Secrets in tfvars

Avoid:

```hcl
db_password = "secret123"
```

Terraform state may contain values.

Use:

```text
AWS Secrets Manager
SSM Parameter Store
external secret systems
```

for secrets.

---

 # 57 Terraform Lifecycle

```text
terraform fmt
      |
      v
terraform init
      |
      v
terraform validate
      |
      v
security scan
      |
      v
terraform plan
      |
      v
review
      |
      v
approval
      |
      v
terraform apply
```

---

 # 58 Format

```bash
terraform fmt -recursive
```

CI should fail when formatting is incorrect.

---

 # 59 Init

```bash
terraform init
```

This:

```text
downloads providers
initializes backend
downloads modules
```

---

 # 60 Validate

```bash
terraform validate
```

Checks Terraform configuration syntax and internal consistency.

---

 # 61 Plan

```bash
terraform plan \
  -var-file=terraform.tfvars \
  -out=tfplan
```

The plan should be reviewed before apply.

---

 # 62 Apply

```bash
terraform apply tfplan
```

Applying the saved plan is preferable in controlled CI workflows because the reviewed plan is the one applied.

---

 # 63 Destroy

Production:

```text
terraform destroy
```

should be heavily restricted.

Do not provide routine destroy permissions to normal CI.

---

 # 64 CI Pipeline

Recommended stages:

```text
Checkout
   |
Terraform fmt check
   |
Terraform validate
   |
Security scan
   |
Plan
   |
Policy evaluation
   |
Publish plan
   |
Manual approval
   |
Apply
   |
Post-apply validation
```

---

 # 65 Terraform Plan Security

Plan files can contain sensitive infrastructure details.

Protect:

```text
tfplan
Terraform logs
state
outputs
```

Do not publish them publicly.

---

 # 66 Security Scanning

Useful tooling:

```text
tfsec
Trivy config
Checkov
OPA/Conftest
```

Example:

```bash
trivy config .
```

Use approved organizational tooling.

---

 # 67 Policy Example

Policies can reject:

```text
public S3 bucket
0.0.0.0/0 SSH
unencrypted storage
unrestricted security group
missing tags
unapproved regions
```

---

 # 68 Example Security Requirement

Bad:

```hcl
cidr_blocks = ["0.0.0.0/0"]
```

for SSH.

Preferred:

```text
no public SSH
SSM Session Manager
bastionless administration
or tightly restricted management CIDRs
```

---

 # 69 SSM

For EC2-based administration, consider:

```text
AWS Systems Manager Session Manager
```

This can remove the need for public SSH.

---

 # 70 EKS Access Through Terraform

Terraform can establish:

```text
EKS access entries
IAM roles
cluster admin bootstrap
platform operator role
read-only role
```

Application deployment should remain in GitOps.

---

 # 71 ALB Controller Prerequisites

Terraform can create:

```text
IAM role
OIDC/provider or EKS Pod Identity configuration
security prerequisites
```

Then GitOps/Helm can install the controller if that is the chosen architecture.

---

 # 72 Terraform and Helm Boundary

Possible design:

```text
Terraform
  |
  +--> AWS infrastructure
  |
  +--> optional bootstrap Helm releases

GitOps
  |
  +--> application Helm releases
  +--> application manifests
```

Avoid having Terraform and Argo CD manage the same Kubernetes resource.

---

 # 73 Ownership Conflict

Bad:

```text
Terraform manages Deployment/catalogue
Argo CD manages Deployment/catalogue
```

This creates competing controllers.

Choose one owner.

For this capstone:

```text
Terraform → infrastructure
Argo CD    → Kubernetes desired state
```

---

 # 74 ECR Ownership

Terraform owns:

```text
repository
repository policy
lifecycle
encryption configuration
```

CI owns:

```text
image build
image push
```

GitOps owns:

```text
which image digest is deployed
```

This is a clean boundary.

---

 # 75 IAM Ownership

Terraform owns:

```text
roles
policies
trust policies
```

Applications use those roles.

GitOps owns:

```text
ServiceAccount
```

when Kubernetes-side configuration is managed there.

---

 # 76 State Drift

Terraform drift occurs when:

```text
Terraform state/desired configuration
        !=
actual AWS infrastructure
```

Example:

```text
security group manually modified
```

Run:

```bash
terraform plan
```

to detect differences.

---

 # 77 Drift Response

Do not automatically apply every drift.

First determine:

```text
Who changed it?
Why?
Was it intentional?
Is it security-relevant?
Should code be updated?
```

Then reconcile safely.

---

 # 78 Terraform Import

If an existing resource must be brought under Terraform:

```bash
terraform import \
  aws_vpc.main \
  vpc-0123456789abcdef0
```

Modern Terraform also supports import blocks.

Example:

```hcl
import {
  to = aws_vpc.main
  id = "vpc-0123456789abcdef0"
}
```

---

 # 79 Import Best Practice

After import:

```text
terraform plan
```

Do not assume import means configuration is complete.

Match the configuration to the real resource carefully.

---

 # 80 Resource Replacement

Terraform may show:

```text
-/+
```

meaning destroy and recreate.

Treat this as high risk for production.

Always inspect:

```bash
terraform plan
```

before approval.

---

 # 81 Lifecycle Ignore Changes

Use carefully:

```hcl
lifecycle {
  ignore_changes = [
    desired_capacity
  ]
}
```

This can be useful when another controller intentionally manages a field.

But broad `ignore_changes` can hide real drift.

---

 # 82 Prevent Destroy

Critical resources may use:

```hcl
lifecycle {
  prevent_destroy = true
}
```

Examples:

```text
production databases
critical KMS keys
state buckets
```

This is not a substitute for backups.

---

 # 83 EKS Upgrade Strategy

Do not blindly change:

```hcl
cluster_version = "new-version"
```

Plan:

```text
compatibility
addons
controllers
node groups
workloads
PodDisruptionBudgets
API removals
```

Then:

```text
upgrade control plane
upgrade addons
upgrade nodes
validate workloads
```

---

 # 84 EKS Add-ons

Terraform can manage approved EKS add-ons such as:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI
```

Pin versions where appropriate.

---

 # 85 EBS CSI

For persistent volumes:

```text
EBS CSI driver
```

requires appropriate IAM permissions.

Storage classes can then be managed through Kubernetes/GitOps.

---

 # 86 S3 and EBS Backup

Terraform can establish:

```text
AWS Backup plans
backup vaults
KMS
backup policies
```

Actual workload backup configuration should be tested separately.

---

 # 87 CloudWatch

Terraform can create:

```text
log groups
retention
alarms
dashboards where required
```

Example:

```hcl
resource "aws_cloudwatch_log_group" "eks" {
  name              = "/aws/eks/${local.name}/cluster"
  retention_in_days = 30

  tags = local.common_tags
}
```

Retention should match operational and compliance requirements.

---

 # 88 AWS Alarms vs Prometheus

AWS-native signals:

```text
ALB
EKS infrastructure
NAT gateway
AWS services
```

can use:

```text
CloudWatch alarms
```

Kubernetes/application metrics can use:

```text
Prometheus
```

Use both where appropriate.

---

 # 89 Terraform and Prometheus

Terraform should not normally create every Prometheus alert.

Instead:

```text
Terraform
  → infrastructure

GitOps
  → PrometheusRule
  → Grafana
  → Alertmanager
```

This keeps Kubernetes observability under GitOps.

---

 # 90 Terraform and ELK

Terraform can create AWS prerequisites:

```text
S3
IAM
security groups
networking
```

but Kubernetes-side:

```text
Logstash
Elasticsearch
Kibana
```

should be owned by the approved Kubernetes/platform deployment mechanism.

---

 # 91 Complete Production Environment

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name               = local.name
  cidr               = var.vpc_cidr
  availability_zones = var.availability_zones
  private_subnets    = var.private_subnets
  public_subnets     = var.public_subnets

  enable_nat_gateway = true

  tags = local.common_tags
}

module "eks" {
  source = "../../modules/eks"

  cluster_name    = local.name
  cluster_version = var.eks_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids

  enable_cluster_creator_admin_permissions = false

  tags = local.common_tags
}

module "ecr" {
  source = "../../modules/ecr"

  repositories = var.ecr_repositories

  tags = local.common_tags
}
```

---

 # 92 Complete variables

```hcl
variable "project" {
  type    = string
  default = "roboshop"
}

variable "environment" {
  type = string
}

variable "aws_region" {
  type = string
}

variable "vpc_cidr" {
  type = string
}

variable "availability_zones" {
  type = list(string)
}

variable "private_subnets" {
  type = list(string)
}

variable "public_subnets" {
  type = list(string)
}

variable "eks_version" {
  type = string
}

variable "ecr_repositories" {
  type = set(string)
}
```

---

 # 93 Complete ECR Repository List

```hcl
ecr_repositories = [
  "roboshop/frontend",
  "roboshop/catalogue",
  "roboshop/cart",
  "roboshop/user",
  "roboshop/payment",
  "roboshop/shipping",
  "roboshop/checkout"
]
```

---

 # 94 Production tfvars

```hcl
project     = "roboshop"
environment = "prod"

aws_region = "ap-south-1"

vpc_cidr = "10.10.0.0/16"

availability_zones = [
  "ap-south-1a",
  "ap-south-1b",
  "ap-south-1c"
]

private_subnets = [
  "10.10.64.0/20",
  "10.10.80.0/20",
  "10.10.96.0/20"
]

public_subnets = [
  "10.10.0.0/20",
  "10.10.16.0/20",
  "10.10.32.0/20"
]

eks_version = "1.33"

ecr_repositories = [
  "roboshop/frontend",
  "roboshop/catalogue",
  "roboshop/cart",
  "roboshop/user",
  "roboshop/payment",
  "roboshop/shipping",
  "roboshop/checkout"
]
```

These are example values. Validate CIDRs, AZ availability and supported EKS versions before use.

---

 # 95 Production Backend

```hcl
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "roboshop/prod/terraform.tfstate"
    region = "ap-south-1"

    encrypt = true
  }
}
```

The CI identity should have access only to the appropriate state path.

---

 # 96 CI Role Separation

Example:

```text
terraform-plan-role
terraform-apply-role
```

Plan may have read-heavy permissions.

Apply has controlled write permissions.

A separate approval step should be used for production.

---

 # 97 GitHub Actions Example

```yaml
name: Terraform

on:
  pull_request:
    paths:
      - "environments/**"
      - "modules/**"

permissions:
  id-token: write
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.TERRAFORM_PLAN_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Init
        working-directory: environments/prod
        run: terraform init

      - name: Format
        working-directory: environments/prod
        run: terraform fmt -check -recursive

      - name: Validate
        working-directory: environments/prod
        run: terraform validate

      - name: Plan
        working-directory: environments/prod
        run: terraform plan -out=tfplan
```

Pin action versions according to organizational policy and supply-chain requirements.

---

 # 98 Terraform Apply Workflow

A production apply can be:

```text
PR
 |
 v
plan
 |
 v
review
 |
 v
merge
 |
 v
protected deployment workflow
 |
 v
approval
 |
 v
apply
```

The exact implementation may use GitHub Actions, Jenkins or another approved CI platform.

---

 # 99 Jenkins Terraform Pipeline

Conceptual stages:

```groovy
pipeline {
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fmt') {
            steps {
                sh 'terraform fmt -check -recursive'
            }
        }

        stage('Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy config .'
            }
        }

        stage('Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Approval') {
            steps {
                input message: 'Approve production Terraform apply?'
            }
        }

        stage('Apply') {
            steps {
                sh 'terraform apply tfplan'
            }
        }
    }
}
```

The actual Jenkins implementation must authenticate using a secure workload identity rather than static AWS credentials.

---

 # 100 Terraform Concurrency

Only one production apply should run at a time.

Use:

```text
CI concurrency controls
state locking
environment locks
```

---

 # 101 Terraform Workspace Strategy

Workspaces can be useful, but for major environments many teams prefer separate root modules/state.

For example:

```text
environments/dev
environments/qa
environments/prod
```

This makes production configuration and access boundaries explicit.

---

 # 102 Production Approval

Production changes should have:

```text
plan review
risk assessment
approval
maintenance window where appropriate
rollback plan
post-apply validation
```

---

 # 103 Terraform Rollback

Terraform does not have a simple universal:

```bash
terraform rollback
```

Instead:

```text
identify previous configuration
restore code
run plan
review
apply
```

Infrastructure rollback can be dangerous because resource recreation may cause data loss.

---

 # 104 Terraform State Recovery

If state is lost:

```text
restore S3 version
```

or:

```text
reconstruct/import resources
```

S3 versioning is therefore important.

---

 # 105 Disaster Recovery

Terraform repository is a critical DR asset.

Recovery:

```text
new AWS environment
      |
      v
restore Terraform backend
      |
      v
terraform init
      |
      v
terraform plan
      |
      v
terraform apply
      |
      v
EKS
      |
      v
Argo CD
      |
      v
GitOps
```

---

 # 106 DR Region

For multi-region DR:

```text
Primary:
ap-south-1

DR:
approved secondary region
```

Terraform can define the secondary infrastructure.

Do not assume that duplicating Terraform code automatically provides application-level DR.

Data replication must also exist.

---

 # 107 Multi-Region Architecture

```text
                    Git
                     |
               Terraform / GitOps
                     |
          +----------+----------+
          |                     |
          v                     v
      Primary AWS           DR AWS
          |                     |
        EKS-A                 EKS-B
          |                     |
        ALB-A                ALB-B
          |                     |
          +---------+-----------+
                    |
              DNS / Global
              traffic strategy
```

The exact DNS/failover mechanism depends on RTO/RPO requirements.

---

 # 108 Terraform Dependencies

Example:

```text
VPC
 |
 +--> EKS
 |
 +--> ALB prerequisites
 |
 +--> ECR
 |
 +--> IAM
```

Terraform's dependency graph handles resource ordering.

Avoid unnecessary explicit `depends_on`.

---

 # 109 Explicit depends_on

Use only when Terraform cannot infer dependency.

Bad:

```hcl
depends_on = [module.vpc]
```

everywhere.

Prefer actual references:

```hcl
vpc_id = module.vpc.vpc_id
```

Terraform then understands the dependency naturally.

---

 # 110 Data Sources

Use data sources to query existing resources.

Example:

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
```

Be careful with dynamic data in production because changes in AWS can alter plans unexpectedly.

---

 # 111 Data Source vs Resource

Resource:

```text
Terraform creates/manages it
```

Data source:

```text
Terraform reads it
```

Example:

```hcl
data "aws_caller_identity" "current" {}
```

---

 # 112 Provider Lock File

Commit:

```text
.terraform.lock.hcl
```

This improves provider reproducibility.

Do not casually delete it.

---

 # 113 Version Pinning

Pin:

```text
Terraform version
AWS provider
modules
CI actions
```

Avoid uncontrolled floating versions in production.

---

 # 114 Module Versioning

A mature module repository can use:

```text
v1.0.0
v1.1.0
v2.0.0
```

Breaking changes should be versioned.

---

 # 115 Terraform Repository CODEOWNERS

Example:

```text
* @platform-team

/modules/ @platform-team
/environments/prod/ @platform-team @sre-team
/policies/ @security-team
```

---

 # 116 Terraform Repository README

Document:

```text
architecture
AWS accounts
regions
state
module usage
environment structure
authentication
plan/apply
security
DR
upgrade procedure
emergency access
```

---

 # 117 Terraform Production Checklist

```text
[ ] Remote state
[ ] State encryption
[ ] State versioning
[ ] State access control
[ ] Locking
[ ] Provider pinned
[ ] Terraform version pinned
[ ] Modules versioned
[ ] No secrets in code
[ ] OIDC/workload identity
[ ] Production approval
[ ] Security scan
[ ] Policy checks
[ ] Plan review
[ ] Apply serialization
[ ] Backups
[ ] DR procedure
```

---

 # 118 VPC Checklist

```text
[ ] multi-AZ
[ ] private EKS subnets
[ ] public ALB subnets
[ ] route tables
[ ] NAT strategy
[ ] VPC endpoints
[ ] DNS enabled
[ ] flow logs where required
[ ] CIDR capacity
[ ] security groups
[ ] NACL review
```

---

 # 119 EKS Checklist

```text
[ ] supported version
[ ] private endpoint strategy
[ ] control plane logs
[ ] encryption
[ ] node groups
[ ] autoscaling
[ ] EBS CSI
[ ] VPC CNI
[ ] CoreDNS
[ ] kube-proxy
[ ] IAM access
[ ] workload identity
[ ] upgrade plan
```

---

 # 120 ECR Checklist

```text
[ ] immutable tags
[ ] scan on push
[ ] lifecycle policy
[ ] encryption
[ ] least-privilege policy
[ ] repository naming
[ ] retention
[ ] CI push role
[ ] EKS pull role
```

---

 # 121 IAM Checklist

```text
[ ] least privilege
[ ] no static keys
[ ] CI OIDC
[ ] workload identity
[ ] separated roles
[ ] trust policies
[ ] break-glass process
[ ] audit logging
```

---

 # 122 Production Incident — Terraform Failure

Symptom:

```text
terraform apply failed
```

Investigation:

```bash
terraform plan
```

Then inspect:

```text
AWS API error
IAM permissions
state lock
resource limits
dependency
```

---

 # 123 Failure — State Lock

Symptom:

```text
Error acquiring state lock
```

Possible cause:

```text
another Terraform operation is running
```

Check CI jobs before attempting lock recovery.

Never blindly force-unlock a state lock while another operation may still be active.

---

 # 124 Failure — AccessDenied

Symptom:

```text
AccessDenied
```

Check:

```text
CI identity
assumed role
trust policy
IAM policy
resource policy
AWS account
AWS region
```

Useful:

```bash
aws sts get-caller-identity
```

---

 # 125 Failure — Wrong AWS Account

Always verify:

```bash
aws sts get-caller-identity
```

before sensitive operations.

In CI, log the account ID safely.

---

 # 126 Failure — Unexpected Destroy

If plan shows:

```text
-/+
```

or:

```text
destroy
```

stop.

Investigate:

```text
resource address
changed argument
state mismatch
provider behavior
module upgrade
```

Do not approve automatically.

---

 # 127 Failure — EKS Unavailable

Check:

```bash
aws eks describe-cluster \
  --name roboshop-prod \
  --region ap-south-1
```

Then:

```text
AWS health
IAM
networking
control plane status
endpoint configuration
```

---

 # 128 Failure — Nodes Not Joining

Check:

```text
node IAM role
subnet routes
security groups
VPC DNS
EKS bootstrap
AMI compatibility
cluster endpoint
CNI
```

Useful:

```bash
kubectl get nodes
```

and AWS console/CLI node-group status.

---

 # 129 Failure — ECR Push Denied

Check:

```text
CI role
ECR permissions
repository existence
region
registry URI
```

Authenticate:

```bash
aws ecr get-login-password --region ap-south-1 |
docker login \
  --username AWS \
  --password-stdin <account>.dkr.ecr.ap-south-1.amazonaws.com
```

---

 # 130 Failure — Terraform Drift

Symptom:

```text
terraform plan shows unexpected changes
```

Investigate:

```text
manual AWS console change
controller-managed field
provider upgrade
module change
state issue
```

Do not blindly apply.

---

 # 131 Production Change Example

Request:

```text
Increase EKS node group maximum from 10 to 20.
```

Flow:

```text
change tfvars
 |
 v
PR
 |
 v
Terraform plan
 |
 v
review
 |
 v
approval
 |
 v
apply
 |
 v
AWS node capacity
 |
 v
Kubernetes scheduling
```

Then validate:

```bash
kubectl get nodes
kubectl get pods -A
```

---

 # 132 Terraform and GitOps Deployment

The complete lifecycle:

```text
Terraform
 |
 +--> VPC
 +--> EKS
 +--> ECR
 +--> IAM
 |
 v
EKS ready
 |
 v
Argo CD bootstrap
 |
 v
GitOps
 |
 v
Applications
```

Infrastructure must exist before dependent Kubernetes resources can operate.

---

 # 133 Bootstrap Dependency

A clean approach:

```text
Layer 1:
AWS infrastructure

Layer 2:
EKS platform controllers

Layer 3:
Argo CD

Layer 4:
applications
```

---

 # 134 Layer 1 — Terraform

Terraform:

```text
VPC
EKS
ECR
IAM
KMS
networking
```

---

 # 135 Layer 2 — Platform

Platform deployment:

```text
AWS Load Balancer Controller
EBS CSI
External Secrets
Prometheus
Grafana
ELK
```

The selected mechanism may be Helm/GitOps after Argo CD bootstrap.

---

 # 136 Layer 3 — Argo CD

Argo CD manages:

```text
platform applications
business applications
monitoring
policies
```

---

 # 137 Layer 4 — RoboShop

```text
frontend
catalogue
cart
user
payment
shipping
checkout
```

---

 # 138 Terraform vs GitOps Decision Table

| Requirement | Terraform | GitOps |
|---|---|---|
| VPC | Yes | No |
| Subnets | Yes | No |
| EKS cluster | Yes | No |
| ECR repository | Yes | No |
| IAM roles | Yes | No |
| KMS | Yes | No |
| Kubernetes Deployment | No | Yes |
| Kubernetes Service | No | Yes |
| HPA | No | Yes |
| PDB | No | Yes |
| PrometheusRule | No | Yes |
| Alertmanager | No | Yes |
| Argo Application | Usually no | Yes |
| Application image version | No | Yes |

The exact boundary can vary, but avoid dual ownership.

---

 # 139 Production Cost Considerations

Terraform should help enforce:

```text
right-sized instances
AZ strategy
NAT strategy
VPC endpoints
ECR lifecycle
log retention
unused resource cleanup
```

Cost optimization must not destroy required production resilience.

---

 # 140 Tagging for Cost

Every resource should receive:

```text
Project
Environment
Owner
CostCenter
ManagedBy
```

Where AWS service support permits tagging.

---

 # 141 Terraform Documentation

Every module should explain:

```text
purpose
inputs
outputs
dependencies
examples
security
upgrade notes
```

---

 # 142 Module README Example

```text
# VPC Module

Creates:
- VPC
- public subnets
- private subnets
- route tables
- NAT gateways
- internet gateway

Inputs:
- name
- cidr
- availability_zones
- private_subnets
- public_subnets

Outputs:
- vpc_id
- public_subnet_ids
- private_subnet_ids
```

---

 # 143 Testing

Terraform testing can include:

```text
terraform validate
terraform plan
policy tests
static analysis
integration tests
```

For mature teams:

```text
terraform test
```

can validate module behavior where suitable.

---

 # 144 Terraform Test Philosophy

Test:

```text
valid inputs
invalid inputs
security controls
expected outputs
module behavior
```

Do not rely only on successful `terraform validate`.

---

 # 145 Production Plan Review

Senior engineer reviews:

```text
resource additions
resource changes
resource replacements
resource deletions
IAM changes
security group changes
network changes
KMS changes
EKS changes
```

The highest risk is often not the number of resources but the type of change.

---

 # 146 Emergency Terraform Change

Emergency infrastructure changes should still have:

```text
incident ID
reason
approval
minimum scope
post-change review
Git reconciliation
```

Never leave production infrastructure permanently changed only through the console.

---

 # 147 Manual AWS Change

If an emergency console change is required:

```text
AWS console
   |
   v
temporary mitigation
   |
   v
update Terraform
   |
   v
plan
   |
   v
reconcile
```

Otherwise future Terraform runs may undo it.

---

 # 148 Terraform Drift Monitoring

Run scheduled:

```bash
terraform plan
```

in a controlled environment.

If changes exist:

```text
notify infrastructure team
```

Do not automatically apply drift unless explicitly designed and approved.

---

 # 149 Security Hardening

Terraform repository should enforce:

```text
no plaintext credentials
least privilege
encrypted state
private networking
restricted security groups
private EKS endpoint where appropriate
KMS
logging
tagging
approved regions
policy checks
```

---

 # 150 Supply Chain Security

Protect:

```text
Terraform provider
Terraform modules
CI actions
container images
Git repository
```

Pin trusted versions and review dependency changes.

---

 # 151 Complete Directory Example

```text
roboshop-infrastructure/
│
├── README.md
├── CODEOWNERS
├── .gitignore
├── .terraform.lock.hcl
│
├── bootstrap/
│   └── backend/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── versions.tf
│   │   └── README.md
│   │
│   ├── eks/
│   ├── ecr/
│   ├── iam/
│   ├── kms/
│   ├── security-groups/
│   ├── vpc-endpoints/
│   └── cloudwatch/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── policies/
│   ├── terraform/
│   └── security/
│
└── scripts/
    ├── fmt-check.sh
    ├── validate.sh
    ├── plan.sh
    └── security-scan.sh
```

---

 # 152 Production Scripts

## fmt-check.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

terraform fmt -check -recursive
```

## validate.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

terraform init -backend=false
terraform validate
```

## security-scan.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

trivy config .
```

---

 # 153 Plan Script

```bash
#!/usr/bin/env bash
set -euo pipefail

ENVIRONMENT="${1:?environment required}"

cd "environments/${ENVIRONMENT}"

terraform init

terraform plan \
  -out=tfplan
```

CI should inject credentials and variable values securely.

---

 # 154 Production Apply Script

```bash
#!/usr/bin/env bash
set -euo pipefail

ENVIRONMENT="${1:?environment required}"

if [[ "$ENVIRONMENT" != "prod" ]]; then
  echo "This script is intended for production only."
  exit 1
fi

cd "environments/prod"

terraform apply tfplan
```

Production approval should happen in CI before invoking this.

---

 # 155 Complete Terraform Workflow

```text
Developer
   |
   v
Terraform PR
   |
   +--> fmt
   +--> validate
   +--> Trivy/Checkov
   +--> policy
   +--> plan
   |
   v
Reviewer
   |
   v
Merge
   |
   v
Production approval
   |
   v
Terraform apply
   |
   v
AWS
   |
   v
EKS
   |
   v
Argo CD
   |
   v
GitOps
```

---

 # 156 Senior Interview — What Does Terraform Manage?

Strong answer:

```text
I use Terraform primarily for infrastructure.

In this architecture that includes the VPC, subnets, routing, security groups, IAM roles, EKS, ECR, KMS and AWS-native supporting resources.

I keep Kubernetes application desired state in the GitOps repository managed by Argo CD.

That separation prevents Terraform and Argo CD from competing over the same Kubernetes resources.
```

---

 # 157 Senior Interview — Why Remote State?

Answer:

```text
Production Terraform state cannot live on an engineer's laptop.

Remote state provides centralized storage, access control, encryption, versioning and concurrency protection.

It also makes CI/CD-based infrastructure operations possible.
```

---

 # 158 Senior Interview — How Do You Secure State?

Answer:

```text
I store state in a private encrypted S3 bucket with versioning and restricted IAM access.

I avoid putting secrets directly into Terraform variables or code because sensitive values can end up in state.

The CI identity gets only the state access and AWS permissions it needs.
```

---

 # 159 Senior Interview — Terraform Drift

Answer:

```text
Terraform drift occurs when infrastructure changes outside the Terraform configuration.

I first run a plan and identify whether the change was intentional.

If it was intentional, I update Terraform configuration.
If it was unauthorized, I investigate and reconcile it carefully.

I don't blindly apply a plan because it may overwrite an emergency or controller-managed change.
```

---

 # 160 Senior Interview — Terraform Rollback

Answer:

```text
Terraform doesn't provide a universal rollback button.

For an infrastructure change, I restore the previous known-good Terraform configuration, generate a new plan, review resource replacement and data-loss risk, and then apply it.

For stateful resources I treat rollback especially carefully because recreation can be destructive.
```

---

 # 161 Senior Interview — Terraform vs Ansible

Answer:

```text
Terraform is primarily infrastructure provisioning and desired-state management for infrastructure resources.

Ansible is commonly used for configuration and procedural automation.

In this architecture Terraform creates AWS infrastructure, while Kubernetes and Argo CD manage application deployment.
```

---

 # 162 Senior Interview — Why Not Terraform Everything?

Answer:

```text
I avoid using one tool for every layer.

Terraform is excellent for AWS infrastructure.
Argo CD is designed for Kubernetes reconciliation.

If Terraform and Argo CD both manage the same Kubernetes Deployment, they can fight over desired state.

Clear ownership boundaries make the platform easier to operate.
```

---

 # 163 Senior Interview — How Would You Rebuild Production?

Answer:

```text
I would restore the Terraform backend and repository, recreate the AWS foundation from Terraform, bootstrap EKS and required platform dependencies, install Argo CD, connect it to the GitOps repository, and allow Argo CD to reconcile workloads.

Data recovery is handled separately according to the database backup and replication strategy.
```

---

 # 164 Senior Interview — EKS Security

Answer:

```text
I use private subnets for worker nodes, restrict the EKS API endpoint, use IAM-based access, Kubernetes RBAC, workload identity, encryption, security groups, NetworkPolicies and control-plane logging.

I also avoid static AWS credentials inside pods.
```

---

 # 165 Senior Interview — ECR Security

Answer:

```text
I use immutable image references, scan images, apply lifecycle policies, restrict repository permissions, and deploy production images by digest.

CI gets push access while workloads get only the pull access they require.
```

---

 # 166 Senior Interview — Infrastructure Change Process

Answer:

```text
Every production Terraform change starts with a pull request.

CI runs formatting, validation, security checks, policy checks and terraform plan.

The plan is reviewed, production approval is obtained, and the approved plan is applied through controlled CI.

After apply, I validate AWS and dependent Kubernetes services.
```

---

 # 167 Production Terraform Principles

```text
Infrastructure as Code
        +
Remote State
        +
Least Privilege
        +
Immutable Artifacts
        +
Policy Validation
        +
Production Approval
        +
Git Review
        +
DR
        =
Reliable Infrastructure Delivery
```

---

 # 168 Final Architecture

```text
                 ┌──────────────────────┐
                 │ Terraform Git        │
                 └──────────┬───────────┘
                            |
                            v
                 ┌──────────────────────┐
                 │ CI                  │
                 │ fmt/validate/scan   │
                 │ plan/policy         │
                 └──────────┬───────────┘
                            |
                         approval
                            |
                            v
                 ┌──────────────────────┐
                 │ Terraform Apply     │
                 └──────────┬───────────┘
                            |
                            v
                       AWS Account
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
       VPC                 EKS                 ECR
        |                   |                   |
        |                   v                   |
        |              Argo CD                 |
        |                   |                   |
        |                   v                   |
        |              GitOps Repo             |
        |                   |                   |
        |                   v                   |
        |              RoboShop Apps <----------+
        |
        +--> ALB
        +--> NAT
        +--> VPC Endpoints
        +--> Security Controls
        +--> KMS
```

---

 # 169 Final Production Checklist

```text
Repository
[ ] Separate infrastructure repo
[ ] README
[ ] CODEOWNERS
[ ] provider lock file
[ ] module versioning
[ ] environment separation

State
[ ] Remote backend
[ ] Encryption
[ ] Versioning
[ ] Locking
[ ] Restricted access
[ ] Recovery process

AWS
[ ] VPC
[ ] Multi-AZ
[ ] Private EKS subnets
[ ] NAT strategy
[ ] VPC endpoints
[ ] EKS
[ ] ECR
[ ] IAM
[ ] KMS
[ ] CloudWatch

Security
[ ] No static credentials
[ ] OIDC/workload identity
[ ] Least privilege
[ ] Security scanning
[ ] Policy validation
[ ] Restricted security groups
[ ] Private API strategy

Operations
[ ] Plan review
[ ] Production approval
[ ] Serialized apply
[ ] Drift detection
[ ] Backup
[ ] DR
[ ] Upgrade procedure

GitOps boundary
[ ] Terraform owns infrastructure
[ ] Argo CD owns Kubernetes desired state
[ ] No dual resource ownership
```

---

 # 170 Final Takeaway

A production Terraform repository should be treated as a critical engineering system, not merely a collection of `.tf` files.

The desired operating model is:

```text
Git
 ↓
Terraform CI
 ↓
Format
 ↓
Validate
 ↓
Security Scan
 ↓
Policy
 ↓
Plan
 ↓
Review
 ↓
Approval
 ↓
Apply
 ↓
AWS Infrastructure
 ↓
EKS
 ↓
Argo CD
 ↓
GitOps
 ↓
Applications
```

The most important architectural boundary is:

```text
Terraform manages infrastructure.

Argo CD manages Kubernetes desired state.

CI builds artifacts.

ECR stores immutable artifacts.

Git provides the audit trail.
```

That separation gives the production DevOps platform clear ownership, controlled changes, repeatability, security and disaster-recovery capability.
