# Docker Health Checks and Resource Limits

## Introduction

Building a Docker image is only half the work.

A production-ready container should also be:

```text
Healthy

Reliable

Resource Controlled

Self Recoverable
```

---

Two important concepts help achieve this:

```text
Health Checks

Resource Limits
```

---

Without them:

```text
Application May Be Running
But Not Working

One Container May Consume
All Server Resources
```

---

# What is a Health Check?

A health check is a mechanism used to verify whether:

```text
Application Is Actually Working
```

not just whether:

```text
Container Is Running
```

---

Example

```text
Container Status

Running
```

---

But application may be:

```text
Hung

Not Responding

Database Connection Failed

Out Of Memory
```

---

Container appears healthy.

Application is not.

---

# Why Health Checks Are Needed?

Suppose

```text
Catalogue Service
```

starts successfully.

---

After some time:

```text
MongoDB Connection Lost
```

---

Application stops serving requests.

---

Container still shows:

```text
Running
```

---

Without health checks:

```text
Docker Cannot Detect Problem
```

---

# Container Status vs Health Status

Container Status

```text
Running
```

means:

```text
Process Exists
```

---

Health Status

```text
Healthy
```

means:

```text
Application Responding Properly
```

---

# Docker HEALTHCHECK Instruction

Docker provides:

```dockerfile
HEALTHCHECK
```

instruction.

---

Example

```dockerfile
HEALTHCHECK \
CMD curl -f http://localhost:8080/health || exit 1
```

---

Meaning

```text
Call Health Endpoint

Success = Healthy

Failure = Unhealthy
```

---

# Basic Example

Dockerfile

```dockerfile
FROM nginx

HEALTHCHECK \
CMD curl -f http://localhost || exit 1
```

---

Build

```bash
docker build -t nginx-health:v1 .
```

---

Run

```bash
docker run -d nginx-health:v1
```

---

Check

```bash
docker ps
```

---

Output

```text
healthy
```

---

# Health Check Parameters

Example

```dockerfile
HEALTHCHECK \
--interval=30s \
--timeout=5s \
--retries=3 \
CMD curl -f http://localhost || exit 1
```

---

# interval

```text
How Frequently
Health Check Runs
```

---

Example

```text
30 Seconds
```

---

# timeout

```text
Maximum Wait Time
```

---

Example

```text
5 Seconds
```

---

# retries

```text
Number Of Failures
Before Marking Unhealthy
```

---

Example

```text
3 Failures
```

---

# start-period

Example

```dockerfile
HEALTHCHECK \
--start-period=60s
```

---

Purpose

```text
Allow Application Startup Time
```

---

Useful for:

```text
Spring Boot

Tomcat

JBoss

WildFly
```

---

# Health Check Flow

```text
Container Starts
        ↓

Health Check Runs
        ↓

Success
        ↓

Healthy
```

---

Failure

```text
Health Check Fails
        ↓

Retry
        ↓

Retry
        ↓

Retry
        ↓

Unhealthy
```

---

# View Health Status

Command

```bash
docker ps
```

---

Example

```text
Up 10 minutes (healthy)
```

---

Detailed Information

```bash
docker inspect container-id
```

---

Shows

```text
Health Status

Last Check

Failure Details
```

---

# Docker Compose Health Check

Example

```yaml
services:

  catalogue:

    image: catalogue:v1

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

---

Benefits

```text
Automatic Health Monitoring

Better Dependency Handling
```

---

# Real Production Example

Catalogue Service

Health Endpoint

```text
/health
```

---

Health Check

```dockerfile
HEALTHCHECK \
CMD curl -f http://localhost:8080/health || exit 1
```

---

Docker verifies:

```text
Application Is Serving Requests
```

---

# What Happens If Health Check Fails?

Docker marks container:

```text
unhealthy
```

---

Applications like:

```text
Docker Compose

Kubernetes

Container Platforms
```

can take action.

---

# Resource Limits

## Why Resource Limits?

Containers share:

```text
CPU

Memory

Disk

Network
```

of the host.

---

Suppose

```text
Container A
```

consumes:

```text
100% Memory
```

---

Result

```text
Other Containers Impacted
```

---

# Resource Control

Docker allows limiting:

```text
CPU

Memory
```

usage.

---

# Memory Limits

Example

```bash
docker run \
-m 512m \
nginx
```

---

Meaning

```text
Maximum Memory

512 MB
```

---

Container cannot exceed:

```text
512 MB
```

---

# CPU Limits

Example

```bash
docker run \
--cpus="1" \
nginx
```

---

Meaning

```text
Maximum CPU

1 Core
```

---

# Combined Example

```bash
docker run \
-m 512m \
--cpus="1" \
nginx
```

---

Container receives:

```text
512 MB RAM

1 CPU
```

---

# Docker Compose Resource Limits

Example

```yaml
services:

  catalogue:

    deploy:
      resources:

        limits:
          memory: 512M

          cpus: '1'
```

---

Benefits

```text
Prevent Resource Exhaustion

Improve Stability
```

---

# Why Memory Limits Matter?

Without Limits

```text
Java Application
```

consumes:

```text
8 GB RAM
```

---

Host Memory

```text
8 GB
```

---

Result

```text
OOM

Other Containers Fail
```

---

With Limit

```bash
-m 512m
```

---

Container restricted.

---

# OOMKilled

Very Common Interview Topic.

---

Meaning

```text
Out Of Memory
```

---

Container exceeds:

```text
Memory Limit
```

---

Linux Kernel kills process.

---

Check

```bash
docker inspect container-id
```

---

Look for

```text
OOMKilled=true
```

---

# Resource Limits in Kubernetes

Docker

```bash
-m 512m
```

---

Kubernetes

```yaml
resources:

  requests:
    memory: "256Mi"

  limits:
    memory: "512Mi"
```

---

Concept is identical.

---

# Production Best Practices

## Health Checks

Use health checks for:

```text
APIs

Web Applications

Microservices
```

---

Avoid

```text
Only Checking Process
```

---

Check actual application response.

---

# Resource Limits

Always define:

```text
Memory Limits

CPU Limits
```

for production workloads.

---

Prevents:

```text
Noisy Neighbor Problem
```

---

# Architecture

```text
Container
      ↓

Health Check
      ↓

Healthy / Unhealthy
```

---

```text
Container
      ↓

CPU Limit

Memory Limit
      ↓

Controlled Resource Usage
```

---

# Common Interview Questions

## What is a Docker Health Check?

### Short Answer

A mechanism to verify that an application inside a container is functioning correctly.

### Detailed Explanation

Health checks periodically execute commands and determine whether an application is healthy or unhealthy.

### Production Example

Checking a Spring Boot /health endpoint.

### Follow-Up Questions

- What happens when a health check fails?
- How is health status viewed?

---

## Difference Between Running and Healthy?

### Short Answer

Running means the process exists; healthy means the application is working properly.

### Detailed Explanation

A container can be running while the application inside it is failing.

### Production Example

Catalogue process running but unable to connect to MongoDB.

### Follow-Up Questions

- Why are health checks required?
- How does Docker detect failures?

---

## What is OOMKilled?

### Short Answer

A container terminated because it exceeded its memory limit.

### Detailed Explanation

The Linux kernel kills processes when memory consumption exceeds allowed limits.

### Production Example

Java application consuming more than configured memory.

### Follow-Up Questions

- How do you verify OOMKilled?
- How do you prevent it?

---

## Why Use Resource Limits?

### Short Answer

To prevent a single container from consuming all host resources.

### Detailed Explanation

Resource limits improve stability and prevent noisy-neighbor issues.

### Production Example

Limiting a Java application to 512 MB RAM and 1 CPU.

### Follow-Up Questions

- How are limits configured?
- How does Kubernetes implement them?

---

# Key Takeaways

```text
Health checks verify application health, not just process status.

HEALTHCHECK instruction enables container health monitoring.

Healthy and Running are not the same.

Health checks are heavily used in Docker Compose and Kubernetes.

Resource limits control CPU and memory consumption.

Memory limits prevent excessive resource usage.

CPU limits prevent containers from monopolizing CPU resources.

OOMKilled indicates memory exhaustion.

Health checks and resource limits are essential production practices.

These concepts directly map to Kubernetes probes and resource limits.
```