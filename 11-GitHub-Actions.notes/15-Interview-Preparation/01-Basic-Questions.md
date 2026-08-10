# GitHub Actions - Basic Interview Questions

GitHub Actions interview preparation should begin with the fundamentals:

    GitHub Actions
        |
        ↓
    Workflow
        |
        ↓
    Event
        |
        ↓
    Job
        |
        ↓
    Runner
        |
        ↓
    Steps
        |
        ↓
    Actions / Commands
        |
        ↓
    Build / Test / Deploy

---

# 1. What Is GitHub Actions?

GitHub Actions is a CI/CD and automation platform provided by GitHub.

It allows you to automate:

    Build
    +
    Test
    +
    Security Scanning
    +
    Packaging
    +
    Deployment
    +
    Infrastructure Automation

Example:

    Developer
        |
        ↓
    git push
        |
        ↓
    GitHub Repository
        |
        ↓
    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Docker Image
        |
        ↓
    Deploy

---

# 2. What Is CI/CD?

CI/CD means:

    CI = Continuous Integration
    CD = Continuous Delivery / Continuous Deployment

Continuous Integration means developers frequently integrate code changes into a shared repository and automatically build and test those changes.

Continuous Delivery means validated changes are kept ready for release.

Continuous Deployment means validated changes are automatically deployed to the target environment.

Example:

    Developer Push
        |
        ↓
    Build
        |
        ↓
    Unit Test
        |
        ↓
    Code Quality
        |
        ↓
    Security Scan
        |
        ↓
    Package
        |
        ↓
    Deploy

---

# 3. Why Do We Use GitHub Actions?

GitHub Actions provides automation directly inside GitHub.

Benefits:

    Native GitHub Integration
    +
    CI/CD Automation
    +
    Event-Based Workflows
    +
    Reusable Actions
    +
    Matrix Builds
    +
    Secrets Management
    +
    Environment Protection
    +
    Deployment Automation

---

# 4. What Is a Workflow?

A workflow is an automated process defined in a YAML file.

Workflow files are stored under:

    .github/workflows/

Example:

    .github/
        |
        └── workflows/
              |
              └── ci.yml

A workflow can contain:

    Trigger
    +
    Jobs
    +
    Permissions
    +
    Environment Variables
    +
    Secrets
    +
    Conditions

---

# 5. What Is a Workflow File?

A workflow file is a YAML file that defines how GitHub Actions should execute an automation process.

Example:

    name: CI

    on:
      push:
        branches:
          - main

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v4

          - run: echo "Build started"

---

# 6. Where Are GitHub Actions Workflows Stored?

Workflow files are stored in:

    .github/workflows/

Example:

    repository/
        |
        ├── src/
        ├── Dockerfile
        └── .github/
              |
              └── workflows/
                    |
                    ├── ci.yml
                    └── cd.yml

---

# 7. What Is YAML?

YAML stands for:

    YAML Ain't Markup Language

GitHub Actions workflow definitions use YAML.

Example:

    name: CI

    jobs:
      build:
        runs-on: ubuntu-latest

YAML uses:

    Indentation
    +
    Key-Value Pairs
    +
    Lists

Incorrect indentation can cause workflow validation errors.

---

# 8. What Is an Event in GitHub Actions?

An event determines when a workflow should run.

Examples:

    push
    +
    pull_request
    +
    workflow_dispatch
    +
    schedule
    +
    workflow_call
    +
    release

Example:

    on:
      push:
        branches:
          - main

This means the workflow runs when code is pushed to the main branch.

---

# 9. What Is the `push` Event?

The `push` event triggers a workflow when commits are pushed to the repository.

Example:

    on:
      push:

The workflow can run for pushes to all branches.

---

# 10. How Do You Trigger a Workflow Only for main?

Example:

    on:
      push:
        branches:
          - main

Flow:

    Developer
        |
        ↓
    Push to main
        |
        ↓
    Workflow Starts

A push to another branch will not trigger this workflow.

---

# 11. What Is the `pull_request` Event?

The `pull_request` event triggers workflows based on pull request activity.

Example:

    on:
      pull_request:
        branches:
          - main

Typical use:

    Pull Request
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Checks
        |
        ↓
    Review
        |
        ↓
    Merge

---

# 12. What Is `workflow_dispatch`?

`workflow_dispatch` allows a workflow to be manually triggered from GitHub.

Example:

    on:
      workflow_dispatch:

Typical use cases:

    Manual Deployment
    +
    Emergency Rollback
    +
    Maintenance
    +
    Operational Tasks

---

# 13. What Is a Scheduled Workflow?

A scheduled workflow runs according to a cron schedule.

Example:

    on:
      schedule:
        - cron: "0 2 * * *"

This can be used for:

    Scheduled Tests
    +
    Cleanup
    +
    Reports
    +
    Maintenance
    +
    Security Checks

---

# 14. What Is Cron?

Cron is a scheduling expression.

Example:

    0 2 * * *

This represents a scheduled execution time according to cron syntax.

Always verify the intended timezone because GitHub Actions schedules use UTC.

---

# 15. What Is a Job?

A job is a group of steps executed together on a runner.

Example:

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v4
          - run: npm install
          - run: npm test

The job contains:

    Runner
    +
    Steps

---

# 16. What Is a Step?

A step is an individual task inside a job.

Example:

    steps:
      - uses: actions/checkout@v4

      - run: npm install

      - run: npm test

Each step performs one operation.

---

# 17. What Is a Runner?

A runner is the machine that executes GitHub Actions jobs.

The runner:

    Receives Job
        |
        ↓
    Executes Steps
        |
        ↓
    Returns Results

Runners can be:

    GitHub-Hosted
    +
    Self-Hosted

---

# 18. What Is a GitHub-Hosted Runner?

GitHub provides and manages the runner infrastructure.

Example:

    runs-on: ubuntu-latest

Common options include:

    ubuntu-latest
    +
    windows-latest
    +
    macos-latest

Advantages:

    No Runner Maintenance
    +
    Easy Setup
    +
    Automatic Provisioning
    +
    GitHub Integration

---

# 19. What Is a Self-Hosted Runner?

A self-hosted runner is infrastructure managed by the organization or team.

Example:

    GitHub
        |
        ↓
    Self-Hosted Runner
        |
        ↓
    Internal Infrastructure

Useful when you need:

    Custom Software
    +
    Private Network Access
    +
    Special Hardware
    +
    Custom Configuration
    +
    Internal Systems Access

---

# 20. GitHub-Hosted vs Self-Hosted Runner

GitHub-Hosted:

    GitHub Manages Infrastructure
        |
        ↓
    Less Maintenance

Self-Hosted:

    Organization Manages Infrastructure
        |
        ↓
    More Control

The choice depends on:

    Security
    +
    Network Requirements
    +
    Customization
    +
    Cost
    +
    Maintenance

---

# 21. What Is `runs-on`?

`runs-on` specifies the runner environment where a job executes.

Example:

    jobs:
      build:
        runs-on: ubuntu-latest

This means the job runs on a GitHub-hosted Ubuntu runner.

---

# 22. What Is `uses`?

`uses` allows a workflow to use an existing GitHub Action.

Example:

    - uses: actions/checkout@v4

This uses the official checkout action.

---

# 23. What Is `run`?

`run` executes shell commands on the runner.

Example:

    - run: npm install

Another example:

    - run: |
        echo "Starting build"
        npm install
        npm test

---

# 24. Difference Between `run` and `uses`

`run`:

    Executes Commands

Example:

    - run: npm test

`uses`:

    Executes a Reusable Action

Example:

    - uses: actions/checkout@v4

---

# 25. What Does `actions/checkout` Do?

`actions/checkout` checks out repository code into the runner workspace.

Example:

    - uses: actions/checkout@v4

Without checkout, many build steps will not have the repository source code available in the workspace.

Flow:

    GitHub Repository
        |
        ↓
    actions/checkout
        |
        ↓
    Runner Workspace
        |
        ↓
    Build / Test

---

# 26. What Is `actions/checkout@v4`?

It means:

    Action:
    actions/checkout

    Version:
    v4

Example:

    - uses: actions/checkout@v4

Pinning an action to a major version helps control which action release line is used.

For stronger supply-chain controls, organizations may pin actions to specific immutable references according to their security policy.

---

# 27. What Is a Job ID?

A job ID is the identifier assigned to a job.

Example:

    jobs:
      build:
        runs-on: ubuntu-latest

Here:

    build

is the job ID.

Job IDs can be referenced by other workflow configuration.

---

# 28. Can a Workflow Have Multiple Jobs?

Yes.

Example:

    jobs:

      build:
        runs-on: ubuntu-latest

      test:
        runs-on: ubuntu-latest

      deploy:
        runs-on: ubuntu-latest

Jobs can execute independently or have dependencies.

---

# 29. How Do Jobs Run by Default?

Independent jobs can run in parallel when runner capacity is available.

Example:

    build ──────┐
                |
    test ───────┼──→
                |
    scan ───────┘

This can reduce total pipeline duration.

---

# 30. What Is `needs`?

`needs` defines a dependency between jobs.

Example:

    jobs:

      build:
        runs-on: ubuntu-latest

      deploy:
        needs: build
        runs-on: ubuntu-latest

Flow:

    Build
      |
      ↓
    Deploy

The deploy job waits for the build job to complete successfully by default.

---

# 31. Can a Job Depend on Multiple Jobs?

Yes.

Example:

    deploy:
      needs:
        - build
        - test
        - security

Flow:

    Build ───────┐
                 |
    Test ────────┼──→ Deploy
                 |
    Security ────┘

Deploy waits for all required jobs.

---

# 32. What Happens If a Required Job Fails?

If a job has:

    needs: build

and the build job fails, the dependent job is normally skipped unless its condition explicitly allows it to run.

Example:

    build
      |
      X
      |
      ↓
    deploy
      |
      ↓
    Skipped

---

# 33. What Is a Workflow Status?

A workflow can have statuses such as:

    Success
    +
    Failure
    +
    Cancelled
    +
    Skipped

These statuses help determine whether subsequent automation should run.

---

# 34. What Is a Job Status?

A job can finish as:

    Success
    +
    Failure
    +
    Cancelled
    +
    Skipped

Job status can be referenced in conditional expressions.

---

# 35. What Is an Environment Variable?

Environment variables provide configuration values to workflow execution.

Example:

    env:
      APP_ENV: production

Then:

    - run: echo "$APP_ENV"

Environment variables can exist at:

    Workflow Level
    +
    Job Level
    +
    Step Level

---

# 36. Workflow-Level Environment Variable

Example:

    name: CI

    on:
      push:

    env:
      APP_ENV: production

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - run: echo "$APP_ENV"

The variable is available throughout the workflow unless overridden.

---

# 37. Job-Level Environment Variable

Example:

    jobs:
      build:
        runs-on: ubuntu-latest

        env:
          APP_ENV: test

        steps:
          - run: echo "$APP_ENV"

The variable applies to the job.

---

# 38. Step-Level Environment Variable

Example:

    steps:
      - name: Run Test
        env:
          TEST_ENV: integration
        run: echo "$TEST_ENV"

The variable applies to that step.

---

# 39. What Are GitHub Actions Secrets?

Secrets are encrypted values used for sensitive information.

Examples:

    AWS Credentials
    +
    API Tokens
    +
    Passwords
    +
    Private Keys

Example:

    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}

Do not hardcode secrets in workflow files.

---

# 40. Why Should Secrets Not Be Hardcoded?

Bad:

    run: |
      export PASSWORD="my-password"

Problems:

    Secret Exposure
    +
    Git History Exposure
    +
    Security Risk

Better:

    env:
      PASSWORD: ${{ secrets.PASSWORD }}

---

# 41. Where Can Secrets Be Stored?

GitHub supports secrets at different scopes, including:

    Repository Secrets
    +
    Environment Secrets
    +
    Organization Secrets

Choose the scope according to the required access.

---

# 42. What Are Variables in GitHub Actions?

GitHub Actions variables provide configuration values that are not necessarily sensitive.

Example:

    ${{ vars.APP_NAME }}

Use:

    Secrets
        |
        ↓
    Sensitive Values

Use:

    Variables
        |
        ↓
    Non-Sensitive Configuration

---

# 43. Secrets vs Variables

Secrets:

    Sensitive
    +
    Encrypted
    +
    Used For Credentials / Tokens

Variables:

    Non-Sensitive Configuration
    +
    Environment Settings
    +
    Application Configuration

Never use ordinary variables for passwords or sensitive credentials.

---

# 44. What Is the GitHub Actions Context?

Contexts provide information about the workflow execution.

Examples:

    github
    +
    env
    +
    vars
    +
    secrets
    +
    runner
    +
    job
    +
    steps
    +
    needs
    +
    matrix

Example:

    ${{ github.ref }}

---

# 45. What Is the `github` Context?

The `github` context provides information about the current workflow execution and repository event.

Examples:

    ${{ github.repository }}

    ${{ github.ref }}

    ${{ github.sha }}

    ${{ github.actor }}

    ${{ github.event_name }}

---

# 46. What Is `github.sha`?

`github.sha` represents the commit SHA associated with the workflow event.

Example:

    - run: echo "${{ github.sha }}"

This is useful for:

    Build Identification
    +
    Docker Image Tagging
    +
    Deployment Tracking
    +
    Auditing

---

# 47. What Is `github.ref`?

`github.ref` identifies the Git ref associated with the workflow event.

For example, it can represent a branch or tag reference.

Example:

    - run: echo "${{ github.ref }}"

---

# 48. What Is `github.actor`?

`github.actor` identifies the GitHub user or actor that initiated the workflow event.

Example:

    - run: echo "${{ github.actor }}"

Useful for:

    Auditing
    +
    Debugging
    +
    Workflow Information

---

# 49. What Is `github.event_name`?

It identifies the event that triggered the workflow.

Example:

    push
    +
    pull_request
    +
    workflow_dispatch

Example:

    - run: echo "${{ github.event_name }}"

---

# 50. What Is an Expression in GitHub Actions?

Expressions are used to dynamically evaluate values and conditions.

Syntax:

    ${{ expression }}

Example:

    ${{ github.ref }}

Another example:

    ${{ github.event_name == 'push' }}

---

# 51. What Is `if` in GitHub Actions?

`if` conditionally executes a job or step.

Example:

    - name: Deploy
      if: github.ref == 'refs/heads/main'
      run: ./deploy.sh

This step runs only when the condition is true.

---

# 52. What Is `if: success()`?

`success()` returns true when previous required steps/jobs have succeeded.

Example:

    - name: Deploy
      if: success()
      run: ./deploy.sh

This is useful for ensuring deployment happens only after successful validation.

---

# 53. What Is `failure()`?

`failure()` returns true when a previous step or required job has failed.

Example:

    - name: Notify Failure
      if: failure()
      run: ./notify.sh

Useful for:

    Failure Notifications
    +
    Incident Automation
    +
    Debugging

---

# 54. What Is `always()`?

`always()` allows a step or job to run regardless of previous success or failure, subject to workflow execution rules.

Example:

    - name: Cleanup
      if: always()
      run: ./cleanup.sh

Useful for:

    Cleanup
    +
    Log Collection
    +
    Diagnostic Artifacts

Use carefully for critical workflows because unconditional execution can have unintended effects.

---

# 55. What Is `cancelled()`?

`cancelled()` evaluates to true when the workflow or job has been cancelled.

Example:

    - name: Cancellation Cleanup
      if: cancelled()
      run: ./cleanup.sh

---

# 56. What Is a Matrix Strategy?

A matrix allows a job to run across multiple combinations of configuration values.

Example:

    strategy:
      matrix:
        node:
          - 18
          - 20
          - 22

This creates separate job executions for each Node.js version.

Flow:

    Node 18
        |
        ├── Test

    Node 20
        |
        ├── Test

    Node 22
        |
        ├── Test

---

# 57. Why Use Matrix Builds?

Matrix builds are useful for:

    Multiple OS Versions
    +
    Multiple Runtime Versions
    +
    Multiple Database Versions
    +
    Compatibility Testing

Example:

    Ubuntu + Node 20
    Ubuntu + Node 22
    Windows + Node 20
    Windows + Node 22

---

# 58. What Is `strategy`?

`strategy` controls how a job executes multiple configurations.

Example:

    strategy:
      matrix:
        node:
          - 20
          - 22

It is commonly used for matrix builds.

---

# 59. What Is `fail-fast`?

`fail-fast` controls whether other matrix jobs should be cancelled when one matrix job fails.

Example:

    strategy:
      fail-fast: false
      matrix:
        node:
          - 20
          - 22

With:

    fail-fast: false

other matrix jobs continue even if one fails.

---

# 60. What Is `max-parallel`?

`max-parallel` limits the number of matrix jobs running simultaneously.

Example:

    strategy:
      max-parallel: 2
      matrix:
        node:
          - 18
          - 20
          - 22
          - 24

This can help control:

    Runner Consumption
    +
    Concurrency
    +
    Cost

---

# 61. What Is an Artifact?

An artifact is a file or collection of files produced by a workflow that can be stored and accessed after the job completes.

Examples:

    Build Package
    +
    Test Report
    +
    Coverage Report
    +
    Logs
    +
    Deployment Package

Typical flow:

    Build
        |
        ↓
    Artifact
        |
        ↓
    Download
        |
        ↓
    Deploy

---

# 62. What Is Artifact Upload?

A workflow can upload files as artifacts.

Conceptually:

    Build
        |
        ↓
    package.zip
        |
        ↓
    Upload Artifact
        |
        ↓
    GitHub Actions Artifact Storage

---

# 63. What Is Artifact Download?

A later job can download an artifact produced by an earlier job.

Flow:

    Build Job
        |
        ↓
    Upload Artifact
        |
        ↓
    Deploy Job
        |
        ↓
    Download Artifact
        |
        ↓
    Deploy

This avoids rebuilding the same package.

---

# 64. Artifact vs Cache

Artifact:

    Stores Workflow Output

Examples:

    Build Package
    +
    Test Report

Cache:

    Speeds Up Repeated Work

Examples:

    Maven Dependencies
    +
    npm Dependencies

Simple rule:

    Artifact = Output

    Cache = Reusable Dependency / Build Data

---

# 65. What Is Dependency Caching?

Dependency caching stores reusable dependencies between workflow executions.

Example:

    Maven Dependencies
        |
        ↓
    Cache
        |
        ↓
    Future Workflow
        |
        ↓
    Restore Cache
        |
        ↓
    Faster Build

Benefits:

    Faster Builds
    +
    Lower Download Time
    +
    Lower Runner Usage

---

# 66. What Is a Service Container?

A service container provides an additional containerized service to a job.

Examples:

    PostgreSQL
    +
    MySQL
    +
    Redis

Example architecture:

    Job Container
        |
        ↓
    Application Tests
        |
        ↓
    PostgreSQL Service

Useful for integration testing.

---

# 67. What Is a Container Job?

A job can execute inside a specified container environment.

Conceptually:

    Runner
        |
        ↓
    Container
        |
        ↓
    Job Steps

This can provide a consistent execution environment.

---

# 68. What Is a GitHub Action?

An Action is a reusable unit of automation.

It can:

    Checkout Code
    +
    Setup Runtime
    +
    Build
    +
    Test
    +
    Authenticate
    +
    Deploy

Actions can be:

    GitHub-Created
    +
    Community-Created
    +
    Organization-Created
    +
    Custom

---

# 69. What Is a Marketplace Action?

GitHub Marketplace contains reusable Actions published by different developers and organizations.

Example:

    actions/checkout

Before using third-party Actions, evaluate:

    Maintainer
    +
    Permissions
    +
    Security
    +
    Version
    +
    Source Code
    +
    Supply Chain Risk

---

# 70. What Is a Custom Action?

A custom Action is an Action created by your organization or team.

Useful when:

    Same Logic Repeated
        |
        ↓
    Create Reusable Action
        |
        ↓
    Use Across Workflows

Example:

    Internal Deployment Action
        |
        ↓
    Service A
    Service B
    Service C

---

# 71. What Is a Composite Action?

A composite Action combines multiple workflow steps into a reusable Action.

Example:

    Composite Action
        |
        +--- Checkout
        +--- Setup Runtime
        +--- Install Dependencies
        +--- Run Validation

Then multiple workflows can call the composite Action.

---

# 72. Why Use Composite Actions?

Benefits:

    Reuse
    +
    Standardization
    +
    Less Duplication
    +
    Easier Maintenance

Example:

    20 Workflows
        |
        ↓
    Same Setup Steps
        |
        ↓
    Composite Action
        |
        ↓
    Reuse Everywhere

---

# 73. What Is a Reusable Workflow?

A reusable workflow is a complete workflow that can be called by another workflow.

It is useful when entire CI/CD processes need to be standardized.

Example:

    Application Workflows
        |
        +--- Service A
        +--- Service B
        +--- Service C
        |
        ↓
    Reusable CI Workflow

---

# 74. Composite Action vs Reusable Workflow

Composite Action:

    Reuses Steps

Reusable Workflow:

    Reuses Workflow Logic

Simple rule:

    Composite Action
        |
        ↓
    Step-Level Reuse

    Reusable Workflow
        |
        ↓
    Workflow-Level Reuse

---

# 75. What Is `workflow_call`?

`workflow_call` allows one workflow to be called by another workflow.

Example:

    on:
      workflow_call:

This is commonly used for reusable workflows.

---

# 76. What Are Workflow Inputs?

Inputs allow values to be passed into reusable workflows or manually triggered workflows.

Example:

    Environment:
        |
        ↓
    dev
    qa
    prod

The workflow can use the selected input to determine behavior.

---

# 77. What Is a GitHub Actions Environment?

An Environment represents a deployment target or environment configuration.

Examples:

    development
    +
    staging
    +
    production

Environments can provide:

    Environment Secrets
    +
    Environment Variables
    +
    Protection Rules
    +
    Deployment Tracking

---

# 78. What Are Environment Protection Rules?

Protection rules can require approvals or other conditions before deployment to an environment.

Example:

    CI
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deploy

This is useful for production deployments.

---

# 79. What Is a Deployment Environment?

A deployment environment identifies where the application is being deployed.

Example:

    Development
        |
        ↓
    Staging
        |
        ↓
    Production

Each environment can have different:

    Secrets
    +
    Variables
    +
    Protection Rules

---

# 80. What Is a GitHub Token?

GitHub Actions can provide the workflow with a temporary token through:

    secrets.GITHUB_TOKEN

It is used for interacting with GitHub resources according to its permissions.

Example:

    ${{ secrets.GITHUB_TOKEN }}

---

# 81. What Are GITHUB_TOKEN Permissions?

Permissions control what the workflow token can do.

Example:

    permissions:
      contents: read

This follows the principle:

    Minimum Required Permission

Do not give workflows broader permissions than necessary.

---

# 82. What Is the Principle of Least Privilege?

Least privilege means providing only the permissions required to perform a task.

Example:

    Build Job
        |
        ↓
    Read Repository
        |
        ↓
    contents: read

A deployment job may require additional permissions depending on the deployment method.

---

# 83. Why Is Least Privilege Important in GitHub Actions?

A compromised workflow can potentially use available permissions.

If permissions are excessive:

    Compromised Workflow
        |
        ↓
    Excessive Access
        |
        ↓
    Higher Security Impact

With least privilege:

    Compromised Workflow
        |
        ↓
    Limited Permissions
        |
        ↓
    Reduced Impact

---

# 84. What Is OIDC in GitHub Actions?

OIDC stands for:

    OpenID Connect

GitHub Actions can use OIDC to obtain short-lived cloud credentials instead of storing long-lived cloud access keys as GitHub secrets.

Conceptually:

    GitHub Actions
        |
        ↓
    OIDC Identity Token
        |
        ↓
    Cloud IAM
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS Resources

---

# 85. Why Is OIDC Preferred Over Long-Lived Cloud Keys?

Long-lived credentials create risks:

    Credential Leakage
    +
    Manual Rotation
    +
    Long Validity
    +
    Larger Blast Radius

OIDC provides:

    Short-Lived Credentials
    +
    Identity-Based Access
    +
    Reduced Secret Management
    +
    Better Security

---

# 86. What Is a GitHub Actions Workflow Permission?

The `permissions` block defines the access available to the `GITHUB_TOKEN`.

Example:

    permissions:
      contents: read

You can define permissions at:

    Workflow Level
    +
    Job Level

---

# 87. What Is Concurrency?

Concurrency controls how workflow runs or jobs are handled when multiple executions happen at the same time.

Example scenario:

    Push 1
        |
        ↓
    Workflow Running

    Push 2
        |
        ↓
    Another Workflow

Concurrency can be used to avoid unnecessary overlapping deployments.

---

# 88. Why Is Concurrency Useful for Deployments?

Example:

    Deployment A
        |
        ↓
    Production

Before it finishes:

    Deployment B
        |
        ↓
    Production

Running both simultaneously can cause deployment conflicts.

Concurrency can ensure:

    One Production Deployment
        |
        ↓
    At A Time

---

# 89. What Is a Workflow Trigger Filter?

Filters restrict when a workflow runs.

Examples:

    Branch Filters
    +
    Path Filters
    +
    Tag Filters

Example:

    on:
      push:
        branches:
          - main

Only pushes to main trigger the workflow.

---

# 90. What Are Path Filters?

Path filters trigger workflows based on changed files.

Example:

    on:
      push:
        paths:
          - "src/**"

This can prevent unnecessary workflows when unrelated files change.

---

# 91. Why Use Path Filters?

Example:

    Documentation Change
        |
        ↓
    No Application Source Change
        |
        ↓
    Skip Expensive Build

Benefits:

    Lower Runner Usage
    +
    Faster Feedback
    +
    Lower CI/CD Cost

---

# 92. What Is Branch Protection?

Branch protection controls how changes can be merged into important branches.

Example:

    Pull Request
        |
        ↓
    Required Review
        |
        ↓
    Required Status Checks
        |
        ↓
    Merge

Commonly protected branch:

    main

---

# 93. How Do GitHub Actions Integrate With Pull Requests?

Typical flow:

    Developer Creates PR
        |
        ↓
    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Unit Tests
        |
        ↓
    Security Checks
        |
        ↓
    Status Check
        |
        ↓
    PR Review
        |
        ↓
    Merge

---

# 94. What Are Status Checks?

Status checks indicate whether automated validation passed or failed.

Examples:

    Build
    +
    Unit Tests
    +
    Integration Tests
    +
    Security Scan

Protected branches can require these checks before merging.

---

# 95. What Is Continuous Integration Using GitHub Actions?

Typical CI workflow:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    Build
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Result
        |
        ↓
    Merge

---

# 96. What Is Continuous Deployment Using GitHub Actions?

Typical CD flow:

    Code Merge
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Docker Image
        |
        ↓
    Registry
        |
        ↓
    Deployment
        |
        ↓
    Validation

---

# 97. How Can GitHub Actions Deploy to AWS?

Common approaches include:

    OIDC
    +
    AWS CLI
    +
    AWS Actions
    +
    Terraform
    +
    Helm
    +
    Kubernetes

Example:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    AWS IAM Role
        |
        ↓
    AWS
        |
        ↓
    EKS / EC2 / S3 / Other Services

---

# 98. How Can GitHub Actions Deploy to Kubernetes?

Typical flow:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push Image to ECR
        |
        ↓
    Update Deployment / Manifest
        |
        ↓
    Kubernetes
        |
        ↓
    Pods

GitOps can also be used:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Update Git Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# 99. What Is a Deployment Pipeline?

A deployment pipeline is an automated sequence that moves software from source code to a deployed environment.

Example:

    Developer
        |
        ↓
    Git Push
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security
        |
        ↓
    Package
        |
        ↓
    Deploy
        |
        ↓
    Validate

---

# 100. What Is a Typical GitHub Actions DevOps Pipeline?

A practical DevOps pipeline can look like:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    Checkout
        |
        ↓
    Maven / npm Build
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Docker Build
        |
        ↓
    Push Image to ECR
        |
        ↓
    Update Deployment Configuration
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Health Checks
        |
        ↓
    Production

---

# 101. How Do You Debug a Failed GitHub Actions Workflow?

Start with:

    Workflow
        |
        ↓
    Failed Job
        |
        ↓
    Failed Step
        |
        ↓
    Error Message
        |
        ↓
    Logs
        |
        ↓
    Reproduce Locally
        |
        ↓
    Fix
        |
        ↓
    Re-run

Check:

    YAML Syntax
    +
    Runner
    +
    Permissions
    +
    Secrets
    +
    Environment Variables
    +
    Dependencies
    +
    Commands
    +
    Network
    +
    External Services

---

# 102. What Should You Check If a Workflow Does Not Start?

Check:

    Workflow File Location
        |
        ↓
    .github/workflows/
        |
        ↓
    YAML Syntax
        |
        ↓
    Event Trigger
        |
        ↓
    Branch Filter
        |
        ↓
    Path Filter
        |
        ↓
    Workflow Enabled
        |
        ↓
    Event Conditions

---

# 103. What Should You Check If a Job Is Queued?

Check:

    Runner Availability
    +
    Runner Labels
    +
    Runner Capacity
    +
    Organization Limits
    +
    Concurrency
    +
    Self-Hosted Runner Status

For self-hosted runners:

    Runner Online?
        |
        ↓
    Correct Labels?
        |
        ↓
    Runner Available?
        |
        ↓
    Job Starts

---

# 104. What Should You Check If a Secret Is Not Working?

Check:

    Secret Name
    +
    Secret Scope
    +
    Environment
    +
    Repository Access
    +
    Organization Policy
    +
    Workflow Context

Example:

    ${{ secrets.AWS_ROLE_ARN }}

Verify that:

    AWS_ROLE_ARN

actually exists in the expected secret scope.

---

# 105. What Should You Check If AWS Authentication Fails?

Check:

    OIDC Configuration
    +
    IAM Role
    +
    Trust Policy
    +
    GitHub Repository
    +
    Branch / Environment Conditions
    +
    Permissions
    +
    AWS Region
    +
    Credentials Configuration

Typical flow:

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust
        |
        ↓
    Assume Role
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS API

---

# 106. What Is a Self-Hosted Runner Security Concern?

Self-hosted runners require careful security because workflow code may execute commands on the runner.

Risks include:

    Malicious Pull Requests
    +
    Secret Exposure
    +
    Persistent Runner State
    +
    Privilege Escalation
    +
    Network Access

Use:

    Least Privilege
    +
    Isolation
    +
    Ephemeral Runners Where Appropriate
    +
    Restricted Network Access
    +
    Trusted Workflows

---

# 107. What Is an Ephemeral Runner?

An ephemeral runner is a runner that is created for a job or short-lived execution and then removed.

Flow:

    Job
        |
        ↓
    Create Runner
        |
        ↓
    Execute Job
        |
        ↓
    Destroy Runner

Benefits:

    Clean Environment
    +
    Reduced Persistence
    +
    Better Isolation
    +
    Reduced Cross-Job Contamination

---

# 108. What Is CI/CD Pipeline Parallelism?

Parallelism means executing independent tasks simultaneously.

Example:

    Build
        |
        +--- Unit Test
        |
        +--- Security Scan
        |
        +--- Lint
        |
        +--- Integration Test

This can reduce total pipeline duration.

---

# 109. How Do You Reduce GitHub Actions Pipeline Time?

Approaches:

    Parallelize Independent Jobs
    +
    Dependency Caching
    +
    Docker Layer Caching
    +
    Smaller Docker Images
    +
    Avoid Unnecessary Workflows
    +
    Path Filters
    +
    Reusable Workflows
    +
    Faster Runners
    +
    Optimize Tests

Goal:

    Long Pipeline
        |
        ↓
    Identify Bottlenecks
        |
        ↓
    Optimize
        |
        ↓
    Shorter Pipeline

---

# 110. What Is Pipeline Bottleneck?

A bottleneck is the stage that significantly limits pipeline performance.

Example:

    Checkout       = 10 sec
    Build          = 2 min
    Unit Test      = 1 min
    Security Scan  = 15 min
    Docker Build   = 2 min

Bottleneck:

    Security Scan

Optimization should focus on the largest contributors first.

---

# 111. What Is a Deployment Approval?

A deployment approval requires a person or protection rule to approve a deployment before it proceeds.

Example:

    CI
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Production
        |
        ↓
    Approval
        |
        ↓
    Deploy

Useful for sensitive production environments.

---

# 112. What Is Manual Approval Used For?

Common examples:

    Production Deployment
    +
    Database Migration
    +
    High-Risk Change
    +
    Infrastructure Change

---

# 113. What Is a Rollback?

Rollback means returning the application to a previously known-good version.

Example:

    Version 10
        |
        ↓
    Deployment
        |
        ↓
    Errors
        |
        ↓
    Rollback
        |
        ↓
    Version 9

GitHub Actions can automate rollback workflows.

---

# 114. What Is a Manual Rollback Workflow?

A workflow can use:

    workflow_dispatch

to allow an operator to manually select or provide a version to deploy.

Example:

    Production
        |
        ↓
    Issue Detected
        |
        ↓
    Manual Workflow
        |
        ↓
    Select Previous Version
        |
        ↓
    Deploy
        |
        ↓
    Validate

---

# 115. What Is GitHub Actions Artifact Retention?

Artifacts can be retained for a defined period.

Retention should balance:

    Debugging
    +
    Compliance
    +
    Rollback Requirements
    +
    Storage Cost

Avoid keeping unnecessary artifacts indefinitely.

---

# 116. What Is a Workflow Run?

A workflow run is one execution instance of a workflow.

Example:

    Push
        |
        ↓
    Workflow Run #101

Another push:

    Push
        |
        ↓
    Workflow Run #102

Each run has its own:

    Jobs
    +
    Logs
    +
    Status
    +
    Artifacts

---

# 117. What Is a Job Summary?

GitHub Actions can display summary information for workflow runs.

Useful for:

    Test Results
    +
    Deployment Information
    +
    Security Findings
    +
    Build Details

This makes pipeline results easier to understand.

---

# 118. What Is the Difference Between CI and CD in GitHub Actions?

CI focuses on:

    Build
    +
    Test
    +
    Validate

CD focuses on:

    Release
    +
    Deploy
    +
    Validate Production

Example:

    CI:

    Commit
      ↓
    Build
      ↓
    Test

    CD:

    Approved Artifact
      ↓
    Deploy
      ↓
    Validate

---

# 119. What Is GitHub Actions Used for in DevOps?

GitHub Actions can automate the complete DevOps lifecycle:

    Code
      |
      ↓
    Build
      |
      ↓
    Test
      |
      ↓
    Security
      |
      ↓
    Package
      |
      ↓
    Infrastructure
      |
      ↓
    Deploy
      |
      ↓
    Monitor
      |
      ↓
    Feedback

---

# 120. Basic Interview Quick Revision

Remember these core concepts:

    GitHub Actions
        |
        ↓
    Automation Platform

    Workflow
        |
        ↓
    YAML Automation Definition

    Event
        |
        ↓
    Workflow Trigger

    Job
        |
        ↓
    Group Of Steps

    Step
        |
        ↓
    Individual Task

    Runner
        |
        ↓
    Machine Executing Job

    Action
        |
        ↓
    Reusable Automation Component

    Secret
        |
        ↓
    Sensitive Configuration

    Variable
        |
        ↓
    Non-Sensitive Configuration

    Artifact
        |
        ↓
    Workflow Output

    Cache
        |
        ↓
    Reusable Build / Dependency Data

    Environment
        |
        ↓
    Deployment Target + Protection

    Matrix
        |
        ↓
    Multiple Configurations

    needs
        |
        ↓
    Job Dependency

    if
        |
        ↓
    Conditional Execution

    permissions
        |
        ↓
    Workflow Token Access

    OIDC
        |
        ↓
    Short-Lived Cloud Authentication

    workflow_dispatch
        |
        ↓
    Manual Execution

    workflow_call
        |
        ↓
    Reusable Workflow

---

# 121. Basic Interview Answer Structure

For basic GitHub Actions interview questions, answer using:

    Definition
        +
    Purpose
        +
    Example
        +
    Real-World Usage

Example:

Question:

    What is HPA?

Answer structure:

    Definition:
    HPA automatically adjusts Kubernetes pod replicas based on
    configured metrics.

    Purpose:
    It allows applications to scale according to workload.

    Example:
    If CPU increases above the configured target, HPA can increase
    replicas.

    Real-World Usage:
    I would use HPA for stateless microservices running on EKS and
    combine it with node-level autoscaling when additional node
    capacity is required.

---

# 122. Final Basic Interview Preparation

Before moving to intermediate questions, you should be comfortable explaining:

    What Is GitHub Actions?

    What Is CI/CD?

    What Is A Workflow?

    What Is YAML?

    What Is An Event?

    What Is A Job?

    What Is A Step?

    What Is A Runner?

    GitHub-Hosted vs Self-Hosted Runners

    What Is An Action?

    What Is `uses`?

    What Is `run`?

    What Is `needs`?

    What Is `if`?

    What Are Secrets?

    What Are Variables?

    What Are Contexts?

    What Is A Matrix?

    What Are Artifacts?

    What Is Caching?

    What Are Environments?

    What Is OIDC?

    What Is GITHUB_TOKEN?

    What Is Least Privilege?

    What Are Reusable Workflows?

    What Are Composite Actions?

    What Is workflow_dispatch?

    What Is workflow_call?

    What Is Concurrency?

    How Do You Debug Failed Workflows?

    How Do You Optimize Pipeline Performance?

    How Do You Deploy To AWS?

    How Do You Deploy To Kubernetes?

    How Do You Implement Secure CI/CD?

---

# 123. Final Concept

GitHub Actions is more than just:

    Run Commands After Git Push

A production-grade GitHub Actions implementation combines:

    GitHub
        +
    Workflows
        +
    Events
        +
    Jobs
        +
    Runners
        +
    Actions
        +
    Secrets
        +
    Environments
        +
    Permissions
        +
    OIDC
        +
    Artifacts
        +
    Caching
        +
    Security
        +
    Deployment
        +
    Monitoring

The goal is:

    CODE
      |
      ↓
    AUTOMATED BUILD
      |
      ↓
    AUTOMATED TEST
      |
      ↓
    SECURITY VALIDATION
      |
      ↓
    PACKAGE
      |
      ↓
    SECURE DEPLOYMENT
      |
      ↓
    VALIDATION
      |
      ↓
    RELIABLE PRODUCTION