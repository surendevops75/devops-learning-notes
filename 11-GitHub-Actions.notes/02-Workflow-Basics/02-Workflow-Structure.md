# Workflow Structure

A GitHub Actions workflow is a YAML file that defines the complete automation process.

A workflow contains multiple sections, and each section has a specific responsibility.

A well-designed workflow is:

- Easy to understand
- Easy to maintain
- Reusable
- Secure
- Production-ready

Enterprise organizations organize workflows carefully to separate Continuous Integration (CI), Continuous Delivery (CD), Infrastructure, Security, and Release pipelines.

---

# Workflow Directory

Every workflow must be stored inside:

```text
.github/workflows/
```

Example:

```text
Repository

│

├── src/

├── pom.xml

├── Dockerfile

└── .github/

      └── workflows/

            ci.yml

            deploy.yml

            terraform.yml

            security.yml

            release.yml
```

GitHub automatically scans this directory for workflow files.

---

# Complete Workflow Structure

```text
Workflow

├── name

├── on

├── permissions

├── env

├── defaults

├── concurrency

└── jobs

      ├── Build

      ├── Test

      ├── Scan

      └── Deploy
```

Every workflow starts with metadata and ends with one or more jobs.

---

# Workflow Hierarchy

Understanding the hierarchy is essential.

```text
Repository

↓

Workflow

↓

Jobs

↓

Steps

↓

Actions / Shell Commands
```

One repository can have many workflows.

One workflow can have many jobs.

One job can have many steps.

---

# Typical CI Workflow Structure

```text
CI Workflow

↓

Checkout Source Code

↓

Install Dependencies

↓

Compile

↓

Run Unit Tests

↓

Code Quality Scan

↓

Security Scan

↓

Build Docker Image

↓

Push Artifact
```

Notice that **deployment is not part of the CI workflow**.

---

# Typical CD Workflow Structure

```text
Deployment Workflow

↓

Download Artifact

↓

Deploy QA

↓

Smoke Tests

↓

Deploy SIT

↓

Integration Tests

↓

Deploy UAT

↓

Business Approval

↓

Deploy Production

↓

Sanity Tests

↓

Notify Team
```

CI validates the application.

CD releases the application.

---

# Enterprise Workflow Organization

Instead of one large workflow, enterprises usually separate workflows based on responsibility.

```text
Repository

│

├── ci.yml

├── docker-build.yml

├── security-scan.yml

├── terraform.yml

├── deploy-qa.yml

├── deploy-sit.yml

├── deploy-uat.yml

├── deploy-prod.yml

└── release.yml
```

Each workflow performs one specific function.

---

# Why Separate Workflows?

Instead of this:

```text
One Workflow

↓

Build

↓

Test

↓

Scan

↓

Docker

↓

Terraform

↓

Deploy

↓

Rollback

↓

Notifications
```

Use this approach:

```text
CI Workflow

↓

Artifact

────────────

Security Workflow

↓

Scan Report

────────────

Infrastructure Workflow

↓

Terraform

────────────

Deployment Workflow

↓

QA

↓

SIT

↓

UAT

↓

Production
```

Advantages:

- Easier maintenance
- Faster execution
- Better troubleshooting
- Reusable workflows
- Independent deployments

---

# Enterprise Banking Workflow

A banking application may have the following workflow organization.

```text
Push

↓

CI Pipeline

↓

Artifact Repository

────────────

Security Pipeline

↓

Compliance Report

────────────

QA Deployment

↓

Smoke Tests

────────────

SIT Deployment

↓

Integration Tests

────────────

UAT Deployment

↓

Business Approval

────────────

Production Deployment

↓

Monitoring
```

Every workflow is independent but connected through artifacts and approvals.

---

# Workflow Lifecycle

Every workflow follows the same lifecycle.

```text
Workflow Triggered

↓

Runner Allocated

↓

Jobs Created

↓

Steps Executed

↓

Results Generated

↓

Logs Stored

↓

Workflow Completed
```

GitHub manages the workflow lifecycle automatically.

---

# Workflow Dependencies

Large projects often use multiple workflows.

Example:

```text
Developer Push

↓

CI Workflow

↓

Artifact

↓

Deployment Workflow

↓

Production
```

The deployment workflow depends on the successful completion of the CI workflow.

---

# Production Deployment Workflow

This reflects a real enterprise release process.

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Pipeline

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Push Image

↓

Deploy QA

↓

Smoke Tests

↓

Deploy SIT

↓

Regression Tests

↓

Deploy UAT

↓

Business Approval

↓

workflow_dispatch

↓

Production Approval

↓

Deploy Production

↓

Developer Sanity Checks

↓

Application Monitoring

↓

Release Complete
```

Notice how **CI**, **CD**, **manual approvals**, and **post-deployment validation** are clearly separated.

---

# Production Release Workflow (Based on Enterprise Process)

This aligns closely with your deployment process.

```text
Developer

↓

workflow_dispatch

↓

Select Environment

↓

Enter Commit SHA

↓

Enter JIRA Ticket

↓

Validate JIRA

↓

Validate Deployment Window

↓

Validate Previous Checks

↓

Deploy Production

↓

Health Check

↓

Smoke Test

↓

Developer Verification

↓

Close Change Request
```

This workflow is commonly used in regulated industries such as banking, healthcare, and finance.

---

# Large Enterprise Repository Example

```text
.github/

└── workflows/

      ci.yml

      build-image.yml

      dependency-scan.yml

      code-quality.yml

      terraform-plan.yml

      terraform-apply.yml

      deploy-dev.yml

      deploy-qa.yml

      deploy-sit.yml

      deploy-uat.yml

      deploy-prod.yml

      rollback.yml

      release.yml

      notifications.yml
```

Keeping workflows small makes them easier to maintain.

---

# Best Practices

- Store all workflows inside `.github/workflows/`.
- Separate CI and CD.
- One workflow should have one primary responsibility.
- Reuse workflows whenever possible.
- Protect production workflows with approvals.
- Use descriptive workflow names.
- Keep deployment logic separate from build logic.
- Document workflow dependencies.

---

# Common Mistakes

- One workflow for the entire application lifecycle.
- Mixing infrastructure provisioning with application builds.
- Deploying directly after every commit.
- Hardcoding deployment environments.
- No approval process for production.
- No separation between testing and deployment.

---

# Summary

A workflow defines the automation process in GitHub Actions.

Enterprise teams organize workflows by responsibility instead of creating one large pipeline.

Typical workflow categories include:

- CI
- Security
- Infrastructure
- Deployment
- Release
- Rollback
- Notifications

This approach improves scalability, maintainability, and operational reliability.

---

# Interview Questions

## Basic

1. Where are workflow files stored?
2. What is a workflow?
3. Can one repository have multiple workflows?
4. Why do we separate CI and CD?
5. What is the workflow lifecycle?

---

## Intermediate

1. Explain the hierarchy of a GitHub Actions workflow.
2. Why do enterprises create multiple workflows?
3. What are workflow dependencies?
4. Explain the difference between CI workflows and deployment workflows.
5. How are workflows organized in enterprise repositories?

---

## Advanced

1. Design the workflow structure for a banking application with CI, security scanning, Terraform, QA, SIT, UAT, and production deployment.
2. Explain how multiple workflows communicate using artifacts and approvals.
3. A company has a single monolithic GitHub Actions workflow. Describe how you would split it into modular, reusable enterprise workflows while maintaining deployment reliability.