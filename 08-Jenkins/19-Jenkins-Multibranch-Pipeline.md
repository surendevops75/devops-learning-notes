# Jenkins Multibranch Pipeline

## Introduction

In modern software development, multiple developers work on different features simultaneously. Instead of committing directly to the `main` branch, each developer creates a feature branch, implements changes, raises a Pull Request (PR), and finally merges the code after validation.

Managing a separate Jenkins job for every branch quickly becomes difficult as the number of branches increases.

A **Jenkins Multibranch Pipeline** automatically discovers Git branches, creates pipelines for them, executes the appropriate `Jenkinsfile`, and removes pipelines when branches are deleted.

This allows teams to build, test, and validate every branch independently without creating manual Jenkins jobs.

---

# Why Do We Need a Multibranch Pipeline?

Imagine an application with multiple developers.

```text
Developer A

↓

feature-login

----------------------

Developer B

↓

feature-cart

----------------------

Developer C

↓

feature-payment

----------------------

Developer D

↓

bugfix-order
```

Without a Multibranch Pipeline, every branch requires a separate Jenkins job.

```text
feature-login

↓

Jenkins Job 1

-------------------

feature-cart

↓

Jenkins Job 2

-------------------

feature-payment

↓

Jenkins Job 3

-------------------

bugfix-order

↓

Jenkins Job 4
```

As the number of branches grows, maintaining these jobs becomes impractical.

Instead, a single Multibranch Pipeline manages all branches automatically.

```text
GitHub Repository

↓

Jenkins Multibranch Pipeline

↓

Discovers Every Branch

↓

Creates Pipeline Automatically

↓

Runs Jenkinsfile
```

---

# Real Enterprise Scenario

Consider an e-commerce platform.

Several teams are working simultaneously.

```text
Inventory Team

↓

feature-stock

----------------------

Payment Team

↓

feature-payment

----------------------

User Team

↓

feature-profile

----------------------

Cart Team

↓

feature-cart
```

Every branch should be:

- Built
- Tested
- Security scanned
- Dockerized

Only after approval should it be merged into the main branch.

A Multibranch Pipeline automates this process.

---

# Single Pipeline vs Multibranch Pipeline

| Feature | Single Pipeline | Multibranch Pipeline |
|----------|----------------|----------------------|
| Supports Multiple Branches | ❌ | ✅ |
| Automatic Branch Discovery | ❌ | ✅ |
| Automatic Job Creation | ❌ | ✅ |
| Pull Request Builds | Limited | ✅ |
| Automatic Branch Cleanup | ❌ | ✅ |
| Enterprise Ready | Small Projects | Yes |

---

# Multibranch Pipeline Architecture

```text
                    Developers
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
 feature-login     feature-cart     feature-payment
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
                  GitHub Repository
                         │
                         ▼
                GitHub Webhook
                         │
                         ▼
           Jenkins Multibranch Pipeline
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
 Pipeline A        Pipeline B        Pipeline C
```

---

# How a Multibranch Pipeline Works

```text
Developer Push

↓

GitHub

↓

Webhook Trigger

↓

Jenkins Scans Repository

↓

New Branch Found

↓

Pipeline Created

↓

Checkout Code

↓

Execute Jenkinsfile

↓

Publish Results
```

---

# What Happens Internally?

Whenever Jenkins receives a webhook notification:

```text
Webhook

↓

Repository Scan

↓

Read Branch List

↓

Compare Existing Jobs

↓

Create Missing Pipelines

↓

Delete Removed Branch Pipelines

↓

Run Jenkinsfile
```

This process is called **Branch Indexing**.

---

# Branch Discovery

Branch Discovery is the process of identifying newly created Git branches.

Example:

```text
main

↓

Developer Creates

feature-coupon

↓

Push

↓

Jenkins Discovers Branch

↓

Creates New Pipeline
```

No manual intervention is required.

---

# Branch Indexing

Branch Indexing keeps Jenkins synchronized with the Git repository.

During indexing Jenkins:

- Detects new branches
- Detects deleted branches
- Detects Pull Requests
- Updates existing pipelines

```text
Git Repository

↓

Index Repository

↓

Update Pipeline List

↓

Execute Changed Pipelines
```

---

# Automatic Pipeline Creation

Each branch gets its own independent pipeline.

```text
feature-login

↓

Pipeline

---------------------

feature-cart

↓

Pipeline

---------------------

feature-payment

↓

Pipeline
```

Each pipeline has its own:

- Console Logs
- Build History
- Test Reports
- Artifacts
- Status