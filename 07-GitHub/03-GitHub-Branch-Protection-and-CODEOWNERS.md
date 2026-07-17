# GitHub Branch Protection And CODEOWNERS

## Introduction

In small teams, developers may push directly to:

```text
main
```

branch.

---

As organizations grow:

```text
10 Developers

50 Developers

100 Developers
```

---

Direct Pushes Become Dangerous.

---

Problems

```text
Accidental Changes

Broken Builds

Security Issues

Production Outages
```

---

Need Controls.

---

GitHub Provides:

```text
Branch Protection Rules

CODEOWNERS
```

---

# Problem Statement

Suppose Developer Accidentally Pushes

```text
Broken Terraform Code
```

to:

```text
main
```

---

CI/CD Pipeline Executes.

---

Production Deployment Starts.

---

Result

```text
Production Failure
```

---

Need Protection.

---

# What Is Branch Protection?

Branch Protection Prevents:

```text
Direct Pushes

Force Pushes

Unreviewed Merges

Unsafe Changes
```

---

On Important Branches Such As:

```text
main

master

release
```

---

# Why Branch Protection Is Important?

Without Protection

```text
Developer

↓

Push Directly

↓

Production Risk
```

---

With Protection

```text
Developer

↓

PR

↓

Review

↓

CI Validation

↓

Merge
```

---

Safer Workflow.

---

# Branch Protection Workflow

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Review

↓

CI Checks

↓

Approval

↓

Merge

↓

Main
```

---

# Common Branch Protection Rules

GitHub Supports

```text
Require Pull Requests

Require Approvals

Require Status Checks

Require Up-To-Date Branch

Restrict Pushes

Prevent Force Push

Prevent Branch Deletion
```

---

# Require Pull Requests

Blocks Direct Pushes.

---

Without PR

```text
Push Rejected
```

---

Developer Must

```text
Create Pull Request
```

---

# Require Approvals

Merge Allowed Only After:

```text
1 Approval

2 Approvals

3 Approvals
```

---

Depending On Company Policy.

---

# Require Status Checks

Before Merge:

```text
Build Must Pass

Tests Must Pass

Security Scans Must Pass
```

---

Example

```text
GitHub Actions

↓

Build

↓

Tests

↓

Success
```

---

Merge Enabled.

---

# Require Up-To-Date Branch

Feature Branch Must Contain:

```text
Latest Main Changes
```

---

Before Merge.

---

Prevents:

```text
Old Code Merges
```

---

# Restrict Who Can Push

Only Specific Users Can:

```text
Push

Merge

Manage Branch
```

---

Example

```text
Release Managers

Platform Team
```

---

# Prevent Force Push

Blocks

```bash
git push --force
```

---

Prevents:

```text
History Rewrites
```

---

Very Important For Production Branches.

---

# Prevent Branch Deletion

Protects

```text
main

release
```

---

From Accidental Deletion.

---

# Real Production Example

Company Uses

```text
main
```

---

Rules

```text
No Direct Push

2 Approvals Required

CI Must Pass

Force Push Disabled
```

---

Result

```text
Controlled Releases
```

---

# What Is CODEOWNERS?

CODEOWNERS Defines:

```text
Who Must Review Specific Files
```

---

Automatically Assigns Reviewers.

---

Used For:

```text
Security

Infrastructure

Critical Applications
```

---

# Why Do We Need CODEOWNERS?

Suppose Developer Modifies

```text
Terraform Infrastructure
```

---

Need Review From:

```text
DevOps Team
```

---

Suppose Developer Modifies

```text
Security Policies
```

---

Need Review From:

```text
Security Team
```

---

CODEOWNERS Automates This.

---

# CODEOWNERS File Location

GitHub Looks For:

```text
.github/CODEOWNERS
```

---

Most Common Location.

---

# CODEOWNERS Syntax

Example

```text
* @team-leads
```

---

Meaning

```text
All Files

↓

Reviewed By Team Leads
```

---

# Specific Folder Ownership

Example

```text
terraform/ @devops-team
```

---

Meaning

```text
Terraform Changes

↓

DevOps Review Required
```

---

# Multiple Owners

Example

```text
terraform/ @devops-team @cloud-team
```

---

Both Teams Can Review.

---

# Real Production CODEOWNERS Example

```text
# Infrastructure

terraform/ @devops-team

# Kubernetes

kubernetes/ @platform-team

# Security

security/ @security-team

# Documentation

docs/ @documentation-team
```

---

# CODEOWNERS Workflow

Developer Changes

```text
terraform/main.tf
```

---

PR Created.

---

GitHub Automatically Assigns

```text
DevOps Team
```

---

Review Required.

---

Merge Blocked Until Review Complete.

---

# Branch Protection + CODEOWNERS

Powerful Combination

```text
CODEOWNERS

↓

Assign Reviewers
```

---

```text
Branch Protection

↓

Require Approval
```

---

Result

```text
Mandatory Expert Review
```

---

# Real Enterprise Example

Repository

```text
roboshop-infra
```

---

Branch Protection

```text
PR Required

2 Approvals

CI Required
```

---

CODEOWNERS

```text
terraform/ → DevOps Team

security/ → Security Team
```

---

Only Approved Changes Reach Production.

---

# Branch Protection In DevOps

Typical Workflow

```text
Developer

↓

Feature Branch

↓

PR

↓

Terraform Validate

↓

Terraform Plan

↓

Security Scan

↓

DevOps Approval

↓

Merge
```

---

# Useful GitHub Settings

```text
Require Pull Requests

Require Reviews

Require Status Checks

Require CODEOWNERS Review

Prevent Force Push

Prevent Branch Deletion
```

---

# Common Interview Questions

## What Is Branch Protection?

### Short Answer

Branch Protection is a GitHub feature that restricts unsafe actions on important branches.

### Detailed Explanation

Branch Protection enforces policies such as pull requests, approvals, status checks, and push restrictions before changes can reach protected branches.

### Production Example

```text
Main Branch

↓

Protected

↓

PR Required
```

### Follow-Up Questions

```text
Can Direct Push Be Disabled?

Can Force Push Be Blocked?

Why Protect Main Branch?
```

---

## Why Is Branch Protection Important?

### Short Answer

It prevents accidental or unauthorized changes from reaching critical branches.

### Detailed Explanation

Branch Protection ensures that all changes are reviewed, validated, and approved before merge.

### Production Example

```text
Developer

↓

PR

↓

Review

↓

CI

↓

Merge
```

### Follow-Up Questions

```text
What Rules Should Be Enabled?

How Does It Improve Security?

Can It Prevent Production Issues?
```

---

## What Is CODEOWNERS?

### Short Answer

CODEOWNERS is a GitHub file that automatically assigns reviewers based on file paths.

### Detailed Explanation

It ensures that changes are reviewed by the appropriate teams or individuals responsible for specific parts of the repository.

### Production Example

```text
terraform/

↓

DevOps Team Review
```

### Follow-Up Questions

```text
Where Is CODEOWNERS Stored?

Can Multiple Owners Be Defined?

Does CODEOWNERS Block Merges?
```

---

## How Do CODEOWNERS And Branch Protection Work Together?

### Short Answer

CODEOWNERS assigns reviewers, while Branch Protection enforces review requirements.

### Detailed Explanation

Together they ensure that critical code cannot be merged without approval from responsible teams.

### Production Example

```text
Terraform Change

↓

CODEOWNERS

↓

DevOps Review

↓

Branch Protection

↓

Merge
```

### Follow-Up Questions

```text
Can Merge Happen Without Approval?

Can CODEOWNERS Be Bypassed?

How Do Enterprises Use This?
```

---

# Production Scenarios

## Scenario 1

Developer Tries Direct Push To Main.

### Solution

```text
Branch Protection Blocks Push
```

---

## Scenario 2

CI Pipeline Failed.

### Solution

```text
Merge Blocked Until Checks Pass
```

---

## Scenario 3

Terraform File Modified.

### Solution

```text
CODEOWNERS Assigns DevOps Team
```

---

## Scenario 4

Developer Attempts Force Push.

### Solution

```text
Force Push Disabled
```

---

## Scenario 5

Security Policy Modified.

### Solution

```text
Security Team Review Required
```

---

# Key Takeaways

```text
Branch Protection secures important branches.

Protected branches typically include main and release branches.

Branch Protection can require PRs, approvals, and CI checks.

Force pushes and branch deletions can be blocked.

CODEOWNERS automatically assigns reviewers.

CODEOWNERS ensures subject matter experts review changes.

Branch Protection and CODEOWNERS are commonly used together.

These features are essential in enterprise GitHub workflows.
```