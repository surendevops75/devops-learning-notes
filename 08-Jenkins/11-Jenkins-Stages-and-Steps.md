# Jenkins Stages and Steps

# Introduction

A Jenkins Pipeline is not just a collection of commands executed one after another. It is a structured workflow that divides the software delivery process into logical phases called **Stages**.

Each stage represents a major milestone in the CI/CD pipeline, such as building the application, running tests, performing security scans, packaging artifacts, or deploying to an environment.

Inside every stage are **Steps**, which are the individual tasks Jenkins executes. These steps may compile code, run shell commands, execute scripts, archive artifacts, or deploy applications.

Think of a Jenkins Pipeline as constructing a building:

- **Pipeline** → The entire construction project
- **Stages** → Foundation, Walls, Roof, Painting
- **Steps** → Individual tasks inside each phase

This structured approach makes pipelines easier to understand, monitor, troubleshoot, and maintain.

---

# Why Do We Need Stages?

Imagine executing every build command in one long script.

```text
Compile Code

Run Unit Tests

Run SonarQube

Run Dependency Check

Build Docker Image

Push Docker Image

Deploy Application

Run Smoke Tests

Notify Team
```

Although Jenkins can execute this, there are several problems.

## Problems Without Stages

- Difficult to identify where a failure occurred.
- Poor visualization in Jenkins UI.
- Hard to restart from a specific point.
- Difficult for teams to collaborate.
- Logs become large and difficult to analyze.
- No logical separation of CI/CD activities.

Now compare it with a stage-based pipeline.

```text
Pipeline

│

├── Build

├── Test

├── Security Scan

├── Package

├── Deploy

└── Post Actions
```

Each stage has a clear responsibility.

When a stage fails, Jenkins immediately highlights it, making troubleshooting much easier.

---

# Real Production Example

Consider a microservices application deployed to Amazon EKS.

Instead of executing hundreds of commands together, the pipeline is divided into stages.

```text
Developer

    │

    ▼

Git Push

    │

    ▼

Build Stage
Compile Application

    │

    ▼

Test Stage
Run Unit Tests

    │

    ▼

Security Stage

SonarQube

OWASP Dependency Check

Trivy

Checkov

    │

    ▼

Package Stage

Docker Build

Docker Push

    │

    ▼

Deploy Stage

Helm Upgrade

    │

    ▼

Smoke Test

    │

    ▼

Success
```

Each stage represents a checkpoint in the software delivery process.

---

# Pipeline Execution Flow

A Declarative Pipeline executes stages sequentially unless parallel execution is explicitly configured.

```text
Pipeline Starts

        │

        ▼

Initialize Agent

        │

        ▼

Load Environment Variables

        │

        ▼

Execute Stage 1

        │

        ▼

Execute Stage 2

        │

        ▼

Execute Stage 3

        │

        ▼

Execute Stage 4

        │

        ▼

Run Post Actions

        │

        ▼

Pipeline Ends
```

By default, Jenkins waits for one stage to finish before starting the next.

---

# What is a Stage?

A **Stage** is a logical phase of a Jenkins Pipeline.

It groups related tasks together so they can be executed, monitored, and reported as a single unit.

Syntax

```groovy
stage('Build') {

    steps {

        echo "Building Application"

    }

}
```

Every stage has:

- A unique name
- A set of steps
- Independent execution status
- Separate logs
- Separate visualization in Jenkins

---

# Stage Lifecycle

Every stage follows the same lifecycle.

```text
Stage Starts

      │

      ▼

Allocate Resources

      │

      ▼

Execute Steps

      │

      ▼

Step Success?

     │      │

    Yes     No

     │       │

     ▼       ▼

Next Stage  Pipeline Stops
```

If any step inside a stage fails, Jenkins marks the entire stage as failed.

Unless configured otherwise, the pipeline stops immediately.

---

# Understanding Steps

A **Step** is the smallest executable unit inside a Jenkins Pipeline.

A stage can contain one step or multiple steps.

Example

```groovy
stage('Build') {

    steps {

        echo "Compiling"

        sh "mvn clean package"

    }

}
```

In this example:

- `echo` is one step.
- `sh` is another step.

Both belong to the **Build** stage.

---

# Stage vs Step

| Stage | Step |
|--------|------|
| Logical phase | Individual task |
| Groups related activities | Performs one action |
| Appears in Jenkins UI | Executes inside a stage |
| Can contain many steps | Cannot contain stages |
| Example: Build | Example: `sh`, `echo`, `git` |

Think of it like this:

```text
Pipeline

│

├── Stage

│     ├── Step

│     ├── Step

│     └── Step

├── Stage

│     ├── Step

│     └── Step

└── Stage

      ├── Step

      └── Step
```

---

# Understanding the Trainer Pipeline

The trainer's Jenkinsfile contains three stages.

```groovy
stages {

    stage('Build') {

        steps {

            script {

                sh """

                echo "Building"

                """

            }

        }

    }

    stage('Test') {

        steps {

            script {

                sh """

                echo "Testing"

                """

            }

        }

    }

    stage('Deploy') {

        steps {

            script {

                sh """

                echo "Deploying"

                """

            }

        }

    }

}
```

Execution Flow

```text
Pipeline

│

├── Build

│      │

│      ▼

│   Execute Shell Commands

│

├── Test

│      │

│      ▼

│   Execute Shell Commands

│

└── Deploy

       │

       ▼

   Execute Shell Commands
```

Each stage performs one responsibility.

This separation makes the pipeline clean and maintainable.

---

# Why Are Stages Important in Production?

Imagine a deployment pipeline for 100 microservices.

Without stages, the console output would contain thousands of lines of logs with no clear indication of which phase failed.

With stages, Jenkins immediately highlights the failing phase.

Example

```text
✔ Build

✔ Unit Test

✔ SonarQube

✔ Dependency Check

✖ Docker Build

Skipped Deploy
```

A DevOps engineer can immediately focus on the Docker build stage instead of searching through the entire log.

---

# Naming Stages

Stage names should clearly describe their purpose.

Good Examples

```text
Build

Unit Test

Security Scan

Docker Build

Deploy to Dev

Deploy to Production
```

Poor Examples

```text
Stage1

Test123

ABC

StepOne
```

Meaningful names improve readability and make Jenkins dashboards easier to understand.

---

# Common Production Stages

Although every organization has its own pipeline, most CI/CD pipelines follow a similar structure.

```text
Checkout

↓

Build

↓

Unit Test

↓

Code Quality

↓

Security Scan

↓

Package

↓

Docker Build

↓

Push Image

↓

Deploy

↓

Smoke Test

↓

Post Actions
```

This workflow ensures that only validated and secure applications progress through the delivery pipeline.
---

# Common Jenkins Steps

A **Step** is the smallest executable unit in a Jenkins Pipeline. Every task performed inside a stage is represented by one or more steps.

Jenkins provides dozens of built-in steps, but a small set is used in almost every production pipeline.

---

# echo Step

## Purpose

The `echo` step prints messages to the Jenkins console log.

It is mainly used to:

- Display progress
- Print variable values
- Improve debugging
- Make logs easier to understand

Syntax

```groovy
echo "Build Started"
```

Production Example

```groovy
echo "Deploying Version ${params.VERSION}"
```

Best Practice

Use meaningful log messages rather than generic text.

Good

```groovy
echo "Running SonarQube Analysis"
```

Poor

```groovy
echo "Done"
```

---

# sh Step

## Purpose

The `sh` step executes shell commands on Linux agents.

It is one of the most frequently used Jenkins steps because most DevOps tools are executed from the command line.

Syntax

```groovy
sh "pwd"
```

Multiple Commands

```groovy
sh """

pwd

ls -l

whoami

"""
```

Production Example

```groovy
sh """

mvn clean package

docker build -t catalogue:${params.VERSION} .

"""
```

Best Practice

Group related shell commands inside one `sh` block to improve readability.

---

# script Step

## Purpose

The `script` step allows Groovy scripting inside a Declarative Pipeline.

Whenever Declarative syntax is insufficient, wrap Groovy logic inside a `script` block.

Syntax

```groovy
script {

    echo "Inside Script Block"

}
```

Production Example

```groovy
script {

    if(params.DEPLOY){

        echo "Deployment Enabled"

    }

}
```

Use `script` only when necessary. Prefer Declarative syntax whenever possible.

---

# git Step

## Purpose

The `git` step checks out source code from a Git repository.

Syntax

```groovy
git branch: "main",
url: "https://github.com/company/catalogue.git"
```

Production Example

```groovy
git branch: params.BRANCH,
url: "https://github.com/company/catalogue.git"
```

This allows developers to build different branches using the same pipeline.

---

# checkout Step

## Purpose

The `checkout` step provides advanced source code checkout functionality.

Unlike the `git` step, it supports multiple SCM systems and advanced Git options.

Example

```groovy
checkout scm
```

Production Example

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'https://github.com/company/catalogue.git'
    ]]
])
```

Use `checkout` when advanced Git configuration is required.

---

# dir Step

## Purpose

Executes commands inside a specific directory.

Syntax

```groovy
dir("backend"){

    sh "mvn clean package"

}
```

Production Example

```text
Workspace

├── frontend

└── backend
```

```groovy
dir("frontend"){

    sh "npm install"

}
```

---

# deleteDir Step

## Purpose

Deletes the current workspace directory.

Syntax

```groovy
deleteDir()
```

Production Use

Useful for cleaning temporary files before or after builds.

---

# archiveArtifacts Step

## Purpose

Stores build artifacts inside Jenkins after the pipeline finishes.

Syntax

```groovy
archiveArtifacts artifacts: "target/*.jar"
```

Production Example

Archive Java JAR files after a successful build.

```groovy
archiveArtifacts artifacts: "target/*.jar"
```

This allows developers to download the generated artifact directly from Jenkins.

---

# stash and unstash

## Purpose

Transfers files between stages running on different agents.

Example

Build Stage

```groovy
stash name: "jar",

includes: "target/*.jar"
```

Deploy Stage

```groovy
unstash "jar"
```

Production Use

Useful when Build and Deploy stages execute on different Jenkins agents.

---

# input Step

## Purpose

Pauses the pipeline and waits for manual approval.

Syntax

```groovy
input "Deploy to Production?"
```

Production Example

```text
Build

↓

Testing

↓

Manager Approval

↓

Production Deployment
```

This helps prevent accidental production deployments.

---

# timeout Step

## Purpose

Stops a stage if it exceeds a specified duration.

Syntax

```groovy
timeout(time:10, unit:'MINUTES'){

    sh "./integration-tests.sh"

}
```

Production Benefit

Prevents pipelines from hanging indefinitely.

---

# retry Step

## Purpose

Retries a failed operation automatically.

Syntax

```groovy
retry(3){

    sh "./deploy.sh"

}
```

Production Example

Temporary network failures often succeed on retry.

---

# sleep Step

## Purpose

Pauses pipeline execution.

Syntax

```groovy
sleep(time:10, unit:"SECONDS")
```

Production Example

Wait for a Kubernetes deployment to become ready before verifying application health.

---

# cleanWs Step

## Purpose

Deletes the Jenkins workspace after pipeline execution.

Trainer Example

```groovy
post{

    always{

        cleanWs()

    }

}
```

Benefits

- Saves disk space
- Prevents stale files
- Ensures clean builds

---

# Typical Production Pipeline

A production pipeline consists of multiple stages, each responsible for one phase of the software delivery lifecycle.

```text
Checkout

↓

Build

↓

Test

↓

Security

↓

Package

↓

Deploy

↓

Post Actions
```

Let's understand each stage.

---

# Build Stage

## Purpose

The Build stage converts source code into executable artifacts.

Typical activities include:

- Dependency download
- Compilation
- Package generation

Example

```groovy
stage("Build"){

    steps{

        sh """

        mvn clean package

        """

    }

}
```

Output

```text
catalogue-1.0.jar
```

---

# Test Stage

## Purpose

Ensures the application functions correctly before deployment.

Typical activities

- Unit Tests
- Integration Tests
- Code Coverage

Example

```groovy
stage("Test"){

    steps{

        sh """

        mvn test

        """

    }

}
```

If tests fail, Jenkins stops the pipeline immediately.

---

# Security Stage

Modern DevSecOps pipelines include security scanning before deployment.

A typical security stage integrates multiple tools.

```text
Source Code

      │

      ▼

SonarQube

      │

      ▼

OWASP Dependency Check

      │

      ▼

Trivy

      │

      ▼

Checkov

      │

      ▼

Deployment
```

---

## SonarQube

Purpose

Detects:

- Bugs
- Code Smells
- Security Hotspots
- Maintainability Issues

Example

```groovy
sh """

mvn sonar:sonar

"""
```

---

## OWASP Dependency Check

Purpose

Identifies vulnerable third-party libraries.

Example

```groovy
sh """

dependency-check.sh \
--scan . \
--format HTML

"""
```

---

## Trivy

Purpose

Scans Docker images for vulnerabilities.

Example

```groovy
sh """

trivy image catalogue:${params.VERSION}

"""
```

---

## Checkov

Purpose

Scans Infrastructure as Code.

Example

```groovy
sh """

checkov -d terraform/

"""
```

Only after all security checks pass should the pipeline continue.

---

# Package Stage

## Purpose

Creates deployable artifacts.

Depending on the application, this may include:

- JAR files
- WAR files
- Docker Images
- Helm Charts

Example

```groovy
stage("Package"){

    steps{

        sh """

        docker build -t catalogue:${params.VERSION} .

        """

    }

}
```

---

# Deploy Stage

## Purpose

Deploys validated artifacts to the target environment.

Example

```groovy
stage("Deploy"){

    when{

        expression{

            params.DEPLOY

        }

    }

    steps{

        sh """

        helm upgrade --install catalogue charts/catalogue

        """

    }

}
```

The deployment stage is usually protected by approvals, environment selection, or feature flags in production environments.

---

# Complete Production DevSecOps Pipeline

The following example demonstrates a simplified production-ready Jenkins pipeline. It includes the common stages used in enterprise DevSecOps environments.

```groovy
pipeline {

    agent any

    stages {

        stage("Checkout") {
            steps {
                git branch: "main",
                url: "https://github.com/company/catalogue.git"
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Unit Test") {
            steps {
                sh "mvn test"
            }
        }

        stage("Code Quality") {
            steps {
                sh "mvn sonar:sonar"
            }
        }

        stage("Dependency Scan") {
            steps {
                sh "dependency-check.sh --scan . --format HTML"
            }
        }

        stage("Container Scan") {
            steps {
                sh "trivy image catalogue:latest"
            }
        }

        stage("IaC Scan") {
            steps {
                sh "checkov -d terraform/"
            }
        }

        stage("Package") {
            steps {
                sh "docker build -t catalogue:latest ."
            }
        }

        stage("Deploy") {
            steps {
                sh "helm upgrade --install catalogue charts/catalogue"
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }

}
```

---

# Best Practices

## Keep Stages Small

Each stage should have one clear responsibility.

Good

```text
Build

↓

Test

↓

Security Scan

↓

Deploy
```

Poor

```text
Everything inside one stage
```

Small stages improve readability and troubleshooting.

---

## Give Meaningful Stage Names

Good

```text
Compile Application

Unit Test

Docker Build

Deploy to Production
```

Poor

```text
Stage1

Stage2

Stage3
```

---

## Fail Fast

Run inexpensive validations first.

Recommended Order

```text
Checkout

↓

Compile

↓

Unit Test

↓

Code Quality

↓

Security Scan

↓

Docker Build

↓

Deploy
```

Avoid building Docker images before verifying the source code.

---

## Keep Steps Related

Each stage should contain only related tasks.

Example

```text
Build Stage

Compile

Package

Archive Artifact
```

Do not mix deployment commands inside the Build stage.

---

## Separate CI and CD

Continuous Integration

- Checkout
- Build
- Test
- Security Scan
- Package

Continuous Deployment

- Deploy
- Smoke Test
- Notifications

This separation makes pipelines easier to maintain.

---

## Use Declarative Pipeline

Prefer Declarative Pipelines unless advanced scripting is required.

Benefits

- Better readability
- Built-in validation
- Easier maintenance
- Consistent structure

---

# Common Mistakes

## Putting Everything in One Stage

Bad

```groovy
stage("Pipeline"){

    steps{

        sh "mvn clean package"

        sh "mvn test"

        sh "docker build ..."

        sh "helm upgrade ..."

    }

}
```

Good

Create separate stages for Build, Test, Package, and Deploy.

---

## Mixing Responsibilities

Bad

```text
Build Stage

↓

Compile

↓

Deploy
```

Good

```text
Build

↓

Deploy
```

Each stage should have only one responsibility.

---

## Ignoring Failed Stages

Some beginners continue pipeline execution even when important validation stages fail.

Always ensure:

- Build succeeds
- Tests pass
- Security scans pass

before deployment.

---

## Poor Logging

Bad

```groovy
echo "Done"
```

Good

```groovy
echo "Running OWASP Dependency Check"
```

Meaningful logs reduce troubleshooting time.

---

## Not Cleaning Workspace

Leaving old files in the workspace can cause inconsistent builds.

Always use

```groovy
post {

    always {

        cleanWs()

    }

}
```

---

# Troubleshooting Guide

## Build Stage Failed

Possible Causes

- Compilation errors
- Missing dependencies
- Invalid Maven configuration
- Incorrect Java version

Commands

```bash
mvn clean package
```

Check

- Console Output
- Maven logs
- pom.xml

---

## Test Stage Failed

Possible Causes

- Unit test failures
- Integration test failures
- Missing test data

Commands

```bash
mvn test
```

Review test reports before retrying the pipeline.

---

## Security Stage Failed

Possible Causes

- SonarQube Quality Gate failure
- Vulnerable dependency detected
- High severity container vulnerability
- Insecure Terraform configuration

Actions

- Fix code quality issues
- Update vulnerable libraries
- Patch container images
- Correct Infrastructure as Code

Never ignore security scan failures.

---

## Deploy Stage Failed

Possible Causes

- Kubernetes cluster unavailable
- Invalid Helm values
- Missing Kubernetes Secret
- Incorrect image tag

Commands

```bash
kubectl get pods

kubectl describe pod

helm status catalogue
```

---

## Pipeline Stops Unexpectedly

Possible Causes

- Shell command returned non-zero exit code
- Agent disconnected
- Timeout exceeded
- Missing credentials

Review the stage where the failure occurred before restarting.

---

# Common Interview Questions

## 1. What is a Stage in Jenkins?

### Short Answer

A Stage is a logical phase in a Jenkins Pipeline that groups related tasks together.

### Detailed Explanation

Stages divide the pipeline into meaningful phases such as Build, Test, Security Scan, and Deploy. Each stage has its own execution status and appears separately in the Jenkins UI, making it easier to monitor progress and troubleshoot failures.

### Production Example

```text
Build

↓

Test

↓

Deploy
```

### Follow-up Questions

- Can a stage contain multiple steps?
- Can stages run in parallel?

---

## 2. What is a Step?

### Short Answer

A Step is the smallest executable unit inside a Jenkins Pipeline.

### Detailed Explanation

Steps perform actual work, such as running shell commands, printing logs, checking out source code, or archiving artifacts. A stage usually contains one or more steps.

### Production Example

```groovy
steps {

    sh "mvn clean package"

}
```

### Follow-up Questions

- What are commonly used Jenkins steps?
- Can a stage exist without steps?

---

## 3. Difference Between Stage and Step?

### Short Answer

A Stage is a logical grouping of work, while a Step performs an individual task.

### Detailed Explanation

Stages improve organization and visualization. Steps execute the actual commands.

### Production Example

```text
Stage

↓

Build

↓

Step

mvn clean package
```

---

## 4. Why Are Stages Important?

### Short Answer

They improve readability, monitoring, troubleshooting, and maintainability.

### Detailed Explanation

Stages allow Jenkins to identify which phase failed and provide a visual representation of pipeline execution.

---

## 5. What Happens If One Stage Fails?

### Short Answer

By default, the pipeline stops and subsequent stages are skipped.

### Detailed Explanation

Jenkins follows a fail-fast approach unless configured otherwise. This prevents deployments after unsuccessful builds or tests.

### Follow-up Questions

- Can failed stages be retried?
- How do `post` actions behave after a failure?

---

## 6. What Is the Purpose of the `script` Step?

### Short Answer

It allows Groovy scripting inside a Declarative Pipeline.

### Detailed Explanation

The `script` step is used when advanced logic, loops, or conditional statements cannot be expressed using Declarative syntax.

### Production Example

```groovy
script {

    if(params.DEPLOY){

        echo "Deploying"

    }

}
```

---

## 7. Difference Between `echo` and `sh`?

### Short Answer

`echo` prints messages in the Jenkins Pipeline, while `sh` executes shell commands on the agent.

### Detailed Explanation

Use `echo` for logging and `sh` for interacting with the operating system or external tools.

---

## 8. Why Is `cleanWs()` Important?

### Short Answer

It removes workspace files after a build.

### Detailed Explanation

Cleaning the workspace prevents stale files from affecting future builds and helps conserve disk space on Jenkins agents.

---

## 9. Why Use `archiveArtifacts`?

### Short Answer

To preserve build outputs after pipeline completion.

### Detailed Explanation

Artifacts such as JAR files or reports remain available for download and auditing even after the workspace is cleaned.

---

## 10. Why Are Security Stages Included Before Deployment?

### Short Answer

To identify vulnerabilities before software reaches production.

### Detailed Explanation

Running SonarQube, OWASP Dependency Check, Trivy, and Checkov before deployment ensures that code quality issues, vulnerable dependencies, insecure container images, and Infrastructure as Code misconfigurations are detected early.

---

# Production Scenarios

## Scenario 1

A build succeeds, but unit tests fail.

**Outcome**

The pipeline stops at the Test stage, and deployment does not begin.

---

## Scenario 2

SonarQube reports a failed Quality Gate.

**Outcome**

The deployment pipeline is blocked until the quality issues are resolved.

---

## Scenario 3

Trivy detects critical vulnerabilities in the Docker image.

**Outcome**

The image is rejected, preventing an insecure deployment.

---

## Scenario 4

The deployment stage fails because the Kubernetes cluster is unavailable.

**Outcome**

The Build, Test, and Security stages remain successful, allowing the team to focus only on the deployment issue.

---

## Scenario 5

Developers want build artifacts for debugging after a failed deployment.

**Solution**

Use `archiveArtifacts` to preserve generated JAR files.

---

## Scenario 6

The same pipeline deploys to Development, Testing, Staging, and Production.

**Solution**

Use parameters for environment selection while keeping the same stage structure.

---

# Production Case Study

## Case Study: Improving Pipeline Visibility

A company maintained a Jenkins pipeline where all build, testing, scanning, packaging, and deployment commands were placed inside a single stage.

### Problems

- More than 2,000 lines of console output
- Difficult to identify failures
- Long troubleshooting time
- Poor visibility in the Jenkins dashboard

### Solution

The DevOps team redesigned the pipeline using separate stages:

- Checkout
- Build
- Unit Test
- Code Quality
- Dependency Scan
- Container Scan
- IaC Scan
- Package
- Deploy

### Result

- Clear pipeline visualization
- Faster root cause analysis
- Easier maintenance
- Reduced deployment failures
- Improved collaboration between development, QA, and operations teams

---

# Revision Notes

Before moving to the next chapter, make sure you can answer the following questions.

- What is a Stage?
- What is a Step?
- How are Stages and Steps related?
- Why are stages important in CI/CD pipelines?
- What are the most commonly used Jenkins steps?
- What happens when a stage fails?
- Why should security stages run before deployment?
- Why is `cleanWs()` recommended?

---

# Key Takeaways

- A Jenkins Pipeline is divided into logical stages.
- Each stage contains one or more executable steps.
- Stages improve readability, visualization, and troubleshooting.
- Steps perform the actual work, such as running shell commands or checking out code.
- Build, Test, Security, Package, and Deploy are common production stages.
- Security validation should occur before packaging and deployment.
- Small, focused stages make pipelines easier to maintain.
- Proper logging and workspace cleanup improve operational efficiency.
- Well-structured stages help teams quickly identify and resolve failures.