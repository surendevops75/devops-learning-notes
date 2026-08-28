# NPM-Repositories

## 1. Purpose

This file covers NPM repositories in JFrog Artifactory from fundamentals through production DevOps usage.

It covers:

- NPM registry fundamentals
- Artifactory NPM local, remote and virtual repositories
- package.json
- package-lock.json
- npm configuration
- scoped packages
- npm install / ci / publish
- authentication
- tokens and CI credentials
- dependency resolution
- npm cache
- private packages
- public package proxying
- package versioning
- semantic versioning
- package immutability
- Jenkins
- GitHub Actions
- GitLab CI
- Docker builds
- Kubernetes workloads
- security and supply-chain controls
- troubleshooting
- production architecture
- interview preparation

---

# PART I — NPM FUNDAMENTALS

## 2. What Is NPM?

NPM is the package ecosystem and command-line tooling commonly used by JavaScript and Node.js applications.

Typical operations:

```text
install
update
publish
pack
audit
run scripts
```

A typical project contains:

```text
package.json
package-lock.json
src/
```

---

## 3. NPM and Artifactory

Artifactory can provide:

```text
private NPM package storage
approved public package proxying
NPM dependency caching
access control
artifact governance
CI/CD publication
```

Typical architecture:

```text
NPM Client
    |
    v
npm-virtual
   /      \
  /        \
npm-local  npm-remote
              |
              v
          npm Registry
```

---

## 4. Why Use Artifactory for NPM?

Without an internal repository:

```text
Developer / CI
      |
      v
Public npm registry
```

With Artifactory:

```text
Developer / CI
      |
      v
Artifactory
      |
 +----+----+
 |         |
Local    Remote
 |         |
Private  Public
Packages Registry
```

Benefits:

```text
centralized dependency access
private package management
caching
reduced internet dependency
security controls
auditability
```

---

## 5. NPM Package

A package normally contains:

```text
package.json
JavaScript/TypeScript files
README
license
dependencies
```

It can be published as an NPM package.

---

## 6. package.json

Example:

```json
{
  "name": "@company/payment-client",
  "version": "4.2.1",
  "description": "Internal payment client",
  "main": "dist/index.js",
  "scripts": {
    "test": "npm test",
    "build": "npm run build"
  },
  "dependencies": {
    "axios": "^1.0.0"
  }
}
```

---

## 7. NPM Package Name

A package can be:

```text
payment-client
```

or scoped:

```text
@company/payment-client
```

Scoped packages are especially useful for organization-owned packages.

---

## 8. NPM Scope

A scope creates a namespace.

Example:

```text
@company/*
```

Possible packages:

```text
@company/ui
@company/payment-client
@company/auth
```

This helps distinguish internal packages from public packages.

---

## 9. Why Use Scopes?

Benefits:

```text
namespace ownership
clear internal identity
reduced naming collision
dependency-confusion protection
```

Use an organization-controlled scope where appropriate.

---

## 10. Semantic Versioning

NPM commonly uses semantic versioning:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
4.2.1
```

Meaning:

```text
4 = major
2 = minor
1 = patch
```

The exact compatibility policy should be defined by the team.

---

## 11. Version Examples

```text
1.0.0
1.1.0
1.1.1
2.0.0
```

Production packages should use controlled versions.

---

## 12. NPM Dependency Ranges

Examples:

```text
^4.2.1
~4.2.1
4.2.1
```

These have different update semantics.

For reproducible CI, `package-lock.json` is important.

---

## 13. package-lock.json

The lock file records resolved dependency information.

It helps ensure:

```text
developer
CI
production
```

resolve consistent dependency versions.

---

## 14. npm install vs npm ci

Common distinction:

```text
npm install
```

can update dependency metadata/lockfile depending on project state.

```text
npm ci
```

is designed for clean, reproducible CI installations using the lock file.

For CI pipelines, `npm ci` is commonly preferred when a valid lockfile is committed.

---

## 15. NPM Local Cache

NPM maintains a local cache on the client/build environment.

This is different from:

```text
Artifactory npm-local
```

and:

```text
Artifactory npm-remote cache
```

Three different concepts can exist:

```text
Developer/CI npm cache
Artifactory local repository
Artifactory remote cache
```

---

# PART II — ARTIFACTORY NPM REPOSITORIES

## 16. NPM Local Repository

Used for private packages.

Example:

```text
npm-local
```

Publishing flow:

```text
npm package
   |
   v
npm publish
   |
   v
npm-local
```

---

## 17. NPM Remote Repository

Used to proxy an external NPM registry.

Conceptually:

```text
npm
 ↓
npm-remote
 ↓
public npm registry
```

Artifactory can cache retrieved packages.

---

## 18. NPM Virtual Repository

Provides a unified endpoint:

```text
npm-virtual
   /      \
local     remote
```

Consumers configure only:

```text
npm-virtual
```

---

## 19. Recommended NPM Architecture

```text
                    Developers / CI
                           |
                           v
                      npm-virtual
                       /        \
                      /          \
                     v            v
                 npm-local     npm-remote
                                  |
                                  v
                              npm registry
```

---

## 20. Dependency Resolution

Internal dependency:

```text
@company/payment-client
```

may be resolved from:

```text
npm-local
```

External dependency:

```text
express
```

may be resolved through:

```text
npm-remote
```

The consumer can use:

```text
npm-virtual
```

as the common endpoint.

---

## 21. Publishing Private Package

Example:

```bash
npm publish
```

The configured registry determines where the package is published.

In production CI, explicitly configure the approved Artifactory endpoint.

---

## 22. NPM Registry Configuration

A common configuration pattern is:

```bash
npm config set registry https://artifactory.company.com/artifactory/api/npm/npm-virtual/
```

The exact URL must match the Artifactory deployment and repository configuration.

---

## 23. .npmrc

NPM commonly uses:

```text
.npmrc
```

for registry and authentication configuration.

Example concept:

```text
registry=https://artifactory.company.com/...
```

Never commit plaintext production credentials.

---

## 24. Scoped Registry

A common enterprise pattern is:

```text
@company:registry=https://artifactory.company.com/...
```

This allows organization-owned packages to use a controlled registry.

---

## 25. Why Scoped Registry Is Useful

It can provide:

```text
private package routing
namespace ownership
clear separation
reduced dependency confusion risk
```

The exact routing model should be tested for the organization's NPM setup.

---

# PART III — NPM AUTHENTICATION

## 26. Authentication Model

There can be:

```text
NPM Client
   ↓
Artifactory
```

and:

```text
Artifactory
   ↓
Public NPM Registry
```

These are separate authentication relationships.

---

## 27. CI Authentication

A CI job should use:

```text
service identity
token
secret
```

through the CI platform's secure credential mechanism.

---

## 28. Do Not Hardcode Tokens

Bad:

```text
registry=https://... 
_authToken=production-token
```

inside a committed repository.

Better:

```text
CI Secret
 ↓
temporary npm configuration
 ↓
npm
 ↓
Artifactory
```

---

## 29. Token Rotation

Recommended:

```text
Create replacement token
 ↓
Update CI
 ↓
Run test build
 ↓
Confirm successful publish/download
 ↓
Revoke old token
```

---

## 30. Least Privilege

A build that only downloads packages needs:

```text
READ
```

A publishing pipeline needs:

```text
READ
+
DEPLOY
```

It normally does not need:

```text
ADMIN
DELETE
```

---

## 31. NPM Authentication Failure

Typical symptom:

```text
401 Unauthorized
```

Check:

```text
.npmrc
token
registry
scope
CI secret
token expiration
```

---

## 32. NPM Authorization Failure

Typical:

```text
403 Forbidden
```

Check:

```text
repository permissions
package namespace
project access
deploy permission
token scope
```

---

# PART IV — PRIVATE NPM PACKAGES

## 33. Private Package Architecture

```text
Developer
   ↓
@company/ui-library
   ↓
npm-local
```

Consumer:

```text
Application
   ↓
npm-virtual
   ↓
npm-local
```

---

## 34. Internal Package Example

```text
@company/auth-client
```

Version:

```text
3.4.0
```

Publication:

```text
npm publish
```

Target:

```text
npm-local
```

---

## 35. Internal Package Governance

Define:

```text
owner
versioning
API compatibility
security
retention
documentation
deprecation
```

---

## 36. Private Package Lifecycle

```text
Develop
 ↓
Test
 ↓
Build
 ↓
Publish
 ↓
Security Scan
 ↓
Release
 ↓
Consume
 ↓
Deprecate
```

---

## 37. Package Immutability

For production releases:

```text
@company/auth-client@3.4.0
```

should identify a stable package version.

Avoid replacing published production content under the same version.

---

## 38. Why Immutability Matters

It supports:

```text
reproducibility
rollback
incident investigation
software provenance
```

---

# PART V — NPM REMOTE REPOSITORIES

## 39. Public Registry Proxy

Architecture:

```text
Application
   ↓
npm-virtual
   ↓
npm-remote
   ↓
Public Registry
```

---

## 40. Remote Cache

First request:

```text
npm
 ↓
Artifactory
 ↓
Upstream
 ↓
Cache
```

Later:

```text
npm
 ↓
Artifactory Cache
```

---

## 41. Cache Miss

If a package is not cached:

```text
Artifactory
 ↓
Upstream
```

If upstream is unavailable:

```text
Dependency resolution may fail
```

---

## 42. Public Registry Outage

If:

```text
npm registry
```

is unavailable:

```text
cached packages
→ may continue working

uncached packages
→ may fail
```

Do not assume the remote cache is a complete mirror.

---

## 43. Upstream Package Risk

Public packages can contain:

```text
vulnerabilities
malware
typosquatting
dependency confusion
license concerns
```

Use approved repository access and security controls.

---

## 44. Approved Remote Registry

Enterprise pattern:

```text
Security Review
      ↓
Approved NPM Remote
      ↓
NPM Virtual
      ↓
CI / Developers
```

---

# PART VI — PACKAGE RESOLUTION

## 45. Internal Package Resolution

Example:

```text
@company/ui
```

Flow:

```text
npm
 ↓
npm-virtual
 ↓
npm-local
 ↓
@company/ui
```

---

## 46. External Package Resolution

Example:

```text
express
```

Flow:

```text
npm
 ↓
npm-virtual
 ↓
npm-remote
 ↓
public registry
```

---

## 47. Dependency Tree

A project can contain:

```text
application
 ├── express
 │    ├── dependency A
 │    └── dependency B
 └── internal client
      └── dependency C
```

NPM resolves the complete dependency graph.

---

## 48. package-lock and Reproducibility

A committed lockfile helps ensure CI resolves the intended dependency graph.

Recommended:

```text
package.json
+
package-lock.json
```

for npm projects where lockfiles are appropriate.

---

## 49. npm ci Production Pattern

```text
Checkout
 ↓
npm ci
 ↓
Tests
 ↓
Security Scan
 ↓
Build
 ↓
Package
 ↓
Publish
```

---

## 50. npm install Production Consideration

`npm install` may modify the lockfile when dependency metadata differs.

For deterministic CI, use a clean workspace and a validated lockfile with:

```bash
npm ci
```

---

# PART VII — SEMANTIC VERSIONING

## 51. Semantic Versioning

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.5.3
```

---

## 52. Major Version

Usually represents incompatible API changes.

Example:

```text
2.x → 3.x
```

---

## 53. Minor Version

Usually adds backward-compatible functionality.

Example:

```text
2.4 → 2.5
```

---

## 54. Patch Version

Usually contains backward-compatible fixes.

Example:

```text
2.5.2 → 2.5.3
```

---

## 55. Version Governance

For production packages:

```text
version
release notes
compatibility
security status
owner
```

should be traceable.

---

# PART VIII — NPM + JENKINS

## 56. Jenkins Architecture

```text
Git
 ↓
Jenkins
 ↓
Node.js
 ↓
npm ci
 ↓
Build/Test
 ↓
Artifactory
```

---

## 57. Jenkins Dependency Flow

```text
Jenkins
 ↓
npm
 ↓
npm-virtual
 ↓
local + remote
```

---

## 58. Jenkins Publish Flow

```text
Jenkins
 ↓
npm publish
 ↓
npm-local
```

---

## 59. Jenkins Credential Pattern

```text
Jenkins Credentials
        ↓
npm configuration
        ↓
npm
        ↓
Artifactory
```

Avoid credentials in:

```text
Jenkinsfile
package.json
source code
```

---

## 60. Jenkins Pipeline Concept

```groovy
pipeline {
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
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

        stage('Publish') {
            steps {
                sh 'npm publish'
            }
        }
    }
}
```

Production pipelines should add security gates and controlled publication.

---

# PART IX — NPM + GITHUB ACTIONS

## 61. GitHub Actions Flow

```text
GitHub
 ↓
GitHub Actions
 ↓
npm ci
 ↓
Test
 ↓
Build
 ↓
Security
 ↓
npm publish
 ↓
Artifactory
```

---

## 62. GitHub Secrets

Use:

```text
GitHub Secrets
```

for credentials/tokens where required.

Never commit:

```text
NPM tokens
Artifactory passwords
API keys
```

---

## 63. GitHub Actions Registry Configuration

Conceptually:

```yaml
- run: npm ci
- run: npm test
- run: npm run build
- run: npm publish
```

Registry and authentication should be injected securely.

---

# PART X — NPM + GITLAB

## 64. GitLab Flow

```text
GitLab
 ↓
Runner
 ↓
npm ci
 ↓
Test
 ↓
Build
 ↓
Scan
 ↓
Publish
```

---

## 65. GitLab CI Variables

Use:

```text
masked variables
protected variables
environment controls
```

for Artifactory credentials.

---

# PART XI — NPM + DOCKER

## 66. Node.js Container Build

Typical:

```text
package.json
package-lock.json
       ↓
npm ci
       ↓
npm run build
       ↓
Docker image
```

---

## 67. Multi-Stage Docker Build

Conceptually:

```dockerfile
FROM node:22 AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

Use organization-approved base images in production.

---

## 68. Why npm Repository Matters to Docker

If Docker build runs:

```text
npm ci
```

the container build depends on the NPM repository.

Therefore:

```text
NPM availability
=
application image build dependency
```

---

## 69. Docker Build Failure

If:

```text
npm ci
```

fails inside the Docker build:

Check:

```text
registry
DNS
network
credentials
package
lockfile
Artifactory
```

---

# PART XII — NPM + KUBERNETES

## 70. Kubernetes Runtime

Kubernetes usually runs the built application rather than installing NPM packages at runtime.

Recommended:

```text
Build dependencies during CI
 ↓
Create immutable image
 ↓
Deploy image
```

---

## 71. Avoid npm install at Runtime

Bad:

```text
Pod starts
 ↓
npm install
 ↓
Public network
```

Better:

```text
CI
 ↓
npm ci
 ↓
Build image
 ↓
Registry
 ↓
Kubernetes
```

---

## 72. Runtime Architecture

```text
NPM Repository
      |
      v
CI Build
      |
      v
Docker Image
      |
      v
Artifactory Docker Repository
      |
      v
Kubernetes
```

---

# PART XIII — SECURITY

## 73. NPM Supply Chain

NPM dependencies are part of the software supply chain.

Security controls should cover:

```text
source
dependency
package
build
artifact
deployment
```

---

## 74. Dependency Scanning

Scan:

```text
direct dependencies
transitive dependencies
published packages
container images
```

---

## 75. npm audit

NPM provides:

```bash
npm audit
```

It can identify known dependency vulnerabilities according to the available advisory data.

Do not rely on one scanner alone for enterprise security.

---

## 76. Dependency Confusion Prevention

Use:

```text
@company/*
```

for private packages.

Control:

```text
public registry access
Artifactory remotes
virtual repositories
package naming
```

---

## 77. Typosquatting

Example:

```text
express
exprees
expresss
```

An attacker may publish a malicious package with a similar name.

Controls:

```text
approved dependencies
lockfiles
scanning
review
```

---

## 78. Malicious Package Response

If a malicious NPM package is discovered:

```text
Block package
 ↓
Identify affected builds
 ↓
Identify deployed applications
 ↓
Remove/replace dependency
 ↓
Rebuild
 ↓
Redeploy
 ↓
Investigate
```

---

## 79. Secret Leakage in NPM Packages

Before publishing:

```text
source review
secret scanning
package content review
```

Do not publish:

```text
.env
private keys
tokens
credentials
internal secrets
```

---

## 80. .npmignore

Use `.npmignore` or package configuration to prevent unnecessary files from being published.

But do not rely solely on `.npmignore` for secret protection.

---

## 81. npm pack Inspection

Before publishing, inspect package contents with:

```bash
npm pack --dry-run
```

This helps identify files that would be included.

---

## 82. Package Provenance

Production organizations should track:

```text
Git commit
CI pipeline
package version
dependencies
publisher
timestamp
```

Build metadata and CI provenance can support this.

---

# PART XIV — TROUBLESHOOTING

## 83. Troubleshooting Layers

Use:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
NPM registry configuration
 ↓
Authentication
 ↓
Authorization
 ↓
Repository
 ↓
Package
 ↓
Upstream
```

---

## 84. npm ERR! 401

Check:

```text
token
registry
.npmrc
scope
CI secret
token expiration
```

---

## 85. npm ERR! 403

Check:

```text
repository permission
package scope
deploy permission
token scope
```

---

## 86. npm ERR! 404

Check:

```text
package name
version
registry URL
virtual repository
local package
remote upstream
```

---

## 87. npm ERR! ERESOLVE

Often indicates dependency resolution conflicts.

Investigate:

```text
package.json
package-lock.json
dependency tree
peer dependencies
```

Avoid blindly using:

```bash
--legacy-peer-deps
```

as a permanent production fix.

---

## 88. npm ci Failure

Check:

```text
package-lock.json
package.json
Node version
npm version
registry
credentials
```

The lockfile must be compatible with the package manifest.

---

## 89. NPM Registry Not Used

If the build unexpectedly contacts the public registry directly:

Check:

```bash
npm config get registry
```

and:

```bash
npm config list
```

Also inspect project/user/global `.npmrc` files.

---

## 90. Scoped Package Uses Wrong Registry

Check:

```text
@company:registry
```

configuration and ensure the scope is mapped to the intended Artifactory endpoint.

---

## 91. Package Published Successfully but Cannot Install

Check:

```text
publication repository
virtual repository inclusion
package scope
version
permissions
```

---

## 92. Public Dependency Cannot Install

Check:

```text
npm-remote
upstream availability
cache
network egress
TLS
```

---

## 93. Artifactory Healthy but npm Fails

Possible cause:

```text
client configuration
wrong registry
bad token
wrong scope
lockfile
Node/npm version
```

---

## 94. npm Works Locally but Fails in CI

Common reason:

```text
developer cache
```

or:

```text
developer .npmrc
```

contains configuration/credentials not present in CI.

Standardize CI configuration.

---

# PART XV — PRODUCTION ARCHITECTURE

## 95. Standard Production Architecture

```text
                   Developers / CI
                          |
                          v
                     npm-virtual
                      /       \
                     /         \
                    v           v
               npm-local     npm-remote
                                |
                                v
                           Public Registry
```

---

## 96. CI Production Flow

```text
Git
 ↓
CI
 ↓
npm ci
 ↓
Tests
 ↓
Security Scan
 ↓
npm run build
 ↓
npm publish
 ↓
npm-local
 ↓
Build Info / Provenance
 ↓
Promotion
 ↓
Docker Image
```

---

## 97. Docker + NPM Production Flow

```text
Git
 ↓
CI
 ↓
npm ci
 ↓
Build
 ↓
Docker Build
 ↓
Security Scan
 ↓
Docker Local Repository
 ↓
Kubernetes
```

---

## 98. Enterprise NPM Architecture

```text
                  External NPM Registry
                           |
                           v
                      npm-remote
                           |
                           v
Security / Policy / Scanning
                           |
                           v
                      npm-virtual
                       /        \
                      /          \
             Developers          CI
                                   |
                                   v
                              npm-local
                                   |
                                   v
                             Release Flow
```

---

## 99. Multi-Team Architecture

```text
                  npm-virtual
                       |
        +--------------+--------------+
        |              |              |
     Team A          Team B        Platform
        |              |              |
    @company/a      @company/b     @company/common
```

Use scopes and permissions to establish ownership.

---

## 100. NPM Repository Capacity

Plan for:

```text
package count
package size
tarball growth
remote cache
download traffic
CI concurrency
retention
backup
```

---

## 101. NPM Performance

Important factors:

```text
network latency
Artifactory latency
storage I/O
package size
dependency graph
CI concurrency
cache hit rate
```

---

## 102. NPM Build Burst

Suppose:

```text
500 CI jobs
```

start simultaneously.

Potentially:

```text
thousands of npm package requests
```

can hit Artifactory.

The remote cache and Artifactory capacity reduce pressure on the public registry.

---

## 103. NPM Monitoring

Monitor:

```text
request rate
latency
HTTP errors
storage
cache activity
authentication failures
upstream failures
```

---

## 104. NPM Audit

Audit:

```text
package publication
deletions
permission changes
repository changes
authentication failures
```

---

# PART XVI — PRODUCTION SCENARIOS

## 105. Scenario — Public NPM Registry Down

If package is cached:

```text
npm
 ↓
Artifactory cache
```

Build may continue.

If uncached:

```text
npm
 ↓
Artifactory
 ↓
upstream unavailable
```

Build may fail.

---

## 106. Scenario — Private Package 404

Check:

```text
package scope
package name
version
npm-local
npm-virtual
permissions
```

---

## 107. Scenario — Publish 403

Check:

```text
token
deploy permission
package namespace
target repository
```

---

## 108. Scenario — npm ci Suddenly Fails

Check:

```text
package-lock.json
package.json
Node/npm version
registry
Artifactory
```

---

## 109. Scenario — CI Uses Public Registry

Run:

```bash
npm config get registry
```

Then inspect:

```text
.npmrc
environment variables
global npm config
project config
```

---

## 110. Scenario — Dependency Vulnerability

Response:

```text
Identify affected package
 ↓
Identify direct/transitive dependency
 ↓
Check fixed version
 ↓
Update package
 ↓
Regenerate lockfile
 ↓
Run tests
 ↓
Security scan
 ↓
Publish new artifact/image
 ↓
Deploy
```

---

## 111. Scenario — Malicious Package Published

Response:

```text
Quarantine/block
 ↓
Identify consumers
 ↓
Identify builds
 ↓
Identify deployments
 ↓
Replace dependency
 ↓
Rebuild
 ↓
Redeploy
 ↓
Audit
```

---

## 112. Scenario — Secret Accidentally Published

Immediate:

```text
Revoke secret
 ↓
Remove package from future consumption
 ↓
Identify consumers
 ↓
Rotate related credentials
 ↓
Investigate
```

Do not assume deleting the package alone invalidates an exposed secret.

---

# PART XVII — INTERVIEW PREPARATION

## 113. What Is an NPM Repository in Artifactory?

Answer:

```text
It is an Artifactory repository configured for NPM packages. It can
be local for private organization packages, remote for an external
NPM registry and virtual for a unified consumer endpoint.
```

---

## 114. Why Use npm-virtual?

Answer:

```text
It provides developers and CI with a stable registry endpoint while
Artifactory manages private packages and approved public dependencies
behind the scenes.
```

---

## 115. Where Do You Publish Private NPM Packages?

Answer:

```text
I publish private packages to an NPM local repository. Consumers
normally access them through an NPM virtual repository.
```

---

## 116. Why Use NPM Scopes?

Answer:

```text
Scopes create an organization-controlled namespace such as
@company/*, which improves package ownership, organization and
dependency-confusion protection.
```

---

## 117. Why Use npm ci in CI?

Answer:

```text
npm ci is designed for clean, reproducible dependency installation
from the lockfile. It avoids treating CI as a developer workstation
and helps ensure the dependency graph is consistent.
```

---

## 118. How Do You Secure NPM CI Authentication?

Answer:

```text
I use dedicated CI identities and secret-managed tokens, configure
the registry dynamically and avoid committing credentials in
.npmrc, source code or pipeline files.
```

---

## 119. How Do You Troubleshoot npm 401?

Answer:

```text
I check the registry URL, .npmrc configuration, token, scope
configuration, CI secret injection and token validity.
```

---

## 120. How Do You Troubleshoot npm 403?

Answer:

```text
I verify that authentication succeeded and then inspect repository
permissions, package scope, deploy permissions and token scope.
```

---

## 121. How Do You Troubleshoot npm 404?

Answer:

```text
I verify package name, scope, version, target repository, virtual
repository membership and whether the package exists locally or in
the configured remote source.
```

---

## 122. What Is the Difference Between npm Cache and Artifactory Remote Cache?

Answer:

```text
The npm client cache is local to the developer or CI environment.
Artifactory's remote cache is a shared repository-side cache that can
serve many consumers and reduce requests to the external registry.
```

---

## 123. How Do You Prevent Dependency Confusion?

Answer:

```text
I use organization-controlled scopes, approved remote repositories,
virtual repository governance, dependency scanning, lockfiles and
restricted direct access to public registries.
```

---

## 124. How Would You Design NPM for 100 Teams?

Answer:

```text
I would standardize npm-local, npm-remote and npm-virtual patterns,
use organization scopes and project/RBAC controls for team ownership,
centralize external registry access and avoid creating unnecessary
repositories for every team.
```

---

## 125. How Does NPM Affect Docker Builds?

Answer:

```text
A Node.js Docker build often runs npm ci, so the NPM repository is a
build dependency. If Artifactory, DNS, authentication or the remote
registry fails, image builds can fail. I therefore treat NPM
repository availability as part of the CI platform dependency chain.
```

---

## 126. Should Kubernetes Run npm install?

Answer:

```text
Normally no. I prefer installing dependencies during CI, creating an
immutable container image and deploying that image to Kubernetes.
This avoids runtime dependency on public or internal package
registries.
```

---

## 127. How Do You Handle an NPM Registry Outage?

Answer:

```text
I first determine whether required packages are already cached in
Artifactory. Cached dependencies may continue to work, while
uncached dependencies can fail. For critical builds I reduce
external dependency risk through approved remotes, caching and
controlled dependency management.
```

---

## 128. How Do You Secure Private NPM Packages?

Answer:

```text
I use scoped package names, private local repositories, least
privilege, CI service identities, token rotation, audit logging,
dependency scanning and immutable production releases.
```

---

# PART XVIII — FINAL PRODUCTION CHECKLIST

## 129. Repository

```text
[ ] npm-local
[ ] npm-remote
[ ] npm-virtual
[ ] naming standard
[ ] owner
[ ] purpose
[ ] package scope
```

---

## 130. Authentication

```text
[ ] CI identity
[ ] secure token
[ ] token rotation
[ ] no plaintext credentials
[ ] read-only runtime access
```

---

## 131. Security

```text
[ ] @company scope
[ ] approved upstreams
[ ] dependency scanning
[ ] secret scanning
[ ] audit
[ ] least privilege
[ ] immutable releases
```

---

## 132. CI/CD

```text
[ ] npm ci
[ ] tests
[ ] build
[ ] security scan
[ ] npm publish
[ ] Build Info/provenance
[ ] promotion
```

---

## 133. Operations

```text
[ ] monitoring
[ ] logging
[ ] cache monitoring
[ ] storage monitoring
[ ] upstream monitoring
[ ] backup
[ ] restore testing
[ ] DR
```

---

## 134. Reliability

```text
[ ] public registry outage strategy
[ ] cache strategy
[ ] CI burst capacity
[ ] rollback versions
[ ] controlled repository changes
```

---

# PART XIX — GOLDEN RULES

## 135. Rules

```text
1. Use Artifactory as the controlled NPM dependency boundary.

2. Use npm-virtual for consumer dependency resolution where
   appropriate.

3. Publish private packages to npm-local.

4. Use npm-remote for approved external NPM sources.

5. Use organization-controlled scopes such as @company/*.

6. Commit package-lock.json where the project uses lockfiles.

7. Prefer npm ci for deterministic CI installation.

8. Do not hardcode NPM or Artifactory tokens.

9. Use dedicated CI service identities.

10. Apply least privilege.

11. Keep production package versions immutable.

12. Do not rely on the public NPM registry directly from every CI job.

13. Do not assume Artifactory remote cache is a complete mirror.

14. Scan direct and transitive dependencies.

15. Inspect package contents before publishing.

16. Do not publish secrets.

17. Revoke exposed secrets immediately.

18. Build NPM dependencies into immutable container images.

19. Avoid npm install at Kubernetes runtime.

20. Monitor Artifactory, cache, upstream and CI traffic.

21. Capacity-plan for large CI bursts.

22. Treat virtual repository changes as production changes.

23. Maintain rollback-capable package versions.

24. Test repository restore procedures.

25. Validate exact NPM and Artifactory configuration against the
    deployed versions before production rollout.
```

---