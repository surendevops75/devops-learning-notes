# Docker Image Building, Tagging and Pushing

## Introduction

Creating custom Docker images is one of the most important Docker skills.

In real projects:

```text
Developers Write Code
        ↓
Docker Image Built
        ↓
Image Stored In Registry
        ↓
Image Pulled By Servers
        ↓
Container Started
```

---

Example:

```text
Catalogue Service

Cart Service

User Service

Frontend Service
```

Each service is packaged as:

```text
Docker Image
```

---

# Image Build Process

Architecture

```text
Dockerfile
      ↓
docker build
      ↓
Docker Image
      ↓
docker tag
      ↓
docker push
      ↓
Docker Registry
      ↓
docker pull
      ↓
Container
```

---

# What is docker build?

docker build creates:

```text
Docker Image
```

from:

```text
Dockerfile
```

---

Syntax

```bash
docker build .
```

---

Meaning

```text
Read Dockerfile

Build Image
```

---

# Why Dot (.)?

Example

```bash
docker build .
```

---

Dot means:

```text
Current Directory
```

---

Docker searches for:

```text
Dockerfile
```

inside current directory.

---

Example

```text
project/
│
├── Dockerfile
├── app.py
└── requirements.txt
```

---

Build command

```bash
docker build .
```

---

Docker automatically finds:

```text
Dockerfile
```

---

# Naming Images

Usually we build images using:

```bash
docker build -t IMAGE:VERSION .
```

---

Example

```bash
docker build -t catalogue:v1 .
```

---

Result

```text
Image Name = catalogue

Version = v1
```

---

Verify

```bash
docker images
```

---

Output

```text
catalogue     v1
```

---

# Image Tags

Tag means:

```text
Version Identifier
```

---

Examples

```text
v1

v2

1.0

1.1

latest
```

---

# Why Tags are Important?

Without tags:

```text
Version Tracking Impossible
```

---

Example

Bad

```text
catalogue
```

---

Good

```text
catalogue:v1

catalogue:v2

catalogue:v3
```

---

# Build Example

Dockerfile

```dockerfile
FROM almalinux:9

RUN dnf install nginx -y

CMD ["nginx","-g","daemon off;"]
```

---

Build

```bash
docker build -t nginx-custom:v1 .
```

---

Result

```text
Custom Image Created
```

---

# Build Cache

Docker uses:

```text
Layer Caching
```

---

Example

```dockerfile
FROM almalinux:9

RUN dnf install nginx -y

RUN mkdir /app
```

---

First Build

```text
All Layers Created
```

---

Second Build

```text
Cached Layers Used
```

---

Result

```text
Much Faster Build
```

---

# No Cache Build

Sometimes cache causes problems.

---

Force fresh build:

```bash
docker build --no-cache -t nginx:v1 .
```

---

Purpose

```text
Ignore Existing Layers

Rebuild Everything
```

---

# Detailed Build Logs

Command

```bash
docker build --progress=plain -t nginx:v1 .
```

---

Purpose

```text
Verbose Output
```

---

Useful During:

```text
Troubleshooting

Debugging Builds
```

---

# Docker Registries

After building image:

```text
Need To Store It Somewhere
```

---

Storage Location:

```text
Docker Registry
```

---

Examples

```text
Docker Hub

AWS ECR

Azure ACR

Google Artifact Registry

JFrog Artifactory
```

---

# Docker Hub

Default Public Registry.

---

Examples

```text
docker.io/nginx

docker.io/mysql

docker.io/python
```

---

# Docker Image Naming Convention

General Format

```text
registry/username/image:version
```

---

Example

```text
docker.io/joindevops/from:v1
```

---

Breakdown

```text
docker.io = Registry

joindevops = Username

from = Image Name

v1 = Tag
```

---

# Logging Into Docker Hub

Before pushing images:

```bash
docker login
```

---

Prompts:

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

# Why docker login?

Needed for:

```text
Push Operations

Private Repositories
```

---

Not required for:

```text
Public Pull Operations
```

---

# Tagging Images

Suppose Image:

```text
catalogue:v1
```

---

Docker Hub expects:

```text
username/catalogue:v1
```

---

Command

```bash
docker tag catalogue:v1 username/catalogue:v1
```

---

Example

```bash
docker tag catalogue:v1 surendra/catalogue:v1
```

---

Result

```text
New Tag Created
```

---

Verify

```bash
docker images
```

---

Output

```text
catalogue:v1

surendra/catalogue:v1
```

---

Both point to:

```text
Same Image ID
```

---

# What is docker tag?

Docker tag:

```text
Does Not Create New Image
```

---

It only:

```text
Creates Additional Reference
```

to existing image.

---

# Push Image

After tagging:

```bash
docker push username/catalogue:v1
```

---

Example

```bash
docker push surendra/catalogue:v1
```

---

Flow

```text
Local Image
      ↓
Docker Hub
      ↓
Stored In Registry
```

---

# Verify Push

Docker Hub

↓

Repositories

↓

Image Visible

---

# Pull Image

Another server:

```bash
docker pull surendra/catalogue:v1
```

---

Flow

```text
Docker Hub
      ↓
Server
      ↓
Local Image
```

---

# Image Pull Logic

Docker follows:

```text
Local Check
     ↓
Registry Check
```

---

Example

```bash
docker run nginx
```

---

Step 1

Check local image.

---

If Found

```text
Use Local Image
```

---

If Not Found

```text
Pull From Docker Hub
```

---

Then:

```text
Create Container

Start Container
```

---

# Real Production Example

Developer Updates:

```text
Catalogue Service
```

---

Pipeline

```text
Code Commit
      ↓
Docker Build
      ↓
catalogue:v2
      ↓
Docker Push
      ↓
AWS ECR
      ↓
Kubernetes Deployment
```

---

# Difference Between Build and Push

Build

```text
Creates Image Locally
```

---

Push

```text
Uploads Image To Registry
```

---

# Difference Between Tag and Push

Tag

```text
Rename Image Reference
```

---

Push

```text
Upload Image
```

---

# Cleanup Containers

Remove All Containers

```bash
docker rm -f `docker ps -a -q`
```

---

Explanation

```text
docker ps -a -q

Returns All Container IDs
```

---

Then

```bash
docker rm -f
```

removes them.

---

Use carefully.

---

# Production Architecture

```text
Developer
     ↓
Dockerfile
     ↓
docker build
     ↓
Docker Image
     ↓
docker tag
     ↓
docker push
     ↓
Docker Registry
     ↓
docker pull
     ↓
Container Running
```

---

# Common Interview Questions

## What Does docker build Do?

### Short Answer

Creates a Docker image from a Dockerfile.

### Detailed Explanation

Docker reads Dockerfile instructions and builds image layers sequentially.

### Production Example

Building catalogue:v1 from application source code.

### Follow-Up Questions

- What is build context?
- What is Docker cache?

---

## What Does docker tag Do?

### Short Answer

Creates a new reference for an existing image.

### Detailed Explanation

Tagging does not duplicate image data. It only adds another name pointing to the same image.

### Production Example

Preparing an image for Docker Hub upload.

### Follow-Up Questions

- Does docker tag create a new image?
- Why tag before push?

---

## What Does docker push Do?

### Short Answer

Uploads an image to a Docker registry.

### Detailed Explanation

Docker transfers image layers from the local system to a registry such as Docker Hub or ECR.

### Production Example

Uploading catalogue:v1 to Docker Hub.

### Follow-Up Questions

- Is docker login required?
- What happens if image already exists?

---

## What Happens During docker run nginx?

### Short Answer

Docker checks locally, pulls if necessary, creates a container, and starts it.

### Detailed Explanation

Docker first searches for the image locally. If absent, it downloads the image from Docker Hub.

### Production Example

Launching an nginx container on a new server.

### Follow-Up Questions

- When does Docker pull automatically?
- How can we avoid pulling?

---

# Key Takeaways

```text
docker build creates images from Dockerfiles.

The dot (.) represents the current build context.

Tags represent image versions.

docker build -t assigns image name and version.

Docker uses layer caching to speed up builds.

--no-cache forces a complete rebuild.

docker login authenticates to registries.

docker tag creates additional image references.

docker push uploads images to registries.

docker pull downloads images from registries.

Docker checks local images before pulling from a registry.

Image building and pushing are core components of modern CI/CD pipelines.
```