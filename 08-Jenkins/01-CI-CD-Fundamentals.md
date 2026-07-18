# CI/CD Fundamentals

## Introduction

Software development is not just about writing code.

After developers complete coding, the application must go through several stages before reaching production.

These stages include:

```text
Clone Source Code

↓

Build Application

↓

Run Unit Tests

↓

Perform Security Scans

↓

Create Artifact / Docker Image

↓

Store Artifact

↓

Deploy Application
```

---

Without automation:

```text
Manual Builds

Manual Testing

Manual Deployment

Higher Failure Rate

Slow Releases
```

---

Modern organizations solve this problem using:

```text
CI/CD
```

---

# What is CI?

CI stands for:

```text
Continuous Integration
```

---

Continuous Integration is the practice of integrating developers' code changes into a central repository multiple times a day.

Each integration automatically performs:

```text
Clone Source Code

↓

Build

↓

Unit Testing

↓

Code Quality Checks

↓

Security Scanning

↓

Artifact Creation
```

---

Goal:

```text
Find Problems Early
```

instead of discovering them during deployment.

---

# What is CD?

CD stands for either:

```text
Continuous Delivery
```

or

```text
Continuous Deployment
```

---

Continuous Delivery

```text
Code

↓

Build

↓

Test

↓

Ready For Production

↓

Manual Approval

↓

Deploy
```

---

Continuous Deployment

```text
Code

↓

Build

↓

Test

↓

Deploy Automatically

↓

Production
```

---

# CI vs CD

## Continuous Integration

Focuses On

```text
Building

Testing

Scanning

Packaging
```

---

Output

```text
Artifact

Docker Image
```

---

## Continuous Delivery

Focuses On

```text
Deployment Readiness
```

---

Requires

```text
Manual Approval
```

before production deployment.

---

## Continuous Deployment

Focuses On

```text
Fully Automated Deployment
```

---

No Human Intervention Required.

---

# Why Do We Need CI/CD?

Imagine:

```text
50 Developers
```

working on the same application.

---

Without CI/CD

```text
Developer A

↓

Developer B

↓

Developer C

↓

Manual Build

↓

Manual Testing

↓

Manual Deployment
```

Problems

```text
Build Failures

Integration Issues

Late Bug Detection

Slow Releases

Human Errors
```

---

With CI/CD

```text
Developer Pushes Code

↓

CI Pipeline Starts

↓

Automatic Build

↓

Automatic Testing

↓

Automatic Security Scan

↓

Artifact Creation

↓

Deployment
```

Benefits

```text
Fast Releases

Reliable Builds

Early Bug Detection

Consistent Deployments

Improved Quality
```

---

# CI Workflow

Typical Continuous Integration Pipeline

```text
Developer

↓

Git Push

↓

Webhook

↓

CI Tool

↓

Clone Repository

↓

Install Dependencies

↓

Compile Code

↓

Run Unit Tests

↓

Static Code Analysis

↓

Security Scans

↓

Package Application

↓

Create Artifact

↓

Store Artifact
```

---

# CD Workflow

Typical Continuous Delivery Pipeline

```text
Artifact

↓

Deploy To DEV

↓

Testing

↓

Deploy To QA

↓

User Acceptance Testing

↓

Approval

↓

Deploy To Production
```

---

# Complete CI/CD Workflow

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Clone

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Dependency Scan

↓

Secrets Scan

↓

IaC Scan

↓

Container Scan

↓

Create Artifact

↓

Docker Image

↓

Push To Registry

↓

Deploy DEV

↓

QA

↓

UAT

↓

Production
```

---

# CI Tools

Popular CI Tools

```text
Jenkins

GitHub Actions

GitLab CI

Azure DevOps

CircleCI

TeamCity

Bamboo
```

---

Most Popular In Enterprises

```text
Jenkins
```

---

# Why Jenkins Is Popular?

Jenkins Provides

```text
Thousands Of Plugins

Easy Integration

RBAC

Scheduling

Logging

Auditing

Webhooks

Pipeline As Code

Shared Libraries
```

---

# What Happens Inside CI?

Every CI Pipeline Performs

```text
Clone Repository
```

Example

```bash
git clone https://github.com/company/roboshop.git
```

---

Build Application

Example

```bash
mvn clean package
```

or

```bash
npm install
npm run build
```

---

Run Unit Tests

Example

```bash
mvn test
```

---

Static Code Analysis

Example

```text
SonarQube
```

---

Dependency Scanning

Example

```text
OWASP Dependency Check
```

---

Filesystem Scan

Example

```bash
trivy fs .
```

---

Secrets Scan

Example

```bash
trufflehog .
```

---

Infrastructure Scan

Example

```bash
checkov -d terraform/
```

---

Container Scan

Example

```bash
trivy image roboshop:v1
```

---

Create Artifact

Examples

```text
JAR

WAR

ZIP

Docker Image
```

---

Store Artifact

Examples

```text
JFrog Artifactory

Nexus

Amazon ECR

Docker Hub
```

---

# Real Production Example

Roboshop Payment Service

Pipeline

```text
Developer Pushes Code

↓

GitHub Webhook

↓

Jenkins Starts

↓

Clone Repository

↓

Maven Build

↓

Unit Tests

↓

SonarQube Scan

↓

Trivy Filesystem Scan

↓

Dependency Check

↓

Docker Build

↓

Container Scan

↓

Push Docker Image To ECR

↓

Update GitOps Repository

↓

ArgoCD Sync

↓

Deploy To Amazon EKS
```

---

# Shift Left Strategy

Modern CI follows

```text
Shift Left
```

---

Instead Of

```text
Testing At The End
```

---

We Perform

```text
Testing

Scanning

Validation
```

during development itself.

---

This Topic Is Covered In Detail In

```text
02-Shift-Left-Strategy.md
```

---

# CI Pipeline Example

Simple Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/company/roboshop.git'
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

        stage('Package') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar'
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
        APP_NAME = "payment"
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

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${APP_NAME}:latest .'
            }
        }

    }
}
```

---

# Benefits Of CI/CD

```text
Faster Releases

Better Code Quality

Automatic Testing

Early Bug Detection

Automatic Security Validation

Reduced Human Errors

Repeatable Builds

Improved Collaboration

Faster Feedback

Production Confidence
```

---

# Common Interview Questions

## What Is Continuous Integration?

### Short Answer

Continuous Integration is the practice of automatically integrating, building, testing, scanning, and packaging code whenever developers commit changes.

### Detailed Explanation

CI ensures that every code change is validated immediately using automated builds, tests, and security checks, allowing teams to detect problems early.

### Production Example

```text
Developer Push

↓

Jenkins

↓

Build

↓

Tests

↓

Scans

↓

Artifact
```

### Follow-Up Questions

```text
Why Is CI Needed?

What Happens During CI?

Which CI Tools Have You Used?
```

---

## Difference Between CI, Continuous Delivery And Continuous Deployment?

### Short Answer

CI focuses on integration and validation, Continuous Delivery prepares software for deployment with manual approval, and Continuous Deployment automatically releases software to production.

### Detailed Explanation

CI automates build and testing, while CD automates deployment. Delivery includes a manual approval gate, whereas Deployment is fully automated.

### Production Example

```text
CI

↓

Artifact

↓

Continuous Delivery

↓

Approval

↓

Production
```

### Follow-Up Questions

```text
Which One Does Your Organization Use?

Why Prefer Continuous Delivery?

When Is Continuous Deployment Suitable?
```

---

## What Happens In A Typical CI Pipeline?

### Short Answer

A CI pipeline clones the repository, builds the application, runs tests, performs security scans, and creates deployable artifacts.

### Detailed Explanation

Modern CI pipelines validate both application quality and security before producing an artifact or container image.

### Production Example

```text
Clone

↓

Build

↓

Test

↓

SonarQube

↓

Trivy

↓

Artifact
```

### Follow-Up Questions

```text
Which Scans Do You Run?

Where Are Artifacts Stored?

What Happens If Tests Fail?
```

---

## Why Is CI Important In DevOps?

### Short Answer

CI provides rapid feedback, improves quality, and reduces deployment failures.

### Detailed Explanation

Automated validation catches defects and vulnerabilities early, reducing integration issues and increasing release confidence.

### Production Example

```text
Developer Commit

↓

CI Validation

↓

Only Verified Code Moves Forward
```

### Follow-Up Questions

```text
How Does CI Reduce Risk?

What Is Shift Left?

How Frequently Should Developers Integrate Code?
```

---

# Production Scenarios

## Scenario 1

Developer Pushes Broken Code.

### Solution

```text
CI Pipeline Stops At Build Or Test Stage
```

---

## Scenario 2

Security Vulnerability Found.

### Solution

```text
Pipeline Fails

↓

Developer Fixes Issue

↓

Pipeline Re-runs
```

---

## Scenario 3

Docker Image Needed For Deployment.

### Solution

```text
CI Builds

↓

Scans

↓

Pushes Image To Registry
```

---

## Scenario 4

Multiple Developers Commit Simultaneously.

### Solution

```text
CI Validates Every Commit Automatically
```

---

## Scenario 5

Infrastructure Changes Submitted.

### Solution

```text
Terraform Validation

↓

Security Scan

↓

Approval

↓

Deployment
```

---

# Key Takeaways

```text
Continuous Integration automates build, testing, scanning, and packaging.

Continuous Delivery prepares software for deployment with approvals.

Continuous Deployment releases software automatically.

CI improves code quality and accelerates feedback.

Modern CI follows Shift Left principles.

Artifacts may be JARs, WARs, ZIPs, or Docker images.

Jenkins is one of the most widely used enterprise CI tools.

A production CI pipeline includes build, tests, security scans, artifact creation, and registry publishing before deployment.
```