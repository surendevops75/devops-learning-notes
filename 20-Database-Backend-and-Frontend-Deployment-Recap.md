# Database, Backend and Frontend Deployment Recap

## Introduction

A typical web application consists of three major components:

1. Database Layer
2. Backend Layer
3. Frontend Layer

All three layers work together to deliver applications to end users.

---

# Three Tier Architecture

```text
User
  │
  ▼
Frontend (Nginx)
  │
  ▼
Backend Application
  │
  ▼
Database
```

Responsibilities:

| Layer | Responsibility |
|---------|---------------|
| Frontend | User Interface |
| Backend | Business Logic |
| Database | Data Storage |

---

# Database Setup

## Database Installation

Install MySQL Server:

```bash
dnf install mysql-server -y
```

---

## Start MySQL Service

```bash
systemctl start mysqld
```

---

## Enable MySQL Service

```bash
systemctl enable mysqld
```

---

## Set Root Password

```bash
mysql_secure_installation --set-root-pass ExpenseApp@1
```

---

# MySQL Commands

Login:

```bash
mysql -u root -pExpenseApp@1
```

---

## Show Databases

```sql
show databases;
```

---

## Select Database

```sql
use <database-name>;
```

Example:

```sql
use transactions;
```

---

## Show Tables

```sql
show tables;
```

---

## View Data

```sql
select * from <table-name>;
```

Example:

```sql
select * from transactions;
```

---

# Backend Deployment

Backend applications contain business logic.

Popular Backend Technologies:

- NodeJS
- Java
- Python
- .NET
- Golang

---

# Backend Deployment Workflow

```text
Install Runtime
      │
      ▼
Create Service User
      │
      ▼
Create Application Directory
      │
      ▼
Download Application Code
      │
      ▼
Install Dependencies
      │
      ▼
Configure Database
      │
      ▼
Create Systemd Service
      │
      ▼
Start Application
```

---

# Install Runtime

Examples:

## NodeJS

```bash
dnf install nodejs -y
```

---

## Java

```bash
dnf install java -y
```

---

## Python

```bash
dnf install python3 -y
```

---

# Create Service User

```bash
useradd expense
```

Purpose:

- Security
- Isolation
- Least Privilege Access

---

# Create Application Directory

```bash
mkdir /app
```

---

# Download Application Code

Example:

```bash
curl -o /tmp/backend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
```

---

# Dependency Files

Different technologies use different dependency files.

| Technology | Dependency File |
|------------|----------------|
| NodeJS | package.json |
| Java | pom.xml |
| Python | requirements.txt |
| .NET | .sln, .csproj |
| Golang | go.mod |

---

# Install Dependencies

## NodeJS

```bash
npm install
```

Reads:

```text
package.json
```

and installs required libraries.

---

# Database Dependencies

Backend applications require:

- Database Host
- Port
- Username
- Password
- Schema
- Tables

Example:

```text
DB_HOST=mysql.daws86s.fun
```

---

# Load Database Schema

Developers usually provide SQL scripts.

Example:

```bash
mysql -h 172.31.30.95 -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

This script may:

```sql
create schema transactions;

create table transactions (...);

create user backend;

grant permissions;
```

---

# Backend Service Configuration

Location:

```text
/etc/systemd/system/backend.service
```

Example:

```ini
[Unit]
Description=Backend Service

[Service]
User=expense
Environment=DB_HOST="mysql.daws86s.fun"
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
```

---

# Reload Systemd

```bash
systemctl daemon-reload
```

---

# Start Backend

```bash
systemctl start backend
```

---

# Enable Backend

```bash
systemctl enable backend
```

---

# Check Backend Status

```bash
systemctl status backend
```

---

# Verify Backend Process

```bash
ps -ef | grep backend
```

or

```bash
ps -ef | grep node
```

---

# Verify Backend Port

```bash
netstat -lntp
```

or

```bash
ss -lntp
```

Backend applications commonly run on:

```text
8080
```

---

# Database Connectivity Testing

Verify backend can reach database:

```bash
telnet DB_IP 3306
```

Example:

```bash
telnet 172.31.30.95 3306
```

---

# Restart Backend

After schema updates:

```bash
systemctl restart backend
```

---

# Frontend Deployment

Frontend applications contain:

- HTML
- CSS
- JavaScript

Popular Frameworks:

- React
- Angular
- Vue

---

# Install Nginx

```bash
dnf install nginx -y
```

---

# Start Nginx

```bash
systemctl start nginx
```

---

# Enable Nginx

```bash
systemctl enable nginx
```

---

# Nginx Default Frontend Directory

```text
/usr/share/nginx/html
```

Nginx serves frontend content from this location.

---

# Nginx Configuration Directory

```text
/etc/nginx
```

Contains:

```text
nginx.conf
conf.d/
default.d/
```

---

# Remove Default Website

Delete default content if required.

Example:

```bash
rm -rf /usr/share/nginx/html/*
```

---

# Deploy Frontend

Download frontend code.

Example:

```bash
curl -o /tmp/frontend.zip <frontend-url>
```

Extract:

```bash
unzip frontend.zip -d /usr/share/nginx/html
```

---

# Configure Reverse Proxy

File:

```text
/etc/nginx/default.d/expense.conf
```

Example:

```nginx
location /api/ {
    proxy_pass http://BACKEND-IP:8080/;
}
```

Purpose:

```text
Frontend → Backend
```

---

# Restart Nginx

```bash
systemctl restart nginx
```

---

# Verify Backend Connectivity

Frontend server should reach backend server.

Example:

```bash
telnet backend_IP 8080
```

---

# Complete Application Flow

```text
User
  │
  ▼
Frontend (Nginx)
  │
  ▼
Backend Application
  │
  ▼
Database
```

---

# Deployment Checklist

## Database

- Installed
- Started
- Enabled
- Root Password Set
- Schema Created
- Tables Created
- Users Created

---

## Backend

- Runtime Installed
- Service User Created
- Code Downloaded
- Dependencies Installed
- Database Connectivity Configured
- Systemd Service Created
- Started Successfully

---

## Frontend

- Nginx Installed
- Started
- Enabled
- Frontend Code Deployed
- Reverse Proxy Configured
- Backend Connectivity Verified

---

# Troubleshooting

## Database Issues

Verify:

```bash
systemctl status mysqld
```

Login:

```bash
mysql -u root -p
```

---

## Backend Issues

Check:

```bash
systemctl status backend
```

Verify:

```bash
journalctl -u backend
```

Verify:

```bash
telnet DB_IP 3306
```

---

## Frontend Issues

Check:

```bash
systemctl status nginx
```

Verify:

```bash
nginx -t
```

Check:

```bash
telnet backend_IP 8080
```

---

# Frequently Asked Interview Questions

## What are the three layers of an application?

- Frontend
- Backend
- Database

---

## Why do backend applications require database connectivity?

To perform CRUD operations and store application data.

---

## Why is systemctl daemon-reload required?

To reload systemd service definitions after creating or modifying service files.

---

## Where does Nginx serve frontend content from?

```text
/usr/share/nginx/html
```

---

## Where are custom service files stored?

```text
/etc/systemd/system/
```

---

## Why test connectivity using telnet?

To verify whether a remote service is reachable on a specific port.

---

## What is the purpose of a reverse proxy?

To forward requests from frontend to backend while hiding backend servers from users.

---

# Key Takeaways

- Modern applications typically use a three-tier architecture.
- Databases store application data.
- Backend applications contain business logic.
- Frontend applications provide the user interface.
- Backend applications require database connectivity.
- Service users improve security.
- Systemd manages application services.
- Nginx hosts frontend applications and acts as a reverse proxy.
- Connectivity testing is a critical troubleshooting skill.
- Successful deployment requires proper configuration of database, backend, and frontend layers.