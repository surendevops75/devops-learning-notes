# Functions

> Functions are one of the most important Python concepts for DevOps automation. Good functions turn repeated operational logic into reusable, testable, observable, and safe building blocks.

---

# 1. What Is a Function?

A function is a reusable block of code that performs a specific task.

Basic example:

```python
def greet():
    print("Hello DevOps")

greet()
```

Instead of repeating the same logic:

```text
Define once
    |
    v
Call whenever required
```

Functions are essential for:

```text
AWS automation
Kubernetes automation
CI/CD scripts
Infrastructure validation
Monitoring checks
Log processing
API integrations
Health checks
Deployment automation
Incident remediation
Testing
```

---

# 2. Why Functions Matter in DevOps

Without functions:

```python
# check dev
# check stage
# check production
# repeated code
```

With functions:

```python
def check_environment(environment):
    ...
```

Then:

```python
check_environment("dev")
check_environment("stage")
check_environment("production")
```

Benefits:

```text
Reusability
Maintainability
Testability
Readability
Consistency
Smaller scripts
Easier troubleshooting
Easier code review
```

---

# 3. Basic Function Syntax

```python
def function_name():
    # function body
    pass
```

Example:

```python
def check_health():
    print("Checking health")
```

Call:

```python
check_health()
```

---

# 4. Function Execution

Defining a function does not execute it.

```python
def deploy():
    print("Deploying")
```

Nothing happens until:

```python
deploy()
```

Think:

```text
def
 |
 v
Create reusable behavior

()
 |
 v
Execute behavior
```

---

# 5. Function With a Parameter

```python
def greet(name):
    print(f"Hello {name}")

greet("Surendra")
```

Output:

```text
Hello Surendra
```

The parameter allows the same function to work with different values.

---

# 6. Multiple Parameters

```python
def check_service(service, port):
    print(
        f"Checking {service}:{port}"
    )

check_service("payment", 8080)
```

---

# 7. Parameters vs Arguments

Parameter:

```python
def deploy(environment):
    ...
```

`environment` is a parameter.

Argument:

```python
deploy("production")
```

`"production"` is the argument.

Interview answer:

> A parameter is the variable defined by the function; an argument is the actual value passed when calling the function.

---

# 8. Return Values

A function can return a result.

```python
def add(a, b):
    return a + b

result = add(10, 20)

print(result)
```

Output:

```text
30
```

---

# 9. `return` vs `print`

Avoid using `print()` when the caller needs the result.

Less reusable:

```python
def check_health():
    print(True)
```

Better:

```python
def check_health():
    return True
```

Then:

```python
if check_health():
    print("Healthy")
```

The second design is easier to test and reuse.

---

# 10. Function Without Explicit `return`

```python
def hello():
    print("Hello")
```

Python returns:

```python
None
```

Example:

```python
result = hello()

print(result)
```

Output:

```text
Hello
None
```

---

# 11. Returning Multiple Values

Python can return multiple values:

```python
def get_health():
    return True, 200

healthy, status_code = get_health()
```

This is actually tuple unpacking.

---

# 12. Returning a Dictionary

For more descriptive results:

```python
def check_service():
    return {
        "healthy": True,
        "status_code": 200,
        "latency_ms": 35
    }
```

This can be useful for DevOps automation reports.

---

# 13. Returning Lists

```python
def get_failed_services(services):
    return [
        service
        for service in services
        if not service["healthy"]
    ]
```

The caller can process the returned collection.

---

# 14. Default Arguments

```python
def connect(host, port=22):
    print(
        f"Connecting to {host}:{port}"
    )
```

Then:

```python
connect("server-01")
```

uses:

```text
port = 22
```

Or:

```python
connect("server-01", 2222)
```

overrides the default.

---

# 15. Default Arguments in DevOps

Example:

```python
def health_check(
    url,
    timeout=5
):
    ...
```

Most callers can use:

```python
health_check(url)
```

while special cases can use:

```python
health_check(url, timeout=10)
```

---

# 16. Keyword Arguments

```python
def deploy(
    environment,
    replicas,
    image
):
    ...
```

Call:

```python
deploy(
    environment="production",
    replicas=3,
    image="app:1.2.0"
)
```

Keyword arguments improve readability when several parameters have similar types.

---

# 17. Positional vs Keyword Arguments

Positional:

```python
deploy(
    "production",
    3,
    "app:1.2.0"
)
```

Keyword:

```python
deploy(
    environment="production",
    replicas=3,
    image="app:1.2.0"
)
```

For operational code, keyword arguments often make intent clearer.

---

# 18. Mixing Positional and Keyword Arguments

Valid:

```python
deploy(
    "production",
    replicas=3,
    image="app:1.2.0"
)
```

A positional argument must come before keyword arguments.

---

# 19. Keyword-Only Arguments

Use `*`:

```python
def deploy(
    environment,
    *,
    replicas,
    timeout
):
    ...
```

Then:

```python
deploy(
    "production",
    replicas=3,
    timeout=300
)
```

This prevents accidental positional use.

---

# 20. Why Keyword-Only Arguments Help DevOps

Consider:

```python
deploy(
    "production",
    3,
    300,
    True
)
```

It is difficult to understand.

Better:

```python
deploy(
    "production",
    replicas=3,
    timeout=300,
    dry_run=True
)
```

Operational safety improves when important flags are explicit.

---

# 21. Variable-Length Arguments: `*args`

```python
def notify(*recipients):
    for recipient in recipients:
        print(recipient)
```

Call:

```python
notify(
    "team-a",
    "team-b",
    "team-c"
)
```

Inside the function:

```text
recipients -> tuple
```

---

# 22. `*args` DevOps Example

```python
def run_checks(*checks):
    for check in checks:
        check()
```

Call:

```python
run_checks(
    check_dns,
    check_tls,
    check_http
)
```

Useful when the number of checks is dynamic.

---

# 23. `**kwargs`

`**kwargs` accepts variable keyword arguments.

```python
def deploy(**options):
    print(options)
```

Call:

```python
deploy(
    environment="production",
    replicas=3,
    dry_run=True
)
```

Inside:

```text
options -> dictionary
```

---

# 24. `**kwargs` DevOps Example

```python
def create_resource(**tags):
    for key, value in tags.items():
        print(
            f"{key}={value}"
        )
```

Call:

```python
create_resource(
    Environment="production",
    Owner="platform",
    Application="payments"
)
```

---

# 25. Combining Parameters, `*args`, and `**kwargs`

```python
def execute(
    command,
    *args,
    timeout=30,
    **kwargs
):
    ...
```

This is powerful but can become difficult to understand.

Use it when it genuinely improves API flexibility.

---

# 26. Function Parameter Ordering

A common order is:

```text
positional parameters
*
keyword-only parameters
**kwargs
```

Example:

```python
def operation(
    resource,
    region,
    *args,
    timeout=30,
    dry_run=False,
    **kwargs
):
    ...
```

Keep function signatures simple whenever possible.

---

# 27. Mutable Default Argument Trap

Avoid:

```python
def add_item(
    item,
    items=[]
):
    items.append(item)
    return items
```

The same list can be reused across calls.

---

# 28. Correct Mutable Default Pattern

Use:

```python
def add_item(
    item,
    items=None
):
    if items is None:
        items = []

    items.append(item)

    return items
```

This creates a new list when no list is supplied.

---

# 29. Function Scope

Variables created inside a function are normally local.

```python
def deploy():
    environment = "production"
    print(environment)
```

Outside:

```python
print(environment)
```

will fail because `environment` is local to the function.

---

# 30. Local Variables

```python
def calculate():
    total = 10 + 20
    return total
```

`total` exists only inside the function.

Prefer local variables because they reduce unintended side effects.

---

# 31. Global Variables

Example:

```python
ENVIRONMENT = "production"

def show_environment():
    print(ENVIRONMENT)
```

The function can read the global variable.

However, excessive global state makes automation harder to test and reason about.

---

# 32. Avoid Global Mutable State

Avoid:

```python
results = []

def process(item):
    results.append(item)
```

Better:

```python
def process(item, results):
    results.append(item)
```

Or preferably return the result.

---

# 33. `global`

Python supports:

```python
counter = 0

def increment():
    global counter
    counter += 1
```

Avoid this pattern unless there is a strong reason.

Passing state explicitly is usually easier to test.

---

# 34. `nonlocal`

`nonlocal` applies to nested functions.

```python
def counter():
    value = 0

    def increment():
        nonlocal value
        value += 1
        return value

    return increment
```

This is more advanced and commonly appears with closures.

---

# 35. Function Naming

Use descriptive names:

```python
check_health()
get_pods()
validate_config()
deploy_application()
rotate_certificate()
collect_metrics()
```

Avoid:

```python
do_it()
run()
thing()
test()
```

unless the context makes the meaning obvious.

---

# 36. Function Naming Convention

Python convention:

```text
snake_case
```

Examples:

```python
check_health()
get_instance_status()
validate_environment()
collect_pod_metrics()
```

---

# 37. Single Responsibility

A function should ideally have one clear responsibility.

Good:

```python
def get_pods():
    ...

def check_pod_health():
    ...

def generate_report():
    ...
```

Harder to maintain:

```python
def monitor_everything():
    # AWS
    # Kubernetes
    # logs
    # deployment
    # email
    # cleanup
```

Large functions become difficult to test and troubleshoot.

---

# 38. Function Size

There is no universal line-count rule.

Instead ask:

```text
Can I explain what this function does in one sentence?
Does it have one main responsibility?
Can I test it independently?
Does the name describe its behavior?
```

---

# 39. Pure Functions

A pure function:

```text
Same input
   |
   v
Same output
```

and does not modify external state.

Example:

```python
def calculate_error_rate(
    errors,
    total
):
    if total == 0:
        return 0

    return errors / total
```

Pure functions are easy to test.

---

# 40. Side-Effect Functions

Example:

```python
def delete_resource(resource_id):
    client.delete(resource_id)
```

This has an external side effect.

Side effects are normal in DevOps, but isolate them where possible.

---

# 41. Pure Logic + Side Effects

Better architecture:

```text
Collect data
   |
   v
Pure validation/calculation
   |
   v
Decision
   |
   v
Side effect
```

Example:

```python
if should_delete(resource):
    delete_resource(resource["id"])
```

The decision can be tested separately from deletion.

---

# 42. Function for Health Check

```python
def check_health(url, timeout=5):
    response = requests.get(
        url,
        timeout=timeout
    )

    return response.status_code == 200
```

Caller:

```python
if check_health(url):
    print("Healthy")
else:
    print("Unhealthy")
```

---

# 43. Better Health Check Result

Instead of returning only `True`:

```python
def check_health(url, timeout=5):
    response = requests.get(
        url,
        timeout=timeout
    )

    return {
        "healthy": response.status_code == 200,
        "status_code": response.status_code,
        "latency_ms": response.elapsed.total_seconds() * 1000
    }
```

This gives the caller more information.

---

# 44. Handling Exceptions in Functions

```python
def check_health(url):
    try:
        response = requests.get(
            url,
            timeout=5
        )
        return response.status_code == 200

    except requests.RequestException:
        return False
```

Whether returning `False` is appropriate depends on the caller's need to distinguish:

```text
HTTP failure
Network failure
DNS failure
Timeout
Application failure
```

Often a structured result or a specific exception is better.

---

# 45. Do Not Hide Exceptions

Avoid:

```python
def deploy():
    try:
        run_deployment()
    except Exception:
        return False
```

This can hide serious programming bugs.

Prefer catching expected operational exceptions.

---

# 46. Custom Exceptions

Create domain-specific exceptions:

```python
class DeploymentFailedError(Exception):
    pass
```

Then:

```python
def deploy():
    if failed:
        raise DeploymentFailedError(
            "Deployment failed"
        )
```

Caller:

```python
try:
    deploy()
except DeploymentFailedError as exc:
    rollback(exc)
```

---

# 47. Exception Boundaries

A good pattern:

```text
Low-level function
    |
    v
Raise meaningful error
    |
    v
Higher-level orchestration
    |
    v
Decide retry / rollback / exit
```

Do not make every low-level function independently decide the entire incident strategy.

---

# 48. Function Documentation

Use docstrings:

```python
def calculate_error_rate(errors, total):
    """Return the error rate as a decimal."""
    if total == 0:
        return 0

    return errors / total
```

Docstrings explain purpose and expected behavior.

---

# 49. Detailed Docstring

```python
def check_service(
    service,
    timeout=5
):
    """
    Check whether a service health endpoint responds successfully.

    Args:
        service: Service configuration.
        timeout: Request timeout in seconds.

    Returns:
        Dictionary containing health status and metadata.

    Raises:
        requests.RequestException: On network failure.
    """
    ...
```

For production libraries, document important failure behavior.

---

# 50. Type Hints

Example:

```python
def calculate_error_rate(
    errors: int,
    total: int
) -> float:
    if total == 0:
        return 0.0

    return errors / total
```

Type hints improve:

```text
Readability
IDE support
Static analysis
Maintenance
Code review
```

---

# 51. Type Hints for DevOps Functions

```python
def check_node(
    node_name: str,
    timeout: int = 10
) -> bool:
    ...
```

The type hints communicate the intended interface.

They are not, by themselves, runtime validation.

---

# 52. Optional Values

Modern Python can express optional values:

```python
def get_region(
    config: dict
) -> str | None:
    return config.get("region")
```

The caller knows the function may return `None`.

---

# 53. Lists in Type Hints

```python
def get_failed_services(
    services: list[dict]
) -> list[dict]:
    ...
```

For richer codebases, define more precise types with dataclasses or typed dictionaries.

---

# 54. Dictionary Type Hints

Simple:

```python
def process_resource(
    resource: dict
) -> None:
    ...
```

More precise:

```python
from typing import TypedDict

class Resource(TypedDict):
    id: str
    state: str
    environment: str
```

Then:

```python
def process_resource(
    resource: Resource
) -> None:
    ...
```

---

# 55. Dataclasses for Structured DevOps Data

```python
from dataclasses import dataclass

@dataclass
class Service:
    name: str
    port: int
    environment: str
```

Then:

```python
service = Service(
    name="payment",
    port=8080,
    environment="production"
)
```

Functions can accept structured objects instead of loosely defined dictionaries.

---

# 56. Function Composition

One function can call another:

```python
def validate():
    ...

def deploy():
    ...

def verify():
    ...

def release():
    validate()
    deploy()
    verify()
```

This creates a workflow.

---

# 57. Deployment Function Architecture

```text
release()
   |
   +--> validate()
   |
   +--> deploy()
   |
   +--> wait_for_rollout()
   |
   +--> verify()
   |
   +--> report()
```

Each function has a focused responsibility.

---

# 58. Function Return Contracts

Define what a function returns.

Bad:

```python
def deploy():
    # sometimes returns True
    # sometimes returns None
    # sometimes prints
```

Better:

```python
def deploy() -> DeploymentResult:
    ...
```

or consistently:

```python
def deploy() -> bool:
    ...
```

Consistency makes automation easier to compose.

---

# 59. Dataclass Result Pattern

```python
from dataclasses import dataclass

@dataclass
class CheckResult:
    name: str
    success: bool
    message: str
```

Then:

```python
def check_dns() -> CheckResult:
    return CheckResult(
        name="dns",
        success=True,
        message="DNS resolved"
    )
```

This is more expressive than returning unrelated tuples.

---

# 60. Function for Preflight Checks

```python
def run_preflight_checks():
    checks = [
        check_cluster_access,
        check_registry_access,
        check_namespace,
        check_secrets
    ]

    failures = []

    for check in checks:
        if not check():
            failures.append(
                check.__name__
            )

    return failures
```

Caller:

```python
failures = run_preflight_checks()

if failures:
    raise RuntimeError(
        f"Preflight failed: {failures}"
    )
```

---

# 61. Function for Kubernetes Pod Checks

Conceptual:

```python
def is_pod_healthy(pod) -> bool:
    if pod["phase"] != "Running":
        return False

    if not pod["ready"]:
        return False

    return True
```

This separates health logic from API retrieval.

---

# 62. Kubernetes Function Architecture

```text
get_pods()
    |
    v
parse_pod()
    |
    v
is_pod_healthy()
    |
    v
collect_failures()
    |
    v
generate_report()
```

This is easier to test than one huge function.

---

# 63. AWS Function Architecture

```text
create_client()
      |
      v
list_resources()
      |
      v
validate_resource()
      |
      v
classify_resource()
      |
      v
generate_report()
```

Avoid mixing all these responsibilities unnecessarily.

---

# 64. AWS Client as a Function Dependency

Instead of:

```python
def list_instances():
    ec2 = boto3.client("ec2")
    ...
```

For testing, consider:

```python
def list_instances(ec2_client):
    ...
```

Then production code passes the real client, while tests can pass a fake/mock client.

---

# 65. Dependency Injection

Example:

```python
def check_instances(
    client
):
    response = client.describe_instances()
    ...
```

Production:

```python
client = boto3.client("ec2")
check_instances(client)
```

Test:

```python
fake_client = FakeEC2Client()
check_instances(fake_client)
```

This improves testability.

---

# 66. Function Dependencies

Functions should receive important dependencies explicitly.

Instead of relying on:

```python
GLOBAL_CLIENT
GLOBAL_CONFIG
GLOBAL_ENVIRONMENT
```

prefer:

```python
def deploy(
    client,
    config,
    environment
):
    ...
```

Explicit dependencies make code easier to understand.

---

# 67. Function Factory

A function can return another function:

```python
def make_checker(threshold):
    def check(value):
        return value < threshold

    return check
```

Then:

```python
check_cpu = make_checker(80)
```

This is an example of a closure.

---

# 68. Closures

A closure is an inner function that remembers values from the enclosing scope.

```python
def make_logger(prefix):
    def log(message):
        print(
            f"{prefix}: {message}"
        )

    return log
```

Use closures when they simplify a design; do not use them merely to demonstrate advanced Python.

---

# 69. Lambda Functions

A lambda is a small anonymous function.

```python
double = lambda x: x * 2
```

Equivalent:

```python
def double(x):
    return x * 2
```

For production DevOps code, normal named functions are often clearer.

---

# 70. Lambda With Sorting

```python
resources = [
    {"name": "a", "cpu": 80},
    {"name": "b", "cpu": 30},
    {"name": "c", "cpu": 60}
]

resources.sort(
    key=lambda resource: resource["cpu"]
)
```

Now resources are sorted by CPU.

---

# 71. Higher-Order Functions

A function can accept another function:

```python
def run_check(check):
    return check()
```

Call:

```python
run_check(check_dns)
```

This pattern is useful for configurable validation pipelines.

---

# 72. Function List

```python
checks = [
    check_dns,
    check_tls,
    check_http
]

for check in checks:
    result = check()
    print(result)
```

Functions are first-class objects in Python.

---

# 73. Decorators

A decorator wraps a function to add behavior.

Basic structure:

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result

    return wrapper
```

---

# 74. Using a Decorator

```python
@decorator
def deploy():
    print("Deploying")
```

Calling:

```python
deploy()
```

runs the wrapper around the original function.

---

# 75. DevOps Use Case — Timing

```python
import time
from functools import wraps

def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.monotonic()

        try:
            return func(*args, **kwargs)
        finally:
            duration = (
                time.monotonic() - start
            )

            print(
                f"{func.__name__} "
                f"took {duration:.2f}s"
            )

    return wrapper
```

Use:

```python
@timed
def collect_metrics():
    ...
```

---

# 76. Why `@wraps` Matters

Without:

```python
@wraps(func)
```

metadata such as the wrapped function name and docstring can be lost.

Using `functools.wraps` preserves useful metadata.

---

# 77. Retry Decorator

Concept:

```python
@retry(...)
def call_api():
    ...
```

A retry decorator can centralize:

```text
Attempt count
Backoff
Jitter
Retryable exceptions
Logging
```

But do not blindly retry every function.

---

# 78. Retry Decorator Design

A production retry wrapper should define:

```text
Which exceptions are retryable?
Maximum attempts?
Maximum elapsed time?
Backoff?
Jitter?
Should final exception be re-raised?
Should retry count be logged?
```

This avoids hidden retry behavior.

---

# 79. Logging Decorator

Concept:

```python
@log_call
def deploy():
    ...
```

The decorator can log:

```text
Function started
Function completed
Duration
Failure
```

Do not log:

```text
Passwords
Tokens
Secrets
Sensitive payloads
```

---

# 80. Metrics Decorator

Conceptually:

```python
@measure_duration
def process_request():
    ...
```

A production implementation can record:

```text
Invocation count
Success count
Failure count
Duration
```

Be careful about high-cardinality labels.

---

# 81. Function and Observability

A reusable function can expose operational information:

```python
def check_service(service):
    start = time.monotonic()

    try:
        result = perform_check()
        return result

    finally:
        duration = (
            time.monotonic() - start
        )
        logger.info(
            "service_check_completed",
            extra={
                "service": service,
                "duration": duration
            }
        )
```

The exact logging approach depends on the application's logging framework.

---

# 82. Function and Structured Logging

Prefer:

```text
event=deployment_completed
environment=production
service=payment
duration_ms=1250
```

over:

```text
"payment deployment took 1250 ms"
```

Structured logging makes centralized search and aggregation easier.

---

# 83. Function and Prometheus Metrics

Conceptually:

```python
def deploy(service):
    deployment_attempts.inc()

    try:
        result = perform_deployment()
        deployment_success.inc()
        return result
    except Exception:
        deployment_failures.inc()
        raise
```

Avoid dynamic values such as unique request IDs as metric labels because they can create high cardinality.

---

# 84. Function and ELK

A function can emit structured application logs:

```python
logger.info(
    "deployment_completed",
    extra={
        "service": service,
        "environment": environment
    }
)
```

The log pipeline can then send those records to Elasticsearch for querying in Kibana.

---

# 85. Function and Kubernetes

A function can encapsulate:

```text
Get pod
Check state
Collect events
Collect logs
Return health result
```

Example architecture:

```text
check_pod()
    |
    +--> get_pod()
    +--> parse_status()
    +--> get_events_if_unhealthy()
    +--> return result
```

---

# 86. Function and EKS

Functions can encapsulate:

```text
EKS client creation
Cluster access validation
Pod inventory
Node checks
Deployment checks
ALB target checks
```

Keep AWS authentication/configuration separate from business logic where possible.

---

# 87. Function for Environment Safety

```python
def require_environment(
    expected: str
) -> None:
    actual = os.getenv(
        "ENVIRONMENT"
    )

    if actual != expected:
        raise RuntimeError(
            f"Expected {expected}, "
            f"got {actual}"
        )
```

Use this before dangerous operations.

---

# 88. Production Guard Function

```python
def require_production_approval(
    approved: bool
) -> None:
    if not approved:
        raise PermissionError(
            "Production approval required"
        )
```

Then:

```python
if environment == "production":
    require_production_approval(
        approved
    )
```

Safety checks should fail closed.

---

# 89. Dry-Run Function

```python
def update_resource(
    resource,
    *,
    dry_run=False
):
    if dry_run:
        return {
            "changed": False,
            "action": "would_update"
        }

    perform_update(resource)

    return {
        "changed": True,
        "action": "updated"
    }
```

This makes destructive automation safer.

---

# 90. Function for Idempotent Operations

```python
def ensure_replicas(
    deployment,
    desired: int
):
    current = get_replicas(
        deployment
    )

    if current == desired:
        return False

    scale(
        deployment,
        desired
    )

    return True
```

The function performs an action only when the desired state differs.

---

# 91. Function for Reconciliation

```python
def reconcile(
    resource,
    desired_state
):
    current_state = get_state(resource)

    if current_state == desired_state:
        return "already-compliant"

    apply_state(
        resource,
        desired_state
    )

    return "changed"
```

This is a fundamental automation pattern.

---

# 92. Function for Deployment Verification

```python
def verify_deployment(
    deployment,
    desired_replicas
):
    status = get_deployment_status(
        deployment
    )

    return (
        status["ready_replicas"]
        == desired_replicas
    )
```

Keep verification separate from deployment execution.

---

# 93. Function for Rollback Decision

```python
def should_rollback(
    error_rate,
    latency,
    error_threshold,
    latency_threshold
):
    return (
        error_rate > error_threshold
        or latency > latency_threshold
    )
```

Pure decision functions are easy to test.

---

# 94. Function for Automated Rollback

```python
def deploy_and_verify(
    deployment
):
    deploy(deployment)

    if not verify(deployment):
        rollback(deployment)
        raise RuntimeError(
            "Deployment verification failed"
        )
```

Production rollback should also account for:

```text
Rollback safety
Database compatibility
Migration state
Traffic state
Previous artifact availability
Approval policy
```

---

# 95. Function for Configuration Validation

```python
def validate_config(config):
    required = [
        "environment",
        "region",
        "cluster_name"
    ]

    missing = [
        key
        for key in required
        if not config.get(key)
    ]

    return missing
```

Caller:

```python
missing = validate_config(config)

if missing:
    raise ValueError(
        f"Missing: {missing}"
    )
```

---

# 96. Function for Secrets Validation

Do not return secret values.

```python
def validate_secret_names(
    config
):
    required = [
        "DB_PASSWORD",
        "API_TOKEN"
    ]

    return [
        name
        for name in required
        if not config.get(name)
    ]
```

Only report missing names.

---

# 97. Function for Log Parsing

```python
def parse_log_line(line):
    parts = line.split()

    return {
        "timestamp": parts[0],
        "level": parts[1],
        "message": " ".join(parts[2:])
    }
```

Real production logs should use structured formats where possible.

---

# 98. Function for Error Classification

```python
def classify_status(
    status_code: int
) -> str:
    if 200 <= status_code < 300:
        return "success"

    if 400 <= status_code < 500:
        return "client_error"

    if 500 <= status_code < 600:
        return "server_error"

    return "other"
```

This is a pure function.

---

# 99. Function for Error Rate

```python
def error_rate(
    errors: int,
    total: int
) -> float:
    if total == 0:
        return 0.0

    return (
        errors / total
    )
```

Unit-test boundary:

```text
0/0
0/100
1/100
100/100
```

---

# 100. Function for SLO Evaluation

```python
def slo_met(
    success_rate: float,
    target: float
) -> bool:
    return success_rate >= target
```

This is a good example of separating business/operational logic from data collection.

---

# 101. Function for MTTR

```python
def calculate_mttr(
    recovery_times: list[float]
) -> float:
    if not recovery_times:
        return 0.0

    return (
        sum(recovery_times)
        / len(recovery_times)
    )
```

Pure calculation functions are easy to test.

---

# 102. Function for Certificate Expiry

```python
def certificate_status(
    days_remaining: int
) -> str:
    if days_remaining <= 7:
        return "critical"

    if days_remaining <= 30:
        return "warning"

    return "normal"
```

Boundary tests are important.

---

# 103. Function for Resource Classification

```python
def classify_resource(resource):
    if resource["protected"]:
        return "protected"

    if resource["state"] == "running":
        return "active"

    return "candidate"
```

This function should not perform deletion.

---

# 104. Separate Decision From Action

Good:

```python
classification = classify_resource(
    resource
)

if classification == "candidate":
    delete_resource(resource)
```

Bad:

```python
def classify_resource(resource):
    if old(resource):
        delete_resource(resource)
        return "deleted"
```

Separating decision and action improves safety and testing.

---

# 105. Function Pipelines

Example:

```python
def process_resource(resource):
    validated = validate_resource(
        resource
    )

    normalized = normalize_resource(
        validated
    )

    result = classify_resource(
        normalized
    )

    return result
```

This creates a predictable processing pipeline.

---

# 106. Function Composition in CI/CD

```text
load_config()
      |
      v
validate_config()
      |
      v
run_tests()
      |
      v
run_security_scan()
      |
      v
build_image()
      |
      v
push_image()
      |
      v
deploy()
      |
      v
verify()
```

Each function should have a clear contract.

---

# 107. Main Function Pattern

Avoid putting the whole script at module level.

Prefer:

```python
def main():
    config = load_config()
    validate_config(config)
    deploy(config)

if __name__ == "__main__":
    main()
```

This makes the code easier to import and test.

---

# 108. Why `if __name__ == "__main__"`?

When the file is executed directly:

```text
__name__ == "__main__"
```

When imported:

```text
__name__ == module name
```

Therefore:

```python
if __name__ == "__main__":
    main()
```

prevents the script's main workflow from automatically running when imported.

---

# 109. CLI Function Architecture

A DevOps script can use:

```text
main()
 |
 +--> parse_arguments()
 |
 +--> load_config()
 |
 +--> validate()
 |
 +--> execute()
 |
 +--> report()
 |
 +--> exit()
```

This structure scales better than a single large script body.

---

# 110. Function Exit Codes

A CLI automation can return:

```python
def main() -> int:
    if success:
        return 0

    return 1
```

Then:

```python
if __name__ == "__main__":
    raise SystemExit(main())
```

This integrates cleanly with CI/CD.

---

# 111. Exit Code Design

Common concept:

```text
0 -> success
1 -> generic failure
2+ -> tool-specific errors
```

Use documented exit codes consistently.

---

# 112. Function for Command Execution

Conceptual:

```python
def run_command(
    command: list[str],
    timeout: int = 60
):
    return subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=True,
        timeout=timeout
    )
```

This centralizes command execution policy.

---

# 113. Safe Command Execution

Prefer:

```python
subprocess.run(
    ["kubectl", "get", "pods"],
    check=True
)
```

over constructing shell strings unnecessarily.

Avoid:

```python
subprocess.run(
    f"kubectl get pods {user_input}",
    shell=True
)
```

when input is untrusted.

---

# 114. Function for Kubernetes Command

```python
def get_pods():
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

    return json.loads(
        result.stdout
    )
```

For larger applications, use a Kubernetes client library instead of repeatedly invoking the CLI.

---

# 115. Function for AWS Client

```python
def create_ec2_client(
    region: str
):
    return boto3.client(
        "ec2",
        region_name=region
    )
```

Authentication should normally come from the AWS credential/provider chain rather than hardcoded credentials.

---

# 116. Function for AWS Inventory

```python
def list_instances(client):
    instances = []

    paginator = client.get_paginator(
        "describe_instances"
    )

    for page in paginator.paginate():
        for reservation in page[
            "Reservations"
        ]:
            instances.extend(
                reservation["Instances"]
            )

    return instances
```

For very large environments, returning the entire list may be memory-heavy; a generator or incremental processor may be better.

---

# 117. Generator Function

A function containing `yield` creates a generator.

```python
def iter_instances(client):
    paginator = client.get_paginator(
        "describe_instances"
    )

    for page in paginator.paginate():
        for reservation in page[
            "Reservations"
        ]:
            for instance in reservation[
                "Instances"
            ]:
                yield instance
```

Then:

```python
for instance in iter_instances(client):
    process(instance)
```

---

# 118. Generator vs List

List:

```python
instances = list_instances()
```

Potential issue:

```text
All records held in memory.
```

Generator:

```python
for instance in iter_instances():
    process(instance)
```

Potential advantage:

```text
Incremental processing
Lower memory usage
Natural streaming
```

---

# 119. Function for Kubernetes Inventory Generator

Concept:

```python
def iter_pods(client):
    for namespace in namespaces:
        pods = client.list_namespaced_pod(
            namespace
        )

        for pod in pods.items:
            yield pod
```

This allows callers to process pods incrementally.

---

# 120. Function for Log Streaming

```python
def iter_error_lines(path):
    with open(path) as file:
        for line in file:
            if "ERROR" in line:
                yield line
```

Then:

```python
for line in iter_error_lines(path):
    process_error(line)
```

---

# 121. Function Return vs Generator

Use a normal return when:

```text
Dataset is small
Caller needs random access
Caller needs length immediately
```

Use a generator when:

```text
Dataset is large
Processing is sequential
Memory matters
Source is paginated/streaming
```

---

# 122. Function Caching

Python supports caching:

```python
from functools import lru_cache

@lru_cache
def get_config(environment):
    return load_config(environment)
```

This can avoid repeated expensive work.

But be careful with:

```text
Stale configuration
Changing cloud state
Memory growth
Cache invalidation
```

Never blindly cache dynamic infrastructure state.

---

# 123. Function Memoization

Memoization stores results for repeated inputs.

Useful for:

```text
Expensive pure calculations
Stable configuration
Repeated deterministic lookups
```

Less suitable for:

```text
Live Kubernetes state
Live EC2 state
Current health checks
Changing deployment status
```

---

# 124. Function Testing

Example:

```python
def calculate_error_rate(
    errors,
    total
):
    if total == 0:
        return 0

    return errors / total
```

Tests:

```python
assert calculate_error_rate(
    0, 100
) == 0

assert calculate_error_rate(
    10, 100
) == 0.1
```

---

# 125. Boundary Testing

Test:

```text
0 errors
0 total
1 error
total == errors
negative values if invalid
very large values
```

For DevOps automation, boundary cases frequently reveal production bugs.

---

# 126. Testing Side-Effect Functions

Suppose:

```python
def delete_resource(client, resource_id):
    client.delete(resource_id)
```

Inject a fake/mock client:

```python
fake_client = FakeClient()

delete_resource(
    fake_client,
    "resource-1"
)

assert fake_client.deleted == [
    "resource-1"
]
```

This avoids deleting a real resource during tests.

---

# 127. Unit Test vs Integration Test

Unit test:

```text
Function in isolation
Mock/fake external dependencies
Fast
```

Integration test:

```text
Real API/service
Real infrastructure or test environment
Slower
Validates integration
```

DevOps automation often benefits from both.

---

# 128. Function Dependency Injection for Testing

Production:

```python
deploy(
    kubernetes_client,
    config
)
```

Test:

```python
deploy(
    fake_kubernetes_client,
    test_config
)
```

This reduces the need to run every test against real infrastructure.

---

# 129. Mocking Time

Polling functions are difficult to test if they always sleep.

A design can inject a sleep function:

```python
def wait_for_ready(
    get_status,
    sleep,
    timeout
):
    ...
```

Test code can provide:

```python
fake_sleep
```

instead of real `time.sleep`.

This makes tests fast.

---

# 130. Dependency Injection for HTTP

Instead of:

```python
def check_health(url):
    response = requests.get(url)
```

consider:

```python
def check_health(
    http_client,
    url
):
    response = http_client.get(url)
```

Now a fake HTTP client can be used in tests.

---

# 131. Function Contract

For every important function define:

```text
Inputs
Outputs
Side effects
Exceptions
Timeout behavior
Retry behavior
Security implications
```

Example:

```text
Function:
deploy_application()

Inputs:
environment, image_tag

Output:
DeploymentResult

Side effects:
Changes Kubernetes deployment

Raises:
DeploymentFailedError

Timeout:
10 minutes
```

---

# 132. Function Design for Production Automation

A strong function should be:

```text
Small enough to understand
Explicit about dependencies
Predictable in output
Clear about failures
Observable
Testable
Idempotent where possible
Safe for production
```

---

# 133. Avoid Giant Functions

Warning signs:

```text
100+ lines
Many nested conditions
Many unrelated responsibilities
Too many parameters
Many global variables
Hard to unit test
Repeated code
```

Refactor into focused functions.

---

# 134. Avoid Too Many Tiny Functions

Do not create:

```python
def get_name():
    return resource["name"]

def get_state():
    return resource["state"]
```

for every trivial operation without benefit.

The goal is useful abstraction, not maximum function count.

---

# 135. Avoid Too Many Parameters

Bad:

```python
def deploy(
    client,
    region,
    environment,
    cluster,
    namespace,
    service,
    image,
    replicas,
    timeout,
    dry_run,
    approval,
    ...
):
    ...
```

Consider a configuration object:

```python
@dataclass
class DeploymentConfig:
    environment: str
    cluster: str
    namespace: str
    service: str
    image: str
    replicas: int
    timeout: int = 600
    dry_run: bool = False
```

Then:

```python
def deploy(
    client,
    config: DeploymentConfig
):
    ...
```

---

# 136. Configuration Object Benefits

A configuration object provides:

```text
Named fields
Type hints
Validation
Less parameter confusion
Easier testing
Easier extension
```

This is useful for complex DevOps tools.

---

# 137. Function Validation

Validate inputs near the function boundary:

```python
def scale(
    deployment: str,
    replicas: int
):
    if replicas < 0:
        raise ValueError(
            "Replicas cannot be negative"
        )

    ...
```

Fail early for invalid inputs.

---

# 138. Environment Validation

```python
def validate_environment(
    environment: str
):
    allowed = {
        "dev",
        "stage",
        "production"
    }

    if environment not in allowed:
        raise ValueError(
            f"Unsupported environment: "
            f"{environment}"
        )
```

This prevents accidental typos from becoming infrastructure changes.

---

# 139. Production Environment Guard

```python
def validate_deployment(
    environment,
    approved
):
    if environment == "production":
        if not approved:
            raise PermissionError(
                "Production approval required"
            )
```

Use defense in depth for destructive actions.

---

# 140. Function Logging

Use structured logs where possible:

```python
logger.info(
    "deployment_started",
    extra={
        "environment": environment,
        "service": service
    }
)
```

For errors:

```python
logger.exception(
    "deployment_failed",
    extra={
        "service": service
    }
)
```

`logger.exception()` is useful inside an exception handler because it includes traceback information.

---

# 141. Avoid Logging Secrets

Bad:

```python
logger.info(
    "token=%s",
    token
)
```

Good:

```python
logger.info(
    "authentication_configured"
)
```

If identifying a secret is necessary, use a safe identifier rather than the value.

---

# 142. Function Metrics

A reusable operation can expose metrics:

```python
def process_resource(resource):
    attempts.inc()

    try:
        result = process(resource)
        successes.inc()
        return result
    except Exception:
        failures.inc()
        raise
```

Do not use unbounded values as metric labels.

---

# 143. Function Tracing Concept

A function that performs an external operation can become a trace span:

```text
deploy()
   |
   +--> validate()
   |
   +--> registry_lookup()
   |
   +--> kubernetes_apply()
   |
   +--> rollout_wait()
```

Tracing implementation belongs to the observability layer, but function boundaries naturally map to meaningful operations.

---

# 144. Function Timeout Design

External functions should have bounded waits:

```python
def get_status(
    client,
    timeout=10
):
    return client.get_status(
        timeout=timeout
    )
```

Do not let a network dependency block forever.

---

# 145. Function Retry Design

Prefer explicit:

```python
def call_registry(
    client,
    *,
    max_attempts=3
):
    ...
```

rather than hidden infinite retry.

The caller should understand the operational behavior.

---

# 146. Function Error Classification

```python
def is_retryable(error):
    return isinstance(
        error,
        (
            TimeoutError,
            RateLimitError
        )
    )
```

Then:

```python
if is_retryable(exc):
    retry()
else:
    raise
```

Centralized classification improves consistency.

---

# 147. Function for Backoff

```python
def calculate_backoff(
    attempt,
    base=2,
    maximum=30
):
    return min(
        base * (2 ** attempt),
        maximum
    )
```

This is pure and easy to test.

Add jitter separately when required.

---

# 148. Function for Polling

```python
def wait_until(
    check,
    *,
    timeout,
    interval
):
    deadline = (
        time.monotonic()
        + timeout
    )

    while time.monotonic() < deadline:
        if check():
            return True

        time.sleep(interval)

    return False
```

This is reusable but should be extended with explicit failure states and exception policies for production use.

---

# 149. Polling Function With Failure State

```python
def wait_for_deployment(
    get_status,
    *,
    timeout=600,
    interval=10
):
    deadline = (
        time.monotonic()
        + timeout
    )

    while time.monotonic() < deadline:
        status = get_status()

        if status == "complete":
            return

        if status == "failed":
            raise RuntimeError(
                "Deployment failed"
            )

        time.sleep(interval)

    raise TimeoutError(
        "Deployment timed out"
    )
```

This is a strong interview example.

---

# 150. Function for Batch Processing

```python
def process_in_batches(
    items,
    batch_size,
    processor
):
    for start in range(
        0,
        len(items),
        batch_size
    ):
        batch = items[
            start:start + batch_size
        ]

        processor(batch)
```

The processing behavior is injected through `processor`.

---

# 151. Function for Paginated Processing

```python
def process_pages(
    fetch_page,
    process_items
):
    token = None

    while True:
        response = fetch_page(token)

        process_items(
            response["items"]
        )

        token = response.get(
            "next_token"
        )

        if not token:
            break
```

Production version should detect repeated tokens and handle retryable API errors.

---

# 152. Function for Resource Validation

```python
def validate_resource(resource):
    errors = []

    if not resource.get("name"):
        errors.append("missing name")

    if not resource.get("environment"):
        errors.append(
            "missing environment"
        )

    return errors
```

Then:

```python
errors = validate_resource(resource)

if errors:
    report(errors)
```

---

# 153. Function for Complete Resource Processing

```python
def process_resource(resource):
    errors = validate_resource(
        resource
    )

    if errors:
        return {
            "success": False,
            "errors": errors
        }

    result = classify_resource(
        resource
    )

    return {
        "success": True,
        "classification": result
    }
```

This creates a clear processing contract.

---

# 154. Function for Incident Collection

```python
def collect_incident_data(
    service
):
    return {
        "service": service,
        "health": check_health(service),
        "logs": collect_logs(service),
        "metrics": collect_metrics(service),
        "events": collect_events(service)
    }
```

In production, each external call should have its own timeout and failure handling.

---

# 155. Incident Diagnostic Architecture

```text
collect_incident_data()
       |
       +--> health
       |
       +--> logs
       |
       +--> metrics
       |
       +--> events
       |
       v
correlate()
       |
       v
diagnosis/report
```

This is useful for automated troubleshooting tools.

---

# 156. Function for Kubernetes Incident Collection

Conceptually:

```python
def collect_pod_diagnostics(
    client,
    namespace,
    pod_name
):
    pod = get_pod(
        client,
        namespace,
        pod_name
    )

    result = {
        "pod": pod,
        "events": [],
        "logs": []
    }

    if not is_healthy(pod):
        result["events"] = get_events(
            client,
            namespace,
            pod_name
        )

        result["logs"] = get_logs(
            client,
            namespace,
            pod_name
        )

    return result
```

Only collect expensive diagnostics when needed.

---

# 157. Function for Automated Evidence Collection

A production incident tool may expose:

```python
def collect_evidence(service):
    return {
        "timestamp": now(),
        "service": service,
        "health": get_health(service),
        "metrics": get_metrics(service),
        "logs": get_logs(service),
        "deployment": get_deployment(service)
    }
```

Evidence collection should be read-only unless explicitly designed otherwise.

---

# 158. Function for Safe Remediation

```python
def remediate(resource, *, dry_run=True):
    if not is_safe_to_remediate(resource):
        return "blocked"

    if dry_run:
        return "would-remediate"

    apply_remediation(resource)
    return "remediated"
```

Defaulting destructive functions to dry-run can be a strong safety design.

---

# 159. Function for Approval

```python
def require_approval(
    *,
    environment,
    approved
):
    if (
        environment == "production"
        and not approved
    ):
        raise PermissionError(
            "Approval required"
        )
```

Keep authorization policy separate from the actual mutation.

---

# 160. Function for Audit Record

```python
def build_audit_record(
    action,
    resource,
    result
):
    return {
        "action": action,
        "resource": resource,
        "result": result,
        "timestamp": time.time()
    }
```

Do not include secret values in audit records.

---

# 161. Function for Report Generation

```python
def summarize_results(results):
    total = len(results)
    successful = sum(
        result["success"]
        for result in results
    )

    return {
        "total": total,
        "successful": successful,
        "failed": total - successful
    }
```

Keep calculations separate from presentation.

---

# 162. Function for Exit Status

```python
def exit_status(summary):
    if summary["failed"] == 0:
        return 0

    return 1
```

Then:

```python
raise SystemExit(
    exit_status(summary)
)
```

This integrates cleanly with CI/CD.

---

# 163. Main Function — Production Structure

```python
def main() -> int:
    config = load_config()

    validate_config(config)

    results = run_checks(
        config
    )

    summary = summarize_results(
        results
    )

    print_summary(summary)

    return exit_status(summary)


if __name__ == "__main__":
    raise SystemExit(main())
```

This is a strong baseline for Python CLI automation.

---

# 164. Main Function With Error Boundary

```python
def main() -> int:
    try:
        config = load_config()
        validate_config(config)
        run(config)
        return 0

    except KnownOperationalError as exc:
        logger.error(
            "operation_failed: %s",
            exc
        )
        return 1
```

Unexpected programming errors should generally remain visible rather than being silently converted into generic failures.

---

# 165. Function Design — Production Checklist

Before shipping a function:

```text
[ ] Is its responsibility clear?
[ ] Is the name descriptive?
[ ] Are inputs explicit?
[ ] Is the return contract clear?
[ ] Are side effects obvious?
[ ] Are exceptions meaningful?
[ ] Are timeouts defined for external calls?
[ ] Are retries limited?
[ ] Are secrets protected?
[ ] Is it testable?
[ ] Are dependencies injectable?
[ ] Is it idempotent where possible?
[ ] Does it log useful operational context?
[ ] Does it avoid unnecessary global state?
[ ] Does it avoid unnecessary API calls?
```

---

# 166. Common Mistake — Function Does Everything

Bad:

```python
def deploy():
    load_env()
    create_aws_client()
    check_cluster()
    read_yaml()
    build_image()
    push_image()
    update_git()
    deploy_argo()
    check_pods()
    send_email()
```

Better:

```python
def deploy():
    config = load_config()
    validate(config)
    artifact = build(config)
    publish(artifact)
    release(config, artifact)
    verify(config)
```

---

# 167. Common Mistake — Returning Inconsistent Types

Bad:

```python
def check():
    if success:
        return True

    return {
        "error": "failed"
    }
```

The caller now needs to handle unrelated types.

Better:

```python
@dataclass
class CheckResult:
    success: bool
    error: str | None = None
```

---

# 168. Common Mistake — Printing Instead of Returning

Bad:

```python
def get_status():
    print("healthy")
```

Better:

```python
def get_status():
    return "healthy"
```

Then the caller decides whether to:

```text
Print
Log
Alert
Store
Test
Return
```

---

# 169. Common Mistake — Hidden Global Client

Bad:

```python
client = boto3.client("ec2")

def get_instances():
    return client.describe_instances()
```

This can make testing harder.

Better:

```python
def get_instances(client):
    return client.describe_instances()
```

---

# 170. Common Mistake — Catching `Exception`

Bad:

```python
def run():
    try:
        ...
    except Exception:
        return False
```

This can hide:

```text
Programming errors
Unexpected failures
Configuration bugs
Incorrect assumptions
```

Catch expected exceptions at the appropriate boundary.

---

# 171. Common Mistake — Hidden Retries

Bad:

```python
def deploy():
    for _ in range(20):
        try:
            return call()
        except Exception:
            time.sleep(1)
```

The caller may not know that deployment can take 20 attempts.

Prefer explicit retry policy or a well-documented retry abstraction.

---

# 172. Common Mistake — No Timeout

Bad:

```python
def check_service(url):
    return requests.get(url)
```

External calls should normally have an explicit timeout.

Better:

```python
def check_service(url):
    return requests.get(
        url,
        timeout=5
    )
```

---

# 173. Common Mistake — Dangerous Default

Bad:

```python
def cleanup(
    dry_run=False
):
    ...
```

If the function is dangerous, consider:

```python
def cleanup(
    *,
    dry_run=True
):
    ...
```

or require an explicit confirmation/approval parameter.

---

# 174. Common Mistake — Too Much Magic

Avoid functions that silently:

```text
Read environment variables
Create clients
Modify infrastructure
Retry
Sleep
Rollback
Send alerts
```

without exposing those behaviors.

Predictability matters in production automation.

---

# 175. Common Mistake — High Cardinality Metrics

Bad:

```python
request_count.labels(
    request_id
).inc()
```

A unique request ID can create enormous metric cardinality.

Prefer stable dimensions:

```text
service
method
route
status
environment
```

where appropriate.

---

# 176. Common Mistake — Logging Function Arguments

Bad:

```python
logger.info(
    "Calling deploy with %s",
    kwargs
)
```

`kwargs` may contain secrets.

Log safe metadata explicitly.

---

# 177. Common Mistake — Function Too Generic

Bad:

```python
def process(data):
    ...
```

Better:

```python
def validate_kubernetes_deployment(
    deployment
):
    ...
```

Specific names improve operational readability.

---

# 178. Common Mistake — Function Too Coupled

Bad:

```python
def check_service():
    requests.get(
        "http://payment:8080/health"
    )
```

The URL is hidden.

Better:

```python
def check_service(
    url,
    timeout=5
):
    ...
```

Now the function can be reused and tested.

---

# 179. Common Mistake — Repeating Functions

If multiple functions contain:

```python
try:
    ...
except TimeoutError:
    ...
```

and the behavior is identical, consider a reusable abstraction.

But avoid creating abstractions before duplication is actually meaningful.

---

# 180. Scenario — Deployment Function Fails

You have:

```python
def deploy():
    run_deployment()
```

Production requirements:

```text
Timeout
Retry transient failures
Detect rollout failure
Collect diagnostics
Rollback when policy allows
Return non-zero exit code
```

Design:

```text
deploy()
   |
   +--> validate()
   +--> apply()
   +--> wait()
   +--> verify()
   |
   +--> failure -> diagnostics
                   |
                   +--> rollback
```

---

# 181. Scenario — AWS Script Is Hard to Test

Current:

```python
def cleanup():
    client = boto3.client("ec2")
    resources = client.describe_instances()
    ...
```

Refactor:

```python
def cleanup(client):
    resources = client.describe_instances()
    ...
```

Production:

```python
client = boto3.client("ec2")
cleanup(client)
```

Testing:

```python
cleanup(fake_client)
```

---

# 182. Scenario — Kubernetes Script Uses Shell Commands Everywhere

Current:

```python
subprocess.run(
    "kubectl ...",
    shell=True
)
```

Refactor where appropriate:

```text
Use Kubernetes client
Return structured objects
Separate collection from decision logic
Inject client for tests
```

CLI commands remain useful for small operational scripts, but structured APIs are generally more robust for larger automation.

---

# 183. Scenario — CI Pipeline Python Script Hides Failure

Bad:

```python
def main():
    try:
        deploy()
    except Exception:
        print("Deployment failed")
        return 0
```

This tells CI:

```text
0 = success
```

even though deployment failed.

Better:

```python
def main():
    try:
        deploy()
        return 0
    except KnownOperationalError:
        return 1
```

---

# 184. Scenario — Function Retries Non-Retryable Error

Fix the function design:

```python
def is_retryable(exc):
    return isinstance(
        exc,
        (
            TimeoutError,
            RateLimitError
        )
    )
```

Then retry only when:

```python
if is_retryable(exc):
    ...
else:
    raise
```

---

# 185. Scenario — Function Has 15 Parameters

Use a configuration object:

```python
@dataclass
class DeploymentConfig:
    environment: str
    cluster: str
    namespace: str
    service: str
    image: str
    replicas: int
    timeout: int
    dry_run: bool
```

Then:

```python
def deploy(
    client,
    config
):
    ...
```

This makes the API easier to evolve.

---

# 186. Scenario — Function Is Slow

Instrument it:

```python
start = time.monotonic()

result = function()

duration = (
    time.monotonic() - start
)
```

Then determine whether time is spent in:

```text
API calls
Database
File I/O
Serialization
Retries
Sleep
CPU computation
```

Optimize the actual bottleneck.

---

# 187. Interview — What Is a Function?

Strong answer:

> "A function is a reusable block of code that encapsulates a specific responsibility. In DevOps I use functions to separate AWS/Kubernetes API calls, validation, deployment logic, health checks, reporting, and error handling so the automation is reusable and testable."

---

# 188. Interview — Why Are Functions Important in DevOps?

Strong answer:

> "Functions prevent duplication, make automation easier to test, and let me separate external side effects from business logic. For example, I can have one function retrieve EKS pods, another determine whether a pod is healthy, and another generate the incident report."

---

# 189. Interview — `return` vs `print`

Strong answer:

> "`print()` sends output to the console, while `return` gives a value back to the caller. I generally use `return` for reusable DevOps functions because the caller may need to log, alert, test, or make another decision based on the result."

---

# 190. Interview — What Are `*args` and `**kwargs`?

Strong answer:

> "`*args` collects additional positional arguments into a tuple, while `**kwargs` collects additional keyword arguments into a dictionary. They are useful for flexible interfaces and wrappers, especially decorators, but I avoid them when an explicit function signature is clearer."

---

# 191. Interview — What Is a Mutable Default Argument Problem?

Strong answer:

> "Default arguments are evaluated when the function is defined. Using a mutable object such as a list as a default can cause state to persist across calls. I use `None` as the default and create the list inside the function."

---

# 192. Interview — What Is Dependency Injection?

Strong answer:

> "Instead of a function creating its own AWS or Kubernetes client, I pass the client into the function. That makes dependencies explicit and allows me to inject fake clients during testing."

---

# 193. Interview — How Would You Structure a DevOps Python Script?

Strong answer:

```text
main()
 |
 +--> parse arguments
 +--> load configuration
 +--> validate inputs
 +--> create/inject clients
 +--> execute workflow
 +--> verify result
 +--> generate summary
 +--> return exit code
```

The main workflow should orchestrate focused functions rather than contain every implementation detail.

---

# 194. Interview — How Do You Handle Exceptions?

Strong answer:

> "I catch expected operational exceptions at the appropriate boundary, preserve the original error context, retry only transient failures, and allow unexpected programming errors to remain visible. I avoid broad `except Exception` blocks that silently hide failures."

---

# 195. Interview — How Do You Make Functions Testable?

Strong answer:

```text
Explicit inputs
Explicit outputs
Dependency injection
Limited global state
Pure logic separated from side effects
Deterministic calculations
Mock/fake external clients
```

This is especially important for AWS and Kubernetes automation.

---

# 196. Interview — How Do You Design a Production Deployment Function?

Strong answer:

```text
validate configuration
      |
      v
validate authorization/approval
      |
      v
apply deployment
      |
      v
poll rollout with timeout
      |
      v
verify health
      |
      +--> success
      |
      +--> failure -> diagnostics
                         |
                         +--> rollback if policy allows
```

---

# 197. Interview — How Do You Add Observability to Functions?

Strong answer:

> "I add structured logs around meaningful state transitions, metrics for stable operational dimensions such as success/failure and duration, and tracing around significant external operations when tracing is available. I avoid logging secrets and avoid high-cardinality metric labels."

---

# 198. Interview — What Is Idempotency?

Strong answer:

> "An idempotent operation can be safely repeated without producing an unintended additional effect. In DevOps automation I prefer functions that ensure a desired state rather than blindly performing the same mutation every time."

Example:

```python
if current_replicas != desired_replicas:
    scale(desired_replicas)
```

---

# 199. Interview — How Do You Make Cleanup Safe?

Strong answer:

```text
Validate environment
Identify candidates
Check ownership
Check protection
Dry-run
Require approval when needed
Perform deletion
Verify
Audit
```

The decision logic should be separate from the destructive function.

---

# 200. Interview — Function vs Class?

Use a function when:

```text
Behavior is small and focused
State does not need to persist
Simple transformation/check
```

Consider a class when:

```text
Related state and behavior belong together
A client/configuration has lifecycle
Multiple operations share dependencies
The abstraction becomes clearer
```

Do not use classes merely because they are more advanced.

---

# 201. Interview — What Is a Generator Function?

Strong answer:

> "A generator function uses `yield` to produce values lazily. It is useful for large AWS/Kubernetes inventories, paginated APIs, and log processing because the caller can process records incrementally without keeping the entire dataset in memory."

---

# 202. Interview — What Is a Decorator?

Strong answer:

> "A decorator wraps a function and adds behavior without changing the function's main implementation. In DevOps automation, decorators can be useful for timing, logging, retry policies, or instrumentation, provided the behavior remains explicit and understandable."

---

# 203. Interview — What Is a Pure Function?

Strong answer:

> "A pure function produces the same output for the same inputs and does not modify external state. I prefer pure functions for validation and calculations because they are deterministic and easy to unit test."

---

# 204. Interview — Why Separate Side Effects?

Strong answer:

> "External operations such as changing an EKS deployment or deleting an AWS resource are harder to test and have real consequences. Separating them from decision logic lets me test the decision independently and add safety controls before executing the side effect."

---

# 205. Interview — Why Use `main()`?

Strong answer:

> "A `main()` function gives the script a clear entry point and allows the rest of the module to be imported without automatically executing the workflow. It also makes CLI exit-code handling and testing cleaner."

---

# 206. Interview — How Would You Test AWS Automation Without AWS?

Strong answer:

> "I would inject the AWS client into my functions and use a fake or mocked client in unit tests. I would test the decision and transformation logic independently, then use integration tests in a controlled AWS environment for actual API behavior."

---

# 207. Interview — How Would You Test Kubernetes Automation?

Strong answer:

> "I would separate Kubernetes API retrieval from the health and decision logic. Unit tests can use fake client responses, while integration tests can run against a controlled test cluster."

---

# 208. Interview — How Do You Handle External API Timeouts?

Strong answer:

> "Every external operation should have an explicit timeout. I distinguish timeout from other errors, retry only if the operation is safe and the failure is transient, use bounded backoff, and eventually fail with enough context for troubleshooting."

---

# 209. Interview — How Do You Prevent Hidden Side Effects?

Strong answer:

> "I keep read-only functions separate from mutation functions. A classification function should return a decision; a separate action function should perform the change. For dangerous actions I add dry-run and approval controls."

---

# 210. Interview — What Makes a Good DevOps Utility Function?

Strong answer:

```text
Focused responsibility
Explicit dependencies
Clear input/output contract
Bounded external calls
Good error handling
Useful observability
Safe defaults
Idempotency where possible
Unit-testable design
```

---

# 211. Advanced Scenario — Production EKS Rollout

Question:

> A Python deployment utility updates an EKS deployment and then waits for it to become healthy. How would you design the functions?

Strong structure:

```text
load_config()
validate_config()
validate_approval()
apply_deployment()
wait_for_rollout()
collect_diagnostics_if_failed()
verify_application()
rollback_if_policy_allows()
generate_report()
main()
```

Each function has one primary responsibility.

---

# 212. Advanced Scenario — AWS Cleanup Tool

Question:

> Design a Python tool that identifies old development EC2 instances and cleans them up.

Architecture:

```text
load_policy()
     |
     v
list_instances()
     |
     v
validate_tags()
     |
     v
classify_candidates()
     |
     v
generate_dry_run()
     |
     v
approval
     |
     v
stop/delete
     |
     v
verify
     |
     v
audit
```

Never make the inventory function itself delete resources.

---

# 213. Advanced Scenario — Monitoring Automation

Question:

> Design a function that checks multiple microservices and returns a production health report.

Architecture:

```text
load service definitions
        |
        v
check each endpoint
        |
        v
collect:
  status
  latency
  error
        |
        v
aggregate results
        |
        v
calculate summary
        |
        v
alert/report
```

Use bounded HTTP timeouts and avoid allowing one unhealthy service to prevent all other checks unless fail-fast is explicitly required.

---

# 214. Advanced Scenario — Centralized Log Processor

Question:

> A Python utility must process millions of log records.

Design:

```text
stream input
    |
    v
parse record
    |
    v
validate
    |
    v
classify
    |
    +--> error
    +--> warning
    +--> normal
    |
    v
aggregate bounded statistics
    |
    v
write report
```

Do not load millions of records into a list.

---

# 215. Advanced Scenario — API Rate Limit

Question:

> Your Python automation receives HTTP 429 responses.

Answer:

```text
Identify 429
   |
   v
Read Retry-After if provided
   |
   v
Apply bounded backoff
   |
   v
Retry only safe operation
   |
   v
Limit attempts/time
   |
   v
Fail with context
```

Also investigate why request volume is excessive.

---

# 216. Advanced Scenario — Failed Deployment

Question:

> Your function says deployment failed, but the pipeline still succeeds. What do you check?

Check:

```text
Exception handling
Return value
main() exit code
SystemExit
subprocess return code
CI shell configuration
Ignored exceptions
Rollback result
Verification result
```

A failed deployment must produce a non-success pipeline status unless explicitly handled as an allowed condition.

---

# 217. Advanced Scenario — Function Works Locally but Fails in CI

Investigate:

```text
Environment variables
AWS credentials
IAM permissions
PATH
Python version
Dependency versions
Working directory
File permissions
Network access
Kubernetes context
Secret availability
```

Functions should make required dependencies explicit.

---

# 218. Advanced Scenario — Function Works Manually but Fails in Kubernetes

Check:

```text
Service account
RBAC
Environment variables
Mounted configuration
DNS
NetworkPolicy
Container user
Filesystem permissions
Resource limits
Timeouts
```

Do not assume the container environment matches a developer workstation.

---

# 219. Advanced Scenario — Automation Hangs

Check:

```text
External API timeout
Infinite loop
Polling condition
Deadlock in dependencies
Subprocess waiting
Queue receive
Database query
Network connection
Missing termination condition
```

Instrument function boundaries and external calls.

---

# 220. Advanced Scenario — Memory Grows During Automation

Check:

```text
Large return lists
Global collections
Caches
Unbounded dictionaries
Stored log lines
Repeated API responses
Closures retaining objects
Batch size
```

Use:

```text
Generators
Pagination
Streaming
Bounded queues
Batch processing
```

---

# 221. Advanced Scenario — Production Function Needs Retry

Before adding retry ask:

```text
Is the error transient?
Is the operation idempotent?
Can the API be rate-limited?
Does the SDK already retry?
What is the maximum duration?
What happens after final failure?
```

Retry is an operational policy, not simply a `for` loop.

---

# 222. Advanced Scenario — Function Needs Multiple Clients

Instead of globals:

```python
def collect_data(
    aws_client,
    k8s_client,
    http_client
):
    ...
```

For larger applications, group related dependencies into a service/context object if that improves readability.

The key principle is:

```text
Dependencies should be explicit.
```

---

# 223. Practical Exercise 1 — Basic Function

Create:

```python
def greet_engineer(name):
    ...
```

Expected:

```text
Hello Surendra
```

Then add a default name.

---

# 224. Practical Exercise 2 — Health Function

Create:

```python
def is_healthy(
    status_code
):
    ...
```

Return:

```text
True for 2xx
False otherwise
```

Test:

```text
200
201
301
404
500
```

---

# 225. Practical Exercise 3 — Error Rate

Implement:

```python
calculate_error_rate(
    errors,
    total
)
```

Test:

```text
0/0
0/100
1/100
10/100
100/100
```

---

# 226. Practical Exercise 4 — Configuration Validation

Create:

```python
validate_config(config)
```

It should return all missing:

```text
environment
region
cluster_name
namespace
```

Do not stop at the first missing key.

---

# 227. Practical Exercise 5 — Kubernetes Health

Create:

```python
is_pod_healthy(pod)
```

Rules:

```text
phase == Running
ready == True
```

Otherwise false.

---

# 228. Practical Exercise 6 — AWS Resource Classification

Create:

```python
classify_resource(resource)
```

Return:

```text
protected
active
candidate
```

Do not perform any mutation.

---

# 229. Practical Exercise 7 — Retry Function

Create:

```python
retry_operation(
    operation,
    max_attempts=3
)
```

Requirements:

```text
Retry TimeoutError
Do not retry ValueError
Raise after final attempt
```

---

# 230. Practical Exercise 8 — Polling Function

Create:

```python
wait_for_status(
    get_status,
    timeout=60,
    interval=5
)
```

Handle:

```text
COMPLETE
FAILED
TIMEOUT
```

---

# 231. Practical Exercise 9 — AWS Dependency Injection

Create:

```python
def get_instances(client):
    ...
```

Use a fake client in a unit test.

Do not create a real AWS client inside the function.

---

# 232. Practical Exercise 10 — Generator

Create:

```python
def read_errors(path):
    ...
```

Use `yield` to return only error lines from a large file.

---

# 233. Practical Exercise 11 — Deployment Result

Create:

```python
@dataclass
class DeploymentResult:
    success: bool
    version: str
    message: str
```

Return it from:

```python
deploy()
```

---

# 234. Practical Exercise 12 — Preflight Pipeline

Create:

```python
run_preflight_checks()
```

with:

```text
check_aws_access
check_cluster_access
check_namespace
check_registry
check_secrets
```

Collect all failures.

---

# 235. Practical Exercise 13 — Dry-Run Cleanup

Create:

```python
cleanup(
    resources,
    dry_run=True
)
```

Requirements:

```text
Identify candidates
Print actions in dry-run
Never delete in dry-run
```

---

# 236. Practical Exercise 14 — Safe Production Guard

Create:

```python
require_production_approval(
    environment,
    approved
)
```

Rules:

```text
dev -> allowed
stage -> allowed
production + approved -> allowed
production + not approved -> fail
```

---

# 237. Practical Exercise 15 — Function Timing Decorator

Implement:

```python
@timed
def slow_operation():
    ...
```

It should report duration without changing the function's return value.

Use `functools.wraps`.

---

# 238. Practical Exercise 16 — Retry Decorator

Implement a simplified:

```python
@retry(max_attempts=3)
```

Only retry:

```text
TimeoutError
```

Do not retry:

```text
ValueError
PermissionError
```

---

# 239. Practical Exercise 17 — Batch Processor

Implement:

```python
process_in_batches(
    items,
    batch_size,
    processor
)
```

Test:

```text
0 items
1 item
exact batch size
multiple batches
partial final batch
```

---

# 240. Practical Exercise 18 — Pagination Function

Implement:

```python
process_pages(
    fetch_page,
    process_items
)
```

Test:

```text
one page
multiple pages
empty page
repeated token
API failure
```

---

# 241. Practical Exercise 19 — Incident Collector

Create:

```python
collect_incident_data(
    service
)
```

Return:

```text
health
metrics
logs
events
deployment
```

Use fake dependencies in tests.

---

# 242. Practical Exercise 20 — End-to-End DevOps Utility

Build:

```text
main()
 |
 +--> load configuration
 +--> validate configuration
 +--> create clients
 +--> run preflight checks
 +--> perform operation
 +--> poll
 +--> verify
 +--> collect diagnostics on failure
 +--> produce report
 +--> return exit code
```

This exercise combines the major function design principles in this file.

---

# 243. Production Architecture — Python DevOps Utility

A scalable utility can look like:

```text
                 CLI
                  |
                  v
               main()
                  |
          +-------+-------+
          |               |
          v               v
       config          logging
          |
          v
       validate
          |
          v
     service layer
          |
     +----+----+
     |    |    |
     v    v    v
    AWS  K8s  HTTP
     |    |    |
     +----+----+
          |
          v
      decision logic
          |
          v
       actions
          |
          v
       verify
          |
          v
       report
```

The key design principle is separation between:

```text
Infrastructure access
Business/operational logic
Mutation
Reporting
```

---

# 244. Production Architecture — Deployment Automation

```text
Git commit
    |
    v
CI pipeline
    |
    v
Python validation
    |
    +--> configuration
    +--> security
    +--> artifact
    |
    v
Deploy
    |
    v
Kubernetes/EKS
    |
    v
Python verification
    |
    +--> rollout
    +--> pod readiness
    +--> health
    +--> error rate
    |
    +--> failure
            |
            v
       diagnostics
            |
            v
       rollback policy
```

Python should support the deployment process rather than silently bypassing the controls of the CI/CD and GitOps system.

---

# 245. Production Architecture — Observability Automation

```text
Application
    |
    +--> Prometheus metrics
    +--> ELK logs
    +--> tracing when available
    |
    v
Python diagnostic utility
    |
    +--> query metrics
    +--> query logs
    +--> inspect Kubernetes
    +--> inspect deployment
    |
    v
Correlate evidence
    |
    v
Incident report
```

This is particularly useful during production troubleshooting.

---

# 246. Production Architecture — AWS Governance

```text
AWS Accounts
      |
      v
Regions
      |
      v
SDK paginators
      |
      v
Resource functions
      |
      v
Validation functions
      |
      v
Policy engine
      |
      +--> compliant
      |
      +--> non-compliant
                |
                v
             report
                |
                v
          optional remediation
```

Use least-privilege IAM and explicit account/environment boundaries.

---

# 247. Production Architecture — Kubernetes Governance

```text
EKS
 |
 +--> namespaces
 |
 +--> deployments
 |
 +--> pods
 |
 +--> services
 |
 +--> ingress
 |
 +--> nodes
 |
 v
Python collector
 |
 v
validation functions
 |
 +--> labels
 +--> resource limits
 +--> readiness
 +--> image policy
 +--> security policy
 |
 v
report
```

For large-scale policy enforcement, dedicated Kubernetes policy engines may be preferable to a custom Python script.

---

# 248. Production Architecture — Incident Automation

```text
Alert
  |
  v
Python diagnostic entry point
  |
  +--> identify service
  |
  +--> collect metrics
  |
  +--> collect logs
  |
  +--> inspect pods
  |
  +--> inspect deployment
  |
  +--> inspect dependencies
  |
  v
correlation
  |
  v
evidence report
  |
  +--> remediation allowed?
          |
       +--+--+
       |     |
      yes    no
       |     |
       v     v
   remediate escalate
```

Automated remediation should be conservative and explicitly authorized.

---

# 249. Production Architecture — Function Boundaries

Good boundary:

```python
metrics = collect_metrics(service)
logs = collect_logs(service)
health = check_health(service)

diagnosis = analyze(
    metrics,
    logs,
    health
)
```

Poor boundary:

```python
diagnose_everything()
```

The first design is easier to test and troubleshoot.

---

# 250. Production Architecture — Test Pyramid

```text
             E2E
              |
        Integration
              |
           Unit
```

For functions:

```text
Many fast unit tests
Some integration tests
Few expensive E2E tests
```

This is especially important when the functions interact with AWS, EKS, databases, or external APIs.

---

# 251. Production Security Checklist

For Python DevOps functions:

```text
[ ] No hardcoded credentials
[ ] No secret values in logs
[ ] Least-privilege IAM
[ ] Kubernetes RBAC
[ ] Validate untrusted input
[ ] Avoid unsafe shell execution
[ ] Explicit timeouts
[ ] Bounded retries
[ ] Secure temporary files
[ ] Safe production guards
[ ] Dry-run for destructive operations
[ ] Audit state-changing actions
[ ] Dependency scanning
```

---

# 252. Production Performance Checklist

```text
[ ] Avoid unnecessary API calls
[ ] Use SDK pagination
[ ] Stream large datasets
[ ] Use generators where appropriate
[ ] Batch operations
[ ] Reuse clients
[ ] Avoid repeated computation
[ ] Respect rate limits
[ ] Bound caches
[ ] Measure before optimizing
```

---

# 253. Production Reliability Checklist

```text
[ ] Explicit timeout
[ ] Retry only transient failures
[ ] Exponential backoff where appropriate
[ ] Jitter where appropriate
[ ] Maximum attempts
[ ] Maximum elapsed time
[ ] Idempotent actions
[ ] Clear failure states
[ ] Graceful shutdown for workers
[ ] Verification after mutation
[ ] Useful operational logs
```

---

# 254. Production Code Review Checklist

Review every important function for:

```text
Responsibility
Inputs
Outputs
Dependencies
Side effects
Exceptions
Timeouts
Retries
Logging
Security
Idempotency
Testing
Performance
```

---

# 255. Function Design Mental Model

Think:

```text
INPUT
  |
  v
VALIDATE
  |
  v
PROCESS
  |
  v
RETURN
```

For external operations:

```text
INPUT
  |
  v
VALIDATE
  |
  v
CALL EXTERNAL SYSTEM
  |
  v
HANDLE FAILURE
  |
  v
VERIFY
  |
  v
RETURN
```

For destructive operations:

```text
DISCOVER
  |
  v
VALIDATE
  |
  v
DRY RUN
  |
  v
APPROVE
  |
  v
ACT
  |
  v
VERIFY
  |
  v
AUDIT
```

---

# 256. Key Interview Takeaways

Remember these answers:

```text
Function
  -> reusable focused behavior

return
  -> sends result to caller

print
  -> console output

*args
  -> extra positional arguments

**kwargs
  -> extra keyword arguments

Dependency injection
  -> pass dependencies explicitly

Generator
  -> lazy/incremental values

Decorator
  -> wrap behavior around a function

Pure function
  -> deterministic, no external side effects

Idempotent function
  -> repeated execution produces the intended state

main()
  -> clear program entry point
```

---

# 257. Final DevOps Function Pattern

A strong production function often looks conceptually like:

```python
def operation(
    client,
    config,
    *,
    timeout=300,
    dry_run=True
):
    validate(config)

    if dry_run:
        return preview(
            client,
            config
        )

    result = execute(
        client,
        config,
        timeout=timeout
    )

    verify(
        client,
        config,
        result
    )

    return result
```

The exact implementation depends on the operation, but the principles are broadly reusable.

---

# 258. Final Summary

Functions are the foundation for turning Python into reliable DevOps automation.

You should be comfortable designing functions for:

```text
AWS
EKS/Kubernetes
Docker
Terraform
CI/CD
GitOps
Monitoring
Prometheus
Grafana
ELK
Logging
Health checks
Deployment verification
Incident diagnostics
Security validation
Resource inventory
Cleanup
Remediation
API integrations
```

The production mindset is:

```text
Reusable
   +
Testable
   +
Observable
   +
Safe
   +
Idempotent
   +
Bounded
   +
Explicit
   =
Production-ready Python automation
```

---

# 259. What To Remember for Interviews

When answering Python-for-DevOps questions, do not stop at syntax.

Explain:

```text
How the function is designed
Why the dependency is injected
How failures are handled
How retries are bounded
How timeouts work
How logs/metrics are emitted
How secrets are protected
How the function is tested
How production safety is enforced
```

Example:

> "For an EKS deployment utility, I would separate configuration validation, Kubernetes API interaction, rollout polling, health verification, diagnostics, and rollback into focused functions. I would inject the Kubernetes client for testing, use explicit timeouts and bounded retries, log structured operational events without secrets, and return a non-zero exit code when the deployment ultimately fails."

That is the level of answer expected from a DevOps engineer rather than only knowing Python syntax.

---