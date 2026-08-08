# What is an Action?

A GitHub Action is a reusable unit of code that performs a specific task inside a GitHub Actions workflow.

Actions are similar to **plugins in Jenkins**.

Instead of writing every command from scratch, we can reuse existing Actions for common tasks such as:

```text
Checkout code
Setup Java
Setup Node.js
Setup Terraform
Login to AWS
Build Docker images
Upload artifacts
Deploy applications
```

---

# Action vs Workflow

A **workflow** defines the complete automation process.

An **Action** performs a specific reusable task inside that workflow.

Example:

```text
Workflow
   |
   ├── Checkout Code
   |       └── Action
   |
   ├── Setup Java
   |       └── Action
   |
   ├── Build
   |       └── Shell Command
   |
   ├── Security Scan
   |       └── Action / Tool
   |
   └── Upload Artifact
           └── Action
```

---

# Basic Example

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Build
        run: mvn clean package
```

Here:

```yaml
uses: actions/checkout@v4
```

is an Action.

And:

```yaml
uses: actions/setup-java@v4
```

is another Action.

---

# `uses`

The `uses` keyword tells GitHub Actions to execute a reusable Action.

Example:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

Structure:

```text
uses:
  owner/repository@version
```

Example:

```text
actions/checkout@v4
```

means:

```text
Owner       → actions
Repository  → checkout
Version     → v4
```

---

# Action Reference Format

General format:

```yaml
uses: OWNER/REPOSITORY@REF
```

Example:

```yaml
uses: actions/checkout@v4
```

Another:

```yaml
uses: actions/setup-node@v4
```

Another:

```yaml
uses: actions/upload-artifact@v4
```

---

# Action Inputs

Actions can accept inputs.

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
```

Here:

```text
distribution
java-version
```

are inputs to the Action.

---

# Action Outputs

Actions can also expose outputs.

Conceptually:

```text
Action
   |
   ├── Input
   |
   ├── Processing
   |
   └── Output
```

A later step can consume an output when the Action exposes one.

Example concept:

```yaml
- name: Get Version
  id: version
  uses: ./actions/get-version

- name: Display Version
  run: echo "${{ steps.version.outputs.version }}"
```

---

# Action `id`

A step can have an `id`.

Example:

```yaml
- name: Get Version
  id: version
  uses: ./actions/get-version
```

The `id` allows later workflow expressions to reference the step.

Example:

```yaml
${{ steps.version.outputs.version }}
```

---

# Action vs `run`

GitHub Actions steps commonly use either:

```yaml
uses:
```

or:

```yaml
run:
```

Example:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

versus:

```yaml
- name: Build
  run: mvn clean package
```

---

# `uses` vs `run`

### `uses`

Uses a reusable Action:

```yaml
- uses: actions/checkout@v4
```

### `run`

Executes a shell command:

```yaml
- run: npm install
```

Conceptually:

```text
uses → Reusable Action

run  → Command
```

---

# Example Pipeline

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install Dependencies
        run: npm ci

      - name: Test
        run: npm test

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: application
          path: dist/
```

Actions:

```text
checkout
setup-node
upload-artifact
```

Commands:

```text
npm ci
npm test
```

---

# Types of Actions

GitHub Actions supports different implementation approaches.

Common categories include:

```text
JavaScript Actions
Docker container Actions
Composite Actions
```

These are covered in detail in the following files:

```text
03-Composite-Actions.md
04-Docker-Actions.md
05-JavaScript-Actions.md
```

---

# Marketplace Actions

GitHub provides a Marketplace where users can discover reusable Actions.

Examples of common categories:

```text
AWS
Azure
Docker
Kubernetes
Terraform
Security
Testing
Deployment
Notifications
```

Marketplace coverage:

```text
02-Marketplace.md
```

---

# Local Actions

An Action does not have to come from the Marketplace.

You can create your own Action inside your repository.

Example:

```text
repository
 |
 ├── .github/
 │    └── workflows/
 │
 └── actions/
      └── build/
           └── action.yml
```

Workflow:

```yaml
- name: Build Application
  uses: ./actions/build
```

---

# Repository Action

Example:

```yaml
- name: Custom Action
  uses: ./actions/custom-action
```

This is a local Action stored in the repository.

---

# Remote Action

Example:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

The Action is maintained outside your repository.

---

# Version Pinning

Actions should be referenced using a deliberate version strategy.

Example:

```yaml
uses: actions/checkout@v4
```

A version tag is commonly used.

For high-security environments, organizations may prefer stronger pinning strategies such as a specific commit SHA, with a process for updating and reviewing the pin.

Example:

```yaml
uses: actions/checkout@<commit-sha>
```

This provides stronger immutability than a moving tag.

---

# Why Pin Actions?

Suppose:

```yaml
uses: some-org/some-action@main
```

The referenced code can change over time.

This can create:

```text
Unexpected Behavior
Security Risk
Build Reproducibility Problems
```

A deliberate versioning strategy provides better control.

---

# Production Recommendation

For production workflows:

```text
Avoid blindly using:
@main
```

Prefer:

```text
Reviewed version
```

or, for higher assurance:

```text
Reviewed commit SHA
```

Organizations should define their own dependency update policy.

---

# Action Permissions

Actions operate within the permissions available to the workflow.

Example:

```yaml
permissions:
  contents: read
```

This gives the workflow read access to repository contents.

For a deployment workflow, additional permissions may be required.

Example:

```yaml
permissions:
  contents: read
  id-token: write
```

This can be used for OIDC-based cloud authentication where configured.

---

# Least Privilege

Do not give every workflow:

```yaml
permissions: write-all
```

Instead, specify only what is required.

Example:

```yaml
permissions:
  contents: read
```

Production workflows should use the minimum required permissions.

---

# Action Security

An Action executes code as part of your workflow.

Therefore:

```text
Third-Party Action
       |
       ↓
Workflow Runner
       |
       ↓
Repository / Cloud / Secrets
```

Treat third-party Actions as dependencies.

Before using an Action, review:

```text
Source repository
Maintainer
Version
Permissions
Recent activity
Security history
Inputs
Outputs
Dependencies
```

---

# Third-Party Action Risk

Example:

```yaml
- name: Some Action
  uses: unknown-org/action@main
```

The Action may execute code with the permissions available to the job.

Therefore:

```text
Do not blindly trust Actions.
```

---

# Action Supply Chain Security

A production pipeline has a software supply chain:

```text
Developer
    |
    ↓
GitHub Repository
    |
    ↓
Workflow
    |
    ↓
Third-Party Actions
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Deployment
```

Every Action becomes part of that supply chain.

---

# Action Review Checklist

Before introducing a third-party Action:

```text
☐ Verify repository owner
☐ Review source code
☐ Check release history
☐ Check latest maintained version
☐ Review permissions
☐ Review dependencies
☐ Avoid unnecessary secrets
☐ Pin version appropriately
☐ Test in non-production
☐ Review security advisories
```

---

# Marketplace Does Not Mean Automatically Trusted

An Action being available in GitHub Marketplace does not mean:

```text
Automatically safe
Automatically maintained
Automatically appropriate for production
```

Always evaluate the Action.

---

# Action Inputs and Secrets

Never pass secrets unnecessarily.

Bad:

```yaml
with:
  password: ${{ secrets.MY_PASSWORD }}
```

if the Action does not genuinely need the secret.

Better:

```text
Only provide the minimum required secret.
```

---

# Secret Handling

Use GitHub Secrets or approved secret-management mechanisms.

Example:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

Avoid:

```yaml
run: echo "${{ secrets.API_TOKEN }}"
```

because secrets should not be intentionally printed to logs.

---

# Environment Secrets

Production deployments can use environment-specific secrets.

Conceptually:

```text
Development
   |
   └── Dev Secrets

QA
   |
   └── QA Secrets

Production
   |
   └── Production Secrets
```

This helps separate credentials between environments.

---

# Action and Environment

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        uses: ./actions/deploy
```

The Action executes within the protected job context.

---

# Action Outputs Example

Suppose an Action produces an image tag.

```yaml
- name: Build Image
  id: image
  uses: ./actions/build-image
```

If the Action defines an output:

```yaml
outputs:
  image:
```

the workflow can reference:

```yaml
${{ steps.image.outputs.image }}
```

Example:

```yaml
- name: Display Image
  run: |
    echo "Image: ${{ steps.image.outputs.image }}"
```

---

# Action Inputs Example

Custom Action:

```yaml
- name: Deploy
  uses: ./actions/deploy
  with:
    environment: production
    version: ${{ github.sha }}
```

Inputs:

```text
environment
version
```

The Action can consume these values.

---

# Action Reusability

Actions are useful because they package repeated logic.

Without an Action:

```text
Workflow A
   ├── 20 commands

Workflow B
   ├── 20 commands

Workflow C
   └── 20 commands
```

With a reusable Action:

```text
Custom Action
     |
     ├── Workflow A
     ├── Workflow B
     └── Workflow C
```

This can reduce duplication.

---

# Action vs Reusable Workflow

These are different concepts.

### Action

Packages a task:

```text
Checkout
Build
Scan
Deploy
```

### Reusable Workflow

Packages an entire workflow/job structure.

Conceptually:

```text
Reusable Workflow
   |
   ├── Job
   ├── Steps
   ├── Permissions
   └── Deployment logic
```

Use the appropriate abstraction.

---

# Example

Action:

```yaml
- uses: actions/checkout@v4
```

Reusable workflow:

```yaml
jobs:
  call-ci:
    uses: ./.github/workflows/reusable-ci.yml
```

They solve different reuse problems.

---

# Action Composition

A workflow can combine many Actions.

Example:

```text
Workflow
 |
 ├── Checkout Action
 |
 ├── Setup Java Action
 |
 ├── Build
 |
 ├── Security Scan Action
 |
 └── Artifact Action
```

This creates a complete CI/CD pipeline from reusable components.

---

# Production CI/CD Example

```yaml
name: Production CI

on:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Build
        run: mvn clean package

      - name: Security Scan
        uses: aquasecurity/trivy-action@<reviewed-version>
        with:
          scan-type: fs
          scan-ref: .

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: application
          path: target/*.jar
```

In a real production repository, pin and approve third-party Action versions according to your organization's security policy.

---

# Action Version Strategy

Possible strategies:

```text
Major Version
@v4

Minor / Patch Version
@v4.x.x

Commit SHA
@<sha>
```

Trade-offs:

```text
Tag
→ Easier maintenance

Commit SHA
→ Stronger immutability
```

A mature organization can use automated dependency updates with review.

---

# Action Updates

Actions should be treated like dependencies.

Example:

```text
Current
checkout@v4

New
checkout@v5
```

Before upgrading:

```text
Review changelog
Test workflow
Review breaking changes
Validate security
Merge through normal review
```

---

# Action Dependency Management

Track:

```text
Action Name
Version
Purpose
Owner
Repository
Security Review
Update Process
```

Example:

| Action | Purpose | Version Strategy |
|---|---|---|
| checkout | Checkout code | Approved version |
| setup-java | Configure Java | Approved version |
| upload-artifact | Store artifacts | Approved version |
| security scan | Scan source/images | Reviewed version |

---

# Internal Actions

Large organizations can create internal Actions.

Example:

```text
Organization
 |
 └── platform-actions
      |
      ├── java-build
      ├── docker-build
      ├── security-scan
      └── deploy
```

Application teams can reuse standardized platform functionality.

---

# Benefits of Internal Actions

- Standardization
- Reduced duplication
- Centralized maintenance
- Consistent security controls
- Easier onboarding
- Platform engineering enablement

---

# Internal Action Example

Workflow:

```yaml
- name: Standard Docker Build
  uses: company/platform-actions/docker-build@v1
```

The platform team maintains:

```text
Build
Tagging
Security
Registry Authentication
Push
```

Application teams consume the standardized Action.

---

# Action Governance

Enterprise organizations should define:

```text
Approved Actions
Approved Versions
Allowed Sources
Version Pinning
Security Review
Update Process
Ownership
```

Example policy:

```text
GitHub Official Actions
       |
       ↓
Approved

Internal Organization Actions
       |
       ↓
Approved

Third-Party Actions
       |
       ↓
Security Review Required
```

---

# Allowlisting Actions

An organization may choose to restrict which Actions repositories can use.

Conceptually:

```text
Allowed:
actions/*
company/*
approved-security/*
```

Unapproved Actions:

```text
Blocked / Review Required
```

This can improve supply-chain security.

---

# Action Permissions Review

For every Action ask:

```text
What does it execute?
What permissions does it need?
What files does it access?
What network access does it require?
Does it use secrets?
Does it modify the repository?
Does it upload data?
```

---

# Action and `GITHUB_TOKEN`

GitHub Actions provides a `GITHUB_TOKEN` to workflows.

Its permissions should be limited.

Example:

```yaml
permissions:
  contents: read
```

If an Action needs to write:

```yaml
permissions:
  contents: write
```

only grant that when required.

---

# Dangerous Pattern

```yaml
permissions: write-all
```

This grants broad write access.

Avoid it unless there is a justified requirement.

---

# Better Pattern

```yaml
permissions:
  contents: read
```

Then add only required permissions.

Example:

```yaml
permissions:
  contents: read
  packages: write
```

---

# Action Failure

If an Action fails:

```text
Workflow
   |
   ↓
Action
   |
   ↓
Failure
```

Check:

```text
Action version
Inputs
Permissions
Runner OS
Environment variables
Secrets
Network
Action logs
Dependencies
```

---

# Action Debugging

A useful process:

```text
1. Read Action error
2. Identify failing step
3. Check inputs
4. Check permissions
5. Check secrets
6. Check runner environment
7. Check Action documentation
8. Check Action source
9. Test with a supported version
```

---

# Debug Logging

GitHub Actions supports debug logging mechanisms that can provide additional workflow diagnostics when enabled appropriately.

Use debug logging carefully because logs can contain sensitive contextual information.

Never intentionally print secrets.

---

# Action Timeout

Actions execute as workflow steps.

Jobs can define timeouts.

Example:

```yaml
jobs:

  build:

    timeout-minutes: 20
```

This prevents a stuck job from running indefinitely.

---

# Action Failure Handling

A workflow can control failure behavior.

Example:

```yaml
- name: Optional Check
  continue-on-error: true
  run: ./optional-check.sh
```

Use this carefully.

Do not hide critical security or deployment failures.

---

# Critical Security Actions

For security stages such as:

```text
SAST
SCA
Container Scanning
DAST
```

avoid blindly using:

```yaml
continue-on-error: true
```

if the scan is intended to be a release gate.

---

# Action in DevSecOps Pipeline

A production pipeline might be:

```text
Checkout
   |
   ↓
Build
   |
   ↓
Unit Test
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
Docker Build
   |
   ↓
Push Image
   |
   ↓
Deploy
```

Actions can implement many of these steps.

---

# Action in GitOps Pipeline

Example:

```text
Checkout
   |
   ↓
Build
   |
   ↓
Security Scan
   |
   ↓
Push Image
   |
   ↓
Update Git Manifest
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

Actions can standardize:

```text
Build
Scan
Registry Login
Manifest Update
```

---

# Production Action Selection Checklist

Before adding an Action to a production workflow:

```text
☐ Is the Action actually necessary?
☐ Is there an official Action?
☐ Is the source trustworthy?
☐ Is it maintained?
☐ What permissions does it require?
☐ Does it access secrets?
☐ Is the version reviewed?
☐ Can it be pinned?
☐ Has it been tested?
☐ Is the Action allowed by organization policy?
☐ What happens if the Action is compromised?
```

---

# Security Boundary

Remember:

```text
Action
   |
   ↓
Runner
   |
   ↓
Workflow Permissions
   |
   ↓
Secrets / Cloud / Repository
```

An Action is code executing inside the workflow.

Therefore:

```text
Third-party Action
=
Third-party code execution
```

Treat it accordingly.

---

# Common Mistakes

### Mistake 1

Using:

```yaml
uses: some-action@main
```

without reviewing the dependency.

---

### Mistake 2

Giving excessive permissions:

```yaml
permissions: write-all
```

---

### Mistake 3

Passing secrets unnecessarily.

---

### Mistake 4

Using an untrusted Action in a production workflow.

---

### Mistake 5

Using `continue-on-error` for critical security checks.

---

### Mistake 6

Installing an Action when a simple `run` command is sufficient.

---

### Mistake 7

Creating custom Actions for trivial one-line commands.

---

### Mistake 8

Duplicating complex logic instead of creating a reusable internal Action.

---

# Best Practices

- Use Actions for reusable tasks.
- Use `run` for simple commands.
- Review third-party Actions before production use.
- Prefer trusted maintainers.
- Pin versions according to organizational policy.
- Use least-privilege permissions.
- Avoid unnecessary secrets.
- Keep Actions updated.
- Test Action upgrades.
- Maintain internal Actions when standardization is valuable.
- Document Action ownership.
- Monitor security advisories.
- Separate trusted production workflows from untrusted PR workflows.
- Avoid unnecessary `continue-on-error`.
- Treat Actions as software supply-chain dependencies.

---

# Production Architecture

A mature GitHub Actions platform can look like:

```text
                    GitHub
                       |
                       ↓
                  Workflow
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Official       Internal     Approved
      Actions        Actions      Third-Party
          |            |            |
          └────────────┼────────────┘
                       ↓
                     Runner
                       |
             ┌─────────┼─────────┐
             ↓         ↓         ↓
           Build     Scan      Deploy
             |
             ↓
            ECR
             |
             ↓
           ArgoCD
             |
             ↓
            EKS
```

---

# Interview Questions

## Basic

1. What is a GitHub Action?
2. What is the difference between `uses` and `run`?
3. What does `actions/checkout@v4` mean?
4. What are Action inputs?
5. What are Action outputs?
6. What is the purpose of an Action `id`?
7. What is the difference between an Action and a workflow?

## Intermediate

8. What types of GitHub Actions exist?
9. What is a Marketplace Action?
10. What is a local Action?
11. How do you pass inputs to an Action?
12. How do you consume outputs from an Action?
13. What is the difference between an Action and a reusable workflow?
14. How would you debug a failing Action?
15. Why should Actions be versioned?
16. What are the security risks of third-party Actions?
17. How would you manage Action versions across an organization?

## Advanced / Production

18. How would you design a secure enterprise policy for third-party GitHub Actions?
19. Why is `@main` potentially risky for production Actions?
20. How would you use commit SHA pinning for Action supply-chain security?
21. How would you audit a third-party Action before allowing it in production?
22. How would you implement an allowlist of approved Actions?
23. How would you design internal Actions for a DevOps platform team?
24. How would you minimize the permissions available to an Action?
25. How would you protect secrets when using third-party Actions?
26. A third-party Action is compromised. What is the potential blast radius?
27. How would you detect and respond to a compromised Action dependency?
28. How would you design a standardized Docker build Action for multiple microservices?
29. How would you integrate SonarQube, Trivy, and Veracode using Actions in a DevSecOps pipeline?
30. How would you design an Action-based GitOps pipeline using ECR, ArgoCD, and EKS?
31. When should you create a custom Action instead of using a shell script?
32. When should you create a reusable workflow instead of a custom Action?
33. How would you manage Action upgrades safely across hundreds of repositories?
34. How would you prevent untrusted pull requests from using privileged Actions with production credentials?
35. Explain how `GITHUB_TOKEN` permissions affect third-party Actions.
36. Why is `permissions: write-all` dangerous?
37. How would you design an enterprise Action governance model covering ownership, versioning, security review, and updates?