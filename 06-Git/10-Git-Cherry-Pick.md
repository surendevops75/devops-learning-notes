# Git Cherry Pick

## Introduction

In software development, there are situations where we need only a specific change from another branch.

---

Example

Developer A fixed:

```text
Login Bug
```

in:

```text
feature/authentication
```

---

Later you realize:

```text
The Same Fix

Is Needed In Another Branch
```

---

Question:

```text
Do We Merge The Entire Branch?
```

---

Answer:

```text
No
```

---

We can copy only the required commit.

---

Git provides:

```text
Git Cherry Pick
```

---

# Problem Statement

Suppose:

```text
feature-payment
```

contains:

```text
Commit A

Commit B

Commit C
```

---

Need only:

```text
Commit B
```

---

Merging the entire branch would bring:

```text
A

B

C
```

---

Need only:

```text
B
```

---

Solution:

```text
Git Cherry Pick
```

---

# What Is Git Cherry Pick?

Git Cherry Pick is used to:

```text
Copy A Specific Commit

From One Branch

To Another Branch
```

---

Think Of It As:

```text
Copy-Paste For Commits
```

---

# How Cherry Pick Works

Current Branch

```text
main
```

---

Another Branch

```text
feature-payment
```

contains:

```text
A → B → C
```

---

Need Only:

```text
Commit B
```

---

Run:

```bash
git cherry-pick <commit-id-of-B>
```

---

Result

```text
main

↓

Commit B Copied
```

---

# Visual Example

Before

```text
feature-payment

A → B → C
```

---

```text
main

X → Y
```

---

Cherry Pick

```text
Commit B
```

---

After

```text
main

X → Y → B'
```

---

Notice

```text
B'
```

---

Git Creates:

```text
New Commit
```

---

Commit Content Same.

---

Commit ID Different.

---

# Why Cherry Pick Creates New Commit?

Because:

```text
Commit History Changes
```

---

Git Generates:

```text
New Commit Hash
```

---

Original Commit Remains Unchanged.

---

# Basic Cherry Pick Command

```bash
git cherry-pick <commit-id>
```

---

Example

```bash
git cherry-pick a84bc52
```

---

Commit Copied.

---

New Commit Created.

---

# How To Find Commit ID?

Command

```bash
git log --oneline
```

---

Example

```text
a84bc52 Fixed Login Bug

c92db71 Added Payment Feature

e11ab20 Updated Dockerfile
```

---

Copy Required Commit ID.

---

Cherry Pick It.

---

# Cherry Pick Multiple Commits

Command

```bash
git cherry-pick commit1 commit2 commit3
```

---

Example

```bash
git cherry-pick a84bc52 c92db71
```

---

Multiple Commits Copied.

---

# Cherry Pick Range Of Commits

Example

```bash
git cherry-pick A^..D
```

---

Copies

```text
A

B

C

D
```

---

Useful During Releases.

---

# Real Production Example

Developer Fixes:

```text
Critical Security Bug
```

---

Branch

```text
feature-auth
```

---

Need Same Fix In

```text
release-v1.0
```

---

Instead Of Merging Entire Branch:

```bash
git cherry-pick <commit-id>
```

---

Only Security Fix Copied.

---

# Another Production Example

Hotfix Created For:

```text
Production Issue
```

---

Fix Exists In:

```text
main
```

---

Need Same Fix In:

```text
release branch
```

---

Use:

```bash
git cherry-pick <commit-id>
```

---

No Need To Merge Entire Branch.

---

# Cherry Pick Conflicts

Cherry Pick Can Create:

```text
Conflicts
```

---

Reason

```text
Target Branch

Already Contains Different Changes
```

---

Example

Developer A

```python
PORT=8080
```

---

Developer B

```python
PORT=9090
```

---

Cherry Pick Occurs.

---

Git Cannot Decide.

---

Conflict Generated.

---

# Resolving Cherry Pick Conflicts

Fix File.

---

Stage Changes.

```bash
git add .
```

---

Continue

```bash
git cherry-pick --continue
```

---

# Abort Cherry Pick

If Something Goes Wrong

```bash
git cherry-pick --abort
```

---

Returns To Previous State.

---

# Cherry Pick Vs Merge

## Merge

```text
Entire Branch
```

---

Brings:

```text
All Commits
```

---

## Cherry Pick

```text
Specific Commit
```

---

Brings:

```text
Selected Changes Only
```

---

# Cherry Pick Vs Rebase

## Rebase

```text
Moves Multiple Commits
```

---

## Cherry Pick

```text
Copies Specific Commit
```

---

# When To Use Cherry Pick?

Useful For

```text
Bug Fixes

Hotfixes

Release Branches

Security Fixes

Selective Commit Migration
```

---

# When Not To Use Cherry Pick?

Avoid When:

```text
Entire Branch Needed
```

---

Use:

```text
Merge

or

Rebase
```

instead.

---

# Useful Commands

Cherry Pick Commit

```bash
git cherry-pick <commit-id>
```

---

Cherry Pick Multiple Commits

```bash
git cherry-pick commit1 commit2
```

---

Continue Cherry Pick

```bash
git cherry-pick --continue
```

---

Abort Cherry Pick

```bash
git cherry-pick --abort
```

---

View Commit History

```bash
git log --oneline
```

---

# Common Interview Questions

## What Is Git Cherry Pick?

### Short Answer

Git Cherry Pick copies a specific commit from one branch to another.

### Detailed Explanation

Cherry Pick allows developers to selectively apply commits without merging the entire branch.

### Production Example

```text
Security Fix

↓

Cherry Pick

↓

Release Branch
```

### Follow-Up Questions

```text
Does Cherry Pick Create New Commit IDs?

Can Cherry Pick Cause Conflicts?

How Is Cherry Pick Different From Merge?
```

---

## Why Do We Use Cherry Pick?

### Short Answer

To copy only required commits instead of merging an entire branch.

### Detailed Explanation

Cherry Pick is useful when only a subset of changes is needed.

### Production Example

```text
Feature Branch

↓

Contains 20 Commits

↓

Need 1 Bug Fix

↓

Cherry Pick
```

### Follow-Up Questions

```text
Why Not Merge?

Why Not Rebase?

When Is Cherry Pick Most Useful?
```

---

## Can Cherry Pick Cause Conflicts?

### Short Answer

Yes.

### Detailed Explanation

If target branch contains conflicting changes, Git cannot automatically apply the commit.

### Production Example

```text
Different Configuration

↓

Cherry Pick

↓

Conflict
```

### Follow-Up Questions

```text
How Do You Resolve Cherry Pick Conflicts?

How Do You Continue Cherry Pick?

How Do You Abort Cherry Pick?
```

---

## Difference Between Merge And Cherry Pick?

### Short Answer

Merge brings all commits from a branch, while Cherry Pick copies only selected commits.

### Detailed Explanation

Merge integrates branch history. Cherry Pick selectively transfers changes.

### Production Example

```text
Merge

↓

Entire Branch
```

```text
Cherry Pick

↓

Specific Commit
```

### Follow-Up Questions

```text
Which Creates New Commit?

Which Preserves History?

When Should Each Be Used?
```

---

# Production Scenarios

## Scenario 1

Need Security Fix In Release Branch.

### Solution

```bash
git cherry-pick <commit-id>
```

---

## Scenario 2

Hotfix Applied To Main.

Need Same Fix In Older Release.

### Solution

```bash
git cherry-pick <commit-id>
```

---

## Scenario 3

Feature Branch Contains 50 Commits.

Need Only One Bug Fix.

### Solution

```text
Cherry Pick Specific Commit
```

---

## Scenario 4

Cherry Pick Creates Conflict.

### Solution

```bash
git add .
git cherry-pick --continue
```

---

## Scenario 5

Wrong Commit Cherry Picked.

### Solution

```bash
git cherry-pick --abort
```

---

# Key Takeaways

```text
Git Cherry Pick copies specific commits from one branch to another.

Cherry Pick creates a new commit with a new commit ID.

Cherry Pick is useful for bug fixes, hotfixes, and release branches.

Cherry Pick allows selective commit migration.

Cherry Pick can create conflicts.

git cherry-pick --continue resumes after conflict resolution.

git cherry-pick --abort cancels the operation.

Cherry Pick is commonly used in production support and release management.
```