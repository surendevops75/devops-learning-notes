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

# Chapter 3 - Jenkins Pipeline Fundamentals (Declarative & Scripted Pipelines)

Jenkins originally used

Freestyle Jobs

for automation.

As applications became larger,

Freestyle Jobs became difficult to

- Maintain
- Version
- Scale
- Review

Jenkins introduced **Pipeline as Code**, allowing CI/CD pipelines to be written as code using a **Jenkinsfile**.

This is now the enterprise standard.

---

# What is a Jenkins Pipeline?

A Jenkins Pipeline is

a series of automated stages

that define the complete CI/CD workflow.

Typical stages include

- Checkout
- Build
- Test
- Security Scan
- Package
- Deploy

Everything is defined in code.

---

# Pipeline Workflow

```text
GitHub Push

↓

Jenkinsfile

↓

Checkout

↓

Build

↓

Test

↓

Docker Build

↓

Deploy

↓

Production
```

---

# Pipeline as Code

Instead of manually configuring jobs,

store pipeline logic inside

```text
Jenkinsfile
```

Benefits

- Version Control
- Code Review
- Easy Rollback
- Reusable Pipelines
- Consistent Builds

---

# Jenkinsfile

A Jenkinsfile

is a text file

stored inside

the Git repository.

Repository

```text
project/

├── src/

├── Dockerfile

├── pom.xml

└── Jenkinsfile
```

Every code change

includes

pipeline changes.

---

# Jenkins Pipeline Architecture

```text
Developer

↓

GitHub

↓

Jenkinsfile

↓

Jenkins Controller

↓

Agent

↓

Pipeline Execution
```

---

# Types of Pipelines

Jenkins supports

- Declarative Pipeline
- Scripted Pipeline

Declarative Pipelines

are recommended

for most enterprise projects.

---

# Declarative Pipeline

Declarative Pipelines

follow

a predefined structure.

Architecture

```text
Pipeline

↓

Stages

↓

Steps
```

Advantages

- Easy to Read
- Easy to Maintain
- Built-in Validation
- Standardized

---

# Scripted Pipeline

Scripted Pipelines

are written

using

Groovy programming.

Advantages

- Flexible
- Powerful
- Complex Logic

Disadvantages

- Harder to Read
- More Difficult to Maintain

---

# Declarative vs Scripted

| Declarative | Scripted |
|--------------|-----------|
| Structured | Flexible |
| Easier to Learn | Advanced |
| Enterprise Standard | Complex Automation |
| Less Code | More Code |

---

# Pipeline Structure

A Declarative Pipeline

typically contains

```text
Pipeline

↓

Agent

↓

Environment

↓

Stages

↓

Post Actions
```

---

# Stage

A Stage

groups

related activities.

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

Stages improve

pipeline readability.

---

# Step

Each Stage

contains

one or more

Steps.

Example

```text
Run Maven

↓

Run Tests

↓

Build Docker Image
```

---

# Agent Directive

Every Pipeline

requires

an Agent.

Architecture

```text
Pipeline

↓

Agent

↓

Execution
```

The Agent

determines

where the pipeline runs.

---

# Environment Directive

Environment variables

can be defined

for the entire pipeline.

Examples

```text
AWS Region

Application Name

Docker Repository

Cluster Name
```

Avoid hardcoding values.

---

# Options Directive

Pipeline options

control execution behavior.

Examples

- Timeout
- Retry
- Build Discarder
- Disable Concurrent Builds

These improve pipeline reliability.

---

# Parameters

Pipelines

can accept

runtime parameters.

Examples

```text
Environment

Application Version

Region

Deployment Type
```

One pipeline

supports multiple deployments.

---

# Triggers

Pipelines

can start automatically.

Examples

```text
Git Push

Pull Request

Webhook

Schedule

Manual Trigger
```

---

# Sequential Execution

Stages execute

in order.

```text
Checkout

↓

Build

↓

Test

↓

Deploy
```

If one stage fails,

the pipeline stops.

---

# Parallel Execution

Independent stages

can run simultaneously.

```text
Build

↓

Unit Test

↓

Security Scan
```

Parallel execution

reduces pipeline duration.

---

# Conditional Stages

Some stages

execute only

when conditions

are met.

Example

```text
Main Branch

↓

Deploy

────────────

Feature Branch

↓

Skip Deployment
```

---

# Post Actions

Post actions

execute

after the pipeline finishes.

Examples

- Cleanup
- Notifications
- Archive Logs
- Publish Reports

These run

whether the pipeline

succeeds or fails.

---

# Pipeline Flow

```text
Checkout

↓

Compile

↓

Unit Test

↓

SonarQube

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS
```

---

# Multi-Stage Pipeline

```text
Stage 1

↓

Checkout

────────────

Stage 2

↓

Build

────────────

Stage 3

↓

Testing

────────────

Stage 4

↓

Deployment
```

---

# Enterprise Pipeline

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout

↓

Build

↓

SonarQube

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS
```

---

# Banking Example

```text
Payment API

↓

Checkout

↓

Compile

↓

Testing

↓

Security Scan

↓

Docker Image

↓

Amazon ECR

↓

Production
```

Every stage

must complete successfully

before deployment.

---

# Pipeline Benefits

- Pipeline as Code
- Version Controlled
- Repeatable
- Automated
- Easier Review
- Easier Maintenance
- Faster Delivery
- Enterprise Standard

---

# Enterprise Best Practices

- Store Jenkinsfiles in Git.
- Prefer Declarative Pipelines.
- Keep stages focused.
- Use environment variables.
- Parameterize deployments.
- Execute independent stages in parallel.
- Add post-build cleanup.
- Version pipeline changes.

---

# Common Mistakes

- Using Freestyle Jobs instead of Pipelines.
- Creating very large pipeline files.
- Hardcoding environment values.
- Running everything sequentially.
- Mixing deployment logic with build logic.
- Ignoring cleanup steps.
- Not version-controlling Jenkinsfiles.

---

# Interview Questions

## Basic

- What is a Jenkins Pipeline?
- What is Pipeline as Code?
- What is a Jenkinsfile?
- Declarative vs Scripted Pipeline.
- What is a Stage?

## Intermediate

- Stage vs Step.
- What is the Agent directive?
- What are environment variables in Jenkins?
- Why use parallel stages?
- What are post actions?

## Advanced

- Design an enterprise Jenkins pipeline for building, testing, scanning, containerizing, and deploying applications to Amazon EKS using Docker, SonarQube, Trivy, Terraform, and Amazon ECR.
- Explain why Declarative Pipelines are preferred over Freestyle Jobs and how Pipeline as Code improves maintainability, scalability, and governance.
- A company is migrating hundreds of Freestyle Jobs to Jenkins Pipelines. Explain how you would redesign the CI/CD architecture using Jenkinsfiles, reusable stages, environment variables, parallel execution, conditional deployments, and enterprise best practices.

---

# Chapter 4 - Jenkinsfile Syntax (Complete Enterprise Guide)

A Jenkins Pipeline is stored inside a

```text
Jenkinsfile
```

Understanding Jenkinsfile syntax is essential for building

- Reusable Pipelines
- Secure Pipelines
- Enterprise CI/CD Platforms

A Jenkinsfile defines

- Where the pipeline runs
- What tasks execute
- When they execute
- How failures are handled

---

# Jenkinsfile Execution Flow

```text
GitHub Push

↓

Webhook

↓

Jenkins

↓

Jenkinsfile

↓

Pipeline

↓

Stages

↓

Deployment
```

Everything is controlled

by the Jenkinsfile.

---

# Jenkinsfile Structure

A Declarative Jenkinsfile contains

```text
Pipeline

↓

Agent

↓

Tools

↓

Environment

↓

Options

↓

Parameters

↓

Triggers

↓

Stages

↓

Post Actions
```

This is the recommended enterprise structure.

---

# Pipeline Block

The pipeline block

is the root

of every Jenkinsfile.

Architecture

```text
Pipeline

├── Agent

├── Environment

├── Stages

└── Post
```

Everything resides inside

the pipeline block.

---

# Agent Directive

The agent directive

defines

where

the pipeline executes.

Architecture

```text
Pipeline

↓

Agent

↓

Linux VM

↓

Docker

↓

Kubernetes Pod
```

Every pipeline

requires an execution environment.

---

# Agent Types

Common agent types

```text
Any

Specific Label

Docker

Kubernetes

None
```

Choose

based on

the workload.

---

# Tools Directive

The tools directive

automatically configures

build tools.

Examples

```text
JDK

Maven

Gradle

Node.js
```

Ensures

consistent tool versions

across builds.

---

# Environment Directive

Environment variables

store

configuration values.

Examples

```text
Application Name

AWS Region

Docker Repository

Cluster Name

Environment
```

Avoid

hardcoded values.

---

# Options Directive

Options control

pipeline behavior.

Common options

- Timeout
- Retry
- Disable Concurrent Builds
- Build Retention
- Timestamp Logs

These improve

pipeline reliability.

---

# Parameters Directive

Parameters

allow users

to provide values

during pipeline execution.

Examples

```text
Environment

Application Version

Deployment Type

Region
```

One pipeline

supports

multiple deployments.

---

# Trigger Directive

Triggers

start pipelines automatically.

Examples

```text
Webhook

Poll SCM

Cron Schedule

Manual Build
```

Pipelines become

event-driven.

---

# Stages Block

Stages

divide

the pipeline

into logical phases.

Typical stages

```text
Checkout

↓

Build

↓

Test

↓

Security Scan

↓

Docker Build

↓

Deploy
```

---

# Stage Block

Each stage

represents

one logical activity.

Example

```text
Build Stage

↓

Compile

↓

Package
```

Stages

improve readability

and troubleshooting.

---

# Steps Block

Steps

contain

individual commands.

Example

```text
Checkout Source

↓

Compile Code

↓

Execute Tests
```

Every stage

contains

one or more steps.

---

# Parallel Stages

Independent stages

can execute

simultaneously.

Example

```text
Unit Test

↓

Security Scan

↓

Lint Check
```

Benefits

- Faster Builds
- Better Resource Utilization

---

# Conditional Execution

Stages

can execute

only

when conditions

are met.

Example

```text
Feature Branch

↓

Skip Production

────────────

Main Branch

↓

Deploy
```

---

# Input Step

Sensitive deployments

may require

manual approval.

Workflow

```text
Build

↓

Approval

↓

Production Deployment
```

Common

for production releases.

---

# Post Block

The post block

runs

after

pipeline execution.

Examples

```text
Success

Failure

Always

Cleanup
```

Useful for

notifications

and cleanup.

---

# Success Actions

Executed only

when

the pipeline succeeds.

Typical tasks

- Send Success Notification
- Publish Reports
- Tag Release

---

# Failure Actions

Executed only

when

the pipeline fails.

Typical tasks

- Send Alert
- Archive Logs
- Create Incident

---

# Always Actions

Executed

regardless

of pipeline status.

Typical tasks

- Cleanup Workspace
- Upload Logs
- Generate Reports

---

# Workspace

During execution

the agent creates

a workspace.

```text
Workspace

↓

Source Code

↓

Build

↓

Artifacts
```

The workspace

is cleaned

after execution.

---

# Build Artifacts

Pipelines

generate artifacts.

Examples

```text
JAR File

Docker Metadata

Terraform Plan

Test Reports
```

Artifacts

can be archived

after the build.

---

# Credentials

Sensitive data

should be retrieved

from

Jenkins Credentials.

Examples

```text
AWS Keys

GitHub Token

Docker Password

SSH Keys
```

Never

hardcode credentials

inside the Jenkinsfile.

---

# Environment Strategy

One Jenkinsfile

can deploy

to multiple environments.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Environment variables

control deployment behavior.

---

# Enterprise Pipeline Flow

```text
Checkout

↓

Compile

↓

Unit Test

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
```

Every stage

is clearly separated.

---

# Banking Example

```text
Developer

↓

Payment Service

↓

Checkout

↓

Build

↓

Security Scan

↓

Approval

↓

Production
```

Production deployment

requires

manual approval.

---

# Enterprise Jenkinsfile Architecture

```text
Pipeline

├── Agent

├── Environment

├── Parameters

├── Stages

│   ├── Checkout

│   ├── Build

│   ├── Test

│   ├── Security

│   ├── Docker

│   └── Deploy

└── Post
```

This structure

is recommended

for enterprise CI/CD.

---

# Enterprise Best Practices

- Use Declarative Pipelines.
- Keep stages small and focused.
- Store configuration in environment variables.
- Use parameters for flexibility.
- Archive important artifacts.
- Use Jenkins Credentials for secrets.
- Add cleanup in post actions.
- Enable timestamps and timeouts.

---

# Common Mistakes

- Hardcoding credentials.
- Creating one large stage.
- Skipping cleanup steps.
- Ignoring pipeline timeouts.
- Running production deployments without approval.
- Using duplicated pipeline logic.
- Storing environment-specific values directly in the Jenkinsfile.

---

# Interview Questions

## Basic

- What is a Jenkinsfile?
- What is the pipeline block?
- What is an agent?
- What are stages?
- What are steps?

## Intermediate

- Environment vs Parameters.
- What is the post block?
- Why use pipeline options?
- Explain parallel stages.
- How do Jenkins Credentials improve security?

## Advanced

- Design a production-ready Jenkinsfile for a cloud-native application integrating GitHub, SonarQube, Trivy, Docker, Amazon ECR, Terraform, and Amazon EKS.
- Explain how Declarative Jenkinsfile syntax supports reusable, maintainable, and secure enterprise CI/CD pipelines.
- A financial organization requires a standardized Jenkins pipeline for hundreds of applications. Explain how you would design the Jenkinsfile structure, parameterization, credentials management, stage organization, approval gates, artifact handling, and post-build actions to support scalability, governance, and compliance.

---

# Chapter 5 - Jenkins Pipeline Syntax (Complete Jenkinsfile Directives)

A Jenkinsfile is built using

multiple directives.

These directives define

- Where the pipeline runs
- How it executes
- What stages run
- What happens after execution
- How failures are handled

Understanding these directives is essential for building enterprise-grade Jenkins pipelines.

---

# Complete Jenkinsfile Flow

```text
Pipeline

↓

Agent

↓

Tools

↓

Environment

↓

Options

↓

Parameters

↓

Triggers

↓

Stages

↓

Post
```

Every enterprise Jenkinsfile

follows this structure.

---

# Agent Directive

The agent directive

defines

where

the pipeline executes.

Options

```text
Any

Specific Label

Docker

Dockerfile

Kubernetes

None
```

Different workloads

can use

different execution environments.

---

# Agent Any

The pipeline

runs

on

any available agent.

Workflow

```text
Pipeline

↓

Available Agent

↓

Execution
```

Useful

for general-purpose pipelines.

---

# Agent Label

Pipelines

can target

specific agents.

Example

```text
linux

docker

terraform

kubernetes

windows
```

Ensures

the correct environment

is selected.

---

# Docker Agent

A stage

or pipeline

can run

inside

a Docker container.

Architecture

```text
Pipeline

↓

Docker Container

↓

Execution

↓

Container Removed
```

Provides

consistent build environments.

---

# Dockerfile Agent

Instead of using

an existing image,

Jenkins

can build

a Docker image

from a Dockerfile

and execute

inside it.

Workflow

```text
Dockerfile

↓

Docker Image

↓

Pipeline
```

---

# Kubernetes Agent

Using

the Kubernetes Plugin,

Jenkins

creates

temporary Pods

for execution.

Architecture

```text
Pipeline

↓

Kubernetes Pod

↓

Execution

↓

Pod Deleted
```

Ideal

for cloud-native environments.

---

# None Agent

When

multiple stages

require different agents,

define

```text
agent none
```

Each stage

selects

its own agent.

---

# Tools Directive

Automatically installs

or configures

required tools.

Examples

```text
JDK

Maven

Gradle

Node.js
```

Ensures

consistent versions.

---

# Environment Directive

Stores

pipeline-wide

variables.

Examples

```text
APP_NAME

AWS_REGION

DOCKER_IMAGE

EKS_CLUSTER

DEPLOY_ENV
```

Avoid

hardcoding values.

---

# Options Directive

Options

control

pipeline execution.

Common options

- Timeout
- Retry
- Disable Concurrent Builds
- Build Retention
- Skip Default Checkout
- Timestamp Logs

These improve

pipeline reliability.

---

# Timeout

Limits

pipeline execution time.

Benefits

- Prevents Hanging Builds
- Frees Build Agents

---

# Retry

Automatically retries

failed stages

or pipelines.

Useful

for temporary issues

such as

network failures.

---

# Disable Concurrent Builds

Only one pipeline

runs

at a time.

```text
Pipeline

↓

Running

↓

Second Build Waits
```

Prevents

deployment conflicts.

---

# Build Discarder

Automatically removes

old builds.

Benefits

- Saves Storage
- Improves Performance

---

# Skip Default Checkout

Disables

automatic repository checkout.

Useful

when

multiple repositories

are used.

---

# Parameters Directive

Pipelines

accept

runtime values.

Examples

```text
Environment

Application Version

Region

Deployment Type
```

One pipeline

supports

multiple deployments.

---

# Parameter Types

Common parameter types

```text
String

Boolean

Choice

Password

File
```

Choose

the appropriate type

for user input.

---

# Triggers Directive

Starts pipelines

automatically.

Examples

```text
GitHub Webhook

Poll SCM

Cron Schedule

Manual Build
```

Automation

eliminates

manual execution.

---

# Cron Trigger

Pipelines

can execute

on schedules.

Examples

```text
Nightly Build

Weekly Scan

Monthly Cleanup
```

---

# Poll SCM

Jenkins

periodically checks

the Git repository

for changes.

Architecture

```text
Repository

↓

Polling

↓

Pipeline
```

Webhooks

are generally preferred.

---

# Webhook Trigger

GitHub

immediately notifies

Jenkins

after

a code push.

```text
Developer

↓

Git Push

↓

Webhook

↓

Pipeline
```

Provides

real-time CI.

---

# Stages Directive

Stages divide

the pipeline

into logical sections.

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

---

# Parallel Directive

Independent stages

execute simultaneously.

Architecture

```text
Build

↓

Unit Test

↓

Security Scan

↓

Lint
```

Reduces

pipeline duration.

---

# Matrix Builds

A pipeline

can execute

across

multiple environments.

Example

```text
Ubuntu

↓

Windows

↓

Java 17

↓

Java 21
```

Useful

for compatibility testing.

---

# When Directive

Executes stages

only

when

conditions are met.

Example

```text
Main Branch

↓

Deploy

────────────

Feature Branch

↓

Skip
```

---

# Input Directive

Requests

manual approval

before continuing.

Workflow

```text
Build

↓

Approval

↓

Production Deployment
```

Common

for production environments.

---

# Script Block

Used

when

Declarative syntax

requires

Groovy logic.

Architecture

```text
Pipeline

↓

Script

↓

Custom Logic
```

Use

only

when necessary.

---

# Post Directive

Defines actions

after

pipeline execution.

Common sections

```text
Always

Success

Failure

Unstable

Cleanup
```

---

# Success Block

Executed only

when

the pipeline succeeds.

Typical tasks

- Notify Team
- Publish Reports
- Tag Release

---

# Failure Block

Executed only

when

the pipeline fails.

Typical tasks

- Archive Logs
- Send Alert
- Create Incident

---

# Cleanup Block

Always performs

cleanup.

Examples

```text
Delete Workspace

Remove Temporary Files

Delete Containers
```

Maintains

clean build agents.

---

# Enterprise Pipeline

```text
GitHub

↓

Webhook

↓

Checkout

↓

Build

↓

Unit Test

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
```

---

# Banking Example

```text
Payment API

↓

Checkout

↓

Compile

↓

Unit Test

↓

Security Scan

↓

Approval

↓

Production Deployment
```

Every production deployment

requires

human approval.

---

# Enterprise Jenkinsfile Architecture

```text
Pipeline

├── Agent

├── Tools

├── Environment

├── Options

├── Parameters

├── Triggers

├── Stages

│   ├── Checkout

│   ├── Build

│   ├── Test

│   ├── Docker

│   ├── Terraform

│   └── Deploy

└── Post
```

This is

the recommended structure

for enterprise CI/CD.

---

# Enterprise Best Practices

- Prefer Declarative Pipelines.
- Use labels for specialized agents.
- Store configuration in environment variables.
- Parameterize deployments.
- Use webhooks instead of polling.
- Keep stages modular.
- Add timeout and retry options.
- Always clean workspaces after builds.

---

# Common Mistakes

- Running everything on one agent.
- Hardcoding environment values.
- Ignoring timeout settings.
- Using Poll SCM when webhooks are available.
- Skipping cleanup tasks.
- Not using parameters.
- Writing large Groovy scripts when Declarative syntax is sufficient.

---

# Interview Questions

## Basic

- What is the Agent directive?
- What is the Tools directive?
- What are Jenkins Parameters?
- What is the Post block?
- What is the When directive?

## Intermediate

- Agent Any vs Agent Label.
- Docker Agent vs Kubernetes Agent.
- Poll SCM vs Webhook.
- Why use timeout and retry?
- What is the Script block?

## Advanced

- Design an enterprise Jenkinsfile using Docker agents, Kubernetes agents, parameters, environment variables, approval gates, and post-build cleanup for deploying applications to Amazon EKS.
- Explain how Jenkinsfile directives such as agent, tools, options, parameters, triggers, stages, when, and post work together to build scalable and maintainable CI/CD pipelines.
- A large enterprise manages hundreds of Jenkins pipelines across multiple teams. Explain how you would standardize Jenkinsfile structure, execution environments, parameterization, approval workflows, cleanup strategies, and error handling while ensuring scalability, governance, and maintainability.

---

# Chapter 6 - Jenkins Credentials Management, Environment Variables & Parameters (Enterprise Guide)

Modern Jenkins pipelines require access to

- Git Repositories
- Docker Registries
- AWS Resources
- Kubernetes Clusters
- Databases
- APIs

Hardcoding credentials inside Jenkinsfiles creates serious security risks.

Enterprise Jenkins platforms use

- Jenkins Credentials
- Environment Variables
- Build Parameters
- Secret Injection

to build secure and reusable CI/CD pipelines.

---

# Configuration Flow

```text
Jenkins

↓

Credentials

↓

Environment Variables

↓

Pipeline

↓

Deployment
```

Sensitive data

flows securely

through the pipeline.

---

# Why Credentials Management?

Without Credentials

```text
Jenkinsfile

↓

AWS Keys

↓

GitHub

↓

Security Risk
```

Problems

- Secret Leakage
- Compliance Issues
- Difficult Rotation
- Unauthorized Access

---

With Credentials

```text
Jenkins Credentials

↓

Pipeline

↓

Temporary Usage

↓

Deployment
```

Secrets remain protected.

---

# Jenkins Credentials

Jenkins stores

sensitive information

inside

its Credentials Store.

Examples

- GitHub Token
- AWS Credentials
- Docker Registry Password
- SSH Keys
- Kubernetes Config
- API Tokens

---

# Credentials Architecture

```text
Jenkins

↓

Credentials Store

↓

Pipeline

↓

External Service
```

The pipeline

retrieves credentials

only when required.

---

# Types of Jenkins Credentials

Common credential types

```text
Username & Password

Secret Text

Secret File

SSH Private Key

Certificate
```

Choose

the appropriate type

for each use case.

---

# Username & Password

Used for

- Docker Registry
- Nexus
- Artifactory
- Legacy Applications

Stores

username

and

password together.

---

# Secret Text

Stores

a single secret.

Examples

```text
API Token

JWT Token

Access Token

Webhook Secret
```

---

# Secret File

Stores files

securely.

Examples

```text
Kubeconfig

GCP Service Account JSON

License File

Configuration File
```

---

# SSH Private Key

Used for

secure authentication

to

- Linux Servers
- Git Repositories
- Bastion Hosts

No password

is exposed

inside the pipeline.

---

# Certificate Credentials

Stores

TLS/SSL certificates

for secure communication

with enterprise systems.

---

# Credentials Scope

Credentials

can exist at

```text
Global

↓

Folder

↓

System
```

Use

the smallest scope

required.

---

# Folder Credentials

Large organizations

often separate

applications

into folders.

Architecture

```text
Payments Folder

↓

Payment Credentials

────────────

Retail Folder

↓

Retail Credentials
```

Improves

security isolation.

---

# Environment Variables

Environment variables

store

configuration values.

Examples

```text
Application Name

AWS Region

Cluster Name

Docker Repository

Deployment Environment
```

---

# Variable Scope

Environment variables

can exist at

```text
Global

↓

Pipeline

↓

Stage
```

Avoid

duplicating configuration.

---

# Parameters

Parameters

allow

runtime input.

Examples

```text
Environment

Application Version

AWS Region

Deployment Strategy
```

One pipeline

supports

multiple deployments.

---

# Parameter Types

Common parameter types

```text
String

Choice

Boolean

Password

File
```

---

# Environment Strategy

One pipeline

supports

multiple environments.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Configuration

changes automatically.

---

# Credentials Injection

Workflow

```text
Jenkins Credentials

↓

Pipeline

↓

Environment Variable

↓

Application
```

Secrets are injected

only during execution.

---

# AWS Authentication

Traditional approach

```text
Jenkins

↓

AWS Access Key

↓

AWS
```

Enterprise recommendation

Use

temporary credentials

or IAM Roles

where possible,

especially on AWS-hosted agents.

---

# Docker Authentication

Workflow

```text
Jenkins

↓

Docker Credentials

↓

Amazon ECR

↓

Docker Push
```

Credentials

should never appear

inside logs.

---

# GitHub Authentication

```text
Jenkins

↓

GitHub Token

↓

Repository
```

Used for

- Checkout
- Pull Requests
- Webhooks
- Releases

---

# Kubernetes Authentication

Jenkins

can authenticate

to Kubernetes

using

```text
Kubeconfig

↓

Credentials

↓

Cluster
```

Only authorized pipelines

should deploy.

---

# Secret Rotation

Enterprise organizations

rotate credentials regularly.

Workflow

```text
Old Credential

↓

New Credential

↓

Update Jenkins

↓

Pipeline Continues
```

Pipelines

should not require

code changes

during rotation.

---

# Enterprise Pipeline

```text
GitHub

↓

Jenkins

↓

Credentials

↓

Docker Build

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS
```

Authentication

is centralized.

---

# Banking Example

```text
Payment Pipeline

↓

Jenkins Credentials

↓

AWS Authentication

↓

Docker Push

↓

Amazon EKS

↓

Production
```

No passwords

exist

inside the Jenkinsfile.

---

# Enterprise Security Strategy

```text
Jenkins

↓

Credentials Store

↓

Role-Based Access

↓

Pipeline

↓

Deployment
```

Only authorized jobs

can access

sensitive credentials.

---

# Enterprise Best Practices

- Store all secrets in Jenkins Credentials.
- Never hardcode passwords.
- Use folder-level credentials when possible.
- Use least-privilege AWS permissions.
- Rotate credentials regularly.
- Separate credentials by environment.
- Restrict credential access using RBAC.
- Audit credential usage.

---

# Common Mistakes

- Hardcoding AWS keys.
- Reusing production credentials in development.
- Giving every pipeline access to all credentials.
- Printing secrets in logs.
- Using global credentials unnecessarily.
- Sharing SSH keys across teams.
- Never rotating credentials.

---

# Interview Questions

## Basic

- What are Jenkins Credentials?
- Why should credentials not be hardcoded?
- What is a Secret Text credential?
- What are environment variables?
- What are Jenkins build parameters?

## Intermediate

- Global vs Folder credentials.
- Username/Password vs Secret Text.
- How are credentials injected into pipelines?
- Explain parameterized builds.
- How do environment variables improve pipeline reusability?

## Advanced

- Design a secure Jenkins authentication strategy integrating GitHub, Docker, Amazon ECR, Terraform, and Amazon EKS using Jenkins Credentials, RBAC, and environment-specific configurations.
- Explain how Jenkins Credentials, environment variables, and build parameters work together to build secure, reusable, and maintainable enterprise CI/CD pipelines.
- A financial organization has separate Development, Testing, and Production AWS accounts. Explain how you would design credential management, access isolation, environment configuration, secret rotation, RBAC, and pipeline parameterization to support secure enterprise deployments.

---

# Chapter 7 - Jenkins Shared Libraries & Reusable Pipelines (Enterprise Guide)

As organizations grow,

they often manage

- Hundreds of Jenkins Pipelines
- Thousands of Jenkinsfiles
- Multiple Development Teams

Copying the same pipeline logic into every Jenkinsfile leads to

- Duplicate Code
- Maintenance Challenges
- Configuration Drift
- Inconsistent CI/CD Standards

Enterprise Jenkins platforms solve this using

- Shared Libraries
- Reusable Functions
- Global Variables
- Pipeline Templates

These features help standardize automation across the organization.

---

# Why Shared Libraries?

Without Shared Libraries

```text
Repository A

↓

Build Pipeline

────────────

Repository B

↓

Copy Pipeline

────────────

Repository C

↓

Copy Pipeline
```

Problems

- Duplicate Code
- Difficult Updates
- Multiple Versions
- Inconsistent Pipelines

---

With Shared Libraries

```text
Shared Library

↓

Repository A

↓

Repository B

↓

Repository C
```

One update

benefits

every project.

---

# Shared Library Architecture

```text
GitHub

↓

Shared Library

↓

Jenkins

↓

Pipeline

↓

Application
```

Pipelines

reuse

common functionality.

---

# What is a Jenkins Shared Library?

A Shared Library

is a Git repository

containing

reusable pipeline code.

Examples

- Build Functions
- Deployment Logic
- Security Checks
- Notifications
- Utility Methods

---

# Enterprise Repository Structure

```text
jenkins-shared-library/

├── vars/

├── src/

├── resources/

└── README.md
```

The library

is version controlled

just like application code.

---

# Shared Library Components

A Shared Library

contains

```text
vars/

↓

Global Variables

────────────

src/

↓

Groovy Classes

────────────

resources/

↓

Configuration Files
```

Each directory

has

a specific purpose.

---

# vars Directory

The `vars/` directory

contains

global pipeline functions.

Examples

```text
buildApp

deployApp

dockerBuild

terraformDeploy

notifyTeam
```

These functions

are directly available

inside Jenkinsfiles.

---

# src Directory

The `src/` directory

contains

Groovy classes

for

complex logic.

Examples

```text
AWS Utility

Docker Utility

Terraform Helper

Notification Helper
```

Reusable business logic

is stored here.

---

# resources Directory

Stores

supporting files.

Examples

```text
YAML

JSON

Templates

Shell Scripts
```

Resources

can be loaded

during pipeline execution.

---

# Shared Library Workflow

```text
Developer

↓

GitHub

↓

Jenkinsfile

↓

Shared Library

↓

Pipeline

↓

Deployment
```

Application pipelines

remain small

and readable.

---

# Global Variables

Global variables

provide

reusable pipeline steps.

Workflow

```text
Pipeline

↓

Shared Function

↓

Execution
```

Common tasks

are written once.

---

# Reusable Build Function

Instead of

duplicating

build logic

```text
Application A

↓

Shared Build

────────────

Application B

↓

Shared Build
```

Every project

uses

the same build process.

---

# Reusable Deployment

Deployment logic

can also be shared.

Architecture

```text
Application

↓

Shared Deploy Function

↓

Amazon EKS
```

Deployment standards

remain consistent.

---

# Reusable Security Scan

Security

should be standardized.

Workflow

```text
Pipeline

↓

Shared SonarQube

↓

Shared Trivy

↓

Deployment
```

Every application

passes

the same security checks.

---

# Versioning Shared Libraries

Shared Libraries

should be versioned.

Example

```text
v1

↓

v2

↓

v3
```

Applications

upgrade

when ready.

---

# Library Loading

Pipelines

load

the Shared Library

before execution.

Workflow

```text
Jenkinsfile

↓

Shared Library

↓

Pipeline Starts
```

---

# Enterprise Pipeline

Without Shared Library

```text
Checkout

↓

Build

↓

Test

↓

Deploy
```

Repeated

across

every repository.

---

With Shared Library

```text
Checkout

↓

Shared Build

↓

Shared Test

↓

Shared Deploy
```

One implementation

serves

all repositories.

---

# Multi-Repository Architecture

```text
Shared Library

├── Build

├── Docker

├── Terraform

├── Kubernetes

├── Notifications

└── Security

↓

Application A

↓

Application B

↓

Application C
```

Centralized automation

improves maintainability.

---

# Enterprise Deployment Flow

```text
Developer

↓

GitHub

↓

Jenkins

↓

Shared Library

↓

Build

↓

Docker

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS
```

The pipeline

remains standardized.

---

# Banking Example

```text
Payment Service

↓

Shared Build

↓

Shared Security

↓

Shared Deployment

↓

Amazon EKS

↓

Production
```

Every banking application

uses

the same deployment process.

---

# Governance

Platform teams

maintain

Shared Libraries.

Development teams

consume

approved pipeline functions.

Benefits

- Standardization
- Security
- Compliance
- Easier Maintenance

---

# Enterprise Best Practices

- Store Shared Libraries in Git.
- Version every library release.
- Keep reusable functions small and focused.
- Separate utility classes from global variables.
- Document every shared function.
- Standardize build and deployment logic.
- Reuse security scanning stages.
- Test Shared Libraries before publishing.

---

# Common Mistakes

- Copying Jenkinsfiles across repositories.
- Creating very large shared functions.
- Hardcoding application-specific values.
- Not versioning Shared Libraries.
- Mixing reusable and project-specific logic.
- Allowing every team to modify the library.
- Skipping documentation.

---

# Interview Questions

## Basic

- What is a Jenkins Shared Library?
- Why do we use Shared Libraries?
- What is the `vars` directory?
- What is the `src` directory?
- What is the `resources` directory?

## Intermediate

- How are Shared Libraries loaded?
- Global Variables vs Groovy Classes.
- Why should Shared Libraries be versioned?
- How do Shared Libraries reduce duplication?
- Explain enterprise pipeline standardization.

## Advanced

- Design a centralized Jenkins Shared Library architecture for hundreds of applications using reusable build, security, Docker, Terraform, and Kubernetes deployment functions.
- Explain how Shared Libraries improve maintainability, governance, and CI/CD standardization across multiple development teams.
- A large enterprise has more than 600 Jenkins pipelines across different business units. Explain how you would design, version, secure, test, and govern Jenkins Shared Libraries to ensure reusable automation, reduce maintenance overhead, and support enterprise-scale CI/CD.

---

# Chapter 8 - Jenkins Docker Integration, Amazon ECR & Amazon EKS Deployment

Modern Jenkins pipelines are responsible for

- Building Applications
- Running Tests
- Building Docker Images
- Scanning Images
- Publishing Images
- Deploying to Kubernetes

Jenkins integrates seamlessly with

- Docker
- Amazon ECR
- Amazon EKS

to automate the complete software delivery lifecycle.

---

# Enterprise Deployment Architecture

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

Amazon EKS

↓

Monitoring
```

Every deployment

follows

a standardized pipeline.

---

# Why Docker Integration?

Without Docker

```text
Build

↓

Package

↓

Manual Server Deployment
```

Problems

- Environment Differences
- Manual Configuration
- Inconsistent Deployments

---

With Docker

```text
Build

↓

Docker Image

↓

Amazon ECR

↓

Amazon EKS
```

Deployments become

portable

and consistent.

---

# Docker Build Stage

The first container stage

creates

a Docker image.

Workflow

```text
Source Code

↓

Dockerfile

↓

Docker Build

↓

Docker Image
```

The Docker image

becomes

the deployment artifact.

---

# Image Tagging Strategy

Every Docker image

should have

a unique tag.

Examples

```text
v1.0.0

Build Number

Git Commit SHA

Release Tag
```

Avoid

using

`latest`

in production.

---

# Security Scanning

Every image

must be scanned

before deployment.

Common tools

- Trivy
- Docker Scout
- Amazon ECR Scan

Critical vulnerabilities

should block

the pipeline.

---

# Amazon Elastic Container Registry (ECR)

Amazon ECR

is a private

container registry.

Architecture

```text
Docker Image

↓

Amazon ECR

↓

Versioned Repository
```

Only approved images

should be stored.

---

# Docker Push Workflow

```text
Build

↓

Docker Image

↓

Security Scan

↓

Amazon ECR
```

The image

is now available

for deployment.

---

# Build Once, Deploy Everywhere

Enterprise CI/CD

builds

one immutable image.

```text
Build

↓

Development

↓

Testing

↓

Staging

↓

Production
```

Never rebuild

for each environment.

---

# Amazon EKS Deployment

After publishing

the Docker image,

Jenkins deploys

to Amazon EKS.

Workflow

```text
Amazon ECR

↓

Amazon EKS

↓

Pods

↓

Application
```

---

# Kubernetes Deployment

Deployment flow

```text
Deployment

↓

ReplicaSet

↓

Pods
```

The Kubernetes cluster

pulls

the approved image

from Amazon ECR.

---

# Rolling Deployment

Recommended deployment strategy

```text
Old Pods

↓

New Pods

↓

Validation

↓

Traffic Shift
```

Minimizes downtime.

---

# Blue-Green Deployment

Alternative strategy

```text
Blue Environment

↓

Green Environment

↓

Validation

↓

Traffic Switch
```

Rollback

is immediate.

---

# Canary Deployment

Production traffic

is shifted

gradually.

```text
10%

↓

25%

↓

50%

↓

100%
```

Reduces deployment risk.

---

# Deployment Approval

Production deployments

often require

manual approval.

Workflow

```text
Security Scan

↓

Approval

↓

Production Deployment
```

Provides

an additional

security checkpoint.

---

# Environment Strategy

One Jenkins pipeline

supports

multiple environments.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Configuration

changes automatically

using parameters

and environment variables.

---

# Rollback Strategy

If deployment fails

```text
Current Version

↓

Previous Docker Image

↓

Redeploy

↓

Production Restored
```

Rollback

should be

quick

and predictable.

---

# Deployment Validation

After deployment

verify

- Pod Status
- Health Checks
- Logs
- Metrics
- Application Availability

Deployment

is successful

only after validation.

---

# Monitoring

Monitor

- Pod Health
- CPU Usage
- Memory Usage
- Restart Count
- Response Time

Use

- Prometheus
- Grafana
- ELK

for observability.

---

# Enterprise Deployment Pipeline

```text
GitHub

↓

Webhook

↓

Jenkins

↓

Build

↓

Unit Test

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
```

The pipeline

automates

the complete delivery process.

---

# Banking Example

```text
Payment Service

↓

Jenkins

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

Amazon EKS

↓

Rolling Deployment

↓

Customers
```

Every release

passes

security

and quality gates.

---

# Enterprise Architecture

```text
Developer

↓

GitHub

↓

Jenkins

↓

Docker

↓

Trivy

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS

↓

Prometheus

↓

Grafana

↓

ELK
```

This architecture

provides

secure,

automated,

and observable deployments.

---

# Enterprise Deployment Checklist

Before production deployment verify

✓ Build Successful

✓ Unit Tests Passed

✓ SonarQube Passed

✓ Trivy Scan Passed

✓ Docker Image Tagged

✓ Image Uploaded to Amazon ECR

✓ Terraform Validated

✓ Amazon EKS Deployment Successful

✓ Monitoring Enabled

✓ Rollback Strategy Ready

---

# Enterprise Best Practices

- Build immutable Docker images.
- Use versioned image tags.
- Scan images before deployment.
- Use Amazon ECR as the source of truth.
- Deploy using rolling or blue-green strategies.
- Validate deployments before completion.
- Automate rollback procedures.
- Monitor production continuously.

---

# Common Mistakes

- Using the `latest` image tag.
- Deploying unscanned images.
- Rebuilding images for each environment.
- Skipping deployment validation.
- Ignoring rollback planning.
- Deploying directly from feature branches.
- Not monitoring production deployments.

---

# Interview Questions

## Basic

- How does Jenkins build Docker images?
- What is Amazon ECR?
- What is Amazon EKS?
- Why is Docker image tagging important?
- Why automate deployments?

## Intermediate

- Explain the deployment pipeline from Jenkins to Amazon EKS.
- Rolling Deployment vs Blue-Green Deployment.
- What is Canary Deployment?
- Why Build Once, Deploy Everywhere?
- How do you validate a Kubernetes deployment?

## Advanced

- Design a complete Jenkins CI/CD pipeline integrating Docker, SonarQube, Trivy, Amazon ECR, Terraform, Amazon EKS, Prometheus, Grafana, and ELK.
- Explain how immutable Docker images, automated security scanning, deployment approvals, and Kubernetes rollout strategies work together to deliver secure and reliable production deployments.
- A financial organization deploys hundreds of microservices to Amazon EKS every day using Jenkins. Explain how you would design the CI/CD pipeline, image management strategy, deployment approval process, rollback mechanism, monitoring, and governance to ensure high availability, security, and compliance.

---

