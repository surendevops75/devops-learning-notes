# Terraform Module Versioning

## Introduction

Modules are reusable Terraform code blocks.

Examples:

```text
VPC Module

EKS Module

ALB Module

Security Group Module

RDS Module
```

---

Instead of writing the same code repeatedly:

```text
Write Once

Reuse Many Times
```

---

Like software packages, modules also have:

```text
Versions
```

---

Module versioning helps:

```text
Maintain Stability

Control Upgrades

Avoid Breaking Changes
```

---

# What is Module Versioning?

Module Versioning allows Terraform to:

```text
Download A Specific Module Version
```

instead of always downloading the latest version.

---

Example:

```hcl
module "vpc" {

  source  = "terraform-aws-modules/vpc/aws"

  version = "5.16.0"

}
```

---

Terraform downloads:

```text
Version 5.16.0
```

of the module.

---

# Why Module Versioning?

Module developers continuously:

```text
Add Features

Fix Bugs

Improve Security

Refactor Code
```

---

New versions may introduce:

```text
Input Changes

Output Changes

Resource Changes

Breaking Changes
```

---

Without version pinning:

```text
Infrastructure May Break Unexpectedly
```

---

# Module Sources

Modules can come from:

```text
Terraform Registry

GitHub

Git Repositories

Private Registries
```

---

# Terraform Registry Example

```hcl
module "vpc" {

  source  = "terraform-aws-modules/vpc/aws"

  version = "5.16.0"

}
```

---

# GitHub Module Example

```hcl
module "vpc" {

  source = "git::https://github.com/company/vpc-module.git?ref=v1.0.0"

}
```

---

Meaning:

```text
Use Git Tag v1.0.0
```

---

# Semantic Versioning

Most modules follow:

```text
Major.Minor.Patch
```

---

Example:

```text
5.16.0
```

---

Meaning:

```text
5 → Major

16 → Minor

0 → Patch
```

---

# Major Version Changes

Example:

```text
4.x → 5.x
```

---

May introduce:

```text
Breaking Changes

Input Changes

Output Changes
```

---

Requires:

```text
Careful Testing
```

---

# Minor Version Changes

Example:

```text
5.15 → 5.16
```

---

Usually introduces:

```text
New Features

Improvements
```

---

Typically backward compatible.

---

# Patch Version Changes

Example:

```text
5.16.0 → 5.16.1
```

---

Usually contains:

```text
Bug Fixes

Security Fixes
```

---

# Version Constraints

Terraform supports:

```text
=

>

>=

<

<=

~>
```

---

# Exact Version

```hcl
version = "= 5.16.0"
```

---

Meaning:

```text
Only Version 5.16.0
```

---

# Range Example

```hcl
version = ">= 5.0, < 6.0"
```

---

Meaning:

```text
Allow All 5.x Versions

Reject 6.x
```

---

# Pessimistic Constraint

Most commonly used:

```hcl
version = "~> 5.16"
```

---

Allows:

```text
5.16

5.17

5.18
```

---

Blocks:

```text
6.0
```

---

# Real Production Example

Current Module:

```text
VPC Module 4.0
```

---

Terraform Configuration:

```hcl
module "vpc" {

  source  = "terraform-aws-modules/vpc/aws"

  version = "4.0"

}
```

---

Everything working correctly.

---

Engineer upgrades:

```text
4.0 → 5.0
```

---

Result:

```text
Some Inputs Renamed

Some Outputs Changed

Code Fails
```

---

# Common Upgrade Problem

Old Module:

```hcl
module.vpc.private_subnets
```

---

New Module:

```hcl
module.vpc.private_subnet_ids
```

---

Result:

```text
Unsupported Attribute
```

---

Plan fails.

---

# Module Upgrade Strategy

Current:

```text
VPC Module 4.x
```

---

Target:

```text
VPC Module 5.x
```

---

Process:

```text
Read Release Notes

Check Breaking Changes

Upgrade DEV

Run Plan

Validate

Upgrade QA

Validate

Upgrade PROD
```

---

Never directly upgrade:

```text
Production
```

---

# Relationship With Provider Version

Architecture:

```text
Terraform Version
        ↓
Provider Version
        ↓
Module Version
        ↓
AWS Resources
```

---

Modules depend on:

```text
Provider Features
```

---

Example:

```text
VPC Module 5.x
```

requires:

```text
AWS Provider 5.x
```

---

But project uses:

```text
AWS Provider 3.x
```

---

Result:

```text
Compatibility Issues
```

---

# Module Compatibility Example

Module:

```text
VPC Module 5.0
```

---

Expected:

```text
AWS Provider 5.x
```

---

Current:

```text
AWS Provider 3.x
```

---

Result:

```text
Plan Failure

Unsupported Arguments

Missing Attributes
```

---

# Private Modules

Organizations often maintain:

```text
Internal Modules
```

---

Examples:

```text
Company VPC Module

Company EKS Module

Company ALB Module
```

---

Versioning becomes even more important.

---

Why?

```text
Multiple Teams Use Same Module
```

---

Breaking changes affect:

```text
Many Projects
```

---

# Production Example

Platform Team:

```text
Maintains EKS Module
```

---

Version:

```text
2.0
```

---

Application Teams:

```text
Use Module Version 2.0
```

---

Platform Team Releases:

```text
3.0
```

---

Teams decide:

```text
When To Upgrade
```

---

instead of automatic upgrades.

---

# Best Practices

```text
Always Pin Module Versions

Avoid Using Latest

Read Release Notes

Test Upgrades In DEV

Upgrade Gradually

Use Semantic Versioning
```

---

# Architecture

```text
Terraform
      ↓
Module Version
      ↓
Provider Version
      ↓
Cloud API
      ↓
Infrastructure
```

---

# Common Interview Questions

## Why Should Module Versions Be Pinned?

### Short Answer

To avoid unexpected breaking changes.

### Detailed Explanation

Module upgrades can change inputs, outputs, and internal resources, affecting existing deployments.

### Production Example

VPC Module upgrade breaking dependent modules.

### Follow-Up Questions

- What happens if versions are not pinned?
- How do you upgrade modules safely?

---

## What is the Difference Between Module Version and Provider Version?

### Short Answer

Module Version controls reusable infrastructure code, while Provider Version controls cloud API communication.

### Detailed Explanation

Modules define infrastructure logic, whereas providers interact with cloud services.

### Production Example

VPC Module 5.16 running with AWS Provider 5.50.

### Follow-Up Questions

- Can module upgrades require provider upgrades?
- Can modules work across provider versions?

---

## Why Are Major Module Upgrades Risky?

### Short Answer

Major versions often introduce breaking changes.

### Detailed Explanation

Inputs, outputs, resource names, and behaviors may change between major releases.

### Production Example

VPC Module 4.x to 5.x upgrade requiring code changes.

### Follow-Up Questions

- How do you safely upgrade modules?
- Why read release notes?

---

# Key Takeaways

```text
Modules are reusable Terraform code packages.

Modules should always be versioned and pinned.

Module upgrades may introduce breaking changes.

Semantic Versioning is commonly used.

Major versions may require code modifications.

Modules depend on provider compatibility.

Module upgrades should be tested in lower environments first.

Never blindly use the latest module version in production.

Module versioning improves stability and predictability.

Understanding module versioning is essential for production Terraform environments.
```