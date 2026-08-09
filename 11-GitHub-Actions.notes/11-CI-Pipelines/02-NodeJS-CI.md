# NodeJS CI

Node.js Continuous Integration (CI) is the process of automatically installing dependencies, validating, testing, analyzing, securing, and packaging Node.js application changes whenever developers push code or create a Pull Request.

The main goal is to detect problems early and ensure that only validated code moves toward deployment.

A typical Node.js CI pipeline looks like:

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
    +-- Setup Node.js
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
    +-- Build
    |
    ↓
Validated Application
```

---

# What Is Node.js CI?

Node.js CI automatically validates Node.js source code whenever changes are submitted.

Typical activities include:

```text
Source Code Checkout
Node.js Setup
Dependency Installation
Linting
Compilation / Build
Unit Testing
Integration Testing
Code Quality Analysis
Security Scanning
Packaging
Artifact Generation
```

The pipeline provides fast feedback to developers.

---

# Why Node.js CI Is Important

Without CI, developers may manually perform:

```text
Pull Code
Install Node.js
Install Dependencies
Run Linter
Run Tests
Build Application
Check Security
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

# Node.js CI Workflow

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
    +-- Node.js Setup
    +-- Dependency Install
    +-- Lint
    +-- Test
    +-- SonarQube
    +-- Security Scan
    +-- Build
    |
    ↓
CI Success
```

---

# Node.js Runtime

Node.js is a JavaScript runtime built on the V8 JavaScript engine.

It allows JavaScript to run outside the browser.

Typical application:

```text
Node.js
   |
   ↓
JavaScript / TypeScript
   |
   ↓
Backend Application
   |
   ↓
API / Microservice
```

---

# Node.js Version

The CI pipeline should use a supported Node.js version.

Examples:

```text
Node.js 18
Node.js 20
Node.js 22
```

The exact version should match the application's requirements.

Check the installed version:

```bash
node --version
```

Check npm:

```bash
npm --version
```

Example:

```text
node v20.x.x
npm 10.x.x
```

---

# Node.js Version Management

Node.js applications can define the expected runtime version.

Example:

```json
{
  "engines": {
    "node": ">=20"
  }
}
```

Another approach is to use a version manager or CI configuration.

Common tools include:

```text
nvm
Volta
asdf
```

The important point is that local and CI environments should use compatible versions.

---

# package.json

The main configuration file in many Node.js projects is:

```text
package.json
```

It can define:

```text
Project Name
Version
Scripts
Dependencies
Development Dependencies
Node.js Requirements
Package Metadata
```

Example:

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "scripts": {
    "test": "jest",
    "build": "npm run build",
    "lint": "eslint ."
  }
}
```

---

# package-lock.json

`package-lock.json` records the resolved dependency tree.

It helps provide more reproducible dependency installation.

Example:

```text
package.json
     |
     ↓
package-lock.json
     |
     ↓
Exact Resolved Dependencies
```

When using npm, CI commonly uses:

```bash
npm ci
```

instead of:

```bash
npm install
```

for clean, reproducible CI installations.

---

# npm

npm is the default package manager commonly distributed with Node.js.

Common commands:

```bash
npm install
npm ci
npm test
npm run build
npm run lint
npm audit
```

---

# npm install

Command:

```bash
npm install
```

It installs project dependencies based on the package configuration and lockfile when available.

It is commonly used during development.

---

# npm ci

Command:

```bash
npm ci
```

`npm ci` is designed for clean automated installations.

Typical CI flow:

```text
package.json
      |
      ↓
package-lock.json
      |
      ↓
npm ci
      |
      ↓
node_modules
```

It is generally preferred for CI when a compatible lockfile is committed.

---

# npm install vs npm ci

| Feature | `npm install` | `npm ci` |
|---|---|---|
| Common use | Development | CI |
| Uses lockfile | Yes, when present | Yes |
| Clean install | Not necessarily | Yes |
| Modifies lockfile | Can | No |
| Reproducibility | Lower | Higher |
| Speed | Can vary | Often faster in CI |

Interview answer:

> I prefer `npm ci` in CI pipelines because it performs a clean installation based on the lockfile and provides more predictable dependency installation.

---

# Node.js Project Structure

A typical project may look like:

```text
myapp/
├── package.json
├── package-lock.json
├── src/
│   ├── server.js
│   ├── routes/
│   └── services/
├── test/
├── Dockerfile
└── README.md
```

A TypeScript project may contain:

```text
src/
├── app.ts
├── routes/
└── services/
```

and compile to:

```text
dist/
```

---

# Node.js CI Pipeline Stages

A practical pipeline can contain:

```text
1. Checkout
2. Setup Node.js
3. Install Dependencies
4. Lint
5. Unit Tests
6. Code Quality
7. Security Scan
8. Build
9. Package
10. Publish Artifact
```

Example:

```text
Checkout
   |
   ↓
Node.js
   |
   ↓
npm ci
   |
   ↓
Lint
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
Build
   |
   ↓
Artifact
```

---

# Checkout Stage

The CI system first retrieves source code.

```text
Git Repository
      |
      ↓
CI Runner
      |
      ↓
Node.js Source
```

The exact checkout operation depends on the CI platform.

---

# Node.js Setup Stage

After checkout:

```text
Source Code
    |
    ↓
Node.js Setup
    |
    ↓
npm
```

Verify:

```bash
node --version
npm --version
```

If the required version is not available, the pipeline should fail early.

---

# Dependency Installation

For npm-based CI:

```bash
npm ci
```

Flow:

```text
package.json
      |
      ↓
package-lock.json
      |
      ↓
npm ci
      |
      ↓
node_modules
```

---

# Dependency Installation with Yarn

Some projects use Yarn.

Example:

```bash
yarn install --frozen-lockfile
```

Modern Yarn projects may use:

```bash
yarn install --immutable
```

The command depends on the project's Yarn version and configuration.

---

# Dependency Installation with pnpm

Some projects use pnpm.

Example:

```bash
pnpm install --frozen-lockfile
```

The package manager should match the project configuration.

---

# Lockfiles

Common lockfiles include:

```text
package-lock.json
yarn.lock
pnpm-lock.yaml
```

A project should normally use the lockfile appropriate to its package manager.

Lockfiles help maintain consistent dependency versions.

---

# Why Lockfiles Matter

Without a lockfile:

```text
Developer
   |
   ↓
Dependency Resolution
   |
   ↓
Version A
```

Another day:

```text
CI
   |
   ↓
Dependency Resolution
   |
   ↓
Version B
```

This can cause unexpected differences.

With a lockfile:

```text
package-lock.json
       |
       ↓
Resolved Versions
       |
       ↓
Developer
       |
       ↓
CI
```

This improves reproducibility.

---

# Node.js Linting

Linting checks code style and common coding problems.

Popular tools include:

```text
ESLint
```

Example:

```bash
npm run lint
```

Pipeline:

```text
Source Code
    |
    ↓
ESLint
    |
    ↓
Lint Result
```

If required lint checks fail:

```text
Lint Failure
    |
    ↓
CI Failed
```

---

# ESLint

ESLint is commonly used to identify:

```text
Syntax Problems
Coding Issues
Style Violations
Potential Bugs
```

Example script:

```json
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

Run:

```bash
npm run lint
```

---

# Unit Testing

Node.js applications can use testing frameworks such as:

```text
Jest
Mocha
Vitest
```

Example:

```bash
npm test
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

# Jest

Jest is a commonly used JavaScript testing framework.

Example:

```javascript
test('adds two numbers', () => {
    expect(2 + 3).toBe(5);
});
```

CI executes the tests automatically.

---

# Test Script

The `package.json` can define:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

Then:

```bash
npm test
```

runs the configured test command.

---

# Test Coverage

Code coverage measures how much of the code is exercised by tests.

Example:

```bash
npm test -- --coverage
```

Possible coverage metrics:

```text
Statements
Branches
Functions
Lines
```

Example:

```text
Statements: 92%
Branches:   85%
Functions:  90%
Lines:      93%
```

Coverage thresholds can be enforced if required.

---

# Integration Testing

Integration tests validate interactions between components.

Example:

```text
Node.js Service
      |
      +-- Database
      |
      +-- REST API
      |
      +-- Message Queue
      |
      +-- External Service
```

Unit tests may mock these dependencies.

Integration tests validate real or test versions of the integrations.

---

# API Testing

For Node.js microservices, API tests can validate:

```text
HTTP Status
Request Validation
Response Body
Authentication
Authorization
Error Handling
```

Example:

```text
POST /orders
      |
      ↓
Node.js API
      |
      ↓
Expected Response
```

---

# Node.js Build

Not every Node.js application requires a compilation step.

For plain JavaScript:

```text
Source
   |
   ↓
Tests
   |
   ↓
Package
```

For TypeScript or bundled applications:

```text
TypeScript
    |
    ↓
Build
    |
    ↓
JavaScript
    |
    ↓
dist/
```

---

# TypeScript CI

TypeScript projects commonly use:

```bash
npm run build
```

Example:

```json
{
  "scripts": {
    "build": "tsc"
  }
}
```

Flow:

```text
TypeScript
    |
    ↓
TypeScript Compiler
    |
    ↓
JavaScript
    |
    ↓
dist/
```

---

# TypeScript Type Checking

A CI pipeline can run:

```bash
npx tsc --noEmit
```

This checks types without producing compiled output.

Flow:

```text
TypeScript
    |
    ↓
Type Checker
    |
    ↓
Errors / Success
```

---

# Node.js Build Output

Depending on the application, build output may be:

```text
dist/
build/
.next/
lib/
```

Example:

```text
src/
 |
 ↓
Build
 |
 ↓
dist/
 ├── server.js
 ├── routes/
 └── services/
```

---

# npm Scripts

A project can define standard CI commands in `package.json`.

Example:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

CI can then use:

```bash
npm run lint
npm test
npm run build
```

This keeps build logic inside the project configuration.

---

# Node.js CI Quality

SonarQube can analyze JavaScript and TypeScript projects.

Typical flow:

```text
Source
   |
   ↓
Build / Test
   |
   ↓
SonarQube
   |
   ↓
Quality Gate
```

It can identify:

```text
Bugs
Vulnerabilities
Code Smells
Duplications
Coverage Information
```

---

# Node.js CI with SonarQube

The exact integration depends on the SonarQube setup.

Conceptually:

```text
npm ci
   |
   ↓
npm test
   |
   ↓
SonarQube Analysis
   |
   ↓
Quality Gate
```

The pipeline should stop when a mandatory quality gate fails.

---

# Node.js Security Scanning

Security checks can be performed during CI.

Possible checks include:

```text
Dependency Vulnerabilities
Source Code Security
Container Vulnerabilities
Secrets
Configuration
```

Tools can include:

```text
Trivy
SonarQube
Veracode
npm audit
```

The exact toolset depends on organizational requirements.

---

# npm Audit

npm provides dependency vulnerability checking.

Command:

```bash
npm audit
```

It can report vulnerabilities in installed dependencies.

Example:

```text
Dependency
    |
    ↓
npm audit
    |
    ↓
Vulnerability
```

The organization should define how vulnerabilities are handled and whether particular severity levels fail CI.

---

# npm Audit Fix

Command:

```bash
npm audit fix
```

This can attempt to update dependencies to resolve known vulnerabilities.

However, updates should be reviewed and tested rather than blindly applied in production projects.

---

# Trivy Filesystem Scan

Trivy can scan the project filesystem.

Example:

```bash
trivy fs .
```

Conceptual flow:

```text
Node.js Project
      |
      ↓
Trivy
      |
      +-- Dependencies
      +-- Configuration
      +-- Vulnerabilities
      |
      ↓
Scan Result
```

---

# Trivy Container Scan

If the Node.js application is containerized:

```bash
trivy image myapp:v1.0.0
```

Flow:

```text
Node.js Application
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

# Veracode in Node.js CI

Veracode can be integrated into the security pipeline depending on the organization's implementation.

Example:

```text
Source
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
Veracode
   |
   ↓
Security Gate
```

---

# Node.js CI Security Flow

A practical security flow can be:

```text
Checkout
   |
   ↓
npm ci
   |
   ↓
npm audit / Dependency Scan
   |
   ↓
Build
   |
   ↓
SonarQube
   |
   ↓
Docker Build
   |
   ↓
Trivy Image Scan
   |
   ↓
Veracode
```

The exact order can vary according to the pipeline design.

---

# Node.js Packaging

A Node.js application can be packaged in different ways.

For example:

```text
Source Files
node_modules
package.json
package-lock.json
dist/
```

For containerized applications, the Docker image itself is often the deployable artifact.

---

# Node.js Docker Build

Typical flow:

```text
Node.js Source
     |
     ↓
npm ci
     |
     ↓
npm run build
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

# Node.js Dockerfile

Example:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY dist ./dist

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

The exact Dockerfile depends on the application architecture.

---

# Multi-Stage Node.js Docker Build

For applications that require a build step:

```dockerfile
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY --from=build /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

The build stage contains development dependencies while the runtime stage can contain only what is required to run the application.

---

# Why Multi-Stage Builds?

A multi-stage build can help:

```text
Reduce Image Size
Reduce Runtime Dependencies
Improve Security
Separate Build and Runtime
```

Flow:

```text
Source
 |
 ↓
Build Stage
 |
 ↓
dist/
 |
 ↓
Runtime Stage
 |
 ↓
Smaller Image
```

---

# Node.js Docker Image Optimization

Avoid unnecessary files in the runtime image.

Use:

```text
.gitignore
.dockerignore
```

Example `.dockerignore`:

```text
node_modules
.git
.gitignore
Dockerfile
npm-debug.log
coverage
```

The exact exclusions should match the build strategy.

---

# Node.js CI and ECR

A typical AWS flow:

```text
Node.js Source
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
<repository>:v1.0.0
```

---

# Node.js CI and Kubernetes

Deployment flow:

```text
Node.js Source
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

# Node.js CI and GitOps

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

The CI pipeline builds and validates the application while the GitOps process manages the desired Kubernetes state.

---

# Node.js CI and ArgoCD

A typical deployment flow:

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

ArgoCD reconciles the desired state stored in Git.

---

# Node.js CI with Jenkins

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

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

Additional security, quality, Docker, and publishing stages can be added.

---

# Node.js CI with GitHub Actions

Example:

```yaml
name: Node.js CI

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

---

# GitHub Actions Node.js CI with Coverage

Example:

```yaml
- name: Test
  run: npm test -- --coverage
```

The resulting coverage report can be collected by the CI system or used by quality analysis tools.

---

# Node.js CI with GitLab

Example:

```yaml
stages:
  - install
  - test
  - build

install:
  image: node:20
  stage: install
  script:
    - npm ci

test:
  image: node:20
  stage: test
  script:
    - npm test

build:
  image: node:20
  stage: build
  script:
    - npm run build
```

In a real pipeline, caching and artifact handling can be added.

---

# Node.js Environment Variables

Applications commonly use environment variables for configuration.

Examples:

```text
NODE_ENV
PORT
DATABASE_URL
API_URL
LOG_LEVEL
```

Do not hardcode sensitive values.

Avoid:

```javascript
const password = "my-secret-password";
```

Prefer:

```javascript
const password = process.env.DB_PASSWORD;
```

Secrets should be injected securely by the CI/CD or runtime platform.

---

# NODE_ENV

Common values include:

```text
development
test
production
```

Example:

```bash
NODE_ENV=test npm test
```

Production:

```bash
NODE_ENV=production node dist/server.js
```

The exact environment behavior depends on the application.

---

# CI Secrets

Sensitive values may include:

```text
NPM Token
AWS Credentials
ECR Credentials
SonarQube Token
Veracode Credentials
Database Credentials
API Keys
```

Do not store these directly in:

```text
package.json
Source Code
Dockerfile
Jenkinsfile
GitHub Actions YAML
```

Use the CI platform's secret management features.

---

# npm Registry Authentication

If a project uses a private npm registry, CI may require authentication.

A common configuration uses:

```text
.npmrc
```

Credentials should be injected securely.

Avoid committing tokens such as:

```text
//registry.example.com/:_authToken=SECRET
```

to source control.

---

# Node.js Dependency Caching

Without cache:

```text
CI Run
   |
   ↓
npm ci
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
   +-- Hit → Reuse Cache
   |
   +-- Miss → Download
   |
   ↓
Build
```

Caching can significantly reduce dependency installation time.

---

# npm Cache

npm cache location depends on the environment.

Check it using:

```bash
npm config get cache
```

CI platforms can cache npm data to speed up builds.

---

# GitHub Actions npm Cache

Example:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
```

This enables npm dependency caching through the action's supported cache mechanism.

---

# Node.js CI Build Reproducibility

A reproducible build should control:

```text
Node.js Version
Package Manager Version
Dependency Versions
Lockfile
Build Commands
Environment
```

Example:

```text
Node.js 20
npm
package-lock.json
npm ci
```

---

# packageManager Field

A project can specify its package manager.

Example:

```json
{
  "packageManager": "npm@10"
}
```

The exact version should match the project requirements.

This can help developers and CI use a consistent package manager version.

---

# Node.js CI Pipeline Failure

The pipeline should fail when required checks fail.

Examples:

```text
Dependency Installation Failure
Lint Failure
Test Failure
Build Failure
Quality Gate Failure
Security Gate Failure
Docker Build Failure
Image Scan Failure
Artifact Publishing Failure
```

Example:

```text
npm ci ✓
   |
npm run lint ✗
   |
Pipeline Failed
```

---

# Fail Fast

CI should identify failures as early as practical.

Example:

```text
Checkout
   |
   ↓
npm ci
   |
   ↓
Lint
   |
   ↓
Test
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

If dependency installation fails, later stages should not run.

---

# Node.js CI Success

Successful pipeline:

```text
Checkout ✓
   |
Node.js Setup ✓
   |
npm ci ✓
   |
Lint ✓
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

# Node.js CI Troubleshooting

## npm ci Fails

Possible causes:

```text
package-lock.json Mismatch
Node.js Version Problem
Private Registry Authentication
Network Problem
Unavailable Dependency
Corrupted Cache
```

Check:

```bash
node --version
npm --version
npm ci
```

---

# package-lock.json Mismatch

If `package.json` and `package-lock.json` are inconsistent, `npm ci` can fail.

Typical solution:

```bash
npm install
```

Update the lockfile locally, review the changes, and commit the correct lockfile.

Then CI can run:

```bash
npm ci
```

---

# Node.js Version Mismatch

Example:

```text
Local:
Node.js 20

CI:
Node.js 18
```

This can produce different behavior.

Check:

```bash
node --version
```

Ensure CI uses the expected version.

---

# npm Registry Failure

Possible error:

```text
401 Unauthorized
```

Possible causes:

```text
Invalid Token
Missing Token
Wrong Registry
Expired Credentials
Incorrect .npmrc
```

Check the CI secret and registry configuration.

---

# Test Failure

Example:

```text
Tests: 100
Passed: 98
Failed: 2
```

The pipeline should fail if those tests are mandatory.

Investigate:

```text
Test Logs
Environment
Dependencies
Test Data
External Services
Timing
Parallel Execution
```

---

# Build Failure

For TypeScript:

```bash
npm run build
```

Possible causes:

```text
Type Errors
Missing Dependencies
Incorrect Configuration
Compilation Errors
Build Script Problems
```

Check:

```bash
npm run build
```

locally using the same Node.js and package manager versions as CI.

---

# Slow npm ci

Possible causes:

```text
Large Dependency Tree
Slow Registry
No Cache
Network Issues
CI Runner Performance
```

Possible improvements:

```text
Dependency Caching
Use npm ci
Reduce Unnecessary Dependencies
Use Efficient Registry Configuration
Optimize CI Runner
```

---

# Large node_modules

A large `node_modules` directory can slow:

```text
Dependency Installation
Docker Build
File Scanning
Artifact Handling
```

Use:

```text
npm ci --omit=dev
```

for a production runtime installation where appropriate.

---

# Production Dependencies

Development dependencies may include:

```text
Jest
ESLint
TypeScript
Build Tools
Testing Libraries
```

Production dependencies may include:

```text
Express
Fastify
Database Drivers
Application Libraries
```

For a runtime container, you may install only production dependencies:

```bash
npm ci --omit=dev
```

---

# Development vs Production Dependencies

Example:

```json
{
  "dependencies": {
    "express": "..."
  },
  "devDependencies": {
    "jest": "...",
    "eslint": "...",
    "typescript": "..."
  }
}
```

Conceptually:

```text
Development
    |
    +-- dependencies
    +-- devDependencies

Production
    |
    +-- dependencies
```

The exact dependency classification should match the application.

---

# Node.js CI and Docker

A complete pipeline can be:

```text
Checkout
   |
   ↓
Node.js Setup
   |
   ↓
npm ci
   |
   ↓
Lint
   |
   ↓
Tests
   |
   ↓
SonarQube
   |
   ↓
Security Scan
   |
   ↓
npm run build
   |
   ↓
Docker Build
   |
   ↓
Trivy Image Scan
   |
   ↓
ECR
```

---

# Node.js CI and ECR

Example:

```text
Node.js Source
      |
      ↓
CI
      |
      ↓
Docker Build
      |
      ↓
myapp:v1.2.0
      |
      ↓
Trivy
      |
      ↓
Amazon ECR
```

The image should be versioned rather than relying only on `latest`.

---

# Node.js CI and Kubernetes

Example:

```text
Node.js Source
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
Kubernetes Deployment
      |
      ↓
EKS
```

With GitOps:

```text
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

# Node.js CI and Microservices

For a microservices platform:

```text
user-service
catalogue-service
cart-service
payment-service
order-service
inventory-service
```

Each service can have a CI process:

```text
Service Source
     |
     ↓
npm ci
     |
     ↓
Test
     |
     ↓
Build
     |
     ↓
Docker
     |
     ↓
ECR
```

This supports independent service validation and deployment.

---

# Monorepo Node.js CI

A monorepo may contain:

```text
services/
├── user
├── payment
├── cart
└── orders
```

The pipeline can determine which services changed.

Conceptually:

```text
Pull Request
     |
     ↓
Detect Changes
     |
     +-- user changed
     |
     +-- payment changed
     |
     ↓
Run Relevant CI
```

This can reduce unnecessary builds.

---

# Node.js Polyrepo CI

In a polyrepo architecture:

```text
user-service-repo
payment-service-repo
cart-service-repo
orders-service-repo
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

# Node.js CI and Git Tags

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

The CI pipeline can trigger on:

```text
v*
```

For example:

```text
v1.0.0
v1.1.0
v2.0.0
```

---

# Node.js Release Versioning

Node.js applications can use Semantic Versioning:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v2.4.1
```

Meaning:

```text
2 → Breaking changes
4 → Backward-compatible features
1 → Bug fixes
```

---

# Node.js Artifact Versioning

For packaged applications:

```text
myapp-1.0.0.tgz
```

For containers:

```text
myapp:v1.0.0
```

For releases:

```text
v1.0.0
```

A consistent version should connect the release components.

---

# Commit to Deployment Traceability

A mature pipeline should provide:

```text
Git Commit
    |
    ↓
Git Tag
    |
    ↓
CI Build
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
    |
    ↓
Production
```

Example:

```text
Commit:
8f3a91d

Tag:
v1.2.0

Image:
myapp:v1.2.0

Deployment:
Production
```

---

# Node.js CI Pipeline Optimization

A practical optimization process:

```text
1. Measure Pipeline Duration
2. Identify Slow Stages
3. Cache npm Dependencies
4. Use npm ci
5. Optimize Tests
6. Avoid Unnecessary Builds
7. Use Efficient Docker Builds
8. Use Appropriate CI Resources
```

Measure before optimizing.

---

# Dependency Caching

Dependency caching can improve build speed.

Without cache:

```text
npm ci
 |
 ↓
Download Everything
 |
 ↓
Build
```

With cache:

```text
npm ci
 |
 ↓
Reuse Cached Packages
 |
 ↓
Build
```

Caching should be invalidated when dependency definitions change.

---

# Cache Key

A good cache key can include dependency lockfile information.

Conceptually:

```text
OS + Node Version + package-lock.json Hash
```

Example:

```text
linux-node20-<lockfile-hash>
```

This helps prevent incompatible dependencies from being reused.

---

# Node.js CI Security Best Practices

```text
Use Supported Node.js Versions
Commit Lockfiles
Use npm ci in CI
Scan Dependencies
Run SAST Where Required
Scan Container Images
Protect CI Secrets
Avoid Hardcoded Credentials
Use Minimal Runtime Images
Use Non-Root Containers Where Appropriate
Keep Dependencies Updated
Enforce Security Gates
```

---

# Node.js CI Best Practices

```text
Use a Supported Node.js Version
Use npm ci for CI
Commit the Lockfile
Run Linting
Run Unit Tests
Run Integration Tests Where Required
Use Code Quality Analysis
Run Dependency Security Checks
Run Container Scanning
Use Versioned Docker Images
Cache Dependencies
Keep CI Configuration in Git
Secure Secrets
Use Reproducible Builds
Publish Versioned Artifacts
Maintain Commit-to-Deployment Traceability
```

---

# Common Node.js CI Mistakes

## Using npm install in CI

Prefer:

```bash
npm ci
```

when the project uses npm and has a compatible lockfile.

---

## Ignoring the Lockfile

Avoid removing:

```text
package-lock.json
```

without a valid reason.

The lockfile helps provide consistent dependency resolution.

---

## Using latest Node.js Automatically

Do not allow the CI runtime to change unexpectedly.

Specify the expected version.

---

## Hardcoding Secrets

Never commit:

```text
API Keys
Passwords
Tokens
Cloud Credentials
Registry Credentials
```

---

## Skipping Tests

Avoid deploying code that has not passed the required automated tests.

---

## Ignoring Security Scans

Dependency and container vulnerabilities should be detected during CI according to organizational policy.

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

or another immutable image reference.

---

# Node.js CI Production Checklist

```text
☐ Node.js version defined
☐ Package manager version controlled
☐ package.json committed
☐ Lockfile committed
☐ npm ci used in CI
☐ Linting enabled
☐ Unit tests enabled
☐ Integration tests enabled where required
☐ Test reports published
☐ SonarQube configured
☐ Security scanning configured
☐ Dependency vulnerabilities checked
☐ Docker image scanned
☐ Secrets stored securely
☐ Build output validated
☐ Versioned artifacts/images created
☐ ECR publishing configured
☐ GitOps integration configured
☐ Rollback version available
☐ CI logs available
```

---

# Node.js CI Interview Questions

## Basic

1. What is Node.js?
2. What is Continuous Integration?
3. What is npm?
4. What is `package.json`?
5. What is `package-lock.json`?
6. What is the difference between `npm install` and `npm ci`?
7. How do you check the Node.js version?
8. How do you check the npm version?
9. What is `node_modules`?
10. What is `npm run build`?
11. What is `npm test`?
12. What is ESLint?
13. What is Jest?
14. What is a lockfile?
15. Why is `npm ci` preferred in CI?

---

# Intermediate Interview Questions

16. How would you design a Node.js CI pipeline?

17. What stages would you include in a Node.js CI pipeline?

18. How do you manage Node.js dependencies in CI?

19. How do you cache npm dependencies?

20. How do you run linting in CI?

21. How do you run unit tests?

22. How do you generate test coverage?

23. How do you integrate SonarQube?

24. How do you scan Node.js dependencies for vulnerabilities?

25. How do you integrate Trivy?

26. How do you build a Docker image for a Node.js application?

27. How do you optimize a Node.js Docker image?

28. How do you handle Node.js version mismatches?

29. How do you troubleshoot `npm ci` failures?

30. How do you publish a Node.js application to ECR?

---

# Advanced Interview Questions

31. Design a production-grade Node.js CI pipeline.

32. How would you implement DevSecOps for a Node.js application?

33. How would you integrate SonarQube, Trivy, npm audit, and Veracode?

34. How would you implement security gates?

35. How would you implement quality gates?

36. How would you optimize a Node.js pipeline that takes 20 minutes?

37. How would you handle a vulnerable npm dependency?

38. How would you make Node.js builds reproducible?

39. How would you securely provide npm registry credentials?

40. How would you build a multi-stage Docker image for Node.js?

41. How would you deploy a Node.js container to EKS?

42. How would you connect Node.js CI with GitOps and ArgoCD?

43. How would you handle a failed production deployment?

44. How would you implement rollback for a Node.js application?

45. How would you maintain commit-to-image-to-deployment traceability?

---

# Scenario Question

## The Node.js CI pipeline fails during npm ci. How would you troubleshoot it?

I would troubleshoot it systematically.

```text
npm ci Failure
      |
      ↓
Check Node.js Version
      |
      ↓
Check npm Version
      |
      ↓
Check package.json
      |
      ↓
Check Lockfile
      |
      ↓
Check Registry
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
node --version
npm --version
npm ci
```

If the project uses a private registry, I would also verify the registry configuration and credentials.

---

# Scenario Question

## The application works locally but fails in CI. What would you check?

I would compare:

```text
Node.js Version
npm Version
Lockfile
Environment Variables
Operating System
Dependencies
Build Commands
Secrets
Network
External Services
```

Commands:

```bash
node --version
npm --version
npm ci
npm test
npm run build
```

I would reproduce the CI commands locally using the same runtime versions where possible.

---

# Scenario Question

## The Node.js CI pipeline takes 15 minutes. How would you optimize it?

I would first measure each stage.

```text
npm ci        → 5 min
Tests         → 6 min
Build         → 2 min
SonarQube     → 1 min
Security      → 1 min
```

Then optimize the bottlenecks.

Possible improvements:

```text
npm Cache
npm ci
Test Optimization
Parallel Independent Checks
Docker Layer Caching
Reduce Unnecessary Work
Improve CI Runner Resources
```

The optimization should be based on measured pipeline behavior.

---

# Scenario Question

## A critical vulnerability is found in an npm dependency. What would you do?

I would:

```text
1. Identify the Vulnerable Dependency
2. Identify the Vulnerable Version
3. Check the Fixed Version
4. Review Compatibility
5. Update the Dependency
6. Update Lockfile
7. Run Unit Tests
8. Run Integration Tests
9. Run Security Scans Again
10. Merge After Required Gates Pass
```

If no immediate fixed version exists, I would follow the organization's vulnerability exception or mitigation process.

---

# Scenario Question

## How would you implement Node.js CI for an EKS microservices platform?

Example:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Checkout
    +-- Node.js Setup
    +-- npm ci
    +-- Lint
    +-- Unit Tests
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

## How would you reduce the size of a Node.js Docker image?

I would consider:

```text
Use Minimal Base Image
Use Multi-Stage Build
Install Production Dependencies Only
Use .dockerignore
Remove Build Artifacts Not Needed at Runtime
Avoid Unnecessary Packages
```

Example:

```bash
npm ci --omit=dev
```

for the runtime stage where appropriate.

---

# Scenario Question

## How would you secure secrets in a Node.js CI pipeline?

I would:

```text
Store Secrets in CI Secret Manager
Inject Them at Runtime
Avoid Hardcoding
Avoid Committing .env Files
Restrict Secret Access
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
Node.js Application
```

---

# Scenario Question

## How would you connect Node.js CI to ArgoCD?

Example:

```text
Node.js Source
      |
      ↓
CI
      |
      ↓
Build
      |
      ↓
Docker Image
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

# Complete Node.js CI Architecture

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
                         Setup Node.js
                               |
                               ↓
                             npm ci
                               |
                               ↓
                             ESLint
                               |
                               ↓
                           Unit Tests
                               |
                               ↓
                           SonarQube
                               |
                               ↓
                         Security Scan
                               |
                               ↓
                           npm build
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

# Quick Revision

Node.js CI:

```text
Git
 ↓
Checkout
 ↓
Node.js
 ↓
npm ci
 ↓
Lint
 ↓
Test
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
node --version
npm --version
npm install
npm ci
npm test
npm run lint
npm run build
npm audit
npm audit fix
npm ci --omit=dev
```

Important files:

```text
package.json
package-lock.json
Dockerfile
.dockerignore
```

Important concepts:

```text
npm ci
Lockfiles
Dependency Caching
Linting
Unit Testing
Code Coverage
SonarQube
Trivy
Veracode
Docker
ECR
GitOps
ArgoCD
EKS
```

Core idea:

> Node.js CI automates dependency installation, linting, testing, code-quality analysis, security scanning, building, and packaging so that Node.js application changes are validated consistently before moving toward deployment.