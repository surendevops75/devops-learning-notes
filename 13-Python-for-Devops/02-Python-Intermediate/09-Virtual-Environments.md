# Virtual-Environments

## DevOps Focus

Python virtual environments isolate project dependencies from the system Python installation.

This matters in DevOps because automation often depends on different versions of:

```text
boto3
kubernetes
PyYAML
requests
Jinja2
pytest
Terraform-related tooling
AWS utilities
security scanners
internal packages
```

Without isolation, one project's dependency upgrade can break another project.

> Core principle: **Keep Python dependencies isolated, explicitly versioned, reproducible, and safe to deploy.**

---

## 1. What Is a Virtual Environment?

A virtual environment is an isolated Python environment containing:

```text
Python interpreter access
pip
installed packages
project-specific dependencies
```

Conceptually:

```text
System Python
     |
     +--> Project A
     |      |
     |      +--> boto3 1.x
     |
     +--> Project B
            |
            +--> boto3 2.x
```

Each project can manage its dependencies independently.

---

## 2. Why DevOps Engineers Need Virtual Environments

Imagine two automation projects:

```text
Project A
boto3 == version A

Project B
boto3 == version B
```

Installing both globally can create:

```text
dependency conflict
```

Virtual environments provide:

```text
Project A -> environment A
Project B -> environment B
```

---

## 3. System Python

Linux systems often have a Python installation used by:

```text
OS tools
system utilities
package managers
```

Do not casually install project dependencies into system Python.

Bad:

```bash
sudo pip install boto3
```

This can modify packages used by the operating system or other applications.

---

## 4. Create a Virtual Environment

Command:

```bash
python3 -m venv .venv
```

This creates:

```text
.venv/
```

inside the project.

Typical project:

```text
devops-tool/
├── .venv/
├── deploy.py
├── requirements.txt
└── README.md
```

---

## 5. Why `.venv`?

`.venv` is a common convention.

Other names are possible:

```text
venv
env
.python-env
```

Using `.venv` makes the environment clearly associated with the project and keeps it visually unobtrusive.

---

## 6. Activate on Linux

```bash
source .venv/bin/activate
```

The shell prompt may become:

```text
(.venv) user@host:~/project$
```

Now:

```bash
python
```

and:

```bash
pip
```

refer to the virtual environment.

---

## 7. Verify Python

Run:

```bash
which python
```

You should see something similar to:

```text
/path/to/project/.venv/bin/python
```

Also:

```bash
python --version
```

---

## 8. Verify pip

```bash
which pip
```

Expected:

```text
/path/to/project/.venv/bin/pip
```

This confirms the active shell is using the project's environment.

---

## 9. Windows Activation

Command Prompt:

```cmd
.venv\Scripts\activate
```

PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

The exact command depends on the shell and operating system.

---

## 10. Deactivate

When finished:

```bash
deactivate
```

The shell returns to the system environment.

---

## 11. Install a Package

After activation:

```bash
pip install boto3
```

Then:

```bash
python -c "import boto3; print(boto3.__version__)"
```

---

## 12. Install Multiple Packages

```bash
pip install \
    boto3 \
    kubernetes \
    PyYAML
```

This is useful for local development.

For reproducible deployments, dependencies should be captured in a dependency file.

---

## 13. `requirements.txt`

Create:

```text
requirements.txt
```

Example:

```text
boto3==1.39.0
kubernetes==33.1.0
PyYAML==6.0.2
```

Then install:

```bash
pip install -r requirements.txt
```

Pinning exact versions can improve reproducibility.

---

## 14. Generate Requirements

A common command:

```bash
pip freeze > requirements.txt
```

This records installed packages.

Example:

```text
boto3==...
botocore==...
PyYAML==...
```

But `pip freeze` captures the entire installed environment, including packages that may not be direct project dependencies.

For larger projects, prefer intentionally maintained dependency files.

---

## 15. Direct vs Transitive Dependencies

Suppose your application requires:

```text
boto3
```

and boto3 requires:

```text
botocore
s3transfer
jmespath
```

Then:

```text
boto3
 |
 +--> botocore
 +--> s3transfer
 +--> jmespath
```

These are transitive dependencies.

Understand the distinction when managing requirements.

---

## 16. Dependency Pinning

Unpinned:

```text
boto3
```

Loosely constrained:

```text
boto3>=1.39
```

Exact:

```text
boto3==1.39.0
```

The right strategy depends on the project's dependency-management approach.

Production automation generally needs controlled upgrades rather than accidental latest-version changes.

---

## 17. Version Ranges

Examples:

```text
boto3>=1.39,<2
```

or:

```text
requests>=2.31,<3
```

This allows compatible updates while preventing major-version changes.

Be aware that transitive dependency resolution can still change.

---

## 18. Why Uncontrolled `pip install` Is Risky

This:

```bash
pip install boto3
```

may install a newer version tomorrow than it did today.

That can cause:

```text
unexpected behavior
API changes
security issues
test failures
production incidents
```

Use controlled dependency management.

---

## 19. Reproducibility

A reproducible Python environment means:

```text
same source
+
controlled dependencies
+
compatible Python version
=
predictable environment
```

This is important for:

```text
developer
CI
staging
production
```

---

## 20. Python Version

Dependencies are not the only compatibility concern.

You should also define the supported Python version:

```text
Python 3.x
```

For example:

```text
Python >=3.11,<3.13
```

depending on project requirements.

---

## 21. Check Python Version

```bash
python --version
```

Example:

```text
Python 3.12.x
```

In CI, explicitly control the Python version rather than relying on whatever happens to be installed on the runner.

---

## 22. Python Version + Dependencies

Compatibility can be represented as:

```text
Python 3.12
   |
   +--> boto3
   +--> kubernetes
   +--> PyYAML
   +--> requests
```

Test the complete combination.

A package working on Python 3.11 does not automatically guarantee identical behavior on every Python release.

---

## 23. `pip` Version

Check:

```bash
python -m pip --version
```

Prefer:

```bash
python -m pip
```

over relying on a potentially unrelated `pip` executable.

---

## 24. Why `python -m pip`?

This:

```bash
python -m pip install boto3
```

ensures pip is executed by the Python interpreter you selected.

This reduces confusion when multiple Python installations exist.

---

## 25. Multiple Python Versions

A system may contain:

```text
python3.10
python3.11
python3.12
```

You can create a venv using a specific interpreter:

```bash
python3.12 -m venv .venv
```

Then:

```bash
source .venv/bin/activate
```

Now the environment is based on Python 3.12.

---

## 26. Check Virtual Environment Interpreter

```bash
python -c \
'import sys; print(sys.executable)'
```

This is useful in troubleshooting.

Expected:

```text
/path/project/.venv/bin/python
```

---

## 27. Check Installed Packages

```bash
python -m pip list
```

or:

```bash
python -m pip freeze
```

`pip list` is useful for human inspection.

`pip freeze` is useful for a requirements-style snapshot.

---

## 28. Show Package Information

```bash
python -m pip show boto3
```

This shows information such as:

```text
Name
Version
Location
Requires
Required-by
```

Useful during dependency troubleshooting.

---

## 29. Upgrade pip

Inside a virtual environment:

```bash
python -m pip install --upgrade pip
```

Do this deliberately in controlled environments.

In CI, consider pinning tool versions rather than blindly upgrading to the newest version.

---

## 30. Install Development Dependencies

You may have:

```text
runtime dependencies
development dependencies
```

Runtime:

```text
boto3
kubernetes
PyYAML
```

Development:

```text
pytest
ruff
mypy
coverage
```

Keep the distinction clear.

---

## 31. Requirements Files

A simple project may use:

```text
requirements.txt
requirements-dev.txt
```

Example:

```text
requirements.txt
    boto3
    kubernetes
    PyYAML
```

and:

```text
requirements-dev.txt
    -r requirements.txt
    pytest
    ruff
```

---

## 32. Requirements File Structure

Example:

```text
# Runtime
boto3==1.39.0
kubernetes==33.1.0
PyYAML==6.0.2

# Development
pytest==8.x
ruff==0.x
```

Comments can document why a dependency exists.

---

## 33. Separate Production and Development Dependencies

Production container:

```text
runtime dependencies only
```

Development environment:

```text
runtime
+
testing
+
linting
+
formatting
```

This reduces production image size and attack surface.

---

## 34. DevOps Example

A deployment automation tool might need:

```text
boto3
kubernetes
PyYAML
requests
```

while development also needs:

```text
pytest
ruff
mypy
```

Do not install every development tool into the production runtime unnecessarily.

---

## 35. Virtual Environment and Git

Do not commit:

```text
.venv/
```

to Git.

The environment is generated locally.

Commit:

```text
requirements.txt
pyproject.toml
lock file
```

depending on the project's dependency strategy.

---

## 36. `.gitignore`

Add:

```text
.venv/
```

Example:

```text
# Python
.venv/
__pycache__/
*.pyc
```

This keeps generated environment files out of source control.

---

## 37. Why Not Commit `.venv`?

A virtual environment can contain:

```text
platform-specific binaries
absolute paths
large files
machine-specific packages
```

It is not a portable source artifact.

Recreate it from dependency definitions.

---

## 38. Recreate Environment

Clone repository:

```bash
git clone <repo>
cd <repo>
```

Create environment:

```bash
python3 -m venv .venv
```

Activate:

```bash
source .venv/bin/activate
```

Install:

```bash
python -m pip install -r requirements.txt
```

Now the developer has a fresh environment.

---

## 39. CI Environment

Typical pipeline:

```text
checkout
   |
   v
install Python
   |
   v
create venv
   |
   v
install dependencies
   |
   v
lint
   |
   v
test
   |
   v
package
```

This avoids depending on pre-installed global Python packages.

---

## 40. CI Example

```bash
python -m venv .venv

source .venv/bin/activate

python -m pip install \
    --upgrade pip

python -m pip install \
    -r requirements.txt

pytest
```

The CI runner gets a clean project environment.

---

## 41. Why Clean CI Environments Matter

A dirty runner may already contain:

```text
old boto3
old Kubernetes client
different PyYAML
different pytest
```

Then tests pass because of accidental dependencies.

A clean environment exposes real dependency requirements.

---

## 42. CI Dependency Cache

CI systems often cache:

```text
pip downloads
virtual environments
package wheels
```

Caching can speed builds.

But cache keys should account for:

```text
Python version
dependency file hash
OS
architecture
```

so stale packages do not cause unexpected behavior.

---

## 43. Dependency Cache Key

Conceptually:

```text
Python version
+
OS
+
requirements hash
=
cache key
```

When requirements change, the cache should be invalidated appropriately.

---

## 44. CI Reproducibility

A strong pipeline controls:

```text
Python version
pip version
dependency versions
OS/base image
environment variables
```

Then:

```text
developer
CI
production
```

are less likely to diverge.

---

## 45. Docker and Virtual Environments

A Docker container already provides process/filesystem isolation.

Therefore, many production Python containers do not need a traditional venv inside the container.

Example:

```text
Docker container
    |
    +--> Python
    +--> installed dependencies
    +--> application
```

This is often simpler.

---

## 46. Docker Without venv

Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

The container itself isolates the application environment.

---

## 47. Docker With venv

You can still use a venv inside Docker:

```dockerfile
RUN python -m venv /opt/venv

ENV PATH="/opt/venv/bin:$PATH"

RUN pip install \
    --no-cache-dir \
    -r requirements.txt
```

This can be useful for specific packaging or multi-stage build designs, but it is not automatically required.

---

## 48. When Venv Is Useful

Use virtual environments heavily for:

```text
local development
Python automation repositories
CI jobs
VM-based deployments
developer tools
testing
```

---

## 49. When Venv May Be Unnecessary

In a container where:

```text
one application
one Python runtime
one dependency environment
```

already exists, a venv may add unnecessary complexity.

Choose based on the deployment model.

---

## 50. Kubernetes + Python

For Python automation running as a Kubernetes workload:

```text
container
   |
   +--> Python
   +--> dependencies
   +--> application
```

Dependency isolation is primarily provided by the container image.

The image should still use controlled dependency versions.

---

## 51. Multi-Stage Docker Build

For Python:

```text
builder
   |
   +--> create environment
   +--> install dependencies
   |
   v
runtime
   |
   +--> copy required environment/packages
   +--> application
```

This can reduce the final image footprint.

Test carefully because compiled dependencies may require runtime libraries.

---

## 52. Requirements vs Docker Image

A Docker image should not depend on:

```text
developer's local .venv
```

Instead:

```text
requirements
   |
   v
Docker build
   |
   v
reproducible image
```

---

## 53. `.venv` and Docker Build Context

If `.venv` exists locally and is not ignored, Docker may accidentally copy it:

```dockerfile
COPY . .
```

This is bad.

Use:

```text
.dockerignore
```

with:

```text
.venv/
__pycache__/
.git/
```

---

## 54. `.dockerignore`

Example:

```text
.venv/
__pycache__/
*.pyc
.git/
.pytest_cache/
```

This reduces:

```text
build context
image size
build time
accidental file inclusion
```

---

## 55. Dependency Security

Python dependencies are part of the application attack surface.

A vulnerable dependency can introduce:

```text
remote code execution
data exposure
privilege escalation
supply-chain risk
```

Do not treat `requirements.txt` as just a convenience file.

---

## 56. Dependency Scanning

Use appropriate security tools to identify:

```text
known CVEs
outdated dependencies
malicious packages
license issues
```

Your organization may use tools such as:

```text
Trivy
SCA platforms
dependency scanners
```

---

## 57. Trivy + Python Dependencies

If the project contains:

```text
requirements.txt
```

Trivy can scan the project/image for vulnerable dependencies depending on configuration and version.

A CI pipeline can perform:

```text
dependency scan
   |
   v
policy check
   |
   +--> pass
   |
   +--> fail
```

---

## 58. Dependency Update Workflow

A controlled process:

```text
current dependency
       |
       v
check available update
       |
       v
security/change review
       |
       v
update requirements
       |
       v
tests
       |
       v
security scan
       |
       v
build image
       |
       v
deploy
```

Do not blindly upgrade production dependencies.

---

## 59. Security Patch vs Feature Upgrade

Security patch:

```text
urgent
```

Feature upgrade:

```text
planned
```

Both should pass:

```text
tests
security checks
compatibility validation
```

---

## 60. Dependency Drift

Dependency drift occurs when environments differ:

```text
developer -> boto3 A
CI        -> boto3 B
production -> boto3 C
```

This can create:

```text
works on my machine
```

problems.

Controlled environments reduce drift.

---

## 61. Detecting Drift

Compare:

```bash
python -m pip freeze
```

across environments.

For production, prefer comparing against:

```text
approved lock/dependency artifact
```

rather than manually comparing large output.

---

## 62. Lock Files

Modern Python projects may use dependency managers that generate lock files.

A lock file records a resolved dependency graph more precisely than a simple top-level requirements file.

The exact tool depends on the project.

---

## 63. `requirements.txt` vs Lock File

`requirements.txt` can describe:

```text
direct requirements
```

A lock file can capture:

```text
resolved versions
transitive dependencies
hashes
```

For reproducible production builds, a lock strategy can be valuable.

---

## 64. Hash Pinning

Some dependency workflows use package hashes:

```text
package==version
    --hash=...
```

This can strengthen supply-chain integrity.

Use the mechanism supported by your package-management workflow.

---

## 65. Dependency Supply Chain

Think:

```text
PyPI / internal repository
          |
          v
package
          |
          v
dependency resolver
          |
          v
build
          |
          v
container
          |
          v
production
```

Every step should be controlled.

---

## 66. JFrog Artifactory

Organizations may use an internal artifact repository such as:

```text
JFrog Artifactory
```

for Python packages.

A controlled architecture can be:

```text
Developer / CI
      |
      v
Artifactory
      |
      v
approved Python packages
      |
      v
build
```

This can improve governance and supply-chain control.

---

## 67. Private Python Repository

A private repository can host:

```text
internal Python packages
approved third-party packages
cached dependencies
```

The authentication mechanism must be handled securely.

Do not put repository credentials directly in source code.

---

## 68. `pip` Index Configuration

Pip can be configured to use an internal package index.

For example:

```bash
python -m pip install \
    --index-url <approved-index> \
    -r requirements.txt
```

In production CI, use secure configuration rather than embedding credentials in the command line.

---

## 69. Dependency Installation in CI

Recommended sequence:

```text
1. Select Python version.
2. Create clean environment.
3. Upgrade/install approved pip tooling.
4. Install locked/controlled dependencies.
5. Run security scan.
6. Run tests.
7. Build artifact/image.
```

---

## 70. Offline Builds

Some production environments have no internet access.

Architecture:

```text
internet-connected build
        |
        v
approved artifact repository
        |
        v
isolated production build
```

Python dependencies should be available from an approved internal source.

---

## 71. Wheel Files

Python packages can be distributed as wheels:

```text
package-version-py3-none-any.whl
```

or platform-specific wheels.

Wheels can make installation faster and support controlled offline builds.

---

## 72. Source Distributions

Some packages may require building from source.

This can introduce requirements such as:

```text
compiler
system libraries
headers
```

For production Docker images, prefer compatible prebuilt wheels when appropriate to reduce complexity.

---

## 73. Native Dependencies

Example:

```text
Python package
    |
    v
C extension
    |
    v
system library
```

A package can work locally but fail in a minimal Docker image because required system libraries are missing.

---

## 74. Troubleshooting Installation Failure

When:

```bash
pip install -r requirements.txt
```

fails, check:

```text
Python version
pip version
package version
OS
CPU architecture
native dependencies
network/index access
dependency conflicts
```

---

## 75. Dependency Conflict

Example:

```text
Package A requires:
requests < 3

Package B requires:
requests >= 3
```

The resolver may reject the combination.

Do not simply force installation.

Review compatible package versions.

---

## 76. Dependency Resolver

Modern pip attempts to resolve dependency requirements.

A failure may look like:

```text
ResolutionImpossible
```

This means the requested dependency constraints cannot all be satisfied.

---

## 77. Fixing Dependency Conflicts

Process:

```text
identify conflicting packages
        |
        v
check supported versions
        |
        v
choose compatible versions
        |
        v
update dependency definitions
        |
        v
run tests
```

Avoid random version changes.

---

## 78. Dependency Constraints

A constraints file can control versions without directly declaring all dependencies.

Example concept:

```text
constraints.txt
```

Then:

```bash
pip install \
    -r requirements.txt \
    -c constraints.txt
```

This can be useful in larger organizations.

---

## 79. Why Constraints Are Useful

Suppose multiple projects use:

```text
requests
```

A central constraints policy can standardize a known-good version.

This helps manage:

```text
security
compatibility
reproducibility
```

---

## 80. DevOps Automation Repository

Recommended:

```text
automation/
├── .venv/
├── .gitignore
├── .dockerignore
├── requirements.txt
├── requirements-dev.txt
├── deploy.py
├── health_check.py
├── k8s_debug.py
└── tests/
```

`.venv` remains local/generated.

---

## 81. Project Bootstrap Script

A simple bootstrap:

```bash
#!/usr/bin/env bash

set -euo pipefail

python3 -m venv .venv

source .venv/bin/activate

python -m pip install \
    --upgrade pip

python -m pip install \
    -r requirements.txt
```

This gives developers a repeatable setup.

---

## 82. Why `set -euo pipefail`?

For shell automation:

```text
-e -> exit on command failure
-u -> fail on unset variables
pipefail -> pipeline failure propagates
```

This is a useful companion to reliable Python automation.

---

## 83. Makefile Workflow

A project may provide:

```makefile
venv:
	python3 -m venv .venv

install:
	. .venv/bin/activate && \
	python -m pip install -r requirements.txt

test:
	. .venv/bin/activate && \
	pytest
```

Then:

```bash
make install
make test
```

This standardizes developer workflows.

---

## 84. Make + Virtual Environment

The Makefile can hide shell details:

```text
developer
   |
   v
make install
   |
   v
venv
   |
   v
dependencies
```

This is especially useful in team repositories.

---

## 85. Python Script to Create Venv

Python can also create environments:

```python
import venv

venv.create(
    ".venv",
    with_pip=True,
)
```

This can be useful when bootstrap logic itself is Python-based.

---

## 86. Programmatic Environment Creation

Example:

```python
from pathlib import Path
import venv

env_dir = Path(".venv")

if not env_dir.exists():
    venv.create(
        env_dir,
        with_pip=True,
    )
```

The script can then instruct the user to activate the environment.

A running Python process cannot simply change its parent's shell environment.

---

## 87. Important Activation Concept

Activation mainly modifies shell environment variables such as:

```text
PATH
VIRTUAL_ENV
```

It does not magically make Python code isolated.

You can directly invoke:

```bash
.venv/bin/python
```

without activating the environment.

---

## 88. CI Without Activation

CI can run:

```bash
.venv/bin/python script.py
```

and:

```bash
.venv/bin/pip install -r requirements.txt
```

This avoids shell activation concerns.

Activation is primarily a convenience for interactive shells.

---

## 89. Shebang

A script may use:

```python
#!/usr/bin/env python3
```

But if you need to guarantee the project environment, invoking:

```bash
.venv/bin/python script.py
```

is explicit.

---

## 90. `VIRTUAL_ENV`

When activated:

```bash
echo "$VIRTUAL_ENV"
```

may show:

```text
/path/project/.venv
```

Useful for troubleshooting.

---

## 91. PATH Troubleshooting

If:

```bash
which python
```

shows:

```text
/usr/bin/python
```

while you expected:

```text
project/.venv/bin/python
```

the environment may not be activated.

Check:

```bash
echo "$PATH"
```

and:

```bash
echo "$VIRTUAL_ENV"
```

---

## 92. Wrong Python Environment

Symptom:

```text
ModuleNotFoundError
```

even though you installed the package.

Possible cause:

```text
package installed into .venv
but script uses system Python
```

Check:

```bash
python -c \
'import sys; print(sys.executable)'
```

and:

```bash
python -m pip show package
```

---

## 93. Wrong pip

Bad workflow:

```bash
pip install boto3
python3 script.py
```

If `pip` and `python3` belong to different installations, the package may not be visible.

Prefer:

```bash
python3 -m pip install boto3
python3 script.py
```

or use the venv interpreter explicitly.

---

## 94. `pip check`

Run:

```bash
python -m pip check
```

This checks installed packages for dependency conflicts.

It is useful in CI validation.

---

## 95. CI Dependency Verification

A useful stage:

```bash
python -m pip check
```

Then:

```bash
pytest
```

This can catch broken dependency relationships before deployment.

---

## 96. Dependency Freeze Verification

You can inspect:

```bash
python -m pip freeze
```

and compare against the expected dependency artifact.

For serious reproducibility, use the project's chosen lock/constraints workflow rather than manually comparing freeze output.

---

## 97. `pip install --upgrade`

Avoid doing this blindly in production:

```bash
pip install --upgrade boto3
```

Instead:

```text
select version
test
scan
build
deploy
```

Controlled change is safer.

---

## 98. Dependency Updates in Production

Recommended:

```text
developer update
      |
      v
tests
      |
      v
security scan
      |
      v
CI
      |
      v
artifact
      |
      v
staging
      |
      v
production
```

---

## 99. Security Patch Workflow

When a critical CVE is found:

```text
identify affected dependency
        |
        v
select patched version
        |
        v
update dependency definition
        |
        v
run tests
        |
        v
run SCA/security scan
        |
        v
build
        |
        v
deploy
```

Track the change.

---

## 100. Dependency Pinning and Availability

Exact pinning improves reproducibility but creates upgrade responsibility.

If:

```text
boto3==old-version
```

is never updated, it can become:

```text
security risk
```

Therefore:

```text
pin
+
monitor
+
upgrade deliberately
```

---

## 101. Virtual Environment Is Not a Security Boundary

A venv provides dependency isolation.

It does not provide:

```text
container isolation
network isolation
IAM isolation
process isolation
secret protection
```

Do not treat `.venv` as a security boundary.

---

## 102. Virtual Environment Is Not Containerization

Compare:

```text
venv
 -> Python package/environment isolation

container
 -> filesystem/process/runtime isolation
```

They solve different problems.

---

## 103. Virtual Environment Is Not a VM

A VM provides:

```text
guest OS
kernel boundary
virtual hardware
```

A venv provides:

```text
Python environment isolation
```

Use the right abstraction for the problem.

---

## 104. Venv and Linux

A common DevOps VM workflow:

```text
EC2 instance
    |
    v
Python
    |
    v
.venv
    |
    +--> boto3
    +--> kubernetes
    +--> requests
```

This avoids polluting system Python.

---

## 105. Venv and EC2

Suppose an EC2 instance hosts:

```text
deployment tool
monitoring tool
cleanup tool
```

Use separate environments if dependencies differ:

```text
/opt/deploy-tool/.venv
/opt/monitor-tool/.venv
/opt/cleanup-tool/.venv
```

This reduces dependency interference.

---

## 106. Venv and Systemd

A systemd service can invoke the venv interpreter directly:

```ini
[Service]
WorkingDirectory=/opt/deploy-tool
ExecStart=/opt/deploy-tool/.venv/bin/python /opt/deploy-tool/app.py
Restart=on-failure
```

This avoids relying on shell activation.

---

## 107. Why Direct Interpreter in systemd?

systemd does not need:

```bash
source .venv/bin/activate
```

It can directly execute:

```text
.venv/bin/python
```

This is more explicit and reliable.

---

## 108. Venv and Cron

Cron can similarly use:

```bash
/opt/tool/.venv/bin/python \
    /opt/tool/cleanup.py
```

rather than assuming the interactive shell environment exists.

---

## 109. Cron Environment

Cron often has a minimal environment.

Do not assume:

```text
PATH
HOME
VIRTUAL_ENV
AWS_PROFILE
```

are identical to your interactive shell.

Use explicit paths and configuration.

---

## 110. Venv + AWS Credentials

A venv does not provide AWS credentials.

The AWS SDK still needs:

```text
IAM role
profile
environment
credential provider
```

The venv only controls Python packages.

---

## 111. Venv + Kubernetes Credentials

Likewise, a venv does not grant Kubernetes access.

The Python Kubernetes client still needs:

```text
kubeconfig
service account
cluster authentication
RBAC
```

Dependency isolation and authorization are separate concerns.

---

## 112. Venv + Terraform

If Python calls:

```bash
terraform
```

the venv manages Python packages, not Terraform itself.

Terraform availability depends on:

```text
PATH
binary
container
runner image
```

Keep these concerns separate.

---

## 113. Python CLI + Venv

A robust tool can be:

```text
Git repository
    |
    +--> Python source
    +--> requirements/lock
    |
    v
venv
    |
    +--> boto3
    +--> kubernetes
    +--> PyYAML
    |
    v
CLI
```

This is a common local/VM automation model.

---

## 114. Internal Python Packages

Large organizations may create:

```text
company_aws
company_k8s
company_logging
company_security
```

packages.

Install them into the venv:

```bash
python -m pip install \
    company-k8s
```

Use an approved internal package repository.

---

## 115. Editable Install

During local development:

```bash
python -m pip install -e .
```

This installs a package in editable mode.

Changes to source can be reflected without reinstalling after every modification.

Do not assume editable installs are appropriate for production images.

---

## 116. `pyproject.toml`

Modern Python projects commonly use:

```text
pyproject.toml
```

for project metadata and tooling configuration.

It can define:

```text
project metadata
dependencies
build configuration
tool configuration
```

The exact dependency workflow depends on the chosen tool.

---

## 117. Simple `pyproject.toml`

Conceptually:

```toml
[project]
name = "devops-tool"
version = "1.0.0"
requires-python = ">=3.11"

dependencies = [
    "boto3",
    "kubernetes",
    "PyYAML",
]
```

This is an alternative to maintaining only a `requirements.txt`.

---

## 118. `requirements.txt` vs `pyproject.toml`

Use the convention adopted by the project.

`requirements.txt` is straightforward for:

```text
deployment scripts
simple automation
pip-based environments
```

`pyproject.toml` is useful for:

```text
packaged applications
libraries
modern Python tooling
project metadata
```

Do not mix dependency sources without understanding precedence.

---

## 119. Dependency Locking Tools

Python teams may use tools such as:

```text
pip-tools
Poetry
uv
PDM
```

The appropriate choice depends on organizational standards.

The important DevOps requirement is:

```text
repeatable dependency resolution
```

---

## 120. `pip-tools`

A common workflow uses:

```text
requirements.in
       |
       v
pip-compile
       |
       v
requirements.txt
```

Conceptually:

```text
direct dependencies
        |
        v
resolved versions
        |
        v
reproducible installation
```

---

## 121. Direct Dependencies File

Example:

```text
requirements.in

boto3
kubernetes
PyYAML
```

Then a resolver generates a pinned output file.

This separates:

```text
what the application needs
```

from:

```text
exact resolved dependency graph
```

---

## 122. Poetry / uv / PDM

Modern Python dependency tools can manage:

```text
virtual environments
dependencies
lock files
project metadata
```

Use the tool approved by your team.

Do not introduce a new dependency manager into a mature repository without understanding the existing workflow.

---

## 123. Why DevOps Engineers Should Understand Dependency Managers

Your CI/CD pipeline may need to:

```text
create environment
install dependencies
run tests
build package
build image
scan dependencies
```

If you understand only `pip install`, you may struggle with production Python projects.

---

## 124. Build Reproducibility

A strong build defines:

```text
Python version
dependency versions
base image
OS packages
source commit
build commands
```

Then:

```text
same commit
+
same controlled inputs
=
reproducible artifact
```

---

## 125. Dependency Artifacts

The CI pipeline can generate:

```text
wheel
source distribution
container image
requirements lock
SBOM
```

These artifacts should be traceable to:

```text
Git commit
build ID
dependency versions
```

---

## 126. SBOM

A Software Bill of Materials can describe:

```text
application dependencies
versions
transitive components
```

This helps with:

```text
vulnerability management
compliance
incident response
supply-chain visibility
```

---

## 127. Python Dependency SBOM in DevSecOps

Pipeline:

```text
source
  |
  v
dependency resolution
  |
  v
build
  |
  v
SBOM
  |
  v
vulnerability scan
  |
  v
policy
```

A vulnerable dependency can block the pipeline according to policy.

---

## 128. Dependency Scanning in CI

Example conceptual pipeline:

```text
Checkout
   |
   v
Create venv
   |
   v
Install dependencies
   |
   v
SCA / Trivy
   |
   v
Tests
   |
   v
Build image
   |
   v
Image scan
```

This is a DevSecOps-friendly approach.

---

## 129. Production Python Image

A production image should ideally contain:

```text
minimal base
runtime dependencies
application code
```

not:

```text
compiler toolchain
pytest
source control metadata
.venv from developer machine
```

unless required.

---

## 130. Multi-Stage Dependency Build

Concept:

```text
builder image
    |
    +--> compilers
    +--> dependency build
    |
    v
runtime image
    |
    +--> required runtime artifacts
```

This can reduce:

```text
image size
attack surface
```

---

## 131. Dependency Installation Failure in Docker

Common causes:

```text
missing compiler
missing system library
unsupported Python version
bad package version
network failure
private repository authentication
architecture mismatch
```

Compare local and container environments.

---

## 132. ARM vs x86

A dependency may behave differently across:

```text
amd64
arm64
```

Especially when native extensions are involved.

CI should build/test for the architectures that production uses.

---

## 133. Python Package Cache

Pip may cache downloaded packages.

For Docker:

```bash
pip install --no-cache-dir \
    -r requirements.txt
```

can reduce retained cache data in the image.

For CI, external caching may improve performance.

---

## 134. Offline Dependency Installation

You can pre-download packages:

```bash
python -m pip download \
    -r requirements.txt \
    -d packages/
```

Then install from an approved local directory/repository.

This can support restricted environments.

---

## 135. Internal Artifact Repository

Enterprise pattern:

```text
PyPI
 |
 v
approved/internal repository
 |
 v
CI
 |
 v
Python environment
 |
 v
artifact/image
```

This gives organizations more control over package sources.

---

## 136. Package Integrity

Use:

```text
trusted repository
version control
hash verification where appropriate
dependency scanning
artifact provenance
```

Do not blindly install arbitrary packages from unknown sources.

---

## 137. Typosquatting Risk

Attackers may publish packages with names similar to popular packages.

Example concept:

```text
real-package
real_packge
```

Always verify:

```text
package name
source
maintainer
internal approval
version
```

---

## 138. Dependency Review

Before adding a package, ask:

```text
Do we actually need it?
Is it maintained?
Is it trusted?
Does it have vulnerabilities?
Does it introduce many dependencies?
Can the standard library solve this?
```

Fewer dependencies generally mean lower operational complexity.

---

## 139. Standard Library First

For simple tasks, prefer built-ins when practical:

```text
argparse
logging
pathlib
json
csv
datetime
subprocess
os
re
```

Avoid adding a dependency for trivial functionality.

---

## 140. Dependency Minimization

If:

```text
10-line standard-library solution
```

solves the problem reliably, adding:

```text
large external package
```

may not be justified.

Evaluate:

```text
security
maintenance
performance
developer productivity
```

---

## 141. Virtual Environment Troubleshooting

Problem:

```text
ModuleNotFoundError
```

Check:

```bash
which python
python -m pip --version
python -m pip show package
```

Then:

```bash
python -c \
'import sys; print(sys.executable)'
```

---

## 142. Problem — Package Installed but Import Fails

Possible reasons:

```text
wrong interpreter
wrong venv
wrong package name
package not installed
multiple Python versions
```

Use:

```bash
python -m pip list
```

and:

```bash
python -c "import package"
```

---

## 143. Problem — `pip` Command Not Found

Use:

```bash
python3 -m pip
```

If pip itself is unavailable, create a proper venv or install the OS-supported Python/pip package according to the platform.

---

## 144. Problem — Venv Creation Fails

Possible:

```text
venv module unavailable
Python installation incomplete
OS package missing
permission problem
disk full
```

Check:

```bash
python3 --version
python3 -m venv --help
df -h
```

and the operating system's Python packaging.

---

## 145. Problem — Permission Denied

Do not immediately use:

```bash
sudo pip install
```

Instead determine:

```text
which Python
which pip
who owns environment
where is venv
```

Create the environment in a directory you own.

---

## 146. Problem — Environment Uses Wrong Python Version

Check:

```bash
.venv/bin/python --version
```

If incorrect, recreate:

```bash
rm -rf .venv
python3.12 -m venv .venv
```

Then reinstall dependencies.

---

## 147. Problem — Dependency Conflict

Use:

```bash
python -m pip check
```

Then inspect:

```bash
python -m pip show package
```

Review the dependency graph and constraints.

---

## 148. Problem — CI Works Locally but Fails in CI

Compare:

```text
Python version
OS
architecture
dependency versions
environment variables
system libraries
network/package index
```

A clean local venv can reproduce CI conditions better than system Python.

---

## 149. Problem — Production Works but Developer Machine Fails

Possible:

```text
developer using different Python
different dependency versions
missing system libraries
different environment variables
```

Recreate from:

```text
documented Python version
dependency lock
bootstrap process
```

---

## 150. Problem — `pip freeze` Is Huge

This often means the environment contains unrelated packages.

Create a clean venv:

```bash
python3 -m venv .venv
```

Install only project dependencies.

Then generate/maintain the dependency artifact from the clean environment if that matches your project's workflow.

---

## 151. Problem — Requirements File Contains Unnecessary Packages

`pip freeze` can include transitive dependencies.

For a mature project, identify:

```text
direct dependencies
transitive dependencies
development dependencies
```

and manage them intentionally.

---

## 152. Problem — Security Scanner Finds Old Package

Process:

```text
identify dependency
   |
   v
check CVE
   |
   v
check fixed versions
   |
   v
update
   |
   v
test
   |
   v
scan
```

Do not suppress a vulnerability without understanding the risk.

---

## 153. Problem — New Dependency Breaks Automation

Check:

```text
API changes
Python compatibility
transitive dependencies
behavior changes
```

Use:

```text
staging
automated tests
canary rollout
rollback
```

for critical automation.

---

## 154. Problem — Dependency Download Is Slow

Possible:

```text
network latency
PyPI throttling
large packages
no CI cache
internal repository issue
```

Use approved:

```text
package cache
artifact repository
CI cache
```

without sacrificing dependency integrity.

---

## 155. Problem — Private Repository Authentication Fails

Check:

```text
repository URL
credential provider
permissions
token expiry
network
certificate
```

Do not print credentials while debugging.

Use sanitized diagnostic logs.

---

## 156. Problem — Docker Build Has Different Dependencies

Check:

```text
requirements file copied
correct file used
build cache
Python version
base image
architecture
```

Use explicit Docker build stages and inspect the installed package versions.

---

## 157. Problem — `.venv` Accidentally Committed

Remove it from Git:

```bash
git rm -r --cached .venv
```

Then add:

```text
.venv/
```

to `.gitignore`.

If secrets were committed, follow the organization's credential rotation and repository remediation process.

---

## 158. Problem — `.venv` Included in Docker Image

Add:

```text
.venv/
```

to:

```text
.dockerignore
```

Then rebuild the image.

---

## 159. Problem — System Python Broken

If global pip installations modified system Python:

```text
stop making additional global changes
```

Determine the OS package ownership and repair using the operating system's package-management mechanism.

Do not attempt random `pip uninstall` operations against system packages.

---

## 160. Virtual Environment and Ansible

If Python automation is used with Ansible tooling, understand that:

```text
controller environment
```

and:

```text
managed node Python
```

can be different environments.

The venv on the controller does not automatically control Python on remote hosts.

---

## 161. Virtual Environment and Jenkins

Jenkins can:

```text
create venv
install dependencies
run scripts
destroy workspace
```

Example:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install \
    -r requirements.txt
.venv/bin/python deploy.py \
    --environment staging
```

---

## 162. Virtual Environment and GitHub Actions

Typical:

```text
checkout
   |
   v
setup Python
   |
   v
create/install environment
   |
   v
test
   |
   v
build
```

The exact setup may use the platform's Python tooling and caching features.

---

## 163. Virtual Environment and GitLab CI

Typical:

```yaml
script:
  - python -m venv .venv
  - .venv/bin/python -m pip install -r requirements.txt
  - .venv/bin/python -m pytest
```

Using the venv interpreter directly avoids shell activation dependency.

---

## 164. Virtual Environment and Jenkins Shared Libraries

If a Jenkins shared automation layer invokes Python:

```text
Jenkins
   |
   v
approved Python environment
   |
   v
automation script
```

Standardize:

```text
Python version
dependencies
logging
exit codes
```

across jobs.

---

## 165. Virtual Environment and Cron

Use:

```cron
0 2 * * * /opt/tool/.venv/bin/python /opt/tool/cleanup.py
```

rather than:

```cron
0 2 * * * python cleanup.py
```

The first is explicit about which Python environment is required.

---

## 166. Virtual Environment and systemd

Use:

```ini
ExecStart=/opt/tool/.venv/bin/python /opt/tool/app.py
```

This is deterministic.

Also define:

```text
WorkingDirectory
User
Environment
Restart policy
Timeouts
```

according to the service requirements.

---

## 167. Virtual Environment and Docker

Use:

```text
venv -> local/VM dependency isolation
container -> runtime packaging/isolation
```

Do not copy a developer's `.venv` into the image.

Build dependencies from controlled dependency definitions.

---

## 168. Production Dependency Architecture

```text
Git
 |
 +--> requirements/lock
 |
 v
CI
 |
 +--> clean Python
 |
 +--> dependency install
 |
 +--> SCA scan
 |
 +--> tests
 |
 v
Docker build
 |
 v
image
 |
 v
ECR
 |
 v
EKS
```

This is a practical DevOps dependency lifecycle.

---

## 169. Dependency Promotion

Do not install dependencies independently in:

```text
dev
staging
production
```

Instead:

```text
build once
   |
   v
test
   |
   v
scan
   |
   v
promote artifact
```

This reduces environment drift.

---

## 170. Immutable Build Principle

A strong production workflow:

```text
source commit
      |
      v
dependency resolution
      |
      v
build artifact
      |
      v
test
      |
      v
scan
      |
      v
publish
      |
      v
promote same artifact
```

Avoid rebuilding different dependency sets for each environment.

---

## 171. Dependency Provenance

Track:

```text
Git commit
Python version
dependency versions
base image
build ID
image digest
```

This helps answer:

```text
What exactly is running in production?
```

---

## 172. Production Incident — Dependency Upgrade

Suppose:

```text
boto3 upgraded
   |
   v
automation starts failing
```

Investigate:

```text
previous version
new version
release notes
dependency tree
CI tests
runtime error
```

Then:

```text
rollback
or
patch
```

depending on the impact.

---

## 173. Production Incident — ModuleNotFoundError

Example:

```text
ModuleNotFoundError:
No module named 'kubernetes'
```

Check:

```bash
which python
python -m pip show kubernetes
```

Then:

```bash
python -c \
'import sys; print(sys.executable)'
```

Most often, the first question is:

> **Which Python interpreter is actually running the script?**

---

## 174. Production Incident — Different Behavior in CI

Compare:

```text
Python
pip
dependencies
OS
architecture
environment
```

Run:

```bash
python --version
python -m pip freeze
python -m pip check
```

in the relevant environments.

---

## 175. Production Incident — CVE in Dependency

Do:

```text
identify affected package
check exploitability
find patched version
test
scan
release
deploy
```

If the package is deeply transitive, determine which direct dependency introduced it.

---

## 176. Production Incident — Broken Build After Python Upgrade

Possible:

```text
package does not support new Python
native extension fails
dependency resolver changes
deprecated API
```

Check the package compatibility matrix and pin a supported combination while preparing the upgrade.

---

## 177. Production Incident — Package Works on x86 but Not ARM

Check:

```text
wheel availability
native dependencies
compiler
architecture-specific code
base image
```

Build/test for the production architecture.

---

## 178. Production Incident — CI Dependency Cache Is Stale

Symptoms:

```text
requirements changed
CI still installs old package
```

Check cache key.

Use a key based on:

```text
Python version
dependency file hash
OS
architecture
```

Invalidate cache when dependency definitions change.

---

## 179. Production Incident — Developer's `.venv` Is Broken

Fast recovery:

```bash
deactivate
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

If the environment is reproducible, rebuilding it should be routine.

---

## 180. Production Incident — Disk Full During pip Install

Check:

```bash
df -h
du -sh ~/.cache/pip
```

Potential cleanup:

```bash
python -m pip cache purge
```

only when appropriate.

Also investigate:

```text
old virtual environments
Docker layers
CI workspace
logs
artifacts
```

---

## 181. Production Incident — Wrong Package Source

A dependency may be installed from an unexpected index.

Verify:

```bash
python -m pip config list
```

and inspect the package metadata/source according to your environment.

Enterprise systems should use approved package repositories.

---

## 182. Production Incident — Dependency Supply-Chain Alert

If a package is suspected malicious:

```text
stop promotion
isolate affected builds
identify versions
identify environments
rotate affected secrets if exposure is possible
remove package
rebuild
scan
investigate provenance
```

Do not assume deleting the package from the local venv is sufficient.

---

## 183. Production Incident — Secret in Dependency Installation Logs

If credentials appeared in:

```text
CI logs
pip command
package repository URL
```

treat them as exposed.

Follow credential rotation and log-access remediation procedures.

Do not simply delete the local environment and continue.

---

## 184. Virtual Environment Best Practices

```text
1. Use one environment per project.
2. Do not install project packages globally.
3. Do not commit .venv.
4. Define supported Python versions.
5. Control dependency versions.
6. Use reproducible dependency resolution.
7. Keep runtime and development dependencies separate.
8. Use clean CI environments.
9. Scan dependencies.
10. Use approved package repositories.
11. Avoid secrets in pip commands.
12. Use python -m pip.
13. Validate dependency health with pip check.
14. Keep Docker builds independent of developer .venv.
15. Rebuild environments instead of manually repairing them.
16. Track dependency changes.
17. Test upgrades before production.
18. Use explicit Python interpreters in cron/systemd.
19. Cache dependencies carefully in CI.
20. Promote tested artifacts rather than rebuilding unpredictably.
```

---

## 185. Virtual Environment Anti-Patterns

Avoid:

```text
sudo pip install ...
pip install into system Python
committing .venv
copying .venv into Docker
uncontrolled pip upgrade in production
unbounded dependency versions
ignoring security vulnerabilities
using unknown package repositories
mixing Python installations
assuming activation is required everywhere
passing secrets in CLI arguments
rebuilding dependencies differently per environment
```

---

## 186. Daily DevOps Commands

Create:

```bash
python3 -m venv .venv
```

Activate:

```bash
source .venv/bin/activate
```

Deactivate:

```bash
deactivate
```

Install:

```bash
python -m pip install -r requirements.txt
```

List:

```bash
python -m pip list
```

Freeze:

```bash
python -m pip freeze
```

Check:

```bash
python -m pip check
```

Show:

```bash
python -m pip show boto3
```

Verify interpreter:

```bash
python -c \
'import sys; print(sys.executable)'
```

---

## 187. Daily DevOps Workflow

```text
clone repository
      |
      v
check Python version
      |
      v
create .venv
      |
      v
activate
      |
      v
install dependencies
      |
      v
pip check
      |
      v
run tests
      |
      v
run security scan
      |
      v
build
      |
      v
deploy
```

---

## 188. Quick Troubleshooting Commands

```bash
which python
```

```bash
which pip
```

```bash
python --version
```

```bash
python -m pip --version
```

```bash
python -m pip list
```

```bash
python -m pip check
```

```bash
python -c \
'import sys; print(sys.executable)'
```

```bash
python -c \
'import site; print(site.getsitepackages())'
```

These quickly reveal environment problems.

---

## 189. `sys.path`

Python searches import locations using:

```python
import sys

print(sys.path)
```

This can help diagnose:

```text
wrong package
local module shadowing
unexpected environment
```

Do not modify `sys.path` as a first-line dependency-management solution.

---

## 190. Package Shadowing

A local file named:

```text
requests.py
```

can shadow the real:

```text
requests
```

package.

Similarly:

```text
boto3.py
logging.py
json.py
```

can create confusing import behavior.

Choose project/module names carefully.

---

## 191. Virtual Environment and `PYTHONPATH`

Avoid relying on global:

```bash
PYTHONPATH
```

for dependency management.

It can cause:

```text
unexpected imports
environment differences
hard-to-debug behavior
```

Use proper packages and environments.

---

## 192. `site-packages`

Installed third-party packages normally live under the environment's:

```text
site-packages
```

Example concept:

```text
.venv/
└── lib/
    └── python3.x/
        └── site-packages/
```

This is where packages installed into the venv are available.

---

## 193. Venv Internals

A typical Linux venv contains:

```text
.venv/
├── bin/
├── include/
├── lib/
├── lib64/
└── pyvenv.cfg
```

Exact structure varies by platform.

---

## 194. `pyvenv.cfg`

This file records information about the environment's base Python installation.

It helps Python understand the environment relationship.

Do not manually edit it as part of normal DevOps operations.

---

## 195. Venv Portability

A virtual environment should generally be recreated rather than copied between:

```text
machines
OS distributions
Python versions
architectures
```

because paths and binaries can be environment-specific.

---

## 196. Rebuild Over Copy

Preferred:

```text
source
+
dependency definition
+
Python version
=
new venv
```

rather than:

```text
copy old .venv
```

This is more reproducible.

---

## 197. Venv Lifecycle

```text
create
  |
  v
install
  |
  v
develop/test
  |
  v
upgrade
  |
  v
recreate
```

Do not become dependent on manually modified local environments.

---

## 198. Dependency Upgrade Strategy

For each upgrade:

```text
1. identify reason
2. update dependency
3. resolve dependencies
4. run unit tests
5. run integration tests
6. run security scan
7. build image
8. deploy to staging
9. validate
10. promote
```

This is especially important for automation that changes production infrastructure.

---

## 199. Interview — What Is a Virtual Environment?

### Answer

> A Python virtual environment isolates project-specific Python packages and tooling from the system Python installation. I use one environment per automation project so dependency versions do not interfere with other projects.

---

## 200. Interview — Why Not Install Packages Globally?

### Answer

> Global installation can create dependency conflicts and can even interfere with OS-managed Python packages. A virtual environment gives each project its own dependency set and makes local development and CI more reproducible.

---

## 201. Interview — How Do You Create a Venv?

### Answer

```bash
python3 -m venv .venv
```

Then:

```bash
source .venv/bin/activate
```

and install:

```bash
python -m pip install -r requirements.txt
```

---

## 202. Interview — Why Use `python -m pip`?

### Answer

> It ensures pip runs under the Python interpreter I'm actually using. This avoids a common problem where `pip` belongs to one Python installation while the script is executed by another.

---

## 203. Interview — Do You Need a Venv Inside Docker?

### Answer

> Not necessarily. A container already provides runtime isolation, so many production Python images install dependencies directly into the container's Python environment. I still use controlled dependency definitions and don't copy a developer's `.venv` into the image.

---

## 204. Interview — How Do You Make Python Dependencies Reproducible?

### Answer

> I control the Python version, explicitly manage dependencies, use pinned or locked dependency resolution appropriate to the project, test the complete dependency set in clean CI environments, and build a known artifact that is promoted across environments.

---

## 205. Interview — What Is `pip freeze`?

### Answer

> `pip freeze` outputs installed packages and their versions. It can be useful for capturing an environment snapshot, but it can include transitive or unrelated packages, so I prefer an intentional dependency-management strategy for mature projects.

---

## 206. Interview — What Is `pip check`?

### Answer

> `pip check` verifies whether installed packages have compatible dependency requirements. I can run it in CI as an additional dependency-health check.

---

## 207. Interview — How Do You Handle Dependency Vulnerabilities?

### Answer

> I identify the affected dependency and CVE, determine a patched compatible version, update the dependency definition, run tests and security scans, rebuild the artifact, and promote the tested version. I avoid blindly upgrading or suppressing vulnerabilities without understanding the risk.

---

## 208. Interview — How Do You Avoid Dependency Drift?

### Answer

> I use a controlled Python version, dependency definitions or lock files, clean CI environments, dependency scanning, and artifact promotion. I avoid manually installing different versions in each environment.

---

## 209. Interview — How Do You Run Python From systemd?

### Answer

> I point `ExecStart` directly to the virtual environment's Python interpreter, for example `/opt/tool/.venv/bin/python /opt/tool/app.py`. systemd doesn't need shell activation.

---

## 210. Interview — How Do You Run a Venv Script From Cron?

### Answer

> I use the absolute interpreter path, such as `/opt/tool/.venv/bin/python /opt/tool/cleanup.py`, because cron has a minimal environment and shouldn't depend on interactive shell activation.

---

## 211. Interview — What Is the Difference Between Venv and Docker?

### Answer

> A virtual environment isolates Python dependencies. Docker provides broader runtime isolation around the process, filesystem, and packaged environment. They solve different problems and can be used together, although a venv is often unnecessary inside a single-application production container.

---

## 212. Interview — How Would You Troubleshoot `ModuleNotFoundError`?

### Answer

> First I check which interpreter is running with `python -c "import sys; print(sys.executable)"`. Then I use `python -m pip --version`, `python -m pip show <package>`, and `python -m pip list` to verify the package is installed in the same environment.

---

## 213. Interview — How Would You Handle Python Dependencies in Jenkins?

### Answer

> I would create a clean environment during the job or use a controlled build image, install dependencies from the repository's dependency definition, run `pip check` and tests, perform security scanning, and then build the deployment artifact.

---

## 214. Interview — How Would You Handle Python Dependencies in GitLab CI?

### Answer

> I would explicitly configure the Python version, create or use a clean environment, install controlled dependencies, run tests and security scans, and use caching carefully with keys based on the Python version and dependency definition.

---

## 215. Scenario — `boto3` Works Locally but Not in CI

Check:

```text
Python version
venv
pip version
requirements file
CI installation step
package cache
```

Then verify:

```bash
python -m pip show boto3
```

inside CI.

---

## 216. Scenario — `kubernetes` Package Is Installed but Import Fails

Check:

```bash
python -c \
'import sys; print(sys.executable)'
```

Then:

```bash
python -m pip show kubernetes
```

If they refer to different environments, fix the interpreter selection.

---

## 217. Scenario — `pip install` Changes Production Behavior

Likely cause:

```text
uncontrolled dependency upgrade
```

Fix:

```text
pin/lock
test
scan
build
promote
```

Do not install latest versions directly on production servers.

---

## 218. Scenario — Security Scanner Finds a CVE in a Transitive Dependency

Trace:

```text
vulnerable package
      |
      v
parent dependency
      |
      v
direct project dependency
```

Then determine whether:

```text
upgrade parent
constraint transitive version
replace package
```

is the safest solution.

---

## 219. Scenario — Python Upgrade Breaks Automation

Check:

```text
package compatibility
deprecated APIs
native extensions
dependency resolver
OS libraries
```

A temporary pin to a known-good Python/dependency combination may be appropriate while preparing a proper upgrade.

---

## 220. Scenario — Developer Committed `.venv`

Fix:

```bash
git rm -r --cached .venv
```

Add:

```text
.venv/
```

to:

```text
.gitignore
```

Then verify that no sensitive data was committed.

---

## 221. Scenario — Docker Image Contains `.venv`

Check:

```text
.dockerignore
Dockerfile COPY instructions
build context
```

Add:

```text
.venv/
```

to `.dockerignore` and rebuild.

---

## 222. Scenario — CI Cache Uses Old Dependencies

Check cache key.

It should vary with:

```text
Python version
requirements/lock hash
OS
architecture
```

Invalidate stale caches when necessary.

---

## 223. Scenario — Dependency Installation Needs Internet but Production Has None

Use:

```text
internal artifact repository
approved package mirror
prebuilt wheels
offline package cache
```

Build the artifact in a controlled environment and promote it.

---

## 224. Scenario — Private Package Installation Fails

Check:

```text
repository endpoint
authentication
permissions
network
certificate
package version
```

Do not print authentication tokens while troubleshooting.

---

## 225. Scenario — Production Server Has Multiple Python Projects

Use separate environments:

```text
/opt/project-a/.venv
/opt/project-b/.venv
/opt/project-c/.venv
```

Each project gets controlled dependencies.

---

## 226. Scenario — Cron Uses Wrong Package Version

Do not rely on:

```text
active shell
```

Use:

```bash
/opt/project/.venv/bin/python \
    /opt/project/job.py
```

Then verify:

```bash
/opt/project/.venv/bin/python \
    -c "import package; print(package.__version__)"
```

---

## 227. Scenario — System Python Has a Broken Package

Do not blindly repair it with:

```bash
sudo pip install ...
```

First identify whether the package is:

```text
OS-managed
application-managed
```

Then use the appropriate package-management mechanism.

---

## 228. Scenario — Two Tools Need Different `boto3` Versions

Use separate environments:

```text
tool-a/.venv
    boto3 version A

tool-b/.venv
    boto3 version B
```

Do not force both projects into the same global package set.

---

## 229. Scenario — Need Reproducible Developer Setup

Provide:

```text
Python version
bootstrap command
dependency definition
README
optional Makefile/task runner
```

Example:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

---

## 230. Scenario — Need Reproducible CI Setup

Use:

```text
known Python image/version
clean environment
controlled dependencies
dependency cache
tests
security scans
artifact build
```

Avoid depending on preinstalled global packages.

---

## 231. Scenario — Need Reproducible Production

Prefer:

```text
build artifact once
   |
   v
test
   |
   v
scan
   |
   v
publish
   |
   v
promote
```

Do not run:

```text
pip install latest
```

directly on production servers.

---

## 232. Scenario — Dependency Repository Is Unavailable

A robust pipeline should have a controlled strategy such as:

```text
approved cache
internal mirror
artifact repository
prebuilt artifact
```

Do not bypass security controls by switching to an unknown package source.

---

## 233. Scenario — Package Upgrade Needs Emergency Rollback

Keep:

```text
previous dependency definition
previous artifact
previous image digest
```

Then rollback the artifact rather than manually reinstalling arbitrary package versions.

---

## 234. Scenario — Python Container Has Huge Image

Check:

```text
development dependencies
pip cache
compiler packages
unnecessary OS packages
.venv copied from host
source artifacts
```

Use:

```text
minimal runtime base
multi-stage build
runtime-only dependencies
.dockerignore
```

where appropriate.

---

## 235. Scenario — Python Container Has Vulnerable Dependencies

Check:

```text
requirements/lock
base image
OS packages
Python packages
transitive dependencies
```

Dependency security includes both:

```text
application packages
+
container operating-system components
```

---

## 236. Scenario — Need Fast Local Rebuild

Use:

```text
dependency cache
wheel cache
controlled venv
```

but make sure the cache does not override the dependency definition.

Speed should not come at the cost of reproducibility.

---

## 237. Scenario — Venv Is Corrupted

The preferred response is usually:

```bash
deactivate
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

A reproducible environment should be disposable.

---

## 238. Scenario — Package Installation Is Non-Deterministic

Check:

```text
unpinned dependencies
unlocked transitive dependencies
changing package index
Python version
OS
architecture
```

Use a controlled lock/resolution strategy.

---

## 239. Scenario — Dependency Has No Compatible Wheel

Options:

```text
choose compatible package version
install build dependencies
build wheel internally
replace dependency
```

Do not simply install arbitrary system packages into production without testing.

---

## 240. Scenario — Security Policy Blocks a Dependency

Evaluate:

```text
business necessity
alternative package
patched version
internal approval
exception process
```

Do not bypass the policy in CI without documented authorization.

---

## 241. Scenario — Need Internal Python Package

Use an approved repository:

```text
Artifactory
internal package registry
approved package source
```

Define the dependency in the project dependency file and test it through CI.

---

## 242. Scenario — Need Package Version Across Many Teams

A central constraints or dependency policy can provide:

```text
approved version
security baseline
compatibility
```

Teams can consume the approved version rather than independently choosing vulnerable releases.

---

## 243. Scenario — Dependency Update Breaks Only Production

Possible:

```text
production Python differs
OS differs
architecture differs
environment variables differ
system libraries differ
```

Compare the full runtime environment rather than only the Python package list.

---

## 244. Scenario — Need Auditability

Track:

```text
source commit
dependency definition commit
build ID
Python version
dependency versions
image digest
deployment version
```

This allows incident responders to reconstruct what was deployed.

---

## 245. Enterprise Python Dependency Flow

```text
Developer
   |
   v
Git
   |
   v
Dependency definition
   |
   v
CI
   |
   +--> dependency resolution
   |
   +--> tests
   |
   +--> SCA
   |
   +--> SBOM
   |
   v
artifact
   |
   v
Artifactory / ECR
   |
   v
deployment
   |
   v
EKS / EC2
```

---

## 246. End-to-End DevOps Example

Project:

```text
eks-deployment-tool/
```

Dependencies:

```text
boto3
kubernetes
PyYAML
```

Development:

```text
pytest
ruff
```

Workflow:

```bash
python3.12 -m venv .venv

source .venv/bin/activate

python -m pip install \
    -r requirements.txt

python -m pip check

pytest

python deploy.py \
    --service payment \
    --environment staging \
    --version abc123
```

---

## 247. End-to-End CI Example

```bash
set -e

python --version

python -m venv .venv

.venv/bin/python -m pip install \
    -r requirements.txt

.venv/bin/python -m pip check

.venv/bin/python -m pytest

.venv/bin/python deploy.py \
    --environment staging \
    --service payment \
    --version "$CI_COMMIT_SHA"
```

The exact deployment command depends on the pipeline's approval model.

---

## 248. End-to-End Docker Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN python -m pip install \
    --no-cache-dir \
    -r requirements.txt

COPY src/ ./src/

CMD ["python", "src/app.py"]
```

Use a non-root runtime user and additional hardening according to production requirements.

---

## 249. End-to-End EKS Flow

```text
Git
 |
 v
CI
 |
 +--> Python dependency install
 +--> tests
 +--> SCA
 |
 v
Docker build
 |
 v
ECR
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
Prometheus/Grafana
 |
 v
ELK
```

The Python environment is one component of the broader DevOps lifecycle.

---

## 250. Final Virtual Environment Mental Model

```text
                  PYTHON PROJECT
                        |
                dependency definition
                        |
                        v
                 virtual environment
                        |
          +-------------+-------------+
          |             |             |
        Python        pip          packages
          |             |             |
          +-------------+-------------+
                        |
                        v
                     tests
                        |
                        v
                  security scan
                        |
                        v
                    CI / build
                        |
                        v
                   artifact/image
                        |
                        v
                    production
```

Remember:

```text
venv
  =
Python dependency isolation

requirements/lock
  =
dependency reproducibility

CI
  =
clean verification

security scan
  =
dependency risk detection

artifact
  =
tested deployable unit

production
  =
promote the tested artifact
```

The key DevOps mindset is:

> **A Python environment should be disposable, reproducible, secure, and identical enough across development and CI to make automation predictable.**

---

## 251. Final Best-Practice Checklist

```text
Environment
[ ] Supported Python version defined
[ ] One environment per project where appropriate
[ ] .venv ignored by Git
[ ] .venv excluded from Docker build context
[ ] Explicit interpreter used in cron/systemd

Dependencies
[ ] Direct dependencies identified
[ ] Versions controlled
[ ] Transitive dependencies understood
[ ] Dependency conflicts checked
[ ] pip check used where appropriate
[ ] Lock/constraints strategy defined

Security
[ ] Dependencies scanned
[ ] Approved package source used
[ ] Secrets not passed to pip commands
[ ] Vulnerabilities tracked
[ ] SBOM generated where required

CI/CD
[ ] Clean Python environment
[ ] Python version controlled
[ ] Dependency cache keyed correctly
[ ] Tests executed
[ ] Security scans executed
[ ] Artifact built once
[ ] Artifact promoted

Containers
[ ] Minimal runtime dependencies
[ ] No developer .venv copied
[ ] No unnecessary build tools
[ ] Base image controlled
[ ] Image scanned

Operations
[ ] Dependency upgrades tested
[ ] Rollback artifact available
[ ] Environment drift minimized
[ ] Production installation not done with uncontrolled pip
```

---

## 252. Python for DevOps — Section 01/02 Completion

The Python fundamentals and intermediate sections now cover:

```text
01-Python-Fundamentals/
├── 01-Python-Introduction.md
├── 02-Variables-and-Data-Types.md
├── 03-Operators.md
├── 04-Conditional-Statements.md
├── 05-Loops.md
├── 06-Functions.md
├── 07-Strings.md
├── 08-Lists-Tuples-Sets.md
├── 09-Dictionaries.md
└── 10-Exception-Handling.md
```

and:

```text
02-Python-Intermediate/
├── 01-Modules-and-Packages.md
├── 02-File-Handling.md
├── 03-OS-and-Sys-Modules.md
├── 04-Regex.md
├── 05-JSON-YAML-CSV.md
├── 06-Datetime-and-Time.md
├── 07-Logging.md
├── 08-Argparse.md
└── 09-Virtual-Environments.md
```

These are not intended as generic Python-only notes. The emphasis is on:

```text
Python fundamentals
        +
Linux
        +
AWS
        +
Kubernetes/EKS
        +
Terraform
        +
CI/CD
        +
DevSecOps
        +
production automation
```

---

## 253. Next Python Section

The next planned section should move beyond basic/intermediate Python into more practical DevOps automation.

Recommended structure:

```text
03-Python-Advanced/
├── 01-OOP-for-DevOps.md
├── 02-Decorators.md
├── 03-Generators-and-Iterators.md
├── 04-Context-Managers.md
├── 05-Advanced-Functions.md
├── 06-Concurrency-and-Threading.md
├── 07-Asyncio.md
├── 08-Subprocess-and-System-Automation.md
├── 09-API-Automation.md
└── 10-Python-Testing-for-DevOps.md
```

The advanced section should continue the same principle:

> **Do not reduce the practical content just because a topic is smaller. Expand large topics and keep smaller topics focused, while maintaining enough real-world DevOps examples, scripts, troubleshooting, production practices, and interview preparation.**
