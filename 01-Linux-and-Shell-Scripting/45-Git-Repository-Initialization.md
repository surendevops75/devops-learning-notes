# Git Repository Initialization

## Introduction

Until now, we have learned:

- Git Fundamentals
- GitHub Workflow
- Clone
- Commit
- Push
- Pull
- Branching

A common real-world scenario is:

```text
You already have a project folder

But it is NOT a Git repository
```

In such cases, we need to convert the existing folder into a Git repository.

---

# What is a Git Repository?

A Git repository is a directory that Git can track.

Git stores:

- File History
- Commits
- Branches
- Tags
- Configuration

inside a hidden directory called:

```text
.git
```

---

# Normal Folder vs Git Repository

## Normal Folder

```text
project/
├── app.py
├── README.md
└── config.yaml
```

Git cannot track changes.

---

## Git Repository

```text
project/
├── .git
├── app.py
├── README.md
└── config.yaml
```

Git can now:

- Track Changes
- Create Commits
- Create Branches
- Push to GitHub

---

# Existing Folder to Repository

Suppose you already have:

```text
expense-project/
├── backend/
├── frontend/
└── scripts/
```

and now you want Git to manage it.

---

# git init

Command:

```bash
git init
```

Purpose:

```text
Initialize Current Folder
As a Git Repository
```

---

# Example

Create a directory:

```bash
mkdir demo-project
```

---

Move into directory:

```bash
cd demo-project
```

---

Initialize repository:

```bash
git init
```

Output:

```text
Initialized empty Git repository
```

---

# What Happens Internally?

Git creates:

```text
.git
```

hidden directory.

---

# Verify Repository

Display hidden files:

```bash
ls -la
```

Output:

```text
.git
```

visible in the folder.

---

# Understanding .git Directory

The `.git` directory contains:

- Commit History
- Branch Information
- Repository Configuration
- Object Database

---

# Important Note

Never manually edit files inside:

```text
.git
```

Git manages them automatically.

---

# Repository Structure

Example:

```text
project/
│
├── .git/
├── app.py
├── README.md
└── config.yaml
```

---

# Git Workflow After Initialization

```text
Working Directory
       │
       ▼
Staging Area
       │
       ▼
Local Repository
       │
       ▼
Remote Repository
```

---

# Check Repository Status

```bash
git status
```

Shows:

- Tracked Files
- Untracked Files
- Modified Files

---

# Example

Create file:

```bash
touch app.sh
```

Check status:

```bash
git status
```

Output:

```text
Untracked files:
app.sh
```

---

# Add File to Staging Area

```bash
git add app.sh
```

---

Add all files:

```bash
git add .
```

---

# What is Staging Area?

A temporary area between:

```text
Working Directory
        and
Local Repository
```

---

# Why Staging Area?

Suppose:

```text
20 Files Modified
```

Only:

```text
10 Files Ready
```

Stage only those 10 files.

---

# Commit Changes

```bash
git commit -m "Initial commit"
```

---

# What is a Commit?

A commit is a snapshot of code at a specific point in time.

Git stores:

- Author
- Date
- Time
- Commit Message

---

# View Commit History

```bash
git log
```

Displays:

```text
Commit ID
Author
Date
Message
```

---

# Configure Git User

Before committing, configure identity.

---

# Set Username

```bash
git config --global user.name "Your Name"
```

Example:

```bash
git config --global user.name "Surendra"
```

---

# Set Email

```bash
git config --global user.email "your-email@example.com"
```

Example:

```bash
git config --global user.email "surendra@example.com"
```

---

# Verify Configuration

```bash
git config --list
```

---

# Connecting to GitHub

After local commits:

Create repository in GitHub.

Example:

```text
my-project
```

---

# Add Remote Repository

```bash
git remote add origin <repo-url>
```

Example:

```bash
git remote add origin https://github.com/user/my-project.git
```

---

# Verify Remote

```bash
git remote -v
```

---

# Push Code to GitHub

```bash
git push -u origin main
```

---

# Authentication

Modern GitHub authentication usually happens through:

```text
Browser Sign-In
```

or

```text
Personal Access Token (PAT)
```

---

# Clone vs Init

## Clone

Used when repository already exists.

```bash
git clone <repo-url>
```

Downloads code.

---

## Init

Used when repository does not exist locally as a Git repository.

```bash
git init
```

Creates a new Git repository.

---

# Example Comparison

Clone:

```text
GitHub Repo Exists
        │
        ▼
Download Repository
```

---

Init:

```text
Folder Exists
        │
        ▼
Convert Folder Into Git Repository
```

---

# Common Git Commands

Initialize Repository:

```bash
git init
```

---

Check Status:

```bash
git status
```

---

Add File:

```bash
git add file-name
```

---

Add All Files:

```bash
git add .
```

---

Commit Changes:

```bash
git commit -m "message"
```

---

View History:

```bash
git log
```

---

View Remotes:

```bash
git remote -v
```

---

Push Changes:

```bash
git push origin main
```

---

# Real DevOps Example

Suppose you create:

```text
roboshop-shell-scripts/
```

and add:

```text
mongodb.sh
mysql.sh
redis.sh
catalogue.sh
```

Convert it into a repository:

```bash
git init
```

---

Add files:

```bash
git add .
```

---

Commit:

```bash
git commit -m "Initial shell scripts"
```

---

Push:

```bash
git push origin main
```

---

# Frequently Asked Interview Questions

## What is git init?

Initializes the current folder as a Git repository.

---

## What Does git init Create?

```text
.git
```

hidden directory.

---

## What is .git Directory?

Stores repository metadata, commits, branches, and history.

---

## Difference Between git clone and git init?

| git clone | git init |
|------------|-----------|
| Downloads Existing Repository | Creates New Repository |
| Requires Remote Repo | Works on Existing Folder |

---

## How Do You Verify a Repository?

```bash
ls -la
```

Look for:

```text
.git
```

---

## How Do You Check Repository Status?

```bash
git status
```

---

## How Do You View Commit History?

```bash
git log
```

---

# Key Takeaways

- `git init` converts an existing folder into a Git repository.
- Git repositories contain a hidden `.git` directory.
- `.git` stores commits, branches, and repository metadata.
- Use `git status` to check repository state.
- Use `git add` to stage files.
- Use `git commit` to create snapshots.
- Use `git log` to view commit history.
- Use `git remote add origin` to connect GitHub repositories.
- `git init` is used for existing folders, while `git clone` downloads existing repositories.