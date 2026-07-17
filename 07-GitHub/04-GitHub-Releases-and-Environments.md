# GitHub Releases And Environments

## Introduction

Writing Code Is Not The Final Goal.

---

The Real Goal Is:

```text
Build

Test

Release

Deploy
```

---

Organizations Need A Way To:

```text
Track Releases

Manage Versions

Control Deployments

Separate Environments
```

---

GitHub Provides:

```text
Releases

Environments
```

---

# Problem Statement

Suppose Application Has:

```text
v1.0

v1.1

v2.0
```

---

Question

```text
How Do We Know

Which Commit

Was Released To Production?
```

---

Need:

```text
Version Tracking
```

---

Also Need:

```text
DEV

QA

UAT

PROD
```

---

With Different Deployment Rules.

---

# What Is A GitHub Release?

A Release Is:

```text
Packaged Version

Of Your Application
```

---

Built On Top Of:

```text
Git Tags
```

---

Think Of It As:

```text
Official Software Version
```

---

# Release Workflow

```text
Code Complete

↓

Tag Created

↓

Release Created

↓

Deployment
```

---

Example

```text
v1.0.0
```

---

# Why Releases Are Important?

Benefits

```text
Version Tracking

Rollback Support

Release Notes

Deployment History

Auditability
```

---

# Tags Vs Releases

## Tag

```text
Points To Commit
```

---

Example

```text
v1.0.0
```

---

## Release

```text
Tag

+

Release Notes

+

Artifacts

+

Documentation
```

---

# Creating A Release

GitHub

↓

Releases

↓

Create New Release

↓

Select Tag

↓

Release Title

↓

Release Notes

↓

Publish Release

---

# Release Components

A Release Usually Contains

```text
Version Number

Tag

Release Notes

Assets

Artifacts
```

---

# Release Notes

Describe

```text
New Features

Bug Fixes

Security Updates

Breaking Changes
```

---

Example

```text
Added Payment Service

Fixed Login Bug

Updated Kubernetes Manifests
```

---

# Release Assets

Can Attach

```text
ZIP Files

Binaries

Reports

Documentation
```

---

Example

```text
application.zip

terraform-plan.pdf

security-report.pdf
```

---

# Semantic Versioning

Format

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

```text
v2.0.0
```

Meaning

```text
Breaking Changes
```

---

# Minor Version

```text
v1.1.0
```

Meaning

```text
New Features
```

---

# Patch Version

```text
v1.0.1
```

Meaning

```text
Bug Fixes
```

---

# Real Production Example

Release

```text
v1.5.0
```

Contains

```text
New Payment Module

Performance Improvements

Security Fixes
```

---

Release Notes Published.

---

Deployment Starts.

---

# What Are GitHub Environments?

GitHub Environments Allow Teams To:

```text
Control Deployments

Protect Environments

Manage Secrets
```

---

Examples

```text
DEV

QA

UAT

PROD
```

---

# Why Environments Are Needed?

Different Environments Require

```text
Different Secrets

Different Approvals

Different Controls
```

---

Example

```text
DEV AWS Account

PROD AWS Account
```

---

Different Credentials.

---

# Environment Workflow

```text
Code Merge

↓

Deploy DEV

↓

Testing

↓

Deploy QA

↓

Validation

↓

Deploy PROD
```

---

# Creating Environments

GitHub Repository

↓

Settings

↓

Environments

↓

Create Environment

---

Example

```text
dev

qa

uat

prod
```

---

# Environment Secrets

Each Environment Can Have:

```text
Different Secrets
```

---

Example

DEV

```text
AWS_DEV_ACCESS_KEY
```

---

PROD

```text
AWS_PROD_ACCESS_KEY
```

---

Safer Deployments.

---

# Environment Protection Rules

GitHub Supports

```text
Required Reviewers

Wait Timers

Deployment Restrictions
```

---

# Required Reviewers

Before Deployment

```text
Approval Required
```

---

Example

```text
Production Deployment

↓

DevOps Manager Approval
```

---

# Wait Timers

Example

```text
Wait 30 Minutes

Before Deployment
```

---

Useful For

```text
Change Windows

Compliance
```

---

# Deployment Branch Restrictions

Allow Deployment Only From

```text
main

release/*
```

---

Prevent Unauthorized Deployments.

---

# Real Production Example

Environment

```text
PROD
```

Rules

```text
Only Main Branch

2 Approvals

Production Secrets

Deployment Allowed
```

---

# GitHub Releases In DevOps

Typical Flow

```text
Merge Code

↓

Tag

↓

Release

↓

Build Artifact

↓

Deploy
```

---

Example

```text
v2.0.0
```

↓

Docker Image

```text
roboshop:v2.0.0
```

↓

Deployment

---

# GitHub Environments In DevOps

Example

```text
GitHub Actions

↓

DEV

↓

QA

↓

PROD
```

---

Each Environment Uses

```text
Different Secrets

Different Controls
```

---

# Rollback Using Releases

Current Version

```text
v2.0.0
```

---

Issue Found.

---

Rollback To

```text
v1.9.0
```

---

Redeploy Previous Release.

---

Issue Resolved.

---

# Useful GitHub Features

```text
Tags

Releases

Release Notes

Environment Secrets

Required Reviewers

Deployment Controls
```

---

# Common Interview Questions

## What Is A GitHub Release?

### Short Answer

A GitHub Release is an official packaged version of software built on top of Git tags.

### Detailed Explanation

Releases provide version tracking, release notes, assets, and deployment history.

### Production Example

```text
Tag

↓

Release

↓

Deployment
```

### Follow-Up Questions

```text
Difference Between Tags And Releases?

Why Are Releases Important?

Can Releases Be Used For Rollbacks?
```

---

## What Are GitHub Environments?

### Short Answer

GitHub Environments are deployment targets with protection rules and secrets management.

### Detailed Explanation

They allow teams to control deployments to DEV, QA, UAT, and PROD environments.

### Production Example

```text
DEV

↓

QA

↓

PROD
```

### Follow-Up Questions

```text
Can Each Environment Have Different Secrets?

Can Approvals Be Required?

How Are Deployments Controlled?
```

---

## Why Use Environment Secrets?

### Short Answer

To securely store different credentials for different environments.

### Detailed Explanation

Environment-specific secrets prevent accidental use of production credentials in non-production environments.

### Production Example

```text
DEV AWS Keys

↓

DEV Environment
```

```text
PROD AWS Keys

↓

PROD Environment
```

### Follow-Up Questions

```text
How Are Secrets Accessed?

Can Secrets Be Shared?

How Does GitHub Encrypt Secrets?
```

---

## How Do Releases Help Rollbacks?

### Short Answer

Releases provide stable version references that can be redeployed when issues occur.

### Detailed Explanation

Teams can quickly identify and redeploy a previous release version.

### Production Example

```text
v2.0.0 Failure

↓

Rollback

↓

v1.9.0
```

### Follow-Up Questions

```text
Can Releases Be Deleted?

What Is Release History?

How Are Releases Linked To Tags?
```

---

# Production Scenarios

## Scenario 1

Need Official Software Version.

### Solution

```text
Create Release
```

---

## Scenario 2

Need Release Notes.

### Solution

```text
GitHub Release Notes
```

---

## Scenario 3

Need Separate Production Secrets.

### Solution

```text
Environment Secrets
```

---

## Scenario 4

Need Approval Before Production Deployment.

### Solution

```text
Required Reviewers
```

---

## Scenario 5

Need Rollback To Previous Version.

### Solution

```text
Redeploy Previous Release
```

---

# Key Takeaways

```text
GitHub Releases are built on top of Git tags.

Releases provide version tracking and release notes.

Semantic Versioning uses MAJOR.MINOR.PATCH.

GitHub Environments control deployments.

Environments support secrets and protection rules.

Different environments can have different credentials.

Production deployments can require approvals.

Releases simplify rollback and audit processes.

GitHub Releases and Environments are heavily used in CI/CD workflows.
```