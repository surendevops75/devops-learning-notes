# Jenkins Multibranch Pipeline

## Introduction

In modern software development, multiple developers work on different features simultaneously. Instead of committing directly to the `main` branch, each developer creates a feature branch, implements changes, raises a Pull Request (PR), and finally merges the code after validation.

Managing a separate Jenkins job for every branch quickly becomes difficult as the number of branches increases.

A **Jenkins Multibranch Pipeline** automatically discovers Git branches, creates pipelines for them, executes the appropriate `Jenkinsfile`, and removes pipelines when branches are deleted.

This allows teams to build, test, and validate every branch independently without creating manual Jenkins jobs.

---

# Why Do We Need a Multibranch Pipeline?

Imagine an application with multiple developers.

```text
Developer A

↓

feature-login

----------------------

Developer B

↓

feature-cart

----------------------

Developer C

↓

feature-payment

----------------------

Developer D

↓

bugfix-order
```

Without a Multibranch Pipeline, every branch requires a separate Jenkins job.

```text
feature-login

↓

Jenkins Job 1

-------------------

feature-cart

↓

Jenkins Job 2

-------------------

feature-payment

↓

Jenkins Job 3

-------------------

bugfix-order

↓

Jenkins Job 4
```

As the number of branches grows, maintaining these jobs becomes impractical.

Instead, a single Multibranch Pipeline manages all branches automatically.

```text
GitHub Repository

↓

Jenkins Multibranch Pipeline

↓

Discovers Every Branch

↓

Creates Pipeline Automatically

↓

Runs Jenkinsfile
```

---

# Real Enterprise Scenario

Consider an e-commerce platform.

Several teams are working simultaneously.

```text
Inventory Team

↓

feature-stock

----------------------

Payment Team

↓

feature-payment

----------------------

User Team

↓

feature-profile

----------------------

Cart Team

↓

feature-cart
```

Every branch should be:

- Built
- Tested
- Security scanned
- Dockerized

Only after approval should it be merged into the main branch.

A Multibranch Pipeline automates this process.

---

# Single Pipeline vs Multibranch Pipeline

| Feature | Single Pipeline | Multibranch Pipeline |
|----------|----------------|----------------------|
| Supports Multiple Branches | ❌ | ✅ |
| Automatic Branch Discovery | ❌ | ✅ |
| Automatic Job Creation | ❌ | ✅ |
| Pull Request Builds | Limited | ✅ |
| Automatic Branch Cleanup | ❌ | ✅ |
| Enterprise Ready | Small Projects | Yes |

---

# Multibranch Pipeline Architecture

```text
                    Developers
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
 feature-login     feature-cart     feature-payment
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
                  GitHub Repository
                         │
                         ▼
                GitHub Webhook
                         │
                         ▼
           Jenkins Multibranch Pipeline
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
 Pipeline A        Pipeline B        Pipeline C
```

---

# How a Multibranch Pipeline Works

```text
Developer Push

↓

GitHub

↓

Webhook Trigger

↓

Jenkins Scans Repository

↓

New Branch Found

↓

Pipeline Created

↓

Checkout Code

↓

Execute Jenkinsfile

↓

Publish Results
```

---

# What Happens Internally?

Whenever Jenkins receives a webhook notification:

```text
Webhook

↓

Repository Scan

↓

Read Branch List

↓

Compare Existing Jobs

↓

Create Missing Pipelines

↓

Delete Removed Branch Pipelines

↓

Run Jenkinsfile
```

This process is called **Branch Indexing**.

---

# Branch Discovery

Branch Discovery is the process of identifying newly created Git branches.

Example:

```text
main

↓

Developer Creates

feature-coupon

↓

Push

↓

Jenkins Discovers Branch

↓

Creates New Pipeline
```

No manual intervention is required.

---

# Branch Indexing

Branch Indexing keeps Jenkins synchronized with the Git repository.

During indexing Jenkins:

- Detects new branches
- Detects deleted branches
- Detects Pull Requests
- Updates existing pipelines

```text
Git Repository

↓

Index Repository

↓

Update Pipeline List

↓

Execute Changed Pipelines
```

---

# Automatic Pipeline Creation

Each branch gets its own independent pipeline.

```text
feature-login

↓

Pipeline

---------------------

feature-cart

↓

Pipeline

---------------------

feature-payment

↓

Pipeline
```

Each pipeline has its own:

- Console Logs
- Build History
- Test Reports
- Artifacts
- Status

---

# Branch-Specific Jenkinsfiles

In a Multibranch Pipeline, every Git branch contains its own `Jenkinsfile`.

When Jenkins discovers a branch, it automatically checks out that branch and executes the Jenkinsfile present in it.

This allows different branches to execute different pipeline logic if required.

---

## Example

```text
GitHub Repository

├── main
│   └── Jenkinsfile
│
├── develop
│   └── Jenkinsfile
│
├── release-v1
│   └── Jenkinsfile
│
└── feature-login
    └── Jenkinsfile
```

Each branch can have different deployment behavior.

Example:

```text
main

↓

Deploy to Production

------------------------

develop

↓

Deploy to Development

------------------------

release

↓

Deploy to QA

------------------------

feature branches

↓

Build + Test Only
```

---

# Pull Request Builds

One of the biggest advantages of a Multibranch Pipeline is automatic Pull Request validation.

Whenever a developer raises a Pull Request, Jenkins automatically executes the pipeline before allowing the merge.

---

## Production Flow

```text
Developer

↓

Create Feature Branch

↓

Code Changes

↓

Push to GitHub

↓

Create Pull Request

↓

Jenkins Triggered

↓

Checkout PR

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Publish Status

↓

Reviewer Approves

↓

Merge
```

---

## Why Validate Pull Requests?

Pull Request validation prevents broken code from reaching the main branch.

It helps detect:

- Compilation errors
- Failed unit tests
- Code quality issues
- Vulnerable dependencies
- Container vulnerabilities

before the code is merged.

---

# Build Strategy for Different Branches

Not every branch should execute the same pipeline.

Different branches usually have different responsibilities.

---

## Feature Branch

```text
Checkout

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy

↓

Stop
```

No deployment.

---

## Develop Branch

```text
Checkout

↓

Build

↓

Security Scan

↓

Docker Build

↓

Push Image

↓

Deploy to Development Environment
```

---

## Release Branch

```text
Checkout

↓

Build

↓

Security Scan

↓

Integration Testing

↓

Deploy to QA
```

---

## Main Branch

```text
Checkout

↓

Build

↓

Security Scan

↓

Docker Build

↓

Push Image

↓

Deploy to Production
```

---

# GitHub Webhook vs Repository Scan

There are two ways Jenkins detects branch changes.

---

## GitHub Webhook

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline Starts
```

### Advantages

- Immediate trigger
- Faster builds
- Less Jenkins workload
- Recommended for production

---

## Repository Scan

```text
Jenkins

↓

Every 5 Minutes

↓

Scan Repository

↓

Detect Changes

↓

Run Pipeline
```

### Advantages

- Works without webhooks
- Simple setup

### Disadvantages

- Delayed builds
- Unnecessary repository scans

---

# Enterprise Workflow Using Shared Library

Instead of maintaining pipeline logic inside every repository,

each branch simply calls the Enterprise Shared Library.

---

## Feature Branch

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                javaEKSPipeline()

            }

        }

    }

}
```

---

## Main Branch

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                javaEKSPipeline()

            }

        }

        stage('Deploy') {

            steps {

                EKSDeploy()

            }

        }

    }

}
```

---

# Complete Enterprise Workflow

```text
Developer

↓

Create Feature Branch

↓

Git Push

↓

GitHub

↓

Webhook

↓

Jenkins Multibranch Pipeline

↓

Automatically Discovers Branch

↓

Creates Pipeline

↓

Load Shared Library

↓

javaEKSPipeline()

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Pipeline Successful

↓

Pull Request Created

↓

Pipeline Executes Again

↓

Reviewer Approval

↓

Merge to Main

↓

Jenkins Starts Main Pipeline

↓

javaEKSPipeline()

↓

Push Docker Image to Amazon ECR

↓

EKSDeploy()

↓

Deploy to Amazon EKS

↓

Rollout Verification

↓

Application Live
```

---

# Complete Enterprise Jenkinsfile

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    triggers {

        githubPush()

    }

    stages {

        stage('Build & Security') {

            steps {

                javaEKSPipeline()

            }

        }

        stage('Deploy to Production') {

            when {

                branch 'main'

            }

            steps {

                EKSDeploy()

            }

        }

    }

    post {

        success {

            echo "Pipeline completed successfully."

        }

        failure {

            echo "Pipeline failed."

        }

        always {

            cleanWs()

        }

    }

}
```

---

# Why This Is Enterprise Ready

```text
Feature Branches

↓

Automatically Discovered

↓

Independent Pipelines

↓

Shared Library

↓

Standard CI/CD

↓

Pull Request Validation

↓

Merge

↓

Production Deployment
```

Benefits:

- No duplicate Jenkins jobs
- Standardized pipelines
- Automatic branch management
- Centralized CI/CD logic
- Easy maintenance
- Faster onboarding
- Scales to hundreds of repositories

---

# Best Practices

## 1. Use One Multibranch Pipeline Per Repository

Create one Multibranch Pipeline for each Git repository instead of creating separate Jenkins jobs for every branch.

```text
GitHub Repository

↓

One Multibranch Pipeline

↓

main

↓

develop

↓

feature/*

↓

release/*
```

This simplifies management and scales well as the number of branches grows.

---

## 2. Keep Jenkinsfiles Small

Move reusable CI/CD logic into a Jenkins Shared Library.

Example

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                javaEKSPipeline()

            }

        }

    }

}
```

The Jenkinsfile should only orchestrate the pipeline, not implement all the logic.

---

## 3. Use GitHub Webhooks Instead of Periodic Scanning

Prefer GitHub Webhooks over repository polling.

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins
```

Advantages:

- Immediate builds
- Lower Jenkins load
- Faster feedback

---

## 4. Deploy Only from Protected Branches

Restrict deployments to trusted branches such as `main` or `release`.

```groovy
when {

    branch 'main'

}
```

Feature branches should only perform validation and testing.

---

## 5. Standardize Every Branch Pipeline

Every branch should execute the same quality checks.

```text
Checkout

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan
```

This ensures consistent quality across all branches.

---

## 6. Keep Feature Branches Lightweight

Avoid deploying feature branches to production.

Recommended workflow:

```text
Feature Branch

↓

Build

↓

Test

↓

Security Scan

↓

Pipeline Ends
```

---

## 7. Enable Pull Request Validation

Always validate Pull Requests before merging.

```text
Pull Request

↓

Pipeline

↓

Build

↓

Tests

↓

Security

↓

Merge
```

Broken code should never reach the main branch.

---

## 8. Clean Up Deleted Branches

Enable automatic removal of pipelines for deleted branches.

```text
Branch Deleted

↓

Pipeline Removed
```

This keeps Jenkins clean and avoids unused jobs.

---

## 9. Version Your Shared Library

Don't always use the latest version.

Example

```groovy
@Library('jenkins-shared-library@v1.2.0') _
```

Versioning makes pipeline behavior predictable.

---

## 10. Secure Your Pipeline

Never store secrets inside the Jenkinsfile.

Use:

- Jenkins Credentials
- IAM Roles
- Secret Managers

instead of hardcoding passwords or tokens.

---

# Troubleshooting

## 1. Branch Not Discovered

### Symptoms

```text
New branch exists in GitHub

↓

No Jenkins Pipeline
```

### Possible Causes

- Repository scan not executed
- Webhook failure
- Branch discovery disabled
- SCM configuration issue

### Resolution

```text
Run

↓

Scan Repository Now

↓

Check Webhook

↓

Verify Branch Source

↓

Review Jenkins Logs
```

---

## 2. Jenkinsfile Not Found

### Symptoms

```text
No Jenkinsfile Found
```

### Verify

- Jenkinsfile exists
- Correct filename
- Present in the branch
- Repository checkout successful

---

## 3. Pull Request Pipeline Not Running

### Verify

```text
GitHub Webhook

↓

Branch Source Plugin

↓

PR Discovery Enabled

↓

Repository Permissions
```

---

## 4. Webhook Not Triggering

### Verify

```text
GitHub Webhook

↓

Payload URL

↓

Secret

↓

HTTP 200 Response

↓

Jenkins Reachability
```

---

## 5. Feature Branch Deploys to Production

### Verify

```groovy
when {

    branch 'main'

}
```

Production deployment must always be restricted.

---

## 6. Shared Library Not Loading

### Verify

```text
Library Name

↓

Credentials

↓

Repository URL

↓

Branch

↓

Version
```

---

## 7. SonarQube Stage Fails

### Verify

```text
Sonar Server

↓

Authentication Token

↓

Credentials ID

↓

Server Reachability
```

---

## 8. Docker Build Fails

### Verify

```text
Docker Installed

↓

Docker Daemon

↓

Dockerfile

↓

Disk Space
```

---

## 9. Amazon EKS Deployment Fails

### Verify

```text
AWS Credentials

↓

kubectl Access

↓

Cluster Connectivity

↓

Deployment YAML

↓

Namespace
```

---

## 10. Pipeline Created but Build Never Starts

### Verify

- Jenkins agent availability
- Executor availability
- Pipeline paused
- SCM checkout errors
- Build queue status

---

# Common Interview Questions

## 1. Why do companies prefer Multibranch Pipelines over normal Jenkins Pipelines?

### Production-Level Answer

In enterprise environments, developers work on multiple feature, release, and hotfix branches simultaneously. A Multibranch Pipeline automatically discovers new branches, creates pipelines, runs branch-specific Jenkinsfiles, and removes pipelines when branches are deleted. This eliminates manual job creation, reduces maintenance, and scales efficiently for repositories with frequent branch creation.

### Follow-Up Questions

- How does Jenkins discover new branches?
- What is Branch Indexing?
- How are deleted branches handled?

---

## 2. Explain the complete workflow of a Jenkins Multibranch Pipeline.

### Production-Level Answer

A developer pushes code to a Git branch, triggering a GitHub webhook. Jenkins scans the repository, discovers the branch, creates or updates the pipeline, checks out the source code, loads the Shared Library, executes build, testing, SonarQube analysis, OWASP Dependency Check, Docker image creation, Trivy scanning, and publishes the results. If the branch is `main`, the deployment stage pushes the image to Amazon ECR and deploys it to Amazon EKS.

### Follow-Up Questions

- Where does Jenkins obtain the Jenkinsfile?
- How does deployment differ for feature branches?
- Why use Shared Libraries with Multibranch Pipelines?

---

## 3. What is Branch Indexing?

### Production-Level Answer

Branch Indexing is the process by which Jenkins synchronizes with the Git repository. It detects newly created branches, deleted branches, updated branches, and Pull Requests, then creates, updates, or removes corresponding pipelines automatically. This keeps Jenkins aligned with the repository without manual intervention.

### Follow-Up Questions

- When does Branch Indexing occur?
- Can it be triggered manually?
- What happens if indexing fails?

---

## 4. How would you prevent feature branches from deploying to production?

### Production-Level Answer

Production deployments should be restricted using branch conditions in the Jenkinsfile. Feature branches should execute only validation stages such as build, testing, SonarQube analysis, OWASP Dependency Check, Docker image creation, and Trivy scanning. Only trusted branches like `main` or `release` should be allowed to deploy.

### Follow-Up Questions

- Would you deploy feature branches anywhere?
- How do you secure production deployments?
- How do protected Git branches help?

---

## 5. Why combine Shared Libraries with Multibranch Pipelines?

### Production-Level Answer

A Multibranch Pipeline manages branch discovery and pipeline creation, while a Shared Library centralizes CI/CD logic. Together they provide automated branch management with standardized build, security, and deployment stages. This minimizes duplicate Jenkinsfiles and allows platform engineers to update pipeline behavior for all projects from a single repository.

### Follow-Up Questions

- What belongs in the Jenkinsfile?
- What belongs in the Shared Library?
- How do you version Shared Libraries?

---

## 6. How do you troubleshoot a branch that Jenkins doesn't discover?

### Production-Level Answer

I would verify GitHub webhook delivery, confirm the Branch Source configuration, manually trigger a repository scan, check whether branch discovery is enabled, and review Jenkins logs for SCM or authentication errors. If necessary, I'd confirm that the branch actually exists in the remote repository.

### Follow-Up Questions

- What plugins are required?
- How do you manually scan a repository?
- What logs would you inspect?

---

## 7. How do enterprises structure Multibranch Pipelines?

### Production-Level Answer

Enterprises typically create one Multibranch Pipeline per repository, keep Jenkinsfiles minimal, use Shared Libraries for reusable pipeline logic, validate every Pull Request, and restrict deployments to protected branches. GitHub webhooks trigger builds immediately, while security scans are enforced across all branches.

### Follow-Up Questions

- How many repositories can one Jenkins instance manage?
- Why are Shared Libraries important?
- How are multiple environments handled?

---

## 8. What are the advantages of Pull Request builds?

### Production-Level Answer

Pull Request builds ensure that code is compiled, tested, scanned for vulnerabilities, and validated before merging. This prevents unstable or insecure code from reaching shared branches and gives reviewers confidence that the proposed changes meet quality standards.

### Follow-Up Questions

- What stages should a PR pipeline include?
- Should PRs deploy applications?
- What happens if a PR build fails?

---

## 9. What improvements would you make to a Multibranch Pipeline?

### Production-Level Answer

I would integrate Shared Libraries, enable webhook-based triggering, add parallel test execution, cache dependencies, enforce quality gates, automate notifications, use versioned libraries, and separate deployment logic from build logic. These improvements reduce build time, improve maintainability, and increase pipeline reliability.

### Follow-Up Questions

- Which stages can run in parallel?
- How do you reduce pipeline execution time?
- How would you optimize Docker builds?

---

## 10. Explain your production Multibranch Pipeline architecture.

### Production-Level Answer

Our repositories use a single Multibranch Pipeline. GitHub webhooks notify Jenkins whenever code is pushed or a Pull Request is created. Jenkins automatically discovers the branch, loads the appropriate Jenkinsfile, invokes our Enterprise Shared Library, executes language-specific build pipelines, performs SonarQube analysis, OWASP Dependency Check, Docker image creation, Trivy scanning, and for the `main` branch, pushes the image to Amazon ECR and deploys it to Amazon EKS. This architecture provides automated branch management, standardized DevSecOps practices, and consistent deployments across all projects.

### Follow-Up Questions

- How is deployment restricted to the main branch?
- Why is deployment separated into `EKSDeploy()`?
- How do you support Java, Node.js, and Python projects?

---

# Key Takeaways

- A Multibranch Pipeline automatically discovers and manages Git branches.
- One Multibranch Pipeline replaces multiple manually created Jenkins jobs.
- Branch Indexing keeps Jenkins synchronized with the Git repository.
- GitHub Webhooks provide faster and more efficient triggering than repository polling.
- Pull Request validation improves code quality before merge.
- Shared Libraries keep Jenkinsfiles small and reusable.
- Production deployments should be restricted to protected branches such as `main`.
- Combining Multibranch Pipelines with Shared Libraries provides a scalable, enterprise-ready CI/CD architecture.