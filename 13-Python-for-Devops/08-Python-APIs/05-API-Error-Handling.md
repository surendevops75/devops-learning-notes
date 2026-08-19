# 05 — Python API Error Handling for DevOps Engineers

## 1. Overview

The previous files covered:

```text
01 — HTTP and REST
02 — Requests Library
03 — API Automation
04 — Authentication
```

Now we focus on one of the most important production topics:

> **How should Python API automation behave when things go wrong?**

In real DevOps environments, failures are normal.

APIs can return:

```text
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
```

Networks can experience:

```text
DNS failures
Connection resets
Timeouts
TLS failures
Proxy failures
```

Distributed workflows can encounter:

```text
Duplicate requests
Partial failures
Lost responses
Expired credentials
Rate limits
Race conditions
Eventual consistency
```

A production Python automation tool must not simply:

```python
try:
    ...
except:
    retry()
```

It needs a deliberate failure strategy.

The production mental model is:

```text
Detect
  ↓
Classify
  ↓
Decide
  ↓
Retry / Reconcile / Fail / Rollback
  ↓
Observe
  ↓
Recover
```

---

# 2. Why Error Handling Matters in DevOps

Consider a deployment workflow:

```text
Build
  ↓
Security Scan
  ↓
Push Image
  ↓
Update Git
  ↓
ArgoCD Sync
  ↓
Kubernetes Rollout
  ↓
Health Verification
```

Any stage can fail.

The dangerous situation is not only:

```text
Failure
```

It is:

```text
Unknown state
```

Example:

```text
POST /deploy
       ↓
Server receives request
       ↓
Deployment starts
       ↓
Network connection breaks
       ↓
Python receives timeout
```

Python cannot automatically conclude:

```text
Deployment failed
```

The correct state is:

```text
Deployment outcome = UNKNOWN
```

Then Python must reconcile the external system.

---

# 3. Error Handling Goals

A production API client should provide:

```text
Correctness
Reliability
Security
Observability
Bounded execution
Recoverability
Clear failure reporting
```

It should avoid:

```text
Infinite retries
Duplicate operations
Silent failures
Credential leakage
Pipeline hangs
False success
False failure
```

---

# 4. Error Categories

Classify errors into major categories:

```text
1. Input errors
2. Authentication errors
3. Authorization errors
4. Resource errors
5. Conflict errors
6. Rate-limit errors
7. Network errors
8. Timeout errors
9. TLS errors
10. Server errors
11. Parsing/schema errors
12. Business/application errors
13. State/reconciliation errors
```

Different categories require different actions.

---

# 5. Error Handling Decision Tree

```text
API operation
     |
     v
Response/error
     |
     v
Classify
     |
     +---- Input ---------> Fix input
     |
     +---- 401 -----------> Re-authenticate
     |
     +---- 403 -----------> Fix permission
     |
     +---- 404 -----------> Validate resource
     |
     +---- 409 -----------> Reconcile
     |
     +---- 429 -----------> Backoff/retry
     |
     +---- 5xx -----------> Retry if safe
     |
     +---- Timeout --------> Reconcile if write
     |
     +---- Network --------> Bounded retry
     |
     +---- Parse ----------> Fail safely
     |
     +---- Business -------> Follow policy
```

---

# 6. HTTP Status Classes

```text
1xx = informational
2xx = success
3xx = redirection
4xx = client/request/auth problem
5xx = server/dependency problem
```

For DevOps automation, the most important are:

```text
2xx
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
```

---

# 7. 400 Bad Request

Usually means:

```text
Invalid request syntax
Invalid parameters
Malformed payload
```

Example:

```json
{
  "environment": ""
}
```

Possible response:

```text
400 Bad Request
```

Normally:

```text
Do not retry
```

Fix the request.

---

# 8. 401 Unauthorized

Usually indicates an authentication problem:

```text
Missing token
Expired token
Invalid token
Wrong credentials
Wrong token audience
```

Action:

```text
Refresh/re-authenticate if supported
```

Do not retry the same invalid credential indefinitely.

---

# 9. 403 Forbidden

Authentication may have succeeded, but authorization failed.

Check:

```text
RBAC
Scopes
IAM
Role
Repository permission
Environment permission
ArgoCD RBAC
Kubernetes RBAC
```

Normally:

```text
Do not retry repeatedly
```

---

# 10. 404 Not Found

Meaning depends on the operation.

For:

```text
GET /users/123
```

404 may be expected.

For:

```text
GET /production/deployment
```

404 may be a critical state problem.

The service/client layer should expose the response and the business layer should decide whether 404 is expected.

---

# 11. 409 Conflict

409 is especially important in automation.

It can mean:

```text
Resource already exists
Concurrent update
State conflict
Operation already running
Version conflict
```

The correct response may be:

```text
GET current state
```

rather than:

```text
Retry immediately
```

---

# 12. 409 Example

Suppose:

```text
POST /deployments
```

returns:

```text
409 Conflict
```

Possible meaning:

```text
Deployment already exists
```

Automation should ask:

```text
Does the desired deployment already exist?
```

If yes:

```text
Continue
```

If no:

```text
Investigate conflict
```

---

# 13. 422 Unprocessable Content

Usually indicates:

```text
Request structure is valid
but values violate business validation
```

Example:

```json
{
  "replicas": -5
}
```

The server understands the request but rejects its content.

Normally:

```text
Do not retry
```

---

# 14. 429 Too Many Requests

Means the client is being rate limited.

Possible headers:

```text
Retry-After
X-RateLimit-Remaining
X-RateLimit-Reset
```

Automation should:

```text
Reduce request rate
Honor Retry-After
Backoff
Add jitter
Limit concurrency
```

---

# 15. 500 Internal Server Error

The server encountered an unexpected condition.

Often:

```text
Retry may be appropriate
```

but not always.

Before retrying a state-changing operation, determine:

```text
Did the server receive the request?
```

---

# 16. 502 Bad Gateway

Often means:

```text
Gateway/proxy
   ↓
Backend
```

communication failed.

Investigate:

```text
ALB
Ingress
Reverse proxy
Upstream service
DNS
Network
```

Transient retry may be appropriate.

---

# 17. 503 Service Unavailable

Usually indicates temporary unavailability.

Examples:

```text
Service restarting
Overloaded
Maintenance
No healthy backend
```

Often retryable with:

```text
Backoff
Jitter
Retry-After
Deadline
```

---

# 18. 504 Gateway Timeout

A gateway waited too long for the upstream service.

Important:

```text
The upstream may still have processed the request.
```

For GET:

```text
Retry is usually simpler.
```

For POST:

```text
Reconcile before retrying.
```

---

# 19. HTTP Error Handling with Requests

Basic:

```python
response = session.get(
    url,
    timeout=10
)

response.raise_for_status()
```

This is useful but not enough for production orchestration.

You need classification.

---

# 20. `raise_for_status()`

It raises:

```text
requests.exceptions.HTTPError
```

for unsuccessful 4xx/5xx responses.

Example:

```python
try:
    response.raise_for_status()
except requests.exceptions.HTTPError as exc:
    ...
```

But your automation still needs to determine:

```text
401?
403?
409?
429?
503?
```

---

# 21. Generic API Error

Create a custom exception:

```python
class APIError(Exception):

    def __init__(
        self,
        message,
        status_code=None,
        request_id=None,
        response_body=None
    ):
        super().__init__(message)

        self.status_code = status_code
        self.request_id = request_id
        self.response_body = response_body
```

Be careful not to store sensitive response content unnecessarily.

---

# 22. Specific Exceptions

Example:

```python
class AuthenticationError(APIError):
    pass


class AuthorizationError(APIError):
    pass


class RateLimitError(APIError):
    pass


class ConflictError(APIError):
    pass


class TransientAPIError(APIError):
    pass
```

Now higher-level workflows can react appropriately.

---

# 23. Central Response Handler

Example:

```python
def handle_response(response):

    if response.status_code == 401:
        raise AuthenticationError(
            "Authentication failed",
            status_code=401
        )

    if response.status_code == 403:
        raise AuthorizationError(
            "Authorization failed",
            status_code=403
        )

    if response.status_code == 409:
        raise ConflictError(
            "Resource conflict",
            status_code=409
        )

    if response.status_code == 429:
        raise RateLimitError(
            "Rate limit exceeded",
            status_code=429
        )

    response.raise_for_status()
```

---

# 24. Preserve Request IDs

When an API returns:

```text
X-Request-ID
```

capture it:

```python
request_id = response.headers.get(
    "X-Request-ID"
)
```

Include it in your structured error.

This is extremely useful when working with:

```text
Platform teams
API owners
Cloud support
```

---

# 25. Safe Error Messages

Bad:

```python
raise RuntimeError(
    f"Request failed: {response.text}"
)
```

The body could contain:

```text
Secrets
Tokens
Internal data
PII
```

Better:

```python
raise APIError(
    "Deployment API request failed",
    status_code=response.status_code,
    request_id=request_id
)
```

Log sanitized diagnostic information separately.

---

# 26. Connection Errors

Example:

```python
try:
    response = session.get(
        url,
        timeout=10
    )

except requests.exceptions.ConnectionError:
    ...
```

Possible causes:

```text
DNS
Network
Firewall
Proxy
Service down
Connection refused
Connection reset
```

---

# 27. Timeout Errors

```python
except requests.exceptions.Timeout:
    ...
```

Possible causes:

```text
Slow server
Network congestion
Overloaded backend
Incorrect timeout
Gateway timeout
```

For a GET:

```text
Retry may be safe.
```

For a state-changing POST:

```text
Reconcile first.
```

---

# 28. TLS Errors

Requests may raise:

```text
requests.exceptions.SSLError
```

Possible causes:

```text
Expired certificate
Untrusted CA
Hostname mismatch
TLS version mismatch
Proxy interception
Incorrect certificate chain
```

Never solve TLS failures by defaulting to:

```python
verify=False
```

---

# 29. DNS Errors

DNS failures often appear as connection-related errors.

Troubleshooting:

```bash
nslookup api.example.com
dig api.example.com
```

Also check:

```text
/etc/resolv.conf
CoreDNS
VPC DNS
Private hosted zones
```

depending on environment.

---

# 30. Proxy Errors

Enterprise environments may use:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

A proxy failure can look like:

```text
ConnectionError
Timeout
502
```

Compare:

```text
Local environment
CI runner
Kubernetes pod
```

because network paths may differ.

---

# 31. Network Troubleshooting Layers

When API calls fail:

```text
DNS
 ↓
Route
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Authentication
 ↓
Authorization
 ↓
Application
```

Troubleshoot from the lowest failing layer.

---

# 32. Retry Fundamentals

Retrying means:

```text
Attempt
 ↓
Temporary failure
 ↓
Wait
 ↓
Attempt again
```

A production retry policy must define:

```text
What to retry
How many times
How long to wait
Which methods
Which status codes
Overall deadline
```

---

# 33. Do Not Retry Everything

Bad:

```python
while True:
    try:
        call()
        break
    except Exception:
        time.sleep(1)
```

Problems:

```text
Infinite loop
Pipeline hang
Duplicate writes
API overload
Hidden incidents
```

---

# 34. Retryable Errors

Common candidates:

```text
ConnectionError
Timeout
429
502
503
504
```

Only when:

```text
Retry is safe
```

---

# 35. Usually Non-Retryable Errors

Usually:

```text
400
401
403
404
422
```

because the problem normally requires:

```text
Input correction
Credential correction
Permission correction
Resource correction
```

---

# 36. Conditional Retry

A more accurate model:

```text
Error
 |
 v
Is it transient?
 |
 +-- No --> Fail
 |
 +-- Yes
      |
      v
Is operation retry-safe?
      |
      +-- No --> Reconcile
      |
      +-- Yes -> Retry
```

---

# 37. Exponential Backoff

Example:

```text
Attempt 1
wait 1 sec

Attempt 2
wait 2 sec

Attempt 3
wait 4 sec

Attempt 4
wait 8 sec
```

This reduces pressure on an unhealthy API.

---

# 38. Maximum Backoff

Do not allow:

```text
1
2
4
8
16
32
64
128
256
...
```

forever.

Use:

```text
max_delay = 30 sec
```

Conceptually:

```python
delay = min(
    base * 2 ** attempt,
    max_delay
)
```

---

# 39. Jitter

Without jitter:

```text
100 clients
   |
   v
all retry at 10 sec
   |
   v
API overload
```

With jitter:

```text
Client A -> 8.7 sec
Client B -> 10.2 sec
Client C -> 11.1 sec
```

This reduces synchronized retry spikes.

---

# 40. Retry-After

If the server says:

```http
Retry-After: 20
```

the client should generally honor it when appropriate.

Example:

```python
retry_after = response.headers.get(
    "Retry-After"
)
```

Do not blindly ignore server guidance.

---

# 41. Rate Limit Headers

Some APIs expose:

```text
X-RateLimit-Limit
X-RateLimit-Remaining
X-RateLimit-Reset
```

Use them to understand:

```text
Capacity
Remaining quota
Reset timing
```

Exact headers vary by API.

---

# 42. Retry Budget

Suppose:

```text
Deployment deadline = 5 minutes
```

A retry strategy must fit inside that deadline.

Bad:

```text
5 retries
+
60-second delay each
+
10-minute polling
```

The workflow could exceed its intended runtime.

---

# 43. Overall Deadline

Use:

```python
import time

deadline = (
    time.monotonic()
    + 300
)
```

Then before every attempt:

```python
remaining = (
    deadline - time.monotonic()
)
```

Stop when:

```text
remaining <= 0
```

---

# 44. Monotonic Time

For elapsed time, prefer:

```python
time.monotonic()
```

instead of:

```python
time.time()
```

because wall-clock time can change.

---

# 45. Retry Example

Conceptual:

```python
for attempt in range(4):

    try:
        response = session.get(
            url,
            timeout=10
        )

        if response.status_code in {
            429,
            502,
            503,
            504
        }:
            raise TransientAPIError()

        response.raise_for_status()

        return response

    except (
        requests.exceptions.Timeout,
        requests.exceptions.ConnectionError
    ):
        if attempt == 3:
            raise

        sleep_with_backoff(attempt)
```

Production implementation should also respect:

```text
deadline
Retry-After
jitter
method safety
```

---

# 46. Retry POST Carefully

Example:

```text
POST /deploy
```

Timeout occurs.

Do not immediately retry.

The server may have:

```text
received request
started deployment
```

The client simply did not receive the response.

---

# 47. Reconciliation Before Retry

Correct:

```text
POST timeout
    |
    v
Check deployment ID
    |
    +-- exists/running -> monitor
    |
    +-- succeeded -> continue
    |
    +-- failed -> evaluate retry
    |
    +-- does not exist -> retry if safe
```

---

# 48. Idempotency Key

If supported:

```http
Idempotency-Key: release-12345
```

Then:

```text
POST
 |
 X timeout
 |
 v
Retry same key
 |
 v
Server recognizes same operation
```

This can prevent duplicate creation.

The server must explicitly support the semantics.

---

# 49. Idempotency vs Retry

These are related but different.

### Retry

```text
Try operation again
```

### Idempotency

```text
Repeating operation does not create an unintended additional effect
```

Safe automation often needs both.

---

# 50. HTTP Method Idempotency

Generally:

```text
GET    -> idempotent
PUT    -> idempotent
DELETE -> intended to be idempotent
POST   -> not necessarily idempotent
PATCH  -> depends on operation
```

Always follow the actual API contract.

---

# 51. Retry Matrix

| Error | GET | PUT | DELETE | POST |
|---|---:|---:|---:|---:|
| Timeout | Retry | Usually retry | Usually retry | Reconcile |
| 429 | Retry | Retry if safe | Retry if safe | Retry only if safe |
| 502 | Retry | Retry if safe | Retry if safe | Reconcile |
| 503 | Retry | Retry if safe | Retry if safe | Reconcile |
| 504 | Retry | Retry if safe | Retry if safe | Reconcile |
| 401 | Refresh auth | Refresh auth | Refresh auth | Refresh auth |
| 403 | No | No | No | No |
| 400 | No | No | No | No |
| 422 | No | No | No | No |

The table is a decision aid, not a substitute for the API contract.

---

# 52. Circuit Breaker

A circuit breaker prevents repeated calls to an unhealthy dependency.

States:

```text
CLOSED
   |
   | failures exceed threshold
   v
OPEN
   |
   | wait
   v
HALF-OPEN
   |
   +-- success -> CLOSED
   |
   +-- failure -> OPEN
```

---

# 53. Why Circuit Breakers Matter

Suppose:

```text
Python
 |
 v
Broken API
```

Without circuit breaker:

```text
1000 requests
```

continue failing.

With circuit breaker:

```text
Failures
 |
 v
OPEN
 |
 v
Stop sending traffic temporarily
```

This protects:

```text
Client
API
Network
CI runner
```

---

# 54. Circuit Breaker in CI/CD

For short-lived CI jobs, a simple:

```text
bounded retry + deadline
```

may be enough.

For long-running automation services:

```text
circuit breaker
```

can be more useful.

Do not add complexity without a requirement.

---

# 55. Bulkhead Pattern

Bulkheads isolate failures.

Example:

```text
GitHub calls
   |
   +-- pool A

ArgoCD calls
   |
   +-- pool B

Prometheus calls
   |
   +-- pool C
```

If Prometheus becomes slow, it should not consume all workers needed for ArgoCD.

---

# 56. Connection Pool Exhaustion

Too much concurrency can exhaust:

```text
Connection pool
File descriptors
Threads
CPU
Memory
```

Use bounded concurrency and appropriate pool settings.

---

# 57. Fallbacks

A fallback is an alternative behavior when a dependency fails.

Example:

```text
Prometheus unavailable
 |
 v
Continue with Kubernetes + health endpoint?
```

Whether this is acceptable depends on release policy.

For security gates:

```text
Scanner unavailable
```

may need:

```text
Fail closed
```

rather than bypassing the scan.

---

# 58. Fail Open vs Fail Closed

### Fail open

Continue when dependency fails.

Example:

```text
Optional notification service unavailable
```

### Fail closed

Stop when dependency fails.

Example:

```text
Mandatory security scan unavailable
```

Choose based on risk.

---

# 59. Critical vs Non-Critical Dependencies

Example:

```text
Mandatory:
ArgoCD
Kubernetes
Security gate

Optional:
Slack notification
Secondary dashboard
```

Failure handling should differ.

---

# 60. Partial Failure

A workflow may succeed in some systems and fail in others.

Example:

```text
ECR push = success
Git commit = success
ArgoCD sync = failure
```

This is not a simple transaction rollback.

You need:

```text
Current state
Known good state
Compensation
Recovery
```

---

# 61. Distributed Transaction Problem

A release can span:

```text
GitHub
Jenkins
ECR
Git
ArgoCD
Kubernetes
Prometheus
ELK
```

There is no single transaction covering all systems.

Therefore use:

```text
State machine
+
Idempotency
+
Reconciliation
+
Compensation
```

---

# 62. Compensation

Suppose:

```text
New deployment
 |
 v
Health check fails
```

Compensation could be:

```text
Restore previous Git revision
```

Then:

```text
ArgoCD
 |
 v
Previous known-good version
```

Compensation is domain-specific.

---

# 63. Rollback Decision

Do not automatically rollback every error.

Examples:

```text
Notification failed
=
Probably no rollback

Health check failed
=
Rollback may be appropriate

Security policy failed
=
Stop before deployment

ArgoCD unavailable
=
Usually wait/retry rather than rollback
```

---

# 64. Error Handling for Long-Running Operations

Example:

```text
POST deployment
 |
 v
202 Accepted
 |
 v
operation_id
 |
 v
poll
 |
 +-- running
 |
 +-- succeeded
 |
 +-- failed
 |
 +-- cancelled
 |
 +-- deadline
```

Each state requires different handling.

---

# 65. Polling Error Handling

Do not:

```python
while True:
    response = session.get(...)
    time.sleep(5)
```

Use:

```text
Deadline
Retry policy
Failure state
Backoff
Jitter
```

---

# 66. Polling Example

Conceptual:

```python
deadline = (
    time.monotonic()
    + 600
)

while True:

    if time.monotonic() >= deadline:
        raise TimeoutError(
            "Deployment deadline exceeded"
        )

    response = session.get(
        status_url,
        timeout=10
    )

    if response.status_code == 429:
        wait_for_rate_limit(response)
        continue

    response.raise_for_status()

    state = response.json()["status"]

    if state == "succeeded":
        return

    if state == "failed":
        raise RuntimeError(
            "Deployment failed"
        )

    sleep_with_jitter()
```

---

# 67. Parsing Errors

A response may be:

```text
HTTP 200
```

but contain invalid JSON.

Example:

```python
try:
    data = response.json()
except ValueError:
    raise APIError(
        "Invalid JSON response"
    )
```

---

# 68. Content-Type Validation

Before parsing JSON, check:

```python
content_type = response.headers.get(
    "Content-Type",
    ""
)
```

Expected:

```text
application/json
```

But do not depend solely on Content-Type if the provider is known to misconfigure it; use safe parsing and explicit contract validation.

---

# 69. Schema Errors

Response:

```json
{
  "state": "healthy"
}
```

Automation expects:

```python
data["status"]
```

This can cause:

```text
KeyError
```

Handle schema changes explicitly.

---

# 70. Defensive Parsing

Example:

```python
status = data.get("status")

if status is None:
    raise APIError(
        "Required field 'status' missing"
    )
```

For complex APIs, use schema/model validation.

---

# 71. Business Errors

HTTP can be:

```text
200
```

while the application reports:

```json
{
  "status": "failed",
  "reason": "migration_error"
}
```

This is a:

```text
Business/application error
```

not an HTTP transport error.

---

# 72. Business Error Handling

```python
data = response.json()

if data.get("status") == "failed":
    raise DeploymentError(
        data.get("reason", "Unknown failure")
    )
```

Do not declare success only because:

```text
HTTP = 200
```

---

# 73. Error Classification Layers

Use:

```text
Transport Error
    |
    v
HTTP Error
    |
    v
API Error
    |
    v
Business Error
    |
    v
Workflow Error
```

Example:

```text
TCP timeout
   ↓
Request exception
   ↓
Unknown deployment state
   ↓
Reconciliation required
   ↓
Release workflow paused
```

---

# 74. Exception Chaining

Python supports exception chaining:

```python
try:
    response = session.get(
        url,
        timeout=10
    )
except requests.exceptions.Timeout as exc:
    raise APIError(
        "Deployment API timed out"
    ) from exc
```

This preserves the original cause.

---

# 75. Why Exception Chaining Matters

Without chaining:

```text
APIError
```

may hide:

```text
Timeout
```

With chaining:

```text
APIError
  caused by
Timeout
```

This improves debugging.

---

# 76. Avoid Bare `except`

Bad:

```python
except:
    pass
```

This hides:

```text
KeyboardInterrupt
SystemExit
Programming bugs
Operational failures
```

Use specific exceptions.

---

# 77. Avoid Silent Failure

Bad:

```python
try:
    deploy()
except Exception:
    return False
```

This loses:

```text
Cause
Status
Request ID
Context
```

Better:

```text
Raise structured error
+
log safely
+
return explicit failure
```

---

# 78. Error Context

A useful error includes:

```text
Service
Environment
Operation
Endpoint/path
Status code
Request ID
Release ID
Attempt number
Elapsed time
```

Example:

```text
ArgoCD sync failed
service=payment
environment=prod
status=503
attempt=3
request_id=req-123
release_id=rel-456
```

---

# 79. Structured Error Object

Example:

```python
from dataclasses import dataclass


@dataclass
class ErrorContext:

    service: str
    operation: str
    environment: str
    status_code: int | None
    request_id: str | None
    attempt: int
```

This can be attached to structured exceptions/logs.

---

# 80. Logging Errors

Log:

```text
error_type
service
operation
environment
status
request_id
duration
attempt
```

Do not log:

```text
token
password
private key
full Authorization header
```

---

# 81. Correlation ID

Use a release ID:

```text
rel-2026-001
```

through the entire workflow.

Example:

```text
Jenkins
 |
 v
Python
 |
 v
GitHub
 |
 v
ArgoCD
 |
 v
Kubernetes
```

All logs can reference:

```text
release_id=rel-2026-001
```

---

# 82. Request ID vs Correlation ID

### Request ID

Identifies one API request.

```text
req-123
```

### Correlation/Release ID

Identifies the broader workflow.

```text
rel-456
```

Use both when possible.

---

# 83. Error Metrics

Track:

```text
api_errors_total
api_timeouts_total
api_retries_total
api_rate_limits_total
api_auth_failures_total
api_server_errors_total
deployments_failed_total
```

Break down by:

```text
service
endpoint
environment
status_code
```

---

# 84. Error Rate

Example:

```text
Error rate =
failed requests / total requests
```

But also track:

```text
Timeout rate
429 rate
5xx rate
Authentication failures
```

Different errors require different action.

---

# 85. Latency Metrics

Track:

```text
p50
p95
p99
```

Example:

```text
p50 = 100ms
p95 = 2s
p99 = 20s
```

The average may hide serious tail latency.

---

# 86. Retry Metrics

Track:

```text
retry count
retry reason
retry endpoint
retry success
retry exhaustion
```

If retries are frequently exhausted:

```text
Dependency reliability problem
```

may exist.

---

# 87. Alerting

Good alerts:

```text
ArgoCD API 5xx > threshold
Deployment failure rate > threshold
Authentication failures spike
429 rate increases
Automation latency exceeds SLO
```

Bad alert:

```text
One transient 503
```

Avoid alert noise.

---

# 88. Error Budget

If automation has an SLO:

```text
99.5% successful deployments
```

the remaining:

```text
0.5%
```

is an error budget.

Use it to understand whether reliability work is needed.

---

# 89. Circuit Breaker Metrics

Monitor:

```text
circuit_state
open_count
half_open_attempts
failure_count
```

This helps determine whether a dependency is consistently unhealthy.

---

# 90. Retry Storm

A retry storm happens when many clients retry aggressively.

Example:

```text
API fails
 |
 v
100 clients retry
 |
 v
API overloaded
 |
 v
More failures
 |
 v
More retries
```

Break the cycle with:

```text
Backoff
Jitter
Rate limits
Circuit breaker
Concurrency control
```

---

# 91. Thundering Herd

A related problem:

```text
Many workers wake up simultaneously
```

and call the same API.

Common in:

```text
Polling
Retries
Cache expiry
Scheduled jobs
```

Use:

```text
Jitter
Staggering
Backoff
```

---

# 92. Retry Amplification

Suppose:

```text
100 jobs
x
5 retries
=
500 requests
```

A temporary failure can become a huge traffic spike.

Always consider the multiplication effect.

---

# 93. Concurrency + Retry Interaction

Example:

```text
20 workers
x
4 retries
=
potentially 80 attempts
```

Control:

```text
worker count
retry count
backoff
API rate
```

together.

---

# 94. Retry Only What Is Needed

Avoid retrying:

```text
Entire workflow
```

when only:

```text
One API request
```

failed.

Prefer:

```text
Retry failed operation
```

provided it is safe.

---

# 95. Resume From Checkpoint

Example:

```text
Build ✓
Scan ✓
Push ✓
Git update ✓
ArgoCD sync ✗
```

Restart should ideally continue from:

```text
ArgoCD sync
```

rather than:

```text
Build
```

This requires explicit workflow state.

---

# 96. Checkpoint Design

Store:

```text
release_id
current_stage
commit
image_digest
environment
operation_id
timestamps
```

Possible stores:

```text
Database
DynamoDB
S3
CI metadata
Git
Approved workflow store
```

Choose according to system requirements.

---

# 97. State Machine Example

```text
VALIDATE
   |
   v
BUILD
   |
   v
SCAN
   |
   v
PUBLISH
   |
   v
GITOPS
   |
   v
SYNC
   |
   v
ROLLOUT
   |
   v
VERIFY
   |
   v
SUCCESS
```

Failure transitions:

```text
Any stage
   |
   +-- retry
   |
   +-- reconcile
   |
   +-- rollback
   |
   +-- failed
```

---

# 98. Error Handling and GitOps

Suppose:

```text
Git update succeeded
ArgoCD unavailable
```

Do not revert Git simply because ArgoCD is temporarily unavailable.

Desired state may already be correct.

Better:

```text
Git = desired state
ArgoCD = temporarily unavailable
```

Wait/retry ArgoCD and verify.

---

# 99. Error Handling and Kubernetes

Suppose:

```text
ArgoCD = Synced
Kubernetes = rollout timeout
```

Investigate:

```text
Pods
Events
ReplicaSet
Probes
Resources
Image
Secrets
ConfigMaps
Node health
```

Do not automatically blame ArgoCD.

---

# 100. Error Handling and Observability

If deployment is:

```text
Kubernetes Ready
```

but:

```text
Prometheus error rate high
```

the application may still be unhealthy.

Use multiple signals.

---

# 101. Fail-Fast Validation

Before making expensive operations:

```text
Validate environment
Validate service
Validate version
Validate permissions
Validate dependencies
```

Then:

```text
Build
```

This prevents wasting CI time.

---

# 102. Preflight Checks

Example:

```text
GitHub reachable
Jenkins reachable
ECR accessible
ArgoCD reachable
Kubernetes accessible
Monitoring accessible
```

Fail early for mandatory dependencies.

---

# 103. Dependency Classification

Create:

```text
Critical dependencies
Optional dependencies
```

Example:

```text
Critical:
Git
ECR
ArgoCD
Kubernetes

Optional:
Slack
Secondary dashboards
```

Different failures require different policies.

---

# 104. Graceful Degradation

If:

```text
Slack API fails
```

deployment may still succeed.

If:

```text
ArgoCD fails
```

deployment may need to stop.

If:

```text
Prometheus fails
```

policy determines whether deployment can continue.

---

# 105. Fail-Closed Security Controls

Security-sensitive gates should normally fail closed.

Example:

```text
Security scanner unavailable
```

Instead of:

```text
Skip scanner
Deploy
```

prefer:

```text
Stop deployment
Investigate
```

unless an explicitly approved emergency process exists.

---

# 106. Error Handling Around Security

Never:

```python
try:
    security_scan()
except Exception:
    pass
```

This can create a security bypass.

Use:

```text
Explicit exception
+
policy decision
+
audit
```

---

# 107. Authentication Error Handling

For:

```text
401
```

possible flow:

```text
401
 |
 v
Is token expired?
 |
 +-- Yes -> refresh
 |
 +-- No -> fail
```

Do not retry invalid credentials indefinitely.

---

# 108. Authorization Error Handling

For:

```text
403
```

flow:

```text
403
 |
 v
Check permissions
 |
 v
Fail
```

A retry normally will not change permissions.

---

# 109. Rate-Limit Error Handling

For:

```text
429
```

flow:

```text
429
 |
 v
Read Retry-After
 |
 v
Backoff
 |
 v
Retry
```

Also consider:

```text
Reduce concurrency
```

---

# 110. Conflict Error Handling

For:

```text
409
```

flow:

```text
409
 |
 v
Read current state
 |
 v
Desired state already exists?
 |
 +-- Yes -> continue
 |
 +-- No -> resolve conflict
```

---

# 111. Server Error Handling

For:

```text
503
```

flow:

```text
503
 |
 v
Is operation read-only?
 |
 +-- Yes -> retry
 |
 +-- No -> determine whether request may have succeeded
              |
              v
          reconcile
```

---

# 112. Timeout Handling

Timeout should produce:

```text
Unknown
```

for many writes.

Not automatically:

```text
Failed
```

This distinction is critical.

---

# 113. Network Partition

A network partition can create:

```text
Client cannot see server
```

while:

```text
Server continues processing
```

Therefore:

```text
Timeout
=
client-side uncertainty
```

not necessarily server-side failure.

---

# 114. Exactly-Once Is Difficult

In distributed systems, exactly-once execution across independent APIs is difficult.

Prefer:

```text
At-least-once attempts
+
Idempotency
+
Reconciliation
```

This is a core DevOps/distributed-systems concept.

---

# 115. Duplicate Request Protection

Methods:

```text
Idempotency key
Unique operation ID
Database uniqueness
Conditional writes
ETag / If-Match
Distributed locks
```

Use whichever mechanism the target API supports.

---

# 116. Optimistic Concurrency

Some APIs support:

```http
If-Match: <etag>
```

Flow:

```text
GET resource
 |
 v
ETag = abc
 |
 v
Modify
 |
 v
PUT + If-Match: abc
```

If another process changed it:

```text
412 Precondition Failed
```

The client can re-read state and retry safely.

---

# 117. Compare-and-Swap

Concept:

```text
Update only if current state = expected state
```

This prevents overwriting another automation run's changes.

---

# 118. Distributed Locks

For production deployments:

```text
Environment lock
```

can prevent:

```text
Run A + Run B
```

from changing the same environment simultaneously.

Use a persistent lock service for multi-runner systems.

---

# 119. Lock Failure Handling

A lock system can fail too.

Never create:

```text
permanent lock
```

without:

```text
TTL
owner
renewal
safe release
```

where appropriate.

---

# 120. Deadlocks

Avoid:

```text
Run A holds lock 1
waits for lock 2

Run B holds lock 2
waits for lock 1
```

Keep lock ordering simple.

---

# 121. Error Handling in Parallel Workflows

Suppose:

```text
service A = success
service B = success
service C = failure
service D = running
```

The orchestrator needs a policy:

```text
Stop all?
Continue independent work?
Cancel remaining?
Rollback successful services?
```

Do not leave this behavior undefined.

---

# 122. Fail-Fast vs Continue

Use fail-fast when:

```text
A dependency is mandatory
```

Continue when:

```text
Operations are independent
```

Example:

```text
Deploy 10 independent staging services
```

could continue after one failure if the goal is to report all results.

Production strategy depends on risk.

---

# 123. Aggregating Errors

For parallel operations:

```python
results = {
    "payment": "success",
    "catalog": "success",
    "orders": "failed"
}
```

Then report:

```text
2 succeeded
1 failed
```

Do not stop at the first exception if comprehensive results are required.

---

# 124. Exception Groups

Modern Python can represent multiple failures using exception groups.

Conceptually:

```python
raise ExceptionGroup(
    "Deployment failures",
    failures
)
```

Useful for parallel automation.

Use only when the application's Python version supports the required language features.

---

# 125. Error Handling in CI/CD

CI should clearly distinguish:

```text
SUCCESS
FAILURE
ABORTED
TIMEOUT
```

Map Python failures to non-zero exit codes.

Example:

```python
raise SystemExit(1)
```

---

# 126. Exit Code Strategy

Example:

```text
0 = success

1 = general failure
2 = validation error
3 = authentication error
4 = authorization error
5 = timeout
6 = dependency failure
```

Only define custom codes if they provide real value and are documented.

---

# 127. Error Messages for Developers

Good:

```text
ArgoCD sync failed.
environment=staging
application=payment
status=503
request_id=req-123
retry_exhausted=true
```

Bad:

```text
Something went wrong.
```

---

# 128. Error Messages for Operators

Include:

```text
What failed
Where
Why
What was attempted
How many retries
Current state
Recommended next step
```

Example:

```text
Deployment verification failed.
payment/prod
release=rel-123
ArgoCD=Synced
Kubernetes=Ready
HTTP health=500
Prometheus error rate=8.2%
Rollback recommended.
```

---

# 129. Error Messages for Security

Do not include:

```text
Credential value
Token
Password
Private key
Secret payload
```

Security and observability must coexist.

---

# 130. Incident Troubleshooting Workflow

When production automation fails:

```text
1. Identify release ID
2. Identify failed stage
3. Identify dependency
4. Check status code/exception
5. Check request ID
6. Check logs
7. Check dependency health
8. Check whether operation succeeded
9. Reconcile state
10. Decide retry/rollback
11. Verify final state
12. Document incident
```

---

# 131. Example Incident — 503 During ArgoCD Sync

```text
Python
 |
 v
ArgoCD
 |
 503
```

Check:

```text
ArgoCD server pods
Ingress/ALB
Service endpoints
Redis
Network
CPU/memory
Recent changes
```

Then:

```text
Retry if safe
```

Do not immediately roll back Git.

---

# 132. Example Incident — 504 After Deployment POST

```text
POST deployment
 |
 v
504
```

Correct approach:

```text
Do not immediately POST again
 |
 v
Search release ID
 |
 v
Check deployment state
 |
 +-- running -> monitor
 +-- success -> continue
 +-- failed -> analyze
 +-- missing -> retry if safe
```

---

# 133. Example Incident — Kubernetes 403

```text
Python
 |
 v
Kubernetes API
 |
 403
```

Check:

```text
ServiceAccount
Role
RoleBinding
Namespace
Resource
Verb
```

Useful:

```bash
kubectl auth can-i \
  get deployments \
  --as=system:serviceaccount:devops:deployment-automation \
  -n prod
```

---

# 134. Example Incident — 429 From GitHub

Check:

```text
API rate limit
Remaining quota
Reset time
Concurrency
Pagination
Duplicate calls
```

Then:

```text
Reduce calls
Honor rate limit
Backoff
Cache where safe
```

---

# 135. Example Incident — Expired OAuth Token

```text
API
 |
 v
401
```

Check:

```text
exp
issuer
audience
refresh mechanism
credential provider
```

If refresh is supported:

```text
Refresh
 |
 v
Retry once
```

If refresh fails:

```text
Fail
```

Do not loop forever.

---

# 136. Example Incident — Invalid JSON

```text
HTTP 502
```

body:

```html
<html>...
```

Code:

```python
response.json()
```

fails.

Correct:

```text
Classify HTTP error first
Inspect sanitized body
Only parse JSON when appropriate
```

---

# 137. Error Handling Order

Recommended order:

```text
1. Transport
2. HTTP status
3. Content type
4. Parse response
5. Validate schema
6. Validate business state
7. Execute workflow transition
```

This prevents misleading errors.

---

# 138. Example Correct Processing

```python
try:
    response = session.get(
        url,
        timeout=10
    )

except requests.exceptions.Timeout as exc:
    raise APIError(
        "API timeout"
    ) from exc

if response.status_code >= 400:
    handle_http_error(response)

data = response.json()

validate_schema(data)

validate_business_state(data)
```

---

# 139. Retry Layer

Keep retry logic separate:

```text
HTTP client
    |
    v
Retry policy
    |
    v
API operation
```

This prevents business logic from becoming:

```text
try
except
sleep
try
except
```

everywhere.

---

# 140. Retry Policy Object

Conceptual:

```python
from dataclasses import dataclass


@dataclass
class RetryPolicy:

    max_attempts: int = 4
    base_delay: float = 1
    max_delay: float = 30
```

A reusable policy makes behavior consistent.

---

# 141. Error Policy Object

Conceptual:

```text
status
 |
 v
RetryPolicy
 |
 v
ReconciliationPolicy
 |
 v
FailurePolicy
```

This separates:

```text
transport behavior
```

from:

```text
business decisions
```

---

# 142. Service-Specific Error Policies

GitHub:

```text
429 -> rate-limit handling
```

ArgoCD:

```text
409 -> state reconciliation
```

Jenkins:

```text
queue timeout -> fail build workflow
```

Kubernetes:

```text
403 -> RBAC issue
```

Each service has unique semantics.

---

# 143. Error Handling Architecture

```text
                 Workflow
                    |
                    v
             Service Client
                    |
                    v
              API Client
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Retry     Timeout    Error Map
          |         |         |
          +---------+---------+
                    |
                    v
                 Requests
                    |
                    v
                  HTTP
```

---

# 144. Production Error Handling Architecture

```text
                       Release
                          |
                          v
                    State Machine
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
      Retry          Reconcile          Rollback
        |                 |                 |
        +-----------------+-----------------+
                          |
                          v
                   Error Classifier
                          |
          +---------------+---------------+
          |       |       |       |       |
          v       v       v       v       v
        4xx     429     5xx    Network  Timeout
          |       |       |       |       |
          +-------+-------+-------+-------+
                          |
                          v
                    Observability
                          |
             +------------+------------+
             |            |            |
             v            v            v
           Logs        Metrics       Alerts
```

---

# 145. Production Checklist

```text
Error Classification
[ ] 4xx classified
[ ] 5xx classified
[ ] Network errors classified
[ ] Timeout errors classified
[ ] TLS errors classified
[ ] Parsing errors classified
[ ] Business errors classified

Retry
[ ] Retryable errors defined
[ ] Non-retryable errors defined
[ ] Maximum attempts
[ ] Backoff
[ ] Jitter
[ ] Retry-After
[ ] Overall deadline
[ ] Method safety considered

Reliability
[ ] Idempotency
[ ] Reconciliation
[ ] State machine
[ ] Checkpoint/recovery
[ ] Concurrency limits
[ ] Circuit breaker where appropriate
[ ] Bulkhead where appropriate

Security
[ ] No secret logging
[ ] Authentication failures handled
[ ] Authorization failures handled
[ ] Fail-closed security gates
[ ] TLS verification

Observability
[ ] Release ID
[ ] Request ID
[ ] Structured logs
[ ] Error metrics
[ ] Retry metrics
[ ] Alerting
[ ] Latency metrics

Recovery
[ ] Rollback policy
[ ] Compensation
[ ] Partial failure handling
[ ] Unknown state handling
[ ] Manual recovery path

Testing
[ ] Timeout tests
[ ] 429 tests
[ ] 5xx tests
[ ] 401/403 tests
[ ] Malformed response tests
[ ] Duplicate request tests
[ ] Partial failure tests
[ ] Recovery tests
```

---

# 146. Interview Questions

## Q1. How do you handle API errors in Python?

I classify errors into:

```text
transport
HTTP
authentication
authorization
rate limit
server
business
parsing
```

Then apply the appropriate:

```text
retry
reconciliation
failure
rollback
```

strategy.

---

## Q2. Should all 5xx errors be retried?

No.

Some are transient, but retries must be bounded and the operation must be safe to retry.

For state-changing operations, I reconcile state before retrying when the outcome is uncertain.

---

## Q3. How do you handle 429?

I inspect `Retry-After` and rate-limit headers, apply bounded backoff with jitter, and reduce concurrency if necessary.

---

## Q4. What do you do when a POST times out?

I do not assume the operation failed.

I reconcile external state using:

```text
operation ID
release ID
resource lookup
server state
```

Then retry only if safe.

---

## Q5. What is the difference between retry and reconciliation?

Retry means:

```text
send the request again
```

Reconciliation means:

```text
inspect external state and determine what actually happened
```

Reconciliation is critical after uncertain state-changing operations.

---

## Q6. What is a circuit breaker?

A mechanism that temporarily stops calls to an unhealthy dependency after repeated failures, allowing the dependency and client to recover.

---

## Q7. What is jitter?

Randomized delay added to retries to prevent many clients from retrying simultaneously.

---

## Q8. Why use `time.monotonic()`?

It measures elapsed time without being affected by wall-clock changes.

---

## Q9. What is a retry storm?

A large number of clients repeatedly retrying a failing dependency, potentially making the outage worse.

---

## Q10. What is fail closed?

Stopping the workflow when a required security/control dependency cannot safely determine whether the operation should continue.

---

# 147. Scenario Interview — ArgoCD 503

### Question

ArgoCD returns 503 during deployment. What do you do?

### Answer

First determine whether the failure is transient.

I would:

```text
1. Capture request/release ID
2. Inspect 503 response
3. Check ArgoCD server/ingress health
4. Check whether sync was already initiated
5. Retry according to bounded policy
6. Reconcile application state
7. Continue if synced/healthy
8. Fail clearly if deadline expires
```

I would not blindly trigger another sync.

---

# 148. Scenario Interview — POST Timeout

### Question

A deployment POST timed out. Should you retry?

### Answer

Not immediately.

The server may have processed the deployment.

I would:

```text
Check operation ID
Check deployment state
Check release history
Check GitOps/ArgoCD
```

Then:

```text
Already running -> monitor
Succeeded -> continue
Failed -> analyze
Missing -> retry if safe
```

---

# 149. Scenario Interview — 429

### Question

Your Python script gets 429 from GitHub.

### Answer

I would:

```text
Read rate-limit headers
Honor Retry-After/reset
Reduce concurrency
Use bounded exponential backoff
Add jitter
Avoid unnecessary API calls
Use pagination efficiently
```

---

# 150. Scenario Interview — 403

### Question

Kubernetes API returns 403.

### Answer

I would treat this as authorization, not connectivity.

Check:

```text
ServiceAccount
Role
RoleBinding
Namespace
Verb
Resource
```

Then validate with:

```bash
kubectl auth can-i
```

---

# 151. Scenario Interview — 500 During POST

### Question

A POST returns 500. Is retry safe?

### Answer

Not automatically.

I would first determine whether the API supports:

```text
Idempotency-Key
```

or whether I can reconcile using:

```text
operation ID
resource lookup
```

If the operation is idempotent and the API contract allows it, bounded retry may be safe.

---

# 152. Scenario Interview — Security Scanner Down

### Question

The security scanner API is unavailable. Should deployment continue?

### Answer

If the security scan is a mandatory release gate, I would fail closed:

```text
Scanner unavailable
 |
 v
Deployment blocked
```

An emergency bypass should require an explicitly approved process and audit trail.

---

# 153. Scenario Interview — Notification API Down

### Question

Slack API is unavailable after deployment succeeds.

### Answer

If notification is non-critical:

```text
Deployment = success
Notification = failed
```

I would retry notification separately and alert on the notification failure without incorrectly marking the deployment as failed.

---

# 154. Scenario Interview — Prometheus Down

### Question

Deployment is healthy in Kubernetes but Prometheus is unavailable.

### Answer

I would follow release policy.

If Prometheus is mandatory for post-deployment verification:

```text
Deployment status = verification incomplete
```

If it is optional:

```text
Continue using other health signals
```

The policy should be explicit rather than decided ad hoc during an incident.

---

# 155. Scenario Interview — Duplicate Deployment

### Question

A pipeline timed out and was restarted. Now two deployments exist.

### Root Cause

The workflow lacked:

```text
Idempotency
State reconciliation
Unique release ID
```

### Prevention

Use:

```text
Idempotency keys
Release IDs
State lookup
Distributed locking
```

where supported.

---

# 156. Senior-Level Question

## Design an error-handling framework for a Python DevOps automation platform.

### Strong Answer

I would separate:

```text
Transport
HTTP classification
Retry
Reconciliation
Business policy
Workflow state
Observability
```

Architecture:

```text
Workflow
   |
   v
Business Policy
   |
   +---- Retry
   +---- Reconcile
   +---- Rollback
   +---- Fail
   |
   v
Service Client
   |
   v
API Client
   |
   +---- Timeout
   +---- Retry Policy
   +---- Error Mapping
   |
   v
Requests
```

Every error should contain enough context to answer:

```text
What failed?
Where?
Why?
Was the operation executed?
Can it be retried?
What is the current external state?
What should happen next?
```

---

# 157. Senior-Level Question — Distributed Failure

## Why is timeout handling harder than HTTP error handling?

Because an HTTP error provides a server response:

```text
503
```

A timeout provides:

```text
No definitive response
```

The server may have:

```text
received
processed
completed
```

the operation.

Therefore a timeout can create an **unknown state**, requiring reconciliation.

---

# 158. Senior-Level Question — Retry Design

## What makes a production retry policy?

It should define:

```text
Retryable conditions
Non-retryable conditions
Maximum attempts
Backoff
Jitter
Retry-After
Overall deadline
Method safety
Idempotency
Concurrency limits
Observability
```

---

# 159. Senior-Level Question — Circuit Breaker

## When would you use a circuit breaker?

For a long-running service making repeated calls to an unstable dependency.

For a short CI job, bounded retries and deadlines may be sufficient.

I would avoid adding a circuit breaker unless the workload actually benefits from it.

---

# 160. Senior-Level Question — Error Handling Architecture

## How would you prevent error-handling code from becoming duplicated everywhere?

Use:

```text
Generic API client
+
central error classification
+
retry policy
+
custom exceptions
+
service-specific policies
```

Business workflows should consume meaningful exceptions instead of repeatedly interpreting raw HTTP status codes.

---

# 161. Production Example — Complete Failure Flow

```text
Python
 |
 | POST deployment
 v
API
 |
 | 202 Accepted
 v
operation_id
 |
 v
Poll
 |
 | 503
 v
Retry with backoff
 |
 | 200
 v
status=running
 |
 v
Poll
 |
 | timeout
 v
UNKNOWN
 |
 v
Reconcile
 |
 v
operation=succeeded
 |
 v
Kubernetes verification
 |
 | ready
 v
Health endpoint
 |
 | 500
 v
Prometheus
 |
 | high error rate
 v
Rollback policy
 |
 v
Git previous revision
 |
 v
ArgoCD
 |
 v
Kubernetes
 |
 v
Verify recovery
 |
 v
ROLLBACK SUCCESS
```

This is what production-grade API error handling looks like.

---

# 162. Final Mental Model

Do not think:

```text
API call
  |
  +-- success
  +-- failure
```

Think:

```text
API call
   |
   v
What happened?
   |
   +-- Success
   |
   +-- Known failure
   |
   +-- Temporary failure
   |
   +-- Authentication failure
   |
   +-- Authorization failure
   |
   +-- Rate limited
   |
   +-- Unknown state
   |
   +-- Partial success
```

Then:

```text
Classify
   ↓
Choose action
   ↓
Retry / Reconcile / Rollback / Fail
   ↓
Verify
   ↓
Observe
```

---

# 163. What You Should Be Able to Explain

Before moving to the final file in this section, you should be comfortable explaining:

```text
HTTP error classes
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

Requests exceptions
ConnectionError
Timeout
SSLError
HTTPError

Retry policy
Exponential backoff
Jitter
Retry-After
Retry budget
Deadline

Idempotency
Idempotency keys
Reconciliation
Unknown state
Distributed failures

Circuit breaker
Bulkhead
Fallback
Fail open
Fail closed

Partial failure
Compensation
Rollback
State machine
Checkpointing
Concurrency
Race conditions
Distributed locks

Exception design
Exception chaining
Structured errors
Request IDs
Correlation IDs
Metrics
Logging
Alerting

DevOps troubleshooting
GitHub
Jenkins
ArgoCD
Kubernetes
AWS
Prometheus
ELK

Production incident handling
```

---

# 164. Section Progress

Current progress:

```text
08-Python-APIs/
├── 01-HTTP-and-REST.md       ✓
├── 02-Requests-Library.md    ✓
├── 03-API-Automation.md      ✓
├── 04-Authentication.md      ✓
├── 05-API-Error-Handling.md  ✓
└── 06-DevOps-API-Projects.md
```

The final file:

# `06-DevOps-API-Projects.md`

will convert everything learned so far into **complete production-style Python DevOps projects**, including:

```text
Project 1:
GitHub API automation

Project 2:
Jenkins build automation

Project 3:
ECR/image verification

Project 4:
ArgoCD deployment automation

Project 5:
Kubernetes deployment verification

Project 6:
Prometheus-based deployment verification

Project 7:
ELK-based log verification

Project 8:
End-to-end CI/CD release orchestrator

Project 9:
Automated rollback

Project 10:
Production-grade DevOps API platform
```

Each project will connect:

```text
Python
+
Requests
+
Authentication
+
Error Handling
+
Retries
+
Reconciliation
+
CI/CD
+
Kubernetes
+
AWS
+
GitOps
+
Observability
```

> **The goal is not to know how to call an API. The goal is to build automation that remains correct when APIs, networks, credentials, and production systems fail.**
