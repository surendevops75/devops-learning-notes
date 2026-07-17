# Git Squash And Interactive Rebase

## Introduction

During feature development, developers often create many commits.

Example

```text
Fixed Typo

Updated Variable

Added Validation

Fixed Lint Issues

Updated Documentation

Minor Bug Fix
```

---

Feature Development Might Create:

```text
50 Commits

100 Commits

200 Commits
```

---

Question:

```text
Do We Really Need

100 Small Commits

In Main Branch?
```

---

Answer:

```text
No
```

---

Need Cleaner History.

---

Git Provides:

```text
Squash

Interactive Rebase
```

---

# Problem Statement

Suppose You Developed:

```text
Payment Feature
```

---

Commit History

```text
A Fixed Typo

B Updated Variable

C Added Validation

D Fixed Lint Issues

E Added Tests
```

---

All Commits Belong To:

```text
One Feature
```

---

Instead Of

```text
5 Commits
```

---

Want

```text
1 Clean Commit
```

---

Solution

```text
Git Squash
```

---

# What Is Git Squash?

Git Squash combines:

```text
Multiple Commits

Into Single Commit
```

---

Example

Before

```text
A

B

C

D

E
```

---

After

```text
F
```

---

Where

```text
F
```

contains:

```text
All Changes
```

---

History Becomes Cleaner.

---

# Why Use Squash?

Benefits

```text
Clean History

Easy Code Review

Easy Rollback

Better Release Tracking
```

---

Very Common In:

```text
GitHub

GitLab

Enterprise Teams
```

---

# What Is Interactive Rebase?

Interactive Rebase allows developers to:

```text
Edit Commits

Delete Commits

Reorder Commits

Rename Commits

Squash Commits
```

---

Command

```bash
git rebase -i <commit-id>
```

---

Most Common Way To Perform Squash.

---

# Trainer Example

Suppose

```text
v1.0
```

and

```text
v1.1
```

exist.

---

Feature Difference

```text
v1.1 - v1.0
```

represents:

```text
New Feature
```

---

Instead Of Showing

```text
100 Commits
```

---

We Can Show

```text
1 Clean Commit
```

using Squash.

---

# How Interactive Rebase Works

Current History

```text
A

B

C

D

E
```

---

Run

```bash
git rebase -i A
```

---

Git Opens Editor

```text
pick B

pick C

pick D

pick E
```

---

Change To

```text
pick B

squash C

squash D

squash E
```

---

Save File.

---

Git Combines Commits.

---

Result

```text
A

F
```

---

# Interactive Rebase Options

## pick

Keep Commit.

---

## squash

Merge Commit Into Previous Commit.

---

## reword

Change Commit Message.

---

## edit

Modify Commit.

---

## drop

Delete Commit.

---

Frequently Asked Interview Topic.

---

# Example Using Real Commit IDs

Suppose

```text
84a22a0
```

is last stable commit.

---

History

```text
84a22a0

a111111

b222222

c333333
```

---

Run

```bash
git rebase -i 84a22a0
```

---

Git Shows

```text
pick a111111

pick b222222

pick c333333
```

---

Convert To

```text
pick a111111

squash b222222

squash c333333
```

---

Result

```text
84a22a0

d444444
```

---

Single Commit Created.

---

# Real Production Example

Developer Working On

```text
Feature Payment
```

---

Creates

```text
45 Commits
```

---

Before PR

```bash
git rebase -i
```

---

Squash Into

```text
1 Commit
```

---

PR Becomes Cleaner.

---

Code Review Easier.

---

# Another Production Example

Developer Creates

```text
Fix Typo

Fix Typo Again

Fix Typo Again
```

---

Instead Of

```text
3 Commits
```

---

Squash Into

```text
1 Commit
```

---

Repository History Cleaner.

---

# Squash During Pull Request

GitHub Supports

```text
Squash And Merge
```

---

Flow

```text
Feature Branch

↓

20 Commits

↓

Squash And Merge

↓

1 Commit In Main
```

---

Very Common In Organizations.

---

# Benefits Of Squash And Merge

```text
Clean History

Simple Rollback

Better Audit Trail

Easy Release Notes
```

---

# Drawbacks Of Squash

Original Commit History Lost.

---

Example

Before

```text
A

B

C

D
```

---

After

```text
F
```

---

Cannot See Individual Commits.

---

# Squash Vs Merge

## Merge

```text
Preserves All Commits
```

---

History

```text
A

B

C

D
```

---

## Squash

```text
Single Commit
```

---

History

```text
F
```

---

# Squash Vs Cherry Pick

## Cherry Pick

```text
Copies Specific Commit
```

---

## Squash

```text
Combines Multiple Commits
```

---

# Useful Commands

Interactive Rebase

```bash
git rebase -i HEAD~5
```

---

Interactive Rebase From Commit

```bash
git rebase -i <commit-id>
```

---

Squash Using GitHub

```text
Squash And Merge
```

Button In PR.

---

# Common Interview Questions

## What Is Git Squash?

### Short Answer

Git Squash combines multiple commits into a single commit.

### Detailed Explanation

Squashing helps keep commit history clean by replacing many small commits with one meaningful commit.

### Production Example

```text
100 Commits

↓

1 Commit
```

### Follow-Up Questions

```text
Why Squash Commits?

What Happens To Original Commits?

When Should Squash Be Used?
```

---

## What Is Interactive Rebase?

### Short Answer

Interactive Rebase is a Git feature used to edit, reorder, squash, rename, or remove commits.

### Detailed Explanation

Interactive Rebase provides complete control over commit history before code is merged.

### Production Example

```bash
git rebase -i HEAD~5
```

### Follow-Up Questions

```text
What Is pick?

What Is squash?

What Is reword?

What Is drop?
```

---

## Why Do Teams Squash Commits Before Merge?

### Short Answer

To maintain clean and meaningful commit history.

### Detailed Explanation

Large features often contain many small commits that are not useful in long-term repository history.

### Production Example

```text
Feature Branch

50 Commits

↓

1 Squashed Commit
```

### Follow-Up Questions

```text
When Should Squash Be Avoided?

Does Squash Rewrite History?

Can Squash Be Done In GitHub?
```

---

## What Is Squash And Merge?

### Short Answer

A GitHub feature that combines all feature branch commits into one commit during merge.

### Detailed Explanation

Squash And Merge keeps the main branch clean while preserving feature functionality.

### Production Example

```text
Feature Branch

↓

Squash And Merge

↓

Single Commit In Main
```

### Follow-Up Questions

```text
Difference Between Merge And Squash Merge?

When Is Squash Merge Useful?

What Are The Drawbacks?
```

---

# Production Scenarios

## Scenario 1

Feature Branch Contains 100 Commits.

### Solution

```bash
git rebase -i
```

Squash Commits.

---

## Scenario 2

PR History Too Noisy.

### Solution

```text
Squash Before Merge
```

---

## Scenario 3

Need To Rename Commit Messages.

### Solution

```bash
git rebase -i
```

Use

```text
reword
```

---

## Scenario 4

Need To Remove Unwanted Commit.

### Solution

```bash
git rebase -i
```

Use

```text
drop
```

---

## Scenario 5

Organization Uses GitHub.

### Solution

```text
Squash And Merge
```

during Pull Request.

---

# Key Takeaways

```text
Git Squash combines multiple commits into one commit.

Interactive Rebase is used to edit commit history.

git rebase -i is the most common way to squash commits.

Squash creates cleaner repository history.

GitHub provides Squash And Merge functionality.

Squashing is commonly performed before Pull Requests are merged.

Interactive Rebase can also reorder, rename, edit, and remove commits.

Squash improves code review and release management.
```