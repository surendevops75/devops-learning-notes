# NodeJS Backend Deployment and Systemd

## Introduction

Backend applications contain the business logic of an application.

Responsibilities include:

- Processing API requests
- Authentication and Authorization
- CRUD Operations
- Database Communication
- Business Logic Execution
- Data Validation

Typical Architecture:

```text
User
  │
  ▼
Frontend (Nginx)
  │
  ▼
Backend Application (NodeJS)
  │
  ▼
MySQL Database
```

---

# Backend Deployment Overview

A backend deployment generally involves:

1. Install Programming Language Runtime
2. Create Application User
3. Create Application Directory
4. Download Application Code
5. Install Dependencies
6. Configure Database Connectivity
7. Create Systemd Service
8. Start Application
9. Verify Connectivity

---

# Service Users

A non-human user that does not require interactive login is called a:

```text
System User
```

or

```text
Service User
```

Examples:

```text
nginx
mysql
jenkins
expense
```

Advantages:

- Better Security
- Limited Permissions
- Isolation of Services

---

# Create Application User

```bash
useradd expense
```

Verify:

```bash
id expense
```

Example Output:

```text
uid=1001(expense)
gid=1001(expense)
```

---

# Application Directory

Create application directory:

```bash
mkdir /app
```

Verify:

```bash
ls -ld /app
```

---

# Application Dependencies

Applications depend on external libraries.

These libraries are managed through dependency files.

---

# Dependency Files by Technology

| Technology | Dependency File |
|------------|----------------|
| NodeJS | package.json |
| Java | pom.xml |
| Python | requirements.txt |
| .NET | .sln, .csproj |
| Golang | go.mod |

---

# NodeJS Dependency File

## package.json

Contains:

- Project Name
- Description
- Version
- Dependencies
- Scripts

Example:

```json
{
  "name": "expense-backend",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

# Download Application Code

Backend application package:

```bash
curl -o /tmp/backend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
```

Explanation:

```text
curl    -> Download utility
-o      -> Output file
/tmp    -> Temporary directory
backend.zip -> Downloaded file
```

---

# Verify Download

```bash
ls -l /tmp/backend.zip
```

---

# Extract Application

Install unzip if required:

```bash
dnf install unzip -y
```

Extract:

```bash
unzip /tmp/backend.zip -d /app
```

---

# Install NodeJS

## Disable Default Version

```bash
dnf module disable nodejs -y
```

---

## Enable NodeJS 20

```bash
dnf module enable nodejs:20 -y
```

---

## Install NodeJS

```bash
dnf install nodejs -y
```

---

# Verify Installation

```bash
node -v
```

```bash
npm -v
```

---

# Install Application Dependencies

Navigate to application directory:

```bash
cd /app
```

Install dependencies:

```bash
npm install
```

---

# What Does npm install Do?

NPM reads:

```text
package.json
```

Downloads all required libraries and creates:

```text
node_modules/
```

directory.

---

# Backend and Database Relationship

Backend applications depend on databases.

Before starting backend application:

Verify:

- Database Server Running
- Database Accessible
- Schema Exists
- Tables Exist

---

# Database Connectivity Details

Applications generally require:

```text
Database Host
Database Port
Database Username
Database Password
Database Name
```

Example:

```text
Host     : 172.31.30.95
Port     : 3306
User     : root
Password : ExpenseApp@1
```

---

# Verify Database Connectivity

Using Telnet:

```bash
telnet 172.31.30.95 3306
```

Successful connection means:

```text
Backend Server can reach Database Server
```

---

# Load Database Schema

Before running application, create required tables.

Example:

```bash
mysql -h 172.31.30.95 -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

Explanation:

```text
-h    -> Database Host
-u    -> Username
-p    -> Password
<     -> Redirect SQL File
```

---

# What is a Schema?

A schema contains:

- Tables
- Views
- Procedures
- Relationships

Hierarchy:

```text
Database Server
      │
      ▼
Database
      │
      ▼
Schema
      │
      ▼
Tables
```

---

# Custom Application Services

Unlike nginx or mysql, custom applications do not automatically integrate with systemd.

To manage applications using:

```bash
systemctl start
systemctl stop
systemctl restart
```

we create a service file.

---

# Systemd Service Files

Location:

```text
/etc/systemd/system/
```

Examples:

```text
backend.service
frontend.service
payment.service
```

---

# Backend Service File

Location:

```text
/ etc/systemd/system/backend.service
```

Contents:

```ini
[Unit]
Description=Backend Service

[Service]
User=expense
Environment=DB_HOST="172.31.30.95"
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
```

---

# Understanding Service File

## [Unit]

Service description.

```ini
Description=Backend Service
```

---

## [Service]

Defines runtime behavior.

Application user:

```ini
User=expense
```

Database host:

```ini
Environment=DB_HOST="172.31.30.95"
```

Application start command:

```ini
ExecStart=/bin/node /app/index.js
```

Logging identifier:

```ini
SyslogIdentifier=backend
```

---

## [Install]

Controls boot-time behavior.

```ini
WantedBy=multi-user.target
```

---

# Reload Systemd

Whenever a new service file is created:

```bash
systemctl daemon-reload
```

Purpose:

```text
Reload systemd configuration
```

---

# Start Backend Service

```bash
systemctl start backend
```

---

# Enable Backend Service

```bash
systemctl enable backend
```

Meaning:

```text
Start automatically after reboot
```

---

# Check Service Status

```bash
systemctl status backend
```

Example:

```text
active (running)
```

---

# Restart Service

```bash
systemctl restart backend
```

---

# Stop Service

```bash
systemctl stop backend
```

---

# Backend Application Port

Most backend applications commonly run on:

```text
8080
```

Verify:

```bash
ss -lntp
```

or

```bash
netstat -lntp
```

---

# Verify Backend Process

```bash
ps -ef | grep node
```

Example:

```text
expense 1234 node /app/index.js
```

---

# Verify Backend Logs

Using journalctl:

```bash
journalctl -u backend
```

Follow logs:

```bash
journalctl -fu backend
```

---

# Deployment Workflow

```text
Install NodeJS
      │
      ▼
Create User
      │
      ▼
Create /app
      │
      ▼
Download Code
      │
      ▼
Extract Code
      │
      ▼
npm install
      │
      ▼
Load Database Schema
      │
      ▼
Create backend.service
      │
      ▼
daemon-reload
      │
      ▼
Start Service
      │
      ▼
Verify Application
```

---

# Common Troubleshooting

## Backend Service Not Starting

Check:

```bash
systemctl status backend
```

---

View logs:

```bash
journalctl -u backend
```

---

Verify NodeJS:

```bash
node -v
```

---

Verify dependencies:

```bash
npm install
```

---

Verify database connectivity:

```bash
telnet 172.31.30.95 3306
```

---

Verify schema:

```bash
mysql -h 172.31.30.95 -uroot -pExpenseApp@1
```

---

Verify process:

```bash
ps -ef | grep node
```

---

Verify port:

```bash
ss -lntp | grep 8080
```

---

# Frequently Asked Interview Questions

## What is a Service User?

A non-human user used to run applications and services.

Example:

```bash
useradd expense
```

---

## Why Do We Use package.json?

To define project metadata and dependencies.

---

## What Does npm install Do?

Reads package.json and installs required dependencies.

---

## Why Create a systemd Service?

To manage custom applications using:

```bash
systemctl
```

commands.

---

## What Does systemctl daemon-reload Do?

Reloads systemd configuration after creating or modifying service files.

---

## Why Check Database Connectivity Before Starting Backend?

Backend applications depend on databases.

Without database connectivity, application startup may fail.

---

## What Port Do Backend Applications Commonly Use?

```text
8080
```

---

## How Do You Verify Backend is Running?

```bash
systemctl status backend
```

```bash
ps -ef | grep node
```

```bash
ss -lntp
```

---

# Key Takeaways

- Backend applications contain business logic.
- Service users improve security.
- NodeJS dependencies are defined in package.json.
- npm install downloads required libraries.
- Backend applications depend on database connectivity.
- Database schemas and tables must exist before application startup.
- Custom applications can be managed through systemd service files.
- systemctl daemon-reload is required after service file changes.
- Backend applications commonly run on port 8080.
- journalctl is useful for troubleshooting application services.
- Verifying connectivity, logs, processes, and ports is essential during deployment and troubleshooting.