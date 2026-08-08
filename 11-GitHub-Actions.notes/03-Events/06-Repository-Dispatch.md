# Repository Dispatch Event

The `repository_dispatch` event allows an external system to trigger a GitHub Actions workflow through the GitHub REST API.

Unlike `push`, `pull_request`, or `schedule`, the event does not originate from a normal GitHub repository activity.

It is useful when an external application, automation platform, deployment system, or enterprise tool needs to start a GitHub Actions workflow.

Common use cases include:

- External CI/CD systems
- JIRA automation
- Release management systems
- Infrastructure automation
- Cross-repository workflows
- Central deployment platforms
- External monitoring systems
- Enterprise orchestration

---

# Why Use repository_dispatch?

Suppose an external system completes an operation and needs to trigger a GitHub Actions workflow.

Instead of requiring a developer to push code:

```text
External System

↓

GitHub API

↓

repository_dispatch

↓

GitHub Actions

↓

Workflow
```

This allows GitHub Actions to become part of a larger enterprise automation platform.

---

# Basic Syntax

The workflow must explicitly listen for the `repository_dispatch` event.

```yaml
name: External Trigger Workflow

on:
  repository_dispatch:

jobs:
  process:
    runs-on: ubuntu-latest

    steps:
      - name: Process Event
        run: echo "External event received"
```

The workflow does not start simply because the YAML exists.

An external system must send the appropriate API request.

---

# High-Level Architecture

```text
External System

↓

GitHub REST API

↓

repository_dispatch Event

↓

GitHub Actions Workflow

↓

Runner

↓

Job

↓

Steps

↓

Application / Deployment
```

---

# Repository Dispatch API

An external system can send a request to GitHub's repository dispatch API.

Conceptually:

```text
POST

↓

/repos/{owner}/{repo}/dispatches

↓

GitHub

↓

repository_dispatch

↓

Workflow
```

The request contains an `event_type` that identifies what kind of external event occurred.

---

# Event Type

Example workflow:

```yaml
name: External Deployment

on:
  repository_dispatch:
    types:
      - deploy
```

The external system sends:

```text
event_type = deploy
```

GitHub then triggers the workflow because the event type matches.

---

# API Request Example

A conceptual API request looks like:

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/repos/OWNER/REPOSITORY/dispatches \
  -d '{
    "event_type": "deploy"
  }'
```

The authentication method and token permissions must be configured securely.

---

# Passing Data with Client Payload

An external system can send additional information through `client_payload`.

Example:

```json
{
  "event_type": "deploy",
  "client_payload": {
    "environment": "qa",
    "version": "a83f91c",
    "jira_ticket": "CR-12345"
  }
}
```

This allows an external system to provide information to the workflow.

---

# Accessing Client Payload

The workflow can access the payload through the GitHub event context.

Example:

```yaml
- name: Display Deployment Information
  run: |
    echo "Environment: ${{ github.event.client_payload.environment }}"
    echo "Version: ${{ github.event.client_payload.version }}"
    echo "JIRA Ticket: ${{ github.event.client_payload.jira_ticket }}"
```

Execution:

```text
External System

↓

Send Payload

├── Environment
├── Version
└── JIRA Ticket

↓

GitHub Actions

↓

Read Payload

↓

Execute Workflow
```

---

# Complete Example

```yaml
name: External Deployment

on:
  repository_dispatch:
    types:
      - deploy

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Display Inputs
        run: |
          echo "Environment: ${{ github.event.client_payload.environment }}"
          echo "Version: ${{ github.event.client_payload.version }}"

      - name: Deploy
        run: |
          echo "Deploying version ${{ github.event.client_payload.version }}"
          echo "Environment: ${{ github.event.client_payload.environment }}"
```

---

# Enterprise Use Case

Suppose an enterprise has a centralized release management system.

The architecture can be:

```text
Developer

↓

Central Release Platform

↓

Release Approved

↓

GitHub API

↓

repository_dispatch

↓

GitHub Actions

↓

Deploy
```

The external platform controls when the GitHub Actions workflow starts.

---

# Cross-Repository Automation

`repository_dispatch` is especially useful when one repository needs to trigger another repository.

Example:

```text
Application Repository

↓

Build

↓

Artifact Published

↓

Trigger Deployment Repository

↓

repository_dispatch

↓

Deployment Workflow

↓

Kubernetes
```

This separates application source code from deployment configuration.

---

# Enterprise GitOps Example

A production architecture can look like:

```text
Application Repository

↓

GitHub Actions

↓

Build Docker Image

↓

Push Image to ECR

↓

Update GitOps Repository

↓

External Orchestrator

↓

repository_dispatch

↓

Deployment Workflow

↓

ArgoCD / Kubernetes

↓

Application
```

The exact integration depends on the organization's architecture, but `repository_dispatch` provides a mechanism for external systems to initiate workflows.

---

# Microservices Example

Suppose an enterprise has separate repositories:

```text
catalogue-service

cart-service

payment-service

deployment-config
```

After a service release:

```text
Catalogue Repository

↓

Build

↓

Test

↓

Publish Image

↓

Trigger deployment repository

↓

repository_dispatch

↓

Deployment Workflow

↓

Update Catalogue

↓

Kubernetes
```

This allows repositories to remain independently managed.

---

# Production Deployment Example

Your enterprise deployment process can be integrated with `repository_dispatch`.

```text
Release Management System

↓

JIRA Change Request

↓

Approval

↓

Deployment Window

↓

GitHub API

↓

repository_dispatch

↓

Production Workflow

↓

Validate Commit SHA

↓

Validate CI Status

↓

Validate Scan Results

↓

Deploy

↓

Smoke Tests

↓

Developer Sanity Check

↓

Monitoring
```

The external system triggers the deployment workflow, but the workflow should still perform its own validations.

---

# JIRA Integration Pattern

Your notes describe a process where JIRA is used to control production deployments.

A conceptual architecture is:

```text
JIRA Change Request

↓

Approval

↓

Deployment Window

↓

Release Automation

↓

GitHub API

↓

repository_dispatch

↓

GitHub Actions

↓

Production Validation

↓

Deploy
```

The workflow can receive information such as:

```text
JIRA Ticket

Commit SHA

Environment

Version
```

through `client_payload`.

---

# Production Payload Example

An enterprise release platform could send:

```json
{
  "event_type": "production-deploy",
  "client_payload": {
    "environment": "prod",
    "version": "a83f91c",
    "jira_ticket": "CR-12345",
    "application": "catalogue"
  }
}
```

The workflow can then process those values.

---

# Production Validation

Never treat the external payload as automatically trusted deployment authorization.

A production workflow should validate the supplied information.

```text
repository_dispatch

↓

Read Payload

↓

Validate JIRA Ticket

↓

Validate Approval

↓

Validate Deployment Window

↓

Validate Commit SHA

↓

Validate CI Status

↓

Validate Security Results

↓

Validate Testing Results

↓

Production Environment Approval

↓

Deploy
```

This creates defense-in-depth.

---

# External Trigger vs Manual Trigger

`repository_dispatch` and `workflow_dispatch` serve different purposes.

| `repository_dispatch` | `workflow_dispatch` |
|---|---|
| Triggered by an external system | Triggered manually by a user |
| Uses GitHub API | Uses GitHub Actions UI/API |
| Useful for automation | Useful for controlled manual operations |
| Supports client payload | Supports typed workflow inputs |
| Good for cross-system integration | Good for operational deployment |

Example:

```text
workflow_dispatch

↓

DevOps Engineer

↓

Manual Deployment
```

Whereas:

```text
repository_dispatch

↓

External Release Platform

↓

Automated Trigger
```

---

# repository_dispatch vs push

A `push` event occurs because code changes in the repository.

```text
Developer

↓

git push

↓

push Event

↓

Workflow
```

A `repository_dispatch` event is explicitly generated through an API request.

```text
External System

↓

GitHub API

↓

repository_dispatch

↓

Workflow
```

---

# repository_dispatch vs workflow_call

These two events are also different.

`workflow_call` is primarily used to create reusable workflows that are called by another workflow.

```text
Workflow A

↓

workflow_call

↓

Reusable Workflow
```

`repository_dispatch` is intended for external API-triggered events.

```text
External System

↓

GitHub API

↓

repository_dispatch

↓

Workflow
```

---

# Security Considerations

External triggers must be protected carefully.

Important considerations include:

- Secure authentication
- Least-privilege tokens
- Payload validation
- Environment protection
- Production approvals
- Commit validation
- JIRA validation
- Deployment-window validation
- Audit logging

Never place a GitHub token directly inside source code.

Bad:

```yaml
TOKEN: ghp_example_secret
```

Use secure secret management instead.

---

# Token Security

The external system requires appropriate authorization to call the GitHub API.

The exact token type should be selected based on organizational security requirements.

Store credentials in:

- Secure secret managers
- GitHub Actions Secrets
- Organization-level secret management
- Enterprise identity systems

Do not expose credentials in workflow logs.

---

# Payload Validation

Suppose an external system sends:

```json
{
  "environment": "prod",
  "version": "abc123"
}
```

The workflow should validate:

```text
Environment

↓

Is prod allowed?

↓

Version

↓

Does commit exist?

↓

CI Status

↓

Passed?

↓

Continue
```

Do not blindly execute deployment commands based only on external input.

---

# Concurrency

Production workflows triggered externally should also prevent concurrent deployments.

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

Execution:

```text
External Trigger A

↓

Production Deployment

↓

Lock

────────────

External Trigger B

↓

Waiting

↓

Deployment A Complete

↓

Deployment B
```

This prevents simultaneous production deployments.

---

# Enterprise Production Workflow

A complete enterprise architecture can look like this:

```text
JIRA

↓

Change Request

↓

Approvals

↓

Deployment Window

↓

Release Management Platform

↓

GitHub REST API

↓

repository_dispatch

↓

Production Workflow

↓

Validate JIRA

↓

Validate Commit SHA

↓

Validate CI Status

↓

Validate Security Scan

↓

Validate Test Results

↓

Environment Approval

↓

Deploy

↓

Smoke Tests

↓

Developer Sanity

↓

Monitoring

↓

Release Complete
```

This approach combines external enterprise governance with GitHub Actions automation.

---

# Production Troubleshooting

## Scenario 1 - repository_dispatch Workflow Does Not Run

Check:

```text
1. Workflow contains repository_dispatch.
2. event_type matches the configured type.
3. Workflow exists on the correct branch.
4. API request succeeded.
5. Authentication is valid.
6. Token has required permissions.
```

---

## Scenario 2 - API Returns an Error

Check:

```text
External System

↓

API URL

↓

Repository

↓

Authentication

↓

Token Permissions

↓

Request Payload

↓

GitHub API Response
```

Do not assume that a network connection means the dispatch was accepted.

---

## Scenario 3 - Workflow Starts but Payload Is Empty

Check:

```text
client_payload

↓

JSON Structure

↓

Field Names

↓

github.event.client_payload

↓

Workflow Logs
```

For example:

```yaml
${{ github.event.client_payload.version }}
```

must match the actual payload field.

---

## Scenario 4 - Wrong Environment Is Passed

Validate the payload before deployment.

```text
Payload

↓

environment = prod

↓

Allowed Environment?

↓

YES → Continue

NO → Stop
```

Use protected GitHub Environments for production.

---

## Scenario 5 - External System Triggers Multiple Deployments

Check:

```text
External System

↓

Duplicate API Requests

↓

repository_dispatch

↓

Multiple Workflow Runs
```

Use:

- Idempotency controls
- Concurrency
- Deployment locks
- External request tracking

to prevent duplicate production deployments.

---

# Best Practices

- Use `repository_dispatch` for external automation.
- Use meaningful `event_type` values.
- Validate every important payload field.
- Use secure authentication.
- Apply least-privilege permissions.
- Use protected environments for production.
- Validate JIRA/change requests before deployment.
- Validate commit SHA and CI results.
- Use concurrency for production deployments.
- Maintain auditability of external triggers.
- Make deployment operations idempotent where possible.

---

# Common Mistakes

- Treating external payload data as trusted.
- Hardcoding GitHub tokens.
- Using excessive token permissions.
- Triggering production without approval validation.
- Not validating the commit SHA.
- Not validating the deployment environment.
- Ignoring duplicate external requests.
- Not implementing concurrency for production deployments.
- Using `repository_dispatch` when a simple `push` or `workflow_dispatch` is more appropriate.

---

# Summary

The `repository_dispatch` event allows external systems to trigger GitHub Actions workflows through the GitHub REST API.

It is especially useful for enterprise automation involving:

- Release management systems
- JIRA
- Cross-repository workflows
- Central deployment platforms
- External automation systems

A production-grade implementation should follow:

```text
External System

↓

GitHub API

↓

repository_dispatch

↓

Validate Payload

↓

Validate JIRA

↓

Validate Commit

↓

Validate CI / Security / Tests

↓

Approval

↓

Deploy

↓

Smoke Tests

↓

Monitoring
```

The key principle is that an external trigger should **start the workflow**, not automatically bypass the workflow's security and deployment controls.

---

# Interview Questions

## Basic

1. What is `repository_dispatch`?
2. How does `repository_dispatch` differ from `push`?
3. How is `repository_dispatch` triggered?
4. What is `event_type`?
5. What is `client_payload`?

## Intermediate

6. How can an external system trigger GitHub Actions?
7. How can data be passed from an external system to a workflow?
8. What is the difference between `repository_dispatch` and `workflow_dispatch`?
9. What is the difference between `repository_dispatch` and `workflow_call`?
10. How would you secure an external GitHub API trigger?

## Advanced

11. Design a cross-repository deployment architecture using `repository_dispatch`.
12. Design an enterprise integration where JIRA approves a production change and an external release platform triggers GitHub Actions.
13. An external system accidentally triggers the same production deployment three times. Explain how you would prevent duplicate deployments using validation, idempotency, and concurrency.
14. Design a production workflow that receives a JIRA ticket, environment, and commit SHA through `client_payload` and validates all three before deployment.
15. A `repository_dispatch` API call succeeds, but the workflow does not start. Explain your troubleshooting process from API authentication through event configuration, event type, workflow branch, and GitHub Actions execution.