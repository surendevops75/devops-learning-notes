# 22-Python-for-DevOps
# 01-Python-Fundamentals
# 05-Loops

> Deep loop notes for DevOps engineers. Focus: automation, AWS/Kubernetes inventory, API pagination, polling, retries, batch processing, log processing, CI/CD, monitoring, production safety, troubleshooting, interview questions, and practical exercises.

---

# 1. What Are Loops?

Loops allow Python to execute a block of code repeatedly.

DevOps automation frequently needs repetition:

```text
Process every EC2 instance
Check every Kubernetes pod
Read every log line
Process every file
Run checks against multiple services
Poll a deployment until completion
Retry a transient operation
Process API pagination
Validate every configuration item
Generate reports from many resources
```

The two primary loop types are:

```text
for
while
```

---

# 2. `for` Loop

A `for` loop iterates over an iterable.

Example:

```python
services = ["user", "cart", "payment"]

for service in services:
    print(service)
```

Output:

```text
user
cart
payment
```

The loop executes once for each item.

---

# 3. Basic `for` Loop Flow

```text
Iterable
   |
   v
Get next item
   |
   v
Execute loop body
   |
   v
More items?
  / \
Yes  No
 |    |
Loop  Exit
```

This is one of the most important patterns in Python automation.

---

# 4. Iterating Over a List

```python
servers = [
    "web-01",
    "web-02",
    "web-03"
]

for server in servers:
    print(f"Checking {server}")
```

This can be extended to:

```text
Health checks
SSH operations
Inventory
Monitoring
Configuration validation
```

---

# 5. Iterating Over a Tuple

```python
regions = (
    "us-east-1",
    "us-west-2",
    "ap-south-1"
)

for region in regions:
    print(region)
```

The loop does not care whether the iterable is a list or tuple.

---

# 6. Iterating Over a Set

```python
environments = {
    "dev",
    "stage",
    "production"
}

for environment in environments:
    print(environment)
```

Remember that sets are unordered collections.

Do not rely on set iteration order.

---

# 7. Iterating Over a String

Strings are iterable:

```python
service = "nginx"

for character in service:
    print(character)
```

Output:

```text
n
g
i
n
x
```

This is useful in some parsing tasks, but do not accidentally iterate over characters when you intended to process a whole string.

---

# 8. Iterating Over a Dictionary

Given:

```python
server = {
    "name": "web-01",
    "environment": "production",
    "status": "running"
}
```

A direct loop iterates over keys:

```python
for key in server:
    print(key)
```

Output:

```text
name
environment
status
```

---

# 9. Dictionary Keys

Explicit form:

```python
for key in server.keys():
    print(key)
```

Usually:

```python
for key in server:
    print(key)
```

is sufficient.

---

# 10. Dictionary Values

```python
for value in server.values():
    print(value)
```

This processes:

```text
web-01
production
running
```

Use this when keys are not required.

---

# 11. Dictionary Keys and Values

Use `.items()`:

```python
for key, value in server.items():
    print(f"{key}={value}")
```

Output:

```text
name=web-01
environment=production
status=running
```

This is extremely common in configuration and API processing.

---

# 12. Iterating Over API Response Data

Example:

```python
resources = [
    {"name": "ec2-01", "state": "running"},
    {"name": "ec2-02", "state": "stopped"},
    {"name": "ec2-03", "state": "running"}
]

for resource in resources:
    print(resource["name"])
```

This is similar to how cloud SDK responses are processed.

---

# 13. `range()`

`range()` generates a sequence of numbers.

```python
for i in range(5):
    print(i)
```

Output:

```text
0
1
2
3
4
```

The stop value is exclusive.

---

# 14. `range(start, stop)`

```python
for i in range(1, 5):
    print(i)
```

Output:

```text
1
2
3
4
```

Useful for:

```text
Retry attempts
Batch numbers
Resource indexes
Pagination
```

---

# 15. `range(start, stop, step)`

```python
for i in range(0, 10, 2):
    print(i)
```

Output:

```text
0
2
4
6
8
```

---

# 16. Reverse `range()`

```python
for i in range(5, 0, -1):
    print(i)
```

Output:

```text
5
4
3
2
1
```

Useful for countdown logic.

---

# 17. `range()` Does Not Store Every Number

`range()` is a range object rather than a full list.

```python
numbers = range(1_000_000)
```

This is memory-efficient compared with creating a million-element list just to iterate.

---

# 18. `enumerate()`

Use `enumerate()` when you need both index and value.

```python
services = [
    "user",
    "cart",
    "payment"
]

for index, service in enumerate(services):
    print(index, service)
```

Output:

```text
0 user
1 cart
2 payment
```

---

# 19. `enumerate(start=1)`

For human-readable numbering:

```python
for number, service in enumerate(
    services,
    start=1
):
    print(number, service)
```

Output:

```text
1 user
2 cart
3 payment
```

Useful for reports.

---

# 20. `zip()`

`zip()` combines iterables.

```python
services = ["user", "cart", "payment"]
ports = [8080, 8081, 8082]

for service, port in zip(services, ports):
    print(service, port)
```

Output:

```text
user 8080
cart 8081
payment 8082
```

---

# 21. `zip()` for Configuration

```python
services = ["user", "cart", "payment"]
replicas = [2, 3, 2]

for service, replica_count in zip(
    services,
    replicas
):
    print(
        f"{service}: {replica_count}"
    )
```

This is useful when related data is stored in parallel collections.

---

# 22. `zip()` Length Behavior

By default, `zip()` stops when the shortest iterable ends.

```python
a = [1, 2, 3]
b = ["a", "b"]

for x, y in zip(a, b):
    print(x, y)
```

Only:

```text
1 a
2 b
```

are processed.

If unequal lengths indicate a configuration problem, validate them explicitly.

---

# 23. `zip(strict=True)`

Modern Python supports:

```python
for service, port in zip(
    services,
    ports,
    strict=True
):
    ...
```

With unequal lengths, Python raises an error instead of silently truncating.

This can be valuable when parallel configuration lists must have exactly matching lengths.

---

# 24. `break`

`break` exits the loop immediately.

```python
for service in services:
    if service == "payment":
        break

    print(service)
```

The loop stops when `payment` is reached.

---

# 25. `break` in Health Checks

```python
for service in services:
    if not check_health(service):
        print(
            f"First unhealthy service: {service}"
        )
        break
```

Use this when the first failure is enough to stop processing.

---

# 26. `continue`

`continue` skips the current iteration and moves to the next.

```python
for service in services:
    if service == "maintenance":
        continue

    deploy(service)
```

The maintenance service is skipped.

---

# 27. `continue` for Filtering

```python
for server in servers:
    if server["state"] != "running":
        continue

    monitor(server)
```

This keeps the main operation path simple.

---

# 28. `pass`

`pass` does nothing.

```python
for service in services:
    if service == "legacy":
        pass
    else:
        deploy(service)
```

Usually `pass` is more useful as a placeholder during development:

```python
def future_function():
    pass
```

Do not confuse:

```text
pass
continue
break
```

They have different behavior.

---

# 29. Difference Between `break`, `continue`, and `pass`

```text
break
  -> exits loop

continue
  -> skips current iteration

pass
  -> does nothing
```

Example:

```python
for item in items:
    if item == "stop":
        break

    if item == "skip":
        continue

    if item == "placeholder":
        pass

    process(item)
```

---

# 30. `while` Loop

A `while` loop runs while a condition remains true.

```python
attempt = 0

while attempt < 3:
    attempt += 1
    print(attempt)
```

Output:

```text
1
2
3
```

---

# 31. `while` Loop Flow

```text
Check condition
      |
      v
    True?
    /   \
  Yes    No
   |      |
Body     Exit
   |
   +----> Check condition again
```

A `while` loop is useful when the number of iterations is not known in advance.

---

# 32. `for` vs `while`

Use `for` when:

```text
You have an iterable
You know what collection to process
You need one pass over items
```

Use `while` when:

```text
You wait for a condition
You poll an external system
You retry until success/timeout
The number of iterations is dynamic
```

---

# 33. Infinite `while` Loop

This can run forever:

```python
while True:
    check_status()
```

Sometimes this is intentional, such as a long-running worker.

But production loops need a clear termination strategy when they are request/automation workflows.

---

# 34. Safe Polling Loop

```python
import time

attempt = 0
max_attempts = 30

while attempt < max_attempts:
    status = get_status()

    if status == "complete":
        break

    attempt += 1
    time.sleep(10)
else:
    raise TimeoutError(
        "Operation did not complete"
    )
```

This has a bounded number of polls.

---

# 35. `while` With Timeout

A time-based timeout is often better than only counting attempts.

```python
import time

deadline = time.monotonic() + 300

while time.monotonic() < deadline:
    status = get_status()

    if status == "complete":
        break

    time.sleep(10)
else:
    raise TimeoutError(
        "Operation timed out"
    )
```

Use `time.monotonic()` for elapsed-time measurements because it is not affected by wall-clock adjustments.

---

# 36. Polling Kubernetes Rollout

Conceptual example:

```python
import time

deadline = time.monotonic() + 600

while time.monotonic() < deadline:
    status = get_rollout_status()

    if status == "complete":
        print("Rollout completed")
        break

    if status == "failed":
        raise RuntimeError(
            "Rollout failed"
        )

    time.sleep(10)
else:
    raise TimeoutError(
        "Rollout timed out"
    )
```

This is a realistic DevOps automation pattern.

---

# 37. Polling AWS Resource State

Example:

```python
deadline = time.monotonic() + 300

while time.monotonic() < deadline:
    state = get_instance_state()

    if state == "running":
        print("Instance is running")
        break

    if state == "terminated":
        raise RuntimeError(
            "Instance terminated unexpectedly"
        )

    time.sleep(5)
else:
    raise TimeoutError(
        "Instance did not reach running state"
    )
```

---

# 38. Polling Rules

Production polling should define:

```text
Expected success state
Expected failure state
Timeout
Polling interval
Transient states
API retry behavior
Logging
```

Avoid:

```python
while True:
    ...
```

without a clear termination or service lifecycle design.

---

# 39. Retry Loop

Basic pattern:

```python
max_attempts = 3

for attempt in range(1, max_attempts + 1):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == max_attempts:
            raise
```

This gives exactly three attempts.

---

# 40. Retry With Backoff

```python
import time

base_delay = 2
max_delay = 30
max_attempts = 5

for attempt in range(max_attempts):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == max_attempts - 1:
            raise

        delay = min(
            base_delay * (2 ** attempt),
            max_delay
        )

        time.sleep(delay)
```

Production distributed systems should normally add jitter.

---

# 41. Retry Only Retryable Errors

Do not retry everything.

```python
try:
    call_api()
except RateLimitError:
    retry()
except TimeoutError:
    retry()
except AuthenticationError:
    raise
except InvalidRequestError:
    raise
```

The retry policy should distinguish transient from permanent failures.

---

# 42. Retry With Jitter

A common concept:

```python
import random

delay = min(
    base_delay * (2 ** attempt),
    max_delay
)

delay += random.uniform(0, 1)

time.sleep(delay)
```

Jitter helps reduce synchronized retries from many clients.

In production, use the retry/backoff behavior recommended by the specific API/client library when available.

---

# 43. Retry Budget

A retry policy should limit:

```text
Maximum attempts
Maximum total elapsed time
Maximum delay
```

Example:

```python
max_attempts = 5
max_delay = 30
```

This prevents a failing dependency from consuming resources indefinitely.

---

# 44. API Pagination

Many APIs return results in pages.

Concept:

```text
Request page
   |
   v
Process results
   |
   v
More pages?
 /       \
Yes       No
 |         |
Next      Done
page
```

A loop is ideal for this.

---

# 45. Page-Based Pagination

```python
page = 1

while True:
    response = get_resources(page)

    for resource in response["items"]:
        process(resource)

    if not response["has_next"]:
        break

    page += 1
```

Always confirm the API's actual pagination mechanism.

---

# 46. Token-Based Pagination

Many cloud APIs use a continuation token.

```python
token = None

while True:
    response = get_resources(
        next_token=token
    )

    for resource in response["items"]:
        process(resource)

    token = response.get("next_token")

    if not token:
        break
```

This pattern is common in cloud APIs.

---

# 47. Pagination Safety

Production pagination should handle:

```text
Empty page
Repeated token
API errors
Rate limits
Maximum page count
Timeout
Malformed response
```

A repeated continuation token can cause an infinite loop.

---

# 48. Detecting a Repeated Pagination Token

```python
token = None
previous_token = object()

while token != previous_token:
    previous_token = token

    response = get_resources(
        next_token=token
    )

    process(response["items"])

    token = response.get("next_token")

    if not token:
        break
```

An even stronger implementation can explicitly track all seen tokens and fail if a token repeats unexpectedly.

---

# 49. Processing Files

```python
from pathlib import Path

for file in Path("/var/log").glob("*.log"):
    print(file)
```

Useful for:

```text
Log collection
File validation
Backup scripts
Cleanup
Report generation
```

---

# 50. Reading a Log File Line by Line

Do not load a huge log file into memory unnecessarily.

Prefer:

```python
with open("application.log") as file:
    for line in file:
        if "ERROR" in line:
            print(line.strip())
```

This processes the file incrementally.

---

# 51. Large Log Processing

For large logs:

```python
with open("application.log") as file:
    for line in file:
        process_line(line)
```

Memory usage remains approximately independent of total file size, aside from the current line and application state.

---

# 52. Counting Errors

```python
error_count = 0

with open("application.log") as file:
    for line in file:
        if "ERROR" in line:
            error_count += 1

print(error_count)
```

This is a basic observability automation example.

---

# 53. Counting Status Codes

```python
status_counts = {}

with open("access.log") as file:
    for line in file:
        status = extract_status(line)

        status_counts[status] = (
            status_counts.get(status, 0) + 1
        )
```

This builds a simple report.

---

# 54. Log Filtering

```python
with open("application.log") as file:
    for line in file:
        if "ERROR" not in line:
            continue

        process_error(line)
```

`continue` keeps the main path focused.

---

# 55. Processing Multiple Services

```python
services = [
    "user",
    "product",
    "cart",
    "order",
    "payment"
]

for service in services:
    status = check_health(service)

    if status != "healthy":
        print(
            f"{service}: unhealthy"
        )
```

This is a common microservices automation pattern.

---

# 56. Stop at First Failure

```python
for service in services:
    if not check_health(service):
        print(
            f"First failure: {service}"
        )
        break
```

Use this when later checks are not useful after the first failure.

---

# 57. Continue Past Failed Services

```python
failed = []

for service in services:
    if not check_health(service):
        failed.append(service)
        continue

    print(
        f"{service}: healthy"
    )
```

This is better when you want a complete health report.

---

# 58. Complete Service Health Report

```python
results = {}

for service in services:
    results[service] = check_health(service)

for service, healthy in results.items():
    status = (
        "HEALTHY"
        if healthy
        else "UNHEALTHY"
    )

    print(
        f"{service}: {status}"
    )
```

This separates collection from reporting.

---

# 59. Nested Loops

Example:

```python
environments = [
    "dev",
    "stage"
]

services = [
    "user",
    "payment"
]

for environment in environments:
    for service in services:
        print(
            environment,
            service
        )
```

Output:

```text
dev user
dev payment
stage user
stage payment
```

Nested loops can become expensive, so use them deliberately.

---

# 60. Nested Loops for AWS Inventory

Concept:

```python
for region in regions:
    resources = list_resources(region)

    for resource in resources:
        process(resource)
```

This is a common cloud inventory pattern.

---

# 61. Nested Loops for Kubernetes

Concept:

```python
for namespace in namespaces:
    pods = list_pods(namespace)

    for pod in pods:
        inspect_pod(pod)
```

This can generate cluster-wide reports.

---

# 62. Complexity of Nested Loops

If:

```text
10 regions
100 resources per region
```

then approximately:

```text
10 × 100 = 1000 resource iterations
```

With larger datasets, API calls may become the bottleneck rather than Python itself.

---

# 63. Avoid Repeated API Calls Inside Deep Loops

Potentially inefficient:

```python
for pod in pods:
    nodes = get_nodes()

    for node in nodes:
        ...
```

If `get_nodes()` always returns the same data, fetch it once:

```python
nodes = get_nodes()

for pod in pods:
    for node in nodes:
        ...
```

Better architecture often means minimizing unnecessary network calls.

---

# 64. Looping Over API Results

```python
response = client.list_resources()

for resource in response["resources"]:
    if resource["state"] == "running":
        process_running(resource)
```

Validate the response structure before assuming fields exist when the API can return incomplete data.

---

# 65. Batch Processing

Suppose:

```python
resources = list_resources()
```

Process in batches:

```python
batch_size = 50

for start in range(
    0,
    len(resources),
    batch_size
):
    batch = resources[
        start:start + batch_size
    ]

    process_batch(batch)
```

Useful for:

```text
API calls
Database writes
Cloud resources
Notifications
Bulk updates
```

---

# 66. Batch Processing Why?

Instead of:

```text
1000 individual operations
```

you may be able to perform:

```text
20 batches × 50 items
```

This can reduce:

```text
Network overhead
API calls
Transaction overhead
Logging noise
```

Only use batch APIs when the target system supports them safely.

---

# 67. Batch Failure Handling

A batch operation needs a policy:

```text
All-or-nothing
Continue after failed item
Retry failed items
Abort after threshold
```

Example:

```python
for item in batch:
    try:
        process(item)
    except TemporaryError:
        retry_item(item)
    except PermanentError:
        record_failure(item)
```

---

# 68. Looping Over Configuration

```python
required_variables = [
    "AWS_REGION",
    "ECR_REPOSITORY",
    "CLUSTER_NAME"
]

for variable in required_variables:
    if not os.getenv(variable):
        raise RuntimeError(
            f"Missing {variable}"
        )
```

This is a common CI/CD validation script.

---

# 69. Environment Validation

```python
required = {
    "AWS_REGION",
    "ENVIRONMENT",
    "IMAGE_TAG"
}

for key in required:
    value = os.getenv(key)

    if not value:
        raise RuntimeError(
            f"Missing environment variable: {key}"
        )
```

Be careful if empty string is a valid value; validate according to the actual contract.

---

# 70. Looping Over Required Files

```python
required_files = [
    "Dockerfile",
    "requirements.txt",
    "deployment.yaml"
]

for filename in required_files:
    path = Path(filename)

    if not path.is_file():
        raise FileNotFoundError(
            f"Missing {filename}"
        )
```

Useful in CI pipelines.

---

# 71. Looping Over Security Checks

```python
checks = [
    run_sast,
    run_dependency_scan,
    run_image_scan
]

for check in checks:
    if not check():
        raise SystemExit(
            "Security gate failed"
        )
```

This creates a simple sequential security gate.

---

# 72. Continue vs Fail-Fast in CI/CD

Fail-fast:

```python
for check in checks:
    if not check():
        raise SystemExit(1)
```

Collect all failures:

```python
failures = []

for check in checks:
    if not check():
        failures.append(check.__name__)

if failures:
    raise SystemExit(
        f"Failed checks: {failures}"
    )
```

Choose based on whether developers need one failure quickly or a complete report.

---

# 73. Looping Over Terraform Resources

Conceptually:

```python
resources = get_terraform_resources()

for resource in resources:
    if resource["drifted"]:
        print(
            f"Drift detected: "
            f"{resource['name']}"
        )
```

Python can automate validation around Terraform plans and state outputs.

---

# 74. Looping Over Kubernetes Pods

Conceptually:

```python
pods = get_pods()

for pod in pods:
    if pod["phase"] != "Running":
        print(
            f"{pod['name']} "
            f"is {pod['phase']}"
        )
```

Production checks should also inspect readiness, restart count, container state, and events where relevant.

---

# 75. Detecting CrashLoopBackOff

```python
for pod in pods:
    if pod["reason"] == "CrashLoopBackOff":
        print(
            f"Investigate {pod['name']}"
        )
```

A script can then collect:

```text
Current logs
Previous logs
Events
Container state
Exit code
Restart count
```

---

# 76. Looping Over Nodes

```python
for node in nodes:
    if node["ready"] is not True:
        print(
            f"Node unhealthy: "
            f"{node['name']}"
        )
```

Do not treat missing `ready` metadata as healthy.

---

# 77. Node Resource Checks

```python
for node in nodes:
    if node["cpu_usage"] >= 90:
        print(
            f"High CPU: {node['name']}"
        )

    if node["memory_usage"] >= 90:
        print(
            f"High memory: {node['name']}"
        )

    if node["disk_usage"] >= 90:
        print(
            f"High disk: {node['name']}"
        )
```

Independent conditions allow multiple alerts per node.

---

# 78. Looping Over Microservices

```python
services = {
    "user": 8080,
    "product": 8081,
    "cart": 8082,
    "order": 8083
}

for service, port in services.items():
    check_endpoint(
        service,
        port
    )
```

This is directly applicable to microservices platforms.

---

# 79. Looping Over Deployment Environments

```python
environments = [
    "dev",
    "stage",
    "production"
]

for environment in environments:
    print(
        f"Validating {environment}"
    )
```

Production may require additional safeguards:

```python
for environment in environments:
    if environment == "production":
        validate_approval()

    validate_environment(environment)
```

---

# 80. Looping Over AWS Regions

```python
regions = [
    "ap-south-1",
    "us-east-1"
]

for region in regions:
    resources = list_resources(
        region=region
    )

    process_resources(resources)
```

Be mindful of:

```text
API rate limits
Cross-region permissions
Region-specific resources
Network latency
```

---

# 81. Parallelism Warning

A simple loop is sequential:

```python
for region in regions:
    scan_region(region)
```

If scanning many regions is slow, concurrency may help.

But do not immediately replace loops with parallelism.

First identify:

```text
I/O bottleneck
API rate limits
Dependency limits
Concurrency safety
```

---

# 82. Loop and Concurrency

Sequential:

```text
Region A -> wait -> Region B -> wait -> Region C
```

Concurrent:

```text
Region A
Region B
Region C
  | | |
  v v v
results
```

Python provides concurrency tools, but those belong to a later topic.

---

# 83. Looping Through JSON

```python
data = {
    "services": [
        {"name": "user", "status": "healthy"},
        {"name": "payment", "status": "failed"}
    ]
}

for service in data["services"]:
    print(
        service["name"],
        service["status"]
    )
```

This is common when processing API output.

---

# 84. Looping Through YAML-Derived Data

After parsing YAML:

```python
config = {
    "services": [
        {"name": "user", "replicas": 2},
        {"name": "payment", "replicas": 3}
    ]
}

for service in config["services"]:
    print(
        service["name"],
        service["replicas"]
    )
```

Python processes the resulting data structure regardless of whether it originated from JSON or YAML.

---

# 85. Looping Through Environment Variables

```python
for key, value in os.environ.items():
    print(
        f"{key}={value}"
    )
```

Be careful:

```text
Environment variables may contain secrets.
```

Do not blindly print production environment variables.

---

# 86. Secure Environment Variable Loop

If you need diagnostics:

```python
sensitive = {
    "PASSWORD",
    "TOKEN",
    "SECRET",
    "API_KEY"
}

for key, value in os.environ.items():
    if any(
        word in key.upper()
        for word in sensitive
    ):
        print(f"{key}=***")
    else:
        print(f"{key}={value}")
```

A stronger approach is to use an explicit allowlist of variables safe to log.

---

# 87. Looping Through Log Files Securely

```python
for path in Path("/var/log").glob("*.log"):
    print(
        f"Processing {path.name}"
    )
```

Avoid logging sensitive contents without redaction.

---

# 88. Generator-Based Processing

A generator can produce items incrementally.

Example:

```python
def read_errors(path):
    with open(path) as file:
        for line in file:
            if "ERROR" in line:
                yield line
```

Then:

```python
for error in read_errors(
    "application.log"
):
    process_error(error)
```

This is useful for large datasets.

---

# 89. Why Generators Matter in DevOps

Generators can help with:

```text
Large log files
Large API result sets
Streaming data
Memory-efficient processing
Pipeline-style transformations
```

Instead of loading everything into memory:

```text
Read one
Process one
Discard/continue
```

---

# 90. Looping Over a Generator

```python
for resource in resource_generator():
    process(resource)
```

The loop receives values as they are generated.

This is often preferable for very large collections.

---

# 91. `else` With `for`

Python loops support an `else` block.

```python
for service in services:
    if service == "payment":
        print("Found payment")
        break
else:
    print("Payment service not found")
```

The `else` runs only if the loop finishes without `break`.

---

# 92. Loop `else` Use Case

Searching:

```python
for pod in pods:
    if pod["name"] == target:
        print("Found")
        break
else:
    print("Pod not found")
```

This can be useful, but some teams prefer a direct helper function for readability.

---

# 93. `while` Loop `else`

The same behavior exists:

```python
attempt = 0

while attempt < 3:
    if operation_succeeded():
        break

    attempt += 1
else:
    print("Operation never succeeded")
```

The `else` executes only if no `break` occurred.

---

# 94. Avoid Confusing Loop `else`

Many developers misunderstand it.

Remember:

```text
break
  -> skips loop else

normal loop completion
  -> executes loop else
```

It does not mean:

```text
if loop condition is false
```

in the same way a normal `if/else` works.

---

# 95. Looping and Exceptions

A robust automation loop can handle failures per item:

```python
for resource in resources:
    try:
        process(resource)
    except TemporaryError:
        retry(resource)
    except PermanentError as exc:
        record_failure(
            resource,
            exc
        )
```

This prevents one bad resource from necessarily stopping all processing.

---

# 96. Fail-Fast vs Continue-on-Error

Fail-fast:

```python
for resource in resources:
    process(resource)
```

If `process()` raises an exception, the loop stops.

Continue:

```python
for resource in resources:
    try:
        process(resource)
    except Exception as exc:
        record_failure(resource, exc)
```

Use the strategy required by the operation.

---

# 97. Do Not Catch Everything Blindly

Avoid:

```python
for resource in resources:
    try:
        process(resource)
    except Exception:
        pass
```

This can hide production failures.

Better:

```python
except ExpectedError as exc:
    record_failure(resource, exc)
```

Unexpected errors should normally remain visible.

---

# 98. Loop Logging

Useful:

```python
for service in services:
    logger.info(
        "Checking service=%s",
        service
    )
```

For large loops, avoid excessive logs.

Use:

```text
Start
Progress
Failures
Summary
```

rather than logging every low-value detail.

---

# 99. Progress Reporting

```python
total = len(resources)

for index, resource in enumerate(
    resources,
    start=1
):
    process(resource)

    if index % 100 == 0:
        print(
            f"Processed {index}/{total}"
        )
```

Modulo can be used to report periodic progress.

---

# 100. Progress for Unknown Total

For streaming/generator data:

```python
processed = 0

for resource in resource_generator():
    process(resource)
    processed += 1

    if processed % 100 == 0:
        print(
            f"Processed {processed}"
        )
```

---

# 101. Looping and Idempotency

Suppose:

```python
for resource in resources:
    if resource["state"] != desired_state:
        update_resource(resource)
```

If the desired state is already present, no action occurs.

This is an important automation principle:

```text
Current state == desired state
        |
        v
     No action
```

---

# 102. Reconciliation Loop

A basic controller-style pattern:

```python
while True:
    current = get_current_state()
    desired = get_desired_state()

    if current != desired:
        reconcile(current, desired)

    time.sleep(30)
```

Real controllers need:

```text
Timeout handling
Backoff
Leader election where applicable
Error handling
Rate limiting
Graceful shutdown
```

But the core idea is repeated state reconciliation.

---

# 103. Kubernetes Controller Concept

Kubernetes controllers repeatedly compare:

```text
Desired state
vs
Current state
```

and take corrective action.

Python scripts can implement small reconciliation workflows, although production Kubernetes controllers are often implemented with dedicated controller frameworks/operators.

---

# 104. Looping Over Terraform Plan Changes

Conceptual data:

```python
changes = [
    {"resource": "aws_instance.web", "action": "update"},
    {"resource": "aws_s3_bucket.logs", "action": "no-op"},
    {"resource": "aws_iam_role.app", "action": "delete"}
]

for change in changes:
    if change["action"] == "delete":
        print(
            f"Review deletion: "
            f"{change['resource']}"
        )
```

This can support policy checks around infrastructure changes.

---

# 105. Looping Over CI/CD Jobs

```python
jobs = [
    "build",
    "test",
    "security",
    "deploy"
]

for job in jobs:
    result = run_job(job)

    if not result.success:
        print(
            f"Job failed: {job}"
        )
        break
```

This is a basic pipeline orchestration model.

---

# 106. Pipeline Continue Policy

If jobs are independent:

```python
failed = []

for job in jobs:
    result = run_job(job)

    if not result.success:
        failed.append(job)

if failed:
    raise SystemExit(
        f"Failed jobs: {failed}"
    )
```

This gives a complete failure summary.

---

# 107. Looping Over Test Results

```python
failures = []

for test in tests:
    result = run_test(test)

    if not result.passed:
        failures.append(
            test.name
        )

if failures:
    print(
        f"Failed tests: {failures}"
    )
```

Useful for CI diagnostics.

---

# 108. Looping Over Security Findings

```python
critical = []

for finding in findings:
    if finding["severity"] == "CRITICAL":
        critical.append(finding)

if critical:
    fail_pipeline(
        "Critical findings detected"
    )
```

Security policies should define which severity levels block the pipeline.

---

# 109. Looping Over Docker Images

Conceptual:

```python
images = list_images()

for image in images:
    if image["tag"] == "latest":
        print(
            f"Review mutable tag: "
            f"{image['name']}"
        )
```

This can support container hygiene checks.

---

# 110. Looping Over ECR Images

```python
for image in images:
    if image["pushed_days_ago"] > 90:
        print(
            f"Old image: "
            f"{image['digest']}"
        )
```

Deletion should use explicit retention policy and protection rules.

---

# 111. Looping Over S3 Objects

Conceptually:

```python
for obj in objects:
    if obj["age_days"] > retention_days:
        candidates.append(obj)
```

For large buckets, use pagination rather than assuming all objects are returned in one API response.

---

# 112. S3 Cleanup Safety

```python
for obj in objects:
    if (
        obj["age_days"] > retention_days
        and not obj["protected"]
    ):
        if dry_run:
            print(
                f"Would delete {obj['key']}"
            )
        else:
            delete_object(obj)
```

Always distinguish candidate discovery from deletion.

---

# 113. Looping Over EC2 Instances

```python
for instance in instances:
    if instance["state"] == "running":
        print(
            f"Running: "
            f"{instance['id']}"
        )
```

Add environment/ownership checks before automation that changes or stops instances.

---

# 114. Looping Over RDS Instances

```python
for db in databases:
    if db["storage_percent"] >= 80:
        print(
            f"Storage warning: "
            f"{db['identifier']}"
        )
```

For database operations, monitoring should avoid creating unnecessary load.

---

# 115. Looping Over ALB Targets

Conceptually:

```python
for target in targets:
    if target["health"] != "healthy":
        print(
            f"Unhealthy target: "
            f"{target['id']}"
        )
```

A production diagnostic can then correlate:

```text
Target health
Pod readiness
Application logs
Error rate
Latency
```

---

# 116. Looping Over Prometheus Results

Conceptually:

```python
for sample in metrics:
    if sample["value"] >= threshold:
        print(
            f"Threshold exceeded: "
            f"{sample['name']}"
        )
```

The actual Prometheus query/API format depends on how the Python script integrates with Prometheus.

---

# 117. Looping Over Alert Results

```python
for alert in alerts:
    if alert["status"] == "firing":
        print(
            f"Active alert: "
            f"{alert['name']}"
        )
```

This can support incident summaries.

---

# 118. Looping Over ELK Search Results

Conceptually:

```python
for hit in search_results:
    if hit["severity"] == "ERROR":
        process_error(hit)
```

For large search results, use pagination/scroll/search-after mechanisms supported by the Elasticsearch version and client.

---

# 119. Looping Over Application Logs

```python
for line in application_logs:
    if "ERROR" in line:
        errors += 1
    elif "WARN" in line:
        warnings += 1
```

This creates simple log statistics.

---

# 120. Log Summary

```python
errors = 0
warnings = 0

for line in logs:
    if "ERROR" in line:
        errors += 1
    elif "WARN" in line:
        warnings += 1

print(
    f"errors={errors}, "
    f"warnings={warnings}"
)
```

For structured logs, parse fields rather than relying on substring matching.

---

# 121. Looping Over JSON Lines

For JSONL:

```python
import json

with open("events.jsonl") as file:
    for line in file:
        event = json.loads(line)

        if event["level"] == "ERROR":
            process(event)
```

Handle malformed lines according to the operational requirement.

---

# 122. Handling Bad Records

```python
for line_number, line in enumerate(
    file,
    start=1
):
    try:
        event = json.loads(line)
    except json.JSONDecodeError:
        print(
            f"Invalid JSON at line "
            f"{line_number}"
        )
        continue

    process(event)
```

This allows one malformed record to be skipped while the rest are processed.

---

# 123. Looping Over Database Rows

Conceptually:

```python
for row in cursor:
    if row["status"] == "failed":
        process_failure(row)
```

For very large datasets, stream/batch rows rather than loading everything into memory.

---

# 124. Database Batch Processing

Concept:

```python
while True:
    rows = fetch_batch()

    if not rows:
        break

    process_batch(rows)
```

This is useful for:

```text
Migration
Data cleanup
Reporting
Backfills
Validation
```

---

# 125. Looping Through a Queue

Conceptual worker:

```python
while True:
    message = queue.receive()

    if message is None:
        break

    process_message(message)
```

Production workers need:

```text
Visibility timeout
Acknowledgement
Retry
Dead-letter handling
Graceful shutdown
Poison message handling
```

---

# 126. Poison Message Handling

A message that always fails should not be retried forever.

Concept:

```python
for attempt in range(max_attempts):
    try:
        process_message(message)
        acknowledge(message)
        break
    except TemporaryError:
        retry()
else:
    send_to_dead_letter(message)
```

This is a common production queue pattern.

---

# 127. Graceful Loop Shutdown

Long-running workers should be able to stop safely.

Concept:

```python
running = True

while running:
    message = receive_message()

    if message:
        process_message(message)
```

In real applications, use signal handling or framework-supported shutdown mechanisms.

---

# 128. Looping and Signals

A production process may receive:

```text
SIGTERM
SIGINT
```

The application should stop accepting new work and finish/abort current work according to its shutdown policy.

This is especially important in Kubernetes.

---

# 129. Kubernetes Pod Shutdown

A worker loop should account for:

```text
SIGTERM
terminationGracePeriodSeconds
in-flight work
queue acknowledgement
connection cleanup
```

A simple infinite loop without graceful shutdown can cause duplicate or lost work.

---

# 130. Looping and Rate Limits

If an API allows:

```text
100 requests/second
```

but your loop sends:

```python
for item in items:
    api_call(item)
```

too quickly, you may hit throttling.

Production handling can include:

```text
Client-side rate limiting
Backoff
Batch APIs
Pagination controls
Concurrency limits
```

Do not solve rate limiting blindly with arbitrary `sleep()` values when the SDK/API already provides throttling support.

---

# 131. Looping With Delays

Simple:

```python
for item in items:
    process(item)
    time.sleep(1)
```

This may be acceptable for small scripts, but fixed sleeps are often inefficient.

Prefer:

```text
API-provided retry-after
Adaptive backoff
Rate limiter
Batching
```

where appropriate.

---

# 132. Loop Performance

For a large loop:

```python
for resource in resources:
    expensive_api_call(resource)
```

The main bottleneck may be:

```text
Network I/O
API latency
Database latency
External service throttling
```

Optimize the actual bottleneck rather than micro-optimizing Python syntax.

---

# 133. Avoid Repeated Computation

Bad:

```python
for resource in resources:
    allowed = load_allowed_resources()

    if resource in allowed:
        process(resource)
```

Better:

```python
allowed = load_allowed_resources()

for resource in resources:
    if resource in allowed:
        process(resource)
```

Move invariant work outside the loop.

---

# 134. Avoid Repeated File Reads

Bad:

```python
for service in services:
    with open("config.yaml") as file:
        config = file.read()

    deploy(service, config)
```

Better:

```python
with open("config.yaml") as file:
    config = file.read()

for service in services:
    deploy(service, config)
```

Read shared data once when appropriate.

---

# 135. Avoid Repeated API Client Creation

Potentially inefficient:

```python
for resource in resources:
    client = create_client()
    process(client, resource)
```

Prefer:

```python
client = create_client()

for resource in resources:
    process(client, resource)
```

Reuse clients when the SDK/client is designed for reuse.

---

# 136. Loop Variable Scope

Python does not create a separate block scope for a `for` loop.

Example:

```python
for service in services:
    last_service = service

print(last_service)
```

The variable remains available after the loop.

Be careful when variable names are reused.

---

# 137. Loop Variable Leakage

Avoid confusing code like:

```python
for item in items:
    ...

for item in other_items:
    ...
```

The same variable is reused.

Prefer descriptive names:

```python
for service in services:
    ...

for resource in resources:
    ...
```

This improves readability.

---

# 138. Mutating a List While Iterating

Dangerous pattern:

```python
for item in items:
    if should_remove(item):
        items.remove(item)
```

This can skip elements.

Prefer:

```python
items = [
    item
    for item in items
    if not should_remove(item)
]
```

or iterate over a copy:

```python
for item in items.copy():
    if should_remove(item):
        items.remove(item)
```

Choose based on clarity and data size.

---

# 139. Dictionary Mutation While Iterating

Avoid:

```python
for key in data:
    if should_remove(key):
        del data[key]
```

This can raise:

```text
RuntimeError: dictionary changed size during iteration
```

Use:

```python
for key in list(data):
    if should_remove(key):
        del data[key]
```

---

# 140. Set Mutation While Iterating

The same principle applies to sets.

Avoid modifying the set while iterating over it.

Use a copy or build a new set.

---

# 141. Looping With Comprehensions

For simple transformations:

```python
names = [
    resource["name"]
    for resource in resources
]
```

This is concise.

But do not force complicated operational workflows into comprehensions.

---

# 142. Comprehension vs Loop

Simple:

```python
running = [
    server
    for server in servers
    if server["state"] == "running"
]
```

Complex:

```python
for server in servers:
    if server["state"] != "running":
        continue

    validate_permissions(server)
    collect_metrics(server)
    update_report(server)
```

Use a normal loop for complex side effects.

---

# 143. Nested Comprehension Warning

Avoid overly complex:

```python
result = [
    ...
    for x in ...
    for y in ...
    if ...
]
```

when it becomes difficult to understand.

Production automation values maintainability over clever syntax.

---

# 144. `enumerate()` for Error Reporting

```python
for line_number, line in enumerate(
    lines,
    start=1
):
    if invalid(line):
        print(
            f"Invalid line {line_number}"
        )
```

This is excellent for configuration and log validation.

---

# 145. `enumerate()` for Kubernetes Reports

```python
for index, pod in enumerate(
    pods,
    start=1
):
    print(
        f"{index}. {pod['name']}"
    )
```

Useful for human-readable incident reports.

---

# 146. `zip()` for Environment Mapping

```python
environments = [
    "dev",
    "stage",
    "production"
]

clusters = [
    "dev-eks",
    "stage-eks",
    "prod-eks"
]

for environment, cluster in zip(
    environments,
    clusters,
    strict=True
):
    print(
        environment,
        cluster
    )
```

This can validate one-to-one mappings.

---

# 147. `zip()` for Service Ports

```python
services = [
    "user",
    "cart",
    "payment"
]

ports = [
    8080,
    8081,
    8082
]

for service, port in zip(
    services,
    ports,
    strict=True
):
    check_port(service, port)
```

`strict=True` prevents accidental truncation.

---

# 148. Looping Over Command Output

```python
result = subprocess.run(
    ["kubectl", "get", "pods", "-o", "name"],
    capture_output=True,
    text=True,
    check=True
)

for pod in result.stdout.splitlines():
    print(pod)
```

For production automation, prefer structured output such as JSON over parsing human-formatted CLI output where practical.

---

# 149. Looping Over `kubectl` JSON

Conceptually:

```python
result = subprocess.run(
    [
        "kubectl",
        "get",
        "pods",
        "-o",
        "json"
    ],
    capture_output=True,
    text=True,
    check=True
)

data = json.loads(result.stdout)

for item in data["items"]:
    print(
        item["metadata"]["name"]
    )
```

Structured data is more reliable than text parsing.

---

# 150. Looping Over AWS CLI JSON

Conceptually:

```python
result = subprocess.run(
    [
        "aws",
        "ec2",
        "describe-instances",
        "--output",
        "json"
    ],
    capture_output=True,
    text=True,
    check=True
)

data = json.loads(result.stdout)

for reservation in data["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

Prefer AWS SDKs such as `boto3` when building substantial Python automation instead of shelling out to the CLI unnecessarily.

---

# 151. Looping With `boto3`

Conceptual:

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_instances()

for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(
            instance["InstanceId"]
        )
```

Real production code must account for pagination.

---

# 152. AWS Pagination With Paginator

```python
paginator = ec2.get_paginator(
    "describe_instances"
)

for page in paginator.paginate():
    for reservation in page["Reservations"]:
        for instance in reservation["Instances"]:
            process(instance)
```

Using SDK paginators is preferable to manually implementing pagination when the SDK provides one.

---

# 153. Kubernetes API Pagination

The exact mechanism depends on the client/library and API.

General pattern:

```python
continue_token = None

while True:
    response = list_resources(
        continue_token=continue_token
    )

    process(response["items"])

    continue_token = (
        response.get("continue")
    )

    if not continue_token:
        break
```

Follow the Kubernetes API/client contract rather than inventing pagination behavior.

---

# 154. Looping Over Helm Values

Conceptually:

```python
values = {
    "replicas": 3,
    "image": "app:1.2.0",
    "service_port": 8080
}

for key, value in values.items():
    print(
        f"{key}={value}"
    )
```

This can support configuration validation.

---

# 155. Looping Over Terraform Variables

```python
required = [
    "region",
    "environment",
    "cluster_name"
]

for variable in required:
    if variable not in config:
        raise ValueError(
            f"Missing variable: {variable}"
        )
```

This is useful for preflight checks.

---

# 156. Looping Over Deployment Artifacts

```python
artifacts = [
    "Dockerfile",
    "deployment.yaml",
    "service.yaml",
    "ingress.yaml"
]

for artifact in artifacts:
    validate_artifact(artifact)
```

If every artifact must be valid, fail-fast may be appropriate.

If you want all validation errors, collect failures.

---

# 157. Collecting Multiple Validation Errors

```python
errors = []

for artifact in artifacts:
    try:
        validate_artifact(artifact)
    except ValueError as exc:
        errors.append(
            f"{artifact}: {exc}"
        )

if errors:
    for error in errors:
        print(error)

    raise SystemExit(1)
```

This gives developers a complete report.

---

# 158. Looping Over Configuration Rules

```python
rules = [
    check_environment,
    check_replicas,
    check_image,
    check_security
]

for rule in rules:
    rule(config)
```

This creates a simple validation pipeline.

---

# 159. Ordered Validation

For dependent checks:

```python
if not config_exists():
    raise RuntimeError("Config missing")

if not config_valid():
    raise RuntimeError("Config invalid")

if not credentials_valid():
    raise RuntimeError("Credentials invalid")

deploy()
```

A loop can help when checks are independent:

```python
for check in independent_checks:
    check(config)
```

Choose based on dependency order.

---

# 160. Looping Over Health Endpoints

```python
endpoints = {
    "user": "http://user/health",
    "cart": "http://cart/health",
    "payment": "http://payment/health"
}

for service, url in endpoints.items():
    response = requests.get(
        url,
        timeout=5
    )

    if response.status_code != 200:
        print(
            f"{service}: unhealthy"
        )
```

In production, reuse HTTP sessions and handle network exceptions explicitly.

---

# 161. Health Check Exception Handling

```python
for service, url in endpoints.items():
    try:
        response = requests.get(
            url,
            timeout=5
        )

        if response.status_code != 200:
            print(
                f"{service}: HTTP failure"
            )

    except requests.RequestException as exc:
        print(
            f"{service}: request failed: {exc}"
        )
```

One service failure does not necessarily need to stop checks for every other service.

---

# 162. Looping Over Prometheus Targets

Conceptual:

```python
for target in targets:
    if target["health"] != "up":
        print(
            f"Target down: "
            f"{target['labels']}"
        )
```

Then collect diagnostics.

---

# 163. Looping Over Alertmanager Alerts

Conceptual:

```python
for alert in alerts:
    if alert["status"] == "firing":
        if alert["labels"].get(
            "severity"
        ) == "critical":
            escalate(alert)
```

Keep routing/escalation policy centralized where possible.

---

# 164. Looping Over Incident Data

```python
for incident in incidents:
    if incident["status"] == "open":
        print(
            f"Open incident: "
            f"{incident['id']}"
        )
```

This can support operational reporting.

---

# 165. Looping Over Backup Jobs

```python
for backup in backups:
    if not backup["successful"]:
        print(
            f"Backup failed: "
            f"{backup['name']}"
        )
```

Add age and retention checks for complete validation.

---

# 166. Backup Validation Loop

```python
for backup in backups:
    if not backup["successful"]:
        continue

    if backup["age_hours"] > 24:
        print(
            f"Backup stale: "
            f"{backup['name']}"
        )

    if not backup["checksum_valid"]:
        print(
            f"Checksum failure: "
            f"{backup['name']}"
        )
```

This shows how `continue` can focus on relevant records.

---

# 167. Certificate Monitoring Loop

```python
for certificate in certificates:
    days = certificate["days_remaining"]

    if days <= 7:
        print(
            f"CRITICAL: "
            f"{certificate['name']}"
        )
    elif days <= 30:
        print(
            f"WARNING: "
            f"{certificate['name']}"
        )
```

This is a common automation task.

---

# 168. Disk Cleanup Candidate Loop

```python
candidates = []

for file in files:
    if file["age_days"] < 30:
        continue

    if file["protected"]:
        continue

    candidates.append(file)
```

Then review:

```python
for candidate in candidates:
    print(
        f"Candidate: "
        f"{candidate['path']}"
    )
```

Separate discovery from deletion.

---

# 169. Production Rule — Two-Phase Destructive Automation

Use:

```text
Phase 1
Discover candidates

Phase 2
Validate candidates

Phase 3
Dry-run/report

Phase 4
Approve

Phase 5
Delete/update
```

Do not combine discovery and destructive action unnecessarily.

---

# 170. Loop-Based Resource Inventory

```python
inventory = []

for region in regions:
    resources = list_resources(region)

    for resource in resources:
        inventory.append({
            "region": region,
            "id": resource["id"],
            "state": resource["state"]
        })
```

This creates a normalized inventory.

---

# 171. Inventory Reporting

```python
for item in inventory:
    print(
        f"{item['region']} "
        f"{item['id']} "
        f"{item['state']}"
    )
```

The same inventory can later be exported to:

```text
CSV
JSON
Database
Dashboard
Report
```

---

# 172. Looping Over Tags

```python
required_tags = {
    "Environment",
    "Owner",
    "Application"
}

for resource in resources:
    missing = (
        required_tags
        - set(resource["tags"])
    )

    if missing:
        print(
            f"{resource['id']} "
            f"missing tags: {missing}"
        )
```

This is useful for cloud governance.

---

# 173. Looping Over Kubernetes Labels

```python
required_labels = {
    "app",
    "environment",
    "team"
}

for pod in pods:
    labels = set(
        pod["labels"].keys()
    )

    missing = required_labels - labels

    if missing:
        print(
            f"{pod['name']} "
            f"missing: {missing}"
        )
```

This supports platform governance checks.

---

# 174. Looping Over Security Policies

```python
required_controls = [
    "sast",
    "sca",
    "container_scan",
    "secret_scan"
]

for control in required_controls:
    if not controls.get(control):
        print(
            f"Missing security control: "
            f"{control}"
        )
```

A policy can then decide whether missing controls block the pipeline.

---

# 175. Looping Over Deployment History

```python
for deployment in deployments:
    if deployment["status"] == "failed":
        print(
            deployment["version"]
        )
```

You can collect:

```text
Failure count
Failure versions
Rollback frequency
Mean time between failures
```

---

# 176. Looping for MTTR Analysis

Concept:

```python
total_recovery_time = 0

for incident in incidents:
    total_recovery_time += (
        incident["recovery_minutes"]
    )

if incidents:
    mttr = (
        total_recovery_time
        / len(incidents)
    )
else:
    mttr = 0
```

This combines loops and arithmetic.

---

# 177. Looping for Error Rate

```python
total = 0
errors = 0

for request in requests:
    total += 1

    if request["status"] >= 500:
        errors += 1

error_rate = (
    errors / total * 100
    if total
    else 0
)
```

This is useful for analyzing sampled request data.

---

# 178. Looping for Availability

```python
total = 0
successful = 0

for request in requests:
    total += 1

    if 200 <= request["status"] < 500:
        successful += 1

availability = (
    successful / total * 100
    if total
    else 0
)
```

The definition of "successful" must match the SLI being measured.

---

# 179. Looping Over SLO Windows

Conceptually:

```python
violations = 0

for sample in samples:
    if sample["availability"] < 99.9:
        violations += 1
```

For real SLO calculations, use the precise event/window definition rather than a simplistic sample count.

---

# 180. Looping Over Error Budget Events

```python
budget_used = 0

for event in events:
    if event["type"] == "error":
        budget_used += event["impact"]
```

Then:

```python
if budget_used > budget:
    print("Budget exceeded")
```

---

# 181. Looping and Data Validation

```python
for record in records:
    if "name" not in record:
        print("Missing name")
        continue

    if "environment" not in record:
        print("Missing environment")
        continue

    process(record)
```

For production, consider collecting all validation failures instead of only printing them.

---

# 182. Looping and Type Validation

```python
for record in records:
    replicas = record.get("replicas")

    if not isinstance(replicas, int):
        print(
            f"Invalid replicas: "
            f"{record.get('name')}"
        )
        continue

    process(record)
```

External data should be validated before operational use.

---

# 183. Looping Over Mixed Input

Avoid assuming every record has the same structure.

Use:

```python
for record in records:
    if not isinstance(record, dict):
        record_failure(
            "Record is not a dictionary"
        )
        continue

    process(record)
```

This makes automation more resilient.

---

# 184. Looping and Memory

Bad for huge data:

```python
records = load_everything()

for record in records:
    process(record)
```

Potentially better:

```python
for record in stream_records():
    process(record)
```

Use generators, paginated APIs, or batch processing for large datasets.

---

# 185. Looping and Backpressure

If the producer is faster than the consumer:

```text
Producer
   |
   v
Queue grows
   |
   v
Consumer too slow
```

A production system may need:

```text
Batching
Rate limiting
Queue limits
Concurrency
Backpressure
```

A simple loop alone does not solve throughput imbalance.

---

# 186. Looping and Graceful Failure

For independent resources:

```python
failures = []

for resource in resources:
    try:
        process(resource)
    except KnownError as exc:
        failures.append(
            (resource["id"], str(exc))
        )

if failures:
    report_failures(failures)
    raise SystemExit(1)
```

This is a strong batch-processing pattern.

---

# 187. Looping and Transaction Boundaries

For database or state-changing operations, define whether:

```text
Each item is independent
```

or:

```text
The whole batch must succeed together
```

Do not assume a Python loop automatically provides transactionality.

---

# 188. Looping and Idempotent API Calls

If an API call is safe to repeat:

```python
for resource in resources:
    ensure_desired_state(resource)
```

This is preferable to blindly issuing create/update commands.

---

# 189. Looping and Dry Run

```python
for resource in resources:
    if should_change(resource):
        if dry_run:
            print(
                f"Would update "
                f"{resource['id']}"
            )
        else:
            update(resource)
```

Dry-run mode is especially valuable for cloud and infrastructure automation.

---

# 190. Looping and Audit Logging

For every state-changing action:

```python
for resource in resources:
    if should_update(resource):
        logger.info(
            "Updating resource=%s",
            resource["id"]
        )

        update(resource)
```

Production logs should include enough context to reconstruct what happened.

---

# 191. Looping and Secrets

Never do:

```python
for secret in secrets:
    print(secret)
```

Instead:

```python
for secret in secrets:
    print(
        f"Validated secret={secret['name']}"
    )
```

Log metadata, not secret values.

---

# 192. Looping Over Secrets for Rotation

Conceptually:

```python
for secret in secrets:
    if secret["age_days"] >= 90:
        if secret["rotation_enabled"]:
            rotate(secret)
        else:
            report(secret)
```

Rotation policy must account for application compatibility and rollback.

---

# 193. Looping Over IAM Findings

```python
for finding in findings:
    if finding["severity"] == "HIGH":
        print(
            f"Review: "
            f"{finding['resource']}"
        )
```

Do not automatically modify IAM permissions without an explicit policy and review process.

---

# 194. Looping Over Docker Containers

Conceptually:

```python
for container in containers:
    if container["status"] != "running":
        print(
            f"Container unhealthy: "
            f"{container['name']}"
        )
```

A stopped container may be intentional, so correlate with desired state.

---

# 195. Looping Over Images for Vulnerability Policy

```python
for image in images:
    if image["critical_vulnerabilities"] > 0:
        print(
            f"Block image: "
            f"{image['name']}"
        )
```

Use the organization's defined severity and exception policy.

---

# 196. Looping Over Build Artifacts

```python
for artifact in artifacts:
    if not artifact["checksum_valid"]:
        raise RuntimeError(
            f"Checksum failed: "
            f"{artifact['name']}"
        )
```

This is useful in secure artifact pipelines.

---

# 197. Looping Over Git Commits

Conceptually:

```python
for commit in commits:
    if commit["author"] not in allowed_authors:
        print(
            f"Review commit: "
            f"{commit['sha']}"
        )
```

Actual policy should use repository controls in addition to scripts.

---

# 198. Looping Over Changed Files

```python
for filename in changed_files:
    if filename.endswith(".tf"):
        terraform_changes = True

    if filename.endswith(".yaml"):
        manifest_changes = True
```

This can support selective CI/CD execution.

---

# 199. Conditional Job Selection

```python
for filename in changed_files:
    if filename.startswith("infra/"):
        run_terraform = True

    if filename.startswith("app/"):
        run_application_tests = True
```

This can reduce CI work when implemented carefully.

---

# 200. Looping Over Test Suites

```python
test_suites = [
    "unit",
    "integration",
    "security",
    "smoke"
]

for suite in test_suites:
    result = run_suite(suite)

    if not result.success:
        print(
            f"{suite} failed"
        )
        break
```

Fail-fast may be appropriate when later suites depend on earlier ones.

---

# 201. Looping Over Independent Checks

```python
checks = [
    check_dns,
    check_tls,
    check_http,
    check_database
]

results = {}

for check in checks:
    results[check.__name__] = check()
```

Then summarize:

```python
for name, result in results.items():
    print(name, result)
```

---

# 202. Looping Over Dependencies

```python
dependencies = [
    "postgres",
    "redis",
    "rabbitmq"
]

for dependency in dependencies:
    if not check_dependency(dependency):
        print(
            f"Dependency unavailable: "
            f"{dependency}"
        )
```

This is useful for application preflight checks.

---

# 203. Preflight Validation

Before deployment:

```python
checks = [
    check_cluster_access,
    check_registry_access,
    check_namespace,
    check_secrets,
    check_dependencies
]

for check in checks:
    if not check():
        raise SystemExit(
            f"Preflight failed: "
            f"{check.__name__}"
        )
```

This prevents starting a deployment when prerequisites are missing.

---

# 204. Looping Over Rollout Steps

```python
steps = [
    "validate",
    "deploy",
    "wait",
    "verify",
    "promote"
]

for step in steps:
    result = execute(step)

    if not result.success:
        rollback()
        break
```

The actual rollout controller should manage complex deployment state, but Python can orchestrate supporting tasks.

---

# 205. Polling With Increasing Intervals

Concept:

```python
delay = 2

while not complete():
    time.sleep(delay)
    delay = min(delay * 2, 30)
```

Add a timeout:

```python
deadline = time.monotonic() + 300
```

Without a timeout, a failed external system can cause indefinite polling.

---

# 206. Polling With Maximum Attempts and Timeout

Production-quality concept:

```python
deadline = time.monotonic() + 300
attempt = 0
max_attempts = 60

while (
    attempt < max_attempts
    and time.monotonic() < deadline
):
    status = get_status()

    if status == "complete":
        break

    if status == "failed":
        raise RuntimeError(
            "Operation failed"
        )

    attempt += 1
    time.sleep(5)
else:
    raise TimeoutError(
        "Operation timed out"
    )
```

Use both limits when an external operation is important.

---

# 207. Loop Invariants

A loop should have values that remain true or controlled throughout execution.

Example:

```python
attempt = 0

while attempt < max_attempts:
    attempt += 1
```

Invariant:

```text
attempt is bounded
```

When designing loops, ask:

```text
What changes each iteration?
What eventually makes the loop stop?
Can the state fail to change?
```

---

# 208. Infinite Loop Troubleshooting

If:

```python
while condition:
    ...
```

never exits, check:

```text
Is condition ever changed?
Is the update statement reached?
Is an exception interrupting the update?
Is external state stuck?
Is the clock moving?
Is the API returning stale data?
```

Add bounded attempts or timeout.

---

# 209. Infinite Pagination Troubleshooting

If pagination never ends:

```text
Check next token
Check whether token changes
Check API response
Check page size
Check duplicate tokens
Check termination condition
```

A repeated token is a strong indication of a pagination bug or API issue.

---

# 210. Retry Loop Troubleshooting

If retry count never reaches the maximum:

```python
attempt += 1
```

may be missing.

If the operation always succeeds/fails unexpectedly, inspect:

```text
Exception type
Retryable classification
Attempt counter
Break condition
Timeout
```

---

# 211. Polling Troubleshooting

If deployment polling never completes:

```text
1. Verify rollout actually started
2. Check current status directly
3. Verify expected success state
4. Verify failure states
5. Check timeout
6. Check polling interval
7. Check API permissions
8. Check stale response/cache
```

---

# 212. Loop Performance Troubleshooting

If a loop is slow:

```text
Measure total runtime
Measure per-item time
Count API calls
Check network latency
Check database queries
Check repeated computation
Check serialization
Check rate limiting
```

Do not assume the Python loop itself is the bottleneck.

---

# 213. API Throttling Troubleshooting

Symptoms:

```text
429 responses
ThrottlingException
Slow responses
Retries
```

Investigate:

```text
Request rate
Concurrency
Batching
SDK retry configuration
Retry-After headers
Backoff
```

A loop may be producing too many API requests.

---

# 214. Memory Troubleshooting

If memory increases during a loop:

Check for:

```text
Growing result lists
Unbounded caches
Loading entire files
Accumulating API responses
Storing every log line
Large nested objects
```

Prefer:

```text
Streaming
Generators
Pagination
Batch processing
Bounded buffers
```

---

# 215. File Processing Troubleshooting

If processing a large log causes high memory:

Bad:

```python
lines = file.readlines()

for line in lines:
    process(line)
```

Better:

```python
for line in file:
    process(line)
```

---

# 216. Batch Processing Troubleshooting

If a batch fails:

```text
Identify failed item
Separate transient vs permanent errors
Retry only retryable items
Record failed IDs
Continue when safe
Report final summary
```

Do not blindly rerun an entire batch if operations are not idempotent.

---

# 217. Production Loop Checklist

Before deploying a loop:

```text
[ ] Is the termination condition clear?
[ ] Can the loop become infinite?
[ ] Is there a timeout?
[ ] Is there a maximum attempt count?
[ ] Are transient errors retried?
[ ] Are permanent errors not retried?
[ ] Are API rate limits respected?
[ ] Is memory bounded?
[ ] Is pagination handled?
[ ] Are repeated tokens detected?
[ ] Is logging useful but not excessive?
[ ] Are secrets protected?
[ ] Is the operation idempotent?
[ ] Is dry-run available for destructive actions?
[ ] Are failures summarized?
[ ] Are boundary conditions tested?
```

---

# 218. Interview — What Is a `for` Loop?

Strong answer:

> "`for` iterates over an iterable such as a list, dictionary, generator, API result, or file. In DevOps I use it to process cloud resources, Kubernetes objects, configuration records, logs, and deployment checks."

---

# 219. Interview — What Is a `while` Loop?

Strong answer:

> "`while` repeatedly executes code while a condition is true. It is useful for polling, retries, workers, and operations where the number of iterations is not known in advance. In production I make sure such loops have a clear termination strategy."

---

# 220. Interview — `for` vs `while`

Strong answer:

> "I use `for` when I am processing a known iterable or collection. I use `while` when I am waiting for a condition or retrying/polling until a state changes. For external operations, I normally combine `while` with a timeout or maximum attempts."

---

# 221. Interview — What Does `break` Do?

Strong answer:

> "`break` immediately exits the nearest loop. For example, I can stop a service-health scan when a first critical failure is found, if the operational requirement is fail-fast."

---

# 222. Interview — What Does `continue` Do?

Strong answer:

> "`continue` skips the current iteration and proceeds with the next item. I use it to filter irrelevant resources while keeping the main processing path readable."

---

# 223. Interview — What Does `pass` Do?

Strong answer:

> "`pass` is a no-op. It satisfies Python's requirement for a statement where a block is syntactically required. It does not skip an iteration or exit the loop."

---

# 224. Interview — How Do You Prevent Infinite Loops?

Strong answer:

```text
Define termination condition
Track progress
Set maximum attempts
Set timeout
Handle terminal failures
Validate external state changes
Log progress
```

Example:

```python
deadline = time.monotonic() + 300

while time.monotonic() < deadline:
    if operation_complete():
        break

    time.sleep(5)
else:
    raise TimeoutError("Timed out")
```

---

# 225. Interview — How Do You Implement Retries?

Strong answer:

> "I classify errors into retryable and non-retryable categories, define a maximum number of attempts and total timeout, and use exponential backoff with jitter where appropriate. I avoid retrying permanent failures such as invalid credentials or malformed requests."

---

# 226. Interview — How Do You Handle API Pagination?

Strong answer:

> "I use the API or SDK's pagination mechanism. If the SDK provides a paginator, I prefer that. Otherwise I continue using the returned page token until it is absent, while handling API errors, repeated tokens, rate limits, and timeouts."

---

# 227. Interview — How Do You Process Large Logs?

Strong answer:

> "I avoid loading the entire file into memory. I iterate line by line or use a generator, process records incrementally, and maintain only the counters or state required for the result."

---

# 228. Interview — How Do You Process Thousands of AWS Resources?

Strong answer:

```text
Use SDK pagination
Reuse clients
Process incrementally
Avoid unnecessary API calls
Respect throttling
Use batching where supported
Add retry/backoff
Log progress
Keep memory bounded
```

If appropriate, introduce concurrency carefully after measuring the bottleneck.

---

# 229. Interview — How Do You Handle One Failed Resource?

Strong answer:

> "It depends on whether resources are independent. For independent inventory/reporting tasks, I normally catch expected errors, record the resource failure, and continue. For transactional or dependent operations, I may fail fast. I make that policy explicit rather than accidentally choosing one behavior."

---

# 230. Interview — How Do You Avoid Duplicate Operations?

Strong answer:

> "I compare current state with desired state before making changes and design actions to be idempotent where possible. For destructive or non-idempotent operations, I use explicit state tracking and safety checks."

---

# 231. Interview — Why Use `enumerate()`?

Strong answer:

> "`enumerate()` gives both the index and value while iterating. It is useful for numbered reports, line numbers, progress tracking, and diagnostics without manually maintaining a counter."

---

# 232. Interview — Why Use `zip()`?

Strong answer:

> "`zip()` lets me iterate over related values from multiple iterables at the same time. For configuration that must have equal lengths, I can use `strict=True` so mismatched data fails instead of being silently truncated."

---

# 233. Interview — How Do You Handle Rate Limits in a Loop?

Strong answer:

> "I first use the SDK's built-in retry/throttling behavior when available. Otherwise I handle rate-limit responses using backoff, respect Retry-After information when provided, reduce concurrency, batch requests where supported, and avoid unnecessary API calls."

---

# 234. Interview — How Do You Make a Destructive Loop Safe?

Strong answer:

```text
Validate environment
Validate ownership
Check protection
Use allowlists
Separate discovery from action
Support dry-run
Require approval where appropriate
Log every action
Make deletion idempotent where possible
Fail closed when required metadata is missing
```

---

# 235. Interview — How Would You Monitor All EKS Pods?

Strong answer:

```text
List namespaces
  |
  v
List pods
  |
  v
For each pod:
  - phase
  - readiness
  - restart count
  - container state
  - resource usage if available
  - events/logs when unhealthy
  |
  v
Aggregate failures
  |
  v
Generate report/alert
```

Use pagination and avoid excessive API calls.

---

# 236. Interview — How Would You Poll a Deployment?

Strong answer:

```text
Start rollout
   |
   v
Set deadline
   |
   v
Get rollout state
   |
   +--> complete -> success
   |
   +--> failed -> fail
   |
   +--> progressing -> sleep and poll
   |
   v
Deadline reached -> timeout
```

This is safer than an unbounded `while True`.

---

# 237. Scenario — EKS Deployment Hangs

Approach:

```text
1. Confirm rollout started
2. Poll current rollout state
3. Set/verify timeout
4. Check desired vs ready replicas
5. Check pending pods
6. Check image pulls
7. Check readiness probes
8. Check events
9. Check resource pressure
10. Stop polling on terminal failure/timeout
```

The Python loop should report the real failure rather than waiting indefinitely.

---

# 238. Scenario — AWS Inventory Script Gets Throttled

Approach:

```text
1. Measure request volume
2. Confirm API throttling
3. Use SDK paginator
4. Reuse client
5. Reduce unnecessary calls
6. Batch operations where supported
7. Apply exponential backoff
8. Respect Retry-After if provided
9. Limit concurrency
10. Log retry counts
```

---

# 239. Scenario — Log Processing Uses Too Much Memory

Likely issue:

```python
lines = file.readlines()
```

Fix:

```python
with open(path) as file:
    for line in file:
        process(line)
```

For more complex processing:

```python
def records(path):
    with open(path) as file:
        for line in file:
            yield parse(line)
```

---

# 240. Scenario — Cleanup Script Skips Files

Potential cause:

```python
for file in files:
    if should_remove(file):
        files.remove(file)
```

Fix by iterating over a copy or building a filtered collection.

Best approach often:

```python
remaining = [
    file
    for file in files
    if not should_remove(file)
]
```

---

# 241. Scenario — Retry Script Retries Authentication Failures

Problem:

```python
except Exception:
    retry()
```

Fix:

```python
except RateLimitError:
    retry()

except TimeoutError:
    retry()

except AuthenticationError:
    raise
```

Classify errors explicitly.

---

# 242. Scenario — Pipeline Stops at First Failed Test

Determine requirement.

If fail-fast is desired:

```python
for test in tests:
    if not run_test(test):
        raise SystemExit(1)
```

If complete reporting is desired:

```python
failures = []

for test in tests:
    if not run_test(test):
        failures.append(test)

if failures:
    report(failures)
    raise SystemExit(1)
```

---

# 243. Scenario — API Pagination Never Ends

Check:

```text
next_token value
token changes between requests
termination condition
API response
client library behavior
maximum page count
```

Use:

```python
seen_tokens = set()

while token:
    if token in seen_tokens:
        raise RuntimeError(
            "Repeated pagination token"
        )

    seen_tokens.add(token)

    response = fetch(token)
    token = response.get("next_token")
```

---

# 244. Scenario — Worker Loop Does Not Stop During Kubernetes Termination

Investigate:

```text
SIGTERM handling
graceful shutdown flag
current message processing
queue acknowledgement
termination grace period
blocking operations
```

A production worker should stop accepting new work and exit cleanly according to its shutdown contract.

---

# 245. Practical Exercise 1 — Service Health

Given:

```python
services = [
    {"name": "user", "healthy": True},
    {"name": "cart", "healthy": False},
    {"name": "payment", "healthy": True}
]
```

Write a loop that:

```text
Prints every service
Identifies unhealthy services
Counts unhealthy services
```

---

# 246. Practical Exercise 2 — Fail-Fast Health Check

Given:

```python
services = [
    "user",
    "cart",
    "payment"
]
```

Stop processing at the first unhealthy service.

Expected concept:

```python
for service in services:
    if not check_health(service):
        print(service)
        break
```

---

# 247. Practical Exercise 3 — Complete Health Report

Process every service and produce:

```text
Healthy:
user
payment

Unhealthy:
cart
order
```

Do not stop after the first failure.

---

# 248. Practical Exercise 4 — Retry

Implement:

```text
Maximum 3 attempts
Retry only temporary failures
Raise permanent failures
Print attempt number
```

Test:

```text
Success on first attempt
Success on second attempt
Failure on all attempts
Permanent failure
```

---

# 249. Practical Exercise 5 — Polling

Simulate:

```text
PENDING
PENDING
PROGRESSING
PROGRESSING
COMPLETE
```

Use a loop that exits when:

```text
COMPLETE
```

and fails when:

```text
FAILED
```

Also add a timeout.

---

# 250. Practical Exercise 6 — Pagination

Simulate pages:

```python
pages = [
    ["a", "b"],
    ["c", "d"],
    ["e"]
]
```

Process every item.

Then modify the exercise to use a token:

```text
token1 -> token2 -> None
```

---

# 251. Practical Exercise 7 — Kubernetes Pod Report

Given:

```python
pods = [
    {
        "name": "user-1",
        "phase": "Running",
        "ready": True
    },
    {
        "name": "payment-1",
        "phase": "Running",
        "ready": False
    },
    {
        "name": "cart-1",
        "phase": "Pending",
        "ready": False
    }
]
```

Classify each as:

```text
HEALTHY
NOT_READY
NOT_RUNNING
```

---

# 252. Practical Exercise 8 — AWS Cleanup Candidates

Given resource records, identify candidates where:

```text
environment == dev
state == stopped
age >= 30
protected == False
```

Then implement:

```text
dry_run=True
```

so the script reports candidates without deleting anything.

---

# 253. Practical Exercise 9 — Log Error Counter

Read a log file line by line and calculate:

```text
ERROR count
WARN count
INFO count
```

Do not use `readlines()` for a large file.

---

# 254. Practical Exercise 10 — Missing Environment Variables

Given:

```python
required = [
    "AWS_REGION",
    "CLUSTER_NAME",
    "IMAGE_TAG"
]
```

Loop through them and report all missing variables instead of stopping at the first missing one.

---

# 255. Practical Exercise 11 — Security Findings

Given findings with severities:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

Loop through them and:

```text
Count each severity
List critical findings
Fail if critical count > 0
```

---

# 256. Practical Exercise 12 — Resource Inventory

Simulate:

```text
3 regions
10 resources per region
```

Build an inventory containing:

```text
Region
Resource ID
State
Environment
```

Then print resources that are:

```text
running
and
development
```

---

# 257. Practical Exercise 13 — Batch Processing

Given:

```python
resources = range(125)
batch_size = 25
```

Process resources in batches of 25.

Expected:

```text
5 batches
```

Make sure the final batch is handled correctly.

---

# 258. Practical Exercise 14 — Certificate Monitoring

Given certificates with:

```text
days_remaining
```

Classify:

```text
> 30     NORMAL
8-30     WARNING
<= 7     CRITICAL
```

Test boundary values:

```text
30
31
7
8
```

---

# 259. Practical Exercise 15 — SLO Analysis

Given request records, calculate:

```text
Total requests
Successful requests
Failed requests
Error rate
```

Then compare the result with:

```python
slo = 99.9
```

Return:

```text
SLO MET
SLO VIOLATED
```

---

# 260. Practical Exercise 16 — MTTR

Given incidents:

```python
incidents = [
    {"recovery_minutes": 20},
    {"recovery_minutes": 40},
    {"recovery_minutes": 60}
]
```

Use a loop to calculate:

```text
Total recovery time
Average recovery time
```

---

# 261. Practical Exercise 17 — Retryable Errors

Given:

```python
retryable = {
    "timeout",
    "rate_limit",
    "temporary_unavailable"
}
```

Loop through failures and:

```text
Retry retryable errors
Fail permanent errors
Count each category
```

---

# 262. Practical Exercise 18 — Loop Safety

Review this code:

```python
while True:
    status = get_status()

    if status == "complete":
        break

    time.sleep(5)
```

Identify the production risks and improve it with:

```text
Timeout
Failure state
Maximum attempts
Logging
```

---

# 263. Practical Exercise 19 — Deployment Verification

Write a function that returns:

```text
NOT_STARTED
IN_PROGRESS
FAILED
NOT_READY
DEGRADED
HEALTHY
```

based on:

```text
started
rollout_status
ready_replicas
desired_replicas
error_rate
```

Test every state.

---

# 264. Practical Exercise 20 — End-to-End DevOps Loop

Build a small script that:

```text
1. Reads services
2. Checks configuration
3. Checks health
4. Collects failures
5. Retries temporary failures
6. Produces a summary
7. Returns exit code 0 if healthy
8. Returns non-zero if failures remain
```

This combines the major loop patterns from this file.

---

# 265. Production Example — EKS Service Validation

Conceptual workflow:

```text
Get namespaces
      |
      v
Get pods
      |
      v
Loop through pods
      |
      +--> Running + Ready -> healthy
      |
      +--> Running + not Ready -> investigate
      |
      +--> Pending -> investigate scheduling
      |
      +--> CrashLoopBackOff -> collect logs/events
      |
      v
Aggregate results
      |
      v
Return deployment health
```

The script should use Kubernetes APIs/client libraries rather than fragile text parsing where possible.

---

# 266. Production Example — AWS Inventory

```text
Regions
   |
   v
SDK paginator
   |
   v
Pages
   |
   v
Resources
   |
   v
Validate tags/state/environment
   |
   +--> compliant
   |
   +--> non-compliant
   |
   v
Report
```

This is a common enterprise automation architecture.

---

# 267. Production Example — CI/CD Validation

```text
Changed files
     |
     v
Loop through changes
     |
     +--> Terraform -> IaC validation
     |
     +--> Python -> unit tests
     |
     +--> Docker -> image checks
     |
     +--> Kubernetes -> manifest validation
     |
     v
Collect results
     |
     v
Security checks
     |
     v
Pass / Fail pipeline
```

---

# 268. Production Example — Centralized Log Analysis

```text
Log source
   |
   v
Stream/read records
   |
   v
Parse
   |
   v
Loop over records
   |
   +--> ERROR -> count/collect
   +--> WARN  -> count
   +--> INFO  -> ignore/aggregate
   |
   v
Summary
```

For enterprise workloads, use centralized logging systems such as ELK rather than relying on local Python log processing alone.

---

# 269. Production Example — Automated Remediation

```text
Detect condition
      |
      v
Loop through affected resources
      |
      v
Validate safety
      |
      +--> unsafe -> escalate
      |
      +--> safe -> remediate
      |
      v
Verify result
      |
      v
Record outcome
```

Always verify that remediation actually restored the desired state.

---

# 270. Production Example — Desired-State Reconciliation

```python
for resource in resources:
    current = get_current_state(resource)
    desired = get_desired_state(resource)

    if current == desired:
        continue

    reconcile(
        resource,
        current,
        desired
    )
```

This pattern is central to infrastructure automation.

---

# 271. Final Mental Model

For DevOps loops, remember:

```text
FOR
  -> process known collections

WHILE
  -> wait/retry/poll until a condition changes

BREAK
  -> stop immediately

CONTINUE
  -> skip current item

PASS
  -> do nothing

ENUMERATE
  -> index + value

ZIP
  -> related values together

RANGE
  -> controlled numeric sequence
```

For production:

```text
Loop
  |
  v
Bound it
  |
  v
Handle failures
  |
  v
Respect rate limits
  |
  v
Control memory
  |
  v
Log useful progress
  |
  v
Make actions idempotent
  |
  v
Test termination and boundaries
```

---

# 272. Next File

```text
22-Python-for-DevOps/
└── 01-Python-Fundamentals/
    ├── 01-Python-Introduction.md
    ├── 02-Variables-and-Data-Types.md
    ├── 03-Operators.md
    ├── 04-Conditional-Statements.md
    └── 05-Loops.md
```

Next topic:

```text
06-Functions.md
```

Planned depth:

```text
Function fundamentals
Parameters and arguments
Return values
Default arguments
Keyword arguments
*args and **kwargs
Scope
Local/global/nonlocal
Lambda functions
Higher-order functions
Decorators
Type hints
Docstrings
Reusable DevOps utilities
AWS automation functions
Kubernetes functions
Error handling
Retry wrappers
Logging decorators
Testing functions
Production design
Common mistakes
Troubleshooting
Interview questions
Scenario-based questions
Practical exercises
