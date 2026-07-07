# Docker Installation and Docker CLI

## Introduction

Before creating and running containers, Docker must be installed on the server.

---

Once installed:

```text
Docker Daemon Starts

Docker CLI Becomes Available

Containers Can Be Managed
```

---

Docker installation typically involves:

```text
Installing Docker Engine

Configuring User Access

Managing Docker Service

Running Docker Commands
```

---

# Docker Installation

## Step 1

Install Required Plugins

```bash
sudo dnf -y install dnf-plugins-core
```

---

Purpose:

```text
Enable Additional DNF Features

Manage External Repositories
```

---

# Step 2

Add Docker Repository

```bash
sudo dnf config-manager \
--add-repo \
https://download.docker.com/linux/rhel/docker-ce.repo
```

---

Purpose:

```text
Add Official Docker Repository
```

---

Without Repository:

```text
Latest Docker Packages
May Not Be Available
```

---

# Step 3

Install Docker Packages

```bash
sudo dnf install \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin -y
```

---

# Important Packages

## docker-ce

```text
Docker Community Edition
```

Main Docker Engine.

---

## docker-ce-cli

```text
Docker Command Line Interface
```

Provides:

```bash
docker
```

command.

---

## containerd.io

Container Runtime.

---

Responsible for:

```text
Container Execution

Container Lifecycle

Container Management
```

---

## docker-buildx-plugin

Provides:

```text
Advanced Image Building
```

Capabilities.

---

Used for:

```text
Multi Architecture Builds

Advanced Build Features
```

---

## docker-compose-plugin

Provides:

```text
docker compose
```

command.

---

Used for:

```text
Multi Container Applications
```

---

# Step 4

Start Docker Service

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

Verify:

```bash
sudo systemctl status docker
```

---

Expected:

```text
Active (running)
```

---

# Verify Installation

Check Version

```bash
docker version
```

---

Example Output

```text
Client

Server
```

---

Check Docker Information

```bash
docker info
```

---

Displays:

```text
Containers

Images

Storage Driver

Networks

Docker Root Directory
```

---

# Docker Service Architecture

```text
Docker CLI
       ↓
Docker Daemon
       ↓
containerd
       ↓
Containers
```

---

# Docker User Access

By default:

```text
Only Root User
```

can run Docker commands.

---

Example

```bash
docker ps
```

---

May Return

```text
Permission Denied
```

---

# Docker Group

During installation Docker creates:

```text
docker
```

group.

---

Check:

```bash
cat /etc/group | grep docker
```

---

Example

```text
docker:x:995:
```

---

# Add User to Docker Group

```bash
sudo usermod -aG docker ec2-user
```

---

Meaning:

```text
Append User

To Docker Group
```

---

# Why Use Docker Group?

Without Docker Group:

```text
sudo docker ps
```

required every time.

---

With Docker Group:

```bash
docker ps
```

works directly.

---

# Important

After Adding User

```bash
exit
```

---

Login Again.

---

Reason:

```text
New Group Membership
Must Be Reloaded
```

---

# Verify Group Membership

```bash
groups
```

---

Expected:

```text
docker
```

listed.

---

# Docker CLI

Docker CLI is:

```text
Command Line Interface
```

used to interact with Docker.

---

# Docker Help

```bash
docker --help
```

---

Displays:

```text
Available Commands
```

---

# Docker Version

```bash
docker version
```

---

Displays:

```text
Client Version

Server Version
```

---

# Docker Info

```bash
docker info
```

---

Useful During Troubleshooting.

---

Displays:

```text
Images

Containers

Storage Driver

Network Driver

Docker Root Directory
```

---

# List Running Containers

```bash
docker ps
```

---

Shows:

```text
Running Containers Only
```

---

Example

```text
CONTAINER ID

IMAGE

STATUS

PORTS
```

---

# List All Containers

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

# List Images

```bash
docker images
```

---

Displays:

```text
Repository

Tag

Image ID

Size
```

---

Example

```text
nginx

ubuntu

mysql
```

---

# Pull Image

```bash
docker pull nginx
```

---

Downloads:

```text
nginx Image
```

from Docker Hub.

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

# Run Container

```bash
docker run nginx
```

---

Equivalent To:

```text
Pull Image
      ↓
Create Container
      ↓
Start Container
```

---

# Stop Container

```bash
docker stop CONTAINER_ID
```

---

Gracefully stops container.

---

# Start Container

```bash
docker start CONTAINER_ID
```

---

Starts existing container.

---

# Restart Container

```bash
docker restart CONTAINER_ID
```

---

Equivalent:

```text
Stop

Then

Start
```

---

# Remove Container

```bash
docker rm CONTAINER_ID
```

---

Deletes container.

---

Container must be:

```text
Stopped First
```

---

# Container Logs

```bash
docker logs CONTAINER_ID
```

---

Used for:

```text
Application Troubleshooting
```

---

# Execute Command Inside Container

```bash
docker exec -it CONTAINER_ID bash
```

---

Purpose:

```text
Access Container Shell
```

---

Common Troubleshooting Command.

---

# Inspect Container

```bash
docker inspect CONTAINER_ID
```

---

Provides:

```text
IP Address

Volumes

Networks

Environment Variables
```

---

# Resource Usage

```bash
docker stats
```

---

Displays:

```text
CPU Usage

Memory Usage

Network Usage
```

---

Useful During:

```text
Performance Troubleshooting
```

---

# Production Example

DevOps Engineer receives alert:

```text
Catalogue Service Not Working
```

---

Investigation:

```bash
docker ps

docker logs catalogue

docker exec -it catalogue bash

docker inspect catalogue
```

---

Identify:

```text
Application Error

Configuration Error

Dependency Issue
```

---

# Most Common Docker Commands

```bash
docker version

docker info

docker ps

docker ps -a

docker images

docker pull

docker run

docker stop

docker start

docker restart

docker rm

docker logs

docker exec

docker inspect

docker stats
```

---

# Common Interview Questions

## Why is Docker Group Required?

### Short Answer

It allows non-root users to execute Docker commands.

### Detailed Explanation

Docker daemon runs with elevated privileges. Users in the docker group can communicate with the daemon without using sudo.

### Production Example

Allowing DevOps engineers to manage containers without root access.

### Follow-Up Questions

- What happens if user is not in docker group?
- Why must we logout and login again?

---

## Difference Between docker ps and docker ps -a?

### Short Answer

docker ps shows running containers, while docker ps -a shows all containers.

### Detailed Explanation

docker ps filters only active containers.

### Production Example

Checking stopped application containers during troubleshooting.

### Follow-Up Questions

- How do you identify exited containers?
- Which command is used most frequently?

---

## What Does docker exec Do?

### Short Answer

Executes commands inside a running container.

### Detailed Explanation

Used for troubleshooting, validation, and configuration checks.

### Production Example

Accessing an nginx container to inspect files.

### Follow-Up Questions

- Difference between exec and run?
- Can exec work on stopped containers?

---

# Key Takeaways

```text
Docker installation requires Docker Engine, CLI, and containerd.

Docker daemon must be running.

Docker group allows non-root users to execute Docker commands.

docker ps displays running containers.

docker images displays available images.

docker logs helps troubleshoot containers.

docker exec provides shell access inside containers.

docker inspect provides detailed metadata.

docker stats displays resource usage.

Docker CLI is the primary interface used to manage containers.
```