# GitHub Actions Timeouts

Timeouts control how long a GitHub Actions job or step is allowed to run.

Timeouts are important for production-grade CI/CD because they prevent workflows from running indefinitely due to:

```text
Hung commands
Network issues
Deadlocked processes
Unavailable services
Broken tests
Terraform operations
Kubernetes commands
Deployment problems
External API failures
```

---

# Why Timeouts Matter

Without a timeout:

```text
Workflow
   |
   ↓
Command hangs
   |
   ↓
Runner keeps waiting
   |
   ↓
Resources remain occupied
   |
   ↓
Pipeline is blocked
```

With a timeout:

```text
Workflow
   |
   ↓
Command hangs
   |
   ↓
Timeout reached
   |
   ↓
Job/Step stops
   |
   ↓
Failure handling
```

---

# Basic Job Timeout

Use:

```yaml
timeout-minutes: 15
```

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:

      - name: Build
        run: |
          ./build.sh
```

The job is limited by the configured timeout.

---

# Timeout at Job Level

Example:

```yaml
jobs:

  test:

    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:

      - name: Run Tests
        run: |
          ./run-tests.sh
```

The timeout applies to the job.

---

# Why Job-Level Timeout Is Important

Suppose a test command becomes stuck:

```text
Run Tests
   |
   ↓
Process hangs
   |
   ↓
No timeout
   |
   ↓
Runner remains occupied
```

A job timeout provides a maximum execution boundary.

---

# Step-Level Timeout

For individual steps, GitHub Actions supports step-level timeout behavior through the appropriate workflow syntax and action/command design.

When designing workflows, distinguish between:

```text
Job timeout
```

and:

```text
Individual command/action timeout
```

Use the job timeout as the primary workflow execution boundary.

---

# Job Timeout vs Command Timeout

These solve different problems.

### Job timeout

Controls the overall job execution window.

```yaml
timeout-minutes: 20
```

### Command-specific timeout

Can control an individual shell command.

Example:

```bash
timeout 300 ./integration-tests.sh
```

This means:

```text
300 seconds
```

---

# Linux `timeout` Command

Example:

```yaml
- name: Integration Tests
  run: |
    timeout 300 ./integration-tests.sh
```

If the command exceeds the specified duration, the command is terminated according to the `timeout` utility behavior.

---

# Why Use Both?

You can use:

```text
Command timeout
+
Job timeout
```

Example:

```text
Job
 └── 30 minutes maximum

     Integration Tests
       └── 10 minutes maximum

     Security Scan
       └── 10 minutes maximum
```

This gives more granular control.

---

# Timeout Design

A good timeout strategy should be based on:

```text
Normal execution time
+
Expected variation
+
Infrastructure/network delays
```

Do not choose an extremely small timeout just to make failures happen faster.

---

# Example

Suppose a test normally takes:

```text
5 minutes
```

Possible timeout:

```text
10–15 minutes
```

instead of:

```text
5 minutes
```

because temporary delays may occur.

---

# Timeout Too Short

Example:

```text
Normal build → 8 minutes

Timeout → 5 minutes
```

Result:

```text
Build
  |
  ↓
Timeout
  |
  ↓
Failure
```

The pipeline fails even though the build is healthy.

---

# Timeout Too Long

Example:

```text
Normal test → 10 minutes

Timeout → 4 hours
```

If the test hangs:

```text
10 minutes
   ↓
20 minutes
   ↓
1 hour
   ↓
4 hours
```

Resources remain occupied unnecessarily.

---

# Recommended Principle

Use:

```text
Realistic timeout
+
Failure handling
+
Observability
```

rather than simply using a very large timeout.

---

# Timeout by Pipeline Stage

Different stages may need different limits.

Example:

```text
Linting
  → 5 minutes

Unit Tests
  → 15 minutes

Build
  → 20 minutes

Security Scan
  → 20 minutes

Integration Tests
  → 30 minutes

Deployment
  → 20 minutes
```

Actual values should be based on your application's historical execution times.

---

# CI Timeout Example

```yaml
name: CI

on:
  push:

jobs:

  build:

    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:

      - name: Build
        run: |
          ./build.sh

      - name: Test
        run: |
          ./test.sh
```

---

# Security Scan Timeout

Security tools can occasionally take longer due to:

```text
Large dependency trees
Large container images
Network access
Scanner service latency
Repository size
```

Example:

```yaml
security:

  runs-on: ubuntu-latest
  timeout-minutes: 30

  steps:

    - name: Trivy Scan
      run: |
        trivy image "$IMAGE"
```

---

# SonarQube Timeout

Example:

```yaml
sonarqube:

  runs-on: ubuntu-latest
  timeout-minutes: 20

  steps:

    - name: SonarQube Scan
      run: |
        mvn sonar:sonar
```

Choose the timeout based on actual project scan duration.

---

# Veracode Timeout

External security services may take longer depending on:

```text
Application size
Upload size
Analysis queue
Network
Service availability
```

Use a realistic job timeout and, where supported, configure the scanner's own wait/polling timeout separately.

---

# Docker Build Timeout

Example:

```yaml
build:

  runs-on: ubuntu-latest
  timeout-minutes: 30

  steps:

    - name: Docker Build
      run: |
        docker build -t "$IMAGE" .
```

Large images may require more time.

---

# Docker Push Timeout

Example:

```yaml
- name: Push Image
  run: |
    docker push "$IMAGE"
```

A slow registry/network can increase execution time.

Use a realistic job timeout.

---

# Kubernetes Deployment Timeout

Example:

```yaml
deploy:

  runs-on: ubuntu-latest
  timeout-minutes: 30

  steps:

    - name: Helm Deploy
      run: |
        helm upgrade --install catalogue ./helm/catalogue \
          --wait \
          --timeout 15m
```

Here there are two different timeout concepts:

```text
GitHub job timeout
        +
Helm deployment timeout
```

---

# Helm Timeout

Example:

```bash
helm upgrade --install catalogue ./helm/catalogue \
  --wait \
  --timeout 15m
```

Helm's timeout controls how long Helm waits for the operation to complete.

GitHub Actions' job timeout controls the overall job.

---

# Kubernetes Rollout Timeout

Example:

```bash
kubectl rollout status \
  deployment/catalogue \
  --timeout=10m
```

This controls the Kubernetes rollout command.

Architecture:

```text
GitHub Job
   |
   └── Helm / kubectl timeout
```

---

# Timeout Layers

A production deployment can have:

```text
GitHub Actions
      |
      ↓
Job Timeout
      |
      ↓
Helm Timeout
      |
      ↓
Kubernetes Rollout Timeout
      |
      ↓
Application Health Checks
```

Each layer serves a different purpose.

---

# Terraform Timeout

Terraform operations may hang because of:

```text
Cloud API
Network
AWS resource provisioning
Dependency issues
Provider behavior
```

Example:

```yaml
apply:

  runs-on: ubuntu-latest
  timeout-minutes: 45

  steps:

    - name: Terraform Apply
      run: |
        terraform apply tfplan
```

Do not choose an aggressive timeout for infrastructure provisioning.

---

# Terraform Resource Timeouts

Terraform resources can also support provider/resource-specific timeout configurations depending on the resource.

Example concept:

```text
GitHub Actions timeout
        ↓
Terraform command timeout boundary

Terraform resource timeout
        ↓
Resource operation timeout
```

These are separate mechanisms.

---

# Database Migration Timeout

Database migrations can be long-running and potentially dangerous.

Example:

```yaml
migration:

  runs-on: ubuntu-latest
  timeout-minutes: 20

  steps:

    - name: Migration
      run: |
        timeout 900 ./migrate.sh
```

The workflow has:

```text
Job timeout
+
Migration command timeout
```

---

# External API Timeout

A deployment may call:

```text
JIRA API
GitHub API
AWS API
Security APIs
Notification APIs
```

Do not allow network calls to hang indefinitely.

For example, with `curl`:

```bash
curl --fail --silent --show-error \
     --connect-timeout 10 \
     --max-time 60 \
     "https://api.example.com/..."
```

---

# `curl` Timeouts

Important options:

```text
--connect-timeout
--max-time
```

Example:

```bash
curl \
  --connect-timeout 10 \
  --max-time 60 \
  --fail \
  "https://api.example.com"
```

This creates an application-level timeout.

---

# Why API Timeouts Matter

Without a timeout:

```text
API call
   |
   ↓
Network issue
   |
   ↓
curl waits
   |
   ↓
Workflow remains blocked
```

With:

```text
connect timeout
+
maximum time
+
job timeout
```

failure happens predictably.

---

# Timeout and Retries

A timeout should often be combined with controlled retries.

Example:

```text
API Call
   |
   ↓
Failure
   |
   ↓
Retry
   |
   ↓
Failure
   |
   ↓
Retry
   |
   ↓
Final Failure
```

Do not retry indefinitely.

---

# Retry vs Timeout

These solve different problems.

### Timeout

```text
How long should we wait?
```

### Retry

```text
How many times should we try again?
```

---

# Example Retry Strategy

```text
Attempt 1
   ↓
Timeout
   ↓
Wait
   ↓
Attempt 2
   ↓
Timeout
   ↓
Wait
   ↓
Attempt 3
   ↓
Failure
```

Use exponential backoff where appropriate.

---

# Timeout and Concurrency

These features work together.

Concurrency:

```text
Controls overlapping runs
```

Timeout:

```text
Controls maximum execution duration
```

Example:

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: false

jobs:

  deploy:

    timeout-minutes: 30
```

---

# Why Both Matter

Without concurrency:

```text
Deploy A
Deploy B
Deploy C
```

may overlap.

Without timeout:

```text
Deploy A
   ↓
Hangs forever
```

Together:

```text
Concurrency
   ↓
One deployment
   ↓
Timeout
   ↓
Maximum duration
```

---

# Timeout and Cancellation

A workflow can stop because of:

```text
Successful completion
Failure
Manual cancellation
Concurrency cancellation
Timeout
```

These are different states and should be handled appropriately.

---

# Timeout Does Not Mean Rollback

Very important:

```text
Timeout
≠
Rollback
```

Example:

```text
Helm upgrade
   |
   ↓
Timeout
   |
   ↓
GitHub job stops
```

This does not automatically mean:

```text
Application rolled back
```

Your deployment mechanism must explicitly handle rollback.

---

# Timeout and Helm Rollback

A deployment design may use:

```text
Helm Upgrade
    |
    ↓
Health Check
    |
    ↓
Failure / Timeout
    |
    ↓
Controlled Rollback
```

For Helm, `--atomic` can be considered when appropriate:

```bash
helm upgrade --install catalogue ./helm/catalogue \
  --wait \
  --timeout 15m \
  --atomic
```

With `--atomic`, Helm handles rollback behavior as part of the Helm operation when the upgrade fails.

Understand the behavior before using it in production.

---

# Timeout and Kubernetes Health

Deployment timeout may happen because:

```text
Pod CrashLoopBackOff
Pod Pending
ImagePullBackOff
Readiness Probe Failure
Insufficient Resources
Node Problems
Service Dependency Failure
```

The correct response is not simply:

```text
Increase timeout
```

First determine why the deployment is taking too long.

---

# Timeout Troubleshooting

If a deployment times out:

```text
1. Check workflow logs
2. Identify the command that timed out
3. Check Kubernetes events
4. Check pod status
5. Check application logs
6. Check readiness probes
7. Check resource limits
8. Check image availability
9. Check dependencies
10. Determine whether rollback is required
```

---

# Kubernetes Troubleshooting Commands

Example:

```bash
kubectl get pods -n production
```

```bash
kubectl describe pod <pod> -n production
```

```bash
kubectl get events -n production --sort-by=.lastTimestamp
```

```bash
kubectl rollout status \
  deployment/catalogue \
  -n production \
  --timeout=10m
```

---

# Timeout and EKS

For EKS deployments:

```text
GitHub Actions
      |
      ↓
AWS Authentication
      |
      ↓
EKS
      |
      ↓
Helm
      |
      ↓
Kubernetes
```

Timeouts can exist at multiple layers:

```text
GitHub job
AWS/API command
Helm
kubectl
Kubernetes rollout
```

---

# Timeout and ArgoCD

In GitOps:

```text
GitHub Actions
      |
      ↓
GitOps Commit
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

The GitHub workflow timeout should not be treated as the same thing as ArgoCD reconciliation timeout.

GitHub Actions may finish after committing the desired state while ArgoCD continues reconciliation.

---

# GitOps Timeout Design

Example:

```text
GitHub Actions
   |
   └── Update GitOps
          |
          └── timeout: 10m

ArgoCD
   |
   └── Reconciliation
          |
          └── Separate health/sync behavior
```

Do not artificially keep GitHub Actions waiting for ArgoCD unless the workflow explicitly requires synchronous verification.

---

# ArgoCD Sync Verification

If your workflow needs to wait for ArgoCD:

```text
Git Commit
   |
   ↓
ArgoCD Sync
   |
   ↓
Application Health
   |
   ↓
Success / Failure
```

The polling command should have its own timeout.

---

# Timeout and Production Deployment

A production deployment should define:

```text
Maximum workflow duration
Maximum deployment wait
Maximum API request duration
Maximum health-check duration
```

Example:

```text
GitHub Job → 30m
Helm → 15m
kubectl rollout → 10m
API request → 60s
```

These values are examples only and must be tuned to the actual application.

---

# Timeout Budget

Think of timeouts as a budget.

Example:

```text
Production Job
      30 minutes
          |
          ├── Validation: 5m
          ├── Helm: 15m
          ├── Verification: 8m
          └── Remaining buffer: 2m
```

The outer timeout should account for the operations inside it.

---

# Bad Timeout Design

```text
GitHub Job → 10m
Helm → 15m
```

This is inconsistent.

The GitHub job can terminate before Helm's own timeout is reached.

---

# Better Timeout Design

```text
GitHub Job → 30m
Helm → 15m
Verification → 10m
```

The outer timeout gives enough room for internal operations and cleanup.

---

# Timeout Hierarchy

```text
Outer Timeout
      |
      ├── Validation Timeout
      |
      ├── Deployment Timeout
      |
      ├── Verification Timeout
      |
      └── Cleanup Timeout
```

Outer limits should be greater than the expected sum of critical inner operations plus reasonable buffer.

---

# Timeout and Cleanup

Example:

```yaml
- name: Cleanup
  if: ${{ always() }}
  run: |
    ./cleanup.sh
```

Be aware that a job-level timeout can terminate the job, so do not assume arbitrary cleanup will always finish after a hard timeout.

Critical cleanup should be designed separately where necessary.

---

# Timeout and Logs

When a timeout occurs, logs should help identify:

```text
Which job?
Which step?
Which command?
Which environment?
Which service?
Which version?
How long did it run?
```

Example metadata:

```text
Service: catalogue
Environment: production
SHA: 8a92f31
Workflow: Production Deployment
Run: 12345
```

---

# Timeout and Observability

Your monitoring stack can help identify why operations are slow.

For your stack:

```text
Prometheus
Grafana
ELK
```

Use:

```text
Metrics
Dashboards
Logs
```

to distinguish:

```text
Slow deployment
Application failure
Infrastructure problem
Network issue
Resource pressure
```

---

# Timeout and Performance

Repeated timeouts may indicate:

```text
Slow build
Slow tests
Slow Docker builds
Slow security scans
Slow API calls
Slow Kubernetes rollout
Resource bottlenecks
```

Do not simply increase timeout values repeatedly.

Investigate the root cause.

---

# Timeout Trend

Example:

```text
Build Duration

10m → 12m → 15m → 20m → 30m
```

If timeout is:

```text
35m
```

the pipeline technically works.

But the trend indicates a performance problem.

---

# Production Principle

```text
Timeout is a safety boundary,
not a performance solution.
```

If a pipeline becomes slower:

```text
Measure
  ↓
Identify bottleneck
  ↓
Optimize
  ↓
Re-evaluate timeout
```

---

# Timeout and CI Optimization

Possible optimization areas:

```text
Caching
Parallel jobs
Dependency caching
Docker layer caching
Test parallelization
Incremental builds
Artifact reuse
```

The next file:

```text
03-Caching.md
```

will cover caching in detail.

---

# Timeout and Long-Running Tests

For long-running integration tests:

```text
Test Suite
    |
    ├── Service A
    ├── Service B
    ├── Service C
    └── Service D
```

Consider parallelization rather than simply increasing the timeout.

---

# Timeout and Matrix

Example:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart
      - orders
```

Each matrix job can have:

```yaml
timeout-minutes: 20
```

This prevents one matrix execution from running indefinitely.

---

# Matrix Timeout

Example:

```yaml
jobs:

  test:

    strategy:
      matrix:
        service:
          - user
          - catalogue
          - cart

    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:

      - name: Test
        env:
          SERVICE: ${{ matrix.service }}
        run: |
          ./test.sh "$SERVICE"
```

---

# Timeout and Reusable Workflows

Reusable workflows should define reasonable timeout boundaries.

Example:

```text
Application Workflow
        |
        ↓
Reusable Build Workflow
        |
        ↓
Reusable Security Workflow
        |
        ↓
Reusable Deployment Workflow
```

Each reusable workflow should have sensible operational limits.

---

# Timeout and Custom Actions

Custom Actions may call:

```text
APIs
Cloud CLIs
Scanners
Deployment tools
```

They should avoid indefinite waits.

Where possible:

```text
Action timeout
+
Command timeout
+
Workflow timeout
```

should be designed together.

---

# Timeout and AWS CLI

AWS CLI commands may interact with:

```text
EC2
EKS
ECR
S3
IAM
RDS
```

If an external command can hang, consider command-level timeout/retry mechanisms appropriate to the tool and operation.

---

# Timeout and ECR

Example:

```text
Docker Push
   |
   ↓
ECR
   |
   ↓
Network problem
```

A job timeout prevents the overall job from running forever.

For better reliability, also investigate:

```text
Network
Registry availability
Image size
Runner performance
```

---

# Timeout and AWS Authentication

If AWS authentication hangs or fails:

```text
GitHub OIDC
    |
    ↓
AWS STS
    |
    ↓
IAM Role
```

Use appropriate command/network timeout behavior and fail clearly rather than waiting indefinitely.

---

# Timeout and JIRA API

For your production change process:

```text
GitHub Actions
     |
     ↓
JIRA API
     |
     ↓
Ticket Validation
```

Use API-level timeouts.

Example:

```bash
curl \
  --connect-timeout 10 \
  --max-time 60 \
  --fail \
  --silent \
  --show-error \
  "$JIRA_URL"
```

---

# JIRA Timeout Handling

If JIRA is unavailable:

```text
JIRA API
   |
   ↓
Timeout
   |
   ↓
Validation Failed
   |
   ↓
Production Deployment Blocked
```

For production change control, failing closed is generally safer than assuming approval.

---

# Security Scan Timeout Handling

If a required security scanner times out:

```text
Trivy / Veracode
      |
      ↓
Timeout
      |
      ↓
Unknown Result
```

Do not automatically interpret:

```text
Timeout
```

as:

```text
Security Passed
```

For a mandatory security gate, treat an unavailable/unknown result according to your organization's fail-open/fail-closed policy; production security gates commonly fail closed.

---

# Timeout and Fail-Closed Design

For critical controls:

```text
Unknown
   ↓
Do not promote
```

Example:

```text
JIRA unavailable
   ↓
No approval verification
   ↓
Production blocked
```

or:

```text
Security scanner unavailable
   ↓
No trusted scan result
   ↓
Production blocked
```

---

# Timeout and Deployment Safety

For production:

```text
Timeout
   ↓
Stop waiting
   ↓
Determine actual system state
   ↓
Verify deployment
   ↓
Rollback if required
```

Do not assume the system is unchanged simply because the GitHub job timed out.

---

# Timeout After Partial Deployment

Example:

```text
Helm upgrade
   |
   ├── Service A updated
   ├── Service B updated
   └── Service C waiting
             |
             ↓
          Timeout
```

The cluster may be in a partially changed state.

Next steps:

```text
Check Helm release
Check Kubernetes state
Check application health
Determine rollback
```

---

# Timeout Troubleshooting Commands

Helm:

```bash
helm status <release> -n <namespace>
```

History:

```bash
helm history <release> -n <namespace>
```

Kubernetes:

```bash
kubectl get pods -n <namespace>
```

Rollout:

```bash
kubectl rollout status deployment/<deployment> \
  -n <namespace> \
  --timeout=10m
```

Events:

```bash
kubectl get events \
  -n <namespace> \
  --sort-by=.lastTimestamp
```

---

# Timeout and Helm History

If a deployment times out:

```bash
helm history catalogue -n production
```

This helps identify:

```text
Previous revision
Current revision
Failed revision
Rollback candidate
```

Do not rollback blindly without checking the actual application state.

---

# Timeout and Rollback

Conceptual workflow:

```text
Deploy
  |
  ↓
Timeout
  |
  ↓
Inspect
  |
  ├── Healthy → Continue
  |
  └── Unhealthy
         |
         ↓
      Rollback
         |
         ↓
      Verify
```

---

# Timeout and Concurrency

Combine:

```yaml
concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false
```

with:

```yaml
timeout-minutes: 30
```

This gives:

```text
One deployment at a time
+
Maximum execution duration
```

---

# Production Deployment Example

```yaml
name: Production Deployment

on:
  workflow_dispatch:
    inputs:

      service:
        required: true
        type: string

      environment:
        required: true
        type: choice
        options:
          - qa
          - uat
          - production

      version:
        required: true
        type: string

concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false

jobs:

  deploy:

    timeout-minutes: 30

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          VERSION: ${{ inputs.version }}
        run: |
          helm upgrade --install "$SERVICE" \
            "./helm/$SERVICE" \
            --namespace production \
            --wait \
            --timeout 15m \
            --atomic \
            --set image.tag="$VERSION"
```

This demonstrates:

```text
Concurrency
+
Job Timeout
+
GitHub Environment
+
Helm Timeout
+
Helm Atomic Rollback
```

The actual workflow should validate that the service, environment, and version are trusted and approved before deployment.

---

# Production Timeout Architecture

```text
                    GitHub Workflow
                           |
                           ↓
                   Job Timeout
                    30 minutes
                           |
                           ↓
                  Deployment Command
                           |
                           ↓
                     Helm Timeout
                       15 minutes
                           |
                           ↓
                  Kubernetes Rollout
                       10 minutes
                           |
                           ↓
                  Health Verification
                           |
                     ┌─────┴─────┐
                     ↓           ↓
                   PASS        FAIL
                     |           |
                     ↓           ↓
                  Success     Rollback
```

---

# Timeout Checklist

Before production:

```text
☐ Job timeout defined
☐ Timeout based on historical execution time
☐ Internal command timeout considered
☐ API connect timeout configured
☐ API maximum duration configured
☐ Retry count bounded
☐ Helm timeout configured
☐ Kubernetes rollout timeout configured
☐ Terraform operation timeout considered
☐ Database migration timeout considered
☐ Deployment rollback strategy defined
☐ Timeout does not automatically assume rollback
☐ Partial deployment recovery documented
☐ Logs identify the timed-out operation
☐ Monitoring available
☐ Production concurrency configured
```

---

# Common Mistakes

### 1. No timeout

```text
Command hangs indefinitely
```

### 2. Timeout too short

```text
Healthy operation fails unnecessarily
```

### 3. Timeout too long

```text
Broken workflow consumes resources
```

### 4. Increasing timeout instead of fixing performance

```text
30m → 60m → 120m
```

without investigating the root cause.

### 5. Assuming timeout means rollback

It does not.

### 6. Ignoring nested timeouts

GitHub:

```text
30m
```

Helm:

```text
60m
```

is poorly aligned.

### 7. No API timeout

Network requests may hang.

### 8. Treating security timeout as success

An unavailable security result should not automatically become:

```text
PASSED
```

---

# Best Practices

- Always define realistic job timeouts for important workflows.
- Base timeout values on historical execution data.
- Use shorter timeouts for simple operations.
- Give long-running operations appropriate buffers.
- Use command-specific timeouts when useful.
- Use API connection and maximum-duration timeouts.
- Bound retry behavior.
- Align nested timeouts with the outer job timeout.
- Treat timeout and rollback as separate mechanisms.
- Verify actual infrastructure state after deployment timeout.
- Use fail-closed behavior for critical validation where appropriate.
- Combine timeout with concurrency for deployment safety.
- Use observability to investigate repeated timeouts.
- Optimize slow workflows instead of continually increasing timeout values.
- Document timeout expectations for production operations.

---

# Key Takeaways

Timeouts provide a maximum execution boundary.

Remember:

```text
timeout-minutes
```

controls the GitHub Actions job execution limit.

Command-level timeouts can control individual operations:

```bash
timeout 300 ./script.sh
```

API-level timeouts can control network requests:

```bash
curl \
  --connect-timeout 10 \
  --max-time 60 \
  ...
```

Deployment tools have their own timeout mechanisms:

```text
GitHub Actions
      ↓
Helm
      ↓
kubectl
      ↓
Kubernetes
```

The critical production principle is:

```text
Timeout
    ≠
Rollback
```

If a deployment times out:

```text
Stop waiting
    ↓
Check actual system state
    ↓
Check logs/events
    ↓
Verify health
    ↓
Rollback if required
    ↓
Verify recovery
```

Another important principle:

```text
Timeout is a safety boundary,
not a performance solution.
```

If workflows repeatedly approach their timeout:

```text
Measure
  ↓
Find bottleneck
  ↓
Optimize
  ↓
Re-evaluate timeout
```

---

# Interview Questions

## Basic

1. What is `timeout-minutes` in GitHub Actions?
2. Why should CI/CD jobs have timeouts?
3. What happens when a GitHub Actions job reaches its timeout?
4. What is the difference between a job timeout and a command timeout?
5. How do you configure a job timeout?
6. Why should different pipeline stages have different timeout values?
7. What is the Linux `timeout` command?
8. What are `curl --connect-timeout` and `curl --max-time`?
9. Why are API timeouts important?
10. Does a GitHub Actions timeout automatically roll back a deployment?

## Intermediate

11. How would you choose a timeout value for a production deployment?
12. How would you configure timeouts for Docker builds?
13. How would you configure timeouts for security scans?
14. How would you configure Helm and GitHub Actions timeouts together?
15. What is the difference between Helm `--timeout` and GitHub `timeout-minutes`?
16. How would you configure a Kubernetes rollout timeout?
17. How would you handle a Terraform apply that exceeds the workflow timeout?
18. How would you handle a JIRA API timeout during production validation?
19. How would you handle a security scanner timeout?
20. Why should security validation generally fail closed when the result is unknown?
21. How do retries and timeouts work together?
22. Why should retry counts be bounded?
23. How would you combine timeout with concurrency?
24. How would you design timeouts for a matrix workflow?
25. How would you troubleshoot a Kubernetes deployment that timed out?

## Advanced / Production

26. Design a timeout strategy for a production EKS deployment using Helm.
27. How would you define timeout budgets for Build → Security → UAT → E2E → Production?
28. How would you design nested timeouts between GitHub Actions, Helm, and Kubernetes?
29. A Helm deployment timed out. How would you determine whether the application actually deployed successfully?
30. A GitHub Actions deployment timed out after Helm updated some resources. How would you recover safely?
31. How would you combine timeout, concurrency, and Helm rollback?
32. How would you handle a timeout during a Terraform apply?
33. How would you distinguish a genuine performance problem from an appropriately configured long-running operation?
34. How would you handle a JIRA API timeout when production approval is mandatory?
35. How would you handle a Trivy or Veracode timeout in a DevSecOps pipeline?
36. How would you design timeout and retry behavior for AWS APIs?
37. How would you design timeout behavior for database migrations?
38. How would you prevent a GitHub Actions timeout from leaving a production deployment in an unknown state?
39. How would you design timeout handling for a GitOps workflow using ArgoCD and EKS?
40. How would you monitor repeated workflow timeouts using Prometheus, Grafana, and ELK?
41. Why is increasing the timeout repeatedly not a proper solution to pipeline performance problems?
42. How would you design timeout values for independent microservices with different deployment durations?
43. How would you design an enterprise-grade timeout strategy across Jenkins/GitHub Actions migration, Terraform, Docker, Trivy, Veracode, Helm, ArgoCD, and EKS?
44. A production deployment reaches its GitHub timeout but the EKS deployment appears healthy. What would you check before declaring success?
45. Design a production-safe GitHub Actions deployment that combines concurrency, timeout boundaries, JIRA validation, security gates, Helm `--atomic`, Kubernetes health verification, ArgoCD, EKS, and rollback recovery.