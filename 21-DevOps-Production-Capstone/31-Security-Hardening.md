# Security Hardening

## 1 Purpose

Production DevOps security is a continuous engineering discipline.

For this capstone, security must protect:

```text
Developer
   |
   v
Source Control
   |
   v
CI/CD
   |
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Container Registry
   |
   v
GitOps
   |
   v
Argo CD
   |
   v
AWS EKS
   |
   +--> Applications
   +--> Services
   +--> ALB
   +--> Secrets
   +--> Storage
   |
   v
Users / Customers
```

Security therefore cannot be limited to Kubernetes manifests.

It must cover:

```text
identity
network
source code
dependencies
containers
CI/CD
GitOps
AWS
Kubernetes
secrets
runtime
observability
logging
incident response
backup
compliance
```

---

 # 2 Security Objectives

A production security program protects:

### Confidentiality

Prevent unauthorized access to:

- credentials
- customer data
- secrets
- source code
- infrastructure state

### Integrity

Prevent unauthorized modification of:

- application images
- manifests
- Terraform
- Git repositories
- Kubernetes resources
- production configuration

### Availability

Protect against:

- resource exhaustion
- destructive changes
- denial of service
- accidental deletion
- compromised workloads

---

 # 3 Defense in Depth

Production security should use multiple independent controls.

```text
                 INTERNET
                    |
                    v
              AWS WAF / ALB
                    |
                    v
               VPC controls
                    |
                    v
                 EKS
                    |
        +-----------+-----------+
        |           |           |
     IAM/RBAC    Network      Runtime
        |         Policies     |
        |           |           |
        +-----------+-----------+
                    |
                 Pods
                    |
              Application
                    |
             Secrets / Data
```

If one layer fails, another layer should reduce the impact.

---

 # 4 Security Layers

The capstone security model:

```text
1. AWS account security
2. IAM
3. VPC
4. Security groups
5. NACLs
6. EKS control plane
7. Kubernetes RBAC
8. NetworkPolicy
9. Pod security
10. Container security
11. Image security
12. Application security
13. Secret management
14. CI/CD security
15. GitOps security
16. Observability security
17. Data security
18. Backup security
19. Incident response
```

---

 # 5 Shared Responsibility

Cloud providers secure the underlying cloud infrastructure.

Customers remain responsible for configuration and workloads.

For example:

```text
AWS:
physical infrastructure
managed service infrastructure
```

Customer:

```text
IAM
security groups
Kubernetes RBAC
applications
container images
secrets
data
configuration
```

A managed EKS control plane does not make the application automatically secure.

---

 # 6 AWS Account Security

Production should use controlled AWS accounts.

A mature structure may separate:

```text
Management
Security
Logging
Production
Non-production
```

The exact account model depends on organizational requirements.

Avoid using the root account for normal operations.

---

 # 7 Root Account Protection

The AWS root account should:

- have MFA
- have no unnecessary access keys
- be used only for account-level tasks
- have recovery information protected
- be monitored

Never use root credentials in:

```text
Terraform
Jenkins
GitHub Actions
application pods
developer laptops
```

---

 # 8 IAM Least Privilege

Principle:

```text
Give only the permissions required.
```

Bad:

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

This is effectively administrative access.

Prefer scoped permissions.

---

 # 9 IAM Roles Over Long-Lived Keys

Prefer:

```text
IAM Role
```

over:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

for workloads that support role-based authentication.

Benefits:

- short-lived credentials
- easier rotation
- lower secret exposure
- centralized control

---

 # 10 EKS IAM Integration

EKS workloads can use AWS IAM roles through modern workload identity mechanisms.

Conceptually:

```text
Pod
 |
 v
ServiceAccount
 |
 v
IAM role
 |
 v
AWS API
```

This is safer than putting AWS access keys inside a Secret.

---

 # 11 Kubernetes RBAC

Kubernetes RBAC controls:

```text
who
can perform
which action
on which resource
```

Core objects:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
ServiceAccount
```

---

 # 12 Namespace-Level RBAC

Prefer namespace-scoped permissions when possible.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: payment-read
  namespace: roboshop
rules:
  - apiGroups: [""]
    resources:
      - pods
      - services
    verbs:
      - get
      - list
      - watch
```

This role does not grant cluster-wide administrative access.

---

 # 13 RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: payment-read-binding
  namespace: roboshop
subjects:
  - kind: ServiceAccount
    name: payment-reader
    namespace: roboshop
roleRef:
  kind: Role
  name: payment-read
  apiGroup: rbac.authorization.k8s.io
```

Important:

```text
Role
=
permissions

RoleBinding
=
who receives permissions
```

---

 # 14 Avoid ClusterRoleBinding for Applications

Avoid:

```yaml
kind: ClusterRoleBinding
```

for ordinary application ServiceAccounts.

A compromised application with cluster-admin privileges can become a cluster-wide security incident.

---

 # 15 Service Accounts

Every application should use an explicit ServiceAccount.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payment
  namespace: roboshop
```

Avoid sharing one highly privileged ServiceAccount across unrelated applications.

---

 # 16 Disable Unnecessary Service Account Token Mounting

If a workload does not need Kubernetes API access:

```yaml
spec:
  automountServiceAccountToken: false
```

This reduces credential exposure.

---

 # 17 Pod Security

Production workloads should follow Pod Security Standards.

Typical objectives:

```text
run as non-root
drop unnecessary capabilities
prevent privileged containers
restrict host access
use read-only root filesystem where practical
```

---

 # 18 Security Context

Example:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  runAsGroup: 10001
  fsGroup: 10001
  seccompProfile:
    type: RuntimeDefault
```

This provides stronger runtime isolation.

---

 # 19 Container Security Context

```yaml
containers:
  - name: payment
    image: <account>.dkr.ecr.<region>.amazonaws.com/payment:<digest>
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
      readOnlyRootFilesystem: true
```

Not every application can immediately support a read-only filesystem. Validate application behavior first.

---

 # 20 Privileged Containers

Avoid:

```yaml
securityContext:
  privileged: true
```

unless there is a documented infrastructure requirement.

Privileged containers significantly increase the blast radius of compromise.

---

 # 21 Linux Capabilities

Linux capabilities split traditional root privileges into smaller permissions.

Instead of granting everything:

```yaml
capabilities:
  drop:
    - ALL
```

Add only narrowly required capabilities.

---

 # 22 Host Namespace Restrictions

Avoid unnecessary:

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
```

These settings can increase access to host-level resources.

---

 # 23 HostPath Risks

Avoid:

```yaml
volumes:
  - name: host
    hostPath:
      path: /
```

A compromised container with broad host access can potentially affect the node.

---

 # 24 Network Segmentation

The production architecture should separate:

```text
Internet
Public subnets
Private application subnets
Data subnets where applicable
Management paths
```

EKS worker nodes should generally reside in private subnets.

---

 # 25 VPC Security Groups

Security groups should allow only required traffic.

Example conceptual flow:

```text
Internet
  |
  v
ALB :443
  |
  v
EKS nodes / load balancer targets
  |
  v
Application ports
```

Do not open:

```text
0.0.0.0/0
```

to arbitrary application ports.

---

 # 26 Security Group Anti-Pattern

Bad:

```text
TCP 0-65535
Source 0.0.0.0/0
```

This exposes the instance broadly.

Better:

```text
443 from approved clients
application port from ALB/security-group source
administrative access through controlled paths
```

---

 # 27 NACLs

Network ACLs operate at subnet boundaries.

Use them as an additional control layer.

Do not treat NACLs as a replacement for:

```text
security groups
Kubernetes NetworkPolicy
application authentication
```

---

 # 28 Kubernetes NetworkPolicy

NetworkPolicy can restrict pod-to-pod communication.

Default-deny example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

This should be implemented carefully because it can break legitimate traffic.

---

 # 29 Application NetworkPolicy

Example concept:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: payment
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: checkout
      ports:
        - protocol: TCP
          port: 8080
```

Now only approved workloads can access the payment service on the specified port, assuming the CNI supports the policy behavior being used.

---

 # 30 EKS Control Plane Security

Protect:

```text
Kubernetes API server
```

Controls include:

- strong IAM
- Kubernetes RBAC
- private endpoint considerations
- restricted public endpoint access
- audit logging
- least privilege
- cluster upgrades
- admission controls

---

 # 31 EKS Endpoint Strategy

A production cluster may use:

```text
private endpoint
```

or a controlled public endpoint depending on organizational architecture.

If public access is enabled:

```text
restrict source CIDRs
```

Do not expose the Kubernetes API broadly without a security reason.

---

 # 32 Kubernetes Audit Logs

Audit logs help answer:

```text
Who changed this resource?
When?
From where?
What action was performed?
```

Important during:

- security investigations
- accidental changes
- incident response
- compliance reviews

---

 # 33 EKS Logging

Enable appropriate EKS control-plane logs according to requirements.

Potential categories include:

```text
api
audit
authenticator
controllerManager
scheduler
```

Costs should be considered, but security evidence should not be removed blindly.

---

 # 34 Secrets Management

Do not store secrets directly in:

```text
Git
Dockerfiles
Helm values
Terraform source
application source code
```

Avoid:

```yaml
password: MyRealPassword123
```

---

 # 35 Kubernetes Secrets

Kubernetes Secrets are API objects, but base64 encoding is not encryption by itself.

This:

```bash
echo -n 'secret' | base64
```

does not make the value cryptographically secure.

Use encrypted storage and external secret management appropriate to the environment.

---

 # 36 AWS Secrets Manager

A production architecture can use:

```text
AWS Secrets Manager
        |
        v
external secret synchronization
        |
        v
Kubernetes Secret
        |
        v
Application
```

Use appropriate IAM permissions for the workload.

---

 # 37 Secret Rotation

Secrets should have a defined lifecycle:

```text
create
→ distribute
→ use
→ rotate
→ revoke
```

Examples:

- database credentials
- API keys
- TLS certificates
- signing keys

---

 # 38 Secret Exposure Response

If a credential is committed to Git:

```text
1. Assume it is compromised.
2. Revoke/rotate it immediately.
3. Investigate access.
4. Remove it from future commits where appropriate.
5. Scan repository history.
6. Identify downstream systems.
7. Document the incident.
```

Deleting the visible line is not enough.

---

 # 39 ECR Image Security

Production images should be:

```text
built in trusted CI
scanned
tagged immutably
stored in controlled registry
deployed using trusted references
```

Prefer immutable image identification.

Example:

```text
image@sha256:<digest>
```

rather than relying only on:

```text
latest
```

---

 # 40 Image Tagging

Avoid:

```text
latest
```

for production deployment.

Prefer:

```text
payment:1.4.7
```

and ideally resolve the deployment to:

```text
payment@sha256:<digest>
```

This improves reproducibility.

---

 # 41 Trivy

Trivy can scan:

```text
container images
filesystem
dependencies
Kubernetes configuration
IaC
```

Example:

```bash
trivy image <image>
```

For CI:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  <image>
```

Thresholds should be aligned with organizational vulnerability policy.

---

 # 42 Vulnerability Management

A vulnerability should be evaluated using:

```text
severity
exploitability
exposure
application context
compensating controls
fix availability
```

Do not blindly block every vulnerability without understanding the risk.

---

 # 43 SonarQube

SonarQube provides code-quality and security analysis.

Pipeline:

```text
Checkout
   |
Build
   |
Unit tests
   |
SonarQube
   |
Quality Gate
   |
Continue
```

The exact quality-gate policy should be defined by the organization.

---

 # 44 Veracode

Veracode can be used as part of application security testing.

A production pipeline may include:

```text
SAST
SCA
policy checks
```

The pipeline should define:

```text
blocking vulnerabilities
approval process
exceptions
remediation ownership
```

---

 # 45 CI/CD Security

The pipeline is a high-value target.

Protect:

```text
source code
pipeline definitions
credentials
runners
artifact registry
deployment credentials
```

A compromised CI system can become a supply-chain compromise.

---

 # 46 CI Runner Security

Prefer:

```text
ephemeral runners
```

where practical.

Benefits:

```text
clean environment
reduced persistence
lower credential residue
```

Do not allow untrusted pull requests to freely access production credentials.

---

 # 47 Pipeline Credential Isolation

Bad:

```text
all pipeline stages use production credentials
```

Better:

```text
build stage
→ build permissions

scan stage
→ scanner permissions

publish stage
→ ECR permissions

deployment stage
→ GitOps permissions
```

Use separate identities.

---

 # 48 GitHub/GitLab Security

Protect:

- branch rules
- protected branches
- pull requests
- CODEOWNERS
- required reviews
- secret scanning
- dependency scanning
- token permissions

Production deployment should not depend on a developer directly pushing to the production branch.

---

 # 49 GitOps Security

Git becomes the desired-state authority.

Therefore protect:

```text
Git repository
   |
   v
GitOps manifests
   |
   v
Argo CD
   |
   v
Production
```

Compromise of the GitOps repository can result in production compromise.

---

 # 50 GitOps Repository Protection

Use:

```text
protected branches
required reviews
CODEOWNERS
signed commits where appropriate
least-privileged tokens
audit logs
secret scanning
```

---

 # 51 Argo CD Security

Argo CD should have:

```text
least-privileged projects
restricted repositories
restricted destinations
RBAC
SSO where appropriate
TLS
audit logging
```

Do not give every Argo CD user unrestricted cluster administration.

---

 # 52 Argo CD Project

Projects can restrict:

```text
source repositories
destination clusters
destination namespaces
resource types
```

Conceptual example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  sourceRepos:
    - 'https://git.example.com/platform/roboshop-gitops.git'
  destinations:
    - namespace: roboshop
      server: https://kubernetes.default.svc
  clusterResourceWhitelist: []
```

The exact allowed resources should be explicitly defined for the application's needs.

---

 # 53 Argo CD RBAC

Use role-based permissions.

Example concept:

```text
developer
→ view applications

release-engineer
→ sync approved applications

platform-admin
→ administration
```

Avoid giving all users:

```text
*
```

permissions.

---

 # 54 GitOps Drift Security

If someone manually changes production:

```bash
kubectl edit deployment payment
```

Argo CD can detect drift.

This is useful because unauthorized changes become visible.

The operational policy should define whether Argo CD automatically self-heals or requires investigation before correction.

---

 # 55 Terraform Security

Protect:

```text
Terraform code
Terraform state
provider credentials
variables
backend
```

Never commit:

```text
terraform.tfstate
```

if it can contain secrets or sensitive values.

Use a secure remote backend.

---

 # 56 Terraform State Security

Production state should have:

```text
encryption
access control
versioning where supported
locking/concurrency protection
backup/recovery
auditability
```

Restrict who can modify state.

---

 # 57 Terraform IAM

Terraform should use an IAM role with only the permissions required to manage its infrastructure.

Avoid permanent administrator keys.

Example workflow:

```text
CI
 |
 v
OIDC / role assumption
 |
 v
Terraform role
 |
 v
AWS
```

---

 # 58 Infrastructure Drift

Security drift can occur when:

```text
manual AWS change
manual Kubernetes change
manual security group change
```

Production should detect drift through:

```text
Terraform plan
Argo CD
AWS Config / security tooling
monitoring
```

---

 # 59 ALB Security

The ALB is the public entry point.

Protect it with:

```text
HTTPS
TLS certificates
security groups
appropriate HTTP headers
WAF where required
rate limiting / application controls
```

---

 # 60 TLS

Production external traffic should generally use:

```text
HTTPS
```

Certificate management should be automated where possible.

Do not allow expired certificates to become a production incident.

Monitor:

```text
certificate expiration
```

---

 # 61 HTTP to HTTPS Redirect

Ingress can redirect:

```text
HTTP
 |
 v
HTTPS
```

The exact ALB Ingress annotations depend on the AWS Load Balancer Controller version and architecture.

Validate generated AWS resources after deployment.

---

 # 62 DNS Security

Protect DNS changes.

Unauthorized DNS modification can redirect users to malicious infrastructure.

Use:

```text
least privilege
MFA
change review
audit logs
```

---

 # 63 Application Security

Infrastructure security cannot compensate for insecure application code.

Applications should address:

```text
authentication
authorization
input validation
injection
secure headers
dependency vulnerabilities
logging
error handling
```

---

 # 64 SQL Injection

Bad:

```text
SELECT * FROM users WHERE name = '" + userInput + "'
```

Prefer parameterized queries.

DevSecOps should catch common application vulnerabilities before deployment.

---

 # 65 Dependency Security

Dependencies should be scanned for:

```text
known CVEs
malicious packages
outdated versions
license concerns
```

For Maven:

```bash
mvn dependency:tree
```

Use organizational dependency-scanning controls as part of CI.

---

 # 66 Supply Chain Security

Software supply-chain risks include:

```text
compromised dependency
compromised base image
compromised CI runner
malicious build script
stolen registry credentials
tampered artifact
```

Controls:

```text
trusted base images
dependency scanning
image scanning
immutable artifacts
restricted CI credentials
artifact provenance
```

---

 # 67 Base Image Security

Avoid unknown images.

Prefer approved sources.

Example:

```dockerfile
FROM eclipse-temurin:21-jre
```

The organization should validate and approve base images.

---

 # 68 Minimal Images

Smaller runtime images generally reduce:

```text
attack surface
storage
pull time
```

Avoid unnecessary packages.

Do not include:

```text
compilers
debug tools
cloud credentials
source code
```

in the runtime image unless required.

---

 # 69 Container User

Avoid:

```dockerfile
USER root
```

for normal application workloads.

Prefer:

```dockerfile
USER 10001
```

when the application supports it.

---

 # 70 Dockerfile Security

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --chown=10001:10001 app.jar .

USER 10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build the image in CI and scan it before publishing.

---

 # 71 Runtime Security

Monitor for:

```text
unexpected process execution
privilege escalation
unexpected network connections
filesystem modifications
abnormal resource consumption
```

Production security monitoring should correlate:

```text
Kubernetes
AWS
application
ELK
```

events.

---

 # 72 ELK Security

Logs can contain sensitive data.

Never log:

```text
passwords
tokens
session secrets
private keys
full payment credentials
```

Use structured logging and data masking.

---

 # 73 Log Access Control

Not every engineer needs unrestricted access to every production log.

Restrict access according to:

```text
team
environment
application
sensitivity
role
```

---

 # 74 Prometheus Security

Prometheus contains operational information.

Protect:

```text
Prometheus endpoint
Grafana
Alertmanager
```

Do not expose management endpoints publicly without appropriate authentication and network controls.

---

 # 75 Grafana Security

Use:

```text
SSO where appropriate
RBAC
least privilege
secure cookies
HTTPS
controlled data-source access
```

Avoid shared administrator accounts.

---

 # 76 Alertmanager Security

Alertmanager may contain:

```text
service names
hostnames
incident information
internal URLs
```

Protect its UI and API.

Notification integrations should use secure credentials stored outside Git.

---

 # 77 Security Monitoring

Security-related alerts may include:

```text
repeated authentication failures
unexpected IAM changes
new privileged role
security group opened to internet
unexpected Kubernetes admin activity
new privileged pod
image vulnerability
secret access anomaly
```

---

 # 78 Security Alert Example

A suspicious security-group change:

```text
TCP 22
0.0.0.0/0
```

Production response:

```text
1. Detect
2. Verify whether change was authorized
3. Identify actor
4. Revert if unauthorized
5. Investigate access logs
6. Rotate credentials if necessary
7. Add preventive control
```

---

 # 79 AWS Security Monitoring

A production environment may integrate services such as:

```text
CloudTrail
AWS Config
GuardDuty
Security Hub
```

where appropriate.

These complement:

```text
Prometheus
ELK
Kubernetes audit logs
```

They do not replace application observability.

---

 # 80 CloudTrail

CloudTrail helps investigate:

```text
AWS API activity
```

Examples:

```text
Who changed IAM?
Who modified security groups?
Who deleted infrastructure?
Who changed an S3 policy?
```

Protect CloudTrail logs from unauthorized modification.

---

 # 81 GuardDuty

GuardDuty can provide managed threat detection signals.

Potential categories include:

```text
credential compromise
malicious network activity
suspicious API behavior
```

Treat findings as investigation inputs, not automatic proof of compromise.

---

 # 82 Security Hub

Security Hub can centralize security findings across AWS security services.

This can help security teams prioritize findings.

---

 # 83 AWS Config

AWS Config can evaluate resource configuration.

Example policy:

```text
security group should not expose SSH globally
```

Configuration monitoring can detect drift from security standards.

---

 # 84 Production Security Baseline

Minimum baseline:

```text
MFA
least privilege IAM
private EKS nodes
controlled API endpoint
RBAC
NetworkPolicy
non-root containers
image scanning
dependency scanning
secret management
encrypted storage
TLS
audit logging
branch protection
GitOps controls
backup protection
security monitoring
```

---

 # 85 Encryption at Rest

Protect:

```text
EBS
RDS where applicable
Secrets
S3
Terraform state
logs where appropriate
```

Use AWS-managed or customer-managed keys according to requirements.

---

 # 86 Encryption in Transit

Use encryption for:

```text
Internet → ALB
ALB → application where required
application → database where supported
service → service where required
CI → AWS
Argo CD → repositories
```

The exact TLS topology depends on application requirements.

---

 # 87 KMS

AWS KMS can manage encryption keys.

Security controls include:

```text
key policies
IAM
rotation policy
audit logging
least privilege
```

Do not grant every workload unrestricted KMS access.

---

 # 88 Backup Security

Backups should be:

```text
encrypted
access-controlled
protected from accidental deletion
tested
monitored
```

A compromised production account should not automatically be able to destroy every backup.

---

 # 89 Immutable Backup Concepts

Where supported and required, use:

```text
retention controls
backup vault protection
cross-account copies
cross-region copies
```

This reduces ransomware and accidental-deletion risk.

---

 # 90 Security and Disaster Recovery

Security incidents can become DR events.

Example:

```text
Production account compromised
        |
        v
Credentials revoked
        |
        v
Traffic isolated
        |
        v
Known-good infrastructure restored
        |
        v
Known-good application images
        |
        v
Data restored
```

DR plans should include cyberattack scenarios.

---

 # 91 Production Security YAML

A hardened Deployment example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
  namespace: roboshop
  labels:
    app: payment
    environment: prod
    team: payments
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
        environment: prod
        team: payments
    spec:
      serviceAccountName: payment
      automountServiceAccountToken: false

      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: payment
          image: <account>.dkr.ecr.<region>.amazonaws.com/payment@sha256:<digest>

          ports:
            - containerPort: 8080

          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi

          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: payment-db
                  key: host

          readinessProbe:
            httpGet:
              path: /health
              port: 8080

          livenessProbe:
            httpGet:
              path: /health
              port: 8080
```

---

 # 92 YAML Security Explanation

Important fields:

```text
serviceAccountName
```

defines the workload identity.

```text
automountServiceAccountToken: false
```

reduces unnecessary Kubernetes API credentials.

```text
runAsNonRoot
```

prevents normal execution as root.

```text
seccompProfile
```

adds syscall restrictions.

```text
allowPrivilegeEscalation: false
```

reduces privilege escalation paths.

```text
capabilities.drop: ALL
```

removes unnecessary Linux capabilities.

```text
readOnlyRootFilesystem
```

prevents many filesystem modifications.

```text
image@sha256
```

pins the deployment to a specific immutable artifact.

---

 # 93 Pod Security Admission

A namespace can be labeled according to Pod Security Standards.

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

Validate application compatibility before enforcing this in production.

---

 # 94 Security Policy as Code

Security should be automated.

Possible checks:

```text
no privileged pods
no hostPath
runAsNonRoot
resource limits
approved registries
no latest tags
required labels
approved namespaces
```

These can be enforced through:

```text
CI
admission policies
policy engines
Terraform policies
```

---

 # 95 Admission Control

Admission controls can prevent insecure resources before they run.

Concept:

```text
kubectl apply
      |
      v
API server
      |
      v
Admission policy
      |
  +---+---+
  |       |
allow   deny
```

Examples of denied resources:

```text
privileged=true
hostNetwork=true
unapproved registry
missing security context
```

---

 # 96 Image Admission

Production policy:

```text
Only images from approved ECR repositories.
```

Possible validation:

```text
registry
tag
digest
signature/provenance
vulnerability policy
```

The exact implementation depends on the admission tooling selected by the organization.

---

 # 97 NetworkPolicy Baseline

A production namespace can start with:

```text
default deny
```

and explicitly allow:

```text
ALB/application ingress
application-to-application
DNS
metrics
logging
required external dependencies
```

Document every exception.

---

 # 98 DNS Policy

Applications often require DNS.

If NetworkPolicy blocks all egress, DNS may fail.

Allow appropriate DNS traffic to the cluster DNS service.

This is a common production NetworkPolicy mistake.

---

 # 99 Egress Control

Do not allow every pod unrestricted outbound internet access if business requirements do not require it.

Egress policy can reduce:

```text
data exfiltration
command-and-control paths
unapproved external dependencies
```

But overly restrictive egress can break:

```text
AWS APIs
external payment APIs
package services
DNS
```

Test carefully.

---

 # 100 Security Hardening Workflow

```text
Discover
   |
   v
Assess risk
   |
   v
Prioritize
   |
   v
Implement controls
   |
   v
Test
   |
   v
Deploy
   |
   v
Monitor
   |
   v
Review
```

---

 # 101 Production Security Review

Before production deployment:

```text
[ ] IAM least privilege
[ ] MFA enabled
[ ] no long-lived secrets in source
[ ] ECR scanning
[ ] dependency scanning
[ ] secure Dockerfile
[ ] non-root container
[ ] RBAC
[ ] NetworkPolicy
[ ] TLS
[ ] private nodes
[ ] restricted API endpoint
[ ] audit logging
[ ] backup encryption
[ ] Git branch protection
[ ] Argo CD RBAC
[ ] Terraform state protected
```

---

 # 102 Security Failure Scenario: Compromised Pod

### Symptom

ELK shows:

```text
unexpected process
```

### Investigation

```bash
kubectl get pod -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

Check:

```text
image
service account
RBAC
network connections
secrets
node
```

### Containment

```text
isolate workload
block network access
scale down compromised deployment if safe
revoke exposed credentials
```

### Recovery

```text
deploy known-good image
restore required configuration
rotate secrets
validate workload
```

### Prevention

```text
non-root
NetworkPolicy
image scanning
least privilege
runtime monitoring
```

---

 # 103 Security Failure Scenario: Exposed AWS Credentials

### Symptom

Unexpected AWS API calls.

### Investigation

```text
CloudTrail
IAM
application logs
CI logs
Git history
```

### Immediate response

```text
revoke credential
rotate credential
identify affected resources
review API activity
```

### Prevention

```text
IAM roles
OIDC
secret scanning
least privilege
```

---

 # 104 Security Failure Scenario: Vulnerable Image

### Symptom

Trivy reports:

```text
CRITICAL vulnerability
```

### Response

```text
1. Determine whether exploitable.
2. Identify affected production workloads.
3. Find patched base/dependency.
4. Rebuild.
5. Scan again.
6. Push new image.
7. Update GitOps.
8. Argo CD deploys.
9. Verify.
```

---

 # 105 Security Failure Scenario: Malicious GitOps Commit

### Symptom

Unexpected production resource appears.

### Investigation

```text
Git history
PR
reviewer
commit author
Argo CD history
Kubernetes audit log
```

### Response

```text
revert commit
disable compromised account
rotate tokens
validate production
review repository permissions
```

---

 # 106 Security Failure Scenario: Public Security Group

### Symptom

Security monitoring reports:

```text
SSH exposed to internet
```

### Response

```text
identify change
verify authorization
remove exposure
review CloudTrail
check access attempts
```

Prevention:

```text
Terraform policy
AWS Config
security review
least privilege
```

---

 # 107 Security Testing

Security controls should be tested.

Examples:

```bash
kubectl auth can-i get secrets \
  --as=system:serviceaccount:roboshop:payment \
  -n roboshop
```

Expected:

```text
no
```

for workloads that do not need Secret API access.

Test NetworkPolicy:

```text
approved pod → approved service = allowed
unapproved pod → protected service = denied
```

---

 # 108 RBAC Verification

Useful:

```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:roboshop:payment \
  -n roboshop
```

This reveals effective permissions and helps detect over-privilege.

---

 # 109 Security Scanning Pipeline

Production pipeline:

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Unit tests
   |
   +--> SonarQube
   |
   +--> Dependency scan
   |
   +--> Veracode
   |
   +--> Docker build
   |
   +--> Trivy image scan
   |
   +--> Push ECR
   |
   v
GitOps update
   |
   v
Argo CD
   |
   v
EKS
```

---

 # 110 Security Gates

A production pipeline may stop when:

```text
critical vulnerability exceeds policy
quality gate fails
secret detected
malicious dependency detected
image from unapproved registry
security policy fails
```

Exceptions should be:

```text
documented
time-bound
approved
tracked
```

---

 # 111 Security and GitOps Separation

The architecture should separate:

```text
application source
```

from:

```text
deployment desired state
```

CI should not require unrestricted Kubernetes administrator credentials.

Instead:

```text
CI
 |
 v
GitOps repository
 |
 v
Argo CD
 |
 v
EKS
```

This reduces direct production credential exposure.

---

 # 112 Production Security Architecture

```text
                         INTERNET
                            |
                            v
                         HTTPS
                            |
                            v
                    +---------------+
                    |   AWS ALB     |
                    +---------------+
                            |
                     Security Groups
                            |
                            v
                    +---------------+
                    |      EKS      |
                    +---------------+
                       |    |    |
                     RBAC Network Pod
                       |  Policy Security
                       |    |    |
                       +----+----+
                            |
                  +---------+---------+
                  |                   |
              Applications        Secrets
                  |                   |
                  v                   v
                ECR              Secrets Manager
                  |
                  v
               Scanning

CI/CD:
Git → CI → SonarQube → Veracode → Trivy → ECR
                         |
                         v
                     GitOps
                         |
                         v
                      Argo CD

Security telemetry:
AWS CloudTrail / GuardDuty / Config
             |
             v
           ELK / Security Operations
```

---

 # 113 Security Ownership

Security responsibilities should be explicit.

Example:

```text
Platform team
→ EKS / IAM / network baseline

DevOps
→ CI/CD / GitOps / infrastructure

Application team
→ application vulnerabilities

Security team
→ policies / threat detection / risk

Database team
→ data protection

Developers
→ secure coding
```

Shared responsibility must not become:

```text
everyone owns security
=
nobody owns security
```

---

 # 114 Security Metrics

Track:

```text
critical vulnerabilities open
mean time to remediate
secrets detected
privileged pods
RBAC violations
publicly exposed resources
failed security checks
image vulnerabilities
certificate expiry
security incidents
```

---

 # 115 Mean Time to Remediate

MTTR for security vulnerabilities should be measured.

Example:

```text
Critical:
24 hours

High:
7 days

Medium:
30 days
```

These are example organizational targets, not universal requirements.

---

 # 116 Security Exception Process

Sometimes remediation cannot happen immediately.

Exception should record:

```text
vulnerability
risk
affected service
reason
compensating controls
owner
approval
expiry date
remediation plan
```

Avoid permanent exceptions.

---

 # 117 Production Hardening Checklist

## AWS

- [ ] Root MFA
- [ ] IAM least privilege
- [ ] Roles instead of keys
- [ ] CloudTrail
- [ ] security monitoring
- [ ] encrypted storage
- [ ] controlled security groups
- [ ] protected backups

## EKS

- [ ] private nodes
- [ ] controlled API endpoint
- [ ] RBAC
- [ ] audit logging
- [ ] Pod Security
- [ ] NetworkPolicy
- [ ] workload identity
- [ ] regular upgrades

## Containers

- [ ] trusted base image
- [ ] Trivy scanning
- [ ] non-root
- [ ] minimal image
- [ ] immutable digest
- [ ] no privileged mode
- [ ] capabilities dropped

## CI/CD

- [ ] protected branches
- [ ] secret scanning
- [ ] SonarQube
- [ ] Veracode
- [ ] dependency scanning
- [ ] Trivy
- [ ] least privilege credentials
- [ ] ephemeral runners

## GitOps

- [ ] protected repository
- [ ] PR review
- [ ] Argo CD RBAC
- [ ] restricted projects
- [ ] audit trail
- [ ] no secrets in Git

---

 # 118 Senior Interview Answer: How Do You Secure EKS?

A strong production answer:

```text
I use defense in depth.

At the AWS layer I use least-privilege IAM, private networking,
controlled security groups, encryption, CloudTrail and security
monitoring.

At the EKS layer I restrict API access, use Kubernetes RBAC,
workload identity, audit logging and Pod Security controls.

At the workload layer I use non-root containers, dropped Linux
capabilities, seccomp, NetworkPolicy, resource limits and immutable
container images.

At the supply-chain layer I protect Git, CI/CD and GitOps, and use
SonarQube, dependency scanning, Veracode and Trivy.

Secrets are managed outside source code and rotated.

Finally, I continuously monitor security findings and test the
controls instead of assuming that configuration alone means security.
```

---

 # 119 Senior Interview Answer: How Do You Prevent Container Escape Risk?

```text
I avoid privileged containers, host namespaces and unnecessary
hostPath mounts. I run workloads as non-root, drop Linux capabilities,
disable privilege escalation, use seccomp and enforce Pod Security
policies.

I also restrict service-account permissions and network access.
Image scanning and runtime monitoring provide additional layers.
```

---

 # 120 Senior Interview Answer: How Do You Secure CI/CD?

```text
I treat CI/CD as production infrastructure.

I use protected repositories and branches, short-lived credentials,
least-privileged identities, isolated or ephemeral runners, secret
scanning, dependency scanning, static analysis and container scanning.

The pipeline should not have unrestricted production Kubernetes
credentials. The preferred flow is CI producing a trusted artifact
and updating GitOps configuration, followed by Argo CD reconciling
the desired state.
```

---

 # 121 Senior Interview Answer: How Do You Handle a Leaked Secret?

```text
I assume the secret is compromised.

First I revoke or rotate it immediately. Then I investigate where it
was exposed, review access logs, identify impacted systems and remove
the secret from future source versions.

I also scan repository history and CI logs, rotate related credentials,
and add preventive controls such as secret scanning and external
secret management.
```

---

 # 122 Senior Interview Answer: How Do You Secure GitOps?

```text
The GitOps repository is part of the production security boundary.

I protect branches, require reviews, restrict repository write access,
avoid secrets in Git, use Argo CD projects and RBAC, restrict allowed
clusters and namespaces, and audit changes.

I also monitor Argo CD drift and Kubernetes audit activity so that
unexpected production changes can be traced back to their source.
```

---

 # 123 Security Operations Workflow

```text
Security Event
     |
     v
Detection
     |
     v
Triage
     |
     v
Severity
     |
     v
Containment
     |
     v
Eradication
     |
     v
Recovery
     |
     v
Validation
     |
     v
Lessons Learned
     |
     v
Preventive Control
```

---

 # 124 Final Production Security Model

The complete security mindset is:

```text
Identity
   +
Network
   +
Application
   +
Container
   +
Kubernetes
   +
CI/CD
   +
GitOps
   +
Secrets
   +
Data
   +
Observability
   +
Backup
   +
Incident Response
```

Security is not a single tool.

It is a chain of controls.

The production DevOps engineer should continuously ask:

```text
Who can access this?

What can they access?

From where?

For how long?

What happens if the credential is compromised?

What happens if the container is compromised?

What happens if Git is compromised?

What happens if AWS credentials are compromised?

Can we detect it?

Can we contain it?

Can we recover?
```

A production system is properly hardened when security controls are:

```text
preventive
+
detective
+
responsive
+
recoverable
```

and are continuously tested.

---

 # 125 Final Security Principle

```text
Least privilege
        +
Defense in depth
        +
Secure software supply chain
        +
Immutable artifacts
        +
Strong identity
        +
Network segmentation
        +
Runtime restrictions
        +
Continuous monitoring
        +
Tested recovery
        =
Production security
```

For the RoboShop production capstone, security is therefore integrated into every stage:

```text
Code
 ↓
CI
 ↓
Security Scans
 ↓
Artifact
 ↓
ECR
 ↓
GitOps
 ↓
Argo CD
 ↓
EKS
 ↓
ALB
 ↓
Application
 ↓
Metrics / Logs
 ↓
Detection
 ↓
Incident Response
 ↓
Recovery
```

That is the production DevSecOps security model.

---