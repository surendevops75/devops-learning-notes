# Artifactory-Architecture

## 1. Purpose

This file explains how JFrog Artifactory is structured internally and how to design it for DevOps and production environments.

The goal is to understand:

```text
Clients
   ↓
DNS / Load Balancer
   ↓
Artifactory
   ↓
Repositories
   ↓
Metadata + Artifact Storage
   ↓
Database / Persistent Storage
```

It also covers production architecture, networking, security boundaries, HA concepts, scaling, AWS deployment considerations, Kubernetes integration, failure domains, observability and interview scenarios.

---

## 2. Artifactory Architecture Mental Model

At a high level:

```text
                 Developers / CI / CD
                         |
                         v
                    DNS / URL
                         |
                         v
                 Load Balancer / Proxy
                         |
                         v
                Artifactory Services
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
     Local            Remote           Virtual
   Repository       Repository        Repository
        |                |                |
        +----------------+----------------+
                         |
                         v
              Metadata / Repository State
                         |
                         v
             Persistent Storage / Database
```

The exact physical implementation varies by Artifactory deployment model and supported product version.

---

## 3. Logical vs Physical Architecture

Separate two concepts.

Logical architecture:

```text
Users
 ↓
Artifactory
 ↓
Repositories
 ↓
Artifacts
```

Physical architecture:

```text
Load Balancer
 ↓
Artifactory Nodes
 ↓
Database / Metadata
 ↓
Artifact Storage
```

Logical repository design does not necessarily mean each repository is a separate physical server.

---

## 4. Main Architectural Layers

A production Artifactory environment can be viewed as these layers:

```text
1. Client layer
2. DNS layer
3. Network/load-balancing layer
4. Application/service layer
5. Repository layer
6. Metadata layer
7. Artifact storage layer
8. Database/persistence layer
9. Security/identity layer
10. Observability layer
11. Backup/DR layer
```

---

## 5. Client Layer

Clients can include:

```text
Developer laptops
Jenkins
GitHub Actions
GitLab CI
Docker
Maven
Gradle
npm
pip
Helm
Kubernetes nodes
Argo CD
Automation scripts
```

All of these act as consumers or publishers.

---

## 6. DNS Layer

A production endpoint might be:

```text
artifactory.company.com
```

DNS resolves the name to the organization's ingress/load-balancing layer.

Example:

```text
artifactory.company.com
          ↓
     Load Balancer
```

DNS should be designed for reliability and controlled changes.

---

## 7. Load Balancer Layer

For production, clients should generally access Artifactory through a stable endpoint.

Conceptually:

```text
                  artifactory.company.com
                           |
                           v
                    Load Balancer
                     /         \
                    /           \
                   v             v
             Artifactory-1  Artifactory-2
```

This provides a stable entry point and can support high availability depending on the overall Artifactory architecture.

---

## 8. Reverse Proxy

An organization may place a reverse proxy or ingress layer in front of Artifactory.

Example:

```text
Internet / Corporate Network
             ↓
        WAF / Proxy
             ↓
       Load Balancer
             ↓
         Artifactory
```

Responsibilities can include:

```text
TLS termination
routing
security headers
request filtering
access control
traffic management
```

The exact supported architecture should follow JFrog's documentation for the deployed version.

---

## 9. TLS

Production Artifactory access should use HTTPS.

Conceptually:

```text
Client
  |
  | HTTPS
  v
artifactory.company.com
```

TLS protects:

```text
credentials
tokens
artifact data
metadata
API traffic
```

---

## 10. Network Segmentation

A production architecture should separate trust zones.

Example:

```text
                 Internet
                    |
                   WAF
                    |
             Public/DMZ Layer
                    |
              Load Balancer
                    |
             Private Network
                    |
              Artifactory
                    |
       +------------+------------+
       |                         |
    Storage                    Database
```

Not every Artifactory installation needs this exact topology, but the principle of reducing unnecessary exposure is important.

---

## 11. Why Artifactory Should Not Be Opened Unnecessarily

If the repository is publicly exposed without appropriate controls, attackers could attempt:

```text
credential attacks
artifact downloads
artifact uploads
repository enumeration
API abuse
supply-chain attacks
```

Use network controls in addition to application-level authentication.

---

## 12. Repository Layer

The repository layer is where Artifactory organizes package content.

Main logical types:

```text
Local
Remote
Virtual
```

These are logical repository constructs.

---

## 13. Local Repository Architecture

```text
CI
 ↓
Maven Local
 ↓
Artifact
```

Example:

```text
payment-service-2.5.0.jar
```

The organization owns this artifact.

---

## 14. Remote Repository Architecture

```text
CI
 ↓
Maven Remote
 ↓
Maven Central
```

First request:

```text
Client
 ↓
Artifactory
 ↓
Upstream
 ↓
Artifactory cache
 ↓
Client
```

Subsequent request may be served from Artifactory's cached content.

---

## 15. Virtual Repository Architecture

```text
                  maven-virtual
                       |
          +------------+------------+
          |            |            |
          v            v            v
      maven-local  maven-remote  other-approved
```

Consumers use one endpoint.

---

## 16. Why Virtual Repositories Are Important

Suppose an application currently depends on:

```text
internal-library
external-library-A
external-library-B
```

Instead of configuring every repository separately:

```text
Application
      ↓
maven-virtual
```

Artifactory handles repository resolution according to configuration.

---

## 17. Repository Resolution

The exact resolution behavior depends on repository configuration, package format and version.

Conceptually:

```text
Consumer
   ↓
Virtual Repository
   ↓
Local repositories
   ↓
Remote repositories
   ↓
Upstream
```

---

## 18. Repository Naming Architecture

A mature organization may define:

```text
maven-local
maven-remote-central
maven-virtual

npm-local
npm-remote
npm-virtual

docker-local
docker-remote
docker-virtual

helm-local
helm-remote
helm-virtual
```

Naming should be documented and consistent.

---

## 19. Avoid Repository Explosion

Do not create a separate repository for every:

```text
team
application
environment
pipeline
developer
```

unless there is a real isolation or governance requirement.

Too many repositories increase:

```text
administration
permissions complexity
monitoring complexity
cleanup complexity
consumer configuration
```

---

## 20. Repository Design Criteria

Choose repository boundaries based on:

```text
package format
security boundary
ownership
lifecycle
retention
promotion
compliance
access pattern
```

---

## 21. Artifact Storage Layer

Artifactory stores artifact content separately from the logical repository concept.

Conceptually:

```text
Repository
    ↓
Artifact
    ↓
Binary/Object Content
```

The exact physical storage implementation depends on deployment architecture.

---

## 22. Metadata Layer

Artifact metadata can include:

```text
name
version
coordinates
properties
checksums
build information
repository information
relationships
```

Metadata is essential for searching and managing artifacts.

---

## 23. Database / Persistence Layer

Artifactory relies on persistent state for metadata and platform operation.

In production, database availability is therefore an important architectural concern.

Do not treat:

```text
Artifactory application node
```

as the only component that matters.

---

## 24. Persistent Storage

Artifact data needs durable storage.

Conceptually:

```text
Artifactory
    |
    +---- Metadata
    |
    +---- Artifact binaries
```

Storage must be sized for:

```text
current artifacts
growth
cache
retention
replication
recovery
```

---

## 25. Storage Growth

Capacity planning should estimate:

```text
daily uploads
daily downloads
remote cache growth
retention period
artifact size
container image size
Helm package size
backup requirements
```

---

## 26. Storage Monitoring

Monitor:

```text
used capacity
free capacity
growth rate
largest repositories
largest artifacts
cache growth
I/O
latency
errors
```

---

## 27. Storage Exhaustion Scenario

Suppose storage reaches 100%.

Potential consequences:

```text
artifact upload failures
metadata operations failing
application instability
CI failures
deployment failures
```

This is why storage alerts should trigger before exhaustion.

---

## 28. Capacity Alerting

Example policy:

```text
70% → warning
80% → investigation
90% → urgent capacity action
```

Exact thresholds should be determined from workload behavior and operational experience.

---

## 29. Artifactory Nodes

A production HA design may contain multiple Artifactory nodes.

Conceptually:

```text
              Load Balancer
                    |
          +---------+---------+
          |                   |
          v                   v
    Artifactory Node 1  Artifactory Node 2
          |                   |
          +---------+---------+
                    |
             Shared/managed
               persistence
```

Do not assume that simply adding nodes automatically creates a valid HA deployment; supported topology and licensing requirements must be followed.

---

## 30. Stateless vs Stateful Thinking

When designing a distributed application, determine which state lives where.

For Artifactory:

```text
Application/service processing
          +
Persistent metadata/state
          +
Artifact storage
```

must be considered together.

---

## 31. Why Shared Persistence Matters

If Node 1 receives an upload and Node 2 later handles a download, both must operate against the same authoritative persistent state according to the supported architecture.

Conceptually:

```text
Upload → Node 1
             ↓
        Shared State
             ↑
Download → Node 2
```

---

## 32. Load Balancing

The load balancer distributes requests.

Potentially:

```text
Client A → Node 1
Client B → Node 2
Client C → Node 1
```

The exact session and routing requirements depend on the deployed architecture.

---

## 33. Health Checks

The load balancer should detect unhealthy application nodes.

Conceptually:

```text
Node 1 → healthy
Node 2 → unhealthy
Node 3 → healthy
```

Traffic should not continue to an unavailable node when the health-check design supports removing it.

---

## 34. Node Failure

If one node fails:

```text
Client
 ↓
Load Balancer
 ↓
Healthy Node
```

The objective of HA is to avoid a complete repository outage from a single application-node failure.

---

## 35. Database Failure

A database or persistence failure can affect Artifactory even if application nodes are healthy.

Therefore:

```text
HA Artifactory
+
HA persistence
```

must be considered together.

---

## 36. Storage Failure

Similarly, application-node HA does not protect against every storage failure.

If artifact storage becomes unavailable:

```text
Node 1 ─┐
Node 2 ─┼→ Storage unavailable
Node 3 ─┘
```

the service may still be impaired.

---

## 37. Complete Failure-Domain Thinking

Production architecture should consider:

```text
node failure
load balancer failure
database failure
storage failure
network failure
DNS failure
identity-provider failure
region failure
```

---

## 38. AWS Reference Architecture

A conceptual AWS design:

```text
                  Route 53
                     |
                     v
              ALB / Ingress
                     |
              Private Subnets
                     |
          +----------+----------+
          |                     |
      Artifactory-1       Artifactory-2
          |                     |
          +----------+----------+
                     |
            Managed Persistence
                     |
              Object/File Storage
```

The exact architecture must follow the supported JFrog deployment model and AWS design requirements.

---

## 39. AWS Network Placement

For a private enterprise Artifactory deployment, application nodes can be placed in private subnets.

Example:

```text
VPC
 |
 +-- Public Subnets
 |      └── Load Balancer
 |
 +-- Private Subnets
        ├── Artifactory
        └── Persistence
```

---

## 40. Security Groups

AWS Security Groups should permit only required traffic.

Example conceptual rules:

```text
ALB
 ↓
443 from approved clients

Artifactory
 ↓
required application port from ALB

Database
 ↓
database port only from Artifactory
```

Avoid broad:

```text
0.0.0.0/0
```

access unless explicitly required.

---

## 41. Network ACLs

NACLs can provide subnet-level stateless filtering.

Use them as an additional layer rather than attempting to replace application-level authorization.

---

## 42. AWS DNS

Example:

```text
artifactory.company.com
        ↓
Route 53
        ↓
ALB
```

DNS should support controlled failover and disaster-recovery requirements.

---

## 43. TLS in AWS

Possible architecture:

```text
Client
 ↓ HTTPS
ALB
 ↓ HTTPS/private traffic where required
Artifactory
```

Certificates should be managed through an appropriate certificate-management process.

---

## 44. WAF

For externally reachable HTTP(S) endpoints, a WAF may help protect against common web attacks.

Conceptually:

```text
Internet
 ↓
WAF
 ↓
ALB
 ↓
Artifactory
```

Do not assume WAF replaces Artifactory authentication and authorization.

---

## 45. IAM

AWS IAM controls AWS resources.

Artifactory identity controls access to Artifactory repositories.

These are different authorization layers.

```text
AWS IAM
 → AWS resources

Artifactory RBAC
 → Artifactory resources
```

---

## 46. S3/Object Storage Concept

Depending on supported Artifactory architecture, object storage can be used for artifact persistence.

Conceptually:

```text
Artifactory
      ↓
Object Storage
```

The actual supported storage architecture must be validated against the JFrog version and deployment model.

---

## 47. EBS / File Storage Concept

Other storage mechanisms may be used depending on deployment requirements.

The design questions remain:

```text
durability
throughput
latency
sharing
backup
recovery
cost
```

---

## 48. Database Architecture

Production Artifactory should use a supported external database architecture where required by the deployment model.

Consider:

```text
availability
backup
replication
connections
latency
capacity
maintenance
```

---

## 49. Database Connection Pool

High CI concurrency can create significant database activity.

Monitor:

```text
connection usage
connection waits
query latency
database CPU
database storage
```

---

## 50. CI Concurrency Impact

Suppose:

```text
500 Jenkins jobs
```

all start simultaneously.

They may generate:

```text
dependency requests
artifact uploads
metadata requests
authentication
API requests
```

Artifactory capacity must be designed for peak load, not only average load.

---

## 51. Burst Traffic

Container scaling can cause bursts.

Example:

```text
EKS scales from
20 pods → 300 pods
```

Potentially hundreds of image pulls can occur.

Registry capacity therefore becomes part of platform scalability.

---

## 52. Kubernetes Registry Architecture

```text
                 EKS
                  |
             Image Pull
                  |
                  v
             Artifactory
                  |
          Docker/OCI Repository
                  |
                  v
               Storage
```

---

## 53. Image Pull Secrets

If authentication is required:

```text
Kubernetes
 ↓
imagePullSecret
 ↓
Artifactory
```

Credentials should be scoped appropriately.

---

## 54. Registry Authentication Failure

Typical symptoms:

```text
ImagePullBackOff
ErrImagePull
401 Unauthorized
403 Forbidden
```

Check:

```text
secret
service account
image path
token
repository permission
DNS
network
```

---

## 55. Helm Architecture

A Kubernetes platform may consume Helm charts from Artifactory:

```text
Developer / CI
      ↓
Helm Chart
      ↓
Artifactory Helm Repository
      ↓
Argo CD / Helm
      ↓
Kubernetes
```

---

## 56. Maven Architecture

```text
Developer / CI
       ↓
Maven
       ↓
maven-virtual
       |
 +-----+------+
 |            |
Local        Remote
 |            |
Internal     Upstream
Artifacts    Cache
```

---

## 57. NPM Architecture

```text
npm client
    ↓
npm-virtual
    |
 +--+----------------+
 |                   |
npm-local       npm-remote
                    |
                 Registry
```

---

## 58. PyPI Architecture

```text
pip
 ↓
pypi-virtual
 ↓
local + remote
 ↓
approved packages
```

---

## 59. Generic Binary Repository

For files that do not fit a package manager:

```text
CI
 ↓
Generic Repository
 ↓
ZIP / TAR / installer / binary
```

Use a clear naming and lifecycle policy.

---

## 60. Repository Security Boundary

Different repositories can represent different trust levels.

Example:

```text
public-proxy
internal-release
production-release
```

Permissions should prevent unauthorized promotion or publication.

---

## 61. Development vs Production Access

Example:

```text
Developer
  ↓
read virtual
deploy dev artifacts

Release Service
  ↓
promote approved artifacts

Production Service
  ↓
read production artifacts
```

Avoid giving developers unnecessary production administration rights.

---

## 62. Service Accounts

CI/CD should use dedicated service identities.

Example:

```text
jenkins-artifactory-publisher
github-actions-artifactory-publisher
gitlab-artifactory-publisher
```

These should not be personal accounts.

---

## 63. Token Scope

A CI token should ideally be limited by:

```text
identity
repository
operation
time
```

where supported.

---

## 64. Credential Rotation

Plan:

```text
create new credential
 ↓
update CI
 ↓
validate
 ↓
revoke old credential
```

Avoid changing credentials without testing dependent pipelines.

---

## 65. Identity Provider Failure

If Artifactory depends on an external identity provider:

```text
IdP outage
 ↓
authentication problems
 ↓
CI/developer access failures
```

Production design should understand the identity dependency and appropriate recovery options.

---

## 66. Observability Architecture

A production Artifactory platform should expose:

```text
metrics
logs
audit events
health information
alerts
```

Conceptually:

```text
Artifactory
   |
   +--> Metrics → Monitoring
   |
   +--> Logs → Central Logging
   |
   +--> Audit → Security / SIEM
```

---

## 67. Metrics to Watch

Examples:

```text
request rate
response latency
HTTP error rate
storage usage
CPU
memory
database health
repository traffic
authentication failures
```

---

## 68. Logging

Centralize important logs where practical.

Search for:

```text
401
403
404
5xx
timeout
connection refused
storage errors
database errors
authentication failures
```

---

## 69. Audit Logging

Audit information can help answer:

```text
Who uploaded?
Who deleted?
Who changed configuration?
Who accessed the repository?
When?
```

Use it as part of security operations.

---

## 70. Alerting

Useful alerts include:

```text
Artifactory unavailable
storage nearing capacity
high error rate
high latency
database unhealthy
authentication failures
node unhealthy
backup failure
DR replication issue
```

---

## 71. Backup Architecture

Conceptually:

```text
Artifactory
    |
    +---- Artifact Storage Backup
    |
    +---- Database Backup
    |
    +---- Configuration Backup
```

Backup copies should be protected from accidental deletion and corruption.

---

## 72. Restore Testing

A backup that has never been restored is not fully validated.

Test:

```text
backup
 ↓
restore
 ↓
verify metadata
 ↓
verify artifacts
 ↓
verify authentication/configuration
 ↓
test client access
```

---

## 73. Disaster Recovery Architecture

A conceptual DR model:

```text
Primary Region
      |
      | replication / backup
      v
DR Region
```

DR design depends on product support, data characteristics and business requirements.

---

## 74. RPO/RTO Example

Business requirement:

```text
RPO = 15 minutes
RTO = 1 hour
```

The architecture must be capable of meeting those targets under realistic failure conditions.

Do not declare RPO/RTO values without testing.

---

## 75. Regional Failure

If an AWS region becomes unavailable:

```text
Primary Artifactory
       X
       |
       v
DR Artifactory
```

The recovery process may require:

```text
DNS failover
credential recovery
artifact availability
metadata recovery
client configuration
```

---

## 76. Backup vs Replication

Backup:

```text
recover historical state
```

Replication:

```text
maintain another copy / target
```

They are complementary, not automatically interchangeable.

---

## 77. Upgrade Architecture

Before upgrading:

```text
backup
 ↓
review compatibility
 ↓
test
 ↓
maintenance plan
 ↓
upgrade
 ↓
validate
 ↓
monitor
```

Never treat production upgrades as an untested package installation.

---

## 78. Version Compatibility

Check compatibility between:

```text
Artifactory
database
storage
reverse proxy
Jenkins plugins
CI clients
Docker clients
Helm clients
Kubernetes
identity provider
```

Exact supported combinations should come from current vendor documentation.

---

## 79. Performance Architecture

Performance depends on:

```text
CPU
memory
storage I/O
database
network
artifact size
request rate
repository design
concurrency
```

Do not troubleshoot performance by changing random settings.

Measure first.

---

## 80. Large Artifact Uploads

Large uploads can stress:

```text
client
proxy
load balancer
network
Artifactory
storage
```

Check:

```text
timeouts
body-size limits
network throughput
storage latency
```

---

## 81. Large Container Images

Poorly optimized container images increase:

```text
storage
network transfer
pull time
startup time
registry load
```

Use small, secure production images.

---

## 82. Repository Cache Strategy

Remote caches should be managed intentionally.

Consider:

```text
cache size
retention
upstream availability
security
refresh behavior
```

---

## 83. Upstream Dependency Risk

Remote repositories create a dependency on external systems.

Risk:

```text
Upstream outage
Upstream compromise
Malicious package
Rate limiting
Network failure
```

Mitigate through:

```text
approved upstreams
caching
scanning
policy
provenance
```

---

## 84. Supply-Chain Trust Boundary

```text
External Internet
        ↓
Approved Remote Repository
        ↓
Security Controls
        ↓
Internal Consumers
```

Do not allow uncontrolled package sources into production pipelines.

---

## 85. Production Architecture With Security

```text
                 Users / CI
                     |
                    DNS
                     |
                    WAF
                     |
                Load Balancer
                     |
                 TLS / Proxy
                     |
             Private Artifactory
               /           \
              /             \
       Repository         Repository
           |                   |
       Metadata             Storage
              \             /
               \           /
                Database
```

---

## 86. Zero-Trust Principle

Do not assume:

```text
internal network = trusted
```

Instead verify:

```text
identity
device/workload
network
permission
request
```

where practical.

---

## 87. Least Privilege Architecture

Example:

```text
Developer
  ├── read
  └── publish development artifacts

CI Publisher
  └── deploy to designated local repository

Release Automation
  └── promotion permissions

Production Runtime
  └── read-only
```

This reduces blast radius.

---

## 88. Blast Radius

If a CI token is compromised, its permissions should limit the damage.

Bad:

```text
CI token
 ↓
global admin
```

Better:

```text
CI token
 ↓
specific repositories
 ↓
specific operations
```

---

## 89. Production Incident: Artifactory 5xx

Initial investigation:

```text
1. Confirm scope.
2. Check load balancer health.
3. Check Artifactory node health.
4. Check database.
5. Check storage.
6. Check CPU/memory.
7. Check logs.
8. Check recent changes.
9. Determine CI/runtime impact.
10. Restore service and validate.
```

---

## 90. Production Incident: High Latency

Investigate:

```text
request rate
CPU
memory
storage latency
database latency
network
large artifacts
upstream dependency
recent configuration changes
```

Use measurements instead of assumptions.

---

## 91. Production Incident: Storage 95%

Immediate actions:

```text
protect service availability
identify growth source
review retention
remove only approved disposable data
increase capacity if necessary
```

Follow-up:

```text
capacity model
cleanup automation
alerting
ownership
```

---

## 92. Production Incident: Database Unhealthy

Check:

```text
database availability
connections
CPU
memory
storage
locks
latency
recent changes
```

Artifactory application-node restarts may not fix a database problem.

---

## 93. Production Incident: All CI Pipelines Fail

Do not debug every pipeline independently first.

Check the shared dependency:

```text
CI
 ↓
Artifactory
```

Look for:

```text
DNS
TLS
authentication
Artifactory health
repository availability
network
```

---

## 94. Production Incident: Kubernetes Pods Cannot Start

If events show:

```text
ImagePullBackOff
```

check the registry path:

```text
Kubernetes
 ↓
DNS
 ↓
Network
 ↓
Authentication
 ↓
Artifactory
 ↓
Repository
 ↓
Image
```

---

## 95. Architecture Review Checklist

```text
[ ] Stable DNS
[ ] TLS
[ ] Load balancing
[ ] Network segmentation
[ ] Supported Artifactory topology
[ ] Repository strategy
[ ] Storage capacity
[ ] Database capacity
[ ] HA requirements
[ ] Monitoring
[ ] Logging
[ ] Audit
[ ] Backup
[ ] Restore test
[ ] DR
[ ] RPO/RTO
[ ] Security
[ ] Least privilege
[ ] CI/CD integration
```

---

## 96. Design Question: Single Node or HA?

Choose based on:

```text
business criticality
CI/CD dependency
runtime dependency
traffic
availability requirements
budget
operational maturity
```

Do not add HA merely because it sounds more production-like; design for an actual availability requirement.

---

## 97. Design Question: Public or Private?

Prefer private access when business requirements allow.

Example:

```text
Corporate Network
       ↓
VPN / Direct Connect
       ↓
Private Artifactory
```

External access should use strong controls when required.

---

## 98. Design Question: One Repository or Many?

Prefer a small, well-governed repository model.

Create separate repositories when there is a meaningful difference in:

```text
package type
security
lifecycle
ownership
retention
promotion
```

---

## 99. Design Question: One Artifactory or Multiple?

Consider:

```text
organization size
regions
business units
compliance boundaries
network isolation
availability
data residency
```

Multiple instances add operational complexity.

---

## 100. Design Question: Where Should Artifactory Run?

Evaluate:

```text
managed service
self-hosted VM
Kubernetes
AWS infrastructure
hybrid
```

Choose the supported model that satisfies:

```text
availability
security
cost
operability
performance
DR
```

---

## 101. Running Artifactory on Kubernetes

If deploying Artifactory on Kubernetes, evaluate:

```text
persistent storage
database
ingress
TLS
resource sizing
pod disruption
backup
upgrade strategy
networking
```

Do not assume Kubernetes automatically solves stateful application HA.

---

## 102. Artifactory on EKS

Conceptually:

```text
Route 53
   ↓
ALB / Ingress
   ↓
EKS
   ↓
Artifactory
   ↓
Persistent storage / database
```

Follow the supported JFrog architecture for the chosen version.

---

## 103. EKS Failure Domains

Spread components across:

```text
Availability Zones
```

where the supported architecture allows it.

Consider:

```text
AZ failure
node failure
storage failure
network failure
```

---

## 104. Availability Zone Strategy

Example:

```text
Region
├── AZ-A
│    └── Artifactory workload
├── AZ-B
│    └── Artifactory workload
└── AZ-C
     └── supporting infrastructure
```

Exact placement depends on the supported architecture.

---

## 105. Network Egress

Remote repositories may need outbound access to approved upstreams.

Example:

```text
Artifactory
 ↓
NAT Gateway / Egress
 ↓
Internet
 ↓
Approved upstream
```

Control outbound access where possible.

---

## 106. Network Ingress

Consumers need inbound access:

```text
CI
 ↓
ALB
 ↓
Artifactory
```

Restrict source networks where business requirements allow.

---

## 107. Proxy Timeouts

Large uploads/downloads may fail if intermediary components have aggressive timeouts.

Check:

```text
client timeout
proxy timeout
load balancer timeout
Artifactory timeout
```

---

## 108. HTTP Status Troubleshooting Matrix

| Status | Typical Direction |
|---|---|
| 200 | Successful request |
| 201 | Successful creation/upload |
| 401 | Authentication problem |
| 403 | Authorization/policy problem |
| 404 | Resource/repository/path problem |
| 409 | Conflict in operation |
| 429 | Rate/traffic limiting |
| 5xx | Server-side/service problem |

Always inspect application-specific response details.

---

## 109. Architecture Anti-Pattern: Public Admin

Bad:

```text
Internet
 ↓
Artifactory Admin
```

Better:

```text
Admin Network
 ↓
Secure Access
 ↓
Artifactory Administration
```

---

## 110. Architecture Anti-Pattern: Shared Personal Credentials

Bad:

```text
Everyone
 ↓
admin@company
```

Better:

```text
Individual users
 ↓
SSO/RBAC

CI
 ↓
Dedicated service identity
```

---

## 111. Architecture Anti-Pattern: Mutable Production Tags

Bad:

```text
production → latest
```

Better:

```text
production → 4.8.2
```

or an immutable digest where applicable.

---

## 112. Architecture Anti-Pattern: Rebuild for Production

Bad:

```text
Staging build
 ↓
Production rebuild
```

Better:

```text
Build once
 ↓
Test
 ↓
Promote
 ↓
Production
```

---

## 113. Architecture Anti-Pattern: No Restore Test

Bad:

```text
Backup configured
```

but never restored.

Better:

```text
Backup
 ↓
Scheduled restore test
 ↓
Validation
```

---

## 114. Architecture Anti-Pattern: No Capacity Model

Bad:

```text
Storage is almost full
```

Better:

```text
Growth rate
 ↓
Forecast
 ↓
Alert
 ↓
Capacity action
```

---

## 115. Architecture Anti-Pattern: Uncontrolled Remote Repositories

Bad:

```text
Any developer
 ↓
Any public registry
```

Better:

```text
Approved upstream
 ↓
Artifactory Remote
 ↓
Security policy
 ↓
Internal consumers
```

---

## 116. Architecture Anti-Pattern: Artifactory as Secret Store

Bad:

```text
Artifact Repository
 ↓
Production passwords
```

Better:

```text
Secret Manager
 ↓
Runtime secrets
```

---

## 117. Architecture Anti-Pattern: Ignoring DNS

A healthy Artifactory node is irrelevant if:

```text
artifactory.company.com
```

does not resolve correctly.

Always include DNS in the architecture.

---

## 118. Architecture Anti-Pattern: Ignoring Identity Dependency

If all access requires an external identity provider, an IdP outage can become an Artifactory access outage.

Document the dependency and recovery process.

---

## 119. Architecture Anti-Pattern: Ignoring Runtime Pull Traffic

A registry may work during CI but fail under Kubernetes scale-out because runtime pull traffic is much higher.

Capacity-plan for:

```text
normal traffic
deployment bursts
autoscaling bursts
incident recovery
```

---

## 120. Reference Production Architecture

```text
                     Users / CI/CD
                           |
                         Route53
                           |
                          WAF
                           |
                          ALB
                           |
                    Private Network
                           |
             +-------------+-------------+
             |                           |
             v                           v
       Artifactory-1              Artifactory-2
             |                           |
             +-------------+-------------+
                           |
                Supported Persistence
                    /              \
                   /                \
             Database          Artifact Storage
                   \                /
                    \              /
                     +------------+
                           |
                    Backup / DR
```

This is a conceptual reference architecture, not a copy-paste deployment diagram.

---

## 121. Production Design Principles

```text
1. Keep Artifactory behind controlled network boundaries.
2. Use TLS everywhere appropriate.
3. Use supported HA architecture for required availability.
4. Separate application, storage and database failure domains.
5. Design repositories around real governance boundaries.
6. Use virtual repositories for consumer simplicity.
7. Protect production repositories with least privilege.
8. Plan storage before onboarding large image/package workloads.
9. Monitor peak CI and Kubernetes pull traffic.
10. Test backup restoration.
11. Define and test DR.
12. Document RPO/RTO.
13. Treat upstream dependencies as external failure domains.
14. Keep immutable production artifacts.
15. Use dedicated CI identities.
16. Centralize logs and audit events.
17. Validate architecture against the exact JFrog release and license.
```

---

## 122. Interview Question — Explain Artifactory Architecture

### Strong Answer

```text
I think about Artifactory in layers. Clients such as Jenkins,
GitHub Actions, package managers and Kubernetes access a stable DNS
endpoint, normally through a load balancer or reverse-proxy layer.
Behind that are the Artifactory services and logical repositories:
local repositories for internal artifacts, remote repositories for
external dependencies and virtual repositories for a unified consumer
endpoint.

The platform also depends on persistent metadata and artifact
storage, so production architecture cannot focus only on the
application nodes. I consider storage, database or persistence,
networking, TLS, identity, monitoring, backup and disaster recovery as
part of the complete system.

For high availability, I use a supported multi-node architecture
where required and make sure the persistence layer and storage do not
remain single points of failure.
```

---

## 123. Interview Question — Why Is the Load Balancer Important?

### Answer

```text
It provides a stable endpoint, distributes traffic across healthy
nodes and can remove unhealthy nodes from service. It also gives us a
central point for TLS and traffic-management controls when the chosen
architecture supports them.
```

---

## 124. Interview Question — Does Multiple Artifactory Nodes Automatically Mean HA?

### Answer

```text
No. HA is an end-to-end architecture. Application nodes, persistent
state, storage, database, networking and supported configuration all
need to satisfy the product's HA requirements. I would always verify
the exact JFrog-supported topology for the deployed version and
edition.
```

---

## 125. Interview Question — What Happens If One Artifactory Node Fails?

### Answer

```text
In a properly designed HA deployment, the load balancer detects the
unhealthy node and routes traffic to healthy nodes. Because the
persistent state and artifact storage are designed for the HA
architecture, a single application-node failure should not require a
complete repository outage.
```

---

## 126. Interview Question — Is HA Enough for Disaster Recovery?

### Answer

```text
No. HA protects primarily against failures within the designed
availability domain. Disaster recovery addresses larger failures such
as regional or major infrastructure loss. I still need backups,
replication where supported, recovery procedures and tested RPO/RTO.
```

---

## 127. Interview Question — What Are Your Main Artifactory Capacity Metrics?

### Answer

```text
I monitor artifact storage growth, request rate, upload/download
traffic, response latency, HTTP error rate, CPU, memory, storage I/O,
database health and CI/runtime burst patterns. I also forecast
capacity based on artifact growth rather than waiting for storage to
become critical.
```

---

## 128. Interview Question — How Would You Design Artifactory on AWS?

### Answer

```text
I would normally place the service in a controlled VPC architecture,
use Route 53 for DNS and an appropriate load-balancing layer, keep
application components in private subnets where possible, restrict
security-group access, use TLS, provide supported persistent storage
and database architecture, and integrate monitoring, logging,
backups and DR.

The exact implementation depends on whether we use a JFrog-managed
service or self-managed Artifactory, so I would validate the
supported architecture before selecting the AWS storage and
deployment components.
```

---

## 129. Interview Question — How Would You Troubleshoot an Artifactory 5xx?

### Answer

```text
I first determine whether the failure is global or repository
specific. Then I check the load balancer and node health, Artifactory
logs, CPU and memory, database health, storage health and recent
changes. I also verify DNS, network and TLS if the failures are
client-facing. I correlate the timestamp with CI failures and
validate recovery using a controlled client request.
```

---

## 130. Interview Question — How Would You Handle Artifactory Storage at 95%?

### Answer

```text
I would first protect service availability and identify what is
causing growth. I would review repository usage, caches, snapshots
and retention policies, remove only approved disposable content,
increase capacity if necessary and then implement a longer-term
capacity model and alerting improvements.
```

---

## 131. Interview Question — How Does Artifactory Affect Kubernetes Availability?

### Answer

```text
Kubernetes nodes may need to pull images during deployment or
autoscaling. If the registry is unavailable, authentication fails or
the required image is missing, pods can enter ImagePullBackOff or
ErrImagePull. Therefore registry availability, DNS, network,
authentication, storage and capacity are part of the Kubernetes
platform's operational reliability.
```

---

## 132. Interview Question — What Is the Most Important Architectural Principle?

### Answer

```text
I treat Artifactory as a critical software-supply-chain dependency,
not simply as binary storage. The architecture must protect artifact
integrity and availability from the client layer through networking,
Artifactory services, persistent storage, database, identity,
monitoring, backup and disaster recovery.
```

---

## 133. Final Architecture Checklist

```text
NETWORK
[ ] DNS
[ ] TLS
[ ] Load Balancer
[ ] WAF where required
[ ] Private networking
[ ] Security Groups / firewalls
[ ] Controlled egress

ARTIFACTORY
[ ] Supported version
[ ] Supported topology
[ ] Repository strategy
[ ] Virtual repositories
[ ] RBAC
[ ] Service identities
[ ] Build Info
[ ] Retention

PERSISTENCE
[ ] Artifact storage
[ ] Metadata/database
[ ] Capacity planning
[ ] I/O monitoring
[ ] Backup
[ ] Restore testing

HA
[ ] Multiple nodes where required
[ ] Health checks
[ ] Load balancing
[ ] Storage resilience
[ ] Database resilience
[ ] Failure-domain analysis

DR
[ ] RPO
[ ] RTO
[ ] Backup
[ ] Replication where supported
[ ] Recovery procedure
[ ] DNS failover
[ ] DR testing

OBSERVABILITY
[ ] Metrics
[ ] Logs
[ ] Audit
[ ] Alerts
[ ] Capacity dashboards
[ ] CI/CD failure correlation

SECURITY
[ ] Least privilege
[ ] Scoped tokens
[ ] Admin isolation
[ ] Approved upstreams
[ ] Artifact scanning
[ ] Immutable releases
```

---