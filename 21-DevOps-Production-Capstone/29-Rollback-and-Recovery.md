# 29. Rollback and Recovery

## 29.1 Purpose

Rollback and recovery are critical production operations used to return a system to a known-good state after a bad deployment, configuration change, infrastructure change, or application failure.

A mature production environment must answer:

```text
How do we stop a bad release?
How do we return to the previous version?
How do we recover if rollback is not safe?
How do we handle database changes?
How do we recover GitOps state?
How do we verify recovery?
```

In this capstone the primary delivery model is:

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Build
   +--> Test
   +--> Security scans
   +--> Container image
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
   v
EKS
```

Rollback must therefore preserve the source-of-truth model wherever possible.

---

# 29.2 Rollback vs Recovery

These terms are related but different.

## Rollback

Return a component to a previous known-good version.

Example:

```text
payment v2.5 → payment v2.4
```

## Recovery

Restore service after failure using whatever safe method is required.

Recovery may involve:

- rollback
- failover
- restore
- replacement
- scaling
- configuration correction
- database recovery
- infrastructure recreation

Therefore:

```text
Rollback is one recovery technique.
```

---

# 29.3 Why Rollback Matters

A deployment can fail because of:

- application bug
- bad configuration
- missing environment variable
- broken image
- dependency incompatibility
- database incompatibility
- incorrect Helm values
- Kubernetes manifest error
- security policy
- network change
- IAM change
- infrastructure change

Fast rollback reduces:

```text
customer impact
downtime
error-budget consumption
incident duration
```

---

# 29.4 Rollback Principles

Production rollback should be:

1. Fast
2. Predictable
3. Tested
4. Auditable
5. Reversible
6. Automated where appropriate
7. Compatible with data changes
8. Observable
9. GitOps-compatible
10. Performed using a documented runbook

Never treat rollback as:

```text
kubectl commands somebody remembers during an outage
```

It should be a designed production capability.

---

# 29.5 Known-Good Version

Rollback requires a known-good version.

For example:

```text
payment:
v2.3
v2.4
v2.5 ← current broken release
```

If v2.4 is known-good:

```text
v2.5 → v2.4
```

The organization should retain:

- immutable image tags
- Git commit history
- Helm revisions
- Argo CD history
- deployment history
- configuration versions

Avoid mutable tags such as:

```text
latest
stable
prod
```

for production rollback decisions.

Prefer:

```text
payment:2.4.17
```

or immutable image digests.

---

# 29.6 Rollback Decision Framework

Before rollback ask:

```text
1. Is customer impact confirmed?
2. Did the problem begin after a recent change?
3. Is the previous version known-good?
4. Is the rollback technically compatible?
5. Did the release change database schema?
6. Did it change APIs?
7. Did it change configuration?
8. Did it change secrets?
9. Will old code work with current data?
10. Can rollback be safely verified?
```

If answers indicate a safe rollback:

```text
rollback
```

If database compatibility is uncertain:

```text
pause
assess data compatibility
```

---

# 29.7 Rollback Hierarchy

A useful preference order is:

```text
1. Git revert
2. GitOps reconciliation
3. Helm rollback where appropriate
4. Kubernetes rollout undo for emergency recovery
5. Image rollback
6. Infrastructure rollback
7. Restore/rebuild
```

The exact order depends on the failure.

For GitOps production environments, the desired permanent state should ultimately exist in Git.

---

# 29.8 Application Rollback

Suppose:

```text
payment v2.4 = known-good
payment v2.5 = broken
```

The deployment should return to:

```text
v2.4
```

A Kubernetes emergency rollback:

```bash
kubectl rollout history deployment/payment -n roboshop
kubectl rollout undo deployment/payment -n roboshop
kubectl rollout status deployment/payment -n roboshop
```

Then verify:

```bash
kubectl get pods -n roboshop
```

However, in GitOps, a direct cluster rollback can be temporary because Argo CD may reconcile the cluster back to Git.

Therefore the permanent rollback should normally be represented in Git.

---

# 29.9 GitOps Rollback

Preferred production workflow:

```text
Bad release
    |
    v
Identify last known-good Git revision
    |
    v
Revert GitOps change
    |
    v
Pull request / approved emergency change
    |
    v
Argo CD detects Git change
    |
    v
Sync
    |
    v
EKS returns to known-good state
```

This creates:

- audit trail
- review history
- reproducibility
- consistency
- future recovery capability

---

# 29.10 Git Revert

Suppose a GitOps commit introduced:

```text
payment image = 2.5
```

and previous commit used:

```text
payment image = 2.4
```

Use:

```bash
git log --oneline
```

Identify the bad commit:

```text
abc1234 Update payment to 2.5
```

Then:

```bash
git revert abc1234
```

Push the resulting commit:

```bash
git push origin main
```

Argo CD detects the desired-state change.

Then:

```bash
argocd app get roboshop-prod
argocd app sync roboshop-prod
```

If auto-sync is enabled, the synchronization may happen automatically.

---

# 29.11 Why Git Revert Is Better Than Git Reset

For shared production branches:

Prefer:

```bash
git revert <commit>
```

over rewriting history with:

```bash
git reset --hard
git push --force
```

A revert:

- preserves history
- records the corrective action
- is safer for collaboration
- is easier to audit

Force-pushing production history is generally undesirable.

---

# 29.12 Argo CD Rollback

Argo CD maintains application history.

Useful commands:

```bash
argocd app history <application>
```

Inspect:

```bash
argocd app get <application>
```

Compare:

```bash
argocd app diff <application>
```

A historical deployment can be useful during emergency recovery.

However, the permanent desired state should still be updated in Git.

---

# 29.13 Argo CD Emergency Recovery Model

Example:

```text
Current:
payment v2.5

Known-good:
payment v2.4
```

Incident:

```text
v2.5 → 5xx ↑
```

Emergency:

```text
Restore v2.4
```

Then:

```text
GitOps repository → v2.4
```

Argo CD:

```text
Git desired state = v2.4
Cluster state = v2.4
Status = Synced
```

The final state should be:

```text
Git = cluster
```

---

# 29.14 Helm Rollback

Helm tracks releases and revisions.

Check releases:

```bash
helm list -n roboshop
```

Check history:

```bash
helm history payment -n roboshop
```

Example:

```text
REVISION  STATUS
1         superseded
2         superseded
3         deployed
```

Rollback:

```bash
helm rollback payment 2 -n roboshop
```

Check:

```bash
helm status payment -n roboshop
```

Then:

```bash
kubectl rollout status deployment/payment -n roboshop
```

---

# 29.15 Helm Rollback in GitOps

Direct Helm rollback is usually an emergency operation when Argo CD manages the application.

Why?

Because:

```text
Helm rollback
      ↓
Cluster changes
      ↓
Git still says new version
      ↓
Argo CD sees drift
      ↓
Argo CD may reconcile new version
```

Therefore:

```text
Emergency cluster rollback
+
Git correction
```

must be coordinated.

---

# 29.16 Image Rollback

If the image is the problem:

```yaml
image:
  repository: <aws-account>.dkr.ecr.<region>.amazonaws.com/payment
  tag: "2.4.17"
```

Change from:

```text
2.5.0
```

to:

```text
2.4.17
```

Better production practice:

```yaml
image:
  repository: <ecr-repository>
  digest: "sha256:<immutable-digest>"
```

Image digests provide stronger immutability than mutable tags.

---

# 29.17 Why Image Tags Should Be Immutable

Bad:

```text
payment:latest
```

Problem:

```text
latest today ≠ latest tomorrow
```

Better:

```text
payment:2.4.17
```

Best:

```text
payment@sha256:<digest>
```

The deployment should identify exactly which artifact is running.

---

# 29.18 Kubernetes Rollback

Check rollout history:

```bash
kubectl rollout history deployment/payment -n roboshop
```

Undo:

```bash
kubectl rollout undo deployment/payment -n roboshop
```

Specific revision:

```bash
kubectl rollout undo deployment/payment \
  --to-revision=4 \
  -n roboshop
```

Watch:

```bash
kubectl rollout status deployment/payment -n roboshop
```

Verify:

```bash
kubectl get pods -n roboshop -l app=payment
```

---

# 29.19 Deployment Rollback Validation

Never stop after:

```text
deployment successfully rolled back
```

Validate:

```text
pods Ready
readiness probes passing
ALB targets healthy
request success rate normal
latency normal
application logs clean
dependencies healthy
business transaction successful
```

A Kubernetes rollout can be technically successful while the application remains broken.

---

# 29.20 Rollback and Readiness Probes

A proper readiness probe prevents traffic from being sent to an unhealthy version.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 3
```

During rollback:

```text
new/old pods start
       ↓
readiness checks
       ↓
only healthy pods receive traffic
```

---

# 29.21 Rollback and Liveness Probes

Liveness probes answer:

```text
Should Kubernetes restart this container?
```

Readiness probes answer:

```text
Should this pod receive traffic?
```

Incorrect probes can make recovery worse.

For example:

```text
Slow application startup
+
aggressive liveness probe
=
continuous restarts
```

This can prevent rollback from stabilizing.

---

# 29.22 Rollback Strategy with RollingUpdate

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 25%
```

This helps maintain capacity during normal deployment.

But rollback behavior depends on:

- replicas
- node capacity
- readiness
- PDB
- application startup time

---

# 29.23 Blue-Green Recovery

Blue-green:

```text
Blue = current production
Green = new version
```

Traffic:

```text
ALB
 |
 +--> Blue
```

After validation:

```text
ALB
 |
 +--> Green
```

If Green fails:

```text
ALB
 |
 +--> Blue
```

Advantages:

- fast switch
- easy rollback
- isolated environments

Disadvantages:

- higher resource cost
- database compatibility complexity
- duplicated capacity

---

# 29.24 Canary Recovery

Canary:

```text
95% → v2.4
5%  → v2.5
```

Monitor:

```text
error rate
latency
business metrics
resource usage
```

If healthy:

```text
5% → 25% → 50% → 100%
```

If unhealthy:

```text
5% → 0%
```

Canary deployment dramatically reduces blast radius.

---

# 29.25 Automated Rollback

Automated rollback can use signals such as:

```text
5xx > threshold
latency > threshold
availability < threshold
pod crash rate > threshold
SLO burn rate > threshold
```

Conceptual flow:

```text
Deployment
    |
    v
Metrics
    |
    v
Evaluation
    |
    +---- Healthy ----> Continue
    |
    +---- Unhealthy --> Rollback
```

Automation should include safeguards against rollback loops.

---

# 29.26 Rollback Loops

Bad automation:

```text
deploy
 ↓
alert
 ↓
rollback
 ↓
auto-deploy detects Git
 ↓
deploy bad version
 ↓
alert
 ↓
rollback
```

This creates a loop.

Prevent it with:

- deployment pause
- Git correction
- rollout controller state
- automated rollback limits
- incident declaration
- clear source-of-truth handling

---

# 29.27 Database Rollback Is Different

Application rollback can be easy.

Database rollback can be dangerous.

Example:

```text
v2.5 migration:
ADD COLUMN new_status
```

Old v2.4 may still work.

But:

```text
v2.5 migration:
DROP COLUMN old_status
```

Old v2.4 may fail.

Therefore database changes should favor:

```text
backward compatibility
```

---

# 29.28 Expand-and-Contract Pattern

Safer database migration:

```text
Phase 1:
Add new schema.

Phase 2:
Deploy code supporting old + new.

Phase 3:
Backfill data.

Phase 4:
Switch reads/writes.

Phase 5:
Remove old schema later.
```

This makes application rollback safer.

---

# 29.29 Example of Safe Database Migration

Old:

```text
name
```

New:

```text
first_name
last_name
```

Do not immediately delete:

```text
name
```

Instead:

```text
add first_name
add last_name
deploy compatible application
backfill
switch application
verify
remove old column later
```

This allows old and new application versions to coexist during deployment.

---

# 29.30 Dangerous Database Rollback

Suppose deployment:

```text
v2.5
```

executes:

```sql
DROP TABLE legacy_orders;
```

Then application rollback:

```text
v2.5 → v2.4
```

may fail because v2.4 still expects the table.

Therefore:

```text
application rollback ≠ database rollback
```

Database rollback requires separate planning.

---

# 29.31 Data Recovery vs Application Rollback

If application data is corrupted:

```text
Rollback application
```

does not automatically restore corrupted data.

You may need:

```text
backup restore
point-in-time recovery
replica failover
data repair
```

The recovery plan must identify the data layer separately.

---

# 29.32 Terraform Rollback

Terraform does not provide a simple universal:

```text
terraform rollback
```

Instead Terraform applies the desired configuration.

If a bad infrastructure change occurred:

```text
Git
 ↓
revert infrastructure change
 ↓
terraform plan
 ↓
review
 ↓
terraform apply
```

Example:

```bash
git revert <bad-terraform-commit>
terraform init
terraform validate
terraform plan
terraform apply
```

Always review the plan carefully.

---

# 29.33 Terraform Rollback Safety

Never blindly run:

```bash
terraform apply
```

during an incident.

Inspect:

```text
resource replacements
deletions
IAM changes
security group changes
subnet changes
EKS changes
database changes
```

A rollback plan that destroys a production database is not a safe rollback.

---

# 29.34 Terraform State

Terraform rollback depends on accurate state.

Check:

```bash
terraform state list
```

Do not manually edit state files unless you understand the consequences and have an approved procedure.

State is critical production data.

---

# 29.35 Infrastructure Recovery Example

Bad change:

```text
security group removes required application egress
```

Impact:

```text
pods cannot reach dependency
```

Recovery:

```text
1. Confirm network failure.
2. Identify recent Terraform change.
3. Revert Git commit.
4. terraform plan.
5. Review network changes.
6. Apply approved change.
7. Validate connectivity.
```

---

# 29.36 ECR Recovery

ECR should retain enough image history for rollback.

Recommended:

- immutable tags
- lifecycle policies
- retention of production versions
- image scanning
- controlled deletion
- cross-region replication where required

Do not delete the previous production image immediately after deployment.

---

# 29.37 ECR Image Retention

Example policy concept:

```text
Keep:
- current production
- previous production
- approved rollback versions
- recent release candidates
```

Balance:

```text
rollback capability
vs
storage cost
```

---

# 29.38 ECR Image Verification

Before rollback:

```bash
aws ecr describe-images \
  --repository-name <repository> \
  --image-ids imageTag=<tag>
```

Verify:

- image exists
- correct architecture
- correct digest
- correct repository
- expected scan status

---

# 29.39 Helm Values Rollback

A release may fail because of configuration rather than code.

Example:

```yaml
resources:
  requests:
    cpu: "4"
    memory: "8Gi"
```

If nodes cannot satisfy this:

```text
Pending pods
```

Rollback configuration:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

But production teams should also fix the underlying capacity/configuration problem rather than repeatedly reverting.

---

# 29.40 ConfigMap Rollback

Configuration changes can break production.

Example:

```yaml
DATABASE_HOST: new-db.internal
```

Previous:

```yaml
DATABASE_HOST: db.internal
```

Rollback:

```text
Git revert
→ Argo CD
→ ConfigMap
→ pod restart if required
```

A ConfigMap update does not always cause application processes to reload configuration automatically.

---

# 29.41 Secret Rollback

Secrets require extra caution.

If a new secret is invalid:

```text
restore known-good secret version
```

But never put secret values into Git history.

Use:

- AWS Secrets Manager
- Kubernetes secret management solution
- external secret integration where configured

Audit secret changes separately.

---

# 29.42 Certificate Recovery

If a certificate change breaks ingress:

Check:

```bash
kubectl describe ingress <ingress> -n <namespace>
kubectl get secret -n <namespace>
```

Validate:

```text
certificate
issuer
hostname
expiration
ALB listener
```

Rollback to a known-good certificate configuration where appropriate.

---

# 29.43 ALB Recovery

If an ingress change causes outage:

```bash
kubectl get ingress -A
kubectl describe ingress <ingress> -n <namespace>
```

Check AWS:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

Rollback the Ingress/Helm configuration through GitOps.

---

# 29.44 DNS Recovery

For a bad DNS change:

```text
Identify incorrect record
↓
Restore previous known-good record
↓
Verify authoritative response
↓
Verify recursive resolution
↓
Verify ALB/application
```

Consider DNS TTL and caching.

Recovery may not be instantaneous because cached responses can persist.

---

# 29.45 Argo CD Rollback and Drift

Suppose:

```text
Git = v2.5
Cluster = v2.4
```

Argo CD:

```text
OutOfSync
```

If v2.4 is the emergency mitigation, update Git:

```text
Git = v2.4
Cluster = v2.4
```

Final:

```text
Synced
Healthy
```

The goal is:

```text
desired state = actual state
```

---

# 29.46 Rollback Verification with Prometheus

Before rollback:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

After rollback:

```text
error rate should decrease
```

Latency:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

Compare:

```text
before
during
after
```

---

# 29.47 Rollback Verification with Logs

Check:

```bash
kubectl logs deployment/payment -n roboshop --tail=200
```

Search ELK for:

```text
ERROR
Exception
timeout
connection refused
```

Confirm that errors associated with the bad release disappear.

---

# 29.48 Rollback Verification with Business Metrics

Technical health is not enough.

For RoboShop:

```text
checkout success
cart operation success
payment success
order creation
customer login
```

A service can report:

```text
HTTP 200
```

while the business transaction is still broken.

---

# 29.49 Rollback Runbook

## Step 1 — Confirm Incident

```text
customer impact
severity
service
environment
```

## Step 2 — Identify Change

```text
Git
Argo CD
Helm
Kubernetes
Terraform
```

## Step 3 — Identify Known-Good Version

```text
image
Git revision
Helm revision
```

## Step 4 — Check Compatibility

```text
database
API
configuration
secrets
```

## Step 5 — Execute Safe Rollback

Use the appropriate mechanism.

## Step 6 — Monitor

```text
Prometheus
Grafana
ELK
ALB
Kubernetes
```

## Step 7 — Verify

```text
availability
latency
errors
business transactions
```

## Step 8 — Correct GitOps State

Ensure:

```text
Git = intended production state
```

## Step 9 — Document

Record:

```text
version
time
reason
commands/actions
result
follow-up
```

---

# 29.50 Emergency Kubernetes Rollback Example

```bash
kubectl config current-context

kubectl get deployment payment -n roboshop

kubectl rollout history deployment/payment -n roboshop

kubectl rollout undo deployment/payment \
  --to-revision=4 \
  -n roboshop

kubectl rollout status deployment/payment \
  -n roboshop

kubectl get pods -n roboshop \
  -l app=payment
```

Then:

```bash
kubectl get endpoints payment -n roboshop
```

And verify application behavior.

---

# 29.51 Emergency Helm Rollback Example

```bash
helm history payment -n roboshop

helm rollback payment 4 -n roboshop

helm status payment -n roboshop

kubectl rollout status \
  deployment/payment \
  -n roboshop
```

Then update Git if Helm is managed by Argo CD.

---

# 29.52 Emergency GitOps Rollback Example

```bash
git log --oneline -- deployment/payment/values.yaml
```

Identify:

```text
bad release commit
```

Then:

```bash
git revert <commit>
git push origin main
```

Check:

```bash
argocd app get roboshop-prod
```

If required:

```bash
argocd app sync roboshop-prod
```

Verify:

```text
Synced
Healthy
```

---

# 29.53 Rollback YAML Example

Example deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
  namespace: roboshop
  labels:
    app.kubernetes.io/name: payment
    app.kubernetes.io/part-of: roboshop
spec:
  replicas: 6
  revisionHistoryLimit: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 25%
  selector:
    matchLabels:
      app.kubernetes.io/name: payment
  template:
    metadata:
      labels:
        app.kubernetes.io/name: payment
        app.kubernetes.io/part-of: roboshop
    spec:
      containers:
        - name: payment
          image: <account>.dkr.ecr.<region>.amazonaws.com/payment@sha256:<immutable-digest>
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 20
            periodSeconds: 10
```

Important rollback-supporting fields:

```text
revisionHistoryLimit
immutable image
readinessProbe
rolling strategy
```

---

# 29.54 Rollback and PodDisruptionBudget

A PDB can protect availability during planned disruption.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payment
  namespace: roboshop
spec:
  minAvailable: 4
  selector:
    matchLabels:
      app.kubernetes.io/name: payment
```

But PDBs do not guarantee that a broken deployment can roll back successfully.

Capacity must exist.

---

# 29.55 Rollback and Horizontal Pod Autoscaler

If HPA is configured:

```text
Deployment replicas
+
HPA
```

The effective replica count may be controlled by HPA.

Check:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa payment -n roboshop
```

During incident response, avoid fighting the controller manually without understanding the desired behavior.

---

# 29.56 Rollback and Cluster Capacity

Suppose:

```text
Current:
12 pods

Rollback requires:
12 old pods
```

But the cluster has only enough capacity for:

```text
8 pods
```

Rollback may leave pods Pending.

Check:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pending-pod> -n roboshop
kubectl get nodes
```

Production clusters should maintain sufficient headroom for recovery.

---

# 29.57 Rollback and Node Autoscaling

If capacity is insufficient:

```text
Cluster Autoscaler / node scaling
```

may add capacity.

But autoscaling is not instantaneous.

Therefore production planning should consider:

```text
normal capacity
+
failure capacity
+
rollback capacity
```

---

# 29.58 Rollback and Service Mesh

This capstone does not require a service mesh.

If one existed, rollback would additionally consider:

- traffic routing
- subsets
- weights
- retries
- circuit breakers

The principle remains:

```text
route traffic to known-good version
```

---

# 29.59 Rollback and API Compatibility

Suppose:

```text
Frontend v3
Payment v2
```

New payment v3 changes API:

```text
POST /payment
```

to:

```text
POST /payments
```

Rolling payment back may break the frontend if the frontend was also changed.

Therefore check:

```text
consumer compatibility
API version
contract
```

---

# 29.60 Rollback and Microservices

For RoboShop:

```text
frontend
catalogue
cart
user
shipping
payment
dispatch
```

A rollback should ideally affect only the faulty service.

Avoid:

```text
rollback everything
```

unless there is a systemic reason.

Targeted rollback reduces blast radius.

---

# 29.61 Coordinated Rollback

Sometimes multiple services must roll back together.

Example:

```text
frontend v3
payment v3
order v3
```

If APIs changed together:

```text
frontend v3 ↔ payment v3
```

then:

```text
frontend v3 → v2
payment v3 → v2
```

may be required.

Use compatibility matrices for major API changes.

---

# 29.62 Rollback Testing

Rollback must be tested before a real outage.

Test:

```text
deployment rollback
Helm rollback
GitOps rollback
image rollback
database compatibility
Terraform recovery
EKS node replacement
```

A rollback process that has never been tested is not a reliable production capability.

---

# 29.63 Recovery Game Day

Example game day:

```text
Scenario:
Payment v2.5 causes 10% errors.
```

Team must:

```text
detect
declare
identify release
rollback
verify
document
```

Measure:

```text
time to decision
time to rollback
time to recovery
errors during recovery
```

---

# 29.64 Rollback Safety Checklist

Before:

- [ ] Correct AWS account
- [ ] Correct region
- [ ] Correct Kubernetes context
- [ ] Correct namespace
- [ ] Customer impact confirmed
- [ ] Known-good version identified
- [ ] Database compatibility checked
- [ ] Dependencies checked
- [ ] Capacity verified
- [ ] Rollback owner identified

During:

- [ ] Change recorded
- [ ] Monitoring active
- [ ] Rollback progress monitored
- [ ] No unrelated changes

After:

- [ ] Pods healthy
- [ ] ALB healthy
- [ ] Error rate normal
- [ ] Latency normal
- [ ] Business transaction verified
- [ ] GitOps state corrected
- [ ] Incident timeline updated

---

# 29.65 Common Rollback Mistakes

## Mistake 1: Rolling back without checking database changes

Result:

```text
old code incompatible with new schema
```

## Mistake 2: Using mutable image tags

Result:

```text
rollback target is ambiguous
```

## Mistake 3: Rolling back directly in cluster under GitOps

Result:

```text
Argo CD reverts emergency change
```

## Mistake 4: Not checking capacity

Result:

```text
rollback pods Pending
```

## Mistake 5: Not verifying business functionality

Result:

```text
Kubernetes healthy
but checkout broken
```

## Mistake 6: Rolling back too many services

Result:

```text
larger blast radius
```

## Mistake 7: Force-pushing production Git

Result:

```text
lost auditability
```

## Mistake 8: Treating Terraform as application rollback

Result:

```text
unnecessary infrastructure changes
```

---

# 29.66 Recovery Validation Matrix

| Layer | Validation |
|---|---|
| DNS | Correct resolution |
| ALB | Healthy targets |
| Ingress | Correct routing |
| Service | Endpoints present |
| Pods | Ready |
| Deployment | Successful rollout |
| Application | Requests successful |
| Database | Healthy |
| Metrics | Errors normal |
| Logs | No recurring failures |
| GitOps | Synced |
| Security | No new exposure |
| Business | Transaction succeeds |

---

# 29.67 Rollback Architecture

```text
                    Git
                     |
                     v
              GitOps Repository
                     |
                     v
                  Argo CD
                     |
                     v
                    EKS
                     |
          +----------+----------+
          |                     |
          v                     v
    Current Release       Known-Good Release
          |                     |
          +----------+----------+
                     |
                     v
                 Monitoring
                     |
             +-------+-------+
             |               |
             v               v
        Prometheus          ELK
             |
             v
        Alertmanager
             |
             v
           On-call
             |
             v
        Rollback decision
```

---

# 29.68 End-to-End Production Rollback Scenario

### Situation

At 09:45:

```text
payment v2.5 deployed
```

At 09:49:

```text
5xx = 11%
p95 latency = 3.8s
```

Alert:

```text
PaymentHighErrorRate
```

### Investigation

```bash
argocd app history roboshop-prod
kubectl rollout history deployment/payment -n roboshop
```

Recent release identified.

Logs:

```text
database connection timeout
```

Database:

```text
connection utilization high
```

### Decision

Known-good:

```text
v2.4
```

Database migration:

```text
backward compatible
```

Decision:

```text
rollback
```

### Action

GitOps change:

```text
v2.5 → v2.4
```

Argo CD:

```text
sync
```

### Verification

Prometheus:

```text
5xx: 11% → 0.5%
```

Latency:

```text
3.8s → 300ms
```

ELK:

```text
connection timeout errors stop
```

Business test:

```text
checkout successful
```

### Follow-up

Root cause:

```text
connection pool regression
```

Prevention:

```text
bounded connection pool
load test
connection saturation alert
canary release
```

---

# 29.69 Failed Rollback Scenario

Suppose:

```text
v2.5 migration removed old database field.
v2.4 requires old field.
```

Rollback starts.

Application:

```text
CrashLoopBackOff
```

Correct response:

```text
STOP repeated rollback attempts.
```

Investigate:

```text
schema compatibility
```

Recovery options:

```text
restore compatible schema
deploy intermediate compatibility version
repair migration
restore data if required
```

This demonstrates why database-aware deployment strategies are essential.

---

# 29.70 Recovery When Rollback Is Impossible

Rollback may be impossible when:

- destructive migration occurred
- old image unavailable
- old dependencies removed
- security vulnerability prevents reuse
- data format changed
- infrastructure dependency no longer exists

Then use:

```text
forward fix
```

Example:

```text
Bad v2.5
  ↓
v2.5.1 hotfix
  ↓
restore service
```

Recovery does not always mean going backward.

---

# 29.71 Forward Fix vs Rollback

Use rollback when:

```text
known-good version
+
safe compatibility
```

Use forward fix when:

```text
rollback unsafe
OR
data schema incompatible
OR
previous version unavailable
```

Senior engineers should be comfortable with both.

---

# 29.72 Recovery from GitOps Repository Failure

If GitOps repository is unavailable:

```text
GitHub/GitLab outage
```

Existing workloads may continue running because Kubernetes does not require Git for every runtime request.

But:

```text
new deployment
rollback through Git
configuration change
```

may be blocked.

Therefore maintain:

- repository backups/mirrors
- protected branches
- emergency access
- documented recovery process

---

# 29.73 Recovery from Argo CD Failure

If Argo CD control plane fails:

```text
existing Kubernetes workloads can continue running
```

but Git reconciliation stops.

Recovery:

```text
restore Argo CD
restore repository access
verify applications
check drift
resume reconciliation
```

The application platform and GitOps control plane should be treated as separate failure domains.

---

# 29.74 Recovery from EKS Control Plane Issue

If EKS control-plane operations are impaired:

```text
existing pods may continue running
```

but management actions may fail.

Do not immediately restart healthy workloads.

First determine:

```text
data plane health
customer impact
AWS service health
control-plane availability
```

Preserve running workloads if they are serving traffic.

---

# 29.75 Recovery from Node Failure

If a node fails:

```text
node → NotReady
```

and workloads are replicated:

```text
scheduler
  ↓
replacement node
  ↓
pods rescheduled
```

Verify:

```bash
kubectl get nodes
kubectl get pods -A -o wide
```

Rollback is not necessarily required because this is an infrastructure failure rather than a release failure.

---

# 29.76 Recovery from Availability-Zone Failure

With multi-AZ EKS:

```text
AZ-A
AZ-B
AZ-C
```

workloads should be distributed where appropriate.

Recovery depends on:

```text
replicas
topology spread
PDB
node capacity
load balancer
database HA
```

A cluster with three AZs is not automatically highly available if all replicas are concentrated in one AZ.

---

# 29.77 Recovery from Region Failure

Regional recovery requires:

```text
secondary region
infrastructure definition
container artifacts
GitOps configuration
data replication/backup
DNS failover
operational runbooks
```

Typical flow:

```text
Primary region failure
        |
        v
Declare DR
        |
        v
Promote/restore data
        |
        v
Provision/activate EKS
        |
        v
Deploy via GitOps
        |
        v
Restore ingress
        |
        v
DNS failover
        |
        v
Validate application
```

This belongs to the broader DR plan covered later in the capstone.

---

# 29.78 Rollback Security

Rollback should respect:

- RBAC
- least privilege
- protected Git branches
- approval policies
- audit logs
- production access controls

Not every engineer should have unrestricted:

```bash
kubectl delete
terraform apply
```

Production recovery must be fast without becoming uncontrolled.

---

# 29.79 Break-Glass Access

A break-glass account or role may be used during severe incidents.

It should have:

- strong authentication
- limited scope
- audit logging
- emergency approval
- automatic review
- credential rotation where applicable

Break-glass access should not become the normal operating model.

---

# 29.80 Rollback Observability

Track:

```text
deployment version
rollback version
rollback start
rollback end
error rate
latency
pod readiness
ALB health
business success
```

A dashboard can show:

```text
Release v2.5
   |
   +--> error rate ↑
   |
   +--> rollback
   |
Release v2.4
   |
   +--> error rate ↓
```

This makes incident analysis easier.

---

# 29.81 Production Rollback Best Practices

1. Keep previous production images.
2. Use immutable image references.
3. Keep sufficient Helm history.
4. Keep Kubernetes revision history.
5. Use Git revert for permanent GitOps rollback.
6. Test rollback regularly.
7. Design backward-compatible migrations.
8. Maintain production capacity headroom.
9. Monitor business metrics.
10. Keep rollback runbooks current.
11. Avoid force-pushing production history.
12. Avoid mutable image tags.
13. Avoid unrelated changes during rollback.
14. Record every emergency action.
15. Reconcile emergency changes back into Git.
16. Prefer targeted rollback.
17. Use canary/blue-green strategies for high-risk releases.
18. Automate safe rollback signals.
19. Prevent rollback loops.
20. Practice recovery through game days.

---

# 29.82 Senior Interview Questions

## Q1. How do you rollback a production deployment in Kubernetes?

I first identify the failing release and known-good revision, verify database and API compatibility, and then use the organization's deployment mechanism. In a GitOps environment I prefer reverting the GitOps change and allowing Argo CD to reconcile. For emergency stabilization I can use `kubectl rollout undo`, but I then make Git match the recovered state.

## Q2. Why can Kubernetes rollback fail?

The previous application version may be incompatible with the current database schema, configuration, API, secrets, or dependencies. Cluster capacity can also prevent old pods from starting.

## Q3. How do you rollback with Argo CD?

I normally revert the GitOps repository to the known-good application configuration. Argo CD detects the desired-state change and reconciles the cluster. I verify the application becomes Synced and Healthy.

## Q4. How do you rollback Helm?

I inspect `helm history`, select a known-good revision, and use `helm rollback`. If Helm is managed by Argo CD, I also correct the Git source of truth to avoid Argo CD reconciling the bad version back.

## Q5. What is the safest way to rollback a container image?

Use an immutable version or digest that is known to be good. Avoid relying on `latest` or mutable tags.

## Q6. Can you always rollback a database migration?

No. Destructive or incompatible schema changes can make application rollback unsafe. I prefer backward-compatible expand-and-contract migrations.

## Q7. What is the difference between rollback and recovery?

Rollback returns a component to a previous state. Recovery is broader and may involve rollback, failover, restore, replacement, or a forward fix.

## Q8. What would you do if rollback itself failed?

I would stop repeated attempts, determine why it failed, assess data/schema compatibility and capacity, and choose another safe recovery path such as an intermediate version, forward fix, data restoration, or infrastructure recovery.

## Q9. Why is GitOps important during rollback?

It provides an auditable desired state. Without correcting Git, an emergency cluster rollback can be overwritten by the GitOps controller.

## Q10. How do you verify rollback success?

I verify deployment health, pod readiness, ALB target health, error rate, latency, logs, dependency health, and business transactions. I do not rely only on `kubectl rollout status`.

## Q11. What is blue-green deployment?

Two production-capable environments exist. Traffic is switched from the current environment to the new one after validation. If the new environment fails, traffic can be switched back.

## Q12. What is canary deployment?

A small percentage of traffic is sent to the new version first. Metrics are evaluated before progressively increasing traffic.

## Q13. What makes a rollback production-safe?

Known-good immutable artifacts, tested procedures, backward-compatible data changes, sufficient capacity, observability, access control, and a clear source-of-truth strategy.

## Q14. How do you rollback Terraform?

Terraform does not have a universal rollback command. I revert the infrastructure configuration in Git, run `terraform plan`, carefully inspect the proposed changes, and apply only after confirming that the recovery will not destroy critical resources.

## Q15. What is your rollback philosophy?

Restore customer-facing reliability quickly while minimizing additional risk. I choose the smallest safe recovery action, verify it using both technical and business signals, and then permanently reconcile the recovered state into the source of truth.

---

# 29.83 Final Production Recovery Model

The production rollback model should be:

```text
                  INCIDENT
                     |
                     v
              IDENTIFY CHANGE
                     |
                     v
            IDENTIFY KNOWN-GOOD
                     |
                     v
          CHECK COMPATIBILITY
                     |
          +----------+----------+
          |                     |
        SAFE                  UNSAFE
          |                     |
          v                     v
      ROLLBACK              FORWARD FIX
          |                     |
          +----------+----------+
                     |
                     v
                 MONITOR
                     |
                     v
                 VERIFY
                     |
                     v
          BUSINESS VALIDATION
                     |
                     v
             UPDATE GIT STATE
                     |
                     v
              ARGO CD SYNC
                     |
                     v
             DOCUMENT + RCA
                     |
                     v
              PREVENT RECURRENCE
```

The key production principle is:

```text
Rollback is not simply going to an old version.

Rollback is a controlled recovery operation that must consider:
application
database
configuration
infrastructure
GitOps
capacity
security
observability
and customer impact.
```

A strong DevOps engineer does not merely know:

```bash
kubectl rollout undo
```

They understand:

```text
WHEN to rollback
WHY to rollback
WHAT can make rollback unsafe
HOW to execute it safely
HOW to verify recovery
HOW to reconcile GitOps state
HOW to prevent the incident from recurring
```

That is the production-level rollback and recovery mindset.
