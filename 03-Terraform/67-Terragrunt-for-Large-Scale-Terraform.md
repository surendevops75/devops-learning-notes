# Terragrunt for Large-Scale Terraform

## Introduction

Terraform works very well for managing infrastructure.

However, as organizations grow, Terraform projects become increasingly complex.

Common problems:

```text
Duplicate Code

Duplicate Backend Configurations

Duplicate Provider Configurations

Multiple AWS Accounts

Multiple Environments

Hundreds Of Terraform Modules
```

---

Consider:

```text
DEV

QA

UAT

PROD
```

---

Each environment requires:

```text
VPC

Security Groups

ALB

EKS

Databases
```

---

Without proper management, engineers often duplicate Terraform code.

---

# The Problem

Example Structure:

```text
dev/
 ├── vpc
 ├── sg
 └── eks

qa/
 ├── vpc
 ├── sg
 └── eks

prod/
 ├── vpc
 ├── sg
 └── eks
```

---

Problems:

```text
Code Duplication

Maintenance Difficulty

Configuration Drift

Human Errors
```

---

# What is Terragrunt?

Terragrunt is:

```text
A Thin Wrapper Around Terraform
```

---

Created to solve:

```text
Terraform Reusability

Code Duplication

Environment Management
```

---

Terragrunt itself does not replace Terraform.

---

Instead:

```text
Terragrunt Uses Terraform Internally
```

---

# Relationship

```text
Terragrunt
      ↓
Terraform
      ↓
AWS
```

---

# Why Terragrunt?

Main Goal:

```text
DRY Principle
```

---

DRY means:

```text
Don't Repeat Yourself
```

---

# Without Terragrunt

100 Environments:

```text
100 Backend Blocks

100 Provider Blocks

100 Variable Files
```

---

Large amount of duplicated code.

---

# With Terragrunt

Common configuration:

```text
Written Once
```

---

Reused everywhere.

---

# Typical Terraform Structure

```text
modules/
├── vpc
├── sg
├── alb
└── eks
```

---

Environment Structure:

```text
dev/
qa/
prod/
```

---

Repeated configurations:

```text
Backend

Provider

Tags

Remote State
```

---

# Terragrunt Solution

Terragrunt centralizes:

```text
Backend Config

Provider Config

Common Variables

Environment Configuration
```

---

# Terragrunt Architecture

```text
Terragrunt
      ↓
Terraform Modules
      ↓
AWS Resources
```

---

# Common Folder Structure

```text
live/
├── dev
│   ├── vpc
│   ├── sg
│   └── eks
│
├── qa
│   ├── vpc
│   ├── sg
│   └── eks
│
└── prod
    ├── vpc
    ├── sg
    └── eks
```

---

Terraform modules:

```text
modules/
├── vpc
├── sg
├── alb
└── eks
```

---

# What is terragrunt.hcl?

Terragrunt uses:

```text
terragrunt.hcl
```

instead of:

```text
main.tf
```

for Terragrunt-specific configuration.

---

Example

```hcl
terraform {
  source = "../../modules/vpc"
}
```

---

Meaning:

```text
Use Existing VPC Module
```

---

# Remote State Simplification

Without Terragrunt:

Every project needs:

```hcl
backend "s3" {
}
```

---

Repeated many times.

---

With Terragrunt:

```text
Centralized Backend Configuration
```

---

Defined once.

---

Used everywhere.

---

# Provider Simplification

Without Terragrunt:

```hcl
provider "aws" {
}
```

appears in many projects.

---

With Terragrunt:

```text
Shared Provider Configuration
```

---

# Multi-Environment Example

Single Module:

```text
VPC Module
```

---

Used by:

```text
DEV

QA

PROD
```

---

Different variables:

```text
CIDR

Region

Tags
```

---

Same module.

---

No duplication.

---

# Dependency Management

Terragrunt supports:

```text
Dependency Blocks
```

---

Example:

```text
Security Group
```

depends on:

```text
VPC
```

---

Terragrunt automatically retrieves:

```text
Module Outputs
```

---

# Example Flow

```text
VPC Module
      ↓
Outputs VPC ID
      ↓
Security Group Module
      ↓
Uses VPC ID
```

---

# Terragrunt Dependency Block

Concept:

```text
Read Outputs

Pass Outputs

Manage Dependencies
```

---

Useful for:

```text
Large Infrastructure
```

---

# Multi-Account Deployments

Organizations often manage:

```text
DEV Account

QA Account

PROD Account
```

---

Terragrunt helps organize:

```text
Account Configurations

Provider Configurations

State Locations
```

---

# Infrastructure at Scale

Example:

```text
100+ Terraform Modules
```

---

Challenges:

```text
State Management

Provider Management

Dependency Management
```

---

Terragrunt simplifies operations.

---

# Run All Operations

Terragrunt provides:

```bash
terragrunt run-all apply
```

---

Purpose:

```text
Execute Across Multiple Modules
```

---

Example:

```text
VPC

SG

ALB

EKS
```

---

Automatically executed in dependency order.

---

# Production Example

Organization:

```text
DEV

QA

PROD
```

---

Resources:

```text
VPC

EKS

ALB

Databases
```

---

Instead of:

```text
Hundreds Of Repeated Files
```

---

Terragrunt manages:

```text
Shared Configuration

Remote State

Dependencies
```

---

# Terragrunt vs Terraform

Terraform:

```text
Infrastructure Provisioning Tool
```

---

Terragrunt:

```text
Terraform Management Tool
```

---

Terraform:

```text
Creates Resources
```

---

Terragrunt:

```text
Organizes Terraform
```

---

# Advantages

```text
Reduced Duplication

DRY Principle

Centralized Configuration

Multi Environment Support

Multi Account Support

Dependency Management

Remote State Management
```

---

# Limitations

Terragrunt:

```text
Additional Learning Curve
```

---

Not always necessary for:

```text
Small Projects
```

---

Most useful for:

```text
Medium To Large Organizations
```

---

# Common Interview Questions

## What is Terragrunt?

### Short Answer

Terragrunt is a wrapper around Terraform that helps manage large-scale Terraform projects.

### Detailed Explanation

Terragrunt reduces duplication and simplifies environment, state, and dependency management.

### Production Example

Managing DEV, QA, and PROD environments using shared Terraform modules.

### Follow-Up Questions

- Does Terragrunt replace Terraform?
- Why was Terragrunt created?

---

## Why Use Terragrunt?

### Short Answer

To reduce code duplication and simplify infrastructure management.

### Detailed Explanation

Terragrunt applies the DRY principle to Terraform by centralizing common configurations.

### Production Example

Sharing backend and provider configurations across multiple environments.

### Follow-Up Questions

- What problems does Terragrunt solve?
- Can Terragrunt manage remote state?

---

## Does Terragrunt Replace Terraform?

### Short Answer

No.

### Detailed Explanation

Terragrunt uses Terraform internally and extends its capabilities.

### Production Example

Terragrunt executing Terraform modules for AWS infrastructure deployment.

### Follow-Up Questions

- What is the relationship between Terragrunt and Terraform?
- Can Terraform be used without Terragrunt?

---

## What are the Main Benefits of Terragrunt?

### Short Answer

DRY configuration, dependency management, and environment standardization.

### Detailed Explanation

Terragrunt helps manage large Terraform estates efficiently by eliminating repetitive configurations.

### Production Example

Managing hundreds of modules across multiple AWS accounts.

### Follow-Up Questions

- Is Terragrunt useful for small projects?
- How does Terragrunt handle dependencies?

---

# Key Takeaways

```text
Terragrunt is a wrapper around Terraform.

It follows the DRY (Don't Repeat Yourself) principle.

Terragrunt reduces code duplication.

It simplifies backend and provider management.

Terragrunt helps manage multi-environment and multi-account deployments.

Dependency management is easier with Terragrunt.

Terragrunt does not replace Terraform.

Terragrunt is most beneficial in medium and large organizations.

Many platform engineering teams use Terragrunt for large-scale infrastructure management.

Understanding Terragrunt is valuable for senior DevOps and platform engineering roles.
```