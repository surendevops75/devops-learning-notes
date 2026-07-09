# Docker ONBUILD

## Introduction

Most Docker images are built for:

```text
Direct Usage
```

Example:

```text
nginx

mysql

redis

node
```

---

You build the image.

↓

Run the container.

↓

Application starts.

---

However, some images are created to be:

```text
Base Images

Template Images

Golden Images

Reusable Platform Images
```

---

In such cases Docker provides:

```text
ONBUILD
```

---

# What is ONBUILD?

ONBUILD is a special Dockerfile instruction that:

```text
Registers An Instruction

To Execute Later
```

---

Important:

```text
ONBUILD Does NOT Execute
Immediately
```

---

Instead:

```text
ONBUILD Executes

When Another Image Uses It
As A Base Image
```

---

# Normal Dockerfile

Example

```dockerfile
FROM almalinux:9

COPY . /app
```

---

During build:

```text
COPY Executes Immediately
```

---

# ONBUILD Dockerfile

Example

```dockerfile
FROM almalinux:9

ONBUILD COPY . /app
```

---

During build:

```text
COPY Does NOT Execute
```

---

Docker stores:

```text
Pending Instruction
```

inside image metadata.

---

# ONBUILD Workflow

Parent Image

↓

Contains ONBUILD

↓

Image Built

↓

Another Dockerfile Uses It

↓

ONBUILD Executes

---

Architecture

```text
Parent Image
(Contains ONBUILD)
          ↓

Child Image
Uses Parent
          ↓

ONBUILD Executes
```

---

# Example

Parent Dockerfile

```dockerfile
FROM almalinux:9

ONBUILD COPY . /app
```

---

Build

```bash
docker build -t base-image:v1 .
```

---

Build completes.

---

Notice:

```text
COPY Never Executed
```

during this build.

---

# Child Dockerfile

Now create:

```dockerfile
FROM base-image:v1
```

---

Build

```bash
docker build -t app:v1 .
```

---

Now Docker automatically executes:

```dockerfile
COPY . /app
```

---

Even though:

```text
Child Dockerfile
Never Mentioned COPY
```

---

# Another Example

Parent Image

```dockerfile
FROM node:20

ONBUILD COPY . /app

ONBUILD RUN npm install
```

---

Build

```bash
docker build -t node-base:v1 .
```

---

No COPY.

No npm install.

---

Only ONBUILD metadata stored.

---

# Child Image

Dockerfile

```dockerfile
FROM node-base:v1
```

---

Build Starts.

---

Docker Automatically Executes:

```dockerfile
COPY . /app

RUN npm install
```

---

Result

```text
NodeJS Dependencies Installed
Automatically
```

---

# Why ONBUILD Exists?

Suppose Platform Team manages:

```text
100 NodeJS Applications
```

---

All applications need:

```text
COPY Source Code

npm install

Security Configuration
```

---

Without ONBUILD

Every Team Writes:

```dockerfile
COPY . /app

RUN npm install
```

---

Repeated everywhere.

---

Using ONBUILD

Parent Image:

```dockerfile
FROM node:20

ONBUILD COPY . /app

ONBUILD RUN npm install
```

---

Application Teams:

```dockerfile
FROM company-node-base:v1
```

---

Build automatically performs:

```text
COPY

npm install
```

---

# Production Use Case

Golden Images

---

Example

Security Team Creates:

```dockerfile
FROM node:20

RUN adduser appuser

ONBUILD COPY . /app

ONBUILD RUN npm install

ONBUILD RUN npm audit
```

---

Every application image automatically:

```text
Copies Source

Installs Packages

Runs Security Audit
```

---

# ONBUILD Supported Instructions

Examples

```dockerfile
ONBUILD COPY . /app

ONBUILD RUN npm install

ONBUILD ENV APP_ENV=dev
```

---

Docker stores:

```text
Future Instructions
```

---

# Inspect ONBUILD Instructions

Command

```bash
docker inspect image-name
```

---

Output contains:

```text
OnBuild
```

metadata.

---

Example

```text
"OnBuild": [
    "COPY . /app",
    "RUN npm install"
]
```

---

# Common Mistake

Parent Image

```dockerfile
FROM node:20

ONBUILD COPY . /app
```

---

Child Build Context

```text
Empty
```

---

Build fails.

---

Reason

```text
ONBUILD COPY Needs Files
```

---

# Nested ONBUILD

Parent

```dockerfile
ONBUILD COPY . /app
```

---

Child Uses Parent.

---

ONBUILD Executes.

---

Grandchild Uses Child.

---

ONBUILD Does Not Execute Again.

---

Reason

```text
ONBUILD Triggers
Only Once
```

---

# Why ONBUILD Is Less Popular Today?

Modern DevOps uses:

```text
Reusable CI/CD Pipelines

Shared GitHub Actions

Shared Jenkins Libraries

Build Templates
```

---

Instead of:

```text
ONBUILD Images
```

---

Therefore:

```text
ONBUILD Is Less Common
```

today.

---

However:

```text
Interview Questions
```

still appear frequently.

---

# ONBUILD vs RUN

RUN

```dockerfile
RUN npm install
```

---

Executes:

```text
Immediately
```

during image build.

---

ONBUILD RUN

```dockerfile
ONBUILD RUN npm install
```

---

Executes:

```text
Later
```

when child image is built.

---

# ONBUILD vs COPY

COPY

```dockerfile
COPY . /app
```

---

Executes:

```text
Current Build
```

---

ONBUILD COPY

```dockerfile
ONBUILD COPY . /app
```

---

Executes:

```text
Future Build
```

---

# Production Example

Parent Image

```dockerfile
FROM node:20

WORKDIR /app

ONBUILD COPY . .

ONBUILD RUN npm install
```

---

Application Team

```dockerfile
FROM company-node-base:v1

CMD ["node","server.js"]
```

---

Build Process

```text
Parent Image
        ↓
Child Image Build
        ↓
COPY Source
        ↓
npm install
        ↓
Image Ready
```

---

# Common Interview Questions

## What is ONBUILD?

### Short Answer

ONBUILD registers instructions that execute when another image uses the current image as its base image.

### Detailed Explanation

Instead of running immediately, ONBUILD instructions are stored in image metadata and triggered during child image builds.

### Production Example

ONBUILD COPY . /app

### Follow-Up Questions

- When does ONBUILD execute?
- Where are ONBUILD instructions stored?

---

## Does ONBUILD Execute During Current Build?

### Short Answer

No.

### Detailed Explanation

ONBUILD instructions are stored and executed only when a child image uses the image as a base.

### Production Example

Parent image contains ONBUILD RUN npm install.

### Follow-Up Questions

- What triggers ONBUILD?
- Can ONBUILD execute multiple times?

---

## Why Is ONBUILD Used?

### Short Answer

To create reusable base images.

### Detailed Explanation

ONBUILD allows common build steps to be automatically applied to child images.

### Production Example

Golden NodeJS base image.

### Follow-Up Questions

- Is ONBUILD commonly used today?
- What alternatives exist?

---

## Is ONBUILD Popular In Modern DevOps?

### Short Answer

Not very common.

### Detailed Explanation

Most organizations now use reusable CI/CD pipelines, templates, and shared build frameworks instead of ONBUILD images.

### Production Example

GitHub Actions templates replacing ONBUILD-based workflows.

### Follow-Up Questions

- Why has usage decreased?
- What replaced ONBUILD?

---

# Key Takeaways

```text
ONBUILD registers instructions for future execution.

ONBUILD instructions do not execute immediately.

ONBUILD executes when a child image uses the image as a base.

ONBUILD is commonly used for reusable base images.

ONBUILD instructions are stored in image metadata.

ONBUILD triggers only once.

ONBUILD helps create golden images.

Modern DevOps often replaces ONBUILD with CI/CD templates and shared pipelines.

ONBUILD remains an important Docker interview topic.
```