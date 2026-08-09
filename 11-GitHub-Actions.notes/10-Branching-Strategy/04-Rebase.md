# Rebase

Rebase is a Git operation that moves or reapplies commits from one branch on top of another commit.

It is commonly used to:

- Keep feature branches updated
- Create a linear history
- Reduce unnecessary merge commits
- Prepare a Pull Request before merging
- Keep development history easier to understand

A typical workflow is:

```text
main
  |
  ↓
Feature Branch
  |
  ↓
main changes
  |
  ↓
Rebase
  |
  ↓
Feature Branch updated
  |
  ↓
CI
  |
  ↓
Pull Request
  |
  ↓
Merge
```

---

# 1. What Is Rebase?

Suppose we have:

```text
main:

A ─── B ─── C
```

A feature branch was created from `B`:

```text
main:
A ─── B ─── C

feature:
       \
        D ─── E
```

The feature branch is now behind `main`.

Rebase moves the feature commits onto the latest `main`:

```text
A ─── B ─── C ─── D' ─── E'
```

The commits `D` and `E` become new commits `D'` and `E'`.

---

# 2. Why Use Rebase?

Rebase can help maintain a linear history.

Without rebase:

```text
A ─── B ─── C ───── M
       \           /
        D ─── E ───
```

With rebase:

```text
A ─── B ─── C ─── D' ─── E'
```

The second history is easier to follow.

---

# 3. Basic Rebase Command

First update your remote information:

```bash
git fetch origin
```

Switch to your feature branch:

```bash
git switch feature/user-login
```

Rebase onto main:

```bash
git rebase origin/main
```

---

# 4. Complete Rebase Workflow

```bash
git switch feature/user-login

git fetch origin

git rebase origin/main
```

If there are no conflicts:

```text
Successfully rebased
```

Then verify:

```bash
git status
```

---

# 5. Why Use `git fetch` First?

`git fetch` updates your local knowledge of the remote repository.

Example:

```bash
git fetch origin
```

This updates:

```text
origin/main
origin/feature/*
```

without changing your working branch.

Then:

```bash
git rebase origin/main
```

uses the latest remote `main`.

---

# 6. Rebase vs Merge

Suppose:

```text
main:
A ─── B ─── C

feature:
       \
        D ─── E
```

### Merge

```text
A ─── B ─── C ───── M
       \           /
        D ─── E ───
```

### Rebase

```text
A ─── B ─── C ─── D' ─── E'
```

---

# 7. Rebase Does Not Move Main

When you run:

```bash
git rebase origin/main
```

while on your feature branch, you are changing the feature branch.

You are not changing:

```text
main
```

The result is:

```text
main
  |
  └── C

feature
  |
  └── C → D' → E'
```

---

# 8. Rebase Changes Commit SHAs

Before rebase:

```text
D = abc123
E = def456
```

After rebase:

```text
D' = 111aaa
E' = 222bbb
```

The content may be logically similar, but the parent relationship changed.

Therefore Git creates new commit identities.

---

# 9. Why Commit SHA Changes

A Git commit contains information about its parent.

Conceptually:

```text
Commit
├── Tree
├── Parent
├── Author
├── Message
└── Metadata
```

When the parent changes:

```text
D → parent B
```

becomes:

```text
D' → parent C
```

The commit identity changes.

---

# 10. Rebase a Feature Branch

Example:

```bash
git switch feature/payment-timeout
git fetch origin
git rebase origin/main
```

After successful rebase:

```text
main:
A → B → C

feature:
A → B → C → D' → E'
```

---

# 11. Push After Rebase

Because rebase rewrites the feature branch history, a normal push may be rejected.

You may need:

```bash
git push --force-with-lease
```

Do not immediately use:

```bash
git push --force
```

---

# 12. Why `--force-with-lease`?

`--force-with-lease` checks whether the remote branch has changed unexpectedly.

Example:

```text
Remote feature:
A → B → C
```

You rebase locally:

```text
A → B → C → D' → E'
```

If someone else pushed:

```text
A → B → C → X
```

then:

```bash
git push --force-with-lease
```

can detect that the remote branch is no longer in the state you expected.

This provides additional protection.

---

# 13. Never Rebase Main

Avoid:

```bash
git switch main
git rebase ...
```

on a shared protected `main`.

The problem is that rebase rewrites history.

For shared branches, prefer:

```text
Merge
or
Revert
```

according to the team's policy.

---

# 14. Shared Branches and Rebase

Avoid rebasing:

```text
main
production
shared release branches
team branches
```

unless the team explicitly follows a controlled history-rewriting policy.

Rebase is safest for:

```text
Personal feature branches
Short-lived development branches
```

---

# 15. Rebase a Branch onto Main

Common workflow:

```bash
git switch feature/user-login

git fetch origin

git rebase origin/main
```

If successful:

```bash
git push --force-with-lease
```

Then:

```text
Feature Branch
      |
      ↓
Updated with main
      |
      ↓
CI
      |
      ↓
PR
      |
      ↓
Review
      |
      ↓
Merge
```

---

# 16. Rebase Conflict

Suppose:

```text
main:
A → B → C

feature:
A → B → D
```

Both `C` and `D` modify the same section.

Running:

```bash
git rebase origin/main
```

may produce:

```text
CONFLICT
```

Git stops the rebase.

---

# 17. Check Rebase Status

Run:

```bash
git status
```

Git will show:

```text
You are currently rebasing
```

and identify conflicted files.

---

# 18. Find Conflict Files

```bash
git status
```

or:

```bash
git diff --name-only --diff-filter=U
```

---

# 19. Conflict Markers

A conflicted file may contain:

```text
<<<<<<< HEAD
main version
=======
feature version
>>>>>>> feature
```

You need to decide which code should remain.

---

# 20. Resolve the Conflict

Edit the file and remove the conflict markers.

Then:

```bash
git add <file>
```

Continue:

```bash
git rebase --continue
```

---

# 21. Multiple Conflicts

A rebase may stop multiple times.

Flow:

```text
Rebase
  ↓
Conflict
  ↓
Resolve
  ↓
git add
  ↓
git rebase --continue
  ↓
Another conflict?
  |
  ├── Yes → Resolve again
  |
  └── No → Rebase complete
```

---

# 22. Abort Rebase

If the rebase becomes too complicated:

```bash
git rebase --abort
```

This attempts to return the branch to the state it was in before the rebase began.

This is useful when:

```text
Conflicts are extensive
Wrong branch was selected
You realize the rebase is unnecessary
You want to restart
```

---

# 23. Skip a Commit

Sometimes a commit may no longer be required.

Git provides:

```bash
git rebase --skip
```

Use this carefully.

Before skipping, understand what the commit contains and why it is safe to omit.

---

# 24. Rebase Continue

After resolving a conflict:

```bash
git add .
git rebase --continue
```

Git continues applying the remaining commits.

---

# 25. Rebase Workflow with Conflicts

```text
git fetch origin
        |
        ↓
git rebase origin/main
        |
        ↓
Conflict?
   ┌────┴────┐
  No        Yes
   |          |
   ↓          ↓
Done      Resolve
             |
             ↓
          git add
             |
             ↓
      git rebase --continue
             |
             ↓
          Complete
```

---

# 26. Interactive Rebase

Interactive rebase allows you to modify commit history.

Example:

```bash
git rebase -i HEAD~5
```

This opens the last five commits for editing.

Possible operations include:

```text
pick
reword
edit
squash
fixup
drop
```

---

# 27. `pick`

Example:

```text
pick abc123 Add login
```

Keeps the commit.

---

# 28. `reword`

Example:

```text
reword abc123 Add login
```

Keeps the changes but allows the commit message to be changed.

---

# 29. `squash`

Example:

```text
pick abc123 Add login
squash def456 Fix login
```

The commits are combined.

Git allows you to edit the resulting commit message.

---

# 30. `fixup`

Example:

```text
pick abc123 Add login
fixup def456 Fix typo
```

The changes are combined and the fixup commit's message is discarded.

This is useful for cleaning up development commits.

---

# 31. `drop`

Example:

```text
pick abc123 Add login
drop def456 Temporary debug
```

The selected commit is removed from the rebased history.

Use carefully.

---

# 32. Clean Feature Branch

Before opening a PR, you might have:

```text
Add feature
Fix typo
Fix test
Fix test again
Update comments
Final fix
```

Interactive rebase can turn this into:

```text
Implement payment timeout
```

This produces a cleaner feature history.

---

# 33. Interactive Rebase Example

```bash
git rebase -i HEAD~5
```

Example editor:

```text
pick abc123 Add payment timeout
pick def456 Fix test
pick ghi789 Fix typo
pick jkl012 Update comments
pick mno345 Final fix
```

Could become:

```text
pick abc123 Add payment timeout
fixup def456 Fix test
fixup ghi789 Fix typo
fixup jkl012 Update comments
fixup mno345 Final fix
```

Result:

```text
One logical commit
```

---

# 34. Rebase Before Pull Request

A developer may use:

```bash
git fetch origin
git rebase origin/main
```

before opening a PR.

This helps ensure the feature is based on the latest main branch.

---

# 35. Rebase Before Merge

Depending on repository policy:

```text
PR
 ↓
Main changed
 ↓
Feature behind
 ↓
Rebase
 ↓
CI again
 ↓
Review
 ↓
Merge
```

Do not assume a PR is still valid simply because it passed CI before `main` changed.

---

# 36. Why CI Must Run Again After Rebase

Suppose:

```text
Feature CI
 ↓
Passed
```

Then `main` changes.

After rebase:

```text
Feature + New Main
```

The combined state may behave differently.

Therefore:

```text
Rebase
 ↓
CI Again
```

is important.

---

# 37. GitHub Actions and Rebase

A Pull Request may trigger GitHub Actions after the feature branch changes.

Example:

```yaml
name: PR CI

on:
  pull_request:
    branches:
      - main

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Test
        run: |
          ./scripts/test.sh
```

After rebasing and pushing:

```text
Feature Branch
      |
      ↓
New Commit History
      |
      ↓
PR Updated
      |
      ↓
GitHub Actions
      |
      ↓
CI
```

---

# 38. Rebase and Branch Protection

Branch protection can require:

```text
PR
 ↓
Required Checks
 ↓
Approval
 ↓
Up-to-date Branch
 ↓
Merge
```

If the branch is behind main, the repository may require the developer to update it.

---

# 39. Rebase and Required Status Checks

After a rebase:

```text
Old SHA
 ↓
New SHA
```

The previous CI result may no longer represent the current commit state.

Therefore GitHub should validate the updated commit.

---

# 40. Rebase and Commit SHA Traceability

Rebase changes commit SHAs.

Example:

Before:

```text
Feature Commit:
abc123
```

After:

```text
Rebased Commit:
789xyz
```

Therefore don't use a feature branch SHA as the permanent production identity before the final merge/release process.

For production traceability, track the final:

```text
Merged Commit
Artifact Digest
Release
Deployment
```

---

# 41. Rebase and Docker

A feature build may use:

```text
abc123
```

After rebase:

```text
789xyz
```

If you build again, the resulting artifact should be associated with the current source state.

Production should use the final validated artifact rather than an obsolete feature build.

---

# 42. Rebase and ECR

Example:

```text
Feature
 ↓
Commit abc123
 ↓
Rebase
 ↓
Commit 789xyz
 ↓
CI
 ↓
Docker Build
 ↓
ECR
```

The final release should reference the validated source/artifact identity.

---

# 43. Rebase and GitOps

Application repository:

```text
Feature
 ↓
Rebase
 ↓
PR
 ↓
Merge
 ↓
Build Image
```

GitOps repository:

```text
Update Image
 ↓
GitOps PR
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
 ↓
EKS
```

The GitOps repository should contain the final desired state.

---

# 44. Rebase and Terraform

Terraform feature branch:

```text
feature/update-vpc
```

Before PR:

```bash
git fetch origin
git rebase origin/main
```

Then:

```text
terraform fmt
terraform validate
terraform plan
```

Then:

```text
PR
 ↓
Review
 ↓
Merge
```

---

# 45. Rebase and Kubernetes

For Kubernetes changes:

```text
feature/update-deployment
        |
        ↓
git rebase origin/main
        |
        ↓
YAML / Helm validation
        |
        ↓
Trivy
        |
        ↓
PR
        |
        ↓
Review
```

---

# 46. Rebase and Helm

Example:

```bash
git fetch origin
git rebase origin/main
```

Then:

```bash
helm lint ./chart
```

Optionally render manifests:

```bash
helm template ./chart
```

Then run the appropriate security and validation checks.

---

# 47. Rebase and DevSecOps

A production feature branch may follow:

```text
Feature
   |
   ↓
Rebase
   |
   ↓
Build
   |
   ↓
Tests
   |
   ↓
SonarQube
   |
   ↓
Trivy
   |
   ↓
Veracode
   |
   ↓
PR
   |
   ↓
Review
```

Rebase does not replace these checks.

---

# 48. Rebase and Security

Rebase itself is not a security control.

Security still requires:

```text
Branch Protection
Least Privilege
CI Security
Secret Management
Code Review
CODEOWNERS
Security Scanning
Environment Protection
```

---

# 49. Rebase and Secrets

Never use a rebase operation as a method of removing a secret from repository history.

If a secret has been committed:

```text
Rebase alone
```

is not sufficient.

The secret must be:

```text
Revoked
Rotated
Removed appropriately
History remediated where required
```

Treat leaked credentials as compromised.

---

# 50. Rebase vs Reset

### Rebase

```bash
git rebase
```

Reapplies commits onto another base.

### Reset

```bash
git reset
```

Moves a branch pointer and can change working/staging state depending on the mode.

They solve different problems.

---

# 51. Rebase vs Merge vs Reset

```text
Merge
 ↓
Combine histories

Rebase
 ↓
Replay commits on new base

Reset
 ↓
Move branch pointer
```

Use the operation appropriate to the situation.

---

# 52. `git pull --rebase`

Git can pull remote changes using rebase.

Example:

```bash
git pull --rebase origin main
```

Conceptually:

```text
Fetch
  +
Rebase
```

This can avoid unnecessary merge commits in some workflows.

---

# 53. Configure Pull Rebase

For a repository:

```bash
git config pull.rebase true
```

For all repositories for the user:

```bash
git config --global pull.rebase true
```

Only configure this if it matches your team's workflow.

---

# 54. Rebase and Local Changes

Before rebasing, check:

```bash
git status
```

Ideally the working tree should be clean.

If you have uncommitted changes:

```text
Working Changes
     |
     ↓
Rebase
```

can complicate the operation.

Commit, stash, or otherwise handle local changes according to the situation.

---

# 55. Stash Before Rebase

If appropriate:

```bash
git stash
```

Then:

```bash
git fetch origin
git rebase origin/main
```

Restore:

```bash
git stash pop
```

Always inspect the resulting changes.

---

# 56. Rebase with Autosquash

Developers can create fixup commits:

```bash
git commit --fixup <commit-sha>
```

Then use:

```bash
git rebase -i --autosquash origin/main
```

Git can automatically position fixup commits next to their target commits.

This is useful for cleaning feature branches.

---

# 57. Rebase and Signed Commits

Rebase changes commit identities.

Therefore signatures associated with old commits may not carry over to new commits in the same way.

Organizations requiring commit signing should understand how their signing policy interacts with:

```text
Rebase
Squash
Merge
```

---

# 58. Rebase and Shared Development

Before rebasing a shared feature branch:

```text
Check whether others use it
```

If others depend on it:

```text
Coordinate first
```

Otherwise they may need to recover from rewritten history.

---

# 59. Rebase Safety Checklist

Before rebasing:

```text
☐ Am I on the correct branch?
☐ Is this branch shared?
☐ Did I fetch latest changes?
☐ Is my working tree clean?
☐ Do I understand the conflicts?
☐ Do I have a recovery point?
```

---

# 60. Create a Backup Branch

Before a complicated rebase:

```bash
git branch backup/payment-feature
```

Then:

```bash
git rebase origin/main
```

If something goes wrong, the backup provides another reference to the previous state.

---

# 61. Inspect History

Before:

```bash
git log --oneline --graph --decorate --all
```

After:

```bash
git log --oneline --graph --decorate --all
```

Compare the history.

This is especially useful after complex rebases.

---

# 62. Verify the Diff

Before pushing:

```bash
git diff origin/main...HEAD
```

Check that the resulting feature changes are exactly what you expect.

Do not blindly push after a complicated conflict resolution.

---

# 63. Verify Tests

After rebase:

```text
Rebase
 ↓
Build
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Security Checks
 ↓
PR
```

A successful rebase does not mean the application is correct.

---

# 64. Production Rebase Workflow

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
git fetch origin
    |
    ↓
git rebase origin/main
    |
    ↓
Resolve Conflicts
    |
    ↓
Run Tests
    |
    ↓
Security Scans
    |
    ↓
git push --force-with-lease
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 65. Rebase in an Enterprise Environment

Recommended policy:

```text
main
    |
    ├── Protected
    ├── No force push
    └── No history rewriting

feature/*
    |
    ├── Short-lived
    ├── Rebase allowed
    └── Force-with-lease allowed when appropriate

release/*
    |
    └── Controlled according to release policy
```

---

# 66. Rebase with Branch Protection

A strong configuration:

```text
main
 |
 ├── Pull Request Required
 ├── Required Reviews
 ├── Required CI
 ├── Required Security
 ├── Force Push Disabled
 └── Deletion Restricted
```

Developers can still rebase their own feature branches before merging.

---

# 67. When NOT to Rebase

Avoid rebasing when:

```text
Branch is shared heavily
Branch is already published and depended upon
Branch is protected
History must remain stable
The team explicitly prohibits history rewriting
```

Use merge or another approved approach instead.

---

# 68. When Rebase Is Useful

Rebase is useful when:

```text
Feature branch is behind main
You want a linear history
You want to clean local commits
You need to resolve integration conflicts before PR merge
You are preparing a short-lived feature branch
```

---

# 69. Rebase Decision Tree

```text
Need to update feature branch?
          |
          ↓
Is it shared?
      ┌───┴───┐
     No      Yes
      |        |
      ↓        ↓
   Rebase   Coordinate
      |        |
      ↓        ↓
   CI Again  Merge/Rebase
               |
               ↓
             Policy
```

---

# 70. Recommended Workflow for Your DevOps Projects

For application repositories:

```text
main
 |
 └── feature/*
       |
       ↓
   Development
       |
       ↓
git fetch origin
       |
       ↓
git rebase origin/main
       |
       ↓
Tests
       |
       ↓
SonarQube
       |
       ↓
Trivy
       |
       ↓
PR
       |
       ↓
Review
       |
       ↓
Squash Merge
       |
       ↓
main
```

For GitOps:

```text
Application PR
      |
      ↓
Merge
      |
      ↓
Build
      |
      ↓
ECR
      |
      ↓
GitOps PR
      |
      ↓
Review
      |
      ↓
Merge
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

---

# 71. Common Mistakes

### Mistake 1: Rebasing Main

```text
Dangerous
```

Avoid history rewriting on protected shared branches.

### Mistake 2: Blind Force Push

Bad:

```bash
git push --force
```

Prefer:

```bash
git push --force-with-lease
```

when force-pushing a rebased personal branch is appropriate.

### Mistake 3: Rebasing Shared Branches

Coordinate first.

### Mistake 4: Not Running CI Again

After rebase:

```text
CI Again
```

### Mistake 5: Not Checking the Diff

Always inspect the resulting changes.

### Mistake 6: Resolving Conflicts Carelessly

A conflict resolution can compile successfully but still introduce incorrect behavior.

### Mistake 7: Assuming Rebase Is a Rollback

Rebase is a history operation, not a production rollback strategy.

### Mistake 8: Forgetting Commit SHA Changes

Rebase creates new commit identities.

---

# 72. Best Practices

```text
☐ Rebase feature branches, not protected main
☐ Fetch before rebasing
☐ Keep feature branches short-lived
☐ Avoid rebasing shared branches without coordination
☐ Resolve conflicts carefully
☐ Run tests after rebase
☐ Run security checks again
☐ Inspect the final diff
☐ Use --force-with-lease when appropriate
☐ Protect main from force pushes
☐ Maintain commit-to-artifact traceability
☐ Keep production artifacts immutable
☐ Use CI to validate the final commit state
☐ Document the team's rebase policy
```

---

# 73. Interview Questions

## Basic

1. What is Git rebase?
2. Why do we use rebase?
3. What is the difference between merge and rebase?
4. Does rebase change commit SHAs?
5. Why does rebase create new commit SHAs?
6. How do you rebase a feature branch onto main?
7. What is `git fetch`?
8. What does `git rebase --continue` do?
9. What does `git rebase --abort` do?
10. What does `git rebase --skip` do?

## Intermediate

11. Why should you avoid rebasing main?
12. What is `git push --force-with-lease`?
13. Why is `--force-with-lease` safer than `--force`?
14. How do you resolve a rebase conflict?
15. How do you verify the result of a rebase?
16. What is interactive rebase?
17. What are `pick`, `reword`, `squash`, `fixup`, and `drop`?
18. How do you clean up feature branch commits?
19. How do you rebase a feature branch before opening a PR?
20. Why should CI run again after a rebase?
21. What happens to a PR after rebasing its source branch?
22. When should you avoid rebasing?
23. What is the difference between `git pull` and `git pull --rebase`?
24. How can you safely recover from a complicated rebase?
25. Why should you inspect the diff after resolving conflicts?

## Advanced / Production

26. Design a safe rebase policy for an enterprise GitHub organization.
27. How would you allow developers to rebase feature branches while preventing force pushes to main?
28. How would you handle a shared feature branch that multiple developers are using?
29. How would you integrate rebase into a GitHub Actions PR workflow?
30. Why must security checks run again after a rebase?
31. How does rebase affect Docker image traceability?
32. How does rebase affect ECR artifact tracking?
33. How does rebase affect GitOps and ArgoCD workflows?
34. How would you maintain JIRA → PR → commit SHA → artifact → deployment traceability after rebasing?
35. How would you handle a rebase conflict involving Terraform infrastructure?
36. How would you handle a rebase conflict involving Kubernetes manifests?
37. How would you handle a rebase conflict in `.github/workflows`?
38. What security risks exist when rebasing and force-pushing?
39. Why should privileged self-hosted runners not be exposed to untrusted PR code?
40. How would you design a production Git workflow using feature branches, rebase, PRs, branch protection, CI, SonarQube, Trivy, Veracode, ECR, GitOps, ArgoCD, and EKS?
41. A feature branch passed CI yesterday but is now behind main. How would you safely update it?
42. A developer accidentally rebased a shared branch and force-pushed it. How would you recover?
43. During a rebase, a Terraform change and Kubernetes change conflict with main. How would you resolve and validate both safely?
44. A PR is green before rebase but fails afterward. How would you investigate?
45. Explain when you would choose rebase, squash merge, or merge commit in a production DevOps organization.