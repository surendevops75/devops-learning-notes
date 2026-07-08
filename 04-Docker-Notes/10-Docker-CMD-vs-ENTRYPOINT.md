# Docker CMD vs ENTRYPOINT

## Introduction

One of the most confusing Docker topics is:

```text
CMD

ENTRYPOINT
```

---

Both are used to:

```text
Start Containers
```

but they behave differently.

---

This is one of the most frequently asked Docker interview questions.

---

# Why Do We Need CMD and ENTRYPOINT?

A Docker Image is:

```text
Static
```

---

When a container starts:

```text
Some Process Must Start
```

---

Example:

```text
Nginx Container
```

must start:

```text
nginx
```

---

NodeJS Container:

```text
node app.js
```

---

Java Container:

```text
java -jar app.jar
```

---

CMD and ENTRYPOINT define:

```text
What Should Run
When Container Starts
```

---

# CMD Instruction

## What is CMD?

CMD defines:

```text
Default Command
```

for a container.

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
CMD Completed
```

---

# CMD Executes At Runtime

Dockerfile

```dockerfile
FROM almalinux:9

CMD ["sleep","300"]
```

---

Build

```bash
docker build -t sleep:v1 .
```

---

Run

```bash
docker run sleep:v1
```

---

Container Runs:

```text
300 Seconds
```

---

CMD executes:

```text
At Container Startup
```

---

# CMD Can Be Overridden

Dockerfile

```dockerfile
FROM almalinux:9

CMD ["echo","hello"]
```

---

Run

```bash
docker run cmd:v1 ls
```

---

Output

```text
Directory Listing
```

---

Reason:

```text
CMD Overridden
```

---

Docker ignores:

```text
echo hello
```

and executes:

```text
ls
```

---

# Why CMD is Useful?

Provides:

```text
Default Behaviour
```

---

User can change:

```text
Runtime Command
```

if required.

---

# ENTRYPOINT Instruction

## What is ENTRYPOINT?

ENTRYPOINT defines:

```text
Fixed Command
```

for the container.

---

Container always starts with:

```text
Specified Command
```

---

Example

```dockerfile
FROM almalinux:9

ENTRYPOINT ["ping"]
```

---

Build

```bash
docker build -t ping:v1 .
```

---

Run

```bash
docker run ping:v1 google.com
```

---

Actual Command

```text
ping google.com
```

---

# ENTRYPOINT Behavior

ENTRYPOINT remains:

```text
Fixed
```

---

Arguments supplied at runtime are:

```text
Appended
```

---

Example

Dockerfile

```dockerfile
ENTRYPOINT ["ping"]
```

---

Command

```bash
docker run ping:v1 localhost
```

---

Docker Executes

```text
ping localhost
```

---

# Why ENTRYPOINT Exists?

Used when:

```text
Container Has One Purpose
```

---

Examples

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

# CMD vs ENTRYPOINT

## CMD

```text
Default Command

Can Be Overridden
```

---

## ENTRYPOINT

```text
Main Command

Cannot Be Easily Overridden
```

---

# Example

CMD

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

Result

```text
ls Executes
```

---

CMD Replaced.

---

# Example

ENTRYPOINT

Dockerfile

```dockerfile
ENTRYPOINT ["echo"]
```

---

Run

```bash
docker run image hello
```

---

Result

```text
echo hello
```

---

ENTRYPOINT Remains.

---

Argument Added.

---

# Combining ENTRYPOINT and CMD

This is the most powerful approach.

---

ENTRYPOINT

```text
Main Command
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

# Override CMD Only

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
Unchanged
```

---

CMD

```text
Overridden
```

---

# Production Example

Dockerfile

```dockerfile
FROM node:20

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

Override CMD

```bash
docker run catalogue:v1 app.js
```

---

Executes

```text
node app.js
```

---

# Why CMD + ENTRYPOINT Together?

Provides:

```text
Fixed Application

Flexible Arguments
```

---

Common Production Pattern.

---

# Exec Form vs Shell Form

## Exec Form (Recommended)

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

Benefits

```text
Better Signal Handling

Preferred By Docker
```

---

## Shell Form

```dockerfile
CMD nginx -g "daemon off;"
```

---

Works but:

```text
Less Preferred
```

---

# Why systemctl Fails?

Dockerfile

```dockerfile
CMD ["systemctl","start","nginx"]
```

---

Usually fails.

---

Reason:

```text
No systemd

No init process
```

inside container.

---

Containers run:

```text
Single Main Process
```

---

# Foreground Requirement

Container stays alive while:

```text
Main Process Runs
```

---

Example

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

---

Meaning:

```text
Run Nginx In Foreground
```

---

# Bad Example

```dockerfile
CMD ["echo","hello"]
```

---

Output

```text
hello
```

---

Container exits immediately.

---

Reason:

```text
Process Finished
```

---

# Container Lifecycle

```text
Container Start
        ↓
CMD / ENTRYPOINT Executes
        ↓
Process Running
        ↓
Container Running
```

---

If Process Stops

```text
Container Stops
```

---

# Production Examples

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

# Common Interview Questions

## Difference Between CMD and ENTRYPOINT?

### Short Answer

CMD provides default commands or arguments, while ENTRYPOINT defines the main executable.

### Detailed Explanation

CMD can be overridden completely. ENTRYPOINT remains fixed and runtime arguments are appended.

### Production Example

ENTRYPOINT ["node"] with CMD ["server.js"].

### Follow-Up Questions

- Can CMD and ENTRYPOINT be used together?
- Which one is overridden?

---

## Can CMD Be Overridden?

### Short Answer

Yes.

### Detailed Explanation

Commands supplied during docker run replace CMD values.

### Production Example

docker run image ls

### Follow-Up Questions

- What happens to ENTRYPOINT?
- How are CMD arguments overridden?

---

## Can ENTRYPOINT Be Overridden?

### Short Answer

Not normally.

### Detailed Explanation

Runtime values are appended to ENTRYPOINT rather than replacing it.

### Production Example

ENTRYPOINT ["ping"] plus runtime argument google.com.

### Follow-Up Questions

- Why use ENTRYPOINT?
- How does Docker combine ENTRYPOINT and CMD?

---

## Why Does systemctl Not Work In Containers?

### Short Answer

Containers do not run systemd.

### Detailed Explanation

Containers typically run a single foreground process rather than a complete operating system.

### Production Example

systemctl start nginx fails inside Docker containers.

### Follow-Up Questions

- What should start the application?
- Why must processes run in foreground?

---

# CMD vs ENTRYPOINT Summary

| Feature | CMD | ENTRYPOINT |
|----------|----------|----------|
| Purpose | Default Command/Arguments | Main Executable |
| Runtime Override | Yes | No (Arguments Appended) |
| Common Usage | Default Arguments | Fixed Command |
| Used Together | Yes | Yes |
| Production Usage | Frequently | Frequently |

---

# Key Takeaways

```text
CMD defines default commands or arguments.

ENTRYPOINT defines the main executable.

CMD can be overridden during docker run.

ENTRYPOINT remains fixed.

CMD and ENTRYPOINT are often used together.

ENTRYPOINT defines the command.

CMD defines default arguments.

Containers stay alive while the main process runs.

Foreground processes are required.

Understanding CMD and ENTRYPOINT is critical for Docker, Kubernetes, and CI/CD pipelines.
```