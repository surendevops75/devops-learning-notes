# Tagging and Releases

Git tagging is used to mark specific points in Git history, usually for releases, versions, milestones, or stable production states.

A tag gives a meaningful name to a specific commit.

```text
Commit A
   |
Commit B
   |
Commit C  ← v1.0.0
   |
Commit D
   |
Commit E  ← v1.1.0
```

Git tags are commonly used for:

```text
Release Management
Versioning
Production Releases
Rollback References
Deployment Traceability
Auditability
Milestones
```

---

# Why Git Tags Matter

Without tags, identifying which commit represents a production release can become difficult.

Example:

```text
Commit A
Commit B
Commit C
Commit D
Commit E
```

With tags:

```text
Commit A
Commit B
Commit C  ← v1.0.0
Commit D
Commit E  ← v1.1.0
```

Now the release version is clearly identified.

Tags provide:

```text
Version Identification
Release Tracking
Rollback Reference
Deployment Traceability
Auditability
```

---

# What Is a Git Tag?

A Git tag is a reference that points to a specific Git object, commonly a commit.

Example:

```bash
git tag v1.0.0
```

Conceptually:

```text
v1.0.0
   |
   ↓
Commit SHA
```

Instead of referring to a commit using:

```text
8f3a91d
```

you can refer to it using:

```text
v1.0.0
```

This makes release management easier.

---

# Tag vs Branch

A branch is normally used for ongoing development.

A tag is normally used to mark a specific point in history.

Branch:

```text
main
  |
  ↓
Commit A → Commit B → Commit C → Commit D
                                      |
                                      ↓
                                    HEAD
```

The branch moves when new commits are added.

Tag:

```text
v1.0.0
   |
   ↓
Commit C
```

New commits can continue after Commit C:

```text
Commit A → Commit B → Commit C → Commit D → Commit E
                       ↑
                    v1.0.0
```

The tag still identifies Commit C.

---

# Branch vs Tag Comparison

| Feature | Branch | Tag |
|---|---|---|
| Purpose | Ongoing development | Mark specific point |
| Moves | Yes | Normally no |
| Used for releases | Sometimes | Commonly |
| Used for development | Yes | No |
| Example | `main` | `v1.0.0` |
| Points to commit | Yes | Yes |
| Normally updated | Yes | No |

Interview answer:

> A branch is a movable pointer used for ongoing development, while a tag normally points to a fixed commit and is commonly used to identify releases.

---

# Types of Git Tags

Git provides two commonly used tag types:

```text
1. Lightweight Tag
2. Annotated Tag
```

---

# Lightweight Tag

A lightweight tag is simply a reference to a commit.

Create:

```bash
git tag v1.0.0
```

List tags:

```bash
git tag
```

Example:

```text
v1.0.0
v1.1.0
v2.0.0
```

A lightweight tag does not contain additional tag metadata.

It is useful when you simply want to mark a commit.

---

# Annotated Tag

An annotated tag is a Git object that contains additional metadata.

It can contain:

```text
Tag Name
Tagger
Date
Message
Referenced Object
```

Create an annotated tag:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

View the tag:

```bash
git show v1.0.0
```

Example:

```text
tag v1.0.0
Tagger: Surendra
Date: ...

Release version 1.0.0
```

For official production releases, annotated tags are generally preferred.

---

# Lightweight vs Annotated Tags

| Feature | Lightweight | Annotated |
|---|---|---|
| Points to commit | Yes | Yes |
| Tag message | No | Yes |
| Tagger information | No | Yes |
| Timestamp | No | Yes |
| Metadata | Minimal | Yes |
| Suitable for releases | Less preferred | Recommended |

Interview answer:

> A lightweight tag is simply a reference to a commit, while an annotated tag is a Git object containing metadata such as the tagger, timestamp, and message. I prefer annotated tags for production releases.

---

# Creating a Git Tag

First check the current branch:

```bash
git branch
```

Check recent commits:

```bash
git log --oneline
```

Example:

```text
8f3a91d Fix payment issue
7b2c123 Add notification service
4a5d789 Update deployment
```

Create an annotated tag:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

---

# Tagging a Specific Commit

You can create a tag on a specific commit.

Example:

```bash
git tag -a v1.0.0 4a5d789 -m "Release version 1.0.0"
```

Result:

```text
v1.0.0
   |
   ↓
4a5d789
```

This is useful when a particular commit has been selected for release.

---

# Listing Tags

List all tags:

```bash
git tag
```

Example:

```text
v1.0.0
v1.1.0
v1.2.0
v2.0.0
```

---

# Searching Tags

You can filter tags using patterns.

Example:

```bash
git tag -l "v1.*"
```

Output:

```text
v1.0.0
v1.1.0
v1.2.0
```

Another example:

```bash
git tag -l "v2.*"
```

---

# Sorting Tags by Version

Git tags can be sorted according to version order.

```bash
git tag --sort=-version:refname
```

Example:

```text
v2.0.0
v1.5.0
v1.4.2
v1.3.0
```

This is useful when working with Semantic Versioning.

---

# Viewing Tag Information

Use:

```bash
git show v1.0.0
```

This can show:

```text
Tag Information
Tagger
Date
Tag Message
Commit SHA
Commit Information
Changes
```

Example:

```text
tag v1.0.0
Tagger: Surendra

Release version 1.0.0

commit 8f3a91d
Author: Surendra
```

---

# Pushing a Tag to Remote

Creating a tag locally does not automatically push it to GitHub or GitLab.

Create:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

Push:

```bash
git push origin v1.0.0
```

Now the tag is available in the remote repository.

---

# Pushing All Tags

To push all local tags:

```bash
git push origin --tags
```

This pushes tags that are not already available on the remote.

For controlled releases, pushing a specific tag is often safer:

```bash
git push origin v1.0.0
```

---

# Fetching Tags

To fetch tags from a remote repository:

```bash
git fetch --tags
```

Then:

```bash
git tag
```

You can see the available tags.

---

# Checking Remote Tags

Use:

```bash
git ls-remote --tags origin
```

Example:

```text
abc123 refs/tags/v1.0.0
def456 refs/tags/v1.1.0
```

This displays tag references available on the remote repository.

---

# Deleting a Local Tag

Delete a tag locally:

```bash
git tag -d v1.0.0
```

This only removes the local tag.

It does not delete the remote tag.

---

# Deleting a Remote Tag

Delete a remote tag:

```bash
git push origin --delete v1.0.0
```

Another syntax is:

```bash
git push origin :refs/tags/v1.0.0
```

The first syntax is easier to understand.

---

# Important Warning About Tag Deletion

Production tags should normally be treated as immutable.

For example:

```text
v1.0.0
   |
   ↓
Commit A
```

should continue pointing to the same release commit.

Avoid deleting and recreating an existing production tag.

Otherwise:

```text
v1.0.0 → Commit A

Later:

v1.0.0 → Commit B
```

The same version now represents different code.

This can create:

```text
Deployment Confusion
Rollback Problems
Audit Problems
Traceability Problems
```

---

# Checking Out a Tag

You can inspect the code from a particular release.

```bash
git switch --detach v1.0.0
```

Older Git syntax:

```bash
git checkout v1.0.0
```

This normally results in a detached HEAD state.

---

# Detached HEAD

When HEAD points directly to a commit or tag instead of a branch, Git is in a detached HEAD state.

Example:

```text
main → Commit D

v1.0.0 → Commit B
             |
             ↓
            HEAD
```

You can inspect or test the code.

If you need to make changes, create a branch:

```bash
git switch -c hotfix-v1.0.0 v1.0.0
```

Now:

```text
hotfix-v1.0.0
       |
       ↓
    v1.0.0
```

---

# Semantic Versioning

Semantic Versioning is commonly called SemVer.

Format:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v2.4.1
```

Meaning:

```text
2 → MAJOR
4 → MINOR
1 → PATCH
```

---

# MAJOR Version

Increase the MAJOR version when there are breaking changes.

Example:

```text
v1.5.0
   ↓
v2.0.0
```

Examples:

```text
Breaking API Change
Breaking Configuration Change
Incompatible Interface Change
```

---

# MINOR Version

Increase the MINOR version when adding backward-compatible functionality.

Example:

```text
v1.4.0
   ↓
v1.5.0
```

Example:

```text
Existing APIs continue working
New functionality is added
```

---

# PATCH Version

Increase the PATCH version for backward-compatible fixes.

Example:

```text
v1.5.0
   ↓
v1.5.1
```

Typical examples:

```text
Bug Fix
Security Fix
Small Correction
```

---

# Semantic Versioning Example

Suppose the current release is:

```text
v2.4.3
```

Bug fix:

```text
v2.4.4
```

New backward-compatible feature:

```text
v2.5.0
```

Breaking change:

```text
v3.0.0
```

---

# Pre-Release Versions

Pre-release versions can be represented using identifiers.

Examples:

```text
v1.0.0-alpha
v1.0.0-beta
v1.0.0-rc.1
```

Common meanings:

```text
alpha → Early development
beta  → Testing / more mature version
rc    → Release Candidate
```

---

# Git Tags in CI/CD

Git tags are very useful in CI/CD pipelines.

Typical flow:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
Code Review
    |
    ↓
main
    |
    ↓
Release Tag
    |
    ↓
v1.2.0
    |
    ↓
CI/CD Pipeline
    |
    ├── Build
    ├── Test
    ├── Security Scan
    ├── Docker Build
    └── Deploy
```

---

# Tag-Based CI/CD

A pipeline can be configured to run when a tag is pushed.

Example:

```bash
git tag -a v1.2.0 -m "Release v1.2.0"

git push origin v1.2.0
```

The CI/CD system detects:

```text
v1.2.0
```

and starts the release pipeline.

---

# Jenkins and Git Tags

A Jenkins pipeline can be configured to build releases based on Git tags.

Typical flow:

```text
Git Tag
   |
   ↓
Jenkins Trigger
   |
   ↓
Checkout Tagged Commit
   |
   ↓
Maven Build
   |
   ↓
SonarQube
   |
   ↓
Trivy
   |
   ↓
Docker Build
   |
   ↓
Push Image
   |
   ↓
Deploy
```

The important point is that the pipeline builds the exact source represented by the release tag.

---

# GitHub Actions and Tags

GitHub Actions can trigger workflows when tags are pushed.

Example:

```yaml
on:
  push:
    tags:
      - 'v*'
```

This can trigger the workflow for:

```text
v1.0.0
v1.1.0
v1.2.0
v2.0.0
```

---

# Release vs Git Tag

A Git tag and a GitHub Release are not the same thing.

A Git tag is a Git reference pointing to a specific Git object, usually a commit.

A GitHub Release is a higher-level release feature associated with a tag.

Conceptually:

```text
Git Commit
    |
    ↓
Git Tag
    |
    ↓
GitHub Release
```

A release can contain:

```text
Release Title
Release Notes
Description
Tag
Release Assets
Artifacts
```

---

# Production Release Process

Suppose the application is ready for production.

Check status:

```bash
git status
```

Update the branch:

```bash
git pull origin main
```

Check commits:

```bash
git log --oneline
```

Create release tag:

```bash
git tag -a v1.0.0 -m "Production release v1.0.0"
```

Push tag:

```bash
git push origin v1.0.0
```

The CI/CD pipeline can then detect the tag and start the release process.

---

# Recommended Release Flow

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
Code Review
      |
      ↓
Merge to Main
      |
      ↓
Automated Tests
      |
      ↓
Security Scanning
      |
      ↓
Release Preparation
      |
      ↓
Create Git Tag
      |
      ↓
Push Tag
      |
      ↓
CI/CD Pipeline
      |
      ↓
Build Artifact
      |
      ↓
Container Image
      |
      ↓
Deploy
      |
      ↓
Validation
      |
      ↓
Production
```

---

# Git Tags and Docker Images

Git tags can be used to create meaningful Docker image versions.

Git tag:

```text
v1.4.0
```

Docker image:

```text
myapp:v1.4.0
```

Pipeline:

```text
Git Tag
   |
   ↓
v1.4.0
   |
   ↓
Docker Build
   |
   ↓
myapp:v1.4.0
   |
   ↓
Container Registry
```

This creates a clear relationship between source code and the container image.

---

# Why Avoid Only Using latest?

Using only:

```text
myapp:latest
```

makes it difficult to determine exactly which version is running.

Instead:

```text
myapp:v1.0.0
myapp:v1.1.0
myapp:v1.2.0
```

provides explicit version information.

Versioned images make:

```text
Rollback
Traceability
Auditing
Troubleshooting
```

easier.

---

# Tag and Deployment Traceability

A good DevOps pipeline should provide traceability:

```text
Git Commit
    |
    ↓
Git Tag
    |
    ↓
Build
    |
    ↓
Docker Image
    |
    ↓
Deployment
```

Example:

```text
Commit:
8f3a91d

Git Tag:
v2.1.0

Docker Image:
myapp:v2.1.0

Deployment:
Production
```

Now the team can determine exactly which source code produced the production deployment.

---

# Git Tags and Rollback

Suppose production is running:

```text
v2.0.0
```

A new release is deployed:

```text
v2.1.0
```

After deployment, users report a critical problem.

Previous known-good release:

```text
v2.0.0
```

Release history:

```text
v1.9.0
   |
   ↓
v2.0.0
   |
   ↓
v2.1.0 ← Problem
```

The deployment can target the artifact or image associated with:

```text
v2.0.0
```

This makes rollback easier and more traceable.

---

# Git Tags and GitOps

In a GitOps environment, version information can be represented in deployment manifests.

Example:

```yaml
image:
  repository: myapp
  tag: v1.2.0
```

Typical flow:

```text
Application Repository
        |
        ↓
Build
        |
        ↓
Container Image
        |
        ↓
ECR
        |
        ↓
GitOps Repository
        |
        ↓
Deployment Manifest
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

Git tags can help identify the source release associated with the deployed image.

---

# Git Tags and ECR

Example:

```text
Git Tag:
v1.4.0
```

Docker image:

```text
myapp:v1.4.0
```

ECR:

```text
ECR
 |
 ├── myapp:v1.2.0
 ├── myapp:v1.3.0
 └── myapp:v1.4.0
```

Each version can be identified independently.

---

# Git Tags and Kubernetes

A Kubernetes deployment can reference a versioned image:

```yaml
containers:
  - name: app
    image: myapp:v2.3.0
```

Flow:

```text
Git Tag
   |
   ↓
v2.3.0
   |
   ↓
Docker Image
   |
   ↓
myapp:v2.3.0
   |
   ↓
Kubernetes
```

This makes the deployed application version explicit.

---

# Git Tags and ArgoCD

In a GitOps workflow:

```text
Application Code
      |
      ↓
Build
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
GitOps Repository
      |
      ↓
Image Version
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

Example:

```yaml
image:
  repository: <ecr-repository>
  tag: v2.3.0
```

ArgoCD reconciles the desired state stored in Git.

The release tag provides traceability back to the source release.

---

# Release Naming Convention

Use a consistent naming convention.

Recommended:

```text
v1.0.0
v1.1.0
v1.1.1
v2.0.0
```

Avoid inconsistent names:

```text
release1
final
final-new
prod-latest
version-new
```

Consistent naming makes automation and troubleshooting easier.

---

# Production Tag Immutability

Production release tags should normally be treated as immutable.

Example:

```text
v2.0.0
   |
   ↓
Commit ABC123
```

Do not later move the same tag to:

```text
Commit XYZ789
```

Instead, create a new version:

```text
v2.0.1
```

This preserves release history.

---

# Reusing an Existing Tag

Avoid reusing the same release tag for different code.

Bad practice:

```text
v1.0.0 → Commit A

Later:

v1.0.0 → Commit B
```

Good practice:

```text
v1.0.0 → Commit A
v1.0.1 → Commit B
```

This maintains reliable version history.

---

# Release Artifacts and Tags

A release tag can be associated with build artifacts.

Example:

```text
Git Tag
v1.2.0
   |
   ↓
Build
   |
   ↓
Artifact
   |
   ↓
Docker Image
myapp:v1.2.0
```

This improves traceability.

---

# Tagging Strategy for Microservices

For a microservices platform, each service may have its own version.

Example:

```text
catalogue:v1.4.0
cart:v2.1.2
payment:v1.8.0
orders:v3.0.1
inventory:v1.6.0
```

Repositories can use release tags such as:

```text
catalogue:
v1.4.0

payment:
v1.8.0
```

This allows independent service releases.

---

# Monorepo Tagging

In a monorepo, multiple services may exist in one repository.

Example:

```text
services/
├── catalogue
├── cart
├── payment
├── orders
└── inventory
```

The organization may use:

```text
v1.0.0
v1.1.0
v2.0.0
```

or service-specific conventions depending on the release strategy.

The important requirement is consistency.

---

# Release Branch vs Release Tag

A release branch is used when continued changes are needed for a release line.

Example:

```text
release/v2.0
```

A release tag marks a specific release:

```text
v2.0.0
```

Conceptually:

```text
release/v2.0
       |
       ├── v2.0.0
       ├── v2.0.1
       └── v2.0.2
```

A release branch can continue receiving maintenance fixes.

Tags identify individual release points.

---

# Hotfix and Tags

Suppose:

```text
Production:
v2.0.0
```

A critical bug is discovered.

Create a hotfix branch:

```text
hotfix/v2.0.1
```

After fixing and validating:

```text
v2.0.1
```

Release flow:

```text
v2.0.0
   |
   ↓
Hotfix
   |
   ↓
v2.0.1
```

This preserves the original release and creates a new version.

---

# Git Tag vs Commit SHA

Both can identify a version.

Commit SHA:

```text
8f3a91d
```

Git tag:

```text
v2.1.0
```

The SHA is the unique identifier of the commit.

The tag is a human-readable reference.

Best practice:

```text
Release:
v2.1.0

Commit:
8f3a91d
```

Maintain both for traceability.

---

# Git Tags in DevSecOps

Git tags can become part of a DevSecOps release process.

Example:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ├── Build
    ├── Unit Tests
    ├── SonarQube
    ├── Trivy
    └── Veracode
          |
          ↓
      Code Review
          |
          ↓
        main
          |
          ↓
      Release Tag
          |
          ↓
       v2.0.0
          |
          ↓
        CI/CD
          |
          ↓
    Docker Image
          |
          ↓
         ECR
          |
          ↓
      GitOps
          |
          ↓
       ArgoCD
          |
          ↓
         EKS
          |
          ↓
      Production
```

The release tag identifies the source version that passed the release process.

---

# Release Tag Governance

Production tags should have governance around:

```text
Who can create tags?
Who can push tags?
Who can delete tags?
Who can modify releases?
Which tags trigger deployment?
```

Example:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
Review
    |
    ↓
main
    |
    ↓
Release Approval
    |
    ↓
Tag
    |
    ↓
Production Pipeline
```

---

# Protecting Release Tags

Release tags can be protected using repository policies where supported.

Typical goals:

```text
Prevent Unauthorized Tag Creation
Prevent Tag Deletion
Prevent Tag Modification
Require Release Permissions
```

Production releases should not be casually changed.

---

# Release Process for a DevOps Team

A practical release process can be:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
CI
      |
      ├── Build
      ├── Unit Tests
      ├── SonarQube
      ├── Trivy
      └── Veracode
      |
      ↓
Code Review
      |
      ↓
Merge to main
      |
      ↓
Release Approval
      |
      ↓
Create Tag
      |
      ↓
v2.0.0
      |
      ↓
Build Artifact
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
GitOps Repository
      |
      ↓
ArgoCD
      |
      ↓
EKS
      |
      ↓
Production
```

---

# Release Candidate Flow

For higher-risk releases:

```text
main
 |
 ↓
v2.0.0-rc.1
 |
 ↓
Testing
 |
 ↓
Bug Fix
 |
 ↓
v2.0.0-rc.2
 |
 ↓
Final Validation
 |
 ↓
v2.0.0
 |
 ↓
Production
```

This provides a controlled release process.

---

# Environment Promotion

A release artifact can be promoted across environments.

Example:

```text
Git Tag
v2.5.0
   |
   ↓
Build Once
   |
   ↓
myapp:v2.5.0
   |
   ├── Development
   |
   ├── QA
   |
   ├── Staging
   |
   └── Production
```

The same built artifact can be promoted instead of rebuilding different binaries for each environment.

---

# Release Rollback Strategy

Suppose:

```text
Current:
v2.1.0
```

Previous:

```text
v2.0.0
```

If v2.1.0 fails:

```text
v2.1.0
   |
   ↓
Production Issue
   |
   ↓
Rollback
   |
   ↓
v2.0.0
```

The deployment should normally roll back to the previously validated artifact or image rather than rebuilding old source code unnecessarily.

---

# Rollback and Docker

Example:

```text
Current:

myapp:v2.1.0
```

Previous:

```text
myapp:v2.0.0
```

Rollback:

```text
myapp:v2.1.0
       |
       ↓
     Issue
       |
       ↓
myapp:v2.0.0
```

This is one reason versioned container images are useful.

---

# Rollback and Kubernetes

Suppose Kubernetes is running:

```yaml
image:
  repository: myapp
  tag: v2.1.0
```

A rollback can update the deployment to:

```yaml
image:
  repository: myapp
  tag: v2.0.0
```

In GitOps, this change should normally be made through the Git repository so that Git remains the source of truth.

---

# Rollback and ArgoCD

GitOps flow:

```text
Production
    |
    ↓
v2.1.0
    |
    ↓
Issue
    |
    ↓
GitOps Repository
    |
    ↓
Change image to v2.0.0
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
v2.0.0
```

This provides an auditable rollback.

---

# Release Traceability

A strong DevOps implementation should be able to trace:

```text
JIRA Ticket
    |
    ↓
Pull Request
    |
    ↓
Commit SHA
    |
    ↓
Git Tag
    |
    ↓
Build
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Change
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Production
```

Example:

```text
JIRA:
DEV-1234

PR:
#245

Commit:
8f3a91d

Tag:
v2.1.0

Image:
myapp:v2.1.0

Registry:
ECR

Deployment:
EKS Production
```

This provides strong auditability and traceability.

---

# Tags and Auditability

A production release should allow the team to answer:

```text
Which code was deployed?
Which commit produced the artifact?
Which tag represented the release?
Which Docker image was deployed?
Which environment received it?
When was it deployed?
Who approved the release?
```

A release chain should look like:

```text
PR
 ↓
Commit
 ↓
Tag
 ↓
Build
 ↓
Image
 ↓
Deployment
```

---

# Best Practices

```text
Use Semantic Versioning
Use Annotated Tags for Releases
Use Consistent Tag Names
Keep Production Tags Immutable
Do Not Reuse Release Tags
Push Only Approved Tags
Protect Release Tags
Use Versioned Docker Images
Maintain Commit Traceability
Build Once and Promote
Automate Release Pipelines
Maintain Rollback Versions
Document Release Procedures
```

---

# Common Mistakes

### Creating a Tag but Not Pushing It

```bash
git tag v1.0.0
```

The tag exists locally until it is pushed.

### Using Only latest

```text
myapp:latest
```

This reduces traceability.

Prefer:

```text
myapp:v1.0.0
```

### Reusing the Same Tag

Bad:

```text
v1.0.0 → Commit A

Later:

v1.0.0 → Commit B
```

Good:

```text
v1.0.0 → Commit A
v1.0.1 → Commit B
```

### Deleting Production Tags

Deleting production tags can damage release history and make rollback more difficult.

### Using Inconsistent Naming

Avoid:

```text
final
final-new
production
production-final
release-latest
```

Prefer:

```text
v1.0.0
v1.1.0
v1.1.1
```

### Using Unversioned Production Images

Avoid:

```text
myapp:latest
```

Prefer:

```text
myapp:v2.1.0
```

or another immutable image reference.

### Losing Traceability

Always maintain:

```text
PR
 ↓
Commit
 ↓
Tag
 ↓
Build
 ↓
Image
 ↓
Deployment
```

---

# Production Release Checklist

```text
☐ Code review completed
☐ CI checks passed
☐ Unit tests passed
☐ Integration tests passed
☐ SonarQube passed
☐ Trivy passed
☐ Veracode checks completed
☐ Release approved
☐ Version selected
☐ Git tag created
☐ Tag verified
☐ Tag pushed
☐ Build completed
☐ Docker image created
☐ Image pushed to ECR
☐ Deployment completed
☐ Post-deployment validation completed
☐ Release recorded
☐ Rollback version identified
```

---

# Git Tag Command Reference

Create lightweight tag:

```bash
git tag v1.0.0
```

Create annotated tag:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

Tag a specific commit:

```bash
git tag -a v1.0.0 <commit-sha> -m "Release v1.0.0"
```

List tags:

```bash
git tag
```

Search tags:

```bash
git tag -l "v1.*"
```

Sort tags:

```bash
git tag --sort=-version:refname
```

Show tag:

```bash
git show v1.0.0
```

Push one tag:

```bash
git push origin v1.0.0
```

Push all tags:

```bash
git push origin --tags
```

Fetch tags:

```bash
git fetch --tags
```

Check remote tags:

```bash
git ls-remote --tags origin
```

Delete local tag:

```bash
git tag -d v1.0.0
```

Delete remote tag:

```bash
git push origin --delete v1.0.0
```

Checkout a tag:

```bash
git switch --detach v1.0.0
```

Create branch from tag:

```bash
git switch -c hotfix-v1.0.0 v1.0.0
```

---

# Interview Questions

## Basic

1. What is a Git tag?
2. Why do we use Git tags?
3. What is the difference between a Git tag and a branch?
4. What is a lightweight tag?
5. What is an annotated tag?
6. Which type of tag do you prefer for production releases?
7. How do you create a Git tag?
8. How do you list Git tags?
9. How do you push a tag to a remote repository?
10. How do you delete a local tag?
11. How do you delete a remote tag?
12. How do you fetch tags?
13. What is Semantic Versioning?
14. What does MAJOR.MINOR.PATCH mean?
15. What is a detached HEAD?

---

## Intermediate

16. How do you create an annotated tag?
17. How do you tag a specific commit?
18. How do you push all tags?
19. How do you view tag information?
20. How do you check remote tags?
21. What is the difference between a Git tag and a GitHub Release?
22. How are Git tags used in CI/CD?
23. How would Jenkins trigger a release based on a Git tag?
24. How would GitHub Actions trigger on a tag?
25. How do you use Git tags for Docker image versioning?
26. Why should production Docker images avoid only using `latest`?
27. How do tags help with rollback?
28. How do tags provide deployment traceability?
29. What is the difference between a release branch and a release tag?
30. Can you create a branch from a tag?

---

## Advanced / Production

31. Design a Git tagging strategy for a production DevOps environment.
32. How would you implement Semantic Versioning?
33. How would you prevent production tags from being modified?
34. How would you connect Git tags to Docker image versions?
35. How would you connect Git tags to ECR?
36. How would you connect Git tags to Kubernetes deployments?
37. How would you integrate Git tags with ArgoCD and GitOps?
38. How would you design rollback using release tags?
39. How would you handle a critical production hotfix?
40. How would you manage tags in a microservices environment?
41. How would you handle releases in a monorepo?
42. Why should production tags be immutable?
43. What problems can occur if the same tag is reused?
44. How would you maintain commit SHA → tag → image → deployment traceability?
45. How would you design a production release process using Git tags, Jenkins, SonarQube, Trivy, Docker, ECR, ArgoCD, and EKS?

---

# Real-World DevOps Scenario

Suppose your microservices application currently runs:

```text
Production:
v1.5.0
```

A new feature is completed.

The team creates:

```text
v1.6.0
```

Release flow:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
CI + Security
      |
      ↓
Code Review
      |
      ↓
Merge to main
      |
      ↓
Create Tag
      |
      ↓
v1.6.0
      |
      ↓
Jenkins
      |
      ↓
Build
      |
      ↓
SonarQube
      |
      ↓
Trivy
      |
      ↓
Docker Build
      |
      ↓
myapp:v1.6.0
      |
      ↓
ECR
      |
      ↓
GitOps Repository
      |
      ↓
ArgoCD
      |
      ↓
EKS
      |
      ↓
Production
```

After deployment, users report a critical problem.

The team identifies:

```text
v1.5.0
```

as the last known-good release.

Rollback:

```text
v1.6.0
    |
    ↓
Production Issue
    |
    ↓
Rollback
    |
    ↓
v1.5.0
```

This demonstrates why versioned releases and immutable artifacts are important.

---

# Complete Release Traceability

A mature DevOps organization should be able to trace:

```text
JIRA Ticket
    |
    ↓
Pull Request
    |
    ↓
Commit SHA
    |
    ↓
Git Tag
    |
    ↓
CI/CD Build
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Production
```

Example:

```text
JIRA:
DEV-1234

PR:
#245

Commit:
8f3a91d

Tag:
v2.1.0

Docker Image:
myapp:v2.1.0

Registry:
ECR

Deployment:
EKS Production
```

This provides strong traceability, auditability, and rollback capability.

---

# Complete Production Architecture

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Commit
    |
    ↓
Push
    |
    ↓
Pull Request
    |
    ├──────────────────────────┐
    ↓                          ↓
Automated CI              Code Review
    |                          |
    ├── Build                  |
    ├── Tests                  |
    ├── SonarQube              |
    ├── Trivy                  |
    └── Veracode               |
    |                          |
    └────────────┬─────────────┘
                 ↓
             Approval
                 |
                 ↓
          Merge to main
                 |
                 ↓
          Release Approval
                 |
                 ↓
             Git Tag
                 |
                 ↓
             v2.0.0
                 |
                 ↓
              Jenkins
                 |
                 ↓
              Build
                 |
                 ↓
          Docker Image
                 |
                 ↓
                ECR
                 |
                 ↓
        GitOps Repository
                 |
                 ↓
              ArgoCD
                 |
                 ↓
                EKS
                 |
                 ↓
             Production
                 |
                 ↓
       Post-Deployment Tests
```

---

# Best Practices Summary

```text
☐ Use Semantic Versioning
☐ Prefer Annotated Tags for Releases
☐ Use Consistent Tag Naming
☐ Keep Production Tags Immutable
☐ Never Reuse Release Tags
☐ Protect Release Tags
☐ Use Versioned Docker Images
☐ Avoid latest for Production
☐ Maintain Commit Traceability
☐ Build Once and Promote
☐ Automate Release Pipelines
☐ Maintain Known-Good Rollback Versions
☐ Integrate Tags with CI/CD
☐ Integrate Tags with GitOps
☐ Document Release Procedures
```

---

# Quick Revision

```text
Git Tag
   ↓
Reference to a specific Git object
```

Types:

```text
Lightweight
Annotated
```

Recommended for production:

```text
Annotated Tag
```

Create:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

List:

```bash
git tag
```

Show:

```bash
git show v1.0.0
```

Push:

```bash
git push origin v1.0.0
```

Push all:

```bash
git push origin --tags
```

Fetch:

```bash
git fetch --tags
```

Delete local:

```bash
git tag -d v1.0.0
```

Delete remote:

```bash
git push origin --delete v1.0.0
```

Semantic Versioning:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v2.4.1
```

Meaning:

```text
2 → Breaking changes
4 → Backward-compatible features
1 → Bug/security fixes
```

CI/CD:

```text
Git Tag
   |
   ↓
CI/CD
   |
   ↓
Build
   |
   ↓
Security Scan
   |
   ↓
Docker Image
   |
   ↓
ECR
   |
   ↓
GitOps
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

Core idea:

> Git tags provide stable release points and help DevOps teams maintain versioning, traceability, rollback capability, auditability, and reproducible deployments.