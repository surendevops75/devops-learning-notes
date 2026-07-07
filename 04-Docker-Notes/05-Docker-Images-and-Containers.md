# Docker Images and Containers

## Introduction

Docker revolves around two fundamental concepts:

```text
Docker Images

Docker Containers
```

---

Understanding these two concepts is essential because every Docker workflow follows:

```text
Image
  ↓
Container
```

---

Without Images:

```text
Containers Cannot Be Created
```

---

# Real World Analogy

Think of:

```text
Architectural Blueprint
```

and

```text
House
```

---

Blueprint:

```text
Design Only
```

---

House:

```text
Actual Running Structure
```

---

Similarly:

```text
Docker Image
       ↓
Docker Container
```

---

# What is a Docker Image?

A Docker Image is:

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

Configuration
```

---

Examples:

```text
nginx

ubuntu

mysql

node

python
```

---

Image is:

```text
Static

Immutable
```

---

Cannot execute by itself.

---

# Docker Image Structure

Example:

```text
Ubuntu Base OS
        ↓
Java Runtime
        ↓
Application Libraries
        ↓
Application Code
```

---

Combined Together:

```text
Docker Image
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

Size:

```text
Several GBs
```

---

Docker Image:

```text
Minimal OS

Runtime

Libraries

Application
```

---

Size:

```text
10 MB – 500 MB
```

---

# What is a Container?

A Container is:

```text
Running Instance Of An Image
```

---

Similar To:

```text
AMI
 ↓
EC2 Instance
```

---

Docker Equivalent:

```text
Image
 ↓
Container
```

---

# Container Characteristics

Containers are:

```text
Lightweight

Portable

Fast

Isolated
```

---

Container contains:

```text
Running Processes

Memory

Network

Filesystem
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

Example

```bash
docker run nginx
```

---

Result:

```text
nginx Container Running
```

---

# Can Multiple Containers Use Same Image?

Yes.

---

Example

```text
nginx Image
      ↓
Container 1

Container 2

Container 3
```

---

Single image can create:

```text
Unlimited Containers
```

---

# Docker Image Lifecycle

```text
Build Image
      ↓
Store Image
      ↓
Push Image
      ↓
Pull Image
      ↓
Run Container
```

---

# Docker Container Lifecycle

```text
Create
  ↓
Start
  ↓
Running
  ↓
Stop
  ↓
Remove
```

---

# Docker Images Command

List Images

```bash
docker images
```

---

Example Output

```text
REPOSITORY

TAG

IMAGE ID

SIZE
```

---

# Pull Image

Download image from registry.

```bash
docker pull nginx
```

---

Result:

```text
Image Stored Locally
```

---

Verify:

```bash
docker images
```

---

# Remove Image

```bash
docker rmi IMAGE_ID
```

---

Purpose:

```text
Delete Image
```

---

# What Happens During docker pull?

Example

```bash
docker pull nginx
```

---

Flow:

```text
Docker Hub
      ↓
Download Layers
      ↓
Store Image Locally
```

---

# Docker Create

Creates container.

---

Does NOT start container.

```bash
docker create nginx
```

---

Container State:

```text
Created
```

---

# Docker Start

Starts existing container.

```bash
docker start CONTAINER_ID
```

---

State:

```text
Running
```

---

# Docker Run

Most common command.

```bash
docker run nginx
```

---

Equivalent To:

```text
docker pull
      ↓
docker create
      ↓
docker start
```

---

# Container States

```text
Created

Running

Paused

Stopped

Exited

Deleted
```

---

# Running Containers

```bash
docker ps
```

---

Shows:

```text
Only Running Containers
```

---

# All Containers

```bash
docker ps -a
```

---

Shows:

```text
Running

Stopped

Exited
```

containers.

---

# Stop Container

```bash
docker stop CONTAINER_ID
```

---

Graceful shutdown.

---

Container State:

```text
Stopped
```

---

# Kill Container

```bash
docker kill CONTAINER_ID
```

---

Immediate termination.

---

Used in emergencies.

---

# Restart Container

```bash
docker restart CONTAINER_ID
```

---

Equivalent:

```text
Stop
 ↓
Start
```

---

# Remove Container

```bash
docker rm CONTAINER_ID
```

---

Deletes:

```text
Stopped Container
```

---

# Force Remove

```bash
docker rm -f CONTAINER_ID
```

---

Stops and removes.

---

# Image Layers

One of Docker's most important concepts.

---

Docker Images are built using:

```text
Layers
```

---

Example

```text
Ubuntu
   ↓
Java
   ↓
Application
```

---

Each step:

```text
Creates Layer
```

---

# Layer Benefits

```text
Reuse

Caching

Smaller Downloads

Faster Builds
```

---

# Layer Example

Container 1:

```text
Ubuntu

Java

Application A
```

---

Container 2:

```text
Ubuntu

Java

Application B
```

---

Docker reuses:

```text
Ubuntu Layer

Java Layer
```

---

Downloads only:

```text
Application Layer
```

---

# Why Layers Matter?

Benefits:

```text
Less Storage

Less Network Usage

Faster Build Time
```

---

# Read Only and Writable Layers

Image:

```text
Read Only
```

---

Container:

```text
Writable Layer
```

added on top.

---

Architecture

```text
Application Layer
      ↓
Java Layer
      ↓
Ubuntu Layer
      ↓
Writable Container Layer
```

---

# What Happens When Container Changes Data?

Changes are stored in:

```text
Container Writable Layer
```

---

Image remains:

```text
Unchanged
```

---

# Container Deletion Problem

Suppose:

```text
Database Container
```

stores data.

---

Container removed.

```bash
docker rm
```

---

Data lost.

---

Reason:

```text
Writable Layer Removed
```

---

Solution:

```text
Docker Volumes
```

---

(covered in future notes)

---

# Production Example

Roboshop Catalogue Service

Image:

```text
catalogue:v1
```

---

Deployment:

```bash
docker run catalogue:v1
```

---

Container Created.

---

Application Running.

---

New Version:

```text
catalogue:v2
```

---

Create:

```text
New Container
```

---

Old Container:

```text
Removed
```

---

# Common Interview Questions

## What is a Docker Image?

### Short Answer

A Docker Image is a read-only template used to create containers.

### Detailed Explanation

It contains application code, dependencies, runtime, and configurations required to run an application.

### Production Example

nginx image used to create nginx containers.

### Follow-Up Questions

- Can images be modified?
- Where are images stored?

---

## What is a Docker Container?

### Short Answer

A Docker Container is a running instance of an image.

### Detailed Explanation

Containers are isolated environments that execute applications using image contents.

### Production Example

Running catalogue service as a Docker container.

### Follow-Up Questions

- Can multiple containers use the same image?
- What happens when a container stops?

---

## Difference Between Image and Container?

### Short Answer

Image is a template, container is a running instance.

### Detailed Explanation

Images are static and immutable, while containers are dynamic and execute processes.

### Production Example

nginx image creating multiple nginx containers.

### Follow-Up Questions

- Can containers exist without images?
- Can images run directly?

---

## What Happens During docker run?

### Short Answer

Docker pulls the image (if needed), creates a container, and starts it.

### Detailed Explanation

docker run combines image retrieval, container creation, and startup into one command.

### Production Example

Launching nginx web server container.

### Follow-Up Questions

- Difference between run and start?
- Difference between create and run?

---

## What are Docker Image Layers?

### Short Answer

Layers are reusable filesystem components that make images efficient and lightweight.

### Detailed Explanation

Each Dockerfile instruction creates a layer, allowing Docker to reuse cached layers.

### Production Example

Reusing Ubuntu and Java layers across multiple application images.

### Follow-Up Questions

- Why are layers important?
- How does Docker caching work?

---

# Key Takeaways

```text
Docker Images are read-only templates.

Containers are running instances of images.

Images contain application code, runtime, libraries, and dependencies.

docker pull downloads images.

docker run creates and starts containers.

Multiple containers can use the same image.

Images are built using reusable layers.

Containers add a writable layer on top of images.

Deleting a container removes its writable layer.

Docker volumes solve container data persistence issues.

Understanding Images and Containers is fundamental to Docker and Kubernetes.
```