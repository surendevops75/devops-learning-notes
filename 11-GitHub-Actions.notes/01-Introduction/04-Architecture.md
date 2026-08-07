# GitHub Actions Architecture

To effectively use GitHub Actions in enterprise environments, it is important to understand how the different components interact during workflow execution.

Unlike traditional CI/CD tools that require dedicated controllers and agents, GitHub Actions is an event-driven platform where workflows are automatically triggered by GitHub events and executed on runners.

Understanding this architecture helps in designing scalable, secure, and maintainable CI/CD pipelines.

---

# High-Level Architecture

The basic GitHub Actions architecture consists of the following components:

- GitHub Repository
- GitHub Events
- Workflow
- Runner
- Jobs
- Steps
- Actions
- Deployment Target

Each component plays a specific role in the execution of a workflow.

---

# Architecture Diagram

```text
Developer

↓

GitHub Repository

↓

GitHub Event

↓

Workflow (.github/workflows)

↓

Runner

↓

Jobs

↓

Steps

↓

Actions / Scripts

↓

Deployment Target
```

---

# Step-by-Step Workflow Execution

When a developer pushes code to GitHub, the following sequence occurs:

### Step 1

Developer pushes code.

```text
Developer

↓

Push Code
```

---

### Step 2

GitHub detects an event.

Examples:

- push
- pull_request
- workflow_dispatch
- release
- schedule

```text
Push

↓

GitHub Event
```

---

### Step 3

GitHub searches for workflow files.

Workflow files are stored inside:

```text
.github/workflows/
```

Example:

```text
.github/workflows/build.yml
```

---

### Step 4

GitHub evaluates the trigger.

Example:

```yaml
on:
  push:
    branches:
      - main
```

If the event matches, GitHub starts the workflow.

---

### Step 5

A Runner is allocated.

Depending on configuration:

- GitHub Hosted Runner
- Self Hosted Runner

```text
Workflow

↓

Runner
```

---

### Step 6

Jobs begin execution.

A workflow may contain one or more jobs.

```text
Workflow

↓

Job 1

Job 2

Job 3
```

Jobs may execute:

- Sequentially
- In Parallel
- Based on dependencies

---

### Step 7

Each job executes multiple steps.

Example:

```text
Checkout

↓

Install Dependencies

↓

Build

↓

Test

↓

Deploy
```

Each step performs one specific task.

---

### Step 8

Steps execute Actions or Shell Commands.

Examples:

```yaml
uses: actions/checkout@v4
```

or

```yaml
run: mvn clean package
```

GitHub executes them in sequence.

---

### Step 9

Results are reported back to GitHub.

GitHub displays:

- Logs
- Artifacts
- Test Results
- Job Status
- Workflow Status

---

# Complete Execution Flow

```text
Developer

↓

Push Code

↓

GitHub Repository

↓

Event Trigger

↓

Workflow

↓

Runner

↓

Job

↓

Step

↓

Action

↓

Result
```

This entire process happens automatically.

---

# Core Components

GitHub Actions architecture consists of several building blocks.

| Component | Purpose |
|------------|----------|
| Repository | Stores source code |
| Event | Starts workflow |
| Workflow | Defines automation |
| Runner | Executes workflow |
| Job | Collection of steps |
| Step | Individual task |
| Action | Reusable functionality |

---

# Runner Architecture

A Runner is the machine responsible for executing workflow jobs.

```text
Workflow

↓

Runner

↓

Execute Jobs

↓

Return Results
```

A runner can be:

- Virtual Machine
- Physical Server
- Cloud Instance
- Kubernetes Pod

---

# Job Execution

Each job executes independently.

```text
Job

↓

Step 1

↓

Step 2

↓

Step 3

↓

Completed
```

Steps within the same job always execute sequentially unless scripting introduces parallelism.

---

# Multiple Jobs

A workflow may contain several jobs.

```text
Workflow

↓

Build

────────────

Test

────────────

Deploy
```

Jobs may run:

- One after another
- Simultaneously
- Based on dependencies

---

# Enterprise CI Architecture

A common enterprise CI workflow.

```text
Developer

↓

Push

↓

GitHub

↓

Workflow

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Package

↓

Artifact Repository
```

This workflow validates every code change before deployment.

---

# Enterprise CD Architecture

Deployment pipeline.

```text
Artifact

↓

QA

↓

Smoke Test

↓

SIT

↓

Integration Test

↓

UAT

↓

Approval

↓

Production
```

Each environment acts as a quality gate.

---

# Enterprise GitOps Architecture

GitHub Actions often works together with ArgoCD.

```text
Developer

↓

GitHub Repository

↓

GitHub Actions

↓

Build Docker Image

↓

Push to Container Registry

↓

Update GitOps Repository

↓

ArgoCD Detects Change

↓

Deploy Kubernetes
```

GitHub Actions performs CI, while ArgoCD continuously delivers changes to Kubernetes.

---

# Enterprise Multi-Environment Architecture

```text
Feature Branch

↓

CI Pipeline

↓

QA

↓

SIT

↓

UAT

↓

Production Approval

↓

Production
```

Each deployment must successfully complete before progressing to the next environment.

---

# Enterprise Production Deployment Workflow

This reflects a real production release process.

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge to Main

↓

GitHub Actions Triggered

↓

Build

↓

Unit Tests

↓

Security Scan

↓

Docker Build

↓

Push Image

↓

Deploy QA

↓

Smoke Tests

↓

Deploy SIT

↓

Integration Tests

↓

Deploy UAT

↓

End-to-End Tests

↓

Manual Approval

↓

Deploy Production

↓

Developer Sanity Check

↓

Monitoring

↓

Release Complete
```

This is the type of workflow commonly used in enterprise DevOps teams to ensure quality, security, and controlled releases.

---

# Architecture Best Practices

- Keep workflows modular.
- Separate CI and CD workflows.
- Use reusable workflows for common tasks.
- Protect production deployments with approvals.
- Store secrets securely.
- Use self-hosted runners only when necessary.
- Monitor workflow execution and failures.
- Keep workflow files under version control.

---

# Common Mistakes

- Putting all stages into one huge workflow.
- Running deployment directly after every commit.
- Mixing CI and CD responsibilities.
- Using self-hosted runners without proper security.
- Not separating environments.
- Ignoring workflow dependencies.
- Hardcoding secrets in workflows.

---

# Summary

GitHub Actions follows an event-driven architecture where GitHub events trigger workflows that execute on runners. Each workflow is divided into jobs, each job contains steps, and each step performs a specific task using actions or shell commands.

In enterprise environments, GitHub Actions integrates with tools such as Docker, Terraform, Kubernetes, ArgoCD, SonarQube, and artifact repositories to automate the complete software delivery lifecycle.

---

# Interview Questions

## Basic

1. What is the architecture of GitHub Actions?
2. What triggers a workflow?
3. Where are workflow files stored?
4. What is a runner?
5. What is the difference between a job and a step?

---

## Intermediate

1. Explain the execution flow of a GitHub Actions workflow.
2. How do multiple jobs execute in a workflow?
3. Why do enterprises separate CI and CD workflows?
4. Explain the role of runners in GitHub Actions.
5. How does GitHub Actions integrate with ArgoCD?

---

## Advanced

1. Design an enterprise GitHub Actions architecture for a Kubernetes-based microservices platform with QA, SIT, UAT, and Production environments.
2. Explain how GitHub Actions integrates with Docker, Terraform, artifact repositories, and GitOps tools in a production environment.
3. A company wants to support thousands of builds per day using GitHub Actions. Design a scalable architecture including runners, workflow organization, deployment stages, and production approval gates.