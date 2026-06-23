# Catalogue Service and Redis Caching

## Introduction

In RoboShop architecture, the Catalogue Service is responsible for managing product information.

Examples:

- Product Name
- Product Description
- Price
- Reviews
- Ratings
- Vendor Information
- Discounts
- Product Categories

The Catalogue Service stores and retrieves product information from MongoDB.

To improve performance and reduce database load, Redis is commonly used as a caching layer.

---

# Catalogue Service Overview

Catalogue Service is a backend microservice.

Responsibilities:

- Product Search
- Product Listing
- Product Details
- Product Filtering
- Product Recommendations

Example Request:

```text
http://daws86s.fun/api/catalogue/products/Artificial%20Intelligence
```

Flow:

```text
User
 │
 ▼
Frontend
 │
 ▼
Catalogue Service
 │
 ▼
MongoDB
```

---

# Catalogue Service Architecture

```text
User
 │
 ▼
Frontend (Nginx)
 │
 ▼
Catalogue Service
 │
 ▼
MongoDB
```

---

# Creating a System User

Applications should not run as root.

Instead, create a dedicated service account.

Example:

```bash
useradd --system \
        --home /app \
        --shell /sbin/nologin \
        --comment "roboshop system user" \
        roboshop
```

---

# Understanding User Creation Options

## Create System User

```bash
--system
```

Creates a service account.

---

## Home Directory

```bash
--home /app
```

Sets:

```text
/app
```

as home directory.

---

## Disable Login

```bash
--shell /sbin/nologin
```

Prevents interactive login.

---

## Comment

```bash
--comment
```

Adds description.

---

# Why Use System Users?

Advantages:

- Improved Security
- Least Privilege Access
- Service Isolation
- Better Auditing

Examples:

```text
nginx
mysql
redis
roboshop
```

---

# Application Deployment Flow

```text
Create User
      │
      ▼
Create Directory
      │
      ▼
Download Application
      │
      ▼
Install Dependencies
      │
      ▼
Create Service
      │
      ▼
Start Application
```

---

# Systemd Service File

Location:

```text
/etc/systemd/system/catalogue.service
```

---

# Catalogue Service Configuration

```ini
[Unit]
Description=Catalogue Service

[Service]
User=roboshop
Environment=MONGO=true
Environment=MONGO_URL="mongodb://<MONGODB-SERVER-IPADDRESS>:27017/catalogue"
ExecStart=/bin/node /app/server.js
SyslogIdentifier=catalogue

[Install]
WantedBy=multi-user.target
```

---

# Understanding the Service File

## Description

```ini
Description=Catalogue Service
```

Provides service description.

---

## User

```ini
User=roboshop
```

Runs application using:

```text
roboshop
```

user.

---

## Environment Variables

```ini
Environment=MONGO=true
```

Enables MongoDB integration.

---

MongoDB Connection:

```ini
Environment=MONGO_URL="mongodb://<MONGODB-SERVER-IPADDRESS>:27017/catalogue"
```

---

# MongoDB Connection String

Format:

```text
mongodb://HOST:PORT/DATABASE
```

Example:

```text
mongodb://172.31.10.20:27017/catalogue
```

Components:

```text
Host     : 172.31.10.20
Port     : 27017
Database : catalogue
```

---

# Application Startup

```ini
ExecStart=/bin/node /app/server.js
```

Meaning:

```text
Start NodeJS Application
```

using:

```text
server.js
```

---

# Syslog Identifier

```ini
SyslogIdentifier=catalogue
```

Used for log identification.

---

# Service Boot Configuration

```ini
WantedBy=multi-user.target
```

Allows service startup during system boot.

---

# Reload Systemd

After creating or modifying service files:

```bash
systemctl daemon-reload
```

---

# Start Catalogue Service

```bash
systemctl start catalogue
```

---

# Enable Catalogue Service

```bash
systemctl enable catalogue
```

---

# Check Service Status

```bash
systemctl status catalogue
```

---

# Restart Service

```bash
systemctl restart catalogue
```

---

# Stop Service

```bash
systemctl stop catalogue
```

---

# Verify Process

```bash
ps -ef | grep catalogue
```

or

```bash
ps -ef | grep node
```

---

# Verify Port

```bash
ss -lntp
```

or

```bash
netstat -lntp
```

---

# MongoDB Connectivity Testing

Verify Catalogue Service can reach MongoDB.

```bash
telnet MONGODB_IP 27017
```

Example:

```bash
telnet 172.31.10.20 27017
```

---

# Redis Introduction

Redis is one of the most popular caching technologies.

Redis stands for:

```text
Remote Dictionary Server
```

Redis is:

```text
In-Memory Database
```

---

# Why Redis?

Accessing data from a database is slower than accessing data from memory.

Flow:

```text
Disk
 │
 ▼
RAM
 │
 ▼
Application
```

Reading from RAM is significantly faster.

---

# Real World Example

Example:

```text
Chief Minister (CM)
```

Instead of meeting everyone directly:

```text
Person
   │
   ▼
PA
   │
   ▼
CM
```

The PA acts as an intermediary.

Redis works similarly.

---

# Without Redis

```text
Application
      │
      ▼
Database
```

Every request reaches the database.

Problems:

- Higher Latency
- Increased Database Load
- Reduced Performance

---

# With Redis

```text
Application
      │
      ▼
Redis
      │
      ▼
Database
```

Redis stores frequently used data.

---

# Cache Hit

Scenario:

Data already exists in Redis.

```text
Application
      │
      ▼
Redis
```

Response returned immediately.

This is called:

```text
Cache Hit
```

---

# Cache Miss

Scenario:

Data not available in Redis.

Flow:

```text
Application
      │
      ▼
Redis
      │
      ▼
Database
```

Database returns data.

Redis stores it for future requests.

This is called:

```text
Cache Miss
```

---

# Cache Population

Flow:

```text
Application
      │
      ▼
Redis
      │
      ▼
No Data Found
      │
      ▼
Database
      │
      ▼
Return Data
      │
      ▼
Store in Redis
      │
      ▼
Return to User
```

---

# Performance Benefits

Redis provides:

- Faster Responses
- Reduced Database Load
- Better User Experience
- Improved Scalability

---

# Common Redis Use Cases

## Product Catalogs

Frequently searched products.

---

## User Sessions

Store logged-in user sessions.

---

## API Response Caching

Store API responses.

---

## Shopping Cart

Store temporary cart information.

---

## Leaderboards

Gaming applications.

---

# Redis Default Port

Redis listens on:

```text
6379
```

Verify:

```bash
ss -lntp | grep 6379
```

---

# Redis Service Management

Start Redis:

```bash
systemctl start redis
```

---

Enable Redis:

```bash
systemctl enable redis
```

---

Check Status:

```bash
systemctl status redis
```

---

Restart Redis:

```bash
systemctl restart redis
```

---

# Common Troubleshooting

## Catalogue Service Not Starting

Check:

```bash
systemctl status catalogue
```

---

View Logs:

```bash
journalctl -u catalogue
```

---

Verify MongoDB Connectivity:

```bash
telnet MONGODB_IP 27017
```

---

Verify Environment Variables:

```text
MONGO_URL
```

---

## Redis Not Working

Check:

```bash
systemctl status redis
```

---

Verify Port:

```bash
ss -lntp | grep 6379
```

---

Check Logs:

```bash
journalctl -u redis
```

---

# Frequently Asked Interview Questions

## What is the Catalogue Service?

A microservice responsible for managing product information.

---

## Why Create a System User?

For security and service isolation.

---

## What is MONGO_URL?

Connection string used to connect to MongoDB.

---

## Why Use Redis?

To improve application performance by caching frequently accessed data.

---

## What is a Cache Hit?

Data found in cache.

---

## What is a Cache Miss?

Data not found in cache and fetched from database.

---

## What is the Default MongoDB Port?

```text
27017
```

---

## What is the Default Redis Port?

```text
6379
```

---

## Why is Redis Faster Than Databases?

Because Redis stores data in memory (RAM).

---

# Key Takeaways

- Catalogue Service manages product data in RoboShop.
- Applications should run using dedicated system users.
- Systemd services manage application lifecycle.
- MongoDB stores product information.
- Redis is an in-memory database used for caching.
- Cache hits improve performance.
- Cache misses fetch data from databases.
- Redis reduces database load and improves response times.
- Redis is commonly used in modern microservices architectures.