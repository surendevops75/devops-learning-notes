# Jenkins Agent Administration

## Introduction

A Jenkins Controller is responsible for scheduling jobs, managing pipelines, storing configurations, and coordinating builds.

However, executing every build on the Controller creates several problems:

- Controller becomes overloaded
- Builds become slow
- Resource contention increases
- Security risks increase
- Single point of failure

To solve this, Jenkins executes builds on **Agents**.

An Agent is a machine that performs the actual build while the Controller manages the overall pipeline.

---

# Why Do We Need Agents?

Imagine a company with hundreds of developers.

```text
                GitHub

                   │

                   ▼

          Hundreds of Commits

                   │

                   ▼

            Jenkins Controller

                   │

         Executes Every Build

                   │

                   ▼

         CPU = 100%

         Memory = 95%

         Builds Waiting
```

Eventually,

```text
Pipeline Queue

↓

10 Builds Waiting

↓

20 Builds Waiting

↓

50 Builds Waiting

↓

Developers Waiting
```

Clearly, the Controller should **coordinate builds**, not execute them.

---

# Solution

Move build execution to Agents.

```text
               Jenkins Controller

                      │

      ┌───────────────┼────────────────┐

      ▼               ▼                ▼

 Linux Agent     Docker Agent     Windows Agent

      │               │                │

 Build Java     Build Containers   Build .NET

 Test           Scan Images        Run Tests

 Package        Push Images        Package
```

The Controller remains lightweight while Agents perform the heavy work.

---

# What is a Jenkins Agent?

A Jenkins Agent is a machine that connects to the Jenkins Controller and executes jobs.

The agent can be:

- Physical server
- Virtual Machine
- Docker Container
- Kubernetes Pod
- Cloud Instance

The Controller sends work to the Agent, and the Agent returns the results.

---

# Controller vs Agent

| Controller | Agent |
|------------|-------|
| Manages Jenkins | Executes Builds |
| Stores Jobs | Runs Pipeline Steps |
| Schedules Builds | Compiles Code |
| Manages Plugins | Runs Tests |
| Coordinates Agents | Creates Artifacts |

---

# Jenkins Distributed Architecture

```text
                  Developers

                       │

                       ▼

                    GitHub

                       │

                 Webhook Trigger

                       │

                       ▼

              Jenkins Controller

                       │

      ┌────────────────┼────────────────┐

      ▼                ▼                ▼

 Linux Agent      Windows Agent     Docker Agent

      │                │                │

 Java Build      .NET Build      Container Build

      │                │                │

      └──────────────┬─────────────────┘

                     ▼

                 Artifacts
```

---

# Build Execution Flow

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Controller Receives Event

↓

Pipeline Scheduled

↓

Agent Selected

↓

Build Executed

↓

Result Returned

↓

Pipeline Completed
```

---

# Production Example

A company has

- Java applications
- Python applications
- NodeJS applications
- .NET applications

Instead of installing every tool on one server,

multiple specialized agents are created.

```text
Linux Agent

↓

Java

Maven

Gradle

Docker

----------------------

Windows Agent

↓

Visual Studio

MSBuild

PowerShell

----------------------

Python Agent

↓

Python

Pip

Poetry

----------------------

NodeJS Agent

↓

Node

npm

Yarn
```

Each build runs on the appropriate agent.

---

# Benefits of Using Agents

- Faster builds
- Parallel execution
- Better scalability
- Better resource utilization
- Isolation between builds
- Easy maintenance
- Platform-specific builds
- Improved security

---

# Agent Communication

```text
Controller

↓

Assign Job

↓

Agent

↓

Execute Pipeline

↓

Send Console Logs

↓

Return Build Status
```

The Controller never performs the build itself (except when intentionally configured).

---

# Agent Lifecycle

```text
Create Agent

↓

Connect Agent

↓

Verify Connection

↓

Assign Labels

↓

Run Builds

↓

Monitor Health

↓

Maintenance

↓

Reconnect

↓

Remove Agent
```

---

# Types of Jenkins Agents

Jenkins supports several agent types.

```text
                 Jenkins Agents

                       │

    ┌──────────────────┼───────────────────┐

    ▼                  ▼                   ▼

 Static             Dynamic           Cloud

    │                  │                   │

 SSH              Docker          Kubernetes

 JNLP             Containers      EC2

 Windows          Ephemeral       Azure

 Linux                             GCP
```

---

# Static Agents

A static agent is a permanently running machine connected to Jenkins.

Example

```text
Controller

↓

Linux VM

↓

Always Online
```

Advantages

- Simple configuration
- Stable
- Easy to manage
- Predictable performance

Disadvantages

- Idle resources
- Higher infrastructure cost
- Manual scaling

---

# Dynamic Agents

Dynamic agents are created only when required.

```text
Build Requested

↓

Create Agent

↓

Execute Build

↓

Destroy Agent
```

Advantages

- Cost effective
- Auto-scaling
- Clean environment
- Faster provisioning in cloud environments

---

# Static vs Dynamic Agents

| Static Agents | Dynamic Agents |
|---------------|----------------|
| Always Running | Created On Demand |
| Higher Cost | Lower Cost |
| Manual Scaling | Automatic Scaling |
| Long-lived | Ephemeral |
| Best for Small Teams | Best for Cloud & Kubernetes |

---

# Dedicated vs Shared Agents

## Dedicated Agent

```text
Payments Team

↓

Dedicated Linux Agent
```

Only one team uses the machine.

Benefits

- Predictable performance
- Strong isolation
- Easier compliance

---

## Shared Agent

```text
Developer A

↓

Shared Agent

↑

Developer B

↑

Developer C
```

Benefits

- Better resource utilization
- Lower infrastructure cost

Challenges

- Resource contention
- Workspace cleanup
- Tool version conflicts

---

# Agent Labels

Labels help Jenkins decide where a pipeline should execute.

Example

```text
linux

windows

docker

python

java

terraform

eks
```

Pipeline

```groovy
agent {
    label 'linux'
}
```

The Controller automatically selects an available agent with the **linux** label.

---

# Production Label Example

```text
Controller

↓

Label = java

↓

Java Agent

↓

Compile Project

----------------------

Label = windows

↓

Windows Agent

↓

Build .NET Project

----------------------

Label = docker

↓

Docker Agent

↓

Build Image
```

---

# Why Labels Are Important

Without labels

```text
Controller

↓

Random Agent

↓

Build Failure
```

Example

A .NET application accidentally runs on a Linux-only agent.

With labels

```text
Pipeline

↓

windows

↓

Windows Agent

↓

Build Successful
```

---

# Best Practices

- Keep the Controller dedicated to orchestration.
- Do not run production builds on the Controller.
- Use labels for workload placement.
- Separate Linux and Windows workloads.
- Keep build environments consistent across agents.
- Monitor CPU, memory, and disk utilization.

---

# Key Points

- Jenkins Agents execute builds while the Controller manages the CI/CD workflow.
- Distributed builds improve scalability, performance, and security.
- Static agents are always available, while dynamic agents are created on demand.
- Labels ensure pipelines run on the correct build environment.
- Production Jenkins deployments rely heavily on dedicated and dynamic agents for efficient resource utilization.

---

# SSH Agents

## What are SSH Agents?

SSH Agents are Linux or Unix machines that connect to the Jenkins Controller using the **Secure Shell (SSH)** protocol.

The Jenkins Controller remotely launches the agent process over SSH.

---

# SSH Agent Architecture

```text
              Jenkins Controller

                     │

            SSH Connection (Port 22)

                     │

                     ▼

              Linux Build Agent

                     │

        Maven | Docker | Terraform

                     │

               Execute Pipeline
```

---

# SSH Connection Flow

```text
Controller

↓

SSH Authentication

↓

Launch Agent Process

↓

Receive Build

↓

Execute Pipeline

↓

Send Logs

↓

Build Completed
```

---

# Production Example

```text
Controller

↓

SSH

↓

Ubuntu VM

↓

Java Build

↓

Docker Build

↓

Push to ECR
```

SSH agents are the most commonly used static agents in enterprise Jenkins environments.

---

# SSH Agent Configuration

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Nodes

↓

New Node

↓

Permanent Agent

↓

Launch Agent via SSH
```

Required information

```text
Hostname

SSH Port

Credentials

Remote Root Directory

Labels
```

---

# Advantages

- Easy to configure
- Secure communication
- Stable
- Enterprise standard
- Supports Linux workloads

---

# JNLP Agents

## What are JNLP Agents?

JNLP (Java Network Launch Protocol) agents initiate the connection **from the agent to the controller**.

Unlike SSH agents, the controller does not log into the machine.

The agent connects itself.

---

# JNLP Architecture

```text
              Jenkins Controller

                     ▲

                     │

          Agent Initiates Connection

                     │

                     ▲

             JNLP Build Agent
```

---

# Why JNLP?

Useful when

- Controller cannot SSH
- Firewall restrictions exist
- Cloud environments
- Windows environments
- Agents behind NAT

---

# JNLP Connection Flow

```text
Agent Starts

↓

Connects to Controller

↓

Authentication

↓

Connection Established

↓

Receive Build

↓

Execute Build
```

---

# Production Example

```text
Private Network

↓

Windows Agent

↓

JNLP

↓

Jenkins Controller

↓

Run .NET Build
```

---

# SSH vs JNLP

| SSH | JNLP |
|------|------|
| Controller connects to Agent | Agent connects to Controller |
| Port 22 | Configurable |
| Common for Linux | Common for Windows & Restricted Networks |
| Simple | Useful behind Firewalls |

---

# Docker Agents

## What are Docker Agents?

Docker Agents execute builds inside Docker containers.

Instead of maintaining permanent build servers,

Jenkins starts a container,

runs the build,

then removes the container.

---

# Docker Agent Architecture

```text
Controller

↓

Docker Host

↓

Create Container

↓

Execute Build

↓

Destroy Container
```

---

# Docker Build Flow

```text
Pipeline Starts

↓

Pull Docker Image

↓

Start Container

↓

Run Build

↓

Run Tests

↓

Container Removed
```

---

# Declarative Pipeline Example

```groovy
pipeline {

    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
        }
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

# Production Example

```text
Developer Push

↓

Controller

↓

Docker Agent

↓

Maven Container

↓

Compile

↓

Package

↓

Container Deleted
```

Every build starts from a clean environment.

---

# Benefits

- Reproducible builds
- No dependency conflicts
- Clean workspace
- Easy version management
- Fast provisioning

---

# Kubernetes Agents

## What are Kubernetes Agents?

Instead of Docker containers on a VM,

Kubernetes creates **temporary Pods** that act as Jenkins agents.

Each pipeline receives a new Pod.

---

# Kubernetes Agent Architecture

```text
              Jenkins Controller

                      │

                      ▼

           Kubernetes API Server

                      │

                Create Pod

                      │

                      ▼

              Jenkins Agent Pod

                      │

            Execute Pipeline

                      │

                Pod Deleted
```

---

# Kubernetes Build Flow

```text
Pipeline Trigger

↓

Create Pod

↓

Container Starts

↓

Build

↓

Test

↓

Push Image

↓

Delete Pod
```

---

# Production Example

```text
GitHub

↓

Webhook

↓

Controller

↓

EKS Cluster

↓

Temporary Jenkins Pod

↓

Build Image

↓

Push to ECR

↓

Pod Removed
```

---

# Kubernetes Advantages

- Automatic scaling
- Fully isolated builds
- No idle infrastructure
- Cost efficient
- Excellent for cloud-native CI/CD

---

# Cloud Agents

Cloud providers can provision agents dynamically.

Examples

```text
AWS EC2

Azure VM

Google Compute Engine
```

---

# Cloud Agent Architecture

```text
Pipeline

↓

Controller

↓

Cloud Plugin

↓

Launch VM

↓

Run Build

↓

Terminate VM
```

---

# Enterprise Example

```text
Morning

↓

5 Developers

↓

5 Agents

---------------------

Afternoon

↓

150 Developers

↓

150 Agents

---------------------

Night

↓

No Builds

↓

0 Agents
```

Cloud resources scale automatically with demand.

---

# Agent Comparison

| Agent Type | Best Use Case |
|------------|---------------|
| SSH | Linux VMs |
| JNLP | Windows / Firewalled Networks |
| Docker | Containerized Builds |
| Kubernetes | Cloud-Native CI/CD |
| EC2 | Auto Scaling |
| Azure VM | Azure Pipelines |
| GCP VM | Google Cloud |

---

# Production Recommendation

```text
Controller

↓

Kubernetes Plugin

↓

Create Agent Pod

↓

Run Build

↓

Delete Pod
```

Modern enterprise Jenkins deployments typically favor Kubernetes agents because they provide isolation, elasticity, and efficient resource utilization.

---

# Best Practices

- Prefer ephemeral Docker or Kubernetes agents for CI workloads.
- Keep static agents only for specialized workloads or licensed software.
- Store tools inside container images instead of installing them manually.
- Regularly update base images.
- Avoid running multiple unrelated builds inside the same long-lived container.

---

# Key Points

- SSH agents are the standard choice for Linux-based static agents.
- JNLP agents are useful when the agent must initiate the connection.
- Docker agents provide clean, reproducible build environments.
- Kubernetes agents create ephemeral Pods for each build, making them ideal for cloud-native CI/CD.
- Cloud agents automatically scale infrastructure based on build demand.

---


# Executors

## What is an Executor?

An **Executor** is a worker thread on a Jenkins Controller or Agent that executes a build.

Think of an executor as a "build slot."

If an agent has:

```text
Executors = 1
```

Only one build can run at a time.

If an agent has:

```text
Executors = 4
```

Four builds can run simultaneously.

---

# Executor Architecture

```text
              Linux Agent

         CPU : 8 Cores

         Executors : 4

                │

        ┌───────┼────────┐

        ▼       ▼        ▼

     Build1  Build2   Build3

                  ▼

               Build4
```

---

# Build Queue

Suppose

```text
Executors = 2

Incoming Builds = 5
```

Execution

```text
Build 1 → Running

Build 2 → Running

Build 3 → Waiting

Build 4 → Waiting

Build 5 → Waiting
```

The remaining builds stay in the Jenkins queue until an executor becomes available.

---

# Choosing Executor Count

There is no fixed number.

It depends on

- CPU cores
- RAM
- Build type
- Docker usage
- Kubernetes resources

Example

```text
Agent

↓

8 CPU

↓

16 GB RAM

↓

4 Executors
```

---

# Production Recommendation

CPU-intensive builds

```text
Executors = Number of CPU Cores / 2
```

Lightweight builds

```text
Executors = CPU Cores
```

Heavy Docker builds

```text
Executors = 1 or 2
```

Always benchmark your workloads before increasing executor count.

---

# Node Management

A **Node** is any machine registered with Jenkins.

It may be

- Controller
- Linux VM
- Windows VM
- Docker Host
- Kubernetes Pod

Every agent is a node.

---

# Node Lifecycle

```text
Create Node

↓

Configure

↓

Connect

↓

Assign Labels

↓

Execute Builds

↓

Monitor Health

↓

Maintenance

↓

Disconnect

↓

Remove
```

---

# Creating a Node

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Nodes

↓

New Node
```

Provide

```text
Node Name

Remote Root Directory

Launch Method

Labels

Executors
```

---

# Node Configuration Example

```text
Node Name

↓

linux-agent-01

Executors

↓

4

Labels

↓

linux docker java terraform

Remote Directory

↓

/home/jenkins
```

---

# Offline Nodes

Sometimes an agent becomes unavailable.

```text
Controller

↓

linux-agent-01

↓

Offline
```

Possible reasons

- VM shutdown
- Network issue
- SSH failure
- Disk full
- Java process stopped

---

# Temporarily Marking an Agent Offline

Administrators often place nodes into maintenance mode.

```text
Agent

↓

Mark Offline

↓

No New Builds

↓

Existing Builds Finish

↓

Maintenance

↓

Bring Online
```

This prevents build interruptions during updates.

---

# Monitoring Agent Health

Regularly monitor

- CPU utilization
- Memory usage
- Disk space
- Java process
- Network latency
- Build queue
- Workspace size

Example

```text
Linux Agent

↓

CPU 35%

Memory 62%

Disk 70%

Healthy
```

---

# Agent Workspace Management

Every build creates files.

Over time

```text
Workspace

↓

2 GB

↓

10 GB

↓

50 GB

↓

Disk Full
```

Builds begin to fail because no free space remains.

---

# Workspace Cleanup

Production pipelines should clean workspaces regularly.

Declarative Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {
            steps {
                cleanWs()
                sh 'mvn clean package'
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

---

# Agent Maintenance

Regular maintenance includes

- OS patching
- Java updates
- Docker updates
- Tool upgrades
- Workspace cleanup
- Log cleanup
- Disk cleanup

---

# Maintenance Flow

```text
Mark Agent Offline

↓

Wait for Running Builds

↓

Install Updates

↓

Restart Services

↓

Verify Connection

↓

Bring Online
```

---

# Scaling Jenkins Agents

As the organization grows

```text
10 Developers

↓

2 Agents

--------------------

50 Developers

↓

8 Agents

--------------------

200 Developers

↓

Auto Scaling

↓

Kubernetes
```

---

# Horizontal Scaling

Instead of increasing one large server,

add more agents.

```text
Controller

↓

Agent 1

Agent 2

Agent 3

Agent 4

Agent 5
```

Advantages

- Better availability
- Better throughput
- Easier maintenance

---

# Vertical Scaling

Increase resources of one agent.

```text
Before

↓

4 CPU

8 GB RAM

↓

After

↓

16 CPU

64 GB RAM
```

Useful for resource-intensive builds.

---

# Production Scaling Example

Morning

```text
20 Builds

↓

5 Agents
```

Afternoon

```text
250 Builds

↓

Kubernetes Plugin

↓

Create 100 Pods
```

Night

```text
No Builds

↓

Delete Pods
```

Cloud-native Jenkins scales automatically with demand.

---

# Agent Security Best Practices

- Do not run builds on the Controller.
- Use dedicated service accounts.
- Restrict SSH access.
- Rotate SSH keys regularly.
- Patch operating systems frequently.
- Keep Java updated.
- Use ephemeral agents for untrusted workloads.
- Isolate production deployment agents.
- Encrypt communication between Controller and Agents.

---

# Common Troubleshooting

## Issue 1: Agent Offline

Possible Causes

- SSH failure
- Network issue
- Java process stopped
- Firewall
- Host unavailable

Resolution

```text
Verify Network

↓

Check SSH

↓

Restart Agent

↓

Reconnect
```

---

## Issue 2: Builds Waiting Forever

Possible Causes

- No available executors
- Label mismatch
- Agent offline

Resolution

```text
Check Queue

↓

Verify Labels

↓

Verify Executors

↓

Bring Agent Online
```

---

## Issue 3: Pipeline Cannot Find Agent

Example

```groovy
agent {
    label 'eks'
}
```

But no agent has the label **eks**.

Resolution

```text
Assign Label

↓

Reconnect Agent

↓

Retry Build
```

---

## Issue 4: Workspace Full

Symptoms

```text
No Space Left on Device
```

Resolution

```text
Clean Workspace

↓

Delete Old Artifacts

↓

Expand Disk

↓

Retry Build
```

---

## Issue 5: Slow Builds

Possible Causes

- Too many executors
- CPU contention
- Low memory
- Shared agent overload

Resolution

- Reduce executor count.
- Add additional agents.
- Move to Kubernetes or Docker agents.
- Monitor CPU and memory utilization.

---

# Production Interview Questions

## Question 1

### Production-Level Answer

What is a Jenkins Agent?

A Jenkins Agent is a machine that executes build tasks on behalf of the Jenkins Controller. The Controller schedules and coordinates builds, while Agents perform the actual work.

### Follow-Up Questions

- What is a node?
- Why shouldn't builds run on the Controller?
- What are ephemeral agents?

---

## Question 2

### Production-Level Answer

Explain the difference between static and dynamic agents.

Static agents are always running and permanently connected to Jenkins. Dynamic agents are provisioned on demand (for example, Docker containers or Kubernetes Pods) and removed after the build completes.

---

## Question 3

### Production-Level Answer

What are Jenkins Executors?

Executors are worker threads that determine how many builds an agent can execute concurrently. More executors allow greater parallelism but require sufficient CPU and memory resources.

---

## Question 4

### Production-Level Answer

When would you use SSH agents instead of Kubernetes agents?

SSH agents are suitable for stable, long-lived servers with persistent tools, while Kubernetes agents are ideal for cloud-native environments requiring elastic scaling and isolated build environments.

---

## Question 5

### Production-Level Answer

Why are labels important in Jenkins?

Labels allow pipelines to target agents with specific capabilities, ensuring workloads run on the correct operating system, tools, or environment.

---

## Question 6

### Production-Level Answer

How do you scale Jenkins for hundreds of developers?

Use a lightweight Controller with dynamic Kubernetes or cloud agents, auto-scaling, workload labels, distributed builds, and monitoring for agent health and queue length.

---

## Question 7

### Production-Level Answer

What causes builds to remain in the queue?

Common causes include unavailable executors, offline agents, label mismatches, or insufficient build resources.

---

## Question 8

### Production-Level Answer

How do you maintain Jenkins agents?

Regularly patch the operating system, update Java and build tools, clean workspaces, monitor resource utilization, rotate SSH keys, and replace unhealthy agents.

---

## Question 9

### Production-Level Answer

Why are Kubernetes agents preferred in modern CI/CD?

They provide ephemeral, isolated build environments, automatic scaling, efficient resource utilization, and eliminate configuration drift between builds.

---

## Question 10

### Production-Level Answer

What is the recommended architecture for enterprise Jenkins?

A lightweight Controller connected to multiple ephemeral Kubernetes or Docker agents, integrated with centralized authentication, RBAC, monitoring, and automated scaling.

---

# Key Takeaways

- Jenkins Agents execute builds while the Controller orchestrates the CI/CD workflow.
- Executors define how many concurrent builds an agent can process.
- Labels ensure jobs run on compatible agents.
- Dynamic agents (Docker/Kubernetes) provide better scalability, isolation, and cost efficiency than static agents.
- Monitor agent health, clean workspaces, and apply regular maintenance to keep build infrastructure reliable.
- Horizontal scaling with multiple agents is generally preferred over relying on a single powerful server.
