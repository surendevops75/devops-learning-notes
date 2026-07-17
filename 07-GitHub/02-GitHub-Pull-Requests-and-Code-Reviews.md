# GitHub Pull Requests And Code Reviews

## Introduction

In modern software development:

```text
Developers Do Not Push Directly To Main Branch
```

---

Instead:

```text
Feature Branch

↓

Pull Request

↓

Code Review

↓

Approval

↓

Merge
```

---

This ensures:

```text
Code Quality

Security

Collaboration

Knowledge Sharing
```

---

# Problem Statement

Suppose Developer A Creates:

```text
Payment Feature
```

---

Developer B Creates:

```text
Notification Feature
```

---

Both Need To Merge Code Into:

```text
Main Branch
```

---

Question

```text
How Do We Validate Code

Before It Reaches Production?
```

---

Answer

```text
Pull Request
```

---

# What Is A Pull Request?

A Pull Request (PR) is a request to merge changes from one branch into another branch.

---

Example

```text
feature/payment

↓

main
```

---

Developer Creates:

```text
Pull Request
```

---

Team Reviews Changes.

---

After Approval:

```text
Merge
```

---

# Why Do We Need Pull Requests?

Without PR

```text
Developer

↓

Direct Push

↓

Production Risk
```

---

With PR

```text
Developer

↓

Pull Request

↓

Review

↓

Approval

↓

Merge
```

---

Safer Development Process.

---

# Pull Request Workflow

Step 1

Create Branch

```bash
git checkout -b feature/payment
```

---

Step 2

Develop Feature

---

Step 3

Push Branch

```bash
git push origin feature/payment
```

---

Step 4

Create Pull Request

---

Step 5

Code Review

---

Step 6

CI/CD Validation

---

Step 7

Approval

---

Step 8

Merge

---

# Pull Request Components

A PR Usually Contains

```text
Title

Description

Commits

Files Changed

Reviewers

Comments

Checks
```

---

# Good Pull Request Title

Bad

```text
Changes
```

---

Good

```text
Add Payment Validation API
```

---

# Good Pull Request Description

Should Explain

```text
What Changed

Why Changed

Testing Performed

Impact
```

---

Example

```text
Added payment validation API.

Added unit tests.

Verified locally.

No breaking changes.
```

---

# What Is Code Review?

Code Review Is The Process Of:

```text
Examining Code

Before Merge
```

---

Purpose

```text
Improve Quality

Find Bugs

Improve Security

Maintain Standards
```

---

# Why Code Reviews Matter?

Reviewers Can Find

```text
Logic Errors

Security Issues

Performance Problems

Coding Standard Violations
```

---

Before Production.

---

# Real Production Example

Developer Writes

```python
password = "admin123"
```

---

Reviewer Finds

```text
Hardcoded Secret
```

---

Issue Fixed Before Merge.

---

# Reviewer Responsibilities

Check

```text
Code Quality

Security

Readability

Architecture

Best Practices
```

---

Reviewers Should Not Only Check

```text
Syntax
```

---

They Should Check

```text
Business Logic
```

---

# Pull Request Comments

Reviewers Can Add

```text
Suggestions

Questions

Corrections
```

---

Example

```text
Can we move this secret to GitHub Secrets?
```

---

Developer Updates Code.

---

PR Updated Automatically.

---

# Approvals

Many Organizations Require

```text
1 Approval

2 Approvals

3 Approvals
```

---

Before Merge Allowed.

---

Example

```text
Backend Lead

+

DevOps Lead
```

---

Approve PR.

---

Merge Allowed.

---

# Status Checks

Organizations Often Require

```text
CI Success

Unit Tests

Security Scans

Code Quality Checks
```

---

Before Merge.

---

Example

```text
GitHub Actions

↓

Build

↓

Tests

↓

SonarQube

↓

Success
```

---

Merge Enabled.

---

# Draft Pull Requests

Sometimes Work Is:

```text
Not Ready
```

---

Create

```text
Draft PR
```

---

Benefits

```text
Early Feedback

Collaboration

No Merge Risk
```

---

# Pull Request Review States

## Comment

```text
Feedback Only
```

---

## Approve

```text
Looks Good
```

---

## Request Changes

```text
Changes Required
```

---

Merge Blocked.

---

# Merge Strategies

GitHub Supports

```text
Merge Commit

Squash Merge

Rebase Merge
```

---

# Merge Commit

Creates

```text
Merge Commit
```

---

Preserves Branch History.

---

# Squash Merge

Combines

```text
Many Commits

↓

One Commit
```

---

Cleaner History.

---

# Rebase Merge

Replays Commits.

---

Produces

```text
Linear History
```

---

# Which Merge Strategy Is Most Common?

Most Organizations Use

```text
Squash Merge
```

---

Reasons

```text
Clean History

Easy Rollback

Easy Release Tracking
```

---

# Pull Requests In DevOps

Typical Workflow

```text
Developer

↓

Feature Branch

↓

PR

↓

GitHub Actions

↓

Code Review

↓

Approval

↓

Merge

↓

Deployment
```

---

# Real Production Example

Developer Creates

```text
feature/eks-upgrade
```

---

PR Created.

---

Pipeline Runs

```text
Terraform Validate

Terraform Plan

Security Scan
```

---

DevOps Team Reviews.

---

Approves.

---

Merge Completed.

---

Infrastructure Updated.

---

# Useful GitHub Features In PRs

```text
Reviewers

Labels

Assignees

Milestones

Draft PRs

Linked Issues
```

---

# Common Interview Questions

## What Is A Pull Request?

### Short Answer

A Pull Request is a request to merge changes from one branch into another branch.

### Detailed Explanation

Pull Requests provide a controlled process for reviewing, validating, and approving changes before merging them into the target branch.

### Production Example

```text
feature/payment

↓

PR

↓

Review

↓

Merge
```

### Follow-Up Questions

```text
Why Are Pull Requests Important?

What Happens During Code Review?

Can Pull Requests Trigger CI/CD?
```

---

## What Is Code Review?

### Short Answer

Code Review is the process of reviewing code before it is merged into the target branch.

### Detailed Explanation

Code reviews help identify bugs, security issues, maintainability problems, and coding standard violations.

### Production Example

```text
Developer

↓

PR

↓

Reviewer

↓

Feedback

↓

Approval
```

### Follow-Up Questions

```text
Who Performs Code Reviews?

What Should Reviewers Check?

Why Are Reviews Important?
```

---

## What Are The Different PR Review States?

### Short Answer

Comment, Approve, and Request Changes.

### Detailed Explanation

Reviewers can provide feedback, approve the changes, or request modifications before merge.

### Production Example

```text
PR

↓

Request Changes

↓

Developer Updates

↓

Approval
```

### Follow-Up Questions

```text
Can PR Be Merged Without Approval?

How Many Approvals Are Required?

Can Reviews Be Dismissed?
```

---

## What Are GitHub Merge Strategies?

### Short Answer

Merge Commit, Squash Merge, and Rebase Merge.

### Detailed Explanation

Each strategy affects commit history differently and organizations choose based on workflow requirements.

### Production Example

```text
Feature Branch

↓

Squash Merge

↓

Single Commit
```

### Follow-Up Questions

```text
Which Strategy Creates Merge Commit?

Which Creates Linear History?

Which Is Most Common?
```

---

## Can Pull Requests Trigger CI/CD Pipelines?

### Short Answer

Yes.

### Detailed Explanation

GitHub Actions and other CI/CD systems can automatically run builds, tests, scans, and validations when a PR is created or updated.

### Production Example

```text
PR Created

↓

Build

↓

Tests

↓

Approval

↓

Merge
```

### Follow-Up Questions

```text
What Checks Should Run On PRs?

Can Failed Checks Block Merge?

How Are Status Checks Configured?
```

---

# Production Scenarios

## Scenario 1

Developer Wants To Merge Feature.

### Solution

```text
Create Pull Request
```

---

## Scenario 2

Security Issue Found During Review.

### Solution

```text
Request Changes

↓

Fix

↓

Review Again
```

---

## Scenario 3

CI Pipeline Failed.

### Solution

```text
Fix Errors

↓

Push Changes

↓

Pipeline Re-runs
```

---

## Scenario 4

PR Contains 50 Small Commits.

### Solution

```text
Squash Merge
```

---

## Scenario 5

Feature Needs Early Feedback.

### Solution

```text
Create Draft Pull Request
```

---

# Key Takeaways

```text
Pull Requests are the standard mechanism for merging code.

Code Reviews improve quality and security.

Pull Requests support comments, approvals, and change requests.

CI/CD pipelines commonly run on Pull Requests.

GitHub supports Merge Commit, Squash Merge, and Rebase Merge.

Squash Merge is commonly used in modern teams.

Draft Pull Requests allow early collaboration.

Every production code change should go through Pull Requests and reviews.
```