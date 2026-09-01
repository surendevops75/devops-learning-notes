# 26 — Backup and Restore

## 1. Purpose

Backup and restore is a core production reliability capability. A production DevOps platform is not considered resilient merely because workloads run across multiple Availability Zones. High availability protects against many infrastructure failures; backups and tested restores protect against data corruption, accidental deletion, ransomware, operator mistakes, failed deployments, compromised credentials, and regional disasters.

This chapter defines a production backup and restore strategy for the RoboShop-style AWS/EKS platform.

The architecture uses:

- AWS
- Amazon EKS
- Amazon ECR
- Amazon VPC
- AWS ALB
- Terraform
- Helm
- Argo CD
- Prometheus
- Grafana
- ELK
- Kubernetes
- Git/GitOps
- Production databases
- AWS backup and snapshot capabilities

The primary principle is:

> A backup is not a recovery strategy until the restore has been successfully tested.

---

# 2. Backup Fundamentals

## 2.1 What is a backup?

A backup is a recoverable copy of data or configuration stored separately from the primary workload.

Examples:

- Database snapshots
- Database point-in-time recovery logs
- EBS snapshots
- S3 object versions
- S3 replication
- EKS/Kubernetes resource backups
- Persistent volume backups
- Terraform source code
- Helm charts
- GitOps manifests
- Application configuration
- Secrets stored in an approved secrets system
- Monitoring configuration
- Logging configuration

---

## 2.2 Backup vs snapshot vs replication

These terms are related but not identical.

### Backup

A recoverable copy retained independently from the production system.

### Snapshot

A point-in-time representation of a storage resource.

Examples:

- EBS snapshot
- RDS snapshot

### Replication

Data is copied continuously or periodically to another system.

Examples:

- Cross-region database replication
- S3 Cross-Region Replication
- ECR image replication

Replication improves availability but does not automatically protect against corruption.

If a bad record is replicated everywhere, the bad record can exist everywhere.

Therefore:

> Replication improves availability; backups provide recovery points.

---

# 3. Why Production Backups Are Required

Production failures include:

- Developer deletes a database table
- Operator deletes a Kubernetes namespace
- Terraform destroys a resource accidentally
- Application writes corrupt data
- Migration damages schema
- Ransomware encrypts data
- Credentials are compromised
- EBS volume is deleted
- EKS cluster is accidentally destroyed
- AWS region becomes unavailable
- Incorrect deployment removes application state
- Log retention configuration deletes required evidence
- S3 object is overwritten
- Secret is deleted
- Bad automation changes production configuration

Without backups, the organization may have no safe recovery point.

---

# 4. Backup Objectives

A production backup design should answer:

1. What is backed up?
2. How often?
3. Where is it stored?
4. How long is it retained?
5. Is it encrypted?
6. Can production administrators delete it?
7. Is there an offsite copy?
8. Is there a cross-region copy?
9. How quickly can it be restored?
10. How is restore tested?
11. Who owns the backup?
12. How is backup failure detected?
13. What is the RPO?
14. What is the RTO?

---

# 5. RPO and Backup Frequency

RPO determines how much data loss is acceptable.

Example:

```text
RPO = 15 minutes
```

This means the organization should be able to recover to a point no more than approximately 15 minutes before the incident, depending on the technology and recovery process.

Example strategy:

| Data | Target RPO | Backup strategy |
|---|---:|---|
| Critical database | 5–15 min | Continuous logs/PITR + snapshots |
| Application PV | 1 hour | Scheduled snapshots |
| S3 objects | Near-zero | Versioning + replication |
| EKS configuration | Minutes | GitOps + scheduled resource backup |
| Terraform | Near-zero | Git repository |
| Helm | Near-zero | Git repository |
| ECR images | Near-zero | Immutable tags + replication |
| Prometheus | 1–24 hours | Snapshot/remote durable storage |
| Grafana | 1 day | Configuration in Git + DB backup |
| ELK | Depends on retention | Snapshot repository |

---

# 6. RTO and Restore Design

RTO determines how quickly service must be restored.

Example:

```text
Critical production application:
RTO = 60 minutes

Critical database:
RTO = 30 minutes

Observability:
RTO = 4 hours
```

The backup architecture must be designed around these objectives.

A backup that takes six hours to restore does not satisfy a one-hour RTO.

---

# 7. Backup Classification

Classify resources before designing backup schedules.

## Tier 0 — Critical Data

Examples:

- Production database
- Payment-related state
- Customer records
- Orders

Requirements:

- Frequent recovery points
- PITR
- Cross-region protection
- Encryption
- Strict retention
- Restore testing

## Tier 1 — Important Application State

Examples:

- Persistent volumes
- Search indexes
- Stateful application data

Requirements:

- Scheduled backups
- Snapshot retention
- Restore testing

## Tier 2 — Platform Configuration

Examples:

- Kubernetes manifests
- Helm charts
- Terraform
- Argo CD applications

Requirements:

- Git versioning
- Protected branches
- Repository backup

## Tier 3 — Observability

Examples:

- Grafana configuration
- Alerting rules
- Elasticsearch data

Requirements depend on business importance.

---

# 8. Production Backup Architecture

A representative design:

```text
                         AWS Organization
                               |
              +----------------+----------------+
              |                                 |
        Production Account                Backup Account
              |                                 |
        +-----+------+                   +------+------+
        |            |                   |             |
       EKS           RDS                 S3          Backup Vault
        |            |                   |             |
       PVs       Snapshots/PITR       Versioning    Immutable Copies
        |            |                   |             |
        +------------+-------------------+-------------+
                             |
                       Cross-Region Copy
                             |
                       DR Region
```

The backup environment should have stronger protection than the production environment.

---

# 9. AWS Backup Strategy

AWS-native backup capabilities can be used for supported AWS resources.

A production organization should define:

- Backup plans
- Backup rules
- Backup vaults
- Retention periods
- Copy actions
- Encryption
- Access policies
- Backup monitoring
- Restore testing

Conceptual flow:

```text
Resource
   |
   v
Backup Plan
   |
   v
Backup Rule
   |
   +--> Primary Backup Vault
   |
   +--> Cross-Region Copy
             |
             v
        DR Vault
```

---

# 10. Backup Vault Separation

Do not keep all backups in the same administrative boundary as production.

Weak design:

```text
Production Admin
      |
      +--> Production
      |
      +--> Backups
```

A compromised production administrator may delete both.

Stronger design:

```text
Production Account
      |
      | backup copy
      v
Central Backup Account
      |
      v
Protected Backup Vault
      |
      v
DR Region
```

Use least privilege so production workloads can create backups without being able to destroy protected recovery points.

---

# 11. Encryption

Backups must be encrypted.

Typical AWS encryption design:

```text
Production Resource
       |
       v
AWS Backup
       |
       v
KMS encryption
       |
       v
Backup Vault
```

Use customer-managed KMS keys where organizational requirements require centralized control.

Consider:

- Key rotation
- Key policies
- Cross-account access
- Cross-region key availability
- Restore permissions
- Break-glass access

Never store encryption keys beside unprotected backup metadata.

---

# 12. RDS Backup Strategy

For production relational databases, use:

- Automated backups
- Point-in-time recovery
- Manual snapshots
- Cross-region snapshots where appropriate
- Retention policies
- Encryption
- Restore testing

Example:

```text
RDS
 |
 +--> Automated backups
 |
 +--> Transaction logs
 |
 +--> PITR
 |
 +--> Manual snapshot before risky change
 |
 +--> Cross-region snapshot
```

---

# 13. Point-in-Time Recovery

PITR allows recovery to a specific time within the available recovery window.

Example:

```text
09:00 normal
09:20 migration starts
09:27 bad migration
09:28 incident detected
```

Instead of restoring only the previous nightly snapshot, the database may be restored to:

```text
09:19:30
```

This can dramatically reduce data loss.

---

# 14. Manual Snapshot Before Database Migration

Before a high-risk production migration:

```text
1. Validate migration script
2. Confirm backup health
3. Create manual snapshot
4. Record snapshot identifier
5. Confirm snapshot completion
6. Execute migration
7. Validate application
8. Monitor
```

Never assume that "the database has automated backups" means a risky migration is safe.

---

# 15. Example AWS CLI — RDS Snapshot

Example:

```bash
aws rds create-db-snapshot \
  --db-instance-identifier roboshop-prod-db \
  --db-snapshot-identifier roboshop-prod-pre-migration-$(date +%Y%m%d%H%M)
```

Verify:

```bash
aws rds describe-db-snapshots \
  --db-instance-identifier roboshop-prod-db
```

Wait:

```bash
aws rds wait db-snapshot-available \
  --db-snapshot-identifier roboshop-prod-pre-migration-202608311030
```

Production practice:

- Store snapshot ID in the change record.
- Record creation time.
- Record migration version.
- Record operator/change ID.
- Do not delete immediately after the migration.

---

# 16. RDS Restore

A restore creates a new database instance from a snapshot.

Example:

```bash
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier roboshop-prod-db-restore-test \
  --db-snapshot-identifier roboshop-prod-pre-migration-202608311030
```

Never overwrite the production database during an initial restore test.

Restore to an isolated instance first.

---

# 17. PITR Restore Example

Conceptually:

```bash
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier roboshop-prod-db \
  --target-db-instance-identifier roboshop-prod-db-pitr-test \
  --restore-time 2026-08-31T10:19:30Z
```

After restore:

```text
Restore
  |
  v
Isolated DB
  |
  +--> Schema validation
  +--> Row count validation
  +--> Application connectivity
  +--> Data integrity checks
```

Only after validation should a controlled cutover be considered.

---

# 18. Database Restore Validation

Do not validate only that the database starts.

Validate:

- Database engine
- Version
- Schema
- Tables
- Row counts
- Indexes
- Constraints
- Users
- Permissions
- Application connectivity
- Critical business records
- Recent transactions
- Referential integrity

Example SQL:

```sql
SELECT COUNT(*) FROM customers;
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM products;
```

Compare against known expected values.

---

# 19. EBS Snapshot Strategy

For EBS-backed workloads:

```text
EBS Volume
    |
    v
EBS Snapshot
    |
    +--> Retention
    |
    +--> Encryption
    |
    +--> Cross-region copy
```

Snapshots should be scheduled based on data criticality.

Avoid blindly snapshotting every temporary volume.

---

# 20. EBS Restore

Example:

```bash
aws ec2 create-volume \
  --availability-zone ap-south-1a \
  --snapshot-id snap-0123456789abcdef0 \
  --volume-type gp3
```

Verify:

```bash
aws ec2 describe-volumes \
  --volume-ids vol-0123456789abcdef0
```

Then attach:

```bash
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-0123456789abcdef0 \
  --device /dev/sdf
```

For Kubernetes-managed storage, do not manually attach storage without understanding the CSI lifecycle.

---

# 21. Kubernetes Persistent Volume Backups

Modern EKS environments commonly use CSI drivers.

For EBS-backed PVCs:

```text
Application
    |
    v
PVC
    |
    v
EBS CSI Driver
    |
    v
EBS Volume
```

Backup design should account for:

- PVC
- PV
- StorageClass
- CSI snapshots
- Volume snapshot content
- Application consistency

---

# 22. Application-Consistent Backups

A filesystem snapshot does not necessarily guarantee application consistency.

For databases:

```text
Application writes
      |
      v
Database memory
      |
      v
Database files
```

A crash-consistent snapshot may capture data at an inconvenient point.

Prefer database-native mechanisms where possible:

- RDS automated backups
- PITR
- Database dumps where appropriate
- Consistent snapshot workflows

---

# 23. Kubernetes Resource Backup

Kubernetes resources include:

- Deployments
- Services
- ConfigMaps
- Secrets
- Ingress
- ServiceAccounts
- Roles
- RoleBindings
- HPA
- PDB
- CRDs
- Custom resources

However, production Kubernetes configuration should primarily be reconstructable from Git.

The correct design is:

```text
GitOps Repository
       |
       v
Argo CD
       |
       v
EKS
```

Therefore Git becomes a major recovery source.

---

# 24. GitOps as a Backup Mechanism

Git provides:

- Version history
- Commit history
- Branches
- Tags
- Pull requests
- Code review
- Revert capability

Example:

```text
gitops/
├── apps/
├── environments/
├── clusters/
├── helm-values/
└── argocd/
```

If the EKS cluster is lost:

```text
Terraform
    |
    v
New EKS
    |
    v
Argo CD
    |
    v
GitOps repository
    |
    v
Applications restored
```

This is one of the most important recovery patterns in the capstone.

---

# 25. Git Repository Protection

Use:

- Protected branches
- Required reviews
- Signed commits where required
- MFA
- SSO
- Access control
- Repository backups
- Multiple maintainers
- Audit logging

Avoid a design where one administrator can delete both production and its recovery configuration.

---

# 26. Terraform Recovery

Terraform source should be version-controlled.

Example:

```text
terraform/
├── modules/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── backend/
```

The Terraform code should recreate:

- VPC
- Subnets
- IAM
- EKS
- Node groups
- Security groups
- Load balancer dependencies
- ECR
- Supporting AWS services

Terraform state is also critical.

---

# 27. Terraform State Backup

If using S3 backend:

```text
Terraform
    |
    v
S3 backend
    |
    +--> Versioning
    +--> Encryption
    +--> Access control
    +--> Replication
```

Example backend:

```hcl
terraform {
  backend "s3" {
    bucket         = "company-prod-terraform-state"
    key            = "roboshop/prod/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

Modern Terraform deployments may use alternative locking mechanisms depending on the backend/version.

---

# 28. S3 Backup Strategy

For critical S3 data enable:

- Versioning
- Encryption
- Lifecycle rules
- Replication where needed
- Access logging/auditing
- Object Lock where regulatory or immutability requirements apply

Example:

```text
Application
   |
   v
S3 bucket
   |
   +--> Versioning
   |
   +--> Replication
   |
   +--> Object retention
   |
   +--> Lifecycle
```

---

# 29. S3 Versioning

Without versioning:

```text
file.json
   |
   v
overwrite
   |
   v
old version lost
```

With versioning:

```text
file.json
  |
  +--> v1
  +--> v2
  +--> v3
```

This can help recover from accidental overwrite or deletion.

---

# 30. S3 Cross-Region Replication

For critical data:

```text
Primary Region
ap-south-1
    |
    v
S3 bucket
    |
    | replication
    v
DR Region
ap-southeast-1
    |
    v
DR S3 bucket
```

Replication must be monitored.

Do not assume that enabling replication guarantees every object is available immediately.

---

# 31. ECR Backup Strategy

Container images are deployment artifacts.

Production images should use:

- Immutable tags
- Image digests
- Lifecycle policies
- Vulnerability scanning
- Cross-region replication for DR
- Retention of approved releases

Avoid relying only on:

```text
latest
```

Prefer:

```text
roboshop/frontend:1.42.3
```

and:

```text
sha256:<digest>
```

---

# 32. ECR Cross-Region Replication

Example architecture:

```text
CI
 |
 v
Primary ECR
 |
 +--> approved image
 |
 +--> replication
          |
          v
      DR ECR
```

During regional recovery:

```text
DR EKS
   |
   v
DR ECR
   |
   v
image pull
```

This avoids rebuilding images during an outage.

---

# 33. Image Immutability

Never overwrite production artifacts.

Bad:

```text
frontend:prod
```

where the underlying image changes.

Better:

```text
frontend:2026.08.31-142
```

or digest pinning:

```yaml
image:
  repository: <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/frontend
  digest: "sha256:REDACTED_DIGEST"
```

---

# 34. Helm Backup

Helm charts should be stored in Git.

Example:

```text
helm/
└── roboshop/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-qa.yaml
    ├── values-prod.yaml
    └── templates/
```

Helm release state exists in Kubernetes, but Git should remain the source of truth in a GitOps architecture.

---

# 35. Argo CD Recovery

Argo CD configuration includes:

- Applications
- AppProjects
- Repositories
- Cluster registrations
- RBAC
- Notifications
- Settings

Where possible, define these as declarative manifests in Git.

Example:

```text
argocd/
├── projects/
├── applications/
├── appsets/
└── config/
```

Recovery:

```text
New EKS
   |
   v
Install Argo CD
   |
   v
Apply bootstrap manifests
   |
   v
Connect Git repository
   |
   v
Argo CD reconciles applications
```

---

# 36. Prometheus Backup

Prometheus primarily stores time-series metrics locally unless configured with durable/remote storage.

Possible strategies:

- Prometheus snapshots
- Persistent volume snapshots
- Remote durable metrics storage
- Rebuild Prometheus from configuration

For many environments, alerting rules and dashboards are more important to recover quickly than historical metrics.

---

# 37. Grafana Backup

Grafana configuration includes:

- Dashboards
- Datasources
- Alerting configuration
- Users/teams
- Folders
- Policies

Recommended approach:

```text
Dashboards
   |
   v
Git / provisioning
```

Configuration should be reproducible.

Example:

```text
grafana/
├── dashboards/
├── provisioning/
│   ├── dashboards/
│   └── datasources/
└── alerting/
```

---

# 38. ELK Backup

Elasticsearch data can be protected using snapshots.

Architecture:

```text
Elasticsearch
      |
      v
Snapshot Repository
      |
      v
S3
      |
      v
DR copy
```

Kibana configuration should also be exported/versioned where required.

Log retention should match business and compliance requirements.

---

# 39. Elasticsearch Snapshot Principle

Do not copy Elasticsearch data directories manually as the primary backup mechanism.

Use Elasticsearch-supported snapshot/restore mechanisms.

Conceptually:

```text
Index
 |
 v
Snapshot
 |
 v
Object storage
 |
 v
Restore
 |
 v
New cluster/index
```

---

# 40. Backup of Secrets

Secrets require special handling.

Do not put plaintext production secrets into Git.

Use:

- AWS Secrets Manager
- AWS KMS
- External secret systems where approved
- Sealed/encrypted secret manifests where appropriate

Recovery requires:

1. Secret source available
2. KMS key available
3. IAM permissions available
4. ExternalSecrets/secret controller available
5. Applications restarted/reconciled

---

# 41. Secret Recovery Dependency Chain

```text
KMS
 |
 v
Secrets Manager
 |
 v
IAM
 |
 v
External Secrets controller
 |
 v
Kubernetes Secret
 |
 v
Application
```

If KMS is unavailable, the rest of the chain may fail.

Therefore KMS recovery is part of application recovery.

---

# 42. Kubernetes Secret Backup Warning

A Kubernetes Secret is not automatically safe merely because it is stored in etcd.

A base64-encoded value is not encryption.

Example:

```bash
echo 'cGFzc3dvcmQ=' | base64 --decode
```

Therefore:

- Protect etcd
- Encrypt secrets at rest
- Avoid plaintext Git
- Use an approved secret manager

---

# 43. Backup Scheduling Example

A representative policy:

```text
Critical DB:
  PITR: continuous
  Snapshot: daily
  Weekly: retained longer
  Monthly: compliance retention

EBS:
  Hourly for critical PVs
  Daily for standard PVs

S3:
  Versioning: enabled
  Replication: enabled for critical data

EKS configuration:
  Git continuously
  Resource backup: daily

Observability:
  Config in Git
  Data snapshots according to retention requirements
```

Actual frequency should be derived from RPO and cost requirements.

---

# 44. Example AWS Backup Plan Concept

Illustrative Terraform:

```hcl
resource "aws_backup_vault" "prod" {
  name        = "roboshop-prod-backup-vault"
  kms_key_arn = aws_kms_key.backup.arn
}

resource "aws_backup_plan" "prod" {
  name = "roboshop-prod-backup-plan"

  rule {
    rule_name         = "daily"
    target_vault_name = aws_backup_vault.prod.name
    schedule          = "cron(0 2 * * ? *)"

    lifecycle {
      delete_after = 35
    }
  }
}
```

Production code should additionally consider:

- Copy actions
- Vault Lock where required
- IAM
- Backup selection
- Tags
- Notifications
- Region strategy

---

# 45. Backup Selection by Tags

Tag-based selection can prevent accidental omission.

Example:

```text
Backup=true
Environment=prod
Criticality=high
```

Terraform:

```hcl
resource "aws_backup_selection" "prod" {
  iam_role_arn = aws_iam_role.backup.arn
  name         = "roboshop-prod-resources"
  plan_id      = aws_backup_plan.prod.id

  selection_tag {
    type  = "STRINGEQUALS"
    key   = "Backup"
    value = "true"
  }
}
```

Use consistent tagging across the environment.

---

# 46. Production Tagging Standard

Recommended tags:

```text
Environment = prod
Application  = roboshop
Owner        = platform-team
Backup       = true
Criticality  = high
CostCenter   = platform
DataClass    = confidential
```

Tags should be standardized through Terraform/modules/policy.

---

# 47. Backup Monitoring

Backup failure must generate an alert.

Monitor:

- Backup job failed
- Backup job missed
- Snapshot failed
- Replication delayed
- Vault access failure
- Restore job failed
- Backup storage growth
- Retention policy drift
- Encryption failure
- Cross-region copy failure

A backup system that silently fails is dangerous.

---

# 48. Backup Alert Example

Prometheus may consume exported backup metrics or monitoring integrations.

Conceptual alert:

```yaml
groups:
  - name: backup-alerts
    rules:
      - alert: BackupJobFailed
        expr: backup_job_status{environment="prod"} == 0
        for: 15m
        labels:
          severity: critical
          team: platform
          environment: prod
        annotations:
          summary: "Production backup job failed"
          description: "A production backup job has failed."
          runbook_url: "https://runbooks.example.internal/backup-job-failed"
```

The exact metric depends on the backup integration.

---

# 49. Backup Completeness Check

Do not check only job status.

Ask:

```text
Was backup scheduled?
Was backup started?
Did backup finish?
Is recovery point valid?
Is backup encrypted?
Was cross-region copy completed?
Is retention correct?
Can restore access the backup?
```

---

# 50. Backup Verification

A backup can appear successful while being unusable due to:

- Corruption
- Missing permissions
- Missing KMS key
- Broken repository
- Incompatible version
- Incomplete replication
- Missing dependencies

Therefore perform automated verification where possible.

---

# 51. Restore Testing

Restore tests should be scheduled.

Example:

```text
Every month:
  restore critical database

Every quarter:
  full application recovery test

Twice yearly:
  regional DR exercise
```

The frequency depends on business requirements.

---

# 52. Restore Test Workflow

```text
Select recovery point
       |
       v
Restore to isolated environment
       |
       v
Validate infrastructure
       |
       v
Validate database
       |
       v
Deploy application
       |
       v
Run smoke tests
       |
       v
Validate business transactions
       |
       v
Measure RTO
       |
       v
Record result
```

---

# 53. Restore Test Must Not Affect Production

Use:

- Isolated account where practical
- Isolated VPC
- Separate namespace
- Separate database endpoint
- Non-production DNS
- Restricted IAM

Never accidentally point restored applications to the production database.

---

# 54. Restore Testing Safety Checklist

Before restore:

```text
[ ] Environment isolated
[ ] DNS isolated
[ ] Production endpoints blocked
[ ] Production credentials not reused unnecessarily
[ ] Restore target identified
[ ] KMS access validated
[ ] Expected data checks defined
```

---

# 55. Full Application Restore

Example sequence:

```text
1. Restore AWS networking
2. Restore IAM dependencies
3. Create EKS
4. Configure node groups
5. Install required controllers
6. Install ALB controller
7. Install metrics/logging
8. Install Argo CD
9. Bootstrap GitOps
10. Restore database
11. Restore persistent application data
12. Deploy applications
13. Validate services
14. Validate ALB
15. Validate DNS
16. Validate monitoring
17. Execute smoke tests
```

---

# 56. Dependency Ordering

Recovery order matters.

Example:

```text
AWS Account / IAM
        |
        v
KMS
        |
        v
VPC
        |
        v
EKS
        |
        v
CSI / ALB / Controllers
        |
        +----> Database
        |
        v
Argo CD
        |
        v
Applications
        |
        v
Ingress
        |
        v
DNS
        |
        v
Users
```

Do not deploy applications before their required infrastructure exists.

---

# 57. Database Before Application

If the application expects a database schema:

```text
Database restore
      |
      v
Schema validation
      |
      v
Application deployment
```

Do not immediately scale the application to production traffic before validating database connectivity.

---

# 58. Backup Retention

Retention should be based on:

- Ransomware protection
- Compliance
- Business requirements
- Storage cost
- Recovery requirements

Example:

```text
Hourly: 48 hours
Daily: 35 days
Weekly: 12 weeks
Monthly: 12 months
Yearly: 7 years where required
```

These are examples, not universal requirements.

---

# 59. Retention Tiering

A common pattern:

```text
Short-term
  |
  +--> frequent backups

Medium-term
  |
  +--> daily/weekly

Long-term
  |
  +--> monthly/yearly
```

Long-term backups should not be unnecessarily kept at expensive storage tiers.

---

# 60. Immutability

Immutability prevents modification or deletion during the retention period.

Important for:

- Ransomware
- Insider threats
- Compromised credentials
- Accidental deletion

Possible AWS controls include:

- Backup Vault Lock
- S3 Object Lock
- Separate backup account
- Restricted IAM

---

# 61. Backup Account Security

The backup account should have:

- MFA
- Restricted administrators
- Separate roles
- CloudTrail
- Security monitoring
- KMS controls
- Vault protection
- Alerting
- Limited production access

A production incident should not automatically give an attacker access to all backups.

---

# 62. Break-Glass Recovery

Define emergency access.

Example:

```text
Normal:
  Platform engineer -> read backup status

Emergency:
  Authorized incident commander
        |
        v
  Break-glass role
        |
        v
  Restore protected backup
```

Every break-glass action should be:

- Audited
- Time-bound
- Approved
- Logged

---

# 63. Backup Restore IAM

Restore permissions should be separate from delete permissions where practical.

Example conceptual roles:

```text
BackupWriter
BackupReader
RestoreOperator
BackupAdministrator
```

Least privilege reduces blast radius.

---

# 64. EKS Cluster Recovery

EKS itself is a managed control plane, but the complete platform is not automatically recoverable just because EKS is managed.

You must recover:

- VPC
- IAM
- EKS
- Node groups
- Add-ons
- CSI
- ALB controller
- External dependencies
- Namespaces
- CRDs
- Applications
- Persistent data
- DNS
- Secrets

---

# 65. EKS Recovery with Terraform

Example:

```bash
cd terraform/environments/prod

terraform init
terraform plan
terraform apply
```

Validate:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name roboshop-prod
```

Then:

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 66. EKS Recovery Validation

Check:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get crd
kubectl get storageclass
kubectl get ingress -A
kubectl get svc -A
```

Then validate:

- Nodes Ready
- CoreDNS healthy
- kube-proxy healthy
- VPC CNI healthy
- EBS CSI healthy
- ALB controller healthy

---

# 67. EBS CSI Recovery

Check:

```bash
kubectl get pods -n kube-system | grep ebs
```

Then:

```bash
kubectl get csidrivers
```

Expected driver:

```text
ebs.csi.aws.com
```

If CSI is unavailable, PVC-backed applications may fail to mount storage.

---

# 68. ALB Recovery

After application recovery:

```bash
kubectl get ingress -A
```

Then:

```bash
kubectl describe ingress <ingress-name> -n <namespace>
```

Check:

- ALB created
- Target groups healthy
- Security groups correct
- Listener configuration correct
- Certificates attached
- Health checks passing

---

# 69. DNS Recovery

If using Route 53:

```text
DNS
 |
 +--> primary region
 |
 +--> health check
 |
 +--> DR region
```

Failover can be:

- Automatic
- Manual
- Weighted
- Latency-based
- Failover policy

DNS TTL should be considered during recovery planning.

---

# 70. Database DNS Abstraction

Applications should avoid hard-coding region-specific database endpoints.

Use configuration such as:

```text
DATABASE_HOST=db.prod.internal
```

Then recovery can change the underlying endpoint without modifying every application image.

---

# 71. GitOps Bootstrap

After cluster creation:

```bash
kubectl create namespace argocd
```

Install Argo CD using the approved production method.

Then apply bootstrap application:

```bash
kubectl apply -f argocd/bootstrap/
```

Argo CD should begin reconciling Git state.

---

# 72. GitOps Recovery Validation

Check:

```bash
argocd app list
```

Then:

```bash
argocd app get roboshop-prod
```

Look for:

```text
Sync Status: Synced
Health Status: Healthy
```

If not healthy, investigate before exposing traffic.

---

# 73. Recovery of Kubernetes Secrets

If using AWS Secrets Manager and External Secrets:

```text
Secrets Manager
       |
       v
External Secrets Operator
       |
       v
Kubernetes Secret
       |
       v
Pod
```

Check:

```bash
kubectl get externalsecrets -A
kubectl get secrets -n roboshop
```

Then inspect:

```bash
kubectl describe externalsecret <name> -n roboshop
```

Do not print secret values into terminals or incident channels.

---

# 74. Recovery of ConfigMaps

ConfigMaps should generally come from GitOps.

Check:

```bash
kubectl get configmaps -n roboshop
```

If missing, inspect Argo CD sync state rather than manually editing production resources first.

---

# 75. Backup Before High-Risk Operations

Before:

- Database migrations
- Major Kubernetes upgrade
- Terraform destructive change
- Storage migration
- EKS upgrade
- Elasticsearch upgrade
- Major application release

perform a relevant backup/checkpoint.

---

# 76. Change Management Backup Gate

A production change process can require:

```text
Change request
      |
      v
Backup verified
      |
      v
Recovery point recorded
      |
      v
Change executed
      |
      v
Validation
      |
      v
Close change
```

If backup validation fails, a high-risk change may need to be postponed.

---

# 77. Terraform Destroy Protection

Never rely solely on backups to protect against accidental Terraform destruction.

Use:

```hcl
lifecycle {
  prevent_destroy = true
}
```

where appropriate.

Example:

```hcl
resource "aws_db_instance" "prod" {
  # configuration

  lifecycle {
    prevent_destroy = true
  }
}
```

This is a safety control, not a backup.

---

# 78. Backup vs Preventative Control

Examples:

```text
prevent_destroy
      |
      v
prevents accidental destruction

backup
      |
      v
recovers after destruction
```

Production systems need both.

---

# 79. Kubernetes Deletion Protection

Use:

- RBAC
- Admission controls
- Protected namespaces
- GitOps
- Change approvals
- PDBs where applicable
- Resource policies
- Backups

A PodDisruptionBudget does not prevent intentional deletion of the workload.

---

# 80. Backup Testing Metrics

Track:

- Backup success rate
- Backup failure rate
- Recovery point age
- Restore success rate
- Restore duration
- RPO achieved
- RTO achieved
- Cross-region copy lag
- Backup storage growth
- Number of unprotected resources

---

# 81. Example Backup Dashboard

Grafana dashboard panels:

```text
Backup Success Rate
Backup Failures
Last Successful Backup
Oldest Recovery Point
Cross-Region Replication Lag
Restore Test Duration
Restore Success Rate
Backup Storage Growth
Unprotected Production Resources
```

---

# 82. Backup SLO

Example:

```text
99.9% of scheduled production backup jobs complete successfully.
```

Another:

```text
Critical database recovery point age remains below 15 minutes.
```

These are operational objectives, not universal defaults.

---

# 83. Backup Alert Severity

Critical:

```text
Critical DB has no valid recovery point
Cross-region backup copy failed repeatedly
Restore verification failed
```

Warning:

```text
Backup delayed
Storage growth unusual
Non-critical backup failed
```

Info:

```text
Scheduled backup completed
```

---

# 84. Backup Failure Runbook

When backup fails:

```text
1. Identify resource
2. Identify failed backup job
3. Check error message
4. Check IAM
5. Check KMS
6. Check resource state
7. Check storage capacity
8. Retry if safe
9. Confirm new recovery point
10. Escalate if RPO is at risk
```

---

# 85. Common Backup Failure — IAM

Symptom:

```text
AccessDenied
```

Investigation:

```bash
aws sts get-caller-identity
```

Check role:

```bash
aws iam get-role --role-name <backup-role>
```

Then review policy and trust relationship.

Root cause:

- Missing permission
- Wrong role
- Incorrect trust relationship
- SCP denial

---

# 86. Common Backup Failure — KMS

Symptom:

```text
KMS AccessDenied
```

Investigate:

- Key policy
- IAM permissions
- Region
- Key state
- Cross-account permissions

Check:

```bash
aws kms describe-key \
  --key-id <key-id>
```

Do not disable encryption merely to make backup succeed.

---

# 87. Common Backup Failure — Resource State

Examples:

- Volume unavailable
- Database maintenance
- Snapshot already in progress
- Repository unavailable

Check AWS resource state before retrying.

---

# 88. Common Restore Failure — KMS

A backup may exist but restore can fail because the restoring role cannot use the encryption key.

Therefore test:

```text
Backup creation permission
AND
Backup restore permission
```

Both matter.

---

# 89. Common Restore Failure — Network

A restored database may be healthy but unreachable because:

- Security group
- Subnet
- Route table
- NACL
- DNS
- IAM authentication

Check connectivity systematically.

---

# 90. Restore Network Troubleshooting

Example:

```bash
kubectl exec -it <pod> -n roboshop -- sh
```

Then:

```bash
nc -vz <database-host> 5432
```

or:

```bash
timeout 5 bash -c '</dev/tcp/<database-host>/5432'
```

Use the actual database port.

---

# 91. Restore Failure — Secrets

Symptom:

```text
Application starts but authentication fails.
```

Possible cause:

- Secret not restored
- Wrong secret version
- KMS access failure
- External secret controller not ready
- Database password changed

Check:

```bash
kubectl describe externalsecret <name> -n roboshop
```

Do not expose the secret value.

---

# 92. Restore Failure — Image

Symptom:

```text
ImagePullBackOff
```

During DR, likely causes include:

- ECR unavailable
- Wrong region
- Image not replicated
- IAM pull permission missing
- Image tag missing

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

---

# 93. Restore Failure — ALB

Symptom:

```text
ALB exists but returns 503
```

Investigate:

```text
DNS
 |
 v
ALB
 |
 v
Target Group
 |
 v
Service
 |
 v
Pod
```

Check:

```bash
kubectl get ingress -n roboshop
kubectl get svc -n roboshop
kubectl get endpoints -n roboshop
kubectl get pods -n roboshop
```

---

# 94. Restore Failure — Persistent Volume

Symptom:

```text
Pod stuck in Pending
```

Check:

```bash
kubectl describe pod <pod> -n roboshop
kubectl get pvc -n roboshop
kubectl describe pvc <pvc> -n roboshop
```

Look for:

- Snapshot restore failure
- StorageClass mismatch
- Availability Zone mismatch
- CSI driver failure
- IAM issue

---

# 95. Database Restore Cutover

A controlled database cutover may look like:

```text
Old DB
  |
  | stop writes
  v
Restore/PITR DB
  |
  v
Validate
  |
  v
Update endpoint
  |
  v
Restart application
  |
  v
Smoke test
  |
  v
Enable traffic
```

The exact process depends on database technology.

---

# 96. Avoiding Split-Brain During Recovery

Do not accidentally allow:

```text
Primary DB
     ^
     |
Application A

DR DB
     ^
     |
Application B
```

with both accepting writes unless the database architecture explicitly supports it.

During disaster recovery:

- Establish authoritative region
- Control DNS
- Stop old writers where possible
- Confirm replication direction
- Communicate cutover state

---

# 97. Backup Restore and Data Integrity

After restore verify:

```text
Counts
Checksums where applicable
Transactions
Foreign keys
Indexes
Application reads
Application writes
Critical business workflows
```

For RoboShop:

```text
Browse product
Add item to cart
Create order
Process order workflow
Validate inventory
Validate payment workflow
```

Use non-production payment integrations during restore tests.

---

# 98. RoboShop Backup Map

Representative services:

```text
frontend
catalogue
user
cart
shipping
payment
dispatch
web
```

Map state:

```text
catalogue -> database
user      -> database
cart      -> database
payment   -> database
```

Exact storage depends on implementation.

The backup plan must be based on actual service dependencies rather than assumptions.

---

# 99. Production Backup Dependency Matrix

| Component | Backup | Restore priority |
|---|---|---:|
| Terraform | Git/S3 state | Critical |
| GitOps | Git | Critical |
| ECR | Replication | Critical |
| Database | PITR/snapshot | Critical |
| S3 | Versioning/replication | Critical |
| EBS | Snapshot | High |
| EKS config | Git/resource backup | High |
| Argo CD | Git/declarative config | High |
| Grafana | Git/DB | Medium |
| Prometheus | Config/PV/remote storage | Medium |
| ELK | Snapshots | Medium/High |

---

# 100. Backup Architecture for RoboShop

```text
                    Developer
                        |
                        v
                  Git repositories
                  /             \
                 /               \
          Terraform             GitOps
              |                   |
              v                   v
             AWS                 Argo CD
              |                   |
       +------+-------+           v
       |              |          EKS
      ECR            RDS          |
       |              |      +----+----+
       |              |      |         |
       |              |    Apps      Observability
       |              |      |         |
       |              |      v         v
       |              |    PVs      Prometheus
       |              |                |
       |              |             Grafana
       |              |
       +--------------+
              |
              v
        Backup system
              |
      +-------+-------+
      |               |
 Primary Vault     DR Vault
      |               |
      +-------+-------+
              |
         Restore tests
```

---

# 101. Complete Production Recovery Example

Scenario:

```text
Production database accidentally corrupted.
```

Timeline:

```text
10:00 normal
10:15 migration begins
10:21 corruption introduced
10:25 incident detected
```

Response:

```text
1. Stop further writes if required
2. Declare incident
3. Preserve evidence
4. Identify corruption timestamp
5. Select PITR target before corruption
6. Restore isolated database
7. Validate data
8. Run application integration tests
9. Prepare controlled cutover
10. Update application endpoint
11. Restart/reload application
12. Validate transactions
13. Monitor
14. Document incident
15. Review migration controls
```

---

# 102. Restore Decision Tree

```text
Data lost?
    |
    +-- No --> Continue investigation
    |
    +-- Yes
          |
          v
      Is source healthy?
          |
          +-- Yes --> Recover logically if safe
          |
          +-- No
                |
                v
            Need point in time?
                |
                +-- Yes --> PITR
                |
                +-- No --> Snapshot restore
```

---

# 103. Logical Restore vs Physical Restore

Logical:

```text
dump -> restore
```

Examples:

- SQL dump
- Table export

Advantages:

- Selective recovery
- Schema-level recovery

Disadvantages:

- Potentially slow
- Large datasets take time

Physical:

```text
snapshot -> restore volume/database
```

Advantages:

- Faster for large systems

Disadvantages:

- Less selective

Use the method appropriate to the recovery objective.

---

# 104. Database Dump Example

For a PostgreSQL-style database:

```bash
pg_dump \
  --format=custom \
  --file=roboshop-prod.dump \
  "$DATABASE_URL"
```

Restore:

```bash
pg_restore \
  --clean \
  --if-exists \
  --dbname="$RESTORE_DATABASE_URL" \
  roboshop-prod.dump
```

Production caution:

- Protect dump files
- Encrypt them
- Restrict access
- Never expose credentials
- Validate compatibility
- Store outside the source database host

---

# 105. MySQL-Style Logical Backup Example

Conceptually:

```bash
mysqldump \
  --single-transaction \
  --routines \
  --triggers \
  -h "$DB_HOST" \
  -u "$DB_USER" \
  -p \
  roboshop > roboshop.sql
```

Restore:

```bash
mysql \
  -h "$RESTORE_DB_HOST" \
  -u "$DB_USER" \
  -p \
  roboshop < roboshop.sql
```

Use database-native tools appropriate to the engine/version.

---

# 106. Backup Security Controls

Minimum controls:

```text
Encryption
Least privilege
MFA
Separate account
Immutable retention
Audit logs
Cross-region copy
Access monitoring
Restore testing
```

---

# 107. Backup Compliance

Depending on business requirements, backups may need:

- Defined retention
- Encryption
- Data residency
- Access logging
- Legal hold
- Immutability
- Audit evidence

Do not assume one retention policy is appropriate for all data.

---

# 108. Backup Cost Optimization

Do not optimize backup cost by deleting required recovery points.

Instead:

- Tier storage
- Compress logical backups
- Use lifecycle policies
- Remove obsolete test backups
- Deduplicate where supported
- Right-size retention
- Avoid redundant copies that do not improve recovery objectives

---

# 109. Backup Cost Example

Suppose:

```text
Production data = 10 TB
Daily backup retention = 35 days
```

Do not simply calculate:

```text
10 TB × 35
```

Snapshot technologies may use incremental storage.

Actual cost depends on:

- Changed blocks
- Retention
- Region
- Copy count
- Storage tier
- Restore traffic

---

# 110. Backup Runbook

## Objective

Verify that production backups are healthy.

## Procedure

```bash
aws backup list-backup-jobs \
  --by-state FAILED
```

Check recent jobs:

```bash
aws backup list-backup-jobs \
  --by-created-after "$(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%SZ)"
```

Then inspect:

```bash
aws backup describe-backup-job \
  --backup-job-id <JOB_ID>
```

Record:

- Resource
- Job status
- Recovery point
- Timestamp
- Error
- Remediation

---

# 111. Restore Runbook — Database

```text
[ ] Incident/change approved
[ ] Recovery point selected
[ ] Target environment isolated
[ ] KMS permissions validated
[ ] Restore started
[ ] Restore completed
[ ] Connectivity verified
[ ] Schema verified
[ ] Data verified
[ ] Application tested
[ ] Cutover approved
[ ] Monitoring active
[ ] Incident/change updated
```

---

# 112. Restore Runbook — EKS

```text
[ ] VPC available
[ ] IAM available
[ ] KMS available
[ ] EKS created
[ ] Nodes Ready
[ ] CoreDNS healthy
[ ] CNI healthy
[ ] CSI healthy
[ ] ALB controller healthy
[ ] Argo CD healthy
[ ] GitOps synced
[ ] Secrets available
[ ] Database available
[ ] Applications healthy
[ ] Ingress healthy
[ ] DNS validated
```

---

# 113. Restore Runbook — ECR

```text
[ ] DR registry available
[ ] Image replication verified
[ ] Required digest exists
[ ] Pull permissions verified
[ ] Kubernetes image reference validated
[ ] Test pod starts
```

---

# 114. Restore Runbook — S3

```text
[ ] Bucket exists
[ ] Versioning enabled
[ ] Encryption enabled
[ ] Replication state verified
[ ] Required object version identified
[ ] Restore/copy completed
[ ] Application access validated
```

---

# 115. Restore Runbook — ELK

```text
[ ] Elasticsearch cluster available
[ ] Snapshot repository available
[ ] Correct snapshot identified
[ ] Restore performed
[ ] Index health verified
[ ] Required indexes restored
[ ] Kibana connectivity verified
[ ] Application logs visible
```

---

# 116. Restore Runbook — Grafana

```text
[ ] Grafana deployed
[ ] Datasources configured
[ ] Dashboards restored
[ ] Alert rules restored
[ ] RBAC validated
[ ] Prometheus connectivity validated
```

---

# 117. Restore Runbook — Prometheus

```text
[ ] Prometheus deployed
[ ] Service discovery working
[ ] Targets healthy
[ ] Rules loaded
[ ] Alertmanager reachable
[ ] Critical alerts evaluated
```

---

# 118. Backup Drill

A backup drill should simulate a real incident.

Example:

```text
Scenario:
Production DB unavailable.
```

Measure:

```text
Detection time
Backup selection time
Restore time
Validation time
Cutover time
Total RTO
Data loss
```

Then compare against the target.

---

# 119. Backup Drill Report

Include:

```text
Date:
Environment:
Scenario:
Recovery point:
RPO target:
RPO achieved:
RTO target:
RTO achieved:
Restore result:
Failures:
Corrective actions:
Owner:
Due date:
```

---

# 120. Restore Testing Automation

A mature organization automates:

```text
Select latest recovery point
       |
       v
Restore
       |
       v
Run health checks
       |
       v
Run SQL/data checks
       |
       v
Run application tests
       |
       v
Publish result
```

This provides evidence that backups are actually usable.

---

# 121. Canary Restore

Instead of restoring every backup:

```text
Latest backup
      |
      v
Periodic sample restore
```

This provides continuous confidence without excessive cost.

---

# 122. Backup Drift

Backup configuration can drift.

Examples:

- Backup tag removed
- Resource excluded
- Retention changed
- Replication disabled
- Encryption changed
- Backup job schedule changed

Use policy checks and periodic audits.

---

# 123. Backup Coverage Report

Generate a report:

```text
Resource                  Backup     Cross-region
-------------------------------------------------
prod-db                   YES        YES
prod-pv                   YES        YES
terraform-state           YES        YES
gitops                    YES        YES
ecr                       YES        YES
critical-s3               YES        YES
grafana                   YES        NO
prometheus                YES        NO
```

Every "NO" should have an intentional reason.

---

# 124. Backup Ownership

Every critical backup should have an owner.

Example:

```text
Database       -> Database team
EKS/PV         -> Platform team
GitOps         -> DevOps team
Terraform      -> Cloud platform team
ELK            -> Observability team
Security logs  -> Security team
```

No orphaned backups.

---

# 125. Backup Documentation

Document:

- What is backed up
- Schedule
- Retention
- Storage location
- Encryption
- Restore procedure
- Owner
- Escalation
- Last restore test
- RPO
- RTO

Documentation should be stored in version control where possible.

---

# 126. Production Backup Checklist

```text
[ ] Critical resources identified
[ ] RPO defined
[ ] RTO defined
[ ] Backup schedule defined
[ ] Retention defined
[ ] Encryption enabled
[ ] Cross-region copy configured
[ ] Backup account separated
[ ] IAM restricted
[ ] Monitoring configured
[ ] Backup failures alerting configured
[ ] Restore procedure documented
[ ] Restore test scheduled
[ ] DR drill performed
```

---

# 127. Senior-Level Backup Architecture

A mature architecture looks like:

```text
                         Production
                             |
        +--------------------+-------------------+
        |                    |                   |
       RDS                  EKS                 S3
        |                    |                   |
   PITR/Snapshot       PV/Config backup     Versioning
        |                    |                   |
        +--------------------+-------------------+
                             |
                             v
                     Backup Control Plane
                             |
                +------------+-------------+
                |                          |
          Primary Vault              DR Vault
                |                          |
          KMS encrypted              KMS encrypted
                |                          |
                +------------+-------------+
                             |
                             v
                      Restore Testing
                             |
                             v
                       Evidence / SLO
```

---

# 128. Production Principles

Remember these principles:

1. Backup everything important.
2. Do not back up everything blindly.
3. Define RPO.
4. Define RTO.
5. Keep backups separate from production.
6. Encrypt backups.
7. Protect backup deletion.
8. Replicate critical backups.
9. Version configuration.
10. Use immutable application artifacts.
11. Test restores.
12. Measure recovery.
13. Monitor backup failures.
14. Document recovery.
15. Practice before the real incident.

---

# 129. Interview Question — What is the difference between HA and backup?

### Answer

HA keeps service available when components fail.

Backup provides a recoverable copy of data/configuration after data loss or corruption.

For example, an EKS cluster spread across multiple AZs provides HA, but if an operator deletes important data, AZ redundancy does not recover it. A database PITR backup can.

---

# 130. Interview Question — Why is replication not enough?

### Answer

Replication can copy corruption or accidental deletion.

If bad data is replicated immediately, every replica may contain the same bad state.

Backups provide historical recovery points, so I can recover to a known-good time.

---

# 131. Interview Question — How would you back up an EKS environment?

### Answer

I would not treat the cluster as a single backup object.

I would separate:

- Terraform infrastructure
- GitOps manifests
- Helm charts
- ECR images
- Kubernetes persistent data
- Databases
- Secrets
- DNS
- Observability configuration

Terraform and Git provide reproducibility, while databases and persistent data require dedicated backup mechanisms.

---

# 132. Interview Question — How do you restore a destroyed EKS cluster?

### Answer

First I restore the infrastructure using Terraform:

```text
VPC -> IAM -> EKS -> node groups -> add-ons
```

Then I install/bootstrap Argo CD and reconcile the GitOps repository.

After that I restore databases and persistent application data, validate secrets and dependencies, and finally expose traffic through ALB/DNS.

I would not manually recreate every Kubernetes object because GitOps should remain the source of truth.

---

# 133. Interview Question — How do you verify backups?

### Answer

I verify both backup creation and restore usability.

I monitor:

- Backup job status
- Recovery point age
- Cross-region copy
- Encryption
- Retention

Then I periodically restore backups into isolated environments and validate data and application functionality.

---

# 134. Interview Question — What happens if the backup account is compromised?

### Answer

I would use separation of duties, least privilege, immutable retention, protected vaults, MFA, audit logging and cross-account/cross-region copies.

The goal is to prevent a production compromise from allowing an attacker to delete every recovery point.

---

# 135. Interview Question — What would you back up before a production migration?

### Answer

I would first verify that the normal backup mechanism is healthy, then create a migration-specific recovery point where appropriate.

For a database migration, I would create a manual snapshot and ensure PITR remains available. I would record the recovery point in the change record before starting the migration.

---

# 136. Interview Question — What if restore works but the application does not?

### Answer

I would separate data recovery from application recovery.

I would check:

```text
DNS
networking
security groups
IAM
KMS
secrets
database connectivity
image availability
PVC mounts
service discovery
ALB
application logs
```

A successful database restore does not prove that the application dependency chain is healthy.

---

# 137. Interview Question — How do you protect backups from ransomware?

### Answer

I would use:

- Separate backup account
- Least privilege
- Immutable retention
- Vault protection
- S3 Object Lock where appropriate
- Cross-region copies
- MFA/break-glass controls
- Audit logging
- Restore testing

I would also monitor for abnormal backup deletion attempts.

---

# 138. Interview Question — What is a good backup strategy for production databases?

### Answer

For a critical database I would use automated backups with PITR, scheduled snapshots, appropriate retention, encryption and cross-region protection.

I would also perform periodic restore tests and measure whether the achieved RPO and RTO meet business requirements.

---

# 139. Interview Question — Why keep Terraform and GitOps in Git?

### Answer

Because infrastructure and desired application state should be reproducible.

If an EKS cluster is destroyed, I can recreate the infrastructure with Terraform and the workloads through Argo CD from Git.

This dramatically reduces manual recovery work.

---

# 140. Interview Question — How would you recover from accidental database deletion?

### Answer

First I would stop dependent writes if necessary and declare the incident.

Then I would identify the latest valid recovery point, restore the database into an isolated environment, validate schema and critical data, and perform a controlled cutover.

I would preserve the deleted resource's evidence and avoid making irreversible changes before understanding the incident.

---

# 141. Interview Question — What is the biggest backup mistake?

### Answer

The biggest mistake is assuming that a successful backup job means recovery is guaranteed.

A backup is only useful if it can actually be restored, decrypted, accessed and validated within the required RTO.

---

# 142. Production Backup Anti-Patterns

Avoid:

```text
latest-only backups
single-region backups for critical data
same-account-only backups
plaintext secrets
manual-only backups
no restore tests
unmonitored backup failures
unlimited retention without cost review
mutable production image tags
manual Kubernetes reconstruction
```

---

# 143. Final Backup Architecture Summary

The production RoboShop platform should use:

```text
                    Git
                 /       \
          Terraform     GitOps
              |            |
              v            v
             AWS          Argo CD
              |             |
       +------+-------+     EKS
       |      |       |      |
      ECR    RDS      S3    Apps
       |      |       |      |
       |   PITR       |     PV
       |   Snapshot   |      |
       +------+-------+------+
              |
              v
        Backup System
              |
       +------+------+
       |             |
 Primary Vault   DR Vault
       |             |
       +------+------+
              |
        Restore Tests
              |
              v
        Recovery Evidence
```

The key production principle is:

> Design every critical data path with a known recovery mechanism, a defined owner, an explicit RPO/RTO, and a tested restore procedure.

---

# 144. Final Operational Checklist

Before declaring the production backup architecture complete:

```text
[ ] RPO documented
[ ] RTO documented
[ ] Critical data inventory completed
[ ] Database PITR enabled
[ ] Database snapshots configured
[ ] EBS backups configured
[ ] S3 versioning enabled where required
[ ] S3 replication configured where required
[ ] ECR replication configured for DR
[ ] Terraform stored in Git
[ ] Terraform state protected
[ ] GitOps repository protected
[ ] Helm charts versioned
[ ] Argo CD configuration reproducible
[ ] Secrets recovery tested
[ ] KMS recovery dependencies documented
[ ] Backup account separated
[ ] Backup encryption enabled
[ ] Immutable protection evaluated
[ ] Cross-region copies configured
[ ] Backup failures monitored
[ ] Restore failures monitored
[ ] Restore runbooks documented
[ ] Database restore tested
[ ] EKS rebuild tested
[ ] Application recovery tested
[ ] ALB recovery tested
[ ] DNS failover tested
[ ] ELK recovery tested
[ ] Grafana recovery tested
[ ] Prometheus recovery tested
[ ] DR drill completed
[ ] RPO measured
[ ] RTO measured
[ ] Findings tracked
```

---

# 145. Key Takeaway

A production backup architecture is not:

```text
cron -> snapshot -> done
```

It is:

```text
Identify
   |
   v
Classify
   |
   v
Backup
   |
   v
Encrypt
   |
   v
Separate
   |
   v
Replicate
   |
   v
Monitor
   |
   v
Restore
   |
   v
Validate
   |
   v
Measure RPO/RTO
   |
   v
Improve
```

For the RoboShop production capstone, Terraform and GitOps provide reproducibility for infrastructure and Kubernetes desired state, while databases, persistent volumes, object data, artifacts and observability data require technology-specific backup strategies.

The final standard is simple:

> If you have never restored it, you do not truly know whether you have a backup.
