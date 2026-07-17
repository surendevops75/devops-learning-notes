# Git Tags And Releases

## Introduction

Software is developed continuously.

---

Features Are Added

```text
Payment Module

Authentication

Notifications

Reports
```

---

At Some Point:

```text
Version Needs To Be Released
```

---

Examples

```text
v1.0

v1.1

v2.0
```

---

Question

```text
How Do We Mark Important Points In Git History?
```

---

Answer

```text
Git Tags
```

---

# Problem Statement

Suppose Application History Is

```text
A → B → C → D → E
```

---

Commit

```text
C
```

was released as:

```text
Version 1.0
```

---

Need A Way To Mark It.

---

Instead Of Remembering

```text
Commit ID
```

---

We Create

```text
Tag
```

---

# What Is A Git Tag?

A Git Tag is:

```text
Named Reference

To A Specific Commit
```

---

Think Of It As:

```text
Bookmark
```

for important commits.

---

Example

```text
v1.0

v1.1

v2.0
```

---

# Why Do We Need Tags?

Without Tags

```text
Need Commit IDs
```

---

Example

```text
84a22a0
```

---

Hard To Remember.

---

With Tags

```text
v1.0
```

---

Easy To Understand.

---

# How Tags Work

History

```text
A → B → C → D → E
```

---

Tag

```text
v1.0
```

points to:

```text
Commit C
```

---

Tag

```text
v1.1
```

points to:

```text
Commit E
```

---

# Types Of Tags

Git Supports

```text
Lightweight Tags

Annotated Tags
```

---

# Lightweight Tag

Simple Reference.

---

Command

```bash
git tag v1.0
```

---

Git Creates

```text
Pointer To Commit
```

---

No Additional Metadata.

---

# Annotated Tag

Recommended For Production.

---

Contains

```text
Tag Name

Author

Date

Message
```

---

Command

```bash
git tag -a v1.0 -m "First Production Release"
```

---

# Why Annotated Tags Are Better?

Provides

```text
Audit Information

Release Notes

Metadata
```

---

Most Organizations Use

```text
Annotated Tags
```

---

# Viewing Tags

List All Tags

```bash
git tag
```

---

Output

```text
v1.0

v1.1

v2.0
```

---

# View Tag Details

Command

```bash
git show v1.0
```

---

Displays

```text
Commit

Tag Message

Author

Date
```

---

# Creating Tag On Specific Commit

Find Commit

```bash
git log --oneline
```

---

Example

```text
84a22a0 Added Payment Module
```

---

Create Tag

```bash
git tag -a v1.0 84a22a0 -m "Production Release"
```

---

Tag Now Points To

```text
Commit 84a22a0
```

---

# Pushing Tags To Remote

Tags Are Not Pushed Automatically.

---

Push Single Tag

```bash
git push origin v1.0
```

---

Push All Tags

```bash
git push origin --tags
```

---

Important Interview Question.

---

# Deleting Tags

Delete Local Tag

```bash
git tag -d v1.0
```

---

Delete Remote Tag

```bash
git push origin --delete v1.0
```

---

# Checking Out A Tag

Command

```bash
git checkout v1.0
```

---

Git Moves To

```text
Version 1.0 Code
```

---

Useful For

```text
Troubleshooting

Bug Analysis

Rollback
```

---

# Tags In Release Management

Example

```text
v1.0

↓

Production Release
```

---

New Features Added

---

Create

```text
v1.1
```

---

More Features Added

---

Create

```text
v2.0
```

---

Every Version Has:

```text
Known Reference Point
```

---

# Semantic Versioning

Most Common Format

```text
MAJOR.MINOR.PATCH
```

---

Example

```text
v1.0.0
```

---

# Major Version

Example

```text
v2.0.0
```

---

Meaning

```text
Breaking Changes
```

---

# Minor Version

Example

```text
v1.1.0
```

---

Meaning

```text
New Features
```

---

# Patch Version

Example

```text
v1.0.1
```

---

Meaning

```text
Bug Fixes
```

---

# Real Production Example

Application

```text
Roboshop
```

---

Release

```text
v1.0.0
```

---

Tag Created

```bash
git tag -a v1.0.0 -m "Production Release"
```

---

Deployment Starts.

---

Future Rollbacks Become Easy.

---

# Tags In CI/CD

Typical Workflow

```text
Code Merge

↓

Create Tag

↓

Build Artifact

↓

Docker Image

↓

Deploy
```

---

Example

```text
v1.2.0
```

---

Docker Image

```text
roboshop:v1.2.0
```

---

# Tags In GitHub Actions

Workflow Trigger

```yaml
on:
  push:
    tags:
      - 'v*'
```

---

Whenever Tag Created

```text
Pipeline Executes
```

---

Very Common In Production.

---

# Release Rollback Using Tags

Production Running

```text
v1.2.0
```

---

Issue Found.

---

Rollback To

```text
v1.1.0
```

---

Command

```bash
git checkout v1.1.0
```

---

Redeploy.

---

Issue Resolved.

---

# Useful Commands

Create Lightweight Tag

```bash
git tag v1.0
```

---

Create Annotated Tag

```bash
git tag -a v1.0 -m "Release"
```

---

List Tags

```bash
git tag
```

---

Show Tag

```bash
git show v1.0
```

---

Push Tag

```bash
git push origin v1.0
```

---

Push All Tags

```bash
git push origin --tags
```

---

Delete Local Tag

```bash
git tag -d v1.0
```

---

Delete Remote Tag

```bash
git push origin --delete v1.0
```

---

# Common Interview Questions

## What Is A Git Tag?

### Short Answer

A Git Tag is a named reference to a specific commit.

### Detailed Explanation

Tags are used to mark important commits such as releases, milestones, and production deployments.

### Production Example

```text
Commit

↓

Tag

↓

v1.0
```

### Follow-Up Questions

```text
Why Use Tags?

How Are Tags Different From Branches?

Can Tags Be Moved?
```

---

## Difference Between Lightweight And Annotated Tags?

### Short Answer

Lightweight tags are simple pointers, while annotated tags contain metadata.

### Detailed Explanation

Annotated tags store author information, timestamps, and release messages, making them more suitable for production use.

### Production Example

```text
Lightweight

↓

Pointer Only
```

```text
Annotated

↓

Pointer + Metadata
```

### Follow-Up Questions

```text
Which Is Preferred In Production?

Why Use Annotated Tags?

Can Both Be Pushed To Remote?
```

---

## Why Are Tags Important In CI/CD?

### Short Answer

Tags identify release versions and can trigger deployment pipelines.

### Detailed Explanation

Tags provide stable version references for building, deploying, and rolling back applications.

### Production Example

```text
Tag Created

↓

Pipeline Triggered

↓

Deployment
```

### Follow-Up Questions

```text
Can GitHub Actions Trigger On Tags?

How Are Docker Images Tagged?

How Do Tags Help Rollbacks?
```

---

## What Is Semantic Versioning?

### Short Answer

A versioning standard using MAJOR.MINOR.PATCH format.

### Detailed Explanation

Semantic versioning helps teams understand the impact of changes between releases.

### Production Example

```text
v1.0.0

v1.1.0

v1.1.1

v2.0.0
```

### Follow-Up Questions

```text
When Should Major Version Change?

What Is Patch Version?

What Is Minor Version?
```

---

# Production Scenarios

## Scenario 1

Need To Release Version 1.0.

### Solution

```bash
git tag -a v1.0 -m "Production Release"
```

---

## Scenario 2

Need To Trigger Release Pipeline.

### Solution

```text
Create Tag

↓

Push Tag
```

---

## Scenario 3

Need To Rollback Production.

### Solution

```text
Checkout Previous Tag

↓

Redeploy
```

---

## Scenario 4

Need To Identify Exact Release Commit.

### Solution

```bash
git show v1.0
```

---

## Scenario 5

Need To Push All Release Tags.

### Solution

```bash
git push origin --tags
```

---

# Key Takeaways

```text
Git Tags are named references to commits.

Tags are commonly used for releases.

Git supports lightweight and annotated tags.

Annotated tags are preferred in production.

Tags are not pushed automatically.

Tags play a major role in CI/CD and release management.

Semantic Versioning follows MAJOR.MINOR.PATCH.

Tags simplify deployments and rollbacks.

Every production release should be tagged.
```