# Jenkins Controller and Agents

# Introduction

As organizations grow, a single Jenkins server cannot efficiently handle all CI/CD workloads.

Modern CI/CD pipelines may involve:

- Multiple repositories
- Different programming languages
- Docker builds
- Kubernetes deployments
- Terraform provisioning
- Parallel testing

To distribute the workload, Jenkins uses a distributed architecture consisting of:

- Controller
- Agents

Earlier versions of Jenkins referred to these as:

```text
Master

↓

Agent (Node)
```

Modern Jenkins documentation uses:

```text
Controller

↓

Agent
```

Both terms are still commonly used in the industry.

---

# Why Can't One Jenkins Server Do Everything?

Your trainer mentioned:

> "Jenkins master can't handle all the load."

This is one of the biggest reasons for using agents.

Imagine a company with:

```text
500 Developers
```

Each developer pushes code multiple times a day.

If every pipeline runs on one server:

```text
Git Clone

↓

Maven Build

↓

Docker Build

↓

Unit Tests

↓

Terraform

↓

Kubernetes Deployment
```

Eventually,

- CPU becomes overloaded
- Memory is exhausted
- Builds become slow
- Queues grow longer

A single machine becomes a bottleneck.

---

# Another Problem

Your trainer also explained:

A single server cannot provide every required platform.

Example

```text
Linux

Windows

macOS

Android

iOS

Chrome

Firefox

Edge
```

Testing every environment on one machine is impossible.

Instead,

Different agents are created for different purposes.

---

# Jenkins Distributed Architecture

```text
                    +----------------------+
                    |   Jenkins Controller |
                    +----------+-----------+
                               |
         ---------------------------------------------
         |                   |                      |
         |                   |                      |
         v                   v                      v

   Linux Agent        Windows Agent          Mac Agent

      Build             .NET Build          iOS Build

      Docker             IIS                Xcode

      Terraform          PowerShell         Swift

      Kubernetes
```

The Controller coordinates all build activities.

The Agents perform the actual work.

---

# What is Jenkins Controller?

The Controller is the central server responsible for managing Jenkins.

Responsibilities:

- Scheduling jobs
- Managing users
- Installing plugins
- Coordinating agents
- Maintaining build history
- Managing credentials
- Providing Web UI

The Controller **should not perform heavy builds** in production.

---

# What is an Agent?

An Agent is a machine connected to Jenkins that executes build jobs.

Agents can be:

- Physical Servers
- Virtual Machines
- EC2 Instances
- Kubernetes Pods
- Docker Containers

Their responsibility is to execute pipeline stages assigned by the Controller.

---

# Controller Workflow

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Controller Receives Event

↓

Controller Selects Agent

↓

Agent Executes Pipeline

↓

Results Sent Back

↓

Controller Displays Build Status
```

---

# Types of Agents

## Linux Agent

Commonly used for

```text
Java

NodeJS

Python

Docker

Terraform

Kubernetes
```

---

## Windows Agent

Used for

```text
.NET

Visual Studio

PowerShell

Windows Applications
```

---

## macOS Agent

Used for

```text
Xcode

Swift

iOS Applications
```

---

# Labels

Agents are identified using Labels.

Example

```text
linux

windows

docker

terraform

kubernetes
```

Pipeline Example

```groovy
pipeline {

    agent {
        label 'linux'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

    }

}
```

Jenkins automatically selects an available Linux agent.

---

# Why Use Multiple Agents?

Example

```text
Controller

↓

Linux Agent

↓

Java Build

↓

Docker Image
```

Another

```text
Controller

↓

Windows Agent

↓

.NET Build
```

Another

```text
Controller

↓

Mac Agent

↓

iOS Build
```

Every workload runs on the appropriate platform.

---

# Parallel Builds

Multiple agents allow builds to run simultaneously.

Example

```text
Controller

↓

----------------------------

↓

Linux Agent

↓

Unit Tests

----------------------------

↓

Windows Agent

↓

Integration Tests

----------------------------

↓

Mac Agent

↓

UI Tests
```

This significantly reduces pipeline execution time.

---

# Real Production Example

A company has:

```text
200 Developers
```

Infrastructure

```text
1 Jenkins Controller

↓

5 Linux Agents

↓

2 Windows Agents

↓

2 Kubernetes Agents
```

Workflow

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Controller

↓

Available Agent

↓

Build

↓

Test

↓

Docker Build

↓

Trivy Scan

↓

Push Image To Amazon ECR

↓

Update GitOps Repository

↓

ArgoCD

↓

Amazon EKS
```

---

# Jenkins Pipeline Example

Run on Any Agent

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                sh 'mvn clean package'

            }

        }

    }

}
```

---

Run on Linux Agent

```groovy
pipeline {

    agent {

        label 'linux'

    }

    stages {

        stage('Build') {

            steps {

                sh 'mvn clean package'

            }

        }

    }

}
```

---

Run on Docker Agent

```groovy
pipeline {

    agent {

        label 'docker'

    }

    stages {

        stage('Docker Build') {

            steps {

                sh 'docker build -t app:latest .'

            }

        }

    }

}
```

---

# Advantages of Distributed Builds

- Better Performance
- Faster Builds
- Platform Independence
- Better Resource Utilization
- Parallel Execution
- High Scalability
- Easier Maintenance

---

# Best Practices

- Keep the Controller lightweight.
- Run builds on agents.
- Assign meaningful labels.
- Use dedicated agents for specialized workloads.
- Monitor agent health.
- Remove offline agents.
- Scale agents as build demand increases.

---

# Common Interview Questions

## What is the difference between a Controller and an Agent?

### Short Answer

The Controller manages Jenkins, while Agents execute build jobs.

### Detailed Explanation

The Controller schedules jobs, manages plugins, credentials, users, and build history. Agents perform the actual build, test, scan, and deployment tasks assigned by the Controller.

### Production Example

A Jenkins Controller receives a GitHub webhook and assigns the build to a Linux Agent, which compiles the application, runs tests, and builds the Docker image.

### Follow-up Questions

- Can the Controller execute builds?
- Why are Agents needed?
- How are Agents connected to Jenkins?

---

## Why do organizations use multiple Agents?

### Short Answer

To distribute workload, improve scalability, and support different operating systems and build environments.

### Detailed Explanation

A single server cannot efficiently build every application type. Multiple agents allow parallel builds and support platform-specific requirements such as Linux, Windows, and macOS.

### Production Example

Java applications run on Linux Agents, while .NET applications run on Windows Agents. This allows teams to use the most suitable environment for each project.

### Follow-up Questions

- What are Labels?
- Can multiple jobs run simultaneously?
- How does Jenkins choose an Agent?

---

## What are Agent Labels?

### Short Answer

Labels are identifiers used to select specific Jenkins Agents for executing pipeline stages.

### Detailed Explanation

Labels help route workloads to agents with the required tools or operating systems, such as Docker, Terraform, Kubernetes, Linux, or Windows.

### Production Example

A Docker build pipeline specifies `agent { label 'docker' }`, ensuring the build runs on an agent with Docker installed.

### Follow-up Questions

- Can an Agent have multiple labels?
- What happens if no matching Agent is available?
- Can labels be changed?

---

# Production Scenarios

## Scenario 1

The Linux Agent is offline.

### Solution

The Controller marks the build as waiting until a suitable Linux Agent becomes available or another matching agent is online.

---

## Scenario 2

Docker is installed only on one Agent.

### Solution

Assign a `docker` label to that Agent and configure Docker pipelines to use `agent { label 'docker' }`.

---

## Scenario 3

The organization needs to run builds faster.

### Solution

Add additional Agents and enable parallel execution to distribute workloads.

---

# Key Takeaways

```text
A Jenkins Controller manages the CI/CD environment.

Agents perform the actual build and deployment tasks.

Distributed builds improve scalability and performance.

Labels help route jobs to the correct Agent.

Modern Jenkins uses the terms Controller and Agent, replacing the older Master and Slave terminology.

Enterprise environments typically use multiple Linux, Windows, or Kubernetes Agents for different workloads.
```