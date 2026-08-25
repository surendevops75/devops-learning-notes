# GitOps-Principles-and-Architecture

## 1. Purpose

This file moves from the fundamentals of GitOps into the architectural principles that make GitOps reliable at production scale.

The previous file established:

```text
Git = desired state
Kubernetes = runtime state
Argo CD = reconciliation/control mechanism
```

This file explains how those ideas become a real architecture.

The focus is:

- GitOps operating principles
- Control-loop architecture
- Desired-state management
- Pull-based delivery
- Reconciliation
- Source-of-truth design
- Repository architecture
- Environment architecture
- Cluster architecture
- Trust boundaries
- Failure domains
- Security boundaries
- Scaling considerations
- Single-cluster GitOps
- Centralized multi-cluster GitOps
- Enterprise GitOps architecture
- Production design decisions
- Interview scenarios

The practical reference architecture remains:

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
Amazon ECR
    |
    v
GitOps Repository
    |
    v
Argo CD
    |
    v
Kubernetes API
    |
    v
Amazon EKS
    |
    v
RoboShop
```

---

# 2. GitOps Architecture in One Picture

A production GitOps system has several logical layers:

```text
+-----------------------------------------------------------+
|                    Source Control                         |
|                                                           |
| Application Git              GitOps Repository            |
+-------------------+-----------------------+---------------+
                    |                       |
                    v                       v
             CI / Security              Argo CD
                    |                       |
                    v                       v
                  ECR                Kubernetes API
                                            |
                                            v
                                      Amazon EKS
                                            |
                       +--------------------+----------------+
                       |                    |                |
                       v                    v                v
                   Workloads            Services           ALB
```

Each layer has a different responsibility.

---

# 3. Layer 1 - Application Source

The application repository contains source code.

For RoboShop:

```text
roboshop-cart/
├── src/
├── tests/
├── Dockerfile
├── pom.xml
└── README.md
```

It answers:

> What software are we building?

It does not necessarily answer:

> What exact version should production run?

That deployment intent belongs in the GitOps configuration.

---

# 4. Layer 2 - Continuous Integration

CI validates and packages the application.

Example:

```text
Application Git
      |
      v
Jenkins / GitHub Actions
      |
      +--> Checkout
      +--> Build
      +--> Unit tests
      +--> SonarQube
      +--> Veracode
      +--> Docker build
      +--> Trivy
      |
      v
Amazon ECR
```

The result is a deployable artifact.

---

# 5. Layer 3 - GitOps Repository

The GitOps repository defines how the artifact should be deployed.

Example:

```text
roboshop-gitops/
├── applications/
├── applicationsets/
├── environments/
├── projects/
├── helm/
├── clusters/
└── platform/
```

It answers:

> Which version should run, where should it run, and with what configuration?

---

# 6. Layer 4 - GitOps Controller

Argo CD consumes the desired state and compares it with the cluster.

Conceptually:

```text
Git
 |
 | desired state
 v
Argo CD
 |
 +--> Render
 +--> Compare
 +--> Sync
 +--> Observe
 +--> Reconcile
 |
 v
Kubernetes
```

Argo CD is therefore the bridge between declarative configuration and the runtime environment.

---

# 7. Layer 5 - Kubernetes

Kubernetes is the runtime control plane.

Argo CD communicates through the Kubernetes API.

```text
Argo CD
   |
   v
Kubernetes API Server
   |
   +--> Deployment Controller
   +--> ReplicaSet Controller
   +--> Service handling
   +--> HPA Controller
   +--> Other controllers
```

Argo CD does not replace these controllers.

It declares and reconciles the Kubernetes resources.

Kubernetes then performs its own runtime reconciliation.

---

# 8. Layer 6 - AWS

In an EKS environment, Kubernetes can interact with AWS.

For example:

```text
Kubernetes Ingress
       |
       v
AWS Load Balancer Controller
       |
       v
AWS ALB
```

This produces layered reconciliation:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes resource
 |
 v
AWS Load Balancer Controller
 |
 v
AWS ALB
```

This layered architecture is important for troubleshooting.

---

# 9. Principle 1 - Declarative Configuration

A GitOps system should represent desired state declaratively.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 3
```

The configuration does not describe every action required to reach three replicas.

It describes the result.

The controllers determine how to achieve it.

---

# 10. Why Declarative Configuration Matters

Declarative configuration provides:

- Reproducibility
- Version control
- Diffability
- Reviewability
- Automation
- Rollback
- Consistency

Suppose production should run:

```text
cart
replicas = 3
image = 2026.08.19-abc123
```

That state can be recreated from Git.

Without declarative state, production knowledge may exist in:

```text
Engineer memory
Jenkins history
Shell history
Manual commands
Documentation
```

That is much harder to reproduce.

---

# 11. Principle 2 - Versioned Desired State

Desired state must be versioned.

Example:

```text
Commit A
cart:1.7.0

Commit B
cart:1.8.0

Commit C
cart:1.8.1
```

Git provides:

```text
Diff
History
Author
Timestamp
Branch
Pull Request
Approval
Revert
```

This gives the desired state a lifecycle.

---

# 12. Principle 3 - Git as the Authoritative State

For GitOps-managed resources:

```text
Git
 |
 v
Desired State
```

A manual cluster modification does not automatically become authoritative.

Example:

```text
Git:
replicas = 3

Manual:
replicas = 1
```

The correct response is generally:

```text
Detect drift
    |
    v
Determine whether manual change was intentional
    |
    +--> Temporary emergency change
    |       |
    |       v
    |    Restore Git state
    |
    +--> Permanent change
            |
            v
       Update Git
```

This preserves the source-of-truth model.

---

# 13. Principle 4 - Automated Synchronization

Once desired state changes in Git, the GitOps controller should be able to synchronize the target environment.

Conceptually:

```text
Git commit
    |
    v
Controller refresh
    |
    v
New desired state
    |
    v
Compare
    |
    v
Sync
```

Automation may be configured differently by environment.

For example:

```text
DEV  -> automatic
QA   -> automatic or controlled
PROD -> approval-controlled
```

There is no universal requirement that every environment use the same sync policy.

---

# 14. Principle 5 - Continuous Reconciliation

GitOps should not stop after deployment.

The controller continually observes the environment.

```text
Observe
   |
Compare
   |
Reconcile
   |
Observe
   |
Compare
   |
Reconcile
   |
Repeat
```

This creates convergence.

---

# 15. Desired State Convergence

Suppose:

```text
Desired:
replicas = 3

Actual:
replicas = 1
```

The reconciliation objective is:

```text
Actual -> Desired
```

After successful reconciliation:

```text
Desired = 3
Actual = 3
```

The environment has converged.

This concept of convergence is central to control-loop-based systems.

---

# 16. Control Loop Architecture

A simplified GitOps control loop is:

```text
        +----------------------+
        |                      |
        |     Git Desired      |
        |       State          |
        |                      |
        +----------+-----------+
                   |
                   v
             +-----------+
             | Reconcile |
             +-----------+
                   |
                   v
             Actual State
                   |
                   |
                   +------ compare
                   |
                   v
              Difference?
                /     \
              No       Yes
              |         |
              v         v
          Continue    Correct
                        |
                        v
                   Actual State
```

The loop repeats.

---

# 17. Why a Control Loop Is Better Than a Script

A deployment script normally executes a sequence:

```text
Step 1
Step 2
Step 3
Step 4
Done
```

If somebody changes the environment afterward, the script does not automatically run again.

A controller continuously evaluates state:

```text
Desired
   |
   v
Controller
   |
   v
Actual
   |
   v
Changed?
   |
   +--> Yes -> reconcile
   |
   +--> No  -> observe
```

This makes the system resilient to state changes.

---

# 18. Push vs Pull Architecture

## Push

```text
Git
 |
 v
CI
 |
 | cluster credentials
 v
EKS
```

CI initiates the deployment.

## Pull

```text
Git
 |
 v
Argo CD
 |
 | cluster credentials
 v
EKS
```

The controller associated with the cluster initiates reconciliation.

---

# 19. Security Difference Between Push and Pull

Push architecture:

```text
Jenkins
   |
   +--> Production credentials
   |
   +--> Kubernetes API
```

Potential blast radius:

```text
Jenkins compromised
       |
       v
Production cluster accessible
```

Pull architecture:

```text
Jenkins
   |
   +--> Git
   |
   +--> ECR

Argo CD
   |
   +--> EKS
```

The access boundary is separated.

This does not automatically make the architecture secure.

Argo CD itself must still be strongly protected.

---

# 20. GitOps Trust Boundaries

A trust boundary is a point where authority or credentials cross from one system to another.

Important boundaries include:

```text
Developer -> Application Git

CI -> ECR

CI -> GitOps Repository

Argo CD -> Git Repository

Argo CD -> Kubernetes API

Kubernetes -> AWS APIs
```

Each should be protected independently.

---

# 21. Trust Boundary: Developer to Git

Developers should generally have access to application repositories based on team ownership.

Production GitOps configuration may have stricter access.

Example:

```text
Developer
    |
    v
Application Repo

Release / Platform Team
    |
    v
GitOps Repo
```

This creates separation between source-code changes and production deployment intent.

---

# 22. Trust Boundary: CI to ECR

CI requires permission to push artifacts.

Example:

```text
Jenkins
   |
   | ECR push
   v
Amazon ECR
```

The CI identity should not automatically have unrestricted AWS permissions.

Use least privilege.

---

# 23. Trust Boundary: CI to GitOps Repository

If CI updates image tags automatically:

```text
CI
 |
 | Git write
 v
GitOps repository
```

the CI identity requires Git write permission.

This permission is powerful because modifying GitOps configuration can indirectly change production.

Therefore:

- Restrict the repository.
- Use a dedicated bot/service identity where appropriate.
- Limit branch access.
- Require pull requests where appropriate.
- Protect production branches.
- Audit changes.

---

# 24. Trust Boundary: Argo CD to Git

Argo CD needs read access to repositories.

For private repositories, credentials may use:

- SSH
- HTTPS tokens
- Credential templates
- Organization-specific identity mechanisms

Argo CD should only have access to repositories it needs.

---

# 25. Trust Boundary: Argo CD to EKS

Argo CD requires access to target Kubernetes APIs.

This is one of the most powerful trust boundaries.

If Argo CD has cluster-admin on a production cluster:

```text
Compromised Argo CD
      |
      v
Potential full cluster control
```

Therefore:

- Protect Argo CD strongly.
- Restrict users with Argo CD RBAC.
- Restrict Application destinations.
- Use AppProjects.
- Use least privilege where feasible.
- Separate highly sensitive clusters if required.

---

# 26. Trust Boundary: Kubernetes to AWS

Workloads may access AWS APIs through AWS IAM mechanisms.

Examples:

```text
Pod
 |
 v
AWS service
```

In EKS, IAM roles for workloads can be used to avoid embedding long-lived AWS credentials.

This is separate from Argo CD's own cluster-management identity.

---

# 27. Principle of Least Privilege

Every identity should have only the permissions it needs.

Example:

```text
CI:
- Push approved images to ECR
- Update designated GitOps branch/path

Argo CD:
- Read designated repositories
- Manage designated clusters/namespaces/resources

Developer:
- Modify application source
- Create deployment PRs
```

Avoid:

```text
Everyone -> admin
```

---

# 28. Source of Truth Is a Governance Decision

Choosing Git as source of truth is not just a technical decision.

It changes the organization's operating model.

Instead of:

```text
"Who changed production?"
```

the organization aims for:

```text
"Which Git change caused production to change?"
```

That makes operational processes more deterministic.

---

# 29. Repository Architecture

A GitOps repository can be organized in multiple ways.

Common approaches:

1. Environment-based
2. Application-based
3. Cluster-based
4. Platform/application split
5. Monorepo
6. Multi-repository
7. Hybrid

There is no single universal structure.

---

# 30. Environment-Based Repository

Example:

```text
gitops/
├── dev/
│   ├── cart/
│   └── catalog/
├── qa/
│   ├── cart/
│   └── catalog/
└── prod/
    ├── cart/
    └── catalog/
```

Advantages:

- Environment ownership is obvious.
- Promotion can be easy to visualize.

Challenges:

- Duplication can grow.
- Shared application configuration can become repetitive.

---

# 31. Application-Based Repository

Example:

```text
gitops/
├── cart/
│   ├── base/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── catalog/
    ├── base/
    ├── dev/
    ├── qa/
    └── prod/
```

Advantages:

- Application ownership is clear.
- All deployment configuration for one service is together.

Challenges:

- Cross-application platform configuration can be harder to organize.

---

# 32. Cluster-Based Repository

Example:

```text
gitops/
├── clusters/
│   ├── dev-eks/
│   ├── qa-eks/
│   └── prod-eks/
└── applications/
```

Advantages:

- Cluster-specific configuration is obvious.
- Useful for multi-cluster environments.

Challenges:

- Application configuration may be spread across cluster directories.

---

# 33. Platform/Application Split

A production enterprise may use:

```text
platform-gitops/
├── ingress/
├── monitoring/
├── security/
└── argocd/

application-gitops/
├── cart/
├── catalog/
├── orders/
└── payment/
```

This creates a strong ownership boundary.

For example:

```text
Platform Team
    |
    v
platform-gitops

Application Teams
    |
    v
application-gitops
```

---

# 34. Monorepo vs Multi-Repo

## Monorepo

```text
one GitOps repository
        |
        +--> all applications
        +--> all environments
        +--> platform
```

Advantages:

- Central visibility
- Easy global changes
- Simple Argo CD repository configuration

Disadvantages:

- Large repository
- Many teams share one repository
- Complex access control

## Multi-repo

```text
cart-gitops
catalog-gitops
orders-gitops
platform-gitops
```

Advantages:

- Strong ownership
- Smaller repositories
- More independent team workflows

Disadvantages:

- More repository management
- More Argo CD configuration
- Cross-repository governance

---

# 35. Hybrid Repository Architecture

A common enterprise compromise:

```text
Platform GitOps
        |
        +--> Argo CD
        +--> monitoring
        +--> ingress
        +--> cluster configuration

Application GitOps
        |
        +--> cart
        +--> catalog
        +--> orders
        +--> payment
```

This can provide centralized platform governance while allowing application teams autonomy.

---

# 36. Environment Architecture

There are several ways to represent environments.

## Same cluster, different namespaces

```text
EKS
├── dev
├── qa
└── prod
```

## Separate clusters

```text
EKS-DEV
EKS-QA
EKS-PROD
```

## Separate AWS accounts and clusters

```text
Dev Account
   |
   v
EKS-DEV

QA Account
   |
   v
EKS-QA

Prod Account
   |
   v
EKS-PROD
```

The third model generally provides stronger isolation.

---

# 37. Namespace Isolation

Using namespaces can provide logical boundaries:

```text
EKS
 |
 +-- roboshop-dev
 |
 +-- roboshop-qa
 |
 +-- roboshop-prod
```

This can work well for development or lower-risk environments.

For production isolation, separate clusters/accounts may be preferable depending on requirements.

---

# 38. Cluster Isolation

Separate clusters provide stronger boundaries:

```text
                AWS Organization
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
     Dev EKS         QA EKS        Prod EKS
```

Benefits include:

- Failure isolation
- Security isolation
- Independent upgrades
- Independent scaling
- Different access policies

Costs include:

- More infrastructure
- More operational overhead
- More monitoring
- More Argo CD management

---

# 39. Account Isolation

AWS account separation provides another boundary:

```text
Dev Account
    |
    v
Dev EKS

QA Account
    |
    v
QA EKS

Production Account
    |
    v
Production EKS
```

This is commonly useful because IAM, billing, quotas, and security controls can be separated.

---

# 40. Production Architecture Selection

Environment isolation should be chosen based on:

- Security requirements
- Compliance
- Blast radius
- Team size
- Cost
- Availability requirements
- Data sensitivity
- Operational maturity
- Upgrade strategy

There is no universal rule:

```text
"Production must always be a separate cluster."
```

But high-risk production systems often benefit from stronger isolation.

---

# 41. Single-Cluster GitOps Architecture

A simple architecture:

```text
                    GitOps Repo
                         |
                         v
                    +---------+
                    | Argo CD |
                    +---------+
                         |
                         v
                  Kubernetes API
                         |
                         v
                       EKS
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
     Dev NS            QA NS            Prod NS
```

This can be suitable for smaller environments.

---

# 42. Single-Cluster Failure Domain

The major concern is:

```text
EKS failure
   |
   +--> Dev affected
   +--> QA affected
   +--> Prod affected
```

All environments share the same cluster failure domain.

This may be unacceptable for critical production.

---

# 43. Multi-Cluster Architecture

A stronger model:

```text
                    Git
                     |
                     v
               Central Argo CD
                /      |      \
               /       |       \
              v        v        v
          EKS-DEV    EKS-QA   EKS-PROD
```

Now:

```text
EKS-DEV failure
     |
     X
EKS-QA and EKS-PROD remain separate
```

This reduces shared runtime failure domains.

---

# 44. Centralized Argo CD as GitOps Control Plane

Argo CD can be deployed into a management cluster:

```text
Management EKS
|
+-- Argo CD
|
+-- Monitoring
|
+-- Platform services
```

It manages:

```text
EKS-DEV
EKS-QA
EKS-PROD
```

Architecture:

```text
                    Git Repository
                          |
                          v
                 +------------------+
                 | Central Argo CD   |
                 | Management EKS    |
                 +------------------+
                    /      |      \
                   /       |       \
                  v        v        v
               DEV EKS   QA EKS  PROD EKS
```

This is a major enterprise pattern.

---

# 45. Centralized Argo CD Benefits

Benefits include:

- Centralized visibility
- Consistent GitOps policy
- One UI/CLI entry point
- Centralized ApplicationSets
- Centralized repository configuration
- Standardized RBAC

But it also introduces a management-plane dependency.

---

# 46. Centralized Argo CD Risks

If the central Argo CD cluster fails:

```text
Central Argo CD failure
       |
       +--> New syncs delayed
       +--> Drift correction delayed
       +--> Multi-cluster management impacted
```

Existing applications generally continue running.

Therefore the architecture should include:

- HA
- Monitoring
- Backup
- Recovery
- Secure access
- Capacity planning

---

# 47. Decentralized Argo CD Architecture

An alternative is:

```text
EKS-DEV
  |
  +-- Argo CD

EKS-QA
  |
  +-- Argo CD

EKS-PROD
  |
  +-- Argo CD
```

Benefits:

- Strong cluster isolation
- Local control plane
- Failure of one Argo CD does not directly affect others

Costs:

- More Argo CD instances
- More administration
- More configuration
- More governance complexity

---

# 48. Centralized vs Decentralized

| Area | Centralized | Decentralized |
|---|---|---|
| Management | Easier centrally | Distributed |
| Visibility | Strong | Fragmented |
| Isolation | Lower | Strong |
| Number of Argo CD instances | Fewer | More |
| Blast radius | Larger | Smaller |
| Governance | Centralized | Local |
| Multi-cluster management | Excellent | More complex |
| Failure isolation | Lower | Higher |

The correct choice depends on organizational requirements.

---

# 49. Hub-and-Spoke Model

Another useful way to visualize centralized GitOps:

```text
                    Git
                     |
                     v
                    Hub
               +-----------+
               | Argo CD   |
               +-----------+
                /   |   \
               /    |    \
              v     v     v
            DEV    QA    PROD
            EKS    EKS    EKS
```

The hub is the management plane.

The clusters are spokes.

This pattern can work well for organizations managing many EKS clusters.

---

# 50. Multi-Cluster Cluster Registration

For Argo CD to manage another cluster, that cluster must be registered.

Conceptually:

```text
Target EKS
    |
    +--> API endpoint
    +--> Authentication
    +--> Credentials
    |
    v
Argo CD
```

Argo CD stores information required to connect to the cluster.

Later files will cover the actual cluster registration commands and Kubernetes secrets.

---

# 51. Cluster Metadata

Multi-cluster management becomes much more powerful when clusters have metadata.

Example:

```text
cluster: eks-dev
environment: dev
region: ap-south-1
team: platform
```

Another:

```text
cluster: eks-prod
environment: prod
region: ap-south-1
team: platform
```

ApplicationSets can use labels to select clusters.

This enables dynamic multi-cluster deployment.

---

# 52. ApplicationSet and Architecture

Instead of manually declaring:

```text
cart-dev
cart-qa
cart-prod
```

an ApplicationSet can use cluster metadata.

Conceptually:

```text
Clusters
 |
 +--> environment=dev
 +--> environment=qa
 +--> environment=prod
          |
          v
   ApplicationSet
          |
          v
 Generate Applications
```

This allows the GitOps control plane to scale with the number of clusters.

---

# 53. Multi-Cluster Failure Scenario

Suppose:

```text
Central Argo CD
   |
   +--> EKS-DEV
   +--> EKS-QA
   +--> EKS-PROD
```

EKS-QA becomes unreachable.

Expected behavior:

```text
DEV -> continues
QA -> sync unavailable
PROD -> continues
```

A well-designed architecture should isolate cluster failures.

Argo CD should report the affected cluster/application as unavailable rather than treating all clusters as failed.

---

# 54. Multi-Account Failure Scenario

Suppose production AWS account has an IAM problem.

```text
Central Argo CD
       |
       +--> DEV -> OK
       +--> QA -> OK
       +--> PROD -> Authentication failure
```

The troubleshooting path should focus on:

- Cluster credentials
- AWS authentication
- IAM permissions
- EKS access
- Network reachability
- Kubernetes API

Do not immediately assume Argo CD itself is broken.

---

# 55. Failure Domains

A failure domain is a boundary within which a failure can propagate.

Examples:

```text
Pod failure
    |
    v
Single workload

Node failure
    |
    v
Pods on node

Cluster failure
    |
    v
All workloads in cluster

AWS account issue
    |
    v
Multiple resources in account

Central Argo CD failure
    |
    v
Management/reconciliation across targets
```

Production architecture should minimize unnecessary shared failure domains.

---

# 56. Blast Radius

Blast radius is the scope of impact if a component is compromised or fails.

For Argo CD:

```text
Argo CD
   |
   +--> 1 namespace
```

has smaller blast radius than:

```text
Argo CD
   |
   +--> cluster-admin
   |
   +--> every cluster
   |
   +--> every namespace
```

Therefore:

```text
Least privilege
+
Application boundaries
+
Project restrictions
=
Reduced blast radius
```

---

# 57. AppProject as an Architectural Boundary

Argo CD Projects can define boundaries around:

- Allowed repositories
- Allowed destination clusters
- Allowed namespaces
- Resource types
- Roles

Conceptually:

```text
AppProject: roboshop-prod
 |
 +--> Git repository allowlist
 +--> Prod cluster allowlist
 +--> roboshop namespace
 +--> Allowed resource types
```

Projects become important for enterprise multi-team GitOps.

Detailed Project configuration will be covered later.

---

# 58. GitOps Team Isolation

Suppose:

```text
Team A -> cart
Team B -> payment
Team C -> catalog
```

A production GitOps system should avoid allowing Team A to modify Team B's workloads.

Possible controls:

```text
Git permissions
+
AppProject
+
Argo CD RBAC
+
Kubernetes RBAC
```

Defense in depth is preferable.

---

# 59. GitOps Architecture for RoboShop

A production-oriented design can be:

```text
                         GitHub
                           |
            +--------------+--------------+
            |                             |
            v                             v
     Application Repos              GitOps Repo
            |                             |
            v                             |
       Jenkins / Actions                 |
            |                             |
      +-----+------+                      |
      |            |                      |
      v            v                      |
   Security       ECR                     |
      |            |                      |
      +-----+------+                      |
            |                             |
            +-------- image/version ------+
                                          |
                                          v
                                     Central Argo CD
                                          |
                         +----------------+----------------+
                         |                |                |
                         v                v                v
                      EKS-DEV          EKS-QA           EKS-PROD
                         |                |                |
                         v                v                v
                      RoboShop         RoboShop          RoboShop
                         |                |                |
                         +----------------+----------------+
                                          |
                                      ALB Ingress
```

This is the target architecture that later files will turn into actual YAML.

---

# 60. GitOps Architecture for RoboShop: Responsibility Matrix

| Component | Responsibility |
|---|---|
| GitHub | Application source |
| Jenkins/GitHub Actions | CI |
| SonarQube | Code quality |
| Trivy | Container/security scanning |
| Veracode | Application security |
| Docker | Container build |
| ECR | Image registry |
| GitOps repo | Desired deployment state |
| Argo CD | GitOps reconciliation |
| EKS | Kubernetes runtime |
| ALB Controller | AWS ALB integration |
| ALB | External traffic |
| Prometheus | Metrics |
| Grafana | Visualization |
| ELK | Logs |
| Terraform | AWS infrastructure |

---

# 61. Separation Between Infrastructure and Application GitOps

A practical boundary can be:

```text
Terraform
   |
   v
AWS Infrastructure
   |
   +--> VPC
   +--> EKS
   +--> IAM
   +--> ECR
   +--> RDS
   +--> networking
```

Then:

```text
Argo CD
   |
   v
Kubernetes Applications
   |
   +--> RoboShop
   +--> monitoring
   +--> ingress
```

This prevents two systems from unintentionally fighting over the same resource.

---

# 62. Ownership Model

A mature platform should define:

```text
Terraform
   owns:
   AWS infrastructure

Argo CD
   owns:
   Kubernetes workloads

Application Team
   owns:
   application behavior

Security Team
   owns:
   security policies

Platform Team
   owns:
   GitOps platform / EKS platform
```

Ownership must be documented.

---

# 63. Avoiding Two Sources of Truth

A common problem is:

```text
Terraform says:
replicas = 3

Argo CD says:
replicas = 5
```

Now two systems attempt to control the same field.

This can create:

```text
Terraform -> 3
Argo CD -> 5
Terraform -> 3
Argo CD -> 5
...
```

Avoid overlapping ownership.

The desired state should have one authoritative manager for each resource field where practical.

---

# 64. Terraform and Argo CD Boundary

A strong practical boundary for the user's environment is:

```text
Terraform:
- VPC
- EKS
- IAM
- ECR
- RDS
- AWS networking
- infrastructure resources

Argo CD:
- Namespace
- Deployment
- Service
- Ingress
- HPA
- ConfigMap
- application-level RBAC
- Helm applications
```

Some platform resources can be managed either way depending on organizational design.

The important rule is to avoid conflicting ownership.

---

# 65. GitOps Bootstrap Architecture

A new EKS environment can be bootstrapped as:

```text
Terraform
   |
   v
VPC + EKS
   |
   v
Argo CD installation
   |
   v
Root Application
   |
   +--> Projects
   +--> Applications
   +--> ApplicationSets
   +--> Platform components
   |
   v
RoboShop workloads
```

This is how GitOps can become self-expanding after the initial platform bootstrap.

---

# 66. Bootstrap Problem

There is a fundamental bootstrap question:

> How do we install the GitOps controller that will manage everything?

A common answer is:

```text
Terraform / Helm / bootstrap automation
        |
        v
Argo CD
        |
        v
GitOps-managed resources
```

After bootstrap, Argo CD manages the majority of Kubernetes configuration.

This is known as a bootstrap boundary.

---

# 67. GitOps Bootstrap Security

The bootstrap process is highly privileged.

It may create:

- Argo CD
- Cluster credentials
- Projects
- Applications
- RBAC
- Platform components

Therefore:

```text
Bootstrap credentials
```

should be protected carefully.

After bootstrap, permissions can be reduced.

---

# 68. Enterprise GitOps Control Plane

A mature architecture can look like:

```text
                       Git Providers
                      /             \
                     /               \
            Application Git       GitOps Git
                   |                  |
                   |                  |
                   +--------+---------+
                            |
                            v
                     CI / Validation
                            |
                            v
                           ECR
                            |
                            v
                     Central Argo CD
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
          Dev EKS         QA EKS          Prod EKS
            |               |               |
            v               v               v
       Applications    Applications    Applications
```

Around this architecture:

```text
                 +--------------------+
                 | Security / Policy  |
                 +--------------------+
                           |
                           v
                 Git + CI + Argo CD

                 +--------------------+
                 | Observability      |
                 +--------------------+
                           |
                           v
                Prometheus / Grafana / ELK
```

---

# 69. Enterprise Architecture Layers

A useful architectural decomposition is:

## Source layer

```text
Git
```

## Validation layer

```text
CI
Security
Policy
```

## Artifact layer

```text
ECR
```

## Desired-state layer

```text
GitOps repository
```

## Reconciliation layer

```text
Argo CD
```

## Runtime layer

```text
EKS
```

## External infrastructure layer

```text
AWS
```

## Observability layer

```text
Prometheus
Grafana
ELK
```

---

# 70. Failure Isolation in Enterprise GitOps

A good architecture tries to ensure:

```text
Application failure
!=
Cluster failure

Cluster failure
!=
Other cluster failure

GitOps controller failure
!=
Immediate application outage

Development failure
!=
Production failure
```

This is achieved through:

- Cluster isolation
- Account isolation
- Namespace isolation
- AppProject boundaries
- RBAC
- HA
- Independent dependencies
- Controlled promotion

---

# 71. GitOps Availability Design

Critical Argo CD installations should consider:

```text
HA
|
+--> Multiple controller replicas
+--> Multiple API server replicas where applicable
+--> Repository access resilience
+--> Redis resilience
+--> Monitoring
+--> Recovery
```

The exact HA architecture depends on Argo CD version and deployment topology.

Detailed component-level HA is covered in:

```text
04-ArgoCD-Architecture-and-Components.md
```

---

# 72. GitOps Scalability

GitOps scaling is not just about Kubernetes cluster size.

The GitOps control plane must scale with:

```text
Number of applications
Number of resources
Number of repositories
Number of clusters
Number of environments
Number of users
Number of reconciliation events
```

A small lab:

```text
10 applications
1 cluster
```

is very different from:

```text
1000 applications
50 clusters
10 AWS accounts
```

The architecture must evolve accordingly.

---

# 73. Scaling Repository Management

At scale, avoid unnecessary duplication.

Instead of:

```text
1000 manually copied YAML files
```

use reusable mechanisms such as:

- Helm
- Kustomize
- ApplicationSets
- Templates
- Environment overlays

The goal is:

```text
Standardization
+
Controlled customization
```

---

# 74. Scaling Application Management

Without ApplicationSets:

```text
100 clusters
x
100 applications
=
10,000 Application resources
```

Manually maintaining these becomes difficult.

ApplicationSets can generate Applications from:

```text
Clusters
Directories
Lists
Git repositories
Pull requests
Matrices
Merges
```

This will be covered deeply in File 09.

---

# 75. Scaling Cluster Management

A centralized Argo CD platform can register clusters such as:

```text
eks-dev-01
eks-dev-02
eks-qa-01
eks-prod-01
eks-prod-02
```

Cluster metadata can identify:

```text
environment=prod
region=ap-south-1
team=platform
```

ApplicationSets can then target appropriate clusters.

---

# 76. GitOps and Regional Architecture

A global organization may have:

```text
AP-SOUTH-1
   |
   +--> EKS Production

US-EAST-1
   |
   +--> EKS Production

EU-WEST-1
   |
   +--> EKS Production
```

A centralized Argo CD may manage all of them, or regional Argo CD instances may be used.

Decision factors include:

- Latency
- Network reliability
- Regulatory requirements
- Failure isolation
- Team ownership
- Operational complexity

---

# 77. Central vs Regional Argo CD

Central:

```text
One Argo CD
  |
  +--> AP
  +--> US
  +--> EU
```

Regional:

```text
Argo CD AP
  |
  +--> AP clusters

Argo CD US
  |
  +--> US clusters

Argo CD EU
  |
  +--> EU clusters
```

Regional architectures reduce blast radius but increase management overhead.

---

# 78. GitOps Disaster Recovery Architecture

A production DR plan can look like:

```text
Primary Git repository
        |
        +--> Backup / replica
        |
        v
DR infrastructure
        |
        v
New EKS
        |
        v
Argo CD bootstrap
        |
        v
GitOps applications
        |
        v
Application recovery
```

Data recovery remains a separate concern.

---

# 79. What Must Be Backed Up?

For GitOps DR, consider:

### Git

- Repositories
- Branches/tags
- Access recovery
- Repository configuration

### Argo CD

- Applications
- Projects
- Repository configuration
- Cluster registrations
- RBAC configuration
- Important controller configuration

### Kubernetes

- Persistent data
- Required cluster-specific resources

### AWS

- Terraform state
- Infrastructure definitions
- Required secrets
- DNS
- Certificates
- Data backups

Git alone is not sufficient for complete recovery.

---

# 80. GitOps Observability Architecture

A production control plane should expose health information.

```text
Argo CD
   |
   +--> Metrics
   |
   v
Prometheus
   |
   v
Grafana
```

Logs:

```text
Argo CD
   |
   v
ELK
```

Alerts can cover:

```text
Application Degraded
Sync failures
Repository errors
Cluster unreachable
Controller failures
```

---

# 81. GitOps Operational SLO Thinking

For critical GitOps platforms, useful operational objectives may include:

```text
Argo CD availability
Repository availability
Sync latency
Successful sync rate
Cluster connectivity
Application health
```

The exact targets should be defined by business requirements.

---

# 82. GitOps Security Architecture

A mature design can be visualized as:

```text
                Identity Provider
                       |
                       v
                 Argo CD SSO
                       |
                       v
                  Argo CD RBAC
                       |
                       v
                  AppProjects
                       |
                       v
              Repository / Cluster
                restrictions
                       |
                       v
                      EKS
```

At the Git level:

```text
Git
 |
 +--> Branch protection
 +--> PR reviews
 +--> CODEOWNERS
 +--> Secret scanning
 +--> Audit
```

At the runtime:

```text
EKS
 |
 +--> Kubernetes RBAC
 +--> NetworkPolicy
 +--> Pod security
 +--> AWS IAM
```

Security is therefore layered.

---

# 83. GitOps Policy Enforcement

Policy can be applied before synchronization:

```text
Pull Request
   |
   +--> YAML validation
   +--> Helm validation
   +--> Security scanning
   +--> Policy checks
   |
   v
Merge
```

Policy can also be enforced at runtime:

```text
Argo CD
   |
   v
Kubernetes admission
   |
   v
Allowed / rejected
```

This creates defense in depth.

---

# 84. Production GitOps Change Flow

A mature change may follow:

```text
Developer
    |
    v
Application Git
    |
    v
CI
    |
    +--> Test
    +--> Security
    |
    v
ECR
    |
    v
GitOps change
    |
    v
Pull Request
    |
    +--> Review
    +--> Policy
    +--> Approval
    |
    v
Merge
    |
    v
Argo CD
    |
    v
Compare
    |
    v
Sync
    |
    v
EKS
    |
    v
Health checks
    |
    v
Observability
```

This is the complete lifecycle.

---

# 85. GitOps and Progressive Delivery Boundary

GitOps declares the intended release state.

A progressive delivery controller can manage the rollout mechanics.

Conceptually:

```text
Git
 |
 v
Argo CD
 |
 v
Rollout resource
 |
 v
Progressive delivery controller
 |
 +--> Canary
 +--> Analysis
 +--> Promotion
```

This prevents Argo CD from becoming responsible for every rollout algorithm.

---

# 86. GitOps and External Systems

GitOps can integrate with systems outside Kubernetes, but ownership must be clear.

For example:

```text
Terraform -> AWS infrastructure
Argo CD -> Kubernetes
AWS Controller -> AWS resources derived from Kubernetes
```

Avoid having multiple systems independently modify the same resource without a defined contract.

---

# 87. Architecture Principle: Clear Ownership

A highly useful production rule is:

> Every managed object should have a clearly identified source of truth and owner.

Example:

```text
VPC
 -> Terraform

EKS cluster
 -> Terraform

Kubernetes Deployment
 -> Argo CD

Application source
 -> GitHub

Container image
 -> ECR

Database data
 -> Database/backup system
```

This eliminates ambiguity.

---

# 88. Architecture Principle: Immutable Promotion

Promote artifacts rather than rebuilding them.

Preferred:

```text
Build once
   |
   v
ECR
   |
   +--> DEV
   +--> QA
   +--> PROD
```

Avoid:

```text
Build for DEV
Build again for QA
Build again for PROD
```

unless the organization has a specific reason.

---

# 89. Architecture Principle: Environment Configuration Is Separate

The same image can use different environment configuration.

Example:

```text
cart image:
2026.08.19-abc123
```

Dev:

```text
database = dev-db
```

QA:

```text
database = qa-db
```

Prod:

```text
database = prod-db
```

The artifact remains the same.

Environment-specific configuration belongs in the appropriate GitOps configuration layer.

---

# 90. Architecture Principle: Production Should Be Deliberate

Automation does not mean every production change should be automatic.

A production strategy may be:

```text
DEV
 |
 | automatic
 v
QA
 |
 | controlled
 v
PROD
 |
 | approval
 v
Sync
```

The right level of automation depends on:

- Risk
- Application criticality
- Test coverage
- Compliance
- Business requirements

---

# 91. Architecture Principle: Fail Safely

GitOps systems should be designed so that failures do not unnecessarily destroy healthy workloads.

Examples:

- Avoid uncontrolled pruning.
- Use appropriate sync policies.
- Validate manifests before merge.
- Use health checks.
- Use progressive delivery where appropriate.
- Protect production branches.
- Restrict permissions.

Automation should reduce operational risk, not amplify it.

---

# 92. Architecture Principle: Observe Before Correcting

Automated self-healing is powerful.

But not every runtime difference should immediately be overwritten.

Example:

```text
HPA changes replicas
```

This can be expected.

Therefore:

```text
Detect difference
      |
      v
Understand ownership
      |
      v
Decide whether correction is appropriate
```

This is why advanced diff configuration matters.

---

# 93. Architecture Principle: Minimize Blast Radius

A secure architecture should prefer:

```text
Application
  |
  v
AppProject
  |
  v
Allowed namespace
  |
  v
Allowed cluster
```

rather than:

```text
Application
  |
  v
Cluster admin everywhere
```

This becomes especially important when one Argo CD instance manages multiple production clusters.

---

# 94. Architecture Principle: Separate Management Plane and Data Plane

In centralized GitOps:

```text
Management Plane
    |
    +--> Argo CD
    +--> Git integration
    +--> RBAC
    +--> ApplicationSets
    |
    v
Data / Runtime Plane
    |
    +--> EKS workloads
    +--> Services
    +--> ALB
```

The management plane controls the runtime plane but is not normally part of application request traffic.

This distinction is useful when planning HA and DR.

---

# 95. Architecture Principle: Runtime Must Survive Control-Plane Interruption

A good GitOps design assumes:

```text
Argo CD temporarily unavailable
```

without immediately causing:

```text
Application outage
```

Existing Kubernetes workloads should continue to run.

The impact is primarily:

```text
No new reconciliation
No new sync
Delayed drift correction
```

This is an important production property.

---

# 96. Architecture Principle: Recoverability

The desired state should be recoverable.

Ask:

```text
Can we rebuild the application configuration?

Can we reinstall Argo CD?

Can we reconnect target clusters?

Can we recreate Applications?

Can we restore secrets?

Can we recover databases?
```

These questions form the basis of a real DR strategy.

---

# 97. GitOps Architecture Decision Matrix

| Decision | Option A | Option B |
|---|---|---|
| Repository | Monorepo | Multi-repo |
| Environment | Namespace | Cluster |
| AWS isolation | Same account | Separate accounts |
| Argo CD | Central | Per-cluster |
| Promotion | Automatic | Approval |
| Manifest engine | Helm | Kustomize |
| Sync | Manual | Automated |
| Secret model | External Secrets | Sealed/SOPS/etc. |
| Infrastructure | Terraform | Other IaC |
| Artifact | Tag | Digest |

These decisions should be made according to requirements rather than personal preference.

---

# 98. Production Architecture Recommendation for the RoboShop Scenario

For a production-oriented RoboShop environment, a reasonable target architecture is:

```text
                    AWS Organization
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          DEV Account   QA Account   PROD Account
             |             |             |
             v             v             v
          EKS-DEV       EKS-QA       EKS-PROD
             ^             ^             ^
              \            |            /
               \           |           /
                +----------+----------+
                           |
                      Central Argo CD
                       Management
                           |
                           v
                     GitOps Repository
                           ^
                           |
                      CI / Approval
                           ^
                           |
                 Application Repositories
                           |
                           v
                          ECR
```

This is a conceptual target, not a mandatory architecture.

It provides:

- Environment isolation
- Centralized GitOps
- Controlled production access
- Immutable artifact promotion
- Multi-cluster management
- Clear ownership

---

# 99. Why ALB Instead of API Gateway Here

For this architecture, external application traffic should follow:

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
RoboShop services
```

The architecture should not introduce API Gateway because it is not part of the stated RoboShop implementation.

The GitOps configuration will therefore manage Kubernetes Ingress resources compatible with the AWS Load Balancer Controller.

---

# 100. Architecture Review Questions

Before approving a production GitOps architecture, ask:

### Source control

- Where is desired state stored?
- Who can modify it?
- How is production protected?

### CI

- Who builds images?
- Where are images stored?
- How are images scanned?
- How are artifacts promoted?

### Argo CD

- Where does Argo CD run?
- How is it secured?
- Which repositories can it access?
- Which clusters can it access?
- Which namespaces can each Project manage?

### Kubernetes

- What does Argo CD own?
- What do other controllers own?
- What happens during drift?

### AWS

- Which account owns each cluster?
- How does Argo CD authenticate?
- How is ALB managed?

### Operations

- How is Argo CD monitored?
- What happens if Git is unavailable?
- What happens if Argo CD is unavailable?
- How is DR performed?

---

# 101. Architecture Review: Five Critical Questions

For any GitOps platform, be able to answer:

## 1. What is the source of truth?

```text
Git
```

## 2. Who reconciles desired state?

```text
Argo CD
```

## 3. Who runs the workloads?

```text
Kubernetes / EKS
```

## 4. Who creates the artifact?

```text
CI
```

## 5. Who stores the artifact?

```text
Amazon ECR
```

This simple model prevents many architectural misunderstandings.

---

# 102. Common Architecture Mistakes

## Mistake 1

Treating Git as backup only.

Git should represent desired state, not merely archive YAML.

## Mistake 2

Giving CI cluster-admin.

This increases the CI blast radius.

## Mistake 3

Using mutable images.

This weakens reproducibility.

## Mistake 4

Allowing multiple systems to own the same resources.

This creates conflicts.

## Mistake 5

Centralizing Argo CD without considering HA.

The management plane becomes a risk.

## Mistake 6

Giving centralized Argo CD unrestricted access to all clusters.

This increases blast radius.

## Mistake 7

Using one repository structure without considering team ownership.

Repository design must match organization and scale.

---

# 103. Production Architecture Checklist

## Git

- [ ] Desired state is declarative.
- [ ] Git is the authoritative source for GitOps-managed state.
- [ ] Production branches are protected.
- [ ] Pull requests are reviewed.
- [ ] Repository recovery is defined.

## CI

- [ ] Build and tests are automated.
- [ ] SonarQube is integrated where required.
- [ ] Trivy scans images.
- [ ] Veracode is integrated where required.
- [ ] Images use immutable identifiers.
- [ ] ECR is the artifact registry.

## Argo CD

- [ ] Argo CD architecture is documented.
- [ ] Repository access is restricted.
- [ ] Cluster access is restricted.
- [ ] RBAC is configured.
- [ ] AppProjects define boundaries.
- [ ] HA is considered.
- [ ] Monitoring is enabled.
- [ ] Recovery is tested.

## EKS

- [ ] Cluster ownership is defined.
- [ ] Namespace strategy exists.
- [ ] Kubernetes RBAC is configured.
- [ ] Resource ownership is clear.
- [ ] ALB integration is documented.

## Multi-cluster

- [ ] Target clusters are registered.
- [ ] Cluster labels are defined.
- [ ] Environment targeting is documented.
- [ ] Cross-account access is controlled.
- [ ] Failure isolation is tested.

---

# 104. Interview Question: Explain GitOps Architecture

### Answer

> A GitOps architecture separates application source, CI, artifact storage, desired deployment state, reconciliation, and runtime. Application code is stored in Git and built through Jenkins or GitHub Actions. CI performs testing and security checks and pushes an immutable image to ECR. The deployment configuration is maintained in a GitOps repository. Argo CD continuously reads that desired state, compares it with Kubernetes, and reconciles the target EKS cluster. Kubernetes controllers then create and manage the runtime resources. In larger environments, a centralized Argo CD can manage multiple EKS clusters, potentially across separate AWS accounts, using Projects, RBAC, cluster registration, and ApplicationSets to control scope.

---

# 105. Interview Question: Why Is GitOps a Control Loop?

### Answer

> GitOps is a control loop because the system continuously observes desired state and actual state, compares them, and takes corrective action when they differ. Argo CD is the reconciliation controller in a Kubernetes GitOps architecture. This means GitOps does not simply deploy once; it continuously works toward convergence.

---

# 106. Interview Question: What Is the Difference Between GitOps and CI/CD?

### Answer

> CI/CD is a broad software delivery practice. CI focuses on integrating, testing, validating, and building artifacts. Traditional CD may push those artifacts directly into environments. GitOps is a specific CD and operations model in which desired state is stored in Git and continuously reconciled by a controller. Argo CD provides that GitOps reconciliation layer for Kubernetes.

---

# 107. Interview Question: Why Separate Application Git and GitOps Git?

### Answer

> Separating application source from deployment configuration can provide clearer ownership, security boundaries, and release control. The application repository contains source code and build logic, while the GitOps repository contains the desired runtime configuration. This also allows the same application artifact to be promoted through environments by changing deployment configuration rather than rebuilding the application.

---

# 108. Interview Question: How Does GitOps Reduce Blast Radius?

### Answer

> GitOps can reduce blast radius by separating CI from direct cluster access, restricting Argo CD through Projects and RBAC, limiting target clusters and namespaces, and isolating environments through clusters or AWS accounts. The goal is to ensure that compromising one component does not automatically provide unrestricted access to the entire production platform.

---

# 109. Interview Question: Can One Argo CD Manage Multiple EKS Clusters?

### Answer

> Yes. Argo CD can operate as a centralized GitOps control plane. Target EKS clusters are registered with Argo CD, and Applications specify their destination clusters. ApplicationSets can dynamically generate Applications for multiple clusters based on cluster metadata or other generators. In a multi-account AWS architecture, access must be carefully controlled using appropriate authentication, RBAC, network connectivity, and least-privilege boundaries.

---

# 110. Interview Scenario: One Cluster Is Down

### Question

A centralized Argo CD manages three EKS clusters. One cluster becomes unreachable. What should happen?

### Answer

I would expect the applications targeting that cluster to report connection or synchronization problems, while applications in the other reachable clusters continue to reconcile.

I would investigate:

```bash
argocd cluster list
argocd app list
argocd app get <application>
```

Then verify:

- Cluster API reachability
- Cluster credentials
- EKS authentication
- IAM permissions
- Network connectivity
- Kubernetes API health

I would not immediately treat the entire Argo CD installation as failed.

---

# 111. Interview Scenario: Terraform and Argo CD Keep Reverting Each Other

### Question

Terraform changes a Kubernetes resource and Argo CD changes it back. What is wrong?

### Answer

There are two competing sources of truth.

For example:

```text
Terraform -> replicas = 3
Argo CD  -> replicas = 5
```

Each system is reconciling toward a different desired state.

I would identify ownership and establish a clear boundary. For example, Terraform can own AWS infrastructure and Argo CD can own Kubernetes application resources. If Terraform must manage a Kubernetes resource, Argo CD should not also manage the same fields without a deliberate design.

---

# 112. Interview Scenario: Argo CD Is Down

### Answer

I would distinguish control-plane availability from application runtime availability.

Existing workloads in EKS should normally continue running.

The immediate impact is:

```text
New Git synchronization unavailable
Drift correction delayed
New deployments delayed
```

I would then investigate Argo CD components, Kubernetes health, repository connectivity, and the management cluster.

This demonstrates that Argo CD is not normally on the application's request path.

---

# 113. Interview Scenario: Git Is Correct but Cluster Is Different

### Answer

I would identify whether the difference is:

1. Genuine drift
2. Expected controller-managed state
3. Ignore-difference case
4. Resource ownership conflict
5. Argo CD comparison issue

I would inspect:

```bash
argocd app get <application>
argocd app diff <application>
kubectl get <resource> -o yaml
```

Then determine which controller owns the differing field before changing anything.

---

# 114. Interview Scenario: Why Not Automatically Self-Heal Everything?

### Answer

Self-healing is valuable, but blindly correcting every difference can be dangerous. Some fields are legitimately changed by Kubernetes controllers, HPAs, operators, or cloud controllers. Production systems therefore need careful resource ownership, diff configuration, and synchronization policies. Automation should correct unexpected drift without fighting legitimate controllers.

---

# 115. Key Architectural Mental Model

Remember this:

```text
                    SOURCE
                      |
                      v
                  Git / PR
                      |
                      v
                    CI
                      |
                      v
                    ECR
                      |
                      v
                 DESIRED STATE
                      |
                      v
                  Argo CD
                      |
                +-----+-----+
                |           |
             Compare      Sync
                |           |
                +-----+-----+
                      |
                      v
               Kubernetes API
                      |
                      v
                  EKS Runtime
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Pods       Services      Ingress
                                  |
                                  v
                                 ALB
```

The control-plane relationship is:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes
 |
 v
Runtime
```

The artifact relationship is:

```text
CI
 |
 v
ECR
```

These are different concerns.

---

# 116. Final Summary

The most important architectural principles are:

1. GitOps is based on declarative desired state.
2. Git provides the versioned desired-state source of truth.
3. A controller continuously reconciles desired and actual state.
4. Argo CD is the reconciliation layer for Kubernetes GitOps.
5. CI should focus on build, test, security, and artifact creation.
6. ECR stores immutable container artifacts.
7. The GitOps repository declares how and where those artifacts should run.
8. Kubernetes remains responsible for runtime orchestration.
9. AWS controllers can perform additional reconciliation, such as ALB provisioning.
10. Resource ownership must be clearly defined.
11. Multiple systems should not fight over the same resource fields.
12. Pull-based GitOps can reduce direct production credentials in CI.
13. Trust boundaries must be explicitly designed.
14. Least privilege should apply to Git, CI, Argo CD, Kubernetes, and AWS.
15. Repository architecture should match team ownership and scale.
16. Environments can be isolated by namespace, cluster, account, or a combination.
17. Centralized Argo CD can manage multiple EKS clusters.
18. Centralized control planes require HA, monitoring, and DR planning.
19. Multi-cluster management requires cluster registration, metadata, RBAC, and destination restrictions.
20. AppProjects provide important Argo CD governance boundaries.
21. ApplicationSets provide scalable application generation.
22. Failure domains and blast radius should drive architecture decisions.
23. Existing applications should normally survive temporary Argo CD outages.
24. GitOps rollback changes desired state, but does not automatically reverse database/data changes.
25. Production GitOps is an operating model, not simply a collection of YAML files.

---