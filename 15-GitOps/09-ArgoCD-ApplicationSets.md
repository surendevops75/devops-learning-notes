# 09-ArgoCD-ApplicationSets

## 1. Purpose

ApplicationSet is one of the most important Argo CD capabilities for production GitOps at scale.

A single Argo CD `Application` manages one desired application deployment.

An `ApplicationSet` manages the generation of many Argo CD `Application` resources from a template and one or more generators.

Conceptually:

```text
Application
    |
    +--> One application definition
    +--> One destination
    +--> One source configuration
```

Whereas:

```text
ApplicationSet
    |
    +--> Generator
    |
    +--> Application template
    |
    +--> Application
    +--> Application
    +--> Application
    +--> ...
```

This file focuses on production use with:

- Kubernetes
- AWS EKS
- Multiple environments
- Multiple clusters
- Helm
- Kustomize
- RoboShop
- Centralized Argo CD
- Cluster labels
- Environment promotion
- Multi-account AWS environments
- Troubleshooting
- Security
- Interview preparation

---

# 2. Why ApplicationSet Exists

Without ApplicationSet, an organization managing many services and clusters may have hundreds of repetitive `Application` manifests.

For example:

```text
20 services
x
3 environments
x
3 clusters
=
180 Application manifests
```

Maintaining those manually creates:

```text
Duplication
Configuration drift
Naming mistakes
Destination mistakes
Slow onboarding
High maintenance
```

ApplicationSet solves this by generating Applications from data.

---

# 3. Application vs ApplicationSet

## Application

An Application is the actual Argo CD object that represents:

```text
One desired application
+
One source
+
One destination
+
One sync policy
```

Example:

```text
cart-prod
```

deploys:

```text
cart
```

to:

```text
EKS production
```

---

## ApplicationSet

An ApplicationSet is a higher-level generator.

Example:

```text
ApplicationSet
       |
       +--> cart-dev
       +--> cart-qa
       +--> cart-prod
```

The generated Applications are then managed by Argo CD.

---

# 4. Critical Mental Model

Do not think:

```text
ApplicationSet = another deployment controller
```

Think:

```text
ApplicationSet
=
Application generator/controller
```

It creates and updates `Application` resources.

Then:

```text
Application Controller
```

reconciles those Applications and deploys the workloads.

---

# 5. ApplicationSet Architecture

```text
                    Git Repository
                          |
                          v
                 ApplicationSet YAML
                          |
                          v
                ApplicationSet Controller
                          |
             +------------+------------+
             |            |            |
             v            v            v
        Application   Application   Application
             |            |            |
             +------------+------------+
                          |
                          v
                  Argo CD Application
                     Controller
                          |
                          v
                    Kubernetes API
                          |
                          v
                         EKS
```

There are therefore two important reconciliation layers:

```text
ApplicationSet Controller
        |
        v
Application resources

Application Controller
        |
        v
Kubernetes resources
```

---

# 6. Two Reconciliation Loops

This distinction is extremely important for interviews.

## Loop 1

ApplicationSet controller:

```text
Generator input
     |
     v
Desired Applications
     |
     v
Create/update/delete Applications
```

## Loop 2

Application controller:

```text
Application desired state
     |
     v
Compare live cluster
     |
     v
Sync/reconcile resources
```

---

# 7. ApplicationSet Controller

The ApplicationSet controller watches:

```text
ApplicationSet resources
```

It evaluates generators and templates.

For each generated parameter set, it produces an Application.

Example:

```text
Generator result:
environment=prod

Template:
name=cart-{{environment}}

Generated:
cart-prod
```

---

# 8. Generator

A generator provides the data used by the ApplicationSet template.

Common generators include:

```text
List
Git
Directory
Cluster
Matrix
Merge
Pull Request
SCM-related generators
```

Each serves a different use case.

---

# 9. Template

The ApplicationSet template defines the Application structure.

Example:

```yaml
template:
  metadata:
    name: '{{name}}'
  spec:
    project: '{{project}}'
    source:
      repoURL: '{{repoURL}}'
      targetRevision: main
      path: '{{path}}'
    destination:
      server: '{{server}}'
      namespace: '{{namespace}}'
```

Generator values are substituted into the template.

---

# 10. Template Parameters

A generator may produce:

```text
name
environment
namespace
server
repoURL
path
```

The template consumes them.

Conceptually:

```text
Generator
environment=prod
namespace=roboshop
cluster=eks-prod

        |

Template

name=cart-{{environment}}

        |

Generated Application

cart-prod
```

---

# 11. List Generator

The List generator is one of the easiest ways to start.

It provides explicit parameter objects.

Example:

```yaml
generators:
  - list:
      elements:
        - environment: dev
        - environment: qa
        - environment: prod
```

The ApplicationSet generates one Application per element.

---

# 12. Simple List Generator

Complete example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-environments
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - environment: dev
          - environment: qa
          - environment: prod

  template:
    metadata:
      name: 'roboshop-{{environment}}'

    spec:
      project: roboshop

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: 'environments/{{environment}}'

      destination:
        name: '{{environment}}'
        namespace: roboshop
```

This generates:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

---

# 13. Why List Generator Is Useful

List is useful when:

```text
The environment set is known
Cluster mappings are explicit
You want simple configuration
You need easy review
```

It is especially useful for:

```text
dev
qa
prod
```

when there are only a few environments.

---

# 14. List Generator With Multiple Fields

Example:

```yaml
generators:
  - list:
      elements:
        - environment: dev
          cluster: eks-dev
          namespace: roboshop-dev

        - environment: qa
          cluster: eks-qa
          namespace: roboshop-qa

        - environment: prod
          cluster: eks-prod
          namespace: roboshop
```

This gives each generated Application explicit configuration.

---

# 15. Production Multi-Environment List Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - environment: dev
            cluster: eks-dev
            namespace: roboshop-dev
            valuesFile: values/dev.yaml

          - environment: qa
            cluster: eks-qa
            namespace: roboshop-qa
            valuesFile: values/qa.yaml

          - environment: prod
            cluster: eks-prod
            namespace: roboshop
            valuesFile: values/prod.yaml

  template:
    metadata:
      name: 'roboshop-{{environment}}'

    spec:
      project: roboshop-{{environment}}

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: helm/roboshop

        helm:
          valueFiles:
            - '{{valuesFile}}'

      destination:
        name: '{{cluster}}'
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true
```

---

# 16. Important Production Note

The exact ApplicationSet templating syntax and available fields can vary by Argo CD version.

Always validate manifests against the installed Argo CD version.

Do not copy production YAML blindly between major Argo CD releases.

---

# 17. Generated Application Example

Given:

```yaml
environment: prod
cluster: eks-prod
namespace: roboshop
```

the generated Application conceptually becomes:

```yaml
metadata:
  name: roboshop-prod

spec:
  project: roboshop-prod

  destination:
    name: eks-prod
    namespace: roboshop
```

ApplicationSet removes repetitive boilerplate.

---

# 18. Git Generator

The Git generator obtains parameters from a Git repository.

It can be useful when directory structure itself represents deployment units.

Example:

```text
environments/
├── dev/
├── qa/
└── prod/
```

ApplicationSet can discover those directories and generate Applications.

---

# 19. Git Directory Generator

Example repository:

```text
gitops/
└── applications/
    ├── cart/
    ├── user/
    ├── payment/
    └── catalogue/
```

A Git directory generator can discover application directories.

Conceptually:

```text
Git
 |
 +--> cart
 +--> user
 +--> payment
 +--> catalogue
 |
 v
ApplicationSet
 |
 v
Applications
```

---

# 20. Why Git Generator Is Powerful

Instead of manually adding:

```yaml
- name: cart
- name: user
- name: payment
```

the repository structure can become the source of application inventory.

Adding:

```text
applications/inventory/
```

can result in a new generated Application, depending on the configured generator and template.

---

# 21. Git Generator With Files

The Git generator can also use repository files as data.

Example concept:

```text
clusters/
├── dev.yaml
├── qa.yaml
└── prod.yaml
```

The generator reads Git content and exposes parameters for the ApplicationSet template.

This allows configuration-driven Application generation.

---

# 22. Directory Generator vs List Generator

### List

Explicit:

```text
dev
qa
prod
```

### Directory

Dynamic:

```text
whatever directories exist
```

Use List when the inventory is intentionally controlled in the ApplicationSet.

Use Git directory discovery when Git structure is the source of the application inventory.

---

# 23. Cluster Generator

The Cluster generator is extremely important for multi-cluster Argo CD.

It discovers clusters registered with Argo CD.

Conceptually:

```text
Argo CD
 |
 +--> EKS DEV
 +--> EKS QA
 +--> EKS PROD
```

ApplicationSet can select clusters and generate Applications for them.

---

# 24. Cluster Generator Mental Model

```text
Registered clusters
        |
        v
Cluster generator
        |
        v
Cluster metadata
        |
        v
Application template
        |
        v
Applications
```

Cluster metadata may include:

```text
server
name
labels
annotations
```

depending on the configuration.

---

# 25. Cluster Labels

Labels are extremely powerful.

Example:

```text
environment=prod
region=ap-south-1
team=platform
tenant=payments
```

Then an ApplicationSet can target:

```text
environment=prod
```

instead of hard-coding every cluster.

---

# 26. Why Cluster Labels Matter

Suppose an organization has:

```text
eks-dev-1
eks-dev-2
eks-qa-1
eks-prod-1
eks-prod-2
```

Instead of manually listing:

```text
eks-prod-1
eks-prod-2
```

label them:

```text
environment=prod
```

ApplicationSet can select the group.

---

# 27. Multi-Cluster Cluster Generator

Conceptually:

```yaml
generators:
  - clusters:
      selector:
        matchLabels:
          environment: prod
```

Then the template uses cluster metadata.

Example:

```yaml
destination:
  server: '{{server}}'
```

This creates an Application for each matching cluster.

---

# 28. Central Argo CD Architecture

A production organization may use:

```text
                     Git
                      |
                      v
               Central Argo CD
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
    EKS DEV         EKS QA        EKS PROD
```

ApplicationSet can automate deployment across the target clusters.

---

# 29. Centralized Argo CD Benefits

```text
Central visibility
Central policy
Central application inventory
Central RBAC
Consistent deployment model
Reduced duplicate Argo CD installations
```

But centralized control also creates:

```text
Larger blast radius
Central dependency
Higher security requirements
```

---

# 30. Multi-Cluster Security

Central Argo CD requires credentials to access target clusters.

Therefore:

```text
Argo CD
    |
    +--> Cluster credential A
    +--> Cluster credential B
    +--> Cluster credential C
```

Each credential should have least privilege.

Do not grant cluster-admin automatically unless required.

---

# 31. Cluster Generator With Environment Labels

Example:

```text
eks-dev:
  environment=dev

eks-qa:
  environment=qa

eks-prod:
  environment=prod
```

ApplicationSet:

```yaml
selector:
  matchLabels:
    environment: prod
```

Result:

```text
Applications only for production clusters
```

---

# 32. Region Labels

Example:

```text
region=ap-south-1
region=ap-southeast-1
```

An ApplicationSet can use labels to select clusters by region.

This becomes useful for:

```text
Disaster recovery
Regional deployment
Latency-sensitive applications
Data residency
```

---

# 33. Cluster Generator for DR

Suppose:

```text
Primary:
environment=prod
role=primary

DR:
environment=prod
role=dr
```

An ApplicationSet can target:

```text
role=primary
```

or:

```text
role=dr
```

depending on the deployment strategy.

---

# 34. Matrix Generator

The Matrix generator combines two generators.

Conceptually:

```text
Generator A
    x
Generator B
    |
    v
Matrix
    |
    v
All combinations
```

Example:

```text
Environments:
dev
qa
prod

Applications:
cart
user
payment
```

Matrix result:

```text
cart-dev
cart-qa
cart-prod

user-dev
user-qa
user-prod

payment-dev
payment-qa
payment-prod
```

---

# 35. Why Matrix Is Powerful

Matrix avoids manually writing:

```text
3 environments
x
20 services
=
60 Applications
```

Instead:

```text
Service generator
      x
Environment generator
      |
      v
60 Applications
```

This is a major scaling pattern.

---

# 36. Matrix Mental Model

```text
Applications
    |
    +--> cart
    +--> user
    +--> payment

        X

Environments
    |
    +--> dev
    +--> qa
    +--> prod

        |

Generated Applications
```

---

# 37. Matrix Production Use

A practical RoboShop design:

```text
Service directories
        x
Environment directories
        |
        v
ApplicationSet
        |
        v
Applications
```

This works well when:

```text
Most services share a common deployment model
```

---

# 38. Matrix Risk

Matrix can create a large number of Applications quickly.

Example:

```text
50 services
x
10 clusters
x
4 environments
=
2,000 Applications
```

At this scale:

```text
ApplicationSet design
Argo CD sizing
Repository structure
Application boundaries
```

must be carefully planned.

---

# 39. Merge Generator

The Merge generator combines generator outputs using matching keys.

Conceptually:

```text
Base application inventory
        +
Cluster-specific overrides
        |
        v
Merged parameters
```

Useful when:

```text
Most applications share defaults
Some environments/clusters require exceptions
```

---

# 40. Merge Generator Example Concept

Base:

```text
service=cart
replicas=3
```

Cluster override:

```text
cluster=prod-dr
replicas=5
```

Merge result:

```text
service=cart
replicas=5
cluster=prod-dr
```

This avoids duplicating the entire configuration.

---

# 41. Pull Request Generator

The Pull Request generator can create temporary Applications for PRs.

Conceptual workflow:

```text
Developer opens PR
        |
        v
ApplicationSet detects PR
        |
        v
Temporary Application
        |
        v
Preview environment
        |
        v
Testing
        |
        v
PR merged/closed
        |
        v
Temporary Application removed
```

This is useful for:

```text
Preview environments
Integration testing
UI testing
Feature validation
```

---

# 42. PR Generator Security

Preview environments can be dangerous if they:

```text
Expose production data
Have excessive AWS permissions
Create public ALBs
Access sensitive services
```

Use:

```text
Restricted namespaces
Restricted cluster
Restricted IAM
NetworkPolicy
Non-production data
```

---

# 43. SCM Generators

SCM-related generators can discover repositories or repository structures from source-control systems.

They can be useful in large organizations with:

```text
Many repositories
Many teams
Standard repository conventions
```

But automatic discovery can create unexpected Applications.

Use filters and governance.

---

# 44. Automatic Repository Discovery Risk

Suppose:

```text
500 repositories
```

and a generator discovers all of them.

You could accidentally create:

```text
500 Applications
```

Therefore use:

```text
Repository filters
Organization rules
Team labels
Naming conventions
Explicit inclusion
```

---

# 45. Template Metadata

Example:

```yaml
template:
  metadata:
    name: '{{name}}'
    labels:
      environment: '{{environment}}'
      managed-by: applicationset
```

Labels help operations.

---

# 46. Template Spec

Typical template fields include:

```text
project
source
destination
syncPolicy
ignoreDifferences
revisionHistoryLimit
```

The template should represent the standard deployment policy.

---

# 47. Dynamic Names

Example:

```yaml
name: 'roboshop-{{environment}}'
```

Generated:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

For multi-cluster:

```yaml
name: 'roboshop-{{environment}}-{{cluster}}'
```

Use names that remain predictable and searchable.

---

# 48. Dynamic Namespace

Example:

```yaml
destination:
  namespace: 'roboshop-{{environment}}'
```

Result:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

Namespace creation must be handled appropriately.

---

# 49. Dynamic Helm Values

Example concept:

```yaml
helm:
  valueFiles:
    - 'values/{{environment}}.yaml'
```

This allows:

```text
dev -> values/dev.yaml
qa -> values/qa.yaml
prod -> values/prod.yaml
```

---

# 50. Dynamic Git Paths

Example:

```yaml
path: 'applications/{{service}}/{{environment}}'
```

Generated Application:

```text
cart/dev
cart/qa
cart/prod
```

This is a clean repository-driven pattern.

---

# 51. Git Directory Structure for Matrix

Example:

```text
gitops/
└── applications/
    ├── cart/
    │   ├── dev/
    │   ├── qa/
    │   └── prod/
    │
    ├── user/
    │   ├── dev/
    │   ├── qa/
    │   └── prod/
    │
    └── payment/
        ├── dev/
        ├── qa/
        └── prod/
```

ApplicationSet can generate from the structure.

---

# 52. Helm-Based Matrix Repository

Alternative:

```text
gitops/
├── charts/
│   ├── cart/
│   ├── user/
│   └── payment/
│
└── values/
    ├── cart/
    │   ├── dev.yaml
    │   ├── qa.yaml
    │   └── prod.yaml
    ├── user/
    └── payment/
```

ApplicationSet generates:

```text
service
+
environment
```

and selects the appropriate values file.

---

# 53. ApplicationSet for RoboShop Services

Desired:

```text
cart
user
catalogue
payment
inventory
orders
shipping
notification
frontend
```

Across:

```text
dev
qa
prod
```

ApplicationSet can produce:

```text
cart-dev
cart-qa
cart-prod
...
frontend-dev
frontend-qa
frontend-prod
```

---

# 54. RoboShop Matrix Architecture

```text
                     GitOps Repo
                          |
             +------------+------------+
             |                         |
        Services                    Environments
             |                         |
   cart/user/payment/...          dev/qa/prod
             |                         |
             +------------+------------+
                          |
                       Matrix
                          |
                          v
                   ApplicationSet
                          |
                          v
              Generated Applications
                          |
                          v
                       Argo CD
                          |
                          v
                         EKS
```

---

# 55. Multi-Cluster RoboShop

Suppose:

```text
eks-dev
eks-qa
eks-prod
eks-prod-dr
```

Labels:

```text
environment=dev
environment=qa
environment=prod
environment=prod
```

and:

```text
region=ap-south-1
region=ap-south-1
region=ap-south-1
region=ap-southeast-1
```

ApplicationSet can target based on these labels.

---

# 56. Centralized Argo CD With Multiple EKS Clusters

```text
                          Git
                           |
                           v
                  Central Argo CD
                           |
                ApplicationSet
                           |
         +-----------------+-----------------+
         |                 |                 |
         v                 v                 v
     EKS-DEV            EKS-QA            EKS-PROD
         |                 |                 |
       Pods              Pods              Pods
```

The management cluster runs Argo CD.

The target clusters run applications.

---

# 57. Management Cluster vs Target Cluster

Management cluster:

```text
Argo CD
ApplicationSets
Projects
Repository credentials
Cluster registrations
```

Target clusters:

```text
RoboShop workloads
Services
Ingress
HPA
ConfigMaps
Secrets
```

The management cluster does not have to host the workloads it manages.

---

# 58. Centralized Argo CD Trust Boundary

```text
Git
 |
 v
Argo CD
 |
 +--> credentials for EKS DEV
 +--> credentials for EKS QA
 +--> credentials for EKS PROD
```

This makes Argo CD a highly privileged control-plane component.

Protect it accordingly.

---

# 59. Multi-Account AWS

A production organization may have:

```text
AWS Account DEV
AWS Account QA
AWS Account PROD
```

with:

```text
EKS DEV
EKS QA
EKS PROD
```

A centralized Argo CD can manage them if appropriate cross-account access is configured.

---

# 60. Multi-Account Access

The important principle is:

```text
Argo CD must authenticate to each target Kubernetes API
```

This can involve:

```text
EKS cluster access
AWS IAM
STS
Kubernetes credentials
RBAC
```

The exact design depends on the organization's EKS authentication architecture.

---

# 61. Least Privilege Across Clusters

Do not assume:

```text
one global cluster-admin credential
```

is necessary.

Instead design:

```text
Argo CD
 |
 +--> limited access to DEV
 +--> limited access to QA
 +--> controlled access to PROD
```

Production access should be especially restricted.

---

# 62. Environment-Based Argo CD Projects

ApplicationSet can generate Applications assigned to different Projects:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

Projects can enforce:

```text
Allowed repositories
Allowed clusters
Allowed namespaces
Resource restrictions
```

This creates a strong security boundary.

---

# 63. Production ApplicationSet With Projects

Conceptual:

```yaml
template:
  spec:
    project: 'roboshop-{{environment}}'
```

Generator:

```text
dev -> roboshop-dev
qa  -> roboshop-qa
prod -> roboshop-prod
```

This prevents a generated production Application from accidentally using a development Project.

---

# 64. Production ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-services
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - service: cart
            environment: dev
            cluster: eks-dev
            namespace: roboshop-dev

          - service: cart
            environment: qa
            cluster: eks-qa
            namespace: roboshop-qa

          - service: cart
            environment: prod
            cluster: eks-prod
            namespace: roboshop

  template:
    metadata:
      name: '{{service}}-{{environment}}'

    spec:
      project: 'roboshop-{{environment}}'

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: 'helm/{{service}}'

      destination:
        name: '{{cluster}}'
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true
```

---

# 65. Why This Is Better Than Three Applications

Without ApplicationSet:

```text
cart-dev.yaml
cart-qa.yaml
cart-prod.yaml
```

With ApplicationSet:

```text
One generator
+
One template
```

This reduces repetition.

---

# 66. ApplicationSet With Cluster Generator

Conceptual production pattern:

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

This can generate an Application for every matching production cluster.

---

# 67. Cluster Generator Advantages

```text
Automatic cluster discovery
Label-based targeting
Easy cluster onboarding
Less hard-coded destination configuration
```

---

# 68. Cluster Generator Risk

If a new cluster receives:

```text
environment=prod
```

it may automatically receive production Applications.

Therefore:

```text
Cluster labels are security-sensitive configuration.
```

Protect the ability to register or modify cluster metadata.

---

# 69. Cluster Onboarding Workflow

```text
Provision EKS
 |
 v
Configure authentication
 |
 v
Register cluster with Argo CD
 |
 v
Apply approved labels
 |
 v
ApplicationSet discovers cluster
 |
 v
Applications generated
 |
 v
Argo CD deploys workloads
```

This creates a highly automated onboarding process.

---

# 70. Cluster Registration Concept

Using CLI, administrators commonly inspect registered clusters with:

```bash
argocd cluster list
```

A cluster can be added using the Argo CD CLI or equivalent supported configuration.

The actual authentication mechanism should match the organization's EKS access model.

---

# 71. Cluster List Verification

Run:

```bash
argocd cluster list
```

Check:

```text
SERVER
NAME
VERSION
STATUS
```

If the target cluster is missing:

```text
ApplicationSet Cluster generator cannot target it.
```

---

# 72. ApplicationSet Status

Inspect:

```bash
kubectl get applicationsets -n argocd
```

Then:

```bash
kubectl describe applicationset <name> -n argocd
```

Look for:

```text
Conditions
Events
Generated Applications
Generator errors
Template errors
```

---

# 73. Generated Application Inspection

```bash
kubectl get applications -n argocd
```

Then:

```bash
kubectl describe application <name> -n argocd
```

This separates:

```text
ApplicationSet generation problem
```

from:

```text
Application synchronization problem
```

---

# 74. ApplicationSet Troubleshooting Decision Tree

```text
Application missing?
        |
        v
Check ApplicationSet
        |
        +--> Generator problem?
        |
        +--> Template problem?
        |
        +--> Selector problem?
        |
        +--> Repository problem?
        |
        v
Application generated?
        |
        v
Check Argo CD Application
        |
        v
Sync/reconciliation troubleshooting
```

---

# 75. ApplicationSet Not Generating Application

Check:

```bash
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd
kubectl get applications -n argocd
```

Possible causes:

```text
Invalid generator
No matching clusters
Wrong Git path
Template error
Invalid parameters
RBAC restrictions
```

---

# 76. Cluster Generator Returns Nothing

Check:

```bash
argocd cluster list
```

Then inspect cluster labels.

The ApplicationSet selector might require:

```text
environment=prod
```

but the cluster has:

```text
env=prod
```

A label mismatch produces zero generated Applications.

---

# 77. Git Generator Returns Nothing

Check:

```text
Repository URL
Revision
Directory path
Include/exclude patterns
Repository credentials
```

Then reproduce repository access independently.

---

# 78. Application Generated for Wrong Cluster

Check:

```text
Cluster labels
Selector
Destination template
Generated Application
```

For example:

```yaml
selector:
  matchLabels:
    environment: prod
```

must not accidentally match:

```text
prod-dr
```

unless intended.

---

# 79. Application Name Collision

Generated names must be unique.

Bad:

```text
cart
cart
cart
```

Better:

```text
cart-dev
cart-qa
cart-prod
```

For multiple clusters:

```text
cart-prod-eks01
cart-prod-eks02
```

---

# 80. Cluster Name in Generated Application

A practical pattern:

```yaml
name: '{{service}}-{{environment}}-{{name}}'
```

where `name` is the cluster name exposed by the cluster generator.

This makes the Application identity explicit.

---

# 81. ApplicationSet and Namespace Creation

If generated Applications use:

```yaml
syncOptions:
  - CreateNamespace=true
```

Argo CD can create the destination namespace when supported and permitted.

But namespace ownership should be deliberate.

In regulated environments, namespaces may be provisioned separately.

---

# 82. Namespace Ownership Strategy

Option A:

```text
Argo CD creates namespaces
```

Option B:

```text
Terraform/platform automation creates namespaces
```

Option C:

```text
Dedicated platform Application manages namespaces
```

Choose one owner.

Avoid competing controllers.

---

# 83. ApplicationSet and Prune

Generated Applications can use:

```yaml
syncPolicy:
  automated:
    prune: true
```

This controls resource pruning inside the generated Application.

ApplicationSet itself also has lifecycle behavior for generated Applications.

Do not confuse:

```text
resource pruning
```

with:

```text
generated Application deletion
```

---

# 84. ApplicationSet Deletion Behavior

If a generator stops producing an Application, the ApplicationSet controller may remove the generated Application according to configured behavior and policy.

This can be powerful and dangerous.

Before removing a generator input:

```text
Understand what will disappear.
```

---

# 85. Production Safety

Before enabling automatic generation/deletion:

```text
Review generated Applications
Review project restrictions
Review prune behavior
Test in dev
```

Use controlled Git changes.

---

# 86. ApplicationSet and GitOps Auditability

The desired Application inventory is represented in:

```text
ApplicationSet
+
Git generator source
+
Git history
```

This provides a traceable answer to:

```text
Why does this Application exist?
```

---

# 87. ApplicationSet Repository Design

A production repository may use:

```text
gitops/
├── applicationsets/
│   ├── roboshop-services.yaml
│   ├── roboshop-clusters.yaml
│   └── platform.yaml
│
├── applications/
├── projects/
├── charts/
├── environments/
└── clusters/
```

---

# 88. ApplicationSet Separation

Avoid one giant ApplicationSet for everything.

Prefer logical boundaries:

```text
platform-applicationset
roboshop-applicationset
monitoring-applicationset
security-applicationset
```

This improves ownership and troubleshooting.

---

# 89. ApplicationSet Ownership

Example:

```text
Platform team
    |
    +--> platform ApplicationSets

Application team
    |
    +--> application ApplicationSets
```

Use Argo CD RBAC and Projects to enforce boundaries.

---

# 90. ApplicationSet Security

Protect:

```text
ApplicationSet creation
ApplicationSet modification
Cluster labels
Cluster registration
Repository credentials
Project configuration
```

An attacker who can manipulate an ApplicationSet may be able to generate privileged Applications.

---

# 91. ApplicationSet and RBAC

A user who can edit:

```text
ApplicationSet
```

should be treated as highly privileged.

Use least privilege:

```text
Read-only
Developer
Operator
Platform admin
```

with appropriate Argo CD RBAC.

---

# 92. ApplicationSet and Project Boundaries

Projects should limit:

```text
sourceRepos
destinations
cluster-scoped resources
namespace-scoped resources
```

Then even a generated Application cannot deploy outside its allowed boundaries.

---

# 93. Production Project Boundary

Example conceptual:

```text
roboshop-prod Project

Allowed repo:
github.com/example/roboshop-gitops

Allowed destination:
eks-prod

Allowed namespaces:
roboshop

Denied:
cluster-admin
unapproved cluster resources
```

ApplicationSet should generate Applications inside this boundary.

---

# 94. ApplicationSet and Production Approval

ApplicationSet does not replace production approval.

A common pattern:

```text
CI
 |
 v
GitOps PR
 |
 v
Code review
 |
 v
Merge
 |
 v
ApplicationSet/Argo CD
 |
 v
Production
```

For highly regulated deployments:

```text
Manual sync
```

may be used for production.

---

# 95. Automatic vs Manual Production Sync

ApplicationSet can generate Applications with:

```yaml
syncPolicy:
  automated:
```

or without automated sync.

A common enterprise strategy:

```text
DEV  -> automated
QA   -> automated or controlled
PROD -> approval/manual
```

The exact model depends on risk tolerance.

---

# 96. Multi-Cluster Production Promotion

Example:

```text
DEV cluster
 |
 v
QA cluster
 |
 v
PROD primary
 |
 v
PROD DR
```

The same immutable image can be promoted through Git changes.

---

# 97. Cluster Labels for Promotion

Example:

```text
environment=dev
environment=qa
environment=prod
```

Additional labels:

```text
promotion=enabled
region=ap-south-1
```

ApplicationSet can use these labels to determine eligible targets.

---

# 98. Avoid Label-Based Accidental Promotion

Do not create a generic selector such as:

```text
environment=prod
```

if untrusted teams can set this label.

Cluster metadata should be centrally controlled.

---

# 99. ApplicationSet With Service and Environment

Conceptual matrix:

```text
service generator
      |
      +--> cart
      +--> user
      +--> payment

environment generator
      |
      +--> dev
      +--> qa
      +--> prod

             |
             v
           Matrix
             |
             v
     ApplicationSet template
```

Result:

```text
9 Applications
```

---

# 100. Production Matrix Example Concept

```yaml
generators:
  - matrix:
      generators:

        - list:
            elements:
              - service: cart
              - service: user
              - service: payment

        - list:
            elements:
              - environment: dev
              - environment: qa
              - environment: prod
```

The template combines:

```text
service
+
environment
```

---

# 101. Matrix With Git Discovery

A more dynamic model can combine:

```text
Git directories
```

with:

```text
Cluster list
```

Conceptually:

```text
Services discovered from Git
        X
Clusters discovered by Argo CD
        |
        v
ApplicationSet
```

This can be powerful for large platforms.

---

# 102. Matrix Guardrails

Before using Matrix broadly:

```text
Estimate Application count
Estimate reconciliation load
Define naming conventions
Define project boundaries
Define cluster selectors
```

Never enable an unbounded Matrix generator in production without understanding the output.

---

# 103. Merge Generator Use Case

Suppose all prod clusters use:

```text
replicas=3
```

but a DR cluster needs:

```text
replicas=5
```

Merge can combine:

```text
common config
+
cluster-specific override
```

without duplicating every field.

---

# 104. ApplicationSet and Helm Values

Generated Applications can select:

```text
values/{{environment}}.yaml
```

For service-specific values:

```text
values/{{service}}/{{environment}}.yaml
```

This produces:

```text
values/cart/dev.yaml
values/cart/qa.yaml
values/cart/prod.yaml
```

---

# 105. Recommended RoboShop Values Structure

```text
values/
├── cart/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
├── user/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
├── payment/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
```

This is easy to understand and review.

---

# 106. Generated Application Example

For:

```text
service=cart
environment=prod
```

ApplicationSet can produce:

```yaml
metadata:
  name: cart-prod

spec:
  source:
    path: helm/cart

    helm:
      valueFiles:
        - values/cart/prod.yaml

  destination:
    name: eks-prod
    namespace: roboshop
```

---

# 107. Multi-Cluster Generated Application Example

For:

```text
service=cart
environment=prod
cluster=eks-prod-ap-south-1
```

Application:

```yaml
metadata:
  name: cart-prod-eks-prod-ap-south-1

spec:
  destination:
    name: eks-prod-ap-south-1
    namespace: roboshop
```

This makes the target obvious.

---

# 108. Central Argo CD and EKS Failure Modes

If central Argo CD fails:

```text
Existing workloads usually continue running.
```

However:

```text
New deployments
Drift correction
Application sync
```

may stop until Argo CD recovers.

This distinction is important.

---

# 109. ApplicationSet Controller Failure

If ApplicationSet controller is unavailable:

```text
Existing generated Applications remain.
```

But:

```text
New Applications may not be generated
Removed Applications may not be reconciled
Generator changes may not propagate
```

The Application controller can still reconcile existing Applications.

---

# 110. Target Cluster Failure

If:

```text
EKS PROD
```

becomes unreachable:

```text
Argo CD cannot reconcile that cluster.
```

Other target clusters can continue operating if Argo CD itself is healthy.

This is one advantage of centralized management.

---

# 111. Git Repository Failure

If Git becomes unavailable:

```text
Existing desired state remains known by Argo CD
```

but:

```text
New changes cannot be fetched
```

Operational recovery should restore repository access.

---

# 112. ApplicationSet and Disaster Recovery

Back up:

```text
ApplicationSets
Applications
Projects
Repository configuration
Cluster registration configuration
RBAC
Argo CD configuration
Git repository
```

The Git repository should remain the primary source for declarative configuration.

---

# 113. ApplicationSet DR Principle

A new Argo CD installation should be able to reconstruct:

```text
Projects
ApplicationSets
Applications
```

from Git as much as possible.

Avoid undocumented manual configuration.

---

# 114. Bootstrap Architecture

A common pattern:

```text
Terraform
 |
 v
EKS management cluster
 |
 v
Install Argo CD
 |
 v
Bootstrap root Application
 |
 v
ApplicationSets
 |
 v
Platform Applications
 |
 v
Business Applications
```

This creates a reproducible platform.

---

# 115. ApplicationSet Bootstrap

The initial bootstrap may install:

```text
platform ApplicationSet
```

which generates:

```text
ALB Controller
External Secrets
Monitoring
Security components
```

Then another ApplicationSet can manage:

```text
RoboShop services
```

---

# 116. ApplicationSet and App of Apps

They solve related but different problems.

### App of Apps

```text
Parent Application
       |
       +--> child Application
       +--> child Application
```

### ApplicationSet

```text
Generator
       |
       +--> generated Application
       +--> generated Application
```

ApplicationSet is usually more dynamic.

App of Apps is often simpler for explicit bootstrapping.

---

# 117. When to Use ApplicationSet

Use it when:

```text
Many similar Applications
Many environments
Many clusters
Dynamic discovery
Repeated deployment patterns
```

---

# 118. When App of Apps May Be Better

Use App of Apps when:

```text
Small number of explicitly defined applications
Bootstrap hierarchy
Platform root application
Simple dependency structure
```

Both can coexist.

---

# 119. ApplicationSet and App of Apps Together

Example:

```text
platform-root
 |
 +--> ApplicationSet: platform
 |
 +--> ApplicationSet: roboshop
 |
 +--> ApplicationSet: observability
```

This can create a scalable hierarchy.

---

# 120. Avoid Recursive Complexity

Do not create:

```text
App
 |
 v
ApplicationSet
 |
 v
Application
 |
 v
ApplicationSet
 |
 v
Application
```

unless there is a strong reason.

The more layers you add:

```text
more controllers
+
more generated resources
+
more troubleshooting complexity
```

---

# 121. ApplicationSet Notifications

ApplicationSet itself is not the same as Argo CD Notifications.

For deployment notifications, use Argo CD notification capabilities.

Possible alerts:

```text
Sync failed
Application degraded
Application healthy
```

ApplicationSet health should also be monitored.

---

# 122. Monitoring ApplicationSet

Monitor:

```text
ApplicationSet count
Generated Application count
Controller errors
Reconciliation latency
API errors
Git generator errors
Cluster generator errors
```

Use Prometheus/Grafana where Argo CD metrics are available.

---

# 123. ELK Integration

The user's observability stack includes:

```text
Prometheus
Grafana
ELK
```

ApplicationSet controller and Argo CD logs can be shipped to ELK.

Useful searches:

```text
applicationset
generator
reconcile
error
permission
cluster
repository
```

---

# 124. Prometheus Metrics

Argo CD exposes metrics that can be scraped by Prometheus depending on the component and configuration.

Monitor:

```text
Controller health
Application health
Sync failures
Reconciliation activity
API latency
```

ApplicationSet-specific metrics should be checked against the installed version.

---

# 125. Grafana Dashboard

A production dashboard can show:

```text
Applications total
Applications Healthy
Applications Degraded
Applications OutOfSync
ApplicationSet count
Generated Application count
Sync failures
Cluster connectivity
Controller errors
```

---

# 126. Alerting

Examples:

```text
Production ApplicationSet generation failures
Production Applications unexpectedly deleted
Multiple Applications OutOfSync
Cluster registration unhealthy
Argo CD controller errors
```

Avoid alerting on every transient condition.

Use meaningful thresholds.

---

# 127. ApplicationSet Repository Webhook

Git-based generators may otherwise discover changes through polling/reconciliation.

Webhooks can reduce detection latency where configured.

Flow:

```text
Git push
 |
 v
Webhook
 |
 v
Argo CD/ApplicationSet refresh
 |
 v
Generated Application update
```

The exact webhook setup depends on the Git provider and Argo CD configuration.

---

# 128. Webhook Security

Protect webhook endpoints with:

```text
Authentication
Secret validation
TLS
Network controls
Rate limiting
```

Do not expose an unauthenticated administrative endpoint unnecessarily.

---

# 129. ApplicationSet Git Generator Authentication

For private Git repositories:

```text
Argo CD repository credentials
```

must allow repository access.

If generator cannot read the repository:

```text
No generator data
```

may result.

---

# 130. Repository Credential Principle

Do not put:

```text
Git token
SSH private key
password
```

inside ApplicationSet YAML.

Use Argo CD repository configuration/credential mechanisms.

---

# 131. ApplicationSet and Private Git

Architecture:

```text
Private Git
    |
    | credentials
    v
Argo CD Repo Server
    |
    v
ApplicationSet generator
    |
    v
Applications
```

Keep credentials least-privileged.

---

# 132. ApplicationSet Git Branch Strategy

For production GitOps:

```text
main
```

can represent approved desired state.

Alternative:

```text
environment branches
```

may be used, but branch-heavy designs can become complex.

A repository-per-environment or directory-per-environment strategy may be easier to audit.

---

# 133. ApplicationSet and Branch Promotion

A controlled model:

```text
dev configuration
 |
 v
test
 |
 v
promotion PR
 |
 v
prod configuration
```

Avoid automatically promoting untested changes to production.

---

# 134. Image Promotion

ApplicationSet does not build images.

CI handles:

```text
Build
Test
Security
Push ECR
```

GitOps/ApplicationSet handles:

```text
Desired deployment configuration
```

This maintains clear CI/CD responsibility.

---

# 135. CI vs ApplicationSet

CI:

```text
Build artifact
Test artifact
Scan artifact
Publish artifact
Update Git
```

ApplicationSet:

```text
Generate Applications
```

Argo CD:

```text
Reconcile Applications
```

Kubernetes:

```text
Run workloads
```

---

# 136. Complete RoboShop Flow

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven/npm/python build
   +--> Tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker Image
   |
   v
AWS ECR
   |
   v
Update GitOps values
   |
   v
GitOps PR
   |
   v
Merge
   |
   v
ApplicationSet
   |
   v
Generated Application
   |
   v
Argo CD
   |
   v
EKS
```

ALB traffic enters through:

```text
AWS ALB
 |
 v
Kubernetes Ingress
 |
 v
Service
 |
 v
Pod
```

---

# 137. ApplicationSet and RoboShop Service Inventory

Example:

```text
cart
catalogue
user
payment
inventory
orders
shipping
notification
frontend
```

ApplicationSet can standardize:

```text
Project
Source
Namespace
Sync policy
Helm chart
Environment
Cluster
```

---

# 138. Production ApplicationSet Repository

Recommended practical structure:

```text
roboshop-gitops/
│
├── applicationsets/
│   ├── roboshop-services.yaml
│   ├── roboshop-prod-clusters.yaml
│   └── platform-services.yaml
│
├── projects/
│   ├── roboshop-dev.yaml
│   ├── roboshop-qa.yaml
│   └── roboshop-prod.yaml
│
├── charts/
│   ├── cart/
│   ├── catalogue/
│   ├── user/
│   ├── payment/
│   └── frontend/
│
├── values/
│   ├── cart/
│   ├── catalogue/
│   ├── user/
│   ├── payment/
│   └── frontend/
│
├── clusters/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── platform/
```

---

# 139. Production Naming Convention

Use predictable names.

Examples:

```text
applicationset:
roboshop-services

application:
cart-dev
cart-qa
cart-prod
```

For multi-cluster:

```text
cart-prod-eks-prod-01
cart-prod-eks-prod-02
```

Avoid random names.

---

# 140. ApplicationSet Labels

Generated Applications can have labels:

```yaml
labels:
  app.kubernetes.io/part-of: roboshop
  environment: '{{environment}}'
  service: '{{service}}'
  managed-by: applicationset
```

This improves filtering and operational visibility.

---

# 141. ApplicationSet Annotations

Annotations can be used for:

```text
Notifications
Operational metadata
Ownership
Links
```

Do not overload annotations with secrets or large configuration blobs.

---

# 142. ApplicationSet Template Defaults

Keep common policy in the template.

Example:

```text
Common:
project
repoURL
revision
sync policy
namespace pattern
labels
```

Generator should provide only values that actually differ.

This keeps ApplicationSets readable.

---

# 143. Avoid Giant List Generator

Bad:

```text
500 entries
```

inside one ApplicationSet.

At scale use:

```text
Git generator
Cluster generator
Matrix
Merge
SCM discovery
```

where appropriate.

---

# 144. Avoid Excessive Generator Logic

ApplicationSet templates can become difficult to understand if they contain many conditional transformations.

Prefer:

```text
Simple generator inputs
+
Simple template
+
Clear repository structure
```

---

# 145. ApplicationSet Reconciliation Frequency

ApplicationSet controllers periodically reconcile resources.

The exact timing is configurable/version-dependent.

For near-real-time Git updates:

```text
Webhooks
```

can reduce delay.

Do not assume instant reconciliation without verifying configuration.

---

# 146. ApplicationSet Scale

Scaling depends on:

```text
Number of ApplicationSets
Number of generated Applications
Number of clusters
Repository size
Generator complexity
Argo CD controller capacity
Git/API latency
```

Benchmark your own environment.

---

# 147. ApplicationSet and Argo CD HA

In production:

```text
Argo CD API Server
Application Controller
Repo Server
Redis
ApplicationSet Controller
```

can be deployed according to HA requirements and supported deployment modes.

ApplicationSet is not a replacement for HA architecture.

---

# 148. Controller Failure Isolation

If ApplicationSet controller fails:

```text
Generated Applications already exist.
```

If Application Controller fails:

```text
Application resources remain.
Kubernetes workloads remain.
Reconciliation pauses.
```

If Repo Server fails:

```text
Manifest generation may fail.
```

Understanding these failure boundaries is essential for production operations.

---

# 149. Disaster Scenario: Management Cluster Lost

Recovery:

```text
1. Recreate EKS management cluster.
2. Install Argo CD.
3. Restore repository access.
4. Restore Projects/RBAC/repository config.
5. Register target clusters.
6. Apply ApplicationSets.
7. Reconcile.
8. Validate generated Applications.
9. Validate workloads.
```

The GitOps repository should contain as much of this declarative configuration as practical.

---

# 150. ApplicationSet Interview Question

## What is ApplicationSet?

### Answer

> ApplicationSet is an Argo CD controller/resource that generates and manages multiple Argo CD Applications from generators and an Application template. It is especially useful for multi-environment and multi-cluster deployments because it removes repetitive Application manifests.

---

# 151. Application vs ApplicationSet Interview Question

### Answer

> An Application represents one desired deployment to a target cluster and namespace. An ApplicationSet generates multiple Applications from a template and generator inputs. The ApplicationSet controller manages Application generation, while the Argo CD Application Controller reconciles the generated Applications against Kubernetes.

---

# 152. List vs Cluster Generator

### Answer

> The List generator uses explicitly defined elements, making it simple and predictable for a small fixed environment inventory. The Cluster generator discovers clusters registered with Argo CD and can select them using labels, making it more suitable for dynamic multi-cluster environments.

---

# 153. Why Use Cluster Generator?

### Answer

> It allows centralized Argo CD to automatically deploy applications to clusters matching labels such as `environment=prod` or `region=ap-south-1`. This reduces hard-coded cluster configuration and simplifies cluster onboarding.

---

# 154. What Is Matrix Generator?

### Answer

> Matrix combines the outputs of two generators to create combinations. For example, a service generator containing cart, user and payment combined with an environment generator containing dev, QA and prod can generate nine Applications.

---

# 155. What Is Merge Generator?

### Answer

> Merge combines generator parameter sets based on matching keys. It is useful when there is a common configuration set plus targeted overrides for particular environments or clusters.

---

# 156. What Is Pull Request Generator?

### Answer

> It can create temporary Applications based on pull requests, enabling preview environments or PR-specific deployments. Security controls are critical because preview environments should not receive production credentials or unrestricted network access.

---

# 157. Scenario: ApplicationSet Generated Nothing

### Answer

I would troubleshoot in this order:

```text
1. kubectl get applicationset -n argocd
2. kubectl describe applicationset <name> -n argocd
3. Check ApplicationSet conditions/events
4. Validate generator
5. Validate Git path/revision
6. Validate cluster selector
7. Validate cluster registration
8. Validate template parameters
9. Check generated Applications
```

---

# 158. Scenario: Prod Cluster Was Accidentally Included

Check:

```text
Cluster labels
ApplicationSet selector
Generated Application
Project restrictions
```

Then:

```text
Fix selector/labels
Review generated Applications
Disable dangerous automation if necessary
Audit the deployment
```

---

# 159. Scenario: New EKS Cluster Should Automatically Receive Applications

Use:

```text
1. Register cluster with Argo CD.
2. Apply approved labels.
3. Ensure Project allows the destination.
4. ApplicationSet Cluster generator selects the label.
5. Generated Applications appear.
6. Argo CD reconciles workloads.
```

This is a production-friendly onboarding model.

---

# 160. Scenario: One Cluster Must Be Excluded

Options include:

```text
Remove matching label
Add exclusion logic
Use a more specific selector
Use a dedicated cluster label
```

Do not simply delete generated Applications repeatedly; fix the source condition.

---

# 161. Scenario: 100 New Clusters Added

Do not create:

```text
100 manual Application YAMLs
```

Instead:

```text
Register clusters
Apply controlled labels
Use Cluster generator
```

This is exactly the kind of scale ApplicationSet is designed to support.

---

# 162. Scenario: One Cluster Needs Different Helm Values

Use:

```text
cluster labels
+
generator parameter
+
cluster-specific values
```

For example:

```text
values/prod/default.yaml
values/prod/dr.yaml
```

Do not duplicate the entire Helm chart.

---

# 163. Scenario: Production Requires Manual Approval

Generate the Application without automated sync:

```yaml
syncPolicy:
```

or use a controlled production policy.

Then:

```text
Git merge
 |
 v
Application OutOfSync
 |
 v
Approved operator sync
 |
 v
Production
```

The exact approval mechanism should match organizational policy.

---

# 164. Scenario: ApplicationSet Deleted a Generated Application

Investigate:

```text
Generator input removed?
Selector changed?
Git directory deleted?
ApplicationSet changed?
```

Then inspect:

```bash
kubectl describe applicationset <name> -n argocd
```

Review Git history before restoring.

---

# 165. ApplicationSet Operational Runbook

### Application missing

```bash
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd
kubectl get applications -n argocd
```

### Cluster generator

```bash
argocd cluster list
```

### Generated Application

```bash
argocd app get <application>
argocd app diff <application>
```

### Runtime

```bash
kubectl get pods -n <namespace>
kubectl get events -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
```

---

# 166. Production Checklist

## ApplicationSet

```text
[ ] Naming convention defined
[ ] Generator bounded
[ ] Template simple
[ ] Projects configured
[ ] Destination restricted
[ ] Repository restricted
[ ] Cluster labels controlled
[ ] Production sync policy reviewed
[ ] Prune behavior understood
[ ] Generated Application count estimated
```

---

# 167. Multi-Cluster Checklist

```text
[ ] Target clusters registered
[ ] Cluster credentials secured
[ ] Least privilege configured
[ ] Cluster labels controlled
[ ] Projects restrict destinations
[ ] Production clusters separated
[ ] Cross-account access reviewed
[ ] DR cluster strategy defined
[ ] Cluster onboarding documented
```

---

# 168. ApplicationSet Design Principles

1. Keep generators understandable.
2. Keep templates small.
3. Use predictable names.
4. Use labels for controlled discovery.
5. Restrict Projects.
6. Do not expose production credentials.
7. Bound generator scope.
8. Test generated Applications in non-production.
9. Monitor ApplicationSet controller health.
10. Keep the desired configuration in Git.
11. Prefer immutable images.
12. Avoid giant ApplicationSets when ownership differs.
13. Separate platform and business applications.
14. Treat cluster labels as privileged configuration.
15. Document the generated resource model.

---

# 169. Important Production Distinction

Remember:

```text
ApplicationSet
=
Generates Applications

Application Controller
=
Deploys/Reconciles Applications

Kubernetes controllers
=
Operate workloads/resources
```

Example:

```text
ApplicationSet
      |
      v
cart-prod Application
      |
      v
Argo CD Application Controller
      |
      v
Deployment
      |
      v
ReplicaSet
      |
      v
Pods
```

---

# 170. Final Architecture: Multi-Environment

```text
                         GitOps Repository
                                |
                                v
                        ApplicationSet
                                |
               +----------------+----------------+
               |                |                |
               v                v                v
          cart-dev         cart-qa          cart-prod
               |                |                |
               v                v                v
            EKS-DEV          EKS-QA           EKS-PROD
```

---

# 171. Final Architecture: Multi-Cluster

```text
                         Git Repository
                              |
                              v
                    Central Argo CD
                              |
                       ApplicationSet
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
       EKS-DEV             EKS-QA             EKS-PROD
          |                   |                   |
       RoboShop            RoboShop            RoboShop
```

---

# 172. Final Architecture: Service × Environment

```text
             Services
                |
       +--------+--------+
       |        |        |
      cart     user    payment
       |
       X
       |
 Environments
       |
  +----+----+
  |    |    |
 dev  qa   prod
       |
       v
 ApplicationSet
       |
       v
 Generated Applications
```

---

# 173. Final Architecture: CI + ApplicationSet + Argo CD

```text
Developer
    |
    v
Source Git
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
Generated Applications
    |
    v
Argo CD
    |
    v
EKS
```

---

# 174. Final Takeaways

ApplicationSet is the scaling layer for Argo CD Applications.

The most important concepts are:

```text
Application
ApplicationSet
Generator
Template
List
Git
Directory
Cluster
Matrix
Merge
Pull Request
SCM
```

For production EKS environments, the most important pattern is:

```text
Central Argo CD
       |
       v
ApplicationSet
       |
       +--> Cluster generator
       |
       +--> Cluster labels
       |
       v
Multiple EKS clusters
```

For RoboShop:

```text
Service
+
Environment
+
Cluster
+
Helm values
```

can be converted into generated Applications.

The complete control plane becomes:

```text
Git
 |
 v
ApplicationSet
 |
 v
Applications
 |
 v
Argo CD Application Controller
 |
 v
EKS
```

---

# 175. Next File

```text
10-ArgoCD-Multi-Cluster-Management.md
```

The next file will go deeper into centralized Argo CD as a multi-cluster GitOps control plane, including:

- Argo CD management cluster
- Target EKS clusters
- Cluster registration
- Cluster credentials
- EKS authentication
- AWS IAM and Kubernetes RBAC boundaries
- Dev/QA/Prod clusters
- Multiple production clusters
- Multi-region EKS
- Multi-account AWS
- Cluster generator
- Cluster labels
- ApplicationSet multi-cluster patterns
- Centralized vs decentralized Argo CD
- Hub-and-spoke architecture
- Failure scenarios
- Cluster onboarding
- Cluster removal
- Security boundaries
- HA considerations
- Disaster recovery
- Production YAMLs
- RoboShop multi-cluster deployment
- Operational commands
- Troubleshooting
- Interview scenarios
