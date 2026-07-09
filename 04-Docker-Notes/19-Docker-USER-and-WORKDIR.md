# Docker USER and WORKDIR

## Introduction

When building Docker images, applications should:

```text
Run With Least Privileges

Use A Consistent Working Directory

Improve Security

Improve Maintainability
```

---

Docker provides two important instructions:

```text
USER

WORKDIR
```

---

These instructions are heavily used in:

```text
Production Dockerfiles

Kubernetes

EKS

Security Hardening
```

---

# USER Instruction

## What is USER?

USER specifies:

```text
Which User

Will Execute Commands

Will Run The Container
```

---

By default:

```text
Docker Runs As root
```

---

Example

```dockerfile
FROM almalinux:9

CMD ["sleep","300"]
```

---

Container User

```text
root
```

---

Verify

```bash
docker exec -it container-id bash

whoami
```

---

Output

```text
root
```

---

# Why Is Running As Root Dangerous?

Suppose:

```text
Application Vulnerability
```

exists.

---

Attacker gains access.

---

Container User

```text
root
```

---

Attacker receives:

```text
Root Privileges
```

inside container.

---

Security Risk.

---

# Production Best Practice

Create dedicated user.

---

Example

```dockerfile
FROM almalinux:9

RUN useradd appuser

USER appuser

CMD ["sleep","300"]
```

---

Build

```bash
docker build -t user:v1 .
```

---

Run

```bash
docker run user:v1
```

---

Verify

```bash
docker exec -it container-id bash

whoami
```

---

Output

```text
appuser
```

---

Container no longer runs as:

```text
root
```

---

# USER Execution Flow

```text
Dockerfile
      ↓
Create User
      ↓
USER appuser
      ↓
Remaining Instructions
      ↓
Container Runs As appuser
```

---

# USER Affects Build Instructions

Example

```dockerfile
FROM almalinux:9

RUN useradd appuser

USER appuser

RUN touch test.txt
```

---

File Created By

```text
appuser
```

---

Not

```text
root
```

---

# Switching Users

Example

```dockerfile
FROM almalinux:9

RUN useradd appuser

USER appuser

RUN mkdir /tmp/app

USER root

RUN dnf install nginx -y
```

---

Docker allows:

```text
Multiple USER Instructions
```

---

Last USER becomes:

```text
Runtime User
```

unless changed again.

---

# Production Example

NodeJS Application

```dockerfile
FROM node:20

WORKDIR /app

COPY . .

RUN npm install

USER node

CMD ["node","server.js"]
```

---

Benefits

```text
Application Not Running As Root

Better Security
```

---

# USER with Numeric IDs

Example

```dockerfile
USER 1001
```

---

Useful in:

```text
OpenShift

Enterprise Kubernetes
```

---

# Common USER Mistakes

## Application Needs Root

Example

```dockerfile
USER appuser

RUN dnf install nginx -y
```

---

Fails.

---

Reason

```text
Package Installation Requires Root
```

---

Correct

```dockerfile
RUN dnf install nginx -y

USER appuser
```

---

# WORKDIR Instruction

## What is WORKDIR?

WORKDIR defines:

```text
Default Working Directory
```

for subsequent instructions.

---

Think of it as:

```text
Permanent cd Command
```

inside Dockerfile.

---

# Without WORKDIR

Example

```dockerfile
RUN mkdir /app

RUN cd /app
```

---

Problem

```text
cd Exists Only For That RUN Layer
```

---

Next RUN

```dockerfile
RUN pwd
```

---

Output

```text
/
```

---

Because:

```text
Every RUN Creates New Layer
```

---

# Using WORKDIR

Example

```dockerfile
WORKDIR /app
```

---

Now

```dockerfile
RUN pwd
```

---

Output

```text
/app
```

---

# Basic Example

```dockerfile
FROM almalinux:9

WORKDIR /app

RUN touch file1

RUN pwd
```

---

Output

```text
/app
```

---

# WORKDIR Automatically Creates Directory

Example

```dockerfile
WORKDIR /app
```

---

If:

```text
/app
```

does not exist

---

Docker automatically creates it.

---

No need for:

```dockerfile
RUN mkdir /app
```

---

# Real Production Example

NodeJS Application

```dockerfile
FROM node:20

WORKDIR /app

COPY . .

RUN npm install

CMD ["node","server.js"]
```

---

Meaning

```text
Application Files Stored In

/app
```

---

# Multiple WORKDIR Instructions

Example

```dockerfile
WORKDIR /app

WORKDIR logs
```

---

Result

```text
/app/logs
```

---

Docker appends paths.

---

Example

```dockerfile
WORKDIR /app

WORKDIR config
```

---

Final Directory

```text
/app/config
```

---

# WORKDIR with COPY

Without WORKDIR

```dockerfile
COPY . /app
```

---

With WORKDIR

```dockerfile
WORKDIR /app

COPY . .
```

---

Cleaner.

---

Production Standard.

---

# USER + WORKDIR Together

Very Common.

Example

```dockerfile
FROM node:20

WORKDIR /app

COPY . .

RUN npm install

USER node

CMD ["node","server.js"]
```

---

Flow

```text
Create Image
      ↓
Set Working Directory
      ↓
Copy Application
      ↓
Install Dependencies
      ↓
Switch User
      ↓
Start Application
```

---

# Production Security Pattern

```dockerfile
FROM node:20

WORKDIR /app

COPY . .

RUN npm install

RUN useradd appuser

RUN chown -R appuser:appuser /app

USER appuser

CMD ["node","server.js"]
```

---

Benefits

```text
Non-root Container

Secure File Ownership

Production Ready
```

---

# Common Interview Questions

## What is USER?

### Short Answer

USER specifies which user executes commands and runs the container.

### Detailed Explanation

Instead of running containers as root, USER allows applications to run with limited privileges.

### Production Example

USER appuser

### Follow-Up Questions

- Why avoid root?
- Can USER be changed multiple times?

---

## Why Should Containers Avoid Running As Root?

### Short Answer

To improve security.

### Detailed Explanation

If an application is compromised, attackers gain only limited privileges instead of root access.

### Production Example

Running NodeJS applications using appuser.

### Follow-Up Questions

- How do you create users?
- What happens if container runs as root?

---

## What is WORKDIR?

### Short Answer

WORKDIR sets the default working directory for Dockerfile instructions.

### Detailed Explanation

It acts like a permanent cd command and simplifies Dockerfiles.

### Production Example

WORKDIR /app

### Follow-Up Questions

- Does WORKDIR create directories?
- Can multiple WORKDIR instructions be used?

---

## Difference Between RUN cd and WORKDIR?

### Short Answer

RUN cd affects only one layer. WORKDIR affects all following instructions.

### Detailed Explanation

WORKDIR persists across Dockerfile instructions while RUN cd does not.

### Production Example

Using WORKDIR /app before COPY and npm install.

### Follow-Up Questions

- Why does RUN cd not persist?
- How does Docker layering affect this?

---

# Key Takeaways

```text
USER defines which user runs the container.

Containers run as root by default.

Running containers as non-root users is a security best practice.

USER can be switched multiple times.

WORKDIR sets the default working directory.

WORKDIR automatically creates directories if they do not exist.

WORKDIR is better than repeatedly using cd commands.

USER and WORKDIR are used in almost every production Dockerfile.

Combining WORKDIR and non-root USER improves maintainability and security.
```