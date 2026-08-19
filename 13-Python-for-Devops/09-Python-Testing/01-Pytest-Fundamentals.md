# 01 — Pytest Fundamentals

## 1. Overview

This is the first file in:

```text
09-Python-Testing/
├── 01-Pytest-Fundamentals.md
├── 02-Unit-Testing.md
├── 03-Mocking.md
├── 04-Test-Automation.md
└── 05-DevOps-Automation-Testing.md
```

The previous Python sections covered:

```text
Python Fundamentals
        ↓
Kubernetes Automation
        ↓
DevOps Tool Automation
        ↓
HTTP / REST APIs
        ↓
Authentication
        ↓
API Error Handling
        ↓
Production DevOps API Projects
```

Now we move into:

> **Testing Python automation before it reaches production.**

For DevOps engineers, testing is not just about testing Python functions.

We need to test:

```text
API clients
AWS automation
Kubernetes automation
CI/CD integrations
Infrastructure automation
Deployment workflows
Retry logic
Error handling
Rollback logic
Security gates
```

The production mindset is:

```text
Code
 ↓
Test
 ↓
Validate
 ↓
Integrate
 ↓
Deploy
 ↓
Observe
```

---

# 2. Why Testing Matters in DevOps

Consider a Python deployment script:

```text
Python
  ↓
Jenkins
  ↓
ECR
  ↓
ArgoCD
  ↓
EKS
```

A small coding mistake can cause:

```text
Wrong image
Wrong namespace
Wrong environment
Wrong deployment
Wrong rollback
Credential failure
Production outage
```

Testing catches these problems before production.

---

# 3. Testing Pyramid

A common testing model:

```text
             /\
            /  \
           / E2E\
          /------\
         /Integr. \
        /----------\
       /   Unit     \
      /--------------\
```

More unit tests:

```text
Fast
Cheap
Isolated
```

Fewer integration/E2E tests:

```text
Slower
More expensive
More dependencies
```

A mature DevOps project balances all three.

---

# 4. Test Levels

We will use:

```text
Unit Tests
Integration Tests
Contract Tests
End-to-End Tests
Smoke Tests
Regression Tests
Failure Tests
```

This file focuses primarily on:

```text
pytest fundamentals
```

Later files will go deeper into:

```text
unit testing
mocking
automation
DevOps testing
```

---

# 5. What Is pytest?

`pytest` is a Python testing framework designed to make tests:

```text
Simple
Readable
Extensible
Powerful
```

Basic test:

```python
def test_addition():
    assert 2 + 3 == 5
```

The test expresses:

```text
Expected behavior
```

---

# 6. Why pytest Is Popular

Advantages:

```text
Simple syntax
Powerful assertions
Fixtures
Parametrization
Markers
Plugins
Parallel execution support
Good CI/CD integration
Readable failure output
```

It works well for:

```text
Application code
API clients
DevOps scripts
Infrastructure automation
Kubernetes automation
AWS automation
```

---

# 7. Installation

Install:

```bash
python -m pip install pytest
```

Verify:

```bash
pytest --version
```

Example:

```text
pytest 8.x.x
```

The exact installed version depends on your environment.

---

# 8. Virtual Environment

Recommended:

```bash
python3 -m venv .venv
```

Activate on Linux/macOS:

```bash
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Then:

```bash
python -m pip install pytest
```

---

# 9. Project Structure

Recommended:

```text
python-project/
│
├── src/
│   └── calculator.py
│
├── tests/
│   └── test_calculator.py
│
├── requirements.txt
└── pyproject.toml
```

For larger DevOps projects:

```text
python-devops-automation/
│
├── src/
│   ├── clients/
│   ├── workflows/
│   ├── retry/
│   └── utils/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── pyproject.toml
└── requirements.txt
```

---

# 10. Test File Naming

pytest automatically discovers common patterns such as:

```text
test_*.py
*_test.py
```

Examples:

```text
test_github.py
test_argocd.py
deployment_test.py
```

Use a consistent convention.

Recommended:

```text
test_<module>.py
```

---

# 11. Test Function Naming

Example:

```python
def test_addition():
    assert 2 + 3 == 5
```

Another:

```python
def test_deployment_name_is_valid():
    ...
```

Good test names describe:

```text
behavior
```

not implementation details.

---

# 12. Running pytest

From the project root:

```bash
pytest
```

Verbose:

```bash
pytest -v
```

More detailed:

```bash
pytest -vv
```

Stop after first failure:

```bash
pytest -x
```

---

# 13. Example Output

```text
================ test session starts ================

collected 3 items

tests/test_math.py ...                       [100%]

================= 3 passed =================
```

A passing test means:

```text
Observed behavior matched expectation.
```

It does not prove the entire system is correct.

---

# 14. First pytest Test

Application:

```python
def add(a, b):
    return a + b
```

Test:

```python
def test_add():
    assert add(2, 3) == 5
```

Run:

```bash
pytest
```

---

# 15. Assertions

pytest uses Python's normal:

```python
assert
```

Examples:

```python
assert result == expected
assert result != unexpected
assert value is None
assert value is not None
assert item in collection
assert condition
```

---

# 16. Assertion Example

```python
def test_environment():
    environment = "prod"

    assert environment == "prod"
```

Failure:

```text
assert 'prod' == 'staging'
```

pytest provides useful failure information automatically.

---

# 17. Testing Collections

```python
def test_services():

    services = [
        "payment",
        "catalog",
        "orders"
    ]

    assert "payment" in services
    assert len(services) == 3
```

---

# 18. Testing Dictionaries

```python
def test_deployment():

    deployment = {
        "service": "payment",
        "version": "1.4.2"
    }

    assert deployment["service"] == "payment"
    assert deployment["version"] == "1.4.2"
```

---

# 19. Testing Strings

```python
def test_image():

    image = "payment:1.4.2"

    assert image.startswith("payment:")
    assert "1.4.2" in image
```

---

# 20. Testing Boolean Results

```python
def is_production(environment):
    return environment == "prod"
```

Test:

```python
def test_is_production():

    assert is_production("prod")
    assert not is_production("staging")
```

---

# 21. Testing Exceptions

Suppose:

```python
def deploy(environment):

    if environment == "invalid":
        raise ValueError("Invalid environment")
```

Test:

```python
import pytest


def test_invalid_environment():

    with pytest.raises(ValueError):
        deploy("invalid")
```

---

# 22. Verify Exception Message

```python
def test_invalid_environment():

    with pytest.raises(
        ValueError,
        match="Invalid environment"
    ):
        deploy("invalid")
```

This verifies both:

```text
exception type
exception message
```

---

# 23. Why Exception Testing Matters

DevOps automation often needs to fail safely.

Example:

```text
Invalid environment
        ↓
ValidationError
        ↓
Pipeline stops
```

Testing verifies that dangerous input does not silently continue.

---

# 24. Testing API Response Validation

Example function:

```python
def validate_response(data):

    if "status" not in data:
        raise ValueError(
            "Missing status"
        )

    return True
```

Test:

```python
def test_response_validation():

    data = {
        "status": "healthy"
    }

    assert validate_response(data)
```

---

# 25. Test Failure Case

```python
def test_missing_status():

    with pytest.raises(
        ValueError,
        match="Missing status"
    ):
        validate_response({})
```

---

# 26. Test Organization

Small project:

```text
tests/
└── test_utils.py
```

Large project:

```text
tests/
├── unit/
│   ├── test_github.py
│   ├── test_argocd.py
│   └── test_retry.py
│
├── integration/
│   ├── test_github_api.py
│   └── test_kubernetes_api.py
│
└── e2e/
    └── test_release.py
```

---

# 27. Unit vs Integration

### Unit

```text
One function/class
No real external dependency
Fast
```

### Integration

```text
Multiple components
Real or controlled external dependency
Slower
```

Example:

```text
Unit:
GitHub client response parser

Integration:
Python → GitHub API
```

---

# 28. Test Discovery

pytest searches test files and test functions/classes based on its discovery rules.

You can run:

```bash
pytest
```

or specific directories:

```bash
pytest tests/unit
```

or specific files:

```bash
pytest tests/unit/test_retry.py
```

---

# 29. Run One Test

Use:

```bash
pytest tests/test_calculator.py::test_add
```

Useful during development.

---

# 30. Run Matching Tests

Use:

```bash
pytest -k deployment
```

This selects tests whose names match the expression.

Example:

```text
test_deployment_success
test_deployment_failure
test_deployment_timeout
```

---

# 31. Test Verbosity

Normal:

```bash
pytest
```

Verbose:

```bash
pytest -v
```

Very verbose:

```bash
pytest -vv
```

Useful when:

```text
CI logs are difficult to understand
```

---

# 32. Stop on First Failure

```bash
pytest -x
```

Useful during debugging.

For example:

```text
test 1 = pass
test 2 = fail
```

pytest stops after:

```text
test 2
```

---

# 33. Run Last Failed Tests

pytest can support:

```bash
pytest --lf
```

This is useful after a large test suite where only a few tests failed.

---

# 34. Run Failed Tests First

```bash
pytest --ff
```

This runs previously failed tests first and then continues with the remaining tests.

---

# 35. Collect Tests

Use:

```bash
pytest --collect-only
```

This helps determine:

```text
Which tests pytest discovered?
```

Very useful when a test unexpectedly does not run.

---

# 36. Test Naming Mistake

Suppose file:

```text
checks.py
```

contains:

```python
def deployment_check():
    ...
```

pytest may not discover it depending on configuration.

Use standard naming:

```text
test_deployment.py
```

and:

```python
def test_deployment():
    ...
```

---

# 37. Test Classes

pytest supports test classes.

Example:

```python
class TestDeployment:

    def test_name(self):
        assert "payment" == "payment"

    def test_version(self):
        assert "1.4.2".startswith("1.")
```

Test methods generally start with:

```text
test_
```

---

# 38. Avoid Overusing Test Classes

You do not need a class for every test.

Functions are often simpler:

```python
def test_deployment_name():
    ...


def test_deployment_version():
    ...
```

Use classes when grouping behavior provides meaningful organization.

---

# 39. Fixtures

Fixtures provide reusable test setup.

Example:

```python
import pytest


@pytest.fixture
def deployment():

    return {
        "name": "payment",
        "version": "1.4.2"
    }
```

Use it:

```python
def test_deployment_name(deployment):

    assert deployment["name"] == "payment"
```

---

# 40. Why Fixtures Matter

Without fixtures:

```text
Repeated setup
Repeated cleanup
Duplicate data
```

Fixtures provide:

```text
Reusable setup
Dependency injection
Cleanup
Scope control
```

---

# 41. Simple Fixture

```python
@pytest.fixture
def service_name():
    return "payment"
```

Test:

```python
def test_service(service_name):

    assert service_name == "payment"
```

pytest automatically injects the fixture by argument name.

---

# 42. Fixture Dependency Injection

```python
@pytest.fixture
def service():
    return {
        "name": "payment",
        "environment": "staging"
    }


def test_service_environment(service):

    assert service["environment"] == "staging"
```

This is one of pytest's most important features.

---

# 43. Fixture Setup and Teardown

Use `yield`:

```python
@pytest.fixture
def resource():

    setup_resource()

    yield resource

    cleanup_resource()
```

Execution:

```text
setup
 ↓
test
 ↓
cleanup
```

---

# 44. Example Temporary Directory

pytest provides fixtures such as:

```python
def test_file(tmp_path):

    file = tmp_path / "config.txt"

    file.write_text("environment=staging")

    assert file.read_text() == \
        "environment=staging"
```

This avoids manually managing temporary paths.

---

# 45. Fixture Scope

Common scopes:

```text
function
class
module
package
session
```

Default:

```text
function
```

That means a fresh fixture instance is normally created for each test function.

---

# 46. Function Scope

```python
@pytest.fixture(scope="function")
def client():
    ...
```

Runs:

```text
once per test
```

Good for:

```text
isolated state
mutable objects
```

---

# 47. Module Scope

```python
@pytest.fixture(scope="module")
def api_client():
    ...
```

Runs once for the module.

Useful when setup is expensive and sharing is safe.

---

# 48. Session Scope

```python
@pytest.fixture(scope="session")
def configuration():
    ...
```

Runs once per pytest session.

Useful for:

```text
Read-only configuration
Expensive global setup
Test environment metadata
```

Be careful with mutable shared state.

---

# 49. Fixture Scope Tradeoff

Larger scope:

```text
Faster
Less setup
More shared state risk
```

Smaller scope:

```text
More isolated
More setup
Potentially slower
```

Choose based on:

```text
cost
isolation
state
```

---

# 50. `conftest.py`

Common shared fixtures belong in:

```text
tests/conftest.py
```

Example:

```python
import pytest


@pytest.fixture
def environment():
    return "staging"
```

Tests under that directory can use it without importing the fixture directly.

---

# 51. Why `conftest.py` Matters

It allows:

```text
Shared fixtures
Hooks
Configuration
Reusable test setup
```

without creating a large import structure.

---

# 52. Directory-Level Fixtures

Example:

```text
tests/
├── conftest.py
├── unit/
│   └── test_api.py
└── integration/
    ├── conftest.py
    └── test_kubernetes.py
```

The integration `conftest.py` can provide fixtures specific to integration tests.

---

# 53. Fixture Composition

One fixture can depend on another:

```python
@pytest.fixture
def config():
    return {
        "environment": "staging"
    }


@pytest.fixture
def client(config):
    return APIClient(
        environment=config["environment"]
    )
```

This creates a dependency chain:

```text
config
  ↓
client
  ↓
test
```

---

# 54. Autouse Fixtures

Example:

```python
@pytest.fixture(autouse=True)
def setup_environment():
    ...
```

It runs automatically for matching tests.

Use sparingly.

Why?

Because hidden setup can make tests harder to understand.

Prefer explicit fixture dependencies when possible.

---

# 55. Parametrization

Instead of writing:

```python
def test_prod():
    assert ...


def test_staging():
    assert ...


def test_dev():
    assert ...
```

use parametrization.

```python
@pytest.mark.parametrize(
    "environment",
    ["dev", "staging", "prod"]
)
def test_environment(environment):

    assert environment in {
        "dev",
        "staging",
        "prod"
    }
```

---

# 56. Parametrize Multiple Values

```python
@pytest.mark.parametrize(
    "status,expected",
    [
        (200, True),
        (201, True),
        (400, False),
        (500, False)
    ]
)
def test_status(status, expected):

    assert (200 <= status < 300) == expected
```

This is extremely useful for API testing.

---

# 57. API Status Testing

Example:

```python
@pytest.mark.parametrize(
    "status",
    [200, 201, 202]
)
def test_success_status(status):

    assert 200 <= status < 300
```

---

# 58. Error Status Testing

```python
@pytest.mark.parametrize(
    "status",
    [400, 401, 403, 404, 409, 429, 500, 503]
)
def test_error_status(status):

    assert status >= 400
```

Real API clients should test the actual behavior associated with each status, not merely the numeric range.

---

# 59. Parametrize IDs

For readable output:

```python
@pytest.mark.parametrize(
    "environment",
    [
        pytest.param(
            "dev",
            id="development"
        ),
        pytest.param(
            "prod",
            id="production"
        )
    ]
)
def test_environment(environment):
    assert environment
```

Useful in CI output.

---

# 60. Markers

Markers categorize tests.

Example:

```python
@pytest.mark.integration
def test_github_api():
    ...
```

Then:

```bash
pytest -m integration
```

---

# 61. Common DevOps Markers

You can define:

```text
unit
integration
e2e
smoke
slow
aws
kubernetes
security
```

Example:

```python
@pytest.mark.kubernetes
def test_deployment():
    ...
```

---

# 62. Marker Configuration

In `pyproject.toml`:

```toml
[tool.pytest.ini_options]
markers = [
    "unit: unit tests",
    "integration: integration tests",
    "e2e: end-to-end tests",
    "kubernetes: Kubernetes tests",
    "aws: AWS integration tests"
]
```

This makes marker intent explicit.

---

# 63. Run Specific Marker

```bash
pytest -m unit
```

Integration:

```bash
pytest -m integration
```

Exclude integration:

```bash
pytest -m "not integration"
```

---

# 64. DevOps CI Strategy

A typical pipeline:

```text
Git Push
   |
   v
Lint
   |
   v
Unit Tests
   |
   v
Security Scan
   |
   v
Build
   |
   v
Integration Tests
   |
   v
Deploy Test Environment
   |
   v
Smoke Tests
```

pytest can participate in multiple stages.

---

# 65. Fast Feedback

Developers should run:

```bash
pytest -m unit
```

before committing.

CI can run:

```bash
pytest -m "unit or integration"
```

Production-like environments can run:

```bash
pytest -m e2e
```

---

# 66. Test Configuration

pytest can be configured in:

```text
pyproject.toml
pytest.ini
tox.ini
setup.cfg
```

Modern Python projects commonly use:

```text
pyproject.toml
```

---

# 67. Basic `pyproject.toml`

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra"
```

This tells pytest:

```text
Look in tests/
Show extra summary information for skipped/xfail/etc.
```

---

# 68. Python Path Configuration

A clean package layout is preferable to relying on ad-hoc path manipulation.

Example:

```text
src/
└── devops_automation/
    ├── __init__.py
    └── clients/
```

Install the package in editable mode during development when appropriate:

```bash
pip install -e .
```

Then tests can import the actual package.

---

# 69. Avoid `sys.path.append()`

Bad test pattern:

```python
import sys

sys.path.append("../src")
```

This hides packaging problems.

Prefer:

```text
proper Python package
+
editable installation
```

---

# 70. Test Dependencies

Keep testing dependencies explicit.

Example:

```text
pytest
pytest-cov
pytest-xdist
```

Additional plugins should be added only when needed.

For a DevOps project, you may also use libraries such as:

```text
responses
respx
moto
```

depending on the type of integration being tested.

These will be covered more deeply in later files.

---

# 71. Test Requirements

You can maintain:

```text
requirements.txt
requirements-dev.txt
```

or use project dependency groups in `pyproject.toml`.

Example concept:

```text
Runtime:
requests
boto3

Development:
pytest
pytest-cov
ruff
```

---

# 72. Code Coverage

Coverage answers:

```text
Which lines/branches were executed by tests?
```

Install:

```bash
python -m pip install pytest-cov
```

Run:

```bash
pytest --cov=src
```

---

# 73. Coverage Example

```bash
pytest \
  --cov=src \
  --cov-report=term-missing
```

This can show:

```text
File
Statements
Missing
Coverage
```

---

# 74. Coverage Is Not Quality

Important:

```text
100% coverage
```

does not guarantee:

```text
100% correctness
```

You can execute every line without testing meaningful behavior.

Good testing focuses on:

```text
behavior
failure modes
business rules
risk
```

---

# 75. Branch Coverage

Consider:

```python
if status == "healthy":
    return True
else:
    return False
```

Testing only:

```text
healthy
```

does not test:

```text
unhealthy
```

Branch-oriented thinking matters.

---

# 76. Test the Happy Path

Example:

```text
Deployment
 |
 v
API 200
 |
 v
Synced
 |
 v
Healthy
 |
 v
Success
```

This should have a test.

---

# 77. Test Failure Paths

Also test:

```text
401
403
429
503
Timeout
Malformed response
Invalid state
```

DevOps automation often gets its greatest value from testing failure behavior.

---

# 78. Test Unknown State

Example:

```text
POST deployment
 |
 v
Timeout
 |
 v
Unknown
```

Your code should:

```text
Reconcile
```

not blindly:

```text
Retry POST
```

This is an important production test case.

---

# 79. Test Idempotency

Example:

```text
Release submitted twice
```

Expected:

```text
One actual deployment
```

Test:

```text
same release ID
same operation
no duplicate side effect
```

---

# 80. Test Retry Behavior

Example:

```text
Attempt 1 -> 503
Attempt 2 -> 503
Attempt 3 -> 200
```

Expected:

```text
success
```

Also test:

```text
Attempt 1 -> 503
Attempt 2 -> 503
Attempt 3 -> 503
Attempt 4 -> 503
```

Expected:

```text
retry exhausted
```

---

# 81. Test Timeout

Simulate:

```text
API does not respond
```

Expected:

```text
Timeout exception
```

and:

```text
bounded execution
```

The test should verify the automation does not hang indefinitely.

---

# 82. Test Rate Limit

Simulate:

```text
429
Retry-After: 10
```

Verify:

```text
Retry policy invoked
```

and ideally:

```text
Retry-After honored
```

without making the test actually sleep for 10 seconds.

Use injected clocks/sleep functions or mocking in appropriate tests; detailed mocking comes in the next file.

---

# 83. Test Authentication Failure

Simulate:

```text
401
```

Expected:

```text
Authentication handling
```

If token refresh is supported:

```text
refresh
 |
 v
retry once
```

If not:

```text
fail
```

---

# 84. Test Authorization Failure

Simulate:

```text
403
```

Expected:

```text
AuthorizationError
```

Not:

```text
infinite retry
```

---

# 85. Test Conflict

Simulate:

```text
409
```

Expected:

```text
reconciliation
```

depending on the API contract.

---

# 86. Test Server Error

Simulate:

```text
503
```

Expected:

```text
bounded retry
```

when the operation is safe.

---

# 87. Test Malformed JSON

Response:

```text
HTTP 200
```

Body:

```text
not-json
```

Expected:

```text
parsing error
```

not:

```text
false success
```

---

# 88. Test Missing Fields

Response:

```json
{
  "status": "healthy"
}
```

Expected schema:

```text
status
revision
health
```

If required field is missing:

```text
validation failure
```

---

# 89. Test Business Failure

Response:

```text
HTTP 200
```

Body:

```json
{
  "status": "failed"
}
```

Expected:

```text
BusinessOperationError
```

This verifies that the application does not confuse:

```text
HTTP success
```

with:

```text
business success
```

---

# 90. Fixtures for DevOps Testing

Example:

```python
@pytest.fixture
def release():

    return {
        "service": "payment",
        "environment": "staging",
        "version": "1.4.2"
    }
```

Test:

```python
def test_release_validation(release):

    assert release["service"] == "payment"
```

---

# 91. Fixture for API Configuration

```python
@pytest.fixture
def api_config():

    return {
        "base_url": "https://api.test.local",
        "timeout": 10
    }
```

This prevents repeated setup.

---

# 92. Fixture for Kubernetes Resource

```python
@pytest.fixture
def deployment():

    return {
        "metadata": {
            "name": "payment"
        },
        "status": {
            "ready_replicas": 3,
            "available_replicas": 3
        }
    }
```

Tests can use controlled data without requiring a real cluster.

---

# 93. Testing Kubernetes Logic

Suppose:

```python
def deployment_ready(status):

    return (
        status.get("ready_replicas", 0)
        == status.get("desired_replicas", 0)
    )
```

Test:

```python
def test_deployment_ready():

    status = {
        "ready_replicas": 3,
        "desired_replicas": 3
    }

    assert deployment_ready(status)
```

---

# 94. Kubernetes Failure Test

```python
def test_deployment_not_ready():

    status = {
        "ready_replicas": 2,
        "desired_replicas": 3
    }

    assert not deployment_ready(status)
```

This tests the business logic without needing a real EKS cluster.

---

# 95. Testing AWS Logic

Suppose:

```python
def image_exists(images, digest):

    return any(
        image["digest"] == digest
        for image in images
    )
```

Test:

```python
def test_image_exists():

    images = [
        {"digest": "sha256:abc"}
    ]

    assert image_exists(
        images,
        "sha256:abc"
    )
```

---

# 96. Testing ArgoCD Logic

Suppose:

```python
def deployment_successful(app):

    return (
        app["sync"] == "Synced"
        and app["health"] == "Healthy"
    )
```

Test:

```python
def test_argocd_success():

    app = {
        "sync": "Synced",
        "health": "Healthy"
    }

    assert deployment_successful(app)
```

---

# 97. Test ArgoCD Degraded

```python
def test_argocd_degraded():

    app = {
        "sync": "Synced",
        "health": "Degraded"
    }

    assert not deployment_successful(app)
```

---

# 98. Testing Prometheus Policy

Suppose:

```python
def metrics_healthy(
    error_rate,
    p95_latency,
    max_error_rate,
    max_latency
):
    return (
        error_rate <= max_error_rate
        and p95_latency <= max_latency
    )
```

Test:

```python
def test_metrics_healthy():

    assert metrics_healthy(
        error_rate=0.002,
        p95_latency=200,
        max_error_rate=0.01,
        max_latency=500
    )
```

---

# 99. Test Metrics Failure

```python
def test_metrics_unhealthy():

    assert not metrics_healthy(
        error_rate=0.08,
        p95_latency=200,
        max_error_rate=0.01,
        max_latency=500
    )
```

---

# 100. Testing Rollback Policy

```python
def should_rollback(
    health,
    error_rate
):
    return (
        health != "Healthy"
        or error_rate > 0.05
    )
```

Test:

```python
def test_rollback_on_high_error_rate():

    assert should_rollback(
        health="Healthy",
        error_rate=0.08
    )
```

---

# 101. Boundary Testing

Always test thresholds.

If:

```text
max_error_rate = 1%
```

test:

```text
0.9%
1.0%
1.1%
```

This catches:

```text
< vs <=
```

bugs.

---

# 102. Parameterized Boundary Test

```python
@pytest.mark.parametrize(
    "error_rate,expected",
    [
        (0.009, True),
        (0.010, True),
        (0.011, False)
    ]
)
def test_error_threshold(
    error_rate,
    expected
):

    assert metrics_healthy(
        error_rate=error_rate,
        p95_latency=100,
        max_error_rate=0.01,
        max_latency=500
    ) == expected
```

---

# 103. Test Naming Strategy

Good:

```text
test_deployment_fails_when_replicas_are_missing
test_api_retries_on_503
test_release_rolls_back_on_high_error_rate
test_invalid_environment_raises_validation_error
```

Weak:

```text
test_1
test_api
test_deployment
```

Names should communicate behavior.

---

# 104. Arrange-Act-Assert

A common test structure:

```text
Arrange
Act
Assert
```

Example:

```python
def test_image_verification():

    # Arrange
    digest = "sha256:abc"

    # Act
    result = verify_digest(digest)

    # Assert
    assert result is True
```

---

# 105. Given-When-Then

Another useful style:

```text
Given
When
Then
```

Example:

```text
Given a deployment with 3 desired replicas
When only 2 replicas are ready
Then rollout verification fails
```

This is particularly useful for scenario-based tests.

---

# 106. Test Independence

Tests should ideally be independent.

Bad:

```text
test_create
   ↓
test_update
   ↓
test_delete
```

If `test_create` fails:

```text
other tests fail
```

Better:

```text
Each test creates/controls its own required state
```

using fixtures.

---

# 107. Test Isolation

Avoid shared mutable state:

```python
services = []
```

used across many tests.

One test can modify it and affect another.

Prefer fresh fixtures.

---

# 108. Deterministic Tests

A good test produces the same result repeatedly.

Avoid dependencies on:

```text
Current time
Randomness
Network
External production API
Shared environment
```

unless the test is specifically designed as an integration/E2E test.

---

# 109. Flaky Tests

A flaky test:

```text
passes sometimes
fails sometimes
```

Possible causes:

```text
Timing
Race condition
External dependency
Shared state
Network
Random data
Eventual consistency
```

Do not simply add:

```text
sleep(10)
```

as the default fix.

---

# 110. Testing Time

Instead of:

```python
time.sleep(10)
```

design code so time can be controlled.

For example:

```python
def wait_until_ready(
    check,
    sleep_fn,
    timeout
):
    ...
```

Then tests can provide a controlled sleep function.

Detailed mocking patterns are covered in:

```text
03-Mocking.md
```

---

# 111. Testing Retry Logic

A retry test should verify:

```text
number of attempts
delay policy
final result
```

Example scenario:

```text
503
503
200
```

Expected:

```text
3 attempts
success
```

---

# 112. Testing Retry Exhaustion

Scenario:

```text
503
503
503
503
```

Expected:

```text
retry exhausted
exception raised
```

This prevents infinite retry bugs.

---

# 113. Testing Maximum Attempts

If:

```text
max_attempts = 4
```

the test should prove:

```text
attempts <= 4
```

This is especially important for CI/CD automation.

---

# 114. Testing Deadline

If workflow deadline is:

```text
5 minutes
```

test:

```text
deadline reached
```

Expected:

```text
Timeout/DeadlineExceeded
```

rather than:

```text
continue indefinitely
```

---

# 115. Testing Idempotency

Example:

```text
Release ID:
rel-payment-prod-1.4.2
```

Run twice.

Expected:

```text
first -> create
second -> existing/reconcile
```

not:

```text
two deployments
```

---

# 116. Testing Reconciliation

Scenario:

```text
POST timed out
```

Then:

```text
GET deployment -> exists
```

Expected:

```text
continue monitoring
```

not:

```text
POST again
```

---

# 117. Testing Rollback

Scenario:

```text
deployment health = degraded
```

Expected:

```text
rollback invoked
```

Then:

```text
previous version healthy
```

Expected:

```text
rollback successful
```

---

# 118. Testing Rollback Failure

Scenario:

```text
deployment failed
rollback started
rollback verification failed
```

Expected:

```text
critical alert
manual intervention state
```

Not:

```text
infinite rollback loop
```

---

# 119. Testing Security Gates

Example:

```text
Trivy critical vulnerabilities = 1
```

Expected:

```text
deployment blocked
```

Test:

```text
security policy
```

separately from:

```text
scanner API client
```

---

# 120. Testing Least Privilege

Integration tests can verify that the automation identity has:

```text
required permissions
```

and does not rely on:

```text
admin privileges
```

For example:

```text
can get deployment = yes
can update deployment = maybe/no
```

depending on design.

---

# 121. Smoke Tests

A smoke test answers:

> "Is the basic system working?"

Example:

```text
API reachable
Authentication works
Basic endpoint returns expected response
```

Run after deployment.

---

# 122. Regression Tests

A regression test protects previously fixed behavior.

Example:

```text
Previously:
429 handling was broken

Fix:
Retry-After implemented

Regression test:
429 -> retry according to policy
```

---

# 123. Integration Test Environment

For DevOps projects:

```text
Test Kubernetes namespace
Test AWS account/resources
Test GitHub repository
Test Jenkins job
Test ArgoCD application
```

Avoid running integration tests against production.

---

# 124. Ephemeral Test Environments

A stronger strategy:

```text
CI
 |
 v
Create temporary environment
 |
 v
Run tests
 |
 v
Destroy environment
```

Benefits:

```text
Isolation
Repeatability
Reduced shared-state problems
```

Terraform can provision the environment where appropriate.

---

# 125. Test Environment Strategy

Possible:

```text
Unit
  ↓
Local

Integration
  ↓
Shared test environment

E2E
  ↓
Ephemeral/staging environment

Production
  ↓
Smoke/verification tests
```

---

# 126. Test Data

Avoid real production data.

Use:

```text
Synthetic data
Fixtures
Factories
Sanitized datasets
```

This protects:

```text
Security
Privacy
Compliance
```

---

# 127. Test Data Factories

Example:

```python
def make_release(
    service="payment",
    environment="staging",
    version="1.4.2"
):
    return {
        "service": service,
        "environment": environment,
        "version": version
    }
```

Then:

```python
def test_release(make_release):
    ...
```

A fixture can provide the factory.

---

# 128. Testing Environment Validation

```python
def validate_environment(environment):

    allowed = {
        "dev",
        "staging",
        "prod"
    }

    if environment not in allowed:
        raise ValueError(
            "Invalid environment"
        )
```

Tests:

```python
@pytest.mark.parametrize(
    "environment",
    ["dev", "staging", "prod"]
)
def test_valid_environment(environment):

    validate_environment(environment)
```

And:

```python
def test_invalid_environment():

    with pytest.raises(ValueError):
        validate_environment("production2")
```

---

# 129. Production Protection Test

A particularly important DevOps test:

```text
Can a staging release accidentally target prod?
```

Test:

```text
Invalid environment mapping
```

Expected:

```text
Deployment blocked
```

This is a safety-critical test.

---

# 130. Test Environment-to-Account Mapping

Example:

```text
dev     -> AWS account A
staging -> AWS account B
prod    -> AWS account C
```

Test:

```text
environment=prod
```

must never resolve to:

```text
staging account
```

Configuration mapping deserves automated tests.

---

# 131. Test Namespace Mapping

Example:

```text
payment/staging -> namespace payment-staging
payment/prod    -> namespace payment-prod
```

Verify:

```text
correct namespace
```

before executing deployment operations.

---

# 132. Test Image Mapping

Given:

```text
service=payment
version=1.4.2
```

expected:

```text
ECR repository=payment
image tag=1.4.2
```

Test the mapping separately.

---

# 133. Test GitOps Mapping

Given:

```text
service=payment
environment=prod
```

expected:

```text
GitOps path:
environments/prod/payment
```

Test configuration logic before actual Git operations.

---

# 134. Test ArgoCD Mapping

Given:

```text
payment
prod
```

expected:

```text
ArgoCD application:
payment-prod
```

Again:

```text
test configuration
```

before:

```text
real API call
```

---

# 135. Why Configuration Testing Matters

Many production failures are not Python syntax errors.

They are:

```text
Wrong environment
Wrong URL
Wrong namespace
Wrong repository
Wrong application
Wrong credential
Wrong threshold
```

Configuration tests prevent these classes of mistakes.

---

# 136. Test API URLs

Validate:

```text
base URL
scheme
path
environment
```

Avoid accidental:

```text
prod URL
```

in:

```text
staging tests
```

---

# 137. Test Credential Selection

Example:

```text
staging -> staging credential
prod -> production credential
```

The test should verify:

```text
environment
credential identity
```

are correctly mapped.

Never assert or print secret values.

---

# 138. Test Logging Safety

A useful security test can ensure logs do not contain:

```text
Authorization header
API token
Password
Private key
```

Use redaction functions and test them explicitly.

---

# 139. Test Audit Events

When deployment starts:

```text
release_started
```

When completed:

```text
release_completed
```

When rollback occurs:

```text
rollback_started
rollback_completed
```

Tests can verify that required audit events are emitted.

---

# 140. Test Metrics

Verify metrics update for:

```text
success
failure
retry
rollback
```

For unit tests, test the metric-producing logic rather than depending on a live Prometheus server.

---

# 141. Test Error Classification

Given:

```text
401
```

expected:

```text
AuthenticationError
```

Given:

```text
403
```

expected:

```text
AuthorizationError
```

Given:

```text
429
```

expected:

```text
RateLimitError
```

Given:

```text
503
```

expected:

```text
TransientAPIError
```

This creates predictable workflow behavior.

---

# 142. Test API Client Contract

For each API client verify:

```text
Method
URL
Headers
Payload
Timeout
Response handling
Error mapping
```

Do not test only:

```text
return value
```

---

# 143. Test Request Construction

Example:

```python
def build_image_url(
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
def test_image_url():

    result = build_image_url(
        "registry.example.com",
        "payment",
        "1.4.2"
    )

    assert result == (
        "registry.example.com/"
        "payment:1.4.2"
    )
```

---

# 144. Test Payload Construction

Example:

```python
def build_deployment_payload(
    version
):
    return {
        "version": version
    }
```

Test:

```python
def test_payload():

    payload = build_deployment_payload(
        "1.4.2"
    )

    assert payload["version"] == "1.4.2"
```

---

# 145. Why Separate Request Construction?

It makes:

```text
Business logic
```

testable without:

```text
real HTTP
```

This leads to simpler unit tests.

---

# 146. Test Response Parsing Separately

Example:

```python
def parse_application(data):

    return {
        "sync": data["status"]["sync"]["status"],
        "health": data["status"]["health"]["status"]
    }
```

Test it with sample API responses.

This is much easier than testing everything through a live API.

---

# 147. Golden/Sample Responses

Keep sanitized API response samples:

```text
tests/fixtures/
├── argocd_application.json
├── github_workflow.json
├── kubernetes_deployment.json
└── prometheus_response.json
```

Use them to test parsers.

---

# 148. Fixture Files

Example:

```python
import json
from pathlib import Path


def load_fixture(name):

    path = Path(
        "tests/fixtures"
    ) / name

    return json.loads(
        path.read_text()
    )
```

This creates reusable test data.

---

# 149. Avoid Overly Large Fixtures

A fixture should contain:

```text
minimum data required
```

unless the test specifically verifies:

```text
full response
```

Smaller fixtures are easier to understand.

---

# 150. Test Schema Changes

If API response changes:

```text
old:
status

new:
state
```

tests should fail clearly.

This is useful because:

```text
silent schema changes
```

can cause incorrect deployment decisions.

---

# 151. Contract Tests

Contract testing verifies:

```text
client expectation
```

against:

```text
provider API behavior
```

Example:

```text
ArgoCD client expects:
status.sync.status
```

If provider changes the response:

```text
contract test fails
```

before production.

---

# 152. API Version Tests

If using:

```text
/v1
```

test:

```text
expected v1 response
```

When migrating to:

```text
/v2
```

run compatibility tests.

---

# 153. Test Backward Compatibility

For automation platforms, breaking an API client can break:

```text
entire deployment pipeline
```

Therefore test:

```text
old response
new response
```

where compatibility is required.

---

# 154. Test Pagination

Example API:

```text
page 1 -> 100 items
page 2 -> 100 items
page 3 -> 20 items
```

Expected:

```text
220 items
```

Test that all pages are processed.

---

# 155. Test Pagination Failure

Scenario:

```text
page 1 = success
page 2 = 503
page 3 = never reached
```

Expected:

```text
retry page 2
```

or:

```text
fail safely
```

depending on policy.

---

# 156. Test Rate Limit Across Pages

If pagination triggers:

```text
429
```

the client should:

```text
backoff
retry
continue from current cursor
```

not:

```text
restart entire operation unnecessarily
```

---

# 157. Test Eventual Consistency

Scenario:

```text
create resource
GET -> 404
GET -> 404
GET -> 200
```

Expected:

```text
bounded polling
success
```

if the API contract documents eventual consistency.

---

# 158. Test Polling

Given:

```text
queued
running
running
success
```

Expected:

```text
4 states observed
final result = success
```

Also test:

```text
queued
running
failed
```

---

# 159. Test Polling Timeout

Given:

```text
running
running
running
...
```

Expected:

```text
deadline exceeded
```

This prevents:

```text
CI job hangs
```

---

# 160. Test Cancellation

If the workflow supports:

```text
cancel
```

test:

```text
running
 ↓
cancel request
 ↓
cancelled
```

Also test:

```text
cancel request timeout
```

and:

```text
already completed
```

---

# 161. Test Concurrency

Example:

```text
Two releases
same service
same environment
```

Expected:

```text
one acquires lock
other waits/rejects
```

depending on policy.

This belongs primarily in integration testing because real concurrency behavior often depends on the lock/store implementation.

---

# 162. Test Race Conditions

Simulate:

```text
Release A reads version 1
Release B changes version 2
Release A writes version 3
```

Without optimistic concurrency:

```text
B's change may be overwritten
```

Test:

```text
ETag / version check
```

when supported.

---

# 163. Test Distributed Lock Expiry

If locks have TTL:

```text
lock acquired
worker crashes
TTL expires
new worker acquires
```

Verify:

```text
no permanent deadlock
```

and ensure stale workers cannot safely continue mutating shared state after losing ownership.

---

# 164. Test CI Environment

CI should execute:

```bash
pytest -m unit
```

first.

Then:

```bash
pytest -m integration
```

in a controlled environment.

---

# 165. Jenkins pytest Stage

Conceptual:

```text
stage("Unit Tests") {
    sh "pytest -m unit -v"
}
```

The exact Jenkinsfile syntax depends on your pipeline.

---

# 166. GitHub Actions pytest Stage

Conceptual:

```yaml
- name: Run tests
  run: pytest -m unit -v
```

Add:

```text
coverage
artifact upload
test result reporting
```

as needed.

---

# 167. Test Failure Must Fail CI

If pytest returns:

```text
exit code != 0
```

the CI stage should fail.

Do not use:

```bash
pytest || true
```

for mandatory tests.

That can hide failures.

---

# 168. Allow Failure Carefully

Some jobs may be informational:

```text
experimental test
non-blocking compatibility check
```

If allowed to fail:

```text
explicitly document it
```

Do not hide failures accidentally.

---

# 169. Parallel Test Execution

Large suites can use:

```bash
pytest -n auto
```

with the appropriate pytest parallel-execution plugin installed.

But tests must be designed for:

```text
parallel isolation
```

---

# 170. Parallel Test Problems

Tests may fail when parallelized because they share:

```text
files
ports
database
namespace
resources
environment variables
```

Fix:

```text
isolation
unique resources
fixtures
cleanup
```

---

# 171. Test Runtime Optimization

Improve speed by:

```text
More unit tests
Fewer unnecessary E2E tests
Reuse expensive read-only setup safely
Parallelize independent tests
Avoid real network in unit tests
```

---

# 172. Test Suite Layers in CI

Recommended:

```text
PR:
unit

Merge:
unit + integration

Release:
unit + integration + security + smoke

Production verification:
smoke + health + metrics + logs
```

Exact gates should reflect your organization's risk model.

---

# 173. Pull Request Testing

A PR should ideally validate:

```text
Code quality
Unit tests
Relevant integration tests
Security checks
```

before merge.

---

# 174. Deployment Testing

After deployment:

```text
Smoke test
Health check
Metrics verification
Log verification
```

This is different from:

```text
pre-deployment unit testing
```

---

# 175. Test Before and After Deployment

### Before

```text
Code
Unit
Integration
Security
Artifact
```

### After

```text
Kubernetes
Health
Metrics
Logs
Business behavior
```

Together:

```text
confidence
```

---

# 176. Production Smoke Test

Example:

```python
def test_health_endpoint():

    response = requests.get(
        health_url,
        timeout=5
    )

    assert response.status_code == 200
```

A real production smoke test should use:

```text
approved endpoint
safe credentials
safe test behavior
```

---

# 177. Avoid Dangerous Production Tests

Do not run tests that:

```text
Create real customer data
Delete resources
Trigger payments
Modify production configuration
Restart production workloads
```

unless explicitly designed and authorized as controlled tests.

---

# 178. Safe Production Verification

Prefer:

```text
GET health
GET version
GET metrics
Read-only API
Synthetic transaction
```

where approved.

---

# 179. Synthetic Monitoring

A synthetic test can perform:

```text
Login
Browse
Create test transaction
Verify result
Cleanup
```

using:

```text
synthetic account
```

This validates actual user flow.

---

# 180. Testing Release Verification

A complete test:

```text
Deployment
 |
 v
Kubernetes ready
 |
 v
Health endpoint
 |
 v
Prometheus healthy
 |
 v
ELK no critical errors
 |
 v
Release success
```

Failure:

```text
any mandatory signal fails
```

then:

```text
policy
```

decides whether to:

```text
retry
pause
rollback
```

---

# 181. Test Policy Separately

Do not mix:

```text
API call
```

with:

```text
rollback decision
```

Test policy independently.

Example:

```python
def release_is_healthy(
    kubernetes,
    application,
    metrics
):
    return (
        kubernetes
        and application
        and metrics
    )
```

Then test combinations.

---

# 182. Policy Matrix Testing

```text
Kubernetes | App | Metrics | Result
-----------+-----+---------+--------
Healthy    | OK  | OK      | Pass
Healthy    | Bad | OK      | Fail
Healthy    | OK  | Bad     | Fail
Bad        | OK  | OK      | Fail
```

This is ideal for parametrized tests.

---

# 183. Parametrized Policy Test

```python
@pytest.mark.parametrize(
    "k8s,app,metrics,expected",
    [
        (True, True, True, True),
        (True, False, True, False),
        (True, True, False, False),
        (False, True, True, False),
    ]
)
def test_release_policy(
    k8s,
    app,
    metrics,
    expected
):

    assert (
        release_is_healthy(
            k8s,
            app,
            metrics
        )
        == expected
    )
```

---

# 184. Testing Configuration Validation

Configuration should fail fast.

Example:

```python
def validate_config(config):

    required = [
        "argocd_url",
        "namespace",
        "environment"
    ]

    for key in required:
        if key not in config:
            raise ValueError(
                f"Missing {key}"
            )
```

Test:

```python
def test_missing_config():

    with pytest.raises(ValueError):
        validate_config({})
```

---

# 185. Test Production Guardrails

Example:

```python
def validate_target(
    environment,
    allow_production
):

    if (
        environment == "prod"
        and not allow_production
    ):
        raise PermissionError(
            "Production deployment blocked"
        )
```

Test:

```python
def test_prod_requires_approval():

    with pytest.raises(
        PermissionError
    ):
        validate_target(
            "prod",
            allow_production=False
        )
```

---

# 186. Test Dry Run

Dry run should:

```text
Validate
Plan
Do not mutate
```

Test:

```text
Git commit not created
ArgoCD sync not called
Kubernetes mutation not performed
```

Later mocking techniques will make these tests easier.

---

# 187. Test Rollback Guardrail

Rollback should not target an unknown version.

Example:

```python
def validate_rollback(
    current,
    previous
):

    if not previous:
        raise ValueError(
            "No known-good version"
        )
```

Test:

```python
def test_rollback_requires_previous():

    with pytest.raises(ValueError):
        validate_rollback(
            "1.4.2",
            None
        )
```

---

# 188. Test Auditability

Every state-changing operation should produce enough metadata for auditing.

Test that:

```text
release ID
service
environment
version
result
```

are present in the event.

Never assert on secret values.

---

# 189. Test Logging Redaction

Example:

```python
def redact_token(message):
    return message.replace(
        "secret-token",
        "[REDACTED]"
    )
```

Test:

```python
def test_token_redaction():

    result = redact_token(
        "token=secret-token"
    )

    assert "secret-token" not in result
    assert "[REDACTED]" in result
```

---

# 190. Test Error Context

A production exception may contain:

```text
status_code
request_id
operation
service
environment
```

Test that these fields are preserved.

This is useful during incidents.

---

# 191. Test Error Chaining

If:

```python
except TimeoutError as exc:
    raise APIError(...) from exc
```

test that:

```text
__cause__
```

contains the original exception when this behavior is important to the error contract.

---

# 192. Test CLI Exit Codes

Example:

```text
success -> 0
failure -> non-zero
```

CI depends on this behavior.

Test:

```text
successful workflow
failed workflow
validation failure
```

---

# 193. Test Command-Line Arguments

If using `argparse`:

```text
--service
--environment
--version
--dry-run
```

test:

```text
valid input
missing input
invalid environment
invalid version
```

---

# 194. Test Environment Variables

Use controlled test configuration rather than modifying the developer's real environment.

Test:

```text
variable present
variable missing
invalid value
```

For secrets:

```text
never print
never assert secret contents in logs
```

---

# 195. Testing External API Dependencies

Do not make every unit test call:

```text
GitHub
AWS
Jenkins
ArgoCD
Kubernetes
```

This causes:

```text
Slow tests
Network failures
Credential dependency
Rate limits
Flakiness
Cost
```

Instead:

```text
Unit tests -> controlled responses
Integration -> controlled test environment
E2E -> real integrated environment
```

---

# 196. Test Boundary Definition

A useful rule:

```text
Unit test:
your logic

Integration test:
your logic + dependency

E2E:
complete workflow
```

This helps keep the test suite maintainable.

---

# 197. What Belongs in Unit Tests?

Good candidates:

```text
Validation
Payload construction
Response parsing
Error classification
Retry policy
Threshold calculations
Release policy
Environment mapping
Rollback decision
```

---

# 198. What Belongs in Integration Tests?

Good candidates:

```text
Real GitHub API sandbox
Real Kubernetes test cluster
Real AWS test resources
Real ArgoCD test application
Authentication
API schema compatibility
```

---

# 199. What Belongs in E2E Tests?

Examples:

```text
Commit
 ↓
CI
 ↓
Image
 ↓
GitOps
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
Verification
```

E2E tests are expensive, so keep them focused.

---

# 200. Test Pyramid for Your DevOps Project

Recommended concept:

```text
                E2E
               /---\
              /     \
             /   5   \
            /---------\
           / Integration\
          /      20      \
         /---------------\
        /      Unit       \
       /       75         \
      /-------------------\
```

The exact percentages are not a strict rule.

The principle is:

```text
Many fast tests
+
fewer expensive tests
```

---

# 201. Testing Philosophy

Do not ask:

> "How can I get 100% coverage?"

Ask:

> "What failures would hurt production, and have I tested them?"

Examples:

```text
Wrong environment
Wrong image
Expired token
403
429
503
Timeout
Duplicate request
Unknown state
Rollback failure
```

These are high-value tests.

---

# 202. Production Test Checklist

```text
[ ] Happy path
[ ] Validation failure
[ ] Authentication failure
[ ] Authorization failure
[ ] Rate limit
[ ] Server error
[ ] Timeout
[ ] Network error
[ ] Malformed response
[ ] Missing fields
[ ] Business failure
[ ] Retry exhaustion
[ ] Unknown state
[ ] Idempotency
[ ] Reconciliation
[ ] Rollback
[ ] Rollback failure
[ ] Concurrency
[ ] Environment protection
[ ] Secret redaction
[ ] Exit codes
```

---

# 203. Interview Questions

## Q1. What is pytest?

pytest is a Python testing framework that provides:

```text
Simple assertions
Fixtures
Parametrization
Markers
Plugins
Readable test output
```

It is widely used for unit, integration, and automation testing.

---

## Q2. Why use pytest instead of `unittest`?

A good answer:

```text
pytest provides concise test syntax,
powerful fixtures,
parametrization,
rich failure output,
and a strong plugin ecosystem.
```

`unittest` is still part of Python's standard library and remains useful, so this is not about one being universally correct.

---

## Q3. What is a fixture?

A fixture provides reusable:

```text
setup
test data
dependencies
cleanup
```

for tests.

---

## Q4. What is `conftest.py`?

It is a pytest configuration/test-support file commonly used for:

```text
shared fixtures
hooks
test configuration
```

Fixtures defined there can be automatically available to tests in the appropriate directory scope.

---

## Q5. What is parametrization?

It allows one test function to run against multiple inputs.

Example:

```python
@pytest.mark.parametrize(
    "status",
    [200, 201, 202]
)
def test_success(status):
    assert 200 <= status < 300
```

---

## Q6. What are pytest markers?

Markers categorize tests.

Examples:

```text
unit
integration
e2e
slow
kubernetes
aws
```

Run:

```bash
pytest -m integration
```

---

## Q7. How do you test exceptions?

Use:

```python
with pytest.raises(ValueError):
    function()
```

---

## Q8. What is the difference between unit and integration testing?

Unit:

```text
isolated
fast
no real external dependency
```

Integration:

```text
multiple components
real or controlled dependency
slower
```

---

## Q9. Should unit tests call AWS?

Normally no.

Unit tests should isolate application logic.

AWS API behavior belongs in:

```text
integration tests
```

or:

```text
contract tests
```

depending on the purpose.

---

## Q10. How do you test Kubernetes automation?

Separate:

```text
business logic
```

from:

```text
Kubernetes API calls
```

Unit-test the logic with controlled objects/responses.

Use an actual test cluster for integration testing.

---

# 204. Scenario Interview — Deployment Script

### Question

You wrote a Python script that deploys to Kubernetes. How would you test it?

### Strong Answer

I would use multiple layers.

### Unit tests

Test:

```text
Manifest construction
Validation
Namespace selection
Image selection
Rollout policy
Error classification
```

### Integration tests

Use:

```text
test Kubernetes namespace/cluster
```

to verify:

```text
Authentication
RBAC
API behavior
Resource creation
Rollout
```

### E2E

Test:

```text
CI
 →
artifact
 →
GitOps
 →
ArgoCD
 →
Kubernetes
 →
health verification
```

---

# 205. Scenario Interview — API Timeout

### Question

How would you test timeout handling?

Answer:

```text
Simulate timeout
Verify exception classification
Verify retry policy
Verify maximum attempts
Verify deadline
Verify final failure state
```

The important thing is proving:

```text
automation does not hang
```

---

# 206. Scenario Interview — 503

### Question

How would you test that the client retries 503?

Expected scenario:

```text
503
503
200
```

Then verify:

```text
3 attempts
final success
```

Also test:

```text
503
503
503
503
```

and verify:

```text
retry exhausted
```

---

# 207. Scenario Interview — 429

### Question

How would you test rate-limit handling?

Test:

```text
429
Retry-After
200
```

Verify:

```text
backoff
retry
success
```

Also verify that:

```text
retry count is bounded
```

---

# 208. Scenario Interview — Production Protection

### Question

How do you test that a staging pipeline cannot deploy to production accidentally?

Test:

```text
environment=staging
```

maps only to:

```text
staging configuration
```

and:

```text
environment=prod
```

requires:

```text
production authorization/approval
```

Also test:

```text
account ID
namespace
cluster
ArgoCD application
```

mapping.

---

# 209. Scenario Interview — Rollback

### Question

How would you test automated rollback?

Scenario:

```text
New release
 ↓
Kubernetes ready
 ↓
High error rate
 ↓
Rollback
 ↓
Previous version
 ↓
Healthy
```

Verify:

```text
rollback triggered
correct previous version selected
ArgoCD reconciled
Kubernetes recovered
metrics recovered
final state recorded
```

---

# 210. Scenario Interview — Unknown State

### Question

The deployment API timed out. How do you test this scenario?

I would simulate:

```text
POST -> timeout
GET -> operation exists/running
```

Expected:

```text
reconciliation
```

Then:

```text
GET -> success
```

Expected:

```text
release success
```

This verifies that the automation does not create a duplicate deployment.

---

# 211. Senior-Level Question

## How would you test a production deployment orchestrator?

I would test it at multiple levels:

```text
Unit
 ↓
API client behavior
 ↓
Integration
 ↓
CI/CD + Kubernetes + AWS
 ↓
E2E
 ↓
Production smoke/verification
```

I would prioritize failure scenarios:

```text
Timeout
429
503
401
403
Duplicate request
Partial failure
Unknown state
Rollback failure
```

I would also test:

```text
security boundaries
environment mappings
RBAC
secret redaction
audit
exit codes
```

---

# 212. Senior-Level Question — Test Reliability

## How do you prevent flaky tests?

I would:

```text
Remove unnecessary sleeps
Control time
Isolate test data
Avoid shared mutable state
Use deterministic inputs
Mock external dependencies in unit tests
Use dedicated integration environments
Clean up resources
```

For eventual consistency:

```text
poll with bounded timeout
```

rather than:

```text
sleep(30)
```

---

# 213. Senior-Level Question — Test Pyramid

## Why not run only E2E tests?

Because E2E tests are:

```text
Slow
Expensive
More fragile
Harder to debug
Dependency-heavy
```

Unit tests provide:

```text
fast feedback
```

Integration tests provide:

```text
real dependency confidence
```

E2E tests validate:

```text
complete workflows
```

We need all three.

---

# 214. Senior-Level Question — Coverage

## Is 100% coverage enough?

No.

Coverage measures:

```text
executed code
```

not:

```text
correctness
```

A strong test strategy focuses on:

```text
critical behavior
failure modes
business rules
security
production risks
```

---

# 215. Senior-Level Question — Testing DevOps Automation

## What is different about testing DevOps automation?

DevOps automation interacts with external systems:

```text
AWS
Kubernetes
GitHub
Jenkins
ArgoCD
Prometheus
ELK
```

Therefore testing must consider:

```text
authentication
permissions
network
eventual consistency
rate limits
timeouts
partial failures
idempotency
environment mapping
```

---

# 216. Senior-Level Question — Testing Security

## How do you test secrets are not leaked?

Test:

```text
logging redaction
exception messages
CI output
configuration handling
```

Also use:

```text
secret scanning
```

in CI.

The test environment should use synthetic credentials.

---

# 217. Senior-Level Question — Testing Production Rollback

## How do you prove rollback is safe?

I would verify:

```text
Known-good revision exists
Current revision is identified
Rollback target is valid
Rollback operation is idempotent/safe
Kubernetes becomes healthy
Application health recovers
Metrics recover
Audit event is recorded
```

Then test:

```text
rollback failure
```

as well.

---

# 218. Senior-Level Question — Testing Concurrency

## Two pipelines deploy the same service simultaneously. How do you test this?

Run concurrent workflows against a controlled test environment and verify:

```text
lock behavior
deduplication
state transitions
final desired state
```

The expected policy may be:

```text
one runs
one waits
```

or:

```text
second rejected
```

but it must be deterministic.

---

# 219. Senior-Level Question — CI/CD Test Strategy

## What tests run at each CI/CD stage?

Example:

```text
Developer/PR:
unit + lint

Merge:
unit + integration + security

Build:
container tests + vulnerability scan

Deployment:
smoke tests

Post-deployment:
health + metrics + logs

Production:
controlled E2E/synthetic verification
```

The exact stages depend on risk and release speed.

---

# 220. Practical Project — Pytest DevOps Validator

Build a small project:

```text
devops-validator/
│
├── src/
│   └── validator.py
│
├── tests/
│   └── test_validator.py
│
├── pyproject.toml
└── README.md
```

---

# 221. Validator Example

```python
def validate_release(release):

    required = [
        "service",
        "environment",
        "version"
    ]

    for field in required:

        if field not in release:
            raise ValueError(
                f"Missing {field}"
            )

    if release["environment"] not in {
        "dev",
        "staging",
        "prod"
    }:
        raise ValueError(
            "Invalid environment"
        )

    return True
```

---

# 222. Tests

```python
import pytest

from src.validator import (
    validate_release
)


def test_valid_release():

    release = {
        "service": "payment",
        "environment": "staging",
        "version": "1.4.2"
    }

    assert validate_release(
        release
    )
```

---

# 223. Missing Field Test

```python
def test_missing_service():

    release = {
        "environment": "staging",
        "version": "1.4.2"
    }

    with pytest.raises(
        ValueError,
        match="Missing service"
    ):
        validate_release(
            release
        )
```

---

# 224. Invalid Environment Test

```python
def test_invalid_environment():

    release = {
        "service": "payment",
        "environment": "production",
        "version": "1.4.2"
    }

    with pytest.raises(
        ValueError,
        match="Invalid environment"
    ):
        validate_release(
            release
        )
```

---

# 225. Parameterized Environment Test

```python
@pytest.mark.parametrize(
    "environment",
    [
        "dev",
        "staging",
        "prod"
    ]
)
def test_valid_environments(
    environment
):

    release = {
        "service": "payment",
        "environment": environment,
        "version": "1.4.2"
    }

    assert validate_release(
        release
    )
```

---

# 226. Project — Deployment Policy Tests

Build:

```text
release_policy.py
```

Test:

```text
Kubernetes health
ArgoCD health
Error rate
Latency
Rollback
```

This becomes a foundation for the larger DevOps automation project from the previous section.

---

# 227. Project — API Client Test Suite

Create:

```text
tests/unit/
├── test_github_client.py
├── test_jenkins_client.py
├── test_argocd_client.py
├── test_kubernetes_client.py
├── test_prometheus_client.py
└── test_elk_client.py
```

Each should test:

```text
request construction
success response
4xx
5xx
timeout
parsing
```

Actual HTTP mocking will be covered in:

```text
03-Mocking.md
```

---

# 228. Project — Release Workflow Tests

Create:

```text
tests/unit/test_release_workflow.py
```

Test:

```text
success
build failure
security failure
image failure
GitOps failure
ArgoCD failure
Kubernetes failure
metrics failure
rollback
```

---

# 229. Project — Integration Tests

Create:

```text
tests/integration/
├── test_github.py
├── test_jenkins.py
├── test_kubernetes.py
└── test_argocd.py
```

These should use:

```text
controlled test resources
```

not production.

---

# 230. Project — E2E

Create:

```text
tests/e2e/
└── test_release.py
```

Scenario:

```text
release
 ↓
build
 ↓
artifact
 ↓
GitOps
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
health
```

Keep this suite small.

---

# 231. CI Test Pipeline

A practical pipeline:

```text
                    Git Push
                       |
                       v
                    Lint
                       |
                       v
                  Unit Tests
                       |
                       v
                Security Scan
                       |
                       v
                   Build
                       |
                       v
              Integration Tests
                       |
                       v
                 Test Deploy
                       |
                       v
                  Smoke Test
                       |
                       v
                E2E / Release
```

---

# 232. Test Reports

Useful outputs:

```text
JUnit XML
Coverage XML
HTML coverage
Console output
```

CI systems can consume these artifacts.

For example:

```bash
pytest \
  --junitxml=test-results.xml
```

---

# 233. Coverage Reports

Example:

```bash
pytest \
  --cov=src \
  --cov-report=term-missing \
  --cov-report=xml
```

The exact coverage threshold should be chosen based on project risk and maintained intentionally.

---

# 234. Quality Gate

A CI quality gate may require:

```text
Unit tests = pass
Coverage = acceptable
Security scan = pass
Lint = pass
Integration tests = pass
```

Only then:

```text
Build artifact
```

---

# 235. Test Failure Investigation

When CI fails:

```text
1. Identify failed test
2. Read assertion
3. Check fixture
4. Check environment
5. Check recent code change
6. Reproduce locally
7. Determine unit vs integration issue
8. Fix root cause
9. Re-run targeted test
10. Run complete suite
```

---

# 236. Common pytest Troubleshooting

## `pytest: command not found`

Use:

```bash
python -m pytest
```

If still missing:

```bash
python -m pip install pytest
```

---

# 237. Test Not Discovered

Check:

```text
File name
Function name
Directory
pytest configuration
```

Use:

```bash
pytest --collect-only
```

---

# 238. Import Error

Possible causes:

```text
Package not installed
Wrong working directory
Incorrect package layout
Missing __init__.py in older/package-specific layouts
Bad PYTHONPATH
```

Prefer:

```text
proper package installation
```

instead of:

```text
sys.path hacks
```

---

# 239. Fixture Not Found

Example:

```text
fixture 'client' not found
```

Check:

```text
fixture name
conftest.py location
scope
plugin availability
```

---

# 240. Unexpected Fixture State

Possible cause:

```text
wrong fixture scope
```

Example:

```text
session fixture
```

shares mutable state across tests.

Try:

```text
function scope
```

when isolation is required.

---

# 241. Test Passes Locally, Fails in CI

Check:

```text
Python version
dependency versions
environment variables
credentials
network
filesystem
OS
timezone
locale
test order
parallelism
```

Do not assume CI is wrong.

---

# 242. Flaky Test Troubleshooting

Check:

```text
sleep
timing
race condition
shared state
external API
randomness
resource cleanup
```

Run repeatedly:

```bash
pytest tests/test_x.py -q
```

and, where useful, use plugins/tools designed to detect flaky behavior.

---

# 243. Slow Tests

Use:

```bash
pytest --durations=10
```

This shows slowest tests.

Then determine:

```text
Why is the test slow?
```

Possible:

```text
network
database
Kubernetes
unnecessary setup
sleep
```

---

# 244. Test Order Dependency

If tests pass individually but fail together:

```text
shared state
```

may exist.

Run:

```bash
pytest
```

then investigate:

```text
fixture scope
global state
environment variables
files
ports
resources
```

---

# 245. Randomness

Tests using randomness should control it.

Example concept:

```python
random.seed(42)
```

But prefer deterministic data generation where possible.

For security-sensitive randomness, never replace cryptographically secure randomness with predictable test seeding in production code.

---

# 246. Timezone Problems

CI may run:

```text
UTC
```

while local machine runs:

```text
IST
```

Avoid tests that assume local timezone.

Prefer:

```text
timezone-aware datetimes
```

and explicit timezone expectations.

---

# 247. Environment Variables

CI may have:

```text
CI=true
```

while local does not.

Tests should not unexpectedly depend on:

```text
developer machine variables
```

Use explicit test configuration.

---

# 248. Network Isolation

Unit tests should ideally run without internet.

Benefits:

```text
Fast
Deterministic
Secure
No API rate limits
```

Integration tests can explicitly enable network access.

---

# 249. Test Security Principle

Never make unit tests depend on:

```text
real production credentials
```

or:

```text
production endpoints
```

This is both unreliable and dangerous.

---

# 250. Testing in Containers

You can run tests inside the same type of container used in CI:

```bash
docker build -t devops-tests .
docker run --rm devops-tests
```

This helps reduce:

```text
works locally
fails in CI
```

differences.

---

# 251. Test Docker Image

A CI pipeline can:

```text
Build image
 ↓
Run pytest inside image
 ↓
Scan image
 ↓
Publish
```

This verifies:

```text
runtime dependencies
```

as well as code.

---

# 252. Kubernetes Test Job

For some integration tests:

```text
CI
 ↓
Kubernetes Job
 ↓
pytest
 ↓
result
```

The job should have:

```text
ServiceAccount
RBAC
network access
test configuration
```

only as required.

---

# 253. EKS Testing

For your AWS/EKS environment:

```text
Unit tests
   ↓
CI
   ↓
Integration tests
   ↓
Test EKS namespace/cluster
   ↓
Deployment verification
```

Avoid using production EKS for destructive tests.

---

# 254. Terraform + pytest

Terraform can create:

```text
test VPC
test EKS
test IAM
test resources
```

Then pytest validates:

```text
API behavior
deployment behavior
connectivity
permissions
```

Terraform handles infrastructure.

Python handles validation/automation.

---

# 255. Ansible + pytest

Ansible can configure:

```text
servers
packages
services
```

pytest can validate:

```text
expected state
service health
API response
configuration
```

This is another useful DevOps testing pattern.

---

# 256. Test Automation Repository

A strong portfolio repository could demonstrate:

```text
Terraform
   ↓
AWS/EKS test environment
   ↓
Python
   ↓
pytest
   ↓
Kubernetes
   ↓
ArgoCD
   ↓
Prometheus
```

This connects your Python knowledge to your DevOps experience.

---

# 257. Production Testing Architecture

```text
Developer
    |
    v
GitHub
    |
    v
CI
    |
    +---- Lint
    |
    +---- Unit Tests
    |
    +---- Security
    |
    +---- Integration
    |
    v
Build
    |
    v
Test Environment
    |
    +---- Kubernetes
    +---- ArgoCD
    +---- AWS
    |
    v
E2E
    |
    v
Production
    |
    +---- Smoke
    +---- Health
    +---- Metrics
    +---- Logs
```

---

# 258. Final Mental Model

Do not think:

```text
pytest = test Python functions
```

Think:

```text
pytest
  |
  +-- Validate logic
  +-- Validate API behavior
  +-- Validate failure handling
  +-- Validate DevOps policies
  +-- Validate configuration
  +-- Validate safety guardrails
  +-- Validate release workflows
```

---

# 259. Most Important Testing Principle

> **Test the failure modes that can hurt production, not just the code paths that are easy to test.**

For DevOps automation, high-value tests include:

```text
Wrong environment
Wrong AWS account
Wrong Kubernetes namespace
Wrong image digest
401
403
429
503
Timeout
Unknown deployment state
Duplicate release
Concurrent release
Rollback failure
Security gate failure
Prometheus unavailable
ELK unavailable
```

---

# 260. Pytest Fundamentals Checklist

```text
pytest
[ ] Installation
[ ] Test discovery
[ ] Test naming
[ ] Assertions
[ ] Exceptions
[ ] Fixtures
[ ] Fixture scopes
[ ] conftest.py
[ ] Parametrization
[ ] Markers
[ ] Test classes
[ ] Test organization
[ ] pyproject.toml
[ ] pytest CLI
[ ] Coverage
[ ] Test isolation
[ ] Deterministic tests
[ ] Flaky test diagnosis
[ ] CI integration
```

DevOps testing:

```text
[ ] API testing
[ ] AWS testing
[ ] Kubernetes testing
[ ] ArgoCD testing
[ ] CI/CD testing
[ ] Configuration testing
[ ] Security testing
[ ] Retry testing
[ ] Timeout testing
[ ] Rollback testing
[ ] Environment protection
```

---

# 261. Interview Readiness Checklist

You should be able to explain:

```text
What is pytest?

Why pytest?

How does test discovery work?

What is a fixture?

What is conftest.py?

What are fixture scopes?

What is parametrization?

What are markers?

How do you test exceptions?

How do you run selected tests?

How do you run only integration tests?

How do you measure coverage?

Why isn't 100% coverage enough?

How do you test API clients?

How do you test Kubernetes automation?

How do you test AWS automation?

How do you test retry logic?

How do you test timeouts?

How do you test rollback?

How do you test production guardrails?

How do you prevent flaky tests?

How do you structure tests in CI/CD?
```

---

# 262. Next File

After completing this fundamentals file:

```text
09-Python-Testing/
├── 01-Pytest-Fundamentals.md       ✓
├── 02-Unit-Testing.md
├── 03-Mocking.md
├── 04-Test-Automation.md
└── 05-DevOps-Automation-Testing.md
```

Next:

# `02-Unit-Testing.md`

That file will go deeper into:

```text
Unit testing architecture
        ↓
Testing functions/classes
        ↓
Test isolation
        ↓
Test fixtures
        ↓
Test design
        ↓
Boundary testing
        ↓
Negative testing
        ↓
API/client unit tests
        ↓
AWS/Kubernetes logic tests
        ↓
Retry/error unit tests
        ↓
Test coverage
        ↓
Production-quality unit test suites
```

The following file after that will focus specifically on:

```text
03-Mocking.md
```

including how to safely mock:

```text
Requests
AWS/Boto3
Kubernetes
GitHub
Jenkins
ArgoCD
Prometheus
ELK
time/sleep
environment variables
filesystem
external dependencies
```

---

# 263. Final Learning Flow

```text
09-Python-Testing

01 Pytest Fundamentals
        ↓
02 Unit Testing
        ↓
03 Mocking
        ↓
04 Test Automation
        ↓
05 DevOps Automation Testing
```

The overall goal is:

> **Build Python DevOps automation that is not only functional, but testable, deterministic, secure, and safe to run in CI/CD and production environments.**
