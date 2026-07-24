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

# Configure SonarQube as a Service

Create a systemd service so SonarQube starts automatically after server reboot.

Create the service file.

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

Add the following configuration.

```ini
[Unit]
Description=SonarQube Service
After=network.target postgresql.service

[Service]
Type=forking

User=sonar
Group=sonar

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

Restart=always

LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

Reload systemd.

```bash
sudo systemctl daemon-reload
```

Enable the service.

```bash
sudo systemctl enable sonarqube
```

Start SonarQube.

```bash
sudo systemctl start sonarqube
```

Verify status.

```bash
sudo systemctl status sonarqube
```

Expected output

```text
active (running)
```

---

# Configure Nginx Reverse Proxy

In production, SonarQube should not be exposed directly on port **9000**.

Recommended Architecture

```text
Internet

↓

HTTPS :443

↓

Nginx

↓

SonarQube :9000
```

Install Nginx.

```bash
sudo apt install nginx -y
```

Create configuration.

```bash
sudo vi /etc/nginx/sites-available/sonarqube
```

Example configuration.

```nginx
server {

    listen 80;

    server_name sonar.company.com;

    location / {

        proxy_pass http://127.0.0.1:9000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }

}
```

Enable configuration.

```bash
sudo ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/
```

Test configuration.

```bash
sudo nginx -t
```

Restart Nginx.

```bash
sudo systemctl restart nginx
```

---

# Enable HTTPS

For production deployments, always use HTTPS.

Example using Let's Encrypt.

```bash
sudo apt install certbot python3-certbot-nginx
```

Generate certificate.

```bash
sudo certbot --nginx
```

Now SonarQube becomes available at

```text
https://sonar.company.com
```

---

# First Login

Open

```text
https://sonar.company.com
```

Default credentials

```text
Username

admin

Password

admin
```

Immediately change the default password.

---

# Create Your First Project

Navigate to

```text
Projects

↓

Create Project
```

Example

```text
Project Name

payment-service

Project Key

payment-service
```

---

# Generate Project Token

Navigate

```text
Project

↓

Administration

↓

Security

↓

Generate Token
```

Example

```text
payment-service-token
```

Save the token securely.

It will later be used by Jenkins or GitHub Actions.

---

# Create Quality Profile

Navigate

```text
Quality Profiles
```

Select language.

Example

```text
Java
```

Copy the built-in profile.

```text
Sonar Way

↓

Copy

↓

Company Java Profile
```

Customize rules according to organizational standards.

---

# Configure Quality Gate

Navigate

```text
Quality Gates
```

Example production Quality Gate.

| Metric | Condition |
|---------|-----------|
| Bugs | 0 Blocker |
| Vulnerabilities | 0 Critical |
| Security Hotspots | Reviewed |
| Coverage | >80% |
| Duplicate Code | <3% |

Pipeline flow.

```text
Sonar Analysis

↓

Quality Gate

↓

PASS

↓

Continue Deployment

OR

FAIL

↓

Pipeline Stops
```

---

# Authentication Methods

Enterprise deployments rarely use only local users.

Supported authentication.

```text
SonarQube

↓

Local Authentication

LDAP

GitHub OAuth

GitLab OAuth

Azure AD

SAML

OIDC
```

---

# LDAP Integration

Common in enterprise environments.

```text
Developer

↓

Corporate Active Directory

↓

LDAP

↓

SonarQube
```

Example configuration.

```properties
sonar.security.realm=LDAP

ldap.url=ldap://ldap.company.com

ldap.bindDn=cn=admin,dc=company,dc=com

ldap.bindPassword=********
```

Benefits

- Centralized authentication
- No local passwords
- Automatic onboarding
- Easier user management

---

# OAuth / OIDC Authentication

Many organizations integrate SonarQube with GitHub or Azure AD.

```text
Developer

↓

GitHub Login

↓

OAuth

↓

SonarQube
```

Advantages

- Single Sign-On (SSO)
- MFA support
- Central identity management

---

# Users and Groups

Instead of assigning permissions directly to users, companies manage access through groups.

Example.

```text
Users

↓

Groups

↓

Permissions
```

Recommended groups.

```text
Administrators

Security Team

Developers

QA Team

Auditors

CI Service Accounts
```

---

# Production RBAC

Example permission model.

| Role | Responsibilities |
|------|------------------|
| System Administrator | Full administration, plugins, upgrades, users, global settings |
| Project Administrator | Manage project settings, quality profiles, branches, permissions |
| Security Team | Review vulnerabilities, hotspots, security reports |
| Developer | Execute analyses, view issues, resolve assigned findings |
| QA Team | View quality reports and coverage metrics |
| Auditor | Read-only access for compliance and audit reviews |
| CI Service Account | Execute pipeline scans using project token only |

**Best Practice:** Never use the default `admin` account in CI/CD pipelines. Create a dedicated **service account** with the minimum required permissions and use project-specific tokens stored in your CI/CD secret manager.

---

# API Tokens

Instead of username/password authentication, CI/CD pipelines should use API tokens.

Example

```text
Jenkins

↓

Credentials Manager

↓

SonarQube Token

↓

Pipeline Authentication
```

Never hardcode tokens in pipeline scripts or repositories.

Store them in:

- Jenkins Credentials
- GitHub Secrets
- GitLab CI Variables
- Azure Key Vault
- HashiCorp Vault
- AWS Secrets Manager

---

