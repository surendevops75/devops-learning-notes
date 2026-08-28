# Production-Artifactory-Architecture

## 1. Purpose

This file presents a complete production architecture for JFrog
Artifactory in an enterprise DevOps, Kubernetes, AWS, CI/CD and
multi-team environment.

The objective is to connect all previous concepts into one practical
platform architecture:

```text
Developers
    |
    v
Git / Source Control
    |
    v
CI/CD
    |
    +--> Build
    +--> Test
    +--> Security Scan
    +--> Publish
    |
    v
Artifactory
    |
    +--> Local Repositories
    +--> Remote Repositories
    +--> Virtual Repositories
    +--> Build Info
    +--> Security
    |
    v
Promotion
    |
    v
Kubernetes / EKS
    |
    v
Production Applications
```

This file covers:

- production architecture principles
- enterprise repository strategy
- naming conventions
- network architecture
- DNS
- load balancing
- TLS
- HA
- database
- filestore
- object storage
- AWS
- EKS
- multi-AZ
- CI/CD
- Jenkins
- GitHub Actions
- GitLab
- Kubernetes
- Helm
- Docker
- Maven
- NPM
- PyPI
- RBAC
- security
- Build Info
- artifact promotion
- environment strategy
- backup
- DR
- monitoring
- logging
- alerting
- capacity planning
- cost
- operational ownership
- troubleshooting
- production incidents
- architecture review
- interview preparation
- production checklist

---

# PART I — PRODUCTION PRINCIPLES

## 2. What Makes an Artifactory Architecture Production-Ready?

A production architecture should provide:

```text
Availability
Security
Scalability
Performance
Recoverability
Observability
Governance
Traceability
Operational simplicity
```

---

## 3. Production Is a System

Do not design only:

```text
Artifactory server
```

Design:

```text
DNS
+
Load Balancer
+
Artifactory
+
Database
+
Storage
+
Network
+
Identity
+
CI/CD
+
Kubernetes
+
Monitoring
+
Backup
+
DR
```

---

## 4. No Single Point of Failure

Review every layer:

```text
DNS
 |
LB
 |
Artifactory
 |
Database
 |
Storage
 |
Network
 |
Identity
```

---

# PART II — REFERENCE ARCHITECTURE

## 5. Enterprise Architecture

```text
                           Users
                             |
                       Identity / SSO
                             |
                             v
                         DNS / WAF
                             |
                             v
                      Load Balancer
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
           Art-A          Art-B          Art-C
              |              |              |
              +--------------+--------------+
                             |
                +------------+------------+
                |                         |
                v                         v
             Database              Artifact Storage
                |                         |
                +------------+------------+
                             |
                         Backup / DR
```

---

## 6. Client Categories

Production Artifactory may serve:

```text
Developers
Jenkins
GitHub Actions
GitLab
Kubernetes
EKS
Argo CD
Helm
Docker clients
Package managers
Security systems
Automation
```

---

# PART III — NETWORK ARCHITECTURE

## 7. Public vs Private

Prefer private Artifactory access for internal enterprise workloads when
the architecture allows it.

Example:

```text
Corporate Network
       |
       v
Private DNS
       |
       v
Private Load Balancer
       |
       v
Artifactory
```

---

## 8. Internet Access

If external access is required:

```text
Internet
   |
   v
WAF / Edge
   |
   v
Load Balancer
   |
   v
Artifactory
```

Expose only required endpoints.

---

## 9. Network Segmentation

Separate:

```text
client traffic
administrative traffic
database traffic
storage traffic
monitoring traffic
```

according to the supported architecture.

---

# PART IV — DNS

## 10. Production DNS

Use a stable name:

```text
artifactory.company.com
```

Do not expose individual node names to clients.

---

## 11. Internal DNS

Example:

```text
artifactory.company.internal
```

Use private DNS where appropriate.

---

## 12. DNS Failover

For DR:

```text
artifactory.company.com
        |
        +--> Primary
        |
        +--> DR
```

---

# PART V — LOAD BALANCING

## 13. Load Balancer

```text
Client
  |
  v
Load Balancer
  |
  +--> Art-A
  +--> Art-B
  +--> Art-C
```

---

## 14. Health Checks

Health checks should identify nodes that cannot safely receive traffic.

Avoid relying only on:

```text
TCP port open
```

when application-level health is required.

---

## 15. TLS

Recommended path:

```text
Client
 |
HTTPS
 |
Load Balancer
 |
HTTPS
 |
Artifactory
```

Exact TLS termination depends on security architecture.

---

# PART VI — ARTIFACTORY NODE LAYER

## 16. Multiple Nodes

Production HA requires multiple application nodes where supported by the
selected JFrog architecture.

Example:

```text
Art-A
Art-B
Art-C
```

---

## 17. Failure Domain Distribution

Distribute nodes across:

```text
availability zones
physical hosts
Kubernetes nodes
failure domains
```

---

## 18. N+1 Capacity

If:

```text
3 nodes
```

are required for normal capacity, consider:

```text
4 nodes
```

when business availability requirements demand continued capacity
after one failure.

---

# PART VII — DATABASE LAYER

## 19. Database

Artifactory metadata depends on its database.

Architecture:

```text
Artifactory
    |
    v
Database
```

---

## 20. Database HA

Avoid:

```text
Multiple Artifactory nodes
        |
        v
Single database server
```

unless the failure model explicitly accepts that risk.

---

## 21. Database Monitoring

Monitor:

```text
CPU
memory
connections
latency
locks
storage
replication
```

---

# PART VIII — ARTIFACT STORAGE

## 22. Filestore

Artifacts include:

```text
Docker layers
JAR
NPM
PyPI
Helm
generic binaries
```

---

## 23. Storage Design

Depending on supported architecture:

```text
shared filesystem
object storage
enterprise storage
```

may be used.

---

## 24. Storage Requirements

Production storage must provide:

```text
availability
durability
performance
capacity
backup
security
```

---

# PART IX — REPOSITORY STRATEGY

## 25. Local

Use local repositories for internally published artifacts.

Example:

```text
docker-prod-local
maven-release-local
npm-prod-local
helm-prod-local
```

---

## 26. Remote

Use remote repositories to control external dependencies.

Example:

```text
docker-approved-remote
maven-central-remote
npmjs-remote
pypi-remote
```

---

## 27. Virtual

Use virtual repositories as controlled consumption endpoints.

Example:

```text
docker-virtual
maven-virtual
npm-virtual
pypi-virtual
```

---

# PART X — NAMING CONVENTIONS

## 28. Naming

Use predictable names:

```text
<ecosystem>-<purpose>-<environment>
```

Example:

```text
docker-prod-local
maven-release-local
npm-remote
helm-prod-local
```

Actual naming should follow enterprise standards.

---

## 29. Avoid Random Names

Bad:

```text
repo1
test123
newrepo
finalrepo
```

---

# PART XI — ENVIRONMENT STRATEGY

## 30. Environment Separation

Possible model:

```text
Development
Staging
Production
```

---

## 31. Repository Separation

Example:

```text
docker-dev-local
docker-stage-local
docker-prod-local
```

---

## 32. Promotion Model

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

# PART XII — BUILD ONCE

## 33. Build Once Principle

Bad:

```text
Build DEV
 ↓
Build STAGE
 ↓
Build PROD
```

Good:

```text
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Promote same artifact
```

---

## 34. Why?

It guarantees:

```text
same binary
same image
same digest
```

across environments.

---

# PART XIII — BUILD INFO

## 35. Build Info

Build Info connects:

```text
Git
 ↓
CI
 ↓
Build
 ↓
Dependencies
 ↓
Artifact
 ↓
Deployment
```

---

## 36. Production Value

During an incident:

```text
Production image
 ↓
Digest
 ↓
Artifactory
 ↓
Build Info
 ↓
CI build
 ↓
Git commit
```

---

# PART XIV — CI/CD ARCHITECTURE

## 37. Jenkins

```text
Git
 |
v
Jenkins
 |
 +--> Test
 +--> Build
 +--> Scan
 +--> Publish
 |
 v
Artifactory
```

---

## 38. GitHub Actions

```text
GitHub
 |
v
Actions
 |
v
Artifactory
```

Use protected environments and secure secrets.

---

## 39. GitLab

```text
GitLab
 |
v
Runner
 |
v
Artifactory
```

Use protected and masked variables.

---

# PART XV — CI SECURITY

## 40. Service Identity

Every pipeline should have a dedicated identity.

Example:

```text
svc-payment-ci
```

---

## 41. Permissions

Example:

```text
READ:
dependency virtual

DEPLOY:
payment-local

DELETE:
NO

ADMIN:
NO
```

---

## 42. Secret Management

Never store:

```text
ARTIFACTORY_TOKEN=...
```

in Git.

Use:

```text
CI secret store
Vault
AWS Secrets Manager
```

where appropriate.

---

# PART XVI — DOCKER ARCHITECTURE

## 43. Docker Flow

```text
Developer
 |
v
Dockerfile
 |
v
CI
 |
v
Build
 |
v
Scan
 |
v
Artifactory Docker Local
 |
v
Promotion
 |
v
Kubernetes
```

---

## 44. Image Naming

Example:

```text
artifactory.company.com/docker-prod-local/payment-service:4.2.1
```

---

## 45. Digest

Production should track:

```text
sha256:...
```

---

# PART XVII — KUBERNETES ARCHITECTURE

## 46. Kubernetes Pull

```text
Pod
 |
v
containerd
 |
v
Artifactory
 |
v
Image
```

---

## 47. Private Registry

Use:

```text
imagePullSecret
```

or an approved workload/registry identity mechanism.

---

## 48. Runtime Permissions

Kubernetes runtime generally needs:

```text
READ
```

not:

```text
DEPLOY
DELETE
```

---

# PART XVIII — EKS ARCHITECTURE

## 49. AWS Production

```text
AWS
 |
 +--> VPC
      |
      +--> EKS
      |
      +--> Artifactory connectivity
```

---

## 50. Private EKS

Typical:

```text
private subnets
private nodes
security groups
route tables
private DNS
```

---

## 51. Artifactory Connectivity

```text
EKS Node
 |
v
Private Network
 |
v
Artifactory LB
```

---

# PART XIX — HELM

## 52. Helm Architecture

```text
CI
 |
v
Artifactory Helm/OCI
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

## 53. Image Reference

Example:

```yaml
image:
  repository: artifactory.company.com/docker-prod-local/payment-service
  tag: "4.2.1"
```

---

# PART XX — GITOPS

## 54. GitOps Model

```text
CI
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
Kubernetes
```

---

## 55. Separation

CI:

```text
create artifact
```

GitOps:

```text
declare deployment state
```

Artifactory:

```text
store artifact
```

---

# PART XXI — SECURITY ARCHITECTURE

## 56. Security Layers

```text
SSO / MFA
    |
RBAC
    |
Network
    |
TLS
    |
Repository Permissions
    |
Artifact Scanning
    |
Signing
    |
Audit
```

---

## 57. Admin Security

Protect:

```text
admin users
admin endpoints
break-glass accounts
tokens
```

---

## 58. Delete Permissions

Restrict:

```text
DELETE
```

to controlled automation or authorized operators.

---

# PART XXII — SUPPLY CHAIN SECURITY

## 59. Dependency Flow

```text
Application
 |
v
Virtual Repository
 |
v
Approved Remote
 |
v
External Source
```

---

## 60. Dependency Scanning

Check:

```text
CVE
license
malware
secrets
policy
```

---

## 61. SBOM

Maintain software component visibility where required.

---

## 62. Signing

For high-assurance environments:

```text
Build
 ↓
Sign
 ↓
Store
 ↓
Verify
 ↓
Deploy
```

---

# PART XXIII — HIGH AVAILABILITY

## 63. HA

Reference:

```text
Load Balancer
 |
+--> Art-A
+--> Art-B
+--> Art-C
```

---

## 64. Supporting HA

Also consider:

```text
database
storage
DNS
network
```

---

## 65. Failure Example

If Art-A fails:

```text
LB
 |
+--> Art-B
+--> Art-C
```

Service continues if capacity is sufficient.

---

# PART XXIV — BACKUP

## 66. Backup Scope

Protect:

```text
database
filestore
configuration
security settings
required certificates/keys
```

according to the supported JFrog recovery architecture.

---

## 67. Backup Isolation

Use:

```text
separate account
immutable storage
restricted credentials
```

where appropriate.

---

# PART XXV — DISASTER RECOVERY

## 68. DR

Example:

```text
Primary Region
      |
      v
Replication / Backup
      |
      v
DR Region
```

---

## 69. DR Endpoint

Use:

```text
artifactory.company.com
```

and switch routing during DR.

---

## 70. DR Testing

Test:

```text
restore
DNS
authentication
artifact pull
artifact publish where required
Kubernetes deployment
CI/CD
```

---

# PART XXVI — OBSERVABILITY

## 71. Metrics

Monitor:

```text
CPU
memory
request rate
latency
errors
database
storage
network
```

---

## 72. Logs

Centralize:

```text
Artifactory
LB
proxy
database
Kubernetes
```

---

## 73. Alerts

High-value alerts:

```text
node failure
5xx spike
storage near full
database failure
certificate expiry
authentication anomalies
```

---

# PART XXVII — CAPACITY PLANNING

## 74. Capacity Dimensions

Plan:

```text
CPU
memory
storage
network
database
concurrency
CI load
Kubernetes pull bursts
```

---

## 75. Image Pull Burst

Example:

```text
Deployment
 ↓
500 Pods
 ↓
500 image pulls
```

Plan for burst capacity.

---

## 76. Storage Growth

Track:

```text
daily growth
monthly growth
retention
repository growth
```

---

# PART XXVIII — PERFORMANCE

## 77. Large Images

Optimize:

```text
base image
layers
dependencies
build context
```

---

## 78. Network Latency

For global environments:

```text
Client
 |
v
Nearest appropriate registry endpoint
```

can reduce latency where architecture supports it.

---

## 79. Caching

Use remote repository caching for external dependencies where
appropriate.

---

# PART XXIX — COST

## 80. Cost Categories

Consider:

```text
license
compute
database
storage
network
backup
DR
monitoring
operations
```

---

## 81. TCO

Do not compare only:

```text
storage price
```

Compare:

```text
Total Cost of Ownership
```

---

# PART XXX — OPERATIONAL OWNERSHIP

## 82. Platform Team

Own:

```text
Artifactory
HA
upgrades
backup
DR
monitoring
capacity
security
```

---

## 83. Application Team

Own:

```text
application artifacts
Dockerfile
dependencies
release version
deployment configuration
```

---

## 84. Security Team

May own or govern:

```text
security policies
vulnerability policy
access reviews
audit
incident response
```

---

# PART XXXI — PRODUCTION CHANGE MANAGEMENT

## 85. Change Process

```text
Plan
 ↓
Review
 ↓
Backup
 ↓
Change
 ↓
Validate
 ↓
Monitor
 ↓
Document
```

---

## 86. High-Risk Changes

Examples:

```text
database migration
storage change
certificate change
network change
major Artifactory upgrade
repository migration
```

Require controlled rollout.

---

# PART XXXII — UPGRADES

## 87. Upgrade

Before:

```text
backup
compatibility check
release notes
test
```

During:

```text
rolling change
```

After:

```text
health
repositories
Docker pull
Maven download
Kubernetes deployment
```

---

# PART XXXIII — TROUBLESHOOTING MODEL

## 88. Layered Troubleshooting

When something fails:

```text
1. DNS
2. Network
3. TLS
4. Load Balancer
5. Artifactory
6. Database
7. Storage
8. Authentication
9. Authorization
10. Artifact
11. Client
```

---

## 89. 401

Usually:

```text
authentication
```

Check:

```text
token
identity
expiration
```

---

## 90. 403

Usually:

```text
authorization
```

Check:

```text
permission target
repository
path
operation
```

---

## 91. 404

Could mean:

```text
wrong repository
wrong artifact
wrong version
artifact not published
```

---

## 92. 5xx

Investigate:

```text
Artifactory
database
storage
LB
network
```

---

# PART XXXIV — PRODUCTION INCIDENTS

## 93. Incident — Artifactory Unavailable

Response:

```text
Check DNS
 ↓
Check LB
 ↓
Check nodes
 ↓
Check DB
 ↓
Check storage
 ↓
Check network
```

---

## 94. Incident — Kubernetes ImagePullBackOff

Check:

```text
image
tag/digest
secret
DNS
TLS
network
READ permission
```

---

## 95. Incident — CI Cannot Publish

Check:

```text
credentials
DEPLOY permission
repository
storage
network
```

---

## 96. Incident — Storage Full

Response:

```text
protect service
identify growth
review retention
expand storage
clean eligible artifacts
```

Do not delete production artifacts blindly.

---

## 97. Incident — Database Failure

Response:

```text
verify DB status
check application dependency
failover/recovery
validate metadata
monitor
```

---

# PART XXXV — PRODUCTION SECURITY CHECKLIST

## 98. Identity

```text
[ ] SSO
[ ] MFA
[ ] service identities
[ ] token rotation
[ ] offboarding
```

---

## 99. Authorization

```text
[ ] least privilege
[ ] permission targets
[ ] no unnecessary ADMIN
[ ] no unnecessary DELETE
[ ] environment separation
```

---

## 100. Network

```text
[ ] private access
[ ] TLS
[ ] firewall
[ ] segmentation
[ ] restricted admin access
```

---

## 101. Supply Chain

```text
[ ] approved upstreams
[ ] scanning
[ ] SBOM
[ ] signing
[ ] immutable releases
[ ] Build Info
```

---

# PART XXXVI — PRODUCTION READINESS

## 102. Before Go-Live

Verify:

```text
[ ] HA
[ ] DB HA
[ ] storage
[ ] DNS
[ ] LB
[ ] TLS
[ ] RBAC
[ ] CI
[ ] Kubernetes
[ ] monitoring
[ ] backup
[ ] DR
```

---

## 103. Failure Tests

Test:

```text
[ ] Artifactory node failure
[ ] database failure where safely testable
[ ] storage failure scenario
[ ] LB failure scenario
[ ] network interruption
[ ] credential rotation
[ ] backup restore
[ ] DR
```

---

# PART XXXVII — INTERVIEW PREPARATION

## 104. Describe Your Production Artifactory Architecture

Answer:

```text
I would design Artifactory as a highly available enterprise artifact
platform behind a highly available load-balancing layer. The
Artifactory nodes would be distributed across failure domains, with
highly available database and artifact storage. Internal clients would
use private DNS and TLS. CI would publish using least-privilege service
identities, while Kubernetes would use separate read-only registry
access. I would also implement security scanning, Build Info,
monitoring, centralized logging, backup and tested disaster recovery.
```

---

## 105. How Do You Design Artifactory for EKS?

Answer:

```text
I place the Artifactory endpoint on a private network path accessible
from EKS, use TLS and controlled registry authentication, distribute
Artifactory components across availability zones, provide HA for
database and storage, and configure Kubernetes workloads with
read-only image access. I also test image pulls during scaling and
node replacement.
```

---

## 106. How Do You Make Artifactory Highly Available?

Answer:

```text
I use multiple Artifactory nodes behind a highly available load
balancer and remove single points of failure from the database,
storage, DNS and network layers. I distribute nodes across failure
domains, maintain N+1 capacity, implement health checks and test
controlled node failures.
```

---

## 107. How Do You Secure It?

Answer:

```text
I use SSO and MFA for humans, dedicated service identities for
automation, least-privilege permission targets, scoped tokens, TLS,
private networking, repository controls, vulnerability scanning,
artifact immutability, signing where required, audit logging and
continuous monitoring.
```

---

## 108. How Do You Handle Disaster Recovery?

Answer:

```text
I define business RTO/RPO first, then protect the database, filestore
and required configuration using supported backup and replication
methods. I maintain a separate recovery environment or recovery
capability, protect backups from deletion, document DNS and credential
failover and regularly test restoration and representative
Kubernetes/CI workflows.
```

---

## 109. How Do You Troubleshoot Production Artifactory Issues?

Answer:

```text
I use a layered approach: DNS, network, TLS, load balancer,
Artifactory node health, database, storage, authentication,
authorization and artifact state. I correlate logs and metrics and
check the exact error code before changing configuration.
```

---

# PART XXXVIII — ARCHITECTURE REVIEW QUESTIONS

## 110. Availability

Ask:

```text
What happens if one node fails?
What happens if one AZ fails?
What happens if the database fails?
What happens if storage fails?
```

---

## 111. Security

Ask:

```text
Who can publish?
Who can delete?
Who can administer?
How are credentials rotated?
How are artifacts scanned?
```

---

## 112. Recovery

Ask:

```text
What is the RTO?
What is the RPO?
When was restore last tested?
When was DR last tested?
```

---

## 113. Scale

Ask:

```text
How many artifacts?
How many daily pulls?
How many CI builds?
How large are images?
What is peak concurrency?
```

---

# PART XXXIX — GOLDEN RULES

## 114. Rules

```text
1. Design Artifactory as a platform, not a single server.

2. Remove single points of failure.

3. Use HA across Artifactory and its critical dependencies.

4. Distribute failure domains.

5. Maintain N+1 capacity where required.

6. Use private networking for internal workloads when practical.

7. Use stable DNS.

8. Protect all traffic with TLS.

9. Use local repositories for controlled internal publication.

10. Use remote repositories for controlled external dependencies.

11. Use virtual repositories as controlled consumption endpoints.

12. Use predictable repository naming.

13. Separate environments according to governance requirements.

14. Build once and promote the same artifact.

15. Track Build Info and image digests.

16. Separate CI publish permissions from runtime pull permissions.

17. Protect administrator accounts.

18. Restrict DELETE.

19. Scan artifacts and dependencies.

20. Use immutable releases.

21. Use SBOM and signing where required.

22. Monitor every critical dependency.

23. Centralize logs.

24. Alert on errors, latency, storage, database and availability.

25. Protect backups separately from production.

26. Test restore.

27. Test DR.

28. Test node and failure-domain recovery.

29. Forecast storage and network growth.

30. Compare total cost of ownership.

31. Define platform ownership clearly.

32. Use controlled change management.

33. Validate upgrades before production.

34. Document operational runbooks.

35. Correlate Git, CI, Build Info, artifact and production deployment.

36. Never assume HA or DR works until it has been tested.

37. Validate the exact Artifactory edition, version and supported
    production topology before implementation.
```

---