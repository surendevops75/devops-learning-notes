# 18-Build-and-Package-Management
# 14-Versioning-and-Release-Management


# 01 Versioning Fundamentals

Versioning gives every software release a stable identity. It connects source code, dependencies, CI execution, artifacts, deployments, and rollback.

Production chain:
Git commit -> CI -> version -> build -> test -> security -> immutable artifact -> repository -> promotion -> production.

A version is useful only when it is traceable. For every production release record the version, source commit, build run, dependency state, artifact digest/checksum, approvals, and deployment target.


---



# 02 Semantic Versioning

Semantic Versioning normally uses MAJOR.MINOR.PATCH.

MAJOR: incompatible API or contract changes.
MINOR: backward-compatible functionality.
PATCH: backward-compatible fixes.

Examples:
1.4.2 -> 1.4.3 = patch
1.4.3 -> 1.5.0 = minor
1.5.0 -> 2.0.0 = major

SemVer is a contract, not merely a numbering convention. The team must define what is considered breaking.


---



# 03 Pre-release Versions

Common pre-release forms include:
2.0.0-alpha.1
2.0.0-beta.1
2.0.0-rc.1

Use pre-release identifiers when a build is not yet the final release. Do not accidentally expose release-candidate packages as stable production dependencies.


---



# 04 Maven Versioning

Maven commonly declares a version in pom.xml. Development may use versions such as 1.5.0-SNAPSHOT, while a production release uses 1.5.0.

Snapshots represent evolving development and should not be treated as immutable production artifacts. In multi-module builds, parent and module versions must follow an explicit organization policy.


---



# 05 npm Versioning

Node packages commonly declare version in package.json. npm supports version operations such as npm version patch, npm version minor, and npm version major.

For production, verify the generated version, Git tag, package contents, and target registry before publishing. Never overwrite a consumed release version.


---



# 06 Python Versioning

Modern Python projects commonly define project metadata in pyproject.toml. The version may be static or dynamically derived from Git tags or another authoritative source.

Choose one source of truth. Avoid manually maintaining the same version independently in several files.


---



# 07 Git Tags

A release tag identifies the exact source state.

Example:
git tag -a v1.4.2 -m "Release v1.4.2"
git push origin v1.4.2

Protect release tags where platform controls allow it. Once a production tag is consumed, do not silently move it to another commit.


---



# 08 Branching Strategies

Common strategies include trunk-based development, release branches, and GitFlow-style models.

Trunk-based development favors short-lived branches and frequent integration. Release branches are useful when several supported release lines or long stabilization periods exist. Avoid branch complexity that does not solve a real delivery problem.


---



# 09 Release Pipeline

A strong release pipeline is:

Checkout exact ref
-> determine version
-> dependency restore
-> build
-> unit tests
-> integration tests
-> security
-> package
-> publish
-> promote
-> deploy
-> verify.

Each gate should produce auditable evidence.


---



# 10 Build Once Promote Many

Build the artifact once and promote that exact artifact through environments.

Source -> CI -> artifact v1.4.2 -> DEV -> STAGE -> PROD.

Do not rebuild separately for each environment. Configuration should normally vary by environment while the binary/package/image remains unchanged.


---



# 11 Immutable Artifacts

An immutable release means a published version cannot silently change.

For containers, record the image digest in addition to the tag. For packages, record repository coordinates and checksums where applicable.

Immutability makes rollback and incident investigation reliable.


---



# 12 Artifact Repositories

Use a durable repository such as an enterprise artifact repository for release artifacts. CI caches are optimization layers, not authoritative release storage.

Store:
name
version
checksum/digest
source commit
build ID
provenance
security evidence where required.


---



# 13 Changelog and Release Notes

Release notes should explain what changed and what operators or consumers must do.

Useful categories:
Added
Changed
Fixed
Deprecated
Removed
Security

For breaking changes, include migration steps, compatibility impact, and rollback considerations.


---



# 14 Automated Versioning

Organizations may automate version decisions from commit conventions, pull-request labels, release configuration, or Git tags.

Automation should validate the result before publication. Never allow a malformed or duplicate version to reach the production repository.


---



# 15 Release Candidates

A release candidate is a build believed to be suitable for final release.

Example:
1.5.0-rc.1
-> test
-> security
-> stage
-> approval
-> 1.5.0

If the candidate changes materially, create a new candidate rather than mutating the old one.


---



# 16 Approvals and Controls

High-risk production releases may require engineering, QA, security, change-management, or business approval.

Use protected environments and least-privilege identities. Approval should be recorded as part of the release audit trail.


---



# 17 Deployment Strategies

Common strategies are rolling, blue/green, canary, and feature-flagged releases.

Rolling replaces instances gradually.
Blue/green maintains two environments and switches traffic.
Canary exposes a small percentage of traffic first.
Feature flags separate deployment from feature activation.

Choose according to application risk and operational capability.


---



# 18 Rollback

Rollback should normally redeploy the previously validated immutable artifact.

Example:
v1.5.0 -> failure -> v1.4.2.

Do not rebuild old source merely to roll back. Verify artifact availability, configuration compatibility, database compatibility, and deployment history.


---



# 19 Hotfixes

A production defect should produce a new patch release.

Example:
1.4.2 -> hotfix -> 1.4.3.

Run the required validation and security checks. Never modify 1.4.2 in place.


---



# 20 Database Changes

Application rollback does not automatically roll back database schema changes.

Prefer backward-compatible migrations during rolling deployments:
expand -> deploy compatible application -> migrate/use new schema -> contract.

Define database recovery separately from application rollback.


---



# 21 API Versioning

APIs can be versioned through paths such as /v1 and /v2 or through headers.

Breaking API changes require communication, migration planning, compatibility testing, and a defined deprecation period.


---



# 22 Dependency Releases

A dependency release can affect many consumers.

Example:
common-lib -> auth -> api.

For breaking dependency changes, release compatible library versions first, then update downstream applications. Validate the complete dependency graph.


---



# 23 Monorepo Releases

Monorepos can use fixed versions for all packages or independent package versions.

A safe process is:
detect changes -> calculate affected packages -> resolve dependency graph -> test -> version -> publish.

Never skip affected-package validation merely to reduce release time.


---



# 24 Release Branch Maintenance

When several release lines are supported, maintain explicit branches and patch versions.

Example:
main -> 3.x
release/2.5 -> 2.5.x

Security fixes may be backported and must be validated independently.


---



# 25 Backports

A fix merged to the main development line may need to be backported to an older supported branch.

Validate the older branch independently because dependencies, APIs, and code structure may differ.


---



# 26 Support Policy

Define:
current version
supported versions
security support period
runtime support
upgrade path
end-of-life date.

A support matrix prevents ambiguity during security incidents and release planning.


---



# 27 Feature Flags

Feature flags can reduce release coupling.

Deploy code with the feature disabled, validate the release, then enable it gradually. Flags require ownership, monitoring, and cleanup; permanent unused flags become technical debt.


---



# 28 Canary Releases

Canary releases expose a small traffic percentage to the new version.

Monitor:
error rate
latency
resource usage
business metrics.

Increase traffic only when the canary is healthy.


---



# 29 Blue Green

Blue/green maintains the current and new environments separately.

Traffic moves from blue to green after validation. Keep rollback possible by retaining the previous environment or artifact until the release is considered stable.


---



# 30 GitOps Releases

A common Kubernetes flow is:

Application CI -> image build -> scan -> registry -> immutable digest -> GitOps repository -> Argo CD -> Kubernetes.

The deployment repository records the desired version and creates an auditable deployment history.


---



# 31 Release Security

Protect:
release branches
release tags
publishing credentials
production environments
CI workflow files.

Use least privilege, protected environments, short-lived credentials where supported, and OIDC/federated identity where available.


---



# 32 Provenance

Release provenance connects:

source commit
-> trusted build
-> artifact
-> deployment.

Record builder, workflow run, source revision, dependency state, artifact digest/checksum, and relevant security evidence.


---



# 33 SBOM

An SBOM describes software components included in a release.

Use it to support:
vulnerability response
license review
dependency inventory
incident investigation.

Associate the SBOM with the exact release artifact.


---



# 34 Artifact Signing

Where organizational policy requires it, sign packages, container images, or release metadata.

Verification should happen before trusted deployment. Signing is most useful when the signing identity and key-management process are also protected.


---



# 35 Release Audit

A release audit trail should answer:

Who released it?
What version?
What source commit?
What artifact?
What tests?
What security checks?
Who approved it?
Where was it deployed?
What happened afterward?

Automate evidence collection rather than relying on screenshots.


---



# 36 Release Metrics

Track:
release frequency
lead time
change failure rate
recovery time
deployment success
rollback rate
CI duration.

Use these metrics to improve the system rather than to encourage unsafe release behavior.


---



# 37 Release Cost

Release cost includes CI minutes, runner resources, artifact storage, approvals, deployment infrastructure, and operational recovery.

Optimize cost and delivery speed together. A faster pipeline is not automatically cheaper.


---



# 38 Release Freeze

Organizations may restrict production releases during major events, migrations, or high-risk periods.

Define:
freeze window
exception process
emergency authority
rollback procedure
communication plan.


---



# 39 Emergency Release

Emergency releases still need traceability.

At minimum:
identify incident
create controlled fix
run critical validation
approve according to emergency policy
publish new version
deploy
monitor
document.

Do not use an emergency process as permission to mutate existing artifacts.


---



# 40 Release Failure

If release validation fails:

stop promotion
preserve logs
identify root cause
correct source or pipeline
create a new candidate/version
rerun required validation.

Do not silently replace a failed release artifact.


---



# 41 Version Collision

Two concurrent pipelines can attempt to publish the same version.

Prevent this with:
central version allocation
release locks/concurrency controls
unique tags
repository immutability.

A duplicate should fail clearly rather than overwrite another release.


---



# 42 Wrong Artifact Published

Check:
Git checkout ref
tag
version calculation
workspace cleanliness
artifact path
repository target
credentials.

Then compare the published artifact's digest/checksum and metadata with the intended source commit.


---



# 43 Stage Differs From Production

First compare immutable artifact identifiers.

If stage and production use different artifacts, the promotion model is wrong. Stop and correct artifact promotion before debugging application behavior.


---



# 44 Release Tag Wrong Commit

Do not silently move a production tag.

Determine whether the artifact was published or deployed, preserve evidence, create a corrected release with a new version, and fix the tag/release validation process.


---



# 45 Artifact Retention

Retention must support:
active deployments
rollback
audit
support
compliance.

Do not delete the only recoverable copy of a production artifact.


---



# 46 Disaster Recovery

Critical release recovery includes:
Git/source
artifact repository
container registry
release metadata
deployment configuration
identity/secrets recovery
backup procedures.

Test restoration instead of assuming backups work.


---



# 47 CI Release with GitHub Actions

A tag-triggered workflow can checkout the tagged commit, configure the runtime, build, test, scan, package, and publish.

Use protected environments for production and minimal workflow permissions. Review third-party actions and use controlled versions according to organizational policy.


---



# 48 CI Release with Jenkins

A Jenkins release pipeline commonly performs:
checkout tag
tool setup
build
test
security
package
publish
promote
deploy.

Store credentials in Jenkins credential management rather than the Jenkinsfile.


---



# 49 Artifactory Promotion

Use the enterprise repository as durable artifact storage. Promotion should preserve the same artifact identity.

CI -> candidate artifact -> validation -> release repository/promotion -> deployment.

Do not rebuild simply because the artifact moves to another environment.


---



# 50 Container Releases

For containers use both a human-readable tag and an immutable digest.

Example:
payment-api:1.4.2
payment-api@sha256:...

Kubernetes deployment should prefer immutable identity for production traceability.


---



# 51 Versioned Packages

Maven, npm, and Python packages should follow the repository's version policy.

Before publication verify:
version
package metadata
dependencies
artifact contents
target repository
credentials
immutability rules.


---



# 52 Release Dependencies

Release order matters when packages depend on each other.

Example:
common-lib 2.0.0 -> auth-client 3.0.0 -> application 5.2.0.

Publish the dependency first, validate consumers, and maintain compatibility where rolling upgrades require it.


---



# 53 Release Validation

A release gate should validate:
correct source
correct version
tests
security
artifact contents
provenance
repository target.

Automate these checks before publish.


---



# 54 Reproducibility

A release should be reproducible from controlled inputs:
source
runtime
build tools
dependencies
lockfiles/constraints
repository
build environment.

Caches may accelerate builds but must not be required for correctness.


---



# 55 Release Rollout

A safe rollout is:
publish -> deploy to lower environment -> validate -> approve -> deploy progressively -> monitor -> complete or rollback.

Define success and failure thresholds before starting the rollout.


---



# 56 Release Communication

Communicate:
release version
scope
breaking changes
migration
security fixes
known issues
deployment timing
rollback path.

Keep operational and customer-facing notes appropriate to each audience.


---



# 57 Changelog Automation

Commit conventions and release tooling can generate changelogs.

Automation should still be reviewed for:
breaking changes
security issues
customer impact
migration notes.


---



# 58 Release Branch vs Trunk

Trunk-based development reduces long-lived branch drift and works well with frequent delivery.

Release branches are useful for supported maintenance lines or longer stabilization.

Select based on actual delivery and support requirements.


---



# 59 Release Governance

Governance should protect production without creating unnecessary manual work.

Automate evidence, testing, security checks, versioning, publishing, and deployment where safe. Reserve manual approval for risk-sensitive decisions.


---



# 60 Production Reference Architecture

Reference:

Git
 -> PR validation
 -> CI
 -> version/tag
 -> build
 -> tests
 -> security
 -> immutable artifact
 -> Artifactory/registry
 -> promotion
 -> DEV
 -> STAGE
 -> approval
 -> PROD
 -> monitoring
 -> rollback if required.

For Kubernetes:
artifact/image -> GitOps repository -> Argo CD -> Kubernetes.


---



# 61 Production Checklist

Version:
[ ] policy followed
[ ] unique
[ ] source tag correct
[ ] metadata matches

Build:
[ ] exact commit
[ ] clean build
[ ] tests pass
[ ] security passes

Artifact:
[ ] immutable
[ ] checksum/digest recorded
[ ] provenance recorded
[ ] retained

Deployment:
[ ] approval
[ ] same artifact promoted
[ ] health checks
[ ] monitoring
[ ] rollback ready.


---



# 62 Interview — Versioning

Q: Why is versioning important?

A: It provides a stable identity for releases and allows us to trace an artifact to its source, CI run, dependency state, deployment, and rollback point.


---



# 63 Interview — Build Once

Q: Why build once and promote?

A: It guarantees that the artifact tested in CI is the same artifact deployed to each environment. Environment-specific behavior should normally be configuration, not a different binary.


---



# 64 Interview — Rollback

Q: How do you roll back?

A: I redeploy the previously validated immutable artifact or container digest. I do not normally rebuild old source because rebuilding can produce a different artifact.


---



# 65 Interview — Hotfix

Q: How do you handle a production hotfix?

A: I create a controlled patch change, run the required tests and security checks, publish a new patch version, deploy it, monitor it, and retain the previous version for rollback.


---



# 66 Interview — Release Security

Q: How do you secure releases?

A: Protected branches and tags, least-privilege identities, protected environments, secret isolation, dependency and artifact scanning, provenance, approvals for high-risk changes, and immutable promotion.


---



# 67 Interview — Monorepo

Q: How do you release a monorepo?

A: I model the dependency graph, detect affected packages, calculate versions according to policy, test affected dependencies, publish required packages, and preserve source-to-artifact traceability.


---



# 68 Interview — SemVer

Q: Explain SemVer.

A: MAJOR normally signals incompatible changes, MINOR compatible functionality, and PATCH compatible fixes, provided the project actually follows the compatibility contract.


---



# 69 Interview — Immutable Artifacts

Q: Why immutable artifacts?

A: They ensure a version cannot change after validation. This gives deterministic promotion, reliable rollback, and strong auditability.


---



# 70 Interview — Release Failure

Q: What if production fails immediately after release?

A: I assess impact, compare the production artifact identity with the validated stage artifact, monitor health, and either roll back to the known-good artifact or execute a controlled forward fix.


---



# 71 Senior Scenario — Two Releases

Two teams attempt version 2.0.0 simultaneously.

Response:
Use concurrency controls or centralized version allocation. Only one release may claim the version. The other pipeline should fail safely and select a new version.


---



# 72 Senior Scenario — Artifact Changed

Stage used artifact A but production contains artifact B.

Response:
Stop rollout. Compare digests, repository records, deployment manifests, and promotion logs. Fix the promotion path so production can only consume the validated immutable artifact.


---



# 73 Senior Scenario — Database Migration

A release changes the database schema and then the application fails.

Response:
Do not assume application rollback is sufficient. Use backward-compatible migrations, restore the known-good application artifact, and handle database recovery with a separately tested strategy.


---



# 74 Senior Scenario — Critical CVE

A critical dependency vulnerability is discovered after release.

Response:
Identify affected versions and deployed consumers, determine exposure, create a patched release, test and scan it, deploy the fixed immutable artifact, and document the incident and affected release lineage.


---



# 75 Senior Scenario — Tag Error

A release tag points to the wrong commit.

Response:
Stop publication/deployment, determine whether consumers received the artifact, preserve evidence, create a corrected release with a new version, and add automated tag-to-commit validation.


---



# 76 Senior Scenario — Release Slow

Release takes 30 minutes.

Response:
Measure queue, dependency restore, build, tests, security, packaging, upload, and deployment. Optimize the dominant stage with caching, parallelism, runner sizing, or test improvements without weakening required gates.


---



# 77 Senior Scenario — Many Release Lines

The company supports three major versions.

Response:
Maintain explicit branches and support policy, backport security fixes deliberately, publish new patch versions for each supported line, and keep independent CI validation and rollback paths.


---



# 78 Senior Scenario — Manual Artifacts

Teams manually copy binaries between environments.

Response:
Move artifacts to a durable repository and automate promotion. Production should consume the same immutable artifact that passed validation, with permissions and audit logs controlling promotion.


---



# 79 Senior Scenario — Release Credential Leak

If release credentials leak:
revoke
-> rotate
-> audit
-> inspect published artifacts
-> review CI access
-> rebuild affected releases if necessary.

Use short-lived credentials and federation where supported to reduce exposure.


---



# 80 Golden Rules 1

1. Treat versioning as a production control.
2. Define one versioning policy.
3. Use a stable source of truth.
4. Associate versions with Git commits.
5. Protect release tags.
6. Do not move consumed production tags.
7. Keep versions unique.
8. Understand SemVer.
9. Use pre-release identifiers deliberately.
10. Do not treat Maven snapshots as immutable releases.
11. Do not overwrite published versions.
12. Build once and promote.
13. Keep artifacts immutable.
14. Store artifacts durably.
15. Record checksums/digests.
16. Record CI build IDs.
17. Record dependency state.
18. Preserve provenance.
19. Generate SBOMs when required.
20. Sign artifacts when required.


---



# 81 Golden Rules 2

21. Use protected production environments.
22. Use least-privilege release identities.
23. Protect publishing credentials.
24. Prefer short-lived credentials where supported.
25. Use OIDC where appropriate.
26. Keep PR workflows away from privileged release secrets.
27. Validate version before publishing.
28. Validate tag-to-commit mapping.
29. Validate artifact-to-source mapping.
30. Validate repository target.
31. Inspect package contents.
32. Publish changelogs.
33. Publish release notes.
34. Document breaking changes.
35. Document migration requirements.
36. Document rollback.
37. Monitor every production release.
38. Track deployment success.
39. Track release frequency.
40. Track change failure rate.


---



# 82 Golden Rules 3

41. Track recovery time.
42. Track lead time.
43. Track rollback rate.
44. Retain artifacts required for rollback.
45. Back up critical repositories.
46. Test artifact recovery.
47. Test rollback.
48. Use canary when appropriate.
49. Use blue/green when appropriate.
50. Use rolling deployment when appropriate.
51. Use feature flags carefully.
52. Promote the same image digest.
53. Do not rely on mutable image tags alone.
54. Keep deployment configuration version-controlled.
55. Use GitOps where appropriate.
56. Keep database migrations backward-compatible.
57. Do not assume application rollback reverses database changes.
58. Version breaking APIs deliberately.
59. Provide deprecation periods.
60. Maintain support matrices.


---



# 83 Golden Rules 4

61. Backport fixes deliberately.
62. Use release branches only when useful.
63. Avoid unnecessary branch complexity.
64. Prevent version collisions.
65. Use release concurrency controls.
66. Do not silently replace failed releases.
67. Preserve failed-release evidence.
68. Separate release failure from deployment failure.
69. Verify artifact identity first.
70. Treat artifact immutability violations seriously.
71. Quarantine compromised artifacts where required.
72. Identify consumers of compromised releases.
73. Rotate credentials after suspected compromise.
74. Rebuild from trusted sources after compromise.
75. Keep release automation under source control.
76. Review release automation as production code.
77. Review third-party CI actions.
78. Use minimal workflow permissions.
79. Automate evidence collection.
80. Keep human approval for risk-sensitive changes.


---



# 84 Golden Rules 5

81. Measure release duration.
82. Measure queue time.
83. Optimize bottlenecks rather than random steps.
84. Preserve security gates during optimization.
85. Preserve test gates during optimization.
86. Use clean build environments.
87. Use reproducible dependencies.
88. Treat caches as acceleration, not truth.
89. Test cold-cache release builds.
90. Keep artifact retention aligned with rollback needs.
91. Define emergency release procedures.
92. Define release freeze procedures.
93. Define who can approve emergency releases.
94. Maintain release audit logs.
95. Track source-to-artifact lineage.
96. Track artifact-to-deployment lineage.
97. Validate lower environments before production.
98. Monitor progressive rollouts.
99. Stop unsafe releases quickly.
100. Always validate the exact versioning, build, repository, promotion,
deployment, monitoring, approval, rollback, and audit behavior used by
the production architecture.


---



# 85 Production Release Decision Matrix

| Situation | Preferred approach |
|---|---|
| Small compatible fix | PATCH release |
| Compatible feature | MINOR release |
| Breaking API | MAJOR release |
| Experimental build | Pre-release |
| Development Maven artifact | SNAPSHOT |
| Production artifact | Immutable release |
| Environment promotion | Same artifact |
| Emergency production defect | New patch version |
| Kubernetes production identity | Image digest |
| Multiple supported majors | Release branches/support lines |
| Gradual high-risk rollout | Canary |
| Fast environment switch | Blue/green |
| Kubernetes desired-state deployment | GitOps |

# 86 Release Runbook

1. Confirm the intended source commit.
2. Confirm version policy.
3. Confirm no existing production artifact owns the version.
4. Create or validate the release tag.
5. Start the release workflow.
6. Verify dependency state.
7. Build in a clean environment.
8. Run unit and integration tests.
9. Run security and quality gates.
10. Generate package/image.
11. Inspect artifact contents.
12. Record checksum or digest.
13. Record provenance and SBOM where required.
14. Publish to the approved repository.
15. Verify repository metadata.
16. Promote the same artifact to the first environment.
17. Validate health.
18. Promote progressively.
19. Obtain production approval if required.
20. Deploy.
21. Monitor application and business health.
22. Close the release with audit evidence.
23. Retain the previous artifact for rollback according to policy.

# 87 Release Troubleshooting Quick Reference

Version wrong:
- inspect tag
- inspect project metadata
- inspect CI variables
- inspect version calculation

Duplicate version:
- inspect concurrent releases
- inspect repository
- inspect release lock
- select a new version

Wrong source:
- inspect checkout ref
- verify tag commit
- use a clean workspace

Publish denied:
- inspect repository
- inspect authentication
- inspect authorization
- inspect package policy

Stage differs from production:
- compare artifact digest
- inspect promotion logs
- inspect deployment manifest

Rollback unavailable:
- inspect retention
- inspect repository recovery
- inspect deployment history
- restore artifact from backup if available

Production database issue:
- inspect migration compatibility
- separate application rollback from schema recovery
- use tested database recovery procedure

# 88 Final Senior Perspective

A senior DevOps engineer should be able to explain versioning as a
complete lifecycle rather than as a Git command.

The mature model is:

source identity
-> version identity
-> deterministic build
-> validated artifact
-> durable storage
-> immutable promotion
-> controlled deployment
-> monitoring
-> rollback
-> audit.

The strongest release system minimizes manual copying, minimizes
mutable state, protects credentials, preserves evidence, and makes the
safe path the easiest path.

# END OF 14-Versioning-and-Release-Management.md
