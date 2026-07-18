# Jenkins Pipeline

# Introduction

As CI/CD processes became more complex, managing Jenkins jobs through the web interface became difficult. Organizations needed a way to define their entire build process as code, making it easier to version, review, and maintain.

Jenkins introduced **Pipeline as Code**, where the complete CI/CD workflow is written in a file called **Jenkinsfile**.

Today, almost every enterprise uses Jenkins Pipelines instead of Freestyle Jobs.

---

# What is a Jenkins Pipeline?

A Jenkins Pipeline is a collection of automated steps that define how an application is built, tested, scanned, packaged, and deployed.

Instead of manually configuring jobs in the Jenkins UI, the entire workflow is stored as code.

This approach is called:

```text
Pipeline as Code
```

The pipeline definition is stored in a file named:

```text
Jenkinsfile
```

inside the application's Git repository.

---

# Why Do We Need Pipelines?

Suppose your application requires the following process:

```text
Clone Repository

↓

Build Application

↓

Run Unit Tests

↓

Run SonarQube Scan

↓

Build Docker Image

↓

Scan Docker Image

↓

Push Image to Amazon ECR

↓

Deploy to Kubernetes
```

Creating all of these steps manually in Jenkins UI is difficult.

Problems include:

- Difficult to maintain
- No version control
- Hard to review
- Difficult to rollback
- Not reusable

Pipelines solve all of these problems by storing the workflow as code.

---

# Pipeline as Code

A Jenkins Pipeline is stored inside a file called:

```text
Jenkinsfile
```

Example Repository

```text
catalogue/

├── src/
├── pom.xml
├── Dockerfile
└── Jenkinsfile
```

Whenever Jenkins starts a build, it reads the Jenkinsfile and executes the instructions.

---

# Jenkins Pipeline Lifecycle

Every pipeline follows a similar execution flow.

```text
Pipeline Trigger

↓

Pre-Build

↓

Build

↓

Post-Build

↓

Pipeline Ends
```

---

# Pre-Build Phase

The Pre-Build phase prepares the environment before the actual build starts.

Typical activities include:

- Selecting the Jenkins Agent
- Setting Environment Variables
- Configuring Build Options
- Reading Parameters
- Trigger Validation

Example

```text
Agent

↓

Environment Variables

↓

Options

↓

Parameters
```

These concepts will be covered in detail in the next chapters.

---

# Build Phase

This is the main execution phase of the pipeline.

Typical activities include:

```text
Checkout Source Code

↓

Compile Application

↓

Run Unit Tests

↓

Run Security Scans

↓

Build Docker Image

↓

Push Image
```

This is where the actual CI process happens.

---

# Post-Build Phase

The Post-Build section runs after all stages have completed.

Typical tasks include:

- Cleaning Workspace
- Sending Notifications
- Publishing Reports
- Archiving Artifacts
- Executing Cleanup Tasks

Depending on the build result, Jenkins can execute different actions for:

- Success
- Failure
- Aborted
- Always

---

# Jenkins Pipeline Workflow

```text
Developer

↓

Git Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Read Jenkinsfile

↓

Pre-Build

↓

Build

↓

Post-Build

↓

Build Result
```

---

# Pipeline Structure

A Declarative Pipeline is divided into three major sections.

```text
Pipeline

├── Pre-Build

├── Build

└── Post-Build
```

Pre-Build

```text
Agent

Environment

Options

Parameters

Triggers
```

Build

```text
Stages

Stage

Steps

Script

Shell Commands
```

Post-Build

```text
Always

Success

Failure

Aborted

Cleanup
```

---

# Simple Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                sh 'echo "Building Application"'

            }

        }

    }

}
```

---

# Production Pipeline Flow

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout Source

↓

Build

↓

Unit Tests

↓

SonarQube Scan

↓

Docker Build

↓

Trivy Image Scan

↓

Push Image to Amazon ECR

↓

Update GitOps Repository

↓

ArgoCD Sync

↓

Deploy to Amazon EKS
```

---

# Pipeline Advantages

- Pipeline as Code
- Version Controlled
- Stored in Git
- Easy Code Review
- Easy Rollback
- Reusable
- Scalable
- Supports Parallel Execution
- Easy Integration with DevOps Tools
- Enterprise Ready

---

# Pipeline vs Freestyle

| Feature | Freestyle | Pipeline |
|----------|-----------|----------|
| Configuration | Jenkins UI | Jenkinsfile |
| Version Control | No | Yes |
| Code Review | No | Yes |
| Rollback | Difficult | Easy |
| Reusable | No | Yes |
| Enterprise Ready | Limited | Yes |

---

# Real Production Example

A company has 50 microservices.

Each repository contains its own Jenkinsfile.

Example

```text
catalogue/

├── Dockerfile

├── pom.xml

└── Jenkinsfile
```

```text
payment/

├── Dockerfile

├── pom.xml

└── Jenkinsfile
```

Whenever developers push code, Jenkins automatically reads the corresponding Jenkinsfile and executes the CI pipeline.

---

# Best Practices

- Store Jenkinsfiles in the Git repository.
- Keep pipelines modular and readable.
- Fail fast on build or test failures.
- Keep shell scripts minimal.
- Reuse common logic using Shared Libraries.
- Secure sensitive values using Jenkins Credentials.
- Clean workspaces after successful builds.

---

# Common Interview Questions

## What is a Jenkins Pipeline?

### Short Answer

A Jenkins Pipeline is a series of automated steps defined as code in a Jenkinsfile to build, test, scan, package, and deploy applications.

### Detailed Explanation

Pipelines replace manual job configuration with a code-based approach. This enables version control, collaboration, repeatability, and easier maintenance of CI/CD workflows.

### Production Example

A developer pushes code to GitHub. Jenkins reads the Jenkinsfile, builds the application, runs tests, scans the Docker image using Trivy, pushes it to Amazon ECR, and triggers deployment through ArgoCD.

### Follow-up Questions

- What is Pipeline as Code?
- Where is the Jenkinsfile stored?
- Why do enterprises prefer Pipelines?

---

## Why are Pipelines preferred over Freestyle Jobs?

### Short Answer

Because they are version-controlled, reviewable, reusable, and easier to maintain.

### Detailed Explanation

Pipeline definitions are stored in Git alongside the application code. Teams can review changes through Pull Requests, revert to previous versions, and reuse common logic across multiple projects.

### Production Example

A team updates its CI process by modifying the Jenkinsfile, reviewing the changes through GitHub, and merging them after approval.

### Follow-up Questions

- What are the limitations of Freestyle Jobs?
- Can a Pipeline be reused?
- How do Shared Libraries help?

---

# Production Scenarios

## Scenario 1

A developer updates the build process.

### Solution

Modify the Jenkinsfile, commit the changes, and push them to Git. Jenkins automatically executes the updated pipeline.

---

## Scenario 2

The pipeline fails during the build stage.

### Solution

Stop execution immediately, review the console logs, fix the issue, and rerun the pipeline.

---

## Scenario 3

The same pipeline logic is required across multiple repositories.

### Solution

Move common functionality into a Shared Library while keeping repository-specific logic in each Jenkinsfile.

---

# Key Takeaways

```text
A Jenkins Pipeline automates the CI/CD workflow.

Pipelines are written as code in a Jenkinsfile.

Every Pipeline consists of Pre-Build, Build, and Post-Build phases.

Pipelines provide version control, repeatability, and easier maintenance compared to Freestyle Jobs.

Modern enterprises use Pipelines as the standard approach for CI/CD automation.
```