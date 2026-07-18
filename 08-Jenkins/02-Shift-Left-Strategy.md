# Shift Left Strategy

## Introduction

Traditionally, software testing and security scanning happened **after** development was completed.

This approach caused:

```text
Late Bug Detection

Late Security Detection

Expensive Fixes

Delayed Releases
```

---

Modern DevOps follows:

```text
Shift Left Strategy
```

where testing and security are moved earlier in the Software Development Life Cycle (SDLC).

---

# What Is Shift Left?

Shift Left means:

```text
Move Testing

↓

Move Security

↓

Move Validation

↓

Earlier In Development
```

instead of performing them just before deployment.

---

# Why Is It Called Shift Left?

Traditional SDLC

```text
Requirements

↓

Design

↓

Development

↓

Testing

↓

Security

↓

Deployment
```

Testing happens at the end.

---

Shift Left SDLC

```text
Requirements

↓

Design

↓

Development

↓

Testing

↓

Security

↓

Deployment
```

Testing and security start during development itself.

---

# Traditional CI Process

```text
Developer

↓

Writes Code

↓

Entire Project Completed

↓

Testing

↓

Security Scan

↓

Build

↓

Deployment
```

Problem

```text
Late Detection

Late Fixes

Delayed Releases
```

---

# Shift Left CI Process

```text
Developer Writes Code

↓

Commit

↓

CI Starts

↓

Build

↓

Unit Tests

↓

Static Code Analysis

↓

Dependency Scan

↓

Secrets Scan

↓

IaC Scan

↓

Container Scan

↓

Artifact Creation
```

---

Problems Are Found Immediately.

---

# Why Shift Left Is Needed?

Imagine

```text
100 Developers
```

working on the same application.

---

Without Shift Left

Developer finishes

```text
10,000 Lines Of Code
```

---

Security Team Finds

```text
100 Vulnerabilities
```

---

Developer Must Revisit Old Code.

---

Time Wasted.

---

With Shift Left

Every Commit Is

```text
Built

↓

Tested

↓

Scanned
```

---

Problems Fixed

```text
Immediately
```

---

# Benefits Of Shift Left

```text
Earlier Bug Detection

Earlier Security Detection

Faster Feedback

Lower Development Cost

Better Code Quality

Safer Releases

Reduced Production Failures
```

---

# Cost Of Fixing Bugs

Industry studies consistently show that defects become significantly more expensive to fix the later they are discovered.

Example

```text
Development

↓

Least Expensive
```

---

```text
Testing

↓

More Expensive
```

---

```text
Production

↓

Most Expensive
```

---

Therefore

```text
Find Bugs Early
```

---

# Shift Left In DevOps

Modern DevOps Pipelines Perform

```text
Clone Repository

↓

Compile Code

↓

Unit Testing

↓

Static Code Analysis

↓

Software Composition Analysis

↓

Secrets Scan

↓

IaC Scan

↓

Container Scan

↓

Build Artifact

↓

Deploy
```

---

# Different Types Of Shift Left Testing

## Unit Testing

Purpose

```text
Verify Individual Functions
```

Example

```bash
mvn test
```

---

## Static Code Analysis (SAST)

Purpose

```text
Find Coding Issues

Code Smells

Security Problems
```

Tool

```text
SonarQube
```

---

## Software Composition Analysis (SCA)

Purpose

```text
Detect Vulnerable Dependencies
```

Examples

```text
OWASP Dependency Check

Snyk
```

---

## Secret Scanning

Purpose

```text
Find Hardcoded Passwords

AWS Keys

API Tokens
```

Tools

```text
TruffleHog

GitLeaks
```

---

## Infrastructure As Code Scan

Purpose

```text
Validate Terraform

CloudFormation

Kubernetes YAML
```

Tools

```text
Checkov

Terrascan
```

---

## Container Scan

Purpose

```text
Find OS Vulnerabilities

Package Vulnerabilities
```

Tool

```text
Trivy
```

---

# Shift Left Pipeline Example

```text
Developer Push

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

Unit Test

↓

SonarQube

↓

Dependency Scan

↓

Secrets Scan

↓

Terraform Scan

↓

Docker Build

↓

Trivy Image Scan

↓

Push Docker Image
```

---

# Real Jenkins Pipeline Example

```groovy
pipeline {

    agent any

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

        stage('Dependency Scan') {
            steps {
                sh 'dependency-check.sh'
            }
        }

        stage('Secrets Scan') {
            steps {
                sh 'trufflehog .'
            }
        }

        stage('Terraform Scan') {
            steps {
                sh 'checkov -d terraform/'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t roboshop:latest .'
            }
        }

        stage('Image Scan') {
            steps {
                sh 'trivy image roboshop:latest'
            }
        }

    }
}
```

---

# Real Production Example

RoboShop Catalogue Service

Developer Pushes Code

↓

Jenkins Starts

↓

Clone Repository

↓

Compile Java Code

↓

Run JUnit Tests

↓

SonarQube Scan

↓

OWASP Dependency Check

↓

TruffleHog Secrets Scan

↓

Checkov Terraform Scan

↓

Docker Build

↓

Trivy Image Scan

↓

Push Image To Amazon ECR

↓

Update GitOps Repository

↓

ArgoCD Deploys To Amazon EKS

---

Security Issues Never Reach Production.

---

# Shift Left Vs Traditional Approach

## Traditional

```text
Development

↓

Deployment

↓

Testing

↓

Security

↓

Fix Issues
```

Problems

```text
Late Feedback

Higher Cost

Production Risk
```

---

## Shift Left

```text
Development

↓

Continuous Validation

↓

Continuous Security

↓

Deployment
```

Benefits

```text
Early Detection

Fast Feedback

Secure Releases
```

---

# Best Practices

```text
Run Unit Tests On Every Commit

Run SonarQube During CI

Scan Dependencies Automatically

Scan Terraform Before Deployment

Scan Docker Images Before Push

Fail Pipeline On Critical Vulnerabilities

Never Skip Security Scans
```

---

# Common Interview Questions

## What Is Shift Left Strategy?

### Short Answer

Shift Left is the practice of moving testing, validation, and security earlier in the software development lifecycle.

### Detailed Explanation

Instead of waiting until the end of development, automated testing and security checks are integrated into the CI pipeline so issues are detected immediately after code changes.

### Production Example

```text
Developer Commit

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
Why Is Shift Left Important?

Which Security Scans Are Performed?

How Does Jenkins Support Shift Left?
```

---

## Why Is Shift Left Better Than Traditional Testing?

### Short Answer

Because problems are detected earlier when they are easier and cheaper to fix.

### Detailed Explanation

Finding defects during development reduces rework, shortens release cycles, and improves software quality.

### Production Example

```text
Bug Found During Commit

↓

Fixed In Minutes
```

Instead Of

```text
Bug Found In Production

↓

Emergency Hotfix
```

### Follow-Up Questions

```text
What Is Shift Right?

Can Shift Left Eliminate All Bugs?

Is Shift Left Only About Security?
```

---

## Which Scans Are Commonly Included In A Shift Left Pipeline?

### Short Answer

Unit tests, static code analysis, dependency scanning, secrets scanning, IaC scanning, and container image scanning.

### Detailed Explanation

A production CI pipeline validates application quality, dependencies, infrastructure, and container security before creating deployment artifacts.

### Production Example

```text
SonarQube

↓

OWASP Dependency Check

↓

TruffleHog

↓

Checkov

↓

Trivy
```

### Follow-Up Questions

```text
Which Tools Have You Used?

Which Scan Runs First?

Should The Pipeline Fail On Critical Issues?
```

---

## How Is Shift Left Implemented In Jenkins?

### Short Answer

By adding testing and security stages directly into the Jenkins pipeline before artifact creation.

### Detailed Explanation

Jenkins executes automated validation stages sequentially or in parallel, ensuring only verified code proceeds to packaging and deployment.

### Production Example

```text
Build

↓

Test

↓

SonarQube

↓

Trivy

↓

Docker Build
```

### Follow-Up Questions

```text
Can Stages Run In Parallel?

What Happens If SonarQube Fails?

Can Security Scans Block Deployment?
```

---

# Production Scenarios

## Scenario 1

Developer Commits Code With Unit Test Failure.

### Solution

```text
Pipeline Stops Immediately
```

---

## Scenario 2

Hardcoded AWS Key Found.

### Solution

```text
Secrets Scan Fails Pipeline
```

---

## Scenario 3

Terraform Configuration Has Security Misconfiguration.

### Solution

```text
Checkov Blocks Deployment
```

---

## Scenario 4

Docker Image Contains Critical CVEs.

### Solution

```text
Trivy Fails Pipeline
```

---

## Scenario 5

SonarQube Quality Gate Fails.

### Solution

```text
Pipeline Stops

↓

Developer Fixes Code

↓

Pipeline Re-runs
```

---

# Key Takeaways

```text
Shift Left moves testing and security earlier in the SDLC.

It enables early detection of defects and vulnerabilities.

Modern CI pipelines include multiple automated validation stages.

Security is integrated into development instead of being a final step.

Jenkins pipelines commonly implement Shift Left using testing and scanning stages.

Shift Left improves software quality, reduces cost, and accelerates delivery.
```