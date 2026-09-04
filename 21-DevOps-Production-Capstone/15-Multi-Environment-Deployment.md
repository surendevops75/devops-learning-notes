# Multi-Environment Deployment

> Deep production deployment strategy for Dev, QA, Staging, and
> Production using GitLab CI, ECR, Helm, GitOps, Argo CD, EKS, immutable
> artifacts, approvals, progressive delivery, rollback, and environment
> isolation.

## Chapter Objective

This chapter defines how the capstone promotes the same verified
artifact across multiple environments without sacrificing security,
reproducibility, auditability, or operational safety.

## 1. Environment Promotion Philosophy

A production-grade deployment model separates application artifact
creation from environment promotion. The same immutable container digest
should move from development to QA, staging, and production after the
required validation and approvals. Environment-specific configuration
changes independently through GitOps. This prevents a production
deployment from being a newly rebuilt artifact that was never tested in
lower environments.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 2. Environment Boundaries

Each environment represents a distinct operational trust boundary.
Development optimizes for rapid feedback, QA validates integration
behavior, staging approximates production, and production protects
availability, security, and customer impact. The repository, Kubernetes
namespace or cluster, AWS account boundaries, secrets, databases, and
deployment permissions should reflect those differences where
appropriate.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 3. Recommended Environment Model

For the capstone, use dev, qa, staging, and prod as explicit
environments. Dev can run in a lower-cost EKS cluster or namespace,
while production should use stronger isolation and capacity. Staging
should resemble production architecture closely enough to detect
deployment and infrastructure problems before release.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 4. Environment Matrix

A useful matrix records cluster, AWS account, namespace, Git path,
approval requirement, synchronization mode, replica baseline, external
endpoint, secrets source, and rollback policy. The matrix becomes an
operational contract and prevents accidental assumptions such as
treating staging as identical to production when it is not.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 5. Same Artifact Promotion

Suppose CI builds image digest sha256:ABC. Dev deploys sha256:ABC, QA
later deploys the same digest, staging deploys sha256:ABC, and
production deploys sha256:ABC after approval. The binary content is
unchanged throughout the promotion chain. Only deployment intent and
environment configuration change.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 6. Why Rebuilding Per Environment Is Dangerous

Rebuilding the application for every environment can introduce different
dependency resolution, base-image changes, timestamps, compiler output,
or build-time configuration. It makes test evidence weaker because the
artifact tested in staging is not necessarily the artifact deployed to
production. Build once and promote the immutable artifact instead.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 7. Promotion Gates

A promotion gate should combine automated and human evidence. Examples
include unit and integration test results, SAST/SCA results, container
scanning, SBOM generation, deployment health, smoke tests, business
validation, change approval, and security approval for high-risk
changes.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 8. Dev Promotion

Development should provide fast feedback. A successful CI pipeline can
update the dev GitOps path automatically. The deployment should still
use the immutable digest and pass GitOps validation. Automatic dev
deployment should not imply automatic production deployment.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 9. QA Promotion

QA promotion should occur only after dev validation succeeds. Automated
integration and API tests can run against the deployed service. Failures
should stop promotion rather than allowing the same artifact to advance
without evidence.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 10. Staging Promotion

Staging should be treated as the final production-like validation
environment. Validate deployment behavior, migrations, ingress,
autoscaling, observability, security controls, and rollback behavior
where feasible. The goal is to reduce production-only surprises.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 11. Production Promotion

Production promotion should require the organization's defined approval
level. The GitOps change should identify the exact digest and
configuration. After merge, Argo CD reconciles the desired state and
operators verify health, metrics, logs, and business-level smoke tests.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 12. Promotion Strategies

There are several valid strategies: a single GitOps repository with
environment directories, separate repositories per environment, release
branches, tags, generated promotion pull requests, or an
environment-controller pattern. The best choice depends on access
isolation, scale, compliance, and team ownership.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 13. Recommended Strategy

For this capstone, use a protected GitOps repository with explicit
environment directories and automated promotion pull requests. CI
publishes the immutable artifact, updates dev automatically, and creates
controlled promotion changes for QA, staging, and production. Production
remains protected by reviewers and policy.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 14. Directory Layout

A practical structure is:

``` text
environments/
├── dev/
│   ├── frontend/
│   ├── catalogue/
│   └── ...
├── qa/
│   ├── frontend/
│   ├── catalogue/
│   └── ...
├── staging/
│   ├── frontend/
│   ├── catalogue/
│   └── ...
└── prod/
    ├── frontend/
    ├── catalogue/
    └── ...
```

Each application directory should contain only the configuration needed
to render the desired state for that environment.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 15. Application Configuration Layering

Separate application defaults from environment overrides. The Helm chart
should define safe defaults. Environment values should override
replicas, resources, ingress hosts, feature flags, endpoints, and
autoscaling settings where necessary. Avoid copying complete charts into
every environment because duplicated templates drift over time.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 16. Helm Values Example

A conceptual production values file is:

``` yaml
image:
  repository: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/catalogue
  digest: sha256:EXACT_DIGEST

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
```

Production values should be reviewed because configuration changes can
alter capacity, cost, availability, and security.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 17. Immutable Promotion Metadata

Store the image digest together with useful release metadata such as
source commit, image tag, build pipeline ID, SBOM identifier, and
release timestamp. The digest remains the authoritative artifact
identifier; metadata makes investigation easier.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 18. GitOps Promotion Pull Request

An automated promotion PR should clearly show the old digest, new
digest, target environment, source commit, security evidence, and
successful lower-environment checks. Reviewers should not have to search
through CI logs to determine what artifact is being promoted.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 19. Promotion PR Example

A promotion change can conceptually look like:

``` diff
 image:
   repository: <ecr-repository>
-  digest: sha256:OLD_DIGEST
+  digest: sha256:NEW_TESTED_DIGEST
```

The PR description should link the source release, CI pipeline, test
evidence, and lower-environment deployment result.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 20. Promotion Ordering

The normal order is dev -\> QA -\> staging -\> production. Do not allow
production to advance before the artifact has passed the required
lower-environment gates. Emergency releases may use a documented
exception path with explicit approval and evidence.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 21. Promotion Automation

A CI job can read the newly published image digest, update the target
GitOps values file, validate the resulting manifest, and create a pull
request. The job must avoid blindly replacing unrelated changes and
should fail safely if the file changed concurrently.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 22. Concurrent Promotion Changes

Two releases may attempt to promote the same service simultaneously.
Protect against lost updates using serialized jobs, pull/rebase/retry
logic, or merge-request workflows. The automation should never overwrite
a human change without detecting it.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 23. Release Candidate

A release candidate represents a specific immutable artifact plus its
deployment metadata. Once a candidate is promoted to staging, its digest
should remain fixed. If a new build is required, create a new candidate
and restart the required validation chain.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 24. Release Freeze

A production release freeze can stop automatic promotion during
incidents, holidays, major business events, or maintenance windows. The
freeze should be visible and have a defined emergency override process.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 25. Change Approval

Approval should correspond to risk. Routine low-risk application changes
can use automated checks plus standard review. Database changes,
networking changes, IAM changes, security policies, and cluster-wide
resources deserve stronger review because their blast radius is larger.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 26. Environment-Specific Secrets

Development, QA, staging, and production must not share production
credentials. Each environment should reference its own secret store
paths or secret objects. Workload identity should provide access only to
the secrets required by that environment.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 27. Secret Path Convention

A consistent naming model such as /roboshop/dev/catalogue/*,
/roboshop/qa/catalogue/*, and /roboshop/prod/catalogue/\* makes access
policies easier to review. The exact convention is organizational, but
environment isolation must be explicit.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 28. AWS Account Separation

High-assurance architectures commonly use separate AWS accounts for
production and non-production. This reduces the blast radius of IAM
mistakes and accidental infrastructure operations. GitOps environment
paths should align with the account and cluster boundaries so the target
is obvious.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 29. Cluster Separation

Production can use a dedicated EKS cluster while lower environments
share one or more clusters. A shared cluster lowers cost but increases
blast radius. Namespace isolation, RBAC, network policies, quotas, and
admission policies become especially important when environments share
infrastructure.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 30. Namespace Separation

If multiple environments share a cluster, use distinct namespaces and
enforce boundaries. Never rely only on naming. Apply RBAC,
NetworkPolicies, ResourceQuotas, LimitRanges, and admission controls to
prevent cross-environment access.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 31. Database Separation

Production databases should be isolated from non-production databases.
Never point QA or staging workloads at production databases unless a
carefully controlled read-only testing architecture explicitly requires
it. Prefer synthetic or sanitized test data.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 32. Data Promotion

Application artifacts can move through environments; production data
generally should not. Database schema changes require
backward-compatible migration strategies and independent validation. Do
not treat a database migration as an ordinary image promotion.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 33. Database Migration Pattern

Use expand-and-contract migrations: first add backward-compatible schema
elements, deploy code that can use both old and new forms, migrate data,
then remove obsolete structures in a later controlled change. This makes
rollback safer because the old application can continue operating during
intermediate phases.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 34. Migration Ordering

If a new application version requires a new column, the schema change
should become available before the application requires it. If the
application starts using a new field before the database supports it,
production can fail immediately. GitOps sync waves can help with
ordering, but compatibility should primarily come from application
design.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 35. Feature Flags

Feature flags can decouple code deployment from feature activation.
Deploy the code through GitOps, keep the feature disabled, validate
infrastructure and application health, then activate the feature through
the approved configuration system. Do not store sensitive feature flags
in plaintext secrets.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 36. Configuration Drift Between Environments

Differences should be intentional and documented. Examples include
replica counts, resource sizes, external URLs, feature flags,
autoscaling ranges, and log levels. Security defaults should generally
remain consistent unless there is a documented reason to differ.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 37. Production-Like Staging

Staging should reproduce production topology where practical: ingress
behavior, TLS, service accounts, network policies, observability,
autoscaling, and deployment strategy. It can use smaller capacity, but
changing fundamental architecture between staging and production weakens
validation.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 38. Smoke Testing

After each promotion, run targeted smoke tests such as health endpoints,
authentication flow, core API requests, service-to-service calls, and
critical user journeys. Smoke tests should be fast enough to run after
every deployment and deterministic enough to provide useful release
evidence.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 39. Integration Testing

Integration tests should validate dependencies such as databases,
caches, queues, and external APIs. They should run against the actual
environment where possible. A green unit-test suite does not prove the
Kubernetes deployment or service integration works.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 40. Post-Deployment Verification

Verification should include pod readiness, Deployment rollout status,
service endpoints, ingress response, error rate, latency, resource
usage, logs, and business smoke tests. A deployment is not complete
merely because Argo CD reports Synced.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 41. Progressive Production Promotion

For high-risk services, promote to a small portion of production first
using canary or blue-green techniques. Metrics such as error rate,
latency, saturation, and business KPIs can determine whether the release
should continue. Keep the promotion controller and thresholds
version-controlled where possible.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 42. Blue-Green

Blue-green deployment maintains two application versions and shifts
traffic between them. It can make rollback fast, but it requires
sufficient capacity and careful handling of sessions, caches, database
compatibility, and external dependencies.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 43. Canary

Canary deployment sends a controlled percentage of traffic to the new
version before full rollout. It is effective when meaningful telemetry
exists. A canary without reliable metrics is only a partial rollout, not
a real safety mechanism.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 44. Rolling Deployment

Standard Kubernetes rolling updates replace old pods progressively.
Configure maxUnavailable and maxSurge appropriately. Rolling updates are
simpler than canaries but do not automatically provide business-level
validation or automatic rollback based on metrics.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 45. Rollback Trigger

Define rollback triggers before deployment. Examples include sustained
elevated error rate, failed readiness, critical business transaction
failures, latency regression, crash loops, resource exhaustion, or
security validation failure. Avoid waiting for a major outage before
deciding what constitutes release failure.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 46. Automatic Rollback

Automatic rollback can be valuable for measurable failures, but it
should be used carefully. Application rollback cannot safely reverse
every database or external-system change. Use automation where the
rollback is deterministic and test it under realistic failure
conditions.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 47. Manual Rollback

A manual rollback changes the GitOps desired digest to the previous
known-good version and allows Argo CD to reconcile. Record the reason
and evidence. After stabilization, investigate why the release passed
earlier gates but failed in the target environment.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 48. Emergency Promotion

A critical security patch may require accelerated promotion. The
emergency path should still identify the exact immutable digest, record
approvals, run the minimum viable validation, and preserve audit
evidence. Emergency does not mean undocumented.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 49. Emergency Rollback

For an active production incident, prioritize service restoration.
Revert the GitOps production digest to the last known-good version,
observe Argo CD and Kubernetes rollout, validate service health, and
then freeze further promotion until root cause is understood.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 50. Environment Access Matrix

A useful access model is: developers can propose changes to application
paths; QA engineers can approve QA; service owners can approve staging;
production requires designated reviewers; platform administrators
control Argo CD and cluster-level configuration. Exact roles should
match the organization's separation-of-duties requirements.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 51. Production Repository Protection

Production paths should require code review, successful GitOps
validation, and appropriate approvals. Direct pushes should be
restricted. Changes to Argo CD Applications, cluster destinations,
security policies, and production secrets references should have
elevated review requirements.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 52. CODEOWNERS Strategy

Use ownership rules such as service owners for application values,
platform owners for Argo CD configuration, and security owners for
policies. This prevents a routine application contributor from silently
changing a cluster-wide security control.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 53. Environment Labels

Apply labels such as environment, app, team, and cluster to Kubernetes
resources. These labels support monitoring, cost allocation, policy
enforcement, and incident investigation. Standardize them through Helm
helpers or shared templates.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 54. Promotion Audit Trail

The audit trail should connect source commit -\> CI pipeline -\> image
digest -\> GitOps PR -\> approval -\> Argo CD sync -\> Kubernetes
rollout. This chain lets an operator answer who deployed what, where,
when, and why.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 55. Release Notes

Release notes should summarize application changes, image digest,
migration requirements, configuration changes, known risks, validation
performed, and rollback instructions. Automated generation from commit
metadata can reduce manual effort.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 56. GitOps Branch Strategy

Avoid complex branch models unless they solve a real isolation
requirement. A protected main branch with environment-specific
directories and promotion PRs is often easier to reason about. If
release branches are used, define how fixes flow back so branches do not
diverge permanently.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 57. Environment Tags

Git tags can mark release points, but environment directories remain the
source of deployment intent. A tag alone does not prove which cluster is
running the release. Keep environment state explicit.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 58. Application Version Compatibility

Before promoting an artifact, verify compatibility with
environment-specific dependencies. Examples include Kubernetes API
versions, database schema, queue contracts, external APIs, and shared
platform versions. Environment promotion is not only an image-copy
operation.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 59. Contract Testing

For microservices, contract tests can detect incompatible API changes
before production. Producer and consumer compatibility should be
validated before promotion when services are released independently.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 60. Messaging Compatibility

For Kafka or RabbitMQ consumers and producers, promotion should preserve
message contract compatibility. Schema evolution should support old and
new consumers during rolling deployment. Do not assume rolling
Kubernetes updates protect you from incompatible message formats.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 61. Observability Gates

Production promotion can require dashboards and alerts to be healthy
before the next release. At minimum, verify error rate, latency,
saturation, restart counts, and dependency health. Observability should
provide enough evidence to decide whether a release is safe.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 62. SLO-Based Promotion

For mature platforms, deployment decisions can use service-level
objectives. A release can be paused when error-budget burn or latency
exceeds a defined threshold. This connects deployment automation to
customer impact rather than only pod health.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 63. Cost-Aware Environments

Non-production environments should use capacity appropriate to their
workload while preserving meaningful architecture. Production should use
capacity and autoscaling settings based on measured demand. Do not copy
production capacity blindly into dev, and do not shrink staging so far
that important behavior disappears.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 64. Autoscaling Differences

HPA minimum and maximum replicas can differ by environment. Staging
should still exercise autoscaling behavior when that is important to
production. Production values should be reviewed carefully because an
incorrect maximum can create both availability and cost problems.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 65. Resource Configuration

Resource requests influence scheduling and capacity; limits constrain
runtime behavior. Environment-specific resource values should be tested
because a configuration that works in staging may fail under production
load. Use load testing when capacity is business-critical.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 66. Network Differences

Development may allow broader outbound access, while production should
use controlled egress and network policies. If staging is too
permissive, security and connectivity problems may not appear until
production. Reproduce important production network restrictions in
staging.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 67. Ingress Differences

Production ingress should use real TLS, WAF or appropriate edge
controls, DNS, and approved hostnames. Staging should exercise the same
ingress controller and certificate workflow with non-production domains.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 68. Service Account Differences

Each environment should have distinct Kubernetes service accounts and
workload identities. Production IAM roles should never be reused by
lower environments. This prevents a compromised non-production workload
from inheriting production AWS permissions.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 69. Policy Differences

Security policies should generally be at least as strict in staging as
production for high-risk controls. If a policy is relaxed in staging,
document the reason and recognize that staging cannot fully validate
production admission behavior.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 70. Promotion Failure Handling

If promotion fails, stop the chain. Investigate whether the failure is
artifact-related, configuration-related, environment-related, or
platform-related. Do not repeatedly retry without understanding the
failure because retries can create duplicate jobs, conflicting commits,
or partial changes.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 71. Failed GitOps PR

If the promotion PR fails validation, leave the target environment
unchanged. Correct the GitOps configuration, rerun validation, and
regenerate or update the PR. The automation should fail closed rather
than merging a partially validated change.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 72. Failed Argo CD Sync

If Argo CD cannot sync the promoted state, inspect the exact resource
failure. If the production application remains healthy on the old
version, do not force destructive actions merely to make the sync green.
Correct the desired state or roll back to the known-good digest.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 73. Failed Smoke Test

If deployment succeeds but smoke tests fail, treat the release as
unsuccessful. Compare application logs, metrics, dependency health, and
configuration between old and new versions. If customer impact is
occurring, roll back quickly while preserving evidence.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 74. Failed Migration

If a migration fails, determine whether it is safe to retry, whether the
migration is partially applied, and whether the application can operate
with the current schema. Never blindly roll back database changes.
Follow the migration-specific recovery procedure.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 75. Promotion Dashboard

Create a release dashboard showing current digest by environment, Argo
CD sync status, rollout status, deployment time, error rate, latency,
restarts, and recent release identifier. This provides a single
operational view during promotion.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 76. Environment Inventory

Maintain an inventory mapping service -\> environment -\> cluster -\>
namespace -\> current digest -\> chart version -\> owner. This can be
generated automatically from GitOps and Argo CD data. The inventory is
extremely useful during incidents and audits.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 77. Production Release Checklist

Before production promotion: verify tested digest, successful lower
environments, security evidence, configuration review, migration plan,
capacity, observability, approval, rollback digest, and change window.
After promotion: verify Argo CD, rollout, application health, metrics,
logs, and business smoke tests.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 78. Senior Interview: How Do You Promote?

I build once and promote the same immutable image digest through dev,
QA, staging, and production. Each environment has its own GitOps
configuration. Promotion updates the digest through a protected Git
workflow, lower-environment validation gates the next stage, and
production requires the defined approval. Argo CD then reconciles the
approved desired state.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 79. Senior Interview: Why Digest?

A digest identifies exact image content. A tag can be moved, so
deploying a tag alone weakens reproducibility. I retain a human-readable
tag for traceability but deploy the immutable digest in production.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 80. Senior Interview: How Do You Handle Environment Differences?

I keep common application logic in Helm and environment-specific
configuration in values or overlays. Differences such as replica counts,
resources, ingress hosts, endpoints, and feature flags are explicit.
Security boundaries and production credentials are isolated rather than
copied between environments.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 81. Senior Interview: Production Approval

The production approval reviews the exact GitOps change, artifact
digest, validation evidence, configuration impact, migration
requirements, and rollback plan. Approval should happen on the desired
state that will actually be deployed, not merely on a CI build.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 82. Senior Interview: Rollback

I revert the production GitOps reference to the last known-good
immutable digest and let Argo CD reconcile. I verify rollout and
business health. If a database migration is involved, I follow the
migration rollback strategy separately because application rollback does
not automatically reverse schema changes.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 83. Senior Interview: Multi-Cluster

I make the cluster destination explicit and use Argo CD Projects and
ApplicationSets to control placement. Production clusters have separate
access boundaries. A promotion changes the desired state for the
intended cluster or cluster fleet rather than relying on an operator's
current kubectl context.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 84. Senior Interview: Why Staging Matters

Staging is the final production-like validation boundary. I use it to
validate Kubernetes rollout behavior, ingress, secrets integration,
autoscaling, observability, security controls, dependencies, and smoke
tests. If staging differs fundamentally from production, its release
evidence has limited value.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 85. Senior Interview: Emergency Release

For an emergency security fix, I still build a uniquely identifiable
immutable artifact, run the minimum required security and functional
validation, obtain the emergency approval, promote through the
controlled GitOps path, and monitor closely. I preserve evidence and
follow up with full validation afterward.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 86. Final Environment Flow

``` text
Developer Commit
      |
      v
GitLab CI
  | tests
  | security
  | build
  | SBOM
  v
ECR Immutable Digest
      |
      v
DEV GitOps ----> Argo CD ----> Dev EKS
      |
      | automated validation
      v
QA Promotion PR
      |
      v
QA GitOps -----> Argo CD ----> QA EKS
      |
      | integration validation
      v
Staging Promotion PR
      |
      v
Staging GitOps -> Argo CD ----> Staging EKS
      |
      | production-like validation
      v
Production Approval
      |
      v
Prod GitOps ----> Argo CD ----> Production EKS
      |
      v
Metrics + Logs + SLOs
```

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

## 87. Final Production Rules

1.  Build once and promote the same immutable artifact.
2.  Keep environment state explicit in Git.
3.  Separate production credentials and AWS permissions.
4.  Protect production GitOps paths.
5.  Validate effective rendered manifests.
6.  Require lower-environment evidence before promotion.
7.  Make database changes backward compatible where possible.
8.  Treat Argo CD health and application health as different signals.
9.  Define rollback triggers before deployment.
10. Preserve a complete deployment audit trail.
11. Make emergency procedures controlled and documented.
12. Test the entire promotion and rollback process regularly.

### Production implementation checks

-   Keep environment boundaries explicit.
-   Promote immutable artifacts rather than rebuilding.
-   Validate the effective desired state before merge.
-   Protect production approvals and credentials.
-   Preserve a deterministic rollback path.

### Operational questions

1.  Which exact artifact is being promoted?
2.  Which environment and cluster will receive it?
3.  What evidence proves the artifact is safe to advance?
4.  What changes are environment-specific?
5.  How will the release be rolled back if health degrades?

---