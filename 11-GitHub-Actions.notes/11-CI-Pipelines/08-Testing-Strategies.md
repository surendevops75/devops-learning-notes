# Testing Strategies with GitHub Actions

Testing is a critical part of CI/CD.

A CI pipeline should automatically validate application changes before they are merged, packaged, published, or deployed.

Typical flow:

Developer
    |
    ↓
Git Push / Pull Request
    |
    ↓
GitHub Actions
    |
    +-- Build
    |
    +-- Unit Tests
    |
    +-- Integration Tests
    |
    +-- Static Analysis
    |
    +-- Security Scanning
    |
    +-- Package
    |
    ↓
Quality Gate
    |
    +-- Fail → Stop
    |
    +-- Pass → Continue
    |
    ↓
Artifact Publishing
    |
    ↓
Deployment

---

# Why Testing Is Important in CI

Without automated testing:

    Developer
        |
        ↓
    Code Change
        |
        ↓
    Build
        |
        ↓
    Deploy
        |
        ↓
    Production Issue

With automated testing:

    Developer
        |
        ↓
    Code Change
        |
        ↓
    GitHub Actions
        |
        ↓
    Tests
        |
        +-- Fail → Stop
        |
        +-- Pass
              |
              ↓
            Build
              |
              ↓
          Publish
              |
              ↓
           Deploy

Testing provides fast feedback before a change reaches later environments.

---

# What Is a Testing Strategy?

A testing strategy defines:

- What should be tested
- When it should be tested
- Where it should be tested
- Which tests should block the pipeline
- Which tests are informational
- How test results are reported
- How failures are handled

A good CI testing strategy balances:

    Speed
      +
    Coverage
      +
    Reliability
      +
    Feedback

---

# Testing Pyramid

A common testing model is the testing pyramid.

             /\
            /  \
           / E2E\
          /------\
         /  API   \
        /----------\
       / Integration\
      /--------------\
     /   Unit Tests   \
    /------------------\

The general principle is:

    Many Fast Unit Tests
          |
          ↓
    Fewer Integration Tests
          |
          ↓
    Fewer End-to-End Tests

Unit tests are usually fast.

End-to-end tests are usually slower and more complex.

---

# Testing Levels

Common testing levels include:

- Unit Testing
- Integration Testing
- API Testing
- Functional Testing
- End-to-End Testing
- Regression Testing
- Smoke Testing
- Performance Testing
- Security Testing

Not every test needs to run on every Pull Request.

---

# Unit Testing

Unit testing validates small units of application logic.

Examples:

Java:

    JUnit

Node.js:

    Jest
    Vitest

Python:

    pytest

Example:

    Function
       |
       ↓
    Unit Test
       |
       ↓
    Pass / Fail

Unit tests should generally be fast and deterministic.

---

# Unit Test Example: Java

Maven:

    mvn test

GitHub Actions:

    - name: Run Unit Tests
      run: mvn test

Flow:

    Java Source
        |
        ↓
    Maven
        |
        ↓
    JUnit
        |
        ↓
    Test Result

---

# Unit Test Example: Node.js

Example:

    - name: Install Dependencies
      run: npm ci

    - name: Run Unit Tests
      run: npm test

The test command should be defined in:

    package.json

Example:

    {
      "scripts": {
        "test": "jest"
      }
    }

---

# Unit Test Example: Python

Example:

    - name: Install Dependencies
      run: pip install -r requirements.txt

    - name: Run Tests
      run: pytest

Python projects commonly use:

    pytest

for automated testing.

---

# Unit Tests in Pull Requests

A typical Pull Request pipeline:

    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    Build
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Merge

If unit tests fail:

    Unit Test
        |
        ↓
      FAIL
        |
        X
    Merge Blocked

---

# Integration Testing

Integration tests verify that multiple components work together.

Example:

    Application
        |
        +-- Database
        +-- Redis
        +-- RabbitMQ
        +-- External Service
        |
        ↓
    Integration Test

Unit testing:

    Function → Function

Integration testing:

    Service → Database

or:

    Service → Message Queue

---

# Integration Test Example

Suppose a microservice uses:

    Application
        |
        ↓
    PostgreSQL

Integration testing validates:

    Application
        |
        ↓
    Database Connection
        |
        ↓
    Query
        |
        ↓
    Response

This catches issues that unit tests may not detect.

---

# Integration Testing with Containers

GitHub Actions can run dependencies using containers.

Example:

    Application
        |
        +-- PostgreSQL Container
        |
        +-- Redis Container
        |
        ↓
    Integration Tests

This creates a more realistic test environment.

---

# GitHub Actions Service Containers

Example:

    jobs:

      test:

        runs-on: ubuntu-latest

        services:

          postgres:
            image: postgres:16
            env:
              POSTGRES_PASSWORD: postgres
              POSTGRES_DB: testdb
            ports:
              - 5432:5432

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Run Tests
            run: |
              npm ci
              npm run test:integration

---

# Service Container Flow

    GitHub Runner
         |
         +-- Application
         |
         +-- PostgreSQL
         |
         +-- Redis
         |
         ↓
    Integration Tests

This allows CI to test interactions between components.

---

# API Testing

API testing validates HTTP or service APIs.

Example:

    Client
      |
      ↓
    API
      |
      ↓
    Response

Possible validations:

- HTTP Status
- Response Body
- Headers
- Authentication
- Authorization
- Error Handling
- Response Time

---

# API Test Example

Example:

    curl -f http://localhost:8080/health

The `-f` option causes curl to return a non-zero exit status for certain HTTP errors.

GitHub Actions:

    - name: API Health Check
      run: |
        curl -f http://localhost:8080/health

If the API is unavailable:

    curl
      |
      ↓
    Non-Zero Exit Code
      |
      ↓
    Job Fails

---

# End-to-End Testing

End-to-end testing validates the complete application flow.

Example:

    User
      |
      ↓
    Frontend
      |
      ↓
    API
      |
      ↓
    Service
      |
      ↓
    Database
      |
      ↓
    Response

E2E tests validate the system as a whole.

---

# E2E Test Example

A microservices application might test:

    Login
      |
      ↓
    Product Search
      |
      ↓
    Add to Cart
      |
      ↓
    Create Order
      |
      ↓
    Payment
      |
      ↓
    Order Confirmation

This provides high confidence but is usually slower than unit testing.

---

# Smoke Testing

Smoke tests verify that the application is basically operational.

Example:

    Deployment
       |
       ↓
    Health Check
       |
       +-- 200 → Continue
       |
       +-- Failure → Stop

Common smoke tests:

    /health
    /ready
    /version

---

# Smoke Test Example

Example:

    - name: Smoke Test
      run: |
        curl -f https://example.com/health

The pipeline can stop if the application does not respond successfully.

---

# Regression Testing

Regression testing verifies that existing functionality still works after changes.

Example:

    Existing Application
          |
          ↓
      New Change
          |
          ↓
    Regression Tests
          |
          ↓
    Existing Features
          |
          ↓
    Pass / Fail

Regression tests are especially important for mature applications.

---

# Functional Testing

Functional tests verify that the application behaves according to business requirements.

Example:

    Input:
      username
      password

    Expected:
      Successful Login

Test:

    Login
      |
      ↓
    Authentication
      |
      ↓
    Expected Result

---

# Performance Testing

Performance testing evaluates:

- Response Time
- Throughput
- Concurrent Users
- Resource Utilization
- System Behavior Under Load

Examples of tools:

- k6
- JMeter
- Gatling

Performance testing is often separated from the fastest Pull Request checks.

---

# Load Testing

Load testing determines how an application behaves under expected traffic.

Example:

    100 Users
        |
        ↓
    Application
        |
        ↓
    Response Time
        |
        ↓
    Throughput

Then increase load:

    100
    500
    1000
    5000

and observe system behavior.

---

# Stress Testing

Stress testing pushes the system beyond normal expected capacity.

Example:

    Normal:
       1,000 users

    Stress:
       10,000 users

Goal:

    Determine system limits
       +
    Observe failure behavior
       +
    Validate recovery

---

# Security Testing

Security testing can include:

- SAST
- SCA
- Container Scanning
- Secret Scanning
- DAST
- IaC Scanning

In your DevSecOps pipeline:

    Source Code
        |
        ↓
    SonarQube
        |
        ↓
    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Security Gate

---

# SonarQube and Testing

SonarQube can analyze source code and use test coverage information where configured.

Conceptually:

    Source Code
        |
        +-- Static Analysis
        |
        +-- Test Coverage
        |
        ↓
    SonarQube
        |
        ↓
    Quality Gate

---

# Test Coverage

Test coverage measures how much of the code is exercised by tests.

Common coverage metrics:

- Line Coverage
- Branch Coverage
- Function Coverage
- Statement Coverage

Example:

    100 Lines
       |
       ↓
    80 Executed
       |
       ↓
    80% Line Coverage

Coverage is useful, but high coverage does not automatically mean high-quality tests.

---

# Coverage Example

Java:

    mvn test

Coverage can be generated using tools such as JaCoCo.

Node.js:

    npm test -- --coverage

Python:

    pytest --cov

---

# Coverage in GitHub Actions

Example:

    - name: Run Tests
      run: npm test -- --coverage

    - name: Upload Coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coverage/

Flow:

    Tests
      |
      ↓
    Coverage
      |
      ↓
    Upload Artifact

---

# Coverage Threshold

A project may define a minimum coverage threshold.

Example:

    Required:
      80%

    Actual:
      74%

Result:

    Coverage Gate
        |
        ↓
      FAIL

The exact threshold should be based on project requirements.

---

# Test Reports

Test reports provide structured information about test execution.

Examples:

    test-results.xml
    junit.xml
    coverage.xml

Reports can be uploaded as GitHub Actions artifacts.

---

# JUnit Test Report

JUnit XML is a common test-report format.

Example:

    - name: Run Tests
      run: mvn test

    - name: Upload Test Results
      uses: actions/upload-artifact@v4
      with:
        name: junit-results
        path: target/surefire-reports/

---

# Test Artifacts

Useful test artifacts include:

- JUnit XML
- Coverage Reports
- Screenshots
- Browser Logs
- Application Logs
- Test Videos
- Debug Information

Use:

    if: always()

when you want diagnostic artifacts even after a test failure.

---

# Testing on Pull Request

Example:

    name: Pull Request Tests

    on:
      pull_request:

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Install Dependencies
            run: npm ci

          - name: Run Tests
            run: npm test

---

# Testing on Push

Example:

    on:
      push:
        branches:
          - main

This validates changes after they are pushed to the main branch.

---

# Pull Request vs Push Testing

Pull Request:

    Developer
       |
       ↓
    Pull Request
       |
       ↓
    Tests
       |
       ↓
    Review
       |
       ↓
    Merge

Push to main:

    Merge
       |
       ↓
    Main Branch
       |
       ↓
    CI
       |
       ↓
    Publish / Deploy

Both can be useful.

---

# Test on Every Pull Request

Typical:

    on:
      pull_request:

This provides early feedback.

The pipeline can include:

    Build
    Unit Test
    Static Analysis
    Security Scan

---

# Test Only Selected Paths

GitHub Actions can use path filters.

Example:

    on:
      pull_request:
        paths:
          - 'src/**'
          - 'tests/**'

This can prevent unnecessary workflow execution for unrelated changes.

---

# Path-Based Testing

Example repository:

    frontend/
    backend/
    infrastructure/

A backend workflow may use:

    paths:
      - 'backend/**'

This avoids triggering the backend pipeline for changes only to frontend files.

---

# Branch-Based Testing

Example:

    on:
      push:
        branches:
          - main
          - develop

This allows testing on selected branches.

---

# Testing Matrix

A matrix allows testing across multiple versions or platforms.

Example:

    strategy:
      matrix:
        node:
          - 18
          - 20
          - 22

Then:

    Node 18
       |
    Node 20
       |
    Node 22

Each version runs independently.

---

# Node.js Matrix Example

    jobs:

      test:

        strategy:
          matrix:
            node-version:
              - 18
              - 20
              - 22

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: ${{ matrix.node-version }}
              cache: npm

          - name: Install
            run: npm ci

          - name: Test
            run: npm test

---

# Java Matrix Example

    strategy:
      matrix:
        java-version:
          - '17'
          - '21'

The application can then be tested against both supported Java versions.

---

# Python Matrix Example

    strategy:
      matrix:
        python-version:
          - '3.10'
          - '3.11'
          - '3.12'

Each supported version can be tested independently.

---

# Operating System Matrix

Example:

    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

This is useful when the application must support multiple operating systems.

---

# Fail-Fast in Matrix Testing

Matrix jobs can use:

    fail-fast: true

or:

    fail-fast: false

With fail-fast enabled, GitHub Actions can cancel in-progress and queued matrix jobs when a matrix job fails.

For broad compatibility testing, disabling fail-fast may provide more complete information.

---

# Test Parallelization

Large test suites can be slow.

Example:

    5,000 Tests
        |
        ↓
    One Runner
        |
        ↓
    30 Minutes

Parallel:

    5,000 Tests
        |
        +-- Runner 1
        +-- Runner 2
        +-- Runner 3
        +-- Runner 4
        |
        ↓
    Reduced Runtime

GitHub Actions matrix jobs can help implement parallel test execution.

---

# Test Sharding

Test sharding divides a large test suite into multiple groups.

Example:

    Tests
      |
      +-- Shard 1
      +-- Shard 2
      +-- Shard 3
      +-- Shard 4

Each shard runs independently.

The exact implementation depends on the testing framework.

---

# Parallel Testing Strategy

Example:

    strategy:
      matrix:
        shard:
          - 1
          - 2
          - 3
          - 4

Each job can run a different test shard.

This can reduce CI execution time when the test suite is large.

---

# Unit + Integration Test Strategy

A common approach:

    Pull Request
        |
        +-- Unit Tests
        |
        +-- Static Analysis
        |
        +-- Security Scan
        |
        ↓
    Merge

After merge:

    Main
        |
        +-- Unit Tests
        +-- Integration Tests
        +-- Package
        +-- Publish
        |
        ↓
    Deployment

This keeps Pull Request feedback relatively fast.

---

# Fast Feedback Strategy

Not every test should run at every stage.

Example:

Pull Request:

    Unit
    Static Analysis
    Security Scan

Main:

    Unit
    Integration
    API
    Security

Nightly:

    E2E
    Performance
    Extended Regression

This balances feedback speed and coverage.

---

# Testing Strategy by Pipeline Stage

    Pull Request
        |
        +-- Unit
        +-- Lint
        +-- SAST
        +-- Dependency Scan
        |
        ↓
    Main
        |
        +-- Integration
        +-- API
        +-- Container Scan
        |
        ↓
    Staging
        |
        +-- E2E
        +-- Smoke
        +-- Regression
        |
        ↓
    Production
        |
        +-- Smoke
        +-- Health Checks

---

# Test Gates

A test gate determines whether the pipeline can continue.

Example:

    Unit Tests
        |
        ↓
    Gate
        |
        +-- Pass → Continue
        |
        +-- Fail → Stop

This is one of the most important CI concepts.

---

# Quality Gate

Quality gate may include:

    Unit Tests
       +
    Coverage
       +
    SonarQube
       +
    Trivy
       |
       ↓
    Quality / Security Gate
       |
       +-- Pass
       |
       +-- Fail

Only successful pipelines should publish production artifacts.

---

# Test Failure Behavior

Normally, if a test command returns a non-zero exit code:

    Test
      |
      ↓
    Exit Code 1
      |
      ↓
    GitHub Actions Step Failed
      |
      ↓
    Job Failed

This automatically protects later stages.

---

# Continue-on-Error

GitHub Actions supports:

    continue-on-error: true

Example:

    - name: Optional Test
      continue-on-error: true
      run: ./optional-test.sh

This allows the workflow to continue even if the step fails.

Use carefully.

---

# When to Use continue-on-error

Potential use cases:

- Informational tests
- Experimental checks
- Non-blocking compatibility tests
- Temporary migration checks

Do not use it to hide critical test failures.

Bad:

    Production Tests
        |
        ↓
    continue-on-error: true

This can allow broken code to continue through the pipeline.

---

# Test Dependencies

Testing may require:

- Database
- Redis
- RabbitMQ
- Kafka
- External API
- Browser
- Service Containers

The CI environment should provide required dependencies consistently.

---

# Test Environment

A typical test environment:

    GitHub Runner
        |
        +-- Application
        +-- Database
        +-- Cache
        +-- Message Queue
        |
        ↓
    Automated Tests

The environment should be isolated and reproducible.

---

# Test Data

Tests require predictable test data.

Good test data should be:

- Reproducible
- Isolated
- Safe
- Non-production
- Easy to reset

Never use real production credentials or sensitive production data in CI tests.

---

# Database Testing

Integration tests may require:

    PostgreSQL
       |
       ↓
    Schema
       |
       ↓
    Test Data
       |
       ↓
    Application
       |
       ↓
    Test

Database state should be reset or isolated between test runs.

---

# Database Migration Testing

If an application uses migrations:

    Migration
        |
        ↓
    Test Database
        |
        ↓
    Application
        |
        ↓
    Tests

This validates that migrations work correctly.

---

# API Contract Testing

Contract testing verifies that service interfaces remain compatible.

Example:

    Service A
        |
        ↓
    API Contract
        |
        ↓
    Service B

This is especially useful in microservices architectures.

---

# Microservices Testing Strategy

For microservices:

    Service
       |
       +-- Unit Tests
       |
       +-- Integration Tests
       |
       +-- Contract Tests
       |
       +-- API Tests
       |
       +-- E2E Tests
       |
       +-- Security Tests

The goal is to test both individual services and critical system interactions.

---

# Microservices CI Pipeline

    Code
      |
      ↓
    Unit Tests
      |
      ↓
    Integration Tests
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
    Artifact
      |
      ↓
    Deployment

---

# Testing a Docker Image

After building an image:

    Docker Build
        |
        ↓
    Container Start
        |
        ↓
    Health Check
        |
        ↓
    API Test
        |
        ↓
    Stop Container

Example:

    docker run -d \
      --name myapp \
      -p 8080:8080 \
      myapp:${GITHUB_SHA}

Then:

    curl -f http://localhost:8080/health

---

# Container Smoke Test

Example:

    - name: Start Container
      run: |
        docker run -d \
          --name myapp \
          -p 8080:8080 \
          myapp:${{ github.sha }}

    - name: Health Check
      run: |
        curl --retry 10 --retry-delay 2 -f \
          http://localhost:8080/health

This validates that the built container can start and respond.

---

# Why Test the Container?

A successful Docker build does not guarantee that the application works correctly.

Possible issue:

    Docker Build
        |
        ↓
      SUCCESS
        |
        ↓
    Container Start
        |
        ↓
      FAILURE

Container smoke tests catch runtime startup problems.

---

# Testing Health Endpoints

Common endpoints:

    /health
    /healthz
    /ready
    /version

Example:

    curl -f http://localhost:8080/health

Health checks can validate basic application availability.

---

# Retry Strategy for Health Checks

Applications may need time to start.

Bad:

    docker run
        |
        ↓
    curl immediately
        |
        ↓
    Failure

Better:

    docker run
        |
        ↓
    Wait / Retry
        |
        ↓
    Health Check

Example:

    curl --retry 10 --retry-delay 2 -f \
      http://localhost:8080/health

---

# Test Timeouts

A test that never finishes can block CI.

GitHub Actions jobs and steps can have time limits.

Example:

    timeout-minutes: 10

Example:

    jobs:

      test:
        runs-on: ubuntu-latest
        timeout-minutes: 10

Timeouts protect CI resources.

---

# Test Flakiness

A flaky test sometimes passes and sometimes fails without a meaningful code change.

Example:

    Run 1 → PASS
    Run 2 → FAIL
    Run 3 → PASS
    Run 4 → FAIL

Flaky tests reduce trust in CI.

---

# Causes of Flaky Tests

Common causes:

- Timing Issues
- Race Conditions
- External Dependencies
- Shared State
- Random Test Data
- Network Dependency
- Poor Cleanup
- Time Zone Differences
- Resource Constraints

---

# Handling Flaky Tests

Do not simply ignore them.

Process:

    Flaky Test
        |
        ↓
    Identify Cause
        |
        ↓
    Fix Test / Application
        |
        ↓
    Validate
        |
        ↓
    Stable Test

Retries may reduce temporary failures but should not replace fixing the root cause.

---

# Test Retries

Some frameworks support retrying failed tests.

Use retries carefully.

Bad:

    Test Fails
       |
       ↓
    Retry
       |
       ↓
    Pass
       |
       ↓
    Ignore Root Cause

Better:

    Test Fails
       |
       ↓
    Investigate
       |
       ↓
    Fix Flakiness

---

# Deterministic Tests

A good CI test should produce the same result when the code and environment are unchanged.

Good:

    Same Input
       |
       ↓
    Same Environment
       |
       ↓
    Same Result

Avoid unnecessary dependence on:

- Current Time
- Randomness
- External APIs
- Shared Environments
- Network State

---

# Testing External APIs

External APIs can make CI unstable.

Possible approach:

    Application
        |
        ↓
    Mock / Stub
        |
        ↓
    External API Behavior

Use real external integrations only where necessary.

---

# Unit Test Mocking

Example:

    Application
        |
        ↓
    Mock Payment Service
        |
        ↓
    Test Payment Logic

This allows unit tests to focus on application behavior without requiring the real payment service.

---

# Integration vs Mocking

Unit testing:

    Application
        |
        ↓
    Mock Database

Integration testing:

    Application
        |
        ↓
    Real Test Database

Both have value.

---

# Test Isolation

Tests should ideally not depend on another test's state.

Bad:

    Test A
       |
       ↓
    Creates Data

    Test B
       |
       ↓
    Expects Data From A

Better:

    Test A
       |
       ↓
    Own Test Data

    Test B
       |
       ↓
    Own Test Data

---

# Parallel Test Safety

When tests run in parallel:

    Test 1
    Test 2
    Test 3

they should not interfere with each other.

Use isolated:

- Databases
- Files
- Ports
- Test Data
- Temporary Directories

---

# Testing and Caching

Caching dependencies can make CI faster.

Example:

    Dependency Cache
        |
        ↓
    npm / Maven / pip
        |
        ↓
    Tests

Caching should not cause stale dependencies to be used incorrectly.

---

# Node.js Dependency Caching

Example:

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

Then:

    npm ci
    npm test

---

# Maven Dependency Caching

Example:

    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'
        cache: maven

Then:

    mvn test

---

# Python Dependency Caching

Example:

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip

Then:

    pip install -r requirements.txt

---

# Testing and Artifacts

A strong test workflow:

    Run Tests
        |
        +-- Pass
        |    |
        |    ↓
        |  Continue
        |
        +-- Fail
             |
             ↓
        Upload Reports
             |
             ↓
           Fail Job

This preserves diagnostics even when the pipeline fails.

---

# Test Reporting

Example:

    - name: Run Tests
      run: npm test -- --coverage

    - name: Upload Test Reports
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: test-reports
        path: |
          coverage/
          reports/

---

# Test Results and Pull Requests

A good CI pipeline should make test results easy to understand.

Developer should quickly know:

    Tests
      |
      +-- 125 Passed
      +-- 0 Failed
      +-- Coverage: 84%
      |
      ↓
    Ready for Review

or:

    Tests
      |
      +-- 120 Passed
      +-- 5 Failed
      |
      ↓
    Fix Required

---

# Required Status Checks

GitHub branch protection can require CI checks before merging.

Example:

    Pull Request
        |
        ↓
    Required Checks
        |
        +-- Build
        +-- Unit Tests
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Merge Allowed

If a required check fails:

    Merge
      |
      X
    Blocked

---

# Testing Strategy for Main Branch

Example:

    Main Branch
        |
        ↓
    Build
        |
        ↓
    Unit Tests
        |
        ↓
    Integration Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Package
        |
        ↓
    Publish Artifact

---

# Testing Strategy for Production

Before production:

    Artifact
       |
       ↓
    Deploy to Staging
       |
       ↓
    Smoke Test
       |
       ↓
    E2E
       |
       ↓
    Approval
       |
       ↓
    Production
       |
       ↓
    Post-Deployment Smoke Test

---

# Post-Deployment Testing

After deployment:

    Production Deployment
           |
           ↓
      Health Check
           |
           ↓
        Smoke Test
           |
           +-- Pass → Success
           |
           +-- Fail → Rollback / Incident

This validates that the deployment actually works.

---

# Test Environment Promotion

Example:

    CI
      |
      ↓
    QA
      |
      ↓
    Integration Tests
      |
      ↓
    UAT
      |
      ↓
    E2E Tests
      |
      ↓
    Production
      |
      ↓
    Smoke Tests

---

# Testing in Enterprise CI/CD

Enterprise workflow:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Code Review
        |
        ↓
    Main
        |
        ↓
    Integration Tests
        |
        ↓
    Artifact
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    Production
        |
        ↓
    Smoke Tests

---

# Testing Strategy for Microservices

For each service:

    Source
       |
       +-- Unit
       |
       +-- Integration
       |
       +-- API
       |
       +-- Security
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    Artifact

System-level:

    Services
       |
       ↓
    E2E
       |
       ↓
    Regression

---

# Testing Strategy for Your DevOps Projects

A practical microservices pipeline can be:

    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       |
       +-- Maven / npm / Python Build
       |
       +-- Unit Tests
       |
       +-- Integration Tests
       |
       +-- SonarQube
       |
       +-- Docker Build
       |
       +-- Trivy
       |
       ↓
    Artifact / ECR
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
    Smoke Test

---

# Testing and GitOps

GitHub Actions:

    Build
    Test
    Security
    Publish

GitOps:

    Deployment Configuration
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

Testing happens before the artifact is promoted.

---

# Testing and ArgoCD

ArgoCD deploys the desired state from Git.

Testing should happen before the deployment artifact or deployment configuration is promoted.

Flow:

    Code
      |
      ↓
    CI
      |
      +-- Test
      +-- SonarQube
      +-- Trivy
      |
      ↓
    Artifact
      |
      ↓
    GitOps
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Smoke Test

---

# Test Automation Principles

Good CI tests should be:

- Fast
- Reliable
- Repeatable
- Isolated
- Deterministic
- Maintainable
- Relevant
- Automated

Avoid tests that are:

- Random
- Slow Without Reason
- Environment Dependent
- Flaky
- Difficult to Debug

---

# Shift Left Testing

Shift-left means testing earlier in the software lifecycle.

Traditional:

    Development
       |
       ↓
    QA
       |
       ↓
    Production
       |
       ↓
    Find Issue

Shift left:

    Development
       |
       ↓
    Unit Tests
       |
       ↓
    Static Analysis
       |
       ↓
    Security Scan
       |
       ↓
    Integration
       |
       ↓
    QA
       |
       ↓
    Production

The goal is to find defects earlier when they are cheaper to fix.

---

# Shift Right Testing

Shift-right practices validate software after deployment.

Examples:

- Production Monitoring
- Smoke Tests
- Synthetic Tests
- Canary Validation
- User Monitoring
- Incident Detection

Flow:

    Deployment
       |
       ↓
    Production
       |
       ↓
    Validation
       |
       ↓
    Monitoring

---

# Testing and Deployment Strategy

For a canary deployment:

    Version 1
       |
       ↓
    Production
       |
       ↓
    Small Traffic
       |
       ↓
    Smoke / Health Checks
       |
       ↓
    Metrics
       |
       +-- Healthy → Increase Traffic
       |
       +-- Unhealthy → Rollback

Testing and observability work together.

---

# Test Automation Anti-Patterns

## Testing Only After Deployment

Bad:

    Build
      |
      ↓
    Deploy Production
      |
      ↓
    Test

Better:

    Build
      |
      ↓
    Test
      |
      ↓
    Security
      |
      ↓
    Publish
      |
      ↓
    Deploy

---

# Anti-Pattern: Only Unit Tests

Unit tests are important but do not validate the complete system.

Need:

    Unit
      +
    Integration
      +
    API
      +
    E2E
      +
    Smoke

Use the appropriate level for the application.

---

# Anti-Pattern: Only E2E Tests

E2E tests are often:

- Slow
- Expensive
- More fragile
- Harder to debug

Use more fast tests and fewer broad E2E tests.

---

# Anti-Pattern: Ignoring Flaky Tests

Flaky tests reduce trust in CI.

If developers repeatedly see:

    FAIL
      |
      ↓
    Retry
      |
      ↓
    PASS

they may stop taking CI failures seriously.

Fix flaky tests.

---

# Anti-Pattern: 100% Coverage as the Only Goal

Coverage measures how much code is executed.

It does not necessarily measure:

- Test quality
- Correct assertions
- Business correctness
- Edge-case coverage

Use coverage as one metric, not the only quality metric.

---

# Anti-Pattern: Using Production Data

Never copy sensitive production data directly into CI.

Instead use:

- Synthetic Data
- Sanitized Data
- Test Fixtures
- Mock Data

---

# Test Security

Protect:

- Test Credentials
- API Keys
- Database Passwords
- Tokens
- Private Certificates

Use GitHub Secrets or appropriate identity mechanisms.

Never print secrets in logs.

---

# Testing Secrets

Bad:

    echo $PASSWORD

Better:

    Use secret only where required.

Avoid commands that expose sensitive values.

---

# Testing Workflow Permissions

Use least privilege.

Example:

    permissions:
      contents: read

Additional permissions should only be granted when required.

---

# Test Execution Time

Track:

    Test Duration
        |
        ↓
    CI Duration

If tests take:

    30 minutes

the developer feedback loop becomes slow.

Optimize:

- Parallelization
- Caching
- Test Selection
- Test Isolation
- Faster Fixtures
- Database Setup

---

# CI Test Optimization

Example:

    Before:

    Unit → 10 min
    Integration → 20 min
    E2E → 30 min

    Total → 60 min

After parallelization:

    Unit --------\
    Integration ---+--> 30 min
    E2E ----------/

Actual optimization depends on dependencies and runner resources.

---

# Testing Pipeline Example

    name: CI Tests

    on:
      pull_request:
      push:
        branches:
          - main

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Unit Tests
            run: npm test -- --coverage

          - name: Build
            run: npm run build

          - name: Smoke Test
            run: |
              echo "Run smoke tests here"

---

# Complete DevSecOps Testing Pipeline

    name: DevSecOps CI

    on:
      pull_request:
      push:
        branches:
          - main

    permissions:
      contents: read

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Unit Tests
            run: npm test -- --coverage

          - name: Build Application
            run: npm run build

          - name: Docker Build
            run: |
              docker build \
                -t myapp:${{ github.sha }} \
                .

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'
              vuln-type: 'os,library'

---

# Recommended CI Testing Order

A practical order is:

    Checkout
       |
       ↓
    Install Dependencies
       |
       ↓
    Unit Tests
       |
       ↓
    Build
       |
       ↓
    Integration Tests
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
    Artifact Publishing

Depending on the project, some checks can run in parallel.

---

# Parallel Quality Checks

After build:

    Build
      |
      +-------- Unit Tests
      |
      +-------- SonarQube
      |
      +-------- Dependency Scan
      |
      +-------- Other Checks
      |
      ↓
    Quality Gate

Parallel execution can reduce total pipeline duration.

---

# Testing Gates

Example:

    Unit Tests
        |
        ↓
      Gate 1
        |
        ↓
    Integration Tests
        |
        ↓
      Gate 2
        |
        ↓
    SonarQube
        |
        ↓
      Gate 3
        |
        ↓
    Trivy
        |
        ↓
      Gate 4
        |
        ↓
    Publish

Each gate protects the next stage.

---

# Failure Handling

If unit tests fail:

    Unit Tests
       |
       ↓
     FAIL
       |
       X
    Publish

If Trivy fails:

    Trivy
       |
       ↓
     FAIL
       |
       X
    Publish

If tests pass:

    Tests
       |
       ↓
     PASS
       |
       ↓
    Continue

---

# Testing and Artifact Publishing

The relationship is:

    Test
      |
      ↓
    Pass
      |
      ↓
    Package
      |
      ↓
    Publish Artifact

Never publish an artifact that failed mandatory tests.

---

# Testing and Deployment

The relationship is:

    Source
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
    Artifact
      |
      ↓
    Deployment
      |
      ↓
    Smoke Test

This creates a controlled delivery pipeline.

---

# Interview Questions

## Basic

1. What is unit testing?
2. What is integration testing?
3. What is API testing?
4. What is end-to-end testing?
5. What is smoke testing?
6. What is regression testing?
7. What is test coverage?
8. What is the testing pyramid?
9. Why are unit tests usually faster?
10. Why are E2E tests usually slower?
11. What is a test artifact?
12. What is a flaky test?
13. What is shift-left testing?
14. What is shift-right testing?
15. What is a quality gate?

---

# Intermediate Interview Questions

16. How do you run unit tests in GitHub Actions?

17. How do you run integration tests?

18. How do you test a database-dependent application?

19. How do you use service containers?

20. How do you upload test reports?

21. How do you upload test reports even when tests fail?

22. How do you implement test coverage?

23. How do you define coverage thresholds?

24. How do you test Docker containers?

25. How do you implement smoke testing?

26. How do you run tests against multiple language versions?

27. How do you parallelize tests?

28. How do you handle flaky tests?

29. How do you reduce CI test execution time?

30. How do you block merging when tests fail?

---

# Advanced Interview Questions

31. Design a complete CI testing strategy for microservices.

32. How would you decide which tests should run on Pull Requests?

33. How would you design testing for QA, UAT, and Production?

34. How would you test a Docker image before pushing it to ECR?

35. How would you combine unit, integration, security, and E2E testing?

36. How would you reduce a 30-minute test suite to under 10 minutes?

37. How would you design parallel testing using GitHub Actions matrix jobs?

38. How would you handle flaky integration tests?

39. How would you test database migrations?

40. How would you test a microservices architecture?

41. How would you design post-deployment validation?

42. How would you prevent failed tests from reaching production?

43. How would you implement testing in a GitOps-based deployment pipeline?

44. How would you test a canary deployment?

45. How would you design enterprise CI quality gates?

---

# Scenario Question

## Your CI pipeline takes 30 minutes. How would you reduce it?

I would first identify the slowest stages.

Example:

    CI
      |
      +-- Build: 5 min
      +-- Unit: 3 min
      +-- Integration: 15 min
      +-- E2E: 7 min

Then optimize:

    Dependency Caching
        +
    Parallel Jobs
        +
    Test Sharding
        +
    Test Selection
        +
    Faster Test Setup

I would measure before and after each optimization.

---

# Scenario Question

## Unit tests pass but integration tests fail. What would you investigate?

I would check:

    1. Database
    2. Service Dependencies
    3. Environment Variables
    4. Network
    5. Test Data
    6. Database Migrations
    7. Credentials
    8. Container Health
    9. Port Configuration
    10. Logs

Unit tests validate isolated logic.

Integration tests validate component interactions.

---

# Scenario Question

## Your tests pass locally but fail in GitHub Actions. What would you check?

I would compare:

    Local Environment
          vs
    GitHub Runner

Check:

- Runtime Version
- Dependency Version
- OS
- Environment Variables
- Secrets
- Database
- Network
- Time Zone
- File Permissions
- Case Sensitivity
- Test Data
- Cached Dependencies

Then reproduce the CI environment locally where possible.

---

# Scenario Question

## One test randomly fails every few runs. What would you do?

I would classify it as potentially flaky.

I would:

    1. Reproduce the failure
    2. Inspect logs
    3. Identify shared state
    4. Check timing
    5. Check race conditions
    6. Check external dependencies
    7. Isolate test data
    8. Fix root cause
    9. Run repeatedly
    10. Confirm stability

I would not permanently hide the failure with retries.

---

# Scenario Question

## How would you test a microservices application?

I would use multiple layers:

    Service
       |
       +-- Unit
       |
       +-- Integration
       |
       +-- Contract
       |
       +-- API
       |
       +-- Security
       |
       ↓
    System
       |
       +-- E2E
       +-- Regression
       +-- Smoke

This provides coverage at both service and system levels.

---

# Scenario Question

## How would you prevent broken code from being merged?

I would configure required GitHub Actions checks.

Example:

    Pull Request
       |
       ↓
    Required Checks
       |
       +-- Build
       +-- Unit Tests
       +-- SonarQube
       +-- Trivy
       |
       ↓
    Branch Protection
       |
       +-- Pass → Merge
       |
       +-- Fail → Block

---

# Scenario Question

## How would you validate a deployment?

I would use:

    Deployment
       |
       ↓
    Pod / Service Health
       |
       ↓
    Health Endpoint
       |
       ↓
    Smoke Test
       |
       ↓
    API Test
       |
       ↓
    Metrics / Logs
       |
       ↓
    Success / Rollback

---

# Scenario Question

## How would you test Terraform changes in CI?

A typical approach:

    Terraform Code
        |
        ↓
    Format
        |
        ↓
    Validate
        |
        ↓
    Trivy Config
        |
        ↓
    Terraform Plan
        |
        ↓
    Review / Approval
        |
        ↓
    Apply

Testing IaC before applying reduces configuration-related risk.

---

# Scenario Question

## How would you test Kubernetes manifests?

I would validate:

    Kubernetes YAML
        |
        +-- YAML Validation
        +-- Schema Validation
        +-- Trivy Config Scan
        +-- Helm Template
        |
        ↓
    Deployment

Then post-deployment:

    Kubernetes
        |
        ↓
    Health Checks
        |
        ↓
    Smoke Tests

---

# Scenario Question

## How would you design testing for QA, UAT, and Production?

QA:

    Integration
    API
    Regression

UAT:

    E2E
    Business Validation
    Regression

Production:

    Smoke
    Health Checks
    Post-Deployment Validation

Example:

    CI
     |
     ↓
    QA
     |
     ↓
    UAT
     |
     ↓
    Production
     |
     ↓
    Smoke

---

# Best Practices Checklist

- Run unit tests on Pull Requests
- Keep unit tests fast
- Run integration tests where required
- Use service containers for dependencies
- Test Docker images after building
- Run security scans
- Generate test reports
- Preserve reports on failure
- Track test coverage
- Use realistic test data
- Keep tests isolated
- Avoid flaky tests
- Parallelize large test suites
- Cache dependencies carefully
- Use matrix testing when needed
- Use required status checks
- Block publishing when mandatory tests fail
- Validate deployments after release
- Use smoke tests after deployment
- Do not use production secrets in tests
- Do not use production data directly
- Measure CI execution time
- Fix root causes instead of hiding failures

---

# Important GitHub Actions Syntax

Pull Request:

    on:
      pull_request:

Push:

    on:
      push:
        branches:
          - main

Path filter:

    on:
      pull_request:
        paths:
          - 'src/**'

Matrix:

    strategy:
      matrix:
        node-version:
          - 18
          - 20
          - 22

Service container:

    services:
      postgres:
        image: postgres:16

Timeout:

    timeout-minutes: 10

Always upload results:

    if: always()

Optional non-blocking step:

    continue-on-error: true

Job dependency:

    needs: test

---

# Important Testing Tools

Java:

    JUnit
    Maven
    JaCoCo

Node.js:

    Jest
    Vitest
    npm

Python:

    pytest
    coverage.py

API:

    curl
    Postman / Newman
    REST clients

Performance:

    k6
    JMeter
    Gatling

Security:

    SonarQube
    Trivy

Container:

    Docker

CI:

    GitHub Actions

Deployment:

    ArgoCD
    Kubernetes
    EKS

---

# Testing Mental Model

Think about testing as multiple layers:

    Code
      |
      ↓
    Unit Tests
      |
      ↓
    Integration Tests
      |
      ↓
    API Tests
      |
      ↓
    Container Tests
      |
      ↓
    Security Tests
      |
      ↓
    E2E Tests
      |
      ↓
    Deployment
      |
      ↓
    Smoke Tests

Each layer answers a different question.

---

# What Does Each Test Answer?

Unit:

    "Does this piece of logic work?"

Integration:

    "Do these components work together?"

API:

    "Does this interface behave correctly?"

Container:

    "Does the built application start correctly?"

Security:

    "Are there known security issues?"

E2E:

    "Does the complete business flow work?"

Smoke:

    "Is the deployed application basically healthy?"

---

# Final CI Testing Flow

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
        +-- Dependency Install
        |
        +-- Unit Tests
        |
        +-- Integration Tests
        |
        +-- Coverage
        |
        +-- SonarQube
        |
        +-- Docker Build
        |
        +-- Trivy
        |
        ↓
    Quality / Security Gate
        |
        +-- Fail → Stop
        |
        +-- Pass
              |
              ↓
          Publish Artifact
              |
              ↓
             QA
              |
              ↓
          Integration / E2E
              |
              ↓
             UAT
              |
              ↓
          Production
              |
              ↓
          Smoke Test

Core idea:

A strong GitHub Actions testing strategy does not depend on one type of test. Fast unit tests provide quick feedback, integration and API tests validate component interactions, E2E tests validate important business flows, security tools such as SonarQube and Trivy provide additional quality and security checks, and post-deployment smoke tests verify that the deployed application is actually healthy. The objective is to catch failures as early as possible while keeping CI fast, reliable, and maintainable.