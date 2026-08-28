# RBAC-and-Permissions

## 1. Purpose

This file covers Role-Based Access Control (RBAC) and permission management in JFrog Artifactory, from fundamentals through enterprise production environments.

It covers:

- RBAC fundamentals
- Authentication vs authorization
- Users and groups
- Service identities
- Projects
- Permission targets
- Repository permissions
- Read, deploy/cache, annotate, delete and manage concepts
- Project roles
- Administrative permissions
- Least privilege
- Repository isolation
- CI/CD permissions
- Jenkins
- GitHub Actions
- GitLab CI
- Docker
- Maven
- NPM
- PyPI
- Helm
- Kubernetes
- Argo CD/GitOps
- Production access models
- Permission inheritance and evaluation concepts
- Access reviews
- Permission troubleshooting
- Audit
- Incident response
- Real-world scenarios
- Interview preparation

---

# PART I — RBAC FUNDAMENTALS

## 2. What Is RBAC?

RBAC means Role-Based Access Control.

Instead of assigning permissions independently to every user:

```text
User
 ↓
Role / Group
 ↓
Permissions
 ↓
Resources
```

The role determines what actions the identity can perform.

---

## 3. Why RBAC Matters in Artifactory

Artifactory contains valuable software supply-chain assets:

```text
Docker images
Maven packages
NPM packages
Python packages
Helm charts
generic artifacts
build information
repositories
projects
```

Poor permission design can allow:

```text
unauthorized downloads
malicious uploads
artifact deletion
production modification
credential abuse
```

---

## 4. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

RBAC primarily addresses authorization.

---

## 5. Example

A Jenkins identity may authenticate successfully:

```text
Authenticated = YES
```

but:

```text
Deploy to production repository = NO
```

because its permissions do not allow deployment.

Result:

```text
403 Forbidden
```

---

# PART II — CORE ACCESS OBJECTS

## 6. Users

A user represents an identity.

Examples:

```text
developer01
release-engineer01
platform-admin01
```

---

## 7. Groups

Groups combine users with similar access requirements.

Examples:

```text
developers
release-engineers
security-team
platform-admins
```

---

## 8. Why Groups Are Better Than Individual Assignments

Instead of:

```text
Alice → READ
Bob → READ
Charlie → READ
David → READ
```

use:

```text
developers
   |
   +-- Alice
   +-- Bob
   +-- Charlie
   +-- David

developers → READ
```

Benefits:

```text
central management
consistent access
simpler onboarding
simpler offboarding
less permission drift
```

---

## 9. Service Identities

Automation should use dedicated identities.

Examples:

```text
svc-payment-build
svc-platform-release
svc-argocd-prod
```

Avoid:

```text
Jenkins → admin-user
```

---

## 10. Projects

Artifactory Projects can provide organizational boundaries for repositories, users, groups and permissions depending on the platform configuration and edition.

Conceptually:

```text
Project
 ├── repositories
 ├── teams
 ├── roles
 └── permissions
```

---

## 11. Permission Target

A permission target defines access to selected Artifactory resources.

Conceptually:

```text
Identity / Group
       ↓
Permission Target
       ↓
Repository / Resource
       ↓
Actions
```

---

## 12. Repository Permissions

Permissions can be scoped to repositories and paths according to the configured Artifactory permission model.

Example:

```text
developers
 ↓
READ
 ↓
npm-virtual
```

---

# PART III — COMMON ARTIFACTORY ACTIONS

## 13. Read

Read generally allows downloading or retrieving artifacts.

Examples:

```text
docker pull
mvn dependency download
npm install
pip install
helm pull
```

---

## 14. Deploy / Cache

Deployment permissions are used for publishing artifacts.

Examples:

```text
mvn deploy
npm publish
twine upload
docker push
helm push
```

Remote repositories may also involve caching behavior controlled by repository configuration and permissions.

---

## 15. Delete

Delete allows removal of artifacts or repository content where the identity is authorized.

This is highly privileged.

Do not grant it to ordinary build consumers.

---

## 16. Annotate

Annotation-related permissions can control metadata operations depending on the Artifactory configuration.

Treat metadata modification as a controlled permission.

---

## 17. Manage

Management permissions are broader than normal artifact consumption.

They should be restricted to trusted platform or repository administrators.

---

## 18. Administrative Access

Administrative capabilities can affect:

```text
repositories
users
groups
projects
permissions
system configuration
```

Keep administrative access extremely limited.

---

# PART IV — LEAST PRIVILEGE

## 19. Principle

Give an identity:

```text
only the permissions it needs
```

Nothing more.

---

## 20. Bad CI Design

```text
Jenkins
 ↓
Artifactory Admin
```

This creates a large blast radius.

---

## 21. Better CI Design

```text
Jenkins
 ↓
svc-payment-build
 ↓
READ:
  maven-virtual
  npm-virtual

DEPLOY:
  payment-maven-local
```

---

## 22. Build vs Release Permissions

Separate:

```text
Build Identity
→ READ dependencies

Release Identity
→ READ dependencies
→ DEPLOY approved artifact
```

This reduces the privileges available to normal builds.

---

## 23. Delete Permission

Do not give:

```text
DELETE
```

to standard application build pipelines unless required.

A compromised build identity should not be able to destroy production artifacts.

---

## 24. Admin Permission

Do not give:

```text
ADMIN
```

to:

```text
Jenkins
GitHub Actions
GitLab Runner
Kubernetes Pods
application developers
```

unless there is an explicitly approved administrative requirement.

---

# PART V — ACCESS DESIGN

## 25. Developer Access

Typical:

```text
READ:
  development virtual repositories

DEPLOY:
  optional team-local development repository

DELETE:
  normally NO
```

---

## 26. CI Build Access

Typical:

```text
READ:
  required virtual repositories

DEPLOY:
  only if this pipeline publishes packages

DELETE:
  NO
```

---

## 27. Release Pipeline

Typical:

```text
READ:
  required repositories

DEPLOY:
  release repositories

DELETE:
  restricted
```

---

## 28. GitOps Controller

Typical:

```text
READ:
  Helm / OCI repository
```

The GitOps controller generally does not need:

```text
DELETE
ADMIN
```

---

## 29. Kubernetes Runtime

Typical:

```text
READ:
  Docker/OCI image repository
```

The application Pod should not have:

```text
DEPLOY
DELETE
ADMIN
```

---

# PART VI — REPOSITORY ISOLATION

## 30. Team Isolation

Example:

```text
Payment Team
 ↓
payment-maven-local
payment-docker-local

Orders Team
 ↓
orders-maven-local
orders-docker-local
```

Each service identity should access only what it requires.

---

## 31. Shared Virtual Repositories

A common pattern:

```text
Team CI
   |
   v
maven-virtual
 /       \
local    remote
```

The team gets:

```text
READ
```

to the virtual repository.

---

## 32. Publish to Local

Publishing should go to a specific local repository:

```text
payment CI
 ↓
payment-maven-local
```

not a broad set of unrelated repositories.

---

## 33. Production Separation

Production repositories should have stronger controls.

Example:

```text
developers
    ↓
READ production

release-service
    ↓
DEPLOY production

platform-admin
    ↓
administrative access
```

---

# PART VII — PROJECT-BASED ACCESS

## 34. Why Projects?

Large organizations may have:

```text
hundreds of repositories
hundreds of teams
many environments
```

Projects can help organize access around teams or business units.

---

## 35. Example

```text
Project: Payments

Repositories:
  payment-maven-local
  payment-docker-local
  payment-helm-local

Teams:
  payment-dev
  payment-release
```

---

## 36. Project Roles

Organizations can define role-oriented access such as:

```text
Project Viewer
Project Developer
Project Release
Project Administrator
```

The exact role names and capabilities depend on the Artifactory/JFrog Platform version and configuration.

---

## 37. Project Isolation

A project can help separate:

```text
payment
orders
customer
platform
```

and reduce accidental cross-team access.

---

# PART VIII — RBAC + CI/CD

## 38. Jenkins

Architecture:

```text
Jenkins
   |
   v
svc-payment-build
   |
   v
Permission Target
   |
   +---- READ maven-virtual
   +---- DEPLOY payment-maven-local
```

---

## 39. Jenkins Should Not Be Admin

Avoid:

```text
Jenkins
 ↓
Admin
```

because a compromised Jenkins controller, agent or pipeline could inherit Artifactory-wide privileges.

---

## 40. GitHub Actions

Use:

```text
GitHub Actions
 ↓
dedicated identity
 ↓
scoped Artifactory permissions
```

---

## 41. GitLab CI

Use:

```text
GitLab Runner
 ↓
service identity
 ↓
specific repositories
```

---

## 42. Pipeline Permission Separation

A mature model:

```text
PR Validation
 → READ

Build
 → READ

Publish Snapshot
 → READ + DEPLOY snapshot repository

Release
 → READ + DEPLOY release repository
```

---

# PART IX — PACKAGE-SPECIFIC RBAC

## 43. Maven

Example:

```text
maven-virtual
  → READ

payment-maven-local
  → DEPLOY
```

---

## 44. NPM

Example:

```text
npm-virtual
  → READ

company-npm-local
  → DEPLOY
```

---

## 45. PyPI

Example:

```text
pypi-virtual
  → READ

company-pypi-local
  → DEPLOY
```

---

## 46. Docker

Example:

```text
docker-virtual
  → READ

payment-docker-local
  → DEPLOY
```

---

## 47. Helm

Example:

```text
helm-virtual
  → READ

platform-helm-local
  → DEPLOY
```

---

# PART X — PATH-LEVEL CONTROL

## 48. Repository Path

Where supported, access can be restricted more narrowly than the entire repository.

Conceptually:

```text
repository
 ├── team-a/*
 ├── team-b/*
 └── shared/*
```

---

## 49. Why Path-Level Permissions?

Useful when one repository contains:

```text
multiple teams
multiple product lines
multiple namespaces
```

However, avoid unnecessarily complex permission structures.

---

## 50. Complexity Risk

Too many permission exceptions create:

```text
permission drift
audit difficulty
unexpected access
administrative overhead
```

Prefer clear repository boundaries when practical.

---

# PART XI — PERMISSION INHERITANCE AND EVALUATION

## 51. Access Evaluation

Conceptually:

```text
Request
 ↓
Identity
 ↓
Groups / Roles
 ↓
Project permissions
 ↓
Repository permissions
 ↓
Path restrictions
 ↓
Requested operation
 ↓
ALLOW / DENY
```

The exact evaluation behavior depends on the Artifactory version and configured permission model.

---

## 52. Direct vs Group Access

A user may receive permissions through:

```text
direct assignment
+
group membership
+
project role
```

Track where access originates.

---

## 53. Why Direct User Permissions Are Risky

Example:

```text
Alice → READ repo-A
Bob   → READ repo-A
Charlie → READ repo-A
```

Over time this becomes difficult to manage.

Prefer:

```text
developers → READ repo-A
```

---

## 54. Permission Drift

Permission drift occurs when access accumulates over time without cleanup.

Example:

```text
Developer changes team
 ↓
Old access remains
 ↓
New access added
 ↓
User now has excessive access
```

---

## 55. Preventing Permission Drift

Use:

```text
groups
role-based access
access reviews
automated onboarding/offboarding
time-bound access where appropriate
```

---

# PART XII — PRODUCTION ACCESS MODEL

## 56. Environment-Based Access

Example:

```text
DEV:
Developers + CI

STAGE:
CI + Release

PROD:
Release + Platform Admin
```

---

## 57. Production Publishing

Production artifact publication should normally be controlled.

Example:

```text
Developer
  ↓
NO direct production publish

Release Pipeline
  ↓
DEPLOY
```

---

## 58. Human Production Access

Prefer:

```text
SSO
+
MFA
+
approved group
+
least privilege
```

---

## 59. Break-Glass Access

Emergency access can be designed separately.

Conceptually:

```text
Normal access
    ↓
restricted

Emergency
    ↓
break-glass identity
    ↓
strong approval + audit
```

Break-glass credentials should be protected and monitored carefully.

---

# PART XIII — ACCESS REVIEWS

## 60. Why Access Reviews?

Review whether:

```text
users still need access
service accounts still exist
permissions are appropriate
old tokens remain active
former employees retain access
```

---

## 61. Review Frequency

Organizations should define an appropriate schedule such as:

```text
quarterly
semi-annually
after major organizational changes
```

The frequency should reflect risk and compliance requirements.

---

## 62. Review Questions

Ask:

```text
Who has access?
Why?
Which repositories?
Which operations?
Who approved it?
Is it still required?
```

---

## 63. Dormant Accounts

Identify:

```text
inactive users
unused service identities
expired integrations
old CI credentials
```

Remove or disable according to policy.

---

# PART XIV — AUDITING

## 64. Audit Events

Monitor:

```text
user creation
user deletion
group changes
permission changes
token creation
token revocation
artifact deployment
artifact deletion
administrative operations
```

---

## 65. Unexpected Deployment

If an unknown identity publishes:

```text
artifact
```

investigate:

```text
identity
source
token
pipeline
repository
time
IP/network information where available
```

---

## 66. Unexpected Delete

A deletion should be correlated with:

```text
user
service identity
pipeline
timestamp
repository
artifact
```

---

# PART XV — SECURITY PATTERNS

## 67. Deny Broad Access

Avoid:

```text
All users → All repositories
```

---

## 68. Grant Specific Access

Prefer:

```text
Payment developers
 → payment repositories

Orders developers
 → orders repositories
```

---

## 69. Read vs Write Separation

Use:

```text
Consumers → READ

Publishers → DEPLOY
```

---

## 70. Delete Separation

Use:

```text
Release operations → DEPLOY
Platform operations → DELETE when necessary
```

---

## 71. Admin Separation

Use:

```text
Platform admins → ADMIN
```

not:

```text
Application CI → ADMIN
```

---

# PART XVI — AUTHENTICATION + RBAC

## 72. Complete Flow

```text
Client
 ↓
Authentication
 ↓
Identity
 ↓
Groups / Roles
 ↓
Permission Target
 ↓
Repository
 ↓
Operation
 ↓
ALLOW / DENY
```

---

## 73. Example — Docker Push

```text
Jenkins
 ↓
svc-payment-build
 ↓
Authenticated
 ↓
Permission target
 ↓
docker-local/payment/*
 ↓
DEPLOY
 ↓
Allowed
```

---

## 74. Example — Docker Pull

```text
Kubernetes
 ↓
registry identity
 ↓
Authenticated
 ↓
Permission target
 ↓
docker-local/payment/*
 ↓
READ
 ↓
Allowed
```

---

## 75. Example — Unauthorized Delete

```text
Jenkins
 ↓
Authenticated
 ↓
Permission check
 ↓
DELETE not granted
 ↓
403
```

This is a desirable security outcome.

---

# PART XVII — TROUBLESHOOTING RBAC

## 76. 401 vs 403

```text
401
→ authentication issue

403
→ authorization issue
```

Always inspect the complete request and server response before final diagnosis.

---

## 77. User Can Login but Cannot Download

Check:

```text
repository READ permission
group membership
project role
permission target
repository path
```

---

## 78. User Can Download but Cannot Publish

This may be intentional.

Check:

```text
DEPLOY permission
target repository
project role
```

---

## 79. CI Can Read but Cannot Push

Check:

```text
service identity
DEPLOY permission
target repository
token scope
```

---

## 80. CI Can Push to Wrong Repository

This is a permission-boundary problem.

Restrict:

```text
DEPLOY
```

to only the intended repository/path.

---

## 81. User Has More Access Than Expected

Investigate:

```text
direct permissions
group membership
project roles
nested groups
old roles
permission targets
```

---

## 82. Access Removed but User Still Has Access

Check:

```text
group membership
active sessions
tokens
service credentials
other identities
```

Do not assume removing one access path removes all possible access.

---

## 83. Permission Change Not Working

Check:

```text
correct permission target
correct repository
correct identity
correct group
correct project
cached/client credentials
```

---

# PART XVIII — REAL-WORLD SCENARIOS

## 84. Scenario — Developer Requests Production Deploy Access

Recommended:

```text
Do not grant permanent direct production DEPLOY
```

Instead:

```text
Developer
 ↓
CI
 ↓
Release approval
 ↓
Release identity
 ↓
Production repository
```

---

## 85. Scenario — Jenkins Token Has Admin Access

Response:

```text
Identify current operations
 ↓
Create least-privilege service identity
 ↓
Grant required READ/DEPLOY
 ↓
Update Jenkins
 ↓
Test
 ↓
Revoke admin credential
 ↓
Audit
```

---

## 86. Scenario — Team Needs Access to New Repository

Process:

```text
Identify owner
 ↓
Define required operation
 ↓
Create/update group
 ↓
Add permission target
 ↓
Test
 ↓
Audit
```

---

## 87. Scenario — Employee Changes Team

Response:

```text
Remove old group membership
 ↓
Add new group membership
 ↓
Review direct permissions
 ↓
Review tokens
 ↓
Validate effective access
```

---

## 88. Scenario — Compromised Service Token

Response:

```text
Revoke token
 ↓
Disable identity if necessary
 ↓
Review audit
 ↓
Identify artifacts touched
 ↓
Check deployments
 ↓
Rotate related credentials
 ↓
Rebuild trusted artifacts
```

---

## 89. Scenario — Malicious Artifact Published

Response:

```text
Block/quarantine
 ↓
Identify publisher
 ↓
Revoke credentials
 ↓
Identify consumers
 ↓
Check production
 ↓
Publish trusted artifact
 ↓
Redeploy
 ↓
Audit
```

---

## 90. Scenario — Cross-Team Artifact Access

If:

```text
Team A
```

can access:

```text
Team B private repository
```

investigate:

```text
group membership
shared permission target
broad project role
direct user permission
virtual repository exposure
```

---

# PART XIX — ENTERPRISE ARCHITECTURE

## 91. Human Access

```text
User
 ↓
SSO + MFA
 ↓
Groups
 ↓
Project / Permission Targets
 ↓
Repositories
```

---

## 92. CI Access

```text
CI
 ↓
Service Identity
 ↓
Scoped Token
 ↓
Permission Target
 ↓
Specific Repositories
```

---

## 93. GitOps Access

```text
Argo CD
 ↓
Dedicated Identity
 ↓
READ
 ↓
Helm/OCI Repositories
```

---

## 94. Runtime Access

```text
Kubernetes
 ↓
Image Pull Identity
 ↓
READ
 ↓
Docker/OCI Repository
```

---

## 95. Platform Administration

```text
Platform Admin
 ↓
SSO + MFA
 ↓
Admin Group
 ↓
Administrative Permissions
```

Use a small number of highly trusted administrators.

---

# PART XX — PRODUCTION DESIGN EXAMPLE

## 96. Example Organization

```text
Project:
Payments

Repositories:
payment-maven-local
payment-docker-local
payment-helm-local
maven-virtual
docker-virtual
helm-virtual
```

---

## 97. Developer Role

```text
READ:
maven-virtual
docker-virtual
helm-virtual

DEPLOY:
development package repository if required

DELETE:
NO

ADMIN:
NO
```

---

## 98. Build Role

```text
READ:
required virtual repositories

DEPLOY:
payment development repository

DELETE:
NO

ADMIN:
NO
```

---

## 99. Release Role

```text
READ:
required repositories

DEPLOY:
approved production release repositories

DELETE:
restricted

ADMIN:
NO
```

---

## 100. Platform Admin

```text
Administrative access
```

Protected by:

```text
SSO
MFA
strong approval
audit
```

---

# PART XXI — INTERVIEW PREPARATION

## 101. What Is RBAC?

Answer:

```text
RBAC is an authorization model where access is assigned through roles,
groups or permission structures instead of managing every permission
individually for every user.
```

---

## 102. What Is a Permission Target?

Answer:

```text
A permission target defines which identities can perform which
operations on selected Artifactory resources, typically repositories
and relevant paths.
```

---

## 103. Why Use Groups?

Answer:

```text
Groups centralize access management. Instead of assigning the same
repository permissions to many individual users, I assign the
permissions to a group and manage membership separately.
```

---

## 104. How Do You Secure Jenkins?

Answer:

```text
I create a dedicated Jenkins service identity, grant only required
READ and DEPLOY permissions, store its token in Jenkins Credentials,
avoid DELETE and ADMIN permissions and rotate the credential
periodically.
```

---

## 105. Why Should Jenkins Not Be Admin?

Answer:

```text
If Jenkins or a pipeline is compromised, an Artifactory admin
credential would create a very large blast radius, potentially
allowing modification or deletion of critical artifacts and
configuration.
```

---

## 106. How Do You Separate Developers and Release Engineers?

Answer:

```text
Developers normally receive read access and development publishing
where required. Release identities receive controlled production
deployment access. Production publishing is performed through the
release pipeline rather than unrestricted developer access.
```

---

## 107. How Do You Troubleshoot 403?

Answer:

```text
I confirm authentication first and then inspect the identity's group
membership, project role, permission target, repository/path access
and requested operation. I determine exactly which permission is
missing.
```

---

## 108. How Do You Prevent Permission Drift?

Answer:

```text
I use groups and roles, avoid unnecessary direct user permissions,
perform periodic access reviews, remove stale accounts and tokens
and automate onboarding/offboarding where possible.
```

---

## 109. How Do You Design RBAC for 100+ Teams?

Answer:

```text
I establish standard project and repository boundaries, define
standard developer/build/release roles, use groups, keep production
permissions tightly controlled, use service identities for
automation and perform centralized access reviews.
```

---

## 110. What Is the Principle of Least Privilege?

Answer:

```text
It means an identity receives only the minimum permissions required
to perform its task. For example, a Kubernetes workload should need
READ access to its image repository, not DEPLOY or ADMIN access.
```

---

## 111. How Do You Handle Emergency Access?

Answer:

```text
I use a controlled break-glass process with strong approval,
short-lived access where possible, comprehensive audit logging and
mandatory post-incident review.
```

---

# PART XXII — PRODUCTION CHECKLIST

## 112. Identity

```text
[ ] users
[ ] groups
[ ] service identities
[ ] owners
[ ] lifecycle
```

---

## 113. Permissions

```text
[ ] READ
[ ] DEPLOY
[ ] DELETE restricted
[ ] ADMIN restricted
[ ] project boundaries
[ ] repository boundaries
[ ] path restrictions where needed
```

---

## 114. CI/CD

```text
[ ] dedicated service identity
[ ] secret store
[ ] least privilege
[ ] read/write separation
[ ] token rotation
[ ] no admin credentials
```

---

## 115. Human Access

```text
[ ] SSO
[ ] MFA
[ ] groups
[ ] production restrictions
[ ] access reviews
[ ] offboarding
```

---

## 116. Security

```text
[ ] audit logging
[ ] failed access monitoring
[ ] credential rotation
[ ] dormant account cleanup
[ ] incident response
[ ] break-glass process
```

---

## 117. Operations

```text
[ ] permission inventory
[ ] periodic reviews
[ ] documented owners
[ ] change management
[ ] DR
```

---

# PART XXIII — GOLDEN RULES

## 118. Rules

```text
1. Authentication identifies the caller; RBAC controls what it can do.

2. Use groups and roles instead of excessive direct user permissions.

3. Use dedicated service identities for automation.

4. Apply least privilege everywhere.

5. Give consumers READ access only when they only consume artifacts.

6. Give publishers DEPLOY only to repositories they own.

7. Restrict DELETE heavily.

8. Restrict ADMIN to trusted platform administrators.

9. Never use Artifactory admin credentials for normal CI/CD.

10. Separate build identities from release identities where practical.

11. Keep production publication controlled.

12. Use SSO + MFA for human administrators.

13. Use scoped machine credentials for CI and GitOps.

14. Keep Kubernetes runtime credentials read-only.

15. Review direct permissions and group membership regularly.

16. Remove stale users, groups, tokens and service identities.

17. Audit permission changes.

18. Monitor unexpected artifact deployments and deletions.

19. Keep repository boundaries simple enough to audit.

20. Avoid permission designs based on hundreds of one-off exceptions.

21. Use project boundaries for large organizational environments where
    appropriate.

22. Design break-glass access separately from normal access.

23. Treat 401 as an authentication investigation and 403 as an
    authorization investigation, while validating the complete
    request context.

24. Revoke compromised credentials immediately.

25. Validate exact RBAC, permission-target, project-role and API
    behavior against the deployed JFrog/Artifactory version before
    production rollout.
```

---