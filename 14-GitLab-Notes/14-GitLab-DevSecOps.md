# 13-GitLab — 14 GitLab DevSecOps

> Production-oriented guide to integrating security into GitLab CI/CD, covering SAST, SCA, secret detection, container scanning, IaC security, DAST, SonarQube, Trivy, Veracode, dependency management, image security, Kubernetes/EKS security, Terraform security, security gates, vulnerability triage, remediation, compliance, SBOM, provenance, production incident response, and senior DevSecOps interview scenarios.

---

## 1. What Is DevSecOps?

DevSecOps integrates security throughout the software delivery lifecycle.

Traditional:

```text
Develop
 ↓
Build
 ↓
Deploy
 ↓
Security review
```

DevSecOps:

```text
Plan
 ↓
Code
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Deploy
 ↓
Monitor
```

---

## 2. Why DevSecOps?

Security should not be a final deployment gate only.

Early security checks:

- find vulnerabilities sooner
- reduce remediation cost
- improve developer feedback
- standardize controls
- reduce production risk

---

## 3. GitLab DevSecOps Lifecycle

```text
Code
 ↓
Commit
 ↓
SAST
 ↓
SCA
 ↓
Secrets
 ↓
Build
 ↓
Container Scan
 ↓
IaC Scan
 ↓
DAST
 ↓
Approval
 ↓
Deploy
```

The exact sequence depends on the application and environment.

---

## 4. Shift Left

Shift-left security means detecting security problems earlier.

Example:

```text
Developer writes code
 ↓
CI security scan
 ↓
Vulnerability detected
 ↓
Fix before production
```

---

## 5. Shift Everywhere

Security should also continue after deployment:

```text
CI Security
+
Runtime Security
+
Monitoring
+
Incident Response
```

Shift-left does not mean shift-security-only-to-CI.

---

## 6. Security as Code

Security rules can be version controlled.

Examples:

```text
IaC policies
CI rules
container policies
Kubernetes manifests
dependency policies
```

---

## 7. DevSecOps Pipeline

A practical pipeline:

```text
Validate
 ↓
Unit Tests
 ↓
SAST
 ↓
SCA
 ↓
Secret Detection
 ↓
Build
 ↓
Container Scan
 ↓
Push Registry
 ↓
IaC Scan
 ↓
DAST
 ↓
GitOps Promotion
```

---

## 8. Security Tool Categories

| Area | Example |
|---|---|
| SAST | SonarQube |
| SCA | Dependency scanners |
| Container | Trivy |
| IaC | Trivy / Checkov |
| DAST | DAST tooling |
| Application security | Veracode |
| Secrets | GitLab Secret Detection / dedicated scanners |
| SBOM | Syft / scanner ecosystem |
| Image registry | ECR |
| Deployment | ArgoCD |

---

## 9. SAST

Static Application Security Testing analyzes source code without executing the application.

Typical findings:

```text
SQL injection
unsafe API usage
hard-coded credentials
insecure patterns
```

---

## 10. SCA

Software Composition Analysis examines third-party dependencies.

Example:

```text
Python package
Java dependency
Node.js package
```

It identifies known vulnerabilities and dependency risks.

---

## 11. Secret Detection

Secret scanning searches for credentials such as:

```text
AWS keys
API tokens
private keys
passwords
```

Secrets should never be committed.

---

## 12. Container Scanning

Container scanning checks:

```text
OS packages
language packages
vulnerable libraries
misconfigurations
```

Trivy is commonly used for this purpose.

---

## 13. IaC Security

Infrastructure code can introduce vulnerabilities.

Examples:

```text
public S3 bucket
0.0.0.0/0 security group
unencrypted RDS
wildcard IAM
public EKS endpoint
```

---

## 14. DAST

Dynamic Application Security Testing evaluates a running application.

Concept:

```text
Deploy test environment
 ↓
DAST
 ↓
HTTP requests
 ↓
Application behavior
```

---

## 15. SonarQube in DevSecOps

SonarQube can provide:

```text
code quality
bugs
security findings
code smells
security hotspots
```

Integrate it early in CI.

---

## 16. SonarQube Pipeline Position

Typical:

```text
Checkout
 ↓
Build/Test
 ↓
SonarQube
 ↓
Security Gate
```

The exact placement depends on the build system.

---

## 17. SonarQube Quality Gate

A quality gate can fail the pipeline when defined conditions are not met.

Examples:

```text
new vulnerabilities
coverage threshold
duplicated code
new bugs
```

---

## 18. Quality Gate vs Security Gate

Quality gate:

```text
code quality
```

Security gate:

```text
security risk
```

They can overlap but are not identical.

---

## 19. SonarQube for Maven

Typical flow:

```text
mvn test
 ↓
Sonar analysis
 ↓
Quality gate
```

Use the project's supported Sonar scanner/plugin approach.

---

## 20. SonarQube for Node.js

Node applications can run:

```text
npm test
Sonar analysis
```

Ensure dependency installation and test results are available to the analysis.

---

## 21. SonarQube for Python

Typical:

```text
pytest
 ↓
coverage
 ↓
Sonar analysis
```

Use the appropriate scanner and project configuration.

---

## 22. SonarQube Project Configuration

Common concepts:

```text
project key
source paths
test paths
coverage report
exclusions
quality gate
```

Avoid broad exclusions that hide real code.

---

## 23. SonarQube Exclusions

Exclusions may be appropriate for:

```text
generated code
vendor code
build artifacts
```

Do not exclude security-sensitive application directories merely to reduce findings.

---

## 24. SonarQube Token Security

Scanner authentication should use:

```text
masked CI variable
protected variable
```

Never commit the token to Git.

---

## 25. SonarQube Failure Handling

When a quality gate fails:

```text
Read finding
 ↓
Identify new issue
 ↓
Fix source
 ↓
Re-run CI
```

Do not disable the gate without understanding the risk.

---

## 26. Trivy Overview

Trivy can scan multiple artifact types.

Examples:

```text
container image
filesystem
repository
IaC
Kubernetes configuration
```

---

## 27. Trivy Image Scan

Example:

```bash
trivy image "$IMAGE"
```

Typical output includes:

```text
package
installed version
fixed version
severity
vulnerability ID
```

---

## 28. Trivy Severity

Common levels:

```text
UNKNOWN
LOW
MEDIUM
HIGH
CRITICAL
```

Organizations define which levels block promotion.

---

## 29. Vulnerability Threshold

Example policy:

```text
CRITICAL → block
HIGH → review/block depending on SLA
MEDIUM → track
LOW → track
```

Do not copy a threshold blindly; base it on organizational risk.

---

## 30. Trivy Exit Code

A CI job can fail when vulnerabilities meet a threshold.

Concept:

```bash
trivy image --severity CRITICAL,HIGH --exit-code 1 "$IMAGE"
```

Tune flags for the installed Trivy version and project policy.

---

## 31. Scan Before Push

Possible:

```text
Build
 ↓
Trivy
 ↓
Push ECR
```

This prevents known-bad images from entering the registry.

---

## 32. Scan After Push

Another pattern:

```text
Build
 ↓
Push ECR
 ↓
Scan
 ↓
Promote
```

This can integrate with registry-side controls.

---

## 33. Defense in Depth

Use multiple controls:

```text
CI scan
+
Registry scan
+
Admission control
+
Runtime monitoring
```

No single scanner detects every risk.

---

## 34. Base Image Security

Use trusted minimal base images.

Examples:

```text
distroless
slim
approved enterprise base
```

Smaller images generally reduce attack surface, though they do not automatically make an image secure.

---

## 35. Base Image Updates

A base image may contain security fixes.

Process:

```text
New base image
 ↓
Build
 ↓
Scan
 ↓
Test
 ↓
Promote
```

---

## 36. Pin Base Images

Prefer immutable references where practical:

```text
image@sha256:...
```

This improves reproducibility.

---

## 37. Dockerfile Security

Avoid:

```dockerfile
USER root
```

when the application does not require root.

Prefer a non-root runtime user.

---

## 38. Dockerfile Secrets

Never:

```dockerfile
ENV AWS_SECRET_ACCESS_KEY=...
```

or copy secret files into an image.

---

## 39. Multi-Stage Builds

Example:

```text
Builder image
 ↓
compile
 ↓
Runtime image
```

This removes unnecessary build dependencies from the runtime image.

---

## 40. Minimize Runtime Packages

A production image should contain only what the application requires.

Benefits:

```text
smaller image
fewer vulnerabilities
smaller attack surface
faster deployment
```

---

## 41. Container User

Verify:

```bash
docker run IMAGE id
```

The application should normally run as a non-root user.

---

## 42. Container Capabilities

Avoid unnecessary Linux capabilities.

Use a restricted runtime configuration.

---

## 43. Read-Only Root Filesystem

Where compatible:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

This reduces the ability of compromised processes to modify the container filesystem.

---

## 44. Kubernetes Security Context

Example:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

Combine with appropriate UID/capability configuration.

---

## 45. Privileged Containers

Avoid:

```yaml
privileged: true
```

unless there is a documented platform requirement.

Privileged containers significantly increase risk.

---

## 46. Host Network

Avoid:

```yaml
hostNetwork: true
```

unless explicitly required.

---

## 47. Host PID/IPC

Avoid:

```yaml
hostPID: true
hostIPC: true
```

unless required.

---

## 48. Kubernetes Admission Security

Production EKS can enforce policies through:

```text
Pod Security Admission
OPA Gatekeeper
Kyverno
other approved policy engines
```

---

## 49. GitOps Security Policy

Policy can block:

```text
privileged pods
hostPath
hostNetwork
missing resource limits
untrusted images
```

---

## 50. Policy as Code

Example:

```text
GitLab CI
 ↓
Manifest/IaC scan
 ↓
Policy
 ↓
Pass/Fail
 ↓
GitOps
```

---

## 51. Trivy IaC Scan

Concept:

```bash
trivy config .
```

Use it to identify infrastructure/configuration security issues.

---

## 52. Terraform Security Scan

Scan Terraform for:

```text
IAM wildcard
public resources
unencrypted storage
open ports
```

---

## 53. Terraform + Trivy

Possible pipeline:

```text
terraform fmt
 ↓
terraform validate
 ↓
trivy config
 ↓
terraform plan
```

---

## 54. Checkov

Checkov can evaluate IaC policies.

Example:

```text
Terraform
CloudFormation
Kubernetes
```

Use the organization's approved scanner.

---

## 55. Scanner Overlap

Multiple scanners may report the same underlying issue.

Example:

```text
Trivy
Checkov
SonarQube
```

may each detect related problems.

Centralize triage.

---

## 56. False Positives

A security scanner can report an issue that is not applicable.

Process:

```text
Finding
 ↓
Validate
 ↓
Determine risk
 ↓
Document exception if justified
```

Never mark findings ignored without a reason.

---

## 57. Vulnerability Triage

Prioritize by:

```text
severity
exploitability
exposure
asset criticality
availability of fix
business impact
```

---

## 58. CVE

CVE identifies a publicly catalogued vulnerability.

Example concept:

```text
CVE-YYYY-NNNN
```

The identifier alone does not determine your actual risk.

---

## 59. CVSS

CVSS provides a standardized severity score.

Use CVSS as one input, not the only risk signal.

---

## 60. Exploitability

A lower-severity vulnerability on a public internet-facing service may deserve more urgency than a higher score on an isolated internal system.

Context matters.

---

## 61. Vulnerability Remediation

Typical workflow:

```text
Detect
 ↓
Triage
 ↓
Identify fix
 ↓
Update dependency/base image
 ↓
Test
 ↓
Scan
 ↓
Deploy
```

---

## 62. Dependency Upgrade

For Python:

```bash
pip list --outdated
```

For Node:

```bash
npm outdated
```

For Maven:

```text
dependency analysis tooling
```

Use controlled dependency updates.

---

## 63. Dependency Pinning

Pin dependencies to known compatible versions.

This improves:

```text
reproducibility
security control
upgrade predictability
```

---

## 64. Lock Files

Examples:

```text
package-lock.json
poetry.lock
requirements lock strategy
Maven dependency management
```

Commit the appropriate lock file.

---

## 65. Dependency Confusion

Dependency confusion can occur when an attacker publishes a malicious package with a name that a build system resolves unexpectedly.

Controls:

```text
private registries
dependency pinning
trusted sources
package policies
```

---

## 66. Typosquatting

Attackers may publish packages with names similar to legitimate dependencies.

Review dependency source and spelling carefully.

---

## 67. Malicious Dependencies

A package may execute code during:

```text
install
build
runtime
```

Use trusted package sources and scanning.

---

## 68. JFrog Artifactory

A private artifact repository can provide controlled dependency distribution.

Potential flow:

```text
Developer
 ↓
Artifactory
 ↓
Build
```

This can reduce dependency supply-chain exposure.

---

## 69. Artifact Provenance

Track:

```text
source commit
builder
pipeline
artifact
scanner result
deployment
```

This supports software supply-chain investigations.

---

## 70. SBOM

Software Bill of Materials lists software components.

Example:

```text
Application
 ├── Python
 ├── requests
 ├── OpenSSL
 └── OS packages
```

---

## 71. SBOM Generation

Possible tools include:

```text
Syft
Trivy ecosystem
language/package tooling
```

Generate SBOMs during CI where required.

---

## 72. SBOM Storage

Store SBOMs as:

```text
CI artifacts
artifact repository
registry metadata
security platform
```

Protect access where they reveal sensitive software inventory.

---

## 73. SBOM Use During Incident

If a critical CVE appears:

```text
CVE
 ↓
SBOM
 ↓
Identify affected images
 ↓
Identify environments
 ↓
Patch
```

This can dramatically speed response.

---

## 74. Image Provenance

Provenance answers:

```text
Where did this image come from?
Which commit built it?
Which pipeline produced it?
```

---

## 75. Image Tagging Strategy

Example:

```text
user:4f9c2e1
```

Better than:

```text
user:latest
```

for production traceability.

---

## 76. Digest Promotion

Best-practice concept:

```text
ECR
 └── user@sha256:ABC

Dev
 └── SHA256:ABC

Stage
 └── SHA256:ABC

Prod
 └── SHA256:ABC
```

Same artifact across environments.

---

## 77. Registry Security

ECR controls should include:

```text
private repositories
IAM access
image scanning
lifecycle policies
encryption
repository policies
```

---

## 78. ECR Lifecycle Policies

Remove obsolete images according to retention requirements.

Do not delete images still required for rollback.

---

## 79. Image Retention

Keep enough historical images to support:

```text
rollback
incident analysis
compliance
```

Balance retention against storage cost.

---

## 80. ECR Repository Policy

Restrict:

```text
pull
push
delete
```

permissions.

Application workloads normally need pull access, not repository administration.

---

## 81. EKS Image Pull

EKS workloads need appropriate access to pull from ECR.

Configure the node/pod identity mechanism according to the EKS architecture.

---

## 82. EKS Pod Identity

EKS Pod Identity can provide AWS permissions to Kubernetes workloads without distributing static AWS credentials.

Use least privilege.

---

## 83. IRSA

IAM Roles for Service Accounts can associate Kubernetes service accounts with AWS IAM roles.

Both IRSA and EKS Pod Identity are valid patterns depending on cluster/version/design.

---

## 84. CI IAM vs Pod IAM

Separate:

```text
GitLab CI IAM
→ infrastructure/deployment automation

Pod IAM
→ application runtime permissions
```

Do not reuse one powerful role for both.

---

## 85. AWS Credentials in Pods

Never bake:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

into application images.

Use workload identity.

---

## 86. Kubernetes Secrets

Kubernetes Secret objects are not automatically equivalent to a secure external secret-management system.

Control:

```text
RBAC
encryption at rest
access
rotation
```

---

## 87. AWS Secrets Manager

Preferred production pattern where applicable:

```text
AWS Secrets Manager
 ↓
External Secrets
 ↓
Kubernetes Secret
 ↓
Pod
```

---

## 88. Secret Rotation

Design rotation:

```text
Generate
 ↓
Store
 ↓
Update consumer
 ↓
Validate
 ↓
Revoke old
```

Applications must support rotation without unexpected downtime.

---

## 89. Secret Scanning in Git

Scan:

```text
commits
branches
merge requests
```

for credentials.

---

## 90. Secret Found in Git

If a real credential is committed:

```text
1. Revoke/rotate immediately
2. Investigate exposure
3. Remove from source
4. Scan history if required
5. Fix the process
```

Deleting the line alone does not make the secret safe.

---

## 91. Git History

A secret committed in Git may remain in historical commits.

Treat it as compromised.

---

## 92. Secret Prevention

Use:

```text
secret scanner
pre-commit checks
CI scanning
developer education
external secret manager
```

---

## 93. API Security Testing

DAST can test:

```text
authentication
authorization
input validation
HTTP behavior
common web vulnerabilities
```

Run against controlled environments.

---

## 94. DAST Environment

Prefer:

```text
ephemeral/test environment
```

rather than aggressive scanning directly against production.

---

## 95. DAST Authentication

Authenticated DAST may require test credentials.

Store them securely and use dedicated test accounts.

---

## 96. Production DAST

If production scanning is required:

```text
approved scope
rate limits
maintenance window
monitoring
emergency stop
```

must be established.

---

## 97. API Rate Limits

Security scanners can generate large request volumes.

Protect:

```text
application
database
external dependencies
```

with controlled scan rates.

---

## 98. WAF

AWS WAF can provide protection for internet-facing applications.

Example:

```text
Internet
 ↓
ALB
 ↓
AWS WAF
 ↓
EKS
```

Exact architecture depends on requirements.

---

## 99. WAF vs DAST

WAF:

```text
runtime traffic protection
```

DAST:

```text
security testing
```

They complement each other.

---

## 100. Network Security

EKS security should include:

```text
security groups
network policies
private subnets
controlled ingress
controlled egress
```

---

## 101. NetworkPolicy

Example intent:

```text
Frontend
 ↓
API
 ↓
Database
```

Only required traffic should be allowed.

---

## 102. Least Privilege Networking

Do not allow every namespace to communicate with every other namespace by default when workload isolation is required.

---

## 103. Kubernetes RBAC

Control:

```text
who can get
who can list
who can create
who can delete
```

resources.

---

## 104. Service Account Security

Use dedicated service accounts.

Avoid giving applications permissions through the default service account.

---

## 105. Automount Service Account Token

Disable automatic mounting when an application does not need Kubernetes API access:

```yaml
automountServiceAccountToken: false
```

---

## 106. Pod Security

Apply:

```text
non-root
no privilege escalation
restricted capabilities
read-only filesystem
seccomp
```

where compatible.

---

## 107. Seccomp

Use an appropriate seccomp profile, commonly:

```text
RuntimeDefault
```

where supported and compatible.

---

## 108. Linux Capabilities

Drop unnecessary capabilities:

```yaml
capabilities:
  drop:
    - ALL
```

Add only required capabilities.

---

## 109. HostPath Risk

Avoid:

```yaml
hostPath:
```

unless explicitly required.

Host filesystem access can expose sensitive node resources.

---

## 110. Container Escape Risk

Reduce escape risk with:

```text
non-root
restricted capabilities
seccomp
no privileged mode
no host namespaces
minimal images
```

---

## 111. Network Egress

Control outbound traffic where practical.

Examples:

```text
EKS
 ↓
NAT
 ↓
approved external services
```

---

## 112. Dependency Egress

Applications should not automatically have unrestricted outbound access when security requirements call for egress controls.

---

## 113. TLS

Use TLS for:

```text
external traffic
service integrations
sensitive internal communication
```

according to architecture.

---

## 114. Certificate Management

Use automated certificate management where possible.

Track:

```text
expiry
issuer
renewal
deployment
```

---

## 115. GitLab Runner Security

Runners execute pipeline code.

A compromised pipeline can potentially access:

```text
CI variables
OIDC credentials
artifacts
cloud resources
```

Protect production runners.

---

## 116. Shared Runner Risk

Do not assume a shared runner is appropriate for privileged production infrastructure operations.

Use protected/trusted runners where required.

---

## 117. Runner Isolation

Use isolation mechanisms such as:

```text
ephemeral runners
container isolation
dedicated production runners
network restrictions
```

based on threat model.

---

## 118. CI Variable Protection

Use GitLab controls such as:

```text
masked
protected
environment scoped
```

for sensitive variables.

---

## 119. Environment-Scoped Variables

Example:

```text
AWS_ROLE_ARN
```

can differ by:

```text
dev
stage
prod
```

This prevents accidental use of production credentials in development jobs.

---

## 120. OIDC Claims

AWS trust policy can restrict role assumption based on token claims such as:

```text
project
branch/ref
environment
```

Use the exact claims supported by the chosen GitLab/AWS setup.

---

## 121. GitLab OIDC Security Model

```text
GitLab Job
 ↓
OIDC token
 ↓
AWS STS
 ↓
IAM trust policy
 ↓
Temporary credentials
 ↓
AWS API
```

Temporary credentials reduce long-lived secret exposure.

---

## 122. No Static AWS Keys

Avoid storing:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

for production deployment if OIDC is available and appropriate.

---

## 123. OIDC Failure Troubleshooting

Check:

```text
token audience
issuer
trust relationship
claims
role ARN
AWS account
job identity
```

---

## 124. Security Exceptions

Sometimes a vulnerability cannot immediately be fixed.

Exception record should include:

```text
finding
reason
risk
owner
compensating controls
expiry date
remediation plan
```

---

## 125. Never Permanent Exceptions

Avoid:

```text
ignore forever
```

Security exceptions should expire and be reviewed.

---

## 126. Vulnerability SLA

Define remediation targets based on:

```text
severity
exposure
asset criticality
exploitability
```

Example:

```text
Critical → fastest response
High → urgent
Medium → planned
Low → backlog
```

Actual timelines should follow organizational policy.

---

## 127. Security Dashboard

Track:

```text
open vulnerabilities
critical findings
mean time to remediate
dependency risk
image risk
IaC findings
secret incidents
```

---

## 128. Mean Time to Remediate

MTTR for vulnerabilities measures:

```text
finding detected
 ↓
finding fixed
```

Use it to evaluate security process effectiveness.

---

## 129. Security Debt

Security debt accumulates when findings remain unresolved.

Track:

```text
age
severity
owner
application
environment
```

---

## 130. Vulnerability Ownership

Every significant finding should map to an owner:

```text
application team
platform team
security team
```

Unowned findings tend to remain unresolved.

---

## 131. Security Review in MR

MR reviewers should consider:

```text
new dependencies
new ports
new IAM
new secrets
new containers
new public endpoints
```

---

## 132. GitLab MR Security

A merge request should expose enough information for reviewers to understand:

```text
what changed
security findings
IaC changes
container changes
```

without leaking sensitive values.

---

## 133. Security Pipeline Failure

When security fails:

```text
Read finding
 ↓
Determine true positive
 ↓
Fix or approved exception
 ↓
Re-run pipeline
```

Do not simply bypass the job.

---

## 134. Allow Failure

Avoid using:

```yaml
allow_failure: true
```

for mandatory production security gates unless there is a deliberate policy.

---

## 135. Security Gate Ordering

Example:

```text
SAST
 ↓
SCA
 ↓
Secrets
 ↓
Build
 ↓
Container
 ↓
IaC
 ↓
DAST
```

Some checks can run in parallel to reduce pipeline duration.

---

## 136. Parallel Security Jobs

Example:

```text
             ┌── SAST
Build ───────┼── SCA
             ├── Secrets
             └── IaC
```

Then:

```text
all pass
 ↓
package/promote
```

---

## 137. Security Pipeline Optimization

Use:

```text
parallel scans
cache
incremental analysis
changes-based rules
approved reusable templates
```

Do not skip critical checks merely for speed.

---

## 138. Reusable GitLab Templates

Central security templates can standardize:

```text
SAST
secret scanning
container scanning
IaC scanning
```

Across repositories.

---

## 139. Template Governance

Central templates should be:

```text
versioned
reviewed
tested
documented
```

Avoid unexpected breaking changes to every application pipeline.

---

## 140. Security Template Pinning

Pin reusable CI templates to approved versions where practical.

This improves pipeline reproducibility.

---

## 141. CI Template Supply Chain

A compromised shared CI template can affect many projects.

Protect:

```text
template repository
release process
permissions
```

---

## 142. GitLab Group-Level Security

At organization/group level, standardize:

```text
security jobs
variables
runners
approval rules
policies
```

where appropriate.

---

## 143. Compliance Pipelines

Regulated environments may require:

```text
mandatory scans
approval
change tracking
artifact retention
audit
```

---

## 144. Separation of Duties

A strong production process can separate:

```text
Developer
 ↓
Code reviewer
 ↓
Security reviewer
 ↓
Production approver
```

based on risk.

---

## 145. Audit Trail

Maintain evidence of:

```text
commit
pipeline
security result
approval
artifact
deployment
```

---

## 146. Security Evidence

Useful evidence includes:

```text
pipeline logs
scan reports
MR approvals
Git history
deployment history
SBOM
```

Protect retention and access.

---

## 147. Artifact Retention

Retain security artifacts according to:

```text
incident response
compliance
audit
```

requirements.

---

## 148. Log Integrity

Security logs should not be casually editable by application developers.

Use centralized retention and appropriate access controls.

---

## 149. Security Monitoring

After deployment monitor:

```text
authentication failures
HTTP errors
suspicious requests
container restarts
privilege changes
AWS security events
```

---

## 150. DevSecOps Is Continuous

```text
Build
 ↓
Secure
 ↓
Deploy
 ↓
Monitor
 ↓
Learn
 ↓
Improve
```

Security is not finished after the pipeline passes.

---

## 151. Incident: Leaked AWS Key

Response:

```text
Disable/rotate key
 ↓
Identify affected systems
 ↓
Review CloudTrail
 ↓
Search Git history
 ↓
Remove secret
 ↓
Improve secret controls
```

Never wait for the next deployment.

---

## 152. Incident: Malicious Dependency

Response:

```text
Identify package
 ↓
Stop promotion
 ↓
Identify affected artifacts
 ↓
Remove dependency
 ↓
Rebuild
 ↓
Rescan
 ↓
Deploy clean artifact
```

---

## 153. Incident: Critical Container CVE

Response:

```text
Identify affected image
 ↓
SBOM/query registry
 ↓
Identify environments
 ↓
Rebuild with patched base/package
 ↓
Scan
 ↓
Promote digest
 ↓
Deploy
```

---

## 154. Incident: Critical IaC Finding

Response:

```text
Block production apply
 ↓
Assess exposure
 ↓
Fix Terraform
 ↓
Run scan
 ↓
Plan
 ↓
Review
 ↓
Apply
```

---

## 155. Incident: Vulnerable Production Dependency

Do not only patch the source branch.

Identify:

```text
running image
deployment digest
environment
```

Then patch and redeploy the actual affected artifact.

---

## 156. Incident: Secret Committed to GitOps

Response:

```text
Rotate secret
 ↓
Remove plaintext value
 ↓
Check Git history/exposure
 ↓
Use external secret reference
 ↓
Redeploy
```

---

## 157. Incident: Compromised GitLab Runner

Treat the runner as potentially untrusted.

Actions may include:

```text
isolate runner
revoke temporary access
review jobs
rotate exposed credentials
review GitLab activity
review AWS activity
```

---

## 158. Incident: Compromised GitOps Repository

Protect:

```text
Git history
ArgoCD access
production branches
deploy credentials
```

Review recent changes and restore known-good desired state.

---

## 159. Security and Rollback

A rollback should restore:

```text
known-good application
known-good configuration
known-good image
```

Do not rollback to an artifact that is known to contain a critical vulnerability unless there is an emergency risk tradeoff.

---

## 160. Emergency Security Tradeoff

Sometimes availability and security conflict.

Document:

```text
decision
risk
approver
temporary mitigation
follow-up
```

---

## 161. Threat Modeling

Before major architecture changes identify:

```text
assets
actors
entry points
trust boundaries
attack paths
controls
```

---

## 162. Threat Model for GitOps

Assets:

```text
Git repository
ECR
AWS IAM
EKS
ArgoCD
secrets
```

Attack paths:

```text
developer account
CI runner
GitOps repository
container
Kubernetes API
```

---

## 163. GitLab Threat

Possible risks:

```text
compromised developer
malicious MR
CI script injection
runner compromise
token theft
```

Controls:

```text
MFA/SSO
protected branches
MR approvals
protected variables
trusted runners
OIDC
```

---

## 164. ArgoCD Threat

Possible risks:

```text
repository credential theft
excessive RBAC
malicious Git change
cluster credential compromise
```

Controls:

```text
RBAC
Projects
protected Git
least privilege
SSO
```

---

## 165. Container Threat

Possible risks:

```text
vulnerable package
malicious image
root container
privileged workload
```

Controls:

```text
scan
sign
non-root
admission policy
minimal image
```

---

## 166. Kubernetes Threat

Possible risks:

```text
RBAC abuse
secret exposure
network lateral movement
container escape
```

Controls:

```text
RBAC
NetworkPolicy
Pod Security
secret management
runtime monitoring
```

---

## 167. AWS Threat

Possible risks:

```text
excessive IAM
public S3
public RDS
open security groups
stolen CI identity
```

Controls:

```text
least privilege
private networking
encryption
OIDC
policy
audit
```

---

## 168. Defense in Depth

```text
Developer security
       ↓
GitLab controls
       ↓
CI security
       ↓
Artifact security
       ↓
GitOps security
       ↓
Kubernetes security
       ↓
AWS security
       ↓
Runtime monitoring
```

---

## 169. Security Architecture Principle

No single control should be trusted to prevent every failure.

Use layered controls.

---

## 170. Least Privilege

Apply least privilege to:

```text
developers
GitLab runners
CI roles
ArgoCD
Kubernetes service accounts
AWS roles
repositories
```

---

## 171. Zero Trust Concept

Do not assume:

```text
inside network = trusted
CI job = trusted
developer = trusted
container = trusted
```

Verify identity and authorization continuously.

---

## 172. Production Security Boundaries

Useful boundaries:

```text
GitLab
 ↓
AWS STS
 ↓
Terraform

GitLab
 ↓
GitOps repo
 ↓
ArgoCD
 ↓
EKS
```

Each boundary should have explicit trust controls.

---

## 173. DevSecOps Reference Architecture

```text
                    GitLab
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
       Source                       CI
                                    │
               ┌────────────────────┼───────────────────┐
               ▼                    ▼                   ▼
             SAST                  SCA              Secrets
               │                    │                   │
               └────────────────────┼───────────────────┘
                                    ▼
                                  Build
                                    │
                                    ▼
                              Container Scan
                                    │
                                    ▼
                                   ECR
                                    │
                              Image Digest
                                    │
                                    ▼
                              GitOps Repo
                                    │
                                    ▼
                                  ArgoCD
                                    │
                                    ▼
                                   EKS
                         ┌──────────┼──────────┐
                         ▼          ▼          ▼
                      Network     Pods       Services
                         │          │
                         └──────────┼──────────┘
                                    ▼
                              Runtime Security
                                    │
                         ┌──────────┼──────────┐
                         ▼          ▼          ▼
                    Prometheus   Grafana      ELK
```

---

## 174. DevSecOps Production Pipeline

```text
1. Checkout
2. Validate
3. Unit tests
4. SAST
5. SCA
6. Secret detection
7. Build image
8. Container scan
9. IaC scan
10. SBOM
11. Push ECR
12. Image provenance/signing
13. GitOps update
14. ArgoCD sync
15. Smoke test
16. Monitor
```

---

## 175. Security Gate Matrix

| Control | Stage | Typical Action |
|---|---|---|
| SAST | CI | Block/Review |
| SCA | CI | Block/Review |
| Secret scan | CI | Block |
| Container scan | CI | Block/Review |
| IaC scan | CI | Block/Review |
| DAST | Test | Block/Review |
| Image policy | Deploy | Block |
| Kubernetes policy | Deploy | Block |
| Runtime monitoring | Production | Alert |

---

## 176. Security Risk-Based Gating

Not every finding must necessarily stop every pipeline.

Consider:

```text
severity
exploitability
exposure
asset criticality
compensating controls
```

Security gates should be explicit and documented.

---

## 177. Security Exception Workflow

```text
Finding
 ↓
Risk assessment
 ↓
Owner
 ↓
Temporary exception
 ↓
Compensating control
 ↓
Expiry
 ↓
Remediation
```

---

## 178. DevSecOps Metrics

Track:

```text
critical vulnerabilities
MTTR
security gate failures
secret leaks
dependency age
image vulnerability count
IaC findings
deployment security incidents
```

---

## 179. Security KPI

A useful KPI is:

```text
Percentage of critical/high findings remediated within SLA
```

This measures actual remediation performance rather than scanner volume alone.

---

## 180. Developer Security Feedback

Good security feedback is:

```text
specific
actionable
early
easy to reproduce
linked to remediation guidance
```

Avoid simply reporting:

```text
Security violation
```

without context.

---

## 181. Security Finding Example

Useful report:

```text
Finding:
CVE-XXXX

Package:
openssl

Current:
X

Fixed:
Y

Severity:
Critical

Affected Image:
user@sha256:...

Recommended Action:
Rebuild with patched base image.
```

---

## 182. DevSecOps Documentation

Document:

```text
scanner
version
policy
severity threshold
exception process
ownership
SLA
```

---

## 183. Scanner Version Management

Security tools themselves need maintenance.

Pin/test:

```text
Trivy
Sonar scanner
CI templates
DAST tooling
IaC scanner
```

---

## 184. Scanner Failure vs Security Failure

Distinguish:

```text
Scanner unavailable
```

from:

```text
Scanner found vulnerability
```

Do not silently treat infrastructure failure as a successful security result.

---

## 185. Fail Closed vs Fail Open

For mandatory production security controls, determine whether failure should:

```text
block deployment
```

rather than silently bypassing security.

Emergency exceptions should be explicit.

---

## 186. Security Pipeline Availability

If a security scanner is temporarily unavailable:

```text
retry
fallback if approved
manual exception
block production
```

Choose according to the risk policy.

---

## 187. Build Reproducibility

Security depends on knowing what was scanned.

Use:

```text
pinned dependencies
pinned base images
lock files
immutable image digests
```

---

## 188. Rebuild After Vulnerability

If a dependency is vulnerable:

```text
Update
 ↓
Rebuild
 ↓
Rescan
```

Do not assume changing source metadata without rebuilding changes the image.

---

## 189. Image Cache Risk

Docker build caches can accidentally retain outdated layers.

Use controlled cache invalidation and always scan the final image.

---

## 190. Artifact Integrity

The artifact promoted to production should be the same artifact that passed required security checks.

```text
Scan digest ABC
 ↓
Promote digest ABC
```

not:

```text
Scan image tag
 ↓
Rebuild tag
 ↓
Deploy different digest
```

---

## 191. GitOps Integrity

The GitOps repository should reference the exact approved artifact.

This creates:

```text
security result
 ↔
image digest
 ↔
deployment
```

traceability.

---

## 192. Promotion Integrity

Production promotion should not silently change:

```text
image
configuration
dependencies
```

without generating a new reviewable Git change.

---

## 193. Security Review for Infrastructure

Terraform MR reviewers should inspect:

```text
IAM
network
encryption
public exposure
logging
backup
```

---

## 194. Security Review for Kubernetes

Review:

```text
service accounts
RBAC
security context
network policies
ingress
secrets
resource limits
```

---

## 195. Security Review for Docker

Review:

```text
base image
user
packages
secrets
ports
entrypoint
```

---

## 196. Security Review for CI

Review:

```text
runner
variables
scripts
tokens
OIDC
artifacts
external downloads
```

---

## 197. Untrusted Pipeline Input

Be careful when pipeline scripts use:

```text
merge request variables
branch names
commit messages
user-controlled values
```

Avoid shell injection.

---

## 198. Shell Injection Example

Risky:

```bash
sh -c "deploy $USER_INPUT"
```

Prefer safe argument handling and validation.

---

## 199. CI Script Security

Treat `.gitlab-ci.yml` as executable code.

Changes to CI can change:

```text
credentials usage
build commands
deployment behavior
security controls
```

Protect it.

---

## 200. CI Configuration Review

Require review for changes to:

```text
.gitlab-ci.yml
security templates
deployment scripts
Terraform
GitOps manifests
```

---

## 201. Security Ownership

A mature model:

```text
Developers
→ secure application code

DevOps/Platform
→ secure CI/CD and infrastructure

Security
→ policy, assessment, governance

Everyone
→ incident response
```

---

## 202. DevSecOps Culture

DevSecOps is not:

```text
Security team blocks developers
```

It is:

```text
Engineering + Security
→ shared ownership
```

---

## 203. Security Automation

Automate repeatable controls:

```text
scan
policy
secret detection
dependency checks
image checks
```

Humans should focus on risk decisions.

---

## 204. Manual Security Review

Use human review for:

```text
critical exceptions
architecture
high-risk IAM
public exposure
production data
major security findings
```

---

## 205. Security-by-Default

Make the secure path the easiest path:

```text
approved base image
approved CI template
approved Terraform module
approved Helm chart
approved secret mechanism
```

---

## 206. Golden Paths

Platform teams can provide secure templates:

```text
GitLab pipeline
Dockerfile
Terraform module
Helm chart
ArgoCD Application
```

This reduces repeated security mistakes.

---

## 207. Golden Image

Maintain approved base images with:

```text
security patches
minimal packages
known ownership
versioning
scan results
```

---

## 208. Golden Terraform Module

Approved modules can enforce:

```text
encryption
tags
logging
private networking
least privilege
```

---

## 209. Golden Kubernetes Template

Approved application templates can include:

```text
non-root
probes
requests/limits
PDB
security context
NetworkPolicy
```

---

## 210. Security Policy Drift

Security policies themselves can drift.

Version-control:

```text
CI policy
IaC policy
Kubernetes policy
exception rules
```

---

## 211. Security Baseline

Define a baseline for:

```text
containers
Kubernetes
AWS
Terraform
GitLab
```

Then continuously check compliance.

---

## 212. Compliance Mapping

Controls can map to required frameworks/policies.

Examples:

```text
access control
logging
change management
vulnerability management
```

Use the organization's applicable standards.

---

## 213. Evidence Automation

Automate evidence collection:

```text
pipeline result
MR approval
scan report
deployment revision
artifact digest
```

This reduces audit effort.

---

## 214. Production Access

Prefer:

```text
GitOps
```

over:

```text
manual kubectl from laptops
```

for normal production changes.

Break-glass access should be restricted and audited.

---

## 215. Break-Glass Access

Emergency access should have:

```text
strong authentication
limited permissions
time restriction where possible
audit
post-incident review
```

---

## 216. Break-Glass and GitOps

If an emergency manual change is made:

```text
Incident
 ↓
Temporary manual fix
 ↓
Record change
 ↓
Update Git desired state
 ↓
Reconcile
```

Otherwise ArgoCD may restore the old state.

---

## 217. Manual Change During Incident

Do not disable self-heal permanently just because an incident requires a temporary manual action.

Document and reconcile the desired state afterward.

---

## 218. Final DevSecOps Checklist

```text
[ ] SAST
[ ] SCA
[ ] Secret scanning
[ ] Container scanning
[ ] IaC scanning
[ ] DAST where appropriate
[ ] SonarQube
[ ] Trivy
[ ] Veracode where applicable
[ ] SBOM
[ ] Image provenance
[ ] Immutable digest
[ ] ECR security
[ ] GitOps protection
[ ] ArgoCD RBAC
[ ] Kubernetes security
[ ] AWS least privilege
[ ] OIDC
[ ] Vulnerability SLA
[ ] Exception process
[ ] Audit trail
[ ] Runtime monitoring
[ ] Incident response
```

---

## 219. Senior Interview — Explain Your DevSecOps Pipeline

> My pipeline integrates security into CI rather than treating it as a final manual step. I use SonarQube for code analysis, dependency/security checks, Trivy for container and IaC scanning, Veracode where required for application security testing, and secret detection. Only an approved immutable artifact is promoted to ECR and then through GitOps to EKS.

---

## 220. Senior Interview — How Do You Prevent Vulnerable Images From Reaching Production?

> I scan the image before promotion, apply severity-based security gates, push only approved images to ECR, use immutable tags/digests, and ensure the GitOps repository references the exact artifact that passed the required checks. Additional registry/admission controls can provide defense in depth.

---

## 221. Senior Interview — What Do You Do With a Critical CVE in Production?

> I identify the affected image digest and environments, determine exploitability and exposure, rebuild with a patched dependency/base image, rescan the new image, promote the new digest through GitOps, deploy it, and verify the application. If immediate remediation is impossible, I apply documented compensating controls and an explicit security exception.

---

## 222. Senior Interview — What Happens If a Secret Is Committed?

> I treat the secret as compromised, immediately rotate or revoke it, investigate exposure, remove it from active source, migrate to an external secret mechanism, and review Git history and access logs as required. Removing the line alone is not sufficient.

---

## 223. Senior Interview — How Do You Secure GitLab CI?

> I protect CI configuration, use protected/masked variables, prefer OIDC over long-lived cloud keys, use trusted production runners, restrict job permissions, protect production environments, review pipeline changes, and avoid exposing secrets in logs.

---

## 224. Senior Interview — What Is the Difference Between SAST, SCA, DAST and Container Scanning?

> SAST analyzes source code, SCA analyzes third-party dependencies, DAST tests the running application, and container scanning analyzes the image and its packages. They cover different layers and should be used together.

---

## 225. Senior Interview — Why Use Trivy?

> Trivy provides practical scanning across container images, filesystems, repositories, IaC and Kubernetes configuration. In a DevSecOps pipeline it can provide fast feedback before artifacts are promoted.

---

## 226. Senior Interview — Why Use SonarQube?

> SonarQube provides source-code quality and security analysis, helping identify bugs, code smells and security issues. I use its quality/security gates to prevent unacceptable new findings from progressing.

---

## 227. Senior Interview — Why Use Veracode?

> Veracode can provide additional application security testing at the application layer. It complements source analysis and dependency/container scanning rather than replacing them.

---

## 228. Senior Interview — How Do You Handle False Positives?

> I validate the finding, determine whether it is actually exploitable in our context, document the reasoning, and create a time-bound exception with ownership and compensating controls if it cannot be immediately fixed. I don't permanently suppress findings without governance.

---

## 229. Senior Interview — How Do You Secure Terraform in DevSecOps?

> I scan Terraform for insecure configurations, enforce policies for encryption/public exposure/IAM, run validation and plan checks in GitLab, protect production apply, use OIDC for AWS authentication, and require review of high-risk infrastructure changes.

---

## 230. Senior Interview — How Do You Secure Kubernetes Deployments?

> I use non-root containers, restricted security contexts, resource controls, RBAC, NetworkPolicies, secret management, image scanning, admission policies, controlled service accounts and least-privilege AWS workload identity.

---

## 231. Senior Interview — How Do You Secure the Entire Supply Chain?

> I establish traceability from source commit to CI pipeline, image digest, security scan, SBOM, registry artifact, GitOps commit and production deployment. I use trusted dependencies, protected repositories, immutable artifacts, least privilege and appropriate signing/admission controls.

---

## 232. Senior Interview — What Is Your Production DevSecOps Architecture?

> GitLab hosts the source and runs CI. The pipeline performs testing, SonarQube analysis, dependency/security checks, secret detection, Trivy scanning and Veracode testing where applicable. The approved image is pushed to ECR, its immutable digest is promoted through GitOps, ArgoCD deploys it to EKS, and Prometheus, Grafana and ELK provide operational visibility.

---

## 233. Complete Production DevSecOps Architecture

```text
                         Developer
                             │
                             ▼
                          GitLab
                             │
                    ┌────────┴─────────┐
                    ▼                  ▼
                 Source               CI/CD
                                         │
              ┌──────────┬───────────────┼───────────────┐
              ▼          ▼               ▼               ▼
           SonarQube    SCA        Secret Scan        Tests
              │          │               │               │
              └──────────┴───────────────┼───────────────┘
                                         ▼
                                      Docker
                                         │
                                         ▼
                                       Trivy
                                         │
                               ┌─────────┴─────────┐
                               ▼                   ▼
                             SBOM             Image Sign
                               │                   │
                               └─────────┬─────────┘
                                         ▼
                                        ECR
                                         │
                                  Immutable Digest
                                         │
                                         ▼
                                  GitOps Repository
                                         │
                                         ▼
                                       ArgoCD
                                         │
                                         ▼
                                        EKS
                              ┌───────────┼───────────┐
                              ▼           ▼           ▼
                            ALB       Kubernetes   NetworkPolicy
                              │           │
                              └───────────┼───────────┘
                                          ▼
                                      Application
                                          │
                           ┌──────────────┼──────────────┐
                           ▼              ▼              ▼
                       Prometheus      Grafana           ELK
```

---

## 234. Final Production DevSecOps Workflow

```text
Developer
 ↓
GitLab MR
 ↓
Review
 ↓
Unit Tests
 ↓
SAST / SonarQube
 ↓
SCA
 ↓
Secret Detection
 ↓
Docker Build
 ↓
Trivy Image Scan
 ↓
IaC Scan
 ↓
SBOM
 ↓
Veracode / DAST where required
 ↓
Security Gate
 ↓
Push Approved Image to ECR
 ↓
Capture Digest
 ↓
Update GitOps Repository
 ↓
Production Approval
 ↓
ArgoCD
 ↓
EKS
 ↓
Smoke Tests
 ↓
Prometheus/Grafana/ELK
 ↓
Continuous Security Monitoring
```

---

## 235. Final DevSecOps Mental Model

```text
             PREVENT
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
    SAST       SCA      Secrets
      │         │         │
      └─────────┼─────────┘
                ▼
             DETECT
                │
        Container / IaC
                │
                ▼
             CONTROL
                │
       Security / Policy
                │
                ▼
             PROMOTE
                │
          Immutable ECR
                │
                ▼
             DEPLOY
                │
             ArgoCD
                │
                ▼
              EKS
                │
                ▼
             MONITOR
                │
        Prometheus / Grafana
                │
                ▼
              ELK
                │
                ▼
             RESPOND
                │
          Incident / Fix
                │
                └──────────► Improve controls
```

> **Core principle:** DevSecOps is not a collection of scanners. It is a controlled software supply chain where security findings are detected early, risk is evaluated, vulnerabilities are remediated, artifacts remain immutable, production deployment is authorized, and runtime behavior continues to be monitored.

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md ✓
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `15-GitLab-Security.md`**
