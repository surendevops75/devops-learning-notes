# Git Branching Strategies

## Introduction

As teams grow, managing code changes becomes difficult.

---

Example

```text
50 Developers

↓

Same Repository

↓

Multiple Features

↓

Bug Fixes

↓

Production Releases
```

---

Question

```text
How Do Teams Work Without Breaking Production?
```

---

Answer

```text
Branching Strategy
```

---

# What Is A Branching Strategy?

A branching strategy is a set of rules that defines:

```text
How Branches Are Created

How Development Happens

How Code Is Merged

How Releases Are Managed
```

---

Without A Strategy

```text
Confusion

Conflicts

Broken Deployments

Production Risks
```

---

With A Strategy

```text
Controlled Development

Better Collaboration

Safer Releases
```

---

# Why Do We Need Branching Strategies?

Suppose:

```text
Developer A

↓

Payment Feature
```

---

```text
Developer B

↓

Notification Feature
```

---

```text
Developer C

↓

Production Bug Fix
```

---

Without Rules

```text
Everyone Pushes Everywhere
```

---

Result

```text
Chaos
```

---

Need

```text
Structured Development Process
```

---

# Popular Branching Strategies

Most Common:

```text
Git Flow

Feature Branching

Trunk Based Development
```

---

# 1. Git Flow

Git Flow is one of the oldest and most structured branching strategies.

---

Created For:

```text
Large Enterprise Releases
```

---

Uses Multiple Branch Types.

---

# Git Flow Architecture

```text
main

↓

develop

↓

feature/*
```

---

Additional Branches

```text
release/*

hotfix/*
```

---

Complete Flow

```text
main

↓

develop

├── feature/payment

├── feature/cart

└── feature/orders

↓

release/v1.0

↓

main
```

---

# Branch Types In Git Flow

## Main Branch

Contains:

```text
Production Code
```

---

## Develop Branch

Contains:

```text
Latest Development Changes
```

---

## Feature Branch

Used For:

```text
New Features
```

---

Example

```text
feature/payment
```

---

## Release Branch

Used For:

```text
Release Preparation
```

---

Example

```text
release/v2.0
```

---

## Hotfix Branch

Used For:

```text
Production Fixes
```

---

Example

```text
hotfix/payment-bug
```

---

# Git Flow Workflow

Step 1

Create Feature Branch

```text
feature/payment
```

---

Step 2

Develop Feature

---

Step 3

Merge To

```text
develop
```

---

Step 4

Create

```text
release/v1.0
```

---

Step 5

Testing

---

Step 6

Merge To

```text
main
```

---

Step 7

Deploy

---

# Advantages Of Git Flow

```text
Highly Structured

Supports Releases

Supports Hotfixes

Good Audit Trail
```

---

# Disadvantages Of Git Flow

```text
Too Many Branches

Complex

Slower Delivery

More Merge Conflicts
```

---

# Where Git Flow Is Used?

Common In

```text
Banks

Insurance

Large Enterprises
```

---

# 2. Feature Branching

Most Common Strategy Today.

---

Every New Feature Gets Its Own Branch.

---

Architecture

```text
main

├── feature/payment

├── feature/cart

├── feature/orders

└── feature/search
```

---

# Feature Branch Workflow

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

Pull Request

↓

Review

↓

Merge To Main
```

---

# Example

Create Branch

```bash
git checkout -b feature/payment
```

---

Develop Feature.

---

Push Branch.

---

Create PR.

---

Review.

---

Merge.

---

Delete Branch.

---

# Why Feature Branching Is Popular?

Simple.

---

Easy To Understand.

---

Works Well With:

```text
GitHub

GitLab

Bitbucket
```

---

Supports:

```text
Pull Requests

Code Reviews

CI/CD
```

---

# Advantages

```text
Easy To Learn

Isolated Development

Supports Code Reviews

Safe For Teams
```

---

# Disadvantages

```text
Long-Lived Branches

Can Cause Conflicts
```

---

# Production Example

DevOps Team

Working On

```text
Terraform Infrastructure
```

---

Create

```text
feature/eks-upgrade
```

---

Development Completed.

---

PR Created.

---

CI/CD Passed.

---

Merged To Main.

---

# 3. Trunk Based Development

Modern Engineering Strategy.

---

Popular At

```text
Google

Netflix

Meta
```

---

Main Idea

```text
Merge Frequently
```

---

Branches Live For:

```text
Hours

Not Weeks
```

---

# Architecture

```text
main

↓

Small Feature Branch

↓

Merge Quickly

↓

main
```

---

Sometimes Teams Commit Directly To Main.

---

Protected By

```text
Automated Testing

CI/CD
```

---

# Trunk Based Workflow

```text
Create Small Branch

↓

Develop

↓

Commit

↓

CI/CD

↓

Merge Quickly

↓

Delete Branch
```

---

# Why Trunk Based Development?

Goal

```text
Continuous Integration
```

---

Avoid

```text
Long Running Branches
```

---

Reduce

```text
Merge Conflicts
```

---

# Advantages

```text
Fast Releases

Continuous Delivery

Fewer Conflicts

Better CI/CD
```

---

# Disadvantages

Requires

```text
Strong Testing

Strong CI/CD

Mature Team
```

---

Without Good Testing

```text
Production Risk
```

---

# Comparison

## Git Flow

```text
Complex

Release Oriented

Many Branches
```

---

## Feature Branching

```text
Simple

Most Common

PR Based
```

---

## Trunk Based Development

```text
Fast

CI/CD Driven

Modern
```

---

# Which Strategy Should You Use?

## Small Teams

```text
Feature Branching
```

---

## Medium Teams

```text
Feature Branching
```

---

## Enterprise Releases

```text
Git Flow
```

---

## Mature DevOps Teams

```text
Trunk Based Development
```

---

# Which Strategy Is Most Common Today?

Most Companies Use:

```text
Feature Branching
```

---

Typical Workflow

```text
Feature Branch

↓

PR

↓

Code Review

↓

CI/CD

↓

Merge

↓

Deployment
```

---

# Real Production Example

Application Team

```text
feature/payment
```

---

Platform Team

```text
feature/eks-upgrade
```

---

Security Team

```text
feature/waf-policy
```

---

Each Team Works Independently.

---

PR Process Controls Quality.

---

# DevOps Perspective

Typical GitHub Workflow

```text
main

↓

feature/argo-upgrade

↓

Commit

↓

Push

↓

PR

↓

GitHub Actions

↓

Approval

↓

Merge

↓

Deployment
```

---

This is what most DevOps Engineers use daily.

---

# Common Interview Questions

## What Is A Branching Strategy?

### Short Answer

A branching strategy defines how teams create, manage, and merge branches during software development.

### Detailed Explanation

Branching strategies provide structure and help teams collaborate safely while protecting production code.

### Production Example

```text
Feature Branch

↓

PR

↓

Review

↓

Merge
```

### Follow-Up Questions

- Why do we need branching strategies?
- What problems do they solve?
- Which strategy does your company use?

---

## What Is Git Flow?

### Short Answer

Git Flow is a branching strategy that uses main, develop, feature, release, and hotfix branches.

### Detailed Explanation

Git Flow provides a structured release process and is commonly used in traditional enterprise environments.

### Production Example

```text
feature

↓

develop

↓

release

↓

main
```

### Follow-Up Questions

- What is a release branch?
- What is a hotfix branch?
- Why is Git Flow less popular today?

---

## What Is Feature Branching?

### Short Answer

Each feature is developed in its own branch and merged through a pull request.

### Detailed Explanation

Feature Branching isolates development work and enables code review and automated validation.

### Production Example

```text
feature/payment

↓

PR

↓

main
```

### Follow-Up Questions

- Why is Feature Branching popular?
- What are its disadvantages?
- How does it support CI/CD?

---

## What Is Trunk Based Development?

### Short Answer

Developers integrate changes into the main branch frequently using short-lived branches.

### Detailed Explanation

Trunk Based Development promotes continuous integration and rapid delivery.

### Production Example

```text
Small Branch

↓

Merge Same Day

↓

main
```

### Follow-Up Questions

- Why do Google and Netflix use it?
- How does CI/CD support it?
- What are the risks?

---

## Which Branching Strategy Is Best?

### Short Answer

There is no universal best strategy. The choice depends on team size, release frequency, and CI/CD maturity.

### Detailed Explanation

Feature Branching is the most common. Git Flow is suited for release-heavy environments. Trunk Based Development is ideal for highly automated DevOps teams.

### Production Example

```text
Startup

↓

Feature Branching
```

```text
Bank

↓

Git Flow
```

```text
Netflix

↓

Trunk Based Development
```

### Follow-Up Questions

- Which strategy have you used?
- Why did your team choose it?
- What challenges did you face?

---

# Production Scenarios

## Scenario 1

50 Developers Working On Same Repository.

### Solution

```text
Feature Branching
```

---

## Scenario 2

Company Releases Once Every Quarter.

### Solution

```text
Git Flow
```

---

## Scenario 3

Need Faster Releases.

### Solution

```text
Trunk Based Development
```

---

## Scenario 4

Too Many Merge Conflicts.

### Recommendation

```text
Short-Lived Branches

Frequent Integration
```

---

## Scenario 5

Production Bug Found.

### Solution

```text
hotfix/payment-issue
```

---

Test.

---

PR.

---

Merge.

---

Deploy.

---

# Key Takeaways

```text
Branching strategies provide structure for team collaboration.

Git Flow uses main, develop, feature, release, and hotfix branches.

Feature Branching is the most commonly used strategy today.

Trunk Based Development emphasizes rapid integration and continuous delivery.

Feature Branching works well with Pull Requests and CI/CD.

Git Flow is suitable for structured enterprise release processes.

Trunk Based Development requires strong automation and testing.

Choosing the right branching strategy depends on team maturity and release requirements.
```