# Jenkins-Artifactory-Integration

## 1. Purpose

This file covers production-grade Jenkins and JFrog Artifactory integration.

It covers:

- Jenkins-Artifactory architecture
- CI/CD integration patterns
- JFrog CLI
- JFrog Jenkins integrations
- Authentication
- Credentials
- Service accounts
- Access tokens
- Maven
- Gradle
- NPM
- PyPI
- Docker
- Helm
- Generic artifacts
- Build Info
- Artifact upload/download
- Promotion
- Release pipelines
- Security scanning
- Webhooks and callbacks concepts
- Pipeline design
- Shared libraries
- Parallel builds
- Multibranch pipelines
- Kubernetes agents
- EKS-based Jenkins
- Production security
- Credential rotation
- Troubleshooting
- Real-world scenarios
- Interview preparation

---

# PART I — FUNDAMENTALS

## 2. Why Integrate Jenkins with Artifactory?

Jenkins performs CI/CD orchestration.

Artifactory provides artifact management.

Typical architecture:

```text
Developer
   |
   v
Git
   |
   v
Jenkins
   |
   +--> Build
   +--> Test
   +--> Security Scan
   |
   v
Artifactory
   |
   +--> Store
   +--> Version
   +--> Promote
   +--> Distribute
```

---

## 3. Jenkins Responsibilities

Jenkins commonly handles:

```text
checkout
build
test
package
scan
publish
promotion
deployment
```

---

## 4. Artifactory Responsibilities

Artifactory commonly handles:

```text
artifact storage
repository management
dependency proxying
versioned packages
artifact metadata
Build Info
access control
promotion workflows
retention
```

---

## 5. Why Not Store Build Artifacts in Jenkins?

Jenkins artifacts are useful for CI job output, but Artifactory is designed for enterprise artifact management.

Artifactory provides:

```text
centralized repository
artifact versioning
RBAC
remote repositories
virtual repositories
high availability options
retention
auditability
multi-tool package support
```

---

# PART II — ARCHITECTURE

## 6. Basic Architecture

```text
                   Git
                    |
                    v
                 Jenkins
                    |
          +---------+---------+
          |         |         |
        Build      Test      Scan
          |         |         |
          +---------+---------+
                    |
                    v
               Artifactory
                    |
          +---------+---------+
          |         |         |
        Maven     Docker     Helm
```

---

## 7. Dependency Flow

A build can consume dependencies from Artifactory:

```text
Jenkins
   |
   v
Artifactory Virtual Repository
   |
   +--> Local
   |
   +--> Remote
```

This gives CI a controlled package source.

---

## 8. Publish Flow

```text
Jenkins
   |
   v
Build
   |
   v
Artifact
   |
   v
Artifactory Local Repository
```

---

## 9. Promotion Flow

```text
Build
 ↓
Artifactory
 ↓
Validate
 ↓
Promote
 ↓
Release Repository
 ↓
Deploy
```

---

# PART III — AUTHENTICATION

## 10. Jenkins Identity

Jenkins should authenticate to Artifactory using a dedicated service identity.

Preferred:

```text
Jenkins
 ↓
Service Identity
 ↓
Scoped Access Token
 ↓
Artifactory
```

---

## 11. Avoid Admin Credentials

Bad:

```text
Jenkins
 ↓
Artifactory Admin
```

This creates a large blast radius.

---

## 12. Least Privilege

Example build identity:

```text
READ:
maven-virtual
npm-virtual
docker-virtual

DEPLOY:
payment-maven-local
payment-docker-local

DELETE:
NO

ADMIN:
NO
```

---

## 13. Jenkins Credentials

Store Artifactory credentials in:

```text
Jenkins Credentials Store
```

Never place secrets directly in:

```text
Jenkinsfile
Git repository
Dockerfile
shell scripts
```

---

## 14. Token Rotation

A production process should support:

```text
create replacement token
 ↓
update Jenkins credential
 ↓
test pipelines
 ↓
revoke old token
```

---

# PART IV — JFROG CLI

## 15. JFrog CLI

JFrog CLI can automate interactions with Artifactory.

Conceptually:

```text
Jenkins
 ↓
JFrog CLI
 ↓
Artifactory
```

---

## 16. Configure CLI

The CI environment should authenticate using a secure service identity.

Example concept:

```bash
jf config add company-artifactory
```

Do not expose credentials in command output or logs.

---

## 17. CI Configuration

A robust pipeline should provide:

```text
Artifactory URL
repository
build name
build number
credentials
```

through secure configuration.

---

# PART V — JENKINS PIPELINE STRUCTURE

## 18. Recommended Pipeline

```text
Checkout
   ↓
Version
   ↓
Build
   ↓
Unit Test
   ↓
Package
   ↓
Security Scan
   ↓
Publish
   ↓
Publish Build Info
   ↓
Integration Test
   ↓
Promotion
   ↓
Deploy
```

---

## 19. Declarative Pipeline Example

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Publish') {
            steps {
                sh 'jf rt upload ...'
            }
        }

        stage('Build Info') {
            steps {
                sh 'jf rt build-publish ...'
            }
        }
    }
}
```

Use organization-specific repository names and secure credential handling.

---

# PART VI — MAVEN INTEGRATION

## 20. Maven Architecture

```text
Jenkins
 ↓
Maven
 ↓
Artifactory Virtual
 ↓
Dependencies
```

Publish:

```text
Jenkins
 ↓
Maven Deploy
 ↓
Artifactory Local
```

---

## 21. Maven Settings

A common pattern is to configure Maven with Artifactory repositories and credentials through managed settings.

Example:

```xml
<server>
    <id>company-artifactory</id>
    <username>${env.ARTIFACTORY_USER}</username>
    <password>${env.ARTIFACTORY_TOKEN}</password>
</server>
```

In production, prefer Jenkins credential bindings rather than exposing credentials as ordinary environment variables.

---

## 22. Maven Build

```bash
mvn clean verify
```

---

## 23. Maven Publish

```bash
mvn deploy
```

The deployment repository should be explicitly controlled.

---

## 24. Maven Production Pattern

```text
maven-virtual
     ↓
dependency READ

payment-maven-local
     ↑
artifact DEPLOY
```

---

# PART VII — GRADLE INTEGRATION

## 25. Gradle

Gradle can resolve dependencies through Artifactory and publish artifacts to Artifactory.

Architecture:

```text
Jenkins
 ↓
Gradle
 ↓
Artifactory
```

---

## 26. Dependency Resolution

```text
Gradle
 ↓
Artifactory Virtual
 ↓
Dependencies
```

---

## 27. Publishing

```text
Gradle
 ↓
Artifactory Local
```

Keep publishing credentials separate from read-only dependency credentials where practical.

---

# PART VIII — NPM INTEGRATION

## 28. NPM Dependency Flow

```text
npm ci
 ↓
Artifactory npm-virtual
 ↓
Dependencies
```

---

## 29. NPM Publish

```bash
npm publish
```

Publish to the intended local repository.

---

## 30. NPM Configuration

A typical project may use an `.npmrc` configured for the organization's Artifactory endpoint.

Credentials should come from Jenkins-managed secrets.

---

## 31. NPM Production Security

Avoid:

```text
plaintext token in repository
```

Prefer:

```text
Jenkins credential
 ↓
temporary pipeline configuration
```

---

# PART IX — PYPI INTEGRATION

## 32. Python Build

```bash
python -m build
```

---

## 33. Python Publish

A pipeline can publish distributions to Artifactory using the approved Python packaging tooling.

Conceptually:

```text
Build
 ↓
Wheel/sdist
 ↓
Artifactory PyPI local
```

---

## 34. Python Consumption

```text
pip
 ↓
Artifactory PyPI virtual
 ↓
Dependencies
```

---

# PART X — DOCKER INTEGRATION

## 35. Docker Architecture

```text
Jenkins
 ↓
docker build
 ↓
Security Scan
 ↓
docker push
 ↓
Artifactory Docker Repository
```

---

## 36. Docker Login

Use Jenkins credentials securely.

Example concept:

```bash
docker login artifactory.company.com
```

Do not place static passwords in the Jenkinsfile.

---

## 37. Docker Build

```bash
docker build \
  -t artifactory.company.com/docker-local/payment-service:4.2.1 \
  .
```

---

## 38. Docker Push

```bash
docker push \
  artifactory.company.com/docker-local/payment-service:4.2.1
```

---

## 39. Digest

After push, capture the image digest:

```text
sha256:...
```

Use the digest for high-assurance production traceability.

---

# PART XI — HELM INTEGRATION

## 40. Helm Build

```bash
helm lint ./chart
helm package ./chart
```

---

## 41. Helm Publish

Publish the packaged chart to the intended Artifactory Helm or OCI repository according to the organization's chosen distribution model.

---

## 42. Helm + Docker

Example:

```text
Helm Chart:
1.4.0

Application:
4.2.1

Docker:
4.2.1
```

The pipeline should keep the relationship traceable.

---

# PART XII — GENERIC ARTIFACTS

## 43. Generic Artifacts

Artifactory can store files that do not use a package-specific format.

Examples:

```text
zip
tar.gz
configuration bundles
release documentation
installer packages
```

---

## 44. Upload

Conceptually:

```bash
jf rt upload \
  "dist/*" \
  "generic-local/payment/4.2.1/"
```

---

## 45. Generic Artifact Versioning

Use immutable paths where appropriate:

```text
payment/
 └── 4.2.1/
```

---

# PART XIII — BUILD INFO

## 46. Build Info

Jenkins should publish Build Info where the integration supports it.

It can connect:

```text
Git commit
 ↓
Jenkins build
 ↓
Dependencies
 ↓
Artifacts
```

---

## 47. Example

```text
Build:
payment-service #721

Commit:
abc1234

Artifacts:
payment-service-4.2.1.jar
payment-service:4.2.1
```

---

## 48. Why Build Info Matters

It helps with:

```text
release traceability
dependency analysis
security investigations
promotion
audit
```

---

# PART XIV — BUILD ONCE, PROMOTE MANY

## 49. Recommended Pattern

```text
Jenkins
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Build Info
 ↓
Promote
 ↓
Stage
 ↓
Production
```

---

## 50. Do Not Rebuild for Production

Avoid:

```text
Build DEV
 ↓
Rebuild STAGE
 ↓
Rebuild PROD
```

Prefer:

```text
Build once
 ↓
Promote exact artifact
```

---

## 51. Promotion

The promotion process should preserve:

```text
same content
same checksum
same image digest
same artifact version
```

---

# PART XV — SECURITY SCANNING

## 52. Scan Before Release

Typical:

```text
Build
 ↓
Dependency Scan
 ↓
Container Scan
 ↓
Quality Gate
 ↓
Publish/Promote
```

---

## 53. Vulnerability Gate

Example policy:

```text
Critical vulnerability
→ block release
```

The exact threshold should be defined by security policy.

---

## 54. License Scanning

For organizations with license requirements:

```text
dependency
 ↓
license evaluation
 ↓
allow/block
```

---

# PART XVI — JENKINS MULTIBRANCH

## 55. Multibranch Pipelines

A repository may contain:

```text
main
develop
feature/*
release/*
```

Jenkins can create pipelines for branches automatically.

---

## 56. Branch Artifact Strategy

Example:

```text
feature:
payment-service:build-721-abc1234

release:
payment-service:4.2.1-rc.1

main:
payment-service:4.2.1
```

Use a clearly documented versioning policy.

---

# PART XVII — PARALLEL BUILDS

## 57. Parallel Pipeline

Example:

```text
             +--> Maven
             |
Checkout --->+--> NPM
             |
             +--> Docker
```

---

## 58. Build Number Collision

Parallel jobs must not accidentally publish the same immutable release version.

Use:

```text
Git release tag
unique CI build identity
controlled release locks
```

---

# PART XVIII — JENKINS SHARED LIBRARIES

## 59. Why Shared Libraries?

Large organizations may have many pipelines.

Instead of repeating:

```text
Artifactory authentication
JFrog CLI setup
Build Info
promotion
```

create reusable pipeline functions.

---

## 60. Example

Conceptually:

```groovy
artifactoryBuild(
    repository: 'docker-local',
    version: releaseVersion
)
```

---

## 61. Benefits

```text
standardization
security
less duplication
faster onboarding
centralized improvements
```

---

# PART XIX — KUBERNETES JENKINS AGENTS

## 62. Kubernetes Agents

Jenkins can create ephemeral agents in Kubernetes.

Architecture:

```text
Jenkins Controller
       |
       v
Kubernetes
       |
       +--> Ephemeral Agent
                  |
                  v
             Artifactory
```

---

## 63. Agent Credentials

The agent should receive only the credentials required for its job.

Avoid placing permanent Artifactory admin credentials inside agent images.

---

## 64. Ephemeral Agent Advantage

After the build:

```text
agent destroyed
```

This reduces persistent workspace and credential exposure.

---

# PART XX — EKS-BASED JENKINS

## 65. EKS Architecture

Example:

```text
AWS
 |
 +--> EKS
       |
       +--> Jenkins Controller
       |
       +--> Dynamic Agents
       |
       +--> Application
```

Jenkins agents connect to Artifactory through approved network paths.

---

## 66. Network Requirements

Ensure:

```text
DNS
TLS
routing
security groups
firewalls
proxy
```

allow Jenkins agents to reach Artifactory.

---

## 67. Private Artifactory

If Artifactory is private:

```text
Jenkins
 ↓
Private network
 ↓
Artifactory
```

Use appropriate:

```text
VPN
private connectivity
proxy
routing
```

depending on architecture.

---

# PART XXI — CREDENTIAL MANAGEMENT

## 68. Credential Types

Common machine credentials include:

```text
access tokens
API credentials
username/password
certificates
```

Prefer short-lived or scoped credentials where supported.

---

## 69. Credential Binding

Jenkins should inject credentials only when required.

Conceptually:

```text
Credential Store
 ↓
Pipeline
 ↓
Command
 ↓
Credential removed from scope
```

---

## 70. Log Masking

Ensure secrets do not appear in:

```text
console output
debug logs
shell trace
error messages
```

Avoid commands such as:

```bash
set -x
```

around credential-bearing operations.

---

# PART XXII — PRODUCTION PIPELINE

## 71. Recommended Production Pipeline

```text
Checkout
 ↓
Version Validation
 ↓
Build
 ↓
Unit Test
 ↓
Integration Test
 ↓
SAST
 ↓
Dependency Scan
 ↓
Package
 ↓
Container Scan
 ↓
Publish
 ↓
Build Info
 ↓
Approval
 ↓
Promotion
 ↓
Deploy
 ↓
Smoke Test
 ↓
Monitor
```

---

# PART XXIII — RELEASE APPROVAL

## 72. Approval Gate

Production deployment may require:

```text
manual approval
change record
security approval
business approval
```

---

## 73. Release Identity

Approval should apply to:

```text
specific version
specific artifact
specific digest
```

not a mutable tag such as:

```text
latest
```

---

# PART XXIV — ROLLBACK

## 74. Rollback Strategy

If:

```text
4.2.2
```

fails:

```text
Deploy 4.2.1
```

---

## 75. Rollback Requirement

Keep:

```text
previous artifact
previous digest
previous Helm chart
```

available according to retention policy.

---

## 76. Rollback Pipeline

A rollback should be controlled and auditable.

```text
Select known-good release
 ↓
Validate
 ↓
Deploy
 ↓
Smoke test
 ↓
Monitor
```

---

# PART XXV — WEBHOOKS AND EVENT-DRIVEN FLOWS

## 77. Event-Driven Concepts

Organizations may use events to trigger downstream automation.

Examples:

```text
artifact published
build completed
promotion completed
```

The exact event mechanism depends on the JFrog/Jenkins integration and deployed versions.

---

## 78. Security

Validate:

```text
source
signature/authentication
payload
replay protection
authorization
```

before allowing an event to trigger production actions.

---

# PART XXVI — TROUBLESHOOTING

## 79. Jenkins Cannot Connect to Artifactory

Check:

```text
DNS
network route
TLS certificate
proxy
firewall
Artifactory availability
```

---

## 80. 401 Unauthorized

Check:

```text
credential
token
identity
token expiration
Artifactory URL
```

---

## 81. 403 Forbidden

Check:

```text
permission target
repository
READ/DEPLOY permission
project role
service identity
```

---

## 82. Maven Dependency Download Fails

Check:

```text
settings.xml
repository URL
credentials
virtual repository
dependency availability
```

---

## 83. Maven Deploy Fails

Check:

```text
deployment repository
version
DEPLOY permission
authentication
release immutability
```

---

## 84. Docker Push Fails

Check:

```text
registry URL
docker login
token
repository permission
image tag
TLS
network
```

---

## 85. Docker Pull Works but Push Fails

Likely:

```text
READ allowed
DEPLOY missing
```

This is usually an authorization issue.

---

## 86. Build Info Missing

Check:

```text
build name
build number
JFrog CLI/integration
publish-build-info step
service permissions
```

---

## 87. Pipeline Logs Expose Token

Immediate:

```text
revoke/rotate token
 ↓
remove unsafe logging
 ↓
review exposure
 ↓
audit
```

---

## 88. Jenkins Agent Cannot Resolve Artifactory

Check:

```text
DNS
CoreDNS
VPC DNS
network policy
security groups
proxy
private DNS
```

---

# PART XXVII — REAL-WORLD SCENARIOS

## 89. Scenario — Jenkins Uses Admin Token

Problem:

```text
Jenkins → Admin
```

Solution:

```text
Create service identity
 ↓
Grant required permissions
 ↓
Update Jenkins credential
 ↓
Test pipelines
 ↓
Revoke admin token
```

---

## 90. Scenario — Multiple Teams Share Jenkins

Use:

```text
team-specific repositories
team-specific service identities
shared read-only virtual repositories
```

Avoid one universal publishing identity when practical.

---

## 91. Scenario — Compromised Jenkins Agent

Response:

```text
Terminate agent
 ↓
Revoke affected credentials
 ↓
Review pipeline activity
 ↓
Review Artifactory audit
 ↓
Identify artifacts published
 ↓
Assess deployments
 ↓
Rotate related credentials
```

---

## 92. Scenario — Wrong Artifact Published

Response:

```text
Stop promotion
 ↓
Identify build
 ↓
Identify source commit
 ↓
Quarantine incorrect artifact if policy requires
 ↓
Build corrected version
 ↓
Scan
 ↓
Publish new immutable version
```

Do not silently overwrite an approved release.

---

## 93. Scenario — Artifactory Is Temporarily Unavailable

CI behavior should be designed explicitly:

```text
dependency downloads fail
publication fails
```

Possible resilience mechanisms:

```text
remote caching
retry policy
pipeline failure
artifact replication
HA architecture
```

Do not let failed artifact publication be silently ignored.

---

# PART XXVIII — PRODUCTION ARCHITECTURE

## 94. Enterprise Architecture

```text
                         Git
                          |
                          v
                       Jenkins
                          |
       +------------------+------------------+
       |                  |                  |
     Build              Test               Scan
       |                  |                  |
       +------------------+------------------+
                          |
                          v
                     Artifactory
                   /      |       \
                  /       |        \
              Maven     Docker     Helm
                  \       |        /
                   \      |       /
                    +-----+------+
                          |
                       Build Info
                          |
                      Promotion
                          |
                          v
                       EKS
                          |
                       Deploy
```

---

## 95. Security Boundaries

```text
Developer
   |
   v
Git
   |
   v
Jenkins
   |
   | scoped credentials
   v
Artifactory
   |
   | approved artifact
   v
Production
```

---

## 96. Network Security

Prefer:

```text
private connectivity
TLS
restricted security groups
firewall rules
network segmentation
```

---

# PART XXIX — PRODUCTION BEST PRACTICES

## 97. Pipeline

```text
[ ] version controlled
[ ] reproducible
[ ] automated tests
[ ] security gates
[ ] Build Info
[ ] promotion
```

---

## 98. Credentials

```text
[ ] dedicated service identity
[ ] least privilege
[ ] secret store
[ ] rotation
[ ] masking
[ ] no hardcoded secrets
```

---

## 99. Artifacts

```text
[ ] immutable releases
[ ] versioned
[ ] checksums/digests
[ ] retention
[ ] rollback
```

---

## 100. Jenkins

```text
[ ] controller secured
[ ] agents ephemeral where possible
[ ] RBAC
[ ] audit
[ ] plugin updates
[ ] backups
```

---

# PART XXX — INTERVIEW PREPARATION

## 101. How Do You Integrate Jenkins with Artifactory?

Answer:

```text
I configure Jenkins to authenticate to Artifactory using a dedicated
service identity and scoped credentials. The pipeline resolves
dependencies from virtual repositories, builds and tests the
application, publishes versioned artifacts to local repositories,
publishes Build Info and promotes the approved artifact through the
release process.
```

---

## 102. How Do You Secure Jenkins-Artifactory Integration?

Answer:

```text
I use least-privilege service identities, scoped access tokens,
Jenkins Credentials Store, TLS, secret masking, credential rotation
and repository-specific permissions. Jenkins should never use an
Artifactory administrator account for normal builds.
```

---

## 103. How Do You Publish Docker Images?

Answer:

```text
The pipeline authenticates to the Artifactory Docker registry, builds
the image, scans it, tags it with an immutable release version and
pushes it to the approved repository. I also capture the resulting
digest for production traceability.
```

---

## 104. How Do You Implement Build Once and Promote?

Answer:

```text
The CI pipeline builds and tests the artifact once, publishes it to
Artifactory and records Build Info. Promotion then moves the same
artifact through staging and production instead of rebuilding it.
```

---

## 105. Jenkins Gets 403 From Artifactory. What Do You Check?

Answer:

```text
I verify the authenticated service identity, group membership,
permission target, target repository and whether the operation is
READ or DEPLOY. I also verify that the pipeline is using the intended
credential.
```

---

## 106. Jenkins Gets 401. What Do You Check?

Answer:

```text
I check the credential, token validity and expiration, Artifactory
URL, credential binding and whether the token belongs to the expected
service identity.
```

---

## 107. Why Use Artifactory Instead of Jenkins Artifacts?

Answer:

```text
Artifactory provides enterprise artifact management, repository
organization, package-specific support, remote caching, virtual
repositories, access control, lifecycle management, promotion and
better long-term artifact availability.
```

---

## 108. How Do You Handle Credentials in Jenkins?

Answer:

```text
I store them in Jenkins Credentials or an approved external secret
manager, bind them only to the required pipeline scope, mask logs and
rotate them regularly.
```

---

## 109. How Do You Troubleshoot Docker Push?

Answer:

```text
I check registry DNS and network connectivity, TLS, docker login,
credential validity, repository path, DEPLOY permission and whether
the tag/version is allowed by repository policy.
```

---

## 110. How Do You Handle a Compromised Jenkins Agent?

Answer:

```text
I terminate the agent, revoke exposed credentials, review Jenkins and
Artifactory audit activity, identify artifacts published by the
agent, assess affected deployments and rotate related credentials.
```

---

# PART XXXI — PRODUCTION CHECKLIST

## 111. Connectivity

```text
[ ] DNS
[ ] routing
[ ] firewall
[ ] proxy
[ ] TLS
[ ] private connectivity
```

---

## 112. Authentication

```text
[ ] service identity
[ ] scoped token
[ ] Jenkins credentials
[ ] rotation
[ ] expiration
```

---

## 113. Authorization

```text
[ ] READ
[ ] DEPLOY
[ ] DELETE restricted
[ ] ADMIN restricted
[ ] repository boundaries
```

---

## 114. Pipeline

```text
[ ] checkout
[ ] build
[ ] test
[ ] scan
[ ] publish
[ ] Build Info
[ ] promotion
[ ] deployment
```

---

## 115. Security

```text
[ ] secrets masked
[ ] no hardcoded credentials
[ ] TLS
[ ] least privilege
[ ] audit
[ ] agent isolation
```

---

## 116. Reliability

```text
[ ] retry strategy
[ ] Artifactory availability
[ ] rollback
[ ] artifact retention
[ ] backup/DR
```

---

# PART XXXII — GOLDEN RULES

## 117. Rules

```text
1. Jenkins orchestrates the pipeline; Artifactory manages artifacts.

2. Use a dedicated least-privilege service identity for Jenkins.

3. Never use an Artifactory administrator account for normal CI.

4. Store credentials in Jenkins Credentials or an approved secret
   manager.

5. Never hardcode Artifactory credentials in Git.

6. Never print credentials in pipeline logs.

7. Use TLS for Jenkins-Artifactory communication.

8. Resolve dependencies through controlled virtual repositories.

9. Publish artifacts to explicitly approved local repositories.

10. Use immutable release versions.

11. Track Docker digests.

12. Publish Build Info.

13. Associate artifacts with Git commits and CI builds.

14. Build once and promote the same artifact.

15. Do not rebuild separately for each environment.

16. Restrict DELETE and ADMIN permissions.

17. Use separate identities when different teams or trust boundaries
    require different permissions.

18. Rotate service credentials.

19. Design pipelines to fail clearly when Artifactory is unavailable.

20. Do not silently continue when artifact publication fails.

21. Use ephemeral Kubernetes agents where practical.

22. Protect Jenkins itself with RBAC, secure plugins and controlled
    administration.

23. Audit artifact publication, promotion and deletion.

24. Verify the live production digest rather than trusting only an
    image tag.

25. Test rollback with retained artifacts.

26. Treat compromised Jenkins agents as credential and supply-chain
    incidents.

27. Keep repository and credential configuration outside application
    source code where practical.

28. Standardize Artifactory integration through Jenkins shared
    libraries for large organizations.

29. Validate exact JFrog plugin, CLI, Jenkins and Artifactory behavior
    against the deployed versions before production rollout.
```

---