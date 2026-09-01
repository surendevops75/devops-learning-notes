# 11 --- CI Pipeline --- Production DevOps Capstone

> Deep production-oriented notes for AWS, EKS, ECR, GitLab CI, Docker,
> DevSecOps, GitOps, and Argo CD.

## How to use this chapter

This chapter is intentionally designed as an implementation and
interview reference. Each section explains the production reasoning, the
control boundary, operational implications, and the failure modes a
DevOps engineer should be able to troubleshoot.

## 1. Purpose and Scope

A production CI pipeline is the controlled path from reviewed source
code to a verified, immutable artifact. It must answer whether a change
is buildable, tested, secure, traceable, reproducible, and safe to
promote. CI should build and verify; GitOps and Argo CD should own
Kubernetes desired state. The pipeline therefore becomes a supply-chain
control plane rather than a collection of shell commands.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `purpose and scope` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 2. Reference Flow

Developer -\> Merge Request -\> validation -\> tests -\> security -\>
Docker build -\> SBOM -\> image scan -\> quality gate -\> Amazon ECR -\>
immutable digest -\> GitOps repository -\> Argo CD -\> EKS -\> health
verification. Every transition should be observable and auditable. The
same artifact should be promoted through environments rather than
rebuilt for every environment.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `reference flow` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 3. CI/CD Boundary

Continuous Integration validates source and produces artifacts.
Continuous Delivery governs release and promotion. GitOps records
desired deployment state. Argo CD reconciles Git into Kubernetes. A
production design avoids giving every CI job cluster-admin access
because that increases credential exposure and makes the CI system a
direct production control plane.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `ci/cd boundary` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 4. Repository Strategy

Keep application source, infrastructure, and environment desired state
logically separated when that improves ownership and security. An
application repository contains source, tests, Dockerfile, Helm chart,
and CI configuration. A GitOps repository contains environment values or
manifests. This creates an audit chain from source commit to image
digest to deployment commit.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `repository strategy` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 5. Branch Protection

Protect the default branch and production release references. Require
merge requests, successful required jobs, appropriate reviewers, and
protected tags. CI configuration is executable code, so changes to
.gitlab-ci.yml, Dockerfiles, build scripts, and runner configuration
deserve the same review discipline as application code.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `branch protection` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 6. Pipeline Stages

A strong baseline is validate, test, security, build, scan, publish, and
GitOps. Independent jobs should run concurrently. Cheap deterministic
checks should fail early, while expensive operations should start only
when useful prerequisites are complete. The exact ordering can be
optimized using DAG dependencies.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `pipeline stages` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 7. Workflow Rules

Differentiate merge-request pipelines, default-branch pipelines,
release-tag pipelines, scheduled security jobs, and manual recovery
workflows. Prevent duplicate pipelines for the same commit. Production
credentials must never become available merely because arbitrary code is
running in a merge-request context.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `workflow rules` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 8. Validation

Validate source syntax, formatting, YAML, JSON, Helm templates,
Terraform syntax where applicable, and policy requirements. Validation
is intentionally cheap. It should prevent obvious failures before
spending runner time on integration testing or container builds.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `validation` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 9. Linting

Lint application code and configuration consistently. Pin tool versions
or use controlled tool images instead of depending on moving latest
tags. Lint failures are deterministic and normally should not be retried
automatically.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `linting` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 10. Unit Testing

Unit tests should be fast, deterministic, and produce machine-readable
results. Retain JUnit-style reports and coverage where supported. A
failed unit test should block publication unless the organization has an
explicit, temporary exception policy.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `unit testing` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 11. Integration Testing

Integration tests validate interactions with databases, caches, queues,
APIs, and other dependencies. Prefer ephemeral test dependencies or
isolated test environments. Never point normal CI tests at production
data stores. Synthetic test data should be disposable and clearly
separated from customer data.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `integration testing` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 12. Test Pyramid

Use many fast unit tests, fewer integration tests, and a deliberately
limited number of end-to-end tests. Excessive end-to-end testing
increases feedback time and environmental flakiness. The goal is high
defect detection with acceptable pipeline duration.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `test pyramid` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 13. Flaky Tests

A flaky test produces different results without a relevant code change.
Track it explicitly, identify the infrastructure or synchronization
cause, and repair it. Do not permanently convert important tests into
allow-failure jobs simply to make the dashboard green.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `flaky tests` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 14. Security Model

Security is layered across source, dependencies, build inputs, container
artifacts, registry, GitOps, and cluster admission. Typical controls
include SAST, secret detection, dependency scanning, IaC scanning,
container scanning, SBOM generation, artifact signing, and Kubernetes
admission policies.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `security model` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 15. SAST

Static analysis searches source code for insecure patterns such as
injection risks, unsafe command execution, insecure cryptography, and
dangerous data handling. Findings should be machine-readable and mapped
to an explicit severity and exception process.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `sast` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 16. Dependency Security

Applications inherit vulnerabilities from direct and transitive
dependencies. Use lock files and controlled dependency updates. Scan
package manifests and resolved dependency graphs. Remediation should
upgrade to fixed versions where practical and should be tested through
the normal CI path.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `dependency security` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 17. Secret Detection

Scan commits and working trees for credentials, tokens, private keys,
and accidental environment files. Detection is not sufficient: prevent
secrets from entering Git, rotate leaked credentials immediately, and
use managed secret systems or workload identity for runtime access.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `secret detection` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 18. AWS Authentication

Never place long-lived AWS access keys in ordinary CI variables when
OIDC federation is available. The preferred flow is CI identity token
-\> AWS IAM OIDC trust -\> STS AssumeRoleWithWebIdentity -\> short-lived
credentials. Restrict the trust policy by project, branch, tag,
environment, and audience as appropriate.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `aws authentication` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 19. IAM Least Privilege

Separate roles according to responsibility. A validation job generally
needs no AWS credentials. A publish job needs only the ECR actions
required to authenticate and upload images. Production promotion should
use a separate protected trust boundary. Avoid broad administrator roles
in CI.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `iam least privilege` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 20. Runner Architecture

Use isolated runners appropriate to trust level. Untrusted merge-request
code should not share unrestricted privileged infrastructure with
production release jobs. Ephemeral runners reduce workspace
contamination and simplify patching. Protect the runner host, container
runtime, network egress, and cached data.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `runner architecture` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 21. Privileged Runner Risk

Combining untrusted code, privileged containers, host-mounted Docker
sockets, and production credentials creates a severe blast radius.
Separate runner pools, restrict protected variables, and use rootless or
daemonless build approaches where practical.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `privileged runner risk` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 22. Docker Build

Production images should be small, deterministic, non-root, and free of
build secrets. Multi-stage builds separate compilation from runtime.
Copy dependency manifests before application source when that improves
cache reuse. Use a .dockerignore to keep Git metadata, logs, local
dependencies, and secrets out of the build context.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `docker build` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 23. BuildKit

BuildKit provides efficient caching, parallel build operations, and
secure build-secret mechanisms. A build secret should be mounted only
for the command that requires it and should never be copied into an
image layer. Prefer short-lived package credentials and private
dependency mirrors when available.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `buildkit` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 24. Base Images

Use approved, maintained base images. Pin versions and, for
high-assurance builds, consider digest pinning. Base image updates
should be automated or regularly reviewed because an application can
become vulnerable even when its own source has not changed.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `base images` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 25. Image Tagging

Tag images with a unique source identity such as the Git commit SHA.
Avoid using latest as the production deployment identity. Tags are
convenient human references; the immutable digest is the strongest
content identity.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `image tagging` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 26. Image Digests

A digest identifies exact image content. A production GitOps reference
such as repository@sha256:digest prevents accidental movement of a
mutable tag. Capture the digest immediately after publication and record
it alongside source and pipeline metadata.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `image digests` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 27. Amazon ECR

Use a private ECR repository per agreed naming convention. CI
authenticates with temporary AWS credentials, pushes the immutable
artifact, and records the resulting digest. Configure lifecycle policies
carefully so rollback artifacts are not deleted before the operational
retention period ends.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `amazon ecr` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 28. ECR Lifecycle

Images accumulate quickly in active repositories. Retain recent builds,
release artifacts, and enough previous production versions for rollback.
Test lifecycle policies in non-production first. A policy that saves
storage but removes the last known-good image is an operational failure.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `ecr lifecycle` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 29. Container Scanning

Scan the built image before promotion. A practical policy may block
critical findings and selected high-severity exploitable findings while
tracking lower-risk issues. Do not blindly ignore all findings; use
documented, expiring exceptions when a risk is accepted.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `container scanning` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 30. Vulnerability Exceptions

An exception should record the CVE, package, severity, exploitability,
rationale, compensating control, owner, approval, ticket, and
expiration. Exceptions are risk decisions, not permanent scanner
configuration.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `vulnerability exceptions` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 31. SBOM

Generate an SPDX or CycloneDX software bill of materials for important
artifacts. SBOM data allows security teams to identify which deployed
services contain a vulnerable component. Store it with the release
metadata or an approved security inventory system.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `sbom` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 32. Provenance

Record repository, commit, pipeline ID, builder, timestamp, dependency
inputs, base image identity, image digest, and source reference. Strong
provenance allows an incident responder to move from a production
workload back to the exact source and build process.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `provenance` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 33. Quality Gates

A publish gate can require lint, tests, SAST, dependency checks, secret
scanning, container scanning, SBOM generation, and build success. Gates
should be explicit. A green pipeline should mean that required controls
actually ran.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `quality gates` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 34. Build Once

Build once and promote the same artifact through development, QA,
staging, and production. Rebuilding for each environment can produce
different artifacts because dependencies, base images, timestamps, or
build environments change. Environment-specific configuration belongs
outside the image.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `build once` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 35. Environment Configuration

Keep API endpoints, feature flags, resource sizing, and
environment-specific settings outside the container image. Helm values,
Kubernetes configuration, and secret systems should provide environment
differences while the artifact remains identical.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `environment configuration` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 36. GitOps Update

After publication, CI should update the GitOps desired state with the
immutable image digest. Argo CD then reconciles the repository into
Kubernetes. The GitOps commit becomes a durable deployment record.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `gitops update` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 37. Why Not kubectl

Direct kubectl access from general CI jobs couples CI to cluster
credentials and makes the pipeline a high-value production target.
GitOps reduces that blast radius, provides reviewable desired state, and
gives operators a declarative rollback mechanism.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `why not kubectl` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 38. GitOps Protection

Protect production GitOps branches. Require validation, review, and
appropriate code ownership. Avoid force pushes. A GitOps repository is
production control-plane data and should receive strong backup, access
control, and audit treatment.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `gitops protection` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 39. Promotion

Promote an existing digest from dev to QA to staging to production.
Promotion should change desired state, not rebuild the application. This
preserves the relationship between tested artifact and production
artifact.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `promotion` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 40. Production Approval

Use explicit production approval where organizational risk requires it.
The approval should operate on a known artifact and known change rather
than approving an ambiguous mutable tag.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `production approval` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 41. Artifact Metadata

A useful release record contains service name, source commit, pipeline
ID, image repository, image digest, build time, scan status, SBOM
reference, and GitOps commit. This metadata becomes invaluable during
incident response and audits.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `artifact metadata` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 42. Pipeline Caching

Distinguish package-manager caches from Docker layer caches. Cache
deterministic dependency data to reduce network time, but never cache
secrets or untrusted sensitive output. Use lock files as cache
invalidation inputs where possible.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `pipeline caching` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 43. Pipeline Parallelism

Independent checks such as linting, unit tests, SAST, and dependency
scanning can run concurrently. Use DAG-style dependencies so build jobs
start as soon as their true prerequisites finish. Optimize measured
bottlenecks rather than guessing.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `pipeline parallelism` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 44. Retry Policy

Retry transient infrastructure failures such as temporary network or
registry errors. Do not automatically retry deterministic compile
errors, failing unit tests, or policy violations. Excessive retries hide
the true reliability of the pipeline.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `retry policy` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 45. Artifact Retention

Retain release metadata, SBOMs, test evidence, and production artifacts
according to operational and compliance requirements. Transient logs can
have shorter retention. Retention must support incident investigation
and rollback.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `artifact retention` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 46. Supply Chain Threat Model

Model the entire path: developer, source repository, dependency
registry, runner, build tool, base image, container registry, GitOps
repository, controller, and cluster. Threats include malicious
dependencies, poisoned caches, compromised runners, unauthorized
registry pushes, and GitOps tampering.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `supply chain threat model` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 47. Dependency Proxies

Enterprise dependency proxies or internal artifact repositories can
reduce public-registry dependency, improve build performance, provide
caching, and improve governance. They should still validate upstream
packages and preserve provenance.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `dependency proxies` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 48. Monorepo Strategy

For monorepos, path-aware rules can avoid rebuilding unrelated services.
However, shared libraries require dependency awareness. A change to a
common library may require testing multiple consumers.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `monorepo strategy` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 49. Microservice Strategy

Each service can follow the same lifecycle: validate, test, security,
build, scan, publish, and GitOps update. Central CI templates provide
consistency while allowing service-specific test and packaging steps.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `microservice strategy` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 50. CI Templates

Centralized CI templates reduce duplicated security and build logic.
Version templates deliberately. A floating latest template can
unexpectedly change every service; versioned templates make upgrades
controlled and auditable.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `ci templates` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 51. Helm Validation

For Helm-based applications, run helm lint, render templates with the
intended values, and validate the rendered Kubernetes objects. Check API
versions, selectors, probes, security contexts, resource requests and
limits, and required metadata before deployment.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `helm validation` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 52. Terraform Validation

When infrastructure changes are included, run formatting and validation
checks before plan or apply. CI validation should avoid unnecessary
cloud credentials. Production apply permissions should be isolated and
protected.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `terraform validation` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 53. Database Migration

Database schema changes require backward compatibility because old and
new application pods can coexist during a rollout. Prefer
expand-and-contract migrations: add compatible schema, deploy code,
migrate data, and remove obsolete structures later.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `database migration` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 54. Zero Downtime

A pipeline cannot guarantee zero downtime by itself. It must support
applications that have readiness probes, graceful shutdown, compatible
APIs, appropriate rolling-update settings, and connection draining.
Post-deployment verification must confirm actual service health.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `zero downtime` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 55. Rollback

Rollback should point GitOps back to a previously healthy immutable
digest. Do not rebuild an old source version during an incident unless
there is no alternative. Reusing the known-good artifact makes recovery
faster and more deterministic.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `rollback` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 56. Emergency Rollback

Declare the incident, identify the previous healthy digest, revert the
GitOps desired state, allow Argo CD to reconcile, verify application
health, and then investigate root cause. Emergency procedures should
remain auditable.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `emergency rollback` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 57. Canary

Canary delivery can deploy a small percentage of traffic to the new
digest, observe error rate and latency, then progressively increase
traffic. The same immutable artifact is used at every stage.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `canary` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 58. Blue-Green

Blue-green deployment keeps the old and new versions available
simultaneously. Traffic can switch to the new version after validation.
Rollback is a traffic or desired-state change back to the old immutable
artifact.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `blue-green` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 59. Feature Flags

Feature flags separate code deployment from feature activation. This can
reduce release risk because code can be deployed and observed before a
feature is enabled for all users.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `feature flags` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 60. Post-Deployment Verification

A successful CI pipeline is not the same as a healthy production
deployment. Verify Argo CD health, rollout status, pod readiness,
service endpoints, smoke tests, and key application metrics after
reconciliation.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `post-deployment verification` control is explicitly represented
    in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 61. Smoke Testing

Smoke tests should be short, deterministic, and low impact. They can
validate health endpoints and a small synthetic business transaction.
They should detect obvious release failures without becoming a second
full test suite.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `smoke testing` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 62. SLO-Aware Delivery

Advanced delivery systems can pause or roll back a rollout when error
rate, latency, or availability violates defined thresholds. This
connects deployment automation to reliability objectives without
rebuilding artifacts.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `slo-aware delivery` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 63. Scheduled Rebuilds

Security programs often need scheduled rebuilds because base images and
dependencies can become vulnerable without application source changes. A
scheduled pipeline can rebuild, rescan, generate a new digest, and open
a promotion path.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `scheduled rebuilds` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 64. Incident: Docker Failure

Check the Dockerfile, base image availability, dependency downloads,
build context, disk, memory, and network. Use plain build logs for
diagnosis. Do not add privileged permissions as the first
troubleshooting step.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: docker failure` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 65. Incident: ECR Login Failure

Check the AWS caller identity, OIDC token claims, IAM trust policy, role
permissions, account, region, and repository. Confirm that the job
received only the credentials it should have received.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: ecr login failure` control is explicitly represented
    in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 66. Incident: ECR Push Failure

Verify repository existence, upload permissions, authentication, region,
account, and network connectivity. Avoid granting ecr:\* merely to make
a failed push succeed.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: ecr push failure` control is explicitly represented
    in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 67. Incident: ImagePullBackOff

If CI successfully pushed an image but EKS cannot pull it, inspect the
pod events, exact repository and digest, node or workload permissions,
network path, ECR policy, and registry availability.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: imagepullbackoff` control is explicitly represented
    in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 68. Incident: GitOps Failure

If ECR publication succeeds but GitOps update fails, inspect repository
authentication, branch protection, conflicts, generated YAML,
validation, and commit permissions. Never silently deploy a different
artifact as a workaround.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: gitops failure` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 69. Incident: Argo CD Failure

If the GitOps commit exists but the application is unhealthy, inspect
Argo CD application status, rendered manifests, sync state, Kubernetes
events, resource health, and image pull status.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: argo cd failure` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 70. Incident: Secret Leak

Immediately revoke or rotate the exposed credential, inspect relevant
audit logs, remove the secret from source and history as required,
replace the mechanism with OIDC or managed secret storage, and
investigate whether the credential was used.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: secret leak` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 71. Incident: Runner Disk Full

Inspect filesystem usage, container image storage, workspace data, and
caches. Long-term controls include ephemeral runners, cleanup,
right-sized disks, bounded caches, and concurrency management.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: runner disk full` control is explicitly represented
    in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 72. Incident: Runner OOM

Determine whether memory pressure comes from Docker builds, tests,
parallel jobs, or runner sizing. Measure before increasing instance
size. Reducing concurrency can be safer than indefinitely increasing
resources.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: runner oom` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 73. Incident: Network Failure

Differentiate application defects from registry, package mirror, DNS,
proxy, certificate, and network outages. Private mirrors and dependency
proxies can improve resilience for critical pipelines.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `incident: network failure` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 74. Private Runner Network

Sensitive runners can run in private subnets with controlled egress and
VPC endpoints for AWS services where appropriate. Avoid unnecessary
public inbound access. Network design should support the minimum
required destinations.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `private runner network` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 75. Egress Control

Restrict sensitive runner egress to approved Git, AWS, registry, and
dependency destinations when organizational policy requires it.
Controlled egress reduces the chance that malicious build code can
exfiltrate credentials or source.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `egress control` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 76. Fork Security

Treat forked or untrusted merge-request code as hostile. Do not expose
protected production variables, deployment roles, or privileged runners
to those pipelines.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `fork security` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 77. Safe Debugging

Avoid env, printenv, set -x, and verbose commands that may expose
secrets. Print safe identifiers such as commit SHA, AWS account ID,
region, and repository name instead.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `safe debugging` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 78. Auditability

For every production release, be able to answer who, what, when, why,
which source commit, which pipeline, which image digest, which GitOps
commit, which approver, and which cluster received the artifact.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `auditability` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 79. DORA Metrics

Track deployment frequency, lead time for changes, change failure rate,
and time to restore service. CI duration, queue time, retry rate, and
test failure rate provide additional engineering feedback.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `dora metrics` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 80. Pipeline Cost

Runner minutes, compute size, registry storage, cache storage, and
network transfer contribute to cost. Optimize cache hit rate,
concurrency, image size, and unnecessary pipeline duplication while
preserving required controls.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `pipeline cost` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 81. Release Freeze

A deployment freeze should stop production promotion without necessarily
stopping validation, testing, scanning, and artifact creation. Teams can
continue producing verified artifacts while production changes are
temporarily paused.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `release freeze` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 82. Hotfix

A production hotfix should use a focused branch or controlled workflow,
required tests and security checks, immutable artifact creation,
explicit emergency approval where necessary, and the normal GitOps
reconciliation path whenever practical.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `hotfix` control is explicitly represented in the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 83. Separation of Duties

A mature chain distinguishes developer, CI system, release approver,
GitOps repository, Argo CD controller, and cluster operator
responsibilities. No single static token should control every layer.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `separation of duties` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 84. Admission Control

Cluster policy can require approved registries, signed images, non-root
containers, resource limits, restricted capabilities, and other
controls. CI is one security boundary; Kubernetes admission is another.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `admission control` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 85. Image Signing

A mature supply chain can sign image digests and verify signatures at
deployment time. The key management and trust-root design must be
treated as production security infrastructure.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `image signing` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 86. Production Readiness

A production CI pipeline is ready when required controls are automated,
credentials are short-lived, artifacts are immutable, security findings
have policy, rollback is tested, GitOps is protected, runners are
isolated, and deployment lineage is easy to prove.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `production readiness` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 87. Master Architecture

The target architecture is Developer -\> GitLab -\> isolated runners -\>
validation/tests/security -\> BuildKit -\> SBOM and scan -\> ECR -\>
immutable digest -\> protected GitOps repository -\> Argo CD -\> EKS -\>
progressive rollout and health verification. This architecture minimizes
privilege, maximizes traceability, and makes recovery deterministic.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `master architecture` control is explicitly represented in the
    pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## 88. Senior Interview Answer

A strong senior answer is: I use CI to validate source, run tests and
DevSecOps checks, build an immutable image, generate SBOM and
provenance, scan it, and publish it to ECR using short-lived OIDC
credentials. I capture the image digest and update GitOps rather than
deploying directly with kubectl. Argo CD reconciles the desired state
into EKS. The same digest is promoted across environments, and rollback
means reverting to a known-good digest. Runner isolation, protected
branches, least privilege, admission controls, audit trails, and
measurable pipeline reliability complete the production design.

### Production checks

-   Define ownership and the failure boundary.
-   Keep credentials scoped to the smallest possible job.
-   Preserve immutable identifiers and audit metadata.
-   Fail safely rather than bypassing a control.
-   Measure the change before optimizing it.

### What to verify

1.  The `senior interview answer` control is explicitly represented in
    the pipeline.
2.  Failure produces a visible, actionable job result.
3.  Logs contain enough safe diagnostic information.
4.  The process can be reproduced or audited from source to artifact.
5.  Recovery does not require rebuilding an unknown artifact.

## Final End-to-End Production Pipeline

``` text
Developer
   |
   v
Merge Request
   |
   +--> lint / validation
   +--> unit tests
   +--> integration tests
   +--> SAST
   +--> secret detection
   +--> dependency scan
   |
   v
Protected Main
   |
   v
Docker / BuildKit
   |
   +--> immutable commit tag
   +--> OCI metadata
   +--> SBOM
   |
   v
Container Scan
   |
   v
Quality Gate
   |
   v
Amazon ECR
   |
   +--> image digest
   |
   v
GitOps Repository
   |
   +--> protected desired state
   |
   v
Argo CD
   |
   v
AWS EKS
   |
   +--> rollout health
   +--> smoke test
   +--> metrics
   |
   v
Production
```

## Final Production Checklist

-   [ ] Protected branches and release tags
-   [ ] Required merge-request checks
-   [ ] Isolated or ephemeral runners
-   [ ] No long-lived AWS keys in CI
-   [ ] OIDC and STS short-lived credentials
-   [ ] Least-privilege IAM
-   [ ] Lint and validation
-   [ ] Unit and integration tests
-   [ ] SAST and dependency scanning
-   [ ] Secret detection
-   [ ] Container scanning
-   [ ] SBOM generation
-   [ ] Immutable image tag
-   [ ] Digest captured
-   [ ] ECR lifecycle policy
-   [ ] GitOps repository protection
-   [ ] Argo CD reconciliation
-   [ ] Post-deployment smoke test
-   [ ] Rollback to known-good digest
-   [ ] Artifact and release metadata
-   [ ] Audit trail
-   [ ] Failure runbooks
-   [ ] Disaster recovery for the delivery system

## Capstone Integration

The CI pipeline is the bridge between the earlier infrastructure and
application-packaging chapters and the later DevSecOps, GitOps, Argo CD,
multi-environment, observability, disaster-recovery, troubleshooting,
and interview chapters. The key production boundary remains:

``` text
CI = build + test + verify + publish
GitOps = desired state
Argo CD = reconciliation
EKS = runtime
ECR = immutable artifact registry
```

## Final Principle

**A production CI pipeline should make it difficult to deploy an unknown
artifact, easy to prove what was deployed, and fast to recover to a
known-good state.**
