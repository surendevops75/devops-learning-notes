# Docker Compose Advanced

## Introduction

As applications grow, a simple Docker Compose file becomes insufficient.

Production applications require:

```text
Environment Variables

Health Checks

Custom Networks

Persistent Storage

Restart Policies

Scaling

Resource Management
```

---

Docker Compose provides advanced features to handle these requirements.

---

# Environment Variables

Environment variables allow applications to receive configuration dynamically.

Example

```yaml
services:

  catalogue:
    image: catalogue:v1

    environment:
      MONGO_HOST: mongodb
      APP_ENV: dev
```

---

Equivalent Docker Run

```bash
docker run \
-e MONGO_HOST=mongodb \
-e APP_ENV=dev
```

---

Benefits

```text
No Hardcoded Values

Environment Specific Configuration

Reusable Images
```

---

# Using .env File

Instead of storing values directly:

```yaml
environment:
  DB_HOST: mongodb
  DB_USER: admin
```

---

Use

```text
.env
```

file.

---

Example

```text
DB_HOST=mongodb
DB_USER=admin
APP_ENV=dev
```

---

Compose File

```yaml
services:

  catalogue:
    image: catalogue:v1

    environment:
      DB_HOST: ${DB_HOST}
      DB_USER: ${DB_USER}
```

---

Start

```bash
docker compose up
```

---

Compose automatically loads:

```text
.env
```

---

Benefits

```text
Cleaner Compose Files

Easy Environment Management

Reusable Configurations
```

---

# Variable Substitution

Compose supports variable expansion.

Example

```yaml
services:

  frontend:
    image: frontend:${VERSION}
```

---

.env

```text
VERSION=v1
```

---

Result

```text
frontend:v1
```

---

# depends_on

Applications often depend on other services.

Example

```text
Catalogue
      ↓

MongoDB
```

---

Compose supports:

```yaml
depends_on
```

---

Example

```yaml
services:

  catalogue:
    image: catalogue:v1

    depends_on:
      - mongodb

  mongodb:
    image: mongo
```

---

Meaning

```text
Start MongoDB First
```

---

# Important Limitation

depends_on only controls:

```text
Container Startup Order
```

---

It does NOT guarantee:

```text
Application Readiness
```

---

Example

```text
MongoDB Container Started

MongoDB Still Initializing
```

---

Catalogue may fail.

---

# Health Checks

Health checks verify whether a service is actually ready.

---

Example

```yaml
services:

  mongodb:

    image: mongo

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

Benefits

```text
Detect Unhealthy Services

Improve Reliability

Enable Better Dependency Management
```

---

# Restart Policies

Containers can fail unexpectedly.

Compose supports automatic restart.

---

Example

```yaml
services:

  catalogue:
    image: catalogue:v1

    restart: always
```

---

Available Policies

```text
no

always

unless-stopped

on-failure
```

---

# Restart Policy Examples

Always Restart

```yaml
restart: always
```

---

Restart Only On Failure

```yaml
restart: on-failure
```

---

Benefits

```text
Higher Availability

Automatic Recovery
```

---

# Custom Networks

Compose automatically creates a network.

---

However custom networks provide:

```text
Isolation

Controlled Communication

Network Segmentation
```

---

Example

```yaml
networks:

  roboshop:
```

---

Services

```yaml
services:

  catalogue:
    image: catalogue:v1

    networks:
      - roboshop
```

---

# Multiple Services on Same Network

```yaml
services:

  mongodb:
    image: mongo

    networks:
      - roboshop

  catalogue:
    image: catalogue:v1

    networks:
      - roboshop

networks:
  roboshop:
```

---

Communication

```text
catalogue
      ↓

mongodb
```

---

Using service names.

---

# Named Volumes

Containers are ephemeral.

If a container is deleted:

```text
Data Is Lost
```

---

Named volumes solve this problem.

---

Example

```yaml
services:

  mongodb:
    image: mongo

    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

---

Benefits

```text
Persistent Data

Docker Managed Storage
```

---

# Scaling Services

Compose supports scaling.

---

Example

```bash
docker compose up \
--scale catalogue=3 -d
```

---

Result

```text
catalogue-1

catalogue-2

catalogue-3
```

---

Benefits

```text
Load Distribution

Testing Multiple Instances
```

---

# Multiple Compose Files

Common Practice

```text
Development

Testing

Production
```

use different configurations.

---

Files

```text
docker-compose.yml

docker-compose.prod.yml
```

---

Run

```bash
docker compose \
-f docker-compose.yml \
-f docker-compose.prod.yml \
up -d
```

---

Benefits

```text
Environment Specific Configurations
```

---

# Resource Limits

Containers should not consume unlimited resources.

---

Example

```yaml
services:

  catalogue:

    deploy:
      resources:
        limits:
          memory: 512M
```

---

Benefits

```text
Prevent Resource Exhaustion

Improve Stability
```

---

# Logging Configuration

Default logging may consume disk space.

---

Example

```yaml
services:

  catalogue:

    logging:
      options:
        max-size: "10m"
        max-file: "3"
```

---

Benefits

```text
Controlled Log Growth

Prevent Disk Full Issues
```

---

# Production Compose Example

```yaml
services:

  mongodb:
    image: mongo

    volumes:
      - mongo-data:/data/db

  catalogue:
    image: catalogue:v1

    depends_on:
      - mongodb

    restart: always

    environment:
      MONGO_HOST: mongodb

networks:
  default:

volumes:
  mongo-data:
```

---

# Common Commands

Validate Compose File

```bash
docker compose config
```

---

View Running Services

```bash
docker compose ps
```

---

View Logs

```bash
docker compose logs
```

---

Restart Services

```bash
docker compose restart
```

---

Stop Application

```bash
docker compose down
```

---

# Troubleshooting

Check Configuration

```bash
docker compose config
```

---

View Container Logs

```bash
docker compose logs catalogue
```

---

Check Running Containers

```bash
docker compose ps
```

---

Inspect Network

```bash
docker network ls
```

---

Inspect Volumes

```bash
docker volume ls
```

---

# Common Interview Questions

## What is the purpose of .env file?

### Short Answer

Stores environment-specific configuration values separately from the Compose file.

### Detailed Explanation

Compose automatically loads variables from .env and substitutes them inside docker-compose.yml.

### Production Example

Database host, usernames, and environment names.

### Follow-Up Questions

- How does variable substitution work?
- Why not hardcode values?

---

## What is depends_on?

### Short Answer

Defines service startup order.

### Detailed Explanation

Compose starts dependent services first but does not guarantee application readiness.

### Production Example

Catalogue depends on MongoDB.

### Follow-Up Questions

- Does it check application health?
- How do health checks help?

---

## Why use named volumes?

### Short Answer

To persist data across container restarts and recreation.

### Detailed Explanation

Docker manages the volume lifecycle independently of containers.

### Production Example

MongoDB storing database files.

### Follow-Up Questions

- What happens if a container is deleted?
- Where are volumes stored?

---

## What is scaling in Docker Compose?

### Short Answer

Running multiple instances of the same service.

### Detailed Explanation

Compose can start multiple containers for a service using the --scale option.

### Production Example

Running three catalogue containers.

### Follow-Up Questions

- How is traffic distributed?
- How is scaling handled in Kubernetes?

---

# Key Takeaways

```text
Docker Compose supports advanced application management.

.env files improve configuration management.

depends_on manages startup order.

Health checks verify service readiness.

Restart policies improve availability.

Custom networks provide isolation and service discovery.

Named volumes persist application data.

Scaling allows multiple service instances.

Multiple Compose files support different environments.

Resource limits prevent excessive resource usage.

Docker Compose is widely used for development and small-scale deployments.
```