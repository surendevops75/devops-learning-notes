# HTTP Methods and Status Codes

## Introduction

HTTP stands for:

```text
Hyper Text Transfer Protocol
```

HTTP is the communication protocol used between:

```text
Client
   │
   ▼
Web Server
```

Examples:

```text
Browser → Website
Mobile App → Backend API
Frontend → Backend
Backend → Backend
```

---

# HTTP Communication Flow

Example:

```text
http://daws86s.fun/api/transaction
```

Request Flow:

```text
Client
   │
   ▼
Nginx/Web Server
   │
   ▼
Backend Application
   │
   ▼
Database
```

Response follows the reverse path.

---

# Resources and Actions

In API Design:

```text
Nouns  → Resources
Verbs  → Actions
```

Example:

```text
transaction
```

is a:

```text
Resource (Noun)
```

Actions performed on resources:

```text
Create
Read
Update
Delete
```

---

# CRUD Operations

CRUD stands for:

```text
Create
Read
Update
Delete
```

Most APIs are designed around CRUD operations.

---

# HTTP Methods

HTTP Methods represent actions performed on resources.

| Method | CRUD Operation |
|----------|---------------|
| GET | Read |
| POST | Create |
| PUT | Update |
| DELETE | Delete |

---

# GET Method

Used to:

```text
Read Data
```

Example:

```http
GET /api/transaction
```

Request:

```text
http://daws86s.fun/api/transaction
```

Response:

```json
[
  {
    "id": 1,
    "amount": 100,
    "description": "snacks"
  }
]
```

Successful Response:

```text
200 OK
```

---

# POST Method

Used to:

```text
Create Data
```

Example:

```http
POST /api/transaction
```

Request Body:

```json
{
  "amount": "100",
  "desc": "snacks"
}
```

Response:

```text
200 OK
```

or

```text
201 Created
```

---

# PUT Method

Used to:

```text
Update Existing Data
```

Example:

```http
PUT /api/transaction
```

Request Body:

```json
{
  "amount": "100",
  "desc": "ticket"
}
```

Response:

```text
200 OK
```

---

# DELETE Method

Used to:

```text
Delete Data
```

Example:

```http
DELETE /api/transaction
```

Response:

```text
200 OK
```

or

```text
204 No Content
```

---

# CRUD Mapping

| CRUD | HTTP Method |
|--------|------------|
| Create | POST |
| Read | GET |
| Update | PUT |
| Delete | DELETE |

---

# Sample API Lifecycle

## Create Transaction

```http
POST /api/transaction
```

```json
{
  "amount": 100,
  "desc": "snacks"
}
```

---

## Read Transaction

```http
GET /api/transaction
```

---

## Update Transaction

```http
PUT /api/transaction
```

```json
{
  "amount": 100,
  "desc": "ticket"
}
```

---

## Delete Transaction

```http
DELETE /api/transaction
```

---

# HTTP Status Codes

HTTP Status Codes indicate the result of a request.

Example:

```text
200
404
500
```

Status codes are grouped into categories.

---

# 1XX Informational

Indicates:

```text
Request Received
Processing Continues
```

Examples:

```text
100 Continue
101 Switching Protocols
```

---

# 2XX Success

Indicates:

```text
Request Successfully Processed
```

Common Codes:

```text
200 OK
201 Created
204 No Content
```

---

## 200 OK

Request completed successfully.

Example:

```http
GET /api/transaction
```

Response:

```text
200 OK
```

---

## 201 Created

Resource successfully created.

Example:

```http
POST /api/transaction
```

Response:

```text
201 Created
```

---

## 204 No Content

Request successful but no response body.

Example:

```http
DELETE /api/transaction
```

Response:

```text
204 No Content
```

---

# 3XX Redirection

Indicates:

```text
Resource Moved
Use Another Location
```

Examples:

```text
301 Moved Permanently
302 Found
304 Not Modified
```

---

## 304 Not Modified

Used heavily in caching.

Example:

```text
Browser Cache Available
```

Server responds:

```text
304 Not Modified
```

Browser uses cached copy.

---

# 4XX Client Errors

Indicates:

```text
Problem in Client Request
```

Examples:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

---

## 400 Bad Request

Client sent invalid data.

Example:

```json
{
  "amount": "abc"
}
```

Response:

```text
400 Bad Request
```

---

## 401 Unauthorized

Authentication required.

Example:

```text
Username/Password Missing
```

Response:

```text
401 Unauthorized
```

---

## 403 Forbidden

User authenticated but lacks permission.

Example:

```text
Employee trying to access Admin API
```

Response:

```text
403 Forbidden
```

---

## 404 Not Found

Requested resource does not exist.

Example:

```http
GET /api/unknown
```

Response:

```text
404 Not Found
```

---

# 5XX Server Errors

Indicates:

```text
Problem Inside Server
```

Examples:

```text
500 Internal Server Error
503 Service Unavailable
504 Gateway Timeout
```

---

## 500 Internal Server Error

Application crashed or encountered an exception.

Example:

```text
Database Query Failure
```

Response:

```text
500 Internal Server Error
```

---

## 503 Service Unavailable

Service temporarily unavailable.

Examples:

```text
Server Maintenance
Application Down
```

Response:

```text
503 Service Unavailable
```

---

## 504 Gateway Timeout

Reverse Proxy cannot reach backend.

Example:

```text
Nginx
   │
   ▼
Backend Down
```

Response:

```text
504 Gateway Timeout
```

---

# Status Code Summary

| Range | Meaning |
|---------|----------|
| 1XX | Informational |
| 2XX | Success |
| 3XX | Redirection |
| 4XX | Client Error |
| 5XX | Server Error |

---

# DevOps Troubleshooting

Suppose:

```text
http://daws86s.fun/api/transaction
```

is not working.

---

## Check Application

```bash
systemctl status backend
```

---

## Check Process

```bash
ps -ef | grep node
```

---

## Check Listening Ports

```bash
netstat -lntp
```

or

```bash
ss -lntp
```

---

## Check Nginx Logs

```bash
tail -f /var/log/nginx/access.log
```

---

```bash
tail -f /var/log/nginx/error.log
```

---

## Check Backend Logs

```bash
tail -f /var/log/messages
```

---

## Verify Backend Health

```bash
curl http://localhost:8080/health
```

Expected:

```text
Healthy
```

or

```json
{
  "status": "UP"
}
```

---

## Verify DNS

```bash
nslookup daws86s.fun
```

---

## Verify Connectivity

```bash
telnet BACKEND_IP 8080
```

---

# Useful Directories

## Nginx Web Content

```text
/usr/share/nginx/html
```

---

## Nginx Configuration

```text
/etc/nginx
```

---

## Linux Logs Directory

```text
/var/log
```

---

## Nginx Logs

```text
/var/log/nginx
```

Files:

```text
access.log
error.log
```

---

## System Logs

```text
/var/log/messages
```

---

# Frequently Asked Interview Questions

## What is HTTP?

Hyper Text Transfer Protocol used for communication between clients and servers.

---

## What is CRUD?

```text
Create
Read
Update
Delete
```

---

## Which HTTP Method is Used for Reading Data?

```text
GET
```

---

## Which HTTP Method is Used for Creating Data?

```text
POST
```

---

## Which HTTP Method is Used for Updating Data?

```text
PUT
```

---

## Which HTTP Method is Used for Deleting Data?

```text
DELETE
```

---

## What Does 404 Mean?

Requested resource not found.

---

## What Does 401 Mean?

Authentication required.

---

## What Does 403 Mean?

Access denied.

---

## What Does 500 Mean?

Internal server error.

---

## What Does 503 Mean?

Service unavailable.

---

## What Does 504 Mean?

Gateway timeout, usually backend server is unreachable.

---

# Key Takeaways

- HTTP is the foundation of web communication.
- APIs expose resources through URLs.
- CRUD operations map to HTTP methods.
- GET reads data.
- POST creates data.
- PUT updates data.
- DELETE removes data.
- Status codes indicate request outcomes.
- 2XX indicates success.
- 3XX indicates redirection or caching.
- 4XX indicates client-side issues.
- 5XX indicates server-side issues.
- Understanding HTTP methods and status codes is essential for DevOps troubleshooting and API debugging.