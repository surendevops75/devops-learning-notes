# Docker Port Mapping and Container Access

## Introduction

Containers run in an isolated environment.

When an application runs inside a container:

```text
External Users

Browsers

Other Systems
```

cannot access it directly.

---

To make applications accessible:

```text
Port Mapping
```

is used.

---

Port Mapping connects:

```text
Host Machine Port
          ↓
Container Port
```

---

# Running a Container

Basic Command

```bash
docker run nginx
```

---

Flow

```text
Pull Image
     ↓
Create Container
     ↓
Start Container
```

---

Container starts successfully.

---

Problem:

```text
Cannot Access Nginx
```

from browser.

---

Reason:

```text
Container Port Not Exposed To Host
```

---

# Detached Mode

Normally:

```bash
docker run nginx
```

---

Container occupies:

```text
Current Terminal
```

---

To run container in background:

```bash
docker run -d nginx
```

---

Option:

```text
-d
```

means:

```text
Detached Mode
```

---

Container runs:

```text
Background
```

---

Verify:

```bash
docker ps
```

---

# What is Port Mapping?

Port Mapping connects:

```text
Host Port
       ↓
Container Port
```

---

Syntax

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

---

Example

```bash
docker run -d -p 80:80 nginx
```

---

Meaning:

```text
Host Port      = 80

Container Port = 80
```

---

Architecture

```text
Browser
    ↓
Server:80
    ↓
Docker Host
    ↓
Container:80
    ↓
Nginx
```

---

# Real Example

Launch Nginx

```bash
docker run -d -p 80:80 nginx
```

---

Check:

```bash
docker ps
```

---

Output

```text
0.0.0.0:80->80/tcp
```

---

Meaning:

```text
Host Port 80
Forwarded To
Container Port 80
```

---

# Accessing Application

Suppose Server IP:

```text
98.92.17.244
```

---

Access:

```text
http://98.92.17.244
```

---

Request Flow

```text
Browser
    ↓
98.92.17.244:80
    ↓
Docker Host
    ↓
Nginx Container
```

---

# Different Host Port

Example

```bash
docker run -d -p 8080:80 nginx
```

---

Meaning:

```text
Host Port      8080

Container Port 80
```

---

Access:

```text
http://SERVER_IP:8080
```

---

# Multiple Containers

Container 1

```bash
docker run -d -p 8080:80 nginx
```

---

Container 2

```bash
docker run -d -p 8081:80 nginx
```

---

Architecture

```text
Host:8080
      ↓
Container1:80

Host:8081
      ↓
Container2:80
```

---

# Why Container Port Cannot Change?

Application inside container listens on:

```text
Configured Port
```

---

Example:

```text
Nginx
```

listens on:

```text
80
```

---

Therefore:

```text
Container Port
```

usually remains fixed.

---

Host Port can change.

---

# Docker Container IDs

Every container receives:

```text
Unique Container ID
```

---

Example

```text
4fa23ab45dc1
```

---

View:

```bash
docker ps
```

---

# Access Container Shell

Command:

```bash
docker exec -it CONTAINER_ID bash
```

---

Example

```bash
docker exec -it 4fa23ab45dc1 bash
```

---

Meaning

```text
Execute Command
Interactive Mode
Terminal Access
```

---

Options

```text
-i = Interactive

-t = Terminal
```

---

# Why Use docker exec?

Used for:

```text
Troubleshooting

Checking Logs

Checking Files

Testing Commands
```

---

Production Example

```bash
docker exec -it catalogue bash
```

---

Check:

```text
Configuration Files

Environment Variables

Application Files
```

---

# Common Commands Inside Container

Check OS

```bash
cat /etc/os-release
```

---

Check Files

```bash
ls -l
```

---

Check Running Processes

```bash
ps -ef
```

---

Check Open Ports

```bash
netstat -lntp
```

---

# Docker Inspect

Command

```bash
docker inspect CONTAINER_ID
```

---

Example

```bash
docker inspect 4fa23ab45dc1
```

---

Provides:

```text
Container IP

Networks

Volumes

Mounts

Environment Variables

Image Details
```

---

# Container IP Address

Get Container IP

```bash
docker inspect CONTAINER_ID
```

---

Example Output

```json
"IPAddress": "172.17.0.2"
```

---

Container Network

```text
Docker Internal Network
```

---

# Production Troubleshooting

Application Not Accessible

---

Check:

```bash
docker ps
```

---

Verify:

```text
Container Running
```

---

Check:

```bash
docker inspect CONTAINER_ID
```

---

Verify:

```text
Port Mapping
```

---

Check:

```bash
docker exec -it CONTAINER_ID bash
```

---

Verify:

```text
Application Running
```

---

# Docker Home Directory

Docker stores data in:

```text
/var/lib/docker
```

---

Contains:

```text
Images

Containers

Networks

Volumes

Metadata
```

---

# Why Increase /var?

In Production:

```text
Many Images

Many Containers

Large Volumes
```

---

Disk Space may exhaust.

---

Common Expansion Commands

```bash
growpart /dev/nvme0n1 4

lvextend -L +30G /dev/mapper/RootVG-varVol

xfs_growfs /var
```

---

Purpose:

```text
Increase Docker Storage Space
```

---

# Remove All Containers

Command

```bash
docker rm -f `docker ps -a -q`
```

---

Explanation

```text
docker ps -a -q
```

returns:

```text
All Container IDs
```

---

Then:

```bash
docker rm -f
```

removes them.

---

Use Carefully.

---

# Architecture Summary

```text
Browser
    ↓
Host Port
    ↓
Docker Host
    ↓
Container Port
    ↓
Application
```

---

# Common Interview Questions

## What is Port Mapping?

### Short Answer

Port mapping connects a host port to a container port.

### Detailed Explanation

Containers are isolated. Port mapping allows external systems to access applications running inside containers.

### Production Example

docker run -d -p 80:80 nginx

### Follow-Up Questions

- Can host and container ports be different?
- Why is port mapping required?

---

## What Does -d Mean in docker run?

### Short Answer

-d runs the container in detached mode.

### Detailed Explanation

The container runs in the background and frees the terminal.

### Production Example

Running nginx continuously without occupying the terminal.

### Follow-Up Questions

- What happens without -d?
- How do you check detached containers?

---

## What is docker exec Used For?

### Short Answer

It executes commands inside a running container.

### Detailed Explanation

docker exec provides shell access for troubleshooting and validation.

### Production Example

Accessing a catalogue container to inspect configuration files.

### Follow-Up Questions

- Difference between docker run and docker exec?
- Can docker exec work on stopped containers?

---

## What is docker inspect?

### Short Answer

docker inspect displays detailed metadata about containers and images.

### Detailed Explanation

It provides information such as networking, IP addresses, volumes, and environment variables.

### Production Example

Checking container port mappings during troubleshooting.

### Follow-Up Questions

- How do you get a container IP?
- Can inspect be used on images?

---

# Key Takeaways

```text
Containers are isolated from the host.

Port mapping exposes container applications.

docker run -p maps host ports to container ports.

-d runs containers in detached mode.

docker exec provides shell access to running containers.

docker inspect provides detailed container metadata.

Docker stores images and containers under /var/lib/docker.

Production Docker servers often require additional storage for /var.

Port mapping is one of the most frequently used Docker runtime features.

Understanding port mapping is essential before learning Docker networking and Kubernetes Services.
```