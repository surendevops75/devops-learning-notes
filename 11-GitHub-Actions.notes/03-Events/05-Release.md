# Release Event

The `release` event triggers a GitHub Actions workflow when activity occurs around a GitHub Release.

GitHub Releases are commonly used to publish a specific version of an application or software package.

Release workflows are useful for:

- Building release artifacts
- Publishing packages
- Building Docker images
- Creating release documentation
- Generating release notes
- Deploying a specific application version
- Publishing artifacts to repositories
- Triggering downstream release processes

---

# What is a GitHub Release?

A GitHub Release represents a published version of software associated with a Git tag.

Example:

```text
Source Code

↓

Commit

↓

Tag v1.5.0

↓

GitHub Release

↓

Release Workflow

↓

Build Artifact

↓

Publish
```

A release provides a clear version reference for the software being delivered.

---

# Release Event Syntax

Basic syntax:

```yaml
name: Release Workflow

on:
  release:
    types:
      - published

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build Release
        run: echo "Building release"
```

The workflow runs when the configured release activity occurs.

---

# Release Activity Types

The `release` event supports different activity types.

Common types include:

- `published`
- `created`
- `edited`
- `deleted`
- `prereleased`
- `released`

For production release automation, `published` is commonly useful because it indicates that the release has been published.

---

# Release Lifecycle

```text
Developer

↓

Merge Code

↓

Create Version Tag

↓

Create GitHub Release

↓

Release Event

↓

GitHub Actions

↓

Build

↓

Test

↓

Package

↓

Publish
```

---

# Release vs Tag

A Git tag identifies a specific commit.

Example:

```text
v1.0.0

↓

Commit SHA
```

A GitHub Release adds release metadata around a version.

Conceptually:

```text
Commit

↓

Tag

↓

GitHub Release

↓

Release Automation
```

A release workflow can use the release information to identify the version being published.

---

# Release Workflow

A simple release workflow:

```yaml
name: Release Build

on:
  release:
    types:
      - published

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build Application
        run: mvn clean package

      - name: Run Tests
        run: mvn test

      - name: Publish Artifact
        run: echo "Publishing release artifact"
```

Execution:

```text
GitHub Release

↓

Workflow Trigger

↓

Checkout

↓

Build

↓

Test

↓

Publish
```

---

# Release Versioning

A common enterprise versioning approach is:

```text
v1.0.0
v1.1.0
v1.2.0
v2.0.0
```

The version identifies the software release.

Example:

```text
v1.5.0

↓

Docker Image

↓

catalogue:v1.5.0

↓

Artifact

↓

Deployment
```

The exact versioning strategy depends on the organization's release process.

---

# Production Release Workflow

A production release can follow this pattern:

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge to main

↓

CI

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Docker Image

↓

Push Image

↓

Create Version Tag

↓

Create GitHub Release

↓

Release Workflow

↓

Publish Artifact

↓

UAT

↓

E2E Tests

↓

Change Request

↓

Production Approval

↓

Production Deployment
```

The release event creates a clear boundary between application validation and versioned release management.

---

# Release Artifact Strategy

A release workflow can publish:

- JAR files
- ZIP files
- Docker images
- Helm packages
- Terraform modules
- Application binaries

Example:

```text
Release v1.5.0

├── Application Artifact
├── Docker Image
├── Helm Package
└── Release Notes
```

---

# Docker Release Workflow

A common enterprise pattern is to build a Docker image from the release version.

```text
GitHub Release

↓

Release v1.5.0

↓

Docker Build

↓

Image

↓

catalogue:v1.5.0

↓

Push to ECR

↓

Deployment
```

Example:

```yaml
name: Release Docker Image

on:
  release:
    types:
      - published

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Docker Image
        run: docker build -t catalogue:${{ github.event.release.tag_name }} .

      - name: Push Image
        run: echo "Push image to container registry"
```

The release tag can be used as the image version.

---

# Release Tag

The release tag can be accessed from the release event context.

Example:

```yaml
${{ github.event.release.tag_name }}
```

For a release named:

```text
v1.5.0
```

the value is:

```text
v1.5.0
```

This can be used for:

- Docker image tags
- Artifact versions
- Release notifications
- Deployment metadata

---

# Enterprise Release Architecture

```text
Developer

↓

Pull Request

↓

PR Validation

↓

Merge to main

↓

CI

↓

Build

↓

Security Scan

↓

Artifact

↓

Release Tag

↓

GitHub Release

↓

Release Workflow

↓

Publish Versioned Artifact

↓

UAT

↓

Approval

↓

Production
```

---

# Release and Commit SHA

A production release should maintain traceability between the release version and the source commit.

```text
Release v1.5.0

↓

Git Tag

↓

Commit SHA

↓

Build

↓

Artifact

↓

Docker Image

↓

Production
```

This allows the team to determine exactly which source version produced the production artifact.

---

# Immutable Release Strategy

Production systems should avoid rebuilding the application independently for every environment.

Recommended:

```text
Source Commit

↓

Build Once

↓

Artifact

↓

QA

↓

SIT

↓

UAT

↓

Production
```

Instead of:

```text
QA Build

↓

SIT Build

↓

UAT Build

↓

Production Build
```

Building once and promoting the same artifact improves consistency and traceability.

---

# Release Workflow with Manual Promotion

The release workflow can create the artifact while production deployment remains controlled.

```text
GitHub Release

↓

Build

↓

Test

↓

Security Scan

↓

Publish Artifact

↓

QA

↓

SIT

↓

UAT

↓

workflow_dispatch

↓

JIRA Validation

↓

Deployment Window

↓

Approval

↓

Production
```

This prevents creating a release from automatically bypassing production controls.

---

# Enterprise Production Release Process

Based on your existing deployment process:

```text
Release v1.5.0

↓

Identify Commit SHA

↓

Verify CI Status

↓

Verify Scan Results

↓

Verify Testing Results

↓

Create / Validate JIRA Change Request

↓

Check Approvals

↓

Check Deployment Window

↓

Production Approval

↓

Deploy Release v1.5.0

↓

Smoke Tests

↓

Developer Sanity Checks

↓

Monitoring
```

The release version, commit SHA, test results, security results, and change request should remain traceable.

---

# Release Rollback

A release version can also help identify the version to roll back.

Example:

```text
Production

↓

v1.5.0

↓

Incident

↓

Known Good Version

↓

v1.4.9

↓

Rollback

↓

Health Check
```

With Kubernetes and Helm:

```text
Production

↓

Helm Release

↓

Rollback to Known Good Revision

↓

Health Check

↓

Monitoring
```

Your existing production approach uses Helm automatic rollbacks, while a release-based process can also identify the exact version that should be restored.

---

# Pre-Release Workflow

Organizations may use pre-release versions for testing.

Example:

```text
v2.0.0-rc.1

↓

Release Candidate

↓

QA

↓

SIT

↓

UAT

↓

Production Release
```

This allows teams to validate a candidate before declaring the final release.

---

# Production Release Controls

A production release workflow should validate:

```text
Release Version

↓

Commit SHA

↓

CI Status

↓

Security Scan

↓

Testing Results

↓

JIRA Change Request

↓

Approvals

↓

Deployment Window

↓

Rollback Plan
```

Only after the required controls pass should production deployment proceed.

---

# Release Workflow with Environment

A release workflow can deploy to a protected environment.

Conceptually:

```text
Release

↓

Production Environment

↓

Required Reviewers

↓

Approval

↓

Deployment
```

This provides an additional control layer around production releases.

---

# Release Notifications

After a successful release, organizations may notify:

- Development Team
- DevOps Team
- QA Team
- Release Management
- Business Stakeholders

Example:

```text
Release v1.5.0

↓

Production Deployment Successful

↓

Notification

↓

Release Complete
```

---

# Production Troubleshooting

## Scenario 1 - Release Workflow Did Not Run

Check:

```text
1. Release event is configured.
2. Correct release activity type is configured.
3. Workflow exists in .github/workflows/.
4. Workflow syntax is valid.
5. Workflow is enabled.
```

---

## Scenario 2 - Wrong Version Was Built

Check:

```text
GitHub Release

↓

Release Tag

↓

Commit SHA

↓

Checkout Version

↓

Build Artifact
```

Verify that the workflow is building the intended release version.

---

## Scenario 3 - Docker Image Has Wrong Tag

Check:

```text
Release Tag

↓

github.event.release.tag_name

↓

Docker Image Tag
```

Example:

```yaml
docker build \
  -t catalogue:${{ github.event.release.tag_name }} .
```

Make sure the release tag is being used consistently.

---

## Scenario 4 - Release Created but Production Was Deployed Automatically

Review:

```text
Release Workflow

↓

Deployment Job

↓

Environment

↓

Approval Rules
```

A release event should not automatically bypass production change-management controls.

---

## Scenario 5 - Production Artifact Cannot Be Traced

Verify:

```text
Production Image

↓

Image Tag / Digest

↓

Release Version

↓

Commit SHA

↓

GitHub Release

↓

Source Code
```

If any link is missing, improve release metadata and artifact traceability.

---

# Best Practices

- Use semantic versioning or an established enterprise versioning strategy.
- Use immutable release artifacts.
- Build once and promote the same artifact.
- Associate releases with specific commit SHAs.
- Use release tags consistently.
- Protect production environments.
- Require approvals for production releases.
- Maintain a rollback plan.
- Keep release and deployment responsibilities clearly defined.
- Record release metadata for auditing.

---

# Common Mistakes

- Rebuilding the application separately for every environment.
- Using mutable release identifiers.
- Losing the relationship between release, commit, and artifact.
- Automatically deploying every release to production without approval.
- Not validating security and testing results.
- Not maintaining a rollback strategy.
- Using inconsistent Docker image tags.

---

# Summary

The `release` event allows GitHub Actions to automate activities associated with GitHub Releases.

A release can represent a versioned, traceable software package that moves through:

```text
Code

↓

Commit

↓

Tag

↓

Release

↓

Artifact

↓

QA

↓

SIT

↓

UAT

↓

Production
```

In enterprise environments, release automation should preserve traceability between the release version, commit SHA, artifact, security results, testing results, change request, and production deployment.

A strong production pattern is:

```text
GitHub Release

↓

Validate Version

↓

Validate Commit

↓

Build / Publish Artifact

↓

Security Results

↓

Testing Results

↓

JIRA Change Request

↓

Approval

↓

Deployment Window

↓

Production Deployment

↓

Smoke Test

↓

Monitoring
```

---

# Interview Questions

## Basic

1. What is the `release` event?
2. What is a GitHub Release?
3. What is the difference between a Git tag and a GitHub Release?
4. What release activity types are commonly used?
5. How do you access the release tag name?

## Intermediate

6. How can GitHub Actions build a Docker image from a GitHub Release?
7. Why is release versioning important?
8. How do you maintain traceability between a release and a commit?
9. Why should production artifacts be immutable?
10. Why should the same artifact be promoted across environments?

## Advanced

11. Design an enterprise release workflow that builds a versioned Docker image and promotes it through QA, SIT, UAT, and Production.
12. Design a production release process that validates the release tag, commit SHA, CI results, security scan results, testing results, JIRA change request, approvals, and deployment window.
13. A GitHub Release was created successfully, but the production deployment used the wrong application version. Explain how you would trace the release, tag, commit SHA, artifact, Docker image, and deployment to identify the failure.
14. Design a rollback strategy for a production release where the current version is unhealthy and a previous known-good release must be restored safely.