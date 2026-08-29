# Build-Security

## 1. Purpose

Build security protects the software delivery path from source code to
production artifact.

The production objective is:

```text
Trusted Source
     |
     v
Trusted Build
     |
     v
Verified Dependencies
     |
     v
Secure Build Environment
     |
     v
Test + Scan
     |
     v
Signed / Attested Artifact
     |
     v
Trusted Repository
     |
     v
Controlled Promotion
     |
     v
Production
```

Build security is broader than running a vulnerability scanner.

It includes:

```text
source protection
CI/CD identity
dependency security
secret protection
build isolation
artifact integrity
supply-chain security
SBOM
provenance
signing
permissions
runner security
container security
package security
release security
auditability
incident response
```

A secure build must remain:

```text
secure
reproducible
traceable
available
maintainable
```

---

# PART I — SECURITY FUNDAMENTALS

## 2. Build Security

Build security protects:

```text
source
dependencies
build tools
CI runners
credentials
artifacts
repositories
deployment inputs
```

---

## 3. Why Build Security Matters

An attacker does not need to compromise production directly.

They may target:

```text
developer workstation
Git repository
CI workflow
dependency
build runner
artifact repository
container registry
```

Then malicious code can flow into production.

---

## 4. Supply-Chain Model

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Dependencies
   |
   v
Build
   |
   v
Artifact
   |
   v
Registry
   |
   v
Deployment
```

Every arrow is a trust boundary.

---

# PART II — THREAT MODEL

## 5. Threat Modeling

Ask:

```text
What can be compromised?
Who can modify it?
What credentials exist?
What artifacts can be published?
What can an untrusted PR execute?
What can a compromised dependency do?
What can a compromised runner access?
```

---

## 6. Assets

Important assets:

```text
source code
secrets
signing keys
cloud credentials
artifact repositories
CI configuration
release tags
production artifacts
```

---

## 7. Attackers

Possible actors:

```text
external attacker
malicious contributor
compromised developer account
compromised dependency maintainer
insider
compromised CI action
compromised runner
```

---

# PART III — LEAST PRIVILEGE

## 8. Principle

Every identity should receive only the permissions required.

Bad:

```text
CI
 |
v
administrator
```

Better:

```text
CI
 |
+--> read source
+--> build
+--> publish required artifact
```

---

## 9. Separate Permissions

Separate:

```text
read
build
publish
promote
deploy
delete
admin
```

where practical.

---

# PART IV — CI IDENTITY

## 10. CI Identity

A pipeline should have a distinct identity.

Example:

```text
release-workflow
 |
v
artifact-publish permission
```

Do not use a personal developer account for production publishing.

---

# PART V — SHORT-LIVED CREDENTIALS

## 11. Long-Lived Secrets

Long-lived credentials increase:

```text
exposure window
rotation effort
blast radius
```

Prefer short-lived credentials where the platform supports them.

---

## 12. OIDC

Concept:

```text
CI
 |
v
OIDC identity
 |
v
cloud/provider trust
 |
v
short-lived credentials
```

This can remove the need to store long-lived cloud access keys.

---

# PART VI — SECRET MANAGEMENT

## 13. Never Commit Secrets

Never commit:

```text
password
API token
private key
cloud secret
repository credential
signing key
```

to source control.

---

## 14. Secret Sources

Use approved mechanisms such as:

```text
CI secret store
cloud secret manager
vault
OIDC
workload identity
```

---

## 15. Secret Masking

CI systems should mask known secret values in logs.

Do not rely on masking as the only protection.

Avoid commands that print environment variables.

---

# PART VII — SECRET SCANNING

## 16. Secret Scan

Scan:

```text
commits
pull requests
repositories
build outputs
container layers
```

where appropriate.

---

## 17. If Secret Is Committed

Do not merely delete it from the latest commit.

Immediately consider:

```text
revoke
rotate
audit
remove from repository history if required
```

The credential must be considered exposed.

---

# PART VIII — PULL REQUEST SECURITY

## 18. Untrusted PRs

A PR can contain malicious build instructions.

Example:

```text
PR
 |
v
CI
 |
v
malicious script
 |
v
secret theft
```

---

## 19. PR Controls

Use:

```text
restricted permissions
read-only tokens
isolated runners
no production credentials
protected environments
```

---

# PART IX — WORKFLOW SECURITY

## 20. CI Configuration

CI YAML is executable infrastructure.

Protect:

```text
.github/workflows
Jenkinsfiles
.gitlab-ci.yml
build scripts
Dockerfiles
release scripts
```

---

## 21. Workflow Review

Review changes to:

```text
permissions
actions
scripts
credentials
publish destinations
deployment commands
```

---

# PART X — THIRD-PARTY ACTIONS

## 22. CI Actions

Third-party actions execute code inside the build environment.

Risks include:

```text
compromised action
malicious update
excessive permissions
credential access
```

---

## 23. Action Controls

Use:

```text
trusted actions
version pinning/control
minimal permissions
review process
allowlists where appropriate
```

---

# PART XI — DEPENDENCY SECURITY

## 24. Dependencies

Every dependency increases the software supply-chain surface.

Examples:

```text
Maven dependency
npm dependency
Python package
Docker base image
GitHub Action
Helm dependency
```

---

## 25. Dependency Inventory

Know:

```text
what dependencies exist
which versions
where they came from
which artifacts consume them
```

---

# PART XII — DEPENDENCY PINNING

## 26. Pinning

Production builds should use controlled versions.

Examples:

```text
lockfile
constraints
exact package version
approved dependency set
```

Avoid uncontrolled floating dependencies.

---

# PART XIII — TRANSITIVE DEPENDENCIES

## 27. Dependency Graph

```text
application
 |
+--> library-a
      |
      +--> library-b
            |
            +--> library-c
```

Security review must account for transitive dependencies.

---

# PART XIV — VULNERABILITY SCANNING

## 28. SCA

Software Composition Analysis identifies known vulnerabilities in
dependencies.

Typical flow:

```text
dependency
 |
v
database
 |
v
CVE
 |
v
severity
 |
v
policy
```

---

## 29. Severity

Common classification:

```text
Critical
High
Medium
Low
```

Organizations should define their own release policy.

---

# PART XV — VULNERABILITY GATES

## 30. Gate

Example policy:

```text
Critical -> block
High -> block
Medium -> review
Low -> track
```

The actual policy must reflect application risk and organizational
requirements.

---

# PART XVI — FALSE POSITIVES

## 31. False Positive

A scanner can report a vulnerability that does not apply to the actual
runtime path.

Do not blindly suppress it.

Validate:

```text
affected component
affected version
runtime usage
exploitability
scanner evidence
```

---

# PART XVII — EXCEPTIONS

## 32. Vulnerability Exception

If a vulnerability cannot immediately be fixed, record:

```text
CVE
reason
risk assessment
owner
mitigation
expiry date
approval
```

Never create permanent unexplained exceptions.

---

# PART XVIII — SECRET SCANNING

## 33. Secret Types

Scan for:

```text
cloud keys
API tokens
database credentials
private keys
registry tokens
webhook secrets
```

---

# PART XIX — SAST

## 34. Static Analysis

SAST analyzes source code without executing the application.

It can detect:

```text
injection
unsafe APIs
authentication issues
data-flow problems
```

Use appropriate tools for the language.

---

# PART XX — DAST

## 35. Dynamic Analysis

DAST tests a running application.

Concept:

```text
Build
 |
v
Deploy test environment
 |
v
DAST
```

DAST complements SAST; it does not replace it.

---

# PART XXI — IAST

## 36. IAST

Interactive approaches can observe application behavior during testing.

Use only where the operational model supports it.

---

# PART XXII — CONTAINER SECURITY

## 37. Image Scanning

Scan:

```text
base image
OS packages
application dependencies
configuration
secrets
```

---

## 38. Minimal Base Images

Smaller images can reduce attack surface.

But do not remove runtime requirements blindly.

---

# PART XXIII — DOCKERFILE SECURITY

## 39. Secure Dockerfile

Avoid:

```dockerfile
COPY . .
```

when the context contains secrets or unnecessary files.

Use:

```text
.dockerignore
```

and controlled build inputs.

---

# PART XXIV — SECRET IN BUILD

## 40. Build Secrets

Do not put secrets into:

```text
Dockerfile ENV
image layer
package
artifact
source
```

Use the build platform's secure secret mechanism where needed.

---

# PART XXV — ROOTLESS

## 41. Containers

Where supported and compatible, avoid running application processes as
root.

Example:

```dockerfile
USER app
```

---

# PART XXVI — BASE IMAGE TRUST

## 42. Base Images

Use:

```text
approved base images
trusted registries
known versions
scanning
```

---

# PART XXVII — DEPENDENCY CONFUSION

## 43. Dependency Confusion

An attacker publishes a malicious package with the same name as an
internal package to a public registry.

Potential flow:

```text
CI
 |
v
package resolver
 |
v
public malicious package
```

---

## 44. Prevention

Use:

```text
private registry
namespace controls
dependency pinning
repository priority
package-source policy
```

---

# PART XXVIII — TYPOSQUATTING

## 45. Typosquatting

Attackers publish packages with names similar to legitimate packages.

Example concept:

```text
company-auth
company-authh
```

Review dependencies carefully.

---

# PART XXIX — MALICIOUS PACKAGE

## 46. Package Review

Check:

```text
publisher
source
release history
maintainer
dependency behavior
install scripts
```

Automated controls should complement human review.

---

# PART XXX — PACKAGE MANAGER SCRIPTS

## 47. npm Scripts

npm packages can execute lifecycle scripts.

This makes package provenance and dependency trust especially important.

---

# PART XXXI — PYTHON INSTALLATION

## 48. Python

Python packages may contain executable build/install logic.

Use trusted repositories, controlled dependencies, scanning, and
reproducible environments.

---

# PART XXXII — MAVEN PLUGINS

## 49. Maven Plugins

Build plugins execute code during builds.

Treat them as supply-chain dependencies.

Control:

```text
plugin versions
repositories
permissions
```

---

# PART XXXIII — GITHUB ACTION SECURITY

## 50. Actions as Dependencies

A workflow action is executable code.

Control:

```text
action source
version
permissions
inputs
secrets
```

---

# PART XXXIV — BUILDER ISOLATION

## 51. Runner Isolation

A compromised build should not have unrestricted access to:

```text
other builds
production
cloud accounts
signing keys
artifact admin
```

---

## 52. Ephemeral Runners

Ephemeral runners reduce persistent state.

```text
Job
 |
v
New runner
 |
v
Build
 |
v
Destroy
```

---

# PART XXXV — PERSISTENT RUNNER RISK

## 53. Persistent Agents

Persistent agents can retain:

```text
source
credentials
build outputs
caches
malicious files
```

Clean and harden them if used.

---

# PART XXXVI — NETWORK ISOLATION

## 54. Build Network

Restrict runner network access where practical.

Separate:

```text
public internet
internal repositories
production networks
```

---

# PART XXXVII — EGRESS CONTROL

## 55. Egress

A malicious dependency may attempt to exfiltrate data.

Controlled egress can reduce:

```text
data theft
command-and-control
unexpected external access
```

---

# PART XXXVIII — ARTIFACT INTEGRITY

## 56. Integrity

Use:

```text
checksums
digests
signatures
provenance
```

to verify artifacts.

---

# PART XXXIX — SIGNING

## 57. Signing Flow

```text
Build
 |
v
Artifact
 |
v
Sign
 |
v
Repository
 |
v
Verify
 |
v
Deploy
```

---

# PART XL — SIGNING KEY SECURITY

## 58. Signing Keys

Protect signing keys with:

```text
KMS
HSM
dedicated signing service
restricted identity
```

where supported.

Never store production signing private keys in source control.

---

# PART XLI — KEY ROTATION

## 59. Rotation

Define:

```text
rotation
revocation
replacement
verification
```

procedures.

---

# PART XLII — PROVENANCE

## 60. Provenance

Provenance records how an artifact was produced.

Useful fields:

```text
source
builder
workflow
commit
dependencies
timestamp
artifact digest
```

---

# PART XLIII — SBOM

## 61. SBOM

An SBOM provides a component inventory.

Use it for:

```text
CVE response
license analysis
dependency visibility
customer disclosure
incident response
```

---

# PART XLIV — SBOM FORMATS

## 62. Common Formats

Common industry formats include:

```text
CycloneDX
SPDX
```

Choose according to organizational and regulatory requirements.

---

# PART XLV — ARTIFACT REPOSITORY SECURITY

## 63. Repository Controls

Protect:

```text
repositories
users
projects
tokens
admin functions
delete operations
```

---

# PART XLVI — IMMUTABILITY

## 64. Release Immutability

Once:

```text
payment-api:1.4.2
```

is released, do not overwrite it.

---

# PART XLVII — REPOSITORY RBAC

## 65. RBAC

Separate:

```text
consumer
publisher
promoter
administrator
```

roles.

---

# PART XLVIII — DELETE PROTECTION

## 66. Delete

Production release identities should generally not have broad delete
permissions.

Deletion can destroy:

```text
rollback
audit
forensics
```

---

# PART XLIX — ARTIFACT PROMOTION

## 67. Promotion

```text
candidate
 |
v
validated
 |
v
production
```

Promotion should preserve artifact identity.

---

# PART L — BUILD REPRODUCIBILITY

## 68. Reproducible Build

Controlled inputs:

```text
source
compiler
runtime
dependencies
build scripts
environment
```

should produce a consistent expected artifact.

---

# PART LI — BUILD ENVIRONMENT

## 69. Standardization

Control:

```text
JDK
Maven
Node
npm
Python
pip
Docker
OS
```

versions.

---

# PART LII — LOCKFILES

## 70. Dependency Locks

Examples:

```text
package-lock.json
poetry.lock
requirements constraints
Maven dependency management
```

Use the appropriate mechanism for the ecosystem.

---

# PART LIII — CHECKSUM VERIFICATION

## 71. Dependency Integrity

Where package-manager and repository capabilities support it, verify
dependency integrity through lockfiles, checksums, signatures, or
repository metadata.

---

# PART LIV — TRUSTED REPOSITORIES

## 72. Enterprise Repository

A private repository can provide:

```text
approved dependencies
remote caching
access control
audit
availability
```

---

# PART LV — ARTIFACTORY SECURITY

## 73. Artifactory

Use Artifactory or the organization's approved repository manager to
centralize:

```text
Maven
npm
PyPI
Docker
Helm
```

and apply consistent access and lifecycle controls.

---

# PART LVI — VIRTUAL REPOSITORY SECURITY

## 74. Virtual Repositories

Control which upstream repositories are accessible.

Do not allow unrestricted public package resolution if organizational
policy requires approved sources.

---

# PART LVII — REMOTE REPOSITORY SECURITY

## 75. Remote Cache

Review:

```text
upstream source
TLS
trust
cache policy
package availability
```

---

# PART LVIII — DEPENDENCY ALLOWLIST

## 76. Allowlist

For high-security environments maintain approved:

```text
registries
packages
versions
base images
CI actions
```

where practical.

---

# PART LIX — DENYLIST

## 77. Denylist

Block known:

```text
malicious packages
compromised versions
unapproved registries
dangerous actions
```

A denylist is supplemental, not a replacement for broader controls.

---

# PART LX — LICENSE SECURITY

## 78. License Compliance

Track package licenses.

Potential policy:

```text
approved
review required
prohibited
```

Legal requirements vary by organization and jurisdiction.

---

# PART LXI — MALWARE SCANNING

## 79. Artifact Malware

Repositories may integrate malware/security scanning.

Scan high-risk artifact types and define quarantine behavior.

---

# PART LXII — CI LOG SECURITY

## 80. Logs

Logs may contain:

```text
tokens
URLs with credentials
environment variables
source snippets
customer data
```

Treat logs as sensitive operational data.

---

# PART LXIII — DEBUG MODE

## 81. Debug Logs

Do not enable highly verbose debugging permanently in production
pipelines if it can expose secrets.

Use controlled troubleshooting windows.

---

# PART LXIV — WORKSPACE CLEANUP

## 82. Clean Workspace

After a build:

```text
source
 |
v
build
 |
v
cleanup
```

Ephemeral runners provide stronger isolation.

---

# PART LXV — CACHE SECURITY

## 83. Cache Poisoning

A malicious job may attempt to populate a shared cache.

Controls:

```text
scoped keys
trusted writers
isolated caches
branch trust boundaries
```

---

# PART LXVI — CACHE SECRETS

## 84. Never Cache Secrets

Do not cache:

```text
credentials
tokens
private keys
secret configuration
```

---

# PART LXVII — ARTIFACT CACHE

## 85. Cache vs Artifact

Cache:

```text
disposable acceleration
```

Artifact:

```text
validated software output
```

Never confuse their security requirements.

---

# PART LXVIII — BUILD APPROVAL

## 86. Trusted Release

Use protected workflows/environments for production publication.

---

# PART LXIX — SEPARATION OF DUTIES

## 87. High-Risk Release

Possible model:

```text
Developer
 |
v
Build

Independent Approver
 |
v
Production promotion
```

---

# PART LXX — PROTECTED BRANCHES

## 88. Branch Controls

Protect:

```text
main
release/*
production configuration
CI workflow files
```

---

# PART LXXI — PROTECTED TAGS

## 89. Tags

Protect release tags from unauthorized modification.

---

# PART LXXII — CODE REVIEW

## 90. Review

Require review for:

```text
application code
dependencies
Dockerfiles
CI workflows
release scripts
build configuration
```

---

# PART LXXIII — SECURITY GATES

## 91. Typical Pipeline

```text
Checkout
 |
v
Secret Scan
 |
v
Build
 |
v
SAST
 |
v
SCA
 |
v
Tests
 |
v
Container Scan
 |
v
SBOM
 |
v
Sign/Attest
 |
v
Publish
```

Order can vary according to tooling and cost.

---

# PART LXXIV — FAILING SECURITY GATES

## 92. Policy

A security gate should clearly report:

```text
what failed
why it failed
severity
affected component
remediation
```

---

# PART LXXV — EXCEPTION PROCESS

## 93. Temporary Exception

Record:

```text
risk
owner
expiry
mitigation
approval
```

Automate expiry reminders where possible.

---

# PART LXXVI — BUILD POLICY AS CODE

## 94. Policy

Security policy can be expressed as code.

Examples:

```text
no unapproved registry
no critical CVEs
no unsigned production image
no privileged release workflow
```

---

# PART LXXVII — CONTAINER POLICY

## 95. Admission

Kubernetes can enforce policies such as:

```text
trusted registry
signed image
approved digest
non-root
resource requirements
```

Use the policy engine supported by the environment.

---

# PART LXXVIII — GITOPS SECURITY

## 96. GitOps

Protect:

```text
GitOps repository
deployment manifests
Argo CD credentials
production clusters
```

---

# PART LXXIX — IMAGE PROMOTION

## 97. Promotion

```text
CI
 |
v
scan
 |
v
registry
 |
v
digest
 |
v
GitOps
 |
v
Argo CD
 |
v
Kubernetes
```

---

# PART LXXX — PRODUCTION IDENTITY

## 98. Immutable Identity

Prefer:

```text
image@sha256:...
```

over a mutable tag alone.

---

# PART LXXXI — INCIDENT RESPONSE

## 99. Build Security Incident

Potential incidents:

```text
stolen token
compromised dependency
malicious artifact
tampered workflow
compromised runner
leaked signing key
```

---

# PART LXXXII — INCIDENT FIRST RESPONSE

## 100. First Actions

```text
contain
 |
v
revoke
 |
v
preserve evidence
 |
v
identify impact
 |
v
recover
 |
v
prevent recurrence
```

---

# PART LXXXIII — COMPROMISED CREDENTIAL

## 101. Response

```text
revoke
rotate
audit
identify affected actions
review artifacts
```

---

# PART LXXXIV — COMPROMISED ARTIFACT

## 102. Response

```text
stop promotion
 |
v
quarantine
 |
v
identify consumers
 |
v
identify deployment
 |
v
rebuild trusted
 |
v
publish new version
 |
v
deploy
```

---

# PART LXXXV — COMPROMISED RUNNER

## 103. Response

For an ephemeral runner:

```text
destroy runner
revoke temporary credentials
audit outputs
```

For persistent infrastructure:

```text
isolate
forensics
rebuild
rotate credentials
```

---

# PART LXXXVI — COMPROMISED SIGNING KEY

## 104. Response

```text
revoke trust
 |
v
rotate key
 |
v
identify signed artifacts
 |
v
assess deployments
 |
v
re-sign trusted releases if appropriate
```

---

# PART LXXXVII — AUDIT

## 105. Audit Trail

Record:

```text
source
commit
builder
workflow
artifact
dependency state
security results
signing
publication
promotion
deployment
approval
```

---

# PART LXXXVIII — SECURITY METRICS

## 106. Metrics

Track:

```text
critical vulnerabilities
high vulnerabilities
mean remediation time
secret incidents
failed security gates
unsigned artifacts
dependency age
```

---

# PART LXXXIX — VULNERABILITY SLA

## 107. Example

An organization may define:

```text
Critical -> remediate within X days
High -> within Y days
```

Actual targets should be set by organizational risk policy.

---

# PART XC — DEPENDENCY AGE

## 108. Outdated Dependencies

Track:

```text
current version
used version
age
security status
```

---

# PART XCI — SECURITY DEBT

## 109. Security Debt

Security debt can include:

```text
old dependencies
unmaintained base images
long-lived credentials
unreviewed actions
unpatched build tools
```

Track and reduce it deliberately.

---

# PART XCII — PRODUCTION BUILD ARCHITECTURE

## 110. Reference

```text
                         Git
                          |
                    Protected PR
                          |
                          v
                         CI
                          |
            +-------------+-------------+
            |             |             |
            v             v             v
       Secret Scan      SAST          Build
                                        |
                                        v
                                  Dependency Scan
                                        |
                                        v
                                      Tests
                                        |
                                        v
                                  Container Scan
                                        |
                                        v
                                    SBOM/Attest
                                        |
                                        v
                                   Sign Artifact
                                        |
                                        v
                                   Artifactory
                                        |
                                        v
                                    Promote
                                        |
                                        v
                                    GitOps
                                        |
                                        v
                                    Argo CD
                                        |
                                        v
                                   Kubernetes
```

---

# PART XCIII — SECURITY CHECKLIST

## 111. Source

```text
[ ] protected branches
[ ] code review
[ ] secret scanning
[ ] workflow review
```

## 112. CI

```text
[ ] least privilege
[ ] isolated runners
[ ] no production secrets in PRs
[ ] controlled third-party actions
[ ] secure logs
```

## 113. Dependencies

```text
[ ] lock/constraints
[ ] SCA
[ ] trusted registries
[ ] transitive dependency review
[ ] exception process
```

## 114. Artifacts

```text
[ ] immutable
[ ] checksum/digest
[ ] SBOM
[ ] provenance
[ ] signing where required
```

## 115. Repository

```text
[ ] RBAC
[ ] delete protection
[ ] audit
[ ] backup
[ ] retention
```

## 116. Deployment

```text
[ ] trusted registry
[ ] immutable digest
[ ] admission policy where required
[ ] monitoring
[ ] rollback
```

---

# PART XCIV — INTERVIEW PREPARATION

## 117. What Is Build Security?

Answer:

```text
Build security protects the entire software supply chain from source
through dependencies, CI runners, credentials, build tools, artifacts,
repositories and promotion. The objective is to ensure that only
trusted and traceable artifacts reach production.
```

## 118. How Do You Secure CI/CD?

Answer:

```text
I use least-privilege identities, protected branches and environments,
isolated runners, secure secret management, dependency scanning, SAST,
artifact scanning, provenance, SBOMs, signing where required and
immutable artifact promotion.
```

## 119. How Do You Protect Secrets?

Answer:

```text
I never store secrets in source. I use the CI secret manager or
federated identity, prefer short-lived credentials, restrict secret
access to trusted jobs and ensure logs cannot expose credentials.
```

## 120. How Do You Secure Pull Requests?

Answer:

```text
I treat PR code as potentially untrusted. PR jobs use restricted
permissions and should not receive production credentials or publishing
permissions. Sensitive release operations run only from trusted
contexts.
```

## 121. What Is Dependency Confusion?

Answer:

```text
It occurs when a package resolver selects a malicious public package
instead of an intended internal package. I mitigate it with private
repositories, namespaces, explicit dependency sources, pinning and
repository policy.
```

## 122. Why Is Artifact Signing Important?

Answer:

```text
Signing provides a mechanism to verify that an artifact was produced or
approved by a trusted identity and was not altered after signing.
```

## 123. What Is Provenance?

Answer:

```text
Provenance describes how an artifact was produced, including source,
builder, workflow and relevant inputs. It allows us to establish the
source-to-artifact relationship.
```

## 124. Why Do You Need an SBOM?

Answer:

```text
An SBOM gives us a component inventory for a release. During a
vulnerability incident we can quickly determine which artifacts contain
the affected component.
```

## 125. How Do You Secure Docker Builds?

Answer:

```text
I use trusted and scanned base images, small build contexts, secure
build secrets, non-root runtime users where appropriate, dependency
scanning, image scanning and immutable image digests.
```

---

# PART XCV — SENIOR-LEVEL SCENARIOS

## 126. Malicious Code Added to CI Workflow

Answer:

```text
I stop affected workflows, revoke potentially exposed credentials,
inspect workflow history and artifacts, determine whether malicious
code executed, rebuild trusted artifacts, and strengthen workflow
review and permissions.
```

## 127. Critical CVE Appears in Production Artifact

Answer:

```text
I identify all affected artifact versions and deployments using the
SBOM and artifact lineage, assess exposure, create a patched release,
scan it, publish it immutably and deploy it through the controlled
promotion process.
```

## 128. PR Can Access Production Token

Answer:

```text
I treat this as a serious CI security design flaw. I revoke or rotate
the exposed credential, audit access, remove production secrets from
untrusted workflows and redesign the workflow around protected
environments and least privilege.
```

## 129. Third-Party Action Is Compromised

Answer:

```text
I identify workflows using the action, pin or replace the affected
version, inspect execution history and artifacts, rotate exposed
credentials if required, and establish stronger action trust controls.
```

## 130. Build Runner Is Compromised

Answer:

```text
I isolate or destroy the runner, revoke credentials available to it,
review generated artifacts and logs, determine whether source or
secrets were accessed, rebuild trusted artifacts and restore clean
runner infrastructure.
```

## 131. Signing Key Leaked

Answer:

```text
I revoke the key's trust, rotate the signing key, identify artifacts
signed with the affected key, assess whether they were deployed, and
re-establish trust using the new signing identity.
```

## 132. Dependency Is Malicious but Scanner Finds No CVE

Answer:

```text
A vulnerability database cannot detect every malicious package. I use
trusted sources, package review, provenance, behavioral controls,
repository policy, dependency pinning and supply-chain monitoring in
addition to CVE scanning.
```

## 133. Artifact Repository Is Compromised

Answer:

```text
I isolate the repository, preserve evidence, determine which artifacts
and metadata were modified, verify trusted copies through checksums or
signatures, rotate credentials, restore clean artifacts and audit
downstream deployments.
```

## 134. Need Secure Air-Gapped Builds

Answer:

```text
I establish an approved dependency and artifact transfer process,
verify checksums and provenance at the transfer boundary, maintain an
offline repository and prevent uncontrolled external network access.
```

## 135. Security Scans Make CI Too Slow

Answer:

```text
I profile scan duration, cache approved databases where safe, parallelize
independent scans, use appropriate incremental strategies and move
non-blocking analysis to suitable stages. I preserve mandatory security
gates rather than simply disabling them.
```

## 136. Many False Positives

Answer:

```text
I create a controlled exception process with evidence, owner,
mitigation, approval and expiry. I tune the scanner, but I do not create
permanent blanket suppressions.
```

## 137. Developer Wants to Use Public npm Directly

Answer:

```text
For enterprise builds I prefer the approved repository or virtual
repository so dependencies are governed, auditable and cacheable. If
public access is permitted, it should still follow the organization's
dependency and security policy.
```

## 138. Production Uses latest Tag

Answer:

```text
I would change production deployment identity to a versioned immutable
artifact and preferably an image digest. latest is useful for some
development workflows but is a poor sole production identity.
```

## 139. Need Prove Which Commit Produced Production

Answer:

```text
I trace deployment to the immutable artifact digest, artifact to CI
build, CI build to source commit/tag, and validate the associated
provenance and release records.
```

## 140. Security Exception Has No Expiry

Answer:

```text
I consider that uncontrolled security debt. The exception should have
an owner, risk justification, mitigation, approval and expiry date.
```

---

# PART XCVI — GOLDEN RULES

## 141. Rules 1

```text
1. Treat build security as supply-chain security.
2. Protect source.
3. Protect CI configuration.
4. Protect dependencies.
5. Protect build tools.
6. Protect runners.
7. Protect credentials.
8. Protect artifacts.
9. Protect repositories.
10. Protect deployment inputs.
11. Use least privilege.
12. Separate read/write/admin permissions.
13. Use dedicated CI identities.
14. Do not use personal accounts for production publishing.
15. Prefer short-lived credentials.
16. Prefer OIDC/federated identity where supported.
17. Never commit secrets.
18. Rotate exposed credentials immediately.
19. Audit leaked credentials.
20. Do not print secrets in logs.
```

## 142. Rules 2

```text
21. Treat PR code as potentially untrusted.
22. Do not expose production credentials to untrusted PRs.
23. Restrict PR token permissions.
24. Isolate sensitive release jobs.
25. Protect CI workflow files.
26. Review changes to release workflows.
27. Review third-party CI actions.
28. Control action versions.
29. Minimize workflow permissions.
30. Maintain trusted action policies.
31. Inventory dependencies.
32. Scan direct dependencies.
33. Scan transitive dependencies.
34. Pin production dependencies appropriately.
35. Use lockfiles/constraints.
36. Protect against dependency confusion.
37. Protect against typosquatting.
38. Use private registries for internal packages.
39. Control public dependency sources.
40. Review suspicious packages.
```

## 143. Rules 3

```text
41. Use SCA.
42. Define vulnerability severity policy.
43. Block critical issues when policy requires it.
44. Review high-risk findings.
45. Manage exceptions explicitly.
46. Give exceptions owners.
47. Give exceptions expiry dates.
48. Record mitigation.
49. Use SAST.
50. Use DAST where appropriate.
51. Scan containers.
52. Scan base images.
53. Protect Docker build secrets.
54. Use .dockerignore.
55. Avoid secrets in image layers.
56. Prefer non-root containers where appropriate.
57. Use trusted base images.
58. Keep base images maintained.
59. Scan Helm and deployment artifacts where appropriate.
60. Secure build scripts.
```

## 144. Rules 4

```text
61. Use ephemeral runners where practical.
62. Harden persistent runners.
63. Clean persistent workspaces.
64. Restrict build network access.
65. Control egress for sensitive builds.
66. Prevent cross-job contamination.
67. Protect caches from poisoning.
68. Never cache secrets.
69. Scope cache keys.
70. Trust only approved cache writers.
71. Use checksums.
72. Use immutable container digests.
73. Use signatures where required.
74. Protect signing keys.
75. Use KMS/HSM/signing services where appropriate.
76. Rotate signing keys.
77. Maintain provenance.
78. Generate SBOMs where required.
79. Associate SBOMs with exact artifacts.
80. Verify artifacts before deployment.
```

## 145. Rules 5

```text
81. Keep release artifacts immutable.
82. Protect artifact repositories.
83. Use repository RBAC.
84. Restrict delete operations.
85. Audit repository activity.
86. Control remote repositories.
87. Control virtual repository upstreams.
88. Prefer approved dependency mirrors.
89. Monitor repository health.
90. Maintain repository backups.
91. Test repository recovery.
92. Separate candidate and release lifecycle.
93. Promote instead of rebuilding.
94. Preserve artifact identity during promotion.
95. Use protected production environments.
96. Use separation of duties for high-risk releases.
97. Protect release branches.
98. Protect release tags.
99. Keep release evidence.
100. Maintain source-to-artifact lineage.
```

## 146. Rules 6

```text
101. Maintain artifact-to-deployment lineage.
102. Track build identity.
103. Track dependency state.
104. Track security results.
105. Track approval.
106. Track promotion.
107. Track deployment.
108. Monitor security-gate failures.
109. Monitor vulnerability age.
110. Monitor remediation time.
111. Monitor secret incidents.
112. Monitor unsigned artifacts.
113. Monitor dependency age.
114. Monitor outdated base images.
115. Test incident-response procedures.
116. Test credential rotation.
117. Test artifact recovery.
118. Test rollback.
119. Test compromised-runner recovery.
120. Test signing-key recovery.
```

## 147. Rules 7

```text
121. Stop promotion when artifact integrity is uncertain.
122. Quarantine suspicious artifacts.
123. Identify consumers of compromised artifacts.
124. Rebuild from trusted source after compromise.
125. Publish a new version after fixing compromised content.
126. Do not mutate old production artifacts.
127. Preserve forensic evidence.
128. Revoke compromised credentials.
129. Rotate exposed secrets.
130. Review CI access after incidents.
131. Review repository access after incidents.
132. Review signing trust after incidents.
133. Maintain emergency security release procedures.
134. Maintain vulnerability response procedures.
135. Maintain dependency incident procedures.
136. Maintain supply-chain incident procedures.
137. Maintain security exception procedures.
138. Automate exception expiry where possible.
139. Keep policies version-controlled.
140. Treat policy changes as security-sensitive.
```

## 148. Rules 8

```text
141. Secure build tools.
142. Secure package managers.
143. Secure Maven plugins.
144. Secure npm lifecycle scripts.
145. Secure Python build backends.
146. Secure Docker builders.
147. Secure Helm dependencies.
148. Secure GitHub Actions.
149. Secure Jenkins agents.
150. Secure GitLab runners.
151. Use approved registries.
152. Use approved base images.
153. Use approved CI components.
154. Review package publishers.
155. Verify dependency provenance where available.
156. Do not assume a package is safe merely because it has no CVE.
157. Do not assume scanners detect every supply-chain attack.
158. Layer automated and manual controls.
159. Prefer deterministic builds.
160. Validate the exact security architecture used in production.
```

---