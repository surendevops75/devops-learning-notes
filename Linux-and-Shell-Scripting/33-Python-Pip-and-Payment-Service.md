# Python, Pip and Payment Service

## Introduction

In RoboShop architecture, the Payment Service is responsible for handling payment-related operations.

Examples:

- Payment Validation
- Payment Processing
- Payment Authorization
- Transaction Verification
- Payment Status Updates

The Payment Service in RoboShop is developed using Python.

---

# Python Overview

Python is one of the most popular programming languages.

Commonly used for:

- Web Applications
- Automation
- DevOps
- Artificial Intelligence
- Machine Learning
- Data Engineering
- API Development

---

# Python File Extension

Python source files use:

```text
.py
```

Examples:

```text
payment.py
main.py
app.py
```

---

# Python Source Code

Example:

```python
print("Hello World")
```

Python code is written by developers and stored in:

```text
.py files
```

---

# Python Execution

Unlike Java:

```text
Java
 ├── Compile
 └── Execute
```

Python generally follows:

```text
Source Code
      │
      ▼
Python Interpreter
      │
      ▼
Execution
```

Example:

```bash
python payment.py
```

or

```bash
python3 payment.py
```

---

# Build Tools

Different technologies use different build tools.

| Technology | Build Tool |
|------------|------------|
| NodeJS | npm |
| Java | Maven |
| Python | pip |
| Golang | go modules |

---

# What is pip?

pip stands for:

```text
Preferred Installer Program
```

It is the package manager for Python.

Used for:

- Installing Dependencies
- Managing Libraries
- Upgrading Packages
- Removing Packages

---

# Dependencies

Applications often require external libraries.

Examples:

- Database Drivers
- HTTP Libraries
- Authentication Libraries
- Payment SDKs

These external libraries are called:

```text
Dependencies
```

---

# Build File

Python dependency information is usually stored in:

```text
requirements.txt
```

---

# Example requirements.txt

```text
flask
requests
pymysql
redis
```

Each line represents a dependency.

---

# Installing Dependencies

Install all dependencies:

```bash
pip install -r requirements.txt
```

Meaning:

```text
Read requirements.txt
Download Dependencies
Install Dependencies
```

---

# Individual Package Installation

Example:

```bash
pip install flask
```

Install:

```text
Flask Framework
```

---

# Upgrade Package

```bash
pip install --upgrade flask
```

---

# Remove Package

```bash
pip uninstall flask
```

---

# View Installed Packages

```bash
pip list
```

---

# Generate requirements.txt

If packages are already installed:

```bash
pip freeze > requirements.txt
```

Creates:

```text
requirements.txt
```

containing all installed packages.

---

# Payment Service Architecture

```text
User
 │
 ▼
Frontend
 │
 ▼
Payment Service
 │
 ▼
Payment Gateway
```

---

# Payment Service Responsibilities

- Validate Payment Requests
- Process Transactions
- Verify Payment Status
- Generate Responses

---

# API Concept

Modern applications communicate using APIs.

API stands for:

```text
Application Programming Interface
```

---

# Real World Example

Consider:

```text
10th Class Results Website
```

You enter:

```text
Hall Ticket Number
```

and click:

```text
Submit
```

---

# API Flow

```text
Browser
   │
   ▼
Government Results API
   │
   ▼
Database
   │
   ▼
Result
```

---

# Request and Response

Request:

```json
{
  "hallticket": "123456789"
}
```

---

Response:

```json
{
  "name": "Ramesh",
  "marks": 580,
  "grade": "A"
}
```

---

# Payment API Example

Request:

```json
{
  "amount": 500,
  "card": "xxxx-xxxx-xxxx-1234"
}
```

---

Response:

```json
{
  "status": "SUCCESS",
  "transactionId": "TXN12345"
}
```

---

# Common Python Deployment Flow

## Install Python

```bash
dnf install python3 -y
```

---

## Create Application User

```bash
useradd --system payment
```

---

## Create Application Directory

```bash
mkdir /app
```

---

## Download Application Code

```bash
curl -o /tmp/payment.zip <URL>
```

---

## Extract Code

```bash
unzip /tmp/payment.zip -d /app
```

---

## Install Dependencies

```bash
cd /app

pip install -r requirements.txt
```

---

## Create Service

```text
/etc/systemd/system/payment.service
```

---

## Reload Systemd

```bash
systemctl daemon-reload
```

---

## Start Service

```bash
systemctl start payment
```

---

## Enable Service

```bash
systemctl enable payment
```

---

## Check Status

```bash
systemctl status payment
```

---

# Troubleshooting

## Service Not Starting

Check:

```bash
systemctl status payment
```

---

## Check Logs

```bash
journalctl -u payment
```

---

## Verify Python

```bash
python3 --version
```

---

## Verify Dependencies

```bash
pip list
```

---

## Verify Port

```bash
ss -lntp
```

or

```bash
netstat -lntp
```

---

## Verify API

```bash
curl http://localhost:8080/health
```

---

# Common DevOps Tasks

- Install Python Runtime
- Install Dependencies
- Manage Services
- Configure Environment Variables
- Monitor Logs
- Restart Applications
- Validate APIs

---

# Frequently Asked Interview Questions

## What is Python?

A high-level programming language used for automation, web applications, AI, and DevOps.

---

## What is pip?

Python package manager used to install and manage dependencies.

---

## What is requirements.txt?

A file containing dependency information for a Python application.

---

## Which Command Installs Dependencies?

```bash
pip install -r requirements.txt
```

---

## Which Command Lists Installed Packages?

```bash
pip list
```

---

## What is an API?

Application Programming Interface used for communication between systems.

---

## What is a Request?

Data sent to an API.

---

## What is a Response?

Data returned by an API.

---

## What Does Payment Service Do?

Processes and validates payment transactions.

---

# Key Takeaways

- Payment Service in RoboShop is developed using Python.
- Python source files use the `.py` extension.
- pip is the package manager for Python.
- Dependencies are stored in `requirements.txt`.
- APIs allow systems to communicate using requests and responses.
- Payment services interact with external payment systems.
- DevOps engineers commonly install runtimes, dependencies, and manage services.
- Understanding Python deployment is important for application support and troubleshooting.