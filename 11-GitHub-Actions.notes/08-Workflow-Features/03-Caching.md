# GitHub Actions Caching

Caching allows GitHub Actions workflows to reuse files from previous workflow runs instead of downloading or generating them again.

Caching can significantly improve CI/CD performance by reducing:

```text
Dependency Download Time
Build Time
Network Usage
Runner Execution Time
Pipeline Cost
```

Typical things that can be cached:

```text
npm dependencies
Maven dependencies
Gradle dependencies
Python packages
Terraform providers
Docker build layers
Package manager metadata
Build outputs
```

---

# Why Caching Matters

Without caching:

```text
Workflow
   |
   ↓
Install Dependencies
   |
   ↓
Download Everything
   |
   ↓
Build
   |
   ↓
Test
```

Every workflow may repeat the same downloads.

With caching:

```text
Workflow
   |
   ↓
Check Cache
   |
   ├── HIT
   │    ↓
   │  Reuse Data
   │
   └── MISS
        ↓
     Download
        ↓
     Save Cache
```

---

# Cache Hit

A cache hit means the requested cache was found.

Example:

```text
Workflow
   |
   ↓
Cache Lookup
   |
   ↓
Cache HIT
   |
   ↓
Restore Dependencies
```

The workflow can continue without downloading everything again.

---

# Cache Miss

A cache miss means the requested cache was not found.

```text
Workflow
   |
   ↓
Cache Lookup
   |
   ↓
CACHE MISS
   |
   ↓
Download Dependencies
   |
   ↓
Build
   |
   ↓
Save Cache
```

The next suitable workflow can potentially reuse the saved cache.

---

# GitHub Actions Cache Action

A common way to use caching is:

```yaml
uses: actions/cache@v4
```

Example:

```yaml
- name: Cache
  uses: actions/cache@v4
  with:
    path: ~/.cache/my-tool
    key: ${{ runner.os }}-my-tool-${{ hashFiles('**/lockfile') }}
```

The important parts are:

```text
path
key
```

---

# Cache Path

`path` tells GitHub Actions what files or directories should be cached.

Example:

```yaml
path: ~/.m2/repository
```

For Maven.

Example:

```yaml
path: ~/.npm
```

For npm.

Example:

```yaml
path: ~/.cache/pip
```

For pip.

---

# Cache Key

The cache key uniquely identifies the cache.

Example:

```yaml
key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

Conceptually:

```text
Operating System
      +
Package Manager
      +
Dependency Lock File Hash
```

---

# Why Lock Files Matter

Consider:

```text
package-lock.json
pom.xml
requirements.txt
```

Dependency definitions can change.

If dependencies change, the cache should normally change accordingly.

Using a dependency file hash helps detect this.

---

# `hashFiles()`

Example:

```yaml
${{ hashFiles('package-lock.json') }}
```

The expression generates a hash based on the matching file contents.

Example key:

```yaml
key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

If the lock file changes:

```text
Old hash
   ↓
New hash
```

A different cache key is produced.

---

# Cache Key Example

```text
Linux
 +
npm
 +
package-lock.json hash
```

Result conceptually:

```text
Linux-npm-abc123...
```

After dependency changes:

```text
Linux-npm-def456...
```

The cache key changes.

---

# Restore Keys

You can provide restore keys.

Example:

```yaml
- name: Cache
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-
```

This allows a broader matching strategy when an exact cache key is unavailable.

---

# Exact Cache Match

Example:

```text
Requested:
Linux-npm-ABC123
```

Available:

```text
Linux-npm-ABC123
```

Result:

```text
Exact HIT
```

---

# Restore Key Match

Requested:

```text
Linux-npm-ABC123
```

Available:

```text
Linux-npm-XYZ999
```

Restore prefix:

```text
Linux-npm-
```

A compatible older cache may be restored depending on cache matching behavior.

---

# Cache Key Strategy

A good cache key should include information that affects the cached data.

Examples:

```text
OS
Architecture
Package Manager
Dependency Lock File
Tool Version
Build Configuration
```

---

# Bad Cache Key

Example:

```yaml
key: dependencies
```

This is too broad.

A dependency update may still reuse:

```text
Old dependency cache
```

when you actually need a new cache.

---

# Better Cache Key

```yaml
key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

This changes when the dependency lock file changes.

---

# Include Architecture When Necessary

For workflows running on multiple architectures:

```yaml
key: ${{ runner.os }}-${{ runner.arch }}-npm-${{ hashFiles('package-lock.json') }}
```

This prevents incompatible caches from being treated as identical.

---

# Cache Isolation

Caches should be designed so that incompatible data is not reused.

Consider:

```text
Linux
Windows
macOS
```

and:

```text
x64
ARM64
```

If the cached files are platform-dependent, include the relevant platform information in the key.

---

# npm Caching

Example:

```yaml
name: Node CI

on:
  push:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm

      - name: Install Dependencies
        run: npm ci

      - name: Test
        run: npm test
```

`setup-node` provides package-manager caching support.

This is often preferable to manually implementing cache logic when the setup Action already supports the package manager.

---

# npm Cache Concept

```text
package-lock.json
       |
       ↓
Cache Key
       |
       ↓
~/.npm
       |
       ↓
npm ci
       |
       ↓
Faster Dependency Installation
```

---

# Maven Caching

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '17'
    cache: maven

- name: Build
  run: mvn -B verify
```

This caches Maven dependencies.

---

# Maven Cache

Typical cache location:

```text
~/.m2/repository
```

Flow:

```text
pom.xml
   |
   ↓
Maven Dependency Cache
   |
   ↓
mvn verify
```

---

# Python Caching

Example using `setup-python`:

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: pip
```

Then:

```yaml
- name: Install
  run: |
    pip install -r requirements.txt
```

---

# Python Cache

Typical package cache:

```text
pip cache
```

The dependency file can be used to determine cache invalidation.

---

# Gradle Caching

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '17'
    cache: gradle
```

Gradle dependencies can then be reused across suitable workflow runs.

---

# Terraform Provider Caching

Terraform downloads providers during initialization.

Caching can reduce repeated downloads.

Example:

```text
Terraform
   |
   ↓
Provider Download
   |
   ↓
Cache
```

A production workflow should consider:

```text
Terraform version
Provider lock file
OS
Architecture
```

when designing provider caching.

---

# Terraform Lock File

Terraform commonly uses:

```text
.terraform.lock.hcl
```

This records provider selections and checksums.

Cache keys should account for the provider dependencies and platform where appropriate.

---

# Terraform Cache Concept

```yaml
- name: Cache Terraform
  uses: actions/cache@v4
  with:
    path: ~/.terraform.d/plugin-cache
    key: ${{ runner.os }}-${{ runner.arch }}-terraform-${{ hashFiles('**/.terraform.lock.hcl') }}
```

A Terraform plugin cache must also be configured correctly for Terraform to use it.

---

# Docker Layer Caching

Docker builds can be expensive.

Without caching:

```text
Docker Build
   |
   ├── Base Image
   ├── Dependencies
   ├── Application
   └── Build
```

Many layers may be rebuilt.

With layer caching:

```text
Docker Build
   |
   ↓
Existing Layers
   |
   ↓
Reuse
   |
   ↓
Build Only Changed Layers
```

---

# Docker Buildx Cache

A common approach uses BuildKit/Buildx caching.

Example:

```yaml
- name: Build and Push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: ${{ env.IMAGE }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

This uses GitHub Actions cache storage for BuildKit cache data.

---

# Docker Cache Flow

```text
Dockerfile
    |
    ↓
Buildx
    |
    ├── Cache HIT → Reuse Layers
    |
    └── Cache MISS → Build Layer
```

---

# Docker Layer Ordering

Docker caching is strongly affected by Dockerfile ordering.

Less optimal:

```dockerfile
COPY . .
RUN npm install
```

Any source change can invalidate the layer containing the dependency installation.

Better:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
```

Now application source changes are less likely to invalidate the dependency installation layer.

---

# Docker Cache Optimization

For Node.js:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

This allows the dependency layer to remain reusable when only application source changes.

---

# Multi-Stage Docker Build

Caching can also improve multi-stage builds.

Example:

```dockerfile
FROM node:20 AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

BuildKit can cache appropriate layers.

---

# Cache and Security

Caching introduces security considerations.

Never casually cache:

```text
Secrets
Credentials
Private Keys
Production Tokens
Sensitive Configuration
```

---

# Do Not Cache Secrets

Bad:

```yaml
path: ~/.aws
```

or:

```yaml
path: .env
```

if these locations contain credentials or sensitive information.

---

# Cache Poisoning

Cache data may become dangerous if untrusted workflows can influence the cached content and trusted workflows later consume it.

Potential flow:

```text
Untrusted PR
     |
     ↓
Writes Cache
     |
     ↓
Trusted Workflow
     |
     ↓
Restores Cache
     |
     ↓
Executes Cached Content
```

This can create a supply-chain/security risk.

---

# Cache Security Principle

```text
Never assume cached content is trustworthy
just because it came from a previous workflow run.
```

Use cache scopes and keys carefully.

---

# Pull Request Security

Be especially careful when workflows run on:

```text
pull_request
```

from forks or other untrusted sources.

Do not create a design where untrusted code can poison data that a privileged production workflow later consumes.

---

# Cache and Secrets

A cache should never be used as a mechanism for transferring secrets between jobs.

Use:

```text
GitHub Secrets
OIDC
Environment Secrets
Secure Artifact Mechanisms
```

as appropriate.

---

# Cache vs Artifact

These are different.

### Cache

Designed primarily for:

```text
Reusable dependencies
Build acceleration
Intermediate reusable data
```

### Artifact

Designed primarily for:

```text
Build outputs
Reports
Logs
Packages
Deployment bundles
```

---

# Cache vs Artifact

```text
Cache
  ↓
Optimize future workflow runs

Artifact
  ↓
Store and transfer workflow outputs
```

Do not use cache as your primary artifact storage mechanism.

---

# Cache vs Git Repository

Do not use cache to store source-of-truth configuration.

Use:

```text
Git
```

for:

```text
Source Code
Terraform
Helm
GitOps Manifests
Workflow Definitions
```

---

# Cache vs ECR

For Docker images:

```text
Cache
  ↓
Speed up build

ECR
  ↓
Store deployable container image
```

The cache should not replace ECR as the production container registry.

---

# Cache and Immutable Artifacts

Production should use:

```text
Immutable Image
```

rather than treating the cache as the deployment artifact.

Example:

```text
Git SHA
   ↓
Docker Build
   ↓
ECR Image
   ↓
Image Digest
   ↓
Production
```

Cache only accelerates the build.

---

# Cache Key Invalidation

The most important caching concept is:

```text
When should this cache stop being reused?
```

Example:

```text
Dependency Change
      |
      ↓
Lock File Changes
      |
      ↓
Hash Changes
      |
      ↓
New Cache Key
```

---

# Cache Invalidation

Common cache invalidation inputs:

```text
Dependency Lock File
Tool Version
OS
Architecture
Build Configuration
Compiler Version
Runtime Version
```

---

# Cache Versioning

You can manually version a cache:

```yaml
key: v2-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
```

If the cache structure changes:

```text
v1 → v2
```

This avoids reusing an incompatible old cache.

---

# When to Change Cache Version

Change the version when:

```text
Cache directory changes
Cache format changes
Tool version changes significantly
Build strategy changes
Cache becomes corrupted
```

Example:

```text
v1
 ↓
Cache structure changed
 ↓
v2
```

---

# Cache Hit Detection

With `actions/cache`, you can capture information about whether an exact cache hit occurred.

Example:

```yaml
- name: Cache
  id: cache
  uses: actions/cache@v4
  with:
    path: ~/.cache/example
    key: example-${{ runner.os }}-${{ hashFiles('**/lockfile') }}

- name: Cache Result
  run: |
    echo "Cache hit: ${{ steps.cache.outputs.cache-hit }}"
```

Use the output appropriately in workflow logic.

---

# Cache Hit Condition

Example:

```yaml
- name: Download Dependencies
  if: steps.cache.outputs.cache-hit != 'true'
  run: |
    ./download-dependencies.sh
```

Be careful: an imperfect/partial restore can still leave useful files in the cache, so your workflow should remain correct even when the cache is not an exact hit.

---

# Cache Should Be an Optimization

Very important:

```text
Build should work without cache.
```

Bad design:

```text
No cache
   ↓
Pipeline fails
```

Good design:

```text
Cache HIT
   ↓
Faster

Cache MISS
   ↓
Download
   ↓
Continue
```

---

# Cache Failure

If cache restoration fails:

```text
Pipeline
   |
   ↓
Cache failure
   |
   ↓
Continue without cache
```

Where possible, the build should remain functionally correct.

---

# Cache and Dependency Installation

Example:

```text
Cache HIT
   ↓
npm ci
   ↓
Uses cached package data
```

or:

```text
Cache MISS
   ↓
npm ci
   ↓
Downloads packages
   ↓
Future runs can benefit
```

---

# Cache and Build Performance

Example:

```text
Without Cache:
Dependency Download → 5 min
Build               → 3 min
Total               → 8 min

With Cache:
Dependency Download → 1 min
Build               → 3 min
Total               → 4 min
```

Actual savings depend on the workload.

---

# Measuring Cache Effectiveness

Track:

```text
Cache Hit Rate
Dependency Install Time
Build Time
Workflow Duration
Network Usage
Runner Cost
```

---

# Cache Hit Rate

Conceptually:

```text
Cache Hit Rate =
Successful Cache Hits
---------------------
Total Cache Lookups
```

A low hit rate may indicate:

```text
Bad Key
Frequent Dependency Changes
Incorrect Path
Too Many Cache Versions
Platform Differences
```

---

# Cache and Pipeline Optimization

Caching can help reduce a pipeline from:

```text
25 minutes
```

to a much shorter duration.

But caching is only one optimization.

Also consider:

```text
Parallel Jobs
Dependency Caching
Docker Layer Caching
Test Parallelization
Build Optimization
Artifact Reuse
```

---

# Cache and Your CI/CD Pipeline

For your DevSecOps workflow:

```text
Checkout
   |
   ↓
Cache Dependencies
   |
   ↓
Build
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
ECR
   |
   ↓
Helm / GitOps
   |
   ↓
ArgoCD
```

Caching should accelerate the earlier build/dependency stages.

---

# Node.js DevSecOps Pipeline

Example:

```yaml
name: Node CI

on:
  push:
  pull_request:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

---

# Maven DevSecOps Pipeline

```yaml
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
          java-version: '17'
          cache: maven

      - name: Build
        run: |
          mvn -B verify
```

---

# Docker Build with Cache

```yaml
- name: Build and Push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: ${{ env.IMAGE }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

# Cache and ECR

Important distinction:

```text
GitHub Actions Cache
        ↓
Build acceleration

ECR
        ↓
Container image registry
```

Example:

```text
Source
  ↓
Docker Build
  ↓
Build Cache
  ↓
ECR
  ↓
EKS
```

---

# Cache and ArgoCD

ArgoCD does not use GitHub Actions dependency cache as the production source of truth.

Correct architecture:

```text
GitHub Actions
   |
   ├── Cache → Build acceleration
   |
   └── GitOps commit → Source of truth
                         |
                         ↓
                       ArgoCD
                         |
                         ↓
                        EKS
```

---

# Cache and Terraform

Correct architecture:

```text
Terraform Provider Cache
        ↓
Faster terraform init

Terraform State
        ↓
S3 / Remote Backend

Terraform Plan
        ↓
Review

Terraform Apply
        ↓
Infrastructure
```

Do not confuse:

```text
Cache
```

with:

```text
Terraform State
```

---

# Cache and State

Cache:

```text
Performance data
```

State:

```text
Authoritative infrastructure state
```

Never use GitHub Actions cache as a replacement for Terraform remote state.

---

# Cache and Security Scans

You can cache dependencies used by security tools where supported.

But do not blindly cache:

```text
Scanner credentials
Private vulnerability databases
Sensitive reports
```

unless the tool/documentation explicitly supports a secure mechanism.

---

# Cache and Trivy

Trivy may maintain vulnerability database/cache data depending on configuration.

If caching Trivy data:

```text
Cache
   ↓
Trivy DB
   ↓
Faster startup
```

But vulnerability data becomes stale over time.

Security scanning must prioritize fresh vulnerability intelligence where required.

---

# Cache Freshness

Caching security databases can create a problem:

```text
Old DB
   ↓
Scan
   ↓
Missing New CVE
```

Therefore:

```text
Performance
vs
Fresh Security Data
```

must be balanced.

---

# Cache and Supply Chain Security

Caching can affect software supply-chain security.

Potential risks:

```text
Cache Poisoning
Stale Dependencies
Unexpected Binary Reuse
Untrusted Cache Data
Cross-branch Data Reuse
```

Use:

```text
Trusted cache scopes
Strong keys
Immutable artifacts
Dependency verification
Lock files
Least privilege
```

---

# Dependency Lock Files

Use lock files where supported:

```text
package-lock.json
yarn.lock
pnpm-lock.yaml
pom.xml / dependency management
requirements lock
.terraform.lock.hcl
```

They help make dependency resolution reproducible.

---

# Cache and Reproducibility

A good pipeline should produce the same logical build whether:

```text
Cache HIT
```

or:

```text
Cache MISS
```

The cache should not alter the intended dependency versions.

---

# Cache and Reproducible Builds

```text
Lock File
   |
   ↓
Dependency Resolution
   |
   ↓
Cache
   |
   ↓
Build
```

The lock file remains the source of dependency versions.

---

# Cache Permissions

Only grant workflows the permissions they actually need.

Caching should not be used as a way to transfer privileged information between trust boundaries.

---

# Cache and Forks

Be careful with caches in pull requests from forks.

The workflow should not allow untrusted code to:

```text
Poison trusted cache
```

or:

```text
Access privileged information through cache behavior
```

---

# Cache and Branch Isolation

Use keys that distinguish important execution dimensions.

Example:

```yaml
key: ${{ runner.os }}-${{ github.ref_name }}-npm-${{ hashFiles('package-lock.json') }}
```

However, do not add branch information blindly if you intentionally want cache reuse across branches.

The key design should match the trust and reuse model.

---

# Cache Scope Principle

Ask:

```text
Who can create this cache?
Who can restore it?
What data does it contain?
Can untrusted code influence it?
```

before designing a cache strategy.

---

# Cache and Production

Production deployment should not depend on an arbitrary cache.

Instead:

```text
Source Commit
     ↓
Build
     ↓
Security
     ↓
Immutable Image
     ↓
ECR
     ↓
Approved Promotion
     ↓
Production
```

Cache is only an optimization.

---

# Production Cache Architecture

```text
Developer Push
      |
      ↓
GitHub Actions
      |
      ├── Dependency Cache
      |
      ├── Docker Build Cache
      |
      ↓
Build
      |
      ↓
Security
      |
      ↓
ECR Image
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

# Cache Lifecycle

```text
Workflow Start
      |
      ↓
Calculate Key
      |
      ↓
Cache Lookup
      |
 ┌────┴────┐
 ↓         ↓
HIT       MISS
 ↓         ↓
Restore   Build/Download
 ↓         ↓
Continue  Save Cache
            |
            ↓
         Continue
```

---

# Cache Key Lifecycle

```text
Lock File
   |
   ↓
hashFiles()
   |
   ↓
Cache Key
   |
   ↓
Lookup
```

If the lock file changes:

```text
New Hash
   |
   ↓
New Key
   |
   ↓
New Cache
```

---

# Cache Versioning Architecture

```text
Cache v1
   |
   ↓
Tool / Structure Changes
   |
   ↓
Cache v2
```

Use an explicit version prefix when necessary:

```yaml
key: v2-${{ runner.os }}-${{ hashFiles('**/lockfile') }}
```

---

# Cache Debugging

If caching does not work, check:

```text
1. Is the path correct?
2. Is the key correct?
3. Is hashFiles() matching the expected file?
4. Is the OS/architecture correct?
5. Is the cache being restored?
6. Is the dependency installation actually using it?
7. Is the cache too frequently invalidated?
8. Is a stale restore key being used?
```

---

# Cache Performance Troubleshooting

If cache exists but pipeline is still slow:

```text
Cache HIT
   |
   ↓
Still slow
```

Investigate:

```text
Build
Tests
Security Scan
Docker Build
Network
Runner CPU
Runner Memory
```

A cache hit does not guarantee a fast pipeline.

---

# Cache Size Considerations

Do not cache everything.

Bad:

```text
Entire workspace
```

Potential problems:

```text
Large cache
Slow save
Slow restore
Frequent invalidation
Unnecessary files
```

Cache only what provides meaningful benefit.

---

# Cache What Is Expensive to Recreate

Good candidates:

```text
Dependency Downloads
Compiler Dependencies
Docker Build Layers
Terraform Providers
Tool Caches
```

Poor candidates:

```text
Small files
Source Code
Secrets
Temporary Logs
Frequently Changing Output
```

---

# Cache vs Workspace

A workspace contains:

```text
Source
Generated Files
Temporary Files
Dependencies
```

Do not automatically cache the entire workspace.

Select only the directories that are expensive and safe to reuse.

---

# Cache and Artifacts

A typical pipeline might use both:

```text
Cache
  ↓
Speed up build

Artifact
  ↓
Transfer build result

ECR
  ↓
Store production image
```

Example:

```text
Dependencies → Cache
Build Report → Artifact
Docker Image → ECR
GitOps Manifest → Git
```

Each storage mechanism has a different purpose.

---

# Recommended Storage Model

```text
Source Code
    → Git

Dependencies / Build Cache
    → GitHub Actions Cache

Reports / Test Results
    → Artifacts

Container Images
    → ECR

Terraform State
    → Remote Backend

GitOps Desired State
    → Git Repository
```

---

# Cache Strategy for Your DevOps Stack

For your environment:

```text
GitHub Actions
 ├── npm/Maven dependency cache
 ├── Docker BuildKit cache
 └── Terraform provider cache
```

Then:

```text
ECR
 └── Immutable container images

S3
 └── Terraform remote state

GitOps Repository
 └── Kubernetes desired state

ArgoCD
 └── Reconciliation

EKS
 └── Runtime
```

---

# Production-Level Pipeline

```text
                    Git Push
                       |
                       ↓
                 GitHub Actions
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Checkout      Cache       Variables
                       |
                       ↓
                    Build
                       |
                       ↓
              Docker Build Cache
                       |
                       ↓
                 Security Gates
              ┌────────┼────────┐
              ↓        ↓        ↓
          SonarQube   Trivy   Veracode
              └────────┼────────┘
                       ↓
                  ECR Image
                       |
                       ↓
                Immutable Digest
                       |
                       ↓
                  GitOps Commit
                       |
                       ↓
                    ArgoCD
                       |
                       ↓
                      EKS
```

---

# Cache Best Practices

```text
☐ Cache only useful data
☐ Use dependency lock files in keys
☐ Include OS when necessary
☐ Include architecture when necessary
☐ Version cache formats
☐ Use restore keys carefully
☐ Do not cache secrets
☐ Do not cache production credentials
☐ Treat cache data as potentially untrusted
☐ Protect against cache poisoning
☐ Do not use cache as artifact storage
☐ Do not use cache as Terraform state
☐ Do not use cache as container registry
☐ Ensure cache miss still works
☐ Monitor cache effectiveness
☐ Keep security databases fresh
```

---

# Common Mistakes

### 1. Caching everything

Creates large and inefficient caches.

### 2. Bad cache key

Old dependencies may be reused.

### 3. No lock file in key

Dependency changes may not invalidate the cache correctly.

### 4. Caching secrets

Potential security exposure.

### 5. Treating cache as an artifact

Caches are not the correct source for production deployment artifacts.

### 6. Treating cache as Terraform state

Never replace remote state with a workflow cache.

### 7. Trusting cache contents blindly

Cache poisoning is a security concern.

### 8. Caching security data indefinitely

Security databases can become stale.

### 9. Making the pipeline depend on cache

A cache miss should normally cause a slower build, not a broken build.

### 10. Using branch keys without considering reuse

This can create unnecessary duplicate caches.

---

# Key Takeaways

Caching improves workflow performance by reusing expensive-to-create data.

The core pattern is:

```text
Calculate Key
     ↓
Cache Lookup
     ↓
   HIT?
  /     \
YES      NO
 |        |
Restore  Download
 |        |
 ↓        ↓
Build   Save
```

Important concepts:

```text
path
key
restore-keys
hashFiles()
cache-hit
```

Common caches:

```text
npm
Maven
Gradle
pip
Terraform providers
Docker BuildKit
```

The most important security principle:

```text
Cache is an optimization mechanism,
not a trusted source of production state.
```

For production:

```text
Cache
  ↓
Build faster

ECR
  ↓
Store immutable images

Git
  ↓
Source / GitOps state

S3
  ↓
Terraform state

ArgoCD
  ↓
Reconcile

EKS
  ↓
Run
```

---

# Interview Questions

## Basic

1. What is caching in GitHub Actions?
2. Why is caching used?
3. What is a cache hit?
4. What is a cache miss?
5. What is the `path` in `actions/cache`?
6. What is the `key` in `actions/cache`?
7. What are `restore-keys`?
8. What is `hashFiles()`?
9. Why should dependency lock files be included in cache keys?
10. What is the difference between cache and artifact?

## Intermediate

11. How would you cache npm dependencies?
12. How would you cache Maven dependencies?
13. How would you cache Python dependencies?
14. How would you cache Terraform providers?
15. How would you cache Docker BuildKit layers?
16. How would you determine whether a cache was an exact hit?
17. How would you version a cache?
18. What happens when the dependency lock file changes?
19. Why should OS and architecture sometimes be included in the cache key?
20. What makes a good cache key?
21. Why should a build work even when the cache misses?
22. How would you troubleshoot a cache that is not being restored?
23. How would you measure cache effectiveness?
24. What is the difference between dependency caching and Docker layer caching?
25. Why shouldn't GitHub Actions cache replace ECR?

## Advanced / Production

26. Design a caching strategy for a Node.js/Java microservices platform.
27. How would you optimize a 25-minute GitHub Actions pipeline using caching?
28. How would you combine npm/Maven caching with Docker BuildKit caching?
29. How would you design Terraform provider caching while keeping `.terraform.lock.hcl` authoritative?
30. Why is cache poisoning a security risk?
31. How could an untrusted pull request influence a cache?
32. How would you protect trusted workflows from cache poisoning?
33. Why should secrets never be stored in GitHub Actions cache?
34. How would you design caching for a DevSecOps pipeline using SonarQube, Trivy, and Veracode?
35. Should a vulnerability database always be cached? Explain the security trade-off.
36. How would you design cache keys for multiple services, environments, operating systems, and architectures?
37. How would you handle stale caches?
38. How would you decide whether to use a cache or artifact?
39. How would you design caching for Docker images while using ECR for production?
40. How would you design caching in a GitOps + ArgoCD + EKS architecture?
41. Why should GitHub Actions cache never be used as Terraform remote state?
42. How would you handle a cache miss in a production CI/CD pipeline?
43. How would you balance cache performance against supply-chain security?
44. How would you prevent untrusted PR workflows from influencing privileged production workflows through cached data?
45. Design an enterprise-grade GitHub Actions caching architecture covering npm/Maven dependencies, Terraform providers, Docker BuildKit, security tooling, ECR, GitOps, ArgoCD, EKS, cache invalidation, cache poisoning protection, and production artifact traceability.