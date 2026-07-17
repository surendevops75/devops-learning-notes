# Git Reflog

## Introduction

Git Reset is powerful.

---

But sometimes developers accidentally delete commits.

Examples:

```text
Hard Reset Executed

Branch Deleted

Wrong Rebase

Wrong Checkout

Lost Commit
```

---

Question:

```text
Can We Recover Lost Commits?
```

---

Answer:

```text
Git Reflog
```

---

# Problem Statement

Suppose

```text
A → B → C → D
```

---

Developer Executes

```bash
git reset --hard HEAD~2
```

---

Now History Becomes

```text
A → B
```

---

Commits

```text
C

D
```

appear lost.

---

Question

```text
Are They Gone Forever?
```

---

Answer

```text
No
```

---

Git Reflog Can Recover Them.

---

# What Is Git Reflog?

Reflog Stands For:

```text
Reference Log
```

---

Git Maintains A Log Of:

```text
HEAD Movements

Branch Movements

Checkouts

Resets

Rebases

Commits
```

---

Think Of It As:

```text
Git Activity History
```

---

Even If A Commit Is Removed From Branch History:

```text
Reflog Remembers It
```

---

# How Reflog Works

Suppose

```text
A → B → C → D
```

---

Developer Executes

```bash
git reset --hard HEAD~2
```

---

History Now

```text
A → B
```

---

But Reflog Contains

```text
A

B

C

D
```

---

Git Knows:

```text
HEAD Previously Pointed To D
```

---

# Viewing Reflog

Command

```bash
git reflog
```

---

Example Output

```text
84a22a0 HEAD@{0}: reset: moving to HEAD~2

d123456 HEAD@{1}: commit: Added Payment API

c789123 HEAD@{2}: commit: Added Validation

b567890 HEAD@{3}: commit: Initial Feature
```

---

Reflog Shows

```text
Every HEAD Movement
```

---

# Understanding Reflog Entries

Example

```text
HEAD@{0}
```

means

```text
Current Position
```

---

Example

```text
HEAD@{1}
```

means

```text
Previous Position
```

---

Example

```text
HEAD@{2}
```

means

```text
Two Steps Earlier
```

---

# Recovering Lost Commit

Suppose

```text
d123456
```

was lost.

---

Recovery Command

```bash
git checkout d123456
```

---

Or

```bash
git reset --hard d123456
```

---

Commit Restored.

---

# Recovering After Hard Reset

Before

```text
A → B → C → D
```

---

Run

```bash
git reset --hard HEAD~2
```

---

After

```text
A → B
```

---

Recovery

```bash
git reflog
```

---

Find

```text
D Commit ID
```

---

Restore

```bash
git reset --hard <commit-id>
```

---

Everything Returns.

---

# Recovering Deleted Branch

Suppose Branch Deleted

```bash
git branch -D feature-payment
```

---

Branch Gone.

---

But Commits Exist In Reflog.

---

Find Commit

```bash
git reflog
```

---

Create Branch Again

```bash
git checkout -b feature-payment <commit-id>
```

---

Branch Recovered.

---

# Recovering After Wrong Rebase

Developer Executes

```bash
git rebase
```

---

History Looks Wrong.

---

Use

```bash
git reflog
```

---

Find Previous State.

---

Restore

```bash
git reset --hard HEAD@{5}
```

---

Repository Returns To Previous State.

---

# Recovering After Accidental Checkout

Developer Checks Out

```bash
git checkout old-branch
```

---

Loses Current Position.

---

Use

```bash
git reflog
```

---

Find Previous HEAD.

---

Return

```bash
git checkout <commit-id>
```

---

# Reflog Vs Log

## Git Log

Shows

```text
Current Branch History
```

---

Cannot Show

```text
Deleted Commits
```

---

## Git Reflog

Shows

```text
HEAD History
```

---

Can Show

```text
Deleted Commits

Lost Commits

Reset History
```

---

# Reflog Expiration

Reflog Entries Do Not Live Forever.

---

Default

```text
90 Days
```

for reachable commits.

---

```text
30 Days
```

for unreachable commits.

---

Eventually Git Cleanup Removes Them.

---

# Real Production Example

Developer Accidentally Executes

```bash
git reset --hard HEAD~10
```

---

10 Commits Disappear.

---

Recovery

```bash
git reflog
```

---

Find Previous HEAD.

---

Restore Commits.

---

Saved Hours Of Work.

---

# Another Production Example

Developer Deletes Feature Branch

```bash
git branch -D feature-auth
```

---

Branch Contained:

```text
2 Weeks Of Work
```

---

Recovery

```bash
git reflog
```

---

Find Commit.

---

Recreate Branch.

---

No Work Lost.

---

# Useful Commands

View Reflog

```bash
git reflog
```

---

Checkout Lost Commit

```bash
git checkout <commit-id>
```

---

Restore Commit

```bash
git reset --hard <commit-id>
```

---

Recover Previous State

```bash
git reset --hard HEAD@{1}
```

---

Create Branch From Lost Commit

```bash
git checkout -b feature-x <commit-id>
```

---

# Common Interview Questions

## What Is Git Reflog?

### Short Answer

Git Reflog records all HEAD and branch reference movements in a local repository.

### Detailed Explanation

Unlike Git Log, Reflog tracks actions such as commits, checkouts, resets, rebases, and branch changes, allowing recovery of lost commits.

### Production Example

```bash
git reset --hard HEAD~3
```

↓

Commits Lost

↓

Recover Using

```bash
git reflog
```

### Follow-Up Questions

```text
How Is Reflog Different From Log?

Can Reflog Recover Deleted Commits?

Does Reflog Exist On Remote Repositories?
```

---

## Difference Between Git Log And Git Reflog?

### Short Answer

Git Log shows commit history, while Git Reflog shows HEAD and reference history.

### Detailed Explanation

Git Log only displays commits reachable from the current branch. Reflog records movements even when commits are no longer visible.

### Production Example

```text
Deleted Commit

↓

Not In Log

↓

Still In Reflog
```

### Follow-Up Questions

```text
Which Is Used For Recovery?

Can Reflog Show Reset Operations?

Can Reflog Show Deleted Branches?
```

---

## Can Reflog Recover Deleted Commits?

### Short Answer

Yes.

### Detailed Explanation

As long as the commit still exists in the reflog, Git can restore it using the commit ID.

### Production Example

```bash
git reset --hard HEAD~5
```

↓

Recover Via

```bash
git reflog
```

### Follow-Up Questions

```text
How Long Does Reflog Keep Entries?

What Happens After Garbage Collection?

Can Recovery Fail?
```

---

## Can Reflog Recover Deleted Branches?

### Short Answer

Yes.

### Detailed Explanation

Even after branch deletion, the commits remain in reflog until garbage collection removes them.

### Production Example

```bash
git branch -D feature-payment
```

↓

Recover Branch Using Commit ID.

### Follow-Up Questions

```text
How Do You Recreate The Branch?

Can Reflog Recover Remote Branches?

Does Branch Deletion Remove Commits Immediately?
```

---

## Is Reflog Stored On Remote Repositories?

### Short Answer

No.

### Detailed Explanation

Reflog is a local repository feature and exists only on the developer's machine.

### Production Example

```text
Developer Laptop

↓

Has Reflog
```

```text
GitHub

↓

No Reflog Access
```

### Follow-Up Questions

```text
Can GitHub Show Reflog?

Why Is Reflog Local?

How Do Teams Recover Shared History?
```

---

# Production Scenarios

## Scenario 1

Accidentally Executed Hard Reset.

### Solution

```bash
git reflog
```

Find Commit.

Restore It.

---

## Scenario 2

Deleted Feature Branch.

### Solution

```bash
git reflog
```

Create Branch Again.

---

## Scenario 3

Wrong Rebase Executed.

### Solution

```bash
git reflog
```

Restore Previous HEAD.

---

## Scenario 4

Need To Find Previous HEAD Position.

### Solution

```bash
git reflog
```

---

## Scenario 5

Commit Missing From Git Log.

### Solution

```text
Check Reflog
```

---

# Key Takeaways

```text
Git Reflog is Git's recovery mechanism.

Reflog records HEAD and branch movements.

Reflog can recover commits lost due to reset.

Reflog can recover deleted branches.

Reflog can recover from bad rebases.

Git Log shows commit history.

Git Reflog shows reference history.

Reflog is stored locally.

Reflog is one of the most important Git recovery tools.

Every DevOps Engineer should know Git Reflog.
```