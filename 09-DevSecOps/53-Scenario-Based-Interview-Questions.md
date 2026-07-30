# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 1

## Jenkins pipeline completed successfully, ArgoCD synced successfully, but users receive HTTP 502 Bad Gateway immediately after deployment. How would you investigate?

### Expected Approach

Investigate from the user entry point:

- ALB Target Groups
- Ingress Controller
- Kubernetes Service
- Endpoints
- Pod Readiness
- Application Logs
- Backend Dependencies

Never assume a successful deployment means a healthy application.

---

# Scenario 2

## SonarQube Quality Gate passed, but the application crashes immediately after deployment. What could be the reason?

### Expected Approach

Remember that SonarQube checks code quality, not runtime behaviour.

Investigate:

- Application startup logs
- Environment variables
- ConfigMaps
- Secrets
- Database connectivity
- JVM/Runtime configuration
- Resource limits

---

# Scenario 3

## Trivy reports a Critical vulnerability in the Docker image just before production deployment. The business wants an urgent release. What would you do?

### Expected Approach

- Validate whether the vulnerability is exploitable.
- Check whether a fixed image exists.
- Assess business impact.
- Block deployment if the risk is unacceptable.
- Use the organization's risk acceptance process only when justified and approved.

---

# Scenario 4

## A developer accidentally commits AWS Access Keys into GitHub. The repository has already been pushed. What actions would you take?

### Expected Approach

Immediately:

- Revoke the exposed keys.
- Generate new credentials.
- Remove secrets from Git history.
- Scan repositories using Gitleaks.
- Review CloudTrail for unauthorized activity.
- Inform the security team.
- Replace hardcoded secrets with AWS Secrets Manager.

---

# Scenario 5

## ArgoCD shows "Synced", but the application version running in Kubernetes is still the old version. How would you troubleshoot?

### Expected Approach

Verify:

- Image tag
- ImagePullPolicy
- Deployment rollout
- ReplicaSet
- Pod status
- Git repository revision
- ArgoCD application history
- Container image digest

---

# Scenario 6

## After deploying a new version, Kubernetes Pods remain in Running state, but Readiness probes continue to fail. What would you investigate?

### Expected Approach

Check:

- Readiness endpoint
- Application startup logs
- Database availability
- External API connectivity
- Configuration
- Secrets
- Resource consumption
- Probe timeout configuration

---

# Scenario 7

## Terraform deployment suddenly fails even though the code hasn't changed. How would you investigate?

### Expected Approach

Review:

- Terraform state
- Backend availability
- Provider version
- Cloud credentials
- IAM permissions
- Resource drift
- API limits
- Recently modified infrastructure

---

# Scenario 8

## Your Amazon EKS application suddenly becomes very slow after a successful deployment. CPU usage is normal. Where would you investigate?

### Expected Approach

Check:

- Database latency
- Redis cache
- DNS resolution
- Network latency
- API dependencies
- Storage performance
- Application logs
- Thread pool utilisation

Performance issues are not always caused by CPU.

---

# Scenario 9

## Falco generates an alert stating that a shell was opened inside a production container. What actions would you take?

### Expected Approach

- Treat the alert as a potential security incident.
- Identify the affected pod.
- Verify who accessed the container.
- Review Kubernetes Audit Logs.
- Collect container logs.
- Isolate the workload if required.
- Investigate for compromise.
- Rotate affected credentials if necessary.

---

# Scenario 10

## A deployment completed successfully, but only users from one region are reporting application failures. How would you approach the investigation?

### Expected Approach

Investigate:

- Route53 routing
- Regional ALB health
- DNS propagation
- Network connectivity
- Regional infrastructure
- Kubernetes cluster health
- Backend service availability
- Recent regional changes

Avoid assuming the application itself is the root cause.

---

# Enterprise Investigation Flow

```text
Understand Problem

↓

Collect Evidence

↓

Check Recent Changes

↓

Infrastructure

↓

Platform

↓

Application

↓

Dependencies

↓

Root Cause

↓

Fix

↓

Validation

↓

Post-Incident Review
```

---

# Enterprise Best Practices

- Never troubleshoot based on assumptions.
- Always verify the scope and business impact first.
- Follow a structured investigation process from infrastructure to application.
- Correlate logs, metrics, events, and deployment history.
- Use GitOps for rollbacks instead of manual production changes.
- Preserve audit logs and evidence during security incidents.
- Document the root cause and preventive actions after every incident.
- Continuously improve monitoring, alerting, and automation based on production learnings.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 11

## A Jenkins pipeline suddenly starts failing at the Docker Build stage even though no code changes were made. How would you investigate?

### Expected Approach

Investigate:

- Jenkins agent availability
- Docker daemon status
- Disk space
- Build cache
- Base image availability
- Dockerfile changes
- Registry authentication
- Recent Jenkins plugin updates

---

# Scenario 12

## After deploying a new application version, Pods start showing `ImagePullBackOff`. The image exists in Amazon ECR. What would you check?

### Expected Approach

Verify:

- Image tag
- Repository name
- IAM Role for Service Account (IRSA)
- Node IAM permissions
- ECR authentication
- ImagePullSecrets
- Network connectivity
- Repository policy

---

# Scenario 13

## Your application works correctly inside the cluster, but users cannot access it through the Application Load Balancer. How would you troubleshoot?

### Expected Approach

Check:

- Route53 DNS
- ALB Listener Rules
- Target Group Health
- Ingress configuration
- Security Groups
- Kubernetes Service
- Service Endpoints
- Pod readiness

---

# Scenario 14

## A Terraform deployment attempts to recreate existing production resources instead of updating them. What could be the cause?

### Expected Approach

Investigate:

- Terraform state file
- Remote backend
- State locking
- Resource import status
- Workspace selection
- Drift between state and infrastructure
- Provider configuration

---

# Scenario 15

## After a successful deployment, users report extremely slow response times. Monitoring shows low CPU and memory utilisation. How would you investigate?

### Expected Approach

Check:

- Database query performance
- External API latency
- Redis cache
- Network latency
- DNS resolution
- Storage IOPS
- Thread pool exhaustion
- Connection pool utilisation

---

# Scenario 16

## Gitleaks detects hardcoded credentials in a Pull Request. The developer says they are test credentials. What would you do?

### Expected Approach

- Validate whether the credentials are active.
- Block the merge until verified.
- Remove secrets from Git history if necessary.
- Rotate credentials if exposed.
- Educate the developer.
- Store secrets in a secrets manager instead of Git.

---

# Scenario 17

## After updating Kubernetes Secrets, the application continues using the old credentials. Why?

### Expected Approach

Verify:

- Secret update completed
- Deployment rollout
- Pod restart
- Secret mount type
- Environment variable loading
- Application secret reload capability

Remember that many applications only read environment variables during startup.

---

# Scenario 18

## ArgoCD reports an application as `OutOfSync`. What would you investigate?

### Expected Approach

Review:

- Git repository
- Desired manifest
- Live Kubernetes resources
- Manual cluster changes
- Sync policy
- Failed sync operations
- Resource health
- Drift between Git and cluster

---

# Scenario 19

## Falco reports that a container attempted to modify a sensitive system file. How would you respond?

### Expected Approach

Treat it as a potential security incident.

Investigate:

- Container identity
- Namespace
- Pod owner
- Recent deployments
- Container logs
- Kubernetes Audit Logs
- User activity
- Possible privilege escalation

Contain the workload if malicious behaviour is confirmed.

---

# Scenario 20

## During a production deployment, SonarQube passes, Trivy passes, Checkov passes, and Jenkins succeeds. However, customers report missing data after deployment. How would you investigate?

### Expected Approach

Check:

- Database migration scripts
- Application version compatibility
- Rollback history
- Backup availability
- Deployment sequence
- API version changes
- Data validation
- Application logs

Remember that security scans validate security posture—they do not guarantee application functionality or data integrity.

---

# Enterprise Investigation Flow

```text
Incident Report

↓

Confirm Impact

↓

Review Recent Changes

↓

Infrastructure

↓

Platform

↓

Application

↓

Dependencies

↓

Root Cause

↓

Recovery

↓

Validation

↓

Lessons Learned
```

---

# Enterprise Best Practices

- Always validate the business impact before beginning technical troubleshooting.
- Compare recent deployments, infrastructure changes, and configuration updates.
- Correlate logs, metrics, traces, and audit events to identify the root cause.
- Avoid manual fixes in GitOps-managed environments unless required for emergency recovery.
- Preserve evidence during security incidents before making irreversible changes.
- Test rollback procedures regularly to ensure rapid recovery.
- Perform post-incident reviews and implement preventive controls.
- Continuously improve observability, automation, and deployment practices based on production experience.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 21

## A developer reports that Jenkins fails during the SonarQube Quality Gate stage, but the SonarQube dashboard shows "Passed." How would you investigate?

### Expected Approach

Check:

- Jenkins SonarQube plugin configuration
- SonarQube webhook delivery
- Project Key
- Authentication token
- Network connectivity
- Quality Gate timeout
- Jenkins logs
- SonarQube server logs

---

# Scenario 22

## After a successful deployment, only one microservice is unable to communicate with another service inside the Kubernetes cluster. How would you troubleshoot?

### Expected Approach

Verify:

- Service DNS resolution
- Kubernetes Service configuration
- Endpoints
- Network Policies
- Pod labels
- Namespace configuration
- Application logs
- Target service availability

---

# Scenario 23

## Your AWS infrastructure deployment fails with an "Access Denied" error, even though Terraform code has not changed. What would you investigate?

### Expected Approach

Review:

- IAM Role permissions
- IAM policy changes
- Service Control Policies (SCP)
- Temporary credential expiration
- AWS account changes
- Terraform backend access
- CloudTrail logs

---

# Scenario 24

## After a production deployment, CPU usage suddenly reaches 95% across multiple Pods. How would you investigate?

### Expected Approach

Check:

- Recent application changes
- Infinite loops
- Traffic spikes
- HPA behaviour
- Resource requests and limits
- JVM or runtime tuning
- Application profiling
- Database response time

---

# Scenario 25

## ArgoCD continuously reports "OutOfSync" even after every successful synchronization. What could be the reason?

### Expected Approach

Investigate:

- Manual cluster changes
- Mutating admission webhooks
- Helm-generated values
- Resource annotations
- Custom Resource Definitions
- Sync options
- IgnoreDifferences configuration

---

# Scenario 26

## A production Pod is repeatedly getting OOMKilled. Increasing memory fixes the issue temporarily, but it happens again after a few hours. How would you investigate?

### Expected Approach

Verify:

- Memory leaks
- Heap dumps
- Application profiling
- Garbage Collection behaviour
- Resource limits
- Traffic patterns
- Large object allocation
- Recent code changes

---

# Scenario 27

## Your Docker image size suddenly increases from 350 MB to 2.5 GB. How would you determine the cause?

### Expected Approach

Review:

- Dockerfile changes
- Base image updates
- Installed packages
- Multi-stage build usage
- Large dependencies
- Build artifacts
- Layer history
- `.dockerignore` configuration

---

# Scenario 28

## A security team reports that an application is using a vulnerable open-source library even though Dependency Scanning passed. How would you investigate?

### Expected Approach

Check:

- Dependency database updates
- Scanner version
- Transitive dependencies
- SBOM contents
- Package lock files
- Manual dependency additions
- False negatives
- Scanner configuration

---

# Scenario 29

## A Kubernetes Deployment remains in the "Progressing" state for a long time and never completes. What would you check?

### Expected Approach

Verify:

- ReplicaSet status
- Pod readiness
- Image availability
- Scheduling events
- Resource availability
- Readiness probe failures
- Persistent Volume issues
- Deployment strategy

---

# Scenario 30

## During a routine security review, you discover that several developers have AdministratorAccess in the AWS Production account. How would you address this?

### Expected Approach

Take a structured approach:

- Review business requirements
- Identify unused permissions
- Apply Least Privilege
- Use IAM Roles instead of users
- Enable MFA
- Review CloudTrail activity
- Implement periodic access reviews
- Document and approve privileged access

---

# Enterprise Investigation Flow

```text
Production Issue

↓

Identify Scope

↓

Review Recent Changes

↓

Infrastructure

↓

Cloud Services

↓

Kubernetes

↓

Application

↓

Security

↓

Root Cause

↓

Recovery

↓

Validation

↓

Prevent Recurrence
```

---

# Enterprise Best Practices

- Verify identity, access, and configuration changes before assuming application defects.
- Correlate cloud logs, Kubernetes events, and CI/CD history during investigations.
- Follow the Principle of Least Privilege for all production access.
- Keep Terraform state, infrastructure, and Git repositories synchronized.
- Continuously monitor resource utilisation and application behaviour after deployments.
- Investigate recurring issues to identify root causes instead of applying temporary fixes.
- Maintain an up-to-date SBOM and regularly update vulnerability databases.
- Conduct post-incident reviews and convert findings into preventive controls and automation.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 31

## A deployment completed successfully, but ArgoCD reports the application as Healthy while customers still receive HTTP 503 Service Unavailable. How would you investigate?

### Expected Approach

Check:

- ALB Target Group health
- Kubernetes Service endpoints
- Pod readiness status
- Ingress configuration
- Backend application logs
- Health check path
- Recent deployment changes
- Network connectivity

---

# Scenario 32

## Your Jenkins pipeline suddenly fails during the Trivy scan because of a database update error. How would you troubleshoot?

### Expected Approach

Verify:

- Internet connectivity
- Trivy database cache
- Trivy version
- Proxy configuration
- Offline database mirror
- Runner disk space
- Registry connectivity
- Retry strategy

---

# Scenario 33

## A developer accidentally deletes the Terraform remote state file stored in Amazon S3. What actions would you take?

### Expected Approach

Investigate:

- S3 Versioning
- Backup availability
- State recovery options
- DynamoDB state lock status
- Cloud resources already provisioned
- Terraform plan before applying
- Team communication
- Recovery validation

---

# Scenario 34

## Falco generates repeated alerts indicating that multiple containers are attempting privilege escalation. How would you respond?

### Expected Approach

Check:

- Affected namespaces
- Container images
- Deployment history
- Kubernetes Audit Logs
- RBAC permissions
- Host activity
- Recent image changes
- Isolate affected workloads if malicious behaviour is confirmed

---

# Scenario 35

## During deployment, Kubernetes Pods cannot connect to Amazon RDS even though the database is healthy. What would you investigate?

### Expected Approach

Verify:

- Security Groups
- Network ACLs
- VPC routing
- RDS endpoint
- DNS resolution
- IAM authentication (if used)
- Secrets
- Database firewall rules

---

# Scenario 36

## After merging a Pull Request, Jenkins builds an older commit instead of the latest code. How would you troubleshoot?

### Expected Approach

Review:

- Git branch configuration
- Jenkins webhook
- SCM polling
- Repository cache
- Branch references
- Detached HEAD state
- Pipeline configuration
- Build parameters

---

# Scenario 37

## A newly deployed application suddenly starts consuming all available database connections. How would you investigate?

### Expected Approach

Check:

- Connection pool configuration
- Connection leaks
- Application logs
- Database monitoring
- Recent code changes
- Query execution time
- Idle connections
- Maximum connection limits

---

# Scenario 38

## Amazon EKS nodes are healthy, but new Pods remain in Pending state. Existing Pods continue running normally. What could be the issue?

### Expected Approach

Verify:

- Available CPU and memory
- Node taints
- Tolerations
- Node selectors
- Affinity rules
- Persistent Volume Claims
- Scheduler events
- Cluster Autoscaler status

---

# Scenario 39

## During a production release, users report intermittent failures while the application appears healthy. How would you approach this issue?

### Expected Approach

Investigate:

- Load Balancer health
- Session persistence
- Multiple application replicas
- Pod restarts
- API latency
- Network packet loss
- DNS resolution
- Backend dependency health

---

# Scenario 40

## A security audit reveals that production Kubernetes Secrets are stored as plain YAML files in the Git repository. How would you remediate this?

### Expected Approach

Take immediate action:

- Remove secrets from Git history
- Rotate all exposed credentials
- Move secrets to AWS Secrets Manager or Kubernetes External Secrets
- Encrypt sensitive configuration
- Enable secret scanning
- Review repository access
- Audit credential usage
- Update deployment process

---

# Enterprise Investigation Flow

```text
Incident Detected

↓

Validate Impact

↓

Review Recent Changes

↓

CI/CD Pipeline

↓

Cloud Infrastructure

↓

Kubernetes

↓

Application

↓

Security Controls

↓

Root Cause

↓

Recovery

↓

Validation

↓

Continuous Improvement
```

---

# Enterprise Best Practices

- Validate infrastructure and application health independently during troubleshooting.
- Protect Terraform state with versioning, backups, and state locking.
- Use automated secret management instead of storing sensitive values in Git.
- Investigate runtime security alerts immediately and preserve evidence.
- Verify cloud networking before assuming application failures.
- Ensure CI/CD pipelines always build the intended source revision.
- Monitor database connection pools and application resource consumption.
- Regularly review Kubernetes scheduling constraints and cluster capacity.
- Implement automated health checks, alerting, and rollback strategies.
- Document every production incident and use the findings to strengthen security and operational resilience.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 41

## Your Jenkins pipeline succeeds, Docker image is pushed to Amazon ECR, but ArgoCD doesn't deploy the latest image. How would you investigate?

### Expected Approach

Check:

- GitOps repository updated
- Image tag in manifests
- ArgoCD sync status
- Auto-sync configuration
- Image Pull Policy
- Git commit history
- Repository access
- ArgoCD controller logs

---

# Scenario 42

## Amazon EKS worker nodes suddenly become NotReady after a scheduled maintenance window. What would you investigate?

### Expected Approach

Verify:

- EC2 instance health
- Kubelet service
- Container runtime
- Node IAM Role
- Network connectivity
- Security Groups
- Disk usage
- Cluster events

---

# Scenario 43

## A production deployment fails because the application cannot retrieve secrets from AWS Secrets Manager. How would you troubleshoot?

### Expected Approach

Check:

- IAM Role permissions
- IRSA configuration
- Secret ARN
- Region configuration
- Network connectivity
- SDK configuration
- Secret policy
- Application logs

---

# Scenario 44

## Checkov suddenly reports hundreds of Infrastructure as Code violations after a Terraform module update. How would you handle this?

### Expected Approach

Investigate:

- Module version
- Policy updates
- Terraform code changes
- False positives
- Baseline comparison
- Policy severity
- Review findings
- Prioritize remediation

---

# Scenario 45

## After a production deployment, Kubernetes Pods continuously fail their Liveness Probe while the application appears healthy. What could be the reason?

### Expected Approach

Verify:

- Probe endpoint
- Probe timeout
- Initial delay
- Resource usage
- Application startup time
- Network latency
- Health endpoint implementation
- Recent configuration changes

---

# Scenario 46

## Developers report that GitHub Actions pipelines fail only when deploying to Production, while Development deployments work successfully. How would you investigate?

### Expected Approach

Check:

- Production secrets
- Environment protection rules
- IAM permissions
- Deployment approvals
- Branch protection
- Runner permissions
- Production variables
- Deployment logs

---

# Scenario 47

## An application deployed in Amazon EKS cannot communicate with another application in a different namespace. What would you investigate?

### Expected Approach

Review:

- Kubernetes Services
- DNS resolution
- Network Policies
- Namespace configuration
- Service names
- RBAC
- Application logs
- Cluster networking

---

# Scenario 48

## OWASP ZAP identifies several High-risk vulnerabilities just before production deployment. The release is scheduled within the next hour. How would you proceed?

### Expected Approach

Determine:

- Vulnerability severity
- Exploitability
- Internet exposure
- Business impact
- Available fixes
- Temporary mitigations
- Risk acceptance process
- Deployment decision

---

# Scenario 49

## Prometheus reports increasing API latency, but CPU, memory, and network metrics appear normal. How would you troubleshoot?

### Expected Approach

Investigate:

- Database response time
- External APIs
- Slow queries
- Application logs
- Thread pools
- Connection pools
- Garbage Collection
- Storage latency

---

# Scenario 50

## During a routine audit, you discover that multiple Kubernetes Service Accounts have ClusterAdmin permissions. How would you remediate this?

### Expected Approach

Take the following steps:

- Identify affected workloads
- Review RBAC assignments
- Apply Least Privilege
- Create dedicated Roles
- Create RoleBindings
- Remove unnecessary ClusterRoleBindings
- Test application functionality
- Continuously audit permissions

---

# Enterprise Investigation Flow

```text
Alert

↓

Confirm Issue

↓

Review Recent Changes

↓

CI/CD

↓

Cloud Platform

↓

Kubernetes

↓

Security

↓

Application

↓

Root Cause

↓

Recovery

↓

Validation

↓

Lessons Learned
```

---

# Enterprise Best Practices

- Ensure GitOps repositories are the single source of truth for deployments.
- Validate IAM permissions whenever cloud services become inaccessible.
- Review Infrastructure as Code findings before suppressing policy violations.
- Configure health probes to reflect realistic application behaviour.
- Separate deployment permissions between Development and Production environments.
- Apply Kubernetes RBAC using the Principle of Least Privilege.
- Treat High and Critical security findings as release blockers unless an approved exception process exists.
- Use observability data from metrics, logs, and traces together during investigations.
- Regularly audit cloud identities, Kubernetes Service Accounts, and CI/CD credentials.
- Document every incident, remediation, and preventive action to improve operational maturity.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 51

## Jenkins successfully builds the application, but the Docker image is not pushed to Amazon ECR. What would you investigate?

### Expected Approach

Check:

- ECR authentication
- IAM permissions
- Repository existence
- Docker login
- Network connectivity
- Repository policy
- Image tag
- Jenkins credentials

---

# Scenario 52

## A Kubernetes Deployment is successful, but one replica continuously enters CrashLoopBackOff while the others run normally. How would you troubleshoot?

### Expected Approach

Investigate:

- Pod logs
- Previous container logs
- Node health
- Mounted volumes
- ConfigMaps
- Secrets
- Resource limits
- Node-specific issues

---

# Scenario 53

## After a Terraform deployment, an Application Load Balancer is created successfully, but users cannot access the application. What would you check?

### Expected Approach

Verify:

- Listener Rules
- Target Groups
- Security Groups
- Route53 records
- Ingress configuration
- Health checks
- Backend service
- Target registration

---

# Scenario 54

## A developer bypasses the Pull Request process and pushes code directly to the main branch. How would you prevent this from happening again?

### Expected Approach

Implement:

- Protected branches
- Mandatory Pull Requests
- Code review approvals
- Branch protection rules
- RBAC
- Audit logging
- CI validation
- Repository policies

---

# Scenario 55

## A Trivy scan identifies Critical vulnerabilities in the base image used by multiple applications. What actions would you take?

### Expected Approach

Review:

- Vulnerability details
- Available patched image
- Impacted applications
- Image rebuild
- Regression testing
- Redeployment
- SBOM update
- Continuous monitoring

---

# Scenario 56

## Pods cannot pull images from Amazon ECR after new worker nodes are added to the EKS cluster. Existing nodes work correctly. What would you investigate?

### Expected Approach

Check:

- Node IAM Role
- IRSA configuration
- ECR permissions
- Network access
- DNS
- Container runtime
- Bootstrap configuration
- Image pull logs

---

# Scenario 57

## Prometheus reports that application latency increased immediately after a deployment, but Grafana dashboards show normal infrastructure metrics. How would you investigate?

### Expected Approach

Verify:

- Application logs
- Slow API endpoints
- Database queries
- Thread pools
- External services
- Recent code changes
- Distributed request flow
- Error logs

---

# Scenario 58

## During deployment, ArgoCD reports "Sync Failed" because one Kubernetes resource already exists. How would you resolve this?

### Expected Approach

Review:

- Existing resource ownership
- Resource annotations
- Git manifests
- Helm release history
- Manual changes
- Resource conflicts
- ArgoCD logs
- Synchronization strategy

---

# Scenario 59

## Your security team reports unusual outbound traffic from one Kubernetes Pod. What steps would you take?

### Expected Approach

Investigate:

- Pod identity
- Container image
- Network connections
- Application logs
- Falco alerts
- Kubernetes Audit Logs
- Recent deployments
- Possible compromise

Contain the workload if malicious behaviour is confirmed.

---

# Scenario 60

## After deploying a new version, all health checks pass, but customers report that one critical business feature no longer works. How would you investigate?

### Expected Approach

Check:

- Business logic changes
- API responses
- Feature flags
- Database schema changes
- Backend services
- Integration tests
- Application logs
- User workflows

Remember that infrastructure health does not guarantee business functionality.

---

# Enterprise Investigation Flow

```text
Issue Reported

↓

Confirm Scope

↓

Review Recent Changes

↓

CI/CD Pipeline

↓

Cloud Infrastructure

↓

Kubernetes

↓

Application

↓

Security

↓

Business Validation

↓

Root Cause

↓

Recovery

↓

Continuous Improvement
```

---

# Enterprise Best Practices

- Validate both technical health and business functionality after every deployment.
- Protect production branches with mandatory reviews and approval policies.
- Keep container base images updated and rebuild dependent applications regularly.
- Verify IAM and node permissions whenever new infrastructure is introduced.
- Correlate application metrics with logs to identify performance bottlenecks.
- Resolve GitOps drift instead of making manual changes in the cluster.
- Investigate unusual network activity immediately and preserve evidence.
- Include smoke tests and business validation tests in deployment pipelines.
- Automate security, compliance, and operational checks throughout the SDLC.
- Perform root cause analysis after every production issue and implement preventive controls.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 61

## A Jenkins pipeline fails during the Terraform Apply stage with a "State Lock" error. Another engineer claims no deployment is running. How would you investigate?

### Expected Approach

Check:

- Active Terraform jobs
- State locking mechanism
- Backend configuration
- Previous failed pipelines
- Stale lock information
- CI/CD concurrency
- Manual Terraform executions
- Lock release procedure

---

# Scenario 62

## After a successful deployment, some users see the new application version while others still see the old version. What would you investigate?

### Expected Approach

Verify:

- Load Balancer routing
- Rolling update status
- ReplicaSet versions
- Browser caching
- CDN cache
- DNS propagation
- Sticky sessions
- Pod versions

---

# Scenario 63

## Kubernetes Pods suddenly start failing with "No space left on device." How would you troubleshoot?

### Expected Approach

Check:

- Node disk usage
- Container logs
- Image cache
- EmptyDir volumes
- Persistent Volumes
- Container runtime storage
- Log rotation
- Node cleanup policies

---

# Scenario 64

## A developer reports that a ConfigMap update is visible in Kubernetes, but the application continues using the old configuration. What would you check?

### Expected Approach

Verify:

- ConfigMap mounted correctly
- Environment variables
- Volume mounts
- Pod restart
- Deployment rollout
- Application reload capability
- Sidecar reload mechanism
- Configuration cache

---

# Scenario 65

## During a production deployment, the Horizontal Pod Autoscaler suddenly scales the application from 10 Pods to 100 Pods. How would you investigate?

### Expected Approach

Review:

- CPU metrics
- Memory metrics
- Custom metrics
- Metrics Server
- Recent traffic spike
- Application behaviour
- HPA thresholds
- Autoscaler events

---

# Scenario 66

## GitHub Actions successfully deploys the application, but ArgoCD never detects the Git changes. What would you investigate?

### Expected Approach

Check:

- Git commit
- Target branch
- GitOps repository
- Repository permissions
- Webhook configuration
- ArgoCD repository connection
- Sync policy
- Application revision

---

# Scenario 67

## During a security audit, you discover that multiple Docker images are using the `latest` tag in Production. Why is this a problem, and how would you fix it?

### Expected Approach

Review:

- Image versioning
- Deployment manifests
- CI/CD tagging strategy
- Image digest usage
- Rollback capability
- GitOps manifests
- Release process
- Image promotion workflow

---

# Scenario 68

## An application deployed on Amazon EKS suddenly loses connectivity to Amazon S3 after an IAM policy update. How would you troubleshoot?

### Expected Approach

Verify:

- IAM Role
- IAM policy changes
- IRSA configuration
- Bucket policy
- AWS Region
- VPC Endpoint
- DNS resolution
- Application logs

---

# Scenario 69

## A new Jenkins agent has been added, but every pipeline executed on that agent fails immediately. What would you investigate?

### Expected Approach

Check:

- Agent connectivity
- Java version
- Jenkins agent configuration
- Workspace permissions
- Required tools
- Network access
- Docker availability
- Pipeline labels

---

# Scenario 70

## Your monitoring system reports a sudden increase in HTTP 404 errors immediately after deployment. Infrastructure appears healthy. How would you investigate?

### Expected Approach

Verify:

- Application routes
- API version changes
- Ingress paths
- Load Balancer rules
- Frontend configuration
- Backend endpoints
- Recent code changes
- Deployment manifests

---

# Enterprise Investigation Flow

```text
Alert Generated

↓

Confirm Impact

↓

Review Recent Changes

↓

CI/CD Pipeline

↓

Cloud Infrastructure

↓

Kubernetes

↓

Application

↓

Dependencies

↓

Root Cause

↓

Recovery

↓

Validation

↓

Post-Incident Review
```

---

# Enterprise Best Practices

- Protect Terraform state with proper locking and avoid concurrent deployments.
- Use immutable image tags instead of `latest` for predictable deployments.
- Monitor node storage and configure automated cleanup to prevent disk exhaustion.
- Restart or reload workloads when configuration changes require application refresh.
- Review autoscaling events regularly to detect abnormal scaling behaviour.
- Ensure GitOps repositories remain the single source of truth for deployments.
- Validate IAM policy changes in lower environments before Production rollout.
- Standardize Jenkins agent configuration to maintain consistent pipeline execution.
- Include functional validation after deployment in addition to infrastructure health checks.
- Capture lessons learned from every incident and update automation, monitoring, and documentation accordingly.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 71

## A deployment is successful, but Prometheus shows a sudden spike in 5xx errors while CPU and Memory remain normal. How would you investigate?

### Expected Approach

Check:

- Application logs
- Recent code changes
- Database connectivity
- API dependencies
- Load Balancer logs
- Pod events
- Error stack traces
- Request patterns

---

# Scenario 72

## During deployment, Jenkins fails while pushing artifacts to JFrog Artifactory. What would you investigate?

### Expected Approach

Verify:

- Artifactory availability
- Repository permissions
- Jenkins credentials
- Network connectivity
- Repository quota
- Artifact version conflicts
- SSL certificates
- Upload logs

---

# Scenario 73

## A production EKS cluster suddenly reports that multiple Pods cannot resolve internal service names. How would you troubleshoot?

### Expected Approach

Check:

- CoreDNS Pods
- CoreDNS logs
- Service records
- DNS configuration
- Network Policies
- Cluster networking
- Node connectivity
- kube-dns service

---

# Scenario 74

## After deploying a new version, users report intermittent authentication failures. Some users can log in while others cannot. What would you investigate?

### Expected Approach

Review:

- Authentication service logs
- Session management
- JWT configuration
- Load Balancer stickiness
- Database replication
- Redis cache
- Token expiration
- Clock synchronization

---

# Scenario 75

## Your Terraform pipeline reports that several AWS resources will be destroyed unexpectedly. What steps would you take before approving the deployment?

### Expected Approach

Review:

- Terraform plan
- State file
- Recent code changes
- Workspace selection
- Resource dependencies
- Import status
- Manual infrastructure changes
- Team approval

Never apply destructive changes without understanding why Terraform generated them.

---

# Scenario 76

## Falco detects execution of `/bin/bash` inside a production container during business hours. How would you respond?

### Expected Approach

Investigate:

- User identity
- Kubernetes Audit Logs
- Pod owner
- Container logs
- Recent deployments
- Interactive sessions
- RBAC permissions
- Possible compromise

Isolate the workload if unauthorized access is confirmed.

---

# Scenario 77

## A GitHub Pull Request passes all automated checks, but the Security team rejects the deployment during review. What could be the reasons?

### Expected Approach

Verify:

- Architecture compliance
- Business security policies
- Manual penetration testing findings
- Data protection requirements
- Compliance requirements
- Risk assessment
- Third-party integrations
- Security exceptions

---

# Scenario 78

## After updating a Kubernetes Ingress resource, all APIs start returning HTTP 404. What would you investigate?

### Expected Approach

Check:

- Ingress rules
- Host configuration
- Path configuration
- Backend Service
- Service port
- Ingress Controller logs
- DNS records
- ALB listener configuration

---

# Scenario 79

## During a production release, Amazon RDS CPU utilization reaches 100%, causing application failures. How would you troubleshoot?

### Expected Approach

Review:

- Slow queries
- Database locks
- Connection pool
- Application traffic
- Recent schema changes
- Index usage
- Query execution plans
- Database monitoring

---

# Scenario 80

## A deployment completed successfully, but ArgoCD reports the application as Degraded. What would you investigate?

### Expected Approach

Check:

- Pod health
- ReplicaSet status
- Deployment rollout
- Kubernetes events
- Resource limits
- Failed hooks
- Health checks
- ArgoCD application events

---

# Enterprise Investigation Flow

```text
Alert

↓

Identify Scope

↓

Review Deployment

↓

CI/CD Pipeline

↓

Cloud Platform

↓

Kubernetes

↓

Application

↓

Dependencies

↓

Root Cause

↓

Recovery

↓

Validation

↓

Documentation
```

---

# Enterprise Best Practices

- Correlate monitoring alerts with deployment timelines before making changes.
- Validate artifact repository connectivity as part of CI/CD health checks.
- Continuously monitor CoreDNS and cluster networking in Kubernetes.
- Design authentication systems to handle distributed deployments consistently.
- Review every Terraform execution plan before applying infrastructure changes.
- Treat unexpected shell access in production containers as a potential security incident.
- Combine automated security checks with manual reviews for high-risk releases.
- Test Ingress and routing changes in non-production environments before deployment.
- Monitor database performance continuously and optimize queries proactively.
- Investigate degraded GitOps applications immediately to prevent production impact.

---
