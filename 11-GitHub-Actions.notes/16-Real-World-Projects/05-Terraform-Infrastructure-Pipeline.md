# GitHub Actions - Terraform Infrastructure CI/CD Pipeline

This project demonstrates how to design and implement a production-style
Infrastructure as Code CI/CD pipeline using GitHub Actions and Terraform.

The pipeline covers:

```
Terraform Code
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Terraform Format
    |
    ↓
Terraform Validate
    |
    ↓
Security Scan
    |
    ↓
Terraform Plan
    |
    ↓
Review / Approval
    |
    ↓
Terraform Apply
    |
    ↓
AWS Infrastructure
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 1. Project Overview

Terraform is used to provision and manage infrastructure as code.

The pipeline automates:

```
Terraform Formatting
    +
Terraform Validation
    +
Security Scanning
    +
Terraform Plan
    +
Plan Review
    +
Terraform Apply
    +
Infrastructure Validation
```

Example infrastructure:

```
AWS
 |
 +--- VPC
 |
 +--- Subnets
 |
 +--- Security Groups
 |
 +--- IAM
 |
 +--- EC2
 |
 +--- EKS
 |
 +--- ALB
 |
 +--- RDS
 |
 +--- S3
 |
 +--- ECR
```

---

# 2. Project Objective

The objective is to create a secure and controlled Terraform pipeline
that:

```
1. Validates Terraform code
2. Enforces formatting
3. Performs security checks
4. Creates a Terraform plan
5. Stores the plan as an artifact where appropriate
6. Reviews infrastructure changes
7. Requires approval for production
8. Applies approved infrastructure changes
9. Validates the resulting infrastructure
10. Maintains Terraform state safely
11. Supports controlled rollback and recovery
12. Provides complete infrastructure traceability
```

---

# 3. Technology Stack

## Infrastructure as Code

```
Terraform
```

## Source Control

```
Git
GitHub
```

## CI/CD

```
GitHub Actions
```

## Cloud

```
AWS
```

## State

```
Amazon S3
Terraform Remote State
```

## Security

```
Trivy
Terraform Security Scanner
Secret Detection
```

## Authentication

```
GitHub OIDC
AWS IAM
```

## Infrastructure Components

```
VPC
Subnets
Security Groups
IAM
EC2
EKS
ALB
RDS
S3
ECR
```

---

# 4. High-Level Architecture

The complete flow is:

```
Developer
    |
    ↓
GitHub
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- fmt
    +--- validate
    +--- security scan
    +--- plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
Terraform Apply
    |
    ↓
AWS
    |
    ↓
Infrastructure
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 5. Repository Structure

A typical Terraform repository can look like:

```
terraform-infrastructure/
│
├── modules/
│   ├── vpc/
│   ├── security-group/
│   ├── ec2/
│   ├── iam/
│   ├── ecr/
│   ├── alb/
│   ├── eks/
│   ├── rds/
│   └── s3/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── providers.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── backend.tf
│
└── .github/
    └── workflows/
        ├── terraform-ci.yml
        └── terraform-cd.yml
```

---

# 6. Terraform Module Structure

A reusable Terraform module can contain:

```
module/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
└── README.md
```

The purpose of modules is to provide reusable infrastructure
components.

Example:

```
VPC Module
    |
    ↓
VPC
    +
Subnets
    +
Route Tables
    +
Internet Gateway
```

---

# 7. Example Infrastructure Modules

A production infrastructure repository may contain modules such as:

```
00-vpc
    |
    ↓
10-security-groups
    |
    ↓
20-bastion
    |
    ↓
30-security-group-rules
    |
    ↓
40-ecr
    |
    ↓
70-acm
    |
    ↓
80-frontend-alb
    |
    ↓
90-eks
```

The exact module structure depends on the infrastructure design.

---

# 8. Terraform Backend

Terraform state should be stored remotely for shared environments.

Example architecture:

```
GitHub Actions
    |
    ↓
Terraform
    |
    ↓
Remote State
    |
    ↓
Amazon S3
```

Remote state provides a centralized source of Terraform state.

---

# 9. Why Remote State?

Without remote state:

```
Developer A
    |
    ↓
Local State

Developer B
    |
    ↓
Different Local State
```

This can create:

```
State Drift
    +
Conflicts
    +
Unsafe Collaboration
```

With remote state:

```
Developer
    |
    ↓
Terraform
    |
    ↓
Shared Remote State
```

---

# 10. Terraform State

Terraform state tracks the relationship between:

```
Terraform Configuration
    |
    +
Real Infrastructure
```

Conceptually:

```
Terraform Code
    |
    ↓
State
    |
    ↓
AWS Infrastructure
```

Terraform uses this information when creating a plan.

---

# 11. State Locking

State locking prevents multiple Terraform operations from modifying
the same state simultaneously when supported by the backend.

The backend configuration should use the locking capabilities
supported by the selected Terraform and S3 backend configuration.

The important principle is:

```
One State
    +
Controlled Access
    +
Safe Concurrent Operations
```

---

# 12. State Security

Terraform state can contain sensitive infrastructure information.

Therefore:

```
Restrict Access
    +
Encrypt State
    +
Use IAM
    +
Avoid Public Access
    +
Audit Access
```

Never treat Terraform state as ordinary public source code.

---

# 13. Terraform Workflow

The standard workflow is:

```
terraform fmt
    |
    ↓
terraform init
    |
    ↓
terraform validate
    |
    ↓
Security Scan
    |
    ↓
terraform plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
terraform apply
```

---

# 14. Workflow Location

GitHub Actions workflows are stored under:

```
.github/workflows/
```

Example:

```
.github/workflows/terraform-ci.yml
```

---

# 15. Workflow Triggers

Typical triggers:

```
pull_request
    +
push
    +
workflow_dispatch
```

Pull requests are used for validation and plan generation.

Production apply should use controlled branches and environments.

---

# 16. Pull Request Flow

Example:

```
Developer
    |
    ↓
Terraform Change
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- fmt
    +--- validate
    +--- security
    +--- plan
    |
    ↓
Plan Review
    |
    ↓
Merge
```

---

# 17. Terraform Format

The first quality check is:

```
terraform fmt
```

Purpose:

```
Standardize Formatting
    +
Improve Readability
    +
Reduce Unnecessary Differences
```

The pipeline can fail when Terraform files are not formatted
according to project standards.

---

# 18. Terraform Format Check

A CI workflow can check formatting without modifying the developer's
repository.

Conceptually:

```
Terraform Code
    |
    ↓
Format Check
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Fix

---

# 19. Terraform Init

Before validation and planning, Terraform initializes:

```
Providers
    +
Modules
    +
Backend
    +
Dependencies
```

Flow:

```
Terraform Code
    |
    ↓
terraform init
    |
    ↓
Initialized Working Directory
```

---

# 20. Provider Configuration

Terraform providers allow Terraform to communicate with cloud APIs.

Example:

```
Terraform
    |
    ↓
AWS Provider
    |
    ↓
AWS APIs
    |
    ↓
Infrastructure
```

Provider versions should be controlled to improve reproducibility.

---

# 21. Provider Version Constraints

Provider versions should be explicitly constrained.

Conceptually:

```
Terraform
    |
    ↓
Required Provider Version
    |
    ↓
AWS Provider
    |
    ↓
AWS
```

This reduces unexpected changes caused by provider upgrades.

---

# 22. Terraform Validate

The pipeline runs:

```
terraform validate
```

Purpose:

```
Syntax Validation
    +
Configuration Validation
    +
Internal Consistency Checks
```

Flow:

```
Terraform Code
    |
    ↓
terraform validate
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 23. Terraform Security Scanning

Terraform code should be scanned for insecure configurations.

Possible checks include:

```
Public Resources
    +
Open Security Groups
    +
Weak IAM Policies
    +
Unencrypted Storage
    +
Insecure Network Configuration
    +
Hardcoded Secrets
```

The organization can use its approved Terraform security scanner.

---

# 24. Security Scan Flow

```
Terraform Code
    |
    ↓
Security Scanner
    |
    ↓
Findings
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Fix

Security severity thresholds should be defined by policy.

---

# 25. Secret Detection

Never commit:

```
AWS Access Keys
    +
Secret Keys
    +
Database Passwords
    +
API Tokens
    +
Private Keys
```

into Terraform files.

Bad:

```
variable = "hardcoded-secret"
```

Preferred:

```
Secure Secret Management
    +
Runtime Injection
    +
GitHub Secrets / OIDC
    +
AWS Secret Management
```

---

# 26. AWS Authentication

GitHub Actions should use OIDC where appropriate.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC Token
    |
    ↓
AWS IAM Role
    |
    ↓
Temporary Credentials
    |
    ↓
Terraform
    |
    ↓
AWS
```

---

# 27. Why OIDC?

OIDC avoids storing long-lived AWS access keys in GitHub.

Traditional approach:

```
GitHub
    |
    ↓
Long-Lived AWS Keys
    |
    ↓
AWS
```

Preferred:

```
GitHub
    |
    ↓
OIDC
    |
    ↓
Temporary IAM Credentials
    |
    ↓
AWS
```

Benefits:

```
Short-Lived Credentials
    +
Reduced Credential Exposure
    +
Fine-Grained Trust
    +
Better Security
```

---

# 28. IAM Role

The Terraform IAM role should have only the permissions required to
manage the intended infrastructure.

Principle:

```
Least Privilege
```

Avoid:

```
AdministratorAccess
```

unless there is a documented and justified requirement.

---

# 29. Terraform Plan

The most important CI/CD Terraform step is:

```
terraform plan
```

Terraform calculates the proposed infrastructure changes.

Flow:

```
Terraform Code
    +
Current State
    +
AWS Infrastructure
    |
    ↓
terraform plan
    |
    ↓
Proposed Changes
```

---

# 30. Terraform Plan Output

The plan identifies operations such as:

```
Create
    +
Update
    +
Destroy
    +
Replace
```

Example:

```
+ Create
~ Update
- Destroy
+/- Replace
```

---

# 31. Why Terraform Plan Is Important

Plan provides visibility before modifying infrastructure.

Without plan:

```
Code
    |
    ↓
Apply
    |
    ↓
Infrastructure Changes
```

With plan:

```
Code
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
Apply
```

---

# 32. Plan Review

A production plan should be reviewed before apply.

Review:

```
Resources To Create
    +
Resources To Change
    +
Resources To Destroy
    +
Security Impact
    +
Cost Impact
    +
Availability Impact
```

---

# 33. Plan Artifact

The Terraform plan can be saved as an artifact where appropriate.

Conceptually:

```
terraform plan
    |
    ↓
Plan File
    |
    ↓
GitHub Actions Artifact
    |
    ↓
Review / Controlled Apply
```

The artifact should be handled securely and only used in a workflow
where the configuration and state assumptions remain valid.

---

# 34. Plan and Apply Consistency

A plan should not be treated as permanently valid.

Between:

```
Plan
    |
    ↓
Apply
```

the infrastructure or state may change.

Therefore production workflows should carefully control when a saved
plan is applied and verify that the plan remains appropriate.

---

# 35. Terraform Apply

After approval:

```
terraform apply
```

Flow:

```
Approved Plan
    |
    ↓
Terraform Apply
    |
    ↓
AWS APIs
    |
    ↓
Infrastructure
```

---

# 36. Production Apply

A secure production flow is:

```
Pull Request
    |
    ↓
CI
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Merge
    |
    ↓
Production Environment
    |
    ↓
Approval
    |
    ↓
Apply
```

---

# 37. GitHub Environments

GitHub Environments can provide:

```
Approval Rules
    +
Environment Protection
    +
Environment Secrets
    +
Deployment Tracking
```

Example:

```
dev
    +
qa
    +
prod
```

Production can require explicit approval.

---

# 38. Separation of Duties

Infrastructure delivery should separate responsibilities.

Example:

```
Developer
    |
    ↓
Terraform Change

Reviewer
    |
    ↓
Plan Review

Security
    |
    ↓
Security Validation

Approver
    |
    ↓
Production Apply
```

This reduces the risk of unauthorized infrastructure changes.

---

# 39. Environment Separation

Use separate configurations or workspaces according to the
organization's Terraform architecture.

Example:

```
DEV
    |
    ↓
QA
    |
    ↓
PROD
```

Each environment should have appropriate:

```
State
    +
Credentials
    +
Variables
    +
Approval Rules
```

---

# 40. Terraform Variables

Variables allow infrastructure configuration to be parameterized.

Example categories:

```
Region
    +
Environment
    +
Instance Type
    +
VPC CIDR
    +
Cluster Name
    +
Tags
```

Avoid hardcoding environment-specific values throughout modules.

---

# 41. Terraform Outputs

Outputs expose important infrastructure information.

Examples:

```
VPC ID
    +
Subnet IDs
    +
EKS Cluster Name
    +
ALB DNS Name
    +
ECR Repository URL
    +
RDS Endpoint
```

Outputs can be consumed by other infrastructure or operational
processes.

---

# 42. Terraform Modules

Modules improve reuse.

Example:

```
VPC Module
    |
    ↓
DEV VPC

Same Module
    |
    ↓
QA VPC

Same Module
    |
    ↓
PROD VPC
```

The module code remains standardized while inputs differ.

---

# 43. Module Versioning

Reusable modules should be version controlled.

Benefits:

```
Reproducibility
    +
Controlled Upgrades
    +
Easier Rollback
    +
Dependency Traceability
```

Avoid uncontrolled module changes in production.

---

# 44. Terraform Dependency Graph

Terraform determines dependencies between resources.

Example:

```
VPC
    |
    ↓
Subnet
    |
    ↓
Security Group
    |
    ↓
EKS
    |
    ↓
Application
```

Terraform uses the dependency graph to determine execution order.

---

# 45. Parallel Resource Creation

Terraform can create independent resources in parallel when
dependencies allow.

Example:

```
VPC
  |
  +---------+
  |         |
  ↓         ↓
Subnet A  Subnet B
```

This can improve execution efficiency.

---

# 46. Terraform Destroy

Destroy removes resources managed by Terraform.

Command concept:

```
terraform destroy
```

This is dangerous in production.

Production destruction should require:

```
Explicit Approval
    +
Plan Review
    +
Appropriate Permissions
    +
Change Management
```

---

# 47. Preventing Accidental Destruction

Use multiple controls:

```
Pull Request Review
    +
Terraform Plan
    +
Production Approval
    +
Least Privilege
    +
Resource Lifecycle Controls
    +
Backup / Recovery
```

Never rely on a single protection mechanism.

---

# 48. Terraform Drift

Drift occurs when real infrastructure differs from Terraform's
expected configuration.

Example:

```
Terraform Code
    |
    ↓
Desired State

AWS
    |
    ↓
Actual State
```

If different:

```
Drift
```

---

# 49. Drift Detection

A scheduled Terraform plan can help identify drift.

Flow:

```
Schedule
    |
    ↓
terraform plan
    |
    ↓
Detect Unexpected Changes
    |
    ↓
Investigate
```

The team should distinguish:

```
Intentional Change
    vs
Unauthorized / Unexpected Drift
```

---

# 50. Manual Infrastructure Change

Scenario:

```
Engineer manually changes an AWS resource.
```

Result:

```
AWS
    |
    ↓
Different From Terraform
```

Next plan may show:

```
Update
    OR
Replace
    OR
Other Difference
```

The correct response is to determine whether the change belongs in
Terraform and then reconcile the source of truth.

---

# 51. Terraform Apply Failure

Terraform can fail after partially creating infrastructure.

Example:

```
Resource A → Created
Resource B → Created
Resource C → Failed
Resource D → Not Created
```

Terraform state records successful resource operations.

---

# 52. Recovery From Partial Apply

Do not immediately destroy everything.

First:

```
Read Error
    |
    ↓
Identify Failed Resource
    |
    ↓
Inspect Terraform State
    |
    ↓
Inspect AWS
    |
    ↓
Fix Root Cause
    |
    ↓
Run Plan
    |
    ↓
Apply Again
```

---

# 53. Terraform State After Failure

Terraform records successful changes in state.

Therefore:

```
Failed Apply
    |
    ↓
State
    |
    ↓
Existing Resources Tracked
```

The next plan should determine what remains to be created or changed.

---

# 54. State Corruption Scenario

If state becomes inconsistent:

```
Stop Automation
    |
    ↓
Protect State
    |
    ↓
Investigate
    |
    ↓
Backup / Recovery
    |
    ↓
Reconcile
    |
    ↓
Validate
```

Never manually edit state casually.

---

# 55. terraform import

If infrastructure exists outside Terraform but needs to become
managed by Terraform:

```
Existing AWS Resource
    |
    ↓
Terraform Import
    |
    ↓
Terraform State
    |
    ↓
Matching Configuration
```

The configuration must then accurately represent the resource.

---

# 56. Terraform Refresh

Modern Terraform workflows reconcile state and real infrastructure
during planning and other operations according to the selected
Terraform behavior.

The important operational principle is:

```
Code
    +
State
    +
Real Infrastructure
    |
    ↓
Terraform Plan
```

---

# 57. Terraform Locking

Concurrent operations against the same state can be dangerous.

Example:

```
Engineer A
    |
    ↓
terraform apply

Engineer B
    |
    ↓
terraform apply
```

Both modifying the same state can create operational risk.

Use backend-supported state locking and controlled CI/CD execution.

---

# 58. CI Concurrency

GitHub Actions can also control concurrent Terraform workflows.

Conceptually:

```
Pull Request A
    |
    ↓
Terraform Plan

Pull Request B
    |
    ↓
Terraform Plan
```

For production applies:

```
Production
    |
    ↓
One Controlled Apply
    |
    ↓
Next Apply
```

This prevents overlapping infrastructure changes.

---

# 59. Plan on Pull Request

A recommended PR process:

```
Pull Request
    |
    ↓
terraform fmt
    |
    ↓
terraform init
    |
    ↓
terraform validate
    |
    ↓
Security Scan
    |
    ↓
terraform plan
    |
    ↓
Plan Review
```

---

# 60. Apply After Merge

After approval and merge:

```
Main Branch
    |
    ↓
GitHub Actions
    |
    ↓
Terraform Init
    |
    ↓
Terraform Plan
    |
    ↓
Production Approval
    |
    ↓
Terraform Apply
```

---

# 61. Plan Comments

The pipeline can publish a summarized plan result to the pull
request.

Example information:

```
Additions
    +
Changes
    +
Deletions
    +
Replacement Operations
```

This helps reviewers understand infrastructure impact.

Sensitive values should not be exposed in pull-request comments.

---

# 62. Cost Impact

Infrastructure changes can affect cloud cost.

Examples:

```
New EC2 Instances
    +
Additional EKS Nodes
    +
Larger RDS Instances
    +
Additional NAT Gateways
    +
Additional Load Balancers
```

Plan review should consider cost impact.

---

# 63. Security Impact

A Terraform change can create security risks.

Example:

```
Security Group
    |
    ↓
0.0.0.0/0
    |
    ↓
Sensitive Port
```

The pipeline should detect and block insecure configurations according
to organizational policy.

---

# 64. Availability Impact

Terraform changes can affect availability.

Examples:

```
Replace Resource
    +
Modify Network
    +
Change Load Balancer
    +
Change EKS Node Group
    +
Modify Database
```

Review whether the plan causes:

```
Downtime
    +
Replacement
    +
Capacity Reduction
```

---

# 65. Production Infrastructure Change

Before applying a major change:

```
Plan
    |
    ↓
Review
    |
    ↓
Security
    |
    ↓
Cost
    |
    ↓
Availability
    |
    ↓
Approval
    |
    ↓
Apply
    |
    ↓
Validate
```

---

# 66. Infrastructure Validation

After Terraform apply:

```
Terraform Apply
    |
    ↓
AWS Infrastructure
    |
    ↓
Validation
```

Validate:

```
VPC
    +
Subnets
    +
Security Groups
    +
EKS
    +
ALB
    +
RDS
    +
ECR
    +
Required Outputs
```

---

# 67. EKS Validation

If Terraform creates EKS:

```
Terraform Apply
    |
    ↓
EKS Cluster
    |
    ↓
Cluster Validation
    |
    ↓
Node Validation
    |
    ↓
Kubernetes Validation
```

The application deployment itself may be handled separately by
GitOps.

---

# 68. ECR Validation

If Terraform creates ECR:

```
Terraform
    |
    ↓
ECR Repository
    |
    ↓
Validate Repository
    |
    ↓
Application Pipeline
```

Infrastructure provisioning and application image delivery remain
separate concerns.

---

# 69. ALB Validation

If Terraform provisions the ALB:

```
Terraform
    |
    ↓
ALB
    |
    ↓
Validate:
    |
    +--- DNS
    +--- Listener
    +--- Target Configuration
    +--- Security
```

---

# 70. RDS Validation

If Terraform provisions RDS:

```
Terraform
    |
    ↓
RDS
    |
    ↓
Validate:
    |
    +--- Status
    +--- Network
    +--- Security
    +--- Subnet Group
    +--- Backup Configuration
```

---

# 71. Terraform Outputs After Apply

After successful apply:

```
Terraform Outputs
    |
    +--- VPC ID
    +--- EKS Cluster
    +--- ALB DNS
    +--- ECR URL
    +--- RDS Endpoint
```

These outputs can help downstream processes.

---

# 72. Infrastructure Pipeline Security

The pipeline should implement:

```
Branch Protection
    +
Pull Request Reviews
    +
CODEOWNERS
    +
OIDC
    +
Least Privilege
    +
Protected Environments
    +
Secret Management
    +
Security Scanning
```

---

# 73. GitHub Permissions

The workflow should request only the permissions it needs.

Principle:

```
Minimum Required Access
```

Avoid unnecessary repository write access.

---

# 74. AWS IAM Security

The Terraform role should have carefully scoped permissions.

For example:

```
VPC Management
    +
EKS Management
    +
ECR Management
    +
ALB Management
    +
RDS Management
```

only where required.

---

# 75. Production Credentials

Never hardcode:

```
AWS Access Keys
    +
Database Passwords
    +
API Tokens
    +
Private Keys
```

inside:

```
.tf Files
    +
.tfvars
    +
Workflow Files
    +
Source Code
```

---

# 76. Sensitive Terraform Variables

Sensitive values should be marked appropriately and supplied through
approved secret-management mechanisms.

Example categories:

```
Database Password
    +
API Token
    +
Private Key
```

The goal is:

```
Secret
    |
    X
Git Repository
```

---

# 77. Terraform tfvars

Environment-specific values can be provided through:

```
Variables
    +
Environment Configuration
    +
Secure Variable Sources
```

Do not commit sensitive production values.

---

# 78. Production tfvars Security

Production configuration should be protected.

Example:

```
Production
    |
    ↓
Protected Environment
    |
    ↓
Approved Variables
    |
    ↓
Terraform
```

---

# 79. Infrastructure Separation of Duties

A strong enterprise workflow can separate:

```
Developer
    |
    ↓
Terraform Code

Reviewer
    |
    ↓
Plan Review

Security
    |
    ↓
Security Validation

Operations
    |
    ↓
Production Approval

Terraform
    |
    ↓
Apply
```

This creates accountability across the infrastructure lifecycle.

---

# 80. Terraform Plan Security

Terraform plans may contain sensitive information.

Therefore:

```
Do Not Publicly Expose Plans
    +
Restrict Artifact Access
    +
Avoid Sensitive Plan Comments
    +
Use Protected Workflows
```

---

# 81. Terraform Drift Monitoring

A scheduled workflow can periodically run:

```
terraform plan
```

The purpose is:

```
Detect Drift
    |
    ↓
Notify Team
    |
    ↓
Investigate
```

It should not automatically modify infrastructure unless explicitly
designed and approved to do so.

---

# 82. Scheduled Infrastructure Validation

A scheduled workflow can check:

```
Terraform Configuration
    +
State
    +
Infrastructure
```

Flow:

```
Schedule
    |
    ↓
Terraform Plan
    |
    ↓
Detect Differences
    |
    ↓
Notification
```

---

# 83. Terraform Version

Terraform versions should be controlled.

Example:

```
Terraform Configuration
    |
    ↓
Required Version
    |
    ↓
GitHub Actions
    |
    ↓
Matching Terraform Version
```

This improves reproducibility.

---

# 84. Terraform Provider Upgrades

Provider upgrades should be treated as controlled changes.

Flow:

```
Current Provider
    |
    ↓
Upgrade Proposal
    |
    ↓
CI
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Test
    |
    ↓
Production
```

Do not upgrade providers blindly in production.

---

# 85. Module Upgrades

Module changes can affect many resources.

Before upgrading:

```
Review Module Changes
    |
    ↓
terraform plan
    |
    ↓
Security Review
    |
    ↓
Test
    |
    ↓
Approve
    |
    ↓
Apply
```

---

# 86. Terraform State Migration

When backend or state structure changes:

```
Backup
    |
    ↓
Review Migration
    |
    ↓
Test
    |
    ↓
Apply Migration
    |
    ↓
Validate State
```

State migration should be treated as a high-risk infrastructure
operation.

---

# 87. Terraform Apply Failure Scenario

Question:

```
Terraform apply failed after provisioning half the infrastructure.
How do you recover?
```

### Strong Answer

I would not immediately destroy the infrastructure.

First I would inspect the Terraform error and identify the resource
that failed.

Then I would inspect:

```
Terraform State
    +
AWS Infrastructure
    +
Dependencies
    +
IAM Permissions
    +
Network Configuration
```

I would fix the root cause.

Then I would run:

```
terraform plan
```

to understand the remaining changes.

If the plan is correct, I would apply again.

Terraform should reconcile the existing managed resources with the
desired configuration instead of blindly recreating everything.

---

# 88. Terraform Plan Shows Unexpected Destroy

Question:

```
Terraform plan shows that an important production resource will
be destroyed. What do you do?
```

### Strong Answer

I would stop the deployment.

I would not apply the plan.

First I would identify why Terraform wants to destroy the resource.

I would check:

```
Configuration
    +
State
    +
Resource Attributes
    +
Provider Version
    +
Module Changes
    +
Recent Commits
```

Then I would determine whether the destruction is:

```
Expected
    OR
Unexpected
```

If unexpected, I would fix the configuration or state issue and
generate a new plan.

---

# 89. Terraform Drift Scenario

Question:

```
Someone manually changed an AWS security group outside Terraform.
What happens?
```

### Strong Answer

The infrastructure now differs from the Terraform configuration.

I would run a plan to identify the drift.

Then I would determine whether the manual change was:

```
Intentional
    OR
Unauthorized
```

If the configuration should remain the source of truth, I would
reconcile the infrastructure through Terraform rather than leaving
the manual change unmanaged.

---

# 90. Production Terraform Scenario

Question:

```
How would you safely deploy Terraform changes to production?
```

### Strong Answer

I would use a pull-request-based workflow.

The developer creates a Terraform change and opens a pull request.

GitHub Actions runs:

```
terraform fmt
    +
terraform init
    +
terraform validate
    +
Security Scan
    +
terraform plan
```

The plan is reviewed by the appropriate engineers.

After merge, the production workflow runs with protected environment
approval.

Only after approval does Terraform apply the reviewed infrastructure
change.

After apply, I validate the infrastructure and check the resulting
outputs and services.

---

# 91. Terraform CI Pipeline

The standard CI pipeline is:

```
Checkout
    |
    ↓
Terraform Setup
    |
    ↓
terraform fmt
    |
    ↓
terraform init
    |
    ↓
terraform validate
    |
    ↓
Security Scan
    |
    ↓
terraform plan
    |
    ↓
Plan Review
```

---

# 92. Terraform CD Pipeline

The production flow is:

```
Merge
    |
    ↓
Terraform Init
    |
    ↓
Terraform Plan
    |
    ↓
Approval
    |
    ↓
Terraform Apply
    |
    ↓
Infrastructure Validation
    |
    ↓
Monitoring
```

---

# 93. Complete End-to-End Flow

```
Developer
    |
    ↓
Terraform Code
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
terraform fmt
    |
    ↓
terraform init
    |
    ↓
terraform validate
    |
    ↓
Security Scan
    |
    ↓
terraform plan
    |
    ↓
Review
    |
    ↓
Merge
    |
    ↓
Production Approval
    |
    ↓
terraform apply
    |
    ↓
AWS
    |
    ↓
Infrastructure
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 94. Infrastructure and Application Separation

A strong enterprise architecture separates infrastructure provisioning
from application deployment.

Infrastructure:

```
Terraform
    |
    ↓
AWS
    |
    ↓
VPC
EKS
ECR
ALB
RDS
IAM
```

Application:

```
GitHub Actions
    |
    ↓
Docker
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 95. Terraform + GitHub Actions + EKS Architecture

```
┌──────────────────────────┐
│        Developer         │
└────────────┬─────────────┘
             │
             ↓
┌──────────────────────────┐
│          GitHub          │
│      Terraform Code      │
└────────────┬─────────────┘
             │
             ↓
┌──────────────────────────┐
│      GitHub Actions      │
│                          │
│ fmt                      │
│ validate                 │
│ security                 │
│ plan                     │
│ approval                 │
│ apply                    │
└────────────┬─────────────┘
             │
             ↓
┌──────────────────────────┐
│        AWS OIDC          │
│     Temporary Access     │
└────────────┬─────────────┘
             │
             ↓
┌──────────────────────────┐
│     Terraform State      │
│        Amazon S3         │
└────────────┬─────────────┘
             │
             ↓
┌──────────────────────────┐
│           AWS            │
│                          │
│ VPC                      │
│ EKS                      │
│ ECR                      │
│ ALB                      │
│ RDS                      │
│ IAM                      │
└──────────────────────────┘
```

---

# 96. Infrastructure Lifecycle

The complete lifecycle is:

```
Design
    |
    ↓
Terraform Code
    |
    ↓
Pull Request
    |
    ↓
Validation
    |
    ↓
Security
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
Apply
    |
    ↓
Validate
    |
    ↓
Monitor
    |
    ↓
Modify
    |
    ↓
Plan Again
```

---

# 97. Production Readiness Checklist

Before applying Terraform to production:

```
[ ] Pull request approved
[ ] Terraform formatted
[ ] Terraform initialized successfully
[ ] Terraform validation passed
[ ] Security scan passed
[ ] Terraform plan reviewed
[ ] No unexpected destroy operations
[ ] Cost impact reviewed
[ ] Security impact reviewed
[ ] Availability impact reviewed
[ ] Production approval completed
[ ] Remote state available
[ ] Correct AWS account selected
[ ] Correct environment selected
[ ] OIDC authentication verified
[ ] Required IAM permissions available
[ ] Apply completed
[ ] Infrastructure validated
[ ] Outputs verified
[ ] Rollback / recovery approach understood
```

---

# 98. Common Mistakes

## Mistake 1

Running `terraform apply` directly from a developer laptop for
production.

### Better

Use a controlled CI/CD workflow.

---

## Mistake 2

Skipping `terraform plan`.

### Better

Always review the proposed infrastructure changes.

---

## Mistake 3

Hardcoding AWS credentials.

### Better

Use GitHub OIDC and IAM roles.

---

## Mistake 4

Storing Terraform state locally for shared production infrastructure.

### Better

Use secure remote state.

---

## Mistake 5

Ignoring state locking.

### Better

Use backend-supported locking and controlled concurrency.

---

## Mistake 6

Automatically applying every pull request to production.

### Better

Use protected environments and approvals.

---

## Mistake 7

Ignoring unexpected resource destruction.

### Better

Stop and investigate the plan.

---

## Mistake 8

Manually editing Terraform state without understanding the impact.

### Better

Use supported Terraform state operations and backups.

---

## Mistake 9

Allowing unlimited IAM permissions.

### Better

Use least privilege.

---

## Mistake 10

Treating Terraform as an application deployment tool.

### Better

Use Terraform primarily for infrastructure provisioning and a
separate deployment mechanism such as GitOps for applications.

---

# 99. Real-World Enterprise Workflow

The enterprise workflow is:

```
Developer
    |
    ↓
Terraform Change
    |
    ↓
Pull Request
    |
    ↓
Code Review
    |
    ↓
GitHub Actions
    |
    +--- fmt
    +--- validate
    +--- security
    +--- plan
    |
    ↓
Plan Review
    |
    ↓
Merge
    |
    ↓
Protected Production Environment
    |
    ↓
Approval
    |
    ↓
Terraform Apply
    |
    ↓
AWS Infrastructure
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 100. Final Terraform CI/CD Principle

The most important principle is:

```
Infrastructure
    |
    ↓
Code
    |
    ↓
Validate
    |
    ↓
Secure
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Approve
    |
    ↓
Apply
    |
    ↓
Validate
    |
    ↓
Monitor
```

Terraform should make infrastructure changes:

```
Declarative
    +
Reproducible
    +
Reviewable
    +
Traceable
    +
Secure
    +
Controlled
```

The objective is not simply:

```
"Run Terraform from GitHub Actions."
```

The objective is:

```
"Create a secure, reviewable, controlled Infrastructure as Code
 delivery process that safely provisions and manages production
 infrastructure."
```
