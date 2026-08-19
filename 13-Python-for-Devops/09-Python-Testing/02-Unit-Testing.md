# Unit Testing

## 1. Overview

Unit testing is the next layer after pytest fundamentals.

The goal is to test small pieces of Python code in isolation:

```text
Function
Class
Validator
Parser
Policy
Transformer
Calculator
```

For DevOps automation, unit testing is especially important because production automation often contains decision-making logic such as:

```text
Should I deploy?
Should I retry?
Should I rollback?
Which environment?
Which namespace?
Which image?
Which AWS account?
Is the deployment healthy?
Should the pipeline stop?
```

A good unit-test suite validates these decisions before they interact with real:

```text
AWS
Kubernetes
GitHub
Jenkins
ArgoCD
Prometheus
ELK
```

---

# 2. Unit Testing Mental Model

Think:

```text
Input
  ↓
Function
  ↓
Output
```

Example:

```text
environment = "prod"
        ↓
validate_environment()
        ↓
True
```

The unit test verifies the behavior.

---

# 3. Unit Test Characteristics

A good unit test should generally be:

```text
Fast
Deterministic
Isolated
Readable
Repeatable
Focused
```

It should avoid unnecessary:

```text
Network calls
Real AWS calls
Real Kubernetes calls
Production databases
External services
Long sleeps
Shared mutable state
```

---

# 4. Unit Test vs Integration Test

Unit:

```text
Function
  ↓
Controlled inputs
  ↓
Expected result
```

Integration:

```text
Python
  ↓
API
  ↓
AWS/Kubernetes/etc.
  ↓
Real behavior
```

Example:

```text
Unit:
parse ArgoCD response

Integration:
call real ArgoCD API and parse response
```

---

# 5. What Should Be Unit Tested?

For a DevOps project:

```text
Configuration validation
Environment mapping
Namespace mapping
Image naming
Payload generation
Response parsing
Error classification
Retry policy
Timeout policy
Rollback decision
Health calculation
Release policy
Security policy
```

These are high-value targets.

---

# 6. First Unit Test

Application:

```python
def add(a, b):
    return a + b
```

Test:

```python
def test_add():

    result = add(2, 3)

    assert result == 5
```

The test has three conceptual stages:

```text
Arrange
Act
Assert
```

---

# 7. Arrange-Act-Assert

```python
def test_add():

    # Arrange
    a = 2
    b = 3

    # Act
    result = add(a, b)

    # Assert
    assert result == 5
```

This structure makes tests easy to understand.

---

# 8. Given-When-Then

The same test can be described as:

```text
Given:
a = 2 and b = 3

When:
add() is called

Then:
result should be 5
```

This style is useful when discussing behavior with developers, testers, and interviewers.

---

# 9. Test One Behavior

Good:

```python
def test_invalid_environment_is_rejected():
    ...
```

Avoid making one test verify:

```text
validation
API
deployment
logging
rollback
metrics
```

all at once.

That creates difficult failures.

---

# 10. Focused Tests

Prefer:

```text
test_valid_environment
test_invalid_environment
test_missing_environment
test_prod_requires_approval
```

rather than:

```text
test_everything
```

---

# 11. Testing Pure Functions

Pure functions are easiest to unit test.

Example:

```python
def normalize_service_name(name):

    return name.lower().replace(
        "_",
        "-"
    )
```

Tests:

```python
def test_normalize_service_name():

    assert normalize_service_name(
        "Payment_Service"
    ) == "payment-service"
```

No external dependency is required.

---

# 12. Test Multiple Inputs

```python
import pytest


@pytest.mark.parametrize(
    "value,expected",
    [
        ("Payment", "payment"),
        ("PAYMENT", "payment"),
        ("payment_service", "payment-service")
    ]
)
def test_normalize_service_name(
    value,
    expected
):

    assert normalize_service_name(
        value
    ) == expected
```

---

# 13. Positive Testing

Positive testing validates valid input.

Example:

```python
def test_valid_version():

    assert validate_version(
        "1.4.2"
    )
```

---

# 14. Negative Testing

Negative testing validates invalid input.

Example:

```python
def test_invalid_version():

    with pytest.raises(ValueError):
        validate_version(
            "invalid"
        )
```

DevOps automation needs strong negative testing because invalid input can cause infrastructure damage.

---

# 15. Boundary Testing

Suppose:

```text
max replicas = 10
```

Test:

```text
9
10
11
```

Why?

Because bugs often occur around:

```text
< 
<=
>
>=
```

boundaries.

---

# 16. Boundary Example

```python
def replicas_valid(count):

    return 1 <= count <= 10
```

Test:

```python
@pytest.mark.parametrize(
    "count,expected",
    [
        (0, False),
        (1, True),
        (10, True),
        (11, False)
    ]
)
def test_replica_limits(
    count,
    expected
):

    assert replicas_valid(
        count
    ) == expected
```

---

# 17. Testing `None`

Always consider:

```text
None
```

Example:

```python
def get_environment(config):

    return config.get("environment")
```

Test:

```python
def test_missing_environment():

    assert get_environment({}) is None
```

---

# 18. Testing Empty Values

Consider:

```text
""
[]
{}
None
```

These can have different meanings.

Example:

```python
def test_empty_services():

    assert get_services({}) == []
```

Do not assume all empty values are interchangeable.

---

# 19. Testing Type Errors

If a function expects:

```text
string
```

test:

```text
None
integer
list
dictionary
```

when those inputs are possible at the boundary.

---

# 20. Input Validation

Example:

```python
def validate_environment(
    environment
):

    allowed = {
        "dev",
        "staging",
        "prod"
    }

    if environment not in allowed:
        raise ValueError(
            "Invalid environment"
        )

    return True
```

Test valid values:

```python
@pytest.mark.parametrize(
    "environment",
    ["dev", "staging", "prod"]
)
def test_valid_environment(
    environment
):

    assert validate_environment(
        environment
    )
```

---

# 21. Invalid Input Matrix

```python
@pytest.mark.parametrize(
    "environment",
    [
        "",
        "production",
        "prod1",
        None
    ]
)
def test_invalid_environment(
    environment
):

    with pytest.raises(
        ValueError
    ):
        validate_environment(
            environment
        )
```

If `None` is expected to raise a different exception, test that contract explicitly.

---

# 22. Testing Configuration

Example:

```python
def build_config(environment):

    configs = {
        "dev": {
            "cluster": "dev-cluster"
        },
        "staging": {
            "cluster": "staging-cluster"
        },
        "prod": {
            "cluster": "prod-cluster"
        }
    }

    return configs[environment]
```

Test:

```python
def test_prod_config():

    result = build_config("prod")

    assert result["cluster"] == "prod-cluster"
```

---

# 23. Production Safety Test

A critical test:

```python
def test_staging_does_not_use_prod_cluster():

    result = build_config(
        "staging"
    )

    assert result["cluster"] != \
        "prod-cluster"
```

This is a small test with potentially large production value.

---

# 24. Environment Mapping

Example:

```text
dev     -> AWS account A
staging -> AWS account B
prod    -> AWS account C
```

Test the mapping independently.

```python
def account_for(environment):

    return {
        "dev": "111",
        "staging": "222",
        "prod": "333"
    }[environment]
```

---

# 25. Mapping Tests

```python
@pytest.mark.parametrize(
    "environment,account",
    [
        ("dev", "111"),
        ("staging", "222"),
        ("prod", "333")
    ]
)
def test_account_mapping(
    environment,
    account
):

    assert account_for(
        environment
    ) == account
```

In real projects, use non-sensitive identifiers or test account aliases rather than embedding sensitive account metadata unnecessarily.

---

# 26. Namespace Mapping

```python
def namespace_for(
    service,
    environment
):

    return f"{service}-{environment}"
```

Test:

```python
def test_namespace():

    assert namespace_for(
        "payment",
        "prod"
    ) == "payment-prod"
```

---

# 27. Image Reference Generation

```python
def image_reference(
    registry,
    repository,
    tag
):

    return (
        f"{registry}/"
        f"{repository}:{tag}"
    )
```

Test:

```python
def test_image_reference():

    assert image_reference(
        "registry.example.com",
        "payment",
        "1.4.2"
    ) == (
        "registry.example.com/"
        "payment:1.4.2"
    )
```

---

# 28. Digest-Based Verification

Prefer immutable artifact identity where possible.

Example:

```python
def digest_matches(
    expected,
    actual
):

    return expected == actual
```

Test:

```python
def test_digest_matches():

    assert digest_matches(
        "sha256:abc",
        "sha256:abc"
    )
```

And:

```python
def test_digest_mismatch():

    assert not digest_matches(
        "sha256:abc",
        "sha256:def"
    )
```

---

# 29. Why Digest Tests Matter

Tags can be mutable:

```text
payment:latest
```

Digest:

```text
payment@sha256:...
```

is immutable for a specific image content.

A deployment validator should distinguish:

```text
tag identity
```

from:

```text
content identity
```

---

# 30. Testing Kubernetes Readiness Logic

Example:

```python
def deployment_ready(
    desired,
    ready
):

    return desired == ready
```

Tests:

```python
@pytest.mark.parametrize(
    "desired,ready,expected",
    [
        (3, 3, True),
        (3, 2, False),
        (3, 0, False),
        (0, 0, True)
    ]
)
def test_deployment_ready(
    desired,
    ready,
    expected
):

    assert deployment_ready(
        desired,
        ready
    ) == expected
```

---

# 31. Kubernetes Status Parsing

Example:

```python
def parse_rollout_status(data):

    status = data.get("status", {})

    return {
        "desired": status.get(
            "replicas",
            0
        ),
        "ready": status.get(
            "readyReplicas",
            0
        )
    }
```

Test with a controlled dictionary.

---

# 32. Missing Kubernetes Fields

Test:

```python
def test_missing_status():

    result = parse_rollout_status({})

    assert result == {
        "desired": 0,
        "ready": 0
    }
```

The expected behavior should match your application contract.

---

# 33. Testing ArgoCD Policy

Example:

```python
def argocd_healthy(app):

    return (
        app["sync"] == "Synced"
        and app["health"] == "Healthy"
    )
```

Test:

```python
def test_argocd_healthy():

    app = {
        "sync": "Synced",
        "health": "Healthy"
    }

    assert argocd_healthy(app)
```

---

# 34. ArgoCD Failure Matrix

```python
@pytest.mark.parametrize(
    "sync,health,expected",
    [
        ("Synced", "Healthy", True),
        ("Synced", "Degraded", False),
        ("OutOfSync", "Healthy", False),
        ("OutOfSync", "Degraded", False)
    ]
)
def test_argocd_health(
    sync,
    health,
    expected
):

    app = {
        "sync": sync,
        "health": health
    }

    assert argocd_healthy(
        app
    ) == expected
```

---

# 35. Testing Prometheus Thresholds

Example:

```python
def error_rate_ok(
    rate,
    threshold
):

    return rate <= threshold
```

Tests:

```python
@pytest.mark.parametrize(
    "rate,expected",
    [
        (0.001, True),
        (0.010, True),
        (0.011, False)
    ]
)
def test_error_rate(
    rate,
    expected
):

    assert error_rate_ok(
        rate,
        0.010
    ) == expected
```

---

# 36. Testing Release Health

```python
def release_healthy(
    kubernetes,
    argocd,
    error_rate,
    threshold
):

    return (
        kubernetes
        and argocd
        and error_rate <= threshold
    )
```

This can be tested without any live system.

---

# 37. Policy Matrix

```text
Kubernetes | ArgoCD | Error Rate | Result
-----------+--------+------------+-------
Healthy    | Healthy| Low        | Pass
Healthy    | Bad    | Low        | Fail
Bad        | Healthy| Low        | Fail
Healthy    | Healthy| High       | Fail
```

This is a strong unit-testing target.

---

# 38. Testing Retry Policy

Suppose:

```python
def should_retry(
    status_code
):

    return status_code in {
        429,
        500,
        502,
        503,
        504
    }
```

Test:

```python
@pytest.mark.parametrize(
    "status",
    [429, 500, 502, 503, 504]
)
def test_retryable_status(
    status
):

    assert should_retry(status)
```

---

# 39. Non-Retryable Errors

```python
@pytest.mark.parametrize(
    "status",
    [400, 401, 403, 404]
)
def test_non_retryable_status(
    status
):

    assert not should_retry(status)
```

This prevents dangerous retry loops.

---

# 40. Error Classification

Example:

```python
def classify_status(status):

    if status == 401:
        return "authentication"

    if status == 403:
        return "authorization"

    if status == 429:
        return "rate_limit"

    if status >= 500:
        return "server"

    if status >= 400:
        return "client"

    return "success"
```

Test each category.

---

# 41. Why Error Classification Matters

The workflow may decide:

```text
authentication -> refresh/fail
authorization  -> fail
rate limit     -> backoff
server error   -> retry
client error   -> fail
success        -> continue
```

Unit tests protect these decisions.

---

# 42. Testing Timeout Policy

```python
def timeout_exceeded(
    elapsed,
    timeout
):

    return elapsed >= timeout
```

Test:

```python
@pytest.mark.parametrize(
    "elapsed,timeout,expected",
    [
        (5, 10, False),
        (10, 10, True),
        (11, 10, True)
    ]
)
def test_timeout(
    elapsed,
    timeout,
    expected
):

    assert timeout_exceeded(
        elapsed,
        timeout
    ) == expected
```

---

# 43. Testing Backoff Calculation

Example:

```python
def backoff_delay(
    attempt,
    base=2
):

    return base ** attempt
```

Test:

```python
@pytest.mark.parametrize(
    "attempt,expected",
    [
        (0, 1),
        (1, 2),
        (2, 4),
        (3, 8)
    ]
)
def test_backoff(
    attempt,
    expected
):

    assert backoff_delay(
        attempt
    ) == expected
```

Production retry algorithms should usually include jitter to reduce synchronized retries; the exact strategy should be tested against its documented behavior.

---

# 44. Testing Maximum Retries

```python
def retry_allowed(
    attempt,
    max_attempts
):

    return attempt < max_attempts
```

Test boundaries:

```python
@pytest.mark.parametrize(
    "attempt,expected",
    [
        (0, True),
        (1, True),
        (2, True),
        (3, False)
    ]
)
def test_retry_limit(
    attempt,
    expected
):

    assert retry_allowed(
        attempt,
        3
    ) == expected
```

---

# 45. Idempotency Testing

Example:

```python
def release_key(
    service,
    version
):

    return f"{service}:{version}"
```

Test:

```python
def test_release_key_is_stable():

    assert release_key(
        "payment",
        "1.4.2"
    ) == release_key(
        "payment",
        "1.4.2"
    )
```

Idempotency should be tested at the workflow level too.

---

# 46. Reconciliation Logic

Example:

```python
def reconcile(
    desired,
    actual
):

    if desired == actual:
        return "no-op"

    return "update"
```

Tests:

```python
def test_reconcile_same_state():

    assert reconcile(
        "1.4.2",
        "1.4.2"
    ) == "no-op"
```

and:

```python
def test_reconcile_different_state():

    assert reconcile(
        "1.4.2",
        "1.4.1"
    ) == "update"
```

---

# 47. Rollback Decision

```python
def should_rollback(
    health,
    error_rate,
    threshold
):

    return (
        health != "Healthy"
        or error_rate > threshold
    )
```

Test:

```python
def test_rollback_on_degraded():

    assert should_rollback(
        "Degraded",
        0.001,
        0.01
    )
```

---

# 48. Rollback Boundary

```python
def test_no_rollback_at_threshold():

    assert not should_rollback(
        "Healthy",
        0.01,
        0.01
    )
```

And:

```python
def test_rollback_above_threshold():

    assert should_rollback(
        "Healthy",
        0.011,
        0.01
    )
```

---

# 49. Testing Security Gates

Example:

```python
def security_gate(
    critical,
    high
):

    return critical == 0 and high <= 5
```

Test:

```python
def test_security_gate_pass():

    assert security_gate(
        critical=0,
        high=2
    )
```

Failure:

```python
def test_security_gate_critical():

    assert not security_gate(
        critical=1,
        high=0
    )
```

---

# 50. SonarQube Policy Logic

Do not test SonarQube itself in a unit test.

Test your policy:

```python
def quality_gate_passed(
    status
):

    return status == "OK"
```

Test:

```python
def test_quality_gate():

    assert quality_gate_passed("OK")
    assert not quality_gate_passed(
        "ERROR"
    )
```

---

# 51. Trivy Policy Logic

```python
def vulnerability_gate(
    critical_count
):

    return critical_count == 0
```

Test:

```python
def test_trivy_gate():

    assert vulnerability_gate(0)
    assert not vulnerability_gate(1)
```

The scanner itself belongs in integration/CI validation.

---

# 52. Veracode Policy Logic

Example:

```python
def security_approval(
    score,
    minimum
):

    return score >= minimum
```

Unit-test the policy boundaries.

---

# 53. Testing Configuration Files

If configuration is YAML/JSON:

```text
load
validate
transform
```

can be separated.

Example:

```python
def validate_config(config):

    required = {
        "environment",
        "cluster",
        "namespace"
    }

    return required <= config.keys()
```

Test missing keys.

---

# 54. Test Configuration Drift Detection

```python
def drifted(
    desired,
    actual
):

    return desired != actual
```

Test:

```python
def test_no_drift():

    assert not drifted(
        {"version": "1.4.2"},
        {"version": "1.4.2"}
    )
```

and:

```python
def test_drift():

    assert drifted(
        {"version": "1.4.2"},
        {"version": "1.4.1"}
    )
```

---

# 55. Testing GitOps Path Logic

```python
def gitops_path(
    environment,
    service
):

    return (
        f"environments/"
        f"{environment}/"
        f"{service}"
    )
```

Test:

```python
def test_gitops_path():

    assert gitops_path(
        "prod",
        "payment"
    ) == (
        "environments/"
        "prod/payment"
    )
```

---

# 56. Testing Deployment Names

```python
def deployment_name(
    service,
    environment
):

    return f"{service}-{environment}"
```

Test:

```python
def test_deployment_name():

    assert deployment_name(
        "payment",
        "prod"
    ) == "payment-prod"
```

---

# 57. Testing Resource Naming Rules

Test:

```text
lowercase
allowed characters
maximum length
environment suffix
service prefix
```

Example:

```python
def valid_name(name):

    return (
        name == name.lower()
        and " " not in name
    )
```

Test invalid values.

---

# 58. Testing Version Validation

Example:

```python
import re


def valid_version(version):

    return bool(
        re.fullmatch(
            r"\d+\.\d+\.\d+",
            version
        )
    )
```

Test:

```python
@pytest.mark.parametrize(
    "version,expected",
    [
        ("1.2.3", True),
        ("10.20.30", True),
        ("1.2", False),
        ("v1.2.3", False),
        ("latest", False)
    ]
)
def test_version(
    version,
    expected
):

    assert valid_version(
        version
    ) == expected
```

---

# 59. Testing SemVer Policies

If the project requires semantic versioning:

```text
major.minor.patch
```

test:

```text
valid version
invalid version
pre-release
build metadata
```

according to the exact versioning contract used by the project.

---

# 60. Testing Kubernetes Label Rules

Example:

```python
def build_labels(
    service,
    environment
):

    return {
        "app": service,
        "environment": environment
    }
```

Test:

```python
def test_labels():

    labels = build_labels(
        "payment",
        "prod"
    )

    assert labels["app"] == "payment"
    assert labels["environment"] == "prod"
```

---

# 61. Testing Manifest Generation

If Python generates Kubernetes manifests:

```python
def build_deployment(
    name,
    image
):

    return {
        "metadata": {
            "name": name
        },
        "spec": {
            "template": {
                "spec": {
                    "containers": [
                        {
                            "name": name,
                            "image": image
                        }
                    ]
                }
            }
        }
    }
```

Test:

```python
def test_manifest_image():

    manifest = build_deployment(
        "payment",
        "payment:1.4.2"
    )

    container = (
        manifest["spec"]
        ["template"]
        ["spec"]
        ["containers"][0]
    )

    assert container["image"] \
        == "payment:1.4.2"
```

---

# 62. Test Required Kubernetes Fields

Verify generated manifests contain:

```text
metadata.name
spec
containers
container name
image
```

according to your deployment contract.

---

# 63. Test Resource Limits

If your automation generates:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

unit-test that required resource fields exist.

This is a valuable production guardrail.

---

# 64. Test Health Probes

If manifests must contain:

```text
livenessProbe
readinessProbe
```

test the generated manifest.

Example:

```python
def test_probes_present(manifest):

    container = (
        manifest["spec"]
        ["template"]
        ["spec"]
        ["containers"][0]
    )

    assert "livenessProbe" in container
    assert "readinessProbe" in container
```

---

# 65. Test Security Context

For secure workloads, test required settings such as:

```text
runAsNonRoot
readOnlyRootFilesystem
allowPrivilegeEscalation=false
```

according to your organization's security policy.

---

# 66. Testing Docker Build Logic

If Python generates:

```text
Docker build arguments
image tags
registry paths
```

unit-test the generated command/configuration.

Do not launch Docker for every unit test.

Docker integration tests belong in a later test layer.

---

# 67. Testing Terraform Automation

If Python generates Terraform variables:

```python
def terraform_vars(
    environment
):

    return {
        "environment": environment
    }
```

Test the mapping.

Terraform execution itself belongs in:

```text
integration/E2E testing
```

---

# 68. Testing Jenkins Parameters

Example:

```python
def build_jenkins_parameters(
    service,
    version
):

    return {
        "SERVICE": service,
        "VERSION": version
    }
```

Test:

```python
def test_jenkins_parameters():

    params = build_jenkins_parameters(
        "payment",
        "1.4.2"
    )

    assert params["SERVICE"] \
        == "payment"

    assert params["VERSION"] \
        == "1.4.2"
```

---

# 69. Testing GitHub Actions Inputs

Validate generated workflow inputs:

```text
service
version
environment
registry
```

Unit-test the configuration builder.

Actual workflow execution is integration/E2E territory.

---

# 70. Testing ArgoCD Application Configuration

Validate:

```text
application name
project
repo
path
target revision
destination
```

before sending it to ArgoCD.

---

# 71. Test Prometheus Query Construction

Example:

```python
def error_rate_query(
    service
):

    return (
        f'error_rate{{'
        f'service="{service}"'
        f'}}'
    )
```

Test:

```python
def test_error_rate_query():

    result = error_rate_query(
        "payment"
    )

    assert 'service="payment"' \
        in result
```

---

# 72. Avoid Testing Prometheus Itself

Your unit test should verify:

```text
query construction
response parsing
threshold logic
```

not:

```text
Prometheus server implementation
```

---

# 73. Testing Log Parsing

Example:

```python
def count_errors(log_lines):

    return sum(
        1
        for line in log_lines
        if "ERROR" in line
    )
```

Test:

```python
def test_count_errors():

    logs = [
        "INFO started",
        "ERROR timeout",
        "INFO retry",
        "ERROR failed"
    ]

    assert count_errors(logs) == 2
```

---

# 74. Testing Structured Logs

For JSON logs:

```python
def parse_log(data):

    return {
        "level": data["level"],
        "message": data["message"]
    }
```

Test:

```python
def test_parse_log():

    data = {
        "level": "ERROR",
        "message": "timeout"
    }

    result = parse_log(data)

    assert result["level"] == "ERROR"
```

---

# 75. Testing Observability Policy

Example:

```python
def release_observable(
    metrics_available,
    logs_available
):

    return (
        metrics_available
        and logs_available
    )
```

Test combinations.

---

# 76. Testing Alert Threshold Logic

Example:

```python
def alert_required(
    error_rate,
    threshold
):

    return error_rate > threshold
```

Boundary tests:

```text
below
equal
above
```

---

# 77. Testing SLO Logic

If:

```text
availability SLO = 99.9%
```

test:

```text
99.8 -> breach
99.9 -> pass
100  -> pass
```

The exact boundary must match your SLO policy.

---

# 78. Testing Release Gates

A release gate may require:

```text
Tests pass
Security pass
Image verified
ArgoCD synced
Kubernetes healthy
Metrics healthy
```

Represent the policy as deterministic logic and unit-test every important combination.

---

# 79. Example Release Gate

```python
def release_allowed(
    tests,
    security,
    image,
    deployment
):

    return (
        tests
        and security
        and image
        and deployment
    )
```

---

# 80. Test Release Gate

```python
def test_release_allowed():

    assert release_allowed(
        True,
        True,
        True,
        True
    )
```

Failure:

```python
def test_security_blocks_release():

    assert not release_allowed(
        True,
        False,
        True,
        True
    )
```

---

# 81. Testing Partial Failure

Suppose:

```text
Tests = pass
Security = pass
Image = pass
Deployment = fail
```

Expected:

```text
release blocked
```

This should be explicitly tested.

---

# 82. Testing Approval Logic

Example:

```python
def production_allowed(
    environment,
    approved
):

    if environment == "prod":
        return approved

    return True
```

Tests:

```python
def test_prod_without_approval():

    assert not production_allowed(
        "prod",
        False
    )
```

and:

```python
def test_prod_with_approval():

    assert production_allowed(
        "prod",
        True
    )
```

---

# 83. Testing Dry-Run Logic

```python
def operation_allowed(
    dry_run
):

    return not dry_run
```

But real dry-run behavior should verify:

```text
read operations occur
mutating operations do not occur
```

The latter is better tested with mocking/integration techniques.

---

# 84. Testing CLI Exit Behavior

For command-line tools, test:

```text
success -> 0
validation failure -> non-zero
deployment failure -> non-zero
```

A CI pipeline depends on reliable exit codes.

---

# 85. Unit Testing Classes

Example:

```python
class DeploymentValidator:

    def __init__(self, environment):
        self.environment = environment

    def is_production(self):
        return self.environment == "prod"
```

Test:

```python
def test_production():

    validator = DeploymentValidator(
        "prod"
    )

    assert validator.is_production()
```

---

# 86. Class Test Isolation

Each test should create its own object when state matters.

Avoid:

```python
validator = DeploymentValidator(...)
```

as a shared mutable global.

Prefer fixtures if construction is repeated.

---

# 87. Testing State Transitions

A release workflow may have:

```text
PENDING
 ↓
RUNNING
 ↓
VERIFYING
 ↓
SUCCESS
```

or:

```text
RUNNING
 ↓
FAILED
 ↓
ROLLING_BACK
 ↓
ROLLED_BACK
```

Test valid transitions and invalid transitions.

---

# 88. State Machine Example

```python
def next_state(
    state,
    event
):

    transitions = {
        ("PENDING", "start"):
            "RUNNING",
        ("RUNNING", "success"):
            "SUCCESS",
        ("RUNNING", "failure"):
            "FAILED"
    }

    return transitions.get(
        (state, event)
    )
```

Test every supported transition.

---

# 89. Invalid State Transition

```python
def test_invalid_transition():

    assert next_state(
        "SUCCESS",
        "retry"
    ) is None
```

This prevents unexpected workflow behavior.

---

# 90. Testing Idempotent State Changes

If:

```text
SUCCESS -> success event
```

is received twice, the workflow should not unexpectedly restart.

Test duplicate events.

---

# 91. Testing Event Ordering

If events can arrive:

```text
success
failure
```

out of order, test that stale events do not overwrite a newer terminal state.

This is important for asynchronous CI/CD systems.

---

# 92. Unit Test Naming for Workflows

Good:

```text
test_failed_deployment_enters_rollback
test_successful_release_enters_completed_state
test_duplicate_success_event_is_ignored
test_stale_event_does_not_change_terminal_state
```

These names communicate system behavior.

---

# 93. Testing Exceptions with Context

If your custom exception has:

```python
class DeploymentError(Exception):

    def __init__(
        self,
        message,
        service,
        environment
    ):
        super().__init__(message)
        self.service = service
        self.environment = environment
```

test:

```python
def test_deployment_error_context():

    error = DeploymentError(
        "failed",
        "payment",
        "prod"
    )

    assert error.service == "payment"
    assert error.environment == "prod"
```

---

# 94. Custom Exceptions

DevOps automation benefits from meaningful exception types:

```text
ValidationError
AuthenticationError
AuthorizationError
RateLimitError
TransientAPIError
DeploymentError
RollbackError
ConfigurationError
```

Unit tests should verify correct classification.

---

# 95. Exception Hierarchy

Possible:

```text
AutomationError
├── ConfigurationError
├── APIError
│   ├── AuthenticationError
│   ├── AuthorizationError
│   ├── RateLimitError
│   └── TransientAPIError
├── DeploymentError
└── RollbackError
```

This allows callers to catch:

```python
AutomationError
```

or a specific subtype.

---

# 96. Testing Exception Hierarchy

```python
def test_rate_limit_error():

    error = RateLimitError()

    assert isinstance(
        error,
        APIError
    )

    assert isinstance(
        error,
        AutomationError
    )
```

---

# 97. Testing Pure Transformation Functions

Examples:

```text
API response -> internal model
Kubernetes object -> status
Config -> normalized config
Release -> deployment payload
```

These are excellent unit-test candidates.

---

# 98. Testing Parsers

Example:

```python
def parse_version(data):

    return data["version"]
```

Tests should cover:

```text
valid response
missing version
None
wrong type
```

according to the parser contract.

---

# 99. Testing Defensive Parsing

Bad:

```python
version = data["version"]
```

If external data is unreliable, defensive parsing may be appropriate.

Example:

```python
version = data.get("version")
```

Then validate:

```python
if not version:
    raise ValueError(
        "Missing version"
    )
```

Unit tests should cover both valid and invalid data.

---

# 100. Contract-Driven Unit Tests

Before writing tests, define:

```text
Input
Expected output
Expected exception
Side effects
```

Example:

```text
Input:
status=503

Output:
retryable

Exception:
none

Side effect:
none
```

This makes tests precise.

---

# 101. Side Effects

A function may:

```text
return a value
```

and:

```text
change external state
```

Unit testing should isolate the external side effect.

Example:

```text
Deployment policy
```

should be tested separately from:

```text
Kubernetes API call
```

---

# 102. Dependency Injection

A useful design:

```python
class DeploymentService:

    def __init__(
        self,
        client
    ):
        self.client = client
```

Then tests can provide a controlled client.

Detailed fake/mock techniques are covered in:

```text
03-Mocking.md
```

---

# 103. Why Dependency Injection Helps Testing

Without dependency injection:

```text
function
 ↓
creates API client internally
 ↓
calls real API
```

Hard to unit test.

With dependency injection:

```text
function
 ↓
provided client
 ↓
controlled test dependency
```

Much easier to isolate.

---

# 104. Testability as an Architecture Property

Good architecture:

```text
Business logic
      |
      +---- API adapter
      |
      +---- Kubernetes adapter
      |
      +---- AWS adapter
```

Business logic can be unit-tested without those adapters.

---

# 105. Recommended DevOps Python Architecture

```text
src/
└── devops_automation/
    ├── config/
    ├── models/
    ├── policies/
    ├── clients/
    │   ├── github.py
    │   ├── jenkins.py
    │   ├── argocd.py
    │   ├── kubernetes.py
    │   └── aws.py
    │
    ├── workflows/
    │   └── release.py
    │
    └── utils/
```

Tests:

```text
tests/
├── unit/
│   ├── test_config.py
│   ├── test_policies.py
│   ├── test_models.py
│   ├── test_clients.py
│   └── test_release.py
│
├── integration/
└── e2e/
```

---

# 106. Unit Testing the Architecture

The unit suite should mostly cover:

```text
models
config
policies
parsers
transformers
workflow decisions
client behavior through controlled dependencies
```

---

# 107. Unit Testing API Clients

Without mocks yet, focus on deterministic pieces:

```text
URL construction
headers
payload
response parsing
status classification
```

Later:

```text
03-Mocking.md
```

will show how to test actual client interaction without real APIs.

---

# 108. Testing Request Headers

Example:

```python
def build_headers(token):

    return {
        "Authorization":
            f"Bearer {token}",
        "Content-Type":
            "application/json"
    }
```

Test:

```python
def test_headers():

    headers = build_headers(
        "test-token"
    )

    assert headers[
        "Content-Type"
    ] == "application/json"
```

Do not print real secrets.

---

# 109. Testing Authentication Logic

Example:

```python
def token_present(token):

    return bool(token)
```

Test:

```python
@pytest.mark.parametrize(
    "token,expected",
    [
        ("abc", True),
        ("", False),
        (None, False)
    ]
)
def test_token_present(
    token,
    expected
):

    assert token_present(
        token
    ) == expected
```

---

# 110. Testing Token Expiration Logic

Example:

```python
def token_expired(
    expires_at,
    now
):

    return now >= expires_at
```

Test:

```text
before expiration
at expiration
after expiration
```

---

# 111. Testing Refresh Policy

Example:

```text
Token valid
 -> use token

Token expired
 -> refresh

Refresh failed
 -> stop
```

Each decision should have a unit test.

---

# 112. Testing Secret Redaction

```python
def redact(
    message,
    secret
):

    return message.replace(
        secret,
        "[REDACTED]"
    )
```

Test:

```python
def test_redaction():

    result = redact(
        "token=abc123",
        "abc123"
    )

    assert "abc123" not in result
    assert "[REDACTED]" in result
```

---

# 113. Test Logging Behavior

You should verify that important events are logged:

```text
release started
release failed
rollback started
rollback completed
```

But avoid making unit tests overly dependent on exact log formatting unless log structure is part of the contract.

---

# 114. Testing Structured Events

Better than raw log strings:

```python
event = {
    "event": "release_failed",
    "service": "payment",
    "environment": "prod"
}
```

Test required fields.

---

# 115. Testing Audit Events

```python
def build_audit_event(
    service,
    environment,
    result
):

    return {
        "service": service,
        "environment": environment,
        "result": result
    }
```

Test:

```python
def test_audit_event():

    event = build_audit_event(
        "payment",
        "prod",
        "success"
    )

    assert event["service"] == \
        "payment"

    assert event["result"] == \
        "success"
```

---

# 116. Testing Configuration Defaults

Example:

```python
def timeout_from_config(
    config
):

    return config.get(
        "timeout",
        10
    )
```

Test:

```python
def test_default_timeout():

    assert timeout_from_config(
        {}
    ) == 10
```

And configured value:

```python
def test_configured_timeout():

    assert timeout_from_config(
        {"timeout": 30}
    ) == 30
```

---

# 117. Test Dangerous Defaults

Some settings should never silently default.

For example:

```text
production account
production namespace
production credentials
```

Unit tests should verify:

```text
missing critical production configuration
```

causes:

```text
explicit failure
```

---

# 118. Test Safe Defaults

Other settings can safely default:

```text
timeout
log level
retry count
```

but the default must be intentional and documented.

---

# 119. Testing Feature Flags

Example:

```python
def feature_enabled(
    config,
    name
):

    return config.get(
        "features",
        {}
    ).get(name, False)
```

Test:

```text
enabled
disabled
missing
```

Feature flags should be tested because incorrect defaults can change production behavior.

---

# 120. Testing Environment Variables

A function might read:

```text
DEPLOY_ENV
```

Tests should cover:

```text
present
missing
invalid
```

Use pytest's environment helpers or controlled fixtures rather than relying on the developer's shell.

---

# 121. Testing File Operations

Example:

```python
def write_version(
    path,
    version
):

    path.write_text(version)
```

Test using pytest's:

```text
tmp_path
```

fixture.

```python
def test_write_version(tmp_path):

    path = tmp_path / "version"

    write_version(
        path,
        "1.4.2"
    )

    assert path.read_text() \
        == "1.4.2"
```

---

# 122. Testing YAML/JSON Generation

If Python generates configuration:

```text
Generate
 ↓
Write
 ↓
Read
 ↓
Validate
```

Unit tests can validate:

```text
required keys
values
types
```

without deploying the configuration.

---

# 123. Testing File Permissions

For automation that creates:

```text
SSH keys
config files
credentials files
```

test intended permission logic where supported by the target platform.

Example expectation:

```text
private key -> restrictive permissions
```

Do not test against actual private credentials.

---

# 124. Testing Shell Command Construction

If Python builds commands:

```python
def docker_tag(
    image,
    tag
):

    return f"{image}:{tag}"
```

Unit-test the generated command/value.

Avoid shell execution in unit tests.

Actual command execution belongs in integration tests.

---

# 125. Avoid Testing Implementation Details

Bad test:

```text
assert internal_variable == ...
```

when that variable is not part of the behavior.

Good:

```text
assert function result == expected behavior
```

Tests should survive safe refactoring.

---

# 126. Refactoring Safety

Good unit tests allow developers to change:

```text
implementation
```

without changing:

```text
behavior
```

If a harmless refactor breaks many tests, the tests may be coupled too tightly to implementation details.

---

# 127. Test Maintainability

A test suite should be:

```text
Readable
Reusable
Fast
Predictable
Easy to debug
```

Avoid excessive abstraction in tests.

A little duplication can sometimes be clearer than an overly complicated test framework.

---

# 128. Test Data Builders

For complex objects:

```python
def make_deployment(
    name="payment",
    replicas=3,
    ready=3
):

    return {
        "name": name,
        "replicas": replicas,
        "ready": ready
    }
```

Tests can override only what matters:

```python
deployment = make_deployment(
    ready=2
)
```

This keeps tests focused.

---

# 129. Testing Complex Objects

Instead of asserting the entire object:

```python
assert result == huge_dictionary
```

assert the important contract:

```python
assert result["name"] == "payment"
assert result["ready"] == 3
```

unless exact object equality is the requirement.

---

# 130. Snapshot-Style Thinking

Large API responses can be difficult to assert manually.

For stable structured contracts, snapshot-style tests can be useful, but use them carefully.

A snapshot that changes every time provides little protection.

---

# 131. Testing Error Messages

Test exact messages only when the message is part of the user/API contract.

Otherwise prefer:

```python
with pytest.raises(
    ValueError,
    match="Invalid environment"
):
    ...
```

or inspect structured exception fields.

---

# 132. Testing Warnings

If code intentionally emits warnings:

```python
import pytest


def test_warning():

    with pytest.warns(
        DeprecationWarning
    ):
        deprecated_function()
```

Useful when migrating automation libraries.

---

# 133. Testing Deprecation

DevOps automation may depend on:

```text
AWS SDK
Kubernetes client
GitHub API
Jenkins API
```

If a method is deprecated, tests should help detect migration requirements before the dependency becomes unavailable.

---

# 134. Dependency Upgrade Testing

Before upgrading:

```text
pytest
```

After upgrading:

```text
pytest
integration tests
```

Compare failures.

Pinning dependencies and testing upgrades are complementary practices.

---

# 135. Unit Test Quality Review

For each critical function ask:

```text
What is valid input?
What is invalid input?
What are boundary values?
What can fail?
What should happen on failure?
Is the operation idempotent?
Is there a timeout?
Is retry appropriate?
Is the result safe?
```

---

# 136. Production Risk-Based Testing

Prioritize tests based on impact.

### High risk

```text
Production deployment
Rollback
Credential selection
Environment mapping
Security gates
IAM decisions
```

### Medium risk

```text
API parsing
Metrics policy
Log parsing
Configuration
```

### Lower risk

```text
Formatting
simple utility functions
```

All code can be tested, but critical paths deserve stronger coverage and failure scenarios.

---

# 137. Unit Testing and DevSecOps

Testing is one stage:

```text
Code
 ↓
Lint
 ↓
Unit Tests
 ↓
SAST
 ↓
SCA
 ↓
Container Scan
 ↓
Integration
 ↓
Deploy
```

Your unit tests should not replace:

```text
SonarQube
Trivy
Veracode
```

They complement them.

---

# 138. Unit Tests and SonarQube

SonarQube can consume:

```text
test execution
coverage
quality metrics
```

A common CI flow:

```text
pytest
 ↓
coverage report
 ↓
SonarQube analysis
 ↓
quality gate
```

---

# 139. Unit Tests and Trivy

Trivy scans:

```text
container
filesystem
dependencies
configuration
```

pytest verifies:

```text
application behavior
```

Both are needed.

---

# 140. Unit Tests and Veracode

Security testing may identify:

```text
code vulnerabilities
```

Unit tests can verify:

```text
security-sensitive behavior
```

Example:

```text
secret redaction
authorization policy
unsafe input rejection
```

---

# 141. CI Quality Gate

A strong pipeline:

```text
Commit
 ↓
pytest
 ↓
coverage
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Build
```

If critical tests fail:

```text
stop pipeline
```

---

# 142. Testing Pipeline Configuration

Even pipeline configuration should be reviewed/tested.

Examples:

```text
Correct pytest command
Correct test path
Correct environment
Correct credentials
Correct report location
Correct failure behavior
```

A pipeline that silently ignores pytest failures is a production risk.

---

# 143. Dangerous CI Pattern

Avoid:

```bash
pytest || true
```

for mandatory tests.

This changes:

```text
test failure
```

into:

```text
pipeline success
```

---

# 144. Test Exit Codes

CI relies on:

```text
0 = success
non-zero = failure
```

Verify your test command behaves correctly.

---

# 145. Test Artifacts

Useful CI artifacts:

```text
JUnit XML
Coverage XML
Coverage HTML
pytest logs
```

These make failures easier to investigate.

---

# 146. Unit Test Debugging

When a unit test fails:

```text
Read assertion
 ↓
Read actual vs expected
 ↓
Check test input
 ↓
Check fixture
 ↓
Check function behavior
 ↓
Reproduce locally
 ↓
Fix root cause
```

Do not immediately modify the expected value just to make the test pass.

---

# 147. Test Smell: Changing Expected Value

Suppose:

```text
Expected = 10
Actual = 9
```

Do not blindly change:

```text
expected = 9
```

First ask:

```text
Is 9 correct?
Did the implementation regress?
Did the requirement change?
Is the test wrong?
```

---

# 148. Test Smell: Excessive Mocking

Too many mocks can create:

```text
tests that verify mocks
```

instead of:

```text
real behavior
```

Use mocks to isolate boundaries, not to recreate the entire application.

Detailed mocking strategy is next.

---

# 149. Test Smell: Huge Test

If a test contains:

```text
100 lines of setup
```

for:

```text
one assertion
```

consider:

```text
fixture
factory
helper
smaller test
```

But do not hide critical behavior behind excessive helpers.

---

# 150. Test Smell: Shared Global State

Avoid:

```python
GLOBAL_STATE = {}
```

modified by many tests.

This causes:

```text
order dependency
parallel failures
flakiness
```

---

# 151. Test Smell: Real Sleep

Bad:

```python
time.sleep(30)
```

in a unit test.

Better:

```text
inject time/sleep
control it
```

Detailed techniques are covered in mocking.

---

# 152. Test Smell: Real Network

Avoid:

```text
unit test -> GitHub
unit test -> AWS
unit test -> Kubernetes
```

Use:

```text
unit tests -> controlled dependency
integration tests -> real test dependency
```

---

# 153. Test Smell: Production Credentials

Never make unit tests depend on:

```text
production AWS keys
production GitHub token
production Kubernetes credentials
```

Use:

```text
fake/test credentials
```

and isolated integration identities.

---

# 154. Test Smell: Production Data

Do not use real production customer data to make tests realistic.

Use:

```text
synthetic data
sanitized data
fixtures
```

---

# 155. Unit Test Review Checklist

For every important function:

```text
[ ] Happy path
[ ] Invalid input
[ ] None/empty input
[ ] Boundary values
[ ] Exception behavior
[ ] Security-sensitive behavior
[ ] Idempotency
[ ] Failure policy
[ ] Deterministic output
```

---

# 156. DevOps Unit Test Checklist

```text
[ ] Environment mapping
[ ] AWS account mapping
[ ] Kubernetes namespace
[ ] Deployment name
[ ] Image reference
[ ] Digest verification
[ ] ArgoCD state
[ ] Prometheus threshold
[ ] Log parsing
[ ] Retry policy
[ ] Timeout policy
[ ] Rollback policy
[ ] Security gate
[ ] Production approval
[ ] Dry-run behavior
[ ] Configuration validation
```

---

# 157. Practical Project 1 — Environment Safety

Build:

```text
environment_policy.py
```

Functions:

```text
validate_environment()
account_for()
cluster_for()
namespace_for()
production_allowed()
```

Create:

```text
tests/unit/test_environment_policy.py
```

Test all mappings and dangerous combinations.

---

# 158. Practical Project 2 — Deployment Policy

Build:

```text
deployment_policy.py
```

Functions:

```text
deployment_ready()
argocd_healthy()
metrics_healthy()
release_allowed()
should_rollback()
```

Create:

```text
tests/unit/test_deployment_policy.py
```

Use parametrized policy matrices.

---

# 159. Practical Project 3 — Release State Machine

Build:

```text
release_state.py
```

States:

```text
PENDING
RUNNING
VERIFYING
SUCCESS
FAILED
ROLLING_BACK
ROLLED_BACK
```

Test:

```text
valid transitions
invalid transitions
duplicate events
stale events
terminal states
```

---

# 160. Practical Project 4 — Configuration Validator

Build:

```text
config_validator.py
```

Validate:

```text
environment
cluster
namespace
registry
ArgoCD URL
timeouts
retry count
security policy
```

Test:

```text
missing fields
invalid values
safe defaults
dangerous defaults
```

---

# 161. Practical Project 5 — Kubernetes Manifest Builder

Build:

```text
manifest_builder.py
```

Generate:

```text
Deployment
Service
Ingress
ConfigMap
Secret references
```

Unit-test:

```text
names
labels
images
ports
probes
resources
security context
```

Actual cluster validation belongs in integration tests.

---

# 162. Practical Project 6 — Release Gate

Build:

```text
release_gate.py
```

Inputs:

```text
unit_tests
security
image
argocd
kubernetes
metrics
```

Output:

```text
ALLOW
BLOCK
ROLLBACK
```

Test every important combination.

---

# 163. Production Architecture

Your unit-testing layer should fit into:

```text
                 GitHub
                    |
                    v
              Jenkins / GHA
                    |
          +---------+---------+
          |         |         |
        Lint      pytest   Security
                    |
             Unit Tests
                    |
             Integration
                    |
                  Build
                    |
                   ECR
                    |
                 ArgoCD
                    |
                   EKS
                    |
          +---------+---------+
          |         |         |
       Metrics     Logs     Health
          |         |         |
          +---------+---------+
                    |
              Release Gate
```

Unit tests should remain fast and independent of the downstream infrastructure.

---

# 164. Production Testing Separation

Use:

```text
Unit
  ↓
logic

Integration
  ↓
dependencies

E2E
  ↓
workflow
```

Do not turn every test into E2E.

---

# 165. Interview Scenario — AWS

### Question

How would you unit-test Python code that selects an AWS account?

Answer:

```text
I would isolate the environment-to-account mapping
into a deterministic function and unit-test every
supported environment plus invalid input.

I would not call AWS from the unit test.

AWS identity and permission behavior would be validated
in integration tests using a controlled test account.
```

---

# 166. Interview Scenario — Kubernetes

### Question

How would you unit-test Kubernetes automation?

Answer:

```text
I would separate Kubernetes API interaction from
deployment decision logic.

Unit tests would validate manifest construction,
namespace selection, readiness calculation,
rollback decisions, and status parsing using
controlled data.

A real test cluster would be used for integration testing.
```

---

# 167. Interview Scenario — ArgoCD

### Question

How do you test ArgoCD automation?

Answer:

```text
Unit:
application name/path construction,
response parsing,
sync/health policy.

Integration:
real test ArgoCD application.

E2E:
Git change -> ArgoCD -> EKS -> health verification.
```

---

# 168. Interview Scenario — Rollback

### Question

How do you unit-test rollback?

Answer:

```text
I test the rollback decision separately from the
rollback API call.

Inputs include deployment health, error rate,
and known-good revision.

I test healthy, degraded, threshold boundary,
missing previous version, and rollback-required cases.
```

---

# 169. Interview Scenario — Retry

### Question

How do you unit-test retries?

Answer:

```text
I verify which errors are retryable,
maximum attempts, backoff calculation,
and final failure behavior.

I test sequences such as:

503 -> 503 -> 200

and:

503 -> 503 -> 503 -> retry exhausted.
```

---

# 170. Interview Scenario — Idempotency

### Question

Why test idempotency?

Because CI/CD systems can retry operations after:

```text
timeouts
network failures
worker crashes
```

The test should prove:

```text
same request
+
same release ID
=
no duplicate side effect
```

---

# 171. Interview Scenario — Flaky Tests

### Question

A test passes locally but fails in Jenkins. What do you check?

I check:

```text
Python version
dependencies
environment variables
timezone
filesystem
network
credentials
test isolation
fixture scope
parallel execution
timing
```

Then I reproduce the smallest failing test and determine whether the problem is:

```text
test
code
environment
dependency
```

---

# 172. Interview Scenario — 100% Coverage

### Question

Would you require 100% coverage?

Strong answer:

```text
I use coverage as a signal, not the definition of
test quality.

I prioritize critical production paths and failure
modes. A 100% line coverage suite can still miss
important business behavior and integration failures.
```

---

# 173. Interview Scenario — Test Pyramid

### Question

Explain your test pyramid.

Answer:

```text
Many fast unit tests
        ↓
Fewer integration tests
        ↓
Small number of E2E tests
```

For DevOps automation:

```text
Unit:
policies/parsers/validators

Integration:
AWS/Kubernetes/ArgoCD

E2E:
complete release workflow
```

---

# 174. Interview Scenario — Production Testing

### Question

Would you run pytest against production?

Answer:

```text
Not the entire suite.

I would run carefully designed read-only or synthetic
smoke tests against production.

Destructive integration/E2E tests should run in an
isolated environment.
```

---

# 175. Interview Scenario — Security

### Question

How do unit tests help DevSecOps?

They validate security-sensitive behavior such as:

```text
authorization decisions
input validation
secret redaction
production guardrails
security gate policies
```

They complement:

```text
SAST
SCA
container scanning
DAST
```

---

# 176. Senior Interview — Testability

### Question

How do you design Python automation for testability?

Answer:

```text
Separate business logic from external adapters.

Use dependency injection where useful.

Keep transformation and policy functions pure.

Avoid hidden global state.

Make timeouts/retries configurable.

Keep side effects at clear boundaries.

Then unit-test the core logic independently.
```

---

# 177. Senior Interview — Test Architecture

### Question

How would you structure a large DevOps Python test suite?

Answer:

```text
tests/
├── unit/
├── integration/
└── e2e/
```

Within unit:

```text
config
models
policies
parsers
clients
workflows
```

Use markers:

```text
unit
integration
e2e
aws
kubernetes
slow
```

and execute appropriate subsets at each CI stage.

---

# 178. Senior Interview — Release Confidence

### Question

How do you establish confidence before production?

Use layered validation:

```text
Lint
 ↓
Unit
 ↓
Security
 ↓
Integration
 ↓
Build
 ↓
Test deployment
 ↓
Smoke
 ↓
E2E
 ↓
Production verification
```

Each layer answers a different risk question.

---

# 179. Senior Interview — Failure Testing

### Question

What failures would you prioritize?

For DevOps automation:

```text
Authentication
Authorization
Rate limiting
Timeout
Server error
Malformed response
Wrong environment
Wrong artifact
Deployment degradation
Metrics breach
Rollback failure
Duplicate request
Concurrent release
```

These have high operational impact.

---

# 180. Senior Interview — Unit vs Mock

### Question

Why not mock everything?

Answer:

```text
Because excessive mocking can make tests verify
the mocked implementation rather than actual behavior.

I mock external boundaries when needed, but keep
core business logic real and deterministic.
```

Detailed mocking strategy comes next.

---

# 181. Production-Ready Unit Testing Principles

```text
1. Test behavior.
2. Keep tests isolated.
3. Keep unit tests fast.
4. Make failures deterministic.
5. Test negative paths.
6. Test boundaries.
7. Test production guardrails.
8. Separate logic from side effects.
9. Avoid real external services in unit tests.
10. Use integration tests for dependency behavior.
11. Keep E2E tests focused.
12. Treat tests as production code.
```

---

# 182. Final Unit Testing Checklist

```text
Unit Fundamentals
[ ] Arrange-Act-Assert
[ ] Given-When-Then
[ ] Positive tests
[ ] Negative tests
[ ] Boundary tests
[ ] None/empty tests
[ ] Exception tests
[ ] Parametrization
[ ] Fixtures
[ ] Test isolation

DevOps Logic
[ ] Environment mapping
[ ] AWS account mapping
[ ] Namespace mapping
[ ] Image reference
[ ] Digest verification
[ ] Kubernetes readiness
[ ] ArgoCD health
[ ] Prometheus thresholds
[ ] Log parsing
[ ] Retry policy
[ ] Timeout policy
[ ] Rollback policy
[ ] Security gates
[ ] Production approval
[ ] Dry-run policy
[ ] State transitions
[ ] Idempotency
[ ] Reconciliation

CI/CD
[ ] pytest in Jenkins
[ ] pytest in GitHub Actions
[ ] Coverage
[ ] JUnit reports
[ ] Failure exit codes
[ ] Test artifacts
[ ] Test markers
[ ] Unit/integration separation

Production
[ ] No production credentials
[ ] No production data
[ ] No accidental production mutation
[ ] Deterministic tests
[ ] Flaky test control
[ ] Security-sensitive behavior
```

---

# 183. Key Takeaway

The most important mindset is:

```text
Do not only test:

"Does the function work?"

Also test:

"What happens when production behaves badly?"
```

For DevOps automation, that means testing:

```text
Wrong input
Wrong environment
Wrong account
Wrong namespace
Wrong artifact
401
403
429
503
Timeout
Malformed response
Unknown state
Duplicate operation
Concurrent operation
Deployment failure
Health failure
Metric breach
Rollback failure
Security failure
```

That is what turns Python automation from:

```text
a script
```

into:

```text
production-grade automation.
```

---

# 184. Next File

```text
09-Python-Testing/
├── 01-Pytest-Fundamentals.md       ✓
├── 02-Unit-Testing.md              ✓
├── 03-Mocking.md
├── 04-Test-Automation.md
└── 05-DevOps-Automation-Testing.md
```

Next:

## `03-Mocking.md`

The next file will focus deeply on isolating external dependencies:

```text
unittest.mock
Mock
MagicMock
patch
patch.object
side_effect
return_value
assert_called_once
call_args
autospec
spec
AsyncMock
mocking requests
mocking boto3
mocking Kubernetes client
mocking GitHub/Jenkins/ArgoCD
mocking time and sleep
mocking environment variables
mocking filesystem
fake vs mock vs stub
integration-test boundaries
common mocking mistakes
production DevOps examples
CI/CD testing
troubleshooting
senior interview scenarios
```
