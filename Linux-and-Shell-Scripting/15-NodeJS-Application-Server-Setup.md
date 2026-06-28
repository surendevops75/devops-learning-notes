# NodeJS Application Server Setup

## Introduction

In a 3-Tier Architecture:

```text
User
  │
  ▼
Load Balancer
  │
  ▼
Web Server
  │
  ▼
Application Server
  │
  ▼
Database Server
```

The Application Server contains the business logic of the application.

Responsibilities include:

- Processing user requests
- Performing CRUD operations
- Connecting to databases
- Authentication and Authorization
- Business Logic Execution
- API Processing

Popular application technologies:

- Java
- NodeJS
- Python
- .NET
- Golang
- PHP
- Ruby

---

# What is NodeJS?

NodeJS is an open-source JavaScript runtime environment.

It allows developers to run JavaScript outside the browser.

Applications built with NodeJS are commonly used for:

- REST APIs
- Microservices
- Web Applications
- Real-Time Applications
- Backend Services

---

# Why NodeJS?

Advantages:

- Fast Performance
- Event Driven
- Non-Blocking I/O
- Lightweight
- Large Ecosystem
- Easy API Development

---

# Application Server Workflow

```text
User Request
      │
      ▼
Load Balancer
      │
      ▼
Web Server (Nginx)
      │
      ▼
NodeJS Application
      │
      ▼
MySQL Database
```

---

# DNF Package Manager

RHEL-based operating systems use:

```bash
dnf
```

DNF is the successor of:

```bash
yum
```

Both are package managers used to:

- Install Packages
- Remove Packages
- Update Packages
- Search Packages

---

# NodeJS Module Management

Many RHEL-based distributions provide multiple NodeJS versions.

Before installing NodeJS, manage the required module stream.

---

## View Available NodeJS Versions

```bash
dnf module list nodejs
```

Example Output:

```text
nodejs 18
nodejs 20
nodejs 22
```

---

## Disable Default NodeJS Version

```bash
dnf module disable nodejs -y
```

Purpose:

- Removes default module stream
- Prevents version conflicts

---

## Enable NodeJS 20

```bash
dnf module enable nodejs:20 -y
```

Purpose:

- Enables NodeJS Version 20 repository

---

## Install NodeJS

```bash
dnf install nodejs -y
```

---

# Verify Installation

## Check NodeJS Version

```bash
node --version
```

Example:

```bash
node -v
```

Output:

```text
v20.x.x
```

---

## Check NPM Version

```bash
npm --version
```

Example:

```bash
npm -v
```

---

# What is NPM?

NPM stands for:

```text
Node Package Manager
```

Used for:

- Installing Dependencies
- Managing Packages
- Running Scripts
- Building Applications

---

# Common NPM Commands

## Check Version

```bash
npm -v
```

---

## Install Dependencies

```bash
npm install
```

---

## Install Specific Package

```bash
npm install express
```

---

## Install Global Package

```bash
npm install -g pm2
```

---

## List Installed Packages

```bash
npm list
```

---

# Service Accounts

Applications should not run using root user.

Instead, create dedicated service accounts.

Example:

```bash
useradd expense
```

Purpose:

- Improved Security
- Better Access Control
- Principle of Least Privilege

---

# Verify User

```bash
id expense
```

Example Output:

```text
uid=1001(expense)
gid=1001(expense)
```

---

# Application Directory Structure

Create application directory:

```bash
mkdir /app
```

Verify:

```bash
ls -ld /app
```

---

# Common Application Structure

```text
/app
│
├── server.js
├── package.json
├── package-lock.json
├── node_modules
├── config
├── routes
├── controllers
├── services
└── logs
```

---

# What is package.json?

The main configuration file of a NodeJS application.

Contains:

- Application Name
- Version
- Dependencies
- Scripts
- Metadata

Example:

```json
{
  "name": "expense-backend",
  "version": "1.0.0"
}
```

---

# Application Deployment Workflow

```text
Install OS
     │
     ▼
Install NodeJS
     │
     ▼
Create Service Account
     │
     ▼
Create Application Directory
     │
     ▼
Copy Application Code
     │
     ▼
Install Dependencies
     │
     ▼
Start Application
```

---

# Copy Application Code

Example:

```bash
cp backend.zip /app
```

or

```bash
git clone <repository-url>
```

---

# Extract Application

```bash
unzip backend.zip
```

---

# Install Dependencies

Navigate to application directory:

```bash
cd /app
```

Install packages:

```bash
npm install
```

---

# Start Application

```bash
node server.js
```

or

```bash
npm start
```

---

# Run Application in Background

Using:

```bash
nohup node server.js &
```

---

# Process Verification

Check application process:

```bash
ps -ef | grep node
```

---

# Check Listening Port

```bash
ss -lntp
```

or

```bash
netstat -lntp
```

---

# Application Logs

View logs:

```bash
cat application.log
```

---

## Monitor Logs

```bash
tail -f application.log
```

---

# Common Troubleshooting

## Application Not Starting

Check:

```bash
node -v
```

---

Check:

```bash
npm install
```

completed successfully.

---

Check:

```bash
ps -ef | grep node
```

---

Check:

```bash
ss -lntp
```

---

Check application logs.

---

# Permissions Issue

Verify ownership:

```bash
ls -ld /app
```

---

Change ownership:

```bash
chown -R expense:expense /app
```

---

# Environment Variables

Applications often use environment variables.

Example:

```bash
export DB_HOST=mysql.example.com
```

Verify:

```bash
echo $DB_HOST
```

---

# Production Best Practices

## Do Not Run as Root

Use:

```bash
useradd expense
```

instead of running applications as root.

---

## Use Dedicated Directories

```text
/app
```

for application code.

---

## Restrict Permissions

```bash
chmod
chown
```

appropriately.

---

## Monitor Application

Use:

```bash
top
ps
journalctl
```

---

## Maintain Logs

Store and rotate application logs.

---

# Common DevOps Commands

## Verify NodeJS

```bash
node -v
```

---

## Verify NPM

```bash
npm -v
```

---

## View Running Processes

```bash
ps -ef | grep node
```

---

## Check Open Ports

```bash
ss -lntp
```

---

## View Resource Usage

```bash
top
```

---

# Frequently Asked Interview Questions

## What is NodeJS?

A JavaScript runtime environment used to build backend applications.

---

## Why Use NodeJS?

- Fast
- Event Driven
- Non-Blocking I/O
- Lightweight

---

## What is NPM?

Node Package Manager used to install and manage dependencies.

---

## What is package.json?

Configuration file containing application metadata and dependencies.

---

## Why Create a Service Account?

To avoid running applications as root and improve security.

Example:

```bash
useradd expense
```

---

## Why Create /app Directory?

To maintain application code in a standard location.

---

## Difference Between yum and dnf

| yum | dnf |
|------|------|
| Older Package Manager | Modern Package Manager |
| Legacy Systems | Current Systems |
| Slower Dependency Resolution | Faster Dependency Resolution |

---

## How Do You Verify a NodeJS Application is Running?

```bash
ps -ef | grep node
```

---

## How Do You Verify Application Port?

```bash
ss -lntp
```

or

```bash
netstat -lntp
```

---

# Key Takeaways

- NodeJS is a backend JavaScript runtime.
- Application servers handle business logic.
- DNF is the package manager for RHEL-based systems.
- NodeJS modules can be managed using DNF module streams.
- NodeJS 20 can be enabled using DNF modules.
- NPM manages NodeJS dependencies.
- Applications should run using service accounts instead of root.
- `/app` is commonly used to store application code.
- Application deployment involves installation, configuration, dependency installation, and startup.
- Monitoring processes, logs, and ports is essential for troubleshooting.
- NodeJS is widely used for APIs and microservices in modern cloud environments.