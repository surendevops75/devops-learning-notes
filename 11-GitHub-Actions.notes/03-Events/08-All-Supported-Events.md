# All Supported Events

GitHub Actions workflows are event-driven.

An event represents an activity that occurs in GitHub or an external system that causes a workflow to execute.

Common events include:

- `push`
- `pull_request`
- `workflow_dispatch`
- `schedule`
- `release`
- `repository_dispatch`
- `workflow_call`
- `workflow_run`
- `issues`
- `issue_comment`
- `pull_request_review`
- `pull_request_review_comment`
- `deployment`
- `deployment_status`
- `create`
- `delete`
- `fork`
- `status`
- `check_run`
- `check_suite`
- `discussion`
- `watch`
- `workflow_job`

For DevOps and enterprise CI/CD, the most important events are:

```text
push
pull_request
workflow_dispatch
schedule
release
repository_dispatch
workflow_call
workflow_run
```

---

# Event Categories

GitHub Actions events can be grouped conceptually into several categories.

```text
Repository Activity
        |
        ├── push
        ├── pull_request
        ├── release
        ├── issues
        ├── issue_comment
        └── pull_request_review

Manual / Scheduled
        |
        ├── workflow_dispatch
        └── schedule

External Integration
        |
        └── repository_dispatch

Workflow Orchestration
        |
        ├── workflow_call
        └── workflow_run

Deployment / Status
        |
        ├── deployment
        ├── deployment_status
        ├── status
        ├── check_run
        └── check_suite
```

---

# 1. push

The `push` event triggers a workflow when commits or other changes are pushed to a repository.

Basic syntax:

```yaml
on:
  push:
```

Branch filtering:

```yaml
on:
  push:
    branches:
      - main
```

Common use cases:

- Continuous Integration
- Build
- Unit tests
- Security scanning
- Docker image creation
- Artifact publishing

Enterprise flow:

```text
Developer
    |
    ↓
Push
    |
    ↓
CI
    |
    ├── Build
    ├── Test
    ├── SonarQube
    └── Trivy
    |
    ↓
Artifact
```

---

# 2. pull_request

The `pull_request` event triggers workflows when Pull Request activity occurs.

Basic syntax:

```yaml
on:
  pull_request:
```

Target branch filtering:

```yaml
on:
  pull_request:
    branches:
      - main
```

Common use cases:

- Build validation
- Unit testing
- SonarQube
- Trivy
- Terraform validation
- Policy checks
- Merge quality gates

Enterprise flow:

```text
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
Build
    |
    ↓
Tests
    |
    ↓
Security
    |
    ↓
Code Review
    |
    ↓
Merge
```

---

# 3. workflow_dispatch

The `workflow_dispatch` event allows a user to manually start a workflow.

Basic syntax:

```yaml
on:
  workflow_dispatch:
```

It can accept inputs.

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment"
        required: true
        type: choice
        options:
          - qa
          - sit
          - uat
          - prod
```

Common use cases:

- Production deployment
- Rollback
- Manual release
- Environment promotion
- Operational tasks
- Emergency deployment

Enterprise flow:

```text
DevOps Engineer
    |
    ↓
Run Workflow
    |
    ↓
Select Environment
    |
    ↓
Enter Version
    |
    ↓
Validate
    |
    ↓
Approval
    |
    ↓
Deploy
```

---

# 4. schedule

The `schedule` event runs a workflow according to a cron expression.

Example:

```yaml
on:
  schedule:
    - cron: '0 2 * * *'
```

Common use cases:

- Nightly security scans
- Dependency checks
- Reports
- Infrastructure validation
- Cleanup
- Health checks
- Scheduled operational tasks

Important:

GitHub Actions scheduled workflows use UTC for cron scheduling.

Enterprise flow:

```text
Scheduled Time
    |
    ↓
Workflow
    |
    ↓
Security Scan
    |
    ↓
Report
    |
    ↓
Notification
```

---

# 5. release

The `release` event triggers a workflow based on GitHub Release activity.

Example:

```yaml
on:
  release:
    types:
      - published
```

Common release activity types include:

- `published`
- `created`
- `edited`
- `deleted`
- `prereleased`
- `released`

Common use cases:

- Build release artifacts
- Publish Docker images
- Publish packages
- Generate release documentation
- Release automation

Enterprise flow:

```text
Commit
    |
    ↓
Version Tag
    |
    ↓
GitHub Release
    |
    ↓
Release Workflow
    |
    ↓
Build Artifact
    |
    ↓
Publish
```

---

# 6. repository_dispatch

The `repository_dispatch` event allows an external system to trigger a GitHub Actions workflow through the GitHub API.

Example:

```yaml
on:
  repository_dispatch:
    types:
      - deploy
```

External system:

```text
External Platform
    |
    ↓
GitHub REST API
    |
    ↓
repository_dispatch
    |
    ↓
GitHub Actions
```

Common use cases:

- External release platforms
- JIRA integrations
- Cross-repository automation
- Central deployment platforms
- Enterprise orchestration

It can also carry additional data through `client_payload`.

Example:

```json
{
  "event_type": "deploy",
  "client_payload": {
    "environment": "prod",
    "version": "a83f91c",
    "jira_ticket": "CR-12345"
  }
}
```

The workflow can access values through:

```yaml
${{ github.event.client_payload.environment }}

${{ github.event.client_payload.version }}

${{ github.event.client_payload.jira_ticket }}
```

---

# 7. workflow_call

The `workflow_call` event is used to create reusable workflows.

Basic syntax:

```yaml
on:
  workflow_call:
```

Another workflow can call it:

```yaml
jobs:
  ci:
    uses: organization/platform/.github/workflows/ci.yml@v1
```

Common use cases:

- Standardized CI
- Standardized CD
- Security pipelines
- Docker builds
- Terraform workflows
- Helm deployments
- Enterprise platform workflows

Enterprise architecture:

```text
Application A
       |
       ↓
Reusable CI

Application B
       |
       ↓
Reusable CI

Application C
       |
       ↓
Reusable CI
```

This avoids duplicating the same pipeline logic across many repositories.

---

# 8. workflow_run

The `workflow_run` event allows one workflow to react to the completion of another workflow.

Example:

```yaml
on:
  workflow_run:
    workflows:
      - CI
    types:
      - completed
```

Conceptual flow:

```text
CI Workflow
    |
    ↓
Completed
    |
    ↓
workflow_run
    |
    ↓
Deployment / Notification Workflow
```

A workflow should verify the result of the previous workflow before performing sensitive operations.

Example concept:

```text
CI Completed
    |
    ↓
Success?
   / \
 YES  NO
  |    |
  ↓    ↓
Deploy Stop
```

---

# 9. issues

The `issues` event triggers automation based on issue activity.

Example:

```yaml
on:
  issues:
    types:
      - opened
```

Common use cases:

- Automatic labeling
- Notifications
- Issue routing
- Operational automation

Example:

```text
Issue Created
    |
    ↓
GitHub Actions
    |
    ↓
Apply Label
    |
    ↓
Notify Team
```

---

# 10. issue_comment

The `issue_comment` event triggers when comments are created or modified on issues or Pull Requests.

Example:

```yaml
on:
  issue_comment:
    types:
      - created
```

A possible automation pattern is:

```text
Pull Request
    |
    ↓
Comment
    |
    ↓
"/deploy qa"
    |
    ↓
Workflow
    |
    ↓
Validate User
    |
    ↓
Deploy QA
```

For privileged operations, carefully validate:

- User
- Permissions
- Comment content
- Target environment
- Deployment authorization

Never allow arbitrary users to execute privileged deployment commands through comments.

---

# 11. pull_request_review

The `pull_request_review` event responds to Pull Request review activity.

Example:

```yaml
on:
  pull_request_review:
    types:
      - submitted
```

Enterprise flow:

```text
Pull Request
    |
    ↓
Reviewer
    |
    ↓
Review Submitted
    |
    ↓
Workflow
    |
    ↓
Process Review Result
```

Possible uses:

- Review automation
- Notifications
- Governance
- Approval tracking

---

# 12. pull_request_review_comment

This event responds to review comments associated with Pull Requests.

Conceptual flow:

```text
Pull Request
    |
    ↓
Code Review
    |
    ↓
Review Comment
    |
    ↓
Workflow
```

It can be useful for review-related automation and notifications.

---

# 13. pull_request_target

The `pull_request_target` event runs in the context of the target repository.

It requires special security consideration.

This event should not be treated as a safer replacement for `pull_request`.

Be extremely careful when processing untrusted Pull Request code.

Never blindly execute untrusted code with:

- Repository write permissions
- Production credentials
- Deployment credentials
- Sensitive secrets

Conceptually:

```text
External Pull Request
    |
    ↓
Target Repository Context
    |
    ↓
Workflow
```

---

# 14. deployment

The `deployment` event relates to deployment activity.

Conceptual flow:

```text
Deployment Created
    |
    ↓
deployment Event
    |
    ↓
Workflow
    |
    ↓
Automation
```

Possible uses:

- Deployment automation
- Deployment tracking
- Notifications
- Integration with deployment platforms

---

# 15. deployment_status

The `deployment_status` event responds to changes in deployment status.

Conceptual flow:

```text
Deployment
    |
    ↓
Status Change
    |
    ├── Success
    ├── Failure
    └── In Progress
    |
    ↓
Workflow
```

Possible uses:

- Notifications
- Release tracking
- Deployment reporting
- Operational automation

---

# 16. create

The `create` event occurs when a Git reference such as a branch or tag is created.

Conceptual flow:

```text
Create Branch / Tag
    |
    ↓
create Event
    |
    ↓
Workflow
```

Possible use case:

```text
Create Release Tag
    |
    ↓
Validation
    |
    ↓
Release Automation
```

---

# 17. delete

The `delete` event occurs when a Git reference is deleted.

Conceptual flow:

```text
Branch / Tag Deleted
    |
    ↓
delete Event
    |
    ↓
Workflow
    |
    ↓
Cleanup / Audit
```

This can be useful for repository governance and cleanup automation.

---

# 18. fork

The `fork` event can trigger automation when a repository is forked.

Conceptual flow:

```text
Repository
    |
    ↓
Fork
    |
    ↓
fork Event
    |
    ↓
Automation
```

Possible use cases:

- Audit
- Notifications
- Repository governance

---

# 19. status

The `status` event relates to commit status activity.

Conceptual flow:

```text
Commit
    |
    ↓
Status
    |
    ↓
Workflow
    |
    ↓
Process Status
```

This can be useful when integrating external CI systems.

---

# 20. check_run

The `check_run` event relates to individual check runs.

Conceptual flow:

```text
Check Run
    |
    ↓
Completed
    |
    ↓
Workflow
    |
    ↓
Process Result
```

This is useful for advanced CI/CD and quality integrations.

---

# 21. check_suite

The `check_suite` event relates to check suites.

Conceptual flow:

```text
Check Suite
    |
    ↓
Completed
    |
    ↓
Workflow
    |
    ↓
Automation
```

This can be useful when integrating advanced validation systems.

---

# 22. discussion

The `discussion` event can respond to GitHub Discussions activity.

Possible use cases:

- Community automation
- Notifications
- Administrative automation
- Project workflows

Conceptual flow:

```text
Discussion Activity
    |
    ↓
Workflow
    |
    ↓
Automation
```

---

# 23. watch

The `watch` event occurs when a user stars a repository.

This is generally more useful for repository or community automation than CI/CD.

Conceptual flow:

```text
Repository
    |
    ↓
Star
    |
    ↓
watch Event
    |
    ↓
Automation
```

---

# 24. page_build

The `page_build` event relates to GitHub Pages builds.

Conceptual flow:

```text
Pages Build
    |
    ↓
page_build
    |
    ↓
Workflow
    |
    ↓
Post-Build Automation
```

---

# 25. package

The `package` event relates to package activity.

Possible use cases:

- Package automation
- Notifications
- Release processing

Conceptual flow:

```text
Package Activity
    |
    ↓
Workflow
    |
    ↓
Automation
```

---

# 26. registry_package

The `registry_package` event relates to package registry activity.

Conceptual flow:

```text
Package Registry
    |
    ↓
Package Event
    |
    ↓
Workflow
```

Possible uses include package publication and release automation.

---

# 27. repository

The `repository` event can be used for repository-level automation where supported.

Possible use cases:

- Repository governance
- Audit
- Repository configuration
- Organization automation

---

# 28. repository_import

The `repository_import` event relates to repository import activity.

Conceptual flow:

```text
Repository Import
    |
    ↓
repository_import
    |
    ↓
Workflow
    |
    ↓
Repository Setup
```

---

# 29. repository_vulnerability_alert

The `repository_vulnerability_alert` event can be used for security-related repository automation.

Conceptual flow:

```text
Dependency Vulnerability
    |
    ↓
Alert
    |
    ↓
Workflow
    |
    ↓
Security Response
```

Possible uses:

- Security notifications
- Vulnerability processing
- Security ticket creation
- Automated remediation workflows

---

# 30. security_advisory

The `security_advisory` event relates to security advisory activity.

Conceptual flow:

```text
Security Advisory
    |
    ↓
Workflow
    |
    ↓
Security Response
```

This is mainly relevant to security and repository governance.

---

# 31. milestone

The `milestone` event relates to milestone activity.

Conceptual flow:

```text
Milestone Activity
    |
    ↓
Workflow
    |
    ↓
Reporting / Notification
```

This is more relevant to project-management automation than normal CI/CD.

---

# 32. label

The `label` event relates to label activity.

Conceptual flow:

```text
Label Added / Changed
    |
    ↓
Workflow
    |
    ↓
Automation
```

Possible uses:

- Issue categorization
- Routing
- Notifications
- Project management

---

# 33. project

The `project` event relates to GitHub project activity where supported.

Conceptual flow:

```text
Project Activity
    |
    ↓
Workflow
    |
    ↓
Automation
```

---

# 34. project_card

The `project_card` event relates to project card activity where supported.

Possible use cases:

- Project management
- Automation
- Notifications

---

# 35. project_column

The `project_column` event relates to project column activity where supported.

Possible use cases:

- Project automation
- Workflow tracking
- Notifications

---

# 36. project_v2

The `project_v2` event relates to modern GitHub Projects activity where supported.

Conceptual flow:

```text
Project Activity
    |
    ↓
GitHub Event
    |
    ↓
Workflow
```

---

# 37. project_v2_item

The `project_v2_item` event relates to GitHub Project item activity where supported.

Conceptual flow:

```text
Project Item Updated
    |
    ↓
Workflow
    |
    ↓
Automation
```

---

# 38. public

The `public` event occurs when a repository changes from private to public.

Conceptual flow:

```text
Private Repository
    |
    ↓
Made Public
    |
    ↓
public Event
    |
    ↓
Security / Governance Automation
```

Enterprise organizations may use this for audit and compliance workflows.

---

# 39. repository_ruleset

Repository ruleset-related activity can be used for governance automation where supported.

Conceptual flow:

```text
Repository Ruleset Activity
    |
    ↓
Workflow
    |
    ↓
Governance Automation
```

This is relevant to organizations that centrally enforce repository policies.

---

# 40. merge_group

The `merge_group` event is useful with merge queues.

Conceptual flow:

```text
Pull Request
    |
    ↓
Merge Queue
    |
    ↓
Merge Group
    |
    ↓
Validation Workflow
    |
    ↓
Merge
```

This is useful when organizations use protected branches and merge queues to validate changes before they enter a protected branch.

---

# 41. workflow_job

The `workflow_job` event relates to workflow job activity.

Conceptual flow:

```text
Workflow Job
    |
    ↓
Job Event
    |
    ↓
Automation
```

Possible use cases:

- Runner monitoring
- Job tracking
- Operational reporting
- Advanced enterprise automation

---

# 42. workflow_dispatch

`workflow_dispatch` is already covered above and is one of the most important manual operational triggers.

Typical production usage:

```text
DevOps Engineer
    |
    ↓
workflow_dispatch
    |
    ↓
JIRA Ticket
    |
    ↓
Commit SHA
    |
    ↓
Deployment Window
    |
    ↓
Approval
    |
    ↓
Production
```

---

# 43. workflow_run

`workflow_run` is particularly useful for workflow orchestration.

Example:

```yaml
on:
  workflow_run:
    workflows:
      - CI
    types:
      - completed
```

Enterprise pattern:

```text
CI
    |
    ↓
Completed
    |
    ↓
workflow_run
    |
    ↓
Release / Deployment
```

Always validate the previous workflow's conclusion before continuing.

---

# Important Events for Your DevOps Career

You do not need to memorize every GitHub event equally.

Focus heavily on these:

```text
1. push
2. pull_request
3. workflow_dispatch
4. schedule
5. release
6. repository_dispatch
7. workflow_call
8. workflow_run
9. merge_group
10. pull_request_review
```

These are especially relevant to enterprise CI/CD, DevSecOps, GitOps, platform engineering, and production operations.

---

# Event Selection Strategy

Choose the event based on what should initiate the automation.

## Code Change

```text
Code Push
    |
    ↓
push
```

Use for CI and build automation.

---

## Pull Request

```text
Pull Request
    |
    ↓
pull_request
```

Use for pre-merge validation.

---

## Manual Operation

```text
Human
    |
    ↓
workflow_dispatch
```

Use for controlled operational tasks.

---

## Recurring Automation

```text
Time
    |
    ↓
schedule
```

Use for recurring tasks.

---

## Software Release

```text
Release
    |
    ↓
release
```

Use for release automation.

---

## External System

```text
External Platform
    |
    ↓
repository_dispatch
```

Use for external integrations.

---

## Reusable Workflow

```text
Workflow A
    |
    ↓
workflow_call
    |
    ↓
Reusable Workflow
```

Use for centralized CI/CD logic.

---

## Previous Workflow Completion

```text
Workflow A
    |
    ↓
Completed
    |
    ↓
workflow_run
    |
    ↓
Workflow B
```

Use for workflow orchestration.

---

# Enterprise Event Architecture

A mature enterprise GitHub Actions platform may use:

```text
                    GitHub Repository
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
        Push           Pull Request       Release
          |                |                |
          ↓                ↓                ↓
         CI             PR Checks        Release
          |                |                |
          └────────────────┼────────────────┘
                           ↓
                        Artifact
                           |
                           ↓
                     Environment
                           |
                ┌──────────┴──────────┐
                ↓                     ↓
          Manual Trigger       External Trigger
                |                     |
                ↓                     ↓
       workflow_dispatch    repository_dispatch
                |                     |
                └──────────┬──────────┘
                           ↓
                     Production
```

Reusable enterprise logic:

```text
Application Workflow
        |
        ↓
workflow_call
        |
        ↓
Platform Reusable Workflow
```

---

# Enterprise CI/CD Event Mapping

| Requirement | Recommended Event |
|---|---|
| Developer pushes code | `push` |
| Pull Request validation | `pull_request` |
| Manual deployment | `workflow_dispatch` |
| Nightly security scan | `schedule` |
| Version release | `release` |
| External deployment trigger | `repository_dispatch` |
| Reusable CI/CD logic | `workflow_call` |
| Trigger after another workflow | `workflow_run` |
| Merge queue validation | `merge_group` |
| Review automation | `pull_request_review` |
| Issue automation | `issues` |
| Comment automation | `issue_comment` |
| Deployment tracking | `deployment_status` |
| Commit status automation | `status` |
| Advanced check automation | `check_run` / `check_suite` |

---

# Combining Multiple Events

A single workflow can listen to multiple events.

Example:

```yaml
name: Application Validation

on:

  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:

  schedule:
    - cron: '0 2 * * *'
```

The same workflow can therefore start because of:

```text
Push
   \
Pull Request
    \
Manual Trigger
     \
Scheduled Trigger
      \
       ↓
    Workflow
```

However, do not combine unrelated responsibilities simply because GitHub allows multiple triggers.

For enterprise environments, separate workflows are often easier to maintain.

---

# Recommended Enterprise Separation

Instead of one huge workflow:

```text
Everything
    |
    ├── CI
    ├── Security
    ├── Release
    ├── Deployment
    ├── Rollback
    └── Reporting
```

Prefer:

```text
CI Workflow
    |
    └── push
        pull_request


Security Workflow
    |
    └── schedule
        workflow_dispatch


Release Workflow
    |
    └── release


Production Deployment
    |
    └── workflow_dispatch
        repository_dispatch


Reusable Platform Workflow
    |
    └── workflow_call
```

This improves:

- Maintainability
- Security
- Troubleshooting
- Ownership
- Auditability

---

# Production Deployment Event Strategy

For a production environment, the trigger should not be the only security control.

Example:

```text
Event
    |
    ↓
Authentication
    |
    ↓
Authorization
    |
    ↓
Input Validation
    |
    ↓
JIRA Validation
    |
    ↓
Commit Validation
    |
    ↓
CI Status
    |
    ↓
Security Results
    |
    ↓
Testing Results
    |
    ↓
Deployment Window
    |
    ↓
Approval
    |
    ↓
Production
```

This is especially important for your enterprise deployment process.

---

# JIRA + GitHub Actions Event Architecture

A production deployment can be initiated through an external enterprise platform.

```text
JIRA Change Request
        |
        ↓
Approvals
        |
        ↓
Deployment Window
        |
        ↓
Release Platform
        |
        ↓
GitHub API
        |
        ↓
repository_dispatch
        |
        ↓
GitHub Actions
        |
        ↓
Validate Commit
        |
        ↓
Validate CI
        |
        ↓
Validate Security
        |
        ↓
Production Approval
        |
        ↓
Deploy
```

The external trigger should initiate the workflow, but the workflow should independently validate production requirements.

---

# Event Security

An event only determines how a workflow starts.

It should not automatically determine whether the workflow is authorized to perform a sensitive action.

Use:

- Least-privilege permissions
- Protected environments
- Required reviewers
- Secure secrets
- Input validation
- Commit validation
- Deployment-window validation
- Concurrency controls
- Audit logging

Production principle:

```text
Trigger

≠

Authorization
```

A workflow being triggered successfully does not mean it should automatically deploy.

---

# Production Troubleshooting Framework

When a workflow does not execute, follow this sequence.

```text
1. Is the workflow file present?
        |
        ↓
2. Is the YAML valid?
        |
        ↓
3. Is the correct event configured?
        |
        ↓
4. Does the branch filter match?
        |
        ↓
5. Does the path filter match?
        |
        ↓
6. Does the activity type match?
        |
        ↓
7. Is the workflow enabled?
        |
        ↓
8. Was a workflow run created?
        |
        ↓
9. Was a runner allocated?
        |
        ↓
10. Which job or step failed?
```

---

# Scenario 1 - Push Workflow Does Not Run

Check:

```text
push
    |
    ↓
Branch Filter
    |
    ↓
Path Filter
    |
    ↓
Workflow Configuration
```

Confirm that the commit actually matches the configured trigger.

---

# Scenario 2 - Pull Request Workflow Does Not Run

Check:

```text
pull_request
    |
    ↓
Target Branch
    |
    ↓
Activity Type
    |
    ↓
Path Filter
```

Also verify that the workflow exists and is enabled.

---

# Scenario 3 - Scheduled Workflow Runs at the Wrong Time

Check:

```text
Required Local Time
    |
    ↓
UTC Conversion
    |
    ↓
Cron Expression
    |
    ↓
Actual Execution
```

Do not assume cron uses your local time zone.

---

# Scenario 4 - Manual Deployment Uses Wrong Version

Check:

```text
workflow_dispatch
    |
    ↓
Input Version
    |
    ↓
Commit SHA
    |
    ↓
Artifact
    |
    ↓
Deployment
```

Validate that the supplied version maps to the expected source and artifact.

---

# Scenario 5 - External System Triggers Production Multiple Times

Check:

```text
repository_dispatch
    |
    ↓
External Requests
    |
    ↓
Duplicate Events
    |
    ↓
Concurrency
    |
    ↓
Idempotency
```

Use deployment locks and validation to prevent duplicate production deployments.

---

# Scenario 6 - Reusable Workflow Change Breaks Multiple Repositories

Check:

```text
workflow_call
    |
    ↓
Reusable Workflow
    |
    ↓
Version Reference
    |
    ↓
Breaking Change
```

Use versioned reusable workflows for production.

Example:

```text
@v1
```

instead of always consuming:

```text
@main
```

for critical production automation.

---

# Scenario 7 - Deployment Starts Before CI Completes

If one workflow depends on another workflow, use an appropriate orchestration pattern.

For example:

```text
CI
    |
    ↓
workflow_run
    |
    ↓
Check Conclusion
    |
    ↓
Deploy
```

Never assume workflow completion means successful workflow execution.

---

# Best Practices

- Select the event based on the actual automation requirement.
- Keep workflows focused.
- Use branch and path filters where appropriate.
- Use `workflow_dispatch` for controlled manual operations.
- Use `schedule` for recurring automation.
- Use `repository_dispatch` for external integrations.
- Use `workflow_call` for reusable workflows.
- Use `workflow_run` for workflow orchestration.
- Use `merge_group` for merge queue validation where applicable.
- Protect production environments.
- Validate deployment inputs.
- Use least-privilege permissions.
- Use concurrency for production deployments.
- Maintain commit and artifact traceability.
- Version reusable workflows.
- Keep production authorization separate from event triggering.

---

# Common Mistakes

- Choosing the wrong event.
- Using `push` for every type of automation.
- Running production deployment from unrestricted events.
- Ignoring branch and path filters.
- Forgetting that scheduled workflows use UTC.
- Trusting external event payloads without validation.
- Using unversioned reusable workflows for critical production automation.
- Combining too many unrelated responsibilities into one workflow.
- Not implementing concurrency for production deployments.
- Assuming a completed workflow was successful.
- Giving workflows excessive permissions.
- Allowing external comments or payloads to directly execute privileged operations.

---

# Summary

GitHub Actions is fundamentally event-driven.

The most important enterprise events are:

```text
push
pull_request
workflow_dispatch
schedule
release
repository_dispatch
workflow_call
workflow_run
merge_group
```

Other events support:

```text
Issues
Reviews
Comments
Deployments
Statuses
Checks
Repositories
Projects
Security
Packages
Discussions
```

The event should be selected according to the automation requirement.

The enterprise approach is:

```text
Code Change
    |
    ↓
push

Pull Request
    |
    ↓
pull_request

Manual Operation
    |
    ↓
workflow_dispatch

Scheduled Automation
    |
    ↓
schedule

Software Release
    |
    ↓
release

External System
    |
    ↓
repository_dispatch

Reusable Workflow
    |
    ↓
workflow_call

Workflow Completion
    |
    ↓
workflow_run
```

For production deployment:

```text
Event
    |
    ↓
Validation
    |
    ↓
Authorization
    |
    ↓
JIRA / Change Request
    |
    ↓
Commit Validation
    |
    ↓
CI / Security / Testing
    |
    ↓
Approval
    |
    ↓
Deployment Window
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
An event starts automation.

An event does not replace authorization.
```

---

# Interview Questions

## Basic

1. What is a GitHub Actions event?
2. What is the `push` event?
3. What is the `pull_request` event?
4. What is `workflow_dispatch`?
5. What is `schedule`?
6. What is `release`?
7. What is `repository_dispatch`?
8. What is `workflow_call`?
9. What is `workflow_run`?

## Intermediate

10. What is the difference between `push` and `pull_request`?
11. When would you use `workflow_dispatch` instead of `push`?
12. When would you use `repository_dispatch`?
13. What is the purpose of `workflow_call`?
14. What is the purpose of `workflow_run`?
15. How do branch and path filters affect event execution?
16. How can multiple events trigger the same workflow?
17. Why should production deployment usually use a controlled trigger?

## Advanced

18. Design an enterprise event architecture for a microservices platform covering Pull Requests, CI, security scanning, releases, UAT, and production deployment.
19. A company currently uses `push` for everything. Explain how you would redesign the event architecture for better security and maintainability.
20. Design a production deployment workflow where JIRA and an external release platform trigger GitHub Actions using `repository_dispatch`.
21. Design a reusable CI/CD platform using `workflow_call` that can be consumed by hundreds of application repositories.
22. A workflow executes successfully for `push` but not for `pull_request`. Explain how you would troubleshoot the difference.
23. A scheduled workflow executes at an unexpected time. Explain how you would identify and correct the problem.
24. A production deployment is triggered by an external event twice. Design controls using event validation, concurrency, idempotency, and environment protection to prevent duplicate deployments.
25. Explain how you would choose between `push`, `pull_request`, `workflow_dispatch`, `schedule`, `release`, `repository_dispatch`, `workflow_call`, and `workflow_run` when designing an enterprise GitHub Actions platform.
26. Explain why an event should be treated as a trigger mechanism rather than as production authorization.