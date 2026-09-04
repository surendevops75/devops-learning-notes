# GitOps Repository

> Deep production-oriented GitOps repository design for GitLab CI, ECR,
> Helm, Argo CD, EKS, multi-environment and multi-cluster deployments.

## Chapter Objective

This chapter designs the GitOps repository as a production control
plane. It covers repository structure, environment separation, immutable
image promotion, Helm integration, secrets, validation, Argo CD
Applications and Projects, multi-cluster patterns, security, drift,
rollback, disaster recovery, troubleshooting, and senior-level interview
reasoning.

## 1. GitOps Definition and Goal

GitOps treats Git as the authoritative, reviewable declaration of the
desired application state. Instead of CI holding cluster credentials and
executing kubectl directly, CI produces and publishes an immutable
artifact and updates the desired state in Git. Argo CD continuously
compares that desired state with the Kubernetes cluster and reconciles
drift. This creates a clean separation between artifact production,
deployment intent, and cluster reconciliation.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 2. Why GitOps for the Capstone

The capstone uses AWS, EKS, ECR, Helm, GitLab CI, Argo CD, multiple
environments, and potentially multiple clusters. A GitOps repository
provides the missing control plane for deployment configuration. It lets
engineers review production changes as Git commits, gives Argo CD a
stable source of truth, makes rollback a Git operation, and provides a
historical relationship between an application version and the
environment in which it was deployed.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 3. CI and GitOps Responsibilities

CI should validate code, run tests and security checks, build the
container, publish it to ECR, capture the immutable digest, and update
the appropriate GitOps location. CI should not normally maintain
permanent Kubernetes credentials. GitOps stores desired state. Argo CD
performs reconciliation. Kubernetes executes the workload. This
separation reduces privilege and makes responsibilities easier to
troubleshoot.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 4. Repository Separation

A production design can use separate repositories for application
source, infrastructure, and GitOps configuration. Application
repositories contain source, tests, Dockerfiles, Helm charts, and CI
definitions. Terraform has its own infrastructure repository or
controlled directory. The GitOps repository contains environment and
cluster deployment configuration. Separation reduces blast radius and
allows different teams and policies to own different layers.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 5. Monorepo vs Polyrepo

A GitOps monorepo is convenient when one platform team owns many
services and wants a single place for environment state. Polyrepo
layouts can improve service ownership and access isolation. The
important requirement is not the number of repositories; it is that the
desired state is protected, reviewable, validated, and unambiguous.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 6. Recommended Capstone Layout

A practical layout is services/`<service>`{=html}/base for common Helm
or Kustomize configuration, environments/dev, environments/qa,
environments/staging, and environments/prod for environment overlays,
plus clusters/`<cluster-name>`{=html} for cluster-specific application
registration. Shared platform configuration should be separate from
application release state so a service update does not accidentally
change cluster-wide infrastructure.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 7. Repository Structure Example

A representative structure is:

``` text
gitops/
├── apps/
│   ├── roboshop/
│   │   ├── frontend/
│   │   ├── catalogue/
│   │   ├── cart/
│   │   ├── user/
│   │   ├── shipping/
│   │   └── payment/
│   └── shared/
├── environments/
│   ├── dev/
│   ├── qa/
│   ├── staging/
│   └── prod/
├── clusters/
│   ├── dev-eks/
│   ├── staging-eks/
│   └── prod-eks/
├── argocd/
│   ├── projects/
│   ├── applications/
│   └── appsets/
├── policies/
└── README.md
```

The exact layout can vary, but ownership boundaries should remain
obvious.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 8. Environment Separation

Development, QA, staging, and production should have explicit
desired-state boundaries. A production image digest should never be
changed because someone edited a development values file.
Environment-specific changes should be visible in Git history and
protected according to the risk of the target environment.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 9. Environment Promotion

Promotion should move an existing image digest from one environment to
the next. For example, dev might reference sha256:A, QA is then changed
to sha256:A, staging to sha256:A, and production to sha256:A after
approval. This is safer than rebuilding source separately for each
environment.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 10. Immutable Digests

The strongest deployment reference is an image digest, for example:

``` yaml
image:
  repository: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/catalogue
  digest: sha256:EXACT_IMAGE_DIGEST
```

A mutable tag such as latest can point to different content over time. A
digest identifies exact content and makes rollback and audit evidence
much stronger.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 11. Tags and Digests Together

Tags can still be retained for human readability, such as
catalogue:commit-abc1234, while Kubernetes consumes the digest. The
release metadata can record both the source commit and digest. Never
assume that a tag remains immutable merely because the tag name looks
unique.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 12. GitOps Commit Metadata

An automated deployment commit should identify the service, environment,
source commit, image digest, and pipeline or release identifier. This
makes an operator's investigation faster: the GitOps commit tells them
what changed, and the artifact registry tells them what the digest
contains.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 13. Helm in GitOps

Helm provides packaging and templating. In GitOps, Argo CD can render a
Helm chart using environment-specific values and apply the resulting
manifests. Keep generic chart logic separate from environment values.
Avoid embedding production secrets in values files.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 14. Helm Values Strategy

A useful hierarchy is values.yaml for safe defaults, values-dev.yaml for
development, values-qa.yaml for QA, values-staging.yaml for staging, and
values-prod.yaml for production. Common security defaults should remain
in the chart so a careless environment file does not have to repeat
every security control.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 15. Values Precedence

When multiple Helm values sources are used, understand the precedence
rules and make them explicit. Unexpected overrides can cause production
differences such as changed replica counts, image references, resource
limits, ingress hosts, or security contexts. Render the final manifest
during validation.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 16. Production Values Review

Production values deserve code review. Review resource requests and
limits, replica counts, autoscaling, topology rules, ingress hosts,
service-account annotations, network policy settings, pod security
configuration, and image references. A one-line values change can have a
production-wide effect.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 17. Secrets in GitOps

Do not place plaintext production credentials in ordinary GitOps values
files. Use an approved secrets architecture such as AWS Secrets Manager
integrated through a Kubernetes secret synchronization mechanism,
External Secrets, or another controlled approach. The GitOps repository
should describe how the secret is consumed without becoming a secret
database.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 18. Sealed Secrets Consideration

Encrypted secret manifests can be committed to Git when an organization
has deliberately chosen that architecture. The encryption model, key
management, controller permissions, rotation, and recovery process must
be understood. Encryption at rest in Git does not remove the need for
runtime access controls.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 19. External Secrets Pattern

A common AWS pattern is EKS workload identity -\> AWS Secrets Manager
-\> External Secrets controller -\> Kubernetes Secret -\> application.
The controller needs narrowly scoped access to only the secret paths it
manages. Applications should not receive credentials for the entire
Secrets Manager account.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 20. Repository Access Control

GitOps write access should be tightly controlled. Developers may need
application-level access, platform engineers may manage shared
configuration, and production approval may require protected reviewers.
CI should receive only the repository permissions needed to update the
intended deployment path.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 21. Branch Protection

Protect the main production branch. Require merge requests, successful
validation, security checks, appropriate reviewers, and no direct pushes
where organizational policy requires it. Disable force pushes for
critical branches. A GitOps repository is production control-plane data
and deserves stronger protection than an ordinary documentation
repository.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 22. CODEOWNERS

Use CODEOWNERS or equivalent ownership rules to require the right
reviewers for sensitive paths. Production manifests, Argo CD projects,
cluster definitions, and security policies should not be modifiable by
arbitrary contributors without appropriate review.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 23. Pull Requests

Production GitOps changes should be reviewable before merge. The review
should show exactly which image digest, Helm value, cluster, namespace,
or policy changes. Automated validation should render the final
configuration so reviewers do not need to mentally evaluate every
template expression.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 24. GitOps Validation Pipeline

The GitOps repository should have its own CI pipeline. Typical checks
include YAML validation, Helm linting, Helm template rendering,
Kubernetes schema validation, policy checks, secret detection,
image-reference validation, and repository structure checks. A GitOps
commit should not be merged merely because it is syntactically valid.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 25. Manifest Rendering

Always validate the effective manifests, not just the Helm source.
Rendering catches invalid values, missing templates, wrong indentation,
and unintended overrides. Render each affected environment using exactly
the values and chart versions that Argo CD will use.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 26. Kubernetes Schema Validation

Rendered manifests should be validated against Kubernetes API schemas.
This catches invalid fields and API-version mismatches before Argo CD
attempts reconciliation. Validation should account for the Kubernetes
version of the target cluster where practical.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 27. Policy Validation

Use policy-as-code checks to enforce organization rules such as non-root
containers, approved registries, required resource limits, prohibited
hostPath mounts, restricted capabilities, required labels, and allowed
namespaces. These checks provide an earlier failure point than cluster
admission.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 28. Image Policy

The GitOps pipeline should reject unexpected image repositories, mutable
production tags, malformed digests, or registries outside the approved
trust boundary. If signed-image verification is used, deployment policy
should require trusted signatures at admission as well.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 29. Argo CD Repository Connection

Argo CD needs read access to the GitOps repository. Use a dedicated
identity with the minimum repository scope. Prefer deploy keys or
repository integration mechanisms that provide read-only access when the
controller only needs to pull manifests.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 30. Argo CD Application

An Argo CD Application declares the source repository, revision, path,
destination cluster, namespace, and synchronization behavior. The
Application itself should be protected because changing its destination
or source can redirect production deployments.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 31. Application Example

A conceptual Application definition is:

``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: catalogue-prod
  namespace: argocd
spec:
  project: production
  source:
    repoURL: <gitops-repository>
    targetRevision: main
    path: environments/prod/catalogue
  destination:
    server: https://kubernetes.default.svc
    namespace: catalogue
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

In production, enable automated behaviors only after understanding their
blast radius and use sync options deliberately.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 32. Argo CD Projects

Argo CD Projects can restrict which repositories, clusters, namespaces,
and resource types an application may use. This is an important security
boundary. A production project should not allow arbitrary repositories
or unrestricted cluster-wide resource creation.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 33. ApplicationSet

ApplicationSet can generate many Argo CD Applications from structured
inputs. It is useful for multi-cluster and multi-environment
deployments. Generator inputs must be treated as production
configuration because a small change can create or modify many
applications.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 34. App of Apps

The App of Apps pattern uses an Argo CD Application to manage other
Application resources. It can simplify bootstrapping but introduces a
hierarchy of control. Protect the parent application and understand that
a change there can affect many child applications.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 35. Bootstrap Strategy

A cluster bootstrap can install Argo CD through Terraform, Helm, or
another controlled platform mechanism. After Argo CD is operational,
GitOps can manage most application and platform resources. Keep the
initial bootstrap path documented because Argo CD itself must be
recovered if the cluster is recreated.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 36. Self-Management

Argo CD can manage some of its own configuration. This provides a
powerful GitOps loop but must be implemented carefully. A mistake in the
Argo CD project, RBAC, or repository configuration can affect the
controller's ability to reconcile the rest of the cluster.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 37. Sync Policy

Argo CD supports manual and automated synchronization. Manual sync
provides stronger human control for sensitive environments. Automated
sync improves speed and reduces manual deployment work. Production
should combine the chosen sync model with protected GitOps changes and
appropriate health checks.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 38. Prune

Pruning removes Kubernetes resources that are no longer declared in Git.
This helps eliminate stale resources but can be dangerous if repository
paths or ownership are wrong. Enable pruning only when resource
ownership is well understood and protected.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 39. Self-Heal

Self-heal allows Argo CD to correct out-of-band changes that drift from
Git. This is a core GitOps property, but it means a manual kubectl edit
may be reverted quickly. Operators must understand whether a change
belongs in Git or is an approved emergency action.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 40. Sync Waves

Sync waves allow dependent resources to be applied in an intentional
sequence. For example, prerequisites can be created before workloads.
Use waves only when dependency ordering is real; excessive ordering can
slow reconciliation and hide poor application design.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 41. Resource Hooks

Argo CD hooks can run jobs during synchronization for tasks such as
controlled migrations or validation. Hooks should be idempotent,
observable, and tightly scoped. Database migrations require special
attention because retries and rollbacks can have data consequences.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 42. Health Checks

Argo CD health status is more useful when workloads expose meaningful
readiness and health behavior. Configure custom health checks only when
necessary. A green sync without healthy application behavior is not
sufficient evidence of a successful release.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 43. Sync Windows

Sync windows can restrict when Argo CD is allowed to deploy changes.
They can support maintenance periods or release freezes. Emergency
procedures should be explicit so operators know how to handle a critical
production change during a freeze.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 44. Rollback Through Git

The clean GitOps rollback is to revert the deployment configuration to a
previously known-good digest and merge that change through the
appropriate emergency or normal review path. Argo CD then reconciles the
old desired state.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 45. Rollback vs Argo CD History

Argo CD can help identify previous sync states, but the long-term source
of truth should remain Git. A rollback should restore the repository to
a valid desired state rather than leaving Git and the cluster
permanently inconsistent.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 46. Drift Detection

Drift means the live cluster differs from the desired state. Common
causes include manual kubectl edits, controllers mutating resources,
admission webhooks, operators, Helm behavior, or out-of-band automation.
Distinguish intentional controller-managed changes from unauthorized
drift.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 47. Out-of-Band Changes

If an engineer manually changes a deployment, GitOps may revert it. That
behavior is expected. The operational rule should be: change desired
state in Git for normal operations; use direct cluster changes only
under documented emergency procedures and reconcile them back into Git
afterward.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 48. Ignore Differences

Some Kubernetes resources legitimately contain dynamic fields that
should not trigger continuous drift. Argo CD supports controlled
difference customization. Do not use broad ignore rules to hide real
configuration drift; ignore only fields that are known to be externally
managed.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 49. Multi-Environment Repository

A single GitOps repository can hold all environments if access control
and review rules are strong. A promotion PR can change staging first and
production later. The history clearly shows the artifact's progression.
For high-isolation organizations, production may be separated into
another repository or account.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 50. Separate Production Repository

Separating production GitOps from lower environments can reduce blast
radius and strengthen access control. It also introduces additional
promotion mechanics. The choice should follow security and
organizational boundaries rather than fashion.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 51. Multi-Cluster Strategy

For multiple EKS clusters, keep cluster identity explicit. A GitOps path
should clearly state which cluster receives a workload. Never rely on an
ambiguous environment name when multiple production clusters exist.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 52. Cluster Directory Pattern

A useful pattern is clusters/prod-us-east-1, clusters/prod-ap-south-1,
and clusters/staging-ap-south-1. Each cluster can reference environment
configuration while maintaining cluster-specific differences such as
ingress hosts, node architecture, capacity, or regional endpoints.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 53. Cluster Labels

ApplicationSet generators can use cluster labels to decide which
applications deploy to which clusters. Labels must be protected because
changing a label can change deployment topology. Treat cluster
registration metadata as sensitive control-plane configuration.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 54. Disaster Recovery for GitOps

Back up Git repositories using the organization's source-control backup
strategy and retain critical repository history. Back up Argo CD
configuration or ensure it can be recreated entirely from Git and
infrastructure code. A new EKS cluster should be able to bootstrap the
GitOps controller and recover desired state.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 55. GitOps Disaster Scenario

If an EKS cluster is lost, recreate the infrastructure through
Terraform, install Argo CD, restore or reconnect the GitOps repository,
configure repository access and projects, and allow Argo CD to reconcile
applications. Immutable images in ECR must remain available for the
recovery window.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 56. ECR Retention and GitOps

GitOps may reference an image digest months after the original build.
Registry lifecycle policies must retain production and rollback
artifacts for at least the required recovery period. Removing an image
that Git still references creates a broken desired state.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 57. Repository Availability

GitOps depends on source control availability. Enterprise designs should
understand repository outages, cached manifests, Argo CD behavior during
temporary Git access failure, and the emergency deployment process. Do
not assume Git is always reachable during an incident.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 58. Emergency Deployment

If GitOps is temporarily unavailable during a severe incident, use the
documented emergency process and preserve the exact artifact and desired
configuration. Once normal service is restored, reconcile the emergency
state back into Git immediately so the source of truth is restored.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 59. GitOps Security Model

The core security chain is developer identity -\> repository permissions
-\> CI validation -\> protected merge -\> Argo CD repository read -\>
Argo CD project authorization -\> Kubernetes RBAC -\> admission policy
-\> workload identity. Each layer should restrict the next layer rather
than assuming trust from the previous one.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 60. Argo CD RBAC

Argo CD RBAC should distinguish read-only observers, application
operators, environment administrators, and platform administrators.
Avoid giving application teams unrestricted Argo CD admin access because
that can become an indirect route to cluster-wide permissions.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 61. Kubernetes RBAC

Argo CD service accounts should receive only the permissions needed for
managed resources. Namespace-scoped permissions can reduce blast radius
when application ownership is separated. Cluster-wide permissions should
be justified and reviewed.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 62. Repository Credential Security

Repository credentials used by Argo CD should be stored using the
supported secure mechanism and rotated according to policy. Do not place
private keys in GitOps YAML. Audit who can modify the repository
connection and repository secrets.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 63. Git Commit Signing

Signed commits can strengthen GitOps integrity by establishing trusted
authorship for sensitive changes. Verification policy should be designed
before requiring it, including key rotation, emergency recovery, and how
automated CI commits are trusted.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 64. CI Bot Identity

Use a dedicated CI identity for automated GitOps updates. The bot should
have write access only where necessary. It should not be a general
developer account or a repository administrator.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 65. Preventing CI Loops

When CI updates GitOps, the resulting GitOps commit can trigger another
pipeline. Use workflow rules, path filters, separate pipelines, or
explicit commit markers to prevent infinite loops. The application
pipeline and GitOps validation pipeline should have clear trigger
boundaries.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 66. Atomic Promotion

A promotion commit should update the intended artifact reference
atomically. Avoid scripts that partially update several environments and
leave the repository in a mixed state if one command fails. Use Git
operations that either produce the expected commit or fail visibly.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 67. Concurrent Updates

Two pipelines can attempt to update the same GitOps file simultaneously.
Handle concurrency with pull/rebase/retry logic, serialized promotion
jobs, or a merge-request based workflow. Never blindly overwrite another
service's desired state.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 68. Release Locks

For high-risk production changes, a release lock or deployment queue can
prevent multiple unrelated promotion operations from colliding. The lock
should be observable and recoverable if a job dies while holding it.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 69. Change Audit

Git history should provide enough context to reconstruct deployment
intent. Commit messages, merge requests, approvals, pipeline IDs, image
digests, and release notes should be linked where possible. Avoid
generic messages such as 'update image' for important production
releases.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 70. Repository Tagging

Release tags can mark known production versions, but they should
complement rather than replace environment state. A tag can identify a
release point while the environment directory records exactly what each
cluster is running.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 71. Configuration Drift Between Environments

Avoid uncontrolled copying of entire production values into lower
environments. Define common defaults and explicit environment
differences. This reduces accidental security regressions and makes
configuration review easier.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 72. Environment Parity

Lower environments should be sufficiently similar to production to
detect meaningful deployment failures. Differences should be
intentional: capacity, external endpoints, data volume, and cost
controls may differ, but core security and deployment mechanics should
remain representative.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 73. GitOps Testing Strategy

Test repository changes before merge, after merge, and at deployment.
Pre-merge tests catch invalid configuration. Argo CD detects
synchronization problems. Kubernetes health checks and smoke tests
detect runtime problems. These are separate stages and should not be
treated as interchangeable.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 74. Policy Testing

Policy changes can break many applications. Test policies against
representative manifests before rollout. A new admission rule should be
deployed in an appropriate audit or controlled mode before becoming an
unconditional production blocker when organizational process allows.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 75. Large-Scale Repository Changes

Bulk updates such as changing a base chart or image digest across many
services should be generated deterministically, reviewed carefully, and
validated against all affected environments. Avoid hand-editing dozens
of files because inconsistent updates are difficult to detect.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 76. Renovation and Dependency Updates

When charts, controllers, or shared deployment components are updated,
validate compatibility with the target Kubernetes versions and
workloads. GitOps provides the desired-state history, but the platform
team still needs a controlled upgrade strategy.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 77. Argo CD Upgrade

Treat Argo CD itself as production software. Upgrade it through a
controlled process, review breaking changes, back up or reproduce
configuration, validate repository connectivity, and test application
reconciliation before broad rollout.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 78. GitOps Observability

Monitor repository pipeline failures, Argo CD synchronization errors,
degraded applications, repeated reconciliation failures, and
authentication problems. A GitOps platform needs its own observability
because a deployment system can fail independently of the application.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 79. Alerting

High-value alerts include production applications stuck out of sync,
repeated sync failures, degraded health, unauthorized repository
changes, expired repository credentials, and inability to reach the
source repository. Avoid alerting on every transient sync retry.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 80. Troubleshooting: Application OutOfSync

Check the Argo CD diff, Git commit, target path, values, destination
cluster, and any ignored differences. Determine whether the drift is
caused by an intended controller mutation, a manual change, or a genuine
desired-state mismatch.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 81. Troubleshooting: Sync Failed

Inspect Argo CD operation details and Kubernetes events. Validate the
rendered manifest independently. Common causes include RBAC denial,
invalid API fields, missing CRDs, immutable fields, admission rejection,
image-pull failures, and namespace/resource ownership conflicts.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 82. Troubleshooting: Healthy but Wrong Version

Confirm the GitOps digest, Argo CD rendered state, Deployment pod
template, ReplicaSet image, and actual container image ID. Do not rely
on a tag displayed in a dashboard. A digest comparison establishes
whether the intended artifact is actually running.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 83. Troubleshooting: Git Authentication

Check repository URL, credential validity, permissions, SSH host
verification where applicable, token expiration, branch access, and Argo
CD repository status. Rotate credentials through the approved process
rather than placing replacement credentials into manifests.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 84. Troubleshooting: Prune Removed Resource

If pruning removes an unexpected resource, identify which Application
owns it, inspect the Git path, resource tracking metadata, and recent
repository changes. Restore the desired resource through Git and then
correct ownership before enabling or continuing automated pruning.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 85. Troubleshooting: Infinite Reconciliation

Repeated changes may be caused by controllers mutating fields,
non-deterministic templates, timestamps generated during rendering,
defaulting behavior, or incorrect ignore rules. Find the exact field
changing on every reconciliation and eliminate the non-determinism
rather than disabling reconciliation.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 86. Production Runbook

A standard release runbook is: verify source pipeline is green -\>
verify image digest and security evidence -\> update or approve GitOps
change -\> validate rendered manifests -\> merge protected branch -\>
observe Argo CD sync -\> verify Kubernetes rollout -\> run smoke tests
-\> inspect key metrics -\> record release result. The rollback runbook
reverses the desired-state change to a known-good digest.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 87. Security Runbook

For a suspected malicious release: stop promotion, identify the digest,
query deployment inventory, inspect source and pipeline evidence, revoke
exposed credentials, isolate affected workloads if required, restore a
trusted digest through GitOps, preserve evidence, and perform root-cause
analysis.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 88. Production Review Checklist

Before approving a production GitOps change, verify target cluster,
namespace, application, image digest, chart version, environment values,
resource changes, security context, network exposure, secret references,
autoscaling settings, and rollback path. Confirm that only intended
files changed.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 89. Senior Interview: Explain GitOps

A strong answer is: Git is the declarative source of truth for desired
application state. CI builds and verifies the immutable artifact and
updates the GitOps repository with its digest. Argo CD continuously
compares Git with the EKS cluster and reconciles drift. This removes
normal Kubernetes deployment credentials from CI, improves auditability,
makes rollback deterministic, and separates artifact creation from
deployment reconciliation.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 90. Senior Interview: Why Not kubectl

Direct kubectl deployment from CI creates a privileged credential path
from the build system into production. GitOps instead records the
desired state in a protected repository and lets Argo CD reconcile it.
This gives review, audit history, drift detection, and a smaller CI
blast radius.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 91. Senior Interview: Multi-Environment

I promote the same immutable digest through environments. The
application image is not rebuilt for QA, staging, and production. GitOps
changes the desired digest and environment-specific configuration. This
preserves artifact integrity while still allowing environment-specific
resource and endpoint settings.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 92. Senior Interview: Rollback

I identify the last known-good immutable digest and revert the GitOps
desired state to that digest. Argo CD reconciles the cluster. I then
verify rollout health, smoke tests, and metrics. I avoid rebuilding an
older source revision during an incident because that can produce a
different artifact.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 93. Senior Interview: Drift

Drift means live state differs from Git. I first determine whether a
controller or admission mechanism intentionally changed a field. If it
is unauthorized manual drift, I restore the desired state through Git.
If the change is legitimate, I update Git so the source of truth
reflects the intended state.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 94. Final Architecture

The production flow is:

``` text
Developer
   |
Application Repository
   |
GitLab CI
   |  validate/test/security
   |  build/scan/SBOM
   v
Amazon ECR
   |
Immutable Image Digest
   |
GitOps Update
   v
Protected GitOps Repository
   |
   | validation / review
   v
Argo CD
   |
   | project/RBAC/admission controls
   v
EKS Cluster(s)
   |
Kubernetes Workloads
   |
Monitoring / Alerting / Audit
```

The architecture is deliberately declarative: CI produces evidence and
artifacts, Git records desired state, Argo CD reconciles, and Kubernetes
executes.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## 95. Final GitOps Principles

1.  Git contains desired state.
2.  Production changes are reviewable.
3.  CI does not need normal cluster-admin credentials.
4.  Deploy immutable image digests.
5.  Promote the same artifact between environments.
6.  Protect production repository paths.
7.  Validate rendered manifests before merge.
8.  Use Argo CD Projects and RBAC to constrain deployment scope.
9.  Treat secrets separately from ordinary Git configuration.
10. Make rollback a tested Git operation.
11. Monitor the GitOps control plane.
12. Preserve enough history to prove who deployed what, where, and when.

### Production implementation checks

-   Make the desired state explicit and reviewable.
-   Keep production paths protected.
-   Validate the effective rendered configuration.
-   Reference immutable artifacts for production.
-   Keep secrets out of ordinary configuration.
-   Preserve ownership and auditability.
-   Define rollback and recovery before an incident.

### Operational questions

1.  What exact cluster and environment does this change target?
2.  Which immutable artifact will run after reconciliation?
3.  What security and validation controls protect the change?
4.  How will drift be detected and corrected?
5.  How can the previous known-good state be restored?

## Complete GitOps Repository Blueprint

``` text
gitops/
├── apps/
│   ├── roboshop/
│   │   ├── frontend/
│   │   ├── catalogue/
│   │   ├── cart/
│   │   ├── user/
│   │   ├── shipping/
│   │   └── payment/
│   │
│   └── shared/
│
├── environments/
│   ├── dev/
│   │   ├── frontend/
│   │   ├── catalogue/
│   │   └── ...
│   ├── qa/
│   ├── staging/
│   └── prod/
│
├── clusters/
│   ├── dev-eks/
│   ├── qa-eks/
│   ├── staging-eks/
│   ├── prod-eks-primary/
│   └── prod-eks-secondary/
│
├── argocd/
│   ├── projects/
│   ├── applications/
│   └── applicationsets/
│
├── policies/
│
└── README.md
```

## End-to-End Promotion Model

``` text
Source Commit
     |
     v
GitLab CI
     |
     +--> Tests
     +--> DevSecOps
     +--> Build
     +--> SBOM
     +--> Scan
     |
     v
ECR
     |
     v
sha256:IMMUTABLE_DIGEST
     |
     v
GitOps Dev
     |
     v
GitOps QA
     |
     v
GitOps Staging
     |
     v
GitOps Production
     |
     v
Argo CD
     |
     v
EKS
```

## Final Production Checklist

-   [ ] Application and GitOps responsibilities are separated
-   [ ] GitOps repository structure is documented
-   [ ] Environment boundaries are explicit
-   [ ] Cluster destinations are explicit
-   [ ] Production branches are protected
-   [ ] CODEOWNERS/reviewer controls exist
-   [ ] GitOps CI validates changes
-   [ ] Helm rendering is tested
-   [ ] Kubernetes schema validation exists
-   [ ] Policy-as-code checks exist
-   [ ] Secrets are not stored in plaintext
-   [ ] Argo CD repository access is least privileged
-   [ ] Argo CD Projects restrict repositories/clusters/namespaces
-   [ ] Applications are clearly owned
-   [ ] ApplicationSet/App-of-Apps blast radius is understood
-   [ ] Image digests are used for production
-   [ ] Promotion reuses the same artifact
-   [ ] GitOps commits are attributable
-   [ ] Concurrent updates are handled safely
-   [ ] Drift is monitored
-   [ ] Prune/self-heal behavior is understood
-   [ ] Rollback is tested
-   [ ] GitOps disaster recovery is documented
-   [ ] ECR retention supports rollback
-   [ ] Argo CD itself is monitored
-   [ ] Production release runbook exists

## Capstone Principle

**GitOps turns deployment from an imperative command into a protected,
reviewable, auditable desired state. The strongest production design
connects source commit -\> verified artifact -\> immutable digest -\>
GitOps commit -\> Argo CD reconciliation -\> EKS workload, with every
transition protected and observable.**

---