# ArgoCD-Installation-and-Configuration

## 1. Purpose

This file covers the practical installation and production configuration of Argo CD on Kubernetes, with AWS EKS as the primary environment.

The objective is not only to install Argo CD successfully, but to understand:

- Installation prerequisites
- Kubernetes namespace
- Installation methods
- Argo CD components
- Helm installation
- Manifest installation
- EKS networking
- ALB exposure
- TLS
- CLI configuration
- Initial administration
- Repository configuration
- Private repository access
- Cluster registration
- RBAC
- Projects
- Resource sizing
- High availability
- Security hardening
- Production validation
- Upgrades
- Troubleshooting
- Operational runbooks
- Interview questions

The previous file explained the architecture.

This file explains how to turn that architecture into a working platform.

---

# 2. Installation Architecture

A basic installation looks like:

```text
AWS
 |
 v
EKS Cluster
 |
 +--> argocd namespace
       |
       +--> argocd-server
       +--> argocd-repo-server
       +--> argocd-application-controller
       +--> argocd-redis
       +--> argocd-applicationset-controller
```

For a production installation:

```text
                    Users
                      |
                      v
                 DNS / HTTPS
                      |
                      v
                  AWS ALB
                      |
                      v
                argocd-server
                      |
          +-----------+-----------+
          |                       |
          v                       v
     Repo Server             Application
                              Controller
          |                       |
          v                       v
         Git                  EKS APIs
```

---

# 3. Installation Philosophy

There are two different goals:

### Learning installation

```text
Install quickly
Understand components
Test applications
```

### Production installation

```text
Install
+
Secure
+
Expose safely
+
Authenticate
+
Scale
+
Monitor
+
Back up/recover
+
Upgrade
```

Do not confuse a successful Pod deployment with a production-ready GitOps control plane.

---

# 4. EKS Prerequisites

Before installing Argo CD on EKS, verify:

```text
AWS account
EKS cluster
kubectl access
AWS CLI
Helm
Git repository
DNS strategy
TLS strategy
Ingress/load-balancer strategy
```

Useful commands:

```bash
aws --version
kubectl version --client
helm version
aws sts get-caller-identity
```

---

# 5. Verify AWS Identity

Run:

```bash
aws sts get-caller-identity
```

This confirms which AWS identity is currently being used.

Example output conceptually:

```text
Account
Arn
UserId
```

Always verify the account before making production changes.

A common operational mistake is running:

```bash
kubectl
helm
aws
```

against the wrong environment.

---

# 6. Verify EKS Access

List clusters:

```bash
aws eks list-clusters
```

Update kubeconfig for the intended cluster:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster-name>
```

Then:

```bash
kubectl config current-context
```

Verify:

```bash
kubectl get nodes
```

---

# 7. Production Safety Check

Before installation, confirm:

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl get nodes
```

These three checks establish:

```text
AWS account
+
Kubernetes context
+
Cluster availability
```

Do this before destructive or production-impacting commands.

---

# 8. Check Cluster Version

Use:

```bash
kubectl version
```

and:

```bash
aws eks describe-cluster \
  --name <cluster-name> \
  --query 'cluster.version'
```

Argo CD versions should be selected with compatibility in mind.

Do not blindly install the newest release into an old production cluster without checking compatibility.

---

# 9. Verify Helm

Run:

```bash
helm version
```

Helm is useful for:

- Argo CD installation
- Argo CD configuration
- Production overrides
- Resource settings
- Ingress configuration
- HA settings

---

# 10. Verify Kubernetes Permissions

Before installing:

```bash
kubectl auth can-i create namespace
kubectl auth can-i create deployments
kubectl auth can-i create customresourcedefinitions
```

The exact permissions depend on the installation method and cluster security model.

---

# 11. Create the Argo CD Namespace

A dedicated namespace is normally used:

```bash
kubectl create namespace argocd
```

Verify:

```bash
kubectl get namespace argocd
```

Expected:

```text
argocd
```

---

# 12. Why Use a Dedicated Namespace?

A dedicated namespace provides:

- Logical isolation
- Easier RBAC
- Easier resource management
- Easier troubleshooting
- Cleaner monitoring
- Easier backup/restore organization

For example:

```text
argocd
 |
 +--> control-plane components

roboshop
 |
 +--> application workloads
```

Do not mix Argo CD control-plane resources with normal application workloads unless there is a specific reason.

---

# 13. Argo CD Installation Methods

Common approaches include:

```text
1. Official Kubernetes manifests
2. Helm
3. Infrastructure automation
4. GitOps bootstrap
```

For learning, official manifests are simple.

For production, Helm or an equivalent declarative installation approach is often easier to customize and manage.

The exact installation method should be standardized by the organization.

---

# 14. Manifest-Based Installation

A typical installation process uses the official Argo CD installation manifest appropriate for the chosen Argo CD release.

Conceptually:

```bash
kubectl apply -n argocd \
  -f <official-argocd-install-manifest>
```

Do not hard-code an old version from a tutorial.

Always pin and review the Argo CD release used in production.

---

# 15. Why Version Pinning Matters

Never build a production process around:

```text
latest
```

Prefer:

```text
known Argo CD version
```

and record it in:

```text
Infrastructure repository
Change documentation
Release management
```

Benefits:

- Reproducibility
- Easier rollback
- Controlled upgrades
- Compatibility testing

---

# 16. Verify Installation

Run:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl get svc -n argocd
```

Then:

```bash
kubectl get deployments -n argocd
```

Also inspect:

```bash
kubectl get crd | grep argoproj
```

You should see Argo CD-related custom resources.

---

# 17. Argo CD CRDs

Argo CD uses Kubernetes Custom Resource Definitions.

Important resources include:

```text
applications.argoproj.io
applicationsets.argoproj.io
appprojects.argoproj.io
```

These enable Kubernetes-native management of:

```text
Applications
ApplicationSets
Projects
```

---

# 18. Kubernetes-Native Model

Argo CD is not a separate database-driven deployment system.

Its core objects are represented through Kubernetes APIs.

For example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
```

This means Applications can be managed declaratively.

---

# 19. Check Argo CD Pods

Run:

```bash
kubectl get pods -n argocd -o wide
```

Typical components include:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-redis
argocd-applicationset-controller
```

The exact workload types and component set depend on the installed version/configuration.

---

# 20. Check Component Services

Run:

```bash
kubectl get svc -n argocd
```

Inspect:

```bash
kubectl describe svc argocd-server -n argocd
```

This helps identify:

- Service type
- Cluster IP
- Ports
- Endpoints
- Selectors

---

# 21. Initial Access

For initial learning access, port-forwarding is convenient:

```bash
kubectl port-forward svc/argocd-server \
  -n argocd 8080:443
```

Then connect to:

```text
https://localhost:8080
```

This avoids exposing Argo CD publicly while learning.

---

# 22. Port-Forwarding Is Not a Production Exposure Model

Port-forwarding is excellent for:

```text
Learning
Troubleshooting
Temporary administration
```

It is not a production ingress architecture.

Production should normally use:

```text
DNS
+
TLS
+
ALB/Ingress
+
Authentication
+
RBAC
```

---

# 23. Initial Admin Credential

Argo CD installations commonly provide an initial administrative credential through a Kubernetes Secret.

Inspect the Secret:

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd
```

Retrieve the password using an appropriate Kubernetes Secret decoding command.

Example:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

The exact Secret name can vary by installation/version, so verify the installed resources.

---

# 24. Initial Admin Account

The initial administrator is useful for bootstrap.

Production practice:

```text
Bootstrap admin
      |
      v
Configure SSO
      |
      v
Create controlled access
      |
      v
Disable/remove unnecessary bootstrap access
```

Do not leave a shared bootstrap administrator as the long-term identity model.

---

# 25. Why SSO Is Better for Production

Shared local credentials create problems:

- No individual identity
- Poor accountability
- Difficult offboarding
- Password sharing
- Harder audit trail

Enterprise SSO provides:

```text
Individual identity
+
Centralized lifecycle
+
MFA
+
Group membership
+
Auditability
```

---

# 26. Argo CD CLI Installation

Install the Argo CD CLI appropriate to the environment.

Verify:

```bash
argocd version --client
```

The CLI version should be compatible with the server version.

---

# 27. Login Through CLI

After exposing the API server:

```bash
argocd login <argocd-host>
```

For a temporary insecure development setup, additional flags may be required depending on TLS configuration.

Do not disable TLS verification in production as a permanent solution.

---

# 28. Verify CLI Access

Run:

```bash
argocd version
```

Then:

```bash
argocd account get-user-info
```

Then:

```bash
argocd app list
```

A successful connection confirms:

```text
CLI
 |
 v
API Server
 |
 v
Argo CD
```

---

# 29. Argo CD Server Exposure on AWS

For production EKS, a common architecture is:

```text
Internet / Corporate Network
          |
          v
        DNS
          |
          v
        AWS ALB
          |
          v
    Argo CD Ingress
          |
          v
    argocd-server
```

The ALB can provide:

- TLS termination
- Routing
- Health checks
- Integration with AWS networking

---

# 30. AWS ALB vs API Gateway

For the user's architecture, use:

```text
AWS ALB Ingress
```

not:

```text
API Gateway
```

The production path is:

```text
Client
  |
  v
Route 53
  |
  v
AWS ALB
  |
  v
Argo CD Server
```

This matches the user's existing EKS/ALB architecture.

---

# 31. ALB Prerequisites

An AWS ALB-based ingress normally requires:

```text
AWS Load Balancer Controller
```

and appropriate:

```text
IAM permissions
Subnet tags
Security groups
Ingress configuration
```

Verify:

```bash
kubectl get deployment -A | grep aws-load-balancer-controller
```

---

# 32. Verify AWS Load Balancer Controller

Run:

```bash
kubectl get pods -n kube-system | \
  grep aws-load-balancer-controller
```

Then:

```bash
kubectl logs deployment/aws-load-balancer-controller \
  -n kube-system
```

Do this before troubleshooting an Argo CD ALB.

---

# 33. Production DNS

Use a stable hostname such as:

```text
argocd.example.com
```

Architecture:

```text
argocd.example.com
       |
       v
Route 53
       |
       v
AWS ALB
       |
       v
Argo CD
```

Avoid exposing the Argo CD service through an unstable node IP.

---

# 34. TLS

Production Argo CD access should use HTTPS.

Example:

```text
https://argocd.example.com
```

TLS can be terminated at the ALB.

The architecture may be:

```text
Client
  |
 HTTPS
  v
ALB
  |
 HTTP/HTTPS
  v
Argo CD Server
```

The exact backend protocol should be selected based on the chosen Argo CD ingress configuration and security requirements.

---

# 35. ACM Certificate

For AWS ALB, AWS Certificate Manager can provide the TLS certificate.

Conceptually:

```text
ACM
 |
 v
ALB
 |
 v
Argo CD
```

The certificate should cover the Argo CD hostname.

---

# 36. Security Group Considerations

The ALB security group should allow only required traffic.

Typical concept:

```text
Internet/Corporate Network
       |
       | HTTPS 443
       v
ALB Security Group
       |
       v
Argo CD Target
```

Do not expose administrative interfaces broadly when a corporate/private access path is available.

---

# 37. Production Ingress Example

A conceptual Kubernetes Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server
  namespace: argocd
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:<region>:<account>:certificate/<certificate-id>
spec:
  ingressClassName: alb
  rules:
    - host: argocd.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
```

This is a production-style starting point, not a universal drop-in manifest.

Validate the Argo CD server service/protocol and AWS Load Balancer Controller behavior in the target version.

---

# 38. Why Internal ALB Can Be Better

For enterprise administration:

```text
Corporate Network
      |
      v
Internal ALB
      |
      v
Argo CD
```

Benefits:

- Reduced public exposure
- Easier network control
- Private DNS
- Corporate identity integration
- Smaller attack surface

Whether an internal or internet-facing ALB is appropriate depends on organizational access requirements.

---

# 39. Argo CD Server Service

Inspect:

```bash
kubectl get svc argocd-server -n argocd -o yaml
```

Important fields include:

```text
type
ports
targetPort
selector
```

The service connects the ingress/load balancer to the Argo CD server Pods.

---

# 40. ALB Troubleshooting

If ALB is not created:

```bash
kubectl describe ingress argocd-server -n argocd
```

Check:

```text
Events
IngressClass
Subnet discovery
Security groups
IAM permissions
Certificate ARN
Target health
```

Then:

```bash
kubectl logs deployment/aws-load-balancer-controller \
  -n kube-system
```

---

# 41. Argo CD Repository Configuration

Once Argo CD is running, configure the Git repositories that contain desired state.

Conceptually:

```text
Argo CD
   |
   v
Repo Server
   |
   v
GitOps Repository
```

Repository configuration should be declarative or otherwise controlled.

---

# 42. Repository Authentication Options

Private Git repositories commonly use:

```text
HTTPS + token
SSH + private key
```

The organization should choose a standardized method.

Security requirements:

```text
Least privilege
Credential rotation
No credentials in Git
Auditability
```

---

# 43. Add Repository Using CLI

The CLI supports repository configuration.

A conceptual example:

```bash
argocd repo add \
  https://github.com/example/gitops.git \
  --username <username> \
  --password <token>
```

For SSH:

```bash
argocd repo add \
  git@github.com:example/gitops.git \
  --ssh-private-key-path ~/.ssh/gitops_ed25519
```

Use the appropriate repository provider authentication model.

Never paste real production credentials into shell history or documentation.

---

# 44. Verify Repository

Run:

```bash
argocd repo list
```

You want to confirm:

```text
Repository URL
Connection status
Credential configuration
```

If the repository is unreachable, inspect Repo Server logs.

---

# 45. Repository Credentials and Security

Do not store:

```text
PAT
SSH private key
Cloud credential
```

inside the GitOps repository.

Instead:

```text
Secure credential
       |
       v
Argo CD repository configuration
       |
       v
Repo Server
```

The exact storage mechanism depends on the Argo CD installation and security architecture.

---

# 46. GitOps Repository Design

A production repository can look like:

```text
gitops-repo/
├── applications/
├── applicationsets/
├── projects/
├── clusters/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── helm/
├── manifests/
└── platform/
```

This structure is a starting point.

Repository organization should optimize:

```text
Ownership
Review
Promotion
Environment isolation
Reuse
```

---

# 47. Helm-Based Repository Structure

For RoboShop:

```text
gitops-repo/
└── helm/
    └── roboshop/
        ├── Chart.yaml
        ├── values.yaml
        ├── templates/
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   ├── ingress.yaml
        │   ├── configmap.yaml
        │   └── hpa.yaml
        └── values/
            ├── dev.yaml
            ├── qa.yaml
            └── prod.yaml
```

This enables environment-specific configuration.

---

# 48. Environment Structure

Another approach:

```text
environments/
├── dev/
│   ├── cart/
│   ├── user/
│   └── payment/
├── qa/
│   ├── cart/
│   ├── user/
│   └── payment/
└── prod/
    ├── cart/
    ├── user/
    └── payment/
```

Choose one primary organization pattern and avoid unnecessary duplication.

---

# 49. Argo CD Application Creation

A simple Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart
  namespace: argocd
spec:
  project: roboshop
  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: environments/dev/cart
  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop
```

This will be deeply explained in the next file.

---

# 50. Apply Application

Save:

```text
roboshop-cart.yaml
```

Then:

```bash
kubectl apply -f roboshop-cart.yaml
```

Verify:

```bash
kubectl get applications -n argocd
```

Then:

```bash
argocd app get roboshop-cart
```

---

# 51. Application Lifecycle

The Application moves through:

```text
Application created
       |
       v
Source fetched
       |
       v
Manifest rendered
       |
       v
Desired state calculated
       |
       v
Compared with cluster
       |
       v
Synced
       |
       v
Health evaluated
```

---

# 52. Manual Sync

If automated sync is not enabled:

```bash
argocd app sync roboshop-cart
```

Then:

```bash
argocd app get roboshop-cart
```

Verify:

```text
Sync = Synced
Health = Healthy
```

---

# 53. Application Diff

Before syncing:

```bash
argocd app diff roboshop-cart
```

This is an important production safety command.

It helps answer:

```text
What exactly will change?
```

Always use diff for significant production changes.

---

# 54. Application History

View deployment history:

```bash
argocd app history roboshop-cart
```

This can help identify:

```text
Revision
Deployment time
Source
Previous versions
```

It becomes useful during rollback analysis.

---

# 55. Rollback

Argo CD supports application rollback mechanisms based on recorded application history.

A conceptual command is:

```bash
argocd app rollback roboshop-cart <history-id>
```

However, GitOps best practice is usually to make Git reflect the desired rollback state.

Preferred:

```text
Identify bad revision
      |
      v
Revert Git commit
      |
      v
Argo CD
      |
      v
Reconcile
```

This keeps Git as the source of truth.

---

# 56. Emergency Rollback

During a critical outage, an operational rollback may be performed according to the organization's runbook.

After emergency action:

```text
Live state
    |
    v
Restore Git consistency
```

Never leave a permanent production rollback that exists only outside Git.

---

# 57. Registering a Target EKS Cluster

A centralized Argo CD instance needs access to target clusters.

Conceptually:

```text
Argo CD Management EKS
          |
          v
Target EKS API
```

The cluster must be registered with appropriate credentials and permissions.

---

# 58. List Registered Clusters

Run:

```bash
argocd cluster list
```

This displays registered destinations.

Look for:

```text
SERVER
NAME
VERSION
STATUS
```

The exact columns depend on the Argo CD CLI version.

---

# 59. Registering the Current Cluster

Argo CD can register Kubernetes contexts.

Conceptually:

```bash
argocd cluster add <kube-context>
```

This configures the necessary access for Argo CD.

Before executing in production, inspect exactly what permissions and service account configuration the command will establish.

---

# 60. Security Warning for cluster add

Do not blindly run:

```bash
argocd cluster add
```

against production.

Understand:

```text
Which identity is created?
Which namespace?
Which permissions?
Which cluster?
```

The target cluster credentials represent powerful deployment access.

---

# 61. EKS Authentication Considerations

EKS cluster authentication involves AWS and Kubernetes authorization.

A production architecture may involve:

```text
Argo CD
 |
 v
Kubernetes authentication
 |
 v
EKS
 |
 v
Kubernetes authorization
```

The exact implementation depends on the EKS access model and Argo CD cluster credential configuration.

---

# 62. EKS Cluster Access Models

Modern EKS environments may use mechanisms such as:

```text
EKS access entries
IAM authentication
Kubernetes RBAC
```

The organization should standardize cluster access.

Avoid mixing several undocumented access models.

---

# 63. Cluster Credentials as Secrets

Argo CD stores target cluster connection information in Kubernetes resources/secrets managed by Argo CD.

Conceptually:

```text
argocd namespace
 |
 +--> cluster credential
 |
 v
Application Controller
 |
 v
Target EKS
```

These credentials must be treated as highly sensitive.

---

# 64. Multi-Cluster Registration

Suppose:

```text
EKS-DEV
EKS-QA
EKS-PROD
```

Register each cluster:

```text
Argo CD
 |
 +--> DEV credentials
 +--> QA credentials
 +--> PROD credentials
```

Then Applications can select the appropriate destination.

---

# 65. Cluster Naming

Use meaningful names:

```text
eks-dev
eks-qa
eks-prod
```

or:

```text
dev-us-east-1
qa-us-east-1
prod-us-east-1
```

Naming should make environment and region obvious.

---

# 66. Cluster Labels

Labels become important for ApplicationSet.

Example conceptual metadata:

```text
environment=prod
team=platform
region=ap-south-1
```

Then ApplicationSets can target:

```text
environment=prod
```

rather than hard-coding every cluster.

---

# 67. AppProject Configuration

Create a Project:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop
  namespace: argocd
spec:
  description: RoboShop applications
  sourceRepos:
    - https://github.com/example/gitops.git
  destinations:
    - namespace: roboshop
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ""
      kind: Namespace
  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
```

This is a production-oriented example, but resource permissions should be adapted to the actual workload.

---

# 68. Why Whitelisting Matters

The project can define:

```text
Allowed sources
Allowed destinations
Allowed resource kinds
```

This creates a security boundary.

For example:

```text
RoboShop project
      |
      +--> GitOps repo only
      |
      +--> roboshop namespace only
      |
      +--> Approved resource kinds
```

---

# 69. Namespace Creation

Applications may use:

```yaml
syncOptions:
  - CreateNamespace=true
```

This can simplify environment bootstrapping.

However, production teams may prefer namespaces to be managed by a platform layer.

There is no universal answer.

---

# 70. Platform vs Application Ownership

A mature model may be:

```text
Platform GitOps
 |
 +--> Namespaces
 +--> RBAC
 +--> Network policies
 +--> Ingress infrastructure
 +--> Observability

Application GitOps
 |
 +--> Deployments
 +--> Services
 +--> ConfigMaps
 +--> HPAs
```

This avoids applications accidentally owning shared platform resources.

---

# 71. Resource Requests and Limits

Argo CD does not replace Kubernetes resource management.

Application manifests should define:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Values should be based on measured application requirements.

---

# 72. Health Probes

Production applications should normally define appropriate:

```text
livenessProbe
readinessProbe
startupProbe
```

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
```

Argo CD can observe resulting Kubernetes health, but the application team must define meaningful probes.

---

# 73. Security Context

Production workloads should use security hardening where supported:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

Exact settings depend on application requirements.

GitOps makes these policies reviewable and repeatable.

---

# 74. Secrets

Do not place real secrets in plain Git:

```yaml
password: SuperSecret123
```

Instead use an external secret-management strategy.

Possible architecture:

```text
AWS Secrets Manager
        |
        v
External Secrets mechanism
        |
        v
Kubernetes Secret
        |
        v
Application
```

Argo CD manages the declarative integration, not necessarily the secret value itself.

Detailed secrets management is covered later.

---

# 75. ECR Image Strategy

For RoboShop:

```text
CI
 |
 v
Docker Build
 |
 v
Security Scan
 |
 v
ECR
```

Use immutable image references where possible.

For example:

```text
roboshop/cart:1.4.7
```

or digest pinning:

```text
image@sha256:<digest>
```

Avoid relying on:

```text
latest
```

for production.

---

# 76. CI + Argo CD Integration

The intended architecture:

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
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
GitOps repository
   |
   v
Argo CD
   |
   v
EKS
```

CI does not directly deploy to EKS in the normal GitOps flow.

---

# 77. Image Tag Update

Suppose CI builds:

```text
roboshop/cart:2026.08.19-abc123
```

CI updates the GitOps value:

```yaml
image:
  repository: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/cart
  tag: 2026.08.19-abc123
```

Then:

```text
Git commit
   |
   v
Argo CD detects change
   |
   v
Sync
   |
   v
EKS
```

---

# 78. Why CI Should Not Directly kubectl Apply

Traditional:

```text
CI
 |
 v
kubectl apply
 |
 v
EKS
```

GitOps:

```text
CI
 |
 +--> ECR
 +--> Git
      |
      v
   Argo CD
      |
      v
     EKS
```

Benefits:

- Auditability
- Reduced cluster credentials in CI
- Declarative state
- Drift correction
- Easier rollback

---

# 79. Production Argo CD Configuration Layers

Think in layers:

```text
Layer 1
Argo CD installation

Layer 2
Authentication / RBAC

Layer 3
Repositories

Layer 4
Clusters

Layer 5
Projects

Layer 6
Applications

Layer 7
ApplicationSets

Layer 8
Observability / Notifications

Layer 9
Backup / Recovery
```

This layered model helps organize implementation.

---

# 80. Production Resource Configuration

For each Argo CD component, evaluate:

```text
CPU request
Memory request
CPU limit
Memory limit
Replicas
Pod anti-affinity
Topology spread
PDB
```

Use measured workload data rather than arbitrary numbers.

---

# 81. High Availability

For production, consider:

```text
Multiple API Server replicas
Multiple Repo Server replicas
Application Controller HA/scaling
Redis HA where appropriate
ApplicationSet Controller availability
```

Also ensure replicas are distributed across nodes/AZs where appropriate.

---

# 82. Pod Anti-Affinity

If multiple replicas exist, avoid placing every replica on the same node.

Conceptually:

```text
Node A
  |
  +--> API Server 1

Node B
  |
  +--> API Server 2
```

This reduces the impact of a node failure.

---

# 83. Availability Zones

For EKS production:

```text
AZ-A
  |
  +--> Argo CD replica

AZ-B
  |
  +--> Argo CD replica

AZ-C
  |
  +--> Argo CD replica
```

The actual topology depends on cluster/node configuration.

The goal is to avoid a single-node or single-AZ control-plane dependency.

---

# 84. PodDisruptionBudget

A PodDisruptionBudget can help maintain availability during voluntary disruptions.

Conceptual example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: argocd-server
  namespace: argocd
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server
```

The actual labels must match the installed Argo CD version.

---

# 85. Network Policies

NetworkPolicies can restrict communication.

Conceptually:

```text
argocd-server
    |
    +--> allowed internal components

repo-server
    |
    +--> Git

controller
    |
    +--> Kubernetes APIs
```

Test carefully because over-restrictive policies can break Argo CD.

---

# 86. Argo CD Server Security

Harden:

```text
TLS
Authentication
RBAC
Ingress
Network access
Session settings
Administrative access
```

Avoid:

```text
Anonymous unrestricted administration
```

---

# 87. RBAC Configuration

Argo CD RBAC can be configured through its configuration resources.

A conceptual policy model:

```text
role:developer
  get applications

role:release-manager
  sync production applications

role:platform-admin
  manage Argo CD configuration
```

The exact syntax should be aligned with the installed Argo CD version and organizational identity model.

---

# 88. Least Privilege Example

Developer:

```text
get application
sync dev applications
```

Release manager:

```text
get application
sync qa/prod according to policy
```

Platform admin:

```text
manage projects
manage repositories
manage clusters
```

Use groups from SSO rather than manually maintaining large local-user lists.

---

# 89. Local Accounts

Local Argo CD accounts can be useful for:

```text
Break-glass access
Automation
Bootstrap
```

But they should not become an uncontrolled substitute for enterprise identity.

Protect:

```text
Tokens
Passwords
API credentials
```

---

# 90. Break-Glass Account

A production platform may retain a tightly controlled emergency account.

Requirements:

```text
Strong credential
MFA where supported by surrounding access model
Restricted storage
Audit logging
Periodic validation
Clear approval process
```

Do not use it for routine work.

---

# 91. Repository Access Hardening

Use:

```text
Dedicated deployment credential
Read-only access when possible
Short-lived credentials where supported
Rotation
Audit
```

If Argo CD only needs to read Git:

```text
Do not grant repository write permission.
```

---

# 92. GitOps Repository Write Access

CI may need write access to a GitOps repository to update image tags.

That creates a special security boundary.

For example:

```text
CI
 |
 | write
 v
GitOps repository
 |
 v
Argo CD
 |
 v
Production
```

Therefore CI's Git credentials can indirectly influence production.

Protect them accordingly.

---

# 93. Production Branch Protection

Require:

```text
Pull request
Required reviews
Status checks
Security checks
```

for production changes.

If CI automatically updates image tags, design a controlled automation path rather than bypassing all repository protections.

---

# 94. Argo CD Configuration as Code

Store important configuration declaratively:

```text
projects/
applications/
applicationsets/
rbac/
platform/
```

This provides:

```text
Review
Version control
Rollback
Audit
Reproducibility
```

---

# 95. Bootstrap Repository

A useful production repository can contain:

```text
bootstrap/
  argocd/
  projects/
  applicationsets/
```

Then:

```text
Terraform creates EKS
       |
       v
Argo CD installed
       |
       v
Bootstrap Application
       |
       v
Projects/ApplicationSets
       |
       v
Platform
       |
       v
Applications
```

---

# 96. Installation Validation Checklist

After installation:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get crd | grep argoproj
argocd version
argocd app list
argocd repo list
argocd cluster list
```

Then validate:

```text
UI
CLI
Git
Application
Sync
Health
```

---

# 97. Smoke Test Application

Create a simple test Application.

Example source:

```text
guestbook
```

or a small internal test application.

The goal is to validate:

```text
Git access
Manifest rendering
Kubernetes access
Sync
Health
```

before onboarding production applications.

---

# 98. Smoke Test Flow

```text
Git repository
      |
      v
Application
      |
      v
Repo Server
      |
      v
Manifest
      |
      v
Application Controller
      |
      v
Kubernetes
      |
      v
Healthy
```

If this works, the core control plane is functional.

---

# 99. Test Drift

After successful deployment, deliberately create a controlled test drift.

For example:

```bash
kubectl scale deployment <name> \
  -n <namespace> \
  --replicas=1
```

Then observe:

```text
OutOfSync
```

if the desired state differs.

This validates the GitOps control loop.

Only perform this in a non-production environment.

---

# 100. Test Self-Healing

If self-heal is configured:

```text
Manual change
     |
     v
Drift
     |
     v
Argo CD detects
     |
     v
Desired state restored
```

This is an excellent practical exercise.

---

# 101. Test Pruning

In a non-production environment:

```text
Remove resource from Git
       |
       v
Commit
       |
       v
Argo CD detects OutOfSync
       |
       v
Prune if enabled
```

This validates lifecycle ownership.

---

# 102. Test Repository Failure

In a controlled environment:

```text
Temporarily make repository inaccessible
```

Observe:

```text
Manifest retrieval failure
```

Then restore credentials/connectivity.

The objective is to understand failure behavior without affecting production.

---

# 103. Test Cluster Failure

For a non-production target cluster, simulate:

```text
API unreachable
```

Observe:

```text
Cluster connection failure
Application health/state impact
```

Restore connectivity and verify reconciliation resumes.

---

# 104. Production Verification Matrix

| Area | Validation |
|---|---|
| EKS | Nodes healthy |
| Argo CD Pods | Running |
| CRDs | Installed |
| CLI | Login works |
| UI | HTTPS works |
| DNS | Resolves |
| ALB | Healthy |
| Git | Connected |
| Cluster | Registered |
| Project | Valid |
| Application | Created |
| Sync | Works |
| Health | Healthy |
| Monitoring | Metrics/logs available |

---

# 105. Installation Troubleshooting: Pods Pending

Run:

```bash
kubectl describe pod <pod> -n argocd
```

Check events.

Possible causes:

- Insufficient CPU
- Insufficient memory
- Node taints
- Affinity rules
- Missing tolerations
- Resource quota

Then:

```bash
kubectl get nodes
kubectl describe node <node>
```

---

# 106. Installation Troubleshooting: CrashLoopBackOff

Run:

```bash
kubectl describe pod <pod> -n argocd
kubectl logs <pod> -n argocd
kubectl logs <pod> -n argocd --previous
```

Check:

```text
Configuration
Secrets
Environment
Permissions
Network
Resource limits
Version compatibility
```

---

# 107. Installation Troubleshooting: ImagePullBackOff

Run:

```bash
kubectl describe pod <pod> -n argocd
```

Check:

```text
Image name
Registry
Network
Image availability
Node runtime
```

For official Argo CD images, also verify the configured release/version and registry connectivity.

---

# 108. Installation Troubleshooting: UI Unavailable

Check:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get ingress -n argocd
```

Then:

```bash
kubectl describe ingress argocd-server -n argocd
```

Check:

```text
DNS
ALB
TLS certificate
Security group
Target health
Argo CD server
```

---

# 109. Installation Troubleshooting: ALB Exists but Unhealthy

Check:

```bash
kubectl get endpoints -n argocd argocd-server
```

and:

```bash
kubectl describe ingress argocd-server -n argocd
```

Then verify:

```text
Target type
Service port
Target port
Health check
Security groups
Network path
```

---

# 110. Installation Troubleshooting: CLI TLS Failure

Possible causes:

```text
Wrong hostname
Certificate mismatch
Self-signed certificate
Ingress TLS configuration
Incorrect port
Proxy
```

Do not solve production TLS problems by permanently disabling certificate verification.

Fix the certificate/hostname/trust configuration.

---

# 111. Installation Troubleshooting: Login Failure

Check:

```bash
argocd account get-user-info
```

after successful login.

If login fails:

```text
Verify endpoint
Verify credentials
Verify TLS
Verify SSO
Verify account status
Verify RBAC
```

---

# 112. Installation Troubleshooting: Repository Failed

Run:

```bash
argocd repo list
```

Then:

```bash
kubectl logs deployment/argocd-repo-server -n argocd
```

Check:

```text
URL
Credentials
Revision
SSH host key
Network
Certificate
Repository permissions
```

---

# 113. Installation Troubleshooting: Cluster Failed

Run:

```bash
argocd cluster list
```

Then investigate:

```text
Cluster endpoint
Authentication
Authorization
Network
EKS access
Kubernetes RBAC
```

Check the target cluster independently:

```bash
kubectl get nodes
```

using the appropriate administrative context.

---

# 114. Installation Troubleshooting: Application Not Found

Check:

```bash
kubectl get applications -A
```

If using a specific namespace:

```bash
kubectl get application <name> -n argocd
```

Potential causes:

- Wrong namespace
- Application not applied
- ApplicationSet not generated
- GitOps bootstrap not synchronized
- Name mismatch

---

# 115. Installation Troubleshooting: Application CRD Missing

Check:

```bash
kubectl get crd applications.argoproj.io
```

If missing, the Argo CD installation is incomplete or CRDs are unavailable.

Check:

```bash
kubectl get crd | grep argoproj
```

and installation logs/resources.

---

# 116. Installation Troubleshooting: Argo CD Server Healthy but Apps Not Syncing

This usually points away from the UI/API layer.

Investigate:

```text
Application Controller
Repo Server
Target cluster
Application configuration
Project
```

Commands:

```bash
argocd app get <app>
kubectl get pods -n argocd
```

---

# 117. Production Backup Strategy

Back up:

```text
Argo CD configuration
Applications
ApplicationSets
Projects
Repositories
RBAC
Cluster registrations
Secrets/credentials
```

But remember:

```text
Git repository
```

already contains much of the declarative desired state if the repository is properly structured.

---

# 118. What Git Does Not Automatically Protect

Git does not necessarily contain:

```text
Argo CD runtime credentials
Cluster connection secrets
Repository private keys
Local account secrets
Internal cache state
```

Therefore backup/recovery must address both:

```text
Git desired state
+
Argo CD control-plane configuration/secrets
```

---

# 119. Disaster Recovery Principle

A good DR design asks:

> If the Argo CD management cluster is destroyed, how quickly can we recreate it?

Ideal:

```text
Terraform
   |
   v
New EKS
   |
   v
Argo CD installation
   |
   v
Restore/bootstrap configuration
   |
   v
Git
   |
   v
Applications
```

This is why infrastructure and GitOps configuration must be reproducible.

---

# 120. DR Test

Do not assume backups work.

Periodically test:

```text
Create recovery environment
Install Argo CD
Restore configuration
Register clusters
Connect Git
Generate Applications
Verify workloads
```

A backup that has never been restored is an assumption, not a tested recovery mechanism.

---

# 121. Upgrade Strategy

Recommended:

```text
Development Argo CD
       |
       v
Upgrade
       |
       v
Application tests
       |
       v
QA Argo CD
       |
       v
Upgrade
       |
       v
Production Argo CD
```

Record:

```text
Version
Date
Change
Validation
Rollback plan
```

---

# 122. Upgrade Checklist

Before upgrading:

```text
Read release notes
Check Kubernetes compatibility
Check CRD changes
Check ApplicationSet behavior
Check plugins
Check Helm/Kustomize versions
Check SSO
Check ingress
Check monitoring
Check backup
```

Then test.

---

# 123. Rollback Plan for Argo CD Upgrade

A control-plane rollback is different from application rollback.

Application rollback:

```text
Git revision
```

Argo CD rollback:

```text
Control-plane version
```

Have a documented way to reinstall the previous Argo CD version if the upgrade fails.

---

# 124. Production Runbook: Initial Installation

```text
1. Confirm AWS account
2. Confirm EKS context
3. Verify kubectl
4. Verify Helm
5. Create argocd namespace
6. Install pinned Argo CD version
7. Verify Pods
8. Verify CRDs
9. Configure ingress
10. Configure TLS
11. Configure DNS
12. Configure authentication
13. Configure RBAC
14. Configure repositories
15. Register target clusters
16. Create Projects
17. Create test Application
18. Validate sync
19. Validate health
20. Configure monitoring
21. Document recovery
```

---

# 125. Production Runbook: New Target EKS Cluster

```text
1. Confirm AWS account
2. Confirm target cluster
3. Confirm cluster health
4. Confirm Argo CD connectivity
5. Configure cluster authentication
6. Register cluster
7. Verify argocd cluster list
8. Apply Project restrictions
9. Label cluster
10. Deploy test Application
11. Validate sync
12. Validate health
13. Add to ApplicationSet
```

---

# 126. Production Runbook: New Git Repository

```text
1. Create repository
2. Enable branch protection
3. Configure required reviews
4. Configure CI validation
5. Create Argo CD credential
6. Add repository
7. Run argocd repo list
8. Create Project sourceRepo restriction
9. Create test Application
10. Validate rendering
11. Validate deployment
```

---

# 127. Production Runbook: Argo CD Access

```text
1. DNS resolves
2. ALB healthy
3. TLS valid
4. Argo CD server healthy
5. SSO works
6. User group mapped
7. RBAC evaluated
8. Application access verified
```

---

# 128. Production Runbook: Security Review

Review:

```text
Who can access Argo CD?
Who can sync production?
Who can modify Projects?
Who can register clusters?
Who can add repositories?
Who can modify GitOps repository?
Who can access cluster credentials?
Who can change RBAC?
```

This should be reviewed periodically.

---

# 129. Production Runbook: Monitoring Review

Check:

```text
Prometheus targets
Grafana dashboards
ELK logs
Argo CD alerts
Application alerts
Cluster alerts
Repository failures
Sync failures
```

---

# 130. Production Architecture for User's Environment

A realistic environment:

```text
                          Developer
                              |
                              v
                      Application Git
                              |
                              v
                   Jenkins / GitHub Actions
                              |
                +-------------+-------------+
                |             |             |
                v             v             v
             Maven         Security       Tests
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          SonarQube          Trivy          Veracode
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
                +-------------+-------------+
                |             |             |
                v             v             v
             Helm         Kubernetes      ALB Ingress
                              |
                              v
                         RoboShop Pods
```

---

# 131. Terraform vs Argo CD

For the user's environment:

### Terraform

Manage:

```text
VPC
Subnets
Security groups
IAM
EKS
RDS
S3
ALB infrastructure
ECR
AWS resources
```

### Argo CD

Manage:

```text
Namespaces
Deployments
Services
Ingress
ConfigMaps
HPAs
Kubernetes policies
Helm applications
Platform workloads
```

This separation should be documented.

---

# 132. What Terraform Should Not Accidentally Own

Avoid having Terraform and Argo CD both independently manage the same Kubernetes resource.

Example:

```text
Terraform -> Deployment/cart
Argo CD   -> Deployment/cart
```

This creates competing desired states.

Prefer:

```text
Terraform -> infrastructure
Argo CD   -> application Kubernetes resources
```

---

# 133. What Argo CD Should Not Accidentally Own

Avoid using Argo CD as the sole owner of:

```text
AWS VPC
AWS IAM
AWS RDS
AWS S3
```

unless a deliberate Kubernetes controller/operator architecture is being used.

Argo CD primarily reconciles Kubernetes resources.

---

# 134. Installation Architecture for RoboShop

Recommended:

```text
Terraform
   |
   +--> AWS VPC
   +--> EKS
   +--> ECR
   +--> IAM
   |
   v
Argo CD
   |
   +--> Helm
   +--> RoboShop
   +--> Prometheus
   +--> Grafana
   +--> ELK integration
   |
   v
AWS ALB
```

This aligns responsibilities cleanly.

---

# 135. Production YAML: Argo CD Server Ingress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server
  namespace: argocd
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: <ACM_CERTIFICATE_ARN>
spec:
  ingressClassName: alb
  rules:
    - host: argocd.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
```

Replace:

```text
<ACM_CERTIFICATE_ARN>
```

with the actual certificate ARN.

Confirm the Argo CD server service port and ALB backend protocol in the target environment.

---

# 136. Production YAML: AppProject

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  description: RoboShop production applications

  sourceRepos:
    - https://github.com/example/roboshop-gitops.git

  destinations:
    - server: https://kubernetes.default.svc
      namespace: roboshop

  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: ""
      kind: Secret
    - group: autoscaling
      kind: HorizontalPodAutoscaler
    - group: networking.k8s.io
      kind: Ingress
```

In real production, Secret access and resource whitelists should be reviewed carefully.

---

# 137. Production YAML: Basic Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: environments/prod/cart

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

The next file will explain this manifest field by field.

---

# 138. Production YAML: Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: helm/roboshop

    helm:
      releaseName: cart

      valueFiles:
        - values/prod.yaml

      parameters:
        - name: image.tag
          value: "2026.08.19-abc123"

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

Use immutable image references in production.

---

# 139. Installation Validation Commands

Keep these commands available:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get ingress -n argocd
kubectl get applications -n argocd
kubectl get appprojects -n argocd
kubectl get applicationsets -n argocd
```

CLI:

```bash
argocd version
argocd app list
argocd repo list
argocd cluster list
argocd proj list
```

---

# 140. Important Command Reference

### Kubernetes

```bash
kubectl get pods -n argocd
kubectl describe pod <pod> -n argocd
kubectl logs <pod> -n argocd
kubectl get svc -n argocd
kubectl get ingress -n argocd
kubectl get applications -n argocd
kubectl get appprojects -n argocd
kubectl get applicationsets -n argocd
kubectl get events -n argocd
```

### Argo CD

```bash
argocd version
argocd login <host>
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app sync <app>
argocd app history <app>
argocd app rollback <app> <id>
argocd repo list
argocd cluster list
argocd proj list
```

---

# 141. Interview Questions

## Q1. How would you install Argo CD on EKS?

### Answer

> I first verify the AWS account, EKS context, kubectl and Helm access. I create the dedicated `argocd` namespace and install a pinned Argo CD version. Then I validate the control-plane Pods and CRDs. For production, I expose the API/UI through a controlled HTTPS ALB/Ingress, configure TLS, SSO and RBAC, add private Git repositories, register target clusters if using centralized multi-cluster management, create AppProjects, and deploy a test Application before onboarding production workloads.

---

## Q2. Would you expose Argo CD directly through a public LoadBalancer?

### Answer

> I would avoid unnecessary public exposure. For an enterprise environment, I would prefer a controlled access path such as an internal ALB, private DNS, corporate network access, HTTPS, SSO, and RBAC when the operational requirements allow it. If public access is required, I would still enforce TLS, strong authentication, least privilege, and network/security controls.

---

## Q3. Why use ALB for Argo CD in your architecture?

### Answer

> My EKS environment uses AWS ALB Ingress for external HTTP/HTTPS entry points. I would therefore expose Argo CD through the AWS Load Balancer Controller and ALB rather than introducing API Gateway into the architecture. DNS can point to the ALB and ACM can provide the TLS certificate.

---

## Q4. How do you secure the initial Argo CD admin account?

### Answer

> I use the initial administrator only for bootstrap. I configure enterprise authentication and RBAC, create appropriate individual/group-based access, validate it, and avoid relying on a shared bootstrap credential for routine operations. Any retained break-glass access is tightly controlled and audited.

---

## Q5. How do you register multiple EKS clusters?

### Answer

> I deploy Argo CD on a designated management cluster, establish secure authentication and authorization to each target EKS cluster, register the clusters, verify them with `argocd cluster list`, apply AppProject restrictions, and use Applications or ApplicationSets to select the correct cluster destinations.

---

## Q6. How do you avoid giving Argo CD unrestricted access to every cluster?

### Answer

> I use separate cluster credentials, Kubernetes RBAC, AppProjects, namespace restrictions, resource restrictions, AWS account boundaries, and where appropriate separate Argo CD instances for stronger isolation. Centralization should not mean unrestricted permissions.

---

## Q7. How do you troubleshoot an Argo CD installation?

### Answer

> I start with the control-plane Pods and services, then identify the failed component. I inspect events and logs. If the UI fails, I check the server, ingress, ALB, DNS and TLS. If manifest generation fails, I check the Repo Server and repository credentials. If applications stop reconciling, I check the Application Controller. For remote targets I also verify cluster connectivity and authorization.

---

# 142. Interview Scenario: Argo CD UI Works but Application Does Not Sync

Answer:

```text
UI/API is healthy
        |
        v
Check Application
        |
        v
Check sync status
        |
        v
argocd app diff
        |
        v
Check Repo Server
        |
        v
Check Application Controller
        |
        v
Check target cluster
```

Do not assume the UI being healthy means the reconciliation engine is healthy.

---

# 143. Interview Scenario: Repository Is Connected but Helm Application Fails

Investigate:

```text
Repository access
Helm chart path
Chart.yaml
values files
Helm parameters
targetRevision
template rendering
```

Useful commands:

```bash
argocd app get <app>
kubectl logs deployment/argocd-repo-server -n argocd
```

If necessary, reproduce Helm rendering independently in a controlled environment.

---

# 144. Interview Scenario: Production Application Became OutOfSync

Answer:

> I would not immediately synchronize it. I would first run `argocd app diff` and identify which resource and field changed. Then I would determine whether the change came from a manual `kubectl` operation, another Kubernetes controller, an operator, Helm behavior, or a Git change. If it is an unauthorized drift, I would restore Git as the source of truth and use self-healing or synchronization as appropriate.

---

# 145. Interview Scenario: Argo CD Management Cluster Is Lost

Answer:

> I would recreate the management infrastructure using Terraform or the organization's infrastructure automation, install the approved Argo CD version, restore required configuration and credentials, reconnect repositories and target clusters, and bootstrap Applications/ApplicationSets from Git. The ability to recreate the control plane without relying on manual undocumented steps is a key DR requirement.

---

# 146. Final Installation Mental Model

Remember this sequence:

```text
AWS Account
    |
    v
EKS
    |
    v
argocd namespace
    |
    v
Argo CD installation
    |
    +--> API Server
    +--> Repo Server
    +--> Application Controller
    +--> Redis
    +--> ApplicationSet Controller
    |
    v
Ingress + TLS + DNS
    |
    v
Authentication + RBAC
    |
    v
Repositories
    |
    v
Clusters
    |
    v
Projects
    |
    v
Applications
    |
    v
GitOps workloads
```

---

# 147. Production Readiness Checklist

Before declaring Argo CD production-ready:

```text
[ ] EKS cluster verified
[ ] Argo CD version pinned
[ ] Dedicated namespace
[ ] Control-plane Pods healthy
[ ] CRDs verified
[ ] ALB/Ingress configured
[ ] TLS configured
[ ] DNS configured
[ ] SSO configured
[ ] RBAC configured
[ ] Initial admin controlled
[ ] Git repository connected
[ ] Repository credentials secured
[ ] Target clusters registered
[ ] AppProjects configured
[ ] Applications tested
[ ] Sync tested
[ ] Drift tested
[ ] Self-healing tested if enabled
[ ] Monitoring configured
[ ] Logging configured
[ ] Alerts configured
[ ] Backup documented
[ ] DR tested
[ ] Upgrade process documented
[ ] Runbooks documented
```

---

# 148. Key Takeaways

1. Argo CD installation is only the beginning of production readiness.

2. Always verify the AWS account and Kubernetes context before modifying EKS.

3. Pin the Argo CD version rather than relying on an unversioned installation.

4. Use a dedicated `argocd` namespace.

5. Understand the difference between:
   - API Server
   - Repo Server
   - Application Controller
   - ApplicationSet Controller
   - Redis

6. Use HTTPS and controlled ingress for production.

7. In the user's architecture, use AWS ALB Ingress rather than API Gateway.

8. Use SSO and RBAC rather than shared administrator credentials.

9. Treat Git repository credentials and cluster credentials as highly sensitive.

10. Use AppProjects to restrict repository, cluster, namespace, and resource access.

11. Centralized Argo CD can manage multiple EKS clusters, but security boundaries and blast radius must be designed carefully.

12. CI should build, test, scan, publish images, and update GitOps state; Argo CD should reconcile Kubernetes state.

13. Production rollback should normally result in a corresponding Git change.

14. Backups must include more than Git because runtime credentials and Argo CD configuration may not be stored there.

15. A production Argo CD platform must have monitoring, alerting, upgrade, and disaster-recovery procedures.

---