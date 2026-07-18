# Jenkins Introduction and Architecture

# Introduction

Continuous Integration (CI) requires a tool that can automatically build, test, scan, package, and orchestrate software whenever developers commit code.

Jenkins is one of the most widely used open-source CI/CD automation servers that helps automate the software delivery lifecycle.

It integrates with hundreds of DevOps tools through plugins and provides an easy-to-use web interface for managing CI/CD pipelines.

---

# What is Jenkins?

Jenkins is an open-source automation server used to automate software development tasks such as:

- Source Code Checkout
- Build Automation
- Unit Testing
- Security Scanning
- Artifact Creation
- Docker Image Build
- Deployment Automation

Instead of manually performing these activities, Jenkins executes them automatically whenever code changes occur.

---

# Why Do We Need Jenkins?

Imagine a company with:

```text
100 Developers
```

Every developer pushes code several times a day.

Without Jenkins:

```text
Developer

↓

Manual Clone

↓

Manual Build

↓

Manual Testing

↓

Manual Deployment
```

Problems:

- Time consuming
- Human errors
- Inconsistent builds
- Difficult to track failures
- No automation

---

With Jenkins:

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Clone

↓

Build

↓

Test

↓

Scan

↓

Package

↓

Deploy
```

Everything happens automatically.

---

# Why is Jenkins Popular?

The biggest strength of Jenkins is its **Plugin Architecture**.

You mentioned:

> "Jenkins power lies in plugins."

This is true because Jenkins itself provides the automation framework, while plugins allow it to communicate with external tools.

Examples:

```text
Git Plugin

↓

GitHub Plugin

↓

Docker Plugin

↓

Kubernetes Plugin

↓

Terraform Plugin

↓

AWS Plugin

↓

SonarQube Plugin

↓

Slack Plugin

↓

Maven Plugin
```

Without plugins, Jenkins would only be a basic automation server.

---

# Jenkins Features

Jenkins provides several enterprise features.

## Plugin Support

Integrates with thousands of DevOps tools.

Examples:

```text
Git

GitHub

Docker

Terraform

AWS

Kubernetes

SonarQube

Trivy

Maven

Slack
```

---

## Web Interface

Provides an easy-to-use dashboard for:

- Creating jobs
- Monitoring builds
- Viewing logs
- Managing users
- Configuring plugins

---

## Logging

Every build stores complete execution logs.

Example:

```text
Build Started

↓

Git Clone

↓

Maven Build

↓

Tests Passed

↓

Docker Build

↓

Build Success
```

Logs help troubleshoot failures quickly.

---

## Scheduling

Jobs can run:

```text
Every Hour

Every Day

Every Week

Specific Time

Cron Schedule
```

Example:

```text
Nightly Build

↓

12:00 AM
```

---

## Webhooks

Instead of checking GitHub continuously, GitHub notifies Jenkins whenever new code is pushed.

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins Starts Pipeline
```

This provides immediate feedback.

---

## RBAC (Role-Based Access Control)

Different users can have different permissions.

Example:

```text
Admin

↓

Create Jobs

Install Plugins

Manage Jenkins
```

Developer

```text
Trigger Build

View Logs

Read Jobs
```

Tester

```text
View Reports

Download Artifacts
```

---

## Shared Libraries

Organizations often have hundreds of Jenkins pipelines.

Instead of repeating common code, reusable functions are stored in Shared Libraries.

Example:

```groovy
@Library('company-library') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildApplication()
            }
        }
    }
}
```

---

## Audit

Jenkins records:

- User login
- Job execution
- Configuration changes
- Plugin installation
- Build history

Useful for enterprise compliance and troubleshooting.

---

# Jenkins Architecture

Basic Architecture

```text
                +----------------------+
                |      Developer       |
                +----------+-----------+
                           |
                           | Git Push
                           |
                           v
                +----------------------+
                |       GitHub         |
                +----------+-----------+
                           |
                      Webhook
                           |
                           v
                +----------------------+
                |      Jenkins         |
                |      Controller      |
                +----------+-----------+
                           |
             -------------------------------
             |              |              |
             |              |              |
             v              v              v
       Linux Agent     Windows Agent    Mac Agent
             |              |              |
             |              |              |
         Build/Test      Build/Test    Build/Test
```

---

# How Jenkins Works

Step 1

Developer pushes code.

↓

Step 2

GitHub sends webhook.

↓

Step 3

Jenkins starts the pipeline.

↓

Step 4

Source code is cloned.

↓

Step 5

Application is built.

↓

Step 6

Unit tests are executed.

↓

Step 7

Security scans are performed.

↓

Step 8

Docker image is built.

↓

Step 9

Artifact/Image is stored.

↓

Step 10

Deployment starts.

---

# Real Production Example

Microservices Application

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Clone Repository

↓

Maven Build

↓

JUnit Tests

↓

SonarQube Scan

↓

Trivy Image Scan

↓

Docker Build

↓

Push Image To Amazon ECR

↓

Update GitOps Repository

↓

ArgoCD Sync

↓

Amazon EKS
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

        stage('Test') {
            steps {
                sh 'mvn test'
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
                sh 'trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:latest'
            }
        }

    }
}
```

---

# Advantages of Jenkins

- Open Source
- Huge Plugin Ecosystem
- Easy Integration
- Platform Independent
- Pipeline as Code
- Distributed Builds
- Easy Monitoring
- Detailed Logs
- RBAC Support
- Shared Libraries
- Large Community Support

---

# Common Interview Questions

## What is Jenkins?

### Short Answer

Jenkins is an open-source automation server used to automate building, testing, scanning, and deploying applications.

### Detailed Explanation

Jenkins enables Continuous Integration by automatically executing pipelines whenever code changes are committed. It integrates with various DevOps tools using plugins and supports Pipeline as Code through Jenkinsfiles.

### Production Example

```text
Developer Push

↓

GitHub

↓

Jenkins

↓

Build

↓

Test

↓

Docker Image

↓

ECR
```

### Follow-up Questions

- Why is Jenkins called an automation server?
- How does Jenkins trigger builds?
- Which CI tools have you used?

---

## Why is Jenkins Popular?

### Short Answer

Jenkins is popular because of its extensive plugin ecosystem, Pipeline as Code support, and strong integration with DevOps tools.

### Detailed Explanation

Its plugin-based architecture allows organizations to integrate Jenkins with source control, cloud providers, container platforms, security scanners, and deployment tools without writing custom integrations.

### Production Example

```text
GitHub

↓

Jenkins

↓

SonarQube

↓

Docker

↓

Trivy

↓

Amazon ECR

↓

ArgoCD
```

### Follow-up Questions

- How many plugins does Jenkins have?
- What is the role of plugins?
- Can Jenkins work without plugins?

---

## What are the key features of Jenkins?

### Short Answer

Plugin support, web interface, logging, scheduling, RBAC, webhooks, auditing, shared libraries, and Pipeline as Code.

### Detailed Explanation

These features make Jenkins suitable for enterprise CI/CD by providing automation, security, scalability, and maintainability.

### Production Example

A production pipeline uses GitHub webhooks, RBAC for user access, shared libraries for reusable code, and plugins for integrating with Docker, Kubernetes, AWS, and SonarQube.

### Follow-up Questions

- What is RBAC?
- What are shared libraries?
- Why are webhooks preferred over polling?

---

# Production Scenarios

## Scenario 1

A developer pushes code to GitHub.

**Solution**

GitHub webhook automatically triggers Jenkins to start the pipeline.

---

## Scenario 2

The organization uses AWS, Docker, Kubernetes, and SonarQube.

**Solution**

Install the required Jenkins plugins to integrate with each tool.

---

## Scenario 3

The same build steps are repeated across multiple projects.

**Solution**

Move the common logic into a Jenkins Shared Library and reuse it across pipelines.

---

# Key Takeaways

- Jenkins is an open-source CI/CD automation server.
- Its biggest strength is its plugin ecosystem.
- Jenkins automates build, test, scan, and deployment workflows.
- Enterprise features include RBAC, webhooks, scheduling, logging, auditing, and shared libraries.
- Jenkins acts as the central orchestrator of a modern DevOps CI pipeline.