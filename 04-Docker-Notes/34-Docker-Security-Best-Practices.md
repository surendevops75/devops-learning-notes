# Docker Security Best Practices

## Introduction

Containers make application deployment easier.

However, containers introduce security risks if not configured properly.

Common risks include:

```text
Running As Root

Vulnerable Images

Hardcoded Secrets

Excessive Privileges

Outdated Packages

Untrusted Images
```

---

A secure container should be:

```text
Minimal

Hardened

Scanned

Least Privileged

Immutable
```

---

# Why Container Security Matters?

Suppose:

```text
Application Vulnerability
```

exists inside a container.

---

If container runs as:

```text
root
```

---

Attacker may gain:

```text
Root Access
```

inside container.

---

This increases security risk.

---

# Security Layers

Container security should be applied at:

```text
Image Level

Container Level

Registry Level

Runtime Level

Kubernetes Level
```

---

Architecture

```text
Source Code
      ↓

Build Image
      ↓

Scan Image
      ↓

Store In Registry
      ↓

Deploy Container
      ↓

Monitor Runtime
```

---

# Use Official Images

Always prefer:

```text
Official Images
```

---

Examples

```text
nginx

redis

mongo

mysql

alpine
```

---

Avoid:

```text
Unknown Community Images
```

---

Reason

```text
Security Risks

Malware

Unmaintained Software
```

---

# Use Minimal Base Images

Bad

```dockerfile
FROM ubuntu
```

---

Better

```dockerfile
FROM alpine
```

---

Or

```dockerfile
FROM eclipse-temurin:17-jre
```

---

Benefits

```text
Smaller Image

Fewer Packages

Smaller Attack Surface
```

---

# Run Containers as Non-Root

Default

```text
root
```

---

Production Best Practice

```dockerfile
RUN useradd appuser

USER appuser
```

---

Benefits

```text
Least Privilege

Reduced Risk

Improved Security
```

---

# Principle of Least Privilege

Containers should receive:

```text
Only Required Permissions
```

---

Avoid

```text
Full Access

Privileged Mode

Root User
```

---

Goal

```text
Minimum Permissions
```

required for application operation.

---

# Avoid Privileged Containers

Dangerous

```bash
docker run --privileged
```

---

Container gains:

```text
Host-Level Capabilities
```

---

Can access:

```text
Kernel Features

Host Resources

Devices
```

---

Use only when absolutely necessary.

---

# Do Not Hardcode Secrets

Bad Example

```dockerfile
ENV DB_PASSWORD=admin123
```

---

Problem

```text
Password Stored In Image
```

---

Anyone with image access can view it.

---

Better

```text
Environment Variables

Secret Management Systems

Kubernetes Secrets
```

---

# Environment Variables for Configuration

Example

```bash
docker run \
-e DB_HOST=mysql \
-e DB_USER=admin
```

---

Configuration separated from image.

---

# Use Docker Secrets (Swarm)

For sensitive information:

```text
Passwords

API Keys

Certificates
```

---

Use secret management solutions.

---

# Keep Images Updated

Outdated images often contain:

```text
Known Vulnerabilities
```

---

Regularly update:

```text
Base Images

Packages

Runtime Components
```

---

Example

```dockerfile
FROM nginx:latest
```

---

Periodically rebuild images.

---

# Image Scanning

Container images should be scanned before deployment.

---

Common Tools

```text
Trivy

Grype

Snyk

Docker Scout
```

---

# Trivy Example

Scan Image

```bash
trivy image catalogue:v1
```

---

Checks

```text
OS Packages

Libraries

Known CVEs
```

---

Example Output

```text
CRITICAL

HIGH

MEDIUM

LOW
```

---

# Vulnerability Management

If vulnerabilities found:

```text
Update Packages

Update Base Image

Rebuild Image
```

---

Never ignore:

```text
Critical Vulnerabilities
```

---

# Multi-Stage Builds Improve Security

Bad

```dockerfile
FROM maven

COPY . .

RUN mvn package
```

---

Contains:

```text
Compiler

Build Tools

Caches
```

---

Better

```dockerfile
Builder Stage
      ↓

Runtime Stage
```

---

Final image contains:

```text
Application Only
```

---

Benefits

```text
Smaller

Safer

Cleaner
```

---

# Use .dockerignore

Exclude unnecessary files.

---

Example

```text
.git

node_modules

logs

temp
```

---

Benefits

```text
Smaller Context

Faster Build

Reduced Exposure
```

---

# Verify Image Contents

Before deployment verify:

```text
Application Files

Configurations

Libraries
```

---

Command

```bash
docker run -it image-name bash
```

---

Inspect

```bash
ls

cat

env
```

---

# Image Signing

Advanced Security Practice.

---

Purpose

```text
Verify Image Authenticity
```

---

Ensures:

```text
Image Not Tampered
```

---

Common Solutions

```text
Cosign

Notary
```

---

# Registry Security

Use:

```text
Private Repositories

IAM Controls

Access Policies
```

---

Examples

```text
ECR

ACR

GCR

Private Docker Hub
```

---

Benefits

```text
Controlled Access

Auditability

Security
```

---

# Resource Limits

Prevent containers from consuming excessive resources.

---

Example

```bash
docker run \
-m 512m \
--cpus=1 \
nginx
```

---

Benefits

```text
Stability

Isolation

Availability
```

---

# Health Checks

Monitor application health.

---

Example

```dockerfile
HEALTHCHECK \
CMD curl -f http://localhost:8080/health || exit 1
```

---

Benefits

```text
Detect Failures

Improve Reliability
```

---

# Immutable Containers

Container images should not change after deployment.

---

Build New Image

```text
Version 1
      ↓

Version 2
```

---

Avoid:

```text
Manual Changes Inside Running Container
```

---

Reason

```text
Inconsistent Environments
```

---

# Logging and Auditing

Monitor:

```text
Container Logs

Security Events

Image Changes
```

---

Commands

```bash
docker logs

docker events
```

---

Useful for:

```text
Incident Investigation

Auditing
```

---

# Supply Chain Security

Modern attacks often target:

```text
Dependencies

Libraries

Build Pipelines

Registries
```

---

Secure:

```text
Source Code

Build Pipeline

Registry

Runtime
```

---

# Production Security Checklist

```text
Use Official Images

Use Minimal Images

Run As Non-Root

Use Multi-Stage Builds

Scan Images

Avoid Hardcoded Secrets

Use Resource Limits

Use Health Checks

Keep Images Updated

Use Private Registries

Apply Least Privilege
```

---

# Real Production Example

CI/CD Pipeline

```text
GitHub
      ↓

Build Image
      ↓

Trivy Scan
      ↓

Push To ECR
      ↓

Deploy To EKS
```

---

Only:

```text
Scanned Images
```

are allowed into production.

---

# Common Interview Questions

## Why Should Containers Not Run As Root?

### Short Answer

Running as root increases security risk.

### Detailed Explanation

If a vulnerability is exploited, attackers gain elevated privileges inside the container.

### Production Example

Using USER appuser instead of root.

### Follow-Up Questions

- How do you create non-root users?
- What are the risks of root containers?

---

## Why Use Minimal Images?

### Short Answer

To reduce image size and attack surface.

### Detailed Explanation

Fewer packages mean fewer vulnerabilities and smaller deployment artifacts.

### Production Example

Using alpine instead of a full operating system image.

### Follow-Up Questions

- What is attack surface?
- Why does image size matter?

---

## What is Trivy?

### Short Answer

An image vulnerability scanner.

### Detailed Explanation

Trivy scans container images and reports known security vulnerabilities.

### Production Example

Scanning images before pushing to ECR.

### Follow-Up Questions

- What types of vulnerabilities are detected?
- How is Trivy used in CI/CD?

---

## Why Avoid Hardcoded Secrets?

### Short Answer

Secrets become visible to anyone with image access.

### Detailed Explanation

Passwords and API keys should be injected at runtime using secure secret management mechanisms.

### Production Example

Using Kubernetes Secrets instead of ENV instructions.

### Follow-Up Questions

- Where should secrets be stored?
- How does Kubernetes handle secrets?

---

# Key Takeaways

```text
Container security begins at image build time.

Use official and minimal base images.

Run containers as non-root users.

Avoid privileged containers.

Do not hardcode secrets.

Scan images regularly using tools like Trivy.

Use multi-stage builds to reduce attack surface.

Apply resource limits and health checks.

Keep images updated.

Use private registries and IAM controls.

Follow the principle of least privilege.

Security should be integrated throughout the container lifecycle.
```