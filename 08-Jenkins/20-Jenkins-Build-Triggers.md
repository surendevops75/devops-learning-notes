# Jenkins Build Triggers

## Introduction

A Jenkins pipeline does not execute automatically unless it is triggered.

A **Build Trigger** defines the event or condition that starts a Jenkins job. In production environments, pipelines are typically triggered by source code changes, pull requests, scheduled jobs, upstream pipelines, or external systems.

Choosing the correct trigger improves automation, reduces manual effort, and ensures CI/CD pipelines execute at the right time.

---

# Why Do We Need Build Triggers?

Without build triggers:

```text
Developer Push

↓

Wait

↓

Login to Jenkins

↓

Click Build Now

↓

Pipeline Starts
```

Problems:

- Manual process
- Delayed feedback
- Human error
- Poor automation

---

With Build Triggers

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Pipeline Starts Automatically
```

Benefits:

- Fully automated
- Faster feedback
- No manual intervention
- Better CI/CD

---

# What Can Trigger a Jenkins Pipeline?

Common build triggers include:

```text
Developer Push

↓

GitHub Webhook

-----------------------

Pull Request

↓

Webhook

-----------------------

Cron Schedule

↓

Nightly Build

-----------------------

Upstream Job

↓

Downstream Job

-----------------------

Manual Build

↓

Build Now

-----------------------

Remote API Trigger

↓

External System
```

---

# Types of Build Triggers

Jenkins supports multiple trigger mechanisms.

| Trigger | Use Case |
|----------|----------|
| Manual Trigger | Testing or debugging |
| GitHub Webhook | Continuous Integration |
| Poll SCM | Periodically check Git |
| Cron Schedule | Nightly builds |
| Upstream Job | CI/CD chaining |
| Remote Trigger | External integrations |

---

# Manual Trigger

The simplest trigger is manually clicking **Build Now**.

```text
User

↓

Build Now

↓

Pipeline Executes
```

Suitable for:

- Testing
- Debugging
- Initial setup

Not recommended for production CI/CD.

---

# GitHub Webhook Trigger

The most common enterprise trigger.

Workflow:

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

Advantages:

- Immediate execution
- No polling
- Lower server load
- Faster developer feedback

---

# Poll SCM

Instead of GitHub notifying Jenkins,

Jenkins periodically checks GitHub.

```text
Jenkins

↓

Every 5 Minutes

↓

Check Repository

↓

Changes Found?

↓

Run Pipeline
```

Disadvantages:

- Delayed execution
- Unnecessary repository scans
- Higher Jenkins load

---

# Cron Trigger

Cron schedules allow pipelines to execute automatically at specific times.

Examples:

```text
Every Midnight

↓

Backup

---------------------

Every Sunday

↓

Security Scan

---------------------

Every Morning

↓

Dependency Update
```

Common cron syntax:

```text
H 2 * * *

↓

Run Daily Around 2 AM
```

---

# Upstream Trigger

A pipeline can start automatically after another pipeline finishes.

```text
Build Pipeline

↓

Docker Image

↓

Deploy Pipeline
```

Useful for separating build and deployment jobs.

---

# Remote Trigger

External systems can invoke Jenkins using its REST API.

Example:

```text
Monitoring Tool

↓

REST API

↓

Jenkins

↓

Pipeline
```

Common integrations:

- ServiceNow
- Jira
- GitHub
- Custom Applications

---

# Production Architecture

```text
                Developer
                     │
                     ▼
                GitHub Push
                     │
                     ▼
              GitHub Webhook
                     │
                     ▼
      Jenkins Multibranch Pipeline
                     │
                     ▼
          Load Shared Library
                     │
                     ▼
           javaEKSPipeline()
                     │
                     ▼
                Build & Test
                     │
                     ▼
                SonarQube
                     │
                     ▼
        OWASP Dependency Check
                     │
                     ▼
               Docker Build
                     │
                     ▼
                Trivy Scan
                     │
                     ▼
             Push Image to ECR
                     │
                     ▼
               EKSDeploy()
                     │
                     ▼
                Deploy to EKS
```