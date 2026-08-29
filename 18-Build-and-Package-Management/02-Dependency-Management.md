# Dependency-Management

## 1. Purpose

Dependency management is one of the most important parts of modern
software delivery.

An application is rarely built from source code alone:

```text
Application
 |
 +--> Frameworks
 +--> Libraries
 +--> Plugins
 +--> Test tools
 +--> Build tools
 +--> Runtime libraries
 +--> System packages
 +--> Container base image
```

A production DevOps engineer must understand how dependencies are:

```text
declared
resolved
downloaded
cached
verified
updated
scanned
locked
published
promoted
```

This file covers dependency management across Maven, npm/Node.js and
Python, with enterprise CI/CD, Artifactory, security, reproducibility,
performance, troubleshooting and production architecture.

---

# PART I — DEPENDENCY FUNDAMENTALS

## 2. What Is a Dependency?

A dependency is software required by an application, build process or
runtime.

Example:

```text
payment-service
 |
 +--> Spring Framework
 +--> Jackson
 +--> PostgreSQL Driver
 +--> Logging Library
```

---

## 3. Why Applications Need Dependencies

Reimplementing everything internally would be expensive.

Instead:

```text
Application
 |
v
Reusable Libraries
 |
v
Faster Development
```

Dependencies provide:

```text
functionality
frameworks
protocol implementations
testing
logging
serialization
database access
security
```

---

## 4. Dependency Categories

Common categories:

```text
compile/runtime dependencies
test dependencies
build dependencies
plugin dependencies
optional dependencies
development dependencies
peer dependencies
transitive dependencies
```

The exact terminology differs by ecosystem.

---

# PART II — DIRECT AND TRANSITIVE DEPENDENCIES

## 5. Direct Dependency

The application explicitly declares:

```text
library-A
```

Example concept:

```text
Application
   |
   +--> library-A
```

---

## 6. Transitive Dependency

If:

```text
Application -> A
A -> B
```

then:

```text
Application
 |
 +--> A
      |
      +--> B
```

B is a transitive dependency.

---

## 7. Why Transitive Dependencies Matter

A security vulnerability can exist in:

```text
A
```

or:

```text
B
```

even when the application never explicitly declared B.

Therefore dependency scanning must inspect the complete resolved graph.

---

# PART III — DEPENDENCY GRAPH

## 8. Graph Example

```text
Application
 |
 +--> A 1.0
 |     |
 |     +--> C 2.0
 |
 +--> B 3.0
       |
       +--> C 1.0
       |
       +--> D 4.0
```

The package manager must resolve C.

---

## 9. Dependency Graph Problems

Common issues:

```text
version conflict
duplicate dependency
vulnerable transitive dependency
incompatible API
dependency cycle
repository outage
```

---

# PART IV — VERSION MANAGEMENT

## 10. Why Versions Matter

Consider:

```text
library-A 1.0
library-A 1.1
library-A 2.0
```

The API and behavior may differ.

---

## 11. Version Constraints

Different ecosystems support different forms:

```text
exact version
minimum version
version range
compatible range
```

Use the strictness appropriate to the application and release policy.

---

## 12. Floating Versions

Avoid uncontrolled versions such as:

```text
always use latest
```

because today's build may resolve differently tomorrow.

---

# PART V — LOCKING

## 13. What Is Dependency Locking?

Locking records the resolved dependency versions so subsequent builds
can reproduce the same dependency graph.

Examples:

```text
package-lock.json
npm-shrinkwrap.json
poetry.lock
uv.lock
```

---

## 14. Why Lock Dependencies?

Without a lock:

```text
Build A
 |
v
C 1.4

Later Build
 |
v
C 1.5
```

even when application source did not change.

---

## 15. Maven and Locking

Maven does not normally use an npm-style universal lockfile.

Maven dependency versions are commonly controlled through:

```text
pom.xml
dependencyManagement
parent POMs
properties
```

Dependency mediation then determines the resolved graph.

---

# PART VI — DEPENDENCY RESOLUTION

## 16. Resolution

The package manager determines:

```text
required packages
versions
transitive dependencies
repositories
download locations
```

---

## 17. Generic Resolution Flow

```text
Build Configuration
 |
v
Dependency Graph
 |
v
Repository Lookup
 |
v
Cache Lookup
 |
v
Download
 |
v
Verify
 |
v
Build
```

---

## 18. Resolution Failure

Example:

```text
Could not resolve dependency
```

Possible causes:

```text
wrong version
repository unavailable
DNS failure
TLS failure
authentication
authorization
artifact missing
proxy failure
```

---

# PART VII — REPOSITORIES

## 19. Public Repositories

Examples:

```text
Maven Central
npm registry
PyPI
```

Public repositories are useful but introduce external availability
and supply-chain risk.

---

## 20. Internal Repository

Enterprise pattern:

```text
CI
 |
v
Artifactory
 |
+--> local packages
+--> cached external packages
```

Benefits:

```text
central control
caching
security
availability
audit
```

---

## 21. Virtual Repository

Example:

```text
Application
 |
v
company-maven-virtual
 |
+--> internal-local
+--> approved-remote
```

The application does not need to know every upstream repository.

---

# PART VIII — MAVEN DEPENDENCY MANAGEMENT

## 22. Maven Dependency Declaration

Conceptual example:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>example-library</artifactId>
    <version>1.2.3</version>
</dependency>
```

---

## 23. Maven Dependency Management

A parent POM can centralize versions:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>example-library</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Child modules can then omit the version when it is managed.

---

## 24. Why Centralize Versions?

Without centralization:

```text
module-a -> library 1.2
module-b -> library 1.3
module-c -> library 1.1
```

With centralized management:

```text
Parent
 |
v
library 1.3
 |
+--> module-a
+--> module-b
+--> module-c
```

This improves consistency.

---

# PART IX — MAVEN TRANSITIVE DEPENDENCIES

## 25. Maven Dependency Tree

Useful command:

```bash
mvn dependency:tree
```

Example:

```text
com.company:payment-service
+- org.example:A:1.0
|  \- org.example:C:2.0
\- org.example:B:3.0
   \- org.example:C:1.0
```

---

## 26. Investigating Conflicts

Use:

```bash
mvn dependency:tree
```

and inspect:

```text
which dependency introduced the library
which version was selected
```

---

## 27. Maven Dependency Mediation

When multiple paths request the same dependency, Maven applies its
dependency mediation rules.

Do not assume the version you intended was selected; inspect the
resolved tree.

---

# PART X — MAVEN EXCLUSIONS

## 28. Why Exclude?

Suppose:

```text
A -> B
```

but B is unwanted or conflicts with another approved version.

A Maven exclusion can prevent B from being pulled transitively.

Concept:

```xml
<exclusions>
    <exclusion>
        <groupId>org.example</groupId>
        <artifactId>library-b</artifactId>
    </exclusion>
</exclusions>
```

---

## 29. Do Not Exclude Blindly

An exclusion can cause:

```text
runtime ClassNotFoundException
NoSuchMethodError
```

if another component actually requires the dependency.

Always test.

---

# PART XI — MAVEN SCOPES

## 30. Common Scopes

Examples include:

```text
compile
provided
runtime
test
system
```

Use scopes according to the application architecture.

---

## 31. Test Dependency

A test-only dependency should not normally be packaged as a production
runtime dependency.

---

# PART XII — NPM DEPENDENCY MANAGEMENT

## 32. package.json

Example:

```json
{
  "dependencies": {
    "express": "^5.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

---

## 33. dependencies vs devDependencies

Typical model:

```text
dependencies
    |
    v
needed by application runtime

devDependencies
    |
    v
needed for development/build/test
```

Actual deployment behavior depends on how the application is packaged.

---

## 34. package-lock.json

The lockfile records the resolved dependency graph and versions.

For CI, commit it to source control for applications where the project
uses npm lockfile-based installation.

---

# PART XIII — npm ci

## 35. Why npm ci?

In CI:

```bash
npm ci
```

provides a clean installation based on the lockfile.

It is generally preferred over:

```bash
npm install
```

for reproducible CI installations when the lockfile is maintained.

---

## 36. npm install vs npm ci

Typical distinction:

```text
npm install
 |
can update lockfile based on package changes

npm ci
 |
clean install based on lockfile
```

---

# PART XIV — NPM VERSION RANGES

## 37. Caret

Example:

```text
^1.2.3
```

allows a defined range of compatible versions according to npm's
semver rules.

---

## 38. Tilde

Example:

```text
~1.2.3
```

allows a narrower patch-oriented range according to npm's semver
rules.

---

## 39. Exact Version

Example:

```text
1.2.3
```

is more restrictive.

Use version ranges intentionally and rely on the lockfile for
repeatable installations.

---

# PART XV — PYTHON DEPENDENCY MANAGEMENT

## 40. Python Files

Common approaches use:

```text
requirements.txt
pyproject.toml
Poetry
uv
```

---

## 41. requirements.txt

Example:

```text
requests==2.32.0
```

Exact pins improve reproducibility but must be maintained.

---

## 42. Python Dependency Graph

```text
Application
 |
+--> requests
|     |
|     +--> urllib3
|
+--> framework
      |
      +--> dependency
```

---

## 43. Python Locking

Modern Python workflows may use lock files generated by the selected
tooling.

Examples:

```text
poetry.lock
uv.lock
```

Use one project-standard approach.

---

# PART XVI — DEPENDENCY PINNING

## 44. Pinning

Pinning means explicitly controlling dependency versions.

Example:

```text
requests==2.32.0
```

---

## 45. Benefits

```text
reproducibility
predictability
controlled upgrades
```

---

## 46. Risks

Over-pinning everything can create:

```text
upgrade burden
stale packages
security exposure
maintenance overhead
```

The goal is controlled dependency management, not permanent stagnation.

---

# PART XVII — DEPENDENCY UPDATES

## 47. Update Strategy

Do not update every dependency randomly.

Use:

```text
inventory
risk assessment
test
security review
release
```

---

## 48. Regular Updates

Example:

```text
Monthly dependency review
 |
v
Security updates
 |
v
Compatibility testing
 |
v
Release
```

Critical vulnerabilities may require immediate action.

---

# PART XVIII — DEPENDENCY VULNERABILITIES

## 49. Vulnerable Dependency

Example:

```text
Application
 |
v
A 1.0
 |
v
B 2.1  <-- CVE
```

Even if B is transitive, the application can be affected.

---

## 50. Response

```text
Identify
 |
v
Assess exploitability
 |
v
Find fixed version
 |
v
Test
 |
v
Release
```

---

## 51. No Fixed Version

If no fixed version exists:

```text
upgrade parent dependency
replace dependency
apply compensating control
remove feature
isolate exposure
```

according to risk.

---

# PART XIX — LICENSE MANAGEMENT

## 52. Dependency Licenses

Organizations may restrict licenses.

Examples can include:

```text
permissive licenses
copyleft licenses
commercial licenses
```

The exact policy is organization-specific.

---

## 53. License Gate

```text
Dependency
 |
v
License Scan
 |
+--> Approved
|
+--> Review
|
+--> Block
```

---

# PART XX — SOFTWARE SUPPLY CHAIN

## 54. Dependency as Supply Chain

```text
External Package
 |
v
Internal Repository
 |
v
CI
 |
v
Artifact
 |
v
Production
```

A dependency can become part of production software.

---

## 55. Trust Model

Do not assume:

```text
public package = safe
```

Instead:

```text
discover
verify
scan
approve
consume
```

---

# PART XXI — DEPENDENCY CONFUSION

## 56. Attack

An attacker may publish a public package using an internal package
name.

If resolution is misconfigured:

```text
Internal package
        |
        X
Public malicious package
```

---

## 57. Prevention

Use:

```text
internal repository
namespace/scopes
approved repositories
dependency policies
```

---

# PART XXII — TYPOSQUATTING

## 58. Attack

Legitimate:

```text
company-security-lib
```

Malicious lookalike:

```text
company-securty-lib
```

---

## 59. Prevention

```text
review dependencies
scan packages
use approved sources
use lockfiles
```

---

# PART XXIII — DEPENDENCY CONFUSION WITH NPM

## 60. Scoped Packages

Example:

```text
@company/payment-utils
```

A private scope can make ownership clearer.

Repository configuration must ensure the package resolves from the
intended registry.

---

# PART XXIV — DEPENDENCY CACHING

## 61. Why Cache?

Without caching:

```text
100 builds
 |
+--> Internet
+--> Internet
+--> Internet
```

With internal caching:

```text
100 builds
 |
v
Internal Repository
 |
v
External Source
```

---

## 62. Cache Benefits

```text
speed
reliability
lower latency
less external traffic
```

---

## 63. Cache Risks

Stale or corrupted cache data can cause failures.

Use repository tooling and lifecycle mechanisms appropriately.

---

# PART XXV — OFFLINE / DEGRADED BUILDS

## 64. External Registry Outage

If a dependency is already available internally:

```text
CI
 |
v
Internal Cache
 |
v
Build
```

The build may continue depending on the package manager and cache
state.

---

## 65. Critical Dependencies

For critical production systems:

```text
identify
cache
verify
retain
```

important dependencies rather than relying on live public services.

---

# PART XXVI — DEPENDENCY PROVENANCE

## 66. Provenance

Record:

```text
package
version
source repository
download source
checksum/integrity information
build
artifact
```

---

## 67. Why?

During an incident:

```text
Which version?
Where did it come from?
Which application uses it?
Which build included it?
```

should be answerable.

---

# PART XXVII — INTEGRITY

## 68. Integrity Verification

Package ecosystems may use:

```text
checksums
lockfile integrity
repository metadata
signatures
```

Use the integrity mechanisms supported by the ecosystem and platform.

---

# PART XXVIII — DEPENDENCY GRAPH VISIBILITY

## 69. Why Inspect the Graph?

The declared dependencies are not the complete runtime picture.

You need:

```text
declared graph
+
transitive graph
```

---

## 70. Maven

```bash
mvn dependency:tree
```

---

## 71. npm

Useful commands include:

```bash
npm ls
npm explain <package>
```

---

## 72. Python

Depending on tooling:

```bash
pip show <package>
pipdeptree
```

Use tools approved by the project.

---

# PART XXIX — DEPENDENCY OVERRIDES

## 73. Why Override?

Suppose:

```text
A -> vulnerable C 1.0
```

but:

```text
C 1.1
```

is compatible and fixes the issue.

A controlled override can force the safe version.

---

## 74. Test Overrides

Never assume compatibility.

Test:

```text
unit
integration
application startup
critical flows
```

---

# PART XXX — DEPENDENCY MANAGEMENT IN CI

## 75. Pipeline

```text
Checkout
 |
v
Install Toolchain
 |
v
Resolve Dependencies
 |
v
Cache
 |
v
Compile
 |
v
Test
 |
v
Security Scan
 |
v
Package
```

---

## 76. Dependency Cache Key

A cache should change when dependency inputs change.

Concept:

```text
OS + Toolchain + Lockfile Hash
```

This avoids incorrectly reusing unrelated dependency state.

---

# PART XXXI — CACHE INVALIDATION

## 77. Common Problem

```text
lockfile changed
 |
v
old cache reused
 |
v
unexpected build
```

Use cache keys that incorporate dependency configuration.

---

# PART XXXII — DEPENDENCY MANAGEMENT WITH ARTIFACTORY

## 78. Enterprise Flow

```text
Developer / CI
      |
      v
Artifactory Virtual
      |
      +--> Internal Local
      |
      +--> Approved Remote
              |
              v
          Public Registry
```

---

## 79. Why This Architecture?

It provides:

```text
central control
caching
auditing
security scanning
stable endpoints
```

---

# PART XXXIII — MAVEN + ARTIFACTORY

## 80. Maven

```text
Maven
 |
v
Artifactory Virtual
 |
+--> Internal Releases
+--> Internal Snapshots
+--> Approved Remote
```

---

## 81. Credentials

CI should use:

```text
service identity
scoped token
```

not a human administrator account.

---

# PART XXXIV — NPM + ARTIFACTORY

## 82. npm

```text
npm
 |
v
Artifactory npm virtual
 |
+--> npm local
+--> approved npm remote
```

---

# PART XXXV — PYTHON + ARTIFACTORY

## 83. Python

```text
pip/build tool
 |
v
Artifactory PyPI virtual
 |
+--> internal
+--> approved external
```

---

# PART XXXVI — DEPENDENCY UPDATE AUTOMATION

## 84. Automation

Organizations can use dependency update tooling to create proposed
updates.

Workflow:

```text
New Version
 |
v
Automated PR
 |
v
Tests
 |
v
Security
 |
v
Review
 |
v
Merge
```

---

## 85. Automation Must Not Mean Automatic Production

A dependency update should pass:

```text
compatibility
security
tests
release policy
```

before production.

---

# PART XXXVII — BREAKING CHANGES

## 86. Major Update

Example:

```text
Framework 2.x -> 3.x
```

may contain breaking changes.

---

## 87. Upgrade Strategy

```text
Read migration guide
 |
v
Create branch
 |
v
Upgrade
 |
v
Compile
 |
v
Test
 |
v
Integration
 |
v
Performance
 |
v
Release
```

---

# PART XXXVIII — DEPENDENCY DRIFT

## 88. What Is Drift?

Dependency drift occurs when environments or projects use different
versions unexpectedly.

Example:

```text
Dev -> A 1.4
Stage -> A 1.5
Prod -> A 1.3
```

---

## 89. Prevention

Use:

```text
version control
lockfiles where supported
artifact promotion
standard parent configurations
automated checks
```

---

# PART XXXIX — DEV/TEST/PROD DEPENDENCIES

## 90. Separation

Not every development dependency belongs in production.

Example:

```text
Build tools
Test frameworks
Linters
```

should generally remain build/test dependencies.

---

# PART XL — RUNTIME VS BUILD DEPENDENCY

## 91. Build Dependency

Required to create software:

```text
compiler
plugin
test framework
bundler
```

---

## 92. Runtime Dependency

Required while application executes:

```text
framework runtime
database driver
serialization library
```

---

# PART XLI — DEPENDENCY GRAPH SECURITY

## 93. Security Scanning

Scan:

```text
direct dependencies
transitive dependencies
build dependencies
container OS packages
```

when relevant.

---

# PART XLII — CONTAINER DEPENDENCIES

## 94. Container Dependency Layers

```text
Base Image
 |
+--> OS Packages
 |
+--> Runtime
 |
+--> Application Dependencies
 |
+--> Application
```

All are part of the production software supply chain.

---

## 95. Base Image Updates

Monitor:

```text
OS CVEs
runtime CVEs
base image lifecycle
```

---

# PART XLIII — DEPENDENCY SBOM

## 96. SBOM

Software Bill of Materials describes components contained in a software
artifact.

Concept:

```text
Artifact
 |
v
SBOM
 |
+--> package A
+--> package B
+--> package C
```

---

## 97. Why SBOM?

Useful for:

```text
vulnerability response
compliance
inventory
incident investigation
customer requirements
```

---

# PART XLIV — DEPENDENCY POLICY

## 98. Enterprise Policy

Possible rules:

```text
only approved registries
no unknown package sources
critical CVEs blocked
restricted licenses reviewed
versions controlled
production artifacts immutable
```

---

# PART XLV — DEPENDENCY TROUBLESHOOTING

## 99. "Could Not Resolve Dependency"

Check:

```text
dependency coordinates
version
repository URL
DNS
network
TLS
authentication
authorization
artifact existence
cache
```

---

## 100. "Works Locally, Fails in CI"

Check:

```text
lockfile
tool version
cache
credentials
repository configuration
environment
```

---

## 101. Different Dependency Version

Check:

```text
lockfile
Maven dependency tree
npm dependency tree
Python resolver output
parent configuration
overrides
```

---

## 102. Security Scan Fails

Check:

```text
affected package
direct/transitive relationship
fixed version
compatibility
exception policy
```

---

# PART XLVI — PRODUCTION INCIDENT

## 103. Vulnerable Transitive Dependency

Scenario:

```text
Application
 |
v
Framework
 |
v
Library X 1.0
 |
v
CVE
```

Response:

```text
identify affected versions
 |
v
find fixed version
 |
v
test upgrade
 |
v
publish new artifact
 |
v
scan
 |
v
promote
```

---

# PART XLVII — PRODUCTION DEPENDENCY ARCHITECTURE

## 104. Reference Architecture

```text
                    Public Registries
                   /       |        \
                  v        v         v
             Maven       npm       PyPI
                \          |         /
                 +---------+--------+
                           |
                           v
                    Enterprise Repository
                           |
                    +------+------+
                    |             |
                    v             v
                 Virtual        Local
                    |
                    v
                    CI
                    |
                    v
                 Build
                    |
                    v
                Artifact
                    |
                    v
                Production
```

---

# PART XLVIII — DEPENDENCY GOVERNANCE

## 105. Ownership

Every important dependency should have:

```text
consumer owner
upgrade strategy
security contact
```

---

## 106. Dependency Inventory

Maintain visibility into:

```text
package
version
application
environment
license
vulnerabilities
```

---

# PART XLIX — DEPENDENCY LIFECYCLE

## 107. Lifecycle

```text
Discover
 |
v
Evaluate
 |
v
Approve
 |
v
Consume
 |
v
Monitor
 |
v
Update
 |
v
Retire
```

---

# PART L — PRODUCTION CHECKLIST

## 108. Dependency Sources

```text
[ ] approved repositories
[ ] internal caching
[ ] stable endpoints
[ ] external source controls
```

## 109. Versions

```text
[ ] versions controlled
[ ] lockfiles where supported
[ ] no uncontrolled latest
[ ] upgrade process
```

## 110. Security

```text
[ ] vulnerability scan
[ ] license review
[ ] provenance
[ ] integrity
[ ] SBOM where required
```

## 111. CI

```text
[ ] clean install
[ ] dependency cache
[ ] cache key includes dependency inputs
[ ] protected credentials
```

## 112. Operations

```text
[ ] dependency inventory
[ ] monitoring
[ ] update process
[ ] incident process
[ ] rollback
```

---

# PART LI — INTERVIEW PREPARATION

## 113. What Is Dependency Management?

Answer:

```text
Dependency management is the process of declaring, resolving,
versioning, downloading, caching, securing and updating the libraries
and tools required by an application and its build.
```

## 114. Direct vs Transitive Dependency?

Answer:

```text
A direct dependency is explicitly declared by the application.
A transitive dependency is introduced through another dependency.
Both matter because vulnerabilities and compatibility problems can
exist in transitive components.
```

## 115. Why Use Lockfiles?

Answer:

```text
They capture resolved dependency versions and improve build
reproducibility. I use them where the ecosystem and project workflow
support them.
```

## 116. How Do You Secure Dependencies?

Answer:

```text
I use approved repositories, controlled versions, vulnerability and
license scanning, provenance, integrity verification, scoped
credentials and a defined update process.
```

## 117. How Do You Handle a Vulnerable Transitive Dependency?

Answer:

```text
I identify which direct dependency introduced it, determine whether
a fixed version exists, test an upgrade or controlled override, scan
the result and release a new artifact if the risk requires it.
```

## 118. How Do You Prevent Dependency Confusion?

Answer:

```text
I use internal repositories, explicit package namespaces or scopes,
controlled repository configuration and dependency policies so
internal packages cannot unexpectedly resolve from an untrusted
public source.
```

## 119. Why Use Artifactory for Dependencies?

Answer:

```text
It provides a centralized and controlled dependency endpoint,
caching, access control, auditing and integration with multiple
package ecosystems. It also reduces direct dependency on public
registries.
```

## 120. npm install vs npm ci?

Answer:

```text
npm install is commonly used for dependency installation and can
update the lockfile when package definitions change. npm ci is
designed for clean CI installation from the lockfile and is generally
preferred for reproducible CI builds.
```

## 121. How Do You Troubleshoot Dependency Resolution?

Answer:

```text
I inspect the dependency coordinates and resolved graph first, then
repository configuration, credentials, DNS, TLS, network, artifact
availability and cache state. I compare local and CI environments
when the failure is environment-specific.
```

---

# PART LII — SENIOR-LEVEL SCENARIOS

## 122. 500 Services Use Different Versions

Answer:

```text
I would first inventory the dependency versions and identify critical
shared libraries. Then I would establish approved version policies,
standardize where practical, automate vulnerability detection and
dependency update proposals, and roll upgrades through tested release
waves rather than forcing an unsafe big-bang upgrade.
```

## 123. Public Registry Is Unavailable

Answer:

```text
I would determine whether required packages are already cached
internally. If not, I would assess the impact and use approved
recovery mechanisms. For long-term reliability, critical dependencies
should be available through controlled internal repositories and
reproducible build inputs.
```

## 124. Critical CVE With No Upgrade

Answer:

```text
I would assess exploitability and exposure, identify compensating
controls, evaluate whether the dependency can be replaced or removed,
and document the risk decision. I would not blindly force an
incompatible version just to make a scanner green.
```

## 125. Dependency Update Breaks Production

Answer:

```text
I would stop further promotion, identify the changed dependency,
compare the previous and new resolved graphs, roll back to the last
known-good artifact if necessary, then reproduce and fix the
compatibility issue before creating a new release.
```

---

# PART LIII — GOLDEN RULES

## 126. Rules

```text
1. Treat dependencies as production software.

2. Control dependency sources.

3. Prefer approved internal repositories.

4. Understand direct and transitive dependencies.

5. Inspect the resolved dependency graph.

6. Control important dependency versions.

7. Use lockfiles where the ecosystem supports them.

8. Do not rely blindly on latest.

9. Separate build and runtime dependencies.

10. Scan transitive dependencies.

11. Track licenses.

12. Verify package integrity.

13. Maintain dependency provenance.

14. Use scoped CI credentials.

15. Cache critical dependencies.

16. Design cache keys around dependency inputs.

17. Do not trust public packages automatically.

18. Protect against dependency confusion.

19. Protect against typosquatting.

20. Automate dependency update proposals.

21. Test dependency updates.

22. Treat major upgrades as migration projects.

23. Avoid unnecessary permanent pinning.

24. Keep security patches moving.

25. Do not blindly override vulnerable dependencies.

26. Generate SBOMs where required.

27. Maintain dependency inventory.

28. Define dependency ownership.

29. Build once and promote the resulting artifact.

30. Do not rebuild production solely because an environment changed.

31. Keep rollback artifacts available.

32. Investigate dependency failures systematically.

33. Compare local and CI environments for reproducibility issues.

34. Use internal repository caching to reduce public-registry
    dependency.

35. Remember that the container base image is also a dependency.

36. Remember that build plugins and build tools are part of the
    supply chain.

37. Security gates should reflect actual risk, not only scanner status.

38. A dependency exception must have an owner and review date.

39. Standardize dependency policy without blocking legitimate
    application requirements.

40. Validate exact package-manager behavior for the versions used by
    the project.
```

---