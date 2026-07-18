# Freestyle vs Pipeline

# Introduction

Jenkins provides multiple ways to automate software builds.

The two most commonly used job types are:

- Freestyle Jobs
- Pipeline Jobs

Freestyle Jobs were the original way of creating Jenkins jobs using the web interface.

As CI/CD pipelines became more complex, Jenkins introduced **Pipeline as Code**, allowing the entire build process to be written in a Jenkinsfile.

Today, most organizations prefer Pipeline Jobs because they are version-controlled, reusable, and easier to maintain.

---

# What is a Freestyle Job?

A Freestyle Job is the simplest type of Jenkins job.

Everything is configured through the Jenkins web interface.

Example

```text
Build Steps

↓

Shell Commands

↓

Post Build Actions
```

No code is required.

---

## Freestyle Job Workflow

```text
Developer Push

↓

GitHub

↓

Jenkins Freestyle Job

↓

Clone Repository

↓

Execute Shell Commands

↓

Build

↓

Archive Artifacts
```

---

## Freestyle Job Configuration

A Freestyle Job is divided into three sections.

```text
Pre-Build

↓

Build

↓

Post-Build
```

---

### Pre-Build

Activities performed before the actual build.

Examples

- Clean Workspace
- Clone Repository
- Download Dependencies

---

### Build

Main build activities.

Examples

```bash
mvn clean package
```

or

```bash
npm install

npm run build
```

---

### Post-Build

Activities performed after the build.

Examples

- Archive Artifacts
- Email Notification
- Trigger Another Job
- Publish Test Reports

---

# Problems with Freestyle Jobs

Your trainer highlighted several drawbacks.

## 1. No Version Control

Job configuration exists only inside Jenkins.

Changes are not stored in Git.

If the Jenkins server crashes and there is no backup, the configuration can be lost.

---

## 2. Difficult to Modify

Every change requires opening the Jenkins UI.

Updating multiple jobs becomes time-consuming.

---

## 3. Cannot Easily Revert

Suppose a working job is modified incorrectly.

Freestyle Jobs do not provide built-in version history like Git.

Returning to a previous configuration is difficult.

---

## 4. Difficult to Track Changes

Questions such as:

- Who changed the job?
- When was it changed?
- What exactly changed?

are difficult to answer.

---

## 5. Hard to Review

Freestyle configurations cannot be reviewed using Pull Requests.

Developers cannot perform code reviews before modifying the CI process.

---

# What is Pipeline?

Pipeline is Jenkins' modern approach for creating CI/CD workflows.

Instead of configuring jobs in the UI, the entire pipeline is written as code.

This is called:

```text
Pipeline as Code
```

The pipeline definition is stored inside a file called:

```text
Jenkinsfile
```

---

# Pipeline Workflow

```text
Developer

↓

Git Push

↓

Jenkinsfile

↓

Clone

↓

Build

↓

Test

↓

Scan

↓

Docker Build

↓

Deploy
```

---

# Simple Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/company/app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

    }

}
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    environment {
        IMAGE_NAME = "catalogue"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image ${IMAGE_NAME}:latest'
            }
        }

    }

}
```

---

# Advantages of Pipeline

- Stored in Git
- Version Controlled
- Easy to Review
- Easy to Modify
- Reusable
- Supports Parallel Execution
- Supports Shared Libraries
- Easy Rollback
- Better Collaboration
- Infrastructure as Code approach

---

# Freestyle vs Pipeline

| Feature | Freestyle | Pipeline |
|----------|-----------|----------|
| Configuration | Jenkins UI | Jenkinsfile |
| Version Control | ❌ | ✅ |
| Git Integration | Limited | Excellent |
| Code Review | ❌ | ✅ |
| Rollback | Difficult | Easy |
| Reusable | ❌ | ✅ |
| Parallel Builds | Limited | Supported |
| Shared Libraries | ❌ | Supported |
| Best for Enterprise | ❌ | ✅ |

---

# Why Enterprises Prefer Pipeline

Large organizations may have:

- Hundreds of repositories
- Thousands of builds
- Multiple teams

Managing all jobs manually becomes difficult.

Using Jenkinsfiles allows:

```text
Git Repository

↓

Jenkinsfile

↓

Pull Request

↓

Code Review

↓

Merge

↓

Automatic Pipeline
```

The CI process is treated like application code.

---

# Real Production Example

A microservices application contains:

```text
20 Services
```

Each service has its own Jenkinsfile.

Example

```text
catalogue

↓

Jenkinsfile

↓

payment

↓

Jenkinsfile

↓

shipping

↓

Jenkinsfile
```

Every repository controls its own CI pipeline.

---

# Best Practices

- Always use Pipeline Jobs for new projects.
- Store Jenkinsfiles in Git.
- Keep pipelines modular.
- Use Shared Libraries for common logic.
- Review pipeline changes through Pull Requests.
- Avoid complex shell scripts inside Jenkinsfiles.

---

# Common Interview Questions

## What is the difference between Freestyle and Pipeline Jobs?

### Short Answer

Freestyle Jobs are configured through the Jenkins UI, while Pipeline Jobs define the CI/CD workflow as code using a Jenkinsfile.

### Detailed Explanation

Freestyle Jobs are suitable for simple tasks but become difficult to manage as projects grow. Pipeline Jobs store the build configuration in Git, making it version-controlled, reviewable, and reusable.

### Production Example

An enterprise with hundreds of repositories stores a Jenkinsfile in each repository, allowing teams to review pipeline changes through Pull Requests before merging.

### Follow-up Questions

- Why is Pipeline preferred in modern DevOps?
- What is Pipeline as Code?
- Where is the Jenkinsfile stored?

---

## What are the limitations of Freestyle Jobs?

### Short Answer

Freestyle Jobs are not version-controlled, difficult to review, and harder to maintain.

### Detailed Explanation

Since the configuration resides in the Jenkins UI, changes cannot be tracked through Git, making collaboration and rollback more challenging.

### Production Example

A pipeline configuration was accidentally modified in the Jenkins UI. Without a backup or version history, restoring the previous state required manually recreating the configuration.

### Follow-up Questions

- Can Freestyle Jobs use Git?
- Why are Pull Requests not possible?
- How do enterprises overcome these limitations?

---

## What is a Jenkinsfile?

### Short Answer

A Jenkinsfile is a text file that defines the Jenkins pipeline as code.

### Detailed Explanation

The Jenkinsfile contains all pipeline stages, steps, environment variables, and post-build actions. It is stored in the application's source code repository alongside the application code.

### Production Example

When a developer updates the build process, they modify the Jenkinsfile and submit it through a Pull Request. After review and merge, Jenkins automatically uses the updated pipeline.

### Follow-up Questions

- Where is the Jenkinsfile stored?
- What languages are Jenkinsfiles written in?
- What are Declarative and Scripted Pipelines?

---

# Production Scenarios

## Scenario 1

A developer needs to update the build process.

### Solution

Modify the Jenkinsfile, create a Pull Request, review the changes, and merge them. Jenkins automatically uses the updated pipeline.

---

## Scenario 2

The same CI process is required for multiple repositories.

### Solution

Store common logic in a Shared Library and reference it from each Jenkinsfile.

---

## Scenario 3

A pipeline update introduces an issue.

### Solution

Revert the Jenkinsfile using Git to restore the previous working pipeline.

---

# Key Takeaways

```text
Freestyle Jobs are UI-based and suitable for simple automation tasks.

Pipeline Jobs define CI/CD workflows as code using a Jenkinsfile.

Pipelines support version control, code reviews, rollback, parallel execution, and shared libraries.

Modern enterprises prefer Pipeline Jobs because they are easier to maintain, scale, and collaborate on.
```