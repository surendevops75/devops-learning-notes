# Jenkins Enterprise Handbook

# Chapter 1 - Jenkins Fundamentals & Enterprise CI/CD Architecture

Modern software delivery requires applications to be

- Built Automatically
- Tested Automatically
- Packaged Automatically
- Scanned Automatically
- Deployed Automatically

Manual software delivery is

- Slow
- Error-Prone
- Difficult to Scale
- Difficult to Audit

Jenkins automates the complete Continuous Integration and Continuous Delivery (CI/CD) lifecycle.

Today, Jenkins remains one of the most widely used enterprise CI/CD platforms.

---

# What is Jenkins?

Jenkins is an

open-source

automation server

used to build,

test,

package,

and deploy applications.

It supports

- Continuous Integration (CI)
- Continuous Delivery (CD)
- Pipeline Automation
- Infrastructure Automation

---

# Why Jenkins?

Without Jenkins

```text
Developer

↓

Build

↓

Manual Testing

↓

Manual Deployment

↓

Production
```

Problems

- Human Errors
- Slow Releases
- Inconsistent Deployments
- Difficult Rollbacks

---

With Jenkins

```text
Git Commit

↓

Jenkins Pipeline

↓

Build

↓

Test

↓

Deploy
```

Everything becomes automated.

---

# Continuous Integration (CI)

Continuous Integration focuses on

```text
Code Commit

↓

Build

↓

Unit Tests

↓

Artifact
```

Developers integrate code frequently,

and every change is automatically validated.

---

# Continuous Delivery (CD)

Continuous Delivery focuses on

```text
Validated Artifact

↓

Approval

↓

Deployment

↓

Production
```

The application is always in a deployable state.

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Build

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS

↓

Monitoring
```

This represents a common enterprise DevOps pipeline.

---

# Jenkins Architecture

```text
Developer

↓

Git Repository

↓

Jenkins Controller

↓

Agent

↓

Pipeline

↓

Deployment
```

Jenkins coordinates the pipeline,

while agents execute the jobs.

---

# Jenkins Components

Jenkins consists of

- Controller
- Agent
- Job
- Pipeline
- Stage
- Step
- Plugins
- Credentials

These components work together to automate software delivery.

---

# Jenkins Controller

The Jenkins Controller

is responsible for

- Managing Jobs
- Scheduling Builds
- Managing Agents
- Storing Configuration
- Coordinating Pipelines

It is the central management server.

---

# Jenkins Agent

An Agent

executes

pipeline stages.

Architecture

```text
Jenkins Controller

↓

Agent

↓

Pipeline Execution
```

Multiple agents

can execute jobs in parallel.

---

# Jenkins Job

A Job

is an automation task.

Examples

- Build Application
- Run Tests
- Deploy Application
- Execute Terraform
- Build Docker Image

---

# Jenkins Pipeline

A Pipeline

defines

the complete CI/CD process.

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

Package

↓

Deploy
```

Pipelines are written as code,

making them version-controlled and repeatable.

---

# Pipeline as Code

Instead of configuring jobs manually,

define the pipeline in a

```text
Jenkinsfile
```

Benefits

- Version Control
- Repeatability
- Code Review
- Easier Maintenance

---

# Jenkinsfile Workflow

```text
Git Repository

↓

Jenkinsfile

↓

Jenkins Pipeline

↓

Build

↓

Deploy
```

The Jenkinsfile becomes the source of truth for CI/CD.

---

# Stages

A Pipeline

is divided into

Stages.

Example

```text
Checkout

↓

Build

↓

Test

↓

Deploy
```

Each stage groups related tasks.

---

# Steps

Each Stage

contains

multiple Steps.

Example

```text
Install Dependencies

↓

Compile Code

↓

Run Tests
```

Steps execute sequentially within a stage.

---

# Plugins

Jenkins supports

thousands of plugins.

Common plugins include

- Git Plugin
- Docker Plugin
- Kubernetes Plugin
- Pipeline Plugin
- SonarQube Plugin
- Credentials Plugin
- Blue Ocean

Plugins extend Jenkins functionality.

---

# Credentials

Sensitive information

should be stored

using Jenkins Credentials.

Examples

```text
AWS Credentials

GitHub Token

Docker Password

SSH Keys
```

Never hardcode credentials inside pipelines.

---

# Jenkins Workflow

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Pipeline

↓

Build

↓

Test

↓

Deploy
```

The process is fully automated.

---

# Enterprise Deployment Flow

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Docker Build

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS

↓

Monitoring
```

This pipeline supports modern cloud-native deployments.

---

# Banking Example

```text
Developer

↓

Payment Service

↓

GitHub Push

↓

Jenkins

↓

Build

↓

Security Scan

↓

Docker Image

↓

Amazon ECR

↓

Amazon EKS

↓

Customers
```

Every release is validated before reaching production.

---

# Jenkins in DevOps

Jenkins integrates with

- GitHub
- GitLab
- Docker
- Kubernetes
- Amazon EKS
- Amazon ECR
- Terraform
- SonarQube
- Trivy
- Nexus
- Artifactory

It acts as the automation engine for the DevOps lifecycle.

---

# Benefits

- Open Source
- Highly Extensible
- Pipeline as Code
- Large Plugin Ecosystem
- Supports Multiple Languages
- Enterprise Ready
- Scalable
- Platform Independent

---

# Best Practices

- Use Pipeline as Code.
- Store Jenkinsfiles in Git.
- Separate controller and agents.
- Use credentials management.
- Automate testing.
- Automate security scanning.
- Keep plugins updated.
- Use dedicated agents for workloads.

---

# Common Mistakes

- Running builds directly on the Jenkins Controller.
- Hardcoding credentials.
- Installing unnecessary plugins.
- Keeping manual freestyle jobs.
- Ignoring plugin updates.
- Giving Jenkins administrator permissions unnecessarily.
- Not backing up Jenkins configuration.

---

# Interview Questions

## Basic

- What is Jenkins?
- What is Continuous Integration?
- What is Continuous Delivery?
- What is a Jenkins Pipeline?
- What is a Jenkinsfile?

## Intermediate

- Jenkins Controller vs Agent.
- What are Jenkins Plugins?
- Explain Pipeline as Code.
- What are Jenkins Credentials?
- How does Jenkins integrate with Docker and Kubernetes?

## Advanced

- Design an enterprise Jenkins CI/CD platform integrating GitHub, SonarQube, Trivy, Docker, Amazon ECR, Terraform, Amazon EKS, and monitoring.
- Explain the complete Jenkins workflow from code commit to production deployment.
- A company wants to migrate from manual deployments to Jenkins-based CI/CD. Explain the architecture, pipeline design, agent strategy, security model, deployment workflow, and governance approach.

---

# Chapter 2 - Jenkins Installation, Architecture & Master-Agent (Controller-Agent) Setup

A Jenkins server can run simple jobs on a single machine.

However,

enterprise environments require

- Scalability
- High Availability
- Workload Isolation
- Better Resource Utilization

For this reason,

organizations use a **Controller-Agent Architecture**.

---

# Jenkins Architecture Overview

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins Controller

↓

Jenkins Agents

↓

Build

↓

Deploy
```

The controller manages the pipeline,

while agents execute the workload.

---

# Why Controller-Agent Architecture?

Without Agents

```text
Jenkins

↓

Build

↓

Test

↓

Docker

↓

Terraform

↓

Deploy
```

Problems

- High CPU Usage
- Memory Bottlenecks
- Slow Pipelines
- Single Point of Execution

---

With Agents

```text
Jenkins Controller

↓

Linux Agent

↓

Docker Build

────────────

Windows Agent

↓

.NET Build

────────────

Kubernetes Agent

↓

Container Deployment
```

Workloads are distributed.

---

# Jenkins Installation Options

Jenkins can be installed using

- Native Package
- WAR File
- Docker
- Kubernetes
- Cloud Marketplace

Enterprise deployments

commonly use

Docker

or

Kubernetes.

---

# Jenkins Controller Responsibilities

The Controller

manages

- Pipelines
- Job Scheduling
- Authentication
- Authorization
- Plugin Management
- Agent Communication
- Build History
- Credentials

The Controller

should not perform

heavy builds.

---

# Jenkins Agent Responsibilities

Agents execute

- Application Builds
- Unit Tests
- Docker Builds
- Terraform Commands
- Kubernetes Deployments
- Security Scans

Agents perform

the computational work.

---

# Enterprise Architecture

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins Controller

├── Linux Agent

├── Docker Agent

├── Terraform Agent

├── Kubernetes Agent

└── Windows Agent
```

Each workload

runs on

the appropriate agent.

---

# Controller-Agent Communication

```text
Controller

↓

Job Assignment

↓

Agent

↓

Execution

↓

Result

↓

Controller
```

The controller

collects

logs,

status,

and artifacts.

---

# Agent Connection Methods

Agents can connect using

- SSH
- JNLP (Inbound Agent)
- Kubernetes Plugin
- Cloud Plugins

Modern Kubernetes deployments

commonly use

ephemeral agents.

---

# Static Agents

Static agents

remain online

continuously.

Architecture

```text
Jenkins Controller

↓

Linux VM

↓

Agent
```

Suitable for

small environments.

---

# Dynamic Agents

Dynamic agents

are created

only when needed.

```text
Pipeline

↓

Create Agent

↓

Execute Job

↓

Terminate Agent
```

This improves

resource efficiency.

---

# Kubernetes Agents

Using the Jenkins Kubernetes Plugin,

agents are launched

as Pods.

Architecture

```text
Jenkins Controller

↓

Kubernetes Plugin

↓

Pod

↓

Pipeline Execution
```

Each pipeline

gets

a clean execution environment.

---

# Docker Agents

Agents can also run

inside Docker containers.

Workflow

```text
Pipeline

↓

Docker Container

↓

Build

↓

Destroy Container
```

This ensures

consistent build environments.

---

# Multi-Agent Pipeline

```text
Checkout

↓

Linux Agent

────────────

Docker Build

↓

Docker Agent

────────────

Terraform

↓

Terraform Agent

────────────

Deployment

↓

Kubernetes Agent
```

Each stage

uses

the most appropriate agent.

---

# Label-Based Scheduling

Agents

are assigned

labels.

Examples

```text
linux

docker

terraform

kubernetes

windows
```

Pipelines

select agents

using labels.

---

# Agent Workspace

Every agent

uses

a workspace

to store

- Source Code
- Build Files
- Temporary Files
- Logs

The workspace

is cleaned

between builds

to avoid conflicts.

---

# High Availability

Enterprise Jenkins

often separates

the Controller

from build execution.

```text
Jenkins Controller

↓

Multiple Agents

↓

Independent Execution
```

Failure of one agent

does not stop

the platform.

---

# Scaling Jenkins

As workload increases,

add more agents.

```text
Controller

↓

5 Agents

↓

20 Agents

↓

100 Agents
```

The controller

distributes jobs automatically.

---

# Enterprise Deployment Architecture

```text
GitHub

↓

Webhook

↓

Jenkins Controller

↓

Linux Agent

↓

SonarQube

↓

Docker Agent

↓

Amazon ECR

↓

Terraform Agent

↓

Amazon EKS
```

Different stages

use

specialized agents.

---

# Banking Example

```text
Developer

↓

Payment Repository

↓

Jenkins Controller

↓

Linux Agent

↓

Docker Agent

↓

Security Scan

↓

Terraform Agent

↓

Amazon EKS

↓

Production
```

Every workload

runs

on dedicated infrastructure.

---

# Jenkins Directory Structure

Typical installation

```text
JENKINS_HOME/

├── jobs/

├── plugins/

├── users/

├── credentials/

├── workspace/

└── logs/
```

`JENKINS_HOME`

contains

all Jenkins configuration.

---

# Security Considerations

Protect the Controller using

- Authentication
- Role-Based Access Control (RBAC)
- HTTPS
- Backups
- Least Privilege

Agents should have

only the permissions

required for their tasks.

---

# Enterprise Best Practices

- Keep the controller lightweight.
- Execute builds on agents.
- Use dedicated agents for Docker and Terraform.
- Prefer ephemeral Kubernetes agents for cloud-native pipelines.
- Label agents clearly.
- Clean workspaces regularly.
- Monitor agent utilization.
- Back up `JENKINS_HOME`.

---

# Common Mistakes

- Running builds on the controller.
- Using one agent for every workload.
- Leaving stale workspaces on agents.
- Giving agents administrator privileges.
- Not monitoring offline agents.
- Installing unnecessary tools on every agent.
- Ignoring controller backups.

---

# Interview Questions

## Basic

- What is the Jenkins Controller?
- What is a Jenkins Agent?
- Why do we use agents?
- What is a Jenkins workspace?
- What is `JENKINS_HOME`?

## Intermediate

- Controller vs Agent.
- Static Agent vs Dynamic Agent.
- SSH Agent vs Kubernetes Agent.
- Why use Docker agents?
- How does Jenkins assign jobs to agents?

## Advanced

- Design an enterprise Jenkins architecture using a controller, Linux agents, Docker agents, Terraform agents, and Kubernetes agents for deploying applications to Amazon EKS.
- Explain how controller-agent architecture improves scalability, security, and resource utilization in Jenkins.
- A company runs hundreds of CI/CD pipelines daily for Java, Python, and .NET applications. Explain how you would design the Jenkins controller, agent infrastructure, labeling strategy, Kubernetes-based ephemeral agents, security controls, workspace management, and scaling strategy to support enterprise workloads.

---

