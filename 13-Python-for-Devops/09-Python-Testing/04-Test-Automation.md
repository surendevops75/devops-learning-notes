# Test Automation

## 1. Overview

Test automation is the process of automatically executing tests, collecting results, generating reports, and enforcing quality gates.

For DevOps engineers, test automation is not limited to:

```text
pytest
```

It is part of the complete software delivery lifecycle:

```text
Developer
   |
   v
Git
   |
   v
CI Pipeline
   |
   +--> Unit Tests
   |
   +--> Static Analysis
   |
   +--> Security Tests
   |
   +--> Integration Tests
   |
   +--> Build
   |
   +--> Deploy to Test
   |
   +--> Smoke Tests
   |
   +--> Quality Gate
   |
   v
Production
   |
   +--> Post-Deployment Tests
```

The objective is:

```text
Fast feedback
+
Repeatability
+
Early failure detection
+
Safe releases
```

---

# 2. Why Test Automation Matters in DevOps

Manual testing becomes difficult when deployments happen frequently.

Example:

```text
10 developers
5 services
multiple environments
multiple deployments per day
```

Manually validating every change is:

```text
slow
inconsistent
hard to reproduce
expensive
```

Automated testing makes the validation process repeatable.

---

# 3. Test Automation Lifecycle

A production pipeline may follow:

```text
Commit
 ↓
Lint
 ↓
Unit Tests
 ↓
Coverage
 ↓
Security Tests
 ↓
Build
 ↓
Integration Tests
 ↓
Package
 ↓
Deploy to Test
 ↓
Smoke Tests
 ↓
Acceptance Tests
 ↓
Approval
 ↓
Production
 ↓
Post-Deployment Verification
```

Not every project needs every stage.

The pipeline should match:

```text
risk
architecture
release frequency
business requirements
```

---

# 4. Test Automation Pyramid

```text
                 E2E
                /   \
          Acceptance
             /     \
       Integration
          /       \
        Unit Tests
```

The lower layers should generally contain more tests.

Typical principle:

```text
Many unit tests
Some integration tests
Few expensive E2E tests
```

---

# 5. Unit Tests

Unit tests validate isolated logic.

Examples:

```text
configuration validation
version parsing
deployment policy
security gate
rollback decision
manifest generation
```

They should normally be:

```text
fast
deterministic
isolated
```

---

# 6. Integration Tests

Integration tests validate real interactions.

Examples:

```text
Python -> AWS
Python -> Kubernetes
Python -> PostgreSQL
Python -> Redis
Python -> ArgoCD
Python -> GitHub
```

These are slower than unit tests.

---

# 7. End-to-End Tests

E2E tests validate complete workflows.

Example:

```text
Git commit
 ↓
CI
 ↓
image build
 ↓
ECR
 ↓
GitOps update
 ↓
ArgoCD
 ↓
EKS
 ↓
application
 ↓
health check
```

E2E tests are expensive but valuable for critical paths.

---

# 8. Smoke Tests

Smoke tests answer:

```text
"Is the deployed system basically working?"
```

Typical checks:

```text
application reachable
health endpoint returns 200
database connection works
critical API responds
Kubernetes pods are ready
```

Smoke tests should be:

```text
fast
small
high-value
```

---

# 9. Regression Tests

Regression tests ensure previously working functionality has not broken.

Examples:

```text
login
checkout
payment
order creation
API authentication
deployment workflow
```

Every significant bug should ideally result in a regression test.

---

# 10. Sanity Tests

Sanity testing focuses on a specific change.

Example:

```text
Changed payment API
```

Sanity tests:

```text
payment endpoint
payment validation
payment database interaction
```

---

# 11. Acceptance Tests

Acceptance tests validate behavior against requirements.

Example:

```text
Given:
valid deployment approval

When:
release is triggered

Then:
production deployment proceeds
```

These can be automated using API/UI/framework-based tests depending on the system.

---

# 12. Test Discovery in Pytest

Pytest automatically discovers files commonly named:

```text
test_*.py
*_test.py
```

Example:

```text
tests/
├── test_config.py
├── test_deployment.py
└── test_security.py
```

Run:

```bash
pytest
```

---

# 13. Test Function Discovery

Pytest generally discovers functions beginning with:

```text
test_
```

Example:

```python
def test_deployment_status():
    ...
```

---

# 14. Test Class Discovery

Classes commonly use:

```text
Test*
```

Example:

```python
class TestDeployment:

    def test_success(self):
        ...

    def test_failure(self):
        ...
```

Avoid unnecessary class structures when simple functions are clearer.

---

# 15. Basic Test Command

```bash
pytest
```

More verbose:

```bash
pytest -v
```

Quiet mode:

```bash
pytest -q
```

---

# 16. Run Specific Test File

```bash
pytest tests/test_deployment.py
```

---

# 17. Run Specific Test

```bash
pytest tests/test_deployment.py::test_success
```

---

# 18. Run Tests by Keyword

```bash
pytest -k deployment
```

This is useful during development.

---

# 19. Stop After First Failure

```bash
pytest -x
```

Useful for fast debugging.

---

# 20. Run N Failures

```bash
pytest --maxfail=3
```

This prevents a huge failure list from overwhelming the developer.

---

# 21. Show Print Output

```bash
pytest -s
```

Normally pytest captures stdout.

Use `-s` when troubleshooting output behavior.

---

# 22. Full Verbose Debugging

```bash
pytest -vv
```

Useful when:

```text
parameterized tests
fixture behavior
test selection
```

need closer inspection.

---

# 23. Test Collection

Use:

```bash
pytest --collect-only
```

This helps troubleshoot:

```text
test discovery
```

If a test is not being collected, inspect:

```text
filename
function name
class name
pytest configuration
```

---

# 24. Test Directory Structure

A scalable structure:

```text
project/
├── src/
│   └── automation/
│       ├── clients/
│       ├── policies/
│       ├── workflows/
│       └── models/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── smoke/
│   ├── e2e/
│   ├── fixtures/
│   └── conftest.py
│
├── pytest.ini
├── pyproject.toml
└── requirements.txt
```

---

# 25. `conftest.py`

`conftest.py` is used for shared pytest fixtures and configuration.

Example:

```python
import pytest


@pytest.fixture
def deployment_config():

    return {
        "environment": "staging"
    }
```

Test:

```python
def test_environment(
    deployment_config
):

    assert deployment_config[
        "environment"
    ] == "staging"
```

---

# 26. Fixture Scope

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

Function-scoped fixtures are recreated for each test.

---

# 27. Session Fixtures

Session fixtures run once for the test session.

Example:

```python
@pytest.fixture(scope="session")
def test_config():

    return load_test_config()
```

Useful for expensive setup.

Be careful with shared mutable state.

---

# 28. Module Fixtures

```python
@pytest.fixture(scope="module")
def client():
    ...
```

The fixture is reused within that test module.

Useful when initialization is expensive but isolation between modules is acceptable.

---

# 29. Fixture Teardown

Use `yield`:

```python
@pytest.fixture
def resource():

    resource = create_resource()

    yield resource

    delete_resource(resource)
```

This ensures cleanup.

---

# 30. Test Automation Cleanup

Always consider:

```text
temporary files
containers
test namespaces
cloud resources
database records
locks
```

A failed test should not leave expensive infrastructure behind.

---

# 31. `try/finally` Cleanup

Application code may use:

```python
resource = create()

try:
    use(resource)
finally:
    delete(resource)
```

Test that cleanup occurs for:

```text
success
failure
exception
```

---

# 32. Fixture Factory

A factory fixture can create customized data.

Example:

```python
@pytest.fixture
def make_release():

    def _make_release(
        service="payment",
        version="1.0.0"
    ):
        return {
            "service": service,
            "version": version
        }

    return _make_release
```

Test:

```python
def test_release(make_release):

    release = make_release(
        service="orders"
    )

    assert release["service"] \
        == "orders"
```

---

# 33. Fixture Composition

Fixtures can depend on other fixtures.

Example:

```python
@pytest.fixture
def config():
    return {"environment": "staging"}


@pytest.fixture
def client(config):
    return Client(config)
```

This creates reusable test infrastructure.

---

# 34. Fixture Anti-Pattern

Avoid huge fixtures that create:

```text
AWS
Kubernetes
database
Git
ArgoCD
```

all at once.

Prefer small composable fixtures.

---

# 35. Autouse Fixtures

Example:

```python
@pytest.fixture(autouse=True)
def reset_state():
    reset_test_state()
```

Use sparingly.

Autouse fixtures can make tests difficult to understand because setup happens implicitly.

---

# 36. Pytest Markers

Markers categorize tests.

Examples:

```text
unit
integration
smoke
e2e
slow
security
```

Example:

```python
import pytest


@pytest.mark.integration
def test_kubernetes_connection():
    ...
```

---

# 37. Running Marked Tests

```bash
pytest -m integration
```

Unit:

```bash
pytest -m unit
```

Smoke:

```bash
pytest -m smoke
```

---

# 38. Excluding Tests

Example:

```bash
pytest -m "not e2e"
```

Useful for pull-request pipelines.

---

# 39. Marker Configuration

Configure markers in:

```text
pyproject.toml
```

or:

```text
pytest.ini
```

Example:

```ini
[pytest]
markers =
    unit: fast isolated tests
    integration: external dependency tests
    smoke: deployment smoke tests
    e2e: end-to-end tests
```

This prevents unknown-marker warnings and documents the test taxonomy.

---

# 40. Test Categories

A practical DevOps taxonomy:

```text
unit
integration
contract
security
smoke
regression
e2e
performance
```

Not every project requires every category.

---

# 41. Separate Fast and Slow Tests

Example:

```bash
pytest -m unit
```

runs fast tests.

Then:

```bash
pytest -m "integration or smoke"
```

runs broader validation.

---

# 42. Pull Request Test Strategy

A good PR pipeline might run:

```text
lint
+
unit
+
security
```

before expensive integration tests.

---

# 43. Main Branch Strategy

Main branch may run:

```text
unit
integration
security
build
```

and optionally:

```text
smoke
```

---

# 44. Deployment Pipeline Strategy

After deployment to staging:

```text
smoke
integration
acceptance
```

After production:

```text
post-deployment smoke
critical API checks
health verification
```

---

# 45. Test Configuration

Centralize pytest configuration.

Example:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra"
markers = [
    "unit: unit tests",
    "integration: integration tests",
    "smoke: smoke tests",
    "e2e: end-to-end tests"
]
```

---

# 46. `addopts`

Useful options may include:

```text
-rA
-v
```

Do not blindly place every CLI option into global configuration.

Keep CI-specific behavior in CI scripts when appropriate.

---

# 47. Requirements

Example:

```text
pytest
pytest-cov
pytest-xdist
requests
boto3
kubernetes
```

Pin versions according to your organization's dependency-management policy.

---

# 48. Virtual Environment

Create:

```bash
python -m venv .venv
```

Activate on Linux:

```bash
source .venv/bin/activate
```

Install:

```bash
pip install -r requirements.txt
```

---

# 49. Reproducible Test Environment

A test environment should control:

```text
Python version
dependencies
environment variables
test data
external service versions
```

This reduces:

```text
"It works on my machine"
```

problems.

---

# 50. Python Version Matrix

If your application supports multiple Python versions:

```text
3.10
3.11
3.12
```

run the test suite against each supported version.

Do not test versions you do not officially support merely to increase matrix size.

---

# 51. CI Matrix

Example concept:

```yaml
strategy:
  matrix:
    python-version:
      - "3.11"
      - "3.12"
```

Each matrix job runs:

```bash
pytest
```

---

# 52. Dependency Caching

CI systems can cache:

```text
pip downloads
virtual environments
Docker layers
```

Use caching carefully.

The cache should improve speed, not become a source of stale dependencies.

---

# 53. Clean CI Principle

A reliable pipeline should be able to run from a clean environment.

Avoid tests that depend on:

```text
developer machine files
local credentials
local AWS configuration
previous test state
```

---

# 54. Environment Variables in CI

Example:

```text
ENVIRONMENT=test
AWS_REGION=ap-south-1
```

Use CI secret management for sensitive values.

Never hard-code credentials in:

```text
test files
YAML
Dockerfiles
Git repositories
```

---

# 55. Secret Management

For CI:

```text
GitHub Actions Secrets
Jenkins Credentials
Vault
cloud identity federation
```

Use the appropriate mechanism.

Prefer short-lived credentials and workload identity/federation over long-lived static keys where supported.

---

# 56. Testing Without Cloud Credentials

Most unit tests should run:

```text
without AWS credentials
```

If they require AWS credentials, you may have accidentally coupled unit tests to infrastructure.

---

# 57. Integration Credentials

Integration tests may need controlled credentials.

Use:

```text
dedicated test account
least privilege
short-lived credentials
```

Never production credentials.

---

# 58. Test Environment Isolation

A good integration environment might use:

```text
test AWS account
test EKS cluster
test namespace
test database
test Git repository
test ArgoCD
```

---

# 59. Kubernetes Test Namespace

Instead of creating an entire cluster for every test:

```text
test namespace
```

can isolate workloads when cluster-level behavior is not under test.

Example:

```bash
kubectl create namespace test-automation
```

Automation should clean it up afterward.

---

# 60. Namespace Cleanup

Use:

```bash
kubectl delete namespace test-automation
```

or programmatic cleanup.

Be careful when tests run concurrently.

Prefer unique namespaces:

```text
test-<build-id>
```

---

# 61. Kubernetes Test Labels

Apply labels:

```text
test-run=<run-id>
```

This makes cleanup safer.

Example:

```text
test-run=12345
```

Then automation can identify only its own resources.

---

# 62. Ephemeral Test Resources

For integration tests:

```text
create
 ↓
test
 ↓
cleanup
```

Avoid leaving:

```text
pods
services
PVCs
load balancers
```

running indefinitely.

---

# 63. Dockerized Test Environment

A common approach:

```text
CI runner
   |
Docker
   |
Python test container
```

Example Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir \
    -r requirements.txt

COPY . .

CMD ["pytest", "-v"]
```

---

# 64. Docker Test Command

```bash
docker build \
  -t python-tests:local \
  .

docker run \
  --rm \
  python-tests:local
```

---

# 65. Docker Test Benefits

```text
consistent Python environment
isolated dependencies
reproducible execution
easy CI integration
```

---

# 66. Docker Test Limitations

A containerized test environment does not automatically provide:

```text
AWS
Kubernetes
database
ArgoCD
```

You still need:

```text
mocks
test services
or integration infrastructure
```

---

# 67. Docker Compose Integration Tests

For local integration testing:

```text
Python
PostgreSQL
Redis
RabbitMQ
```

can be launched together.

Example architecture:

```text
docker-compose
  |
  +-- app-test
  +-- postgres
  +-- redis
  +-- rabbitmq
```

This is useful for local development.

---

# 68. Testcontainers

Testcontainers can create disposable service containers for tests.

Useful for:

```text
PostgreSQL
Redis
Kafka
RabbitMQ
Elasticsearch
```

Use only when the real dependency behavior adds value.

---

# 69. Kubernetes Test Jobs

CI can run tests as a Kubernetes Job:

```text
CI
 ↓
Kubernetes Job
 ↓
test container
 ↓
pytest
 ↓
results
```

Useful when tests need access to:

```text
cluster networking
service discovery
Kubernetes resources
```

---

# 70. Kubernetes Test Job Example

Conceptually:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: automation-tests
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: tests
          image: python-tests:latest
          command:
            - pytest
            - -v
```

In production CI, use immutable image references rather than mutable `latest` tags.

---

# 71. Test Job Cleanup

Use:

```text
ttlSecondsAfterFinished
```

where supported.

This helps automatically remove completed Jobs.

---

# 72. Kubernetes Test RBAC

If tests access Kubernetes:

```text
ServiceAccount
Role
RoleBinding
```

should grant only required permissions.

Never give test automation:

```text
cluster-admin
```

unless there is a very specific controlled requirement.

---

# 73. Integration Test Security

Use:

```text
least privilege
isolated namespace
dedicated resources
short-lived credentials
```

---

# 74. Test Data Management

Test data should be:

```text
deterministic
safe
repeatable
minimal
```

Avoid using:

```text
production database dumps
real customer information
production secrets
```

---

# 75. Test Data Factory

Create functions:

```python
def make_user():
    return {
        "id": "test-user-1",
        "name": "Test User"
    }
```

Factories reduce duplication.

---

# 76. Unique Test IDs

For integration tests:

```text
test-user-<run-id>
```

can avoid collisions.

Use deterministic IDs when possible for unit tests.

---

# 77. Test Database Strategy

Options:

```text
in-memory database
containerized database
dedicated test database
ephemeral database
```

Choose based on behavior being tested.

---

# 78. Database Cleanup

Common patterns:

```text
transaction rollback
truncate
delete test records
drop schema
destroy database
```

Choose the safest method for your test architecture.

---

# 79. Test Isolation

Tests should ideally not depend on:

```text
test A running before test B
```

Bad:

```text
test_create_user
test_update_user
test_delete_user
```

where the second test depends on the first.

Better:

```text
each test creates its own required state
```

---

# 80. Ordering Tests

Do not rely on test ordering unless there is a strong reason.

If ordering is genuinely required:

```text
document it
```

and consider whether the test design should be changed.

---

# 81. Parallel Testing

Pytest can use:

```text
pytest-xdist
```

Example:

```bash
pytest -n auto
```

This runs tests across available workers.

---

# 82. When Parallel Testing Helps

Useful for:

```text
large unit-test suites
independent tests
CI pipelines
```

---

# 83. Parallel Testing Risks

Tests may fail if they share:

```text
files
ports
database records
Kubernetes resources
environment variables
global state
```

Before enabling parallelism:

```text
make tests isolated
```

---

# 84. Unique Resource Naming

Parallel integration tests should use:

```text
run ID
worker ID
test ID
```

Example:

```text
namespace-test-123-worker2
```

---

# 85. Test Race Conditions

Parallel tests can expose:

```text
shared-state bugs
```

which is useful.

But do not "fix" these failures by adding arbitrary sleeps.

---

# 86. Coverage

Coverage measures which code paths were executed by tests.

Run:

```bash
pytest --cov
```

---

# 87. Coverage by Package

```bash
pytest \
  --cov=src/automation
```

---

# 88. Coverage Report

Terminal report:

```bash
pytest \
  --cov=src/automation \
  --cov-report=term-missing
```

This shows lines that were not covered.

---

# 89. HTML Coverage

```bash
pytest \
  --cov=src/automation \
  --cov-report=html
```

Output:

```text
htmlcov/
```

---

# 90. Coverage Threshold

Example:

```bash
pytest \
  --cov=src/automation \
  --cov-fail-under=80
```

The pipeline fails if coverage is below the configured threshold.

---

# 91. Coverage Is Not Quality

Important:

```text
90% coverage
```

does not automatically mean:

```text
90% good tests
```

A poorly designed test can execute code without validating meaningful behavior.

---

# 92. What Coverage Should Measure

Focus on:

```text
critical business logic
security gates
failure handling
rollback
configuration
production safety
```

---

# 93. Branch Coverage

Line coverage may miss branches.

Example:

```python
if approved:
    deploy()
else:
    block()
```

Both branches should be tested.

Use branch coverage where useful.

---

# 94. Coverage for DevOps Automation

High-value areas:

```text
environment validation
account validation
deployment decisions
rollback decisions
security gates
retry logic
error classification
```

---

# 95. JUnit XML Reports

CI systems often consume JUnit XML.

Example:

```bash
pytest \
  --junitxml=test-results.xml
```

The report contains:

```text
passed
failed
skipped
duration
test names
```

---

# 96. Why JUnit Reports Matter

CI can display:

```text
Test summary
Failed tests
History
Trends
```

without parsing terminal logs manually.

---

# 97. HTML Test Reports

Plugins can generate detailed HTML reports.

Useful for:

```text
local debugging
CI artifacts
test history
```

---

# 98. CI Artifacts

Save:

```text
JUnit XML
coverage HTML
logs
screenshots
test output
failure diagnostics
```

as pipeline artifacts.

---

# 99. Test Artifacts for Kubernetes

When a test fails, collect:

```bash
kubectl get pods -A
kubectl describe pod ...
kubectl logs ...
kubectl get events ...
```

Store the output as CI artifacts.

This dramatically improves troubleshooting.

---

# 100. Automated Failure Diagnostics

A Kubernetes integration test pipeline can do:

```text
pytest
 ↓
failure?
 ↓
kubectl get pods
kubectl get events
kubectl logs
 ↓
save artifacts
```

Only run diagnostic collection when needed if pipeline time matters.

---

# 101. Test Logging

Use structured logs where practical:

```text
timestamp
test
environment
run_id
service
result
```

Avoid secrets.

---

# 102. CI Exit Codes

A CI test command should return:

```text
0 -> success
non-zero -> failure
```

Example:

```bash
pytest
echo $?
```

The CI runner uses the exit status to determine job outcome.

---

# 103. Shell Safety

In CI scripts, consider:

```bash
set -euo pipefail
```

when appropriate.

This helps prevent silent command failures.

Understand how pipelines and expected non-zero commands interact with shell settings before applying globally.

---

# 104. Jenkins Test Automation

Typical Jenkins pipeline:

```text
Checkout
 ↓
Create Python environment
 ↓
Install dependencies
 ↓
Lint
 ↓
Unit tests
 ↓
Coverage
 ↓
Security
 ↓
Integration tests
 ↓
Publish reports
```

---

# 105. Jenkins Stage Example

Conceptually:

```groovy
stage('Unit Tests') {
    steps {
        sh 'pytest -m unit --junitxml=unit.xml'
    }
}
```

Then publish the test results.

---

# 106. Jenkins Coverage

Example concept:

```text
pytest
  |
coverage.xml
  |
Jenkins/quality tooling
```

Coverage can become a quality gate.

---

# 107. Jenkins Integration Tests

A separate stage can run:

```bash
pytest -m integration
```

This stage can use:

```text
test environment
```

rather than production.

---

# 108. Jenkins Smoke Tests

After deployment:

```bash
pytest -m smoke
```

If smoke tests fail:

```text
pipeline fails
```

and rollback may be triggered according to release policy.

---

# 109. Jenkins Post Actions

Useful actions:

```text
archive reports
publish JUnit
archive logs
send notification
cleanup resources
```

---

# 110. GitHub Actions Test Automation

Typical workflow:

```text
push
 ↓
checkout
 ↓
setup Python
 ↓
install dependencies
 ↓
pytest
 ↓
coverage
 ↓
upload reports
```

---

# 111. GitHub Actions Example

Conceptually:

```yaml
- name: Run tests
  run: |
    pytest \
      -m unit \
      --junitxml=test-results.xml
```

---

# 112. GitHub Actions Matrix

Use a matrix for:

```text
Python versions
OS versions
dependency versions
```

only when compatibility requirements justify it.

---

# 113. GitHub Actions Artifacts

Upload:

```text
test-results.xml
coverage.xml
htmlcov/
```

using the workflow's artifact mechanism.

---

# 114. Pull Request Checks

Branch protection can require:

```text
unit tests
security checks
coverage gate
build
```

before merging.

---

# 115. CI Quality Gate

Example:

```text
Unit tests -> PASS
Coverage -> PASS
SonarQube -> PASS
Trivy -> PASS
Integration -> PASS
```

Then:

```text
merge/deploy allowed
```

---

# 116. DevSecOps Test Pipeline

A production-oriented pipeline:

```text
Git push
 ↓
Lint
 ↓
Unit Tests
 ↓
SAST / SonarQube
 ↓
Dependency Scan
 ↓
Build
 ↓
Trivy
 ↓
Integration Tests
 ↓
Artifact
 ↓
Deploy Test
 ↓
Smoke
 ↓
Approval
 ↓
Production
```

---

# 117. Test Before Build

Run lightweight tests before expensive build stages.

Example:

```text
syntax
lint
unit
```

If they fail:

```text
do not build image
```

This saves CI resources.

---

# 118. Test After Build

After image build:

```text
container startup test
image vulnerability scan
application health
```

This catches:

```text
bad Dockerfile
missing dependency
wrong entrypoint
configuration issue
```

---

# 119. Container Test

Example:

```bash
docker run \
  --rm \
  image-under-test \
  pytest
```

Or run a dedicated health command.

---

# 120. Image Smoke Test

Typical flow:

```text
build image
 ↓
run container
 ↓
wait
 ↓
health endpoint
 ↓
stop container
```

---

# 121. Container Health Check

Example:

```bash
curl \
  --fail \
  http://localhost:8080/health
```

Use a bounded timeout.

---

# 122. Kubernetes Deployment Test

Flow:

```text
Build image
 ↓
Push image
 ↓
Deploy test namespace
 ↓
Wait for rollout
 ↓
Run smoke tests
 ↓
Collect diagnostics
 ↓
Cleanup
```

---

# 123. Kubernetes Rollout Test

Example:

```bash
kubectl rollout status \
  deployment/payment \
  -n test
```

Use an explicit timeout.

---

# 124. Kubernetes Test Validation

Check:

```text
desired replicas
ready replicas
pod status
service endpoints
ingress
application health
```

---

# 125. Kubernetes Test Failure Diagnostics

Automatically collect:

```bash
kubectl get pods -n test
kubectl describe pods -n test
kubectl logs -n test --all-containers
kubectl get events -n test
```

Store results as artifacts.

---

# 126. Test Automation with Helm

Pipeline:

```text
helm lint
 ↓
helm template
 ↓
policy validation
 ↓
deploy test
 ↓
smoke
```

---

# 127. Helm Lint

```bash
helm lint ./chart
```

Catches chart issues before deployment.

---

# 128. Helm Template Testing

```bash
helm template \
  payment \
  ./chart
```

Validate rendered YAML.

---

# 129. Kubernetes Manifest Validation

Use tools appropriate to the project, such as:

```text
kubectl --dry-run
schema validation
policy tools
```

Do not rely only on `helm lint`.

---

# 130. Terraform Test Automation

Pipeline can include:

```text
terraform fmt -check
terraform validate
terraform plan
policy checks
```

Unit-test Python code that generates Terraform configuration separately.

---

# 131. Terraform Plan as Test

A plan can detect:

```text
unexpected resources
dangerous changes
destroy actions
```

Use policy checks to prevent unsafe changes.

---

# 132. Terraform Integration Tests

Integration tests can validate:

```text
VPC
security groups
IAM
EKS
ALB
RDS
```

in a dedicated test environment.

Destroy resources afterward.

---

# 133. Ansible Test Automation

Validate:

```text
syntax
lint
inventory
variables
idempotency
```

Then run integration tests against disposable hosts.

---

# 134. Ansible Idempotency Test

Run playbook twice:

```text
first run -> changes
second run -> minimal/no changes
```

This is a high-value automation test.

---

# 135. GitOps Test Automation

Validate:

```text
manifest
image tag
repository
branch
ArgoCD application
```

before committing changes.

---

# 136. ArgoCD Smoke Test

After GitOps deployment:

```text
Synced
+
Healthy
+
ready pods
```

should be verified.

Remember:

```text
Synced != necessarily Healthy
```

---

# 137. ArgoCD Integration Test

A controlled environment can validate:

```text
Git change
 ↓
ArgoCD reconciliation
 ↓
Kubernetes deployment
```

This should not be required for every unit-test run.

---

# 138. Prometheus Test Automation

After deployment:

```text
query error rate
query latency
query availability
```

Then enforce release thresholds.

---

# 139. ELK Test Automation

Use logs to validate:

```text
application started
critical errors absent
expected migration completed
```

Avoid relying on logs as the only health signal.

---

# 140. Multi-Signal Verification

Strong post-deployment validation combines:

```text
ArgoCD health
+
Kubernetes readiness
+
application health
+
Prometheus metrics
+
critical logs
```

No single signal should be trusted blindly for high-risk releases.

---

# 141. Test Automation Timeout

Every external test should have a timeout.

Avoid:

```python
while not ready:
    check()
```

without a bound.

Prefer:

```text
maximum attempts
or
maximum duration
```

---

# 142. Polling Test

Example:

```python
for _ in range(30):

    if is_ready():
        return True

    sleep(2)

return False
```

Unit-test:

```text
ready immediately
ready after retries
never ready
```

---

# 143. Test Retry Policy

Separate:

```text
retryable
non-retryable
```

Examples:

```text
503 -> often retryable
timeout -> often retryable
403 -> usually not retryable
invalid configuration -> not retryable
```

The exact policy depends on the dependency.

---

# 144. Avoid Blind Retries

Bad:

```text
retry every error
```

This can amplify:

```text
load
incidents
rate limits
```

Use:

```text
bounded retries
backoff
jitter where appropriate
error classification
```

---

# 145. Testing Backoff

Verify:

```text
attempt 1 -> delay
attempt 2 -> larger delay
attempt 3 -> larger delay
```

Mock sleep/time.

Do not actually wait.

---

# 146. Test Flakiness

A flaky test:

```text
passes sometimes
fails sometimes
```

without code changes.

Common causes:

```text
time
network
shared state
ordering
randomness
race conditions
external services
```

---

# 147. Flaky Test Detection

If a test intermittently fails:

```text
run repeatedly
```

Example:

```bash
pytest tests/test_x.py \
  --count=20
```

if the relevant plugin is installed.

Do not hide failures by blindly adding retries.

---

# 148. Flaky Test Strategy

When a test flakes:

```text
identify root cause
 ↓
remove nondeterminism
 ↓
isolate state
 ↓
control time
 ↓
control randomness
 ↓
fix race
```

Retries should be a last-resort diagnostic or narrowly justified mechanism.

---

# 149. CI Test Retries

Retries may be appropriate for:

```text
known transient integration infrastructure failures
```

But:

```text
unit tests should normally not require retries
```

If a unit test needs retries, investigate the test.

---

# 150. Test Quarantine

If a known flaky test cannot be immediately fixed:

```text
quarantine
```

but track:

```text
owner
reason
ticket
deadline
```

Do not silently ignore it.

---

# 151. Test Duration Tracking

Track:

```text
test duration
suite duration
stage duration
```

A test suite that grows from:

```text
2 minutes
```

to:

```text
30 minutes
```

needs optimization.

---

# 152. Slow Test Identification

Pytest:

```bash
pytest --durations=10
```

shows the slowest tests.

Use this to find optimization targets.

---

# 153. Test Parallelization Strategy

Split:

```text
unit
integration
e2e
```

and parallelize independent work.

Example:

```text
Unit workers
Integration workers
```

with appropriate environment isolation.

---

# 154. Test Sharding

Large CI environments can divide tests:

```text
Shard 1
Shard 2
Shard 3
Shard 4
```

Each shard runs a subset.

This reduces wall-clock time.

---

# 155. Test Selection

Not every change requires the same tests.

Possible strategy:

```text
Python policy change
 -> unit + relevant integration

Terraform change
 -> Terraform validation + infra tests

Kubernetes manifest change
 -> manifest validation + deployment smoke
```

Use change-aware testing carefully; critical suites should still run at appropriate boundaries.

---

# 156. Contract Testing

Contract tests verify assumptions between services.

Example:

```text
Service A expects:
POST /orders
response:
id
status
```

The contract test verifies the provider still satisfies it.

---

# 157. API Contract Automation

Can validate:

```text
status code
headers
schema
required fields
types
```

This catches breaking API changes.

---

# 158. OpenAPI Validation

For APIs using OpenAPI:

```text
request schema
response schema
```

can be validated automatically.

---

# 159. DevOps API Testing

For Python API automation:

```text
authentication
status codes
timeouts
schema
pagination
rate limits
```

should be tested.

---

# 160. Security Test Automation

Automate:

```text
SAST
SCA
container scanning
secret scanning
IaC scanning
dependency checks
```

Security should be integrated into CI/CD, not performed only before release.

---

# 161. SonarQube Integration

Typical flow:

```text
pytest
 ↓
coverage.xml
 ↓
SonarQube analysis
 ↓
quality gate
```

Coverage can feed the quality platform.

---

# 162. Trivy Integration

Typical flow:

```text
build image
 ↓
trivy image
 ↓
critical/high policy
 ↓
pass/block
```

Configure severity and ignore policies intentionally.

---

# 163. Test Security Failure

If scanner reports:

```text
critical = 1
```

pipeline should follow explicit policy:

```text
block
```

or:

```text
allow with approved exception
```

Never silently ignore.

---

# 164. Test Exceptions

Security exceptions should have:

```text
reason
owner
expiry
scope
approval
```

Automation can verify expiry.

---

# 165. Test Quality Gates

A quality gate can combine:

```text
unit tests
coverage
SAST
dependency scan
container scan
integration tests
```

Example:

```text
ALL PASS
    |
    v
Deploy
```

---

# 166. Quality Gate Failure

If any required gate fails:

```text
deployment blocked
```

The pipeline should clearly identify:

```text
which gate failed
why
where to find details
```

---

# 167. Pipeline Test Result Summary

A good pipeline should show:

```text
Unit:        PASS
Integration: PASS
Security:    PASS
Coverage:    87%
Smoke:       PASS
```

rather than forcing engineers to inspect raw logs.

---

# 168. Test Report Publishing

Publish:

```text
JUnit
coverage
security reports
test logs
Kubernetes diagnostics
```

as CI artifacts.

---

# 169. Test Failure Notification

For important pipelines:

```text
failure
 ↓
Slack/Email/Teams
```

Notification should contain:

```text
pipeline
branch
commit
stage
failure summary
link to logs
```

Do not send secrets or huge logs in notifications.

---

# 170. Test Automation and GitOps

A GitOps pipeline may use:

```text
CI
 ↓
test
 ↓
image
 ↓
manifest update
 ↓
Git
 ↓
ArgoCD
 ↓
EKS
 ↓
smoke
```

Testing should happen both:

```text
before Git change
```

and:

```text
after deployment
```

---

# 171. Pre-Merge Tests

Recommended:

```text
lint
unit
security
manifest validation
```

Fast feedback is important.

---

# 172. Pre-Deployment Tests

Recommended:

```text
integration
artifact validation
security gates
configuration validation
```

---

# 173. Post-Deployment Tests

Recommended:

```text
rollout status
health endpoint
critical API
metrics
logs
```

---

# 174. Production Verification

After production deployment:

```text
deployment
 ↓
wait
 ↓
health
 ↓
metrics
 ↓
critical API
 ↓
success
```

If failure:

```text
rollback/escalate
```

according to release policy.

---

# 175. Automated Rollback

A rollback should not be triggered solely by one noisy signal.

Use a defined policy:

```text
multiple consecutive failures
or
critical health condition
or
explicit hard failure
```

---

# 176. Rollback Test Automation

Test:

```text
healthy -> no rollback
temporary metric spike -> policy response
persistent failure -> rollback
rollback success
rollback failure
```

---

# 177. Canary Test Automation

Flow:

```text
deploy 5%
 ↓
smoke
 ↓
metrics
 ↓
continue
 ↓
25%
 ↓
metrics
 ↓
100%
```

Automate the gates.

---

# 178. Blue/Green Test Automation

Flow:

```text
deploy green
 ↓
validate green
 ↓
switch traffic
 ↓
validate
```

Test both:

```text
switch success
switch blocked
```

---

# 179. Deployment Verification Function

Keep policy pure:

```python
def deployment_is_safe(
    healthy,
    error_rate,
    threshold
):

    return (
        healthy
        and error_rate <= threshold
    )
```

Then unit-test extensively.

External metric retrieval is tested separately.

---

# 180. Test Automation Architecture

```text
                 CI/CD
                   |
                   v
             Test Orchestrator
                   |
       +-----------+-----------+
       |           |           |
      Unit     Integration    E2E
       |           |           |
       v           v           v
    Python      Test Infra   Test Env
    Logic       Services     EKS/etc.
```

---

# 181. Test Orchestrator

A Python test orchestrator may:

```text
create environment
run tests
collect results
collect diagnostics
destroy environment
publish report
```

This is useful for advanced DevOps automation.

---

# 182. Environment Provisioning for Tests

Possible flow:

```text
Terraform
 ↓
test infrastructure
 ↓
integration tests
 ↓
destroy
```

Use separate state and credentials.

---

# 183. Terraform + Pytest Architecture

```text
CI
 |
 +--> terraform init
 |
 +--> terraform apply
 |
 +--> pytest integration
 |
 +--> diagnostics
 |
 +--> terraform destroy
```

Ensure cleanup occurs even when tests fail.

---

# 184. Cleanup Trap

A dangerous pipeline:

```text
terraform apply
 ↓
pytest FAIL
 ↓
pipeline exits
```

Infrastructure remains.

Better:

```text
apply
 ↓
pytest
 ↓
always cleanup
```

using the CI system's post/finally mechanism.

---

# 185. Test Environment TTL

For ephemeral infrastructure, use:

```text
TTL
auto cleanup
scheduled cleanup
```

to protect against abandoned environments.

---

# 186. Cost Control

Automated integration environments can become expensive.

Control:

```text
runtime
instance size
number of environments
cleanup
test frequency
parallelism
```

---

# 187. Test Environment Naming

Use:

```text
project
branch
build
run ID
```

Example:

```text
devops-tests-pr-142-7812
```

This makes ownership obvious.

---

# 188. Test Environment Labels

Apply:

```text
owner=ci
build=7812
purpose=integration-test
```

Labels help:

```text
cleanup
cost tracking
troubleshooting
```

---

# 189. Test Environment Failure

If provisioning fails:

```text
do not run application tests
```

Instead:

```text
collect Terraform logs
collect cloud errors
cleanup partial resources
fail clearly
```

---

# 190. Integration Test Health Check

Before running tests:

```text
cluster reachable
namespace exists
dependencies ready
```

Then start the suite.

---

# 191. Dependency Readiness

Avoid:

```text
sleep 60
```

Prefer:

```text
poll until ready
with timeout
```

Example:

```text
PostgreSQL ready?
Redis ready?
Kubernetes API ready?
```

---

# 192. Readiness Polling

```python
def wait_until_ready(
    check,
    attempts=30
):

    for _ in range(attempts):

        if check():
            return True

    return False
```

Production code should include bounded delays/backoff as appropriate.

---

# 193. Integration Test Retry

Infrastructure can be transient.

Retry:

```text
dependency readiness
```

but do not automatically retry:

```text
assertion failures
```

because those usually represent real defects.

---

# 194. Test Classification

Important distinction:

```text
Infrastructure transient failure
```

vs:

```text
Application test failure
```

CI should make this distinction visible.

---

# 195. Test Failure Categories

Useful categories:

```text
TEST_FAILURE
INFRA_FAILURE
TIMEOUT
CONFIGURATION_FAILURE
SECURITY_FAILURE
ENVIRONMENT_FAILURE
```

This improves incident response.

---

# 196. Test Automation Logging

Include:

```text
test suite
environment
build ID
commit SHA
namespace
cluster
timestamp
```

This makes CI troubleshooting much easier.

---

# 197. Correlation ID

Generate a test run ID:

```text
RUN-2026-00123
```

Use it in:

```text
Kubernetes labels
logs
temporary resources
reports
notifications
```

---

# 198. Test Result Storage

For long-running projects, store:

```text
pass/fail
duration
commit
branch
environment
```

This allows trend analysis.

---

# 199. Test Trends

Track:

```text
pass rate
failure rate
flaky rate
average duration
coverage
```

This helps identify deteriorating test health.

---

# 200. Test Automation Metrics

Useful metrics:

```text
Test Pass Rate
Test Failure Rate
Flaky Test Rate
Mean Test Duration
Coverage
Pipeline Duration
Defect Escape Rate
```

---

# 201. Defect Escape Rate

A useful engineering metric:

```text
production defects
------------------
total releases
```

The goal is not simply:

```text
more tests
```

but:

```text
fewer production defects
```

---

# 202. Test Effectiveness

Ask:

```text
Did the test catch meaningful failures?
```

not only:

```text
Did the test execute?
```

---

# 203. Mutation Testing

Mutation testing changes code intentionally to see whether tests detect it.

Concept:

```text
original:
if approved:

mutated:
if not approved:
```

If tests still pass:

```text
test suite may be weak
```

Use selectively for critical logic.

---

# 204. Test Automation and Code Review

Pull requests should show:

```text
tests added
tests changed
coverage
security
```

Reviewers should ask:

```text
What failure does this test protect against?
```

---

# 205. Regression Test Workflow

When production bug occurs:

```text
incident
 ↓
root cause
 ↓
write regression test
 ↓
fix code
 ↓
test passes
 ↓
merge
```

This prevents recurrence.

---

# 206. Incident-to-Test Example

Production issue:

```text
Kubernetes deployment succeeds
but readiness probe fails
```

Add regression test:

```text
invalid readiness configuration
```

Then ensure:

```text
deployment validation blocks bad configuration
```

---

# 207. Security Regression

Production/security issue:

```text
secret accidentally logged
```

Add test:

```text
secret value not present in logs
```

This converts the incident into permanent protection.

---

# 208. Infrastructure Regression

Issue:

```text
Terraform change destroys resource
```

Add test/policy:

```text
destroy count > 0
```

blocks deployment.

---

# 209. CI Regression

Issue:

```text
pipeline skipped security stage
```

Add pipeline validation that verifies required stages are present.

---

# 210. Kubernetes Regression

Issue:

```text
wrong namespace
```

Add test:

```text
deployment namespace == expected namespace
```

---

# 211. GitOps Regression

Issue:

```text
image tag updated on wrong branch
```

Add test:

```text
branch == expected deployment branch
```

---

# 212. API Regression

Issue:

```text
API response field missing
```

Add:

```text
contract test
```

---

# 213. Test Automation for DevSecOps

A mature pipeline:

```text
Code
 |
 +--> Unit
 |
 +--> SAST
 |
 +--> SCA
 |
 +--> Secret Scan
 |
 +--> Build
 |
 +--> Container Scan
 |
 +--> Integration
 |
 +--> IaC Scan
 |
 +--> Deploy Test
 |
 +--> Smoke
 |
 +--> Approval
 |
 +--> Production
```

---

# 214. Test Automation for Kubernetes

A mature Kubernetes pipeline:

```text
Python/unit
 ↓
Docker build
 ↓
Trivy
 ↓
Helm lint
 ↓
Manifest validation
 ↓
Deploy test namespace
 ↓
Rollout
 ↓
Smoke
 ↓
Prometheus verification
 ↓
Cleanup
```

---

# 215. Test Automation for EKS

Example architecture:

```text
GitHub/Jenkins
      |
      v
Python Test Runner
      |
      +--> AWS APIs
      |
      +--> EKS
      |
      +--> ECR
      |
      +--> ArgoCD
      |
      +--> Prometheus
```

Unit tests mock all external APIs.

Integration tests use the test environment.

---

# 216. Test Automation for Microservices

For services:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

test:

```text
service health
API contracts
critical workflows
dependency failures
```

---

# 217. Microservice Smoke Test

After deployment:

```text
GET /health
```

for each critical service.

Then test:

```text
critical end-to-end path
```

rather than every endpoint.

---

# 218. Microservice Integration

Example:

```text
Orders
  |
  +--> Payment
  |
  +--> Inventory
```

Integration test should validate:

```text
request
response
failure handling
```

using controlled service dependencies where appropriate.

---

# 219. Consumer-Driven Contracts

A consumer can define:

```text
expected API behavior
```

Provider tests verify:

```text
contract remains valid
```

This reduces breaking changes between microservices.

---

# 220. Test Data for Microservices

Use:

```text
synthetic users
synthetic orders
synthetic products
```

with deterministic IDs.

---

# 221. Idempotent Integration Tests

A test should ideally be safe to rerun.

For example:

```text
create test namespace
```

should handle:

```text
already exists
```

or use unique names.

---

# 222. Cleanup on Failure

Use:

```text
fixture finalizers
CI post stages
try/finally
TTL resources
```

to prevent abandoned infrastructure.

---

# 223. Test Automation Failure Troubleshooting

When CI says:

```text
tests failed
```

first determine:

```text
test failure
or
environment failure
```

---

# 224. Step 1 — Read Test Summary

Identify:

```text
failed test
error type
duration
stage
```

---

# 225. Step 2 — Reproduce Locally

Run:

```bash
pytest \
  tests/path/test_x.py::test_y \
  -vv
```

If possible, reproduce with the same:

```text
Python version
dependencies
environment variables
```

---

# 226. Step 3 — Inspect CI Artifacts

Check:

```text
JUnit
coverage
logs
Kubernetes diagnostics
screenshots
```

---

# 227. Step 4 — Check Environment

Verify:

```text
Python
dependencies
AWS credentials
Kubernetes context
namespace
network
DNS
secrets
```

---

# 228. Step 5 — Check External Dependencies

For integration tests:

```text
AWS
EKS
database
ArgoCD
GitHub
Jenkins
Prometheus
```

---

# 229. Step 6 — Determine Flakiness

Run the failing test repeatedly.

If:

```text
passes/fails randomly
```

investigate:

```text
race
time
network
shared state
```

---

# 230. Step 7 — Collect Diagnostics

For Kubernetes:

```bash
kubectl get pods -n test
kubectl get events -n test
kubectl describe pod ...
kubectl logs ...
```

---

# 231. Step 8 — Fix Root Cause

Do not simply:

```text
increase timeout
```

unless the timeout is genuinely too short.

Do not:

```text
add retries
```

to hide a deterministic application failure.

---

# 232. Common CI Failure

```text
ModuleNotFoundError
```

Check:

```text
requirements
virtual environment
Python version
package installation
working directory
```

---

# 233. Common CI Failure

```text
AWS credentials unavailable
```

For unit tests:

```text
remove dependency on real credentials
```

For integration:

```text
configure secure test identity
```

---

# 234. Common CI Failure

```text
Kubernetes connection refused
```

Check:

```text
cluster
context
network
credentials
API endpoint
RBAC
```

---

# 235. Common CI Failure

```text
pytest cannot find tests
```

Check:

```text
test filename
test function
testpaths
working directory
```

Run:

```bash
pytest --collect-only
```

---

# 236. Common CI Failure

```text
coverage gate failed
```

Check:

```text
new untested code
excluded paths
coverage configuration
test selection
```

Do not add meaningless tests solely to increase the percentage.

---

# 237. Common CI Failure

```text
JUnit report missing
```

Check:

```text
pytest command
output path
workspace
artifact upload
```

---

# 238. Common CI Failure

```text
integration tests timeout
```

Determine whether:

```text
application is slow
dependency is unavailable
test is waiting forever
network issue
```

Then fix the correct layer.

---

# 239. Common CI Failure

```text
test passes locally but fails in CI
```

Compare:

```text
Python
OS
dependency versions
environment variables
filesystem
timezone
network
credentials
test order
parallelism
```

---

# 240. Common CI Failure

```text
test passes alone but fails in suite
```

Likely causes:

```text
shared state
fixture leakage
global variables
environment variables
filesystem
database
ordering
```

---

# 241. Common CI Failure

```text
test passes serially but fails parallel
```

Check:

```text
shared resources
ports
namespaces
database records
files
global state
```

---

# 242. Common CI Failure

```text
Docker test works locally but not CI
```

Check:

```text
Docker availability
image architecture
network
credentials
resource limits
registry access
```

---

# 243. Common CI Failure

```text
Kubernetes test leaves resources
```

Check:

```text
cleanup fixture
finally block
CI post stage
resource labels
TTL
```

---

# 244. Common CI Failure

```text
AWS test leaves resources
```

Immediately improve:

```text
cleanup
tags
TTL
test account
resource inventory
```

Costly test leakage is a DevOps operational issue.

---

# 245. Test Automation Observability

Treat CI as a system.

Monitor:

```text
pipeline duration
failure rate
test duration
flaky tests
resource leaks
```

---

# 246. CI Pipeline Dashboard

Useful dashboard metrics:

```text
Build Success Rate
Average Test Duration
Failed Tests
Flaky Tests
Coverage
Security Failures
Deployment Failures
```

---

# 247. Test Ownership

Every critical suite should have:

```text
owner
documentation
failure procedure
environment
```

Avoid:

```text
orphaned tests
```

that nobody understands.

---

# 248. Test Documentation

Document:

```text
how to run
required dependencies
environment variables
integration environment
cleanup
troubleshooting
```

---

# 249. Local Test Command

A good project should make common commands simple:

```bash
pytest -m unit
```

and:

```bash
pytest -m integration
```

---

# 250. Makefile

A DevOps project may provide:

```makefile
test:
	pytest

unit:
	pytest -m unit

integration:
	pytest -m integration

smoke:
	pytest -m smoke
```

This gives developers consistent commands.

---

# 251. Makefile CI

CI can call:

```bash
make unit
```

rather than duplicating long commands in every pipeline.

---

# 252. Task Runner

Alternative tools can provide:

```text
test
lint
security
build
integration
```

commands.

The important point is consistency.

---

# 253. Test Automation as Code

Store:

```text
tests
fixtures
CI config
Dockerfiles
scripts
test environment definitions
```

in Git.

Treat test infrastructure as code.

---

# 254. Version-Controlled Test Environments

Keep:

```text
Dockerfile
Helm chart
Terraform
Kubernetes manifests
pytest config
```

versioned together where appropriate.

---

# 255. Branch Strategy

A practical flow:

```text
feature branch
 ↓
PR tests
 ↓
merge
 ↓
main tests
 ↓
staging
 ↓
smoke
 ↓
production
```

---

# 256. Required PR Checks

Recommended:

```text
unit
lint
security
```

plus relevant tests based on the changed area.

---

# 257. Required Main Checks

Recommended:

```text
unit
integration
security
build
```

---

# 258. Release Checks

Recommended:

```text
artifact verification
deployment
smoke
metrics
approval
```

---

# 259. Production Gate

A release should proceed only when:

```text
required tests pass
required security checks pass
required approvals exist
required environment validation passes
```

---

# 260. Test Automation and GitOps

Git remains the source of desired state.

Tests validate:

```text
configuration
manifest
policy
deployment
runtime
```

ArgoCD performs reconciliation.

Python automation can orchestrate:

```text
validation
verification
diagnostics
```

without bypassing GitOps principles.

---

# 261. Avoid Direct Production Mutation

If production is managed by GitOps:

```text
Python
```

should generally not bypass the desired-state workflow by directly changing resources unless that action is explicitly part of the architecture.

Prefer:

```text
Python
 ↓
Git change
 ↓
ArgoCD
 ↓
Kubernetes
```

for declarative deployments.

---

# 262. Test GitOps Guardrails

Test that:

```text
wrong branch -> blocked
wrong environment -> blocked
invalid manifest -> blocked
security gate -> blocked
approval missing -> blocked
```

---

# 263. Test ArgoCD Verification

After reconciliation:

```text
sync status
health status
operation state
```

should be validated.

---

# 264. Test Kubernetes Verification

Then:

```text
rollout
pods
services
ingress
```

should be validated.

---

# 265. Test Application Verification

Finally:

```text
health API
critical workflow
metrics
logs
```

should be checked.

---

# 266. Multi-Layer Test Strategy

```text
Layer 1:
Python unit tests

Layer 2:
API/client integration tests

Layer 3:
Kubernetes deployment tests

Layer 4:
Application smoke tests

Layer 5:
Production verification
```

---

# 267. Test Automation and SRE Principles

Even when not using an SRE role, good automation should emphasize:

```text
reliability
observability
safe failure
rollback
repeatability
```

---

# 268. Failure Budget

Do not allow tests to become:

```text
always green because failures are ignored
```

A green pipeline should mean:

```text
required validation actually happened
```

---

# 269. Test Skips

Use skips for:

```text
environment-specific tests
unsupported platform
optional dependency
```

But document why.

Do not skip failing tests merely to make CI green.

---

# 270. Expected Failures

If a framework supports expected-failure behavior, use it only for known conditions with clear ownership.

Do not turn real defects into permanent expected failures.

---

# 271. Test Review Checklist

For every new test:

```text
What behavior is tested?
What failure does it prevent?
Is it deterministic?
Does it require external infrastructure?
Can it be unit-level?
Is cleanup required?
Does it expose secrets?
Does it add meaningful coverage?
```

---

# 272. Senior DevOps Interview — Test Automation

### Question

How would you design automated testing for a Python DevOps automation project?

### Answer

```text
I would use a layered test strategy.

Fast unit tests validate pure business logic and use mocks
for external APIs such as AWS, Kubernetes, GitHub, ArgoCD,
and Jenkins.

Integration tests validate real dependency interactions in
isolated environments.

After deployment, smoke and acceptance tests validate the
application.

The CI pipeline would publish JUnit and coverage reports,
enforce security and quality gates, collect diagnostics on
failure, and clean up ephemeral resources.
```

---

# 273. Interview — Test Pyramid

### Question

Why not run only E2E tests?

Answer:

```text
E2E tests are slower, more expensive, more environment
dependent, and harder to troubleshoot.

Unit tests provide fast feedback and isolate failures.
Integration tests validate real dependencies.
E2E tests validate critical user or release workflows.

I use all three at appropriate layers.
```

---

# 274. Interview — CI Pipeline

### Question

What testing stages would you put in a DevOps pipeline?

Answer:

```text
Lint
Unit tests
Coverage
SAST/SCA
Build
Container scan
Integration tests
Deploy to test
Smoke tests
Acceptance tests
Production verification
```

The exact stages depend on the application's risk and architecture.

---

# 275. Interview — Flaky Tests

### Question

How do you handle flaky tests?

Answer:

```text
First I identify the root cause instead of simply retrying.

I check shared state, timing, randomness, network dependency,
parallel execution, resource cleanup, and external services.

For known transient infrastructure failures, a bounded retry
may be acceptable, but unit tests should normally be deterministic.
```

---

# 276. Interview — Coverage

### Question

Is 100% coverage necessary?

Answer:

```text
Not necessarily.

Coverage is a useful signal, not proof of test quality.

I prioritize critical business logic, security gates,
deployment decisions, rollback logic, and failure paths.

I use a reasonable threshold and focus on meaningful assertions.
```

---

# 277. Interview — Integration Environment

### Question

How do you safely run integration tests against AWS?

Answer:

```text
I use a dedicated test account or isolated environment,
least-privilege credentials, ephemeral resources where
possible, unique resource names/tags, bounded test duration,
and guaranteed cleanup.

Production credentials and production resources should not
be used for ordinary integration tests.
```

---

# 278. Interview — Kubernetes Integration Tests

### Question

How would you test Kubernetes automation?

Answer:

```text
Unit tests mock the Kubernetes API client.

Integration tests deploy into a dedicated namespace or
ephemeral cluster.

I wait for readiness using bounded polling, run smoke tests,
collect pod/events/log diagnostics on failure, and clean up
all test resources.
```

---

# 279. Interview — CI Failure

### Question

A test passes locally but fails in Jenkins. What do you check?

Answer:

```text
Python version
dependency versions
environment variables
credentials
filesystem
working directory
network
timezone
test ordering
parallelism
external dependencies
```

Then I reproduce using the same environment and inspect CI artifacts.

---

# 280. Interview — Test Reports

### Question

Why publish JUnit and coverage reports?

Answer:

```text
JUnit gives CI systems structured test results.

Coverage provides visibility into exercised code.

Together they make failures, trends, and quality gates easier
to understand than raw console output.
```

---

# 281. Interview — Smoke Tests

### Question

What should a post-deployment smoke test contain?

Answer:

```text
I keep it small and high-value.

For Kubernetes services I might verify rollout status,
pod readiness, service reachability, health endpoints, and
one or two critical APIs.

For high-risk releases I can additionally verify key metrics
and error signals.
```

---

# 282. Interview — Rollback

### Question

Would you automatically rollback whenever one test fails?

Answer:

```text
Not blindly.

I define release policies that distinguish hard failures,
transient failures, and noisy signals.

Rollback should be based on reliable health signals and
bounded failure conditions.
```

---

# 283. Interview — Test Data

### Question

How do you manage test data?

Answer:

```text
I use deterministic synthetic data for unit tests and isolated
test data for integration tests.

For concurrent tests I use unique run identifiers.

I avoid production customer data and production secrets.
```

---

# 284. Interview — Parallel Tests

### Question

What can break when pytest runs in parallel?

Answer:

```text
Shared files
ports
database records
Kubernetes resources
environment variables
global state
```

I make tests isolated and use unique resources before enabling
parallel execution.
```

---

# 285. Interview — CI Resource Leaks

### Question

What happens if integration tests fail halfway through?

Answer:

```text
Cleanup must run independently of test success.

I use fixture teardown, try/finally logic, CI post stages,
resource labels, TTLs, and dedicated test environments.

For cloud resources, I also use tags and periodic cleanup
as a safety net.
```

---

# 286. Interview — Test Automation Architecture

### Question

How would you structure a Python DevOps automation repository?

Answer:

```text
clients/
    external API adapters

policies/
    pure deployment/security logic

workflows/
    orchestration

models/
    domain objects

tests/unit/
tests/integration/
tests/smoke/
tests/e2e/

conftest.py
pytest configuration
CI pipeline
```

This keeps external infrastructure separate from business logic.

---

# 287. Interview — GitOps Testing

### Question

How would you test a GitOps deployment?

Answer:

```text
Before Git changes, I validate manifests, image references,
environment configuration, security gates, and policy.

After ArgoCD reconciliation, I verify sync and health, Kubernetes
rollout status, application health, and important metrics.

I keep unit tests independent from the actual cluster and use
integration/E2E tests for the complete Git-to-cluster path.
```

---

# 288. Interview — DevSecOps Testing

### Question

Where would security testing happen?

Answer:

```text
Early and continuously.

I would include SAST, dependency/SCA checks, secret scanning,
IaC checks, and container scanning in CI.

The pipeline should enforce explicit security gates before
deployment.
```

---

# 289. Interview — Production Safety

### Question

How do you prevent test automation from modifying production?

Answer:

```text
Environment validation
AWS account validation
Kubernetes context validation
least-privilege identity
approval checks
dry-run support
production-specific guardrails
```

I also make destructive operations explicit and test the
negative paths.
```

---

# 290. Interview — Test vs Monitoring

### Question

Are tests and monitoring the same?

Answer:

```text
No.

Tests validate expected behavior under controlled conditions.

Monitoring observes real production behavior continuously.

Both are necessary.
```

---

# 291. Interview — Observability in Test Automation

### Question

How do you troubleshoot a failed Kubernetes test?

Answer:

```text
I collect pod status, events, describe output, container logs,
deployment rollout status, and relevant application health
information.

I publish them as CI artifacts so the failure can be diagnosed
without rerunning the test immediately.
```

---

# 292. Interview — Test Automation Metrics

Useful metrics:

```text
Pass rate
Failure rate
Flaky rate
Test duration
Coverage
Pipeline duration
Defect escape rate
```

---

# 293. Interview — Test Automation Optimization

If a pipeline is too slow:

```text
identify slow tests
parallelize independent tests
cache dependencies
run fast tests earlier
separate unit/integration/e2e
reduce unnecessary E2E tests
reuse controlled environments where safe
```

Do not sacrifice critical coverage merely to reduce runtime.

---

# 294. Production Project Example

Consider a Python release automation platform:

```text
GitHub/Jenkins
      |
      v
Python Automation
      |
      +--> Validate AWS account
      |
      +--> Validate Kubernetes context
      |
      +--> Check security gates
      |
      +--> Update Git manifest
      |
      +--> Trigger/observe ArgoCD
      |
      +--> Check EKS rollout
      |
      +--> Query Prometheus
      |
      +--> Query ELK when diagnostics needed
      |
      +--> Rollback if required
```

---

# 295. Unit-Test Architecture

Mock:

```text
AWS
Kubernetes
GitHub
ArgoCD
Prometheus
ELK
```

Execute for real:

```text
validation
policy
parsing
decision logic
rollback logic
```

---

# 296. Integration-Test Architecture

Use:

```text
test AWS account
test EKS
test ECR
test Git repository
test ArgoCD
test Prometheus
```

Run:

```text
release workflow
```

with controlled test services.

---

# 297. Smoke-Test Architecture

After deployment:

```text
ArgoCD Healthy
+
Pods Ready
+
Health API 200
+
Critical API 200
+
Error rate acceptable
```

---

# 298. Failure-Test Architecture

Simulate:

```text
wrong AWS account
wrong cluster
security failure
image failure
ArgoCD failure
Kubernetes rollout failure
Prometheus timeout
application 5xx
```

Then verify:

```text
block
retry
rollback
alert
```

according to policy.

---

# 299. Complete CI/CD Test Flow

```text
Developer
   |
   v
Git
   |
   v
PR
   |
   +--> Lint
   |
   +--> Unit
   |
   +--> Coverage
   |
   +--> SonarQube
   |
   +--> Dependency Scan
   |
   v
Merge
   |
   +--> Build
   |
   +--> Trivy
   |
   +--> Integration
   |
   +--> Publish Artifact
   |
   v
Test Deployment
   |
   +--> ArgoCD
   |
   +--> EKS
   |
   +--> Smoke
   |
   +--> Prometheus
   |
   v
Approval
   |
   v
Production
   |
   +--> Smoke
   +--> Metrics
   +--> Logs
   |
   v
Success / Rollback
```

---

# 300. Final Test Automation Principles

```text
1. Test early.
2. Keep unit tests fast.
3. Mock external boundaries.
4. Keep business logic real.
5. Use integration tests for real dependencies.
6. Use E2E only for critical workflows.
7. Automate smoke tests.
8. Publish structured reports.
9. Collect diagnostics automatically.
10. Make tests deterministic.
11. Control time and randomness.
12. Isolate test data.
13. Clean up test infrastructure.
14. Protect production.
15. Treat security as part of testing.
16. Track flaky tests.
17. Measure test duration.
18. Convert incidents into regression tests.
19. Do not confuse coverage with quality.
20. Make the pipeline enforce meaningful gates.
```

---

# 301. Production Test Automation Checklist

## Code

```text
[ ] Unit tests
[ ] Integration tests
[ ] Smoke tests
[ ] Regression tests
[ ] Critical E2E tests
[ ] Failure-path tests
```

## Pytest

```text
[ ] Fixtures
[ ] conftest.py
[ ] Markers
[ ] Parametrization
[ ] Coverage
[ ] JUnit
[ ] Test discovery
[ ] Parallel execution where safe
```

## CI/CD

```text
[ ] Jenkins/GitHub Actions
[ ] PR checks
[ ] Main branch checks
[ ] Security gates
[ ] Test reports
[ ] Artifacts
[ ] Notifications
```

## Kubernetes

```text
[ ] Test namespace
[ ] Rollout validation
[ ] Pod readiness
[ ] Health endpoints
[ ] Events/log collection
[ ] Resource cleanup
```

## AWS

```text
[ ] Dedicated test account/environment
[ ] Least privilege
[ ] Short-lived credentials
[ ] Resource tagging
[ ] Cleanup
[ ] Cost control
```

## Security

```text
[ ] No production secrets
[ ] SAST
[ ] SCA
[ ] Secret scanning
[ ] Container scanning
[ ] IaC scanning
[ ] Security gate
```

## Reliability

```text
[ ] Timeout
[ ] Retry policy
[ ] Backoff
[ ] Failure classification
[ ] Rollback testing
[ ] Idempotency
[ ] Flaky test tracking
```

---

# 302. Final Takeaway

Test automation in DevOps is not simply:

```bash
pytest
```

It is a complete engineering system:

```text
Code
 ↓
Automated Tests
 ↓
Security
 ↓
Build
 ↓
Integration
 ↓
Deployment
 ↓
Smoke
 ↓
Observability
 ↓
Production Verification
```

For Python-based DevOps automation, the strongest approach is:

```text
Pure business logic
        |
        +--> extensive unit tests
        |
External clients
        |
        +--> mocked in unit tests
        |
Real infrastructure
        |
        +--> integration tests
        |
Complete release workflow
        |
        +--> limited E2E tests
```

The goal is not to maximize the number of tests.

The goal is to create a delivery system where:

```text
bad code
   -> fails early

bad configuration
   -> fails before deployment

security issue
   -> blocks release

deployment problem
   -> detected automatically

production regression
   -> caught quickly

failure
   -> produces useful diagnostics

recovery
   -> is tested before it is needed
```

That is the foundation of production-grade DevOps test automation.

---

# 303. Next File

```text
09-Python-Testing/
├── 01-Pytest-Fundamentals.md       ✓
├── 02-Unit-Testing.md              ✓
├── 03-Mocking.md                   ✓
├── 04-Test-Automation.md           ✓
└── 05-DevOps-Automation-Testing.md
```

Next:

## `05-DevOps-Automation-Testing.md`

The final file should combine Python testing with real DevOps workflows:

```text
AWS automation testing
Kubernetes/EKS automation testing
Terraform automation testing
Docker testing
Helm testing
Jenkins testing
GitHub Actions testing
ArgoCD/GitOps testing
DevSecOps pipeline testing
Prometheus/ELK verification
CI/CD test architecture
Integration environments
End-to-end deployment testing
Rollback testing
Failure injection
Production safety testing
Security testing
Idempotency
Reliability
Observability
Real-world project architecture
Troubleshooting
Senior/FAANG-level interview scenarios
Production checklists
```
