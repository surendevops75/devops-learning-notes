# Docker CMD

## Introduction

CMD is one of the most important Dockerfile instructions.

It is used to:

```text
Start Applications

Run Commands

Keep Containers Alive
```

when a container starts.

---

CMD defines:

```text
Default Command
```

for a container.

---

Without CMD or ENTRYPOINT:

```text
Container May Start

And Exit Immediately
```

---

# What is CMD?

CMD specifies:

```text
Default Command
```

that Docker executes when the container starts.

---

Example

```dockerfile
FROM almalinux:9

CMD ["echo","Hello Docker"]
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
Hello Docker
```

---

Container Stops.

---

Reason:

```text
Command Completed
```

---

# When Does CMD Execute?

CMD executes during:

```text
Container Startup
```

---

NOT during:

```text
Image Build
```

---

Interview Question

```text
RUN vs CMD
```

---

RUN

```text
Build Time
```

---

CMD

```text
Run Time
```

---

# CMD Execution Flow

```text
Dockerfile
      ↓
docker build
      ↓
Image Created
      ↓
docker run
      ↓
CMD Executes
```

---

# CMD Forms

Docker supports:

```text
Exec Form

Shell Form
```

---

# Exec Form (Recommended)

Example

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

Advantages

```text
Better Signal Handling

Preferred By Docker

Production Standard
```

---

# Shell Form

Example

```dockerfile
CMD nginx -g "daemon off;"
```

---

Works.

---

But:

```text
Less Preferred
```

for production.

---

# Why CMD is Important?

Every container needs:

```text
Main Process
```

---

Examples

```text
Nginx Container
      ↓
nginx

NodeJS Container
      ↓
node app.js

Java Container
      ↓
java -jar app.jar
```

---

CMD defines:

```text
What Process Runs
```

when container starts.

---

# Container Lifecycle

```text
Container Starts
        ↓
CMD Executes
        ↓
Process Running
        ↓
Container Running
```

---

If Process Stops:

```text
Container Stops
```

---

# Why Containers Exit?

Example

```dockerfile
CMD ["echo","hello"]
```

---

Run

```bash
docker run test:v1
```

---

Output

```text
hello
```

---

Container Exits.

---

Reason:

```text
Process Finished
```

---

Docker only keeps container alive while:

```text
Main Process Runs
```

---

# Foreground vs Background

Very Important Concept.

---

Bad Example

```dockerfile
CMD ["nginx"]
```

---

Historically nginx:

```text
Starts

Moves To Background
```

---

Result

```text
Container Stops
```

---

Because:

```text
Main Process Ended
```

---

Correct Example

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

Meaning:

```text
Run In Foreground
```

---

Container remains:

```text
Running
```

---

# Why systemctl Does Not Work?

Common Mistake

```dockerfile
CMD ["systemctl","start","nginx"]
```

---

Usually fails.

---

Reason

Containers do not run:

```text
systemd

init

full operating system
```

---

Containers generally run:

```text
Single Process
```

---

Therefore:

```text
systemctl
```

cannot manage services.

---

# CMD Can Be Overridden

Dockerfile

```dockerfile
CMD ["echo","hello"]
```

---

Run

```bash
docker run image ls
```

---

Docker Executes

```text
ls
```

---

CMD Replaced.

---

# Why Override CMD?

Useful when:

```text
Testing

Debugging

Troubleshooting
```

---

Example

Dockerfile

```dockerfile
CMD ["node","server.js"]
```

---

Temporarily Run

```bash
docker run image bash
```

---

Container Starts:

```text
Bash Shell
```

instead of:

```text
Node Application
```

---

# Multiple CMD Instructions

Example

```dockerfile
CMD ["echo","hello"]

CMD ["echo","docker"]

CMD ["echo","world"]
```

---

Result

```text
Last CMD Wins
```

---

Docker Uses

```text
CMD ["echo","world"]
```

---

Previous CMD Instructions Ignored.

---

# Real Production Examples

## Nginx

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

## NodeJS

```dockerfile
CMD ["node","server.js"]
```

---

## Java

```dockerfile
CMD ["java","-jar","app.jar"]
```

---

## Python

```dockerfile
CMD ["python","app.py"]
```

---

# Production Troubleshooting

Container Not Running

---

Check

```bash
docker ps -a
```

---

Output

```text
Exited
```

---

Check Logs

```bash
docker logs container-id
```

---

Often Cause

```text
CMD Finished

Wrong Command

Application Crash
```

---

# Common Interview Questions

## What is CMD?

### Short Answer

CMD specifies the default command executed when a container starts.

### Detailed Explanation

CMD runs during container startup and defines the container's default behavior.

### Production Example

CMD ["node","server.js"]

### Follow-Up Questions

- Can CMD be overridden?
- When does CMD execute?

---

## Difference Between RUN and CMD?

### Short Answer

RUN executes during image build. CMD executes during container startup.

### Detailed Explanation

RUN configures the image while CMD starts the application.

### Production Example

RUN installs nginx, CMD starts nginx.

### Follow-Up Questions

- Which creates image layers?
- Which executes during runtime?

---

## Why Do Containers Exit Immediately?

### Short Answer

The main process defined by CMD exits.

### Detailed Explanation

Docker keeps containers alive only while the primary process is running.

### Production Example

CMD ["echo","hello"] causes immediate container exit.

### Follow-Up Questions

- How do you keep containers alive?
- Why must applications run in foreground?

---

# Key Takeaways

```text
CMD defines the default command for a container.

CMD executes during container startup.

CMD can be overridden.

Only the last CMD instruction is used.

Containers stay alive while the main process runs.

Foreground processes are required.

systemctl typically does not work inside containers.

CMD is one of the most important Dockerfile instructions.
```