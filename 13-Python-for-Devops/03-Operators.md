# Operators

> Deep operator notes for DevOps engineers. Focus: automation logic, health checks, thresholds, retries, CI/CD gates, AWS/Kubernetes conditions, validation, troubleshooting, production patterns, and interviews.

---

# 1. What Are Operators?

Operators perform operations on values and variables.

Example:

```python
cpu = 85

if cpu > 80:
    print("High CPU")
```

Here:

```text
cpu  -> operand
>    -> operator
80   -> operand
```

Operators are fundamental to DevOps automation because scripts constantly make decisions such as:

```text
Is CPU above threshold?
Are all replicas ready?
Did the deployment succeed?
Is the response code 2xx?
Should we retry?
Has the backup expired?
Is disk usage critical?
```

---

# 2. Main Operator Categories

Python operators can be grouped into:

```text
Arithmetic
Comparison
Logical
Assignment
Bitwise
Membership
Identity
```

There are also useful operators for:

```text
Conditional expressions
Dictionary access
Function calls
Object creation
```

The most important for everyday DevOps automation are:

```text
Arithmetic
Comparison
Logical
Assignment
Membership
Identity
```

---

# 3. Arithmetic Operators

Main arithmetic operators:

```text
+     Addition
-     Subtraction
*     Multiplication
/     Division
//    Floor division
%     Modulo
**    Exponentiation
```

Example:

```python
total = 10
failed = 2

successful = total - failed
```

---

# 4. Addition

```python
a = 10
b = 5

result = a + b
```

Result:

```text
15
```

For strings:

```python
first = "Dev"
second = "Ops"

result = first + second
```

Result:

```text
DevOps
```

Be careful not to mix incompatible types.

---

# 5. Subtraction

```python
total_replicas = 5
ready_replicas = 3

unready = total_replicas - ready_replicas
```

This is useful for deployment health:

```text
Desired replicas = 5
Ready replicas   = 3
Unready replicas = 2
```

---

# 6. Multiplication

```python
servers = 3
cores_per_server = 4

total_cores = servers * cores_per_server
```

Can also repeat strings:

```python
print("-" * 40)
```

Useful for reports and separators.

---

# 7. Division

```python
failed = 2
total = 100

failure_ratio = failed / total
```

Result:

```text
0.02
```

Percentage:

```python
failure_percentage = (failed / total) * 100
```

Result:

```text
2.0
```

Always protect against division by zero.

---

# 8. Division by Zero

This fails:

```python
total = 0
failed = 0

percentage = failed / total
```

Raises:

```text
ZeroDivisionError
```

Production validation:

```python
if total == 0:
    raise ValueError("Cannot calculate percentage with zero total")

percentage = failed / total * 100
```

---

# 9. Floor Division

```python
result = 10 // 3
```

Result:

```text
3
```

It returns the floor of the division result.

Useful for:

```text
Batch calculations
Chunking work
Capacity calculations
```

Be careful with negative numbers because floor division rounds toward negative infinity.

---

# 10. Modulo Operator

```python
result = 10 % 3
```

Result:

```text
1
```

Modulo gives the remainder.

DevOps uses:

```text
Batch processing
Periodic jobs
Partitioning work
Retry counters
Scheduling logic
```

Example:

```python
if retry_count % 3 == 0:
    print("Perform extended health check")
```

---

# 11. Exponentiation

```python
result = 2 ** 3
```

Result:

```text
8
```

Useful in mathematical calculations, but not common in normal infrastructure automation.

---

# 12. Arithmetic With Monitoring Data

Example:

```python
total_requests = 10000
failed_requests = 120

error_rate = (
    failed_requests / total_requests
) * 100

print(f"Error rate: {error_rate:.2f}%")
```

Result:

```text
Error rate: 1.20%
```

This kind of calculation can feed health checks and deployment gates.

---

# 13. Comparison Operators

Comparison operators return booleans.

```text
==    equal
!=    not equal
>     greater than
<     less than
>=    greater than or equal
<=    less than or equal
```

Example:

```python
cpu = 85

print(cpu > 80)
```

Result:

```text
True
```

---

# 14. Equal To

```python
status = "healthy"

if status == "healthy":
    print("Service is healthy")
```

`==` compares values.

Do not confuse it with assignment:

```python
=
```

---

# 15. Assignment vs Comparison

Assignment:

```python
status = "healthy"
```

Comparison:

```python
status == "healthy"
```

Common mistake:

```python
if status = "healthy":
```

This is invalid Python syntax.

---

# 16. Not Equal

```python
status = "failed"

if status != "healthy":
    print("Investigate service")
```

Useful for:

```text
Non-200 responses
Unexpected deployment states
Incorrect environments
Unexpected Kubernetes phases
```

---

# 17. Greater Than and Less Than

```python
cpu = 85

if cpu > 80:
    print("High CPU")
```

Another example:

```python
latency_ms = 450

if latency_ms > 300:
    print("Latency above threshold")
```

---

# 18. Greater Than or Equal

```python
disk_usage = 90

if disk_usage >= 90:
    print("Critical disk usage")
```

The difference between:

```text
>
>=
```

matters when thresholds are exact.

---

# 19. Comparison Chaining

Python supports:

```python
80 <= cpu < 90
```

Equivalent concept:

```python
cpu >= 80 and cpu < 90
```

Example:

```python
if 80 <= cpu < 90:
    print("WARNING")
```

This is readable for numeric ranges.

---

# 20. Logical Operators

Main logical operators:

```text
and
or
not
```

They combine conditions.

Example:

```python
if cpu > 80 and memory > 80:
    print("Resource pressure")
```

---

# 21. and

Both conditions must be true:

```python
if deployment_ready and health_check_passed:
    print("Deployment can proceed")
```

Truth concept:

```text
True  and True  -> True
True  and False -> False
False and True  -> False
False and False -> False
```

---

# 22. or

At least one condition must be true:

```python
if cpu > 90 or memory > 90:
    print("Critical resource pressure")
```

Truth concept:

```text
True  or True  -> True
True  or False -> True
False or True  -> True
False or False -> False
```

---

# 23. not

`not` reverses a boolean:

```python
healthy = False

if not healthy:
    print("Service is unhealthy")
```

Common pattern:

```python
if not deployment_ready:
    ...
```

---

# 24. Combining Conditions

Example:

```python
if (
    environment == "production"
    and error_rate > 5
    and latency_ms > 500
):
    print("Production degradation detected")
```

This is much clearer than deeply nested `if` statements.

---

# 25. Complex DevOps Condition

Example:

```python
if (
    environment == "production"
    and deployment_status == "success"
    and ready_replicas == desired_replicas
    and error_rate < 1
):
    print("Deployment passed")
```

This resembles a real post-deployment validation gate.

---

# 26. Operator Precedence

Python evaluates operators according to precedence.

Example:

```python
result = 2 + 3 * 4
```

Multiplication happens first:

```text
2 + 12
= 14
```

Use parentheses when the intended logic matters:

```python
result = (2 + 3) * 4
```

Result:

```text
20
```

---

# 27. Practical Precedence Order

A simplified useful order is:

```text
()
**
unary + -
* / // %
+ -
comparisons
not
and
or
```

Assignment happens later.

When writing production conditions, use parentheses if they improve clarity.

---

# 28. Why Parentheses Matter in Production

Avoid ambiguous logic:

```python
if environment == "prod" and cpu > 80 or memory > 80:
    ...
```

A reader may misunderstand the intended logic.

Prefer:

```python
if (
    environment == "prod"
    and (cpu > 80 or memory > 80)
):
    ...
```

The second version makes the business rule explicit.

---

# 29. Short-Circuit Evaluation

Python may stop evaluating conditions early.

For `and`:

```python
if value is not None and value > 80:
    ...
```

If:

```python
value is not None
```

is false, Python does not evaluate:

```python
value > 80
```

This prevents some errors.

---

# 30. Short-Circuit With or

```python
value = None

result = value or "unknown"
```

Result:

```text
unknown
```

Useful for defaults, but be careful because all falsy values trigger the fallback:

```text
0
False
""
[]
None
```

---

# 31. `and` and `or` Return Values

Python's logical operators do not necessarily return `True` or `False`.

Example:

```python
value = None

result = value or "default"
```

Result:

```text
"default"
```

This is useful, but can be confusing if you expect a boolean.

---

# 32. Assignment Operators

Basic:

```python
x = 10
```

Compound operators:

```text
+=
-=
*=
/=
//=
%=
**=
```

Example:

```python
retry_count = 0

retry_count += 1
```

Now:

```text
1
```

---

# 33. Retry Counter

Common DevOps pattern:

```python
retry_count = 0
max_retries = 3

while retry_count < max_retries:
    retry_count += 1
    print(f"Attempt {retry_count}")
```

Be careful to define exactly what `max_retries` means.

For example:

```text
Initial attempt + 3 retries
```

is different from:

```text
3 total attempts
```

---

# 34. Accumulating Results

```python
total_errors = 0

for service in services:
    total_errors += service["errors"]
```

This is useful for:

```text
Error totals
Request counts
Resource counts
Backup sizes
```

---

# 35. Decrementing Counters

```python
remaining = 5

remaining -= 1
```

Useful for:

```text
Retry budgets
Remaining resources
Queue processing
Countdown logic
```

---

# 36. Bitwise Operators

Bitwise operators work at the binary level:

```text
&
|
^
~
<<
>>
```

They are less common in everyday DevOps scripting but are useful to understand.

Example:

```python
a = 5
b = 3

print(a & b)
```

---

# 37. Bitwise AND

Binary:

```text
5 = 101
3 = 011

101
011
---
001
```

Result:

```text
1
```

Bitwise operations can be useful in low-level flags and permission-like numeric representations.

---

# 38. Bitwise OR

```python
5 | 3
```

Binary:

```text
101
011
---
111
```

Result:

```text
7
```

Used in some systems that combine feature or permission flags.

---

# 39. Bitwise XOR

```python
5 ^ 3
```

Binary:

```text
101
011
---
110
```

Result:

```text
6
```

XOR is important in computer science but relatively uncommon in standard DevOps automation.

---

# 40. Bitwise NOT

```python
~5
```

Python integers use signed integer representation semantics, so the result may surprise beginners:

```text
~5 == -6
```

Do not confuse bitwise NOT with logical `not`.

---

# 41. Left and Right Shift

```python
2 << 2
```

Result:

```text
8
```

And:

```python
8 >> 2
```

Result:

```text
2
```

These are mainly useful for binary-level operations.

---

# 42. Membership Operators

Membership:

```text
in
not in
```

Example:

```python
environment = "production"

if environment in {"dev", "stage", "production"}:
    print("Known environment")
```

This is very useful for configuration validation.

---

# 43. Membership in Lists

```python
services = [
    "user",
    "payment",
    "order"
]

if "payment" in services:
    print("Payment service exists")
```

For frequent membership checks, a set can be more appropriate:

```python
services = {"user", "payment", "order"}
```

---

# 44. Membership in Strings

```python
log_line = "ERROR database connection failed"

if "ERROR" in log_line:
    print("Error detected")
```

For simple text checks this is often better than unnecessary regex.

---

# 45. Membership in Dictionaries

For dictionaries:

```python
config = {
    "environment": "production",
    "replicas": 3
}
```

This checks keys:

```python
if "replicas" in config:
    print("Replica configuration exists")
```

It does not check values.

---

# 46. Checking Dictionary Values

```python
if "production" in config.values():
    print("Production configuration found")
```

For key/value logic, be explicit about which collection you are checking.

---

# 47. Identity Operators

Identity operators:

```text
is
is not
```

Example:

```python
value = None

if value is None:
    print("No value")
```

Identity asks:

```text
Are these the same object?
```

Value equality asks:

```text
Do these objects have equal values?
```

---

# 48. `is` vs `==` Production Rule

Use:

```python
value is None
```

not:

```python
value == None
```

For ordinary values:

```python
status == "running"
```

Use `==`.

Do not use:

```python
status is "running"
```

for string comparison.

---

# 49. Conditional Expression

Python supports a compact conditional expression:

```python
status = "healthy" if health_check else "unhealthy"
```

Equivalent:

```python
if health_check:
    status = "healthy"
else:
    status = "unhealthy"
```

Use it for simple expressions only.

---

# 50. Conditional Expression in Reporting

```python
result = "PASS" if error_rate < 1 else "FAIL"
```

Useful in concise reports.

Avoid deeply nested conditional expressions because they become difficult to maintain.

---

# 51. Arithmetic Operators in Monitoring

Example:

```python
total_requests = 10000
failed_requests = 150

error_rate = failed_requests / total_requests * 100
```

Then:

```python
if error_rate >= 5:
    print("Critical")
elif error_rate >= 1:
    print("Warning")
else:
    print("Healthy")
```

This combines arithmetic and comparison operators.

---

# 52. CPU Threshold Logic

```python
CPU_WARNING = 80
CPU_CRITICAL = 90

cpu = 87

if cpu >= CPU_CRITICAL:
    status = "CRITICAL"
elif cpu >= CPU_WARNING:
    status = "WARNING"
else:
    status = "NORMAL"
```

This pattern is common in monitoring scripts.

---

# 53. Disk Threshold Logic

```python
DISK_WARNING = 80
DISK_CRITICAL = 90

disk = 92

if disk >= DISK_CRITICAL:
    print("CRITICAL: Disk usage is high")
elif disk >= DISK_WARNING:
    print("WARNING: Disk usage is elevated")
else:
    print("Disk usage is normal")
```

---

# 54. Memory Threshold Logic

```python
memory = 76

if memory >= 90:
    print("CRITICAL")
elif memory >= 80:
    print("WARNING")
else:
    print("NORMAL")
```

Avoid copying threshold logic throughout many scripts. Centralize configuration where appropriate.

---

# 55. Deployment Gate

Example:

```python
if (
    deployment_status == "success"
    and desired_replicas == ready_replicas
    and error_rate < 1
):
    print("Deployment approved")
else:
    print("Deployment rejected")
```

This is a realistic CI/CD use of comparison and logical operators.

---

# 56. Health Check Logic

```python
http_status = 200
latency_ms = 120

healthy = (
    200 <= http_status < 300
    and latency_ms < 500
)

if healthy:
    print("Healthy")
else:
    print("Unhealthy")
```

Notice the use of:

```text
range comparison
and
```

---

# 57. Kubernetes Replica Validation

```python
desired = 5
ready = 5

if ready == desired:
    print("All replicas ready")
else:
    print(
        f"Replica mismatch: "
        f"desired={desired}, ready={ready}"
    )
```

More production-safe logic can also check:

```text
Available replicas
Updated replicas
Unavailable replicas
Pod readiness
CrashLoopBackOff
```

---

# 58. Kubernetes Pod State Validation

```python
phase = "Running"
ready = True

if phase == "Running" and ready:
    print("Pod healthy")
else:
    print("Pod requires investigation")
```

Do not assume `Running` alone means the application is healthy. Readiness is also important.

---

# 59. AWS Resource Filtering

```python
environment = "production"
state = "running"

if environment == "production" and state == "running":
    print("Production running resource")
```

This kind of condition is useful for AWS inventory and reporting.

---

# 60. AWS Cleanup Safety

Before deleting a resource:

```python
if (
    environment == "development"
    and state == "stopped"
    and age_days > 30
    and not protected
):
    print("Eligible for cleanup")
```

This demonstrates why multiple operators matter for safe automation.

---

# 61. CI/CD Quality Gate

Example:

```python
tests_passed = True
security_scan_passed = True
deployment_config_valid = True

if (
    tests_passed
    and security_scan_passed
    and deployment_config_valid
):
    print("Pipeline can continue")
else:
    print("Pipeline must stop")
```

This is a common DevSecOps pattern.

---

# 62. Retry Logic With Operators

```python
attempt = 0
max_attempts = 3

while attempt < max_attempts:
    attempt += 1

    print(f"Attempt {attempt}")

    if operation_succeeded:
        break
```

Important distinction:

```text
attempt < max_attempts
```

controls loop execution.

The success condition controls whether the loop exits early.

---

# 63. Exponential Backoff Calculation

A basic formula:

```python
delay = base_delay * (2 ** retry_count)
```

Example:

```python
base_delay = 2
retry_count = 3

delay = base_delay * (2 ** retry_count)
```

Result:

```text
16 seconds
```

In production, add a maximum delay and usually jitter to avoid synchronized retries.

---

# 64. Retry With Maximum Delay

```python
base_delay = 2
max_delay = 30

delay = min(
    base_delay * (2 ** retry_count),
    max_delay
)
```

Operators involved:

```text
**
*
min()
```

This prevents unbounded waiting.

---

# 65. Error Budget Calculation

Example:

```python
total_requests = 1_000_000
allowed_error_rate = 0.001

error_budget = total_requests * allowed_error_rate

print(error_budget)
```

Result:

```text
1000
```

This connects Python arithmetic to SLI/SLO concepts.

---

# 66. Availability Calculation

```python
successful_requests = 999500
total_requests = 1_000_000

availability = (
    successful_requests / total_requests
) * 100

print(f"{availability:.3f}%")
```

This can be used for reporting SLO compliance.

---

# 67. Safe Division Helper

Reusable function:

```python
def percentage(part, total):
    if total == 0:
        return 0.0

    return (part / total) * 100
```

Then:

```python
error_rate = percentage(
    failed_requests,
    total_requests
)
```

This avoids repeating zero-division protection.

---

# 68. Comparing Version Numbers — Important Warning

Do not compare versions as ordinary decimal numbers:

```python
if version > 1.9:
    ...
```

Version strings such as:

```text
1.10
1.9
```

do not behave like simple decimal versions.

Use a version-aware parser when version comparison matters.

---

# 69. String Comparison

String comparisons are lexicographic:

```python
"a" < "b"
```

is true.

Do not rely on string comparison for semantic versioning or numerical values that arrive as strings.

Convert/parse them first.

---

# 70. Comparing Numeric Strings

Bad:

```python
cpu = "100"

if cpu > "80":
    ...
```

String comparison is not numeric comparison.

Correct:

```python
cpu = int(cpu)

if cpu > 80:
    ...
```

This is a common environment-variable bug.

---

# 71. Chained Conditions for Ranges

Readable:

```python
if 80 <= cpu < 90:
    print("WARNING")
```

For HTTP:

```python
if 200 <= status_code < 300:
    print("Success")
```

For ports:

```python
if 1024 <= port <= 65535:
    print("Valid non-privileged port")
```

---

# 72. `not in`

Example:

```python
environment = "test"

allowed = {"dev", "stage", "prod"}

if environment not in allowed:
    raise ValueError("Unsupported environment")
```

This is useful for deployment safety.

---

# 73. Guard Clauses

Instead of deeply nested conditions:

```python
if config_valid:
    if environment_valid:
        if credentials_valid:
            deploy()
```

Prefer:

```python
if not config_valid:
    raise ValueError("Invalid configuration")

if not environment_valid:
    raise ValueError("Invalid environment")

if not credentials_valid:
    raise ValueError("Invalid credentials")

deploy()
```

Guard clauses make failure paths clearer.

---

# 74. Operator-Based Validation Function

```python
def validate_deployment(
    environment,
    replicas,
    ready_replicas,
    error_rate
):
    if environment not in {"dev", "stage", "prod"}:
        return False

    if replicas < 1:
        return False

    if ready_replicas != replicas:
        return False

    if not 0 <= error_rate <= 100:
        return False

    return True
```

Production code may return richer validation errors instead of only `True/False`.

---

# 75. Production Example — Monitoring Gate

```python
CPU_CRITICAL = 90
MEMORY_CRITICAL = 90
DISK_CRITICAL = 90

if (
    cpu >= CPU_CRITICAL
    or memory >= MEMORY_CRITICAL
    or disk >= DISK_CRITICAL
):
    status = "CRITICAL"
else:
    status = "NORMAL"
```

This expresses:

```text
Any critical resource -> critical state
```

---

# 76. Production Example — Multi-Signal Health

```python
healthy = (
    http_status >= 200
    and http_status < 300
    and latency_ms < 500
    and error_rate < 1
    and ready_replicas == desired_replicas
)
```

This combines:

```text
HTTP health
Latency
Error rate
Kubernetes readiness
```

into one deployment decision.

---

# 77. Production Example — Incident Trigger

```python
incident = (
    environment == "production"
    and error_rate >= 5
    and request_count > 100
)
```

The request-count condition can prevent a tiny sample from generating a misleading incident.

Real alert design should also consider duration, baselines, burn rate, and alert noise.

---

# 78. Production Example — Backup Validation

```python
backup_valid = (
    backup_exists
    and backup_size > 0
    and checksum_valid
    and age_hours < 24
)
```

Then:

```python
if backup_valid:
    print("Backup validation passed")
else:
    print("Backup validation failed")
```

---

# 79. Production Example — Certificate Expiry

```python
days_remaining = 18

if days_remaining <= 7:
    status = "CRITICAL"
elif days_remaining <= 30:
    status = "WARNING"
else:
    status = "NORMAL"
```

This type of threshold logic can be automated with Python.

---

# 80. Production Example — Log Classification

```python
line = "HTTP 503 from payment service"

if "503" in line or "ERROR" in line:
    print("Potential service failure")
```

For complicated log formats, use structured parsing rather than increasingly complex string conditions.

---

# 81. Operator Precedence in Alert Logic

Potentially confusing:

```python
if error_rate > 5 and cpu > 90 or memory > 90:
    ...
```

Python interprets `and` before `or`.

If the intended rule is:

```text
(error rate > 5 AND CPU > 90)
OR memory > 90
```

write:

```python
if (
    (error_rate > 5 and cpu > 90)
    or memory > 90
):
    ...
```

Always make important alert logic explicit.

---

# 82. Avoid Overly Complex Conditions

Bad:

```python
if a and b or c and not d or e and f:
    ...
```

Better:

```python
resource_critical = cpu > 90 or memory > 90
deployment_safe = tests_passed and security_passed
production_gate = environment == "production"

if production_gate and resource_critical and not deployment_safe:
    ...
```

Named intermediate conditions improve incident debugging.

---

# 83. Operators and Readability

Production automation should prioritize:

```text
Correctness
Readability
Predictability
Testability
```

Do not write clever one-liners when the condition represents a business or operational rule.

---

# 84. Operator Testing

For threshold logic, test boundary values.

If:

```python
CRITICAL = 90
```

test:

```text
89.99
90
90.01
```

Similarly, if:

```python
WARNING = 80
```

test:

```text
79.99
80
80.01
```

Boundary testing catches many monitoring bugs.

---

# 85. Unit Tests for Operators

Example:

```python
def classify_cpu(cpu):
    if cpu >= 90:
        return "CRITICAL"
    if cpu >= 80:
        return "WARNING"
    return "NORMAL"
```

Tests should include:

```python
assert classify_cpu(50) == "NORMAL"
assert classify_cpu(80) == "WARNING"
assert classify_cpu(89.99) == "WARNING"
assert classify_cpu(90) == "CRITICAL"
```

---

# 86. Operator Errors in CI/CD

A wrong operator can cause serious consequences.

Example:

```python
if replicas > desired_replicas:
    print("Deployment healthy")
```

This is wrong if health requires equality.

Correct:

```python
if replicas == desired_replicas:
    print("Deployment healthy")
```

Always define the operational requirement before writing the condition.

---

# 87. Operator Errors in Cleanup Automation

Dangerous:

```python
if age_days > 30:
    delete(resource)
```

This may delete resources from every environment.

Safer:

```python
if (
    environment == "development"
    and age_days > 30
    and not protected
):
    delete(resource)
```

Destructive automation should use multiple safety conditions.

---

# 88. Operator Errors in Alerting

Bad:

```python
if cpu > 80:
    alert()
```

This may alert continuously.

Production alerting often needs:

```text
Threshold
+
Duration
+
Relevant scope
+
Meaningful impact
```

Example concept:

```text
CPU > 80%
for 10 minutes
on production workload
```

The exact alert rule belongs in the monitoring system, but Python automation must understand the same logic.

---

# 89. Operators and SLO Calculations

Example:

```python
availability = successful / total * 100

if availability >= 99.9:
    print("SLO met")
else:
    print("SLO violated")
```

The boundary is important:

```text
99.9 >= 99.9 -> True
```

This is why comparison operators must match the SLO definition exactly.

---

# 90. Operators and Error Budgets

```python
budget_remaining = allowed_errors - actual_errors

if budget_remaining > 0:
    print("Error budget remains")
elif budget_remaining == 0:
    print("Error budget exhausted")
else:
    print("Error budget exceeded")
```

This connects Python logic directly to production reliability practices.

---

# 91. Operators and HTTP Status Codes

```python
if 200 <= status_code < 300:
    print("Success")
elif 400 <= status_code < 500:
    print("Client error")
elif 500 <= status_code < 600:
    print("Server error")
else:
    print("Unexpected status")
```

This is much better than checking every individual status code.

---

# 92. Operators and Kubernetes Exit Logic

A deployment validator can use:

```python
if ready_replicas != desired_replicas:
    raise SystemExit(1)

if error_rate >= 5:
    raise SystemExit(1)

raise SystemExit(0)
```

CI/CD can then interpret:

```text
0 -> success
non-zero -> failure
```

---

# 93. Operators and AWS Cleanup

Safe eligibility:

```python
eligible = (
    environment == "dev"
    and state == "stopped"
    and age_days >= 30
    and not production_protected
)
```

Then:

```python
if eligible:
    print("Eligible for cleanup")
```

A production cleanup tool should still support dry-run, logging, and explicit confirmation/safety controls.

---

# 94. Operators and Retry Backoff

Concept:

```python
delay = min(
    base_delay * (2 ** retry_count),
    max_delay
)
```

Production retry design should additionally consider:

```text
Maximum attempts
Maximum delay
Jitter
Retryable errors
Non-retryable errors
Overall timeout
```

Do not retry permanent failures indefinitely.

---

# 95. Operators and Rate Limiting

A simple conceptual calculation:

```python
requests_per_second = requests / elapsed_seconds
```

Then:

```python
if requests_per_second > limit:
    print("Rate limit exceeded")
```

For real distributed systems, rate limiting often needs a dedicated algorithm or library rather than a single local counter.

---

# 96. Operators and Capacity Planning

Example:

```python
current_capacity = 100
current_usage = 85

utilization = (
    current_usage / current_capacity
) * 100

if utilization >= 80:
    print("Capacity planning review required")
```

The same arithmetic and comparison operators appear in cloud operations.

---

# 97. Operators and Cost Controls

Example:

```python
monthly_cost = 1250
budget = 1000

if monthly_cost > budget:
    print("Budget exceeded")
```

Production cost automation can combine:

```text
Environment
Owner
Resource state
Age
Cost
Protection
```

before taking any action.

---

# 98. Operators and Security Validation

Example:

```python
port = 22
allowed_ports = {22, 80, 443}

if port not in allowed_ports:
    print("Unexpected port")
```

Another example:

```python
if environment == "production" and destructive_action:
    require_confirmation = True
```

Operators can implement security guardrails, but authorization should still be enforced by the underlying system.

---

# 99. Operators and File Cleanup

```python
if (
    file_age_days > retention_days
    and not important_file
):
    delete_file()
```

The safety condition should identify the correct file scope before deletion.

---

# 100. Operators and Health Check Exit Codes

```python
healthy = (
    status_code >= 200
    and status_code < 300
    and latency_ms < 500
)

if healthy:
    raise SystemExit(0)

raise SystemExit(1)
```

This allows a Python health check to act as a CI/CD gate or Kubernetes-style external validation.

---

# 101. Common Mistakes

Avoid:

```text
Using = instead of ==
Comparing numeric strings
Using is for string comparison
Forgetting zero-division protection
Ignoring operator precedence
Overly complex boolean expressions
Wrong threshold boundaries
Silent defaults
Incorrect retry conditions
Unsafe destructive conditions
```

---

# 102. Common Mistake — Wrong Threshold Boundary

Requirement:

```text
Critical when CPU >= 90
```

Wrong:

```python
if cpu > 90:
    status = "CRITICAL"
```

At exactly:

```text
CPU = 90
```

the alert does not trigger.

Correct:

```python
if cpu >= 90:
    status = "CRITICAL"
```

---

# 103. Common Mistake — Wrong Logical Operator

Requirement:

```text
Alert when CPU OR memory is critical
```

Wrong:

```python
if cpu >= 90 and memory >= 90:
    alert()
```

This requires both.

Correct:

```python
if cpu >= 90 or memory >= 90:
    alert()
```

---

# 104. Common Mistake — Wrong `not in`

Correct:

```python
if environment not in allowed_environments:
    raise ValueError("Invalid environment")
```

Avoid complicated negated membership expressions when a direct `not in` communicates the intent clearly.

---

# 105. Common Mistake — No Parentheses

Instead of:

```python
if prod and high_cpu or high_memory:
    ...
```

make the intended rule explicit:

```python
if prod and (high_cpu or high_memory):
    ...
```

This is especially important for production alert and cleanup logic.

---

# 106. Common Mistake — Boolean Strings

Bad:

```python
enabled = os.getenv("ENABLED")

if enabled:
    deploy()
```

If:

```text
ENABLED=false
```

the condition is still true because the string is non-empty.

Parse it explicitly.

---

# 107. Common Mistake — Infinite Retry

Dangerous:

```python
while not success:
    retry()
```

If success never happens, the process may run forever.

Use:

```python
while attempt < max_attempts:
    ...
```

and an overall timeout where appropriate.

---

# 108. Common Mistake — Counter Off-by-One

Be explicit:

```python
max_attempts = 3
attempt = 0

while attempt < max_attempts:
    attempt += 1
    ...
```

This produces exactly three attempts.

Document whether your variable represents:

```text
attempt number
retry number
remaining retries
```

---

# 109. Practical Exercise 1 — CPU Classifier

Write:

```python
classify_cpu(cpu)
```

Rules:

```text
< 80       NORMAL
80-89.99   WARNING
>= 90      CRITICAL
```

Test:

```text
79
80
89.99
90
95
```

---

# 110. Practical Exercise 2 — HTTP Health Check

Given:

```python
status_code = 200
latency_ms = 250
```

Return healthy only when:

```text
HTTP status is 2xx
AND
latency < 500 ms
```

Test:

```text
200 / 250 -> healthy
503 / 250 -> unhealthy
200 / 600 -> unhealthy
```

---

# 111. Practical Exercise 3 — Deployment Gate

Given:

```python
desired = 5
ready = 5
error_rate = 0.4
security_passed = True
```

Deployment passes only when:

```text
desired == ready
AND
error rate < 1
AND
security passed
```

Return:

```text
PASS
```

Otherwise:

```text
FAIL
```

---

# 112. Practical Exercise 4 — Cleanup Eligibility

A resource can be deleted only if:

```text
environment == dev
AND
state == stopped
AND
age >= 30 days
AND
protected == False
```

Write a boolean expression and test boundary cases.

---

# 113. Practical Exercise 5 — Retry Calculation

Given:

```python
base_delay = 2
max_delay = 30
retry_count = 0
```

Calculate:

```text
Retry 0 -> 2
Retry 1 -> 4
Retry 2 -> 8
Retry 3 -> 16
Retry 4 -> 30
Retry 5 -> 30
```

Use:

```python
min()
```

to enforce the maximum delay.

---

# 114. Practical Exercise 6 — Error Budget

Given:

```python
total_requests = 1_000_000
allowed_error_rate = 0.001
actual_errors = 1200
```

Calculate:

```text
Allowed errors
Remaining budget
Whether budget is exceeded
```

Expected:

```text
Allowed errors = 1000
Remaining budget = -200
Budget exceeded
```

---

# 115. Practical Exercise 7 — Multi-Signal Health

Given:

```python
cpu = 70
memory = 82
disk = 65
error_rate = 0.5
latency_ms = 300
```

Define:

```text
Resource healthy when CPU < 90
AND memory < 90
AND disk < 90

Application healthy when:
error rate < 1
AND latency < 500
```

Then calculate overall health.

---

# 116. Practical Exercise 8 — Environment Safety

Allowed environments:

```python
{"dev", "stage", "prod"}
```

Reject:

```text
test
production
qa
unknown
```

Then extend the logic so that destructive operations are allowed only in `dev` unless an explicit production safety flag is enabled.

---

# 117. Interview — What Are the Main Python Operators?

Answer:

```text
Arithmetic
Comparison
Logical
Assignment
Bitwise
Membership
Identity
```

For DevOps automation, the most frequently used are arithmetic, comparison, logical, assignment, membership, and identity operators.

---

# 118. Interview — Difference Between `=` and `==`

Strong answer:

> "`=` assigns a value to a variable, while `==` compares two values. I use assignment when setting configuration or state and comparison when implementing conditions."

---

# 119. Interview — Difference Between `==` and `is`

Strong answer:

> "`==` checks value equality, while `is` checks object identity. In normal DevOps code I use `is None` for checking `None`, while strings, numbers, and statuses should normally be compared with `==`."

---

# 120. Interview — Difference Between `and` and `or`

Strong answer:

> "`and` requires all relevant conditions to be true, while `or` requires at least one condition to be true. For example, a critical resource alert might use CPU OR memory above a critical threshold."

---

# 121. Interview — What Is Short-Circuit Evaluation?

Strong answer:

> Python can stop evaluating a logical expression as soon as the final result is known. With `and`, a false condition can stop further evaluation. With `or`, a true condition can stop further evaluation. This can improve efficiency and can also prevent unsafe evaluation of later expressions.

Example:

```python
if value is not None and value > 80:
    ...
```

---

# 122. Interview — How Do You Design Threshold Logic?

Strong answer:

> "I first define the exact operational requirement and boundary conditions. Then I use named constants, explicit comparisons, and tests for boundary values. For important production logic I also avoid ambiguous operator precedence and use parentheses or named intermediate conditions."

---

# 123. Interview — How Do You Avoid Retry Problems?

Strong answer:

> "I define the maximum attempts, retryable errors, timeout, and backoff strategy. For distributed systems I generally use exponential backoff with jitter and a maximum delay. I don't blindly retry permanent failures such as invalid credentials or malformed requests."

---

# 124. Interview — How Would You Build a Deployment Gate?

Strong answer:

```text
Collect deployment state
        |
        v
Validate desired vs ready replicas
        |
        v
Check application health
        |
        v
Check error rate/latency
        |
        v
Check security/test results
        |
        v
PASS -> exit 0
FAIL -> exit non-zero
```

The conditions should be explicit and tested at their boundaries.

---

# 125. Interview — How Do Operators Help in Monitoring?

Strong answer:

> "Operators allow automation to calculate rates and percentages, compare metrics against thresholds, combine multiple signals, validate health conditions, calculate error budgets, and decide whether a deployment or operational action should proceed."

---

# 126. Interview — How Do You Prevent Unsafe Automation?

Strong answer:

```text
Use explicit conditions
Validate environment
Use allowlists
Check resource ownership
Use dry-run
Use confirmation for destructive actions
Use least privilege
Log the decision
Test boundary conditions
Limit retries
```

---

# 127. Production Design Pattern

A mature operator-driven automation workflow looks like:

```text
Input
  |
  v
Normalize types
  |
  v
Validate values
  |
  v
Calculate derived values
  |
  v
Evaluate conditions
  |
  v
Apply safety conditions
  |
  v
Perform action
  |
  v
Log decision
  |
  v
Return result/exit code
```

This is the pattern to remember for DevOps Python.

---

# 128. Final Takeaway

Operators look simple, but production automation depends heavily on getting them exactly right.

Remember:

```text
=      assignment
==     value comparison
is     object identity
> >=   upper thresholds
< <=   lower thresholds
and    all required conditions
or     any acceptable condition
not    inverse condition
in     membership
not in inverse membership
+=     increment
-=     decrement
%      remainder
**     exponentiation
```

For DevOps, operators become powerful when combined with:

```text
Monitoring
CI/CD
AWS
Kubernetes
Health checks
SLOs
Error budgets
Retries
Cleanup automation
Security guardrails
```

The most important production habit is:

> Define the operational rule first, then translate it into clear, testable Python conditions.

---

# 129. Next File

```text
22-Python-for-DevOps/
└── 01-Python-Fundamentals/
    ├── 01-Python-Introduction.md
    ├── 02-Variables-and-Data-Types.md
    └── 03-Operators.md
```

Next topic:

```text
04-Conditional-Statements.md
```

It will cover:

```text
if
elif
else
nested conditions
guard clauses
condition composition
truthiness
validation flows
DevOps decision logic
deployment gates
health checks
AWS/Kubernetes examples
common mistakes
production patterns
interview questions
scenario-based questions
practical exercises
```
