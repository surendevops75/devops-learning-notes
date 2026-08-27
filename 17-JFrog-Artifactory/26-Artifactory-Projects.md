# 17-JFrog-Artifactory
# 26-Artifactory-Projects

## 1. Purpose

This file provides a production-oriented guide to JFrog Artifactory
Projects and how to use them for enterprise-scale organization,
multi-team governance, repository isolation, permissions, quotas,
security, ownership, and operational control.

The objective is to move from a simple repository model:

```text
One Artifactory
   |
   +--> Many Repositories
```

to a structured enterprise model:

```text
Artifactory Platform
        |
        +--> Project: Payments
        |      |
        |      +--> Repositories
        |      +--> Teams
        |      +--> Permissions
        |      +--> Builds
        |
        +--> Project: Orders
        |      |
        |      +--> Repositories
        |      +--> Teams
        |      +--> Permissions
        |
        +--> Shared Platform
               |
               +--> Common Repositories
               +--> Security
               +--> Administration
```

This file covers:

- Artifactory Projects fundamentals
- project architecture
- project administrators
- project members
- project roles
- repository association
- project repositories
- permission design
- teams
- service identities
- CI/CD
- Jenkins
- GitHub Actions
- GitLab
- Kubernetes
- EKS
- Docker
- Maven
- NPM
- PyPI
- Helm
- project isolation
- shared repositories
- project security
- quotas and governance
- naming conventions
- environment strategy
- Build Info
- release management
- project onboarding
- project offboarding
- production architecture
- troubleshooting
- migration
- operational governance
- interview preparation
- production checklists

---

# PART I — PROJECT FUNDAMENTALS

## 2. What Is an Artifactory Project?

A Project is an organizational and governance boundary inside
Artifactory.

It helps structure resources around:

```text
team
application
business unit
product
organization
```

The exact Project capabilities depend on the Artifactory/JFrog
Platform edition and version.

---

## 3. Why Projects?

Without Projects:

```text
Artifactory
 |
 +--> repo-a
 +--> repo-b
 +--> repo-c
 +--> repo-d
 +--> repo-e
```

As the organization grows, ownership becomes difficult to manage.

With Projects:

```text
Artifactory
 |
 +--> Payments
 |     +--> repos
 |     +--> users
 |     +--> teams
 |
 +--> Orders
 |     +--> repos
 |     +--> users
 |     +--> teams
```

---

## 4. Projects vs Repositories

Repository answers:

```text
Where are artifacts stored?
```

Project answers:

```text
Who owns and governs a group of resources?
```

---

## 5. Project Example

```text
Project: payments

Repositories:
payment-maven-local
payment-docker-local
payment-helm-local

Teams:
payments-developers
payments-release
payments-platform
```

---

# PART II — PROJECT ARCHITECTURE

## 6. Enterprise Project Model

```text
JFrog Platform
      |
      +--> Platform Administration
      |
      +--> Security Governance
      |
      +--> Payments Project
      |       |
      |       +--> repositories
      |       +--> teams
      |       +--> builds
      |
      +--> Orders Project
      |       |
      |       +--> repositories
      |       +--> teams
      |
      +--> Shared Resources
```

---

## 7. Project Boundary

A project should represent a meaningful ownership boundary.

Good examples:

```text
Payments
Orders
Customer-Platform
Mobile
Data-Platform
```

Avoid creating projects for every tiny experiment unless governance
requires it.

---

# PART III — PROJECT OWNERSHIP

## 8. Ownership

Every production project should have an accountable owner.

Example:

```text
Project:
payments

Business Owner:
Payments Engineering

Platform Owner:
DevOps Platform

Security Owner:
Security Engineering
```

---

## 9. Ownership Matters

Without ownership:

```text
permission issue
   |
   v
Who fixes it?
```

With ownership:

```text
Payments Platform Team
        |
        v
Responsible for project
```

---

# PART IV — PROJECT MEMBERS

## 10. Members

Project members may include:

```text
developers
release engineers
platform engineers
security engineers
service identities
```

---

## 11. Group-Based Membership

Prefer groups:

```text
payments-developers
payments-release
payments-admins
```

instead of managing dozens of users individually.

---

## 12. Lifecycle

When a developer joins:

```text
Identity Provider
       |
       v
payments-developers
       |
       v
Project access
```

When the developer leaves:

```text
remove identity
 ↓
group membership removed
 ↓
project access removed
```

---

# PART V — PROJECT ROLES

## 13. Role Concept

Use roles to separate:

```text
administration
development
release
read-only
security
```

Exact built-in roles and permissions depend on the JFrog Platform
version and configuration.

---

## 14. Least Privilege

Developer:

```text
READ
```

CI:

```text
READ + DEPLOY
```

Release:

```text
READ + controlled promotion
```

Administrator:

```text
project administration
```

---

# PART VI — REPOSITORY ASSOCIATION

## 15. Project Repositories

A project can organize repositories relevant to that team or product.

Example:

```text
Payments
 |
 +--> payment-docker-local
 +--> payment-maven-local
 +--> payment-helm-local
```

---

## 16. Repository Ownership

Clearly define:

```text
repository owner
project owner
platform owner
security owner
```

---

## 17. Shared Repository

Some repositories are shared:

```text
shared-maven-virtual
shared-docker-remote
```

Do not duplicate the same dependency repository for every project
unless isolation requires it.

---

# PART VII — SHARED VS PROJECT RESOURCES

## 18. Shared Resources

Examples:

```text
approved Maven remote
approved NPM remote
approved PyPI remote
base container images
security scanning
```

---

## 19. Project Resources

Examples:

```text
payment application images
payment Helm charts
payment release artifacts
```

---

## 20. Design Principle

Centralize common controls:

```text
Security
Approved Dependencies
Platform Standards
```

while decentralizing:

```text
Application Artifacts
Team Ownership
Release Workflows
```

---

# PART VIII — PROJECT SECURITY

## 21. Security Boundary

A project should not automatically give access to every repository in
Artifactory.

---

## 22. Example

Payments team:

```text
READ:
shared-docker-virtual

DEPLOY:
payment-docker-local
```

Should not automatically have:

```text
DEPLOY:
orders-docker-local
```

---

## 23. Cross-Project Access

Sometimes one project needs another project's artifact.

Example:

```text
Payments
   |
   v
shared-platform-library
```

Use controlled read access rather than broad platform permissions.

---

# PART IX — CI/CD PROJECT MODEL

## 24. CI Identity

Create project-specific service identities.

Example:

```text
svc-payments-jenkins
svc-payments-github
svc-payments-gitlab
```

---

## 25. CI Permissions

Example:

```text
READ:
shared dependencies

DEPLOY:
payments-local

DELETE:
NO

ADMIN:
NO
```

---

## 26. Why Project-Specific CI Identities?

If one credential is compromised:

```text
blast radius = one project
```

rather than:

```text
blast radius = entire Artifactory
```

---

# PART X — JENKINS PROJECT

## 27. Jenkins Flow

```text
Payments Git
     |
     v
Jenkins
     |
     +--> Build
     +--> Test
     +--> Scan
     |
     v
Payments Artifactory Project
```

---

## 28. Jenkins Credential

Use:

```text
Jenkins Credentials
```

with a scoped service identity.

Never:

```text
admin credentials
```

for routine builds.

---

# PART XI — GITHUB ACTIONS PROJECT

## 29. GitHub Actions

```text
GitHub
  |
  v
Actions
  |
  v
Payments Project
  |
  v
Artifactory
```

---

## 30. Secret Protection

Use:

```text
repository secrets
environment secrets
protected environments
OIDC/federated identity where supported
```

---

# PART XII — GITLAB PROJECT

## 31. GitLab

```text
GitLab
  |
  v
Runner
  |
  v
Project Service Identity
  |
  v
Artifactory Project
```

Use:

```text
masked variables
protected variables
protected branches
protected environments
```

---

# PART XIII — DOCKER PROJECT

## 32. Docker

Example:

```text
Payments
 |
 +--> payment-docker-dev-local
 +--> payment-docker-stage-local
 +--> payment-docker-prod-local
```

---

## 33. Image Promotion

Preferred:

```text
Build
 ↓
Scan
 ↓
Candidate
 ↓
Promote
 ↓
Production
```

---

## 34. Digest Tracking

Production should record:

```text
image digest
```

rather than relying only on:

```text
latest
```

---

# PART XIV — MAVEN PROJECT

## 35. Maven

Example:

```text
Payments
 |
 +--> payment-maven-release-local
 +--> payment-maven-snapshot-local
```

---

## 36. Dependency Access

```text
Payments
 |
v
maven-virtual
 |
+--> internal
+--> approved external
```

---

# PART XV — NPM PROJECT

## 37. NPM

Example:

```text
Payments
 |
 +--> payment-npm-local
```

Use project-specific package publishing permissions.

---

# PART XVI — PYPI PROJECT

## 38. PyPI

Example:

```text
Payments
 |
 +--> payment-pypi-local
```

Control:

```text
READ
DEPLOY
DELETE
```

---

# PART XVII — HELM PROJECT

## 39. Helm

Example:

```text
Payments
 |
 +--> payment-helm-local
```

Flow:

```text
CI
 ↓
Helm package
 ↓
Artifactory
 ↓
GitOps
 ↓
EKS
```

---

# PART XVIII — KUBERNETES PROJECT

## 40. Runtime Model

Kubernetes should generally receive:

```text
READ
```

access to the required project repository.

---

## 41. EKS

```text
EKS
 |
v
Registry Authentication
 |
v
Payments Docker Repository
 |
v
Image
```

---

## 42. Runtime Separation

CI:

```text
DEPLOY
```

Runtime:

```text
READ
```

This reduces credential blast radius.

---

# PART XIX — PROJECT RBAC

## 43. Example RBAC

```text
payments-developers
    |
    +--> READ artifacts
    +--> READ dependencies

payments-ci
    |
    +--> READ
    +--> DEPLOY

payments-release
    |
    +--> READ
    +--> promotion operations

payments-admins
    |
    +--> project administration
```

---

## 44. Do Not Over-Permission

Bad:

```text
payments-developers
        |
        v
Artifactory Admin
```

Good:

```text
payments-developers
        |
        v
Project-scoped access
```

---

# PART XX — PROJECT NAMING

## 45. Naming Convention

Use:

```text
<business-domain>
```

Examples:

```text
payments
orders
customer-platform
data-platform
```

---

## 46. Repository Naming

Use:

```text
<project>-<ecosystem>-<purpose>-<environment>
```

Example:

```text
payments-docker-prod-local
```

---

# PART XXI — ENVIRONMENT STRATEGY

## 47. Environment Model

```text
DEV
 |
STAGE
 |
PROD
```

---

## 48. Repository Model

```text
payments-docker-dev-local
payments-docker-stage-local
payments-docker-prod-local
```

---

## 49. Promotion

Avoid rebuilding:

```text
DEV artifact
 ↓
rebuild
 ↓
PROD artifact
```

Prefer:

```text
same artifact
 ↓
promotion
```

---

# PART XXII — PROJECT BUILD INFO

## 50. Build Info

Build Info can connect:

```text
Project
 |
v
CI Build
 |
v
Git Commit
 |
v
Dependencies
 |
v
Artifact
```

---

## 51. Incident Investigation

Example:

```text
Production Image
 |
v
Digest
 |
v
Artifactory Project
 |
v
Build Info
 |
v
CI Build
 |
v
Git Commit
```

This gives strong traceability.

---

# PART XXIII — PROJECT GOVERNANCE

## 52. Governance

Define:

```text
naming
ownership
retention
security
permissions
repository standards
release policy
```

---

## 53. Standard Template

Every new project should have:

```text
owner
repositories
groups
service identities
RBAC
retention
security policy
monitoring
backup requirements
```

---

# PART XXIV — PROJECT ONBOARDING

## 54. New Project

Example:

```text
New Product: Payments
```

Process:

```text
1. Request
2. Business owner
3. Platform review
4. Security review
5. Create Project
6. Create groups
7. Create repositories
8. Configure permissions
9. Configure CI
10. Configure monitoring
11. Validate
12. Handover
```

---

## 55. Onboarding Checklist

```text
[ ] project name
[ ] owner
[ ] teams
[ ] repositories
[ ] CI identity
[ ] runtime identity
[ ] RBAC
[ ] security
[ ] retention
[ ] monitoring
```

---

# PART XXV — PROJECT OFFBOARDING

## 56. Offboarding

When an application is retired:

```text
1. Confirm retirement
2. Freeze publishing
3. Identify consumers
4. Archive artifacts if required
5. Disable CI
6. Remove credentials
7. Remove unused permissions
8. Apply retention policy
9. Document closure
```

---

# PART XXVI — PROJECT SECURITY

## 57. Project Security Controls

Use:

```text
least privilege
MFA for privileged humans
scoped tokens
TLS
private networking
artifact scanning
audit
immutability
```

---

## 58. Credential Rotation

Project service credentials should be rotated without requiring
platform-wide changes.

---

# PART XXVII — PROJECT QUOTAS AND GOVERNANCE

## 59. Resource Governance

Where the deployed JFrog platform supports project-level quotas or
limits, use them to prevent one project from consuming uncontrolled
platform resources.

---

## 60. Why?

Without governance:

```text
Project A
   |
   v
huge artifact growth
   |
   v
storage pressure
   |
   v
all projects affected
```

---

## 61. Monitoring Growth

Track:

```text
artifact count
storage
downloads
uploads
builds
```

---

# PART XXVIII — MULTI-TEAM ARCHITECTURE

## 62. Example

```text
JFrog
 |
 +--> Payments
 |
 +--> Orders
 |
 +--> Customer
 |
 +--> Data
 |
 +--> Shared Platform
```

---

## 63. Shared Platform

Owns:

```text
approved dependencies
base images
common tooling
platform libraries
```

---

## 64. Application Projects

Own:

```text
application artifacts
release lifecycle
CI/CD
application-specific repositories
```

---

# PART XXIX — MULTI-BUSINESS-UNIT ARCHITECTURE

## 65. Enterprise

```text
Enterprise Artifactory
 |
 +--> Banking
 |
 +--> Payments
 |
 +--> Commerce
 |
 +--> Analytics
```

Use projects when the organizational boundary matches the required
governance model.

---

# PART XXX — PROJECT AND REPOSITORY ISOLATION

## 66. Isolation Levels

Consider:

```text
logical isolation
permission isolation
network isolation
account isolation
platform isolation
```

---

## 67. When One Artifactory Is Enough

One platform can work when:

```text
shared governance
compatible security requirements
manageable scale
```

exist.

---

## 68. When Separate Platforms May Be Better

Consider separate Artifactory environments when there are requirements
such as:

```text
strict regulatory separation
different trust boundaries
independent availability requirements
different administrative ownership
```

---

# PART XXXI — PROJECT + SECURITY SCANNING

## 69. Project Security Pipeline

```text
Build
 |
v
Artifactory
 |
v
Scan
 |
+--> PASS
|      |
|      v
|   Promotion
|
+--> FAIL
       |
       v
     Block
```

---

## 70. Policy

Define:

```text
critical vulnerability
high vulnerability
license violation
malware
```

thresholds according to organizational policy.

---

# PART XXXII — PROJECT + RELEASE MANAGEMENT

## 71. Release Flow

```text
Developer
 |
v
CI
 |
v
Build
 |
v
Project Repository
 |
v
Security
 |
v
Approval
 |
v
Production
```

---

## 72. Release Identity

Track:

```text
project
artifact
version
digest
Build Info
Git commit
release
```

---

# PART XXXIII — PROJECT + GITOPS

## 73. GitOps

```text
Project CI
 |
v
Artifactory
 |
v
GitOps Repository
 |
v
Argo CD
 |
v
EKS
```

---

## 74. Separation of Responsibilities

```text
Artifactory:
artifact storage

Git:
desired configuration

Argo CD:
deployment reconciliation

Kubernetes:
runtime
```

---

# PART XXXIV — PROJECT + OBSERVABILITY

## 75. Project Metrics

Monitor:

```text
artifact growth
downloads
uploads
build failures
security failures
```

---

## 76. Project Logs

Correlate:

```text
project
repository
user
CI
artifact
```

---

# PART XXXV — PROJECT + INCIDENT RESPONSE

## 77. Project-Specific Incident

Example:

```text
Payments project
   |
   v
malicious image detected
```

Response:

```text
block promotion
 ↓
identify image
 ↓
identify project consumers
 ↓
audit
 ↓
rotate credentials
 ↓
replace artifact
```

---

# PART XXXVI — TROUBLESHOOTING PROJECTS

## 78. Project Not Visible

Check:

```text
user membership
group membership
project role
platform permissions
```

---

## 79. Repository Not Accessible

Check:

```text
project association
permission target
user/group
repository path
```

---

## 80. CI Cannot Publish

Check:

```text
service identity
project membership
DEPLOY permission
repository
token
```

---

## 81. Cross-Project Access Failure

Check:

```text
source project
target repository
READ permission
permission target
identity
```

---

# PART XXXVII — PRODUCTION ARCHITECTURE

## 82. Enterprise Project Architecture

```text
                       JFrog Platform
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
        Payments          Orders          Customer
             |                |                |
        +----+----+      +----+----+      +----+----+
        |         |      |         |      |         |
      Docker    Maven  Docker    Helm   Docker    NPM
        |         |      |         |      |         |
        +---------+------+---------+------+---------+
                          |
                    Shared Services
                          |
              +-----------+-----------+
              |                       |
              v                       v
           Security              Monitoring
              |                       |
              +-----------+-----------+
                          |
                       Backup/DR
```

---

# PART XXXVIII — PRODUCTION PROJECT DESIGN

## 83. Recommended Pattern

For a large enterprise:

```text
Platform
 |
 +--> Shared Repositories
 |
 +--> Project: Payments
 |      |
 |      +--> App repositories
 |      +--> CI identities
 |      +--> teams
 |
 +--> Project: Orders
 |      |
 |      +--> App repositories
 |      +--> CI identities
 |      +--> teams
```

---

## 84. Blast Radius

Project isolation should reduce:

```text
credential blast radius
permission blast radius
operational blast radius
```

---

# PART XXXIX — PROJECT LIFECYCLE

## 85. Lifecycle

```text
Request
 ↓
Design
 ↓
Create
 ↓
Onboard
 ↓
Operate
 ↓
Scale
 ↓
Audit
 ↓
Retire
```

---

## 86. Periodic Review

Review:

```text
members
groups
service identities
tokens
repositories
permissions
storage
security findings
```

---

# PART XL — PROJECT AUDIT

## 87. Access Review

Periodically verify:

```text
Who has access?
Why?
What can they do?
Is the access still required?
```

---

## 88. Service Identity Review

Check:

```text
active tokens
unused identities
permissions
last usage
owner
```

---

# PART XLI — PROJECT MIGRATION

## 89. Existing Repositories

Migration process:

```text
Inventory
 ↓
Map ownership
 ↓
Create Project
 ↓
Associate repositories
 ↓
Create groups
 ↓
Apply permissions
 ↓
Test
 ↓
Migrate CI
 ↓
Validate
```

---

## 90. Avoid Big-Bang Migration

Prefer:

```text
pilot project
 ↓
validate
 ↓
repeat
```

---

# PART XLII — PROJECT PERFORMANCE

## 91. Project Growth

One project can generate:

```text
millions of artifacts
high CI traffic
large container images
```

Monitor resource usage.

---

## 92. Repository Design

Avoid unnecessary duplication:

```text
100 copies of same remote dependency
```

Prefer controlled shared repositories when governance permits.

---

# PART XLIII — PROJECT SECURITY INCIDENT

## 93. Compromised Project Credential

If:

```text
svc-payments-ci
```

is compromised:

```text
revoke token
 ↓
audit project
 ↓
inspect uploads
 ↓
inspect downloads
 ↓
check deployments
 ↓
rotate credential
```

Project scoping limits blast radius.

---

# PART XLIV — INTERVIEW PREPARATION

## 94. What Are Artifactory Projects?

Answer:

```text
Projects provide an organizational and governance boundary for
grouping repositories, teams, permissions and related resources around
a product, application or business unit. They help manage large
Artifactory platforms without giving every team broad platform-level
access.
```

---

## 95. Why Use Projects?

Answer:

```text
I use Projects to establish clear ownership, isolate team access,
scope service identities, organize repositories and simplify
enterprise governance. They are especially useful when many teams
share the same Artifactory platform.
```

---

## 96. Projects vs Repositories?

Answer:

```text
A repository is primarily an artifact storage and resolution boundary.
A Project is an organizational and governance boundary that can group
resources, users and permissions around a team or product.
```

---

## 97. How Do You Secure Projects?

Answer:

```text
I use group-based membership, least-privilege roles, project-scoped
service identities, repository-specific permissions, protected
credentials, artifact scanning, audit logging and periodic access
reviews.
```

---

## 98. How Do You Design Multi-Team Artifactory?

Answer:

```text
I create logical Projects for major ownership boundaries, maintain
shared repositories for approved common dependencies, use project
specific repositories for application artifacts, separate CI and
runtime identities and enforce least-privilege cross-project access.
```

---

## 99. How Do You Prevent One Team From Accessing Another Team's Images?

Answer:

```text
I use project and repository-scoped permissions. The consuming team
gets only the required READ access, while publishing and deletion
permissions remain restricted to the owning project and controlled
release identities.
```

---

## 100. How Do You Handle Cross-Project Dependencies?

Answer:

```text
I identify the producer repository and grant the consumer's service
identity only the required READ access. I avoid granting broad
platform permissions and document the dependency relationship.
```

---

## 101. How Do You Onboard a New Project?

Answer:

```text
I first establish ownership and requirements, then create the
Project, teams, repositories, service identities, permission model,
security controls and monitoring. Finally I integrate CI/CD, test
artifact publication and consumption, and document ownership.
```

---

## 102. What Happens When a Project Is Retired?

Answer:

```text
I freeze publishing, identify downstream consumers, archive required
artifacts, disable CI credentials, remove unused permissions and
service identities, apply retention policy and document the retirement.
```

---

# PART XLV — PRODUCTION CHECKLIST

## 103. Ownership

```text
[ ] project owner
[ ] platform owner
[ ] security owner
[ ] escalation contact
```

---

## 104. Access

```text
[ ] groups
[ ] roles
[ ] service identities
[ ] least privilege
[ ] token rotation
[ ] access review
```

---

## 105. Repositories

```text
[ ] local
[ ] remote
[ ] virtual
[ ] naming
[ ] environment separation
[ ] ownership
```

---

## 106. CI/CD

```text
[ ] Jenkins
[ ] GitHub Actions
[ ] GitLab
[ ] service identity
[ ] secrets
[ ] Build Info
```

---

## 107. Kubernetes

```text
[ ] registry access
[ ] READ permission
[ ] imagePullSecret/workload identity
[ ] digest tracking
[ ] EKS connectivity
```

---

## 108. Security

```text
[ ] scanning
[ ] TLS
[ ] RBAC
[ ] audit
[ ] immutable releases
[ ] incident response
```

---

## 109. Operations

```text
[ ] monitoring
[ ] storage growth
[ ] backup
[ ] DR
[ ] lifecycle
[ ] periodic review
```

---

# PART XLVI — GOLDEN RULES

## 110. Rules

```text
1. Use Projects for meaningful organizational boundaries.

2. Do not create projects merely because repositories exist.

3. Give every production project a clear owner.

4. Prefer group-based membership.

5. Use least-privilege roles.

6. Separate project administration from application development.

7. Use project-specific CI identities.

8. Keep runtime identities read-only where possible.

9. Restrict DELETE.

10. Use shared repositories for approved common dependencies where
   governance allows.

11. Keep application artifacts scoped to owning projects.

12. Grant cross-project access explicitly.

13. Avoid broad platform-wide credentials.

14. Use predictable project and repository names.

15. Separate environments where required.

16. Build once and promote the same artifact.

17. Track Build Info and artifact digests.

18. Protect project credentials.

19. Review project membership periodically.

20. Rotate service credentials.

21. Monitor project storage and artifact growth.

22. Apply security scanning and policy controls.

23. Document project ownership and dependencies.

24. Use a standard onboarding process.

25. Use a controlled offboarding process.

26. Reduce blast radius through project isolation.

27. Do not confuse project boundaries with complete security isolation.

28. Use separate Artifactory platforms when regulatory or trust
    boundaries genuinely require stronger isolation.

29. Test CI and Kubernetes access after permission changes.

30. Validate the exact JFrog Platform edition and version before
    relying on specific Project features or role names.
```

---

# END OF 26-Artifactory-Projects.md
