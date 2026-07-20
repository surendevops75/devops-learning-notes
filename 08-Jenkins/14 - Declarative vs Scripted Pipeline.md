# Declarative vs Scripted Pipeline

## Introduction

Jenkins supports two pipeline syntaxes.

```text
Declarative Pipeline

Scripted Pipeline
```

---

Both automate CI/CD but differ in syntax, flexibility, and use cases.

---

# Why Do We Need Two Pipeline Types?

Simple pipelines can be written using Declarative syntax.

Complex workflows may require Scripted syntax.

---

Example

```text
Simple CI/CD

↓

Declarative Pipeline
```

---

```text
Complex Logic

↓

Scripted Pipeline
```

---

# Declarative Pipeline

Declarative Pipeline is a structured and easy-to-read pipeline syntax.

It follows predefined rules and is recommended for most CI/CD pipelines.

---

Characteristics

```text
Simple

Readable

Easy to Maintain

Built-in Validation
```

---

Basic Structure

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

}
```

---

Workflow

```text
pipeline
    │
    ▼
agent
    │
    ▼
stages
    │
    ▼
stage
    │
    ▼
steps
```

---

Production Example

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

Trivy

↓

Deploy

↓

Post Actions
```

---

# Scripted Pipeline

Scripted Pipeline is written using Groovy programming.

It provides complete programming flexibility.

---

Characteristics

```text
Flexible

Programmable

Supports Loops

Supports Conditions

Supports Functions
```

---

Basic Structure

```groovy
node {

    stage("Build") {

        sh "mvn clean package"

    }

}
```

---

Workflow

```text
node
 │
 ▼
stage
 │
 ▼
Groovy Logic
 │
 ▼
steps
```

---

Production Example

```text
Multiple Applications

↓

Loop Through Projects

↓

Build Each Project

↓

Deploy
```

---

# Declarative vs Scripted

| Feature | Declarative | Scripted |
|----------|-------------|-----------|
| Syntax | Simple | Groovy |
| Learning Curve | Easy | Moderate |
| Flexibility | Medium | High |
| Validation | Built-in | Limited |
| Readability | High | Medium |
| Best For | Standard CI/CD | Complex Workflows |

---

# Syntax Comparison

Declarative

```groovy
pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                sh "mvn clean"

            }

        }

    }

}
```

---

Scripted

```groovy
node {

    stage("Build") {

        sh "mvn clean"

    }

}
```

---

# When Should You Use Declarative?

Choose Declarative when

```text
Standard CI/CD

Team Collaboration

Easy Maintenance

Enterprise Pipelines
```

---

# When Should You Use Scripted?

Choose Scripted when

```text
Complex Logic

Loops

Dynamic Pipelines

Reusable Functions
```

---

# Loops and Conditions

One of the biggest differences between Declarative and Scripted Pipelines is handling program logic.

---

Declarative Pipeline

Supports basic conditions.

```text
when

environment

parameters

post
```

---

Scripted Pipeline

Supports full Groovy programming.

```text
if

else

for

while

functions

try-catch
```

---

## Declarative Example

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

## Scripted Example

```groovy
node {

    if(env.BRANCH_NAME == "main"){

        stage("Deploy"){

            sh "./deploy.sh"

        }

    }

}
```

---

Workflow

```text
Branch = main ?

      │

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Run      Skip
```

---

# Loops in Scripted Pipeline

Loops are not directly supported in Declarative Pipeline.

Scripted Pipeline can execute repetitive tasks easily.

---

Example

```groovy
node {

    def services = [
        "cart",
        "catalogue",
        "user"
    ]

    for(service in services){

        stage(service){

            echo "Building ${service}"

        }

    }

}
```

---

Execution

```text
cart

↓

catalogue

↓

user
```

---

Production Example

Microservices Repository

```text
cart

catalogue

user

payment

shipping
```

---

Instead of writing five build stages,

Scripted Pipeline loops through each service automatically.

---

# Error Handling

Declarative Pipeline

Provides built-in support using the `post` block.

---

Example

```groovy
post {

    failure {

        echo "Build Failed"

    }

}
```

---

Scripted Pipeline

Uses Groovy exception handling.

---

Example

```groovy
node {

    try{

        sh "mvn clean package"

    }

    catch(Exception e){

        echo "Build Failed"

    }

}
```

---

Workflow

```text
Build

↓

Exception ?

 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Catch   Continue
```

---

# Environment Variables

Both pipeline types support environment variables.

---

Declarative

```groovy
environment {

    APP_NAME = "cart"

}
```

---

Scripted

```groovy
node {

    env.APP_NAME = "cart"

}
```

---

# Parallel Execution

Both pipelines support parallel execution.

---

Example

```text
Build

↓

Parallel

├── Unit Test

├── Integration Test

└── Security Scan
```

---

Declarative

```groovy
parallel {

    stage("Unit Test") {

        steps {

            sh "./unit.sh"

        }

    }

    stage("Security Scan") {

        steps {

            sh "./scan.sh"

        }

    }

}
```

---

Scripted

```groovy
parallel(

UnitTest:{

    sh "./unit.sh"

},

Security:{

    sh "./scan.sh"

}

)
```

---

# Production Level Pipeline

## Scenario

A company deploys a Java microservice to AWS EKS.

Pipeline Flow

```text
Developer Push
      │
      ▼
Checkout
      │
      ▼
Build
      │
      ▼
Unit Test
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
Update Kubernetes Manifest
      │
      ▼
Deploy to AWS EKS
      │
      ▼
Post Actions
```

---

### Declarative Pipeline

Best for this scenario.

```text
Readable

Easy to Maintain

Standard Enterprise Pipeline
```

---

### Scripted Pipeline

Useful when

```text
Building Multiple Applications

Dynamic Stage Creation

Complex Deployment Logic

Custom Groovy Functions
```

---

Example

```text
Repository

↓

Detect Changed Services

↓

Loop Through Services

↓

Build Only Changed Services

↓

Deploy
```

---

Enterprise companies generally prefer:

```text
Declarative Pipeline

+

Small Scripted Blocks

↓

Maintainable CI/CD
```

---

# Real-World Recommendation

```text
Simple Pipeline

↓

Declarative
```

---

```text
Enterprise CI/CD

↓

Declarative Pipeline

+

Shared Libraries
```

---

```text
Highly Dynamic Workflow

↓

Scripted Pipeline
```

---

# Best Practices

## Use Declarative Pipeline by Default

Declarative Pipeline is easier to understand and maintain.

Recommended for most CI/CD pipelines.

---

Best Choice

```text
Build

↓

Test

↓

Security Scan

↓

Deploy
```

---

## Use Scripted Only When Required

Choose Scripted Pipeline only for complex automation.

Examples

```text
Loops

Dynamic Stages

Custom Functions

Complex Decision Making
```

---

## Avoid Mixing Too Much Scripted Code

Bad

```text
Declarative

↓

Large Scripted Blocks

↓

Hard to Maintain
```

---

Good

```text
Declarative Pipeline

↓

Small Scripted Logic

↓

Easy Maintenance
```

---

## Use Shared Libraries

Move reusable code into Shared Libraries.

Avoid copying the same Groovy code into multiple Jenkinsfiles.

---

Example

```text
Project A

↓

Shared Library

↑

Project B

↑

Project C
```

---

## Keep Pipelines Simple

Each stage should perform one task.

Example

```text
Checkout

↓

Build

↓

Test

↓

Scan

↓

Deploy
```

---

## Separate Business Logic

Avoid writing application logic inside Jenkinsfiles.

Jenkins should automate the workflow, not replace application code.

---

# Troubleshooting

## Pipeline Validation Error

Usually occurs in Declarative Pipeline.

Example

```text
Missing

agent

stages

steps
```

---

Solution

Validate the pipeline syntax before committing.

---

## Scripted Pipeline Compilation Error

Usually caused by invalid Groovy syntax.

Example

```text
Missing {

Missing }

Missing )
```

---

Solution

Review the Groovy syntax carefully.

---

## Dynamic Stage Not Created

Possible Causes

```text
Loop Error

Incorrect Variable

Stage Name Issue
```

---

Verify

```groovy
echo service
```

inside the loop.

---

## Environment Variable Not Found

Verify

```groovy
echo env.APP_NAME
```

Ensure the variable is defined before use.

---

## Parallel Stage Failed

Check individual branch logs.

Example

```text
Parallel

├── Build ✓

├── Unit Test ✗

└── Security Scan ✓
```

---

Debug the failed branch instead of the entire pipeline.

---

## Pipeline Difficult to Maintain

Common Cause

```text
Large Jenkinsfile

↓

Hundreds of Lines

↓

Complex Logic
```

---

Solution

```text
Shared Libraries

Reusable Functions

Small Stages
```

---

# Common Interview Questions

## 1. What is a Declarative Pipeline?

### Short Answer

Declarative Pipeline is a structured Jenkins pipeline syntax recommended for most CI/CD workflows.

### Detailed Explanation

It provides built-in validation, improved readability, and standardized syntax, making it easier to develop and maintain.

### Production Example

A Java application pipeline with Build, Test, SonarQube, Docker Build, Trivy Scan, and Deployment.

### Follow-Up Questions

- Why is Declarative preferred?
- What are its limitations?

---

## 2. What is a Scripted Pipeline?

### Short Answer

Scripted Pipeline is a Groovy-based pipeline that provides maximum flexibility.

### Detailed Explanation

It supports loops, functions, conditions, exception handling, and dynamic stage creation.

### Production Example

Building multiple microservices using a loop.

### Follow-Up Questions

- Which language does Scripted Pipeline use?
- When should it be used?

---

## 3. Difference Between Declarative and Scripted Pipeline?

### Short Answer

Declarative focuses on simplicity, while Scripted focuses on flexibility.

### Detailed Explanation

Declarative follows a predefined structure. Scripted provides complete programming control using Groovy.

### Production Example

```text
Standard CI/CD

↓

Declarative
```

---

```text
Dynamic Multi-Service Build

↓

Scripted
```

### Follow-Up Questions

- Which one is recommended by Jenkins?
- Can both be used together?

---

## 4. Which Pipeline is Better?

### Short Answer

For most projects, Declarative Pipeline.

### Detailed Explanation

It is easier to maintain, easier for teams to understand, and reduces scripting errors.

Scripted should be reserved for advanced use cases.

### Follow-Up Questions

- Why do enterprises prefer Declarative?
- When would Scripted be necessary?

---

## 5. Can Declarative and Scripted Pipelines Be Combined?

### Short Answer

Yes.

### Detailed Explanation

Declarative Pipelines can include small Scripted blocks using the `script` step.

### Production Example

```groovy
script {

    if(params.DEPLOY){

        echo "Deploy"

    }

}
```

### Follow-Up Questions

- Why is the `script` block used?
- Should large Groovy code be written inside it?

---

## 6. Why Do Enterprises Prefer Declarative Pipeline?

### Short Answer

Because it is standardized and easier to maintain.

### Detailed Explanation

Large DevOps teams benefit from a consistent structure that is easier to review and troubleshoot.

### Follow-Up Questions

- How does it improve collaboration?
- How does it reduce maintenance?

---

## 7. When Should You Use Scripted Pipeline?

### Short Answer

When advanced programming logic is required.

### Detailed Explanation

Examples include dynamic stage creation, loops, reusable functions, and complex deployment workflows.

### Production Example

Automatically build only changed microservices.

### Follow-Up Questions

- Can Declarative perform loops?
- Which pipeline supports custom functions?

---

## 8. What is the `script` Block?

### Short Answer

The `script` block allows Groovy code inside a Declarative Pipeline.

### Detailed Explanation

It is useful for implementing small pieces of custom logic without converting the entire pipeline to Scripted syntax.

### Production Example

```groovy
script {

    if(params.DEPLOY){

        sh "./deploy.sh"

    }

}
```

### Follow-Up Questions

- Is `script` mandatory?
- Can all Scripted features be used inside it?

---

## 9. Which Pipeline is Easier to Debug?

### Short Answer

Declarative Pipeline.

### Detailed Explanation

Its structured syntax makes validation errors easier to identify and fix compared to complex Groovy scripts.

### Follow-Up Questions

- Why is Groovy debugging more difficult?
- How do you validate a Declarative Pipeline?

---

## 10. How Do You Choose Between Declarative and Scripted?

### Short Answer

Choose based on the complexity of the pipeline.

### Detailed Explanation

Use Declarative for standard CI/CD pipelines and Scripted only when dynamic logic cannot be implemented cleanly.

### Production Example

```text
Standard Application

↓

Declarative
```

---

```text
Dynamic Multi-Service Platform

↓

Scripted
```

### Follow-Up Questions

- Which pipeline do you use in your project?
- Why?

---

## 11. Scenario: Build Only Changed Microservices

### Answer

Use a Scripted Pipeline with loops to detect changed services and create stages dynamically.

---

## 12. Scenario: Standard Enterprise CI/CD

### Answer

Use a Declarative Pipeline with Build, Test, SonarQube, OWASP Dependency Check, Docker Build, Trivy Scan, Deploy, and Post Actions.

---

# Key Takeaways

```text
Jenkins supports Declarative and Scripted Pipelines.

Declarative Pipeline is recommended for most CI/CD workflows.

Scripted Pipeline provides complete Groovy flexibility.

Declarative is easier to read and maintain.

Scripted is suitable for dynamic and complex workflows.

Both pipelines support environment variables and parallel execution.

Shared Libraries reduce duplicate code.

Large enterprises typically use Declarative Pipelines with small Scripted blocks where necessary.

Choose the pipeline type based on complexity, not personal preference.
```