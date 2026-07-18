# 12 - Jenkins Conditional Execution

# Introduction

In a real-world CI/CD pipeline, not every stage should execute every time the pipeline runs.

For example:

- Developers should be able to build feature branches without deploying to production.
- Security scans may execute only when application code changes.
- Infrastructure scans should execute only when Terraform files are modified.
- Production deployments should occur only from the `main` branch after approval.
- Release deployments may execute only when a Git tag is created.

If Jenkins executed every stage on every pipeline run, it would result in:

- Unnecessary deployments
- Increased build time
- Wasted compute resources
- Higher infrastructure costs
- Increased production risk

To solve this problem, Jenkins provides **Conditional Execution** through the **`when` directive**.

The `when` directive allows Jenkins to evaluate one or more conditions before executing a stage.

If the condition evaluates to **true**, the stage executes.

If the condition evaluates to **false**, Jenkins skips the stage and continues with the remaining pipeline.

Conditional execution is one of the most widely used features in Declarative Pipelines because it enables organizations to build intelligent, flexible, and production-ready CI/CD workflows.

---

# Learning Objectives

After completing this chapter, you should be able to:

- Understand why conditional execution is important.
- Use the `when` directive effectively.
- Control stage execution based on:
  - Branch
  - Environment
  - Parameters
  - Git Tags
  - Build Triggers
  - Changed Files
  - Custom Expressions
- Build production-ready deployment pipelines.
- Troubleshoot skipped stages.
- Answer advanced interview questions on Jenkins conditional execution.

---

# Why Do We Need Conditional Execution?

Consider a simple pipeline.

```text
Checkout

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Docker Push

↓

Deploy Dev

↓

Deploy Test

↓

Deploy Stage

↓

Deploy Production
```

Now imagine a developer pushes code to a feature branch.

Should Jenkins deploy to Production?

Obviously not.

Yet without conditions, Jenkins has no way of distinguishing between:

- Feature branches
- Development branches
- Release branches
- Production branches

Similarly, suppose a developer updates only documentation.

Should Jenkins build Docker images?

Should it run Checkov?

Should it deploy Kubernetes resources?

Again, the answer is no.

Conditional execution solves these problems.

---

# Real Production Example

A company has four deployment environments.

```text
feature/*

↓

Build

↓

Test

↓

Stop
```

```text
develop

↓

Build

↓

Test

↓

Deploy DEV
```

```text
release/*

↓

Build

↓

Test

↓

Security Scan

↓

Deploy STAGE
```

```text
main

↓

Build

↓

Test

↓

Security Scan

↓

Deploy Production
```

Notice that the same Jenkinsfile is used for every branch.

Only the execution path changes.

This is possible because of conditional execution.

---

# Pipeline Without Conditions

Suppose a pipeline contains four stages.

```groovy
stages {

    stage("Build") {

    }

    stage("Test") {

    }

    stage("Deploy") {

    }

    stage("Production") {

    }

}
```

Execution

```text
Build

↓

Test

↓

Deploy

↓

Production
```

Every stage executes.

Even if deployment is not required.

---

# Pipeline With Conditions

```groovy
stage("Deploy") {

    when {

        expression {

            params.DEPLOY

        }

    }

    steps {

        sh "./deploy.sh"

    }

}
```

Execution

```text
Build

↓

Test

↓

Deploy?

│

├── Yes

│     ↓

│   Deploy

│

└── No

      ↓

    Skip Stage
```

The deployment stage executes only when the condition evaluates to **true**.

---

# How Jenkins Evaluates Conditions

Every time Jenkins reaches a stage containing a `when` block, it performs the following sequence.

```text
Previous Stage Finished

        │

        ▼

Read when Condition

        │

        ▼

Evaluate Condition

        │

   ┌────┴─────┐

   │          │

 TRUE       FALSE

   │          │

   ▼          ▼

Run Stage   Skip Stage
```

This evaluation happens before Jenkins executes any step inside the stage.

---

# What is the when Directive?

The `when` directive tells Jenkins whether a stage should execute.

General Syntax

```groovy
stage("Deploy") {

    when {

        condition

    }

    steps {

        ...

    }

}
```

Every Declarative Pipeline can contain multiple `when` blocks.

Each stage evaluates its own conditions independently.

---

# Trainer Example

The trainer uses the following condition.

```groovy
stage("Deploy") {

    when {

        expression {

            "$params.DEPLOY" == "true"

        }

    }

    steps {

        sh """

        echo "Deploying"

        """

    }

}
```

Although this works, a cleaner approach is:

```groovy
stage("Deploy") {

    when {

        expression {

            params.DEPLOY

        }

    }

    steps {

        sh "echo Deploying"

    }

}
```

Since `DEPLOY` is already a Boolean Parameter, there is no need to compare it with a string.

---

# Benefits of Conditional Execution

Conditional execution provides several advantages.

## Improved Performance

Skip unnecessary stages.

Instead of executing

```text
Checkout

↓

Build

↓

Test

↓

Docker

↓

Deploy
```

Jenkins may execute only

```text
Checkout

↓

Build

↓

Test
```

saving several minutes.

---

## Faster Feedback

Developers receive build results sooner because unnecessary stages are skipped.

---

## Better Security

Production deployment stages can be restricted to approved branches.

Example

```text
main

↓

Deploy Production
```

All other branches automatically skip deployment.

---

## Reduced Infrastructure Cost

Skipping Docker builds, container scans, and Kubernetes deployments reduces compute consumption.

---

## Cleaner Pipelines

Instead of maintaining multiple Jenkinsfiles

```text
Build Pipeline

Deploy Pipeline

Release Pipeline

Hotfix Pipeline
```

one parameterized pipeline can support all workflows.

---

# Types of Conditions Supported by Jenkins

Declarative Pipelines support multiple condition types.

| Condition | Purpose |
|-----------|---------|
| branch | Execute on a specific Git branch |
| expression | Evaluate custom Groovy expressions |
| environment | Check environment variable values |
| equals | Compare values |
| tag | Execute when building Git tags |
| buildingTag | Check whether the current build is a tag |
| changelog | Search commit messages |
| changeset | Detect modified files |
| triggeredBy | Detect build trigger |
| anyOf | Logical OR |
| allOf | Logical AND |
| not | Negate conditions |

Each of these conditions solves a different production requirement.

---

# Branch Condition

## Purpose

The `branch` condition executes a stage only when the current branch matches the specified value.

Syntax

```groovy
when {

    branch "main"

}
```

Execution Flow

```text
Current Branch

        │

        ▼

main ?

   │

┌──┴──┐

│     │

Yes   No

│     │

▼     ▼

Run   Skip
```

Production Example

```groovy
stage("Deploy Production") {

    when {

        branch "main"

    }

    steps {

        sh "./deploy-prod.sh"

    }

}
```

Only the `main` branch can deploy to Production.

Feature branches continue to build and test but never deploy.

---

# Why Use Branch Conditions?

Suppose a Git repository contains:

```text
main

develop

feature/login

feature/payment

release/1.2

hotfix/api
```

Not every branch should perform identical actions.

Typical production workflow:

| Branch | Action |
|---------|--------|
| feature/* | Build + Test |
| develop | Build + Test + Deploy DEV |
| release/* | Build + Security Scan + Deploy STAGE |
| main | Build + Security Scan + Deploy PROD |

The `branch` condition makes this possible without maintaining separate pipelines.

---

# Branch Pattern Matching

The `branch` condition also supports pattern matching, making it suitable for repositories that follow Git branching strategies such as Git Flow or Feature Branching.

Example

```groovy
stage("Deploy Development") {

    when {

        branch "develop"

    }

    steps {

        sh "./deploy-dev.sh"

    }

}
```

Example

```groovy
stage("Release Deployment") {

    when {

        branch "release/*"

    }

    steps {

        sh "./deploy-stage.sh"

    }

}
```

Example

```groovy
stage("Hotfix Validation") {

    when {

        branch "hotfix/*"

    }

    steps {

        sh "./run-hotfix-tests.sh"

    }

}
```

---

# Expression Condition

## Purpose

The `expression` condition allows Jenkins to evaluate a custom Groovy expression.

It is the most flexible condition because it can evaluate:

- Parameters
- Variables
- Environment Variables
- File existence
- Numeric values
- Boolean values
- Custom logic

Syntax

```groovy
when {

    expression {

        expression_here

    }

}
```

Example

```groovy
when {

    expression {

        params.DEPLOY

    }

}
```

Pipeline Flow

```text
Boolean Parameter

↓

params.DEPLOY

↓

True?

│

├── Yes

│      ▼

│    Deploy

│

└── No

       ▼

     Skip
```

---

# Production Example

Deploy only when the deployment checkbox is selected.

```groovy
parameters {

    booleanParam(
        name: 'DEPLOY',
        defaultValue: false
    )

}

stage("Deploy") {

    when {

        expression {

            params.DEPLOY

        }

    }

    steps {

        sh "./deploy.sh"

    }

}
```

---

# Multiple Conditions Using Expression

Example

```groovy
when {

    expression {

        params.DEPLOY &&
        env.BRANCH_NAME == "main"

    }

}
```

Deployment occurs only if:

- DEPLOY is true
- Current branch is main

---

# Environment Condition

## Purpose

Executes a stage only when an environment variable contains a specific value.

Syntax

```groovy
when {

    environment name: "ENV",

    value: "prod"

}
```

Pipeline

```text
Environment Variable

↓

ENV

↓

prod ?

│

├── Yes

│      ▼

│    Execute

│

└── No

       ▼

      Skip
```

---

# Production Example

```groovy
environment {

    ENV="prod"

}

stage("Deploy") {

    when {

        environment

        name: "ENV",

        value: "prod"

    }

    steps {

        sh "./deploy-prod.sh"

    }

}
```

---

# Equals Condition

## Purpose

Compares two values.

Syntax

```groovy
when {

    equals expected: "prod",

    actual: params.ENVIRONMENT

}
```

Production Example

```groovy
parameters {

    choice(

        name: "ENVIRONMENT",

        choices: [

            "dev",

            "test",

            "stage",

            "prod"

        ]

    )

}

stage("Production") {

    when {

        equals

        expected: "prod",

        actual: params.ENVIRONMENT

    }

    steps {

        sh "./deploy-prod.sh"

    }

}
```

---

# Tag Condition

## Purpose

Executes a stage only when the build is triggered from a Git tag.

Typical use case

```text
Git Tag

↓

v1.0.0

↓

Production Release
```

Syntax

```groovy
when {

    tag "v*"

}
```

Production Example

```groovy
stage("Release") {

    when {

        tag "v*"

    }

    steps {

        sh "./release.sh"

    }

}
```

Only version tags trigger releases.

---

# BuildingTag Condition

## Purpose

Checks whether the current build originates from any Git tag.

Syntax

```groovy
when {

    buildingTag()

}
```

Difference

```text
tag

↓

Checks specific pattern

buildingTag

↓

Checks whether current build is any Git tag
```

Production Example

```groovy
stage("Publish Release") {

    when {

        buildingTag()

    }

    steps {

        sh "./publish.sh"

    }

}
```

---

# Changelog Condition

## Purpose

Evaluates commit messages.

Syntax

```groovy
when {

    changelog ".*JIRA.*"

}
```

Pipeline

```text
Commit

↓

Fixed Login

↓

No Match

↓

Skip
```

```text
Commit

↓

JIRA-210 Payment Fix

↓

Match

↓

Execute
```

Production Example

Run additional validation only for commits containing release keywords.

```groovy
stage("Release Validation") {

    when {

        changelog ".*RELEASE.*"

    }

    steps {

        sh "./validate-release.sh"

    }

}
```

---

# Changeset Condition

## Purpose

Executes a stage only when specified files are modified.

One of the most useful conditions in production pipelines.

Syntax

```groovy
when {

    changeset "**/*.tf"

}
```

Execution Flow

```text
Git Commit

↓

Files Changed

↓

Terraform?

│

├── Yes

│      ▼

│   Run Checkov

│

└── No

       ▼

     Skip
```

---

# Production Example

Run Checkov only when Infrastructure as Code changes.

```groovy
stage("IaC Scan") {

    when {

        changeset "**/*.tf"

    }

    steps {

        sh """

        checkov -d terraform/

        """

    }

}
```

This reduces pipeline execution time by skipping unnecessary infrastructure scans.

---

# Another Production Example

Run Docker build only when Docker-related files change.

```groovy
stage("Docker Build") {

    when {

        anyOf {

            changeset "Dockerfile"

            changeset "docker/**"

        }

    }

    steps {

        sh """

        docker build -t catalogue:${BUILD_NUMBER} .

        """

    }

}
```

---

# TriggeredBy Condition

## Purpose

Executes a stage based on the event that started the pipeline.

Examples of triggers

- Manual Build
- GitHub Webhook
- Timer
- Upstream Pipeline

Syntax

```groovy
when {

    triggeredBy "SCMTrigger"

}
```

Production Example

Deploy automatically only for webhook-triggered builds.

```groovy
stage("Deploy") {

    when {

        triggeredBy "SCMTrigger"

    }

    steps {

        sh "./deploy.sh"

    }

}
```

---

# anyOf Condition

## Purpose

Implements logical OR.

The stage executes when at least one condition evaluates to true.

Syntax

```groovy
when {

    anyOf {

        branch "develop"

        branch "main"

    }

}
```

Execution

```text
develop ?

│

True

↓

Run
```

```text
main ?

│

True

↓

Run
```

```text
feature/login

↓

False

↓

Skip
```

Production Example

Deploy to shared environments from either `develop` or `main`.

```groovy
stage("Deploy Shared") {

    when {

        anyOf {

            branch "develop"

            branch "main"

        }

    }

    steps {

        sh "./deploy.sh"

    }

}
```

---

# allOf Condition

## Purpose

Implements logical AND.

Every condition must evaluate to true.

Syntax

```groovy
when {

    allOf {

        branch "main"

        expression {

            params.DEPLOY

        }

    }

}
```

Execution

```text
Branch = main

↓

Yes

↓

DEPLOY=true

↓

Yes

↓

Execute Stage
```

If either condition fails, Jenkins skips the stage.

---

# Production Example

Deploy to Production only when:

- Branch is main
- Deployment parameter is enabled

```groovy
stage("Production Deployment") {

    when {

        allOf {

            branch "main"

            expression {

                params.DEPLOY

            }

        }

    }

    steps {

        sh "./deploy-prod.sh"

    }

}
```

---

# not Condition

## Purpose

Negates another condition.

Syntax

```groovy
when {

    not {

        branch "main"

    }

}
```

Execution

```text
Current Branch

↓

main ?

│

Yes

↓

Skip
```

```text
feature/login

↓

Not main

↓

Execute
```

Production Example

Skip Production-only validation for feature branches.

```groovy
stage("Feature Validation") {

    when {

        not {

            branch "main"

        }

    }

    steps {

        sh "./run-feature-tests.sh"

    }

}
```

---