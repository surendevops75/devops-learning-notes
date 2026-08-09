# Terraform Deployment in CD Pipelines

Terraform is an Infrastructure as Code (IaC) tool used to define, provision, modify, and manage infrastructure through declarative configuration files.

In a CD pipeline, Terraform can be used to automate infrastructure changes such as:

- VPC
- Subnets
- Security Groups
- EC2
- IAM
- S3
- RDS
- EKS
- Load Balancers
- Route53
- Other cloud resources

A typical flow is:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Terraform Format
        +-- Terraform Validate
        +-- Terraform Plan
        +-- Approval
        |
        ↓
    Terraform Apply
        |
        ↓
    AWS Infrastructure

---

# Why Terraform Is Used in CD

Without Infrastructure as Code, infrastructure changes may be performed manually.

Example:

    Engineer
       |
       +-- Create VPC
       +-- Create Subnets
       +-- Create Security Groups
       +-- Create EKS
       +-- Create IAM
       +-- Create Load Balancer

This can lead to:

- Manual errors
- Configuration drift
- Poor repeatability
- Difficult auditing
- Inconsistent environments

Terraform allows infrastructure to be managed as code.

---

# Infrastructure as Code

IaC means infrastructure configuration is stored in version-controlled files.

Example:

    GitHub Repository
         |
         +-- main.tf
         +-- variables.tf
         +-- outputs.tf
         +-- providers.tf
         +-- backend.tf
         |
         ↓
      Terraform
         |
         ↓
       AWS

---

# Terraform CD Mental Model

Remember:

    Terraform Code
         |
         ↓
    terraform init
         |
         ↓
    terraform validate
         |
         ↓
    terraform plan
         |
         ↓
    Approval
         |
         ↓
    terraform apply
         |
         ↓
    Cloud Infrastructure

---

# Terraform Configuration

Terraform configuration files normally use the `.tf` extension.

Example:

    main.tf
    variables.tf
    outputs.tf
    providers.tf
    backend.tf

Terraform loads all `.tf` files in the working directory.

---

# main.tf

`main.tf` commonly contains infrastructure resources.

Example:

    resource "aws_s3_bucket" "app" {
      bucket = var.bucket_name
    }

The exact organization depends on the project structure.

---

# providers.tf

The provider configuration tells Terraform which infrastructure platform it will manage.

Example:

    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 6.0"
        }
      }
    }

    provider "aws" {
      region = var.aws_region
    }

Provider versions should be managed deliberately.

---

# variables.tf

Variables make Terraform configurations reusable.

Example:

    variable "aws_region" {
      type        = string
      description = "AWS region"
    }

    variable "environment" {
      type        = string
      description = "Deployment environment"
    }

---

# terraform.tfvars

Variable values can be supplied through a `.tfvars` file.

Example:

    aws_region  = "ap-south-1"
    environment = "production"

Sensitive information should not be committed into Git.

---

# outputs.tf

Outputs expose useful information from Terraform.

Example:

    output "vpc_id" {
      value = aws_vpc.main.id
    }

Other resources or automation can use Terraform outputs.

---

# Terraform Backend

Terraform state needs to be stored somewhere.

For team environments, remote state is commonly preferred.

Example AWS architecture:

    GitHub Actions
         |
         ↓
      Terraform
         |
         ↓
    S3 Backend
         |
         ↓
    Terraform State

Remote state allows teams and CI/CD systems to work with a shared state.

---

# S3 Terraform Backend

Example concept:

    terraform {
      backend "s3" {
        bucket = "company-terraform-state"
        key    = "prod/network/terraform.tfstate"
        region = "ap-south-1"
        lock   = true
      }
    }

The exact backend configuration depends on the Terraform version and AWS architecture.

---

# Terraform State

Terraform state records the relationship between Terraform configuration and real infrastructure.

Conceptually:

    Terraform Code
          |
          ↓
    Terraform State
          |
          ↓
    AWS Infrastructure

Terraform uses state during planning and application.

---

# Why State Is Important

State allows Terraform to determine:

    What Terraform manages
    What currently exists
    What needs to change
    What needs to be created
    What needs to be destroyed

Without correctly managed state, infrastructure automation becomes unsafe.

---

# Never Store Production State Locally

For team-managed production infrastructure, avoid relying on a developer's local state file.

Better:

    GitHub Actions
         |
         ↓
    Remote Terraform State
         |
         ↓
    AWS

Remote state improves collaboration and consistency.

---

# Terraform Init

Command:

    terraform init

It initializes the Terraform working directory.

It can:

- Initialize the backend
- Download providers
- Download modules
- Prepare the working directory

Typical flow:

    Terraform Code
        |
        ↓
    terraform init
        |
        ↓
    Initialized

---

# Terraform Format

Command:

    terraform fmt

It formats Terraform configuration files.

CI can check formatting with:

    terraform fmt -check

This prevents formatting inconsistencies.

---

# Terraform Validate

Command:

    terraform validate

It validates the Terraform configuration syntax and internal consistency.

Typical CI flow:

    terraform fmt -check
        |
        ↓
    terraform validate
        |
        ↓
    PASS / FAIL

---

# Terraform Plan

Command:

    terraform plan

Terraform compares:

    Configuration
        +
    State
        +
    Current Infrastructure
        |
        ↓
    Proposed Changes

Example:

    + Create
    ~ Modify
    - Destroy

---

# Terraform Plan Output

Example concept:

    + aws_s3_bucket.app

        Will be created

    ~ aws_instance.app

        Will be updated

    - aws_security_group.old

        Will be destroyed

The plan should be reviewed before applying infrastructure changes.

---

# Terraform Apply

Command:

    terraform apply

Terraform executes the approved infrastructure changes.

Flow:

    Terraform Plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    terraform apply
        |
        ↓
    Infrastructure

---

# Terraform Plan and Apply

The recommended production flow is:

    Code Change
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

Avoid automatically applying unreviewed infrastructure changes to production.

---

# Terraform Destroy

Command:

    terraform destroy

This removes resources managed by Terraform.

This is powerful and potentially destructive.

Production destroy permissions should be heavily restricted.

---

# Terraform CD Pipeline

A common pipeline is:

    Pull Request
        |
        ↓
    terraform fmt -check
        |
        ↓
    terraform validate
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
    Approval
        |
        ↓
    terraform apply
        |
        ↓
    Infrastructure

---

# Pull Request Terraform Workflow

A Pull Request pipeline can run:

    Checkout
        |
        ↓
    Terraform Init
        |
        ↓
    Terraform Format Check
        |
        ↓
    Terraform Validate
        |
        ↓
    Terraform Plan
        |
        ↓
    Plan Result
        |
        ↓
    Pull Request Review

This gives reviewers visibility into infrastructure changes before merge.

---

# Terraform Apply Workflow

After approval:

    Merge
       |
       ↓
    Protected Branch
       |
       ↓
    Production Environment
       |
       ↓
    Approval
       |
       ↓
    Terraform Apply
       |
       ↓
    Infrastructure

---

# Terraform With GitHub Actions

A typical workflow is:

    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       +-- Terraform Init
       +-- Terraform Fmt
       +-- Terraform Validate
       +-- Terraform Plan
       |
       ↓
    Approval
       |
       ↓
    Terraform Apply
       |
       ↓
    AWS

---

# Terraform Authentication in GitHub Actions

For AWS deployments, GitHub Actions can use OIDC.

Flow:

    GitHub Actions
         |
         ↓
    GitHub OIDC
         |
         ↓
    AWS IAM Role
         |
         ↓
    AWS APIs
         |
         ↓
    Infrastructure

This avoids storing long-lived AWS access keys in GitHub Secrets.

---

# OIDC Security Model

The deployment role should be restricted by conditions such as:

    Repository
    Organization
    Branch
    Environment
    Workflow

Conceptually:

    GitHub Repository
          |
          ↓
        OIDC
          |
          ↓
    Restricted IAM Role
          |
          ↓
         AWS

Use least privilege.

---

# Terraform AWS Permissions

Terraform requires permissions to create and modify the resources it manages.

For example, an EKS infrastructure pipeline may require permissions related to:

    EC2
    VPC
    IAM
    EKS
    ECR
    ELB
    S3
    RDS
    Route53

The exact permissions should be limited to what the Terraform configuration actually needs.

---

# Terraform Modules

Modules allow reusable infrastructure components.

Example:

    Terraform Root Module
          |
          +-- VPC Module
          +-- Security Module
          +-- EKS Module
          +-- ALB Module
          +-- RDS Module
          +-- S3 Module

This is useful for large infrastructure projects.

---

# Example Terraform Module Structure

A module can contain:

    modules/
      |
      └── vpc/
          |
          ├── main.tf
          ├── variables.tf
          └── outputs.tf

Root configuration:

    main.tf
    variables.tf
    outputs.tf

---

# Terraform Module Usage

Example:

    module "vpc" {
      source = "./modules/vpc"

      environment = var.environment
      cidr_block  = var.vpc_cidr
    }

The root module passes values into the reusable module.

---

# Infrastructure Layering

A large platform may separate infrastructure:

    00-vpc
       |
       ↓
    10-security
       |
       ↓
    20-bastion
       |
       ↓
    30-security-rules
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

The exact numbering and structure depends on the project.

---

# Terraform Dependency

Terraform builds a dependency graph.

Example:

    VPC
     |
     ↓
    Subnets
     |
     ↓
    EKS
     |
     ↓
    Kubernetes Resources

Terraform determines resource creation order from dependencies.

---

# Explicit Dependency

Terraform supports:

    depends_on

Example:

    resource "example_resource" "app" {

      depends_on = [
        example_resource.network
      ]
    }

Use explicit dependencies when Terraform cannot infer the relationship automatically.

---

# Terraform Plan in CI

A CI pipeline should generate a plan before apply.

Example:

    terraform init

    terraform fmt -check

    terraform validate

    terraform plan

The plan should be available for review.

---

# Terraform Plan Artifact

A pipeline can save the plan:

    terraform plan -out=tfplan

Then apply the reviewed plan:

    terraform apply tfplan

This can help ensure that the exact planned changes are what gets applied.

---

# Plan Artifact Security

Terraform plans can contain sensitive information.

Therefore:

    Do not expose plans publicly
    Protect CI artifacts
    Restrict access
    Follow organization security policies

Treat Terraform plan artifacts as potentially sensitive.

---

# Terraform State Locking

State locking prevents multiple Terraform operations from modifying the same state simultaneously.

Conceptually:

    Pipeline A
       |
       ↓
    State Lock
       |
       ↓
    Terraform State

    Pipeline B
       |
       ↓
    Wait / Fail

This protects against concurrent state modifications.

---

# Terraform Concurrency

A production Terraform workflow should prevent two apply jobs from modifying the same state simultaneously.

For example:

    Production Apply
          |
          ↓
      Concurrency
          |
          ↓
    One Apply At A Time

GitHub Actions concurrency controls can be used in addition to Terraform's state locking mechanisms.

---

# Terraform Drift

Drift occurs when infrastructure changes outside Terraform.

Example:

    Terraform
        |
        ↓
    AWS Infrastructure

Then someone manually changes AWS:

    Engineer
        |
        ↓
    AWS Console
        |
        ↓
    Infrastructure Changed

Now:

    Terraform Configuration
        ≠
    Real Infrastructure

This is configuration drift.

---

# Detecting Drift

Run:

    terraform plan

Terraform may detect changes that differ from the desired configuration.

Flow:

    Terraform Plan
        |
        ↓
    Compare
        |
        +-- No Changes
        |
        +-- Drift Detected

---

# Drift Prevention

Best practice:

    Git
      |
      ↓
    Terraform
      |
      ↓
    AWS

Avoid unmanaged manual changes.

If an emergency manual change is required, bring the infrastructure back under controlled Terraform management.

---

# Terraform Import

If existing infrastructure needs to be managed by Terraform, it can be imported.

Example:

    terraform import <resource> <id>

Import does not automatically create a perfect Terraform configuration.

The configuration should be reviewed and aligned with the imported resource.

---

# Terraform State Commands

Common commands include:

    terraform state list

    terraform state show <resource>

    terraform state pull

    terraform state mv

    terraform state rm

State manipulation should be performed carefully, especially in production.

---

# Terraform Workspace

Terraform workspaces can separate state for different configurations.

Example:

    dev
    qa
    prod

However, workspaces are not always the best environment separation mechanism.

For larger production environments, separate state configurations or directories are often easier to reason about.

---

# Environment Separation

Possible structures include:

    environments/
      |
      ├── dev/
      ├── qa/
      ├── uat/
      └── prod/

Each environment can have its own configuration and state.

---

# Terraform Environment Pipeline

Example:

    DEV
      |
      ↓
    Terraform Plan
      |
      ↓
    Terraform Apply
      |
      ↓
    QA
      |
      ↓
    Terraform Plan
      |
      ↓
    Terraform Apply
      |
      ↓
    UAT
      |
      ↓
    Approval
      |
      ↓
    PROD

---

# Terraform Variables by Environment

Example:

    dev.tfvars
    qa.tfvars
    uat.tfvars
    prod.tfvars

Deployment:

    terraform plan \
      -var-file=prod.tfvars

This allows the same Terraform configuration to be parameterized.

---

# Terraform Secrets

Do not place sensitive production credentials directly into:

    .tf
    .tfvars
    GitHub Repository

Use appropriate secret-management mechanisms.

Examples:

    AWS Secrets Manager
    GitHub Secrets
    GitHub OIDC
    Environment Variables
    External Secret Systems

The appropriate choice depends on the architecture.

---

# Terraform Sensitive Variables

Example:

    variable "db_password" {
      type      = string
      sensitive = true
    }

This tells Terraform to treat the variable as sensitive in appropriate output contexts.

It does not mean the value is automatically stored securely everywhere.

State and pipeline logs still need protection.

---

# Terraform Remote State Security

Remote state can contain sensitive infrastructure information.

Protect it using:

    Encryption
    Access Control
    IAM
    Versioning
    Restricted Bucket Access
    Appropriate State Locking

Never make production Terraform state publicly accessible.

---

# Terraform Backend Architecture

Example:

    GitHub Actions
         |
         ↓
    Terraform
         |
         ↓
    S3 Backend
         |
         +-- State
         |
         +-- Access Control
         |
         +-- Encryption
         |
         ↓
    AWS Infrastructure

---

# Terraform CD and Approval

Production infrastructure changes should normally have controlled approval.

Example:

    Pull Request
        |
        ↓
    Terraform Plan
        |
        ↓
    Code Review
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
    Terraform Apply

---

# Terraform Change Review

Review:

    Resources Added
    Resources Modified
    Resources Destroyed
    IAM Changes
    Network Changes
    Security Changes
    Database Changes
    EKS Changes

Special attention should be given to:

    - Destroy
    - IAM
    - Security Groups
    - Network
    - Database
    - Production Resources

---

# Terraform Plan Symbols

Terraform commonly uses:

    + Create
    ~ Update in-place
    - Destroy
    -/+ Replace
    <= Read

The plan should be reviewed before applying changes.

---

# Resource Replacement

Some Terraform changes cannot be performed in place.

Terraform may show:

    -/+ resource

Meaning:

    Destroy Existing
        |
        ↓
    Create New

This can cause downtime depending on the resource.

Always review replacement operations carefully.

---

# Terraform Lifecycle

Terraform supports lifecycle controls.

Examples include:

    create_before_destroy
    prevent_destroy
    ignore_changes

These should be used intentionally.

---

# create_before_destroy

This lifecycle behavior can create the new resource before destroying the old one where supported.

Conceptually:

    Old Resource
        |
        ↓
    New Resource Created
        |
        ↓
    Old Resource Removed

This can reduce downtime for suitable resources.

---

# prevent_destroy

Example:

    lifecycle {
      prevent_destroy = true
    }

This can protect critical resources from accidental Terraform destruction.

Use it carefully because it can also prevent legitimate changes or cleanup.

---

# ignore_changes

`ignore_changes` can tell Terraform to ignore selected externally managed attributes.

This should be used carefully because excessive use can hide configuration drift.

---

# Terraform Idempotency

Terraform is designed around desired state.

If you run:

    terraform apply

and there are no changes:

    No changes

Running the same configuration again should not recreate resources unnecessarily.

---

# Terraform CD Idempotency

A good pipeline should be safe to rerun.

Example:

    Pipeline
       |
       ↓
    Terraform Apply
       |
       ↓
    Infrastructure

Run again:

    Terraform Apply
       |
       ↓
    No Changes

This is an important IaC characteristic.

---

# Terraform and EKS

Terraform can provision EKS infrastructure.

Typical architecture:

    Terraform
       |
       +-- VPC
       +-- Subnets
       +-- IAM
       +-- Security Groups
       +-- EKS
       +-- Node Groups
       +-- Load Balancer Components
       |
       ↓
    AWS EKS

After the cluster is created, application delivery can use:

    Helm
    ArgoCD
    kubectl
    Other CD Tools

---

# Terraform EKS Pipeline

Example:

    GitHub
       |
       ↓
    GitHub Actions
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
    VPC
       |
       ↓
    EKS
       |
       ↓
    Application CD

Terraform handles infrastructure.

Helm or ArgoCD can handle application deployment.

---

# Infrastructure vs Application CD

Infrastructure CD:

    Terraform
       |
       ↓
    VPC
    EKS
    IAM
    RDS
    ALB
    S3

Application CD:

    Helm / ArgoCD
       |
       ↓
    Deployment
    Service
    Ingress
    ConfigMap
    Application Pods

Keeping these responsibilities clear improves pipeline design.

---

# Terraform and Helm Together

A platform may use:

    Terraform
        |
        +-- VPC
        +-- EKS
        +-- IAM
        +-- ECR
        +-- ALB Infrastructure
        |
        ↓
    Helm
        |
        +-- Deployment
        +-- Service
        +-- Ingress
        +-- ConfigMap
        |
        ↓
    Application

---

# Terraform and ArgoCD Together

Another architecture:

    Terraform
        |
        ↓
    EKS Infrastructure
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes Applications

Terraform provisions the platform.

ArgoCD continuously deploys applications from Git.

---

# Terraform Module Example

A production infrastructure repository may contain:

    modules/
      |
      ├── vpc/
      ├── security-group/
      ├── iam/
      ├── ecr/
      ├── eks/
      ├── alb/
      ├── rds/
      └── s3/

Environment:

    environments/
      |
      ├── dev/
      ├── qa/
      ├── uat/
      └── prod/

This structure supports reuse and environment separation.

---

# Terraform Pipeline Structure

A mature Terraform repository can use:

    terraform/
      |
      ├── modules/
      |
      └── environments/
          |
          ├── dev/
          ├── qa/
          ├── uat/
          └── prod/

Each environment can have its own:

    backend
    variables
    providers
    Terraform state

---

# Terraform CI Pipeline

A Terraform CI pipeline can be:

    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    terraform fmt -check
        |
        ↓
    terraform init
        |
        ↓
    terraform validate
        |
        ↓
    terraform plan
        |
        ↓
    Review

---

# Terraform CD Pipeline

A Terraform CD pipeline can be:

    Merge
       |
       ↓
    Protected Environment
       |
       ↓
    Approval
       |
       ↓
    terraform init
       |
       ↓
    terraform plan
       |
       ↓
    terraform apply
       |
       ↓
    Validate Infrastructure
       |
       ↓
    Success

---

# GitHub Actions Terraform Example

Conceptual CI workflow:

    name: Terraform Plan

    on:
      pull_request:

    jobs:

      terraform:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Terraform
            uses: hashicorp/setup-terraform@v4

          - name: Terraform Init
            run: terraform init

          - name: Terraform Format
            run: terraform fmt -check

          - name: Terraform Validate
            run: terraform validate

          - name: Terraform Plan
            run: terraform plan

---

# Terraform Apply Example

Conceptual deployment workflow:

    name: Terraform Apply

    on:
      workflow_dispatch:

    jobs:

      apply:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Terraform
            uses: hashicorp/setup-terraform@v4

          - name: Configure AWS Credentials
            uses: aws-actions/configure-aws-credentials@v6

          - name: Terraform Init
            run: terraform init

          - name: Terraform Apply
            run: terraform apply -auto-approve

For production, use appropriate environment protection and approval controls instead of blindly allowing unrestricted automatic apply.

---

# Terraform Auto Approve

Command:

    terraform apply -auto-approve

This skips the interactive confirmation.

It is useful for controlled automation.

However, production pipelines should rely on:

    Reviewed Plan
    Protected Environment
    Approval
    Controlled Apply

rather than using `-auto-approve` without safeguards.

---

# Terraform Plan Artifact Pipeline

A stronger pattern is:

    Pull Request
        |
        ↓
    Terraform Plan
        |
        ↓
    Save Plan
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    Protected Environment
        |
        ↓
    Apply Approved Plan

This can improve consistency between review and execution.

---

# Terraform Plan File

Example:

    terraform plan -out=tfplan

Then:

    terraform apply tfplan

The plan file should be protected because it may contain sensitive information.

---

# Terraform Lock File

Terraform creates:

    .terraform.lock.hcl

This records selected provider versions and checksums.

It should normally be committed to version control.

This helps make provider selection more reproducible.

---

# Terraform Provider Version

Provider versions should be constrained.

Example:

    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 6.0"
      }
    }

Avoid allowing uncontrolled provider upgrades in production.

Provider upgrades should be tested.

---

# Terraform Version

Terraform itself should also be controlled.

Example:

    terraform {
      required_version = ">= 1.9.0, < 2.0.0"
    }

The exact constraint should match the organization's tested version.

---

# Terraform Version Management

CI should use a known Terraform version.

Example:

    GitHub Actions
        |
        ↓
    Setup Terraform
        |
        ↓
    Known Version
        |
        ↓
    Terraform Pipeline

This reduces differences between developer machines and CI runners.

---

# Terraform Security Scanning

Terraform code can be scanned for security and policy issues.

Possible categories:

    IAM
    Public Resources
    Encryption
    Network Security
    Storage Security
    Misconfiguration

Security scanning can be added before the plan or apply stage.

---

# Terraform DevSecOps Pipeline

Example:

    Pull Request
        |
        ↓
    Terraform Format
        |
        ↓
    Terraform Validate
        |
        ↓
    IaC Security Scan
        |
        ↓
    Terraform Plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Terraform Apply

This brings security into infrastructure delivery.

---

# Terraform and SonarQube

SonarQube is primarily focused on code quality.

For Terraform, organizations may use specialized IaC security and policy tools where appropriate.

The pipeline should use the tool that matches the type of validation required.

---

# Terraform Deployment Failure

If Terraform apply fails:

    Terraform Apply
         |
         ↓
        FAIL
         |
         ↓
    Inspect Error
         |
         ↓
    Inspect State
         |
         ↓
    Check AWS Resource
         |
         ↓
    Fix Configuration
         |
         ↓
    terraform plan
         |
         ↓
    Apply Again

Do not immediately destroy and recreate everything.

---

# Partial Terraform Apply

Terraform may successfully create some resources before another resource fails.

Example:

    VPC
      |
      ↓
    Subnets
      |
      ↓
    Security Groups
      |
      ↓
    EKS
      |
      X
    Failed

Terraform state should reflect resources that were successfully created.

The next plan determines what remains to be done.

---

# Recovering From Partial Apply

Recommended approach:

    1. Read the Terraform error
    2. Inspect Terraform state
    3. Inspect real infrastructure
    4. Run terraform plan
    5. Identify remaining changes
    6. Fix root cause
    7. Apply again
    8. Validate infrastructure

Avoid manually deleting resources unless there is a clear reason and the Terraform state is considered afterward.

---

# Terraform State Recovery

If state becomes inconsistent or corrupted, recovery should be handled carefully.

Possible tools include:

    terraform state list
    terraform state show
    terraform state pull

For serious state problems:

    Backup State
    Understand Current Infrastructure
    Review State
    Use State Operations Carefully
    Validate With Plan

Production state operations should be performed by experienced engineers.

---

# Terraform State Backup

Remote state should have appropriate:

    Versioning
    Encryption
    Access Control
    Backup / Recovery
    Audit Logging

The exact implementation depends on the backend architecture.

---

# Terraform Concurrency Problem

Bad:

    Pipeline A
       |
       ↓
    terraform apply
       |
       ↓
    State

    Pipeline B
       |
       ↓
    terraform apply
       |
       ↓
    Same State

This can create conflicts.

Better:

    Pipeline
       |
       ↓
    Concurrency Control
       |
       ↓
    Terraform State Lock
       |
       ↓
    One Controlled Apply

---

# Production Terraform Workflow

A mature workflow is:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Terraform Checks
        |
        +-- fmt
        +-- validate
        +-- security
        +-- plan
        |
        ↓
    Code Review
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
    Terraform Apply
        |
        ↓
    Infrastructure Validation
        |
        ↓
    Success

---

# Terraform Best Practices

- Store Terraform code in Git
- Use remote state
- Protect state
- Use state locking
- Use modules for reusable infrastructure
- Pin Terraform versions
- Pin provider versions
- Commit `.terraform.lock.hcl`
- Run `terraform fmt`
- Run `terraform validate`
- Run `terraform plan`
- Review production plans
- Protect production apply
- Use OIDC for AWS authentication
- Use least-privilege IAM
- Avoid hardcoded secrets
- Detect infrastructure drift
- Avoid manual production changes
- Test infrastructure changes
- Maintain rollback/recovery procedures
- Protect Terraform state
- Control Terraform concurrency

---

# Terraform Anti-Pattern

## Manual Infrastructure Changes

Bad:

    Engineer
       |
       ↓
    AWS Console
       |
       ↓
    Production

Better:

    Git
      |
      ↓
    Terraform
      |
      ↓
    Production

---

# Terraform Anti-Pattern

## Hardcoded Credentials

Bad:

    access_key = "xxxxx"
    secret_key = "xxxxx"

Better:

    GitHub OIDC
        |
        ↓
    AWS IAM Role
        |
        ↓
    Terraform

---

# Terraform Anti-Pattern

## Local State in Production

Bad:

    Developer Laptop
        |
        ↓
    terraform.tfstate

Better:

    Terraform
        |
        ↓
    Secure Remote Backend

---

# Terraform Anti-Pattern

## Direct Production Apply From Any Branch

Bad:

    Feature Branch
        |
        ↓
    Production
        |
        ↓
    terraform apply

Better:

    Pull Request
        |
        ↓
    Review
        |
        ↓
    Protected Branch
        |
        ↓
    Production Approval
        |
        ↓
    Terraform Apply

---

# Terraform Anti-Pattern

## Ignoring Terraform Plan

Bad:

    terraform apply

without reviewing the proposed infrastructure changes.

Better:

    terraform plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Apply

---

# Terraform Anti-Pattern

## Using Broad IAM Permissions

Bad:

    Terraform Role
        |
        ↓
    AdministratorAccess

Better:

    Terraform Role
        |
        ↓
    Required Permissions Only

The exact IAM policy depends on the resources Terraform manages.

---

# Terraform Scenario

## Terraform Apply Failed After Creating Half the Infrastructure. What Would You Do?

I would not immediately destroy the infrastructure.

I would:

    1. Read the Terraform error
    2. Check Terraform state
    3. Check AWS resources
    4. Run terraform plan
    5. Identify incomplete resources
    6. Fix the root cause
    7. Run terraform plan again
    8. Apply the remaining changes
    9. Validate infrastructure

Terraform is designed to manage incremental changes, so recovery should start with state and plan analysis.

---

# Terraform Scenario

## Two Engineers Run Terraform Apply at the Same Time. What Happens?

Terraform state locking is designed to prevent conflicting state modifications.

The second operation may have to wait or fail depending on the backend and lock behavior.

In CI/CD, I would also implement pipeline concurrency controls.

Example:

    Production Terraform
           |
           ↓
       One Apply
           |
           ↓
      State Lock

---

# Terraform Scenario

## How Would You Detect Infrastructure Drift?

I would run:

    terraform plan

Terraform compares the desired configuration with the current infrastructure and state.

If someone manually changed AWS resources, Terraform may show the difference.

---

# Terraform Scenario

## Someone Manually Changed a Production Security Group. What Would You Do?

First I would identify:

    What Changed
    Why It Changed
    Who Changed It
    Current Terraform Configuration
    Current AWS Configuration

Then:

    terraform plan

If the manual change is not desired, Terraform can restore the configuration.

If the change is legitimate, I would update the Terraform code and bring the infrastructure back under controlled management.

---

# Terraform Scenario

## How Would You Secure Terraform State?

I would:

    1. Store state remotely
    2. Enable encryption
    3. Restrict access
    4. Use IAM
    5. Use state locking
    6. Enable versioning / recovery
    7. Protect backend credentials
    8. Audit access
    9. Never expose state publicly

---

# Terraform Scenario

## How Would You Deploy Infrastructure to DEV, QA, UAT, and PROD?

I would separate environments while reusing modules.

Example:

    modules/
      |
      +-- vpc
      +-- eks
      +-- iam
      +-- rds

    environments/
      |
      +-- dev
      +-- qa
      +-- uat
      +-- prod

Each environment can have separate:

    State
    Variables
    Backend
    Permissions

---

# Terraform Scenario

## How Would You Secure Production Terraform Apply?

I would use:

    Protected Branch
        |
        ↓
    Pull Request
        |
        ↓
    Terraform Plan
        |
        ↓
    Code Review
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    OIDC
        |
        ↓
    Least-Privilege IAM Role
        |
        ↓
    Terraform Apply

---

# Terraform Scenario

## How Would You Deploy EKS Using Terraform?

I would provision:

    VPC
      |
      ↓
    Subnets
      |
      ↓
    Security Groups
      |
      ↓
    IAM
      |
      ↓
    EKS Control Plane
      |
      ↓
    Node Groups
      |
      ↓
    Supporting AWS Resources

Then application deployment can use:

    Helm
    ArgoCD
    kubectl

---

# Terraform Scenario

## Terraform or Helm: Which One Would You Use?

Terraform:

    Infrastructure

Examples:

    VPC
    EKS
    IAM
    RDS
    S3
    ECR
    AWS Load Balancer Infrastructure

Helm:

    Kubernetes Application Resources

Examples:

    Deployment
    Service
    ConfigMap
    Ingress
    HPA

The tools solve different problems and can be used together.

---

# Terraform Scenario

## Terraform or ArgoCD: Which One Would You Use?

Terraform:

    Infrastructure Provisioning

ArgoCD:

    Kubernetes Application GitOps Deployment

Example:

    Terraform
       |
       ↓
    EKS Cluster
       |
       ↓
    ArgoCD
       |
       ↓
    Applications

---

# Terraform Scenario

## Why Use Terraform Modules?

Modules provide:

    Reusability
    Standardization
    Maintainability
    Consistency

Example:

    VPC Module

can be reused for:

    DEV
    QA
    UAT
    PROD

with different variables.

---

# Terraform Scenario

## How Would You Prevent Accidental Production Destruction?

I would use multiple layers:

    Protected Repository
        +
    Pull Request Review
        +
    Terraform Plan Review
        +
    Protected Environment
        +
    Manual Approval
        +
    Least-Privilege IAM
        +
    prevent_destroy Where Appropriate

This reduces the risk of destructive changes.

---

# Terraform Scenario

## Terraform Plan Shows a Resource Will Be Replaced. What Would You Do?

I would inspect the plan carefully.

Look for:

    -/+

This means Terraform intends to destroy and recreate the resource.

I would determine:

    Why Replacement Is Required
    Whether It Causes Downtime
    Whether There Is a Safer Configuration
    Whether create_before_destroy Is Applicable

Then review the change before applying.

---

# Terraform Scenario

## How Would You Make Terraform Pipelines Reproducible?

I would control:

    Terraform Version
    Provider Versions
    Provider Lock File
    Module Versions
    Variables
    Backend
    Environment Configuration

I would also commit:

    .terraform.lock.hcl

and use controlled CI environments.

---

# Terraform Scenario

## How Would You Design Terraform for a Large Organization?

I would use:

    Reusable Modules
        +
    Environment Separation
        +
    Remote State
        +
    State Locking
        +
    Pull Request Review
        +
    Automated Plan
        +
    Protected Apply
        +
    Least-Privilege IAM
        +
    Security Scanning
        +
    Auditability

---

# Terraform Interview Questions

## Basic

1. What is Terraform?
2. What is Infrastructure as Code?
3. What is Terraform state?
4. What is a Terraform provider?
5. What is a Terraform resource?
6. What is a Terraform module?
7. What is `terraform init`?
8. What is `terraform validate`?
9. What is `terraform plan`?
10. What is `terraform apply`?
11. What is `terraform destroy`?
12. What is `terraform fmt`?
13. What is Terraform backend?
14. What is remote state?
15. What is state locking?

---

# Intermediate Interview Questions

16. How do you use Terraform in CI/CD?

17. How do you manage Terraform state?

18. How do you secure Terraform state?

19. How do you manage multiple environments?

20. How do you use Terraform modules?

21. How do you detect drift?

22. How do you recover from a failed Terraform apply?

23. How do you authenticate Terraform to AWS?

24. Why use GitHub OIDC with Terraform?

25. How do you manage Terraform secrets?

26. What is `.terraform.lock.hcl`?

27. How do you prevent concurrent Terraform applies?

28. What is the difference between plan and apply?

29. What does `-auto-approve` do?

30. What happens during `terraform init`?

---

# Advanced Interview Questions

31. Design an enterprise Terraform CI/CD pipeline.

32. How would you securely deploy Terraform to AWS from GitHub Actions?

33. How would you manage Terraform state for multiple environments?

34. How would you recover from Terraform state corruption?

35. How would you handle infrastructure drift?

36. How would you recover from a partially completed Terraform apply?

37. How would you prevent accidental production destruction?

38. How would you design reusable Terraform modules?

39. How would you manage Terraform provider upgrades?

40. How would you separate infrastructure CD from application CD?

41. How would you integrate Terraform with EKS?

42. How would you combine Terraform, Helm, and ArgoCD?

---

# Complete Terraform CD Architecture

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
        +-- Terraform Fmt
        +-- Terraform Validate
        +-- Security Scan
        +-- Terraform Plan
        |
        ↓
    Code Review
        |
        ↓
    Merge
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    GitHub OIDC
        |
        ↓
    AWS IAM Role
        |
        ↓
    Terraform Apply
        |
        ↓
    Remote State
        |
        ↓
    AWS Infrastructure
        |
        +-- VPC
        +-- IAM
        +-- ECR
        +-- EKS
        +-- RDS
        +-- ALB
        +-- S3
        |
        ↓
    Infrastructure Validation
        |
        ↓
    Application CD
        |
        ↓
    Helm / ArgoCD
        |
        ↓
    Kubernetes Applications

---

# Terraform + Helm + ArgoCD

A complete DevOps platform can use all three tools:

    Terraform
        |
        ↓
    AWS Infrastructure
        |
        +-- VPC
        +-- EKS
        +-- IAM
        +-- ECR
        |
        ↓
    Helm
        |
        ↓
    Kubernetes Application Packaging
        |
        ↓
    ArgoCD
        |
        ↓
    GitOps Deployment
        |
        ↓
    EKS Applications

Each tool has a clear responsibility.

---

# Responsibility Separation

Terraform:

    Provision Infrastructure

Helm:

    Package Kubernetes Applications

ArgoCD:

    Continuously Deploy Kubernetes Applications From Git

GitHub Actions:

    Automate CI/CD Workflows

ECR:

    Store Container Images

EKS:

    Run Kubernetes Applications

This separation creates a clean DevOps architecture.

---

# Final Terraform CD Mental Model

Remember:

    Git
      |
      ↓
    Terraform Code
      |
      ↓
    fmt
      |
      ↓
    validate
      |
      ↓
    security checks
      |
      ↓
    plan
      |
      ↓
    review
      |
      ↓
    approval
      |
      ↓
    apply
      |
      ↓
    AWS Infrastructure
      |
      ↓
    EKS
      |
      ↓
    Helm / ArgoCD
      |
      ↓
    Kubernetes Application

---

# Final Terraform Checklist

    [ ] Terraform version controlled
    [ ] Provider versions controlled
    [ ] .terraform.lock.hcl committed
    [ ] Terraform code stored in Git
    [ ] Remote state configured
    [ ] State access restricted
    [ ] State locking configured
    [ ] terraform fmt passes
    [ ] terraform validate passes
    [ ] Security checks pass
    [ ] terraform plan reviewed
    [ ] Production approval configured
    [ ] OIDC authentication configured
    [ ] IAM permissions follow least privilege
    [ ] No hardcoded secrets
    [ ] Environment separation implemented
    [ ] Modules used appropriately
    [ ] Drift monitored
    [ ] Rollback / recovery procedure documented
    [ ] Terraform concurrency controlled
    [ ] Infrastructure changes audited

---

# Final Concept

Terraform-based CD means infrastructure changes are delivered through a controlled, repeatable, version-controlled pipeline instead of unmanaged manual changes.

The core process is:

    Infrastructure Code
          |
          ↓
    Pull Request
          |
          ↓
    Validation
          |
          ↓
    Security Checks
          |
          ↓
    Terraform Plan
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
    AWS Infrastructure
          |
          ↓
    Validation

The key principle is:

    Infrastructure should be treated as code.

Terraform provides the foundation for repeatable infrastructure provisioning, while Helm and ArgoCD can take responsibility for deploying and continuously managing applications running on Kubernetes.