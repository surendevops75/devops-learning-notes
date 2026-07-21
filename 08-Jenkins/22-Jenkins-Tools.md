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

