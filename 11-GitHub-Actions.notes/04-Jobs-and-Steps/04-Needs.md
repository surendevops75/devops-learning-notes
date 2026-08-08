# GitHub Actions Needs

`needs` defines dependencies between jobs.

By default, independent jobs can run in parallel.

When one job must wait for another job to complete, use:

```yaml
needs:
```

Basic example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Build"

  test:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: echo "Test"

  deploy:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploy"
```

Execution:

```text
Build
  |
  ↓
Test
  |
  ↓
Deploy
```

---

# Why `needs` Is Important

`needs` creates a dependency graph between jobs.

Without `needs`:

```text
Build       Test       Security       Deploy
  |           |            |             |
  └───────────┴────────────┴─────────────┘
              Parallel
```

With `needs`:

```text
Build
  |
  ↓
Test
  |
  ↓
Deploy
```

This is essential when later stages depend on earlier stages.

---

# Basic Syntax

Single dependency:

```yaml
test:
  needs: build
```

Equivalent list form:

```yaml
test:
  needs:
    - build
```

Multiple dependencies:

```yaml
deploy:
  needs:
    - test
    - security
```

---

# Single Job Dependency

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:
      - run: echo "Building"

  test:

    needs: build

    runs-on: ubuntu-latest

    steps:
      - run: echo "Testing"
```

Execution:

```text
Build
  |
  ↓
Test
```

The `test` job waits for `build`.

---

# Multiple Dependencies

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

  test:
    needs: build
    runs-on: ubuntu-latest

  security:
    needs: build
    runs-on: ubuntu-latest

  deploy:
    needs:
      - test
      - security
    runs-on: ubuntu-latest
```

Execution:

```text
             Build
               |
        ┌──────┴──────┐
        ↓             ↓
      Test         Security
        |             |
        └──────┬──────┘
               ↓
             Deploy
```

`test` and `security` can run in parallel.

`deploy` waits for both.

---

# Dependency Graph

The GitHub Actions workflow can be viewed as a Directed Acyclic Graph (DAG).

Example:

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
        Test    Security    Lint
          |        |        |
          └────────┼────────┘
                   ↓
                 Deploy
```

This structure allows:

- Parallel execution
- Dependency enforcement
- Faster pipelines
- Clear failure propagation

---

# Parallelism with needs

`needs` does not mean every job must run sequentially.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

  unit-test:
    needs: build
    runs-on: ubuntu-latest

  sonar:
    needs: build
    runs-on: ubuntu-latest

  trivy:
    needs: build
    runs-on: ubuntu-latest

  deploy:
    needs:
      - unit-test
      - sonar
      - trivy
    runs-on: ubuntu-latest
```

Execution:

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
      Unit Test  SonarQube  Trivy
          |        |        |
          └────────┼────────┘
                   ↓
                 Deploy
```

This is a strong production CI/CD pattern.

---

# needs vs Steps

Steps within a job:

```text
Job
 |
 ├── Step 1
 ├── Step 2
 └── Step 3
```

Jobs using `needs`:

```text
Job A
  |
  ↓
Job B
  |
  ↓
Job C
```

Important difference:

```text
Steps
→ execute inside the same job

needs
→ controls dependencies between jobs
```

---

# Job Failure and needs

Suppose:

```yaml
deploy:
  needs: security
```

If:

```text
Security
   |
   ↓
Failed
```

then:

```text
Deploy
   |
   X
Skipped
```

By default, a dependent job does not continue after a required dependency fails.

---

# Failure Propagation

Example:

```text
Build
  |
  ↓
Security
  |
  ↓
FAILED
  |
  ↓
Deploy
  |
  X
Skipped
```

This is useful for production security gates.

For example:

```text
Trivy Scan
   |
   ↓
Critical Vulnerability
   |
   ↓
Security Job Failed
   |
   ↓
Production Deployment Blocked
```

---

# Multiple Dependency Failure

Suppose:

```yaml
deploy:
  needs:
    - test
    - security
    - uat
```

Execution:

```text
Test ─────────── SUCCESS
Security ─────── FAILED
UAT ──────────── SUCCESS
                    |
                    ↓
                  Deploy
                    X
```

Because a required dependency failed, the deployment does not proceed normally.

---

# needs and if

A dependent job can use an `if` condition.

Example:

```yaml
deploy:

  needs:
    - test
    - security

  if: ${{ success() }}

  runs-on: ubuntu-latest

  steps:
    - run: ./deploy.sh
```

The deployment proceeds only when the required dependencies succeed.

---

# always() with needs

`always()` can change the default dependency behavior.

Example:

```yaml
diagnostics:

  needs:
    - test
    - security

  if: ${{ always() }}

  runs-on: ubuntu-latest

  steps:
    - name: Collect Diagnostics
      run: ./collect-diagnostics.sh
```

This can be useful when you want diagnostics to run even when a dependency failed.

Use carefully.

Do not use:

```yaml
if: ${{ always() }}
```

to bypass mandatory production security or approval gates.

---

# failure() with needs

A diagnostic job can run when an upstream dependency fails.

Example:

```yaml
failure-diagnostics:

  needs:
    - test
    - security

  if: ${{ failure() }}

  runs-on: ubuntu-latest

  steps:
    - name: Collect Logs
      run: ./collect-logs.sh
```

Conceptual flow:

```text
Test ───────── SUCCESS
Security ───── FAILED
                  |
                  ↓
             Diagnostics
```

---

# Conditional Deployment

A deployment can depend on several jobs.

```yaml
deploy:

  needs:
    - build
    - test
    - security
    - uat

  if: ${{ success() }}

  runs-on: ubuntu-latest

  steps:

    - name: Deploy
      run: ./deploy.sh
```

Flow:

```text
Build ────────┐
Test ─────────┤
Security ─────┼──→ Deploy
UAT ──────────┘
```

All required jobs must succeed.

---

# Production CI/CD Pattern

A production-grade workflow can use:

```text
Build
  |
  ├── Unit Tests
  ├── SonarQube
  ├── Trivy
  └── Integration Tests
          |
          ↓
        Artifact
          |
          ↓
         UAT
          |
          ↓
       E2E Tests
          |
          ↓
       Approval
          |
          ↓
      Production
```

The dependencies should reflect the actual release gates.

---

# Build Job

Example:

```yaml
build:

  runs-on: ubuntu-latest

  steps:

    - name: Checkout
      uses: actions/checkout@v4

    - name: Build
      run: mvn clean package
```

---

# Test Job

```yaml
test:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Run Tests
      run: mvn test
```

---

# Security Job

```yaml
security:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Trivy Scan
      run: trivy image "$IMAGE"
```

---

# UAT Job

```yaml
uat:

  needs:
    - test
    - security

  runs-on: ubuntu-latest

  steps:

    - name: Deploy to UAT
      run: ./deploy-uat.sh

    - name: Run E2E Tests
      run: ./e2e-tests.sh
```

---

# Production Job

```yaml
production:

  needs:
    - build
    - test
    - security
    - uat

  runs-on: ubuntu-latest

  environment:
    name: production

  steps:

    - name: Deploy Production
      run: ./deploy-prod.sh
```

Architecture:

```text
              Build
                |
        ┌───────┴────────┐
        ↓                ↓
      Test            Security
        |                |
        └───────┬────────┘
                ↓
               UAT
                |
                ↓
            Production
```

---

# Production Approval

`needs` establishes job dependency.

Environment protection can establish production approval.

Together:

```text
Validation Jobs
      |
      ↓
Production Job
      |
      ↓
Production Environment
      |
      ↓
Required Approval
      |
      ↓
Deploy
```

Important:

```text
needs
=
Dependency

environment protection
=
Deployment Protection
```

They solve different problems.

---

# needs with Environments

Example:

```yaml
deploy-prod:

  needs:
    - test
    - security
    - uat

  environment:
    name: production

  runs-on: ubuntu-latest

  steps:
    - name: Deploy
      run: ./deploy.sh
```

Execution:

```text
Tests
  |
  ↓
Security
  |
  ↓
UAT
  |
  ↓
Deploy Job
  |
  ↓
Production Environment
  |
  ↓
Approval
  |
  ↓
Deployment
```

---

# needs and Artifacts

Jobs do not automatically share files.

Example:

```text
Build
  |
  ↓
Upload Artifact
  |
  ↓
Test
  |
  ↓
Download Artifact
```

Dependencies and artifacts solve different problems.

```text
needs
→ controls execution order

artifact
→ transfers files
```

---

# Example: Build Artifact

```yaml
build:

  runs-on: ubuntu-latest

  steps:

    - name: Build
      run: mvn clean package

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: application
        path: target/*.jar
```

Deployment:

```yaml
deploy:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Download Artifact
      uses: actions/download-artifact@v4
      with:
        name: application

    - name: Deploy
      run: ./deploy.sh
```

Here:

```text
needs
→ waits for Build

artifact
→ transfers application
```

---

# needs and Job Outputs

`needs` also provides access to outputs from dependent jobs.

Example:

```yaml
build:

  runs-on: ubuntu-latest

  outputs:
    image_tag: ${{ steps.image.outputs.tag }}

  steps:

    - id: image
      run: echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
```

Another job:

```yaml
deploy:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Display Image Tag
      run: echo "${{ needs.build.outputs.image_tag }}"
```

Architecture:

```text
Build
 |
 └── Output: image_tag
       |
       ↓
Deploy
 |
 └── needs.build.outputs.image_tag
```

---

# needs Context

The `needs` context provides information about dependent jobs.

For example:

```yaml
${{ needs.build.result }}
```

Possible result values include:

```text
success
failure
cancelled
skipped
```

Example:

```yaml
- name: Show Build Result
  run: echo "${{ needs.build.result }}"
```

This is useful when designing advanced workflow logic.

---

# Checking Dependency Result

Example:

```yaml
diagnostics:

  needs:
    - build
    - security

  if: ${{ failure() }}

  runs-on: ubuntu-latest

  steps:

    - name: Show Results
      run: |
        echo "Build: ${{ needs.build.result }}"
        echo "Security: ${{ needs.security.result }}"
```

This can help troubleshoot complex pipelines.

---

# Complex Dependency Graph

A production workflow may look like:

```text
                         Build
                           |
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Unit Test     SonarQube       Trivy
             |             |             |
             └─────────────┼─────────────┘
                           ↓
                     Integration
                           |
                           ↓
                          UAT
                           |
                     ┌─────┴─────┐
                     ↓           ↓
                   E2E        Approval
                     |           |
                     └─────┬─────┘
                           ↓
                      Production
```

This gives the organization a clear release path.

---

# Avoid Unnecessary Dependencies

Bad:

```yaml
test:
  needs: build

security:
  needs: test
```

If security does not actually require test results, this unnecessarily makes security wait for tests.

Better:

```yaml
test:
  needs: build

security:
  needs: build
```

Then:

```yaml
deploy:
  needs:
    - test
    - security
```

Execution:

```text
          Build
            |
       ┌────┴────┐
       ↓         ↓
     Test     Security
       |         |
       └────┬────┘
            ↓
          Deploy
```

This is faster while preserving required dependencies.

---

# Dependency Optimization

When optimizing a pipeline, ask:

```text
Does Job B actually require Job A?

        |
       YES
        ↓
     needs: A

        |
       NO
        ↓
    Run in Parallel
```

This is an important technique for reducing CI/CD execution time.

---

# Bad Sequential Pipeline

```text
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
Deploy
```

If Test, SonarQube, and Trivy only require the build artifact:

```text
             Build
               |
       ┌───────┼────────┐
       ↓       ↓        ↓
      Test   SonarQube  Trivy
       |       |        |
       └───────┼────────┘
               ↓
             Deploy
```

The optimized graph can reduce pipeline duration.

---

# Dependency Design Principle

Do not add `needs` just to create a visually sequential pipeline.

Add `needs` when there is a real dependency.

Good:

```text
Build → Test
```

because Test requires the build.

Good:

```text
Build → Security
```

if Security requires the built artifact.

Good:

```text
Test + Security → Deploy
```

because Deploy requires both validation results.

---

# needs and Production Security

A mandatory security gate should be included in the production dependency graph.

Example:

```yaml
deploy:

  needs:
    - build
    - security
    - uat
```

Architecture:

```text
Build
  |
  ↓
Security
  |
  ↓
UAT
  |
  ↓
Production
```

If security fails:

```text
Security
   |
   ↓
FAILED
   |
   ↓
Production
   X
```

Do not use `always()` on the deployment job simply to bypass this protection.

---

# needs and Change Management

For an enterprise production deployment:

```text
Build
  |
  ↓
Tests
  |
  ↓
Security
  |
  ↓
UAT
  |
  ↓
Change Request
  |
  ↓
Approval
  |
  ↓
Production
```

The GitHub Actions dependency graph can enforce technical gates.

External change-management validation can enforce organizational approval.

Both should be respected.

---

# Production Deployment Example

```yaml
production:

  needs:
    - build
    - unit-tests
    - security
    - uat

  if: ${{ success() }}

  environment:
    name: production

  runs-on:
    - self-hosted
    - linux
    - production

  concurrency:
    group: production
    cancel-in-progress: false

  timeout-minutes: 30

  permissions:
    contents: read

  steps:

    - name: Deploy
      run: ./deploy-prod.sh

    - name: Verify Deployment
      run: ./smoke-test.sh
```

This combines:

```text
needs
if
environment
runner
concurrency
timeout
permissions
```

---

# Failure Diagnostics

Use a separate diagnostic job when necessary.

Example:

```yaml
diagnostics:

  needs:
    - build
    - test
    - security
    - deploy

  if: ${{ failure() }}

  runs-on: ubuntu-latest

  steps:

    - name: Collect Diagnostics
      run: |
        echo "Collecting failure diagnostics"
        ./scripts/collect-diagnostics.sh
```

The exact dependency structure should be designed so that the diagnostic job can execute when the relevant upstream job fails.

---

# needs and Rollback

For production deployment:

```text
Validation
    |
    ↓
Production Deploy
    |
    ↓
Smoke Test
    |
    ├── Success
    |
    └── Failure
          |
          ↓
       Rollback
```

A rollback job can depend on the deployment and validation results.

Example concept:

```yaml
rollback:

  needs:
    - deploy
    - smoke-test

  if: ${{ failure() }}

  runs-on: ubuntu-latest

  steps:

    - name: Rollback
      run: ./rollback.sh
```

Rollback logic must be designed carefully so that it cannot accidentally roll back a successful deployment because of an unrelated failure.

---

# needs with Matrix Jobs

A matrix job can also participate in dependencies.

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
```

The deployment waits for the required matrix job to complete according to the workflow dependency behavior.

Matrix strategy is covered separately in:

```text
06-Matrix.md
```

---

# Troubleshooting needs

When a job is unexpectedly skipped:

```text
1. Check the Job ID
2. Check the needs value
3. Check dependency status
4. Check if conditions
5. Check environment protection
6. Check workflow graph
7. Check job results
```

Example:

```text
Deploy
  |
  ↓
Skipped
```

Check:

```text
Deploy needs:
   |
   ├── Build → success
   ├── Test → success
   └── Security → failure
```

The dependency failure explains the skipped deployment.

---

# Scenario 1: Deploy Runs Too Early

Problem:

```text
Build
  |
  ↓
Deploy

Security
  |
  ↓
Running
```

Cause:

```yaml
deploy:
  needs: build
```

Security was not included.

Fix:

```yaml
deploy:
  needs:
    - build
    - security
```

---

# Scenario 2: Pipeline Is Too Slow

Problem:

```text
Build
  |
  ↓
Test
  |
  ↓
Security
  |
  ↓
Deploy
```

If Security does not require Test:

```yaml
security:
  needs: build
```

Then:

```text
       Build
         |
    ┌────┴────┐
    ↓         ↓
  Test     Security
    |         |
    └────┬────┘
         ↓
       Deploy
```

This enables parallel execution.

---

# Scenario 3: Deployment Happens After Failed Security

Check whether the deployment uses:

```yaml
needs:
  - security
```

and whether the deployment has an inappropriate condition such as:

```yaml
if: ${{ always() }}
```

If Security is a mandatory gate, do not bypass its failure.

---

# Scenario 4: Diagnostic Job Does Not Run

If diagnostics should run after failures, check:

```yaml
if: ${{ failure() }}
```

or:

```yaml
if: ${{ always() }}
```

depending on the intended behavior.

Also verify that the diagnostic job's `needs` includes the jobs whose results it needs to observe.

---

# Scenario 5: Production Job Starts But Approval Is Missing

Remember:

```text
needs
```

does not create a human approval by itself.

Use:

```yaml
environment:
  name: production
```

with appropriate environment protection rules.

Architecture:

```text
needs
  |
  ↓
Validation Complete
  |
  ↓
Production Job
  |
  ↓
Environment Protection
  |
  ↓
Human Approval
```

---

# Best Practices

- Use `needs` only for real dependencies.
- Use parallel jobs when dependencies allow.
- Keep production dependencies explicit.
- Include mandatory security gates in deployment dependencies.
- Use artifacts for files and `needs` for execution order.
- Use job outputs with `needs` for controlled data transfer.
- Use environment protection for human production approvals.
- Avoid `always()` on sensitive deployment jobs.
- Use `failure()` or `always()` for diagnostics where appropriate.
- Keep dependency graphs simple and understandable.
- Optimize unnecessary sequential dependencies.
- Validate the complete dependency graph before production use.

---

# Common Mistakes

- Forgetting to add a required job to `needs`.
- Adding unnecessary dependencies and making the pipeline slower.
- Confusing `needs` with artifact transfer.
- Assuming `needs` provides human approval.
- Using `always()` to bypass production gates.
- Not checking dependency results.
- Creating overly complicated dependency graphs.
- Allowing deployment to run independently of security validation.
- Ignoring skipped jobs caused by failed dependencies.

---

# Summary

`needs` controls the dependency relationship between jobs.

Basic:

```yaml
test:
  needs: build
```

Multiple dependencies:

```yaml
deploy:
  needs:
    - test
    - security
    - uat
```

A good CI/CD dependency graph can combine parallelism and controlled sequencing:

```text
                 Build
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
        Test    SonarQube  Trivy
          |        |        |
          └────────┼────────┘
                   ↓
                  UAT
                   |
                   ↓
              Production
```

The key principle is:

```text
Use needs to enforce real job dependencies,
not to make every job run sequentially.
```

For production:

```text
Validation
    |
    ↓
Security
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

`needs` provides the technical dependency.

Environment protection provides the deployment approval control.

Artifacts provide file transfer.

Job outputs provide controlled data transfer.

Together, these mechanisms form the foundation of a production-grade GitHub Actions pipeline.

---

# Interview Questions

## Basic

1. What is `needs` in GitHub Actions?
2. Why is `needs` used?
3. How do you make one job wait for another?
4. Can a job have multiple dependencies?
5. What happens when a required dependency fails?

## Intermediate

6. What is the difference between `needs` and steps?
7. How does `needs` affect parallel execution?
8. How do you create a dependency on multiple jobs?
9. How can you access a dependency's result?
10. How can you use job outputs through `needs`?
11. What is the difference between `needs` and artifacts?
12. How does `if` work with `needs`?
13. When would you use `failure()`?
14. When would you use `always()`?

## Advanced

15. Design a CI pipeline where Build runs first, Unit Tests, SonarQube, and Trivy run in parallel, and Production waits for all validation jobs.
16. A production deployment starts before the Trivy scan finishes. Explain how you would troubleshoot the dependency graph.
17. A pipeline takes 30 minutes because every job waits for the previous job. Explain how you would optimize the `needs` relationships.
18. A security job fails but production still deploys. Explain the possible causes and how you would prevent this.
19. Explain the difference between `needs`, artifacts, job outputs, and environment protection.
20. Design a production pipeline with Build, parallel security checks, UAT, E2E testing, approval, production deployment, smoke testing, and rollback.
21. How would you design a failure-diagnostics job that runs when any mandatory validation job fails?
22. How would you prevent `always()` from accidentally bypassing production security gates?
23. A deployment job is skipped even though the build succeeded. Explain how you would investigate all dependency and condition-related causes.
24. Explain how you would use `needs` to implement a "build once, promote many" deployment strategy.
25. Design a dependency graph for a microservices platform where multiple services build in parallel, validation runs in parallel, and deployment occurs only after all required checks succeed.