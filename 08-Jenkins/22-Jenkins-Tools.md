# Jenkins Tools

## Introduction

Most Jenkins pipelines depend on external tools to build, test, package, and deploy applications.

Jenkins itself does not include Java, Maven, Gradle, Node.js, Git, or Docker. Instead, it manages these tools through the **Global Tool Configuration**.

This allows administrators to configure tool versions once and reuse them across all pipelines.

---

# Why Do We Need Jenkins Tools?

Without configured tools:

```text
Developer Push

↓

Jenkins Pipeline

↓

Build Step

↓

Java Not Found

↓

Build Failed
```

---

After configuring tools:

```text
Developer Push

↓

Jenkins Pipeline

↓

Configured JDK

↓

Configured Maven

↓

Build Successful
```

---

# What Are Jenkins Tools?

A Jenkins Tool is an external software package managed by Jenkins.

Examples include:

- JDK
- Git
- Maven
- Gradle
- Node.js
- Docker (CLI)
- Sonar Scanner

These tools can be installed automatically or referenced from existing installations.

---

# Global Tool Configuration

All tools are managed centrally.

```text
Dashboard

↓

Manage Jenkins

↓

Tools

↓

Configure Tool

↓

Save
```

Once configured, every pipeline can use the same tool definition.

---

# Common Jenkins Tools

| Tool | Purpose |
|------|---------|
| Git | Clone source code |
| JDK | Compile Java applications |
| Maven | Build Java projects |
| Gradle | Build automation |
| Node.js | Build JavaScript applications |
| Docker | Build container images |
| Sonar Scanner | Static code analysis |

---

# Tool Architecture

```text
              Jenkins

                 │

      Global Tool Configuration

                 │

 ┌────────┬────────┬─────────┬──────────┐
 │        │        │         │          │
 ▼        ▼        ▼         ▼          ▼
 JDK     Git     Maven    Node.js    Docker
```

---

# How Jenkins Uses Tools

```text
Pipeline Starts

↓

Load Tool

↓

Add Tool to PATH

↓

Execute Commands

↓

Continue Pipeline
```

---

# Example: Using Git

```text
Pipeline

↓

Git Tool

↓

Clone Repository

↓

Workspace Ready
```

---

# Example: Using JDK

```text
Source Code

↓

JDK

↓

Compile

↓

Class Files
```

---

# Example: Using Maven

```text
pom.xml

↓

Maven

↓

Download Dependencies

↓

Compile

↓

Test

↓

Package

↓

JAR File
```

---

# Example: Using Node.js

```text
package.json

↓

Node.js

↓

npm install

↓

npm test

↓

npm build
```

---

# Example: Using Docker

```text
Application

↓

Docker CLI

↓

Build Image

↓

Push to Registry
```

---

# Production Tool Flow

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Git

↓

JDK

↓

Maven

↓

Sonar Scanner

↓

OWASP

↓

Docker

↓

Trivy

↓

Amazon ECR

↓

Amazon EKS
```

---

# Auto Installation vs Manual Installation

## Auto Installation

```text
Jenkins

↓

Download Tool

↓

Install Automatically
```

Advantages

- Easy setup
- Consistent versions
- Good for small environments

---

## Manual Installation

```text
Administrator

↓

Install Tool

↓

Configure Jenkins

↓

Pipeline Uses Tool
```

Advantages

- Better control
- Enterprise standardization
- Offline environments

---

# Tool Labels

Different agents may use different tool versions.

Example

```text
Linux Agent

↓

JDK 17

↓

Maven 3.9

--------------------

Windows Agent

↓

JDK 21

↓

Maven 3.8
```

Jenkins automatically selects the configured tool for the executing agent.

---

# Production Example

Java Microservice

```text
Git

↓

JDK 17

↓

Maven

↓

JUnit

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker

↓

Trivy

↓

Amazon ECR

↓

Amazon EKS
```
---

# Tool Installation

Jenkins can use tools in two ways:

1. Automatic Installation
2. Manual Installation

The installation method depends on your organization's requirements.

---

# Automatic Tool Installation

Jenkins downloads and installs the required tool automatically.

Supported tools include:

- JDK
- Maven
- Gradle
- Node.js

---

## Workflow

```text
Pipeline Starts

↓

Tool Not Available

↓

Jenkins Downloads Tool

↓

Installs Tool

↓

Adds to PATH

↓

Pipeline Continues
```

---

## Advantages

- Quick setup
- Consistent versions
- No manual installation
- Easy for development environments

---

## Disadvantages

- Requires Internet access
- Less control over versions
- Not suitable for highly secured environments

---

# Manual Tool Installation

In enterprise environments, administrators install tools manually on Jenkins agents.

Jenkins only references their installation paths.

---

## Workflow

```text
Administrator

↓

Install Tool

↓

Configure Jenkins

↓

Pipeline Uses Tool
```

---

## Advantages

- Full control
- Enterprise standardization
- Offline installation
- Better compliance

---

## Disadvantages

- Manual maintenance
- Version upgrades require administrator effort

---

# Global Tool Configuration

All tools are configured from a single location.

```text
Dashboard

↓

Manage Jenkins

↓

Tools

↓

Configure Tool

↓

Save
```

---

## Configured Tools

```text
Git

↓

JDK

↓

Maven

↓

Gradle

↓

Node.js

↓

Sonar Scanner
```

---

# Tool Naming

Each configured tool has a unique name.

Example

```text
JDK

↓

jdk17

------------------------

Maven

↓

maven-3.9

------------------------

Git

↓

git-default
```

The pipeline refers to these names rather than installation paths.

---

# Using Tools in Jenkins Pipeline

Declarative Pipelines use the `tools` block.

## Example

```groovy
pipeline {

    agent any

    tools {

        jdk 'jdk17'
        maven 'maven-3.9'

    }

    stages {

        stage('Build') {

            steps {

                sh 'java -version'
                sh 'mvn clean package'

            }

        }

    }

}
```

---

## Pipeline Execution

```text
Pipeline Starts

↓

Load JDK

↓

Load Maven

↓

Update PATH

↓

Execute Build
```

---

# Multiple Tool Example

```groovy
pipeline {

    agent any

    tools {

        jdk 'jdk17'
        maven 'maven-3.9'
        nodejs 'node18'

    }

    stages {

        stage('Build Java') {

            steps {

                sh 'mvn clean package'

            }

        }

        stage('Build UI') {

            steps {

                sh 'npm install'
                sh 'npm run build'

            }

        }

    }

}
```

---

# Production Example

Microservices often contain multiple technologies.

```text
Java Service

↓

JDK

↓

Maven

-----------------------

Frontend

↓

Node.js

↓

npm Build

-----------------------

Docker

↓

Container Image
```

---

# Tool Resolution

When a pipeline starts,

Jenkins performs:

```text
Read Jenkinsfile

↓

Find Tool

↓

Locate Installation

↓

Update PATH

↓

Execute Commands
```

---

# Automatic PATH Configuration

Jenkins automatically updates the PATH variable.

Instead of

```bash
/usr/lib/jvm/java-17/bin/java
```

the pipeline simply uses

```bash
java
```

Similarly,

Instead of

```bash
/opt/apache-maven/bin/mvn
```

the pipeline uses

```bash
mvn
```

---

# Tool Versions

Different projects may require different versions.

Example

```text
Project A

↓

JDK 17

↓

Maven 3.9

------------------------

Project B

↓

JDK 21

↓

Gradle 8

------------------------

Project C

↓

Node.js 20
```

Jenkins selects the appropriate version based on the Jenkinsfile.

---

# Enterprise Tool Architecture

```text
                    Jenkins Controller
                           │
          Global Tool Configuration
                           │
      ┌─────────┬──────────┬──────────┐
      ▼         ▼          ▼          ▼
    JDK 17   Maven 3.9   Node 20   Git
                           │
                           ▼
                 Jenkins Pipeline
                           │
            Load Configured Tools
                           │
                           ▼
                 Build Application
```

---

# Production CI/CD Example

```text
Developer Push

↓

GitHub

↓

Jenkins

↓

Load Git

↓

Checkout Code

↓

Load JDK

↓

Load Maven

↓

Compile

↓

Load Sonar Scanner

↓

Static Analysis

↓

OWASP Scan

↓

Docker Build

↓

Trivy Scan

↓

Push to Amazon ECR

↓

Deploy to Amazon EKS
```

---

# Manual vs Automatic Installation

```text
| Feature | Automatic | Manual |
|---------|-----------|---------|
| Internet Required | Yes | No |
| Enterprise Friendly | Limited | Yes |
| Version Control | Basic | Complete |
| Offline Support | No | Yes |
| Setup Time | Fast | Moderate |
| Recommended for Production | No | Yes |
```
---


