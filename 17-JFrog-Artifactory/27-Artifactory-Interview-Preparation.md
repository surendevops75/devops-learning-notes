# 17-JFrog-Artifactory
# 27-Artifactory-Interview-Preparation

## 1. Purpose

This is the final interview-preparation file for the JFrog Artifactory
section.

It is designed for a DevOps / DevSecOps engineer targeting large
production environments and covers:

- Artifactory fundamentals
- architecture
- repositories
- Maven
- NPM
- PyPI
- Docker
- Helm
- authentication
- RBAC
- versioning
- lifecycle
- Build Info
- Jenkins
- GitHub Actions
- GitLab
- Kubernetes
- EKS
- ECR comparison
- security
- HA
- backup
- DR
- production architecture
- troubleshooting
- Projects
- supply-chain security
- incident response
- scenario-based interviews
- architecture interviews
- troubleshooting interviews
- behavioral questions
- senior-level answers
- rapid-fire questions
- production checklists

The answers are written to demonstrate production ownership rather
than memorized definitions.

---

# PART I — HOW TO ANSWER ARTIFACTORY INTERVIEWS

## 2. Answer Structure

For most technical questions use:

```text
Definition
   |
Architecture
   |
Production implementation
   |
Security
   |
Failure handling
   |
Monitoring
```

---

## 3. Avoid Weak Answers

Weak:

```text
Artifactory is an artifact repository.
```

Strong:

```text
Artifactory is an enterprise binary repository and artifact
management platform that I use to centralize package dependencies,
store build outputs, control promotion, integrate with CI/CD and
Kubernetes, enforce access policies, provide traceability through
build metadata, and support secure production release workflows.
```

---

# PART II — FUNDAMENTALS

## 4. What Is JFrog Artifactory?

Answer:

```text
JFrog Artifactory is a universal artifact repository platform used
to store, manage, resolve, secure and distribute software packages
and build artifacts across the software delivery lifecycle.

It supports multiple ecosystems such as Docker/OCI, Maven, npm, PyPI,
Helm and generic artifacts, and integrates with CI/CD and Kubernetes.
```

---

## 5. Why Use Artifactory?

Answer:

```text
I use Artifactory to centralize dependencies and build outputs,
control access to artifacts, cache external dependencies, provide
reliable package distribution, trace artifacts back to builds, and
create a controlled promotion path from development to production.
```

---

## 6. What Problem Does It Solve?

Answer:

```text
Without an artifact repository, teams often depend directly on
external package sources and distribute binaries through inconsistent
mechanisms.

Artifactory provides a controlled internal source for dependencies and
release artifacts, improving reliability, security and traceability.
```

---

# PART III — ARCHITECTURE

## 7. Explain Production Artifactory Architecture

Answer:

```text
I would place Artifactory behind a highly available load-balancing
layer and use multiple application nodes where the selected JFrog
architecture supports HA.

The application layer would use highly available supporting services
such as the database and artifact storage. Internal clients would
normally access the service through stable private DNS and TLS.

CI/CD would use scoped service identities, Kubernetes would use
read-only registry access, and the platform would have monitoring,
centralized logging, backup and tested DR.
```

---

## 8. What Are the Main Components?

Answer:

```text
The important architectural components are:

- clients
- DNS
- load balancer/reverse proxy
- Artifactory application nodes
- database
- filestore/object storage
- authentication/identity
- CI/CD integrations
- Kubernetes consumers
- monitoring
- backup and DR
```

---

## 9. Is Artifactory Stateless?

Answer:

```text
The application layer can be deployed as multiple nodes, but the
overall platform is not simply stateless because metadata and binary
artifacts depend on persistent database and storage services.

Therefore HA must include the supporting data services.
```

---

# PART IV — REPOSITORIES

## 10. What Are the Repository Types?

Answer:

```text
Local repositories store artifacts published by the organization.

Remote repositories represent external sources and can cache
dependencies.

Virtual repositories provide a single controlled endpoint over
multiple local and remote repositories.
```

---

## 11. Local vs Remote vs Virtual

Answer:

```text
Local:
internal artifact publication.

Remote:
controlled access and caching of external dependencies.

Virtual:
a unified consumption endpoint that can aggregate repositories.
```

---

## 12. Why Use Virtual Repositories?

Answer:

```text
They give developers and CI a stable endpoint while allowing the
platform team to control which local and external repositories are
available behind that endpoint.
```

---

## 13. What Happens If an External Repository Goes Down?

Answer:

```text
If the required dependency is already cached, the remote repository
may continue serving the cached artifact depending on the repository
and request behavior.

For critical production dependencies I also prefer controlled,
repeatable dependency management rather than relying on live external
availability.
```

---

# PART V — ARTIFACT VERSIONING

## 14. Why Is Versioning Important?

Answer:

```text
Versioning provides immutable identification of build outputs and
allows us to promote a known artifact rather than rebuilding it for
each environment.
```

---

## 15. Why Avoid latest?

Answer:

```text
latest is mutable from a release-management perspective and makes
rollback and traceability harder.

For production I prefer immutable versions and, for containers,
tracking the image digest.
```

---

## 16. Build Once, Promote Many

Answer:

```text
I build the artifact once, test and scan it, publish it to
Artifactory, and then promote the same artifact through environments.

That ensures the production artifact is exactly the artifact that
passed validation.
```

---

# PART VI — BUILD INFO

## 17. What Is Build Info?

Answer:

```text
Build Info provides metadata about a build, including the build
identity, artifacts, dependencies and other provenance information
supported by the integration.

It helps trace a production artifact back to the CI build and source
revision.
```

---

## 18. Why Is Build Info Important?

Answer:

```text
During an incident I can start with a production artifact or image
and trace it to the build, dependencies and source revision. That
greatly improves release auditing and root-cause analysis.
```

---

# PART VII — MAVEN

## 19. How Does Maven Integrate with Artifactory?

Answer:

```text
Maven can consume dependencies through an Artifactory virtual or
remote repository and publish organization-owned artifacts to a
local repository.

CI credentials are scoped to the repositories and operations required.
```

---

## 20. Maven 401

Answer:

```text
I check the configured credentials, token validity, identity and
client configuration.
```

---

## 21. Maven 403

Answer:

```text
I check the user's or service identity's repository permissions,
path permissions and requested operation.
```

---

## 22. Maven 404

Answer:

```text
I verify groupId, artifactId, version, repository URL and whether the
artifact actually exists or is available through the configured
virtual repository.
```

---

# PART VIII — NPM

## 23. NPM Artifactory Integration

Answer:

```text
I configure npm to use the appropriate Artifactory registry and
separate package installation permissions from package publication
permissions.

For scoped packages I verify the scope-to-registry configuration and
token permissions.
```

---

## 24. NPM Security

Answer:

```text
I avoid distributing broad credentials and use scoped identities,
protected CI secrets and controlled remote repositories for external
dependencies.
```

---

# PART IX — PYPI

## 25. How Do You Configure PyPI?

Answer:

```text
pip and related tooling can use an Artifactory PyPI virtual or local
endpoint. I validate the index URL, TLS trust, authentication,
repository access and package version.
```

---

# PART X — DOCKER

## 26. How Does Docker Use Artifactory?

Answer:

```text
Artifactory can act as an enterprise container registry. CI builds
and scans an image, publishes it to the appropriate repository, and
Kubernetes pulls the approved image through the registry endpoint.
```

---

## 27. Docker Pull vs Push Permissions

Answer:

```text
Kubernetes runtime identities normally require READ access.

CI publishing identities require DEPLOY access.

DELETE and administrative permissions should be separately controlled.
```

---

## 28. Why Track Digests?

Answer:

```text
A tag can point to different content over time. A digest identifies
the exact image content, so production deployment and rollback become
more deterministic.
```

---

# PART XI — HELM

## 29. Artifactory and Helm

Answer:

```text
Artifactory can store Helm packages and, depending on the supported
version and configuration, Helm/OCI artifacts.

CI publishes the chart, GitOps references the approved artifact, and
Argo CD or another deployment mechanism reconciles it into Kubernetes.
```

---

# PART XII — AUTHENTICATION

## 30. Authentication vs Authorization

Answer:

```text
Authentication answers who are you?

Authorization answers what are you allowed to do?
```

---

## 31. How Do You Authenticate Humans?

Answer:

```text
For enterprise environments I prefer centralized identity such as
SSO with MFA, while maintaining tightly controlled break-glass
administrative access.
```

---

## 32. How Do You Authenticate CI?

Answer:

```text
I use dedicated service identities and scoped tokens or supported
federated authentication mechanisms rather than personal credentials.
```

---

## 33. How Do You Rotate Tokens?

Answer:

```text
I create the replacement credential, update the consumer securely,
validate the workflow, and revoke the old credential.

For production systems I prefer a rotation process that avoids
unnecessary downtime.
```

---

# PART XIII — RBAC

## 34. What Is Least Privilege?

Answer:

```text
Every identity receives only the permissions required for its role
and nothing broader.

For example, Kubernetes only needs READ for image pulls, while CI
needs READ and DEPLOY for its target repository.
```

---

## 35. Why Avoid Admin Credentials in CI?

Answer:

```text
If the CI credential is compromised, administrator permissions could
allow attackers to delete repositories, modify configuration or
access unrelated projects.

Scoped identities significantly reduce blast radius.
```

---

# PART XIV — SECURITY

## 36. How Do You Secure Artifactory?

Answer:

```text
I use SSO/MFA for humans, scoped service identities, least-privilege
repository permissions, TLS, private networking where appropriate,
artifact and dependency scanning, immutable releases, audit logging,
credential rotation and protected backups.
```

---

## 37. Supply-Chain Security

Answer:

```text
I control external dependency sources, scan dependencies and
artifacts, maintain provenance through Build Info, generate or
consume SBOM information where required, sign artifacts when required
and prevent untrusted artifacts from reaching production.
```

---

## 38. What If a Malicious Package Is Found?

Answer:

```text
I stop promotion, identify the artifact and affected consumers,
quarantine or block it according to policy, inspect audit and
provenance information, rotate credentials if compromise is suspected,
and replace the affected artifact with a known-good version.
```

---

# PART XV — HIGH AVAILABILITY

## 39. How Do You Make Artifactory HA?

Answer:

```text
I use multiple Artifactory nodes behind a highly available load
balancer and distribute them across failure domains.

I also make sure critical dependencies such as the database and
artifact storage do not introduce a single point of failure.
```

---

## 40. What Happens When One Node Fails?

Answer:

```text
The load balancer detects the unhealthy node and removes it from
traffic. Remaining nodes continue serving requests if they have
sufficient capacity and the shared dependencies remain healthy.
```

---

## 41. Why Is Two Nodes Not the Whole HA Story?

Answer:

```text
Two application nodes do not remove a single point of failure in a
database, storage system, load balancer, DNS path or network.

HA must be evaluated end to end.
```

---

# PART XVI — BACKUP AND DR

## 42. What Do You Back Up?

Answer:

```text
I protect the database, artifact filestore and required Artifactory
configuration and security data according to the supported JFrog
backup architecture.
```

---

## 43. Backup vs DR

Answer:

```text
Backup provides historical recovery points.

DR provides the architecture and procedures for restoring service
after a major failure.

HA, backup and DR solve different problems and should be designed
together.
```

---

## 44. What Are RTO and RPO?

Answer:

```text
RTO is the target time to restore service.

RPO is the target maximum acceptable data loss measured in time.
```

---

## 45. How Do You Test Backups?

Answer:

```text
I periodically restore into an isolated environment, validate the
database and artifact data, start Artifactory, test repository access,
download representative artifacts and validate important CI and
Kubernetes workflows.
```

---

# PART XVII — PRODUCTION ARCHITECTURE

## 46. Give a Complete Architecture

Answer:

```text
Developers / CI / Kubernetes
            |
            v
       Private DNS
            |
            v
     Load Balancer / WAF
            |
      +-----+-----+
      |     |     |
      v     v     v
    Art-A Art-B Art-C
      |     |     |
      +-----+-----+
            |
      +-----+------+
      |            |
      v            v
   DB HA       Artifact Storage
      |            |
      +-----+------+
            |
         Backup
            |
            v
          DR

CI uses scoped publish identities.
Kubernetes uses read-only runtime access.
Monitoring, logging, security and audit cover the complete platform.
```

---

# PART XVIII — AWS / EKS

## 47. Artifactory and AWS

Answer:

```text
In AWS I would place Artifactory in an appropriate VPC architecture,
use private connectivity for internal consumers where possible,
distribute workloads across availability zones, use highly available
database and storage services, and integrate with AWS monitoring,
identity and backup controls.
```

---

## 48. Artifactory and EKS

Answer:

```text
EKS workloads access the Artifactory registry through a private or
controlled network path.

The runtime identity has only READ access to the required repository.
I validate DNS, TLS, security groups, routing and image pull
credentials and test image pulls during scaling scenarios.
```

---

# PART XIX — ECR VS ARTIFACTORY

## 49. When Would You Use ECR?

Answer:

```text
ECR is a strong choice when the organization primarily needs a native
AWS container registry tightly integrated with AWS workloads.

Artifactory becomes particularly valuable when the organization needs
a broader enterprise artifact platform across multiple package
ecosystems and external dependency management.
```

---

## 50. ECR vs Artifactory

Answer:

```text
ECR:
AWS-focused container registry.

Artifactory:
multi-ecosystem enterprise artifact platform.

The right decision depends on organizational architecture rather than
simply choosing the product with the largest feature list.
```

---

# PART XX — PROJECTS

## 51. What Are Artifactory Projects?

Answer:

```text
Projects provide an organizational and governance boundary for
grouping repositories, teams, permissions and related resources
around a product, application or business unit.
```

---

## 52. Why Use Projects?

Answer:

```text
Projects help large organizations separate ownership, scope access,
reduce credential blast radius and provide a cleaner governance model
when many teams share an Artifactory platform.
```

---

## 53. Cross-Project Access

Answer:

```text
I grant the consumer project only the required READ access to the
producer repository and avoid broad platform-level permissions.
```

---

# PART XXI — TROUBLESHOOTING

## 54. How Do You Troubleshoot 403?

Answer:

```text
I confirm the identity and operation, then inspect group membership,
permission targets, repository scope and path permissions. I compare
the failing identity with a known-good identity and review recent
permission changes.
```

---

## 55. How Do You Troubleshoot 401?

Answer:

```text
I check credentials, token validity, expiration, identity-provider
status, client configuration and clock-related issues.
```

---

## 56. How Do You Troubleshoot 404?

Answer:

```text
I verify the repository, artifact name, version, path and virtual or
remote repository configuration. I also confirm that the artifact was
actually published.
```

---

## 57. How Do You Troubleshoot 503?

Answer:

```text
I start with the load balancer and backend health, then inspect
Artifactory nodes and shared dependencies such as database, storage
and network. I correlate the failure time with logs and metrics.
```

---

## 58. How Do You Troubleshoot Slow Artifactory?

Answer:

```text
I break the latency into network, load balancer, Artifactory,
database, storage and upstream dependency layers. I check request
rate, CPU, memory, database latency, storage latency and artifact
size before deciding whether scaling is actually necessary.
```

---

# PART XXII — CI/CD SCENARIOS

## 59. Jenkins Cannot Upload

Answer:

```text
I verify the registry endpoint, TLS, Jenkins credential, service
identity, repository, DEPLOY permission, artifact size and storage.
Then I determine whether the problem affects one job or all jobs.
```

---

## 60. GitHub Actions Cannot Publish

Answer:

```text
I verify the configured secret or federated identity, runner network,
repository permissions, protected environment rules and Artifactory
DEPLOY permission.
```

---

## 61. GitLab Runner Cannot Download

Answer:

```text
I check the runner's network connectivity, certificate trust,
credential, repository READ permission and package URL.
```

---

# PART XXIII — KUBERNETES SCENARIOS

## 62. ImagePullBackOff

Answer:

```text
I run kubectl describe pod and classify the underlying error.

401:
authentication.

403:
authorization.

404:
repository/image/path.

TLS:
certificate trust.

timeout:
network or backend availability.

Then I test the registry endpoint from an appropriate environment.
```

---

## 63. Kubernetes Can Pull but CI Cannot Push

Answer:

```text
That can be expected because runtime and CI should have different
permissions.

Kubernetes needs READ.

CI needs DEPLOY.

I would investigate CI credentials and permissions rather than
granting Kubernetes additional access.
```

---

# PART XXIV — PERFORMANCE

## 64. Artifactory Is Slow During Deployment

Answer:

```text
I first determine whether the bottleneck is caused by an image pull
burst, network bandwidth, storage latency, database pressure or
Artifactory resource saturation.

I then scale or optimize the actual bottleneck and verify the result
with metrics.
```

---

## 65. Storage Is Full

Answer:

```text
I protect the service, identify the growth source, check retention
and large artifacts, expand capacity where necessary and remove only
artifacts that are safe and approved for deletion.
```

---

# PART XXV — INCIDENT RESPONSE

## 66. Production Artifactory Is Down

Answer:

```text
I first establish impact and scope.

Then I check:

DNS
load balancer
Artifactory nodes
database
storage
network
TLS

I restore service safely, preserve evidence, identify root cause and
document preventive actions.
```

---

## 67. Region Failure

Answer:

```text
I follow the documented DR runbook: declare the incident, validate
the DR data and dependencies, activate the recovery environment,
switch DNS or routing, validate artifact access, test CI and
Kubernetes workloads, and monitor the recovered service.
```

---

# PART XXVI — SECURITY INCIDENTS

## 68. CI Token Leaked

Answer:

```text
I revoke the token immediately, identify its permissions and usage,
review audit events for unauthorized activity, create a replacement
credential, update CI securely and validate the pipeline.
```

---

## 69. Unauthorized Artifact Uploaded

Answer:

```text
I identify the publishing identity, revoke or disable it if
necessary, inspect audit logs and artifact provenance, quarantine the
artifact, determine whether it was consumed and rotate credentials
if compromise is confirmed.
```

---

# PART XXVII — ARCHITECTURE QUESTIONS

## 70. How Would You Design for 1,000 Developers?

Answer:

```text
I would separate teams logically using Projects, use group-based
RBAC, standardize repository naming, centralize approved remote
dependencies, provide virtual repositories, use dedicated CI
identities, distribute Artifactory across failure domains and
implement strong monitoring and capacity planning.

I would also design for burst traffic from CI and Kubernetes rather
than sizing only for average traffic.
```

---

## 71. How Would You Design Multi-Region Artifactory?

Answer:

```text
I would start with RTO/RPO and determine whether the requirement is
active/passive or another supported topology.

I would protect the database and artifact storage using supported
replication and backup mechanisms, maintain a recovery environment,
secure cross-region connectivity and test DNS failover and complete
application recovery.
```

---

## 72. How Would You Reduce Blast Radius?

Answer:

```text
I use project boundaries, repository-scoped permissions, separate
service identities, least-privilege tokens, separate environments
and restricted administrative access.

A compromised project identity should not automatically control the
entire Artifactory platform.
```

---

# PART XXVIII — REAL-WORLD DESIGN SCENARIO

## 73. Scenario

Requirement:

```text
500 developers
100 CI pipelines
20 Kubernetes clusters
multiple package ecosystems
production must be highly available
```

Answer:

```text
I would establish a centralized Artifactory platform with Projects
for major ownership boundaries.

I would use local repositories for internal releases, remote
repositories for approved external dependencies and virtual
repositories for standardized consumption.

CI would use project-specific identities and publish immutable
artifacts. Kubernetes would use read-only access.

The Artifactory layer would be HA behind a load balancer, with
highly available database and storage. The platform would have
centralized monitoring, logging, security scanning, backup and DR.

I would then load-test CI and Kubernetes pull bursts and establish
capacity thresholds before production rollout.
```

---

# PART XXIX — BEHAVIORAL QUESTIONS

## 74. Tell Me About a Difficult Artifactory Incident

Strong structure:

```text
Situation
Task
Action
Result
```

Example:

```text
We had a production deployment failure because Kubernetes could not
pull an image.

I first established that the issue was limited to one cluster and
checked the Pod events. The error was a registry authorization
failure. I compared the runtime identity with a known-good cluster,
found a permission change, restored the intended READ access, and
validated image pulls.

Afterward I added an access-change review and a pre-production
registry connectivity test.
```

---

## 75. Tell Me About a Security Improvement

Answer:

```text
I replaced broad CI credentials with project-scoped service
identities, separated runtime READ access from CI DEPLOY access,
restricted DELETE operations and added credential rotation and audit
review.

The result was a significantly smaller credential blast radius and
better traceability.
```

---

## 76. Tell Me About a Production Failure

Answer:

```text
I focus on the investigation process rather than blaming a
component.

I explain the impact, how I narrowed the scope, the evidence that
identified the failure, the mitigation, the root cause and the
preventive action.
```

---

# PART XXX — SENIOR-LEVEL QUESTIONS

## 77. What Makes a Senior Artifactory Engineer Different?

Answer:

```text
A senior engineer does not only configure repositories.

They understand architecture, availability, security, dependency
management, CI/CD, Kubernetes, capacity, disaster recovery,
governance and operational risk.

They can design the platform, troubleshoot failures and explain the
business impact of technical decisions.
```

---

## 78. What Would You Monitor?

Answer:

```text
I monitor request rate, latency, errors, CPU, memory, storage,
database health, network usage, authentication failures, repository
growth and backup health.

I also create alerts around user-visible symptoms such as high 5xx
rates and artifact pull failures.
```

---

## 79. How Do You Plan Capacity?

Answer:

```text
I measure current artifact storage, daily growth, request rate,
network bandwidth, CI concurrency and Kubernetes pull bursts.

Then I forecast growth and maintain failure and maintenance headroom
rather than sizing only for average utilization.
```

---

# PART XXXI — RAPID-FIRE QUESTIONS

## 80. Local Repository?

```text
Stores internally published artifacts.
```

## 81. Remote Repository?

```text
Represents and can cache an external dependency source.
```

## 82. Virtual Repository?

```text
Unified endpoint over multiple repositories.
```

## 83. 401?

```text
Authentication failure.
```

## 84. 403?

```text
Authorization failure.
```

## 85. 404?

```text
Resource/path not found.
```

## 86. Build Once?

```text
Create one artifact and promote it.
```

## 87. Build Info?

```text
Build provenance and metadata.
```

## 88. HA?

```text
Multiple application nodes and highly available dependencies.
```

## 89. RTO?

```text
Target recovery time.
```

## 90. RPO?

```text
Target maximum acceptable data loss.
```

## 91. Kubernetes Permission?

```text
Normally READ for image pulls.
```

## 92. CI Permission?

```text
READ plus DEPLOY for publishing.
```

## 93. Why Digest?

```text
Exact immutable image identity.
```

## 94. Why Projects?

```text
Ownership, governance and scoped access.
```

---

# PART XXXII — PRODUCTION CHECKLIST

## 95. Architecture

```text
[ ] HA
[ ] load balancer
[ ] database HA
[ ] storage resilience
[ ] DNS
[ ] TLS
[ ] failure domains
```

---

## 96. Repositories

```text
[ ] local
[ ] remote
[ ] virtual
[ ] naming
[ ] lifecycle
[ ] promotion
```

---

## 97. Security

```text
[ ] SSO/MFA
[ ] RBAC
[ ] least privilege
[ ] scoped service identities
[ ] token rotation
[ ] scanning
[ ] audit
[ ] signing where required
```

---

## 98. CI/CD

```text
[ ] Jenkins
[ ] GitHub Actions
[ ] GitLab
[ ] Build Info
[ ] secure secrets
[ ] artifact promotion
```

---

## 99. Kubernetes

```text
[ ] Docker/OCI
[ ] READ access
[ ] image pull authentication
[ ] TLS trust
[ ] EKS connectivity
[ ] digest tracking
```

---

## 100. Operations

```text
[ ] monitoring
[ ] logs
[ ] alerts
[ ] capacity
[ ] backup
[ ] restore testing
[ ] DR testing
[ ] runbooks
```

---

# PART XXXIII — FINAL INTERVIEW GOLDEN RULES

## 101. Rules

```text
1. Explain Artifactory as a platform, not just a repository.

2. Always connect repository design to CI/CD and production usage.

3. Use Build Once, Promote Many.

4. Prefer immutable versions.

5. Track container digests.

6. Use virtual repositories for controlled consumption.

7. Separate local, remote and virtual responsibilities.

8. Use least privilege.

9. Never use admin credentials for routine CI.

10. Separate CI DEPLOY access from Kubernetes READ access.

11. Use Projects to establish ownership and governance.

12. Reduce credential blast radius.

13. Design HA end to end.

14. Do not ignore database and storage dependencies.

15. Start troubleshooting with scope and evidence.

16. 401 usually means authentication.

17. 403 usually means authorization.

18. 404 requires repository/artifact/path investigation.

19. 5xx requires platform/dependency investigation.

20. Protect both metadata and binary artifacts.

21. HA does not replace backup.

22. Backup does not replace DR.

23. Test restores.

24. Test DR.

25. Monitor user-visible symptoms.

26. Preserve evidence during security incidents.

27. Rotate compromised credentials immediately.

28. Keep production architecture documented.

29. Know the difference between ECR and a universal artifact platform.

30. Explain decisions in terms of security, reliability, scalability,
    cost and operational risk.

31. Do not claim a feature exists without verifying the deployed
    JFrog edition and version.

32. In senior interviews, explain trade-offs rather than presenting
    one architecture as universally correct.
```

---

# END OF 27-Artifactory-Interview-Preparation.md

# END OF 17-JFrog-Artifactory SECTION
