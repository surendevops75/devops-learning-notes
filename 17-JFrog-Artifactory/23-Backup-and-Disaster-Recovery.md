# Backup-and-Disaster-Recovery

## 1. Purpose

This file provides a production-oriented guide to backup, restore,
disaster recovery, business continuity, and recovery testing for JFrog
Artifactory.

The objective is to protect the complete artifact platform:

```text
Artifactory
   |
   +--> Configuration
   +--> Database
   +--> Artifact Metadata
   +--> Filestore
   +--> Security Configuration
   +--> Certificates / Keys
   +--> Repository Configuration
   |
   v
Backup
   |
   v
Protected Backup Storage
   |
   v
DR Environment
   |
   v
Restore / Recovery
   |
   v
Production Service
```

This file covers:

- Backup fundamentals
- Backup vs HA vs DR
- RTO
- RPO
- backup architecture
- database backup
- filestore backup
- configuration backup
- metadata
- object storage
- snapshots
- replication
- cross-region DR
- AWS architecture
- EKS
- S3
- database recovery
- restore procedures
- recovery validation
- ransomware protection
- backup security
- retention
- immutability
- encryption
- monitoring
- alerting
- DR drills
- failure scenarios
- migration considerations
- production architecture
- troubleshooting
- interview preparation
- production checklists

---

# PART I — FUNDAMENTALS

## 2. What Is Backup?

Backup is a recoverable copy of data or configuration stored separately
from the primary production system.

Example:

```text
Production
    |
    v
Backup
```

---

## 3. What Is Disaster Recovery?

Disaster Recovery (DR) is the process and architecture used to restore
service after a major failure.

Example:

```text
Primary Region
      |
      X
      |
      v
DR Region
```

---

## 4. Backup vs DR

Backup:

```text
Recover data
```

DR:

```text
Recover service
```

A DR environment often depends on backups or replicated data.

---

## 5. High Availability vs Backup

HA:

```text
Node failure
 ↓
another node serves traffic
```

Backup:

```text
Data loss
 ↓
restore
```

---

## 6. High Availability vs DR

HA generally addresses failures inside the primary operating
environment.

DR addresses larger events:

```text
region loss
site loss
major corruption
ransomware
catastrophic infrastructure failure
```

---

# PART II — RTO AND RPO

## 7. Recovery Time Objective

RTO defines the maximum acceptable time to restore service.

Example:

```text
RTO = 30 minutes
```

---

## 8. Recovery Point Objective

RPO defines the maximum acceptable amount of data loss measured in
time.

Example:

```text
RPO = 5 minutes
```

---

## 9. Example Requirement

Suppose the business requires:

```text
RTO = 1 hour
RPO = 15 minutes
```

The backup and DR architecture must realistically achieve those
targets.

Do not document an RTO/RPO that has never been tested.

---

# PART III — WHAT MUST BE PROTECTED

## 10. Artifactory Data Categories

A recovery design may need to protect:

```text
repository configuration
artifact metadata
binary artifacts
database
security configuration
users/groups
permissions
projects
system configuration
certificates
keys
integration settings
```

The exact backup scope must follow the supported JFrog architecture
for the deployed version.

---

## 11. Database

The database contains important metadata and configuration.

Concept:

```text
Artifactory
   |
   v
Database
```

Protect it with a supported backup mechanism.

---

## 12. Filestore

The filestore contains artifact binaries.

Examples:

```text
JAR
WAR
Docker layers
NPM packages
Python packages
Helm artifacts
generic binaries
```

---

## 13. Configuration

Configuration may include:

```text
repository definitions
security settings
system settings
integration settings
```

Back up configuration according to the supported JFrog process.

---

# PART IV — BACKUP ARCHITECTURE

## 14. Basic Backup

```text
Artifactory
   |
   +--> Database Backup
   |
   +--> Filestore Backup
   |
   +--> Configuration Backup
```

---

## 15. Protected Backup

Better:

```text
Production
   |
   v
Backup System
   |
   v
Separate Backup Account
   |
   v
Immutable Storage
```

---

## 16. Why Separate Accounts?

If an attacker compromises production credentials, they should not
automatically be able to:

```text
delete all backups
```

---

# PART V — 3-2-1 BACKUP PRINCIPLE

## 17. 3-2-1 Concept

A common resilience principle is:

```text
3 copies
2 different storage/media locations
1 offsite copy
```

Modern cloud environments may implement the same resilience objective
using logically separated storage, accounts and regions.

---

## 18. Example

```text
Primary
   |
   +--> Backup Vault A
   |
   +--> Backup Vault B
   |
   +--> DR Region
```

---

# PART VI — BACKUP TYPES

## 19. Full Backup

Copies the required protected dataset.

Benefits:

```text
simple recovery
```

Cost:

```text
storage
time
```

---

## 20. Incremental Backup

Copies data changed since an earlier backup.

Benefits:

```text
smaller backup
faster backup
```

Recovery may require a chain of backups depending on implementation.

---

## 21. Snapshot

A snapshot captures storage state at a point in time.

Examples:

```text
database snapshot
volume snapshot
object storage version
```

A snapshot is not automatically equivalent to a complete DR strategy.

---

# PART VII — DATABASE BACKUP

## 22. Database Recovery

The database must be recoverable to a known consistent state.

---

## 23. Database Backup Frequency

Determine frequency from:

```text
RPO
database workload
backup capability
storage cost
recovery design
```

---

## 24. Point-in-Time Recovery

If supported by the selected database architecture, point-in-time
recovery can reduce data loss.

Concept:

```text
Base Backup
    +
Transaction Logs
    |
    v
Target Recovery Time
```

---

## 25. Database Backup Validation

Do not assume:

```text
backup job = recoverable backup
```

Test restore.

---

# PART VIII — FILESTORE BACKUP

## 26. Binary Artifacts

The filestore contains the actual artifacts.

Therefore:

```text
database backup
```

without:

```text
filestore backup
```

may be insufficient for complete recovery.

---

## 27. Consistency

Database metadata and binary storage must be recoverable consistently.

Follow JFrog-supported backup procedures for the exact architecture.

---

# PART IX — OBJECT STORAGE

## 28. Object Storage

Cloud deployments may use object storage for artifact binaries.

Concept:

```text
Artifactory
    |
    v
Object Storage
```

---

## 29. Object Storage Protection

Use controls such as:

```text
versioning
encryption
access policies
retention
replication
immutability
```

where supported and appropriate.

---

# PART X — AWS BACKUP ARCHITECTURE

## 30. Example AWS Design

```text
                         AWS Primary
                              |
                         Artifactory
                        /           \
                       /             \
                 Database          S3/Object
                     |                |
                     +-------+--------+
                             |
                           Backup
                             |
                             v
                      Backup Account
                             |
                             v
                       DR Region
```

Exact implementation depends on Artifactory architecture and AWS
services selected.

---

# PART XI — CROSS-REGION DR

## 31. Why Cross-Region?

Protects against:

```text
regional outage
major infrastructure failure
regional disaster
```

---

## 32. Example

```text
ap-south-1
    |
    v
Backup / Replication
    |
    v
ap-southeast-1
```

The exact region selection should follow business requirements,
latency, compliance and supported architecture.

---

## 33. DR Endpoint

Production clients should use a stable endpoint:

```text
artifactory.company.com
```

DR can be exposed through:

```text
DNS
load balancer
routing strategy
```

---

# PART XII — REPLICATION

## 34. Replication vs Backup

Replication:

```text
Production
   |
   v
Secondary
```

Backup:

```text
Production
   |
   v
Historical recovery copy
```

Replication alone may replicate corruption or accidental deletion.

---

## 35. Use Both

Strong design:

```text
Replication
+
Backup
+
Immutable retention
```

---

# PART XIII — IMMUTABLE BACKUPS

## 36. Why Immutability?

Ransomware may attempt:

```text
encrypt production
 ↓
delete backups
```

Immutable backups can provide recovery protection.

---

## 37. Backup Locking

Use an approved immutable/WORM-style storage mechanism where supported.

---

# PART XIV — ENCRYPTION

## 38. Encryption at Rest

Protect:

```text
database
filestore
backup
```

using appropriate encryption.

---

## 39. Encryption in Transit

Use:

```text
TLS
```

for:

```text
client → Artifactory
Artifactory → dependencies
Artifactory → database
backup transfers
```

where applicable.

---

# PART XV — BACKUP ACCESS CONTROL

## 40. Least Privilege

Backup identities should have only required permissions.

---

## 41. Backup Delete Protection

Avoid giving production application identities the ability to delete
backup data.

---

## 42. Separate Backup Roles

Example:

```text
Production Admin
    X
Backup Delete

Backup Operator
    |
    v
Backup Management
```

Use separation of duties according to policy.

---

# PART XVI — RETENTION

## 43. Retention Policy

Define:

```text
daily
weekly
monthly
annual
```

according to business requirements.

---

## 44. Example

```text
Daily:
14 days

Weekly:
8 weeks

Monthly:
12 months
```

This is an example only. Actual retention must come from compliance
and business requirements.

---

## 45. Artifact Retention vs Backup Retention

Do not confuse:

```text
artifact lifecycle
```

with:

```text
backup lifecycle
```

An artifact may be deleted from production storage while still being
needed in backup history.

---

# PART XVII — BACKUP MONITORING

## 46. Monitor Backup Jobs

Track:

```text
success
failure
duration
size
growth
last successful backup
```

---

## 47. Alert

Critical alert:

```text
No successful backup in required window
```

---

## 48. Backup Age

Monitor:

```text
last successful backup timestamp
```

rather than simply:

```text
backup job ran
```

---

# PART XVIII — RESTORE TESTING

## 49. Why Restore Tests Matter

A backup is only valuable if it can be restored.

---

## 50. Restore Test

```text
Backup
 ↓
Restore
 ↓
Start Artifactory
 ↓
Authenticate
 ↓
List repositories
 ↓
Download artifact
 ↓
Push test artifact if appropriate
 ↓
Validate
```

---

## 51. Digest Validation

For container images:

```text
Original digest
        =
Restored digest
```

This verifies artifact integrity.

---

# PART XIX — DR DRILL

## 52. DR Exercise

A controlled drill can simulate:

```text
Primary unavailable
```

Then:

```text
Activate DR
 ↓
Restore/replicate data
 ↓
Start services
 ↓
Update endpoint
 ↓
Test CI
 ↓
Test Kubernetes pull
 ↓
Validate artifacts
```

---

## 53. Measure

Record:

```text
actual RTO
actual RPO
failures
manual steps
missing dependencies
```

---

# PART XX — RANSOMWARE

## 54. Ransomware Scenario

```text
Attacker
 ↓
Production compromise
 ↓
Artifact deletion/encryption
 ↓
Backup attack
```

---

## 55. Protection

Use:

```text
separate backup account
immutable backups
least privilege
MFA
audit
offline/isolated copies where appropriate
```

---

# PART XXI — ACCIDENTAL DELETION

## 56. Scenario

An operator deletes:

```text
production artifact
```

Recovery:

```text
Identify artifact
 ↓
Find backup
 ↓
Restore
 ↓
Validate checksum/digest
 ↓
Return artifact
```

---

# PART XXII — DATABASE CORRUPTION

## 57. Scenario

Database corruption occurs.

Response:

```text
Stop affected operations
 ↓
Assess corruption
 ↓
Select recovery point
 ↓
Restore database
 ↓
Restore/validate filestore consistency
 ↓
Start Artifactory
 ↓
Validate repositories
```

Follow the supported recovery procedure for the deployed architecture.

---

# PART XXIII — REGION LOSS

## 58. Scenario

Primary region is unavailable.

```text
Region A
   X

Region B
   |
   v
DR Artifactory
```

---

## 59. Recovery

```text
Declare DR
 ↓
Validate DR data
 ↓
Activate services
 ↓
Update DNS/routing
 ↓
Validate CI
 ↓
Validate Kubernetes
 ↓
Monitor
```

---

# PART XXIV — EKS DR

## 60. Kubernetes Recovery

If Artifactory is consumed by EKS:

```text
DR EKS
  |
  v
DR Artifactory
```

The cluster must have:

```text
network
DNS
TLS
credentials
image access
```

---

## 61. GitOps

GitOps can help recreate desired application state:

```text
Git
 ↓
Argo CD
 ↓
DR EKS
 ↓
DR Artifactory
```

---

# PART XXV — CREDENTIAL RECOVERY

## 62. Credentials

DR must account for:

```text
Artifactory credentials
CI credentials
Kubernetes pull credentials
certificates
keys
federated identity
```

---

## 63. Secret Dependencies

A DR system is not ready if:

```text
Artifactory restored
```

but:

```text
CI cannot authenticate
```

or:

```text
Kubernetes cannot pull images
```

---

# PART XXVI — DNS FAILOVER

## 64. Stable Endpoint

Applications should use:

```text
artifactory.company.com
```

rather than hardcoded infrastructure addresses.

---

## 65. Failover

Concept:

```text
DNS
 |
 +--> Primary
 |
 X
 |
 +--> DR
```

---

## 66. TTL

DNS TTL affects failover speed.

However, cached DNS records may continue to be used until their TTL
expires.

Test actual client behavior.

---

# PART XXVII — LOAD BALANCER DR

## 67. DR Load Balancer

The DR environment needs a reachable load-balancing endpoint.

---

## 68. Health Checks

Before declaring DR ready:

```text
LB
 ↓
Artifactory
 ↓
Database
 ↓
Storage
```

must be validated.

---

# PART XXVIII — BACKUP SECURITY

## 69. Backup Encryption

Use encryption at rest.

---

## 70. Backup Credentials

Protect:

```text
backup keys
access credentials
KMS permissions
```

---

## 71. Backup Audit

Audit:

```text
backup creation
restore
delete
configuration changes
access
```

---

# PART XXIX — COMPLIANCE

## 72. Compliance Requirements

May require:

```text
retention
immutability
encryption
access control
audit
geographic restrictions
```

---

## 73. Evidence

Keep evidence of:

```text
successful backups
restore tests
DR drills
access reviews
retention policy
```

---

# PART XXX — BACKUP PERFORMANCE

## 74. Backup Window

Large artifact repositories can require substantial time to back up.

Measure:

```text
backup size
throughput
duration
network
storage performance
```

---

## 75. Backup Impact

Do not allow backup workloads to degrade:

```text
production artifact downloads
CI builds
Kubernetes image pulls
```

---

# PART XXXI — STORAGE GROWTH

## 76. Growth Model

Track:

```text
daily artifact growth
monthly growth
repository distribution
retention deletion
backup growth
```

---

## 77. Forecast

Example:

```text
Current = 10 TB
Growth = 200 GB/day
```

Calculate future capacity requirements.

---

# PART XXXII — BACKUP COST

## 78. Cost Factors

Consider:

```text
storage
cross-region transfer
backup operations
snapshot costs
replication
restore operations
```

---

## 79. Optimize Carefully

Do not reduce retention below business requirements just to save cost.

---

# PART XXXIII — RECOVERY RUNBOOK

## 80. Runbook Structure

A production runbook should contain:

```text
1. Incident declaration
2. Decision authority
3. Recovery environment
4. Credentials
5. DNS
6. Database
7. Filestore
8. Artifactory
9. Load balancer
10. Kubernetes
11. CI/CD
12. Validation
13. Communication
14. Return to primary
```

---

# PART XXXIV — RECOVERY VALIDATION

## 81. Basic Validation

Check:

```bash
curl -I https://artifactory.company.com/
```

---

## 82. Repository Validation

Check:

```text
repositories available
permissions work
artifact metadata present
```

---

## 83. Artifact Validation

Download known artifacts:

```text
JAR
Docker image
Helm chart
NPM package
```

---

## 84. Kubernetes Validation

Test:

```text
new Pod
image pull
application startup
health check
```

---

# PART XXXV — RETURN TO PRIMARY

## 85. Failback

After primary recovery:

```text
DR
 ↓
validate primary
 ↓
synchronize data
 ↓
test
 ↓
switch traffic
 ↓
monitor
```

---

## 86. Do Not Rush Failback

The primary must be:

```text
healthy
synchronized
tested
secure
```

before traffic returns.

---

# PART XXXVI — PRODUCTION ARCHITECTURE

## 87. Enterprise DR Model

```text
                         Global DNS
                            |
              +-------------+-------------+
              |                           |
              v                           v
         Primary Region              DR Region
              |                           |
         LB + Artifactory             LB + Artifactory
              |                           |
          DB + Storage                 DB + Storage
              |                           |
              +------------+--------------+
                           |
                       Backup System
                           |
                           v
                    Immutable Storage
```

---

# PART XXXVII — TROUBLESHOOTING

## 88. Backup Job Failed

Check:

```text
credentials
storage capacity
network
database state
backup service
permissions
```

---

## 89. Backup Succeeds but Restore Fails

Investigate:

```text
backup consistency
missing dependencies
unsupported restore procedure
corruption
version mismatch
```

---

## 90. DR Starts but Images Cannot Be Pulled

Check:

```text
DNS
TLS
registry endpoint
Kubernetes Secret
Artifactory READ permission
storage
network
```

---

## 91. Artifacts Missing After Restore

Check:

```text
database
filestore
backup scope
restore order
consistency
```

---

## 92. DR Is Too Slow

Measure:

```text
restore throughput
database recovery
storage recovery
DNS propagation
network
manual steps
```

Then optimize the actual bottleneck.

---

# PART XXXVIII — REAL PRODUCTION SCENARIOS

## 93. Scenario — Accidental Delete

```text
Artifact deleted
 ↓
Find backup
 ↓
Restore
 ↓
Verify digest
 ↓
Deploy/restore
```

---

## 94. Scenario — Ransomware

```text
Compromise
 ↓
Isolate
 ↓
Protect backups
 ↓
Identify clean recovery point
 ↓
Restore
 ↓
Rotate credentials
 ↓
Validate
 ↓
Resume service
```

---

## 95. Scenario — Region Outage

```text
Primary unavailable
 ↓
Declare DR
 ↓
Activate secondary
 ↓
Switch DNS
 ↓
Validate CI/Kubernetes
 ↓
Operate from DR
```

---

## 96. Scenario — Corrupt Backup

If the newest backup is corrupt:

```text
select previous known-good backup
 ↓
restore
 ↓
validate
```

This is why multiple recovery points are important.

---

# PART XXXIX — INTERVIEW PREPARATION

## 97. Why Do We Need Backup If Artifactory Is HA?

Answer:

```text
HA protects availability when components fail, but it does not
protect against accidental deletion, corruption, ransomware or major
regional failures. Backup provides recoverable historical data and
supports DR.
```

---

## 98. What Are RTO and RPO?

Answer:

```text
RTO is the target maximum time to restore service. RPO is the target
maximum acceptable data loss measured in time.
```

---

## 99. What Do You Back Up?

Answer:

```text
I protect the database, artifact filestore, repository and security
configuration and other supported Artifactory configuration required
to reconstruct the service. I design the backup scope according to
the deployed Artifactory architecture and documented recovery
procedure.
```

---

## 100. How Do You Test Backups?

Answer:

```text
I periodically restore into an isolated environment, start
Artifactory, validate repositories and permissions, download known
artifacts, validate container image digests and test representative
CI/Kubernetes workflows.
```

---

## 101. How Do You Design DR?

Answer:

```text
I start with business RTO/RPO, then design a secondary environment,
protected data copies or replication, secure credentials, DNS and
load-balancing failover, artifact validation, monitoring and a
tested recovery runbook.
```

---

## 102. What Is the Difference Between Replication and Backup?

Answer:

```text
Replication maintains a secondary copy, often for availability or
regional recovery. Backup preserves historical recovery points.
Replication can reproduce corruption or deletion, so it should not
be treated as the only backup.
```

---

## 103. How Do You Protect Against Ransomware?

Answer:

```text
I separate backup credentials and accounts, use immutable backup
storage where supported, restrict deletion, encrypt backups, protect
administrative access, monitor access and regularly test restoration
from a known-good recovery point.
```

---

# PART XL — PRODUCTION CHECKLIST

## 104. Backup

```text
[ ] database backup
[ ] filestore protection
[ ] configuration backup
[ ] backup schedule
[ ] retention
[ ] encryption
[ ] immutability
```

---

## 105. Recovery

```text
[ ] RTO
[ ] RPO
[ ] restore procedure
[ ] recovery runbook
[ ] validation tests
[ ] failback procedure
```

---

## 106. DR

```text
[ ] secondary environment
[ ] network
[ ] DNS
[ ] load balancer
[ ] database
[ ] storage
[ ] credentials
[ ] monitoring
```

---

## 107. Testing

```text
[ ] restore test
[ ] DR drill
[ ] artifact validation
[ ] Docker pull test
[ ] Kubernetes test
[ ] CI test
[ ] RTO measurement
[ ] RPO measurement
```

---

# PART XLI — GOLDEN RULES

## 108. Rules

```text
1. HA is not backup.

2. Backup is not DR.

3. Start with business RTO and RPO.

4. Protect both metadata and binary artifacts.

5. Use supported JFrog backup and recovery procedures.

6. Store backups separately from production.

7. Protect backups with separate identities.

8. Use immutable backup storage where supported.

9. Encrypt backups at rest.

10. Protect backup transfer with secure transport.

11. Monitor backup success, not merely backup job execution.

12. Test restores regularly.

13. Keep multiple recovery points.

14. Do not assume replication replaces backup.

15. Protect against accidental deletion and ransomware.

16. Document DNS and endpoint failover.

17. Include Kubernetes and CI/CD in DR testing.

18. Verify container image digests after recovery.

19. Test both failover and failback.

20. Measure actual RTO and RPO during drills.

21. Keep recovery credentials available but protected.

22. Review backup retention against compliance requirements.

23. Forecast backup storage growth.

24. Protect backup systems from production compromise.

25. Maintain a clear recovery ownership and escalation model.

26. Do not declare DR successful until representative workloads work.

27. Validate database and filestore consistency.

28. Keep the recovery runbook current.

29. Test after major Artifactory, database, storage or network changes.

30. A backup that has never been restored is an assumption, not proven
    recovery capability.
```

---