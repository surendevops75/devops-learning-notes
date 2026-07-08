# Docker Multi-Stage Builds

## Introduction

One of the biggest problems in Docker is:

```text
Large Image Size
```

---

Many applications require:

```text
Build Tools

Compilers

SDKs

Dependencies
```

during build.

---

However:

```text
These Tools Are Not Required
In Production
```

---

Yet traditional Docker images often contain:

```text
Build Tools

Source Code

Temporary Files

Caches
```

---

Result:

```text
Large Images

Security Risks

Slower Deployments
```

---

Docker solves this using:

```text
Multi-Stage Builds
```

---

# What is a Multi-Stage Build?

A Multi-Stage Build allows:

```text
Multiple FROM Instructions
```

inside a single Dockerfile.

---

Each FROM creates:

```text
Separate Build Stage
```

---

Typical Flow

```text
Build Stage
      ↓
Compile Application
      ↓
Copy Required Files
      ↓
Runtime Stage
      ↓
Run Application
```

---

# Why Multi-Stage Builds?

Suppose:

```text
Java Application
```

requires:

```text
Maven

JDK

Source Code
```

for building.

---

Production only needs:

```text
JRE

Application JAR
```

---

Keeping Maven and Source Code:

```text
Wastes Space
```

---

Multi-stage builds remove:

```text
Build Dependencies
```

from the final image.

---

# Traditional Build

Dockerfile

```dockerfile
FROM maven:3.9

COPY . .

RUN mvn clean package

CMD ["java","-jar","target/app.jar"]
```

---

Problem

Final Image Contains

```text
Maven

JDK

Source Code

Build Cache

Application
```

---

Image Size

```text
Very Large
```

---

# Multi-Stage Build Architecture

```text
Stage 1
--------
Build Application

        ↓

Stage 2
--------
Run Application
```

---

Only required files move:

```text
Build Stage
      ↓
Runtime Stage
```

---

# Basic Example

```dockerfile
FROM almalinux:9 AS builder

RUN dnf install zip -y

RUN touch app.txt

FROM almalinux:9

COPY --from=builder /app.txt /app.txt
```

---

Result

```text
zip Package Not Present

Only app.txt Copied
```

---

# Stage Naming

Example

```dockerfile
FROM node:20 AS builder
```

---

Meaning

```text
Create Stage Named

builder
```

---

Later

```dockerfile
COPY --from=builder
```

can reference it.

---

# How COPY --from Works

Example

```dockerfile
COPY --from=builder \
/app/dist \
/usr/share/nginx/html
```

---

Meaning

```text
Copy Files

From Build Stage

To Runtime Stage
```

---

# NodeJS Example

Application

```text
React Frontend
```

---

Requires

```text
NodeJS

npm
```

during build.

---

Final Application

```text
Static HTML

CSS

JavaScript
```

---

Dockerfile

```dockerfile
FROM node:20 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:1.27

COPY --from=builder \
/app/dist \
/usr/share/nginx/html
```

---

Result

Final Image Contains

```text
Nginx

Static Files
```

---

Not Included

```text
NodeJS

npm

Source Code
```

---

# Java Example

Stage 1

```dockerfile
FROM maven:3.9 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package
```

---

Stage 2

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder \
/app/target/app.jar \
app.jar

CMD ["java","-jar","app.jar"]
```

---

Final Image Contains

```text
JRE

JAR File
```

---

Not Included

```text
Maven

Source Code

Build Cache
```

---

# Go Example

Build Stage

```dockerfile
FROM golang:1.24 AS builder

WORKDIR /app

COPY . .

RUN go build -o app
```

---

Runtime Stage

```dockerfile
FROM alpine:3.22

COPY --from=builder \
/app/app \
/app

CMD ["/app"]
```

---

Result

```text
Extremely Small Image
```

---

# Real Production Example

Roboshop Frontend

---

Build Stage

```text
NodeJS

npm install

npm run build
```

---

Runtime Stage

```text
Nginx

Static Files
```

---

Final Image

```text
Smaller

Faster

More Secure
```

---

# Benefits of Multi-Stage Builds

## Smaller Images

Before

```text
1.2 GB
```

---

After

```text
120 MB
```

---

## Better Security

Remove

```text
Compilers

Package Managers

Source Code
```

---

Attack surface reduced.

---

## Faster Deployments

Smaller Image

↓

Less Download Time

↓

Faster Startup

---

## Lower Registry Storage

Smaller images consume:

```text
Less ECR Storage

Less Docker Hub Storage
```

---

# Security Advantage

Bad

```text
Source Code
```

inside image.

---

Attacker gains access.

---

Good

```text
Only Compiled Artifact
```

inside image.

---

Much safer.

---

# Multi-Stage Build Flow

```text
Developer
      ↓
Build Stage
      ↓
Compile Application
      ↓
Copy Artifact
      ↓
Runtime Stage
      ↓
Final Image
```

---

# Common Mistakes

## Copy Entire Workspace

Bad

```dockerfile
COPY . .
```

into runtime stage.

---

Result

```text
Source Code Included
```

---

Avoid whenever possible.

---

## Using Build Image In Production

Bad

```dockerfile
FROM maven:3.9
```

as final image.

---

Contains

```text
Maven

Build Tools
```

---

Unnecessary.

---

# CI/CD Example

Jenkins Pipeline

```groovy
docker build \
-t catalogue:v1 .
```

---

Multi-stage build automatically:

```text
Builds

Compiles

Packages

Produces Runtime Image
```

---

No extra scripts needed.

---

# Architecture Diagram

```text
Stage 1 (Builder)
----------------
Source Code
      ↓
Build
      ↓
Artifact

        ↓

Stage 2 (Runtime)
-----------------
Artifact
      ↓
Application Running
```

---

# Interview Questions

## What is a Multi-Stage Build?

### Short Answer

A Docker feature that uses multiple FROM instructions to separate build and runtime environments.

### Detailed Explanation

Applications are built in one stage and only required artifacts are copied into the final image.

### Production Example

Building a Java application using Maven and running it using JRE.

### Follow-Up Questions

- Why are multi-stage builds important?
- How do you copy files between stages?

---

## Why Use Multi-Stage Builds?

### Short Answer

To reduce image size and improve security.

### Detailed Explanation

Build tools remain in the build stage and are excluded from the final image.

### Production Example

React build using NodeJS and deployment using Nginx.

### Follow-Up Questions

- What happens without multi-stage builds?
- How much image size reduction is possible?

---

## What Does COPY --from Do?

### Short Answer

Copies files from one build stage to another.

### Detailed Explanation

It allows artifacts generated in a builder stage to be transferred into the final runtime image.

### Production Example

Copying app.jar from Maven build stage into JRE runtime image.

### Follow-Up Questions

- Can stages be named?
- Can files be copied between any stages?

---

# Key Takeaways

```text
Multi-stage builds use multiple FROM instructions.

Each FROM creates a separate stage.

Build tools stay in builder stages.

Only required artifacts move to runtime stages.

COPY --from transfers files between stages.

Multi-stage builds reduce image size.

Multi-stage builds improve security.

Multi-stage builds speed up deployments.

React, Java, Go, and NodeJS applications commonly use multi-stage builds.

Multi-stage builds are considered a production best practice.
```