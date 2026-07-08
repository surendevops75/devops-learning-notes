# Docker ENV (Runtime Variables)

## Introduction

Applications often require configuration values.

Examples:

```text
Database Host

Database Port

Application Environment

API URLs

Log Levels
```

---

Hardcoding these values inside source code creates:

```text
Poor Flexibility

Deployment Problems

Environment Differences
```

---

Docker provides:

```text
ENV
```

to solve this problem.

---

# What is ENV?

ENV stands for:

```text
Environment Variable
```

---

ENV is used to define:

```text
Runtime Configuration
```

for containers.

---

Unlike ARG:

```text
ENV Remains Available
Inside Running Containers
```

---

# Why ENV?

Without ENV

```python
db_host = "mysql-prod-server"
```

---

Problem:

```text
Need Different Values For

DEV

QA

UAT

PROD
```

---

Changing code repeatedly:

```text
Not Recommended
```

---

Use ENV instead.

---

# ENV Syntax

```dockerfile
ENV KEY=VALUE
```

---

Example

```dockerfile
ENV APP_ENV=dev
```

---

Container receives:

```text
APP_ENV=dev
```

---

# Multiple ENV Variables

Dockerfile

```dockerfile
ENV APP_ENV=dev

ENV PORT=8080

ENV LOG_LEVEL=INFO
```

---

Alternative

```dockerfile
ENV APP_ENV=dev \
    PORT=8080 \
    LOG_LEVEL=INFO
```

---

Result

```text
Three Environment Variables Created
```

---

# How ENV Works

Dockerfile

```dockerfile
FROM almalinux:9

ENV APP_ENV=dev
```

---

Build Image

```bash
docker build -t env:v1 .
```

---

Run Container

```bash
docker run env:v1
```

---

Inside Container

```bash
env
```

---

Output

```text
APP_ENV=dev
```

---

# Verify ENV Variables

Access Container

```bash
docker exec -it container-id bash
```

---

Display Variables

```bash
env
```

or

```bash
printenv
```

---

Check Specific Variable

```bash
echo $APP_ENV
```

---

Output

```text
dev
```

---

# Real Example

Dockerfile

```dockerfile
FROM node:20

ENV APP_ENV=dev

ENV PORT=8080
```

---

Application Reads

```text
APP_ENV

PORT
```

at runtime.

---

# Runtime Override

ENV values can be overridden.

---

Dockerfile

```dockerfile
ENV APP_ENV=dev
```

---

Run

```bash
docker run -e APP_ENV=prod image
```

---

Result

```text
APP_ENV=prod
```

---

Runtime value wins.

---

# docker run -e

Syntax

```bash
docker run -e KEY=VALUE image
```

---

Example

```bash
docker run \
-e DB_HOST=mysql \
-e DB_PORT=3306 \
image
```

---

Container receives:

```text
DB_HOST=mysql

DB_PORT=3306
```

---

# Why Runtime Override?

Same image can be used for:

```text
DEV

QA

UAT

PROD
```

---

Only environment variables change.

---

Example

DEV

```bash
docker run -e DB_HOST=mysql-dev image
```

---

PROD

```bash
docker run -e DB_HOST=mysql-prod image
```

---

Same image.

Different configuration.

---

# Production Example

Catalogue Service

Dockerfile

```dockerfile
ENV MONGO_HOST=mongodb

ENV LOG_LEVEL=INFO
```

---

Container

```bash
docker run \
-e MONGO_HOST=prod-mongodb \
catalogue:v1
```

---

Application connects to:

```text
prod-mongodb
```

---

No code change required.

---

# ENV and Application Configuration

Applications commonly read:

```text
Database Details

API Endpoints

Environment Names

Feature Flags

Logging Levels
```

from environment variables.

---

# Common Environment Variables

Database

```text
DB_HOST

DB_PORT

DB_USER
```

---

Application

```text
APP_ENV

LOG_LEVEL

APP_NAME
```

---

Cloud

```text
AWS_REGION

AWS_PROFILE
```

---

# ENV in Docker Compose

Example

```yaml
services:
  catalogue:
    image: catalogue:v1
    environment:
      APP_ENV: prod
      DB_HOST: mongodb
```

---

Container receives:

```text
APP_ENV=prod

DB_HOST=mongodb
```

---

# ENV in Kubernetes

Very Common.

---

Deployment

```yaml
env:
  - name: APP_ENV
    value: prod

  - name: DB_HOST
    value: mongodb
```

---

Pod receives:

```text
APP_ENV=prod

DB_HOST=mongodb
```

---

# ENV vs Hardcoding

Bad

```python
db_host = "prod-db"
```

---

Good

```python
db_host = os.getenv("DB_HOST")
```

---

Benefits

```text
Flexible

Reusable

Environment Independent
```

---

# ENV and Secrets

ENV can store:

```text
Configuration
```

---

But avoid storing:

```text
Passwords

API Keys

Tokens
```

directly in Dockerfiles.

---

Bad Example

```dockerfile
ENV DB_PASSWORD=password123
```

---

Reason

```text
Visible In Image Metadata

Visible Through docker inspect
```

---

# Better Approaches

Use:

```text
Docker Secrets

Kubernetes Secrets

AWS Secrets Manager

HashiCorp Vault
```

---

# ENV Lifecycle

```text
Dockerfile
      ↓
Image Build
      ↓
Image Created
      ↓
Container Starts
      ↓
ENV Available
```

---

Unlike ARG:

```text
ENV Remains Available
```

after container starts.

---

# Real DevOps Example

Roboshop Catalogue

Dockerfile

```dockerfile
ENV MONGO_HOST=mongodb
```

---

DEV

```bash
docker run \
-e MONGO_HOST=dev-mongo
```

---

QA

```bash
docker run \
-e MONGO_HOST=qa-mongo
```

---

PROD

```bash
docker run \
-e MONGO_HOST=prod-mongo
```

---

Same image.

Different environments.

---

# Architecture

```text
Dockerfile
      ↓
ENV Variable
      ↓
Image
      ↓
Container
      ↓
Application Reads Value
```

---

# Common Interview Questions

## What is ENV?

### Short Answer

ENV defines environment variables available inside running containers.

### Detailed Explanation

ENV provides runtime configuration values that applications can access during execution.

### Production Example

Database host and application environment configuration.

### Follow-Up Questions

- How do you override ENV values?
- Where are ENV variables stored?

---

## How Do You Override ENV Variables?

### Short Answer

Using docker run -e.

### Detailed Explanation

Runtime values supplied using -e take precedence over values defined in the Dockerfile.

### Production Example

docker run -e APP_ENV=prod image

### Follow-Up Questions

- Which value wins?
- Can multiple variables be supplied?

---

## Is ENV Available Inside Containers?

### Short Answer

Yes.

### Detailed Explanation

ENV variables are stored in image metadata and are available to running containers.

### Production Example

echo $APP_ENV inside a container.

### Follow-Up Questions

- How do you list all variables?
- What command shows them?

---

## Should Secrets Be Stored Using ENV?

### Short Answer

Generally no.

### Detailed Explanation

ENV variables can be exposed through container inspection and image metadata.

### Production Example

Using AWS Secrets Manager instead of ENV for passwords.

### Follow-Up Questions

- What is a better alternative?
- How are secrets managed in Kubernetes?

---

# Key Takeaways

```text
ENV defines runtime environment variables.

ENV variables remain available inside running containers.

ENV values can be overridden using docker run -e.

ENV is heavily used for application configuration.

Same image can be reused across environments using ENV.

Applications should read configuration from environment variables.

ENV is commonly used in Docker Compose and Kubernetes.

Avoid storing secrets directly in ENV.

ENV is one of the most important Docker runtime configuration mechanisms.
```