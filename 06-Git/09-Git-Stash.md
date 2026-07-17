# Git Stash

## Introduction

While working on a feature, developers sometimes need to switch tasks immediately.

---

Example

You are working on:

```text
Payment Feature
```

---

Suddenly:

```text
Production Issue Reported
```

---

Need To:

```text
Switch Branch

Create Hotfix

Fix Production Issue
```

---

But Current Changes Are:

```text
Incomplete

Not Ready For Commit
```

---

Question:

```text
How Do We Temporarily Save Work?
```

---

Answer:

```text
Git Stash
```

---

# Problem Statement

Suppose You Are Working On:

```text
feature/payment
```

---

Modified Files

```text
payment.py

config.yaml

Dockerfile
```

---

Changes Not Yet Committed.

---

Need To Move To:

```text
hotfix/payment-failure
```

---

Git Normally Prevents Branch Switches When Changes Exist.

---

Need Temporary Storage.

---

# What Is Git Stash?

Git Stash is used to:

```text
Temporarily Save Changes

Without Creating Commit
```

---

Think Of It As:

```text
Temporary Clipboard
```

for your code.

---

Git Stores Changes In:

```text
Stash Stack
```

---

You Can Retrieve Them Later.

---

# How Stash Works

Before

```text
Working Directory

↓

Modified Files
```

---

Run

```bash
git stash
```

---

After

```text
Working Directory Clean
```

---

Changes Stored In:

```text
Stash
```

---

# Basic Stash Command

```bash
git stash
```

---

Git Saves:

```text
Modified Files

Staged Changes
```

---

Working Directory Becomes Clean.

---

# Real Workflow

Current Branch

```text
feature/payment
```

---

Modified Files

```text
payment.py

config.yaml
```

---

Run

```bash
git stash
```

---

Switch Branch

```bash
git checkout hotfix/payment-bug
```

---

Fix Issue.

---

Return To Feature Branch.

---

Restore Changes.

---

# Viewing Stash List

Command

```bash
git stash list
```

---

Example Output

```text
stash@{0}

stash@{1}

stash@{2}
```

---

Latest Stash

```text
stash@{0}
```

---

Older Stashes

```text
stash@{1}

stash@{2}
```

---

# Restoring Stashed Changes

Command

```bash
git stash apply
```

---

Restores:

```text
Latest Stash
```

---

Important:

```text
Stash Still Exists
```

---

# Stash Pop

Command

```bash
git stash pop
```

---

Restores Changes.

---

Removes Stash Entry.

---

Difference

## Apply

```text
Restore

Keep Stash
```

---

## Pop

```text
Restore

Delete Stash
```

---

# Apply Specific Stash

Command

```bash
git stash apply stash@{1}
```

---

Useful When Multiple Stashes Exist.

---

# Drop Stash

Delete Single Stash

```bash
git stash drop stash@{0}
```

---

Removes Specific Stash.

---

# Clear All Stashes

Command

```bash
git stash clear
```

---

Deletes:

```text
All Stashes
```

---

Use Carefully.

---

# Stash With Message

Default Stash Names Are Not Helpful.

---

Better

```bash
git stash push -m "payment feature changes"
```

---

View

```bash
git stash list
```

---

Output

```text
payment feature changes
```

---

Much Easier To Identify.

---

# Stashing Untracked Files

By Default

```text
Git Does Not Stash Untracked Files
```

---

Use

```bash
git stash -u
```

---

Now Git Stashes:

```text
Tracked Files

Untracked Files
```

---

# Stashing Everything

Including Ignored Files

```bash
git stash -a
```

---

Rarely Used.

---

# Real Production Example

Developer Working On:

```text
EKS Upgrade
```

---

Files Modified

```text
terraform.tfvars

eks.tf
```

---

Production Incident Occurs.

---

Need Immediate Hotfix.

---

Run

```bash
git stash
```

---

Switch Branch.

---

Fix Incident.

---

Return Later.

---

Run

```bash
git stash pop
```

---

Continue Work.

---

# Another Production Example

Developer Has:

```text
Half Completed Feature
```

---

Manager Requests:

```text
Urgent Bug Fix
```

---

Instead Of:

```text
Creating Temporary Commit
```

---

Use

```bash
git stash
```

---

Cleaner Workflow.

---

# Stash Internals

Git Creates:

```text
Temporary Commit
```

---

Stores It In:

```text
.refs/stash
```

---

Not Part Of Normal Branch History.

---

# Useful Commands

Create Stash

```bash
git stash
```

---

View Stashes

```bash
git stash list
```

---

Apply Stash

```bash
git stash apply
```

---

Pop Stash

```bash
git stash pop
```

---

Drop Stash

```bash
git stash drop stash@{0}
```

---

Clear All Stashes

```bash
git stash clear
```

---

Stash With Message

```bash
git stash push -m "message"
```

---

# Common Interview Questions

## What Is Git Stash?

### Short Answer

Git Stash temporarily saves uncommitted changes without creating a commit.

### Detailed Explanation

Git Stash allows developers to switch tasks without losing unfinished work by storing changes in a temporary stash area.

### Production Example

```text
Feature Work

↓

Production Issue

↓

Stash

↓

Hotfix

↓

Pop
```

### Follow-Up Questions

```text
Where Are Stashes Stored?

Can Stashes Be Shared?

Does Stash Create Commits?
```

---

## Difference Between Stash Apply And Stash Pop?

### Short Answer

Apply restores changes and keeps the stash. Pop restores changes and removes the stash.

### Detailed Explanation

Apply is useful when you may need the stash again. Pop is useful when the stash is no longer needed.

### Production Example

```text
Apply

↓

Restore

↓

Keep Copy
```

```text
Pop

↓

Restore

↓

Delete Copy
```

### Follow-Up Questions

```text
Which Is Safer?

Can Pop Cause Conflicts?

Can Deleted Stashes Be Recovered?
```

---

## Why Use Stash Instead Of Commit?

### Short Answer

Because work is incomplete and not ready to become part of commit history.

### Detailed Explanation

Temporary commits pollute repository history. Stash provides a cleaner way to pause work.

### Production Example

```text
Half Finished Feature

↓

Stash

↓

Hotfix

↓

Resume
```

### Follow-Up Questions

```text
When Should You Commit Instead?

When Should You Stash?

Can Stashes Be Pushed?
```

---

## Can Git Stash Cause Conflicts?

### Short Answer

Yes.

### Detailed Explanation

If files changed after the stash was created, applying or popping the stash may create conflicts.

### Production Example

```text
Stash Created

↓

Main Branch Changed

↓

Stash Pop

↓

Conflict
```

### Follow-Up Questions

```text
How Are Stash Conflicts Resolved?

Can Rebase Cause Similar Problems?

How Do Teams Avoid This?
```

---

# Production Scenarios

## Scenario 1

Production Issue Arrives During Feature Development.

### Solution

```bash
git stash
```

Switch Branch.

Fix Issue.

```bash
git stash pop
```

---

## Scenario 2

Need To Pause Feature Development.

### Solution

```bash
git stash
```

---

## Scenario 3

Multiple Incomplete Tasks.

### Solution

```bash
git stash push -m "message"
```

---

## Scenario 4

Need To See Available Stashes.

### Solution

```bash
git stash list
```

---

## Scenario 5

Need To Restore Specific Stash.

### Solution

```bash
git stash apply stash@{1}
```

---

# Key Takeaways

```text
Git Stash temporarily stores uncommitted changes.

Stash is useful when switching tasks.

git stash saves changes.

git stash apply restores changes and keeps stash.

git stash pop restores changes and removes stash.

Stashes are stored locally.

Stash can also save untracked files.

Stash is commonly used during production hotfix situations.

Git Stash is a very common DevOps and developer productivity tool.
```