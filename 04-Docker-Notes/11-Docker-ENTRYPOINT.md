# Docker ENTRYPOINT

## Introduction

ENTRYPOINT is one of the most important Dockerfile instructions.

It is used to define:

```text
Main Executable

Primary Process

Fixed Container Behavior
```

---

Like CMD:

```text
ENTRYPOINT Executes
When Container Starts
```

---

However:

```text
ENTRYPOINT Is Designed To Be Fixed

CMD Is Designed To Be Flexible
```

---

This distinction is extremely important in production environments.

---

# What is ENTRYPOINT?

ENTRYPOINT defines:

```text
The Main Command
```

that always executes when a container starts.

---

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

Docker Executes

```text
echo hello
```

---

Notice:

```text
ENTRYPOINT Remains

Argument Is Appended
```

---

# Why ENTRYPOINT Exists?

Many containers have:

```text
One Purpose
```

---

Examples:

```text
Nginx Container

MySQL Container

Redis Container

Application Container
```

---

Container should always start:

```text
Specific Process
```

---

ENTRYPOINT ensures:

```text
Process Cannot Be Accidentally Replaced
```

---

# ENTRYPOINT Execution Flow

```text
Dockerfile
      ↓
docker build
      ↓
Image Created
      ↓
docker run
      ↓
ENTRYPOINT Executes
```

---

# ENTRYPOINT Forms

Docker supports:

```text
Exec Form

Shell Form
```

---

# Exec Form (Recommended)

Example

```dockerfile
ENTRYPOINT ["nginx","-g","daemon off;"]
```

---

Benefits

```text
Signal Handling

PID 1 Management

Production Standard
```

---

Most commonly used.

---

# Shell Form

Example

```dockerfile
ENTRYPOINT nginx -g "daemon off;"
```

---

Works.

---

But:

```text
Less Preferred
```

---

Production Dockerfiles generally use:

```text
Exec Form
```

---

# ENTRYPOINT Cannot Be Easily Overridden

Dockerfile

```dockerfile
ENTRYPOINT ["echo"]
```

---

Run

```bash
docker run entry:v1 hello
```

---

Result

```text
echo hello
```

---

Run

```bash
docker run entry:v1 world
```

---

Result

```text
echo world
```

---

ENTRYPOINT:

```text
Still Exists
```

---

Runtime arguments:

```text
Appended
```

---

# Runtime Arguments

Dockerfile

```dockerfile
ENTRYPOINT ["ping"]
```

---

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

Run

```bash
docker run ping:v1 localhost
```

---

Docker Executes

```text
ping localhost
```

---

ENTRYPOINT remains:

```text
ping
```

---

Arguments change.

---

# Why This Is Useful?

Suppose image purpose:

```text
Always Run Ping
```

---

Only destination should vary.

---

ENTRYPOINT provides:

```text
Fixed Command

Flexible Parameters
```

---

# Real Production Example

Application Image

Dockerfile

```dockerfile
ENTRYPOINT ["node"]
```

---

Run

```bash
docker run app:v1 server.js
```

---

Docker Executes

```text
node server.js
```

---

Another Example

```bash
docker run app:v1 app.js
```

---

Docker Executes

```text
node app.js
```

---

ENTRYPOINT:

```text
node
```

remains unchanged.

---

# Why ENTRYPOINT is Preferred for Applications?

Application container exists to run:

```text
Specific Application
```

---

Examples

```text
Catalogue Service

User Service

Cart Service

Frontend Service
```

---

Purpose:

```text
Run Application
```

---

ENTRYPOINT ensures:

```text
Container Always Starts Application
```

---

# ENTRYPOINT and Container Lifecycle

Container Start

↓

ENTRYPOINT Executes

↓

Application Starts

↓

Application Running

↓

Container Running

---

Application Stops

↓

Container Stops

---

# Foreground Requirement

Just like CMD:

```text
ENTRYPOINT Must Run Foreground Process
```

---

Example

```dockerfile
ENTRYPOINT ["nginx","-g","daemon off;"]
```

---

Meaning

```text
Run Nginx In Foreground
```

---

Container stays:

```text
Alive
```

---

# Bad Example

```dockerfile
ENTRYPOINT ["echo","hello"]
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

Container exits.

---

Reason

```text
Process Finished
```

---

# ENTRYPOINT with Applications

NodeJS

```dockerfile
ENTRYPOINT ["node","server.js"]
```

---

Java

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```

---

Python

```dockerfile
ENTRYPOINT ["python","app.py"]
```

---

Nginx

```dockerfile
ENTRYPOINT ["nginx","-g","daemon off;"]
```

---

# Production Troubleshooting

Container Exiting Immediately

---

Check

```bash
docker ps -a
```

---

Status

```text
Exited
```

---

Check Logs

```bash
docker logs container-id
```

---

Possible Causes

```text
Wrong ENTRYPOINT

Application Crash

Missing Files

Invalid Arguments
```

---

# Difference Between CMD and ENTRYPOINT (Quick View)

CMD

```text
Default Command

Can Be Replaced
```

---

ENTRYPOINT

```text
Main Command

Arguments Appended
```

---

We'll cover the full comparison in the next file.

---

# Common Interview Questions

## What is ENTRYPOINT?

### Short Answer

ENTRYPOINT defines the main executable that runs when a container starts.

### Detailed Explanation

ENTRYPOINT provides a fixed command while allowing runtime arguments to be appended.

### Production Example

ENTRYPOINT ["node"]

### Follow-Up Questions

- How is it different from CMD?
- Why use ENTRYPOINT?

---

## Can ENTRYPOINT Be Overridden?

### Short Answer

Not normally. Runtime arguments are appended rather than replacing it.

### Detailed Explanation

Docker treats ENTRYPOINT as the primary executable and appends additional parameters supplied at runtime.

### Production Example

ENTRYPOINT ["ping"] with runtime argument google.com.

### Follow-Up Questions

- What happens when arguments are supplied?
- Why is this behavior useful?

---

## Why Use ENTRYPOINT?

### Short Answer

To ensure containers always run their intended application.

### Detailed Explanation

ENTRYPOINT prevents accidental replacement of the primary application command.

### Production Example

Catalogue container always starting NodeJS application.

### Follow-Up Questions

- When should ENTRYPOINT be preferred?
- Can CMD be used together with ENTRYPOINT?

---

# Key Takeaways

```text
ENTRYPOINT defines the primary executable.

ENTRYPOINT executes when the container starts.

ENTRYPOINT is designed to remain fixed.

Runtime arguments are appended to ENTRYPOINT.

ENTRYPOINT is ideal for application containers.

Foreground processes are required.

Application termination causes container termination.

ENTRYPOINT is commonly used with NodeJS, Java, Python, and Nginx applications.

Understanding ENTRYPOINT is essential before learning CMD + ENTRYPOINT combinations.
```