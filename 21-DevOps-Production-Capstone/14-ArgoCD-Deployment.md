# 14 --- ArgoCD Deployment --- Production DevOps Capstone

> Deep production-oriented Argo CD deployment and operations guide for
> AWS EKS, GitOps, Helm, GitLab, multi-environment and multi-cluster
> platforms.

## Chapter Objective

This chapter takes the GitOps repository from the previous section and
builds the production Argo CD control plane that consumes it. The focus
is deployment architecture, high availability, security, authentication,
RBAC, Projects, Applications, ApplicationSets, multi-cluster management,
sync behavior, observability, troubleshooting, disaster recovery,
upgrades, and production operations.

## 1. Argo CD Role in the Production Platform

Argo CD is the continuous delivery and reconciliation component of the
capstone. It watches the approved GitOps desired state, compares it with
Kubernetes live state, and performs reconciliation. In production, Argo
CD should be treated as a critical platform service rather than as a
simple deployment utility. Its availability, access control, repository
credentials, cluster permissions, observability, backup strategy, and
upgrade process all require operational ownership.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 2. Target Architecture

The target architecture places Argo CD inside EKS and connects it to the
GitOps repository and one or more Kubernetes clusters. Application
manifests or Helm charts are stored in Git. Argo CD pulls the desired
state, renders Helm where configured, evaluates sync and health status,
and applies resources using its Kubernetes credentials. Users interact
with Argo CD through authenticated UI/API access, while normal
application changes flow through Git.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 3. Production Components

A production installation normally includes the Argo CD API server,
repository server, application controller, Redis, repo-server
sidecar/plugin components when required, and supporting Kubernetes
resources. The exact deployment model varies by Argo CD version and
scale. High availability requires multiple replicas for components that
support horizontal scaling and a careful understanding of which
components are stateful or coordination-sensitive.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 4. API Server

The API server provides the UI, API, authentication integration, and
application operations interface. Protect it with TLS, authentication,
network controls, and appropriate RBAC. Do not expose it publicly
without a deliberate ingress, identity, and security design.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 5. Application Controller

The application controller performs reconciliation and tracks
application state. It is one of the most important components because
excessive permissions or an incorrect configuration can affect many
workloads. Production installations should size it according to
application count, resource count, reconciliation frequency, and cluster
count.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 6. Repository Server

The repository server retrieves source repositories and renders
application configuration. Treat repository access as a sensitive trust
boundary. Repository credentials should be scoped and rotated. If custom
plugins are used, isolate and review them because they execute tooling
against repository content.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 7. Redis

Argo CD uses Redis for cached application and repository-related data.
Understand what is cache versus durable source of truth. Git remains the
desired-state source. Redis availability still affects controller
behavior and should be covered by production reliability and monitoring
plans.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 8. High Availability

High availability is not simply setting replicas to two. It requires
understanding component roles, scheduling across nodes or zones,
disruption budgets, resource requests, topology spread, service routing,
Redis behavior, repository availability, and cluster dependencies. Test
failure of individual pods and nodes rather than assuming the
installation is highly available.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 9. Namespace Strategy

A dedicated argocd namespace keeps the control plane separate from
application workloads. Do not allow arbitrary application resources to
be placed in the control namespace. Restrict who can modify resources in
the namespace and apply appropriate pod security and network controls.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 10. Installation Method

Helm is a common installation method because it is declarative and
integrates well with Terraform or GitOps. The installation itself should
be version-pinned and managed as code. Avoid manually changing
production Argo CD configuration without recording the equivalent
desired state.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 11. Helm Installation Baseline

A production Helm deployment should explicitly configure the Argo CD
version, replica counts, resources, service exposure, TLS behavior,
security context, affinity or topology rules, metrics, and any required
ingress. Review chart defaults for the exact chart version instead of
assuming values from another release.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 12. Version Pinning

Pin the Argo CD Helm chart and application version used by production.
Upgrade deliberately after reading compatibility notes and validating
against the target Kubernetes version. Avoid automatically consuming an
untested latest version in production.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 13. Resource Requests and Limits

Set realistic CPU and memory requests for Argo CD components based on
observed workload. Limits should be used carefully because overly low
limits can cause throttling or OOMKills during repository rendering or
reconciliation bursts. Capacity planning should be based on application
count and manifest complexity.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 14. Scheduling Across Failure Domains

Use pod anti-affinity or topology spread constraints where supported so
critical Argo CD components do not all land on one node. For a regional
production EKS cluster, distribute replicas across independent failure
domains when the node architecture and AWS availability zones permit it.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 15. Pod Disruption Budgets

PodDisruptionBudgets can protect availability during voluntary
disruptions. They must be compatible with replica counts; an overly
strict PDB can block node maintenance. Validate PDB behavior during
controlled drain tests.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 16. Security Context

Run Argo CD components with restrictive security contexts where
supported by the deployment. Drop unnecessary Linux capabilities, avoid
privileged execution, use non-root users where compatible, and use
read-only filesystems where the component supports them.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 17. Network Policies

Restrict communication to only what Argo CD requires: API clients to the
API server, controller to repository server and Kubernetes APIs,
repository server to approved source repositories, and necessary metrics
or notification endpoints. A default-deny model can be introduced
carefully with explicit allow rules.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 18. Ingress Architecture

A production Argo CD UI can be exposed through an AWS Application Load
Balancer using the Kubernetes AWS Load Balancer Controller. Terminate
TLS appropriately, restrict source access where possible, integrate
enterprise authentication if available, and avoid exposing
administrative endpoints more broadly than necessary.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 19. ALB Considerations

When using an ALB, account for HTTPS, health checks, listener rules,
security groups, idle timeouts, hostnames, certificate management, and
the traffic behavior of Argo CD's API/UI. Validate WebSocket or
streaming behavior if the selected Argo CD functionality requires it.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 20. TLS

Use TLS for external access and secure internal communication according
to the deployment's supported configuration. Certificates should be
managed through an approved mechanism such as AWS Certificate Manager
for the ingress boundary. Monitor expiration and define renewal
behavior.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 21. Authentication

Use a centralized identity provider when organizationally appropriate.
Authentication proves who the user is; Argo CD RBAC determines what that
identity can do. Do not treat successful SSO authentication as
equivalent to administrative authorization.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 22. SSO

OIDC-based SSO can integrate Argo CD with an enterprise identity
provider. Map groups or claims to Argo CD roles. Keep privileged groups
narrowly controlled and test behavior when group membership changes.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 23. Local Admin

The initial Argo CD local administrator account should be protected and
its use minimized. Store bootstrap credentials securely, rotate them
according to policy, and use SSO for normal operations where possible.
Do not leave default credentials in scripts or documentation.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 24. Argo CD RBAC

Argo CD RBAC should separate read-only access, application operations,
environment administration, and platform administration. Grant the
minimum required actions on the minimum required applications or
projects. Avoid broad wildcard permissions for application teams.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 25. Projects

Argo CD Projects are a major security boundary. A project can restrict
source repositories, destination clusters/namespaces, and permitted
resource kinds. Production projects should explicitly define what is
allowed rather than permitting arbitrary destinations and resource
creation.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 26. Production Project

A production project should allow only the approved GitOps repositories,
approved production clusters, and namespaces owned by the project.
Cluster-scoped resource creation should be limited to explicitly
required resource kinds and should receive additional review.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 27. Repository Registration

Register Git repositories with Argo CD using dedicated credentials.
Read-only access is preferred when the controller only needs to pull
configuration. SSH keys, tokens, or other supported methods should be
stored through Argo CD's secure secret mechanisms rather than Git.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 28. Private Git Repository

For a private GitLab repository, configure the repository URL and
credential with the required scope. Validate connectivity before
creating production Applications. When authentication fails, inspect
Argo CD repository status and credential expiry rather than repeatedly
changing Application manifests.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 29. Repository Credential Rotation

Credential rotation should be a planned operation. Create the new
credential, validate access, update Argo CD, confirm repository
synchronization, and then revoke the old credential. Avoid rotating
credentials during an active incident unless compromise requires it, in
which case follow the emergency process.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 30. Application Definition

An Argo CD Application connects a source path or chart to a destination
cluster and namespace. Keep Application definitions in Git where
possible. The definition itself is sensitive because changing the
repository path or destination can redirect deployments.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 31. Application Metadata

Use labels and annotations to identify owner, service, environment, cost
center, repository, and lifecycle information. Consistent metadata makes
filtering, auditing, and incident response easier.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 32. Application Source

The source can be a Git directory, Helm chart, or another supported
configuration source. For the capstone, prefer the GitOps repository as
the controlled source of production desired state and use pinned
revisions or protected branches according to the release model.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 33. Target Revision

The target revision determines which Git branch, tag, or commit Argo CD
follows. Production commonly follows a protected branch or a controlled
release revision. Pinning an exact commit can increase reproducibility
but requires an explicit promotion process when the desired state
changes.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 34. Destination

The destination identifies the target cluster and namespace. In
multi-cluster environments, avoid ambiguous names. Cluster registrations
and Application destinations should make it obvious which production
cluster will receive the workload.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 35. Automated Sync

Automated synchronization removes manual deployment steps. It is
appropriate when the GitOps review process is already a strong control.
Production automation should be paired with protected Git changes,
validated manifests, health checks, and a tested rollback path.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 36. Manual Sync

Manual synchronization gives operators an additional deployment approval
step. It can be useful for high-risk environments or migrations. Manual
sync is not a substitute for repository protection; unauthorized Git
changes can still alter the desired state.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 37. Prune

Prune allows Argo CD to remove resources no longer declared by Git. Use
it only when resource ownership is understood. A repository path mistake
can otherwise remove valid workloads. Test prune behavior in lower
environments before enabling it broadly in production.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 38. Self Heal

Self-heal corrects drift from the Git-defined desired state. This is
useful for protecting against accidental manual changes. During
emergency operations, responders must understand that direct changes may
be reverted and should be reconciled back into Git.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 39. Sync Options

Argo CD sync options influence resource application behavior. Options
should be explicitly chosen for known requirements such as server-side
apply or namespace creation. Do not enable options globally simply
because one workload requires them.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 40. Sync Waves

Use sync waves to control dependency ordering. A typical sequence might
create namespaces and configuration before deployments, then deploy
workloads that depend on those resources. Keep the dependency graph
simple and avoid using waves to hide architectural coupling.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 41. Hooks

PreSync, Sync, and PostSync hooks can execute controlled jobs around
deployment. Hooks should be idempotent and observable. A migration hook
that fails repeatedly can block an application indefinitely, so define
timeout, retry, and recovery behavior.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 42. Database Migration

Database migrations deserve special handling. Kubernetes application
rollback does not automatically roll back database schema changes.
Prefer backward-compatible schema migrations, separate schema deployment
from application rollout when appropriate, and document irreversible
changes.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 43. Health Assessment

Argo CD health status depends on Kubernetes resource conditions and, for
some resource types, custom health logic. A Deployment may be synced but
degraded if pods are unavailable. Use meaningful readiness probes and
resource conditions so Argo CD reflects real application health.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 44. Custom Health Checks

Custom health checks can be implemented for CRDs that need
application-specific health interpretation. Keep them deterministic and
version-controlled. A poorly designed health script can report healthy
when the resource is actually unusable.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 45. Sync Status vs Health Status

Synced means the live resource matches the desired configuration
according to Argo CD's comparison logic. Healthy means the resource
reports an acceptable operational condition. Production release
verification needs both; a synced but degraded application is not a
successful deployment.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 46. Notifications

Argo CD notifications can send deployment success, failure, sync, and
health events to approved channels. Notifications should be actionable
and routed to the service owner. Avoid creating noise for every routine
reconciliation event.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 47. Notification Security

Webhook URLs, tokens, and messaging credentials must be stored securely.
Notification payloads should not expose Kubernetes Secrets or sensitive
configuration. Validate the recipient endpoint and access controls.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 48. Metrics

Argo CD exposes metrics that can be scraped by Prometheus. Monitor
application sync status, health, reconciliation errors, controller queue
behavior, repository errors, API latency, and component resource
consumption. Use service-level indicators rather than collecting every
metric without purpose.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 49. Logging

Centralize Argo CD logs through the cluster logging pipeline. Useful
logs include repository authentication errors, application
reconciliation failures, Kubernetes API authorization failures, sync
errors, and controller exceptions. Keep enough context to correlate a
Git commit with a deployment event.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 50. Auditability

A production deployment should be traceable from Git commit to Argo CD
operation to Kubernetes workload. Preserve Git merge-request approvals,
CI evidence, image digest, Argo CD synchronization information, and
Kubernetes audit events according to retention requirements.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 51. ApplicationSet for Multi-Cluster

ApplicationSet can generate Applications from registered clusters or
structured Git directories. It is useful for fleet management, but a
generator mistake can deploy an application to every matching cluster.
Protect generator inputs and test changes in non-production clusters
first.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 52. Cluster Registration

Each managed EKS cluster should have a clear identity, environment,
region, and ownership. Store registration configuration securely and
limit which Argo CD Projects can target each cluster.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 53. Multi-Cluster Security

Argo CD requires credentials to access managed external clusters. Those
credentials are high-value because they can control workloads in the
target cluster. Use least privilege, rotate credentials, and restrict
each project to the clusters it actually manages.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 54. Hub-and-Spoke Model

A central Argo CD control plane can manage multiple clusters, while
separate Argo CD instances can provide stronger isolation for sensitive
environments. The correct model depends on organizational blast radius,
network connectivity, compliance, and operational scale.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 55. Production Cluster Isolation

For highly sensitive production environments, a dedicated Argo CD
instance or dedicated project boundary can reduce blast radius. If one
control plane manages development and production, a configuration error
in that control plane can potentially affect both.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 56. Disaster Recovery

Argo CD can be reconstructed when its configuration is declarative and
stored in Git. Keep installation configuration, Projects, Applications,
ApplicationSets, repository definitions, and required bootstrap
dependencies reproducible. Test rebuilding Argo CD in a clean cluster
rather than assuming backups alone are sufficient.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 57. Backup Strategy

Back up the GitOps repository through the source-control organization's
backup and retention strategy. Argo CD itself should be treated as
reconstructable infrastructure. Any state not represented in Git, such
as credentials or external identity configuration, needs an explicit
backup and recovery procedure.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 58. EKS Cluster Rebuild

A cluster recovery flow is: recreate AWS infrastructure with Terraform
-\> install required EKS add-ons/controllers -\> install Argo CD -\>
restore repository connectivity -\> apply Projects and Applications -\>
synchronize workloads -\> validate secrets, ingress, storage,
networking, and application health. This should be rehearsed.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 59. Upgrade Planning

Before an Argo CD upgrade, record the current version, Helm chart
version, Kubernetes version, repository configuration, plugins, custom
health checks, notification configuration, and known compatibility
constraints. Test in a lower environment before production.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 60. Upgrade Execution

Upgrade one controlled environment first, verify API/UI availability,
repository connectivity, application reconciliation, health evaluation,
notifications, and metrics, then proceed to production. Maintain a
rollback or downgrade plan where the supported upgrade path permits it.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 61. CRD Management

Argo CD and related controllers may depend on CRDs. CRD upgrades can
have compatibility implications. Treat CRD changes as platform changes
and validate them separately from ordinary application releases.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 62. Security Hardening

Harden Argo CD by limiting administrative access, protecting ingress,
using secure repository credentials, restricting Projects, minimizing
cluster permissions, applying network policies, running restrictive pod
security settings, and monitoring authentication and administrative
events.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 63. Admin Access

Administrative access should be exceptional and audited. Platform
administrators should use named identities rather than shared accounts.
If emergency access is required, document the reason, duration, and
actions performed.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 64. Least Privilege to Kubernetes

Argo CD needs permissions for resources it manages, but cluster-admin is
broader than necessary in many architectures. Design resource
permissions according to actual managed resource types and namespaces.
Be careful with cluster-scoped CRDs, RBAC, namespaces, and admission
resources because they have broad effects.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 65. Resource Ownership

Avoid multiple Argo CD Applications managing the same resource.
Ownership conflicts can produce unexpected reconciliation behavior and
difficult outages. Define ownership boundaries clearly, especially for
namespaces, shared ingress objects, CRDs, and cluster-wide policies.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 66. Sync Collision

If two Applications attempt to manage the same resource, determine which
Application should own it. Split shared resources into a dedicated
platform Application or redesign the configuration so each resource has
a single owner.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 67. Application Deletion

Deleting an Argo CD Application can optionally cascade to managed
resources depending on configuration. Protect critical Applications and
require deliberate approval for destructive operations.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 68. Finalizers

Argo CD uses Kubernetes finalizers to track resource ownership and
deletion behavior. Understand the consequences before manually removing
a finalizer during an incident; doing so can bypass intended cleanup
behavior and leave orphaned resources.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 69. Namespace Creation

Automatically creating namespaces can simplify onboarding but can also
hide namespace ownership and security policy requirements. Production
namespaces should have labels, resource quotas, network policies, and
pod security configuration applied consistently.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 70. Resource Quotas

Production namespaces should use resource quotas where appropriate to
prevent one workload from exhausting shared cluster capacity. Argo CD
should deploy quota definitions as part of the namespace platform
configuration when it owns that responsibility.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 71. Limit Ranges

LimitRanges can provide default or minimum resource settings. Ensure
these defaults do not conflict with Helm chart resource configuration.
Unexpected defaults can alter scheduling behavior and application
performance.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 72. Application Onboarding

A new service onboarding workflow should create the GitOps path, Helm
values, Application or ApplicationSet registration, ownership metadata,
resource policies, ingress configuration, secret references, monitoring
configuration, and rollback documentation. Do not treat adding one YAML
file as complete onboarding.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 73. Release Workflow

The production workflow is: developer merges code -\> GitLab CI tests
and secures the build -\> image is pushed to ECR -\> immutable digest is
captured -\> GitOps promotion updates the desired state -\> GitOps CI
validates -\> protected merge occurs -\> Argo CD detects the commit -\>
sync occurs -\> Kubernetes rollout completes -\> health and business
checks are verified.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 74. Canary and Progressive Delivery

Argo CD can be combined with progressive delivery tooling for canary or
blue-green strategies. If the capstone adopts progressive delivery, keep
rollout configuration in Git and make metrics-driven promotion explicit.
Do not assume a Kubernetes Deployment rolling update provides canary
analysis.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 75. Rollback

Rollback should change the desired GitOps state back to a known-good
artifact digest. Argo CD then reconciles the rollback. If an emergency
direct rollback is required, reconcile the repository immediately
afterward to prevent Git drift.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 76. Troubleshooting: Repo Unreachable

Check Argo CD repository status, DNS, network policies, egress security
groups, proxy configuration, TLS trust, repository credentials, token
expiration, and Git provider availability. Confirm whether the failure
affects one repository or all repositories.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 77. Troubleshooting: Permission Denied

Determine whether the failure is at Git authentication, Argo CD Project
authorization, Kubernetes RBAC, or admission policy. The error location
matters: changing Kubernetes permissions will not fix a Git repository
authentication problem.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 78. Troubleshooting: Application Stuck

Inspect application operation history, controller logs, resource events,
health status, sync waves, hooks, and pending resources. A stuck hook,
missing CRD, failed admission policy, or unavailable dependency can
block synchronization.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 79. Troubleshooting: Pods Not Starting

Argo CD may correctly report the desired Deployment while Kubernetes
reports degraded health. Inspect Deployment, ReplicaSet, Pod events,
image pull status, service-account permissions, secrets, config maps,
node capacity, probes, and admission events.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 80. Troubleshooting: Wrong Cluster

Confirm the Application destination, registered cluster identity,
ApplicationSet generator output, project destination rules, and actual
Kubernetes context. Never assume a production-looking Application name
proves the destination is correct.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 81. Troubleshooting: Continuous OutOfSync

Compare desired and live manifests and identify the exact field
changing. Determine whether a controller is mutating it, whether Helm
rendering is non-deterministic, or whether an external process is
modifying the resource. Use controlled ignore differences only when
ownership is intentional.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 82. Troubleshooting: Sync Rejected

Admission controllers can reject resources even when Argo CD has correct
Kubernetes RBAC. Inspect admission webhook messages and policy reports.
Fix the desired configuration in Git instead of repeatedly forcing the
sync.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 83. Troubleshooting: Argo CD API Down

Check API server pods, service endpoints, ingress or ALB health,
certificates, authentication provider availability, resource pressure,
and network policies. If the API is unavailable but the controller
continues operating, application reconciliation may still proceed;
verify actual application state rather than assuming deployments have
stopped.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 84. Troubleshooting: Controller Overloaded

Look for excessive application count, very large manifests, high
reconciliation frequency, repository rendering load, API-server
throttling, and insufficient controller resources. Scale or tune based
on measured bottlenecks rather than blindly increasing replicas.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 85. Troubleshooting: Redis Problems

Inspect Redis availability, connection errors, pod restarts, resource
pressure, and network connectivity. Determine whether the impact is
cache-related or prevents required controller/API behavior. Follow the
supported Argo CD recovery process rather than deleting Redis data
casually.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 86. Production Runbook

For a normal release: validate GitOps change -\> verify Argo CD
application target -\> merge protected change -\> wait for sync -\>
inspect operation result -\> verify health -\> validate workload rollout
-\> execute smoke tests -\> inspect metrics/logs -\> record release. For
failure: stop further promotion -\> identify failing boundary -\>
restore or correct desired state -\> verify recovery -\> document the
incident.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 87. Security Incident Runbook

If Argo CD credentials, repository access, or cluster credentials are
suspected compromised, restrict access, revoke or rotate affected
credentials, inspect Git and Argo CD audit evidence, identify
unauthorized Applications or sync operations, restore trusted
configuration, and validate all managed clusters.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 88. Senior Interview: Explain Argo CD

A strong answer is: Argo CD is a declarative GitOps continuous delivery
controller. It watches the desired state in Git, compares it with
Kubernetes live state, reports sync and health status, and reconciles
differences. In our EKS architecture, GitLab CI builds and verifies
immutable images and updates the GitOps repository; Argo CD performs the
actual Kubernetes reconciliation. This reduces CI-to-cluster credentials
and improves auditability and drift correction.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 89. Senior Interview: Why Argo CD Instead of kubectl

Direct kubectl from CI requires a deployment identity with cluster
permissions. Argo CD moves that responsibility to a dedicated controller
and makes Git the deployment intent. We gain reviewable changes,
reconciliation, drift detection, health visibility, and a consistent
multi-cluster deployment model.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 90. Senior Interview: How Do You Secure Argo CD

I protect ingress with TLS and enterprise authentication, restrict Argo
CD RBAC, use Projects to constrain repositories and destinations, use
least-privilege cluster permissions, protect repository credentials,
isolate the namespace, apply network and pod security controls, monitor
audit and reconciliation events, and keep Argo CD configuration
reproducible through Git and infrastructure code.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 91. Senior Interview: How Do You Manage Multiple Clusters

I register each EKS cluster with an explicit identity and use Argo CD
Projects and ApplicationSets to control which applications can target
which clusters. Cluster-specific differences are represented
declaratively. For highly sensitive environments, I evaluate separate
Argo CD instances to reduce blast radius.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 92. Senior Interview: What Is Drift

Drift occurs when live Kubernetes state differs from the desired state
in Git. I inspect the diff to determine whether the difference is
intentional controller mutation or unauthorized change. With self-heal,
Argo CD can restore the desired state. If the live change is legitimate,
I update Git so the source of truth remains correct.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 93. Senior Interview: Production Failure

If an application is stuck during sync, I first identify the Argo CD
operation failure and exact resource. Then I inspect Kubernetes events,
admission errors, RBAC, missing CRDs, hooks, image pulls, and health
conditions. I avoid random changes and fix the boundary that actually
rejected the desired state.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 94. Senior Interview: Disaster Recovery

I rebuild EKS from Terraform, install platform dependencies, bootstrap
Argo CD, restore repository and identity access, apply Projects and
Applications, and allow reconciliation. Because the desired application
state and infrastructure are declarative and images are retained by
digest, recovery can be repeatable. I validate the process regularly
instead of relying on documentation alone.

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 95. Final Production Architecture

``` text
                    GitOps Repository
                           |
                    Protected Branch
                           |
                    GitOps CI Validation
                           |
                           v
                    +---------------+
                    |    Argo CD    |
                    |               |
                    | API / UI       |
                    | Controller    |
                    | Repo Server   |
                    | Redis         |
                    +-------+-------+
                            |
              +-------------+-------------+
              |                           |
          EKS Cluster A              EKS Cluster B
              |                           |
        Production Apps             DR/Secondary Apps
              |                           |
        Kubernetes API              Kubernetes API
              |
        Admission / RBAC
              |
          Workloads
              |
       Monitoring / Logs
```

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?

## 96. Final Production Checklist

-   [ ] Argo CD version is pinned and upgrade process documented
-   [ ] HA design matches workload scale
-   [ ] Critical pods are distributed across failure domains
-   [ ] Resources are sized from measurements
-   [ ] PDBs are tested
-   [ ] TLS and ingress are secured
-   [ ] SSO and RBAC are configured
-   [ ] Local admin is protected
-   [ ] Projects restrict repositories and destinations
-   [ ] Repository credentials are least privileged
-   [ ] Application ownership is unambiguous
-   [ ] Production Applications are protected
-   [ ] Sync/prune/self-heal behavior is understood
-   [ ] Health checks are meaningful
-   [ ] Notifications are actionable
-   [ ] Metrics and logs are collected
-   [ ] Multi-cluster access is least privileged
-   [ ] ApplicationSet blast radius is controlled
-   [ ] GitOps state is backed up
-   [ ] Argo CD bootstrap is reproducible
-   [ ] EKS rebuild procedure is tested
-   [ ] Rollback is tested
-   [ ] Incident runbook exists
-   [ ] Upgrade process is tested

### Production implementation checks

-   Keep the control plane reproducible and version-controlled.
-   Protect every path that can change a production destination.
-   Use least privilege for repositories and clusters.
-   Validate behavior under failure, not only successful installation.
-   Keep rollback and disaster recovery executable.

### Operational questions

1.  What happens if this component fails?
2.  What permissions does it actually require?
3.  How is the configuration recovered?
4.  How is a production change audited?
5.  What is the safest troubleshooting path?
