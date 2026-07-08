# Dockerfile Best Practices

## Introduction

A Dockerfile can always be written in many ways.

Example:

```text
Dockerfile A
```

and

```text
Dockerfile B
```

may both work correctly.

---

However:

```text
Not Every Working Dockerfile
Is Production Ready
```

---

Production Dockerfiles should be:

```text
Small

Secure

Fast

Maintainable

Efficient
```

---

This file covers industry-standard Dockerfile best practices used in:

```text
Kubernetes

EKS

OpenShift

CI/CD Pipelines

Production Deployments
```

---

# Why Dockerfile Best Practices Matter?

Bad Dockerfiles create:

```text
Large Images

Slow Builds

Security Risks

Wasted Storage

Long Deployments
```

---

Good Dockerfiles provide:

```text
Faster Builds

Smaller Images

Better Security

Lower Costs
```

---

# Best Practice 1

## Use Small Base Images

Bad

```dockerfile
FROM ubuntu:latest
```

---

Why?

```text
Large Image Size

Unnecessary Packages
```

---

Better

```dockerfile
FROM node:20-alpine
```

---

Benefits

```text
Smaller Image

Faster Download

Less Attack Surface
```

---

# Example

Ubuntu Based

```text
300MB+
```

---

Alpine Based

```text
Less Than 20MB
```

---

# Best Practice 2

## Avoid latest Tag

Bad

```dockerfile
FROM nginx:latest
```

---

Problem

```text
Build Today
≠
Build Tomorrow
```

---

New Version Released:

```text
Image Changes Automatically
```

---

Better

```dockerfile
FROM nginx:1.27
```

---

Benefits

```text
Predictable Builds

Stable Deployments
```

---

# Best Practice 3

## Minimize Layers

Every Dockerfile instruction creates:

```text
Image Layer
```

---

Bad

```dockerfile
RUN dnf install nginx -y

RUN dnf install git -y

RUN dnf install curl -y
```

---

Creates:

```text
3 Layers
```

---

Better

```dockerfile
RUN dnf install nginx git curl -y
```

---

Creates:

```text
1 Layer
```

---

Benefits

```text
Smaller Images

Faster Builds
```

---

# Best Practice 4

## Remove Temporary Files

Bad

```dockerfile
RUN curl file.zip

RUN unzip file.zip
```

---

Problem

```text
Temporary Files Remain
```

---

Better

```dockerfile
RUN curl file.zip && \
    unzip file.zip && \
    rm -f file.zip
```

---

Benefits

```text
Less Disk Usage
```

---

# Best Practice 5

## Use COPY Instead Of ADD

ADD

```dockerfile
ADD app.tar.gz /app
```

---

COPY

```dockerfile
COPY app.tar.gz /app
```

---

Rule

```text
Use COPY By Default
```

---

Use ADD only when:

```text
Archive Extraction Needed

URL Download Needed
```

---

Benefits

```text
Predictable Builds

Cleaner Dockerfiles
```

---

# Best Practice 6

## Run Containers As Non-Root

Bad

```dockerfile
FROM node:20

CMD ["node","server.js"]
```

---

Default User

```text
root
```

---

Security Risk

```text
Container Compromise
      ↓
Root Privileges
```

---

Better

```dockerfile
RUN useradd appuser

USER appuser
```

---

Benefits

```text
Improved Security
```

---

# Best Practice 7

## Use .dockerignore

Project

```text
node_modules/

.git/

logs/

tmp/
```

---

Bad

```dockerfile
COPY . .
```

---

Problem

```text
Everything Copied
```

---

Including:

```text
Git History

Logs

Temporary Files
```

---

Solution

```text
.dockerignore
```

---

Example

```text
.git

node_modules

*.log
```

---

Benefits

```text
Smaller Build Context

Faster Build
```

---

# Best Practice 8

## Order Instructions Properly

Docker uses:

```text
Layer Cache
```

---

Bad

```dockerfile
COPY . .

RUN npm install
```

---

Every code change:

```text
npm install Executes Again
```

---

Better

```dockerfile
COPY package.json .

RUN npm install

COPY . .
```

---

Benefits

```text
Cache Reused

Faster Builds
```

---

# Best Practice 9

## Use Specific Versions

Bad

```dockerfile
RUN pip install flask
```

---

Problem

```text
Version Changes
```

---

Better

```dockerfile
RUN pip install flask==3.0.0
```

---

Benefits

```text
Reproducible Builds
```

---

# Best Practice 10

## Keep One Responsibility Per Container

Bad

```dockerfile
CMD nginx

CMD mysql

CMD java
```

---

Container doing:

```text
Too Many Things
```

---

Better

```text
Nginx Container

Database Container

Application Container
```

---

Benefits

```text
Scalability

Maintainability

Observability
```

---

# Best Practice 11

## Use Multi-Stage Builds

Bad

```text
Build Tools
```

remain inside final image.

---

Result

```text
Large Image
```

---

Better

```text
Builder Stage

Runtime Stage
```

---

Benefits

```text
Small Image

Better Security
```

---

Dedicated file later:

```text
18-Multi-Stage-Builds.md
```

---

# Best Practice 12

## Avoid Hardcoded Configuration

Bad

```dockerfile
ENV DB_HOST=prod-db
```

---

Problem

```text
Environment Specific
```

---

Better

```text
Pass During Runtime
```

---

Examples

```bash
docker run -e DB_HOST=mysql
```

---

Kubernetes

```yaml
env:
```

---

Benefits

```text
Reusable Images
```

---

# Best Practice 13

## Don't Store Secrets In Images

Bad

```dockerfile
ENV PASSWORD=password123
```

---

Bad

```dockerfile
ARG PASSWORD=password123
```

---

Risks

```text
Visible In Metadata

Visible In History

Visible In Image
```

---

Use

```text
AWS Secrets Manager

HashiCorp Vault

Kubernetes Secrets

Docker Secrets
```

---

# Best Practice 14

## Add Metadata

Example

```dockerfile
LABEL maintainer="devops-team"

LABEL project="roboshop"

LABEL version="1.0"
```

---

Benefits

```text
Traceability

Automation

Inventory Management
```

---

# Best Practice 15

## Keep Dockerfiles Readable

Bad

```dockerfile
RUN yum install nginx -y && mkdir /app && useradd test && chmod 777 /app
```

---

Hard To Read.

---

Better

```dockerfile
RUN yum install nginx -y && \
    mkdir /app && \
    useradd test && \
    chmod 755 /app
```

---

Benefits

```text
Maintainability
```

---

# Production Example

Roboshop Catalogue Service

Bad

```dockerfile
FROM node:latest

COPY . .

RUN npm install

CMD ["node","server.js"]
```

---

Issues

```text
Latest Tag

Poor Caching

Large Build Context
```

---

Improved

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

USER node

CMD ["node","server.js"]
```

---

Benefits

```text
Smaller

Safer

Faster
```

---

# Production Checklist

Before pushing image:

```text
✓ Fixed Version Tags

✓ Small Base Image

✓ Non-root User

✓ Secrets Removed

✓ .dockerignore Present

✓ Layer Optimization

✓ Minimal Dependencies

✓ Readable Dockerfile
```

---

# Common Interview Questions

## Why Avoid latest Tag?

### Short Answer

latest makes builds unpredictable.

### Detailed Explanation

The image behind latest may change at any time, causing different build results.

### Production Example

nginx:latest pulling a newer version unexpectedly.

### Follow-Up Questions

- What should be used instead?
- Why is reproducibility important?

---

## Why Use Non-Root Users?

### Short Answer

Improves container security.

### Detailed Explanation

If a container is compromised, the attacker gains limited privileges.

### Production Example

Running NodeJS applications as appuser instead of root.

### Follow-Up Questions

- How do you create users?
- Why is root dangerous?

---

## Why Use .dockerignore?

### Short Answer

Reduces build context size.

### Detailed Explanation

Prevents unnecessary files from being copied during builds.

### Production Example

Excluding node_modules and .git directories.

### Follow-Up Questions

- What happens without .dockerignore?
- Does it improve build speed?

---

## What Are Docker Layers?

### Short Answer

Layers are reusable filesystem changes created by Dockerfile instructions.

### Detailed Explanation

Reducing layers improves image size and build efficiency.

### Production Example

Combining package installations into a single RUN instruction.

### Follow-Up Questions

- Which instructions create layers?
- How does Docker caching work?

---

# Key Takeaways

```text
Use small base images whenever possible.

Avoid latest tags in production.

Minimize image layers.

Remove temporary files.

Prefer COPY over ADD.

Run containers as non-root users.

Use .dockerignore.

Optimize instruction order for caching.

Pin dependency versions.

Keep one responsibility per container.

Avoid storing secrets in images.

Use labels for metadata.

Write readable and maintainable Dockerfiles.

Production Dockerfiles focus on security, performance, and reproducibility.
```