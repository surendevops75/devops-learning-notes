# Dockerfile COPY, ADD, ENV and ARG

## Introduction

After learning:

```text
FROM

RUN

CMD

LABEL

EXPOSE
```

the next important Dockerfile instructions are:

```text
COPY

ADD

ENV

ARG
```

---

These instructions are heavily used in:

```text
Application Containerization

CI/CD Pipelines

Kubernetes Deployments

Production Docker Images
```

---

# COPY Instruction

## What is COPY?

COPY is used to:

```text
Copy Files

Copy Directories
```

from:

```text
Host Machine
```

to:

```text
Docker Image
```

during image build.

---

Syntax

```dockerfile
COPY <source> <destination>
```

---

Example

```dockerfile
COPY app.py /app/
```

---

Meaning

```text
Copy app.py

To

/app directory inside image
```

---

# Real Example

Project Structure

```text
project/
│
├── Dockerfile
├── app.py
└── requirements.txt
```

---

Dockerfile

```dockerfile
FROM python:3.12

COPY app.py /app/

COPY requirements.txt /app/
```

---

Build

```bash
docker build -t python-app:v1 .
```

---

Result

```text
Files Become Part Of Image
```

---

# Copy Entire Directory

Example

```dockerfile
COPY . /app
```

---

Meaning

```text
Copy Everything

From Current Directory

Into /app
```

---

Very common in projects.

---

# Why COPY is Important?

Applications need:

```text
Source Code

Configuration Files

Scripts

Libraries
```

inside image.

---

COPY makes this possible.

---

# ADD Instruction

## What is ADD?

ADD is similar to:

```text
COPY
```

but has additional capabilities.

---

Syntax

```dockerfile
ADD <source> <destination>
```

---

Example

```dockerfile
ADD app.tar.gz /app/
```

---

# Difference Between COPY and ADD

COPY

```text
Only Copies Files
```

---

ADD

```text
Copies Files

Downloads URLs

Extracts Archives
```

---

# Capability 1

Download Content From Internet

Example

```dockerfile
ADD https://example.com/app.zip /tmp/
```

---

Build Process

```text
Docker Build
       ↓
Download File
       ↓
Store Inside Image
```

---

# Capability 2

Auto Extract Archives

Example

```dockerfile
ADD application.tar.gz /app/
```

---

Docker automatically:

```text
Extracts Archive
```

during build.

---

COPY cannot do this.

---

# Why COPY is Preferred?

Although ADD has extra features:

```text
COPY Is Recommended
```

for normal file copying.

---

Reason

```text
More Predictable

Less Complexity

Better Security
```

---

Docker Best Practice

```text
Use COPY

Use ADD Only When Required
```

---

# ENV Instruction

## What is ENV?

ENV defines:

```text
Environment Variables
```

inside image and container.

---

Syntax

```dockerfile
ENV KEY=VALUE
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

# Multiple ENV Variables

Example

```dockerfile
ENV APP_ENV=dev

ENV PORT=8080

ENV DATABASE=mysql
```

---

# Verify ENV

Inside container

```bash
env
```

---

Example Output

```text
APP_ENV=dev

PORT=8080
```

---

# Production Example

```dockerfile
ENV DB_HOST=mysql

ENV DB_PORT=3306

ENV LOG_LEVEL=INFO
```

---

Application reads:

```text
Database Details

Application Configuration

Runtime Settings
```

from environment variables.

---

# Why ENV is Important?

Used heavily in:

```text
Kubernetes

Docker Compose

CI/CD Pipelines
```

---

Avoids:

```text
Hard Coding Values
```

---

# ARG Instruction

## What is ARG?

ARG defines:

```text
Build Time Variables
```

---

Unlike ENV:

```text
Available Only During Build
```

---

Syntax

```dockerfile
ARG VERSION=1.0
```

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

Build

```bash
docker build --build-arg VERSION=8 .
```

---

Result

```text
Uses almalinux:8
```

---

# Special Case

Normally:

```text
FROM First Instruction
```

---

Exception:

```text
ARG Can Appear Before FROM
```

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

Used for:

```text
Dynamic Base Images
```

---

# Build Argument Example

Dockerfile

```dockerfile
ARG APP_VERSION=1.0

FROM almalinux:9

RUN echo ${APP_VERSION}
```

---

Build

```bash
docker build \
--build-arg APP_VERSION=2.0 \
-t app:v2 .
```

---

Result

```text
APP_VERSION=2.0
```

during build.

---

# ARG Scope

Very Important Interview Question

---

Example

```dockerfile
ARG VERSION=9

FROM almalinux:${VERSION}
```

---

After FROM:

```text
ARG Not Automatically Available
```

---

Need:

```dockerfile
ARG VERSION
```

again if required later.

---

# ARG vs ENV

| Feature | ARG | ENV |
|-----------|---------|---------|
| Available During Build | Yes | Yes |
| Available During Runtime | No | Yes |
| Visible Inside Container | No | Yes |
| Can Be Overridden During Build | Yes | No |
| Typical Usage | Build Variables | Runtime Configuration |

---

# Example

ARG

```dockerfile
ARG VERSION=9
```

Used for:

```text
Image Version

Package Version

Build Decisions
```

---

ENV

```dockerfile
ENV DB_HOST=mysql
```

Used for:

```text
Application Configuration
```

---

# Real Production Example

NodeJS Application

```dockerfile
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}

ENV APP_ENV=prod

COPY . /app
```

---

Build

```bash
docker build \
--build-arg NODE_VERSION=22 \
-t catalogue:v1 .
```

---

Runtime

```text
APP_ENV=prod
```

available.

---

NODE_VERSION

```text
Not Available
```

inside container.

---

# Common Interview Questions

## Difference Between COPY and ADD?

### Short Answer

COPY only copies files. ADD can copy files, download URLs, and extract archives.

### Detailed Explanation

COPY is preferred for predictable file transfers. ADD should be used only when its extra capabilities are needed.

### Production Example

COPY application source code into image.

### Follow-Up Questions

- Can COPY extract tar files?
- Why is COPY preferred?

---

## What is ENV?

### Short Answer

ENV defines environment variables available inside containers.

### Detailed Explanation

ENV stores runtime configuration values used by applications.

### Production Example

Database host and application environment configuration.

### Follow-Up Questions

- How do you view environment variables?
- Can ENV be overridden?

---

## What is ARG?

### Short Answer

ARG defines build-time variables.

### Detailed Explanation

ARG values are available during image creation but are not available inside running containers.

### Production Example

Choosing NodeJS version during build.

### Follow-Up Questions

- Can ARG be used before FROM?
- Why is ARG not visible inside containers?

---

## Difference Between ARG and ENV?

### Short Answer

ARG is for build time. ENV is for runtime.

### Detailed Explanation

ARG disappears after image creation, while ENV remains available inside containers.

### Production Example

ARG for image version selection and ENV for database configuration.

### Follow-Up Questions

- Which one is used in Kubernetes?
- Which one is visible inside containers?

---

# Key Takeaways

```text
COPY copies files from host to image.

ADD provides COPY functionality plus URL download and archive extraction.

COPY is preferred over ADD for normal file transfers.

ENV defines runtime environment variables.

ENV variables remain available inside containers.

ARG defines build-time variables.

ARG variables are not available inside running containers.

ARG can be used before FROM for dynamic base images.

ARG and ENV serve different purposes.

Understanding COPY, ADD, ENV and ARG is essential for writing production-grade Dockerfiles.
```