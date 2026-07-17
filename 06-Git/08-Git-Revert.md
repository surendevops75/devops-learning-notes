# Git Revert

## Introduction

Developers sometimes make mistakes after code has already been pushed.

Examples:

```text
Wrong Configuration

Buggy Code

Incorrect Terraform Changes

Security Issue

Application Failure
```

---

Question:

```text
How Do We Undo Changes

Without Rewriting History?
```

---

Answer:

```text
Git Revert
```

---

# Problem Statement

Suppose:

```text
A → B → C
```

---

Commit C contains:

```text
Production Bug
```

---

Need To:

```text
Undo Commit C
```

---

But:

```text
Commit Already Pushed

Other Developers Already Pulled It
```

---

Using:

```bash
git reset
```

is dangerous.

---

Need Safer Solution.

---

# What Is Git Revert?

Git Revert is used to:

```text
Undo Changes

Without Deleting History
```

---

Unlike Reset:

```text
Revert Does Not Remove Commits
```

---

Instead:

```text
Creates New Commit
```

that reverses previous changes.

---

# How Revert Works

Before

```text
A → B → C
```

---

Run

```bash
git revert C
```

---

After

```text
A → B → C → D
```

---

Where

```text
D
```

contains:

```text
Opposite Changes Of C
```

---

# Why Revert Is Safe?

Because:

```text
History Remains Intact
```

---

No Commit IDs Changed.

---

No History Rewritten.

---

Safe For:

```text
Shared Branches

Remote Branches

Main Branch
```

---

# Revert Workflow

Suppose Commit

```text
C
```

added:

```python
PORT=9090
```

---

Revert Creates Commit

```text
D
```

that changes:

```python
PORT=8080
```

---

Result

```text
Original Commit Exists

Revert Commit Exists
```

---

Complete History Preserved.

---

# Basic Revert Command

Revert Latest Commit

```bash
git revert HEAD
```

---

Git Creates:

```text
New Revert Commit
```

---

# Revert Specific Commit

```bash
git revert <commit-id>
```

---

Example

```bash
git revert a84bc52
```

---

Git Creates New Commit.

---

Original Commit Still Exists.

---

# Reverting Multiple Commits

Example

```text
A → B → C → D
```

---

Need To Revert

```text
C

and

D
```

---

Run

```bash
git revert C D
```

---

Git Creates:

```text
Separate Revert Commits
```

---

# Revert Without Auto Commit

Command

```bash
git revert --no-commit <commit-id>
```

---

Changes Applied.

---

Commit Not Created.

---

Useful When:

```text
Combining Multiple Reverts
```

---

# Real Production Example

Developer Pushes:

```text
Wrong Database Endpoint
```

---

Application Fails.

---

Commit Already In:

```text
Main Branch
```

---

Solution

```bash
git revert <commit-id>
```

---

Push Revert Commit.

---

Application Restored.

---

# Another Production Example

Developer Merges PR.

---

Later Finds:

```text
Critical Bug
```

---

Need Immediate Rollback.

---

Solution

```bash
git revert <merge-commit>
```

---

Deploy Reverted Version.

---

# Reverting Merge Commits

Merge Commits Have:

```text
Two Parents
```

---

Need Parent Selection.

---

Command

```bash
git revert -m 1 <merge-commit>
```

---

Very Common Interview Question.

---

# Reset vs Revert

## Reset

```text
Deletes Commits

Rewrites History

Private Branches
```

---

## Revert

```text
Keeps Commits

Creates Revert Commit

Shared Branches
```

---

# When To Use Reset?

```text
Local Commits

Private Branches

Personal Development
```

---

# When To Use Revert?

```text
Production Branches

Main Branch

Shared Branches

Already Pushed Commits
```

---

# Golden Rule

If Commit Is Already Shared:

```text
Use Revert
```

---

If Commit Is Local:

```text
Use Reset
```

---

# Useful Commands

Revert Latest Commit

```bash
git revert HEAD
```

---

Revert Specific Commit

```bash
git revert <commit-id>
```

---

Revert Without Commit

```bash
git revert --no-commit <commit-id>
```

---

Revert Merge Commit

```bash
git revert -m 1 <merge-commit>
```

---

# Common Interview Questions

## What Is Git Revert?

### Short Answer

Git Revert is used to undo changes by creating a new commit that reverses a previous commit.

### Detailed Explanation

Git Revert does not remove commits or rewrite history. Instead, it creates a new commit containing opposite changes.

### Production Example

```bash
git revert HEAD
```

Used to rollback a faulty commit in production.

### Follow-Up Questions

```text
Does Revert Rewrite History?

Is Revert Safe On Main Branch?

How Is Revert Different From Reset?
```

---

## Difference Between Reset And Revert?

### Short Answer

Reset removes commits and rewrites history, while Revert preserves history and creates a new commit.

### Detailed Explanation

Reset changes commit history. Revert adds a new commit that undoes previous changes.

### Production Example

```text
Reset

↓

Delete Commit
```

```text
Revert

↓

Create Undo Commit
```

### Follow-Up Questions

```text
Which Is Safer?

Which Should Be Used In Production?

Which Is Suitable For Shared Branches?
```

---

## Why Is Revert Safer Than Reset?

### Short Answer

Because Revert preserves commit history.

### Detailed Explanation

Other developers can continue working without synchronization problems since no commit history is changed.

### Production Example

```text
Bad Commit

↓

Revert Commit

↓

History Preserved
```

### Follow-Up Questions

```text
What Happens After Revert?

Can Revert Be Reverted?

How Do Teams Rollback Production Changes?
```

---

## Can Merge Commits Be Reverted?

### Short Answer

Yes.

### Detailed Explanation

Git requires specifying which parent branch should be treated as the mainline.

### Production Example

```bash
git revert -m 1 <merge-commit>
```

### Follow-Up Questions

```text
What Is Parent 1?

Why Does Git Need -m?

What Happens If Wrong Parent Is Selected?
```

---

# Production Scenarios

## Scenario 1

Buggy Commit Already Pushed.

### Solution

```bash
git revert <commit-id>
```

---

## Scenario 2

Production Deployment Failed.

### Solution

```text
Revert Faulty Commit
```

Deploy Again.

---

## Scenario 3

Wrong Configuration Merged Into Main.

### Solution

```bash
git revert <commit-id>
```

---

## Scenario 4

Need To Undo Merge Commit.

### Solution

```bash
git revert -m 1 <merge-commit>
```

---

## Scenario 5

Multiple Bad Commits In Production.

### Solution

```bash
git revert <commit-id>
```

or

```bash
git revert --no-commit
```

Combine Reverts.

---

# Key Takeaways

```text
Git Revert undoes changes without deleting commit history.

Revert creates a new commit.

Revert is safe for shared and production branches.

Reset rewrites history, Revert preserves history.

Use Revert for already pushed commits.

Use Revert for production rollbacks.

Merge commits can also be reverted.

Revert is one of the safest Git recovery mechanisms.
```