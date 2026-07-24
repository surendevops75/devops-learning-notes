# SonarQube

## Introduction

SonarQube is an enterprise Static Application Security Testing (SAST) platform used to continuously inspect source code for security vulnerabilities, bugs, code smells, duplicate code, and maintainability issues.

In production environments, SonarQube is integrated into CI/CD pipelines to enforce Quality Gates and prevent insecure or low-quality code from reaching production.

---

# Why Companies Use SonarQube

Large organizations deploy hundreds of applications every day.

Manual code reviews alone cannot identify every security issue.

SonarQube automates code quality and security validation before deployment.

Benefits include:

- Detect security vulnerabilities early
- Improve code quality
- Reduce technical debt
- Enforce coding standards
- Block insecure releases
- Generate compliance reports
- Improve developer productivity

---

# Where SonarQube Fits in DevSecOps

SonarQube is one stage of a complete DevSecOps pipeline.

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Pipeline

↓

Build Application

↓

Unit Tests

↓

Code Coverage

↓

SonarQube Analysis

↓

Quality Gate

↓

Container Build

↓

Trivy Scan

↓

SBOM

↓

Cosign

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

SonarQube is executed immediately after unit testing and before container image creation.

If the Quality Gate fails, the pipeline stops immediately.

---

# Enterprise Architecture

```text
                        Developers
                             │
                             ▼
                        GitHub Repository
                             │
                             ▼
                    Jenkins / GitHub Actions
                             │
                             ▼
                     Sonar Scanner (CLI/Maven)
                             │
                             ▼
                     SonarQube Server
                    ┌──────────┴──────────┐
                    ▼                     ▼
             PostgreSQL Database     Quality Profiles
                    │                     │
                    └──────────┬──────────┘
                               ▼
                         Quality Gate
                               │
                      PASS ─────┴───── FAIL
                        │               │
                        ▼               ▼
                Continue Pipeline   Stop Pipeline
```

---

# Production Architecture

```text
Developer

↓

GitHub Enterprise

↓

Jenkins Controller

↓

Build Agent

↓

Sonar Scanner

↓

SonarQube Server

↓

PostgreSQL

↓

Quality Gate

↓

Docker Build

↓

Trivy Scan

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS
```

---

# Prerequisites

Before installing SonarQube, ensure the following components are available.

| Component | Version |
|------------|----------|
| Ubuntu Server | 22.04 LTS |
| Java | OpenJDK 17 |
| PostgreSQL | 15+ |
| RAM | Minimum 4 GB (8 GB Recommended) |
| CPU | 2 vCPU Minimum |
| Docker (Optional) | Latest |
| Jenkins | Recommended |
| Git | Latest |

---

# Network Ports

| Port | Purpose |
|------|----------|
| 9000 | SonarQube Web UI |
| 5432 | PostgreSQL |
| 22 | SSH |
| 80/443 | Reverse Proxy |

---

# SonarQube Deployment Options

Production environments generally use one of the following deployment models.

## Option 1

Dedicated Virtual Machine

```text
Ubuntu Server

↓

PostgreSQL

↓

SonarQube

↓

Nginx

↓

Developers
```

Recommended for medium-sized organizations.

---

## Option 2

Docker

```text
Docker Host

↓

PostgreSQL Container

↓

SonarQube Container

↓

Reverse Proxy
```

Suitable for small and medium environments.

---

## Option 3

Kubernetes

```text
Amazon EKS

↓

SonarQube Pod

↓

Persistent Volume

↓

PostgreSQL

↓

Ingress

↓

Developers
```

Recommended for enterprise-scale deployments.

---

# Production Installation (Ubuntu)

## Step 1 — Update Server

```bash
sudo apt update

sudo apt upgrade -y
```

---

## Step 2 — Install Java

```bash
sudo apt install openjdk-17-jdk -y
```

Verify installation

```bash
java -version
```

Expected output

```text
openjdk version "17"
```

---

## Step 3 — Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

Verify

```bash
sudo systemctl status postgresql
```

Status should be

```text
active (running)
```

---

## Step 4 — Create SonarQube Database

Login

```bash
sudo -u postgres psql
```

Create database

```sql
CREATE DATABASE sonarqube;
```

Create user

```sql
CREATE USER sonar WITH ENCRYPTED PASSWORD 'StrongPassword';
```

Grant permissions

```sql
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
```

Exit

```sql
\q
```

---

# Download SonarQube

Download the latest Community or Enterprise Edition from SonarSource.

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<version>.zip
```

Extract

```bash
sudo unzip sonarqube-*.zip -d /opt
```

Rename

```bash
sudo mv /opt/sonarqube-* /opt/sonarqube
```

---

# Create SonarQube User

```bash
sudo groupadd sonar

sudo useradd -r -g sonar -d /opt/sonarqube sonar

sudo chown -R sonar:sonar /opt/sonarqube
```

---

# Configure Database Connection

Edit

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

Configure

```properties
sonar.jdbc.username=sonar

sonar.jdbc.password=StrongPassword

sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

---

# Configure Java

```bash
sudo vi /opt/sonarqube/conf/wrapper.conf
```

Set Java Home if required.

---

# Linux Kernel Configuration

Production servers require Elasticsearch kernel settings.

Edit

```bash
sudo vi /etc/sysctl.conf
```

Add

```text
vm.max_map_count=524288

fs.file-max=131072
```

Apply

```bash
sudo sysctl -p
```

---

# Configure Limits

```bash
sudo vi /etc/security/limits.conf
```

Add

```text
sonar soft nofile 131072
sonar hard nofile 131072
sonar soft nproc 8192
sonar hard nproc 8192
```

---