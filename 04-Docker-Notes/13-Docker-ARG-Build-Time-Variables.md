# Docker ARG (Build-Time Variables)

## Introduction

In real-world projects, Docker images are rarely built with hardcoded values.

Consider:

```text
NodeJS Version

Java Version

Python Version

Application Version

Environment Name
```

---

Hardcoding these values makes Dockerfiles:

```text
Less Flexible

Hard To Maintain

Difficult To Reuse
```

---

Docker provides:

```text
ARG
```

to solve this problem.

---

# What is ARG?

ARG stands for:

```text
Argument
```

---

ARG is used to define:

```text
Build-Time Variables
```

that can be supplied while building a Docker image.

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

During build:

```bash
docker build \
--build-arg VERSION=8 \
-t app:v1 .
```

---

Docker uses:

```text
almalinux:8
```

instead of:

```text
almalinux:9
```

---

# Why ARG?

Without ARG

```dockerfile
FROM node:20
```

---

Need Node 22?

Modify Dockerfile.

---

Need Node 18?

Modify Dockerfile Again.

---

This creates:

```text
Multiple Dockerfiles

Maintenance Problems
```

---

Using ARG

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

Build

```bash
docker build \
--build-arg NODE_VERSION=22 .
```

---

Same Dockerfile.

Different versions.

---

# ARG Syntax

## Simple Example

```dockerfile
ARG VERSION=9
```

---

Format

```dockerfile
ARG VARIABLE=VALUE
```

---

Components

```text
ARG

Variable Name

Default Value
```

---

# ARG with Default Value

Example

```dockerfile
ARG VERSION=9
```

---

Build

```bash
docker build .
```

---

Uses

```text
VERSION=9
```

---

# ARG with Override

Dockerfile

```dockerfile
ARG VERSION=9
```

---

Build

```bash
docker build \
--build-arg VERSION=8 .
```

---

Result

```text
VERSION=8
```

---

# ARG Before FROM

Normally:

```text
FROM
```

is the first instruction.

---

Exception:

```text
ARG
```

can appear before FROM.

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

Why?

Because:

```text
FROM Needs Value
```

before image creation starts.

---

# Dynamic Base Images

Example

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

Build

```bash
docker build \
--build-arg NODE_VERSION=22 .
```

---

Result

```text
node:22
```

used as base image.

---

# Real Production Example

Suppose:

```text
Development Team
```

tests:

```text
NodeJS 20
```

---

Production requires:

```text
NodeJS 22
```

---

Instead of:

```text
Changing Dockerfile
```

---

Use:

```bash
docker build \
--build-arg NODE_VERSION=22
```

---

Same Dockerfile.

---

# ARG Scope

Very Important Interview Topic.

---

Dockerfile

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

Question:

Can VERSION be used after FROM?

---

Answer:

```text
No
```

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}

RUN echo $VERSION
```

---

Result

```text
May Not Work As Expected
```

---

Reason

```text
ARG Scope Ends
After FROM
```

---

# Re-Declare ARG

If required later:

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}

ARG VERSION

RUN echo ${VERSION}
```

---

Now accessible.

---

# ARG Lifecycle

```text
Dockerfile
      ↓
docker build
      ↓
ARG Available
      ↓
Image Created
      ↓
ARG Removed
```

---

Important:

```text
ARG Exists Only During Build
```

---

# Is ARG Available Inside Container?

No.

---

Dockerfile

```dockerfile
ARG VERSION=1.0

FROM almalinux:9
```

---

Build Image.

---

Run Container.

---

Inside Container

```bash
env
```

---

Output

```text
VERSION Not Available
```

---

Reason

```text
ARG Is Build-Time Only
```

---

# Common ARG Use Cases

## Base Image Version

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

## Application Version

```dockerfile
ARG APP_VERSION=1.0
```

---

## Package Version

```dockerfile
ARG NGINX_VERSION=1.25
```

---

## Environment Selection

```dockerfile
ARG ENVIRONMENT=dev
```

---

# ARG in CI/CD Pipelines

Very Common.

---

Jenkins

```groovy
sh """
docker build \
--build-arg APP_VERSION=${BUILD_NUMBER} \
-t catalogue:${BUILD_NUMBER} .
"""
```

---

Result

```text
Every Build Uses
Different Version
```

---

# GitHub Actions Example

```yaml
- name: Build Image
  run: |
    docker build \
    --build-arg VERSION=${{ github.run_number }} \
    -t catalogue:${{ github.run_number }} .
```

---

# GitLab CI Example

```yaml
docker build \
--build-arg VERSION=$CI_PIPELINE_ID
```

---

# AWS DevOps Example

Dockerfile

```dockerfile
ARG APP_VERSION

FROM node:20

RUN echo ${APP_VERSION}
```

---

Pipeline

```bash
docker build \
--build-arg APP_VERSION=2.5.1 \
-t catalogue:2.5.1 .
```

---

Image built using:

```text
Application Version 2.5.1
```

---

# Build-Time Configuration

ARG is ideal for:

```text
Versions

Build Metadata

Temporary Values

Compilation Settings
```

---

Not ideal for:

```text
Passwords

Secrets

API Keys
```

---

Reason

```text
Build History May Expose Them
```

---

# ARG vs Hardcoding

Hardcoded

```dockerfile
FROM node:20
```

---

Flexible

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

Preferred:

```text
ARG
```

---

# Architecture

```text
Dockerfile
      ↓
ARG Value
      ↓
docker build
      ↓
Image Created
      ↓
ARG Removed
```

---

# Common Interview Questions

## What is ARG?

### Short Answer

ARG defines build-time variables used during image creation.

### Detailed Explanation

ARG allows values to be supplied dynamically while building Docker images.

### Production Example

Selecting NodeJS version during image build.

### Follow-Up Questions

- Can ARG be used before FROM?
- Is ARG available inside containers?

---

## Can ARG Be Used Before FROM?

### Short Answer

Yes.

### Detailed Explanation

This is the only instruction allowed before FROM because base image selection may depend on ARG values.

### Production Example

ARG VERSION=9 followed by FROM almalinux:${VERSION}.

### Follow-Up Questions

- Why is this special?
- What happens if ARG is after FROM?

---

## Is ARG Available Inside Containers?

### Short Answer

No.

### Detailed Explanation

ARG exists only during image build and disappears after image creation.

### Production Example

Build version variable not visible in running container.

### Follow-Up Questions

- Which instruction is available inside containers?
- How do you pass runtime configuration?

---

## What Are Common Uses of ARG?

### Short Answer

Build-time configuration and version selection.

### Detailed Explanation

ARG is commonly used for image versions, package versions, and CI/CD build metadata.

### Production Example

Passing Jenkins build number to Docker build.

### Follow-Up Questions

- Why not hardcode versions?
- Can ARG be used for secrets?

---

# Key Takeaways

```text
ARG defines build-time variables.

ARG values are available only during image creation.

ARG can be overridden using --build-arg.

ARG can appear before FROM.

ARG is commonly used for dynamic base images.

ARG is heavily used in CI/CD pipelines.

ARG is not available inside running containers.

ARG scope changes after FROM.

ARG is ideal for versioning and build metadata.

ARG should not be used for secrets.
```