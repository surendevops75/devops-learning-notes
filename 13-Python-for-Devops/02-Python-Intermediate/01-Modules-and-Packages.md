# Modules-and-Packages

## 1. Overview

As Python scripts grow beyond a few dozen lines, putting everything into one file becomes difficult to maintain.

DevOps automation commonly contains reusable functionality for:

- AWS
- Kubernetes
- Docker
- Terraform
- Linux
- CI/CD
- monitoring
- logging
- configuration
- APIs
- security tools

Python modules and packages allow us to organize that functionality into reusable components.

A practical DevOps automation project might look like:

```text
devops-automation/
├── scripts/
│   ├── check_disk.py
│   ├── aws_inventory.py
│   └── k8s_health.py
├── devops_utils/
│   ├── __init__.py
│   ├── aws.py
│   ├── kubernetes.py
│   ├── docker.py
│   ├── shell.py
│   └── logging_utils.py
├── tests/
│   ├── test_aws.py
│   └── test_kubernetes.py
├── requirements.txt
└── README.md
```

The goal is:

```text
write once → reuse everywhere → test independently → maintain centrally
```

---

## 2. What Is a Module?

A Python module is normally a `.py` file containing Python code.

```python
# server_utils.py

def check_server(server):
    print(f"Checking {server}")
```

Use it:

```python
import server_utils

server_utils.check_server("web-01")
```

---

## 3. Why Modules Matter in DevOps

Imagine five scripts all contain the same `run_command()` implementation. If you discover a bug, you may need to fix it in five places.

Instead create:

```text
shell.py
├── run_command()
├── check_command()
└── run_with_timeout()
```

Then every automation script imports the shared functions.

Benefits:

- less duplication
- centralized fixes
- consistent behavior
- easier testing
- easier troubleshooting
- easier code review

---

## 4. Creating and Importing a Module

Create `server_utils.py`:

```python
def check_server(server):
    print(f"Checking {server}")
```

Use:

```python
import server_utils

server_utils.check_server("web-01")
```

---

## 5. Import a Specific Function

```python
from server_utils import check_server

check_server("web-01")
```

Use this when the imported names are few and clear.

---

## 6. Import Aliases

```python
import datetime as dt
```

or:

```python
import server_utils as su

su.check_server("web-01")
```

Aliases are useful for long module names and conventional libraries.

---

## 7. Multiple Imports

```python
from server_utils import check_server, restart_service
```

Avoid very large import lists because dependencies become harder to understand.

---

## 8. Avoid Wildcard Imports

Avoid:

```python
from server_utils import *
```

It can cause:

- unclear dependencies
- name collisions
- difficult code review
- unclear ownership of functions

Prefer explicit imports.

---

## 9. Important Standard Library Modules for DevOps

```text
os          filesystem/environment
sys         runtime/arguments/exit codes
subprocess  system commands
pathlib     filesystem paths
shutil      file operations
json        APIs/configuration
re          regex/log parsing
logging     application/script logs
argparse    CLI arguments
datetime    timestamps/time zones
time        delays
socket      networking
urllib       HTTP
 tempfile   temporary files
```

Examples:

```python
import os
import sys
import subprocess
import json
import logging
```

---

## 10. `subprocess` — Core DevOps Use

```python
import subprocess

result = subprocess.run(
    ["uname", "-a"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

This allows Python to orchestrate commands such as:

```text
kubectl
helm
terraform
git
docker
aws
systemctl
df
du
curl
```

When a mature Python SDK exists, prefer the SDK when it gives better structured behavior than parsing CLI output.

---

## 11. `pathlib`

```python
from pathlib import Path

log_file = Path("/var/log/app.log")

print(log_file.exists())
```

`pathlib` is generally cleaner than manually manipulating path strings.

---

## 12. `json`

```python
import json

data = {
    "service": "payment",
    "replicas": 3,
}

text = json.dumps(data)
print(text)
```

Useful for AWS APIs, Kubernetes APIs, REST APIs and configuration.

---

## 13. `re`

```python
import re

text = "server=10.0.1.10"
match = re.search(r"\d+\.\d+\.\d+\.\d+", text)

if match:
    print(match.group())
```

Regex is covered deeply in `04-Regex.md`.

---

## 14. `logging`

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("Deployment started")
```

Production automation should normally use logging rather than scattered `print()` statements.

---

## 15. `argparse`

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--environment", required=True)
args = parser.parse_args()

print(args.environment)
```

This turns a Python file into a proper CLI utility. It is covered deeply in `08-Argparse.md`.

---

## 16. `datetime`

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
print(now)
```

Use timezone-aware timestamps in production systems. More detail is covered in `06-Datetime-and-Time.md`.

---

## 17. Module Search Path

Python searches locations in its module search path.

```python
import sys

for path in sys.path:
    print(path)
```

This is useful when troubleshooting `ModuleNotFoundError`.

---

## 18. `PYTHONPATH`

`PYTHONPATH` can add directories to the import search path:

```bash
export PYTHONPATH=/opt/devops/lib:$PYTHONPATH
```

However, production projects should generally prefer proper packages and controlled environments instead of relying heavily on manually configured `PYTHONPATH`.

---

## 19. `ModuleNotFoundError`

Example:

```python
import aws_utils
```

If Python cannot find it:

```text
ModuleNotFoundError: No module named 'aws_utils'
```

Troubleshoot:

```text
1. Check module name
2. Check project/package structure
3. Check current execution context
4. Check Python interpreter
5. Check virtual environment
6. Check installed dependencies
7. Check sys.path
```

---

## 20. `ImportError`

This can occur when the module exists but the requested name does not:

```python
from aws_utils import deploy
```

if `deploy` is not available.

---

## 21. `__name__`

Every Python module has `__name__`.

When run directly:

```python
print(__name__)
```

normally produces:

```text
__main__
```

When imported, it normally contains the module name.

---

## 22. `if __name__ == "__main__"`

Use:

```python
def main():
    print("Running deployment")


if __name__ == "__main__":
    main()
```

This allows the file to be both executed directly and imported without automatically executing the main workflow.

This pattern is essential for reusable DevOps scripts.

---

## 23. Recommended DevOps Script Structure

```python
import logging


def check_disk():
    ...


def main():
    check_disk()


if __name__ == "__main__":
    main()
```

Later we can add argument parsing, configuration, logging and meaningful exit codes.

---

## 24. Reusable Shell Module

Create `devops_utils/shell.py`:

```python
import subprocess


def run_command(command, timeout=30):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=False,
        timeout=timeout,
    )

    return {
        "command": command,
        "returncode": result.returncode,
        "stdout": result.stdout,
        "stderr": result.stderr,
    }
```

Use it:

```python
from devops_utils.shell import run_command

result = run_command(["kubectl", "get", "pods"])

if result["returncode"] != 0:
    raise RuntimeError(result["stderr"])
```

This is a realistic reusable DevOps pattern.

---

## 25. Shell Utility — Error Handling

```python
import subprocess


def run_command(command, timeout=30):
    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=False,
            timeout=timeout,
        )

        return {
            "returncode": result.returncode,
            "stdout": result.stdout,
            "stderr": result.stderr,
        }

    except subprocess.TimeoutExpired as exc:
        raise RuntimeError(
            f"Command timed out: {command}"
        ) from exc
```

Production automation should never be allowed to hang indefinitely.

---

## 26. Avoid `shell=True` With Untrusted Input

Avoid:

```python
subprocess.run(user_input, shell=True)
```

Prefer:

```python
subprocess.run(
    ["kubectl", "get", "pods", namespace],
    check=True,
)
```

Also validate `namespace` and any other external input.

This helps prevent command injection.

---

## 27. AWS Utility Module

Create:

```text
devops_utils/aws.py
```

Example:

```python
import boto3


def create_session(region=None):
    return boto3.Session(region_name=region)


def get_ec2_client(region=None):
    session = create_session(region)
    return session.client("ec2")
```

Use:

```python
from devops_utils.aws import get_ec2_client

ec2 = get_ec2_client("ap-south-1")
```

Credentials should come from the AWS credential chain or approved identity mechanism, never hard-coded secrets.

---

## 28. Why Centralize AWS Logic?

A reusable AWS module can centralize:

```text
session creation
region handling
retry behavior
common API calls
logging
error translation
pagination helpers
```

Then scripts can focus on the business decision rather than SDK setup.

---

## 29. Kubernetes Utility Module

Possible structure:

```text
devops_utils/kubernetes.py
```

Functions might include:

```python
def get_pods(...):
    ...


def get_deployments(...):
    ...


def find_unhealthy_pods(...):
    ...


def rollout_status(...):
    ...
```

Keep Kubernetes communication separate from the logic that decides whether the environment is healthy.

---

## 30. Docker Utility Module

Possible functions:

```text
devops_utils/docker.py
```

```python
def list_images():
    ...


def container_status(container):
    ...


def remove_image(image):
    ...


def cleanup_unused_images():
    ...
```

Keep CLI-specific argument parsing in the script entry point.

---

## 31. Terraform Utility Module

A Terraform wrapper can standardize:

```text
working directory
environment variables
timeouts
return codes
stdout/stderr
logging
```

Example concept:

```python
def terraform_plan(directory):
    ...
```

Do not hide Terraform failures. Return enough information for the caller and CI pipeline to make a decision.

---

## 32. Separate Responsibilities

Good design:

```text
script
  |
  +-- parse arguments
  +-- configure logging
  +-- load configuration
  +-- call utility
  +-- handle result
  +-- exit
```

Utility module:

```text
library
  |
  +-- AWS operation
  +-- Kubernetes operation
  +-- filesystem operation
  +-- shell operation
```

This separation improves testing and reuse.

---

## 33. What Is a Package?

A package organizes related modules into a directory.

```text
devops_utils/
├── __init__.py
├── aws.py
├── kubernetes.py
└── shell.py
```

This groups related automation functionality.

---

## 34. `__init__.py`

A conventional package contains:

```python
# devops_utils/__init__.py
```

It may be empty or contain lightweight package metadata/exports.

Avoid putting expensive production operations in it. Importing a package should not unexpectedly query AWS, modify Kubernetes, or delete resources.

Modern Python also supports namespace packages without `__init__.py`, but explicit `__init__.py` remains common for normal application packages.

---

## 35. Package Import

```python
from devops_utils import shell
```

or:

```python
from devops_utils.shell import run_command
```

---

## 36. Subpackages

Large automation projects may be organized as:

```text
devops_utils/
├── aws/
│   ├── __init__.py
│   ├── ec2.py
│   ├── s3.py
│   └── eks.py
├── kubernetes/
│   ├── __init__.py
│   ├── pods.py
│   └── deployments.py
└── shell/
    ├── __init__.py
    └── commands.py
```

Use this only when the project is large enough to benefit from the extra structure.

---

## 37. Relative Imports

Inside a package:

```python
from .shell import run_command
```

The `.` means the current package.

Avoid deeply nested relative imports when they reduce readability.

---

## 38. Absolute Imports

Prefer clear imports such as:

```python
from devops_utils.shell import run_command
```

Absolute imports are usually easier to understand in larger DevOps projects.

---

## 39. Circular Imports

Avoid:

```text
aws.py imports kubernetes.py
kubernetes.py imports aws.py
```

If both need a shared helper, move it into an appropriate lower-level module:

```text
common/validation.py
```

Then:

```text
AWS ---------> validation
Kubernetes --> validation
```

---

## 40. Module-Level Side Effects

Avoid code such as:

```python
# dangerous design
connect_to_production()
delete_old_resources()
```

at module import time.

Use explicit functions and a controlled `main()` workflow instead.

---

## 41. Constants Module

A module can contain genuine constants:

```python
DEFAULT_TIMEOUT = 30

SUPPORTED_ENVIRONMENTS = {
    "dev",
    "staging",
    "production",
}
```

Do not use a constants module as a dumping ground for mutable global state.

---

## 42. Reusable Health Check Module

Example:

```python
import urllib.request


def check_http(url, timeout=5):
    try:
        with urllib.request.urlopen(url, timeout=timeout) as response:
            return response.status == 200
    except Exception:
        return False
```

For production code, catch specific exceptions and use an appropriate HTTP client when needed.

---

## 43. Health Check Script

```python
from devops_utils.health import check_http


def main():
    urls = [
        "https://example.com/health",
    ]

    for url in urls:
        if check_http(url):
            print(f"OK: {url}")
        else:
            print(f"FAILED: {url}")


if __name__ == "__main__":
    main()
```

This demonstrates the module/script separation.

---

## 44. Reusable File Utility

```python
from pathlib import Path


def read_text(path):
    return Path(path).read_text(encoding="utf-8")
```

Use:

```python
from devops_utils.files import read_text

content = read_text("/tmp/app.log")
```

Detailed file operations are covered in `02-File-Handling.md`.

---

## 45. Reusable Retry Utility

A shared retry module might provide:

```python
import time


def retry(operation, attempts=3, delay=2):
    ...
```

Production retry logic should consider:

```text
maximum attempts
timeout
backoff
jitter
retryable errors
non-retryable errors
overall deadline
```

Never create an infinite retry loop around a production operation.

---

## 46. Dependency Management

Common DevOps Python dependencies include:

```text
boto3
kubernetes
requests
PyYAML
```

Track dependencies explicitly instead of assuming they happen to be installed on an engineer's machine.

Example `requirements.txt`:

```text
boto3
kubernetes
PyYAML
requests
```

Install:

```bash
python -m pip install -r requirements.txt
```

---

## 47. `python -m pip`

Prefer:

```bash
python -m pip install boto3
```

rather than relying on:

```bash
pip install boto3
```

because `python -m pip` makes it explicit which Python interpreter owns the installation.

---

## 48. Dependency Pinning

For reproducible automation, use a controlled dependency strategy, for example:

```text
boto3==...
requests==...
```

or approved compatible version ranges.

The goal is:

```text
reproducible
+ tested
+ maintainable
```

Blindly installing latest versions in production automation is risky.

---

## 49. Virtual Environments

Create:

```bash
python -m venv .venv
```

Activate on Linux/macOS:

```bash
source .venv/bin/activate
```

Install:

```bash
python -m pip install -r requirements.txt
```

Virtual environments are covered deeply in `09-Virtual-Environments.md`.

---

## 50. Why Virtual Environments Matter

Without isolation:

```text
Project A -> dependency version X
Project B -> dependency version Y
```

may conflict.

With isolation:

```text
Project A -> .venv -> dependencies A
Project B -> .venv -> dependencies B
```

---

## 51. Production Repository Structure

A practical automation repository:

```text
devops-automation/
├── scripts/
│   ├── check_disk.py
│   ├── check_k8s.py
│   └── aws_inventory.py
├── devops_utils/
│   ├── __init__.py
│   ├── aws.py
│   ├── kubernetes.py
│   ├── shell.py
│   ├── files.py
│   └── retry.py
├── tests/
│   ├── test_aws.py
│   ├── test_shell.py
│   └── test_kubernetes.py
├── requirements.txt
├── README.md
└── .gitignore
```

---

## 52. Thin Script Principle

Instead of a 500-line script, aim for:

```text
parse arguments
      ↓
load config
      ↓
call reusable library
      ↓
format result
      ↓
exit
```

Complex reusable logic belongs in modules.

---

## 53. Return Data Instead of Printing Inside Utilities

Less reusable:

```python
def get_pods():
    print(pods)
```

Better:

```python
def get_pods():
    return pods
```

Then the caller decides how to present the result.

This allows the same function to be used by:

```text
CLI
CI/CD
API
scheduled job
tests
```

---

## 54. Separate Business Logic From Output

Instead of:

```python
def check_disk():
    print("Disk is full")
```

return structured data:

```python
def check_disk():
    return {
        "status": "critical",
        "usage": 94,
    }
```

Then the CLI layer can format:

```text
CRITICAL: Disk usage 94%
```

---

## 55. Reusable Disk Utility

```python
import shutil


def disk_usage(path="/"):
    total, used, free = shutil.disk_usage(path)
    percent = used / total * 100

    return {
        "total": total,
        "used": used,
        "free": free,
        "percent": percent,
    }
```

Use:

```python
from devops_utils.disk import disk_usage

usage = disk_usage("/")

if usage["percent"] > 90:
    print("CRITICAL")
```

This is the type of reusable function used in practical DevOps scripting.

---

## 56. Reusable Service Utility

```python
import subprocess


def service_status(service):
    result = subprocess.run(
        ["systemctl", "is-active", service],
        capture_output=True,
        text=True,
        check=False,
    )

    return result.returncode == 0
```

Use:

```python
if not service_status("nginx"):
    print("nginx is down")
```

---

## 57. Dependency Injection

Instead of creating an AWS client inside every function:

```python
def get_instances():
    ec2 = boto3.client("ec2")
    ...
```

pass it in:

```python
def get_instances(ec2):
    ...
```

Production provides the real client; tests can provide a fake client.

---

## 58. Dependency Injection Example

```python
def list_instances(ec2_client):
    response = ec2_client.describe_instances()
    return response
```

Production:

```python
ec2 = boto3.client("ec2")
instances = list_instances(ec2)
```

Testing:

```python
fake_ec2 = FakeEC2Client()
instances = list_instances(fake_ec2)
```

No real AWS call is required for the unit test.

---

## 59. Pure Functions

Prefer pure transformation functions where possible:

```python
def find_failed_services(services):
    return [
        service
        for service in services
        if service["status"] != "healthy"
    ]
```

This function does not call AWS, Kubernetes or write files. It is easy to test.

---

## 60. Keep External Effects at the Boundary

A strong architecture is:

```text
AWS / Kubernetes / API
        ↓
structured data
        ↓
pure transformation
        ↓
decision
        ↓
external action
```

This makes production automation easier to reason about and test.

---

## 61. Complete Example — Disk Check

Module:

```python
# devops_utils/disk.py

import shutil


def get_disk_usage(path="/"):
    total, used, free = shutil.disk_usage(path)

    return {
        "path": path,
        "total": total,
        "used": used,
        "free": free,
        "percent": used / total * 100,
    }
```

Script:

```python
# scripts/check_disk.py

from devops_utils.disk import get_disk_usage


def main():
    usage = get_disk_usage("/")

    print(
        f"{usage['path']}: "
        f"{usage['percent']:.1f}% used"
    )


if __name__ == "__main__":
    main()
```

---

## 62. Complete Example — Kubernetes Health

A reusable health function:

```python
def unhealthy_pods(pods):
    return [
        pod
        for pod in pods
        if pod.get("status") != "Running"
    ]
```

Script:

```python
from devops_utils.kubernetes import unhealthy_pods


def main():
    pods = [
        {"name": "user-1", "status": "Running"},
        {"name": "payment-1", "status": "CrashLoopBackOff"},
    ]

    failed = unhealthy_pods(pods)

    for pod in failed:
        print(f"UNHEALTHY: {pod['name']} {pod['status']}")


if __name__ == "__main__":
    main()
```

Later the input can come from a real Kubernetes client.

---

## 63. Complete Example — AWS Inventory Index

```python
# devops_utils/aws_inventory.py

def index_instances(instances):
    return {
        instance["InstanceId"]: instance
        for instance in instances
    }
```

Then:

```python
inventory = index_instances(instances)

instance = inventory.get("i-001")
```

This is much faster and cleaner for repeated resource lookup than scanning the list every time.

---

## 64. Complete Example — Configuration Validation Module

```python
# devops_utils/validation.py

def validate_required_keys(config, required):
    missing = set(required) - config.keys()

    if missing:
        raise ValueError(
            f"Missing required keys: {sorted(missing)}"
        )
```

Use:

```python
config = {
    "environment": "production",
    "image": "payment:v42",
}

validate_required_keys(
    config,
    {"environment", "image", "replicas"},
)
```

---

## 65. Package API Design

A package should expose a clear public interface.

Example:

```python
from devops_utils.aws import list_running_instances
```

Internal helpers can use a leading underscore:

```python
def _normalize_instance(instance):
    ...
```

This is a convention, not a security boundary.

---

## 66. Module Documentation

A module can start with a docstring:

```python
"""Utilities for executing Linux commands safely."""
```

Shared modules should document:

```text
purpose
parameters
return values
exceptions
side effects
permissions
timeouts
```

---

## 67. Avoid Huge Utility Modules

Avoid one file such as:

```text
utils.py
```

containing AWS, Kubernetes, Docker, logging and filesystem logic.

Prefer focused modules:

```text
aws.py
kubernetes.py
docker.py
filesystem.py
shell.py
validation.py
retry.py
```

Clear ownership makes maintenance easier.

---

## 68. Dependency Direction

A healthy project often follows:

```text
CLI scripts
     ↓
service/domain modules
     ↓
low-level utilities
```

Avoid random dependencies between every module. This reduces circular imports.

---

## 69. Package Versioning

Internal automation libraries may be versioned:

```text
1.0.0
1.1.0
2.0.0
```

Versioning allows CI/CD jobs and repositories to depend on known library behavior.

---

## 70. Internal Package Repositories

Enterprise teams may publish internal Python packages to:

```text
JFrog Artifactory
AWS CodeArtifact
private PyPI
```

Then automation projects install approved versions.

This connects Python development with your DevSecOps supply-chain practices.

---

## 71. Dependency Supply Chain

Production Python automation should consider:

```text
approved packages
version control
vulnerability scanning
dependency updates
private repositories
license policy
SBOM requirements where applicable
```

Do not treat third-party dependencies as automatically trusted.

---

## 72. Dynamic Imports

Python supports:

```python
import importlib

module = importlib.import_module("devops_utils.aws")
```

This can support plugin architectures, but it adds complexity and security considerations. Prefer explicit imports unless dynamic loading is actually required.

---

## 73. Import Performance and Caching

Python keeps imported modules in:

```python
sys.modules
```

Within a process, repeated imports normally reuse the already loaded module.

The important DevOps lesson is to keep module initialization lightweight.

---

## 74. `__pycache__`

Python may create:

```text
__pycache__/
```

and `.pyc` bytecode files.

Do not normally commit them to Git. Add to `.gitignore`:

```text
__pycache__/
*.py[cod]
```

---

## 75. Troubleshooting — Local Works, CI Fails

If a script works locally but Jenkins/GitLab/GitHub Actions fails:

```text
Compare:
Python version
pip version
virtual environment
installed dependencies
working directory
package structure
PYTHONPATH
OS image
credentials
```

Useful commands:

```bash
which python
python --version
python -m pip --version
python -m pip list
```

---

## 76. Troubleshooting — Wrong Python Interpreter

You run:

```bash
pip install boto3
```

but:

```bash
python script.py
```

still reports:

```text
ModuleNotFoundError: No module named 'boto3'
```

Possible cause:

```text
pip → Python A
python → Python B
```

Use:

```bash
python -m pip install boto3
python -m pip show boto3
```

---

## 77. Troubleshooting — Import Path Problems

Do not casually solve package problems with:

```python
sys.path.append("../")
```

This can hide packaging issues and make execution dependent on the current working directory.

Prefer a proper package structure and controlled execution environment.

---

## 78. Troubleshooting — Circular Imports

Symptoms may include:

```text
ImportError
partially initialized module
cannot import name
```

Check whether:

```text
A imports B
B imports A
```

Move shared logic into a lower-level module.

---

## 79. Troubleshooting — Production Script Hangs

Possible causes:

```text
AWS API call
Kubernetes API call
kubectl process
network request
unbounded retry
```

Production response:

```text
add timeout
limit retries
capture stderr
log the operation
return structured failure
use an overall deadline
```

---

## 80. Troubleshooting — Duplicate Automation Logic

If five repositories contain the same AWS or Kubernetes logic:

```text
duplicate code
      ↓
extract reusable module
      ↓
test it
      ↓
package it
      ↓
publish internally
      ↓
consume approved version
```

This is a practical enterprise DevOps pattern.

---

## 81. Daily DevOps Script Architecture

```text
check_k8s.py
     ↓
argparse
     ↓
logging
     ↓
kubernetes.py
     ↓
Kubernetes API
     ↓
structured result
     ↓
health evaluation
     ↓
exit code
```

The same pattern can be used for AWS, Docker, Terraform and Linux automation.

---

## 82. Daily DevOps Example — Disk Check

```text
scripts/check_disk.py
        ↓
devops_utils/filesystem.py
        ↓
Linux filesystem
```

The same filesystem module can support:

```text
cleanup_logs.py
disk_report.py
server_health.py
```

---

## 83. Daily DevOps Example — AWS Inventory

```text
scripts/aws_inventory.py
        ↓
devops_utils/aws.py
        ↓
boto3
        ↓
AWS
```

The same AWS module can support:

```text
EC2 reports
ECR checks
S3 inventory
EKS checks
```

---

## 84. Daily DevOps Example — Kubernetes Health

```text
scripts/k8s_health.py
        ↓
devops_utils/kubernetes.py
        ↓
Kubernetes API
        ↓
EKS
```

The script can later be called from Jenkins, GitHub Actions, GitLab CI/CD or a scheduled operational job.

---

## 85. Daily DevOps Example — Terraform Wrapper

```text
scripts/tf_validate.py
        ↓
devops_utils/terraform.py
        ↓
subprocess
        ↓
terraform
```

The wrapper can standardize working directory, timeout, environment, output and exit-code handling.

---

## 86. Daily DevOps Example — Security Scan

A security module can wrap:

```text
Trivy
SonarQube APIs
Veracode APIs
```

Return structured results such as:

```python
{
    "status": "failed",
    "critical": 2,
    "high": 5,
}
```

Then the CI/CD pipeline decides whether deployment should be blocked.

---

## 87. Daily DevOps Example — Deployment Report

Multiple modules can contribute:

```text
Git commit
    +
Docker image
    +
Terraform result
    +
Kubernetes rollout
    +
security scan
    ↓
deployment_report.py
```

Result:

```python
{
    "commit": "abc123",
    "image": "payment:v42",
    "security": "passed",
    "terraform": "passed",
    "rollout": "passed",
}
```

---

## 88. Production Architecture

```text
                 Git
                  ↓
          Python Automation
                  ↓
       +----------+----------+
       |          |          |
       ↓          ↓          ↓
      AWS     Kubernetes    Linux
       |          |          |
       +----------+----------+
                  ↓
         Structured Results
                  ↓
         Logging / Reporting
                  ↓
                CI/CD
```

Modules form the reusable layer between scripts and external systems.

---

## 89. Testing Modular DevOps Code

If:

```python
def is_healthy(status):
    return status == "Running"
```

test:

```text
Running → True
Pending → False
Failed → False
```

Pure functions are easy to unit test.

For AWS/Kubernetes calls, use mocks, fakes or dependency injection instead of requiring a real production account or cluster for every unit test.

---

## 90. Production Code Quality Checklist

```text
[ ] Clear module responsibility
[ ] Explicit imports
[ ] No wildcard imports
[ ] No production side effects on import
[ ] Reusable functions
[ ] External dependencies controlled
[ ] Timeouts configured
[ ] Retries bounded
[ ] Errors handled
[ ] Logging used appropriately
[ ] No hard-coded secrets
[ ] Input validated
[ ] Useful return values
[ ] Meaningful exit codes
[ ] Unit tests
[ ] CI validation
[ ] Documentation
[ ] Least-privilege permissions
```

---

# 91. Interview Questions

## What is a module?

> A module is a Python file containing reusable code such as functions, classes and constants. In DevOps I use modules to separate AWS, Kubernetes, Docker, filesystem, shell and validation functionality so multiple automation scripts can reuse the same logic.

## What is a package?

> A package is a structured collection of related Python modules. A DevOps package could contain AWS, Kubernetes, Docker, shell and validation modules.

## Why use `if __name__ == "__main__"`?

> It allows a Python file to have a controlled entry point when executed directly while preventing the main workflow from automatically running when the file is imported.

## Why avoid wildcard imports?

> They make dependencies unclear and can create name collisions. Explicit imports make code easier to review and maintain.

## How do you structure a large DevOps Python project?

> I keep CLI scripts thin and move reusable functionality into domain modules such as AWS, Kubernetes, Docker, Terraform, shell, filesystem and validation. I keep tests separate and manage dependencies through a controlled environment.

## How do you handle external dependencies in tests?

> I use dependency injection, mocks or fakes. For example, an AWS function can accept an EC2 client rather than constructing it internally, allowing tests to provide a fake client without making real AWS calls.

## Why not put everything in one script?

> A monolithic script becomes difficult to test, reuse, troubleshoot and maintain. Modules allow focused responsibilities and centralized fixes.

## How do you troubleshoot `ModuleNotFoundError`?

> I check the Python interpreter, `python -m pip --version`, installed dependencies, virtual environment, project structure and `sys.path`. In CI/CD I also verify the build agent's Python environment.

## Why use `python -m pip`?

> It explicitly invokes pip using the selected Python interpreter, reducing confusion when multiple Python installations exist.

## Why use dependency injection?

> It separates business logic from external systems and makes unit testing easier because a real AWS/Kubernetes client can be replaced with a fake implementation.

---

# 92. Scenario — Jenkins Script Works Locally but Fails

Likely difference:

```text
local virtual environment
        ≠
Jenkins Python environment
```

Check:

```bash
python --version
python -m pip --version
python -m pip list
python -c "import boto3; print(boto3.__version__)"
```

Then make dependency installation explicit in the pipeline.

---

# 93. Scenario — Kubernetes Automation Hangs

Investigate:

```text
API timeout
kubectl waiting
network connectivity
authentication
unbounded retry
```

Use:

```text
timeouts
bounded retries
stderr capture
structured logging
overall deadlines
```

---

# 94. Scenario — Five Repositories Have the Same AWS Logic

Do not keep fixing five copies.

Use:

```text
shared module
    ↓
tests
    ↓
internal package
    ↓
JFrog Artifactory / private package repository
    ↓
versioned dependency
```

This gives centralized fixes and controlled upgrades.

---

# 95. Scenario — AWS and Kubernetes in One Automation

Use domain modules:

```text
main.py
├── aws.py
├── kubernetes.py
├── validation.py
└── reporting.py
```

Then:

```text
main
 ├── AWS
 ├── Kubernetes
 ├── validation
 └── reporting
```

Avoid putting every API operation into `main.py`.

---

# 96. Scenario — Utility Module Became 1000 Lines

Split by responsibility:

```text
utils.py
   ↓
aws.py
filesystem.py
shell.py
network.py
validation.py
retry.py
```

A module should have a focused responsibility.

---

# 97. Practical Exercise — Shell Module

Create:

```text
devops_utils/shell.py
```

Implement:

```python
run_command(command, timeout=30)
```

Return:

```text
returncode
stdout
stderr
```

Handle timeout and command failure.

---

# 98. Practical Exercise — Disk Module

Create:

```text
devops_utils/disk.py
```

Implement:

```python
get_disk_usage(path="/")
```

Return:

```python
{
    "path": path,
    "total": ...,
    "used": ...,
    "free": ...,
    "percent": ...,
}
```

---

# 99. Practical Exercise — Kubernetes Health Module

Create:

```text
devops_utils/kubernetes.py
```

Implement:

```python
find_unhealthy_pods(pods)
```

Input:

```python
[
    {"name": "user", "status": "Running"},
    {"name": "payment", "status": "CrashLoopBackOff"},
]
```

Return only unhealthy pods.

---

# 100. Practical Exercise — AWS Inventory Module

Create:

```text
devops_utils/aws_inventory.py
```

Implement:

```python
index_instances(instances)
```

Use `InstanceId` as the dictionary key.

---

# 101. Practical Exercise — Validation Module

Create:

```text
devops_utils/validation.py
```

Implement:

```python
validate_required_keys(config, required)
```

Raise a clear exception when required keys are missing.

---

# 102. Practical Exercise — Retry Module

Create:

```text
devops_utils/retry.py
```

Implement a retry function with:

```text
maximum attempts
delay
```

Then improve it with:

```text
exponential backoff
retryable exceptions
timeout
```

---

# 103. Practical Exercise — Real CLI

Create:

```text
scripts/check_disk.py
```

Requirements:

```text
--path
--threshold
```

Flow:

```text
arguments
   ↓
disk utility
   ↓
compare threshold
   ↓
print result
   ↓
exit code
```

---

# 104. Practical Exercise — Kubernetes CLI

Create:

```text
scripts/k8s_health.py
```

Argument:

```text
--namespace
```

Flow:

```text
CLI
 ↓
Kubernetes module
 ↓
pod status
 ↓
find unhealthy pods
 ↓
report
 ↓
exit 0 / non-zero
```

---

# 105. Practical Exercise — AWS Inventory CLI

Create:

```text
scripts/aws_inventory.py
```

Arguments:

```text
--region
--environment
```

Output:

```text
Instance ID
Name
State
Private IP
```

Use the AWS SDK rather than parsing formatted CLI output when practical.

---

# 106. Practical Exercise — CI/CD Integration

Integrate `check_disk.py` into a CI job:

```text
checkout
   ↓
create Python environment
   ↓
install dependencies
   ↓
run tests
   ↓
run script
   ↓
use exit code
```

The CI job should fail when the script returns a failure status.

---

# 107. Daily DevOps Pattern to Remember

```text
CLI script
    ↓
argparse
    ↓
logging
    ↓
configuration
    ↓
reusable module
    ↓
external system
    ↓
structured result
    ↓
validation
    ↓
exit code
```

This pattern will appear repeatedly throughout the Python-for-DevOps section.

---

# 108. Final Mental Model

Do not think of Python modules as only a language feature.

Think of them as the foundation for a reusable DevOps automation platform:

```text
              Python Automation
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
      AWS       Kubernetes       Linux
        |             |             |
        +-------------+-------------+
                      |
                Reusable Modules
                      |
        +------+------+------+------+
        |      |      |      |      |
       CLI   Retry  Config  Logs  Validation
        |      |      |      |      |
        +------+------+------+------+
                      |
                      v
                    CI/CD
```

The objective is not to write one-off scripts.

The objective is to build **reliable, reusable, testable automation** that can be executed manually, from Jenkins/GitLab/GitHub Actions, through scheduled jobs, or as part of production operational workflows.

---