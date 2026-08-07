# Workflow Syntax

A GitHub Actions workflow is written using **YAML (Yet Another Markup Language)**.

The workflow syntax defines:

- Workflow Name
- Trigger Events
- Permissions
- Environment Variables
- Jobs
- Steps
- Conditions
- Outputs
- Defaults
- Concurrency
- Timeout
- Strategy

Every workflow is stored inside the following directory.

```text
.github/workflows/
```

Example:

```text
.github/

└── workflows/

    ci.yml

    deploy.yml

    release.yml
```

GitHub automatically detects and executes workflows stored in this location.

---

# Basic Workflow Syntax

```yaml
name:

on:

permissions:

env:

jobs:
```

These are the top-level keywords of a workflow.

---

# Workflow Skeleton

```yaml
name: Java CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4
```

Although this is a minimal workflow, it contains all the essential building blocks.

---

# Workflow Hierarchy

```text
Workflow

↓

Top-Level Keywords

↓

Jobs

↓

Steps

↓

Actions / Commands
```

Everything inside GitHub Actions follows this hierarchy.

---

# Top-Level Keywords

The most commonly used top-level keywords are:

| Keyword | Purpose |
|----------|----------|
| name | Workflow display name |
| on | Event trigger |
| permissions | Workflow permissions |
| env | Global environment variables |
| defaults | Default shell and working directory |
| concurrency | Prevent multiple workflow executions |
| jobs | Collection of jobs |

These keywords are defined only once at the workflow level.

---

# name

The **name** keyword gives a friendly name to the workflow.

Example

```yaml
name: Spring Boot CI Pipeline
```

Benefits

- Easy identification
- Better workflow history
- Clear pipeline visibility

---

# on

The **on** keyword specifies when the workflow should execute.

Example

```yaml
on:
  push:
```

Multiple events are also supported.

```yaml
on:
  push:

  pull_request:

  workflow_dispatch:
```

GitHub starts the workflow whenever one of these events occurs.

---

# permissions

Permissions define what the workflow is allowed to do.

Example

```yaml
permissions:
  contents: read
```

Enterprise best practice is to grant only the minimum permissions required.

---

# env

The **env** keyword defines global environment variables.

Example

```yaml
env:
  APP_NAME: catalogue

  AWS_REGION: us-east-1
```

Every job can access these variables.

---

# defaults

Defines default settings for all jobs.

Example

```yaml
defaults:
  run:
    shell: bash
```

This avoids repeating the same configuration in every step.

---

# concurrency

Concurrency prevents multiple executions of the same workflow.

This directly relates to your note:

> At a time only one build should run.

Example

```yaml
concurrency:
  group: production
```

Enterprise example

```text
Developer A

↓

Deploy Production

↓

Lock

────────────

Developer B

↓

Waiting

↓

Deployment Starts After Lock Release
```

Concurrency prevents accidental simultaneous deployments.

---

# jobs

The **jobs** section contains all the work performed by the workflow.

Example

```yaml
jobs:

  build:

  test:

  deploy:
```

Each job performs one logical task.

---

# runs-on

Every job requires a runner.

Example

```yaml
runs-on: ubuntu-latest
```

Other examples

```yaml
runs-on: windows-latest

runs-on: macos-latest

runs-on: self-hosted
```

GitHub allocates the requested runner before executing the job.

---

# steps

Steps are individual tasks executed inside a job.

Example

```yaml
steps:

- name: Checkout

- name: Build

- name: Test
```

Steps execute sequentially.

---

# name (Step Name)

Each step can have a descriptive name.

Example

```yaml
- name: Build Java Application
```

Meaningful names make workflow logs easier to understand.

---

# uses

Runs a reusable Action.

Example

```yaml
uses: actions/checkout@v4
```

Examples

- Checkout repository
- Setup Java
- Setup NodeJS
- Setup Terraform
- Login to Docker

---

# run

Executes shell commands.

Example

```yaml
run: mvn clean package
```

Unlike **uses**, **run** executes commands directly on the runner.

---

# with

Passes inputs to an Action.

Example

```yaml
uses: actions/setup-java@v4

with:

  java-version: '21'
```

Many Actions require configuration through **with**.

---

# if

Executes a job or step only if a condition is satisfied.

Example

```yaml
if: github.ref == 'refs/heads/main'
```

This prevents unnecessary execution.

---

# needs

Defines job dependencies.

Example

```yaml
Build

↓

Test

↓

Deploy
```

Deploy starts only after Test completes successfully.

---

# timeout-minutes

Limits job execution time.

Example

```yaml
timeout-minutes: 20
```

If execution exceeds the limit, GitHub automatically stops the job.

---

# strategy

Used for matrix builds.

Example

```yaml
strategy:

  matrix:
```

Allows the same workflow to execute against multiple environments.

---

# Enterprise CI Workflow Syntax

```text
Push

↓

Build Job

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Push Image

↓

Publish Artifact
```

Each stage is represented by one or more jobs.

---

# Enterprise Production Deployment Syntax

Based on your production deployment process.

```text
workflow_dispatch

↓

Environment Input

↓

JIRA Ticket Input

↓

Validate JIRA

↓

Validate Commit SHA

↓

Validate Deployment Window

↓

Production Approval

↓

Deploy

↓

Smoke Tests

↓

Developer Sanity

↓

Complete
```

The workflow syntax supports implementing each of these stages.

---

# Best Practices

- Give workflows meaningful names.
- Keep jobs independent.
- Use reusable Actions.
- Avoid hardcoding values.
- Use concurrency for production deployments.
- Set timeout values.
- Use conditions instead of duplicate workflows.
- Grant minimum permissions.

---

# Common Mistakes

- Writing everything in one job.
- No timeout configuration.
- Running multiple production deployments simultaneously.
- Hardcoding secrets.
- Giving unnecessary permissions.
- Not using reusable Actions.
- Poor workflow naming.

---

# Summary

Workflow syntax defines how GitHub Actions executes automation.

The most important keywords include:

- name
- on
- permissions
- env
- jobs
- steps
- uses
- run
- if
- needs
- concurrency
- timeout-minutes

Understanding these keywords is the foundation for writing enterprise-grade GitHub Actions workflows.

---

# Interview Questions

## Basic

1. What is workflow syntax?
2. What does the `on` keyword do?
3. What is the purpose of `jobs`?
4. Difference between `run` and `uses`.
5. What is `runs-on`?

---

## Intermediate

1. Explain the purpose of `needs`.
2. Why is `concurrency` important?
3. What is the use of `permissions`?
4. Why should `timeout-minutes` be configured?
5. Explain the workflow hierarchy.

---

## Advanced

1. Design a production deployment workflow using `workflow_dispatch`, `concurrency`, approvals, and validation steps.
2. Explain how workflow syntax supports enterprise CI/CD pipelines.
3. A company wants only one production deployment at a time. How would you implement this using GitHub Actions workflow syntax?