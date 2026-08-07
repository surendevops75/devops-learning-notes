# GitHub Actions Core Components

GitHub Actions is built using several core components that work together to automate software development workflows.

Understanding these components is essential before writing workflows because every GitHub Actions pipeline is created using these building blocks.

The core components are:

- Workflow
- Events
- Jobs
- Steps
- Actions
- Runners

These components work together to execute an automated CI/CD pipeline.

---

# GitHub Actions Component Architecture

```text
Developer

↓

GitHub Event

↓

Workflow

↓

Runner

↓

Job

↓

Steps

↓

Actions

↓

Deployment
```

Every workflow execution follows this sequence.

---

# Workflow

A **Workflow** is an automated process defined in a YAML file.

A workflow contains one or more jobs that perform various tasks such as building, testing, scanning, or deploying an application.

Workflow files are stored inside the repository.

```text
.github/workflows/
```

Example:

```text
.github/workflows/build.yml
```

A repository can contain multiple workflows.

For example:

```text
.github/workflows/

build.yml

deploy.yml

terraform.yml

security-scan.yml
```

Each workflow is independent.

---

# Workflow Example

```text
Developer Push

↓

Build Workflow

↓

Build

↓

Test

↓

Docker Build

↓

Push Image
```

---

# Events

An **Event** is an activity that occurs inside a GitHub repository.

Events trigger workflows automatically.

Examples include:

- Push to a branch
- Pull Request
- Merge Pull Request
- Release Creation
- Tag Creation
- Issue Creation
- Manual Trigger
- Scheduled Execution

Example:

```text
Developer Pushes Code

↓

Push Event

↓

Workflow Starts
```

Without an event, a workflow does not execute.

---

# Common Events

| Event | Purpose |
|---------|----------|
| push | Trigger after code push |
| pull_request | Trigger for PR activities |
| workflow_dispatch | Manual execution |
| schedule | Run on a schedule |
| release | Trigger on GitHub release |
| workflow_call | Reusable workflow |
| repository_dispatch | External API trigger |

These events will be covered in detail later.

---

# Jobs

A **Job** is a collection of steps that execute on the same runner.

Each job performs a logical unit of work.

Examples:

- Build
- Test
- Scan
- Deploy

Example workflow:

```text
Workflow

↓

Build Job

↓

Test Job

↓

Deploy Job
```

Jobs may execute:

- Sequentially
- In Parallel
- Based on dependencies

---

# Job Example

```text
Build Job

↓

Checkout Code

↓

Install Dependencies

↓

Compile Application

↓

Package Application
```

This entire sequence represents one job.

---

# Steps

A **Step** is an individual task inside a job.

Every job consists of multiple steps.

Example:

```text
Build Job

↓

Checkout Code

↓

Install Java

↓

Build Application

↓

Upload Artifact
```

Each step performs only one task.

Steps execute in order.

---

# Actions

An **Action** is a reusable unit of code that performs a predefined task.

Think of Actions as reusable building blocks.

Examples include:

- Checkout source code
- Install Java
- Configure Terraform
- Login to Docker Registry
- Upload artifacts

Example:

```yaml
- uses: actions/checkout@v4

- uses: actions/setup-java@v4
```

Instead of writing shell scripts repeatedly, developers can reuse Actions published in the GitHub Marketplace.

---

# Why Use Actions?

Without reusable Actions:

```text
Workflow

↓

Write Shell Script

↓

Install Software

↓

Configure Environment
```

With reusable Actions:

```text
Workflow

↓

Use Existing Action

↓

Ready to Use
```

This reduces development time and improves consistency.

---

# Runners

A **Runner** is the machine that executes workflow jobs.

Whenever a workflow starts, GitHub assigns a runner to execute the jobs.

Two runner types are available:

- GitHub Hosted Runner
- Self Hosted Runner

Example:

```text
Workflow

↓

Runner

↓

Execute Jobs
```

The runner performs all build and deployment tasks.

---

# GitHub Hosted Runner

GitHub provides managed virtual machines.

Examples:

- Ubuntu
- Windows
- macOS

Benefits:

- No infrastructure management
- Ready to use
- Automatically maintained
- Fast setup

---

# Self Hosted Runner

Organizations can configure their own servers to execute workflows.

Examples:

- Linux Server
- Virtual Machine
- Kubernetes Pod
- Cloud Instance

Benefits:

- Complete control
- Custom software
- Internal network access
- No GitHub-hosted runtime limits

---

# Complete Component Relationship

```text
Developer

↓

Push Code

↓

Event

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

Application Build
```

Every GitHub Actions workflow follows this hierarchy.

---

# Enterprise CI Workflow

```text
Developer

↓

Push Code

↓

Push Event

↓

Build Workflow

↓

GitHub Hosted Runner

↓

Checkout

↓

Maven Build

↓

Unit Tests

↓

SonarQube Scan

↓

Trivy Scan

↓

Docker Build

↓

Push Image

↓

Publish Artifact
```

Each stage is implemented using jobs, steps, and reusable actions.

---

# Enterprise CD Workflow

```text
workflow_dispatch

↓

Select Environment

↓

Self Hosted Runner

↓

Download Artifact

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

Production Approval

↓

Deploy Production

↓

Developer Sanity Check
```

Notice how different GitHub Actions components work together to automate a production deployment.

---

# Component Hierarchy

```text
Repository

↓

Workflow

↓

Jobs

↓

Steps

↓

Actions

↓

Execution
```

This hierarchy is fundamental to GitHub Actions.

---

# Best Practices

- Keep workflows focused on a single purpose.
- Separate CI and CD workflows.
- Break large workflows into multiple jobs.
- Use reusable Actions whenever possible.
- Minimize custom shell scripting.
- Use self-hosted runners only when required.
- Organize workflows by functionality.
- Use descriptive names for workflows, jobs, and steps.

---

# Common Mistakes

- Confusing workflows with jobs.
- Treating Actions as workflows.
- Placing unrelated tasks in one job.
- Writing long shell scripts instead of using Actions.
- Running deployment jobs on GitHub-hosted runners when private network access is required.
- Creating one massive workflow instead of multiple focused workflows.

---

# Summary

GitHub Actions consists of six primary components:

- **Workflow** defines the automation.
- **Events** trigger the workflow.
- **Jobs** organize related work.
- **Steps** perform individual tasks.
- **Actions** provide reusable functionality.
- **Runners** execute the jobs.

Understanding these components is essential before learning workflow syntax and writing production-grade pipelines.

---

# Interview Questions

## Basic

1. What is a workflow?
2. What is an event?
3. What is a job?
4. What is a step?
5. What is an Action?
6. What is a runner?

---

## Intermediate

1. Explain the relationship between workflows, jobs, and steps.
2. Why do we use reusable Actions?
3. What is the difference between GitHub-hosted and self-hosted runners?
4. Can a workflow contain multiple jobs?
5. How are workflows triggered?

---

## Advanced

1. Design an enterprise GitHub Actions workflow that separates CI and CD while using reusable Actions and self-hosted runners for production deployments.
2. Explain how workflows, jobs, steps, Actions, and runners interact during an enterprise deployment pipeline.
3. A company wants to build, test, scan, package, and deploy multiple microservices using GitHub Actions. Explain how you would organize workflows, jobs, runners, and reusable Actions for scalability and maintainability.