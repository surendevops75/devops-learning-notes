# Artifact Publishing with GitHub Actions

Artifact publishing is the process of taking a validated build output and storing it in a repository or artifact storage system so that it can be consumed by later stages of the software delivery lifecycle.

In GitHub Actions, artifacts can be used to:

- Store build outputs
- Share files between jobs
- Preserve test reports
- Preserve logs
- Publish packages
- Publish Docker images
- Store release binaries
- Promote the same artifact across environments
- Support troubleshooting and auditing

Typical CI/CD flow:

Developer
    |
    ↓
GitHub Repository
    |
    ↓
GitHub Actions
    |
    +-- Checkout
    |
    +-- Build
    |
    +-- Test
    |
    +-- Security Scan
    |
    ↓
Artifact
    |
    +-- GitHub Actions Artifact
    +-- JFrog Artifactory
    +-- Amazon ECR
    +-- GitHub Packages
    +-- Other Artifact Repository
    |
    ↓
CD Pipeline
    |
    ↓
Deployment

---

# What Is a Build Artifact?

A build artifact is an output produced by a build process.

Examples:

Java:

    application.jar

Node.js:

    dist/

Python:

    package/

Frontend:

    build/
    dist/

Docker:

    Docker Image

Terraform:

    Plan / Configuration Artifacts

Reports:

    test-results.xml
    coverage.xml

The artifact is the result that can be consumed by another stage.

---

# Why Publish Artifacts?

Without artifact publishing:

    Source Code
        |
        ↓
    Build
        |
        ↓
    Temporary Output
        |
        ↓
    Runner Destroyed
        |
        ↓
    Output Lost

With artifact publishing:

    Source Code
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    Artifact Repository
        |
        ↓
    Deployment

This creates a reliable handoff between CI and CD.

---

# Build Once, Deploy Many

One of the most important CI/CD principles is:

    Build Once
        |
        ↓
    Test Once
        |
        ↓
    Publish Artifact
        |
        ↓
    Deploy Same Artifact
        |
        +-- QA
        +-- SIT
        +-- UAT
        +-- Production

The application should not be rebuilt separately for every environment.

---

# Why Rebuilding Is Dangerous

Bad approach:

    Build
      |
      ↓
    QA

    Build Again
      |
      ↓
    UAT

    Build Again
      |
      ↓
    Production

Each build could potentially produce a different artifact.

Better:

    Build
      |
      ↓
    Artifact
      |
      +----> QA
      |
      +----> UAT
      |
      +----> Production

The same tested artifact is promoted.

---

# GitHub Actions Artifacts

GitHub Actions provides artifact storage for files generated during workflow execution.

Common use cases:

- Build packages
- Test reports
- Coverage reports
- Logs
- Debug files
- Deployment packages

Example:

    build/
        |
        ↓
    Upload Artifact
        |
        ↓
    GitHub Actions Artifact

---

# Uploading an Artifact

GitHub provides the official artifact action:

    actions/upload-artifact

Example:

    - name: Upload Build Artifact
      uses: actions/upload-artifact@v4
      with:
        name: application
        path: build/

This uploads the contents of:

    build/

as a GitHub Actions artifact.

---

# Downloading an Artifact

The corresponding action is:

    actions/download-artifact

Example:

    - name: Download Build Artifact
      uses: actions/download-artifact@v4
      with:
        name: application

Flow:

    Job 1
      |
      ↓
    Build
      |
      ↓
    Upload Artifact
      |
      ↓
    GitHub Actions Storage
      |
      ↓
    Job 2
      |
      ↓
    Download Artifact
      |
      ↓
    Deploy

---

# Basic Artifact Workflow

Example:

    name: Build and Publish Artifact

    on:
      push:
        branches:
          - main

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build
            run: |
              mkdir -p build
              echo "Application Build" > build/application.txt

          - name: Upload Artifact
            uses: actions/upload-artifact@v4
            with:
              name: application
              path: build/

---

# Workflow Architecture

    GitHub Repository
          |
          ↓
    GitHub Actions
          |
          ↓
       Build
          |
          ↓
      build/
          |
          ↓
    upload-artifact
          |
          ↓
    GitHub Artifact
          |
          ↓
    download-artifact
          |
          ↓
      Deployment

---

# Artifact Name

Example:

    with:
      name: application

The artifact name identifies the uploaded artifact.

Examples:

    application
    backend-build
    frontend-build
    test-results
    coverage-report

Use meaningful names.

---

# Artifact Path

Example:

    with:
      path: build/

The path tells GitHub Actions which files or directories should be uploaded.

Examples:

    path: dist/

    path: target/*.jar

    path: reports/

    path: |
      build/
      reports/

---

# Multiple Paths

Multiple files or directories can be provided.

Example:

    - name: Upload Build and Reports
      uses: actions/upload-artifact@v4
      with:
        name: application-output
        path: |
          build/
          reports/

This stores both build output and reports in the same artifact.

---

# Artifact Name and Path

Remember:

    name
        |
        ↓
    Artifact Identifier

    path
        |
        ↓
    Files to Upload

Example:

    name: backend-build
    path: target/

---

# Java Artifact Publishing

A Maven project may generate:

    target/
    └── application.jar

GitHub Actions:

    - name: Build
      run: mvn clean package

    - name: Upload JAR
      uses: actions/upload-artifact@v4
      with:
        name: java-application
        path: target/*.jar

Flow:

    Maven
      |
      ↓
    target/*.jar
      |
      ↓
    GitHub Artifact

---

# Node.js Artifact Publishing

Example:

    - name: Install Dependencies
      run: npm ci

    - name: Build
      run: npm run build

    - name: Upload Build
      uses: actions/upload-artifact@v4
      with:
        name: node-application
        path: dist/

Flow:

    npm run build
        |
        ↓
    dist/
        |
        ↓
    upload-artifact

---

# Python Artifact Publishing

A Python application may produce a package.

Example:

    python -m build

Output:

    dist/
    ├── application.tar.gz
    └── application.whl

Upload:

    - name: Upload Python Package
      uses: actions/upload-artifact@v4
      with:
        name: python-package
        path: dist/

---

# Frontend Artifact

Example:

    npm ci

    npm run build

Output:

    dist/

Upload:

    - name: Upload Frontend
      uses: actions/upload-artifact@v4
      with:
        name: frontend-build
        path: dist/

The artifact can later be deployed to a web server or cloud storage.

---

# Test Reports as Artifacts

Artifacts are not limited to application binaries.

Example:

    - name: Run Tests
      run: npm test

    - name: Upload Test Reports
      uses: actions/upload-artifact@v4
      with:
        name: test-results
        path: reports/

This allows developers to inspect test results after the workflow completes.

---

# Coverage Reports

Example:

    - name: Run Tests
      run: npm test -- --coverage

    - name: Upload Coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coverage/

Coverage can then be downloaded from the workflow run.

---

# Logs as Artifacts

When troubleshooting production-like CI failures, logs can be preserved.

Example:

    - name: Upload Logs
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: application-logs
        path: logs/

Using:

    if: always()

allows the upload step to run even when an earlier step fails.

---

# Upload Artifacts on Failure

This is useful for:

- Test reports
- Logs
- Screenshots
- Debug output
- Crash information

Example:

    - name: Upload Test Results
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: test-results
        path: test-results/

---

# Artifact Retention

GitHub Actions artifacts can have a retention period.

Example:

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: application
        path: build/
        retention-days: 7

This allows organizations to control how long workflow artifacts are retained.

---

# Why Use Retention?

Artifacts consume storage.

Example:

    Every Build
        |
        ↓
    500 MB Artifact
        |
        ↓
    100 Builds
        |
        ↓
    Large Storage Usage

Retention policies help manage storage.

---

# Artifact Retention Strategy

Possible strategy:

    Pull Request
        |
        ↓
    Short Retention

    Main Branch
        |
        ↓
    Longer Retention

    Release
        |
        ↓
    Long-Term Storage

The exact retention period depends on organizational requirements.

---

# Artifact Compression

GitHub Actions artifact uploads use compression where applicable.

For already-compressed files such as:

    .zip
    .gz
    .jar

compression behavior may provide less benefit.

Large artifact uploads should be evaluated for size and performance.

---

# Hidden Files

When uploading artifacts, be aware of hidden files.

Examples:

    .env
    .config
    .npmrc
    .terraform/

Do not accidentally publish sensitive files.

Always verify the path being uploaded.

---

# Never Upload Secrets

Do not upload:

    .env
    credentials.json
    private-key.pem
    aws-credentials
    password files
    tokens

Bad:

    path: .

This may upload the entire repository workspace.

Better:

    path: build/

Upload only the files required by the next stage.

---

# Artifact Security

Before publishing:

    Check
      |
      +-- Secrets
      +-- Credentials
      +-- Private Keys
      +-- Temporary Files
      +-- Debug Data
      +-- Sensitive Configuration

Then:

    Publish Artifact

---

# Artifact Naming with Git SHA

A useful naming strategy is:

    application-${{ github.sha }}

Example:

    application-8f3a91d

This makes the artifact traceable to a Git commit.

However, artifact names must follow the naming rules supported by the GitHub artifact service.

---

# GitHub Run Number

Another identifier is:

    ${{ github.run_number }}

Example:

    application-build-152

This identifies the workflow run.

Git SHA provides stronger source traceability.

---

# Versioned Artifact

For a release:

    application-v1.4.0

This is useful for human-readable releases.

Possible combination:

    application-v1.4.0-8f3a91d

This provides both:

- Release Version
- Git Commit

---

# Artifact Traceability

A production artifact should ideally be traceable:

    Production
        |
        ↓
    Docker Image
        |
        ↓
    Artifact Version
        |
        ↓
    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Developer Change

This helps with debugging and auditing.

---

# Artifact Promotion

Artifact promotion means moving the same validated artifact through environments.

Example:

    Artifact
       |
       ↓
    QA
       |
       ↓
    SIT
       |
       ↓
    UAT
       |
       ↓
    Production

The artifact itself does not change.

---

# Environment Configuration

Application configuration should be separated from the artifact where practical.

Example:

    Artifact
        |
        +-- Application Code
        +-- Dependencies
        +-- Binary
        |
        ↓
    Environment Configuration
        |
        +-- QA
        +-- UAT
        +-- Production

This supports Build Once, Deploy Many.

---

# Artifact vs Environment

Artifact:

    WHAT to deploy

Environment:

    WHERE to deploy
    HOW to configure

Example:

    application.jar
        |
        +-- QA configuration
        +-- UAT configuration
        +-- Production configuration

The same application artifact can be promoted.

---

# GitHub Actions Job-to-Job Artifacts

Artifacts can be used to transfer files between jobs.

Example:

    jobs:

      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v6

          - name: Build
            run: |
              mkdir build
              echo "application" > build/app.txt

          - name: Upload
            uses: actions/upload-artifact@v4
            with:
              name: application
              path: build/

      deploy:
        needs: build
        runs-on: ubuntu-latest

        steps:

          - name: Download
            uses: actions/download-artifact@v4
            with:
              name: application

          - name: Deploy
            run: |
              ls -la
              echo "Deploying application"

---

# Why needs Is Important

Without:

    needs: build

the deployment job may run independently.

With:

    needs: build

the dependency becomes:

    Build
      |
      ↓
    Deploy

If Build fails:

    Build
      |
      ↓
    FAILED
      |
      X
    Deploy

---

# Multi-Job Artifact Flow

    Job 1
    Build
      |
      ↓
    Upload
      |
      ↓
    Artifact
      |
      ↓
    Job 2
    Test
      |
      ↓
    Download
      |
      ↓
    Test Artifact
      |
      ↓
    Job 3
    Deploy

---

# Separate Build and Deploy Jobs

Recommended structure:

    jobs:

      build:
        ...

      deploy:
        needs: build
        ...

This creates clear separation between:

    Build
        |
        ↓
    Artifact
        |
        ↓
    Deploy

---

# Build Artifact Workflow

Example:

    name: Build and Deploy

    on:
      push:
        branches:
          - main

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build
            run: |
              mkdir build
              echo "application" > build/application.txt

          - name: Upload Artifact
            uses: actions/upload-artifact@v4
            with:
              name: application
              path: build/

      deploy:

        needs: build
        runs-on: ubuntu-latest

        steps:

          - name: Download Artifact
            uses: actions/download-artifact@v4
            with:
              name: application

          - name: Deploy
            run: |
              ls -la
              echo "Deploying artifact"

---

# Download Artifact Path

Example:

    - name: Download
      uses: actions/download-artifact@v4
      with:
        name: application
        path: deployment/

The artifact is downloaded into:

    deployment/

Example:

    deployment/
    └── application.txt

---

# Download Multiple Artifacts

Suppose the workflow creates:

    backend
    frontend
    reports

They can be downloaded separately.

Example:

    - name: Download Backend
      uses: actions/download-artifact@v4
      with:
        name: backend
        path: backend/

    - name: Download Frontend
      uses: actions/download-artifact@v4
      with:
        name: frontend
        path: frontend/

---

# Download All Artifacts

The download action can also download multiple artifacts from the workflow run.

Example concept:

    - name: Download Artifacts
      uses: actions/download-artifact@v4

The exact directory structure depends on the selected options and artifact names.

---

# Artifact in Matrix Jobs

Suppose:

    strategy:
      matrix:
        service:
          - user
          - catalog
          - payment

Each matrix job can create an artifact.

Possible artifact names:

    user-build
    catalog-build
    payment-build

A naming strategy should avoid unintended collisions.

---

# Matrix Artifact Example

    jobs:

      build:

        strategy:
          matrix:
            service:
              - user
              - catalog
              - payment

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build
            run: |
              mkdir -p build
              echo "${{ matrix.service }}" > build/service.txt

          - name: Upload
            uses: actions/upload-artifact@v4
            with:
              name: ${{ matrix.service }}-build
              path: build/

Result:

    user-build
    catalog-build
    payment-build

---

# Artifact Naming in Matrix Jobs

Avoid:

    name: application

for every matrix job when multiple jobs upload the same artifact name.

Prefer:

    name: ${{ matrix.service }}-build

This creates unique artifact names.

---

# Artifact Overwrite

If multiple jobs intentionally upload the same artifact name, understand the overwrite behavior of the artifact action version being used.

For v4, artifacts are immutable once uploaded.

If multiple jobs need to contribute to one logical output, use unique artifact names and merge them later, or use an appropriate artifact strategy.

---

# Immutable Artifacts

An immutable artifact means that once uploaded, its contents are not silently modified.

This is important for:

- Reproducibility
- Security
- Auditing
- Traceability

Conceptually:

    Build 1
       |
       ↓
    Artifact A
       |
       ↓
    Immutable
       |
       ↓
    Deploy

---

# Artifact Immutability

Strong model:

    Source
      |
      ↓
    Build
      |
      ↓
    Test
      |
      ↓
    Artifact
      |
      ↓
    Immutable
      |
      +----> QA
      +----> UAT
      +----> Production

---

# GitHub Actions Artifact vs Package Registry

These are not the same thing.

GitHub Actions Artifact:

    Temporary / workflow-related build output

Package Registry:

    Long-term package distribution

Examples of package registries:

- GitHub Packages
- JFrog Artifactory
- npm Registry
- Maven Repository
- PyPI
- Amazon ECR

---

# GitHub Actions Artifact vs ECR

GitHub Actions Artifact:

    Build Output
        |
        ↓
    Workflow Storage

ECR:

    Docker Image
        |
        ↓
    Container Registry

Example:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    ECR

For Docker images, ECR is the deployment artifact repository.

---

# GitHub Actions Artifact vs JFrog Artifactory

GitHub Actions Artifact:

    Workflow-specific output

JFrog Artifactory:

    Enterprise artifact repository

Artifactory can store:

- Maven artifacts
- npm packages
- Python packages
- Docker images
- Other binaries

---

# When to Use GitHub Actions Artifacts

Use GitHub Actions artifacts for:

- Build output needed by later jobs
- Test reports
- Coverage reports
- Logs
- Debug files
- Temporary workflow handoffs

---

# When to Use Artifact Repository

Use an artifact repository for:

- Enterprise package management
- Long-term package storage
- Versioned binaries
- Container images
- Cross-pipeline consumption
- Release distribution

---

# Artifact Repository Architecture

    Developer
       |
       ↓
    GitHub
       |
       ↓
    GitHub Actions
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Security Scan
       |
       ↓
    Publish
       |
       ↓
    Artifact Repository
       |
       ↓
    Deployment

---

# Maven Artifact Publishing

For an enterprise Java application:

    Source
      |
      ↓
    Maven Build
      |
      ↓
    application.jar
      |
      ↓
    JFrog Artifactory
      |
      ↓
    Deployment

GitHub Actions can execute the Maven publishing command.

Example concept:

    mvn deploy

The repository credentials should be stored securely.

---

# npm Package Publishing

For Node.js packages:

    npm package
        |
        ↓
    npm publish
        |
        ↓
    Package Registry

Possible registries:

- npm
- GitHub Packages
- JFrog Artifactory

---

# Python Package Publishing

Python packages can be built and published.

Example:

    python -m build
        |
        ↓
    dist/
        |
        ↓
    twine upload
        |
        ↓
    Package Registry

Credentials should be stored securely.

---

# Docker Artifact Publishing

Docker images are published to container registries.

Example:

    Dockerfile
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Docker Push
        |
        ↓
    ECR

The Docker image itself is the deployable artifact.

---

# Docker Image Tagging

Example:

    docker build \
      -t $REGISTRY/myapp:${GITHUB_SHA} \
      .

Then:

    docker push \
      $REGISTRY/myapp:${GITHUB_SHA}

This provides traceability.

---

# Artifact Publishing with ECR

Typical AWS pipeline:

    GitHub Actions
         |
         ↓
    AWS Authentication
         |
         ↓
    ECR Login
         |
         ↓
    Docker Build
         |
         ↓
    Trivy
         |
         ↓
    Docker Push
         |
         ↓
    ECR

---

# Artifact Publishing with JFrog

Typical pipeline:

    GitHub Actions
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
    Trivy
         |
         ↓
    JFrog Artifactory
         |
         ↓
    Deployment

---

# Package vs Artifact

A package is generally intended for distribution and consumption by package managers.

An artifact is a broader concept.

Examples:

    Artifact
       |
       +-- JAR
       +-- ZIP
       +-- Docker Image
       +-- npm Package
       +-- Python Wheel
       +-- Test Report
       +-- Binary

---

# Artifact Metadata

Useful metadata includes:

- Application Name
- Version
- Git SHA
- Branch
- Build Number
- Build Date
- Commit
- Environment
- Dependency Information

This improves traceability.

---

# Artifact Manifest

A pipeline can create a metadata file.

Example:

    cat > build/metadata.txt <<EOF
    application=myapp
    version=${GITHUB_SHA}
    commit=${GITHUB_SHA}
    run=${GITHUB_RUN_NUMBER}
    EOF

Then upload:

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: application
        path: build/

This provides useful build metadata.

---

# Artifact Provenance

A strong software supply chain should answer:

    What was built?
    |
    ↓
    From which commit?
    |
    ↓
    By which workflow?
    |
    ↓
    With which dependencies?
    |
    ↓
    Which security checks passed?
    |
    ↓
    Where was it deployed?

Artifact metadata and immutable identifiers help answer these questions.

---

# Artifact Security Checks

Before publishing:

    Build
      |
      ↓
    Test
      |
      ↓
    SonarQube
      |
      ↓
    Trivy
      |
      ↓
    Security Gate
      |
      ↓
    Publish

This ensures security checks occur before artifact promotion.

---

# Artifact Publishing Pipeline

Example:

    jobs:

      build:
        ...

      test:
        needs: build
        ...

      sonarqube:
        needs: test
        ...

      security:
        needs: sonarqube
        ...

      publish:
        needs: security
        ...

Conceptually:

    Build
      ↓
    Test
      ↓
    SonarQube
      ↓
    Trivy
      ↓
    Publish

---

# Job Dependencies

Use:

    needs:

to define the pipeline order.

Example:

    publish:
      needs:
        - test
        - security

The publish job runs only after its required jobs succeed.

---

# Conditional Publishing

You may want publishing only from specific branches.

Example:

    if: github.ref == 'refs/heads/main'

Conceptually:

    Pull Request
        |
        ↓
    Build / Test / Scan
        |
        ↓
    No Publish

    Main Branch
        |
        ↓
    Build / Test / Scan
        |
        ↓
    Publish

---

# Publish Only After Successful Checks

Example:

    publish:
      needs:
        - build
        - test
        - security
      if: success()

This ensures publishing occurs only after required jobs succeed.

---

# Pull Request vs Main

Typical model:

    Pull Request
       |
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Trivy
       |
       ↓
    No Production Publish

    Main
       |
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Trivy
       +-- Publish
       |
       ↓
    Artifact Repository

---

# Release Artifact

For a release:

    Git Tag
       |
       ↓
    GitHub Actions
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Security
       |
       ↓
    Package
       |
       ↓
    Publish
       |
       ↓
    Release Artifact

---

# Tag-Based Publishing

Example:

    on:
      push:
        tags:
          - 'v*'

This can trigger the workflow for version tags such as:

    v1.0.0
    v1.1.0
    v2.0.0

---

# Release Artifact Naming

Example:

    myapp-v1.0.0.jar

or:

    myapp-v1.0.0-linux-amd64.tar.gz

Versioned names make release management easier.

---

# GitHub Releases

GitHub Releases can be used to distribute release assets.

Flow:

    Git Tag
       |
       ↓
    GitHub Actions
       |
       ↓
    Build
       |
       ↓
    Release Binary
       |
       ↓
    GitHub Release

This is different from ordinary workflow artifacts.

---

# Workflow Artifact vs Release Asset

Workflow Artifact:

    Generated during workflow
        |
        ↓
    Workflow Run

Release Asset:

    Associated with a GitHub Release
        |
        ↓
    Long-term Release Distribution

Use the appropriate mechanism for the use case.

---

# Artifact Expiration

Workflow artifacts may expire according to configured retention settings.

Therefore, do not use short-lived workflow artifacts as the only permanent storage for production release binaries.

For long-term storage, use an appropriate artifact or package repository.

---

# Enterprise Artifact Strategy

A mature organization may use:

    GitHub Actions
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
    Trivy
         |
         ↓
    Artifact Repository
         |
         ↓
    QA
         |
         ↓
    UAT
         |
         ↓
    Production

The artifact repository becomes the controlled source for promoted artifacts.

---

# Artifact Promotion Strategy

Example:

    JFrog Artifactory

    snapshot
       |
       ↓
    QA Repository
       |
       ↓
    UAT Repository
       |
       ↓
    Release Repository

The exact repository layout depends on the organization's artifact-management strategy.

---

# Immutable Promotion

A strong promotion model:

    Artifact A
       |
       ↓
    QA
       |
       ↓
    UAT
       |
       ↓
    Production

Do not modify Artifact A between environments.

Instead, change environment configuration.

---

# Why Immutable Artifacts Matter

Benefits:

- Reproducibility
- Traceability
- Faster Rollbacks
- Consistent Deployments
- Easier Auditing
- Reduced Configuration Drift

---

# Rollback Using Artifacts

Suppose:

    v1.0.0 → Production
    v1.1.0 → Production

If v1.1.0 fails:

    Production
        |
        ↓
    Rollback
        |
        ↓
    v1.0.0

If artifacts are immutable and available, rollback is straightforward.

---

# Artifact Rollback

    Current
       |
       ↓
    v1.1.0
       |
       X
    Problem
       |
       ↓
    Previous
       |
       ↓
    v1.0.0

This is one reason artifact repositories are important.

---

# Artifact Cleanup

Artifacts should be cleaned according to policy.

Possible cleanup criteria:

- Age
- Branch
- Build Status
- Release Status
- Environment
- Retention Period

Avoid deleting artifacts that are still required for production rollback or compliance.

---

# Artifact Storage Optimization

To reduce storage:

- Remove unnecessary files
- Compress large outputs
- Use appropriate retention
- Avoid duplicate artifacts
- Avoid uploading source trees unnecessarily
- Store release artifacts in proper repositories
- Clean temporary workflow artifacts

---

# Do Not Upload node_modules

Bad:

    path: .

This may upload:

    node_modules/

which can be extremely large.

Better:

    path: dist/

Build the application and upload only the required output.

---

# Do Not Upload .git

Avoid uploading:

    .git/

The Git repository itself is not normally required as a deployment artifact.

---

# Artifact Contents Review

Before upload:

    build/
    |
    +-- application.jar
    +-- config/
    +-- logs/
    +-- .env
    +-- credentials.json

Remove sensitive files:

    .env
    credentials.json

Then:

    build/
    |
    +-- application.jar
    +-- required configuration

---

# Artifact Validation

Before publishing:

    Build
      |
      ↓
    Validate
      |
      +-- File Exists
      +-- Correct Version
      +-- Correct SHA
      +-- Tests Passed
      +-- Security Passed
      |
      ↓
    Publish

---

# Artifact Checksum

Checksums can help verify file integrity.

Example:

    sha256sum application.jar

Output:

    abc123... application.jar

The checksum can be stored as metadata.

Conceptually:

    Artifact
       |
       ↓
    SHA-256
       |
       ↓
    Integrity Verification

---

# Artifact Integrity

When an artifact is downloaded:

    Download
       |
       ↓
    Calculate Checksum
       |
       ↓
    Compare
       |
       +-- Match → Valid
       |
       +-- Different → Investigate

---

# Artifact Signing

Enterprise environments may also digitally sign artifacts.

Conceptually:

    Build
      |
      ↓
    Artifact
      |
      ↓
    Sign
      |
      ↓
    Publish
      |
      ↓
    Verify Before Deployment

Signing provides stronger supply-chain assurance than a checksum alone.

---

# Artifact Supply Chain

A secure artifact lifecycle:

    Source
      |
      ↓
    Build
      |
      ↓
    Test
      |
      ↓
    Static Analysis
      |
      ↓
    Security Scan
      |
      ↓
    Artifact
      |
      ↓
    Sign
      |
      ↓
    Publish
      |
      ↓
    Promote
      |
      ↓
    Deploy

---

# Common Artifact Publishing Mistakes

## Mistake 1: Building Separately for Each Environment

Bad:

    Build → QA
    Build → UAT
    Build → Production

Better:

    Build
      ↓
    Artifact
      ↓
    QA
      ↓
    UAT
      ↓
    Production

---

# Mistake 2: Uploading the Entire Workspace

Bad:

    path: .

This may include:

- Secrets
- Dependencies
- Git files
- Temporary files
- Logs
- Unnecessary data

Better:

    path: build/

---

# Mistake 3: Using Non-Unique Artifact Names

Matrix jobs can overwrite or conflict if artifact names are not designed correctly.

Use:

    ${{ matrix.service }}-build

instead of:

    application

for every matrix job.

---

# Mistake 4: Publishing Before Security Checks

Bad:

    Build
      |
      ↓
    Publish
      |
      ↓
    Trivy

Better:

    Build
      |
      ↓
    Trivy
      |
      ↓
    Publish

---

# Mistake 5: Treating Workflow Artifacts as Permanent Storage

Workflow artifacts have retention policies.

For permanent production package storage, use an appropriate artifact repository.

---

# Mistake 6: Hardcoding Credentials

Never:

    username: admin
    password: password123

Use:

    GitHub Secrets

or appropriate workload identity mechanisms.

---

# Mistake 7: Rebuilding During Deployment

Bad:

    CI Build
       |
       ↓
    Deploy
       |
       ↓
    Build Again

Better:

    CI Build
       |
       ↓
    Artifact
       |
       ↓
    Deploy Same Artifact

---

# Artifact Publishing Best Practices

- Build once
- Test before publishing
- Run SonarQube before publishing
- Run Trivy before publishing
- Publish only validated artifacts
- Use immutable versions
- Use Git SHA for traceability
- Use meaningful artifact names
- Upload only required files
- Never upload secrets
- Use retention policies
- Use artifact repositories for long-term storage
- Separate artifact from environment configuration
- Keep deployment artifacts immutable
- Maintain rollback versions
- Use checksums or signing where required
- Document artifact ownership
- Control permissions
- Audit artifact access

---

# Production CI/CD Example

A production-style workflow:

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
       +-- Build
       |
       +-- Unit Tests
       |
       +-- SonarQube
       |
       +-- Docker Build
       |
       +-- Trivy
       |
       ↓
    Security / Quality Gate
       |
       +-- Fail → Stop
       |
       +-- Pass
             |
             ↓
        Publish Artifact
             |
             ↓
          ECR / JFrog
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

# Interview Questions

## Basic

1. What is an artifact?
2. What is artifact publishing?
3. Why do we publish build artifacts?
4. What is a GitHub Actions artifact?
5. What is actions/upload-artifact?
6. What is actions/download-artifact?
7. What is the difference between artifact and package?
8. What is an artifact repository?
9. Why should artifacts be immutable?
10. What is Build Once, Deploy Many?

---

# Intermediate Interview Questions

11. How do you upload a build artifact in GitHub Actions?

12. How do you download an artifact in another job?

13. How do you pass build output between jobs?

14. How do you publish a Java JAR?

15. How do you publish a Node.js build?

16. How do you publish a Python package?

17. How do you publish a Docker image?

18. How do you store test reports as artifacts?

19. How do you retain artifacts after a failed workflow?

20. How do you configure artifact retention?

21. How do you handle artifacts in matrix jobs?

22. How do you prevent artifact naming conflicts?

23. What is the difference between GitHub Actions artifacts and ECR?

24. What is the difference between GitHub Actions artifacts and JFrog Artifactory?

25. How do you implement artifact traceability?

---

# Advanced Interview Questions

26. Design an enterprise artifact publishing pipeline using GitHub Actions.

27. How would you implement Build Once, Deploy Many?

28. How would you prevent unscanned artifacts from being published?

29. How would you design artifact promotion across QA, UAT, and Production?

30. How would you implement immutable artifact promotion?

31. How would you design artifact rollback?

32. How would you manage artifact retention?

33. How would you protect artifacts from unauthorized access?

34. How would you prevent secrets from being published as artifacts?

35. How would you implement artifact signing?

36. How would you implement artifact checksums?

37. How would you integrate GitHub Actions with JFrog Artifactory?

38. How would you integrate GitHub Actions with Amazon ECR?

39. How would you design artifact management for microservices?

40. How would you use GitHub Actions artifacts between multiple jobs?

41. How would you design a secure software supply chain?

---

# Scenario Question

## A build succeeds but the deployment job cannot find the build artifact. How would you troubleshoot it?

I would check:

    1. Did the build job succeed?
    2. Was upload-artifact executed?
    3. Is the artifact name correct?
    4. Is the path correct?
    5. Does the deployment job use needs: build?
    6. Is download-artifact using the same artifact name?
    7. Is the download path correct?
    8. Did the artifact expire?
    9. Was the artifact uploaded from the expected directory?

Flow:

    Build
      |
      ↓
    Upload
      |
      ↓
    Artifact Exists?
      |
      ↓
    Download
      |
      ↓
    Verify Files
      |
      ↓
    Deploy

---

# Scenario Question

## Your matrix jobs are failing because of artifact conflicts. What would you do?

I would make artifact names unique using the matrix value.

Example:

    name: ${{ matrix.service }}-build

Instead of:

    name: application

This creates:

    user-build
    catalog-build
    payment-build

---

# Scenario Question

## The deployment team wants the same artifact deployed to QA, UAT, and Production. How would you design it?

I would use Build Once, Deploy Many.

    Build
      |
      ↓
    Test
      |
      ↓
    Security
      |
      ↓
    Publish Artifact
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    Production

I would not rebuild the application for each environment.

Environment-specific configuration would be injected separately.

---

# Scenario Question

## How would you prevent a vulnerable Docker image from being published?

I would place Trivy before the publishing step.

    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    Security Gate
       |
       +-- Fail → No Push
       |
       +-- Pass → Push ECR

The publish job would depend on the successful security job.

---

# Scenario Question

## How would you roll back an application using artifacts?

I would keep immutable, versioned artifacts.

Example:

    v1.0.0
    v1.1.0
    v1.2.0

If v1.2.0 fails:

    Production
        |
        ↓
    v1.2.0
        |
        X
    Rollback
        |
        ↓
    v1.1.0

The deployment system would deploy the previously validated artifact.

---

# Scenario Question

## How would you integrate GitHub Actions with JFrog Artifactory?

I would design:

    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Trivy
       |
       ↓
    JFrog Artifactory
       |
       ↓
    Deployment

Credentials would be stored securely and publishing would happen only after required CI checks pass.

---

# Scenario Question

## How would you integrate GitHub Actions with ECR?

I would use:

    GitHub Actions
        |
        ↓
    AWS Authentication
        |
        ↓
    ECR Login
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Docker Push
        |
        ↓
    ECR

For AWS authentication, GitHub OIDC can be used to avoid long-lived AWS access keys where organizational policy supports it.

---

# Scenario Question

## Why should the same artifact be deployed to production that was tested in QA?

Because rebuilding can change the artifact.

Example:

    Tested Artifact
          |
          ↓
        QA
          |
          ↓
        UAT
          |
          ↓
    Production

This ensures the exact artifact tested earlier is what reaches production.

---

# Scenario Question

## What should you store in GitHub Actions artifacts?

Good candidates:

- Build Output
- Test Reports
- Coverage Reports
- Logs
- Debug Information
- Generated Documentation
- Temporary Deployment Packages

Avoid storing:

- Passwords
- API Keys
- Private Keys
- Secrets
- Entire Repository
- Unnecessary Dependencies

---

# Scenario Question

## What is the difference between an artifact and a Docker image?

Artifact:

    General Build Output

Docker Image:

    Containerized Application Artifact

Example:

    Java
      |
      ↓
    application.jar

Docker:

    Dockerfile
      |
      ↓
    Docker Image
      |
      ↓
    ECR

Both are artifacts, but they are stored and consumed differently.

---

# Quick Revision

Remember:

    Source Code
        |
        ↓
    GitHub Actions
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
    Trivy
        |
        ↓
    Artifact
        |
        ↓
    Artifact Repository
        |
        ↓
    Deployment

GitHub Actions artifact:

    actions/upload-artifact

Download:

    actions/download-artifact

Artifact name:

    name:

Artifact files:

    path:

Retention:

    retention-days:

Job dependency:

    needs:

Conditional publishing:

    if:

Important principle:

    Build Once
        |
        ↓
    Test Once
        |
        ↓
    Publish Once
        |
        ↓
    Deploy Same Artifact

---

# Important GitHub Actions Syntax

Upload:

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: application
        path: build/

Download:

    - name: Download Artifact
      uses: actions/download-artifact@v4
      with:
        name: application
        path: deployment/

Retention:

    retention-days: 7

Always upload:

    if: always()

Job dependency:

    needs: build

Multiple dependencies:

    needs:
      - build
      - test
      - security

Branch condition:

    if: github.ref == 'refs/heads/main'

---

# Important Tools and Platforms

GitHub Actions:

    Build Automation
    CI/CD

GitHub Actions Artifacts:

    Workflow Output Storage

JFrog Artifactory:

    Enterprise Artifact Repository

Amazon ECR:

    Container Registry

GitHub Packages:

    Package Registry

SonarQube:

    Code Quality / Security Analysis

Trivy:

    Vulnerability / Security Scanning

Docker:

    Container Image Creation

ArgoCD:

    GitOps Deployment

EKS:

    Kubernetes Platform

---

# Final Mental Model

Think of an artifact as:

    "The exact output produced by CI
     that CD should deploy."

The complete flow is:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Checkout
        |
        +-- Build
        |
        +-- Test
        |
        +-- SonarQube
        |
        +-- Docker Build
        |
        +-- Trivy
        |
        ↓
    Validated Artifact
        |
        ↓
    Artifact Repository
        |
        ↓
    QA
        |
        ↓
    SIT
        |
        ↓
    UAT
        |
        ↓
    Production

Core idea:

Artifact publishing creates a controlled handoff between CI and CD. GitHub Actions artifacts are useful for workflow outputs such as build files, reports, and logs, while dedicated artifact repositories such as JFrog Artifactory, GitHub Packages, or Amazon ECR are appropriate for longer-lived packages and deployment artifacts. The most important DevOps principle is to build and validate an artifact once, then promote the same validated artifact through environments instead of rebuilding it for every environment.