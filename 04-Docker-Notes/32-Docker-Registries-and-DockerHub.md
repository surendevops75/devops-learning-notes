# Docker Registries and Docker Hub

## Introduction

Docker images are typically built on one machine and run on another.

Example:

```text
Developer Laptop
        ↓

Build Docker Image
        ↓

Push To Registry
        ↓

Production Server
        ↓

Pull Image
        ↓

Run Container
```

---

Question:

```text
Where Are Docker Images Stored?
```

---

Answer:

```text
Docker Registry
```

---

# What is a Docker Registry?

A Docker Registry is a centralized repository used to:

```text
Store Images

Manage Image Versions

Distribute Images

Share Images
```

---

Think of it as:

```text
GitHub For Docker Images
```

---

# Why Do We Need a Registry?

Suppose:

```text
Developer Builds Image
```

```bash
docker build -t catalogue:v1 .
```

---

Image exists only on:

```text
Developer Machine
```

---

Production server cannot access it.

---

Need:

```text
Central Storage
```

---

Solution:

```text
Docker Registry
```

---

# Registry Workflow

```text
Developer
      ↓

Build Image
      ↓

Push Image
      ↓

Docker Registry
      ↓

Production Server
      ↓

Pull Image
      ↓

Run Container
```

---

# Popular Docker Registries

## Docker Hub

```text
Most Popular Registry
```

---

## Amazon ECR

```text
AWS Registry Service
```

---

## Google Artifact Registry

```text
Google Cloud Registry
```

---

## Azure Container Registry

```text
Microsoft Azure Registry
```

---

## Harbor

```text
Self Hosted Registry
```

---

# What is Docker Hub?

Docker Hub is:

```text
Official Docker Registry
```

---

Website:

```text
hub.docker.com
```

---

Contains:

```text
Public Images

Private Images

Official Images

Community Images
```

---

# Official Images

Examples

```text
nginx

redis

mongo

mysql

ubuntu

alpine
```

---

Pull Example

```bash
docker pull nginx
```

---

Docker downloads image from:

```text
Docker Hub
```

automatically.

---

# Public Repository

Anyone can:

```text
Pull Images
```

---

Example

```bash
docker pull nginx
```

---

No authentication required.

---

# Private Repository

Access restricted.

---

Requires:

```text
Authentication

Authorization
```

---

Used for:

```text
Internal Applications

Production Images

Enterprise Environments
```

---

# Docker Image Naming Convention

Format

```text
registry/username/image:tag
```

---

Example

```text
docker.io/library/nginx:latest
```

---

Components

```text
docker.io
      ↓
Registry

library
      ↓
Repository

nginx
      ↓
Image Name

latest
      ↓
Tag
```

---

# Image Tags

Tags identify image versions.

---

Example

```text
catalogue:v1

catalogue:v2

catalogue:latest
```

---

Benefits

```text
Version Control

Rollback Support

Release Management
```

---

# Why Tags Matter?

Bad Practice

```text
latest
```

---

Problem

```text
Version Changes Unexpectedly
```

---

Recommended

```text
v1.0.0

v1.1.0

v2.0.0
```

---

# Creating Docker Hub Account

Create Account

```text
hub.docker.com
```

---

Example Username

```text
surendevops75
```

---

# Docker Login

Authenticate with Docker Hub.

---

Command

```bash
docker login
```

---

Enter

```text
Username

Password
```

---

Successful Login

```text
Login Succeeded
```

---

# Tagging Images

Suppose Image

```text
catalogue:v1
```

---

Tag For Docker Hub

```bash
docker tag \
catalogue:v1 \
username/catalogue:v1
```

---

Example

```bash
docker tag \
catalogue:v1 \
surendevops75/catalogue:v1
```

---

Verify

```bash
docker images
```

---

Output

```text
surendevops75/catalogue:v1
```

---

# Pushing Images

Command

```bash
docker push username/catalogue:v1
```

---

Example

```bash
docker push surendevops75/catalogue:v1
```

---

Flow

```text
Local Image
      ↓

Docker Hub
```

---

# Pulling Images

On another server:

```bash
docker pull surendevops75/catalogue:v1
```

---

Docker downloads image from:

```text
Docker Hub
```

---

# Running Pulled Images

```bash
docker run -d \
-p 8080:8080 \
surendevops75/catalogue:v1
```

---

Container starts successfully.

---

# Docker Pull Process

Command

```bash
docker pull nginx
```

---

Docker Checks

```text
Local Cache
```

---

If Found

```text
Use Local Image
```

---

If Not Found

```text
Pull From Registry
```

---

# Image Layers

Docker images are:

```text
Layer Based
```

---

Example

```text
Base OS Layer

Runtime Layer

Application Layer
```

---

Only changed layers are downloaded.

---

Benefits

```text
Faster Downloads

Reduced Storage
```

---

# Repository

Repository stores:

```text
Multiple Versions
```

---

Example

```text
catalogue:v1

catalogue:v2

catalogue:v3
```

---

All belong to:

```text
catalogue Repository
```

---

# Private Repository Workflow

```text
Developer
      ↓

Build Image
      ↓

Push To Private Repo
      ↓

Production Pulls Image
```

---

Access controlled using:

```text
Credentials

Permissions
```

---

# Registry Architecture

```text
Developer
      ↓

docker build
      ↓

Local Image
      ↓

docker push
      ↓

Registry
      ↓

docker pull
      ↓

Server
      ↓

docker run
```

---

# Production Example

Roboshop

Build

```bash
docker build -t catalogue:v1 .
```

---

Tag

```bash
docker tag \
catalogue:v1 \
surendevops75/catalogue:v1
```

---

Push

```bash
docker push \
surendevops75/catalogue:v1
```

---

Production

```bash
docker pull \
surendevops75/catalogue:v1
```

---

Run

```bash
docker run \
-d \
catalogue:v1
```

---

# Common Registry Commands

Login

```bash
docker login
```

---

Logout

```bash
docker logout
```

---

Pull

```bash
docker pull nginx
```

---

Push

```bash
docker push username/image:v1
```

---

Tag

```bash
docker tag image:v1 username/image:v1
```

---

List Images

```bash
docker images
```

---

# Common Interview Questions

## What is a Docker Registry?

### Short Answer

A centralized repository used to store and distribute Docker images.

### Detailed Explanation

Registries enable image sharing between developers, CI/CD pipelines, and production environments.

### Production Example

Docker Hub storing application images.

### Follow-Up Questions

- What is Docker Hub?
- What is ECR?

---

## What is Docker Hub?

### Short Answer

Docker Hub is the official public Docker image registry.

### Detailed Explanation

Docker Hub hosts official, community, and private Docker images.

### Production Example

Pulling nginx and redis images.

### Follow-Up Questions

- What are official images?
- What are private repositories?

---

## Difference Between docker pull and docker push?

### Short Answer

docker pull downloads images, docker push uploads images.

### Detailed Explanation

Pull retrieves images from registries, while push publishes local images to registries.

### Production Example

Publishing catalogue:v1 to Docker Hub.

### Follow-Up Questions

- Why is tagging required?
- What registry authentication is needed?

---

## Why Are Image Tags Important?

### Short Answer

Tags identify image versions.

### Detailed Explanation

Tags enable version control, release management, and rollback capabilities.

### Production Example

catalogue:v1, catalogue:v2.

### Follow-Up Questions

- Why avoid latest in production?
- How do tags help rollbacks?

---

# Key Takeaways

```text
Docker Registry stores and distributes Docker images.

Docker Hub is the most widely used registry.

Registries can be public or private.

docker login authenticates users.

docker tag prepares images for publishing.

docker push uploads images to registries.

docker pull downloads images from registries.

Tags provide image versioning.

Docker images are layer-based.

Registries are a critical part of CI/CD and container deployments.
```