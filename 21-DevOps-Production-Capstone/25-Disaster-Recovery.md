# 25 — Disaster Recovery

## 1. Purpose

Disaster Recovery (DR) is the set of architecture, processes, automation, backups, procedures, and operational practices used to restore business services after a major failure.

This chapter extends the production RoboShop DevOps capstone from normal high availability into recovery from:

- Availability Zone failure
- AWS regional outage
- EKS control-plane or cluster failure
- Worker-node fleet failure
- VPC/network failure
- Accidental infrastructure deletion
- Kubernetes namespace/resource deletion
- GitOps repository loss
- ECR/image loss
- Database corruption
- Database deletion
- ELK data loss
- Prometheus/Grafana loss
- Security incidents
- Bad deployments
- Terraform state loss
- Credential or IAM incidents
- Human operational mistakes

The objective is not simply to say "we have backups."

A production DR design must answer:

1. What can fail?
2. What must be restored first?
3. Where is the authoritative configuration?
4. Where is application state stored?
5. How much data can we lose?
6. How long can the service remain unavailable?
7. Who declares a disaster?
8. Who performs failover?
9. How is recovery validated?
10. How do we return to the primary environment safely?

---

# 2. DR vs High Availability

High Availability and Disaster Recovery are related but solve different problems.

## High Availability

HA is primarily designed to keep a service running when individual components fail.

Examples:

- Multiple EKS nodes
- Multiple Availability Zones
- ALB across AZs
- Kubernetes replicas
- Pod anti-affinity
- Multiple NAT gateways
- Multi-AZ databases
- PodDisruptionBudgets
- Auto Scaling

Typical failure:

```text
Node-A fails
    |
    v
Kubernetes schedules replacement pod
    |
    v
Service remains available
```

## Disaster Recovery

DR is designed for larger failures where the primary environment cannot safely or quickly continue operating.

Example:

```text
Primary AWS Region
       |
       X
Regional outage
       |
       v
DR Region
       |
       v
Restore infrastructure
       |
       v
Restore application state
       |
       v
Deploy workloads
       |
       v
Redirect traffic
```

## Key distinction

```text
HA = survive component failure

DR = recover from major environment failure
```

A production organization normally needs both.

---

# 3. Why DR Is Required

Production systems depend on many layers:

```text
Users
  |
  v
DNS
  |
  v
ALB
  |
  v
EKS
  |
  +--> Applications
  |
  +--> Kubernetes services
  |
  +--> Secrets/configuration
  |
  v
Databases
  |
  v
Object storage
```

A single disaster can affect several layers simultaneously.

For example:

```text
AWS Region failure
        |
        +--> EKS unavailable
        +--> ALB unavailable
        +--> EC2 unavailable
        +--> NAT unavailable
        +--> regional database endpoints unavailable
```

Without a DR strategy, rebuilding manually may take many hours.

With Infrastructure as Code and GitOps:

```text
Terraform
   |
   v
Infrastructure
   |
   v
EKS
   |
   v
Argo CD
   |
   v
GitOps desired state
   |
   v
Applications
```

Recovery becomes repeatable.

---

# 4. Business Continuity vs Disaster Recovery

Business Continuity is broader than technical DR.

Business Continuity asks:

> How does the business continue operating?

DR asks:

> How do we restore the technology required by the business?

Example:

```text
Business Continuity
|
+-- Customer communication
+-- Support process
+-- Alternate operations
+-- Vendor coordination
+-- Financial process
|
+-- Disaster Recovery
    |
    +-- AWS
    +-- EKS
    +-- Databases
    +-- Applications
    +-- Networking
```

DevOps engineers are primarily responsible for the technical recovery portion but must understand business priorities.

---

# 5. Recovery Objectives

Two critical metrics are RPO and RTO.

---

# 6. RPO — Recovery Point Objective

RPO answers:

> How much data can the business afford to lose?

Example:

```text
Last successful backup = 10:00
Disaster = 10:20
Recovery starts = 10:30
```

If RPO is 30 minutes, losing data since 10:00 may be acceptable.

If RPO is 5 minutes, the architecture requires much more frequent replication.

## Example

```text
RPO = 15 minutes
```

Means:

> We should be able to recover with no more than approximately 15 minutes of data loss.

RPO is primarily a data-protection requirement.

---

# 7. RTO — Recovery Time Objective

RTO answers:

> How long can the service remain unavailable?

Example:

```text
Disaster detected: 10:00
Service restored: 10:45

RTO = 45 minutes
```

If the contractual RTO is 60 minutes, the recovery meets the target.

---

# 8. RPO vs RTO

| Metric | Question |
|---|---|
| RPO | How much data can we lose? |
| RTO | How long can the service be unavailable? |

Example:

```text
RPO = 15 minutes
RTO = 60 minutes
```

This means:

- Recover data to within approximately 15 minutes of the failure
- Restore service within 60 minutes

---

# 9. Example RoboShop DR Objectives

A realistic example for this capstone:

```text
Production EKS
RTO:
  Critical application: <= 60 minutes
  Non-critical observability: <= 4 hours

RPO:
  Critical database: <= 15 minutes
  Configuration: near-zero because Git is authoritative
  Container images: near-zero if replicated to DR ECR
  Logs: best effort depending on retention/replication
```

These are example targets.

Real values must be approved by business and application owners.

---

# 10. Tiered Recovery

Not every system has the same business importance.

Example:

```text
Tier 0
Critical customer traffic
RTO <= 30 min
RPO <= 5 min

Tier 1
Core application services
RTO <= 60 min
RPO <= 15 min

Tier 2
Operational tooling
RTO <= 4 hr
RPO <= 1 hr

Tier 3
Historical/non-critical data
RTO <= 24 hr
RPO <= 24 hr
```

This prevents overspending on systems that do not require aggressive recovery.

---

# 11. Production DR Architecture

A production architecture can use:

```text
                 Global DNS
                    |
          +---------+---------+
          |                   |
          v                   v
     Primary Region       DR Region
          |                   |
        ALB                 ALB
          |                   |
        EKS                 EKS
          |                   |
   +------+-----+      +------+-----+
   |            |      |            |
 Apps         Services Apps       Services
   |            |      |            |
   +------DB----+      +-----DB-----+
          |                   ^
          +---- Replication --+
```

Infrastructure is defined using Terraform.

Applications are deployed using GitOps.

Databases use native AWS backup/replication capabilities where appropriate.

Container images are replicated or independently available in the DR region.

---

# 12. Primary and DR Regions

Example:

```text
Primary:
us-east-1

DR:
us-west-2
```

The exact regions are organizational decisions.

Avoid choosing regions solely because they are geographically far apart.

Consider:

- Service availability
- Compliance
- Data residency
- Latency
- Cost
- AWS service parity
- Quotas
- Capacity
- DNS behavior
- Cross-region transfer cost

---

# 13. Multi-AZ Is Not Multi-Region

This distinction is extremely important.

Multi-AZ:

```text
Region
|
+-- AZ-A
+-- AZ-B
+-- AZ-C
```

Protects against:

- Individual AZ failure
- Some infrastructure failures
- Local network failures

It does not fully protect against:

- Region-wide outage
- Region-wide control-plane problems
- Regional service disruption
- Certain large-scale configuration failures

Multi-region:

```text
Region A
    |
    +-- AZ-A
    +-- AZ-B
    +-- AZ-C

Region B
    |
    +-- AZ-A
    +-- AZ-B
    +-- AZ-C
```

Provides stronger regional isolation.

---

# 14. Active-Active vs Active-Passive

Two common DR patterns are:

```text
Active-Active
```

Both regions serve traffic.

```text
Active-Passive
```

Primary serves traffic and DR remains ready for recovery.

---

# 15. Active-Active

```text
                DNS
              /     \
             v       v
         Region A  Region B
          50%        50%
```

Advantages:

- Faster failover
- Better resource utilization
- Continuous DR validation
- Lower recovery time

Disadvantages:

- Higher cost
- More operational complexity
- Data consistency becomes harder
- Cross-region networking is more complex

---

# 16. Active-Passive

```text
              DNS
               |
               v
          Primary Region
               |
            traffic

          DR Region
        standby/ready
```

Advantages:

- Lower cost
- Simpler
- Easier data consistency

Disadvantages:

- DR environment may drift
- Failover requires coordinated action
- Capacity must still be available
- Recovery testing is critical

For many organizations, active-passive is a reasonable starting point.

---

# 17. Warm Standby

A warm standby keeps core infrastructure ready but may run at reduced capacity.

Example:

```text
Primary EKS:
  10 nodes

DR EKS:
  2 nodes
```

During disaster:

```text
DR capacity
     |
     v
Scale up
     |
     v
Deploy workloads
     |
     v
Accept traffic
```

This reduces cost while maintaining relatively fast recovery.

---

# 18. Pilot Light

Pilot light keeps only essential components ready.

Example:

```text
DR:
  VPC
  IAM
  ECR
  Database replica
  Terraform
  GitOps
```

Application compute is created/scaled during recovery.

This is cheaper but has higher RTO.

---

# 19. Backup and Restore

Traditional backup architecture:

```text
Production
   |
   +--> Database backup
   +--> EBS snapshots
   +--> S3 versioning
   +--> Kubernetes backup
   +--> Configuration in Git
   |
   v
Backup storage
```

Backup alone does not guarantee DR.

You must test restore.

---

# 20. The Three Questions of Backup

For every backup, ask:

1. Is it being created?
2. Is it protected from deletion/corruption?
3. Can it actually be restored?

A backup that has never been restored in testing is an assumption, not proven recovery capability.

---

# 21. Backup Immutability

Production backups should be protected against:

- Accidental deletion
- Malicious deletion
- Compromised credentials
- Ransomware
- Operator mistakes

Use controls such as:

- S3 Object Lock where appropriate
- Backup vault controls
- Separate backup accounts
- Restricted IAM
- Versioning
- Cross-account backup copies
- Cross-region copies

---

# 22. AWS Backup Strategy

Use AWS-native backup services where they fit the workload.

Possible resources:

```text
EBS
RDS
DynamoDB
EFS
Aurora
Other supported resources
```

A centralized backup strategy can enforce:

- Backup schedules
- Retention
- Cross-region copies
- Vault policies
- Compliance reporting

---

# 23. EKS DR

An EKS disaster recovery strategy must distinguish between:

```text
EKS control plane
+
Worker compute
+
Kubernetes resources
+
Persistent application state
```

Do not assume that recreating the EKS cluster automatically recreates the entire application.

---

# 24. Rebuilding EKS from Terraform

A strong DR architecture stores the EKS definition in Terraform.

Example:

```text
terraform/
├── environments/
│   ├── prod-primary/
│   └── prod-dr/
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── iam/
│   ├── ecr/
│   └── observability/
└── backend/
```

Recovery:

```text
Terraform
   |
   v
VPC
   |
   v
EKS
   |
   v
IAM
   |
   v
ECR
   |
   v
Add-ons
```

---

# 25. EKS Cluster Recovery Principle

Do not treat the running cluster as the only source of truth.

The authoritative sources should be:

```text
Infrastructure:
Terraform

Application desired state:
GitOps repository

Container artifacts:
ECR

Secrets:
Approved secrets-management system

Data:
Database backup/replication
```

This is one of the most important production DR principles.

---

# 26. GitOps as a DR Mechanism

GitOps significantly improves recovery.

Suppose the entire production cluster disappears.

Recovery can be:

```text
Create VPC
   |
   v
Create EKS
   |
   v
Install Argo CD
   |
   v
Connect Git repository
   |
   v
Argo CD sync
   |
   v
Applications recreated
```

Without GitOps:

```text
Engineer remembers:
  deployment settings
  services
  ingress
  HPA
  RBAC
  config
  monitoring
  policies
```

That is fragile.

---

# 27. Git Repository DR

Git repositories must themselves be protected.

Controls:

- Multiple repository mirrors
- Organization-level backups
- Protected branches
- Restricted deletion permissions
- MFA/SSO
- Audit logs
- Repository export
- Separate recovery credentials

GitOps should not depend on one engineer's workstation.

---

# 28. Terraform State DR

Terraform state is critical.

If remote state is stored in S3:

```text
S3 bucket
   |
   +-- Versioning
   +-- Encryption
   +-- Restricted IAM
   +-- Replication/backup
   +-- Logging
```

For DynamoDB state locking where applicable:

```text
Terraform
    |
    +--> S3 state
    |
    +--> locking mechanism
```

Never casually delete Terraform state.

---

# 29. Terraform State Recovery

If state is accidentally modified:

```bash
aws s3api list-object-versions \
  --bucket <terraform-state-bucket> \
  --prefix prod/terraform.tfstate
```

Inspect versions before restoring.

Example:

```bash
aws s3api get-object \
  --bucket <terraform-state-bucket> \
  --key prod/terraform.tfstate \
  --version-id <VERSION_ID> \
  restored.tfstate
```

Use extreme care before replacing live state.

Always coordinate state recovery with the infrastructure owner.

---

# 30. ECR DR

Container images are application artifacts.

If primary ECR becomes unavailable:

```text
GitOps
  |
  v
Deployment
  |
  v
DR ECR
  |
  v
Pods
```

Possible strategies:

1. Cross-region ECR replication
2. Build and push to multiple regions
3. Replicate only production release images
4. Maintain immutable image tags/digests

---

# 31. Image Immutability

Avoid using:

```yaml
image: roboshop/catalog:latest
```

Prefer immutable versions:

```yaml
image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/catalog@sha256:<DIGEST>
```

or controlled immutable tags:

```yaml
image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/catalog:1.8.4
```

Digest pinning provides stronger reproducibility.

---

# 32. ECR Recovery Consideration

If an image is referenced by digest but the digest is unavailable in DR, deployment fails.

Therefore:

```text
Application artifact
       |
       +--> Primary ECR
       |
       +--> DR ECR
```

The artifact lifecycle must be part of the DR plan.

---

# 33. Database DR

Databases are usually the most important stateful component.

The recovery design depends on:

- Database engine
- Data volume
- Transaction rate
- RPO
- RTO
- Replication support
- Consistency requirements
- Backup method

Possible strategies:

```text
Backup/restore
Read replica
Cross-region replica
Global database
Continuous replication
```

---

# 34. Database Backup vs Replication

Backup:

```text
DB
 |
 v
Periodic snapshot
 |
 v
Backup storage
```

Replication:

```text
Primary DB
    |
    | continuous changes
    v
DR DB
```

Replication usually provides lower RPO.

Backup is still necessary because replication can copy corruption or accidental deletes.

---

# 35. Why Replication Alone Is Not Enough

Suppose:

```text
10:00 accidental DELETE
10:01 replication copies DELETE
10:02 primary fails
```

The DR database contains the same bad state.

Therefore production strategy should often combine:

```text
Replication
+
Point-in-time recovery
+
Snapshots
```

---

# 36. RDS/Aurora DR

For AWS-managed relational databases, consider:

- Automated backups
- Point-in-time recovery
- Cross-region read replicas
- Cross-region database features
- Snapshots
- Encryption
- KMS key strategy
- Parameter groups
- Subnet groups
- Security groups

Do not restore only the database data.

Also restore:

```text
Networking
Security
Parameter configuration
IAM integration
Secrets
Application connection configuration
```

---

# 37. Database Recovery Sequence

A safe recovery sequence:

```text
1. Declare disaster
2. Freeze conflicting writes if possible
3. Identify last known good point
4. Select recovery target
5. Restore/promote database
6. Validate schema
7. Validate critical tables
8. Validate application credentials
9. Start application
10. Run smoke tests
11. Redirect traffic
12. Monitor
```

---

# 38. Kubernetes Persistent Storage DR

For workloads using persistent storage:

```text
Pod
 |
 v
PVC
 |
 v
EBS/EFS
```

Deleting the pod does not necessarily delete the data.

But cluster recovery must ensure:

- Storage classes exist
- CSI driver exists
- IAM permissions exist
- Encryption keys exist
- Snapshot/backup exists
- PVC definitions are recoverable

---

# 39. EBS Snapshot Strategy

EBS volumes should be protected through appropriate snapshot policies.

Example conceptual workflow:

```text
Production EBS
      |
      v
Snapshot
      |
      v
Cross-region copy
      |
      v
DR Region
```

Snapshots are incremental, but recovery design must consider restore duration and volume size.

---

# 40. EFS DR

For EFS-based workloads, evaluate:

- Backup
- Replication
- Mount targets
- Security groups
- IAM
- KMS
- Network paths

Creating the filesystem without its network and IAM dependencies is incomplete recovery.

---

# 41. ELK DR

ELK contains operational logs.

Logs are valuable but may have lower business criticality than customer transaction data.

Possible strategy:

```text
Applications
   |
   v
Log shipping
   |
   v
ELK primary
   |
   +--> Snapshot
   |
   v
S3 backup
```

For Elasticsearch/OpenSearch-like systems, snapshot repositories can provide recoverability.

Do not assume Elasticsearch replicas alone equal DR.

---

# 42. Log Retention

Define:

```text
Hot retention
Warm retention
Cold retention
Archive retention
```

Example:

```text
Hot: 7 days
Warm: 30 days
Archive: 1 year
```

Actual retention depends on:

- Compliance
- Cost
- Incident requirements
- Storage
- Search requirements

---

# 43. Prometheus DR

Prometheus is primarily a monitoring data system.

For short-lived cluster recovery, dashboards and alert rules are often more important than old metrics.

Desired state should live in Git:

```text
PrometheusRule
Alertmanager
ServiceMonitor
Grafana dashboards
```

Historical metric data can be backed up where business requirements justify it.

---

# 44. Grafana DR

Grafana recovery should preserve:

- Dashboards
- Data sources
- Alert configuration if applicable
- Folders
- Permissions
- Provisioning configuration

Prefer dashboard-as-code/provisioning where practical.

Example:

```text
Git
 |
 +--> dashboards/
 +--> datasources/
 +--> alerting/
```

This makes Grafana rebuildable.

---

# 45. Alertmanager DR

Alertmanager configuration should be stored as code.

Protect:

```text
Routes
Receivers
Inhibition rules
Templates
Silence procedures
```

During DR, monitoring itself must not become a single point of failure.

---

# 46. Secrets DR

Secrets require special treatment.

Do not store production secrets in plain Git.

Recovery must account for:

- Secret manager
- KMS keys
- IAM permissions
- Service accounts
- External secret synchronization
- Database credentials
- TLS certificates
- Webhook credentials

Example:

```text
Argo CD
   |
   v
External Secrets
   |
   v
Secrets Manager
   |
   v
KMS
```

The DR region must be able to access the required secrets securely.

---

# 47. KMS DR

Encrypted backups are useless if the recovery environment cannot decrypt them.

Track:

```text
KMS key
|
+-- Key policy
+-- IAM permissions
+-- Grants
+-- Rotation
+-- Cross-region strategy
```

For multi-region DR, determine which KMS strategy is supported by each AWS service.

---

# 48. DNS DR

Traffic redirection can be performed using DNS mechanisms.

Possible patterns:

```text
DNS
 |
 +--> Primary
 |
 +--> DR
```

Routing strategies can include:

- Failover
- Weighted
- Latency
- Geolocation
- Health-check-based routing

DNS TTL must be considered.

Lower TTL can improve failover responsiveness but increases DNS query volume.

---

# 49. DNS Is Not Instantaneous

Changing a DNS record does not guarantee every client immediately uses the new destination.

Reasons include:

- Recursive resolver caching
- Client caching
- ISP behavior
- Application-level caching
- Existing connections

Therefore DNS failover time must be included in RTO calculations.

---

# 50. ALB DR

ALB is regional.

Therefore:

```text
Primary ALB
   |
   X regional outage
```

cannot simply be moved to another AWS region.

A DR region requires:

```text
DR VPC
  |
  v
DR EKS
  |
  v
DR ALB
```

DNS then redirects traffic.

---

# 51. AWS Load Balancer Controller

EKS commonly uses AWS Load Balancer Controller to create/manage ALBs.

The DR cluster needs:

- Controller installation
- IAM permissions
- OIDC/IRSA configuration
- Ingress resources
- Subnet tagging
- Security group strategy

GitOps should restore the Ingress resources.

---

# 52. Argo CD DR

Argo CD itself is not the source of truth.

Git is.

This distinction makes recovery easier.

```text
Git repository
      |
      v
Argo CD
      |
      v
EKS
```

If Argo CD is destroyed:

```text
Create EKS
   |
   v
Install Argo CD
   |
   v
Register cluster
   |
   v
Configure applications
   |
   v
Sync Git
```

---

# 53. Argo CD Multi-Cluster DR

If a central Argo CD manages multiple clusters:

```text
              Argo CD
             /       \
            v         v
        Cluster A  Cluster B
```

A DR design must consider what happens if the central Argo CD cluster fails.

Options:

- Rebuild Argo CD quickly
- Maintain backup of Argo CD configuration
- Store application definitions in Git
- Keep bootstrap manifests in a separate recovery repository
- Use a secondary management plane where justified

---

# 54. Argo CD Bootstrap

A useful recovery concept is:

```text
Terraform
  |
  v
EKS
  |
  v
Argo CD bootstrap
  |
  v
App-of-Apps / ApplicationSet
  |
  v
Applications
```

The bootstrap configuration should be small, version-controlled, and independently recoverable.

---

# 55. Kubernetes Resource Recovery

Kubernetes resources generally fall into:

```text
Stateless desired state
|
+-- Deployment
+-- Service
+-- Ingress
+-- HPA
+-- ConfigMap
+-- RBAC
+-- NetworkPolicy
+-- PrometheusRule

Stateful data
|
+-- PVC
+-- Database
+-- Object data
```

GitOps handles desired state.

Backup systems handle data.

---

# 56. What Git Can Restore

Git can restore:

- Deployments
- Services
- Ingress
- ConfigMaps
- RBAC
- NetworkPolicies
- HPAs
- PodDisruptionBudgets
- PrometheusRules
- Argo Applications
- Helm values

Git cannot magically restore:

- Database contents
- Persistent volume data
- External AWS resources not defined in code
- Lost secrets
- Deleted external artifacts

---

# 57. DR Dependency Graph

A production recovery dependency graph might be:

```text
AWS Account
    |
    v
IAM
    |
    v
KMS
    |
    v
VPC
    |
    +--> Subnets
    +--> Route tables
    +--> NAT
    +--> Security
    |
    v
EKS
    |
    +--> Add-ons
    +--> AWS Load Balancer Controller
    |
    v
Argo CD
    |
    v
GitOps
    |
    v
Applications
    |
    +--> ECR
    +--> Secrets
    +--> Database
```

The order matters.

---

# 58. Recovery Order

A typical recovery order:

```text
1. Incident command
2. AWS account/access
3. IAM/KMS
4. Networking
5. ECR/artifacts
6. EKS
7. Cluster add-ons
8. Secrets
9. Database
10. Argo CD
11. Application workloads
12. ALB/DNS
13. Monitoring
14. Logging
15. Validation
```

Exact ordering may vary depending on the failure.

---

# 59. DR Runbook Principle

Every recovery step should have:

```text
Owner
Command/procedure
Expected result
Failure condition
Rollback
Validation
```

Bad:

```text
Restore EKS.
```

Good:

```text
Provision DR VPC using Terraform.
Validate three AZs.
Validate private subnet routes.
Validate NAT.
Provision EKS.
Validate nodes Ready.
Install CSI.
Install ALB controller.
Bootstrap Argo CD.
Sync applications.
Restore database.
Run smoke tests.
Redirect DNS.
```

---

# 60. Example Terraform DR Configuration

A DR environment can use the same modules with different variables.

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = "roboshop-dr-vpc"

  region = var.aws_region

  availability_zones = [
    "${var.aws_region}a",
    "${var.aws_region}b",
    "${var.aws_region}c"
  ]

  enable_nat_gateway = true

  single_nat_gateway = false

  enable_flow_logs = true
}
```

The exact module implementation belongs in the Terraform chapter.

---

# 61. Production Terraform DR Variables

Example:

```hcl
variable "aws_region" {
  type        = string
  description = "AWS region for this environment."
}

variable "environment" {
  type        = string
  description = "Environment identifier."
}

variable "cluster_name" {
  type        = string
  description = "EKS cluster name."
}

variable "enable_dr_capacity" {
  type        = bool
  default     = true
}
```

Keep environment-specific values separate from reusable modules.

---

# 62. DR Backend Strategy

Never accidentally point primary and DR at the same state key.

Bad:

```text
prod/terraform.tfstate
prod/terraform.tfstate
```

Safer:

```text
prod-primary/terraform.tfstate
prod-dr/terraform.tfstate
```

Use separate state ownership and IAM permissions where practical.

---

# 63. EKS DR Terraform Example

Conceptual:

```hcl
module "eks" {
  source = "../../modules/eks"

  cluster_name    = var.cluster_name
  kubernetes_version = var.kubernetes_version

  vpc_id          = module.vpc.vpc_id
  private_subnets = module.vpc.private_subnet_ids

  managed_node_groups = {
    system = {
      desired_size = var.system_desired_size
      min_size     = var.system_min_size
      max_size     = var.system_max_size
      instance_types = ["m6i.large"]
    }

    application = {
      desired_size = var.application_desired_size
      min_size     = var.application_min_size
      max_size     = var.application_max_size
      instance_types = ["m6i.xlarge"]
    }
  }
}
```

Production values should be selected based on workload capacity testing.

---

# 64. Recovery Capacity Planning

Do not provision DR based only on CPU count.

Calculate:

```text
Expected traffic
        |
        v
Requests/sec
        |
        v
Pods required
        |
        v
CPU/memory
        |
        v
Node count
        |
        v
AZ distribution
```

Include:

- Peak traffic
- Pod requests
- DaemonSets
- System pods
- Autoscaling delay
- Cluster autoscaler/Karpenter behavior
- Reserved capacity

---

# 65. DR Quotas

A common recovery failure is AWS quota exhaustion.

Before DR testing verify:

```text
EC2 quotas
EBS quotas
Elastic IP quotas
ALB quotas
VPC quotas
NAT gateway limits
EKS limits
IAM limits
ECR limits
```

A Terraform plan can succeed but resource creation can still fail because of quotas.

---

# 66. DR IAM

Recovery engineers need enough access to restore systems but should not receive uncontrolled permanent administrator access.

Use:

- Break-glass role
- MFA
- Short-lived credentials
- Auditing
- Session logging
- Approval workflow
- Separate DR role

Example:

```text
Operator
   |
   v
MFA
   |
   v
DR Role
   |
   v
Recovery actions
```

---

# 67. Break-Glass Access

Break-glass credentials should be:

- Rarely used
- Strongly protected
- Audited
- Tested periodically
- Stored securely
- Independent of the failed environment where necessary

Do not discover during a disaster that nobody can access the AWS account.

---

# 68. Security During DR

Security requirements do not disappear during an outage.

Avoid:

```text
"Temporary" public database
"Temporary" 0.0.0.0/0
"Temporary" admin credentials
"Temporary" disabled TLS
```

Temporary insecure settings often become permanent production vulnerabilities.

---

# 69. DR Secrets Recovery

Example dependency:

```text
Application
   |
   v
External Secret
   |
   v
AWS Secrets Manager
   |
   v
KMS
   |
   v
IAM
```

All four layers must be recoverable.

---

# 70. Certificate DR

TLS certificates must be available in the DR region.

Depending on implementation:

- ACM certificates are regional
- DNS validation must remain available
- Ingress configuration must be reproducible
- Certificate issuance permissions must work

Therefore DR needs a certificate strategy.

---

# 71. Network DR

Rebuild:

```text
VPC
|
+-- Public subnets
+-- Private subnets
+-- Route tables
+-- Internet gateway
+-- NAT gateways
+-- Security groups
+-- Network ACLs
+-- VPC endpoints
+-- DNS settings
+-- Flow logs
```

Do not restore only EC2/EKS.

---

# 72. VPC Endpoint DR

If workloads use private AWS APIs through VPC endpoints:

```text
Pod
 |
 v
VPC Endpoint
 |
 v
AWS service
```

The DR VPC must contain equivalent endpoints where required.

Otherwise private workloads may fail to reach:

- ECR
- S3
- STS
- CloudWatch
- Secrets Manager
- KMS

---

# 73. NAT Gateway DR

If private subnets require internet access:

```text
Private subnet
     |
     v
NAT Gateway
     |
     v
Internet Gateway
```

For production HA, use NAT gateways across AZs.

For DR, ensure enough NAT capacity and correct routes.

---

# 74. EKS Add-ons During DR

Important add-ons can include:

- VPC CNI
- CoreDNS
- kube-proxy
- EBS CSI driver
- EFS CSI driver where required
- AWS Load Balancer Controller
- Metrics components
- Prometheus
- Grafana
- ELK agents

A cluster can show nodes as Ready while applications remain broken because a required add-on is missing.

---

# 75. CoreDNS DR

If CoreDNS is unhealthy:

```text
Pod
 |
DNS query
 |
X CoreDNS
```

Applications may fail even though services exist.

Validate:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Then:

```bash
kubectl get svc -n kube-system kube-dns
```

---

# 76. EBS CSI DR

Without EBS CSI:

```text
PVC
 |
X
 |
EBS volume cannot attach
```

Validate:

```bash
kubectl get pods -n kube-system | grep ebs-csi
```

Check:

```bash
kubectl get csidrivers
```

---

# 77. DR Monitoring

Monitoring must be restored early enough to validate recovery.

Minimum monitoring:

```text
Cluster health
Node health
Pod health
Application availability
Latency
Error rate
ALB target health
Database health
DNS
```

Prometheus and Grafana can then provide operational visibility.

---

# 78. DR Alerting

Alerting during recovery should avoid creating thousands of irrelevant alerts.

Use a recovery mode where appropriate.

Example:

```text
DR environment:
severity=warning
environment=dr
```

But critical safety alerts must remain enabled.

Do not simply disable Alertmanager.

---

# 79. ELK During Recovery

Logging is essential for diagnosis.

Application pods should continue writing structured logs.

Example:

```json
{
  "timestamp": "2026-08-31T10:30:00Z",
  "service": "catalog",
  "level": "ERROR",
  "message": "database connection failed",
  "environment": "dr"
}
```

---

# 80. DR Validation

Recovery is not complete when pods show Running.

Validate:

```text
DNS
  |
  v
ALB
  |
  v
Ingress
  |
  v
Service
  |
  v
Pod
  |
  v
Database
```

Then test:

- Login
- Product retrieval
- Cart
- Orders
- Payments
- User operations
- Critical APIs

---

# 81. Smoke Test Example

```bash
curl -f https://roboshop.example.com/health
```

Then test representative APIs:

```bash
curl -f https://roboshop.example.com/catalog/health
curl -f https://roboshop.example.com/cart/health
curl -f https://roboshop.example.com/order/health
```

Use application-specific endpoints.

---

# 82. Synthetic Monitoring

A strong DR validation approach uses synthetic transactions.

Example:

```text
Synthetic user
      |
      v
DNS
      |
      v
ALB
      |
      v
Application
      |
      v
Database
```

This tests the entire customer path.

---

# 83. DR Drill

A DR drill intentionally exercises recovery procedures without waiting for a real disaster.

Example:

```text
Saturday 09:00
Declare simulated region outage

09:05
Freeze deployments

09:10
Start DR provisioning

09:25
EKS ready

09:35
Database promoted

09:45
Argo CD synchronized

09:50
Smoke tests

10:00
Traffic switched
```

Then compare actual performance with RTO.

---

# 84. DR Drill Types

Useful exercises:

### Tabletop

People walk through the scenario without changing infrastructure.

### Component failure

Simulate a specific failure.

### Cluster rebuild

Destroy/recreate a non-production cluster.

### Region failover

Test actual cross-region traffic.

### Database restore

Restore production-like data into isolated environment.

### Full DR exercise

Exercise the entire recovery chain.

---

# 85. Game Days

A game day is a controlled resilience exercise.

Examples:

```text
Kill worker nodes
Break ingress
Delete deployment
Expire credentials
Block database
Simulate region outage
Corrupt configuration
```

The objective is to discover weaknesses before real incidents.

---

# 86. DR Test Evidence

Record:

```text
Test date
Scenario
Participants
Start time
Detection time
Recovery start
Service restored
RTO achieved
RPO achieved
Problems
Actions
Owner
Due date
```

This turns DR from documentation into measurable engineering.

---

# 87. Failover

Failover means moving service from primary to DR.

Typical sequence:

```text
Detect disaster
    |
    v
Declare disaster
    |
    v
Freeze primary changes
    |
    v
Verify DR readiness
    |
    v
Promote database
    |
    v
Scale DR
    |
    v
Sync applications
    |
    v
Validate
    |
    v
Change DNS
    |
    v
Monitor
```

---

# 88. Failback

Failback means returning service to the primary environment.

Never immediately switch back.

First:

```text
Repair primary
   |
   v
Rebuild missing resources
   |
   v
Synchronize data
   |
   v
Validate primary
   |
   v
Test application
   |
   v
Switch traffic gradually
```

---

# 89. Failback Data Risk

Suppose DR accepted writes during an outage.

Then:

```text
Primary database
    old data

DR database
    newer data
```

Failback requires data synchronization.

Do not simply restore the old primary backup over the new DR data.

---

# 90. Failback Procedure

Example:

```text
1. Announce failback window
2. Reduce write traffic if required
3. Synchronize database
4. Verify replication lag = acceptable
5. Validate primary infrastructure
6. Deploy application
7. Run smoke tests
8. Redirect traffic
9. Monitor
10. Keep DR ready
```

---

# 91. Split-Brain

Split-brain occurs when both environments believe they should accept authoritative writes.

Example:

```text
Primary DB <---- writes
      ^
      |
      |
      v
DR DB     <---- writes
```

This can cause conflicting data.

DR architecture must define exactly which side is authoritative.

---

# 92. Deployment Freeze During DR

During a disaster:

```text
CI/CD
  |
  X deployment freeze
```

Unless a deployment is explicitly required for recovery.

Reasons:

- Reduce variables
- Prevent accidental changes
- Improve incident investigation
- Preserve reproducibility

Emergency changes should be documented.

---

# 93. GitOps During DR

GitOps can continue to enforce desired state, but operators must avoid fighting the recovery process.

Example:

```text
Engineer manually scales DR
          |
          v
Argo CD desired replicas = 3
          |
          v
Argo CD scales back
```

Therefore emergency operational changes must be reconciled with Git.

---

# 94. DR and Git Repositories

A disaster can affect source control access.

Keep recovery bootstrap information available through:

- Secondary Git mirror
- Secure repository export
- Offline emergency documentation
- Protected backup
- Artifact storage

The goal is not to duplicate all repositories manually forever; it is to ensure recovery does not depend on one unavailable system.

---

# 95. CI/CD DR

CI must remain recoverable.

If Jenkins is unavailable:

```text
GitHub Actions
```

may be an alternative if the organization supports it.

However, emergency recovery should not depend on rebuilding the entire CI system before production can start.

Production images should already exist in ECR.

---

# 96. Jenkins DR

If Jenkins is used:

Protect:

```text
Jenkinsfile
Job configuration
Plugins list
Credentials references
Agents configuration
Shared libraries
```

Prefer pipeline-as-code.

Avoid making critical pipeline behavior exist only in Jenkins UI.

---

# 97. Artifact DR

Recovery should not require rebuilding software from source.

Ideal:

```text
Approved release image
        |
        +--> Primary ECR
        |
        +--> DR ECR
```

Rebuilding during a disaster introduces:

- Dependency changes
- Registry failures
- Build failures
- New vulnerabilities
- Non-reproducible artifacts

Use known-good immutable artifacts.

---

# 98. Security Scanning and DR

Production recovery should use already approved artifacts.

If emergency image rebuild is required:

```text
Build
 |
 v
Unit tests
 |
 v
SonarQube
 |
 v
Trivy
 |
 v
Veracode where required
 |
 v
ECR
```

Do not bypass security controls casually.

---

# 99. DR and Supply Chain

A supply-chain attack during a disaster is especially dangerous because teams are under pressure.

Use:

- Signed/verified artifacts where supported
- Immutable tags
- Digests
- ECR controls
- Restricted deployment roles
- Git protection
- Audit logs

---

# 100. Disaster Declaration

Not every incident is a disaster.

Examples:

```text
Single pod failure
     = normal incident

Node failure
     = normal HA recovery

Entire AZ failure
     = major incident

Regional outage
     = disaster

Database corruption affecting all production
     = potential disaster
```

Define objective declaration criteria.

---

# 101. Disaster Commander

A disaster should have one accountable incident commander.

Example roles:

```text
Incident Commander
|
+-- Cloud/AWS Lead
+-- Kubernetes Lead
+-- Database Lead
+-- Application Lead
+-- Network/DNS Lead
+-- Security Lead
+-- Communications Lead
```

Avoid ten people independently changing infrastructure.

---

# 102. Communication

During DR, communication should include:

```text
What happened?
What is affected?
What is the business impact?
What recovery stage are we in?
What is the current ETA?
Who owns each action?
```

Keep technical and executive communications appropriate to their audience.

---

# 103. DR Change Control

Emergency changes should still be recorded.

Minimum:

```text
Timestamp
Operator
Action
Reason
Command/change
Result
```

This helps post-incident reconstruction.

---

# 104. DR Architecture Review Checklist

Review:

```text
[ ] RPO defined
[ ] RTO defined
[ ] Critical services identified
[ ] Dependencies documented
[ ] Primary region defined
[ ] DR region defined
[ ] Terraform available
[ ] Terraform state protected
[ ] GitOps repository protected
[ ] ECR replication available
[ ] Database replication/backup tested
[ ] Secrets recoverable
[ ] KMS recoverable
[ ] DNS failover tested
[ ] EKS rebuild tested
[ ] ALB recreated
[ ] Monitoring restored
[ ] Logging restored
[ ] Smoke tests documented
[ ] DR drill completed
```

---

# 105. Production DR YAML — Argo CD Application

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-prod-dr
  namespace: argocd
  labels:
    environment: dr
    team: platform
spec:
  project: production
  source:
    repoURL: https://git.example.com/platform/roboshop-gitops.git
    targetRevision: main
    path: environments/dr
  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Important points:

- `environment: dr` makes ownership explicit.
- `source.repoURL` identifies the GitOps source.
- `path` selects DR-specific configuration.
- `prune` removes resources no longer desired.
- `selfHeal` corrects drift.

---

# 106. DR-Specific Helm Values

Example:

```yaml
global:
  environment: dr
  region: us-west-2

catalog:
  replicaCount: 2

cart:
  replicaCount: 2

order:
  replicaCount: 2

payment:
  replicaCount: 2

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20

ingress:
  enabled: true
  scheme: internet-facing
```

During recovery, values should be controlled through Git.

---

# 107. DR Namespace

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: dr
    managed-by: argocd
```

Do not create production namespaces manually unless emergency procedures require it.

---

# 108. DR NetworkPolicy

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Then explicitly allow required traffic.

Security posture should remain consistent in DR.

---

# 109. DR PDB

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: catalog
  namespace: roboshop
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: catalog
```

PDBs protect availability during controlled disruptions.

---

# 110. DR Service

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalog
  namespace: roboshop
spec:
  selector:
    app: catalog
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

---

# 111. DR Ingress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /health
spec:
  ingressClassName: alb
  rules:
    - host: roboshop-dr.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: catalog
                port:
                  number: 80
```

Production DNS strategy determines how the DR hostname is exposed.

---

# 112. DR Monitoring Rule

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: dr-cluster-alerts
  namespace: monitoring
  labels:
    release: prometheus
    environment: dr
spec:
  groups:
    - name: dr.cluster
      rules:
        - alert: DRNodeNotReady
          expr: kube_node_status_condition{
            condition="Ready",
            status="true"
          } == 0
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: dr
          annotations:
            summary: "DR Kubernetes node is not Ready"
            description: "A DR node has not reported Ready for more than 10 minutes."
```

---

# 113. DR Alerting Principle

During failover:

```text
Critical recovery alerts
       |
       v
Remain active

Non-actionable noise
       |
       v
Group/suppress appropriately
```

Never suppress alerts that indicate the DR environment itself is unhealthy.

---

# 114. DR Validation Commands

Check cluster:

```bash
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A
```

Check critical workloads:

```bash
kubectl get deployments -n roboshop
kubectl get pods -n roboshop
kubectl get svc -n roboshop
kubectl get ingress -n roboshop
```

---

# 115. Check Kubernetes Events

```bash
kubectl get events -A \
  --sort-by=.lastTimestamp
```

Look for:

- FailedScheduling
- FailedMount
- ImagePullBackOff
- CrashLoopBackOff
- FailedAttachVolume
- FailedCreatePodSandBox

---

# 116. Check ALB

Use AWS CLI:

```bash
aws elbv2 describe-load-balancers \
  --region <DR_REGION>
```

Then:

```bash
aws elbv2 describe-target-groups \
  --region <DR_REGION>
```

Then inspect target health:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN> \
  --region <DR_REGION>
```

---

# 117. Check ECR

```bash
aws ecr describe-repositories \
  --region <DR_REGION>
```

Check images:

```bash
aws ecr list-images \
  --repository-name catalog \
  --region <DR_REGION>
```

---

# 118. Check EKS

```bash
aws eks describe-cluster \
  --name <DR_CLUSTER> \
  --region <DR_REGION>
```

Then:

```bash
aws eks list-nodegroups \
  --cluster-name <DR_CLUSTER> \
  --region <DR_REGION>
```

---

# 119. Check DNS

Example:

```bash
dig roboshop.example.com
```

or:

```bash
nslookup roboshop.example.com
```

Verify that the returned destination matches the intended DR endpoint.

---

# 120. Check Application Health

```bash
curl -I https://roboshop.example.com
```

For API health:

```bash
curl -fsS https://roboshop.example.com/health
```

Use `-f` so HTTP failures produce a non-zero exit code.

---

# 121. Recovery Failure — EKS Created but Pods Pending

Symptom:

```text
EKS = healthy
Pods = Pending
```

Investigation:

```bash
kubectl describe pod <POD> -n roboshop
```

Likely causes:

- Insufficient capacity
- Wrong node selectors
- Taints
- Missing node group
- Availability zone constraints
- PVC unavailable

Prevention:

- Test DR capacity
- Validate scheduling rules
- Maintain node groups
- Monitor quotas

---

# 122. Recovery Failure — ImagePullBackOff

Symptom:

```text
ImagePullBackOff
```

Investigation:

```bash
kubectl describe pod <POD> -n roboshop
```

Likely causes:

- Image absent from DR ECR
- Incorrect repository URI
- Node IAM problem
- ECR network endpoint unavailable
- Image digest missing

Fix:

```text
Replicate image
Correct repository
Fix IAM
Validate ECR connectivity
```

---

# 123. Recovery Failure — PVC Pending

Symptom:

```text
PVC = Pending
```

Investigate:

```bash
kubectl describe pvc <PVC> -n roboshop
```

Check:

```bash
kubectl get storageclass
kubectl get csidrivers
```

Likely causes:

- CSI driver missing
- Storage class missing
- IAM missing
- Snapshot unavailable
- AZ mismatch

---

# 124. Recovery Failure — Database Connection Failed

Symptom:

```text
Application = Running
API = 500
```

Logs:

```bash
kubectl logs deploy/catalog -n roboshop
```

Possible root causes:

- Secret not restored
- Wrong DB endpoint
- Security group
- Network route
- DNS
- DB not promoted
- TLS configuration

Running pods does not prove application recovery.

---

# 125. Recovery Failure — ALB Has No Healthy Targets

Check:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <ARN> \
  --region <REGION>
```

Then:

```bash
kubectl get ingress -n roboshop
kubectl get svc -n roboshop
kubectl get pods -n roboshop
```

Common causes:

- Health check path wrong
- Target port wrong
- Security group
- Pod not Ready
- Application listening on different port

---

# 126. Recovery Failure — DNS Still Points Primary

Symptom:

```text
DR healthy
Users still receive errors
```

Check:

```bash
dig roboshop.example.com
```

Then inspect DNS routing and TTL.

Possible causes:

- Record not updated
- Health check not healthy
- Resolver cache
- Wrong hosted zone
- Traffic policy incorrect

---

# 127. Recovery Failure — Argo CD Sync Fails

Check:

```bash
argocd app get roboshop-prod-dr
```

Possible causes:

- Git repository unavailable
- Invalid manifests
- Cluster authentication issue
- Missing namespace
- Missing CRD
- Missing secret
- Resource health failure

---

# 128. Recovery Failure — Prometheus Missing

If Prometheus is not available:

```text
Application may work
but observability is degraded
```

Recovery should prioritize:

```text
Application health
+
ALB
+
Database
+
Basic monitoring
```

Then restore historical monitoring.

---

# 129. Recovery Failure — KMS AccessDenied

Symptom:

```text
AccessDeniedException
```

Check:

```text
IAM policy
KMS key policy
Region
Role
Trust relationship
```

Do not create unencrypted replacement data merely to bypass the issue.

---

# 130. Recovery Failure — AWS Quota

Symptom:

```text
ResourceLimitExceeded
```

Check:

```bash
aws service-quotas list-service-quotas \
  --service-code ec2 \
  --region <DR_REGION>
```

Prevention:

- Request quotas before disaster
- Include quota validation in DR drills
- Document expected capacity

---

# 131. Recovery Failure — Terraform Drift

After recovery:

```bash
terraform plan
```

If unexpected changes appear:

```text
Do not immediately apply.
```

Investigate:

- Manual changes
- Wrong state
- Wrong account
- Wrong region
- Wrong workspace
- Missing import

---

# 132. Production DR Checklist

## Before Disaster

```text
[ ] RTO approved
[ ] RPO approved
[ ] DR region selected
[ ] Terraform tested
[ ] State protected
[ ] GitOps protected
[ ] ECR replication tested
[ ] Database backup tested
[ ] Database replication tested
[ ] Secrets tested
[ ] KMS tested
[ ] DNS failover tested
[ ] EKS rebuild tested
[ ] ALB tested
[ ] Monitoring tested
[ ] Logging tested
[ ] Runbook approved
[ ] Contacts current
```

## During Disaster

```text
[ ] Declare incident
[ ] Assign commander
[ ] Freeze deployments
[ ] Assess primary
[ ] Validate DR
[ ] Recover database
[ ] Recover EKS
[ ] Bootstrap Argo CD
[ ] Deploy workloads
[ ] Validate
[ ] Switch DNS
[ ] Monitor
```

## After Disaster

```text
[ ] Stabilize
[ ] Verify data integrity
[ ] Continue monitoring
[ ] Document timeline
[ ] Plan failback
[ ] Execute failback
[ ] Review gaps
[ ] Update runbook
[ ] Repeat DR test
```

---

# 133. Production DR Architecture — Complete View

```text
                       Users
                         |
                         v
                 Global DNS / Routing
                    /            \
                   /              \
                  v                v
          PRIMARY REGION        DR REGION
          --------------        ----------
               |                    |
              ALB                  ALB
               |                    |
              EKS                  EKS
               |                    |
        +------+-------+      +-----+------+
        |              |      |            |
   Applications    Platform Applications Platform
        |              |      |            |
        +-------> Database <---+            |
                    ^                       |
                    |                       |
             Replication/Backup             |
                    |                       |
             Backup Storage <---------------+
                    |
                    v
              Recovery Data

Terraform
   |
   +--> Primary infrastructure
   +--> DR infrastructure

Git
   |
   v
GitOps
   |
   +--> Argo CD
   +--> Primary EKS
   +--> DR EKS

ECR
   |
   +--> Primary region
   +--> DR region

Prometheus/Grafana
   |
   +--> Primary observability
   +--> DR observability

ELK
   |
   +--> Centralized logs
   +--> Snapshot/archive
```

---

# 134. Senior DevOps DR Mental Model

When asked:

> "How would you design disaster recovery for an EKS production platform?"

A strong answer is:

```text
First define business RTO/RPO.

Then identify stateful dependencies.

Use Terraform so infrastructure can be recreated.

Use GitOps so Kubernetes desired state is reproducible.

Replicate approved container images to the DR region.

Use database-native replication plus backups/PITR.

Protect secrets and KMS dependencies.

Create equivalent networking, EKS, ALB, and security controls in DR.

Use DNS-based traffic failover.

Test the complete recovery path regularly.

Measure actual RTO/RPO rather than relying on documentation.

Finally, document failback and data synchronization.
```

---

# 135. Interview Question — What is RPO?

**Answer:**

RPO is Recovery Point Objective.

It defines the maximum acceptable amount of data loss measured backward from the disaster.

For example:

```text
RPO = 15 minutes
```

means the recovery strategy should allow recovery to a point no more than approximately 15 minutes behind the failure.

---

# 136. Interview Question — What is RTO?

**Answer:**

RTO is Recovery Time Objective.

It defines how quickly a service must be restored after a disaster.

Example:

```text
RTO = 60 minutes
```

means the service should be restored within approximately one hour.

---

# 137. Interview Question — Is Multi-AZ a DR Strategy?

**Answer:**

Multi-AZ is primarily a high-availability strategy.

It protects against Availability Zone failures but does not provide complete protection against a regional outage.

For regional disaster recovery, I would use a second AWS region with independently recoverable infrastructure, data, artifacts, secrets, and DNS failover.

---

# 138. Interview Question — How Would You Recover EKS?

**Answer:**

I would not manually recreate the cluster.

I would:

1. Provision the DR VPC using Terraform.
2. Create EKS using Terraform.
3. Install required add-ons.
4. Configure IAM and KMS.
5. Configure ECR access.
6. Bootstrap Argo CD.
7. Connect it to the GitOps repository.
8. Restore or promote databases.
9. Deploy applications from Git.
10. Validate ALB, DNS, application health, and monitoring.
11. Redirect traffic.

---

# 139. Interview Question — Why Is GitOps Useful for DR?

**Answer:**

GitOps makes Kubernetes desired state reproducible.

Instead of relying on what happened to exist inside the failed cluster, the desired configuration is stored in Git.

After rebuilding EKS, Argo CD can reconcile that desired state into the new cluster.

This reduces manual recovery and configuration drift.

---

# 140. Interview Question — Is Database Replication Enough?

**Answer:**

No.

Replication can reproduce accidental deletes or corrupted data.

I would use replication for low RPO combined with backups and point-in-time recovery for protection against logical corruption and operator mistakes.

---

# 141. Interview Question — How Would You Test DR?

**Answer:**

I would perform regular DR drills.

For example:

```text
Simulate primary region failure
        |
        v
Start recovery timer
        |
        v
Provision/recover DR
        |
        v
Promote database
        |
        v
Deploy via Argo CD
        |
        v
Run synthetic transactions
        |
        v
Switch DNS
        |
        v
Measure RTO/RPO
```

I would record failures and convert them into remediation actions.

---

# 142. Interview Question — What Happens If Argo CD Is Lost?

**Answer:**

Argo CD is not the source of truth.

Git is.

I would rebuild EKS, install Argo CD, restore its bootstrap configuration, reconnect the GitOps repository, and allow Argo CD to recreate the desired resources.

---

# 143. Interview Question — What Happens If ECR Is Lost?

**Answer:**

Production release images should be available in the DR region through replication or another controlled artifact strategy.

I would use immutable image tags or digests and verify that the referenced artifact exists in DR before starting application recovery.

---

# 144. Interview Question — What Happens If Terraform State Is Lost?

**Answer:**

I would not immediately recreate resources.

First I would recover the previous state version from protected remote state backups/versioning.

Then I would validate the recovered state against the actual AWS environment.

If resources exist but state is missing, I would carefully import resources rather than allowing Terraform to create duplicates.

---

# 145. Interview Question — How Do You Prevent Split Brain?

**Answer:**

The DR design must define one authoritative writer.

During failover, the primary write path must be stopped or fenced before DR becomes authoritative.

Before failback, data synchronization must be completed and the primary must be validated.

---

# 146. Interview Question — Why Is DR Testing Important?

**Answer:**

Because documentation can be wrong.

A backup may be corrupted.

An IAM role may be missing.

A KMS key may be inaccessible.

A Terraform module may be broken.

An ECR image may not exist in DR.

A DNS record may not fail over.

Testing exposes these failures before a real disaster.

---

# 147. Interview Question — What Is Warm Standby?

**Answer:**

Warm standby maintains a partially running DR environment.

Core infrastructure and critical dependencies are ready, but capacity may be lower than production.

During failover, the environment is scaled and applications are activated.

It provides a balance between cost and recovery speed.

---

# 148. Interview Question — Active-Active vs Active-Passive?

**Answer:**

Active-active runs workloads in both regions and can provide faster failover but requires more infrastructure and data-consistency complexity.

Active-passive keeps one region primary and another ready for recovery.

The choice depends on business RTO/RPO, budget, data architecture, and operational maturity.

---

# 149. Interview Question — What Should Be Stored in Git for DR?

**Answer:**

I would store declarative configuration such as:

```text
Terraform
Helm charts
Kubernetes manifests
Argo CD Applications
PrometheusRule
Alertmanager configuration
Grafana provisioning
Network policies
RBAC
Application configuration
```

I would not store plaintext production credentials.

Stateful data and secrets require dedicated recovery mechanisms.

---

# 150. Interview Question — What Is Your DR Priority?

A practical priority is:

```text
1. Business-critical data
2. Network and IAM foundations
3. Compute platform
4. Application dependencies
5. Customer-facing applications
6. Traffic routing
7. Monitoring/logging
8. Historical observability
```

The exact order depends on application dependencies.

---

# 151. Final Production DR Principles

Remember these principles:

```text
1. Define RTO and RPO first.
2. HA is not the same as DR.
3. Multi-AZ is not multi-region.
4. Terraform should rebuild infrastructure.
5. GitOps should rebuild Kubernetes desired state.
6. ECR artifacts must be recoverable.
7. Databases need replication and backups.
8. Secrets and KMS are DR dependencies.
9. DNS failover must be tested.
10. ALB is regional.
11. Argo CD is not the source of truth; Git is.
12. Backups must be restore-tested.
13. DR capacity must be tested.
14. Quotas must be checked.
15. Security must remain enabled during recovery.
16. Freeze unnecessary deployments during incidents.
17. Prevent split-brain.
18. Failback requires data synchronization.
19. Measure actual RTO/RPO.
20. Run DR drills regularly.
```

---

# 152. Final End-to-End Recovery Scenario

Assume the production AWS region becomes unavailable.

The recovery process is:

```text
REGIONAL OUTAGE
      |
      v
Incident Commander
      |
      v
Declare Disaster
      |
      v
Freeze Deployments
      |
      v
Validate DR Region
      |
      v
Terraform
      |
      v
VPC + IAM + KMS
      |
      v
EKS
      |
      v
Cluster Add-ons
      |
      v
ECR Artifacts
      |
      v
Database Promotion/Restore
      |
      v
Secrets
      |
      v
Argo CD
      |
      v
GitOps
      |
      v
Applications
      |
      v
ALB
      |
      v
Health Checks
      |
      v
Synthetic Transactions
      |
      v
DNS Failover
      |
      v
Production Traffic
      |
      v
Prometheus + Grafana
      |
      v
ELK
      |
      v
Continuous Monitoring
```

The recovery is successful only when the business transaction path works.

```text
DNS
 |
 v
ALB
 |
 v
EKS
 |
 v
Application
 |
 v
Database
 |
 v
Successful customer transaction
```

That is the real definition of production recovery.

---

# 153. Chapter Summary

A mature production DevOps environment treats disaster recovery as an engineered capability, not a document.

For this RoboShop production capstone:

```text
AWS
 |
 +--> Primary Region
 |
 +--> DR Region
 |
 +--> VPC
 +--> EKS
 +--> ALB
 +--> ECR
 |
Terraform
 |
 v
Infrastructure Recovery

Git
 |
 v
GitOps
 |
 v
Argo CD
 |
 v
Kubernetes Recovery

Database
 |
 +--> Replication
 +--> Backup
 +--> PITR

Observability
 |
 +--> Prometheus
 +--> Grafana
 +--> ELK

Traffic
 |
 v
DNS Failover
```

The most important production mindset is:

> **Do not ask whether you have backups. Ask whether you can repeatedly restore the entire service within the required RTO and with the required RPO.**

That distinction separates a documented DR plan from a tested production recovery capability.
