# Docker Architecture

## Introduction

Docker follows a client-server architecture.

When we execute a Docker command:

```bash
docker run nginx
```

Docker performs multiple operations behind the scenes.

Understanding Docker architecture helps in:

```text
Troubleshooting

Interview Preparation

Production Operations

Container Lifecycle Understanding
```

---

# What is Docker Architecture?

Docker Architecture consists of:

```text
Docker Client

Docker Host

Docker Daemon

Docker Objects

Docker Local Registry

Docker Central Registry
```

---

# High Level Architecture

```text
+------------------+
|  Docker Client   |
+--------+---------+
         |
         v

+------------------+
| Docker Daemon    |
|   (dockerd)      |
+--------+---------+
         |
    -------------
    |     |     |
    v     v     v

 Images Containers Networks
 Volumes

         |
         v

+------------------+
| Local Registry   |
+--------+---------+
         |
         v

+------------------+
| Docker Hub / ECR |
+------------------+
```

---

# Docker Client

Docker Client is the interface through which users interact with Docker.

Examples:

```bash
docker build

docker run

docker ps

docker pull

docker push
```

---

When a user executes:

```bash
docker run nginx
```

the command is sent to:

```text
Docker Daemon
```

for execution.

---

# Responsibilities of Docker Client

```text
Accept User Commands

Send Requests To Docker Daemon

Display Results To Users
```

---

# Docker Host

Docker Host is the machine where Docker is installed.

It can be:

```text
Physical Server

Virtual Machine

Cloud Instance

Developer Laptop
```

---

The Docker Host contains:

```text
Docker Daemon

Images

Containers

Volumes

Networks
```

---

Architecture

```text
Docker Host
│
├── Docker Daemon
├── Images
├── Containers
├── Networks
└── Volumes
```

---

# Docker Daemon

Docker Daemon is the core Docker service.

Service Name:

```text
dockerd
```

---

Docker Daemon continuously runs in the background.

---

Responsibilities:

```text
Build Images

Run Containers

Stop Containers

Manage Volumes

Manage Networks

Communicate With Registries
```

---

Check Docker Service

```bash
systemctl status docker
```

---

Start Docker Service

```bash
systemctl start docker
```

---

Enable Docker Service

```bash
systemctl enable docker
```

---

# Docker Objects

Docker works with several objects.

---

# Images

Images are:

```text
Read Only Templates
```

used to create containers.

---

Example

```bash
docker pull nginx
```

---

Image Contains:

```text
Base OS

Application Runtime

Application Code

Dependencies
```

---

# Containers

Containers are:

```text
Running Instances Of Images
```

---

Example

```bash
docker run nginx
```

---

Relationship

```text
Image
   ↓

Container
```

---

One image can create:

```text
Multiple Containers
```

---

# Volumes

Volumes provide:

```text
Persistent Storage
```

for containers.

---

Example

```bash
docker volume create mongo-data
```

---

Use Cases

```text
MongoDB

MySQL

RabbitMQ

Redis
```

---

# Networks

Networks enable:

```text
Container Communication
```

---

Example

```bash
docker network create roboshop
```

---

Types:

```text
Bridge

Host

None

Overlay
```

---

# Docker Local Registry

Docker stores downloaded images locally.

---

View Local Images

```bash
docker images
```

---

Example Output

```text
nginx

redis

mongo

catalogue
```

---

Local registry acts as:

```text
Image Cache
```

---

Benefits

```text
Faster Container Startup

Reduced Downloads

Reduced Internet Usage
```

---

# Docker Central Registry

Central registry stores images for sharing.

---

Examples

```text
Docker Hub

Amazon ECR

Azure ACR

Google Artifact Registry

Harbor
```

---

Most common:

```text
Docker Hub
```

---

# Docker Pull Process

Command

```bash
docker run nginx
```

---

Docker follows:

```text
Check Local Image
        ↓

Image Found?
   ↓          ↓

 Yes         No
  ↓           ↓

Run      Pull From Registry
              ↓

       Store Locally
              ↓

            Run
```

---

# Docker Push Process

Build Image

```bash
docker build -t catalogue:v1 .
```

---

Tag Image

```bash
docker tag \
catalogue:v1 \
username/catalogue:v1
```

---

Push Image

```bash
docker push username/catalogue:v1
```

---

Flow

```text
Local Image
      ↓

Docker Registry
```

---

# Complete Request Flow

Suppose user executes:

```bash
docker run nginx
```

---

Step 1

```text
Docker Client Receives Command
```

---

Step 2

```text
Docker Client Sends Request
To Docker Daemon
```

---

Step 3

Docker Daemon checks:

```text
Local Registry
```

for image.

---

Step 4

If image not available:

```text
Pull Image From Docker Hub
```

---

Step 5

Store image locally.

---

Step 6

Create container.

---

Step 7

Start container.

---

Final Result

```text
Running Container
```

---

# Docker Architecture in Production

Example

```text
Developer
      ↓

Docker Build
      ↓

Local Image
      ↓

Push To ECR
      ↓

EKS Pulls Image
      ↓

Pod Starts
```

---

# Architecture Components Summary

| Component | Purpose |
|------------|----------|
| Docker Client | Accepts Commands |
| Docker Daemon | Executes Docker Operations |
| Docker Host | Machine Running Docker |
| Images | Read Only Templates |
| Containers | Running Instances |
| Volumes | Persistent Storage |
| Networks | Container Communication |
| Local Registry | Local Image Cache |
| Central Registry | Shared Image Repository |

---

# Common Interview Questions

## Explain Docker Architecture.

### Short Answer

Docker Architecture consists of Docker Client, Docker Daemon, Docker Host, Docker Objects, Local Registry, and Central Registry.

### Detailed Explanation

The Docker Client sends requests to the Docker Daemon. The Docker Daemon manages images, containers, volumes, and networks while communicating with registries to pull or push images.

### Production Example

Developer builds an image, pushes it to ECR, and EKS pulls the image for deployment.

### Follow-Up Questions

- What is dockerd?
- What is the difference between Docker Client and Docker Daemon?

---

## What is Docker Daemon?

### Short Answer

Docker Daemon is the background service responsible for managing Docker operations.

### Detailed Explanation

It handles image builds, container lifecycle management, networking, volumes, and registry communication.

### Production Example

Creating containers from images when deployment requests arrive.

### Follow-Up Questions

- How do you check daemon status?
- What happens if dockerd stops?

---

## What Happens When You Run docker run nginx?

### Short Answer

Docker checks for the image locally, pulls it if necessary, creates a container, and starts it.

### Detailed Explanation

The Docker Client sends a request to the Docker Daemon, which manages the complete lifecycle.

### Production Example

Running Nginx on a development or production server.

### Follow-Up Questions

- Where is the image downloaded from?
- What if the image already exists locally?

---

# Key Takeaways

```text
Docker follows a client-server architecture.

Docker Client is the user interface.

Docker Daemon executes Docker operations.

Docker Host contains images, containers, networks, and volumes.

Images are stored locally in the Docker host.

Docker Hub and ECR act as central registries.

docker run triggers a sequence of client, daemon, image, and container operations.

Understanding Docker Architecture is essential for troubleshooting and interviews.

Docker Architecture forms the foundation for Kubernetes container orchestration.
```