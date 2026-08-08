# GitHub Actions Matrix

A **matrix strategy** allows the same job to run multiple times with different combinations of configuration values.

Matrix is useful when you need to test or build the same application against:

- Multiple operating systems
- Multiple language versions
- Multiple runtime versions
- Multiple application versions
- Multiple configurations
- Multiple architectures

Instead of creating separate jobs manually, GitHub Actions generates the jobs automatically.

---

# Why Matrix?

Without matrix:

```yaml
jobs:

  test-node-20:
    ...

  test-node-22:
    ...

  test-node-24:
    ...
```

This creates duplicate workflow code.

With matrix:

```yaml
jobs:

  test:

    strategy:
      matrix:
        node-version:
          - 20
          - 22
          - 24
```

GitHub creates multiple job executions automatically.

Conceptually:

```text
Test Job

   |
   ├── Node 20
   ├── Node 22
   └── Node 24
```

---

# Basic Matrix Syntax

```yaml
jobs:

  test:

    runs-on: ubuntu-latest

    strategy:

      matrix:
        node-version:
          - 20
          - 22
          - 24

    steps:

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Test
        run: npm test
```

The matrix value is accessed through:

```yaml
${{ matrix.node-version }}
```

---

# Matrix Execution

For:

```yaml
matrix:
  node-version:
    - 20
    - 22
    - 24
```

GitHub creates:

```text
Test
 |
 ├── Node 20
 ├── Node 22
 └── Node 24
```

These matrix jobs can run in parallel depending on runner availability and workflow configuration.

---

# Matrix with Operating Systems

Example:

```yaml
jobs:

  test:

    strategy:

      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

    runs-on: ${{ matrix.os }}

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Test
        run: echo "Running on ${{ matrix.os }}"
```

Execution:

```text
Test
 |
 ├── Ubuntu
 ├── Windows
 └── macOS
```

---

# Matrix with Multiple Variables

A matrix can contain multiple dimensions.

Example:

```yaml
strategy:

  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node-version:
      - 20
      - 22
```

GitHub generates combinations.

```text
Ubuntu + Node 20
Ubuntu + Node 22
Windows + Node 20
Windows + Node 22
```

Total:

```text
2 × 2 = 4 jobs
```

---

# Matrix Combination Formula

If a matrix has:

```text
2 operating systems

×

3 Node versions
```

Then:

```text
2 × 3 = 6 combinations
```

Example:

```text
Ubuntu + Node 20
Ubuntu + Node 22
Ubuntu + Node 24
Windows + Node 20
Windows + Node 22
Windows + Node 24
```

This is important because matrix size can grow quickly.

---

# Matrix with Java

Example:

```yaml
jobs:

  test:

    strategy:

      matrix:
        java-version:
          - 17
          - 21

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java-version }}
          distribution: temurin

      - name: Test
        run: mvn test
```

Execution:

```text
Java 17
   |
   └── Test

Java 21
   |
   └── Test
```

---

# Matrix with Python

```yaml
jobs:

  test:

    strategy:

      matrix:
        python-version:
          - '3.10'
          - '3.11'
          - '3.12'

    runs-on: ubuntu-latest

    steps:

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Test
        run: pytest
```

---

# Matrix with Terraform Versions

Matrix can also be useful for compatibility testing.

Example:

```yaml
jobs:

  terraform-test:

    strategy:

      matrix:
        terraform-version:
          - '1.8'
          - '1.9'

    runs-on: ubuntu-latest

    steps:

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ matrix.terraform-version }}

      - name: Terraform Version
        run: terraform version

      - name: Validate
        run: terraform validate
```

---

# Matrix Values in Steps

Matrix values are available through the `matrix` context.

Example:

```yaml
run: echo "${{ matrix.node-version }}"
```

Another example:

```yaml
run: echo "OS: ${{ matrix.os }}"
```

Multiple values:

```yaml
run: |
  echo "OS: ${{ matrix.os }}"
  echo "Node: ${{ matrix.node-version }}"
```

---

# Matrix Include

`include` allows you to add or customize specific matrix combinations.

Example:

```yaml
strategy:

  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node-version:
      - 20
      - 22

    include:
      - os: ubuntu-latest
        node-version: 24
```

This adds:

```text
Ubuntu + Node 24
```

to the generated matrix.

---

# Include Additional Variables

`include` can also add additional values.

Example:

```yaml
strategy:

  matrix:

    environment:
      - qa
      - uat

    include:

      - environment: qa
        namespace: catalogue-qa

      - environment: uat
        namespace: catalogue-uat
```

Then:

```yaml
run: echo "${{ matrix.namespace }}"
```

This allows related configuration to travel with the matrix entry.

---

# Matrix Include for Environment Mapping

Example:

```yaml
strategy:

  matrix:

    include:

      - environment: qa
        namespace: catalogue-qa
        cluster: qa-cluster

      - environment: uat
        namespace: catalogue-uat
        cluster: uat-cluster
```

The workflow can use:

```yaml
${{ matrix.environment }}
```

```yaml
${{ matrix.namespace }}
```

```yaml
${{ matrix.cluster }}
```

---

# Matrix Exclude

`exclude` removes specific combinations.

Example:

```yaml
strategy:

  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node-version:
      - 20
      - 22

    exclude:

      - os: windows-latest
        node-version: 20
```

Generated combinations:

```text
Ubuntu + Node 20
Ubuntu + Node 22
Windows + Node 22
```

The excluded combination:

```text
Windows + Node 20
```

is not created.

---

# Include vs Exclude

Use:

```text
include
```

to add or customize combinations.

Use:

```text
exclude
```

to remove combinations.

Example:

```text
Matrix
 |
 ├── Include special combination
 |
 └── Exclude unsupported combination
```

---

# Matrix Fail-Fast

By default, matrix behavior can stop other in-progress or queued matrix jobs when a failure occurs.

You can control this with:

```yaml
strategy:

  fail-fast: false

  matrix:
    node-version:
      - 20
      - 22
      - 24
```

With:

```yaml
fail-fast: false
```

other matrix combinations continue even if one combination fails.

---

# When to Use fail-fast: false

Use it when you want complete compatibility information.

Example:

```text
Node 20 → PASS
Node 22 → FAIL
Node 24 → PASS
```

With `fail-fast: false`, you can see all results.

This is useful for:

- Compatibility testing
- Multi-version testing
- Release validation
- Cross-platform testing

---

# Production Consideration for fail-fast

For a production deployment matrix, blindly setting:

```yaml
fail-fast: false
```

may waste resources or allow unnecessary work after a critical failure.

Choose the behavior based on whether matrix jobs are:

```text
Independent validation
```

or:

```text
Required deployment gates
```

---

# max-parallel

Matrix jobs can be limited using:

```yaml
max-parallel:
```

Example:

```yaml
strategy:

  max-parallel: 2

  matrix:
    node-version:
      - 20
      - 22
      - 24
      - 25
```

Only two matrix jobs are allowed to execute concurrently.

Conceptually:

```text
Available:
4 jobs

max-parallel:
2

Execution:
Job 1 ───┐
Job 2 ───┘
          ↓
Job 3 ───┐
Job 4 ───┘
```

---

# Why Use max-parallel?

It can help control:

- Runner consumption
- Cloud costs
- External API load
- Test infrastructure load
- Database load
- Kubernetes cluster load

---

# Matrix with Different Test Suites

Example:

```yaml
strategy:

  matrix:

    test-suite:
      - unit
      - integration
      - e2e
```

The command can be selected using:

```yaml
run: ./run-tests.sh ${{ matrix.test-suite }}
```

Execution:

```text
Test Job
 |
 ├── Unit
 ├── Integration
 └── E2E
```

Use this carefully because E2E tests may have different infrastructure requirements from unit tests.

---

# Matrix and Job Outputs

Matrix jobs can produce outputs, but collecting multiple matrix results requires careful workflow design.

For example:

```text
Matrix
 |
 ├── Node 20
 ├── Node 22
 └── Node 24
       |
       ↓
Multiple Results
```

Do not assume that a matrix automatically creates one simple output containing all results.

For complex aggregation, use:

- Artifacts
- A dedicated aggregation job
- External storage
- Test reporting systems

---

# Matrix and needs

A matrix job can participate in a dependency graph.

Example:

```text
Build
  |
  ↓
Test Matrix
  |
  ├── Node 20
  ├── Node 22
  └── Node 24
  |
  ↓
Deploy
```

Conceptually:

```yaml
deploy:

  needs: test

  runs-on: ubuntu-latest

  steps:

    - name: Deploy
      run: ./deploy.sh
```

The downstream job depends on the matrix job.

---

# Matrix as a Quality Gate

Matrix testing can act as a compatibility gate.

Example:

```text
Build
  |
  ↓
Test Matrix
  |
  ├── Node 20 → PASS
  ├── Node 22 → PASS
  └── Node 24 → PASS
  |
  ↓
Deploy
```

If the matrix job is required and a combination fails, the downstream deployment should not proceed normally.

---

# Production Compatibility Pipeline

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: npm run build

  compatibility:

    needs: build

    strategy:

      fail-fast: false

      matrix:

        node-version:
          - 20
          - 22
          - 24

    runs-on: ubuntu-latest

    steps:

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Test
        run: npm test

  deploy:

    needs: compatibility

    runs-on: ubuntu-latest

    environment: production

    steps:

      - name: Deploy
        run: ./deploy.sh
```

Architecture:

```text
Build
  |
  ↓
Compatibility Matrix
  |
  ├── Node 20
  ├── Node 22
  └── Node 24
  |
  ↓
Production
```

---

# Matrix for Multi-Architecture Builds

Matrix can be used to build for different architectures.

Example concept:

```yaml
strategy:

  matrix:

    architecture:
      - amd64
      - arm64
```

Execution:

```text
Build
 |
 ├── amd64
 └── arm64
```

This can be useful for container platforms supporting multiple architectures.

For actual multi-architecture Docker builds, consider whether Docker Buildx is a better fit than independently building and publishing images.

---

# Matrix for Microservices

Matrix can be used when the same workflow logic applies to multiple services.

Example:

```yaml
strategy:

  matrix:

    service:
      - catalogue
      - user
      - cart
      - payment
      - inventory
```

Then:

```yaml
run: ./scripts/build-service.sh ${{ matrix.service }}
```

Execution:

```text
Application Build
 |
 ├── Catalogue
 ├── User
 ├── Cart
 ├── Payment
 └── Inventory
```

This can reduce duplicate workflow definitions.

---

# Microservices Matrix Architecture

```text
                 Application
                     |
                     ↓
                 Matrix Job
                     |
       ┌─────┬───────┼───────┬─────┐
       ↓     ↓       ↓       ↓     ↓
   Catalogue User   Cart  Payment Inventory
       |     |       |       |       |
       └─────┴───────┼───────┴───────┘
                     ↓
               Validation
                     |
                     ↓
                 Deployment
```

However, if services have significantly different build or deployment requirements, separate jobs or reusable workflows may be clearer.

---

# Matrix with Docker Images

Example:

```yaml
strategy:

  matrix:

    service:
      - catalogue
      - cart
      - payment

steps:

  - name: Build Image
    run: |
      docker build \
        -t ${{ matrix.service }}:${GITHUB_SHA::7} \
        ./${{ matrix.service }}
```

This can build multiple service images using the same workflow logic.

---

# Matrix and Artifact Names

When multiple matrix jobs produce artifacts, use unique names.

Example:

```yaml
- name: Upload Test Results
  uses: actions/upload-artifact@v4
  with:
    name: test-results-${{ matrix.node-version }}
    path: test-results/
```

This prevents different matrix executions from confusing their artifacts.

---

# Matrix and Environment Names

When using matrix environments:

```yaml
environment:
  name: ${{ matrix.environment }}
```

Example:

```yaml
strategy:

  matrix:

    environment:
      - qa
      - uat
```

Be very careful with production.

Do not automatically create a matrix that includes:

```text
qa
uat
prod
```

and then deploy all environments in parallel.

Production usually requires a controlled promotion path.

---

# Bad Production Matrix

Avoid blindly doing:

```text
Matrix
 |
 ├── QA
 ├── UAT
 └── Production
```

This could allow production deployment to happen independently.

A safer architecture is:

```text
Build
  |
  ↓
QA
  |
  ↓
UAT
  |
  ↓
Approval
  |
  ↓
Production
```

Matrix is better suited to parallel validation than uncontrolled environment promotion.

---

# Matrix vs Separate Jobs

Use matrix when:

```text
Same logic
+
Different values
```

Example:

```text
Same test
+
Node 20 / 22 / 24
```

Use separate jobs when:

```text
Different logic
+
Different responsibilities
```

Example:

```text
Build
Security
UAT
Production
```

---

# Matrix vs Reusable Workflow

Matrix:

```text
Same job logic
+
Multiple parameter combinations
```

Reusable workflow:

```text
Reusable multi-job pipeline
```

Example:

```text
Matrix
 |
 ├── Node 20
 ├── Node 22
 └── Node 24
```

Reusable workflow:

```text
Reusable CI
 |
 ├── Build Job
 ├── Test Job
 ├── Security Job
 └── Artifact Job
```

---

# Matrix Cost Considerations

Matrix can multiply resource consumption.

Example:

```text
3 OS
×
4 Runtime Versions
×
2 Architectures
```

Total:

```text
3 × 4 × 2 = 24 jobs
```

This can significantly increase:

- Runner usage
- Execution time
- Cost
- External service load

Before creating a matrix, calculate the possible combinations.

---

# Matrix Optimization

Use:

```text
include
exclude
max-parallel
fail-fast
```

to control matrix behavior.

Example:

```yaml
strategy:

  fail-fast: false

  max-parallel: 3

  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node-version:
      - 20
      - 22
      - 24

    exclude:

      - os: windows-latest
        node-version: 20
```

---

# Matrix and Concurrency

Concurrency can control overlapping workflow executions.

Example:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

This is different from:

```yaml
max-parallel:
```

Difference:

```text
max-parallel
→ controls matrix jobs within one matrix strategy

concurrency
→ controls overlapping workflow/job executions
```

---

# Matrix and Production Deployments

Matrix should generally not be used to bypass controlled production promotion.

Good use:

```text
Production Compatibility Testing

Matrix
 |
 ├── Kubernetes Version A
 ├── Kubernetes Version B
 └── Kubernetes Version C
```

Then:

```text
All Required Tests
       |
       ↓
Approval
       |
       ↓
Production
```

---

# Matrix for Kubernetes Testing

Example:

```yaml
strategy:

  matrix:

    kubernetes-version:
      - '1.30'
      - '1.31'
      - '1.32'
```

This can help test compatibility with supported Kubernetes versions.

Actual cluster provisioning strategy depends on the organization's infrastructure.

---

# Matrix for Helm Testing

Matrix can test different Kubernetes or Helm configurations.

Example:

```yaml
strategy:

  matrix:

    values-file:
      - values-qa.yaml
      - values-uat.yaml
```

Then:

```yaml
run: |
  helm template catalogue ./helm/catalogue \
    -f ./helm/catalogue/${{ matrix.values-file }}
```

This validates multiple configurations.

---

# Matrix for Terraform Validation

Example:

```yaml
strategy:

  matrix:

    terraform-version:
      - '1.8'
      - '1.9'

steps:

  - name: Setup Terraform
    uses: hashicorp/setup-terraform@v3
    with:
      terraform_version: ${{ matrix.terraform-version }}

  - name: Terraform Validate
    run: terraform validate
```

This is useful for compatibility testing.

---

# Matrix and Security Scanning

Matrix can scan multiple services.

Example:

```yaml
strategy:

  matrix:

    service:
      - catalogue
      - cart
      - payment
      - inventory
```

Then:

```yaml
- name: Scan Image
  run: trivy image "${{ matrix.service }}:${GITHUB_SHA::7}"
```

Architecture:

```text
             Security
                 |
       ┌─────────┼─────────┐
       ↓         ↓         ↓
   Catalogue    Cart    Payment
       |         |         |
       └─────────┼─────────┘
                 ↓
             Security Gate
```

---

# Matrix Failure Strategy

Suppose:

```text
Node 20 → PASS
Node 22 → FAIL
Node 24 → PASS
```

If this matrix represents a mandatory compatibility gate:

```text
Any required failure
        |
        ↓
Gate Failure
        |
        ↓
Production Blocked
```

Do not ignore the failed combination just because other combinations passed.

---

# Matrix Results Aggregation

For advanced pipelines, you may want one job to aggregate matrix results.

Architecture:

```text
             Test Matrix
                 |
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     Node 20   Node 22   Node 24
       |         |         |
       └─────────┼─────────┘
                 ↓
            Aggregation
                 |
                 ↓
              Deploy
```

Artifacts or test-reporting systems can be used to collect detailed results.

---

# Matrix Security

Matrix values may influence commands.

Avoid blindly constructing shell commands from untrusted matrix values.

For controlled values defined inside the workflow, the risk is lower.

Still prefer safe patterns:

```yaml
env:
  SERVICE: ${{ matrix.service }}

run: ./scripts/build-service.sh "$SERVICE"
```

rather than embedding dynamic values directly into complex shell expressions.

Validate values when appropriate.

---

# Matrix and Secrets

Be careful when matrix values select environments.

Avoid exposing production secrets to every matrix combination.

Bad design:

```text
Matrix
 |
 ├── QA → Production Secret
 ├── UAT → Production Secret
 └── Production → Production Secret
```

Prefer environment-specific secret access:

```text
QA
 |
 └── QA secrets

UAT
 |
 └── UAT secrets

Production
 |
 └── Production secrets
```

And keep production deployment separately controlled.

---

# Production Matrix Design Principle

Use matrix primarily for:

```text
Compatibility
Validation
Testing
Parallel Builds
Service Processing
Architecture Builds
```

Use controlled sequential jobs for:

```text
QA Promotion
UAT Promotion
Production Promotion
```

---

# Troubleshooting Matrix Jobs

When a matrix job fails:

```text
1. Identify failed matrix combination
2. Check OS
3. Check runtime version
4. Check matrix variables
5. Check included/excluded combinations
6. Check runner
7. Check step logs
8. Check external dependencies
```

Example:

```text
Test
 |
 ├── Node 20 → PASS
 ├── Node 22 → FAIL
 └── Node 24 → PASS
```

Investigate Node 22 specifically before changing the entire workflow.

---

# Scenario: Too Many Matrix Jobs

Problem:

```text
5 OS
×
5 Runtime Versions
×
4 Services
=
100 Jobs
```

This may be excessive.

Solutions:

```text
Reduce combinations

Use include/exclude

Use max-parallel

Separate critical compatibility tests

Run extended tests on schedule

Keep PR tests lightweight
```

---

# Scenario: Production Runs Multiple Times

If production is accidentally part of a matrix:

```text
Matrix
 |
 ├── QA
 ├── UAT
 └── PROD
```

Redesign:

```text
Build
  |
  ↓
Matrix Validation
  |
  ↓
QA
  |
  ↓
UAT
  |
  ↓
Approval
  |
  ↓
PROD
```

Production should be a controlled promotion stage.

---

# Scenario: One Matrix Combination Fails

Example:

```text
Ubuntu + Node 20 → PASS
Ubuntu + Node 22 → PASS
Windows + Node 20 → FAIL
Windows + Node 22 → PASS
```

Check whether:

```text
Windows + Node 20
```

is actually supported.

If it is not supported, use `exclude`.

If it should work, investigate the specific compatibility issue.

---

# Scenario: Matrix Is Too Slow

Possible causes:

```text
Too many combinations

Slow tests

Limited runners

max-parallel too low

External service bottlenecks
```

Optimization:

```text
Reduce unnecessary combinations
       +
Increase appropriate parallelism
       +
Optimize tests
       +
Use caching
```

---

# Best Practices

- Use matrix when the same logic must run with different values.
- Calculate the number of combinations before implementation.
- Use `include` for special combinations.
- Use `exclude` for unsupported combinations.
- Use `fail-fast: false` when complete compatibility results are required.
- Use `max-parallel` to control resource consumption.
- Use unique artifact names for matrix jobs.
- Use controlled matrix values.
- Validate matrix values before using them in shell commands.
- Do not expose production secrets unnecessarily.
- Prefer matrix for testing and parallel processing.
- Keep production promotion controlled and sequential.
- Use artifacts or aggregation jobs for collecting matrix results.
- Monitor runner and infrastructure consumption.

---

# Common Mistakes

- Creating an unnecessarily large matrix.
- Forgetting that combinations multiply.
- Using matrix for unrelated deployment stages.
- Deploying QA, UAT, and Production in parallel.
- Not using `exclude` for unsupported combinations.
- Not controlling matrix parallelism.
- Using non-unique artifact names.
- Assuming matrix jobs automatically share files.
- Passing unvalidated matrix values into shell commands.
- Exposing production secrets to every matrix job.
- Ignoring failed matrix combinations.
- Using matrix when separate jobs would be clearer.

---

# Summary

A matrix allows the same job to execute across multiple configurations.

Basic:

```yaml
strategy:

  matrix:
    node-version:
      - 20
      - 22
      - 24
```

Multiple dimensions:

```yaml
strategy:

  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node-version:
      - 20
      - 22
```

This creates:

```text
Ubuntu + Node 20
Ubuntu + Node 22
Windows + Node 20
Windows + Node 22
```

Important matrix controls:

```text
matrix
include
exclude
fail-fast
max-parallel
```

Use matrix for:

```text
Testing
Compatibility
Parallel Builds
Multiple Services
Multiple Architectures
```

Avoid using matrix as an uncontrolled production promotion mechanism.

A strong production architecture is:

```text
Build
  |
  ↓
Matrix Validation
  |
  ├── Runtime A
  ├── Runtime B
  └── Runtime C
  |
  ↓
QA
  |
  ↓
UAT
  |
  ↓
Approval
  |
  ↓
Production
```

The key principle is:

```text
Matrix is for controlled parallelism.
Production promotion should remain controlled.
```

---

# Interview Questions

## Basic

1. What is a matrix strategy in GitHub Actions?
2. Why would you use a matrix?
3. How do you define a matrix?
4. How do you access a matrix value?
5. What is `include`?
6. What is `exclude`?
7. What is `fail-fast`?
8. What is `max-parallel`?

## Intermediate

9. How do you test multiple Node.js versions using a matrix?
10. How do you test multiple operating systems?
11. How do multiple matrix dimensions work?
12. How do you calculate the number of matrix jobs?
13. How do you exclude unsupported combinations?
14. How do you add custom values using `include`?
15. What is the difference between `max-parallel` and concurrency?
16. How do you upload unique artifacts from matrix jobs?

## Advanced

17. Design a matrix for testing a Java application against Java 17 and 21 on Ubuntu and Windows.
18. A matrix generates 60 jobs and makes the pipeline expensive. Explain how you would optimize it.
19. Design a microservices workflow that builds five services using a matrix and then performs a centralized validation stage.
20. Explain how you would use `include` and `exclude` to handle special compatibility combinations.
21. A matrix contains QA, UAT, and Production environments and accidentally deploys all three in parallel. Explain how you would redesign the workflow.
22. Design a production compatibility pipeline where Node 20, 22, and 24 are tested in parallel, but production deployment occurs only after all required tests pass.
23. Explain how `fail-fast: false` affects a compatibility testing pipeline.
24. Explain how you would control runner consumption using `max-parallel`.
25. Design a multi-architecture container build using a matrix and explain the limitations of using a matrix versus Docker Buildx.
26. Explain how you would safely handle matrix values that are later passed into shell commands.
27. Explain why production secrets should not be exposed to every matrix combination.
28. Design a matrix-based security scan for multiple microservices and make the results a mandatory production gate.