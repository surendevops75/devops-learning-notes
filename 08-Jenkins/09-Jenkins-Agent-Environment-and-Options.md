# Jenkins Agent, Environment and Options

# Introduction

Every Jenkins Pipeline begins with a **Pre-Build phase** before executing the actual build stages. During this phase, Jenkins prepares the execution environment by deciding **where the pipeline should run**, **what configuration values are available**, and **what execution rules must be followed**.

In a Declarative Pipeline, the most common Pre-Build directives are:

- `agent`
- `environment`
- `options`

These directives are evaluated before the first stage starts executing and play a crucial role in ensuring that the pipeline runs correctly.

Example from the trainer's Jenkinsfile:

```groovy
pipeline {

    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE = "Jenkins"
    }

    options {
        timeout(time:10, unit:'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

    }

}
```

Although this configuration appears simple, it controls some of the most important aspects of pipeline execution in production environments.

---

# Why Do We Need a Pre-Build Section?

Imagine Jenkins starts executing a pipeline immediately after receiving a GitHub webhook.

Before executing even a single shell command, Jenkins must answer several questions.

- Which machine should execute this build?
- Does that machine have Maven installed?
- Does it have Docker installed?
- Which AWS Region should be used?
- Should multiple builds run simultaneously?
- How long should Jenkins wait before cancelling a stuck build?

Without answering these questions, Jenkins cannot safely execute the pipeline.

The Pre-Build section exists to prepare the execution environment before the actual build begins.

---

# Pre-Build Workflow

Every Declarative Pipeline follows the same initialization sequence.

```text
GitHub Push

        │
        ▼

Webhook Trigger

        │
        ▼

Read Jenkinsfile

        │
        ▼

Select Agent

        │
        ▼

Load Environment Variables

        │
        ▼

Apply Pipeline Options

        │
        ▼

Start Pipeline Stages
```

Notice that none of the build stages execute until all pre-build activities are completed successfully.

---

# Understanding the Agent

## What is an Agent?

An **Agent** is the machine where Jenkins executes the pipeline.

The Jenkins Controller coordinates builds, but it does not necessarily execute them.

Instead, it delegates the work to one of the available agents.

Think of the Controller as a project manager.

The manager assigns work.

The employees actually perform the work.

Similarly,

- Controller schedules the build.
- Agent executes the build.

---

# Trainer Example

```groovy
agent {
    node {
        label 'AGENT-1'
    }
}
```

This tells Jenkins:

> "Execute this pipeline only on the node whose label is **AGENT-1**."

---

# Why Can't Jenkins Execute Everything on the Controller?

Suppose your organization has

- 50 developers
- 120 Jenkins pipelines
- Hundreds of builds every day

If every build runs on the Controller,

- CPU becomes overloaded
- Memory becomes exhausted
- Disk usage increases rapidly
- Jenkins UI becomes slow
- Controller may even crash

Instead, Jenkins distributes the workload across multiple build machines.

```text
                Jenkins Controller

          Build Scheduling Only

                    │
     ┌──────────────┼──────────────┐
     │              │              │
     ▼              ▼              ▼

 Java Agent     Docker Agent   Windows Agent

     │              │              │

 Executes       Executes      Executes
 Java Builds    Docker Jobs   .NET Builds
```

This architecture provides better performance, scalability, and fault tolerance.

---

# Responsibilities of the Controller

The Jenkins Controller is responsible for:

- Scheduling builds
- Reading Jenkinsfiles
- Managing plugins
- Managing credentials
- User authentication
- Build queue management
- Assigning agents
- Displaying build history

The Controller **should not** be used for heavy build workloads in production.

---

# Responsibilities of an Agent

An Agent performs the actual work.

Typical responsibilities include:

- Checking out source code
- Compiling applications
- Running unit tests
- Building Docker images
- Running SonarQube scans
- Running Trivy scans
- Executing Terraform
- Deploying applications

The Controller delegates these tasks to the appropriate agent.

---

# Pipeline Execution Flow

The following diagram shows how Jenkins uses an Agent.

```text
Developer Pushes Code

        │
        ▼

GitHub Repository

        │
        ▼

Webhook

        │
        ▼

Jenkins Controller

        │
 Reads Jenkinsfile

        │
 Finds Label

        ▼

AGENT-1

        │

 Executes

 Build

 Test

 Scan

 Package

 Deploy
```

---

# Agent Labels

Every Jenkins Agent can have one or more labels.

Labels help Jenkins decide where a pipeline should run.

Example:

| Agent | Labels |
|--------|--------|
| Agent-1 | java maven linux |
| Agent-2 | docker linux |
| Agent-3 | terraform aws |
| Agent-4 | windows dotnet |

A pipeline can request a specific capability instead of a specific machine.

Example:

```groovy
agent {
    label 'docker'
}
```

Jenkins automatically selects an available agent with the **docker** label.

This makes pipelines portable and easier to maintain.

---

# Types of Agents

## 1. Controller

The Controller itself executes the build.

Suitable only for:

- Learning
- Small projects
- Personal environments

Not recommended for production.

---

## 2. Static Agent

A permanently configured machine.

Examples:

- Physical Server
- Virtual Machine
- EC2 Instance

Advantages

- Stable
- Predictable
- Easy to configure

Disadvantages

- Resource wastage
- Manual maintenance
- Fixed capacity

---

## 3. Dynamic Agent

Created only when a build starts.

Examples:

- Docker Containers
- Kubernetes Pods
- Cloud Instances

Advantages

- Automatic scaling
- Better resource utilization
- Faster cleanup
- Cost efficient

Modern enterprises prefer dynamic agents over static agents.

---

# Agent Syntax

## Any Available Agent

```groovy
pipeline {

    agent any

}
```

Jenkins executes the pipeline on any available agent.

This is commonly used in development environments.

---

## Specific Agent

```groovy
pipeline {

    agent {

        label 'docker'

    }

}
```

Only agents with the **docker** label can execute this pipeline.

---

## Agent None

```groovy
pipeline {

    agent none

}
```

No global agent is assigned.

Each stage chooses its own agent.

Example:

```groovy
pipeline {

    agent none

    stages {

        stage("Build") {

            agent {

                label 'java'

            }

            steps {

                sh 'mvn clean package'

            }

        }

        stage("Docker") {

            agent {

                label 'docker'

            }

            steps {

                sh 'docker build -t app .'

            }

        }

    }

}
```

This approach improves resource utilization because each stage runs on the most suitable machine.

---

# Production Example

Consider a microservices platform with different technology stacks.

```text
Inventory Service

↓

Java

↓

Java Agent
```

```text
Frontend

↓

React

↓

Node.js Agent
```

```text
Infrastructure

↓

Terraform

↓

Terraform Agent
```

Instead of installing every tool on every machine, Jenkins schedules each pipeline on an agent that already has the required software installed.

This simplifies maintenance, reduces build failures caused by missing dependencies, and allows teams to optimize agents for specific workloads.

---

# Best Practices (Agent)

- Never run production builds on the Jenkins Controller.
- Use labels instead of hardcoding machine names.
- Create dedicated agents for Java, Docker, Terraform, and Kubernetes workloads.
- Keep agent images lightweight and updated.
- Prefer dynamic agents (Docker or Kubernetes) for scalable environments.
- Avoid installing unnecessary tools on every agent.
- Monitor agent CPU, memory, and disk usage.
- Use ephemeral agents for security and automatic cleanup.
- Restrict agent access using appropriate permissions.
- Regularly update agent software and plugins.

---

# Understanding Environment Variables

## What is the Environment Block?

Environment variables are key-value pairs that store configuration values used throughout the pipeline.

Instead of hardcoding the same values multiple times, define them once inside the `environment` block and reuse them anywhere in the pipeline.

The trainer's Jenkinsfile defines the following environment variable:

```groovy
environment {
    COURSE = "Jenkins"
}
```

This creates an environment variable named `COURSE` with the value `Jenkins`.

Whenever the pipeline executes, this variable becomes available to all stages.

---

# Why Do We Need Environment Variables?

Imagine a pipeline that builds and deploys an application.

Without environment variables, the same values might be repeated multiple times.

Example:

```groovy
sh '''
docker build -t catalogue .
docker tag catalogue 123456789.dkr.ecr.ap-south-1.amazonaws.com/catalogue
docker push 123456789.dkr.ecr.ap-south-1.amazonaws.com/catalogue
'''
```

Now imagine the application name changes from **catalogue** to **product-service**.

Every occurrence must be updated manually.

This increases maintenance effort and introduces the possibility of mistakes.

Using environment variables solves this problem.

```groovy
environment {

    IMAGE_NAME = "catalogue"

    ECR_REPOSITORY = "123456789.dkr.ecr.ap-south-1.amazonaws.com"

}
```

Now the pipeline becomes:

```groovy
sh '''
docker build -t $IMAGE_NAME .

docker tag $IMAGE_NAME $ECR_REPOSITORY/$IMAGE_NAME

docker push $ECR_REPOSITORY/$IMAGE_NAME
'''
```

Only the variable value needs to change.

---

# Pipeline Flow with Environment Variables

```text
Pipeline Starts

        │
        ▼

Load Environment Variables

        │
        ▼

Variables Become Available

        │
        ▼

Stages Execute

        │
        ▼

Shell Commands Read Variables
```

Environment variables are loaded before the build stages begin.

---

# Accessing Environment Variables

## Inside Shell Commands

Example:

```groovy
environment {

    COURSE = "Jenkins"

}

steps {

    sh '''

        echo $COURSE

    '''

}
```

Output

```text
Jenkins
```

---

## Inside Groovy

Environment variables can also be accessed using the `env` object.

Example:

```groovy
script {

    echo env.COURSE

}
```

Output

```text
Jenkins
```

---

## Trainer Example

The trainer uses the variable inside the shell script.

```groovy
script {

    sh """

        echo $COURSE

    """

}
```

Since the command executes inside the shell, `$COURSE` is automatically expanded.

---

# Multiple Environment Variables

A pipeline can define multiple variables.

```groovy
environment {

    APP_NAME = "catalogue"

    AWS_REGION = "ap-south-1"

    ENVIRONMENT = "dev"

    TEAM = "Platform"

}
```

All stages can access these values.

---

# Stage-Level Environment Variables

Environment variables can also be defined inside a stage.

Example

```groovy
pipeline {

    agent any

    stages {

        stage("Build") {

            environment {

                BUILD_TOOL = "Maven"

            }

            steps {

                sh '''

                    echo $BUILD_TOOL

                '''

            }

        }

    }

}
```

Stage-level variables are available only within that stage.

Once the stage completes, they are no longer accessible.

---

# Global vs Stage-Level Environment

| Global Environment | Stage Environment |
|--------------------|-------------------|
| Available throughout the pipeline | Available only within one stage |
| Defined at pipeline level | Defined inside a stage |
| Suitable for common configuration | Suitable for stage-specific values |

---

# Production Example

A microservices application consists of multiple services.

```text
User Service

Catalog Service

Order Service

Payment Service
```

Instead of hardcoding values inside every command, the pipeline defines them centrally.

```groovy
environment {

    AWS_REGION = "ap-south-1"

    REGISTRY = "123456789.dkr.ecr.ap-south-1.amazonaws.com"

    IMAGE_NAME = "catalogue"

}
```

Now every stage can use the same variables.

Build

```bash
docker build -t $IMAGE_NAME .
```

Tag

```bash
docker tag $IMAGE_NAME $REGISTRY/$IMAGE_NAME
```

Push

```bash
docker push $REGISTRY/$IMAGE_NAME
```

This makes the pipeline cleaner, reusable, and easier to maintain.

---

# Best Practices for Environment Variables

- Define reusable values only once.
- Use meaningful variable names.
- Use uppercase naming conventions.
- Avoid duplicate variables.
- Keep application configuration centralized.
- Separate environment-specific values.
- Use stage-level variables only when required.
- Keep variables descriptive and easy to understand.
- Store secrets using Jenkins Credentials instead of plain environment variables.
- Document important variables used by the pipeline.

---

# Understanding Pipeline Options

## What is the Options Block?

The `options` block controls how Jenkins executes the pipeline.

Unlike build stages, options do not perform application-related tasks.

Instead, they define execution rules that Jenkins must follow during the pipeline run.

Trainer Example:

```groovy
options {

    timeout(time:10, unit:'MINUTES')

    disableConcurrentBuilds()

}
```

These options are processed before the pipeline stages begin.

---

# Why Are Pipeline Options Needed?

Imagine a build that gets stuck while downloading Maven dependencies.

Without any timeout, the build may continue running for several hours.

This wastes:

- Jenkins executors
- CPU
- Memory
- Build queue capacity

Similarly, imagine two developers trigger deployments at exactly the same time.

Without proper controls:

```text
Developer A

        │

Deploy Starts

        │

Developer B

        │

Deploy Starts

        │

Both Deploy Together

        │

Production Conflict
```

Pipeline options help prevent these situations.

---

# timeout()

The `timeout()` option limits how long a pipeline is allowed to run.

Trainer Example:

```groovy
options {

    timeout(time:10, unit:'MINUTES')

}
```

If the pipeline exceeds ten minutes, Jenkins automatically aborts it.

---

# Pipeline Flow

```text
Pipeline Starts

        │

Timer Starts

        │

Pipeline Completes?

      /     \

    Yes      No

    │         │

Success   Time Exceeded

              │

Abort Pipeline
```

---

# Production Example

Suppose a Docker build hangs because the Docker daemon stops responding.

Without a timeout:

- Executor remains occupied.
- Future builds remain queued.
- Resources are wasted.

With `timeout()`, Jenkins terminates the build automatically after the configured duration.

---

# disableConcurrentBuilds()

This option prevents multiple executions of the same pipeline at the same time.

Trainer Example:

```groovy
options {

    disableConcurrentBuilds()

}
```

Only one build can execute.

Additional builds wait in the queue.

---

# Without disableConcurrentBuilds()

```text
Commit A

        │

Pipeline Starts

        │

Commit B

        │

Second Pipeline Starts

        │

Both Deploy

        │

Deployment Conflict
```

---

# With disableConcurrentBuilds()

```text
Commit A

        │

Pipeline Starts

        │

Commit B

        │

Wait in Queue

        │

First Pipeline Finishes

        │

Second Pipeline Starts
```

This ensures deployments happen sequentially.

---

# Real Production Scenario

A company deploys applications directly to a production Kubernetes cluster.

If two deployment pipelines execute simultaneously:

- Pods may restart unexpectedly.
- Helm releases may conflict.
- Terraform state may become inconsistent.
- Database migrations may overlap.

Using `disableConcurrentBuilds()` ensures that only one deployment pipeline runs at any given time, reducing the risk of production issues.

---

# Additional Useful Pipeline Options

Although your trainer covered two options, Jenkins provides several others that are commonly used in production.

| Option | Purpose |
|---------|---------|
| `timeout()` | Stops long-running builds |
| `disableConcurrentBuilds()` | Prevents simultaneous builds |
| `retry()` | Retries failed stages |
| `timestamps()` | Adds timestamps to console logs |
| `buildDiscarder()` | Deletes old build history |
| `skipDefaultCheckout()` | Skips automatic SCM checkout |

These will be covered in more detail in later chapters as they become relevant.

---

# Complete Production Jenkinsfile

The following Jenkinsfile combines the concepts covered in this chapter.

```groovy
pipeline {

    agent {
        node {
            label 'docker-agent'
        }
    }

    environment {

        APP_NAME = "catalogue"

        AWS_REGION = "ap-south-1"

        DOCKER_REGISTRY = "123456789.dkr.ecr.ap-south-1.amazonaws.com"

    }

    options {

        timeout(time:20, unit:'MINUTES')

        disableConcurrentBuilds()

    }

    stages {

        stage("Build") {

            steps {

                sh '''

                    echo "Building $APP_NAME"

                    mvn clean package

                '''

            }

        }

        stage("Docker Build") {

            steps {

                sh '''

                    docker build -t $APP_NAME .

                '''

            }

        }

    }

    post {

        always {

            cleanWs()

        }

    }

}
```

This Jenkinsfile demonstrates how the **Agent**, **Environment**, and **Options** directives work together before and during pipeline execution.

---

# Best Practices

Follow these best practices when designing enterprise Jenkins pipelines.

### Agent

- Never run production builds on the Jenkins Controller.
- Use labels instead of machine names.
- Create dedicated agents for Java, Docker, Terraform, and Kubernetes.
- Use ephemeral agents whenever possible.
- Keep agents lightweight.
- Install only required software on each agent.
- Monitor CPU, memory, and disk usage.
- Regularly update build tools.

### Environment

- Use meaningful variable names.
- Define reusable values globally.
- Use stage-level variables only when required.
- Avoid hardcoding AWS Regions and repository URLs.
- Keep naming conventions consistent.
- Store secrets using Jenkins Credentials instead of plain environment variables.
- Document important variables.

### Options

- Always configure a timeout.
- Prevent concurrent deployments.
- Keep build history under control using `buildDiscarder()`.
- Add timestamps for easier troubleshooting.
- Retry only transient failures.
- Skip unnecessary source checkouts when applicable.

---

# Common Mistakes

### Running Builds on the Controller

Many beginners execute every build on the Jenkins Controller.

This works for small environments but creates performance problems in production.

Always use dedicated build agents.

---

### Hardcoding Values

Bad Example

```groovy
docker tag catalogue 123456789.dkr.ecr.ap-south-1.amazonaws.com/catalogue
```

Good Example

```groovy
environment {

    IMAGE_NAME="catalogue"

    REGISTRY="123456789.dkr.ecr.ap-south-1.amazonaws.com"

}
```

---

### No Timeout

Without timeout configuration, a hung build can occupy an executor indefinitely.

Always configure reasonable execution limits.

---

### Concurrent Deployments

Running multiple deployment pipelines simultaneously can result in:

- Failed deployments
- Helm conflicts
- Terraform state corruption
- Application downtime

Use:

```groovy
disableConcurrentBuilds()
```

---

### Storing Passwords in Environment Variables

Avoid this:

```groovy
environment {

    PASSWORD="admin123"

}
```

Instead, use Jenkins Credentials.

---

# Troubleshooting Guide

## Pipeline Waiting Forever

Possible Causes

- No available agent
- Incorrect agent label
- All executors busy

Solution

- Verify agent status.
- Check labels.
- Increase executor capacity.

---

## Environment Variable Not Found

Possible Causes

- Typographical error
- Stage-level scope
- Incorrect variable reference

Solution

- Verify variable names.
- Check variable scope.
- Use `echo` or `printenv` to confirm values.

---

## Pipeline Times Out

Possible Causes

- Slow dependency downloads
- Docker daemon issues
- Network latency
- Long-running tests

Solution

- Review console logs.
- Optimize build steps.
- Increase timeout only if justified.

---

## Concurrent Build Issues

Possible Causes

- Missing `disableConcurrentBuilds()`
- Multiple webhook triggers

Solution

- Enable concurrent build protection.
- Verify webhook configuration.

---

# Common Interview Questions

## 1. What is an Agent in Jenkins?

### Short Answer

An Agent is the machine that executes the pipeline.

### Detailed Explanation

The Jenkins Controller schedules builds, while Agents perform the actual work such as compiling applications, running tests, building Docker images, and deploying applications.

### Production Example

A Java application runs on a Linux agent with Maven installed, while Docker image creation runs on a Docker-enabled agent.

### Follow-up Questions

- What is `agent any`?
- What is `agent none`?
- Why shouldn't builds run on the Controller?

---

## 2. What is the purpose of Labels?

### Short Answer

Labels allow Jenkins to select the appropriate agent.

### Detailed Explanation

Instead of selecting a specific machine, pipelines request capabilities such as `java`, `docker`, or `terraform`. Jenkins schedules the build on a matching agent.

### Production Example

A Docker pipeline runs only on agents that have Docker installed.

---

## 3. Why should production builds avoid the Controller?

### Short Answer

To improve scalability, stability, and security.

### Detailed Explanation

Running builds on the Controller consumes CPU, memory, and disk resources, affecting Jenkins responsiveness.

### Production Example

An organization with hundreds of daily builds distributes workloads across multiple agents.

---

## 4. What are Environment Variables?

### Short Answer

Environment variables store reusable configuration values for a pipeline.

### Detailed Explanation

They eliminate duplication and make pipelines easier to maintain by centralizing configuration.

### Production Example

Store the Docker registry URL once and reuse it across build, tag, and push steps.

---

## 5. Difference between Global and Stage-Level Environment Variables?

### Short Answer

Global variables are available throughout the pipeline, while stage-level variables exist only within a specific stage.

---

## 6. Why is `timeout()` important?

### Short Answer

It prevents builds from running indefinitely.

### Detailed Explanation

Timeouts free Jenkins executors when builds hang due to network, infrastructure, or tool failures.

---

## 7. What does `disableConcurrentBuilds()` do?

### Short Answer

It allows only one execution of a pipeline at a time.

### Production Example

Multiple production deployments are queued instead of executing simultaneously.

---

## 8. Can multiple agents exist in Jenkins?

### Short Answer

Yes.

### Detailed Explanation

Enterprise Jenkins environments commonly have dozens or even hundreds of agents dedicated to different workloads.

---

## 9. Why should secrets not be stored in Environment Variables?

### Short Answer

Because they can be exposed in logs or source code.

### Production Example

Use Jenkins Credentials and inject them securely during pipeline execution.

---

## 10. What happens if Jenkins cannot find the requested Agent?

### Short Answer

The pipeline waits in the queue until a matching agent becomes available.

---

# Production Scenarios

## Scenario 1

A Java pipeline accidentally starts on a Windows agent.

### Solution

Use agent labels to target Linux Java agents.

---

## Scenario 2

A Docker build fails because Docker is not installed.

### Solution

Create a dedicated Docker agent and assign the appropriate label.

---

## Scenario 3

A production deployment is triggered twice within a minute.

### Solution

Enable `disableConcurrentBuilds()` to serialize deployments.

---

## Scenario 4

A Maven download hangs for thirty minutes.

### Solution

Configure `timeout()` to abort the build automatically.

---

## Scenario 5

The AWS Region changes for all deployments.

### Solution

Update the global environment variable instead of modifying every stage.

---

## Scenario 6

Different applications require different build tools.

### Solution

Assign labels such as `java`, `node`, `docker`, and `terraform` to dedicated agents.

---

# Production Case Study

## Case Study: Scaling Jenkins for a Microservices Platform

A company hosts **80 microservices** on AWS.

Each microservice has its own Jenkins Pipeline.

Daily workload:

- 200+ builds
- Docker image creation
- SonarQube analysis
- Trivy image scanning
- Terraform infrastructure updates
- Kubernetes deployments

### Challenges

- Build queue delays
- Controller overload
- Resource contention
- Simultaneous production deployments

### Solution

- Dedicated Java agents
- Dedicated Docker agents
- Dedicated Terraform agents
- Kubernetes-based dynamic agents
- Global environment variables for common configuration
- `timeout()` for long-running jobs
- `disableConcurrentBuilds()` for deployment pipelines

### Result

- Faster pipeline execution
- Better resource utilization
- Improved reliability
- Reduced deployment conflicts
- Easier pipeline maintenance

---

# Revision Notes

Before moving to the next chapter, ensure you can answer the following:

- What is the purpose of an Agent?
- What is the difference between the Controller and an Agent?
- Why are labels used?
- When should you use `agent any` and `agent none`?
- What are Environment Variables?
- Difference between Global and Stage-level variables?
- Why should secrets use Jenkins Credentials?
- What does `timeout()` do?
- Why use `disableConcurrentBuilds()`?
- Why is the Pre-Build phase important?

---

# Key Takeaways

- The Pre-Build phase prepares the pipeline before any stage executes.
- The **Agent** determines where the pipeline runs.
- Labels allow Jenkins to select the appropriate execution machine.
- Environment variables centralize reusable configuration values.
- Global variables are available throughout the pipeline, while stage-level variables are scoped to a single stage.
- The `options` block controls pipeline execution behavior.
- `timeout()` prevents stuck builds from consuming resources indefinitely.
- `disableConcurrentBuilds()` avoids simultaneous execution of the same pipeline, reducing deployment conflicts.
- Production environments should use dedicated agents rather than the Jenkins Controller for build execution.
- Proper use of Agents, Environment Variables, and Options results in pipelines that are scalable, maintainable, and production-ready.