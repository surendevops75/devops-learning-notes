# MongoDB and NoSQL Fundamentals

## Introduction

Traditional databases such as MySQL store data in rows and columns.

Modern applications often deal with:

- Product Catalogs
- User Profiles
- Reviews
- Recommendations
- Search Results
- Dynamic Data Structures

For these use cases, NoSQL databases are commonly used.

MongoDB is one of the most popular NoSQL databases.

---

# What is NoSQL?

NoSQL stands for:

```text
Not Only SQL
```

Unlike relational databases, NoSQL databases do not require a fixed table structure.

They provide:

- Flexible Schema
- Faster Development
- Horizontal Scalability
- High Performance for Certain Workloads

---

# SQL vs NoSQL

## SQL Database

Examples:

- MySQL
- PostgreSQL
- Oracle
- MSSQL

Structure:

```text
Database
   │
   ▼
Tables
   │
   ▼
Rows and Columns
```

Example:

| ProductID | Name | Price |
|------------|--------|---------|
| 1 | Laptop | 50000 |

Schema must be predefined.

---

## NoSQL Database

Examples:

- MongoDB
- Cassandra
- DynamoDB
- CouchDB

Structure:

```text
Database
   │
   ▼
Collections
   │
   ▼
Documents
```

Schema is flexible.

---

# Why NoSQL for E-Commerce?

Product data can vary significantly.

Example:

Product 1:

```json
{
  "name": "Some Food",
  "price": "123 INR",
  "description": "Food products daily consumable",
  "expiry": "01-01-2026"
}
```

Product 2:

```json
{
  "name": "A Toy",
  "price": "123 INR",
  "description": "Kids toy",
  "subscription": "Yes, Monthly"
}
```

Notice:

```text
Different Fields
Different Structures
```

No schema modification is required.

---

# What is MongoDB?

MongoDB is a document-oriented NoSQL database.

Instead of storing data in rows and columns, it stores data as documents.

Structure:

```text
MongoDB
   │
   ▼
Database
   │
   ▼
Collection
   │
   ▼
Documents
```

---

# MongoDB Terminology

| Relational Database | MongoDB |
|---------------------|----------|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |

---

# Document Example

```json
{
  "name": "Artificial Intelligence Book",
  "price": "499 INR",
  "rating": 4.8,
  "reviews": 1200
}
```

Each document contains:

```text
Key → Value Pairs
```

---

# Collections

A collection is similar to a table in MySQL.

Example:

```text
catalogue
```

Collection may contain:

```json
{
  "name": "Laptop"
}
```

```json
{
  "name": "Mobile",
  "discount": "10%"
}
```

Different documents can have different fields.

---

# BSON

Internally MongoDB stores data as:

```text
BSON
```

BSON stands for:

```text
Binary JSON
```

Advantages:

- Faster Processing
- Efficient Storage
- Supports Additional Data Types

---

# MongoDB Architecture

```text
Application
     │
     ▼
MongoDB Server
     │
     ▼
Database
     │
     ▼
Collection
     │
     ▼
Documents
```

---

# RoboShop and MongoDB

In RoboShop:

```text
Catalogue Service
```

uses MongoDB to store product information.

Example:

```json
{
  "name": "Artificial Intelligence",
  "price": "999 INR",
  "description": "AI Learning Course",
  "rating": 4.9
}
```

---

# MongoDB Default Port

MongoDB listens on:

```text
27017
```

Verify:

```bash
ss -lntp | grep 27017
```

or

```bash
netstat -lntp | grep 27017
```

---

# MongoDB Repository Configuration

Repository files are stored in:

```text
/etc/yum.repos.d/
```

Example:

```text
mongodb-org.repo
```

Used by:

```bash
dnf
```

to install MongoDB packages.

---

# MongoDB Binding

MongoDB can listen on specific IP addresses.

This is controlled by:

```text
bindIp
```

configuration.

---

# Localhost Binding

Example:

```text
127.0.0.1:27017
```

Meaning:

```text
Only Local Machine Can Connect
```

Architecture:

```text
Application
     │
     ▼
MongoDB
```

Remote servers cannot access MongoDB.

---

# Open Binding

Example:

```text
0.0.0.0:27017
```

Meaning:

```text
Listen On All Network Interfaces
```

Architecture:

```text
Server-1
      │
      ▼
MongoDB Server
      ▲
      │
Server-2
```

Remote connections are allowed.

---

# Security Considerations

Opening MongoDB to:

```text
0.0.0.0
```

requires:

- Firewall Rules
- Security Groups
- Authentication
- Network Restrictions

Otherwise MongoDB may be exposed publicly.

---

# Verify MongoDB Connectivity

From another server:

```bash
telnet MONGODB_IP 27017
```

Example:

```bash
telnet 172.31.20.10 27017
```

Successful connection means:

```text
Network Connectivity Exists
```

---

# MongoDB Service Management

Start MongoDB:

```bash
systemctl start mongod
```

---

Enable MongoDB:

```bash
systemctl enable mongod
```

---

Check Status:

```bash
systemctl status mongod
```

---

Restart MongoDB:

```bash
systemctl restart mongod
```

---

# MongoDB Process Verification

```bash
ps -ef | grep mongod
```

---

# MongoDB Logs

View logs:

```bash
journalctl -u mongod
```

Follow logs:

```bash
journalctl -fu mongod
```

---

# Advantages of MongoDB

## Flexible Schema

Different documents can contain different fields.

---

## Fast Development

Developers can add fields without modifying table structures.

---

## Horizontal Scalability

Supports scaling across multiple servers.

---

## JSON-like Documents

Easy integration with modern applications.

---

# When to Use MongoDB?

Good For:

- Product Catalogs
- Content Management Systems
- User Profiles
- Event Data
- IoT Data
- Real-Time Applications

---

# When SQL Databases are Better?

Good For:

- Banking Applications
- Financial Transactions
- ERP Systems
- Strict Data Relationships

---

# SQL vs MongoDB Comparison

| SQL | MongoDB |
|------|----------|
| Tables | Collections |
| Rows | Documents |
| Fixed Schema | Flexible Schema |
| Joins | Embedded Documents |
| Relational Data | Document Data |

---

# Common MongoDB Troubleshooting

## MongoDB Not Starting

Check:

```bash
systemctl status mongod
```

---

Check logs:

```bash
journalctl -u mongod
```

---

## Connection Refused

Verify:

```bash
ss -lntp | grep 27017
```

---

Check:

```text
bindIp Configuration
```

---

Check:

```text
Security Groups
Firewall Rules
```

---

## Catalogue Service Cannot Connect

Verify:

```bash
telnet MONGODB_IP 27017
```

---

Verify:

```text
MONGO_URL
```

configuration.

---

# Frequently Asked Interview Questions

## What is NoSQL?

A database model that does not require a fixed relational schema.

---

## What is MongoDB?

A document-oriented NoSQL database.

---

## What is the Default MongoDB Port?

```text
27017
```

---

## What is a Document?

A JSON-like data structure containing key-value pairs.

---

## What is a Collection?

A group of related documents.

---

## Difference Between Table and Collection?

| SQL | MongoDB |
|------|----------|
| Table | Collection |
| Row | Document |

---

## What is BSON?

Binary JSON used internally by MongoDB.

---

## Difference Between 127.0.0.1 and 0.0.0.0?

```text
127.0.0.1 → Local Access Only

0.0.0.0 → Remote Access Allowed
```

---

## Why is MongoDB Popular for E-Commerce?

Because product data can have varying structures and MongoDB supports flexible schemas.

---

# Key Takeaways

- MongoDB is a document-oriented NoSQL database.
- Data is stored as documents inside collections.
- Documents use key-value pairs.
- MongoDB uses BSON internally.
- Default port is 27017.
- 127.0.0.1 allows only local access.
- 0.0.0.0 allows remote access.
- MongoDB is commonly used for product catalogs and dynamic data.
- Flexible schema is one of MongoDB's biggest advantages.
- Understanding MongoDB is important for modern DevOps and Cloud environments.