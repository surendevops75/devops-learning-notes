# Docker Introduction and Architecture

## Introduction

Containerization became popular because modern applications moved from:

```text
Monolithic Applications
        ↓
Microservices
```

---

Microservices introduced challenges:

```text
Application Dependencies

Runtime Differences

Environment Inconsistencies

Deployment Complexity
```

---

Example:

Application works on:

```text
Developer Laptop
```

but fails on:

```text
QA Server

Production Server
```

---

Reason:

```text
Different Environment

Different Libraries

Different Runtime Versions
```

---

Docker solves this problem.

---

# What is Docker?

Docker is an:

```text
Open Source Containerization Platform
```

used to:

```text
Build

Package

Ship

Run Applications
```

inside containers.

---

Docker packages:

```text
Application

Libraries

Dependencies

Runtime
```

into a:

```text
Docker Image
```

---

The image can run anywhere Docker is installed.

---

# Why Docker?

Before Docker:

```text
Developer Machine
       ↓
QA Server
       ↓
Production
```

---

Each environment may have:

```text
Different OS

Different Java Version

Different Python Version

Different Libraries
```

---

Result:

```text
Works On My Machine Problem
```

---

Docker eliminates this issue.

---

# Docker Goal

```text
Build Once

Run Anywhere
```

---

Same image runs on:

```text
Laptop

EC2

Data Center

Kubernetes

Cloud
```

---

# Docker Architecture Overview

Docker follows a:

```text
Client Server Architecture
```

---

Components:

```text
Docker Client

Docker Daemon

Docker Images

Docker Containers

Docker Registry
```

---

Architecture

```text
Docker Client
       ↓
Docker Daemon
       ↓
Images
       ↓
Containers
```

---

# Docker Client

Docker Client is the command line interface.

---

Examples:

```bash
docker run nginx

docker ps

docker images

docker stop
```

---

Whenever we execute:

```bash
docker run nginx
```

the request goes to:

```text
Docker Daemon
```

---

# Docker Daemon

Docker Daemon is the main Docker service.

---

Process:

```text
dockerd
```

---

Responsibilities:

```text
Manage Images

Manage Containers

Manage Networks

Manage Volumes
```

---

Without Docker Daemon:

```text
Docker Commands Will Not Work
```

---

# Docker Engine

Docker Engine consists of:

```text
Docker Client

Docker Daemon

REST API
```

---

Architecture

```text
Docker Client
       ↓
REST API
       ↓
Docker Daemon
```

---

# Docker Registry

Registry stores Docker Images.

---

Examples:

```text
Docker Hub

AWS ECR

Azure ACR

Google Artifact Registry

JFrog Artifactory
```

---

Think of Registry as:

```text
GitHub For Docker Images
```

---

# Docker Hub

Default Public Registry.

---

Example:

```bash
docker pull nginx
```

---

Docker downloads image from:

```text
Docker Hub
```

---

# Docker Image

Docker Image is:

```text
Read Only Template
```

used to create containers.

---

Contains:

```text
Application Code

Runtime

Libraries

Dependencies
```

---

Examples:

```text
nginx

ubuntu

node

python

mysql
```

---

Image is:

```text
Static
```

---

Cannot execute directly.

---

# Docker Container

Container is:

```text
Running Instance Of Docker Image
```

---

Example:

```text
Image
 ↓
Container
```

---

Similar To:

```text
AMI
 ↓
EC2 Instance
```

---

# AMI vs Docker Image

AMI:

```text
Operating System

System Packages

Runtime

Application

Libraries
```

---

Typical Size:

```text
Several GBs
```

---

Docker Image:

```text
Minimal OS

Runtime

Application

Libraries
```

---

Typical Size:

```text
Few MBs To Few Hundred MBs
```

---

# Image to Container Flow

```text
Docker Image
       ↓
docker run
       ↓
Docker Container
```

---

Example:

```bash
docker run nginx
```

---

Flow:

```text
Pull Image
      ↓
Create Container
      ↓
Start Container
```

---

# Docker Workflow

Step 1

```bash
docker pull nginx
```

---

Downloads image.

---

Step 2

```bash
docker create nginx
```

---

Creates container.

---

Step 3

```bash
docker start CONTAINER_ID
```

---

Starts container.

---

# Simplified Command

Instead of:

```bash
docker pull nginx

docker create nginx

docker start CONTAINER_ID
```

---

Use:

```bash
docker run nginx
```

---

Equivalent To:

```text
Pull
 ↓
Create
 ↓
Start
```

---

# Container Runtime

Docker does not run containers directly.

---

Modern Docker uses:

```text
containerd
```

---

containerd manages:

```text
Container Lifecycle

Container Execution

Container Resources
```

---

Architecture

```text
Docker Client
       ↓
Docker Daemon
       ↓
containerd
       ↓
Containers
```

---

# Why Docker Became Popular?

Advantages:

```text
Portable

Lightweight

Fast Startup

Efficient Resource Usage

Consistent Deployments
```

---

# Production Example

Roboshop Microservices:

```text
Catalogue

Cart

User

Shipping

Payment
```

---

Without Docker:

```text
Install Runtime

Install Dependencies

Deploy Application
```

on every server.

---

With Docker:

```text
Build Image Once
```

---

Deploy Anywhere:

```text
EC2

EKS

Kubernetes

Docker Hosts
```

---

# Real World Example

Developer builds:

```text
catalogue:v1
```

---

Pushes image to:

```text
AWS ECR
```

---

Deployment flow:

```text
Developer
      ↓
Docker Build
      ↓
Docker Image
      ↓
ECR
      ↓
EKS
      ↓
Container Running
```

---

# Docker Architecture Summary

```text
Docker Client
       ↓
Docker Daemon
       ↓
Docker Registry
       ↓
Docker Images
       ↓
Docker Containers
```

---

# Common Interview Questions

## What is Docker?

### Short Answer

Docker is an open-source containerization platform used to build and run applications in containers.

### Detailed Explanation

Docker packages applications and dependencies into portable containers that run consistently across environments.

### Production Example

Running Roboshop microservices as Docker containers.

### Follow-Up Questions

- Why is Docker popular?
- What problems does Docker solve?

---

## What is Docker Architecture?

### Short Answer

Docker uses a client-server architecture consisting of Docker Client, Docker Daemon, Images, Containers, and Registries.

### Detailed Explanation

The Docker Client sends commands to the Docker Daemon, which manages images and containers.

### Production Example

Executing docker run nginx to create a running container.

### Follow-Up Questions

- What is Docker Daemon?
- What is Docker Engine?

---

## What is the Difference Between Image and Container?

### Short Answer

An image is a template, while a container is a running instance of that image.

### Detailed Explanation

Images are static and immutable. Containers are running processes created from images.

### Production Example

nginx image creating an nginx container.

### Follow-Up Questions

- Can multiple containers use the same image?
- Can containers exist without images?

---

## What Happens When We Run docker run nginx?

### Short Answer

Docker pulls the image (if not available), creates a container, and starts it.

### Detailed Explanation

docker run combines image download, container creation, and container startup into a single command.

### Production Example

Starting an nginx web server container.

### Follow-Up Questions

- Difference between docker create and docker run?
- Difference between docker start and docker run?

---

# Key Takeaways

```text
Docker is a containerization platform.

Docker solves environment consistency problems.

Docker follows a client-server architecture.

Docker Client communicates with Docker Daemon.

Docker Images are templates used to create containers.

Containers are running instances of images.

Docker Hub is the default public registry.

Docker Engine consists of Client, Daemon, and REST API.

containerd is responsible for container execution.

Docker enables Build Once, Run Anywhere deployments.
```