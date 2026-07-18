# Jenkins Jobs and Build Process

# Introduction

Jenkins automates software delivery by executing **Jobs**.

Whenever a developer commits code or manually triggers a job, Jenkins starts a **Build**.

Understanding how Jenkins Jobs and Builds work is essential because every CI/CD pipeline ultimately executes as a Jenkins Job.

---

# What is a Job?

A Job is a task or automation configured in Jenkins.

It contains all the instructions required to perform a specific activity.

Examples

- Build Java Application
- Build Docker Image
- Deploy Kubernetes Application
- Execute Terraform
- Run Unit Tests

Think of a Job as a blueprint.

The blueprint defines **what needs to be done**.

---

# What is a Build?

A Build is one execution (run) of a Job.

Example

Suppose we have a Job named:

```text
Catalogue-Build
```

Each execution generates a build.

```text
Build #1

↓

Build #2

↓

Build #3

↓

Build #4
```

Every build has:

- Build Number
- Build Status
- Console Output
- Execution Time
- Build Logs

---

# Job vs Build

| Job | Build |
|------|--------|
| Blueprint | Execution |
| Created Once | Runs Multiple Times |
| Contains Configuration | Contains Execution Details |
| Defines Pipeline | Executes Pipeline |

Example

```text
Job

↓

Build #1

↓

Build #2

↓

Build #3
```

---

# Jenkins Build Lifecycle

Whenever Jenkins starts a build, it follows a sequence.

```text
Job Triggered

↓

Allocate Agent

↓

Clone Repository

↓

Checkout Branch

↓

Install Dependencies

↓

Build Application

↓

Run Unit Tests

↓

Security Scans

↓

Create Artifact

↓

Archive Artifact

↓

Post Build Actions

↓

Build Completed
```

---

# Build Triggers

A Jenkins Job can start in multiple ways.

## Manual Trigger

Developer clicks

```text
Build Now
```

---

## GitHub Webhook

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins
```

---

## Poll SCM

Jenkins checks Git periodically.

Example

```text
Every 5 Minutes
```

If code changes are detected,

Pipeline starts.

---

## Scheduled Build

Using Cron

Example

```text
Every Day

↓

12:00 AM
```

Useful for

- Nightly Builds
- Cleanup Jobs
- Backup Jobs

---

## Trigger From Another Job

Example

```text
Build Application

↓

Deploy Application

↓

Run Integration Tests
```

One pipeline automatically starts another.

---

# Build Status

Every Jenkins build ends with a status.

## Success

```text
SUCCESS
```

Everything executed successfully.

---

## Failed

```text
FAILED
```

One or more stages failed.

Example

```text
Compilation Error

↓

Pipeline Stops
```

---

## Aborted

```text
ABORTED
```

Build manually stopped.

---

## Unstable

```text
UNSTABLE
```

Build completed but tests failed.

Example

```text
Application Built Successfully

↓

Unit Tests Failed
```

---

# Build Number

Every execution receives a unique number.

Example

```text
Build #1

↓

Build #2

↓

Build #3

↓

Build #4
```

Interview Question

Can two builds have the same number?

Answer

```text
No
```

Each build number is unique within a job.

---

# Console Output

Every build stores logs.

Example

```text
Started by GitHub Push

↓

Cloning Repository

↓

Running Maven

↓

Running Tests

↓

Docker Build

↓

Finished SUCCESS
```

Console Output is the first place engineers check during failures.

---

# Workspace

During execution Jenkins creates a workspace.

Example

```text
/var/lib/jenkins/workspace/catalogue
```

Workspace contains

- Source Code
- Dependencies
- Temporary Files
- Build Output

Workspace is recreated or reused depending on job configuration.

---

# Build Artifacts

Artifacts are files generated during a build.

Examples

```text
JAR

WAR

ZIP

Docker Image
```

Artifacts are usually stored in

```text
JFrog Artifactory

↓

Nexus

↓

Amazon ECR

↓

Docker Hub
```

---

# Build History

Jenkins stores every execution.

Example

```text
Build #20

SUCCESS

↓

Build #21

FAILED

↓

Build #22

SUCCESS
```

History allows teams to investigate failures.

---

# Build Queue

If all agents are busy,

Jobs wait in the queue.

```text
Job 1

↓

Running

----------------

Job 2

↓

Waiting

----------------

Job 3

↓

Waiting
```

Once an agent becomes available,

Next build starts automatically.

---

# Build Parameters

Jobs can accept user inputs.

Example

```text
Environment

DEV

QA

UAT

PROD
```

Pipeline Example

```groovy
pipeline {

    agent any

    parameters {

        choice(
            name: 'ENV',
            choices: ['DEV','QA','PROD'],
            description: 'Select Environment'
        )

    }

    stages {

        stage('Deploy') {

            steps {

                echo "Deploying to ${params.ENV}"

            }

        }

    }

}
```

---

# Real Production Build Flow

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins Job

↓

Build #245

↓

Linux Agent

↓

Clone Repository

↓

Maven Build

↓

JUnit Tests

↓

SonarQube

↓

Docker Build

↓

Trivy Image Scan

↓

Push Image To Amazon ECR

↓

Archive Reports

↓

SUCCESS
```

---

# Build Retention

Keeping every build forever consumes storage.

Example

```text
Build #1

↓

Build #2

↓

...

↓

Build #5000
```

Production Best Practice

Keep only

```text
Last 20 Builds

or

Last 30 Days
```

Old builds should be deleted automatically.

---

# Build Reports

Jenkins can publish reports such as

- JUnit Reports
- Code Coverage
- SonarQube Reports
- HTML Reports

These reports help developers analyze build quality.

---

# Best Practices

- Keep workspaces clean.
- Archive only required artifacts.
- Configure build retention.
- Use meaningful job names.
- Monitor build duration.
- Fail builds immediately on critical issues.
- Store artifacts in a repository instead of Jenkins.

---

# Common Interview Questions

## What is the difference between a Job and a Build?

### Short Answer

A Job defines the automation process, while a Build is one execution of that job.

### Detailed Explanation

A Job acts as a reusable template. Every time the job runs, Jenkins creates a new build with its own logs, status, artifacts, and build number.

### Production Example

A Jenkins Job named `payment-build` executes every time code is pushed. Each execution becomes Build #101, Build #102, Build #103, and so on.

### Follow-up Questions

- Can a Job have multiple Builds?
- What is a Build Number?
- Where are Build Logs stored?

---

## What is a Workspace?

### Short Answer

A Workspace is the directory where Jenkins checks out source code and performs the build.

### Detailed Explanation

Each build uses a workspace to clone the repository, compile the application, generate artifacts, and store temporary files during execution.

### Production Example

The `catalogue` service is built in:

```text
/var/lib/jenkins/workspace/catalogue
```

### Follow-up Questions

- Can Workspaces be cleaned?
- Does every Job have a Workspace?
- Where is the default Workspace located?

---

## What is Console Output?

### Short Answer

Console Output is the log generated during a Jenkins build.

### Detailed Explanation

It records every command executed during the pipeline, making it the primary source for troubleshooting failed builds.

### Production Example

A Maven build fails due to a missing dependency. The Console Output clearly shows the error message and the stage where it occurred.

### Follow-up Questions

- How do you debug a failed Build?
- Where are Build Logs stored?
- Can Console Output be downloaded?

---

## What is Build Retention?

### Short Answer

Build Retention automatically removes old builds to save disk space.

### Detailed Explanation

Production Jenkins servers often execute thousands of builds. Retention policies help manage storage by deleting old logs, artifacts, and workspaces after a defined period or build count.

### Production Example

A Jenkins Job is configured to keep only the latest 30 builds, preventing the server from running out of disk space.

### Follow-up Questions

- Why is Build Retention important?
- Can old Builds be restored?
- How do you archive important artifacts?

---

# Production Scenarios

## Scenario 1

A build is stuck in the queue.

### Solution

Check whether an appropriate Agent is available and online.

---

## Scenario 2

The build fails during the Maven stage.

### Solution

Open the Console Output and review the Maven error messages to identify the root cause.

---

## Scenario 3

The Jenkins server runs out of storage.

### Solution

Configure Build Retention, clean old workspaces, and archive artifacts in an external repository such as JFrog Artifactory or Amazon ECR.

---

## Scenario 4

A deployment should occur only after a successful build.

### Solution

Configure downstream jobs or include deployment stages that execute only when the build status is SUCCESS.

---

# Key Takeaways

```text
A Job defines the CI/CD process.

A Build is one execution of a Job.

Every Build has a unique Build Number.

Console Output is the primary source for troubleshooting.

Workspaces contain source code and temporary build files.

Artifacts should be stored in external repositories.

Configure Build Retention to prevent disk space issues.

Jenkins automatically queues builds when Agents are busy.
```