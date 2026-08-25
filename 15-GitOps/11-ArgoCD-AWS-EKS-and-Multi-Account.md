# ArgoCD-AWS-EKS-and-Multi-Account

## 1. Purpose

This file connects Argo CD GitOps with a production AWS environment.

The focus is not simply:

```text
Argo CD -> Kubernetes
```

but:

```text
AWS Accounts
      |
      v
VPC / Networking
      |
      v
EKS
      |
      v
Kubernetes authentication
      |
      v
Kubernetes RBAC
      |
      v
Argo CD
      |
      v
Applications
```

The practical target architecture is the user's RoboShop platform using:

- AWS
- EKS
- ECR
- Terraform
- Kubernetes
- Helm
- Argo CD
- Jenkins / GitHub Actions
- SonarQube
- Trivy
- Veracode
- Prometheus
- Grafana
- ELK
- AWS ALB Ingress

This file covers:

- EKS architecture
- AWS account boundaries
- IAM
- EKS access
- Kubernetes RBAC
- IRSA
- EKS Pod Identity concepts
- Private EKS API
- VPC networking
- Transit Gateway
- Cross-account access
- Multi-region architecture
- ECR
- AWS Load Balancer Controller
- ALB Ingress
- Terraform + Argo CD boundaries
- Production YAMLs
- Security
- DR
- Troubleshooting
- Interview preparation

---

# 2. Complete AWS GitOps Mental Model

A production deployment can be viewed as several layers:

```text
Layer 1
AWS Organization / Accounts

Layer 2
VPC / Networking / IAM

Layer 3
EKS Cluster

Layer 4
Kubernetes Platform

Layer 5
Argo CD

Layer 6
ApplicationSet / Applications

Layer 7
RoboShop Workloads
```

Each layer has a different owner and responsibility.

---

# 3. Responsibility Separation

A clean architecture is:

```text
Terraform
   |
   +--> AWS infrastructure
   +--> VPC
   +--> EKS
   +--> IAM
   +--> Security Groups
   +--> RDS
   +--> ECR
   +--> Route 53
   |
   v
EKS
   |
   +--> Kubernetes platform
   |
   v
Argo CD
   |
   +--> Kubernetes applications
```

This prevents infrastructure and application controllers from competing.

---

# 4. Why This Separation Matters

Consider an EKS cluster.

Terraform may manage:

```text
EKS cluster
Node groups
VPC
Subnets
IAM
Security groups
ECR
RDS
```

Argo CD may manage:

```text
Namespace
Deployment
Service
Ingress
ConfigMap
HPA
Application configuration
```

If both tools manage the same object, they can overwrite each other's changes.

---

# 5. AWS Account Strategy

A production organization may use:

```text
AWS Organization
|
+-- Shared/Management Account
|
+-- Dev Account
|
+-- QA Account
|
+-- Production Account
|
+-- Security Account
|
+-- Logging Account
```

The exact organization structure varies.

The important principle is:

```text
Production should have stronger isolation than development.
```

---

# 6. Example Multi-Account Architecture

```text
                 AWS Organization
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
   DEV Account       QA Account      PROD Account
       |                |                |
    EKS DEV          EKS QA          EKS PROD
```

Central Argo CD may operate from:

```text
Platform/Management Account
```

if the organization accepts the security and networking model.

---

# 7. Why Separate AWS Accounts?

Separate accounts provide:

```text
Blast-radius reduction
Billing isolation
IAM isolation
Security boundaries
Production protection
Compliance boundaries
```

An accidental development change should not automatically have production permissions.

---

# 8. Management Account vs Workload Account

A management/platform account can host:

```text
Shared tooling
CI/CD
Argo CD
Security tooling
Central observability
```

Workload accounts host:

```text
EKS
RDS
Application resources
ECR
```

The exact division depends on enterprise architecture.

---

# 9. Central Argo CD Multi-Account Architecture

```text
                 Git Repository
                       |
                       v
                Central Argo CD
                       |
             +---------+---------+
             |         |         |
             v         v         v
          DEV AWS    QA AWS    PROD AWS
             |         |         |
          EKS DEV   EKS QA    EKS PROD
```

This is a hub-and-spoke control-plane architecture.

---

# 10. Network Connectivity

Central Argo CD must communicate with the Kubernetes APIs of target clusters.

Possible AWS designs include:

```text
VPC Peering
Transit Gateway
PrivateLink-related designs
Shared networking
Controlled public EKS API access
```

The preferred design depends on:

```text
Scale
Security
Network topology
Account structure
Compliance
```

---

# 11. Private EKS API

A production EKS cluster may use a private Kubernetes API endpoint.

Architecture:

```text
Argo CD VPC
      |
      v
Transit Gateway / private routing
      |
      v
Production VPC
      |
      v
Private EKS API
```

Benefits include:

```text
Reduced public exposure
Private network path
Stronger network controls
```

---

# 12. Public EKS API

EKS can also expose a public Kubernetes endpoint.

If used:

```text
Restrict allowed CIDRs
Use strong authentication
Use network controls
Monitor access
```

Avoid unnecessarily allowing:

```text
0.0.0.0/0
```

to the Kubernetes API.

---

# 13. EKS Endpoint Modes

The EKS Kubernetes API endpoint can be configured using public/private access settings.

Typical concepts:

```text
Public only
Private only
Public + private
```

For centralized production Argo CD, private connectivity can be attractive when the networking architecture supports it.

---

# 14. Private Endpoint Failure

If Argo CD cannot reach a private EKS endpoint, investigate:

```text
VPC routes
Transit Gateway
Security groups
DNS
Resolver
Network ACLs
```

Do not immediately modify Kubernetes RBAC.

Network problems occur before authorization.

---

# 15. Network Troubleshooting Layers

Use this order:

```text
1. DNS
2. Network route
3. TCP connectivity
4. TLS
5. Authentication
6. Authorization
7. Kubernetes resource
```

This prevents jumping to the wrong layer.

---

# 16. AWS VPC Architecture

Example:

```text
Production VPC
|
+-- Private Subnet AZ-a
|      |
|      +-- EKS nodes
|
+-- Private Subnet AZ-b
|      |
|      +-- EKS nodes
|
+-- Private Subnet AZ-c
       |
       +-- EKS nodes
```

The Kubernetes API endpoint is managed by AWS.

---

# 17. EKS Worker Networking

RoboShop pods run on EKS worker nodes.

Depending on the networking model, pod IPs are integrated into the VPC networking model.

This means:

```text
VPC design
Subnet capacity
Security groups
Routing
```

can affect Kubernetes workload scalability.

---

# 18. EKS and Argo CD Communication

Conceptually:

```text
Argo CD Application Controller
            |
            v
      Kubernetes API
            |
            v
          EKS
```

Argo CD does not need SSH access to EKS worker nodes.

It communicates with the Kubernetes API.

---

# 19. No Node SSH Required

For normal GitOps deployment:

```text
Argo CD
   |
   v
Kubernetes API
```

not:

```text
Argo CD
   |
   v
SSH
   |
   v
Worker node
```

This is an important security advantage.

---

# 20. IAM

AWS IAM controls:

```text
Who can call AWS APIs
```

Examples:

```text
eks:DescribeCluster
ecr:GetAuthorizationToken
sts:AssumeRole
```

IAM is not a replacement for Kubernetes RBAC.

---

# 21. Kubernetes RBAC

Kubernetes RBAC controls:

```text
Who can access Kubernetes resources
```

Examples:

```text
get deployments
create services
update configmaps
delete pods
```

The final authorization decision is made inside Kubernetes.

---

# 22. IAM + EKS + Kubernetes RBAC

A useful mental model:

```text
AWS IAM
   |
   v
EKS Authentication
   |
   v
Kubernetes Identity
   |
   v
Kubernetes RBAC
   |
   v
Allowed Resource
```

Authentication gets the identity into Kubernetes.

RBAC determines permissions.

---

# 23. EKS Access Concepts

EKS supports AWS-integrated access mechanisms.

Depending on the EKS configuration and version, organizations may use:

```text
EKS access entries
```

and/or legacy:

```text
aws-auth ConfigMap
```

The recommended approach for new designs should follow the current AWS/EKS guidance for the cluster version and access mode.

---

# 24. Access Entries

EKS access entries provide a managed way to associate AWS identities with EKS cluster access.

Conceptually:

```text
IAM principal
      |
      v
EKS access entry
      |
      v
Kubernetes permissions
```

This reduces reliance on manually editing authentication configuration.

---

# 25. Legacy aws-auth

Older EKS environments commonly use:

```text
aws-auth ConfigMap
```

to map IAM roles/users to Kubernetes identities.

Example concept:

```yaml
mapRoles:
  - rolearn: arn:aws:iam::<ACCOUNT>:role/<ROLE>
    username: <identity>
    groups:
      - system:masters
```

Avoid granting:

```text
system:masters
```

unless truly necessary.

---

# 26. Production Recommendation

For new EKS designs:

```text
Evaluate EKS access entries first.
```

For existing clusters:

```text
Understand current aws-auth configuration
before changing authentication.
```

Do not modify access mechanisms during a production incident without understanding the blast radius.

---

# 27. Argo CD Cluster Identity

Argo CD needs a Kubernetes identity that can perform its required operations.

Conceptually:

```text
Argo CD
   |
   v
Kubernetes API
   |
   v
Argo CD identity
   |
   v
RBAC
```

The identity can be represented through the cluster credentials/configuration Argo CD uses.

---

# 28. Least Privilege Argo CD

If RoboShop only needs:

```text
Deployment
Service
Ingress
ConfigMap
Secret
HPA
```

do not automatically assume it needs:

```text
ClusterRole
CRD
Node
Namespace
```

unless the deployment actually manages those resources.

---

# 29. Cluster-Scoped Platform Resources

Some platform applications require cluster-scoped resources.

Examples:

```text
AWS Load Balancer Controller
CRDs
ClusterRoles
ClusterRoleBindings
IngressClass
```

Therefore platform Argo CD permissions may need to be broader than a business application Argo CD Project.

---

# 30. Platform vs Application Permissions

A useful separation:

```text
Platform Argo CD Project
   |
   +--> cluster-scoped resources

RoboShop Application Project
   |
   +--> namespace-scoped resources
```

This reduces application-team privileges.

---

# 31. IRSA

IRSA means:

```text
IAM Roles for Service Accounts
```

It allows a Kubernetes ServiceAccount to assume an AWS IAM role.

Conceptually:

```text
Pod
 |
 v
ServiceAccount
 |
 v
IAM Role
 |
 v
AWS API
```

This avoids putting static AWS credentials into pods.

---

# 32. Why IRSA Matters

A RoboShop pod may need AWS access.

For example:

```text
Application
    |
    v
AWS API
```

Instead of:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside a Kubernetes Secret, use workload identity.

---

# 33. EKS Pod Identity

AWS also provides EKS Pod Identity as a newer mechanism for associating IAM permissions with Kubernetes workloads.

Conceptually:

```text
Pod
 |
 v
ServiceAccount
 |
 v
EKS Pod Identity
 |
 v
IAM Role
 |
 v
AWS API
```

For new designs, compare EKS Pod Identity and IRSA against:

```text
EKS version
AWS recommendations
Application requirements
Existing platform standards
```

---

# 34. IRSA vs EKS Pod Identity

Both solve:

```text
Pod -> AWS permissions
```

The implementation differs.

IRSA uses:

```text
OIDC + IAM role trust
```

EKS Pod Identity uses:

```text
EKS-managed pod identity association
```

The right choice depends on the organization's EKS architecture.

---

# 35. Argo CD Does Not Need Pod IAM for Kubernetes API Access

Do not confuse:

```text
Argo CD -> Kubernetes API
```

with:

```text
Application Pod -> AWS API
```

These are different identity paths.

Argo CD needs Kubernetes authorization.

RoboShop pods may need AWS IAM permissions.

---

# 36. Example: ALB Controller Identity

AWS Load Balancer Controller needs AWS permissions to create/manage:

```text
ALB
Target Groups
Security Group-related resources
Listeners
```

It should use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

rather than static credentials.

---

# 37. Example: RoboShop Pod Identity

If a RoboShop service needs S3:

```text
RoboShop Pod
    |
    v
ServiceAccount
    |
    v
IAM Role
    |
    v
S3
```

Argo CD deploys the ServiceAccount and workload configuration.

AWS IAM owns the permission.

---

# 38. ECR Architecture

CI builds an image:

```text
Docker build
     |
     v
Security scanning
     |
     v
ECR
```

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart:2026.08.19-abc123
```

Argo CD does not build the image.

---

# 39. CI + ECR + GitOps

Production flow:

```text
Developer
   |
   v
Git
   |
   v
Jenkins/GitHub Actions
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
```

---

# 40. Immutable Image Tags

Avoid:

```text
latest
```

for production deployments.

Prefer:

```text
2026.08.19-abc123
```

or:

```text
git-abc123
```

or another immutable release identifier.

---

# 41. Why Immutable Tags Matter

With:

```text
latest
```

the same Git configuration may point to a different image over time.

That weakens:

```text
Auditability
Rollback
Reproducibility
Incident analysis
```

Use immutable tags or immutable digests.

---

# 42. Image Digest

An even stronger reference is:

```text
image@sha256:<digest>
```

A digest uniquely identifies image content.

Production promotion can use:

```text
same digest
```

across environments.

---

# 43. Image Promotion

A strong promotion model:

```text
Build once
   |
   v
Scan once
   |
   v
Push image
   |
   v
DEV
   |
   v
QA
   |
   v
PROD
```

The artifact does not change between environments.

Only desired deployment configuration changes.

---

# 44. ECR Permissions

The Kubernetes workload may need permission to pull images.

Depending on the EKS node/runtime configuration, image pulls can use the node IAM role or other supported ECR authentication mechanisms.

Do not unnecessarily give application pods broad ECR permissions.

---

# 45. ECR and Cross-Account Deployment

If:

```text
ECR in shared account
```

and:

```text
EKS in production account
```

then configure appropriate ECR repository access.

Conceptually:

```text
EKS PROD
   |
   v
ECR PROD/SHARED
```

Use repository policies and IAM least privilege.

---

# 46. ECR Cross-Region

For DR:

```text
ECR ap-south-1
        |
        v
ECR replica / second region
        |
        v
EKS DR
```

Using regional image availability can reduce dependency on a single region.

The exact replication strategy should follow AWS ECR capabilities and organizational requirements.

---

# 47. ECR Failure During Deployment

If EKS cannot pull the image:

```text
Pod -> ImagePullBackOff
```

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

```text
authentication
repository not found
tag not found
network
permissions
```

Argo CD may report the Deployment as:

```text
Progressing
Degraded
```

depending on resource health.

---

# 48. AWS Load Balancer Controller

For the user's architecture:

```text
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

The controller watches Kubernetes resources and creates/manages AWS load-balancing resources.

---

# 49. Why Argo CD Does Not Directly Create the ALB

Argo CD applies:

```yaml
kind: Ingress
```

The AWS Load Balancer Controller observes it.

The controller then reconciles:

```text
Ingress
   |
   v
ALB
Target Groups
Listeners
```

This is controller-based infrastructure reconciliation.

---

# 50. ALB Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

In production, add the appropriate TLS, health-check, security and routing annotations.

---

# 51. ALB Controller IAM

The AWS Load Balancer Controller requires AWS permissions.

Use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

rather than embedding credentials.

---

# 52. Terraform + ALB Controller Boundary

Terraform may provision:

```text
VPC
Subnet
Security infrastructure
EKS
IAM role
```

Argo CD may deploy:

```text
AWS Load Balancer Controller
Ingress
```

The controller creates runtime AWS resources from Kubernetes declarations.

Define ownership carefully to avoid Terraform and controller conflicts.

---

# 53. Route 53

A production architecture may use:

```text
Route 53
   |
   v
ALB
   |
   v
Ingress
```

Terraform can manage:

```text
Hosted zones
DNS records
```

or another approved automation system can own them.

---

# 54. ACM

TLS certificates may be managed through:

```text
AWS Certificate Manager
```

Argo CD can manage Kubernetes Ingress references/annotations where appropriate.

Terraform may manage:

```text
ACM certificate
validation
DNS
```

Again, define a single owner for each resource.

---

# 55. EKS + ALB + Argo CD

Complete request path:

```text
Client
 |
 v
Route 53
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
```

Deployment control path:

```text
Git
 |
 v
Argo CD
 |
 v
Ingress/Service/Deployment
 |
 v
Kubernetes
 |
 v
AWS Load Balancer Controller
 |
 v
ALB
```

---

# 56. Multi-Account ALB

Each EKS cluster generally has its own AWS networking context.

Example:

```text
DEV Account
   |
   +--> ALB DEV

PROD Account
   |
   +--> ALB PROD
```

Central Argo CD manages Kubernetes Ingress objects, while the local AWS controller manages AWS load balancers.

---

# 57. Production EKS Platform Components

A typical cluster may include:

```text
AWS Load Balancer Controller
External Secrets
Metrics Server
Prometheus
Grafana
Logging agents
Argo CD-managed platform resources
```

The exact stack depends on platform standards.

---

# 58. Platform Bootstrap

A common flow:

```text
Terraform
   |
   v
EKS
   |
   v
Argo CD
   |
   v
Platform Applications
   |
   +--> ALB Controller
   +--> External Secrets
   +--> Monitoring
   +--> Logging
   |
   v
RoboShop Applications
```

---

# 59. Bootstrap Chicken-and-Egg

Question:

```text
How does Argo CD get installed if Argo CD is supposed to install everything?
```

Answer:

Use a bootstrap layer such as:

```text
Terraform
Helm
Cloud-init
Management tooling
```

to install initial Argo CD.

After that:

```text
Argo CD manages Argo CD-related configuration
```

through an approved bootstrap pattern.

---

# 60. Terraform Installing Argo CD

Terraform can install Argo CD using:

```text
Helm provider
```

or other supported mechanisms.

Then GitOps can take ownership of subsequent configuration.

Do not allow two systems to continuously manage the same Helm release without a deliberate ownership model.

---

# 61. App of Apps Bootstrap

A common architecture:

```text
Terraform
   |
   v
Install Argo CD
   |
   v
platform-root Application
   |
   +--> Projects
   +--> ApplicationSets
   +--> Platform controllers
   +--> Applications
```

This gives GitOps a controlled entry point.

---

# 62. EKS Cluster Bootstrap Repository

Example:

```text
gitops/
├── bootstrap/
│   ├── root-application.yaml
│   └── projects/
│
├── applicationsets/
│   ├── platform.yaml
│   └── roboshop.yaml
│
├── platform/
│   ├── alb-controller/
│   ├── monitoring/
│   └── external-secrets/
│
└── applications/
    └── roboshop/
```

---

# 63. Multi-Account Repository Strategy

Option:

```text
gitops/
├── accounts/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── clusters/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── applications/
```

This separates account-level and cluster-level intent.

---

# 64. Environment Values

For RoboShop:

```text
values/
├── dev/
├── qa/
└── prod/
```

Example:

```text
values/prod/cart.yaml
values/prod/payment.yaml
```

Use Git review to control changes.

---

# 65. Production Git Repository Example

```text
roboshop-gitops/
│
├── bootstrap/
│
├── projects/
│
├── applicationsets/
│
├── clusters/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── alb-controller/
│   ├── monitoring/
│   ├── logging/
│   └── secrets/
│
├── charts/
│   ├── cart/
│   ├── user/
│   ├── catalogue/
│   ├── payment/
│   └── frontend/
│
└── values/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 66. Secrets Architecture

Do not store plaintext AWS credentials in Git.

Avoid:

```yaml
AWS_SECRET_ACCESS_KEY: ...
```

in a normal Git repository.

Use:

```text
AWS IAM
EKS Pod Identity
IRSA
External Secrets
AWS Secrets Manager
```

as appropriate.

---

# 67. External Secrets Pattern

A common architecture:

```text
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
RoboShop Pod
```

Argo CD can manage:

```text
ExternalSecret
```

and related configuration.

---

# 68. Secret Ownership

A strong ownership model:

```text
Secrets Manager
   |
   +--> secret data

GitOps
   |
   +--> ExternalSecret definition

Kubernetes
   |
   +--> generated Secret
```

Git stores references/configuration, not plaintext secret values.

---

# 69. AWS Secrets Manager Permissions

The workload or External Secrets controller needs AWS permissions.

Use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

with least privilege.

Example permission concept:

```text
secretsmanager:GetSecretValue
```

for only the required secret ARN(s).

---

# 70. Multi-Account Secrets

Production may use:

```text
DEV Secrets Manager
QA Secrets Manager
PROD Secrets Manager
```

Each cluster accesses only its environment's secrets.

This prevents:

```text
DEV workload -> PROD secrets
```

through accidental IAM permissions.

---

# 71. Cross-Account Secrets

If a centralized secrets account is used:

```text
EKS workload
    |
    v
IAM role
    |
    v
Cross-account access
    |
    v
Secrets Manager
```

Use explicit resource policies and IAM conditions.

---

# 72. Production Security Principle

Do not solve:

```text
cross-account access
```

by granting:

```text
AdministratorAccess
```

Instead define the smallest permission required.

---

# 73. Multi-Account ECR Security

A target EKS cluster should only pull images it needs.

Use:

```text
ECR repository policy
IAM
```

to restrict access.

Avoid broad:

```text
ecr:*
```

when narrower permissions are sufficient.

---

# 74. ECR Repository Naming

Example:

```text
roboshop/cart
roboshop/user
roboshop/payment
```

or:

```text
roboshop-cart
roboshop-user
roboshop-payment
```

Use a consistent organizational naming standard.

---

# 75. Image Lifecycle

ECR lifecycle policies can clean old images.

Keep enough versions for:

```text
Rollback
Incident analysis
Compliance
```

Do not immediately delete every old image.

---

# 76. Production Image Promotion

Example:

```text
cart:git-a1b2c3
```

is tested in:

```text
DEV
```

Then the exact same image is promoted to:

```text
QA
```

and:

```text
PROD
```

Git changes:

```text
environment configuration
```

not:

```text
source code artifact
```

---

# 77. Multi-Account Image Strategy

Option A:

```text
ECR per account
```

Option B:

```text
Central ECR
+
cross-account repository access
```

Option C:

```text
ECR replicated across regions/accounts
```

Select based on:

```text
Security
Latency
DR
Cost
Operational simplicity
```

---

# 78. AWS IAM Role Assumption

Cross-account architecture often uses:

```text
Management Role
       |
       v
sts:AssumeRole
       |
       v
Target Account Role
```

Trust policies must explicitly allow the intended principal.

---

# 79. Trust Policy Principle

A role trust policy answers:

```text
Who can assume this role?
```

The role permission policy answers:

```text
What can the role do?
```

Do not confuse these two.

---

# 80. Cross-Account Failure

If Argo CD-related AWS access fails:

```text
AccessDenied
```

check:

```text
Source identity
Target role trust policy
Target role permissions
External ID/conditions if used
Region
Account
```

---

# 81. AWS STS Troubleshooting

Run:

```bash
aws sts get-caller-identity
```

Then verify the expected role:

```text
arn:aws:iam::<ACCOUNT>:role/<ROLE>
```

If the role is wrong:

```text
Everything downstream may fail.
```

---

# 82. EKS Cluster Identity Troubleshooting

Check:

```bash
aws eks describe-cluster \
  --name <cluster> \
  --region <region>
```

Review:

```text
endpoint
certificateAuthority
resourcesVpcConfig
accessConfig
status
```

---

# 83. EKS API Endpoint Debugging

If endpoint is private:

```text
Can the Argo CD network resolve it?
Can it route to it?
Can security groups allow it?
```

If public:

```text
Is Argo CD's source CIDR allowed?
```

---

# 84. Kubernetes Authorization Debugging

Where appropriate, use:

```bash
kubectl auth can-i get deployments -n roboshop
```

and:

```bash
kubectl auth can-i create deployments -n roboshop
```

The exact identity must be the same identity being tested.

---

# 85. Resource-Level Authorization

Test the specific resource:

```bash
kubectl auth can-i create ingress -n roboshop
kubectl auth can-i update deployment -n roboshop
kubectl auth can-i create service -n roboshop
```

This is better than only checking:

```text
cluster-admin
```

---

# 86. Production RBAC Example

A namespace-scoped Role might look like:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: roboshop-deployer
  namespace: roboshop

rules:
  - apiGroups: ["apps"]
    resources:
      - deployments
      - replicasets
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete

  - apiGroups: [""]
    resources:
      - services
      - configmaps
      - secrets
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete

  - apiGroups: ["networking.k8s.io"]
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
```

This is illustrative; the actual Argo CD permissions must include every required resource and any cluster-scoped resources your deployment uses.

---

# 87. RoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: roboshop-deployer
  namespace: roboshop

subjects:
  - kind: ServiceAccount
    name: argocd-deployer
    namespace: argocd

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: roboshop-deployer
```

This is an illustrative model. Production Argo CD installations should use the supported cluster credential and service-account configuration for the installed Argo CD version.

---

# 88. ClusterRole Example

Cluster-scoped resources require a ClusterRole.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-deployer

rules:
  - apiGroups:
      - apiextensions.k8s.io
    resources:
      - customresourcedefinitions
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
```

Do not grant this to application teams unless required.

---

# 89. EKS Access Entry Concept

The exact AWS CLI syntax depends on the cluster configuration and AWS CLI version.

Conceptually:

```text
Create EKS access entry
       |
       v
Associate approved policy/access
       |
       v
IAM principal can authenticate
       |
       v
Kubernetes authorization
```

Always use current AWS documentation for exact production commands.

---

# 90. EKS Access Entry Design

Separate roles:

```text
PlatformAdminRole
DeploymentRole
ReadOnlyRole
BreakGlassRole
```

Do not use one identity for all operations.

---

# 91. Argo CD Management Role

A dedicated role may be used for:

```text
Argo CD target cluster access
```

It should be:

```text
Dedicated
Audited
Least privilege
Non-human
Rotatable
```

---

# 92. Human vs Machine Identity

Human:

```text
Developer
Platform Engineer
Operator
```

Machine:

```text
Argo CD
CI
External Secrets
ALB Controller
```

Use separate identities.

Never reuse a human admin credential for Argo CD automation.

---

# 93. Argo CD vs CI Cluster Access

A strong GitOps architecture does not require CI to directly deploy application manifests.

CI:

```text
Build
Test
Scan
Push image
Update Git
```

Argo CD:

```text
Read Git
Deploy to EKS
Reconcile
```

This reduces CI cluster credentials.

---

# 94. Security Advantage

Without GitOps:

```text
Jenkins
   |
   v
Kubernetes API
```

CI requires cluster credentials.

With GitOps:

```text
Jenkins
   |
   v
Git
   |
   v
Argo CD
   |
   v
Kubernetes API
```

CI does not need direct production cluster deployment permissions.

---

# 95. RoboShop Security Flow

```text
Developer
   |
   v
Git
   |
   v
Jenkins/GitHub Actions
   |
   +--> Security checks
   |
   v
GitOps PR
   |
   v
Argo CD
   |
   v
EKS
```

This creates a stronger separation of duties.

---

# 96. Production Approval

A production promotion can require:

```text
CI success
+
Security scan success
+
Code review
+
GitOps PR approval
+
Production sync approval
```

This is stronger than:

```text
Jenkins -> kubectl apply
```

---

# 97. Multi-Account Production Flow

```text
                    GitOps Repository
                           |
                           v
                    Central Argo CD
                           |
              +------------+------------+
              |            |            |
              v            v            v
          DEV Account   QA Account   PROD Account
              |            |            |
           EKS DEV      EKS QA       EKS PROD
              |            |            |
           RoboShop     RoboShop      RoboShop
```

---

# 98. Multi-Region Production Flow

```text
                     Central Argo CD
                            |
              +-------------+-------------+
              |                           |
              v                           v
         ap-south-1                 ap-southeast-1
              |                           |
          EKS PROD                    EKS DR
              |                           |
          RoboShop                    RoboShop
```

---

# 99. Regional ECR

For regional resilience:

```text
ECR Region A
     |
     v
ECR Region B
```

Then:

```text
EKS Region A -> ECR Region A
EKS Region B -> ECR Region B
```

This reduces dependency on a single-region registry endpoint.

---

# 100. Disaster Recovery Layers

Do not treat DR as one thing.

There are at least four layers:

```text
1. AWS infrastructure
2. Kubernetes platform
3. Application configuration
4. Application data
```

GitOps primarily helps with:

```text
Kubernetes platform/application desired state
```

while Terraform and AWS services handle other layers.

---

# 101. DR Example

```text
Primary Region
   |
   +--> EKS
   +--> RDS
   +--> ECR
   |
   v
Replication/Backup
   |
   v
DR Region
   |
   +--> EKS DR
   +--> RDS recovery
   +--> ECR availability
```

Argo CD then reconciles applications into the DR cluster.

---

# 102. DR Runbook

```text
1. Detect regional failure.
2. Activate DR decision process.
3. Confirm DR EKS availability.
4. Confirm ECR images.
5. Confirm secrets.
6. Confirm database recovery.
7. Confirm networking.
8. Confirm Argo CD cluster connectivity.
9. Enable/target DR ApplicationSets.
10. Sync applications.
11. Validate ALB.
12. Validate application health.
13. Validate monitoring.
14. Route traffic.
```

---

# 103. DR Testing

Do not only write:

```text
DR runbook exists.
```

Actually test:

```text
Cluster recovery
Argo CD recovery
ECR recovery
Secrets recovery
Database recovery
ALB recovery
DNS failover
Application startup
```

---

# 104. RTO and RPO

RTO:

```text
How quickly must service recover?
```

RPO:

```text
How much data loss is acceptable?
```

GitOps configuration usually has a strong recovery posture because Git provides version history.

Application data has separate RPO requirements.

---

# 105. Production Monitoring

Monitor:

```text
Argo CD
EKS
AWS Load Balancer Controller
ECR
Application pods
Kubernetes API
```

For the user's stack:

```text
Prometheus
Grafana
ELK
```

are used for monitoring/logging.

---

# 106. Argo CD Metrics

Monitor:

```text
Application health
Sync status
Controller errors
Reconciliation latency
Cluster connection status
Repository failures
```

---

# 107. EKS Metrics

Monitor:

```text
Node CPU
Node memory
Pod CPU
Pod memory
Pod restarts
Pending pods
API server health
Cluster capacity
```

---

# 108. ECR Monitoring

Monitor:

```text
Image availability
Image pull failures
Repository growth
Lifecycle cleanup
Security findings
```

The exact vulnerability scanning implementation should follow the organization's ECR/security standards.

---

# 109. ALB Monitoring

Monitor:

```text
HTTP 4xx
HTTP 5xx
Target health
Latency
Request count
Unhealthy targets
```

Prometheus/Grafana can complement AWS-native metrics.

---

# 110. ELK Logs

Centralized logs should include:

```text
account
cluster
namespace
pod
service
environment
region
```

This allows:

```text
PROD
+
EKS-PROD-02
+
payment
```

to be isolated quickly.

---

# 111. Production Incident: ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Then verify:

```text
ECR repository
Image tag
Image digest
IAM permissions
Network
Node role
```

Do not assume Argo CD is broken because the Application is Degraded.

---

# 112. Production Incident: ALB Not Created

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Then inspect:

```text
AWS Load Balancer Controller pods
Controller logs
IAM permissions
subnet tags
security groups
IngressClass
annotations
```

---

# 113. Production Incident: Ingress Exists but ALB Unhealthy

Check:

```text
Target group health
Service
Endpoints
Pod readiness
Health-check path
Security groups
```

Argo CD may correctly report the Ingress as synced while the application is still unhealthy.

---

# 114. Production Incident: Argo CD Synced but Application Broken

Important distinction:

```text
Synced
```

means desired manifests match Git/live resource definitions.

It does not always mean:

```text
Application is serving users correctly.
```

Check:

```text
Health
Pods
Services
Ingress
ALB
Dependencies
Logs
```

---

# 115. Production Incident: EKS Cluster Healthy but Argo CD Unknown

Possible causes:

```text
Argo CD -> EKS network path
EKS API endpoint
DNS
cluster credentials
authentication
RBAC
```

The EKS workload itself can be perfectly healthy.

---

# 116. Production Incident: Argo CD Repo Works but ECR Pull Fails

This is a workload identity/registry issue.

Check:

```text
ECR image
node/pod identity
ECR permissions
network
image tag
```

Git and Argo CD may be completely healthy.

---

# 117. Production Incident: Cross-Account AccessDenied

Check:

```text
Caller identity
Target role
Trust policy
Permission policy
Resource policy
AWS account
Region
```

Use:

```bash
aws sts get-caller-identity
```

to verify identity.

---

# 118. Production Incident: Secret Missing

If using External Secrets:

```text
Check ExternalSecret
Check controller
Check AWS Secrets Manager
Check IAM role
Check secret ARN
Check target Secret
```

Commands:

```bash
kubectl get externalsecret -n roboshop
kubectl describe externalsecret <name> -n roboshop
kubectl get secret -n roboshop
```

---

# 119. Secret Security

Never troubleshoot by pasting secret values into:

```text
Chat
Tickets
Logs
Git
Slack
```

Inspect metadata and status instead.

---

# 120. Production YAML: Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    app.kubernetes.io/part-of: roboshop
    environment: prod
```

Namespace ownership should be assigned to one automation system.

---

# 121. Production YAML: ServiceAccount

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
```

If AWS access is required, add the organization's approved workload identity configuration.

---

# 122. Production YAML: Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
spec:
  replicas: 3

  selector:
    matchLabels:
      app.kubernetes.io/name: cart

  template:
    metadata:
      labels:
        app.kubernetes.io/name: cart
        app.kubernetes.io/part-of: roboshop

    spec:
      serviceAccountName: cart

      securityContext:
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart:git-abc123

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi

          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10

          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 20

          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            runAsNonRoot: true
```

The exact port, health endpoint and UID must match the application.

---

# 123. Production YAML: Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  type: ClusterIP

  selector:
    app.kubernetes.io/name: cart

  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```

---

# 124. Production YAML: HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cart
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cart

  minReplicas: 3
  maxReplicas: 10

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
    scaleDown:
      stabilizationWindowSeconds: 300

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Tune values using actual workload measurements.

---

# 125. Production YAML: Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb

  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Production TLS, WAF and health-check requirements should be added according to the organization's AWS architecture.

---

# 126. Production YAML: ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-config
  namespace: roboshop
data:
  LOG_LEVEL: INFO
  ENVIRONMENT: prod
```

Do not put credentials in ConfigMaps.

---

# 127. Production YAML: ExternalSecret Concept

```yaml
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: cart-secrets
  namespace: roboshop

spec:
  refreshInterval: 1h

  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore

  target:
    name: cart-secrets

  data:
    - secretKey: DATABASE_PASSWORD
      remoteRef:
        key: prod/roboshop/cart
        property: DATABASE_PASSWORD
```

Verify the External Secrets Operator API version against the installed version before production use.

---

# 128. Production YAML: Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-prod
  namespace: argocd

spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: environments/prod

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
```

---

# 129. Production YAML: Multi-Account Application

The Application itself does not need to know:

```text
AWS account ID
```

if the registered Argo CD cluster destination already encapsulates the target cluster connection.

Conceptually:

```text
Application
   |
   v
destination.name = eks-prod
   |
   v
Argo CD cluster registration
   |
   v
AWS/EKS target
```

This keeps Application manifests focused on Kubernetes destinations.

---

# 130. ApplicationSet + AWS EKS

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-eks-prod
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
```

---

# 131. Terraform Example: ECR

Illustrative ownership:

```hcl
resource "aws_ecr_repository" "cart" {
  name                 = "roboshop/cart"
  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}
```

Terraform owns the repository infrastructure.

CI owns image publishing.

Argo CD owns the Kubernetes image reference.

---

# 132. Terraform Example: EKS Boundary

Terraform can manage:

```hcl
module "eks" {
  source = "terraform-aws-modules/eks/aws"

  cluster_name = "roboshop-prod"

  # networking, node groups, IAM and
  # other infrastructure configuration
}
```

Keep actual module/version configuration aligned with your organization's tested Terraform setup.

---

# 133. Terraform Example: Argo CD Bootstrap

A Helm provider can install Argo CD:

```hcl
resource "helm_release" "argocd" {
  name       = "argocd"
  namespace  = "argocd"
  repository = "https://argoproj.github.io/argo-helm"
  chart      = "argo-cd"

  create_namespace = true
}
```

Production configuration should pin tested chart/app versions and use organization-specific values.

---

# 134. Ownership Warning

Avoid:

```text
Terraform manages Argo CD Application
+
Argo CD manages same Application
```

unless the pattern is deliberately designed.

A cleaner boundary is:

```text
Terraform -> infrastructure
Argo CD -> application desired state
```

---

# 135. EKS Cluster Upgrade

A production upgrade sequence can be:

```text
Review compatibility
   |
   v
Upgrade control plane
   |
   v
Upgrade add-ons
   |
   v
Upgrade nodes
   |
   v
Validate workloads
   |
   v
Validate Argo CD
```

Argo CD continues to manage workloads during the infrastructure lifecycle where connectivity remains available.

---

# 136. EKS Add-ons

Common EKS/platform components include:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI
AWS Load Balancer Controller
External Secrets
Metrics Server
```

Infrastructure/platform ownership should be documented.

---

# 137. Add-on Ownership

Possible model:

```text
Terraform
   |
   +--> EKS managed add-ons

Argo CD
   |
   +--> AWS Load Balancer Controller
   +--> External Secrets
   +--> Monitoring
```

Or another approved platform model.

The key is:

```text
One clear owner per resource.
```

---

# 138. EKS Version Upgrade Risk

Before upgrade:

```text
Check Argo CD compatibility
Check CRDs
Check admission webhooks
Check ingress controller
Check CSI drivers
Check Helm charts
Check Kubernetes API removals
```

Run tests in:

```text
DEV
```

before:

```text
PROD
```

---

# 139. Production Upgrade Strategy

```text
DEV EKS
   |
   v
QA EKS
   |
   v
PROD canary
   |
   v
PROD fleet
```

ApplicationSet can help standardize application deployment across the cluster fleet.

Infrastructure upgrades remain controlled separately.

---

# 140. Multi-Account Security Boundaries

Example:

```text
DEV Account
  |
  +--> Dev IAM
  +--> EKS DEV
  +--> ECR DEV

PROD Account
  |
  +--> Prod IAM
  +--> EKS PROD
  +--> ECR PROD
```

Do not reuse:

```text
DEV role
```

for:

```text
PROD administration
```

---

# 141. Central Argo CD Blast Radius

A compromised central Argo CD may potentially affect every registered cluster it is authorized to manage.

Therefore:

```text
Argo CD
=
high-value security target
```

Protect:

```text
UI
API
RBAC
Secrets
Management cluster
Network
Repository credentials
Cluster credentials
```

---

# 142. Repository Compromise

If the GitOps repository is compromised:

```text
Attacker can potentially modify desired state.
```

Protect it with:

```text
Branch protection
Required reviews
SSO/MFA
CODEOWNERS
Signed commits if appropriate
Secret scanning
CI validation
Security scanning
```

---

# 143. GitOps Supply Chain

A secure deployment chain is:

```text
Source Code
   |
   v
CI
   |
   +--> SAST
   +--> SCA
   +--> Image scan
   +--> DAST where applicable
   |
   v
Image
   |
   v
ECR
   |
   v
GitOps PR
   |
   v
Argo CD
   |
   v
EKS
```

This integrates the user's DevSecOps tooling into GitOps.

---

# 144. GitOps Repository Security

Protect:

```text
main branch
production paths
ApplicationSets
Projects
cluster definitions
```

A developer should not automatically have permission to modify:

```text
production ApplicationSet
```

unless explicitly authorized.

---

# 145. Production CODEOWNERS Concept

Example:

```text
/applicationsets/    @platform-team
/projects/           @platform-team
/clusters/prod/      @platform-team
/values/prod/        @platform-team @service-owners
```

This creates review boundaries.

---

# 146. GitOps PR Validation

Before merge:

```text
YAML validation
Helm template
Kustomize build
Policy checks
Security checks
Image validation
ApplicationSet validation
```

Do not wait for production Argo CD to discover a syntax error.

---

# 147. Policy as Code

Production GitOps can enforce policies such as:

```text
No privileged containers
No hostNetwork
No latest image
Resources required
Approved registries only
Required labels
Required probes
```

Tools may include policy engines appropriate to the organization's platform.

---

# 148. Image Registry Policy

A production policy can require:

```text
*.dkr.ecr.*.amazonaws.com/*
```

instead of allowing arbitrary public registries.

This reduces supply-chain risk.

---

# 149. Pod Security

Production workloads should consider:

```text
runAsNonRoot
allowPrivilegeEscalation=false
capabilities drop ALL
seccomp RuntimeDefault
readOnlyRootFilesystem where possible
```

Do not blindly apply these settings if the application requires different behavior; test first.

---

# 150. NetworkPolicy

For sensitive workloads, consider:

```text
NetworkPolicy
```

to restrict:

```text
Pod -> Pod
Namespace -> Namespace
```

This is independent of Argo CD but can be GitOps-managed.

---

# 151. EKS Security Groups for Pods

Depending on architecture, AWS security controls can be applied to pod traffic.

Use them where there is a clear requirement.

Do not use network-level controls as a substitute for Kubernetes RBAC.

---

# 152. AWS Secrets Manager

Recommended high-level flow:

```text
Secret owner
    |
    v
AWS Secrets Manager
    |
    v
External Secrets
    |
    v
Kubernetes Secret
    |
    v
RoboShop
```

Git contains:

```text
reference
```

not:

```text
secret value
```

---

# 153. Production Secret Rotation

If a database password rotates:

```text
AWS Secrets Manager
       |
       v
External Secrets
       |
       v
Kubernetes Secret
       |
       v
Application
```

Applications must be designed to react correctly to secret updates.

Some workloads require restart/reload behavior.

---

# 154. Multi-Account Secret Isolation

Example:

```text
DEV workload
  |
  +--> DEV secret

PROD workload
  |
  +--> PROD secret
```

Do not use one shared secret role across all environments.

---

# 155. AWS CloudTrail

Although the user's resume intentionally does not list CloudTrail in the monitoring section, AWS audit architecture can still use CloudTrail where required.

For production governance, CloudTrail can help investigate:

```text
IAM actions
EKS API-related AWS calls
ECR actions
STS role assumptions
```

This is separate from the user's Prometheus/Grafana/ELK application observability stack.

---

# 156. Production Observability Boundary

The user's stack:

```text
Prometheus
Grafana
ELK
```

can cover Kubernetes/application monitoring and logging.

AWS audit services can separately provide:

```text
security/audit evidence
```

Do not mix application observability with AWS audit responsibilities.

---

# 157. Argo CD Notifications

Production alerts may include:

```text
Production sync failed
Application degraded
Cluster unreachable
ApplicationSet generation failed
```

Route notifications through the organization's approved channels.

---

# 158. Multi-Cluster Alert Context

Always include:

```text
cluster
account
region
environment
application
namespace
```

Example:

```text
Application: cart
Environment: prod
Cluster: eks-prod-02
Account: production
Region: ap-south-1
Status: Degraded
```

This dramatically improves incident response.

---

# 159. Production Architecture: Full AWS

```text
                         Git
                          |
                          v
                  Jenkins/GitHub Actions
                          |
                    Build/Test/Scan
                          |
                          v
                         ECR
                          |
                          v
                    GitOps Repository
                          |
                          v
                    Central Argo CD
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
      DEV Account      QA Account      PROD Account
          |               |               |
        EKS DEV         EKS QA         EKS PROD
          |               |               |
        ALB             ALB             ALB
          |               |               |
      RoboShop        RoboShop        RoboShop
```

---

# 160. Full Control Plane Architecture

```text
Git
 |
 v
Argo CD API
 |
 v
ApplicationSet Controller
 |
 v
Applications
 |
 v
Application Controller
 |
 v
Kubernetes APIs
 |
 +--> EKS DEV
 +--> EKS QA
 +--> EKS PROD
 +--> EKS DR
```

---

# 161. Production Trust Boundaries

```text
[Git Trust Boundary]
       |
       v
[Argo CD Trust Boundary]
       |
       v
[AWS Account Boundary]
       |
       v
[EKS/Kubernetes Boundary]
       |
       v
[Application Namespace]
```

Each boundary should have:

```text
Authentication
Authorization
Audit
Least privilege
```

---

# 162. Centralized Argo CD Security Checklist

```text
[ ] Private management cluster where appropriate
[ ] Argo CD UI protected
[ ] SSO
[ ] MFA through identity provider
[ ] RBAC
[ ] Project restrictions
[ ] Least privilege cluster access
[ ] Private EKS API where appropriate
[ ] Network segmentation
[ ] Repository protection
[ ] Secrets encrypted
[ ] Cluster credentials protected
[ ] Audit logging
[ ] Backup
[ ] DR
```

---

# 163. EKS Production Checklist

```text
[ ] Multi-AZ
[ ] Private worker subnets
[ ] Appropriate API endpoint configuration
[ ] Restricted API access
[ ] EKS authentication configured
[ ] Least privilege IAM
[ ] Kubernetes RBAC
[ ] Node groups/autoscaling
[ ] ECR access
[ ] Load Balancer Controller
[ ] Secrets integration
[ ] Monitoring
[ ] Logging
[ ] Backup/DR
```

---

# 164. Argo CD + EKS Production Checklist

```text
[ ] Argo CD installed using supported version
[ ] HA evaluated
[ ] Repository credentials configured
[ ] Target clusters registered
[ ] Projects configured
[ ] ApplicationSets configured
[ ] Cluster labels standardized
[ ] Application destinations restricted
[ ] Production sync policy reviewed
[ ] ECR images immutable
[ ] Secrets externalized
[ ] ALB controller configured
[ ] Monitoring enabled
[ ] DR tested
```

---

# 165. Interview Question: Why Use Argo CD With EKS?

### Answer

> EKS provides the managed Kubernetes platform, while Argo CD provides declarative GitOps application delivery and reconciliation. This separates infrastructure provisioning from application deployment and gives the organization auditability, drift detection, controlled rollouts and centralized application management.

---

# 166. Interview Question: How Does Argo CD Authenticate to EKS?

### Answer

> Argo CD needs valid credentials and Kubernetes authorization to access the target EKS API. In modern EKS environments, AWS-integrated authentication mechanisms such as EKS access entries can be used, while Kubernetes RBAC controls what the resulting identity can do. The exact configuration depends on the EKS access mode and Argo CD version.

---

# 167. Interview Question: IAM vs Kubernetes RBAC?

### Answer

> IAM controls AWS API authorization and identity, while Kubernetes RBAC controls permissions against Kubernetes resources. In EKS, AWS authentication establishes the identity that reaches Kubernetes, and Kubernetes RBAC determines what that identity can access.

---

# 168. Interview Question: What Is IRSA?

### Answer

> IRSA, or IAM Roles for Service Accounts, allows Kubernetes workloads to obtain AWS permissions through a Kubernetes ServiceAccount mapped to an IAM role, avoiding static AWS credentials inside application pods.

---

# 169. Interview Question: What Is EKS Pod Identity?

### Answer

> EKS Pod Identity is an AWS capability for associating IAM permissions with Kubernetes workloads without distributing static credentials. It provides an alternative to IRSA and should be evaluated based on the EKS version, AWS guidance and platform requirements.

---

# 170. Interview Question: How Does Argo CD Deploy to a Private EKS Cluster?

### Answer

> Argo CD needs private network connectivity to the EKS Kubernetes API, such as VPC routing or Transit Gateway connectivity, plus correct DNS resolution, TLS, authentication and Kubernetes RBAC. Argo CD communicates with the Kubernetes API; it does not need SSH access to worker nodes.

---

# 171. Interview Question: How Would You Manage Multiple AWS Accounts?

### Answer

> I would separate accounts for environment isolation, use a centralized Argo CD only if the network and security model supports it, register target EKS clusters, apply controlled labels, use Argo CD Projects to restrict destinations, and use least-privilege IAM/EKS/Kubernetes permissions. Terraform would manage infrastructure while Argo CD manages application resources.

---

# 172. Interview Question: How Does ECR Fit Into GitOps?

### Answer

> CI builds and scans the image and publishes it to ECR. GitOps stores the desired image tag or digest. Argo CD reads the GitOps configuration and deploys that immutable artifact to EKS. Argo CD itself does not build or publish the image.

---

# 173. Interview Question: Why Not Let Jenkins Deploy Directly?

### Answer

> Direct Jenkins-to-cluster deployment requires CI to hold production Kubernetes credentials and weakens the separation between build and deployment. With GitOps, CI produces the artifact and updates Git, while Argo CD pulls the desired state and reconciles it to EKS.

---

# 174. Interview Question: How Do You Handle Secrets?

### Answer

> I avoid storing plaintext secrets in Git. In AWS I can use Secrets Manager with External Secrets and EKS workload identity such as IRSA or EKS Pod Identity. Git stores the ExternalSecret configuration/reference, while the secret value remains in the external secret store.

---

# 175. Interview Question: How Do You Deploy ALB With Argo CD?

### Answer

> Argo CD deploys the Kubernetes Ingress resource. The AWS Load Balancer Controller watches that Ingress and reconciles it into an AWS ALB and related resources. Argo CD therefore manages the desired Kubernetes configuration while the AWS controller manages the AWS load-balancing implementation.

---

# 176. Interview Scenario: EKS API Is Private and Argo CD Cannot Sync

### Answer

I would check:

```text
1. EKS endpoint configuration.
2. DNS resolution.
3. VPC routing.
4. Transit Gateway/VPC connectivity.
5. Security groups.
6. Network ACLs.
7. TLS.
8. Authentication.
9. Kubernetes RBAC.
10. Argo CD controller logs.
```

I would not make the endpoint public as the first troubleshooting step.

---

# 177. Interview Scenario: Argo CD Has Access to DEV but Not PROD

Compare:

```text
AWS account
IAM role
EKS access configuration
network path
Kubernetes RBAC
Argo CD cluster credentials
Project destination
```

The fact that DEV works proves the Argo CD installation itself is not necessarily broken.

---

# 178. Interview Scenario: ECR Works in DEV but Not PROD

Check:

```text
ECR account
repository policy
node/pod IAM
region
image tag
network
```

Then inspect:

```bash
kubectl describe pod <pod> -n roboshop
```

for the exact image-pull error.

---

# 179. Interview Scenario: ALB Works in QA but Not PROD

Compare:

```text
Ingress
IngressClass
annotations
subnet tags
security groups
AWS Load Balancer Controller
IAM permissions
DNS
certificate
```

Do not assume Argo CD caused the issue simply because Argo CD deployed the Ingress.

---

# 180. Interview Scenario: Prod Cluster Is Compromised Through Argo CD

Containment priorities:

```text
1. Restrict Argo CD access.
2. Stop dangerous automated sync if necessary.
3. Revoke/rotate compromised credentials.
4. Restrict target cluster access.
5. Preserve audit evidence.
6. Identify malicious Git changes.
7. Restore trusted Git state.
8. Reconcile known-good configuration.
9. Rotate affected secrets.
10. Review blast radius across other clusters.
```

A centralized Argo CD design requires strong incident response because one control plane may manage multiple environments.

---

# 181. Production Golden Path

The user's preferred production architecture can be summarized as:

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
Central Argo CD
    |
    +--> ApplicationSet
    |
    +--> DEV EKS
    +--> QA EKS
    +--> PROD EKS
    |
    v
Kubernetes
    |
    +--> Deployment
    +--> Service
    +--> HPA
    +--> Ingress
    |
    v
AWS Load Balancer Controller
    |
    v
AWS ALB
```

---

# 182. Final Mental Model

Remember these four boundaries:

```text
Terraform
=
AWS infrastructure

CI
=
Artifact creation and security validation

GitOps
=
Desired application state

Argo CD
=
Continuous reconciliation
```

And:

```text
AWS IAM
=
AWS identity/permissions

EKS authentication
=
AWS-to-Kubernetes identity integration

Kubernetes RBAC
=
Kubernetes authorization

Argo CD RBAC
=
Argo CD platform authorization
```

And for the user's RoboShop:

```text
ECR
=
Container artifact registry

EKS
=
Container orchestration

ALB
=
External traffic entry

Argo CD
=
GitOps deployment/reconciliation

ApplicationSet
=
Multi-environment/multi-cluster Application generation
```

---

# 183. Production Architecture Summary

```text
                         AWS ORGANIZATION
                                |
             +------------------+------------------+
             |                  |                  |
          DEV ACCOUNT        QA ACCOUNT         PROD ACCOUNT
             |                  |                  |
           EKS DEV            EKS QA            EKS PROD
             |                  |                  |
             +------------------+------------------+
                                ^
                                |
                         Central Argo CD
                                ^
                                |
                         ApplicationSet
                                ^
                                |
                         GitOps Repository
                                ^
                                |
                      Jenkins / GitHub Actions
                                ^
                                |
                           Application Git
```

Supporting services:

```text
ECR
Secrets Manager
AWS Load Balancer Controller
ALB
Prometheus
Grafana
ELK
Terraform
```

This is a practical production-grade AWS GitOps architecture.

---