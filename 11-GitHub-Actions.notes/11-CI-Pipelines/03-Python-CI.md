# Python CI

Python Continuous Integration (CI) is the process of automatically installing dependencies, validating, testing, analyzing, securing, and packaging Python application changes whenever developers push code or create a Pull Request.

The main goal is to detect problems early and ensure that only validated code moves toward deployment.

A typical Python CI pipeline looks like:

```text
Developer
    |
    ↓
Git Push / Pull Request
    |
    ↓
CI Pipeline
    |
    +-- Checkout Code
    |
    +-- Setup Python
    |
    +-- Install Dependencies
    |
    +-- Lint
    |
    +-- Unit Tests
    |
    +-- Code Quality
    |
    +-- Security Scan
    |
    +-- Build / Package
    |
    ↓
Validated Application
```

---

# What Is Python CI?

Python CI automatically validates Python source code whenever developers submit changes.

Typical activities include:

```text
Source Code Checkout
Python Setup
Dependency Installation
Linting
Formatting Checks
Type Checking
Unit Testing
Integration Testing
Code Quality Analysis
Security Scanning
Packaging
Artifact Generation
```

The CI pipeline provides fast feedback to developers.

---

# Why Python CI Is Important

Without CI, developers may manually perform:

```text
Pull Code
Install Python
Create Environment
Install Dependencies
Run Tests
Run Linter
Run Security Checks
Build Application
```

This can lead to:

```text
Human Errors
Inconsistent Environments
Dependency Problems
Late Bug Detection
Integration Problems
Broken Releases
```

CI automates these activities.

---

# Python CI Workflow

A typical workflow is:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
CI Pipeline
    |
    +-- Checkout
    +-- Python Setup
    +-- Dependency Install
    +-- Lint
    +-- Type Check
    +-- Test
    +-- SonarQube
    +-- Security Scan
    +-- Build
    |
    ↓
CI Success
```

---

# Python Version

The CI pipeline should use a supported Python version.

Examples:

```text
Python 3.10
Python 3.11
Python 3.12
Python 3.13
```

The exact version should match the application's requirements.

Check the version:

```bash
python --version
```

or:

```bash
python3 --version
```

---

# Python Package Manager

Common Python package management approaches include:

```text
pip
Poetry
Pipenv
uv
```

A project should use a consistent package-management strategy.

For a traditional pip-based project:

```text
requirements.txt
```

is commonly used.

---

# requirements.txt

A `requirements.txt` file lists Python dependencies.

Example:

```text
Flask==3.0.0
requests==2.31.0
gunicorn==21.2.0
```

Install dependencies:

```bash
pip install -r requirements.txt
```

The exact versions should match the project's tested dependency set.

---

# requirements.txt and Version Pinning

Uncontrolled versions can cause inconsistent builds.

For example:

```text
requests
```

allows dependency resolution to change over time.

A pinned version:

```text
requests==2.31.0
```

provides more predictable installation.

Modern projects may use lockfiles or dependency-management tools for stronger reproducibility.

---

# Python Virtual Environment

A virtual environment isolates project dependencies.

Create:

```bash
python -m venv .venv
```

Activate on Linux:

```bash
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

Then install dependencies:

```bash
pip install -r requirements.txt
```

---

# Why Virtual Environments Matter

Without isolation:

```text
System Python
    |
    +-- Project A Dependencies
    |
    +-- Project B Dependencies
```

This can create conflicts.

With virtual environments:

```text
Project A
    |
    ↓
.venv
    |
    +-- Dependencies

Project B
    |
    ↓
.venv
    |
    +-- Dependencies
```

Each project can maintain its own dependency environment.

---

# Python CI Environment

The CI runner should create a clean environment.

Example:

```text
CI Runner
    |
    ↓
Python
    |
    ↓
Virtual Environment
    |
    ↓
Dependencies
    |
    ↓
Build
```

This prevents accidental reliance on packages installed outside the project.

---

# pip

`pip` is a common Python package installer.

Check version:

```bash
pip --version
```

Install dependency:

```bash
pip install requests
```

Install from requirements:

```bash
pip install -r requirements.txt
```

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

---

# pip Freeze

To view installed packages:

```bash
pip freeze
```

Example:

```text
Flask==3.0.0
requests==2.31.0
gunicorn==21.2.0
```

Output can be saved:

```bash
pip freeze > requirements.txt
```

However, generated requirements files should be reviewed carefully rather than blindly committed.

---

# Python CI Pipeline Stages

A practical pipeline can contain:

```text
1. Checkout
2. Setup Python
3. Create Environment
4. Install Dependencies
5. Lint
6. Format Check
7. Type Check
8. Unit Tests
9. Code Quality
10. Security Scan
11. Build / Package
12. Publish Artifact
```

Example:

```text
Checkout
   |
   ↓
Python Setup
   |
   ↓
Create Environment
   |
   ↓
Install Dependencies
   |
   ↓
Lint
   |
   ↓
Type Check
   |
   ↓
Test
   |
   ↓
SonarQube
   |
   ↓
Security Scan
   |
   ↓
Package
```

---

# Checkout Stage

The CI system retrieves source code from Git.

```text
Git Repository
      |
      ↓
CI Runner
      |
      ↓
Python Source Code
```

The exact checkout implementation depends on the CI platform.

---

# Python Setup Stage

After checkout:

```text
Source Code
    |
    ↓
Python Setup
    |
    ↓
pip / Package Manager
```

Verify:

```bash
python --version
pip --version
```

The pipeline should use the required Python version.

---

# Dependency Installation

For a pip-based project:

```bash
pip install -r requirements.txt
```

Typical flow:

```text
requirements.txt
       |
       ↓
pip
       |
       ↓
Dependencies
       |
       ↓
Python Environment
```

---

# Dependency Installation with Virtual Environment

Example:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

CI flow:

```text
Python
  |
  ↓
Create .venv
  |
  ↓
Activate
  |
  ↓
Install Dependencies
```

---

# Python Linting

Linting checks Python source code for potential issues and style violations.

Common tools include:

```text
Ruff
Flake8
Pylint
```

Example:

```bash
ruff check .
```

or:

```bash
flake8 .
```

The tool should match the project's established configuration.

---

# Ruff

Ruff is a fast Python linter and formatter.

Example lint command:

```bash
ruff check .
```

Format check:

```bash
ruff format --check .
```

A CI pipeline can fail when mandatory lint or formatting checks fail.

---

# Flake8

Flake8 is another Python linting tool.

Example:

```bash
flake8 .
```

It can identify:

```text
Syntax Problems
Style Violations
Potential Errors
Unused Imports
```

---

# Pylint

Pylint provides static analysis for Python code.

Example:

```bash
pylint app/
```

It can detect:

```text
Errors
Warnings
Code Smells
Unused Variables
Design Problems
```

The exact configuration should be defined by the project.

---

# Python Formatting

Code formatting can be automated.

Common tools include:

```text
Black
Ruff
```

Example:

```bash
black --check .
```

This checks whether files conform to the configured formatting rules.

---

# Formatting vs Linting

Formatting:

```text
Code Style
Indentation
Line Length
Quotes
Spacing
```

Linting:

```text
Potential Bugs
Unused Imports
Code Smells
Style Problems
```

A pipeline can run both.

```text
Source
   |
   +-- Formatter Check
   |
   +-- Linter
   |
   ↓
Tests
```

---

# Python Type Checking

Python is dynamically typed, but projects can use static type checking.

Common tools:

```text
mypy
Pyright
```

Example:

```bash
mypy .
```

Type checking can detect problems before runtime.

---

# Type Hints

Example:

```python
def calculate_total(price: float, quantity: int) -> float:
    return price * quantity
```

A type checker can validate how the function is used.

CI can run:

```bash
mypy .
```

---

# Unit Testing

Python applications commonly use:

```text
pytest
unittest
```

Example:

```bash
pytest
```

Typical flow:

```text
Source Code
    |
    ↓
Test Framework
    |
    ↓
Unit Tests
    |
    ↓
Test Results
```

---

# pytest

`pytest` is a popular Python testing framework.

Example:

```python
def add(a, b):
    return a + b


def test_add():
    assert add(2, 3) == 5
```

Run:

```bash
pytest
```

---

# pytest Test Discovery

pytest can automatically discover test files such as:

```text
test_*.py
*_test.py
```

Example:

```text
tests/
├── test_users.py
├── test_orders.py
└── test_payment.py
```

Run:

```bash
pytest tests/
```

---

# pytest Verbose Output

Use:

```bash
pytest -v
```

This provides more detailed test output.

Example:

```text
test_users.py::test_create_user PASSED
test_orders.py::test_create_order PASSED
test_payment.py::test_payment FAILED
```

---

# Test Coverage

Coverage measures how much of the application code is exercised by tests.

Common tool:

```text
coverage.py
pytest-cov
```

Example:

```bash
pytest --cov=app
```

Possible metrics:

```text
Statements
Branches
Functions
Lines
```

Example:

```text
TOTAL
Coverage: 92%
```

---

# Coverage Threshold

A project can enforce a minimum coverage threshold.

Example:

```bash
pytest --cov=app --cov-fail-under=80
```

If coverage falls below the threshold:

```text
Coverage
   |
   ↓
80% Required
   |
   ↓
75% Actual
   |
   ↓
CI Failed
```

The threshold should be appropriate for the project rather than chosen arbitrarily.

---

# Integration Testing

Integration tests verify interactions between components.

Example:

```text
Python Service
     |
     +-- Database
     |
     +-- REST API
     |
     +-- Message Queue
     |
     +-- External Service
```

Unit tests usually isolate individual components.

Integration tests validate component interactions.

---

# API Testing

Python API applications can be tested for:

```text
HTTP Status
Request Validation
Response Body
Authentication
Authorization
Error Handling
Database Interaction
```

Example:

```text
POST /orders
      |
      ↓
Python API
      |
      ↓
Expected Response
```

---

# Python Build

Not every Python application has a compilation step like Java.

For a simple application:

```text
Source
   |
   ↓
Tests
   |
   ↓
Package
```

For applications that require packaging:

```text
Source
   |
   ↓
Build
   |
   ↓
Distribution Package
```

---

# Python Packaging

Python applications can be packaged into:

```text
Wheel
Source Distribution
Docker Image
```

A wheel commonly has:

```text
.whl
```

Example:

```text
myapp-1.0.0-py3-none-any.whl
```

---

# Python Build Tool

Modern Python packaging commonly uses:

```text
pyproject.toml
```

It can define project metadata and build configuration.

Example:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

The exact build backend depends on the project.

---

# pyproject.toml

`pyproject.toml` can contain configuration for:

```text
Project Metadata
Build System
Tool Configuration
Dependencies
Linting
Formatting
Testing
```

Many modern Python tools use it for their configuration.

---

# Building a Python Package

A common approach is:

```bash
python -m build
```

This can generate:

```text
dist/
├── myapp-1.0.0.tar.gz
└── myapp-1.0.0-py3-none-any.whl
```

The exact outputs depend on the project's packaging configuration.

---

# Python CI Code Quality

SonarQube can be used to analyze Python projects.

Typical flow:

```text
Python Source
    |
    ↓
Tests
    |
    ↓
SonarQube
    |
    ↓
Quality Gate
```

Possible analysis includes:

```text
Bugs
Vulnerabilities
Code Smells
Duplications
Coverage Information
```

---

# Python CI with SonarQube

Conceptually:

```text
Install Dependencies
        |
        ↓
Run Tests
        |
        ↓
Generate Coverage
        |
        ↓
SonarQube Analysis
        |
        ↓
Quality Gate
```

Coverage information can be supplied to SonarQube depending on the project's configuration.

---

# Python Security Scanning

Security checks can include:

```text
Dependency Vulnerability Scanning
Source Code Security Analysis
Secret Detection
Container Scanning
```

Possible tools:

```text
Trivy
SonarQube
Veracode
pip-audit
Bandit
```

The exact tools depend on the organization's security process.

---

# pip-audit

`pip-audit` can check Python dependencies for known vulnerabilities.

Example:

```bash
pip-audit
```

Flow:

```text
Python Dependencies
        |
        ↓
pip-audit
        |
        ↓
Vulnerability Results
```

---

# Bandit

Bandit performs static security analysis of Python code.

Example:

```bash
bandit -r .
```

It can identify certain insecure coding patterns.

Example flow:

```text
Python Source
     |
     ↓
Bandit
     |
     ↓
Security Findings
```

---

# Trivy Filesystem Scan

Trivy can scan the project filesystem.

Example:

```bash
trivy fs .
```

It can identify vulnerabilities according to the scanners and configuration being used.

---

# Trivy Container Scan

If the Python application is containerized:

```bash
trivy image myapp:v1.0.0
```

Flow:

```text
Python Application
      |
      ↓
Docker Build
      |
      ↓
myapp:v1.0.0
      |
      ↓
Trivy
      |
      ↓
Security Result
```

---

# Veracode in Python CI

Veracode can be integrated into a Python security pipeline according to the organization's implementation.

Example:

```text
Source
   |
   ↓
Build / Package
   |
   ↓
SonarQube
   |
   ↓
Trivy
   |
   ↓
Veracode
   |
   ↓
Security Gate
```

---

# Python CI Security Flow

A practical security flow can be:

```text
Checkout
   |
   ↓
Install Dependencies
   |
   ↓
Dependency Scan
   |
   ↓
Lint / Static Analysis
   |
   ↓
Unit Tests
   |
   ↓
SonarQube
   |
   ↓
Build
   |
   ↓
Docker Build
   |
   ↓
Trivy Image Scan
```

The exact order depends on the pipeline design.

---

# Python Docker Build

Typical flow:

```text
Python Source
     |
     ↓
Dependency Installation
     |
     ↓
Tests
     |
     ↓
Docker Build
     |
     ↓
Docker Image
     |
     ↓
ECR
```

Example:

```bash
docker build -t myapp:v1.0.0 .
```

---

# Python Dockerfile

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

The exact base image and startup command depend on the application.

---

# Python Docker Image Optimization

A Python Docker image can be optimized by:

```text
Using Minimal Base Images
Using Multi-Stage Builds Where Appropriate
Avoiding Unnecessary Dependencies
Using --no-cache-dir
Using .dockerignore
Removing Build-Only Files
Running as Non-Root Where Appropriate
```

---

# .dockerignore

Example:

```text
.git
.gitignore
.venv
__pycache__
*.pyc
.pytest_cache
coverage
.env
```

This prevents unnecessary files from being sent to the Docker build context.

---

# Python Multi-Stage Docker Build

For projects with build dependencies:

```dockerfile
FROM python:3.12-slim AS build

WORKDIR /app

COPY requirements.txt .

RUN pip install --prefix=/install \
    --no-cache-dir \
    -r requirements.txt


FROM python:3.12-slim

WORKDIR /app

COPY --from=build /install /usr/local

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

The exact approach depends on the dependencies and application architecture.

---

# Python CI and ECR

Typical AWS flow:

```text
Python Source
      |
      ↓
CI Pipeline
      |
      ↓
Docker Build
      |
      ↓
Trivy Scan
      |
      ↓
Amazon ECR
```

Example image:

```text
myapp:v1.0.0
```

Versioned images are preferable to relying only on `latest`.

---

# Python CI and Kubernetes

Deployment flow:

```text
Python Source
      |
      ↓
CI
      |
      ↓
Docker Build
      |
      ↓
ECR
      |
      ↓
GitOps Repository
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

---

# Python CI and GitOps

Example:

```text
Application Repository
        |
        ↓
CI
        |
        ↓
Docker Image
        |
        ↓
ECR
        |
        ↓
GitOps Repository
        |
        ↓
Image Version Update
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

The GitOps repository contains the desired Kubernetes state.

---

# Python CI and ArgoCD

Typical flow:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    ↓
Python Tests
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# Python CI with Jenkins

Example Jenkins pipeline:

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup') {
            steps {
                sh 'python3 --version'
                sh 'python3 -m venv .venv'
                sh '. .venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Lint') {
            steps {
                sh '. .venv/bin/activate && ruff check .'
            }
        }

        stage('Test') {
            steps {
                sh '. .venv/bin/activate && pytest'
            }
        }

        stage('Build') {
            steps {
                sh '. .venv/bin/activate && python -m build'
            }
        }
    }
}
```

The exact commands should match the project's package and tool configuration.

---

# Python CI with GitHub Actions

Example:

```yaml
name: Python CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint
        run: ruff check .

      - name: Test
        run: pytest

      - name: Build
        run: python -m build
```

---

# GitHub Actions Python CI with Cache

A dependency cache can be enabled through the Python setup action.

Example:

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'
    cache-dependency-path: requirements.txt
```

Then:

```yaml
- name: Install Dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```

---

# Python CI with GitLab

Example:

```yaml
stages:
  - test
  - build

test:
  image: python:3.12
  stage: test
  script:
    - pip install -r requirements.txt
    - pytest

build:
  image: python:3.12
  stage: build
  script:
    - pip install build
    - python -m build
```

Additional linting and security stages can be added.

---

# Python Environment Variables

Python applications commonly use environment variables.

Examples:

```text
APP_ENV
PORT
DATABASE_URL
API_URL
LOG_LEVEL
```

Example:

```python
import os

database_url = os.environ["DATABASE_URL"]
```

Sensitive values should be injected securely.

---

# Python Secrets

Sensitive values may include:

```text
AWS Credentials
Database Passwords
API Keys
JWT Secrets
SonarQube Tokens
Veracode Credentials
Private Registry Credentials
```

Do not commit secrets into:

```text
Source Code
.env
Dockerfile
Jenkinsfile
GitHub Actions YAML
```

Use the CI platform's secret-management capabilities.

---

# Python CI and .env Files

A local development environment may use:

```text
.env
```

Example:

```text
DATABASE_URL=...
API_KEY=...
```

The `.env` file containing real secrets should not be committed.

Use:

```text
.env.example
```

for non-secret configuration examples.

---

# Python CI Pipeline Failure

The pipeline should fail when required checks fail.

Examples:

```text
Dependency Installation Failure
Lint Failure
Type Check Failure
Unit Test Failure
Coverage Failure
Quality Gate Failure
Security Gate Failure
Build Failure
Docker Build Failure
Image Scan Failure
Artifact Publishing Failure
```

---

# Fail Fast

A practical pipeline:

```text
Checkout
   |
   ↓
Python Setup
   |
   ↓
Dependencies
   |
   ↓
Lint
   |
   ↓
Type Check
   |
   ↓
Tests
   |
   ↓
Quality
   |
   ↓
Security
   |
   ↓
Build
```

If dependency installation fails, there is no reason to continue with later stages.

---

# Python CI Success

Successful pipeline:

```text
Checkout ✓
   |
Python Setup ✓
   |
Dependencies ✓
   |
Lint ✓
   |
Type Check ✓
   |
Tests ✓
   |
SonarQube ✓
   |
Security ✓
   |
Build ✓
   |
Docker ✓
   |
Image Scan ✓
```

Result:

```text
CI PASSED
```

---

# Python Dependency Caching

Without cache:

```text
CI Run
   |
   ↓
pip install
   |
   ↓
Download Dependencies
   |
   ↓
Build
```

With cache:

```text
CI Run
   |
   ↓
Dependency Cache
   |
   +-- Hit → Reuse Packages
   |
   +-- Miss → Download
   |
   ↓
Build
```

Caching can significantly improve pipeline execution time.

---

# Cache Invalidation

A dependency cache should be tied to dependency definitions.

Conceptually:

```text
Python Version
+
requirements.txt Hash
```

or:

```text
Python Version
+
Lockfile Hash
```

When dependencies change, the cache should be refreshed.

---

# Python CI Reproducibility

A reproducible build should control:

```text
Python Version
Package Manager
Dependency Versions
Lockfile / Requirements
Build Configuration
Environment
```

Example:

```text
Python 3.12
pip
Pinned Dependencies
requirements.txt
```

Modern projects may use a dedicated lockfile mechanism for stronger reproducibility.

---

# Poetry

Poetry is a Python dependency and packaging tool.

Typical files:

```text
pyproject.toml
poetry.lock
```

Example:

```bash
poetry install
```

CI should use the project's committed lockfile for reproducible dependency resolution.

---

# Poetry CI Flow

```text
pyproject.toml
      |
      ↓
poetry.lock
      |
      ↓
poetry install
      |
      ↓
Tests
      |
      ↓
Build
```

The exact Poetry commands depend on the project version and configuration.

---

# uv

`uv` is another modern Python package and project management tool.

Example:

```bash
uv sync
```

A project may use:

```text
pyproject.toml
uv.lock
```

Again, the package manager should be selected based on the project's established workflow.

---

# Python CI Package Managers

| Tool | Common Project Files | Typical CI Command |
|---|---|---|
| pip | requirements.txt | `pip install -r requirements.txt` |
| Poetry | pyproject.toml, poetry.lock | `poetry install` |
| uv | pyproject.toml, uv.lock | `uv sync` |
| Pipenv | Pipfile, Pipfile.lock | `pipenv install` |

Use one consistent dependency-management approach per project.

---

# Python CI and Artifact Publishing

Python packages can be published to artifact repositories.

Examples:

```text
JFrog Artifactory
AWS CodeArtifact
PyPI
Private Python Repository
```

Flow:

```text
Python Source
    |
    ↓
Tests
    |
    ↓
Build
    |
    ↓
Wheel
    |
    ↓
Artifact Repository
```

---

# Python Wheel

A wheel is a built Python distribution package.

Example:

```text
myapp-1.0.0-py3-none-any.whl
```

It can be installed using:

```bash
pip install myapp-1.0.0-py3-none-any.whl
```

---

# Python Source Distribution

A source distribution commonly ends with:

```text
.tar.gz
```

Example:

```text
myapp-1.0.0.tar.gz
```

A project can publish both:

```text
Wheel
Source Distribution
```

depending on its packaging requirements.

---

# Python CI with JFrog Artifactory

Example flow:

```text
Python Source
    |
    ↓
CI
    |
    ↓
Tests
    |
    ↓
Build
    |
    ↓
Wheel
    |
    ↓
JFrog Artifactory
```

The repository credentials should be injected securely.

---

# Python CI and Docker Artifact

For containerized applications, the Docker image is commonly the deployable artifact.

Example:

```text
Python Application
       |
       ↓
Docker Build
       |
       ↓
myapp:v1.0.0
       |
       ↓
ECR
```

The image should be traceable to a specific Git commit.

---

# Python CI Artifact Versioning

Example:

```text
myapp-1.0.0.whl
myapp-1.0.1.whl
myapp-1.1.0.whl
```

Container:

```text
myapp:v1.0.0
myapp:v1.0.1
myapp:v1.1.0
```

Avoid relying only on:

```text
latest
```

for production traceability.

---

# Git Commit to Artifact Traceability

A mature pipeline should connect:

```text
Git Commit
    |
    ↓
Build
    |
    ↓
Python Package
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
Deployment
```

Example:

```text
Commit:
8f3a91d

Version:
1.2.0

Image:
myapp:v1.2.0
```

---

# Python CI and Git Tags

A Git tag can represent a release.

Example:

```text
main
 |
 +-- Commit A
 |
 +-- Commit B ← v1.0.0
 |
 +-- Commit C
 |
 +-- Commit D ← v1.1.0
```

A CI pipeline can trigger release workflows from tags.

---

# Python Semantic Versioning

Semantic Versioning follows:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v2.4.1
```

Conceptually:

```text
MAJOR → Breaking Changes
MINOR → New Backward-Compatible Features
PATCH → Bug Fixes
```

---

# Python CI Pipeline Optimization

A practical optimization process:

```text
1. Measure Pipeline Time
2. Identify Slow Stages
3. Cache Dependencies
4. Optimize Tests
5. Avoid Unnecessary Work
6. Parallelize Independent Checks
7. Optimize Docker Builds
8. Use Appropriate CI Resources
```

Measure first.

Then optimize the actual bottleneck.

---

# Slow Python CI

Possible causes:

```text
Dependency Downloads
Large Test Suite
No Cache
Slow Docker Build
Large Docker Context
Heavy Security Scans
Limited CI Resources
External Service Dependencies
```

Possible improvements:

```text
Dependency Caching
Test Parallelization
Docker Layer Caching
Smaller Build Context
Selective Tests
Appropriate Runner Resources
```

---

# Python CI Testing Strategy

A balanced testing strategy:

```text
Unit Tests
    |
    ↓
Fast Feedback
    |
    ↓
Integration Tests
    |
    ↓
Application Validation
    |
    ↓
End-to-End Tests
    |
    ↓
Critical User Flow Validation
```

Not every pipeline needs every test type at every stage.

---

# Python CI and Parallel Testing

Large test suites can sometimes be split.

Example:

```text
CI
 |
 +-- Test Group A
 |
 +-- Test Group B
 |
 +-- Test Group C
 |
 ↓
All Results
 |
 ↓
Pipeline Result
```

This can reduce total wall-clock time when the test suite and CI infrastructure support parallel execution.

---

# Python CI and Microservices

For a microservices platform:

```text
user-service
catalogue-service
cart-service
payment-service
order-service
inventory-service
```

Each service can have its own CI process.

Example:

```text
payment-service
      |
      ↓
Python CI
      |
      +-- Dependencies
      +-- Lint
      +-- Tests
      +-- Security
      +-- Build
      |
      ↓
Docker Image
      |
      ↓
ECR
```

---

# Python CI and Monorepo

A monorepo may contain:

```text
services/
├── user/
├── payment/
├── cart/
└── order/
```

The pipeline can detect which services changed.

Conceptually:

```text
Pull Request
      |
      ↓
Detect Changes
      |
      +-- payment changed
      |
      ↓
Run Payment CI
```

This can avoid unnecessary builds.

---

# Python CI and Polyrepo

A polyrepo setup may contain:

```text
user-service-repo
payment-service-repo
cart-service-repo
order-service-repo
```

Each repository can have its own pipeline.

Example:

```text
payment-service
      |
      ↓
CI
      |
      ↓
Docker
      |
      ↓
ECR
```

---

# Python CI and Kubernetes

Example:

```text
Python Application
      |
      ↓
CI
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
GitOps
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

---

# Python CI and Terraform

Python applications may be deployed using infrastructure managed by Terraform.

Example:

```text
Python CI
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
Terraform-Managed EKS
    |
    ↓
Kubernetes
```

Terraform manages infrastructure while CI/CD handles application delivery.

---

# Python CI and Observability

After deployment, observability can provide feedback.

Example:

```text
Python Application
       |
       +-- Metrics
       +-- Logs
       |
       ↓
Prometheus
       |
       ↓
Grafana
```

Logs can be processed using:

```text
ELK Stack
```

This helps identify production problems after deployment.

---

# Python CI Best Practices

```text
Use a Supported Python Version
Use Consistent Package Management
Commit Dependency Definitions
Use Virtual Environments
Pin or Lock Dependencies
Run Linting
Run Formatting Checks
Run Type Checking Where Appropriate
Run Unit Tests
Generate Coverage
Run Security Scans
Use Quality Gates
Use Security Gates
Cache Dependencies
Build Versioned Packages
Build Versioned Docker Images
Scan Container Images
Protect Secrets
Keep CI Configuration in Git
Maintain Commit-to-Deployment Traceability
```

---

# Common Python CI Mistakes

## Installing Dependencies Globally

Avoid relying on global packages:

```text
System Python
    |
    +-- Many Project Dependencies
```

Prefer an isolated environment:

```text
Project
   |
   ↓
Virtual Environment
```

---

## Ignoring Dependency Versions

Uncontrolled dependencies can create inconsistent builds.

Prefer:

```text
Pinned Versions
```

or:

```text
Lockfile
```

depending on the project's package-management approach.

---

## Skipping Tests

Do not allow unvalidated changes to move through the pipeline when tests are required.

---

## Hardcoding Secrets

Never commit:

```text
API Keys
Passwords
Cloud Credentials
Database Credentials
Tokens
```

---

## Using latest Docker Tag

Avoid relying only on:

```text
myapp:latest
```

Prefer:

```text
myapp:v1.2.0
```

or another immutable reference.

---

# Python CI Interview Questions

## Basic

1. What is Continuous Integration?
2. What is Python CI?
3. What is pip?
4. What is requirements.txt?
5. What is pyproject.toml?
6. What is a virtual environment?
7. How do you create a Python virtual environment?
8. How do you install dependencies?
9. What is pytest?
10. What is linting?
11. What is Ruff?
12. What is Flake8?
13. What is Pylint?
14. What is mypy?
15. What is `pip-audit`?

---

# Intermediate Interview Questions

16. How would you design a Python CI pipeline?

17. What stages would you include in a Python CI pipeline?

18. How do you manage Python dependencies in CI?

19. How do you cache pip dependencies?

20. How do you run pytest in CI?

21. How do you generate test coverage?

22. How do you enforce a minimum coverage threshold?

23. How do you integrate SonarQube?

24. How do you scan Python dependencies for vulnerabilities?

25. How do you integrate Trivy?

26. How do you integrate Bandit?

27. How do you build a Python Docker image?

28. How do you optimize a Python Docker image?

29. How do you handle Python version mismatches?

30. How do you troubleshoot dependency installation failures?

---

# Advanced Interview Questions

31. Design a production-grade Python CI pipeline.

32. How would you implement DevSecOps for a Python application?

33. How would you integrate SonarQube, Trivy, Bandit, and Veracode?

34. How would you implement security gates?

35. How would you implement quality gates?

36. How would you optimize a Python pipeline that takes 20 minutes?

37. How would you handle a vulnerable Python dependency?

38. How would you make Python builds reproducible?

39. How would you securely provide private package repository credentials?

40. How would you build a multi-stage Docker image for Python?

41. How would you deploy a Python application to EKS?

42. How would you connect Python CI with GitOps and ArgoCD?

43. How would you implement rollback for a Python application?

44. How would you maintain commit-to-image-to-deployment traceability?

45. How would you handle a failing Python CI pipeline?

---

# Scenario Question

## The Python CI pipeline fails during dependency installation. How would you troubleshoot it?

I would troubleshoot it systematically.

```text
Dependency Failure
       |
       ↓
Check Python Version
       |
       ↓
Check pip Version
       |
       ↓
Check requirements / lockfile
       |
       ↓
Check Package Version
       |
       ↓
Check Repository
       |
       ↓
Check Authentication
       |
       ↓
Check Network
       |
       ↓
Check Cache
```

Commands:

```bash
python --version
pip --version
pip install -r requirements.txt
```

If a private package repository is involved, I would verify its configuration and credentials.

---

# Scenario Question

## The Python application works locally but fails in CI. What would you check?

I would compare:

```text
Python Version
pip Version
Package Versions
Lockfile / Requirements
Environment Variables
Operating System
System Dependencies
Build Commands
External Services
Secrets
```

I would reproduce the CI commands locally using the same Python version and dependency definitions where possible.

---

# Scenario Question

## The Python CI pipeline takes 20 minutes. How would you optimize it?

First, I would measure each stage.

```text
Dependency Install → 5 min
Unit Tests         → 8 min
Security Scan      → 3 min
Docker Build       → 3 min
Other              → 1 min
```

Then optimize the actual bottlenecks.

Possible improvements:

```text
Dependency Caching
Test Parallelization
Selective Testing
Docker Layer Caching
Smaller Docker Context
Security Scan Optimization
Better CI Runner Resources
```

---

# Scenario Question

## A critical vulnerability is found in a Python dependency. What would you do?

I would:

```text
1. Identify the Vulnerable Package
2. Identify the Vulnerable Version
3. Find a Fixed Version
4. Check Compatibility
5. Update Dependency
6. Update Lockfile / Requirements
7. Run Unit Tests
8. Run Integration Tests
9. Run Security Scan Again
10. Merge After Required Gates Pass
```

If there is no fixed version, I would follow the organization's security exception and mitigation process.

---

# Scenario Question

## How would you implement Python CI for an EKS microservices platform?

Example:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
Python CI
    |
    +-- Python Setup
    +-- Dependencies
    +-- Ruff
    +-- Type Check
    +-- pytest
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    +-- Build
    |
    ↓
Docker Build
    |
    ↓
Image Scan
    |
    ↓
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# Scenario Question

## How would you reduce a Python Docker image size?

I would consider:

```text
Use a Minimal Base Image
Use Multi-Stage Builds
Remove Build Dependencies
Use --no-cache-dir
Use .dockerignore
Install Only Required Runtime Dependencies
Remove Unnecessary Files
Run as Non-Root Where Appropriate
```

---

# Scenario Question

## How would you secure secrets in a Python CI pipeline?

I would:

```text
Store Secrets in CI Secret Management
Inject Secrets at Runtime
Avoid Hardcoding
Do Not Commit .env Files
Restrict Access
Rotate Credentials
Mask Secrets in Logs
```

Example:

```text
CI Secret
    |
    ↓
Environment Variable
    |
    ↓
Python Application
```

---

# Scenario Question

## How would you connect Python CI to ArgoCD?

Example:

```text
Python Source
      |
      ↓
CI
      |
      ↓
Tests
      |
      ↓
Docker Build
      |
      ↓
ECR
      |
      ↓
Update GitOps Manifest
      |
      ↓
GitOps Repository
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

The GitOps repository remains the source of truth for the Kubernetes desired state.

---

# Scenario Question

## How would you implement rollback for a Python application?

Use versioned Docker images.

Example:

```text
myapp:v1.0.0
myapp:v1.1.0
myapp:v1.2.0
```

If `v1.2.0` causes a production problem:

```text
v1.2.0
   |
   ↓
Production Problem
   |
   ↓
Rollback
   |
   ↓
v1.1.0
```

The rollback should use a previously validated image.

---

# Complete Python CI Architecture

```text
                         Git Repository
                               |
                               ↓
                         Pull Request
                               |
                               ↓
                           CI Runner
                               |
                               ↓
                         Setup Python
                               |
                               ↓
                    Create / Use Environment
                               |
                               ↓
                      Install Dependencies
                               |
                               ↓
                            Ruff
                               |
                               ↓
                           Type Check
                               |
                               ↓
                           pytest
                               |
                               ↓
                          Coverage
                               |
                               ↓
                         SonarQube
                               |
                               ↓
                       Security Scanning
                               |
                               ↓
                         Python Package
                               |
                               ↓
                         Docker Build
                               |
                               ↓
                      Trivy Image Scan
                               |
                               ↓
                              ECR
                               |
                               ↓
                       GitOps Repository
                               |
                               ↓
                             ArgoCD
                               |
                               ↓
                              EKS
                               |
                               ↓
                          Production
                               |
                    +----------+----------+
                    |          |          |
                    ↓          ↓          ↓
                Prometheus  Grafana      ELK
```

---

# Python CI Best-Practice Checklist

```text
☐ Use a supported Python version
☐ Use consistent package management
☐ Use virtual environments
☐ Pin or lock dependencies
☐ Commit dependency definitions
☐ Run linting
☐ Run formatting checks
☐ Run type checking where appropriate
☐ Run unit tests
☐ Generate coverage
☐ Enforce required coverage thresholds
☐ Run dependency security scans
☐ Run SonarQube
☐ Run Trivy
☐ Run required SAST/security checks
☐ Enforce quality gates
☐ Enforce security gates
☐ Cache dependencies
☐ Build versioned packages
☐ Build versioned Docker images
☐ Scan container images
☐ Secure CI secrets
☐ Keep CI configuration in Git
☐ Maintain commit-to-deployment traceability
☐ Maintain rollback capability
```

---

# Quick Revision

Python CI:

```text
Git
 ↓
Checkout
 ↓
Python
 ↓
Virtual Environment
 ↓
Dependencies
 ↓
Lint
 ↓
Type Check
 ↓
pytest
 ↓
Coverage
 ↓
SonarQube
 ↓
Security Scan
 ↓
Build
 ↓
Docker
 ↓
Trivy
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

Important commands:

```bash
python --version
pip --version
python -m venv .venv
pip install -r requirements.txt
pip freeze
pytest
pytest -v
pytest --cov=app
ruff check .
ruff format --check .
mypy .
pip-audit
bandit -r .
python -m build
docker build -t myapp:v1.0.0 .
trivy fs .
trivy image myapp:v1.0.0
```

Important files:

```text
requirements.txt
pyproject.toml
poetry.lock
uv.lock
Dockerfile
.dockerignore
```

Important concepts:

```text
Virtual Environment
Dependency Management
Lockfiles
pip
Poetry
uv
Ruff
Flake8
Pylint
mypy
pytest
Coverage
SonarQube
pip-audit
Bandit
Trivy
Veracode
Docker
ECR
GitOps
ArgoCD
EKS
```

Core idea:

> Python CI automates dependency installation, linting, type checking, testing, code-quality analysis, security scanning, packaging, and container validation so that Python application changes are consistently validated before moving toward deployment.