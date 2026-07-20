# GitHub to Jenkins Integration

## Introduction

A CI/CD pipeline starts when developers push code to GitHub.

Jenkins automatically detects the change, builds the application, performs security scans, and deploys it.

---

Without Integration

```text
Developer Push

↓

Login to Jenkins

↓

Click Build Now
```

---

With Integration

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline Starts Automatically
```

---

# Why Integrate GitHub with Jenkins?

GitHub integration enables Continuous Integration.

Every code change is automatically validated.

---

Benefits

```text
Automatic Builds

Faster Feedback

Reduced Manual Work

Continuous Integration

Early Bug Detection
```

---

# Integration Architecture

```text
Developer
     │
     ▼
Git Commit
     │
     ▼
Git Push
     │
     ▼
GitHub Repository
     │
     ▼
Webhook
     │
     ▼
Jenkins
     │
     ▼
Pipeline Execution
```

---

# Prerequisites

Before integrating GitHub with Jenkins, ensure the following are available.

```text
GitHub Account

Git Repository

Jenkins Server

Git Plugin

GitHub Plugin

Network Connectivity
```

---

# Integration Workflow

```text
Developer

↓

Commit Code

↓

Push Code

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout Source

↓

Execute Pipeline
```

---

# Authentication Methods

Jenkins can authenticate with GitHub using:

```text
Username & Password (Deprecated)

Personal Access Token (Recommended)

SSH Key
```

---

For production environments,

```text
Personal Access Token (PAT)
```

is recommended.

---

# Personal Access Token (PAT)

A Personal Access Token is used instead of a GitHub password.

---

Benefits

```text
More Secure

Easy to Revoke

Granular Permissions
```

---

Workflow

```text
GitHub

↓

Generate PAT

↓

Store in Jenkins Credentials

↓

Access Repository
```

---

# Configure Jenkins Credentials

Navigate to

```text
Manage Jenkins

↓

Credentials

↓

Global Credentials

↓

Add Credentials
```

---

Store

```text
GitHub Username

Personal Access Token
```

---

Result

```text
Jenkins Can Access
Private Repository
```

---

# Configure GitHub Repository

Repository

↓

Settings

↓

Webhooks

↓

Add Webhook

---

Payload URL

```text
http://jenkins-ip:8080/github-webhook/
```

---

Content Type

```text
application/json
```

---

Events

```text
Just the push event
```

---

# Webhook Flow

```text
Developer Push

↓

GitHub

↓

Webhook Payload

↓

Jenkins

↓

Pipeline Triggered
```

---

# Configure Jenkins Job

Pipeline Job

↓

Source Code Management

↓

Git

↓

Repository URL

↓

Credentials

↓

Branch

---

Example

```text
Repository

↓

https://github.com/company/cart.git
```

---

Branch

```text
main
```

---

# Build Triggers

Jenkins supports multiple build triggers.

```text
GitHub Webhook

Poll SCM

Manual Build

Cron Schedule
```

---

Recommended

```text
GitHub Webhook
```

---

# Poll SCM vs Webhook

| Feature | Poll SCM | Webhook |
|----------|-----------|----------|
| Trigger | Periodic | Instant |
| Performance | Lower | Better |
| Resource Usage | High | Low |
| Recommended | No | Yes |

---

Workflow

Poll SCM

```text
Every 5 Minutes

↓

Check Repository

↓

Build If Changed
```

---

Webhook

```text
Push Code

↓

Immediate Build
```

---

# Webhook Payload

GitHub sends an HTTP POST request.

---

Payload Contains

```text
Repository

Branch

Commit ID

Author

Modified Files
```

---

Jenkins uses this information to start the pipeline.

---

# Branch-Based Build

```text
feature/login

↓

Build
```

---

```text
main

↓

Build

↓

Deploy
```

---

Branch filtering can be implemented using the Jenkins `when` directive.

---