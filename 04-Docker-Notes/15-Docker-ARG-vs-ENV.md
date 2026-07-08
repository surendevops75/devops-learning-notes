# Docker ARG vs ENV

## Introduction

One of the most frequently asked Docker interview questions is:

```text
ARG vs ENV
```

---

Although both store values:

```text
Their Purpose

Lifecycle

Scope

Usage
```

are completely different.

---

Understanding ARG vs ENV is essential for:

```text
Docker

CI/CD

Kubernetes

Helm

Production Deployments
```

---

# Quick Overview

## ARG

```text
Build-Time Variable
```

Used while:

```text
Building Image
```

---

## ENV

```text
Runtime Variable
```

Used while:

```text
Running Container
```

---

# Real World Analogy

Suppose you are building a house.

---

During Construction

```text
Which Cement?

Which Paint?

Which Tiles?
```

---

These decisions exist only during:

```text
Construction
```

---

Equivalent:

```text
ARG
```

---

After House Is Built

```text
Room Temperature

WiFi Name

TV Channel
```

---

These are used while:

```text
Living In House
```

---

Equivalent:

```text
ENV
```

---

# What is ARG?

ARG defines:

```text
Build-Time Variables
```

---

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

used during image creation.

---

# What is ENV?

ENV defines:

```text
Runtime Variables
```

---

Example

```dockerfile
ENV APP_ENV=dev
```

---

Container receives:

```text
APP_ENV=dev
```

---

Application can read it.

---

# Lifecycle Comparison

## ARG Lifecycle

```text
Docker Build Starts
        ↓
ARG Available
        ↓
Image Created
        ↓
ARG Removed
```

---

ARG disappears after:

```text
Image Creation
```

---

## ENV Lifecycle

```text
Docker Build
      ↓
Image Created
      ↓
Container Starts
      ↓
ENV Available
      ↓
Application Reads ENV
```

---

ENV survives:

```text
Build

Image

Container
```

---

# Visibility Comparison

## ARG

Dockerfile

```dockerfile
ARG VERSION=1.0
```

---

Run Container

```bash
docker exec -it container bash
```

---

Check

```bash
env
```

---

Output

```text
VERSION Not Found
```

---

Reason

```text
ARG Not Available
Inside Container
```

---

## ENV

Dockerfile

```dockerfile
ENV APP_ENV=prod
```

---

Inside Container

```bash
env
```

---

Output

```text
APP_ENV=prod
```

---

Reason

```text
ENV Available
Inside Container
```

---

# Build-Time Example

Dockerfile

```dockerfile
ARG JAVA_VERSION=21

FROM openjdk:${JAVA_VERSION}
```

---

Build

```bash
docker build \
--build-arg JAVA_VERSION=17 .
```

---

Image uses:

```text
Java 17
```

---

Application never sees:

```text
JAVA_VERSION
```

---

# Runtime Example

Dockerfile

```dockerfile
ENV DB_HOST=mysql
```

---

Run

```bash
docker run \
-e DB_HOST=prod-mysql \
image
```

---

Application sees:

```text
DB_HOST=prod-mysql
```

---

# CI/CD Example

Jenkins

```groovy
docker build \
--build-arg VERSION=${BUILD_NUMBER}
```

---

Purpose

```text
Build Metadata
```

---

ARG is perfect.

---

Why?

```text
Needed Only During Build
```

---

# Kubernetes Example

Deployment

```yaml
env:
  - name: DB_HOST
    value: mongodb
```

---

Purpose

```text
Runtime Configuration
```

---

ENV is perfect.

---

Why?

```text
Application Reads It
```

---

# Production Example

Roboshop Catalogue

---

Build Time

```dockerfile
ARG NODE_VERSION=20
```

---

Used For

```text
NodeJS Version
```

---

Runtime

```dockerfile
ENV MONGO_HOST=mongodb
```

---

Used For

```text
Database Connection
```

---

# Common Mistake #1

Using ENV For Build Version

---

Bad Example

```dockerfile
ENV NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

Problem

```text
ENV Not Available Before FROM
```

---

Correct

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
```

---

# Common Mistake #2

Using ARG For Runtime Configuration

---

Bad Example

```dockerfile
ARG DB_HOST=mysql
```

---

Application Needs

```text
DB_HOST
```

---

Container Starts

---

Application Cannot See

```text
DB_HOST
```

---

Correct

```dockerfile
ENV DB_HOST=mysql
```

---

# Common Mistake #3

Using ARG For Secrets

---

Example

```dockerfile
ARG PASSWORD=secret123
```

---

Bad Practice.

---

Reason

```text
May Be Exposed
In Build History
```

---

# Common Mistake #4

Using ENV For Secrets

---

Example

```dockerfile
ENV PASSWORD=secret123
```

---

Bad Practice.

---

Reason

```text
Visible Through

docker inspect

Container Metadata
```

---

Use:

```text
Docker Secrets

Kubernetes Secrets

AWS Secrets Manager

HashiCorp Vault
```

---

# Combining ARG and ENV

Very Common Pattern.

---

Dockerfile

```dockerfile
ARG APP_VERSION=1.0

FROM node:20

ENV APP_VERSION=${APP_VERSION}
```

---

Build

```bash
docker build \
--build-arg APP_VERSION=2.0 .
```

---

Result

```text
APP_VERSION Available
Inside Container
```

---

Useful when:

```text
Build Information
Must Be Visible At Runtime
```

---

# Example

Build

```bash
docker build \
--build-arg VERSION=2.5.1 .
```

---

Dockerfile

```dockerfile
ARG VERSION

ENV APP_VERSION=${VERSION}
```

---

Container

```bash
echo $APP_VERSION
```

---

Output

```text
2.5.1
```

---

# Architecture

ARG

```text
Dockerfile
      ↓
Build
      ↓
Image
      ↓
Removed
```

---

ENV

```text
Dockerfile
      ↓
Image
      ↓
Container
      ↓
Application
```

---

# Side-by-Side Comparison

| Feature | ARG | ENV |
|----------|----------|----------|
| Purpose | Build-Time Variable | Runtime Variable |
| Available During Build | Yes | Yes |
| Available During Runtime | No | Yes |
| Available Inside Container | No | Yes |
| Can Be Overridden During Build | Yes | N/A |
| Can Be Overridden During Run | No | Yes |
| Typical Usage | Versions, Build Config | Application Config |
| Used In CI/CD | Yes | Sometimes |
| Used In Kubernetes | Rarely | Very Common |

---

# Interview Scenario

Question:

```text
I Need To Select
NodeJS Version
At Build Time.
Which Should I Use?
```

---

Answer:

```text
ARG
```

---

Question:

```text
I Need To Pass
Database Host
To Running Container.
Which Should I Use?
```

---

Answer:

```text
ENV
```

---

Question:

```text
Application Must Read Value
After Container Starts.
Which Should I Use?
```

---

Answer:

```text
ENV
```

---

Question:

```text
Value Needed Only During Image Build.
Which Should I Use?
```

---

Answer:

```text
ARG
```

---

# Common Interview Questions

## Difference Between ARG and ENV?

### Short Answer

ARG is used during image build, while ENV is used during container runtime.

### Detailed Explanation

ARG exists only during image creation. ENV remains available inside running containers.

### Production Example

ARG for NodeJS version and ENV for database host.

### Follow-Up Questions

- Which one survives after build?
- Which one is visible inside containers?

---

## Is ARG Available Inside Containers?

### Short Answer

No.

### Detailed Explanation

ARG values disappear after image creation.

### Production Example

Build version variable not available inside running container.

### Follow-Up Questions

- How do you expose build information?
- Can ARG be converted to ENV?

---

## Can ARG and ENV Be Used Together?

### Short Answer

Yes.

### Detailed Explanation

ARG can provide build-time values that are copied into ENV for runtime usage.

### Production Example

APP_VERSION passed during build and exposed at runtime.

### Follow-Up Questions

- Why use both?
- What is a practical use case?

---

# Key Takeaways

```text
ARG is a build-time variable.

ENV is a runtime variable.

ARG disappears after image creation.

ENV remains available inside running containers.

ARG is commonly used for versioning.

ENV is commonly used for application configuration.

ARG can be overridden using --build-arg.

ENV can be overridden using docker run -e.

Applications can read ENV values.

ARG and ENV can be combined for advanced use cases.

Understanding ARG vs ENV is essential for Docker interviews and production deployments.
```