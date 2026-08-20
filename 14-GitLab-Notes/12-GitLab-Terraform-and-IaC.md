# GitLab Terraform and IaC

> Production-oriented guide to running Terraform with GitLab CI/CD, including repository structure, modules, remote state, planning, approvals, OIDC authentication, AWS infrastructure, EKS, ECR, networking, security, drift, imports, state recovery, concurrency, policy checks, reusable pipelines, multi-environment architecture, troubleshooting, and senior DevOps interview scenarios.

---

## 1. Why Terraform with GitLab?

GitLab CI provides automation around Terraform:

```text
Developer
 ↓
GitLab MR
 ↓
Terraform CI
 ↓
Validate
 ↓
Plan
 ↓
Review
 ↓
Apply
 ↓
AWS
```

This creates a controlled Infrastructure as Code workflow.

---

## 2. Infrastructure as Code

IaC represents infrastructure declaratively.

Instead of manually creating:

```text
VPC
EKS
IAM
ECR
RDS
ALB
```

Terraform describes the desired infrastructure in code.

---

## 3. Terraform Desired State

Concept:

```text
Terraform Code
      ↓
Desired State
      ↓
Terraform Plan
      ↓
AWS
```

Terraform compares configuration with known/current infrastructure state.

---

## 4. GitLab as Terraform Workflow Platform

GitLab can provide:

```text
Repository
Merge Requests
CI/CD
Variables
Protected environments
Approvals
Artifacts
Runners
Audit trail
```

Terraform provides the infrastructure engine.

---

## 5. Typical Repository Structure

Example:

```text
terraform/
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── ecr/
│   ├── iam/
│   ├── rds/
│   └── alb/
├── environments/
│   ├── dev/
│   ├── stage/
│   └── prod/
└── README.md
```

Keep reusable modules separate from environment composition.

---

## 6. Root Module vs Child Module

Root module:

```text
Environment
 ↓
Calls modules
```

Child module:

```text
Reusable infrastructure component
```

Example:

```hcl
module "vpc" {
  source = "../../modules/vpc"
}
```

---

## 7. Terraform Module Benefits

Modules improve:

- reuse
- consistency
- maintainability
- standardization
- reviewability

Avoid creating modules so granular that simple infrastructure becomes difficult to understand.

---

## 8. Environment Separation

Possible structure:

```text
environments/
├── dev/
├── staging/
└── production/
```

Each environment can have separate:

```text
variables
backend
provider configuration
module versions
```

---

## 9. Separate AWS Accounts

Stronger production isolation:

```text
Dev Account
Stage Account
Prod Account
```

GitLab assumes different IAM roles.

This is preferable when organizational security requirements demand account isolation.

---

## 10. Workspace Strategy

Terraform workspaces can represent multiple environments.

Example:

```text
dev
stage
prod
```

However, workspaces are not a replacement for strong account/permission boundaries.

---

## 11. Directory vs Workspace

### Directories

```text
environments/dev
environments/prod
```

Pros:

- explicit
- easier review
- separate backends
- clearer production boundaries

### Workspaces

Pros:

- shared configuration
- convenient for similar environments

Choose based on architecture.

---

## 12. Terraform Backend

A backend stores Terraform state.

For AWS:

```text
Terraform
 ↓
S3 backend
 ↓
Terraform state
```

State is critical infrastructure metadata.

---

## 13. Remote State

Remote state enables:

```text
CI Runner A
CI Runner B
Developer
```

to work with the same infrastructure state.

Do not keep production state only on a developer laptop.

---

## 14. S3 Backend Security

Protect state with:

```text
private bucket
encryption
restricted IAM
versioning
audit logging
```

State may contain sensitive infrastructure information.

---

## 15. State Locking

Concurrent Terraform operations must be controlled.

Use the locking capabilities supported by the selected backend and Terraform version.

The goal is:

```text
Apply A
 ↓
State lock
 ↓
Apply B waits/fails
```

rather than both modifying state concurrently.

---

## 16. Terraform State

State maps:

```text
Terraform resource
 ↔
Real infrastructure
```

Example:

```text
aws_vpc.main
 ↔
vpc-012345
```

---

## 17. Never Edit State Manually

Avoid modifying the state file directly.

Use Terraform state commands:

```bash
terraform state list
terraform state show
terraform state mv
terraform state rm
```

for controlled state operations.

---

## 18. Terraform Init

First command in most CI jobs:

```bash
terraform init
```

It initializes:

```text
backend
provider plugins
modules
```

---

## 19. Terraform Format

Run:

```bash
terraform fmt -check -recursive
```

This enforces consistent formatting.

---

## 20. Terraform Validate

Run:

```bash
terraform validate
```

It checks Terraform configuration syntax and internal consistency.

---

## 21. Terraform Plan

Run:

```bash
terraform plan
```

Plan shows intended changes:

```text
+ create
~ update
- destroy
-/+ replace
```

---

## 22. Terraform Apply

Run:

```bash
terraform apply
```

Apply changes infrastructure.

In production, apply should normally be protected by approvals and environment controls.

---

## 23. CI Plan and Apply Separation

Recommended:

```text
plan job
 ↓
review
 ↓
protected apply job
```

Do not automatically apply production changes from arbitrary branches.

---

## 24. Terraform Plan Artifact

A plan can be saved:

```bash
terraform plan -out=tfplan
```

Then a later controlled job can use:

```bash
terraform apply tfplan
```

Ensure the plan artifact is protected and consumed in the correct environment/context.

---

## 25. Plan File Security

A Terraform plan may contain sensitive information.

Do not publish it broadly.

Use:

```text
protected artifacts
limited retention
restricted access
```

---

## 26. Plan from Merge Request

A strong workflow:

```text
MR
 ↓
fmt
 ↓
validate
 ↓
security scan
 ↓
plan
 ↓
MR review
```

This makes infrastructure changes visible before merge.

---

## 27. Production Apply

Example:

```text
main
 ↓
Plan
 ↓
Approval
 ↓
Protected Runner
 ↓
Production IAM role
 ↓
Apply
```

---

## 28. Protected GitLab Environment

Configure production Terraform apply as a protected environment.

This limits who can trigger/approve it.

---

## 29. Protected Branch

Production infrastructure should normally be deployed from:

```text
protected branch/tag
```

not arbitrary feature branches.

---

## 30. GitLab OIDC for Terraform

Preferred flow:

```text
GitLab job
 ↓
OIDC token
 ↓
AWS STS
 ↓
Terraform IAM role
 ↓
AWS
```

No long-lived AWS access key needs to be stored in CI.

---

## 31. Terraform AWS Provider

Example:

```hcl
provider "aws" {
  region = var.aws_region
}
```

Credentials should come from the AWS credential chain.

Do not hard-code secrets.

---

## 32. AWS Credential Chain

Terraform/AWS SDK can obtain credentials through supported mechanisms such as:

```text
environment
web identity
instance role
shared configuration
```

Use the mechanism appropriate to the execution environment.

---

## 33. Terraform IAM Role

The Terraform role may manage:

```text
VPC
EKS
ECR
IAM
RDS
S3
ALB
```

Scope permissions to the infrastructure actually managed.

---

## 34. Terraform IAM vs Application IAM

Separate:

```text
Terraform role
```

from:

```text
Application runtime role
```

Terraform needs provisioning permissions.

Application workloads should receive only runtime permissions.

---

## 35. Terraform Role Blast Radius

A highly privileged Terraform role can change large parts of AWS.

Therefore protect:

```text
role assumption
branch
environment
apply job
Runner
```

---

## 36. AWS Account Validation

Before production apply:

```bash
aws sts get-caller-identity
```

Verify the expected account and role.

This is a simple but powerful guardrail.

---

## 37. AWS Region Validation

Verify:

```text
AWS_REGION
```

before applying.

A correct Terraform configuration against the wrong region can create unexpected resources.

---

## 38. Terraform Variables

Example:

```hcl
variable "environment" {
  type = string
}
```

Use variables for environment-specific values.

---

## 39. Variable Validation

Example:

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "stage", "prod"], var.environment)
    error_message = "Invalid environment."
  }
}
```

Validation prevents incorrect input.

---

## 40. Sensitive Variables

Mark sensitive values:

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

But sensitive flags do not magically secure the value everywhere.

Prefer external secret management.

---

## 41. Terraform Outputs

Outputs expose useful information:

```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

Avoid outputting secrets unnecessarily.

---

## 42. Locals

Locals simplify repeated expressions.

Example:

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"
}
```

Use locals to improve readability.

---

## 43. Data Sources

Data sources read existing infrastructure.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

They are useful when infrastructure is shared or pre-existing.

---

## 44. Resource Dependencies

Terraform automatically builds dependency relationships from references.

Example:

```text
VPC
 ↓
Subnets
 ↓
EKS
```

Avoid unnecessary explicit `depends_on`.

---

## 45. Explicit `depends_on`

Use when Terraform cannot infer a dependency.

Example:

```hcl
depends_on = [module.iam]
```

Do not add it everywhere because it can make plans less efficient.

---

## 46. Terraform Provider Versions

Pin compatible provider versions.

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

Use the version range appropriate for the project.

---

## 47. Terraform Lock File

Commit:

```text
.terraform.lock.hcl
```

It records selected provider versions/checksums.

This improves reproducibility.

---

## 48. Provider Upgrade

Do not blindly run:

```bash
terraform init -upgrade
```

in production.

Upgrade providers deliberately:

```text
change
 ↓
plan
 ↓
test
 ↓
review
 ↓
production
```

---

## 49. Terraform Version Pinning

Use a controlled Terraform version.

Example:

```hcl
terraform {
  required_version = ">= 1.8, < 2.0"
}
```

Use the actual supported project version/range.

---

## 50. CI Terraform Image

Use a controlled CI image containing:

```text
Terraform
AWS CLI
security tools
utility tools
```

Pin the image version.

---

## 51. GitLab Terraform CI Stages

Recommended:

```yaml
stages:
  - validate
  - security
  - plan
  - apply
```

Separate stages make failures easier to identify.

---

## 52. Terraform Validate Job

Conceptual:

```yaml
terraform_validate:
  stage: validate
  script:
    - terraform fmt -check -recursive
    - terraform init
    - terraform validate
```

The exact working directory depends on repository structure.

---

## 53. Terraform Plan Job

Conceptual:

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

Protect the artifact.

---

## 54. Terraform Apply Job

Conceptual:

```yaml
terraform_apply:
  stage: apply
  when: manual
  script:
    - terraform init
    - terraform apply tfplan
```

Production should add protected environment and identity controls.

---

## 55. Plan/Apply Consistency

A saved plan should be applied in a controlled context matching:

```text
configuration
provider
backend
variables
credentials
environment
```

Do not blindly transfer a plan between unrelated environments.

---

## 56. Terraform State Per Environment

Example:

```text
s3://terraform-state/
 ├── dev/
 ├── stage/
 └── prod/
```

Use clear separation.

---

## 57. State Isolation

Never allow:

```text
Dev pipeline
 ↓
Production state
```

unless explicitly authorized.

Environment identity should be enforced at multiple layers.

---

## 58. Backend Configuration

Backend configuration can be supplied per environment.

Example concept:

```text
dev backend
stage backend
prod backend
```

This avoids accidental state overlap.

---

## 59. Terraform Backend Credentials

Do not put long-lived credentials directly into backend configuration.

Use the AWS credential chain/OIDC-compatible approach.

---

## 60. State Bucket IAM

Restrict:

```text
GetObject
PutObject
ListBucket
```

to the required state paths/resources.

Do not give every developer unrestricted access to all production state.

---

## 61. Terraform State Versioning

S3 versioning provides historical state versions.

If a bad state write occurs:

```text
Current state
 ↓
Previous version
 ↓
Recovery
```

Recovery should be tested.

---

## 62. Terraform State Recovery

Before recovery:

```text
Stop Terraform operations
 ↓
Identify correct state version
 ↓
Backup current state
 ↓
Restore carefully
 ↓
Run terraform plan
```

Never overwrite state casually during an incident.

---

## 63. Terraform Import

Import existing resources:

```bash
terraform import aws_instance.example i-012345
```

Import associates real infrastructure with Terraform state.

The configuration should then accurately represent the resource.

---

## 64. Import Workflow

```text
Existing AWS resource
 ↓
Terraform configuration
 ↓
terraform import
 ↓
terraform plan
 ↓
Fix configuration differences
```

Do not assume import automatically generates perfect configuration.

---

## 65. Terraform Import in Production

Prefer:

```text
controlled branch
review
backup
plan
approval
```

Import modifies state and can have operational consequences.

---

## 66. Terraform State Move

Use:

```bash
terraform state mv
```

when refactoring resource addresses.

Example:

```text
aws_instance.web
 ↓
module.web.aws_instance.this
```

This avoids unnecessary destroy/create operations.

---

## 67. Terraform State Remove

Use:

```bash
terraform state rm
```

when Terraform should stop managing a resource.

It does not delete the actual AWS resource.

---

## 68. Resource Replacement

Terraform may show:

```text
-/+ replace
```

This means resource recreation.

Review replacement carefully in production.

---

## 69. `create_before_destroy`

Where supported and appropriate:

```hcl
lifecycle {
  create_before_destroy = true
}
```

can reduce downtime during replacement.

It may temporarily increase resource usage.

---

## 70. Prevent Destroy

For critical resources:

```hcl
lifecycle {
  prevent_destroy = true
}
```

This is a guardrail, not a substitute for access controls.

---

## 71. Ignore Changes

Example:

```hcl
lifecycle {
  ignore_changes = [tags]
}
```

Use sparingly.

Overuse can hide real drift.

---

## 72. Terraform Drift

Drift occurs when:

```text
Terraform configuration/state
 ≠
AWS actual state
```

Detect with:

```bash
terraform plan
```

---

## 73. Manual AWS Change

Example:

```text
Engineer changes security group manually
 ↓
Terraform plan
 ↓
Shows drift
```

Do not automatically overwrite a production manual change until its intent is understood.

---

## 74. Drift Remediation

Options:

```text
Keep manual change
 → update Terraform

or

Reject manual change
 → Terraform restores desired state
```

Choose deliberately.

---

## 75. Terraform Refresh

Modern Terraform planning reads remote infrastructure state as part of planning.

Use standard plan/state operations rather than relying on obsolete workflows.

---

## 76. Terraform Taint

Older Terraform workflows used:

```bash
terraform taint
```

Modern Terraform generally uses:

```bash
terraform apply -replace='resource.address'
```

for explicit replacement.

---

## 77. `-replace`

Example:

```bash
terraform plan -replace='aws_instance.web'
```

Review the impact before apply.

---

## 78. Terraform Targeting

Commands such as:

```bash
terraform plan -target=...
```

can be useful for exceptional recovery situations but should not become the normal deployment strategy.

Targeted plans can omit dependencies.

---

## 79. Terraform Destroy

Avoid:

```bash
terraform destroy
```

from production automation unless the workflow explicitly requires controlled teardown.

Protect destroy capabilities.

---

## 80. CI Rules

GitLab rules can restrict Terraform jobs by:

```text
branch
tag
pipeline source
file changes
environment
```

Example concept:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

---

## 81. Changes-Only Terraform Pipelines

A monorepo may contain:

```text
terraform/
application/
docs/
```

Use GitLab `rules:changes` to avoid running infrastructure jobs for unrelated application changes.

---

## 82. Child Pipelines

Large Terraform repositories can use child pipelines:

```text
Main pipeline
 ├── VPC pipeline
 ├── EKS pipeline
 └── RDS pipeline
```

Use carefully to avoid excessive complexity.

---

## 83. Parent-Child Pipeline

A parent pipeline can trigger infrastructure-specific child pipelines.

This is useful when different infrastructure components have independent lifecycles.

---

## 84. Dynamic Pipelines

Generate pipeline configuration when infrastructure topology requires dynamic job creation.

Use only when static GitLab CI becomes insufficient.

---

## 85. Terraform Monorepo

Example:

```text
infra/
├── networking/
├── security/
├── eks/
├── ecr/
└── databases/
```

Each component can have independent validation/plan jobs.

---

## 86. Terraform Module Repository

Central modules:

```text
terraform-vpc
terraform-eks
terraform-rds
terraform-ecr
```

Applications consume versioned modules.

---

## 87. Module Versioning

Example:

```hcl
module "vpc" {
  source = "git::https://...//modules/vpc?ref=v2.1.0"
}
```

Pin module versions for reproducibility.

---

## 88. Module Upgrade

Treat module upgrades like application releases:

```text
upgrade
 ↓
plan
 ↓
test
 ↓
review
 ↓
production
```

---

## 89. Terraform Registry

Modules can be sourced from:

```text
Terraform Registry
GitLab repositories
GitHub repositories
private registries
```

Use approved sources.

---

## 90. Private Module Access

Private GitLab modules require appropriate authentication.

Avoid embedding personal access tokens in source URLs.

Use supported CI authentication mechanisms.

---

## 91. GitLab Package/Module Registry

Organizations can publish reusable Terraform modules through internal GitLab capabilities where supported.

This centralizes approved infrastructure components.

---

## 92. Module Governance

Define:

```text
owners
versioning
security standards
documentation
upgrade policy
deprecation
```

for shared modules.

---

## 93. Standard EKS Module

A reusable EKS module may manage:

```text
cluster
IAM
node groups
add-ons
security
logging
```

Keep application workloads outside the infrastructure module.

---

## 94. ECR Terraform Module

A reusable ECR module can manage:

```text
repository
encryption
lifecycle policy
tag immutability
repository policy
```

The application pipeline then pushes images.

---

## 95. VPC Terraform Module

Can manage:

```text
VPC
public subnets
private subnets
route tables
NAT
Internet Gateway
VPC endpoints
```

Use consistent CIDR planning.

---

## 96. IAM Terraform Module

Can create:

```text
roles
policies
trust relationships
OIDC configuration
```

Be careful: IAM changes can lock out automation.

---

## 97. GitLab OIDC Terraform

Terraform can manage:

```text
AWS OIDC provider
IAM roles
trust policies
permission policies
```

Then GitLab CI can assume those roles.

---

## 98. Terraform and GitLab OIDC Bootstrap

A common challenge:

```text
Terraform needs AWS credentials
 ↓
Terraform creates OIDC role
```

Initial bootstrap requires a trusted identity.

Do not assume the role being created can create itself.

---

## 99. Bootstrap Account

A controlled bootstrap identity can create:

```text
OIDC provider
Terraform role
state bucket
KMS key
```

Then normal pipelines use the restricted Terraform role.

---

## 100. Bootstrap Security

Bootstrap credentials should be:

```text
restricted
audited
rarely used
rotated
protected
```

Avoid leaving administrator credentials permanently available to CI.

---

## 101. Terraform State Bootstrap

State infrastructure may need to exist before the main Terraform stack.

Example:

```text
Bootstrap
 ├── S3
 ├── KMS
 └── locking capability

Main Terraform
 └── infrastructure
```

---

## 102. Terraform Workspace/Backend Mistake

If Terraform suddenly shows hundreds of resources to create:

```text
Stop apply
 ↓
Check backend
 ↓
Check workspace
 ↓
Check account
 ↓
Check region
```

Never assume the plan is correct.

---

## 103. Wrong AWS Account Detection

Always validate:

```bash
aws sts get-caller-identity
```

A wrong-account Terraform plan is a critical stop condition.

---

## 104. Wrong State Detection

Check:

```bash
terraform state list
```

Confirm expected resources are present.

---

## 105. Empty State Plan

If Terraform wants to recreate everything:

Possible causes:

```text
wrong backend
wrong workspace
wrong credentials
wrong region
state loss
state migration issue
```

Stop and investigate.

---

## 106. Provider Authentication Failure

Symptoms:

```text
No valid credential sources
AccessDenied
ExpiredToken
```

Check:

```text
OIDC token
role ARN
trust policy
AWS region
Runner environment
```

---

## 107. OIDC Token Failure

Check GitLab job configuration for:

```text
id_tokens
audience
```

and ensure AWS trust conditions match the actual GitLab token claims.

---

## 108. IAM Trust Policy Failure

A role may have correct permissions but still fail assumption.

Why?

```text
Trust policy
 ≠
Permission policy
```

The trust relationship must allow the GitLab OIDC principal/claims.

---

## 109. Terraform `AccessDenied`

Determine:

```text
Which API?
Which resource?
Which role?
Which account?
Which condition?
```

Then review the policy.

---

## 110. Permission Boundary Failure

A role may have:

```text
iam:CreateRole
```

in its identity policy but still fail because a permission boundary limits the maximum permissions.

Check all authorization layers.

---

## 111. SCP Failure

AWS Organizations SCPs can deny actions even when IAM allows them.

Check organizational policies for production accounts.

---

## 112. KMS Access Failure

Terraform may fail when creating encrypted resources.

Check:

```text
KMS key policy
IAM role
region
grant
```

---

## 113. S3 Backend Access Failure

Symptoms:

```text
AccessDenied
NoSuchBucket
NoSuchKey
```

Check:

```text
bucket
key
region
IAM
encryption
```

---

## 114. State Lock Failure

If state is locked:

```text
Identify active Terraform operation
 ↓
Verify no pipeline is running
 ↓
Inspect lock
 ↓
Use supported recovery procedure
```

Do not force-unlock blindly.

---

## 115. CI Concurrency

Two pipelines may run Terraform simultaneously:

```text
Pipeline A
 ↓
plan/apply

Pipeline B
 ↓
plan/apply
```

Use state locking plus GitLab concurrency controls where needed.

---

## 116. GitLab Resource Groups

GitLab `resource_group` can serialize jobs that operate on the same environment.

Concept:

```yaml
resource_group: production
```

This can prevent concurrent production deployments.

---

## 117. Terraform Environment Resource Group

Use separate resource groups:

```text
dev
stage
production
```

to avoid conflicting applies.

---

## 118. Plan Staleness

A saved plan can become stale if infrastructure changes after the plan was created.

Therefore:

```text
Plan
 ↓
Controlled approval
 ↓
Apply promptly
```

and ensure the workflow handles stale plans safely.

---

## 119. Production Plan Review

Review:

```text
creates
updates
replacements
destroys
IAM changes
network changes
database changes
```

Pay special attention to:

```text
-/+ replacement
destroy
```

---

## 120. IAM Changes Review

IAM changes can affect:

```text
CI
EKS
applications
developers
automation
```

Treat IAM changes as high-risk infrastructure changes.

---

## 121. Network Changes Review

Changes to:

```text
route tables
security groups
NACLs
NAT
subnets
```

can interrupt production.

Review dependency paths.

---

## 122. Database Changes Review

RDS changes may cause:

```text
replacement
downtime
storage changes
parameter changes
```

Use controlled approvals and backups.

---

## 123. EKS Changes Review

EKS changes can affect:

```text
cluster
nodes
IAM
network
add-ons
applications
```

Use non-production testing first.

---

## 124. Terraform Security Scanning

Use IaC scanners such as:

```text
Checkov
tfsec
Trivy config scanning
```

depending on organizational standards.

---

## 125. IaC Security Pipeline

Example:

```text
fmt
 ↓
validate
 ↓
IaC security scan
 ↓
plan
 ↓
review
 ↓
apply
```

Security scanning should occur before production mutation.

---

## 126. Terraform Compliance

Policy can enforce:

```text
encrypted storage
no public S3
restricted security groups
approved regions
required tags
```

---

## 127. Policy as Code

Tools/patterns can enforce policies before apply.

Concept:

```text
Terraform plan
 ↓
Policy evaluation
 ↓
Pass/Fail
 ↓
Approval
```

---

## 128. Plan JSON

Terraform can produce machine-readable plan output for policy analysis.

Concept:

```bash
terraform show -json tfplan
```

Protect plan data because it may contain sensitive information.

---

## 129. Security Gate

Example:

```text
Terraform plan
 ↓
Checkov
 ↓
Policy
 ↓
Fail if prohibited
```

Do not rely solely on human review for repeatable security rules.

---

## 130. Cost Policy

Terraform plans can also be evaluated for cost impact.

Example concept:

```text
Plan
 ↓
Cost estimate
 ↓
Threshold
 ↓
Approval
```

This can catch unexpectedly expensive resources.

---

## 131. Terraform Documentation

Every production module should document:

```text
Purpose
Inputs
Outputs
Dependencies
Version
Examples
Security considerations
Upgrade notes
```

---

## 132. README for Infrastructure

Example:

```text
Module
 ├── Purpose
 ├── Usage
 ├── Inputs
 ├── Outputs
 ├── Example
 └── Upgrade notes
```

Documentation reduces operational dependency on individuals.

---

## 133. Code Review Standards

Review Terraform for:

```text
security
naming
tags
dependencies
resource replacement
IAM
networking
cost
DR
```

---

## 134. Terraform Testing

Possible levels:

```text
fmt
validate
static analysis
plan
module tests
integration tests
```

Use deeper tests for critical modules.

---

## 135. Terraform Unit-Like Testing

Test module behavior using supported Terraform testing approaches.

Examples:

```text
expected variables
resource configuration
outputs
conditions
```

---

## 136. Integration Testing

For critical infrastructure:

```text
Terraform apply test environment
 ↓
Validate AWS resources
 ↓
Destroy test environment
```

Use isolated accounts/environments where practical.

---

## 137. Terraform Test Environment

A test environment can validate:

```text
VPC
EKS
IAM
ECR
RDS
```

before production.

Control cost and cleanup carefully.

---

## 138. Terraform Destroy Test Environment

Automated cleanup:

```text
Test complete
 ↓
Destroy
 ↓
Verify resources
```

Do not reuse production credentials.

---

## 139. Terraform Provider Locking

Commit:

```text
.terraform.lock.hcl
```

and review lock changes.

Unexpected provider checksum/version changes should be investigated.

---

## 140. Dependency Updates

Terraform dependencies include:

```text
Terraform version
providers
modules
CI image
security tools
```

Manage them deliberately.

---

## 141. Terraform Supply Chain

Use:

```text
trusted providers
trusted modules
version pinning
checksums
code review
```

Avoid untrusted Terraform modules.

---

## 142. Private Provider/Module Sources

Enterprise environments may use private registries.

Control:

```text
authentication
versioning
availability
provenance
```

---

## 143. Terraform Secrets in State

Even if a variable is marked:

```hcl
sensitive = true
```

the value may still be present in state.

Therefore:

```text
secure backend
restricted access
encryption
```

remain mandatory.

---

## 144. Terraform State as a Security Boundary

Anyone with unrestricted state access may learn:

```text
resource details
identifiers
configuration
possibly secrets
```

Treat state like sensitive infrastructure data.

---

## 145. State Access Audit

Monitor:

```text
GetObject
PutObject
DeleteObject
```

for the Terraform state bucket.

Unexpected access should be investigated.

---

## 146. Terraform CI Secret Hygiene

Never:

```text
echo AWS_SECRET_ACCESS_KEY
```

or print sensitive Terraform variables.

Use masked/protected mechanisms.

---

## 147. CI Log Security

Avoid commands that dump:

```text
environment
credentials
tokens
secret files
```

Use safe diagnostics.

---

## 148. Debugging Terraform CI

Useful safe commands:

```bash
terraform version
aws sts get-caller-identity
terraform providers
terraform state list
```

Do not enable unrestricted shell tracing when secrets are present.

---

## 149. Terraform Logging

Terraform supports logging controls.

Use debug logging temporarily for troubleshooting and ensure logs do not expose sensitive information.

---

## 150. Terraform Plan Failure

Separate:

```text
configuration error
provider error
permission error
state error
AWS API error
```

This speeds diagnosis.

---

## 151. Terraform Apply Failure

An apply may partially change infrastructure before failing.

After failure:

```text
Check state
 ↓
Check AWS actual state
 ↓
Run plan
 ↓
Understand remaining changes
```

Do not blindly rerun destructive operations.

---

## 152. Partial Infrastructure Creation

Example:

```text
VPC created
EKS failed
```

Terraform state may already contain the VPC.

Run a new plan after understanding the failure.

---

## 153. Terraform Dependency Failure

Example:

```text
IAM role
 ↓
EKS
```

If IAM is incorrect, EKS may fail.

Fix root dependency rather than repeatedly retrying the downstream resource.

---

## 154. Terraform Timeouts

Some AWS resources take significant time.

Terraform resources often support timeout configuration where appropriate.

Do not increase timeouts blindly; determine why provisioning is slow.

---

## 155. AWS Eventual Consistency

IAM and some AWS APIs can exhibit propagation delays.

A resource created immediately before another operation may not be visible everywhere.

Use provider-supported behavior/retries and avoid arbitrary sleep commands unless there is no better approach.

---

## 156. Avoid `sleep` Workarounds

Bad pattern:

```bash
sleep 120
```

without understanding the dependency.

Prefer:

```text
correct dependency
provider retry
resource readiness check
```

---

## 157. Terraform Apply Retry

Retry only:

```text
transient failures
```

Do not automatically retry:

```text
AccessDenied
invalid configuration
resource conflicts
```

---

## 158. GitLab Retry Policy

GitLab jobs can be configured to retry selected failures.

Use carefully for infrastructure jobs.

A blind retry can repeat a dangerous operation.

---

## 159. Manual Production Apply

Manual approval is not enough by itself.

Combine:

```text
protected branch
+
protected environment
+
restricted IAM
+
review
+
plan
+
audit
```

---

## 160. Terraform Production Guardrails

```text
[ ] Protected branch
[ ] Protected environment
[ ] OIDC role
[ ] Least privilege
[ ] Plan review
[ ] Approval
[ ] State lock
[ ] Resource group
[ ] Account validation
[ ] Region validation
[ ] Security scan
[ ] Backup/recovery
```

---

## 161. GitLab Terraform Environment Matrix

| Environment | AWS Account | Role | Apply |
|---|---|---|---|
| Dev | Dev | Dev Terraform role | Automated/controlled |
| Stage | Stage | Stage Terraform role | Controlled |
| Prod | Production | Protected Prod role | Approved |

Exact automation level depends on organizational policy.

---

## 162. Terraform Pipeline Matrix

```text
Feature Branch
 ↓
fmt
 ↓
validate
 ↓
security

Merge Request
 ↓
plan

Main
 ↓
approved apply

Production
 ↓
protected approval
 ↓
apply
```

---

## 163. GitLab MR Terraform Comment

A pipeline can publish summarized plan information to the MR.

Useful information:

```text
Create: 4
Change: 2
Destroy: 0
```

Avoid exposing sensitive plan details.

---

## 164. Plan Summary

A safe summary helps reviewers focus on:

```text
resource count
critical resources
replacement
destroy
IAM
network
database
```

---

## 165. Terraform Destroy Detection

Production pipeline should highlight:

```text
destroy > 0
```

and especially:

```text
database destroy
VPC destroy
EKS destroy
```

for explicit review.

---

## 166. Replacement Detection

Highlight:

```text
-/+ replace
```

because replacement can cause downtime/data loss depending on resource.

---

## 167. Terraform and Database Migration

Infrastructure provisioning and application database migration should have clearly defined ownership.

Avoid coupling a Terraform apply to irreversible schema changes unless deliberately designed.

---

## 168. Terraform and EKS

Terraform can provision:

```text
VPC
EKS
Node groups
IAM
ECR
Add-ons
```

Application manifests should normally remain under GitOps management.

---

## 169. Terraform and ALB

Terraform can manage:

```text
network
IAM
supporting AWS infrastructure
```

while AWS Load Balancer Controller manages ALBs generated from Kubernetes Ingress.

Avoid managing the same ALB through both Terraform and Kubernetes.

---

## 170. Terraform and ECR

Terraform creates:

```text
ECR repository
lifecycle
encryption
policies
```

GitLab CI handles:

```text
image build
scan
push
```

---

## 171. Terraform and RDS

Terraform can manage:

```text
RDS instance
subnet group
security group
parameter group
backup settings
```

Application secrets should be managed separately.

---

## 172. Terraform and S3

Terraform can manage:

```text
buckets
policies
encryption
versioning
lifecycle
```

Be careful when Terraform manages its own backend infrastructure.

---

## 173. Terraform and Route 53

Terraform can manage:

```text
hosted zones
records
aliases
```

Use change review because DNS changes can affect production globally.

---

## 174. Terraform and ACM

Terraform can manage:

```text
certificate
DNS validation
validation records
```

Certificate lifecycle should be planned to avoid production TLS disruption.

---

## 175. Terraform and IAM OIDC

Terraform can create:

```text
OIDC provider
trust policies
GitLab deployment roles
EKS workload roles
```

IAM changes should receive elevated review.

---

## 176. Terraform and ArgoCD

Terraform can provision:

```text
EKS
ArgoCD infrastructure/support
IAM
```

ArgoCD should manage:

```text
Kubernetes application state
```

Avoid resource ownership overlap.

---

## 177. GitOps Repository Boundary

Recommended:

```text
Infrastructure Repo
 ↓
Terraform
 ↓
AWS

Application/GitOps Repo
 ↓
Helm/Kustomize
 ↓
ArgoCD
 ↓
EKS
```

This provides clear ownership.

---

## 178. Infrastructure Repository Security

Protect:

```text
main branch
production directories
Terraform modules
backend configuration
IAM code
```

Use CODEOWNERS for critical directories.

---

## 179. CODEOWNERS

Example concept:

```text
terraform/iam/ @platform-security
terraform/networking/ @platform-team
terraform/prod/ @platform-leads
```

This ensures specialized review.

---

## 180. Terraform Release Process

```text
Change
 ↓
MR
 ↓
Validation
 ↓
Security
 ↓
Plan
 ↓
Review
 ↓
Merge
 ↓
Protected Apply
 ↓
Validation
```

Treat infrastructure like production software.

---

## 181. Terraform Change Ticket

For regulated environments, connect:

```text
GitLab MR
+
Terraform plan
+
Approval
+
Change record
```

This creates traceability.

---

## 182. Infrastructure Rollback

Terraform does not have a simple application-style rollback button.

Rollback usually means:

```text
Revert code
 ↓
Plan
 ↓
Apply
```

State and actual infrastructure must be considered.

---

## 183. Terraform Rollback Risk

Reverting code can produce:

```text
resource replacement
downgrade
destructive changes
```

Always review the new plan.

---

## 184. Infrastructure Recovery

If a change breaks infrastructure:

```text
Stop further changes
 ↓
Assess AWS state
 ↓
Identify last known-good configuration
 ↓
Restore/revert
 ↓
Plan
 ↓
Apply controlled fix
```

---

## 185. State Recovery vs Infrastructure Recovery

These are different:

```text
State recovery
→ restore Terraform metadata

Infrastructure recovery
→ restore actual AWS resources
```

You may need one or both.

---

## 186. Terraform Backup

Protect:

```text
state versions
Git repository
module versions
provider lock file
CI configuration
```

These collectively support infrastructure recovery.

---

## 187. Git Repository as Recovery Source

If Terraform code is version controlled:

```text
Git commit
 ↓
Terraform configuration
```

You can reconstruct infrastructure definitions.

State still needs separate protection.

---

## 188. Terraform Disaster Recovery Test

Perform controlled tests:

```text
State recovery
Module recovery
AWS resource recreation
EKS recovery
```

Document recovery times.

---

## 189. RTO and RPO

Infrastructure DR should define:

```text
RTO
→ How quickly can infrastructure recover?

RPO
→ How much state/data can be lost?
```

Terraform state backup supports the recovery model.

---

## 190. Terraform Cost Controls

Use policy to detect expensive changes:

```text
large EC2
NAT Gateway
RDS class
EKS node count
load balancer
```

before apply.

---

## 191. Resource Tag Enforcement

Require:

```text
Environment
Application
Owner
ManagedBy
CostCenter
```

through Terraform modules/policy.

---

## 192. Naming Policy

Standard names reduce operational ambiguity:

```text
roboshop-prod-eks
roboshop-prod-ecr-user
roboshop-prod-rds
```

Avoid inconsistent naming across environments.

---

## 193. Terraform Logging and Observability

Pipeline logs should capture:

```text
job
commit
environment
AWS account
region
plan result
apply result
```

Do not capture secrets.

---

## 194. Terraform Metrics

Track:

```text
plan duration
apply duration
failure rate
resource changes
drift incidents
pipeline frequency
```

These are useful platform engineering metrics.

---

## 195. GitLab Terraform Monitoring

Useful operational signals:

```text
failed plan jobs
failed apply jobs
OIDC failures
state lock failures
AWS AccessDenied
pipeline duration
```

---

## 196. Terraform Pipeline Optimization

Improve:

```text
provider/module caching
Runner size
parallel jobs
changes-based rules
child pipelines
state separation
```

Do not optimize by weakening safety controls.

---

## 197. Provider Plugin Cache

Caching Terraform provider plugins can reduce repeated downloads on ephemeral runners.

Ensure cache integrity and version control.

---

## 198. CI Dependency Cache

Cache:

```text
Terraform providers
security tool databases
```

where safe.

Do not cache:

```text
Terraform state
credentials
secrets
```

as ordinary shared cache data.

---

## 199. Terraform Runner Isolation

Infrastructure jobs should run on trusted runners.

For production:

```text
protected runner
+
protected environment
```

is preferable.

---

## 200. Runner Compromise Risk

If a Runner can assume a powerful Terraform role:

```text
Runner compromise
 ↓
AWS infrastructure access
```

Therefore protect:

```text
Runner
IAM role
GitLab environment
```

together.

---

## 201. Terraform Job Identity Separation

Example:

```text
Validate
 → no AWS write

Plan
 → read/limited access

Apply
 → production write role
```

This reduces blast radius.

---

## 202. Read-Only Plan Role

A plan may require substantial read permissions.

Where practical, separate:

```text
plan identity
apply identity
```

while accounting for Terraform data-source/resource behavior.

---

## 203. Plan and Apply Role Difference

A plan may need to inspect resources.

An apply needs:

```text
create
update
delete
```

permissions.

The apply role therefore requires stronger protection.

---

## 204. GitLab Job Token

Use GitLab's built-in identity mechanisms for GitLab-to-GitLab operations where possible.

Do not use personal credentials for automated infrastructure workflows unnecessarily.

---

## 205. Terraform Module Security Review

Review modules for:

```text
public resources
open security groups
wildcard IAM
unencrypted storage
public databases
hard-coded secrets
untrusted downloads
```

---

## 206. Security Misconfiguration Example

Bad:

```hcl
cidr_blocks = ["0.0.0.0/0"]
```

on an administrative port.

Better:

```text
restricted source
private connectivity
bastion/SSM
```

according to architecture.

---

## 207. Public Database Risk

Do not expose RDS publicly unless explicitly required and heavily controlled.

Preferred:

```text
EKS private subnet
 ↓
RDS private subnet
```

---

## 208. Public S3 Risk

Do not make infrastructure/state buckets public.

Use:

```text
Block Public Access
```

and restrictive policies.

---

## 209. Encryption by Default

Terraform modules should enable encryption for:

```text
EBS
RDS
S3
ECR where supported/configured
Kubernetes secrets at rest where applicable
```

---

## 210. IAM Wildcards

Avoid:

```hcl
actions   = ["*"]
resources = ["*"]
```

unless there is a documented exceptional requirement.

---

## 211. Terraform Security Checklist

```text
[ ] No hard-coded credentials
[ ] OIDC for GitLab
[ ] Least-privilege IAM
[ ] Protected production role
[ ] Private state bucket
[ ] Encryption
[ ] Versioning
[ ] Locking
[ ] Provider lock file
[ ] Module versioning
[ ] IaC scanning
[ ] Policy as code
[ ] Secrets externalized
[ ] Protected branches
[ ] CODEOWNERS
[ ] Plan review
[ ] Apply approval
```

---

## 212. Senior Interview — How Do You Run Terraform in GitLab?

> I use GitLab CI stages for formatting, validation, security scanning, planning, and controlled apply. Merge requests show the infrastructure impact, while production apply is protected by branch/environment controls, approvals, and a least-privilege AWS role obtained through OIDC.

---

## 213. Senior Interview — How Do You Secure Terraform State?

> I use a private encrypted S3 backend with restricted IAM access, versioning, supported state locking, audit controls, and recovery procedures. State is treated as sensitive infrastructure data and is never exposed as a normal public artifact.

---

## 214. Senior Interview — How Do You Prevent Two Terraform Pipelines From Running Together?

> I use Terraform backend locking and GitLab resource groups or equivalent pipeline serialization for shared environments. This prevents concurrent production applies and reduces state corruption or conflicting infrastructure changes.

---

## 215. Senior Interview — What Happens If Terraform Wants to Destroy Production Resources?

> I stop and inspect the plan. I check state, backend, workspace, account, region, recent code changes, provider/module changes, and resource dependencies. I never approve a destructive production plan without understanding why those resources are being replaced or destroyed.

---

## 216. Senior Interview — How Do You Handle Terraform Drift?

> I run a plan to identify the difference between configuration and AWS. Then I determine whether the manual change is intended. If it should remain, I update Terraform; otherwise Terraform can restore the declared configuration through a controlled apply.

---

## 217. Senior Interview — How Do You Handle Terraform State Loss?

> I stop concurrent Terraform operations, identify the latest valid state version from the protected backend, preserve the current state for investigation, restore carefully, and run a plan to compare Terraform state with actual AWS resources before making changes.

---

## 218. Senior Interview — Why Use Terraform Modules?

> Modules standardize infrastructure patterns and reduce duplication. I use modules for reusable components such as VPC, EKS, ECR, IAM and RDS while keeping environment composition explicit.

---

## 219. Senior Interview — How Do You Manage Terraform Versions?

> I control the Terraform CLI version, pin provider versions through `required_providers`, commit `.terraform.lock.hcl`, and upgrade providers/modules through a tested change process rather than automatically upgrading production.

---

## 220. Senior Interview — How Do You Secure Terraform in AWS?

> GitLab assumes a restricted AWS role through OIDC. The production role is protected, Terraform state is secured in S3, sensitive values are externalized, infrastructure is scanned for security issues, and high-risk changes such as IAM, networking, EKS and databases receive additional review.

---

## 221. Senior Interview — How Do You Separate Terraform and ArgoCD?

> Terraform manages AWS infrastructure and supporting platform resources. ArgoCD manages Kubernetes application desired state. This avoids overlapping ownership and makes infrastructure and application lifecycle management clearer.

---

## 222. Senior Interview — How Do You Handle Terraform Apply Failure?

> I first determine what was actually created or changed by checking Terraform state and AWS. Then I run a new plan, identify the remaining changes, fix the root cause, and apply the corrected configuration. I don't blindly rerun destructive commands.

---

## 223. Senior Interview — How Do You Handle Wrong AWS Account?

> I treat it as a hard stop. Before production operations I run `aws sts get-caller-identity`, verify the account and role, validate the region, and only then allow Terraform operations to proceed.

---

## 224. Senior Interview — How Do You Protect Production Terraform?

> Production uses a protected GitLab environment, protected branch, dedicated AWS role, OIDC trust restrictions, plan review, approval, serialized apply jobs, state locking, and audit logging.

---

## 225. Senior Interview — What Is Your Production Terraform Architecture?

> GitLab stores version-controlled Terraform code. CI validates and scans it, generates a reviewable plan, and a protected apply job assumes a restricted AWS role through OIDC. Terraform uses a secure remote S3 backend and provisions AWS infrastructure such as VPC, EKS, ECR, IAM, ALB-related infrastructure and RDS.

---

## 226. Complete GitLab + Terraform Architecture

```text
                           Developer
                               │
                               ▼
                           GitLab MR
                               │
                     ┌─────────┴─────────┐
                     ▼                   ▼
                   Validate            Security
                     │                   │
                     └─────────┬─────────┘
                               ▼
                            Terraform
                               │
                               ▼
                              Plan
                               │
                         Review/Approval
                               │
                               ▼
                         Protected Apply
                               │
                         GitLab OIDC
                               │
                               ▼
                              STS
                               │
                               ▼
                         Terraform IAM Role
                               │
                               ▼
                              AWS
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
             VPC              EKS              RDS
              │                │
              ▼                ▼
             ECR            Kubernetes
                               │
                               ▼
                            ArgoCD
```

---

## 227. Final Production Terraform Workflow

```text
Code
 ↓
MR
 ↓
fmt
 ↓
validate
 ↓
IaC security
 ↓
plan
 ↓
Review
 ↓
Merge
 ↓
Protected environment
 ↓
AWS OIDC
 ↓
Terraform Apply
 ↓
AWS
 ↓
Post-change validation
```

---

## 228. Final Terraform Production Checklist

```text
[ ] Version-controlled Terraform
[ ] Reusable modules
[ ] Separate environments
[ ] Separate production AWS account where appropriate
[ ] Remote S3 state
[ ] State encryption
[ ] State versioning
[ ] State locking
[ ] OIDC authentication
[ ] Environment-specific IAM roles
[ ] Protected production apply
[ ] Resource group/concurrency control
[ ] fmt
[ ] validate
[ ] plan
[ ] IaC security scan
[ ] Policy checks
[ ] Plan review
[ ] IAM review
[ ] Network review
[ ] Database review
[ ] EKS review
[ ] Recovery procedures
[ ] Drift detection
[ ] Audit trail
[ ] Cost controls
```

---

## 229. Final Mental Model

```text
                    GitLab
                       │
                       ▼
                 Terraform CI
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        fmt         Security       Plan
                       │            │
                       └──────┬─────┘
                              ▼
                           Review
                              │
                              ▼
                        Protected Apply
                              │
                           OIDC/STS
                              │
                              ▼
                         AWS IAM Role
                              │
                              ▼
                              AWS
        ┌─────────────┬───────┼────────┬───────────┐
        ▼             ▼       ▼        ▼           ▼
       VPC           EKS     ECR      RDS         S3
        │             │
        │             ▼
        │          ArgoCD
        │             │
        └─────────────┴──────► Kubernetes Apps
```

> **The core principle is simple: GitLab controls the Terraform workflow, Terraform defines infrastructure, AWS IAM controls what CI can change, S3 securely stores state, and ArgoCD remains responsible for Kubernetes application reconciliation. Production safety comes from separation, least privilege, immutable versions, reviewable plans, protected applies, and tested recovery.**

---