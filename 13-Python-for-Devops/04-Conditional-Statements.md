# Conditional-Statements

> Deep conditional-logic notes for DevOps engineers. Focus: reliable decision-making, validation, CI/CD gates, AWS/Kubernetes automation, monitoring checks, production safety, troubleshooting, interview questions, and practical exercises.

---

# 1. What Are Conditional Statements?

Conditional statements allow Python to make decisions based on conditions.

Basic structure:

```python
if condition:
    action()
```

DevOps automation constantly uses conditions:

```text
Is the deployment healthy?
Is CPU above the threshold?
Is this a production environment?
Did the security scan pass?
Are all Kubernetes replicas ready?
Should the pipeline continue?
Should a resource be deleted?
Should an alert be generated?
```

---

# 2. The `if` Statement

Example:

```python
cpu = 85

if cpu > 80:
    print("High CPU usage")
```

The block runs only when the condition evaluates to `True`.

Flow:

```text
CPU value
   |
   v
cpu > 80 ?
 /       \
Yes       No
 |         |
Alert     Continue
```

---

# 3. Python Indentation

Python uses indentation to define blocks.

Correct:

```python
if cpu > 80:
    print("High CPU")
    print("Investigate")
```

Incorrect:

```python
if cpu > 80:
print("High CPU")
```

Standard practice:

```text
4 spaces per indentation level
```

Avoid mixing tabs and spaces.

---

# 4. Multiple Statements in an `if`

```python
if deployment_failed:
    print("Deployment failed")
    collect_logs()
    notify_team()
```

All indented statements belong to the same block.

For production automation, keep the condition and resulting actions readable.

---

# 5. `if` With Comparison Operators

```python
status_code = 503

if status_code >= 500:
    print("Server error")
```

Common DevOps comparisons:

```python
cpu >= 90
memory >= 90
disk >= 90
error_rate >= 5
ready_replicas != desired_replicas
status_code >= 500
```

---

# 6. `if` With Strings

```python
environment = "production"

if environment == "production":
    print("Production environment")
```

Use:

```python
==
```

for value comparison.

Do not use:

```python
is
```

for normal string comparison.

---

# 7. `if` With Membership

```python
environment = "production"

if environment in {"dev", "stage", "production"}:
    print("Supported environment")
```

This is cleaner than:

```python
if (
    environment == "dev"
    or environment == "stage"
    or environment == "production"
):
    ...
```

---

# 8. `if` With `not in`

```python
environment = "test"

allowed = {"dev", "stage", "production"}

if environment not in allowed:
    raise ValueError("Unsupported environment")
```

This is an excellent pattern for configuration validation.

---

# 9. `if` With `is None`

```python
private_ip = None

if private_ip is None:
    print("Private IP unavailable")
```

Correct:

```python
is None
```

Avoid:

```python
== None
```

---

# 10. `if` With Multiple Conditions

```python
if (
    environment == "production"
    and error_rate > 5
):
    print("Production degradation")
```

The `and` operator means both conditions must be true.

---

# 11. `if` With `or`

```python
if cpu >= 90 or memory >= 90:
    print("Critical resource pressure")
```

This means:

```text
CPU critical
OR
Memory critical
```

At least one condition must be true.

---

# 12. Combining `and`, `or`, and `not`

Example:

```python
if (
    environment == "production"
    and (cpu >= 90 or memory >= 90)
    and not maintenance_mode
):
    trigger_incident()
```

Interpretation:

```text
Production
AND
(CPU critical OR memory critical)
AND
Not in maintenance
```

Use parentheses to make operational rules explicit.

---

# 13. The `else` Statement

`else` handles the alternative path.

```python
cpu = 70

if cpu >= 80:
    print("High CPU")
else:
    print("CPU is normal")
```

Flow:

```text
Condition?
   |
  / \
Yes  No
 |    |
High Normal
```

---

# 14. `if` + `else` Health Check

```python
status_code = 200

if 200 <= status_code < 300:
    print("Health check passed")
else:
    print("Health check failed")
```

This can be used in CI/CD or monitoring scripts.

---

# 15. The `elif` Statement

Use `elif` when there are multiple mutually exclusive cases.

```python
cpu = 87

if cpu >= 90:
    print("CRITICAL")
elif cpu >= 80:
    print("WARNING")
else:
    print("NORMAL")
```

Only the first matching branch executes.

---

# 16. Threshold Classification

A common monitoring pattern:

```python
def classify_cpu(cpu):
    if cpu >= 90:
        return "CRITICAL"
    elif cpu >= 80:
        return "WARNING"
    else:
        return "NORMAL"
```

Boundary behavior:

```text
90     -> CRITICAL
80     -> WARNING
79.99  -> NORMAL
```

---

# 17. Order of `if` / `elif`

Conditions are checked from top to bottom.

Correct:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
```

If you reverse them:

```python
if cpu >= 80:
    status = "WARNING"
elif cpu >= 90:
    status = "CRITICAL"
```

then `95` becomes `WARNING`.

The first matching condition wins.

---

# 18. Production Rule — Most Specific First

When ranges overlap:

```text
Critical
Warning
Normal
```

Check the most severe/specific condition first.

Example:

```python
if error_rate >= 5:
    status = "CRITICAL"
elif error_rate >= 1:
    status = "WARNING"
else:
    status = "NORMAL"
```

---

# 19. Nested `if` Statements

A conditional can contain another conditional.

```python
if environment == "production":
    if deployment_status == "success":
        print("Production deployment succeeded")
```

This works, but excessive nesting reduces readability.

---

# 20. Nested Conditions — DevOps Example

```python
if deployment_status == "success":
    if ready_replicas == desired_replicas:
        if error_rate < 1:
            print("Deployment healthy")
```

This can usually be simplified.

---

# 21. Flattening Nested Conditions

Instead of:

```python
if deployment_status == "success":
    if ready_replicas == desired_replicas:
        if error_rate < 1:
            deploy_complete = True
```

Prefer:

```python
if (
    deployment_status == "success"
    and ready_replicas == desired_replicas
    and error_rate < 1
):
    deploy_complete = True
```

This is easier to review.

---

# 22. Guard Clauses

Guard clauses reject invalid situations early.

Example:

```python
def deploy(environment):
    if environment not in {"dev", "stage", "production"}:
        raise ValueError("Invalid environment")

    print("Starting deployment")
```

This keeps the main path simple.

---

# 23. Guard Clause for Configuration

```python
def deploy(config):
    if not config:
        raise ValueError("Configuration is required")

    if "environment" not in config:
        raise ValueError("Environment is required")

    if "image" not in config:
        raise ValueError("Image is required")

    deploy_application(config)
```

Validate before performing operational actions.

---

# 24. Guard Clause for Production

```python
def destructive_operation(environment):
    if environment == "production":
        raise RuntimeError(
            "Destructive operation blocked in production"
        )

    perform_operation()
```

This is a simple safety barrier.

In real environments, authorization and policy controls should provide additional protection.

---

# 25. Truthiness

Python conditions do not always require explicit `True`/`False`.

Example:

```python
services = []

if services:
    print("Services found")
else:
    print("No services")
```

An empty list is falsy.

---

# 26. Common Falsy Values

These are commonly falsy:

```text
False
None
0
0.0
""
[]
{}
set()
```

Example:

```python
if not servers:
    print("No servers found")
```

---

# 27. Truthy Values

Most non-empty/non-zero values are truthy.

Examples:

```python
if "production":
    print("Runs")

if [1, 2]:
    print("Runs")

if {"status": "running"}:
    print("Runs")
```

Understand this when processing API responses and configuration.

---

# 28. Truthiness Pitfall With Numeric Values

Suppose:

```python
replicas = 0
```

Then:

```python
if replicas:
    print("Replicas configured")
```

does not execute.

If `0` is a valid configured value and you need to distinguish it from missing data, use an explicit check:

```python
if replicas is not None:
    print("Replica value provided")
```

---

# 29. Truthiness Pitfall With Environment Variables

```python
enabled = os.getenv("ENABLED")

if enabled:
    deploy()
```

If:

```text
ENABLED=false
```

the string is non-empty and therefore truthy.

Parse environment variables before using them as booleans.

---

# 30. Explicit Boolean Parsing

```python
import os

raw_enabled = os.getenv("ENABLED", "false").strip().lower()

if raw_enabled not in {"true", "false"}:
    raise ValueError("ENABLED must be true or false")

enabled = raw_enabled == "true"
```

Now:

```text
"true"  -> True
"false" -> False
```

---

# 31. Conditional Expressions

Simple one-line conditional:

```python
status = "PASS" if tests_passed else "FAIL"
```

Equivalent:

```python
if tests_passed:
    status = "PASS"
else:
    status = "FAIL"
```

Use conditional expressions only when they remain easy to read.

---

# 32. Avoid Nested Conditional Expressions

Avoid:

```python
status = (
    "CRITICAL"
    if cpu >= 90
    else "WARNING"
    if cpu >= 80
    else "NORMAL"
)
```

Although valid, a normal function is clearer:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
else:
    status = "NORMAL"
```

---

# 33. Conditional Logic With Functions

Instead of:

```python
if cpu >= 90:
    status = "CRITICAL"
```

create reusable logic:

```python
def classify_cpu(cpu):
    if cpu >= 90:
        return "CRITICAL"
    elif cpu >= 80:
        return "WARNING"
    return "NORMAL"
```

Then:

```python
status = classify_cpu(cpu)
```

---

# 34. Returning From Conditions

```python
def is_healthy(status_code):
    if 200 <= status_code < 300:
        return True

    return False
```

Can be simplified:

```python
def is_healthy(status_code):
    return 200 <= status_code < 300
```

Use the shorter form when the condition itself is clear.

---

# 35. Kubernetes Pod Health

A basic check:

```python
def pod_healthy(phase, ready):
    if phase == "Running" and ready:
        return True

    return False
```

Important:

```text
Running != necessarily healthy
```

A pod can be running while its application is not ready.

---

# 36. Kubernetes Deployment Gate

```python
def deployment_healthy(
    desired,
    ready,
    error_rate
):
    if desired != ready:
        return False

    if error_rate >= 1:
        return False

    return True
```

This can be used after a deployment to decide whether a pipeline should continue.

---

# 37. Kubernetes Condition With Status

```python
if phase != "Running":
    print("Pod is not running")
elif not ready:
    print("Pod is running but not ready")
else:
    print("Pod is healthy")
```

This provides more useful diagnostic output than one generic failure message.

---

# 38. AWS Resource Validation

```python
def is_cleanup_candidate(resource):
    if resource["environment"] != "dev":
        return False

    if resource["state"] != "stopped":
        return False

    if resource["age_days"] < 30:
        return False

    if resource["protected"]:
        return False

    return True
```

This is safer than one large destructive condition.

---

# 39. AWS Environment Guard

```python
if environment == "production":
    print("Production environment detected")
elif environment == "stage":
    print("Staging environment detected")
elif environment == "dev":
    print("Development environment detected")
else:
    raise ValueError("Unknown environment")
```

Explicitly reject unknown environments.

---

# 40. CI/CD Gate

```python
if not tests_passed:
    raise SystemExit("Tests failed")

if not security_scan_passed:
    raise SystemExit("Security scan failed")

if not image_available:
    raise SystemExit("Container image unavailable")

print("Pipeline can continue")
```

Guard clauses make pipeline failures immediate and clear.

---

# 41. DevSecOps Gate

A realistic sequence:

```python
if not build_passed:
    fail_pipeline("Build failed")

if not unit_tests_passed:
    fail_pipeline("Unit tests failed")

if not sast_passed:
    fail_pipeline("SAST failed")

if not image_scan_passed:
    fail_pipeline("Image scan failed")

if not deployment_manifest_valid:
    fail_pipeline("Manifest validation failed")

deploy()
```

This separates failure reasons.

---

# 42. Production Deployment Gate

```python
if environment == "production":
    if not approval_received:
        fail_pipeline("Production approval missing")

    if not health_checks_passed:
        fail_pipeline("Health checks failed")

    if error_rate >= 1:
        fail_pipeline("Error rate too high")

deploy()
```

Production workflows should also enforce authorization outside the script.

---

# 43. Monitoring Health Classification

```python
def classify_health(cpu, memory, disk):
    if cpu >= 90 or memory >= 90 or disk >= 90:
        return "CRITICAL"

    if cpu >= 80 or memory >= 80 or disk >= 80:
        return "WARNING"

    return "NORMAL"
```

This gives a single health state from multiple resource metrics.

---

# 44. Application Health Classification

```python
def application_health(
    status_code,
    latency_ms,
    error_rate
):
    if not 200 <= status_code < 300:
        return "UNHEALTHY"

    if latency_ms >= 500:
        return "DEGRADED"

    if error_rate >= 1:
        return "DEGRADED"

    return "HEALTHY"
```

This is more useful than only checking HTTP status.

---

# 45. Multi-Signal Decision Logic

A production application can have:

```text
HTTP 200
but
high latency

HTTP 200
but
high error rate

Pods Running
but
readiness failing
```

Therefore conditional logic should combine relevant signals.

Example:

```python
healthy = (
    200 <= status_code < 300
    and latency_ms < 500
    and error_rate < 1
    and ready_replicas == desired_replicas
)
```

---

# 46. Maintenance Mode

Maintenance can suppress some actions:

```python
if maintenance_mode:
    print("Maintenance mode active")
else:
    check_health()
    send_alerts()
```

However, do not blindly suppress critical safety/security checks.

---

# 47. Alert Suppression Example

```python
if maintenance_mode:
    print("Alert suppressed during maintenance")
elif error_rate >= 5:
    trigger_incident()
```

A production monitoring platform should generally handle alert inhibition/silencing centrally.

---

# 48. Feature Flags

```python
if enable_new_deployment:
    deploy_new_version()
else:
    deploy_current_version()
```

Feature flags should be explicitly configured and auditable.

---

# 49. Configuration-Based Behavior

```python
if config["environment"] == "production":
    replicas = 5
else:
    replicas = 1
```

Better:

```python
replicas = {
    "production": 5,
    "stage": 2,
    "dev": 1
}[config["environment"]]
```

But validate the key before indexing when the input is external:

```python
if config["environment"] not in replicas_by_environment:
    raise ValueError("Unknown environment")
```

---

# 50. `elif` vs Multiple Independent `if`

These are different.

With `elif`:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
```

Only one branch executes.

With independent `if`:

```python
if cpu >= 90:
    status = "CRITICAL"

if cpu >= 80:
    status = "WARNING"
```

For `cpu = 95`, the second condition overwrites the first.

Use `elif` for mutually exclusive classification.

---

# 51. Independent Conditions

Multiple `if` statements are correct when multiple actions may need to happen.

Example:

```python
if cpu >= 90:
    alert_cpu()

if memory >= 90:
    alert_memory()

if disk >= 90:
    alert_disk()
```

If all three are critical, all three checks should execute.

---

# 52. `if` / `elif` / `else` Decision Tree

Example:

```python
if status_code >= 500:
    result = "SERVER_ERROR"
elif status_code >= 400:
    result = "CLIENT_ERROR"
elif status_code >= 300:
    result = "REDIRECT"
elif status_code >= 200:
    result = "SUCCESS"
else:
    result = "INVALID"
```

This is a useful status-classification pattern.

---

# 53. HTTP Status Classification — Better Version

```python
def classify_http(status_code):
    if 500 <= status_code <= 599:
        return "SERVER_ERROR"

    if 400 <= status_code <= 499:
        return "CLIENT_ERROR"

    if 300 <= status_code <= 399:
        return "REDIRECT"

    if 200 <= status_code <= 299:
        return "SUCCESS"

    return "OTHER"
```

Separate `if` statements with immediate returns are also clear.

---

# 54. File Existence Check

```python
from pathlib import Path

config_file = Path("config.yaml")

if config_file.exists():
    print("Configuration exists")
else:
    raise FileNotFoundError("config.yaml not found")
```

This is common in deployment scripts.

---

# 55. File Type Check

```python
if config_file.is_file():
    print("Valid file")
elif config_file.is_dir():
    print("Path is a directory")
else:
    print("Path does not exist")
```

This avoids assuming a path has the expected type.

---

# 56. Command Result Validation

Example:

```python
result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True
)

if result.returncode == 0:
    print("Nginx is active")
else:
    print("Nginx is not active")
```

This is a practical infrastructure automation pattern.

---

# 57. Command Result With Output Validation

```python
if result.returncode != 0:
    print("Command failed")
elif "active" in result.stdout:
    print("Service is active")
else:
    print("Unexpected service state")
```

Do not rely only on output text when a reliable exit code is available.

---

# 58. API Response Validation

```python
response = {
    "status_code": 200,
    "body": {
        "healthy": True
    }
}

if response["status_code"] != 200:
    raise RuntimeError("API request failed")

if not response["body"].get("healthy", False):
    raise RuntimeError("Application unhealthy")
```

Validate the transport result and application result separately.

---

# 59. Required vs Optional Fields

Bad:

```python
replicas = response.get("replicas", 1)
```

if replicas are required.

Better:

```python
if "replicas" not in response:
    raise ValueError("Missing replicas")

replicas = response["replicas"]
```

Use defaults only when absence has a valid operational meaning.

---

# 60. Safe Nested Conditions

Potentially unsafe:

```python
if response["status"]["ready"]:
    ...
```

If fields are missing, the script fails with `KeyError`.

For optional API structures:

```python
status = response.get("status", {})
ready = status.get("ready", False)

if ready:
    ...
```

For required fields, fail explicitly instead.

---

# 61. Conditional Validation Function

```python
def validate_config(config):
    if not config:
        return False

    if config.get("environment") not in {
        "dev",
        "stage",
        "production"
    }:
        return False

    replicas = config.get("replicas")

    if not isinstance(replicas, int):
        return False

    if replicas < 1:
        return False

    return True
```

For production applications, richer error messages are usually preferable to a single boolean.

---

# 62. Better Validation With Explicit Errors

```python
def validate_config(config):
    if not config:
        raise ValueError("Configuration is empty")

    environment = config.get("environment")

    if environment not in {"dev", "stage", "production"}:
        raise ValueError("Invalid environment")

    replicas = config.get("replicas")

    if not isinstance(replicas, int):
        raise ValueError("Replicas must be an integer")

    if replicas < 1:
        raise ValueError("Replicas must be at least 1")
```

This is easier to troubleshoot.

---

# 63. Conditional Logic for Resource Cleanup

```python
def can_delete(resource):
    if resource["environment"] == "production":
        return False

    if resource["protected"]:
        return False

    if resource["state"] != "stopped":
        return False

    if resource["age_days"] < 30:
        return False

    return True
```

This is a safer pattern than one giant boolean expression.

---

# 64. Dry-Run Before Action

```python
if can_delete(resource):
    if dry_run:
        print(f"Would delete {resource['name']}")
    else:
        delete(resource)
```

Always design destructive automation so it can be tested without performing the destructive action.

---

# 65. Conditional Logging

```python
if deployment_failed:
    logger.error("Deployment failed")
elif deployment_degraded:
    logger.warning("Deployment degraded")
else:
    logger.info("Deployment healthy")
```

Use appropriate log levels.

---

# 66. Conditional Exit Codes

```python
if health_check_failed:
    raise SystemExit(1)

raise SystemExit(0)
```

CI/CD systems can use these exit codes:

```text
0     success
non-0 failure
```

---

# 67. Conditional Retry

```python
if error_code in retryable_errors:
    retry()
else:
    fail()
```

Not every failure should be retried.

Typical non-retryable examples:

```text
Invalid credentials
Malformed request
Permission denied
Invalid configuration
```

Typical retryable examples may include:

```text
Temporary network failure
Transient service unavailable
Rate limiting
Temporary connection failure
```

---

# 68. Conditional Retry With Attempt Limit

```python
if retryable and attempt < max_attempts:
    retry()
else:
    fail()
```

A production retry system should also account for total elapsed time.

---

# 69. Conditional Backoff

```python
if retry_count == 0:
    delay = 1
elif retry_count == 1:
    delay = 2
else:
    delay = 5
```

For real systems, formula-based exponential backoff is generally more maintainable:

```python
delay = min(
    base_delay * (2 ** retry_count),
    max_delay
)
```

---

# 70. Monitoring Script Example

```python
cpu = 91
memory = 70
disk = 65

if cpu >= 90:
    print("CRITICAL: CPU")
elif cpu >= 80:
    print("WARNING: CPU")

if memory >= 90:
    print("CRITICAL: Memory")
elif memory >= 80:
    print("WARNING: Memory")

if disk >= 90:
    print("CRITICAL: Disk")
elif disk >= 80:
    print("WARNING: Disk")
```

Notice the independent `if` blocks. Multiple resources can be unhealthy at the same time.

---

# 71. Deployment Monitoring Example

```python
if deployment_started:
    if not rollout_complete:
        print("Rollout still in progress")
    elif not readiness_passed:
        print("Rollout completed but readiness failed")
    elif error_rate >= 1:
        print("Rollout completed but error rate is high")
    else:
        print("Deployment healthy")
```

This provides progressive diagnostics.

---

# 72. Better With Guard Clauses

```python
if not deployment_started:
    raise RuntimeError("Deployment did not start")

if not rollout_complete:
    raise RuntimeError("Rollout incomplete")

if not readiness_passed:
    raise RuntimeError("Readiness failed")

if error_rate >= 1:
    raise RuntimeError("Error rate too high")

print("Deployment healthy")
```

This is usually easier to operate and troubleshoot.

---

# 73. Production Decision Function

```python
def deployment_status(
    started,
    rollout_complete,
    readiness_passed,
    error_rate
):
    if not started:
        return "NOT_STARTED"

    if not rollout_complete:
        return "IN_PROGRESS"

    if not readiness_passed:
        return "NOT_READY"

    if error_rate >= 1:
        return "DEGRADED"

    return "HEALTHY"
```

This converts several signals into a clear state.

---

# 74. Why Clear States Matter

Instead of:

```text
False
```

returning:

```text
NOT_STARTED
IN_PROGRESS
NOT_READY
DEGRADED
HEALTHY
```

This improves:

```text
Troubleshooting
Logging
Alerting
Dashboards
CI/CD messages
Incident analysis
```

---

# 75. Conditional Logic and Observability

A monitoring automation can evaluate:

```text
Metrics
Logs
Health endpoints
Deployment state
Kubernetes state
AWS resource state
```

Example:

```python
if error_rate >= 5 and request_count > 100:
    incident = True
else:
    incident = False
```

The goal is not merely detecting a metric threshold, but deciding whether the condition represents meaningful impact.

---

# 76. Conditional Logic and SLOs

```python
if availability < 99.9:
    slo_status = "VIOLATED"
else:
    slo_status = "MET"
```

For more mature systems, SLO alerting may use burn-rate logic rather than only a simple threshold.

---

# 77. Error Budget Condition

```python
if actual_errors > allowed_errors:
    print("Error budget exceeded")
elif actual_errors == allowed_errors:
    print("Error budget exhausted")
else:
    print("Error budget available")
```

Boundary conditions matter.

---

# 78. Environment-Specific Behavior

```python
if environment == "production":
    timeout = 30
    replicas = 5
elif environment == "stage":
    timeout = 15
    replicas = 2
elif environment == "dev":
    timeout = 10
    replicas = 1
else:
    raise ValueError("Unknown environment")
```

Do not allow unexpected environments to silently fall into a default production-like behavior.

---

# 79. Feature Flag Safety

```python
if feature_enabled and environment != "production":
    enable_feature()
```

If production requires approval:

```python
if (
    feature_enabled
    and (
        environment != "production"
        or approval_received
    )
):
    enable_feature()
```

Complex policy logic should be encapsulated in a named function.

---

# 80. Conditional Permissions

Example:

```python
if environment == "production" and destructive_action:
    if not approved:
        raise PermissionError(
            "Production destructive action not approved"
        )
```

Application logic is not a replacement for IAM/RBAC, but it can provide an additional safety layer.

---

# 81. Conditional Configuration Loading

```python
if environment == "production":
    config_file = "config-prod.yaml"
elif environment == "stage":
    config_file = "config-stage.yaml"
else:
    config_file = "config-dev.yaml"
```

Prefer centralized configuration management for larger systems.

---

# 82. Conditional AWS Region Handling

```python
if region not in allowed_regions:
    raise ValueError(
        f"Unsupported region: {region}"
    )
```

This can prevent accidental operations in unauthorized or unintended regions.

---

# 83. Conditional Kubernetes Namespace Handling

```python
allowed_namespaces = {
    "dev",
    "stage",
    "production"
}

if namespace not in allowed_namespaces:
    raise ValueError("Invalid namespace")
```

For destructive operations, namespace validation is especially important.

---

# 84. Production Incident Decision

Example:

```python
if (
    environment == "production"
    and error_rate >= 5
    and request_count >= 100
):
    incident = True
```

Better:

```python
def should_trigger_incident(
    environment,
    error_rate,
    request_count
):
    if environment != "production":
        return False

    if request_count < 100:
        return False

    if error_rate < 5:
        return False

    return True
```

The second version is easier to test.

---

# 85. Conditions With Time

Example:

```python
if certificate_days_remaining <= 30:
    print("Certificate renewal required")
```

For production:

```text
<= 30 -> warning
<= 7  -> critical
```

Use timezone-aware date/time calculations when comparing timestamps.

---

# 86. Conditions With File Age

```python
if file_age_days > retention_days:
    delete_file()
```

Add safety:

```python
if (
    file_age_days > retention_days
    and not protected
    and environment == "dev"
):
    delete_file()
```

---

# 87. Conditions With Resource Cost

```python
if (
    resource["environment"] == "dev"
    and resource["monthly_cost"] > 100
    and resource["unused"]
):
    print("Candidate for review")
```

A recommendation/review action is safer than automatically deleting based on cost alone.

---

# 88. Conditions With Security Scans

```python
if not sast_passed:
    fail_pipeline("SAST failed")

if critical_vulnerabilities > 0:
    fail_pipeline(
        "Critical vulnerabilities detected"
    )

if image_scan_passed:
    deploy()
```

Security policy should define exactly which severities block releases.

---

# 89. Conditional Pipeline Flow

```text
Source
  |
  v
Build
  |
  v
Tests?
 /   \
No    Yes
 |      |
Stop   Security
          |
          v
       Passed?
        /   \
       No    Yes
       |      |
      Stop   Deploy
                |
                v
             Health?
              /   \
             No    Yes
             |      |
            Rollback Complete
```

Python can implement individual decision points inside CI/CD scripts.

---

# 90. Conditional Rollback

```python
if error_rate >= rollback_threshold:
    rollback()
elif not readiness_passed:
    rollback()
else:
    continue_deployment()
```

Production rollback logic should also consider rollout state, blast radius, and whether rollback is actually safer than mitigation.

---

# 91. Conditional Canary Promotion

```python
if (
    error_rate < 1
    and latency_ms < 300
    and ready_replicas == desired_replicas
):
    promote_canary()
else:
    stop_canary()
```

This is a realistic Kubernetes deployment automation pattern.

---

# 92. Conditional Blue/Green Validation

```python
if (
    new_environment_healthy
    and smoke_tests_passed
    and error_rate < 1
):
    switch_traffic()
else:
    keep_current_environment()
```

The condition protects the traffic switch.

---

# 93. Conditional Auto-Remediation

```python
if pod_restarts > restart_threshold:
    collect_diagnostics()

    if application_config_valid:
        restart_pod()
    else:
        escalate_incident()
```

Do not automatically remediate every symptom. First distinguish between safe transient failures and configuration or application defects.

---

# 94. Conditions and Idempotency

A remediation action should ideally be safe to run repeatedly.

Example:

```python
if desired_replicas != current_replicas:
    scale_deployment(desired_replicas)
```

If they are already equal, do nothing.

This is a basic idempotent decision pattern.

---

# 95. Conditional Resource Creation

```python
if not resource_exists:
    create_resource()
else:
    print("Resource already exists")
```

This prevents unnecessary duplicate creation.

---

# 96. Conditional Configuration Update

```python
if current_config != desired_config:
    update_config(desired_config)
else:
    print("Configuration already matches desired state")
```

This is the conceptual foundation of declarative automation.

---

# 97. Conditional State Reconciliation

```text
Current state
     |
     v
Compare with desired state
     |
     +---- same ----> No action
     |
     +---- different -> Apply change
```

Python conditions are often used to implement this basic reconciliation model.

---

# 98. Production Example — Safe Service Restart

```python
if service_status != "active":
    if environment == "production":
        if approval_received:
            restart_service()
        else:
            escalate()
    else:
        restart_service()
```

A cleaner version:

```python
if service_status == "active":
    return

if environment == "production" and not approval_received:
    escalate()
    return

restart_service()
```

Guard clauses reduce nesting.

---

# 99. Production Example — Disk Cleanup

```python
if disk_usage < 80:
    return

if environment == "production":
    create_incident()
    return

cleanup_candidates = find_old_files()

if dry_run:
    report(cleanup_candidates)
else:
    delete(cleanup_candidates)
```

This separates:

```text
Detection
Environment safety
Candidate discovery
Dry-run
Action
```

---

# 100. Production Example — Deployment Verification

```python
if rollout_status != "complete":
    return "ROLLOUT_IN_PROGRESS"

if ready_replicas != desired_replicas:
    return "REPLICA_MISMATCH"

if error_rate >= 1:
    return "HIGH_ERROR_RATE"

if latency_ms >= 500:
    return "HIGH_LATENCY"

return "HEALTHY"
```

This is a strong pattern for deployment validation.

---

# 101. Troubleshooting Conditional Logic

When a script makes the wrong decision:

```text
1. Print/log the input values
2. Verify their Python types
3. Verify threshold values
4. Check comparison operators
5. Check AND/OR grouping
6. Check if/elif ordering
7. Check truthiness
8. Check boundary conditions
9. Check default values
10. Add a focused test
```

Example:

```python
print(
    f"cpu={cpu!r}, "
    f"type={type(cpu).__name__}, "
    f"threshold={CPU_THRESHOLD}"
)
```

---

# 102. Troubleshooting a False Alert

Suppose:

```python
cpu = "90"
```

and:

```python
if cpu >= 90:
    alert()
```

This fails because the types differ.

Debug:

```python
print(type(cpu))
```

Then convert:

```python
cpu = float(cpu)
```

This is why input normalization should happen before decision logic.

---

# 103. Troubleshooting a Missing Alert

Requirement:

```text
Alert at CPU >= 90
```

Code:

```python
if cpu > 90:
    alert()
```

At exactly `90`, nothing happens.

Test the boundary:

```python
assert classify_cpu(90) == "CRITICAL"
```

---

# 104. Troubleshooting Wrong Branch

If:

```python
cpu = 95
```

but output is:

```text
WARNING
```

inspect the ordering:

```python
if cpu >= 80:
    status = "WARNING"
elif cpu >= 90:
    status = "CRITICAL"
```

The first condition matches.

Fix:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
```

---

# 105. Troubleshooting Boolean Logic

Requirement:

```text
Production AND high CPU OR high memory
```

Clarify whether the intended rule is:

```text
(production AND high CPU)
OR high memory
```

or:

```text
production AND (high CPU OR high memory)
```

These are different.

Write parentheses explicitly.

---

# 106. Troubleshooting `if` vs `elif`

If multiple actions should happen:

```python
if cpu >= 90:
    alert_cpu()

if memory >= 90:
    alert_memory()
```

If only one classification should happen:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
else:
    status = "NORMAL"
```

Choose based on the business requirement.

---

# 107. Unit Testing Conditional Logic

For:

```python
def classify_disk(disk):
    if disk >= 90:
        return "CRITICAL"
    elif disk >= 80:
        return "WARNING"
    return "NORMAL"
```

Test:

```python
def test_disk_boundaries():
    assert classify_disk(79.99) == "NORMAL"
    assert classify_disk(80) == "WARNING"
    assert classify_disk(89.99) == "WARNING"
    assert classify_disk(90) == "CRITICAL"
```

Boundary tests are especially valuable for monitoring logic.

---

# 108. Testing All Branches

For:

```python
if A:
    ...
elif B:
    ...
else:
    ...
```

test:

```text
A true
A false + B true
A false + B false
```

For production logic, also test:

```text
Missing values
Boundary values
Invalid values
Unexpected states
```

---

# 109. Decision Tables

For complex rules, create a decision table before coding.

Example:

```text
Environment   Error Rate   Requests   Action
------------------------------------------------
dev           any          any        no incident
prod          < 5          < 100      no incident
prod          >= 5         < 100      investigate sample
prod          >= 5         >= 100     incident
```

Then translate the table into code.

This prevents ambiguous conditions.

---

# 110. Production Safety Checklist

Before deploying conditional automation:

```text
[ ] Inputs are normalized
[ ] Input types are validated
[ ] Required fields are present
[ ] Thresholds are explicit
[ ] Boundary values are tested
[ ] if/elif ordering is correct
[ ] and/or grouping is explicit
[ ] Production safeguards exist
[ ] Destructive actions support dry-run
[ ] Retry limits exist
[ ] Unknown environments fail safely
[ ] Decisions are logged
[ ] Exit codes are meaningful
[ ] Unit tests cover every branch
```

---

# 111. Interview Question — What Is an `if` Statement?

Strong answer:

> "`if` evaluates a condition and executes a block when the condition is true. In DevOps automation I use it for validation, health checks, deployment gates, environment-specific behavior, and operational safety checks."

---

# 112. Interview Question — `if` vs `elif` vs `else`

Strong answer:

> "`if` evaluates the first condition, `elif` provides additional mutually exclusive conditions, and `else` handles the remaining case. I use `elif` for classifications such as NORMAL/WARNING/CRITICAL and separate `if` statements when multiple independent actions may need to execute."

---

# 113. Interview Question — Why Is `elif` Ordering Important?

Strong answer:

> "Python evaluates branches from top to bottom and executes the first matching branch. Therefore overlapping thresholds must usually be ordered from the most specific or severe condition to the least severe."

Example:

```python
if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 80:
    status = "WARNING"
```

---

# 114. Interview Question — What Is a Guard Clause?

Strong answer:

> "A guard clause handles an invalid or exceptional condition early and returns or raises an error before the main workflow continues. It reduces nested conditions and makes production automation easier to read and troubleshoot."

---

# 115. Interview Question — What Is Truthiness?

Strong answer:

> "Python allows objects to be evaluated directly in conditions. Values such as `None`, `False`, zero, empty strings, lists, dictionaries and sets are falsy, while most non-empty values are truthy."

---

# 116. Interview Question — Why Is `if os.getenv("ENABLED")` Dangerous?

Strong answer:

> "Environment variables are strings. The string `'false'` is non-empty and therefore truthy. I explicitly parse boolean environment variables instead of relying on Python truthiness."

---

# 117. Interview Question — How Would You Implement a CPU Alert?

Strong answer:

```python
CPU_WARNING = 80
CPU_CRITICAL = 90

if cpu >= CPU_CRITICAL:
    alert("CRITICAL")
elif cpu >= CPU_WARNING:
    alert("WARNING")
```

Then explain that production alerting should additionally consider duration, service scope, baseline behavior, and alert noise.

---

# 118. Interview Question — How Would You Validate a Kubernetes Deployment?

Strong answer:

```text
Check rollout status
Check desired vs ready replicas
Check pod readiness
Check application health
Check error rate
Check latency
Fail the pipeline if required conditions are not satisfied
```

Example:

```python
if ready_replicas != desired_replicas:
    fail_pipeline("Replica mismatch")

if error_rate >= 1:
    fail_pipeline("High error rate")

print("Deployment healthy")
```

---

# 119. Interview Question — How Would You Make Cleanup Safe?

Strong answer:

> "I would use explicit environment and resource-state checks, ownership/protection checks, age or policy conditions, dry-run mode, logging, and ideally approval for high-risk operations. I would fail closed when required metadata is missing."

---

# 120. Interview Question — How Do You Handle Unknown Input?

Strong answer:

> "I validate it before making an operational decision. Unknown environments, invalid thresholds, missing required fields, and unsupported states should normally fail clearly rather than silently falling back to a potentially unsafe default."

---

# 121. Interview Question — How Do You Test Conditional Logic?

Strong answer:

> "I test every branch and especially boundary conditions. For threshold logic I test just below, exactly at, and just above the threshold. I also test missing, invalid, and unexpected inputs."

---

# 122. Scenario — Deployment Says Success but Pods Are Not Ready

Approach:

```text
1. Do not trust deployment exit status alone
2. Check rollout status
3. Compare desired vs ready replicas
4. Check pod readiness
5. Inspect pod events/logs
6. Check application health
7. Check error rate and latency
8. Fail/rollback according to deployment policy
```

Python decision example:

```python
if rollout_status != "complete":
    return "IN_PROGRESS"

if ready_replicas != desired_replicas:
    return "NOT_READY"

if error_rate >= 1:
    return "DEGRADED"

return "HEALTHY"
```

---

# 123. Scenario — Pipeline Deploys to Wrong Environment

Potential causes:

```text
Incorrect environment variable
Wrong comparison
Unknown environment silently accepted
Default configuration
Wrong branch mapping
Incorrect configuration file
```

Use explicit validation:

```python
allowed = {"dev", "stage", "production"}

if environment not in allowed:
    raise ValueError("Invalid environment")
```

Then map branches/environments explicitly.

---

# 124. Scenario — Cleanup Deleted a Protected Resource

Investigate:

```text
Was protection metadata read?
Was environment validated?
Was the condition inverted?
Was missing data treated as safe?
Was dry-run enabled?
Were unit tests present?
```

Safer principle:

```text
Missing protection information
        |
        v
Do not delete
```

Fail closed for destructive automation.

---

# 125. Scenario — Health Check Passes but Users Report Errors

A simple condition may be incomplete:

```python
if status_code == 200:
    healthy = True
```

A better check may include:

```text
HTTP status
Latency
Error rate
Dependency health
Application-specific response
Kubernetes readiness
```

A `200` response alone does not prove end-to-end application health.

---

# 126. Scenario — Alert Fires Too Often

Possible causes:

```text
Threshold too sensitive
No duration condition
No grouping
No inhibition
No maintenance handling
No baseline
Alerting on symptoms rather than impact
```

Python can implement filtering logic, but centralized alerting systems should handle production alert routing, grouping, silencing, and inhibition.

---

# 127. Scenario — Automation Runs Forever

Inspect:

```python
while condition:
    ...
```

Check:

```text
Does condition eventually change?
Is there a maximum attempt count?
Is there a timeout?
Is retry_count incremented?
Are terminal errors handled?
```

Safer pattern:

```python
for attempt in range(max_attempts):
    if operation_succeeded():
        break
else:
    raise RuntimeError("Maximum attempts exceeded")
```

---

# 128. Scenario — Production Logic Is Too Complex

Refactor:

```text
Large if expression
       |
       v
Named helper conditions
       |
       v
Small validation functions
       |
       v
Clear decision function
       |
       v
Unit tests
```

Example:

```python
def production_safe(config):
    return (
        config.environment == "production"
        and config.approved
        and not config.maintenance
    )
```

Then:

```python
if production_safe(config):
    deploy()
```

---

# 129. Production Architecture Pattern

Conditional logic commonly sits here:

```text
             AWS / Kubernetes / APIs
                       |
                       v
                Python Automation
                       |
          +------------+-------------+
          |            |             |
          v            v             v
       Validate     Calculate     Classify
          |            |             |
          +------------+-------------+
                       |
                       v
                  Decision Logic
                       |
          +------------+-------------+
          |            |             |
          v            v             v
       Continue      Retry        Escalate
          |            |             |
          v            v             v
       Deploy       Backoff       Incident
```

The Python script should not blindly act on raw external data. Normalize, validate, decide, then act.

---

# 130. Production Design Principles

Use:

```text
Explicit conditions
Named thresholds
Guard clauses
Small decision functions
Clear states
Safe defaults
Fail-closed behavior for destructive actions
Boundary tests
Dry-run support
Meaningful logs
Deterministic outcomes
```

Avoid:

```text
Deep nesting
Magic numbers
Ambiguous boolean expressions
Implicit environment defaults
Infinite retries
Silent failures
Destructive actions on incomplete data
```

---

# 131. Final Mental Model

Remember:

```text
INPUT
  |
  v
Normalize
  |
  v
Validate
  |
  v
if / elif / else
  |
  +---- invalid ----> fail safely
  |
  +---- warning ----> report / continue according to policy
  |
  +---- healthy ----> continue
  |
  +---- critical ---> remediate / rollback / escalate
```

Conditional statements are the decision-making layer of DevOps automation.

The key is not simply knowing Python syntax.

The real DevOps skill is translating an operational requirement into:

```text
Clear condition
+
Correct boundary
+
Safe action
+
Observable result
+
Testable behavior
```