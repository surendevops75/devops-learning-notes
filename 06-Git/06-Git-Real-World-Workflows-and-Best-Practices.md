# Git Real World Workflows And Best Practices

## Introduction

Learning Git commands is easy.

Understanding:

```text
When To Use Them

Why To Use Them

Which Workflow To Follow
```

is what separates beginners from experienced engineers.

---

Most production Git issues happen because:

```text
Wrong Branch Strategy

Wrong Merge Strategy

Poor Conflict Resolution

Environment Branching Mistakes
```

---

This file covers practical Git concepts used in real organizations.

---

# Merge vs Rebase In Real Projects

Both Merge and Rebase combine changes.

---

But they work differently.

---

# Merge

Merge creates:

```text
Merge Commit
```

---

Merge Commit contains:

```text
Two Parents
```

---

Example

```text
main

A --- B --- C
       \     \
        D --- E --- M
```

---

M = Merge Commit

---

Benefits

```text
History Preserved

Safe For Teams

No Commit IDs Changed
```

---

Recommended For

```text
Shared Branches

Team Branches

Public Branches
```

---

# Rebase

Rebase does not create:

```text
Merge Commit
```

---

Instead it:

```text
Rewrites History

Changes Commit IDs

Creates Linear History
```

---

Example

```text
A --- B --- C --- D' --- E'
```

---

Benefits

```text
Clean History

Easy To Read
```

---

Recommended For

```text
Local Branches

Private Branches

Personal Feature Branches
```

---

# Interview Question

## When Should You Use Merge And Rebase?

### Answer

```text
Merge

↓

Shared Branches
```

---

```text
Rebase

↓

Private Branches
```

---

Reason

```text
Rebase Rewrites History
```

---

# How Git Conflicts Really Happen?

Trainer Example

```text
Two Persons Started Egg Dosa
```

---

Both are cooking:

```text
Same Item
```

---

Now both make different changes.

---

Question:

```text
Which One Is Correct?
```

---

Need Human Decision.

---

Git Works The Same Way.

---

# Real Git Example

Developer A

```python
PORT=8080
```

---

Developer B

```python
PORT=9090
```

---

Both modified:

```text
Same Line
```

---

Git Cannot Decide.

---

Git Creates:

```text
Conflict
```

---

# Best Interview Answer

```text
Git conflicts occur when multiple developers modify the same line of code and Git cannot automatically determine which version should be preserved.
```

---

# Long-Lived vs Short-Lived Branches

Very Important Interview Topic.

---

# Long-Lived Branches

Examples

```text
main

develop

release
```

---

Exist For:

```text
Months

Years
```

---

# Short-Lived Branches

Examples

```text
feature/payment

feature/cart

bugfix/login
```

---

Exist For:

```text
Hours

Days
```

---

Deleted After Merge.

---

# Git Flow Model

Traditional Enterprise Model.

---

Architecture

```text
main

↓

develop

├── feature/*
├── bugfix/*
├── hotfix/*
└── release/*
```

---

# Development Flow

```text
main

↓

develop

↓

feature/payment

↓

PR

↓

develop
```

---

Deploy To

```text
DEV
```

---

# Release Flow

```text
develop

↓

release/v1.0
```

---

Deploy To

```text
QA

UAT
```

---

Bug Fixes

```text
bugfix/*
```

---

After Validation

```text
release

↓

main
```

---

Tag Release.

---

Merge Back To

```text
develop
```

---

# Hotfix Flow

Production Issue Found.

---

Create

```text
hotfix/payment-bug
```

from:

```text
main
```

---

Fix Issue.

---

Deploy To Production.

---

Merge To

```text
main
```

and

```text
develop
```

---

# Where Git Flow Is Useful?

Best For

```text
Waterfall Model

Large Enterprises

Multiple Product Versions

Long Release Cycles
```

---

# GitHub Feature Branching Model

Most Common Today.

---

Architecture

```text
main

↓

feature/*
```

---

Workflow

```text
main

↓

feature/payment

↓

CI Pipeline

↓

PR

↓

Code Review

↓

main
```

---

Simple.

---

Easy To Manage.

---

Works Great With:

```text
GitHub

GitHub Actions

Jenkins
```

---

# Trunk Based Development

Modern Engineering Model.

---

Architecture

```text
main

↓

Small Branch

↓

Merge Quickly

↓

main
```

---

Goal

```text
Continuous Integration
```

---

Popular In

```text
Google

Netflix

Meta
```

---

# Environment Branch Anti-Pattern

Many Beginners Create

```text
dev branch

qa branch

uat branch

prod branch
```

---

This Is Wrong.

---

Why?

Because:

```text
Different Branches

Different Code
```

---

Results

```text
Works In DEV

Fails In PROD
```

---

# Correct DevOps Approach

Same Code.

---

Different Configurations.

---

Flow

```text
main

↓

DEV

↓

QA

↓

UAT

↓

PROD
```

---

Only Change

```text
Configuration

Secrets

Environment Variables
```

---

Code Must Remain Same.

---

# Real Production Example

Developer Creates

```text
feature/eks-upgrade
```

---

Pipeline Runs

```text
Clone

Build

Unit Tests

Security Scans
```

---

PR Approved.

---

Merged To Main.

---

Same Artifact Moves Through

```text
DEV

QA

UAT

PROD
```

---

Without Rebuilding.

---

# Common Interview Questions

## Why Is Rebase Not Recommended On Shared Branches?

### Answer

```text
Rebase Rewrites History

Changes Commit IDs

Can Break Other Developers
```

---

## Why Is Merge Preferred For Teams?

### Answer

```text
Preserves History

No Commit Rewriting

Safe For Shared Branches
```

---

## Why Shouldn't We Create Environment Branches?

### Answer

```text
Different Branches Mean Different Code.

Production Should Run The Same Code That Was Tested Earlier.
```

---

## Which Branching Strategy Is Most Common?

### Answer

```text
Feature Branching Model
```

---

Used By Most Organizations Today.

---

# Key Takeaways

```text
Use Merge For Shared Branches.

Use Rebase For Private Branches.

Conflicts occur when multiple developers modify the same code.

Git Flow is suitable for enterprise release-based development.

Feature Branching is the most commonly used strategy today.

Trunk Based Development is popular in highly automated organizations.

Avoid environment-specific branches.

The same code should move from DEV to QA to UAT to PROD.

Only configurations should change between environments.
```