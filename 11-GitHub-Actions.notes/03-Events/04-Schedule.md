# Schedule Event

The `schedule` event allows a GitHub Actions workflow to run automatically at a defined time or interval using a POSIX cron expression.

Unlike `push` and `pull_request`, the `schedule` event does not depend on a developer changing code.

It is useful for recurring automation such as:

- Security scans
- Dependency checks
- Infrastructure validation
- Database maintenance
- Cleanup jobs
- Reports
- Health checks
- Scheduled deployments
- Repository maintenance
- Operational automation

---

# Basic Syntax

```yaml
name: Scheduled Workflow

on:
  schedule:
    - cron: '0 2 * * *'

jobs:
  scheduled-job:
    runs-on: ubuntu-latest

    steps:
      - name: Run Scheduled Task
        run: echo "Scheduled workflow executed"
```

The workflow runs according to the configured cron schedule.

---

# Cron Format

GitHub Actions uses POSIX cron syntax.

```text
┌──────────── minute
│ ┌────────── hour
│ │ ┌──────── day of month
│ │ │ ┌────── month
│ │ │ │ ┌──── day of week
│ │ │ │ │
* * * * *
```

The five fields are:

```text
Minute
Hour
Day of Month
Month
Day of Week
```

---

# Common Cron Examples

## Every Day at Midnight

```yaml
cron: '0 0 * * *'
```

---

## Every Day at 2 AM

```yaml
cron: '0 2 * * *'
```

---

## Every Monday at 6 AM

```yaml
cron: '0 6 * * 1'
```

---

## Every Hour

```yaml
cron: '0 * * * *'
```

---

## Every 15 Minutes

```yaml
cron: '*/15 * * * *'
```

---

# Important Time Zone Consideration

Scheduled workflows use **UTC** for cron scheduling.

Therefore, when defining a production schedule, convert the desired business or operational time to UTC.

Example:

```text
Business Requirement

↓

Run at 8:00 AM local time

↓

Convert to UTC

↓

Configure cron
```

Always verify the intended execution time before using a schedule for production operations.

---

# Execution Flow

```text
Cron Schedule

↓

GitHub Scheduler

↓

Workflow Trigger

↓

Runner

↓

Job

↓

Steps

↓

Result
```

No developer push is required.

---

# Scheduled Security Scan

A common enterprise use case is running security scans every night.

```text
Every Night

↓

GitHub Actions

↓

Checkout Code

↓

Dependency Scan

↓

Trivy Scan

↓

Security Report

↓

Notify Security Team
```

Example:

```yaml
name: Nightly Security Scan

on:
  schedule:
    - cron: '0 2 * * *'

jobs:
  security:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Security Scan
        run: echo "Running security scan"
```

---

# Scheduled Dependency Check

Scheduled workflows can periodically check dependencies for new vulnerabilities.

```text
Nightly Schedule

↓

Dependency Check

↓

Critical Vulnerability?

        /       \
      YES        NO
       |          |
       ↓          ↓
    Alert       Success
```

This allows security checks to continue even when developers have not pushed new code.

---

# Production Use Case - Dependabot Alert Check

Your notes include a GitHub API call for checking critical Dependabot alerts.

A scheduled workflow can periodically call the GitHub API and inspect open critical alerts.

Conceptual flow:

```text
Scheduled Workflow

↓

GitHub API

↓

Dependabot Alerts

↓

Filter Critical + Open

↓

Alerts Found?

      /       \
    YES        NO
     |          |
     ↓          ↓
  Notify      Success
```

Example API request:

```bash
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open"
```

The response can be processed with `jq`.

Example:

```bash
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open" \
  | jq '. | length'
```

The resulting number represents the number of matching alerts returned by the API.

---

# Scheduled Infrastructure Validation

Terraform infrastructure can also be checked periodically.

```text
Nightly Schedule

↓

Checkout Terraform

↓

terraform init

↓

terraform validate

↓

terraform plan

↓

Review Changes

↓

Notify Team
```

Example:

```yaml
name: Scheduled Terraform Validation

on:
  schedule:
    - cron: '0 3 * * *'

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan
```

For production infrastructure, credentials, permissions, state management, and approval requirements must be handled carefully.

---

# Scheduled Kubernetes Health Check

A scheduled workflow can also perform operational checks.

```text
Scheduled Workflow

↓

Authenticate

↓

Connect to Cluster

↓

Check Nodes

↓

Check Pods

↓

Check Deployments

↓

Check Failed Workloads

↓

Generate Report
```

Example:

```text
Every 6 Hours

↓

Kubernetes Health Check

↓

Unhealthy Workloads?

      /       \
    YES        NO
     |          |
     ↓          ↓
   Alert       Success
```

---

# Scheduled Cleanup Workflow

Temporary resources and artifacts can sometimes be cleaned automatically.

```text
Weekly Schedule

↓

Identify Old Resources

↓

Validate Cleanup Criteria

↓

Delete Approved Resources

↓

Generate Report
```

Production cleanup workflows should include strict safeguards to avoid deleting active resources.

---

# Scheduled Reporting

A scheduled workflow can generate daily or weekly reports.

```text
Every Morning

↓

Collect CI/CD Results

↓

Collect Security Results

↓

Collect Deployment Results

↓

Generate Report

↓

Send Notification
```

Possible reports include:

- Failed builds
- Security findings
- Deployment status
- Test results
- Infrastructure changes

---

# Multiple Scheduled Jobs

A workflow can contain multiple jobs.

```text
Scheduled Workflow

        |
 ┌──────┼──────┐
 ↓      ↓      ↓
Build  Scan  Report
```

Independent jobs can execute in parallel.

---

# Schedule + Manual Trigger

A powerful enterprise pattern is combining `schedule` with `workflow_dispatch`.

```yaml
name: Security Scan

on:
  schedule:
    - cron: '0 2 * * *'

  workflow_dispatch:

jobs:
  scan:
    runs-on: ubuntu-latest

    steps:
      - name: Run Scan
        run: echo "Running security scan"
```

Now the same workflow can run:

```text
Automatically

↓

Scheduled Execution
```

or:

```text
Manually

↓

workflow_dispatch
```

This is useful when an operations team needs to run a scheduled task immediately without waiting for the next scheduled execution.

---

# Production Emergency Execution

Suppose a nightly security scan normally runs at 2 AM.

A critical vulnerability is announced at 10 AM.

Instead of waiting for the next schedule:

```text
Critical Vulnerability

↓

Security Team

↓

Manual workflow_dispatch

↓

Run Security Scan Immediately
```

The same workflow supports both scheduled and manual execution.

---

# Schedule + Push

A workflow can also respond to multiple events.

```yaml
on:
  push:
    branches:
      - main

  schedule:
    - cron: '0 2 * * *'
```

Execution:

```text
Push to main
       \
        \
         → Same Workflow
        /
Nightly Schedule
```

This can be useful when the same validation needs to run both after code changes and periodically.

---

# Production Deployment and Scheduling

Scheduled production deployments require additional caution.

For example:

```text
Scheduled Trigger

↓

Validate Change Request

↓

Validate Deployment Window

↓

Validate Commit

↓

Check Approvals

↓

Deploy

↓

Smoke Tests
```

A schedule should **not** bypass production controls.

The schedule only starts the workflow. The workflow must still validate the required deployment conditions.

---

# Enterprise Scheduled Production Workflow

```text
Scheduled Trigger

↓

Read Deployment Configuration

↓

Check Current Time

↓

Check JIRA Change Request

↓

Check Approval Status

↓

Check Deployment Window

↓

Check Commit SHA

↓

Check CI Status

↓

Check Security Results

↓

Production Approval

↓

Deploy

↓

Smoke Tests

↓

Developer Sanity Check

↓

Monitoring
```

This aligns with the enterprise change-management process from your notes.

---

# Schedule with Concurrency

Scheduled jobs can accidentally overlap if one execution takes longer than the schedule interval.

For example:

```text
02:00

↓

Deployment Starts

↓

Still Running

↓

03:00

↓

Next Scheduled Execution
```

Concurrency can prevent overlapping executions.

```yaml
concurrency:
  group: scheduled-production
  cancel-in-progress: false
```

Execution becomes:

```text
Job A

↓

Running

↓

Job B

↓

Waiting

↓

Job A Complete

↓

Job B Starts
```

For production operations, this prevents simultaneous executions of the same deployment process.

---

# Schedule with Environment Protection

Production workflows can use GitHub Environments.

```text
Schedule

↓

Production Environment

↓

Required Approval

↓

Deploy
```

This ensures a scheduled trigger does not automatically bypass required human approval.

---

# Production Example

```yaml
name: Scheduled Production Validation

on:
  schedule:
    - cron: '0 1 * * *'

  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: production-validation
  cancel-in-progress: false

jobs:

  validate:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate Commit
        run: echo "Checking deployment commit"

      - name: Check Security Results
        run: echo "Checking security scan results"

      - name: Check Deployment Window
        run: echo "Checking deployment window"

  production:
    needs: validate

    runs-on: self-hosted

    environment: production

    steps:

      - name: Deploy
        run: echo "Production deployment"

      - name: Smoke Test
        run: echo "Running smoke tests"
```

This demonstrates a controlled scheduled production workflow.

---

# Scheduled Workflow Security

Scheduled workflows can execute without a developer being present.

Therefore, security is important.

Use:

- Least-privilege permissions
- Secure secrets
- Protected environments
- Restricted self-hosted runners
- Approval gates for sensitive operations
- Strong validation before destructive actions

Never hardcode credentials inside scheduled workflows.

---

# Scheduled Workflow Reliability

Production scheduled workflows should be designed to handle failures.

Recommended flow:

```text
Schedule

↓

Validation

↓

Execution

↓

Health Check

↓

Success?

   /      \
 YES       NO
  |         |
  ↓         ↓
Complete   Alert
```

A failed scheduled job should generate an actionable notification rather than silently failing.

---

# Production Troubleshooting

## Scenario 1 - Scheduled Workflow Did Not Run

Check:

```text
1. Workflow file exists.
2. Workflow is enabled.
3. cron syntax is valid.
4. Expected UTC time is correct.
5. Repository and workflow configuration are valid.
6. The workflow has not been disabled due to repository inactivity or policy.
```

---

## Scenario 2 - Workflow Runs at the Wrong Time

Check:

```text
Business Time

↓

UTC Conversion

↓

Cron Expression

↓

Actual Execution
```

Remember that cron schedules use UTC.

---

## Scenario 3 - Scheduled Deployment Runs Twice

Check:

```text
1. Multiple cron entries
2. Push trigger
3. Manual trigger
4. Duplicate workflows
5. Overlapping scheduled executions
```

Use concurrency where appropriate.

---

## Scenario 4 - Scheduled Security Scan Fails

Check:

```text
Workflow

↓

Runner

↓

Authentication

↓

GitHub API

↓

Token Permissions

↓

API Response

↓

jq Processing
```

For API-based scans, verify both authentication and response handling.

---

# Best Practices

- Use schedules for recurring automation.
- Verify cron expressions carefully.
- Convert business time to UTC.
- Combine `schedule` with `workflow_dispatch` when manual execution is useful.
- Use concurrency to prevent overlapping executions.
- Use protected environments for sensitive deployments.
- Use least-privilege permissions.
- Alert on scheduled workflow failures.
- Keep scheduled workflows idempotent where possible.
- Validate change requests before scheduled production operations.

---

# Common Mistakes

- Assuming cron uses local time.
- Running destructive operations without safeguards.
- Hardcoding credentials.
- Allowing scheduled production deployments to bypass approvals.
- Ignoring overlapping executions.
- Not monitoring scheduled workflow failures.
- Using overly broad permissions.
- Forgetting that scheduled workflows can run even when no developer has changed the repository.

---

# Summary

The `schedule` event allows GitHub Actions workflows to execute automatically according to a cron expression.

It is useful for:

- Security scans
- Dependency checks
- Infrastructure validation
- Kubernetes health checks
- Cleanup
- Reporting
- Operational automation
- Controlled scheduled deployments

For enterprise environments, scheduled workflows should be combined with proper security, validation, concurrency, monitoring, and approval controls.

A strong production pattern is:

```text
Schedule

↓

Validate

↓

Check Security

↓

Check Change Request

↓

Check Deployment Window

↓

Approval

↓

Execute

↓

Smoke Test

↓

Monitor
```

---

# Interview Questions

## Basic

1. What is the `schedule` event?
2. What syntax does GitHub Actions use for scheduled workflows?
3. What is a cron expression?
4. What are the five cron fields?
5. What time zone is used for GitHub Actions schedules?

## Intermediate

6. How do you run a workflow every day?
7. How do you run a workflow every Monday?
8. Why would you combine `schedule` with `workflow_dispatch`?
9. How can scheduled workflows overlap?
10. How can concurrency prevent overlapping executions?

## Advanced

11. Design a nightly enterprise security scanning workflow using GitHub Actions.
12. Design a scheduled Dependabot alert-checking workflow using the GitHub REST API and `jq`.
13. Design a scheduled production deployment workflow that validates a JIRA change request, deployment window, commit SHA, CI status, security results, and approval before deployment.
14. A scheduled workflow is executing at the wrong time. Explain how you would troubleshoot the cron expression and UTC conversion.
15. A scheduled deployment starts while a previous deployment is still running. Explain how you would prevent concurrent production executions safely.