# Jenkins Build Triggers

## Introduction

A Jenkins pipeline does not execute automatically unless it is triggered.

A **Build Trigger** defines the event or condition that starts a Jenkins job. In production environments, pipelines are typically triggered by source code changes, pull requests, scheduled jobs, upstream pipelines, or external systems.

Choosing the correct trigger improves automation, reduces manual effort, and ensures CI/CD pipelines execute at the right time.

---

# Why Do We Need Build Triggers?

Without build triggers:

```text
Developer Push

↓

Wait

↓

Login to Jenkins

↓

Click Build Now

↓

Pipeline Starts
```

Problems:

- Manual process
- Delayed feedback
- Human error
- Poor automation

---

With Build Triggers

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Pipeline Starts Automatically
```

Benefits:

- Fully automated
- Faster feedback
- No manual intervention
- Better CI/CD

---

# What Can Trigger a Jenkins Pipeline?

Common build triggers include:

```text
Developer Push

↓

GitHub Webhook

-----------------------

Pull Request

↓

Webhook

-----------------------

Cron Schedule

↓

Nightly Build

-----------------------

Upstream Job

↓

Downstream Job

-----------------------

Manual Build

↓

Build Now

-----------------------

Remote API Trigger

↓

External System
```

---

# Types of Build Triggers

Jenkins supports multiple trigger mechanisms.

| Trigger | Use Case |
|----------|----------|
| Manual Trigger | Testing or debugging |
| GitHub Webhook | Continuous Integration |
| Poll SCM | Periodically check Git |
| Cron Schedule | Nightly builds |
| Upstream Job | CI/CD chaining |
| Remote Trigger | External integrations |

---

# Manual Trigger

The simplest trigger is manually clicking **Build Now**.

```text
User

↓

Build Now

↓

Pipeline Executes
```

Suitable for:

- Testing
- Debugging
- Initial setup

Not recommended for production CI/CD.

---

# GitHub Webhook Trigger

The most common enterprise trigger.

Workflow:

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline Starts
```

Advantages:

- Immediate execution
- No polling
- Lower server load
- Faster developer feedback

---

# Poll SCM

Instead of GitHub notifying Jenkins,

Jenkins periodically checks GitHub.

```text
Jenkins

↓

Every 5 Minutes

↓

Check Repository

↓

Changes Found?

↓

Run Pipeline
```

Disadvantages:

- Delayed execution
- Unnecessary repository scans
- Higher Jenkins load

---

# Cron Trigger

Cron schedules allow pipelines to execute automatically at specific times.

Examples:

```text
Every Midnight

↓

Backup

---------------------

Every Sunday

↓

Security Scan

---------------------

Every Morning

↓

Dependency Update
```

Common cron syntax:

```text
H 2 * * *

↓

Run Daily Around 2 AM
```

---

# Upstream Trigger

A pipeline can start automatically after another pipeline finishes.

```text
Build Pipeline

↓

Docker Image

↓

Deploy Pipeline
```

Useful for separating build and deployment jobs.

---

# Remote Trigger

External systems can invoke Jenkins using its REST API.

Example:

```text
Monitoring Tool

↓

REST API

↓

Jenkins

↓

Pipeline
```

Common integrations:

- ServiceNow
- Jira
- GitHub
- Custom Applications

---

# Production Architecture

```text
                Developer
                     │
                     ▼
                GitHub Push
                     │
                     ▼
              GitHub Webhook
                     │
                     ▼
      Jenkins Multibranch Pipeline
                     │
                     ▼
          Load Shared Library
                     │
                     ▼
           javaEKSPipeline()
                     │
                     ▼
                Build & Test
                     │
                     ▼
                SonarQube
                     │
                     ▼
        OWASP Dependency Check
                     │
                     ▼
               Docker Build
                     │
                     ▼
                Trivy Scan
                     │
                     ▼
             Push Image to ECR
                     │
                     ▼
               EKSDeploy()
                     │
                     ▼
                Deploy to EKS
```
---

# GitHub Webhook Trigger

A GitHub Webhook is the most commonly used trigger in enterprise CI/CD environments.

Instead of Jenkins continuously checking GitHub for changes, GitHub immediately notifies Jenkins whenever an event occurs.

This event-driven approach provides faster builds and reduces unnecessary server load.

---

## How GitHub Webhook Works

```text
Developer

↓

Git Push

↓

GitHub Repository

↓

Webhook Event Generated

↓

Jenkins Receives Event

↓

Pipeline Starts

↓

Build

↓

Test

↓

Deploy
```

---

## Webhook Events

A webhook can trigger Jenkins for various GitHub events.

Common events include:

```text
Push

↓

Pull Request

↓

Tag Creation

↓

Branch Creation

↓

Release
```

For CI/CD, the most commonly used event is:

```text
Push Event
```

---

## Why Webhooks Instead of Polling?

### Poll SCM

```text
Jenkins

↓

Check GitHub Every 5 Minutes

↓

Any Changes?

↓

Run Pipeline
```

Problems

- Delayed builds
- Continuous GitHub requests
- Wasted resources
- Increased Jenkins workload

---

### GitHub Webhook

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline Starts Immediately
```

Advantages

- Real-time builds
- No unnecessary polling
- Lower CPU usage
- Better scalability

---

# Poll SCM

Poll SCM is another trigger mechanism where Jenkins periodically checks the repository for changes.

---

## Workflow

```text
Jenkins Scheduler

↓

Every X Minutes

↓

Check Repository

↓

Changes Found?

↓

Yes

↓

Pipeline Starts
```

---

## Example Poll Schedule

```groovy
pollSCM('H/5 * * * *')
```

Meaning

```text
Every 5 Minutes

↓

Check Repository

↓

Run Build If Code Changed
```

---

## When Should Poll SCM Be Used?

Suitable when

- GitHub Webhooks cannot be configured.
- Repository is behind a firewall.
- Legacy Git servers are used.

Otherwise,

```text
Always Prefer

↓

GitHub Webhooks
```

---

# Cron Trigger

Sometimes pipelines should execute even if developers do not push code.

Examples include:

- Nightly builds
- Backup jobs
- Security scans
- Dependency updates
- Cleanup jobs

---

## Workflow

```text
Jenkins Scheduler

↓

Scheduled Time

↓

Pipeline Starts

↓

Execute Job
```

---

## Common Cron Examples

Run every night

```groovy
H 2 * * *
```

Run every Sunday

```groovy
H 1 * * 0
```

Run every hour

```groovy
H * * * *
```

Run every weekday

```groovy
H 9 * * 1-5
```

---

## Enterprise Example

```text
2 AM

↓

Security Scan

↓

Dependency Update

↓

Email Report
```

---

# Upstream Trigger

One pipeline automatically starts another pipeline.

Example

```text
Pipeline A

↓

Application Build

↓

Pipeline B

↓

Docker Image

↓

Pipeline C

↓

Deployment
```

---

## Enterprise Workflow

```text
Developer Push

↓

Build Pipeline

↓

Unit Test Pipeline

↓

Security Pipeline

↓

Deployment Pipeline
```

Each pipeline has a single responsibility.

---

# Remote Trigger

Jenkins jobs can also be triggered using REST APIs.

Useful for integrations with

- ServiceNow
- Jira
- Monitoring Systems
- Custom Applications
- Internal Portals

---

## Workflow

```text
External Application

↓

REST API

↓

Jenkins

↓

Pipeline Starts
```

---

# Trigger Comparison

| Trigger | Speed | Recommended |
|----------|-------|-------------|
| Manual | Slow | Development |
| GitHub Webhook | Instant | ✅ Production |
| Poll SCM | Delayed | Legacy Systems |
| Cron | Scheduled | Maintenance Jobs |
| Upstream | Instant | CI/CD Pipelines |
| Remote API | Instant | Integrations |

---

# Production Jenkinsfile

## GitHub Webhook Trigger

```groovy
pipeline {

    agent any

    triggers {

        githubPush()

    }

    stages {

        stage('Build') {

            steps {

                javaEKSPipeline()

            }

        }

        stage('Deploy') {

            when {

                branch 'main'

            }

            steps {

                EKSDeploy()

            }

        }

    }

}
```

---

## Poll SCM Example

```groovy
pipeline {

    agent any

    triggers {

        pollSCM('H/5 * * * *')

    }

    stages {

        stage('Build') {

            steps {

                javaEKSPipeline()

            }

        }

    }

}
```

---

## Cron Example

```groovy
pipeline {

    agent any

    triggers {

        cron('H 2 * * *')

    }

    stages {

        stage('Nightly Security Scan') {

            steps {

                javaEKSPipeline()

            }

        }

    }

}
```

---

# Production Trigger Flow

```text
                 Developer
                      │
                      ▼
                 Git Push
                      │
                      ▼
              GitHub Repository
                      │
             Webhook Notification
                      │
                      ▼
          Jenkins Multibranch Pipeline
                      │
                      ▼
           Load Shared Library
                      │
                      ▼
            javaEKSPipeline()
                      │
                      ▼
                  Build
                      ▼
                 Unit Test
                      ▼
                SonarQube
                      ▼
        OWASP Dependency Check
                      ▼
              Docker Build
                      ▼
               Trivy Scan
                      ▼
          Push Image to Amazon ECR
                      ▼
               EKSDeploy()
                      ▼
             Deploy to Amazon EKS
                      ▼
              Post Build Actions
```

---

# Trigger Selection Guide

```text
Developer Push

↓

GitHub Webhook

------------------------

Nightly Security Scan

↓

Cron Trigger

------------------------

Legacy Git Server

↓

Poll SCM

------------------------

Build Pipeline

↓

Upstream Trigger

------------------------

External Application

↓

Remote API Trigger
```

---

# Best Practices

## 1. Prefer GitHub Webhooks for CI/CD

GitHub Webhooks provide real-time pipeline execution and eliminate unnecessary repository polling.

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Pipeline Starts
```

Benefits

- Immediate feedback
- Reduced Jenkins workload
- Better scalability
- Lower Git server load

---

## 2. Use Poll SCM Only When Necessary

Use Poll SCM only if webhooks cannot be configured.

Suitable scenarios:

- Legacy Git servers
- Firewall restrictions
- Third-party SCM limitations

Avoid using Poll SCM in production when webhooks are available.

---

## 3. Separate Different Trigger Types

Different jobs should use different triggers based on their purpose.

Example

```text
Developer Push

↓

GitHub Webhook

-------------------------

Nightly Security Scan

↓

Cron Trigger

-------------------------

Deployment Pipeline

↓

Upstream Trigger
```

Avoid using one trigger for every job.

---

## 4. Trigger Deployments Only from Protected Branches

Even if every branch triggers a build, production deployment should happen only from trusted branches.

```groovy
when {

    branch 'main'

}
```

Feature branches should validate code but not deploy to production.

---

## 5. Keep Scheduled Jobs Independent

Nightly jobs should perform maintenance tasks only.

Examples

```text
Dependency Update

↓

Security Scan

↓

Workspace Cleanup

↓

Backup

↓

Report Generation
```

Avoid mixing scheduled maintenance with application deployments.

---

## 6. Chain Pipelines Instead of Creating One Huge Pipeline

Split CI/CD into independent jobs.

```text
Pipeline 1

↓

Build

↓

Pipeline 2

↓

Security

↓

Pipeline 3

↓

Deployment
```

This improves maintainability and simplifies troubleshooting.

---

## 7. Secure Remote Triggers

If using REST API triggers,

always require

- Authentication
- API Tokens
- HTTPS
- Authorization

Never expose anonymous remote triggers.

---

## 8. Monitor Failed Webhooks

Configure monitoring for webhook failures.

```text
GitHub

↓

Webhook Delivery

↓

HTTP Status

↓

Jenkins Response

↓

Retry if Failed
```

Webhook failures often delay CI/CD pipelines.

---

## 9. Avoid Excessive Cron Jobs

Running many scheduled jobs simultaneously can overload Jenkins.

Instead of

```text
2:00 AM

↓

100 Jobs
```

Use

```text
2:00 AM

↓

20 Jobs

↓

2:15 AM

↓

20 Jobs

↓

2:30 AM

↓

20 Jobs
```

Distribute workload across time.

---

## 10. Log Trigger Information

Every pipeline should clearly indicate why it started.

Example

```text
Started By

↓

GitHub Push

or

Cron Schedule

or

Upstream Job

or

Manual Build
```

This simplifies debugging.

---

# Troubleshooting

## 1. GitHub Webhook Not Triggering

### Symptoms

```text
Developer Push

↓

No Jenkins Build
```

### Verify

- GitHub Webhook URL
- Payload URL
- Secret
- HTTP Response
- Jenkins Reachability

---

## 2. Poll SCM Never Detects Changes

### Verify

```text
Repository URL

↓

Git Credentials

↓

Poll Schedule

↓

SCM Configuration
```

---

## 3. Cron Job Never Executes

### Verify

- Cron syntax
- Jenkins server time
- Time zone
- Job enabled
- Build history

---

## 4. Upstream Pipeline Doesn't Trigger

### Verify

```text
Pipeline A Completed

↓

Trigger Configuration

↓

Pipeline B Exists

↓

Permissions
```

---

## 5. Remote Trigger Returns 403

### Verify

- API Token
- Authentication
- User Permissions
- CSRF Protection

---

## 6. Pipeline Starts Multiple Times

### Possible Causes

- Duplicate webhooks
- Multiple triggers configured
- SCM polling and webhook enabled together

### Resolution

Keep only one trigger unless multiple are intentionally required.

---

## 7. Pull Request Doesn't Trigger Build

### Verify

```text
GitHub Plugin

↓

Multibranch Configuration

↓

PR Discovery

↓

Webhook Delivery
```

---

## 8. Build Triggered but Pipeline Stays Queued

### Verify

- Available agents
- Executor count
- Offline agents
- Queue status

---

## 9. Wrong Branch Gets Deployed

### Verify

```groovy
when {

    branch 'main'

}
```

Ensure deployment stages execute only for approved branches.

---

## 10. Scheduled Job Overloads Jenkins

### Resolution

- Stagger Cron schedules
- Use distributed agents
- Archive old builds
- Monitor executor usage

---

# Common Interview Questions

## 1. Which build trigger do you use in production and why?

### Production-Level Answer

For CI/CD pipelines, I prefer GitHub Webhooks because they provide event-driven execution. Whenever developers push code or create Pull Requests, GitHub immediately notifies Jenkins, which starts the pipeline without waiting for periodic polling. This reduces server load, provides faster feedback, and scales well for large repositories.

### Follow-Up Questions

- Why not use Poll SCM?
- How are webhooks configured?
- What happens if the webhook fails?

---

## 2. Explain the difference between GitHub Webhook and Poll SCM.

### Production-Level Answer

GitHub Webhooks are push-based, where GitHub notifies Jenkins immediately after an event occurs. Poll SCM is pull-based, where Jenkins periodically checks the repository for changes. Webhooks provide near real-time execution and lower resource usage, while Poll SCM introduces delays and unnecessary repository requests. In production, webhooks are the preferred approach unless repository constraints prevent their use.

### Follow-Up Questions

- When would you use Poll SCM?
- Which approach scales better?
- How do you troubleshoot webhook failures?

---

## 3. When would you use a Cron trigger?

### Production-Level Answer

Cron triggers are best suited for scheduled jobs that don't depend on code changes. Examples include nightly security scans, dependency updates, backup jobs, workspace cleanup, and report generation. I avoid using Cron for normal application builds because developer pushes should trigger those immediately via webhooks.

### Follow-Up Questions

- Explain Jenkins cron syntax.
- What scheduled jobs exist in your environment?
- How do you avoid running many Cron jobs simultaneously?

---

## 4. What is an Upstream Trigger?

### Production-Level Answer

An Upstream Trigger starts one Jenkins job after another completes successfully. This helps separate build, security scanning, and deployment into independent pipelines. Splitting responsibilities makes pipelines easier to maintain, troubleshoot, and reuse across projects.

### Follow-Up Questions

- Why separate pipelines?
- How do you pass artifacts between jobs?
- Would you always use downstream pipelines?

---

## 5. How do you secure Remote Triggers?

### Production-Level Answer

Remote triggers should always require authentication using API tokens or service accounts over HTTPS. Access should be restricted to authorized systems, and audit logs should be enabled. Anonymous remote triggering should never be allowed in production because it could expose deployment pipelines to unauthorized users.

### Follow-Up Questions

- What authentication methods are supported?
- Would you expose Jenkins publicly?
- How do you audit remote requests?

---

## 6. How do you troubleshoot a webhook that isn't triggering Jenkins?

### Production-Level Answer

I first verify webhook delivery in GitHub, check the HTTP response from Jenkins, ensure the webhook URL is correct, validate authentication and secrets, confirm Jenkins is reachable, and review Jenkins logs. If delivery succeeds but no build starts, I inspect the Multibranch Pipeline configuration and webhook plugin settings.

### Follow-Up Questions

- Where do you check webhook deliveries?
- Which Jenkins logs would you inspect?
- How do you manually test a webhook?

---

## 7. Why shouldn't every pipeline use the same trigger?

### Production-Level Answer

Different workloads have different requirements. Application builds should use GitHub Webhooks, maintenance tasks should use Cron schedules, deployment chains should use Upstream Triggers, and external systems should use Remote Triggers. Choosing the right trigger improves performance, reduces unnecessary builds, and keeps CI/CD workflows organized.

### Follow-Up Questions

- Which trigger do you use most frequently?
- Can multiple triggers exist in one pipeline?
- How do you prioritize builds?

---

## 8. How do enterprises manage Jenkins triggers?

### Production-Level Answer

Enterprises typically use GitHub Webhooks for CI pipelines, Multibranch Pipelines for branch discovery, Cron schedules for maintenance jobs, Upstream Triggers for deployment workflows, and Remote API triggers for integrations with external systems. Trigger configurations are standardized and monitored to ensure reliable pipeline execution.

### Follow-Up Questions

- How do you monitor failed triggers?
- How are triggers documented?
- How do you scale trigger management?

---

## 9. What improvements would you make to Jenkins trigger management?

### Production-Level Answer

I would standardize webhook usage, eliminate unnecessary Poll SCM jobs, stagger Cron schedules to balance workload, monitor webhook delivery failures, secure Remote Triggers with API tokens, and document trigger strategies for every pipeline. These improvements reduce infrastructure load and improve CI/CD reliability.

### Follow-Up Questions

- How do you monitor Jenkins performance?
- Which trigger creates the least overhead?
- How would you reduce duplicate builds?

---

## 10. Explain your production trigger architecture.

### Production-Level Answer

In our environment, developers push code to GitHub, which immediately triggers Jenkins through GitHub Webhooks. Multibranch Pipelines discover the affected branch, load the Enterprise Shared Library, execute language-specific CI/CD pipelines, and deploy only from the protected `main` branch. Scheduled jobs such as security scans and maintenance tasks use Cron triggers, while deployment pipelines can be chained using Upstream Triggers. This architecture provides fast feedback, efficient resource utilization, and scalable automation.

### Follow-Up Questions

- Why combine Multibranch Pipelines with Webhooks?
- Which jobs use Cron schedules?
- How do you secure deployment triggers?

---

# Key Takeaways

- Build Triggers determine when Jenkins pipelines start.
- GitHub Webhooks are the preferred trigger for CI/CD because they provide immediate, event-driven execution.
- Poll SCM should be used only when webhooks cannot be configured.
- Cron triggers are ideal for scheduled maintenance tasks.
- Upstream Triggers help chain independent pipelines into complete CI/CD workflows.
- Remote Triggers enable secure integrations with external systems.
- Choosing the appropriate trigger improves automation, scalability, and pipeline performance.
```