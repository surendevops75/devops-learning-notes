# Jenkins Stages and Steps

# Introduction

A Jenkins Pipeline is not just a collection of commands executed one after another. It is a structured workflow that divides the software delivery process into logical phases called **Stages**.

Each stage represents a major milestone in the CI/CD pipeline, such as building the application, running tests, performing security scans, packaging artifacts, or deploying to an environment.

Inside every stage are **Steps**, which are the individual tasks Jenkins executes. These steps may compile code, run shell commands, execute scripts, archive artifacts, or deploy applications.

Think of a Jenkins Pipeline as constructing a building:

- **Pipeline** → The entire construction project
- **Stages** → Foundation, Walls, Roof, Painting
- **Steps** → Individual tasks inside each phase

This structured approach makes pipelines easier to understand, monitor, troubleshoot, and maintain.

---

# Why Do We Need Stages?

Imagine executing every build command in one long script.

```text
Compile Code

Run Unit Tests

Run SonarQube

Run Dependency Check

Build Docker Image

Push Docker Image

Deploy Application

Run Smoke Tests

Notify Team
```

Although Jenkins can execute this, there are several problems.

## Problems Without Stages

- Difficult to identify where a failure occurred.
- Poor visualization in Jenkins UI.
- Hard to restart from a specific point.
- Difficult for teams to collaborate.
- Logs become large and difficult to analyze.
- No logical separation of CI/CD activities.

Now compare it with a stage-based pipeline.

```text
Pipeline

│

├── Build

├── Test

├── Security Scan

├── Package

├── Deploy

└── Post Actions
```

Each stage has a clear responsibility.

When a stage fails, Jenkins immediately highlights it, making troubleshooting much easier.

---

# Real Production Example

Consider a microservices application deployed to Amazon EKS.

Instead of executing hundreds of commands together, the pipeline is divided into stages.

```text
Developer

    │

    ▼

Git Push

    │

    ▼

Build Stage
Compile Application

    │

    ▼

Test Stage
Run Unit Tests

    │

    ▼

Security Stage

SonarQube

OWASP Dependency Check

Trivy

Checkov

    │

    ▼

Package Stage

Docker Build

Docker Push

    │

    ▼

Deploy Stage

Helm Upgrade

    │

    ▼

Smoke Test

    │

    ▼

Success
```

Each stage represents a checkpoint in the software delivery process.

---

# Pipeline Execution Flow

A Declarative Pipeline executes stages sequentially unless parallel execution is explicitly configured.

```text
Pipeline Starts

        │

        ▼

Initialize Agent

        │

        ▼

Load Environment Variables

        │

        ▼

Execute Stage 1

        │

        ▼

Execute Stage 2

        │

        ▼

Execute Stage 3

        │

        ▼

Execute Stage 4

        │

        ▼

Run Post Actions

        │

        ▼

Pipeline Ends
```

By default, Jenkins waits for one stage to finish before starting the next.

---

# What is a Stage?

A **Stage** is a logical phase of a Jenkins Pipeline.

It groups related tasks together so they can be executed, monitored, and reported as a single unit.

Syntax

```groovy
stage('Build') {

    steps {

        echo "Building Application"

    }

}
```

Every stage has:

- A unique name
- A set of steps
- Independent execution status
- Separate logs
- Separate visualization in Jenkins

---

# Stage Lifecycle

Every stage follows the same lifecycle.

```text
Stage Starts

      │

      ▼

Allocate Resources

      │

      ▼

Execute Steps

      │

      ▼

Step Success?

     │      │

    Yes     No

     │       │

     ▼       ▼

Next Stage  Pipeline Stops
```

If any step inside a stage fails, Jenkins marks the entire stage as failed.

Unless configured otherwise, the pipeline stops immediately.

---

# Understanding Steps

A **Step** is the smallest executable unit inside a Jenkins Pipeline.

A stage can contain one step or multiple steps.

Example

```groovy
stage('Build') {

    steps {

        echo "Compiling"

        sh "mvn clean package"

    }

}
```

In this example:

- `echo` is one step.
- `sh` is another step.

Both belong to the **Build** stage.

---

# Stage vs Step

| Stage | Step |
|--------|------|
| Logical phase | Individual task |
| Groups related activities | Performs one action |
| Appears in Jenkins UI | Executes inside a stage |
| Can contain many steps | Cannot contain stages |
| Example: Build | Example: `sh`, `echo`, `git` |

Think of it like this:

```text
Pipeline

│

├── Stage

│     ├── Step

│     ├── Step

│     └── Step

├── Stage

│     ├── Step

│     └── Step

└── Stage

      ├── Step

      └── Step
```

---

# Understanding the Trainer Pipeline

The trainer's Jenkinsfile contains three stages.

```groovy
stages {

    stage('Build') {

        steps {

            script {

                sh """

                echo "Building"

                """

            }

        }

    }

    stage('Test') {

        steps {

            script {

                sh """

                echo "Testing"

                """

            }

        }

    }

    stage('Deploy') {

        steps {

            script {

                sh """

                echo "Deploying"

                """

            }

        }

    }

}
```

Execution Flow

```text
Pipeline

│

├── Build

│      │

│      ▼

│   Execute Shell Commands

│

├── Test

│      │

│      ▼

│   Execute Shell Commands

│

└── Deploy

       │

       ▼

   Execute Shell Commands
```

Each stage performs one responsibility.

This separation makes the pipeline clean and maintainable.

---

# Why Are Stages Important in Production?

Imagine a deployment pipeline for 100 microservices.

Without stages, the console output would contain thousands of lines of logs with no clear indication of which phase failed.

With stages, Jenkins immediately highlights the failing phase.

Example

```text
✔ Build

✔ Unit Test

✔ SonarQube

✔ Dependency Check

✖ Docker Build

Skipped Deploy
```

A DevOps engineer can immediately focus on the Docker build stage instead of searching through the entire log.

---

# Naming Stages

Stage names should clearly describe their purpose.

Good Examples

```text
Build

Unit Test

Security Scan

Docker Build

Deploy to Dev

Deploy to Production
```

Poor Examples

```text
Stage1

Test123

ABC

StepOne
```

Meaningful names improve readability and make Jenkins dashboards easier to understand.

---

# Common Production Stages

Although every organization has its own pipeline, most CI/CD pipelines follow a similar structure.

```text
Checkout

↓

Build

↓

Unit Test

↓

Code Quality

↓

Security Scan

↓

Package

↓

Docker Build

↓

Push Image

↓

Deploy

↓

Smoke Test

↓

Post Actions
```

This workflow ensures that only validated and secure applications progress through the delivery pipeline.