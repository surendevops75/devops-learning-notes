# Git Merge vs Rebase

## Introduction

One of the most frequently asked Git interview questions is:

```text
Merge vs Rebase
```

---

Almost every developer uses:

```bash
git merge
```

or

```bash
git rebase
```

---

But many developers do not fully understand:

```text
What Happens Internally?
```

---

Understanding Merge and Rebase is important because:

```text
Code Integration

Pull Requests

Branch Synchronization

Conflict Resolution
```

depend on them.

---

# Why Do We Need Merge Or Rebase?

Suppose:

```text
main
```

contains:

```text
Commit A

Commit B
```

---

You create:

```text
payment-feature
```

branch.

---

While you are developing:

```text
Developer B

Adds Commit C

To Main
```

---

Now Branches Look Like:

```text
main

A --- B --- C
```

```text
payment-feature

A --- B --- D --- E
```

---

Need To Combine Changes.

---

Options:

```text
Merge

or

Rebase
```

---

# What Is Git Merge?

Merge combines changes from one branch into another.

---

Command

```bash
git merge main
```

---

Example

Current Branch

```text
payment-feature
```

---

Run

```bash
git merge main
```

---

Git Creates:

```text
Merge Commit
```

---

# What Is A Merge Commit?

A Merge Commit is a special commit that has:

```text
Two Parents
```

---

Normal Commit

```text
One Parent
```

---

Merge Commit

```text
Two Parents
```

---

# Merge Example

Before Merge

```text
main

A --- B --- C
```

```text
payment-feature

A --- B --- D --- E
```

---

After Merge

```text
A --- B --- C
       \     \
        D --- E --- M
```

---

M = Merge Commit

---

# Why Merge Commit Exists?

Git wants to preserve:

```text
Complete Branch History
```

---

Nothing is rewritten.

---

History remains intact.

---

# Advantages Of Merge

## Preserves History

Complete branch history remains visible.

---

## Safe

No commit rewriting.

---

## Good For Teams

Easy to understand.

---

# Disadvantages Of Merge

History can become:

```text
Messy
```

---

Example

```text
Merge Commit

Merge Commit

Merge Commit

Merge Commit
```

---

Large projects can become difficult to read.

---

# What Is Git Rebase?

Rebase moves commits from one base to another.

---

Instead of creating:

```text
Merge Commit
```

Git rewrites commit history.

---

Command

```bash
git rebase main
```

---

# Rebase Example

Before Rebase

```text
main

A --- B --- C
```

```text
payment-feature

A --- B --- D --- E
```

---

Run

```bash
git rebase main
```

---

Git Replays

```text
D

E
```

on top of:

```text
C
```

---

After Rebase

```text
A --- B --- C --- D' --- E'
```

---

Notice:

```text
D'

E'
```

---

These are:

```text
New Commits
```

---

History Rewritten.

---

# Why Rebase Creates New Commits?

Because Git changes:

```text
Parent References
```

---

Original Commits

```text
D

E
```

---

Become

```text
D'

E'
```

---

New Commit IDs Generated.

---

# Visual Comparison

## Merge

```text
A --- B --- C
       \     \
        D --- E --- M
```

---

History Preserved.

---

## Rebase

```text
A --- B --- C --- D' --- E'
```

---

History Linear.

---

# Merge Workflow

Current Branch

```bash
git checkout payment-feature
```

---

Merge Main

```bash
git merge main
```

---

Git Creates

```text
Merge Commit
```

---

Done.

---

# Rebase Workflow

Current Branch

```bash
git checkout payment-feature
```

---

Rebase

```bash
git rebase main
```

---

Commits Replayed.

---

History Rewritten.

---

# Real Production Example

Main Branch

```text
Production
```

---

Feature Branch

```text
Payment Feature
```

---

Main Receives:

```text
10 New Commits
```

---

Developer Wants Latest Changes.

---

Option 1

```bash
git merge main
```

---

Option 2

```bash
git rebase main
```

---

Both work.

---

History looks different.

---

# Merge Conflict During Merge

Example

Developer A

```text
Line 10

PORT=8080
```

---

Developer B

```text
Line 10

PORT=9090
```

---

Git Cannot Decide.

---

Conflict Occurs.

---

# Conflict Example

Git Shows

```text
<<<<<<< HEAD

PORT=8080

=======

PORT=9090

>>>>>>> main
```

---

Developer Must Choose.

---

Resolve

```text
PORT=8080

or

PORT=9090
```

---

Commit Changes.

---

# Conflict During Rebase

Rebase Can Also Produce Conflicts.

---

Flow

```bash
git rebase main
```

---

Conflict Appears.

---

Fix File.

---

Continue

```bash
git rebase --continue
```

---

Abort Rebase

```bash
git rebase --abort
```

---

# When To Use Merge?

Recommended When:

```text
Working In Teams

Preserve History

Shared Branches
```

---

Common For:

```text
Pull Requests
```

---

GitHub Default Strategy.

---

# When To Use Rebase?

Recommended When:

```text
Cleaning History

Updating Feature Branch

Before PR
```

---

Keeps History Linear.

---

# Golden Rule Of Rebase

Never Rebase:

```text
Public Shared Branches
```

---

Because Rebase:

```text
Rewrites History
```

---

Can Break Other Developers.

---

# GitHub Pull Request Merge Types

GitHub Supports

## Merge Commit

```text
Default
```

---

## Squash And Merge

Combines Multiple Commits.

---

## Rebase And Merge

Uses Rebase.

---

No Merge Commit Created.

---

# Production Workflow

Feature Branch

```text
payment-feature
```

---

Before Creating PR

```bash
git rebase main
```

---

Keeps Branch Updated.

---

Create PR.

---

Merge To Main.

---

Very Common Workflow.

---

# Common Interview Questions

## What Is Git Merge?

### Short Answer

Git Merge combines changes from one branch into another by creating a merge commit.

### Detailed Explanation

Merge preserves complete branch history and creates a special commit with two parent commits.

### Production Example

```text
main

↓

feature

↓

Merge

↓

Merge Commit
```

### Follow-Up Questions

- What is a merge commit?
- How many parents does it have?
- Does merge rewrite history?

---

## What Is Git Rebase?

### Short Answer

Git Rebase moves commits to a new base and rewrites history.

### Detailed Explanation

Rebase replays commits on top of another branch, creating a cleaner linear history.

### Production Example

```text
main

↓

Rebase Feature Branch

↓

Linear History
```

### Follow-Up Questions

- Does rebase create new commits?
- Why are commit IDs changed?
- Can rebase cause conflicts?

---

## Difference Between Merge And Rebase?

### Short Answer

Merge preserves history using a merge commit, while rebase rewrites history and creates a linear commit structure.

### Detailed Explanation

Merge is safer for shared branches. Rebase creates cleaner commit history but changes commit hashes.

### Production Example

Merge

```text
Branch History Preserved
```

Rebase

```text
Linear History
```

### Follow-Up Questions

- Which is safer?
- Which creates merge commits?
- Which rewrites history?

---

## Why Shouldn't We Rebase Shared Branches?

### Short Answer

Because rebase changes commit hashes and rewrites history.

### Detailed Explanation

Other developers may already have the original commits. Rewriting them causes synchronization problems and conflicts.

### Production Example

```text
Developer A Rebase

↓

Developer B Pull

↓

History Mismatch
```

### Follow-Up Questions

- What branches can be rebased?
- What happens if shared branch is rebased?
- How do teams avoid this issue?

---

# Production Scenarios

## Scenario 1

Feature Branch Behind Main.

### Solution

```bash
git merge main
```

or

```bash
git rebase main
```

---

## Scenario 2

PR Shows Merge Conflicts.

### Solution

```bash
git merge main
```

Resolve Conflicts.

Commit Again.

---

## Scenario 3

Need Cleaner Commit History.

### Solution

```bash
git rebase main
```

---

## Scenario 4

Team Uses Shared Branch.

### Recommendation

```text
Use Merge

Avoid Rebase
```

---

## Scenario 5

Rebase Gone Wrong.

### Abort

```bash
git rebase --abort
```

---

# Key Takeaways

```text
Git Merge combines branches using a merge commit.

Merge commits have two parent commits.

Git Rebase rewrites history and creates new commit hashes.

Merge preserves complete branch history.

Rebase creates a cleaner linear history.

Both merge and rebase can produce conflicts.

Merge is safer for shared branches.

Rebase is useful for cleaning feature branch history.

Never rebase public shared branches.

Understanding Merge vs Rebase is one of the most important Git interview topics.
```