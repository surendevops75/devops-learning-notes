# 01 — HTTP and REST for DevOps Engineers

## 1. Overview

Modern DevOps work depends heavily on APIs.

Almost every major platform exposes an API:

```text
GitHub
Jenkins
GitLab
AWS
Kubernetes
ArgoCD
Docker Registry
ECR
Terraform Cloud
SonarQube
Prometheus
Elasticsearch
```

Python becomes much more powerful when you understand what happens underneath an API call.

The basic model is:

```text
Python / curl / browser
          |
          v
       HTTP Request
          |
          v
      Load Balancer
          |
          v
       API Server
          |
          v
      Application
          |
          v
        Database
```

For DevOps automation, you should understand:

```text
HTTP
REST
Methods
Headers
Status codes
JSON
Authentication
TLS
Timeouts
Retries
Idempotency
API versioning
Pagination
Rate limits
```

The key principle is:

> **Before automating an API with Python, understand the HTTP request and response that the Python code is creating.**

---

# 2. What Is an API?

API stands for:

```text
Application Programming Interface
```

An API provides a defined interface through which one system communicates with another.

Example:

```text
Python
   |
   | HTTP request
   v
GitHub API
   |
   | JSON response
   v
Python
```

Python does not need to know how GitHub internally stores repositories.

It only needs to understand the API contract.

---

# 3. What Is HTTP?

HTTP stands for:

```text
HyperText Transfer Protocol
```

It is an application-layer protocol used for communication between clients and servers.

Typical flow:

```text
Client
  |
  | HTTP Request
  v
Server
  |
  | HTTP Response
  v
Client
```

Examples of HTTP clients:

```text
Browser
curl
Postman
Python requests
GitHub Actions
Jenkins
Kubernetes clients
```

---

# 4. HTTP in DevOps

When you execute:

```bash
curl https://api.github.com
```

the high-level flow is:

```text
curl
 |
 v
DNS
 |
 v
TCP connection
 |
 v
TLS handshake
 |
 v
HTTP request
 |
 v
Server
 |
 v
HTTP response
```

Understanding these layers makes troubleshooting much easier.

---

# 5. HTTP Request

An HTTP request generally contains:

```text
Method
URL
Headers
Body
```

Example:

```http
GET /api/v1/applications/payment HTTP/1.1
Host: argocd.example.com
Authorization: Bearer <token>
Accept: application/json
```

Conceptually:

```text
Request
 |
 +-- Method
 +-- URL
 +-- Headers
 +-- Body
```

Not every request contains a body.

---

# 6. HTTP Response

A response generally contains:

```text
Status code
Headers
Body
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "name": "payment",
  "status": "healthy"
}
```

Conceptually:

```text
Response
 |
 +-- Status
 +-- Headers
 +-- Body
```

---

# 7. HTTP Request/Response Lifecycle

Production flow:

```text
Client
 |
 | 1. Resolve DNS
 v
IP address
 |
 | 2. Establish TCP
 v
TCP connection
 |
 | 3. TLS handshake
 v
Encrypted connection
 |
 | 4. HTTP request
 v
Load Balancer
 |
 v
API server
 |
 v
Application
 |
 | 5. HTTP response
 v
Client
```

A failure can occur at any layer.

---

# 8. HTTP Methods

The most important HTTP methods are:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

For DevOps automation, the most commonly used are:

```text
GET
POST
PUT
PATCH
DELETE
```

---

# 9. GET

GET retrieves a resource.

Example:

```http
GET /api/v1/applications/payment
```

Conceptually:

```text
Client
 |
 | GET
 v
Server
 |
 | resource
 v
Client
```

Typical response:

```http
200 OK
```

---

# 10. GET Example with curl

```bash
curl https://api.example.com/users
```

With headers:

```bash
curl \
  -H "Accept: application/json" \
  https://api.example.com/users
```

Authenticated:

```bash
curl \
  -H "Authorization: Bearer $TOKEN" \
  https://api.example.com/users
```

---

# 11. GET Should Be Safe

A GET request should not intentionally modify server state.

Example:

```text
GET /users/123
```

should retrieve the user.

It should not:

```text
Delete user
Change password
Deploy application
```

This property is important for safe automation.

---

# 12. GET Should Be Idempotent

Repeated GET requests should produce the same intended server-side effect:

```text
GET
GET
GET
```

The resource can change between requests, but GET itself should not cause state mutation.

This makes GET safe for:

```text
Polling
Health checks
Monitoring
Verification
```

---

# 13. POST

POST commonly creates a resource or triggers an operation.

Example:

```http
POST /api/v1/deployments
```

Request:

```json
{
  "application": "payment",
  "version": "abc123"
}
```

Possible response:

```http
201 Created
```

or:

```http
202 Accepted
```

depending on whether the operation is completed synchronously or asynchronously.

---

# 14. POST in DevOps

POST is commonly used for:

```text
Trigger Jenkins build
Create GitHub issue
Create deployment
Trigger ArgoCD operation
Create AWS resource through API
Send webhook
```

Example:

```text
Python
 |
 | POST
 v
Jenkins
 |
 v
Build started
```

---

# 15. POST Is Usually Not Idempotent

Example:

```http
POST /deployments
```

Calling it twice could create:

```text
Deployment 1
Deployment 2
```

Therefore, retrying POST blindly can create duplicate operations.

Production automation must consider this carefully.

---

# 16. Idempotency Keys

Some APIs support:

```text
Idempotency-Key
```

Example:

```http
Idempotency-Key: release-2026-001
```

The server can recognize repeated requests representing the same operation.

Conceptually:

```text
POST + key A
       |
       v
Operation created

POST + key A again
       |
       v
Existing operation returned
```

This is extremely useful for reliable deployment automation.

---

# 17. PUT

PUT commonly replaces a resource representation.

Example:

```http
PUT /users/123
```

Body:

```json
{
  "name": "Surendra",
  "role": "DevOps Engineer"
}
```

PUT is generally idempotent:

```text
PUT
PUT
PUT
```

should result in the same final resource state.

---

# 18. PATCH

PATCH partially modifies a resource.

Example:

```http
PATCH /users/123
```

Body:

```json
{
  "role": "DevSecOps Engineer"
}
```

Only the specified field is changed.

---

# 19. PUT vs PATCH

Think:

```text
PUT
=
replace/update complete representation
```

```text
PATCH
=
partial modification
```

Example:

```text
Current:
{
  name: "A",
  role: "Developer",
  team: "Platform"
}
```

PUT may replace the complete representation.

PATCH may change only:

```json
{
  "role": "DevOps Engineer"
}
```

---

# 20. DELETE

DELETE removes a resource.

Example:

```http
DELETE /applications/payment
```

Possible response:

```http
204 No Content
```

DELETE is generally expected to be idempotent.

After the resource is deleted:

```text
DELETE
DELETE
DELETE
```

should leave the resource absent.

However, exact API behavior depends on the implementation.

---

# 21. HEAD

HEAD is similar to GET but returns headers without the response body.

Useful for:

```text
Checking resource existence
Checking headers
Checking content length
Checking cache information
```

Example:

```bash
curl -I https://example.com
```

---

# 22. OPTIONS

OPTIONS tells the client what operations or communication options are supported.

It is commonly encountered with:

```text
CORS
```

Example:

```http
OPTIONS /api/users
```

A server may return:

```text
Allow: GET, POST, OPTIONS
```

---

# 23. HTTP Method Summary

| Method | Typical use | Safe | Idempotent |
|---|---|---:|---:|
| GET | Read | Yes | Yes |
| POST | Create/trigger | No | Usually No |
| PUT | Replace | No | Yes |
| PATCH | Partial update | No | Not inherently |
| DELETE | Delete | No | Yes |
| HEAD | Headers only | Yes | Yes |
| OPTIONS | Capabilities/CORS | Yes | Yes |

The exact semantics depend on the API implementation, but these are the standard expectations.

---

# 24. HTTP Status Codes

Status codes communicate the result of a request.

Main categories:

```text
1xx Informational
2xx Success
3xx Redirection
4xx Client Error
5xx Server Error
```

For DevOps engineers, 2xx, 4xx, and 5xx are especially important.

---

# 25. 2xx Success

Common:

```text
200 OK
201 Created
202 Accepted
204 No Content
```

---

# 26. 200 OK

The request succeeded.

Example:

```http
GET /api/applications/payment
```

Response:

```http
200 OK
```

Typical for:

```text
GET
Successful update
Successful API operation
```

---

# 27. 201 Created

A new resource was created.

Example:

```http
POST /api/users
```

Response:

```http
201 Created
```

The response may include:

```text
Location header
```

pointing to the created resource.

---

# 28. 202 Accepted

The request was accepted for asynchronous processing.

This is very important for DevOps APIs.

Example:

```text
POST /deploy
       |
       v
202 Accepted
       |
       v
Deployment running
```

It does NOT mean the deployment has finished successfully.

---

# 29. 202 Accepted and Polling

Example:

```text
POST deployment
 |
 v
202 Accepted
 |
 v
GET deployment status
 |
 +-- running
 |
 +-- running
 |
 +-- succeeded
```

Python automation often needs this pattern.

---

# 30. 204 No Content

The request succeeded but there is no response body.

Common with:

```text
DELETE
```

Example:

```http
DELETE /users/123
```

Response:

```http
204 No Content
```

Python should not blindly attempt to parse every 204 response as JSON.

---

# 31. 3xx Redirection

Common:

```text
301 Moved Permanently
302 Found
307 Temporary Redirect
308 Permanent Redirect
```

A client may follow redirects automatically depending on configuration.

For APIs, unexpected redirects can indicate:

```text
Wrong URL
HTTP -> HTTPS redirect
Authentication gateway
Ingress configuration
```

---

# 32. 400 Bad Request

The server cannot process the request because the request is invalid.

Examples:

```text
Invalid JSON
Missing required field
Invalid parameter
Malformed request
```

Example:

```json
{
  "environment": ""
}
```

could produce:

```http
400 Bad Request
```

Usually, retrying the exact same request will not help.

---

# 33. 401 Unauthorized

Usually means authentication is missing or invalid.

Examples:

```text
Missing token
Expired token
Invalid token
```

Important:

```text
401
=
authentication problem
```

Do not simply retry 401 repeatedly.

Refresh credentials or fix authentication.

---

# 34. 403 Forbidden

The server understood the request but refuses authorization.

Example:

```text
Token valid
+
Permission insufficient
=
403
```

This often indicates:

```text
RBAC
IAM
API permissions
ArgoCD project permissions
```

---

# 35. 401 vs 403

Remember:

```text
401
=
Who are you?
```

```text
403
=
I know who you are,
but you cannot do this.
```

This distinction is extremely useful in DevOps troubleshooting.

---

# 36. 404 Not Found

The requested resource does not exist.

Possible reasons:

```text
Wrong URL
Wrong API version
Wrong resource name
Wrong namespace
Wrong application
Resource deleted
```

In automation, validate identifiers before retrying.

---

# 37. 405 Method Not Allowed

The endpoint exists but the HTTP method is not supported.

Example:

```text
POST /users/123
```

when the API only supports:

```text
GET
PATCH
DELETE
```

Response:

```http
405 Method Not Allowed
```

Check the API documentation.

---

# 38. 409 Conflict

The request conflicts with the current server state.

Examples:

```text
Duplicate resource
Concurrent modification
Version conflict
State transition conflict
```

This is important in CI/CD systems.

Example:

```text
Two deployment operations
       |
       v
409 Conflict
```

Do not blindly retry without understanding the conflict.

---

# 39. 422 Unprocessable Content

The server understands the request format but rejects the semantic content.

Example:

```json
{
  "replicas": -1
}
```

The JSON is valid, but the value is invalid.

Some APIs use 422 for validation errors.

---

# 40. 429 Too Many Requests

The client is being rate limited.

Example:

```text
Python automation
 |
 +-- hundreds of requests
 |
 v
429
```

Response may include:

```text
Retry-After
```

Production automation should respect it.

---

# 41. 5xx Server Errors

Common:

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

These generally indicate a server-side or upstream problem.

Some are retryable.

---

# 42. 500 Internal Server Error

Generic server-side failure.

Possible causes:

```text
Application exception
Database failure
Unexpected state
Bug
```

Check server logs and correlation IDs.

---

# 43. 502 Bad Gateway

A gateway/proxy could not obtain a valid response from the upstream service.

Architecture:

```text
Client
 |
 v
Load Balancer / Proxy
 |
 X
Upstream
```

Possible causes:

```text
Upstream unavailable
Connection failure
Invalid upstream response
```

---

# 44. 503 Service Unavailable

The service is temporarily unavailable.

Possible causes:

```text
Overload
Maintenance
No healthy upstream
Pod unavailable
Dependency unavailable
```

Often retryable with backoff.

---

# 45. 504 Gateway Timeout

A gateway did not receive a response from the upstream service within its timeout.

Architecture:

```text
Client
 |
 v
ALB / Proxy
 |
 v
Application
 |
    slow/no response
```

Important troubleshooting areas:

```text
Application latency
Database
Network
Upstream service
Load balancer timeout
```

---

# 46. Status Code Decision Table

| Status | Typical meaning | Retry exact request? |
|---|---|---|
| 200 | Success | No |
| 201 | Created | No |
| 202 | Accepted | Poll status |
| 204 | Success/no body | No |
| 400 | Invalid request | No |
| 401 | Authentication | Fix/refresh |
| 403 | Authorization | Fix permission |
| 404 | Not found | Validate resource |
| 409 | Conflict | Reconcile state |
| 422 | Validation failure | Fix input |
| 429 | Rate limit | Yes, with backoff |
| 500 | Server error | Sometimes |
| 502 | Gateway failure | Often |
| 503 | Unavailable | Often |
| 504 | Gateway timeout | Often |

---

# 47. HTTP Headers

Headers carry metadata.

Examples:

```text
Authorization
Content-Type
Accept
User-Agent
Host
Content-Length
Cache-Control
ETag
If-None-Match
Retry-After
```

Headers are critical for API automation.

---

# 48. Authorization Header

Common format:

```http
Authorization: Bearer <token>
```

Python:

```python
headers = {
    "Authorization":
        f"Bearer {token}"
}
```

Do not log the complete header.

---

# 49. Content-Type

Content-Type tells the server what the request body contains.

Common:

```text
application/json
application/x-www-form-urlencoded
multipart/form-data
```

Example:

```http
Content-Type: application/json
```

---

# 50. Accept

Accept tells the server what response format the client prefers.

Example:

```http
Accept: application/json
```

Difference:

```text
Content-Type
=
What I am sending

Accept
=
What I want back
```

---

# 51. User-Agent

Identifies the client.

Example:

```http
User-Agent: my-devops-automation/1.0
```

A useful production automation User-Agent can make API logs easier to identify.

---

# 52. Retry-After

A server may tell the client how long to wait.

Example:

```http
Retry-After: 30
```

Commonly associated with:

```text
429
503
```

Python automation should honor this where appropriate.

---

# 53. Request ID / Correlation ID

Production systems often use:

```text
X-Request-ID
X-Correlation-ID
```

or another organization-specific header.

Example:

```http
X-Request-ID: release-001
```

This helps trace:

```text
Python
 |
 v
API Gateway
 |
 v
Service
 |
 v
Logs
```

---

# 54. JSON

JSON stands for:

```text
JavaScript Object Notation
```

It is the most common API data format.

Example:

```json
{
  "application": "payment",
  "environment": "prod",
  "healthy": true,
  "replicas": 3
}
```

---

# 55. JSON Types

JSON supports:

```text
String
Number
Boolean
Null
Object
Array
```

Example:

```json
{
  "name": "payment",
  "replicas": 3,
  "healthy": true,
  "metadata": null,
  "tags": [
    "production",
    "microservice"
  ]
}
```

---

# 56. Python and JSON

Python's standard library includes:

```python
import json
```

Convert Python object to JSON:

```python
json.dumps(data)
```

Convert JSON to Python object:

```python
json.loads(text)
```

The Requests library also provides:

```python
response.json()
```

for JSON responses.

---

# 57. Request Body

POST/PUT/PATCH commonly send a body.

Example:

```json
{
  "replicas": 3
}
```

Conceptually:

```text
POST
 |
 +-- Headers
 |
 +-- JSON body
```

---

# 58. Query Parameters

Query parameters appear after `?`.

Example:

```text
/api/users?page=2
```

Multiple:

```text
/api/users?page=2&limit=50
```

Typical uses:

```text
Pagination
Filtering
Sorting
Search
Feature flags
```

---

# 59. Path Parameters

Path parameters identify a specific resource.

Example:

```text
/api/users/123
```

Here:

```text
123
```

is the user identifier.

Compare:

```text
/api/users?page=2
```

with:

```text
/api/users/123
```

The first uses a query parameter.

The second uses a path parameter.

---

# 60. Query vs Path Parameter

Use path when identifying a resource:

```text
/users/123
```

Use query when modifying how a collection is queried:

```text
/users?page=2&role=devops
```

This is a common REST API design pattern.

---

# 61. REST

REST stands for:

```text
Representational State Transfer
```

REST is an architectural style for designing networked APIs.

A RESTful API generally emphasizes:

```text
Resources
HTTP methods
Stateless communication
Representations
Uniform interface
```

---

# 62. REST Resource Model

Instead of:

```text
/getUsers
/createUser
/deleteUser
```

REST commonly models resources:

```text
/users
/users/123
```

Then HTTP methods describe the action:

```text
GET    /users
POST   /users
GET    /users/123
PATCH  /users/123
DELETE /users/123
```

---

# 63. Resource-Oriented Design

Good:

```text
GET /applications/payment
```

Less REST-oriented:

```text
GET /getPaymentApplication
```

The resource is:

```text
applications/payment
```

The HTTP method expresses the operation.

---

# 64. REST Statelessness

Stateless means each request contains the information needed to process it.

The server should not rely on hidden client session state for normal API processing.

Example:

```http
GET /users/123
Authorization: Bearer <token>
```

The request carries the necessary identity information.

---

# 65. Why Stateless APIs Matter for DevOps

Stateless APIs are easier to:

```text
Scale
Load balance
Retry
Automate
Monitor
Deploy
```

Example:

```text
Python
 |
 +------+
 |      |
 v      v
API-1  API-2
```

Requests can be distributed across instances.

---

# 66. REST and Load Balancers

Typical production architecture:

```text
Python
 |
 v
DNS
 |
 v
ALB
 |
 +------+
 |      |
 v      v
API-1  API-2
 |
 v
Database
```

The API can scale horizontally.

---

# 67. REST Constraints

Important REST principles include:

```text
Client-server
Stateless
Cacheable
Uniform interface
Layered system
Optional code-on-demand
```

In practical DevOps interviews, focus especially on:

```text
Statelessness
Resource orientation
HTTP semantics
```

---

# 68. API Versioning

APIs evolve.

Common versioning styles:

```text
/api/v1/users
```

or:

```text
Accept: application/vnd.company.v1+json
```

URL versioning is easy to understand and commonly used.

---

# 69. Why API Versioning Matters

Without versioning:

```text
Client A expects old behavior
Client B expects new behavior
```

A breaking change can break automation.

Versioning allows:

```text
v1
v2
```

to coexist during migration.

---

# 70. Backward Compatibility

A production API should avoid unnecessary breaking changes.

Safer:

```text
Add optional field
```

Riskier:

```text
Remove existing field
Change field type
Change status semantics
```

DevOps automation depends heavily on stable API contracts.

---

# 71. API Pagination

Large APIs should not return millions of objects in one response.

Common approaches:

```text
page + limit
offset + limit
cursor
```

Example:

```text
GET /applications?page=2&limit=50
```

---

# 72. Cursor Pagination

A response may include:

```json
{
  "items": [],
  "next_cursor": "abc123"
}
```

Next request:

```text
GET /applications?cursor=abc123
```

Cursor pagination is often more reliable for changing datasets.

---

# 73. Python Pagination Concept

```python
items = []

url = "/applications"

while url:

    data = client.get(url)

    items.extend(
        data["items"]
    )

    url = data.get(
        "next"
    )
```

Production code should also enforce:

```text
Maximum pages
Timeout
Rate-limit handling
```

---

# 74. Filtering

Example:

```text
GET /applications?environment=prod
```

Filtering reduces unnecessary data transfer.

Useful for:

```text
Production applications
Failed builds
Open pull requests
Running jobs
```

---

# 75. Sorting

Example:

```text
GET /deployments?sort=created_at
```

APIs vary in syntax.

Never assume a sorting parameter exists; check the API contract.

---

# 76. API Response Size

Large responses can cause:

```text
Memory usage
Latency
Timeout
Network overhead
```

Prefer:

```text
Pagination
Filtering
Field selection
```

when supported.

---

# 77. Content Negotiation

Clients and servers can negotiate representations.

Example:

```http
Accept: application/json
```

The server may return JSON.

Some APIs support:

```text
application/json
application/xml
```

For modern DevOps automation, JSON is usually the primary format.

---

# 78. HTTP Cookies

Cookies store client-related state.

Example:

```http
Set-Cookie: session=abc...
```

Browsers use cookies extensively.

API automation may instead use:

```text
Bearer tokens
API keys
OAuth
```

depending on the API.

---

# 79. Cookies vs Bearer Tokens

### Cookie

Common for:

```text
Browser sessions
Web applications
```

### Bearer token

Common for:

```text
REST APIs
Automation
Service-to-service calls
```

A bearer token must be protected because possession generally grants its associated permissions.

---

# 80. HTTPS

HTTPS is HTTP over TLS.

Architecture:

```text
HTTP
 +
TLS
 =
HTTPS
```

Example:

```text
https://api.example.com
```

---

# 81. Why HTTPS Matters

TLS provides:

```text
Encryption
Server authentication
Integrity
```

Without TLS:

```text
Token
Password
Request body
```

could potentially be exposed on an untrusted network.

---

# 82. TLS High-Level Flow

```text
Client
 |
 | ClientHello
 v
Server
 |
 | Certificate
 v
Client verifies certificate
 |
 | Key agreement
 v
Encrypted session
 |
 v
HTTP
```

You do not need to implement TLS manually in Python.

Libraries such as Requests use TLS support from the underlying environment.

---

# 83. TLS Certificate Validation

Python API clients should validate certificates.

Do not solve certificate errors with:

```python
verify=False
```

unless there is a controlled, explicit reason and compensating security controls.

For production:

```text
Valid certificate
Trusted CA
Correct hostname
```

---

# 84. DNS in API Communication

When Python calls:

```text
https://api.example.com
```

the system needs to resolve:

```text
api.example.com
```

to an IP address.

Flow:

```text
Python
 |
 v
DNS
 |
 v
IP
 |
 v
TCP/TLS
```

DNS failures can look like API failures even though the application is healthy.

---

# 85. TCP Before HTTP

HTTP commonly runs over TCP.

Simplified:

```text
DNS
 |
 v
TCP connection
 |
 v
TLS
 |
 v
HTTP
```

If TCP cannot connect:

```text
HTTP request never reaches server
```

---

# 86. Connection Refused

Example:

```text
Connection refused
```

Possible causes:

```text
Nothing listening
Wrong port
Service unavailable
Firewall behavior
```

Check:

```bash
curl
nc
ss
```

where appropriate.

---

# 87. Connection Timeout

Example:

```text
Connection timed out
```

Possible causes:

```text
Network path
Security group
Firewall
Route
Service unavailable
Proxy
```

Difference:

```text
Connection refused
=
host reachable, port actively rejected

Connection timeout
=
connection could not complete in time
```

Exact interpretation depends on the network path.

---

# 88. HTTP Timeout Types

There are multiple stages:

```text
Connect timeout
Read timeout
Overall operation timeout
```

Example:

```text
Client
 |
 | connect
 |------ timeout
 |
 | read response
 |------ timeout
```

Production clients should set explicit timeouts.

---

# 89. Why Timeouts Matter

Without timeouts:

```python
requests.get(url)
```

could wait longer than expected.

In automation:

```text
Pipeline hangs
Runner remains occupied
Deployment blocks
```

Always define sensible timeouts.

---

# 90. Load Balancer and HTTP

AWS architecture:

```text
Client
 |
 v
Route53
 |
 v
ALB
 |
 +------+
 |      |
 v      v
Pod    Pod
```

The ALB handles:

```text
Traffic distribution
TLS termination
Health checks
Routing
```

depending on configuration.

---

# 91. API Gateway vs ALB

API Gateway and ALB solve different problems.

### API Gateway

Common for:

```text
API management
Authentication integrations
Throttling
Request transformation
Serverless/API workloads
```

### ALB

Common for:

```text
HTTP/HTTPS load balancing
Kubernetes ingress
Path/host routing
Container workloads
```

Your EKS architecture can use ALB with the AWS Load Balancer Controller.

---

# 92. REST API Through ALB

Example:

```text
Python
 |
 v
Route53
 |
 v
ALB
 |
 v
Ingress
 |
 v
Service
 |
 v
Pod
```

Troubleshooting can follow the same path.

---

# 93. Kubernetes API

Kubernetes itself exposes an API server.

Example:

```text
Python
 |
 v
Kubernetes API Server
 |
 v
Cluster state
```

The Kubernetes Python client eventually communicates through HTTP-based APIs.

This is why HTTP/REST knowledge is directly useful for Kubernetes automation.

---

# 94. ArgoCD API

ArgoCD also exposes APIs.

Architecture:

```text
Python
 |
 v
ArgoCD API
 |
 v
ArgoCD
 |
 v
Kubernetes
```

Python can:

```text
Get application
Get health
Get sync
Trigger operations
```

The exact API transport may include REST/gRPC depending on the operation and client.

---

# 95. GitHub API

GitHub exposes APIs for:

```text
Repositories
Issues
Pull requests
Actions
Releases
Branches
Workflow runs
Artifacts
```

Example:

```text
Python
 |
 v
GitHub API
 |
 v
Workflow run
```

---

# 96. Jenkins API

Jenkins provides HTTP APIs for automation.

Python can:

```text
Trigger jobs
Check builds
Get build status
Read metadata
```

Typical flow:

```text
Python
 |
 | POST
 v
Jenkins
 |
 v
Build
 |
 | GET status
 v
Python
```

---

# 97. SonarQube API

Python can use APIs to retrieve:

```text
Quality gate
Project status
Analysis results
Metrics
```

This can be integrated into CI/CD gates.

---

# 98. Prometheus HTTP API

Prometheus exposes an HTTP API.

Python can query metrics.

Conceptual:

```text
Python
 |
 | HTTP GET
 v
Prometheus
 |
 | JSON
 v
Python
```

Example query:

```text
rate(http_requests_total[5m])
```

The API response can be processed by Python.

---

# 99. Elasticsearch API

Elasticsearch exposes HTTP APIs.

Python can use them to:

```text
Search logs
Query errors
Aggregate events
Investigate incidents
```

Architecture:

```text
Python
 |
 v
Elasticsearch
 |
 v
Logs
```

---

# 100. REST API in a DevOps Toolchain

A production toolchain can look like:

```text
                  Python
                    |
       +------------+-------------+
       |            |             |
       v            v             v
    GitHub       Jenkins        AWS
       |            |             |
       +------------+-------------+
                    |
                    v
                 ECR
                    |
                    v
                 ArgoCD
                    |
                    v
                  EKS
                    |
            +-------+-------+
            |               |
            v               v
       Prometheus          ELK
```

HTTP/API knowledge is the common foundation connecting these systems.

---

# 101. REST API Design for DevOps Tools

A good API should expose resources clearly.

Example:

```text
/applications
/applications/{id}
/deployments
/deployments/{id}
/releases
/releases/{id}
```

Use methods:

```text
GET
POST
PATCH
DELETE
```

according to semantics.

---

# 102. Deployment Resource Example

```text
POST /deployments
```

Request:

```json
{
  "application": "payment",
  "environment": "prod",
  "image_digest": "sha256:..."
}
```

Response:

```json
{
  "deployment_id": "dep-123",
  "status": "accepted"
}
```

Then:

```text
GET /deployments/dep-123
```

Response:

```json
{
  "deployment_id": "dep-123",
  "status": "succeeded"
}
```

This asynchronous model is common in DevOps platforms.

---

# 103. Synchronous vs Asynchronous API

### Synchronous

```text
Request
 |
 v
Server processes
 |
 v
Response
```

The client waits for completion.

### Asynchronous

```text
Request
 |
 v
Server accepts
 |
 v
202 Accepted
 |
 v
Background job
 |
 v
Client polls status
```

CI/CD systems commonly use asynchronous operations.

---

# 104. Polling

Polling means repeatedly checking status.

Example:

```text
GET /deployments/123
```

every few seconds.

Good polling:

```text
5–15 seconds
bounded duration
backoff when appropriate
```

Bad polling:

```text
100 requests/second
forever
```

---

# 105. Polling with Backoff

Conceptual:

```text
2 sec
 |
 v
4 sec
 |
 v
8 sec
 |
 v
10 sec
```

Cap the delay.

This reduces API load.

---

# 106. Long-Running Operations

Examples:

```text
Terraform apply
EKS deployment
Docker build
Jenkins pipeline
ArgoCD sync
AWS provisioning
```

A good API may return:

```text
operation ID
```

rather than keeping the HTTP request open for the entire operation.

---

# 107. Operation Resource

Example:

```text
POST /deployments
```

Response:

```json
{
  "operation_id": "op-123",
  "status": "running"
}
```

Then:

```text
GET /operations/op-123
```

This separates:

```text
Request acceptance
```

from:

```text
Operation completion
```

---

# 108. HTTP Keep-Alive

HTTP connections can be reused.

Conceptually:

```text
Connection
 |
 +-- Request 1
 +-- Response 1
 |
 +-- Request 2
 +-- Response 2
```

This reduces connection setup overhead.

Python's `requests.Session()` can reuse connections.

This becomes important when automating many API calls.

---

# 109. HTTP/1.1 vs HTTP/2

HTTP/1.1 commonly uses:

```text
Persistent connections
```

HTTP/2 adds:

```text
Multiplexing
Header compression
Binary framing
```

From an automation perspective, the important point is:

> Use a mature HTTP client/library and let it handle protocol details unless you have a specific need to control them.

---

# 110. API Rate Limiting

Providers may limit:

```text
Requests per second
Requests per minute
Requests per hour
```

Example:

```text
Python
 |
 +-- request
 +-- request
 +-- request
 |
 v
429 Too Many Requests
```

Handle with:

```text
Backoff
Retry-After
Request reduction
Pagination
Caching
Concurrency limits
```

---

# 111. Rate Limit Headers

Some APIs return:

```text
X-RateLimit-Limit
X-RateLimit-Remaining
X-RateLimit-Reset
```

Use these when available.

They can help the automation adapt before hitting the limit.

---

# 112. Caching

Caching reduces API calls.

Example:

```text
GET repository metadata
```

If the same information is requested repeatedly and is safe to cache:

```text
Cache
 |
 v
Avoid API call
```

Do not cache sensitive or rapidly changing state without understanding the consistency requirements.

---

# 113. ETags

HTTP may use:

```text
ETag
```

to identify a representation version.

Client can send:

```http
If-None-Match: "abc123"
```

Server may respond:

```http
304 Not Modified
```

This reduces unnecessary data transfer.

---

# 114. Optimistic Concurrency

APIs may use:

```text
ETag
If-Match
```

to prevent overwriting newer changes.

Conceptually:

```text
Client reads version A
 |
 v
Another client changes resource
 |
 v
Client tries update with version A
 |
 v
Conflict
```

This is useful for GitOps and configuration APIs.

---

# 115. API Contracts

An API contract defines:

```text
Endpoint
Method
Request schema
Headers
Authentication
Response schema
Status codes
Errors
```

For production automation, the contract is the source of truth.

Do not guess API behavior.

---

# 116. OpenAPI

OpenAPI describes REST APIs in a machine-readable format.

It can define:

```text
Paths
Methods
Parameters
Schemas
Authentication
Responses
```

Example:

```text
OpenAPI
 |
 +-- GET /applications
 +-- POST /deployments
 +-- GET /deployments/{id}
```

OpenAPI can also support client generation and validation.

---

# 117. API Documentation

Before automating an API, identify:

```text
Base URL
API version
Authentication
Required headers
Endpoints
Request schema
Response schema
Status codes
Rate limits
Pagination
Timeout expectations
```

This prevents many production bugs.

---

# 118. API Troubleshooting Method

When an API call fails:

```text
1. Check DNS
2. Check connectivity
3. Check TLS
4. Check URL
5. Check method
6. Check headers
7. Check authentication
8. Check request body
9. Check status code
10. Check response body
11. Check server logs
12. Check upstream dependencies
```

This should become a standard troubleshooting workflow.

---

# 119. curl as a Debugging Tool

Before debugging Python, reproduce the API call with curl.

Example:

```bash
curl -v \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/json" \
  https://api.example.com/health
```

This separates:

```text
API problem
```

from:

```text
Python implementation problem
```

---

# 120. curl Verbose Mode

```bash
curl -v https://api.example.com
```

can reveal:

```text
DNS/connection information
TLS handshake details
Request headers
Response headers
Status code
```

Do not expose secrets in shared terminal output.

---

# 121. Testing POST with curl

Example:

```bash
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"environment":"staging"}' \
  https://api.example.com/deployments
```

For destructive operations, validate the target carefully before executing.

---

# 122. API Gateway Troubleshooting

If:

```text
Python -> API Gateway -> service
```

fails, check:

```text
DNS
TLS
API route
Authorization
Rate limits
Integration
Backend
```

A 403 could be:

```text
API authorization
```

not necessarily the backend.

---

# 123. ALB Troubleshooting

If:

```text
Python -> ALB -> EKS
```

fails:

```text
DNS
 |
 v
ALB
 |
 +-- Listener
 +-- Rule
 +-- Target group
 |
 v
Service
 |
 v
Pod
```

Check each layer.

---

# 124. Kubernetes API Troubleshooting

If Python cannot connect:

```text
Python
 |
 v
Kubernetes API
```

check:

```text
Kubeconfig
Authentication
Endpoint
TLS
Network
RBAC
```

Then determine:

```text
401
```

vs:

```text
403
```

because they indicate different problems.

---

# 125. REST API Security

Common controls:

```text
HTTPS
Authentication
Authorization
Input validation
Rate limiting
Audit logging
Secret management
Request size limits
```

---

# 126. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
Valid token
+
No permission
=
403
```

---

# 127. Input Validation

Never trust API input.

Validate:

```text
Environment
Resource ID
Image digest
Namespace
Repository
File paths
Numeric values
```

Example:

```python
if environment not in {
    "dev",
    "staging",
    "prod"
}:
    raise ValueError(
        "Invalid environment"
    )
```

---

# 128. Command Injection Risk

Dangerous:

```python
os.system(
    f"curl {user_url}"
)
```

If input is malicious, it can become command injection.

Prefer:

```python
requests.get(
    validated_url,
    timeout=10
)
```

For subprocesses, pass arguments as a list rather than constructing shell strings.

---

# 129. SSRF Risk

Server-Side Request Forgery can occur when an application accepts arbitrary URLs and makes requests to them.

Dangerous design:

```text
User supplies URL
 |
 v
Server fetches URL
```

Potential targets can include:

```text
Internal services
Cloud metadata endpoints
Private networks
```

Validate and restrict outbound destinations where applicable.

This is especially important in DevOps automation platforms.

---

# 130. Request Size Limits

Large request bodies can consume:

```text
CPU
Memory
Network
```

Production APIs should define reasonable limits.

---

# 131. API Authentication Preparation

The next topic is:

```text
04-Authentication.md
```

You should understand the basic HTTP layer first:

```text
Authorization header
Bearer token
API key
Cookie
```

Then authentication mechanisms can be studied properly.

---

# 132. API Error Response Design

A good API should return structured errors.

Example:

```json
{
  "error": {
    "code": "INVALID_ENVIRONMENT",
    "message": "Environment must be dev, staging, or prod",
    "request_id": "req-123"
  }
}
```

This is much better for automation than:

```text
Something went wrong
```

---

# 133. Request IDs in Errors

If an API returns:

```json
{
  "error": "deployment failed",
  "request_id": "req-123"
}
```

record:

```text
request_id
```

in your pipeline logs.

Then operations teams can search the server logs using the same ID.

---

# 134. API Observability

Monitor:

```text
Request rate
Latency
Error rate
Status code distribution
Timeouts
Retries
Rate limits
```

A useful dashboard:

```text
Requests/sec
p50 latency
p95 latency
p99 latency
4xx rate
5xx rate
429 rate
```

---

# 135. RED Method

For APIs, RED metrics are useful:

```text
Rate
Errors
Duration
```

Example:

```text
Rate:
100 req/s

Errors:
2%

Duration:
p95 = 250ms
```

These metrics help determine API health.

---

# 136. API Logs

Log useful metadata:

```text
timestamp
method
path
status
duration
request_id
client
```

Avoid logging:

```text
Authorization header
Passwords
Tokens
Sensitive request bodies
```

---

# 137. API Monitoring in DevOps

Example:

```text
Python release automation
 |
 v
ArgoCD API
 |
 v
Prometheus
```

If ArgoCD API latency increases:

```text
Deployment automation
```

may also become slow.

This is why dependencies should be monitored.

---

# 138. API Dependency Chain

Example:

```text
GitHub Actions
 |
 v
Python
 |
 v
GitHub API
 |
 v
GitOps
 |
 v
ArgoCD API
 |
 v
Kubernetes API
 |
 v
EKS
```

One dependency failure can affect the entire deployment.

---

# 139. Failure Domain Thinking

If Python gets:

```text
503 from ArgoCD
```

ask:

```text
Is ArgoCD down?
Is argocd-server overloaded?
Is ingress broken?
Is Kubernetes overloaded?
Is Redis unhealthy?
```

Do not immediately assume the Python code is wrong.

---

# 140. API Dependency Timeout Budget

Suppose the overall release must finish in:

```text
30 minutes
```

Do not allow individual API calls to consume unlimited time.

Define:

```text
Connect timeout
Read timeout
Retry budget
Overall operation deadline
```

This prevents cascading delays.

---

# 141. Circuit Breaker Concept

If an external API repeatedly fails:

```text
Python
 |
 +-- request -> fail
 +-- request -> fail
 +-- request -> fail
```

A circuit breaker can temporarily stop sending requests.

Conceptually:

```text
Closed
 |
 v
Failures
 |
 v
Open
 |
 v
Wait
 |
 v
Half-open
 |
 +-- success -> Closed
 +-- failure -> Open
```

This is useful for high-volume integrations.

---

# 142. API Retry Storm

Bad design:

```text
100 workers
x
retry immediately
```

This can overload an already unhealthy API.

Use:

```text
Backoff
Jitter
Concurrency limits
Circuit breaker
```

---

# 143. API Client Pooling

When making many requests:

```python
import requests

session = requests.Session()
```

A session can reuse connections.

This can improve:

```text
Performance
Latency
Resource utilization
```

The next file will cover Requests in much greater depth.

---

# 144. REST vs SOAP

### REST

Commonly uses:

```text
HTTP
JSON
Resources
```

### SOAP

Uses:

```text
XML
Strict messaging standards
WSDL
```

Modern DevOps tooling commonly exposes REST APIs, although SOAP still exists in enterprise systems.

---

# 145. REST vs GraphQL

### REST

Multiple resource endpoints:

```text
/users
/users/123
/orders
```

### GraphQL

Often a single endpoint:

```text
/graphql
```

where the client specifies the requested data.

For DevOps automation, REST is still extremely common across infrastructure and CI/CD platforms.

---

# 146. Webhooks vs REST Polling

### Polling

```text
Python
 |
 +-- GET
 +-- GET
 +-- GET
```

### Webhook

```text
Event
 |
 v
Provider
 |
 | HTTP POST
 v
Your service
```

Webhooks can reduce polling.

Example:

```text
GitHub push
 |
 v
Webhook
 |
 v
CI service
```

---

# 147. Webhook Security

Validate:

```text
Signature
Timestamp
Source
Payload
Replay protection
```

Do not trust arbitrary incoming POST requests.

---

# 148. REST API in a Production Release

A mature deployment system might work like:

```text
1. POST release
2. Receive 202
3. Receive operation ID
4. Poll operation
5. Detect artifact published
6. Check GitOps revision
7. Check ArgoCD
8. Check EKS
9. Verify health
10. Return release result
```

This is the architecture you should be able to design in an interview.

---

# 149. Production API Checklist

```text
[ ] HTTPS enabled
[ ] TLS certificates valid
[ ] Authentication configured
[ ] Authorization configured
[ ] Input validation implemented
[ ] Timeouts configured
[ ] Retry policy defined
[ ] Rate limits understood
[ ] Pagination implemented
[ ] API version known
[ ] Response schema validated
[ ] Error schema handled
[ ] Request IDs captured
[ ] Secrets excluded from logs
[ ] API metrics available
[ ] API logs available
[ ] Dependency failures classified
[ ] Idempotency considered
[ ] Concurrency controlled
[ ] Webhook signatures validated
[ ] Sensitive endpoints protected
```

---

# 150. Interview Questions

## Q1. What is HTTP?

HTTP is an application-layer protocol used for communication between clients and servers.

A typical API interaction is:

```text
Client
 |
 v
HTTP Request
 |
 v
Server
 |
 v
HTTP Response
```

---

## Q2. What is REST?

REST is an architectural style that models APIs around resources and uses standard HTTP semantics.

Example:

```text
GET /users/123
PATCH /users/123
DELETE /users/123
```

---

## Q3. What is the difference between PUT and PATCH?

PUT generally replaces the complete resource representation.

PATCH partially modifies a resource.

---

## Q4. What is the difference between 401 and 403?

```text
401 = authentication problem
403 = authorization problem
```

---

## Q5. What is 202 Accepted?

It means the server accepted the request for processing but the operation may still be running.

This is common for:

```text
Deployments
Builds
Provisioning
```

The client should check operation status.

---

## Q6. Why should you not blindly retry POST?

POST is generally not inherently idempotent.

A retry can create duplicate operations.

Use:

```text
Idempotency key
Operation ID
State reconciliation
```

where supported.

---

## Q7. What is statelessness?

Each request contains the information required to process it rather than relying on hidden server-side session state.

---

## Q8. What is the difference between a path parameter and query parameter?

Path:

```text
/users/123
```

identifies a resource.

Query:

```text
/users?page=2
```

modifies how the collection is queried.

---

## Q9. What does 429 mean?

The client has exceeded a rate limit.

Use:

```text
Retry-After
Backoff
Jitter
```

and reduce request volume.

---

## Q10. What does 502 mean?

A gateway or proxy could not obtain a valid response from its upstream.

---

## Q11. What does 504 mean?

A gateway timed out waiting for the upstream service.

---

## Q12. Why are timeouts important in CI/CD automation?

Without timeouts, a pipeline can hang indefinitely while waiting for an external service.

Timeouts provide bounded failure behavior.

---

## Q13. How do you troubleshoot a REST API failure?

I start with:

```text
DNS
Connectivity
TLS
URL
Method
Headers
Authentication
Body
Status code
Response
Server logs
Dependencies
```

---

## Q14. Why should you reproduce a Python API problem with curl?

It helps determine whether the problem is:

```text
API/server
```

or:

```text
Python client implementation
```

---

## Q15. What is idempotency?

An operation is idempotent when repeating the same operation produces the same intended final server state.

Typical examples:

```text
GET
PUT
DELETE
```

POST is not generally idempotent.

---

# 151. Scenario-Based Interview Questions

## Scenario 1 — Python Gets 401

### Answer

I would check:

```text
Token present?
Token expired?
Correct authentication scheme?
Correct API endpoint?
```

I would not keep retrying the same invalid token.

---

## Scenario 2 — Python Gets 403

### Answer

Authentication likely succeeded, so I would investigate:

```text
RBAC
API permissions
ArgoCD project permissions
AWS IAM
Repository permissions
```

---

## Scenario 3 — Python Gets 429

### Answer

I would:

```text
Read Retry-After
Reduce concurrency
Apply exponential backoff
Add jitter
Respect provider limits
```

I would not immediately increase retry frequency.

---

## Scenario 4 — POST Deployment Returns 202

### Answer

I would treat it as:

```text
Request accepted
```

not:

```text
Deployment successful
```

Then I would use the returned operation/deployment ID to poll status until:

```text
Succeeded
Failed
Timeout
```

---

## Scenario 5 — API Returns 504

### Answer

I would determine whether the timeout occurred at:

```text
Client
ALB
API Gateway
Application
Upstream dependency
```

Then inspect latency and logs.

---

## Scenario 6 — API Works with curl but Not Python

### Answer

I would compare:

```text
URL
Method
Headers
Authentication
Body
TLS verification
Proxy
Timeout
```

I would capture the actual HTTP request characteristics without exposing secrets.

---

## Scenario 7 — DELETE Request Timed Out

### Answer

I would not immediately issue DELETE again.

Because the request may have reached the server and succeeded.

I would:

```text
GET resource
Check operation state
Check server logs
Determine current state
```

Then decide whether another request is necessary.

---

## Scenario 8 — REST API Is Slow

### Answer

I would check:

```text
DNS
Network latency
TLS
Load balancer
API server
Database
External dependencies
```

Then inspect:

```text
p50
p95
p99
```

rather than relying only on average latency.

---

## Scenario 9 — API Is Returning 500

### Answer

I would classify it as a server-side failure and inspect:

```text
API logs
Request ID
Stack trace
Database
Dependencies
Recent deployment
```

Retry only if the operation is safe and the failure is transient.

---

## Scenario 10 — API Works Internally but Not from CI Runner

### Answer

I would investigate the network path:

```text
CI runner
 |
 v
DNS
 |
 v
Proxy/firewall
 |
 v
Load balancer
 |
 v
API
```

Potential causes:

```text
Private endpoint
Security group
Firewall
DNS
Proxy
Routing
```

---

# 152. Senior-Level Interview Question

## Design a Python API client for a production DevOps platform.

### Strong Answer

I would build a reusable client with:

```text
Base URL
Session
Authentication
Timeouts
TLS verification
Retries
Backoff
Structured errors
Request IDs
Logging
Rate-limit handling
Pagination
```

Architecture:

```text
Release Orchestrator
       |
       v
API Client
       |
 +-----+-----+
 |     |     |
 v     v     v
GitHub AWS ArgoCD
```

I would keep business logic separate from HTTP transport.

For example:

```text
api_client.py
github_client.py
argocd_client.py
aws_client.py
release.py
```

I would make state-changing operations idempotent where possible and reconcile state after timeouts.

---

# 153. Senior Scenario

## A production deployment API returns 202, then the Python process crashes.

What happens when the automation restarts?

### Strong Answer

The automation should not assume the deployment failed.

It should use a persistent identifier:

```text
Release ID
Operation ID
Deployment ID
```

Then query the external system:

```text
GET operation
```

and reconcile current state.

Example:

```text
Python process
     |
     X crash
     |
     v
Restart
     |
     v
Find release ID
     |
     v
Query deployment
     |
     +-- Running -> continue
     +-- Success -> verify
     +-- Failed -> handle failure
```

This is a core production automation pattern.

---

# 154. Senior Scenario

## Why is `requests.get(url)` not enough for production?

Because production automation needs:

```text
Timeouts
Authentication
TLS validation
Retries
Rate-limit handling
Error classification
Logging
Connection reuse
Response validation
```

A simple API call is fine for experimentation, but production automation needs controlled failure behavior.

---

# 155. Final Mental Model

Remember HTTP/REST as:

```text
                    CLIENT
                      |
                      v
                     DNS
                      |
                      v
                     TCP
                      |
                      v
                     TLS
                      |
                      v
                 HTTP REQUEST
                      |
          +-----------+-----------+
          |                       |
        Method                  Headers
          |                       |
          +-----------+-----------+
                      |
                      v
                 Load Balancer
                      |
                      v
                   API Server
                      |
                      v
                  Application
                      |
                      v
                 HTTP RESPONSE
                      |
          +-----------+-----------+
          |                       |
      Status Code              JSON Body
          |                       |
          +-----------+-----------+
                      |
                      v
                    Client
```

For DevOps:

```text
HTTP
 |
 +-- GitHub
 +-- Jenkins
 +-- AWS
 +-- ECR
 +-- Kubernetes
 +-- ArgoCD
 +-- SonarQube
 +-- Prometheus
 +-- Elasticsearch
```

The common language between these systems is often:

```text
HTTP + JSON + Authentication
```

---

# 156. What You Should Know Before Moving to Requests

You should be able to explain:

```text
What is HTTP?
What is REST?
What is an endpoint?
What is a request?
What is a response?
What is a header?
What is JSON?
GET vs POST?
PUT vs PATCH?
What is DELETE?
401 vs 403?
404?
409?
422?
429?
500?
502?
503?
504?
What is idempotency?
What is statelessness?
What is pagination?
What is API versioning?
What is HTTPS?
What is TLS?
What is a timeout?
What is rate limiting?
What is polling?
What is a webhook?
```

Then the next topic becomes much easier:

```text
08-Python-APIs/
└── 02-Requests-Library.md
```

---

# 157. Final Production Principle

Do not think of an API as:

```text
URL + Python code
```

Think of it as:

```text
DNS
  |
  v
Network
  |
  v
TLS
  |
  v
HTTP
  |
  v
Authentication
  |
  v
Authorization
  |
  v
API contract
  |
  v
Application
  |
  v
Database/dependencies
  |
  v
Response
```

When a DevOps automation fails, troubleshoot from the outside inward.

> **Understand the protocol first. Then automate it with Python.**
