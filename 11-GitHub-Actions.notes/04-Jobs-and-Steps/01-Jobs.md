# GitHub Actions Jobs

A **job** is a collection of steps that runs on a runner.

A workflow can contain one or multiple jobs.

Basic structure:

```yaml
name: Application CI

on:
  push:

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

The basic hierarchy is:

```text
Workflow
   |
   └── Jobs
         |
         └── Steps
```

---

# Job Structure

A job can contain several configuration elements:

```text
Job ID
   |
   ├── Name
   ├── Runner
   ├── Permissions
   ├── Environment
   ├── Dependencies
   ├── Conditions
   ├── Timeout
   ├── Concurrency
   └── Steps
```

Example:

```yaml
jobs:

  build:

    name: Build Application

    runs-on: ubuntu-latest

    timeout-minutes: 20

    permissions:
      contents: read

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

---

# Job ID

Every job needs a unique Job ID.

Example:

```yaml
jobs:

  build:
```

Here:

```text
build
```

is the Job ID.

Multiple jobs:

```yaml
jobs:

  build:

  test:

  security:

  deploy:
```

Job IDs:

```text
build
test
security
deploy
```

Job IDs are used when creating dependencies with `needs` and when referencing job outputs.

---

# Job Name

A job can have a human-readable name.

```yaml
jobs:

  build:

    name: Build Application
```

There is a difference between:

```text
build
```

and:

```text
Build Application
```

`build` is the Job ID.

`Build Application` is the display name.

Use meaningful names because they make the GitHub Actions UI easier to understand.

---

# Runner

Every job executes on a runner.

Example:

```yaml
runs-on: ubuntu-latest
```

The runner is the environment where the job's steps execute.

Architecture:

```text
GitHub Actions
      |
      ↓
   Runner
      |
      ↓
     Job
      |
      ↓
    Steps
```

---

# GitHub-Hosted Runners

GitHub provides managed runners.

Example:

```yaml
runs-on: ubuntu-latest
```

Common runner labels include:

```yaml
runs-on: ubuntu-latest
```

```yaml
runs-on: windows-latest
```

```yaml
runs-on: macos-latest
```

GitHub manages the underlying runner infrastructure.

Advantages:

- Easy to configure
- No server maintenance
- Preinstalled development tools
- Automatically provisioned
- Suitable for most CI workloads

Execution:

```text
Workflow
   |
   ↓
GitHub
   |
   ↓
Runner
   |
   ↓
Job
   |
   ↓
Runner Released
```

---

# Self-Hosted Runners

Organizations can configure their own runners.

Example:

```yaml
runs-on: self-hosted
```

Self-hosted runners are useful when an organization requires:

- Private network access
- Internal systems
- Custom software
- Special hardware
- Custom security configuration
- Access to internal infrastructure

Architecture:

```text
GitHub Actions
      |
      ↓
Self-Hosted Runner
      |
      ↓
Private Network
      |
      ↓
Internal Infrastructure
```

---

# Runner Labels

Self-hosted runners can use labels.

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This allows a job to target a runner with specific characteristics.

Conceptually:

```text
Job
 |
 ↓
Required Labels
 |
 ↓
Matching Runner
 |
 ↓
Execution
```

---

# Steps Inside a Job

A job contains one or more steps.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

Steps within a job execute sequentially.

```text
Checkout
   |
   ↓
Install Dependencies
   |
   ↓
Build
   |
   ↓
Test
```

---

# Multiple Jobs

A workflow can contain multiple jobs.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Build"

  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Test"

  security:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Security"
```

Independent jobs can run in parallel.

```text
              Workflow
                  |
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Build      Test    Security
```

---

# Why Use Multiple Jobs?

Separate jobs provide:

- Parallel execution
- Clear responsibilities
- Better visibility
- Easier troubleshooting
- Independent runner environments
- Separate permissions
- Separate environments
- Better pipeline organization

Example:

```text
Build Job
Test Job
Security Job
Deployment Job
```

Instead of putting everything into one large job.

---

# Parallel Jobs

If jobs do not depend on each other, they can run concurrently.

Example:

```yaml
jobs:

  unit-test:
    runs-on: ubuntu-latest

  sonar:
    runs-on: ubuntu-latest

  trivy:
    runs-on: ubuntu-latest
```

Execution:

```text
                Build
                  |
        ┌─────────┼─────────┐
        ↓         ↓         ↓
    Unit Test   SonarQube   Trivy
```

Parallel execution can reduce total pipeline execution time.

---

# Enterprise DevSecOps Parallelization

For a DevSecOps pipeline, independent checks can run in parallel.

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
      Unit Test  SonarQube  Trivy
          |        |        |
          └────────┼────────┘
                   ↓
                Artifact
```

This can be more efficient than:

```text
Build
  |
  ↓
Unit Test
  |
  ↓
SonarQube
  |
  ↓
Trivy
```

provided the checks do not actually depend on each other.

---

# Sequential Jobs

Jobs can be made dependent using:

```yaml
needs:
```

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:
      - run: echo "Build"

  test:

    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: echo "Test"

  deploy:

    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploy"
```

Execution:

```text
Build
  |
  ↓
Test
  |
  ↓
Deploy
```

---

# Multiple Job Dependencies

A job can depend on multiple jobs.

Example:

```yaml
deploy:

  needs:
    - test
    - security
    - uat

  runs-on: ubuntu-latest
```

Execution:

```text
Test ───────────┐
                |
Security ───────┼──→ Deploy
                |
UAT ────────────┘
```

The deployment waits for all required dependencies.

---

# Production Deployment Dependencies

A production deployment should normally wait for required validation stages.

Example architecture:

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
        Tests   SonarQube  Trivy
          |        |        |
          └────────┼────────┘
                   ↓
                Artifact
                   |
                   ↓
                  UAT
                   |
                   ↓
                E2E Tests
                   |
                   ↓
                Approval
                   |
                   ↓
              Production
```

This creates clear production gates.

---

# Job Conditions

Jobs can run conditionally using:

```yaml
if:
```

Example:

```yaml
deploy:

  if: github.ref == 'refs/heads/main'

  runs-on: ubuntu-latest

  steps:

    - name: Deploy
      run: echo "Deploying"
```

The job runs only when the condition evaluates to true.

---

# Production Condition

A basic production condition can be:

```yaml
if: github.ref == 'refs/heads/main'
```

However, branch checking alone should not be considered sufficient production protection.

Production should additionally use:

- Protected environments
- Required approvals
- Change-management validation
- Security checks
- Testing
- Deployment controls

---

# Job Environment

A job can target a GitHub Environment.

Example:

```yaml
jobs:

  deploy:

    runs-on: ubuntu-latest

    environment: production

    steps:

      - name: Deploy
        run: ./deploy.sh
```

The environment can provide:

- Required reviewers
- Environment-specific secrets
- Deployment protection
- Controlled production access

Architecture:

```text
Deployment Job
      |
      ↓
Production Environment
      |
      ↓
Environment Protection
      |
      ↓
Approval
      |
      ↓
Deployment
```

---

# Job-Level Environment Variables

A job can define environment variables.

Example:

```yaml
jobs:

  deploy:

    runs-on: ubuntu-latest

    env:
      ENVIRONMENT: production

    steps:

      - name: Deploy
        run: echo "Deploying to $ENVIRONMENT"
```

The variable is available to the steps in that job.

---

# Job Timeout

Jobs should have reasonable timeout values.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    timeout-minutes: 20
```

A timeout prevents a stuck job from consuming runner resources indefinitely.

Production pipelines should use realistic timeouts based on expected execution time.

---

# Job Permissions

Permissions can be configured at the job level.

Example:

```yaml
jobs:

  build:

    permissions:
      contents: read
```

Use the principle of least privilege.

Example:

```text
Build Job
   |
   └── contents: read

Security Job
   |
   └── Only required permissions

Deployment Job
   |
   └── Only required deployment permissions
```

Do not give every job broad permissions.

---

# Job Isolation

Jobs normally execute in separate runner environments.

Example:

```text
Build Job
    |
    └── Runner A

Test Job
    |
    └── Runner B

Deploy Job
    |
    └── Runner C
```

Therefore:

```text
Files created in Build Job

        X

Automatically available in Test Job
```

The next job must explicitly obtain required files.

Use:

- Artifacts
- Job outputs
- External artifact repositories
- Repository checkout

---

# Artifacts Between Jobs

Example:

```text
Build Job
    |
    ↓
Build Application
    |
    ↓
Upload Artifact
    |
    ↓
Test Job
    |
    ↓
Download Artifact
    |
    ↓
Test
    |
    ↓
Deploy Job
    |
    ↓
Download Approved Artifact
    |
    ↓
Deploy
```

This is important in production pipelines.

---

# Build Once, Promote Many

A strong enterprise CI/CD principle is:

```text
Source Code
    |
    ↓
Build Once
    |
    ↓
Artifact
    |
    ↓
QA
    |
    ↓
SIT
    |
    ↓
UAT
    |
    ↓
Production
```

Avoid rebuilding different artifacts for each environment.

Bad approach:

```text
Source
  |
  ├── QA Build
  |
  ├── SIT Build
  |
  ├── UAT Build
  |
  └── Production Build
```

The artifact promoted to production should be the same artifact that passed the required validation stages.

---

# Job Outputs

Jobs can produce outputs that other jobs consume.

Conceptually:

```text
Build Job
    |
    ↓
Image Tag
    |
    ↓
Job Output
    |
    ↓
Deploy Job
    |
    ↓
Use Image Tag
```

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - id: version
        run: echo "version=a83f91c" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Display Version
        run: echo "${{ needs.build.outputs.version }}"
```

Outputs are useful for passing controlled values between jobs.

---

# Job-Level Secrets

A job may require secrets.

Example:

```yaml
jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          TOKEN: ${{ secrets.DEPLOYMENT_TOKEN }}

        run: ./deploy.sh
```

Expose secrets only to jobs that need them.

Never hardcode credentials:

```yaml
TOKEN: my-secret-token
```

Use GitHub Secrets or an appropriate enterprise secret-management solution.

---

# Self-Hosted Runner Production Example

Suppose a deployment needs access to a private network.

Architecture:

```text
GitHub Actions
      |
      ↓
Deployment Job
      |
      ↓
Self-Hosted Runner
      |
      ↓
Private Network
      |
      ↓
Internal Infrastructure
```

Example:

```yaml
deploy:

  runs-on:
    - self-hosted
    - linux
    - production
```

Self-hosted runners must be properly secured and maintained.

---

# Job Concurrency

Production deployments should prevent conflicting deployments.

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

Conceptual behavior:

```text
Deployment A
    |
    ↓
Running

Deployment B
    |
    ↓
Waiting
```

This prevents multiple production deployments from running simultaneously.

---

# Production Deployment Job

A production deployment job can combine multiple controls.

```yaml
jobs:

  deploy-prod:

    needs:
      - build
      - security
      - uat
      - e2e

    runs-on:
      - self-hosted
      - linux
      - production

    environment:
      name: production

    concurrency:
      group: production
      cancel-in-progress: false

    timeout-minutes: 30

    permissions:
      contents: read

    steps:

      - name: Deploy
        run: ./deploy.sh

      - name: Smoke Test
        run: ./smoke-test.sh
```

Important controls:

```text
Dependencies
Runner
Environment
Concurrency
Timeout
Permissions
Deployment
Smoke Test
```

---

# Production Job Architecture

A mature production pipeline can look like:

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
        Tests   SonarQube  Trivy
          |        |        |
          └────────┼────────┘
                   ↓
                Artifact
                   |
                   ↓
                  UAT
                   |
                   ↓
                E2E Tests
                   |
                   ↓
                Approval
                   |
                   ↓
              Production
                   |
                   ↓
              Smoke Test
                   |
                   ↓
               Monitoring
```

---

# Microservices Jobs

For a microservices platform, independent services can be processed in parallel.

Example:

```text
Application CI
      |
 ┌────┼─────┬──────┐
 ↓    ↓     ↓      ↓
User Cart  Order  Payment
 ↓    ↓     ↓      ↓
Build Build Build Build
```

After all required builds complete:

```text
All Services
      |
      ↓
Integration Tests
      |
      ↓
Security Validation
      |
      ↓
Deployment
```

This approach can improve CI/CD performance.

---

# Faster Pipeline Design

Suppose a pipeline currently takes 25 minutes.

A sequential design might be:

```text
Build
  |
  ↓
Unit Test
  |
  ↓
SonarQube
  |
  ↓
Trivy
  |
  ↓
Integration Test
  |
  ↓
Deploy
```

If the checks are independent, they can be parallelized:

```text
             Build
               |
       ┌───────┼────────┐
       ↓       ↓        ↓
   Unit Test SonarQube Trivy
       |       |        |
       └───────┼────────┘
               ↓
        Integration Test
               |
               ↓
             Deploy
```

This can reduce total execution time.

Do not parallelize jobs when a real dependency exists.

---

# Job Failure Behavior

Suppose:

```text
Build
  |
  ├── Test
  ├── SonarQube
  └── Trivy
```

If a mandatory security job fails:

```text
Trivy
  |
  ↓
Failed
  |
  ↓
Deployment
  X
```

Production deployment should not continue when a mandatory security gate fails.

---

# Job Failure Troubleshooting

Check the following:

```text
1. Job status
2. Runner status
3. Job condition
4. needs dependencies
5. Environment protection
6. Permissions
7. Secrets
8. Step logs
9. Artifact availability
10. External dependencies
```

Start with the workflow graph, then inspect the failed job and its individual steps.

---

# Scenario: Job Is Skipped

Common reasons:

```text
if condition = false

OR

Dependency failed

OR

Dependency was skipped

OR

Environment protection prevented execution
```

Check:

```text
Workflow Graph
      |
      ↓
Job Status
      |
      ↓
Condition
      |
      ↓
Dependencies
      |
      ↓
Environment
```

---

# Scenario: Job Cannot Access Artifact

Remember:

```text
Job A
  |
  ↓
Runner A
  |
  ↓
Artifact
  |
  ↓
Upload
  |
  ↓
Job B
  |
  ↓
Download
```

Do not assume the artifact exists on Job B's runner.

---

# Scenario: Production Deploy Starts Too Early

Check the `needs` configuration.

Example:

```yaml
deploy:

  needs:
    - build
    - security
    - uat
    - e2e
```

If a required validation job is missing from `needs`, the deployment may not wait for it.

---

# Scenario: Multiple Production Deployments

Use concurrency:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

Also consider:

- Environment protection
- Deployment locks
- Idempotent deployment procedures
- Change-request validation
- Deployment-window validation

---

# Production-Level Job Design

A production-grade GitHub Actions job should consider:

```text
1. Clear responsibility
2. Correct runner
3. Least-privilege permissions
4. Required dependencies
5. Environment protection
6. Timeout
7. Concurrency
8. Secret management
9. Artifact handling
10. Failure handling
11. Logging
12. Rollback strategy
```

For example:

```text
Production Deployment Job

        |
        ├── Required CI checks
        |
        ├── Security checks
        |
        ├── UAT
        |
        ├── E2E tests
        |
        ├── Change approval
        |
        ├── Deployment window
        |
        ├── Production environment
        |
        ├── Deployment
        |
        ├── Smoke test
        |
        └── Rollback if required
```

---

# Best Practices

- Give each job a clear responsibility.
- Use meaningful Job IDs.
- Use descriptive job names.
- Use parallel jobs when dependencies allow.
- Use `needs` for required dependencies.
- Use artifacts to transfer files between jobs.
- Use outputs for controlled values between jobs.
- Use protected environments for production.
- Use least-privilege permissions.
- Set reasonable timeouts.
- Use concurrency for production deployments.
- Use self-hosted runners only when required.
- Keep production deployment behind validation gates.
- Build artifacts once and promote the same artifact.
- Keep secrets out of source code.
- Make deployment procedures idempotent where possible.

---

# Common Mistakes

- Putting the entire pipeline into one huge job.
- Forgetting `needs`.
- Assuming jobs share the same filesystem.
- Giving every job excessive permissions.
- Not using artifacts between isolated jobs.
- Running production deployment without validation dependencies.
- Allowing multiple production deployments simultaneously.
- Using self-hosted runners without proper security.
- Not setting reasonable job timeouts.
- Rebuilding different artifacts for every environment.
- Using `continue-on-error` on mandatory production gates.
- Exposing secrets to jobs that do not need them.
- Using a branch condition as the only production protection.

---

# Summary

A GitHub Actions job is a logical unit of work executed on a runner.

Jobs can run:

```text
Sequentially

or

In Parallel

or

Based on Dependencies
```

Important job concepts include:

```text
Job ID
Runner
Steps
needs
if
Environment
Timeout
Concurrency
Outputs
Artifacts
Permissions
Secrets
```

A production-grade pipeline can be organized as:

```text
Build
  |
  ├── Unit Tests
  ├── SonarQube
  └── Trivy
        |
        ↓
     Artifact
        |
        ↓
       UAT
        |
        ↓
     E2E Tests
        |
        ↓
      Approval
        |
        ↓
   Production
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring
```

The key principle is:

```text
Use jobs to separate responsibilities,
parallelize independent work,
and enforce dependencies for required stages.
```

---

# Interview Questions

## Basic

1. What is a job in GitHub Actions?
2. What is a Job ID?
3. What is `runs-on`?
4. What is the difference between a job and a step?
5. Can multiple jobs exist in one workflow?
6. What is a GitHub-hosted runner?
7. What is a self-hosted runner?

## Intermediate

8. How do jobs execute by default?
9. How do you make jobs run sequentially?
10. What is `needs`?
11. How do you run jobs in parallel?
12. How do you pass files between jobs?
13. Why don't jobs automatically share the same filesystem?
14. What are job outputs?
15. What is a job environment?
16. Why should jobs have timeouts?
17. What is job-level `if`?
18. What is job-level `permissions`?

## Advanced

19. Design a GitHub Actions CI pipeline where Build runs first, Unit Tests, SonarQube, and Trivy run in parallel, and Deployment waits for all of them.
20. Design a production deployment job that uses a self-hosted runner, production environment protection, concurrency, timeout, least-privilege permissions, and deployment dependencies.
21. A deployment job starts before the security job completes. Explain how you would troubleshoot and fix the job dependency configuration.
22. A file created during the Build job is missing in the Deployment job. Explain why this happens and how you would solve it.
23. A GitHub Actions pipeline takes 25 minutes. Explain how you would analyze the job dependency graph and identify opportunities for parallel execution.
24. Design a microservices CI pipeline where multiple services build in parallel and deployment occurs only after all required services pass validation.
25. Explain how job-level permissions, environments, runners, artifacts, and dependencies should be designed for a production deployment pipeline.
26. How would you prevent two production deployment jobs from running simultaneously?
27. Why is "build once, promote many" preferred in production CI/CD?
28. How would you troubleshoot a job that is unexpectedly skipped?
29. How would you securely handle secrets required by only the deployment job?
30. How would you design a job so that failure of a mandatory security scan prevents production deployment?