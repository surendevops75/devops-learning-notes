# Conditional Execution

## Introduction

Not every stage in a Jenkins pipeline should execute for every build.

Example

```text
Feature Branch
      │
      ▼
Build

Test

Skip Deploy
```

---

```text
Main Branch
     │
     ▼
Build

Test

Deploy
```

---

Conditional execution helps Jenkins decide whether a stage should run or be skipped.

---

# Why Do We Need Conditional Execution?

Without conditions,

every stage executes.

```text
Checkout
     │
     ▼
Build
     │
     ▼
Test
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
Deploy
```

---

Problem

```text
Long Build Time

Unnecessary Resource Usage

Accidental Deployments
```

---

Conditional execution solves these problems.

---

# What is Conditional Execution?

Conditional execution allows Jenkins to execute stages only when specified conditions are satisfied.

---

Workflow

```text
Evaluate Condition
        │
        ▼
   True or False
        │
 ┌──────┴──────┐
 │             │
True         False
 │             │
 ▼             ▼
Run Stage   Skip Stage
```

---

# Jenkins when Directive

The `when` directive is used to define stage execution conditions.

---

Syntax

```groovy
stage("Deploy") {

    when {
        branch "main"
    }

    steps {
        sh "./deploy.sh"
    }

}
```

---

Execution Flow

```text
Stage Starts
     │
     ▼
Evaluate when
     │
     ▼
Condition True ?
 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

# Branch Condition

Executes a stage only when the current Git branch matches.

---

Syntax

```groovy
when {

    branch "main"

}
```

---

Example

```groovy
stage("Deploy") {

    when {

        branch "main"

    }

    steps {

        sh "./deploy.sh"

    }

}
```

---

Workflow

```text
Current Branch
      │
      ▼
main ?
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
feature/*
      │
      ▼
Build

Test
```

---

```text
main
 │
 ▼
Build

Security Scan

Deploy
```

---

# Expression Condition

Executes a stage using custom Groovy expressions.

---

Syntax

```groovy
when {

    expression {

        params.DEPLOY

    }

}
```

---

Example

```groovy
parameters {

    booleanParam(
        name: "DEPLOY",
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

Workflow

```text
DEPLOY=true ?
      │
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
Build
   │
   ▼
Security Scan
   │
   ▼
DEPLOY Parameter
   │
 ┌─┴─┐
 │   │
No  Yes
 │   │
 ▼   ▼
End Deploy
```

---

# Environment Condition

Runs a stage when an environment variable matches the specified value.

---

Syntax

```groovy
when {

    environment name: "ENV",
                value: "prod"

}
```

---

Example

```groovy
environment {

    ENV = "prod"

}
```

---

Workflow

```text
ENV=prod ?
    │
┌───┴────┐
│        │
Yes      No
│        │
▼        ▼
Run     Skip
```

---

# Equals Condition

Compares two values.

---

Syntax

```groovy
when {

    equals expected: "prod",
           actual: params.ENV

}
```

---

Workflow

```text
Compare Values
      │
      ▼
Equal ?
 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

# Tag Condition

Runs the stage only when the build is triggered by a Git tag.

---

Example

```text
Git Tag

↓

v2.0.0

↓

Release Pipeline
```

---

Syntax

```groovy
when {

    tag "v*"

}
```

---

# buildingTag

Runs the stage if the current build originates from any Git tag.

---

Syntax

```groovy
when {

    buildingTag()

}
```

---

Difference

| tag | buildingTag |
|------|-------------|
| Matches pattern | Checks any tag |
| Example: v* | Any Git tag |

---

# changeset

Runs a stage only when specific files are modified.

---

Syntax

```groovy
when {

    changeset "**/*.tf"

}
```

---

Workflow

```text
Terraform Changed ?
        │
 ┌──────┴──────┐
 │             │
Yes           No
 │             │
 ▼             ▼
Run         Skip
```

---

Production Example

```text
README Changed

↓

Skip Checkov
```

---

```text
Terraform Changed

↓

Run Checkov
```

---

# changelog

Runs a stage based on commit messages.

---

Syntax

```groovy
when {

    changelog ".*HOTFIX.*"

}
```

---

Example

```text
Commit

↓

HOTFIX

↓

Regression Test
```

---

# triggeredBy Condition

Runs a stage based on how the pipeline was triggered.

---

Common Triggers

```text
GitHub Webhook

Manual Build

Cron Schedule

Upstream Pipeline
```

---

Syntax

```groovy
when {

    triggeredBy "SCMTrigger"

}
```

---

Workflow

```text
Pipeline Started
       │
       ▼
SCM Trigger ?
 ┌─────┴─────┐
 │           │
Yes         No
 │           │
 ▼           ▼
Run        Skip
```

---

Production Example

```text
Git Push

↓

GitHub Webhook

↓

Deploy
```

---

```text
Manual Build

↓

Skip Deployment
```

---

# anyOf Condition

Runs a stage if **any one** condition is true.

---

Syntax

```groovy
when {

    anyOf {

        branch "develop"

        branch "main"

    }

}
```

---

Workflow

```text
Branch = develop ?

        OR

Branch = main ?

        │

        ▼

Any True ?
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
develop

↓

Deploy DEV
```

---

```text
main

↓

Deploy PROD
```

---

```text
feature/login

↓

Skip Deployment
```

---

# allOf Condition

Runs a stage only if **all conditions** are true.

---

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

---

Workflow

```text
Branch = main
      │
      ▼
DEPLOY=true
      │
      ▼
Run
```

---

```text
Branch = main
      │
      ▼
DEPLOY=false
      │
      ▼
Skip
```

---

Production Example

```text
main

↓

Security Scan Passed

↓

Deployment Approved

↓

Deploy
```

---

# not Condition

Reverses another condition.

---

Syntax

```groovy
when {

    not {

        branch "main"

    }

}
```

---

Workflow

```text
Branch = main ?

      │

     Yes

      │

     not

      │

      ▼

Skip
```

---

Production Example

```text
Feature Branch

↓

Run Debug Stage
```

---

```text
Main Branch

↓

Skip Debug Stage
```

---

# Combining Conditions

Multiple conditions can be combined to create secure deployment pipelines.

---

Example

```groovy
stage("Deploy") {

    when {

        allOf {

            branch "main"

            expression {

                params.DEPLOY

            }

            environment name: "ENV",
                        value: "prod"

        }

    }

    steps {

        sh "./deploy.sh"

    }

}
```

---

Execution Flow

```text
Branch = main ?
        │
   ┌────┴────┐
   │         │
 No         Yes
 │           │
 ▼           ▼
Skip    DEPLOY=true ?
              │
         ┌────┴────┐
         │         │
        No        Yes
         │         │
         ▼         ▼
      Skip     ENV=prod ?
                   │
              ┌────┴────┐
              │         │
             No        Yes
              │         │
              ▼         ▼
           Skip       Deploy
```

---

# Production Pipeline Example

```text
Developer Push

↓

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

Trivy Scan

↓

Branch = main ?

↓

DEPLOY = true ?

↓

Deploy to EKS
```

---

# Best Practices

## Use Branch Protection

```text
Deploy Production

↓

Only From

↓

main
```

---

## Use Parameters

Avoid hardcoding values.

Use

```text
params.DEPLOY

params.ENV

params.VERSION
```

---

## Skip Unnecessary Stages

Example

```text
README Changed

↓

Skip Docker Build

Skip Trivy
```

---

## Keep Conditions Simple

Prefer

```text
branch

environment

anyOf

allOf
```

instead of long Groovy expressions.

---

## Test Both Paths

Verify

```text
Condition = True

↓

Stage Executes
```

and

```text
Condition = False

↓

Stage Skipped
```

---

# Troubleshooting

## Stage Always Skipped

Check

```text
Branch Name

Parameter Value

Environment Variable

when Condition
```

---

Useful Commands

```groovy
echo env.BRANCH_NAME

echo params.DEPLOY

sh "printenv"
```

---

## Stage Always Executes

Verify the condition is not always returning `true`.

---

## Wrong Branch

```groovy
echo env.BRANCH_NAME
```

---

## Parameter Not Working

```groovy
echo params.toString()
```

---

## changeset Not Matching

Verify the file pattern.

Example

```groovy
changeset "**/*.tf"
```

---

## Deployment Not Triggering

Verify

```text
Branch

↓

Parameter

↓

Environment

↓

Trigger
```

A failed condition skips the stage.

---

# Common Interview Questions

## 1. What is Conditional Execution in Jenkins?

### Short Answer

Conditional execution allows Jenkins to run a stage only when specified conditions are satisfied.

### Detailed Explanation

Instead of executing every stage for every build, Jenkins evaluates the `when` condition before running the stage. If the condition is true, the stage executes; otherwise, it is skipped.

### Production Example

Deploy the application only from the `main` branch.

### Follow-Up Questions

- Which directive is used for conditional execution?
- When is the `when` block evaluated?

---

## 2. What is the `when` Directive?

### Short Answer

The `when` directive controls whether a stage executes.

### Detailed Explanation

It is available in Declarative Pipelines and supports conditions such as branch, tag, environment, parameters, expressions, and triggers.

### Production Example

Run the deployment stage only for the production environment.

### Follow-Up Questions

- Can `when` be used inside steps?
- Is it supported in Scripted Pipeline?

---

## 3. What is the Difference Between `branch` and `expression`?

### Short Answer

`branch` checks Git branches, while `expression` evaluates custom Groovy logic.

### Detailed Explanation

Use `branch` for branch-based conditions and `expression` when custom logic involving parameters or environment variables is required.

### Production Example

```text
branch "main"

↓

Deploy
```

```text
expression { params.DEPLOY }

↓

Deploy
```

### Follow-Up Questions

- Which one is easier to maintain?
- Which one is preferred for branch validation?

---

## 4. What is the Difference Between `anyOf` and `allOf`?

### Short Answer

`anyOf` works like **OR**, while `allOf` works like **AND**.

### Detailed Explanation

`anyOf` executes when any one condition is true.

`allOf` executes only when every condition is true.

### Production Example

```text
anyOf

develop

OR

main

↓

Deploy
```

---

```text
allOf

main

AND

DEPLOY=true

↓

Deploy
```

### Follow-Up Questions

- Which one is commonly used for Production deployment?
- Can they be nested?

---

## 5. What is the Difference Between `tag` and `buildingTag`?

### Short Answer

`tag` checks a specific tag pattern, while `buildingTag` checks whether the build was triggered from any Git tag.

### Detailed Explanation

Use `tag` when releases follow naming conventions such as `v1.*`.

Use `buildingTag` when any tag should trigger the stage.

### Production Example

```text
v2.0.0

↓

Release Pipeline
```

### Follow-Up Questions

- Why are Git tags used for releases?
- Which condition is more flexible?

---

## 6. What is the Purpose of the `changeset` Condition?

### Short Answer

It executes a stage only when specified files are modified.

### Detailed Explanation

Jenkins checks the changed files and skips unrelated stages, reducing build time.

### Production Example

```text
Terraform Changed

↓

Run Checkov
```

---

```text
README Changed

↓

Skip Checkov
```

### Follow-Up Questions

- Which file patterns are supported?
- How does it improve pipeline performance?

---

## 7. Why Should Production Deployment Be Conditional?

### Short Answer

To prevent accidental deployments.

### Detailed Explanation

Production deployment should occur only after validating the branch, approvals, parameters, and security checks.

### Production Example

```text
main

↓

Security Scan

↓

Approval

↓

Deploy
```

### Follow-Up Questions

- Which conditions would you combine?
- Would you deploy from feature branches?

---

## 8. How Do You Skip a Stage in Jenkins?

### Short Answer

Use the `when` directive.

### Detailed Explanation

If the condition evaluates to false, Jenkins automatically skips the stage.

### Production Example

```text
DEPLOY=false

↓

Deploy Stage Skipped
```

### Follow-Up Questions

- Is the skipped stage considered a failure?
- Does the pipeline continue?

---

## 9. How Do You Debug a Stage That Is Always Skipped?

### Short Answer

Verify the condition and runtime values.

### Detailed Explanation

Check the current branch, parameters, environment variables, and trigger information.

### Useful Commands

```groovy
echo env.BRANCH_NAME

echo params.DEPLOY

sh "printenv"
```

### Follow-Up Questions

- How do you print environment variables?
- How do you verify parameter values?

---

## 10. How Does Conditional Execution Improve Pipeline Performance?

### Short Answer

It prevents unnecessary stages from executing.

### Detailed Explanation

Only relevant stages consume build agents, CPU, memory, and execution time.

### Production Example

```text
README Updated

↓

Skip

Docker Build

Trivy

Deployment
```

### Follow-Up Questions

- Which condition helps optimize builds?
- Does it reduce infrastructure cost?

---

## 11. Scenario: Deploy Only from Main Branch

### Answer

```groovy
when {

    branch "main"

}
```

### Production Example

```text
feature/login

↓

Skip Deploy
```

---

```text
main

↓

Deploy
```

---

## 12. Scenario: Deploy Only After Manual Approval

### Answer

```groovy
when {

    expression {

        params.DEPLOY

    }

}
```

### Production Example

```text
DEPLOY=false

↓

Skip
```

---

```text
DEPLOY=true

↓

Deploy
```

---

## 13. Scenario: Run Checkov Only for Terraform Changes

### Answer

```groovy
when {

    changeset "**/*.tf"

}
```

### Production Example

```text
Terraform Updated

↓

Run Checkov
```

---

```text
Java Updated

↓

Skip Checkov
```

---

## 14. Scenario: Release Only When a Git Tag is Created

### Answer

```groovy
when {

    tag "v*"

}
```

### Production Example

```text
git tag v2.1.0

↓

Release Pipeline
```

---

## 15. Scenario: Secure Production Deployment

### Answer

```groovy
when {

    allOf {

        branch "main"

        expression {

            params.DEPLOY

        }

        environment name: "ENV",
                    value: "prod"

    }

}
```

### Production Flow

```text
main

↓

DEPLOY=true

↓

ENV=prod

↓

Deploy
```

---

# Key Takeaways

```text
Conditional execution controls stage execution.

The 'when' directive is used in Declarative Pipelines.

branch checks Git branches.

expression evaluates custom Groovy logic.

environment validates environment variables.

changeset executes stages based on modified files.

tag and buildingTag support release pipelines.

anyOf works like OR.

allOf works like AND.

not reverses a condition.

Conditional execution reduces build time and infrastructure cost.

Production deployments should always use multiple conditions for safety.
```