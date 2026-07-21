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