# Final-DevOps-Mock-Interview

# Final Senior DevOps / DevSecOps Production Mock Interview

## 1. Purpose

This final mock interview is designed as a production-oriented simulation for a Senior DevOps / DevSecOps engineer working with AWS, Terraform, EKS, Kubernetes, Helm, CI/CD, GitOps, Argo CD, ALB, Prometheus, Grafana, ELK, Linux, networking, security, reliability, and incident management.

The interview deliberately focuses on:

- Production architecture
- Real operational decisions
- Troubleshooting under pressure
- Failure analysis
- Security
- Scalability
- High availability
- Disaster recovery
- Cost optimization
- CI/CD
- GitOps
- Kubernetes
- AWS
- Observability
- Incident response
- Senior-level ownership

The expected answer style is not a memorized definition. A strong senior answer should explain:

1. What is happening
2. Why the design exists
3. How it works
4. How it is implemented
5. How it fails
6. How it is monitored
7. How it is secured
8. How it is recovered
9. What trade-offs were considered

---

# 2. Mock Interview Rules

## Interviewer Expectations

The interviewer will frequently ask:

- Why?
- How?
- What happens if it fails?
- How do you troubleshoot it?
- How do you monitor it?
- How do you secure it?
- How do you recover it?
- What would you change at scale?
- What would you do first in production?

Do not answer only with commands.

A senior engineer explains the reasoning behind the commands.

## Candidate Answer Framework

For architecture questions:

> Requirement → Design → Components → Data flow → Security → HA → Observability → Failure handling → Trade-offs

For troubleshooting:

> Symptom → Scope → Evidence → Hypothesis → Validation → Root cause → Fix → Prevention

For incidents:

> Detect → Triage → Mitigate → Communicate → Recover → Validate → RCA → Prevent recurrence

---

# 3. Round 1 — Professional Introduction

## Question 1: Tell me about yourself.

### Strong production-style answer

I have 4+ years of DevOps experience, including around 2 years working with production environments. My primary experience is around AWS, Linux, Git, Terraform, Docker, Kubernetes, EKS, Helm, CI/CD, GitOps, Argo CD, monitoring, logging, and security automation.

In production-oriented environments, I have worked on infrastructure provisioning using Terraform, containerized application deployment on Kubernetes, CI/CD pipelines for build and validation, container image management through ECR, GitOps-based deployment using Argo CD, AWS ALB-based ingress, and observability using Prometheus, Grafana, and ELK.

My approach is not just to deploy applications. I focus on the complete operational lifecycle: secure infrastructure, repeatable deployments, monitoring, alerting, troubleshooting, rollback, disaster recovery, and cost optimization.

When production issues occur, I start by establishing the blast radius and collecting evidence before making changes. I try to restore service quickly while preserving enough information for root-cause analysis.

---

## Question 2: Describe your production environment.

### Strong answer

The production environment is based on AWS with a VPC spanning multiple Availability Zones.

Terraform manages infrastructure such as:

- VPC
- Public and private subnets
- Route tables
- NAT gateways
- Security groups
- IAM
- EKS
- ECR
- Supporting AWS resources

EKS hosts the microservices application.

Application traffic enters through an AWS Application Load Balancer managed through Kubernetes ALB Ingress.

The deployment flow is:

Developer → Git → CI → build/test/security scans → container image → ECR → GitOps repository → Argo CD → EKS.

For observability:

- Prometheus handles metrics and alert evaluation.
- Grafana provides visualization and dashboards.
- ELK provides centralized logging.
- Alertmanager handles alert routing, grouping, silencing, and escalation.

The architecture is designed around repeatability, least privilege, high availability, observability, and controlled deployment.

---

# 4. Round 2 — Architecture

## Question 3: Explain the complete production architecture.

### Strong answer

At the infrastructure layer, AWS provides the network and compute foundation.

The VPC is distributed across multiple Availability Zones. Public subnets host internet-facing components where required, while worker nodes and internal resources are placed in private subnets.

EKS provides the Kubernetes control plane and worker capacity.

Applications run as Kubernetes Deployments and Services.

External application traffic reaches an AWS ALB through Kubernetes ALB Ingress.

The CI pipeline performs:

- Source checkout
- Dependency resolution
- Unit tests
- Static analysis
- Security scanning
- Application packaging
- Container image build
- Image scanning
- Push to ECR

The CI pipeline does not directly become the source of truth for production runtime configuration.

Instead, CI updates the GitOps configuration repository.

Argo CD monitors that repository and reconciles the desired state into EKS.

Prometheus collects metrics from Kubernetes and applications. Alert rules evaluate operational conditions and send alerts to Alertmanager.

Grafana visualizes metrics.

ELK collects and searches centralized application and platform logs.

This creates a separation between:

- Infrastructure provisioning
- Application build
- Deployment configuration
- Runtime reconciliation
- Observability
- Incident response

---

## Question 4: Why use GitOps?

### Strong answer

GitOps gives us Git as the declarative source of truth for the desired Kubernetes state.

Instead of an engineer manually running kubectl against production, deployment configuration is changed through Git.

The workflow becomes:

1. Developer changes application code.
2. CI validates the code.
3. CI builds and scans the image.
4. Image is pushed to ECR.
5. Deployment configuration is updated in Git.
6. Argo CD detects the Git change.
7. Argo CD compares desired state with cluster state.
8. Argo CD applies the required changes.
9. Kubernetes controllers converge the cluster.
10. Monitoring validates the rollout.

Benefits include:

- Auditability
- Reviewable changes
- Reproducibility
- Drift detection
- Controlled promotion
- Easy rollback through Git
- Reduced manual production access

---

# 5. Round 3 — AWS

## Question 5: Why use private subnets for EKS worker nodes?

### Strong answer

Worker nodes generally do not need to accept direct inbound internet traffic.

Putting them in private subnets reduces the attack surface.

Outbound traffic can use NAT gateways where required, while inbound application traffic enters through controlled components such as the ALB.

This creates a topology such as:

Internet
→ ALB
→ Kubernetes Services
→ Pods

while nodes remain private.

Security groups and network policies provide additional controls.

---

## Question 6: What happens if an Availability Zone fails?

### Strong answer

The architecture should avoid concentrating production capacity in a single Availability Zone.

EKS worker capacity should be distributed across multiple AZs.

Application replicas should also be spread across zones using:

- topology spread constraints
- pod anti-affinity
- multiple replicas

The ALB is regional and can route traffic across healthy targets.

If one AZ becomes unavailable, Kubernetes and AWS infrastructure should continue serving traffic from remaining AZs, assuming sufficient capacity exists.

I would validate:

- Node distribution
- Pod distribution
- Subnet availability
- Load balancer target health
- Capacity in remaining AZs
- Stateful workload behavior

---

# 6. Round 4 — Terraform

## Question 7: Why Terraform?

### Strong answer

Terraform provides infrastructure as code.

It allows us to define infrastructure declaratively and manage changes through version-controlled configuration.

For production I would structure Terraform using reusable modules and environment-specific configurations.

Typical layers are:

- Networking
- IAM
- EKS
- ECR
- Supporting AWS services

State should be stored remotely with locking and controlled access.

Production Terraform changes should go through:

1. Pull request
2. Formatting
3. Validation
4. Security checks
5. Terraform plan
6. Review
7. Approval
8. Terraform apply

---

## Question 8: What if Terraform state is locked?

### Strong answer

I would not immediately force-unlock it.

First I would determine:

- Which operation created the lock
- Whether another Terraform process is running
- Whether a CI job is currently active
- Whether the previous operation failed or is still running

Only after confirming that no legitimate operation is using the state would I consider removing a stale lock.

Force-unlocking a state that another process is actively modifying can cause state corruption or concurrent infrastructure changes.

---

## Question 9: Terraform says the resource exists but configuration wants to create it again. What do you check?

### Strong answer

I would check:

1. Terraform state
2. Actual AWS resource
3. Resource ID
4. Provider configuration
5. Account and region
6. Workspace or backend configuration
7. Resource import status
8. State drift

If the resource exists outside Terraform state, I may import it rather than create a duplicate.

---

# 7. Round 5 — EKS and Kubernetes

## Question 10: Explain the difference between Pod, Deployment, ReplicaSet, and Service.

### Strong answer

A Pod is the smallest Kubernetes scheduling unit.

A Deployment manages the desired number and rollout strategy of Pods.

A ReplicaSet is used by a Deployment to maintain the requested number of Pod replicas.

A Service provides stable network access to a group of Pods selected by labels.

For example:

Deployment
→ ReplicaSet
→ Pods

Service
→ selects Pods

Ingress
→ routes external traffic to Services.

---

## Question 11: A Pod is in CrashLoopBackOff. What do you do?

### Strong answer

I would not immediately restart it.

First:

```bash
kubectl get pod -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
```

I would check:

- Container exit code
- Application logs
- Environment variables
- ConfigMaps
- Secrets
- Mounted volumes
- Liveness probe
- Readiness probe
- Resource limits
- Image
- Dependency connectivity

`--previous` is especially important because the current container may have already restarted.

Possible causes include:

- Application startup failure
- Missing configuration
- Invalid secret
- Dependency unavailable
- OOMKilled
- Bad image
- Probe failure

After identifying the root cause, I would fix the underlying issue rather than repeatedly restarting the Pod.

---

## Question 12: What does OOMKilled mean?

### Strong answer

It means the container or process was terminated because it exceeded its available memory limit or the node experienced memory pressure.

I would check:

```bash
kubectl describe pod <pod> -n roboshop
kubectl top pod -n roboshop
kubectl top node
```

I would inspect:

- Memory requests
- Memory limits
- Application memory behavior
- JVM configuration if Java
- Node memory pressure
- Recent deployment changes

The correct fix may be:

- Application memory optimization
- Appropriate container limit
- JVM tuning
- Scaling replicas
- Scaling nodes

I would not simply increase memory without understanding why memory usage increased.

---

# 8. Round 6 — Kubernetes Networking

## Question 13: A Service is not reachable. How do you troubleshoot?

### Strong answer

I would verify the chain:

Pod
→ labels
→ Service selector
→ Endpoints
→ Service port
→ Target port
→ Network policy
→ Ingress if applicable
→ ALB
→ DNS

Commands:

```bash
kubectl get svc -n roboshop
kubectl describe svc <service> -n roboshop
kubectl get endpoints <service> -n roboshop
kubectl get pods --show-labels -n roboshop
```

If endpoints are empty, I would inspect the selector and Pod labels first.

If endpoints exist, I would test connectivity from another Pod.

---

## Question 14: Ingress returns 503. What do you check?

### Strong answer

I would treat 503 as a traffic-path problem rather than assuming the application itself is broken.

I would check:

1. ALB target health
2. Ingress configuration
3. Kubernetes Service
4. Service endpoints
5. Pod readiness
6. Target group configuration
7. Health check path
8. Security groups
9. Network connectivity
10. Application listener

Commands include:

```bash
kubectl describe ingress <ingress> -n roboshop
kubectl get svc -n roboshop
kubectl get endpoints -n roboshop
kubectl get pods -n roboshop
```

If the ALB has no healthy targets, I would follow the target health chain until finding the first failed dependency.

---

# 9. Round 7 — ALB

## Question 15: Why AWS ALB instead of API Gateway?

### Strong answer

For this capstone, ALB is the selected ingress architecture because the application is a Kubernetes-hosted web and microservices platform requiring HTTP/HTTPS routing directly into EKS.

ALB integrates naturally with Kubernetes through the AWS Load Balancer Controller and supports:

- Host-based routing
- Path-based routing
- TLS termination
- Health checks
- AWS security controls
- Integration with EKS targets

API Gateway is a different architectural choice and is not required for this workload.

---

# 10. Round 8 — CI/CD

## Question 16: Explain your CI/CD pipeline.

### Strong answer

The pipeline is divided into validation, security, packaging, artifact management, and deployment configuration.

Typical stages:

1. Checkout
2. Dependency installation
3. Unit tests
4. Static analysis
5. SonarQube quality analysis
6. Security scanning
7. Build
8. Docker image build
9. Trivy image scan
10. Veracode or application security analysis
11. Push image to ECR
12. Update GitOps deployment configuration
13. Git commit and push
14. Argo CD reconciliation
15. Deployment verification

The exact order can vary, but expensive or security-critical checks should fail the pipeline before production promotion.

---

## Question 17: Should CI directly run kubectl apply against production?

### Strong answer

In a GitOps architecture, I would avoid making direct production `kubectl apply` the normal deployment mechanism.

The GitOps repository is the desired-state source of truth.

CI produces a validated artifact and updates the desired deployment configuration.

Argo CD then reconciles the cluster.

Emergency break-glass access can exist, but it should be controlled, audited, and exceptional.

---

# 11. Round 9 — GitOps and Argo CD

## Question 18: Argo CD says OutOfSync. What do you do?

### Strong answer

I would determine whether the difference is intentional, automatic, or caused by drift.

I would inspect:

- Application status
- Sync status
- Health status
- Diff
- Sync history
- Git revision
- Kubernetes object state

CLI:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

If the cluster was manually changed, Argo CD may correctly report drift.

If the Git configuration is wrong, I would fix Git rather than manually patching the cluster.

If a controller mutates a resource automatically, I would determine whether that difference should be ignored or represented properly in the desired state.

---

## Question 19: Why is drift detection important?

### Strong answer

Without drift detection, the actual production environment can silently diverge from the reviewed configuration.

That creates:

- Unknown state
- Difficult rollback
- Audit problems
- Configuration inconsistency
- Troubleshooting complexity

GitOps makes drift visible and gives operations a controlled path to restore desired state.

---

# 12. Round 10 — Helm

## Question 20: Why Helm?

### Strong answer

Helm packages Kubernetes resources into reusable charts.

It allows us to parameterize environment-specific values such as:

- Replica count
- Image repository
- Image tag
- Resources
- Ingress
- Environment variables
- Autoscaling
- Configuration

A common model is:

Chart templates
+
values-dev.yaml
+
values-qa.yaml
+
values-prod.yaml

This avoids duplicating large amounts of Kubernetes YAML.

---

## Question 21: Helm deployment failed. How do you investigate?

### Strong answer

I would start with:

```bash
helm list -n roboshop
helm status <release> -n roboshop
helm history <release> -n roboshop
helm get values <release> -n roboshop
helm get manifest <release> -n roboshop
```

Then inspect Kubernetes objects:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

I would separate template/rendering failures from runtime failures.

---

# 13. Round 11 — Prometheus

## Question 22: What is Prometheus?

### Strong answer

Prometheus is a metrics monitoring and alerting system.

It collects time-series metrics and stores them with labels.

Prometheus can scrape:

- Kubernetes components
- Exporters
- Application metrics endpoints
- Node metrics
- kube-state-metrics
- Custom metrics

PromQL is used for querying and alert expressions.

Prometheus evaluates alert rules and sends firing alerts to Alertmanager.

---

## Question 23: What is the difference between metrics and logs?

### Strong answer

Metrics are numerical time-series measurements.

Examples:

- CPU utilization
- Request rate
- Error rate
- Latency
- Memory usage

Logs are event records containing detailed contextual information.

Metrics are excellent for detecting that something is wrong.

Logs are often better for understanding why it is wrong.

In this architecture:

- Prometheus = metrics
- Grafana = visualization
- ELK = logs

---

# 14. Round 12 — Alerting

## Question 24: What makes a good production alert?

### Strong answer

A good alert should be:

- Actionable
- Relevant
- Owned
- Clearly described
- Correctly prioritized
- Low-noise
- Linked to a runbook
- Associated with the affected service
- Associated with an environment
- Associated with severity

An alert saying:

> CPU is 80%

may not be actionable.

An alert saying:

> Production checkout service has sustained high CPU and request latency for 10 minutes, causing SLO risk

is much more useful.

---

## Question 25: Explain Alertmanager.

### Strong answer

Prometheus evaluates alert rules.

When an alert fires, Prometheus sends it to Alertmanager.

Alertmanager handles:

- Grouping
- Deduplication
- Routing
- Inhibition
- Silencing
- Notification delivery

For example:

Critical production alerts
→ PagerDuty-style on-call escalation

Warning alerts
→ Slack-style operations channel

Informational alerts
→ dashboard or lower-priority channel.

---

# 15. Round 13 — Golden Signals

## Question 26: What are the four golden signals?

### Strong answer

The four golden signals are:

1. Latency
2. Traffic
3. Errors
4. Saturation

For a microservice platform, I would monitor these per important service and endpoint.

For example:

- Request rate
- 95th percentile latency
- HTTP 5xx rate
- CPU/memory saturation
- Queue saturation where relevant

Golden signals help focus monitoring on user-visible service health rather than collecting every possible metric.

---

# 16. Round 14 — SLI, SLO, SLA

## Question 27: Explain SLI, SLO, and SLA.

### Strong answer

An SLI is a measured indicator of service behavior.

Example:

Successful requests / total valid requests.

An SLO is the internal reliability target.

Example:

> 99.9% successful requests over 30 days.

An SLA is a contractual commitment that may include business consequences.

Alerting should increasingly focus on SLO impact rather than arbitrary infrastructure thresholds.

---

# 17. Round 15 — Logging

## Question 28: Explain ELK in your environment.

### Strong answer

ELK consists of:

- Elasticsearch
- Logstash
- Kibana

Application and infrastructure logs are collected centrally.

Logstash can parse and transform records before indexing them into Elasticsearch.

Kibana provides search, visualization, and investigation capabilities.

During incidents, I correlate:

- Alert timestamp
- Application logs
- Deployment timestamp
- Kubernetes events
- ALB behavior
- Metrics

This correlation is more useful than examining logs in isolation.

---

# 18. Round 16 — Security

## Question 29: How do you secure EKS?

### Strong answer

I would use defense in depth.

At AWS level:

- IAM least privilege
- Security groups
- Private subnets
- Restricted network access
- Encryption
- CloudTrail
- Controlled administrative access

At Kubernetes level:

- RBAC
- Service accounts
- Least privilege
- Network policies
- Pod security controls
- Resource limits
- Image scanning
- Admission controls where appropriate
- Secret management

At CI/CD level:

- SAST
- Dependency scanning
- Container scanning
- Secret scanning
- Image provenance
- Protected branches
- Approval controls

Security should be integrated into the lifecycle rather than treated as a final step.

---

# 19. Round 17 — Secrets

## Question 30: Where should production secrets be stored?

### Strong answer

Secrets should not be committed into Git repositories or embedded in container images.

A production design should use an approved secret management mechanism and tightly control access.

Kubernetes Secrets may be used with appropriate encryption and RBAC, but external secret stores are often preferable for sensitive production credentials.

The key principles are:

- No plaintext secrets in Git
- Least privilege
- Rotation
- Auditability
- Encryption
- Short-lived credentials where possible

---

# 20. Round 18 — Production Troubleshooting

## Question 31: Users report that the application is slow. What do you do?

### Strong answer

I first establish scope.

Questions:

- Is all traffic affected?
- One endpoint?
- One service?
- One region?
- One AZ?
- All users?
- Specific users?
- When did it start?

Then I inspect the golden signals.

Metrics:

```promql
rate(http_requests_total[5m])
```

Latency:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

Errors:

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

Then inspect:

- CPU
- Memory
- Pod restarts
- Network
- Dependencies
- Database latency
- ALB metrics
- Logs
- Recent deployments

I correlate the beginning of the problem with recent changes.

---

# 21. Question 32: A deployment completed but users receive errors.

### Strong answer

I would compare:

- Current version
- Previous version
- Error rate
- Pod readiness
- Application logs
- ALB target health
- Dependency connectivity

If the release clearly caused the incident and rollback is safe, I would prioritize restoring service.

I would use the established rollback mechanism rather than manually changing random resources.

After recovery, I would preserve evidence for RCA.

---

# 22. Question 33: CPU is high but request traffic is normal.

### Strong answer

I would not automatically scale out.

I would check:

- Which Pods consume CPU
- Per-container metrics
- Application profiling if available
- Recent code deployment
- Background jobs
- Retry loops
- Dependency failures
- CPU throttling
- Node contention

For Kubernetes:

```bash
kubectl top pods -n roboshop
kubectl top nodes
```

If traffic has not increased, an application behavior change or inefficient code path is a stronger hypothesis than simple traffic growth.

---

# 23. Question 34: Disk usage reaches 95% on a Linux server.

### Strong answer

First determine where the disk is consumed:

```bash
df -h
df -i
du -xhd1 /var | sort -h
```

I would inspect:

- Logs
- Container runtime storage
- Deleted-but-open files
- Temporary files
- Package cache
- Application data

For deleted-but-open files:

```bash
lsof +L1
```

I would not simply delete random files from production.

The permanent fix might involve:

- Log rotation
- Retention policy
- Storage expansion
- Application cleanup
- Centralized logging
- Monitoring thresholds

---

# 24. Round 19 — Incident Response

## Question 35: Production is down. What are your first five actions?

### Strong answer

1. Confirm the incident and scope.
2. Identify whether the issue is still actively causing impact.
3. Start incident communication and assign ownership.
4. Stabilize or mitigate the service.
5. Collect evidence while restoring availability.

I avoid making many unrelated changes simultaneously.

The priority is:

> Protect customers → stabilize service → understand cause → recover → prevent recurrence.

---

## Question 36: What is the difference between mitigation and root-cause fixing?

### Strong answer

Mitigation reduces or removes customer impact quickly.

Root-cause fixing addresses the underlying reason the incident happened.

For example:

If a deployment causes errors, rollback may be the mitigation.

Fixing the application defect is the root-cause solution.

A mature incident process separates the two because restoring service should not wait for a complete root-cause investigation.

---

# 25. Round 20 — Rollback

## Question 37: How do you rollback a Kubernetes deployment?

### Strong answer

For a Deployment:

```bash
kubectl rollout history deployment/checkout -n roboshop
kubectl rollout undo deployment/checkout -n roboshop
kubectl rollout status deployment/checkout -n roboshop
```

For Helm:

```bash
helm history checkout -n roboshop
helm rollback checkout <REVISION> -n roboshop
```

For GitOps, the preferred rollback should generally be represented in Git so that desired state remains consistent.

---

# 26. Round 21 — Disaster Recovery

## Question 38: Explain RPO and RTO.

### Strong answer

RPO is Recovery Point Objective.

It answers:

> How much data loss can the business tolerate?

RTO is Recovery Time Objective.

It answers:

> How long can the service be unavailable?

The DR design must be based on business requirements rather than selecting technologies first.

---

## Question 39: Is multi-AZ the same as disaster recovery?

### Strong answer

No.

Multi-AZ primarily improves availability against an Availability Zone failure.

Disaster recovery addresses larger failure scenarios such as:

- Region failure
- Major infrastructure failure
- Data corruption
- Account-level problems
- Operational mistakes

A multi-AZ architecture can be highly available without being a complete multi-region DR strategy.

---

# 27. Round 22 — Backup and Restore

## Question 40: What should be backed up?

### Strong answer

The backup scope depends on what can be recreated and what cannot.

Typical production backup areas include:

- Databases
- Persistent application data
- Critical configuration
- Terraform state
- Git repositories
- Required artifact metadata
- Observability data where required
- Disaster recovery configuration

Kubernetes manifests managed in Git may be recreated, but persistent application data still requires appropriate backups.

---

# 28. Round 23 — Cost Optimization

## Question 41: AWS bill suddenly increases. What do you check?

### Strong answer

I would identify the cost driver before changing infrastructure.

I would examine:

- Cost by service
- Cost by account
- Cost by environment
- EKS compute
- NAT gateway usage
- Load balancers
- EBS
- Data transfer
- Logging
- Elasticsearch
- Idle resources
- Overprovisioned nodes

Then correlate the increase with:

- Deployment
- Traffic growth
- Scaling changes
- Logging changes
- New environments

Cost optimization should not blindly reduce resources if doing so damages reliability.

---

# 29. Round 24 — Linux

## Question 42: CPU is 100%. What commands do you use?

### Strong answer

```bash
top
htop
ps aux --sort=-%cpu | head
uptime
vmstat 1
mpstat -P ALL 1
```

I distinguish:

- User CPU
- System CPU
- I/O wait
- Steal time
- Load average

High load does not automatically mean CPU saturation.

I also determine whether the issue is host-level or caused by a particular process.

---

## Question 43: How do you investigate memory pressure?

### Strong answer

```bash
free -h
vmstat 1
top
ps aux --sort=-%mem | head
dmesg | grep -i oom
```

I inspect swap activity and kernel OOM messages.

In Kubernetes I correlate node memory pressure with Pod requests, limits, and OOMKilled events.

---

# 30. Round 25 — Networking

## Question 44: How do you troubleshoot DNS?

### Strong answer

I identify whether the failure is:

- Client-side
- DNS resolver
- Kubernetes CoreDNS
- Route 53
- External DNS
- Application configuration

Commands:

```bash
dig example.com
nslookup example.com
getent hosts example.com
```

In Kubernetes:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

I verify both name resolution and subsequent network connectivity.

---

# 31. Round 26 — IAM

## Question 45: Terraform receives AccessDenied. What do you check?

### Strong answer

I verify:

- Which identity is actually being used
- IAM policy
- Resource policy
- Trust policy
- Permission boundaries
- SCPs
- Session policies
- Region/account
- Explicit denies

I use AWS identity information to confirm the executing principal before modifying permissions.

The principle is:

> Grant the minimum permission required for the operation.

---

# 32. Round 27 — Production Scenario

## Question 46: A production deployment causes 30% HTTP 5xx errors. Walk me through your response.

### Strong answer

First I confirm the alert and impact using metrics.

I inspect:

- Error rate
- Request volume
- Affected endpoint
- Pod health
- Deployment version
- ALB target health
- Application logs
- Dependency errors

Then I compare the incident start time with the deployment.

If there is strong evidence that the release caused the failure, I initiate rollback according to the incident procedure.

After rollback:

```bash
kubectl rollout status deployment/checkout -n roboshop
```

I validate:

- Error rate returned to baseline
- Latency recovered
- Pods healthy
- ALB targets healthy
- Business transactions working

Then I document the incident and start RCA.

---

# 33. Production Scenario — Argo CD Drift

## Question 47: An engineer manually changed production and Argo CD reports OutOfSync.

### Answer

I would determine why the manual change was made.

If it was an emergency mitigation, I would document it and reproduce the intended configuration in Git.

If it was unauthorized, I would revert the drift through the GitOps workflow.

The important principle is:

> The emergency action may be necessary, but Git must eventually become the source of truth again.

---

# 34. Production Scenario — Node Failure

## Question 48: An EKS node suddenly disappears.

### Answer

I would check:

```bash
kubectl get nodes
kubectl describe node <node>
kubectl get pods -A -o wide
```

Then determine:

- Whether Pods were rescheduled
- Whether sufficient capacity exists
- Whether disruption budgets prevent movement
- Whether stateful workloads are affected
- Whether the node belongs to an autoscaling group or managed node group

I would inspect AWS infrastructure and node lifecycle events.

If workload capacity is insufficient, scaling may be required.

---

# 35. Production Scenario — Database Dependency Failure

## Question 49: The application Pods are healthy but requests fail because the database is unavailable.

### Answer

Healthy Pods do not prove healthy application service.

I would inspect:

- Application error logs
- Connection pool metrics
- Database endpoint
- DNS
- Security groups
- Network path
- Database health
- Connection limits
- Timeouts

I would also verify whether retries are creating a retry storm.

A mature application should fail gracefully rather than consuming all resources while a dependency is unavailable.

---

# 36. Production Scenario — Alert Storm

## Question 50: Hundreds of alerts fire simultaneously. What do you do?

### Answer

I first identify whether they share a common root cause.

For example:

Node failure
→ Pods unavailable
→ Service endpoints reduced
→ Application errors
→ Latency increase
→ SLO alerts

The root cause may be one infrastructure failure rather than hundreds of independent incidents.

Alertmanager grouping and inhibition should reduce this noise.

I would also review whether alert dependencies and parent-child relationships are represented correctly.

---

# 37. Senior Design Questions

## Question 51: How would you design production for high availability?

### Strong answer

I would design HA across multiple failure domains.

Infrastructure:

- Multiple AZs
- Private subnets
- Redundant worker capacity
- Regional ALB

Kubernetes:

- Multiple replicas
- Pod distribution
- Pod disruption budgets
- Health probes
- Autoscaling

Application:

- Stateless services where possible
- Externalized state
- Graceful shutdown
- Timeouts
- Retries with backoff

Operations:

- Monitoring
- Alerting
- Automated rollback
- Tested backups
- DR procedures

---

# 38. Question 52: How would you scale this architecture?

### Answer

I would identify the actual bottleneck before scaling.

Possible scaling layers:

- ALB
- Kubernetes Pods
- Nodes
- Database
- Cache
- Queue
- Observability stack

For Kubernetes:

- HPA for Pod scaling
- Cluster/node autoscaling for capacity
- Resource requests and limits
- Topology-aware scheduling

For large workloads I would also examine:

- Application concurrency
- Database connection limits
- Network throughput
- API rate limits
- Logging volume

---

# 39. Question 53: What would you improve in this capstone?

### Strong answer

I would prioritize improvements based on risk and business value.

Potential improvements include:

- Stronger SLO-based alerting
- Automated deployment verification
- Progressive delivery
- More comprehensive policy-as-code
- Better secret rotation
- Automated DR testing
- Stronger supply-chain security
- Cost allocation by service/team
- Capacity forecasting
- Better dependency dashboards

I would not introduce technology simply because it is popular.

Every additional component creates operational cost.

---

# 40. Behavioral Round

## Question 54: Tell me about a difficult production incident.

### Answer framework

Use:

Situation → Impact → Investigation → Action → Recovery → Root cause → Prevention.

Example:

A deployment caused elevated error rates.

I confirmed the scope through Prometheus and application logs, correlated the beginning of the errors with the deployment, and compared the new version against the previous release.

We rolled back to restore service, validated recovery through application metrics and user-facing checks, and then investigated the failed release.

The root cause was identified and the CI validation and deployment checks were strengthened to prevent recurrence.

---

# 41. Question 55: Have you ever disagreed with another engineer?

### Strong answer

I focus the discussion on evidence rather than personal preference.

For production architecture I would compare:

- Reliability
- Security
- Cost
- Complexity
- Operational burden
- Recovery characteristics

If both approaches are viable, I prefer documenting the trade-offs and choosing based on the system requirements.

---

# 42. Question 56: What do you do when you don't know something?

### Strong answer

I do not guess in production.

I identify what I know, what I do not know, and what evidence can reduce the uncertainty.

I use documentation, logs, metrics, configuration, controlled testing, and experienced teammates when appropriate.

The important part is maintaining safe operational behavior while learning quickly.

---

# 43. Rapid-Fire Round

## Question 57: What is immutable infrastructure?

Infrastructure is replaced rather than manually modified in place whenever practical.

## Question 58: What is idempotency?

Running the same operation repeatedly produces the same intended end state.

## Question 59: What is configuration drift?

Actual infrastructure differs from the declared desired configuration.

## Question 60: What is a readiness probe?

It determines whether a Pod should receive traffic.

## Question 61: What is a liveness probe?

It determines whether a container should be restarted.

## Question 62: What is a startup probe?

It protects slow-starting applications from premature liveness failures.

## Question 63: What is a PodDisruptionBudget?

It limits voluntary disruption to maintain application availability.

## Question 64: What is HPA?

Horizontal Pod Autoscaler adjusts Pod replicas based on metrics.

## Question 65: What is a StatefulSet?

A controller designed for stateful workloads requiring stable identity and storage characteristics.

## Question 66: What is a DaemonSet?

A controller that runs a Pod on selected nodes, commonly for agents such as log collectors.

## Question 67: What is a ConfigMap?

A Kubernetes resource for non-sensitive configuration.

## Question 68: What is a Secret?

A Kubernetes object intended for sensitive configuration, subject to appropriate encryption and access controls.

## Question 69: What is RBAC?

Role-Based Access Control.

## Question 70: What is a ServiceAccount?

An identity used by workloads inside Kubernetes.

## Question 71: What is ECR?

AWS Elastic Container Registry.

## Question 72: What is EKS?

AWS managed Kubernetes.

## Question 73: What is ALB?

AWS Application Load Balancer.

## Question 74: What is PromQL?

Prometheus Query Language.

## Question 75: What is Alertmanager?

A service that routes and manages Prometheus alerts.

## Question 76: What is GitOps?

Operating infrastructure and applications using Git as the declarative source of truth.

## Question 77: What is Argo CD?

A GitOps continuous delivery controller for Kubernetes.

## Question 78: What is Helm?

A package manager and templating system for Kubernetes applications.

## Question 79: What is Terraform?

Infrastructure-as-code tooling.

## Question 80: What is RPO?

Maximum acceptable data-loss window.

## Question 81: What is RTO?

Maximum acceptable recovery time.

---

# 44. Command-Based Interview Round

## Question 82: Show how you check Kubernetes workload health.

```bash
kubectl get pods -A
kubectl get deployments -A
kubectl get statefulsets -A
kubectl get daemonsets -A
kubectl get events -A --sort-by=.lastTimestamp
```

## Question 83: Show how you inspect a failing Pod.

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

## Question 84: Show deployment rollout status.

```bash
kubectl rollout status deployment/<deployment> -n <namespace>
```

## Question 85: Show rollout history.

```bash
kubectl rollout history deployment/<deployment> -n <namespace>
```

## Question 86: Show Service endpoints.

```bash
kubectl get endpoints <service> -n <namespace>
```

## Question 87: Show node resource usage.

```bash
kubectl top nodes
```

## Question 88: Show Pod resource usage.

```bash
kubectl top pods -A
```

## Question 89: Validate a Helm chart.

```bash
helm lint ./chart
helm template checkout ./chart -f values-prod.yaml
```

## Question 90: Inspect Helm history.

```bash
helm history checkout -n roboshop
```

## Question 91: Check Terraform configuration.

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan
```

---

# 45. Prometheus Interview Round

## Question 92: Write a basic CPU alert expression.

```promql
100 *
(1 - avg by (instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
)) > 80
```

The threshold should be tuned to the workload and should normally include a sustained duration.

---

## Question 93: How do you alert on HTTP 5xx rate?

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
> 0.05
```

This should generally be combined with a `for` duration and service-specific labels.

---

## Question 94: How do you alert on container restarts?

```promql
increase(kube_pod_container_status_restarts_total[15m]) > 3
```

The exact threshold depends on the workload.

---

# 46. Production PrometheusRule Example

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-production-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
    environment: prod
    team: platform
spec:
  groups:
    - name: roboshop.application
      rules:
        - alert: RoboshopHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="roboshop",
                status=~"5.."
              }[5m]))
              /
              sum(rate(http_requests_total{
                namespace="roboshop"
              }[5m]))
            ) > 0.05
          for: 10m
          labels:
            severity: critical
            environment: prod
            team: application
            service: roboshop
          annotations:
            summary: "High application error rate"
            description: "The RoboShop application has sustained HTTP 5xx errors above 5%."
            runbook_url: "https://runbooks.example.internal/roboshop/high-error-rate"
```

Production considerations:

- Use stable labels.
- Keep ownership explicit.
- Avoid high-cardinality labels.
- Use a meaningful `for` period.
- Include a runbook.
- Make the alert actionable.
- Tune thresholds using historical behavior.

---

# 47. Alertmanager Interview Round

## Question 95: Explain alert routing.

A production routing tree can use:

- Severity
- Team
- Environment
- Service

Example concept:

```text
critical + prod
        |
        +--> platform team
        |
        +--> on-call escalation

warning + prod
        |
        +--> operations channel

non-prod
        |
        +--> development channel
```

---

## Question 96: Why grouping?

Grouping prevents dozens of related alerts from generating dozens of notifications.

For example, one node failure may trigger many downstream alerts.

Grouping can create one meaningful notification representing the incident.

---

## Question 97: What is inhibition?

Inhibition suppresses dependent alerts when a higher-level alert is already firing.

For example:

```text
NodeDown
   |
   +--> inhibit PodNotReady
   +--> inhibit ContainerRestarting
```

This reduces alert noise during known parent failures.

---

# 48. Production Alertmanager Example

```yaml
global:
  resolve_timeout: 5m

route:
  group_by:
    - alertname
    - cluster
    - environment
    - team
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  receiver: default

  routes:
    - matchers:
        - severity="critical"
        - environment="prod"
      receiver: prod-critical
      continue: false

    - matchers:
        - severity="warning"
        - environment="prod"
      receiver: prod-warning
      continue: false

    - matchers:
        - environment=~"dev|qa"
      receiver: nonprod
      continue: false

receivers:
  - name: default

  - name: prod-critical
    webhook_configs:
      - url: "https://incident.example.internal/webhook"
        send_resolved: true

  - name: prod-warning
    webhook_configs:
      - url: "https://chat.example.internal/webhook"
        send_resolved: true

  - name: nonprod
    webhook_configs:
      - url: "https://chat.example.internal/nonprod-webhook"
        send_resolved: true

inhibit_rules:
  - source_matchers:
      - alertname="KubernetesNodeDown"
      - severity="critical"
    target_matchers:
      - severity=~"warning|critical"
    equal:
      - cluster
      - environment
      - node
```

URLs above are placeholders.

Credentials or webhook secrets should not be hard-coded.

---

# 49. Interview Question — Alert Noise

## Question 98: How do you reduce alert fatigue?

### Strong answer

I would:

1. Remove non-actionable alerts.
2. Tune thresholds from historical data.
3. Add `for` durations.
4. Group related alerts.
5. Use inhibition.
6. Route by ownership.
7. Separate warning from critical.
8. Use SLO-based alerts where possible.
9. Review alert frequency.
10. Treat repeated false positives as engineering problems.

An alert that is repeatedly ignored is effectively a failed monitoring system.

---

# 50. Production Incident Simulation 1

## Scenario

At 10:15 AM, production checkout latency increases sharply.

Prometheus fires:

```text
CheckoutHighLatency
CheckoutSLORisk
```

### Candidate response

I acknowledge the incident and establish impact.

I check:

```text
Traffic
Latency
Errors
Saturation
```

Then:

- ALB metrics
- Checkout Pods
- CPU
- Memory
- Restarts
- Logs
- Database latency
- Recent deployment
- Dependency health

If the incident began immediately after a deployment, I compare versions.

If the database is slow, I investigate the database dependency rather than scaling application Pods blindly.

If customer impact is severe and the latest release is implicated, rollback is considered.

---

# 51. Production Incident Simulation 2

## Scenario

All Pods in one service are Pending.

### Investigation

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl get nodes
kubectl describe nodes
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Look for:

- Insufficient CPU
- Insufficient memory
- Taints
- Node selectors
- Affinity rules
- Resource requests
- PVC issues

Then inspect cluster capacity and autoscaling.

---

# 52. Production Incident Simulation 3

## Scenario

Pods are Running but users receive 404.

### Investigation chain

```text
DNS
→ ALB
→ Listener
→ Ingress
→ Rule
→ Service
→ Endpoint
→ Application route
```

I verify whether the 404 originates from:

- ALB
- Ingress
- Application

The source of the response matters.

---

# 53. Production Incident Simulation 4

## Scenario

Argo CD continuously changes a resource back after an engineer manually edits it.

### Answer

This is expected GitOps reconciliation behavior if Git remains the desired state.

I would identify the reason for the manual change and update the Git source of truth if the change is legitimate.

If the change was temporary emergency mitigation, I would document it and restore the intended Git-managed state after recovery.

---

# 54. Production Incident Simulation 5

## Scenario

Prometheus stops receiving metrics.

### Investigation

I check:

```bash
kubectl get pods -n monitoring
kubectl get servicemonitors -n monitoring
kubectl get prometheusrules -n monitoring
```

Then inspect Prometheus targets.

Possible causes:

- Scrape target unavailable
- ServiceMonitor selector mismatch
- Network policy
- Authentication/TLS issue
- Prometheus resource pressure
- Target endpoint failure
- Configuration error

The first objective is determining whether Prometheus itself is unhealthy or only particular targets are failing.

---

# 55. Production Incident Simulation 6

## Scenario

ELK disk usage reaches critical levels.

### Answer

I would inspect:

- Index growth
- Retention
- Shards
- Replica configuration
- Ingestion rate
- Large documents
- Application logging changes

The immediate mitigation might be reducing nonessential ingestion or removing data according to an approved retention policy.

The long-term solution may involve:

- Retention management
- Capacity planning
- Index lifecycle policies
- Better log levels
- Storage scaling

---

# 56. Production Incident Simulation 7

## Scenario

A developer asks you to grant cluster-admin because their deployment is failing.

### Answer

I would not grant cluster-admin as the first response.

I would determine exactly which permission is missing.

Then provide the minimum required RBAC permission.

Production troubleshooting should not become an excuse for privilege escalation.

---

# 57. Production Incident Simulation 8

## Scenario

A secret was accidentally committed to Git.

### Answer

I treat the secret as compromised.

I would:

1. Stop further exposure.
2. Rotate or revoke the credential.
3. Remove the secret from the repository history where appropriate.
4. Review access logs.
5. Search for additional exposure.
6. Update secret-management controls.
7. Add secret scanning/prevention.

Removing the text from the latest commit is not sufficient if the credential was already exposed.

---

# 58. Senior Architecture Challenge

## Question 99: Design a zero-downtime deployment strategy.

### Strong answer

I would combine:

- Multiple replicas
- Readiness probes
- Rolling updates
- PodDisruptionBudgets
- Graceful shutdown
- Connection draining
- ALB health checks
- Automated rollout verification
- Fast rollback

For higher-risk services, progressive delivery can be introduced later.

The application itself must support graceful termination.

---

# 59. Senior Architecture Challenge

## Question 100: How would you prevent a bad image from reaching production?

### Answer

Controls should exist at multiple stages:

CI:

- Unit tests
- SAST
- Dependency scanning
- Image scanning
- Quality gates

Artifact:

- Immutable image tags/digests
- ECR access control
- Image provenance

Deployment:

- Approved Git change
- Environment promotion
- Policy validation
- Deployment health checks

Runtime:

- Admission controls where appropriate
- Monitoring
- Automated rollback

No single scanner should be considered sufficient.

---

# 60. Senior Architecture Challenge

## Question 101: How would you handle multiple EKS clusters?

### Answer

Argo CD can operate as a centralized GitOps control plane managing multiple clusters.

A repository can define environment and cluster-specific desired state.

For example:

```text
Git
 |
 +-- dev cluster
 +-- qa cluster
 +-- prod cluster-a
 +-- prod cluster-b
```

Argo CD applications or ApplicationSets can target the appropriate clusters.

Important considerations include:

- Cluster registration
- RBAC
- Network connectivity
- Secret management
- Environment isolation
- Application ownership
- Blast radius

---

# 61. Senior Architecture Challenge

## Question 102: How do you avoid a centralized GitOps control plane becoming a single point of failure?

### Answer

I would distinguish control-plane availability from application availability.

If Argo CD becomes temporarily unavailable, already deployed Kubernetes workloads generally continue running.

The concern is deployment and reconciliation capability.

For critical environments, I would design Argo CD for high availability and protect:

- Repository access
- Argo CD state
- Authentication
- Cluster connectivity

The application should not immediately disappear just because GitOps control is temporarily unavailable.

---

# 62. Senior Architecture Challenge

## Question 103: How do you design observability for a microservices platform?

### Answer

I start with service-level questions.

For each critical service:

- Is it available?
- Is it fast?
- Is it returning errors?
- Is it saturated?
- What dependency is failing?

Prometheus provides metrics.

Grafana provides dashboards.

ELK provides centralized logs.

Alerts are based on actionable conditions and SLO risk.

I avoid building dashboards that display hundreds of metrics without helping operators answer operational questions.

---

# 63. Interview Evaluation Rubric

## Weak Answer

A weak answer:

- Gives only definitions
- Lists tools without explaining relationships
- Uses commands without reasoning
- Ignores failure scenarios
- Ignores security
- Ignores rollback
- Ignores observability
- Says "I will restart the Pod" for every problem

## Intermediate Answer

An intermediate answer:

- Understands the tools
- Can execute common commands
- Understands basic architecture
- Can troubleshoot common failures
- Understands CI/CD

## Senior Answer

A senior answer:

- Starts with impact and scope
- Uses evidence
- Understands system dependencies
- Explains trade-offs
- Designs for failure
- Uses least privilege
- Thinks about HA and DR
- Understands observability
- Can rollback safely
- Protects customer impact
- Communicates during incidents
- Prevents recurrence

---

# 64. Final Capstone Architecture Explanation

## Question 104: Explain the complete capstone in five minutes.

### Model answer

The platform is a production-oriented microservices environment running on AWS EKS.

Terraform provisions the AWS foundation, including VPC networking, IAM, EKS, and ECR.

The application is containerized and packaged with Helm.

Developers commit code to Git.

The CI pipeline checks out the code, builds it, runs tests, performs SonarQube analysis, security scanning through tools such as Trivy and Veracode, builds the container image, and pushes the approved artifact to ECR.

The deployment configuration is maintained separately through GitOps.

CI updates the GitOps repository with the approved image version.

Argo CD watches that repository and reconciles the desired state into EKS.

The applications run as Kubernetes workloads across multiple Availability Zones.

External HTTP/HTTPS traffic enters through AWS ALB using Kubernetes ALB Ingress.

Prometheus collects metrics and evaluates alert rules.

Grafana provides dashboards.

Alertmanager routes alerts based on severity, environment, team, and service.

ELK provides centralized logging.

When an incident occurs, operators use alerts to identify the affected service, then correlate metrics, logs, Kubernetes events, deployment history, and AWS signals.

If a recent deployment caused the problem, the team can roll back through the controlled deployment process.

For disaster recovery, the architecture depends on tested backups, infrastructure-as-code, GitOps configuration, recoverable artifacts, database backups, and documented RPO/RTO objectives.

The overall design emphasizes automation, security, observability, reproducibility, high availability, controlled change, and rapid recovery.

---

# 65. Final Whiteboard Exercise

## Question 105: Draw the architecture.

A strong candidate should be able to draw:

```text
                    Developers
                        |
                        v
                  Git Repository
                        |
                        v
                 CI/CD Pipeline
                        |
        +---------------+----------------+
        |               |                |
        v               v                v
     Testing         Security          Build
                       |                 |
                       +--------+--------+
                                |
                                v
                              ECR
                                |
                                v
                         GitOps Repository
                                |
                                v
                              Argo CD
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
                 EKS AZ-1                EKS AZ-2
                    |                       |
                    +-----------+-----------+
                                |
                                v
                         Kubernetes Services
                                |
                                v
                          ALB Ingress
                                |
                                v
                              Users


EKS
 |
 +--> Prometheus --> Alertmanager --> On-call
 |
 +--> Grafana
 |
 +--> ELK --> Operations
```

The candidate should explain every arrow.

---

# 66. Final Troubleshooting Method

When an interviewer presents an unknown failure, use this sequence:

## Step 1 — Confirm

Is the issue real?

## Step 2 — Scope

Who and what is affected?

## Step 3 — Timeline

When did it start?

## Step 4 — Recent Changes

What changed before the incident?

## Step 5 — Dependencies

What does the affected component depend on?

## Step 6 — Evidence

Collect:

- Metrics
- Logs
- Events
- Configuration
- Deployment history
- AWS signals

## Step 7 — Hypothesis

Form a testable explanation.

## Step 8 — Validate

Use the smallest safe test.

## Step 9 — Mitigate

Restore service.

## Step 10 — Root Cause

Identify why it happened.

## Step 11 — Prevention

Add:

- Monitoring
- Tests
- Automation
- Documentation
- Security control
- Capacity planning

---

# 67. Final Incident Command Structure

For a major production incident:

```text
Incident Commander
        |
        +--> Technical Lead
        |
        +--> Communications Lead
        |
        +--> Application Owner
        |
        +--> Platform/AWS Owner
        |
        +--> Database Owner
```

The exact organization may differ, but separating coordination from deep technical work prevents the person troubleshooting from also becoming overwhelmed by communication and coordination.

---

# 68. Final Interview Questions — No Preparation

Answer these without looking at notes:

1. Design an EKS production environment.
2. Explain why nodes are private.
3. Explain ALB Ingress.
4. Explain Terraform state.
5. Explain GitOps.
6. Explain Argo CD reconciliation.
7. Explain Helm.
8. Troubleshoot CrashLoopBackOff.
9. Troubleshoot Pending Pods.
10. Troubleshoot 503 from ALB.
11. Troubleshoot 404.
12. Troubleshoot DNS.
13. Troubleshoot high CPU.
14. Troubleshoot high memory.
15. Troubleshoot disk pressure.
16. Explain Prometheus.
17. Explain PromQL.
18. Explain Alertmanager.
19. Design alert routing.
20. Reduce alert fatigue.
21. Explain SLI/SLO/SLA.
22. Explain golden signals.
23. Explain ELK.
24. Design centralized logging.
25. Secure EKS.
26. Secure CI/CD.
27. Protect secrets.
28. Handle a compromised credential.
29. Roll back a deployment.
30. Handle Argo CD drift.
31. Handle node failure.
32. Handle AZ failure.
33. Handle database failure.
34. Handle ECR problems.
35. Handle Terraform failure.
36. Design backup.
37. Design DR.
38. Explain RPO/RTO.
39. Reduce AWS cost.
40. Respond to a major outage.

---

# 69. Final Senior-Level Checklist

Before claiming production readiness, the engineer should be able to explain:

## AWS

- VPC
- Subnets
- Routing
- NAT
- Security groups
- IAM
- EKS
- ECR
- ALB
- DNS
- Availability Zones
- Cost controls

## Linux

- CPU
- Memory
- Disk
- Processes
- Networking
- Filesystems
- Logs
- Permissions

## Kubernetes

- Pods
- Deployments
- Services
- Ingress
- ConfigMaps
- Secrets
- RBAC
- Probes
- Requests/limits
- HPA
- PDB
- Scheduling
- Storage
- Networking

## Terraform

- State
- Modules
- Plan
- Apply
- Drift
- Import
- Locking
- Remote backend
- Security

## CI/CD

- Build
- Test
- Quality
- Security
- Image
- Artifact
- Promotion
- Deployment
- Verification
- Rollback

## GitOps

- Git source of truth
- Argo CD
- Sync
- Drift
- Rollback
- Multi-cluster

## Observability

- Metrics
- Prometheus
- PromQL
- Grafana
- Logs
- ELK
- Alerts
- Alertmanager
- SLOs

## Security

- IAM
- RBAC
- Secrets
- Network controls
- Image security
- SAST
- Dependency scanning
- Container scanning
- Auditability

## Reliability

- HA
- Multi-AZ
- Scaling
- Backup
- Restore
- DR
- RPO
- RTO
- Incident response
- Rollback

---

# 70. Final Mock Interview — Closing Question

## Question 106: Why should we hire you for a senior DevOps role?

### Strong answer

I bring a production-oriented approach rather than focusing only on individual tools.

I understand how infrastructure, applications, CI/CD, Kubernetes, GitOps, security, observability, and incident response fit together as one operating system.

I can work from infrastructure provisioning with Terraform through application delivery on EKS, GitOps reconciliation with Argo CD, ingress through ALB, metrics through Prometheus, dashboards through Grafana, and centralized logging through ELK.

More importantly, I understand that production engineering is about managing failure.

I think about:

- What happens when a Pod fails?
- What happens when a node fails?
- What happens when an AZ fails?
- What happens when a deployment is bad?
- What happens when a dependency is unavailable?
- What happens when monitoring fails?
- What happens when credentials are compromised?
- What happens when infrastructure must be rebuilt?

My goal is to build systems that are automated, observable, secure, recoverable, and maintainable.

I also understand that DevOps is not simply deploying faster. It is enabling teams to deliver changes safely while maintaining reliability and business continuity.

---

# 71. Final Self-Assessment

Score yourself from 1–5.

| Area | Score |
|---|---:|
| Linux | /5 |
| Networking | /5 |
| AWS | /5 |
| Terraform | /5 |
| Docker | /5 |
| Kubernetes | /5 |
| EKS | /5 |
| Helm | /5 |
| Jenkins / CI | /5 |
| GitHub Actions | /5 |
| GitOps | /5 |
| Argo CD | /5 |
| ALB | /5 |
| Prometheus | /5 |
| Grafana | /5 |
| ELK | /5 |
| Alerting | /5 |
| Security | /5 |
| Troubleshooting | /5 |
| Incident Response | /5 |
| Disaster Recovery | /5 |
| Cost Optimization | /5 |
| Architecture | /5 |
| Communication | /5 |

Interpretation:

- 1 = Beginner
- 2 = Basic working knowledge
- 3 = Production capable
- 4 = Strong senior capability
- 5 = Can design, operate, troubleshoot, and teach

The goal is not to score 5 everywhere immediately.

The goal is to identify weak areas and close them through hands-on practice.

---

# 72. Final Production Mindset

A senior DevOps engineer should think beyond:

> "How do I deploy this?"

The better questions are:

> "How will I deploy this safely?"

> "How will I know it is healthy?"

> "What happens if it fails?"

> "How quickly can I recover?"

> "Who receives the alert?"

> "Can another engineer operate this system?"

> "Can I reproduce the environment?"

> "Can I roll back?"

> "Can I restore the data?"

> "Can I prove what changed?"

> "Can I secure it?"

> "Can I operate it at 2 AM?"

That mindset is the difference between knowing DevOps tools and operating production systems.

---

# 73. Final Capstone Summary

The complete production capstone demonstrates the lifecycle:

```text
PLAN
 |
 v
ARCHITECT
 |
 v
TERRAFORM
 |
 v
AWS VPC
 |
 v
EKS
 |
 v
APPLICATION
 |
 v
CI
 |
 +--> TEST
 +--> SONARQUBE
 +--> SECURITY SCANS
 +--> BUILD IMAGE
 +--> TRIVY
 +--> VERACODE
 |
 v
ECR
 |
 v
GITOPS
 |
 v
ARGO CD
 |
 v
KUBERNETES
 |
 v
ALB
 |
 v
USERS
 |
 +-----------------------+
 |                       |
 v                       v
PROMETHEUS             ELK
 |                       |
 v                       v
ALERTMANAGER           KIBANA
 |
 v
ON-CALL
 |
 v
INCIDENT RESPONSE
 |
 v
RECOVERY
 |
 v
RCA
 |
 v
PREVENTION
```

This completes the senior-level production DevOps mock interview.

The strongest preparation is to take every scenario above and explain it aloud without reading the answer.

For each scenario, practice:

```text
What happened?
What is the blast radius?
What evidence do I need?
What is my hypothesis?
How do I validate it?
How do I mitigate it?
How do I recover?
How do I prevent recurrence?
```

That is the operational thinking expected from a production DevOps engineer.
