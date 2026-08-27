# 17-JFrog-Artifactory
# 07-PyPI-Repositories

## 1. Purpose

This file covers Python Package Index (PyPI) repositories in JFrog Artifactory from fundamentals through production DevOps usage.

It covers:

- Python package management
- PyPI fundamentals
- Artifactory PyPI local, remote and virtual repositories
- Python package structure
- pyproject.toml
- setup.py and setup.cfg concepts
- wheels and source distributions
- pip configuration
- package indexes
- authentication
- private Python packages
- package versioning
- dependency resolution
- requirements.txt
- constraints files
- lockfile concepts
- CI/CD integration
- Jenkins
- GitHub Actions
- GitLab CI
- Docker
- Kubernetes
- security and supply-chain controls
- dependency confusion
- troubleshooting
- production architecture
- disaster recovery considerations
- interview preparation

---

# PART I — PYTHON PACKAGE FUNDAMENTALS

## 2. What Is PyPI?

PyPI is the Python Package Index, a major public package repository for Python packages.

Python applications commonly consume packages using:

```bash
pip install package-name
```

A typical application may depend on:

```text
requests
fastapi
pydantic
boto3
```

---

## 3. What Is pip?

`pip` is the commonly used Python package installer.

Typical operations include:

```bash
pip install
pip uninstall
pip list
pip show
pip freeze
```

Modern Python projects may use additional tools for dependency management and packaging, but pip remains fundamental to many production workflows.

---

## 4. PyPI and Artifactory

Artifactory can provide:

```text
private Python package storage
approved external package proxying
remote caching
dependency governance
access control
CI/CD publication
artifact lifecycle management
```

Typical architecture:

```text
Python Client
      |
      v
  pypi-virtual
    /       \
   /         \
  v           v
pypi-local  pypi-remote
               |
               v
              PyPI
```

---

## 5. Why Use Artifactory for Python?

Without a repository manager:

```text
Developer / CI
      |
      v
     PyPI
```

With Artifactory:

```text
Developer / CI
      |
      v
 Artifactory
   /      \
local     remote
            |
            v
           PyPI
```

Benefits:

```text
centralized package access
private package hosting
caching
reduced external dependency
security controls
auditability
```

---

# PART II — PYTHON PACKAGE STRUCTURE

## 6. Python Package

A Python package contains Python modules and package metadata.

Typical project:

```text
payment-client/
├── pyproject.toml
├── src/
│   └── payment_client/
│       ├── __init__.py
│       └── client.py
├── tests/
└── README.md
```

---

## 7. pyproject.toml

Modern Python packaging commonly uses:

```text
pyproject.toml
```

It can describe:

```text
project metadata
dependencies
build system
version
optional dependencies
entry points
```

Example:

```toml
[project]
name = "company-payment-client"
version = "4.2.1"
dependencies = [
    "requests>=2.0"
]
```

The exact configuration depends on the selected build backend.

---

## 8. Build Backend

Python packaging can use different build backends.

Examples include:

```text
setuptools
hatchling
poetry-core
flit
```

The backend creates distribution artifacts.

---

## 9. Python Distribution Artifacts

Common distribution formats:

```text
Wheel
Source Distribution (sdist)
```

Examples:

```text
company_payment_client-4.2.1-py3-none-any.whl
company_payment_client-4.2.1.tar.gz
```

---

## 10. Wheel

A wheel is a built Python distribution format.

Example:

```text
company_payment_client-4.2.1-py3-none-any.whl
```

Benefits:

```text
faster installation
pre-built content
platform-specific variants where needed
```

---

## 11. Source Distribution

An sdist is source-oriented distribution content.

Example:

```text
company_payment_client-4.2.1.tar.gz
```

Installing an sdist can require build tooling.

---

## 12. Why Wheels Matter in CI

If a compatible wheel is available:

```text
pip
 ↓
download wheel
 ↓
install
```

This can avoid building native components during every CI run.

---

# PART III — ARTIFACTORY PYPI REPOSITORIES

## 13. PyPI Local Repository

A local repository stores private packages owned by the organization.

Example:

```text
pypi-local
```

Publication:

```text
Python source
 ↓
Build wheel/sdist
 ↓
Upload
 ↓
pypi-local
```

---

## 14. PyPI Remote Repository

A remote repository proxies an external Python package source.

Example:

```text
pypi-remote
```

Architecture:

```text
pip
 ↓
pypi-virtual
 ↓
pypi-remote
 ↓
PyPI
```

---

## 15. PyPI Virtual Repository

A virtual repository aggregates:

```text
pypi-local
pypi-remote
```

Example:

```text
pypi-virtual
```

Consumers use:

```text
pypi-virtual
```

instead of configuring every underlying repository.

---

## 16. Recommended Architecture

```text
                    Developers / CI
                           |
                           v
                      pypi-virtual
                       /        \
                      /          \
                     v            v
                pypi-local     pypi-remote
                                  |
                                  v
                                 PyPI
```

---

## 17. Internal Package Resolution

Example:

```text
company-payment-client
```

Flow:

```text
pip
 ↓
pypi-virtual
 ↓
pypi-local
```

---

## 18. External Package Resolution

Example:

```text
requests
```

Flow:

```text
pip
 ↓
pypi-virtual
 ↓
pypi-remote
 ↓
PyPI
```

---

## 19. Why Virtual Repositories Matter

They provide:

```text
one endpoint
centralized governance
simpler client configuration
flexibility to change underlying repository topology
```

---

# PART IV — PYTHON VERSIONING

## 20. Package Version

Example:

```text
4.2.1
```

Python packaging supports standardized version schemes.

Use a consistent organizational policy.

---

## 21. Release Version

Example:

```text
4.2.1
```

Production releases should normally be immutable.

---

## 22. Pre-Release Versions

Examples can include:

```text
4.3.0a1
4.3.0b1
4.3.0rc1
```

These should be controlled according to release policy.

---

## 23. Development Versions

Python packaging also supports development version concepts.

Examples may include:

```text
4.3.0.dev1
```

Treat development versions separately from production releases.

---

## 24. Version Immutability

Once:

```text
company-payment-client 4.2.1
```

is approved for production, avoid replacing the content behind that version.

Benefits:

```text
reproducibility
rollback
auditability
incident investigation
```

---

# PART V — DEPENDENCY MANAGEMENT

## 25. requirements.txt

A common dependency file is:

```text
requirements.txt
```

Example:

```text
requests==2.32.0
fastapi==0.115.0
boto3==1.35.0
```

Pinning versions can improve reproducibility.

---

## 26. Requirements Files

Projects may use multiple files:

```text
requirements.txt
requirements-dev.txt
requirements-test.txt
```

Example:

```text
requirements.txt
→ production dependencies

requirements-dev.txt
→ development tooling
```

---

## 27. Constraints Files

A constraints file can control versions without necessarily declaring the complete dependency set.

Example:

```text
constraints.txt
```

Conceptually:

```bash
pip install -r requirements.txt -c constraints.txt
```

This can help standardize transitive dependency versions.

---

## 28. Locking Dependencies

Modern Python projects may use dedicated dependency-management tools or lock mechanisms.

The important production goal is:

```text
same input
 ↓
predictable dependency graph
```

The exact lockfile approach depends on the project's packaging toolchain.

---

## 29. Dependency Pinning

For critical production applications, uncontrolled dependency ranges can create unexpected upgrades.

Compare:

```text
requests>=2.0
```

with a controlled resolution strategy such as:

```text
requests==2.32.0
```

or an organization-approved constraints/lock strategy.

---

## 30. Transitive Dependencies

Example:

```text
Application
 ↓
Package A
 ↓
Package B
 ↓
Package C
```

Security and compatibility checks must include transitive dependencies.

---

## 31. Dependency Tree

Use appropriate Python tooling to inspect resolved dependencies.

For pip-based environments, commands and tooling can include:

```bash
pip show package-name
pip list
pip freeze
```

Additional dependency-tree tools can be used where required.

---

# PART VI — PIP CONFIGURATION

## 32. pip Index

pip can use an index URL to locate packages.

Conceptually:

```text
pip
 ↓
pypi-virtual
```

---

## 33. index-url

A common configuration concept is:

```bash
pip install --index-url https://artifactory.company.com/... package
```

The exact URL depends on the Artifactory deployment.

---

## 34. Extra Index URL

pip also supports an additional index configuration.

However, production organizations should be careful with multiple indexes because package resolution can introduce supply-chain and dependency-confusion risks.

Prefer a clearly governed repository architecture.

---

## 35. Why Multiple Uncontrolled Indexes Are Risky

Example:

```text
Internal Index
+
Public PyPI
```

If package naming and resolution are not controlled, an attacker could publish a package with an internal-looking name to a public index.

Use controlled internal namespaces and repository policy.

---

## 36. pip Configuration Files

pip configuration may exist at:

```text
global level
user level
virtual environment level
```

The exact locations vary by operating system and Python environment.

---

## 37. Environment Variables

pip configuration can also be supplied through environment variables.

This is useful in CI because:

```text
CI secret
 ↓
environment
 ↓
pip
```

Avoid exposing secrets in logs.

---

# PART VII — PYPI AUTHENTICATION

## 38. Authentication Flow

Consumer authentication:

```text
Developer / CI
      |
      v
Artifactory
```

Upstream authentication, if required:

```text
Artifactory
      |
      v
External package source
```

These are separate trust relationships.

---

## 39. CI Authentication

Use:

```text
service identity
token
secure CI secret
```

depending on the organization's Artifactory authentication design.

---

## 40. Never Commit Credentials

Do not commit:

```text
username
password
API token
access token
```

to:

```text
requirements.txt
pyproject.toml
pip configuration
source code
CI YAML
```

---

## 41. Token Rotation

Recommended:

```text
Create new token
 ↓
Update CI
 ↓
Run test
 ↓
Confirm package download/publish
 ↓
Revoke old token
```

---

## 42. Least Privilege

A dependency consumer generally needs:

```text
READ
```

A publishing pipeline needs:

```text
READ
+
DEPLOY
```

Avoid:

```text
ADMIN
DELETE
```

unless explicitly required.

---

## 43. 401 Authentication Failure

Check:

```text
token
credentials
index URL
environment variables
CI secret injection
token expiration
```

---

## 44. 403 Authorization Failure

Check:

```text
repository permissions
project permissions
package deployment rights
token scope
```

---

# PART VIII — PUBLISHING PYTHON PACKAGES

## 45. Build Package

Modern packaging can build artifacts using:

```bash
python -m build
```

This commonly creates:

```text
dist/
```

containing:

```text
.whl
.tar.gz
```

---

## 46. Inspect Build Output

Example:

```bash
ls dist/
```

Expected:

```text
company_payment_client-4.2.1-py3-none-any.whl
company_payment_client-4.2.1.tar.gz
```

---

## 47. Publish Using Twine

A common publishing tool is:

```text
twine
```

Conceptually:

```bash
twine upload --repository <configured-repository> dist/*
```

The exact repository configuration depends on the CI environment.

---

## 48. Twine Authentication

Credentials should come from:

```text
CI secrets
environment
secure configuration
```

not from source control.

---

## 49. Publication Flow

```text
Git
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Security Scan
 ↓
python -m build
 ↓
twine upload
 ↓
pypi-local
```

---

## 50. Publish Once

Prefer:

```text
Build package
 ↓
Validate
 ↓
Publish immutable artifact
 ↓
Promote
```

rather than rebuilding different artifacts for each environment.

---

# PART IX — PYPI + JENKINS

## 51. Jenkins Architecture

```text
Git
 ↓
Jenkins
 ↓
Python
 ↓
pip
 ↓
pypi-virtual
```

For publication:

```text
Jenkins
 ↓
python -m build
 ↓
twine upload
 ↓
pypi-local
```

---

## 52. Jenkins Dependency Installation

Typical:

```bash
pip install -r requirements.txt
```

with the approved Artifactory index configuration.

For CI reproducibility, use a controlled dependency resolution strategy.

---

## 53. Jenkins Package Build

Typical:

```bash
python -m build
```

---

## 54. Jenkins Package Publication

Typical:

```bash
twine upload dist/*
```

Credentials should be injected securely.

---

## 55. Jenkins Pipeline Concept

```groovy
pipeline {
    stages {
        stage('Install') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('Build Package') {
            steps {
                sh 'python -m build'
            }
        }

        stage('Publish') {
            steps {
                sh 'twine upload dist/*'
            }
        }
    }
}
```

Production pipelines should add security and release controls.

---

# PART X — PYPI + GITHUB ACTIONS

## 56. GitHub Actions Flow

```text
GitHub
 ↓
GitHub Actions
 ↓
Python Setup
 ↓
pip install
 ↓
Tests
 ↓
Build
 ↓
Scan
 ↓
Publish
```

---

## 57. GitHub Secrets

Use:

```text
GitHub Secrets
```

or approved workload identity mechanisms.

Never store Artifactory tokens in:

```text
pyproject.toml
requirements.txt
workflow YAML
```

---

## 58. GitHub Actions Package Build

Conceptually:

```yaml
- uses: actions/checkout@v4
- uses: actions/setup-python@v5
- run: python -m pip install --upgrade pip build twine
- run: pip install -r requirements.txt
- run: pytest
- run: python -m build
- run: twine upload dist/*
```

Use versions and actions approved by the organization's security policy.

---

# PART XI — PYPI + GITLAB

## 59. GitLab CI Flow

```text
GitLab
 ↓
Runner
 ↓
Python
 ↓
pip
 ↓
Tests
 ↓
Build
 ↓
Security
 ↓
Twine
 ↓
Artifactory
```

---

## 60. GitLab Variables

Use:

```text
masked variables
protected variables
environment controls
```

for repository credentials.

---

## 61. GitLab Pipeline Concept

```yaml
build:
  script:
    - pip install -r requirements.txt
    - pytest
    - python -m build

publish:
  script:
    - twine upload dist/*
```

Production pipelines should separate validation from controlled publication.

---

# PART XII — PYPI + DOCKER

## 62. Python Docker Build

Typical flow:

```text
pyproject.toml
requirements.txt
       ↓
pip install
       ↓
Application
       ↓
Docker Image
```

---

## 63. Docker Build Example

Conceptually:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Use organization-approved base images and secure build practices.

---

## 64. Artifactory Dependency During Docker Build

If:

```text
pip install
```

uses Artifactory:

```text
Docker build
 ↓
Artifactory PyPI
 ↓
dependencies
```

Therefore Docker build networking and authentication must support Artifactory access.

---

## 65. Avoid Public PyPI Directly from Production Builds

Prefer:

```text
Docker Build
 ↓
Approved Artifactory PyPI Virtual
 ↓
Approved Remote
```

rather than uncontrolled direct access to PyPI.

---

## 66. Multi-Stage Python Image

A multi-stage build can separate build tooling from the runtime image.

Conceptually:

```text
Build Image
 ↓
Compile/build dependencies
 ↓
Runtime Image
```

This can reduce runtime image size and attack surface.

---

# PART XIII — PYPI + KUBERNETES

## 67. Runtime Dependency Principle

Normally:

```text
CI
 ↓
Install dependencies
 ↓
Build image
 ↓
Scan
 ↓
Registry
 ↓
Kubernetes
```

Avoid:

```text
Pod starts
 ↓
pip install
 ↓
Internet
```

---

## 68. Kubernetes Production Flow

```text
Git
 ↓
CI
 ↓
pypi-virtual
 ↓
Build
 ↓
Docker Image
 ↓
Docker Artifactory
 ↓
Kubernetes
```

---

## 69. Why Avoid pip at Runtime?

Runtime package installation introduces:

```text
network dependency
startup latency
non-determinism
security risk
external registry dependency
```

Build dependencies into the image.

---

# PART XIV — SECURITY

## 70. Python Supply Chain

Python dependencies can introduce:

```text
vulnerabilities
malicious packages
typosquatting
dependency confusion
license issues
```

---

## 71. Dependency Scanning

Scan:

```text
direct dependencies
transitive dependencies
Python distributions
container images
```

---

## 72. pip-audit

A commonly used tool is:

```bash
pip-audit
```

It can identify known vulnerabilities according to its supported advisory sources.

Use it as one part of a broader enterprise security process.

---

## 73. Dependency Confusion

Suppose the internal package is:

```text
company-payment-client
```

An attacker could publish a similarly named package publicly.

Mitigations:

```text
internal naming policy
approved indexes
Artifactory virtual repository
restricted public access
dependency scanning
```

---

## 74. Internal Namespace Strategy

Python package names do not have the same first-class scope mechanism as NPM.

Organizations should establish naming conventions such as:

```text
company_payment_client
company_platform_auth
company_data_utils
```

and control package resolution through the approved Artifactory endpoint.

---

## 75. Multiple Index Risk

Avoid uncontrolled configurations such as:

```text
internal repository
+
random public indexes
+
developer-specific indexes
```

Every package source should be governed.

---

## 76. Package Secret Scanning

Before publishing, inspect the package for:

```text
API keys
passwords
tokens
private keys
.env files
cloud credentials
```

---

## 77. Build Artifact Inspection

Inspect:

```text
wheel
sdist
```

before production publication.

Useful checks include:

```bash
python -m build
```

followed by inspection of:

```text
dist/
```

---

## 78. Reproducibility

Production builds should control:

```text
Python version
build backend
dependency versions
repository endpoint
source commit
artifact version
```

---

# PART XV — TROUBLESHOOTING

## 79. Troubleshooting Layers

Use:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
pip configuration
 ↓
Authentication
 ↓
Authorization
 ↓
Repository
 ↓
Package
 ↓
Upstream
```

---

## 80. pip 401

Check:

```text
index URL
credentials
token
environment variables
CI secrets
```

---

## 81. pip 403

Check:

```text
repository permissions
package access
token scope
project permissions
```

---

## 82. pip 404

Check:

```text
package name
version
index URL
virtual repository
local repository
remote upstream
```

---

## 83. Could Not Find a Version

Example symptom:

```text
ERROR: Could not find a version that satisfies the requirement ...
```

Check:

```text
package exists
index URL
Python version
platform compatibility
repository permissions
virtual repository membership
```

---

## 84. SSL Certificate Error

Possible causes:

```text
missing CA certificate
expired certificate
hostname mismatch
proxy TLS interception
incorrect trust store
```

Do not solve production TLS problems by permanently disabling certificate verification.

---

## 85. pip Timeout

Check:

```text
DNS
network
proxy
Artifactory
load balancer
upstream
```

---

## 86. Package Installs Locally but Fails in CI

Possible reasons:

```text
local pip cache
local credentials
different Python version
different index URL
different requirements
```

Standardize CI.

---

## 87. Build Works but Publication Fails

Check:

```text
Twine configuration
repository URL
credentials
deploy permission
artifact version
repository policy
```

---

## 88. Publication Succeeds but Installation Fails

Check:

```text
pypi-local
pypi-virtual
virtual membership
package version
consumer permission
```

---

# PART XVI — PRODUCTION ARCHITECTURE

## 89. Standard Architecture

```text
                    Developers / CI
                           |
                           v
                      pypi-virtual
                       /        \
                      /          \
                     v            v
                pypi-local     pypi-remote
                                  |
                                  v
                                 PyPI
```

---

## 90. Production CI Flow

```text
Git
 ↓
CI
 ↓
Python Setup
 ↓
pip install
 ↓
Tests
 ↓
Security Scan
 ↓
python -m build
 ↓
Artifact Validation
 ↓
twine upload
 ↓
pypi-local
 ↓
Build Info / Provenance
 ↓
Promotion
```

---

## 91. Python + Docker Production Flow

```text
Git
 ↓
CI
 ↓
pypi-virtual
 ↓
pip install
 ↓
Build/Test
 ↓
Docker Build
 ↓
Security Scan
 ↓
Docker Repository
 ↓
Kubernetes
```

---

## 92. Enterprise Architecture

```text
                    External PyPI
                         |
                         v
                    pypi-remote
                         |
                         v
                  Security / Policy
                         |
                         v
                    pypi-virtual
                    /          \
                   /            \
            Developers          CI
                                 |
                                 v
                            pypi-local
                                 |
                                 v
                           Release Artifact
```

---

## 93. Large Organization

For many teams:

```text
pypi-virtual
     |
     +---- Team A packages
     +---- Team B packages
     +---- Platform packages
     +---- Approved external dependencies
```

Use project and permission controls to establish ownership.

---

## 94. Capacity Planning

Plan for:

```text
package count
artifact size
wheel count
sdist count
remote cache
download traffic
CI concurrency
backup
retention
```

---

## 95. Performance

Important factors:

```text
network latency
Artifactory response time
storage I/O
package size
dependency count
cache hit rate
CI concurrency
```

---

## 96. CI Burst

A large organization may run:

```text
hundreds of Python builds
```

simultaneously.

This can generate:

```text
large package-download traffic
metadata requests
artifact uploads
```

Plan capacity accordingly.

---

## 97. Monitoring

Monitor:

```text
request rate
latency
HTTP errors
storage
cache activity
upstream errors
authentication failures
```

---

## 98. Audit

Audit:

```text
package uploads
package deletions
permission changes
repository changes
authentication failures
unexpected clients
```

---

# PART XVII — PRODUCTION SCENARIOS

## 99. Scenario — PyPI Outage

If package is cached:

```text
pip
 ↓
Artifactory cache
```

Installation may continue.

If package is uncached:

```text
pip
 ↓
Artifactory
 ↓
PyPI unavailable
```

Installation may fail.

---

## 100. Scenario — Private Package 404

Check:

```text
package name
version
pypi-local
pypi-virtual
permissions
```

---

## 101. Scenario — Publish 403

Check:

```text
credentials
deploy permission
repository
project
package version
```

---

## 102. Scenario — pip Cannot Find Package

Check:

```text
pip index URL
package spelling
version
Python compatibility
virtual repository
remote repository
```

---

## 103. Scenario — CI Uses Public PyPI Directly

Inspect:

```bash
pip config list
```

and relevant environment variables.

Also inspect project configuration.

Correct the CI index strategy.

---

## 104. Scenario — Dependency Vulnerability

Flow:

```text
Identify vulnerable package
 ↓
Identify dependency path
 ↓
Find fixed version
 ↓
Update requirements/constraints
 ↓
Test
 ↓
Scan
 ↓
Build
 ↓
Publish
 ↓
Deploy
```

---

## 105. Scenario — Malicious Python Package

Response:

```text
Block/quarantine
 ↓
Identify affected applications
 ↓
Identify builds
 ↓
Identify deployments
 ↓
Replace dependency
 ↓
Rebuild
 ↓
Redeploy
 ↓
Audit
```

---

## 106. Scenario — Secret Published in Package

Immediate actions:

```text
Revoke exposed secret
 ↓
Rotate related credentials
 ↓
Identify package consumers
 ↓
Remove package from approved consumption
 ↓
Rebuild
 ↓
Redeploy
 ↓
Investigate
```

---

# PART XVIII — INTERVIEW PREPARATION

## 107. What Is a PyPI Repository in Artifactory?

Answer:

```text
It is an Artifactory repository configured for Python packages. It
can be local for private packages, remote for external PyPI content
and virtual for a unified package-consumption endpoint.
```

---

## 108. Why Use pypi-virtual?

Answer:

```text
It provides a stable endpoint for developers and CI while Artifactory
aggregates internal packages and approved external dependencies.
```

---

## 109. Where Do You Publish Private Python Packages?

Answer:

```text
I publish private Python distributions to a PyPI local repository and
let consumers access them through the approved PyPI virtual
repository.
```

---

## 110. What Is the Difference Between Wheel and sdist?

Answer:

```text
A wheel is a built Python distribution format designed for efficient
installation. An sdist is a source distribution that may require
build tooling during installation.
```

---

## 111. Why Use python -m build?

Answer:

```text
It provides a standard way to invoke the configured Python build
backend and generate distribution artifacts such as wheels and source
distributions.
```

---

## 112. Why Use Twine?

Answer:

```text
Twine is commonly used to securely upload Python distribution
artifacts to package repositories such as Artifactory.
```

---

## 113. How Do You Secure Python CI Authentication?

Answer:

```text
I use dedicated CI identities and secret-managed credentials,
configure the approved Artifactory index dynamically and avoid
committing tokens into source code or package configuration.
```

---

## 114. How Do You Troubleshoot pip 401?

Answer:

```text
I verify the index URL, token or credentials, environment variables,
CI secret injection and token validity.
```

---

## 115. How Do You Troubleshoot pip 403?

Answer:

```text
I verify authentication first and then inspect repository permissions,
project access, token scope and package publication permissions.
```

---

## 116. How Do You Troubleshoot pip 404?

Answer:

```text
I verify package name, version, index URL, virtual repository
membership and whether the package exists in the local or remote
repository.
```

---

## 117. Why Can pip Work Locally but Fail in CI?

Answer:

```text
The developer may have cached packages, local pip configuration or
credentials. CI may have a clean environment, exposing incorrect
index, authentication or dependency configuration.
```

---

## 118. How Do You Prevent Dependency Confusion?

Answer:

```text
I use controlled package naming, a centralized Artifactory virtual
repository, approved upstreams, restricted direct public-index
access, dependency scanning and reproducible dependency resolution.
```

---

## 119. Should Kubernetes Run pip install?

Answer:

```text
Normally no. I install dependencies during CI, build an immutable
container image, scan it and deploy that image. Runtime package
installation introduces unnecessary network and reproducibility risks.
```

---

## 120. How Do You Handle PyPI Outages?

Answer:

```text
I check whether required packages are already cached in Artifactory.
Cached dependencies may remain available, while uncached packages can
fail. For critical builds I use controlled remotes and reduce
unnecessary external dependency risk.
```

---

## 121. How Would You Design PyPI for 100+ Teams?

Answer:

```text
I would standardize local, remote and virtual repository patterns,
define package naming conventions, use projects/RBAC for ownership,
centralize external package access and define retention, security and
lifecycle policies.
```

---

# PART XIX — PRODUCTION CHECKLIST

## 122. Repository

```text
[ ] pypi-local
[ ] pypi-remote
[ ] pypi-virtual
[ ] naming standard
[ ] owner
[ ] package purpose
[ ] approved upstream
```

---

## 123. Python Packaging

```text
[ ] pyproject.toml
[ ] build backend
[ ] wheel
[ ] sdist
[ ] version policy
[ ] dependency strategy
[ ] constraints/lock strategy
```

---

## 124. Authentication

```text
[ ] CI identity
[ ] secure token
[ ] token rotation
[ ] no plaintext credentials
[ ] least privilege
```

---

## 125. Security

```text
[ ] approved upstreams
[ ] dependency scanning
[ ] transitive dependency scanning
[ ] secret scanning
[ ] internal naming policy
[ ] immutable production releases
```

---

## 126. CI/CD

```text
[ ] dependency installation
[ ] tests
[ ] security scan
[ ] package build
[ ] artifact inspection
[ ] publication
[ ] provenance
[ ] promotion
```

---

## 127. Operations

```text
[ ] monitoring
[ ] logging
[ ] audit
[ ] storage
[ ] cache
[ ] upstream health
[ ] backup
[ ] restore testing
[ ] DR
```

---

## 128. Reliability

```text
[ ] PyPI outage strategy
[ ] remote cache strategy
[ ] CI burst capacity
[ ] rollback versions
[ ] controlled repository changes
```

---

# PART XX — GOLDEN RULES

## 129. Rules

```text
1. Use Artifactory as the controlled Python dependency boundary.

2. Use pypi-virtual for consumer dependency resolution where
   appropriate.

3. Publish private Python distributions to pypi-local.

4. Use pypi-remote for approved external package sources.

5. Do not allow uncontrolled direct access to public indexes from CI.

6. Use a clear internal Python package naming convention.

7. Control dependency versions for reproducible production builds.

8. Use package-lock/constraints/approved dependency-management
   mechanisms appropriate to the Python toolchain.

9. Do not hardcode credentials.

10. Use dedicated CI service identities.

11. Apply least privilege.

12. Keep production package versions immutable.

13. Scan direct and transitive dependencies.

14. Inspect wheels and source distributions before release.

15. Do not publish secrets inside Python packages.

16. Revoke exposed credentials immediately.

17. Build Python dependencies into immutable container images.

18. Avoid pip install during Kubernetes runtime.

19. Monitor Artifactory, remote cache and PyPI upstream health.

20. Plan for CI package-download bursts.

21. Treat virtual repository configuration as a production interface.

22. Maintain rollback-capable package versions.

23. Back up and test restoration of required repository data.

24. Document repository owners, upstreams and retention.

25. Validate exact commands, URLs and repository behavior against the
    deployed Python, pip, Artifactory and JFrog versions before
    production rollout.
```

---

# END OF 07-PyPI-Repositories.md
