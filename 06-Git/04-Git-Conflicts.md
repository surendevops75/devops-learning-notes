# Git Conflicts

## Introduction

Git allows multiple developers to work on the same codebase simultaneously.

---

Example

```text
Developer A

↓

Updates payment.py
```

---

```text
Developer B

↓

Updates payment.py
```

---

Both developers modify:

```text
Same File

or

Same Line
```

---

When Git cannot automatically determine which change to keep:

```text
Conflict Occurs
```

---

# What Is A Git Conflict?

A Git conflict occurs when:

```text
Git Cannot Automatically Merge Changes
```

from multiple branches.

---

Git requires:

```text
Human Intervention
```

to decide which change should be kept.

---

# Why Do Conflicts Happen?

Most common reasons:

```text
Same Line Modified

Same File Modified

File Deleted In One Branch

File Renamed In Another Branch

Long-Lived Feature Branches
```

---

# Conflict Example

Main Branch

```python
PORT=8080
```

---

Feature Branch

```python
PORT=9090
```

---

Developer Runs

```bash
git merge main
```

---

Git Cannot Decide

```text
Which Value Is Correct?
```

---

Conflict Generated.

---

# What Happens Internally?

Before Merge

```text
main

A --- B --- C
```

```text
feature

A --- B --- D
```

---

Git Tries To Combine

```text
C

and

D
```

---

If Changes Overlap

```text
Conflict
```

---

# Conflict Markers

Git Modifies File

```text
<<<<<<< HEAD

PORT=9090

=======

PORT=8080

>>>>>>> main
```

---

Meaning

```text
HEAD

↓

Current Branch
```

---

```text
main

↓

Incoming Changes
```

---

Developer Must Decide.

---

# Resolving Conflict

Step 1

Open File.

---

Step 2

Choose Correct Version.

---

Example

```python
PORT=8080
```

---

Remove Markers

```text
<<<<<<<

=======

>>>>>>>
```

---

Step 3

Stage Changes

```bash
git add .
```

---

Step 4

Commit

```bash
git commit -m "Resolved merge conflict"
```

---

Conflict Resolved.

---

# Merge Conflict Example

Developer A

```python
DB_HOST=dev-db
```

---

Developer B

```python
DB_HOST=prod-db
```

---

Merge Attempt

```bash
git merge main
```

---

Git Cannot Decide.

---

Developer Resolves Conflict.

---

Commit Created.

---

# Rebase Conflicts

Conflicts can occur during:

```bash
git rebase main
```

---

Reason

```text
Rebase Replays Commits
```

---

If Replay Causes Overlap

```text
Conflict Occurs
```

---

# Rebase Conflict Workflow

Start Rebase

```bash
git rebase main
```

---

Conflict Appears.

---

Fix File.

---

Stage File

```bash
git add .
```

---

Continue Rebase

```bash
git rebase --continue
```

---

# Abort Rebase

If Rebase Goes Wrong

```bash
git rebase --abort
```

---

Repository Returns To Previous State.

---

# Common Conflict Scenarios

## Scenario 1

Same Line Modified.

---

Developer A

```text
PORT=8080
```

---

Developer B

```text
PORT=9090
```

---

Conflict.

---

## Scenario 2

File Deleted In One Branch

Developer A

```text
Deletes config.yaml
```

---

Developer B

```text
Modifies config.yaml
```

---

Conflict.

---

## Scenario 3

File Renamed

Developer A

```text
Rename app.py
```

---

Developer B

```text
Modify app.py
```

---

Conflict.

---

## Scenario 4

Long-Lived Feature Branch

Branch Exists For:

```text
2 Months
```

---

Main Branch Changed Hundreds Of Times.

---

Huge Conflict Risk.

---

# How To Reduce Conflicts?

## Pull Frequently

```bash
git pull
```

---

Stay Updated.

---

## Rebase Frequently

```bash
git rebase main
```

---

Keep Branch Current.

---

## Small Pull Requests

Smaller Changes

↓

Fewer Conflicts.

---

## Short-Lived Branches

Avoid Long Running Branches.

---

## Team Communication

Avoid Multiple Developers Editing Same Files Simultaneously.

---

# Real Production Example

Team Working On:

```text
EKS Upgrade Automation
```

---

Developer A

Updates

```text
terraform/main.tf
```

---

Developer B

Updates

```text
terraform/main.tf
```

---

PR Created.

---

Git Reports Conflict.

---

Developer Pulls Latest Changes.

---

Resolves Conflict.

---

Pushes Updated Branch.

---

PR Merged Successfully.

---

# GitHub Pull Request Conflicts

Sometimes GitHub Shows:

```text
This Branch Has Conflicts
```

---

Meaning

```text
PR Cannot Be Merged Automatically
```

---

Need To

```bash
git pull

git merge main
```

---

Resolve Conflicts.

---

Push Again.

---

# Conflict Resolution Workflow

```text
Conflict Detected

↓

Open File

↓

Review Changes

↓

Select Correct Code

↓

Remove Markers

↓

git add

↓

git commit

↓

Push
```

---

# Useful Commands

Check Status

```bash
git status
```

---

Merge Branch

```bash
git merge main
```

---

Rebase Branch

```bash
git rebase main
```

---

Continue Rebase

```bash
git rebase --continue
```

---

Abort Rebase

```bash
git rebase --abort
```

---

View Differences

```bash
git diff
```

---

# Common Interview Questions

## What Is A Git Conflict?

### Short Answer

A Git conflict occurs when Git cannot automatically merge changes from multiple branches.

### Detailed Explanation

Conflicts occur when multiple developers modify the same part of a file and Git cannot determine which version should be kept.

### Production Example

```text
Developer A

PORT=8080
```

```text
Developer B

PORT=9090
```

↓

Conflict

### Follow-Up Questions

- How do conflicts occur?
- Can conflicts happen during rebase?
- How are conflicts resolved?

---

## How Do You Resolve A Merge Conflict?

### Short Answer

Open the conflicting file, choose the correct changes, remove conflict markers, stage the file, and commit.

### Commands

```bash
git add .
git commit
```

### Production Example

```text
Conflict

↓

Manual Fix

↓

Commit
```

### Follow-Up Questions

- What are conflict markers?
- What happens if conflict isn't resolved?
- Can IDEs help resolve conflicts?

---

## Can Rebase Cause Conflicts?

### Short Answer

Yes.

### Detailed Explanation

Rebase replays commits onto a new base. If replayed commits overlap with existing changes, conflicts occur.

### Production Example

```bash
git rebase main
```

↓

Conflict

↓

Fix

↓

git rebase --continue


---

### Follow-Up Questions

- How do you continue a rebase?
- How do you abort a rebase?
- Why do conflicts occur during rebase?

---

## How Do You Avoid Git Conflicts?

### Short Answer

Pull frequently, create small PRs, use short-lived branches, and communicate with team members.

### Detailed Explanation

Keeping branches synchronized reduces the likelihood of overlapping changes.

### Production Example

```text
Daily Pull

↓

Smaller Differences

↓

Fewer Conflicts
```

### Follow-Up Questions

- Why are long-lived branches risky?
- How does rebase help?
- What branching strategies reduce conflicts?

---

# Production Scenarios

## Scenario 1

PR Shows:

```text
This Branch Has Conflicts
```

### Solution

```bash
git checkout feature-branch

git merge main
```

Resolve Conflict.

Commit.

Push.

---

## Scenario 2

Rebase Stopped Midway.

### Solution

```bash
git status
```

Fix Conflict.

```bash
git add .
```

```bash
git rebase --continue
```

---

## Scenario 3

Wrong Conflict Resolution Applied.

### Solution

```bash
git reset
```

or

```bash
git reflog
```

Recover Changes.

---

## Scenario 4

Feature Branch Is 3 Months Old.

### Risk

```text
Massive Merge Conflicts
```

### Recommendation

```text
Rebase Frequently
```

---

## Scenario 5

Multiple Developers Editing Same Terraform File.

### Risk

```text
Frequent Conflicts
```

### Recommendation

```text
Smaller Changes

Better Coordination
```

---

# Key Takeaways

```text
Git conflicts occur when Git cannot automatically merge changes.

Conflicts commonly occur when the same file or line is modified.

Conflicts can occur during merge and rebase.

Git inserts conflict markers into affected files.

Developers must manually resolve conflicts.

git rebase --continue continues a rebase after conflict resolution.

git rebase --abort cancels a rebase operation.

Frequent pulls and short-lived branches reduce conflict frequency.

Conflict resolution is one of the most common Git interview topics.
```