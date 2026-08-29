# npm-and-Node.js-Builds

## 1. Purpose

Node.js applications are commonly built and packaged with npm or
compatible package managers. In production DevOps environments, the
important concern is not only running `npm install`.

A reliable build flow is:

```text
Git
 |
v
CI
 |
+--> Node.js version
+--> package manager
+--> lockfile
+--> private registry
+--> dependency install
+--> lint
+--> unit tests
+--> build
+--> security
+--> package
+--> publish
 |
v
Artifact / Container Registry
 |
v
Deployment
```

This file covers Node.js and npm fundamentals, `package.json`,
lockfiles, dependency types, semantic versioning, npm scripts,
`npm install` vs `npm ci`, registries, private packages, Artifactory,
authentication, workspaces, monorepos, caching, Node.js runtime
management, CI/CD, GitHub Actions, Jenkins, Docker, Kubernetes,
security, supply-chain controls, artifact publishing, releases,
troubleshooting, production architecture and interview preparation.

---

# PART I — NODE.JS AND NPM FUNDAMENTALS

## 2. What Is Node.js?

Node.js is a JavaScript runtime commonly used for:

```text
APIs
microservices
CLI tools
frontend build systems
event-driven services
automation
```

---

## 3. What Is npm?

npm is a package manager and ecosystem used to install, manage and
publish JavaScript packages.

Typical workflow:

```text
package.json
     |
     v
npm
     |
     v
dependencies
```

---

## 4. npm in DevOps

npm participates in:

```text
dependency management
build
test
lint
packaging
publishing
```

---

# PART II — PACKAGE.JSON

## 5. package.json

A typical project contains:

```text
package.json
package-lock.json
src/
```

Example:

```json
{
  "name": "payment-api",
  "version": "1.4.0",
  "scripts": {
    "test": "jest",
    "build": "tsc",
    "lint": "eslint ."
  }
}
```

---

## 6. package.json Responsibilities

It can define:

```text
project metadata
version
scripts
dependencies
devDependencies
peerDependencies
engines
package configuration
```

---

# PART III — LOCKFILES

## 7. package-lock.json

The npm lockfile records resolved dependency information so that builds
can be reproduced more consistently.

---

## 8. Why Lockfiles Matter

Without a lockfile:

```text
package.json
 |
v
version ranges
 |
v
different dependency resolution
```

With a lockfile:

```text
package.json
      +
package-lock.json
      |
      v
deterministic dependency tree
```

---

## 9. Commit the Lockfile

For applications, normally commit:

```text
package-lock.json
```

Do not routinely regenerate it on every CI run.

---

# PART IV — npm install VS npm ci

## 10. npm install

Typical developer command:

```bash
npm install
```

It can resolve dependencies and update the lockfile when necessary.

---

## 11. npm ci

CI should generally use:

```bash
npm ci
```

when a supported lockfile is committed.

---

## 12. npm ci Behavior

Conceptually:

```text
package.json
      +
lockfile
      |
      v
clean dependency installation
```

It is designed for automated environments.

---

## 13. Why npm ci in CI?

Benefits include:

```text
clean installation
lockfile-based resolution
more predictable builds
avoids relying on an existing node_modules directory
```

---

# PART V — DEPENDENCY TYPES

## 14. dependencies

Runtime dependencies belong in:

```json
"dependencies": {}
```

Example:

```text
express
```

---

## 15. devDependencies

Build and development tooling commonly belongs in:

```json
"devDependencies": {}
```

Examples:

```text
typescript
eslint
jest
```

---

## 16. peerDependencies

Used when a package expects the consuming application to provide a
compatible dependency.

Common in:

```text
libraries
plugins
framework integrations
```

---

## 17. optionalDependencies

Dependencies can be marked optional when the package supports optional
functionality.

Understand the package manager's installation behavior before relying
on optional dependencies for core production functionality.

---

# PART VI — VERSIONING

## 18. Semantic Versioning

Common form:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.4.1
```

---

## 19. Version Ranges

Examples:

```text
^2.4.1
~2.4.1
>=2.4.0
```

Ranges can allow dependency changes between installations.

Lockfiles reduce this risk for application CI.

---

# PART VII — npm SCRIPTS

## 20. Scripts

Example:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest --ci",
    "build": "tsc"
  }
}
```

---

## 21. Standard Commands

```bash
npm run lint
npm test
npm run build
```

---

# PART VIII — NODE.JS VERSION MANAGEMENT

## 22. Node Version

A project should explicitly define supported Node.js versions.

Example:

```json
{
  "engines": {
    "node": ">=20 <23"
  }
}
```

Use the range that matches the application's actual support policy.

---

## 23. Version Consistency

Control:

```text
Node.js
npm
package manager
OS/base image
native build tools
```

---

# PART IX — .NVMRC

## 24. .nvmrc

A project may provide:

```text
20
```

or an organization-approved exact Node version.

This helps developers align local environments.

---

# PART X — COREPACK AND PACKAGE MANAGERS

## 25. npm vs Other Package Managers

Node.js ecosystems can use:

```text
npm
Yarn
pnpm
```

Do not mix package-manager lockfiles casually.

---

## 26. Lockfile Consistency

For an npm project:

```text
package.json
package-lock.json
npm
```

should be aligned.

---

# PART XI — REGISTRIES

## 27. npm Registry

Packages are retrieved from a registry.

Concept:

```text
npm
 |
v
Registry
 |
v
Package
```

---

## 28. Public vs Private Registry

Enterprise architecture:

```text
CI
 |
v
Corporate npm Registry
 |
+--> Internal Packages
 |
+--> Approved Remote Cache
```

---

# PART XII — ARTIFACTORY npm REPOSITORY

## 29. Artifactory

A repository manager can provide:

```text
local npm repository
remote npm repository
virtual npm repository
```

---

## 30. Virtual Registry

```text
npm
 |
v
Virtual Repository
 |
+--> internal packages
+--> remote cache
```

This provides a stable endpoint for developers and CI.

---

# PART XIII — .NPMRC

## 31. .npmrc

npm configuration can be provided through:

```text
project .npmrc
user configuration
environment variables
CI-generated configuration
```

---

## 32. Registry Configuration

Concept:

```ini
registry=https://repo.company.example/npm/
```

Use the actual repository endpoint for the environment.

---

## 33. Do Not Commit Secrets

Avoid putting long-lived tokens directly in committed `.npmrc`.

Prefer:

```text
CI secrets
environment variables
short-lived credentials
OIDC where supported
```

---

# PART XIV — AUTHENTICATION

## 34. npm Authentication

Private registries require authentication.

Concept:

```text
CI
 |
v
Token / Identity
 |
v
Private Registry
```

---

## 35. Read vs Publish

Normal CI:

```text
read
```

Release workflow:

```text
publish
```

Separate permissions.

---

# PART XV — PRIVATE PACKAGES

## 36. Internal Package

Example:

```text
@company/auth-client
```

Flow:

```text
Application
 |
v
npm
 |
v
Private Registry
 |
v
@company/auth-client
```

---

# PART XVI — SCOPES

## 37. npm Scope

Scopes can organize private packages:

```text
@company/package-a
@company/package-b
```

Repository routing should be designed to prevent namespace confusion.

---

# PART XVII — DEPENDENCY CONFUSION

## 38. Risk

Suppose the organization uses:

```text
@company/payment-client
```

An attacker may attempt to publish a similarly named package to a public
registry if repository resolution is poorly controlled.

---

## 39. Controls

Use:

```text
private registry
approved upstreams
namespace ownership
registry policy
package provenance
```

---

# PART XVIII — npm CACHE

## 40. npm Cache

npm maintains a local cache.

Inspect:

```bash
npm cache verify
```

---

## 41. Cache Is Not Source of Truth

A cache should improve performance.

It should not be required for correctness.

---

# PART XIX — node_modules

## 42. node_modules

Installed dependencies normally exist under:

```text
node_modules/
```

Do not commit it for normal application repositories.

---

## 43. Why Ignore?

```text
large
platform-dependent
reproducible from lockfile
```

Typical Git rule:

```text
node_modules/
```

---

# PART XX — CI INSTALLATION

## 44. Standard CI Flow

```bash
npm ci
npm run lint
npm test
npm run build
```

---

## 45. Better Pipeline

```text
Checkout
 |
v
Node setup
 |
v
npm ci
 |
v
Lint
 |
v
Unit Tests
 |
v
Build
 |
v
Security
 |
v
Package
```

---

# PART XXI — BUILD ARTIFACT

## 46. Build Output

Examples:

```text
dist/
build/
lib/
```

The exact directory depends on the project.

---

## 47. Package vs Runtime

A Node.js application may be deployed as:

```text
source + production dependencies
```

or:

```text
compiled/bundled artifact
```

or:

```text
container image
```

Choose according to application architecture.

---

# PART XXII — npm PACK

## 48. npm pack

A package can be created using:

```bash
npm pack
```

This produces a tarball suitable for package publication or inspection.

---

# PART XXIII — npm PUBLISH

## 49. Publishing

Example:

```bash
npm publish
```

The target registry must be configured correctly.

---

## 50. Publish Pipeline

```text
Build
 |
v
Test
 |
v
Security
 |
v
Package
 |
v
Publish
```

---

# PART XXIV — PACKAGE IMMUTABILITY

## 51. Published Versions

Do not design production processes around replacing an already published
package version.

Use a new version when the package contents need to change.

---

# PART XXV — RELEASE MANAGEMENT

## 52. Release

Example:

```text
1.4.0
```

A release should be traceable to:

```text
Git commit
CI run
dependency lockfile
build environment
published package
```

---

# PART XXVI — BUILD ONCE

## 53. Build Once

```text
Git
 |
v
CI
 |
v
Build
 |
v
Validated Artifact
 |
v
Registry
 |
+--> DEV
+--> STAGE
+--> PROD
```

Avoid rebuilding the same source independently for each environment.

---

# PART XXVII — TESTING

## 54. Unit Testing

Common frameworks:

```text
Jest
Vitest
Mocha
```

Use the framework selected by the application.

---

## 55. CI Test

Example:

```bash
npm test -- --ci
```

Exact arguments depend on the test framework.

---

# PART XXVIII — LINTING

## 56. Lint

Example:

```bash
npm run lint
```

Linting catches:

```text
style violations
possible bugs
unsafe patterns
```

depending on configuration.

---

# PART XXIX — TYPESCRIPT

## 57. TypeScript Build

Common:

```bash
npm run build
```

which may invoke:

```bash
tsc
```

---

## 58. TypeScript CI

```text
npm ci
 |
v
tsc
 |
v
tests
```

Do not rely solely on editor diagnostics.

---

# PART XXX — SECURITY

## 59. npm Audit

Example:

```bash
npm audit
```

Interpret results carefully and integrate them with the organization's
security policy.

---

## 60. Dependency Scanning

Enterprise security may use dedicated SCA tools.

Scan:

```text
direct dependencies
transitive dependencies
```

---

# PART XXXI — PACKAGE INTEGRITY

## 61. Integrity

Lockfiles can contain integrity information for resolved packages.

Do not bypass integrity verification to hide dependency problems.

---

# PART XXXII — SUPPLY CHAIN

## 62. npm Supply Chain

```text
Public Package
 |
v
Approved Registry
 |
v
Corporate Cache
 |
v
CI
 |
v
Application
```

---

## 63. Supply-Chain Risks

```text
malicious package
typosquatting
dependency confusion
maintainer compromise
token theft
malicious lifecycle scripts
```

---

# PART XXXIII — LIFECYCLE SCRIPTS

## 64. npm Scripts During Install

Packages can define lifecycle scripts.

This means:

```text
npm ci
 |
v
third-party package code
```

may execute code during installation depending on package and npm
configuration.

---

## 65. Security Control

Review:

```text
dependencies
scripts
registry
runner permissions
secrets
```

Do not give untrusted dependency installation unnecessary access to
high-value credentials.

---

# PART XXXIV — PRODUCTION DEPENDENCIES

## 66. npm prune

Production packaging may remove development dependencies depending on
the application's deployment model.

Modern npm supports:

```bash
npm prune --omit=dev
```

Use the command appropriate to the chosen packaging strategy.

---

# PART XXXV — CONTAINER BUILDS

## 67. Basic Docker Flow

```text
package.json
package-lock.json
 |
v
npm ci
 |
v
npm run build
 |
v
Container
```

---

# PART XXXVI — MULTI-STAGE DOCKER

## 68. Build Stage

```text
Node Builder
 |
+--> npm ci
+--> npm run build
 |
v
dist/
```

---

## 69. Runtime Stage

```text
Minimal Runtime
 |
v
dist/
 |
v
Application
```

This can reduce the runtime image footprint.

---

# PART XXXVII — NODE CONTAINER SECURITY

## 70. Production

Prefer:

```text
minimal base
non-root user
read-only filesystem where practical
limited capabilities
```

Avoid shipping:

```text
source-control metadata
development dependencies
build caches
secrets
```

---

# PART XXXVIII — NODE MEMORY

## 71. Build Memory

Large frontend or TypeScript builds may require more Node heap.

Example:

```bash
NODE_OPTIONS="--max-old-space-size=4096"
```

Do not blindly increase memory; measure runner/container resources.

---

# PART XXXIX — CI CACHING

## 72. Cache npm Data

CI systems can cache npm's package cache.

The cache should be keyed by relevant inputs such as:

```text
OS
Node version
lockfile hash
package manager
```

---

## 73. Do Not Cache node_modules Blindly

Caching `node_modules` can be faster in some cases but can introduce:

```text
platform mismatch
native module issues
stale packages
```

Prefer lockfile-driven clean installs unless there is a measured reason
to use a node_modules cache.

---

# PART XL — NATIVE MODULES

## 74. Native Dependencies

Some npm packages compile native components.

They may require:

```text
Python
C/C++ compiler
make
system libraries
```

---

## 75. CI Problem

A build may work on one machine but fail on another because the native
toolchain differs.

Standardize build images.

---

# PART XLI — NODE VERSION FAILURE

## 76. Symptoms

```text
syntax error
unsupported engine
native module failure
dependency incompatibility
```

Check:

```bash
node --version
npm --version
```

---

# PART XLII — PACKAGE LOCK FAILURE

## 77. npm ci Error

If `package.json` and the lockfile are inconsistent, `npm ci` can fail.

Correct process:

```text
developer updates dependencies
 |
v
npm install
 |
v
commit package.json + lockfile
 |
v
CI npm ci
```

---

# PART XLIII — PRIVATE REGISTRY FAILURE

## 78. 401

Check:

```text
token
registry URL
scope
credential injection
```

---

## 79. 403

Check:

```text
package permission
publish permission
token scope
```

---

## 80. 404

Check:

```text
package name
version
registry routing
scope
package existence
```

---

# PART XLIV — ARTIFACTORY FAILURE

## 81. Registry Path

Check:

```text
npm registry
virtual repository
remote repository
local repository
authentication
```

---

# PART XLV — JENKINS INTEGRATION

## 82. Jenkins Flow

```text
Jenkins
 |
v
Node.js
 |
v
npm ci
 |
v
lint
 |
v
test
 |
v
build
 |
v
security
 |
v
publish
```

Use managed credentials for private registry access.

---

# PART XLVI — GITHUB ACTIONS

## 83. Example

```yaml
name: Node CI

on:
  push:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm test -- --ci
      - run: npm run build
```

Pin action versions according to organizational security policy.

---

# PART XLVII — GITHUB ACTIONS SECRETS

## 84. Private Registry

Concept:

```text
GitHub Actions
 |
v
Secret / OIDC
 |
v
Private npm Registry
```

Never print authentication tokens.

---

# PART XLVIII — OIDC

## 85. Federated Identity

Where the registry/platform supports it:

```text
GitHub Actions
 |
v
OIDC
 |
v
Identity Provider
 |
v
Short-lived credentials
```

This reduces dependence on long-lived static secrets.

---

# PART XLIX — WORKSPACES

## 86. npm Workspaces

Workspaces support multiple packages in one repository.

Example:

```text
repo/
├── package.json
├── package-lock.json
└── packages/
    ├── api/
    ├── auth/
    └── ui/
```

---

## 87. Workspace Flow

```text
Root npm
 |
v
Workspace graph
 |
+--> api
+--> auth
+--> ui
```

---

# PART L — MONOREPO

## 88. Monorepo Challenges

Large Node.js monorepos can have:

```text
many packages
large dependency graph
long CI duration
```

---

## 89. Optimization

Use:

```text
workspace-aware builds
affected-package detection
cache
parallelism
incremental builds
```

Do not skip required dependency-impact testing.

---

# PART LI — WORKSPACE PUBLISHING

## 90. Package Publishing

For multiple packages:

```text
workspace
 |
v
test
 |
v
build
 |
v
version
 |
v
publish
```

Versioning and dependency ordering must be handled carefully.

---

# PART LII — PACKAGE MANAGER CONSISTENCY

## 91. One Repository, One Strategy

Avoid:

```text
npm install
yarn install
pnpm install
```

randomly across the same project.

Standardize the package manager.

---

# PART LIII — DEPENDENCY UPDATE PROCESS

## 92. Update

```text
Dependency update
 |
v
lockfile
 |
v
CI
 |
+--> tests
+--> security
+--> build
```

---

# PART LIV — AUTOMATED UPDATES

## 93. Dependabot/Renovate

Automated tooling can create dependency update pull requests.

Review:

```text
breaking changes
security impact
test results
lockfile changes
```

---

# PART LV — RELEASE PROVENANCE

## 94. Track

Record:

```text
Git commit
Node version
npm version
lockfile hash
workflow run
package version
registry
checksum
```

---

# PART LVI — SBOM

## 95. SBOM

Generate an SBOM when required by security/compliance processes.

It can identify:

```text
package
version
transitive dependency
license
```

---

# PART LVII — LICENSE MANAGEMENT

## 96. Open Source Licenses

Review dependency licenses where required.

Potential concerns:

```text
license incompatibility
restricted use
attribution
policy violations
```

---

# PART LVIII — PACKAGE PROVENANCE

## 97. Provenance

The goal is to establish:

```text
source
 |
v
workflow
 |
v
artifact
 |
v
deployment
```

---

# PART LIX — RELEASE STRATEGY

## 98. Release Candidate

```text
PR
 |
v
CI
 |
v
candidate
 |
v
security
 |
v
publish
```

---

## 99. Production

```text
validated package
 |
v
promotion
 |
v
production
```

---

# PART LX — ROLLBACK

## 100. Package Rollback

Use a previously known-good package/container image.

```text
1.5.0
 |
X
1.4.2
 |
v
deploy
```

Do not rebuild old source merely to perform a rollback.

---

# PART LXI — OBSERVABILITY

## 101. Build Metrics

Track:

```text
queue time
Node setup
dependency install
lint
tests
build
security
publish
```

---

## 102. Failure Rate

Track:

```text
success rate
failure rate
duration
flaky tests
```

---

# PART LXII — PERFORMANCE

## 103. Slow npm ci

Investigate:

```text
registry latency
cache
dependency count
network
lockfile
native compilation
runner resources
```

---

## 104. Do Not Blindly Add CPU

Measure the actual bottleneck.

---

# PART LXIII — TROUBLESHOOTING

## 105. npm ci Fails

Check:

```text
Node version
npm version
package.json
package-lock.json
registry
credentials
network
native dependencies
```

---

## 106. Build Works Locally but Fails in CI

Compare:

```text
Node
npm
OS
environment variables
registry
lockfile
native toolchain
```

---

## 107. Private Package Cannot Be Found

Check:

```text
scope
registry URL
.npmrc
credentials
package version
repository routing
```

---

## 108. npm Publish Fails

Check:

```text
package version
registry
authentication
authorization
package policy
```

---

## 109. Native Module Fails

Check:

```text
Node ABI
compiler
Python
system libraries
base image
```

---

# PART LXIV — SECURITY INCIDENT

## 110. Token Leaked

Response:

```text
revoke
 |
v
rotate
 |
v
audit
 |
v
inspect package publishing
 |
v
review workflow
```

---

## 111. Malicious Package

Response:

```text
identify package
 |
v
block/remove according to policy
 |
v
identify consumers
 |
v
upgrade/replace
 |
v
rebuild
 |
v
redeploy
```

Follow organizational incident-response procedures.

---

# PART LXV — PRODUCTION ARCHITECTURE

## 112. Reference

```text
                         Git
                          |
                          v
                    GitHub/Jenkins
                          |
                          v
                    Ephemeral Runner
                          |
              +-----------+-----------+
              |                       |
              v                       v
          Node.js/npm             Credentials
              |
              v
         npm ci -- lockfile
              |
              v
       Corporate npm Registry
              |
       +------+------+
       |             |
       v             v
    Internal      Approved Remote
    Packages          Cache
       |
       v
      Lint
       |
       v
      Tests
       |
       v
     Security
       |
       v
      Build
       |
       v
     Package
       |
       v
   Artifact/Image
       |
       v
  Container Registry
       |
       v
      GitOps
       |
       v
   Kubernetes
```

---

# PART LXVI — PRODUCTION CONTAINER ARCHITECTURE

## 113. Multi-Stage

```text
Builder
 |
+--> npm ci
+--> test
+--> build
 |
v
dist/

Runtime
 |
+--> minimal files
+--> non-root
 |
v
Container
```

---

# PART LXVII — PRODUCTION CHECKLIST

## 114. Node

```text
[ ] supported Node version
[ ] controlled npm version
[ ] lockfile committed
[ ] engine policy
```

## 115. Registry

```text
[ ] private registry
[ ] virtual repository
[ ] approved upstreams
[ ] read/write separation
[ ] immutable releases
```

## 116. CI

```text
[ ] npm ci
[ ] lint
[ ] unit tests
[ ] build
[ ] security
[ ] reports
```

## 117. Security

```text
[ ] secret protection
[ ] dependency scanning
[ ] action review
[ ] runner isolation
[ ] SBOM
[ ] provenance
```

## 118. Containers

```text
[ ] multi-stage build
[ ] minimal runtime
[ ] non-root
[ ] no secrets
[ ] no unnecessary dev dependencies
```

---

# PART LXVIII — INTERVIEW PREPARATION

## 119. npm install vs npm ci?

Answer:

```text
npm install is commonly used for development and can update the
lockfile. npm ci is intended for clean CI installation from the
lockfile and provides a more predictable automated build.
```

## 120. Why Commit package-lock.json?

Answer:

```text
It records resolved dependency information and helps make application
builds more reproducible.
```

## 121. How Do You Secure npm Dependencies?

Answer:

```text
I use a controlled private registry, approved upstreams, lockfiles,
dependency scanning, least-privilege credentials, package provenance,
SBOM where required and controlled CI runners.
```

## 122. How Do You Configure a Private npm Registry?

Answer:

```text
I configure the project or CI npm registry endpoint through .npmrc or
supported environment configuration, authenticate through protected
credentials or short-lived identity and ensure read and publish
permissions are separated.
```

## 123. Why Should node_modules Not Be Committed?

Answer:

```text
It is large, can contain platform-specific native binaries and is
reproducible from package.json and the lockfile.
```

## 124. How Do You Optimize npm Builds?

Answer:

```text
I measure dependency installation, registry latency, cache restore,
test and build times. I use lockfile-based installation, controlled
npm caching, appropriate parallelism and optimized container stages.
```

## 125. How Do You Handle npm Supply-Chain Risk?

Answer:

```text
I control registries and upstreams, review dependencies, scan for
vulnerabilities, protect credentials, review lifecycle scripts and
third-party CI actions, and maintain artifact provenance.
```

---

# PART LXIX — SENIOR-LEVEL SCENARIOS

## 126. npm ci Suddenly Becomes Very Slow

Answer:

```text
I compare dependency-install timing with the previous baseline and
check registry latency, cache effectiveness, network, dependency graph
changes, native compilation and runner resources. I optimize the
measured bottleneck rather than simply adding CPU.
```

## 127. Private Package Works Locally but Not in CI

Answer:

```text
I compare .npmrc, registry URL, package scope, credentials, token
permissions, Node/npm versions and local cache. A local cache can hide
an incorrect registry configuration, so I test CI with a clean
dependency state.
```

## 128. npm ci Fails After a Dependency Update

Answer:

```text
I inspect the package.json and lockfile together, verify that the
lockfile was generated by the expected npm/package-manager version,
review peer dependency changes and regenerate the lockfile through the
normal developer update process if required.
```

## 129. Production Package Contains Development Dependencies

Answer:

```text
I review the packaging strategy, dependency classification and runtime
installation command. I separate build-time dependencies from runtime
dependencies and ensure the final package/container contains only what
the application needs.
```

## 130. Malicious npm Package Is Discovered

Answer:

```text
I identify affected versions and consumers, block or quarantine the
package according to policy, rotate any credentials potentially exposed
during installation, replace the dependency, rebuild and redeploy.
I also investigate provenance and CI access.
```

## 131. Organization Has Hundreds of Node.js Repositories

Answer:

```text
I standardize Node versions, npm usage, lockfile policy, registry
configuration, security scanning and reusable CI workflows. Teams can
retain application-specific build steps while inheriting common
enterprise controls.
```

## 132. Node Build Runs Out of Memory

Answer:

```text
I inspect Node heap, concurrency, dependency graph, build tooling and
runner/container memory limits. I may increase the heap only after
confirming the workload genuinely needs it.
```

## 133. Same Git Commit Produces Different npm Artifacts

Answer:

```text
I compare lockfile, Node/npm versions, registry contents, build
environment, generated timestamps, native dependencies and workflow
versions. I identify uncontrolled inputs and make the build
reproducible.
```

## 134. Rollback Is Required

Answer:

```text
I deploy the previously validated immutable package or container image.
I avoid rebuilding the old source because rollback should restore a
known-good artifact.
```

---

# PART LXX — GOLDEN RULES

## 135. Rules

```text
1. Treat Node.js builds as production supply-chain processes.

2. Standardize the Node.js version.

3. Standardize the npm/package-manager version.

4. Commit the application lockfile.

5. Use npm ci in CI for supported npm lockfile workflows.

6. Do not blindly run npm install in every CI job.

7. Keep package.json and package-lock.json synchronized.

8. Do not commit node_modules.

9. Separate dependencies from devDependencies correctly.

10. Understand peerDependencies.

11. Understand optionalDependencies.

12. Use semantic versioning deliberately.

13. Do not rely blindly on version ranges for reproducibility.

14. Define supported Node versions.

15. Use .nvmrc or another version-management mechanism where useful.

16. Standardize the package manager.

17. Do not mix npm/Yarn/pnpm lockfiles casually.

18. Use npm scripts for repeatable project commands.

19. Use a controlled private registry in enterprise environments.

20. Prefer a virtual registry endpoint.

21. Control public upstream repositories.

22. Cache approved external packages centrally.

23. Separate read and publish permissions.

24. Use least-privilege registry credentials.

25. Never commit npm tokens.

26. Protect .npmrc credentials.

27. Prefer short-lived credentials where supported.

28. Consider OIDC/federated identity where supported.

29. Protect PR workflows from production credentials.

30. Treat npm lifecycle scripts as executable third-party code.

31. Review third-party dependencies.

32. Scan direct dependencies.

33. Scan transitive dependencies.

34. Review licenses where required.

35. Generate SBOMs where required.

36. Preserve package provenance.

37. Record Git commit to package version.

38. Record CI run to package.

39. Record lockfile state.

40. Record Node/npm versions.

41. Use npm cache for performance.

42. Treat cache as an optimization, not source of truth.

43. Test cold-cache installation.

44. Avoid blindly caching node_modules.

45. Monitor registry latency.

46. Monitor dependency-install duration.

47. Monitor build duration.

48. Monitor CI failure rate.

49. Use clean CI environments.

50. Prefer ephemeral runners where practical.

51. Protect self-hosted runners.

52. Do not expose secrets to untrusted dependency installation.

53. Use npm pack to inspect package contents when appropriate.

54. Do not overwrite production package versions.

55. Publish a new version when package contents change.

56. Build once and promote.

57. Do not rebuild separately for every environment.

58. Roll back using known-good immutable artifacts.

59. Do not rebuild during rollback.

60. Use multi-stage container builds where appropriate.

61. Keep build tooling out of the final runtime image where possible.

62. Do not ship source-control metadata unnecessarily.

63. Do not ship development secrets.

64. Run containers as non-root where practical.

65. Use minimal runtime images.

66. Monitor Node memory.

67. Tune NODE_OPTIONS only after measuring memory behavior.

68. Standardize native build toolchains.

69. Treat native npm modules as platform-sensitive dependencies.

70. Test the same base image used for production builds.

71. Use GitHub Actions or Jenkins reusable workflows for standardization.

72. Review third-party GitHub Actions.

73. Use least-privilege GitHub workflow permissions.

74. Protect production environments.

75. Separate validation workflows from release workflows.

76. Use matrix testing only when compatibility requirements justify it.

77. Use workspace-aware builds for monorepos.

78. Use affected-package optimization carefully.

79. Never skip dependency-impact testing merely to reduce build time.

80. Keep package publishing behind quality and security gates.

81. Publish test reports.

82. Publish security reports where appropriate.

83. Preserve build logs without leaking secrets.

84. Do not use debug output casually in production CI.

85. Investigate shared registry failures when many repositories fail.

86. Distinguish 401 from 403.

87. Treat 404 as a registry/routing/package/version question.

88. Compare clean and cached environments when troubleshooting.

89. Compare local and CI Node/npm versions.

90. Validate exact Node.js, npm, registry, runner, CI, build-tool and
    deployment behavior for the production architecture actually used.
```

---