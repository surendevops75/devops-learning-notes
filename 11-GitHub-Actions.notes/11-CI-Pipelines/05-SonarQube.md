# SonarQube with GitHub Actions

SonarQube is a code quality and security analysis platform that can be integrated with GitHub Actions to automatically analyze source code during CI.

In a GitHub Actions pipeline, SonarQube can help identify:

- Bugs
- Vulnerabilities
- Security Hotspots
- Code Smells
- Code Duplication
- Test Coverage
- Maintainability Issues

Typical flow:

Developer
    |
    ↓
GitHub Repository
    |
    ↓
Pull Request / Push
    |
    ↓
GitHub Actions
    |
    +-- Checkout
    |
    +-- Setup Runtime
    |
    +-- Install Dependencies
    |
    +-- Build
    |
    +-- Test
    |
    +-- SonarQube Scan
    |
    ↓
SonarQube
    |
    ↓
Quality Analysis
    |
    ↓
Quality Gate
    |
    +-- Pass
    |
    +-- Fail

---

# Why Use SonarQube with GitHub Actions?

Without automated code-quality analysis:

Developer
    |
    ↓
Push Code
    |
    ↓
Build
    |
    ↓
Deploy

Potential code-quality problems may reach later environments.

With SonarQube:

Developer
    |
    ↓
Push / Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
SonarQube
    |
    ↓
Quality Gate
    |
    +-- Pass → Continue
    |
    +-- Fail → Stop

This provides early feedback.

---

# What Is SonarQube?

SonarQube performs static analysis of source code.

It analyzes code without requiring the application to be running.

Conceptually:

Source Code
    |
    ↓
SonarQube Scanner
    |
    ↓
SonarQube
    |
    ↓
Analysis Results

SonarQube supports many programming languages and technologies.

---

# SonarQube Server vs SonarQube Cloud

SonarQube can be used through:

- SonarQube Server
- SonarQube Cloud

The GitHub Actions integration differs mainly in how the SonarQube endpoint is configured.

For SonarQube Cloud:

    SONAR_TOKEN

is required.

For SonarQube Server:

    SONAR_TOKEN
    SONAR_HOST_URL

are commonly required.

---

# GitHub Actions SonarQube Integration

GitHub Actions uses a SonarSource-maintained action for scanning.

Example:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

The exact action version should be checked and controlled according to your organization's dependency-management policy.

---

# GitHub Actions Workflow Location

GitHub Actions workflow files must be stored under:

    .github/workflows/

Example:

    .github/
    └── workflows/
        └── sonarqube.yml

GitHub automatically discovers workflow YAML files from this directory.

---

# Basic SonarQube Workflow

A simple workflow can look like:

    name: SonarQube Analysis

    on:
      push:
        branches:
          - main

    jobs:
      sonarqube:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout Code
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

The workflow:

    Push
      |
      ↓
    Runner
      |
      ↓
    Checkout
      |
      ↓
    SonarQube Scan

---

# Why Checkout Is Required

SonarQube needs access to the source code.

Therefore:

    - uses: actions/checkout@v6

is normally used before the scan.

Flow:

GitHub Repository
       |
       ↓
actions/checkout
       |
       ↓
Runner Workspace
       |
       ↓
SonarQube Scanner

---

# Why fetch-depth: 0?

For SonarQube analysis, a full Git history can improve analysis and reporting.

Example:

    - uses: actions/checkout@v6
      with:
        fetch-depth: 0

This disables the shallow clone.

Conceptually:

Shallow Clone
    |
    ↓
Limited Git History

Full Clone
    |
    ↓
Complete Git History

For SonarQube analysis, using:

    fetch-depth: 0

is commonly recommended.

---

# SONAR_TOKEN

The scanner needs authentication.

The token should be stored as a GitHub Actions secret.

Example:

    SONAR_TOKEN

Then reference it:

    ${{ secrets.SONAR_TOKEN }}

Do not hardcode:

    SONAR_TOKEN=actual-secret-value

inside the workflow.

---

# Creating a GitHub Secret

Conceptually:

GitHub Repository
    |
    ↓
Settings
    |
    ↓
Secrets and variables
    |
    ↓
Actions
    |
    ↓
New repository secret
    |
    ↓
SONAR_TOKEN

The token is then referenced using:

    ${{ secrets.SONAR_TOKEN }}

---

# Why Use GitHub Secrets?

Secrets protect sensitive values such as:

- SonarQube Tokens
- Cloud Credentials
- Registry Credentials
- API Keys
- Deployment Credentials

Example:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

The workflow receives the secret without storing its value directly in the repository.

---

# SONAR_HOST_URL

For SonarQube Server, the scanner needs to know where the server is hosted.

Example:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}

Example variable value:

    https://sonarqube.example.com

The host URL is generally configuration rather than a secret, so it can be stored as a GitHub Actions variable.

---

# SonarQube Cloud

For SonarQube Cloud, the host URL is not normally required in the same way as a self-hosted SonarQube Server.

Example:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

Project configuration can identify the SonarQube Cloud organization and project.

---

# SonarQube Server

For a self-hosted SonarQube Server:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}

Flow:

GitHub Actions
      |
      ↓
SonarQube Scanner
      |
      ↓
SONAR_HOST_URL
      |
      ↓
SonarQube Server

---

# sonar-project.properties

SonarQube project configuration can be stored in:

    sonar-project.properties

Example:

    sonar.projectKey=my-nodejs-app
    sonar.sources=src

This file is normally placed in the repository root.

Example:

    project/
    ├── src/
    ├── tests/
    ├── package.json
    ├── sonar-project.properties
    └── .github/
        └── workflows/
            └── sonarqube.yml

---

# sonar.projectKey

The project key identifies the SonarQube project.

Example:

    sonar.projectKey=my-nodejs-app

The exact project key depends on the SonarQube project configuration.

---

# sonar.sources

Defines the source directories to analyze.

Example:

    sonar.sources=src

For multiple source directories, configure the property according to the SonarQube project requirements.

---

# sonar.tests

Test directories can be configured separately.

Example:

    sonar.tests=tests

Conceptually:

    src/
        |
        ↓
    Production Code

    tests/
        |
        ↓
    Test Code

SonarQube can then distinguish source files from test files.

---

# Basic sonar-project.properties

Example:

    sonar.projectKey=my-python-app
    sonar.sources=app
    sonar.tests=tests

The exact configuration depends on the project's structure and language.

---

# SonarQube GitHub Actions Architecture

Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +-- Checkout
    |
    +-- Setup Runtime
    |
    +-- Install Dependencies
    |
    +-- Build
    |
    +-- Test
    |
    +-- SonarQube Scan
    |
    ↓
SonarQube
    |
    ↓
Analysis
    |
    ↓
Quality Gate
    |
    ↓
GitHub Checks

---

# SonarQube in a CI Pipeline

A practical GitHub Actions pipeline can look like:

    name: CI

    on:
      push:
        branches:
          - main
      pull_request:

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'

          - name: Install Dependencies
            run: npm ci

          - name: Test
            run: npm test -- --coverage

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

---

# Recommended Pipeline Order

A common order is:

    Checkout
       |
       ↓
    Setup Runtime
       |
       ↓
    Install Dependencies
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Coverage
       |
       ↓
    SonarQube
       |
       ↓
    Quality Gate
       |
       ↓
    Docker Build

The exact order depends on the application and SonarQube configuration.

---

# SonarQube and Build

Some projects need to be built before analysis.

Example:

    Checkout
       |
       ↓
    Setup Java
       |
       ↓
    Maven Build
       |
       ↓
    Tests
       |
       ↓
    SonarQube

For compiled languages, build information may be required for accurate analysis.

---

# SonarQube and Node.js

Example GitHub Actions workflow:

    name: Node.js CI with SonarQube

    on:
      push:
        branches:
          - main
      pull_request:

    jobs:
      quality:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Test
            run: npm test -- --coverage

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

---

# SonarQube and Python

Example:

    name: Python CI with SonarQube

    on:
      push:
        branches:
          - main
      pull_request:

    jobs:
      quality:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Python
            uses: actions/setup-python@v5
            with:
              python-version: '3.12'

          - name: Install Dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt
              pip install pytest pytest-cov

          - name: Test
            run: |
              pytest --cov=. --cov-report=xml

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            with:
              args: >
                -Dsonar.python.coverage.reportPaths=coverage.xml

The exact coverage property depends on the project's SonarQube configuration.

---

# SonarQube and Java

For Maven projects, SonarSource recommends using the appropriate SonarScanner for Maven rather than treating the generic scan action as the default Maven integration.

Conceptually:

    GitHub Actions
        |
        ↓
    Checkout
        |
        ↓
    Setup Java
        |
        ↓
    Maven Build / Test
        |
        ↓
    SonarScanner for Maven
        |
        ↓
    SonarQube

The build tool-specific scanner should be selected according to the project.

---

# SonarQube and Docker

Docker image scanning and source-code analysis are different activities.

SonarQube:

    Source Code
        |
        ↓
    Code Quality / Security Analysis

Trivy:

    Docker Image
        |
        ↓
    Container Vulnerability Scan

A DevSecOps pipeline can use both.

---

# SonarQube + Trivy

Example:

    GitHub
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    SonarQube
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

SonarQube analyzes source code.

Trivy analyzes the container image and its applicable contents.

---

# SonarQube Quality Gate

A Quality Gate determines whether the analyzed project meets configured quality criteria.

Conceptually:

    SonarQube Analysis
          |
          ↓
      Quality Gate
          |
      +---+---+
      |       |
     Pass    Fail
      |       |
      ↓       ↓
   Continue   Stop

Typical quality criteria can involve:

- Bugs
- Vulnerabilities
- Code Smells
- Coverage
- Duplications
- Maintainability
- Reliability

The exact gate configuration is managed in SonarQube.

---

# Quality Gate vs SonarQube Scan

These are different concepts.

SonarQube Scan:

    Analyze Code

Quality Gate:

    Evaluate Analysis Results

Flow:

    Source Code
        |
        ↓
    SonarQube Scan
        |
        ↓
    Analysis Results
        |
        ↓
    Quality Gate
        |
        ↓
    Pass / Fail

---

# SonarQube Analysis Result

After the scanner finishes:

    Source Code
        |
        ↓
    Scanner
        |
        ↓
    SonarQube
        |
        +-- Bugs
        +-- Vulnerabilities
        +-- Code Smells
        +-- Coverage
        +-- Duplication
        +-- Security Hotspots
        |
        ↓
    Dashboard

---

# Pull Request Analysis

SonarQube can be integrated into Pull Request workflows.

Typical flow:

Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
SonarQube
    |
    ↓
Analysis
    |
    ↓
Quality Gate
    |
    ↓
PR Check

This allows quality analysis before merging code.

---

# Pull Request Workflow

Example:

    on:
      pull_request:
        types:
          - opened
          - synchronize
          - reopened

This runs the workflow when:

- A Pull Request is opened
- New commits are pushed to the PR
- A previously closed PR is reopened

---

# Push and Pull Request

A workflow can run for both:

    on:
      push:
        branches:
          - main
      pull_request:

Conceptually:

    Developer
       |
       ↓
    Pull Request
       |
       ↓
    SonarQube

and:

    Merge
       |
       ↓
    Push to main
       |
       ↓
    SonarQube

---

# SonarQube Scan Action

The official SonarSource scan action is:

    SonarSource/sonarqube-scan-action

Example:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

Using a specific version rather than an unqualified moving reference provides better workflow reproducibility.

---

# Pinning an Action Version

Example:

    uses: SonarSource/sonarqube-scan-action@v8.2.1

An organization may also pin actions to a commit SHA according to its security policy.

Conceptually:

    Tag
      |
      ↓
    v8.2.1

or:

    Commit SHA
      |
      ↓
    Immutable Reference

Action pinning helps control unexpected changes.

---

# SonarQube Action Inputs

The scan action supports inputs such as:

- projectBaseDir
- scannerVersion
- args
- scannerBinariesUrl
- skipSignatureVerification

Example:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      with:
        projectBaseDir: app

---

# projectBaseDir

If the project is located in a subdirectory:

    project/
    ├── app/
    │   ├── src/
    │   └── sonar-project.properties
    └── .github/
        └── workflows/

The scanner can be configured with:

    with:
      projectBaseDir: app

This tells the scanner where the analysis base directory is.

---

# SonarQube args

Additional analysis parameters can be passed through:

    args

Example:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      with:
        args: >
          -Dsonar.sources=src
          -Dsonar.tests=tests

The exact argument syntax should match the current action version.

---

# Important args Change

The SonarQube Scan GitHub Action changed how the `args` input is parsed in version 6.

The newer parsing is more restricted and is intended to avoid command-injection risks.

Therefore, when upgrading older workflows, review quoting and escaping in `args`.

Do not blindly copy an old workflow into a newer action version.

---

# sonar-project.properties vs args

Configuration can be placed in:

    sonar-project.properties

or passed through:

    with:
      args: >

A common approach is to keep stable project configuration in:

    sonar-project.properties

and use workflow inputs for environment-specific configuration when appropriate.

---

# SonarQube Configuration Example

Example:

    sonar-project.properties

    sonar.projectKey=my-application
    sonar.sources=src
    sonar.tests=tests

Workflow:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

This keeps the project analysis configuration inside the repository.

---

# SonarQube Coverage

SonarQube can use test coverage reports generated by the application's testing tools.

Conceptually:

    Source Code
        |
        ↓
    Tests
        |
        ↓
    Coverage Report
        |
        ↓
    SonarQube
        |
        ↓
    Coverage Result

The coverage report path must match the language and SonarQube configuration.

---

# Coverage Workflow

Example:

    Test
      |
      ↓
    Coverage XML
      |
      ↓
    SonarQube Scan
      |
      ↓
    Coverage Analysis

For Python:

    pytest --cov=. --cov-report=xml

Then provide the appropriate SonarQube property.

---

# SonarQube and GitHub Checks

SonarQube analysis can provide feedback associated with GitHub workflows and Pull Requests.

Conceptually:

    Pull Request
        |
        ↓
    GitHub Actions
        |
        ↓
    SonarQube
        |
        ↓
    Analysis
        |
        ↓
    GitHub Check / PR Feedback

This helps developers see quality information during code review.

---

# SonarQube and Branch Protection

SonarQube results can be part of a protected merge process when the relevant GitHub status/check requirements are configured.

Conceptually:

    Pull Request
        |
        ↓
    Required Checks
        |
        +-- Tests
        +-- Build
        +-- SonarQube
        |
        ↓
    Merge Allowed

If a required check fails:

    Merge
      |
      ↓
    Blocked

---

# SonarQube in Branch Protection

Example policy:

    Required Status Checks

    ✓ Build
    ✓ Test
    ✓ SonarQube

Then:

    PR
     |
     ↓
    All Required Checks
     |
     +-- Pass → Merge
     |
     +-- Fail → No Merge

The exact check name depends on the workflow and integration.

---

# GitHub Actions Permissions

A workflow can explicitly define permissions.

Example:

    permissions:
      contents: read

This follows the principle of least privilege for the workflow.

For workflows that need additional GitHub permissions, grant only what is required.

---

# SonarQube Secret Security

Good practice:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

Bad practice:

    env:
      SONAR_TOKEN: "actual-token-value"

Never commit real tokens into:

- Workflow YAML
- Source Code
- README
- Dockerfile
- Shell Scripts

---

# Secret Exposure Through Logs

Avoid:

    echo $SONAR_TOKEN

This can expose sensitive information.

Do not intentionally print:

- Tokens
- Passwords
- API Keys
- Cloud Credentials

---

# Organization-Level Secrets

For organizations with many repositories, SonarQube credentials can be managed at organization level where appropriate.

Conceptually:

    GitHub Organization
          |
          ↓
    Organization Secret
          |
          +-- Repository A
          +-- Repository B
          +-- Repository C

This can reduce duplicated secret management.

---

# Repository Variables

Non-sensitive configuration can be stored as GitHub Actions variables.

Example:

    vars.SONAR_HOST_URL

Workflow:

    env:
      SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}

Use secrets for sensitive values.

Use variables for non-sensitive configuration.

---

# Secrets vs Variables

Secrets:

    ${{ secrets.SONAR_TOKEN }}

Variables:

    ${{ vars.SONAR_HOST_URL }}

Conceptually:

    Sensitive
       |
       ↓
    Secret

    Non-Sensitive Configuration
       |
       ↓
    Variable

---

# Self-Hosted SonarQube Server

If SonarQube Server is hosted internally:

    GitHub Actions Runner
          |
          ↓
    Network Access
          |
          ↓
    SonarQube Server

The runner must be able to reach the SonarQube Server.

Possible architecture:

    GitHub
       |
       ↓
    Self-Hosted Runner
       |
       ↓
    Corporate Network
       |
       ↓
    SonarQube Server

---

# Self-Hosted Runner Considerations

When using a self-hosted runner:

- Ensure SonarQube is reachable
- Ensure required utilities are installed
- Configure proxy if required
- Configure certificates if required
- Protect the runner
- Keep runner software updated

SonarSource notes that self-hosted runners need required utilities available for the scan action, depending on the action version and runner environment.

---

# SONAR_ROOT_CERT

If SonarQube Server or a secured proxy uses a certificate that requires additional trust configuration, the scan action supports:

    SONAR_ROOT_CERT

Example:

    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
      SONAR_ROOT_CERT: ${{ secrets.SONAR_ROOT_CERT }}

The certificate should be stored securely.

---

# Proxy Configuration

Enterprise environments may use:

    HTTPS_PROXY

or:

    https_proxy

The SonarQube scanner can use supported proxy configuration when accessing required external resources.

The exact configuration depends on the runner environment.

---

# SonarQube Scanner

The scan action downloads and executes the SonarScanner as part of the analysis process.

Conceptually:

    GitHub Actions
         |
         ↓
    SonarQube Scan Action
         |
         ↓
    SonarScanner
         |
         ↓
    SonarQube Server / Cloud

---

# Scanner Version

The action can allow specifying a scanner version when required.

Example:

    - name: SonarQube Scan
      uses: SonarSource/sonarqube-scan-action@v8.2.1
      with:
        scannerVersion: 6.2.0.4584

Use a scanner version only when there is a specific compatibility or organizational requirement.

---

# SonarQube Action Version vs Scanner Version

These are different.

Action version:

    SonarSource/sonarqube-scan-action@v8.2.1

Scanner version:

    scannerVersion

The action is the GitHub Actions integration.

The scanner performs the actual code analysis.

---

# SonarQube Workflow for Microservices

For a microservices repository:

    services/
    ├── user/
    ├── catalog/
    ├── cart/
    ├── order/
    └── payment/

Possible workflow:

    Checkout
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    SonarQube
       |
       ↓
    Quality Gate

Depending on repository architecture, each service can be analyzed separately or as part of a larger project.

---

# Monorepo SonarQube

For a monorepo:

    repository/
    ├── service-a/
    ├── service-b/
    ├── service-c/
    └── .github/
        └── workflows/

The scanner can be configured with an appropriate project base directory or project structure.

The correct SonarQube organization depends on how the monorepo is managed.

---

# SonarQube with Matrix Jobs

GitHub Actions matrix jobs can run multiple application variants.

Example:

    strategy:
      matrix:
        service:
          - user
          - payment
          - order

Conceptually:

    Matrix
      |
      +-- user
      +-- payment
      +-- order

Each job can run its own build and analysis.

---

# SonarQube and Reusable Workflows

SonarQube scanning can be included in a reusable GitHub Actions workflow.

Conceptually:

    Application Repository
          |
          ↓
    Reusable CI Workflow
          |
          +-- Build
          +-- Test
          +-- SonarQube
          |
          ↓
    Result

This reduces duplicated workflow configuration across repositories.

---

# Reusable SonarQube Workflow

Conceptual structure:

    .github/
    └── workflows/
        ├── ci.yml
        └── reusable-quality.yml

Application workflow:

    jobs:
      quality:
        uses: organization/repository/.github/workflows/reusable-quality.yml@main

The exact reusable workflow syntax depends on whether the workflow is local or cross-repository.

---

# SonarQube Workflow Example

A more complete workflow:

    name: CI - Code Quality

    on:
      push:
        branches:
          - main

      pull_request:
        types:
          - opened
          - synchronize
          - reopened

    permissions:
      contents: read

    jobs:
      quality:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Run Tests
            run: npm test -- --coverage

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

---

# Complete DevSecOps GitHub Actions Flow

A production-oriented pipeline can look like:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Checkout
        |
        +-- Setup Runtime
        |
        +-- Install Dependencies
        |
        +-- Unit Tests
        |
        +-- SonarQube
        |
        +-- Security Scan
        |
        +-- Docker Build
        |
        +-- Trivy
        |
        +-- Push ECR
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# SonarQube vs Trivy

SonarQube:

    Source Code
        |
        ↓
    Code Quality
    Code Security
    Bugs
    Code Smells
    Coverage

Trivy:

    Filesystem / Dependencies / Container Image
        |
        ↓
    Vulnerability Scanning

They solve different problems and can complement each other.

---

# SonarQube vs GitHub Code Scanning

GitHub Actions can integrate with multiple security and code-analysis tools.

SonarQube provides:

- Code Quality
- Code Smells
- Bugs
- Vulnerabilities
- Coverage
- Security Hotspots

Other GitHub security tools can provide additional security analysis.

The tools can coexist depending on organizational requirements.

---

# SonarQube Quality Workflow

A useful mental model:

    Code
     |
     ↓
    Build
     |
     ↓
    Test
     |
     ↓
    SonarQube
     |
     ↓
    Quality Gate
     |
     +-- Pass
     |
     ↓
    Continue Pipeline

---

# Quality Gate Failure

If the organization's quality gate is configured as a mandatory deployment condition:

    SonarQube
       |
       ↓
    Quality Gate
       |
       +-- FAILED
             |
             ↓
         Stop Pipeline

The exact mechanism for making a workflow wait for or enforce a SonarQube quality gate depends on the integration being used.

Do not assume that simply running the scan action automatically blocks every later job based on the server-side gate.

---

# Why Quality Gates Matter

Without a quality gate:

    SonarQube
       |
       ↓
    Analysis
       |
       ↓
    Continue

With a quality gate:

    SonarQube
       |
       ↓
    Analysis
       |
       ↓
    Quality Gate
       |
       +-- Pass → Continue
       |
       +-- Fail → Block

This makes quality a measurable release condition.

---

# Common SonarQube GitHub Actions Problems

Common issues include:

- Invalid SONAR_TOKEN
- Incorrect SONAR_HOST_URL
- Project Does Not Exist
- Incorrect Project Key
- Runner Cannot Reach SonarQube Server
- Incorrect Source Directory
- Missing Coverage Report
- Unsupported Runtime
- Incorrect Action Version
- Incorrect args Syntax
- Certificate Trust Problem
- Proxy Problem
- Shallow Git Checkout

---

# Troubleshooting: Authentication Failure

Possible causes:

    SONAR_TOKEN
        |
        +-- Missing
        +-- Expired
        +-- Incorrect
        +-- Wrong Scope

Check:

    GitHub Repository
        |
        ↓
    Settings
        |
        ↓
    Secrets and variables
        |
        ↓
    Actions
        |
        ↓
    SONAR_TOKEN

Never print the token to logs.

---

# Troubleshooting: SonarQube Server Unreachable

Possible causes:

- Incorrect SONAR_HOST_URL
- DNS Problem
- Firewall
- Proxy
- Network Routing
- Server Down
- Self-hosted Runner Network Problem

Flow:

    GitHub Runner
         |
         ↓
    DNS
         |
         ↓
    Network
         |
         ↓
    SonarQube Server

Verify network connectivity from the runner environment.

---

# Troubleshooting: Project Key Problem

Possible issue:

    sonar.projectKey

does not match the configured project.

Check:

- SonarQube Project
- Project Key
- sonar-project.properties
- Workflow args

Example:

    sonar.projectKey=my-application

---

# Troubleshooting: No Source Files

Possible causes:

- Incorrect sonar.sources
- Incorrect projectBaseDir
- Repository not checked out
- Wrong working directory
- Files excluded by configuration

Example:

    sonar.sources=src

Verify:

    src/

actually exists in the checked-out workspace.

---

# Troubleshooting: Coverage Not Showing

Check:

    1. Tests ran
    2. Coverage was generated
    3. Coverage file exists
    4. Correct SonarQube property is configured
    5. Path is relative to the correct analysis base directory

Example:

    pytest --cov=. --cov-report=xml

Then configure the appropriate SonarQube coverage property.

---

# Troubleshooting: Shallow Clone

If Git history is too shallow:

    actions/checkout

may be using the default shallow checkout.

Use:

    - uses: actions/checkout@v6
      with:
        fetch-depth: 0

This provides the complete history to the scanner.

---

# Troubleshooting: Action Version Upgrade

If an older workflow suddenly fails after upgrading the action:

Check:

    Action Version
        |
        ↓
    Scanner Version
        |
        ↓
    args Syntax
        |
        ↓
    Runner Requirements

In particular, review changes to the `args` input when moving to newer versions.

---

# Troubleshooting: Self-Hosted Runner

Check that the runner has:

- curl or wget where required
- unzip
- gpg
- dirmngr where required by signature verification
- Network Access
- Correct Certificates
- Correct Proxy Configuration

GitHub-hosted runners generally provide required utilities automatically.

---

# SonarQube Security Best Practices

- Store SONAR_TOKEN in GitHub Secrets
- Never hardcode tokens
- Do not print tokens
- Use least-privilege workflow permissions
- Protect production branches
- Review Pull Requests
- Run analysis on Pull Requests
- Use quality gates
- Keep the scan action updated
- Consider pinning action versions
- Secure self-hosted runners
- Protect SonarQube Server access
- Use HTTPS
- Protect SonarQube credentials

---

# GitHub Actions Security Best Practices for SonarQube

Use:

    permissions:
      contents: read

when that is sufficient.

Avoid unnecessary permissions such as:

    contents: write

unless the workflow actually needs them.

Principle:

    Minimum Permission
           |
           ↓
    Required Functionality

---

# SonarQube Action Version Management

Do not blindly use:

    @main

for production workflows.

Prefer a controlled version such as:

    @v8.2.1

or an approved immutable commit SHA.

Organizations should define their own dependency update and approval process.

---

# SonarQube and Pull Request Protection

Recommended architecture:

    Pull Request
        |
        +-- Build
        +-- Test
        +-- SonarQube
        |
        ↓
    Required Checks
        |
        +-- Pass → Merge
        |
        +-- Fail → Fix Code

This helps prevent low-quality code from entering the protected branch.

---

# SonarQube and DevSecOps

SonarQube can represent one stage of the DevSecOps process.

Example:

    PLAN
      |
      ↓
    CODE
      |
      ↓
    BUILD
      |
      ↓
    TEST
      |
      ↓
    SONARQUBE
      |
      ↓
    SECURITY SCAN
      |
      ↓
    DOCKER
      |
      ↓
    TRIVY
      |
      ↓
    ECR
      |
      ↓
    DEPLOY

---

# SonarQube Interview Questions

## Basic

1. What is SonarQube?
2. Why do we use SonarQube?
3. What is static code analysis?
4. What is a SonarQube Quality Gate?
5. What is the difference between SonarQube Server and SonarQube Cloud?
6. How do you integrate SonarQube with GitHub Actions?
7. What is SONAR_TOKEN?
8. What is SONAR_HOST_URL?
9. Where should SONAR_TOKEN be stored?
10. What is sonar-project.properties?
11. What is sonar.projectKey?
12. What is sonar.sources?
13. Why do we use fetch-depth: 0?
14. What is the SonarQube Scan GitHub Action?
15. What is code coverage?

---

# Intermediate Interview Questions

16. How would you add SonarQube to an existing GitHub Actions pipeline?

17. How would you configure SonarQube for Pull Requests?

18. How would you store SonarQube credentials securely?

19. How would you configure SonarQube Server in GitHub Actions?

20. How would you configure SonarQube Cloud?

21. How do you pass SonarQube properties to the scanner?

22. What is projectBaseDir?

23. How do you configure coverage reporting?

24. How do you troubleshoot a SonarQube authentication failure?

25. How do you troubleshoot a SonarQube connectivity problem?

26. How do you troubleshoot a missing source directory?

27. How do you use SonarQube with Node.js?

28. How do you use SonarQube with Python?

29. How do you integrate SonarQube with a Docker-based CI pipeline?

30. How do you protect the main branch using SonarQube-related checks?

---

# Advanced Interview Questions

31. Design a production-grade GitHub Actions pipeline with SonarQube.

32. How would you implement SonarQube as a mandatory quality gate?

33. How would you integrate SonarQube and Trivy in the same pipeline?

34. How would you handle SonarQube Server inside a private network?

35. How would you configure a self-hosted GitHub Actions runner for SonarQube?

36. How would you manage SonarQube secrets across multiple repositories?

37. How would you implement SonarQube in a GitHub Actions reusable workflow?

38. How would you implement SonarQube for a monorepo?

39. How would you optimize a slow SonarQube analysis?

40. How would you troubleshoot a SonarQube workflow that works locally but fails in GitHub Actions?

41. How would you secure the SonarQube GitHub Actions integration?

42. How would you implement quality gates before Docker image publishing?

43. How would you integrate SonarQube into a DevSecOps pipeline?

44. How would you handle a failed SonarQube quality gate?

45. How would you maintain SonarQube configuration across multiple microservices?

---

# Scenario Question

## Your GitHub Actions SonarQube step fails with an authentication error. What would you check?

I would check:

    1. SONAR_TOKEN exists
    2. Secret name is correct
    3. Token is valid
    4. Token has appropriate permissions
    5. Workflow references secrets.SONAR_TOKEN
    6. Correct SonarQube project is being analyzed

I would never print the token to logs.

---

# Scenario Question

## SonarQube works locally but fails in GitHub Actions. What would you check?

I would compare:

- SonarQube URL
- Token
- Project Key
- Working Directory
- Source Directory
- Git History
- Scanner Version
- Action Version
- Java / Runtime Requirements
- Network Connectivity
- Environment Variables

Then reproduce the workflow commands on a comparable environment.

---

# Scenario Question

## SonarQube cannot find source files. How would you troubleshoot it?

I would verify:

    GitHub Workspace
        |
        ↓
    Repository Checkout
        |
        ↓
    Current Directory
        |
        ↓
    sonar.sources
        |
        ↓
    Actual Source Directory

For example:

    sonar.sources=src

requires the expected:

    src/

directory to exist relative to the configured analysis base directory.

---

# Scenario Question

## SonarQube quality gate fails. What would you do?

I would:

    1. Open SonarQube
    2. Review the failed quality condition
    3. Identify the affected code
    4. Check bugs / vulnerabilities / code smells / coverage
    5. Fix the issue
    6. Push a new commit
    7. Re-run GitHub Actions
    8. Verify the quality gate again

I would not simply disable the quality gate to make the pipeline pass unless there is an approved organizational exception.

---

# Scenario Question

## How would you implement SonarQube for a Node.js microservice?

Example:

    GitHub
       |
       ↓
    Pull Request
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       +-- Setup Node.js
       +-- npm ci
       +-- npm test
       +-- Coverage
       +-- SonarQube
       |
       ↓
    Quality Gate
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

---

# Scenario Question

## How would you implement SonarQube for a Python application?

Example:

    GitHub
       |
       ↓
    Pull Request
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       +-- Setup Python
       +-- Install Dependencies
       +-- pytest
       +-- Coverage
       +-- SonarQube
       |
       ↓
    Quality Gate
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

---

# Scenario Question

## How would you implement SonarQube in a production DevSecOps pipeline?

I would design the pipeline as:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Checkout
        +-- Build
        +-- Unit Tests
        +-- Coverage
        +-- SonarQube
        +-- Quality Gate
        +-- Security Scan
        +-- Docker Build
        +-- Trivy
        +-- Security Gate
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

The important principle is to prevent the pipeline from promoting code when mandatory quality or security checks fail.

---

# Scenario Question

## Why would you use SonarQube and Trivy together?

Because they address different layers.

SonarQube:

    Application Source
        |
        ↓
    Code Quality
    Code Security
    Code Smells
    Bugs
    Coverage

Trivy:

    Dependencies / Filesystem / Container
        |
        ↓
    Vulnerability Scanning

Together:

    Source Analysis
          +
    Dependency / Container Security
          |
          ↓
    Stronger DevSecOps Pipeline

---

# Production GitHub Actions SonarQube Example

Example:

    name: CI - SonarQube

    on:
      push:
        branches:
          - main
          - develop

      pull_request:
        types:
          - opened
          - synchronize
          - reopened

    permissions:
      contents: read

    jobs:

      quality:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Run Tests
            run: npm test -- --coverage

          - name: SonarQube Scan
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
              SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}

---

# GitHub Actions SonarQube Mental Model

Remember:

    GitHub Event
        |
        ↓
    Workflow
        |
        ↓
    Job
        |
        ↓
    Runner
        |
        ↓
    Checkout
        |
        ↓
    Build / Test
        |
        ↓
    SonarQube Action
        |
        ↓
    SonarScanner
        |
        ↓
    SonarQube
        |
        ↓
    Analysis
        |
        ↓
    Quality Gate
        |
        ↓
    GitHub Check / Pipeline Decision

---

# Important Files

GitHub Actions workflow:

    .github/workflows/sonarqube.yml

SonarQube project configuration:

    sonar-project.properties

Application source:

    src/

Test source:

    tests/

GitHub repository configuration:

    Settings
        |
        +-- Secrets
        +-- Variables
        +-- Actions

---

# Important GitHub Actions Syntax

Workflow:

    name:

Trigger:

    on:

Jobs:

    jobs:

Runner:

    runs-on:

Step:

    steps:

Action:

    uses:

Shell command:

    run:

Environment variable:

    env:

Secret:

    ${{ secrets.SONAR_TOKEN }}

Variable:

    ${{ vars.SONAR_HOST_URL }}

---

# Important SonarQube Variables

    SONAR_TOKEN

Used for authentication.

    SONAR_HOST_URL

Used for SonarQube Server URL.

    SONAR_ROOT_CERT

Used when additional certificate trust configuration is required.

---

# Important SonarQube Properties

    sonar.projectKey

Identifies the project.

    sonar.sources

Defines source directories.

    sonar.tests

Defines test directories.

Coverage properties:

    Language-specific coverage property

The exact coverage property depends on the language and coverage format.

---

# Quick Revision

GitHub Actions + SonarQube:

    Developer
       ↓
    Pull Request
       ↓
    GitHub Actions
       ↓
    Checkout
       ↓
    Build
       ↓
    Test
       ↓
    Coverage
       ↓
    SonarQube Scan
       ↓
    Quality Gate
       ↓
    Docker Build
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

Important action:

    SonarSource/sonarqube-scan-action

Important secret:

    SONAR_TOKEN

Important Server variable:

    SONAR_HOST_URL

Important configuration file:

    sonar-project.properties

Important GitHub Actions directory:

    .github/workflows/

Important concepts:

- SonarQube
- SonarQube Server
- SonarQube Cloud
- SonarScanner
- GitHub Actions
- SONAR_TOKEN
- SONAR_HOST_URL
- sonar-project.properties
- sonar.projectKey
- sonar.sources
- sonar.tests
- Code Quality
- Static Analysis
- Code Smells
- Bugs
- Vulnerabilities
- Security Hotspots
- Code Coverage
- Quality Gates
- Pull Request Analysis
- Branch Protection
- GitHub Secrets
- GitHub Variables
- Reusable Workflows
- Self-Hosted Runners
- DevSecOps
- Trivy
- Docker
- ECR
- GitOps
- ArgoCD
- EKS

Core idea:

SonarQube in GitHub Actions provides automated source-code quality and security analysis as part of CI. GitHub Actions checks out the code, builds and tests the application, runs the SonarQube scan, and can use the resulting quality information as part of the organization's merge and delivery controls. The exact enforcement mechanism for a Quality Gate should be explicitly configured rather than assumed from simply running the scanner.