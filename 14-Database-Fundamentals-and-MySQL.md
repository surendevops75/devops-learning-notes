# Database Fundamentals and MySQL

## Introduction

Applications need a place to store and retrieve data.

Examples:

- User Information
- Product Catalog
- Orders
- Payments
- Employee Records
- Inventory Data

A Database is used to store, organize, manage, and retrieve this information efficiently.

---

# What is a Database?

A Database is an organized collection of data.

Examples:

```text
Amazon Product Data
Facebook User Data
Bank Customer Data
Hospital Records
```

Without databases, applications cannot store persistent information.

---

# Database Server

A Database Server is a machine where database software is installed and running.

```text
Server
=
Machine
=
Node
=
Host
```

Example:

```text
Database Server
```

contains:

```text
Operating System
+
Database Software
+
Database Data
```

---

# Database Architecture

A typical application communicates with a database as shown below:

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

---

# Database Connectivity Requirements

To connect to any database, we generally need:

```text
IP Address
Port Number
Protocol
Username
Password
```

Example:

```text
IP        : 10.0.1.100
Port      : 3306
Database  : MySQL
Username  : root
Password  : ********
```

---

# RDBMS

RDBMS stands for:

```text
Relational Database Management System
```

Data is stored in:

- Tables
- Rows
- Columns

There is a relationship between tables.

---

# Excel vs RDBMS

## Excel

```text
Rows
Columns
Sheets
```

---

## RDBMS

```text
Rows
Columns
Tables
Relationships
```

---

# Example

## Customers Table

| Customer_ID | Name | City |
|------------|------|------|
| 101 | Surendra | Hyderabad |
| 102 | Kumar | Bengaluru |

---

## Orders Table

| Order_ID | Customer_ID | Amount |
|----------|------------|---------|
| 1001 | 101 | 500 |
| 1002 | 102 | 750 |

Relationship:

```text
Customers.Customer_ID
          │
          ▼
Orders.Customer_ID
```

---

# Popular RDBMS Databases

## MySQL

Most popular open-source database.

---

## Oracle Database

Enterprise database solution.

---

## Microsoft SQL Server (MSSQL)

Microsoft relational database.

---

## PostgreSQL

Advanced open-source relational database.

---

# SQL Databases

Examples:

- MySQL
- PostgreSQL
- Oracle
- SQL Server

Characteristics:

- Structured Data
- Tables
- Rows
- Columns
- Relationships

---

# NoSQL Databases

Examples:

- MongoDB
- Cassandra
- DynamoDB

Characteristics:

- Flexible Schema
- High Scalability
- Distributed Systems

---

# MySQL

MySQL is an open-source relational database management system.

Uses:

- Web Applications
- E-Commerce Platforms
- Enterprise Applications
- Content Management Systems

---

# MySQL Default Port

```text
3306
```

Interview Question:

### Which port does MySQL use?

Answer:

```text
3306
```

---

# MySQL Default Administrative User

```text
root
```

Root user has full administrative access.

---

# MySQL Installation Security

After installation, secure the database using:

```bash
mysql_secure_installation --set-root-pass ExpenseApp@1
```

Purpose:

- Set root password
- Improve security
- Disable insecure defaults

---

# MySQL Login

## Syntax

```bash
mysql -u <username> -p<password>
```

---

## Example

```bash
mysql -u root -pExpenseApp@1
```

Important:

```text
No space between -p and password
```

Correct:

```bash
mysql -u root -pExpenseApp@1
```

Incorrect:

```bash
mysql -u root -p ExpenseApp@1
```

---

# MySQL Login Workflow

```text
Client
  │
  ▼
IP Address
  │
  ▼
Port 3306
  │
  ▼
Username
  │
  ▼
Password
  │
  ▼
Database Access
```

---

# Common MySQL Commands

## Show Databases

```sql
SHOW DATABASES;
```

---

## Create Database

```sql
CREATE DATABASE expense;
```

---

## Select Database

```sql
USE expense;
```

---

## Show Tables

```sql
SHOW TABLES;
```

---

## Create Table

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(50)
);
```

---

## View Table Structure

```sql
DESC users;
```

---

## Insert Data

```sql
INSERT INTO users VALUES (1,'Surendra');
```

---

## Read Data

```sql
SELECT * FROM users;
```

---

## Update Data

```sql
UPDATE users
SET name='DevOps'
WHERE id=1;
```

---

## Delete Data

```sql
DELETE FROM users
WHERE id=1;
```

---

# CRUD Operations

CRUD is the foundation of most applications.

---

## Create

Insert new data.

```sql
INSERT INTO users VALUES (1,'Surendra');
```

---

## Read

Retrieve data.

```sql
SELECT * FROM users;
```

---

## Update

Modify existing data.

```sql
UPDATE users
SET name='DevOps'
WHERE id=1;
```

---

## Delete

Remove data.

```sql
DELETE FROM users
WHERE id=1;
```

---

# Database Hierarchy

MySQL follows this structure:

```text
Database Server
     │
     ▼
Database (Schema)
     │
     ▼
Tables
     │
     ▼
Rows
     │
     ▼
Columns
```

---

# Schema vs Database

In MySQL, Schema and Database are generally treated the same.

Example:

```sql
CREATE DATABASE expense;
```

or

```sql
CREATE SCHEMA expense;
```

Both create a database structure.

---

# SQL Query Types

## DDL (Data Definition Language)

Used to define database structures.

Examples:

```sql
CREATE
ALTER
DROP
TRUNCATE
```

---

## DML (Data Manipulation Language)

Used to manipulate data.

Examples:

```sql
INSERT
UPDATE
DELETE
```

---

## DQL (Data Query Language)

Used to retrieve data.

Example:

```sql
SELECT
```

---

## DCL (Data Control Language)

Used to control permissions.

Examples:

```sql
GRANT
REVOKE
```

---

# MySQL User Management

## Create User

```sql
CREATE USER 'appuser'@'%' IDENTIFIED BY 'Password123';
```

---

## Grant Permissions

```sql
GRANT ALL PRIVILEGES ON expense.* TO 'appuser'@'%';
```

---

## Reload Permissions

```sql
FLUSH PRIVILEGES;
```

---

# Common MySQL Administration Commands

## Check MySQL Version

```sql
SELECT VERSION();
```

---

## Show Current User

```sql
SELECT USER();
```

---

## Show Active Databases

```sql
SHOW DATABASES;
```

---

# Database Backup

## Backup Database

```bash
mysqldump -u root -pExpenseApp@1 expense > expense.sql
```

---

## Restore Database

```bash
mysql -u root -pExpenseApp@1 expense < expense.sql
```

---

# Production Database Best Practices

- Use strong passwords
- Do not use root for applications
- Take regular backups
- Restrict database access
- Monitor database performance
- Enable encryption when required

---

# Common Database Ports

| Database | Port |
|-----------|------|
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Oracle | 1521 |
| MSSQL | 1433 |
| MongoDB | 27017 |
| Redis | 6379 |

---

# Frequently Asked Interview Questions

## What is a Database?

An organized collection of data.

---

## What is RDBMS?

Relational Database Management System.

Stores data in tables with relationships.

---

## Difference Between Database and Database Server

| Database | Database Server |
|-----------|-----------|
| Collection of Data | Machine Running Database Software |
| Logical Entity | Physical/Virtual Server |

---

## What is MySQL?

An open-source relational database management system.

---

## What is the Default MySQL Port?

```text
3306
```

---

## What is the Default Administrative User in MySQL?

```text
root
```

---

## What is CRUD?

| Operation | Meaning |
|------------|----------|
| Create | Insert Data |
| Read | Retrieve Data |
| Update | Modify Data |
| Delete | Remove Data |

---

## Difference Between SQL and NoSQL

| SQL | NoSQL |
|------|------|
| Structured | Flexible |
| Tables | Documents |
| Relationships | Schema-less |
| MySQL | MongoDB |

---

## How Do Applications Connect to Databases?

Using:

- IP Address
- Port
- Username
- Password
- Database Name

---

# Key Takeaways

- Databases store application data.
- RDBMS stores data in tables.
- MySQL is a popular relational database.
- MySQL uses Port 3306.
- Root is the default administrative user.
- CRUD operations are Create, Read, Update, and Delete.
- Applications connect using IP, Port, Username, and Password.
- Database → Tables → Rows → Columns is the basic hierarchy.
- SQL databases use structured data.
- Database security and backups are critical in production environments.