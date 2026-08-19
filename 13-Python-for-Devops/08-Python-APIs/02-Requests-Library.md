# 02 — Python Requests Library for DevOps Engineers

## 1. Overview

The `requests` library is one of the most commonly used Python libraries for HTTP API automation.

It provides a simple interface for:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

For DevOps automation, Requests can be used to integrate with:

```text
GitHub
Jenkins
GitLab
ArgoCD
Kubernetes APIs
AWS services through HTTP-based APIs
SonarQube
Prometheus
Elasticsearch
Internal deployment APIs
Monitoring systems
Webhooks
```

Basic example:

```python
import requests

response = requests.get(
    "https://api.example.com/health",
    timeout=10
)

print(response.status_code)
print(response.json())
```

But production automation requires much more than simply calling:

```python
requests.get(url)
```

You need to understand:

```text
Sessions
Headers
Authentication
JSON
Parameters
Timeouts
Retries
Connection pooling
TLS verification
Exceptions
Status handling
Pagination
Rate limits
Logging
Testing
Security
```

The key principle is:

> **Use Requests as the HTTP transport layer, and keep production orchestration, validation, retries, and business logic above it.**

---

# 2. Installation

Install with:

```bash
python -m pip install requests
```

Verify:

```bash
python -c "import requests; print(requests.__version__)"
```

For a project:

```text
requirements.txt
```

Example:

```text
requests==<approved-version>
```

In a production project, pin or otherwise lock dependency versions according to the organization's dependency-management strategy.

---

# 3. Basic GET Request

```python
import requests

response = requests.get(
    "https://api.example.com/users",
    timeout=10
)

print(response.status_code)
print(response.text)
```

The response object contains:

```text
status_code
headers
text
content
json()
url
cookies
history
elapsed
```

---

# 4. Why Set a Timeout?

Avoid:

```python
requests.get(url)
```

in production automation.

A missing timeout can allow a request to wait unexpectedly long.

Better:

```python
requests.get(
    url,
    timeout=10
)
```

A timeout provides bounded failure behavior.

---

# 5. Connect vs Read Timeout

Requests supports a timeout value such as:

```python
timeout=10
```

You can also specify:

```python
timeout=(5, 20)
```

Meaning conceptually:

```text
Connect timeout = 5 seconds
Read timeout    = 20 seconds
```

This distinction matters when troubleshooting:

```text
Cannot establish connection
```

versus:

```text
Connected but server is too slow
```

---

# 6. Basic POST Request

```python
import requests

payload = {
    "service": "payment",
    "environment": "staging"
}

response = requests.post(
    "https://api.example.com/deployments",
    json=payload,
    timeout=10
)

print(response.status_code)
```

Using:

```python
json=payload
```

is usually cleaner than manually calling:

```python
json.dumps(payload)
```

and setting Content-Type yourself.

---

# 7. JSON Parameter

This:

```python
requests.post(
    url,
    json=payload
)
```

serializes the Python object as JSON and sends the appropriate content type.

Compare with:

```python
requests.post(
    url,
    data=payload
)
```

`data` has different behavior and should not be confused with `json`.

---

# 8. Query Parameters

Instead of manually constructing:

```python
url = (
    "https://api.example.com/users"
    "?page=2&limit=50"
)
```

use:

```python
params = {
    "page": 2,
    "limit": 50
}

response = requests.get(
    url,
    params=params,
    timeout=10
)
```

Requests handles URL encoding.

---

# 9. Path Parameters

For:

```text
/users/123
```

build the path from validated identifiers.

Example:

```python
user_id = "123"

url = (
    f"https://api.example.com/users/{user_id}"
)

response = requests.get(
    url,
    timeout=10
)
```

For untrusted input, validate allowed characters and expected format before constructing URLs.

---

# 10. Headers

Example:

```python
headers = {
    "Accept": "application/json",
    "Content-Type": "application/json"
}

response = requests.get(
    url,
    headers=headers,
    timeout=10
)
```

Remember:

```text
Accept
=
response format preferred

Content-Type
=
request body format
```

---

# 11. Authorization Header

Bearer token:

```python
headers = {
    "Authorization": f"Bearer {token}"
}
```

Production rule:

> Never print the token.

Avoid:

```python
print(headers)
```

if the headers contain credentials.

---

# 12. API Key

Some APIs use:

```text
X-API-Key
```

Example:

```python
headers = {
    "X-API-Key": api_key
}
```

Some providers use:

```text
Authorization: Api-Key ...
```

The exact scheme is API-specific.

Always follow the provider's documentation.

---

# 13. Environment Variables

Do not hardcode secrets:

Bad:

```python
TOKEN = "abc123secret"
```

Better:

```python
import os

token = os.environ["API_TOKEN"]
```

CI systems can inject the value securely.

---

# 14. `.env` Files

Local development may use:

```text
.env
```

Example:

```text
API_TOKEN=...
API_URL=https://api.example.com
```

Do not commit `.env` files containing real credentials.

Add:

```text
.env
```

to:

```text
.gitignore
```

for local secret files.

Production should use an approved secret-management mechanism.

---

# 15. Reading Response Text

```python
response = requests.get(
    url,
    timeout=10
)

print(response.text)
```

`text` returns decoded text.

This is useful when the response is:

```text
Plain text
HTML
Unexpected error body
```

---

# 16. Reading Raw Content

```python
response.content
```

returns bytes.

Useful for:

```text
Binary files
Archives
Images
PDFs
```

For normal JSON APIs, use:

```python
response.json()
```

---

# 17. Parsing JSON

```python
data = response.json()

print(
    data["name"]
)
```

Example response:

```json
{
  "name": "payment",
  "status": "healthy"
}
```

Python:

```python
data = response.json()

if data["status"] == "healthy":
    print("Healthy")
```

---

# 18. JSON Parsing Can Fail

Do not assume every successful HTTP response contains valid JSON.

This can fail:

```python
data = response.json()
```

if the server returns:

```text
HTML
Plain text
Empty response
Invalid JSON
```

Production code should handle unexpected response formats.

---

# 19. Status Code Validation

Basic:

```python
if response.status_code == 200:
    ...
```

Better for many APIs:

```python
response.raise_for_status()
```

This raises an exception for 4xx/5xx responses.

---

# 20. `raise_for_status()`

Example:

```python
response = requests.get(
    url,
    timeout=10
)

response.raise_for_status()

data = response.json()
```

This prevents code from continuing as though a failed response were successful.

---

# 21. Important Limitation of `raise_for_status()`

It does not automatically understand business semantics.

For example:

```text
202 Accepted
```

is technically successful but may mean:

```text
Deployment still running
```

Your application logic must interpret it.

---

# 22. Handling Expected Status Codes

Example:

```python
response = requests.post(
    url,
    json=payload,
    timeout=10
)

if response.status_code == 202:
    print("Deployment accepted")

elif response.status_code == 201:
    print("Deployment created")

else:
    response.raise_for_status()
```

---

# 23. Session

Use:

```python
requests.Session()
```

when making multiple requests to the same service.

Example:

```python
session = requests.Session()

response = session.get(
    url,
    timeout=10
)
```

A session provides:

```text
Connection reuse
Shared headers
Shared cookies
Authentication configuration
```

---

# 24. Why Sessions Matter in DevOps

Suppose Python makes:

```text
GET repository
GET branches
GET workflow
GET artifacts
GET release
```

Using a session can reuse connections.

This reduces unnecessary connection setup overhead.

---

# 25. Session Headers

```python
session = requests.Session()

session.headers.update({
    "Accept": "application/json",
    "Authorization":
        f"Bearer {token}"
})
```

Now every request from that session gets the common headers.

---

# 26. Session Cookies

Sessions can persist cookies:

```python
session = requests.Session()

response = session.post(
    login_url,
    json=payload,
    timeout=10
)

response = session.get(
    protected_url,
    timeout=10
)
```

This is more common in session-based web applications than token-based DevOps APIs.

---

# 27. Session vs Individual Request

Without session:

```python
requests.get(...)
requests.get(...)
requests.get(...)
```

With session:

```python
session.get(...)
session.get(...)
session.get(...)
```

For repeated API communication, prefer a session.

---

# 28. Production API Client Structure

A reusable client might look like:

```text
APIClient
 |
 +-- Session
 +-- Base URL
 +-- Headers
 +-- Authentication
 +-- Timeout
 +-- Retry policy
 +-- Error handling
```

Then higher-level clients use it:

```text
GitHubClient
ArgoCDClient
JenkinsClient
SonarQubeClient
```

---

# 29. Base URL

Instead of repeating:

```python
https://api.example.com
```

define:

```python
class APIClient:

    def __init__(self, base_url):
        self.base_url = (
            base_url.rstrip("/")
        )
```

Then:

```python
self.base_url + "/users"
```

---

# 30. Avoid Double Slashes

If:

```text
base_url = https://api.example.com/
```

and:

```text
path = /users
```

naive concatenation produces:

```text
https://api.example.com//users
```

Normalize:

```python
base_url.rstrip("/")
```

and:

```python
path.lstrip("/")
```

---

# 31. Basic API Client

```python
import requests


class APIClient:

    def __init__(
        self,
        base_url,
        token
    ):
        self.base_url = (
            base_url.rstrip("/")
        )

        self.session = requests.Session()

        self.session.headers.update({
            "Authorization":
                f"Bearer {token}",
            "Accept":
                "application/json"
        })

    def get(self, path):

        response = self.session.get(
            self.base_url + path,
            timeout=10
        )

        response.raise_for_status()

        return response.json()
```

This is a starting point, not a complete production client.

---

# 32. Centralize Timeout Configuration

Avoid scattering:

```python
timeout=10
```

through hundreds of functions.

Use configuration:

```python
DEFAULT_TIMEOUT = (
    5,
    20
)
```

Then:

```python
session.get(
    url,
    timeout=DEFAULT_TIMEOUT
)
```

---

# 33. Per-Operation Timeouts

Different operations may require different timeouts.

Example:

```text
GET health
10 sec

GET metadata
10 sec

POST deployment
10 sec request timeout

Deployment polling
5 min overall deadline
```

Do not confuse:

```text
HTTP request timeout
```

with:

```text
overall deployment timeout
```

---

# 34. HTTP Exceptions

Requests exceptions include:

```text
requests.exceptions.Timeout
requests.exceptions.ConnectionError
requests.exceptions.HTTPError
requests.exceptions.RequestException
```

Catch the most specific exception you can handle meaningfully.

---

# 35. Timeout Handling

```python
import requests

try:
    response = requests.get(
        url,
        timeout=10
    )

except requests.exceptions.Timeout:
    print("API timed out")
```

Then decide whether retrying is safe.

---

# 36. Connection Error

```python
try:
    response = requests.get(
        url,
        timeout=10
    )

except requests.exceptions.ConnectionError:
    print(
        "Unable to connect to API"
    )
```

Possible causes:

```text
DNS
Network
Firewall
Service down
Wrong port
Proxy
```

---

# 37. HTTP Error

```python
try:
    response = requests.get(
        url,
        timeout=10
    )

    response.raise_for_status()

except requests.exceptions.HTTPError as exc:
    print(
        f"HTTP failure: {exc}"
    )
```

You can then inspect:

```python
response.status_code
response.text
```

where appropriate.

---

# 38. General RequestException

Requests exceptions inherit from:

```text
RequestException
```

Example:

```python
except requests.exceptions.RequestException as exc:
    ...
```

This is useful as a final transport-layer safety net.

---

# 39. Exception Classification

A production client should distinguish:

```text
Timeout
Connection error
Authentication failure
Authorization failure
Validation failure
Rate limit
Server failure
Unexpected response
```

This enables correct remediation.

---

# 40. Don't Retry Everything

Bad:

```python
while True:
    try:
        ...
    except:
        retry()
```

This can create:

```text
Infinite loops
API overload
Pipeline hangs
Duplicate operations
```

Retries should be:

```text
bounded
selective
observable
```

---

# 41. Retryable Errors

Often candidates:

```text
429
502
503
504
connection timeout
temporary network failure
```

But the operation must also be safe to retry.

---

# 42. Non-Retryable Errors

Usually:

```text
400
401
403
404
422
```

Do not repeatedly retry invalid input or invalid credentials.

---

# 43. 409 Requires Reconciliation

A 409 can be:

```text
Conflict
```

It may require:

```text
GET current state
```

rather than:

```text
retry immediately
```

This is especially important for deployment automation.

---

# 44. Retry-After

Example:

```python
retry_after = response.headers.get(
    "Retry-After"
)
```

If provided, follow the API's recommended wait.

---

# 45. Exponential Backoff

Conceptual:

```text
Attempt 1 -> 1 sec
Attempt 2 -> 2 sec
Attempt 3 -> 4 sec
Attempt 4 -> 8 sec
```

Add a maximum:

```text
max delay = 30 sec
```

---

# 46. Jitter

Without jitter:

```text
100 clients
 |
 +-- retry at exactly 10 sec
 |
 v
API overloaded again
```

With jitter:

```text
Client A -> 8.4 sec
Client B -> 10.2 sec
Client C -> 11.7 sec
```

Jitter spreads load.

---

# 47. Requests + urllib3 Retry

Requests uses urllib3 internally.

A retry policy can be configured through an adapter.

Example:

```python
from requests import Session
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry


retry = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[
        429,
        502,
        503,
        504
    ],
    allowed_methods=[
        "GET",
        "HEAD",
        "OPTIONS"
    ],
    respect_retry_after_header=True
)

session = Session()

adapter = HTTPAdapter(
    max_retries=retry
)

session.mount(
    "https://",
    adapter
)

session.mount(
    "http://",
    adapter
)
```

The allowed methods list is intentionally conservative.

Do not automatically retry state-changing requests unless their idempotency is understood.

---

# 48. Retry Configuration Principles

A good retry policy defines:

```text
Maximum attempts
Retryable status codes
Retryable methods
Backoff
Jitter where needed
Retry-After support
Overall deadline
```

---

# 49. Why POST Retry Is Dangerous

Suppose:

```text
POST /deployments
```

starts deployment.

Python sends request.

Server starts deployment.

Network fails before Python receives the response.

Python sees:

```text
Timeout
```

If Python blindly retries:

```text
POST /deployments
```

the server may start another deployment.

Better:

```text
Check operation state
```

or use:

```text
Idempotency-Key
```

if supported.

---

# 50. Authentication Headers

Common:

```python
headers = {
    "Authorization":
        f"Bearer {token}"
}
```

Keep token creation outside business logic when possible.

---

# 51. Basic Authentication

Requests supports:

```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    url,
    auth=HTTPBasicAuth(
        username,
        password
    ),
    timeout=10
)
```

Do not use Basic Auth over plain HTTP.

Use HTTPS.

---

# 52. API Key Authentication

Example:

```python
session.headers.update({
    "X-API-Key": api_key
})
```

Protect the key like a password.

---

# 53. Bearer Authentication

Example:

```python
session.headers.update({
    "Authorization":
        f"Bearer {token}"
})
```

Common with:

```text
OAuth 2.0
JWT
Personal access tokens
Service tokens
```

The exact token semantics depend on the provider.

---

# 54. TLS Verification

Requests verifies TLS certificates by default.

Do not disable verification casually.

Bad:

```python
requests.get(
    url,
    verify=False
)
```

This can expose the client to man-in-the-middle attacks.

---

# 55. Custom CA Certificate

For an internal API using an enterprise CA:

```python
requests.get(
    url,
    verify="/path/to/ca-bundle.pem",
    timeout=10
)
```

This is preferable to disabling certificate verification.

---

# 56. Client Certificates

Some internal systems use mutual TLS.

Conceptually:

```python
requests.get(
    url,
    cert=(
        "/path/client.crt",
        "/path/client.key"
    ),
    timeout=10
)
```

Protect the private key carefully.

---

# 57. Proxy Configuration

Requests can use proxy configuration.

Example:

```python
proxies = {
    "https": "http://proxy.example.com:8080"
}

response = requests.get(
    url,
    proxies=proxies,
    timeout=10
)
```

Enterprise CI runners may require proxies for external access.

---

# 58. Proxy Troubleshooting

If:

```text
curl works
```

but Python fails, compare:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

Requests can use environment proxy configuration depending on session settings.

---

# 59. NO_PROXY and Kubernetes

Internal endpoints may need to bypass the proxy.

Example:

```text
NO_PROXY:
localhost,
127.0.0.1,
.cluster.local
```

Exact values depend on the environment.

Incorrect proxy configuration can cause:

```text
Timeouts
DNS behavior differences
Internal API failures
```

---

# 60. Environment Variables for API Clients

Useful configuration:

```text
API_BASE_URL
API_TOKEN
API_TIMEOUT
HTTPS_PROXY
NO_PROXY
```

Never place secrets directly into source code.

---

# 61. Request Logging

Production logs should contain:

```text
Method
Path
Status
Duration
Request ID
```

Example:

```text
GET /applications/payment 200 120ms req-123
```

Avoid:

```text
Authorization: Bearer ...
```

---

# 62. Safe Logging

Create a safe representation:

```python
logger.info(
    "API request",
    extra={
        "method": "GET",
        "path": "/applications/payment"
    }
)
```

Do not log:

```text
Token
Password
Private key
Sensitive payload
```

---

# 63. Response Logging

Useful:

```text
status_code
duration
request_id
```

For failures, log a sanitized response body if it does not contain sensitive information.

---

# 64. Correlation IDs

Add:

```python
headers = {
    "X-Request-ID": release_id
}
```

if the API supports custom correlation IDs.

This helps trace:

```text
CI
 |
 v
Python
 |
 v
API
 |
 v
Application logs
```

---

# 65. Response Headers

Inspect:

```python
request_id = response.headers.get(
    "X-Request-ID"
)
```

Some platforms generate their own request IDs.

Always capture them when troubleshooting.

---

# 66. Response Validation

Do not only check:

```python
status_code == 200
```

Validate expected content.

Example:

```python
data = response.json()

if data.get("status") != "healthy":
    raise RuntimeError(
        "Application is unhealthy"
    )
```

HTTP success does not necessarily mean business success.

---

# 67. Schema Validation

For complex APIs, consider validating responses with:

```text
Pydantic
JSON Schema
Typed models
```

Example conceptual model:

```python
class Deployment:
    id: str
    status: str
```

This prevents silently accepting malformed API responses.

---

# 68. Pagination

Basic loop:

```python
items = []

page = 1

while True:

    response = session.get(
        url,
        params={
            "page": page,
            "limit": 100
        },
        timeout=10
    )

    response.raise_for_status()

    data = response.json()

    items.extend(
        data["items"]
    )

    if not data.get("next"):
        break

    page += 1
```

Production additions:

```text
Maximum pages
Rate limits
Timeout
Retry policy
```

---

# 69. Cursor Pagination

Example:

```python
cursor = None

while True:

    params = {
        "limit": 100
    }

    if cursor:
        params["cursor"] = cursor

    response = session.get(
        url,
        params=params,
        timeout=10
    )

    response.raise_for_status()

    data = response.json()

    process(
        data["items"]
    )

    cursor = data.get(
        "next_cursor"
    )

    if not cursor:
        break
```

---

# 70. Pagination Safety

Never assume pagination ends correctly.

Protect against:

```text
Infinite pagination
Repeated cursor
Unexpected empty page
API bug
```

Example:

```python
seen_cursors = set()

if cursor in seen_cursors:
    raise RuntimeError(
        "Pagination cursor repeated"
    )

seen_cursors.add(cursor)
```

---

# 71. Rate Limit Handling

Inspect:

```python
if response.status_code == 429:
    retry_after = response.headers.get(
        "Retry-After"
    )
```

Then wait according to policy.

Do not create a tight loop.

---

# 72. Polling an API

Example:

```python
import time

deadline = time.time() + 300

while time.time() < deadline:

    response = session.get(
        status_url,
        timeout=10
    )

    response.raise_for_status()

    status = response.json()["status"]

    if status == "succeeded":
        break

    if status == "failed":
        raise RuntimeError(
            "Operation failed"
        )

    time.sleep(5)

else:
    raise TimeoutError(
        "Operation timed out"
    )
```

This is a basic pattern.

Production code should use monotonic time for elapsed deadlines.

---

# 73. Use `time.monotonic()`

For elapsed-time calculations:

```python
import time

deadline = (
    time.monotonic()
    + 300
)
```

Why?

System wall-clock time can change.

Monotonic clocks are designed for measuring elapsed duration.

---

# 74. Polling State Machine

Use:

```text
PENDING
RUNNING
SUCCEEDED
FAILED
CANCELLED
```

Then:

```text
PENDING/RUNNING
      |
      v
   Poll again

SUCCEEDED
      |
      v
   Success

FAILED/CANCELLED
      |
      v
   Failure
```

Do not assume every API uses these exact names.

---

# 75. Session Context Manager

A session can be used with:

```python
with requests.Session() as session:

    response = session.get(
        url,
        timeout=10
    )
```

This makes resource lifecycle explicit.

---

# 76. Streaming Responses

For large downloads:

```python
with session.get(
    url,
    stream=True,
    timeout=30
) as response:

    response.raise_for_status()

    for chunk in response.iter_content(
        chunk_size=8192
    ):
        process(chunk)
```

This avoids loading the entire response into memory.

---

# 77. Why Streaming Matters in DevOps

Large artifacts can include:

```text
Logs
Reports
Archives
Artifacts
Backups
```

Loading a multi-gigabyte response into memory can crash a CI runner.

---

# 78. Downloading Files

Example:

```python
with session.get(
    url,
    stream=True,
    timeout=30
) as response:

    response.raise_for_status()

    with open(
        "artifact.zip",
        "wb"
    ) as file:

        for chunk in response.iter_content(
            chunk_size=8192
        ):
            file.write(chunk)
```

Validate:

```text
Expected content
Size
Checksum
Signature
```

when appropriate.

---

# 79. Uploading Files

Requests supports multipart uploads.

Example:

```python
with open(
    "report.json",
    "rb"
) as file:

    response = session.post(
        url,
        files={
            "file": file
        },
        timeout=30
    )
```

Do not accidentally send huge files without appropriate limits and timeout strategy.

---

# 80. Streaming vs JSON

Use:

```python
response.json()
```

for structured API responses.

Use:

```python
stream=True
```

for large response bodies.

---

# 81. HEAD Request

```python
response = session.head(
    url,
    timeout=10
)
```

Useful for:

```text
Existence
Headers
Content-Length
ETag
```

Check whether the specific API/server supports HEAD correctly.

---

# 82. OPTIONS Request

```python
response = session.options(
    url,
    timeout=10
)
```

Useful for API capability/CORS-related inspection.

---

# 83. Redirects

Requests generally follows redirects by default.

You can inspect:

```python
response.history
```

Example:

```python
for item in response.history:
    print(item.status_code)
```

For security-sensitive API calls, understand redirect behavior before assuming credentials or methods are preserved as desired.

---

# 84. Disable Redirects

```python
response = session.get(
    url,
    allow_redirects=False,
    timeout=10
)
```

Useful for debugging:

```text
301
302
307
308
```

---

# 85. URL Validation

Avoid constructing arbitrary requests from untrusted URLs.

Validate:

```text
Scheme
Hostname
Allowed domain
Allowed path
```

Prefer:

```text
https
```

over:

```text
http
```

for sensitive API calls.

---

# 86. SSRF Protection

If automation accepts a URL from a user:

```python
url = user_input
requests.get(url)
```

this can be dangerous.

A malicious URL could target internal services.

Production systems should implement:

```text
Allowlist
DNS/IP validation
Private-range blocking where appropriate
Redirect controls
Network egress controls
```

---

# 87. Request Hooks

Requests supports hooks for certain lifecycle events.

Use them carefully.

For production systems, explicit wrapper functions are often easier to reason about than excessive magic.

---

# 88. Custom HTTPAdapter

Adapters can customize:

```text
Connection pools
Retries
Transport behavior
```

Example:

```python
adapter = HTTPAdapter(
    max_retries=retry
)

session.mount(
    "https://",
    adapter
)
```

---

# 89. Connection Pooling

A Session maintains connection pools through urllib3.

This helps when:

```text
Many requests
Same host
Short intervals
```

Example:

```text
Python
 |
 +-- GET
 +-- GET
 +-- GET
 |
 v
Same API host
```

Connections can be reused.

---

# 90. Pool Size

For high concurrency, pool configuration may need tuning.

However:

> Do not increase connection pools simply because a number looks larger.

Size them according to:

```text
Concurrency
API limits
Runner resources
Server capacity
```

---

# 91. Threading

Requests is commonly used with threads for I/O-bound workloads.

Example concept:

```text
Thread 1 -> API
Thread 2 -> API
Thread 3 -> API
```

But excessive concurrency can cause:

```text
429
Connection exhaustion
API overload
```

Use bounded concurrency.

---

# 92. Requests and Async

Requests itself is synchronous.

For asynchronous/high-concurrency applications, other libraries may be more appropriate.

For typical DevOps automation:

```text
Sequential Requests
+
Session
+
Bounded concurrency
```

is often sufficient.

Do not introduce async complexity without a real requirement.

---

# 93. Thread Safety

A shared Session should be handled carefully when used concurrently.

For complex concurrent systems, consider:

```text
Per-thread sessions
Controlled access
A suitable async client
```

The safest design depends on the concurrency model.

---

# 94. Unit Testing Requests Code

Do not make real production API calls during unit tests.

Mock the transport layer.

Possible tools:

```text
unittest.mock
responses
requests-mock
```

Choose an approved testing approach.

---

# 95. Mocking with `unittest.mock`

Conceptual:

```python
from unittest.mock import patch

@patch("requests.get")
def test_health(mock_get):

    mock_get.return_value.status_code = 200

    response = requests.get(
        "https://example.com",
        timeout=10
    )

    assert response.status_code == 200
```

For real projects, test your own wrapper/client rather than testing Requests itself.

---

# 96. Test Cases for API Clients

Test:

```text
200
201
202
204
400
401
403
404
409
422
429
500
502
503
504
Timeout
ConnectionError
Invalid JSON
Missing fields
Rate limit
Pagination
```

This is far more valuable than testing only the happy path.

---

# 97. Contract Testing

Verify that your client and server agree on:

```text
Request schema
Response schema
Status codes
Headers
```

Contract tests help detect API changes before production.

---

# 98. Mock vs Integration Test

### Unit test

```text
Python client
 |
 v
Mock server
```

Fast.

### Integration test

```text
Python client
 |
 v
Real test API
```

Slower but verifies real behavior.

Use both where appropriate.

---

# 99. Environment Testing

A production API client should be tested against:

```text
Local/mock
Development
Staging
```

before production.

Avoid testing experimental behavior directly against production.

---

# 100. Dependency Pinning

Requests is itself a dependency.

Use controlled versions.

Example:

```text
requests==approved-version
```

Also monitor transitive dependencies.

A dependency update can change:

```text
TLS behavior
Retry behavior
URL parsing
Security behavior
```

---

# 101. Security Updates

Do not pin a version forever without reviewing security updates.

Production dependency management should balance:

```text
Reproducibility
Security
Compatibility
```

---

# 102. User-Agent

Set a meaningful User-Agent:

```python
session.headers.update({
    "User-Agent":
        "devops-release-automation/1.0"
})
```

This helps API operators identify automation traffic.

---

# 103. API Client Logging Example

Conceptual:

```python
start = time.monotonic()

response = session.get(
    url,
    timeout=10
)

duration = (
    time.monotonic() - start
)

logger.info(
    "API request completed",
    extra={
        "method": "GET",
        "path": "/applications/payment",
        "status": response.status_code,
        "duration": duration
    }
)
```

Do not include secrets.

---

# 104. Error Object

Create a structured exception:

```python
class APIError(Exception):

    def __init__(
        self,
        message,
        status_code=None,
        request_id=None
    ):
        super().__init__(message)

        self.status_code = status_code
        self.request_id = request_id
```

This makes higher-level orchestration easier.

---

# 105. Central Error Handler

Example:

```python
def handle_response(
    response
):

    if response.status_code == 429:
        raise RateLimitError()

    if response.status_code == 401:
        raise AuthenticationError()

    if response.status_code == 403:
        raise AuthorizationError()

    response.raise_for_status()
```

Then the release layer can decide what to do.

---

# 106. Separate Transport and Business Logic

Bad:

```text
One function:
  HTTP request
  retry
  parse JSON
  update Git
  deploy
  verify
  notify
```

Better:

```text
HTTP Client
     |
     v
API Client
     |
     v
Service Client
     |
     v
Release Orchestrator
```

Each layer has one responsibility.

---

# 107. Recommended Architecture

```text
               Release Orchestrator
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
   GitHubClient     ArgoCDClient    AWSClient
        |               |               |
        +---------------+---------------+
                        |
                        v
                 HTTP API Client
                        |
                        v
                 Requests Session
                        |
                        v
                      HTTP
```

This is a strong production architecture.

---

# 108. GitHub Client Example

Conceptual:

```python
class GitHubClient:

    def __init__(self, api_client):
        self.api = api_client

    def get_workflow_run(
        self,
        repo,
        run_id
    ):
        return self.api.get(
            f"/repos/{repo}/actions/runs/{run_id}"
        )
```

The GitHub-specific logic is separate from HTTP transport.

---

# 109. ArgoCD Client Example

Conceptual:

```python
class ArgoCDClient:

    def __init__(self, api_client):
        self.api = api_client

    def get_application(
        self,
        name
    ):
        return self.api.get(
            f"/api/v1/applications/{name}"
        )
```

The exact endpoint depends on ArgoCD version/configuration.

---

# 110. Jenkins Client Example

Conceptual:

```python
class JenkinsClient:

    def __init__(self, api_client):
        self.api = api_client

    def trigger_job(
        self,
        job
    ):
        return self.api.post(
            f"/job/{job}/build"
        )
```

Real Jenkins installations may require:

```text
Crumb
Authentication
Job parameters
```

depending on configuration.

---

# 111. Prometheus Client Example

Conceptual:

```python
class PrometheusClient:

    def __init__(self, api_client):
        self.api = api_client

    def query(
        self,
        expression
    ):
        return self.api.get(
            "/api/v1/query",
        )
```

The actual query should be passed safely as a parameter.

---

# 112. Query Parameters with Prometheus

Example:

```python
params = {
    "query": (
        'rate(http_requests_total[5m])'
    )
}

response = session.get(
    prometheus_url,
    params=params,
    timeout=10
)
```

Requests handles URL encoding.

---

# 113. DevOps Example — Check Deployment Health

```python
def verify_health(
    client,
    url
):

    response = client.get(
        url
    )

    if response.status_code != 200:
        raise RuntimeError(
            "Health endpoint failed"
        )

    data = response.json()

    if data.get("status") != "healthy":
        raise RuntimeError(
            "Application unhealthy"
        )
```

This combines:

```text
HTTP
JSON
Status code
Business validation
```

---

# 114. DevOps Example — Poll ArgoCD

Conceptual:

```python
deadline = time.monotonic() + 600

while time.monotonic() < deadline:

    app = argocd.get_application(
        "payment"
    )

    sync = app["status"]["sync"]["status"]
    health = app["status"]["health"]["status"]

    if (
        sync == "Synced"
        and health == "Healthy"
    ):
        return

    time.sleep(10)

raise TimeoutError(
    "ArgoCD application did not become healthy"
)
```

Production code should also inspect:

```text
Operation state
Failure message
Resource health
Deployment revision
```

---

# 115. DevOps Example — Trigger Jenkins

Conceptual:

```python
response = jenkins.trigger_job(
    "payment-build"
)

if response.status_code not in {
    200,
    201,
    202
}:
    response.raise_for_status()
```

Then query the build status rather than assuming the trigger means success.

---

# 116. Trigger vs Completion

Important:

```text
POST /build
```

may mean:

```text
Build accepted
```

not:

```text
Build successful
```

Use:

```text
Trigger
 |
 v
Build ID
 |
 v
Poll
 |
 v
Success/failure
```

This pattern appears throughout DevOps APIs.

---

# 117. API Client Configuration

Example:

```python
from dataclasses import dataclass


@dataclass
class APIConfig:

    base_url: str
    timeout: tuple
    verify_tls: bool = True
```

Keep environment-specific configuration outside the client implementation.

---

# 118. Production Configuration

Example:

```text
DEV:
api.dev.example.com

STAGING:
api.staging.example.com

PROD:
api.example.com
```

Never allow an arbitrary environment to silently select production.

---

# 119. Environment Guard

Example:

```python
if environment == "prod":
    require_approval()
```

Production approval belongs in the CI/CD control plane too.

Python should not be the only security boundary.

---

# 120. API Client Security Boundaries

Use multiple layers:

```text
GitHub environment protection
+
IAM
+
RBAC
+
Python validation
+
API authentication
+
API authorization
```

Defense in depth is better than relying on one script.

---

# 121. Secrets Rotation

API tokens should be rotatable.

Avoid architectures where:

```text
One token
 |
 v
Every pipeline
```

Instead:

```text
Scoped credential
+
Short lifetime where possible
+
Rotation
+
Audit
```

---

# 122. Token Expiration

If a token expires during a long-running workflow:

```text
API call
 |
 v
401
```

The client may need to:

```text
Refresh token
```

if the authentication system supports refresh.

Do not repeatedly retry the expired token.

---

# 123. Requests and OAuth

Requests can send the resulting bearer token:

```python
session.headers.update({
    "Authorization":
        f"Bearer {access_token}"
})
```

OAuth token acquisition itself is an authentication concern and will be covered in:

```text
04-Authentication.md
```

---

# 124. API Client Health Check

Before a major release:

```python
def check_api(
    session,
    url
):

    response = session.get(
        url,
        timeout=10
    )

    response.raise_for_status()

    return True
```

Use preflight checks for critical dependencies.

---

# 125. Dependency Preflight

Example:

```text
CI runner
 |
 +-- GitHub API reachable
 +-- ECR reachable
 +-- ArgoCD reachable
 +-- Kubernetes reachable
 +-- Monitoring reachable
 |
 v
Release
```

Fail early when required dependencies are unavailable.

---

# 126. Fail Fast vs Retry

Use fail-fast for:

```text
Invalid configuration
401
403
400
422
Missing required input
```

Use bounded retry for:

```text
429
502
503
504
Transient network failures
```

Use reconciliation for:

```text
409
Timeout after state-changing operation
```

---

# 127. Retry Budget

Suppose:

```text
Overall API operation:
60 seconds
```

Do not configure:

```text
10 retries x 30 seconds
```

because the retry policy exceeds the operation deadline.

Always think in terms of:

```text
Overall time budget
```

---

# 128. Production Retry Example

Conceptually:

```text
Request
 |
 +-- 503
 |
 v
Wait 1s + jitter
 |
 v
Retry
 |
 +-- 503
 |
 v
Wait 2s + jitter
 |
 v
Retry
 |
 +-- 200
 |
 v
Success
```

Stop after the defined limit.

---

# 129. Retry Metrics

Record:

```text
retry_count
retry_reason
status_code
endpoint
duration
```

This helps identify unstable dependencies.

If an API consistently requires retries, the underlying reliability problem should be investigated.

---

# 130. API Metrics from Python

The automation can expose:

```text
api_requests_total
api_request_failures_total
api_request_duration_seconds
api_retries_total
```

These can be exported to your observability system.

---

# 131. Production API Dashboard

Example:

```text
API Request Rate
API Error Rate
API p95 Latency
429 Rate
Retry Rate
Timeout Rate
```

Break down by:

```text
Service
Endpoint
Environment
Status
```

---

# 132. Common Mistakes

### Mistake 1

No timeout:

```python
requests.get(url)
```

### Mistake 2

Hardcoded token.

### Mistake 3

`verify=False`.

### Mistake 4

Retry every exception.

### Mistake 5

Retry POST blindly.

### Mistake 6

Log credentials.

### Mistake 7

Assume 202 means success.

### Mistake 8

Ignore 429.

### Mistake 9

Load huge responses into memory.

### Mistake 10

Mix HTTP transport with business logic.

---

# 133. Another Common Mistake — Catching Everything

Bad:

```python
try:
    ...
except Exception:
    pass
```

This hides failures.

Better:

```python
except requests.exceptions.Timeout:
    ...
```

and handle known cases explicitly.

---

# 134. Another Common Mistake — Checking Only HTTP 200

Bad:

```python
if response.status_code == 200:
    success()
```

A successful creation may be:

```text
201
```

An accepted asynchronous operation:

```text
202
```

A successful deletion:

```text
204
```

Interpret status codes according to the API contract.

---

# 135. Another Common Mistake — Assuming JSON

Bad:

```python
data = response.json()
```

without considering:

```text
204
HTML error page
empty body
malformed JSON
```

Validate content type and response expectations where necessary.

---

# 136. Another Common Mistake — Hardcoded URLs

Bad:

```python
url = "https://prod.example.com"
```

Better:

```text
Configuration
Environment mapping
```

and validate the environment.

---

# 137. Another Common Mistake — No API Version

Bad:

```text
/api/users
```

when the provider requires a version and behavior can change.

Prefer the documented version:

```text
/api/v1/users
```

when applicable.

---

# 138. Another Common Mistake — Unbounded Pagination

Bad:

```python
while True:
    fetch_next_page()
```

without safeguards.

Add:

```text
Maximum pages
Repeated-cursor detection
Overall deadline
```

---

# 139. Another Common Mistake — Excessive Concurrency

Bad:

```text
1000 concurrent requests
```

to a provider with a much lower rate limit.

Use:

```text
Bounded worker count
Connection pool tuning
Rate limiting
Backoff
```

---

# 140. Another Common Mistake — Ignoring Proxy Configuration

A CI runner may work locally but fail in production because:

```text
HTTPS_PROXY
NO_PROXY
```

are different.

Always consider network environment.

---

# 141. Another Common Mistake — Disabling TLS Verification

Never treat:

```text
SSL error
```

as:

```text
verify=False
```

by default.

Instead investigate:

```text
Certificate
CA trust
Hostname
Proxy interception
Internal CA
```

---

# 142. Another Common Mistake — No Response Validation

Example:

```python
response.raise_for_status()
```

passes.

But the response could still contain:

```json
{
  "status": "failed"
}
```

HTTP success is not always application success.

---

# 143. Production API Client Checklist

```text
[ ] Requests dependency controlled
[ ] Session used for repeated calls
[ ] Base URL configured
[ ] Timeout configured
[ ] Connect/read timeout understood
[ ] TLS verification enabled
[ ] Custom CA handled correctly
[ ] Proxy configuration understood
[ ] Authentication externalized
[ ] Secrets not hardcoded
[ ] Secrets not logged
[ ] Status codes handled
[ ] Response schema validated
[ ] JSON parsing protected
[ ] Retry policy defined
[ ] Retry-After supported
[ ] Backoff implemented
[ ] Retry methods carefully selected
[ ] Idempotency considered
[ ] Rate limits handled
[ ] Pagination bounded
[ ] Polling bounded
[ ] Correlation IDs captured
[ ] Structured logging implemented
[ ] API errors classified
[ ] Unit tests implemented
[ ] Integration tests implemented
[ ] Dependency versions controlled
[ ] Production environment protected
```

---

# 144. Interview Questions

## Q1. What is Requests?

Requests is a Python HTTP client library that provides a simple interface for communicating with HTTP/HTTPS APIs.

---

## Q2. How do you make a GET request?

```python
response = requests.get(
    url,
    timeout=10
)
```

Then validate:

```python
response.raise_for_status()
```

and process the response.

---

## Q3. Why use `json=` instead of `data=`?

`json=` is intended for sending JSON and handles JSON serialization and the appropriate content type.

---

## Q4. Why use a Session?

A Session provides:

```text
Connection reuse
Shared headers
Cookies
Authentication configuration
```

It is useful for multiple requests to the same service.

---

## Q5. What happens if you don't specify a timeout?

The request can wait unexpectedly long, potentially causing a pipeline or worker to hang.

---

## Q6. How do you handle 429?

I inspect `Retry-After`, apply bounded exponential backoff with jitter where appropriate, and reduce request concurrency.

---

## Q7. Should you retry POST?

Not blindly.

I first determine whether the operation is idempotent or whether the API supports an idempotency key. Otherwise I reconcile state before retrying.

---

## Q8. What is `raise_for_status()`?

It raises an `HTTPError` for unsuccessful HTTP status codes in the 4xx and 5xx ranges.

---

## Q9. How do you handle TLS certificates?

I keep certificate verification enabled. For internal CAs, I configure the trusted CA bundle rather than disabling verification.

---

## Q10. How do you handle large downloads?

Use:

```python
stream=True
```

and process chunks instead of loading the entire response into memory.

---

## Q11. How do you make Requests production-ready?

I would add:

```text
Session
Timeouts
Authentication
TLS
Retries
Backoff
Rate-limit handling
Structured errors
Logging
Request IDs
Response validation
Testing
```

---

## Q12. How do you separate API client and business logic?

I use layers:

```text
Requests transport
      |
      v
Generic API client
      |
      v
Service-specific client
      |
      v
Business/release orchestrator
```

This keeps the code maintainable and testable.

---

# 145. Scenario-Based Interview Questions

## Scenario 1 — API Times Out During Deployment

### Answer

First I determine whether the state-changing operation might already have succeeded.

I would:

```text
1. Capture release/operation ID
2. Check external deployment state
3. Check ArgoCD
4. Check Kubernetes
5. Only then decide whether retry is safe
```

I would not blindly send another POST.

---

## Scenario 2 — API Returns 503 Three Times

### Answer

I would use bounded retry with:

```text
Exponential backoff
Jitter
Retry-After if available
Maximum attempts
Overall deadline
```

If it remains unavailable, fail the pipeline with enough context for troubleshooting.

---

## Scenario 3 — API Returns 403

### Answer

I would verify:

```text
Identity
Token
Role
RBAC
API permissions
Environment
```

I would not keep retrying because authorization failures normally require a configuration or permission change.

---

## Scenario 4 — Internal API Certificate Is Invalid

### Answer

I would investigate:

```text
Certificate expiration
Hostname mismatch
CA trust
Proxy interception
Internal CA
```

If the organization uses an internal CA, I would configure Requests with the approved CA bundle.

I would not simply set:

```python
verify=False
```

---

## Scenario 5 — CI Runner Memory Is Exhausted

### Answer

If the automation downloads a large API response, I would check whether the code loads the entire response into memory.

For large content:

```python
stream=True
```

and chunk processing can reduce memory usage.

I would also investigate concurrency and response size.

---

## Scenario 6 — GitHub API Starts Returning 429

### Answer

I would:

```text
Inspect rate-limit headers
Reduce concurrency
Use pagination efficiently
Cache data where safe
Honor Retry-After
Use backoff
```

I would not simply add more parallel workers.

---

## Scenario 7 — Python Works Locally but Fails in Jenkins

### Answer

I would compare:

```text
Python version
Requests version
Environment variables
Proxy
NO_PROXY
DNS
Network access
CA certificates
Credentials
```

Then reproduce the same API call from the Jenkins agent using a safe curl test.

---

## Scenario 8 — HTTP 200 but Deployment Failed

### Answer

HTTP 200 only means the API request itself succeeded.

I would inspect the response body and deployment state:

```text
operation status
ArgoCD health
Kubernetes rollout
application health
```

This is a business-level failure, not necessarily an HTTP failure.

---

## Scenario 9 — POST Timed Out

### Answer

I would assume the server may have received the request.

I would:

```text
Check operation ID if available
Query current state
Check deployment history
Check server logs
```

Then determine whether another POST is safe.

---

## Scenario 10 — 204 Response Causes JSON Error

### Answer

204 means:

```text
No Content
```

So I would not call:

```python
response.json()
```

unless the API contract says a body is present.

---

# 146. Senior-Level Design Question

## Design a reusable Requests-based client for your DevOps automation platform.

### Strong Answer

I would design:

```text
                    Release Layer
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
 GitHubClient       ArgoCDClient      JenkinsClient
       |                 |                 |
       +-----------------+-----------------+
                         |
                         v
                  Generic APIClient
                         |
                  +------+------+
                  |             |
                  v             v
               Session      Error Handling
                  |
          +-------+-------+
          |       |       |
          v       v       v
       Timeout  Retry    TLS
```

The generic client would provide:

```text
Base URL
Session
Headers
Authentication
Timeouts
Retries
Backoff
Rate-limit handling
Request IDs
Logging
Error mapping
```

Service-specific clients would contain:

```text
GitHub API semantics
ArgoCD API semantics
Jenkins API semantics
```

The release orchestrator would remain independent of raw HTTP details.

---

# 147. Senior Scenario — Designing Retry Behavior

## Question

A deployment API uses:

```text
POST /deployments
```

and returns:

```text
202 Accepted
```

but sometimes the network times out.

How would you design the client?

### Strong Answer

I would not blindly retry POST.

I would first determine whether the API supports:

```text
Idempotency-Key
```

If yes:

```text
POST
+
Idempotency-Key: release-123
```

Then retrying the same operation is safe according to the API contract.

If not, I would:

```text
1. Capture release ID
2. Query deployment state
3. Determine whether the operation exists
4. Continue monitoring if it does
5. Create a new deployment only if state proves it was not accepted
```

This prevents duplicate deployments.

---

# 148. Senior Scenario — API Dependency Is Unstable

## Question

Your CI pipeline depends on an internal API that returns intermittent 502/503 errors.

What would you implement?

### Strong Answer

I would use:

```text
Connection timeout
Read timeout
Bounded retries
Exponential backoff
Jitter
Retry-After
Circuit breaker where appropriate
Metrics
Request IDs
Dependency health checks
```

I would also investigate the underlying API because retries should mitigate transient failures, not hide a systemic reliability problem.

---

# 149. Senior Scenario — Production API Security

## Question

How would you secure a Python automation client?

### Strong Answer

I would use:

```text
HTTPS
TLS verification
OIDC/short-lived credentials where possible
Least-privilege API tokens
Secret manager/CI secret store
No secrets in source
No secrets in logs
Input validation
SSRF protection for user-supplied URLs
Rate limiting
Audit logging
Dependency security scanning
```

---

# 150. Production Architecture

```text
                         GitHub / Jenkins
                                |
                                v
                       Python Release Tool
                                |
                     +----------+----------+
                     |                     |
                     v                     v
              Configuration          Secret Provider
                     |                     |
                     +----------+----------+
                                |
                                v
                         Generic API Client
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
             Requests Session          Error Handler
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      TLS         Retry      Timeout
        |           |           |
        +-----------+-----------+
                    |
                    v
              External APIs
          /       |       |       \
         v        v       v        v
     GitHub    Jenkins  ArgoCD    AWS
                                  |
                                  v
                                 ECR
                                  |
                                  v
                                 EKS
                                  |
                         +--------+--------+
                         |                 |
                         v                 v
                    Prometheus            ELK
```

---

# 151. End-to-End DevOps API Automation

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins / GitHub Actions
    |
    v
Python
    |
    +-- GET repository
    |
    +-- POST build
    |
    +-- GET build status
    |
    +-- GET security result
    |
    +-- POST/update release
    |
    +-- GET artifact
    |
    +-- update GitOps
    |
    +-- GET ArgoCD status
    |
    +-- GET Kubernetes state
    |
    +-- GET Prometheus metrics
    |
    +-- GET Elasticsearch logs
    |
    v
Release Decision
```

This is the core reason API knowledge is important for DevOps engineers.

---

# 152. Production Rules to Remember

### Rule 1

Always configure timeouts.

### Rule 2

Never hardcode secrets.

### Rule 3

Keep TLS verification enabled.

### Rule 4

Do not retry blindly.

### Rule 5

Treat POST carefully.

### Rule 6

Use sessions for repeated requests.

### Rule 7

Respect rate limits.

### Rule 8

Validate response content, not only status codes.

### Rule 9

Use bounded polling.

### Rule 10

Separate transport from business logic.

### Rule 11

Log request context, not secrets.

### Rule 12

Reconcile state after uncertain failures.

---

# 153. Final Mental Model

Think about Requests as this layer:

```text
                 DevOps Automation
                        |
                        v
              Business / Release Logic
                        |
                        v
                 Service Client
                        |
                        v
                  API Client
                        |
                        v
              requests.Session()
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Headers       Timeout        Retry
          |             |             |
          +-------------+-------------+
                        |
                        v
                       TLS
                        |
                        v
                      HTTP
                        |
                        v
                 External Service
```

The Requests library is not the entire automation solution.

It is the reliable HTTP transport foundation underneath your DevOps API integrations.

---

# 154. What You Should Be Able to Explain in an Interview

You should be comfortable explaining:

```text
How to install Requests
How to make GET/POST/PUT/PATCH/DELETE
json= vs data=
params=
headers=
Session
Timeout
Connect vs read timeout
raise_for_status()
HTTP exceptions
Retry strategy
Exponential backoff
Jitter
Retry-After
Rate limiting
TLS verification
Custom CA
Proxy/NO_PROXY
Authentication headers
Pagination
Polling
Streaming
Connection pooling
Response validation
Request IDs
Structured logging
Testing
Mocking
API client architecture
Idempotency
POST retry risk
SSRF
Production troubleshooting
```

Most importantly:

> **Requests makes HTTP communication easy; production engineering makes that communication safe, observable, bounded, and reliable.**

---

# 155. Next Topic

The next file in this section is:

```text
08-Python-APIs/
├── 01-HTTP-and-REST.md       ✓
├── 02-Requests-Library.md    ✓
├── 03-API-Automation.md
├── 04-Authentication.md
├── 05-API-Error-Handling.md
└── 06-DevOps-API-Projects.md
```

The next topic will build directly on this file:

# `03-API-Automation.md`

It will move from learning the Requests library to designing **real Python API automation workflows across GitHub, Jenkins, AWS, ECR, Kubernetes, ArgoCD, Prometheus, Elasticsearch, and CI/CD systems**.
