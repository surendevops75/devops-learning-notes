# Workflow Execution

A GitHub Actions workflow does not execute continuously.

Instead, it follows a well-defined execution lifecycle that begins with a GitHub event and ends when all jobs are completed or a failure occurs.

Understanding workflow execution is important because it helps troubleshoot pipeline failures, optimize execution time, and design enterprise-grade CI/CD pipelines.

---

# Workflow Execution Lifecycle

Every workflow follows the same execution process.

```text
GitHub Event

↓

Workflow Identified

↓

Workflow Validation

↓

Runner Allocation

↓

Job Execution

↓

Step Execution

↓

Action / Script Execution

↓

Results Generated

↓

Workflow Completed
```

Each stage must complete successfully before moving to the next stage.

---

# Step 1 - GitHub Event Occurs

Everything begins with an event.

Examples:

- Push
- Pull Request
- Release
- Tag
- Manual Trigger
- Scheduled Workflow

Example

```text
Developer

↓

Push Code

↓

GitHub Detects Event
```

GitHub immediately checks whether any workflow is configured for that event.

---

# Step 2 - Workflow Discovery

GitHub searches the repository for workflow files.

```text
.github/workflows/
```

Example

```text
.github/workflows/

ci.yml

deploy.yml

terraform.yml
```

GitHub reads every workflow file and checks its trigger conditions.

---

# Step 3 - Event Matching

GitHub compares the incoming event with the workflow trigger.

Example

```yaml
on:

  push:

    branches:

      - main
```

If the event matches:

```text
Push to main

↓

Workflow Starts
```

If the event does not match:

```text
Push to feature branch

↓

Workflow Ignored
```

---

# Step 4 - Workflow Validation

Before execution starts, GitHub validates:

- YAML syntax
- Workflow structure
- Required permissions
- Referenced Actions
- Job definitions

If validation fails, the workflow never starts.

---

# Step 5 - Runner Allocation

Once validation succeeds, GitHub allocates a runner.

Example

```yaml
runs-on: ubuntu-latest
```

Execution flow

```text
Workflow

↓

Allocate Runner

↓

Runner Ready
```

No jobs execute until a runner becomes available.

---

# Step 6 - Job Scheduling

GitHub schedules all jobs.

Example

```text
Workflow

↓

Build

↓

Test

↓

Deploy
```

Jobs may run:

- Sequentially
- Parallel
- Dependency based

depending on workflow configuration.

---

# Sequential Execution

Jobs execute one after another.

```text
Build

↓

Test

↓

Deploy
```

Deploy begins only after Test succeeds.

---

# Parallel Execution

Independent jobs can run simultaneously.

```text
Build

────────────

Lint

────────────

Security Scan
```

This reduces overall pipeline execution time.

---

# Dependency-Based Execution

Using `needs`, jobs wait for required jobs to finish.

```text
Build

↓

Test

↓

Security Scan

↓

Deploy
```

Deployment starts only after all dependent jobs succeed.

---

# Step 7 - Step Execution

Inside each job, GitHub executes every step in order.

```text
Build Job

↓

Checkout

↓

Install Java

↓

Compile

↓

Package
```

Steps cannot execute in parallel within the same job.

---

# Step 8 - Action Execution

Each step performs one of two operations.

## Execute an Action

```yaml
uses: actions/checkout@v4
```

or

## Execute Shell Commands

```yaml
run: mvn clean package
```

GitHub executes each step and records its output.

---

# Step 9 - Result Collection

During execution GitHub continuously records:

- Logs
- Exit Codes
- Artifacts
- Test Reports
- Job Status
- Workflow Status

These results are available in the Actions tab.

---

# Successful Workflow Execution

```text
Event

↓

Workflow

↓

Runner

↓

Jobs

↓

Steps

↓

Success

↓

Workflow Completed
```

Every job completed successfully.

---

# Failed Workflow Execution

```text
Event

↓

Workflow

↓

Runner

↓

Build

↓

Test

↓

Failure

↓

Workflow Stopped
```

Unless configured otherwise, downstream jobs do not execute after a failure.

---

# Enterprise CI Workflow Execution

```text
Developer Push

↓

CI Workflow

↓

Checkout

↓

Build

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

↓

Workflow Completed
```

No deployment occurs in the CI workflow.

---

# Enterprise CD Workflow Execution

```text
workflow_dispatch

↓

Select Environment

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

Business Approval

↓

Deploy Production

↓

Health Check

↓

Deployment Completed
```

Deployment progresses through each environment only after successful validation.

---

# Enterprise Production Workflow Execution

Based on your deployment process.

```text
Developer

↓

Merge Pull Request

↓

CI Workflow

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

UAT Deployment

↓

End-to-End Tests

↓

workflow_dispatch

↓

Enter JIRA Ticket

↓

Validate JIRA

↓

Validate Deployment Window

↓

Validate Commit SHA

↓

Production Approval

↓

Deploy Production

↓

Developer Sanity Check

↓

Monitoring

↓

Release Complete
```

This is a common execution flow in enterprise organizations where production deployments require validations beyond the CI pipeline.

---

# Workflow Status

Each workflow ends with one of the following statuses:

| Status | Description |
|---------|-------------|
| Success | All jobs completed successfully |
| Failure | One or more jobs failed |
| Cancelled | Workflow was manually cancelled |
| Skipped | Workflow or job was skipped due to conditions |

---

# What Happens When a Job Fails?

Suppose the Build job fails.

```text
Build ❌

↓

Test

↓

Deploy
```

By default:

- Test does not execute.
- Deploy does not execute.
- Workflow is marked as failed.

This behavior prevents deploying unverified code.

---

# Best Practices

- Keep CI and CD as separate workflows.
- Run independent jobs in parallel where possible.
- Use job dependencies for deployment stages.
- Validate workflows before merging.
- Keep workflows short and focused.
- Monitor workflow execution from the Actions tab.
- Use artifacts instead of rebuilding in deployment workflows.

---

# Common Mistakes

- Assuming steps within a job run in parallel.
- Running deployments directly after every push.
- Ignoring workflow validation errors.
- Combining unrelated jobs into one workflow.
- Not using dependencies for deployment stages.
- Rebuilding artifacts during deployment instead of reusing published artifacts.

---

# Summary

Workflow execution in GitHub Actions follows a predictable lifecycle:

1. GitHub detects an event.
2. Matching workflows are identified.
3. Workflow syntax is validated.
4. A runner is allocated.
5. Jobs are scheduled.
6. Steps execute sequentially within each job.
7. Results are collected and displayed.

Understanding this lifecycle helps design efficient, reliable, and production-ready CI/CD pipelines.

---

# Interview Questions

## Basic

1. What triggers a GitHub Actions workflow?
2. What happens after a workflow is triggered?
3. When is a runner allocated?
4. Do steps execute in parallel?
5. Where can you view workflow logs?

---

## Intermediate

1. Explain the workflow execution lifecycle.
2. What happens if a job fails?
3. How are jobs scheduled?
4. Explain sequential vs parallel execution.
5. Why are deployment workflows usually separated from CI workflows?

---

## Advanced

1. Design the execution flow for an enterprise CI/CD pipeline with separate CI, QA, SIT, UAT, and Production workflows.
2. Explain how GitHub Actions schedules jobs and executes steps in large enterprise repositories.
3. A production deployment workflow failed after the QA deployment but before Production. Describe how you would analyze the workflow execution, identify the failure point, and safely resume or redeploy without rebuilding the application.