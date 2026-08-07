# CI/CD Basics

Before understanding GitHub Actions, it's important to understand the concepts of Continuous Integration (CI) and Continuous Deployment/Delivery (CD).

GitHub Actions is a CI/CD platform that automates software development workflows from code commit to production deployment.

---

# Traditional Software Delivery

Before CI/CD, software releases were mostly manual.

```text
Developer

↓

Write Code

↓

Share Code

↓

Manual Build

↓

Manual Testing

↓

Manual Deployment

↓

Production
```

## Problems

- Slow releases
- Manual errors
- Inconsistent deployments
- Difficult rollback
- Long feedback cycle
- High operational effort

As the team size grows, managing releases manually becomes increasingly difficult.

---

# What is Continuous Integration (CI)?

Continuous Integration (CI) is the practice of frequently merging code changes into a shared repository where every change is automatically verified.

Whenever a developer pushes code, an automated pipeline performs tasks such as:

- Build the application
- Run unit tests
- Execute code quality checks
- Perform security scans
- Generate reports

The goal is to identify issues early before they reach production.

---

# Continuous Integration Workflow

```text
Developer

↓

Commit Code

↓

Push to GitHub

↓

GitHub Actions Triggered

↓

Checkout Code

↓

Build

↓

Unit Tests

↓

Code Quality Scan

↓

Security Scan

↓

Package Artifact

↓

CI Success
```

---

# Benefits of Continuous Integration

- Early bug detection
- Faster feedback
- Consistent builds
- Automated testing
- Improved code quality
- Reduced integration problems
- Faster software delivery

---

# What is Continuous Delivery (CD)?

Continuous Delivery extends Continuous Integration by preparing every successful build for deployment.

The application is automatically built, tested, packaged, and deployed to a staging or testing environment.

The final production deployment usually requires manual approval.

---

# Continuous Delivery Workflow

```text
Developer

↓

CI Pipeline

↓

QA Deployment

↓

Integration Testing

↓

User Acceptance Testing

↓

Approval

↓

Production Deployment
```

Production deployment is controlled through an approval process.

---

# What is Continuous Deployment?

Continuous Deployment goes one step further.

Every successful pipeline automatically deploys to production without manual approval.

```text
Developer

↓

Commit

↓

CI

↓

Tests

↓

Security Scan

↓

Deploy Production

↓

Users
```

Only organizations with strong testing and monitoring practices usually adopt Continuous Deployment.

---

# Continuous Delivery vs Continuous Deployment

| Continuous Delivery | Continuous Deployment |
|---------------------|-----------------------|
| Manual production approval | Automatic production deployment |
| Human approval required | No manual approval |
| Lower deployment risk | Faster release cycle |
| Common in enterprises | Common in SaaS companies |

---

# CI/CD Pipeline

A CI/CD pipeline is a sequence of automated stages executed whenever code changes.

Typical stages include:

- Source Checkout
- Build
- Unit Testing
- Static Code Analysis
- Security Scanning
- Artifact Creation
- Docker Build
- Image Push
- Deployment
- Smoke Testing
- Notifications

---

# Typical Enterprise CI Pipeline

```text
Developer

↓

Push Code

↓

Checkout

↓

Build

↓

Unit Tests

↓

SonarQube Scan

↓

Trivy Scan

↓

Build Docker Image

↓

Push to Container Registry

↓

Publish Artifact
```

This pipeline validates the application before deployment begins.

---

# Typical Enterprise CD Pipeline

```text
Artifact

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

End-to-End Tests

↓

Approval

↓

Deploy Production

↓

Post Deployment Validation
```

Every environment acts as a quality gate before production.

---

# Production Deployment Workflow

A typical enterprise deployment process.

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge to Main

↓

CI Pipeline

↓

QA

↓

SIT

↓

UAT

↓

Approval

↓

Production
```

Production deployment is usually protected by multiple approval gates.

---

# CI/CD in GitHub Actions

GitHub Actions automates the complete software delivery lifecycle.

```text
GitHub Event

↓

Workflow Triggered

↓

Runner

↓

Build

↓

Test

↓

Security Scan

↓

Package

↓

Deploy

↓

Notification
```

Everything is defined using YAML workflow files stored inside the repository.

---

# Enterprise Workflow Example

A banking application deployment process.

```text
Developer Push

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Dependency Scan

↓

Docker Build

↓

Push to Registry

↓

Deploy QA

↓

QA Approval

↓

Deploy SIT

↓

Integration Tests

↓

Deploy UAT

↓

Business Approval

↓

Production Deployment

↓

Smoke Tests

↓

Monitoring
```

Notice that production deployment is **not** triggered immediately after a successful build. Multiple validation stages ensure application quality and compliance.

---

# Best Practices

- Commit small and frequent changes.
- Keep pipelines fast.
- Automate testing.
- Fail fast when errors occur.
- Integrate security scanning into CI.
- Use approval gates before production.
- Monitor deployments after release.
- Automate rollback where possible.

---

# Common Mistakes

- Running long builds for every commit.
- Skipping automated tests.
- Deploying directly to production.
- Ignoring failed security scans.
- Mixing build and deployment logic.
- No rollback strategy.
- No deployment approvals for production.
- Lack of monitoring after deployment.

---

# Summary

CI/CD enables teams to automate software delivery from code commit to deployment.

Continuous Integration focuses on validating code changes, while Continuous Delivery and Continuous Deployment automate application releases.

GitHub Actions provides a flexible platform to implement enterprise-grade CI/CD pipelines using workflows, jobs, runners, and reusable actions.

---

# Interview Questions

## Basic

1. What is CI?
2. What is CD?
3. Difference between Continuous Delivery and Continuous Deployment.
4. Why do we need CI/CD?
5. What are the stages of a CI/CD pipeline?

---

## Intermediate

1. Explain a typical enterprise CI pipeline.
2. Why should security scanning be part of CI?
3. What quality gates exist before production deployment?
4. How does GitHub Actions implement CI/CD?
5. What is the difference between a build pipeline and a deployment pipeline?

---

## Advanced

1. Design a complete CI/CD pipeline for a microservices application deployed on Kubernetes using GitHub Actions.
2. Explain how approval gates, automated testing, security scanning, and rollback strategies improve production reliability.
3. A financial organization requires multiple approval stages before deployment. Design a GitHub Actions workflow that supports QA, SIT, UAT, production approvals, and post-deployment validation.