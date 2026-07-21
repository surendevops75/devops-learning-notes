# Jenkins Plugins

## Introduction

Jenkins is designed with a modular architecture. The core Jenkins installation provides only basic functionality, while additional capabilities are added through **plugins**.

Plugins enable Jenkins to integrate with source code repositories, cloud providers, container platforms, security tools, notification systems, testing frameworks, and deployment platforms.

Without plugins, Jenkins would only be able to execute simple build jobs.

---

# What is a Plugin?

A plugin is an extension that adds new functionality to Jenkins.

Think of Jenkins Core as an operating system.

Plugins are similar to applications installed on that operating system.

```text
               Jenkins Core
                    │
    ┌───────────────┼────────────────┐
    ▼               ▼                ▼
 GitHub Plugin  Docker Plugin   Kubernetes Plugin
    ▼               ▼                ▼
 More Features  More Features  More Features
```

---

# Why Do We Need Plugins?

Suppose a developer pushes code to GitHub.

Without plugins:

```text
Developer Push

↓

GitHub

↓

Jenkins

↓

Cannot Connect
```

Jenkins doesn't know how to communicate with GitHub.

---

After installing the GitHub Plugin:

```text
Developer Push

↓

GitHub

↓

GitHub Plugin

↓

Jenkins

↓

Pipeline Starts
```

---

# How Plugins Work

Plugins extend Jenkins by adding:

- New build steps
- Pipeline syntax
- Credentials support
- Cloud integrations
- UI components
- Notifications
- Security scanning
- Deployment capabilities

```text
Jenkins Core

↓

Install Plugin

↓

New Feature Available
```

---

# Common Jenkins Plugins

| Plugin | Purpose |
|---------|---------|
| Git Plugin | Clone Git repositories |
| GitHub Plugin | GitHub integration |
| Pipeline Plugin | Pipeline support |
| Credentials Plugin | Secret management |
| Docker Pipeline Plugin | Docker builds |
| Kubernetes Plugin | Dynamic Kubernetes agents |
| SonarQube Scanner Plugin | Static code analysis |
| OWASP Dependency Check Plugin | Dependency scanning |
| HTML Publisher Plugin | Publish HTML reports |
| JUnit Plugin | Publish test reports |
| Workspace Cleanup Plugin | Clean workspaces |
| Email Extension Plugin | Email notifications |
| Blue Ocean Plugin | Modern pipeline UI |

---

# Plugin Categories

```text
Source Control

↓

Git

↓

GitHub

---------------------

Container

↓

Docker

↓

Kubernetes

---------------------

Security

↓

SonarQube

↓

OWASP

↓

Trivy (CLI Integration)

---------------------

Reporting

↓

JUnit

↓

HTML Publisher

---------------------

Notifications

↓

Email

↓

Slack

↓

Teams
```

---

# Plugin Architecture

```text
                 Jenkins Core
                      │
      ┌───────────────┼─────────────────┐
      ▼               ▼                 ▼
 Git Plugins     Pipeline Plugins   Security Plugins
      │               │                 │
      └───────────────┼─────────────────┘
                      ▼
                 Jenkins Pipeline
```

---

# Plugins Used in Our Production Pipeline

Based on our Enterprise CI/CD pipeline, the following plugins are commonly used:

```text
Git Plugin

↓

Checkout Source Code

-----------------------

GitHub Plugin

↓

Webhook Integration

-----------------------

Pipeline Plugin

↓

Declarative Pipeline

-----------------------

Credentials Plugin

↓

GitHub PAT

AWS Credentials

Sonar Token

-----------------------

SonarQube Scanner Plugin

↓

Code Quality Analysis

-----------------------

OWASP Dependency Check Plugin

↓

Dependency Scanning

-----------------------

Docker Pipeline Plugin

↓

Docker Build

-----------------------

HTML Publisher Plugin

↓

Publish OWASP Reports

-----------------------

JUnit Plugin

↓

Publish Unit Test Results

-----------------------

Workspace Cleanup Plugin

↓

Clean Workspace
```

---

# Production Plugin Flow

```text
Developer Push

↓

GitHub Plugin

↓

Pipeline Plugin

↓

Git Plugin

↓

Credentials Plugin

↓

Shared Library

↓

SonarQube Plugin

↓

OWASP Plugin

↓

Docker Plugin

↓

JUnit Plugin

↓

HTML Publisher

↓

Workspace Cleanup
```