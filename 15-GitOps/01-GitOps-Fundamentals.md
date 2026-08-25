# GitOps-Fundamentals

## 1. Purpose of This File

This file establishes the foundation for the entire **GitOps with Argo CD** section.

The goal is not simply to define GitOps. The goal is to understand the operating model deeply enough to design, implement, troubleshoot, secure, and explain a production GitOps platform running on Kubernetes and AWS EKS.

The production environment used throughout these notes will be based on:

```text
AWS
├── EKS
├── ECR
├── ALB
├── IAM
└── supporting AWS services

Kubernetes
├── Deployments
├── Services
├── Ingress
├── ConfigMaps
├── Secrets
├── HPA
├── Namespaces
└── Helm

CI/CD
├── Jenkins
└── GitHub Actions

DevSecOps
├── SonarQube
├── Trivy
└── Veracode

GitOps
└── Argo CD

Observability
├── Prometheus
├── Grafana
└── ELK
```

The practical application used for examples will be the RoboShop microservices platform.

---

# 2. What Is GitOps?

GitOps is an operational model in which the desired state of an application or infrastructure environment is represented declaratively in a version-controlled Git repository, and an automated controller continuously reconciles the runtime environment toward that desired state.

For Kubernetes, a simplified architecture is:

```text
                  Git
                   |
                   | Desired State
                   v
              +---------+
              | Argo CD |
              +---------+
                   |
                   | Reconciliation
                   v
            Kubernetes API
                   |
                   v
                EKS
                   |
                   v
             Application
```

The important part is not merely that YAML files are stored in Git.

The important part is the **continuous reconciliation model**.

Git describes:

```text
"What should exist?"
```

Kubernetes contains:

```text
"What currently exists?"
```

The GitOps controller continuously works to make:

```text
Actual State -> Desired State
```

---

# 3. The Simplest Mental Model

A useful mental model is:

```text
Git = Desired State

Kubernetes = Actual State

Argo CD = Reconciliation Engine
```

Suppose Git declares:

```yaml
spec:
  replicas: 3
```

but Kubernetes currently has:

```text
replicas = 1
```

There is a mismatch.

Argo CD detects that mismatch and, depending on configuration, can reconcile the cluster toward:

```text
replicas = 3
```

Therefore:

```text
Desired = 3
Actual  = 1
       |
       v
    Drift
       |
       v
 Reconciliation
       |
       v
Desired = 3
Actual  = 3
```

This model is the foundation for understanding Argo CD.

---

# 4. Why GitOps Was Needed

Before GitOps became widely adopted, Kubernetes deployments were commonly implemented through approaches such as:

```text
Developer
   |
   v
CI/CD Server
   |
   +---- kubectl
   |
   +---- helm
   |
   v
Kubernetes
```

This can work.

The problem is that the deployment system can become the source of operational truth instead of the configuration repository.

For example:

```text
Jenkins job #827
Jenkins job #828
Manual kubectl command
Manual Helm upgrade
Emergency production change
```

The resulting cluster state may no longer be easy to reproduce.

GitOps attempts to make the desired state explicit, reviewable, versioned, and continuously reconciled.

---

# 5. Problems GitOps Solves

GitOps addresses several common operational problems.

## 5.1 Manual deployments

Without GitOps:

```bash
kubectl apply -f deployment.yaml
```

may be executed manually.

With GitOps:

```text
Git change
   |
   v
Argo CD
   |
   v
Kubernetes
```

The deployment becomes a controlled state transition.

---

## 5.2 Configuration drift

A production cluster may be manually modified.

Example:

```bash
kubectl scale deployment cart --replicas=1 -n roboshop
```

while Git declares:

```yaml
replicas: 3
```

The cluster has drifted.

GitOps provides a mechanism to detect and potentially correct that drift.

---

## 5.3 Poor auditability

A command such as:

```bash
kubectl edit deployment cart
```

does not automatically provide a good organizational change record.

Git provides:

```text
Author
Commit
Timestamp
Diff
Pull request
Review
Approval
Revert
```

This is much easier to audit.

---

## 5.4 Difficult rollback

If a deployment was performed through a sequence of manual commands, rollback can require remembering the previous configuration.

Git provides version history.

For example:

```text
Commit A -> cart 1.7.0
Commit B -> cart 1.8.0
Commit C -> cart 1.8.1
```

If C causes a problem, the desired state can be reverted to a known-good revision.

---

## 5.5 Excessive production credentials in CI

A traditional deployment pipeline may require:

```text
Jenkins
  |
  +-- Kubernetes credentials
  |
  +-- Production cluster access
  |
  +-- kubectl/Helm
```

GitOps can instead use:

```text
Jenkins
  |
  +-- Build
  +-- Test
  +-- Scan
  +-- Push image
  +-- Update Git
  |
  v
GitOps Repository

Argo CD
   |
   v
EKS
```

The CI system does not have to directly modify production Kubernetes resources.

---

# 6. GitOps Is More Than "YAML in Git"

A common beginner definition is:

> GitOps means storing Kubernetes YAML files in Git.

That is incomplete.

YAML in Git alone is not GitOps.

For example:

```text
Git repository
     |
     v
Engineer manually runs:
kubectl apply
```

This gives version-controlled configuration, but it does not provide continuous reconciliation.

A stronger GitOps model includes:

1. Declarative desired state
2. Version-controlled configuration
3. Automated delivery
4. Continuous reconciliation
5. Drift detection
6. Controlled recovery
7. Auditable changes

---

# 7. GitOps and Declarative Configuration

GitOps depends heavily on declarative management.

Declarative configuration describes the desired end state.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 3
```

This says:

```text
The cart Deployment should have 3 replicas.
```

It does not explicitly say:

```text
Create replica #1.
Create replica #2.
Create replica #3.
```

Kubernetes controllers determine how to achieve the desired state.

This is an important connection:

```text
GitOps
   |
   v
Declarative desired state
   |
   v
Kubernetes controllers
   |
   v
Actual runtime state
```

---

# 8. Imperative vs Declarative

## Imperative

An imperative command tells the system what action to perform.

Example:

```bash
kubectl scale deployment cart \
  --replicas=5 \
  -n roboshop
```

The command is an action.

## Declarative

A manifest describes the intended result:

```yaml
spec:
  replicas: 5
```

The controller determines the necessary actions.

### Why declarative is valuable

Declarative configuration can be:

- Reviewed
- Versioned
- Compared
- Reproduced
- Promoted
- Reverted
- Automatically reconciled

That makes it particularly suitable for GitOps.

---

# 9. Desired State

Desired state is the state that the organization wants the environment to have.

For a RoboShop Cart service, desired state could include:

```text
Namespace:
roboshop

Deployment:
cart

Replicas:
3

Container image:
cart:2026.08.19-abc123

CPU request:
100m

Memory request:
128Mi

CPU limit:
500m

Memory limit:
512Mi

Readiness probe:
enabled

Liveness probe:
enabled

Service:
ClusterIP

Ingress:
managed through AWS ALB
```

These requirements can be represented through Kubernetes manifests and Helm values.

---

# 10. Actual State

Actual state is what exists in the target Kubernetes cluster.

For example:

```text
Deployment/cart
replicas = 3

Pods:
cart-abc123
cart-def456
cart-ghi789

Image:
cart:2026.08.19-abc123
```

If all values match Git, the application is converged.

If the cluster has:

```text
replicas = 2
```

or:

```text
image = cart:2026.08.18-old
```

there is drift.

---

# 11. Configuration Drift

Configuration drift means the actual runtime state differs from the approved desired state.

Example:

```text
Git:
replicas = 3

EKS:
replicas = 1
```

Another example:

```text
Git:
image = cart:1.8.0

EKS:
image = cart:1.7.0
```

Another example:

```text
Git:
service type = ClusterIP

EKS:
service type = LoadBalancer
```

These are all potential forms of drift.

---

# 12. Sources of Drift

Drift can occur because of:

- Manual kubectl commands
- `kubectl edit`
- Manual Helm commands
- Emergency changes
- Incorrect automation
- Kubernetes controllers
- Admission controllers
- Operators
- External automation
- Cloud integrations
- Configuration mistakes

Not every difference is necessarily a defect.

For example, Kubernetes may populate or update certain fields dynamically.

Therefore, GitOps systems need mechanisms to understand which differences are meaningful and which are expected.

Argo CD provides features such as:

- Resource tracking
- Diffing
- Ignore differences
- Sync options
- Health assessment

These will be covered later.

---

# 13. Reconciliation

Reconciliation is the process of comparing desired state with actual state and taking corrective action when necessary.

Conceptually:

```text
             Git
              |
              v
       Desired State
              |
              v
       +--------------+
       | Reconciler   |
       +--------------+
              |
              v
       Actual State
              |
              v
         Difference?
          /       \
        No         Yes
        |           |
        v           v
     Healthy    Reconcile
                    |
                    v
              Kubernetes API
```

This is the most important operational idea behind GitOps.

---

# 14. Continuous Reconciliation

Traditional deployment may behave like:

```text
Change
  |
  v
Deploy
  |
  v
Done
```

GitOps behaves more like:

```text
Observe
   |
Compare
   |
Reconcile
   |
Observe again
   |
Compare
   |
Reconcile
   |
Repeat
```

This means the controller remains involved after the original deployment.

That is why GitOps is not simply another deployment script.

---

# 15. Kubernetes Controllers and GitOps

Kubernetes itself is built around controllers.

A simplified Kubernetes controller model is:

```text
Desired state
      |
      v
Controller
      |
      v
Actual state
      |
      |
      +---- compare
      |
      v
Correct state
```

For example, a Deployment controller ensures that the desired number of Pods exists.

GitOps introduces another control loop at a higher level:

```text
Git desired state
        |
        v
Argo CD
        |
        v
Kubernetes API
        |
        v
Kubernetes controllers
        |
        v
Pods / Services / Ingress
```

Therefore, GitOps and Kubernetes controllers complement each other.

---

# 16. GitOps Control Plane

A GitOps platform can be thought of as a control plane for desired application state.

For Argo CD:

```text
Git
 |
 v
Argo CD
 |
 +--> Compare
 |
 +--> Render
 |
 +--> Sync
 |
 +--> Observe
 |
 +--> Health assessment
 |
 +--> Reconcile
 |
 v
Kubernetes
```

Argo CD does not replace Kubernetes.

Instead, it uses the Kubernetes API to manage desired application state.

---

# 17. Push-Based Deployment

A push-based architecture commonly looks like:

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins
   |
   +---- build
   +---- test
   +---- scan
   +---- image
   |
   v
ECR
   |
   v
kubectl / Helm
   |
   v
EKS
```

Jenkins actively connects to the cluster and performs the deployment.

This is a valid deployment pattern, but it creates a direct relationship:

```text
CI -> Production Cluster
```

---

# 18. Pull-Based GitOps

A pull-based architecture looks like:

```text
Application Git
      |
      v
     CI
      |
      v
     ECR
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

Argo CD operates as the deployment controller.

The important security boundary becomes:

```text
CI
 |
 | write access
 v
GitOps repository

Argo CD
 |
 | cluster access
 v
EKS
```

This can reduce the blast radius of CI credentials.

---

# 19. Why Pull-Based GitOps Is Attractive

Advantages include:

### Reduced CI cluster credentials

CI does not necessarily require direct Kubernetes credentials.

### Continuous reconciliation

Argo CD remains active after deployment.

### Drift correction

Manual changes can be detected.

### Clear separation of responsibilities

```text
CI = artifact production

Argo CD = deployment and reconciliation
```

### Better Kubernetes locality

Argo CD can operate close to the cluster it manages.

---

# 20. GitOps Security Boundary

A production GitOps design should establish clear trust boundaries.

Example:

```text
             Developer
                 |
                 v
        Application Repository
                 |
                 v
               CI
                 |
          +------+------+
          |             |
          v             v
       Security        ECR
        Gates           |
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
                       EKS
```

Each component should have only the permissions it needs.

For example:

```text
CI:
- read application source
- push image to ECR
- update approved GitOps configuration

Argo CD:
- read approved repositories
- access authorized clusters
- manage authorized namespaces/resources
```

This principle becomes critical in multi-cluster environments.

---

# 21. Git as the Source of Truth

In GitOps, the Git repository is the authoritative representation of the desired state for resources managed through GitOps.

Example:

```text
gitops-repo/
└── environments/
    └── prod/
        └── cart-values.yaml
```

Suppose it declares:

```yaml
replicaCount: 3
image:
  tag: "2026.08.19-abc123"
```

That configuration is the desired state.

If someone manually changes the cluster:

```bash
kubectl scale deployment cart --replicas=1
```

the manual change should not silently become the new source of truth.

The Git repository remains authoritative unless the organization intentionally changes Git.

---

# 22. GitOps and Pull Requests

A production change can follow this model:

```text
Developer / Release Engineer
          |
          v
Create Pull Request
          |
          v
Review
          |
          v
Approval
          |
          v
Merge
          |
          v
Argo CD detects revision
          |
          v
Sync
```

This gives production changes a review boundary.

For example:

```text
PR:
Update cart from 1.8.0 to 1.9.0

Reviewers:
DevOps
Application Owner
Production Approver
```

After merge:

```text
Argo CD -> EKS
```

---

# 23. GitOps and Auditability

A production GitOps change should answer:

- Who changed it?
- What changed?
- Why did it change?
- Who reviewed it?
- Which commit introduced it?
- Which environment received it?
- When was it synchronized?
- What was the previous revision?

Git provides much of the change history.

Argo CD provides deployment/application state and history.

Together:

```text
Git audit trail
      +
Argo CD deployment state
      +
Kubernetes runtime state
```

provide a strong operational trail.

---

# 24. GitOps and Rollbacks

Suppose production is running:

```text
cart:1.7.0
```

A Git change updates it to:

```text
cart:1.8.0
```

Production becomes unhealthy.

A rollback can be performed by restoring the previous desired state.

For example:

```text
Commit A
cart:1.7.0
        |
        v
Commit B
cart:1.8.0
        |
        v
Problem detected
        |
        v
Revert Commit B
        |
        v
Git = cart:1.7.0
        |
        v
Argo CD reconciles
        |
        v
EKS = cart:1.7.0
```

This is one reason GitOps makes deployment state reproducible.

---

# 25. Rollback Is Not Always Data Recovery

A critical production distinction:

Rolling back application configuration does not automatically roll back data.

Example:

```text
Application 1.8.0
     |
     v
Database migration
     |
     v
Schema changed
     |
     v
Application failure
```

Reverting the application image may not undo the database schema migration.

Therefore:

```text
GitOps rollback
    !=
Complete system rollback
```

Production rollback plans must consider:

- Application code
- Kubernetes configuration
- Database schema
- Persistent volumes
- External services
- Queues
- Data compatibility

---

# 26. GitOps and Immutable Artifacts

A production GitOps system should use identifiable artifact versions.

Avoid:

```yaml
image: cart:latest
```

Prefer:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart
  tag: "2026.08.19-abc123"
```

Even stronger reproducibility can be achieved by pinning a digest:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart
  digest: "sha256:..."
```

The exact image-promotion strategy depends on the organization's release process.

---

# 27. GitOps Image Promotion

A production promotion model can look like:

```text
Build
  |
  v
cart:2026.08.19-abc123
  |
  v
ECR
  |
  v
DEV
  |
  | validation
  v
QA
  |
  | approval
  v
PROD
```

The same immutable image should ideally move through environments rather than rebuilding the application separately for each environment.

This improves confidence that the artifact tested in QA is the artifact promoted to production.

---

# 28. CI Responsibilities

CI should generally handle artifact creation and validation.

For the user's environment:

```text
Developer
    |
    v
GitHub
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

The output is a trusted container artifact.

---

# 29. GitOps CD Responsibilities

Argo CD handles the deployment side:

```text
GitOps Repository
       |
       v
     Argo CD
       |
       +--> Render desired manifests
       +--> Compare state
       +--> Sync
       +--> Track resources
       +--> Evaluate health
       +--> Detect drift
       +--> Reconcile
       |
       v
      EKS
```

Therefore:

```text
CI creates the artifact.

GitOps defines where and how the artifact should run.

Argo CD reconciles that desired state.
```

---

# 30. Complete RoboShop GitOps Flow

The production-oriented flow for the RoboShop platform is:

```text
                     Developer
                         |
                         v
                   Application Git
                         |
                         v
              Jenkins / GitHub Actions
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       Build          Tests        Security
                                    |
                         +----------+----------+
                         |          |          |
                         v          v          v
                     SonarQube    Trivy    Veracode
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
                         | desired image/version
                         v
                      Argo CD
                         |
                         v
                 Kubernetes API
                         |
                         v
                      Amazon EKS
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       Services       Deployments    Configurations
          |
          v
      ALB Ingress
          |
          v
       End Users
```

This is the primary GitOps architecture used throughout the notes.

---

# 31. Application Repository vs GitOps Repository

A practical organization can have two repositories.

## Application repository

```text
roboshop-cart/
├── src/
├── Dockerfile
├── pom.xml
├── tests/
└── README.md
```

Responsibilities:

- Application source
- Build configuration
- Tests
- Dockerfile
- Application documentation

## GitOps repository

```text
roboshop-gitops/
├── applications/
├── applicationsets/
├── environments/
├── helm/
├── clusters/
├── projects/
└── platform/
```

Responsibilities:

- Kubernetes desired state
- Helm values
- Kustomize overlays
- Argo CD Applications
- ApplicationSets
- Projects
- Environment configuration

This separation is not mandatory, but it is common and useful at scale.

---

# 32. GitOps Repository Design Principles

A GitOps repository should be designed for:

### Discoverability

Engineers should easily locate:

```text
Where is production?
Where is QA?
Where is this application's Helm values?
Where is the Argo CD Application?
```

### Ownership

Teams should understand who owns:

```text
Application
Environment
Cluster
Platform
Security configuration
```

### Controlled access

Production configuration should have stronger access controls than development configuration where appropriate.

### Reproducibility

The repository should contain enough declarative information to recreate managed resources.

---

# 33. Example Production GitOps Structure

A scalable structure can look like:

```text
roboshop-gitops/
│
├── applications/
│   ├── cart.yaml
│   ├── catalog.yaml
│   ├── orders.yaml
│   ├── payment.yaml
│   └── user.yaml
│
├── applicationsets/
│   ├── roboshop-environments.yaml
│   └── roboshop-clusters.yaml
│
├── projects/
│   └── roboshop.yaml
│
├── environments/
│   ├── dev/
│   │   ├── values/
│   │   └── kustomization.yaml
│   │
│   ├── qa/
│   │   ├── values/
│   │   └── kustomization.yaml
│   │
│   └── prod/
│       ├── values/
│       └── kustomization.yaml
│
├── helm/
│   └── roboshop/
│       ├── Chart.yaml
│       ├── templates/
│       └── values.yaml
│
├── clusters/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── platform/
    ├── namespaces/
    ├── ingress/
    └── monitoring/
```

The exact organization will be refined later when Helm, ApplicationSets, multi-cluster management, and enterprise strategies are covered.

---

# 34. GitOps Environment Strategy

A production organization may have:

```text
DEV
 |
 v
QA
 |
 v
PROD
```

Each environment can have:

- Separate namespace
- Separate cluster
- Separate AWS account
- Separate Git path
- Separate values
- Separate Argo CD Application
- Separate access controls

There is no universal requirement that every environment use a separate cluster.

Possible designs include:

### Namespace isolation

```text
One EKS cluster
├── dev namespace
├── qa namespace
└── prod namespace
```

### Cluster isolation

```text
EKS-DEV
EKS-QA
EKS-PROD
```

### Account and cluster isolation

```text
AWS Dev Account
  -> EKS Dev

AWS QA Account
  -> EKS QA

AWS Production Account
  -> EKS Production
```

The last pattern provides stronger isolation and is common for sensitive production environments.

---

# 35. GitOps and Infrastructure as Code

GitOps and Terraform solve related but different problems.

Terraform can provision:

```text
AWS
├── VPC
├── Subnets
├── Security Groups
├── EKS
├── IAM
├── ECR
├── RDS
├── S3
└── ALB-related infrastructure
```

Argo CD can manage:

```text
Kubernetes
├── Namespaces
├── Deployments
├── Services
├── Ingress
├── ConfigMaps
├── HPA
├── Helm releases
├── Application resources
└── Platform applications
```

A practical architecture is:

```text
Terraform
    |
    v
AWS infrastructure
    |
    v
EKS foundation
    |
    v
Argo CD
    |
    v
Kubernetes workloads
```

This separation should be explicit in a production operating model.

---

# 36. What GitOps Should Manage

Good GitOps candidates include resources whose desired configuration should be declarative and version-controlled.

Examples:

```text
Namespaces
Deployments
Services
Ingress
ConfigMaps
HPA
NetworkPolicies
RBAC
Helm releases
Argo CD Applications
ApplicationSets
Projects
Platform configurations
```

---

# 37. What Requires Special Consideration

Not every resource should blindly be placed under GitOps.

Examples requiring careful design:

- Highly dynamic resources
- Runtime-generated configuration
- Secrets
- Database data
- Ephemeral resources
- Resources controlled by other operators
- External systems with their own state
- Resources whose lifecycle is better handled by Terraform

The goal is not:

> Put everything into Git.

The goal is:

> Establish a clear source of truth and ownership for every important piece of desired state.

---

# 38. Secrets and GitOps

Never assume that a private Git repository automatically makes plaintext secrets safe.

Avoid:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database
stringData:
  password: "real-production-password"
```

Possible production approaches include:

```text
GitOps
   |
   +--> External Secrets Operator
   |         |
   |         v
   |    AWS Secrets Manager
   |
   +--> Sealed Secrets
   |
   +--> SOPS
   |
   +--> Secret Store CSI Driver
```

Secrets management is intentionally covered in a separate file:

```text
15-GitOps-Secrets-Management.md
```

---

# 39. GitOps and Kubernetes Namespaces

Namespaces are useful for logical isolation.

Example:

```text
EKS
 |
 +-- roboshop-dev
 |
 +-- roboshop-qa
 |
 +-- roboshop-prod
```

Argo CD can restrict Applications to specific namespaces.

This becomes especially important when multiple teams share a cluster.

---

# 40. GitOps and Kubernetes RBAC

GitOps does not remove Kubernetes security requirements.

Argo CD needs permissions to manage resources.

Those permissions should follow least privilege.

Conceptually:

```text
Argo CD
   |
   +--> allowed cluster
   |
   +--> allowed namespace
   |
   +--> allowed resource types
```

In enterprise environments, permissions should be carefully constrained.

This will be covered in:

```text
14-GitOps-Security-and-RBAC.md
```

---

# 41. GitOps Production Security Model

A strong production model may look like:

```text
Developer
   |
   | PR
   v
Application Git
   |
   v
CI
   |
   +--> Security gates
   |
   v
ECR
   |
   v
GitOps PR
   |
   | approval
   v
GitOps main branch
   |
   v
Argo CD
   |
   +--> AppProject restrictions
   +--> RBAC
   +--> Repository allowlist
   +--> Cluster allowlist
   |
   v
EKS
```

The important security principle is that no single component should automatically have unrestricted access to every system.

---

# 42. GitOps Failure Scenarios

GitOps introduces a new control plane, so its own failure modes must be understood.

## Git repository unavailable

Argo CD may be unable to retrieve new desired state.

Existing deployed workloads may continue running.

## Argo CD unavailable

Applications may continue running in Kubernetes, but new reconciliation and synchronization may be delayed.

## Kubernetes API unavailable

Argo CD cannot apply or inspect resources.

## ECR unavailable

New Pods may fail to pull images.

## Invalid Git configuration

Argo CD may fail to render manifests.

## Permission denied

Argo CD may detect desired changes but fail to apply them.

These scenarios will be examined deeply in the troubleshooting file.

---

# 43. GitOps Availability Principle

A useful production principle is:

> The GitOps control plane should not become a single point of failure for running applications.

For example:

```text
Argo CD temporarily unavailable
          |
          v
Existing Pods
          |
          v
Continue serving traffic
```

Kubernetes does not require Argo CD to be continuously present for already-running Pods to continue operating.

However:

```text
Argo CD unavailable
          |
          +--> No new Git reconciliation
          +--> No new automated sync
          +--> Drift may remain
          +--> New desired state may be delayed
```

Therefore Argo CD availability is operationally important even though it is not normally on the request path of application traffic.

---

# 44. GitOps and Disaster Recovery

GitOps can simplify recovery of declarative application configuration.

Example:

```text
Production EKS lost
       |
       v
Terraform
       |
       v
Recreate infrastructure
       |
       v
EKS
       |
       v
Bootstrap Argo CD
       |
       v
Connect GitOps repository
       |
       v
Applications recreated
```

But full disaster recovery also requires:

- Database backups
- Persistent volume recovery
- Secrets recovery
- DNS recovery
- Certificates
- AWS infrastructure recovery
- External dependency recovery
- Data consistency planning

GitOps is an important component of DR, not the complete DR system.

---

# 45. GitOps and Business Continuity

A mature GitOps strategy should answer:

```text
If the production cluster disappears:

1. Where is the infrastructure definition?
2. Where is the Kubernetes desired state?
3. Where are secrets?
4. Where is application data?
5. How is Argo CD bootstrapped?
6. How are target clusters registered?
7. How are DNS and certificates restored?
8. How long does recovery take?
9. Who performs the recovery?
10. How is recovery validated?
```

These questions should be tested through disaster-recovery exercises.

---

# 46. GitOps Anti-Patterns

## Anti-pattern 1: Git contains YAML but deployment is manual

```text
Git
 |
 v
Manual kubectl
 |
 v
EKS
```

This is version-controlled configuration, but not a complete GitOps operating model.

---

## Anti-pattern 2: CI still owns all production deployment

```text
Jenkins
 |
 +--> Git
 +--> Kubernetes
 +--> production credentials
```

This can undermine the separation GitOps is intended to provide.

---

## Anti-pattern 3: Mutable image tags

```text
image: latest
```

This reduces reproducibility.

---

## Anti-pattern 4: Plaintext secrets

```text
password: production-password
```

This creates a security risk.

---

## Anti-pattern 5: No review for production

```text
git push
   |
   v
PROD
```

Automated deployment does not mean uncontrolled deployment.

---

## Anti-pattern 6: Excessive manual production changes

If engineers constantly modify production directly, Git stops representing reality.

---

## Anti-pattern 7: Unrestricted Argo CD permissions

Giving Argo CD unrestricted access to every cluster and namespace increases blast radius.

---

## Anti-pattern 8: Blind automatic pruning

Prune can remove resources that are no longer declared.

It must be designed and tested carefully.

---

# 47. GitOps Production Best Practices

## Repository

- Use protected branches.
- Require pull-request review.
- Use CODEOWNERS where appropriate.
- Keep production changes auditable.
- Use clear directory conventions.
- Avoid unnecessary duplication.
- Document ownership.

## Images

- Use immutable versions.
- Prefer digest pinning for high-assurance workloads.
- Scan images before promotion.
- Promote the same tested artifact.

## Argo CD

- Use AppProjects.
- Configure RBAC.
- Restrict repositories.
- Restrict destinations.
- Use least privilege.
- Monitor synchronization failures.
- Use HA for critical installations.
- Back up required configuration.

## Kubernetes

- Use resource requests and limits.
- Use readiness/liveness/startup probes appropriately.
- Use security contexts.
- Use NetworkPolicies where appropriate.
- Use namespace boundaries.
- Avoid excessive cluster-admin access.

## Production synchronization

- Decide which applications can auto-sync.
- Require approval where business risk demands it.
- Understand prune behavior.
- Use sync waves for dependencies.
- Test rollback.

---

# 48. GitOps Operational Model

A mature team can define responsibilities like:

```text
Application Team
|
+-- application code
+-- tests
+-- application behavior
|
v

CI Platform
|
+-- build
+-- test
+-- security
+-- image creation
|
v

Artifact Registry
|
+-- ECR
|
v

Platform / Release Team
|
+-- GitOps repository
+-- Helm/Kustomize
+-- Argo CD
+-- cluster configuration
|
v

Kubernetes Platform
|
+-- EKS
+-- nodes
+-- networking
+-- runtime
```

Ownership should be explicit.

---

# 49. GitOps Maturity Levels

A useful way to assess adoption is:

## Level 0 - Manual

```text
kubectl from engineer laptop
```

## Level 1 - CI/CD Automation

```text
Git -> CI -> kubectl/Helm -> cluster
```

## Level 2 - Declarative Configuration

```text
Kubernetes configuration stored in Git
```

## Level 3 - GitOps Reconciliation

```text
Git -> Argo CD -> Kubernetes
```

with drift detection and synchronization.

## Level 4 - Enterprise GitOps

Includes:

```text
Multi-cluster
Multi-environment
RBAC
Security
ApplicationSets
Policy
Observability
Progressive delivery
DR
Automated promotion
```

The goal of this syllabus is to reach Level 4 understanding.

---

# 50. GitOps with AWS EKS

A production EKS implementation may look like:

```text
AWS Account
|
+------------------------------------------------+
|                                                |
|                  EKS                           |
|                                                |
|   +----------------------------------------+   |
|   |              Argo CD                   |   |
|   |                                        |   |
|   |   Git -> Reconcile -> Kubernetes API   |   |
|   +----------------------------------------+   |
|                    |                           |
|                    v                           |
|              Application Pods                 |
|                    |                           |
|                    v                           |
|              AWS ALB Ingress                  |
|                                                |
+------------------------------------------------+
```

Argo CD can also run in a dedicated management cluster and manage multiple EKS clusters.

That architecture will be covered in detail in:

```text
10-ArgoCD-Multi-Cluster-Management.md
11-ArgoCD-AWS-EKS-and-Multi-Account.md
```

---

# 51. Single-Cluster GitOps Architecture

A basic production architecture:

```text
                GitOps Repository
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
        +--------------+--------------+
        |              |              |
        v              v              v
      Cart          Catalog        Orders
        |              |              |
        +--------------+--------------+
                       |
                       v
                  ALB Ingress
```

Argo CD is responsible for desired-state reconciliation.

Kubernetes is responsible for running workloads.

ALB provides external HTTP/HTTPS traffic routing.

---

# 52. Multi-Cluster GitOps Architecture

A centralized Argo CD model can look like:

```text
                         Git Repository
                               |
                               v
                       +---------------+
                       |    Argo CD    |
                       | Management    |
                       |   Cluster     |
                       +---------------+
                         /      |      \
                        /       |       \
                       v        v        v
                    EKS-DEV   EKS-QA  EKS-PROD
```

The central Argo CD instance stores or uses credentials needed to access authorized target clusters.

Applications can specify destinations such as:

```text
EKS-DEV
EKS-QA
EKS-PROD
```

ApplicationSets can generate Applications dynamically for multiple clusters.

This is a major capability of Argo CD and will be covered later.

---

# 53. GitOps Multi-Account Concept

A larger AWS organization may use:

```text
AWS Organization
|
+-- Dev Account
|    |
|    +-- EKS-DEV
|
+-- QA Account
|    |
|    +-- EKS-QA
|
+-- Production Account
     |
     +-- EKS-PROD
```

A centralized GitOps control plane can potentially manage these clusters, provided cross-account access and security boundaries are designed correctly.

Important concerns include:

- AWS IAM
- EKS authentication
- Network connectivity
- Cluster credentials
- Argo CD RBAC
- Repository access
- Environment isolation
- Blast radius
- Compliance

---

# 54. GitOps and AWS IAM

GitOps does not eliminate AWS IAM.

A production EKS environment may involve:

```text
Developer
   |
   v
AWS IAM / Identity Provider
   |
   v
EKS access
```

Argo CD also needs a secure way to authenticate to target clusters.

Depending on architecture, cluster access can involve Kubernetes credentials and/or AWS-supported authentication mechanisms.

The exact mechanism will be covered later when EKS and multi-account management are discussed.

---

# 55. GitOps and Observability

GitOps deployments need observability.

A production stack may include:

```text
Prometheus
    |
    v
Metrics

Grafana
    |
    v
Dashboards

ELK
    |
    v
Logs
```

Argo CD should also be monitored.

Important signals include:

- Application sync status
- Application health
- Reconciliation errors
- Repository errors
- Controller errors
- API errors
- Failed hooks
- Sync duration
- Resource failures

Observability is covered in:

```text
18-GitOps-Observability.md
```

---

# 56. GitOps and Progressive Delivery

GitOps can control progressive delivery.

For example:

```text
Version 1
   |
   v
5% traffic
   |
   v
25%
   |
   v
50%
   |
   v
100%
```

Tools and patterns can support:

- Canary releases
- Blue/green deployment
- Automated analysis
- Progressive promotion

The GitOps repository can define the desired rollout state while specialized controllers manage the progressive behavior.

This is covered in:

```text
17-GitOps-Progressive-Delivery.md
```

---

# 57. GitOps and Platform Engineering

GitOps is often part of a larger platform engineering model.

A platform team may provide:

```text
Developer Platform
|
+-- Git templates
+-- CI templates
+-- Helm charts
+-- Argo CD
+-- ApplicationSets
+-- Kubernetes clusters
+-- Observability
+-- Security
```

Application teams consume standardized platform capabilities.

This reduces repeated deployment engineering across teams.

---

# 58. GitOps and Standardization

Without standardization:

```text
Team A -> Helm
Team B -> raw YAML
Team C -> custom scripts
Team D -> manual kubectl
```

This increases operational complexity.

A platform can define standards such as:

```text
Every service must have:
- Deployment
- Service
- Probes
- Resource requests/limits
- Security context
- Standard labels
- Standard logging
- Standard monitoring
- GitOps Application
```

This is especially useful for a microservices platform such as RoboShop.

---

# 59. GitOps and Policy

GitOps can be combined with policy controls.

Examples:

```text
Git
 |
 v
Pull Request
 |
 +--> CI validation
 +--> Security scan
 +--> Policy validation
 |
 v
Merge
 |
 v
Argo CD
```

Policies can check things such as:

- Required resource limits
- Allowed image registries
- Security context
- Privileged containers
- Required labels
- Namespace restrictions
- Approved deployment strategies

The exact policy tooling is outside this file, but the GitOps model should leave room for policy enforcement.

---

# 60. GitOps Change Lifecycle

A production change can be modeled as:

```text
1. Developer changes application
        |
        v
2. Application Git commit
        |
        v
3. CI pipeline
        |
        +--> Build
        +--> Test
        +--> SonarQube
        +--> Trivy
        +--> Veracode
        |
        v
4. Push immutable image to ECR
        |
        v
5. Update GitOps desired state
        |
        v
6. Pull request / approval
        |
        v
7. Merge
        |
        v
8. Argo CD detects revision
        |
        v
9. Argo CD renders desired manifests
        |
        v
10. Argo CD compares desired/actual
        |
        v
11. Sync
        |
        v
12. Kubernetes controllers reconcile
        |
        v
13. Pods become ready
        |
        v
14. Argo CD health becomes healthy
        |
        v
15. Monitoring validates runtime
```

This lifecycle will be repeatedly used throughout the syllabus.

---

# 61. What Happens During a GitOps Deployment?

Suppose a GitOps Helm value changes:

```yaml
image:
  tag: "2026.08.19-abc123"
```

The sequence is approximately:

```text
Git commit
   |
   v
Argo CD refresh
   |
   v
Read source configuration
   |
   v
Render Helm/Kustomize/manifests
   |
   v
Calculate desired resources
   |
   v
Compare with cluster
   |
   v
OutOfSync
   |
   v
Sync
   |
   v
Kubernetes API
   |
   v
Deployment updated
   |
   v
ReplicaSet created/updated
   |
   v
Pods created
   |
   v
Readiness checks
   |
   v
Application health
   |
   v
Synced + Healthy
```

The internal Argo CD implementation is more detailed and will be covered in the architecture files.

---

# 62. Synced Does Not Always Mean Healthy

This distinction is extremely important.

An application can have:

```text
Sync Status: Synced
Health: Degraded
```

Meaning:

```text
The cluster matches Git
```

but:

```text
The application is not healthy
```

For example:

```text
Deployment image is exactly what Git specifies
BUT
Pods are CrashLoopBackOff
```

Therefore:

```text
Synced != Healthy
```

This distinction will become critical when learning Argo CD troubleshooting.

---

# 63. GitOps Desired State Can Be Correct but Application Can Fail

GitOps ensures configuration reconciliation.

It does not guarantee application correctness.

Example:

```text
Git:
image = cart:2.0.0
```

Argo CD successfully deploys it.

But:

```text
cart:2.0.0
```

contains an application bug.

Result:

```text
Sync = successful
Health = degraded
```

Therefore application-level monitoring remains essential.

---

# 64. GitOps and Application Health

A production GitOps system should monitor both:

### Desired-state status

```text
Synced
OutOfSync
Unknown
```

### Runtime health

```text
Healthy
Progressing
Degraded
Missing
Unknown
```

Argo CD combines these concepts to provide a useful operational view.

Later files will explain exactly how these statuses are determined.

---

# 65. GitOps and Manual Emergency Changes

Production systems sometimes require emergency intervention.

Example:

```text
Major incident
     |
     v
Engineer changes replica count
     |
     v
Service stabilizes
```

The change should then be reconciled back into the GitOps process.

A good operational rule is:

```text
Emergency manual change
        |
        v
Stabilize production
        |
        v
Document change
        |
        v
Update Git desired state
        |
        v
Review / merge
        |
        v
Return to normal GitOps operation
```

Otherwise the emergency change becomes undocumented drift.

---

# 66. GitOps and Break-Glass Access

Production teams may maintain controlled emergency access.

The principle should be:

```text
Normal path:
Git -> Argo CD -> EKS

Emergency path:
Authorized engineer -> EKS
```

The emergency path should be:

- Restricted
- Audited
- Time-limited where possible
- Documented
- Followed by Git reconciliation

GitOps should not make legitimate emergency response impossible.

---

# 67. GitOps and Change Ownership

Every configuration area should have an owner.

Example:

```text
applications/
   |
   +-- Application Team

applicationsets/
   |
   +-- Platform Team

projects/
   |
   +-- Platform/Security

clusters/
   |
   +-- Platform Team

production environment values/
   |
   +-- Release / Application Owners
```

Ownership can be enforced with Git permissions and review rules.

---

# 68. GitOps and Repository Governance

A production GitOps repository should normally have:

```text
Protected main branch
Required pull requests
Required reviews
CI validation
Security scanning
Policy checks
CODEOWNERS
Audit logging
```

Potential workflow:

```text
Engineer
   |
   v
Feature branch
   |
   v
Pull Request
   |
   +--> YAML validation
   +--> Helm lint
   +--> Security checks
   +--> Policy checks
   |
   v
Approval
   |
   v
Merge
```

Argo CD then observes the merged state.

---

# 69. GitOps Validation Before Merge

Production GitOps should validate configuration before it reaches the main branch.

Examples:

```bash
kubectl apply --dry-run=client -f manifest.yaml
```

For Helm:

```bash
helm lint ./chart
helm template ./chart
```

For YAML:

```text
YAML syntax validation
```

For Kubernetes policies:

```text
Policy validation
```

For security:

```text
Image scanning
Manifest scanning
Secret scanning
```

The exact CI implementation will be covered in the CI/CD integration file.

---

# 70. GitOps and Helm

GitOps can manage Helm applications.

Instead of:

```bash
helm upgrade --install cart ...
```

manually, Git can declare:

```yaml
source:
  repoURL: ...
  path: ...
  helm:
    valueFiles:
      - values-prod.yaml
```

Argo CD renders the Helm chart and reconciles the resulting Kubernetes resources.

Important:

> Helm is the packaging/template mechanism; Argo CD is the GitOps reconciliation/deployment controller.

This distinction becomes important later.

---

# 71. GitOps and Kustomize

Kustomize can also represent environment differences.

Example:

```text
base/
├── deployment.yaml
└── service.yaml

overlays/
├── dev/
├── qa/
└── prod/
```

GitOps can point Argo CD at the appropriate overlay.

This gives:

```text
Common base
     |
     +--> dev
     +--> qa
     +--> prod
```

Helm and Kustomize will be covered in:

```text
08-ArgoCD-Helm-and-Kustomize.md
```

---

# 72. GitOps and ApplicationSets

When an organization has many environments or clusters, manually creating every Argo CD Application becomes repetitive.

Example:

```text
cart-dev
cart-qa
cart-prod

catalog-dev
catalog-qa
catalog-prod

orders-dev
orders-qa
orders-prod
```

ApplicationSets can generate Applications from templates.

Conceptually:

```text
ApplicationSet
      |
      +--> cart-dev
      +--> cart-qa
      +--> cart-prod
      +--> catalog-dev
      +--> catalog-qa
      +--> catalog-prod
```

This becomes especially powerful for multi-cluster EKS environments.

---

# 73. GitOps Multi-Cluster Model

One Argo CD installation can manage multiple Kubernetes clusters.

For example:

```text
                   Git
                    |
                    v
               Central Argo CD
               /      |      \
              /       |       \
             v        v        v
          EKS-DEV   EKS-QA   EKS-PROD
```

This is not simply a single-cluster deployment mechanism.

Argo CD can act as a centralized GitOps control plane.

Important concerns include:

- Cluster registration
- Cluster credentials
- RBAC
- Network access
- AWS account isolation
- Application destination restrictions
- Cluster labels
- ApplicationSet generators
- Blast radius

These will be covered deeply later.

---

# 74. GitOps and Multi-Account AWS

An enterprise may use:

```text
AWS Organization
|
+-- Dev Account
|    |
|    +-- EKS Dev
|
+-- QA Account
|    |
|    +-- EKS QA
|
+-- Prod Account
     |
     +-- EKS Prod
```

A centralized Argo CD installation may manage the target clusters.

This requires carefully designed access.

A central controller should not automatically receive unrestricted permissions across all AWS accounts.

---

# 75. GitOps and Environment Isolation

Environment isolation can be achieved through:

- Separate AWS accounts
- Separate EKS clusters
- Separate namespaces
- Separate Argo CD Projects
- Separate repositories
- Separate Git paths
- Separate access groups

A strong production architecture often combines several layers.

Example:

```text
AWS Account
   |
   v
EKS Cluster
   |
   v
Namespace
   |
   v
Argo CD Project
   |
   v
Application
```

Each layer provides a boundary.

---

# 76. GitOps and the Principle of Least Privilege

Least privilege means:

> Give each identity only the permissions required to perform its intended function.

For example:

```text
CI
- Push images to ECR
- Update GitOps repository through controlled process

Argo CD
- Read approved repositories
- Manage approved target clusters
- Manage approved namespaces/resources

Developer
- Modify application source
- Request deployment through Git
```

Avoid:

```text
Everyone -> cluster-admin
```

That defeats many of the security advantages of GitOps.

---

# 77. GitOps and Compliance

GitOps can help satisfy requirements for:

- Change approval
- Audit trail
- Separation of duties
- Reproducibility
- Controlled production access
- Versioned configuration

Example:

```text
Production change
      |
      v
Pull Request
      |
      +--> reviewer
      +--> approval
      +--> automated validation
      |
      v
Merge
      |
      v
Argo CD
      |
      v
Production
```

This is much easier to audit than undocumented manual changes.

---

# 78. GitOps and Separation of Duties

A production organization can separate:

```text
Developer
   |
   +--> Application code

Release Engineer
   |
   +--> GitOps configuration

Security
   |
   +--> Security approval

Platform Team
   |
   +--> Argo CD / cluster platform
```

The exact ownership model depends on organization size.

The important principle is to avoid a single identity having unrestricted control over the entire software delivery chain.

---

# 79. GitOps Repository Availability

Git is now operationally important.

If the GitOps repository becomes unavailable:

```text
New desired state
       |
       X
    unavailable
```

Existing workloads may continue running.

But:

```text
New deployments
Drift reconciliation based on new commits
Environment promotion
```

may be delayed.

Therefore production GitOps requires:

- Reliable Git hosting
- Backups
- Access recovery
- Disaster recovery procedures
- Repository protection

---

# 80. GitOps Controller Availability

Similarly:

```text
Argo CD unavailable
       |
       v
Existing workloads continue
       |
       +--> New sync unavailable
       +--> Drift correction delayed
       +--> Application updates delayed
```

This is why production Argo CD installations may use:

- High availability
- Multiple replicas
- Dedicated infrastructure
- Monitoring
- Backup/recovery procedures

---

# 81. GitOps Network Considerations

The architecture should answer:

```text
Can Argo CD reach Git?
Can Argo CD reach Kubernetes APIs?
Can EKS reach required registries?
Can workloads reach required dependencies?
```

For private EKS environments:

```text
Git
 |
 | network access
 v
Argo CD
 |
 | private Kubernetes API
 v
EKS
```

Network controls can include:

- VPC
- Security groups
- Network ACLs
- Private endpoints
- Proxy configuration
- DNS
- Egress controls

---

# 82. GitOps and AWS ALB

For the user's architecture, external application traffic is handled through AWS ALB Ingress.

Example:

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
Service
   |
   v
Pods
```

GitOps can manage the Kubernetes Ingress configuration.

The AWS Load Balancer Controller then reconciles Kubernetes Ingress resources into AWS ALB resources.

This creates another controller relationship:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes Ingress
 |
 v
AWS Load Balancer Controller
 |
 v
AWS ALB
```

This is an important real-world example of layered reconciliation.

---

# 83. Layered Reconciliation in EKS

A production EKS environment can contain multiple control loops.

Example:

```text
Git desired state
       |
       v
Argo CD
       |
       v
Kubernetes API
       |
       v
Kubernetes controllers
       |
       +--> Deployment -> Pods
       |
       +--> Service
       |
       +--> HPA
       |
       v
AWS Load Balancer Controller
       |
       v
AWS ALB
```

Understanding these layers is important when troubleshooting.

A resource may be:

```text
Synced by Argo CD
```

but still fail because another controller or the application itself has a problem.

---

# 84. GitOps and HPA

Suppose Git defines:

```yaml
replicas: 3
```

but an HPA changes runtime replicas based on CPU:

```text
minReplicas: 3
maxReplicas: 10
```

The actual number of replicas may legitimately change.

This is a classic case where blindly treating every difference as drift can be problematic.

GitOps systems need to understand resources controlled dynamically by other Kubernetes controllers.

This is one reason advanced Argo CD diff and ignore-difference mechanisms matter.

---

# 85. GitOps and Controllers

The general relationship is:

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
Specialized controller
 |
 v
Runtime resource
```

Examples:

```text
Ingress -> AWS Load Balancer Controller -> ALB

Deployment -> Kubernetes Deployment Controller -> ReplicaSets/Pods

HPA -> HPA Controller -> Replica count

Service -> Kubernetes networking implementation
```

Therefore, troubleshooting must identify **which controller owns which part of the state**.

---

# 86. GitOps Source of Truth vs Runtime Truth

There are two useful concepts:

### Desired-state truth

```text
Git
```

### Runtime truth

```text
Kubernetes API
```

GitOps compares them.

However, Kubernetes itself may have controllers that legitimately transform desired resource specifications into runtime state.

Therefore:

```text
Git
  |
  v
Argo CD desired resource
  |
  v
Kubernetes desired resource
  |
  v
Controllers
  |
  v
Runtime
```

This layered model prevents oversimplified troubleshooting.

---

# 87. GitOps and Resource Ownership

A production GitOps platform should know which resources it owns.

For example:

```text
Argo CD Application: cart
|
+-- Deployment/cart
+-- Service/cart
+-- ConfigMap/cart
+-- HPA/cart
```

If another system owns the same resource:

```text
Argo CD
    +
Helm CLI
    +
Operator
```

they can fight over configuration.

This can produce reconciliation loops or unexpected changes.

Therefore resource ownership must be clearly defined.

---

# 88. GitOps Anti-Conflict Rule

A strong production rule is:

> One resource should have one clearly defined source of management authority.

For example:

```text
Deployment/cart
    |
    v
Argo CD
```

Avoid:

```text
Argo CD
   +
Jenkins
   +
manual kubectl
   +
Helm CLI
```

all modifying the same Deployment.

If multiple controllers legitimately interact with a resource, their responsibilities must be clearly separated.

---

# 89. GitOps and Platform Bootstrapping

Argo CD itself can be bootstrapped through GitOps.

A high-level pattern is:

```text
Terraform
   |
   v
EKS
   |
   v
Install Argo CD
   |
   v
Root Application
   |
   v
Platform Applications
   |
   +--> ingress
   +--> monitoring
   +--> security
   +--> workloads
```

This becomes the foundation for the App of Apps pattern.

The detailed implementation will appear later.

---

# 90. GitOps and App of Apps Concept

A root Argo CD Application can manage child Applications.

Conceptually:

```text
platform-root
      |
      +--> cart
      +--> catalog
      +--> orders
      +--> payment
      +--> user
      +--> monitoring
      +--> ingress
```

This allows a platform to bootstrap a complete environment from one root application.

The App of Apps pattern will be covered in the advanced Argo CD portion.

---

# 91. GitOps and ApplicationSets

ApplicationSets solve a different scaling problem.

Instead of manually creating:

```text
cart-dev
cart-qa
cart-prod
```

an ApplicationSet can generate Applications from structured inputs.

For example:

```text
Environment list
    |
    +--> dev
    +--> qa
    +--> prod
```

becomes:

```text
cart-dev
cart-qa
cart-prod
```

With cluster generators, the same concept can scale across:

```text
cluster-a
cluster-b
cluster-c
```

This is one of the most important capabilities in enterprise Argo CD.

---

# 92. GitOps at Enterprise Scale

A large organization may have:

```text
Hundreds of services
       |
       v
Multiple environments
       |
       v
Multiple EKS clusters
       |
       v
Multiple AWS accounts
       |
       v
Multiple teams
```

Manual Argo CD Application management becomes difficult.

Enterprise GitOps therefore introduces:

- ApplicationSets
- AppProjects
- RBAC
- Standard Helm charts
- Environment overlays
- Cluster labels
- Automated promotion
- Policy
- Observability
- HA
- DR
- Governance

The later files build this model step by step.

---

# 93. GitOps Production Decision: One Repo or Multiple Repos?

There is no universal answer.

## Single GitOps repository

Advantages:

- Central visibility
- Simple discovery
- Easier centralized governance

Challenges:

- Large repository
- Many teams touching one repository
- Access-control complexity

## Multiple GitOps repositories

Advantages:

- Stronger ownership boundaries
- Team autonomy
- Smaller repositories

Challenges:

- Cross-repository governance
- More Argo CD repository configuration
- More operational overhead

A common enterprise design is a combination:

```text
Platform GitOps Repo
Application Team GitOps Repos
```

---

# 94. GitOps Branching Strategies

Possible models include:

### Single main branch

```text
main
 |
 +--> dev
 +--> qa
 +--> prod
```

Environment is represented through directories or values.

### Environment branches

```text
dev
qa
prod
```

Each branch represents an environment.

### Release branches

```text
main
release/*
```

The correct strategy depends on promotion and governance requirements.

For many GitOps systems, environment directories/overlays combined with pull requests provide clearer promotion than long-lived environment branches, but there is no single universal rule.

---

# 95. GitOps Environment Promotion Through Git

Example:

```text
environments/
├── dev/
│   └── cart-values.yaml
├── qa/
│   └── cart-values.yaml
└── prod/
    └── cart-values.yaml
```

Dev:

```yaml
image:
  tag: "2026.08.19-abc123"
```

After validation, QA is updated:

```yaml
image:
  tag: "2026.08.19-abc123"
```

After production approval:

```yaml
image:
  tag: "2026.08.19-abc123"
```

The same artifact is promoted.

---

# 96. GitOps and Release Approval

Production environments may require manual approval.

Example:

```text
CI
 |
 v
ECR
 |
 v
DEV
 |
 v
Automated tests
 |
 v
QA
 |
 v
Approval
 |
 v
PROD
```

The approval can occur through Git pull requests.

This means the deployment process remains auditable while still supporting controlled production releases.

---

# 97. GitOps and Automated Promotion

Some organizations automate promotion.

Example:

```text
Image scan
   |
   v
Dev deployment
   |
   v
Automated validation
   |
   v
QA
   |
   v
Automated analysis
   |
   v
Production
```

Automation should only be used when:

- Tests are strong
- Monitoring is strong
- Rollback is reliable
- Business risk is acceptable

GitOps makes automation easier because promotion is represented as a declarative state change.

---

# 98. GitOps and Production Guardrails

Production guardrails may include:

```text
Allowed image registry
Required image digest
Required resource limits
Required probes
No privileged containers
Required labels
Approved namespaces
Approved repositories
Approved clusters
Production approval
```

Guardrails can be implemented at different layers:

```text
Git
CI
Policy engine
Argo CD
Kubernetes admission
AWS IAM
```

Defense in depth is preferred.

---

# 99. GitOps Observability Requirements

At minimum, operational teams should know:

```text
Which applications are OutOfSync?
Which applications are Degraded?
Which syncs are failing?
Which repositories are failing?
Which clusters are unreachable?
Which controllers are unhealthy?
```

A production dashboard may show:

```text
Total Applications: 120
Synced: 112
OutOfSync: 5
Degraded: 3
Unknown: 0
```

The exact metrics and dashboards will be covered later.

---

# 100. GitOps Troubleshooting Mental Model

When something goes wrong, do not immediately run random commands.

Use this chain:

```text
1. Git
   |
   | Is desired state correct?
   v
2. Argo CD source
   |
   | Can it fetch/render?
   v
3. Argo CD comparison
   |
   | What is OutOfSync?
   v
4. Sync operation
   |
   | Did Kubernetes accept it?
   v
5. Kubernetes resource
   |
   | Is the resource healthy?
   v
6. Pod
   |
   | Logs/events/probes?
   v
7. Application
```

This prevents jumping directly to the wrong layer.

---

# 101. Example Troubleshooting Scenario

Suppose Argo CD reports:

```text
Synced
Degraded
```

Do not assume Argo CD failed.

Start with:

```bash
argocd app get roboshop-cart
```

Then inspect Kubernetes:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Possible root causes:

```text
CrashLoopBackOff
ImagePullBackOff
OOMKilled
Failed probe
Configuration error
Dependency failure
```

Argo CD may have successfully deployed the desired configuration.

The application itself may be unhealthy.

---

# 102. Production Runbook Principle

A GitOps runbook should identify the layer of failure.

```text
Git problem?
      |
      v
Source problem?
      |
      v
Render problem?
      |
      v
Argo CD problem?
      |
      v
Kubernetes API problem?
      |
      v
Resource problem?
      |
      v
Pod problem?
      |
      v
Application problem?
```

This layered model is essential for real production troubleshooting.

---

# 103. Interview: What Is GitOps?

### Strong answer

> GitOps is a declarative continuous delivery and operations model where Git stores the desired state of an environment. A GitOps controller such as Argo CD continuously compares that desired state with the actual Kubernetes state and reconciles differences. In our EKS architecture, CI builds, tests, scans, and pushes immutable container images to ECR, then the approved deployment configuration is updated in a GitOps repository. Argo CD detects that change and reconciles the workload into EKS. This provides version control, auditability, drift detection, controlled rollback, and a reduced need for direct production cluster access from CI.

---

# 104. Interview: Why Use GitOps Instead of kubectl From Jenkins?

### Strong answer

> Direct kubectl deployment from Jenkins creates a push-based relationship where Jenkins needs production cluster credentials and directly modifies the runtime environment. With GitOps, Jenkins can build and validate the image, push it to ECR, and update the desired deployment configuration in Git. Argo CD then pulls that desired state and reconciles EKS. This improves separation of duties, auditability, drift management, and security while keeping deployment state version controlled.

---

# 105. Interview: What Happens If Someone Manually Changes Production?

### Strong answer

> The manual change can create configuration drift if it differs from the desired state in Git. Argo CD detects the difference during reconciliation. If automated sync with self-healing is enabled and the resource is eligible for reconciliation, Argo CD can restore the Git-defined state. If self-healing is not enabled, the application may remain OutOfSync until a sync is performed.

---

# 106. Interview: Does GitOps Mean Git Contains Everything?

### Strong answer

> No. GitOps should contain the declarative desired state for the systems managed through GitOps, but not necessarily every piece of runtime or external state. Secrets may need a dedicated secrets-management solution, databases contain persistent data outside Git, and infrastructure may be managed separately with Terraform. The important requirement is to define clear ownership and a reliable source of truth for each category of state.

---

# 107. Interview: GitOps vs IaC

### Strong answer

> Infrastructure as Code is a broader practice of managing infrastructure through code. Terraform is commonly used to provision AWS infrastructure such as VPCs, EKS, IAM, RDS, and ECR. GitOps is an operational model centered on declarative state stored in Git and continuously reconciled by a controller. In a production EKS platform, Terraform can provision the infrastructure and Argo CD can manage Kubernetes application and platform resources.

---

# 108. Interview: What Is Drift?

### Strong answer

> Drift occurs when the actual state of a managed environment differs from the desired state declared in Git. For example, if Git specifies three replicas but someone manually scales the Deployment to one replica, the cluster has drifted. Argo CD can detect that difference and, with appropriate synchronization and self-healing configuration, reconcile the cluster back toward the desired state.

---

# 109. Interview: Why Is Reconciliation Important?

### Strong answer

> Reconciliation makes GitOps continuous rather than a one-time deployment mechanism. The controller repeatedly evaluates desired and actual state. If someone changes a resource manually or the runtime deviates from the declared configuration, the controller can detect the difference and take corrective action. This is the core behavior that enables drift management and continuous convergence.

---

# 110. Interview: Is Argo CD a CI Tool?

### Strong answer

> Argo CD is primarily a Kubernetes GitOps continuous delivery and reconciliation tool, not a CI build system. CI tools such as Jenkins or GitHub Actions build, test, scan, and publish artifacts. Argo CD consumes the desired deployment configuration and reconciles Kubernetes resources. The two systems complement each other rather than replacing each other.

---

# 111. Interview: What Is Pull-Based Deployment?

### Strong answer

> In a pull-based GitOps model, the deployment controller running with access to the Kubernetes environment retrieves the desired state from Git and reconciles the cluster. The CI system does not need to actively push Kubernetes changes into production. This can reduce production credentials in CI and provides continuous reconciliation through the controller.

---

# 112. Interview: Can Argo CD Manage Multiple EKS Clusters?

### Strong answer

> Yes. Argo CD can act as a centralized GitOps control plane and manage multiple Kubernetes clusters. Target EKS clusters are registered with Argo CD, and Applications specify their destinations. ApplicationSets can generate Applications for multiple clusters using cluster metadata and labels. In an enterprise AWS setup, the clusters may exist in separate environments or AWS accounts, with access controlled through appropriate authentication, RBAC, and network boundaries.

---

# 113. Interview: What Is the Difference Between Synced and Healthy?

### Strong answer

> Synced describes whether the application resources match the desired state represented by the configured Git revision. Healthy describes the runtime health of those resources. An application can therefore be Synced but Degraded—for example, Argo CD may have successfully deployed the exact image declared in Git, but the Pods may be CrashLoopBackOff.

---

# 114. Interview: What Is a Good GitOps Rollback Strategy?

### Strong answer

> The preferred rollback strategy is to restore a known-good desired state through Git when possible. For example, if a production image update causes an incident, the Git commit can be reverted so the GitOps repository again declares the previous image. Argo CD then reconciles the cluster. However, application rollback must also consider database migrations, persistent data, and external dependencies because reverting Kubernetes manifests alone does not automatically reverse data changes.

---

# 115. Scenario Interview: Argo CD Says Synced but Application Is Down

### Question

Argo CD shows:

```text
Synced
Degraded
```

What do you do?

### Answer

First, I would distinguish desired-state synchronization from runtime health.

I would check:

```bash
argocd app get roboshop-cart
```

Then:

```bash
kubectl get pods -n roboshop
```

If Pods are failing:

```bash
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
```

Then inspect events:

```bash
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

I would investigate:

- CrashLoopBackOff
- OOMKilled
- ImagePullBackOff
- Failed probes
- Missing ConfigMaps/Secrets
- Application dependency failures
- Resource pressure

The key point is that Argo CD may have successfully synchronized the desired state while the application itself is unhealthy.

---

# 116. Scenario Interview: Production Drift

### Question

An engineer manually changes a production Deployment. What should happen?

### Answer

The manual change should be treated as an emergency or exceptional action, not as the new source of truth.

I would:

1. Determine what was changed.
2. Confirm whether the change was intentional.
3. If temporary, allow GitOps to restore the desired state when safe.
4. If the change should remain, update the GitOps repository.
5. Review and merge the corresponding Git change.
6. Verify Argo CD becomes Synced and Healthy.
7. Record the incident/change if required.

This maintains Git as the authoritative desired state.

---

# 117. Scenario Interview: Git Repository Is Down

### Question

What happens if the GitOps repository becomes unavailable?

### Answer

Existing Kubernetes workloads normally continue running because Kubernetes does not require Argo CD or Git for every Pod operation. However, Argo CD may not be able to retrieve new desired state, so new deployments and some reconciliation operations can be delayed.

Production controls should include:

- Reliable Git hosting
- Repository backups
- Access recovery
- Monitoring
- Disaster recovery procedures

---

# 118. Scenario Interview: Argo CD Is Down

### Answer

Existing workloads should normally continue running because Argo CD is not in the application request path.

However:

```text
New Git changes -> delayed
Drift correction -> delayed
Automated synchronization -> unavailable
```

Therefore a production Argo CD installation should have appropriate availability, monitoring, and recovery procedures.

---

# 119. Scenario Interview: Why Not Use latest?

### Answer

A mutable `latest` tag makes it difficult to determine which exact artifact is running and makes rollback less deterministic.

I would use an immutable version such as:

```text
2026.08.19-abc123
```

or an image digest.

This allows Git to identify the exact artifact that should run in each environment.

---

# 120. Scenario Interview: Same Image or Rebuild Per Environment?

### Answer

I would prefer promoting the same immutable artifact across environments:

```text
Build once
   |
   v
ECR
   |
   +--> DEV
   |
   +--> QA
   |
   +--> PROD
```

Environment-specific differences should generally be represented through configuration, not by rebuilding the application image for every environment.

This reduces the risk that QA validates a different artifact from the one eventually deployed to production.

---

# 121. Practical GitOps Learning Exercise

Create a small repository:

```text
gitops-lab/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── applications/
├── applicationsets/
└── projects/
```

Create a basic Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: demo
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Commit it.

The purpose at this stage is to understand the desired-state concept.

Argo CD implementation begins in the later files.

---

# 122. Practical Drift Exercise

After deployment, intentionally change the runtime:

```bash
kubectl scale deployment demo \
  --replicas=1 \
  -n demo
```

Then compare:

```text
Git:
replicas = 2

Cluster:
replicas = 1
```

This creates an observable drift scenario.

Later, repeat the exercise using Argo CD with self-healing enabled.

---

# 123. Practical Rollback Exercise

Create two Git revisions:

```text
Commit A:
image = nginx:1.27

Commit B:
image = nginx:<new-version>
```

Deploy B.

Then revert B:

```bash
git revert <commit-b>
git push
```

Observe how the GitOps controller returns the environment to the desired state.

This teaches an important principle:

```text
Rollback is a desired-state change.
```

---

# 124. Practical RoboShop Exercise

For the RoboShop platform, identify:

### Application repository

```text
roboshop-cart
```

### Artifact

```text
Amazon ECR
```

### GitOps configuration

```text
roboshop-gitops
```

### Controller

```text
Argo CD
```

### Runtime

```text
Amazon EKS
```

### External traffic

```text
AWS ALB Ingress
```

Draw the flow:

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins / GitHub Actions
   |
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker
   |
   v
ECR
   |
   v
GitOps Repo
   |
   v
Argo CD
   |
   v
EKS
   |
   v
ALB
```

This architecture will become the practical foundation for the RoboShop GitOps project later.

---

# 125. Production Readiness Checklist

Before adopting GitOps for a production EKS workload, verify:

## Source control

- [ ] GitOps repository exists.
- [ ] Main/production branches are protected.
- [ ] Pull requests are required.
- [ ] Appropriate reviewers are configured.
- [ ] Ownership is documented.
- [ ] Repository backups/recovery are understood.

## CI

- [ ] Application builds successfully.
- [ ] Tests run.
- [ ] SonarQube checks are configured where required.
- [ ] Trivy scans images.
- [ ] Veracode checks are integrated where required.
- [ ] Images are pushed to ECR.
- [ ] Immutable image versions are used.

## GitOps

- [ ] Desired state is declarative.
- [ ] Argo CD is installed.
- [ ] Repositories are configured securely.
- [ ] Applications are defined.
- [ ] Projects restrict access.
- [ ] Sync policy is documented.
- [ ] Prune behavior is understood.
- [ ] Self-healing behavior is understood.

## Kubernetes

- [ ] Namespace strategy exists.
- [ ] Resource requests/limits are defined.
- [ ] Probes are configured.
- [ ] Security contexts are used.
- [ ] Ingress is defined.
- [ ] HPA is configured where appropriate.
- [ ] RBAC follows least privilege.

## Operations

- [ ] Argo CD is monitored.
- [ ] Application health is monitored.
- [ ] Sync failures are alerted.
- [ ] Rollback procedure is tested.
- [ ] Emergency access procedure exists.
- [ ] Disaster recovery procedure exists.

---

# 126. Key Concepts to Remember

The most important mental model from this file is:

```text
              Git
               |
               | Desired State
               v
            Argo CD
               |
               | Reconciliation
               v
        Kubernetes API
               |
               v
              EKS
               |
               | Actual State
               v
          Running Workloads
```

And the production delivery model is:

```text
Developer
   |
   v
Application Git
   |
   v
CI
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
Argo CD
   |
   v
EKS
   |
   v
Kubernetes
   |
   v
ALB / Application
```

---

# 127. Final Interview Summary

If asked to explain GitOps in one answer:

> GitOps is a declarative operational model where Git stores the desired state of applications and infrastructure configuration, and a GitOps controller continuously reconciles the runtime environment with that state. In a Kubernetes environment, Argo CD acts as the GitOps controller. In our AWS EKS architecture, Jenkins or GitHub Actions handles CI activities such as build, testing, SonarQube, Trivy, Veracode, and pushing immutable images to ECR. The deployment configuration and image version are then maintained in a GitOps repository. Argo CD detects the desired-state change, renders the configuration, compares it with the actual cluster state, and synchronizes the resources through the Kubernetes API. Continuous reconciliation provides drift detection and, when configured, self-healing. GitOps also gives us versioned change history, pull-request approvals, auditability, controlled rollback, and reduced direct production access from CI. For larger environments, Argo CD can operate as a centralized control plane managing multiple EKS clusters across environments and AWS accounts.

---

# 128. What This File Established

This file established the foundation required before learning Argo CD:

```text
GitOps
  |
  +--> Declarative state
  |
  +--> Git source of truth
  |
  +--> Desired vs actual state
  |
  +--> Drift
  |
  +--> Reconciliation
  |
  +--> Pull model
  |
  +--> CI/CD separation
  |
  +--> Rollback
  |
  +--> Auditability
  |
  +--> Security boundaries
  |
  +--> Environment management
  |
  +--> EKS integration
  |
  +--> Multi-cluster concept
  |
  +--> Production operations
```