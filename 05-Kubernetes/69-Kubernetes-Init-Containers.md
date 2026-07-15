# Kubernetes Init Containers

## Introduction

Most applications have dependencies.

Examples:

```text
Database

Cache

Secrets

Configuration Files

External APIs
```

---

Before an application starts, these dependencies may need to be:

```text
Verified

Fetched

Initialized

Configured
```

---

This is where:

```text
Init Containers
```

become useful.

---

# What Are Init Containers?

Init Containers are special containers that run:

```text
Before Main Containers
```

---

They execute:

```text
One By One

Sequentially
```

---

Only after all Init Containers complete successfully:

```text
Main Container Starts
```

---

# Basic Flow

```text
Init Container 1

↓

Completed

↓

Init Container 2

↓

Completed

↓

Init Container 3

↓

Completed

↓

Main Application Starts
```

---

# Important Rule

All Init Containers must:

```text
Complete Successfully
```

---

If even one Init Container fails:

```text
Main Container

Will Not Start
```

---

# Why Use Init Containers?

Common Use Cases:

```text
Dependency Validation

Secrets Retrieval

Configuration Generation

Database Migrations

Environment Preparation
```

---

# Production Example

Application:

```text
Catalogue Service
```

---

Dependencies:

```text
MySQL

Redis

Secrets Manager
```

---

Before Catalogue Starts:

```text
Verify MySQL

Verify Redis

Fetch Password
```

---

Then:

```text
Start Application
```

---

# Init Container vs Main Container

| Feature | Init Container | Main Container |
|----------|----------|----------|
| Runs First | Yes | No |
| Runs Sequentially | Yes | No |
| Long Running | No | Yes |
| Must Complete | Yes | N/A |
| Used For Setup | Yes | No |

---

# Example Architecture

```text
Pod

├── Init Container 1
│    Check Database
│
├── Init Container 2
│    Fetch Secret
│
├── Init Container 3
│    Run Migration
│
└── Main Container
     Start Application
```

---

# Use Case 1 - Dependency Check

Suppose:

```text
Application

Depends On MySQL
```

---

Without Validation:

```text
Application Starts

↓

MySQL Not Ready

↓

Application Crash
```

---

Better Approach

```text
Init Container

Checks MySQL

↓

MySQL Ready

↓

Application Starts
```

---

# Example

Init Container

```bash
until mysql is reachable
do
  sleep 5
done
```

---

Then

```text
Main Container Starts
```

---

# Use Case 2 - Fetch Secrets

Production applications often require:

```text
Database Passwords

API Keys

Tokens
```

---

Secrets should not be hardcoded.

---

Init Container can:

```text
Read Secrets

Store Temporarily

Pass To Application
```

---

# Example Flow

```text
Init Container

↓

AWS Secrets Manager

↓

Fetch Password

↓

Store In Shared Volume

↓

Main Container Reads Password
```

---

# Use Case 3 - Generate Configuration

Suppose:

```text
Environment Variables

Need Dynamic Configuration
```

---

Init Container Creates:

```text
config.yaml

application.properties

.env
```

---

Before Application Starts.

---

# Use Case 4 - Database Migrations

Common Production Scenario.

---

Application Version

```text
v5
```

---

Database Schema

```text
v4
```

---

Need Migration.

---

Flow

```text
Run Migration Script

↓

Update Schema

↓

Start Application
```

---

# Why Not Run Migration In Application?

Because:

```text
Application May Start

Before Migration Completes
```

---

Result

```text
Failures
```

---

# Sequential Execution

Example

```text
Init-1

Check MySQL
```

---

Success

↓

```text
Init-2

Fetch Secrets
```

---

Success

↓

```text
Init-3

Run Migration
```

---

Success

↓

```text
Application Starts
```

---

# Failure Scenario

Init Container

```text
Fetch Secret
```

---

Fails

↓

```text
Application Never Starts
```

---

This is intentional.

---

# Pod Status

When Init Containers Are Running

```text
Init:0/3

Init:1/3

Init:2/3
```

---

Meaning

```text
Initialization In Progress
```

---

# Example

Command

```bash
kubectl get pods
```

---

Output

```text
catalogue

Init:2/3
```

---

Meaning

```text
2 Init Containers Completed

1 Remaining
```

---

# Viewing Init Container Logs

Command

```bash
kubectl logs pod-name \
-c init-container-name
```

---

Example

```bash
kubectl logs catalogue \
-c mysql-check
```

---

# Describe Pod

Command

```bash
kubectl describe pod catalogue
```

---

Useful For

```text
Init Container Failures

Events

Exit Codes
```

---

# Shared Volumes

Init Containers often work with:

```text
emptyDir Volume
```

---

Flow

```text
Init Container

Writes Secret

↓

Shared Volume

↓

Main Container Reads Secret
```

---

# Production Example

Application

```text
Catalogue
```

---

Init Container

```text
Fetch MySQL Password
```

---

Writes

```text
/tmp/mysql-password
```

---

Main Container

```text
Reads Password

Sets Environment Variable

Starts Application
```

---

# Docker Entrypoint Integration

Many production applications use:

```text
Custom Entrypoint Scripts
```

---

Flow

```text
Init Container

↓

Create Password File

↓

Entrypoint Script Reads File

↓

Exports Environment Variable

↓

Starts Application
```

---

# Example

Docker Image

```dockerfile
ENTRYPOINT ["docker-entrypoint.sh"]
```

---

Application Startup

```text
Read Password

↓

Export Variable

↓

Execute Main Process
```

---

# Init Container Example YAML

```yaml
apiVersion: v1
kind: Pod

spec:

  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:

  - name: mysql-check
    image: busybox

    command:
    - sh
    - -c
    - |
      until nc -z mysql 3306
      do
        sleep 5
      done

  containers:

  - name: app
    image: catalogue:5.0
```

---

# Production Secret Fetching Architecture

```text
Pod

↓

Init Container

↓

AWS Secrets Manager

↓

Shared Volume

↓

Application Container

↓

Application Starts
```

---

# Benefits Of Init Containers

```text
Dependency Validation

Secure Secret Retrieval

Database Migration Support

Configuration Generation

Controlled Startup
```

---

# Common Production Problems

## Init Container Stuck

Symptoms

```text
Init:0/1
```

---

Cause

```text
Dependency Unavailable
```

---

Example

```text
Database Down
```

---

# Secret Fetch Failure

Cause

```text
IAM Permission Issue

IRSA Issue

AWS Access Issue
```

---

Result

```text
Application Never Starts
```

---

# Migration Failure

Cause

```text
SQL Error

Database Lock

Connection Failure
```

---

Result

```text
Pod Stuck In Init State
```

---

# Troubleshooting Commands

Pods

```bash
kubectl get pods
```

---

Logs

```bash
kubectl logs pod-name \
-c init-container
```

---

Describe

```bash
kubectl describe pod pod-name
```

---

Events

```bash
kubectl get events
```

---

# Common Interview Questions

## What Are Init Containers?

### Short Answer

Init Containers are special containers that run before the main application container starts.

### Detailed Explanation

They perform initialization tasks such as dependency validation, secret retrieval, configuration generation, and database migrations. All Init Containers must complete successfully before the application starts.

### Production Example

```text
Check MySQL

↓

Fetch Secrets

↓

Run Migration

↓

Start Application
```

### Follow-Up Questions

- Why not use a normal container?
- Can multiple init containers exist?
- Do init containers run in parallel?
- What happens if one fails?

---

## What Happens If An Init Container Fails?

### Short Answer

The main application container will never start.

### Detailed Explanation

Kubernetes requires every Init Container to complete successfully before proceeding to the next Init Container or the main container.

### Production Example

```text
Fetch Secret

↓

Failure

↓

Application Does Not Start
```

### Follow-Up Questions

- How do you troubleshoot failures?
- Where do you check logs?
- What pod status appears?
- How do retries work?

---

## Why Use Init Containers For Database Migrations?

### Short Answer

To ensure schema changes complete before application startup.

### Detailed Explanation

Applications may fail if they start before required database schema updates are applied.

### Production Example

```text
Migration Script

↓

Schema Updated

↓

Application Starts
```

### Follow-Up Questions

- Why not run migrations inside the application?
- What happens if migration fails?
- How do you rollback migrations?
- What tools perform migrations?

---

## How Are Secrets Retrieved Using Init Containers?

### Short Answer

The Init Container retrieves secrets before the application starts and stores them in a shared volume.

### Detailed Explanation

This pattern prevents secrets from being embedded in container images and ensures secrets are available before application startup.

### Production Example

```text
Secrets Manager

↓

Init Container

↓

Shared Volume

↓

Application
```

### Follow-Up Questions

- Why use a shared volume?
- How is access controlled?
- What happens if secret retrieval fails?
- What AWS service stores the secret?

---

## How Do You Troubleshoot Init Containers?

### Short Answer

Check pod status, logs, events, and container exit codes.

### Detailed Explanation

Most Init Container issues involve dependency failures, secret retrieval problems, or migration errors.

### Production Example

Commands:

```bash
kubectl get pods

kubectl logs pod-name -c init-container

kubectl describe pod pod-name
```

### Follow-Up Questions

- What does Init:0/1 mean?
- Where are exit codes shown?
- How do you debug networking issues?
- How do you identify permission failures?