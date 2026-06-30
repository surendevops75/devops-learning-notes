# Version Control and Git Fundamentals

## Introduction

As organizations grow, applications become larger and more complex.

Multiple developers work on the same project simultaneously.

Without proper version control:

- Code can be lost
- Changes can overwrite each other
- Collaboration becomes difficult
- Tracking history becomes impossible

Version Control Systems (VCS) solve these problems.

---

# Why Scripting?

Automation goals:

```text
Speed
Accuracy
Consistency
Scalability
Documentation
Reusability
```

---

## Speed

Manual tasks consume time.

Example:

```text
Install Software
Configure Server
Deploy Application
```

Automation executes tasks much faster.

---

## Accuracy

Humans make mistakes.

Example:

```bash
dnf install nginx -y
```

Mistakenly typing:

```bash
dnf install nginxxx -y
```

causes failures.

Scripts reduce human errors.

---

## Consistency

Manual execution may vary between engineers.

Automation ensures:

```text
Same Process
Same Result
```

every time.

---

## Scalability

Manual work becomes difficult when managing:

```text
10 Servers
100 Servers
1000 Servers
```

Scripts can execute the same process across an unlimited number of servers.

---

## Documentation

Scripts act as documentation.

A script shows:

```text
What Happened
When It Happened
How It Happened
```

Deployment history becomes easier to understand.

---

## Reusability

Once written:

```text
Write Once
Use Multiple Times
```

---

# Where Should Scripts Be Stored?

Storing scripts on a local machine creates problems.

Issues:

- Security Risks
- Cannot Share Easily
- No Backup
- No Version History

Solution:

```text
Version Control Systems
```

---

# What is a Version Control System?

A Version Control System (VCS) tracks changes made to files over time.

Benefits:

- Track Changes
- Maintain History
- Team Collaboration
- Rollback Capability
- Backup and Recovery

---

# Types of Version Control Systems

There are two major types:

1. Centralized Version Control
2. Distributed Version Control

---

# Centralized Version Control

All code is stored in a central server.

Architecture:

```text
Developer A
      │
      ▼
Central Server
      ▲
      │
Developer B
```

---

# Example

```text
SVN (Subversion)
```

is a centralized version control system.

---

# Problems with Centralized Systems

## Single Point of Failure

If the server goes down:

```text
Nobody Can Access Code
```

---

## Dependency on Central Server

All operations depend on server availability.

---

## Network Dependency

Without network access:

```text
Work Stops
```

---

# Real World Example

## Central Library

Imagine a city with only one library.

Problems:

1. Library Closed
2. Building Collapse
3. Overcrowding
4. Limited Availability

Everyone depends on a single location.

This is similar to centralized version control.

---

# Distributed Version Control

Every developer has a complete copy of the repository.

Architecture:

```text
Developer A
      │
      ▼
Local Repository

Developer B
      │
      ▼
Local Repository

      │
      ▼

Central Repository
```

---

# Advantages

## No Single Point of Failure

Every developer has a complete copy.

---

## Offline Work

Developers can continue working without internet.

---

## Faster Operations

Many operations occur locally.

---

## Better Collaboration

Multiple developers can work independently.

---

# Git

Git is the most popular distributed version control system.

Created by:

```text
Linus Torvalds
```

Creator of:

```text
Linux
Git
```

---

# Why Git Was Created?

Linux kernel development involved thousands of contributors.

Git was created to provide:

- Speed
- Reliability
- Distributed Development
- Efficient Collaboration

---

# Git Architecture

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

# Repository

A repository is a storage location for source code and project files.

Contains:

- Source Code
- Documentation
- Configuration Files
- History

---

# Normal Folder vs Git Repository

## Normal Folder

```text
project/
```

Only files exist.

No history.

---

## Git Repository

```text
project/
└── .git/
```

Contains:

```text
History
Branches
Commits
Configuration
```

---

# Hidden .git Directory

When a folder becomes a Git repository:

```text
.git
```

is created.

This hidden directory stores:

- Commit History
- Branch Information
- Repository Metadata

---

# How to Check Hidden Files

```bash
ls -la
```

Example:

```text
.git
README.md
app.py
```

---

# SVN vs Git

| Feature | SVN | Git |
|----------|-----|-----|
| Architecture | Centralized | Distributed |
| Offline Work | Limited | Full Support |
| Speed | Slower | Faster |
| Local History | No | Yes |
| Single Point of Failure | Yes | No |

---

# Why Git Became Popular?

- Distributed Architecture
- Fast Performance
- Easy Branching
- Better Collaboration
- Strong Community Support

---

# Common Git Terms

## Repository

Storage for project files.

---

## Commit

Snapshot of code changes.

---

## Branch

Independent line of development.

---

## Clone

Download repository locally.

---

## Push

Upload changes to remote repository.

---

## Pull

Download latest changes from remote repository.

---

# Installing Git

RHEL / Amazon Linux:

```bash
dnf install git -y
```

---

Ubuntu:

```bash
apt install git -y
```

---

Verify Installation:

```bash
git --version
```

---

# Initial Git Configuration

Configure username:

```bash
git config --global user.name "Ramesh"
```

---

Configure email:

```bash
git config --global user.email "ramesh@example.com"
```

---

Verify Configuration:

```bash
git config --list
```

---

# Frequently Asked Interview Questions

## What is Version Control?

A system used to track changes in files over time.

---

## Why Do We Need Version Control?

- Collaboration
- History Tracking
- Backup
- Rollback

---

## What is Git?

A distributed version control system created by Linus Torvalds.

---

## What is SVN?

A centralized version control system.

---

## Difference Between SVN and Git?

SVN is centralized.

Git is distributed.

---

## What is a Repository?

A storage location containing source code and project history.

---

## What is the .git Directory?

A hidden folder containing Git metadata and history.

---

## Who Created Git?

Linus Torvalds.

---

## What Are the Advantages of Distributed Version Control?

- No Single Point of Failure
- Offline Work
- Faster Operations
- Better Collaboration

---

# Key Takeaways

- Automation requires proper script storage and version management.
- Version Control Systems track changes over time.
- SVN is a centralized version control system.
- Git is a distributed version control system.
- Git was created by Linus Torvalds.
- Repositories store source code and history.
- Git repositories contain a hidden `.git` directory.
- Distributed version control eliminates single points of failure.
- Git is the industry standard version control system used in modern DevOps practices.