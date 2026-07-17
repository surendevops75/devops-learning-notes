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

Example

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

## When Should You Use Merge And Rebase?

### Short Answer

Use Merge for shared branches and Rebase for private or local branches.

### Detailed Explanation

Merge preserves commit history by creating a merge commit and does not modify existing commit IDs. Rebase rewrites history by creating new commit IDs and maintaining a linear history.

Because rebase changes commit history, it should not be used on branches shared with other developers.

### Production Example

```text
Shared Branch

↓

Merge
```

```text
Feature Branch

↓

Rebase
```

### Follow-Up Questions

```text
What is a Merge Commit?

Why does Rebase change commit IDs?

Can Rebase create conflicts?

Why is Rebase dangerous on shared branches?
```

---

## How Do Git Conflicts Occur?

### Short Answer

Git conflicts occur when multiple developers modify the same line of code and Git cannot determine which version should be preserved.

### Detailed Explanation

Git automatically merges changes when modifications occur in different areas of a file. When changes occur on the same line or overlapping sections, Git requires manual intervention.

### Production Example

Developer A

```python
PORT=8080
```

Developer B

```python
PORT=9090
```

Git cannot determine which value is correct.

```text
Conflict
```

### Follow-Up Questions

```text
Can conflicts occur during Rebase?

How do you resolve conflicts?

How can teams reduce conflicts?
```

---

## What Is The Difference Between Long-Lived And Short-Lived Branches?

### Short Answer

Long-lived branches exist for extended periods, while short-lived branches are created for specific tasks and deleted after merging.

### Detailed Explanation

Long-lived branches typically represent environments or major development streams. Short-lived branches are used for features, bug fixes, and hotfixes.

Short-lived branches reduce merge conflicts and simplify integration.

### Production Example

Long-Lived

```text
main

develop

release
```

Short-Lived

```text
feature/payment

bugfix/login

hotfix/checkout
```

### Follow-Up Questions

```text
Why are long-lived feature branches risky?

Which branching model uses more short-lived branches?

How do short-lived branches reduce conflicts?
```

---

## What Is Git Flow?

### Short Answer

Git Flow is a branching strategy that uses dedicated branches for development, releases, features, bug fixes, and hotfixes.

### Detailed Explanation

Git Flow separates production, development, release preparation, and emergency fixes into dedicated branches. It provides strong release management but introduces additional complexity.

### Production Example

```text
main

↓

develop

├── feature/*

├── bugfix/*

├── release/*

└── hotfix/*
```

### Follow-Up Questions

```text
What is a Release Branch?

What is a Hotfix Branch?

Why is Git Flow less common in cloud-native teams?
```

---

## What Is GitHub Feature Branching Model?

### Short Answer

A branching strategy that uses a main branch and short-lived feature branches.

### Detailed Explanation

Developers create feature branches from main, implement changes, raise pull requests, and merge changes back into main after review and validation.

### Production Example

```text
main

↓

feature/payment

↓

PR

↓

main
```

### Follow-Up Questions

```text
Why is Feature Branching popular?

How does CI/CD integrate with Pull Requests?

What are the advantages over Git Flow?
```

---

## What Is Trunk Based Development?

### Short Answer

Developers integrate changes into the main branch frequently using short-lived branches.

### Detailed Explanation

Trunk Based Development emphasizes continuous integration, rapid feedback, and frequent merging. Teams rely heavily on automated testing and CI/CD pipelines.

### Production Example

```text
Small Branch

↓

Merge Quickly

↓

main
```

### Follow-Up Questions

```text
Why do Google and Netflix use Trunk Based Development?

How does CI/CD support this model?

What are the risks without automated testing?
```

---

## Why Shouldn't We Create Environment Branches?

### Short Answer

Environment branches can result in different code existing in different environments.

### Detailed Explanation

The recommended DevOps practice is to promote the same artifact through DEV, QA, UAT, and PROD. Only configurations should differ between environments.

### Production Example

Incorrect

```text
dev

qa

uat

prod
```

Correct

```text
Same Build

↓

DEV

↓

QA

↓

UAT

↓

PROD
```

### Follow-Up Questions

```text
What changes between environments?

Why should code remain identical?

How do configuration files support environment differences?
```

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