# GitHub Workflow API

The GitHub REST API provides endpoints for working with GitHub Actions workflows and workflow runs.

The Workflow API can be used to automate:

```text
List Workflows
Get Workflow Details
List Workflow Runs
Get Workflow Run Details
Trigger Workflows
Re-run Workflows
Cancel Workflow Runs
Download Workflow Logs
Manage Workflow Usage
```

For DevOps automation, this is useful when GitHub Actions needs to interact with other workflows, repositories, deployment systems, or external automation platforms.

---

# Workflow vs Workflow Run

These are different concepts.

## Workflow

The workflow is the YAML definition.

Example:

```text
.github/workflows/ci.yml
```

It defines:

```text
Triggers
Jobs
Steps
Permissions
Runners
```

## Workflow Run

A workflow run is one execution of that workflow.

Example:

```text
Workflow:
CI

Run:
#152
```

Think:

```text
Workflow
   |
   ├── Run #150
   ├── Run #151
   ├── Run #152
   └── Run #153
```

---

# Workflow API Architecture

```text
Client
   |
   ↓
GitHub REST API
   |
   ↓
Repository
   |
   ↓
GitHub Actions
   |
   ├── Workflows
   |
   └── Workflow Runs
```

---

# Workflow API Base URL

GitHub REST API:

```text
https://api.github.com
```

Workflow endpoint pattern:

```text
/repos/OWNER/REPOSITORY/actions/workflows
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows"
```

---

# List Workflows

The workflows endpoint returns workflows configured in the repository.

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows"
```

---

# Extract Workflow Names

Using `jq`:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows" \
  | jq '.workflows[] | {
      id,
      name,
      path,
      state
    }'
```

---

# Workflow Information

A workflow can have information such as:

```text
Workflow ID
Name
Path
State
Created Date
Updated Date
```

Example:

```text
ID:
123456

Name:
CI

Path:
.github/workflows/ci.yml
```

---

# Workflow ID

Every workflow has an identifier.

Example:

```text
Workflow ID:
123456
```

The workflow ID can be used in subsequent API requests.

For example:

```text
Workflow
   |
   ↓
Workflow ID
   |
   ↓
Specific Workflow API Operation
```

---

# Workflow File Name

Instead of a workflow ID, some workflow API endpoints can use the workflow file name.

Example:

```text
ci.yml
```

Conceptually:

```text
/repos/OWNER/REPO/actions/workflows/ci.yml
```

Using the workflow file name can be convenient when the file is known.

---

# Get a Specific Workflow

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/ci.yml"
```

---

# Workflow State

The workflow response includes state information.

Conceptually:

```text
active
disabled
```

Depending on the API response and endpoint, additional state information may be available.

---

# Workflow Runs

A workflow can execute many times.

Example:

```text
CI Workflow
     |
 ┌───┼──────────────┐
 ↓   ↓              ↓
Run 1 Run 2        Run 3
```

The Workflow Runs API allows automation to inspect those executions.

---

# List Workflow Runs

Endpoint pattern:

```text
/repos/OWNER/REPOSITORY/actions/runs
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs"
```

---

# Filter Runs by Workflow

Conceptually:

```text
/repos/OWNER/REPO/actions/workflows/WORKFLOW_ID/runs
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/ci.yml/runs"
```

---

# Workflow Run Information

A workflow run can provide:

```text
Run ID
Run Number
Workflow Name
Status
Conclusion
Branch
Commit SHA
Event
Created Time
Updated Time
HTML URL
```

This is extremely useful for deployment traceability.

---

# Status vs Conclusion

These are different.

### Status

Describes the current execution state.

Examples:

```text
queued
in_progress
completed
```

### Conclusion

Describes the result after completion.

Examples can include:

```text
success
failure
cancelled
skipped
timed_out
```

Conceptually:

```text
Run
 |
 ↓
Status
 |
 ├── queued
 ├── in_progress
 └── completed
          |
          ↓
      Conclusion
```

---

# Get Run Information

Endpoint pattern:

```text
/repos/OWNER/REPO/actions/runs/RUN_ID
```

Example:

```bash
RUN_ID="123456789"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}"
```

---

# Extract Run Status

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}" \
  | jq '{
      status,
      conclusion,
      head_branch,
      head_sha,
      run_number
    }'
```

---

# Get Latest Workflow Run

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/ci.yml/runs?per_page=1" \
  | jq '.workflow_runs[0] | {
      id,
      run_number,
      status,
      conclusion,
      head_sha
    }'
```

This is useful for external automation that needs to determine the latest workflow result.

---

# Workflow Run Filtering

Workflow run APIs can support filtering by parameters such as:

```text
branch
event
status
created
actor
```

Use the filters appropriate to the API endpoint and your automation requirement.

---

# Example: Branch Filtering

Conceptually:

```text
?branch=main
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs?branch=main"
```

---

# Example: Status Filtering

Conceptually:

```text
?status=completed
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs?status=completed"
```

---

# Workflow API in GitHub Actions

A GitHub Actions workflow can call the API itself.

Example:

```yaml
name: Workflow API Check

on:
  workflow_dispatch:

permissions:
  actions: read
  contents: read

jobs:

  check:

    runs-on: ubuntu-latest

    steps:

      - name: List Workflow Runs
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          curl --fail-with-body -sS \
            -H "Authorization: Bearer ${GH_TOKEN}" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}/actions/runs"
```

---

# Actions Permission

For Actions API operations, the workflow token must have the required Actions-related permission.

Example:

```yaml
permissions:
  actions: read
  contents: read
```

For write operations, configure the required write permission only when necessary.

Always use the smallest permission set that supports the operation.

---

# Triggering a Workflow

A workflow can be configured to support:

```yaml
on:
  workflow_dispatch:
```

This creates a manually dispatchable workflow.

The REST API can then trigger that workflow using the supported workflow dispatch endpoint.

---

# Workflow Dispatch Architecture

```text
External System
      |
      ↓
GitHub REST API
      |
      ↓
workflow_dispatch
      |
      ↓
GitHub Actions
      |
      ↓
Workflow Run
```

---

# Workflow Dispatch Endpoint

Conceptually:

```text
POST
/repos/OWNER/REPO/actions/workflows/WORKFLOW_ID/dispatches
```

Example:

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  -d '{
    "ref": "main"
  }'
```

The workflow must support `workflow_dispatch`.

---

# Workflow Dispatch with Inputs

Workflow:

```yaml
on:

  workflow_dispatch:

    inputs:

      environment:
        description: Deployment environment
        required: true
        type: choice
        options:
          - qa
          - uat
          - production

      image:
        description: Image digest
        required: true
        type: string
```

API request:

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  -d '{
    "ref": "main",
    "inputs": {
      "environment": "production",
      "image": "sha256:example"
    }
  }'
```

---

# Important Production Rule

Never allow an external caller to freely choose:

```text
production
```

without appropriate authorization.

Use:

```text
Trusted Source
+
Validation
+
GitHub Environment
+
Required Reviewers
```

for production deployments.

---

# Workflow Dispatch Example

```text
Jenkins
   |
   ↓
GitHub API
   |
   ↓
workflow_dispatch
   |
   ↓
Production Workflow
   |
   ↓
Environment Approval
   |
   ↓
Deployment
```

---

# Cross-Repository Workflow Trigger

Example:

```text
Repository A
    |
    ↓
Build Complete
    |
    ↓
GitHub API
    |
    ↓
Repository B
    |
    ↓
Deployment Workflow
```

This is useful when:

```text
Application Repository
```

and:

```text
GitOps Repository
```

are separated.

---

# Example Cross-Repository Architecture

```text
Application Repository
        |
        ↓
Build + Security
        |
        ↓
ECR
        |
        ↓
GitHub API
        |
        ↓
GitOps Repository
        |
        ↓
Deployment Workflow
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

The authentication and permissions must be designed specifically for the cross-repository operation.

---

# Re-running a Workflow Run

The API supports re-running workflow runs.

Conceptually:

```text
POST
/repos/OWNER/REPO/actions/runs/RUN_ID/rerun
```

Example:

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/rerun"
```

---

# Re-run Use Case

Suppose:

```text
Workflow
   ↓
Temporary Failure
```

For example:

```text
Transient Network Failure
```

An authorized automation system may re-run the workflow.

But do not blindly retry every production deployment.

---

# Re-run Failed Jobs

GitHub also provides API capabilities for re-running failed jobs where supported.

Conceptually:

```text
Workflow Run
   |
   ↓
Failed Job
   |
   ↓
Re-run
```

This can be useful for transient failures.

---

# Re-run Decision

Use caution:

```text
Transient failure
→ Possible retry

Code failure
→ Fix code first

Security failure
→ Do not blindly retry

Deployment failure
→ Investigate before retry
```

---

# Cancel Workflow Run

A running workflow can be cancelled through the API.

Conceptually:

```text
POST
/repos/OWNER/REPO/actions/runs/RUN_ID/cancel
```

Example:

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/cancel"
```

---

# Cancel Use Case

Example:

```text
Deployment A
      |
      ↓
Problem Detected
      |
      ↓
Cancel Run
      |
      ↓
Prevent Further Actions
```

Cancellation should be used carefully when the workflow may already have performed partial changes.

---

# Production Deployment Cancellation

Suppose:

```text
Deploy
 ↓
50% Complete
 ↓
Problem
```

Cancelling the workflow does not automatically mean:

```text
Rollback
```

These are different operations.

```text
Cancel
→ Stop workflow execution

Rollback
→ Restore previous application state
```

---

# Workflow Logs

The API can be used to access workflow logs.

Conceptually:

```text
Workflow Run
      |
      ↓
Logs
      |
      ↓
Download
```

Logs are useful for:

```text
Troubleshooting
Auditing
Incident Investigation
Failure Analysis
```

---

# Download Workflow Logs

Conceptually:

```text
GET
/repos/OWNER/REPO/actions/runs/RUN_ID/logs
```

Example:

```bash
curl -L \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/logs" \
  -o workflow-logs.zip
```

The endpoint returns a downloadable archive.

---

# Job Logs

Workflow runs contain jobs.

Conceptually:

```text
Workflow Run
   |
   ├── Build Job
   ├── Test Job
   ├── Security Job
   └── Deploy Job
```

The API can be used to inspect jobs associated with a run.

---

# List Jobs for a Workflow Run

Endpoint pattern:

```text
/repos/OWNER/REPO/actions/runs/RUN_ID/jobs
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/jobs"
```

---

# Extract Job Status

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/jobs" \
  | jq '.jobs[] | {
      id,
      name,
      status,
      conclusion
    }'
```

---

# Job-Level Troubleshooting

Example:

```text
Workflow Run
     |
     ↓
Jobs
     |
 ┌───┼──────────┐
 ↓   ↓          ↓
Build Test    Deploy
             |
             ↓
           Failed
```

The API can help identify which job failed.

---

# Workflow Failure Investigation

Production automation can collect:

```text
Run ID
Run Number
Commit SHA
Branch
Workflow
Job
Status
Conclusion
Logs
```

Then generate:

```text
Failure Report
```

---

# Example Failure Report

```text
Workflow: Production Deployment

Run: #152

Commit:
8a92f31

Environment:
production

Failed Job:
deploy

Conclusion:
failure
```

This can be integrated into incident automation.

---

# Workflow API + Monitoring

External monitoring can query:

```text
Workflow Runs
```

and detect:

```text
Repeated Failures
Long Running Workflows
Failed Deployments
```

Architecture:

```text
Monitoring
    |
    ↓
GitHub Workflow API
    |
    ↓
Workflow Runs
    |
    ↓
Alert
```

---

# Workflow API + Incident Management

```text
Workflow Failure
       |
       ↓
GitHub API
       |
       ↓
Failure Information
       |
       ↓
Incident System
```

Possible integrations:

```text
JIRA
PagerDuty
Slack
Email
Internal Incident Platform
```

---

# Workflow API + JIRA

Example:

```text
Production Workflow
       |
       ↓
Failure
       |
       ↓
GitHub API
       |
       ↓
Collect Run / Job
       |
       ↓
Create JIRA Incident
```

Include useful information:

```text
Repository
Workflow
Run ID
Commit SHA
Branch
Failed Job
Run URL
```

Avoid including secrets or unnecessary sensitive logs.

---

# Workflow API + Deployment Tracking

Production deployment:

```text
Commit
 ↓
Workflow Run
 ↓
Environment
 ↓
Deployment
```

The API can help retrieve the workflow-side metadata.

---

# Workflow API + Commit SHA

Example:

```bash
jq '.workflow_runs[] | {
  run_id: .id,
  commit: .head_sha,
  branch: .head_branch,
  status,
  conclusion
}'
```

This allows automation to correlate:

```text
Commit
```

with:

```text
Workflow Result
```

---

# Build Once, Deploy Many

A strong production model:

```text
Commit
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Artifact
 ↓
QA
 ↓
UAT
 ↓
Production
```

Do not rebuild a different artifact for every environment unless your architecture specifically requires it.

The Workflow API can help track which workflow run produced a specific artifact or deployment event.

---

# Workflow API + Release Traceability

Example:

```text
Commit:
8a92f31

Workflow:
Build #150

Artifact:
catalogue-image

Image:
sha256:...

Environment:
production

Deployment:
#152
```

This creates strong release traceability.

---

# Workflow API + GitOps

A common architecture:

```text
Application Repository
        |
        ↓
GitHub Actions
        |
        ↓
Build / Security
        |
        ↓
ECR
        |
        ↓
GitHub API
        |
        ↓
GitOps Repository
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

The Workflow API can be used to trigger or inspect workflows involved in this process.

---

# Workflow API + ArgoCD

Example:

```text
GitHub Actions
     |
     ↓
Update GitOps
     |
     ↓
ArgoCD
     |
     ↓
EKS
```

If a GitOps workflow is used:

```text
Application Workflow
     |
     ↓
Workflow Dispatch API
     |
     ↓
GitOps Workflow
     |
     ↓
ArgoCD
```

---

# Production Workflow Trigger Architecture

```text
Developer
    |
    ↓
Application Repository
    |
    ↓
CI
    |
    ↓
Security
    |
    ↓
Build Image
    |
    ↓
ECR
    |
    ↓
GitHub Workflow API
    |
    ↓
GitOps Workflow
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# Workflow API Security

Workflow APIs can perform powerful operations.

Potentially dangerous actions include:

```text
Trigger Production Workflow
Re-run Deployment
Cancel Deployment
Modify Workflow State
```

Therefore:

```text
Authentication
+
Authorization
+
Environment Protection
```

are essential.

---

# Prevent Unauthorized Workflow Dispatch

Do not allow:

```text
Untrusted User
    ↓
Production Workflow
```

without appropriate controls.

Better:

```text
Trusted Automation
       |
       ↓
Authenticated API
       |
       ↓
Validated Inputs
       |
       ↓
Production Environment
       |
       ↓
Approval
```

---

# Workflow Inputs

Example:

```yaml
on:

  workflow_dispatch:

    inputs:

      service:
        required: true
        type: choice
        options:
          - catalogue
          - orders
          - payment

      environment:
        required: true
        type: choice
        options:
          - qa
          - uat
          - production
```

Using `choice` restricts values to the defined list.

---

# Validate Inputs

Even with typed inputs, production scripts should validate important values.

Example:

```bash
case "$SERVICE" in
  catalogue|orders|payment)
    ;;
  *)
    echo "Invalid service"
    exit 1
    ;;
esac
```

This adds another defensive layer.

---

# Workflow API + Production Approval

Triggering a production workflow does not necessarily mean it should immediately deploy.

Better:

```text
API Trigger
   |
   ↓
Workflow
   |
   ↓
Production Environment
   |
   ↓
Required Reviewer
   |
   ↓
Deployment
```

---

# Workflow API + Concurrency

Production workflow:

```yaml
concurrency:
  group: production-${{ inputs.service }}
  cancel-in-progress: false
```

This prevents overlapping deployment runs for the same service.

---

# Workflow API + Timeout

Example:

```yaml
jobs:

  deploy:

    timeout-minutes: 30

    steps:
      - name: Deploy
        run: ./deploy.sh
```

A triggered workflow should still have execution boundaries.

---

# Workflow API + Security Gates

A triggered production workflow should validate:

```text
Tests
Security
Artifact
Commit
Change Request
Environment
```

before deployment.

---

# Complete Production Trigger Flow

```text
External System
      |
      ↓
Authentication
      |
      ↓
Workflow Dispatch API
      |
      ↓
Validate Inputs
      |
      ↓
Validate Commit
      |
      ↓
Security Gates
      |
      ↓
JIRA / CR
      |
      ↓
Production Environment
      |
      ↓
Approval
      |
      ↓
Concurrency
      |
      ↓
Deploy
      |
      ↓
Verify
      |
      ↓
Success / Rollback
```

---

# Workflow API Error Handling

Common HTTP errors:

```text
401
Authentication problem

403
Permission / policy / rate-limit issue

404
Resource not found or inaccessible

422
Validation failure
```

Always inspect the response body when troubleshooting.

---

# API Error Handling Example

```bash
if ! response="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "$API_URL"
)"; then

  echo "GitHub API request failed."
  exit 1
fi
```

---

# Retry Strategy

For transient failures:

```text
Attempt 1
   ↓
Failure
   ↓
Wait
   ↓
Attempt 2
   ↓
Failure
   ↓
Wait
   ↓
Attempt 3
   ↓
Final Result
```

Use bounded retries.

Do not retry:

```text
Invalid Authentication
Invalid Input
Unauthorized Operation
```

indefinitely.

---

# Workflow API Rate Limits

Automation should account for:

```text
API Rate Limits
Pagination
Request Volume
Concurrent Requests
Retry Behavior
```

For large organizations:

```text
100 repositories
 ×
multiple API calls
```

can quickly become expensive in API request terms.

Design automation efficiently.

---

# Workflow API Pagination

List endpoints may return multiple pages.

Example:

```text
Page 1
 ↓
Page 2
 ↓
Page 3
```

Use appropriate pagination logic for production reporting.

---

# Workflow API + Organization Reporting

A platform team can build:

```text
Repository Inventory
       |
       ↓
Workflow API
       |
       ↓
Workflow Runs
       |
       ↓
CI/CD Health
```

Metrics:

```text
Successful Runs
Failed Runs
Failure Rate
Average Duration
Long-Running Runs
Deployment Failures
```

---

# CI/CD Health Dashboard

Conceptual:

```text
Repositories
     |
     ↓
Workflow API
     |
     ↓
Data Collection
     |
 ┌───┼──────────┐
 ↓   ↓          ↓
Pass Fail     Duration
     |
     ↓
Dashboard
```

---

# Workflow Failure Rate

Example calculation:

```text
Failed Runs
---------------- × 100
Total Runs
```

This can help identify unstable pipelines.

---

# Long-Running Workflow Detection

Example:

```text
Expected:
5 minutes

Actual:
35 minutes
```

Automation can detect:

```text
duration > threshold
```

and report it.

---

# Workflow API + Cost Optimization

Workflow API data can help identify:

```text
Slow workflows
Repeated workflows
Failed/retried workflows
Unused workflows
```

Then optimize:

```text
Caching
Parallelization
Matrix Strategy
Runner Choice
Job Dependencies
```

---

# Workflow API + Self-Hosted Runners

Workflow runs can provide information that helps platform teams understand runner usage.

Example:

```text
Workflow
 ↓
Runner
 ↓
Duration
 ↓
Capacity
```

This can help with runner fleet planning.

---

# Workflow API + Runner Governance

Organization-level automation can analyze:

```text
Which workflows use self-hosted runners?
Which repositories use them?
Which workflows run most frequently?
```

This supports security and capacity management.

---

# Workflow API + Deployment Evidence

A production deployment record can contain:

```text
Repository
Workflow
Run ID
Run Number
Commit SHA
Branch
Environment
Artifact
Deployment Result
```

Example:

```text
Service:
catalogue

Commit:
8a92f31

Workflow:
Production Deployment

Run:
#152

Environment:
production

Result:
success
```

---

# Workflow API + Auditability

This creates an audit trail:

```text
Who
 ↓
Triggered What
 ↓
From Which Commit
 ↓
Using Which Workflow
 ↓
To Which Environment
 ↓
With What Result
```

This is valuable for production operations.

---

# Best Practices

```text
☐ Distinguish workflows from workflow runs
☐ Use authenticated API requests
☐ Use least-privilege permissions
☐ Prefer GITHUB_TOKEN when sufficient
☐ Validate workflow inputs
☐ Protect production environments
☐ Use required reviewers
☐ Use concurrency for deployments
☐ Use timeouts
☐ Handle pagination
☐ Handle rate limits
☐ Use bounded retries
☐ Check HTTP status
☐ Preserve commit SHA
☐ Collect failure diagnostics
☐ Do not expose secrets in logs
☐ Do not blindly re-run failed deployments
☐ Separate cancellation from rollback
☐ Use immutable artifacts
☐ Maintain deployment traceability
```

---

# Common Mistakes

### 1. Confusing Workflow and Workflow Run

```text
Workflow = YAML definition

Run = Execution
```

### 2. Triggering production without protection

API access should not bypass production controls.

### 3. Using excessive permissions

Use least privilege.

### 4. Ignoring API errors

A failed API request should not silently become a successful deployment decision.

### 5. No input validation

Never trust arbitrary dispatch inputs.

### 6. Infinite retries

Use bounded retries.

### 7. Re-running every failed deployment

Investigate the failure first.

### 8. Treating cancellation as rollback

Cancellation does not restore application state.

### 9. Ignoring concurrency

Multiple production deployments can overlap.

### 10. No traceability

Always know:

```text
Commit
Workflow
Run
Artifact
Environment
Result
```

---

# Production-Level Example

```yaml
name: Production Deployment

on:

  workflow_dispatch:

    inputs:

      service:
        description: Service to deploy
        required: true
        type: choice
        options:
          - catalogue
          - orders
          - payment

      image_digest:
        description: Immutable image digest
        required: true
        type: string

      jira_ticket:
        description: Approved change request
        required: true
        type: string

permissions:
  contents: read
  id-token: write
  actions: read

jobs:

  validate:

    runs-on: ubuntu-latest

    steps:

      - name: Validate Inputs
        env:
          SERVICE: ${{ inputs.service }}
          IMAGE_DIGEST: ${{ inputs.image_digest }}
          JIRA_TICKET: ${{ inputs.jira_ticket }}
        run: |

          set -euo pipefail

          case "$SERVICE" in
            catalogue|orders|payment)
              ;;
            *)
              echo "Invalid service"
              exit 1
              ;;
          esac

          test -n "$IMAGE_DIGEST"
          test -n "$JIRA_TICKET"

  deploy:

    needs: validate

    runs-on: ubuntu-latest

    timeout-minutes: 30

    concurrency:
      group: production-${{ inputs.service }}
      cancel-in-progress: false

    environment:
      name: production

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS
        run: |
          ./scripts/configure-aws.sh

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          IMAGE_DIGEST: ${{ inputs.image_digest }}
        run: |
          ./scripts/deploy.sh \
            "$SERVICE" \
            "$IMAGE_DIGEST"

      - name: Verify
        env:
          SERVICE: ${{ inputs.service }}
        run: |
          kubectl rollout status \
            deployment/"$SERVICE" \
            -n production \
            --timeout=10m

      - name: Collect Diagnostics
        if: ${{ always() }}
        run: |
          mkdir -p diagnostics

          kubectl get pods \
            -n production \
            > diagnostics/pods.txt

          kubectl get events \
            -n production \
            --sort-by=.lastTimestamp \
            > diagnostics/events.txt

      - name: Upload Diagnostics
        if: ${{ always() }}
        uses: actions/upload-artifact@v4
        with:
          name: production-diagnostics
          path: diagnostics/
```

This example demonstrates:

```text
Workflow Dispatch
Input Validation
Least Privilege
OIDC Permission
Production Environment
Concurrency
Timeout
Immutable Image Digest
Deployment
Verification
Diagnostics
Artifacts
```

The actual AWS authentication, JIRA validation, deployment implementation, and required permissions must match the organization's architecture.

---

# Complete GitHub Workflow API Architecture

```text
                  External System
                        |
                        ↓
                 GitHub REST API
                        |
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          Trigger     Inspect    Control
          Workflow    Runs       Runs
             |          |          |
             ↓          ↓          ↓
          Actions    Status     Cancel/
                                Re-run
             |
             ↓
        GitHub Actions
             |
       ┌─────┼─────────────┐
       ↓     ↓             ↓
     Test  Security      Build
       |     |             |
       └─────┼─────────────┘
             ↓
            ECR
             |
             ↓
          Production
             |
             ↓
           ArgoCD
             |
             ↓
            EKS
```

---

# Interview Questions

## Basic

1. What is the GitHub Workflow API?
2. What is the difference between a workflow and a workflow run?
3. How do you list workflows using the REST API?
4. How do you get a specific workflow?
5. How do you list workflow runs?
6. What is a workflow run ID?
7. What is the difference between workflow status and conclusion?
8. How do you retrieve workflow jobs?
9. How do you retrieve workflow logs?
10. What is `workflow_dispatch`?

## Intermediate

11. How do you trigger a workflow using the GitHub REST API?
12. How do you pass inputs when triggering a workflow?
13. How do you re-run a workflow?
14. How do you cancel a workflow run?
15. How do you determine whether a workflow run succeeded?
16. How do you find the latest workflow run?
17. How do you filter workflow runs by branch?
18. How do you filter workflow runs by status?
19. How do you retrieve failed jobs from a workflow run?
20. How do you download workflow logs?
21. What permissions are required for Actions API operations?
22. How would you authenticate Workflow API requests?
23. How do you trigger a workflow in another repository?
24. How do you handle API pagination?
25. How do you handle API rate limits?

## Advanced / Production

26. Design a cross-repository GitHub Actions triggering architecture.
27. How would you trigger a GitOps deployment workflow after an application image is pushed to ECR?
28. How would you securely trigger a production workflow through the REST API?
29. How would you prevent arbitrary users from triggering production deployments?
30. How would you combine Workflow API with GitHub Environments?
31. How would you combine Workflow API with JIRA change requests?
32. How would you implement commit SHA validation before production deployment?
33. How would you identify repeated workflow failures using the API?
34. How would you build a CI/CD health dashboard using Workflow API data?
35. How would you detect long-running workflows?
36. How would you implement retry logic for transient Workflow API failures?
37. When would you re-run a failed workflow and when would you investigate first?
38. What is the difference between cancelling a workflow and rolling back a deployment?
39. How would you collect workflow logs automatically after production failure?
40. How would you preserve workflow, commit, artifact, and deployment traceability?
41. How would you secure a GitHub Actions workflow that triggers another repository?
42. How would you design Workflow API authentication using GITHUB_TOKEN versus GitHub App?
43. How would you handle multiple microservices and independent deployment concurrency?
44. How would you integrate Workflow API, ECR, GitOps, ArgoCD, and EKS?
45. Design an enterprise-grade GitHub Workflow API architecture covering workflow discovery, run monitoring, dispatch, cross-repository automation, authentication, permissions, pagination, rate limits, retries, production environments, JIRA/CR validation, immutable artifacts, ECR, GitOps, ArgoCD, EKS, rollback, diagnostics, and auditability.