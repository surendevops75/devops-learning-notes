# Dependency Caching with GitHub Actions

Dependency caching is a technique used in CI/CD pipelines to reuse previously downloaded dependencies instead of downloading them again on every workflow run.

Without caching:

    GitHub Actions Runner
            |
            ↓
    Download Dependencies
            |
            ↓
    Build / Test
            |
            ↓
    Runner Destroyed
            |
            ↓
    Next Workflow
            |
            ↓
    Download Dependencies Again

With caching:

    First Run
        |
        ↓
    Download Dependencies
        |
        ↓
    Cache
        |
        ↓
    Next Run
        |
        ↓
    Restore Cache
        |
        ↓
    Build / Test

The main goal is to reduce CI execution time and unnecessary network downloads.

---

# Why Dependency Caching Is Important

Modern applications depend on many external libraries.

Examples:

Java:

    Maven Dependencies

Node.js:

    npm Dependencies

Python:

    pip Dependencies

Terraform:

    Provider Plugins

Docker:

    Base Images / Build Cache

Without caching, every workflow run may spend significant time downloading the same dependencies.

Caching can improve:

- Build speed
- Test speed
- CI feedback time
- Network usage
- Developer productivity

---

# What Is a Dependency?

A dependency is software required by an application.

Example:

    Node.js Application
        |
        +-- express
        +-- axios
        +-- lodash
        +-- jest
        +-- typescript

These packages are downloaded during dependency installation.

Java:

    application
        |
        +-- Spring
        +-- Jackson
        +-- JUnit
        +-- Maven Plugins

Python:

    application
        |
        +-- requests
        +-- flask
        +-- pytest
        +-- boto3

---

# Dependency Installation Without Cache

Example Node.js pipeline:

    Checkout
       |
       ↓
    npm ci
       |
       ↓
    Download Dependencies
       |
       ↓
    npm test
       |
       ↓
    Build

Every workflow may repeat the dependency download.

---

# Dependency Installation With Cache

    Checkout
       |
       ↓
    Restore npm Cache
       |
       +-- Cache Hit
       |      |
       |      ↓
       |   npm ci
       |
       +-- Cache Miss
              |
              ↓
        Download Dependencies
              |
              ↓
           Save Cache
              |
              ↓
             Test

---

# GitHub Actions Cache

GitHub Actions provides caching through:

    actions/cache

Example:

    - name: Cache Dependencies
      uses: actions/cache@v4
      with:
        path: ~/.cache
        key: cache-${{ runner.os }}-${{ hashFiles('**/lockfile') }}

The cache consists of files that can be reused by future workflow runs.

---

# Cache Key

The cache key identifies a specific cache.

Example:

    key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}

The key contains:

    Operating System
          +
    Package Manager
          +
    Dependency Lock File Hash

This makes the cache specific to the dependency state.

---

# Why Use Lock Files in Cache Keys?

Suppose:

    package-lock.json

changes.

The dependency set may also change.

If the cache key contains:

    hashFiles('package-lock.json')

then a different lock file generates a different cache key.

Flow:

    package-lock.json
          |
          ↓
       Hash
          |
          ↓
      Cache Key
          |
          ↓
       Cache

When dependencies change:

    New Lock File
          |
          ↓
      New Hash
          |
          ↓
      New Cache Key
          |
          ↓
      New Cache

---

# hashFiles()

GitHub Actions provides:

    hashFiles()

It calculates a hash based on matching files.

Example:

    ${{ hashFiles('package-lock.json') }}

Another example:

    ${{ hashFiles('**/pom.xml') }}

Another:

    ${{ hashFiles('**/requirements.txt') }}

This is useful for creating dependency-aware cache keys.

---

# Node.js Dependency Caching

GitHub Actions provides convenient caching through:

    actions/setup-node

Example:

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

This enables npm dependency caching based on the package lock file.

---

# Node.js Cache Example

    name: Node CI

    on:
      pull_request:

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Run Tests
            run: npm test

---

# What setup-node Caching Does

With:

    cache: npm

GitHub Actions caches npm's global package data.

It does not simply cache the entire:

    node_modules/

directory.

The dependency installation command still runs:

    npm ci

but packages can be restored from the cache instead of being downloaded from the network.

---

# Why npm ci Is Still Used

Even with caching:

    npm ci

is preferred for CI because it installs dependencies from the lock file in a clean and predictable manner.

Caching improves download speed.

It does not replace dependency installation.

---

# Node.js Cache Dependency File

By default, setup-node can use the package manager's dependency file for the cache key.

For npm:

    package-lock.json

For other package managers, corresponding lock files can be used.

---

# Multiple Node.js Lock Files

If a repository contains multiple lock files:

    frontend/package-lock.json
    backend/package-lock.json

you can specify the dependency path.

Example:

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm
        cache-dependency-path: |
          frontend/package-lock.json
          backend/package-lock.json

---

# Node.js Monorepo Caching

Example:

    repository/
    |
    ├── frontend/
    │   └── package-lock.json
    |
    └── backend/
        └── package-lock.json

Cache configuration:

    cache-dependency-path: |
      frontend/package-lock.json
      backend/package-lock.json

This allows the cache key to account for both dependency files.

---

# npm Cache

To understand the concept:

    npm ci
       |
       ↓
    npm Cache
       |
       +-- Hit → Faster Download
       |
       +-- Miss → Download
                    |
                    ↓
                 Cache Data

The application still receives dependencies through npm.

---

# Maven Dependency Caching

For Java projects, GitHub Actions can cache Maven dependencies using:

    actions/setup-java

Example:

    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'
        cache: maven

Then:

    - name: Build
      run: mvn clean package

---

# Maven Cache Location

Maven commonly stores downloaded dependencies under:

    ~/.m2/repository

The setup-java caching mechanism handles Maven dependency caching.

Flow:

    pom.xml
       |
       ↓
    Maven
       |
       ↓
    ~/.m2/repository
       |
       ↓
    Cache

---

# Maven Cache Example

    name: Java CI

    on:
      pull_request:

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Java
            uses: actions/setup-java@v4
            with:
              distribution: temurin
              java-version: '21'
              cache: maven

          - name: Build and Test
            run: mvn clean verify

---

# Maven Cache Key

The cache should change when Maven dependency definitions change.

Important files can include:

    pom.xml

and potentially other Maven configuration files depending on the project.

A dependency change should result in a different cache state.

---

# Maven Multi-Module Project

Example:

    project/
    |
    ├── pom.xml
    ├── service-a/
    │   └── pom.xml
    ├── service-b/
    │   └── pom.xml
    └── service-c/
        └── pom.xml

The caching configuration should account for the relevant dependency definition files.

---

# Python Dependency Caching

GitHub Actions provides caching through:

    actions/setup-python

Example:

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip

Then:

    - name: Install Dependencies
      run: pip install -r requirements.txt

---

# Python Cache Example

    name: Python CI

    on:
      pull_request:

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Python
            uses: actions/setup-python@v5
            with:
              python-version: '3.12'
              cache: pip

          - name: Install Dependencies
            run: pip install -r requirements.txt

          - name: Run Tests
            run: pytest

---

# Python Cache Files

Depending on the project, dependency definitions may be:

    requirements.txt

    requirements-dev.txt

    pyproject.toml

    poetry.lock

    Pipfile.lock

The cache configuration should use the appropriate dependency file.

---

# Python Multiple Dependency Files

Example:

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip
        cache-dependency-path: |
          requirements.txt
          requirements-dev.txt

---

# pip Cache

The cache typically contains downloaded package files rather than the complete installed environment.

Flow:

    requirements.txt
          |
          ↓
        pip
          |
          ↓
    pip Cache
          |
          ↓
    Installation
          |
          ↓
       Testing

---

# Cache vs Artifact

Caching and artifacts are different.

Cache:

    Optimize Workflow Performance

Artifact:

    Preserve / Transfer Workflow Output

Example:

    Dependency Cache
        |
        ↓
    npm packages

Artifact:

    Build
        |
        ↓
    application.jar

---

# Cache vs Artifact Example

Cache:

    node_modules package data
    Maven dependencies
    pip packages

Artifact:

    JAR
    ZIP
    Test Reports
    Coverage Reports

Think:

    Cache = Speed

    Artifact = Output

---

# Cache vs Docker Image

Cache:

    Reusable Build Data

Docker Image:

    Deployable Application Artifact

Example:

    Dependency Cache
         |
         ↓
    Docker Build
         |
         ↓
    Docker Image
         |
         ↓
    ECR

---

# Docker Build Cache

Docker builds can also benefit from caching.

Example:

    Dockerfile

    FROM node:20

    WORKDIR /app

    COPY package*.json ./

    RUN npm ci

    COPY . .

The Docker build cache can reuse the dependency installation layer when package files have not changed.

---

# Docker Layer Caching

Example:

    COPY package*.json ./
          |
          ↓
        npm ci
          |
          ↓
      COPY source
          |
          ↓
       Application

If only source code changes:

    package*.json
        |
        ↓
    Same
        |
        ↓
    npm ci Layer
        |
        ↓
    Reused

This can significantly speed up Docker builds.

---

# Bad Dockerfile Ordering

Example:

    COPY . .

    RUN npm ci

Any source change can invalidate the layer before npm install.

This can cause dependencies to be installed again.

---

# Better Dockerfile Ordering

Example:

    COPY package*.json ./

    RUN npm ci

    COPY . .

This separates dependency files from source code.

Result:

    Dependency Files
        |
        ↓
    Install Dependencies
        |
        ↓
    Source Code
        |
        ↓
    Application

---

# Docker BuildKit Cache

Docker BuildKit supports advanced build caching.

GitHub Actions can use:

    docker/build-push-action

with cache configuration.

Example concept:

    - name: Build Docker Image
      uses: docker/build-push-action@v6
      with:
        context: .
        push: false
        tags: myapp:${{ github.sha }}

Build cache can be configured using cache-from and cache-to.

---

# Docker Cache Flow

    GitHub Actions
         |
         ↓
    Docker Build
         |
         ↓
    Cache
         |
         +-- Hit → Reuse Layers
         |
         +-- Miss → Build Layer
                         |
                         ↓
                       Cache

---

# Cache-from and Cache-to

Example concept:

    cache-from: type=gha
    cache-to: type=gha,mode=max

This uses GitHub Actions cache storage for BuildKit cache data.

The exact configuration should match the Docker build requirements and GitHub Actions limits.

---

# Docker Cache Example

    - name: Build Docker Image
      uses: docker/build-push-action@v6
      with:
        context: .
        push: false
        tags: myapp:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

This can reduce repeated Docker build work across workflow runs.

---

# Terraform Provider Caching

Terraform downloads provider plugins.

Example:

    terraform init
        |
        ↓
    AWS Provider
        |
        ↓
    Download
        |
        ↓
    Cache

Terraform supports plugin caching through configuration such as:

    plugin_cache_dir

The exact implementation depends on the Terraform environment.

---

# Terraform CI

Typical flow:

    Checkout
       |
       ↓
    Terraform Init
       |
       ↓
    Terraform Validate
       |
       ↓
    Terraform Plan
       |
       ↓
    Apply

Provider caching can reduce repeated downloads during CI.

---

# Dependency Cache Key Design

A good cache key often includes:

    Operating System
        +
    Package Manager
        +
    Runtime Version
        +
    Lock File Hash

Example:

    Linux
      +
    Node 20
      +
    npm
      +
    package-lock hash

---

# Example Cache Key

Conceptually:

    ${{ runner.os }}-node-20-${{ hashFiles('package-lock.json') }}

Result:

    Linux
      |
      ↓
    Node 20
      |
      ↓
    Lock File Hash
      |
      ↓
    Cache Key

---

# Why Include Operating System?

Dependencies may differ between operating systems.

Example:

    Ubuntu
        |
        ↓
    Linux Dependencies

    Windows
        |
        ↓
    Windows Dependencies

Using:

    ${{ runner.os }}

helps avoid using an incompatible cache across operating systems.

---

# Why Include Runtime Version?

Different runtime versions can require different dependencies.

Example:

    Node 18
       |
       ↓
    Cache A

    Node 20
       |
       ↓
    Cache B

Do not blindly share runtime-specific caches.

---

# Why Include Lock File Hash?

The lock file defines dependency versions.

If:

    package-lock.json

changes:

    Hash Changes
        |
        ↓
    Cache Key Changes
        |
        ↓
    New Cache

This prevents stale dependency caches from being treated as the exact dependency state.

---

# Cache Hit

A cache hit means GitHub Actions found a matching cache.

Flow:

    Cache Key
       |
       ↓
    Matching Cache
       |
       ↓
    Cache Hit
       |
       ↓
    Restore Data
       |
       ↓
    Faster Build

---

# Cache Miss

A cache miss means there is no matching cache.

Flow:

    Cache Key
       |
       ↓
    No Matching Cache
       |
       ↓
    Cache Miss
       |
       ↓
    Download Dependencies
       |
       ↓
    Save Cache

The workflow still works.

It is simply slower.

---

# Cache Restore Key

The cache action supports restore keys.

Example:

    - name: Cache
      uses: actions/cache@v4
      with:
        path: ~/.cache
        key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-npm-

Restore keys can allow a broader cache to be considered when the exact key is unavailable.

---

# Exact Cache Key vs Restore Key

Exact key:

    Linux-npm-ABC123

Restore key:

    Linux-npm-

Exact match:

    Linux-npm-ABC123
          |
          ↓
       Hit

No exact match but prefix available:

    Linux-npm-XYZ999
          |
          ↓
    Restore broader cache

---

# Cache Fallback

Conceptually:

    Exact Cache
       |
       +-- Found → Use
       |
       +-- Not Found
              |
              ↓
        Restore Key
              |
              +-- Found → Restore
              |
              +-- Not Found → Fresh Download

---

# Cache Version

GitHub Actions manages cache metadata and cache versions internally.

Changing cache configuration or paths can result in a different cache state.

The important principle is:

    Same Cache Configuration
          +
    Same Key
          |
          ↓
    Reusable Cache

---

# Cache Scope

GitHub Actions cache availability is subject to GitHub's cache scope and access rules.

A cache created from one branch may not be directly available to all other branches in exactly the same way.

Do not design a pipeline assuming every branch can access every cache.

---

# Pull Request Cache Considerations

Pull Requests from forks require special care.

Do not expose sensitive data through cache contents.

Caches can contain files that may be reused by later workflow runs according to GitHub's cache rules.

Never place secrets inside dependency caches.

---

# Cache Security Risk

Caches can become a security concern if untrusted workflow code can influence cache contents or restore paths.

Do not store:

    Secrets
    Credentials
    Private Keys
    Tokens

inside caches.

Cache only non-sensitive build dependencies.

---

# Cache Poisoning

Cache poisoning occurs when an attacker causes malicious or unexpected data to be stored in a cache that is later trusted by another workflow.

Potential impact:

    Malicious Cache
          |
          ↓
    Restored by CI
          |
          ↓
    Build / Test
          |
          ↓
    Compromised Pipeline

Mitigation:

- Avoid caching sensitive outputs
- Use precise cache keys
- Separate trusted and untrusted workflows
- Use least-privilege permissions
- Do not execute untrusted cached content blindly

---

# Cache Security Principle

Cache:

    Dependency Data

Do not cache:

    Secrets
    Credentials
    Production Data
    Authentication Tokens

---

# Dependency Lock Files

Common lock files:

Node.js:

    package-lock.json

Yarn:

    yarn.lock

pnpm:

    pnpm-lock.yaml

Java:

    pom.xml
    Gradle Lock Files

Python:

    requirements.txt
    poetry.lock
    Pipfile.lock
    pyproject.toml

Lock files improve reproducibility.

---

# Why Lock Files Matter for CI

Without lock files:

    Build
       |
       ↓
    Dependency Version
       |
       ↓
    Could Change

With lock files:

    Build
       |
       ↓
    Locked Dependency Versions
       |
       ↓
    Predictable Installation

Caching works best when dependency versions are deterministic.

---

# npm ci vs npm install

For CI:

    npm ci

is generally preferred when a lock file is available.

Why?

- Clean installation
- Uses lock file
- Predictable dependency versions
- Designed for automated environments

Caching improves the download side of the process.

---

# Maven Dependency Resolution

Maven:

    pom.xml
       |
       ↓
    Dependency Resolution
       |
       ↓
    ~/.m2/repository
       |
       ↓
    Build

Caching:

    Existing Dependencies
       |
       ↓
    Reuse
       |
       ↓
    Faster Build

---

# Python Dependency Resolution

Python:

    requirements.txt
       |
       ↓
    pip install
       |
       ↓
    pip Cache
       |
       ↓
    Environment
       |
       ↓
    Tests

Caching reduces repeated downloads.

---

# Cache Invalidation

Cache invalidation means determining when an old cache should no longer be reused.

Typical triggers:

- Lock File Changed
- Runtime Changed
- OS Changed
- Dependency Configuration Changed
- Cache Path Changed

A good cache key helps automate invalidation.

---

# Cache Invalidation Example

Initial:

    package-lock.json
        |
        ↓
    Hash A
        |
        ↓
    Cache A

Dependency update:

    package-lock.json
        |
        ↓
    Hash B
        |
        ↓
    Cache B

The old cache can remain available according to retention rules, but the workflow now uses the new dependency-specific cache.

---

# Cache Too Broad

Bad key:

    key: npm-cache

This does not distinguish:

- Node versions
- OS
- Dependency versions

Potential problem:

    Wrong Cache
        |
        ↓
    Dependency Installation
        |
        ↓
    Unexpected Behavior

Use more specific keys.

---

# Cache Too Specific

A cache key that changes on every tiny event can reduce cache reuse.

Example:

    key includes:
    Git SHA

If every commit produces a new key:

    Commit A → Cache A
    Commit B → Cache B
    Commit C → Cache C

This may reduce cache effectiveness.

For dependency caches, the lock file is often a better invalidation signal than the full commit SHA.

---

# Good Cache Key

Example:

    ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}

This changes when the dependency lock file changes.

---

# Cache Strategy for Monorepo

Example:

    services/
    |
    +-- user/
    |    └── package-lock.json
    |
    +-- catalog/
    |    └── package-lock.json
    |
    +-- payment/
         └── package-lock.json

You can use service-specific caches when services have independent dependencies.

Example concept:

    user-${{ hashFiles('services/user/package-lock.json') }}

---

# Monorepo Cache Problem

If one global lock file is used:

    repository/package-lock.json

one cache may be sufficient.

If every service has independent lock files:

    service A
    service B
    service C

consider dependency-specific caching.

---

# Cache for Microservices

Example:

    Microservices
       |
       +-- User
       |     ↓
       |   npm Cache
       |
       +-- Catalog
       |     ↓
       |   Maven Cache
       |
       +-- Payment
             ↓
           Python Cache

Each service can use the appropriate package manager cache.

---

# Cache and Matrix Jobs

Suppose:

    Node 18
    Node 20
    Node 22

Each version should have a compatible dependency cache.

Conceptually:

    Node 18 → Cache 18
    Node 20 → Cache 20
    Node 22 → Cache 22

Runtime version should be part of the cache strategy when needed.

---

# Matrix Cache Example

    strategy:
      matrix:
        node:
          - 18
          - 20
          - 22

    steps:

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm

This lets setup-node manage the dependency cache appropriately.

---

# Cache and Self-Hosted Runners

Self-hosted runners may have persistent local files.

This can create a different caching model.

GitHub-hosted runner:

    Job
      |
      ↓
    Runner
      |
      ↓
    Destroyed

Self-hosted runner:

    Job
      |
      ↓
    Persistent Runner
      |
      ↓
    Local Files May Remain

Do not rely on accidental leftover files.

Use explicit cache or cleanup strategies.

---

# Why GitHub-Hosted Runners Need Caching

GitHub-hosted runners are generally ephemeral.

Example:

    Job 1
       |
       ↓
    Runner
       |
       ↓
    Dependencies
       |
       ↓
    Job Ends
       |
       ↓
    Runner Removed

The next job may start with a fresh environment.

Caching provides persistence between workflow runs.

---

# Caching in CI Architecture

    GitHub Repository
           |
           ↓
    GitHub Actions
           |
           ↓
    Ephemeral Runner
           |
           +-- Restore Cache
           |
           +-- Build
           |
           +-- Test
           |
           +-- Save Cache
           |
           ↓
    Runner Destroyed

---

# Cache Lifecycle

    Cache Request
         |
         ↓
    Generate Key
         |
         ↓
    Search Cache
         |
         +-- Hit
         |    |
         |    ↓
         |  Restore
         |
         +-- Miss
              |
              ↓
        Download Dependencies
              |
              ↓
          Save Cache

---

# Cache Optimization

Before adding caching:

    Measure CI

Then:

    Add Cache

Then measure:

    Before:
       8 minutes

    After:
       5 minutes

Caching should be evaluated using actual pipeline performance.

---

# Cache Hit Rate

A useful metric is cache hit rate.

Example:

    100 Workflow Runs
        |
        +-- 90 Cache Hits
        +-- 10 Cache Misses

Cache Hit Rate:

    90%

A low hit rate may indicate:

- Poor key design
- Lock file changing frequently
- Runtime changes
- Multiple unrelated cache keys

---

# Cache Performance

Caching is useful when:

    Download Time
        >
    Cache Restore Overhead

For very small dependencies, caching may provide little benefit.

Always measure.

---

# Large Cache Considerations

Large caches can take time to restore.

Example:

    Dependency Cache
        |
        ↓
      2 GB
        |
        ↓
    Restore Time
        |
        ↓
    Significant

Avoid caching unnecessary files.

---

# What Should Be Cached?

Good candidates:

- Package Manager Caches
- Maven Dependencies
- npm Cache
- pip Cache
- Gradle Cache
- Docker Build Cache
- Terraform Provider Cache
- Other Reusable Build Data

---

# What Should Not Be Cached?

Avoid caching:

- Secrets
- Passwords
- API Tokens
- Private Keys
- Production Data
- Temporary Logs
- Unnecessary Build Outputs
- Files that can be cheaply regenerated

---

# Dependency Cache Best Practices

- Use lock files
- Use precise cache keys
- Include OS when required
- Include runtime version when required
- Avoid using Git SHA as the only key
- Cache package-manager data
- Do not cache secrets
- Measure cache effectiveness
- Keep caches reasonably sized
- Use built-in setup action caching where available
- Use Docker BuildKit caching for Docker builds
- Understand cache scope
- Handle forked Pull Requests carefully
- Do not rely on stale caches
- Keep CI deterministic

---

# Built-In Caching vs actions/cache

Two common approaches:

    Built-In Caching
        |
        +-- setup-node
        +-- setup-java
        +-- setup-python

    Manual Caching
        |
        ↓
    actions/cache

Prefer built-in caching when the official setup action provides the required functionality.

Use actions/cache when custom caching behavior is needed.

---

# Node.js Built-In Cache

Use:

    actions/setup-node

Example:

    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

---

# Java Built-In Cache

Use:

    actions/setup-java

Example:

    - uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'
        cache: maven

---

# Python Built-In Cache

Use:

    actions/setup-python

Example:

    - uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip

---

# Manual Cache Example

Example:

    - name: Cache npm
      uses: actions/cache@v4
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-

    - name: Install Dependencies
      run: npm ci

---

# Manual Maven Cache

Example:

    - name: Cache Maven
      uses: actions/cache@v4
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-

    - name: Build
      run: mvn clean verify

---

# Manual Python Cache

Example:

    - name: Cache pip
      uses: actions/cache@v4
      with:
        path: ~/.cache/pip
        key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
        restore-keys: |
          ${{ runner.os }}-pip-

    - name: Install
      run: pip install -r requirements.txt

---

# Cache Path

The path tells GitHub Actions which files to cache.

Example:

    path: ~/.npm

or:

    path: ~/.m2/repository

or:

    path: ~/.cache/pip

Only cache data that can safely be recreated.

---

# Cache Restore and Save

The basic lifecycle:

    Restore
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Save

GitHub Actions handles cache persistence for the cache action.

---

# Cache Failure Should Not Break CI

A cache miss should not normally cause the build to fail.

Correct behavior:

    Cache Hit
       |
       ↓
    Faster Build

    Cache Miss
       |
       ↓
    Normal Dependency Download
       |
       ↓
    Build

The cache is an optimization, not a required dependency.

---

# Dependency Caching and Reliability

A pipeline should work even when:

    Cache Miss

This means:

    Cache = Optimization

not:

    Cache = Required Infrastructure

If the build depends on a cache existing, the pipeline is fragile.

---

# Dependency Caching and Reproducibility

Caching should not change dependency versions.

Use:

    Lock File
        +
    Deterministic Install
        +
    Cache

Example:

    package-lock.json
        |
        ↓
    npm ci
        |
        ↓
    Cached Downloads
        |
        ↓
    Same Dependency Versions

---

# Dependency Caching and Security

A secure strategy:

    Lock File
        |
        ↓
    Dependency Install
        |
        ↓
    Cache
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan

Caching does not replace dependency security scanning.

---

# Dependency Vulnerability Scanning

Caching dependencies does not mean skipping security checks.

Example:

    Restore Cache
        |
        ↓
    Install Dependencies
        |
        ↓
    Dependency Scan
        |
        ↓
    Build

or:

    Build
        |
        ↓
    Security Scan
        |
        ↓
    Publish

The exact placement depends on the tools and project.

---

# Caching and SonarQube

Caching dependency downloads can speed up the build that feeds SonarQube.

Flow:

    Restore Cache
        |
        ↓
    Install
        |
        ↓
    Build
        |
        ↓
    Tests
        |
        ↓
    SonarQube

Caching does not replace SonarQube analysis.

---

# Caching and Trivy

For container pipelines:

    Restore Build Cache
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Publish

Docker build caching can reduce build time.

Trivy should still scan the resulting image.

---

# Full DevSecOps Pipeline with Caching

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    Restore Dependency Cache
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
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Quality / Security Gate
        |
        ↓
    Artifact / ECR
        |
        ↓
    Deployment

---

# Caching in a Java Pipeline

Example:

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Java
            uses: actions/setup-java@v4
            with:
              distribution: temurin
              java-version: '21'
              cache: maven

          - name: Build and Test
            run: mvn clean verify

This is a simple and recommended approach for Maven projects.

---

# Caching in a Node.js Pipeline

Example:

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install
            run: npm ci

          - name: Test
            run: npm test

---

# Caching in a Python Pipeline

Example:

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Setup Python
            uses: actions/setup-python@v5
            with:
              python-version: '3.12'
              cache: pip

          - name: Install
            run: pip install -r requirements.txt

          - name: Test
            run: pytest

---

# Dependency Caching Decision Tree

Ask:

    Is the data expensive to download?
             |
             +-- No → Caching may not be necessary
             |
             +-- Yes
                   |
                   ↓
          Is it safe to cache?
                   |
                   +-- No → Do not cache
                   |
                   +-- Yes
                         |
                         ↓
                Does official setup action
                support caching?
                         |
                         +-- Yes → Use built-in cache
                         |
                         +-- No → Consider actions/cache

---

# Troubleshooting Cache Misses

If caching is not working:

    1. Check cache path
    2. Check cache key
    3. Check lock file
    4. Check runtime version
    5. Check OS
    6. Check cache scope
    7. Check workflow logs
    8. Check dependency manager
    9. Check cache size
    10. Check whether the dependency files changed

---

# Troubleshooting: Cache Key Changes Every Run

Possible problem:

    key:
      ${{ github.sha }}

Every commit produces:

    New SHA
       |
       ↓
    New Cache

For dependency caching, prefer dependency-state-based keys such as:

    hashFiles('package-lock.json')

when appropriate.

---

# Troubleshooting: Cache Never Hits

Check:

    Cache Key
       |
       ↓
    Does it change?
       |
       +-- Yes → Investigate key design
       |
       +-- No
            |
            ↓
        Check Path
            |
            ↓
        Check Scope
            |
            ↓
        Check Cache Logs

---

# Troubleshooting: Cache Restored but Build Still Slow

Possible reasons:

- Cache contains only package metadata
- Installation still performs work
- Build itself is slow
- Tests are slow
- Docker layers are not cached
- Cache is too large
- Cache restore is slow

Measure each stage separately.

---

# Troubleshooting: Wrong Dependencies

If incorrect dependencies are restored:

    Check Lock File
        |
        ↓
    Check Cache Key
        |
        ↓
    Check Runtime Version
        |
        ↓
    Check OS
        |
        ↓
    Check Dependency Manager

Avoid using overly broad cache keys.

---

# Troubleshooting: Cache Corruption

If a cache appears corrupted:

    1. Verify dependency lock file
    2. Review cache key
    3. Change cache key if required
    4. Recreate cache
    5. Run clean installation

The pipeline should still be capable of rebuilding dependencies from the package registry.

---

# Scenario: CI Takes 20 Minutes

Suppose:

    Dependency Download → 8 min
    Build               → 5 min
    Tests               → 7 min

After dependency caching:

    Dependency Download → 1 min
    Build               → 5 min
    Tests               → 7 min

New total:

    13 minutes

Caching helped because dependency downloads were a significant portion of the pipeline.

---

# Scenario: CI Takes 25 Minutes

Suppose:

    Dependencies → 2 min
    Build        → 3 min
    Tests        → 20 min

Caching dependencies may reduce only a small portion.

Better optimization:

    Test Parallelization
    Test Sharding
    Test Optimization

Always optimize the actual bottleneck.

---

# Scenario: Dependency Changes Frequently

If:

    package-lock.json

changes on almost every build:

    Cache A
    Cache B
    Cache C
    Cache D

The cache hit rate may be lower.

Investigate why dependency definitions change so frequently.

---

# Scenario: Security Concern About Cache

If an organization is concerned about cache security:

    Do Not Cache:
        Secrets
        Credentials
        Production Data

    Cache:
        Re-downloadable Dependency Data

Also review:

- Workflow trust boundaries
- Forked Pull Requests
- Cache scope
- Permissions
- Untrusted code execution

---

# Scenario: Monorepo with Multiple Languages

Example:

    Repository
       |
       +-- Java Service
       |      ↓
       |    Maven Cache
       |
       +-- Node Service
       |      ↓
       |    npm Cache
       |
       +-- Python Service
              ↓
            pip Cache

Each service should use its appropriate dependency cache.

---

# Scenario: Docker Microservices

Example:

    User Service
       |
       ↓
    Docker Build
       |
       ↓
    BuildKit Cache

    Catalog Service
       |
       ↓
    Docker Build
       |
       ↓
    BuildKit Cache

    Payment Service
       |
       ↓
    Docker Build
       |
       ↓
    BuildKit Cache

Caching can significantly improve repeated image builds.

---

# Scenario: Production Pipeline

A production pipeline should still work if the cache is unavailable.

Correct:

    Cache
      |
      +-- Hit → Faster
      |
      +-- Miss → Download
                    |
                    ↓
                  Build
                    |
                    ↓
                 Publish

Incorrect:

    Cache
      |
      ↓
    Required
      |
      ↓
    Cache Missing
      |
      ↓
    Pipeline Failure

---

# Interview Questions

## Basic

1. What is dependency caching?
2. Why do we use caching in GitHub Actions?
3. What is the difference between cache and artifact?
4. What is actions/cache?
5. What is a cache key?
6. What is hashFiles()?
7. What is a cache hit?
8. What is a cache miss?
9. Why use lock files?
10. What should not be stored in a cache?

---

# Intermediate Interview Questions

11. How do you cache npm dependencies?

12. How do you cache Maven dependencies?

13. How do you cache Python dependencies?

14. How does setup-node caching work?

15. How does setup-java caching work?

16. How does setup-python caching work?

17. How do you design a cache key?

18. Why should the operating system be part of the cache key?

19. Why should the lock file be part of the cache key?

20. Why should runtime version sometimes be part of the cache strategy?

21. What is a restore key?

22. What happens when a cache miss occurs?

23. Does a cache miss fail the pipeline?

24. How do you cache Docker build layers?

25. How do you cache dependencies in a monorepo?

---

# Advanced Interview Questions

26. Design a dependency caching strategy for a large GitHub Actions environment.

27. How would you optimize a CI pipeline using caching?

28. How would you troubleshoot a cache that never hits?

29. How would you protect GitHub Actions caches from poisoning?

30. How would you design caching for a microservices repository?

31. How would you cache Docker BuildKit layers?

32. How would you balance cache size and cache performance?

33. How would you design caching for multiple Node.js versions?

34. How would you design caching for multiple operating systems?

35. How would you handle caching in Pull Requests from forks?

36. How would you design caching for Java, Node.js, and Python services in one repository?

37. How would you measure whether caching actually improved the pipeline?

---

# Scenario Question

## Your GitHub Actions pipeline takes 15 minutes, and dependency installation takes 7 minutes. What would you do?

I would first confirm that dependency download is the actual bottleneck.

Then:

    Identify Package Manager
          |
          ↓
    Enable Official Cache
          |
          ↓
    Use Lock File
          |
          ↓
    Run CI Again
          |
          ↓
    Measure Improvement

For example:

    Node.js
        |
        ↓
    setup-node
        |
        ↓
    cache: npm

If the dependency installation drops significantly, caching is providing value.

---

# Scenario Question

## Your cache never hits. How would you troubleshoot it?

I would inspect:

    Cache Key
        |
        +-- OS
        +-- Runtime
        +-- Package Manager
        +-- Lock File Hash
        |
        ↓
    Cache Path
        |
        ↓
    Cache Scope
        |
        ↓
    Workflow Logs

I would also check whether the dependency lock file changes on every run.

---

# Scenario Question

## A developer says caching is causing stale dependencies. What would you check?

I would check:

    1. Lock file
    2. Cache key
    3. Dependency manager
    4. Runtime version
    5. OS
    6. Cache path
    7. Installation command

For npm, I would normally use:

    npm ci

with the lock file as the source of dependency versions.

The cache should speed up downloads, not determine which dependency versions are installed.

---

# Scenario Question

## How would you cache a Node.js application?

I would use:

    actions/setup-node

Example:

    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

Then:

    npm ci
    npm test

The package lock file is used to identify the dependency state.

---

# Scenario Question

## How would you cache a Maven project?

I would use:

    actions/setup-java

Example:

    - uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'
        cache: maven

Then:

    mvn clean verify

This caches Maven dependency data.

---

# Scenario Question

## How would you cache Python dependencies?

I would use:

    actions/setup-python

Example:

    - uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip

Then:

    pip install -r requirements.txt
    pytest

---

# Scenario Question

## How would you improve Docker build performance?

I would:

    1. Optimize Dockerfile layer ordering
    2. Copy dependency files before source
    3. Use Docker BuildKit
    4. Configure cache-from
    5. Configure cache-to
    6. Avoid unnecessary build context
    7. Use appropriate base images

Example:

    COPY package*.json ./
    RUN npm ci
    COPY . .

This allows the dependency layer to be reused when only application source changes.

---

# Scenario Question

## Should caching be mandatory for a CI pipeline?

No.

Caching should be an optimization.

The pipeline should still work when:

    Cache Hit
        |
        ↓
      Fast

or:

    Cache Miss
        |
        ↓
      Slower
        |
        ↓
      Still Works

---

# Scenario Question

## What would you cache in a DevSecOps pipeline?

I would cache reusable dependency/build data such as:

    npm Cache
    Maven Cache
    pip Cache
    Docker Build Cache
    Terraform Provider Cache

I would not cache:

    Secrets
    Credentials
    Production Data
    Private Keys

Security scans would still run after dependencies are restored and artifacts are built.

---

# Scenario Question

## How would you optimize a 25-minute GitHub Actions pipeline?

I would first measure the pipeline.

Example:

    Dependency Download → 8 min
    Build               → 4 min
    Tests               → 10 min
    Security            → 3 min

Then:

    Dependency Cache
        +
    Test Parallelization
        +
    Build Cache
        +
    Dependency Optimization

I would optimize the largest bottlenecks first rather than adding caching blindly.

---

# Best Practices Checklist

- Use dependency lock files
- Use official setup-action caching when available
- Use precise cache keys
- Include OS when appropriate
- Include runtime version when appropriate
- Include dependency lock file state
- Use restore keys carefully
- Do not use Git SHA as the dependency cache key
- Do not cache secrets
- Do not cache production credentials
- Do not depend on caches for correctness
- Measure cache hit rate
- Measure actual CI improvement
- Keep cache sizes reasonable
- Use Docker BuildKit for Docker build caching
- Optimize Dockerfile layer order
- Use separate caches where dependencies are independent
- Consider monorepo architecture
- Consider matrix runtime versions
- Understand cache scope
- Treat cache as an optimization, not a source of truth

---

# Important GitHub Actions Syntax

Node.js:

    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

Java:

    - uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'
        cache: maven

Python:

    - uses: actions/setup-python@v5
      with:
        python-version: '3.12'
        cache: pip

Manual cache:

    - uses: actions/cache@v4
      with:
        path: ~/.cache
        key: my-cache-${{ runner.os }}-${{ hashFiles('lockfile') }}

Restore keys:

    restore-keys: |
      my-cache-${{ runner.os }}-

Docker BuildKit:

    cache-from: type=gha
    cache-to: type=gha,mode=max

---

# Important Tools

GitHub Actions:

    CI/CD Automation

actions/cache:

    General-Purpose Workflow Caching

actions/setup-node:

    Node.js Setup + npm/yarn/pnpm Caching

actions/setup-java:

    Java Setup + Maven/Gradle Caching

actions/setup-python:

    Python Setup + pip Caching

Docker BuildKit:

    Docker Build Cache

Maven:

    Java Dependency Management

npm:

    Node.js Dependency Management

pip:

    Python Dependency Management

Terraform:

    Infrastructure Provider Management

---

# Cache Mental Model

Think of caching as:

    "I already downloaded or generated this
     reusable data before, so I don't want
     to repeat the expensive work."

The flow:

    Dependency Definition
          |
          ↓
       Cache Key
          |
          ↓
    Search Existing Cache
          |
          +-- HIT
          |    |
          |    ↓
          |  Restore
          |
          +-- MISS
               |
               ↓
        Download / Generate
               |
               ↓
            Build
               |
               ↓
          Save Cache

---

# Final CI Performance Model

Without caching:

    Checkout
       |
       ↓
    Download Dependencies
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Destroy Runner

With caching:

    Checkout
       |
       ↓
    Restore Cache
       |
       ↓
    Download Only Missing Dependencies
       |
       ↓
    Build
       |
       ↓
    Test
       |
       ↓
    Save Updated Cache

Core idea:

Dependency caching is a CI performance optimization. The cache should contain reusable, non-sensitive dependency or build data, while the lock file and deterministic installation process remain the source of truth for dependency versions. In GitHub Actions, prefer the built-in caching capabilities of actions such as setup-node, setup-java, and setup-python when they meet the requirement, and use actions/cache for custom caching needs. A cache miss should make the pipeline slower, not make it incorrect or unusable.