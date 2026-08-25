# 24-GitOps-Mock-Interview.md

# GitOps with Argo CD — Final Mock Interview

## How to Use This Mock Interview

This is the final practical interview simulation for the GitOps with Argo CD section.

It is intentionally different from the previous interview-preparation file.

The previous file is a knowledge bank.

This file simulates an actual interview:

```text
Interviewer
    ↓
Question
    ↓
Candidate Answer
    ↓
Follow-up Challenge
    ↓
Candidate Response
    ↓
Interviewer Assessment
```

Practice answering each primary question aloud before reading the model answer.

Do not try to memorize every sentence.

Your objective is to communicate:

```text
clarity
technical correctness
production experience
troubleshooting ability
architecture thinking
security awareness
trade-offs
ownership
```

---

# PART 1 — INTERVIEWER SETUP

## Candidate Profile Used for This Mock Interview

The candidate is interviewing for:

```text
DevOps Engineer
Cloud DevOps Engineer
DevSecOps Engineer
```

Experience level:

```text
4+ years
```

Practical environment:

```text
AWS
EKS
ECR
Kubernetes
Helm
ALB Ingress
Jenkins
GitHub Actions
GitLab CI/CD
Argo CD
Terraform
SonarQube
Trivy
Veracode
Prometheus
Grafana
ELK
```

Primary project:

```text
RoboShop microservices platform
```

---

# PART 2 — INTERVIEW RULES

During the interview:

1. Start with the direct answer.
2. Explain the architecture.
3. Give a practical example.
4. Mention production considerations.
5. Do not over-explain unless asked.
6. Never claim tools or experience you did not actually use.
7. When troubleshooting, show a sequence rather than guessing.
8. Distinguish Argo CD problems from Kubernetes problems.
9. Distinguish application problems from infrastructure problems.
10. Explain trade-offs when there is no single correct answer.

---

# PART 3 — ROUND 1: INTRODUCTION

## Interviewer

Tell me about yourself.

### Strong Candidate Answer

> I am a DevOps/DevSecOps Engineer with 4+ years of experience working with AWS, Kubernetes, CI/CD, infrastructure automation and security integration. My main experience includes AWS services such as EKS, ECR, VPC, IAM, ALB and RDS, along with Kubernetes, Helm, Terraform, Jenkins, GitHub Actions, GitLab CI/CD and Argo CD.
>
> I have worked on a microservices platform where I handled infrastructure provisioning, containerization, Kubernetes deployment, CI/CD and GitOps. In the CI stage I worked with tools such as SonarQube, Trivy and Veracode, while Argo CD was used for GitOps-based deployment to EKS. For observability I worked with Prometheus, Grafana and ELK.
>
> My focus is on building reliable, secure and repeatable deployment workflows rather than only automating individual commands.

### Interviewer Follow-up

What part of this architecture are you strongest in?

### Strong Answer

> I am strongest in the combination of AWS, Kubernetes, CI/CD, Terraform and GitOps. I can explain the complete lifecycle from infrastructure provisioning and application build through security scanning, image publishing, GitOps deployment and Kubernetes troubleshooting.

### Interviewer Follow-up

What would you like to improve?

### Strong Answer

> I continuously deepen my knowledge of production-scale Kubernetes and GitOps operations, particularly multi-cluster architecture, deployment strategies, security and reliability.

---

# PART 4 — ROUND 2: GITOPS FUNDAMENTALS

## Question 1

What is GitOps?

### Strong Answer

> GitOps is an operating model where the desired state of infrastructure or applications is stored declaratively in Git, and a reconciliation system continuously works to make the runtime environment match that desired state. With Argo CD, Git becomes the source of truth and Argo CD continuously reconciles the Kubernetes cluster against that state.

### Follow-up

Why is Git not enough?

### Strong Answer

> Git stores the desired state, but Git itself does not continuously reconcile that state into Kubernetes. Argo CD provides the reconciliation control loop.

### Follow-up

So is Argo CD GitOps?

### Strong Answer

> Argo CD is a GitOps tool, while GitOps itself is the operating model and set of principles.

---

# Question 2

What problem does GitOps solve?

### Strong Answer

> GitOps addresses configuration drift, inconsistent deployment processes, weak auditability and excessive direct cluster access. It gives us a version-controlled desired state, reviewable changes, automated reconciliation and a clear audit trail.

---

# Question 3

Explain desired state versus actual state.

### Strong Answer

> Desired state is what we declare in Git. Actual state is what currently exists in Kubernetes.
>
> For example, if Git says three replicas and the cluster has five replicas, there is drift. Argo CD detects the difference and, when configured for self-healing, can reconcile the live state back toward the desired state.

---

# Question 4

What is configuration drift?

### Strong Answer

> Configuration drift occurs when the live environment differs from the approved configuration stored in Git.

### Follow-up

How can drift happen?

### Strong Answer

> Someone may use kubectl directly, another controller may modify a field, an emergency change may be made manually, or infrastructure configuration may change outside the GitOps workflow.

---

# Question 5

What is the difference between push and pull deployment?

### Strong Answer

> In a push model, CI connects to the target cluster and applies the deployment. In a pull model, a controller such as Argo CD operates from the Kubernetes/GitOps side, retrieves the desired state from Git and reconciles the cluster.

---

# Question 6

Why is pull-based deployment attractive from a security perspective?

### Strong Answer

> It reduces the need for CI systems to hold direct production cluster credentials. CI can build and publish artifacts and update the GitOps repository, while Argo CD handles deployment using controlled cluster access.

---

# PART 5 — ROUND 3: CI VS GITOPS CD

## Question 7

Explain CI and GitOps CD in your architecture.

### Strong Answer

```text
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
```

> CI is responsible for producing and validating the artifact. GitOps CD is responsible for declaring and reconciling the deployment state.

### Follow-up

Why not let Jenkins run kubectl?

### Strong Answer

> It is possible, but in a mature GitOps model I prefer CI to publish the artifact and update the desired deployment state. Argo CD then owns Kubernetes reconciliation. This gives better separation of duties, auditability and drift management.

---

# Question 8

Who should update the image tag?

### Strong Answer

> CI can update the GitOps repository with the new immutable image reference through an approved Git workflow. In production, I prefer a PR-based process so the change can be validated and approved before Argo CD deploys it.

---

# Question 9

Should every environment rebuild the image?

### Strong Answer

> Preferably no. Build once and promote the same immutable artifact across environments.

Example:

```text
build
  ↓
ECR
  ↓
DEV
  ↓
QA
  ↓
PROD
```

This improves reproducibility.

---

# PART 6 — ROUND 4: ARGO CD ARCHITECTURE

## Question 10

Explain Argo CD architecture.

### Strong Answer

```text
                 Git
                  |
                  v
              Repo Server
                  |
                  v
             Application
             Controller
                  |
                  v
            Kubernetes API
                  |
                  v
                EKS

Users/CI
   |
   v
API Server

ApplicationSet
Controller
   |
   v
Applications
```

Important components include:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-applicationset-controller
Redis
```

### Follow-up

Which component performs reconciliation?

### Answer

> Application Controller.

### Follow-up

Which component handles repository manifest generation?

### Answer

> Repo Server.

### Follow-up

Which component provides API/UI access?

### Answer

> API Server.

### Follow-up

Which component generates Applications from templates?

### Answer

> ApplicationSet Controller.

---

# Question 11

What does the Application Controller actually do?

### Strong Answer

> It watches Argo CD Applications, compares desired and live state, evaluates resource health, detects differences and performs reconciliation/synchronization according to the Application configuration.

---

# Question 12

What happens if the Argo CD server is down?

### Strong Answer

> The Argo CD management interface and API operations are affected. Existing Kubernetes workloads normally continue serving traffic because Argo CD is not in the application's request path. However, new deployment operations and reconciliation can be affected.

### Follow-up

Does that mean Argo CD is not important?

### Strong Answer

> It is still critical for deployment control and desired-state reconciliation. That is why production environments need appropriate HA, monitoring and recovery procedures.

---

# PART 7 — ROUND 5: ARGO CD APPLICATION

## Question 13

Explain an Argo CD Application.

### Strong Answer

> An Application defines the desired source and destination for a workload. It tells Argo CD which Git repository and revision to use, which path or Helm/Kustomize source to render, which cluster and namespace to deploy into, and what synchronization policy should be used.

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart
  namespace: argocd
spec:
  project: roboshop

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: helm/cart

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# Question 14

Explain source versus destination.

### Strong Answer

> Source describes where the desired configuration comes from. Destination describes where it should be deployed.

```text
SOURCE
Git repository
branch/tag/revision
path
Helm/Kustomize

DESTINATION
cluster
namespace
```

---

# Question 15

What is targetRevision?

### Answer

> It specifies the Git revision Argo CD should use, such as a branch, tag or commit revision.

### Follow-up

What would you prefer for production?

### Strong Answer

> I would use a controlled Git promotion model. The exact revision strategy depends on the repository workflow, but production should not silently follow an uncontrolled mutable reference.

---

# Question 16

What is an AppProject?

### Strong Answer

> An AppProject provides boundaries for Argo CD Applications. It can restrict which repositories Applications can use, which clusters and namespaces they can target, and which Kubernetes resource types they can manage.

---

# Question 17

Why is AppProject important for security?

### Strong Answer

> It prevents an Application from automatically having unrestricted access across repositories, clusters and namespaces. It is an important least-privilege boundary.

---

# PART 8 — ROUND 6: SYNC AND RECONCILIATION

## Question 18

What is Sync?

### Answer

> Sync is the process of applying the desired Git state to the target cluster.

---

# Question 19

What is OutOfSync?

### Answer

> OutOfSync means Argo CD has identified a difference between the desired state and live state.

---

# Question 20

What is Synced?

### Answer

> Synced means the desired and live resource state are considered aligned according to Argo CD's comparison.

---

# Question 21

What is Self-Heal?

### Answer

> Self-heal allows Argo CD to automatically correct eligible live-state changes that differ from the Git-defined desired state.

---

# Question 22

What is Prune?

### Answer

> Prune removes resources that were previously managed by the Application but are no longer present in the desired state.

### Follow-up

Why can prune be dangerous?

### Strong Answer

> A bad Git change can intentionally or accidentally remove a resource from the desired state. If pruning is enabled, that can result in deletion from the cluster. Production controls such as PR reviews, protected branches, AppProjects and controlled sync policies are therefore important.

---

# Question 23

Can an application be Synced but Degraded?

### Answer

> Yes. Sync status and health status are different. Argo CD may successfully apply the desired manifests while the application itself remains unhealthy, for example because Pods are failing readiness or crashing.

---

# Question 24

Can an application be Healthy but OutOfSync?

### Answer

> Yes. The application can currently work while its live configuration differs from Git.

---

# PART 9 — ROUND 7: TROUBLESHOOTING

## Question 25

Production application is OutOfSync. What do you do?

### Strong Answer

I would not immediately click Sync.

First:

```bash
argocd app get <app>
argocd app diff <app>
```

Then determine:

```text
which resource?
which field?
intentional or accidental?
another controller changing it?
```

If it is legitimate drift:

```text
fix Git
```

If another controller owns the field:

```text
define ownership
```

If it is unauthorized drift:

```text
reconcile
```

---

# Question 26

Application is Degraded. What do you check?

### Strong Answer

First identify the unhealthy resource:

```bash
argocd app get <app>
```

Then inspect Kubernetes:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl get events -n <namespace>
```

Then investigate:

```text
image
configuration
secrets
probes
resources
dependencies
networking
```

---

# Question 27

Application is stuck Progressing for 20 minutes.

### Strong Answer

I would check whether the rollout is actually progressing:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace>
```

Then check:

```text
readiness
startup
image pull
resource capacity
scheduling
dependencies
```

---

# Question 28

Argo CD sync failed with a permission error.

### Strong Answer

I would check:

```text
AppProject
destination
resource restrictions
Kubernetes RBAC
namespace
service account
```

I would not immediately grant cluster-admin.

---

# Question 29

Git repository authentication failed.

### Strong Answer

Run:

```bash
argocd repo list
```

Then verify:

```text
repository URL
SSH key/token
credential validity
repository permissions
network
TLS/certificate
```

---

# Question 30

Helm rendering fails.

### Strong Answer

I would reproduce the rendering outside Argo CD:

```bash
helm lint <chart>
helm template <release> <chart>
```

Then check:

```text
values
templates
dependencies
chart version
API versions
missing files
```

---

# Question 31

Argo CD cannot connect to an EKS cluster.

### Strong Answer

Check:

```bash
argocd cluster list
```

Then investigate:

```text
cluster registration
API endpoint
authentication
Kubernetes RBAC
network connectivity
EKS status
```

---

# Question 32

ApplicationSet is not generating Applications.

### Strong Answer

I would check:

```bash
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd
```

Then validate:

```text
generator
Git path
cluster selector
cluster labels
template
controller logs
```

---

# PART 10 — ROUND 8: APPLICATIONSET

## Question 33

Why do we need ApplicationSet?

### Answer

> ApplicationSet is useful when we need to create and manage many Argo CD Applications from a common template. It reduces repetitive Application definitions and is especially useful for multi-environment and multi-cluster deployments.

---

# Question 34

Explain the List generator.

### Answer

> The List generator takes explicitly defined parameter sets and generates Applications from them.

Example:

```text
dev
qa
prod
```

---

# Question 35

Explain the Cluster generator.

### Answer

> The Cluster generator uses clusters registered with Argo CD and can use cluster metadata/labels to select the appropriate target clusters.

Example:

```text
environment=prod
```

---

# Question 36

How would you deploy RoboShop to DEV, QA and PROD?

### Strong Answer

I could use an ApplicationSet with a List generator:

```text
dev → dev cluster + dev values
qa  → qa cluster + qa values
prod → prod cluster + prod values
```

For larger fleets I would use the Cluster generator and cluster labels.

---

# Question 37

How would you deploy to multiple EKS clusters?

### Strong Answer

```text
Central Argo CD
      |
      +--> EKS-DEV
      +--> EKS-QA
      +--> EKS-PROD
```

I would register the target clusters, apply labels such as:

```text
environment
region
account
team
```

and use ApplicationSets to generate Applications based on those labels.

---

# PART 11 — ROUND 9: MULTI-CLUSTER

## Question 38

Can one Argo CD instance manage multiple clusters?

### Answer

> Yes. One Argo CD control plane can manage multiple registered Kubernetes clusters, including EKS clusters, provided it has the required connectivity, authentication and authorization.

---

# Question 39

How do you prevent deployment to the wrong cluster?

### Strong Answer

Use multiple controls:

```text
Application destination
AppProject destination restrictions
cluster labels
ApplicationSet selectors
Argo CD RBAC
Git review
```

---

# Question 40

What is the risk of centralized Argo CD?

### Answer

> Centralization simplifies governance but increases the blast radius of a control-plane compromise or outage. For highly isolated organizations, multiple Argo CD instances can be appropriate.

---

# Question 41

When would you use multiple Argo CD instances?

### Answer

Consider:

```text
regulatory isolation
business-unit boundaries
regional autonomy
security domains
blast-radius reduction
```

---

# PART 12 — ROUND 10: AWS EKS

## Question 42

Explain your GitOps architecture on AWS.

### Strong Answer

```text
Developer
   |
   v
Git
   |
   v
Jenkins/GitHub Actions
   |
   +--> Tests
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
AWS ALB
```

---

# Question 43

Where does ALB fit?

### Answer

```text
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

AWS Load Balancer Controller reconciles Kubernetes Ingress configuration into AWS load-balancing resources.

---

# Question 44

Why use ALB instead of exposing every service?

### Strong Answer

> It reduces public exposure, infrastructure cost and operational complexity. Internal services can remain ClusterIP while external HTTP/HTTPS traffic enters through the ALB.

---

# Question 45

How would you troubleshoot ALB 503?

### Strong Answer

Check:

```bash
kubectl get ingress -n <namespace>
kubectl describe ingress <name> -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpointslice -n <namespace>
```

Then investigate:

```text
Pod readiness
target health
security groups
Ingress configuration
Service selectors
AWS Load Balancer Controller
```

---

# PART 13 — ROUND 11: SECURITY

## Question 46

How do you secure Argo CD?

### Strong Answer

I would use:

```text
SSO
RBAC
AppProjects
least privilege
private repositories
protected Git branches
credential rotation
network controls
HA
monitoring
audit
```

---

# Question 47

How do you secure production GitOps?

### Strong Answer

```text
MFA/SSO
protected branches
PR approvals
CODEOWNERS
least privilege
private repositories
secret management
image scanning
immutable artifacts
audit
```

---

# Question 48

Would you store passwords in Git?

### Answer

> No, not as plaintext. I would use an external secret store such as AWS Secrets Manager and an appropriate Kubernetes integration.

---

# Question 49

How do you prevent a developer from deploying to production?

### Strong Answer

Use:

```text
Git permissions
CODEOWNERS
production branch protection
Argo CD RBAC
AppProjects
cluster/namespace restrictions
```

---

# Question 50

What is the difference between Argo CD RBAC and Kubernetes RBAC?

### Answer

> Argo CD RBAC controls what users can do within Argo CD. Kubernetes RBAC controls what identities can do against the Kubernetes API. They are separate authorization layers.

---

# PART 14 — ROUND 12: SECRETS

## Question 51

Explain your secrets strategy.

### Strong Answer

```text
AWS Secrets Manager
        |
        v
External Secrets integration
        |
        v
Kubernetes Secret
        |
        v
Pod
```

Git contains the configuration/reference, not the plaintext secret.

---

# Question 52

Why not just base64 encode secrets?

### Answer

> Base64 is encoding, not encryption. A Kubernetes Secret manifest containing base64 values should not be treated as a secure secret-management strategy.

---

# Question 53

What happens if a secret changes?

### Strong Answer

The secret-management integration should synchronize the updated value into Kubernetes according to its configured refresh behavior. The application may need a restart or reload mechanism depending on how it consumes the secret.

---

# PART 15 — ROUND 13: HELM

## Question 54

Why use Helm with Argo CD?

### Answer

> Helm provides reusable packaging and templating. Argo CD uses the Helm source to render manifests and then manages synchronization and reconciliation.

---

# Question 55

What is values.yaml?

### Answer

> It contains configurable values used by Helm templates.

---

# Question 56

How do you manage DEV/QA/PROD values?

### Answer

Example:

```text
helm/cart/
├── Chart.yaml
├── templates/
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
└── values-prod.yaml
```

Argo CD selects the appropriate values file for each Application.

---

# Question 57

What is a Helm anti-pattern?

Examples:

```text
huge duplicated values
hardcoded secrets
latest image
environment logic hidden inside templates
unreviewed production values
```

---

# PART 16 — ROUND 14: APP OF APPS

## Question 58

Explain App of Apps.

### Strong Answer

> App of Apps uses a parent Argo CD Application to manage child Applications stored in Git.

Example:

```text
platform-root
 |
 +--> ingress
 +--> external-secrets
 +--> monitoring
 +--> roboshop
```

---

# Question 59

Why use App of Apps?

### Answer

> It provides a bootstrapping pattern and allows platform components and application Applications to be managed declaratively from a root entry point.

---

# Question 60

What is a risk of App of Apps?

### Answer

> The parent becomes a powerful control point. A bad change to the parent or child Application definitions can affect many workloads, so repository protection, review and access controls are important.

---

# PART 17 — ROUND 15: PRODUCTION YAML

## Question 61

What makes a Kubernetes YAML production-ready?

### Strong Answer

Depending on workload:

```text
resource requests
resource limits
readiness probe
liveness/startup probe
securityContext
replica strategy
HPA
PDB
labels
annotations
Service
Ingress
NetworkPolicy
namespace
```

---

# Question 62

What would you include in a production Deployment?

### Answer

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    spec:
      containers:
        - name: cart
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
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
```

Also consider:

```text
securityContext
topology spread
PDB
HPA
```

---

# Question 63

Why are requests important?

### Answer

> Kubernetes scheduling uses resource requests to determine whether a node has sufficient allocatable resources. Requests also affect scheduling and autoscaling behavior.

---

# Question 64

Why are limits important?

### Answer

> Limits establish a ceiling for resource usage according to Kubernetes resource semantics. They should be set based on workload behavior rather than arbitrary numbers.

---

# PART 18 — ROUND 16: PROGRESSIVE DELIVERY

## Question 65

What is progressive delivery?

### Answer

> Progressive delivery gradually exposes a new version to users or traffic instead of switching everyone immediately.

Examples:

```text
canary
blue/green
traffic splitting
```

---

# Question 66

Would you use Argo CD alone for advanced canary traffic management?

### Answer

> Argo CD provides the GitOps synchronization layer. Advanced traffic shifting typically requires an additional progressive-delivery or traffic-management mechanism.

---

# Question 67

What is the benefit?

### Answer

```text
smaller blast radius
early validation
automated rollback
```

---

# PART 19 — ROUND 17: OBSERVABILITY

## Question 68

How do you monitor GitOps?

### Strong Answer

Monitor:

```text
Application sync status
Application health
OutOfSync duration
sync failures
controller health
repo-server errors
cluster connectivity
```

---

# Question 69

How does Prometheus fit?

### Answer

> Prometheus collects metrics from the platform and workloads. Grafana can visualize them and alerting can be built around important operational conditions.

---

# Question 70

How does ELK fit?

### Answer

> ELK provides centralized log collection, processing/search and visualization.

---

# Question 71

An Argo CD sync succeeds but users receive 503. What do you do?

### Strong Answer

> I separate the GitOps control-plane status from application runtime. I would check the Pod health first, then Service endpoints, readiness, Ingress and ALB target health.

---

# PART 20 — ROUND 18: DISASTER RECOVERY

## Question 72

How would you recover if the Argo CD management cluster was lost?

### Strong Answer

```text
rebuild infrastructure
        ↓
install Argo CD
        ↓
restore/bootstrap configuration
        ↓
restore repository credentials
        ↓
register clusters
        ↓
restore Applications/ApplicationSets
        ↓
validate reconciliation
```

---

# Question 73

Can GitOps recover a database?

### Answer

> GitOps can recreate the Kubernetes configuration required to connect to a database, but it does not replace database backups, replication and data recovery mechanisms.

---

# Question 74

What should be backed up?

### Answer

```text
Git repositories
Argo configuration/metadata where needed
secrets
AWS infrastructure state/configuration
databases
artifact availability
DNS configuration
observability configuration
```

---

# PART 21 — ROUND 19: ROBO SHOP PROJECT

## Question 75

Walk me through your RoboShop deployment.

### Strong Answer

> Developers commit application code. Jenkins or GitHub Actions performs build, tests and security checks. The Docker image is built and pushed to ECR. The deployment configuration is stored separately in the GitOps repository using Helm. A Git change updates the image reference. Argo CD detects the change and reconciles it into EKS. Kubernetes runs the services, and external traffic enters through ALB Ingress. Prometheus, Grafana and ELK provide monitoring and logging.

---

# Question 76

Why did you separate application code and GitOps configuration?

### Answer

> It provides separation of concerns and allows deployment configuration to have its own review, promotion and access controls.

---

# Question 77

How would you deploy a new RoboShop version?

### Answer

```text
code change
 ↓
CI
 ↓
test
 ↓
security
 ↓
build image
 ↓
ECR
 ↓
GitOps image update
 ↓
PR
 ↓
approval
 ↓
Argo CD
 ↓
EKS
```

---

# Question 78

A new RoboShop version is crashing. What do you do?

### Strong Answer

First establish impact.

Then:

```bash
argocd app get roboshop
kubectl get pods -n roboshop
kubectl get events -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> --previous -n roboshop
```

If rollback is necessary:

```text
restore known-good Git revision
```

Then investigate the failed release.

---

# Question 79

Would you immediately run `kubectl rollout undo`?

### Strong Answer

> In an emergency it can be used as a temporary operational action, but I would also make Git reflect the intended final state. Otherwise Argo CD may reconcile the deployment back to the Git-defined version.

---

# Question 80

Why is that important?

### Answer

> Because Git is the source of truth. A temporary cluster-only rollback creates drift.

---

# PART 22 — ROUND 20: SENIOR SCENARIOS

## Question 81

You have 50 EKS clusters. How would you manage them?

### Strong Answer

I would consider:

```text
centralized or regional Argo CD control planes
ApplicationSets
Cluster Generator
cluster labels
AppProjects
RBAC
HA
observability
repository organization
```

The final architecture would depend on security boundaries and blast-radius requirements.

---

# Question 82

You have DEV, QA and PROD in separate AWS accounts. How would you design it?

### Strong Answer

```text
DEV account
QA account
PROD account
```

Use:

```text
separate EKS
IAM boundaries
registered clusters
cluster labels
ApplicationSets
AppProjects
protected Git
production approval
```

---

# Question 83

A developer asks for cluster-admin in production. What do you do?

### Strong Answer

> I would not grant cluster-admin by default. I would identify the actual operation they need and provide the minimum required access through Argo CD RBAC, Kubernetes RBAC or a controlled operational process.

---

# Question 84

Someone manually edits production using kubectl. What do you do?

### Strong Answer

```text
1. determine whether it was an emergency
2. assess impact
3. check Argo drift
4. decide desired final state
5. update Git if change should remain
6. allow reconciliation
7. review access/process
```

---

# Question 85

Argo CD keeps reverting a change made by an HPA. What could be happening?

### Answer

> There may be ownership conflict around the replicas field. I would determine whether Argo CD is managing replicas directly while HPA is also modifying them. Then I would establish the correct ownership model and configure differences appropriately rather than disabling self-heal blindly.

---

# Question 86

A developer deletes a YAML file from Git. What happens?

### Answer

> If the resource is managed by Argo CD and pruning is enabled, Argo CD may delete the corresponding live resource after reconciliation. This is why production Git controls are critical.

---

# Question 87

A Git commit accidentally removes a production ApplicationSet. What is your response?

### Strong Answer

```text
stop further propagation if possible
identify affected Applications
restore trusted Git revision
review Argo state
validate clusters
check whether child resources were pruned
restore services
investigate root cause
```

---

# PART 23 — ROUND 21: RAPID FIRE

## Question 88

GitOps source of truth?

> Git.

## Question 89

Argo CD reconciliation engine?

> Application Controller.

## Question 90

Manifest generation component?

> Repo Server.

## Question 91

API/UI component?

> API Server.

## Question 92

Application generation controller?

> ApplicationSet Controller.

## Question 93

Multiple clusters?

> Yes.

## Question 94

EKS?

> Yes.

## Question 95

Helm?

> Yes.

## Question 96

Kustomize?

> Yes.

## Question 97

Drift detection?

> Yes.

## Question 98

Self-healing?

> Yes, when configured.

## Question 99

Pruning?

> Yes, when configured.

## Question 100

Production secrets in plaintext Git?

> No.

## Question 101

Use latest tag?

> Avoid it.

## Question 102

CI direct production kubectl?

> Prefer GitOps reconciliation.

## Question 103

AppProject?

> Application security/configuration boundary.

## Question 104

ApplicationSet?

> Generates Applications.

## Question 105

App of Apps?

> Parent Application managing child Applications.

---

# PART 24 — ROUND 22: INTERVIEWER CHALLENGES

## Challenge 1

### Interviewer

You said Git is the source of truth. But Kubernetes has the real configuration. Why do you call Git the source of truth?

### Strong Response

> Kubernetes contains the live state, but Git contains the approved desired state. The distinction is important. If live state differs from Git, Git remains authoritative for Git-managed resources unless another ownership model explicitly defines otherwise.

---

## Challenge 2

### Interviewer

If Argo CD is down, Kubernetes still works. So why do you call Argo CD critical?

### Strong Response

> It is not necessarily on the request path, so existing workloads can continue. But it is critical to deployment and reconciliation. Without it, new Git changes and automated drift correction are delayed.

---

## Challenge 3

### Interviewer

Why not put all YAML in one repository?

### Strong Response

> A single repository can work, especially for centralized governance, but at scale repository structure should follow ownership, security and lifecycle requirements. Multiple repositories may be better when teams need stronger access boundaries.

---

## Challenge 4

### Interviewer

Why not use only Terraform?

### Strong Response

> Terraform is excellent for infrastructure provisioning. Argo CD is specialized for continuous Kubernetes application reconciliation. I prefer clear ownership: Terraform manages infrastructure and Argo CD manages application resources.

---

## Challenge 5

### Interviewer

Why not use only Helm?

### Strong Response

> Helm handles packaging and templating. Argo CD provides GitOps reconciliation, drift detection, application health and multi-cluster management.

---

## Challenge 6

### Interviewer

Why do you need both Jenkins and Argo CD?

### Strong Response

> They solve different parts of delivery. Jenkins can perform CI activities such as build, test and security scanning. Argo CD provides GitOps CD and continuously reconciles the Kubernetes desired state.

---

## Challenge 7

### Interviewer

What if the Git repository itself is compromised?

### Strong Response

> Git becomes a high-value control plane, so I protect it using MFA, least privilege, protected branches, CODEOWNERS, review requirements, secret scanning and monitoring. I would also have an incident process for credential rotation and restoring a trusted revision.

---

## Challenge 8

### Interviewer

What if the database migration is irreversible?

### Strong Response

> I would not treat Kubernetes rollback as a complete rollback strategy. Database migrations need backward-compatible design, backups and a separate recovery plan.

---

# PART 25 — ROUND 23: DEBUGGING UNDER PRESSURE

## Scenario 1

### Interviewer

Production deployment completed 10 minutes ago. Users report failures. What do you do first?

### Strong Answer

> First I determine impact and whether the new deployment correlates with the incident. Then I check Argo CD status and Kubernetes workload health.

```bash
argocd app get <app>
kubectl get pods -n <namespace>
kubectl get events -n <namespace>
```

Then inspect:

```text
logs
probes
Service
Ingress
ALB
dependencies
```

---

## Scenario 2

### Interviewer

The Pods are Running. Is the application healthy?

### Strong Answer

> Not necessarily. Running only indicates the containers are running. I would check readiness, application health, Service endpoints and actual traffic.

---

## Scenario 3

### Interviewer

Argo says Healthy. Users still cannot connect.

### Strong Answer

> I would investigate resources outside the reported health scope, especially:

```text
Service
EndpointSlice
Ingress
ALB
security groups
DNS
TLS
```

---

## Scenario 4

### Interviewer

Argo says OutOfSync. You have 30 seconds.

### Strong Answer

```bash
argocd app get <app>
argocd app diff <app>
```

Then:

> I identify the resource and field that differ before deciding whether to sync or investigate controller ownership.

---

# PART 26 — ROUND 24: WHITEBOARD DESIGN

## Interviewer

Design a GitOps platform for RoboShop with:

```text
DEV
QA
PROD
```

Three EKS clusters.

### Candidate Whiteboard

```text
                 Application Git
                        |
                        v
                  Jenkins/GHA
                        |
          +-------------+-------------+
          |             |             |
      SonarQube       Trivy       Veracode
          |             |             |
          +-------------+-------------+
                        |
                        v
                       ECR
                        |
                        v
                 GitOps Repository
                        |
                        v
                     Argo CD
                  /      |      \
                 v       v       v
              DEV EKS  QA EKS  PROD EKS
                 |       |       |
                ALB     ALB     ALB
```

Then add:

```text
ApplicationSet
AppProjects
RBAC
Secrets Manager
Prometheus
Grafana
ELK
```

### Strong Explanation

> CI produces a validated immutable artifact. The GitOps repository defines the desired Kubernetes deployment state. Argo CD reconciles that state into the appropriate EKS cluster. ApplicationSet reduces repetitive Application definitions, while AppProjects and RBAC control destinations and permissions.

---

# PART 27 — ROUND 25: ARCHITECTURE TRADE-OFFS

## Question 1

Central Argo CD or one Argo CD per cluster?

### Strong Answer

> Centralized Argo CD reduces management overhead and provides centralized governance, but increases the control-plane blast radius. Per-cluster Argo CD improves isolation but increases operational overhead. I would choose based on cluster count, security boundaries, compliance and failure-domain requirements.

---

## Question 2

One GitOps repository or multiple?

### Strong Answer

> One repository can simplify centralized governance. Multiple repositories can provide stronger team and environment access boundaries. I would base the decision on ownership, scale and security requirements.

---

## Question 3

Automatic production sync or manual?

### Strong Answer

> Automated sync provides speed and consistency, but production may require approval or controlled synchronization depending on risk and compliance. I would not choose automatically for every environment.

---

## Question 4

Tag or digest?

### Strong Answer

> Tags are easier to operate, while digests provide stronger immutability and reproducibility. For high-assurance production, I prefer immutable references such as image digests where practical.

---

# PART 28 — FINAL MOCK INCIDENT

## Incident

At 10:00 AM:

```text
Developer merges cart version 2.0
```

CI:

```text
tests pass
SonarQube pass
Trivy pass
Veracode pass
image pushed to ECR
```

GitOps:

```text
image updated
PR approved
merge completed
```

Argo CD:

```text
Sync = Synced
Health = Degraded
```

Users:

```text
HTTP 503
```

---

## Interviewer

Walk me through the incident.

### Strong Candidate Response

> The CI and GitOps parts appear to have completed successfully, but Argo CD health is Degraded and users receive 503, so I would move from deployment state into runtime troubleshooting.

First:

```bash
argocd app get cart
```

Then:

```bash
kubectl get pods -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

If Pods are unhealthy:

```bash
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> --previous -n roboshop
```

If Pods are healthy:

```bash
kubectl get svc -n roboshop
kubectl get endpointslice -n roboshop
kubectl get ingress -n roboshop
```

Then inspect ALB target health and AWS Load Balancer Controller.

If the new version is confirmed as the cause and impact is significant, I would restore the last known-good GitOps revision, validate recovery and then investigate version 2.0.

---

# PART 29 — INTERVIEWER FOLLOW-UP ON INCIDENT

## Interviewer

Why not just restart the Pods?

### Strong Answer

> Restarting Pods may temporarily hide a symptom but does not identify the root cause. I would first determine whether the issue is application, configuration, image, resource, readiness, Service or network related.

---

## Interviewer

Why not rollback immediately?

### Strong Answer

> If the new deployment clearly correlates with a severe production incident, rollback can be appropriate. But I still want enough evidence to ensure I understand the failure and to avoid masking a deeper infrastructure issue.

---

## Interviewer

How do you rollback in GitOps?

### Strong Answer

> For the durable desired state, I prefer reverting the GitOps change to the known-good image/version and letting Argo CD reconcile that state. A direct emergency rollback can be used when necessary, but Git should ultimately represent the intended final state.

---

# PART 30 — FINAL MOCK: 10 HARD QUESTIONS

## Hard Question 1

Why does Argo CD need continuous reconciliation if Git changes only occasionally?

### Strong Answer

Because the cluster can change independently of Git. Continuous reconciliation detects and corrects drift.

---

## Hard Question 2

What happens if two controllers manage the same field?

### Strong Answer

They can fight each other, producing continuous reconciliation loops. Ownership must be clearly defined.

---

## Hard Question 3

What happens if Git says replicas 3 but HPA wants 10?

### Strong Answer

There is a potential ownership conflict around replicas. I would design the workload so HPA owns scaling and avoid Argo continuously forcing a static replica count over it.

---

## Hard Question 4

What happens if the GitOps repo is unavailable for an hour?

### Strong Answer

Existing workloads generally continue. New desired state cannot be retrieved until repository access is restored. The incident affects deployment/reconciliation rather than necessarily application traffic.

---

## Hard Question 5

What happens if ECR is unavailable?

### Strong Answer

Existing Pods may continue if their images are already available locally. New Pods requiring unavailable images may fail to start.

---

## Hard Question 6

What happens if the EKS API is unavailable?

### Strong Answer

Existing workloads may continue depending on the failure, but scheduling, deployments, Kubernetes control operations and Argo reconciliation are affected.

---

## Hard Question 7

Can Argo CD guarantee application availability?

### Answer

> No. It manages desired Kubernetes state. Availability depends on application design, infrastructure, dependencies, capacity, networking and operational controls.

---

## Hard Question 8

Can GitOps replace monitoring?

### Answer

> No. GitOps manages desired state and reconciliation. Monitoring tells us whether the resulting system is healthy and meeting operational requirements.

---

## Hard Question 9

Can GitOps replace incident response?

### Answer

> No. GitOps improves repeatability and recovery, but incidents still require detection, diagnosis, mitigation, communication and root-cause analysis.

---

## Hard Question 10

What is the most dangerous GitOps anti-pattern?

### Strong Answer

> Ambiguous ownership combined with excessive privileges. If multiple systems can modify the same production resources and users have broad cluster permissions, reconciliation and security become difficult to control.

---

# PART 31 — INTERVIEWER SCORING RUBRIC

Score each area from:

```text
1 = weak
2 = basic
3 = acceptable
4 = strong
5 = production-level
```

## GitOps Fundamentals

```text
1  2  3  4  5
```

Can the candidate explain:

```text
desired state
actual state
reconciliation
drift
pull model
source of truth
```

---

## Argo CD

```text
1  2  3  4  5
```

Can the candidate explain:

```text
Application Controller
Repo Server
API Server
Application
Project
ApplicationSet
sync
health
```

---

## Kubernetes

```text
1  2  3  4  5
```

Can the candidate troubleshoot:

```text
Pods
Deployment
Service
Ingress
probes
resources
HPA
events
```

---

## AWS/EKS

```text
1  2  3  4  5
```

Can the candidate explain:

```text
EKS
ECR
IAM
ALB
Route 53
Secrets Manager
```

---

## Security

```text
1  2  3  4  5
```

Can the candidate explain:

```text
RBAC
AppProjects
least privilege
secrets
Git security
artifact security
```

---

## Troubleshooting

```text
1  2  3  4  5
```

Can the candidate:

```text
form a hypothesis
collect evidence
narrow scope
identify root cause
mitigate
restore
prevent recurrence
```

---

## Architecture

```text
1  2  3  4  5
```

Can the candidate discuss:

```text
multi-cluster
multi-account
HA
DR
security boundaries
trade-offs
```

---

# PART 32 — SCORE INTERPRETATION

## 32–40

Strong production-level GitOps interview readiness.

Candidate can:

```text
design
troubleshoot
explain trade-offs
```

---

## 24–31

Good readiness.

Improve:

```text
architecture
scenario troubleshooting
security
multi-cluster
```

---

## 16–23

Fundamentals exist, but practical depth needs improvement.

Focus on:

```text
hands-on labs
failure scenarios
production YAML
commands
```

---

## Below 16

Return to:

```text
GitOps Fundamentals
Argo CD Architecture
Applications
Sync/Reconciliation
ApplicationSets
Multi-Cluster
Troubleshooting
```

Then repeat this mock interview.

---

# PART 33 — SELF-PRACTICE MODE

For each question:

### Step 1

Read only:

```text
Interviewer Question
```

### Step 2

Answer aloud.

### Step 3

Compare with:

```text
Strong Candidate Answer
```

### Step 4

Ask yourself:

```text
Did I explain why?
Did I explain how?
Did I give production context?
Did I mention security?
Did I explain failure?
```

### Step 5

Repeat without reading the answer.

---

# PART 34 — 15-MINUTE QUICK MOCK

If time is limited, answer these 15 questions:

```text
1. What is GitOps?
2. Why use Argo CD?
3. Explain desired vs actual state.
4. Explain Argo CD architecture.
5. What is Application Controller?
6. What is an Application?
7. What is ApplicationSet?
8. How does multi-cluster work?
9. How do you secure production?
10. How do you manage secrets?
11. CI vs CD?
12. How do you troubleshoot OutOfSync?
13. How do you troubleshoot Degraded?
14. Explain RoboShop architecture.
15. Explain your rollback strategy.
```

---

# PART 35 — 30-MINUTE INTERVIEW

Use:

```text
5 min  → introduction
5 min  → GitOps/Argo fundamentals
5 min  → Kubernetes/AWS
5 min  → troubleshooting
5 min  → architecture
5 min  → RoboShop/project discussion
```

---

# PART 36 — 60-MINUTE SENIOR INTERVIEW

Suggested sequence:

```text
0–5    Introduction
5–15   GitOps fundamentals
15–25  Argo CD architecture
25–35  Kubernetes/AWS
35–45  Production troubleshooting
45–55  Architecture/system design
55–60  Candidate questions
```

---

# PART 37 — QUESTIONS YOU SHOULD ASK THE INTERVIEWER

At the end of the interview, ask relevant questions.

## Question 1

> How is your current Kubernetes/GitOps platform structured across environments and clusters?

## Question 2

> Are application teams responsible for their own deployment configuration, or is there a centralized platform team?

## Question 3

> How do you handle production deployment approvals?

## Question 4

> What are the biggest reliability challenges in the current platform?

## Question 5

> How do you currently manage secrets and workload identity?

## Question 6

> What observability stack do you use for Kubernetes and application monitoring?

## Question 7

> How is disaster recovery handled for the Kubernetes platform?

These questions demonstrate platform-level thinking.

---

# PART 38 — WHAT NOT TO SAY

Avoid:

```text
"I know everything about Argo CD."

"Argo CD automatically fixes every problem."

"GitOps means Jenkins is not needed."

"Terraform cannot deploy Kubernetes."

"Argo CD guarantees zero downtime."

"Running Pods means the application is healthy."

"Rollback always fixes production."

"Base64 means encrypted."

"Cluster-admin is easier."

"Latest is fine."

"I just run kubectl."

```

Replace confidence without evidence with:

```text
"In that situation, I would first check..."
```

This demonstrates troubleshooting maturity.

---

# PART 39 — STRONG TROUBLESHOOTING LANGUAGE

Instead of:

> I think the Pod is broken.

Say:

> I would first inspect Pod status, events and logs to determine whether the failure is caused by the application, image, configuration, resources or dependency.

Instead of:

> I would restart it.

Say:

> I would collect evidence before restarting it so that we do not lose useful failure information.

Instead of:

> I would rollback.

Say:

> If the new deployment is confirmed as the cause and the impact is significant, I would restore the known-good Git revision and allow Argo CD to reconcile it.

---

# PART 40 — STRONG ARCHITECTURE LANGUAGE

Use phrases such as:

```text
clear ownership
least privilege
separation of concerns
source of truth
desired state
continuous reconciliation
failure domain
blast radius
immutable artifact
controlled promotion
auditability
defense in depth
operational boundary
```

These communicate senior-level thinking when used accurately.

---

# PART 41 — FINAL ROBO SHOP MOCK

## Interviewer

Explain your project from code commit to production traffic.

### Candidate

```text
Developer
   |
   v
Application Git
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
   |
   v
Deployment
   |
   v
Service
   |
   v
Ingress
   |
   v
AWS ALB
   |
   v
Users
```

Observability:

```text
Prometheus → Metrics
Grafana    → Dashboards
ELK        → Logs
```

Infrastructure:

```text
Terraform → AWS infrastructure
```

Secrets:

```text
AWS Secrets Manager
```

---

# PART 42 — INTERVIEWER FINAL CHALLENGE

### Interviewer

You have explained the architecture. Now convince me you understand production operations.

### Strong Candidate Answer

> I would not consider the platform production-ready only because deployment succeeds. I would look at the complete lifecycle: secure source control, validated immutable artifacts, controlled promotion, GitOps reconciliation, least-privilege cluster access, workload health, observability, rollback, drift management, disaster recovery and operational ownership.
>
> I would also make sure Terraform and Argo CD have clear ownership boundaries, production Git changes are reviewed, secrets are externalized, clusters are isolated appropriately, and there is a tested recovery process.
>
> When an incident happens, I would start with evidence and narrow the failure domain rather than immediately changing resources. That is important because a GitOps platform can report a successful sync while the application itself is unhealthy.

---

# PART 43 — FINAL 60-SECOND ANSWER

If the interviewer asks:

> What is your understanding of production GitOps?

Answer:

> Production GitOps is more than storing Kubernetes YAML in Git. Git represents the approved desired state, CI produces secure immutable artifacts, and a reconciliation controller such as Argo CD continuously compares that desired state with Kubernetes. Applications are managed through Applications, Projects and ApplicationSets, while multi-cluster deployments can be controlled through registered EKS clusters and cluster labels. Security is implemented through Git protection, RBAC, AppProjects, least privilege and external secret management. Production also requires observability, drift handling, controlled promotion, rollback, HA and disaster recovery. The important principle is clear ownership between Git, CI, Argo CD, Kubernetes controllers and infrastructure tools.

---

# PART 44 — FINAL 30-SECOND ROBO SHOP ANSWER

> In my RoboShop environment, CI builds and validates the microservices, performs SonarQube, Trivy and Veracode checks, creates Docker images and pushes them to ECR. Deployment configuration is maintained separately in Git using Helm. Argo CD reconciles that desired state into EKS. ApplicationSets can manage multiple environments or clusters, AppProjects provide boundaries, ALB Ingress exposes HTTP traffic, and Prometheus, Grafana and ELK provide observability. Terraform manages the AWS infrastructure while Argo CD manages application Kubernetes resources.

---

# PART 45 — FINAL MENTAL MODEL

Memorize the flow, not individual sentences:

```text
                    SOURCE
                      |
                  Application
                     Git
                      |
                      v
                     CI
                      |
        +-------------+-------------+
        |             |             |
      Tests        Security       Build
        |             |             |
        +-------------+-------------+
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
             +--------+--------+
             |                 |
       Desired State       Live State
             |                 |
             +--------+--------+
                      |
                  Compare
                      |
                  Reconcile
                      |
                      v
                     EKS
                      |
          +-----------+-----------+
          |           |           |
      Deployment    Service     Ingress
                                  |
                                  v
                                 ALB
                                  |
                                  v
                                Users

Feedback:
Prometheus → Metrics
Grafana    → Visualization
ELK        → Logs
```

---

# PART 46 — FINAL PRODUCTION PRINCIPLES

Remember these 15 principles:

```text
1. Git should represent desired state.
2. Build once and promote immutable artifacts.
3. CI and CD should have clear responsibilities.
4. Argo CD should reconcile Kubernetes state.
5. Avoid direct production changes.
6. Define resource ownership clearly.
7. Use least privilege.
8. Protect production Git.
9. Keep secrets out of plaintext Git.
10. Use AppProjects for boundaries.
11. Use ApplicationSets for fleet-scale Applications.
12. Monitor both GitOps and runtime health.
13. Treat rollback and DR as separate capabilities.
14. Do not confuse successful sync with application availability.
15. Always troubleshoot from evidence.
```

---

# PART 47 — FINAL SELF-ASSESSMENT

Before considering the GitOps section interview-ready, you should be able to explain without notes:

```text
[ ] What GitOps is
[ ] Why Git is the source of truth
[ ] Desired vs actual state
[ ] Pull model
[ ] Reconciliation
[ ] Drift
[ ] CI vs GitOps CD
[ ] Argo CD architecture
[ ] Application Controller
[ ] Repo Server
[ ] API Server
[ ] ApplicationSet Controller
[ ] Application
[ ] AppProject
[ ] Sync
[ ] Refresh
[ ] Health
[ ] OutOfSync
[ ] SelfHeal
[ ] Prune
[ ] Sync waves
[ ] Hooks
[ ] Rollback
[ ] Helm
[ ] Kustomize
[ ] ApplicationSet
[ ] Multi-cluster
[ ] Multi-account EKS
[ ] RBAC
[ ] SSO
[ ] Secrets
[ ] ECR
[ ] ALB
[ ] Terraform boundary
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] Production YAML
[ ] Troubleshooting
[ ] DR
[ ] HA
[ ] RoboShop architecture
```

---

# PART 48 — FINAL INTERVIEW READINESS TEST

You are ready to move forward when you can answer these five questions confidently:

## 1. Explain GitOps in 60 seconds.

## 2. Draw and explain Argo CD architecture.

## 3. Troubleshoot an OutOfSync/Degraded application without guessing.

## 4. Design centralized Argo CD for multiple EKS clusters.

## 5. Explain your RoboShop GitOps implementation from developer commit to production traffic.

If you can do those five things clearly, you have moved beyond memorizing Argo CD commands and can discuss GitOps as a production engineering system.

---

# FINAL SUMMARY

The complete GitOps interview story is:

```text
Developer
   ↓
Application Git
   ↓
CI
   ↓
Test + Security
   ↓
Docker Image
   ↓
ECR
   ↓
GitOps Repository
   ↓
Argo CD
   ↓
Desired vs Live Comparison
   ↓
Reconciliation
   ↓
EKS
   ↓
Kubernetes
   ↓
ALB
   ↓
Users
```

And the operational feedback loop is:

```text
Users
   ↓
Application
   ↓
Metrics / Logs
   ↓
Prometheus / Grafana / ELK
   ↓
Engineer
   ↓
Git Change
   ↓
Argo CD
   ↓
Reconciliation
```

That is the core production GitOps mental model.

# End of GitOps Mock Interview
