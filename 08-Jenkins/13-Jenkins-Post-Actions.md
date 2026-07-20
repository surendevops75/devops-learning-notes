# Jenkins Post Actions

## Introduction

A Jenkins pipeline does not end after the last stage.

After the pipeline finishes, Jenkins can execute additional actions.

These are called **Post Actions**.

---

Example

```text
Build
   │
   ▼
Test
   │
   ▼
Deploy
   │
   ▼
Post Actions
```

---

# Why Do We Need Post Actions?

After a pipeline completes, we may need to:

```text
Send Notifications

Archive Reports

Clean Workspace

Publish Test Results

Generate Logs
```

---

Without Post Actions

```text
Pipeline Ends

↓

No Cleanup

No Notifications

No Reports
```

---

With Post Actions

```text
Pipeline Ends

↓

Cleanup

↓

Notification

↓

Archive Reports
```

---

# What is the post Block?

The `post` block defines actions that execute **after** the pipeline or stage completes.

---

Workflow

```text
Pipeline Execution
        │
        ▼
Pipeline Finished
        │
        ▼
Execute Post Actions
```

---

# Syntax

```groovy
pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                sh "mvn clean package"

            }

        }

    }

    post {

        always {

            echo "Pipeline Completed"

        }

    }

}
```

---

# Post Conditions

Jenkins supports multiple post conditions.

```text
always

success

failure

unstable

changed

aborted

cleanup
```

---

Each condition executes based on the pipeline result.

---

# Pipeline Flow

```text
Pipeline
    │
    ▼
Result
    │
    ├── Success
    ├── Failure
    ├── Unstable
    ├── Aborted
    └── Changed
          │
          ▼
     Execute Post Action
```

---

# always

Executes regardless of the pipeline result.

---

Workflow

```text
Success
    │
    ▼
Run
```

```text
Failure
    │
    ▼
Run
```

```text
Aborted
    │
    ▼
Run
```

---

Syntax

```groovy
post {

    always {

        echo "Pipeline Finished"

    }

}
```

---

Production Example

```text
Pipeline Ends

↓

Archive Logs

↓

Clean Workspace
```

---

# success

Executes only when the pipeline completes successfully.

---

Syntax

```groovy
post {

    success {

        echo "Deployment Successful"

    }

}
```

---

Workflow

```text
Pipeline Success ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

Production Example

```text
Build Success

↓

Deploy Success

↓

Notify Team
```

---

# failure

Executes only when the pipeline fails.

---

Syntax

```groovy
post {

    failure {

        echo "Build Failed"

    }

}
```

---

Workflow

```text
Pipeline Failed ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

Production Example

```text
Build Failed

↓

Collect Logs

↓

Notify DevOps Team
```

---

# unstable

Executes when the pipeline completes with an **Unstable** status.

---

An unstable build means the pipeline completed, but some validations failed.

Example

```text
Build Success

↓

Unit Tests Passed

↓

Few Test Cases Failed

↓

Pipeline = Unstable
```

---

Syntax

```groovy
post {

    unstable {

        echo "Build is Unstable"

    }

}
```

---

Workflow

```text
Pipeline Result

↓

Unstable ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

Production Example

```text
Application Built

↓

JUnit Reports

↓

Some Tests Failed

↓

Notify QA Team
```

---

# changed

Executes when the pipeline result changes compared to the previous build.

---

Example

```text
Previous Build

↓

Failure
```

---

```text
Current Build

↓

Success
```

---

Result

```text
Pipeline Status Changed
```

---

Syntax

```groovy
post {

    changed {

        echo "Build Status Changed"

    }

}
```

---

Workflow

```text
Previous Build

↓

Current Build

↓

Status Changed ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

Production Example

```text
Yesterday

↓

Build Failed
```

---

```text
Today

↓

Build Passed
```

---

```text
Notify Team

↓

Issue Resolved
```

---

# aborted

Executes when a build is aborted manually or automatically.

---

Example

```text
Pipeline Running

↓

Abort Build

↓

Execute aborted
```

---

Syntax

```groovy
post {

    aborted {

        echo "Pipeline Aborted"

    }

}
```

---

Workflow

```text
Pipeline

↓

Aborted ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

Production Example

```text
Deployment Started

↓

Critical Issue Found

↓

Abort Deployment

↓

Notify Team
```

---

# cleanup

Executes after all other post conditions.

It is mainly used for cleanup activities.

---

Syntax

```groovy
post {

    cleanup {

        cleanWs()

    }

}
```

---

Workflow

```text
Pipeline Finished

↓

Post Actions

↓

Cleanup
```

---

Production Example

```text
Pipeline Completed

↓

Delete Workspace

↓

Free Disk Space
```

---

# Complete Example

```groovy
pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                sh "mvn clean package"

            }

        }

    }

    post {

        always {

            echo "Pipeline Completed"

        }

        success {

            echo "Deployment Successful"

        }

        failure {

            echo "Deployment Failed"

        }

        cleanup {

            cleanWs()

        }

    }

}
```

---

# Production Pipeline

```text
Checkout

↓

Build

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Deploy

↓

Post Actions

├── Success Notification

├── Failure Notification

├── Archive Reports

└── Cleanup
```

---

# Best Practices

## Use `always` for Cleanup Tasks

The `always` block executes regardless of the pipeline result.

Suitable for

```text
Archive Logs

Publish Reports

Clean Workspace
```

---

## Use `success` for Notifications

Notify users only after a successful build.

Example

```text
Build Success

↓

Deployment Success

↓

Notify Team
```

---

## Use `failure` for Alerts

Notify the DevOps team immediately when a pipeline fails.

Example

```text
Build Failed

↓

Collect Logs

↓

Send Alert
```

---

## Keep Cleanup Separate

Use the `cleanup` block for deleting temporary files and workspaces.

Avoid placing cleanup commands inside `always`.

---

## Archive Reports Before Cleanup

Good Flow

```text
Generate Reports

↓

Archive Reports

↓

Cleanup Workspace
```

---

## Don't Put Build Logic Inside post

Bad

```text
Build

↓

post

↓

Docker Build
```

---

Good

```text
Build Stage

↓

post

↓

Notification
```

The `post` block should only handle post-build activities.

---

# Troubleshooting

## `post` Block Not Executing

Verify the `post` block is defined correctly.

```groovy
post {

    always {

        echo "Completed"

    }

}
```

---

## `success` Not Running

Check whether the pipeline actually completed successfully.

Example

```text
Unit Test Failed

↓

Pipeline = Failure

↓

success Skipped
```

---

## `failure` Not Triggering

Verify the pipeline status.

A manually aborted pipeline executes the `aborted` block, not `failure`.

---

## Workspace Not Cleaned

Verify

```groovy
cleanup {

    cleanWs()

}
```

Also ensure the Workspace Cleanup Plugin is installed.

---

## Notification Not Sent

Verify

```text
Notification Logic

↓

Credentials

↓

Network Connectivity
```

---

## Reports Missing

Archive reports before cleanup.

Correct Flow

```text
Generate Report

↓

Archive Report

↓

Cleanup
```

---

# Common Interview Questions

## 1. What is the `post` block in Jenkins?

### Short Answer

The `post` block defines actions executed after a pipeline or stage finishes.

### Detailed Explanation

It is commonly used for notifications, report publishing, cleanup, and logging. The executed action depends on the pipeline result.

### Production Example

Archive test reports and clean the workspace after every pipeline.

### Follow-Up Questions

- Can `post` be used inside a stage?
- When is it executed?

---

## 2. What is the difference between `always` and `success`?

### Short Answer

`always` executes for every pipeline result, while `success` executes only after a successful build.

### Detailed Explanation

Use `always` for cleanup and report publishing. Use `success` for success notifications or deployment confirmation.

### Production Example

```text
always

↓

Archive Logs
```

```text
success

↓

Notify Deployment Success
```

### Follow-Up Questions

- Which one is suitable for cleanup?
- Which one is used for deployment notifications?

---

## 3. When is the `failure` block executed?

### Short Answer

It executes only when the pipeline fails.

### Detailed Explanation

The `failure` block is useful for collecting logs, sending alerts, and creating incident tickets.

### Production Example

```text
Build Failed

↓

Notify DevOps Team
```

### Follow-Up Questions

- Does it execute for aborted builds?
- Can multiple post conditions exist together?

---

## 4. What is an Unstable Build?

### Short Answer

An unstable build completes successfully but contains issues such as failed test cases.

### Detailed Explanation

Compilation may succeed, but quality checks or unit tests can mark the build as unstable.

### Production Example

```text
Build

↓

JUnit Tests

↓

Some Tests Failed

↓

Unstable
```

### Follow-Up Questions

- What causes an unstable build?
- Is an unstable build considered successful?

---

## 5. What is the Purpose of the `cleanup` Block?

### Short Answer

It performs cleanup activities after all other post actions finish.

### Detailed Explanation

Typical tasks include deleting the workspace and removing temporary files.

### Production Example

```text
Archive Reports

↓

Cleanup Workspace
```

### Follow-Up Questions

- Why should cleanup be the last step?
- What command is commonly used inside cleanup?

---

## 6. What is the Difference Between `failure` and `aborted`?

### Short Answer

`failure` runs for failed builds, while `aborted` runs when a build is stopped.

### Detailed Explanation

A failed build indicates an execution error, whereas an aborted build indicates manual or automatic cancellation.

### Production Example

```text
Compilation Error

↓

failure
```

```text
Manual Stop

↓

aborted
```

### Follow-Up Questions

- Can both execute together?
- Who typically aborts a pipeline?

---

## 7. What Does the `changed` Condition Do?

### Short Answer

It executes when the current build result differs from the previous build.

### Detailed Explanation

This helps notify teams when a failed pipeline becomes successful or vice versa.

### Production Example

```text
Previous Build

↓

Failure
```

```text
Current Build

↓

Success

↓

Notify Team
```

### Follow-Up Questions

- Why is `changed` useful?
- How does Jenkins determine the previous build status?

---

## 8. Can a Pipeline Have Multiple Post Conditions?

### Short Answer

Yes.

### Detailed Explanation

A single `post` block can contain `always`, `success`, `failure`, `unstable`, `aborted`, `changed`, and `cleanup`.

### Production Example

```text
Pipeline Finished

↓

Success

↓

Notify
```

```text
Pipeline Finished

↓

Failure

↓

Collect Logs
```

### Follow-Up Questions

- Which block executes last?
- Can multiple conditions run in one build?

---

## 9. Where Are Post Actions Commonly Used?

### Short Answer

After build, test, security scan, or deployment stages.

### Detailed Explanation

They automate notifications, report publishing, workspace cleanup, and artifact handling.

### Production Example

```text
Deploy

↓

Notify

↓

Archive Reports

↓

Cleanup
```

### Follow-Up Questions

- Are post actions limited to pipelines?
- Can they be used after individual stages?

---

## 10. Why Are Post Actions Important?

### Short Answer

They automate activities after pipeline execution.

### Detailed Explanation

Without post actions, teams would manually collect logs, send notifications, and clean workspaces after every build.

### Production Example

```text
Pipeline Finished

↓

Automatic Cleanup

↓

Automatic Notification

↓

Ready for Next Build
```

### Follow-Up Questions

- Which post condition do you use most frequently?
- How do post actions improve CI/CD?

---

# Key Takeaways

```text
The post block executes after a pipeline or stage finishes.

always executes regardless of pipeline status.

success executes only after successful builds.

failure executes only after failed builds.

unstable executes for unstable builds.

changed executes when the build result changes.

aborted executes for cancelled builds.

cleanup executes after all other post actions.

Post actions are commonly used for notifications, report publishing, log collection, and workspace cleanup.

Using post actions keeps CI/CD pipelines clean, automated, and production-ready.
```