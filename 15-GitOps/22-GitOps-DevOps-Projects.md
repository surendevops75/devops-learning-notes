# 22-GitOps-DevOps-Projects.md

# GitOps DevOps Projects — Production Implementation Guide

## 1. Purpose

This file converts the GitOps and Argo CD concepts into practical DevOps/DevSecOps projects.

The projects are designed around realistic AWS/EKS environments and the user's RoboShop microservices platform.

The goal is to move from:

```text
Learning GitOps
      ↓
Building GitOps
      ↓
Operating GitOps
      ↓
Troubleshooting GitOps
      ↓
Designing production GitOps
```

Every project emphasizes:

- real repository structure
- production-oriented YAML
- CI/CD integration
- Argo CD
- Kubernetes
- Helm
- AWS EKS
- ECR
- ALB Ingress
- Terraform boundaries
- security
- observability
- troubleshooting
- rollback
- disaster recovery
- interview discussion

---

# 2. Project Portfolio

Recommended project sequence:

```text
Project 1  → Basic GitOps Application
Project 2  → Helm + Argo CD
Project 3  → Multi-Environment GitOps
Project 4  → ApplicationSet Fleet Management
Project 5  → Multi-Cluster EKS GitOps
Project 6  → App of Apps Platform Bootstrap
Project 7  → CI + GitOps DevSecOps Pipeline
Project 8  → RoboShop Full GitOps
Project 9  → Progressive Delivery
Project 10 → GitOps Security and Governance
Project 11 → GitOps Observability
Project 12 → Disaster Recovery
Project 13 → Enterprise Multi-Account GitOps
Project 14 → Complete Production Capstone
```

---

# 3. Project 1 — Basic GitOps Application

## Objective

Deploy a simple Kubernetes application through Argo CD.

Architecture:

```text
Git
 |
 v
Argo CD
 |
 v
EKS
 |
 v
Deployment
 |
 v
Service
```

---

# 4. Project 1 Repository

```text
gitops-basic/
├── manifests/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── argocd/
    └── application.yaml
```

---

# 5. Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
  labels:
    app.kubernetes.io/part-of: gitops-demo
    environment: dev
```

---

# 6. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: demo
  labels:
    app.kubernetes.io/name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          readinessProbe:
            httpGet:
              path: /
              port: 80
          livenessProbe:
            httpGet:
              path: /
              port: 80
```

---

# 7. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: demo
spec:
  selector:
    app.kubernetes.io/name: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

---

# 8. Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/example/gitops-basic.git
    targetRevision: main
    path: manifests

  destination:
    server: https://kubernetes.default.svc
    namespace: demo

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 9. Project 1 Workflow

```text
Git commit
    ↓
Argo CD refresh
    ↓
Manifest comparison
    ↓
OutOfSync
    ↓
Sync
    ↓
Kubernetes API
    ↓
Deployment
    ↓
Pods
```

---

# 10. Project 1 Validation

```bash
kubectl get applications -n argocd
kubectl get application nginx-demo -n argocd
argocd app get nginx-demo
argocd app sync nginx-demo
kubectl get pods -n demo
kubectl get svc -n demo
```

---

# 11. Project 2 — Helm + Argo CD

## Objective

Package the application using Helm and deploy it through Argo CD.

Repository:

```text
gitops-helm/
├── charts/
│   └── webapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── ingress.yaml
└── argocd/
    └── application.yaml
```

---

# 12. Helm Chart

```yaml
apiVersion: v2
name: webapp
description: Production-style web application chart
type: application
version: 0.1.0
appVersion: "1.0.0"
```

---

# 13. Helm Values

```yaml
replicaCount: 2

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/webapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

---

# 14. Argo CD Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/example/gitops-helm.git
    targetRevision: main
    path: charts/webapp

    helm:
      releaseName: webapp

  destination:
    server: https://kubernetes.default.svc
    namespace: webapp

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

# 15. Project 2 Learning Goals

Understand:

```text
Helm chart
values
templates
Argo CD Helm rendering
releaseName
environment values
```

---

# 16. Project 3 — Multi-Environment GitOps

## Objective

Deploy:

```text
DEV
QA
PROD
```

from the same application definition.

Repository:

```text
gitops-environments/
├── charts/
│   └── webapp/
├── environments/
│   ├── dev/
│   │   └── values.yaml
│   ├── qa/
│   │   └── values.yaml
│   └── prod/
│       └── values.yaml
└── argocd/
    ├── dev.yaml
    ├── qa.yaml
    └── prod.yaml
```

---

# 17. Dev Values

```yaml
replicaCount: 1

image:
  tag: "1.5.0"

resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

---

# 18. QA Values

```yaml
replicaCount: 2

image:
  tag: "1.5.0"

resources:
  requests:
    cpu: 200m
    memory: 256Mi
```

---

# 19. Production Values

```yaml
replicaCount: 3

image:
  tag: "1.5.0"

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: "1"
    memory: 1Gi
```

Production should also define appropriate:

```text
HPA
PDB
probes
securityContext
NetworkPolicy
Ingress
```

---

# 20. Multi-Environment Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp-prod
  namespace: argocd
spec:
  project: production

  source:
    repoURL: https://github.com/example/gitops-environments.git
    targetRevision: main
    path: charts/webapp

    helm:
      valueFiles:
        - ../../environments/prod/values.yaml

  destination:
    server: https://prod-cluster.example
    namespace: webapp

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 21. Production Environment Controls

Production should usually have:

```text
protected Git branch
required reviewers
CODEOWNERS
restricted Argo RBAC
AppProject
controlled sync
audit
rollback process
```

---

# 22. Project 4 — ApplicationSet Fleet Management

## Objective

Automatically generate Applications.

Architecture:

```text
ApplicationSet
      |
      +--> DEV
      +--> QA
      +--> PROD
```

---

# 23. List Generator

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: webapp-environments
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - environment: dev
            namespace: webapp-dev
          - environment: qa
            namespace: webapp-qa
          - environment: prod
            namespace: webapp-prod

  template:
    metadata:
      name: 'webapp-{{environment}}'
    spec:
      project: default

      source:
        repoURL: https://github.com/example/gitops.git
        targetRevision: main
        path: charts/webapp

      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

# 24. Project 4 Validation

```bash
kubectl get applicationset -n argocd
kubectl get applications -n argocd
argocd app list
```

Expected:

```text
webapp-dev
webapp-qa
webapp-prod
```

---

# 25. Project 5 — Multi-Cluster EKS GitOps

## Objective

Manage multiple EKS clusters using one Argo CD instance.

Architecture:

```text
                    Git
                     |
                     v
                  Argo CD
               /      |      \
              v       v       v
          EKS-DEV  EKS-QA  EKS-PROD
```

---

# 26. Cluster Registration

Verify:

```bash
argocd cluster list
```

Register a cluster according to the organization's approved EKS authentication and Argo CD setup.

A typical CLI workflow may use:

```bash
argocd cluster add <eks-context>
```

Do not blindly grant cluster-admin in production.

---

# 27. Cluster Labels

Use meaningful labels such as:

```text
environment=dev
environment=qa
environment=prod
region=ap-south-1
account=prod
team=platform
```

---

# 28. Cluster Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-clusters
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
        namespace: roboshop-prod

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

# 29. Project 5 Failure Scenarios

Test:

```text
cluster unreachable
expired credentials
wrong server
wrong namespace
RBAC denied
ApplicationSet selector mismatch
```

Troubleshoot:

```bash
argocd cluster list
argocd app get <app>
kubectl get applications -n argocd
kubectl describe application <app> -n argocd
```

---

# 30. Project 6 — App of Apps

## Objective

Build a root Application that manages child Applications.

Repository:

```text
app-of-apps/
├── root/
│   └── platform-root.yaml
└── applications/
    ├── frontend.yaml
    ├── cart.yaml
    ├── catalogue.yaml
    ├── user.yaml
    └── orders.yaml
```

---

# 31. Root Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-root
  namespace: argocd
spec:
  project: platform

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: applications

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 32. Child Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart
  namespace: argocd
spec:
  project: roboshop

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: helm/roboshop/cart

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

---

# 33. App of Apps Cleanup

Consider:

```yaml
metadata:
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

Finalizers can ensure child resources are handled according to Argo CD deletion behavior.

Use deletion behavior deliberately because removing a parent can have large consequences.

---

# 34. Project 7 — CI + GitOps DevSecOps

## Objective

Integrate:

```text
Jenkins/GitHub Actions
SonarQube
Trivy
Veracode
ECR
GitOps
Argo CD
EKS
```

Architecture:

```text
Developer
   |
   v
Git
   |
   v
CI
 |
 +--> Unit Tests
 +--> SonarQube
 +--> Build
 +--> Trivy
 +--> Veracode
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

# 35. CI Responsibility

CI should:

```text
checkout
test
build
scan
package
push
```

The deployment state should be represented in Git.

---

# 36. Image Build

Example:

```bash
docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
```

Login and push using the organization's secure AWS authentication method.

Then:

```bash
docker push $ECR_REPOSITORY:$IMAGE_TAG
```

---

# 37. Image Tag

Use:

```text
1.2.7
```

or:

```text
git-<commit-sha>
```

or digest pinning.

Avoid:

```text
latest
```

for production.

---

# 38. GitOps Image Update

Example:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart
  tag: "1.2.7"
```

CI creates a controlled GitOps change.

---

# 39. GitOps PR Flow

```text
CI
 |
 v
image built
 |
 v
ECR
 |
 v
Update GitOps
 |
 v
Pull Request
 |
 v
Validation
 |
 v
Review
 |
 v
Merge
 |
 v
Argo CD
```

---

# 40. Project 8 — RoboShop Full GitOps

## Objective

Implement the complete microservices platform.

Services:

```text
frontend
user
catalogue
cart
orders
payment
shipping
notification
```

Supporting components depend on the actual application implementation.

---

# 41. RoboShop Repository

```text
roboshop-gitops/
├── argocd/
│   ├── projects/
│   ├── applications/
│   └── applicationsets/
│
├── helm/
│   └── roboshop/
│       ├── frontend/
│       ├── user/
│       ├── catalogue/
│       ├── cart/
│       ├── orders/
│       ├── payment/
│       ├── shipping/
│       └── notification/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── ingress/
│   ├── monitoring/
│   └── secrets/
│
└── clusters/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 42. RoboShop Argo Project

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop
  namespace: argocd
spec:
  description: RoboShop application project

  sourceRepos:
    - https://github.com/example/roboshop-gitops.git

  destinations:
    - namespace: roboshop
      server: https://prod-cluster.example

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
    - group: networking.k8s.io
      kind: Ingress
    - group: autoscaling
      kind: HorizontalPodAutoscaler
```

Production should restrict resources further based on actual requirements.

---

# 43. RoboShop Production Deployment

Each service should define:

```text
Deployment
Service
ConfigMap
Secret reference
HPA
PDB where needed
ServiceAccount
securityContext
probes
resources
```

---

# 44. Production Deployment Template

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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart:1.0.0

          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
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
```

Adjust ports and health endpoints to the actual application.

---

# 45. RoboShop Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  selector:
    app.kubernetes.io/name: cart

  ports:
    - name: http
      port: 8080
      targetPort: 8080

  type: ClusterIP
```

---

# 46. RoboShop HPA

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

---

# 47. RoboShop PDB

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: cart
  namespace: roboshop
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: cart
```

PDB values should be aligned with actual replicas and availability requirements.

---

# 48. RoboShop ALB Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
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

Certificate, TLS, health-check, subnet and security-group annotations should be configured according to the actual AWS environment.

---

# 49. Project 8 GitOps Flow

```text
Cart source change
      |
      v
CI
      |
      +--> tests
      +--> SonarQube
      +--> Trivy
      +--> Veracode
      |
      v
ECR
      |
      v
GitOps PR
      |
      v
Approval
      |
      v
Argo CD
      |
      v
EKS
      |
      v
Cart Deployment
```

---

# 50. Project 8 Rollback

Use:

```bash
argocd app history roboshop-cart
argocd app rollback roboshop-cart <ID>
```

Also understand Git-based rollback:

```text
revert Git commit
      |
      v
Argo CD
      |
      v
previous desired state
```

Git revert is often preferable when Git is the authoritative deployment history.

---

# 51. Project 9 — Progressive Delivery

## Objective

Implement controlled releases.

Architecture:

```text
New version
    |
    v
10%
    |
 validation
    |
    v
50%
    |
 validation
    |
    v
100%
```

Use an approved progressive delivery controller/tool where required.

---

# 52. Canary Validation

Monitor:

```text
HTTP 5xx
latency
CPU
memory
business errors
ALB health
```

If metrics deteriorate:

```text
stop
rollback
investigate
```

---

# 53. Project 10 — GitOps Security

## Objective

Implement:

```text
AppProjects
RBAC
SSO
protected Git
secret management
policy
least privilege
```

---

# 54. Security Lab

Create:

```text
readonly
developer
production-deployer
platform-admin
```

Test:

```text
read
sync
delete
project modification
cluster access
```

Expected result:

```text
permissions are explicit
```

---

# 55. Secret Management Lab

Implement:

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

Never commit:

```text
password
access key
private key
database secret
```

to normal GitOps YAML.

---

# 56. Project 11 — GitOps Observability

## Objective

Monitor the complete deployment system.

Monitor:

```text
Argo CD
EKS
Pods
ALB
Applications
CI
ECR
```

---

# 57. Prometheus

Track:

```text
CPU
memory
Pod restarts
request rate
latency
errors
HPA
node health
```

---

# 58. Grafana Dashboards

Recommended dashboards:

```text
Argo CD Overview
EKS Cluster
Node Capacity
RoboShop Application
ALB
Deployment Health
```

---

# 59. ELK

Use ELK for:

```text
application logs
container logs
Kubernetes logs
ingress logs where integrated
```

Search by:

```text
namespace
pod
service
environment
version
request ID
```

---

# 60. GitOps Deployment Observability

Create alerts for:

```text
Application Degraded
Application OutOfSync for too long
sync failure
Pod CrashLoopBackOff
OOMKilled
ALB target unhealthy
high error rate
high latency
```

---

# 61. Project 12 — GitOps Disaster Recovery

## Objective

Prove that the environment can be reconstructed.

Scenario:

```text
Management cluster lost
```

Recovery:

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
Root Application
  |
  v
ApplicationSets
  |
  v
Applications
```

---

# 62. DR Exercise

Destroy a controlled non-production environment.

Then rebuild:

```text
VPC
EKS
IAM
Argo CD
GitOps
Applications
Ingress
Secrets integration
```

Validate the complete application.

Do not perform destructive testing against production without an approved DR exercise.

---

# 63. Project 13 — Enterprise Multi-Account GitOps

## Architecture

```text
Management Account
       |
       v
Central Argo CD
       |
       +--------+---------+
       |        |         |
       v        v         v
     DEV      QA        PROD
   Account  Account    Account
      |        |         |
     EKS      EKS       EKS
```

---

# 64. Enterprise Cluster Labels

Example:

```text
environment=prod
region=ap-south-1
account=production
business-unit=commerce
```

ApplicationSet selects:

```text
environment=prod
```

---

# 65. Enterprise AppProject

Production AppProject should restrict:

```text
Git repository
cluster
namespace
resource types
```

This prevents an application team from deploying arbitrary resources to arbitrary clusters.

---

# 66. Project 14 — Complete Production Capstone

## Objective

Build the full platform:

```text
Terraform
+
AWS
+
EKS
+
ECR
+
Jenkins/GitHub Actions
+
SonarQube
+
Trivy
+
Veracode
+
Helm
+
Argo CD
+
ApplicationSet
+
ALB
+
Prometheus
+
Grafana
+
ELK
+
Secrets Manager
```

---

# 67. Capstone Architecture

```text
                           Developer
                               |
                               v
                         Application Git
                               |
                               v
                    Jenkins / GitHub Actions
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
          SonarQube          Trivy          Veracode
              |                |                |
              +----------------+----------------+
                               |
                               v
                              ECR
                               |
                               v
                         GitOps Repository
                               |
                         Pull Request
                               |
                         Approval/Gates
                               |
                               v
                            Argo CD
                               |
                 +-------------+-------------+
                 |             |             |
                 v             v             v
              DEV EKS       QA EKS       PROD EKS
                 |             |             |
                ALB           ALB           ALB
                 |             |             |
             RoboShop      RoboShop       RoboShop
```

---

# 68. Capstone Repository

```text
roboshop-gitops/
│
├── argocd/
│   ├── projects/
│   │   ├── platform.yaml
│   │   ├── roboshop-dev.yaml
│   │   ├── roboshop-qa.yaml
│   │   └── roboshop-prod.yaml
│   │
│   ├── applications/
│   │   ├── platform-root.yaml
│   │   └── roboshop-root.yaml
│   │
│   └── applicationsets/
│       ├── roboshop-environments.yaml
│       └── roboshop-clusters.yaml
│
├── helm/
│   └── roboshop/
│       ├── frontend/
│       ├── user/
│       ├── catalogue/
│       ├── cart/
│       ├── orders/
│       ├── payment/
│       ├── shipping/
│       └── notification/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── ingress/
│   ├── monitoring/
│   └── secrets/
│
└── clusters/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 69. Capstone Terraform Boundary

Terraform repository:

```text
roboshop-infra/
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── iam/
│   ├── ecr/
│   ├── rds/
│   └── security/
│
└── environments/
    ├── dev/
    ├── qa/
    └── prod/
```

Terraform provisions infrastructure.

GitOps deploys Kubernetes workloads.

---

# 70. Capstone Application Repository

Example:

```text
roboshop-cart/
├── src/
├── tests/
├── Dockerfile
├── pom.xml
└── Jenkinsfile
```

or the equivalent GitHub Actions workflow.

---

# 71. Capstone CI Pipeline

```text
Checkout
   |
Unit Tests
   |
Build
   |
SonarQube
   |
Docker Build
   |
Trivy
   |
Veracode
   |
ECR Push
   |
GitOps Update
   |
Pull Request
```

---

# 72. Capstone Production Promotion

```text
DEV
 |
 v
automated validation
 |
 v
QA
 |
 v
integration validation
 |
 v
production approval
 |
 v
PROD
```

Use the same immutable artifact where possible.

---

# 73. Capstone Multi-Cluster

```text
                         Central Argo CD
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           DEV EKS         QA EKS          PROD EKS
              |               |               |
           dev namespace   qa namespace   prod namespace
```

---

# 74. Capstone ApplicationSet

Use:

```text
List generator
```

for explicit environments.

Use:

```text
Cluster generator
```

for registered EKS clusters.

Use:

```text
Matrix generator
```

when combining dimensions such as:

```text
environment × cluster
```

where appropriate.

---

# 75. Capstone Environment Matrix

Conceptually:

```text
              DEV     QA      PROD
Cluster A      X       X
Cluster B                      X
Cluster C                      X
```

ApplicationSet can generate only the desired combinations.

---

# 76. Capstone Production Sync

Development:

```text
automated
prune
selfHeal
```

QA:

```text
automated or controlled
```

Production:

```text
approval
controlled sync
audit
rollback
```

Exact behavior depends on organizational policy.

---

# 77. Capstone Rollback

Application rollback:

```bash
argocd app history roboshop-cart
argocd app rollback roboshop-cart <ID>
```

Git rollback:

```bash
git revert <commit>
git push
```

Then:

```text
Argo CD
   |
   v
reconciliation
```

---

# 78. Capstone Troubleshooting Workflow

When deployment fails:

```text
1. Check Argo Application
2. Check sync status
3. Check diff
4. Check operation state
5. Check events
6. Check controller logs
7. Check Repo Server
8. Check rendered Helm output
9. Check Kubernetes resources
10. Check Pod logs
11. Check Service
12. Check ALB
13. Check application dependencies
```

---

# 79. Capstone Commands

```bash
argocd app list
argocd app get roboshop-cart
argocd app diff roboshop-cart
argocd app history roboshop-cart
argocd app sync roboshop-cart
argocd app wait roboshop-cart

argocd cluster list
argocd repo list

kubectl get applications -n argocd
kubectl get applicationsets -n argocd
kubectl describe application roboshop-cart -n argocd
kubectl get events -n roboshop-prod
kubectl get pods -n roboshop-prod
kubectl describe pod <pod> -n roboshop-prod
kubectl logs <pod> -n roboshop-prod
```

---

# 80. Capstone Production Validation

After deployment:

```text
Argo CD = Synced
Application = Healthy
Deployment = Available
Pods = Ready
Service = endpoints present
ALB = healthy targets
Prometheus = metrics available
Grafana = dashboard healthy
ELK = logs available
```

---

# 81. Capstone Security Validation

Check:

```text
Git branch protection
PR approvals
Argo RBAC
AppProject restrictions
cluster credentials
AWS IAM
secret references
Pod security
NetworkPolicy
image provenance
container scanning
```

---

# 82. Capstone Reliability Validation

Check:

```text
multi-AZ
replicas
HPA
PDB
resource requests
probes
rolling update
capacity
failure recovery
```

---

# 83. Capstone DR Validation

Demonstrate:

```text
Argo CD rebuild
cluster rebuild
application sync
secret recovery
image availability
database recovery
DNS/ALB recovery
```

---

# 84. Project Portfolio GitHub Presentation

For each project create:

```text
README.md
architecture diagram
repository structure
deployment instructions
troubleshooting
screenshots
lessons learned
```

Do not publish:

```text
AWS credentials
private keys
production secrets
internal endpoints
sensitive account information
```

---

# 85. README Template

Each project README should include:

```text
# Project Name

## Objective

## Architecture

## Technologies

## Repository Structure

## Prerequisites

## Installation

## Configuration

## Deployment

## Validation

## Troubleshooting

## Security

## Production Considerations

## Disaster Recovery

## Lessons Learned
```

---

# 86. Project Evidence

For an interview portfolio, capture:

```text
Argo CD Application healthy
ApplicationSet generated apps
multi-cluster view
Helm repository
CI pipeline
security scans
ECR image
EKS workloads
ALB
Grafana
Kibana
Git history
rollback demonstration
```

Do not expose secrets in screenshots.

---

# 87. Project Interview Story

A strong project explanation:

> I implemented GitOps for a Kubernetes microservices platform on AWS EKS. CI handled testing, code quality, security scanning, Docker image creation and publishing to ECR. The deployment configuration was maintained separately in Git. Argo CD continuously reconciled that desired state into EKS. For multiple environments and clusters, I used ApplicationSets and AppProjects to automate deployment while maintaining security boundaries. AWS Load Balancer Controller exposed applications through ALB, and Prometheus, Grafana and ELK provided observability.

---

# 88. Interview Follow-Up

If asked:

### Why did you separate application and GitOps repositories?

Answer:

> It separates source-code lifecycle from deployment-state lifecycle. CI can build and scan artifacts while the GitOps repository represents the approved desired state for each environment.

---

# 89. Interview Follow-Up

### Why use Argo CD instead of Jenkins kubectl?

Answer:

> Argo CD provides a continuous reconciliation model, Git as the source of truth, drift detection, declarative deployment state and stronger separation between CI and production cluster access.

---

# 90. Interview Follow-Up

### How did you deploy to multiple EKS clusters?

Answer:

> I registered the target clusters with Argo CD and used ApplicationSets, especially the cluster generator, to dynamically generate Applications based on cluster labels such as environment and region. AppProjects restricted which repositories, clusters and namespaces each application could use.

---

# 91. Interview Follow-Up

### How did you handle secrets?

Answer:

> I avoided storing plaintext production secrets in Git. The GitOps manifests reference secret-management resources, while the actual secret values are retrieved from AWS Secrets Manager through a Kubernetes secret-management integration.

---

# 92. Interview Follow-Up

### How did you handle rollback?

Answer:

> I can roll back through Argo CD history when appropriate, but because Git is the source of truth, a controlled Git revert is often preferable for a permanent rollback. The new desired state is then reconciled by Argo CD.

---

# 93. Interview Follow-Up

### What if Argo CD is down?

Answer:

> Existing workloads normally continue because Argo CD is not in the application traffic path. New synchronization and drift correction are affected. This is why production Argo CD should have an appropriate HA and recovery design.

---

# 94. Interview Follow-Up

### What if Kubernetes is changed manually?

Answer:

> Argo CD detects the difference between Git and the live cluster. If self-heal is enabled and the resource is intended to be fully Git-managed, Argo CD can restore the desired state.

---

# 95. Interview Follow-Up

### What if HPA changes replicas?

Answer:

> HPA is an intentional controller. I avoid creating ownership conflicts by understanding which fields are dynamically managed. Argo CD configuration should account for legitimate controller-managed fields rather than continuously fighting them.

---

# 96. Interview Follow-Up

### How does Terraform coexist with Argo CD?

Answer:

> Terraform provisions AWS and cluster infrastructure such as VPC, EKS, IAM, ECR and databases. Argo CD manages application-level Kubernetes resources. I avoid having both tools manage the same resource.

---

# 97. Interview Follow-Up

### Why use ALB instead of API Gateway?

Answer:

> In this architecture, Kubernetes Ingress is exposed through the AWS Load Balancer Controller, which provisions an AWS Application Load Balancer. This fits the application's Kubernetes ingress requirement without introducing API Gateway.

---

# 98. Interview Follow-Up

### How do you promote an image?

Answer:

> CI builds and scans the image once, pushes an immutable image to ECR, and the GitOps configuration is updated to reference that artifact. The same artifact can then be promoted through environments using controlled Git changes.

---

# 99. Interview Follow-Up

### How do you secure production?

Answer:

> I use protected Git branches, pull-request approvals, AppProjects, RBAC, least-privilege cluster access, secret management, image scanning, workload security contexts, network policies and environment/account isolation.

---

# 100. Project Troubleshooting Scenarios

## Scenario 1 — ApplicationSet generated nothing

Check:

```bash
kubectl get applicationset -n argocd
kubectl describe applicationset roboshop-clusters -n argocd
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-applicationset-controller
```

Check:

```text
generator selector
cluster labels
template
repository
controller health
```

---

# 101. Scenario 2 — PROD Application Missing

Check:

```bash
kubectl get applications -n argocd
argocd app list
```

Then:

```text
ApplicationSet selector
cluster registration
environment label
namespace
project
```

---

# 102. Scenario 3 — ECR ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop-prod
```

Look for:

```text
image name
tag
digest
ECR permissions
node/pod AWS identity
network connectivity
```

---

# 103. Scenario 4 — ALB Not Created

Check:

```bash
kubectl describe ingress roboshop -n roboshop-prod
kubectl get ingress -n roboshop-prod
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

Investigate:

```text
IngressClass
subnet tags
IAM permissions
security groups
annotations
controller health
```

---

# 104. Scenario 5 — Deployment Progressing Forever

Check:

```bash
kubectl rollout status deployment/cart -n roboshop-prod
kubectl get pods -n roboshop-prod
kubectl describe pod <pod> -n roboshop-prod
kubectl logs <pod> -n roboshop-prod
```

Possible causes:

```text
readiness failure
image pull
resource shortage
config error
secret missing
dependency failure
```

---

# 105. Scenario 6 — GitOps Sync Succeeds but Application Is Broken

This means:

```text
desired resources were accepted
```

but runtime health is failing.

Check:

```text
Pod
Service
EndpointSlice
Ingress
ALB
dependencies
application logs
metrics
```

GitOps success does not automatically mean application success.

---

# 106. Scenario 7 — Manual Change Reappears

Example:

```bash
kubectl edit deployment cart -n roboshop-prod
```

You change:

```text
replicas
```

Then Argo CD restores it.

This is expected when:

```text
selfHeal=true
```

and the field is Git-owned.

---

# 107. Scenario 8 — Production Rollback

Immediate containment:

```text
stop promotion
assess impact
rollback
validate
```

Possible:

```bash
argocd app history roboshop-cart
argocd app rollback roboshop-cart <ID>
```

Then create the permanent Git correction.

---

# 108. Scenario 9 — Argo CD Cannot Reach Cluster

Check:

```bash
argocd cluster list
```

Then investigate:

```text
API endpoint
authentication
authorization
network
security groups
cluster status
credentials
```

---

# 109. Scenario 10 — Repo Server Cannot Render Helm

Check:

```text
repo URL
branch
path
Chart.yaml
values files
Helm syntax
dependencies
credentials
```

Use local validation:

```bash
helm lint .
helm template .
```

---

# 110. Project Documentation Deliverables

Every project should produce:

```text
1. README
2. architecture diagram
3. Git repository
4. YAML manifests
5. Helm chart
6. CI pipeline
7. Argo Application
8. ApplicationSet where applicable
9. troubleshooting guide
10. production checklist
```

---

# 111. Recommended Project Sequence

Do not start with the largest project.

Follow:

```text
Project 1
Basic GitOps
     ↓
Project 2
Helm
     ↓
Project 3
Environments
     ↓
Project 4
ApplicationSet
     ↓
Project 5
Multi-Cluster
     ↓
Project 6
App of Apps
     ↓
Project 7
CI + DevSecOps
     ↓
Project 8
RoboShop
     ↓
Project 9
Progressive Delivery
     ↓
Project 10
Security
     ↓
Project 11
Observability
     ↓
Project 12
DR
     ↓
Project 13
Multi-Account
     ↓
Project 14
Capstone
```

---

# 112. Minimum Production Project

If time is limited, prioritize:

```text
RoboShop
+
Helm
+
Argo CD
+
ApplicationSet
+
multi-environment
+
multi-cluster
+
ECR
+
ALB
+
CI
+
security
+
observability
+
rollback
```

This gives the strongest interview demonstration.

---

# 113. Production Project Architecture Diagram

```text
                     APPLICATION GIT
                            |
                            v
                         CI/CD
                            |
          +-----------------+-----------------+
          |                 |                 |
      SonarQube           Trivy          Veracode
          |                 |                 |
          +-----------------+-----------------+
                            |
                            v
                           ECR
                            |
                            v
                       GITOPS GIT
                            |
                            v
                     ARGO CD CONTROL PLANE
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
         DEV EKS          QA EKS          PROD EKS
            |               |               |
           ALB             ALB             ALB
            |               |               |
        RoboShop         RoboShop        RoboShop
            |               |               |
        Prometheus       Prometheus      Prometheus
        Grafana          Grafana         Grafana
        ELK              ELK             ELK
```

---

# 114. Project Success Criteria

The capstone is complete when:

```text
[ ] source code builds
[ ] CI passes
[ ] SonarQube passes
[ ] Trivy passes
[ ] Veracode checks complete
[ ] image reaches ECR
[ ] GitOps PR is created
[ ] PR is reviewed
[ ] Argo CD syncs
[ ] EKS Pods become Ready
[ ] ALB targets become healthy
[ ] application works
[ ] metrics are available
[ ] logs are available
[ ] drift is detected
[ ] self-heal is tested
[ ] rollback is tested
[ ] multi-environment deployment works
[ ] multi-cluster deployment works
[ ] secret management works
[ ] DR procedure is documented
```

---

# 115. What This Project Demonstrates in an Interview

It demonstrates practical knowledge of:

```text
Git
GitOps
Kubernetes
AWS
EKS
ECR
Helm
Argo CD
ApplicationSet
multi-cluster
CI/CD
DevSecOps
Terraform
ALB
RBAC
secrets
observability
HA
DR
troubleshooting
```

More importantly, it demonstrates understanding of **how the tools work together**, not merely individual commands.

---

# 116. Final Project Mental Model

The entire platform should be understood as:

```text
                DEVELOPMENT PLANE
                       |
                       v
                  Application Git
                       |
                       v
                       CI
                       |
            +----------+----------+
            |          |          |
          Build      Security   Tests
            |          |          |
            +----------+----------+
                       |
                       v
                      ECR
                       |
                       v

                  DELIVERY PLANE
                       |
                       v
                  GitOps Git
                       |
                       v
                    Argo CD
                       |
            +----------+----------+
            |          |          |
            v          v          v
          DEV EKS    QA EKS    PROD EKS
            |          |          |
            v          v          v
           ALB        ALB        ALB
            |          |          |
            v          v          v
        Applications Applications Applications

                       ^
                       |
                Reconciliation

                       |
                       v
               OBSERVABILITY
          Prometheus / Grafana / ELK
```

---

# 117. Final Production Principle

A production GitOps project is not:

```text
Install Argo CD
+
Create Application
```

A production GitOps platform is:

```text
Secure source control
+
Secure CI
+
Immutable artifacts
+
Declarative desired state
+
Argo CD reconciliation
+
Kubernetes runtime
+
Environment isolation
+
Multi-cluster strategy
+
Security boundaries
+
Secrets management
+
Observability
+
Rollback
+
Disaster recovery
+
Operational ownership
```

That is the level expected from a DevOps/DevSecOps engineer working with GitOps in production.
