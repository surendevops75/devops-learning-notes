# Docker Build

Docker Build in a CI pipeline is the process of creating a Docker image from application source code and a Dockerfile in an automated and repeatable way.

The main goal is to convert validated application code into a versioned, portable container image that can be scanned, stored, and deployed consistently across environments.

A typical Docker build flow looks like:

Application Source
        |
        ↓
CI Pipeline
        |
        ↓
Build Application
        |
        ↓
Docker Build
        |
        ↓
Docker Image
        |
        ↓
Security Scan
        |
        ↓
Container Registry

---

# What Is a Docker Image?

A Docker image is an immutable package containing everything required to run an application.

It can contain:

- Application Code
- Runtime
- Libraries
- Dependencies
- Configuration
- Startup Command

Example:

    myapp:v1.0.0

---

# What Is a Container?

A container is a running instance of a Docker image.

Docker Image
     |
     ↓
Container
     |
     ↓
Running Application

Example:

    myapp:v1.0.0
          |
          ↓
       Container
          |
          ↓
      Application

---

# Docker Build in CI

A CI pipeline can automatically build an image after application validation.

Git Push
   |
   ↓
Checkout
   |
   ↓
Build
   |
   ↓
Test
   |
   ↓
Security Checks
   |
   ↓
Docker Build
   |
   ↓
Image Scan
   |
   ↓
Push Image

---

# Why Docker Build Is Important

Docker provides:

- Consistent Runtime
- Portable Application
- Dependency Isolation
- Repeatable Builds
- Environment Consistency
- Easy Deployment
- Versioned Artifacts

Without containers:

Developer Environment
       |
       ↓
Different Runtime
       |
       ↓
Production Problem

With containers:

Application
    |
    ↓
Docker Image
    |
    ↓
Same Image
    |
    +-- Development
    +-- Testing
    +-- Staging
    +-- Production

---

# Dockerfile

A Dockerfile defines how the image is built.

Example:

    FROM node:20-alpine
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci
    COPY . .
    EXPOSE 3000
    CMD ["npm", "start"]

The exact Dockerfile depends on the application.

---

# Dockerfile Instructions

Common Dockerfile instructions include:

- FROM
- WORKDIR
- COPY
- ADD
- RUN
- ENV
- ARG
- EXPOSE
- USER
- ENTRYPOINT
- CMD

---

# FROM

FROM defines the base image.

Example:

    FROM node:20-alpine

Another example:

    FROM python:3.12-slim

Java:

    FROM eclipse-temurin:17-jre

---

# Base Image

The base image provides the starting environment.

Base Image
    |
    +-- Operating System Files
    +-- Runtime
    +-- Libraries

Then application components are added.

Base Image
    +
Application
    +
Dependencies
    =
Docker Image

---

# WORKDIR

WORKDIR defines the working directory inside the image.

Example:

    WORKDIR /app

After this:

    /app

becomes the working directory for subsequent instructions.

---

# COPY

COPY copies files from the build context into the image.

Example:

    COPY package.json .

Copy application code:

    COPY . .

---

# ADD

ADD can also copy files into the image.

Example:

    ADD app.tar.gz /app/

For ordinary file copying, COPY is generally clearer and preferred.

---

# RUN

RUN executes commands during image construction.

Example:

    RUN npm ci

Python:

    RUN pip install --no-cache-dir -r requirements.txt

Java:

    RUN ./mvnw clean package

---

# CMD

CMD defines the default command used when a container starts.

Example:

    CMD ["npm", "start"]

Python:

    CMD ["python", "app.py"]

Java:

    CMD ["java", "-jar", "app.jar"]

---

# ENTRYPOINT

ENTRYPOINT defines the main executable for the container.

Example:

    ENTRYPOINT ["java", "-jar", "app.jar"]

A common distinction is:

ENTRYPOINT
    |
    ↓
Main executable

CMD
    |
    ↓
Default arguments / command

---

# CMD vs ENTRYPOINT

CMD:

- Provides a default command or arguments.

ENTRYPOINT:

- Defines the main executable.

Example:

    ENTRYPOINT ["java", "-jar"]
    CMD ["app.jar"]

The exact design depends on how the image is intended to be used.

---

# EXPOSE

EXPOSE documents the port the application listens on.

Example:

    EXPOSE 8080

It does not by itself publish the port to the host.

Runtime configuration is still required.

---

# ENV

ENV defines environment variables in the image.

Example:

    ENV APP_ENV=production

Avoid putting sensitive secrets into Dockerfile ENV values.

---

# ARG

ARG defines build-time variables.

Example:

    ARG APP_VERSION=1.0.0

Use:

    RUN echo $APP_VERSION

Important distinction:

    ARG → Build Time
    ENV → Runtime Environment

Sensitive information should not be passed as ordinary build arguments because build metadata and layers can expose information depending on how the image is constructed.

---

# USER

A container can run as a non-root user.

Example:

    USER appuser

Running as non-root can reduce the impact of container compromise.

---

# Docker Build Context

When running:

    docker build -t myapp:v1.0.0 .

The final "." represents the build context.

Conceptually:

Current Directory
       |
       ↓
Build Context
       |
       ↓
Docker Build

The Docker builder receives files from the build context that are not excluded by .dockerignore.

---

# .dockerignore

.dockerignore prevents unnecessary files from entering the build context.

Example:

    .git
    .gitignore
    node_modules
    .venv
    __pycache__
    *.pyc
    coverage
    .env

Benefits:

- Smaller Build Context
- Faster Builds
- Less Unnecessary Data
- Reduced Risk of Copying Sensitive Files

---

# Docker Build Command

Basic command:

    docker build -t myapp:v1.0.0 .

Explanation:

    docker build
        ↓
    Build Image

    -t
        ↓
    Assign Tag

    myapp:v1.0.0
        ↓
    Image Name + Tag

    .
        ↓
    Build Context

---

# Docker Build with Dockerfile

If the Dockerfile has a different name:

    docker build -f Dockerfile.prod -t myapp:v1.0.0 .

Here:

    -f

specifies the Dockerfile.

---

# Docker Image Tags

An image can have:

- Repository
- Tag

Example:

    myapp:v1.0.0

Where:

    myapp
        ↓
    Repository / Image Name

    v1.0.0
        ↓
    Tag

---

# Why Version Docker Images?

Avoid relying only on:

    myapp:latest

Prefer:

    myapp:v1.0.0
    myapp:v1.0.1
    myapp:v1.1.0

Versioned images improve:

- Traceability
- Rollback
- Auditability
- Deployment Control

---

# Docker Image ID

Docker images also have an image ID.

Command:

    docker images

Output can contain:

    REPOSITORY   TAG      IMAGE ID
    myapp        v1.0.0   abc123...

The image ID identifies the image locally.

---

# Docker Image Digest

A registry can identify an image using a digest.

Example:

    sha256:xxxxxxxx...

A digest provides an immutable image reference.

Conceptually:

    myapp:v1.0.0
          |
          ↓
      Image Digest
          |
          ↓
      sha256:...

For strong deployment immutability, digest-based references can be useful.

---

# Docker Build Layers

Docker images are built from layers.

Example:

Application Layer
       |
       ↓
Dependencies Layer
       |
       ↓
Runtime Layer
       |
       ↓
Base Image

---

# Why Layers Matter

Layers help with:

- Caching
- Build Performance
- Storage Efficiency

If an earlier layer does not change, Docker can often reuse its cached result.

---

# Docker Build Cache

Example Dockerfile ordering:

    COPY package*.json ./
    RUN npm ci
    COPY . .

If only application source changes:

    package.json
        |
        ↓
    Same
        |
        ↓
    npm ci Layer
        |
        ↓
    Cached

Then only later layers need rebuilding.

---

# Poor Dockerfile Ordering

Example:

    COPY . .
    RUN npm ci

If any source file changes:

    COPY Layer
        |
        ↓
      Changed
        |
        ↓
      npm ci
        |
        ↓
    Rebuild Dependencies

This can make builds slower.

---

# Better Dockerfile Ordering

Prefer:

    COPY package*.json ./
    RUN npm ci
    COPY . .

Now:

Dependency Files
      |
      ↓
Install Dependencies
      |
      ↓
Application Source

Source changes are less likely to invalidate the dependency installation layer.

---

# Docker Build Cache Strategy

General principle:

Less Frequently Changed Files
        |
        ↓
Earlier Layers

Frequently Changed Files
        |
        ↓
Later Layers

Example:

Base Image
    ↓
System Dependencies
    ↓
Application Dependencies
    ↓
Application Source

---

# Docker BuildKit

Modern Docker uses BuildKit for improved build functionality.

BuildKit can provide:

- Improved Caching
- Parallel Build Operations
- Better Build Performance
- Secrets Handling
- Cache Export / Import
- Multi-Platform Builds

---

# BuildKit Cache

CI systems can use remote or persistent caches.

Conceptually:

CI Runner
    |
    ↓
BuildKit
    |
    +-- Local Cache
    |
    +-- Remote Cache
    |
    ↓
Docker Image

This can significantly reduce repeated builds.

---

# Docker Build in CI

Typical pipeline:

Checkout
   |
   ↓
Application Build
   |
   ↓
Unit Tests
   |
   ↓
Quality Checks
   |
   ↓
Security Checks
   |
   ↓
Docker Build

Only build the production image after required validation passes.

---

# CI Docker Build Flow

Git Repository
       |
       ↓
CI Runner
       |
       ↓
Checkout
       |
       ↓
Build Application
       |
       ↓
Run Tests
       |
       ↓
SonarQube
       |
       ↓
Security Checks
       |
       ↓
Docker Build
       |
       ↓
Docker Image
       |
       ↓
Trivy Scan
       |
       ↓
Container Registry

---

# Should Docker Build Run Before Tests?

Both approaches are possible.

For many pipelines:

Application Tests
      |
      ↓
Docker Build

This avoids building an image when application validation already failed.

However, some teams may build earlier for specific testing strategies.

The correct order depends on pipeline requirements.

---

# Docker Build and Security

Docker images should be scanned before publishing or deployment.

Example:

Docker Build
      |
      ↓
Trivy
      |
      ↓
Vulnerability Scan
      |
      +-- Pass → Push
      |
      +-- Fail → Stop

---

# Trivy Image Scan

Example:

    trivy image myapp:v1.0.0

It can identify vulnerabilities according to the configured scanners and databases.

---

# Security Gate

A pipeline can enforce vulnerability thresholds.

Conceptually:

Trivy
  |
  ↓
Critical Findings?
  |
  +-- No → Continue
  |
  +-- Yes → Fail

The exact threshold should follow organizational security policy.

---

# Dockerfile Security

Avoid relying blindly on:

    FROM some-image:latest

when deterministic builds are required.

Prefer a controlled and approved image version.

Example:

    FROM node:20-alpine

Even stronger reproducibility can be achieved by pinning the base image by digest.

---

# Base Image Security

Base images can contain vulnerabilities.

Application
    |
    ↓
Base Image
    |
    ↓
OS / Runtime Vulnerabilities

Therefore:

Base Image
    |
    ↓
Regular Scanning
    |
    ↓
Updates

should be part of the container lifecycle.

---

# Minimal Base Images

Examples include:

- Alpine
- Slim
- Distroless

The appropriate image depends on application compatibility and security requirements.

Smaller images can reduce:

- Attack Surface
- Image Size
- Download Time
- Storage

But minimal images may also make troubleshooting or compatibility more difficult.

---

# Alpine Considerations

Alpine uses:

    musl libc

instead of glibc.

Some applications or native dependencies may behave differently.

Therefore:

    Small Image
        ≠
    Always Best Image

Test the application thoroughly.

---

# Distroless Images

Distroless images contain fewer components than traditional Linux images.

They can reduce the runtime attack surface.

Typical concept:

Build Image
     |
     ↓
Application
     |
     ↓
Distroless Runtime

They may provide fewer shell/debugging utilities, so operational practices should account for that.

---

# Multi-Stage Docker Build

Multi-stage builds separate build-time and runtime environments.

Example:

    FROM node:20-alpine AS build

    WORKDIR /app

    COPY package*.json ./

    RUN npm ci

    COPY . .

    RUN npm run build

    FROM node:20-alpine

    WORKDIR /app

    COPY package*.json ./

    RUN npm ci --omit=dev

    COPY --from=build /app/dist ./dist

    CMD ["node", "dist/server.js"]

---

# Why Multi-Stage Builds?

Benefits include:

- Smaller Runtime Image
- Fewer Build Tools
- Reduced Attack Surface
- Cleaner Runtime
- Better Separation

Conceptually:

Build Stage
    |
    +-- Compiler
    +-- Build Tools
    +-- Dev Dependencies
    |
    ↓
Application Output
    |
    ↓
Runtime Stage
    |
    +-- Runtime
    +-- Application

---

# Java Docker Build

Example:

    FROM eclipse-temurin:17-jre

    WORKDIR /app

    COPY target/myapp.jar app.jar

    EXPOSE 8080

    ENTRYPOINT ["java", "-jar", "app.jar"]

CI:

    mvn clean package
    docker build -t myapp:v1.0.0 .

---

# Node.js Docker Build

Example:

    FROM node:20-alpine

    WORKDIR /app

    COPY package*.json ./

    RUN npm ci --omit=dev

    COPY dist ./dist

    EXPOSE 3000

    CMD ["node", "dist/server.js"]

CI:

    npm ci
    npm test
    npm run build
    docker build -t myapp:v1.0.0 .

---

# Python Docker Build

Example:

    FROM python:3.12-slim

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install --no-cache-dir -r requirements.txt

    COPY . .

    EXPOSE 8000

    CMD ["python", "app.py"]

CI:

    pip install -r requirements.txt
    pytest
    docker build -t myapp:v1.0.0 .

---

# Docker Build Arguments

Build arguments can be supplied using:

    docker build \
      --build-arg APP_VERSION=1.0.0 \
      -t myapp:v1.0.0 .

Dockerfile:

    ARG APP_VERSION
    RUN echo "Building version $APP_VERSION"

Do not use build arguments as a secure secret store.

---

# Docker Build Secrets

BuildKit provides mechanisms for securely supplying build secrets without intentionally embedding them into the final image.

Conceptually:

CI Secret
    |
    ↓
BuildKit Secret
    |
    ↓
Build Step

Secrets should be handled according to the CI and Docker BuildKit configuration.

---

# Docker Build and Private Registries

A Docker build may need access to private package repositories.

Examples:

- Private npm Registry
- Private PyPI
- Maven Repository
- JFrog Artifactory

Credentials should be provided securely.

Do not write credentials directly into:

- Dockerfile
- Source Code
- Git Repository

---

# Docker Registry

A container registry stores Docker images.

Examples:

- Amazon ECR
- Docker Hub
- JFrog Artifactory
- GitHub Container Registry
- GitLab Container Registry

Typical flow:

Docker Build
    |
    ↓
Docker Image
    |
    ↓
Security Scan
    |
    ↓
Registry

---

# Docker Login

Before pushing:

    docker login

For cloud registries, authentication is normally handled through the provider's supported authentication mechanism.

Credentials should not be exposed in command history or CI logs.

---

# Docker Push

Example:

    docker push myapp:v1.0.0

Flow:

Local Image
     |
     ↓
docker push
     |
     ↓
Registry

---

# Docker Pull

To retrieve an image:

    docker pull myapp:v1.0.0

Deployment systems can pull the image from the registry.

---

# Docker Build and ECR

Typical AWS flow:

Source Code
    |
    ↓
CI
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Amazon ECR

Example:

    123456789012.dkr.ecr.region.amazonaws.com/myapp:v1.0.0

The exact registry URI depends on the AWS account and region.

---

# ECR Image Tagging

Example:

    docker tag myapp:v1.0.0 \
      <ecr-repository>:v1.0.0

Then:

    docker push <ecr-repository>:v1.0.0

The CI system should construct the registry reference securely.

---

# Docker Build and Git Commit

A useful strategy is to associate the image with a Git commit.

Example:

    Git Commit:
    8f3a91d

    Docker Image:
    myapp:8f3a91d

Another approach:

    myapp:v1.2.0

Both can be useful depending on release and traceability requirements.

---

# Image Tagging Strategy

Possible tags:

- v1.2.0
- 8f3a91d
- build-125

Avoid using only:

    latest

for production deployment tracking.

---

# Immutable Image Strategy

A stronger deployment approach is:

Build Once
   |
   ↓
Scan
   |
   ↓
Push
   |
   ↓
Deploy Same Image

Do not rebuild different images for each environment if the goal is artifact immutability.

---

# Build Once, Deploy Many

Recommended flow:

Source
   |
   ↓
CI Build
   |
   ↓
Docker Image
   |
   ↓
Scan
   |
   ↓
Registry
   |
   +----> Dev
   |
   +----> Stage
   |
   +----> Production

The same validated image can move through environments.

---

# Docker Build and Environment Configuration

Avoid baking environment-specific configuration into the image when it should vary between environments.

Bad approach:

    dev-image
    stage-image
    prod-image

Preferred concept:

Same Image
    |
    +-- Dev Configuration
    +-- Stage Configuration
    +-- Prod Configuration

Configuration should be injected at runtime where appropriate.

---

# Docker Image Immutability

Once an image version is published, it should ideally not be modified.

Example:

    myapp:v1.0.0

should always refer to the same image content in the deployment workflow.

This improves:

- Traceability
- Rollback
- Auditability
- Reproducibility

---

# Docker Build in Jenkins

Example:

    pipeline {
        agent any

        stages {

            stage('Checkout') {
                steps {
                    checkout scm
                }
            }

            stage('Test') {
                steps {
                    sh 'npm ci'
                    sh 'npm test'
                }
            }

            stage('Docker Build') {
                steps {
                    sh 'docker build -t myapp:${BUILD_NUMBER} .'
                }
            }

            stage('Image Scan') {
                steps {
                    sh 'trivy image myapp:${BUILD_NUMBER}'
                }
            }
        }
    }

The exact registry and security configuration should be added according to the environment.

---

# Docker Build in GitHub Actions

Example:

    name: Docker CI

    on:
      push:
        branches:
          - main

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Build Application
            run: |
              npm ci
              npm test
              npm run build

          - name: Build Docker Image
            run: |
              docker build -t myapp:${{ github.sha }} .

The example assumes a Node.js application. The application build commands should be adjusted for Java, Python, or another stack.

---

# GitHub Actions Docker Metadata

A CI workflow can generate tags based on:

- Git SHA
- Git Tag
- Branch
- Release Version

Example:

    myapp:8f3a91d
    myapp:v1.2.0

This improves traceability.

---

# Docker Build in GitLab CI

Example:

    stages:
      - test
      - build

    test:
      stage: test
      script:
        - npm ci
        - npm test

    docker-build:
      stage: build
      script:
        - docker build -t myapp:$CI_COMMIT_SHA .

The exact runner configuration depends on how Docker builds are enabled in GitLab.

---

# Docker Build Performance

A slow Docker build can be caused by:

- Large Build Context
- Poor Layer Ordering
- No Cache
- Large Base Image
- Repeated Dependency Installation
- Unnecessary Files
- Slow Network
- Large Dockerfile Operations

---

# How to Improve Docker Build Speed

Use:

- .dockerignore
- Layer Caching
- Correct Layer Ordering
- Multi-Stage Builds
- BuildKit
- Remote Cache
- Smaller Build Context
- Efficient Dependency Installation

---

# Large Build Context

Bad:

    Project
    ├── .git
    ├── node_modules
    ├── coverage
    ├── logs
    ├── temporary files
    └── source

All are sent as build context.

Better:

    Project
    ├── .dockerignore
    └── source

Only required files are included.

---

# Docker Build Cache in CI

CI runners are often temporary.

Therefore:

Runner 1
   |
   ↓
Build
   |
   ↓
Cache Lost

A remote cache can help:

Runner
   |
   ↓
BuildKit
   |
   ↓
Remote Cache

Then future builds can reuse previously generated layers.

---

# Docker Build and Parallelism

Some BuildKit operations can be optimized automatically.

Conceptually:

Build
 |
 +-- Dependency Layer
 |
 +-- Application Preparation
 |
 +-- Other Independent Steps
 |
 ↓
Final Image

The exact parallelism depends on the Dockerfile and build graph.

---

# Docker Build Reproducibility

A reproducible image build should control:

- Base Image
- Dependency Versions
- Build Commands
- Application Source
- Build Arguments
- Package Manager

Example:

    Node.js 20
    package-lock.json
    npm ci
    Approved Base Image

---

# Base Image Pinning

Using:

    FROM node:20-alpine

still allows the tag to move as the image is updated.

For stronger immutability, a base image can be pinned by digest.

Conceptually:

    FROM node:20-alpine@sha256:<digest>

Organizations should balance reproducibility with their base-image update process.

---

# Docker Build Security Best Practices

- Use Approved Base Images
- Keep Base Images Updated
- Scan Images
- Use Multi-Stage Builds
- Use Minimal Runtime Images
- Do Not Hardcode Secrets
- Use .dockerignore
- Avoid Running as Root Where Appropriate
- Pin Important Dependencies
- Use Versioned Image Tags
- Use Immutable Image References
- Limit Build Context

---

# Dockerfile Best Practices

- Use a Clear Base Image
- Use WORKDIR
- Use COPY Carefully
- Order Layers for Caching
- Use Multi-Stage Builds
- Use .dockerignore
- Avoid Unnecessary Packages
- Avoid Secrets
- Use Non-Root User Where Appropriate
- Use Explicit Startup Commands
- Keep Images Small

---

# Docker Build Failure

Common failure:

    failed to solve

Possible causes:

- Invalid Dockerfile
- Missing File
- Dependency Failure
- Network Failure
- Base Image Unavailable
- Permission Problem
- Build Context Problem

Check:

    docker build -t myapp:test .

Read the failing Dockerfile instruction carefully.

---

# Missing File During Build

Example:

    COPY target/app.jar app.jar

fails because:

    target/app.jar

does not exist.

Check:

    ls -l target/

Ensure the application build happens before the Docker build.

---

# Docker Build Permission Problem

Possible causes:

- Incorrect File Ownership
- Build User Permissions
- Registry Authentication
- Docker Daemon Access

Check the CI runner and Docker configuration.

Avoid blindly using privileged access to solve permission problems.

---

# Base Image Pull Failure

Example:

    failed to resolve source metadata

Possible causes:

- Registry Problem
- Network Problem
- Invalid Image Name
- Authentication Problem
- Rate Limit
- Unavailable Tag

Check:

    docker pull <base-image>

where appropriate.

---

# Docker Build Network Failure

Possible causes:

- Internet Connectivity
- Private Registry Access
- Proxy Configuration
- DNS
- Firewall
- Registry Outage

Check the CI runner's network access.

---

# Docker Build and Proxy

Enterprise environments may require:

- HTTP Proxy
- HTTPS Proxy
- NO_PROXY

These should be configured securely at the CI or Docker runtime level.

Do not hardcode credentials into the Dockerfile.

---

# Docker Build and Private Dependencies

Example:

Docker Build
    |
    ↓
Private npm Registry

or:

Docker Build
    |
    ↓
Private Maven Repository

or:

Docker Build
    |
    ↓
Private PyPI

Credentials should be supplied using secure mechanisms.

---

# Docker Build and Artifact Repository

Application artifact:

    myapp.jar

or:

    myapp.whl

can be copied into the image.

Example:

Build Artifact
      |
      ↓
Docker Build
      |
      ↓
Docker Image

---

# Java Artifact to Docker Image

Java Source
    |
    ↓
Maven
    |
    ↓
myapp.jar
    |
    ↓
Docker Build
    |
    ↓
myapp:v1.0.0

---

# Node.js Build to Docker Image

Node.js Source
    |
    ↓
npm ci
    |
    ↓
npm run build
    |
    ↓
dist/
    |
    ↓
Docker Build
    |
    ↓
myapp:v1.0.0

---

# Python Build to Docker Image

Python Source
    |
    ↓
Dependencies
    |
    ↓
Tests
    |
    ↓
Package / Application
    |
    ↓
Docker Build
    |
    ↓
myapp:v1.0.0

---

# Docker Build in DevSecOps

Docker Build should be integrated into the DevSecOps pipeline.

Example:

Developer
    |
    ↓
Git
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
SonarQube
    |
    ↓
Dependency Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Security Gate
    |
    ↓
Registry

---

# Docker Build with Quality Gate

Example:

Application Build
       |
       ↓
Tests
       |
       ↓
SonarQube
       |
       ↓
Quality Gate
       |
       +-- Fail → Stop
       |
       +-- Pass
             |
             ↓
        Docker Build

This prevents unnecessary container builds when mandatory quality checks fail.

---

# Docker Build with Security Gate

Example:

Docker Build
      |
      ↓
Trivy Scan
      |
      ↓
Security Gate
      |
      +-- Fail → Stop
      |
      +-- Pass
            |
            ↓
          Push

---

# Build Once, Scan Once, Deploy Many

Recommended concept:

Source
   |
   ↓
Build
   |
   ↓
Docker Image
   |
   ↓
Scan
   |
   ↓
Registry
   |
   +----> Dev
   |
   +----> Stage
   |
   +----> Production

Do not rebuild different image contents for every environment unless there is a specific requirement.

---

# Docker Image Promotion

A mature process can promote the same image through environments.

Build
  |
  ↓
Image
  |
  ↓
Scan
  |
  ↓
Registry
  |
  ↓
Development
  |
  ↓
Staging
  |
  ↓
Production

This improves confidence that the tested artifact is the one deployed.

---

# Docker Build and GitOps

Example:

Application Repository
       |
       ↓
CI
       |
       ↓
Docker Image
       |
       ↓
ECR
       |
       ↓
GitOps Repository
       |
       ↓
Image Tag Update
       |
       ↓
ArgoCD
       |
       ↓
EKS

The application image is created by CI, while deployment state is managed through GitOps.

---

# Docker Build and ArgoCD

ArgoCD does not build the application image.

Typical separation:

CI
 |
 +-- Build
 +-- Test
 +-- Docker Build
 +-- Security Scan
 +-- Push Image
 |
 ↓
ECR

GitOps
 |
 +-- Image Reference
 |
 ↓
ArgoCD
 |
 ↓
EKS

---

# Docker Build and Terraform

Terraform can provision infrastructure required for container deployment.

Example:

Terraform
    |
    +-- VPC
    +-- EKS
    +-- ECR
    +-- IAM
    +-- Load Balancer
    |
    ↓
Infrastructure

CI
    |
    ↓
Docker Image
    |
    ↓
ECR

Terraform manages infrastructure while CI manages application image creation.

---

# Docker Build Interview Questions

## Basic

1. What is Docker?
2. What is a Docker image?
3. What is a container?
4. What is a Dockerfile?
5. What is Docker Build?
6. What is a Docker build context?
7. What is .dockerignore?
8. What is the difference between COPY and ADD?
9. What is RUN?
10. What is CMD?
11. What is ENTRYPOINT?
12. What is EXPOSE?
13. What is ARG?
14. What is ENV?
15. What is a Docker image tag?

---

# Intermediate Interview Questions

16. How does Docker build an image?
17. What are Docker layers?
18. How does Docker layer caching work?
19. How would you optimize a Dockerfile?
20. What is a multi-stage Docker build?
21. Why should you use .dockerignore?
22. How do you build a Docker image in Jenkins?
23. How do you build a Docker image in GitHub Actions?
24. How do you scan a Docker image with Trivy?
25. How do you push an image to ECR?
26. Why should you avoid using only latest?
27. What is the difference between an image tag and digest?
28. How do you troubleshoot a Docker build failure?
29. How do you reduce Docker image size?
30. How do you securely handle secrets during Docker builds?

---

# Advanced Interview Questions

31. Design a production-grade Docker build pipeline.
32. How would you implement Docker build security?
33. How would you optimize a Docker build that takes 10 minutes?
34. How would you implement Docker layer caching in CI?
35. How would you use BuildKit in CI?
36. How would you implement multi-stage builds?
37. How would you select a secure base image?
38. How would you make Docker builds reproducible?
39. How would you implement immutable image deployment?
40. How would you connect Docker Build with ECR?
41. How would you connect Docker Build with GitOps and ArgoCD?
42. How would you implement build-once-deploy-many?
43. How would you handle a vulnerable base image?
44. How would you troubleshoot a Docker build that works locally but fails in CI?
45. How would you trace a production container back to a Git commit?

---

# Scenario Question

## The Docker build works locally but fails in CI. How would you troubleshoot it?

I would compare:

- Docker Version
- BuildKit Configuration
- Dockerfile
- Build Context
- Base Image
- Network
- Credentials
- Environment Variables
- File Permissions
- Architecture
- Build Arguments

First I would reproduce:

    docker build -t myapp:test .

Then inspect the exact Dockerfile instruction that failed.

---

# Scenario Question

## Docker builds are taking 10 minutes. How would you reduce the time?

First measure where time is spent.

Base Image Pull
      |
      ↓
Dependency Install
      |
      ↓
Application Copy
      |
      ↓
Build

Possible improvements:

- Layer Cache
- Better Layer Ordering
- .dockerignore
- Multi-Stage Builds
- BuildKit
- Remote Cache
- Smaller Build Context
- Dependency Cache
- Smaller Base Image

I would optimize based on measured bottlenecks.

---

# Scenario Question

## Your Docker image is 2 GB. How would you reduce it?

I would investigate:

- Base Image
- Application Dependencies
- Build Tools
- Unnecessary Files
- Logs
- Caches
- Development Dependencies
- Build Artifacts

Then consider:

- Minimal Base Image
- Multi-Stage Build
- Production Dependencies Only
- .dockerignore
- Remove Package Caches
- Remove Build Tools

Then rebuild and measure the resulting image.

---

# Scenario Question

## Trivy finds critical vulnerabilities in the Docker image. What would you do?

I would:

1. Identify Vulnerable Packages
2. Identify Whether They Come From Base Image or Application
3. Check Fixed Versions
4. Update Base Image
5. Update Application Dependencies
6. Rebuild Image
7. Run Tests
8. Run Trivy Again
9. Review Security Gate
10. Publish Only After Required Policy Passes

If no fix exists, follow the organization's vulnerability exception and mitigation process.

---

# Scenario Question

## How would you implement an immutable Docker image strategy?

I would:

Build Image Once
       |
       ↓
Assign Version / Commit Tag
       |
       ↓
Scan
       |
       ↓
Push to Registry
       |
       ↓
Reference Exact Image
       |
       ↓
Deploy

For stronger immutability, deployment can reference an image digest.

---

# Scenario Question

## How would you implement Docker CI for your microservices project?

Example:

Developer
    |
    ↓
Git
    |
    ↓
CI
    |
    +-- Checkout
    +-- Application Build
    +-- Unit Tests
    +-- SonarQube
    +-- Security Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS

Each microservice can have its own image and version.

---

# Scenario Question

## How would you implement rollback?

Use immutable image versions.

Example:

    myapp:v1.0.0
    myapp:v1.1.0
    myapp:v1.2.0

Production:

    v1.2.0
       |
       ↓
    Problem
       |
       ↓
    Rollback
       |
       ↓
    v1.1.0

With GitOps:

GitOps Repository
      |
      ↓
Previous Image Version
      |
      ↓
ArgoCD
      |
      ↓
EKS

---

# Scenario Question

## How would you secure a Dockerfile?

I would:

- Use Approved Base Image
- Keep Base Image Updated
- Scan Image
- Use Multi-Stage Build
- Avoid Secrets
- Use .dockerignore
- Run as Non-Root Where Appropriate
- Minimize Installed Packages
- Use Versioned Images
- Use Immutable References

---

# Scenario Question

## How would you implement Docker Build with GitHub Actions?

Example:

Git Push
   |
   ↓
GitHub Actions
   |
   ↓
Checkout
   |
   ↓
Application Test
   |
   ↓
Docker Build
   |
   ↓
Trivy
   |
   ↓
ECR

The workflow should authenticate to ECR using secure GitHub/AWS credentials and push only after required validation succeeds.

---

# Complete Docker CI Architecture

                         Git Repository
                               |
                               ↓
                         Pull Request
                               |
                               ↓
                           CI Runner
                               |
                               ↓
                        Application Build
                               |
                               ↓
                              Tests
                               |
                               ↓
                           SonarQube
                               |
                               ↓
                         Security Checks
                               |
                               ↓
                         Docker Build
                               |
                               ↓
                      Docker Image Created
                               |
                               ↓
                         Trivy Scan
                               |
                               ↓
                         Security Gate
                               |
                         +-----+-----+
                         |           |
                       Fail         Pass
                         |           |
                         ↓           ↓
                       Stop         ECR
                                     |
                                     ↓
                              GitOps Repository
                                     |
                                     ↓
                                   ArgoCD
                                     |
                                     ↓
                                    EKS
                                     |
                                     ↓
                                Production

---

# Docker Build Best-Practice Checklist

- Use approved base images
- Use supported runtime versions
- Keep base images updated
- Use .dockerignore
- Keep build context small
- Order Dockerfile layers correctly
- Use layer caching
- Use multi-stage builds where appropriate
- Avoid unnecessary packages
- Avoid hardcoded secrets
- Use secure build-secret mechanisms where required
- Run as non-root where appropriate
- Scan images with Trivy
- Enforce security gates
- Use versioned image tags
- Prefer immutable image references
- Push only validated images
- Maintain commit-to-image traceability
- Use build-once-deploy-many
- Maintain rollback capability

---

# Quick Revision

Docker Build:

Git
 ↓
Checkout
 ↓
Application Build
 ↓
Tests
 ↓
SonarQube
 ↓
Security Checks
 ↓
Docker Build
 ↓
Docker Image
 ↓
Trivy
 ↓
Security Gate
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS

Important commands:

    docker build -t myapp:v1.0.0 .
    docker images
    docker tag myapp:v1.0.0 <registry>/myapp:v1.0.0
    docker push <registry>/myapp:v1.0.0
    docker pull <registry>/myapp:v1.0.0
    docker run myapp:v1.0.0
    docker inspect myapp:v1.0.0
    trivy image myapp:v1.0.0

Important Dockerfile instructions:

- FROM
- WORKDIR
- COPY
- ADD
- RUN
- ARG
- ENV
- EXPOSE
- USER
- ENTRYPOINT
- CMD

Important concepts:

- Docker Image
- Docker Container
- Dockerfile
- Build Context
- .dockerignore
- Layers
- Layer Caching
- BuildKit
- Multi-Stage Builds
- Base Images
- Image Tags
- Image Digests
- Image Security
- Trivy
- Container Registry
- ECR
- Immutable Images
- Build Once Deploy Many
- GitOps
- ArgoCD
- EKS

Core idea:

Docker Build converts validated application code into a versioned and reproducible container image. In a DevSecOps pipeline, the image should be built, scanned, securely stored, and then promoted through environments without rebuilding the application artifact.