# GitHub Fundamentals

## Introduction

Git is a:

```text
Version Control System
```

---

GitHub is a:

```text
Code Hosting Platform
```

built on top of Git.

---

Think Of It Like:

```text
Git

↓

Engine
```

---

```text
GitHub

↓

Car
```

---

Git performs:

```text
Version Control
```

---

GitHub provides:

```text
Remote Repositories

Collaboration

Pull Requests

Code Reviews

Security

CI/CD Integrations
```

---

# Problem Statement

Suppose You Created A Project

```text
Roboshop
```

---

Stored Locally On Laptop.

---

Problems

```text
No Backup

No Team Collaboration

No Code Review

No Central Repository
```

---

Need A Platform To:

```text
Store Code

Collaborate

Track Changes

Manage Development
```

---

Answer

```text
GitHub
```

---

# What Is GitHub?

GitHub is a cloud platform that hosts Git repositories.

---

Developers Can:

```text
Store Code

Share Code

Collaborate

Review Changes

Track Issues

Manage Releases
```

---

GitHub Is Used By:

```text
Developers

DevOps Engineers

Cloud Engineers

Open Source Communities

Enterprises
```

---

# Git Vs GitHub

## Git

```text
Version Control Tool
```

Examples

```bash
git add
git commit
git merge
git rebase
```

---

Runs Locally.

---

## GitHub

```text
Remote Hosting Platform
```

Examples

```text
Repositories

Pull Requests

Issues

Actions

Code Reviews
```

---

Runs In Cloud.

---

# What Is A Repository?

Repository (Repo) is:

```text
A Project Storage Location
```

---

Contains

```text
Source Code

Documentation

Branches

Commit History
```

---

Example

```text
roboshop-cart

roboshop-user

roboshop-payment
```

---

# Public Repository

Anyone Can:

```text
View

Clone

Fork
```

---

Examples

```text
Open Source Projects
```

---

# Private Repository

Only Authorized Users Can Access.

---

Used For

```text
Enterprise Projects

Internal Applications
```

---

# Creating Repository

GitHub

↓

New Repository

↓

Repository Name

↓

Public Or Private

↓

Create Repository
```

---

# Clone Repository

Copy Remote Repository To Local Machine.

---

Command

```bash
git clone <repository-url>
```

---

Example

```bash
git clone https://github.com/company/roboshop.git
```

---

# Remote Repository

GitHub Repository Acts As:

```text
Central Repository
```

---

Developers Push Changes To:

```text
GitHub
```

---

Other Developers Pull Changes From:

```text
GitHub
```

---

# Fork Repository

Fork Creates:

```text
Personal Copy
```

of another repository.

---

Common In:

```text
Open Source Projects
```

---

Flow

```text
Original Repository

↓

Fork

↓

Your Repository
```

---

# Clone Vs Fork

## Clone

```text
Copy Repository To Local Machine
```

---

## Fork

```text
Create Personal Copy On GitHub
```

---

# What Is README.md?

README Is:

```text
Project Documentation
```

---

Usually Contains

```text
Project Overview

Installation Steps

Usage Instructions

Architecture

Contributors
```

---

First File Most Recruiters See.

---

# What Are Stars?

GitHub Users Can:

```text
Star Repository
```

---

Similar To

```text
Bookmark
```

---

Indicates

```text
Popularity

Community Interest
```

---

# What Is Watch?

Watch Means

```text
Receive Notifications
```

for repository activities.

---

Examples

```text
New Issues

Pull Requests

Releases
```

---

# What Are Issues?

Issues Are Used For:

```text
Bug Tracking

Feature Requests

Tasks

Discussions
```

---

Example

```text
Payment Service Fails During Checkout
```

---

Create Issue.

---

Assign Developer.

---

Track Progress.

---

# Labels

Used To Categorize Issues.

---

Examples

```text
Bug

Enhancement

Documentation

Security

High Priority
```

---

# Milestones

Used To Group Issues.

---

Example

```text
Release v1.0
```

---

Contains

```text
10 Features

5 Bug Fixes
```

---

# GitHub Projects

Project Management Tool.

---

Provides

```text
Kanban Board

Task Tracking

Sprint Planning
```

---

Example

```text
To Do

In Progress

Done
```

---

# Organizations

Used By Companies.

---

Example

```text
Company

↓

Organization

↓

Repositories
```

---

Examples

```text
Netflix

Google

Microsoft
```

---

# Teams

Organizations Can Create:

```text
Teams
```

---

Examples

```text
DevOps Team

Backend Team

Frontend Team

Security Team
```

---

Permissions Managed At Team Level.

---

# Collaborators

Users Who Can Access Repository.

---

Permissions

```text
Read

Write

Admin
```

---

# Repository Structure Example

```text
roboshop

├── src/
├── docs/
├── terraform/
├── kubernetes/
├── README.md
├── .gitignore
└── LICENSE
```

---

# What Is .gitignore?

Used To Exclude Files.

---

Examples

```text
.log

.env

node_modules

terraform.tfstate
```

---

Prevent Sensitive Files From Being Committed.

---

# Real Production Example

Company

```text
Microgate Technologies
```

---

Organization

```text
microgate
```

---

Repositories

```text
roboshop-app

roboshop-infra

roboshop-eks

roboshop-argocd
```

---

Developers Collaborate Through GitHub.

---

# Useful GitHub Features

```text
Repositories

Branches

Pull Requests

Issues

Projects

Releases

Actions

Security
```

---

# Common Interview Questions

## What Is GitHub?

### Short Answer

GitHub is a cloud-based platform used to host and manage Git repositories.

### Detailed Explanation

GitHub extends Git by providing collaboration features such as pull requests, code reviews, issues, releases, and access control.

### Production Example

```text
Developer

↓

Push Code

↓

GitHub Repository

↓

Team Collaboration
```

### Follow-Up Questions

```text
Difference Between Git And GitHub?

What Are GitHub Alternatives?

Why Do Organizations Use GitHub?
```

---

## Difference Between Git And GitHub?

### Short Answer

Git is a version control system, while GitHub is a platform that hosts Git repositories.

### Detailed Explanation

Git performs version control operations locally, whereas GitHub provides centralized collaboration and repository management.

### Production Example

```text
Git

↓

Local Operations
```

```text
GitHub

↓

Remote Collaboration
```

### Follow-Up Questions

```text
Can Git Work Without GitHub?

What Are Other Git Hosting Platforms?

How Does GitHub Use Git?
```

---

## What Is Fork In GitHub?

### Short Answer

Fork creates a personal copy of another repository in your GitHub account.

### Detailed Explanation

Forking is commonly used in open-source projects where contributors do not have direct write access.

### Production Example

```text
Open Source Project

↓

Fork

↓

Make Changes

↓

Pull Request
```

### Follow-Up Questions

```text
Difference Between Fork And Clone?

When Should Fork Be Used?

Can Forks Be Synchronized?
```

---

## What Is .gitignore?

### Short Answer

A file used to exclude specific files and folders from Git tracking.

### Detailed Explanation

It prevents unnecessary, generated, or sensitive files from being committed.

### Production Example

```text
.env

terraform.tfstate

node_modules
```

### Follow-Up Questions

```text
Can Already Tracked Files Be Ignored?

Why Ignore Terraform State Files?

How Does Git Process .gitignore?
```

---

# Production Scenarios

## Scenario 1

Need Central Repository For Team.

### Solution

```text
Create GitHub Repository
```

---

## Scenario 2

Need To Contribute To Open Source Project.

### Solution

```text
Fork Repository

↓

Clone

↓

Make Changes

↓

Pull Request
```

---

## Scenario 3

Need To Prevent Secrets From Being Committed.

### Solution

```text
Use .gitignore
```

---

## Scenario 4

Need To Track Product Bugs.

### Solution

```text
GitHub Issues
```

---

## Scenario 5

Need Sprint Tracking.

### Solution

```text
GitHub Projects
```

---

# Key Takeaways

```text
GitHub is a cloud platform for hosting Git repositories.

Git and GitHub are different technologies.

Repositories store source code and history.

Repositories can be public or private.

Fork creates a personal copy of a repository.

Clone copies a repository to a local machine.

README.md provides project documentation.

Issues help track bugs and tasks.

Organizations and Teams support enterprise collaboration.

.gitignore prevents unwanted files from being tracked.
```