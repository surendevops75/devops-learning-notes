# GitHub Actions Enterprise Handbook

# Chapter 1 - GitHub Actions Fundamentals & Enterprise CI/CD Architecture

Modern software development requires applications to be

- Built Automatically
- Tested Automatically
- Scanned Automatically
- Deployed Automatically

Manually performing these tasks is

- Slow
- Error-prone
- Difficult to Scale

GitHub Actions automates the complete software delivery lifecycle.

Today, it is one of the most widely used CI/CD platforms for cloud-native applications.

---

# What is GitHub Actions?

GitHub Actions is GitHub's native

**Continuous Integration and Continuous Delivery (CI/CD)** platform.

It automates workflows directly from a GitHub repository.

Examples

- Build Applications
- Run Unit Tests
- Scan Code
- Build Docker Images
- Deploy Infrastructure
- Deploy Applications
- Send Notifications

---

# Why GitHub Actions?

Without GitHub Actions

```text
Developer

↓

Build

↓

Test

↓

Deploy

↓

Production
```

Everything is performed manually.

Problems

- Human Errors
- Slow Releases
- No Standardization
- Difficult Rollbacks

---

With GitHub Actions

```text
Git Push

↓

Workflow

↓

Build

↓

Test

↓

Deploy
```

Every deployment

follows the same automated process.

---

# CI/CD Overview

Continuous Integration

focuses on

```text
Code

↓

Build

↓

Test
```

Continuous Delivery

focuses on

```text
Validated Build

↓

Deploy

↓

Approval

↓

Production
```

GitHub Actions supports

both.

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub Repository

↓

GitHub Actions

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

Monitoring
```

---

# GitHub Actions Components

GitHub Actions consists of

- Workflow
- Event
- Job
- Step
- Runner
- Action
- Artifact
- Secrets

These are the core building blocks.

---

# Workflow

A Workflow is

an automated process

defined using YAML.

Workflow

contains

- Jobs
- Steps
- Triggers

One repository

may contain

multiple workflows.

---

# Workflow Architecture

```text
Workflow

├── Job 1

├── Job 2

└── Job 3
```

Each workflow

performs

a complete automation process.

---

# Event

An Event

triggers

a workflow.

Common Events

- Push
- Pull Request
- Release
- Schedule
- Workflow Dispatch

---

# Event Flow

```text
Git Push

↓

Workflow Trigger

↓

Pipeline Starts
```

---

# Job

A Job

is a collection

of related steps.

Example

```text
Build Job

↓

Compile

↓

Test

↓

Package
```

Jobs

can execute

sequentially

or in parallel.

---

# Step

A Step

is

a single task.

Examples

```text
Checkout Code

↓

Install Dependencies

↓

Run Tests

↓

Build Docker Image
```

---

# Runner

A Runner

executes

workflow jobs.

Types

- GitHub Hosted Runner
- Self-hosted Runner

---

# GitHub Hosted Runner

GitHub provides

managed virtual machines.

Benefits

- Easy Setup
- Automatic Maintenance
- Scalable
- No Infrastructure Management

---

# Self-hosted Runner

Organizations

can host

their own runners.

Architecture

```text
GitHub

↓

Self-hosted Runner

↓

Private Network

↓

Deployment
```

Useful

for accessing

internal infrastructure.

---

# Action

An Action

is a reusable automation component.

Examples

- Checkout Repository
- Setup Java
- Setup Node.js
- Login to AWS
- Upload Artifact

Actions reduce

duplicate workflow logic.

---

# Workflow Execution

```text
Event

↓

Workflow

↓

Job

↓

Step

↓

Runner

↓

Result
```

---

# Artifacts

Artifacts

store

workflow outputs.

Examples

- Build Packages
- Reports
- Test Results
- Terraform Plans
- Docker Images Metadata

Artifacts

can be downloaded later.

---

# Secrets

Sensitive values

should never be stored

inside workflows.

Use GitHub Secrets

for

- AWS Credentials
- Tokens
- Passwords
- API Keys

---

# Repository Structure

```text
project/

├── src/

├── Dockerfile

├── compose.yaml

└── .github/

    └── workflows/

        ├── build.yml

        ├── deploy.yml

        └── terraform.yml
```

All workflows

are stored under

`.github/workflows`.

---

# Enterprise Deployment Flow

```text
Developer

↓

Git Push

↓

GitHub Actions

↓

Build

↓

Test

↓

Security Scan

↓

Docker Build

↓

Amazon ECR

↓

Amazon EKS
```

---

# Banking Example

```text
Developer

↓

Payment Service

↓

GitHub Push

↓

GitHub Actions

↓

Build

↓

Tests

↓

Docker Image

↓

Amazon ECR

↓

Amazon EKS

↓

Customers
```

Every deployment

is automated.

---

# GitHub Actions in DevOps

GitHub Actions integrates with

- Docker
- Terraform
- Kubernetes
- Amazon EKS
- Amazon ECR
- AWS IAM
- Argo CD
- SonarQube
- Trivy
- Prometheus

Making it

a complete DevOps automation platform.

---

# Benefits

- Native GitHub Integration
- Automated CI/CD
- Easy Workflow Management
- Cloud Native
- Reusable Actions
- Built-in Secrets
- Fast Deployment
- Enterprise Ready

---

# Best Practices

- Keep workflows modular.
- Store secrets securely.
- Use reusable actions.
- Separate CI and CD workflows.
- Keep YAML files readable.
- Automate testing.
- Automate security scanning.
- Version reusable actions.

---

# Common Mistakes

- Hardcoding credentials.
- Creating one massive workflow.
- Running production deployments on every push.
- Ignoring workflow failures.
- Giving workflows excessive permissions.
- Not using branch protection.
- Skipping automated testing.

---

# Interview Questions

## Basic

- What is GitHub Actions?
- What is CI/CD?
- What is a Workflow?
- What is a Job?
- What is a Step?

## Intermediate

- Explain GitHub Hosted Runner vs Self-hosted Runner.
- What are GitHub Actions?
- What are workflow triggers?
- Explain workflow execution.
- How do GitHub Actions integrate with Docker and Kubernetes?

## Advanced

- Design an enterprise GitHub Actions CI/CD platform for deploying Docker applications to Amazon EKS using Terraform, Amazon ECR, security scanning, and automated approvals.
- Explain the complete GitHub Actions workflow from code commit to production deployment.
- A company wants to replace Jenkins with GitHub Actions for all CI/CD pipelines. Explain the architecture, migration strategy, workflow organization, runner strategy, security model, deployment process, and governance approach.

---

# Chapter 2 - GitHub Actions Workflow Syntax (Deep Dive)

Every GitHub Actions automation starts with a **Workflow YAML file**.

Understanding workflow syntax is the foundation for building production-grade CI/CD pipelines.

This chapter explains

- Workflow Structure
- YAML Syntax
- Events
- Jobs
- Steps
- Expressions
- Variables
- Contexts
- Dependencies
- Timeouts
- Permissions

---

# Workflow File Location

Every workflow must be stored inside

```text
.github/

↓

workflows/

↓

workflow.yml
```

GitHub automatically detects

all workflow files

inside this directory.

---

# Workflow Architecture

```text
GitHub Event

↓

Workflow

↓

Jobs

↓

Steps

↓

Runner

↓

Result
```

Every workflow follows this execution model.

---

# Basic Workflow Structure

A workflow consists of

```text
Name

↓

Trigger

↓

Jobs

↓

Steps
```

These are the minimum required components.

---

# Workflow Lifecycle

```text
Developer Push

↓

GitHub Event

↓

Workflow Starts

↓

Runner Allocated

↓

Jobs Execute

↓

Workflow Completed
```

---

# Workflow Name

Every workflow

should have

a meaningful name.

Example

```text
Build Application

Terraform Deployment

Docker Build

Production Deployment
```

Meaningful names

improve visibility.

---

# Events (Triggers)

Events determine

when

a workflow starts.

Common events

```text
Push

Pull Request

Release

Schedule

Manual Trigger
```

---

# Push Event

Workflow starts

whenever code

is pushed.

Architecture

```text
Developer

↓

Git Push

↓

Workflow
```

Used for

Continuous Integration.

---

# Pull Request Event

Triggered

when

a Pull Request

is created,

updated,

or merged.

Workflow

```text
Developer

↓

Pull Request

↓

Validation

↓

Review
```

Useful

for code quality checks.

---

# Release Event

Triggered

when

a GitHub Release

is published.

Typical workflow

```text
Release

↓

Build

↓

Deploy
```

---

# Schedule Event

Workflows

can execute

automatically

using schedules.

Examples

- Nightly Builds
- Weekly Scans
- Monthly Reports

---

# Manual Workflow

GitHub supports

manual execution.

Workflow

```text
Engineer

↓

Manual Trigger

↓

Workflow
```

Useful

for production deployments.

---

# Multiple Triggers

One workflow

may respond

to multiple events.

Example

```text
Push

↓

OR

↓

Pull Request

↓

Workflow
```

---

# Jobs

A Job

groups

related tasks.

Example

```text
Build Job

↓

Compile

↓

Test

↓

Package
```

---

# Multiple Jobs

One workflow

can contain

multiple jobs.

Architecture

```text
Workflow

├── Build

├── Test

├── Security

└── Deploy
```

Jobs

can execute

in parallel

or sequentially.

---

# Job Dependencies

Example

```text
Build

↓

Test

↓

Deploy
```

Deployment

starts only

after testing succeeds.

---

# Parallel Jobs

Independent jobs

can execute

simultaneously.

```text
Build

↓

Test

↓

Lint

↓

Security Scan
```

This reduces

pipeline execution time.

---

# Steps

Each Job

contains

multiple Steps.

Example

```text
Checkout Code

↓

Install Dependencies

↓

Run Tests

↓

Build Docker Image
```

Steps execute

in order.

---

# Actions

Most steps

use reusable

GitHub Actions.

Examples

```text
Checkout Repository

Setup Java

Setup Node

Upload Artifact

AWS Login
```

---

# Run Commands

Steps

may also execute

shell commands.

Workflow

```text
Runner

↓

Shell

↓

Command

↓

Output
```

---

# Runner Selection

Every Job

requires

a Runner.

Options

```text
GitHub Hosted

↓

OR

↓

Self-hosted
```

---

# Environment Variables

Workflows

support

environment variables.

Architecture

```text
Workflow

↓

Environment Variable

↓

Job

↓

Step
```

Used for

configuration.

---

# Variables Scope

Variables

can exist

at

```text
Workflow Level

↓

Job Level

↓

Step Level
```

Choose

the appropriate scope

to reduce duplication.

---

# Expressions

GitHub Actions

supports expressions

for dynamic logic.

Workflow

```text
Condition

↓

Expression

↓

Execute
```

Used for

conditional execution.

---

# Contexts

Contexts provide

workflow information.

Examples

```text
Repository

↓

Branch

↓

Commit

↓

Actor

↓

Event
```

Contexts make

workflows dynamic.

---

# Conditional Execution

Jobs

or Steps

may execute

only when

certain conditions

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

# Matrix Strategy

A matrix

runs

the same job

multiple times.

Example

```text
Ubuntu

↓

Windows

↓

macOS
```

Useful

for cross-platform testing.

---

# Artifacts

Artifacts

store

workflow outputs.

Examples

```text
Build Package

Test Reports

Terraform Plan

Coverage Report
```

Artifacts

are available

after workflow completion.

---

# Timeouts

Jobs

should define

reasonable timeouts.

Benefits

- Prevent Hanging Jobs
- Reduce Runner Usage
- Faster Failure Detection

---

# Permissions

GitHub Actions

supports

fine-grained permissions.

Grant only

required permissions.

Avoid

full repository access

unless necessary.

---

# Enterprise Workflow Architecture

```text
Push

↓

Workflow

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

Deploy
```

---

# Banking Example

```text
Developer

↓

Git Push

↓

Workflow

↓

Build

↓

Security Scan

↓

Approval

↓

Production
```

Every deployment

is validated

before production.

---

# Enterprise Workflow Structure

```text
.github/

└── workflows/

    ├── build.yml

    ├── test.yml

    ├── docker.yml

    ├── terraform.yml

    ├── deploy.yml

    └── cleanup.yml
```

Each workflow

has

one responsibility.

---

# Best Practices

- Use meaningful workflow names.
- Keep workflows modular.
- Separate CI and CD.
- Use job dependencies.
- Minimize workflow permissions.
- Reuse common actions.
- Use artifacts between jobs.
- Set job timeouts.

---

# Common Mistakes

- One huge workflow for everything.
- Hardcoding environment values.
- Running production deployments on every push.
- Giving workflows write permissions unnecessarily.
- Ignoring failed jobs.
- Not using reusable actions.
- Duplicating workflow logic.

---

# Interview Questions

## Basic

- What is a GitHub Actions Workflow?
- What are workflow triggers?
- What is a Job?
- What is a Step?
- What is a Runner?

## Intermediate

- Explain workflow execution.
- Job vs Step.
- Workflow vs Action.
- Explain matrix strategy.
- What are GitHub contexts?

## Advanced

- Design a modular GitHub Actions workflow architecture for a large enterprise with separate workflows for Build, Testing, Security, Docker, Terraform, and Production Deployment.
- Explain how workflow syntax, job dependencies, reusable actions, artifacts, permissions, and conditional execution help create scalable CI/CD pipelines.
- A company has over 300 repositories and wants standardized CI/CD pipelines. Explain how you would organize workflow files, reuse actions, manage permissions, optimize execution time, and maintain consistency across all projects.

---

# Chapter 3 - GitHub Actions Runners (GitHub Hosted & Self-hosted) - Enterprise Guide

Every GitHub Actions workflow needs a machine to execute its jobs.

This machine is called a **Runner**.

Without a runner,

a workflow cannot execute

- Build
- Test
- Scan
- Deploy
- Package

Understanding runners is essential for designing secure and scalable CI/CD pipelines.

---

# What is a Runner?

A Runner is

the execution environment

that runs

GitHub Actions jobs.

Architecture

```text
GitHub

↓

Workflow

↓

Runner

↓

Job Execution
```

Every job

runs on

a runner.

---

# Runner Workflow

```text
Developer Push

↓

Workflow Trigger

↓

Runner Assigned

↓

Job Executes

↓

Results Uploaded
```

---

# Types of Runners

GitHub Actions supports

- GitHub Hosted Runner
- Self-hosted Runner

Each has

different use cases.

---

# GitHub Hosted Runner

GitHub provides

fully managed virtual machines.

Architecture

```text
GitHub

↓

GitHub Hosted Runner

↓

Workflow

↓

Result
```

GitHub

creates,

manages,

and destroys

the runner automatically.

---

# Hosted Runner Lifecycle

```text
Workflow Starts

↓

Runner Created

↓

Jobs Execute

↓

Runner Destroyed
```

Each workflow

gets

a fresh environment.

---

# Advantages of Hosted Runners

- No Infrastructure Management
- Automatic Updates
- Fast Provisioning
- Clean Environment
- Easy Maintenance
- Scalable

Ideal for

most CI workloads.

---

# Limitations of Hosted Runners

- Limited Runtime
- Internet Dependency
- No Access to Private Networks
- Shared Infrastructure
- Limited Customization

---

# Self-hosted Runner

A Self-hosted Runner

is managed

by your organization.

Architecture

```text
GitHub

↓

Self-hosted Runner

↓

Private Network

↓

Infrastructure
```

Useful when

deployment targets

are inside private networks.

---

# Self-hosted Runner Lifecycle

```text
Runner Installed

↓

Registers with GitHub

↓

Waits for Jobs

↓

Executes Jobs

↓

Ready Again
```

Unlike hosted runners,

self-hosted runners

remain online

until stopped.

---

# Hosted vs Self-hosted

| GitHub Hosted | Self-hosted |
|---------------|-------------|
| Managed by GitHub | Managed by Organization |
| Temporary VM | Persistent Machine |
| Easy Setup | Requires Maintenance |
| Public Internet | Private Networks Supported |
| Automatic Updates | Manual Updates |

---

# Runner Labels

Runners

can have

labels

to identify capabilities.

Examples

```text
Linux

Windows

macOS

Docker

Terraform

Production
```

Workflows

select runners

using labels.

---

# Runner Groups

Large organizations

group runners.

Example

```text
Development

↓

Dev Runners

────────────

Production

↓

Prod Runners
```

Improves

security

and organization.

---

# Operating Systems

GitHub Hosted Runners

support

```text
Ubuntu

Windows

macOS
```

Teams choose

the appropriate runner

based on application requirements.

---

# Runner Resources

Each runner provides

- CPU
- Memory
- Storage
- Network

Resource-intensive workloads

may require

larger runners

or self-hosted infrastructure.

---

# Parallel Execution

Multiple runners

allow

parallel execution.

Example

```text
Build

↓

Runner 1

────────────

Test

↓

Runner 2

────────────

Security Scan

↓

Runner 3
```

Pipeline execution time

is significantly reduced.

---

# Runner Communication

```text
GitHub

↓

Runner

↓

Repository

↓

Workflow

↓

Results
```

The runner

downloads

workflow instructions,

executes them,

and uploads results.

---

# Runner Security

A runner executes

repository code.

Therefore,

security is critical.

Protect runners using

- Least Privilege
- Network Isolation
- IAM Roles
- Short-lived Credentials

---

# Self-hosted Runner Security

Best practices

- Dedicated Machines
- Private Network
- Regular Updates
- Restricted Repository Access
- Monitoring
- Least Privilege

Never share

production runners

with development workloads.

---

# Runner Cleanup

Hosted runners

are destroyed automatically.

Self-hosted runners

require periodic cleanup.

Remove

- Temporary Files
- Docker Images
- Build Cache
- Logs

to maintain performance.

---

# Auto Scaling Self-hosted Runners

Enterprise platforms

often auto-scale runners.

Architecture

```text
GitHub

↓

Auto Scaling

↓

EC2 Instances

↓

Workflow

↓

Terminate After Completion
```

This combines

flexibility

with cost optimization.

---

# Private Deployment Example

```text
GitHub

↓

Self-hosted Runner

↓

Private VPC

↓

Amazon EKS

↓

Deployment
```

The runner

can access

private AWS resources

that hosted runners cannot.

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

Runner

↓

Docker Build

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS
```

The runner

executes

every stage

of the pipeline.

---

# Banking Example

```text
Developer

↓

GitHub

↓

Self-hosted Runner

↓

Private AWS Account

↓

Terraform

↓

Amazon EKS

↓

Production
```

Sensitive deployments

remain inside

the organization's network.

---

# Runner Selection Strategy

| Scenario | Recommended Runner |
|----------|--------------------|
| Open Source Project | GitHub Hosted |
| Internal Application | GitHub Hosted |
| Private Infrastructure | Self-hosted |
| Production Deployment | Self-hosted (Common) |
| High Security Environment | Self-hosted |

---

# Enterprise Best Practices

- Use GitHub Hosted runners for standard CI jobs.
- Use Self-hosted runners for private infrastructure.
- Separate development and production runners.
- Apply least-privilege permissions.
- Keep runners updated.
- Enable monitoring and logging.
- Auto-scale runners where possible.
- Remove unused runners.

---

# Common Mistakes

- Running production deployments from shared runners.
- Giving runners excessive permissions.
- Using one runner for all environments.
- Ignoring runner updates.
- Allowing unrestricted repository access.
- Leaving self-hosted runners unmonitored.
- Accumulating Docker images and build cache on self-hosted runners.

---

# Interview Questions

## Basic

- What is a GitHub Actions Runner?
- What are the types of runners?
- What is a GitHub Hosted Runner?
- What is a Self-hosted Runner?
- Why does every workflow require a runner?

## Intermediate

- GitHub Hosted vs Self-hosted Runner.
- Explain runner labels.
- What are runner groups?
- Why are self-hosted runners used?
- How do runners execute workflows?

## Advanced

- Design a secure GitHub Actions runner architecture for deploying applications to private Amazon EKS clusters using Docker, Terraform, and IAM roles.
- Explain how GitHub Hosted and Self-hosted runners differ in terms of scalability, security, maintenance, and enterprise deployment strategies.
- A financial organization requires CI/CD pipelines to deploy applications into private AWS accounts without exposing infrastructure to the public internet. Explain how you would design the runner architecture, networking, IAM permissions, auto-scaling strategy, monitoring, and governance for GitHub Actions.

---

