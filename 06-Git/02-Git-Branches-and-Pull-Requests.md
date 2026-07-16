# Git Branches And Pull Requests

## Introduction

In real projects, developers never work directly on:

```text
main
```

branch.

---

Why?

Because:

```text
main

↓

Production Code
```

---

A bug in main can directly impact:

```text
Customers

Production Systems

Business Operations
```

---

Need a safer development process.

---

Git provides:

```text
Branches
```

---

# Why Branches Were Created?

Suppose two developers are working.

---

Developer A

```text
Payment Feature
```

---

Developer B

```text
Notification Feature
```

---

Without Branches

```text
Both Modify Main Branch
```

---

Problems

```text
Code Conflicts

Broken Builds

Production Risks
```

---

Need Isolation.

---

Branches Solve This.

---

# What Is A Branch?

A branch is:

```text
Independent Line Of Development
```

---

Think Of It As:

```text
Copy Of Main Branch
```

where developers can safely work.

---

Example

```text
main

├── payment-feature

├── notification-feature

└── bugfix-login
```

---

Each developer works independently.

---

# Default Branch

Most repositories use:

```text
main
```

---

This branch usually represents:

```text
Production Ready Code
```

---

Rule

```text
Never Develop Directly On Main
```

---

# Feature Branch Workflow

Real Production Flow

```text
main

↓

Create Feature Branch

↓

Development

↓

Testing

↓

Push Branch

↓

Pull Request

↓

Code Review

↓

Merge To Main

↓

CI/CD

↓

Deployment
```

---

This is the most common workflow.

---

# Creating A Branch

Command

```bash
git checkout -b payment-feature
```

---

Example

```bash
git checkout -b user-authentication
```

---

Creates

```text
New Branch

+

Switches To Branch
```

---

Verify

```bash
git branch
```

---

Output

```text
* user-authentication

main
```

---

Current branch marked with:

```text
*
```

---

# Switching Between Branches

Command

```bash
git checkout main
```

---

Or

```bash
git switch main
```

---

Example

```bash
git switch payment-feature
```

---

# Branch Development Example

Current Branch

```text
payment-feature
```

---

Developer Changes

```text
payment.py
```

---

Commit Changes

```bash
git add .

git commit -m "Added payment validation"
```

---

Branch Contains New Feature.

---

Main Branch Unaffected.

---

# Viewing Branches

List Branches

```bash
git branch
```

---

List Local And Remote

```bash
git branch -a
```

---

List Remote Only

```bash
git branch -r
```

---

# Push Branch To GitHub

Local Branch Exists.

---

Need To Share With Team.

---

Command

```bash
git push origin payment-feature
```

---

Flow

```text
Local Branch

↓

GitHub Branch
```

---

Team Can Now Review Code.

---

# What Is A Pull Request (PR)?

One Of The Most Important Git Concepts.

---

A Pull Request is:

```text
Request To Merge Changes
```

from one branch into another.

---

Example

```text
payment-feature

↓

main
```

---

Developer Creates PR.

---

Reviewers Examine:

```text
Code Quality

Security

Standards

Logic
```

---

Then Approve Or Reject.

---

# Why Pull Requests?

Without PR

```text
Anyone Can Push Anything
```

to main.

---

Risky.

---

PR Provides

```text
Code Review

Discussion

Approval Process

Audit Trail
```

---

# Pull Request Workflow

```text
Create Branch

↓

Code Changes

↓

Commit

↓

Push Branch

↓

Create PR

↓

Code Review

↓

Approval

↓

Merge

↓

Delete Branch
```

---

# Real Production Example

Developer Creates

```text
payment-feature
```

---

Adds:

```text
Payment Validation Logic
```

---

Pushes Branch

```bash
git push origin payment-feature
```

---

Creates PR

```text
payment-feature

↓

main
```

---

Reviewer Checks

```text
Code

Security

Performance
```

---

Approves PR.

---

Changes Merged.

---

# Pull Request States

## Open

Waiting For Review.

---

## Approved

Ready To Merge.

---

## Changes Requested

Developer Must Update Code.

---

## Merged

Changes Integrated.

---

## Closed

Rejected Without Merge.

---

# Why CI/CD Runs On Pull Requests?

Most companies configure:

```text
GitHub Actions

Jenkins

GitLab CI
```

to run automatically.

---

Flow

```text
PR Created

↓

Build

↓

Unit Tests

↓

Security Scan

↓

Approval
```

---

Only Successful Code Gets Merged.

---

# Branch Protection Rules

Production repositories usually protect:

```text
main
```

---

Example Rules

```text
No Direct Push

PR Required

Code Review Required

CI/CD Must Pass
```

---

This prevents accidental changes.

---

# Production GitHub Workflow

```text
main

↓

Create Feature Branch

↓

Development

↓

Commit

↓

Push

↓

PR

↓

Review

↓

CI/CD

↓

Merge

↓

Deployment
```

---

Most DevOps teams follow this model.

---

# Branch Naming Conventions

Feature

```text
feature/payment-api
```

---

Bug Fix

```text
bugfix/login-error
```

---

Hotfix

```text
hotfix/payment-failure
```

---

Release

```text
release/v1.2.0
```

---

# Deleting Branches

Delete Local Branch

```bash
git branch -d payment-feature
```

---

Force Delete

```bash
git branch -D payment-feature
```

---

Delete Remote Branch

```bash
git push origin --delete payment-feature
```

---

# Common Interview Questions

## Why Don't Developers Work Directly On Main Branch?

### Short Answer

Because main represents production-ready code and direct changes can introduce bugs.

### Detailed Explanation

Feature branches isolate development work from production code. Changes are reviewed and tested before reaching main.

### Production Example

```text
Developer

↓

Feature Branch

↓

PR

↓

Review

↓

Main
```

### Follow-Up Questions

- What happens if someone pushes directly to main?
- How do branch protection rules help?
- What is trunk-based development?

---

## What Is A Git Branch?

### Short Answer

A branch is an independent line of development.

### Detailed Explanation

Branches allow developers to work on features, bug fixes, or experiments without affecting other branches.

### Production Example

```text
main

├── payment-feature

└── notification-feature
```

### Follow-Up Questions

- How are branches created?
- Are branches copies of code?
- How does Git store branches?

---

## What Is A Pull Request?

### Short Answer

A Pull Request is a request to merge changes from one branch into another.

### Detailed Explanation

PRs provide code review, collaboration, automated testing, and approval workflows before changes reach production branches.

### Production Example

```text
payment-feature

↓

Pull Request

↓

main
```

### Follow-Up Questions

- Why are PRs important?
- Who approves PRs?
- Can CI/CD run on PRs?

---

## Why Is CI/CD Integrated With Pull Requests?

### Short Answer

To automatically validate code before merging.

### Detailed Explanation

Builds, tests, linting, security scans, and quality checks run automatically whenever a PR is created or updated.

### Production Example

```text
PR

↓

Build

↓

Tests

↓

Security Scan

↓

Merge
```

### Follow-Up Questions

- What happens if tests fail?
- Can PR be merged if pipeline fails?
- What tools run these checks?

---

# Production Scenarios

## Scenario 1

Developer Accidentally Commits To Main.

### Best Practice

```text
Protect Main Branch
```

using branch protection rules.

---

## Scenario 2

Multiple Developers Working On Same Feature.

### Solution

Create:

```text
Feature Branch
```

for each developer.

---

## Scenario 3

PR Cannot Be Merged.

### Check

```text
Code Review Status

Pipeline Status

Merge Conflicts
```

---

## Scenario 4

Developer Finished Feature.

### Flow

```text
Push Branch

↓

Create PR

↓

Review

↓

Merge

↓

Delete Branch
```

---

## Scenario 5

Production Bug Requires Immediate Fix.

### Solution

```text
Hotfix Branch
```

---

# Key Takeaways

```text
Branches provide isolated development environments.

Main branch should contain production-ready code.

Developers should work in feature branches.

Pull Requests enable code review and collaboration.

Most organizations require PR approval before merge.

CI/CD pipelines commonly run on Pull Requests.

Branch protection rules help secure production branches.

Feature branch workflow is one of the most common Git workflows in modern software development.
```