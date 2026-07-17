# Git Reset

## Introduction

While working with Git, developers often make mistakes.

Examples:

```text
Wrong Commit

Wrong Files Added

Wrong Commit Message

Committed Too Early

Committed Sensitive Data
```

---

Need a way to:

```text
Undo Changes
```

---

Git provides:

```text
Git Reset
```

---

# Problem Statement

Suppose you committed changes.

```text
Commit A

↓

Commit B

↓

Commit C
```

---

Later you realize:

```text
Commit C Is Wrong
```

---

Question:

```text
How Can We Undo It?
```

---

Answer:

```text
Git Reset
```

---

# What Is Git Reset?

Git Reset is used to:

```text
Move HEAD To Previous Commit

Undo Commits

Modify Commit History
```

---

Important:

```text
Git Reset Rewrites History
```

---

Therefore:

```text
Suitable For

Private Branches

Local Commits
```

---

Avoid On:

```text
Shared Branches

Remote Branches

Main Branch
```

---

# Understanding HEAD

HEAD points to:

```text
Current Commit
```

---

Example

```text
A → B → C
        ↑
       HEAD
```

---

Current Branch Points To:

```text
Commit C
```

---

# What Happens During Reset?

Before Reset

```text
A → B → C
```

---

Command

```bash
git reset B
```

---

After Reset

```text
A → B
    ↑
   HEAD
```

---

Commit C is removed from branch history.

---

# Types Of Reset

Git Supports:

```text
Soft Reset

Mixed Reset

Hard Reset
```

---

Most Frequently Asked Git Interview Topic.

---

# Soft Reset

Command

```bash
git reset --soft HEAD~1
```

---

Removes:

```text
Commit
```

---

Keeps:

```text
Staging Area

Working Directory
```

---

# Flow

Before

```text
Working Directory

↓

Staging Area

↓

Commit
```

---

After

```text
Working Directory

↓

Staging Area
```

---

Commit Removed.

---

Changes Still Staged.

---

# When To Use Soft Reset?

Useful When:

```text
Wrong Commit Message

Forgot To Add Files

Need To Recommit
```

---

Example

```bash
git commit -m "test"
```

---

Need Better Message.

---

Use

```bash
git reset --soft HEAD~1
```

---

Commit Again

```bash
git commit -m "Added Payment Validation"
```

---

# Mixed Reset

Default Reset Type.

---

Command

```bash
git reset HEAD~1
```

---

or

```bash
git reset --mixed HEAD~1
```

---

Removes:

```text
Commit

Staging Area
```

---

Keeps:

```text
Working Directory
```

---

# Flow

Before

```text
Working Directory

↓

Staging Area

↓

Commit
```

---

After

```text
Working Directory
```

---

Files Become:

```text
Unstaged
```

---

Changes Still Exist.

---

# When To Use Mixed Reset?

Useful When:

```text
Wrong Files Added

Need To Review Changes Again

Need To Stage Selectively
```

---

# Hard Reset

Most Dangerous Reset.

---

Command

```bash
git reset --hard HEAD~1
```

---

Removes:

```text
Commit

Staging Area

Working Directory
```

---

Everything Is Deleted.

---

# Flow

Before

```text
Working Directory

↓

Staging Area

↓

Commit
```

---

After

```text
Nothing Remains
```

---

# When To Use Hard Reset?

Only When:

```text
Changes Not Needed

Discard Experimental Work

Rollback Local Changes
```

---

# Reset Comparison

## Soft Reset

Removes

```text
Commit
```

Keeps

```text
Staging Area

Working Directory
```

---

## Mixed Reset

Removes

```text
Commit

Staging Area
```

Keeps

```text
Working Directory
```

---

## Hard Reset

Removes

```text
Commit

Staging Area

Working Directory
```

Keeps

```text
Nothing
```

---

# Real Production Example

Developer Commits:

```text
Debug Statements
```

---

Commit Not Yet Pushed.

---

Use

```bash
git reset --soft HEAD~1
```

---

Remove Debug Logs.

---

Commit Again.

---

# Another Production Example

Developer Commits:

```text
Wrong Terraform Changes
```

---

Needs Review.

---

Use

```bash
git reset HEAD~1
```

---

Changes Stay In Working Directory.

---

Review Again.

---

# Why Reset Is Not Recommended On Shared Branches?

Developer A Pushes:

```text
Commit C
```

---

Developer B Pulls:

```text
Commit C
```

---

Developer A Executes:

```bash
git reset --hard HEAD~1
```

---

Then Force Pushes.

---

Now:

```text
Developer A History

≠

Developer B History
```

---

This Creates Problems.

---

# Golden Rule

Use Reset For:

```text
Local Commits

Private Branches
```

---

Do Not Use Reset For:

```text
Main Branch

Shared Branches

Remote Branches
```

---

# Useful Commands

Soft Reset

```bash
git reset --soft HEAD~1
```

---

Mixed Reset

```bash
git reset HEAD~1
```

---

Hard Reset

```bash
git reset --hard HEAD~1
```

---

Reset To Specific Commit

```bash
git reset --hard <commit-id>
```

---

# Common Interview Questions

## What Is Git Reset?

### Short Answer

Git Reset is used to undo commits by moving the branch pointer to a previous commit.

### Detailed Explanation

Git Reset modifies commit history and moves HEAD to a specified commit. Depending on the reset type, it can preserve or remove staged and working directory changes.

### Production Example

```bash
git reset --soft HEAD~1
```

Used when a developer wants to change a commit message without losing changes.

### Follow-Up Questions

```text
Does Reset Rewrite History?

Can Reset Delete Commits?

When Should Reset Be Used?

Why Is Reset Unsafe On Shared Branches?
```

---

## Difference Between Soft, Mixed And Hard Reset?

### Short Answer

Soft keeps staging and working directory, mixed keeps only working directory, hard removes everything.

### Detailed Explanation

The difference lies in what Git preserves after moving HEAD.

### Production Example

```text
Soft

↓

Keep Staged Changes
```

```text
Mixed

↓

Keep Working Directory Changes
```

```text
Hard

↓

Delete Everything
```

### Follow-Up Questions

```text
Which Reset Is Default?

Which Is Most Dangerous?

Which Is Most Common?
```

---

## Why Is Hard Reset Dangerous?

### Short Answer

Because it removes commits, staged changes, and working directory changes.

### Detailed Explanation

Hard Reset permanently removes local work unless it can be recovered through Git Reflog.

### Production Example

```bash
git reset --hard HEAD~1
```

↓

Work Lost.

### Follow-Up Questions

```text
Can Reflog Recover It?

Should Hard Reset Be Used On Main?

When Is Hard Reset Appropriate?
```

---

## Why Is Reset Not Recommended On Shared Branches?

### Short Answer

Because Reset rewrites history.

### Detailed Explanation

When commits have already been shared with other developers, rewriting history can cause synchronization issues and merge problems.

### Production Example

```text
Developer A

↓

Reset

↓

Force Push

↓

Developer B History Breaks
```

### Follow-Up Questions

```text
What Should Be Used Instead?

What Happens After Force Push?

Why Is Revert Safer?
```

---

# Production Scenarios

## Scenario 1

Wrong Commit Message.

### Solution

```bash
git reset --soft HEAD~1
```

---

## Scenario 2

Wrong Files Included In Commit.

### Solution

```bash
git reset HEAD~1
```

---

## Scenario 3

Need To Discard Local Changes.

### Solution

```bash
git reset --hard HEAD~1
```

---

## Scenario 4

Commit Already Pushed To Main.

### Recommendation

```text
Use Revert

Not Reset
```

---

## Scenario 5

Accidentally Deleted Commit Using Reset.

### Solution

```text
Git Reflog
```

Recover Commit.

---

# Key Takeaways

```text
Git Reset is used to undo commits.

Reset rewrites history.

Soft Reset keeps staged and working directory changes.

Mixed Reset keeps working directory changes.

Hard Reset removes everything.

Reset is best suited for local commits and private branches.

Avoid Reset on shared branches.

Hard Reset is powerful but dangerous.

Git Reflog can help recover deleted commits.
```