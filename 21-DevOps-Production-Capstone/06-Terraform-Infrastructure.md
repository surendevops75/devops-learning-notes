# 06 — Terraform Infrastructure

## 1. Purpose

This document defines the production Terraform implementation for the DevOps Production Capstone.

The objective is to convert the AWS account and VPC architecture into a repeatable Infrastructure as Code platform.

The Terraform design must provide:

```text
Reproducibility
Consistency
Security
Version Control
Peer Review
Change Visibility
Environment Isolation
State Safety
Drift Detection
Controlled Production Changes
```

Terraform is not simply used to create AWS resources.

In a production DevOps environment, Terraform becomes the controlled delivery mechanism for infrastructure.

---

# 2. Terraform Architecture

Target flow:

```text
Developer
   |
   v
Git Repository
   |
   v
Pull Request
   |
   v
Terraform fmt
   |
   v
Terraform validate
   |
   v
Security / Policy Checks
   |
   v
Terraform Plan
   |
   v
Review / Approval
   |
   v
Terraform Apply
   |
   v
AWS
```

Production infrastructure should be changed through this controlled workflow.

---

# 3. Infrastructure Layers

The capstone infrastructure is divided into logical layers:

```text
AWS Organization
        |
Accounts
        |
Networking
        |
Security
        |
EKS
        |
Supporting Services
        |
Applications
```

Terraform should reflect these dependencies.

---

# 4. Recommended Repository

Example:

```text
terraform-infrastructure/
|
+-- README.md
+-- versions.tf
+-- providers.tf
+-- backend.tf
+-- locals.tf
+-- variables.tf
+-- outputs.tf
|
+-- modules/
|   |
|   +-- vpc/
|   +-- security-groups/
|   +-- iam/
|   +-- kms/
|   +-- endpoints/
|   +-- flow-logs/
|   +-- eks/
|   +-- node-groups/
|   +-- ecr/
|   +-- route53/
|   +-- alb/
|   +-- monitoring/
|
+-- environments/
    |
    +-- dev/
    +-- staging/
    +-- production/
    +-- dr/
```

---

# 5. Why Modules?

Modules provide reusable infrastructure building blocks.

Example:

```text
VPC module
   |
+-- Dev VPC
+-- Staging VPC
+-- Production VPC
+-- DR VPC
```

Instead of copying hundreds of lines between environments, the module contains reusable logic while environment configuration supplies inputs.

---

# 6. Module Design Principle

A good module should have:

```text
Clear Inputs
Predictable Outputs
Minimal Hidden Behavior
Documentation
Validation
Versioning
Tests
```

Avoid modules that try to create an entire enterprise platform with hundreds of unrelated resources.

---

# 7. Root Module vs Child Module

Root module:

```text
environments/production
```

Child modules:

```text
modules/vpc
modules/eks
modules/ecr
```

Example:

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = "production"
  cidr = "10.0.0.0/16"
}
```

---

# 8. Terraform Version Pinning

Pin the Terraform version.

Example:

```hcl
terraform {
  required_version = "~> 1.9"
}
```

The exact version should be selected and upgraded deliberately.

Avoid:

```hcl
required_version = ">= 0.12"
```

because very broad version ranges make behavior less predictable.

---

# 9. Provider Version Pinning

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

Provider versions should be reviewed and upgraded through controlled changes.

---

# 10. Provider Lock File

Commit:

```text
.terraform.lock.hcl
```

This records provider selections and checksums.

It improves:

```text
Reproducibility
Security
CI consistency
```

---

# 11. Provider Configuration

Example:

```hcl
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
      Project     = "devops-capstone"
    }
  }
}
```

Default tags reduce repeated configuration.

---

# 12. Required Provider Configuration

Typical production stack:

```text
AWS Provider
TLS Provider
Kubernetes Provider
Helm Provider
```

Not every layer should use every provider.

Keep provider usage intentional.

---

# 13. AWS Provider Aliases

Multi-account deployments can use provider aliases.

Example:

```hcl
provider "aws" {
  region = var.aws_region
}

provider "aws" {
  alias  = "security"
  region = var.aws_region
}
```

A module can receive the required provider:

```hcl
providers = {
  aws = aws.security
}
```

---

# 14. Cross-Account Role Assumption

A CI role in one account may assume a role in another account.

Conceptually:

```text
GitLab OIDC
    |
CI Role
    |
STS AssumeRole
    |
Production Terraform Role
    |
AWS Resources
```

Do not distribute long-lived AWS access keys to CI.

---

# 15. OIDC Authentication

Preferred CI architecture:

```text
GitLab
   |
OIDC Token
   |
AWS STS
   |
AssumeRoleWithWebIdentity
   |
Temporary Credentials
```

Benefits:

```text
No long-lived secret
Short-lived credentials
Auditable access
Scoped trust policy
```

---

# 16. Terraform State

Terraform state is critical.

It records:

```text
Resources
IDs
Attributes
Dependencies
Provider information
Terraform-managed metadata
```

Never treat state as disposable.

---

# 17. Remote State

Production state should normally be stored remotely.

Example:

```text
Terraform
   |
S3 State
   |
Versioning
   |
Encryption
   |
Restricted IAM
```

Use the currently supported locking mechanism for the selected Terraform/AWS architecture and document it explicitly.

---

# 18. State Isolation

Do not use one state file for every environment.

Prefer:

```text
dev state
staging state
production state
dr state
```

This limits blast radius.

---

# 19. State Key Structure

Example:

```text
terraform-state/
|
+-- dev/network/terraform.tfstate
+-- staging/network/terraform.tfstate
+-- production/network/terraform.tfstate
+-- dr/network/terraform.tfstate
```

A production platform may further separate:

```text
network
security
eks
shared-services
```

when independent lifecycle and blast-radius requirements justify it.

---

# 20. State Bucket Security

The state bucket should have:

```text
Encryption
Versioning
Block Public Access
Restricted IAM
Audit logging
Lifecycle management
```

Never make Terraform state public.

---

# 21. State Contains Sensitive Information

Terraform state may contain sensitive values depending on resources.

Therefore:

```text
State = Sensitive Infrastructure Data
```

Protect it like production infrastructure.

---

# 22. State Locking

Concurrent Terraform applies can cause corruption or conflicting changes.

Use a supported locking strategy for the Terraform version and backend design.

The objective:

```text
Engineer A -> Lock -> Apply
Engineer B -> Wait
```

Do not allow multiple production applies simultaneously.

---

# 23. State Recovery

If state becomes damaged:

```text
Stop Terraform
   |
Identify last known good version
   |
Backup state
   |
Review version history
   |
Restore carefully
   |
Validate
   |
Plan
```

Never overwrite state blindly.

---

# 24. Terraform State Commands

Useful commands:

```bash
terraform state list
terraform state show <address>
terraform state mv <source> <destination>
terraform state rm <address>
```

These commands are powerful and should be used carefully.

---

# 25. State Move

When refactoring:

```text
module.old.aws_vpc.main
```

to:

```text
module.network.aws_vpc.main
```

Terraform must understand that the real resource did not change.

Use:

```text
terraform state mv
```

or modern configuration-driven state migration mechanisms where appropriate.

---

# 26. State Remove

Do not use:

```bash
terraform state rm
```

as a way to "fix" an unwanted resource without understanding the consequences.

It removes Terraform's management relationship.

The AWS resource may continue to exist.

---

# 27. Terraform Import

If an existing resource must be brought under management:

```bash
terraform import <resource-address> <resource-id>
```

Modern Terraform also supports import blocks.

The important production rule:

```text
Import
+
Write matching configuration
+
Plan
+
Review
```

---

# 28. Drift

Drift occurs when infrastructure changes outside Terraform.

Example:

```text
Terraform:
ALB timeout = 60

AWS console:
ALB timeout = 120
```

Terraform detects a difference during plan.

---

# 29. Drift Prevention

Use:

```text
Restricted console access
IAM controls
Terraform-only production changes
AWS Config
CloudTrail
Scheduled plans
```

The goal is not merely detecting drift.

The goal is preventing uncontrolled drift.

---

# 30. Terraform Plan

Plan shows the proposed changes.

Typical symbols:

```text
+ create
- destroy
~ update
-/+ replace
```

Never interpret a large plan as automatically safe.

Read every destructive change.

---

# 31. Production Plan Review

Review:

```text
Resources created
Resources changed
Resources destroyed
IAM changes
Network changes
Security changes
Data resources
EKS resources
Potential replacements
```

For production, a human should understand the plan before apply.

---

# 32. Terraform Apply

Normal workflow:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

Saving the plan creates a reviewable artifact.

CI should use the approved plan artifact for the corresponding apply stage where practical.

---

# 33. Avoid Manual Production Apply

Bad:

```text
Engineer laptop
   |
AWS credentials
   |
terraform apply
```

Preferred:

```text
Pull Request
   |
CI
   |
Plan
   |
Approval
   |
CI Apply
```

This creates auditability.

---

# 34. Environment Strategy

Use separate root configurations:

```text
environments/
|
+-- dev/
+-- staging/
+-- production/
+-- dr/
```

Each environment can have:

```text
backend
providers
variables
module configuration
```

---

# 35. Workspaces vs Directories

Terraform workspaces can separate state, but they are not a complete environment isolation strategy.

For production:

```text
Separate environment roots
+
Separate state
```

is often easier to reason about.

Workspaces can still be useful for controlled use cases.

---

# 36. Production Directory

Example:

```text
environments/production/
|
+-- backend.tf
+-- providers.tf
+-- main.tf
+-- variables.tf
+-- outputs.tf
+-- terraform.tfvars
```

Sensitive values should not be committed to Git.

---

# 37. Variables

Example:

```hcl
variable "environment" {
  type        = string
  description = "Environment name"

  validation {
    condition = contains(
      ["dev", "staging", "production", "dr"],
      var.environment
    )

    error_message = "Invalid environment."
  }
}
```

Validation prevents accidental invalid configuration.

---

# 38. Sensitive Variables

Example:

```hcl
variable "database_password" {
  type      = string
  sensitive = true
}
```

However:

```text
sensitive = true
```

does not mean the value is absent from state.

Use a secrets management architecture.

---

# 39. Secrets Architecture

Prefer:

```text
AWS Secrets Manager
       |
Terraform references metadata
       |
Application retrieves secret
```

Avoid putting production passwords directly into:

```text
.tf files
.tfvars committed to Git
CI variables without controls
```

---

# 40. Locals

Use locals for derived values.

Example:

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"

  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

Avoid excessive locals that hide simple logic.

---

# 41. Naming Convention

Recommended:

```text
<project>-<environment>-<component>
```

Examples:

```text
roboshop-production-vpc
roboshop-production-eks
roboshop-production-alb
```

Consistent names improve operations.

---

# 42. Tagging Strategy

Recommended tags:

```text
Project
Environment
Owner
ManagedBy
CostCenter
Application
Criticality
DataClassification
```

Tags support:

```text
Cost allocation
Inventory
Operations
Security
Automation
```

---

# 43. VPC Module

The VPC module should support:

```text
CIDR
AZs
Public subnets
Private app subnets
Private data subnets
NAT
Route tables
Endpoints
Flow Logs
Tags
```

---

# 44. VPC Module Inputs

Example:

```hcl
variable "name" {
  type = string
}

variable "vpc_cidr" {
  type = string
}

variable "availability_zones" {
  type = list(string)
}

variable "public_subnet_cidrs" {
  type = list(string)
}

variable "private_app_subnet_cidrs" {
  type = list(string)
}

variable "private_data_subnet_cidrs" {
  type = list(string)
}
```

---

# 45. VPC Module Validation

Validate matching lengths:

```text
AZ count
Public subnet count
Private app subnet count
Private data subnet count
```

Example:

```hcl
validation {
  condition = length(var.availability_zones) == length(var.public_subnet_cidrs)
  error_message = "Each AZ requires a public subnet CIDR."
}
```

---

# 46. VPC Module Outputs

Output only useful values:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "private_app_subnet_ids" {
  value = aws_subnet.private_app[*].id
}

output "private_data_subnet_ids" {
  value = aws_subnet.private_data[*].id
}
```

---

# 47. Security Group Module

Separate security groups by responsibility:

```text
ALB
EKS nodes
Application
Database
Redis
Messaging
Endpoints
```

Avoid a single universal SG.

---

# 48. KMS Module

Use KMS for supported encryption requirements.

Examples:

```text
EBS
S3
Secrets Manager
CloudWatch Logs
RDS
EKS secrets encryption where configured
```

Centralized key policies must be carefully designed.

---

# 49. IAM Module

IAM module may manage:

```text
Terraform execution roles
EKS roles
Node roles
CI roles
Service roles
Cross-account roles
```

Apply least privilege.

---

# 50. IAM Anti-Pattern

Avoid:

```hcl
Action   = "*"
Resource = "*"
```

unless there is a documented, unavoidable requirement.

Terraform execution roles should be powerful enough for their lifecycle but not automatically equivalent to unrestricted administrator access.

---

# 51. ECR Module

Create repositories with:

```text
Image scanning
Lifecycle policy
Encryption
Repository policy
Tag mutability strategy
```

Example:

```text
roboshop/frontend
roboshop/catalogue
roboshop/user
roboshop/cart
```

---

# 52. ECR Lifecycle

Example policy concept:

```text
Keep production images longer
Remove unreferenced old images
Keep release tags
Expire temporary CI images
```

Do not delete images still required for rollback.

---

# 53. EKS Module

EKS module should define:

```text
Cluster
IAM
Networking
Endpoint configuration
Encryption
Logging
Add-ons
Node groups
```

Keep cluster configuration separate from application deployment.

---

# 54. EKS Dependencies

```text
VPC
 |
Private subnets
 |
IAM
 |
KMS
 |
EKS
 |
Node groups
 |
Add-ons
```

Terraform dependency graph should represent actual dependencies.

---

# 55. EKS Endpoint

Production configuration should deliberately choose:

```text
Private
Public restricted
Public + private
```

Do not use the default without evaluating access requirements.

---

# 56. EKS Encryption

Configure encryption for supported Kubernetes secret storage using KMS.

Concept:

```text
Kubernetes Secret
      |
EKS encryption
      |
KMS
```

Application-level secret management remains important.

---

# 57. EKS Add-ons

Typical add-ons include:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI
```

Versions should be compatible with the selected EKS/Kubernetes version.

---

# 58. Node Groups

Separate node groups can provide:

```text
System
General application
Memory optimized
Compute optimized
Special workloads
```

Example:

```text
system-ng
application-ng
```

Do not create node groups without workload or operational justification.

---

# 59. EKS Autoscaling

Terraform can provision the infrastructure required for:

```text
Cluster Autoscaler
Karpenter
HPA
```

The actual application autoscaling logic should remain aligned with the Kubernetes platform architecture.

---

# 60. Terraform and Helm

Terraform can install Helm releases, but be careful about ownership.

A clean boundary can be:

```text
Terraform:
Infrastructure

ArgoCD:
Kubernetes applications
```

This capstone uses GitOps for application delivery.

---

# 61. Terraform and ArgoCD Boundary

Recommended:

```text
Terraform
 |
+-- VPC
+-- EKS
+-- IAM
+-- ECR
+-- AWS infrastructure

ArgoCD
 |
+-- Namespaces
+-- Deployments
+-- Services
+-- Ingress
+-- ConfigMaps
+-- Application manifests
```

Avoid having Terraform and ArgoCD continuously manage the same Kubernetes objects.

---

# 62. Terraform CI Pipeline

Recommended stages:

```text
format
validate
lint
security scan
plan
approval
apply
```

Example:

```text
terraform fmt
terraform validate
tflint
tfsec/checkov
terraform plan
terraform apply
```

Tool choices can evolve.

---

# 63. Terraform Format

Run:

```bash
terraform fmt -check -recursive
```

Developers can format:

```bash
terraform fmt -recursive
```

Formatting should be automated.

---

# 64. Terraform Validate

Run:

```bash
terraform init
terraform validate
```

This catches configuration and syntax problems.

Validation does not replace a plan.

---

# 65. Terraform Linting

A linter can detect:

```text
Deprecated arguments
Potential mistakes
Style issues
Provider-specific problems
```

Example:

```bash
tflint
```

---

# 66. Terraform Security Scanning

Tools can inspect:

```text
IAM
S3
Security Groups
Encryption
Public exposure
Logging
```

Examples include:

```text
Checkov
tfsec-compatible tooling
Trivy IaC scanning
```

Use one or more according to organizational standards.

---

# 67. Policy as Code

Enterprise Terraform may use policy controls.

Conceptually:

```text
Terraform Plan
      |
Policy Engine
      |
+-----+------+
|            |
Pass        Fail
|            |
Apply       Stop
```

Policies can prevent:

```text
Public databases
Unencrypted storage
0.0.0.0/0 sensitive ports
Unapproved regions
Missing tags
```

---

# 68. Terraform Plan Artifact

CI should publish:

```text
plan output
JSON plan where needed
scan results
logs
```

This supports:

```text
Audit
Review
Troubleshooting
Approval
```

---

# 69. Production Approval

Example:

```text
Developer PR
     |
Plan
     |
Security checks
     |
Reviewer
     |
Production approval
     |
Apply
```

Production infrastructure changes should require appropriate authorization.

---

# 70. Branch Protection

Production Terraform repository should use:

```text
Protected main branch
Required reviews
Required CI
No direct pushes
Status checks
```

This is infrastructure governance.

---

# 71. Terraform State Permissions

Only the Terraform execution identity should normally have write access to production state.

Human engineers may receive:

```text
Read access
Troubleshooting access
```

based on role.

---

# 72. Break-Glass Terraform Access

Emergency access should be:

```text
Restricted
Audited
Time-limited
Approved
Documented
```

Do not make emergency access the normal workflow.

---

# 73. Terraform Lock Timeout

In environments with occasional contention:

```bash
terraform apply -lock-timeout=10m
```

Use only with the backend's supported locking behavior.

Do not hide process problems with excessive lock timeouts.

---

# 74. Terraform Parallelism

Terraform can create resources concurrently.

Default behavior is usually sufficient.

Changing:

```bash
-parallelism
```

should be based on a real provider/API throttling or dependency problem.

Do not randomly lower it.

---

# 75. AWS API Throttling

Large plans can trigger throttling.

Mitigation:

```text
Correct dependency graph
Reasonable parallelism
Retry behavior
Module decomposition
Provider configuration
```

Do not solve throttling by blindly slowing every pipeline.

---

# 76. Terraform Refresh

Terraform reconciles state with real infrastructure during planning operations.

If drift is detected:

```text
State != AWS
```

the plan reflects the difference.

Understand whether the difference is:

```text
Expected
Drift
Provider behavior
Ignored attribute
```

---

# 77. Lifecycle Ignore Changes

Example:

```hcl
lifecycle {
  ignore_changes = [
    tags["SomeExternalController"]
  ]
}
```

Use `ignore_changes` sparingly.

It can hide legitimate drift.

---

# 78. Create Before Destroy

For resources supporting it:

```hcl
lifecycle {
  create_before_destroy = true
}
```

This can reduce downtime.

However, some resources cannot be duplicated because of:

```text
Unique names
Quotas
IP constraints
External dependencies
```

---

# 79. Prevent Destroy

For critical resources:

```hcl
lifecycle {
  prevent_destroy = true
}
```

Use carefully.

It can protect:

```text
Production database
Critical KMS key
```

but can also block legitimate infrastructure evolution.

---

# 80. Terraform Dependency Graph

Terraform automatically builds a graph.

Example:

```text
VPC
 |
Subnets
 |
EKS
 |
Node Groups
```

Explicit `depends_on` should be used only when Terraform cannot infer the dependency.

---

# 81. Avoid Excessive depends_on

Bad:

```hcl
depends_on = [module.everything]
```

This destroys parallelism and obscures real dependencies.

Prefer precise dependencies.

---

# 82. Data Sources

Use data sources to query existing infrastructure.

Example:

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
```

Do not use data sources to hide important infrastructure ownership.

---

# 83. Data vs Resource

Resource:

```text
Terraform creates/manages it.
```

Data source:

```text
Terraform reads it.
```

Example:

```text
aws_vpc        = resource
aws_ami        = data source
```

---

# 84. Remote Data Dependencies

Avoid uncontrolled dependency on mutable external values.

For example:

```text
latest AMI
```

can change unexpectedly.

Production environments should pin versions or use a controlled image selection strategy.

---

# 85. AMI Strategy

For worker nodes:

```text
Approved image
+
Known Kubernetes compatibility
+
Security patching
```

Do not let production nodes silently change because a "latest" image changed.

---

# 86. Terraform Upgrades

Upgrade in stages:

```text
Terraform version
   |
Provider version
   |
Module versions
   |
AWS resources
```

Do not upgrade everything simultaneously without testing.

---

# 87. Upgrade Workflow

```text
Create branch
   |
Update versions
   |
terraform init -upgrade
   |
validate
   |
plan
   |
Review
   |
Test in dev
   |
Staging
   |
Production
```

---

# 88. Provider Upgrade Risk

Provider upgrades may change:

```text
Defaults
Validation
Resource behavior
Deprecated arguments
State representation
```

Always inspect the plan.

---

# 89. Module Versioning

Internal modules should be versioned.

Example:

```text
vpc-module v1.4.0
vpc-module v1.5.0
```

Consumers should upgrade deliberately.

---

# 90. Semantic Versioning

Use:

```text
MAJOR
MINOR
PATCH
```

Conceptually:

```text
1.4.2
```

Major changes may require migration.

---

# 91. Terraform Testing

Testing layers:

```text
Formatting
Validation
Linting
Security
Plan
Unit-like module tests
Integration tests
Deployment tests
```

Production infrastructure needs more than syntax validation.

---

# 92. Terraform Test

Modern Terraform supports configuration testing.

Conceptually:

```text
terraform test
```

Use it for reusable modules where practical.

---

# 93. Terratest

Some organizations use Go-based integration testing.

Example:

```text
Terraform
   |
Provision test infrastructure
   |
Run assertions
   |
Destroy
```

Useful for critical reusable modules.

---

# 94. VPC Integration Tests

Validate:

```text
VPC exists
Subnets span AZs
Routes correct
NAT reachable
Endpoints reachable
Flow Logs configured
Security Groups expected
```

---

# 95. EKS Integration Tests

Validate:

```text
Cluster active
Nodes Ready
CoreDNS healthy
VPC CNI healthy
EBS CSI healthy
API reachable
Pod scheduling works
Load Balancer integration works
```

---

# 96. Terraform Plan Failure

If plan fails:

```text
Read first error
Identify provider/resource
Check authentication
Check dependency
Check variables
Check state
Check AWS API
```

Do not immediately delete `.terraform` or state.

---

# 97. Authentication Failure

Typical message:

```text
AccessDenied
InvalidClientTokenId
ExpiredToken
```

Check:

```text
OIDC
STS
Role trust
Permissions
Account ID
Region
CI variables
```

---

# 98. Region Mistake

Example:

```text
Provider:
us-east-1

Resource:
Expected:
ap-south-1
```

Make region explicit and consistent.

For global services, understand regional behavior separately.

---

# 99. Account Mistake

A dangerous failure is applying to the wrong AWS account.

CI should validate:

```text
Expected account ID
Expected environment
Expected role
```

Example conceptual guard:

```text
Terraform production
     |
STS caller identity
     |
Expected production account?
     |
No -> Stop
Yes -> Continue
```

---

# 100. Production Account Guard

Use a data source:

```hcl
data "aws_caller_identity" "current" {}
```

Then validate the expected account through configuration or policy.

This is an important production safety mechanism.

---

# 101. Workspace Guard

If workspaces are used, validate:

```text
workspace = production
```

for production-specific configurations.

But separate root/state architecture remains preferable for stronger isolation.

---

# 102. Destructive Plan Guard

CI should fail or require explicit approval when:

```text
Database replacement
KMS key destruction
VPC replacement
EKS replacement
Production subnet destruction
```

appears in a plan.

---

# 103. Terraform Logging

Useful environment variable:

```bash
TF_LOG=INFO
```

For deep troubleshooting:

```bash
TF_LOG=DEBUG
```

Do not expose sensitive values in CI logs.

Disable verbose logging after troubleshooting.

---

# 104. Terraform Debugging

Useful:

```bash
terraform providers
terraform graph
terraform state list
terraform state show
terraform show
terraform plan
```

Use these to understand the dependency and state model.

---

# 105. Terraform Graph

Generate graph:

```bash
terraform graph
```

It helps visualize:

```text
Resource dependencies
Module relationships
```

For large infrastructures, simplify or render the graph externally when necessary.

---

# 106. Terraform Output

Outputs should expose operationally useful values:

```text
VPC ID
Subnet IDs
EKS cluster name
EKS endpoint
ECR repository URLs
Load Balancer DNS
```

Do not output secrets.

---

# 107. Sensitive Outputs

If absolutely required:

```hcl
output "secret_value" {
  value     = var.secret
  sensitive = true
}
```

But prefer not exposing secrets through Terraform outputs at all.

---

# 108. Terraform Documentation

Every production module should document:

```text
Purpose
Inputs
Outputs
Dependencies
Examples
Security considerations
Upgrade notes
```

A README is part of infrastructure quality.

---

# 109. Example Module README

```text
# VPC Module

## Purpose
Creates production VPC networking.

## Inputs
vpc_cidr
availability_zones
public_subnet_cidrs
private_app_subnet_cidrs
private_data_subnet_cidrs

## Outputs
vpc_id
public_subnet_ids
private_app_subnet_ids
private_data_subnet_ids
```

---

# 110. CI Terraform Directory

Example:

```text
.gitlab-ci.yml
```

Concept:

```yaml
terraform_validate:
  stage: validate
  script:
    - terraform init
    - terraform fmt -check
    - terraform validate
```

Production CI should add security scanning and plan/apply controls.

---

# 111. Terraform Plan Job

Concept:

```yaml
terraform_plan:
  stage: plan
  script:
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan
```

The real pipeline should include environment-specific state and credentials.

---

# 112. Terraform Apply Job

Concept:

```yaml
terraform_apply:
  stage: deploy
  script:
    - terraform apply tfplan
  when: manual
```

Production apply should be protected by:

```text
Protected environment
Approved branch
Appropriate role
Required approvals
```

---

# 113. CI Environment Variables

Examples:

```text
AWS_REGION
AWS_ROLE_ARN
TF_VAR_environment
TF_VAR_project
```

Do not store static AWS access keys if OIDC is available.

---

# 114. Terraform Security in CI

Ensure:

```text
OIDC
Protected branches
Protected variables
Masked secrets
Least privilege
Artifact protection
Restricted runners
```

CI is part of the infrastructure security boundary.

---

# 115. Terraform Runner

Use a trusted runner with:

```text
Terraform installed
AWS CLI if needed
Security scanners
Network access to AWS
No persistent credentials
```

Ephemeral runners are preferred where practical.

---

# 116. Runner Network

The runner may be:

```text
GitLab-hosted
Self-hosted
Private
```

If self-hosted in AWS:

```text
Private subnet
NAT / VPC endpoints
Restricted IAM
```

The runner does not need public exposure merely to deploy Terraform.

---

# 117. Terraform and Private EKS

If Terraform needs to interact with a private EKS API:

```text
CI Runner
   |
VPN / VPC connectivity
   |
Private EKS Endpoint
```

This creates an important dependency.

Plan CI network connectivity before making the cluster API private-only.

---

# 118. Terraform and Public EKS

If the endpoint is public but restricted:

```text
CI Runner Public IP
       |
EKS API allowlist
```

This can simplify access but increases the importance of IP stability and allowlist management.

---

# 119. Infrastructure Separation

Recommended lifecycle separation:

```text
Network Terraform
       |
EKS Terraform
       |
Shared AWS Services
       |
GitOps
```

Avoid a single state file controlling every resource in the company.

---

# 120. Blast Radius

If one state contains:

```text
VPC
EKS
RDS
IAM
ECR
```

a mistake can affect everything.

Separating state by lifecycle can reduce blast radius.

---

# 121. State Layer Example

```text
production/network
production/security
production/eks
production/data
production/shared
```

The exact decomposition should balance:

```text
Blast radius
Dependency complexity
Operational overhead
```

---

# 122. Dependency Between States

If EKS needs VPC outputs:

```text
Network State
     |
Remote State / explicit interface
     |
EKS State
```

Use controlled outputs.

Avoid hidden dependencies.

---

# 123. Remote State Data

Terraform can consume outputs from another state.

Conceptually:

```hcl
data "terraform_remote_state" "network" {
  ...
}
```

Use carefully because it creates coupling between state layers.

---

# 124. Better Interface Principle

Network state should expose:

```text
vpc_id
private subnet IDs
public subnet IDs
security group IDs
```

EKS state consumes these.

The interface should remain stable.

---

# 125. Terraform Workspace Locking

Never allow:

```text
Two production applies
```

at the same time.

Use:

```text
Backend locking
CI environment protection
Pipeline concurrency controls
```

---

# 126. Pipeline Concurrency

A production pipeline should prevent:

```text
Pipeline A -> Apply
Pipeline B -> Apply
```

simultaneously.

Prefer:

```text
Pipeline A -> Apply
Pipeline B -> Wait
```

---

# 127. Plan Staleness

A plan generated hours ago may no longer represent current infrastructure.

Risk:

```text
Plan generated
      |
External change
      |
Apply old plan
```

Keep plan-to-apply windows controlled.

Re-plan when material changes occur.

---

# 128. Terraform Apply Failure

Example:

```text
20 resources planned
10 created
11th failed
```

Terraform records state for successful operations.

Do not assume everything rolled back.

Terraform is not a database transaction.

---

# 129. Partial Apply Recovery

After failure:

```text
Read error
   |
Inspect state
   |
Inspect AWS
   |
Fix root cause
   |
Run plan
   |
Apply
```

Do not manually recreate resources without understanding state.

---

# 130. Resource Replacement

Terraform may show:

```text
-/+ replace
```

This can cause downtime.

Check:

```text
Why replacement?
Can lifecycle prevent downtime?
Can blue/green architecture help?
Is replacement acceptable?
```

---

# 131. Database Protection

Production database resources should have:

```text
Deletion protection
Backup
Encryption
prevent_destroy where justified
```

Terraform is not the only protection layer.

---

# 132. KMS Key Protection

KMS keys are critical.

Use:

```text
Key policies
Deletion windows
Restricted access
Audit logging
```

Avoid accidental key destruction.

---

# 133. S3 Terraform State Protection

State bucket:

```text
Block Public Access
Versioning
Encryption
Restricted bucket policy
CloudTrail
```

Consider object-level protections appropriate to organizational policy.

---

# 134. Terraform Supply Chain

Pin:

```text
Terraform version
Provider versions
Module versions
Provider checksums
```

Review third-party modules before adoption.

---

# 135. Third-Party Modules

Before using a public module:

```text
Review source
Review maintainers
Review releases
Review permissions
Review resources
Review IAM
Review security history
```

Do not blindly copy infrastructure code from the Internet.

---

# 136. Internal Modules

For the capstone, critical modules should be controlled internally:

```text
vpc
iam
eks
security
ecr
logging
```

This gives the team ownership over architecture.

---

# 137. Terraform Registry

Public registry modules can accelerate implementation.

But production teams should pin versions and review module behavior.

Convenience should not replace architecture review.

---

# 138. Policy: Public Resources

A policy can reject:

```text
Public RDS
Public Redis
Public Kafka
Public EKS nodes
```

unless explicitly approved.

---

# 139. Policy: Required Tags

Reject resources missing:

```text
Environment
Project
Owner
ManagedBy
CostCenter
```

This improves governance.

---

# 140. Policy: Encryption

Require encryption for:

```text
S3
EBS
RDS
Secrets
Logs
```

where supported and required.

---

# 141. Policy: Approved Regions

Production Terraform should use approved AWS regions.

Example:

```text
Allowed:
ap-south-1

Denied:
unapproved regions
```

Organization SCPs should also enforce this boundary.

---

# 142. Policy: Sensitive Ports

Reject Internet exposure for:

```text
22
3306
5432
6379
5672
9092
```

unless there is a documented exception.

---

# 143. Terraform and AWS Organizations

Terraform can manage organization-level resources, but organization governance should be separated carefully from workload infrastructure.

Example:

```text
Organization Terraform
      |
OUs
Accounts
SCPs
```

while:

```text
Production Terraform
      |
VPC
EKS
ECR
```

This reduces accidental organization-wide impact.

---

# 144. Account Bootstrap

New account process:

```text
Create account
   |
Baseline IAM
   |
CloudTrail
   |
GuardDuty
   |
Config
   |
Security controls
   |
Terraform execution role
   |
Networking
```

Account bootstrap can itself be automated.

---

# 145. Terraform Bootstrap Problem

Terraform needs a backend and credentials before it can create the backend and credentials.

This creates a bootstrap layer.

Example:

```text
Bootstrap
   |
State bucket
IAM roles
OIDC trust
Logging
   |
Main Terraform
```

---

# 146. Bootstrap State

Bootstrap may use:

```text
Dedicated bootstrap account
```

or a tightly controlled initial process.

After bootstrap, normal infrastructure should be managed through standard pipelines.

---

# 147. Backend Bootstrap

Do not have every engineer manually create production state buckets.

Automate or centrally manage:

```text
State bucket
Encryption
Versioning
Access
Logging
```

---

# 148. Terraform Security Boundary

The Terraform execution role can potentially modify:

```text
Network
IAM
EKS
Storage
Security
```

Therefore it is a high-value identity.

Protect it aggressively.

---

# 149. Terraform Role Permissions

Prefer separate roles where practical:

```text
Network Terraform Role
EKS Terraform Role
Security Terraform Role
```

or a carefully controlled environment role.

The correct design depends on lifecycle and organizational scale.

---

# 150. Terraform Role Trust

Trust policy should restrict:

```text
OIDC issuer
Repository/project
Branch/tag
Audience
Environment
```

Do not trust every repository.

---

# 151. GitLab OIDC Trust Concept

```text
GitLab project
      |
Protected branch
      |
OIDC token
      |
AWS trust policy
      |
Production role
```

This prevents unrelated pipelines from assuming the production role.

---

# 152. Terraform Repository Structure

Full example:

```text
21-DevOps-Production-Capstone/
|
+-- terraform/
    |
    +-- modules/
    |   +-- vpc/
    |   +-- iam/
    |   +-- kms/
    |   +-- endpoints/
    |   +-- eks/
    |   +-- ecr/
    |   +-- security-groups/
    |
    +-- environments/
        +-- dev/
        +-- staging/
        +-- production/
        +-- dr/
```

---

# 153. Production VPC Root Example

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = "roboshop-production"

  vpc_cidr = "10.0.0.0/16"

  availability_zones = [
    "ap-south-1a",
    "ap-south-1b",
    "ap-south-1c"
  ]

  public_subnet_cidrs = [
    "10.0.0.0/20",
    "10.0.16.0/20",
    "10.0.32.0/20"
  ]

  private_app_subnet_cidrs = [
    "10.0.64.0/20",
    "10.0.80.0/20",
    "10.0.96.0/20"
  ]

  private_data_subnet_cidrs = [
    "10.0.128.0/20",
    "10.0.144.0/20",
    "10.0.160.0/20"
  ]
}
```

---

# 154. EKS Root Example

```hcl
module "eks" {
  source = "../../modules/eks"

  cluster_name = "roboshop-production"

  vpc_id = module.vpc.vpc_id

  private_subnet_ids =
    module.vpc.private_app_subnet_ids
}
```

---

# 155. ECR Root Example

```hcl
module "ecr" {
  source = "../../modules/ecr"

  repositories = [
    "frontend",
    "catalogue",
    "user",
    "cart",
    "shipping",
    "payment"
  ]
}
```

---

# 156. Security Groups

Example architecture:

```text
ALB SG
  |
App SG
  |
DB SG
```

Terraform should model these relationships explicitly.

---

# 157. Terraform Graph for Capstone

```text
Account
  |
VPC
  |
+-- Subnets
+-- Routes
+-- NAT
+-- Endpoints
+-- Flow Logs
  |
Security
  |
+-- IAM
+-- KMS
+-- SG
  |
EKS
  |
+-- Cluster
+-- Add-ons
+-- Node Groups
  |
ECR
  |
GitLab CI
  |
ArgoCD
```

---

# 158. Terraform + GitOps Complete Flow

```text
Developer
   |
GitLab
   |
+---------------------+
| Terraform Repository|
+---------------------+
   |
CI
   |
AWS Infrastructure
   |
EKS
   |
ArgoCD
   |
Application Repository
   |
Kubernetes
```

Terraform builds the platform.

GitOps deploys applications.

---

# 159. Terraform Responsibilities

Terraform owns:

```text
AWS Accounts where applicable
VPC
IAM
KMS
EKS
ECR
Networking
Load Balancer infrastructure
AWS logging
AWS security services
```

---

# 160. ArgoCD Responsibilities

ArgoCD owns:

```text
Deployments
Services
ConfigMaps
Ingress
HPA
NetworkPolicies
Application configuration
```

The ownership boundary should be documented.

---

# 161. Manual Change Policy

Production manual changes should be:

```text
Exception only
Approved
Audited
Documented
Reconciled back into Terraform
```

If a manual emergency change remains permanently, update the source of truth.

---

# 162. Terraform Drift Workflow

```text
Drift detected
      |
Investigate
      |
Was it intentional?
   /       \
 Yes        No
 |           |
Update      Revert
Terraform   drift
```

Do not blindly apply Terraform.

---

# 163. Emergency Change

If production is actively failing:

```text
Incident
   |
Emergency mitigation
   |
Restore service
   |
Document change
   |
Update Terraform
   |
Plan
   |
Apply
```

The emergency action should not become permanent unmanaged infrastructure.

---

# 164. Terraform Disaster Recovery

Protect:

```text
Terraform code
State
Provider lock file
Module versions
CI configuration
Documentation
```

Code should exist in Git.

State should have versioning/backups.

---

# 165. State Backup

State bucket versioning allows recovery from accidental changes.

Example:

```text
Version 10
Version 11
Version 12
```

If Version 12 is corrupted:

```text
Review Version 11
```

Recovery should follow a documented runbook.

---

# 166. Terraform DR

A DR account can have:

```text
Separate state
Separate provider configuration
Separate VPC
Separate EKS
Separate resources
```

Do not accidentally point DR Terraform at production state.

---

# 167. Production State Isolation

Strong rule:

```text
Production state
NEVER shared with Dev
```

and:

```text
Production CI role
NEVER uses Dev state
```

---

# 168. Terraform Credential Isolation

Each environment should have a distinct role:

```text
terraform-dev-role
terraform-staging-role
terraform-production-role
terraform-dr-role
```

This creates a strong IAM boundary.

---

# 169. Terraform Environment Variables

CI can select:

```text
AWS_ROLE_ARN
TF_STATE_KEY
AWS_REGION
```

based on protected environment configuration.

---

# 170. Protected Production Environment

GitLab production environment should require:

```text
Protected branch
Approved deployment
Authorized maintainers
```

The exact control set depends on organizational policy.

---

# 171. Terraform Pipeline Security

Pipeline should prevent:

```text
Fork -> Production role
Untrusted branch -> Production
Merge request -> Production apply
Developer -> Direct apply
```

Production credentials should be available only in protected contexts.

---

# 172. Terraform Secret Leakage

Never print:

```text
AWS credentials
Database passwords
Private keys
Secret values
OIDC tokens
```

Be careful with:

```bash
set -x
TF_LOG=DEBUG
terraform show
```

Logs can become sensitive.

---

# 173. Terraform Plan Security

Plan files may contain sensitive information.

Treat:

```text
tfplan
```

as a protected CI artifact.

Restrict access and retention.

---

# 174. Terraform State Encryption

At rest:

```text
S3 encryption
```

In transit:

```text
TLS
```

Access:

```text
IAM
Bucket policy
```

Audit:

```text
CloudTrail
```

---

# 175. Terraform Naming and Cost

Consistent tags allow:

```text
AWS Cost Explorer
Cost allocation
Budgets
Chargeback
```

Terraform should apply organizational tags automatically where possible.

---

# 176. Terraform Cost Controls

Infrastructure code can enforce:

```text
Approved instance types
Maximum node counts
Required autoscaling
No accidental public resources
```

Policy as code can stop expensive mistakes before apply.

---

# 177. Example Cost Guard

Concept:

```text
Production node group:
min = 3
max = 20

Policy:
max <= approved limit
```

A plan attempting:

```text
max = 100
```

should require review or be rejected.

---

# 178. Quota Awareness

Terraform can fail because of AWS quotas.

Examples:

```text
VPCs
EIPs
NAT Gateways
ENIs
Security Groups
Load Balancers
EKS nodes
```

Production planning should include quota review.

---

# 179. Terraform and AWS Service Quotas

Before scaling:

```text
Current quota
   |
Expected capacity
   |
Headroom
```

Request increases before production expansion when necessary.

---

# 180. Terraform Timeout

Some AWS resources take significant time.

Terraform resources support resource-specific timeouts where available.

Do not increase every timeout globally without understanding the underlying issue.

---

# 181. Eventual Consistency

AWS APIs can be eventually consistent.

Terraform/provider implementations normally handle many cases.

When custom dependencies are required:

```text
Precise dependency
Correct resource ordering
Appropriate retries
```

Avoid arbitrary sleep commands.

---

# 182. Null Resources

Avoid using `null_resource` as a default mechanism for infrastructure.

Bad pattern:

```text
terraform
   |
local-exec
   |
aws cli
   |
manual resource creation
```

Prefer native Terraform resources.

---

# 183. Local Exec

Use `local-exec` only when necessary.

Risks:

```text
Environment dependency
Idempotency problems
Security concerns
CI differences
```

Native resources are usually better.

---

# 184. Terraform Provisioners

Provisioners should be a last resort.

Prefer:

```text
AMI
User Data
AWS resources
Cloud-init
SSM
Configuration management
```

instead of imperative provisioning inside Terraform.

---

# 185. Immutable Infrastructure

Prefer:

```text
New image
New node group
Controlled rollout
```

over:

```text
SSH into production
Change server manually
```

Terraform should define infrastructure state, not become a remote shell wrapper.

---

# 186. EKS Node Replacement

Example:

```text
Old node group
       |
New node group
       |
Workloads migrate
       |
Old group removed
```

This supports safer upgrades.

---

# 187. Terraform EKS Upgrade

Typical sequence:

```text
Review EKS version compatibility
   |
Upgrade cluster
   |
Upgrade add-ons
   |
Upgrade node groups
   |
Validate workloads
```

Exact sequencing depends on AWS-supported version paths.

---

# 188. Terraform and ALB

Terraform can provision:

```text
ALB
Target groups
Listeners
Security Groups
```

Kubernetes AWS Load Balancer Controller can also dynamically create ALBs.

Avoid duplicate ownership.

For the capstone:

```text
Terraform -> platform prerequisites
Kubernetes Controller -> application-driven ALB resources
```

where appropriate.

---

# 189. Terraform and Route 53

Terraform can manage:

```text
Hosted zones
DNS records
Alias records
```

Example:

```text
app.example.com
    |
ALB
```

Application DNS ownership should be documented.

---

# 190. Terraform and ACM

Terraform can manage:

```text
ACM certificates
DNS validation records
```

Certificate validation should be automated where possible.

---

# 191. Terraform and CloudWatch

Terraform can configure:

```text
Log groups
Retention
Alarms
Dashboards
```

Application metrics and Kubernetes monitoring can remain in the observability stack.

---

# 192. Terraform and CloudTrail

Terraform can manage or integrate with:

```text
Trail
S3 destination
CloudWatch integration
KMS
```

Organization-wide audit architecture should remain centrally governed.

---

# 193. Terraform and GuardDuty

Security services may be managed centrally.

Avoid enabling security controls independently in every workload state if centralized organization governance is intended.

---

# 194. Terraform and Config

AWS Config can enforce or report:

```text
Required tags
Public exposure
Encryption
Configuration drift
```

Use it alongside Terraform, not instead of Terraform.

---

# 195. Terraform vs AWS Config

Terraform:

```text
Desired infrastructure
```

AWS Config:

```text
Observed/compliance state
```

They complement each other.

---

# 196. Terraform vs CloudFormation

Terraform provides:

```text
Multi-provider capability
Module ecosystem
State model
Common workflow
```

CloudFormation provides deep AWS-native integration.

For this capstone:

```text
Terraform = primary IaC
```

---

# 197. Terraform vs Ansible

Terraform:

```text
Infrastructure provisioning
```

Ansible:

```text
Configuration / automation
```

Modern cloud-native environments should minimize mutable server configuration.

---

# 198. Terraform vs Helm

Terraform:

```text
AWS infrastructure
```

Helm:

```text
Kubernetes package management
```

ArgoCD:

```text
GitOps reconciliation
```

Each tool should have a clear boundary.

---

# 199. Production Terraform Architecture

```text
                    GitLab
                       |
                Pull Request
                       |
              Terraform CI
                       |
        +--------------+--------------+
        |              |              |
      Format         Security        Plan
        |              |              |
        +--------------+--------------+
                       |
                   Approval
                       |
                 Terraform Apply
                       |
              Production Role
                       |
                 AWS Account
                       |
        +--------------+--------------+
        |              |              |
       VPC            EKS            ECR
        |              |              |
      Network        Platform      Images
                       |
                    ArgoCD
                       |
                  Applications
```

---

# 200. End-to-End Production Workflow

```text
1. Engineer changes Terraform.
2. Commit to feature branch.
3. Open merge request.
4. CI runs formatting.
5. CI validates configuration.
6. CI runs linting.
7. CI runs security scanning.
8. CI generates plan.
9. Reviewer checks plan.
10. Required approvals complete.
11. Merge to protected branch.
12. Protected production pipeline starts.
13. CI assumes production role using OIDC.
14. Terraform acquires state lock.
15. Approved plan is applied.
16. Terraform updates state.
17. CI records result.
18. Monitoring validates infrastructure.
19. Drift and health checks continue.
```

---

# 201. Production Terraform Checklist

```text
[ ] Terraform version pinned
[ ] Provider versions pinned
[ ] .terraform.lock.hcl committed
[ ] Remote state configured
[ ] State encrypted
[ ] State versioning enabled
[ ] State access restricted
[ ] Environment state isolated
[ ] OIDC authentication
[ ] No long-lived AWS keys
[ ] Production role protected
[ ] Branch protection enabled
[ ] Plan required
[ ] Apply approval required
[ ] Security scan enabled
[ ] Policy checks enabled
[ ] Account guard enabled
[ ] Region guard enabled
[ ] Destructive changes reviewed
[ ] Terraform modules documented
```

---

# 202. Terraform Troubleshooting Matrix

| Problem | First Checks |
|---|---|
| AccessDenied | IAM, trust policy, account |
| Wrong account | STS caller identity |
| Wrong region | Provider configuration |
| State lock | Existing pipeline/process |
| Drift | AWS changes, plan |
| Resource replacement | Immutable attributes |
| API throttling | Parallelism, quotas |
| EKS failure | IAM, subnet, SG, version |
| VPC failure | CIDR, routes, dependencies |
| Import mismatch | Resource config vs actual state |
| Apply partial failure | State + AWS |
| Provider error | Provider version/docs |
| CI failure | OIDC, runner, variables |
| Plan unexpectedly destroys | State, addresses, lifecycle |

---

# 203. Senior Interview Question — Why Terraform?

Strong answer:

```text
I use Terraform because I want infrastructure to be version controlled,
reviewable, repeatable, and consistently deployed across environments.
For production I combine Terraform with remote state, locking, OIDC-based
AWS authentication, protected branches, plan review, policy checks, and
separate state boundaries to reduce blast radius.
```

---

# 204. Interview — Why Remote State?

```text
Remote state allows the team and CI system to share a single controlled
state representation. It also provides centralized security, locking,
versioning, and recovery capabilities. Production state is protected
because it can contain sensitive infrastructure information.
```

---

# 205. Interview — How Do You Protect Terraform State?

```text
I use a private encrypted backend with versioning, restricted IAM,
audit logging, and environment-specific state. I treat state as
sensitive infrastructure data and avoid giving broad write access.
```

---

# 206. Interview — How Do You Prevent Production Mistakes?

```text
I use separate production state and execution roles, OIDC-based
temporary credentials, protected branches, mandatory validation and
security checks, plan review, approval gates, account/region guards,
and controlled CI applies. Destructive changes receive additional
review.
```

---

# 207. Interview — What Happens If Apply Fails Halfway?

```text
Terraform is not transactional. Resources created before the failure
may remain and their state is normally recorded. I inspect the Terraform
state and AWS resources, identify the root cause, fix it, then run a new
plan before continuing. I never assume that a failed apply rolled
everything back.
```

---

# 208. Interview — How Do You Handle Drift?

```text
I first determine whether the change was intentional. If it was
intentional, I update Terraform. If it was unauthorized drift, I
reconcile it according to the incident or change process. I also use
restricted production access and AWS audit/compliance controls to
reduce uncontrolled changes.
```

---

# 209. Interview — Modules

```text
I use modules to standardize reusable infrastructure such as VPC,
security groups, EKS, IAM, and ECR. Modules expose controlled inputs
and outputs while environment-specific root configurations determine
how each environment is deployed. I avoid overly generic modules that
hide important behavior.
```

---

# 210. Interview — Workspaces

```text
Workspaces are useful for some state separation use cases, but I don't
treat them as a complete production isolation mechanism. For critical
environments I prefer separate root configurations and separate remote
state because the environment boundary is easier to understand and
protect.
```

---

# 211. Interview — Terraform and Kubernetes

```text
I keep infrastructure ownership and application ownership separate.
Terraform creates the AWS platform and EKS foundation. ArgoCD manages
Kubernetes application manifests through GitOps. I avoid having
Terraform and ArgoCD continuously manage the same Kubernetes objects.
```

---

# 212. Interview — OIDC

```text
I prefer OIDC federation from GitLab to AWS STS because it provides
short-lived credentials instead of storing long-lived AWS access keys.
The trust policy is restricted to the approved project, branch or
environment, and the role has only the permissions required for that
infrastructure lifecycle.
```

---

# 213. Interview — State Separation

```text
I separate state according to lifecycle and blast radius. At minimum,
development, staging, production, and DR have separate state. For a
large production platform I may also separate networking, security,
EKS, and data layers when their independent lifecycle justifies the
additional operational complexity.
```

---

# 214. Interview — Terraform Security

```text
I secure Terraform at multiple layers: protected source control,
provider and module pinning, OIDC authentication, least-privilege
roles, encrypted remote state, policy-as-code, IaC security scanning,
restricted production applies, and audit logging.
```

---

# 215. Interview — Terraform Upgrade

```text
I upgrade Terraform and providers deliberately. I test the upgrade in
development, inspect the plan for behavioral or replacement changes,
validate compatibility, then promote through staging and production.
I avoid combining unrelated infrastructure changes with the upgrade.
```

---

# 216. Interview — Production Account Protection

```text
Before applying production infrastructure, CI verifies that it is
using the expected AWS account and role. The production role is
available only to protected CI contexts, and the production state is
separate from other environments. This prevents a configuration mistake
from accidentally applying to the wrong account.
```

---

# 217. Interview — Terraform Blast Radius

```text
I reduce blast radius by separating environments and, where useful,
separating state by infrastructure lifecycle. I also use least
privilege, protected approvals, and policy checks so that a change to
one layer does not automatically put unrelated layers at risk.
```

---

# 218. Production Terraform Operating Model

```text
SOURCE OF TRUTH:
Git

STATE:
Remote protected backend

AUTH:
OIDC + STS

VALIDATION:
fmt + validate + lint

SECURITY:
IaC scan + policy

CHANGE:
Plan + review

PRODUCTION:
Protected apply

AUDIT:
Git + CI + CloudTrail

RECOVERY:
State versioning + Git history
```

---

# 219. Final Terraform Architecture

```text
                    Git Repository
                          |
                   Protected Branch
                          |
                     CI Pipeline
                          |
        +-----------------+-----------------+
        |                 |                 |
       Format           Security          Plan
        |                 |                 |
        +-----------------+-----------------+
                          |
                       Approval
                          |
                     OIDC / STS
                          |
                 Production IAM Role
                          |
                 +--------+--------+
                 |                 |
              State              AWS
                 |                 |
               S3       +---------+---------+
                        |         |         |
                       VPC       EKS       ECR
                        |         |
                     Network   Platform
                                  |
                                ArgoCD
                                  |
                             Applications
```

---

# 220. Final Principles

The production Terraform implementation should follow these rules:

```text
1. Everything important is version controlled.
2. Production state is remote and protected.
3. No long-lived AWS credentials in CI.
4. OIDC provides temporary access.
5. Production changes require review.
6. Plans are inspected before apply.
7. State is isolated by environment.
8. Modules are reusable but understandable.
9. Terraform and ArgoCD have clear ownership boundaries.
10. Drift is detected and controlled.
11. Destructive changes receive additional scrutiny.
12. Infrastructure is tested before production.
13. Provider and module versions are controlled.
14. Auditability is treated as a production requirement.
15. Terraform is the source of truth for AWS infrastructure.
```

---

# 221. Completion Criteria

The Terraform layer is considered production-ready when:

```text
[ ] AWS provider works through OIDC
[ ] Remote state is secured
[ ] State locking works
[ ] VPC deploys successfully
[ ] NAT strategy works
[ ] VPC endpoints work
[ ] Security groups are validated
[ ] Flow Logs are enabled
[ ] ECR repositories exist
[ ] EKS cluster is healthy
[ ] Node groups are healthy
[ ] KMS encryption is configured
[ ] CI plan succeeds
[ ] CI apply succeeds
[ ] Production approval works
[ ] Drift detection works
[ ] Rollback/recovery procedure is tested
[ ] Documentation is complete
```

---

# 222. Conclusion

Terraform is the infrastructure delivery foundation of this production capstone.

The mature implementation is not:

```text
terraform apply
```

It is:

```text
Code
  |
Review
  |
Validate
  |
Secure
  |
Plan
  |
Approve
  |
Apply
  |
Audit
  |
Monitor
  |
Recover
```

The combination of:

```text
Terraform
+
Remote State
+
OIDC
+
IAM
+
Policy as Code
+
CI/CD
+
AWS Governance
+
GitOps
```

creates a controlled production infrastructure lifecycle.

The next layer is the EKS platform itself.

That platform must consume the VPC, IAM, security, and Terraform foundation established here and turn it into a highly available production Kubernetes environment.
