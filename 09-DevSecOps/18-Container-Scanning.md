# Container Scanning

## Introduction

Container Scanning is the process of analyzing container images for security vulnerabilities, misconfigurations, malware, hardcoded secrets, and compliance issues before they are deployed.

In DevSecOps, Container Scanning ensures that only secure and trusted container images are deployed to Kubernetes, Docker, or cloud environments.

---

## Why Do We Need Container Scanning?

Container images are built using operating system packages, application dependencies, and third-party libraries.

Without Container Scanning:

- Vulnerable packages reach production
- Malware can be deployed
- Secrets may exist inside images
- Outdated base images remain in use
- Compliance violations go unnoticed

Container Scanning helps identify these risks before deployment.

---

## What is Container Scanning?

Container Scanning examines every layer of a container image to detect security issues.

It scans for:

- OS Package Vulnerabilities
- CVEs
- Outdated Base Images
- Hardcoded Secrets
- Misconfigurations
- Malware
- Compliance Violations
- License Issues

---

## How It Works

```text
Developer

↓

Docker Build

↓

Container Image

↓

Container Scanner

↓

Analyze Image Layers

↓

Check CVE Database

↓

Generate Report

↓

Fix Vulnerabilities

↓

Push Image
```

---

## Architecture

```text
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

Docker Build

↓

Container Scanner

↓

Vulnerability Database

↓

Security Report

↓

Container Registry

↓

Kubernetes
```

---

## Workflow

```text
Write Application

↓

Build Docker Image

↓

Scan Image

↓

Review Findings

↓

Fix Vulnerabilities

↓

Push Image

↓

Deploy
```

---

# What Does Container Scanning Detect?

## Operating System Vulnerabilities

Example

```text
Ubuntu Base Image

↓

Old OpenSSL Package

↓

Known CVE

↓

Critical Finding
```

---

## Vulnerable Application Libraries

Example

```text
Container

↓

Java Libraries

↓

Log4j

↓

Known Vulnerability
```

---

## Hardcoded Secrets

Examples

- AWS Access Keys
- Database Passwords
- API Keys
- Tokens

Example

```text
Docker Image

↓

Environment Variables

↓

Secret Found
```

---

## Outdated Base Images

Example

```text
ubuntu:18.04

↓

Unsupported

↓

Upgrade Required
```

---

## Malware Detection

Example

```text
Image Layer

↓

Malicious Binary

↓

Security Alert
```

---

## Misconfigurations

Examples

- Running as Root
- Privileged Containers
- Insecure File Permissions
- Weak User Configuration

---

# Image Layers

Container images are composed of multiple immutable layers.

```text
Application Layer

↓

Dependencies

↓

Runtime

↓

Operating System

↓

Base Image
```

Each layer is scanned independently.

---

# Popular Container Scanning Tools

| Tool | Purpose |
|------|---------|
| Trivy | Container & Kubernetes scanning |
| Clair | Container vulnerability scanning |
| Grype | CVE scanning |
| Prisma Cloud | Enterprise container security |
| Aqua Security | Container protection |
| Anchore | Container image analysis |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Build Application

↓

Docker Build

↓

Container Scan

↓

Review Report

↓

Fix Vulnerabilities

↓

Push Image

↓

Deploy Kubernetes

↓

Runtime Monitoring
```

Critical vulnerabilities should prevent container images from being pushed to the registry.

---

## Production Workflow

```text
Developer

↓

Docker Build

↓

Scan Image

↓

Approve Image

↓

Push Registry

↓

Deploy Kubernetes

↓

Runtime Monitoring
```

---

## Production Example

A development team builds a Docker image based on an outdated Ubuntu image.

```text
Docker Build

↓

Trivy Scan

↓

Critical OpenSSL CVE Found

↓

Pipeline Failed

↓

Update Base Image

↓

Rebuild Image

↓

Scan Passed

↓

Push Registry
```

The vulnerable image never reaches production.

---

## Best Practices

- Scan every image build
- Use minimal base images
- Keep base images updated
- Remove unnecessary packages
- Fail builds for critical vulnerabilities
- Scan before pushing images
- Store images in trusted registries
- Sign container images
- Continuously rescan registry images
- Combine image scanning with runtime security

---

## Common Mistakes

- Using outdated base images
- Ignoring critical CVEs
- Running containers as root
- Storing secrets inside images
- Skipping image scans
- Installing unnecessary packages
- Trusting public images without verification
- Scanning only production images

---

# Common Troubleshooting

## Issue 1

### Critical CVEs Detected During Build

**Cause**

The container image uses vulnerable packages.

**Resolution**

```text
Review Scan Report

↓

Update Base Image

↓

Upgrade Packages

↓

Rebuild Image

↓

Re-run Scan
```

---

## Issue 2

### Secrets Found Inside Image

**Cause**

Secrets were copied into the container during the build.

**Resolution**

```text
Remove Secrets

↓

Use Secrets Manager

↓

Rebuild Image

↓

Scan Again
```

---

## Issue 3

### Container Runs as Root

**Cause**

Dockerfile does not specify a non-root user.

**Resolution**

```text
Create User

↓

USER appuser

↓

Rebuild Image

↓

Deploy
```

---

## Issue 4

### Pipeline Takes Too Long

**Cause**

Every image layer is scanned from scratch.

**Resolution**

```text
Enable Cache

↓

Incremental Scan

↓

Parallel Jobs

↓

Complete Pipeline
```

---

## Issue 5

### New CVE Published After Deployment

**Cause**

A dependency becomes vulnerable after the image is already deployed.

**Resolution**

```text
Continuous Registry Scan

↓

Identify Affected Images

↓

Rebuild Image

↓

Redeploy
```

---

# Production Interview Questions

## Question 1

### What is Container Scanning?

Container Scanning is the process of analyzing container images for vulnerabilities, misconfigurations, secrets, malware, and compliance issues before deployment.

---

## Question 2

### Why is Container Scanning important?

It prevents vulnerable images from reaching production and reduces the risk of exploiting known software vulnerabilities.

---

## Question 3

### What types of issues can Container Scanning detect?

Operating system vulnerabilities, application dependency vulnerabilities, hardcoded secrets, malware, outdated base images, and image misconfigurations.

---

## Question 4

### Which container scanning tools have you used?

Common tools include Trivy, Grype, Clair, Anchore, Aqua Security, and Prisma Cloud.

---

## Question 5

### At which stage should Container Scanning be performed?

Immediately after building the container image and before pushing it to the container registry.

---

## Question 6

### Why should minimal base images be used?

Minimal images reduce the attack surface, decrease image size, improve performance, and reduce the number of vulnerabilities.

---

## Question 7

### Should a CI/CD pipeline fail if critical vulnerabilities are detected?

Yes. Images with critical vulnerabilities should not be pushed or deployed until the issues are resolved.

---

## Question 8

### Why shouldn't containers run as the root user?

Running as root increases the impact of a container compromise and may allow attackers to gain elevated privileges.

---

## Question 9

### Does Container Scanning replace runtime security?

No. Container Scanning secures images before deployment, while runtime security tools monitor containers after deployment.

---

## Question 10

### How does Container Scanning support DevSecOps?

Container Scanning integrates into CI/CD pipelines to identify insecure images, enforce security policies, block vulnerable deployments, and improve software supply chain security.

---

# Key Takeaways

- Container Scanning identifies vulnerabilities before deployment.
- Scan every container image during the CI/CD pipeline.
- Use updated and minimal base images.
- Never store secrets inside container images.
- Combine Container Scanning with runtime security, image signing, and Kubernetes security for a complete DevSecOps strategy.