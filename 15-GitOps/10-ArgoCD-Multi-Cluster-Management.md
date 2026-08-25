# 10-ArgoCD-Multi-Cluster-Management

## 1. Purpose

Argo CD is not limited to deploying applications into the Kubernetes cluster where Argo CD itself is installed.

A production Argo CD installation can operate as a centralized GitOps control plane for multiple Kubernetes clusters, including multiple AWS EKS clusters.

This file covers:

- Centralized Argo CD
- Management cluster
- Target clusters
- Cluster registration
- EKS authentication
- Cluster credentials
- Kubernetes RBAC
- AWS IAM boundaries
- Multi-environment management
- Multi-region management
- Multi-account AWS
- ApplicationSet integration
- Cluster labels
- Cluster onboarding
- Cluster removal
- Centralized vs decentralized Argo CD
- Hub-and-spoke architecture
- Security
- High availability
- Failure scenarios
- Disaster recovery
- Production YAMLs
- RoboShop
- Troubleshooting
- Interview preparation

---

# 2. Core Concept

A single Argo CD installation can manage:

```text
Argo CD Management Cluster
        |
        +--> EKS DEV
        +--> EKS QA
        +--> EKS PROD
        +--> EKS PROD-DR
```

The Argo CD installation is the control plane.

The target clusters are where applications run.

---

# 3. Management Cluster vs Target Cluster

## Management Cluster

The management cluster hosts:

```text
Argo CD
Application Controller
ApplicationSet Controller
Repo Server
API Server
Redis
Projects
Application definitions
```

It is responsible for GitOps control.

---

## Target Cluster

Target clusters host:

```text
RoboShop workloads
Deployments
Pods
Services
Ingress
HPA
ConfigMaps
Secrets
```

They are controlled by Argo CD.

---

# 4. Basic Architecture

```text
                     Git Repository
                           |
                           v
                    +-------------+
                    |  Argo CD    |
                    | Management  |
                    |   Cluster   |
                    +-------------+
                           |
              +------------+------------+
              |            |            |
              v            v            v
          EKS-DEV       EKS-QA       EKS-PROD
              |            |            |
          Workloads    Workloads    Workloads
```

---

# 5. Important Mental Model

Argo CD does not move an application image from one cluster to another.

Instead:

```text
Git
 |
 v
Desired state
 |
 v
Argo CD
 |
 v
Target Kubernetes API
 |
 v
Kubernetes controllers
 |
 v
Pods
```

Argo CD tells the target cluster what Kubernetes resources should exist.

---

# 6. Control Plane Responsibility

Central Argo CD is responsible for:

```text
Git synchronization
Manifest generation
Application reconciliation
Cluster communication
Application health
Sync status
Drift detection
ApplicationSet generation
```

It is not responsible for:

```text
Running application containers
Replacing kube-scheduler
Replacing kube-controller-manager
Building Docker images
Building application binaries
```

---

# 7. Target Cluster Responsibility

The target Kubernetes cluster remains responsible for:

```text
Scheduling
Replica management
Service endpoints
Pod lifecycle
Node management
Container execution
HPA behavior
Ingress controller behavior
Storage controllers
```

Argo CD supplies desired state.

Kubernetes operates that state.

---

# 8. Centralized Multi-Cluster Flow

```text
Developer
   |
   v
Git
   |
   v
Central Argo CD
   |
   +----------------+----------------+
   |                |                |
   v                v                v
EKS DEV           EKS QA           EKS PROD
   |                |                |
Pods              Pods             Pods
```

---

# 9. Why Organizations Use Centralized Argo CD

Centralized management provides:

```text
One UI
One CLI endpoint
Central application inventory
Central GitOps policy
Central RBAC
Central audit visibility
Central ApplicationSet management
Standardized deployment workflows
```

This can be especially useful when an organization has:

```text
Many AWS accounts
Many EKS clusters
Many environments
Many teams
```

---

# 10. The Tradeoff

Centralization also creates a larger blast radius.

If the Argo CD management cluster is unavailable:

```text
Existing workloads usually continue running.
```

But:

```text
New deployments
Drift correction
Application synchronization
ApplicationSet generation
```

may be interrupted until Argo CD recovers.

Therefore central Argo CD requires strong:

```text
HA
Security
Backup
Monitoring
DR
```

---

# 11. Hub-and-Spoke Architecture

A common model is:

```text
                    Central Argo CD
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
      EKS DEV          EKS QA          EKS PROD
```

Argo CD is the hub.

Clusters are spokes.

---

# 12. Multi-Region Hub-and-Spoke

Example:

```text
                      Argo CD
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
     ap-south-1     ap-southeast-1   us-east-1
        EKS             EKS             EKS
```

Useful for:

```text
Global applications
Disaster recovery
Regional workloads
Latency optimization
Business continuity
```

---

# 13. Multi-Account Architecture

A larger enterprise may use:

```text
AWS Organization
       |
       +--> DEV Account
       |      |
       |      +--> EKS DEV
       |
       +--> QA Account
       |      |
       |      +--> EKS QA
       |
       +--> PROD Account
              |
              +--> EKS PROD
```

Central Argo CD can manage these clusters when appropriate authentication and network connectivity are established.

---

# 14. Network Connectivity

Central Argo CD must reach target Kubernetes APIs.

Possible architecture:

```text
Argo CD VPC
      |
      +--> Transit Gateway
      |
      +--> VPC Peering
      |
      +--> Private connectivity
      |
      v
Target EKS VPC
```

The exact AWS network design depends on organizational requirements.

---

# 15. Public vs Private EKS API

An EKS cluster can expose its Kubernetes API using different endpoint configurations.

For production, many organizations prefer private API access or tightly restricted public access.

If Argo CD cannot reach the API endpoint:

```text
Cluster registration may succeed initially
```

but synchronization can fail later.

Connectivity must be validated.

---

# 16. Network Security Groups

If using private connectivity, ensure:

```text
Route tables
Security groups
Network ACLs
DNS
Transit Gateway/VPC routing
```

allow the required traffic.

Do not expose Kubernetes APIs broadly just to make Argo CD connectivity easier.

---

# 17. DNS Considerations

Private EKS API access may depend on:

```text
VPC DNS
Route 53 Resolver
Private DNS
Cross-VPC DNS resolution
```

A common failure is:

```text
Network route exists
but DNS resolution fails
```

Always test both.

---

# 18. Basic Connectivity Test

From an environment with the appropriate credentials:

```bash
kubectl cluster-info
```

For EKS:

```bash
aws eks describe-cluster \
  --name <cluster-name> \
  --region <region>
```

Then test Kubernetes access:

```bash
kubectl get nodes
```

---

# 19. EKS Authentication

Argo CD needs valid authentication to the Kubernetes API.

Modern EKS authentication commonly involves AWS IAM-based access mechanisms.

The exact setup depends on the EKS authentication mode and Argo CD version.

The important principle is:

```text
AWS IAM identity
       |
       v
EKS authentication
       |
       v
Kubernetes authorization
       |
       v
Allowed resources
```

Authentication and authorization are separate.

---

# 20. Authentication vs Authorization

Authentication asks:

```text
Who are you?
```

Authorization asks:

```text
What are you allowed to do?
```

For EKS:

```text
IAM / EKS access mechanism
```

helps establish identity.

Kubernetes RBAC determines permissions for Kubernetes operations.

---

# 21. Least Privilege

Avoid blindly using:

```text
cluster-admin
```

for every Argo CD target.

Instead define:

```text
Required API groups
Required resources
Required namespaces
Required verbs
```

and restrict access.

However, platform-level Argo CD installations may legitimately require broader permissions depending on the resources they manage.

---

# 22. Namespace-Scoped vs Cluster-Scoped Management

Argo CD may manage:

```text
Namespace-scoped resources
```

such as:

```text
Deployment
Service
ConfigMap
Secret
```

and may also need:

```text
Cluster-scoped resources
```

such as:

```text
ClusterRole
ClusterRoleBinding
CustomResourceDefinition
IngressClass
```

depending on the platform.

---

# 23. Security Decision

Ask:

```text
Does this Argo CD installation need cluster-scoped resources?
```

If:

```text
No
```

restrict it.

If:

```text
Yes
```

grant only the required cluster-level permissions.

---

# 24. Cluster Registration

Before Argo CD can deploy to a target cluster:

```text
Cluster must be registered with Argo CD.
```

Common CLI:

```bash
argocd cluster list
```

This shows registered clusters.

---

# 25. Cluster Registration Workflow

```text
Create EKS
    |
    v
Validate API connectivity
    |
    v
Configure EKS authentication
    |
    v
Register cluster with Argo CD
    |
    v
Apply cluster labels
    |
    v
Validate Project destination
    |
    v
ApplicationSet discovers cluster
    |
    v
Applications generated
```

---

# 26. Registering a Cluster

The exact CLI arguments depend on the authentication method and Argo CD version.

A common interactive approach is:

```bash
argocd cluster add <context-name>
```

This uses a Kubernetes context and configures the required Argo CD cluster access.

Do not run this blindly against production.

Review the permissions it creates.

---

# 27. Verify Registration

```bash
argocd cluster list
```

Look for:

```text
SERVER
NAME
VERSION
STATUS
```

The target cluster should report healthy connectivity.

---

# 28. Kubernetes Context

Before registration:

```bash
kubectl config get-contexts
```

Select the correct context:

```bash
kubectl config use-context <context>
```

Then verify:

```bash
kubectl get nodes
```

Only after validating the context should you register the cluster.

---

# 29. Production Safety Before Registration

Verify:

```text
[ ] Correct AWS account
[ ] Correct EKS cluster
[ ] Correct region
[ ] Correct Kubernetes context
[ ] Correct IAM identity
[ ] Correct permissions
[ ] Correct network path
[ ] Correct Argo CD environment
```

A context mistake can register the wrong production cluster.

---

# 30. Cluster Metadata

Registered clusters can have metadata such as:

```text
name
server
labels
annotations
```

Labels are particularly useful for ApplicationSet.

Example:

```text
environment=prod
region=ap-south-1
account=production
team=platform
```

---

# 31. Cluster Labels as Routing

ApplicationSet can use:

```text
environment=prod
```

to determine target clusters.

Architecture:

```text
Cluster labels
      |
      v
ApplicationSet selector
      |
      v
Generated Applications
      |
      v
Target clusters
```

---

# 32. Production Label Taxonomy

A practical taxonomy might include:

```text
environment=dev|qa|prod
region=ap-south-1
account=dev|qa|prod
tier=application|platform
role=primary|dr
compliance=standard|restricted
```

Do not create unnecessary labels.

---

# 33. Environment Label

Example:

```text
environment=prod
```

Use it for:

```text
Production application selection
Production policies
Monitoring
Reporting
```

---

# 34. Region Label

Example:

```text
region=ap-south-1
```

Useful for:

```text
Regional deployment
DR
Latency
Capacity planning
```

---

# 35. Account Label

Example:

```text
account=prod
```

Useful when cluster names alone are insufficient.

For example:

```text
prod-cluster-1
```

may exist in multiple AWS accounts.

---

# 36. Primary/DR Label

Example:

```text
role=primary
```

and:

```text
role=dr
```

This allows controlled deployment policies.

---

# 37. Cluster Generator Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-prod
  namespace: argocd

spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: prod

  template:
    metadata:
      name: 'roboshop-{{name}}'

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: environments/prod

      destination:
        server: '{{server}}'
        namespace: roboshop
```

This produces one Application per matching cluster.

---

# 38. Multi-Cluster Application Naming

Good:

```text
roboshop-prod-eks-prod-01
roboshop-prod-eks-prod-02
```

Bad:

```text
roboshop
roboshop
```

Names must be unique inside the Argo CD namespace.

---

# 39. Cluster Selector

Simple:

```yaml
selector:
  matchLabels:
    environment: prod
```

More restrictive:

```yaml
selector:
  matchLabels:
    environment: prod
    region: ap-south-1
```

This is useful when only specific production clusters should receive the workload.

---

# 40. Cluster Generator With Multiple Environments

Separate ApplicationSets can provide clearer governance:

```text
roboshop-dev-clusters
roboshop-qa-clusters
roboshop-prod-clusters
```

Each uses a controlled selector.

---

# 41. Why Separate Production ApplicationSets?

Benefits:

```text
Clear ownership
Clear blast radius
Simpler troubleshooting
Independent policies
Different sync behavior
Different Projects
```

For enterprise environments, this can be easier to operate than one enormous generator.

---

# 42. Centralized Multi-Cluster RoboShop

```text
                         GitOps Repository
                                |
                                v
                         Central Argo CD
                                |
                    +-----------+-----------+
                    |                       |
              ApplicationSet          ApplicationSet
                 DEV/QA                    PROD
                    |                       |
             +------+-----+          +------+------+
             |            |          |             |
             v            v          v             v
          EKS-DEV       EKS-QA   EKS-PROD-1    EKS-PROD-2
```

---

# 43. Production vs DR

A common model:

```text
PROD PRIMARY
   |
   +--> EKS ap-south-1

PROD DR
   |
   +--> EKS ap-southeast-1
```

Both can be managed from the same Argo CD instance if connectivity and security requirements permit.

---

# 44. Active-Active vs Active-Passive

## Active-Active

Both clusters serve traffic.

GitOps deploys the same application to both.

Useful for:

```text
Global availability
Load distribution
Regional traffic
```

---

## Active-Passive

Primary serves traffic.

DR cluster is ready but not necessarily serving production traffic.

GitOps can still keep configuration synchronized.

---

# 45. Important DR Distinction

GitOps restores:

```text
Desired Kubernetes configuration
```

It does not automatically restore:

```text
Database data
S3 objects
EBS volumes
External state
```

Application DR must include stateful dependencies.

---

# 46. Multi-Cluster Stateful Applications

For RoboShop:

```text
Application workloads
```

can be redeployed using GitOps.

But databases such as:

```text
RDS
```

must have their own:

```text
Backup
Replication
Recovery
Failover
```

strategy.

Terraform and AWS-native services may manage those parts.

---

# 47. Argo CD and Terraform Boundary

A clean boundary:

```text
Terraform
   |
   +--> VPC
   +--> EKS
   +--> IAM
   +--> RDS
   +--> ECR
   +--> ALB infrastructure
```

Argo CD:

```text
Kubernetes application resources
Helm
Deployments
Services
Ingress
ConfigMaps
HPA
```

Avoid two systems fighting over the same resource.

---

# 48. Terraform + Argo CD Architecture

```text
Terraform
    |
    v
AWS Infrastructure
    |
    v
EKS
    |
    v
Argo CD
    |
    v
Kubernetes Workloads
```

This is a strong production separation.

---

# 49. ALB Ingress

For the user's RoboShop architecture:

```text
Internet
   |
   v
AWS ALB
   |
   v
Kubernetes Ingress
   |
   v
Frontend Service
   |
   v
Frontend Pods
```

Argo CD can manage:

```text
Ingress
Service
Deployment
```

while AWS Load Balancer Controller handles the ALB lifecycle.

---

# 50. Do Not Use API Gateway Here

The target architecture uses:

```text
AWS ALB Ingress
```

not:

```text
API Gateway
```

Argo CD manages the Kubernetes Ingress manifest.

AWS Load Balancer Controller reconciles the Ingress into an AWS ALB.

---

# 51. Cluster Registration Security

Cluster credentials stored by Argo CD are sensitive.

Protect:

```text
Argo CD secrets
Encryption at rest
Kubernetes RBAC
Argo CD RBAC
Repository access
Management cluster
```

Access to cluster credentials effectively provides control over target clusters.

---

# 52. Management Cluster Security

Harden:

```text
EKS API
Argo CD API
Argo CD UI
Ingress
Secrets
Service accounts
IAM roles
Network policies
Pod security
```

Restrict access to Argo CD itself.

---

# 53. Argo CD UI Exposure

Do not expose Argo CD publicly without protection.

Common controls:

```text
Private access
SSO
MFA through identity provider
Network restrictions
TLS
RBAC
```

The exact implementation depends on the organization's identity architecture.

---

# 54. Argo CD RBAC vs Kubernetes RBAC

There are two separate authorization layers.

## Argo CD RBAC

Controls:

```text
Who can sync an Application
Who can modify an Application
Who can view projects
Who can manage repositories
```

## Kubernetes RBAC

Controls:

```text
What Argo CD can do inside target clusters
```

Both must be configured correctly.

---

# 55. Production Security Boundary

```text
Human
 |
 v
SSO
 |
 v
Argo CD RBAC
 |
 v
Application
 |
 v
Kubernetes API
 |
 v
Kubernetes RBAC
```

This creates defense in depth.

---

# 56. Production Project Restrictions

Example conceptual Project:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd

spec:
  sourceRepos:
    - https://github.com/example/roboshop-gitops.git

  destinations:
    - namespace: roboshop
      server: https://prod-cluster.example.internal

  clusterResourceWhitelist: []

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: networking.k8s.io
      kind: Ingress
```

The exact production allow-list should match the resources actually required.

---

# 57. Why Projects Matter in Multi-Cluster

Without Projects:

```text
ApplicationSet
      |
      +--> potentially broad destinations
```

With Projects:

```text
ApplicationSet
      |
      v
Project
      |
      +--> allowed repo
      +--> allowed cluster
      +--> allowed namespace
      +--> allowed resources
```

This reduces blast radius.

---

# 58. Multi-Cluster Production ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-production
  namespace: argocd

spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: prod
            application: roboshop

  template:
    metadata:
      name: 'roboshop-{{name}}'
      labels:
        app.kubernetes.io/part-of: roboshop
        environment: prod
        managed-by: applicationset

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: environments/prod

      destination:
        server: '{{server}}'
        namespace: roboshop

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true
```

---

# 59. Production Caution About Automated Sync

For production clusters, automated sync should be enabled only after validating:

```text
Git review process
Project restrictions
Prune behavior
Rollback strategy
Monitoring
On-call readiness
```

Some organizations use manual production sync for stronger change control.

---

# 60. Cluster Onboarding as GitOps

A mature platform can make onboarding declarative.

Example:

```text
clusters/
└── prod/
    └── eks-prod-03.yaml
```

This file can define:

```text
cluster identity
environment
region
application eligibility
```

A controlled process then registers and labels the cluster.

The exact cluster-registration mechanism should be chosen carefully because credentials and external cluster state are security-sensitive.

---

# 61. Cluster Lifecycle

A complete lifecycle is:

```text
Provision
   |
Register
   |
Label
   |
Deploy
   |
Monitor
   |
Upgrade
   |
Drain
   |
Remove
```

Every stage should be documented.

---

# 62. New Cluster Onboarding Runbook

```text
1. Provision EKS with Terraform.
2. Configure networking.
3. Configure EKS authentication.
4. Install required platform controllers.
5. Validate Kubernetes API.
6. Register cluster with Argo CD.
7. Apply approved labels.
8. Validate Argo CD cluster status.
9. Verify Project destination.
10. Verify ApplicationSet selection.
11. Confirm generated Applications.
12. Confirm workloads.
13. Confirm ALB/Ingress.
14. Confirm monitoring.
15. Confirm alerts.
```

---

# 63. Cluster Removal Runbook

Before removing a cluster:

```text
1. Stop new Application generation.
2. Remove/disable cluster from selectors.
3. Confirm Applications are no longer targeting it.
4. Decide whether workloads should be deleted.
5. Remove cluster registration.
6. Decommission EKS.
```

Do not destroy infrastructure first and troubleshoot Argo CD afterward.

---

# 64. Cluster Removal Risk

If a cluster is deleted while Argo CD still believes it exists:

```text
Applications may become Unknown
```

because:

```text
Kubernetes API unreachable
```

Clean removal avoids stale state and confusing alerts.

---

# 65. Cluster Connectivity Troubleshooting

Symptoms:

```text
Application Unknown
Sync failed
Cluster connection error
```

Check:

```bash
argocd cluster list
```

Then test from a suitable network/credential context:

```bash
kubectl cluster-info
kubectl get nodes
```

Check:

```text
DNS
Network
Security groups
EKS endpoint
IAM
Kubernetes RBAC
```

---

# 66. EKS Authentication Troubleshooting

Check AWS identity:

```bash
aws sts get-caller-identity
```

Check cluster:

```bash
aws eks describe-cluster \
  --name <cluster> \
  --region <region>
```

Then:

```bash
aws eks update-kubeconfig \
  --name <cluster> \
  --region <region>
```

Validate:

```bash
kubectl get nodes
```

---

# 67. Wrong AWS Account Troubleshooting

Run:

```bash
aws sts get-caller-identity
```

Verify:

```text
Account ID
ARN
Role
```

A common production incident is using:

```text
DEV AWS credentials
```

while targeting:

```text
PROD EKS
```

Always verify identity.

---

# 68. Wrong Region Troubleshooting

Check:

```bash
aws configure get region
```

Then explicitly use:

```bash
aws eks describe-cluster \
  --name <cluster> \
  --region <expected-region>
```

Do not rely on a default region during production operations.

---

# 69. Kubernetes RBAC Failure

Symptoms:

```text
permission denied
forbidden
cannot get resource
cannot create resource
```

Determine:

```text
Which identity is Argo CD using?
Which resource is being requested?
Which namespace?
Which API group?
```

Then review:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
EKS authentication
```

---

# 70. Application Permission Failure

Example:

```text
Argo CD can reach cluster
but cannot create Deployment
```

This means:

```text
Authentication works
Authorization fails
```

Do not troubleshoot DNS first.

---

# 71. API Server Reachability Failure

Symptoms:

```text
timeout
connection refused
i/o timeout
TLS errors
```

Check:

```text
EKS endpoint
Network route
Security group
DNS
Proxy
TLS
```

---

# 72. Private EKS Troubleshooting

If EKS API is private:

```text
Argo CD management cluster
        |
        v
Private networking
        |
        v
EKS API
```

Check:

```text
VPC routing
Transit Gateway
Security groups
DNS
Resolver
```

---

# 73. Public EKS API Troubleshooting

If public endpoint is used:

```text
Restrict public access
```

using appropriate EKS endpoint access controls.

Avoid:

```text
0.0.0.0/0
```

unless there is a compelling and approved architecture.

---

# 74. Multi-Account Network Pattern

```text
                  Management VPC
                       |
                 Central Argo CD
                       |
                Transit Gateway
            /          |          \
           /           |           \
      DEV VPC        QA VPC       PROD VPC
         |             |             |
      EKS DEV        EKS QA       EKS PROD
```

This is one possible enterprise pattern.

---

# 75. Cross-Account Authentication

A multi-account design may involve:

```text
Management AWS account
        |
        v
IAM role assumption
        |
        v
Target AWS account
        |
        v
EKS access
```

Exact implementation depends on the EKS authentication configuration and security model.

---

# 76. Cross-Account Least Privilege

Do not allow a management identity to:

```text
AdministratorAccess
```

across every account simply because Argo CD needs EKS access.

Separate:

```text
Infrastructure administration
Kubernetes deployment permissions
Application access
```

---

# 77. Centralized vs Decentralized Argo CD

## Centralized

```text
One Argo CD
 |
 +--> many clusters
```

Advantages:

```text
Central visibility
Central governance
Lower duplicated control-plane cost
```

Disadvantages:

```text
Larger blast radius
Cross-cluster connectivity
Central dependency
```

---

# 78. Decentralized

```text
EKS DEV -> Argo CD DEV
EKS QA  -> Argo CD QA
EKS PROD -> Argo CD PROD
```

Advantages:

```text
Isolation
Smaller blast radius
Independent upgrades
```

Disadvantages:

```text
More Argo CD instances
More operational overhead
Duplicated configuration
```

---

# 79. When Centralized Is Appropriate

Centralized often fits:

```text
Single organization
Strong central platform team
Reliable network connectivity
Central security model
Many similar clusters
```

---

# 80. When Decentralized Is Appropriate

Decentralized may fit:

```text
Strong account isolation
Regulatory boundaries
Independent business units
Highly isolated networks
Different upgrade schedules
```

---

# 81. Hybrid Model

Some enterprises use:

```text
Argo CD DEV
Argo CD QA
Central/isolated PROD Argo CD
```

or:

```text
Argo CD per region
```

There is no universal architecture.

Choose based on:

```text
Security
Network
Scale
Ownership
Compliance
Availability
```

---

# 82. Failure Scenario: Central Argo CD Down

If Argo CD management cluster fails:

```text
EKS clusters continue running current workloads.
```

But:

```text
No GitOps reconciliation
No automatic drift correction
No new deployments
```

until Argo CD is restored.

This is why Kubernetes applications should not depend on Argo CD being continuously available to keep running.

---

# 83. Failure Scenario: One Target Cluster Down

Suppose:

```text
EKS-QA
```

fails.

Argo CD can still manage:

```text
EKS-DEV
EKS-PROD
```

provided the Argo CD control plane remains healthy.

This is a key benefit of centralized multi-cluster management.

---

# 84. Failure Scenario: Git Down

Existing workloads can continue running.

But:

```text
New desired state
```

cannot be fetched.

Argo CD should recover automatically when Git connectivity returns, depending on the failure.

---

# 85. Failure Scenario: Network Partition

If:

```text
Argo CD <----X----> EKS PROD
```

then:

```text
Production workloads continue running
```

but:

```text
Argo CD cannot reconcile
```

After connectivity returns, Argo CD can detect drift and reconcile.

---

# 86. Drift During Network Partition

Suppose someone manually changes:

```yaml
replicas: 5
```

while Git says:

```yaml
replicas: 3
```

Argo CD cannot detect this while disconnected.

After reconnection:

```text
Argo CD sees actual=5
Git desired=3
```

and if self-heal is enabled:

```text
5 -> 3
```

---

# 87. Production Drift Prevention

Use:

```text
RBAC
Self-heal
Admission policies
Restricted kubectl access
Audit logs
```

The strongest model is:

```text
Git is the approved change path.
```

---

# 88. Break-Glass Access

Production operations may require emergency access.

A break-glass model should have:

```text
Limited users
Strong authentication
Time-bound access
Audit logging
Post-incident review
```

Manual Kubernetes changes should be reconciled back to Git afterward.

---

# 89. Argo CD and Cluster Upgrades

When upgrading EKS:

```text
Argo CD does not replace EKS upgrade tooling.
```

Infrastructure lifecycle:

```text
Terraform / AWS
```

Application lifecycle:

```text
Argo CD
```

Keep responsibilities clear.

---

# 90. Kubernetes Version Compatibility

Before upgrading:

```text
EKS
Argo CD
Helm
CRDs
Ingress controller
```

verify compatibility.

Argo CD should be upgraded according to its supported version matrix and organizational testing.

---

# 91. Argo CD HA

For a centralized control plane, evaluate:

```text
Multiple API Server replicas
Application Controller scaling
Repo Server scaling
Redis HA requirements
ApplicationSet Controller
Pod anti-affinity
Pod disruption budgets
```

Use the Argo CD installation method and version's documented HA model.

---

# 92. Management Cluster Sizing

Sizing depends on:

```text
Number of Applications
Number of resources
Number of clusters
Repository size
Manifest generation load
Sync frequency
ApplicationSet complexity
```

Do not size solely by number of Kubernetes clusters.

A single cluster with thousands of Applications can create significant load.

---

# 93. Controller Scaling

Monitor:

```text
CPU
Memory
Reconciliation latency
API requests
Git operations
Manifest generation
Queue depth
```

Scale based on observed workload.

---

# 94. Repo Server Scaling

Repo Server is important for:

```text
Git fetch
Helm rendering
Kustomize rendering
Plugin operations
Manifest generation
```

If Repo Server is overloaded:

```text
Applications may become unable to generate manifests.
```

This can affect multiple target clusters at once.

---

# 95. Application Controller Scaling

Application Controller handles application reconciliation.

Large centralized deployments can require:

```text
Resource tuning
Controller sharding/scaling
Careful reconciliation configuration
```

Use the Argo CD version's supported scaling/sharding model.

---

# 96. ApplicationSet Controller Scaling

ApplicationSet controller handles:

```text
Generator evaluation
Application generation
Application updates
```

Large generator outputs can increase controller load.

Monitor it independently.

---

# 97. Redis Role

Redis is used by Argo CD for internal caching/state-related functions depending on the architecture/version.

A Redis failure may affect Argo CD operation even though target Kubernetes clusters themselves continue running.

Production deployments should use the supported HA architecture when required.

---

# 98. Management Cluster Backup

Back up:

```text
Argo CD configuration
Projects
RBAC
Repository credentials
Cluster registration
ApplicationSets
Applications
Custom configuration
```

Secrets require secure backup handling.

---

# 99. Git as Primary Recovery Source

The most important recovery asset should be:

```text
Git repository
```

because it contains:

```text
Desired application configuration
ApplicationSets
Projects
Helm charts
Kustomize overlays
Environment configuration
```

The Git repository itself should have:

```text
Backups
Protected branches
Access controls
MFA/SSO
Audit logs
```

---

# 100. DR Rebuild

A practical rebuild:

```text
AWS infrastructure
      |
      v
Management EKS
      |
      v
Argo CD
      |
      v
Repository configuration
      |
      v
Projects/RBAC
      |
      v
Cluster registration
      |
      v
ApplicationSets
      |
      v
Generated Applications
      |
      v
Target clusters
```

---

# 101. DR Does Not Mean Zero Downtime

Central Argo CD loss may not immediately stop workloads.

But if:

```text
Argo CD recovery takes 4 hours
```

then:

```text
GitOps deployments and drift correction
```

may be unavailable for four hours.

Define:

```text
RTO
RPO
```

for the control plane.

---

# 102. ApplicationSet Multi-Cluster YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-multi-cluster
  namespace: argocd

spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            application: roboshop
            environment: prod

  template:
    metadata:
      name: 'roboshop-{{name}}'

      labels:
        application: roboshop
        environment: prod

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: environments/prod

        helm:
          valueFiles:
            - values/prod.yaml

      destination:
        server: '{{server}}'
        namespace: roboshop

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true

      revisionHistoryLimit: 10
```

---

# 103. Multi-Cluster ApplicationSet With Region

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-ap-south
  namespace: argocd

spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            application: roboshop
            environment: prod
            region: ap-south-1

  template:
    metadata:
      name: 'roboshop-{{name}}'

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: environments/prod

      destination:
        server: '{{server}}'
        namespace: roboshop
```

---

# 104. Multi-Cluster ApplicationSet With DR

Conceptual labels:

```text
environment=prod
role=dr
application=roboshop
```

ApplicationSet selector:

```yaml
selector:
  matchLabels:
    environment: prod
    role: dr
    application: roboshop
```

This gives an explicit DR target group.

---

# 105. DR Deployment Policy

A DR cluster might use:

```text
same application version
different replica count
different ingress behavior
different autoscaling
different external dependencies
```

Therefore DR should not always blindly reuse production values.

Use environment/cluster-specific configuration.

---

# 106. Multi-Cluster Helm Values

Example:

```text
values/
├── prod.yaml
├── prod-primary.yaml
└── prod-dr.yaml
```

The ApplicationSet can choose the correct values according to generator parameters.

---

# 107. Cluster-Specific Configuration Principle

Keep common values in:

```text
prod.yaml
```

and overrides in:

```text
prod-dr.yaml
```

Avoid copying hundreds of lines between files.

---

# 108. ApplicationSet and Resource Requests

Argo CD can deploy:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi

  limits:
    cpu: 500m
    memory: 512Mi
```

The ApplicationSet does not itself enforce resource requests.

It generates the Application that points to manifests containing them.

---

# 109. ApplicationSet and HPA

Similarly:

```text
ApplicationSet
   |
   v
Application
   |
   v
Helm/Kustomize
   |
   v
Deployment + HPA
```

HPA remains a Kubernetes controller responsibility.

---

# 110. ApplicationSet and ALB

```text
ApplicationSet
    |
    v
Application
    |
    v
Ingress
    |
    v
AWS Load Balancer Controller
    |
    v
AWS ALB
```

This separation is important.

---

# 111. ApplicationSet and Platform Controllers

A platform ApplicationSet may deploy:

```text
AWS Load Balancer Controller
External Secrets
Metrics Server
Prometheus
Grafana
Security controllers
```

Business ApplicationSets can then deploy:

```text
RoboShop services
```

Separate these concerns.

---

# 112. Platform vs Application Clusters

One option:

```text
Platform ApplicationSet
    |
    +--> ingress controller
    +--> observability
    +--> security

Application ApplicationSet
    |
    +--> RoboShop
```

This gives clear ownership.

---

# 113. Cluster Bootstrap Ordering

A target cluster may need:

```text
Namespace
CRDs
Controllers
RBAC
Ingress controller
Secret integration
Monitoring
Applications
```

before workloads are deployed.

Use:

```text
App of Apps
Sync waves
Separate ApplicationSets
```

where appropriate.

---

# 114. Multi-Cluster Sync Ordering

Do not assume Applications generated by separate ApplicationSets automatically have the desired dependency order.

If one application depends on another:

```text
Platform
  |
  v
Application
```

explicitly model the dependency.

---

# 115. Production Cluster Bootstrap

```text
EKS
 |
 v
Argo CD registration
 |
 v
Platform controllers
 |
 v
Namespaces
 |
 v
Secrets integration
 |
 v
ALB controller
 |
 v
Monitoring
 |
 v
RoboShop
```

---

# 116. Multi-Cluster Observability

Central Prometheus/Grafana can be:

```text
centralized
```

or each cluster can have:

```text
local monitoring
```

A centralized Argo CD does not require centralized monitoring.

---

# 117. ELK Across Clusters

Each cluster can ship logs:

```text
EKS DEV
   |
EKS QA
   |
EKS PROD
   |
   v
ELK
```

Include cluster metadata:

```text
cluster
environment
namespace
pod
container
```

This allows cross-cluster investigation.

---

# 118. Grafana Cluster Dashboards

Useful variables:

```text
cluster
environment
namespace
service
pod
```

Then one dashboard can switch between:

```text
EKS-DEV
EKS-QA
EKS-PROD
```

---

# 119. Argo CD Multi-Cluster Monitoring

Track:

```text
Cluster connectivity
Application health by cluster
OutOfSync count
Sync failure count
Reconciliation latency
ApplicationSet generation failures
```

A cluster outage should not be confused with an application failure.

---

# 120. Cluster-Level Incident Example

Symptoms:

```text
All Applications in EKS-PROD are Unknown
```

If many unrelated Applications fail simultaneously:

```text
Suspect cluster/API/network connectivity
```

rather than individual manifests.

---

# 121. Application-Level Incident Example

Symptoms:

```text
Only cart-prod is Degraded
```

while:

```text
user-prod
payment-prod
inventory-prod
```

are healthy.

Suspect:

```text
Application manifest
Deployment
Pod
Image
Config
Secret
Probe
```

rather than cluster connectivity.

---

# 122. Repo-Level Incident Example

Symptoms:

```text
Many Applications cannot generate manifests
```

across different clusters.

Suspect:

```text
Repo Server
Git authentication
Git repository
Helm rendering
Kustomize rendering
```

---

# 123. ApplicationSet-Level Incident Example

Symptoms:

```text
Existing Applications healthy
New clusters not receiving Applications
```

Suspect:

```text
ApplicationSet controller
Cluster generator
Labels
Selectors
```

---

# 124. Central Argo CD Incident Isolation

Use the scope of failure:

```text
One resource
    -> workload problem

One Application
    -> application problem

Many Applications in one cluster
    -> cluster/API problem

Many Applications across clusters
    -> Argo CD/repository problem

Only newly generated Applications
    -> ApplicationSet problem
```

This is a powerful production troubleshooting method.

---

# 125. Command Set: Argo CD

```bash
argocd cluster list
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app sync <app>
argocd app history <app>
argocd app resources <app>
```

Use the commands appropriate to the incident.

---

# 126. Command Set: Kubernetes

```bash
kubectl get applications -n argocd
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd
kubectl describe application <name> -n argocd
kubectl get events -n argocd
```

---

# 127. Command Set: Target Cluster

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A
kubectl get ingress -A
kubectl get svc -A
```

Then narrow the investigation.

---

# 128. ApplicationSet Controller Logs

Find the controller:

```bash
kubectl get pods -n argocd
```

Then inspect its logs using the pod selected from the current installation:

```bash
kubectl logs <applicationset-controller-pod> -n argocd
```

Look for:

```text
generator
reconcile
permission
Git
cluster
selector
error
```

---

# 129. Application Controller Logs

```bash
kubectl logs <application-controller-pod> -n argocd
```

Use this when:

```text
Applications exist
but synchronization/reconciliation is failing.
```

---

# 130. Repo Server Logs

```bash
kubectl logs <repo-server-pod> -n argocd
```

Use this for:

```text
Helm
Kustomize
Git
manifest generation
repository access
```

problems.

---

# 131. Management Cluster Troubleshooting

First:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl get events -n argocd
```

Check:

```text
API Server
Application Controller
ApplicationSet Controller
Repo Server
Redis
```

before blaming target clusters.

---

# 132. Production Runbook: Target Cluster Unknown

```text
1. argocd cluster list
2. Check target cluster status
3. aws sts get-caller-identity
4. Verify EKS API endpoint
5. Verify network connectivity
6. Verify DNS
7. Verify EKS authentication
8. Verify Kubernetes RBAC
9. Check Argo CD controller logs
10. Re-test Application
```

---

# 133. Production Runbook: Wrong Application Placement

```text
1. Inspect generated Application.
2. Check destination.server.
3. Check ApplicationSet selector.
4. Check cluster labels.
5. Check ApplicationSet generator output.
6. Check Project destination restrictions.
7. Fix source configuration.
8. Review generated Application.
9. Confirm workload destination.
```

---

# 134. Production Runbook: Cluster Added but No Applications

```text
1. argocd cluster list
2. Verify cluster is registered.
3. Verify labels.
4. Verify ApplicationSet selector.
5. kubectl describe applicationset
6. Check controller logs.
7. Confirm Project allows destination.
8. Check generated Applications.
```

---

# 135. Production Runbook: Application Exists but Cannot Sync

```text
1. argocd app get <app>
2. argocd app diff <app>
3. Check source revision/path.
4. Check repo access.
5. Check target cluster connectivity.
6. Check Kubernetes RBAC.
7. Check resource events.
8. Check controller logs.
```

---

# 136. Production Runbook: Remove a Cluster Safely

```text
1. Identify all Applications targeting cluster.
2. Stop ApplicationSet generation.
3. Confirm generated Applications.
4. Decide workload cleanup.
5. Remove cluster from ApplicationSet selection.
6. Remove cluster registration.
7. Decommission EKS.
8. Verify no stale Applications remain.
```

---

# 137. Production Security Checklist

```text
[ ] Argo CD UI protected
[ ] SSO enabled where appropriate
[ ] MFA enforced through identity provider
[ ] Argo CD RBAC configured
[ ] Projects restrict destinations
[ ] Projects restrict repositories
[ ] Cluster credentials protected
[ ] EKS API access restricted
[ ] Private networking evaluated
[ ] IAM least privilege
[ ] Kubernetes RBAC least privilege
[ ] Break-glass process documented
[ ] Audit logging enabled
[ ] Secrets encrypted
```

---

# 138. Production Multi-Cluster Checklist

```text
[ ] Management cluster HA
[ ] Target clusters registered
[ ] Cluster labels standardized
[ ] ApplicationSet selectors reviewed
[ ] Projects defined
[ ] Cross-account access reviewed
[ ] Cross-region connectivity reviewed
[ ] DNS tested
[ ] EKS API connectivity tested
[ ] DR strategy tested
[ ] Backup tested
[ ] Monitoring configured
[ ] Alerting configured
[ ] Cluster onboarding runbook documented
[ ] Cluster removal runbook documented
```

---

# 139. Interview Question: Can Argo CD Manage Multiple Clusters?

### Answer

> Yes. A single Argo CD installation can manage multiple Kubernetes clusters. The target clusters are registered with Argo CD, and Applications specify their destinations. ApplicationSet can use the Cluster generator and cluster labels to automatically generate Applications for multiple EKS clusters.

---

# 140. Interview Question: Does Argo CD Have to Run in the Target Cluster?

### Answer

> No. Argo CD can run in a dedicated management cluster and manage remote Kubernetes clusters. This is a common centralized GitOps architecture.

---

# 141. Interview Question: What Happens If Argo CD Goes Down?

### Answer

> Existing Kubernetes workloads generally continue running because Kubernetes does not depend on Argo CD for normal pod execution. However, GitOps reconciliation, new deployments, drift correction and ApplicationSet generation are affected until Argo CD is restored.

---

# 142. Interview Question: What Happens If One Target Cluster Goes Down?

### Answer

> Applications targeting that cluster become unable to reconcile and may show an unknown or connection-related state, but Argo CD can continue managing other healthy target clusters.

---

# 143. Interview Question: How Do You Select Production Clusters?

### Answer

> I use controlled cluster labels such as `environment=prod`, optionally combined with region, application and role labels. An ApplicationSet Cluster generator selects matching clusters. Argo CD Projects further restrict which destinations are allowed.

---

# 144. Interview Question: How Would You Manage 20 EKS Clusters?

### Answer

> I would consider centralized Argo CD with ApplicationSets, standardized cluster labels, Argo CD Projects, environment-specific GitOps configuration and least-privilege cluster access. For larger or highly isolated organizations, I would evaluate multiple Argo CD instances based on security, compliance, network and blast-radius requirements.

---

# 145. Interview Question: Centralized or One Argo CD Per Cluster?

### Answer

> There is no universal answer. Centralized Argo CD gives strong visibility and governance with less duplicated control-plane overhead, while per-cluster Argo CD provides stronger isolation and a smaller blast radius. I would choose based on organization structure, security boundaries, connectivity, scale and compliance requirements.

---

# 146. Interview Scenario: Production Cluster Is Unreachable

### Answer

I would first determine whether it is a cluster-wide connectivity problem:

```bash
argocd cluster list
```

Then validate:

```text
EKS API
DNS
network route
security groups
AWS IAM
EKS authentication
Kubernetes RBAC
```

I would also check whether unrelated Applications in the same cluster are failing. If many Applications are simultaneously Unknown, that strongly suggests cluster/API connectivity rather than an individual workload problem.

---

# 147. Interview Scenario: New Cluster Was Added Automatically

### Answer

> I would inspect the ApplicationSet Cluster generator selector and the labels on the newly registered cluster. If the cluster unexpectedly matched a production selector, I would immediately correct the label or selector, review generated Applications and audit whether any production resources were deployed.

---

# 148. Interview Scenario: Central Argo CD Is a Single Point of Failure

### Answer

> The workloads themselves do not normally stop merely because Argo CD is unavailable, but GitOps control-plane functionality is interrupted. I would mitigate this with an HA management cluster, monitored Argo CD components, backups, tested recovery procedures and appropriate RTO/RPO targets. For highly isolated environments, I might use multiple Argo CD instances.

---

# 149. Interview Scenario: Multiple AWS Accounts

### Answer

> I would separate infrastructure ownership from application deployment. Terraform provisions VPCs, EKS and IAM. Argo CD manages Kubernetes workloads. For cross-account deployment, I would use tightly controlled AWS IAM and EKS authentication, restricted network connectivity and Kubernetes RBAC, rather than granting broad administrator permissions.

---

# 150. Interview Scenario: Multi-Region Production

### Answer

A production design could be:

```text
Central Argo CD
      |
      +--> EKS ap-south-1
      |
      +--> EKS ap-southeast-1
```

Cluster labels identify:

```text
environment=prod
region=<region>
role=primary|dr
```

ApplicationSet then controls which workloads deploy to which clusters.

---

# 151. Interview Scenario: DR Cluster

### Answer

> I would keep the application configuration in Git, register the DR cluster, apply controlled labels, and use ApplicationSet or another explicit deployment mechanism to deploy the approved application version. I would separately design data replication and recovery for RDS and other stateful systems because GitOps does not restore application data.

---

# 152. Interview Scenario: Argo CD Can Reach Cluster but Deployment Fails

### Answer

> That indicates authentication probably works, so I would investigate authorization. I would inspect the Kubernetes RBAC permissions of the Argo CD identity and verify whether it can create the specific resource in the target namespace. I would also check Project restrictions and controller logs.

---

# 153. Interview Scenario: All Clusters Fail

### Answer

If:

```text
DEV
QA
PROD
```

all fail at approximately the same time, I would investigate the central control plane:

```text
Argo CD Application Controller
Repo Server
Redis
network
Git
management cluster
```

rather than troubleshooting each target cluster independently.

---

# 154. Interview Scenario: Only New Applications Fail

If existing Applications remain healthy but newly generated Applications fail:

```text
ApplicationSet
generator
template
Git discovery
cluster selector
```

become high-priority suspects.

---

# 155. Interview Scenario: Git Is Healthy but Applications Cannot Generate

Check:

```text
Repo Server
Helm
Kustomize
repository credentials
revision
path
manifest rendering
```

The target cluster may be completely healthy.

---

# 156. Production Architecture Summary

```text
                           Git
                            |
                            v
                    Central Argo CD
                            |
             +--------------+--------------+
             |                             |
      ApplicationSets                 Projects/RBAC
             |
             v
      Generated Applications
             |
      +------+------+------+
      |             |      |
      v             v      v
   EKS DEV        EKS QA  EKS PROD
      |             |      |
    Apps           Apps   Apps
```

---

# 157. RoboShop Production Architecture

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Build
   +--> Test
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker Image
   |
   v
ECR
   |
   v
GitOps Repository
   |
   v
ApplicationSet
   |
   v
Central Argo CD
   |
   +----------+----------+
   |          |          |
   v          v          v
 EKS DEV    EKS QA    EKS PROD
   |          |          |
RoboShop    RoboShop   RoboShop
```

---

# 158. ALB Integration

For production RoboShop:

```text
Internet
   |
   v
AWS ALB
   |
   v
Ingress
   |
   v
Frontend Service
   |
   v
Frontend Pod
```

The ALB is reconciled by:

```text
AWS Load Balancer Controller
```

The desired Kubernetes Ingress is managed by:

```text
Argo CD
```

---

# 159. Final Production Principles

1. Treat Argo CD as a control plane.
2. Treat target EKS clusters as workload planes.
3. Separate authentication from authorization.
4. Use least privilege.
5. Use Projects to constrain destinations.
6. Use ApplicationSets for scale.
7. Use cluster labels carefully.
8. Protect cluster registration.
9. Protect the Argo CD management cluster.
10. Design for management-plane failure.
11. Monitor every cluster independently.
12. Separate Terraform infrastructure from Argo CD application ownership.
13. Keep desired state in Git.
14. Use immutable image tags.
15. Test DR instead of only documenting it.
16. Avoid unnecessarily centralized blast radius.
17. Use private networking where appropriate.
18. Do not expose EKS APIs broadly for convenience.
19. Keep platform and application ownership clear.
20. Document cluster onboarding and removal.

---

# 160. Final Mental Model

The most important architecture to remember is:

```text
                    GitOps Repository
                           |
                           v
                    Central Argo CD
                           |
                  ApplicationSet
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          App-DEV       App-QA       App-PROD
             |             |             |
             v             v             v
          EKS-DEV        EKS-QA        EKS-PROD
             |             |             |
          Kubernetes    Kubernetes    Kubernetes
          workloads     workloads     workloads
```

And the control responsibility is:

```text
Terraform
   |
   +--> AWS infrastructure
   +--> EKS
   +--> IAM
   +--> networking
   +--> ECR

CI
   |
   +--> build
   +--> test
   +--> security
   +--> image
   +--> ECR
   +--> GitOps update

ApplicationSet
   |
   +--> generate Applications

Argo CD
   |
   +--> reconcile Applications

Kubernetes
   |
   +--> run workloads
```

This separation is the foundation of a production-grade multi-cluster GitOps platform.

---

# 161. Next File

```text
11-ArgoCD-AWS-EKS-and-Multi-Account.md
```

The next file will specialize further in AWS/EKS, including:

- EKS architecture for Argo CD
- AWS account boundaries
- EKS access modes
- IAM roles
- IRSA
- EKS Pod Identity concepts
- Kubernetes RBAC
- Private EKS API
- VPC networking
- Transit Gateway
- Cross-account access
- Multi-region architecture
- ECR integration
- AWS Load Balancer Controller
- ALB Ingress
- Terraform + Argo CD boundaries
- Production AWS diagrams
- Multi-account RoboShop architecture
- Security
- Disaster recovery
- Operational troubleshooting
- Production YAMLs
- Interview scenarios
