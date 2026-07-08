# Dockerfile Fundamentals - FROM, RUN, CMD, LABEL, EXPOSE

## Introduction

Docker provides pre-built images like:

```text
nginx

ubuntu

mysql

node

python
```

---

However, in real projects we rarely use images directly.

Instead we create:

```text
Custom Images
```

for our applications.

---

Example:

```text
Catalogue Service

Cart Service

User Service

Frontend Service
```

---

To create custom images Docker uses:

```text
Dockerfile
```

---

# What is a Dockerfile?

A Dockerfile is:

```text
A Text File Containing Instructions
To Build A Docker Image
```

---

Docker reads instructions one by one and creates:

```text
Docker Image
```

---

Example

```dockerfile
FROM nginx

RUN echo "Hello Docker"

CMD ["nginx", "-g", "daemon off;"]
```

---

# Docker Build Process

Dockerfile

↓

docker build

↓

Docker Image

↓

docker run

↓

Docker Container

---

Architecture

```text
Dockerfile
      ↓
docker build
      ↓
Docker Image
      ↓
docker run
      ↓
Container
```

---

# Dockerfile Instructions

Most Common Instructions

```text
FROM

RUN

CMD

LABEL

EXPOSE
```

---

Additional Instructions (Future Topics)

```text
COPY

ADD

ENTRYPOINT

ENV

ARG

WORKDIR

USER
```

---

# FROM Instruction

## What is FROM?

FROM specifies:

```text
Base Image
```

for our Docker image.

---

It is usually:

```text
Operating System

Or

Existing Docker Image
```

---

Dockerfile must start with:

```text
FROM
```

---

Example

```dockerfile
FROM almalinux:9
```

---

Meaning:

```text
Use AlmaLinux 9
As Base Image
```

---

# Why FROM is Required?

Every image needs:

```text
Operating System

Libraries

Filesystem
```

---

FROM provides:

```text
Starting Point
```

for image creation.

---

# Examples

Ubuntu

```dockerfile
FROM ubuntu:24.04
```

---

Nginx

```dockerfile
FROM nginx:latest
```

---

NodeJS

```dockerfile
FROM node:20
```

---

Python

```dockerfile
FROM python:3.12
```

---

# FROM Workflow

```text
Base Image
      ↓
Add Packages
      ↓
Add Application
      ↓
Build Custom Image
```

---

# RUN Instruction

## What is RUN?

RUN executes commands:

```text
During Image Build
```

---

Used for:

```text
Package Installation

Configuration

User Creation

Directory Creation
```

---

# Example

```dockerfile
FROM almalinux:9

RUN dnf install nginx -y
```

---

Build Flow

```text
Build Starts
      ↓
RUN Executes
      ↓
Nginx Installed
      ↓
Image Created
```

---

# Multiple RUN Instructions

Allowed

```dockerfile
RUN dnf install nginx -y

RUN mkdir /app

RUN useradd roboshop
```

---

Each RUN creates:

```text
New Layer
```

---

# Why RUN is Important?

Used for:

```text
Software Installation

Package Updates

System Configuration

File Creation
```

---

# Production Example

```dockerfile
RUN npm install

RUN pip install flask

RUN yum install nginx -y
```

---

# CMD Instruction

## What is CMD?

CMD specifies:

```text
Default Command
```

executed when container starts.

---

RUN executes during:

```text
Image Build
```

---

CMD executes during:

```text
Container Start
```

---

# Example

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

---

When container starts:

```text
Nginx Starts
```

---

# Important Concept

RUN

```text
Build Time
```

---

CMD

```text
Run Time
```

---

Interview Favorite Question

```text
RUN vs CMD
```

---

# Why systemctl Does Not Work?

Example

```dockerfile
CMD ["systemctl", "start", "nginx"]
```

---

This generally fails.

---

Reason:

Containers do not run:

```text
Full Operating System
```

---

No:

```text
systemd

init process
```

available.

---

Therefore:

```text
systemctl
```

usually fails inside containers.

---

# Foreground Process Requirement

Container remains running only while:

```text
Main Process Running
```

---

Example

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

---

Meaning:

```text
Run Nginx In Foreground
```

---

# Why Foreground?

If CMD finishes:

```text
Container Stops
```

---

Example

```dockerfile
CMD ["echo", "hello"]
```

---

Output

```text
hello
```

---

Container exits immediately.

---

# CMD Rules

Only one CMD should exist.

---

Example

```dockerfile
CMD ["echo","hello"]

CMD ["nginx","-g","daemon off;"]
```

---

Result:

```text
Last CMD Wins
```

---

Docker uses:

```text
Second CMD Only
```

---

# RUN vs CMD

| Feature | RUN | CMD |
|----------|------|------|
| Execution Time | Image Build | Container Start |
| Creates Layer | Yes | No |
| Multiple Allowed | Yes | Technically Yes but Last One Used |
| Purpose | Configure Image | Start Container |

---

# LABEL Instruction

## What is LABEL?

LABEL adds:

```text
Metadata
```

to image.

---

Metadata means:

```text
Additional Information
```

---

Example

```dockerfile
LABEL author="Surendra"

LABEL environment="dev"

LABEL project="roboshop"
```

---

Used for:

```text
Documentation

Filtering

Automation

Image Management
```

---

# Production Example

```dockerfile
LABEL team="devops"

LABEL application="catalogue"

LABEL version="1.0"
```

---

# View Labels

```bash
docker inspect IMAGE_ID
```

---

Shows:

```text
Labels

Metadata
```

---

# EXPOSE Instruction

## What is EXPOSE?

EXPOSE provides information about:

```text
Container Port
```

used by application.

---

Example

```dockerfile
EXPOSE 80
```

---

Meaning:

```text
Application Uses Port 80
```

---

# Important Interview Point

EXPOSE:

```text
Does NOT Open Port
```

---

EXPOSE:

```text
Does NOT Publish Port
```

---

It only documents:

```text
Container Port Usage
```

---

# Example

```dockerfile
FROM nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

# Actual Port Publishing

Still Required:

```bash
docker run -p 80:80 nginx
```

---

EXPOSE alone:

```text
Cannot Make Application Accessible
```

---

# Sample Dockerfile

```dockerfile
FROM almalinux:9

LABEL project="roboshop"

RUN dnf install nginx -y

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

Build Process

```text
Base Image
      ↓
Install Nginx
      ↓
Add Metadata
      ↓
Declare Port
      ↓
Define Startup Command
      ↓
Image Created
```

---

# Real Production Example

Catalogue Service

```dockerfile
FROM node:20

LABEL application="catalogue"

RUN npm install

EXPOSE 8080

CMD ["node","server.js"]
```

---

Flow

```text
Docker Build
      ↓
Image Created
      ↓
Container Started
      ↓
NodeJS Application Running
```

---

# Common Interview Questions

## What is a Dockerfile?

### Short Answer

A Dockerfile is a text file containing instructions used to build Docker images.

### Detailed Explanation

Docker reads Dockerfile instructions sequentially and creates an image layer by layer.

### Production Example

Building a NodeJS application image.

### Follow-Up Questions

- How is Dockerfile used?
- What happens during docker build?

---

## What is FROM?

### Short Answer

FROM specifies the base image.

### Detailed Explanation

It provides the starting filesystem and environment for the custom image.

### Production Example

FROM almalinux:9.

### Follow-Up Questions

- Can Dockerfile have multiple FROM instructions?
- Why must FROM usually be first?

---

## Difference Between RUN and CMD?

### Short Answer

RUN executes during image build, CMD executes when the container starts.

### Detailed Explanation

RUN configures the image while CMD starts the application.

### Production Example

RUN installs nginx, CMD starts nginx.

### Follow-Up Questions

- Which instruction creates layers?
- Can CMD be overridden?

---

## Why Does systemctl Not Work Inside Containers?

### Short Answer

Containers do not run a full operating system with systemd.

### Detailed Explanation

Most containers run a single process and do not have init systems like traditional servers.

### Production Example

CMD ["systemctl","start","nginx"] fails inside containers.

### Follow-Up Questions

- What should be used instead?
- Why must applications run in foreground?

---

## What Does EXPOSE Do?

### Short Answer

EXPOSE documents the container port used by the application.

### Detailed Explanation

It does not actually publish or open the port.

### Production Example

EXPOSE 8080 for a NodeJS application.

### Follow-Up Questions

- Difference between EXPOSE and -p?
- Can EXPOSE make the application accessible?

---

# Key Takeaways

```text
Dockerfile is used to build custom Docker images.

FROM specifies the base image.

RUN executes commands during image build.

Each RUN creates a new image layer.

CMD defines the default container startup command.

CMD executes during container startup.

Containers stop when the main process exits.

LABEL adds metadata to images.

EXPOSE documents container ports.

EXPOSE does not publish ports.

RUN and CMD are among the most important Docker interview topics.
```