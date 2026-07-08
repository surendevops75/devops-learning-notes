# Docker CMD vs ENTRYPOINT

## Introduction

CMD and ENTRYPOINT are among the most confusing Docker concepts.

Both are used to:

```text
Start Containers

Execute Commands

Run Applications
```

---

However their behavior is completely different.

---

Understanding CMD vs ENTRYPOINT is important because:

```text
Docker

Kubernetes

EKS

CI/CD Pipelines

Production Containers
```

depend heavily on these concepts.

---

# Quick Overview

CMD

```text
Default Command

Can Be Overridden
```

---

ENTRYPOINT

```text
Main Command

Arguments Are Appended
```

---

# CMD Recap

Example

```dockerfile
FROM almalinux:9

CMD ["echo","hello"]
```

---

Build

```bash
docker build -t cmd:v1 .
```

---

Run

```bash
docker run cmd:v1
```

---

Output

```text
hello
```

---

Override

```bash
docker run cmd:v1 ls
```

---

Output

```text
Directory Listing
```

---

Reason

```text
CMD Replaced
```

---

# ENTRYPOINT Recap

Example

```dockerfile
FROM almalinux:9

ENTRYPOINT ["echo"]
```

---

Build

```bash
docker build -t entry:v1 .
```

---

Run

```bash
docker run entry:v1 hello
```

---

Output

```text
echo hello
```

---

ENTRYPOINT:

```text
Remains Fixed
```

---

Argument:

```text
Appended
```

---

# Fundamental Difference

CMD

```text
Can Be Replaced
```

---

ENTRYPOINT

```text
Cannot Be Replaced Normally
```

---

# Example 1

CMD Only

Dockerfile

```dockerfile
FROM almalinux:9

CMD ["echo","hello"]
```

---

Run

```bash
docker run image
```

---

Output

```text
hello
```

---

Run

```bash
docker run image world
```

---

Output

```text
world
```

---

Docker Replaces

```text
echo hello
```

with

```text
world
```

---

# Example 2

ENTRYPOINT Only

Dockerfile

```dockerfile
FROM almalinux:9

ENTRYPOINT ["echo"]
```

---

Run

```bash
docker run image hello
```

---

Output

```text
echo hello
```

---

Run

```bash
docker run image world
```

---

Output

```text
echo world
```

---

ENTRYPOINT remains:

```text
echo
```

---

Arguments change.

---

# CMD + ENTRYPOINT Together

This is the most common production pattern.

---

ENTRYPOINT

```text
Main Executable
```

---

CMD

```text
Default Arguments
```

---

Example

```dockerfile
FROM almalinux:9

ENTRYPOINT ["ping"]

CMD ["localhost"]
```

---

Build

```bash
docker build -t ping:v1 .
```

---

Run

```bash
docker run ping:v1
```

---

Docker Executes

```text
ping localhost
```

---

# Overriding CMD

Run

```bash
docker run ping:v1 google.com
```

---

Docker Executes

```text
ping google.com
```

---

ENTRYPOINT

```text
Still ping
```

---

CMD

```text
Overridden
```

---

# Why CMD + ENTRYPOINT is Powerful?

Provides:

```text
Fixed Application

Flexible Parameters
```

---

Real Example

```dockerfile
ENTRYPOINT ["node"]

CMD ["server.js"]
```

---

Default

```bash
docker run catalogue:v1
```

---

Executes

```text
node server.js
```

---

Override

```bash
docker run catalogue:v1 app.js
```

---

Executes

```text
node app.js
```

---

# Production Use Case

Suppose:

```text
Catalogue Service
```

must always run:

```text
NodeJS
```

---

But script name may change.

---

ENTRYPOINT

```dockerfile
ENTRYPOINT ["node"]
```

---

CMD

```dockerfile
CMD ["server.js"]
```

---

Benefits

```text
Fixed Runtime

Flexible Startup Arguments
```

---

# How Docker Builds Final Command

Example

```dockerfile
ENTRYPOINT ["ping"]

CMD ["localhost"]
```

---

Docker Combines

```text
ENTRYPOINT + CMD
```

---

Result

```text
ping localhost
```

---

# Runtime Override

Command

```bash
docker run ping:v1 google.com
```

---

Docker Combines

```text
ping google.com
```

---

CMD replaced.

---

ENTRYPOINT retained.

---

# Common Production Examples

## Nginx

```dockerfile
ENTRYPOINT ["nginx"]

CMD ["-g","daemon off;"]
```

---

Final Command

```text
nginx -g daemon off;
```

---

## Java

```dockerfile
ENTRYPOINT ["java"]

CMD ["-jar","app.jar"]
```

---

Final Command

```text
java -jar app.jar
```

---

## Python

```dockerfile
ENTRYPOINT ["python"]

CMD ["app.py"]
```

---

Final Command

```text
python app.py
```

---

## NodeJS

```dockerfile
ENTRYPOINT ["node"]

CMD ["server.js"]
```

---

Final Command

```text
node server.js
```

---

# CMD Override Scenario

Dockerfile

```dockerfile
CMD ["node","server.js"]
```

---

Run

```bash
docker run image bash
```

---

Container Starts

```text
bash
```

---

Application Not Started.

---

This is useful for:

```text
Troubleshooting

Debugging

Manual Investigation
```

---

# ENTRYPOINT Scenario

Dockerfile

```dockerfile
ENTRYPOINT ["node"]
```

---

Run

```bash
docker run image bash
```

---

Docker Executes

```text
node bash
```

---

Usually fails.

---

Reason

```text
bash Treated As Argument
```

---

# Why ENTRYPOINT is Safer?

Prevents:

```text
Accidental Command Replacement
```

---

Ensures:

```text
Application Always Starts
```

---

Useful for:

```text
Production Containers

Business Applications

Microservices
```

---

# Architecture

```text
Container Start
       ↓

ENTRYPOINT
       ↓

CMD Arguments
       ↓

Final Command
       ↓

Application Running
```

---

# Common Interview Scenario

Question:

```text
Why Use CMD And ENTRYPOINT Together?
```

---

Answer:

```text
ENTRYPOINT Defines Main Command

CMD Provides Default Arguments
```

---

This gives:

```text
Control

Flexibility

Consistency
```

---

# Troubleshooting Example

Container Exits Immediately.

---

Check

```bash
docker ps -a
```

---

Check Logs

```bash
docker logs container-id
```

---

Possible Reasons

```text
Wrong CMD

Wrong ENTRYPOINT

Invalid Arguments

Application Failure
```

---

# CMD vs ENTRYPOINT Comparison

| Feature | CMD | ENTRYPOINT |
|----------|----------|----------|
| Purpose | Default Command/Arguments | Main Command |
| Can Be Overridden | Yes | No (Normally) |
| Runtime Parameters | Replace CMD | Appended To ENTRYPOINT |
| Typical Usage | Default Arguments | Main Executable |
| Production Usage | Often With ENTRYPOINT | Very Common |
| Flexibility | High | Lower |

---

# When To Use CMD?

Use CMD when:

```text
Users May Need To Change Command

Testing Required

Debugging Required
```

---

# When To Use ENTRYPOINT?

Use ENTRYPOINT when:

```text
Container Has Single Purpose

Application Must Always Start

Production Services
```

---

# When To Use Both?

Best Practice

```text
ENTRYPOINT → Main Executable

CMD → Default Arguments
```

---

Most production-grade Docker images use:

```text
CMD + ENTRYPOINT
```

together.

---

# Common Interview Questions

## Difference Between CMD and ENTRYPOINT?

### Short Answer

CMD provides default commands or arguments, while ENTRYPOINT defines the main executable.

### Detailed Explanation

CMD can be replaced completely. ENTRYPOINT remains fixed and runtime values are appended.

### Production Example

ENTRYPOINT ["node"] with CMD ["server.js"].

### Follow-Up Questions

- Can they be used together?
- Which one should define the application?

---

## Why Use CMD and ENTRYPOINT Together?

### Short Answer

To define a fixed executable with flexible arguments.

### Detailed Explanation

ENTRYPOINT specifies the application while CMD supplies default parameters that can be overridden.

### Production Example

node server.js

### Follow-Up Questions

- What happens when arguments are passed?
- Which one gets overridden?

---

## Which is Preferred in Production?

### Short Answer

ENTRYPOINT + CMD together.

### Detailed Explanation

This provides both consistency and flexibility.

### Production Example

Java, NodeJS, Python, and Nginx containers.

### Follow-Up Questions

- Why not only CMD?
- Why not only ENTRYPOINT?

---

# Key Takeaways

```text
CMD defines default commands or arguments.

ENTRYPOINT defines the main executable.

CMD can be overridden.

ENTRYPOINT remains fixed.

Runtime arguments replace CMD values.

Runtime arguments are appended to ENTRYPOINT.

CMD and ENTRYPOINT are commonly used together.

ENTRYPOINT defines the application.

CMD defines default parameters.

Using CMD + ENTRYPOINT together is a production best practice.
```