# JavaScript Actions

A **JavaScript Action** is a GitHub Action implemented using JavaScript or TypeScript and executed using the Node.js runtime provided by GitHub Actions.

Conceptually:

```text
GitHub Workflow
       |
       ↓
JavaScript Action
       |
       ↓
Node.js Runtime
       |
       ↓
JavaScript / TypeScript Code
```

JavaScript Actions are useful when an Action needs more logic than a simple shell command or Composite Action can conveniently provide.

---

# Why JavaScript Actions?

JavaScript Actions are useful for:

```text
API Integration
GitHub API Operations
Complex Logic
Input Processing
Output Generation
File Processing
Custom Automation
Reusable DevOps Tooling
```

Example:

```text
Workflow
   |
   ↓
JavaScript Action
   |
   ├── Read Inputs
   ├── Call API
   ├── Process Data
   ├── Apply Logic
   └── Set Outputs
```

---

# JavaScript Action vs Composite Action

### Composite Action

```text
YAML
 |
 ├── Shell Commands
 └── Existing Actions
```

### JavaScript Action

```text
JavaScript / TypeScript
 |
 ├── Logic
 ├── APIs
 ├── File Processing
 └── Outputs
```

Use JavaScript when complex programmatic logic is easier to maintain than shell/YAML logic.

---

# JavaScript Action vs Docker Action

### JavaScript Action

```text
Runner
 |
 └── Node.js
      |
      └── JavaScript Action
```

### Docker Action

```text
Runner
 |
 └── Docker Container
      |
      └── Action
```

JavaScript Actions generally avoid the need to build and start a Docker container for the Action itself.

---

# Basic JavaScript Action Structure

A typical repository can look like:

```text
.github/
└── actions/
    └── example/
        ├── action.yml
        ├── src/
        │   └── main.ts
        ├── package.json
        ├── tsconfig.json
        └── dist/
            └── index.js
```

Common files:

```text
action.yml
package.json
src/
dist/
```

---

# `action.yml`

Example:

```yaml
name: Repository Information

description: Display repository information

runs:
  using: node24
  main: dist/index.js
```

The exact supported Node runtime should follow the current GitHub Actions runtime support and your organization's compatibility policy.

---

# `main`

The `main` property identifies the JavaScript file that GitHub Actions executes.

Example:

```yaml
runs:
  using: node24
  main: dist/index.js
```

Flow:

```text
Workflow
   |
   ↓
action.yml
   |
   ↓
main
   |
   ↓
dist/index.js
```

---

# Source Code vs Distribution Code

Many JavaScript Actions are developed using TypeScript.

Example:

```text
src/main.ts
```

is compiled into:

```text
dist/index.js
```

The Action executes the distribution file.

Conceptually:

```text
TypeScript
    |
    ↓
Build
    |
    ↓
JavaScript
    |
    ↓
dist/index.js
```

---

# Why `dist/` Matters

The GitHub Actions runner needs the executable distribution file referenced by `action.yml`.

Example:

```yaml
main: dist/index.js
```

Therefore, after changing:

```text
src/main.ts
```

the distribution output needs to be rebuilt and updated according to the Action's release process.

---

# `package.json`

Example:

```json
{
  "name": "repository-info-action",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "build": "ncc build src/main.ts --minify"
  },
  "dependencies": {
    "@actions/core": "^1.11.1",
    "@actions/github": "^6.0.0"
  },
  "devDependencies": {
    "@vercel/ncc": "^0.38.3",
    "typescript": "^5.0.0"
  }
}
```

The exact dependency versions should be selected and maintained according to the project's dependency policy.

---

# `@actions/core`

The `@actions/core` package provides utilities commonly used when developing GitHub Actions.

Typical functionality includes:

```text
Get Inputs
Set Outputs
Logging
Warnings
Errors
Environment Variables
Secrets Masking
Exit Codes
```

Example:

```javascript
const core = require('@actions/core');

const name = core.getInput('name');

core.info(`Hello ${name}`);
```

---

# `@actions/github`

The `@actions/github` package provides helpers for interacting with GitHub APIs and context.

Example concept:

```javascript
const github = require('@actions/github');
```

It can be used together with:

```text
GITHUB_TOKEN
```

to interact with GitHub APIs when the workflow has the required permissions.

---

# Inputs

JavaScript Actions can define inputs in `action.yml`.

Example:

```yaml
name: Greeting Action

inputs:

  name:
    description: Name to greet
    required: true

runs:
  using: node24
  main: dist/index.js
```

---

# Reading Inputs

JavaScript:

```javascript
const core = require('@actions/core');

const name = core.getInput('name');

core.info(`Hello ${name}`);
```

Workflow:

```yaml
- name: Greeting
  uses: ./.github/actions/greeting
  with:
    name: Surendra
```

Flow:

```text
Workflow
   |
   ↓
name = Surendra
   |
   ↓
JavaScript Action
   |
   ↓
core.getInput()
```

---

# Required Inputs

Example:

```yaml
inputs:

  environment:
    description: Target environment
    required: true
```

If the required input is missing, the Action should fail clearly.

---

# Optional Inputs

Example:

```yaml
inputs:

  environment:
    description: Target environment
    required: false
    default: qa
```

If the caller does not provide the value:

```text
environment = qa
```

---

# Input Validation

Do not blindly trust inputs.

Example:

```javascript
const core = require('@actions/core');

const environment = core.getInput('environment');

const allowed = ['dev', 'qa', 'uat', 'production'];

if (!allowed.includes(environment)) {
  core.setFailed(`Invalid environment: ${environment}`);
  return;
}
```

This prevents unsupported values from reaching deployment logic.

---

# Outputs

JavaScript Actions can expose outputs.

`action.yml`:

```yaml
outputs:

  image-tag:
    description: Generated image tag

runs:
  using: node24
  main: dist/index.js
```

JavaScript:

```javascript
core.setOutput('image-tag', process.env.GITHUB_SHA);
```

---

# Consuming Outputs

Workflow:

```yaml
- name: Generate Image Tag
  id: image
  uses: ./.github/actions/image-tag

- name: Display Tag
  run: |
    echo "${{ steps.image.outputs.image-tag }}"
```

Flow:

```text
JavaScript Action
       |
       ↓
setOutput()
       |
       ↓
Workflow
       |
       ↓
steps.image.outputs.image-tag
```

---

# Logging

Use Action logging utilities instead of relying only on `console.log`.

Example:

```javascript
core.info('Starting build');
core.warning('This is a warning');
core.error('An error occurred');
```

Useful log levels:

```text
info
warning
error
debug
```

---

# Debug Logging

Example:

```javascript
core.debug('Debug information');
```

Debug information can be enabled when troubleshooting.

Do not put secrets into debug messages.

---

# Secret Masking

If sensitive values must appear in logs indirectly, use appropriate masking mechanisms.

Example:

```javascript
core.setSecret(secret);
```

Never intentionally print credentials.

---

# Failure Handling

Use:

```javascript
core.setFailed('Deployment failed');
```

Example:

```javascript
try {
  // operation
} catch (error) {
  core.setFailed(`Operation failed: ${error.message}`);
}
```

This causes the Action step to fail.

---

# Exit Codes

Conceptually:

```text
Success
   ↓
Exit 0

Failure
   ↓
Non-zero
```

GitHub Actions uses the result to determine whether the step succeeded.

---

# Environment Variables

JavaScript Actions can access environment variables.

Example:

```javascript
const repository = process.env.GITHUB_REPOSITORY;
const sha = process.env.GITHUB_SHA;

core.info(`Repository: ${repository}`);
core.info(`Commit: ${sha}`);
```

Common GitHub environment variables include:

```text
GITHUB_REPOSITORY
GITHUB_SHA
GITHUB_REF
GITHUB_WORKSPACE
GITHUB_ACTOR
GITHUB_EVENT_NAME
```

---

# GitHub Context

GitHub Actions provides context information.

Example:

```javascript
const github = require('@actions/github');

const context = github.context;

core.info(`Repository: ${context.repo.owner}/${context.repo.repo}`);
```

The context can provide:

```text
Repository
Owner
Event
Ref
SHA
Issue
Pull Request
Actor
```

depending on the event.

---

# GitHub API

JavaScript Actions can interact with GitHub APIs.

Example concept:

```text
JavaScript Action
       |
       ↓
GitHub API
       |
       ↓
Repository / PR / Issues / Releases
```

This is one reason JavaScript Actions can be useful for automation.

---

# GitHub Token

A workflow can provide:

```text
GITHUB_TOKEN
```

Example:

```yaml
permissions:
  contents: read
```

The Action should receive only the permissions it actually needs.

---

# API Authentication

Example:

```javascript
const token = core.getInput('token', { required: true });

const octokit = github.getOctokit(token);
```

The token should normally be supplied securely.

For example:

```yaml
with:
  token: ${{ secrets.GITHUB_TOKEN }}
```

or, when appropriate, the built-in GitHub token can be used through supported Action patterns.

---

# Least Privilege

If the Action only needs to read repository information:

```yaml
permissions:
  contents: read
```

Do not grant:

```yaml
contents: write
```

unless the Action genuinely needs to modify repository contents.

---

# Job-Level Permissions

If only one job needs write access:

```yaml
jobs:

  release:

    permissions:
      contents: write
```

Other jobs can remain restricted.

This reduces blast radius.

---

# JavaScript Action for GitHub API

Example concept:

```javascript
const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  const token = core.getInput('token', { required: true });

  const client = github.getOctokit(token);

  const { data } = await client.rest.repos.get({
    owner: github.context.repo.owner,
    repo: github.context.repo.repo
  });

  core.info(`Repository: ${data.full_name}`);
}

run().catch(error => {
  core.setFailed(error.message);
});
```

---

# API Error Handling

Always handle API failures.

Example:

```javascript
try {
  const { data } = await client.rest.repos.get({
    owner,
    repo
  });

  core.info(data.full_name);
} catch (error) {
  core.setFailed(`GitHub API request failed: ${error.message}`);
}
```

Possible failures:

```text
401 Unauthorized
403 Forbidden
404 Not Found
429 Rate Limited
5xx Server Error
```

---

# Retry Logic

For transient failures, controlled retry logic can be useful.

Conceptually:

```text
API Request
    |
    ↓
Failure?
   / \
 NO   YES
 |      |
 ↓      ↓
Done   Retry
          |
          ↓
      Retry Limit
          |
          ↓
        Fail
```

Do not retry indefinitely.

---

# API Rate Limits

GitHub APIs have rate limits.

A production JavaScript Action should consider:

```text
Rate Limit
Retry
Backoff
Caching
Request Count
```

Avoid unnecessary API calls.

---

# JavaScript Action for Dependabot

A JavaScript Action could query repository security information.

Conceptually:

```text
Workflow
   |
   ↓
JavaScript Action
   |
   ↓
GitHub API
   |
   ↓
Dependabot Alerts
   |
   ↓
Policy
   |
   ↓
Pass / Fail
```

This is useful for automated security gates.

---

# JavaScript Action Example: Critical Alerts

Conceptually:

```javascript
const alerts = await getDependabotAlerts();

const critical = alerts.filter(
  alert => alert.security_advisory?.severity === 'critical'
);

if (critical.length > 0) {
  core.setFailed(
    `Found ${critical.length} critical Dependabot alert(s)`
  );
}
```

The exact API implementation should follow the current GitHub API contract.

---

# JavaScript Action for Pull Requests

A custom Action can process:

```text
Pull Request
   |
   ↓
Changed Files
   |
   ↓
Policy
   |
   ↓
Validation
```

Examples:

```text
Require CODEOWNERS
Check changed directories
Validate labels
Check PR metadata
Enforce naming conventions
```

---

# JavaScript Action for Change Management

For a production deployment workflow:

```text
Workflow
   |
   ↓
JavaScript Action
   |
   ↓
JIRA API
   |
   ↓
Change Request
   |
   ↓
Approved?
   |
  / \
 NO  YES
 |     |
Fail  Continue
```

This is useful when deployment must satisfy organizational change-control requirements.

---

# Production Deployment Example

Your production workflow could conceptually perform:

```text
Input:
JIRA ticket
Version / SHA
Environment
```

Then:

```text
JavaScript Action
      |
      ├── Query JIRA
      ├── Verify status
      ├── Verify deployment window
      ├── Validate version
      └── Return result
```

Then:

```text
Validation Passed
       |
       ↓
Production Deployment
```

---

# JavaScript Action for Deployment Window

Conceptually:

```javascript
if (currentTime < startTime || currentTime > endTime) {
  core.setFailed('Deployment is outside the approved window');
}
```

A production implementation should use a reliable time source and clearly defined timezone rules.

---

# JavaScript Action for Approval Validation

Conceptually:

```text
JIRA Ticket
    |
    ↓
Status
    |
    ↓
Approved?
    |
 ┌──┴──┐
 NO    YES
 |      |
Fail   Continue
```

This can be integrated into a production deployment workflow.

---

# JavaScript Action for Commit Validation

A deployment Action can validate:

```text
Expected SHA
Actual SHA
Artifact SHA
Deployment SHA
```

Example:

```text
Git Commit
    |
    ↓
Build Artifact
    |
    ↓
Container Image
    |
    ↓
Deployment
```

The SHA should remain traceable throughout the pipeline.

---

# Traceability

Production pipeline:

```text
Commit SHA
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
Deployment
```

A JavaScript Action can help validate metadata at different stages.

---

# JavaScript Action for Release Validation

Example:

```text
Release Workflow
      |
      ↓
JavaScript Action
      |
      ├── Check tag
      ├── Check branch
      ├── Check commit
      ├── Check approvals
      └── Check required status
      |
      ↓
Release
```

---

# JavaScript Action for GitHub Checks

JavaScript Actions can integrate with GitHub APIs to inspect or interact with repository checks and workflow-related information, subject to the permissions available.

Conceptually:

```text
Workflow
   |
   ↓
JavaScript Action
   |
   ↓
GitHub API
   |
   ↓
Checks / PR / Repository
```

---

# JavaScript Action for Release Automation

Example:

```text
Merge to Main
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
JavaScript Release Action
      |
      ├── Determine Version
      ├── Create Release
      └── Publish Metadata
```

---

# JavaScript Action for Notifications

A JavaScript Action can integrate with:

```text
Slack
Microsoft Teams
Email APIs
JIRA
ServiceNow
Custom APIs
```

Example:

```text
Deployment
   |
   ↓
JavaScript Action
   |
   ↓
Notification API
   |
   ↓
Team
```

---

# API Integration Pattern

```text
Input
  |
  ↓
Validate
  |
  ↓
Authenticate
  |
  ↓
API Request
  |
  ↓
Process Response
  |
  ↓
Policy
  |
  ↓
Output / Fail
```

This is a strong use case for JavaScript Actions.

---

# JavaScript Action for Policy Enforcement

Example:

```text
Pull Request
      |
      ↓
Policy Action
      |
      ├── Branch Check
      ├── Label Check
      ├── Required Review
      └── File Policy
      |
      ↓
Pass / Fail
```

---

# JavaScript Action and Security

JavaScript Actions execute code in the workflow environment.

Therefore:

```text
JavaScript Action
      |
      ↓
Runner
      |
      ↓
Permissions
      |
      ↓
Secrets / APIs / Infrastructure
```

Treat third-party JavaScript Actions as executable dependencies.

---

# Third-Party JavaScript Action Security

Before using one:

```text
Review Source
Review Dependencies
Review Maintainer
Review Releases
Review Permissions
Review Secrets
Review Network Access
Review Security History
```

---

# Dependency Security

JavaScript Actions commonly use npm packages.

Dependency chain:

```text
Action
 |
 ├── npm package A
 ├── npm package B
 └── npm package C
```

A vulnerable dependency can affect the Action.

---

# Lockfile

Use a dependency lockfile such as:

```text
package-lock.json
```

when appropriate.

This helps make dependency resolution more predictable.

---

# Dependency Updates

Treat npm dependencies as part of the Action's software supply chain.

Process:

```text
Dependency Update
      |
      ↓
Tests
      |
      ↓
Security Scan
      |
      ↓
Review
      |
      ↓
Release
```

---

# Bundle Dependencies

A JavaScript Action commonly bundles its runtime dependencies into the distribution artifact.

Conceptually:

```text
src/
  |
  ↓
Build / Bundle
  |
  ↓
dist/index.js
```

This helps ensure the Action can execute using the expected dependencies.

---

# `ncc`

A common bundling tool for JavaScript/TypeScript Actions is:

```text
@vercel/ncc
```

Example:

```json
{
  "scripts": {
    "build": "ncc build src/main.ts --minify"
  }
}
```

This can produce:

```text
dist/index.js
```

---

# Development Workflow

A JavaScript Action project can follow:

```text
Write Code
   |
   ↓
npm install
   |
   ↓
Unit Tests
   |
   ↓
Build
   |
   ↓
Bundle
   |
   ↓
GitHub Actions Test
   |
   ↓
Release
```

---

# Unit Testing

Test JavaScript logic independently.

Example:

```text
Input
  |
  ↓
Function
  |
  ↓
Expected Output
```

Test:

```text
Valid Input
Invalid Input
API Success
API Failure
Missing Data
Edge Cases
```

---

# Integration Testing

Unit tests are not enough.

Test the Action inside a real workflow.

Example:

```text
Pull Request
    |
    ↓
Test Workflow
    |
    ↓
JavaScript Action
    |
    ↓
GitHub API / External API
```

---

# Mock External APIs

For unit tests, mock external API calls where appropriate.

This avoids:

```text
Network Dependency
API Rate Limits
Real Production Changes
```

---

# Production Action Testing

Use multiple layers:

```text
Lint
 ↓
Unit Test
 ↓
Build
 ↓
Dependency Scan
 ↓
Integration Test
 ↓
Workflow Test
 ↓
Release
```

---

# JavaScript Action Release Process

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
Tests
      |
      ↓
Security Scan
      |
      ↓
Review
      |
      ↓
Merge
      |
      ↓
Release
      |
      ↓
Version Tag
```

---

# Semantic Versioning

For a reusable Action:

```text
v1.0.0
v1.1.0
v1.1.1
v2.0.0
```

General interpretation:

```text
MAJOR → Breaking changes
MINOR → Backward-compatible features
PATCH → Backward-compatible fixes
```

---

# Major Version Tags

Consumers may use:

```yaml
uses: company/action@v1
```

The Action maintainer can move the `v1` major tag to approved compatible releases.

For stronger immutability, consumers may pin to a commit SHA according to organizational policy.

---

# JavaScript Action and SHA Pinning

Example:

```yaml
uses: company/action@<commit-sha>
```

Benefits:

```text
Known Code
Predictable Execution
Supply Chain Protection
```

Trade-off:

```text
Manual / automated update process required
```

---

# JavaScript Action Performance

JavaScript Actions can be efficient for logic-heavy operations because they execute directly using the Node runtime rather than starting a Docker container for each Action.

Performance depends on:

```text
API Calls
Dependencies
File Processing
Network
CPU
Action Logic
```

---

# Avoid Unnecessary API Calls

Bad:

```text
Call GitHub API
Call GitHub API
Call GitHub API
Call GitHub API
```

Better:

```text
Get Required Data
      |
      ↓
Process Locally
```

Use caching or batching where appropriate.

---

# API Backoff

For transient API failures:

```text
Request
  |
  ↓
429 / Temporary Error
  |
  ↓
Wait
  |
  ↓
Retry
```

Use bounded exponential backoff where appropriate.

---

# Timeouts

External API requests should have reasonable timeouts.

Do not allow:

```text
API Request
     |
     ↓
Hanging indefinitely
```

A production Action should fail predictably.

---

# JavaScript Action Error Handling

Good:

```javascript
try {
  await deploy();
} catch (error) {
  core.setFailed(`Deployment failed: ${error.message}`);
}
```

Better production handling may distinguish:

```text
Validation Error
Authentication Error
Authorization Error
Not Found
Rate Limit
Transient Error
Unexpected Error
```

---

# Production JIRA Validation Action

Conceptual flow:

```text
Input:
JIRA Ticket
Environment
Version
Deployment Window
```

Then:

```text
Validate Ticket
       |
       ↓
Check Status
       |
       ↓
Check Approval
       |
       ↓
Check Window
       |
       ↓
Check Version
       |
       ↓
PASS / FAIL
```

---

# Production Deployment Gate

Example:

```text
Developer
    |
    ↓
GitHub Workflow
    |
    ↓
JavaScript Validation Action
    |
    ├── JIRA Status
    ├── Approval
    ├── Deployment Window
    └── Commit SHA
    |
    ↓
Production Deployment
```

This is an example of where JavaScript logic can be more appropriate than a collection of shell commands.

---

# JavaScript Action for Dependabot Policy

Concept:

```text
Dependabot Alerts
       |
       ↓
JavaScript Action
       |
       ├── Critical Count
       ├── High Count
       └── Policy
       |
       ↓
PASS / FAIL
```

For example:

```text
Critical > 0
    |
    ↓
Fail workflow
```

---

# JavaScript Action for CR Process

Production change request flow:

```text
Input
 ├── JIRA Ticket
 └── SHA

      ↓

JIRA API
      |
      ↓
Check Status
      |
      ↓
Check Approvals
      |
      ↓
Check Deployment Window
      |
      ↓
Validate Commit
      |
      ↓
Production Deployment
```

This aligns well with controlled deployment workflows.

---

# JavaScript Action for Environment Promotion

Example:

```text
DEV
 ↓
QA
 ↓
SIT
 ↓
UAT
 ↓
PROD
```

A JavaScript Action can validate promotion criteria:

```text
Commit Status
Deployment Status
Test Status
Approval
Change Request
```

before the next environment.

---

# Production Promotion Gate

```text
UAT Deployment
       |
       ↓
E2E Tests
       |
       ↓
Success?
       |
      / \
    NO   YES
    |      |
   Stop   PROD Gate
             |
             ↓
         CR Approved?
             |
            / \
          NO   YES
          |      |
         Stop   Deploy
```

JavaScript Actions can implement policy checks in these gates.

---

# JavaScript Action and GitOps

Example:

```text
Build
 |
 ↓
Security
 |
 ↓
ECR
 |
 ↓
JavaScript GitOps Action
 |
 ├── Validate manifest
 ├── Update image SHA
 ├── Commit
 └── Push
 |
 ↓
ArgoCD
 |
 ↓
EKS
```

Keep Git write permissions limited to the job that performs the GitOps update.

---

# JavaScript Action and AWS

For cloud access:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM Role
      |
      ↓
Temporary Credentials
```

Avoid embedding:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

in Action source code.

---

# JavaScript Action and ECR

Conceptual flow:

```text
Docker Image
     |
     ↓
AWS Authentication
     |
     ↓
ECR
```

A JavaScript Action can orchestrate API calls, but use the simplest and most secure supported mechanism for authentication and image operations.

---

# JavaScript Action and EKS

For direct deployment:

```text
JavaScript Action
       |
       ↓
AWS / Kubernetes API
       |
       ↓
EKS
```

For GitOps:

```text
JavaScript Action
       |
       ↓
Git Repository
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

The GitOps pattern can reduce direct cluster access from CI.

---

# Production Security Model

```text
JavaScript Action
      |
      ├── Minimal Permissions
      ├── Minimal Secrets
      ├── Controlled Dependencies
      ├── Validated Inputs
      └── Reviewed Code
```

---

# Common Security Risks

### 1. Dependency Vulnerability

```text
npm dependency
      ↓
Vulnerability
      ↓
Action
```

### 2. Excessive Permissions

```text
Action
  ↓
write-all
```

### 3. Secret Exposure

```text
Secret
  ↓
Log
```

### 4. Unsafe Input Handling

```text
User Input
  ↓
Command / API
```

### 5. Untrusted Third-Party Action

```text
Third Party
  ↓
Runner
```

---

# Secure Coding Practices

- Validate inputs.
- Avoid shell execution when an API/library can perform the operation safely.
- Never hardcode secrets.
- Never print secrets.
- Handle API failures.
- Use timeouts.
- Use bounded retries.
- Keep dependencies updated.
- Review npm dependencies.
- Use lockfiles.
- Bundle dependencies.
- Scan dependencies.
- Use least-privilege permissions.
- Document required permissions.
- Pin third-party Actions according to policy.

---

# JavaScript Action Repository Structure

Recommended:

```text
my-action/
│
├── action.yml
├── package.json
├── package-lock.json
├── tsconfig.json
│
├── src/
│   └── main.ts
│
├── dist/
│   └── index.js
│
├── tests/
│   └── main.test.ts
│
└── README.md
```

---

# Example `action.yml`

```yaml
name: Deployment Validation

description: Validate deployment requirements

inputs:

  environment:
    description: Deployment environment
    required: true

  version:
    description: Version or commit SHA
    required: true

outputs:

  validation-result:
    description: Validation result

runs:
  using: node24
  main: dist/index.js
```

---

# Example TypeScript

```typescript
import * as core from '@actions/core';

async function run(): Promise<void> {
  try {
    const environment = core.getInput('environment', {
      required: true
    });

    const version = core.getInput('version', {
      required: true
    });

    core.info(`Environment: ${environment}`);
    core.info(`Version: ${version}`);

    core.setOutput('validation-result', 'success');

  } catch (error) {
    core.setFailed(
      error instanceof Error
        ? error.message
        : String(error)
    );
  }
}

run();
```

---

# Build

Example:

```bash
npm install
npm run build
```

Output:

```text
dist/index.js
```

---

# Test

Example:

```bash
npm test
```

Production CI should test:

```text
Source
Dependencies
Build
Distribution
Action execution
```

---

# JavaScript Action CI

Example:

```yaml
name: Action CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm test

      - name: Build
        run: npm run build

      - name: Check Distribution
        run: git diff --exit-code -- dist/
```

The exact Node version should match the Action runtime and project support policy.

---

# Security Pipeline for JavaScript Action

A mature Action repository can use:

```text
Pull Request
    |
    ↓
Lint
    |
    ↓
Unit Tests
    |
    ↓
npm Audit / Dependency Scan
    |
    ↓
Build
    |
    ↓
Action Integration Test
    |
    ↓
Review
    |
    ↓
Release
```

---

# Release Pipeline

```text
Merge
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
Release
  |
  ↓
Tag
  |
  ↓
v1
```

---

# Enterprise JavaScript Action

A platform team could maintain:

```text
company/platform-actions
 |
 ├── jira-validation
 ├── security-gate
 ├── release-validation
 ├── deployment-window
 └── gitops-update
```

Application teams consume these Actions.

---

# JavaScript Action for Your Production Workflow

A realistic platform could use:

```text
CI
 |
 ├── Checkout
 ├── Build
 ├── SonarQube
 ├── Trivy
 └── Veracode

CD
 |
 ├── Deploy QA
 ├── E2E Tests
 ├── UAT Approval
 └── Production Gate

Production Gate
 |
 ├── JIRA Ticket
 ├── CR Status
 ├── Approvals
 ├── Deployment Window
 └── Commit SHA
```

JavaScript Actions are particularly suitable for the policy/API validation portions.

---

# JavaScript Action Design Principles

A good JavaScript Action should be:

```text
Focused
Reusable
Testable
Secure
Versioned
Observable
Documented
```

Avoid:

```text
Huge Action
Hidden Side Effects
Broad Permissions
Hardcoded Credentials
Unbounded Retries
Uncontrolled Dependencies
```

---

# When to Choose JavaScript Action

Choose JavaScript when you need:

```text
Complex Logic
GitHub API
External API Integration
Structured Data Processing
Reusable Programmatic Logic
Custom Validation
```

---

# When to Choose Composite Action

Choose Composite when:

```text
Simple Workflow Steps
Shell Commands
Existing Actions
Reusable Step Sequence
```

---

# When to Choose Docker Action

Choose Docker when:

```text
Custom Runtime
Complex Dependencies
Specialized Tooling
Containerized Execution Environment
```

---

# Decision Guide

```text
Need reusable workflow steps?
        |
        ↓
   Composite Action

Need complex programmatic logic/API?
        |
        ↓
   JavaScript Action

Need isolated/custom runtime?
        |
        ↓
   Docker Action
```

---

# Production Decision Example

Requirement:

```text
Check JIRA ticket status
Check deployment window
Check approvals
```

Best candidate:

```text
JavaScript Action
```

because it involves:

```text
API Calls
Structured Data
Business Logic
Validation
```

Requirement:

```text
Run Maven
Run tests
```

Good candidate:

```text
Composite Action
```

Requirement:

```text
Run a custom security scanner with many dependencies
```

Possible candidate:

```text
Docker Action
```

---

# Common Mistakes

### 1. Using JavaScript for simple shell commands

Unnecessary complexity.

### 2. Not bundling dependencies

The Action may fail on the runner.

### 3. Forgetting to update `dist/`

The source changes but the executed Action remains outdated.

### 4. Hardcoding secrets

Never do this.

### 5. Excessive API calls

Can cause rate-limit problems.

### 6. No timeout or retry strategy

External APIs can hang or fail.

### 7. No input validation

Can produce unsafe behavior.

### 8. Broad permissions

Increase blast radius.

### 9. Unmaintained npm dependencies

Creates supply-chain risk.

### 10. No integration tests

The Action may work in unit tests but fail in a real workflow.

---

# Best Practices

- Use TypeScript when it improves maintainability.
- Keep the Action focused.
- Define clear inputs and outputs.
- Validate inputs.
- Use `@actions/core` for Action utilities.
- Use GitHub's supported APIs and SDK helpers where appropriate.
- Handle API failures.
- Use bounded retries and timeouts.
- Keep dependencies updated.
- Use lockfiles.
- Bundle dependencies into the distribution.
- Keep `dist/` synchronized with source changes.
- Test the Action in actual workflows.
- Use least-privilege permissions.
- Avoid unnecessary secrets.
- Never log credentials.
- Version releases.
- Document required permissions and inputs.
- Assign ownership.

---

# Key Takeaways

```text
JavaScript Action
=
GitHub Action implemented using JavaScript / TypeScript
and executed using a Node.js runtime.
```

It is especially useful for:

```text
API Integration
Complex Logic
Policy Validation
Automation
GitHub Integration
Data Processing
```

Typical structure:

```text
action.yml
package.json
src/
dist/
tests/
```

Basic execution:

```yaml
runs:
  using: node24
  main: dist/index.js
```

Production model:

```text
Source
 ↓
Tests
 ↓
Security Scan
 ↓
Build
 ↓
Bundle
 ↓
Integration Test
 ↓
Review
 ↓
Release
```

The key principle:

```text
Use JavaScript Actions when the problem is primarily
programmatic logic or API-driven automation.
```

---

# Interview Questions

## Basic

1. What is a JavaScript Action?
2. How does a JavaScript Action execute?
3. What is `action.yml`?
4. What is the purpose of `main`?
5. What is the `dist` directory?
6. Why are JavaScript Actions commonly built with TypeScript?
7. What is `@actions/core`?
8. What is `@actions/github`?
9. How do you define inputs?
10. How do you define outputs?

## Intermediate

11. How do you read inputs in a JavaScript Action?
12. How do you set outputs?
13. How do you handle errors?
14. How do you access GitHub context?
15. How would you call the GitHub API?
16. How would you authenticate an API request?
17. How do you handle API rate limits?
18. How would you implement retries?
19. Why should you use a lockfile?
20. Why should dependencies be bundled?
21. Why is `dist/index.js` important?
22. How do you test a JavaScript Action?
23. What is the difference between unit testing and integration testing for an Action?
24. How would you version a JavaScript Action?
25. What security risks exist in JavaScript Action dependencies?

## Advanced / Production

26. Design a JavaScript Action that validates a JIRA change request before production deployment.
27. How would you validate deployment windows using a JavaScript Action?
28. How would you verify that a JIRA ticket is approved before production deployment?
29. How would you validate the deployed commit SHA against the approved version?
30. How would you design a JavaScript Action that checks critical Dependabot alerts?
31. How would you integrate a JavaScript Action with GitHub APIs securely?
32. How would you minimize the permissions available to a JavaScript Action?
33. How would you secure npm dependencies used by a JavaScript Action?
34. How would you respond if a dependency used by your JavaScript Action had a critical vulnerability?
35. How would you prevent secrets from appearing in Action logs?
36. How would you design retries for GitHub/JIRA API failures?
37. How would you prevent an API integration from running indefinitely?
38. How would you design a JavaScript Action for a production approval gate?
39. How would you integrate JavaScript Actions into a DevSecOps pipeline using SonarQube, Trivy, and Veracode?
40. How would you integrate a JavaScript Action into a GitOps pipeline using ECR, ArgoCD, and EKS?
41. How would you combine JavaScript Actions with GitHub OIDC and AWS IAM?
42. How would you securely run a JavaScript Action on self-hosted runners?
43. How would you secure a JavaScript Action running through ARC?
44. How would you design a centralized enterprise repository for reusable JavaScript Actions?
45. How would you test a JavaScript Action before releasing it to hundreds of repositories?
46. How would you handle a breaking change in a JavaScript Action?
47. Why is SHA pinning useful for third-party JavaScript Actions?
48. A JavaScript Action works locally but fails in GitHub Actions. How would you troubleshoot it?
49. A JavaScript Action's source code changed but the workflow still executes old behavior. What would you check?
50. When would you choose a JavaScript Action over a Composite Action or Docker Action?