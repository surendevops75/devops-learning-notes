# Git Introduction And Workflow

## Introduction

Before Git, software teams faced a major problem:

```text
Multiple Developers

Working On Same Code
```

---

Example

Developer A modifies:

```text
login.py
```

---

Developer B modifies:

```text
login.py
```

---

Question:

```text
Which Version Is Correct?
```

---

Managing source code manually becomes difficult.

---

Need:

```text
Version Control System
```

---

# What Is Version Control?

Version Control is a system that tracks:

```text
Code Changes

Who Changed It

When It Was Changed

Why It Was Changed
```

---

Benefits

```text
Collaboration

History Tracking

Rollback

Backup

Branching
```

---

# Why Git Was Created?

Before Git, many organizations used:

```text
SVN

CVS
```

---

These are:

```text
Centralized Version Control Systems
```

---

Problem

```text
Single Central Server
```

---

If Server Goes Down

```text
Development Stops
```

---

Need:

```text
Distributed Version Control
```

---

Git Solves This.

---

# Centralized Version Control

Architecture

```text
Developer

↓

Central Server

↓

Code Repository
```

---

Examples

```text
SVN

CVS
```

---

Problems

```text
Single Point Of Failure

Slow Operations

Network Dependency
```

---

# Distributed Version Control

Architecture

```text
Developer

↓

Local Repository

↓

Remote Repository
```

---

Every Developer Has:

```text
Complete Repository Copy
```

---

Benefits

```text
Fast

Reliable

Offline Work

No Single Point Of Failure
```

---

Git Is A Distributed Version Control System.

---

# What Is Git?

Git is a:

```text
Distributed Version Control System
```

used to:

```text
Track Source Code

Manage Changes

Collaborate With Teams
```

---

Created By:

```text
Linus Torvalds
```

---

Originally Developed For:

```text
Linux Kernel
```

---

# What Is A Repository?

Repository (Repo) is a place where Git stores:

```text
Files

Folders

Commit History
```

---

Think Of Repository As:

```text
Project Database
```

---

Example

```text
roboshop-app

terraform-eks

kubernetes-manifests
```

---

# Types Of Repositories

## Local Repository

Exists On Your Machine.

---

Example

```text
Laptop

Desktop

EC2 Instance
```

---

## Remote Repository

Exists On Remote Platform.

---

Examples

```text
GitHub

GitLab

Bitbucket
```

---

# Git Architecture

One Of The Most Important Concepts.

---

Whenever you modify code:

```text
Working Directory

↓

Staging Area

↓

Local Repository

↓

Remote Repository
```

---

Understanding this flow explains most Git commands.

---

# Working Directory

The place where developers write code.

---

Example

```text
app.py

main.tf

deployment.yaml
```

---

Changes here are:

```text
Not Tracked Yet
```

---

# Staging Area

Temporary area before commit.

---

Purpose

```text
Select Which Changes

Should Be Committed
```

---

Example

```bash
git add app.py
```

---

File Moves To:

```text
Staging Area
```

---

# Local Repository

Stores:

```text
Commit History
```

---

Created Using:

```bash
git commit
```

---

Commits exist only on your machine.

---

# Remote Repository

Shared repository used by team members.

---

Examples

```text
GitHub

GitLab

Bitbucket
```

---

Changes reach remote repository using:

```bash
git push
```

---

# Complete Git Workflow

Developer Creates Code.

---

Flow

```text
Working Directory

↓

git add

↓

Staging Area

↓

git commit

↓

Local Repository

↓

git push

↓

Remote Repository
```

---

This is the most important Git workflow.

---

# Git Init

Creates a new Git repository.

---

Command

```bash
git init
```

---

Example

```bash
mkdir project

cd project

git init
```

---

Output

```text
Initialized empty Git repository
```

---

Git Creates:

```text
.git
```

directory.

---

# What Is .git Directory?

Contains:

```text
Commits

Branches

Configuration

History
```

---

This is the actual Git database.

---

# Git Clone

Used to copy an existing repository.

---

Command

```bash
git clone <repository-url>
```

---

Example

```bash
git clone https://github.com/company/project.git
```

---

Flow

```text
Remote Repository

↓

Local Repository

↓

Working Directory
```

---

# Difference Between Git Init And Git Clone

## Git Init

Creates New Empty Repository.

---

Example

```bash
git init
```

---

## Git Clone

Copies Existing Repository.

---

Example

```bash
git clone https://github.com/company/project.git
```

---

# Git Status

Shows repository status.

---

Command

```bash
git status
```

---

Displays

```text
Modified Files

Staged Files

Untracked Files
```

---

One Of The Most Frequently Used Commands.

---

# Git Add

Adds files to staging area.

---

Command

```bash
git add app.py
```

---

Add All Files

```bash
git add .
```

---

Flow

```text
Working Directory

↓

Staging Area
```

---

# Git Commit

Stores changes in local repository.

---

Command

```bash
git commit -m "Added login feature"
```

---

Purpose

```text
Create Checkpoint
```

---

Flow

```text
Staging Area

↓

Local Repository
```

---

# Git Push

Uploads commits to remote repository.

---

Command

```bash
git push origin main
```

---

Flow

```text
Local Repository

↓

Remote Repository
```

---

# Git Pull

Downloads latest changes from remote repository.

---

Command

```bash
git pull origin main
```

---

Flow

```text
Remote Repository

↓

Local Repository

↓

Working Directory
```

---

# Production Workflow Example

Suppose Developer Joins Project.

---

Step 1

Clone Repository

```bash
git clone https://github.com/company/project.git
```

---

Step 2

Make Changes

```text
Update Application Code
```

---

Step 3

Add Changes

```bash
git add .
```

---

Step 4

Commit Changes

```bash
git commit -m "Added payment validation"
```

---

Step 5

Push Changes

```bash
git push origin main
```

---

Repository Updated.

---

# Git Configuration

Configure Username

```bash
git config --global user.name "Surendra"
```

---

Configure Email

```bash
git config --global user.email "surendra@example.com"
```

---

View Configuration

```bash
git config --list
```

---

# Common Commands Summary

Check Status

```bash
git status
```

---

Initialize Repository

```bash
git init
```

---

Clone Repository

```bash
git clone <repo-url>
```

---

Add Files

```bash
git add .
```

---

Commit Changes

```bash
git commit -m "message"
```

---

Push Changes

```bash
git push origin main
```

---

Pull Changes

```bash
git pull origin main
```

---

# Common Interview Questions

## What Is Git?

### Short Answer

Git is a distributed version control system used to track and manage source code changes.

### Detailed Explanation

Git allows multiple developers to collaborate on the same codebase while maintaining complete change history and supporting rollback, branching, and merging.

### Production Example

```text
Developer

↓

Commit

↓

Push

↓

GitHub

↓

CI/CD Pipeline
```

### Follow-Up Questions

- Why Git?
- Why Not SVN?
- Who Created Git?

---

## What Is The Difference Between Git Init And Git Clone?

### Short Answer

Git init creates a new repository, while git clone copies an existing repository.

### Detailed Explanation

Git init starts version control for a new project. Git clone downloads an existing repository including its commit history.

### Production Example

```bash
git init
```

New Project.

```bash
git clone repo-url
```

Existing Project.

### Follow-Up Questions

- Which creates .git directory?
- Which downloads history?
- Which is used most in companies?

---

## What Is The Difference Between Git Add And Git Commit?

### Short Answer

Git add moves changes to staging area, while git commit stores them in local repository.

### Detailed Explanation

Git add selects changes to be included in a commit. Git commit creates a permanent checkpoint in local Git history.

### Production Example

```text
Working Directory

↓

git add

↓

Staging Area

↓

git commit

↓

Local Repository
```

### Follow-Up Questions

- Can we commit without add?
- What is staging area?
- Why does Git use staging?

---

## What Is The Difference Between Git Push And Git Pull?

### Short Answer

Git push sends local commits to remote repository, while git pull downloads remote changes.

### Detailed Explanation

Push uploads your work. Pull synchronizes your local repository with the remote repository.

### Production Example

```text
git push

Local → Remote
```

```text
git pull

Remote → Local
```

### Follow-Up Questions

- What happens during pull?
- Can pull cause conflicts?
- When should pull be used?

---

# Production Scenarios

## Scenario 1

Developer Modified Files But Forgot To Commit.

### Check

```bash
git status
```

---

## Scenario 2

Developer Wants To Share Changes.

### Use

```bash
git push
```

---

## Scenario 3

Repository Already Exists On GitHub.

### Use

```bash
git clone
```

---

## Scenario 4

Team Member Pushed New Changes.

### Update Local Repository

```bash
git pull
```

---

## Scenario 5

Need To Start Git Tracking For New Project.

### Use

```bash
git init
```

---

# Key Takeaways

```text
Git is a distributed version control system.

Git tracks code changes and maintains history.

Repository stores files and commit history.

Git architecture consists of Working Directory, Staging Area, Local Repository, and Remote Repository.

git init creates a new repository.

git clone copies an existing repository.

git add moves changes to staging area.

git commit stores changes in local repository.

git push uploads changes to remote repository.

git pull downloads latest changes from remote repository.

Understanding the Git workflow is essential before learning branches, pull requests, merge, and rebase.
```