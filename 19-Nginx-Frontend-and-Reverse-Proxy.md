# Nginx Frontend and Reverse Proxy

## Introduction

Nginx is one of the most popular web servers used in modern applications.

It is commonly used as:

- Web Server
- Reverse Proxy
- Load Balancer
- SSL/TLS Termination Server
- Static Content Server

Nginx is lightweight, fast, and widely used in production environments.

---

# What is Nginx?

Nginx is an open-source web server and reverse proxy server.

Common Uses:

- Hosting Frontend Applications
- Routing Requests
- Reverse Proxy
- Load Balancing
- SSL Offloading

---

# Nginx in 3-Tier Architecture

```text
User
  │
  ▼
Load Balancer
  │
  ▼
Nginx (Frontend/Web Server)
  │
  ▼
Backend Application
  │
  ▼
Database
```

---

# Frontend Application

Frontend applications contain:

- HTML
- CSS
- JavaScript

Popular Frontend Frameworks:

- React
- Angular
- Vue
- Next.js

Frontend Responsibilities:

- User Interface
- Forms
- Navigation
- User Interaction

---

# Nginx Installation

Install Nginx:

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

# Verify Nginx Status

```bash
systemctl status nginx
```

Expected Output:

```text
active (running)
```

---

# Verify Listening Port

```bash
ss -lntp | grep nginx
```

or

```bash
netstat -lntp
```

Expected Port:

```text
80
```

---

# Default Nginx Directory Structure

## Frontend Content Directory

```text
/usr/share/nginx/html
```

This is the default location where frontend code is placed.

Example:

```text
index.html
css/
js/
images/
```

---

## Nginx Configuration Directory

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

# Important Nginx Files

## Main Configuration File

```text
/etc/nginx/nginx.conf
```

Controls global Nginx behavior.

---

## Additional Configuration Files

```text
/etc/nginx/conf.d/
```

Used for custom virtual hosts.

---

## Default Configuration Files

```text
/etc/nginx/default.d/
```

Used for default site configurations.

---

# Frontend Deployment

Create application directory if required:

```bash
mkdir -p /usr/share/nginx/html
```

---

# Download Frontend Code

Example:

```bash
curl -o /tmp/frontend.zip <frontend-url>
```

---

# Extract Frontend Files

```bash
unzip frontend.zip -d /usr/share/nginx/html
```

---

# Verify Files

```bash
ls -l /usr/share/nginx/html
```

Expected:

```text
index.html
assets/
css/
js/
```

---

# Access Frontend

```text
http://SERVER-IP
```

Example:

```text
http://54.88.122.243
```

---

# What is a Reverse Proxy?

A Reverse Proxy sits between users and backend servers.

Architecture:

```text
User
  │
  ▼
Nginx
  │
  ▼
Backend Server
```

Users interact only with Nginx.

Backend servers remain hidden.

---

# Benefits of Reverse Proxy

- Security
- Load Balancing
- SSL Termination
- Request Routing
- Performance Improvement

---

# Frontend and Backend Communication

Frontend applications usually call backend APIs.

Example:

```text
Frontend
     │
     ▼
/api/transaction
     │
     ▼
Backend
```

Example URL:

```text
http://54.88.122.243/api/transaction
```

---

# Why Reverse Proxy is Required

Without Reverse Proxy:

```text
Frontend → Backend-IP:8080
```

Problems:

- Backend exposed to users
- Security concerns
- Difficult management

---

With Reverse Proxy:

```text
Frontend
    │
    ▼
Nginx
    │
    ▼
Backend:8080
```

Backend remains private.

---

# Nginx Reverse Proxy Configuration

Create configuration file:

```bash
vim /etc/nginx/default.d/expense.conf
```

---

# Example Configuration

```nginx
location /api/ {
    proxy_pass http://localhost:8080/;
}
```

Explanation:

```text
/api/*
      │
      ▼
Forward Request
      │
      ▼
Backend Port 8080
```

---

# Complete Example

```nginx
location /api/ {
    proxy_pass http://localhost:8080/;
    proxy_http_version 1.1;
}
```

---

# Verify Nginx Configuration

Before restarting:

```bash
nginx -t
```

Expected Output:

```text
syntax is ok
test is successful
```

---

# Restart Nginx

```bash
systemctl restart nginx
```

---

# Reload Nginx

```bash
systemctl reload nginx
```

Difference:

```text
restart -> stop and start

reload -> apply configuration without stopping service
```

---

# Verify API Routing

Frontend Request:

```text
http://54.88.122.243/api/transaction
```

Flow:

```text
Browser
   │
   ▼
Nginx
   │
   ▼
Backend Application
   │
   ▼
Database
```

---

# Nginx Logs

## Access Logs

```text
/var/log/nginx/access.log
```

---

## Error Logs

```text
/var/log/nginx/error.log
```

---

# View Access Logs

```bash
tail -f /var/log/nginx/access.log
```

---

# View Error Logs

```bash
tail -f /var/log/nginx/error.log
```

---

# Common Nginx Commands

## Start

```bash
systemctl start nginx
```

---

## Stop

```bash
systemctl stop nginx
```

---

## Restart

```bash
systemctl restart nginx
```

---

## Reload

```bash
systemctl reload nginx
```

---

## Enable

```bash
systemctl enable nginx
```

---

## Disable

```bash
systemctl disable nginx
```

---

## Status

```bash
systemctl status nginx
```

---

# Troubleshooting Nginx

## Nginx Not Starting

Check:

```bash
systemctl status nginx
```

---

Check configuration:

```bash
nginx -t
```

---

Check logs:

```bash
tail -f /var/log/nginx/error.log
```

---

# Website Not Accessible

Verify:

```bash
ss -lntp
```

Check:

```text
Port 80 Listening?
```

---

Verify:

```bash
curl http://localhost
```

---

Verify Security Groups:

```text
Port 80 Open?
```

---

# API Not Working

Check backend:

```bash
systemctl status backend
```

---

Verify backend port:

```bash
ss -lntp | grep 8080
```

---

Verify reverse proxy:

```bash
nginx -t
```

---

Check logs:

```bash
tail -f /var/log/nginx/error.log
```

---

# Common Production Architecture

```text
Internet
    │
    ▼
Load Balancer
    │
    ▼
Nginx
    │
    ▼
Backend (8080)
    │
    ▼
MySQL (3306)
```

---

# Frequently Asked Interview Questions

## What is Nginx?

A web server, reverse proxy, and load balancer.

---

## What is a Reverse Proxy?

A server that accepts requests from clients and forwards them to backend servers.

---

## What is the Default Nginx Web Root?

```text
/usr/share/nginx/html
```

---

## Where are Nginx Configuration Files Stored?

```text
/ etc/nginx
```

Important files:

```text
/etc/nginx/nginx.conf
/etc/nginx/conf.d/
/etc/nginx/default.d/
```

---

## How Do You Test Nginx Configuration?

```bash
nginx -t
```

---

## Difference Between Reload and Restart?

| Reload | Restart |
|----------|----------|
| No Downtime | Brief Downtime |
| Re-read Config | Stop and Start Service |

---

## How Do You Verify Nginx is Running?

```bash
systemctl status nginx
```

---

## Where are Nginx Logs Stored?

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

---

## Why Use Reverse Proxy?

- Security
- Scalability
- SSL Termination
- Load Balancing
- Simplified Routing

---

# Key Takeaways

- Nginx is a web server and reverse proxy.
- Frontend applications are usually hosted in `/usr/share/nginx/html`.
- Nginx configuration files are stored in `/etc/nginx`.
- Reverse proxies hide backend servers from users.
- Backend applications commonly run on port 8080.
- Nginx can route API requests to backend services.
- `nginx -t` validates configuration syntax.
- Access and error logs are essential for troubleshooting.
- Nginx is one of the most important technologies in DevOps and production environments.