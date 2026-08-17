# 22-Python-for-DevOps
# 01-Python-Fundamentals
# 09-Exception-Handling

> Exception handling is critical in DevOps automation because production scripts interact with systems that fail: APIs time out, AWS calls are throttled, files disappear, Kubernetes resources change, commands return non-zero exit codes, credentials expire, and deployments can partially succeed.
>
> A production-grade Python script must distinguish expected operational failures from programming bugs, provide useful diagnostics, clean up resources, return meaningful exit codes, and avoid hiding failures.

---

# 1. Why Exception Handling Matters in DevOps

DevOps automation commonly interacts with:

```text
AWS APIs
Kubernetes APIs
Linux commands
SSH
Docker
Terraform
Ansible
CI/CD systems
files
databases
HTTP APIs
DNS
load balancers
container registries
monitoring systems
security scanners
```

Every external dependency can fail.

A reliable script should answer:

```text
What failed?
Why did it fail?
Can it be retried?
Should the script stop?
What cleanup is required?
What should the CI/CD pipeline report?
```

---

# 2. Exception vs Error

An exception is Python's mechanism for reporting an abnormal condition during execution.

Example:

```python
result = 10 / 0
```

Python raises:

```text
ZeroDivisionError
```

In DevOps, exceptions often represent operational conditions:

```text
FileNotFoundError
PermissionError
TimeoutError
ConnectionError
JSONDecodeError
```

---

# 3. Basic `try/except`

```python
try:
    value = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
```

The exception is handled instead of terminating the program immediately.

---

# 4. `try/except/else`

```python
try:
    result = int("100")
except ValueError:
    print("Invalid number")
else:
    print("Conversion succeeded")
```

`else` runs only when the `try` block succeeds.

This is useful when you want the success path separated from exception handling.

---

# 5. `try/except/finally`

```python
file = None

try:
    file = open("config.txt")
    content = file.read()
except OSError as exc:
    print(f"File operation failed: {exc}")
finally:
    if file:
        file.close()
```

For files, prefer:

```python
with open("config.txt") as file:
    ...
```

The `with` statement handles cleanup automatically.

---

# 6. Full Exception Structure

```python
try:
    operation()
except SpecificError as exc:
    handle_error(exc)
else:
    handle_success()
finally:
    cleanup()
```

Mental model:

```text
try
 |
 +--> success --> else
 |
 +--> exception --> except
 |
 +--> always --> finally
```

---

# 7. Catch Specific Exceptions

Prefer:

```python
try:
    config = load_config()
except FileNotFoundError:
    ...
```

Avoid:

```python
try:
    config = load_config()
except Exception:
    ...
```

unless you have a specific reason to handle all application-level exceptions at that boundary.

Specific exceptions make recovery behavior clearer.

---

# 8. Why Broad `except Exception` Can Be Dangerous

Example:

```python
try:
    deploy()
except Exception:
    print("Deployment failed")
```

This hides the actual cause.

It can mask:

```text
programming bugs
unexpected data
security failures
permission problems
API failures
incorrect assumptions
```

If broad handling is necessary, log/record the exception and re-raise or convert it appropriately.

---

# 9. Never Silently Ignore Exceptions

Bad:

```python
try:
    deploy()
except Exception:
    pass
```

This can make CI report:

```text
SUCCESS
```

even though deployment failed.

Production automation should fail explicitly when required.

---

# 10. Built-in Exceptions Important for DevOps

Common exceptions:

```text
FileNotFoundError
PermissionError
FileExistsError
IsADirectoryError
NotADirectoryError
OSError
ValueError
TypeError
KeyError
IndexError
AttributeError
TimeoutError
ConnectionError
JSONDecodeError
subprocess.CalledProcessError
subprocess.TimeoutExpired
```

Know what each means and what recovery makes sense.

---

# 11. `FileNotFoundError`

```python
from pathlib import Path

path = Path("config.yaml")

try:
    content = path.read_text(
        encoding="utf-8"
    )
except FileNotFoundError:
    raise RuntimeError(
        f"Configuration missing: {path}"
    )
```

Do not silently create an empty configuration unless that is explicitly the intended behavior.

---

# 12. `PermissionError`

```python
try:
    content = Path(
        "/etc/app/config.yaml"
    ).read_text(
        encoding="utf-8"
    )
except PermissionError:
    raise RuntimeError(
        "Insufficient permission to read configuration"
    )
```

In containers, investigate:

```text
UID/GID
file ownership
mount permissions
securityContext
read-only filesystem
```

before changing privileges.

---

# 13. `FileExistsError`

Useful with exclusive creation:

```python
try:
    with open(
        "deployment.lock",
        "x"
    ) as file:
        file.write("locked")
except FileExistsError:
    print("Lock already exists")
```

Whether this is a failure depends on the automation's design.

---

# 14. `IsADirectoryError`

```python
try:
    Path("logs").read_text()
except IsADirectoryError:
    print("Expected a file, received a directory")
```

Validate file type before processing when the distinction matters.

---

# 15. `NotADirectoryError`

Example:

```text
/app/config
```

is a file, but code expects:

```text
/app/config/settings.yaml
```

Python may raise:

```text
NotADirectoryError
```

This often indicates an incorrect path assumption.

---

# 16. `ValueError`

Used when a value has the correct type but is invalid.

```python
replicas = int(value)

if replicas < 1:
    raise ValueError(
        "Replicas must be >= 1"
    )
```

---

# 17. `TypeError`

Example:

```python
sum("100")
```

may produce:

```text
TypeError
```

This generally indicates an incompatible object/type was supplied.

Do not catch `TypeError` and assume it is always bad user input; it can also reveal a programming bug.

---

# 18. `KeyError`

```python
config = {
    "environment": "production"
}

try:
    region = config["region"]
except KeyError:
    raise ValueError(
        "Missing required field: region"
    )
```

For required configuration keys, explicit validation is often cleaner than waiting for `KeyError`.

---

# 19. `.get()` vs `KeyError`

Optional field:

```python
timeout = config.get(
    "timeout",
    30
)
```

Required field:

```python
if "region" not in config:
    raise ValueError(
        "region is required"
    )
```

Avoid using `.get()` for required values if missing configuration should fail.

---

# 20. `IndexError`

```python
items = []

try:
    first = items[0]
except IndexError:
    ...
```

In production automation, validate collection length when an empty result is an expected operational condition.

---

# 21. `AttributeError`

Example:

```python
response = None

response.status
```

can raise:

```text
AttributeError
```

Do not broadly catch `AttributeError` to hide bugs. It often means the code's object assumptions are wrong.

---

# 22. `JSONDecodeError`

```python
import json

try:
    with open(
        "report.json",
        encoding="utf-8"
    ) as file:
        report = json.load(file)
except json.JSONDecodeError as exc:
    raise ValueError(
        "Invalid JSON report"
    ) from exc
```

Useful in CI when consuming generated reports.

---

# 23. Exception Chaining

Use:

```python
try:
    config = load_config()
except FileNotFoundError as exc:
    raise RuntimeError(
        "Application configuration is missing"
    ) from exc
```

This preserves the original cause.

The resulting error communicates both:

```text
high-level meaning
+
original technical cause
```

---

# 24. Why `raise ... from exc` Matters

Without chaining:

```text
configuration loading failed
```

With chaining:

```text
configuration loading failed
caused by:
FileNotFoundError: ...
```

This is valuable during production troubleshooting.

---

# 25. Re-Raising an Exception

```python
try:
    process()
except TimeoutError:
    logger.error("Operation timed out")
    raise
```

`raise` without an argument re-raises the active exception.

Useful when you want to log context but preserve the original failure.

---

# 26. Converting Exceptions

Suppose a low-level API raises:

```text
ConnectionError
```

Your application may expose:

```python
class DeploymentError(Exception):
    pass
```

Then:

```python
try:
    deploy_api()
except ConnectionError as exc:
    raise DeploymentError(
        "Deployment API unavailable"
    ) from exc
```

This creates a domain-specific error boundary.

---

# 27. Custom Exceptions

```python
class ConfigurationError(Exception):
    """Invalid application configuration."""
```

Use:

```python
raise ConfigurationError(
    "Production region is missing"
)
```

Custom exceptions make large automation projects easier to reason about.

---

# 28. Exception Hierarchy

Example:

```python
class DevOpsError(Exception):
    pass


class ConfigurationError(DevOpsError):
    pass


class DeploymentError(DevOpsError):
    pass


class VerificationError(DevOpsError):
    pass
```

Then:

```python
except DevOpsError:
    ...
```

can handle application-level failures while still allowing specific subclasses.

---

# 29. Exception Hierarchy Design

Possible project structure:

```text
DevOpsError
 |
 +-- ConfigurationError
 |
 +-- AuthenticationError
 |
 +-- APIError
 |
 +-- DeploymentError
 |
 +-- VerificationError
 |
 +-- ArtifactError
```

Do not create dozens of custom exceptions without a useful distinction.

---

# 30. Operational vs Programming Errors

Operational:

```text
API timeout
missing file
permission denied
AWS throttling
Kubernetes resource not found
network unavailable
```

Programming:

```text
wrong variable
invalid attribute
logic bug
unexpected None
incorrect function contract
```

Retry operational failures selectively.

Do not blindly retry programming bugs.

---

# 31. Fail Fast

If required configuration is missing:

```python
if not config.get("region"):
    raise ConfigurationError(
        "AWS region is required"
    )
```

Do not continue and fail later with a confusing API error.

---

# 32. Fail Fast vs Graceful Degradation

Fail fast when:

```text
required configuration missing
security validation fails
artifact integrity fails
deployment target is invalid
credentials are unavailable
```

Graceful degradation may be appropriate when:

```text
optional metrics unavailable
non-critical report cannot be generated
optional cache is unavailable
```

The decision depends on the business requirement.

---

# 33. Retryable vs Non-Retryable Errors

Potentially retryable:

```text
timeout
temporary network failure
HTTP 429
HTTP 503
temporary connection reset
```

Usually non-retryable:

```text
HTTP 400
invalid configuration
authentication failure
permission denied
malformed JSON
missing required parameter
```

There are exceptions, so classify based on the API/system semantics.

---

# 34. Retry Is Not Exception Handling Alone

Bad:

```python
try:
    deploy()
except Exception:
    deploy()
```

This can:

```text
repeat permanent failures
duplicate side effects
increase load
hide root causes
```

Use:

```text
classification
bounded attempts
backoff
jitter
timeouts
idempotency
```

---

# 35. Exponential Backoff

Concept:

```text
attempt 1 -> 1 sec
attempt 2 -> 2 sec
attempt 3 -> 4 sec
attempt 4 -> 8 sec
```

Formula:

```text
delay = base * 2^attempt
```

Use a maximum delay.

---

# 36. Jitter

Without jitter, many workers may retry simultaneously.

```text
worker A -> retry at 8 sec
worker B -> retry at 8 sec
worker C -> retry at 8 sec
```

This creates a retry storm.

Jitter randomizes the delay.

```text
A -> 7.2 sec
B -> 8.7 sec
C -> 6.9 sec
```

---

# 37. Retry Function

```python
import random
import time


def retry_operation(
    operation,
    attempts=3,
    base_delay=1
):
    for attempt in range(attempts):
        try:
            return operation()

        except TimeoutError:
            if attempt == attempts - 1:
                raise

            delay = (
                base_delay * (2 ** attempt)
            )

            delay += random.uniform(
                0,
                delay * 0.2
            )

            time.sleep(delay)
```

This is a basic pattern; production retry libraries can provide richer controls.

---

# 38. Add a Maximum Delay

```python
delay = min(
    delay,
    30
)
```

Avoid retries sleeping for unbounded durations.

---

# 39. Total Retry Budget

A production system should think about:

```text
max attempts
max per-attempt timeout
max total retry time
```

Example:

```text
3 attempts
10-second timeout each
30-second total budget
```

A retry policy without a time budget can make a pipeline appear hung.

---

# 40. Retry Only the Right Operation

Bad:

```python
retry(
    lambda: deploy_and_notify_and_cleanup()
)
```

If deployment succeeded but notification failed, retrying the entire operation could redeploy.

Better:

```text
deploy
 |
 v
verify
 |
 v
notify
```

Retry only the retryable step.

---

# 41. Idempotency

An operation is idempotent when repeating it produces the same intended final state.

Example:

```text
set replicas = 3
```

is generally safer to retry than:

```text
create resource
```

if the create operation is not protected against duplicates.

---

# 42. DevOps Retry + Idempotency

Strong pattern:

```text
timeout
 |
 v
determine whether operation completed
 |
 v
check current state
 |
 v
retry only if required
```

Do not assume:

```text
timeout = operation did not happen
```

A request can succeed remotely while the response is lost.

---

# 43. HTTP Errors

Using `requests`:

```python
import requests

response = requests.get(
    url,
    timeout=10
)

response.raise_for_status()
```

This raises an HTTP-related exception for unsuccessful status codes.

---

# 44. Handle HTTP Errors

```python
try:
    response = requests.get(
        url,
        timeout=10
    )
    response.raise_for_status()

except requests.Timeout:
    ...

except requests.ConnectionError:
    ...

except requests.HTTPError as exc:
    ...
```

Classify status codes before deciding whether to retry.

---

# 45. HTTP Status Classification

Typical categories:

```text
2xx -> success
3xx -> redirect
4xx -> client/request problem
5xx -> server problem
```

Potential retry:

```text
429
502
503
504
```

But follow the API's documented retry semantics.

---

# 46. Timeout Everything External

Do not do:

```python
requests.get(url)
```

without a deliberate timeout policy in production automation.

Prefer:

```python
requests.get(
    url,
    timeout=10
)
```

A timeout prevents one dependency from hanging the entire job indefinitely.

---

# 47. Connect vs Read Timeout

Some HTTP libraries support:

```text
connect timeout
read timeout
```

This is useful because:

```text
server connection
```

and:

```text
server response
```

are different failure modes.

---

# 48. AWS API Exceptions

With boto3:

```python
from botocore.exceptions import (
    ClientError,
    BotoCoreError
)
```

Example:

```python
try:
    response = client.describe_instances()

except ClientError as exc:
    ...
```

Inspect the AWS error code rather than treating every `ClientError` identically.

---

# 49. AWS Error Classification

Examples:

```text
ThrottlingException
AccessDenied
ResourceNotFoundException
ValidationException
```

Possible responses:

```text
throttling -> backoff/retry
access denied -> fail and investigate IAM
not found -> decide whether expected
validation -> fix input
```

---

# 50. AWS Throttling

When throttled:

```text
do not immediately retry in a tight loop
```

Use:

```text
exponential backoff
jitter
bounded attempts
API pagination
request reduction
caching where appropriate
```

AWS SDKs may already provide retry behavior, so understand SDK configuration before implementing another retry layer.

---

# 51. AWS Pagination

A common cause of failures is inefficient API usage.

Prefer paginators:

```python
paginator = client.get_paginator(
    "describe_instances"
)

for page in paginator.paginate():
    process(page)
```

This reduces the need to manually manage page tokens and supports incremental processing.

---

# 52. Kubernetes API Exceptions

A Kubernetes client can encounter:

```text
404 Not Found
401 Unauthorized
403 Forbidden
409 Conflict
429 Too Many Requests
500/503
timeout
```

Classify them.

Example:

```text
404 -> resource may not exist
403 -> RBAC problem
409 -> state changed concurrently
429 -> API pressure
503 -> control-plane/service availability
```

---

# 53. Kubernetes `409 Conflict`

A conflict often means another actor changed the resource.

Do not blindly retry the same stale update.

Better:

```text
read latest state
 |
 v
recalculate change
 |
 v
update
```

This is optimistic concurrency handling.

---

# 54. Kubernetes `403 Forbidden`

Usually investigate:

```text
ServiceAccount
Role
ClusterRole
RoleBinding
ClusterRoleBinding
namespace
API resource
verb
```

Do not fix it by giving:

```text
cluster-admin
```

without justification.

---

# 55. Kubernetes `404 Not Found`

Ask:

```text
Was the resource expected?
Was it already deleted?
Wrong namespace?
Wrong cluster?
Wrong API version?
Typo?
```

A 404 is not automatically retryable.

---

# 56. Kubernetes Timeouts

Possible causes:

```text
network
API server load
DNS
proxy
client timeout
control-plane issue
```

Use:

```text
bounded timeout
limited retry
backoff
observability
```

---

# 57. Subprocess Exceptions

Python:

```python
import subprocess

subprocess.run(
    ["kubectl", "get", "pods"],
    check=True
)
```

If the command exits non-zero:

```text
CalledProcessError
```

can be raised.

---

# 58. Capture Subprocess Output

```python
result = subprocess.run(
    ["kubectl", "get", "pods"],
    check=True,
    text=True,
    capture_output=True
)

print(result.stdout)
```

On failure:

```python
try:
    ...
except subprocess.CalledProcessError as exc:
    print(exc.returncode)
    print(exc.stderr)
```

Avoid logging sensitive command arguments.

---

# 59. `TimeoutExpired`

```python
try:
    subprocess.run(
        ["kubectl", "get", "pods"],
        check=True,
        timeout=20
    )
except subprocess.TimeoutExpired:
    print("kubectl timed out")
```

Always put a deliberate timeout around external commands that could hang.

---

# 60. Shell Commands and Security

Avoid:

```python
subprocess.run(
    f"rm -rf {user_input}",
    shell=True
)
```

This can introduce command injection.

Prefer argument lists:

```python
subprocess.run(
    ["rm", "-rf", validated_path],
    check=True
)
```

Even then, validate destructive paths.

---

# 61. `shell=True`

Use only when shell semantics are actually required.

Risks include:

```text
command injection
quoting complexity
unexpected expansion
environment behavior
```

Prefer:

```python
subprocess.run(
    ["command", "arg1", "arg2"]
)
```

---

# 62. Environment Variables

```python
import os

region = os.environ.get(
    "AWS_REGION"
)
```

Required variable:

```python
if not region:
    raise ConfigurationError(
        "AWS_REGION is required"
    )
```

Do not catch a missing variable and silently invent a production value.

---

# 63. Authentication Failures

Authentication errors generally should not be blindly retried.

Examples:

```text
expired token
invalid credentials
wrong role
missing credential
```

Correct response:

```text
fail
report clearly
refresh/repair credential source
retry only after state changes
```

---

# 64. Authorization Failures

Examples:

```text
403
AccessDenied
RBAC denied
PermissionError
```

Usually:

```text
not transient
```

Investigate:

```text
IAM
RBAC
filesystem permissions
security policy
```

---

# 65. DNS Failures

Possible Python exceptions:

```text
socket.gaierror
ConnectionError
```

Investigate:

```text
DNS configuration
resolver
service discovery
network policy
hostname
```

A retry can help with temporary DNS issues, but repeated retries do not fix a wrong hostname.

---

# 66. Network Failure Pattern

```python
try:
    call_service()

except TimeoutError:
    retry()

except ConnectionError:
    retry()

except PermissionError:
    fail()

except ValueError:
    fail()
```

The key is classification.

---

# 67. Exception Logging

Use logging:

```python
import logging

logger = logging.getLogger(__name__)

try:
    deploy()
except DeploymentError:
    logger.exception(
        "Deployment failed"
    )
    raise
```

`logger.exception()` includes traceback when called inside an exception handler.

---

# 68. `logger.error()` vs `logger.exception()`

```python
logger.error("Deployment failed")
```

logs the message.

```python
logger.exception("Deployment failed")
```

logs the message plus current traceback.

Use traceback logging when the stack is useful for diagnosis.

---

# 69. Do Not Log Secrets

Bad:

```python
logger.exception(
    "Request failed: %s",
    request
)
```

if request contains:

```text
Authorization
password
token
API key
secret
```

Sanitize before logging.

---

# 70. Structured Error Logging

Example:

```python
logger.error(
    "deployment_failed",
    extra={
        "service": "payment",
        "environment": "production",
        "attempt": 2
    }
)
```

The exact structured logging approach depends on the logging framework.

Useful fields:

```text
service
environment
operation
request_id
deployment_id
attempt
error_type
```

Never include secret values.

---

# 71. Correlation IDs

For distributed automation:

```text
request_id
deployment_id
incident_id
```

can connect:

```text
Python logs
CI job
AWS API
Kubernetes events
application logs
```

This makes troubleshooting much easier.

---

# 72. Exception Context

Bad:

```python
raise RuntimeError("Failed")
```

Better:

```python
raise RuntimeError(
    f"Failed to deploy {service} "
    f"to {environment}"
) from exc
```

Include useful non-sensitive context.

---

# 73. Error Messages Should Be Actionable

Bad:

```text
Something went wrong
```

Better:

```text
Failed to read /app/config.yaml:
permission denied. Verify container UID
and mounted-file permissions.
```

A good error helps the next engineer act.

---

# 74. Exit Codes in DevOps

Typical convention:

```text
0 -> success
non-zero -> failure
```

Example:

```python
raise SystemExit(1)
```

CI/CD systems use this result to decide pipeline status.

---

# 75. CLI Exception Boundary

A clean architecture:

```text
main()
 |
 v
application logic
 |
 v
specific exceptions
 |
 v
main catches known application errors
 |
 v
log message
 |
 v
exit non-zero
```

Do not scatter:

```python
sys.exit()
```

through every helper function.

---

# 76. Example CLI Boundary

```python
def main():
    deploy()


if __name__ == "__main__":
    try:
        main()
    except ConfigurationError as exc:
        print(f"CONFIG ERROR: {exc}")
        raise SystemExit(2)
    except DeploymentError as exc:
        print(f"DEPLOY ERROR: {exc}")
        raise SystemExit(1)
```

This creates predictable CLI behavior.

---

# 77. Exit Code Design

Possible:

```text
0 -> success
1 -> operational failure
2 -> invalid usage/configuration
3 -> verification failure
```

The exact scheme should be documented and consistent.

---

# 78. CI/CD Failure Semantics

A pipeline stage should distinguish:

```text
script crashed
configuration invalid
security gate failed
deployment failed
verification failed
```

This helps automation and humans understand what happened.

---

# 79. Cleanup With `finally`

Example:

```python
temp_dir = create_temp_dir()

try:
    build(temp_dir)
    test(temp_dir)
finally:
    cleanup(temp_dir)
```

Even if:

```text
build fails
test fails
unexpected exception
```

cleanup runs.

---

# 80. Cleanup With Context Managers

Better:

```python
with tempfile.TemporaryDirectory() as temp_dir:
    build(temp_dir)
    test(temp_dir)
```

The context manager handles cleanup.

---

# 81. Cleanup Must Not Hide Original Failure

Dangerous:

```python
try:
    deploy()
finally:
    cleanup()
```

If cleanup itself raises, it can obscure the original deployment exception.

When cleanup can fail independently, handle/log cleanup errors carefully while preserving the primary failure.

---

# 82. Cleanup Error Pattern

```python
try:
    deploy()
finally:
    try:
        cleanup()
    except OSError:
        logger.exception(
            "Cleanup failed"
        )
```

This is useful when cleanup failure should be recorded but should not replace the main error.

---

# 83. Exception During Rollback

Consider:

```text
deployment fails
 |
 v
rollback
 |
 v
rollback fails
```

Now there are two failures.

Report:

```text
primary deployment failure
+
rollback failure
```

Do not hide either.

---

# 84. Deployment Transaction Mental Model

```text
prepare
 |
 v
validate
 |
 v
deploy
 |
 v
verify
 |
 +--> success
 |
 +--> failure
       |
       v
     rollback
       |
       v
     verify rollback
```

Python exception handling should represent these stages clearly.

---

# 85. Partial Success

Suppose deployment targets:

```text
10 services
```

Results:

```text
8 succeeded
2 failed
```

Do not automatically report:

```text
success
```

Define policy:

```text
all must succeed
or
partial success accepted
or
failed services require rollback
```

---

# 86. Batch Processing Exceptions

Bad:

```python
for item in items:
    process(item)
```

One failure may stop the entire batch.

If independent processing is intended:

```python
failures = []

for item in items:
    try:
        process(item)
    except Exception as exc:
        failures.append(
            (item, exc)
        )
```

But do not use this pattern if one failure should stop the operation.

---

# 87. Continue-on-Error

Appropriate for:

```text
inventory collection
independent health checks
multi-resource reporting
best-effort diagnostics
```

Not always appropriate for:

```text
database migration
security deployment
critical configuration rollout
ordered infrastructure changes
```

---

# 88. Collecting Exceptions

```python
failures = []

for service in services:
    try:
        deploy(service)
    except DeploymentError as exc:
        failures.append({
            "service": service,
            "error": str(exc)
        })

if failures:
    raise DeploymentError(
        f"{len(failures)} services failed"
    )
```

This allows the script to finish independent operations while still returning failure.

---

# 89. Exception Groups

Modern Python supports exception groups:

```python
ExceptionGroup
```

and:

```python
except* SomeError:
    ...
```

This is useful for concurrent operations where multiple failures may occur.

Understand it conceptually; use it only when the concurrency design benefits from grouped failures.

---

# 90. Concurrent Operations

When using:

```python
concurrent.futures
```

exceptions from worker tasks may be raised when retrieving results.

Example:

```python
future = executor.submit(
    deploy,
    service
)

try:
    result = future.result()
except DeploymentError as exc:
    ...
```

Always define how worker failures affect the overall job.

---

# 91. ThreadPool Failure Strategy

Possible policies:

```text
fail immediately
collect all failures
cancel pending tasks
continue independent tasks
```

Choose explicitly.

---

# 92. ProcessPool Failure Strategy

The same principle applies:

```text
worker failure
 |
 v
main process
 |
 v
classify
 |
 v
continue/cancel/fail
```

Do not assume worker exceptions automatically produce the desired pipeline behavior.

---

# 93. Async Exceptions

With `asyncio`:

```python
try:
    await operation()
except TimeoutError:
    ...
```

For multiple tasks, determine:

```text
whether one failure cancels others
whether exceptions are collected
whether partial success is acceptable
```

---

# 94. Timeout Handling

Every external operation should have a bounded duration where practical:

```text
HTTP
AWS API
Kubernetes API
subprocess
database
SSH
```

Otherwise:

```text
one hung dependency
       |
       v
entire CI job hangs
```

---

# 95. Timeout Is a Failure Signal

Do not treat:

```text
timeout
```

as proof that:

```text
operation failed remotely
```

The operation may have completed while the response was delayed or lost.

This is especially important for deployment operations.

---

# 96. Retry + Timeout

Good pattern:

```text
attempt
 |
 +--> timeout
 |
 v
classify
 |
 v
backoff
 |
 v
retry
```

Set:

```text
per-attempt timeout
max attempts
total time budget
```

---

# 97. Circuit Breaker Concept

For repeated downstream failures:

```text
closed
  |
  v
failures increase
  |
  v
open
  |
  v
stop sending requests
  |
  v
half-open
  |
  v
test request
  |
  +--> success -> closed
  +--> failure -> open
```

Useful for long-running services.

For short CI scripts, bounded retries and timeouts may be sufficient.

---

# 98. Exception Handling and Observability

Every failure should be observable through appropriate:

```text
logs
metrics
exit codes
alerts
reports
trace/correlation IDs
```

Do not rely on a traceback alone in production automation.

---

# 99. Metrics for Failures

Useful counters:

```text
deployment_failures_total
api_retries_total
timeout_total
validation_failures_total
artifact_verification_failures_total
```

This makes repeated failures visible over time.

---

# 100. Alerting on Automation Failures

Not every script failure requires a page.

Classify:

```text
critical deployment failure -> page
nightly report failure -> ticket/email
optional inventory failure -> log
```

Alert severity should reflect business impact.

---

# 101. Exception Handling and SLOs

A deployment automation service may have an SLO such as:

```text
99.9% successful automation executions
```

Track:

```text
total runs
successful runs
failed runs
timeout runs
retry counts
```

Exceptions become measurable reliability signals.

---

# 102. Exception Handling in Scheduled Jobs

For cron/systemd/automation:

```text
catch expected error
log context
return non-zero
```

Do not:

```python
except Exception:
    print("done")
```

because schedulers may interpret success incorrectly.

---

# 103. Exception Handling in Jenkins

A Python script returning non-zero causes a shell step to fail under normal pipeline behavior.

Example:

```bash
python deploy.py
```

If Python exits:

```text
1
```

Jenkins can mark the stage failed.

---

# 104. Exception Handling in GitHub Actions

Example:

```yaml
- name: Deploy
  run: python deploy.py
```

Non-zero exit code:

```text
step fails
job normally fails
```

Do not catch deployment errors and exit zero just to keep the pipeline green.

---

# 105. Exception Handling in GitLab CI

Example:

```yaml
deploy:
  script:
    - python deploy.py
```

Non-zero exit code:

```text
job fails
```

Use explicit `allow_failure` only when the business requirement genuinely allows failure.

---

# 106. Security Scanner Exceptions

A security scanner may fail because:

```text
vulnerability found
scanner unavailable
authentication failed
invalid image
network failure
```

These are not necessarily equivalent.

Define:

```text
security finding -> policy failure
scanner unavailable -> infrastructure failure
invalid scan input -> pipeline failure
```

---

# 107. Do Not Confuse Tool Failure With Security Success

Bad:

```python
try:
    run_trivy()
except Exception:
    print("No vulnerabilities")
```

This is extremely dangerous.

A scanner outage must not become:

```text
zero vulnerabilities
```

---

# 108. Terraform Exception Strategy

A Python wrapper around Terraform should distinguish:

```text
Terraform command missing
init failure
validation failure
plan failure
apply failure
timeout
authentication failure
state lock failure
```

Do not reduce every failure to:

```text
terraform failed
```

---

# 109. Terraform State Lock Failure

Possible response:

```text
do not blindly force unlock
```

First determine:

```text
Is another Terraform run active?
Is the lock stale?
Who owns the lock?
```

Force-unlock is an operational decision, not a generic retry.

---

# 110. Ansible Exception Strategy

Ansible may return non-zero because:

```text
task failure
unreachable host
syntax error
authentication failure
```

A Python wrapper should capture:

```text
exit code
stdout
stderr
```

and classify carefully.

---

# 111. Docker Command Failures

Example:

```python
subprocess.run(
    ["docker", "build", "-t", image, "."],
    check=True,
    timeout=1800
)
```

Possible causes:

```text
Docker daemon unavailable
build failure
network pull failure
authentication
disk full
```

Log useful context but avoid dumping secrets.

---

# 112. Disk Full Exception

Possible:

```text
OSError
No space left on device
```

Response:

```text
stop writing
identify filesystem
inspect disk usage
clean according to policy
verify space
retry only after condition is resolved
```

Do not blindly retry a disk-full operation.

---

# 113. Memory Pressure

Python may fail because:

```text
MemoryError
```

But in containers, the process may instead be killed by the runtime:

```text
OOMKilled
```

No Python exception is guaranteed in the latter case.

This is an important production distinction.

---

# 114. Exception Handling Does Not Catch Process Kills

These are different:

```text
Python exception
    -> application can catch

SIGKILL / OOM kill
    -> application cannot catch normally
```

Therefore monitor:

```text
container memory
exit code
Kubernetes reason
node memory pressure
```

---

# 115. Signals

Python can handle some signals:

```python
import signal
```

For example:

```python
signal.SIGTERM
```

Useful for graceful shutdown.

But:

```text
SIGKILL
```

cannot be caught or handled.

---

# 116. Graceful Shutdown

Long-running automation/service:

```text
SIGTERM
 |
 v
stop accepting new work
 |
 v
finish/cancel current operation
 |
 v
cleanup
 |
 v
exit
```

This is different from exception handling but belongs to robust failure management.

---

# 117. `finally` and Signals

Do not assume every shutdown path behaves exactly like a normal Python exception.

External termination can occur at process level.

Use:

```text
signal handling
context managers
timeouts
external supervisor
```

as appropriate.

---

# 118. Exception Handling and Resource Leaks

Resources can include:

```text
file descriptors
temporary files
network connections
locks
child processes
threads
cloud sessions
```

Use:

```text
context managers
finally
bounded concurrency
cleanup
```

---

# 119. Lock Cleanup

If a script creates a lock:

```python
lock = acquire_lock()

try:
    deploy()
finally:
    release_lock(lock)
```

Otherwise a failure may leave stale coordination state.

For distributed locks, use the provider's lease/expiration mechanisms.

---

# 120. Temporary Credential Cleanup

If a script creates temporary credentials or files:

```text
create
 |
 v
use
 |
 v
revoke/delete
```

Cleanup should happen even when an operation fails.

Better still, prefer short-lived credentials that naturally expire.

---

# 121. Database Exceptions

Typical categories:

```text
connection failure
timeout
authentication
constraint violation
deadlock
serialization failure
syntax error
```

Only some are retryable.

For transactions:

```text
begin
 |
 v
operation
 |
 +--> success -> commit
 |
 +--> failure -> rollback
```

---

# 122. Transaction Rollback

```python
try:
    transaction()
    connection.commit()
except Exception:
    connection.rollback()
    raise
```

This pattern is essential when the database driver does not provide a context manager that already handles it.

---

# 123. Deadlocks

A database deadlock may be retryable.

Pattern:

```text
transaction
 |
 v
deadlock
 |
 v
rollback
 |
 v
backoff
 |
 v
retry
```

Use bounded retries.

Do not retry arbitrary SQL errors.

---

# 124. External API Authentication Refresh

Sometimes:

```text
token expired
 |
 v
refresh token
 |
 v
retry original request
```

But only retry once or according to the provider's documented mechanism.

Avoid infinite refresh loops.

---

# 125. Exception Handling and Secrets

Never include:

```python
raise RuntimeError(
    f"Authentication failed: {token}"
)
```

Instead:

```python
raise RuntimeError(
    "Authentication failed"
)
```

Secret values must never become exception messages.

---

# 126. Sanitizing Exceptions

An underlying exception may contain sensitive data.

Before exposing it externally:

```text
internal logs -> detailed, sanitized
user/CI message -> safe summary
```

Do not blindly print every exception to a public CI log.

---

# 127. Exception Handling Layers

Recommended architecture:

```text
Layer 1
external library/API

Layer 2
adapter translates errors

Layer 3
application domain exception

Layer 4
CLI/service boundary

Layer 5
logging + metrics + exit code
```

Example:

```text
botocore.ClientError
       |
       v
AWSAdapterError
       |
       v
DeploymentError
       |
       v
exit 1
```

---

# 128. Adapter Pattern for APIs

Instead of exposing provider-specific errors everywhere:

```python
class AWSClient:
    def get_instances(self):
        try:
            return self.client.describe_instances()
        except ClientError as exc:
            raise APIError(
                "AWS instance lookup failed"
            ) from exc
```

Application logic now handles:

```text
APIError
```

instead of provider-specific details everywhere.

---

# 129. Error Taxonomy

A practical taxonomy:

```text
CONFIGURATION
AUTHENTICATION
AUTHORIZATION
NETWORK
TIMEOUT
RATE_LIMIT
VALIDATION
NOT_FOUND
CONFLICT
DEPENDENCY
DEPLOYMENT
VERIFICATION
SECURITY
INTERNAL
```

This can drive:

```text
retry
alert
exit code
rollback
```

---

# 130. Error Handling Decision Tree

```text
Exception
   |
   v
Is it expected?
   |
  yes
   |
   v
Can it recover?
   |
 +--+--+
 |     |
yes    no
 |     |
retry  fail
 |
 v
bounded?
 |
 v
backoff + jitter
```

If unexpected:

```text
log traceback
fail safely
investigate bug
```

---

# 131. Retry Decision Table

| Failure | Usually Retry? | Typical Action |
|---|---:|---|
| Timeout | Yes | Backoff + bounded retry |
| Connection reset | Yes | Backoff |
| HTTP 429 | Yes | Honor Retry-After/backoff |
| HTTP 503 | Often | Backoff |
| HTTP 400 | No | Fix request |
| HTTP 401 | No/refresh | Fix credentials |
| HTTP 403 | No | Fix permissions |
| File missing | Usually no | Fix path/dependency |
| Permission denied | No | Fix permissions |
| Invalid JSON | No | Fix producer/input |
| Disk full | No until resolved | Free space |
| Kubernetes 409 | Re-read/reconcile | Retry with fresh state |
| Terraform state lock | Not blindly | Investigate lock |

---

# 132. `Retry-After`

Some APIs provide:

```text
Retry-After
```

Respect it when documented.

Do not always calculate your own delay if the server provides explicit guidance.

---

# 133. Retry Budget

A production retry policy should define:

```text
maximum attempts
maximum delay
maximum total duration
retryable exceptions
retryable status codes
jitter
```

Document the policy.

---

# 134. Retry Library

For larger projects, libraries such as:

```text
tenacity
```

can provide:

```text
retry predicates
wait strategies
stop conditions
logging hooks
```

But understand the policy rather than blindly decorating every function with retries.

---

# 135. Retry Anti-Pattern

Bad:

```python
@retry
def deploy():
    ...
```

without understanding:

```text
side effects
idempotency
timeouts
maximum attempts
```

A retry decorator does not make a non-idempotent deployment safe.

---

# 136. Exception Handling and Idempotent Deployment

Safer:

```text
desired state
 |
 v
read current state
 |
 v
calculate difference
 |
 v
apply
 |
 v
verify
```

This is more robust than repeatedly executing arbitrary imperative commands.

---

# 137. Desired State

DevOps systems often use desired state:

```text
desired replicas = 3
current replicas = 2
```

Automation:

```text
reconcile
```

If an operation times out:

```text
query actual state
```

before retrying.

---

# 138. Exception Handling and GitOps

With GitOps:

```text
Git desired state
       |
       v
ArgoCD reconciliation
       |
       v
Kubernetes actual state
```

If deployment reports timeout:

```text
check actual cluster state
```

before assuming failure.

Do not immediately submit the same deployment again.

---

# 139. Exception Handling and Monitoring

If deployment fails:

```text
exception
 |
 +--> log
 +--> metric
 +--> CI failure
 +--> notification
 +--> incident context
```

Monitoring should make repeated failure patterns visible.

---

# 140. Exception Handling and Alertmanager

A Python automation service may expose:

```text
automation_failures_total
```

Prometheus can alert when:

```text
failure rate > threshold
```

This is better than manually watching logs.

---

# 141. Error Budget Connection

If automation is part of a production delivery system:

```text
failed deployments
timeout rate
rollback rate
```

can contribute to operational SLOs.

Exception handling therefore affects reliability engineering, not just coding style.

---

# 142. Testing Exceptions

Use:

```python
import pytest

def test_missing_config():
    with pytest.raises(
        FileNotFoundError
    ):
        load_config(
            "missing.yaml"
        )
```

Tests should verify failure behavior, not only successful paths.

---

# 143. Testing Custom Exceptions

```python
def test_invalid_config():
    with pytest.raises(
        ConfigurationError
    ):
        validate_config({})
```

This makes the contract explicit.

---

# 144. Testing Retry Logic

Test:

```text
first attempt fails
second succeeds
```

and:

```text
all attempts fail
```

Verify:

```text
number of attempts
delay policy if mocked
final exception
```

Avoid making real tests sleep for long periods.

---

# 145. Mocking Retry Sleep

Use dependency injection or mocking for:

```python
time.sleep
```

so tests can run quickly.

The test should validate behavior rather than actually waiting 30 seconds.

---

# 146. Testing Timeout Handling

Mock the dependency to raise:

```python
TimeoutError
```

Then verify:

```text
retry occurred
eventual failure occurred
exit code is correct
cleanup executed
```

---

# 147. Testing Cleanup

Test that temporary resources are removed when:

```text
success
failure
unexpected exception
```

This catches resource-leak bugs.

---

# 148. Testing Partial Failure

For:

```text
10 services
```

simulate:

```text
8 success
2 failure
```

Verify the script:

```text
records both failures
returns correct final status
does not hide failure
does not unnecessarily stop independent work
```

---

# 149. Testing External Commands

Mock:

```python
subprocess.run
```

Simulate:

```text
success
non-zero exit
timeout
missing command
```

Verify error classification.

---

# 150. Testing AWS Failures

Mock SDK responses/exceptions:

```text
AccessDenied
Throttling
ResourceNotFound
Timeout
```

Verify:

```text
retry only throttling
fail immediately on authorization
```

---

# 151. Testing Kubernetes Failures

Simulate:

```text
404
403
409
429
503
timeout
```

Verify the intended policy for each.

---

# 152. Exception Handling and Static Analysis

Tools such as:

```text
ruff
pylint
mypy
```

can catch some error-handling problems.

Examples:

```text
unused exception variables
unreachable code
incorrect types
bad exception patterns
```

Use them as part of CI.

---

# 153. Exception Handling and Type Hints

```python
def load_config(
    path: Path
) -> dict:
    ...
```

Document exceptions when useful:

```python
def load_config(path: Path) -> dict:
    """Load configuration.

    Raises:
        FileNotFoundError: If the file is missing.
        ConfigurationError: If content is invalid.
    """
```

This makes function contracts clearer.

---

# 154. Exception Handling and Documentation

For important functions document:

```text
inputs
outputs
side effects
exceptions
retry behavior
timeouts
idempotency
```

Especially for deployment functions.

---

# 155. Production Function Contract

Example:

```text
deploy(service, version)

Inputs:
  service
  version

Success:
  desired version becomes active

Exceptions:
  ConfigurationError
  AuthenticationError
  DeploymentError
  VerificationError

Retry:
  network timeout only

Timeout:
  10 minutes

Side effects:
  modifies production workload
```

This is much clearer than a generic function description.

---

# 156. Exception Handling and Security Boundaries

Never let an exception reveal:

```text
password
token
private key
full connection string
secret headers
```

Exception messages are data.

Treat them as potentially externally visible.

---

# 157. Exception Handling and PII

Logs may contain:

```text
email
user ID
IP address
request payload
```

Do not assume traceback output is safe.

Use:

```text
redaction
structured fields
data minimization
access controls
retention
```

---

# 158. Exception Handling and Compliance

Production automation may need:

```text
audit trail
failure timestamp
actor
resource
operation
result
correlation ID
```

Do not log sensitive payloads just to create an audit trail.

Log the minimum required metadata.

---

# 159. Incident Debugging From Exceptions

When a production automation fails, collect:

```text
timestamp
service
environment
operation
error type
error message
request/deployment ID
attempt
duration
exit code
dependency status
```

This turns an exception into actionable incident evidence.

---

# 160. Example Structured Failure

```python
failure = {
    "operation": "deploy",
    "service": "payment",
    "environment": "production",
    "error_type": "TimeoutError",
    "attempt": 3,
    "status": "failed"
}
```

Write to a structured report rather than only a free-form string.

---

# 161. Avoid Error Swallowing in Helpers

Bad:

```python
def check():
    try:
        ...
    except Exception:
        return False
```

This makes:

```text
permission denied
timeout
bug
invalid data
```

all look identical.

Return structured results or raise meaningful exceptions.

---

# 162. Result Object vs Exception

Use exceptions for:

```text
exceptional failure
```

Use result objects for:

```text
expected alternative outcomes
```

Example:

```text
health check:
  healthy
  unhealthy
```

may naturally be a result.

A malformed configuration:

```text
ConfigurationError
```

should normally be an exception.

---

# 163. Health Check Example

```python
def check_service():
    response = requests.get(
        url,
        timeout=5
    )

    return response.status_code == 200
```

Here:

```text
HTTP 500
```

may be an expected health-check result rather than an unexpected program exception.

But network timeout should still be classified separately.

---

# 164. Exception vs Expected Failure

Ask:

```text
Can the caller reasonably continue?
```

If yes:

```text
return result/status
```

If no:

```text
raise exception
```

This makes APIs easier to use.

---

# 165. Exception Handling in Monitoring Scripts

Example:

```text
collect CPU
collect memory
collect disk
collect service status
```

If disk collection fails:

```text
should CPU collection also stop?
```

For independent checks, often:

```text
record disk collection failure
continue other checks
```

Then return an overall degraded status.

---

# 166. Monitoring Collection Pattern

```python
results = {}

checks = {
    "cpu": check_cpu,
    "memory": check_memory,
    "disk": check_disk
}

for name, check in checks.items():
    try:
        results[name] = check()
    except Exception as exc:
        results[name] = {
            "status": "error",
            "error": type(exc).__name__
        }
```

Be careful not to expose sensitive exception strings.

---

# 167. Production Monitoring Error Policy

```text
one metric unavailable
        |
        v
mark metric degraded
        |
        v
continue collection
        |
        v
overall health = degraded
```

This is often better than making one optional metric failure crash the whole exporter.

---

# 168. Exception Handling in Log Processors

A malformed log line should not necessarily terminate processing of a million-line log.

Pattern:

```python
for line in file:
    try:
        event = json.loads(line)
        process(event)
    except json.JSONDecodeError:
        malformed += 1
        continue
```

At the end:

```text
processed
malformed
failed
```

Report all counts.

---

# 169. When Not to Continue

Stop processing if:

```text
file is corrupted
security validation fails
input format is fundamentally wrong
data integrity cannot be trusted
continuing could cause destructive changes
```

Continue only when the remaining data can still be safely processed.

---

# 170. Exception Handling for Backups

Backup failure should be explicit:

```text
source read failure
 |
 v
backup failed
 |
 v
non-zero exit
```

Do not report backup success merely because the destination directory exists.

Verify:

```text
copy completed
size/checksum
restore test
```

where required.

---

# 171. Exception Handling for Restore

Restore is higher risk.

Pattern:

```text
validate backup
 |
 v
validate target
 |
 v
restore
 |
 v
verify
 |
 +--> success
 |
 +--> rollback/stop
```

Do not automatically overwrite production data without strong safeguards.

---

# 172. Exception Handling for Database Migration

Migration may involve:

```text
pre-check
backup
migration
verification
```

If migration fails:

```text
rollback if supported
or
restore using tested recovery procedure
```

Do not assume arbitrary database migrations can always be rolled back.

---

# 173. Exception Handling for Kubernetes Deployment

```text
apply
 |
 v
wait
 |
 v
timeout?
 |
 +--> no -> verify
 |
 +--> yes -> query actual state
                  |
                  v
             determine result
```

Never treat timeout as automatic proof of failed deployment.

---

# 174. Exception Handling for Docker Push

Potential failures:

```text
authentication
network
registry unavailable
repository missing
manifest conflict
quota
```

Retry network/transient errors selectively.

Authentication and authorization errors require credential/permission correction.

---

# 175. Exception Handling for JFrog Artifactory

Typical:

```text
401 -> authentication
403 -> permission
404 -> repository/path
429 -> rate limiting if supported
5xx -> server issue
timeout -> network/server
```

Use the repository API's documented semantics.

---

# 176. Exception Handling for Git

Python wrappers around Git may encounter:

```text
authentication
merge conflict
branch missing
repository unavailable
dirty working tree
non-fast-forward
```

Do not automatically force-push or discard changes after an exception.

---

# 177. GitOps Exception Principle

If Git is source of truth:

```text
do not "fix" production by changing files directly
```

Instead:

```text
identify failure
 |
 v
update Git
 |
 v
CI validation
 |
 v
ArgoCD reconciliation
 |
 v
verify
```

Exception handling should preserve architectural boundaries.

---

# 178. Exception Handling and Rollbacks

A rollback should itself be observable:

```text
deployment_failed
rollback_started
rollback_completed
rollback_failed
```

These should be separate states.

---

# 179. Rollback Is Not Always Safe

Rollback may fail because:

```text
old image deleted
schema changed
database migration irreversible
dependency changed
configuration changed
```

Therefore:

```text
rollback plan
```

must be tested, not merely assumed.

---

# 180. Exception Handling and Disaster Recovery

Automation failures can occur during:

```text
backup
restore
failover
DNS switch
region recovery
```

For DR automation:

```text
timeout
partial success
retries
idempotency
verification
```

must be carefully designed.

---

# 181. Exception Handling and HA

In HA systems:

```text
node A failure
 |
 v
retry/failover
 |
 v
node B
 |
 v
verify
```

Do not retry blindly if the first node may still be processing the operation.

Use health/state verification.

---

# 182. Exception Handling and Rate Limits

If an API returns:

```text
429
```

the correct response may be:

```text
respect Retry-After
reduce concurrency
backoff
jitter
```

Do not increase request rate during throttling.

---

# 183. Exception Handling and Concurrency

More concurrency can create:

```text
rate limits
connection exhaustion
file descriptor exhaustion
API throttling
CPU/memory pressure
```

Exception handling should be combined with:

```text
bounded concurrency
timeouts
backpressure
```

---

# 184. Exception Handling and Backpressure

If downstream is slow:

```text
producer
  |
  v
queue
  |
  v
slow consumer
```

Do not allow unlimited work to accumulate.

Use:

```text
bounded queues
timeouts
rate limits
load shedding
```

Long-running systems need a broader resilience design than try/except alone.

---

# 185. Common Anti-Pattern — Retry Everything

```python
except Exception:
    retry()
```

Why bad:

```text
bugs repeat
auth failures repeat
invalid data repeats
rate limits worsen
side effects duplicate
```

Classify first.

---

# 186. Common Anti-Pattern — Catch and Return `None`

```python
def load():
    try:
        ...
    except Exception:
        return None
```

Now callers cannot distinguish:

```text
missing
invalid
permission denied
API failure
bug
```

Use meaningful exceptions or explicit result types.

---

# 187. Common Anti-Pattern — Print Only

```python
except Exception as exc:
    print(exc)
```

Problems:

```text
no traceback
poor structured logging
may return exit code 0
hard to aggregate
```

Use logging plus correct exit behavior.

---

# 188. Common Anti-Pattern — Logging and Suppressing

```python
except Exception:
    logger.exception("Failed")
```

If the caller needs to know:

```python
raise
```

Otherwise the script may continue incorrectly.

---

# 189. Common Anti-Pattern — Re-Raising Wrong Type

Bad:

```python
except Exception:
    raise ValueError("something failed")
```

This destroys useful classification.

Prefer:

```python
except SomeExpectedError as exc:
    raise DomainError(
        "Deployment operation failed"
    ) from exc
```

---

# 190. Common Anti-Pattern — No Timeout

```python
requests.get(url)
```

Can hang indefinitely.

Production automation should define timeouts.

---

# 191. Common Anti-Pattern — Infinite Retry

```python
while True:
    try:
        operation()
        break
    except TimeoutError:
        continue
```

This can create:

```text
infinite pipeline
API overload
resource exhaustion
hidden incident
```

Use bounded retries.

---

# 192. Common Anti-Pattern — Retry Without Jitter

Many workers:

```text
retry at exactly 8 seconds
```

can synchronize and overload the dependency.

Use jitter where appropriate.

---

# 193. Common Anti-Pattern — Retry Non-Idempotent Operations

```text
create payment
```

followed by timeout.

Blind retry can create duplicate effects.

First determine whether the original operation completed or use idempotency keys.

---

# 194. Common Anti-Pattern — Rollback Everything

Not every failure can be safely rolled back.

Before implementing automatic rollback, identify:

```text
what changed
whether rollback is supported
dependencies
data migrations
external side effects
```

---

# 195. Common Anti-Pattern — One Giant `try`

Bad:

```python
try:
    load()
    validate()
    build()
    deploy()
    verify()
    notify()
except Exception:
    ...
```

This makes it difficult to identify:

```text
which stage failed
what can be retried
what cleanup is needed
```

Use meaningful operation boundaries.

---

# 196. Better Stage-Based Error Handling

```text
load
 |
 +--> ConfigurationError

validate
 |
 +--> ValidationError

build
 |
 +--> BuildError

deploy
 |
 +--> DeploymentError

verify
 |
 +--> VerificationError
```

Each stage has clear semantics.

---

# 197. Production Pipeline Architecture

```text
                +----------------+
                | Configuration  |
                +-------+--------+
                        |
                  exception?
                        |
                        v
                +----------------+
                |   Validation   |
                +-------+--------+
                        |
                  exception?
                        |
                        v
                +----------------+
                |     Build      |
                +-------+--------+
                        |
                  exception?
                        |
                        v
                +----------------+
                |    Deploy      |
                +-------+--------+
                        |
                  exception?
                        |
                        v
                +----------------+
                |    Verify      |
                +-------+--------+
                        |
                  exception?
                        |
                        v
                +----------------+
                | Report/Exit    |
                +----------------+
```

---

# 198. Production Error Flow

```text
Exception
   |
   v
Classify
   |
   +--> retryable
   |       |
   |       v
   |    backoff
   |       |
   |       v
   |    retry budget
   |
   +--> non-retryable
           |
           v
      cleanup
           |
           v
      log context
           |
           v
      metric/event
           |
           v
      exit/rollback
```

---

# 199. Exception Handling Checklist — Code

```text
[ ] Catch specific exceptions
[ ] Do not use bare except
[ ] Do not silently pass
[ ] Preserve original causes
[ ] Use custom exceptions where useful
[ ] Separate operational and programming failures
[ ] Use finally/context managers for cleanup
```

---

# 200. Exception Handling Checklist — Network

```text
[ ] Timeout configured
[ ] Retry policy defined
[ ] Retryable errors classified
[ ] Exponential backoff
[ ] Jitter where appropriate
[ ] Maximum attempts
[ ] Maximum total duration
[ ] Idempotency considered
```

---

# 201. Exception Handling Checklist — AWS

```text
[ ] Inspect ClientError code
[ ] Handle throttling correctly
[ ] Do not retry AccessDenied blindly
[ ] Use SDK retry behavior appropriately
[ ] Use pagination
[ ] Use IAM least privilege
[ ] Do not expose credentials
```

---

# 202. Exception Handling Checklist — Kubernetes

```text
[ ] Classify 403/404/409/429/5xx
[ ] Handle timeouts
[ ] Re-read state after uncertain operations
[ ] Consider RBAC
[ ] Consider namespace
[ ] Avoid blind retries
[ ] Verify desired state
```

---

# 203. Exception Handling Checklist — CI/CD

```text
[ ] Correct exit code
[ ] Clear stage failure
[ ] No swallowed errors
[ ] No secret leakage
[ ] Artifacts validated
[ ] Temporary files cleaned
[ ] Retry budget bounded
```

---

# 204. Exception Handling Checklist — Security

```text
[ ] No secrets in exception messages
[ ] No tokens in logs
[ ] Sanitize external errors
[ ] Avoid shell injection
[ ] Validate paths
[ ] Preserve audit information
[ ] Do not turn scanner failure into security success
```

---

# 205. Exception Handling Checklist — Reliability

```text
[ ] Timeouts
[ ] Retries
[ ] Backoff
[ ] Jitter
[ ] Idempotency
[ ] Circuit breaking where appropriate
[ ] Bounded concurrency
[ ] Cleanup
[ ] Verification
```

---

# 206. Interview — What Is Exception Handling?

Strong answer:

> "Exception handling is Python's mechanism for detecting and managing runtime failures. In DevOps automation I use it to distinguish expected operational failures such as API timeouts or missing files from programming bugs, while ensuring cleanup, useful logging, correct exit codes, and safe recovery."

---

# 207. Interview — `try`, `except`, `else`, `finally`

Strong answer:

> "`try` contains the operation, `except` handles matching exceptions, `else` runs when no exception occurs, and `finally` runs regardless of success or failure. I commonly use context managers instead of manual cleanup for files and similar resources."

---

# 208. Interview — Why Avoid `except Exception`?

Strong answer:

> "Because it can hide programming errors and make unrelated failures look identical. I prefer specific exceptions so the script can decide whether to retry, fail, continue, or escalate based on the actual failure."

---

# 209. Interview — What Is Exception Chaining?

Strong answer:

> "Exception chaining uses `raise NewError(...) from exc` to preserve the original cause while exposing a higher-level domain error. This is useful when translating AWS, Kubernetes, or library-specific errors into application-level errors without losing the traceback."

---

# 210. Interview — When Do You Retry?

Strong answer:

> "I retry only failures that are plausibly transient, such as timeouts, connection resets, throttling, and selected 5xx responses. I use bounded attempts, exponential backoff, jitter, timeouts, and consider idempotency before retrying operations with side effects."

---

# 211. Interview — Why Is Idempotency Important?

Strong answer:

> "Because a timeout does not prove the remote operation failed. A deployment request may have succeeded while the response was lost. Before retrying, I check actual state or use an idempotency mechanism so I don't create duplicate side effects."

---

# 212. Interview — How Do You Handle AWS Throttling?

Strong answer:

> "I classify the AWS error, use exponential backoff with jitter, respect SDK retry behavior and service guidance, reduce unnecessary requests, use pagination, and keep the retry budget bounded."

---

# 213. Interview — How Do You Handle Kubernetes 409?

Strong answer:

> "A 409 usually indicates a resource changed concurrently. I don't blindly retry the stale request. I fetch the latest state, recalculate the desired update, and then retry if appropriate."

---

# 214. Interview — How Do You Handle HTTP 429?

Strong answer:

> "I treat 429 as rate limiting. I respect `Retry-After` when provided, otherwise use bounded exponential backoff with jitter, and reduce concurrency if the workload is generating sustained throttling."

---

# 215. Interview — How Do You Handle a Timeout During Deployment?

Strong answer:

> "I don't immediately assume deployment failed. I query the target system to determine whether the operation completed. If it did not, I retry only if the operation is safely retryable; otherwise I reconcile or use the documented rollback procedure."

---

# 216. Interview — How Do You Make a CI Script Fail Correctly?

Strong answer:

> "I allow expected failures to reach the CLI boundary as meaningful exceptions, log a sanitized diagnostic, and return a non-zero exit code. I never catch an operational failure and exit zero simply to keep the pipeline green."

---

# 217. Interview — How Do You Prevent Secret Leakage From Exceptions?

Strong answer:

> "I avoid putting credentials in exception messages, sanitize provider errors where necessary, avoid dumping request objects, and log only safe contextual fields such as service, environment, operation, attempt, and correlation ID."

---

# 218. Interview — How Do You Handle Partial Failures?

Strong answer:

> "For independent operations I collect failures and continue, then return an overall failure if policy requires it. For dependent or transactional operations I stop at the appropriate boundary and execute a tested recovery or rollback procedure."

---

# 219. Interview — How Do You Handle Subprocess Failures?

Strong answer:

> "I use `subprocess.run(..., check=True)` when a non-zero exit should fail the operation, capture stdout/stderr when needed, set a timeout, avoid unsafe `shell=True`, and classify `CalledProcessError` and `TimeoutExpired` separately."

---

# 220. Interview — How Do You Handle a Security Scanner Failure?

Strong answer:

> "I distinguish a scanner finding from a scanner infrastructure failure. A vulnerability finding can fail a security policy, while scanner unavailability should normally fail or block the pipeline rather than be interpreted as zero vulnerabilities."

---

# 221. Interview — How Do You Handle Cleanup?

Strong answer:

> "I use context managers and `finally` blocks for resources that must be released. Cleanup errors should not hide the primary failure, so I log secondary cleanup failures while preserving the original exception."

---

# 222. Interview — How Do You Handle Exceptions in Concurrent Tasks?

Strong answer:

> "I define the failure policy before introducing concurrency: whether one failure cancels others, whether independent failures are collected, and how the overall exit status is calculated. I retrieve future results so worker exceptions are not silently lost."

---

# 223. Interview — What Is the Difference Between Retry and Rollback?

Strong answer:

> "Retry attempts the same logical operation again, usually for transient failures. Rollback restores a previous known state after an operation has changed the system. They solve different problems and require different safety considerations."

---

# 224. Interview — Can Every Failure Be Retried?

Strong answer:

> "No. Authentication, authorization, invalid input, malformed configuration, and programming bugs usually require correction rather than retry. Retrying those failures can increase load and hide the real issue."

---

# 225. Scenario — Python Script Hangs in CI

Investigate:

```text
HTTP timeout missing
subprocess timeout missing
AWS API waiting
Kubernetes API waiting
DNS issue
deadlock
infinite retry
large file processing
```

Fix:

```text
timeouts
bounded retries
progress logging
resource limits
```

---

# 226. Scenario — Script Is Green but Deployment Failed

Likely:

```text
exception swallowed
subprocess exit code ignored
API error treated as success
```

Fix:

```text
raise
check=True
correct exit code
verification
```

---

# 227. Scenario — Retry Storm

Symptoms:

```text
many workers fail
all retry together
API throttles
failure rate increases
```

Fix:

```text
exponential backoff
jitter
bounded concurrency
Retry-After
retry budget
```

---

# 228. Scenario — Deployment Duplicated Resources

Possible cause:

```text
request timed out
script assumed failure
blindly retried create
```

Fix:

```text
idempotency key
state lookup
upsert/reconcile
unique resource identity
```

---

# 229. Scenario — Cleanup Deleted the Wrong Directory

Possible cause:

```text
unsafe user-controlled path
relative path assumption
symlink
missing root validation
```

Fix:

```text
resolve
validate root boundary
reject unsafe paths
dry-run
least privilege
```

---

# 230. Scenario — Kubernetes Deployment Timeout

Investigation:

```text
kubectl get deployment
kubectl get pods
kubectl describe pod
kubectl get events
```

Then determine:

```text
image pull
scheduling
probe
resource
application startup
network
RBAC
```

Do not automatically redeploy without determining current state.

---

# 231. Scenario — AWS API Fails Intermittently

Investigate:

```text
error code
request rate
throttling
network
timeouts
credential expiry
service health
```

Apply:

```text
bounded retry
backoff
jitter
pagination
state verification
```

---

# 232. Scenario — Terraform Wrapper Fails

Capture:

```text
command
working directory
exit code
stderr
stdout if safe
Terraform version
environment
stage
```

Do not expose:

```text
TF_VAR secrets
state
credentials
```

---

# 233. Scenario — Trivy Scan Fails

Determine whether:

```text
vulnerability found
image inaccessible
registry authentication failed
scanner crashed
network unavailable
invalid output
```

Only the first is necessarily a security-policy finding.

---

# 234. Scenario — Log Parser Stops on One Bad Line

If lines are independent:

```python
for line in file:
    try:
        process(line)
    except json.JSONDecodeError:
        malformed += 1
```

At the end:

```text
processed = ...
malformed = ...
```

Fail the job if malformed data exceeds the agreed threshold.

---

# 235. Scenario — Configuration Parse Failure

Flow:

```text
read file
 |
 v
parse
 |
 +--> invalid
       |
       v
configuration error
       |
       v
stop before deployment
```

Never deploy with a guessed/default configuration unless explicitly designed.

---

# 236. Scenario — Permission Denied in Production

Do not immediately:

```text
chmod 777
```

Investigate:

```text
UID/GID
ownership
mode bits
ACL
mount
SELinux/AppArmor
container securityContext
```

Apply least-privilege correction.

---

# 237. Scenario — Disk Full During Report Generation

Possible response:

```text
catch OSError
 |
 v
check disk
 |
 v
cleanup approved temporary files
 |
 v
retry only if enough space is restored
 |
 v
otherwise fail clearly
```

Avoid endless retries.

---

# 238. Scenario — MemoryError During Processing

Reduce memory:

```text
stream
chunk
JSONL
bounded buffers
generator
pagination
```

Then inspect:

```text
container memory limit
node memory
data volume
algorithm
```

---

# 239. Scenario — External API Returns Invalid JSON

Do not retry endlessly.

Check:

```text
response status
Content-Type
body
provider issue
proxy
authentication page returned unexpectedly
```

Log sanitized metadata and fail if the response contract is violated.

---

# 240. Scenario — Exception During Rollback

Report:

```text
primary failure
rollback attempted
rollback result
current system state
```

Escalate because the system may be in an uncertain state.

---

# 241. Production Incident Exception Flow

```text
Failure
  |
  v
Classify
  |
  +--> transient
  |      |
  |      v
  |   bounded retry
  |
  +--> permanent
  |      |
  |      v
  |   stop/recover
  |
  +--> uncertain
         |
         v
     query actual state
         |
         v
      reconcile
         |
         v
       verify
```

The "uncertain" category is especially important for distributed systems.

---

# 242. Exception Handling and Distributed Systems

The hardest failures are not:

```text
success
failure
```

but:

```text
unknown result
```

Example:

```text
request sent
 |
 v
server processes
 |
 v
network connection breaks
 |
 v
client receives timeout
```

The client cannot assume the server did nothing.

Always consider uncertain outcomes.

---

# 243. Distributed Systems Rule

After a timeout:

```text
Do not immediately repeat side effects.
```

Instead:

```text
check state
use idempotency key
use request ID
reconcile
```

This is one of the most important production concepts for DevOps automation.

---

# 244. Exception Handling and State Machines

Deployment can be represented as:

```text
PENDING
  |
  v
PREPARING
  |
  v
DEPLOYING
  |
  v
VERIFYING
  |
  +--> SUCCESS
  |
  +--> FAILED
  |
  +--> ROLLBACK
          |
          v
      ROLLBACK_FAILED
```

This is often safer than treating the deployment as one giant function.

---

# 245. State-Aware Error Recovery

If current state is:

```text
DEPLOYING
```

do not automatically start another deployment.

First determine:

```text
current version
pod status
rollout status
deployment controller state
```

Then decide.

---

# 246. Exception Handling and Reconciliation

A robust DevOps automation pattern is:

```text
desired state
      |
      v
observe actual state
      |
      v
calculate difference
      |
      v
apply change
      |
      v
observe again
      |
      v
converged?
```

If an exception occurs, return to observation rather than assuming state.

---

# 247. Exception Handling and GitLab CI Retry

GitLab can retry some jobs, but application-level retry still needs careful design.

Avoid:

```text
CI retries whole deployment
+
Python retries deployment
+
SDK retries API
```

This can multiply attempts.

Define retry ownership.

---

# 248. Retry Layering

Potential layers:

```text
SDK retry
HTTP client retry
Python application retry
CI job retry
```

If all retry independently:

```text
3 x 3 x 2 = many attempts
```

Choose the layer(s) intentionally.

---

# 249. Retry Ownership

A good design may be:

```text
SDK -> handles low-level transient API retries
Python -> handles business-level recovery
CI -> handles only safe whole-job reruns
```

Document this.

---

# 250. Exception Handling and Alert Fatigue

Do not alert on every transient exception.

Instead:

```text
single timeout
    -> retry

repeated timeout
    -> metric

sustained failure
    -> alert
```

Use monitoring thresholds rather than paging on every log line.

---

# 251. Production Error Budget

Track:

```text
failure rate
retry rate
timeout rate
rollback rate
mean recovery time
```

If automation repeatedly fails, fixing the underlying dependency is better than increasing retries indefinitely.

---

# 252. Exception Handling Documentation

Every production automation should document:

```text
expected exceptions
retryable exceptions
non-retryable exceptions
timeouts
exit codes
cleanup
rollback
idempotency
```

This reduces operational ambiguity.

---

# 253. Code Review Checklist

Reviewers should ask:

```text
What happens if the file is missing?
What happens if permission is denied?
What happens if the API times out?
What happens if the request actually succeeded?
What happens if retry occurs?
What happens if rollback fails?
What happens if cleanup fails?
What exit code is returned?
Can secrets leak?
```

---

# 254. Production Readiness Checklist

```text
[ ] Specific exceptions
[ ] Clear exception hierarchy
[ ] Context preserved
[ ] Timeouts
[ ] Retry classification
[ ] Backoff
[ ] Jitter
[ ] Idempotency
[ ] Cleanup
[ ] Correct exit codes
[ ] Structured logs
[ ] Secret redaction
[ ] State verification
[ ] Tests
[ ] Monitoring
[ ] Documentation
```

---

# 255. Practical Exercise 1 — File Loader

Build:

```python
load_json(path)
```

Requirements:

```text
FileNotFoundError
PermissionError
JSONDecodeError
```

Convert them into meaningful application-level errors.

---

# 256. Practical Exercise 2 — Retry Utility

Build:

```python
retry(
    operation,
    attempts=3,
    timeout=...
)
```

Support:

```text
exponential backoff
jitter
maximum delay
specific retryable exceptions
```

---

# 257. Practical Exercise 3 — AWS Wrapper

Create:

```python
describe_instances()
```

Classify:

```text
throttling
access denied
not found
timeout
```

Implement safe retry behavior.

---

# 258. Practical Exercise 4 — Kubernetes Wrapper

Create:

```python
wait_for_rollout()
```

Handle:

```text
timeout
404
403
409
API unavailable
```

After timeout, query current state before deciding what to do.

---

# 259. Practical Exercise 5 — Subprocess Runner

Build:

```python
run_command(
    command,
    timeout=60
)
```

Requirements:

```text
non-zero exit
timeout
missing executable
stdout/stderr
safe argument handling
```

---

# 260. Practical Exercise 6 — Deployment CLI

Create:

```text
deploy.py
```

Flow:

```text
load config
 |
 v
validate
 |
 v
build
 |
 v
deploy
 |
 v
verify
 |
 v
report
```

Create custom exceptions for each major stage.

---

# 261. Practical Exercise 7 — Failure Classification

Given:

```text
HTTP 429
HTTP 403
HTTP 500
TimeoutError
FileNotFoundError
JSONDecodeError
```

Classify:

```text
retry
fail
reconcile
```

and explain why.

---

# 262. Practical Exercise 8 — Partial Failure

Deploy to:

```text
service-a
service-b
service-c
service-d
```

Simulate:

```text
A success
B timeout
C success
D permission denied
```

Collect failures and return the correct final exit code.

---

# 263. Practical Exercise 9 — Cleanup

Create a temporary directory.

Run:

```text
success
failure
unexpected exception
```

Verify the directory is cleaned in every expected path.

---

# 264. Practical Exercise 10 — Retry + Idempotency

Simulate:

```text
request sent
response timeout
```

Then make the server state show:

```text
operation succeeded
```

Your client must detect the completed state and avoid duplicating the operation.

---

# 265. Practical Exercise 11 — Security Scanner

Simulate:

```text
vulnerability found
scanner unavailable
authentication failure
invalid output
```

Ensure:

```text
vulnerability -> policy failure
scanner unavailable -> infrastructure failure
auth failure -> credential failure
invalid output -> pipeline failure
```

---

# 266. Practical Exercise 12 — CI Exit Codes

Create:

```text
0 success
1 operational failure
2 configuration failure
3 verification failure
```

Test each path.

---

# 267. Practical Exercise 13 — Structured Errors

Generate:

```json
{
  "operation": "deploy",
  "service": "payment",
  "environment": "production",
  "error_type": "TimeoutError",
  "attempt": 3,
  "status": "failed"
}
```

Do not include secrets.

---

# 268. Practical Exercise 14 — Exception Tests

Write pytest tests for:

```text
missing config
invalid JSON
permission failure
API timeout
retry success
retry exhaustion
cleanup failure
correct exit code
```

---

# 269. Practical Exercise 15 — Production Incident Simulator

Simulate:

```text
deployment request
 |
 v
timeout
 |
 v
query Kubernetes
 |
 v
deployment actually completed
 |
 v
verification
 |
 v
success
```

This teaches why timeout does not necessarily equal failure.

---

# 270. Mini Project — Resilient Deployment CLI

Build:

```text
deploy.py
```

Features:

```text
configuration validation
timeouts
retry
backoff
jitter
idempotency/state check
structured logging
custom exceptions
cleanup
verification
exit codes
```

---

# 271. Mini Project — AWS Resource Reporter

Build:

```text
aws_report.py
```

Features:

```text
pagination
timeouts
retry classification
throttling handling
IAM error handling
JSON output
CSV output
structured errors
```

---

# 272. Mini Project — Kubernetes Rollout Verifier

Build:

```text
rollout.py
```

Features:

```text
deployment lookup
rollout monitoring
timeout
409 handling
404 handling
403 handling
pod verification
final report
exit code
```

---

# 273. Mini Project — CI Security Gate

Inputs:

```text
Trivy
SonarQube
Veracode
```

Process:

```text
load
 |
 v
validate
 |
 v
classify
 |
 v
policy
 |
 v
report
 |
 v
exit
```

Do not turn scanner failures into false security success.

---

# 274. Mini Project — Production Health Collector

Collect:

```text
CPU
memory
disk
service status
HTTP health
Kubernetes status
```

Handle independent failures separately.

Output:

```json
{
  "overall": "degraded",
  "checks": {
    "cpu": "ok",
    "memory": "ok",
    "disk": "error",
    "http": "ok"
  }
}
```

---

# 275. Production Architecture — Resilient Python Automation

```text
                    +----------------+
                    |      CLI       |
                    +-------+--------+
                            |
                            v
                    +---------------+
                    | Configuration |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    |  Validation   |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Domain Logic  |
                    +-------+-------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
            AWS         Kubernetes     Linux/API
              |             |             |
              +-------------+-------------+
                            |
                            v
                    +---------------+
                    | Error Adapter |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Retry/Recover |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Verify State  |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Report/Metric |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Exit / Alert  |
                    +---------------+
```

---

# 276. Production Resilience Layers

```text
Layer 1
input validation

Layer 2
timeouts

Layer 3
exception classification

Layer 4
bounded retry

Layer 5
backoff + jitter

Layer 6
idempotency

Layer 7
state reconciliation

Layer 8
verification

Layer 9
rollback/recovery

Layer 10
observability
```

Exception handling is one layer of a larger resilience architecture.

---

# 277. Final Mental Model

Remember:

```text
TRY
 |
 v
CLASSIFY
 |
 +--> EXPECTED
 |      |
 |      v
 |   RECOVER/RETRY
 |
 +--> PERMANENT
 |      |
 |      v
 |     FAIL
 |
 +--> UNCERTAIN
        |
        v
    CHECK STATE
        |
        v
     RECONCILE
        |
        v
      VERIFY
```

---

# 278. Final DevOps Takeaway

Good exception handling is not:

```python
try:
    ...
except:
    print("error")
```

Production-grade exception handling means:

```text
specific failures
+
safe recovery
+
bounded retries
+
timeouts
+
backoff
+
idempotency
+
state verification
+
cleanup
+
security
+
observability
+
correct exit codes
```

The most important DevOps principle is:

> **Do not blindly retry an uncertain operation. Determine the actual system state first.**

That principle prevents duplicate deployments, duplicate resources, inconsistent infrastructure, and many difficult production incidents.

---

# 279. Next File

```text
22-Python-for-DevOps/
└── 01-Python-Fundamentals/
    ├── 01-Python-Introduction.md
    ├── 02-Variables-and-Data-Types.md
    ├── 03-Operators.md
    ├── 04-Conditional-Statements.md
    ├── 05-Loops.md
    ├── 06-Functions.md
    ├── 07-Data-Structures.md
    ├── 08-File-Handling.md
    └── 09-Exception-Handling.md
```

Next:

```text
10-Modules-and-Packages.md
```

Focus:

```text
import
from/import
module structure
__name__
__main__
packages
__init__.py
standard library
third-party packages
pip
requirements.txt
constraints
virtual environments
dependency management
package versioning
private packages
JFrog Artifactory
CI/CD dependency installation
reproducible environments
security
dependency scanning
Python project structure
DevOps automation architecture
interview questions
scenario-based questions
practical exercises
production project
```
