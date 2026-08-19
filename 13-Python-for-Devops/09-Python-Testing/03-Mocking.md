# 03 — Mocking

## 1. Overview

Mocking is used to isolate the code being tested from external dependencies.

In DevOps automation, Python code commonly interacts with:

```text
AWS
Kubernetes
GitHub
Jenkins
ArgoCD
Prometheus
ELK
Databases
Filesystem
Environment variables
Time
HTTP APIs
```

A unit test should normally test:

```text
Your Python logic
```

without requiring:

```text
Real AWS
Real EKS
Real ArgoCD
Real GitHub
Real Jenkins
Real network
```

Mocking provides controlled substitutes for those dependencies.

---

# 2. The Core Mental Model

Without mocking:

```text
Test
 ↓
Python code
 ↓
AWS API
 ↓
Internet
 ↓
AWS
```

With mocking:

```text
Test
 ↓
Python code
 ↓
Mock AWS client
 ↓
Controlled response
```

This makes the test:

```text
Fast
Deterministic
Safe
Repeatable
```

---

# 3. What Is Mocking?

A mock is a controlled test double that can:

```text
Return predefined values
Raise exceptions
Record calls
Verify arguments
Simulate failures
Control external behavior
```

Example:

```python
from unittest.mock import Mock

client = Mock()

client.get_status.return_value = "Healthy"
```

Now:

```python
client.get_status()
```

returns:

```text
Healthy
```

without calling a real service.

---

# 4. Python Mocking Library

Python provides:

```python
unittest.mock
```

Common objects:

```text
Mock
MagicMock
AsyncMock
patch
call
PropertyMock
```

Import:

```python
from unittest.mock import Mock, patch
```

---

# 5. Why DevOps Engineers Need Mocking

Consider a deployment function:

```python
def deploy(client, service):

    response = client.deploy(
        service
    )

    return response["status"]
```

A unit test should not deploy to EKS.

Instead:

```python
mock_client = Mock()

mock_client.deploy.return_value = {
    "status": "success"
}
```

Then test the logic safely.

---

# 6. First Mock Example

Application:

```python
def get_release_status(client):

    response = client.get_status()

    return response["status"]
```

Test:

```python
def test_get_release_status():

    client = Mock()

    client.get_status.return_value = {
        "status": "Healthy"
    }

    result = get_release_status(client)

    assert result == "Healthy"
```

---

# 7. Verify the Dependency Was Called

```python
client.get_status.assert_called_once()
```

This verifies:

```text
get_status()
```

was called exactly once.

---

# 8. Verify Arguments

Application:

```python
def deploy_service(
    client,
    service,
    version
):

    return client.deploy(
        service,
        version
    )
```

Test:

```python
def test_deploy_service():

    client = Mock()

    client.deploy.return_value = {
        "status": "success"
    }

    deploy_service(
        client,
        "payment",
        "1.4.2"
    )

    client.deploy.assert_called_once_with(
        "payment",
        "1.4.2"
    )
```

---

# 9. `return_value`

Use:

```python
mock.method.return_value = value
```

Example:

```python
client.get_status.return_value = "Healthy"
```

Every call returns:

```text
Healthy
```

unless configured otherwise.

---

# 10. `side_effect`

Use `side_effect` when behavior must vary or an exception must be raised.

Example:

```python
client.get_status.side_effect = TimeoutError
```

Then:

```python
client.get_status()
```

raises:

```text
TimeoutError
```

---

# 11. Testing Timeout Handling

Application:

```python
def check_status(client):

    try:
        return client.get_status()

    except TimeoutError:
        return "timeout"
```

Test:

```python
def test_timeout():

    client = Mock()

    client.get_status.side_effect = \
        TimeoutError

    result = check_status(client)

    assert result == "timeout"
```

---

# 12. `side_effect` with Multiple Results

```python
client.get_status.side_effect = [
    "Degraded",
    "Degraded",
    "Healthy"
]
```

Calls return:

```text
1st -> Degraded
2nd -> Degraded
3rd -> Healthy
```

This is useful for retry testing.

---

# 13. Retry Testing

Application:

```python
def wait_for_health(
    client,
    attempts
):

    for _ in range(attempts):

        status = client.get_status()

        if status == "Healthy":
            return True

    return False
```

Test:

```python
def test_retry_until_healthy():

    client = Mock()

    client.get_status.side_effect = [
        "Degraded",
        "Degraded",
        "Healthy"
    ]

    assert wait_for_health(
        client,
        3
    )

    assert client.get_status.call_count == 3
```

---

# 14. Retry Exhaustion

```python
def test_retry_exhausted():

    client = Mock()

    client.get_status.return_value = \
        "Degraded"

    result = wait_for_health(
        client,
        3
    )

    assert result is False
    assert client.get_status.call_count == 3
```

---

# 15. Exception Side Effect

```python
client.deploy.side_effect = \
    RuntimeError("deployment failed")
```

Test:

```python
import pytest


def test_deployment_failure():

    client = Mock()

    client.deploy.side_effect = \
        RuntimeError(
            "deployment failed"
        )

    with pytest.raises(
        RuntimeError,
        match="deployment failed"
    ):
        deploy_service(
            client,
            "payment",
            "1.4.2"
        )
```

---

# 16. `patch`

`patch` temporarily replaces an object during a test.

Example:

```python
from unittest.mock import patch
```

Suppose:

```python
import requests


def get_health():

    response = requests.get(
        "https://example.com/health"
    )

    return response.status_code
```

Test:

```python
@patch("module.requests.get")
def test_get_health(mock_get):

    mock_get.return_value.status_code = 200

    assert get_health() == 200
```

The real network is never called.

---

# 17. Critical `patch` Rule

Patch where the object is **looked up**, not necessarily where it was originally defined.

Example:

```python
# app.py

from requests import get


def health():

    response = get(
        "https://example.com"
    )

    return response.status_code
```

Correct patch:

```python
@patch("app.get")
```

not:

```python
@patch("requests.get")
```

because `app.py` imported its own reference.

This is one of the most important mocking interview concepts.

---

# 18. Why Wrong Patch Targets Cause Problems

Suppose:

```text
app.py
  |
  +-- from requests import get
```

The function uses:

```text
app.get
```

Therefore patch:

```text
app.get
```

not:

```text
requests.get
```

General rule:

```text
Patch the name used by the system under test.
```

---

# 19. `patch.object`

Use when patching an attribute on a specific object/class.

Example:

```python
class DeploymentClient:

    def status(self):
        return "Healthy"
```

Test:

```python
client = DeploymentClient()

with patch.object(
    client,
    "status",
    return_value="Degraded"
):

    assert client.status() \
        == "Degraded"
```

---

# 20. Patch as Context Manager

```python
with patch(
    "app.requests.get"
) as mock_get:

    mock_get.return_value.status_code = 200

    result = health()
```

The patch exists only inside the block.

After the block:

```text
original behavior restored
```

---

# 21. Patch as Decorator

```python
@patch("app.requests.get")
def test_health(mock_get):

    mock_get.return_value.status_code = 200

    assert health() == 200
```

Useful for concise tests.

---

# 22. Patch Multiple Dependencies

```python
@patch("app.argocd_client")
@patch("app.kubernetes_client")
def test_release(
    mock_k8s,
    mock_argocd
):

    ...
```

Remember decorator argument ordering can be confusing.

Prefer clear test structure when many patches are required.

---

# 23. Too Many Patches

If a test requires:

```text
10 patches
```

that may indicate:

```text
high coupling
```

Consider redesigning the application.

For example:

```text
Business logic
        |
   interfaces
        |
+-------+-------+
|       |       |
AWS    K8s    ArgoCD
```

Then inject the dependencies.

---

# 24. Dependency Injection

Instead of:

```python
def deploy():

    client = KubernetesClient()

    return client.create_deployment()
```

Prefer:

```python
def deploy(client):

    return client.create_deployment()
```

Now:

```text
production -> real client
test -> mock client
```

---

# 25. Dependency Injection Architecture

```text
Production:

DeploymentService
       |
       v
Real KubernetesClient


Testing:

DeploymentService
       |
       v
Mock KubernetesClient
```

This is cleaner than patching every internal construction.

---

# 26. Mock vs Stub

A stub primarily provides controlled responses.

Example:

```python
stub.get_status.return_value = \
    "Healthy"
```

A mock can also verify:

```text
Was it called?
How many times?
With what arguments?
```

---

# 27. Fake

A fake is a lightweight working implementation.

Example:

```python
class FakeDeploymentClient:

    def __init__(self):
        self.deployments = {}

    def deploy(
        self,
        name,
        version
    ):
        self.deployments[name] = version
```

This behaves more like a small real system.

---

# 28. Mock vs Fake

Mock:

```text
behavior configured per test
```

Fake:

```text
small working implementation
```

Example:

```text
Mock Kubernetes client
Fake in-memory deployment store
```

---

# 29. Spy

A spy observes calls while preserving behavior.

In Python, mock objects can often provide similar call-recording behavior.

Conceptually:

```text
real behavior
+
call tracking
```

Use the simplest test double that gives useful confidence.

---

# 30. Test Double Summary

```text
Dummy
  -> passed only to satisfy a parameter

Stub
  -> returns controlled data

Mock
  -> controls + verifies interactions

Spy
  -> observes real behavior

Fake
  -> simplified working implementation
```

---

# 31. `MagicMock`

`MagicMock` extends mock behavior to Python magic methods.

Useful for:

```text
__enter__
__exit__
__iter__
__len__
__getitem__
__setitem__
```

Example:

```python
from unittest.mock import MagicMock
```

---

# 32. Context Manager Mocking

Suppose:

```python
def read_config(client):

    with client.connection() as conn:
        return conn.read()
```

A `MagicMock` can model:

```text
__enter__
__exit__
```

Example:

```python
client = MagicMock()

client.connection.return_value.__enter__.return_value.read.return_value = \
    "config"
```

Then test the function without a real connection.

---

# 33. Mocking Iteration

```python
mock = MagicMock()

mock.__iter__.return_value = iter(
    ["dev", "staging", "prod"]
)
```

Now:

```python
list(mock)
```

returns the controlled values.

---

# 34. Mocking Dictionary Access

```python
mock = MagicMock()

mock["environment"] = "prod"
```

This can be useful for code that interacts with mapping-like objects.

But for simple dictionaries, use real dictionaries rather than mocks.

---

# 35. Use Real Data When Possible

Bad:

```python
config = MagicMock()
config.get.return_value = ...
```

when the code only needs:

```python
config = {
    "environment": "prod"
}
```

Use real data for simple values.

Mock external behavior, not ordinary data structures.

---

# 36. Mocking HTTP Requests

Suppose:

```python
import requests


def get_release():

    response = requests.get(
        "https://api.example.com/release"
    )

    response.raise_for_status()

    return response.json()
```

Test:

```python
@patch("module.requests.get")
def test_get_release(mock_get):

    response = Mock()

    response.json.return_value = {
        "version": "1.4.2"
    }

    mock_get.return_value = response

    result = get_release()

    assert result["version"] == "1.4.2"
```

---

# 37. Verify HTTP Request

```python
mock_get.assert_called_once_with(
    "https://api.example.com/release"
)
```

If headers are required:

```python
mock_get.assert_called_once_with(
    "https://api.example.com/release",
    headers=expected_headers
)
```

---

# 38. Mock HTTP Status Codes

```python
response.status_code = 503
```

Or:

```python
response.raise_for_status.side_effect = \
    requests.HTTPError("503")
```

Test how your application handles the error.

---

# 39. Mock Timeout

```python
mock_get.side_effect = \
    requests.Timeout()
```

Test:

```text
timeout
retry
failure
fallback
```

according to application behavior.

---

# 40. Mock Connection Error

```python
mock_get.side_effect = \
    requests.ConnectionError()
```

Verify that the application:

```text
does not crash unexpectedly
```

and follows the intended retry/failure policy.

---

# 41. Mock 401

```python
response.status_code = 401
```

Test:

```text
authentication failure
```

---

# 42. Mock 403

```python
response.status_code = 403
```

Test:

```text
authorization failure
```

Do not automatically retry authorization failures unless your application explicitly requires that behavior.

---

# 43. Mock 429

```python
response.status_code = 429
```

Test:

```text
rate-limit handling
backoff
retry limit
```

---

# 44. Mock 500/503

```text
500
502
503
504
```

These often represent transient server-side conditions.

Test the exact retry policy.

---

# 45. Mock Malformed JSON

```python
response.json.side_effect = \
    ValueError("invalid JSON")
```

Test:

```text
malformed response
```

This is important because APIs can fail in unexpected ways.

---

# 46. Mock Missing JSON Fields

```python
response.json.return_value = {}
```

Test:

```text
missing version
missing status
missing health
```

Your parser should follow an explicit contract.

---

# 47. Mocking Boto3

Suppose:

```python
import boto3


def get_instance(instance_id):

    ec2 = boto3.client("ec2")

    return ec2.describe_instances(
        InstanceIds=[instance_id]
    )
```

Unit tests should not call AWS.

Patch:

```python
@patch("module.boto3.client")
def test_get_instance(mock_client):

    ec2 = Mock()

    mock_client.return_value = ec2

    ec2.describe_instances.return_value = {
        "Reservations": []
    }

    result = get_instance("i-test")

    assert result["Reservations"] == []
```

---

# 48. Verify Boto3 Client Construction

```python
mock_client.assert_called_once_with(
    "ec2"
)
```

This verifies the correct AWS service is selected.

---

# 49. Mock AWS Response

Example:

```python
ec2.describe_instances.return_value = {
    "Reservations": [
        {
            "Instances": [
                {
                    "InstanceId": "i-test",
                    "State": {
                        "Name": "running"
                    }
                }
            ]
        }
    ]
}
```

Then test your parsing logic.

---

# 50. AWS Error Mocking

Boto3 raises service-specific exceptions.

For unit tests, you can often inject or configure a controlled exception path rather than constructing complicated SDK exceptions unless the exact exception type is part of your contract.

Test:

```text
AccessDenied
Throttling
ResourceNotFound
Network failure
```

according to your code's handling policy.

---

# 51. AWS Permission Logic

Suppose:

```python
def classify_aws_error(error):

    if "AccessDenied" in str(error):
        return "authorization"

    return "unknown"
```

Unit-test:

```python
def test_access_denied():

    error = Exception(
        "AccessDenied"
    )

    assert classify_aws_error(
        error
    ) == "authorization"
```

For production code, prefer structured exception types/codes where available rather than string matching.

---

# 52. Mocking ECR

Suppose code calls:

```python
ecr.describe_images(...)
```

Mock:

```python
ecr = Mock()

ecr.describe_images.return_value = {
    "imageDetails": [
        {
            "imageDigest":
                "sha256:abc"
        }
    ]
}
```

Test:

```text
digest extraction
image existence
tag mapping
```

---

# 53. Mocking EKS

For EKS control-plane API operations, mock the AWS API client.

For Kubernetes workload operations, mock the Kubernetes Python client.

Do not confuse:

```text
EKS AWS API
```

with:

```text
Kubernetes API
```

They are different interfaces.

---

# 54. Mocking Kubernetes Python Client

Suppose:

```python
def get_deployment(
    apps_api,
    name,
    namespace
):

    return apps_api.read_namespaced_deployment(
        name=name,
        namespace=namespace
    )
```

Test:

```python
def test_get_deployment():

    apps_api = Mock()

    deployment = Mock()

    apps_api.read_namespaced_deployment.return_value = \
        deployment

    result = get_deployment(
        apps_api,
        "payment",
        "prod"
    )

    assert result is deployment
```

---

# 55. Verify Kubernetes Call

```python
apps_api.read_namespaced_deployment.assert_called_once_with(
    name="payment",
    namespace="prod"
)
```

This verifies:

```text
correct resource
correct namespace
```

---

# 56. Mock Kubernetes Status

Kubernetes Python client objects are often nested objects.

For example:

```python
deployment.status.replicas
deployment.status.ready_replicas
```

You can build a simple fake object instead of deeply mocking everything.

Example:

```python
from types import SimpleNamespace

deployment = SimpleNamespace(
    status=SimpleNamespace(
        replicas=3,
        ready_replicas=3
    )
)
```

Then:

```python
apps_api.read_namespaced_deployment.return_value = \
    deployment
```

This is often more readable than a huge mock chain.

---

# 57. Mock vs SimpleNamespace

Use:

```text
Mock
```

when you need:

```text
call verification
side effects
interaction testing
```

Use:

```text
SimpleNamespace
```

or a dataclass when you need:

```text
simple structured test data
```

---

# 58. Kubernetes API Exception

Kubernetes client code may raise:

```text
ApiException
```

Test:

```text
404 -> resource not found
403 -> forbidden
500 -> server error
```

Use the actual exception type where your application depends on its fields.

---

# 59. Mocking Kubernetes 404

Application behavior might be:

```text
deployment exists -> update
404 -> create
```

Unit test both branches.

Conceptually:

```python
apps_api.read_namespaced_deployment.side_effect = \
    not_found_error
```

Then verify:

```text
create_namespaced_deployment()
```

was called.

---

# 60. Kubernetes Create-or-Update Logic

```text
Read
 ↓
Found?
 ├── yes -> update
 └── no  -> create
```

Tests:

```text
resource found
resource missing
unexpected API error
```

---

# 61. Verify No Unwanted Side Effect

If resource exists:

```python
apps_api.create_namespaced_deployment.assert_not_called()
```

If missing:

```python
apps_api.create_namespaced_deployment.assert_called_once()
```

This is important for idempotent automation.

---

# 62. Mocking Kubernetes Watch

If your code watches events:

```text
ADDED
MODIFIED
DELETED
```

mock the event stream.

Example concept:

```python
watch_events = [
    {"type": "MODIFIED"},
    {"type": "MODIFIED"},
    {"type": "MODIFIED"}
]
```

Then test:

```text
event handling
termination condition
timeout
```

---

# 63. Mocking ArgoCD

Suppose:

```python
def get_application(
    client,
    name
):

    return client.get_application(
        name
    )
```

Test:

```python
client = Mock()

client.get_application.return_value = {
    "status": {
        "sync": {
            "status": "Synced"
        },
        "health": {
            "status": "Healthy"
        }
    }
}
```

Then test parsing and policy separately.

---

# 64. ArgoCD Mocking Principle

Do not make one test simultaneously verify:

```text
ArgoCD API call
response parsing
health policy
rollback
```

Separate them:

```text
Test 1 -> client interaction
Test 2 -> parser
Test 3 -> policy
Test 4 -> workflow decision
```

This makes failures easier to diagnose.

---

# 65. Mocking Jenkins

Suppose:

```python
def trigger_job(
    client,
    job,
    parameters
):

    return client.build_job(
        job,
        parameters
    )
```

Test:

```python
client = Mock()

client.build_job.return_value = 123

result = trigger_job(
    client,
    "deploy-payment",
    {"VERSION": "1.4.2"}
)

assert result == 123
```

Verify:

```python
client.build_job.assert_called_once_with(
    "deploy-payment",
    {"VERSION": "1.4.2"}
)
```

---

# 66. Mocking GitHub

Potential operations:

```text
get repository
get branch
create commit
update file
create pull request
get workflow
```

Mock these interactions.

Example:

```python
github = Mock()

github.get_branch.return_value = {
    "name": "main"
}
```

Test the application's branch-validation logic.

---

# 67. GitOps Commit Automation

A common workflow:

```text
Build image
 ↓
Get digest
 ↓
Update Git manifest
 ↓
Commit
 ↓
ArgoCD sync
```

Unit tests should isolate each decision:

```text
digest formatting
manifest update
commit payload
sync policy
```

---

# 68. Mocking Git Commit

Test:

```text
correct file
correct branch
correct content
correct commit message
```

Example:

```python
github.update_file.assert_called_once_with(
    path="payment/deployment.yaml",
    branch="main",
    message="Update payment image",
    content=expected_content
)
```

Only assert the arguments that form part of your contract.

---

# 69. Mocking Prometheus

Suppose:

```python
def query_prometheus(
    client,
    query
):

    return client.query(query)
```

Mock:

```python
client = Mock()

client.query.return_value = {
    "status": "success",
    "data": {
        "result": []
    }
}
```

Test:

```text
success
empty result
malformed result
API error
```

---

# 70. Mocking Metrics

For release gates:

```text
error rate
latency
availability
CPU
memory
```

mock the returned metric value.

Example:

```python
prometheus.query.return_value = 0.005
```

Then test:

```text
0.005 -> pass
0.010 -> boundary
0.011 -> fail
```

---

# 71. Mocking ELK

For log-search automation:

```text
query logs
count errors
find deployment failures
```

mock the search response.

Test:

```text
matching logs
no logs
malformed response
backend failure
```

Do not require a live Elasticsearch cluster for every unit test.

---

# 72. Mocking Databases

Use mocks for:

```text
connection creation
query execution
transaction behavior
error handling
```

For complicated SQL behavior, integration tests against an isolated database are often more valuable than mocking every database detail.

---

# 73. Mocking Filesystem

Python code:

```python
from pathlib import Path


def load_version(path):

    return Path(path).read_text()
```

You can either:

```text
use tmp_path
```

or mock:

```text
Path.read_text
```

Prefer `tmp_path` when actual filesystem behavior is what you want to verify.

---

# 74. Mocking Environment Variables

Pytest provides:

```python
monkeypatch
```

Example:

```python
def test_environment(monkeypatch):

    monkeypatch.setenv(
        "DEPLOY_ENV",
        "prod"
    )

    ...
```

This is often preferable to manually patching `os.environ`.

---

# 75. `monkeypatch`

Useful for:

```text
environment variables
module attributes
dictionary values
object attributes
working directory
```

Example:

```python
def test_config(monkeypatch):

    monkeypatch.setenv(
        "TIMEOUT",
        "30"
    )

    assert load_timeout() == 30
```

---

# 76. `monkeypatch` vs `patch`

Use `monkeypatch` for:

```text
simple environment/config changes
```

Use `unittest.mock.patch` for:

```text
mocking callable dependencies
verifying calls
return values
side effects
```

Both are useful.

---

# 77. Mocking Time

Time-dependent code is a common source of flaky tests.

Bad:

```python
datetime.now()
```

deep inside business logic.

Better:

```python
def is_expired(
    expires_at,
    now
):

    return now >= expires_at
```

Then tests control `now`.

---

# 78. Dependency Injection for Time

Production:

```python
now = datetime.now()
```

Testing:

```python
now = fixed_time
```

This is often cleaner than globally patching datetime.

---

# 79. Mocking Sleep

Bad unit test:

```python
time.sleep(30)
```

Instead, inject a sleep function or patch it.

Example:

```python
@patch("module.time.sleep")
def test_retry_sleep(mock_sleep):

    ...
```

Then verify:

```python
mock_sleep.assert_called()
```

No real 30-second delay.

---

# 80. Testing Backoff

Suppose:

```python
sleep(attempt * 2)
```

Test:

```python
mock_sleep.assert_has_calls(...)
```

Verify the intended delay sequence.

---

# 81. Mocking Randomness

Randomness can cause flaky tests.

Instead of:

```python
random.random()
```

directly in business logic, inject or patch it.

Example:

```python
@patch("module.random.random")
def test_jitter(mock_random):
    mock_random.return_value = 0.5
```

Now the result is deterministic.

---

# 82. Mocking UUIDs

If release IDs use:

```python
uuid.uuid4()
```

tests may need deterministic IDs.

Patch the lookup location:

```python
@patch("module.uuid.uuid4")
```

Return a known UUID.

---

# 83. Mocking OS Commands

Suppose:

```python
import subprocess


def terraform_plan():

    return subprocess.run(
        ["terraform", "plan"],
        check=True
    )
```

Unit test can patch:

```python
@patch("module.subprocess.run")
def test_terraform_plan(mock_run):

    terraform_plan()

    mock_run.assert_called_once_with(
        ["terraform", "plan"],
        check=True
    )
```

Do not run Terraform in the unit test.

---

# 84. Testing Command Failure

```python
mock_run.side_effect = \
    subprocess.CalledProcessError(
        1,
        ["terraform", "plan"]
    )
```

Then test the application's failure handling.

---

# 85. Mocking Docker Commands

If Python runs:

```text
docker build
docker push
docker tag
```

unit-test:

```text
command construction
arguments
failure handling
```

Actual Docker execution belongs in integration testing.

---

# 86. Mocking Helm

Test:

```text
release name
chart
namespace
values
timeout
```

without invoking Helm.

Integration tests can run Helm against a test cluster.

---

# 87. Mocking Terraform

Unit-test:

```text
variable construction
backend configuration
workspace/environment mapping
command construction
```

Integration-test:

```text
terraform init
terraform plan
terraform apply
```

in a controlled environment.

---

# 88. Mocking Ansible

Unit-test:

```text
inventory generation
variable generation
command construction
```

Integration-test actual playbook execution in a disposable environment.

---

# 89. Mocking Jenkins/GitHub Actions Pipeline Decisions

Python may determine:

```text
which job
which environment
which parameters
```

Mock the API call and test the decision.

The CI runner itself is not the unit-test target.

---

# 90. `assert_called_once`

Use:

```python
mock.assert_called_once()
```

when the dependency should be invoked exactly once.

---

# 91. `assert_called_once_with`

Use:

```python
mock.assert_called_once_with(
    "payment",
    "prod"
)
```

when both count and arguments matter.

---

# 92. `assert_called`

Use when:

```text
at least one call
```

is expected.

But it does not verify the exact number of calls.

---

# 93. `call_count`

```python
assert mock.call_count == 3
```

Useful for retry loops.

---

# 94. `call_args`

```python
args, kwargs = mock.call_args
```

This gives the most recent call's arguments.

Use only when direct assertion methods do not make the test clearer.

---

# 95. `call_args_list`

```python
mock.call_args_list
```

contains all calls.

Example:

```python
[
    call("dev"),
    call("staging"),
    call("prod")
]
```

Useful for verifying sequences.

---

# 96. `assert_has_calls`

```python
from unittest.mock import call

mock.assert_has_calls(
    [
        call("dev"),
        call("prod")
    ]
)
```

Use when call order matters.

---

# 97. Ordered vs Unordered Calls

By default:

```python
assert_has_calls(...)
```

checks calls in order when interpreting the sequence.

If order does not matter:

```python
assert_has_calls(
    expected_calls,
    any_order=True
)
```

Only use order assertions when order is part of the behavior.

---

# 98. `assert_not_called`

Example:

```python
mock.rollback.assert_not_called()
```

Useful for healthy deployments:

```text
Healthy
 ↓
No rollback
```

---

# 99. Negative Interaction Testing

Example:

```python
def test_no_prod_deploy_without_approval():

    deploy = Mock()

    run_release(
        deploy,
        environment="prod",
        approved=False
    )

    deploy.assert_not_called()
```

This is an important production safety test.

---

# 100. Testing Dangerous Side Effects

For infrastructure automation, negative tests are extremely valuable.

Examples:

```text
No production deploy without approval
No rollback when healthy
No create when resource exists
No retry on 403
No secret logging
No apply during dry-run
No delete during validation
```

---

# 101. `autospec`

`autospec` makes mocks follow the signature of the target.

Example:

```python
@patch(
    "module.DeploymentClient",
    autospec=True
)
def test_client(mock_client):
    ...
```

This helps catch incorrect method usage.

---

# 102. Why `autospec` Helps

Without a specification, a mock can accept almost anything:

```python
mock.deploy(
    unexpected="value"
)
```

even if the real object does not support that call.

With:

```text
autospec=True
```

the mock better reflects the real API.

---

# 103. `spec`

Example:

```python
client = Mock(
    spec=DeploymentClient
)
```

The mock is constrained by the target interface.

Useful for catching:

```text
misspelled methods
invalid attributes
incorrect assumptions
```

---

# 104. `spec_set`

`spec_set` is stricter.

It prevents adding attributes not present on the specification.

Useful when you want stronger interface protection.

---

# 105. Mocking Can Hide Bugs

Example:

```python
client.deply()
```

Misspelled:

```text
deply
```

A plain `Mock` may happily create:

```text
mock.deply
```

and your test could pass.

Using:

```text
spec
autospec
```

helps detect such mistakes.

---

# 106. Production Recommendation

For important client interfaces:

```text
Prefer autospec/spec where practical.
```

Especially for:

```text
AWS adapters
Kubernetes adapters
GitHub clients
Jenkins clients
ArgoCD clients
```

---

# 107. `AsyncMock`

For asynchronous Python code:

```python
from unittest.mock import AsyncMock
```

Example:

```python
client.get_status = AsyncMock(
    return_value="Healthy"
)
```

Then:

```python
result = await client.get_status()
```

returns:

```text
Healthy
```

---

# 108. Testing Async Calls

```python
client.get_status.assert_awaited_once()
```

And:

```python
client.get_status.assert_awaited_once_with(
    "payment"
)
```

Use `AsyncMock` instead of regular `Mock` for async callables.

---

# 109. Async DevOps Use Cases

Async automation may involve:

```text
parallel health checks
multiple API calls
event processing
Kubernetes watches
concurrent deployments
```

Mocking async dependencies correctly is important.

---

# 110. Async Side Effects

```python
client.get_status.side_effect = [
    "Degraded",
    "Healthy"
]
```

Use async mocks and test the exact sequence.

---

# 111. Mocking Concurrent Operations

If code uses:

```python
asyncio.gather(...)
```

mock each async dependency.

Test:

```text
all succeed
one fails
multiple fail
timeout
cancellation
```

Do not make concurrency tests depend on real network timing.

---

# 112. Mocking Threaded Code

If code uses:

```text
ThreadPoolExecutor
```

avoid testing thread scheduling itself in a unit test.

Test:

```text
worker behavior
result aggregation
exception handling
```

Use integration tests for meaningful concurrency behavior.

---

# 113. Mocking Environment Configuration

Example:

```python
def get_region():

    return os.getenv(
        "AWS_REGION",
        "ap-south-1"
    )
```

Test:

```python
def test_region(monkeypatch):

    monkeypatch.setenv(
        "AWS_REGION",
        "us-east-1"
    )

    assert get_region() \
        == "us-east-1"
```

---

# 114. Testing Missing Environment Variables

```python
def test_missing_region(monkeypatch):

    monkeypatch.delenv(
        "AWS_REGION",
        raising=False
    )

    ...
```

Verify the intended fallback or failure.

---

# 115. Mocking Configuration Loader

Instead of reading real production configuration:

```python
config_loader = Mock()

config_loader.load.return_value = {
    "environment": "staging"
}
```

Then test the business logic.

---

# 116. Mocking Secret Manager

For code that calls:

```text
AWS Secrets Manager
HashiCorp Vault
Kubernetes Secrets
```

unit-test:

```text
secret retrieval success
secret missing
permission failure
timeout
```

Never use actual production secrets.

---

# 117. Secret Mock Example

```python
secrets = Mock()

secrets.get_secret.return_value = {
    "username": "test-user",
    "password": "test-password"
}
```

Then verify your application consumes the values correctly.

---

# 118. Secret Redaction Test

If an exception contains a secret:

```python
error = "password=test-password"
```

verify logs/output do not expose it.

```python
assert "test-password" \
    not in sanitized_output
```

---

# 119. Mocking IAM Decisions

If your code maps:

```text
operation
environment
role
```

to an authorization decision, unit-test that policy.

Do not call IAM for every unit test.

---

# 120. Mocking AWS STS Identity

For code that checks the current AWS account:

```python
sts = Mock()

sts.get_caller_identity.return_value = {
    "Account": "TEST"
}
```

Test:

```text
expected account
unexpected account
missing identity
```

---

# 121. Production Guardrail

A deployment tool should potentially verify:

```text
expected account
```

before production mutation.

Unit-test:

```text
matching account -> allowed
different account -> blocked
```

Then integration-test the actual STS call.

---

# 122. Mocking Kubernetes Current Context

If code reads kubeconfig/current context, unit-test the context validation logic separately.

Example:

```text
expected context = prod-eks
actual context = dev-eks
```

Expected:

```text
BLOCK
```

This is a high-value safety check.

---

# 123. Mocking `subprocess`

For:

```text
kubectl
helm
terraform
docker
aws
git
```

unit-test command construction.

Example:

```python
@patch("module.subprocess.run")
def test_kubectl_apply(mock_run):

    apply_manifest()

    mock_run.assert_called_once()
```

Then inspect arguments.

---

# 124. Avoid Over-Specifying Commands

If the exact ordering of flags is not important, do not make tests unnecessarily brittle.

Test the contract:

```text
kubectl apply
target manifest
```

rather than every incidental formatting detail.

---

# 125. Testing Command Failure

```python
mock_run.side_effect = \
    subprocess.CalledProcessError(
        returncode=1,
        cmd=["kubectl", "apply"]
    )
```

Verify:

```text
error classification
logging
rollback decision
exit status
```

---

# 126. Mocking File Uploads

For artifact upload:

```text
S3
Artifactory
GitHub
```

mock:

```text
upload call
```

and verify:

```text
bucket/repository
path
artifact
metadata
```

---

# 127. Mocking JFrog Artifactory

Typical flow:

```text
Build
 ↓
Artifact
 ↓
Artifactory upload
 ↓
Version
```

Unit-test:

```text
repository selection
artifact path
version
upload decision
```

Mock the actual HTTP/client call.

---

# 128. Mocking Container Registry

For ECR/Artifactory registry workflows:

```text
image exists?
digest?
tag?
push?
```

Use controlled responses.

Test:

```text
image exists
image missing
digest mismatch
push failure
```

---

# 129. Mocking Release Verification

Example:

```python
def verify_release(
    argocd,
    prometheus
):

    app = argocd.get_status()

    if app != "Healthy":
        return False

    rate = prometheus.error_rate()

    return rate < 0.01
```

Test each dependency independently where possible.

---

# 130. Better Design

Separate:

```python
def argocd_healthy(status):
    ...

def error_rate_healthy(rate):
    ...

def release_healthy(
    argocd_status,
    error_rate
):
    ...
```

Then the core policy is pure.

Only API adapters need mocks.

---

# 131. Mock Boundary Architecture

Recommended:

```text
                Release Workflow
                       |
             +---------+---------+
             |                   |
       Business Logic       External APIs
             |                   |
          Pure code        Adapters/Clients
                                 |
                   +------+------+------+
                   |      |      |      |
                  AWS    EKS   ArgoCD  GitHub
```

Unit-test:

```text
business logic
```

Mock at:

```text
external boundary
```

---

# 132. Testing Adapters

An adapter might:

```text
call API
translate response
raise application-specific error
```

Unit-test:

```text
API response -> adapter result
API exception -> application exception
```

Integration-test:

```text
adapter -> real test API
```

---

# 133. Contract Tests

If your adapter depends on an external API contract, consider contract tests.

Example:

```text
ArgoCD response shape
GitHub API response
internal service API
```

Unit tests use controlled data.

Contract/integration tests validate that the real service still matches assumptions.

---

# 134. Mocking Is Not Integration Testing

Important:

```text
Mock says:
"The code behaves correctly when the API returns X."
```

Integration says:

```text
"The real API actually returns X and the client works."
```

Both provide different confidence.

---

# 135. Test Pyramid with Mocking

```text
             E2E
            /   \
       Integration
         /       \
      Unit + Mocks
```

Most tests should remain:

```text
fast
isolated
cheap
```

---

# 136. Mocking Strategy for DevOps

### Unit

Mock:

```text
AWS
Kubernetes
ArgoCD
GitHub
Jenkins
Prometheus
```

### Integration

Use:

```text
test AWS account
test EKS
test ArgoCD
test GitHub repository
```

where appropriate.

### E2E

Validate:

```text
complete deployment workflow
```

---

# 137. Testing AWS Automation

Architecture:

```text
Python
  |
AWS Adapter
  |
Boto3
```

Unit:

```text
Mock Boto3/client adapter
```

Integration:

```text
Test AWS account
```

E2E:

```text
Infrastructure workflow
```

---

# 138. Testing Kubernetes Automation

Architecture:

```text
Python
  |
Kubernetes Adapter
  |
Kubernetes API
```

Unit:

```text
Mock adapter/client
```

Integration:

```text
Test cluster
```

E2E:

```text
deploy -> verify -> rollback
```

---

# 139. Testing GitOps Automation

```text
Python
  |
GitHub API
  |
Git
  |
ArgoCD
  |
EKS
```

Unit tests should stop at:

```text
GitHub/ArgoCD boundary
```

Integration/E2E validates the entire chain.

---

# 140. Testing Jenkins Automation

```text
Python
  |
Jenkins API
  |
Pipeline
```

Unit:

```text
job selection
parameters
decision logic
response parsing
```

Integration:

```text
test Jenkins
```

E2E:

```text
trigger -> pipeline -> deployment
```

---

# 141. Testing CI/CD Failure Paths

Mock:

```text
Jenkins failure
GitHub failure
ArgoCD failure
Kubernetes failure
```

Verify:

```text
correct failure state
correct logs
correct notification
correct exit code
```

---

# 142. Notification Mocking

If deployment sends:

```text
Slack
Email
Teams
PagerDuty
```

mock the notification client.

Test:

```text
success notification
failure notification
rollback notification
```

Also verify:

```text
no duplicate notifications
```

when required.

---

# 143. Mocking Monitoring Alerts

A release workflow may call:

```text
Prometheus
```

and then:

```text
notification service
```

Unit-test:

```text
metric breach -> notification
metric healthy -> no notification
```

---

# 144. Mocking Feature Flags

```python
flags = Mock()

flags.enabled.return_value = True
```

Test behavior when:

```text
feature enabled
feature disabled
flag service unavailable
```

If unavailable behavior is safety-critical, explicitly test the fail-open/fail-closed policy.

---

# 145. Fail-Open vs Fail-Closed

Security decision:

```text
Dependency unavailable
```

Should system:

```text
ALLOW
```

or:

```text
BLOCK
```

Examples:

```text
Security policy -> usually fail closed
Optional metrics -> may degrade gracefully
```

The exact policy must be explicit and tested.

---

# 146. Mocking Circuit Breakers

Test states:

```text
CLOSED
 ↓ failures
OPEN
 ↓ timeout
HALF_OPEN
 ↓ success
CLOSED
```

Mock dependency failures and time.

This is useful for automation interacting with unstable APIs.

---

# 147. Mocking Rate Limiting

Test:

```text
request allowed
rate limit reached
retry-after
backoff
retry exhausted
```

Mock the API response and time rather than waiting in real time.

---

# 148. Mocking Pagination

API responses may contain:

```text
page 1
page 2
page 3
```

Use:

```python
client.list.side_effect = [
    page1,
    page2,
    page3
]
```

Test:

```text
all items collected
pagination stops
empty page
API failure
```

---

# 149. Mocking Polling

Polling example:

```text
check
 ↓
not ready
 ↓
check
 ↓
not ready
 ↓
check
 ↓
ready
```

Mock status sequence:

```python
client.status.side_effect = [
    "Pending",
    "Pending",
    "Ready"
]
```

Verify:

```text
three calls
successful termination
```

---

# 150. Polling Timeout

Test:

```text
Pending
Pending
Pending
...
timeout
```

Verify:

```text
stop polling
raise/return timeout
```

Never let a unit test actually wait for production timeout durations.

---

# 151. Mocking Kubernetes Rollout Polling

Sequence:

```text
0 ready
1 ready
2 ready
3 ready
```

or:

```text
3 desired
2 ready
2 ready
2 ready
timeout
```

Test both success and timeout.

---

# 152. Mocking ArgoCD Sync Polling

Possible:

```text
OutOfSync
Syncing
Synced
Healthy
```

Test that the workflow waits for the intended terminal condition.

Also test:

```text
Synced + Degraded
```

which should not necessarily be considered success.

---

# 153. Mocking Concurrent Releases

If two releases target the same service:

```text
Release A
Release B
```

test lock/decision logic separately.

Example:

```text
lock exists -> block
lock absent -> proceed
```

Mock the lock store.

---

# 154. Mocking Distributed Locks

For:

```text
DynamoDB
Redis
database
Kubernetes Lease
```

unit-test:

```text
acquire
already acquired
release
expiration
failure
```

Use integration tests for actual distributed lock behavior.

---

# 155. Mocking Cache

Test:

```text
cache hit
cache miss
cache stale
cache failure
```

Mock the cache client.

Do not make every unit test depend on Redis.

---

# 156. Mocking DNS

If automation validates:

```text
Route53
DNS
endpoint
```

mock DNS/API responses for unit tests.

Integration tests can verify actual DNS behavior.

---

# 157. Mocking Load Balancer Health

For ALB automation:

```text
target healthy
target unhealthy
no targets
```

mock AWS responses.

Test:

```text
healthy -> proceed
unhealthy -> block/rollback
```

---

# 158. Mocking RDS Health

Test application policy based on:

```text
available
maintenance
failed
```

without querying a real RDS instance.

---

# 159. Mocking S3

Test:

```text
object exists
object missing
permission denied
upload success
upload failure
```

Mock the S3 client.

Integration-test actual S3 behavior in a controlled account.

---

# 160. Mocking S3 State Lock Logic

If automation uses S3 for state:

```text
state exists
state missing
state inaccessible
```

test the application behavior.

Do not confuse state-lock/application logic with Terraform's own backend implementation.

---

# 161. Mocking Terraform State

If Python reads Terraform state:

```text
valid state
missing output
malformed state
stale state
```

mock the state source and test parsing.

---

# 162. Mocking Ansible Results

Test:

```text
changed
ok
failed
unreachable
```

and verify release policy.

---

# 163. Mocking Docker Registry Failures

Test:

```text
authentication failure
image missing
push failure
digest mismatch
rate limit
```

This is important in CI/CD automation.

---

# 164. Mocking SonarQube

Mock:

```text
quality gate = OK
quality gate = ERROR
quality gate = NONE
```

Then verify release policy.

Do not test SonarQube itself with unit tests.

---

# 165. Mocking Trivy Results

Mock scanner output:

```json
{
  "critical": 0,
  "high": 2
}
```

Test your gate.

Also test:

```text
critical > 0
high > allowed
malformed output
scanner failure
```

---

# 166. Mocking Veracode Results

Mock:

```text
approved
rejected
scan unavailable
```

Test:

```text
release decision
```

---

# 167. Mocking Git Status

For automation:

```text
clean
modified
conflicted
```

Test whether deployment is allowed.

---

# 168. Mocking Git Branch Protection

Test:

```text
main -> allowed
feature -> blocked
```

or whatever policy your project requires.

---

# 169. Mocking Pull Request Checks

Example:

```text
unit tests -> passed
security -> passed
approval -> passed
```

Mock GitHub check responses.

Test:

```text
all passed -> merge allowed
one failed -> merge blocked
```

---

# 170. Mocking Deployment Approval

Mock approval service:

```python
approval.get.return_value = True
```

Test:

```text
approved -> proceed
not approved -> block
```

Also test:

```text
approval expired
approval missing
approval service unavailable
```

---

# 171. Mocking Audit Service

If release actions must be audited:

```text
audit.record(...)
```

Mock it and verify:

```text
correct event
correct service
correct environment
```

Do not include secrets.

---

# 172. Mocking Time in Approval Expiration

Example:

```text
approval created 10:00
expires 11:00
current 11:01
```

Inject controlled time.

Test:

```text
before expiry
at expiry
after expiry
```

---

# 173. Mocking Notification Failures

What happens if:

```text
deployment failed
notification failed
```

Should the deployment result become:

```text
failed
```

or:

```text
failed + notification warning
```

Define and unit-test the policy.

---

# 174. Mocking Observability Failures

If Prometheus is unavailable:

```text
Does release stop?
Does it continue?
Does it enter unknown state?
```

This is a production policy decision.

Test it explicitly.

---

# 175. Mocking Unknown States

External APIs can return unexpected states.

Example:

```text
Unknown
Pending
None
```

Never assume only:

```text
Healthy
Degraded
```

Test unknown-state behavior.

---

# 176. Mocking Schema Changes

If API response changes:

```text
status.health
```

becomes:

```text
status.health.status
```

your parser may break.

Use representative fixture data and contract tests to detect schema changes.

---

# 177. Mock Data Should Be Realistic

Bad:

```python
response = {
    "status": "x"
}
```

when production response is deeply nested.

Better:

Use a minimal but structurally accurate response:

```python
response = {
    "status": {
        "health": {
            "status": "Healthy"
        }
    }
}
```

---

# 178. Do Not Copy Huge Production Responses

Avoid putting:

```text
5000-line API response
```

into every test.

Keep fixtures:

```text
minimal
relevant
readable
```

Create specialized fixtures for edge cases.

---

# 179. Fixture Organization

Example:

```text
tests/
├── fixtures/
│   ├── argocd_healthy.json
│   ├── argocd_degraded.json
│   ├── trivy_clean.json
│   └── trivy_critical.json
```

Then tests can reuse realistic data.

---

# 180. Mock Fixture Naming

Good:

```text
argocd_synced_healthy
argocd_out_of_sync
deployment_ready
deployment_not_ready
trivy_critical
```

Avoid:

```text
data1
data2
mock1
mock2
```

Readable test data improves troubleshooting.

---

# 181. Avoid Mock Chains

This:

```python
client.foo.return_value.bar.return_value.baz.return_value.status
```

is usually a smell.

It indicates:

```text
deep coupling
```

Prefer:

```text
simple response object
```

or:

```text
adapter
```

---

# 182. Example of Mock Chain Problem

Bad:

```python
argocd.get_app.return_value.status.sync.status = \
    "Synced"
```

This can work, but it tightly couples the test to internal object navigation.

Better:

```python
app = SimpleNamespace(
    status=SimpleNamespace(
        sync=SimpleNamespace(
            status="Synced"
        )
    )
)

argocd.get_app.return_value = app
```

Even better: parse into an internal model and test the model.

---

# 183. Adapter Pattern

Create:

```text
ArgoCDClient
```

that converts:

```text
external API response
```

into:

```text
ApplicationStatus
```

Then business logic uses:

```python
status.sync
status.health
```

rather than raw API dictionaries everywhere.

---

# 184. Unit Testing Adapter

Test:

```text
raw response -> ApplicationStatus
```

with mocked HTTP/API behavior.

---

# 185. Unit Testing Business Logic

Use:

```text
ApplicationStatus
```

as real test data.

No ArgoCD mock is required.

This makes tests:

```text
faster
clearer
less brittle
```

---

# 186. Mocking at the Correct Boundary

A useful rule:

```text
Mock the boundary.
Keep the business logic real.
```

Example:

```text
AWS API
   |
 [mock]
   |
AWS adapter
   |
business logic
```

Do not mock the business logic itself.

---

# 187. Testing Through Interfaces

Define an interface/protocol:

```python
from typing import Protocol


class DeploymentClient(Protocol):

    def deploy(
        self,
        service: str,
        version: str
    ):
        ...
```

Production implementation:

```text
KubernetesDeploymentClient
```

Test implementation:

```text
Mock/spec client
```

This improves maintainability.

---

# 188. `Protocol` and Mocking

A `Protocol` can document the dependency contract.

Then tests can use a mock with the expected interface.

This is especially useful for large automation systems.

---

# 189. Mocking Database Transactions

Test:

```text
begin
commit
rollback
```

Example:

```python
db = Mock()

db.commit.assert_called_once()
```

For failures:

```python
db.execute.side_effect = \
    RuntimeError("DB failure")
```

Verify rollback.

---

# 190. Transaction Safety Test

Expected:

```text
execute succeeds
 ↓
commit
```

Failure:

```text
execute fails
 ↓
rollback
```

Unit-test both paths.

---

# 191. Mocking Message Queues

For:

```text
RabbitMQ
SQS
Kafka
```

test:

```text
publish
consume
ack
nack
retry
dead-letter decision
```

Use mocks for the broker client in unit tests.

Integration tests validate real message delivery.

---

# 192. RabbitMQ Example

Mock:

```python
rabbit = Mock()

rabbit.publish.return_value = True
```

Test:

```text
message payload
routing key
exchange
```

and failure behavior.

---

# 193. SQS Example

Mock:

```text
send_message
receive_message
delete_message
```

Test:

```text
successful processing
processing failure
visibility timeout policy
dead-letter handling
```

---

# 194. Testing Message Idempotency

If the same message is delivered twice:

```text
same event ID
```

the application should not perform duplicate destructive work if the design requires idempotency.

Mock the state store and verify behavior.

---

# 195. Mocking Email/Slack

Use a mock notification client.

Verify:

```text
recipient/channel
message type
service
environment
release ID
```

Avoid asserting large exact strings unless the message format itself is a contract.

---

# 196. Mocking PagerDuty

For critical alerts:

```text
trigger
resolve
deduplicate
```

Test that duplicate failures do not create unintended duplicate incidents when deduplication is part of the design.

---

# 197. Mocking Metrics Client

If your Python service emits metrics:

```text
counter
gauge
histogram
```

mock the metrics client and verify important increments/observations.

But do not over-test the metrics library itself.

---

# 198. Mocking OpenTelemetry/Jaeger

If another project uses them, mock exporters/clients at the boundary.

For your current observability stack, focus primarily on:

```text
Prometheus
Grafana
ELK
```

and keep observability logic separate from core deployment decisions.

---

# 199. Mocking Logging

Python's `caplog` fixture can often be preferable to mocking the logging library.

Example:

```python
def test_release_failed(caplog):

    ...

    assert "release failed" \
        in caplog.text
```

Use this when verifying logging behavior.

---

# 200. `caplog` vs Mock

Use:

```text
caplog
```

for:

```text
log content/levels
```

Use:

```text
Mock
```

for:

```text
external logger dependency interactions
```

---

# 201. Mocking File Upload APIs

Example:

```text
Artifactory
S3
GitHub
```

Test:

```text
correct artifact
correct destination
correct metadata
failure behavior
```

No actual upload in unit tests.

---

# 202. Mocking Security Scanner Process

If Python invokes:

```text
trivy
sonar-scanner
veracode
```

mock `subprocess.run`.

Return:

```text
exit code
stdout
stderr
```

Test how your application interprets them.

---

# 203. Scanner Exit Code Tests

Example:

```text
0 -> success
1 -> findings/failure
other -> scanner execution error
```

The exact meaning depends on the tool configuration.

Unit-test your configured interpretation rather than assuming every tool uses identical exit semantics.

---

# 204. Mocking Scanner Output

Test:

```text
clean
critical findings
malformed JSON
empty output
scanner unavailable
```

---

# 205. Mocking Git Diff

A deployment script may check:

```text
application code changed
infra changed
manifest changed
```

Mock Git output.

Test:

```text
relevant change -> deployment
irrelevant change -> skip
```

---

# 206. Mocking Change Detection

Example:

```python
def deployment_required(
    files
):

    return any(
        path.startswith("services/")
        for path in files
    )
```

This logic should be unit-tested with real lists.

No Git mock is needed for the pure function.

---

# 207. Mock Only the Git Boundary

```text
Git command/API
      ↓
mock
      ↓
file list
      ↓
real change-detection logic
```

This is a strong test design.

---

# 208. Mocking Feature Rollouts

For canary deployments:

```text
5%
10%
25%
50%
100%
```

mock metrics/health results.

Test:

```text
advance
pause
rollback
```

according to policy.

---

# 209. Mocking Canary Analysis

Example:

```text
error rate
latency
availability
```

Mock metric results:

```text
good -> continue
bad -> rollback
```

Test threshold boundaries.

---

# 210. Mocking Blue/Green

Test:

```text
blue healthy
green healthy
switch traffic
```

and:

```text
green unhealthy
```

should prevent switching or trigger rollback depending on policy.

---

# 211. Mocking ALB Target Health

Mock:

```text
healthy
unhealthy
initial
draining
```

Test traffic-switch decisions.

---

# 212. Mocking DNS Cutover

For Route53:

```text
old target
new target
```

unit-test:

```text
record construction
validation
rollback policy
```

Integration-test actual DNS changes.

---

# 213. Mocking RDS Migration Checks

Before deployment:

```text
migration complete?
```

Mock database response.

Test:

```text
complete -> continue
pending -> block
failed -> rollback/block
```

---

# 214. Mocking Kubernetes ConfigMap/Secret References

Do not mock the secret value itself if not needed.

Test:

```text
reference name
key
mount/env mapping
```

Keep actual secret material out of tests.

---

# 215. Mocking ConfigMap Data

For non-sensitive configuration:

```python
configmap = {
    "data": {
        "LOG_LEVEL": "INFO"
    }
}
```

Use real dictionaries.

---

# 216. Mocking Kubernetes Secrets

Use placeholders:

```text
TEST_SECRET
```

never production values.

Test:

```text
secret exists
secret missing
reference invalid
```

---

# 217. Mocking Namespace Creation

Test:

```text
namespace exists -> no create
namespace missing -> create
creation failure -> fail
```

This is another idempotency pattern.

---

# 218. Mocking Service/Ingress Creation

Test:

```text
resource exists
resource missing
update
delete
```

Verify API method and arguments.

---

# 219. Mocking ALB Ingress

Unit-test:

```text
host
path
service
port
annotations
```

without creating an actual ALB.

Integration tests should validate the actual ingress controller behavior.

---

# 220. Mocking Kubernetes Events

Test event handling:

```text
Normal
Warning
Failed
Scheduled
Pulled
Started
```

Verify the automation's interpretation.

---

# 221. Mocking Pod Status

Use controlled objects:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Test:

```text
deployment health
troubleshooting classification
```

---

# 222. Mocking Container States

Test:

```text
Waiting
Running
Terminated
```

and reasons:

```text
CrashLoopBackOff
OOMKilled
ImagePullBackOff
Error
```

This is highly relevant to DevOps troubleshooting automation.

---

# 223. Example Pod Classification

```python
def classify_pod(
    reason
):

    if reason == "OOMKilled":
        return "memory"

    if reason == "ImagePullBackOff":
        return "image"

    if reason == "CrashLoopBackOff":
        return "application"

    return "unknown"
```

Unit-test every reason.

---

# 224. Mocking Troubleshooting Automation

A troubleshooting script may:

```text
get pod
 ↓
get logs
 ↓
get events
 ↓
classify failure
 ↓
recommend action
```

Mock:

```text
Kubernetes API
```

and test the classification workflow.

---

# 225. Do Not Mock the Classification Logic

Use real:

```python
pod = ...
events = ...
logs = ...
```

Then:

```text
mock only Kubernetes retrieval
```

This gives more meaningful tests.

---

# 226. Mocking `kubectl` vs Kubernetes Client

If production code uses:

```text
kubectl subprocess
```

mock:

```text
subprocess.run
```

If production code uses:

```text
kubernetes Python client
```

mock:

```text
client methods
```

Test the same boundary your application actually uses.

---

# 227. Mocking Helm Upgrade

Test:

```text
release name
chart
namespace
values
timeout
```

and:

```text
success
failure
timeout
```

Do not execute Helm in a unit test.

---

# 228. Mocking Helm Rollback

Test:

```text
rollback command
release name
revision
```

and verify it occurs only when policy requires it.

---

# 229. Mocking Terraform Plan

Test:

```text
plan success
plan failure
destroy detected
unexpected changes
```

A production-safe automation should potentially block dangerous plans.

---

# 230. Terraform Destroy Guard

Example:

```text
terraform plan
 ↓
destroy detected
 ↓
BLOCK
```

Unit-test:

```text
destroy count = 0 -> allowed
destroy count > 0 -> blocked
```

if that matches your organization's policy.

---

# 231. Mocking Terraform Apply

Test:

```text
approval
plan validation
apply command
```

Actual infrastructure mutation belongs in integration/E2E.

---

# 232. Mocking Ansible Failure

Test:

```text
host unreachable
task failed
partial success
```

and verify the release workflow.

---

# 233. Mocking Parallel Host Execution

If multiple servers are configured:

```text
host A -> success
host B -> failure
host C -> success
```

test aggregation logic.

Do not rely on actual parallel SSH in unit tests.

---

# 234. Mocking SSH

Unit-test:

```text
command construction
host selection
key path
timeout
failure handling
```

Actual SSH should be integration-tested with disposable hosts.

---

# 235. Mocking Network Reachability

Instead of pinging real infrastructure:

```python
network = Mock()

network.check.return_value = True
```

Test:

```text
reachable
unreachable
timeout
```

---

# 236. Mocking DNS Lookup

Use controlled responses:

```text
record found
record missing
NXDOMAIN
timeout
```

Test application behavior.

---

# 237. Mocking TLS Certificate Checks

Test:

```text
valid
expired
hostname mismatch
untrusted
```

using controlled certificate metadata.

Actual endpoint verification can be integration-tested.

---

# 238. Mocking API Authentication

Test:

```text
token exists
token missing
token expired
refresh success
refresh failure
```

Do not put real credentials in tests.

---

# 239. Mocking OAuth

For OAuth flows:

```text
authorization response
token response
refresh response
invalid token
```

mock the HTTP boundary.

---

# 240. Mocking JWT Validation

For application policy, use deterministic test tokens/claims.

Test:

```text
valid
expired
wrong issuer
wrong audience
missing role
```

Use a real token library where appropriate rather than mocking the validation algorithm itself.

---

# 241. Mocking Encryption

Do not mock cryptographic correctness if the library itself is trusted.

Instead:

```text
unit-test your usage/configuration
```

and use integration/security tests for actual encryption workflows.

---

# 242. What Not to Mock

Avoid mocking:

```text
simple Python functions
simple dictionaries
simple lists
pure business logic
standard calculations
```

unless there is a specific reason.

---

# 243. What to Mock

Usually mock:

```text
network
cloud SDKs
Kubernetes APIs
Git providers
CI/CD APIs
external services
time
randomness
subprocess
message brokers
databases
```

when the goal is isolated unit testing.

---

# 244. Mocking Decision Tree

Ask:

```text
Is this dependency external?
        |
       yes
        |
Can the test safely use it?
        |
       no
        |
Mock/fake it
```

If:

```text
pure deterministic function
```

use the real function.

---

# 245. Mocking Decision Tree for DevOps

```text
Business rule?
   -> real code

Data transformation?
   -> real code

AWS API?
   -> mock

Kubernetes API?
   -> mock

ArgoCD API?
   -> mock

GitHub API?
   -> mock

Jenkins API?
   -> mock

Prometheus API?
   -> mock

kubectl subprocess?
   -> mock

Real test cluster?
   -> integration
```

---

# 246. Mocking Interview Question

### Why do we mock external services?

Answer:

```text
To isolate the unit under test, avoid real side effects,
make tests deterministic, reduce execution time, and
simulate failures that are difficult or unsafe to reproduce
against real infrastructure.
```

---

# 247. Interview — Mock vs Integration

### Question

Why not just use integration tests?

Answer:

```text
Integration tests provide valuable confidence that
components work together, but they are slower, more
expensive, environment-dependent, and harder to run
for every code change.

Unit tests give fast feedback on business logic.
Integration tests validate real dependency behavior.
```

---

# 248. Interview — Where to Patch?

### Question

Where should you patch?

Answer:

```text
Patch the name where the system under test looks it up,
not necessarily the module where the dependency was
originally defined.
```

Example:

```text
module imports requests.get
        ↓
patch module.requests.get
```

or:

```text
module imports get directly
        ↓
patch module.get
```

---

# 249. Interview — Mocking AWS

### Question

How do you unit-test Boto3 code?

Answer:

```text
I isolate the AWS client behind an adapter or inject the
client as a dependency. Unit tests use a mock/spec client
with controlled responses and exceptions.

I verify service selection, method calls, arguments,
response parsing, and error handling.

Actual AWS behavior is covered by integration tests.
```

---

# 250. Interview — Mocking Kubernetes

### Question

How do you test Kubernetes automation?

Answer:

```text
I mock the Kubernetes API client at the adapter boundary.

I simulate resource-not-found, forbidden, server errors,
successful responses, and rollout states.

I verify API calls and then separately test the deployment
policy using real Python data structures.
```

---

# 251. Interview — Mocking ArgoCD

### Question

How would you mock ArgoCD?

Answer:

```text
I mock the ArgoCD client/API boundary and return controlled
application status responses such as Synced, OutOfSync,
Healthy, and Degraded.

I then test the release policy independently from the
client interaction.
```

---

# 252. Interview — Excessive Mocking

### Question

What is excessive mocking?

Answer:

```text
When tests replace so much of the system that they mainly
verify mock interactions instead of real behavior.

It can hide integration problems and make tests brittle.

I mock external boundaries and keep core business logic real.
```

---

# 253. Interview — `Mock` vs `MagicMock`

Answer:

```text
Mock provides standard mock behavior.

MagicMock additionally supports Python magic methods such
as context managers, iteration, indexing, and length.

I use MagicMock only when those behaviors are required.
```

---

# 254. Interview — `side_effect`

Answer:

```text
I use side_effect to raise exceptions, return different
values across calls, or execute controlled behavior.

It is particularly useful for testing retries, timeouts,
transient failures, and recovery logic.
```

---

# 255. Interview — `return_value`

Answer:

```text
return_value defines what the mocked callable returns.

For example, an ArgoCD client's get_status() can return
Healthy without contacting a real ArgoCD server.
```

---

# 256. Interview — `assert_called_once_with`

Answer:

```text
It verifies both that a dependency was called exactly once
and that it received the expected arguments.
```

---

# 257. Interview — `autospec`

Answer:

```text
autospec creates a mock based on the real callable/object
interface, which helps catch incorrect method names,
arguments, and interface assumptions.
```

---

# 258. Interview — Mocking Time

### Question

How do you test code using `datetime.now()`?

Strong answer:

```text
Prefer dependency injection for time when possible.

The business function accepts a current-time value or clock
dependency.

For legacy code, I can patch the lookup location.

This avoids real-time-dependent tests and flakiness.
```

---

# 259. Interview — Retry Testing

### Question

How do you test retry logic?

Answer:

```text
I use side_effect to return a sequence such as:

503
503
200

Then I verify three attempts.

I also test retry exhaustion and non-retryable errors such
as 401 or 403 according to the application policy.
```

---

# 260. Interview — Production Safety

### Question

How would you test that Python automation cannot accidentally deploy to production?

Answer:

```text
I separate environment/account validation from the API call.

Unit tests verify:

wrong account -> block
wrong cluster -> block
missing approval -> block
dry-run -> no mutation
correct environment -> allowed

Then integration tests validate the actual environment
identity and API behavior.
```

---

# 261. Interview — Mocking vs Fake

### Question

When would you use a fake instead of a mock?

Answer:

```text
I use a fake when the dependency has meaningful stateful
behavior that is easier to model with a small working
implementation.

For example, an in-memory deployment repository can be
more readable than a large set of mock interactions.
```

---

# 262. Interview — Mocking a Message Queue

Answer:

```text
Unit tests mock publish/consume operations and validate
payloads, retry behavior, acknowledgement logic, and
dead-letter decisions.

Integration tests verify actual broker behavior.
```

---

# 263. Interview — Mocking CI/CD

### Question

How do you test Jenkins automation?

Answer:

```text
I mock the Jenkins API client.

Unit tests verify job selection, parameters, authentication
handling, response parsing, and failure policy.

I use a test Jenkins environment for integration testing.
```

---

# 264. Interview — Test Pyramid

Answer:

```text
Most tests should be fast unit tests.

A smaller set should be integration tests.

Only a small number should be full E2E tests.

Mocking supports the unit layer by isolating external
dependencies.
```

---

# 265. Production Mocking Architecture

Recommended structure:

```text
src/
└── devops_automation/
    ├── policies/
    │   ├── deployment.py
    │   ├── rollback.py
    │   └── security.py
    │
    ├── clients/
    │   ├── aws.py
    │   ├── kubernetes.py
    │   ├── argocd.py
    │   ├── github.py
    │   ├── jenkins.py
    │   └── prometheus.py
    │
    ├── workflows/
    │   └── release.py
    │
    └── models/
```

Tests:

```text
tests/
├── unit/
│   ├── test_policies.py
│   ├── test_clients.py
│   └── test_workflows.py
│
├── integration/
└── e2e/
```

---

# 266. Recommended Mock Boundary

```text
                 Workflow
                    |
          +---------+---------+
          |                   |
      Business              Clients
       Logic                  |
          |          +--------+--------+
          |          |        |        |
          |         AWS      EKS     ArgoCD
          |
      Unit Tests
          |
    Real business logic
          +
    Mocked boundaries
```

---

# 267. Production Example — Deployment

Imagine:

```text
Jenkins
  |
  v
Python Release Automation
  |
  +-- Validate environment
  |
  +-- Get image digest
  |
  +-- Update Git
  |
  +-- Wait for ArgoCD
  |
  +-- Check EKS
  |
  +-- Check Prometheus
  |
  +-- Rollback if required
```

Unit tests should mock:

```text
ECR
GitHub
ArgoCD
Kubernetes
Prometheus
```

and test the release decision logic.

---

# 268. Production Example — Failure

Scenario:

```text
Image = valid
Git = updated
ArgoCD = Synced
Kubernetes = Healthy
Error rate = 5%
Threshold = 1%
```

Expected:

```text
rollback
```

Unit test:

```text
mock external statuses
run real rollback policy
assert rollback decision
```

---

# 269. Production Example — Wrong AWS Account

Scenario:

```text
Expected account:
production

Actual:
development
```

Expected:

```text
BLOCK
```

Unit test:

```text
real account validation logic
mock STS response
```

This is an excellent production safety test.

---

# 270. Production Example — Kubernetes 404

Scenario:

```text
Deployment does not exist
```

Expected:

```text
create deployment
```

Unit test:

```text
mock read -> 404
verify create called
```

---

# 271. Production Example — Kubernetes 403

Scenario:

```text
Forbidden
```

Expected:

```text
fail immediately
```

Usually:

```text
do not retry indefinitely
```

Test:

```text
mock 403
assert correct exception/policy
assert no repeated calls
```

---

# 272. Production Example — ArgoCD Degraded

Scenario:

```text
sync = Synced
health = Degraded
```

Expected:

```text
deployment not healthy
```

The unit test must not incorrectly treat:

```text
Synced
```

as equivalent to:

```text
Healthy
```

---

# 273. Production Example — Prometheus Timeout

Scenario:

```text
deployment healthy
metrics unavailable
```

Policy could be:

```text
fail closed
```

or:

```text
mark verification unknown
```

The correct behavior depends on your release policy.

Unit-test whichever policy you choose.

---

# 274. Production Example — Retry

Scenario:

```text
ArgoCD API
503
503
200
```

Expected:

```text
retry twice
success on third attempt
```

Mock:

```python
side_effect = [
    error_503,
    error_503,
    success
]
```

Verify:

```text
3 calls
```

---

# 275. Production Example — Retry Exhaustion

```text
503
503
503
```

Expected:

```text
release verification failed
```

Verify:

```text
max attempts respected
```

---

# 276. Production Example — Dry Run

Command:

```bash
python deploy.py --dry-run
```

Expected:

```text
validation runs
manifest generated
plan displayed
NO production mutation
```

Mock:

```text
create
update
delete
```

and assert:

```text
not called
```

---

# 277. Production Example — Approval

Scenario:

```text
environment = prod
approval = missing
```

Expected:

```text
deployment blocked
```

Mock deployment client.

Verify:

```python
deploy.assert_not_called()
```

---

# 278. Production Example — Secret Failure

Scenario:

```text
secret retrieval fails
```

Expected:

```text
deployment blocked
```

and:

```text
secret value not logged
```

Unit-test both behavior and redaction.

---

# 279. Production Example — Security Gate

Scenario:

```text
Trivy critical = 1
```

Expected:

```text
release blocked
```

Mock scanner result.

Verify:

```text
deployment client not called
```

---

# 280. Production Example — Terraform Destroy

Scenario:

```text
terraform plan
destroy = 2
```

Expected:

```text
approval/block
```

according to policy.

Unit-test the decision without executing Terraform.

---

# 281. Production Example — Image Digest

Scenario:

```text
Expected digest = sha256:A
Actual digest   = sha256:B
```

Expected:

```text
deployment blocked
```

This is a strong supply-chain guardrail.

---

# 282. Production Example — GitOps Drift

Scenario:

```text
Git desired = version 1.4.2
Cluster = version 1.4.1
```

Expected:

```text
drift detected
```

Test:

```text
real drift comparison logic
mock Git/Kubernetes retrieval
```

---

# 283. Production Example — Duplicate Release

Scenario:

```text
same release request received twice
```

Expected:

```text
second operation becomes no-op
```

Mock state store.

Verify:

```text
external deployment call occurs only once
```

---

# 284. Production Example — Stale Event

Scenario:

```text
Release already SUCCESS
late FAILED event arrives
```

Expected:

```text
do not revert terminal state
```

Test with real state machine logic.

---

# 285. Production Example — Notification Failure

Scenario:

```text
deployment fails
Slack API fails
```

Expected behavior should be explicitly defined.

Possible:

```text
release remains failed
notification failure recorded separately
```

Do not let notification failure accidentally hide the real deployment failure.

---

# 286. Production Example — Monitoring Failure

Scenario:

```text
deployment healthy
Prometheus unavailable
```

Possible policy:

```text
verification = UNKNOWN
```

Then:

```text
release blocked
```

if verification is mandatory.

Unit-test this policy.

---

# 287. Mocking and Observability

Mocks should allow tests to simulate:

```text
success
latency
timeout
errors
empty responses
malformed responses
```

Observability-related logic should be testable without a live monitoring stack.

---

# 288. Mocking and Security

Mocking should never become a reason to skip security behavior.

Test:

```text
authorization
secret handling
production guardrails
scanner gates
artifact verification
```

with controlled inputs.

---

# 289. Mocking and Reliability

Reliability testing requires failures.

Mocks make it possible to simulate:

```text
timeouts
5xx
rate limits
partial failure
dependency unavailable
```

quickly and deterministically.

---

# 290. Mocking and Disaster Recovery

For rollback automation, mock:

```text
deployment failure
previous revision
rollback API
rollback failure
```

Test:

```text
rollback success
rollback failure
```

and resulting state.

---

# 291. Mocking Rollback Failure

Scenario:

```text
deployment failed
rollback attempted
rollback failed
```

Expected state might be:

```text
ROLLBACK_FAILED
```

and:

```text
critical notification
```

Unit-test the state transition.

---

# 292. Mocking Multi-Service Deployment

For:

```text
user
catalog
cart
orders
payment
inventory
notification
```

mock service health responses.

Test:

```text
all healthy
one unhealthy
multiple unhealthy
```

The workflow should follow its documented policy.

---

# 293. Service Dependency Failure

Example:

```text
payment healthy
inventory unhealthy
```

If orders depends on inventory:

```text
orders release blocked
```

Unit-test dependency graph logic separately from Kubernetes/API calls.

---

# 294. Dependency Graph Testing

Represent:

```text
orders -> inventory
orders -> payment
```

Test:

```text
dependency healthy -> allowed
dependency unhealthy -> blocked
```

---

# 295. Mocking Deployment Order

If the system requires:

```text
inventory
 ↓
orders
 ↓
notification
```

mock completion results and verify order when ordering is part of the contract.

Do not assert incidental call order that the design does not require.

---

# 296. Mocking Parallel Deployments

If independent services deploy concurrently:

```text
catalog
payment
notification
```

test aggregation:

```text
all success
one failure
```

Do not over-test scheduler implementation details.

---

# 297. Mocking Canary Metrics

Example:

```text
5xx rate = 0.5%
latency = 150ms
```

Policy:

```text
thresholds
```

Test:

```text
pass
fail
boundary
```

Use real numeric values, not mocks, for the pure policy function.

---

# 298. Mocking Metric Retrieval

Mock:

```text
Prometheus query client
```

Return:

```text
0.005
```

Then pass that value to:

```text
real policy
```

This is cleaner than mocking the policy itself.

---

# 299. The Best Mocking Pattern

```text
External API
     |
   [Mock]
     |
  API Adapter
     |
 Real parser
     |
 Real business logic
     |
 Expected decision
```

This provides meaningful coverage.

---

# 300. Anti-Pattern

Avoid:

```text
Mock API
Mock adapter
Mock parser
Mock policy
Assert mock called
```

This can produce a test that passes even if the actual business behavior is broken.

---

# 301. Test Real Logic, Mock External State

The guiding principle:

```text
Mock external behavior.
Execute your application logic for real.
```

---

# 302. Mocking Checklist

Before adding a mock, ask:

```text
Why am I mocking this?
Is it external?
Does it cause side effects?
Is it slow?
Is it nondeterministic?
Can I use a real simple object instead?
Would a fake be clearer?
Am I verifying meaningful behavior?
```

---

# 303. Mocking Checklist — DevOps

```text
[ ] AWS API
[ ] Kubernetes API
[ ] ArgoCD API
[ ] GitHub API
[ ] Jenkins API
[ ] Prometheus API
[ ] ELK API
[ ] Docker subprocess
[ ] Helm subprocess
[ ] Terraform subprocess
[ ] Ansible subprocess
[ ] SSH
[ ] Message queues
[ ] Database
[ ] Time
[ ] Sleep
[ ] Randomness
[ ] Environment variables
```

---

# 304. Mocking Mistakes Checklist

```text
[ ] Wrong patch target
[ ] Excessive mocking
[ ] Mocking pure logic
[ ] Mocking simple data
[ ] Deep mock chains
[ ] No spec/autospec
[ ] Real sleep
[ ] Real network
[ ] Real credentials
[ ] Real production data
[ ] Testing implementation instead of behavior
[ ] Verifying incidental calls
```

---

# 305. Troubleshooting Mock Not Working

Symptom:

```text
Real API is being called
```

Check:

```text
Did I patch the correct lookup path?
```

Remember:

```text
patch where used
```

not necessarily:

```text
patch where defined
```

---

# 306. Troubleshooting Wrong Call Count

If:

```python
assert mock.call_count == 1
```

fails:

Check:

```text
retry loop
fixture scope
setup code
multiple branches
parallel calls
```

Then decide whether:

```text
code
```

or:

```text
test expectation
```

is wrong.

---

# 307. Troubleshooting Unexpected Mock Attribute

If:

```text
mock.some_method
```

exists when it should not:

Use:

```text
spec
autospec
spec_set
```

to catch interface mistakes.

---

# 308. Troubleshooting Async Mock

If you get:

```text
coroutine was never awaited
```

check whether the dependency is async.

Use:

```text
AsyncMock
```

and:

```text
assert_awaited_once()
```

where appropriate.

---

# 309. Troubleshooting `side_effect`

If:

```python
side_effect = [
    response1,
    response2
]
```

is exhausted, a subsequent call can fail because the sequence has no remaining values.

Set enough responses for the expected calls or use a callable side effect.

---

# 310. Callable `side_effect`

Example:

```python
def response_for(
    value
):

    if value == "prod":
        return "Healthy"

    return "Unknown"
```

Then:

```python
mock.status.side_effect = \
    response_for
```

The callable receives the arguments passed to the mock.

---

# 311. Dynamic Side Effects

Useful for:

```text
input-dependent behavior
```

Example:

```text
dev -> success
prod -> approval required
```

Test multiple inputs while retaining controlled behavior.

---

# 312. Mocking Exception Sequence

Example:

```python
client.call.side_effect = [
    TimeoutError(),
    TimeoutError(),
    "success"
]
```

This is excellent for retry testing.

---

# 313. Verifying Recovery

After:

```text
failure
failure
success
```

verify:

```text
success returned
```

and:

```text
expected call count
```

Also verify:

```text
no unnecessary rollback
```

if recovery succeeded.

---

# 314. Testing Failure Classification

Use mocks to generate:

```text
401
403
404
429
500
503
timeout
connection error
```

Then assert:

```text
correct application classification
```

---

# 315. Mocking HTTP Headers

Test:

```text
Authorization
Content-Type
Accept
Idempotency-Key
User-Agent
```

when those headers are part of the API contract.

---

# 316. Idempotency-Key Testing

For API automation:

```python
client.create.assert_called_once_with(
    ...,
    headers={
        "Idempotency-Key": "release-123"
    }
)
```

This can help verify duplicate-request protection.

---

# 317. Mocking Correlation IDs

If requests carry:

```text
X-Request-ID
X-Correlation-ID
```

test that the workflow generates/propagates them correctly.

---

# 318. Mocking Trace IDs

If tracing is part of another implementation:

```text
trace ID
span ID
```

test propagation logic separately from the tracing backend.

---

# 319. Mocking Configuration Reload

If a service reloads configuration:

```text
old config
new config
```

mock the configuration provider.

Test:

```text
valid reload
invalid reload
rollback to previous config
```

---

# 320. Mocking Dynamic Secrets

If credentials rotate:

```text
old secret
new secret
```

test:

```text
refresh
retry
failure
```

without using real credentials.

---

# 321. Mocking Certificate Rotation

Test:

```text
certificate valid
certificate expiring
certificate expired
renewal failure
```

using controlled metadata.

---

# 322. Mocking AWS STS Role Assumption

Test:

```text
role assumption success
AccessDenied
expired credentials
```

without actually assuming a production role.

---

# 323. Mocking Multi-Account Deployment

Scenario:

```text
central CI
 ↓
assume role
 ↓
target account
```

Mock:

```text
STS
```

and test:

```text
correct role ARN
correct target account
wrong account blocked
```

---

# 324. Mocking Region Selection

Test:

```text
environment -> region
```

and:

```text
unsupported region
```

This prevents deploying resources into the wrong region.

---

# 325. Mocking VPC/Infrastructure Decisions

If Python decides:

```text
subnet
security group
VPC
```

unit-test the selection logic.

Actual AWS resource creation belongs to integration tests.

---

# 326. Mocking IAM Policy Generation

If Python generates policy JSON:

```text
actions
resources
conditions
```

unit-test the generated structure.

Use security testing to validate least privilege.

---

# 327. Mocking S3 Bucket Policy Generation

Unit-test:

```text
bucket
principal
actions
conditions
```

without applying it to AWS.

---

# 328. Mocking Kubernetes RBAC Generation

Unit-test:

```text
Role
RoleBinding
ServiceAccount
```

generation.

Verify:

```text
expected verbs
expected resources
expected namespace
```

Avoid granting more privileges than required.

---

# 329. Mocking Network Policy Generation

Unit-test:

```text
allowed ingress
allowed egress
namespace selectors
pod selectors
ports
```

Actual enforcement should be integration-tested in Kubernetes.

---

# 330. Mocking Ingress Generation

Unit-test:

```text
host
path
backend
TLS
annotations
```

Actual ALB behavior belongs to integration testing.

---

# 331. Mocking Service Generation

Test:

```text
ClusterIP
NodePort
LoadBalancer
```

according to your application's configuration.

Verify:

```text
ports
selectors
type
```

---

# 332. Mocking Deployment Generation

Test:

```text
replicas
image
ports
env
resources
probes
securityContext
```

This is one of the highest-value Kubernetes automation unit tests.

---

# 333. Mocking ConfigMap Generation

Test:

```text
required keys
environment
service configuration
```

Do not include secrets.

---

# 334. Mocking Secret References

Test:

```text
secretName
key
env variable
volume mount
```

without exposing secret values.

---

# 335. Mocking Helm Values

Test generated values:

```text
image.repository
image.tag
replicas
resources
ingress
environment
```

Actual Helm rendering can be integration-tested.

---

# 336. Mocking ArgoCD Application YAML

Unit-test:

```text
repoURL
path
revision
destination
namespace
sync policy
```

Actual ArgoCD behavior belongs to integration/E2E.

---

# 337. Mocking GitHub Actions Workflow Generation

Unit-test:

```text
branch
environment
image tag
secrets references
```

Do not embed actual secrets.

---

# 338. Mocking Jenkinsfile Parameters

If Python generates pipeline parameters, unit-test:

```text
name
type
default
allowed values
```

---

# 339. Mocking Pipeline Approval

Test:

```text
manual approval
automatic approval
approval timeout
rejection
```

using controlled approval service responses.

---

# 340. Mocking Pipeline Cancellation

Test:

```text
user cancellation
timeout cancellation
dependency cancellation
```

and verify cleanup behavior.

---

# 341. Mocking Cleanup

If deployment fails:

```text
temporary files
locks
test resources
```

may need cleanup.

Mock cleanup dependencies and verify they run when required.

---

# 342. Cleanup Failure

Test:

```text
main operation failed
cleanup also failed
```

Expected behavior should preserve the original failure while recording cleanup failure.

---

# 343. Mocking Temporary Files

Prefer:

```text
tmp_path
```

for real temporary filesystem behavior.

Mock only when you need to verify interaction with a filesystem abstraction.

---

# 344. Mocking Artifact Metadata

Test:

```text
version
commit SHA
image digest
build timestamp
```

using controlled metadata.

---

# 345. Mocking Git Commit SHA

If release metadata uses:

```python
subprocess
```

mock the Git command.

Test:

```text
correct SHA included in release metadata
```

---

# 346. Mocking Build Metadata

Test:

```text
build number
commit
image
environment
```

and ensure release records are complete.

---

# 347. Mocking Release Audit

Example:

```text
service = payment
version = 1.4.2
environment = prod
result = success
```

Verify audit event generation.

---

# 348. Mocking Change Management

If deployment requires a ticket:

```text
ticket exists
ticket approved
ticket expired
```

mock the change-management API.

---

# 349. Production Approval Chain

```text
Release
 ↓
Change ticket
 ↓
Approval
 ↓
Security gate
 ↓
Deployment
```

Unit-test the decision logic.

---

# 350. Mocking Compliance Checks

If automation checks:

```text
required labels
required approvals
required scans
```

mock the compliance provider.

Test:

```text
compliant
non-compliant
provider unavailable
```

---

# 351. Mocking Policy Engines

If external policy engine exists:

```text
allow
deny
unknown
```

mock responses.

Test fail-open/fail-closed behavior.

---

# 352. Mocking OPA

For policy evaluation:

```text
input
policy response
```

unit-test:

```text
allow
deny
missing decision
```

Actual OPA integration can be tested separately.

---

# 353. Mocking Vault

If using Vault:

```text
secret success
permission denied
secret expired
Vault unavailable
```

Mock Vault client.

Do not use production Vault credentials in unit tests.

---

# 354. Mocking CloudWatch

If another project uses CloudWatch:

```text
metric response
alarm state
API error
```

can be mocked.

For your current notes, keep the main observability examples centered on:

```text
Prometheus
Grafana
ELK
```

---

# 355. Mocking Grafana

If automation queries Grafana APIs:

```text
dashboard
alert
annotation
```

mock the API client.

Test policy/response parsing.

---

# 356. Mocking ELK Search

Test:

```text
query construction
result parsing
error count
```

mock Elasticsearch client/API.

---

# 357. Mocking Log Correlation

If Python correlates:

```text
release ID
pod name
error logs
```

use controlled log records.

Test:

```text
matching
non-matching
missing correlation ID
```

---

# 358. Mocking Health Checks

Health checker may call:

```text
HTTP
Kubernetes
Prometheus
ArgoCD
```

Mock each dependency.

Test:

```text
healthy
unhealthy
timeout
unknown
```

---

# 359. Health Aggregation

Example:

```text
Kubernetes = Healthy
ArgoCD = Healthy
Prometheus = Healthy
```

Expected:

```text
release healthy
```

One unhealthy:

```text
release unhealthy
```

Use real boolean logic and controlled dependency results.

---

# 360. Mocking Readiness vs Liveness

Test that:

```text
readiness failure
```

is not automatically interpreted as:

```text
application crash
```

unless policy explicitly maps them.

---

# 361. Mocking OOMKilled

Controlled Kubernetes status:

```text
reason = OOMKilled
```

Expected:

```text
memory-related classification
```

Then remediation recommendation can be tested separately.

---

# 362. Mocking CrashLoopBackOff

Controlled status:

```text
CrashLoopBackOff
```

Expected:

```text
application failure classification
```

---

# 363. Mocking ImagePullBackOff

Expected:

```text
image/registry classification
```

Potential checks:

```text
image
tag
registry
credentials
```

---

# 364. Mocking Node Pressure

If automation reads node conditions:

```text
MemoryPressure
DiskPressure
PIDPressure
```

test classification logic with controlled node objects.

---

# 365. Mocking Disk Usage

If Python checks:

```text
disk usage > threshold
```

mock command output or use a controlled data provider.

Test:

```text
below
equal
above
```

---

# 366. Mocking Memory Usage

Same pattern:

```text
memory < threshold
memory = threshold
memory > threshold
```

Test alert/restart policy.

---

# 367. Mocking Process Health

If automation checks:

```text
process running
process stopped
process zombie
```

mock process inspection.

Actual host-level behavior belongs in integration tests.

---

# 368. Mocking Network Ports

Test:

```text
port open
port closed
timeout
```

using controlled network client responses.

---

# 369. Mocking Service Discovery

For:

```text
DNS
Consul
Kubernetes service discovery
```

mock the lookup.

Test:

```text
found
missing
multiple endpoints
```

---

# 370. Mocking Load Balancer Routing

Test:

```text
healthy target
unhealthy target
no healthy targets
```

and traffic decision.

---

# 371. Mocking Deployment Strategies

Test policy for:

```text
Rolling
Blue/Green
Canary
Recreate
```

with controlled health data.

---

# 372. Mocking Rollout Pause

If canary pauses at:

```text
10%
```

mock approval/metrics.

Test:

```text
continue
pause
rollback
```

---

# 373. Mocking Rollout Resume

Test:

```text
resume approved
resume denied
resume expired
```

---

# 374. Mocking Rollback Revision

If previous version is:

```text
1.4.1
```

verify rollback chooses:

```text
1.4.1
```

not an arbitrary tag.

---

# 375. Mocking Release History

Mock:

```text
current release
previous successful release
failed releases
```

Test selection of rollback target.

---

# 376. Mocking Database Migration Status

Test:

```text
migration complete
migration pending
migration failed
```

and deployment policy.

---

# 377. Mocking Application Compatibility

If deployment requires:

```text
database schema >= version
```

unit-test compatibility logic.

---

# 378. Mocking Dependency Version Checks

Test:

```text
compatible
incompatible
unknown
```

without calling real package registries.

---

# 379. Mocking Package Registry

For:

```text
PyPI
npm
Maven
JFrog
```

mock version responses.

Test:

```text
package exists
version exists
version missing
registry unavailable
```

---

# 380. Mocking Vulnerability Database

If Python consumes vulnerability information:

```text
no critical
critical found
database unavailable
```

test policy.

---

# 381. Mocking License Checks

Test:

```text
approved license
forbidden license
unknown license
```

without calling a real license service.

---

# 382. Mocking Supply Chain Verification

Test:

```text
signature valid
signature invalid
artifact missing signature
```

using controlled verification results.

Actual cryptographic verification should use real libraries/integration tests where appropriate.

---

# 383. Mocking Artifact Signing

Unit-test:

```text
signing command construction
signature metadata
verification policy
```

Actual signing workflows should be tested securely outside ordinary unit tests.

---

# 384. Mocking SBOM Processing

Test:

```text
component count
critical vulnerabilities
license violations
```

using controlled SBOM data.

---

# 385. Mocking Deployment Metadata

Release object:

```text
service
version
commit
image
digest
environment
```

Use a real dataclass/model rather than a giant mock.

---

# 386. Dataclass Test Data

Example:

```python
from dataclasses import dataclass


@dataclass
class Release:

    service: str
    version: str
    environment: str
```

Test:

```python
release = Release(
    "payment",
    "1.4.2",
    "prod"
)
```

This is clearer than:

```python
Mock()
```

for ordinary domain data.

---

# 387. Mock External Service, Not Domain Model

Strong pattern:

```text
Real Release object
Real policy
Mock API client
```

This gives better test confidence.

---

# 388. Mocking Object Creation

If a class internally creates a client:

```python
client = KubernetesClient()
```

you can patch the constructor:

```python
@patch("module.KubernetesClient")
```

But dependency injection is usually cleaner for new code.

---

# 389. Constructor Mocking

If patched:

```python
mock_client_class.return_value
```

represents the created instance.

Configure:

```python
mock_client_class.return_value.get_status.return_value = \
    "Healthy"
```

---

# 390. Constructor Mock Verification

```python
mock_client_class.assert_called_once_with(
    expected_config
)
```

Use when construction itself is part of the behavior.

---

# 391. Mocking Class Methods

Patch:

```python
@patch.object(
    DeploymentClient,
    "deploy"
)
```

Then:

```python
mock_deploy.return_value = ...
```

Use carefully; dependency injection often produces cleaner tests.

---

# 392. Mocking Properties

For properties, `PropertyMock` may be appropriate.

```python
from unittest.mock import PropertyMock
```

Use only when property access itself needs controlled behavior.

---

# 393. Avoid Mocking Properties Excessively

If you can create a real object:

```python
deployment.status = ...
```

that is usually clearer.

---

# 394. Mocking Static Methods

Possible with:

```text
patch
```

but if static methods make testing difficult, consider whether the design should use injected dependencies instead.

---

# 395. Mocking Classmethod Dependencies

Same principle:

```text
patch lookup location
```

and verify behavior.

---

# 396. Mocking Module-Level Constants

Use:

```python
patch(
    "module.TIMEOUT",
    30
)
```

But configuration injection is usually cleaner when the value is behaviorally important.

---

# 397. Mocking Configuration Safely

Prefer:

```text
explicit configuration object
```

over:

```text
global constants
```

for testable production automation.

---

# 398. Mocking Current Environment

If application reads:

```text
ENVIRONMENT
AWS_REGION
CLUSTER_NAME
```

inject a configuration object where practical.

Then unit tests can use:

```python
Config(
    environment="staging",
    region="ap-south-1"
)
```

---

# 399. Mocking Is a Design Feedback Tool

If mocking is extremely difficult:

```text
many patches
deep chains
complex setup
```

it may indicate:

```text
high coupling
hidden dependencies
large functions
mixed responsibilities
```

Refactoring may improve both code and tests.

---

# 400. Good DevOps Python Design

Separate:

```text
1. Configuration
2. Domain models
3. Policy
4. External clients
5. Workflow
6. CLI
```

Then each layer becomes easier to test.

---

# 401. CLI Layer

CLI should translate:

```text
arguments
```

into:

```text
configuration/workflow
```

Unit-test argument parsing and command decisions.

The workflow should not depend directly on `sys.argv`.

---

# 402. CLI Dependency Injection

Prefer:

```python
def main(
    args,
    client
):
    ...
```

over deeply embedding:

```text
AWS
Kubernetes
Git
```

inside `main()`.

---

# 403. Mocking CLI Commands

Test:

```text
--dry-run
--environment
--service
--version
--rollback
```

and verify workflow invocation.

---

# 404. Mocking Exit Codes

Test:

```text
success -> 0
validation failure -> 2
deployment failure -> non-zero
```

according to your CLI contract.

---

# 405. Mocking Signals

For long-running automation:

```text
SIGTERM
SIGINT
```

may trigger cleanup.

Unit-test cleanup logic separately from actual OS signal delivery.

---

# 406. Mocking Cancellation

Test:

```text
operation canceled
```

and verify:

```text
cleanup
state update
notification
```

---

# 407. Mocking Thread/Process Failure

If worker fails:

```text
worker exception
```

test parent workflow behavior.

Avoid testing actual multiprocessing mechanics in ordinary unit tests.

---

# 408. Mocking Queue Retry

Test:

```text
attempt 1
attempt 2
attempt 3
dead-letter
```

using controlled queue responses.

---

# 409. Mocking Dead-Letter Logic

If processing fails after max attempts:

```text
DLQ
```

Test:

```text
message moved
ack policy
alert
```

---

# 410. Mocking Scheduled Jobs

For scheduled automation:

```text
run now
skip
already running
```

test scheduling policy using controlled timestamps/state.

---

# 411. Mocking Lock Contention

Test:

```text
lock available
lock held
lock expired
```

without requiring real Redis/DynamoDB/Kubernetes Lease.

---

# 412. Mocking Distributed Coordination

Test:

```text
leader
follower
leader unavailable
```

at policy level.

Real distributed coordination requires integration testing.

---

# 413. Mocking Retry-After

HTTP response:

```text
429
Retry-After: 10
```

Test that application calculates intended delay.

Do not actually sleep 10 seconds.

---

# 414. Mocking HTTP Pagination

Example:

```text
page1 -> next page
page2 -> next page
page3 -> no next
```

Test all items are collected.

---

# 415. Mocking HTTP Redirects

If your application handles redirects:

```text
301
302
```

test:

```text
follow
reject
```

according to policy.

---

# 416. Mocking TLS Errors

Test:

```text
certificate verification failure
```

and verify:

```text
fail securely
```

Never make production code silently disable TLS verification just to satisfy a test.

---

# 417. Mocking Proxy Errors

Test:

```text
proxy unavailable
connection timeout
authentication failure
```

if the automation runs through corporate proxies.

---

# 418. Mocking DNS Failure

Test:

```text
NameResolutionError
```

and retry/failure policy.

---

# 419. Mocking Network Partition

Simulate:

```text
connection timeout
```

rather than disconnecting the developer machine.

---

# 420. Mocking Partial Responses

Example:

```text
API responds
but required field is missing
```

Test parser failure.

This is often more valuable than testing only successful responses.

---

# 421. Mocking Corrupt Data

Test:

```text
invalid JSON
invalid YAML
unexpected type
truncated response
```

and verify safe failure.

---

# 422. Mocking API Schema Version

If API supports:

```text
v1
v2
```

test your client behavior for supported versions.

---

# 423. Mocking Backward Compatibility

If response fields differ:

```text
old field
new field
```

test compatibility logic.

---

# 424. Mocking Deprecation

If API returns:

```text
deprecated endpoint
```

test:

```text
warning
fallback
migration path
```

according to application policy.

---

# 425. Mocking Service Unavailability

For external service:

```text
503
```

test:

```text
retry
fallback
fail
```

depending on policy.

---

# 426. Mocking Fallback

Example:

```text
Primary config service unavailable
 ↓
Fallback config
```

Test:

```text
primary success -> primary used
primary failure -> fallback used
```

---

# 427. Mocking Cache Fallback

Same pattern:

```text
API unavailable
 ↓
cache
```

Test stale-cache policy explicitly.

---

# 428. Mocking Stale Cache

Test:

```text
fresh
stale
missing
```

and policy.

---

# 429. Mocking Feature Rollback

If feature flag is enabled and causes errors:

```text
disable flag
```

Test policy without changing a real feature flag service.

---

# 430. Mocking Deployment Approval Expiry

Use fixed time.

Test:

```text
approval valid
approval expired
```

and verify deployment decision.

---

# 431. Mocking Multi-Region Deployment

If:

```text
region A
region B
```

are deployed sequentially:

```text
A success -> B
A failure -> stop
```

mock region results and test orchestration.

---

# 432. Mocking Multi-Region Rollback

Test:

```text
A success
B failure
```

then:

```text
rollback A
```

if that is the desired strategy.

---

# 433. Mocking Disaster Recovery

Test:

```text
primary unavailable
DR healthy
```

and failover decision.

Do not actually fail production infrastructure for unit tests.

---

# 434. Mocking Backup Validation

If automation checks backup status:

```text
backup available
backup missing
backup stale
```

mock the backup API.

Test recovery policy.

---

# 435. Mocking Restore Automation

Unit-test:

```text
restore command
source selection
target environment
validation
```

Actual restore should be integration-tested.

---

# 436. Mocking Data Migration

Test:

```text
migration pending
migration complete
migration failed
```

and release gating.

---

# 437. Mocking Database Connection Pool

Test:

```text
connection success
pool exhausted
connection timeout
```

using controlled client behavior.

---

# 438. Mocking Connection Cleanup

Verify:

```text
close
rollback
release
```

when required.

---

# 439. Mocking Context Managers

Use `MagicMock` when code uses:

```python
with connection:
    ...
```

Verify:

```text
enter
exit
commit/rollback
```

where those interactions matter.

---

# 440. Mocking Async Context Managers

Use:

```text
AsyncMock
```

and configure:

```text
__aenter__
__aexit__
```

for async resource management.

---

# 441. Mocking HTTP Client Sessions

If code uses:

```text
requests.Session
httpx.Client
aiohttp
```

patch the session/client lookup location.

Test:

```text
request
headers
timeout
response
close
```

---

# 442. Mocking Session Reuse

If connection reuse is important:

```text
one session
multiple requests
```

verify the expected construction/reuse behavior.

---

# 443. Mocking HTTP Retry Adapter

If using a library's built-in retry mechanism, don't unit-test the library.

Test:

```text
your configuration
```

and use integration tests to verify the configured behavior when necessary.

---

# 444. Mocking Third-Party Libraries

General rule:

```text
Do not test third-party library internals.
```

Test:

```text
your code's interaction with them
```

---

# 445. Mocking Library Version Changes

After upgrading:

```text
run unit tests
run integration tests
```

Mocks can catch interface changes, but only integration tests can prove the real dependency still behaves as expected.

---

# 446. Mocking Contract Boundaries

For external APIs:

```text
request contract
response contract
error contract
```

unit-test all three.

---

# 447. Mocking API Request Payload

Verify required fields:

```text
service
version
environment
image
approval
```

Do not assert irrelevant field ordering in JSON.

---

# 448. Mocking JSON Serialization

If serialization itself is part of your code, test it with real objects.

Mock only the external transport.

---

# 449. Mocking HTTP Transport

Ideal boundary:

```text
Business logic
 ↓
Client
 ↓
Transport [mock]
```

Then:

```text
Client behavior
```

can be tested without a network.

---

# 450. Mocking Notification Transport

Same pattern:

```text
Notification policy
 ↓
Notification client
 ↓
Slack/Email API [mock]
```

---

# 451. Mocking Metrics Transport

```text
Metrics policy
 ↓
Metrics client
 ↓
Prometheus [mock]
```

---

# 452. Mocking Deployment Transport

```text
Deployment policy
 ↓
Deployment client
 ↓
Kubernetes [mock]
```

This architecture creates clean unit-test boundaries.

---

# 453. The Main Principle

```text
Business logic = real
External infrastructure = controlled
```

---

# 454. Final Mocking Checklist

```text
Fundamentals
[ ] Mock
[ ] MagicMock
[ ] AsyncMock
[ ] patch
[ ] patch.object
[ ] return_value
[ ] side_effect
[ ] call_count
[ ] call_args
[ ] assert_called_once
[ ] assert_called_once_with
[ ] assert_not_called
[ ] assert_has_calls

Interface Safety
[ ] spec
[ ] spec_set
[ ] autospec

DevOps
[ ] AWS
[ ] EKS
[ ] Kubernetes
[ ] ArgoCD
[ ] GitHub
[ ] Jenkins
[ ] Prometheus
[ ] ELK
[ ] Terraform
[ ] Helm
[ ] Docker
[ ] Ansible
[ ] JFrog
[ ] SSH
[ ] RabbitMQ/SQS
[ ] Database

Reliability
[ ] timeout
[ ] retry
[ ] rate limit
[ ] polling
[ ] backoff
[ ] failure
[ ] recovery
[ ] idempotency
[ ] concurrency

Security
[ ] no production credentials
[ ] secret redaction
[ ] production guardrails
[ ] account validation
[ ] approval validation
[ ] security gates

Architecture
[ ] dependency injection
[ ] adapter boundaries
[ ] real business logic
[ ] controlled external dependencies
[ ] unit/integration separation
```

---

# 455. Final Takeaway

Mocking is not:

```text
"Make every dependency fake."
```

The correct mindset is:

```text
Isolate external side effects
while executing meaningful application logic for real.
```

For DevOps Python automation:

```text
AWS       -> mock boundary
Kubernetes-> mock boundary
ArgoCD    -> mock boundary
GitHub    -> mock boundary
Jenkins   -> mock boundary
Prometheus-> mock boundary
ELK       -> mock boundary

Policies  -> real code
Validators-> real code
Parsers   -> real code
Models    -> real code
Decision  -> real code
```

That gives you:

```text
Fast feedback
+
Safe testing
+
Deterministic failures
+
Production-oriented coverage
```

---

# 456. Next File

```text
09-Python-Testing/
├── 01-Pytest-Fundamentals.md       ✓
├── 02-Unit-Testing.md              ✓
├── 03-Mocking.md                   ✓
├── 04-Test-Automation.md
└── 05-DevOps-Automation-Testing.md
```

Next:

## `04-Test-Automation.md`

The next topic should build on pytest + unit testing + mocking and cover:

```text
Automated test execution
Test suites
Test discovery
Markers
Fixtures at scale
Test configuration
Coverage
JUnit/HTML reports
Parallel execution
Test environments
CI/CD integration
Jenkins
GitHub Actions
Dockerized test environments
Kubernetes test jobs
Test data management
Integration test orchestration
Smoke tests
Regression tests
Pre-deployment tests
Post-deployment tests
Quality gates
Flaky test detection
Retries
Test artifacts
Notifications
Production-safe automation
Failure troubleshooting
DevOps project architecture
Senior interview scenarios
```
