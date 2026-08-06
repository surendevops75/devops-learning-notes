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

# Chapter 4 - GitHub Actions Variables, Secrets & Contexts (Enterprise Guide)

Modern CI/CD pipelines must support

- Multiple Environments
- Dynamic Configuration
- Secure Credentials
- Conditional Logic

Hardcoding values inside workflows makes pipelines

- Difficult to Maintain
- Insecure
- Environment Dependent

GitHub Actions solves this using

- Variables
- Secrets
- Contexts
- Expressions
- Environment Variables

These features make workflows secure, reusable, and enterprise-ready.

---

# Configuration Flow

```text
Repository

↓

Variables

↓

Secrets

↓

Contexts

↓

Workflow

↓

Job

↓

Step
```

Dynamic values

flow through

the workflow.

---

# Why Variables?

Without variables

```text
Workflow

↓

Hardcoded Values

↓

Development

↓

Production

↓

Modify YAML
```

Every environment

requires editing the workflow.

---

With variables

```text
Variables

↓

Workflow

↓

Development

↓

Testing

↓

Production
```

One workflow

supports multiple environments.

---

# Types of Variables

GitHub Actions supports

- Environment Variables
- Repository Variables
- Organization Variables
- Environment-specific Variables

Each serves

a different purpose.

---

# Environment Variables

Environment variables

store

configuration values.

Examples

```text
Application Name

Environment

Region

Port

Cluster Name
```

They are available

during workflow execution.

---

# Repository Variables

Repository variables

are shared

across workflows

inside one repository.

Example

```text
AWS Region

Application Name

Docker Repository
```

Useful

for reusable configuration.

---

# Organization Variables

Organizations

can define

shared variables

used across

multiple repositories.

Architecture

```text
Organization

↓

Repository

↓

Workflow
```

This reduces duplication.

---

# Environment Variables (GitHub Environment)

GitHub Environments

allow

environment-specific values.

Example

```text
Development

↓

EKS Dev

────────────

Production

↓

EKS Prod
```

One workflow

deploys to

different environments.

---

# Variable Scope

Variables

can exist at

```text
Workflow

↓

Job

↓

Step
```

Choose

the smallest scope

required.

---

# What are Secrets?

Secrets store

sensitive information.

Examples

```text
AWS Credentials

GitHub Token

API Keys

Database Passwords
```

Secrets

are encrypted

and masked

during workflow execution.

---

# Secret Flow

```text
GitHub Secret

↓

Workflow

↓

Runner

↓

Application
```

Secrets

are never exposed

in logs.

---

# Why Secrets?

Without secrets

```text
Workflow

↓

Hardcoded Password

↓

Git Repository
```

Major security risk.

---

With secrets

```text
Encrypted Secret

↓

Workflow

↓

Temporary Usage
```

Credentials remain protected.

---

# Repository Secrets

Repository Secrets

are available

only inside

that repository.

Examples

```text
AWS_ACCESS_KEY

AWS_SECRET_KEY

DOCKER_PASSWORD
```

---

# Organization Secrets

Organization Secrets

are shared

across

multiple repositories.

Architecture

```text
Organization

↓

Repositories

↓

Workflows
```

Ideal

for enterprise platforms.

---

# Environment Secrets

GitHub Environments

can store

environment-specific secrets.

Example

```text
Development

↓

Dev Credentials

────────────

Production

↓

Prod Credentials
```

This prevents

cross-environment credential reuse.

---

# Variables vs Secrets

| Variables | Secrets |
|------------|---------|
| Plain Configuration | Sensitive Data |
| Visible | Encrypted |
| Application Settings | Credentials |
| Region | Password |
| Cluster Name | API Token |

---

# GitHub Contexts

Contexts provide

dynamic workflow information.

Examples

```text
Repository

Branch

Commit

Actor

Workflow

Runner
```

Contexts

make workflows

dynamic.

---

# Context Architecture

```text
GitHub Event

↓

Context

↓

Workflow

↓

Decision
```

---

# Common Contexts

Frequently used contexts

```text
github

env

runner

job

steps

secrets

vars
```

Each provides

specific information.

---

# GitHub Context

Provides

repository metadata.

Examples

```text
Repository Name

Branch Name

Commit SHA

Actor

Event Name
```

Useful

for deployment logic.

---

# Runner Context

Provides information

about

the runner.

Example

```text
Operating System

Architecture

Temporary Directory
```

---

# Job Context

Contains

job-related information.

Examples

```text
Job Status

Job ID

Outputs
```

---

# Step Context

Provides information

from previous steps.

Example

```text
Step Result

Outputs

Status
```

---

# Expressions

GitHub Actions

supports expressions

for dynamic workflows.

Workflow

```text
Condition

↓

Expression

↓

Decision
```

---

# Conditional Execution

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

One workflow

supports multiple scenarios.

---

# Dynamic Deployment

```text
Branch

↓

Expression

↓

Development

────────────

Main

↓

Production
```

Deployment target

changes automatically.

---

# Enterprise Variable Strategy

```text
Repository Variables

↓

Development

↓

Testing

↓

Production

↓

Workflow
```

Configuration remains centralized.

---

# Secure Authentication

Instead of

storing AWS keys,

modern workflows use

```text
GitHub

↓

OIDC

↓

AWS IAM Role

↓

Temporary Credentials
```

This eliminates

long-lived credentials.

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub

↓

Variables

↓

Secrets

↓

Workflow

↓

Amazon ECR

↓

Amazon EKS
```

Sensitive values

remain protected.

---

# Banking Example

```text
Developer

↓

GitHub

↓

Repository Variables

↓

Environment Secrets

↓

GitHub Actions

↓

Production Deployment
```

Different environments

use

different credentials

without modifying workflows.

---

# Enterprise Best Practices

- Store configuration in variables.
- Store credentials in secrets.
- Use environment-specific secrets.
- Prefer OIDC over long-lived AWS keys.
- Minimize variable scope.
- Never print secrets in logs.
- Use organization secrets for shared platforms.
- Keep workflows environment-independent.

---

# Common Mistakes

- Hardcoding passwords.
- Storing AWS credentials in YAML.
- Printing secrets in workflow logs.
- Using production credentials in development.
- Duplicating variables across repositories.
- Using repository secrets when environment secrets are more appropriate.
- Ignoring least-privilege access.

---

# Interview Questions

## Basic

- What are GitHub Actions Variables?
- What are GitHub Secrets?
- Variables vs Secrets.
- What are Contexts?

## Intermediate

- Explain GitHub Contexts.
- Repository Variables vs Organization Variables.
- Repository Secrets vs Environment Secrets.
- How do expressions work?
- Why should secrets never be hardcoded?

## Advanced

- Design a secure GitHub Actions workflow using variables, environment-specific secrets, GitHub contexts, expressions, and OIDC authentication for deploying applications to Amazon EKS.
- Explain how GitHub Actions variables, secrets, contexts, and expressions work together to build reusable and secure enterprise CI/CD pipelines.
- A large enterprise manages hundreds of repositories deploying applications across Development, Testing, and Production AWS accounts. Explain how you would design a centralized variable strategy, secure secret management, environment isolation, conditional deployments, and governance using GitHub Actions.

---

# Chapter 5 - GitHub Actions Artifacts, Cache & Dependency Management (Enterprise Guide)

Modern CI/CD pipelines generate many files during execution.

Examples

- Build Packages
- Test Reports
- Docker Metadata
- Terraform Plans
- Coverage Reports
- Dependencies

Without proper storage,

every job would rebuild everything,

making pipelines

slow and inefficient.

GitHub Actions provides

- Artifacts
- Cache

to improve

pipeline efficiency.

---

# Build Pipeline

```text
Source Code

↓

Build

↓

Artifacts

↓

Deploy
```

Build outputs

can be reused

by later jobs.

---

# Why Artifacts?

Without Artifacts

```text
Build

↓

Package

↓

Job Ends

↓

Package Lost
```

The next job

must rebuild

everything.

---

With Artifacts

```text
Build

↓

Artifact

↓

Store

↓

Download

↓

Deploy
```

The build output

is reused.

---

# What is an Artifact?

An Artifact is

a file

or collection of files

generated during

workflow execution.

Examples

- JAR Files
- ZIP Packages
- HTML Reports
- Terraform Plans
- Test Results

Artifacts

are available

after workflow completion.

---

# Artifact Lifecycle

```text
Workflow

↓

Build

↓

Upload Artifact

↓

Storage

↓

Download Artifact

↓

Deployment
```

---

# Artifact Workflow

```text
Build Job

↓

Artifact

↓

Test Job

↓

Deploy Job
```

Multiple jobs

share

the same build output.

---

# Common Artifact Examples

Artifacts commonly include

```text
Application Package

Docker Metadata

Terraform Plan

Coverage Report

JUnit Report
```

---

# Artifact Storage

GitHub

stores artifacts

temporarily.

Workflow

```text
Runner

↓

Upload

↓

GitHub Storage

↓

Download
```

Artifacts

can be downloaded

from workflow history.

---

# Artifact Retention

Artifacts

remain available

for

a configurable period.

Example

```text
Build

↓

Artifact

↓

Retention Period

↓

Automatic Cleanup
```

Older artifacts

are removed automatically.

---

# Enterprise Artifact Flow

```text
Developer

↓

Build

↓

Artifact

↓

Security Scan

↓

Deploy
```

The same build

moves

through

every stage.

---

# What is Cache?

Cache stores

dependencies

that rarely change.

Examples

```text
Maven Dependencies

NPM Packages

Python Libraries

Gradle Cache

Terraform Plugins
```

Unlike artifacts,

cache

is intended

to speed up

future workflow executions.

---

# Cache Workflow

```text
Dependencies

↓

Cache

↓

Next Build

↓

Reuse
```

Downloads

are avoided.

---

# Why Cache?

Without Cache

```text
Workflow

↓

Download Dependencies

↓

Build

↓

Finish
```

Every execution

downloads

the same packages.

---

With Cache

```text
Workflow

↓

Cache Found

↓

Reuse Dependencies

↓

Fast Build
```

Pipeline execution

becomes much faster.

---

# Cache Lifecycle

```text
First Build

↓

Download Dependencies

↓

Save Cache

────────────

Second Build

↓

Restore Cache

↓

Build
```

---

# Cache vs Artifact

| Cache | Artifact |
|--------|----------|
| Speeds Future Builds | Shares Build Output |
| Dependencies | Build Results |
| Reused Across Runs | Used Within Pipeline |
| Performance Optimization | Deployment Asset |

---

# Cache Examples

Common cache directories

```text
Maven

↓

Repository

────────────

Node.js

↓

node_modules

────────────

Python

↓

pip Cache
```

---

# Docker Layer Cache

Docker builds

also benefit

from caching.

Workflow

```text
Base Image

↓

Dependencies

↓

Application

↓

Reuse Layers
```

Only changed layers

are rebuilt.

---

# Terraform Cache

Terraform downloads

providers

and modules.

Caching them

reduces

pipeline execution time.

---

# Multi-Job Workflow

```text
Build

↓

Upload Artifact

↓

Security Scan

↓

Download Artifact

↓

Deploy
```

No rebuilding

is required.

---

# CI/CD Optimization

```text
Cache

↓

Dependencies

↓

Artifacts

↓

Deployment
```

Both mechanisms

work together

to optimize pipelines.

---

# Enterprise Architecture

```text
Developer

↓

GitHub Actions

↓

Build

↓

Cache

↓

Artifact

↓

Security Scan

↓

Amazon ECR

↓

Amazon EKS
```

---

# Banking Example

```text
Payment Service

↓

Build

↓

Artifact

↓

Security Scan

↓

Approval

↓

Production
```

The exact same build

is promoted

to production.

---

# Artifact Promotion

Best practice

```text
Build Once

↓

Artifact

↓

Development

↓

Testing

↓

Production
```

Never rebuild

between environments.

---

# Enterprise Pipeline

```text
Build

↓

Artifact

↓

Quality Checks

↓

Security Scan

↓

Deployment
```

Every stage

uses

the same artifact.

---

# Performance Optimization

Use cache for

- Dependencies
- Package Managers
- Terraform Plugins
- Docker Layers

Use artifacts for

- Build Packages
- Reports
- Deployment Files

---

# Best Practices

- Build once, deploy everywhere.
- Use artifacts for deployment packages.
- Use cache for dependencies.
- Keep artifacts versioned.
- Configure retention policies.
- Avoid caching unnecessary files.
- Reuse Docker layer cache.
- Download artifacts instead of rebuilding.

---

# Common Mistakes

- Using artifacts as dependency cache.
- Rebuilding applications in every job.
- Caching temporary files.
- Keeping artifacts forever.
- Uploading unnecessary large artifacts.
- Ignoring cache invalidation.
- Mixing artifacts from different builds.

---

# Interview Questions

## Basic

- What are GitHub Actions Artifacts?
- What is GitHub Actions Cache?
- Cache vs Artifact.
- Why do we use artifacts?

## Intermediate

- How do artifacts move between jobs?
- Why is dependency caching important?
- Explain Docker layer caching in CI/CD.
- How do Terraform providers benefit from caching?
- What is artifact retention?

## Advanced

- Design an enterprise GitHub Actions pipeline using artifacts, dependency caching, Docker layer caching, Terraform plugin caching, Amazon ECR, and Amazon EKS to minimize build time while ensuring consistent deployments.
- Explain the difference between artifacts and caches, and describe how both contribute to scalable and efficient CI/CD pipelines.
- A large enterprise builds hundreds of Java, Node.js, and Python applications daily. Explain how you would design artifact management, dependency caching, retention policies, build promotion, and pipeline optimization to reduce execution time while maintaining deployment consistency and traceability.


---

# Chapter 6 - GitHub Actions Reusable Workflows, Composite Actions & Workflow Reuse (Enterprise Guide)

As organizations grow,

they often manage

- Hundreds of Repositories
- Thousands of Workflow Files
- Multiple Development Teams

Copying the same workflow across repositories leads to

- Duplicate Code
- Maintenance Issues
- Configuration Drift

GitHub Actions solves this using

- Reusable Workflows
- Composite Actions
- Reusable Actions

These features help standardize CI/CD across the enterprise.

---

# Why Workflow Reuse?

Without reuse

```text
Repository A

↓

Build Workflow

────────────

Repository B

↓

Copy Workflow

────────────

Repository C

↓

Copy Workflow
```

Problems

- Duplicate YAML
- Difficult Updates
- Inconsistent Pipelines

---

With reuse

```text
Reusable Workflow

↓

Repository A

↓

Repository B

↓

Repository C
```

One update

benefits

every repository.

---

# Workflow Reuse Architecture

```text
Reusable Workflow

↓

Repository

↓

Job

↓

Deployment
```

Reusable workflows

act like

shared CI/CD templates.

---

# What is a Reusable Workflow?

A reusable workflow

is a workflow

that can be called

by another workflow.

Architecture

```text
Caller Workflow

↓

Reusable Workflow

↓

Jobs Execute
```

This avoids

duplicating YAML code.

---

# Reusable Workflow Flow

```text
Developer Push

↓

Main Workflow

↓

Reusable Workflow

↓

Build

↓

Test

↓

Deploy
```

---

# Enterprise Workflow Structure

```text
.github/

└── workflows/

    ├── build.yml

    ├── test.yml

    ├── deploy.yml

    └── reusable-build.yml
```

Repositories

share

common workflows.

---

# Workflow Inputs

Reusable workflows

accept

inputs.

Examples

```text
Application Name

Docker Repository

AWS Region

Environment
```

One workflow

supports

multiple projects.

---

# Workflow Outputs

Reusable workflows

can return

outputs.

Example

```text
Docker Image

↓

Image Tag

↓

Deployment Workflow
```

Outputs

flow

between workflows.

---

# Secrets in Reusable Workflows

Secrets

can be passed

securely.

Workflow

```text
Repository Secret

↓

Reusable Workflow

↓

Deployment
```

Sensitive values

remain protected.

---

# Workflow Call Hierarchy

```text
Main Workflow

↓

Reusable Build

↓

Reusable Test

↓

Reusable Deploy
```

Large organizations

split pipelines

into reusable components.

---

# What are Composite Actions?

Composite Actions

combine

multiple workflow steps

into

one reusable action.

Architecture

```text
Composite Action

↓

Step 1

↓

Step 2

↓

Step 3
```

---

# Composite Action vs Reusable Workflow

| Composite Action | Reusable Workflow |
|------------------|-------------------|
| Reuses Steps | Reuses Entire Workflow |
| Inside Job | Entire Workflow |
| Smaller Unit | Larger Unit |
| Task Automation | Pipeline Automation |

---

# When to Use Composite Actions

Examples

```text
Login to AWS

↓

Configure CLI

↓

Validate Environment
```

Instead of repeating

these steps

in every workflow,

create

one composite action.

---

# Action Marketplace

GitHub provides

a Marketplace

containing

thousands of reusable actions.

Examples

- Checkout Repository
- Setup Java
- Setup Node.js
- Upload Artifact
- AWS Login
- Docker Build

---

# Internal Enterprise Actions

Large organizations

often maintain

private actions.

Architecture

```text
Platform Team

↓

Reusable Action

↓

Development Teams
```

This standardizes

CI/CD.

---

# Workflow Modularity

Instead of

one large workflow

split responsibilities.

```text
Build

↓

Test

↓

Security

↓

Deploy
```

Each component

is reusable.

---

# Enterprise Repository Strategy

```text
Platform Repository

↓

Reusable Workflows

↓

Reusable Actions

────────────

Application Repositories

↓

Consume Shared Workflows
```

Platform teams

maintain

shared automation.

---

# CI/CD Standardization

```text
Platform Team

↓

Reusable Workflow

↓

Every Repository

↓

Consistent Pipeline
```

Every application

uses

the same quality standards.

---

# Enterprise Deployment Flow

```text
Developer

↓

Repository

↓

Reusable Build

↓

Reusable Test

↓

Reusable Security Scan

↓

Reusable Deploy

↓

Amazon EKS
```

---

# Banking Example

```text
Payment API

↓

Reusable Build

↓

Reusable Security Scan

↓

Reusable Deployment

↓

Production
```

Every banking service

uses

the same deployment process.

---

# Multi-Repository Architecture

```text
Platform Repository

├── Build Workflow

├── Docker Workflow

├── Terraform Workflow

├── Kubernetes Workflow

└── Security Workflow

↓

Application Repositories

↓

Reuse Everything
```

This minimizes

maintenance effort.

---

# Workflow Versioning

Reusable workflows

should be versioned.

Example

```text
Build Workflow

↓

v1

↓

v2

↓

v3
```

Repositories

can upgrade

when ready.

---

# Governance

Platform teams

control

approved workflows.

Benefits

- Security
- Standardization
- Compliance
- Easier Maintenance

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub Repository

↓

Reusable Workflow

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

One standardized pipeline

serves

all applications.

---

# Best Practices

- Create reusable workflows for common pipelines.
- Use composite actions for repeated tasks.
- Keep reusable workflows modular.
- Version reusable workflows.
- Store shared automation in a central repository.
- Pass secrets securely.
- Document workflow inputs and outputs.
- Avoid workflow duplication.

---

# Common Mistakes

- Copying workflows across repositories.
- Creating overly large reusable workflows.
- Hardcoding application-specific values.
- Ignoring workflow versioning.
- Mixing reusable and project-specific logic.
- Duplicating AWS authentication steps.
- Not documenting reusable workflows.

---

# Interview Questions

## Basic

- What is a reusable workflow?
- What is a composite action?
- Why do we reuse workflows?
- Composite Action vs Reusable Workflow.

## Intermediate

- How do reusable workflows receive inputs?
- How do workflows return outputs?
- Why should reusable workflows be versioned?
- What is GitHub Marketplace?
- Explain workflow standardization.

## Advanced

- Design an enterprise GitHub Actions platform using reusable workflows, composite actions, centralized CI/CD templates, GitHub Marketplace actions, and Amazon EKS deployments.
- Explain how reusable workflows and composite actions reduce duplication, improve governance, and standardize CI/CD across hundreds of repositories.
- A large enterprise has 500 application repositories maintained by different teams. Explain how you would design a centralized GitHub Actions platform with reusable workflows, composite actions, versioning, governance, security controls, and deployment automation to ensure consistency and maintainability across the organization.

---

# Chapter 7 - GitHub Actions Docker Build, Amazon ECR & Amazon EKS Deployment (Enterprise Guide)

Modern cloud-native applications follow a standardized deployment pipeline.

Instead of manually building and deploying applications,

GitHub Actions automates

- Docker Image Build
- Security Scanning
- Image Push
- Infrastructure Validation
- Kubernetes Deployment

This chapter explains how GitHub Actions integrates with Docker, Amazon ECR, and Amazon EKS in enterprise environments.

---

# Enterprise Deployment Architecture

```text
Developer

↓

GitHub Repository

↓

GitHub Actions

↓

Docker Build

↓

Security Scan

↓

Amazon ECR

↓

Amazon EKS

↓

Pods

↓

Customers
```

This is one of the most common enterprise CI/CD architectures.

---

# CI/CD Deployment Flow

```text
Git Push

↓

Build

↓

Unit Tests

↓

Docker Image

↓

Security Scan

↓

Amazon ECR

↓

Deployment

↓

Amazon EKS
```

Every deployment follows

the same automated workflow.

---

# Why Automate Deployments?

Manual deployments

often result in

- Human Errors
- Inconsistent Releases
- Downtime
- Slow Delivery

Automation provides

- Consistency
- Reliability
- Faster Releases
- Repeatability

---

# Docker Build Stage

The first deployment stage

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

The image becomes

the deployment artifact.

---

# Image Tagging Strategy

Every build

should generate

a unique image tag.

Examples

```text
v1.0.0

v2.1.3

Commit SHA

Build Number
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
- Amazon ECR Scan
- Docker Scout

Only approved images

should continue

through the pipeline.

---

# Amazon Elastic Container Registry (ECR)

Amazon ECR

stores

Docker images.

Architecture

```text
Docker Image

↓

Amazon ECR

↓

Versioned Repository
```

The registry

becomes

the source of truth

for deployments.

---

# Image Push Workflow

```text
GitHub Actions

↓

Docker Build

↓

Security Scan

↓

Push Image

↓

Amazon ECR
```

The image

is now available

for deployment.

---

# Build Once, Deploy Everywhere

Enterprise pipelines

build

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

After the image

is stored

in Amazon ECR,

GitHub Actions

deploys it

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

GitHub Actions

updates

the Kubernetes deployment.

Architecture

```text
Deployment

↓

ReplicaSet

↓

Pods
```

The cluster

pulls

the latest approved image.

---

# Rolling Deployment

Production deployments

should use

rolling updates.

```text
Old Pods

↓

New Pods

↓

Validation

↓

Traffic Shift
```

Downtime

is minimized.

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

Switch Traffic
```

Rollback

is immediate

if issues occur.

---

# Canary Deployment

Traffic

is shifted gradually.

```text
10%

↓

25%

↓

50%

↓

100%
```

Production risk

is reduced.

---

# Deployment Approval

Many organizations

require

manual approval

before production.

Workflow

```text
Security Scan

↓

Approval

↓

Production Deployment
```

---

# Environment Strategy

One workflow

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

Each environment

uses

its own configuration.

---

# Deployment Pipeline

```text
Developer

↓

Git Push

↓

GitHub Actions

↓

Build

↓

Docker Image

↓

Amazon ECR

↓

Amazon EKS
```

Fully automated

from commit

to deployment.

---

# Rollback Strategy

If deployment fails

```text
Current Version

↓

Previous Image

↓

Redeploy

↓

Application Restored
```

Rollback

should be fast

and predictable.

---

# Deployment Verification

After deployment

verify

- Pod Status
- Health Checks
- Application Logs
- Metrics
- Alerts

Deployment

is complete

only after validation.

---

# Monitoring

After deployment

monitor

- Pod Health
- CPU
- Memory
- Restart Count
- Latency

Use

- Prometheus
- Grafana
- ELK

for observability.

---

# Enterprise Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

Amazon EKS

↓

Prometheus

↓

Grafana

↓

ELK
```

This provides

secure,

observable,

and automated deployments.

---

# Banking Example

```text
Developer

↓

Payment API

↓

GitHub Actions

↓

Docker Build

↓

Image Scan

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

quality and security gates.

---

# Enterprise Deployment Checklist

Before deployment verify

✓ Build Successful

✓ Unit Tests Passed

✓ Security Scan Passed

✓ Docker Image Tagged

✓ Image Uploaded to Amazon ECR

✓ Kubernetes Manifest Validated

✓ Approval Completed

✓ Monitoring Enabled

✓ Rollback Plan Ready

---

# Best Practices

- Build once, deploy everywhere.
- Use immutable Docker images.
- Tag images with versions or commit SHAs.
- Scan images before deployment.
- Use rolling or blue-green deployments.
- Keep production deployments behind approvals.
- Validate deployments after rollout.
- Monitor applications continuously.

---

# Common Mistakes

- Deploying unscanned images.
- Using `latest` in production.
- Rebuilding images for each environment.
- Skipping deployment validation.
- Ignoring rollback planning.
- Deploying directly to production from feature branches.
- Not monitoring deployments after release.

---

# Interview Questions

## Basic

- How does GitHub Actions deploy Docker applications?
- What is Amazon ECR?
- What is Amazon EKS?
- Why is image tagging important?
- Why should deployments be automated?

## Intermediate

- Explain the deployment pipeline from GitHub to Amazon EKS.
- Rolling Deployment vs Blue-Green Deployment.
- What is Canary Deployment?
- Why Build Once, Deploy Everywhere?
- How do you validate a deployment?

## Advanced

- Design a complete GitHub Actions deployment pipeline integrating Docker, Trivy, Amazon ECR, Amazon EKS, Prometheus, Grafana, and ELK Stack.
- Explain how immutable Docker images, automated security scanning, deployment approvals, and Kubernetes rollout strategies work together to deliver secure and reliable production deployments.
- A financial organization deploys hundreds of microservices to Amazon EKS every day. Explain how you would design the CI/CD pipeline, image management strategy, deployment approval process, rollback mechanism, monitoring, and governance to ensure high availability, security, and compliance.

---

# Chapter 8 - GitHub Actions Security, OIDC Authentication & Enterprise Best Practices

CI/CD pipelines have access to

- Source Code
- Docker Images
- Cloud Infrastructure
- Secrets
- Production Environments

If a pipeline is compromised,

an attacker may gain access to the entire software delivery process.

Enterprise GitHub Actions pipelines are designed using

- Least Privilege
- Short-lived Credentials
- Identity Federation
- Security Scanning
- Environment Protection
- Deployment Approvals

Security must be enforced throughout the CI/CD lifecycle.

---

# CI/CD Security Lifecycle

```text
Developer

↓

GitHub Repository

↓

Workflow

↓

Authentication

↓

Security Scan

↓

Deployment Approval

↓

Production
```

Security controls

exist at every stage.

---

# Why GitHub Actions Security?

Without security

```text
Workflow

↓

Long-lived Credentials

↓

Repository

↓

Production
```

Risks include

- Credential Theft
- Unauthorized Deployments
- Infrastructure Compromise

---

With security

```text
GitHub Actions

↓

OIDC

↓

IAM Role

↓

Temporary Credentials

↓

AWS
```

No permanent credentials

are stored.

---

# Principle of Least Privilege

Every workflow

should receive

only the permissions

required to complete its task.

Avoid

```text
Administrator Access

Full Repository Access

Production Write Access
```

Grant

only

minimum required permissions.

---

# GitHub Token Permissions

Every workflow

receives

a GitHub token.

Restrict its permissions.

Example

```text
Read Repository

↓

Build

↓

No Admin Access
```

Avoid giving

unnecessary write permissions.

---

# Repository Security

Protect repositories using

- Branch Protection
- Pull Request Reviews
- Required Status Checks
- Signed Commits (if applicable)
- Protected Branches

Unauthorized code

must never

reach production.

---

# Branch Protection Workflow

```text
Developer

↓

Pull Request

↓

Code Review

↓

Pipeline

↓

Merge

↓

Deployment
```

Every change

is validated.

---

# GitHub Secrets

Store sensitive values

using GitHub Secrets.

Examples

```text
API Keys

Database Passwords

Tokens

Certificates
```

Never hardcode

credentials

inside workflows.

---

# Why OIDC?

Traditionally

AWS authentication

required

long-lived access keys.

Architecture

```text
GitHub Actions

↓

AWS Access Key

↓

AWS
```

Problems

- Credential Rotation
- Secret Leakage
- Long-lived Credentials

---

# OIDC Authentication

OpenID Connect (OIDC)

allows GitHub Actions

to authenticate

without storing AWS keys.

Architecture

```text
GitHub Actions

↓

OIDC Token

↓

AWS IAM

↓

Temporary Credentials

↓

Deployment
```

This is the recommended approach.

---

# OIDC Workflow

```text
Workflow Starts

↓

GitHub Issues OIDC Token

↓

AWS Verifies Identity

↓

IAM Role Assumed

↓

Temporary Credentials

↓

Deployment
```

---

# Benefits of OIDC

- No Long-lived Credentials
- Automatic Credential Rotation
- Improved Security
- Better Auditability
- IAM Integration

---

# IAM Role for GitHub Actions

Instead of

creating IAM users,

create

IAM Roles.

Architecture

```text
GitHub Actions

↓

OIDC

↓

IAM Role

↓

AWS Resources
```

The workflow

assumes

the required role.

---

# Trust Relationship

AWS verifies

that

only approved repositories

can assume

the IAM role.

Architecture

```text
GitHub Repository

↓

OIDC

↓

IAM Trust Policy

↓

IAM Role
```

This prevents

unauthorized access.

---

# Environment Protection

GitHub Environments

can enforce

- Manual Approval
- Deployment Restrictions
- Environment Secrets

Example

```text
Development

↓

Automatic

────────────

Production

↓

Manual Approval
```

---

# Deployment Approval

Production deployments

should require

manual approval.

Workflow

```text
Security Scan

↓

Approval

↓

Production Deployment
```

This reduces

deployment risk.

---

# Security Scanning

CI/CD pipelines

should automatically run

- SonarQube
- Trivy
- Dependency Scanning
- Secret Scanning

Only secure builds

should proceed.

---

# Secret Scanning

Prevent accidental commits of

- AWS Keys
- API Tokens
- Passwords
- Private Certificates

Repositories

should reject

committed secrets.

---

# Dependency Scanning

Libraries

may contain

known vulnerabilities.

Automatically scan

dependencies

before deployment.

---

# Container Security

Pipeline should verify

- Dockerfile
- Base Image
- Image Vulnerabilities
- Image Size

before pushing images

to Amazon ECR.

---

# Workflow Permissions

Each workflow

should define

explicit permissions.

Avoid

granting

repository-wide write access

unless required.

---

# Runner Security

Protect runners using

- Dedicated Runners for Production
- Network Isolation
- Least Privilege
- Automatic Cleanup

Self-hosted runners

should never

be shared

between production

and development.

---

# Audit Logging

Track

- Workflow Executions
- Deployments
- Approvals
- Failed Logins
- Secret Usage

Audit logs

support

security investigations.

---

# Enterprise Security Pipeline

```text
Developer

↓

Pull Request

↓

Code Review

↓

GitHub Actions

↓

SonarQube

↓

Trivy

↓

OIDC Authentication

↓

Amazon ECR

↓

Amazon EKS

↓

Monitoring
```

---

# Banking Example

```text
Developer

↓

GitHub

↓

Protected Branch

↓

GitHub Actions

↓

OIDC

↓

IAM Role

↓

Amazon EKS

↓

Production
```

No AWS access keys

are stored

inside GitHub.

---

# Enterprise Architecture

```text
GitHub

↓

Workflow

↓

OIDC

↓

IAM Role

↓

Docker Build

↓

Amazon ECR

↓

Amazon EKS

↓

Prometheus

↓

Grafana
```

Every deployment

uses

temporary credentials.

---

# Security Checklist

Before production deployment verify

✓ Protected Branches Enabled

✓ Pull Request Reviews Required

✓ Workflow Permissions Restricted

✓ GitHub Secrets Configured

✓ OIDC Authentication Enabled

✓ IAM Roles Configured

✓ Security Scan Passed

✓ Environment Approval Enabled

✓ Runner Security Reviewed

✓ Audit Logging Enabled

---

# Enterprise Best Practices

- Prefer OIDC over AWS access keys.
- Use IAM roles instead of IAM users.
- Restrict GitHub token permissions.
- Protect production branches.
- Require pull request approvals.
- Use environment protection rules.
- Scan code and containers automatically.
- Separate production and development runners.
- Rotate secrets regularly if OIDC cannot be used.
- Continuously audit workflow activity.

---

# Common Mistakes

- Storing AWS credentials in GitHub Secrets when OIDC is available.
- Giving workflows administrator permissions.
- Allowing direct commits to the main branch.
- Skipping security scans.
- Using the same secrets across all environments.
- Running production deployments without approval.
- Sharing self-hosted runners between development and production.
- Ignoring audit logs.

---

# Interview Questions

## Basic

- Why is GitHub Actions security important?
- What are GitHub Secrets?
- What is OIDC?
- Why should AWS access keys be avoided?

## Intermediate

- Explain OIDC authentication.
- IAM User vs IAM Role.
- What are environment protection rules?
- Why should workflows use least privilege?
- How do protected branches improve CI/CD security?

## Advanced

- Design a secure GitHub Actions platform using OIDC, IAM roles, protected branches, environment approvals, SonarQube, Trivy, Amazon ECR, and Amazon EKS.
- Explain how OIDC authentication replaces long-lived AWS credentials and improves the security of enterprise CI/CD pipelines.
- A financial organization requires a highly secure CI/CD platform with strict compliance requirements. Explain how you would design authentication, authorization, workflow permissions, environment protection, runner isolation, security scanning, deployment approvals, and audit logging to ensure secure and compliant software delivery.

---

# Chapter 9 - GitHub Actions Production Troubleshooting (50+ Enterprise Scenarios)

GitHub Actions pipelines automate

- Builds
- Testing
- Security Scanning
- Docker Image Creation
- Infrastructure Provisioning
- Kubernetes Deployment

When pipelines fail,

the impact can include

- Failed Releases
- Production Outages
- Infrastructure Drift
- Deployment Delays

A Senior DevOps Engineer should follow

a structured troubleshooting methodology

instead of rerunning pipelines repeatedly.

---

# Enterprise Troubleshooting Framework

Always investigate in this order.

```text
Alert

↓

Understand Business Impact

↓

Review Workflow Run

↓

Check Trigger

↓

Review Logs

↓

Verify Runner

↓

Verify Authentication

↓

Check Dependencies

↓

Check Deployment

↓

Root Cause

↓

Fix

↓

Validate

↓

Postmortem
```

---

# Scenario 1 - Workflow Not Triggered

## Symptoms

```text
Git Push

↓

No Workflow
```

---

## Investigation

Verify

- Workflow Location
- Trigger Configuration
- Branch
- Repository Settings

---

## Resolution

Confirm

workflow exists

under

`.github/workflows`

and trigger conditions match the event.

---

# Scenario 2 - Workflow Syntax Error

## Symptoms

Workflow fails immediately.

---

## Investigation

Check

- YAML Syntax
- Indentation
- Missing Keys

---

## Resolution

Validate

workflow YAML

before committing.

---

# Scenario 3 - Runner Not Available

Possible Causes

- Hosted Runner Issue
- Self-hosted Runner Offline
- Incorrect Runner Labels

---

## Resolution

Verify

runner availability

and labels.

---

# Scenario 4 - Job Stuck in Queue

Check

- Runner Capacity
- Organization Limits
- Concurrency Settings

---

# Scenario 5 - Checkout Failed

Verify

- Repository Permissions
- GitHub Token Permissions
- Repository Availability

---

# Scenario 6 - Secret Not Available

Check

- Repository Secret
- Environment Secret
- Organization Secret

Verify

workflow access.

---

# Scenario 7 - Variable Missing

Review

- Repository Variables
- Environment Variables
- Workflow Scope

---

# Scenario 8 - OIDC Authentication Failed

Check

- IAM Role
- Trust Policy
- OIDC Provider
- Repository Configuration

---

# Scenario 9 - AWS Authentication Failed

Verify

- IAM Role
- AWS Region
- Temporary Credentials

---

# Scenario 10 - Docker Build Failed

Review

- Dockerfile
- Build Context
- Base Image
- Dependencies

---

# Scenario 11 - Docker Push Failed

Verify

- Amazon ECR Login
- Repository Exists
- IAM Permissions

---

# Scenario 12 - Image Scan Failed

Investigate

- Critical CVEs
- Base Image
- Outdated Packages

Rebuild

using updated dependencies.

---

# Scenario 13 - Amazon EKS Deployment Failed

Check

- Cluster Access
- IAM Permissions
- Kubernetes Manifest
- Image Availability

---

# Scenario 14 - Terraform Apply Failed

Review

- Backend
- State Lock
- Variables
- Provider Versions

---

# Scenario 15 - Artifact Upload Failed

Verify

- File Exists
- Storage Limits
- Artifact Path

---

# Scenario 16 - Artifact Download Failed

Check

- Artifact Name
- Previous Job
- Workflow Dependencies

---

# Scenario 17 - Cache Not Restored

Review

- Cache Key
- Dependency Changes
- Cache Scope

---

# Scenario 18 - Pipeline Slow

Investigate

- Cache Usage
- Parallel Jobs
- Runner Performance

---

# Scenario 19 - Job Timeout

Check

- Infinite Loops
- Long-running Scripts
- Timeout Configuration

---

# Scenario 20 - Workflow Cancelled

Verify

- Manual Cancellation
- Concurrency Rules
- Repository Limits

---

# Scenario 21 - Pull Request Checks Failed

Review

- Unit Tests
- Linting
- Security Scans
- Required Status Checks

---

# Scenario 22 - Branch Protection Blocks Merge

Check

- Required Reviews
- Required Checks
- Branch Protection Rules

---

# Scenario 23 - Deployment Approval Pending

Verify

- GitHub Environment Rules
- Required Reviewers

---

# Scenario 24 - Self-hosted Runner Offline

Review

- Runner Service
- Network
- GitHub Connectivity

---

# Scenario 25 - Docker Image Uses Wrong Tag

Check

- Workflow Variables
- Image Tag Strategy
- Deployment Manifest

---

# Scenario 26 - Kubernetes Pulls Old Image

Verify

- Image Tag
- Deployment Manifest
- Image Pull Policy

---

# Scenario 27 - SonarQube Scan Failed

Review

- Connectivity
- Authentication
- Project Configuration

---

# Scenario 28 - Trivy Scan Failed

Investigate

- Vulnerability Database
- Image Availability
- Scanner Configuration

---

# Scenario 29 - GitHub API Rate Limit

Check

- API Usage
- Authentication
- Retry Logic

---

# Scenario 30 - Matrix Job Failed

Review

- Matrix Configuration
- Platform-specific Errors

---

# Scenario 31 - Environment Variables Incorrect

Verify

- Workflow Scope
- Job Scope
- Step Scope

---

# Scenario 32 - Workflow Executes on Wrong Branch

Review

trigger configuration.

---

# Scenario 33 - Production Deployment Triggered Accidentally

Investigate

- Branch Protection
- Environment Rules
- Conditional Logic

---

# Scenario 34 - Multiple Deployments Running Simultaneously

Review

- Concurrency Configuration
- Environment Locks

---

# Scenario 35 - Runner Disk Full

Clean

- Docker Images
- Build Cache
- Temporary Files
- Artifacts

---

# Scenario 36 - Memory Exhausted

Review

- Build Process
- Runner Size
- Application Build

---

# Scenario 37 - Workflow Uses Old Action Version

Verify

action versions.

Keep

reusable actions updated.

---

# Scenario 38 - Permission Denied

Check

- Repository Permissions
- GitHub Token
- IAM Policies

---

# Scenario 39 - Deployment Rollback Required

Recovery

```text
Previous Image

↓

Redeploy

↓

Validate

↓

Production Restored
```

---

# Scenario 40 - Notification Not Sent

Review

- Notification Step
- Integration Configuration
- Secrets

---

# Scenario 41 - Scheduled Workflow Did Not Run

Verify

- Schedule
- Repository Activity
- Workflow Configuration

---

# Scenario 42 - Manual Workflow Cannot Start

Check

- Repository Permissions
- Workflow Configuration
- User Permissions

---

# Scenario 43 - Reusable Workflow Failed

Review

- Workflow Version
- Inputs
- Outputs
- Secrets

---

# Scenario 44 - Composite Action Failed

Verify

- Action Structure
- Required Inputs
- Repository Access

---

# Scenario 45 - Deployment Succeeded but Application Failed

Check

- Pod Health
- Application Logs
- Readiness Checks
- Configuration

---

# Scenario 46 - Wrong Environment Deployed

Verify

- Branch Mapping
- Environment Variables
- Conditional Logic

---

# Scenario 47 - Security Policy Blocks Deployment

Investigate

- Scan Results
- Compliance Rules
- Approval Requirements

---

# Scenario 48 - Pipeline Succeeds but Infrastructure Drift Exists

Run

Terraform Plan

before deployment.

---

# Scenario 49 - Complete Pipeline Failure

Recovery

```text
Previous Stable Pipeline

↓

Previous Image

↓

Rollback

↓

Production Restored
```

---

# Scenario 50 - Disaster Recovery

Recovery Plan

```text
GitHub Repository

↓

Workflow

↓

Docker Build

↓

Amazon ECR

↓

Terraform

↓

Amazon EKS

↓

Production Restored
```

Infrastructure

should always

be reproducible.

---

# Enterprise Troubleshooting Checklist

Always verify

✓ Workflow Trigger

✓ YAML Syntax

✓ Runner

✓ Variables

✓ Secrets

✓ OIDC Authentication

✓ Docker Build

✓ Image Scan

✓ Amazon ECR

✓ Amazon EKS

✓ Terraform

✓ Deployment

✓ Monitoring

---

# Incident Response Workflow

```text
Alert

↓

Workflow Logs

↓

Runner

↓

Authentication

↓

Deployment

↓

Application

↓

Root Cause

↓

Fix

↓

Validation

↓

Postmortem
```

---

# Best Practices

- Read workflow logs before rerunning pipelines.
- Use OIDC instead of long-lived credentials.
- Version reusable workflows.
- Validate Docker images before deployment.
- Protect production environments with approvals.
- Keep runners updated.
- Use rollback strategies for production failures.
- Document root cause analyses (RCA).

---

# Common Mistakes

- Rerunning failed workflows without investigating.
- Ignoring workflow logs.
- Using production credentials in development.
- Hardcoding secrets.
- Deploying directly from feature branches.
- Running outdated reusable workflows.
- Skipping deployment validation.
- Ignoring failed security scans.

---

# Interview Questions

## Basic

- How do you troubleshoot a failed GitHub Actions workflow?
- Why might a workflow not trigger?
- What causes runner failures?

## Intermediate

- How do you troubleshoot OIDC authentication failures?
- What causes Docker push failures?
- How do you investigate a failed Amazon EKS deployment?
- Explain cache-related pipeline failures.
- How do you troubleshoot reusable workflows?

## Advanced

- Design a production troubleshooting runbook for GitHub Actions pipelines covering runners, authentication, Docker, Terraform, Amazon ECR, Amazon EKS, and deployment validation.
- Explain your end-to-end troubleshooting methodology when a production deployment fails after a successful GitHub Actions workflow.
- A financial organization's GitHub Actions pipeline successfully builds and scans a Docker image but fails during production deployment to Amazon EKS. Explain how you would investigate the workflow, authentication, runner, image, Kubernetes deployment, monitoring, rollback strategy, and preventive improvements.

---

# Chapter 10 - GitHub Actions Enterprise Best Practices

GitHub Actions makes CI/CD automation simple.

However,

building an **enterprise-grade CI/CD platform** requires much more than writing workflow YAML files.

Enterprise pipelines must be

- Secure
- Reliable
- Scalable
- Auditable
- Maintainable

Large organizations establish standards so that every repository follows the same deployment process.

---

# Enterprise CI/CD Architecture

```text
Developer

↓

GitHub

↓

Pull Request

↓

GitHub Actions

↓

Build

↓

Unit Tests

↓

Security Scan

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

Every deployment

passes through

quality gates.

---

# Treat Pipelines as Code

GitHub Actions workflows

should be treated exactly like

application code.

Store workflows in Git.

Use

- Pull Requests
- Code Reviews
- Branch Protection
- Version Control

Never edit workflows directly

in production.

---

# Modular Workflow Design

Avoid one large workflow

containing everything.

Recommended

```text
Build

↓

Test

↓

Security

↓

Docker

↓

Terraform

↓

Deploy
```

Each workflow

should have

one responsibility.

---

# Reusable Workflows

Shared logic

should exist

only once.

Architecture

```text
Reusable Workflow

↓

Application A

↓

Application B

↓

Application C
```

Platform teams

maintain

shared workflows.

---

# Composite Actions

Repeated tasks

should become

composite actions.

Examples

```text
AWS Login

↓

Docker Login

↓

Common Validation
```

Avoid copying

the same steps

across repositories.

---

# Repository Structure

```text
project/

├── src/

├── Dockerfile

├── terraform/

├── k8s/

└── .github/

    └── workflows/

        ├── build.yml

        ├── test.yml

        ├── deploy.yml

        └── reusable.yml
```

Keep workflows

organized

and easy to maintain.

---

# Build Once, Deploy Everywhere

Build

one immutable artifact.

```text
Build

↓

Artifact

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

between environments.

---

# Secure Authentication

Preferred authentication

```text
GitHub Actions

↓

OIDC

↓

IAM Role

↓

Temporary Credentials

↓

AWS
```

Avoid

long-lived

AWS access keys.

---

# Least Privilege

Grant workflows

only

the permissions

they require.

Avoid

```text
Administrator

Repository Write

Production Access
```

unless absolutely necessary.

---

# Protected Branches

Protect

production branches

using

- Pull Requests
- Code Reviews
- Required Checks
- Merge Restrictions

No direct commits

to production branches.

---

# Environment Protection

GitHub Environments

should enforce

- Manual Approval
- Required Reviewers
- Environment Secrets
- Deployment Restrictions

Production

must never deploy

without approval.

---

# Secrets Management

Store sensitive values

using

- GitHub Secrets
- GitHub Environment Secrets
- Organization Secrets

Never store

credentials

inside

workflow YAML.

---

# Dependency Caching

Cache

- Maven
- Gradle
- npm
- pip
- Terraform Providers

Benefits

- Faster Builds
- Lower Network Usage
- Reduced Pipeline Time

---

# Artifact Management

Artifacts should contain

- Build Output
- Test Reports
- Terraform Plans
- Deployment Packages

Configure

retention policies

to avoid unnecessary storage.

---

# Version Everything

Version

- Workflows
- Composite Actions
- Docker Images
- Terraform Modules

Versioning simplifies

rollback

and maintenance.

---

# Security Scanning

Every Pull Request

should include

- SonarQube
- Trivy
- Dependency Scanning
- Secret Scanning

Deployments

should stop

if critical issues

are detected.

---

# Separate CI and CD

Continuous Integration

```text
Build

↓

Test

↓

Security Scan
```

Continuous Delivery

```text
Approval

↓

Deploy

↓

Validation
```

Independent workflows

are easier to maintain.

---

# Runner Strategy

Use

GitHub Hosted Runners

for

- Builds
- Testing
- Static Analysis

Use

Self-hosted Runners

for

- Private Infrastructure
- Production Deployment
- Internal Resources

---

# Deployment Strategy

Production deployments

should use

- Rolling Updates
- Blue-Green Deployments
- Canary Releases

Always prepare

a rollback strategy.

---

# Monitoring

Monitor

- Workflow Success Rate
- Pipeline Duration
- Deployment Frequency
- Failure Rate
- Runner Health

Track

CI/CD performance

continuously.

---

# Logging

Store

- Workflow Logs
- Deployment Logs
- Audit Logs
- Security Events

Logs support

incident investigation

and compliance.

---

# Governance

Enterprise pipelines

should enforce

- Naming Standards
- Workflow Templates
- Security Policies
- Deployment Approvals
- Audit Requirements

Automation

must remain consistent

across repositories.

---

# Enterprise Platform Architecture

```text
Developer

↓

GitHub

↓

Reusable Workflows

↓

GitHub Actions

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

Prometheus

↓

Grafana

↓

ELK
```

---

# Banking Example

```text
Developer

↓

Pull Request

↓

Reusable Workflow

↓

Security Scan

↓

Approval

↓

Amazon EKS

↓

Production
```

Every deployment

is reviewed,

validated,

and approved.

---

# GitHub Actions Maturity Model

```text
Level 1

↓

Manual Deployments

────────────

Level 2

↓

Basic CI

────────────

Level 3

↓

CI/CD Automation

────────────

Level 4

↓

Enterprise CI/CD

────────────

Level 5

↓

Platform Engineering

↓

Reusable Workflows

↓

Security

↓

Governance

↓

Observability
```

Organizations

should aim

for Level 5 maturity.

---

# Enterprise Checklist

Before every production deployment verify

✓ Pull Request Approved

✓ Branch Protection Enabled

✓ Workflow Passed

✓ Unit Tests Passed

✓ SonarQube Passed

✓ Trivy Scan Passed

✓ OIDC Authentication Used

✓ Artifact Generated

✓ Docker Image Versioned

✓ Amazon ECR Updated

✓ Terraform Validated

✓ Amazon EKS Deployment Verified

✓ Monitoring Enabled

✓ Rollback Plan Available

---

# Benefits

- Standardized CI/CD
- Faster Releases
- Better Security
- Easier Rollbacks
- Centralized Governance
- Improved Maintainability
- Reduced Operational Risk
- Enterprise Compliance

---

# Common Mistakes

- Creating one massive workflow.
- Hardcoding secrets.
- Using AWS access keys instead of OIDC.
- Deploying directly from feature branches.
- Ignoring workflow versioning.
- Skipping security scans.
- Running production deployments without approvals.
- Not separating CI and CD workflows.

---

# Interview Questions

## Basic

- What are GitHub Actions best practices?
- Why should CI and CD be separated?
- Why use reusable workflows?
- Why is OIDC preferred over AWS access keys?

## Intermediate

- Explain enterprise runner strategies.
- How do you secure GitHub Actions workflows?
- Why should workflows be modular?
- Explain Build Once, Deploy Everywhere.
- Why use environment protection rules?

## Advanced

- Design an enterprise GitHub Actions platform using reusable workflows, OIDC authentication, SonarQube, Trivy, Docker, Amazon ECR, Terraform, Amazon EKS, Prometheus, and Grafana.
- Explain how GitHub Actions best practices improve security, scalability, maintainability, and governance across hundreds of repositories.
- Your organization has over 500 repositories and multiple development teams. Explain how you would standardize CI/CD pipelines, manage reusable workflows, secure deployments, optimize runner usage, enforce governance, and ensure compliance while supporting rapid software delivery.

---

# Chapter 11 - GitHub Actions Enterprise Interview Handbook

GitHub Actions is one of the most important CI/CD platforms used in modern DevOps.

Enterprise interviews focus on much more than writing YAML files.

Interviewers evaluate your understanding of

- CI/CD Design
- Workflow Architecture
- Security
- OIDC Authentication
- Docker Integration
- Kubernetes Deployment
- Terraform Automation
- Enterprise Governance
- Production Troubleshooting

This chapter consolidates all GitHub Actions concepts into a practical interview handbook.

---

# GitHub Actions Learning Roadmap

```text
GitHub Actions Fundamentals

↓

Workflow Syntax

↓

Runners

↓

Variables & Secrets

↓

Artifacts & Cache

↓

Reusable Workflows

↓

Docker Integration

↓

OIDC Security

↓

Production Troubleshooting

↓

Enterprise Architecture
```

Master these topics

before attending

senior DevOps interviews.

---

# Enterprise CI/CD Flow

```text
Developer

↓

GitHub Repository

↓

Pull Request

↓

GitHub Actions

↓

Build

↓

Unit Tests

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

This architecture

is commonly used

in enterprise environments.

---

# Interview Answer Framework

When answering

architecture questions,

follow this structure.

```text
Requirements

↓

Architecture

↓

Workflow

↓

Security

↓

Deployment

↓

Monitoring

↓

Rollback

↓

Best Practices
```

A structured answer

demonstrates

real-world experience.

---

# Frequently Asked Interview Questions

## GitHub Actions Fundamentals

1. What is GitHub Actions?
2. What is CI/CD?
3. Why use GitHub Actions?
4. What is a Workflow?
5. What is an Event?
6. What is a Job?
7. What is a Step?
8. What is a Runner?
9. What is an Action?
10. What are Artifacts?

---

## Workflow Syntax

11. Explain workflow execution.
12. Job vs Step.
13. Workflow vs Action.
14. What are workflow triggers?
15. Push vs Pull Request trigger.
16. What is a Matrix Strategy?
17. What are workflow permissions?
18. How do conditional jobs work?
19. What are GitHub contexts?
20. How do reusable workflows work?

---

## Runners

21. GitHub Hosted Runner vs Self-hosted Runner.
22. Why use self-hosted runners?
23. Explain runner labels.
24. Explain runner groups.
25. How do runners execute workflows?
26. How do you secure runners?
27. How do self-hosted runners communicate with GitHub?
28. How do you scale self-hosted runners?
29. Runner cleanup strategy.
30. Production runner architecture.

---

## Variables & Secrets

31. Variables vs Secrets.
32. Repository Secrets vs Environment Secrets.
33. Repository Variables vs Organization Variables.
34. Why should secrets never be hardcoded?
35. What are GitHub contexts?
36. Explain expressions.
37. What is environment protection?
38. Why use GitHub Environments?
39. How do workflows receive configuration?
40. Explain dynamic workflows.

---

## Artifacts & Cache

41. Artifact vs Cache.
42. Why use dependency caching?
43. Explain artifact promotion.
44. Docker layer caching.
45. Terraform provider cache.
46. Artifact retention.
47. Build Once, Deploy Everywhere.
48. How do artifacts move between jobs?
49. How do caches improve performance?
50. Pipeline optimization strategies.

---

## Docker & Deployment

51. How does GitHub Actions build Docker images?
52. Explain Docker image tagging.
53. Why use Amazon ECR?
54. Explain deployment to Amazon EKS.
55. Rolling Deployment vs Blue-Green Deployment.
56. What is Canary Deployment?
57. How do you validate deployments?
58. Explain rollback strategy.
59. Why Build Once, Deploy Everywhere?
60. How do you monitor deployments?

---

## Security

61. Why is GitHub Actions security important?
62. What is OIDC?
63. IAM User vs IAM Role.
64. Why use OIDC instead of AWS Access Keys?
65. Explain GitHub Secrets.
66. How do protected branches improve security?
67. What are environment approvals?
68. Explain least privilege.
69. How do you secure GitHub runners?
70. Explain CI/CD security scanning.

---

## Enterprise Best Practices

71. Why should workflows be modular?
72. Why use reusable workflows?
73. Why separate CI and CD?
74. How do you standardize pipelines?
75. Explain governance.
76. Why version workflows?
77. Explain deployment approvals.
78. Workflow monitoring strategy.
79. Enterprise runner strategy.
80. Explain GitHub Actions maturity model.

---

# Scenario-Based Questions

## Scenario 1

A workflow is not triggered after a push.

How would you troubleshoot it?

---

## Scenario 2

A Docker image builds successfully

but cannot be pushed

to Amazon ECR.

Explain your investigation.

---

## Scenario 3

Amazon EKS deployment fails

after a successful build.

How would you troubleshoot it?

---

## Scenario 4

Your GitHub Actions pipeline takes

30 minutes.

How would you reduce it

to under 5 minutes?

---

## Scenario 5

A self-hosted runner

is offline.

Explain your investigation.

---

## Scenario 6

Security scanning reports

critical vulnerabilities.

Would you deploy?

Explain your reasoning.

---

## Scenario 7

A deployment succeeds,

but the application is unhealthy.

How would you investigate?

---

## Scenario 8

Production deployment

was accidentally triggered

from a feature branch.

How would you prevent this?

---

## Scenario 9

A workflow uses

AWS Access Keys

stored as secrets.

How would you migrate

to OIDC authentication?

---

## Scenario 10

Your organization manages

500 repositories.

Explain how you would standardize

CI/CD pipelines

using reusable workflows,

composite actions,

security controls,

and governance.

---

# Enterprise Architecture Questions

## Architecture 1

Design a complete CI/CD platform using

- GitHub
- GitHub Actions
- SonarQube
- Trivy
- Docker
- Amazon ECR
- Terraform
- Amazon EKS
- Prometheus
- Grafana

Explain the end-to-end deployment flow.

---

## Architecture 2

Your organization has

multiple AWS accounts

for

- Development
- Testing
- Production

Explain how GitHub Actions,

OIDC,

IAM Roles,

and GitHub Environments

work together

to deploy securely.

---

## Architecture 3

A company wants to migrate

from Jenkins

to GitHub Actions.

Explain

- Migration Strategy
- Workflow Organization
- Runner Strategy
- Security
- Deployment Pipeline
- Governance

---

## Architecture 4

A financial organization

requires

- Highly Secure CI/CD
- Deployment Approvals
- Immutable Deployments
- Full Audit Trail
- Multi-Team Governance

Design

the GitHub Actions platform.

---

# Production Readiness Checklist

Before production deployment verify

✓ Workflow Passed

✓ Pull Request Approved

✓ Branch Protection Enabled

✓ Unit Tests Passed

✓ SonarQube Passed

✓ Trivy Scan Passed

✓ OIDC Authentication Successful

✓ Docker Image Versioned

✓ Amazon ECR Updated

✓ Terraform Validated

✓ Amazon EKS Deployment Successful

✓ Monitoring Enabled

✓ Rollback Plan Ready

✓ Documentation Updated

---

# GitHub Actions Troubleshooting Checklist

Always verify

✓ Workflow Trigger

✓ YAML Syntax

✓ Runner Status

✓ Variables

✓ Secrets

✓ OIDC Authentication

✓ Docker Build

✓ Amazon ECR

✓ Terraform

✓ Amazon EKS

✓ Monitoring

✓ Deployment Validation

---

# GitHub Actions Handbook Summary

This handbook covered

- ✅ GitHub Actions Fundamentals
- ✅ Workflow Syntax
- ✅ Runners
- ✅ Variables, Secrets & Contexts
- ✅ Artifacts & Cache
- ✅ Reusable Workflows
- ✅ Docker, Amazon ECR & Amazon EKS Integration
- ✅ OIDC Authentication & Security
- ✅ Production Troubleshooting
- ✅ Enterprise Best Practices
- ✅ 80+ Interview Questions
- ✅ 10 Production Scenarios
- ✅ 4 Enterprise Architecture Discussions
- ✅ Production Checklists
- ✅ Enterprise Troubleshooting Framework

---

# File Completed

**File Name:** `115-GitHub-Actions-Enterprise-Handbook.md`