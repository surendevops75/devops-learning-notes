# Python-Package-Management

## 1. Purpose

Python package management is a core part of modern DevOps because Python
is widely used for automation, APIs, data services, platform tooling,
CI/CD utilities and infrastructure automation.

Production package management is much more than:

```bash
pip install requests
```

A controlled enterprise flow looks like:

```text
Git
 |
v
CI
 |
+--> Python version
+--> package manager
+--> dependency declaration
+--> lock/constraints
+--> private package repository
+--> isolated environment
+--> install
+--> test
+--> security
+--> build
+--> package
+--> publish
 |
v
Artifact Repository
 |
v
Container / Deployment
```

This file covers Python packaging fundamentals, pip, virtual
environments, requirements files, `pyproject.toml`, build backends,
wheels, source distributions, dependency resolution, constraints,
private repositories, Artifactory, authentication, package indexes,
dependency groups, Poetry concepts, uv concepts, pip-tools concepts,
CI/CD, Jenkins, GitHub Actions, Docker, Kubernetes, security,
reproducibility, SBOM, publishing, monorepos, troubleshooting,
production architecture and interview preparation.

---

# PART I — PYTHON PACKAGE MANAGEMENT FUNDAMENTALS

## 2. What Is Python Package Management?

Python package management controls:

```text
dependencies
versions
installation
builds
distribution
publishing
```

---

## 3. Common Tools

The Python ecosystem includes:

```text
pip
venv
virtualenv
pip-tools
Poetry
uv
build
twine
```

Organizations should standardize the tools used by each project.

---

## 4. Package Repository

Python packages are retrieved from package indexes.

Concept:

```text
pip
 |
v
Package Index
 |
v
Package
```

Enterprise:

```text
CI
 |
v
Corporate PyPI Repository
 |
+--> Internal Packages
+--> Approved Remote Cache
```

---

# PART II — VIRTUAL ENVIRONMENTS

## 5. Why Virtual Environments?

Avoid installing every project dependency globally.

Instead:

```text
Project A
 |
v
venv A

Project B
 |
v
venv B
```

---

## 6. Create venv

```bash
python -m venv .venv
```

---

## 7. Activate

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

---

## 8. Production Principle

CI should use isolated environments.

```text
Job
 |
v
Clean Python environment
 |
v
Install dependencies
 |
v
Build/test
```

---

# PART III — PYTHON VERSION

## 9. Version Control

Standardize:

```text
Python
pip
build backend
package manager
OS
```

---

## 10. Check Version

```bash
python --version
python -m pip --version
```

Prefer invoking pip through the intended Python interpreter:

```bash
python -m pip
```

This reduces ambiguity when multiple Python installations exist.

---

# PART IV — REQUIREMENTS FILE

## 11. requirements.txt

Traditional Python applications may define:

```text
requests==2.32.0
PyYAML==6.0.2
```

---

## 12. Install

```bash
python -m pip install -r requirements.txt
```

---

## 13. Pinning

Exact pins can improve reproducibility:

```text
requests==2.32.0
```

But transitive dependencies also need consideration.

---

# PART V — CONSTRAINTS

## 14. constraints.txt

Constraints can restrict versions without necessarily being the primary
dependency declaration.

Example:

```text
urllib3==2.2.2
```

Install conceptually:

```bash
python -m pip install -r requirements.txt \
    -c constraints.txt
```

---

## 15. Why Constraints?

Useful for:

```text
organization-wide compatibility
security pinning
transitive dependency control
temporary version restrictions
```

---

# PART VI — PYPROJECT.TOML

## 16. Modern Python Packaging

Modern projects commonly use:

```text
pyproject.toml
```

It can define:

```text
project metadata
dependencies
build system
optional dependencies
tool configuration
```

---

## 17. Example

```toml
[build-system]
requires = ["setuptools>=69"]
build-backend = "setuptools.build_meta"

[project]
name = "payment-client"
version = "1.4.0"
dependencies = [
    "requests>=2.31,<3"
]
```

The exact build backend and versions depend on the project.

---

# PART VII — BUILD SYSTEM

## 18. Build Backend

Common backends include:

```text
setuptools
hatchling
poetry-core
flit
```

A build frontend such as `python -m build` can invoke the configured
backend.

---

## 19. Why Build Systems Matter

The build system converts project source into distributable artifacts.

```text
Source
 |
v
Build Backend
 |
+--> wheel
+--> source distribution
```

---

# PART VIII — WHEEL

## 20. Wheel

A wheel is a built Python distribution format.

Typical:

```text
package_name-1.4.0-py3-none-any.whl
```

---

## 21. Benefits

```text
fast installation
prebuilt files
predictable distribution
```

---

# PART IX — SOURCE DISTRIBUTION

## 22. sdist

A source distribution commonly ends in:

```text
.tar.gz
```

Example:

```text
payment_client-1.4.0.tar.gz
```

---

## 23. Wheel vs sdist

```text
wheel
 |
built distribution

sdist
 |
source distribution
```

Prefer wheels where appropriate to avoid unnecessary compilation during
deployment.

---

# PART X — BUILD PACKAGE

## 24. Build

A common modern command is:

```bash
python -m build
```

It may produce:

```text
dist/
├── package-1.4.0.tar.gz
└── package-1.4.0-py3-none-any.whl
```

---

# PART XI — PACKAGE METADATA

## 25. Metadata

A Python distribution can contain:

```text
name
version
dependencies
license
authors
project URLs
Python requirements
```

---

# PART XII — PIP

## 26. pip

pip is commonly used to install packages.

Examples:

```bash
python -m pip install requests
python -m pip uninstall requests
python -m pip list
```

---

## 27. pip freeze

```bash
python -m pip freeze
```

This can produce installed package information, but it should not
automatically be treated as the ideal application dependency-management
strategy.

---

# PART XIII — DEPENDENCY RESOLUTION

## 28. Direct Dependency

```text
application
 |
v
requests
```

---

## 29. Transitive Dependency

```text
application
 |
v
requests
 |
v
urllib3
```

---

## 30. Dependency Graph

```text
Application
 |
+--> requests
|      |
|      +--> urllib3
|
+--> boto3
       |
       +--> botocore
```

The resolver must find a compatible set of versions.

---

# PART XIV — DEPENDENCY CONFLICT

## 31. Conflict

Example:

```text
Package A -> urllib3 <2
Package B -> urllib3 >=2
```

This can make resolution impossible depending on all constraints.

---

## 32. Resolution Strategy

Do not solve conflicts by randomly removing packages.

Investigate:

```text
dependency graph
supported versions
package compatibility
```

---

# PART XV — LOCKING

## 33. Reproducibility

Applications benefit from a reproducible dependency set.

Approaches vary by tool:

```text
lock files
compiled requirements
constraints
exact pins
```

---

## 34. Tool-Specific Locking

Examples:

```text
Poetry -> poetry.lock
uv -> uv.lock
pip-tools -> generated requirements
```

Use the mechanism adopted by the project.

---

# PART XVI — PIP-TOOLS

## 35. pip-tools

A common workflow is:

```text
requirements.in
       |
       v
pip-compile
       |
       v
requirements.txt
       |
       v
pip-sync
```

---

## 36. Why?

It separates:

```text
declared top-level dependencies
```

from:

```text
fully resolved dependency set
```

---

# PART XVII — POETRY

## 37. Poetry

Poetry can manage:

```text
dependencies
lockfile
virtual environments
build
publishing
```

---

## 38. Common Files

```text
pyproject.toml
poetry.lock
```

---

# PART XVIII — UV

## 39. uv

uv is a modern Python package and environment management tool.

It can support:

```text
dependency installation
virtual environments
locking
tool management
project workflows
```

---

## 40. Standardization

Do not mix:

```text
pip
Poetry
uv
pip-tools
```

without a clear project strategy.

---

# PART XIX — PYTHON INDEX

## 41. Public Index

The Python ecosystem commonly uses PyPI as a public package index.

Enterprise environments may route access through an internal repository
manager.

---

# PART XX — PRIVATE PYPI

## 42. Enterprise Architecture

```text
Python Application
 |
v
pip
 |
v
Corporate PyPI Virtual Repository
 |
+--> Internal Packages
 |
+--> Approved Remote Cache
```

---

## 43. Benefits

```text
centralized control
caching
availability
security
audit
```

---

# PART XXI — ARTIFACTORY PYPI

## 44. Repository Types

An enterprise repository manager can provide:

```text
PyPI local
PyPI remote
PyPI virtual
```

---

## 45. Virtual Repository

```text
pip
 |
v
virtual-pypi
 |
+--> internal
+--> remote cache
```

---

# PART XXII — PIP CONFIGURATION

## 46. Index URL

Concept:

```bash
python -m pip install \
  --index-url https://repo.company.example/pypi/simple \
  package
```

Use the actual enterprise endpoint.

---

## 47. Extra Index

Be cautious with:

```bash
--extra-index-url
```

Using multiple indexes without understanding resolution behavior can
increase supply-chain risk.

Prefer a controlled repository architecture.

---

# PART XXIII — AUTHENTICATION

## 48. Private Registry

Private package installation requires authentication.

Concept:

```text
CI
 |
v
Identity
 |
v
Private PyPI
```

---

## 49. Credentials

Use:

```text
CI secrets
token
service identity
OIDC where supported
```

Do not commit passwords or tokens.

---

# PART XXIV — .PYPIRC

## 50. Publishing Configuration

Python publishing tools can use configuration such as:

```text
~/.pypirc
```

Protect credentials stored there.

For CI, prefer securely injected credentials.

---

# PART XXV — TWINE

## 51. Publishing

A common publishing tool is:

```text
twine
```

Example:

```bash
python -m twine upload dist/*
```

The target repository must be configured correctly.

---

# PART XXVI — PUBLISH FLOW

## 52. Package Publishing

```text
Git
 |
v
CI
 |
v
Test
 |
v
Build
 |
v
Security
 |
v
Wheel/sdist
 |
v
PyPI Repository
```

---

# PART XXVII — IMMUTABILITY

## 53. Published Version

Do not build production processes around silently replacing the same
package version.

Use a new version when the artifact contents need to change.

---

# PART XXVIII — BUILD ONCE

## 54. Build Once

```text
Source
 |
v
CI
 |
v
Validated Package
 |
v
Repository
 |
+--> DEV
+--> STAGE
+--> PROD
```

---

# PART XXIX — TESTING

## 55. Unit Tests

Common frameworks:

```text
pytest
unittest
```

---

## 56. pytest

Example:

```bash
pytest
```

CI:

```bash
pytest -q
```

Use project-specific options.

---

# PART XXX — LINTING

## 57. Lint

Common tools include:

```text
ruff
flake8
pylint
```

---

## 58. Formatting

Common tools:

```text
black
ruff format
```

Use one standardized project strategy.

---

# PART XXXI — TYPE CHECKING

## 59. Static Type Checking

Common:

```text
mypy
pyright
```

Example:

```bash
mypy .
```

---

# PART XXXII — QUALITY PIPELINE

## 60. Recommended

```text
Install
 |
v
Lint
 |
v
Type Check
 |
v
Unit Test
 |
v
Integration Test
 |
v
Security
 |
v
Build
 |
v
Publish
```

---

# PART XXXIII — SECURITY

## 61. Dependency Scanning

Scan:

```text
direct dependencies
transitive dependencies
```

---

## 62. pip-audit

A project may use tools such as:

```bash
pip-audit
```

Integrate results with the organization's security policy.

---

## 63. Secrets

Never store:

```text
API keys
cloud credentials
registry tokens
private keys
```

in package source or committed configuration.

---

# PART XXXIV — SUPPLY-CHAIN SECURITY

## 64. Risks

```text
malicious package
typosquatting
dependency confusion
compromised maintainer
malicious build script
credential theft
```

---

## 65. Controls

```text
private registry
approved upstreams
dependency scanning
lock/constraint management
package provenance
SBOM
least privilege
```

---

# PART XXXV — INSTALL-TIME CODE

## 66. Build/Install Scripts

Some Python packages can execute build-related code during installation
or building.

Treat third-party package installation as potentially executing
untrusted code.

---

## 67. CI Isolation

Use:

```text
ephemeral runner
minimal credentials
network controls
isolated environment
```

---

# PART XXXVI — PACKAGE CONTENT

## 68. Inspect Artifacts

Before publishing, inspect:

```bash
tar -tzf dist/*.tar.gz
```

and wheel contents using appropriate tooling.

---

## 69. Do Not Publish Secrets

Check for:

```text
.env
credentials
private keys
CI files
source secrets
internal configuration
```

---

# PART XXXVII — DOCKER

## 70. Python Container

Basic model:

```text
Docker Build
 |
v
Python
 |
v
install dependencies
 |
v
application
```

---

# PART XXXVIII — MULTI-STAGE DOCKER

## 71. Builder

```text
Builder
 |
+--> create venv
+--> install/build
+--> test
 |
v
artifact
```

---

## 72. Runtime

```text
Minimal Runtime
 |
v
application
 |
v
production
```

---

# PART XXXIX — CONTAINER DEPENDENCIES

## 73. Production Install

If using a wheel or requirements set:

```text
build dependencies
```

can be separated from:

```text
runtime dependencies
```

---

# PART XL — NON-ROOT

## 74. Runtime Security

Prefer:

```text
non-root user
minimal image
read-only filesystem where practical
limited capabilities
```

---

# PART XLI — PYTHON CACHE

## 75. pip Cache

pip can cache downloaded packages.

The cache can improve speed but should not become the source of truth.

---

## 76. CI Cache

Cache keys should account for:

```text
OS
Python version
lock/requirements hash
package manager
```

---

# PART XLII — NATIVE EXTENSIONS

## 77. Native Packages

Some packages require compilation:

```text
C/C++
Rust
system libraries
compiler toolchain
```

Examples can include packages using native extensions.

---

## 78. Production Problem

A package may install locally but fail in CI because:

```text
compiler missing
system library missing
Python ABI differs
platform differs
```

Standardize build images.

---

# PART XLIII — PYTHON ABI

## 79. Compatibility

Binary wheels may depend on:

```text
Python version
OS
architecture
ABI
```

This is why a wheel working on one platform may not work on another.

---

# PART XLIV — CI WITH JENKINS

## 80. Flow

```text
Jenkins
 |
v
Python
 |
v
venv
 |
v
pip install / lock sync
 |
v
lint
 |
v
test
 |
v
security
 |
v
build
 |
v
publish
```

---

# PART XLV — JENKINS CREDENTIALS

## 81. Private PyPI

Use Jenkins-managed secrets for:

```text
repository token
service identity
```

Do not place them in:

```text
Jenkinsfile
requirements.txt
pyproject.toml
```

---

# PART XLVI — GITHUB ACTIONS

## 82. Example

```yaml
name: Python CI

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: pip

      - run: python -m pip install --upgrade pip
      - run: python -m pip install -r requirements.txt
      - run: pytest
```

Adapt commands to the project's dependency-management strategy.

---

# PART XLVII — GITHUB ACTION SECURITY

## 83. Permissions

Use minimal workflow permissions.

Concept:

```yaml
permissions:
  contents: read
```

---

## 84. OIDC

Where supported:

```text
GitHub Actions
 |
v
OIDC
 |
v
Identity Provider
 |
v
Short-lived credential
 |
v
Registry
```

---

# PART XLVIII — MATRIX TESTING

## 85. Python Matrix

Example:

```yaml
strategy:
  matrix:
    python-version: ['3.11', '3.12', '3.13']
```

Use only versions actually supported by the project.

---

## 86. Compatibility

Useful for:

```text
libraries
SDKs
platform tooling
multi-version support
```

---

# PART XLIX — MONOREPOS

## 87. Python Monorepo

```text
repo/
├── pyproject.toml
├── packages/
│   ├── api/
│   ├── auth/
│   └── cli/
```

---

## 88. Challenges

```text
dependency graph
build ordering
shared tooling
CI duration
package publishing
```

---

# PART L — WORKSPACES / PROJECT GROUPS

## 89. Multi-Package Repository

Tool-specific project/workspace mechanisms can manage multiple Python
packages.

Standardize the chosen mechanism rather than mixing incompatible
approaches.

---

# PART LI — VERSIONING

## 90. Package Version

Example:

```text
1.4.0
```

---

## 91. Git Tag

Example:

```text
v1.4.0
```

Track:

```text
Git commit
package version
CI run
artifact checksum
```

---

# PART LII — RELEASE MANAGEMENT

## 92. Release Flow

```text
Tag
 |
v
CI
 |
v
Test
 |
v
Security
 |
v
Build
 |
v
Publish
```

---

# PART LIII — PROVENANCE

## 93. Traceability

Record:

```text
source commit
Python version
package manager
lock/requirements state
CI workflow
build backend
artifact checksum
```

---

# PART LIV — SBOM

## 94. SBOM

Generate an SBOM when required.

It can describe:

```text
package
version
dependency
license
```

---

# PART LV — ARTIFACT PROMOTION

## 95. Promotion

```text
Validated Package
 |
v
Repository
 |
+--> DEV
+--> STAGE
+--> PROD
```

Do not rebuild for each environment.

---

# PART LVI — ROLLBACK

## 96. Rollback

Use a known-good immutable package or container image.

```text
1.5.0
 |
X
1.4.2
 |
v
deploy
```

---

# PART LVII — REPRODUCIBILITY

## 97. Reproducible Inputs

Control:

```text
Python
package manager
lock/constraints
build backend
dependencies
source
OS
container
registry
```

---

# PART LVIII — BUILD ISOLATION

## 98. Clean Environment

A production build should not depend on:

```text
developer global packages
old virtual environments
unknown local files
```

---

# PART LIX — OFFLINE BUILD

## 99. Offline Strategy

Organizations with strict network controls may pre-cache approved
packages in their internal repository.

The goal is:

```text
CI
 |
v
internal repository
 |
v
approved packages
```

rather than unrestricted internet access.

---

# PART LX — AIR-GAPPED ENVIRONMENT

## 100. Air-Gapped

A controlled environment may require:

```text
pre-approved package mirror
offline wheelhouse
internal repository
```

---

## 101. Wheelhouse

Concept:

```text
Approved packages
 |
v
wheelhouse/
 |
v
offline installation
```

---

# PART LXI — PRODUCTION PYTHON BUILD

## 102. Reference

```text
Git
 |
v
CI
 |
v
Ephemeral Runner
 |
v
Python
 |
v
Virtual Environment
 |
v
Lock / Constraints
 |
v
Private PyPI
 |
v
Install
 |
v
Lint
 |
v
Type Check
 |
v
Tests
 |
v
Security
 |
v
Build Wheel
 |
v
Publish
 |
v
Container
 |
v
Registry
 |
v
Deployment
```

---

# PART LXII — REPOSITORY SECURITY

## 103. Private Registry Controls

```text
HTTPS
RBAC
approved upstreams
read/write separation
token rotation
audit logs
artifact immutability
```

---

# PART LXIII — PRODUCTION OBSERVABILITY

## 104. Build Metrics

Track:

```text
dependency installation time
cache restore
test time
build time
publish time
workflow duration
failure rate
```

---

# PART LXIV — PERFORMANCE

## 105. Slow pip Install

Investigate:

```text
registry latency
cache
dependency count
large wheels
native compilation
network
runner CPU
```

---

## 106. Do Not Blindly Add CPU

Measure the actual bottleneck.

---

# PART LXV — TROUBLESHOOTING

## 107. Package Cannot Be Installed

Check:

```text
package name
version
Python version
registry
credentials
network
platform
wheel availability
```

---

## 108. No Matching Distribution

Possible causes:

```text
Python version unsupported
OS unsupported
architecture unsupported
version constraint
private package unavailable
```

---

## 109. Build Dependency Failure

Check:

```text
compiler
Python headers
system libraries
Rust/C toolchain
base image
```

---

# PART LXVI — AUTHENTICATION FAILURES

## 110. 401

Check:

```text
token
credential injection
registry URL
```

---

## 111. 403

Check:

```text
authorization
package permission
publish permission
token scope
```

---

## 112. 404

Check:

```text
package name
version
index URL
repository routing
```

---

# PART LXVII — HASH / INTEGRITY

## 113. Integrity Failure

Possible causes:

```text
corrupt cache
unexpected artifact
repository issue
incorrect package metadata
```

Investigate rather than bypassing verification.

---

# PART LXVIII — PYPROJECT FAILURE

## 114. Build Failure

Check:

```text
build-system
backend
backend version
project metadata
Python version
dependency
```

---

# PART LXIX — CI VS LOCAL

## 115. Local Works, CI Fails

Compare:

```text
Python
pip
OS
architecture
environment variables
registry
lock/constraints
system libraries
```

Test from a clean environment.

---

# PART LXX — SECURITY INCIDENT

## 116. Registry Token Leak

Response:

```text
revoke
 |
v
rotate
 |
v
audit
 |
v
inspect package publishing
 |
v
review CI
```

---

## 117. Malicious Dependency

Response:

```text
identify package/version
 |
v
block or quarantine
 |
v
identify consumers
 |
v
replace dependency
 |
v
rebuild
 |
v
redeploy
```

Follow organizational incident-response procedures.

---

# PART LXXI — PRODUCTION CHECKLIST

## 118. Python

```text
[ ] supported Python version
[ ] controlled pip/package manager
[ ] isolated environment
[ ] dependency policy
[ ] lock/constraints strategy
```

## 119. Repository

```text
[ ] private PyPI
[ ] virtual repository
[ ] approved upstreams
[ ] read/write separation
[ ] immutable releases
```

## 120. CI

```text
[ ] clean environment
[ ] install
[ ] lint
[ ] type check
[ ] tests
[ ] security
[ ] build
[ ] publish
```

## 121. Security

```text
[ ] dependency scanning
[ ] secret protection
[ ] runner isolation
[ ] SBOM
[ ] provenance
```

## 122. Container

```text
[ ] minimal runtime
[ ] non-root
[ ] no secrets
[ ] no unnecessary build tools
[ ] reproducible dependency installation
```

---

# PART LXXII — INTERVIEW PREPARATION

## 123. Why Use a Virtual Environment?

Answer:

```text
It isolates project dependencies from the system Python installation
and from other projects, making local development and CI more
predictable.
```

## 124. requirements.txt vs pyproject.toml?

Answer:

```text
requirements files are commonly used to describe installable
dependency sets, while pyproject.toml is the modern standardized
project configuration and packaging interface. The exact dependency
workflow depends on the selected build and package-management tools.
```

## 125. What Is a Wheel?

Answer:

```text
A wheel is a built Python distribution format that can generally be
installed without rebuilding the package from source, provided a
compatible wheel exists.
```

## 126. How Do You Manage Python Dependencies in Production?

Answer:

```text
I standardize the Python version and package-management workflow,
maintain a reproducible dependency set using the project's lock or
constraints mechanism, consume dependencies through a controlled
private registry, run security checks and publish immutable artifacts.
```

## 127. How Do You Secure PyPI Access?

Answer:

```text
I route CI through an approved private repository, restrict upstreams,
use least-privilege identities, protect credentials, rotate tokens,
scan dependencies and maintain artifact provenance.
```

## 128. How Do You Build a Python Package?

Answer:

```text
I define project metadata and the build backend in pyproject.toml,
run python -m build to produce wheel and/or source distributions,
validate the contents and publish through the approved repository.
```

## 129. Why Prefer Wheels?

Answer:

```text
A compatible wheel can avoid compilation during installation, making
deployment faster and more predictable. Native compatibility still
needs to be considered.
```

## 130. How Do You Troubleshoot pip Failures?

Answer:

```text
I check Python and pip versions, dependency constraints, registry URL,
credentials, package availability, network, platform/ABI compatibility,
wheel availability and build-tool requirements.
```

---

# PART LXXIII — SENIOR-LEVEL SCENARIOS

## 131. pip Install Suddenly Becomes Slow

Answer:

```text
I measure dependency installation time, registry latency, cache hit
behavior, dependency graph changes, wheel availability and native
compilation. I optimize the actual bottleneck instead of blindly
adding runner resources.
```

## 132. Package Works Locally but Not in CI

Answer:

```text
I compare Python, pip, OS, architecture, dependency lock/constraints,
registry configuration and system build tools. I reproduce from a clean
environment to eliminate hidden local state.
```

## 133. No Matching Distribution Found

Answer:

```text
I verify the package version, supported Python versions, platform and
architecture, configured index, private repository availability and
whether a compatible wheel or source distribution exists.
```

## 134. Production Dependency Has a Critical Vulnerability

Answer:

```text
I identify affected versions and consumers, determine whether a safe
upgrade exists, test compatibility, update the reproducible dependency
set, rebuild, scan again and redeploy. I preserve the old artifact for
audit/rollback according to policy.
```

## 135. Private Package Is Downloadable but Cannot Be Published

Answer:

```text
Read access is working, so I focus on publishing configuration:
repository target, authentication, authorization, package version
policy and publishing tool configuration.
```

## 136. Python Build Requires Native Compilation

Answer:

```text
I standardize the build image and explicitly install the required
compiler and system libraries. I prefer compatible prebuilt wheels
where available and ensure the produced artifact matches the target
runtime platform.
```

## 137. CI Has a Long-Lived Virtual Environment

Answer:

```text
I would prefer clean or ephemeral environments because persistent
virtual environments can accumulate stale packages and hide dependency
declarations. If caching is needed, I would cache package downloads
rather than relying on an uncontrolled installed environment.
```

## 138. Same Source Produces Different Wheels

Answer:

```text
I compare Python version, build backend, backend version, source state,
dependency state, build environment, generated metadata and timestamps.
I identify uncontrolled inputs and make the build environment
reproducible.
```

## 139. Rollback Is Required

Answer:

```text
I deploy the previously validated package or container image rather than
rebuilding old source. This preserves the exact artifact that was
previously tested.
```

---

# PART LXXIV — GOLDEN RULES

## 140. Rules

```text
1. Treat Python dependency management as a software supply-chain
   concern.

2. Standardize the Python version.

3. Standardize the package-management tool.

4. Use isolated environments.

5. Prefer python -m pip over ambiguous global pip commands.

6. Keep dependency declarations under source control.

7. Use pyproject.toml for modern project metadata and build
   configuration where appropriate.

8. Understand the project's selected build backend.

9. Understand requirements files.

10. Understand constraints files.

11. Use a reproducible dependency strategy.

12. Use lockfiles or compiled dependency sets when the selected tool
    supports them.

13. Keep dependency resolution deterministic enough for production.

14. Do not rely on developer global packages.

15. Do not rely on persistent CI virtual environments.

16. Use clean build environments.

17. Use approved package repositories.

18. Prefer an enterprise PyPI repository in controlled environments.

19. Prefer a virtual repository endpoint when supported.

20. Control public upstream access.

21. Cache approved external packages centrally.

22. Separate package read and publish permissions.

23. Use least-privilege identities.

24. Never commit package repository passwords.

25. Protect .pypirc credentials.

26. Prefer short-lived credentials where supported.

27. Consider OIDC/federated identity where supported.

28. Protect PR workflows from production credentials.

29. Treat package installation as potentially executable third-party
    code.

30. Isolate dependency installation.

31. Review direct dependencies.

32. Review transitive dependencies.

33. Scan dependencies.

34. Review licenses where required.

35. Generate SBOMs where required.

36. Preserve package provenance.

37. Record source commit to package version.

38. Record CI run to artifact.

39. Record Python and package-manager versions.

40. Prefer compatible wheels where appropriate.

41. Understand native extension requirements.

42. Standardize compiler and system-library environments.

43. Understand Python ABI compatibility.

44. Do not assume a wheel works on every platform.

45. Build wheels deliberately.

46. Inspect package contents before publishing.

47. Never publish secrets.

48. Keep release versions immutable.

49. Publish a new version when package contents change.

50. Build once and promote.

51. Do not rebuild separately for every environment.

52. Roll back using known-good immutable artifacts.

53. Do not rebuild old source during rollback.

54. Use pip cache for performance where appropriate.

55. Treat cache as an optimization, not the source of truth.

56. Test cold-cache installation.

57. Do not blindly cache installed virtual environments.

58. Monitor package registry latency.

59. Monitor dependency installation duration.

60. Measure before optimizing.

61. Use pytest or the project's selected test framework consistently.

62. Enforce linting.

63. Enforce type checking where applicable.

64. Enforce security gates.

65. Publish test reports.

66. Protect CI logs from secrets.

67. Use Jenkins/GitHub Actions credentials securely.

68. Use minimal GitHub workflow permissions.

69. Review third-party GitHub Actions.

70. Protect self-hosted runners.

71. Prefer ephemeral runners where practical.

72. Use matrix testing only when compatibility coverage requires it.

73. Standardize monorepo package management.

74. Understand package dependency ordering.

75. Avoid mixing Poetry, uv, pip-tools and raw pip without a clear
    strategy.

76. Use a single authoritative dependency-management workflow per
    project.

77. Keep build and runtime dependencies separate where appropriate.

78. Keep build tools out of minimal runtime containers.

79. Use multi-stage Docker builds where appropriate.

80. Run production containers as non-root where practical.

81. Do not ship build caches or secrets in runtime images.

82. Use minimal runtime images.

83. Route production dependency access through approved registries.

84. Consider offline/wheelhouse strategies for restricted environments.

85. Investigate registry failures before changing dependency versions.

86. Distinguish 401 from 403.

87. Treat 404 as package/index/routing/version evidence.

88. Investigate no-matching-distribution errors using Python, platform,
    ABI and package-version compatibility.

89. Investigate build failures using the configured build backend.

90. Compare local and CI environments when troubleshooting.

91. Preserve old artifacts according to rollback and audit requirements.

92. Associate packages with Git commits.

93. Associate packages with CI runs.

94. Associate packages with container images.

95. Associate images with deployments.

96. Maintain rollback procedures.

97. Test dependency recovery procedures.

98. Maintain repository backup/DR according to platform requirements.

99. Do not bypass integrity or security checks simply to make a build
    pass.

100. Validate exact Python, pip/package manager, build backend, registry,
     runner, CI, container and deployment behavior for the production
     architecture actually used.
```

---