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