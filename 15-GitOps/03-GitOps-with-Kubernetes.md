# GitOps-with-Kubernetes

## 1. Purpose

This file connects GitOps principles directly to Kubernetes.

The previous files established:

```text
Git = Desired State
Argo CD = GitOps Reconciliation Controller
Kubernetes = Runtime Control Plane
EKS = Managed Kubernetes Platform
```

This file explains how these systems work together in a real Kubernetes and AWS EKS environment.

The focus is:

- Kubernetes declarative management
- Kubernetes API architecture
- Kubernetes controllers
- Kubernetes reconciliation
- GitOps control loops
- Resource ownership
- Namespaces
- Deployments
- ReplicaSets
- Pods
- Services
- ConfigMaps
- Secrets
- Ingress
- AWS ALB
- HPA
- Resource requests and limits
- Probes
- SecurityContext
- Helm
- Kustomize
- Kubernetes drift
- GitOps resource lifecycle
- Production repository structure
- Production YAML examples
- Commands
- Troubleshooting
- RoboShop implementation
- Interview scenarios

The target production flow is:

```text
Developer
    |
    v
Application Git
    |
    v
Jenkins / GitHub Actions
    |
    +--> Build
    +--> Test
    +--> SonarQube
    +--> Trivy
    +--> Veracode
    |
    v
Docker Image
    |
    v
Amazon ECR
    |
    v
GitOps Repository
    |
    v
Argo CD
    |
    v
Kubernetes API
    |
    v
Amazon EKS
    |
    +--> Deployment
    +--> Service
    +--> ConfigMap
    +--> Secret
    +--> HPA
    +--> Ingress
    |
    v
AWS ALB
```

---

# 2. Why Kubernetes Is a Natural Platform for GitOps

Kubernetes was designed around declarative desired state.

A Kubernetes manifest says:

```text
I want this resource to exist in this state.
```

For example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 3
```

Kubernetes controllers continuously work toward this desired state.

GitOps adds another layer:

```text
Git
 |
 | Desired configuration
 v
Argo CD
 |
 | Kubernetes resources
 v
Kubernetes controllers
 |
 v
Runtime
```

This makes Kubernetes especially suitable for GitOps.

---

# 3. Kubernetes Desired State

Kubernetes resources are declarative.

A Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 3
```

declares:

```text
There should be three replicas of cart.
```

The Deployment controller determines how to achieve that.

The developer does not need to manually create:

```text
ReplicaSet
Pod 1
Pod 2
Pod 3
```

Kubernetes handles those details.

---

# 4. Kubernetes Actual State

The Kubernetes API stores the current resource state.

Commands such as:

```bash
kubectl get deployment cart -n roboshop
```

show runtime information.

For example:

```text
NAME   READY   UP-TO-DATE   AVAILABLE
cart   3/3     3            3
```

GitOps compares the desired configuration with the state represented in Kubernetes.

---

# 5. GitOps + Kubernetes Control Loops

There are multiple control loops.

At the GitOps layer:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes API
```

At the Kubernetes layer:

```text
Kubernetes desired state
 |
 v
Kubernetes controllers
 |
 v
Pods / Services / Runtime
```

The complete model is:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes API
 |
 +--> Deployment Controller
 |        |
 |        v
 |      Pods
 |
 +--> HPA Controller
 |
 +--> Service handling
 |
 +--> Other controllers
 |
 v
Runtime
```

This layered reconciliation model is critical when troubleshooting production systems.

---

# 6. Kubernetes API Server

The Kubernetes API server is the central API endpoint for Kubernetes.

Conceptually:

```text
kubectl
   |
   v
API Server
   |
   +--> Authentication
   +--> Authorization
   +--> Admission
   |
   v
Kubernetes State
```

Argo CD also interacts with Kubernetes through the API.

```text
Argo CD
   |
   v
Kubernetes API Server
```

Argo CD does not normally bypass the Kubernetes API to manipulate Pods directly.

---

# 7. Why the Kubernetes API Matters to GitOps

The API server is the boundary between Argo CD and Kubernetes.

If the API is unavailable:

```text
Argo CD
   |
   X
API Server
```

Argo CD cannot reliably:

- Read resources
- Compare state
- Apply resources
- Delete resources
- Check resource status

The existing workloads may continue running, but reconciliation is impaired.

---

# 8. Kubernetes Authentication

Kubernetes API requests must be authenticated.

For EKS, authentication can involve AWS identity and EKS access mechanisms.

Conceptually:

```text
Argo CD
   |
   | credentials / authentication
   v
EKS Kubernetes API
```

The exact authentication mechanism depends on the EKS and Argo CD architecture.

The important production principle is:

```text
Argo CD must have controlled access to the target cluster.
```

---

# 9. Kubernetes Authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

Kubernetes RBAC can control:

```text
User
ServiceAccount
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

For example:

```text
Argo CD
   |
   v
Kubernetes RBAC
   |
   +--> Allowed resources
   +--> Allowed namespaces
   +--> Allowed operations
```

Least privilege should be used where practical.

---

# 10. Admission Control

After authentication and authorization, Kubernetes can apply admission controls.

Conceptually:

```text
Request
  |
  v
Authentication
  |
  v
Authorization
  |
  v
Admission
  |
  v
Resource accepted/rejected
```

Admission policies can enforce requirements such as:

- Security constraints
- Allowed registries
- Required labels
- Resource limits
- Pod security requirements

Therefore Argo CD can successfully attempt a sync while Kubernetes rejects a resource.

---

# 11. GitOps and Admission Policies

The flow can be:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes API
 |
 v
Admission Policy
 |
 +--> Allowed
 |
 +--> Rejected
```

If rejected:

```text
Argo CD sync
     |
     v
Kubernetes API error
     |
     v
Application Sync Failed
```

This is an important production troubleshooting scenario.

---

# 12. Kubernetes Namespace

A namespace provides a logical boundary inside a Kubernetes cluster.

Example:

```text
EKS
|
+-- roboshop-dev
|
+-- roboshop-qa
|
+-- roboshop-prod
```

GitOps can manage namespaces declaratively.

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: prod
    managed-by: argocd
```

---

# 13. Why Namespaces Matter in GitOps

Namespaces can provide:

- Logical isolation
- Resource organization
- RBAC boundaries
- ResourceQuota boundaries
- NetworkPolicy boundaries
- Environment separation

However:

> A namespace is not the same as a separate Kubernetes cluster.

For strong production isolation, separate clusters and/or AWS accounts may be preferred.

---

# 14. Namespace Ownership

A common production decision is:

```text
Terraform
    |
    v
EKS

Argo CD
    |
    v
Namespace
```

or:

```text
Terraform
    |
    v
EKS + Namespace
```

Either can work.

The important rule is to avoid:

```text
Terraform -> Namespace
Argo CD   -> Namespace
```

both managing the same resource without a clear ownership design.

---

# 15. Kubernetes Deployment

A Deployment describes how an application workload should run.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart:2026.08.19-abc123
          ports:
            - containerPort: 8080
```

GitOps can store this manifest.

Argo CD can synchronize it into EKS.

---

# 16. Deployment Reconciliation

The flow is:

```text
Git:
replicas = 3
       |
       v
Argo CD
       |
       v
Kubernetes Deployment
       |
       v
Deployment Controller
       |
       v
ReplicaSet
       |
       v
Pods
```

If a Pod dies, Kubernetes recreates it.

If Git changes the Deployment image, Argo CD updates the Deployment and Kubernetes performs the rollout.

---

# 17. ReplicaSet

A Deployment normally creates a ReplicaSet.

Conceptually:

```text
Deployment
    |
    v
ReplicaSet
    |
    +--> Pod
    +--> Pod
    +--> Pod
```

The Deployment manages rollout history and ReplicaSets.

GitOps generally manages the Deployment rather than manually managing its generated ReplicaSets.

---

# 18. Pod

Pods are the runtime execution unit in Kubernetes.

Example:

```text
cart Deployment
      |
      v
ReplicaSet
      |
      +--> cart-pod-1
      +--> cart-pod-2
      +--> cart-pod-3
```

Argo CD generally manages the higher-level desired resources.

Kubernetes controllers manage the resulting Pods.

---

# 19. Resource Ownership Chain

A typical workload ownership chain is:

```text
Argo CD Application
        |
        v
Deployment
        |
        v
ReplicaSet
        |
        v
Pod
```

This is important when interpreting Argo CD resource trees.

If a Pod is unhealthy:

```text
Argo CD
  |
  v
Deployment
  |
  v
ReplicaSet
  |
  v
Pod
```

Troubleshooting should move down the ownership chain.

---

# 20. Kubernetes Service

A Service provides stable network access to Pods.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: cart
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

The GitOps repository can declare this Service.

Argo CD synchronizes it.

Kubernetes provides the runtime networking behavior.

---

# 21. Why ClusterIP Is Common

For internal microservices:

```text
cart
catalog
orders
payment
```

should generally not each receive an external load balancer.

A typical architecture is:

```text
ALB
 |
 v
Frontend / Ingress
 |
 v
Internal Services
 |
 +--> cart
 +--> catalog
 +--> orders
 +--> payment
```

This reduces unnecessary exposure and infrastructure cost.

---

# 22. Kubernetes ConfigMap

ConfigMaps store non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-config
  namespace: roboshop
data:
  LOG_LEVEL: "INFO"
  CART_PORT: "8080"
```

A Deployment can consume it:

```yaml
envFrom:
  - configMapRef:
      name: cart-config
```

GitOps can manage ConfigMaps because they are declarative resources.

Do not use ConfigMaps for secrets.

---

# 23. Kubernetes Secret

Kubernetes Secrets represent sensitive values.

Example structure:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cart-secret
  namespace: roboshop
type: Opaque
stringData:
  API_KEY: example
```

However, production GitOps should not normally commit real plaintext secrets.

Use a secure secret-management architecture.

Examples:

```text
AWS Secrets Manager
External Secrets Operator
SOPS
Sealed Secrets
Secret Store CSI Driver
```

Detailed secrets management is covered in:

```text
15-GitOps-Secrets-Management.md
```

---

# 24. Deployment Environment Variables

A production Deployment may use:

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: cart-config
        key: LOG_LEVEL

  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: cart-secret
        key: DATABASE_PASSWORD
```

The Deployment declares how the application consumes configuration.

The actual secret value should come from a secure mechanism.

---

# 25. Kubernetes Ingress

Ingress defines HTTP/HTTPS routing into Kubernetes.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

For the user's architecture, AWS ALB is used.

---

# 26. AWS Load Balancer Controller

The AWS Load Balancer Controller watches Kubernetes resources and provisions/configures AWS load-balancing resources.

Conceptually:

```text
Git
 |
 v
Argo CD
 |
 v
Ingress
 |
 v
AWS Load Balancer Controller
 |
 v
AWS ALB
```

This is a layered reconciliation chain.

---

# 27. Why ALB Is Used in This Architecture

The RoboShop architecture uses:

```text
AWS ALB Ingress
```

rather than API Gateway.

External traffic:

```text
Client
 |
 v
AWS ALB
 |
 v
Kubernetes Ingress
 |
 v
Service
 |
 v
Pod
```

GitOps can manage the Ingress declaration.

AWS controllers handle the cloud-side implementation.

---

# 28. Ingress Annotations

AWS ALB behavior may be configured using annotations.

Example:

```yaml
metadata:
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
```

These annotations become part of desired Kubernetes state.

Argo CD manages the Ingress resource.

The AWS Load Balancer Controller interprets the annotations.

---

# 29. HPA

Horizontal Pod Autoscaler adjusts replica count based on metrics.

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cart
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cart
  minReplicas: 3
  maxReplicas: 10
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

GitOps can manage the HPA configuration.

The HPA controller manages runtime replicas.

---

# 30. HPA and GitOps Drift

Suppose Git declares:

```yaml
minReplicas: 3
maxReplicas: 10
```

The HPA may dynamically set:

```text
Deployment replicas = 7
```

This is expected.

The HPA owns the runtime replica value.

Therefore GitOps must understand controller-owned fields.

This is one reason resource comparison is more complex than simply comparing every field literally.

---

# 31. Resource Requests

Production workloads should define resource requests.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```

Requests help Kubernetes schedule Pods.

For example:

```text
CPU request = 100m
Memory request = 128Mi
```

means the scheduler considers those resources when placing the Pod.

---

# 32. Resource Limits

Limits define upper boundaries.

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

A production Deployment may therefore use:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

The exact values should come from workload measurement rather than arbitrary defaults.

---

# 33. Why GitOps Should Manage Resources

Resource requests and limits are operational behavior.

If they are managed manually:

```text
Git:
unknown

Cluster:
cpu=100m
memory=128Mi
```

the desired state is incomplete.

By storing them in Git:

```text
Git:
requests/limits defined
```

they become reviewable and reproducible.

---

# 34. Readiness Probe

Readiness determines whether a Pod should receive traffic.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3
```

If readiness fails:

```text
Pod remains running
but
Service should not route traffic to it
```

This is especially important during GitOps deployments.

---

# 35. Liveness Probe

Liveness checks whether the application remains alive.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 20
  timeoutSeconds: 3
  failureThreshold: 3
```

If liveness repeatedly fails, Kubernetes may restart the container.

---

# 36. Startup Probe

Startup probes are useful for slow-starting applications.

Example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 30
```

This allows a slow application time to initialize before liveness checks become active.

---

# 37. Probes and GitOps Health

Argo CD may report an application as unhealthy when Kubernetes resources are not ready.

For example:

```text
Git -> Deployment
      |
      v
Pods
      |
      v
Readiness fails
      |
      v
Application not healthy
```

This is why a successful sync does not necessarily mean successful application deployment.

---

# 38. SecurityContext

Production Pods should use appropriate security settings.

Example:

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

Container-level:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

These should be validated against the application's requirements.

Do not blindly enable settings that break legitimate application behavior.

---

# 39. Production Deployment Example

A more realistic Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
    app.kubernetes.io/managed-by: argocd
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: cart
  template:
    metadata:
      labels:
        app.kubernetes.io/name: cart
        app.kubernetes.io/part-of: roboshop
    spec:
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart:2026.08.19-abc123
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
          envFrom:
            - configMapRef:
                name: cart-config
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          startupProbe:
            httpGet:
              path: /health
              port: http
            periodSeconds: 10
            failureThreshold: 30
          readinessProbe:
            httpGet:
              path: /health
              port: http
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: http
            periodSeconds: 20
            timeoutSeconds: 3
            failureThreshold: 3
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

This is a production-oriented pattern, but the exact probe paths, ports, resources, and security settings must match the actual application.

---

# 40. GitOps Repository for Kubernetes

A practical repository can be:

```text
roboshop-gitops/
├── applications/
│   └── cart.yaml
│
├── applicationsets/
│
├── projects/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── helm/
│   └── roboshop/
│
├── clusters/
│
└── platform/
    ├── namespaces/
    ├── ingress/
    └── monitoring/
```

The repository structure will evolve as the syllabus reaches ApplicationSets and multi-cluster management.

---

# 41. GitOps Resource Lifecycle

A Kubernetes resource managed through GitOps follows:

```text
1. Define in Git
        |
        v
2. Commit
        |
        v
3. Pull Request
        |
        v
4. Validation
        |
        v
5. Merge
        |
        v
6. Argo CD refresh
        |
        v
7. Desired state calculated
        |
        v
8. Compare with EKS
        |
        v
9. Sync
        |
        v
10. Kubernetes API
        |
        v
11. Kubernetes controllers
        |
        v
12. Runtime
        |
        v
13. Health evaluation
```

---

# 42. What Argo CD Actually Sends to Kubernetes

Suppose Git contains:

```text
Helm chart
```

Argo CD may render it into Kubernetes manifests.

Conceptually:

```text
Git
 |
 v
Helm Chart
 |
 v
Rendered Kubernetes YAML
 |
 v
Argo CD
 |
 v
Kubernetes API
```

Kubernetes receives Kubernetes resources, not the concept of "GitOps."

This distinction is important.

---

# 43. GitOps with Helm

Helm provides:

- Packaging
- Templating
- Values
- Release-oriented structure

Argo CD provides:

- Git integration
- Desired-state comparison
- Sync
- Drift detection
- Health
- Reconciliation

Architecture:

```text
Git
 |
 v
Helm Chart + Values
 |
 v
Argo CD renders
 |
 v
Kubernetes manifests
 |
 v
EKS
```

Detailed Helm behavior is covered in File 08.

---

# 44. GitOps with Kustomize

Kustomize provides:

- Base resources
- Overlays
- Environment-specific patches
- Resource customization

Example:

```text
base/
├── deployment.yaml
└── service.yaml

overlays/
├── dev/
├── qa/
└── prod/
```

Argo CD can render the appropriate Kustomize configuration.

---

# 45. Helm vs Kustomize in GitOps

| Feature | Helm | Kustomize |
|---|---|---|
| Packaging | Strong | Limited |
| Templating | Strong | Patch/overlay model |
| Values | Yes | Overlays |
| Reuse | Charts | Bases |
| Complexity | Can become high | Often straightforward |
| Argo CD support | Native | Native |

The choice depends on application/platform requirements.

---

# 46. Resource Ownership

GitOps requires clear ownership.

Example:

```text
Argo CD Application
       |
       +--> Deployment
       +--> Service
       +--> ConfigMap
       +--> HPA
```

Kubernetes creates:

```text
ReplicaSet
Pods
```

Another controller may create:

```text
ALB
```

This means not every runtime object should necessarily be independently managed by GitOps.

---

# 47. Generated Resources

Some resources are generated by controllers.

Example:

```text
Deployment
   |
   v
ReplicaSet
   |
   v
Pod
```

Argo CD may display them in the resource tree, but their lifecycle is controlled through their owner.

Do not manually add generated resources to Git just because they appear in Kubernetes.

---

# 48. Owner References

Kubernetes resources can have owner relationships.

For example:

```text
Deployment
   |
   v
ReplicaSet
   |
   v
Pod
```

This allows Kubernetes to understand resource ownership.

When troubleshooting GitOps, use ownership to determine which controller should be changed.

---

# 49. Kubernetes Labels

Labels are critical for resource selection.

Example:

```yaml
labels:
  app.kubernetes.io/name: cart
  app.kubernetes.io/part-of: roboshop
  app.kubernetes.io/component: backend
```

Services select Pods through labels:

```yaml
selector:
  app.kubernetes.io/name: cart
```

A mismatch can cause:

```text
Deployment healthy
Pods running
Service has no endpoints
```

This is a common production issue.

---

# 50. GitOps Troubleshooting: Service Has No Endpoints

Check:

```bash
kubectl get pods -n roboshop --show-labels
kubectl get service cart -n roboshop
kubectl describe service cart -n roboshop
kubectl get endpoints cart -n roboshop
```

Then compare:

```yaml
Deployment pod labels
```

with:

```yaml
Service selector
```

If they do not match:

```text
Pods exist
but
Service selects nothing
```

Argo CD may correctly report the manifests as Synced.

The application can still be unhealthy.

---

# 51. GitOps Troubleshooting: Pods Not Starting

Start with:

```bash
kubectl get pods -n roboshop
```

Then:

```bash
kubectl describe pod <pod> -n roboshop
```

Check events:

```bash
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Then logs:

```bash
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
```

Possible causes:

- ImagePullBackOff
- CrashLoopBackOff
- OOMKilled
- Failed scheduling
- Missing Secret
- Missing ConfigMap
- Probe failure
- Permission problem

---

# 52. GitOps Troubleshooting: ImagePullBackOff

If Git declares:

```text
cart:2026.08.19-abc123
```

but the image does not exist in ECR:

```text
Pod
 |
 v
ImagePullBackOff
```

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Failed to pull image
```

Then verify:

- ECR repository exists
- Tag exists
- Image URI is correct
- Node/pod has required ECR access
- Network access is available
- Image architecture is compatible

Argo CD may still report:

```text
Synced
```

because Git matches the cluster configuration.

---

# 53. GitOps Troubleshooting: CrashLoopBackOff

Check:

```bash
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
kubectl describe pod <pod> -n roboshop
```

Potential causes:

- Application startup failure
- Bad configuration
- Missing dependency
- Invalid environment variable
- Memory exhaustion
- Incorrect command
- Probe failure

Again:

```text
Argo CD Sync = successful
```

does not mean:

```text
Application = healthy
```

---

# 54. GitOps Troubleshooting: OOMKilled

Check:

```bash
kubectl get pod <pod> -n roboshop -o json
```

Look for:

```text
reason: OOMKilled
```

Then evaluate:

```yaml
resources:
  requests:
    memory: ...
  limits:
    memory: ...
```

If the limit is too low, the container can be killed.

GitOps provides the advantage that resource configuration is version controlled.

---

# 55. GitOps Troubleshooting: Probe Failure

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Readiness probe failed
Liveness probe failed
Startup probe failed
```

Validate:

```text
Path
Port
Scheme
Initial delay
Timeout
Threshold
Application startup time
```

A probe configuration should reflect real application behavior.

Do not use an aggressive liveness probe that kills a legitimately slow-starting application.

---

# 56. GitOps Troubleshooting: Scheduling Failure

If Pods remain Pending:

```bash
kubectl describe pod <pod> -n roboshop
```

Check for:

```text
Insufficient CPU
Insufficient memory
Node selector mismatch
Taints/tolerations
Affinity rules
PVC issues
```

GitOps may have correctly synchronized the Deployment.

The Kubernetes scheduler may still be unable to place the Pod.

---

# 57. GitOps Troubleshooting: ConfigMap Missing

If a Pod references:

```yaml
envFrom:
  - configMapRef:
      name: cart-config
```

but the ConfigMap does not exist:

```text
Pod startup may fail
```

Check:

```bash
kubectl get configmap -n roboshop
kubectl describe deployment cart -n roboshop
```

In GitOps, ensure the ConfigMap is part of the application desired state or provided through an approved platform mechanism.

---

# 58. GitOps Troubleshooting: Secret Missing

Check:

```bash
kubectl get secret -n roboshop
kubectl describe pod <pod> -n roboshop
```

Do not print sensitive Secret values unnecessarily.

Verify the secret-management system is functioning.

Possible causes:

- External Secrets failure
- Wrong secret name
- Wrong namespace
- Missing secret key
- IAM permission issue
- Secret synchronization failure

---

# 59. GitOps Troubleshooting: Ingress/ALB Failure

Start with:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Then check the AWS Load Balancer Controller.

Possible causes:

- Invalid annotation
- Missing subnet tags
- Security group issue
- IAM permission problem
- Target health failure
- Service selector mismatch
- Incorrect IngressClass
- Certificate issue

The layered model is:

```text
Git
 |
 v
Argo CD
 |
 v
Ingress
 |
 v
AWS Load Balancer Controller
 |
 v
ALB
 |
 v
Target Service
 |
 v
Pods
```

Troubleshoot layer by layer.

---

# 60. GitOps Troubleshooting: HPA Not Scaling

Check:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa cart -n roboshop
```

Then:

```bash
kubectl get deployment cart -n roboshop
```

Possible causes:

- Metrics unavailable
- Incorrect resource requests
- Incorrect HPA target
- Application load insufficient
- HPA configuration error

GitOps should manage the HPA specification, while Kubernetes handles runtime scaling.

---

# 61. GitOps Troubleshooting: Argo CD Says OutOfSync

The first step is:

```bash
argocd app get <application>
```

Then:

```bash
argocd app diff <application>
```

Determine:

```text
What differs?
```

Then inspect the resource:

```bash
kubectl get <resource> <name> -n <namespace> -o yaml
```

Possible causes:

- Manual modification
- Generated field
- Controller-managed field
- Helm rendering difference
- Wrong target revision
- Wrong values
- Resource ownership conflict
- Expected dynamic value

Do not blindly click Sync without understanding the difference.

---

# 62. GitOps Troubleshooting: Sync Failed

Check:

```bash
argocd app get <application>
```

Then:

```bash
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Look for:

```text
Permission denied
Invalid manifest
Admission rejected
Missing resource
Invalid field
API unavailable
```

Then inspect the failing resource.

---

# 63. GitOps Troubleshooting: Render Failure

If Helm/Kustomize rendering fails, the cluster may never receive the intended resources.

For Helm:

```bash
helm lint ./chart
helm template ./chart
```

Check:

- values files
- template syntax
- missing values
- invalid Kubernetes fields
- chart dependencies

Argo CD's repository/rendering components will be covered in detail later.

---

# 64. GitOps Troubleshooting: Kubernetes API Failure

If Argo CD cannot communicate with the cluster, investigate:

```text
API endpoint
Network
Authentication
Authorization
Certificate
Cluster health
IAM/EKS access
```

Commands:

```bash
argocd cluster list
```

Then use appropriate Kubernetes connectivity tests from the relevant environment.

---

# 65. GitOps Troubleshooting: Permission Failure

A resource may fail to sync because Argo CD lacks permission.

Conceptually:

```text
Argo CD
   |
   v
Kubernetes API
   |
   v
RBAC
   |
   X
Forbidden
```

Typical error:

```text
forbidden
```

Investigate:

- ServiceAccount
- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- AppProject restrictions
- Destination restrictions

---

# 66. Production GitOps YAML: Namespace

A production-oriented namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    app.kubernetes.io/part-of: roboshop
    environment: prod
    managed-by: argocd
```

The labels help identify ownership and environment.

Additional controls such as ResourceQuota, LimitRange, and NetworkPolicy can be added.

---

# 67. Production GitOps YAML: ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-config
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
data:
  LOG_LEVEL: "INFO"
  CART_PORT: "8080"
```

Keep non-sensitive configuration here.

---

# 68. Production GitOps YAML: Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: cart
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
```

The Service selects Pods using labels.

---

# 69. Production GitOps YAML: HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cart
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cart
  minReplicas: 3
  maxReplicas: 10
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

The exact values should be validated through performance testing.

---

# 70. Production GitOps YAML: ALB Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

This assumes the AWS Load Balancer Controller is already installed and configured.

---

# 71. Production GitOps YAML: Deployment + Service

A simplified complete application set:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart:2026.08.19-abc123
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          readinessProbe:
            httpGet:
              path: /health
              port: http
          livenessProbe:
            httpGet:
              path: /health
              port: http
---
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: cart
  ports:
    - name: http
      port: 80
      targetPort: http
```

This can be adapted to Helm later.

---

# 72. GitOps and Rolling Updates

A Deployment can use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This expresses a desired rollout strategy.

The Kubernetes Deployment controller performs the rollout.

Argo CD manages the desired Deployment configuration.

---

# 73. Why maxUnavailable = 0 Can Be Useful

For a highly available service:

```text
Current replicas = 3
```

using:

```yaml
maxUnavailable: 0
```

requests that existing availability be preserved during rollout, subject to application and cluster constraints.

This does not magically guarantee zero downtime.

The application must also have:

- Correct readiness probes
- Enough capacity
- Compatible versions
- Proper termination behavior
- Adequate node resources

---

# 74. GitOps and Pod Disruption

Production workloads may also use:

```text
PodDisruptionBudget
```

to protect availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: cart
  namespace: roboshop
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: cart
```

This is another Kubernetes resource that can be managed by GitOps.

---

# 75. GitOps and NetworkPolicy

NetworkPolicies can define allowed traffic.

Example concept:

```text
cart
 |
 +--> allowed from frontend
 |
 +--> allowed to database
 |
 +--> denied from unrelated workloads
```

NetworkPolicy can therefore become part of GitOps security configuration.

The exact policy depends on the networking implementation and application architecture.

---

# 76. GitOps and ResourceQuota

For shared clusters, ResourceQuota can enforce namespace limits.

Example concept:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop
  namespace: roboshop
spec:
  requests.cpu: "10"
  requests.memory: 20Gi
  limits.cpu: "20"
  limits.memory: 40Gi
```

This can prevent one namespace from consuming uncontrolled resources.

---

# 77. GitOps and LimitRange

A LimitRange can provide default or minimum resource constraints.

Example:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: defaults
  namespace: roboshop
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
```

These policies can be managed through GitOps where appropriate.

---

# 78. GitOps and ServiceAccounts

Applications may need Kubernetes ServiceAccounts.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
```

The ServiceAccount can be associated with appropriate permissions.

For AWS access, an EKS workload identity mechanism can be used so the application does not need long-lived AWS keys.

---

# 79. GitOps and AWS Workload Identity

A production Pod may need access to AWS services.

Avoid:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

embedded in the container.

Prefer an AWS identity mechanism designed for workloads.

Conceptually:

```text
Pod
 |
 v
Kubernetes ServiceAccount
 |
 v
AWS IAM Role
 |
 v
AWS Service
```

This is separate from Argo CD's cluster-management permissions.

---

# 80. GitOps and Application Dependencies

RoboShop services may have dependencies.

Example:

```text
Frontend
   |
   +--> Catalog
   +--> Cart
   +--> User

Orders
   |
   +--> Cart
   +--> Payment
   +--> Inventory
```

GitOps should represent desired application resources, but startup ordering must be handled carefully.

Do not assume:

```text
Deployment created
=
Application dependency ready
```

Health checks and application retry behavior are still required.

---

# 81. GitOps and Kubernetes Service Discovery

Kubernetes Services provide stable names.

Example:

```text
cart.roboshop.svc.cluster.local
```

A service can discover another service without knowing Pod IPs.

GitOps can manage the Service configuration.

Kubernetes manages the runtime endpoint mapping.

---

# 82. GitOps and Config Separation

Separate configuration into:

```text
Application image
Environment configuration
Secret configuration
Infrastructure configuration
```

For example:

```text
Image:
cart:2026.08.19-abc123

Config:
LOG_LEVEL=INFO

Secret:
database password

Infrastructure:
EKS / ALB
```

Each should have a clear owner and source of truth.

---

# 83. GitOps and Immutable Infrastructure Concepts

GitOps and immutable infrastructure share an important philosophy:

```text
Prefer replacing desired state
over manually modifying runtime state.
```

For example:

```text
Old Deployment revision
        |
        v
New Deployment revision
```

rather than:

```text
SSH to node
Edit configuration manually
Restart process
```

Kubernetes already supports declarative replacement patterns.

---

# 84. GitOps and Node Management

GitOps typically manages workloads, not every node-level operation.

For EKS:

```text
Terraform / AWS
    |
    v
Node infrastructure

Argo CD
    |
    v
Application workloads
```

Node groups, networking, and cluster infrastructure may be managed separately.

This prevents infrastructure and application controllers from fighting.

---

# 85. GitOps and Cluster Add-ons

Some Kubernetes add-ons can be managed by Argo CD.

Examples:

```text
AWS Load Balancer Controller
Prometheus
Grafana
ELK components
External Secrets
Policy controllers
```

This allows the platform itself to become GitOps-managed.

Example:

```text
platform/
├── ingress-controller/
├── monitoring/
├── secrets/
└── policy/
```

---

# 86. Platform GitOps

A mature EKS environment may have:

```text
platform-root
 |
 +--> AWS Load Balancer Controller
 +--> External Secrets
 +--> Prometheus
 +--> Grafana
 +--> ELK
 +--> namespaces
 +--> policies
```

Then:

```text
application-root
 |
 +--> cart
 +--> catalog
 +--> orders
 +--> payment
```

This separation becomes useful when implementing App of Apps.

---

# 87. GitOps Bootstrap Sequence for EKS

A practical sequence:

```text
1. Terraform creates VPC
        |
        v
2. Terraform creates EKS
        |
        v
3. Install Argo CD
        |
        v
4. Configure Git repository
        |
        v
5. Create root Application
        |
        v
6. Deploy platform components
        |
        v
7. Deploy application Applications
        |
        v
8. Deploy RoboShop
        |
        v
9. Monitor health
```

Later files will convert this into production implementation.

---

# 88. GitOps and Kubernetes Validation

Before merging a Kubernetes change, validate:

```text
YAML syntax
Kubernetes schema
Helm rendering
Kustomize rendering
Security policy
Resource requirements
Image availability
```

Example commands:

```bash
kubectl apply --dry-run=client -f deployment.yaml
```

For Helm:

```bash
helm lint ./chart
helm template ./chart
```

These are pre-merge safeguards.

---

# 89. GitOps and Server-Side Validation

Depending on the workflow, validation can also be performed against a Kubernetes API server.

The exact validation mechanism depends on the pipeline and target cluster.

The important idea is:

```text
Validate before production
```

rather than discovering configuration errors only during Argo CD synchronization.

---

# 90. GitOps and Manifest Quality

A production manifest should be:

- Explicit
- Consistent
- Reviewable
- Secure
- Environment-aware
- Observable
- Reproducible

Avoid unnecessary fields copied from:

```bash
kubectl get deployment -o yaml
```

because runtime-generated fields can pollute Git.

Do not blindly commit the entire live object.

---

# 91. Desired Manifest vs Live Manifest

Live Kubernetes objects contain generated information such as:

```text
resourceVersion
uid
managedFields
creationTimestamp
status
```

These generally should not be copied into Git as desired configuration.

Desired manifests should focus on fields the team intends to manage.

---

# 92. GitOps and managedFields

Kubernetes may record field-management metadata.

This can cause confusing differences in some workflows.

When Argo CD reports unexpected differences, determine whether:

```text
A controller owns the field
```

rather than assuming Git is wrong.

This becomes especially important with:

- HPA
- Operators
- Mutating admission
- AWS controllers

---

# 93. Mutating Admission and GitOps

A mutating admission controller may modify a resource after Argo CD submits it.

Example:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes API
 |
 v
Mutating admission
 |
 +--> inject field
 |
 v
Stored resource
```

Now:

```text
Git != live object
```

This can be expected.

Argo CD's diff configuration can help distinguish intentional mutation from unwanted drift.

---

# 94. GitOps and Operators

Operators can manage complex applications.

Example:

```text
Git
 |
 v
Argo CD
 |
 v
Custom Resource
 |
 v
Operator
 |
 v
Complex application resources
```

Argo CD manages the Custom Resource.

The operator manages the resulting resources.

This is another example of layered ownership.

---

# 95. GitOps and Custom Resource Definitions

CRDs extend Kubernetes.

Examples include:

```text
Application
ApplicationSet
Rollout
ExternalSecret
ServiceMonitor
```

GitOps can manage CRDs and their custom resources.

However, CRD installation ordering matters.

For example:

```text
CRD
 |
 v
Custom Resource
```

The CRD must exist before the API server can accept the custom resource.

This is why sync ordering and platform bootstrapping become important.

---

# 96. GitOps Dependency Ordering

Some resources depend on others.

Example:

```text
Namespace
   |
   v
CRD
   |
   v
Custom Resource
   |
   v
Application
```

Argo CD later provides mechanisms such as:

- Sync waves
- Hooks
- Health checks

to handle more complex ordering.

---

# 97. GitOps with Kubernetes: Complete Flow

The complete conceptual flow is:

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   +--> Test
   +--> Security
   |
   v
ECR
   |
   v
GitOps Git
   |
   v
Argo CD
   |
   +--> Fetch source
   +--> Render
   +--> Compare
   +--> Sync
   |
   v
Kubernetes API
   |
   +--> Admission
   +--> Authorization
   |
   v
Kubernetes Controllers
   |
   +--> Deployment
   +--> HPA
   +--> Service
   +--> Ingress
   |
   v
EKS Runtime
   |
   v
ALB / Users
```

---

# 98. Production Example: Cart Deployment Change

Suppose the current production version is:

```text
cart:1.8.0
```

CI builds:

```text
cart:1.9.0
```

and pushes it to ECR.

The GitOps repository changes:

```yaml
image:
  tag: "1.9.0"
```

Then:

```text
Git commit
   |
   v
Argo CD refresh
   |
   v
OutOfSync
   |
   v
Sync
   |
   v
Deployment updated
   |
   v
ReplicaSet rollout
   |
   v
Pods
   |
   v
Readiness
   |
   v
Healthy
```

This is a complete GitOps deployment.

---

# 99. Production Example: Manual Drift

Git:

```text
cart:1.9.0
replicas:3
```

Engineer changes:

```bash
kubectl scale deployment cart --replicas=1 -n roboshop
```

Actual:

```text
replicas:1
```

Argo CD sees:

```text
desired=3
actual=1
```

If self-healing is enabled:

```text
Argo CD
   |
   v
restore replicas=3
```

If self-healing is disabled:

```text
Application remains OutOfSync
```

until a sync is performed.

---

# 100. Production Example: Bad Image

Git:

```text
cart:2.0.0
```

ECR:

```text
cart:2.0.0 exists
```

Argo CD sync:

```text
successful
```

But the application crashes.

Result:

```text
Sync = Synced
Health = Degraded
```

Troubleshooting:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

The problem is application/runtime behavior, not necessarily GitOps synchronization.

---

# 101. Production Example: Invalid Manifest

Suppose Git contains:

```yaml
spec:
  replicas: "three"
```

instead of:

```yaml
spec:
  replicas: 3
```

Argo CD may fail to apply/render the resource.

Better practice:

```text
PR
 |
 v
CI validation
 |
 v
Reject before merge
```

This is why GitOps requires configuration validation.

---

# 102. Production Example: ALB Failure

Git declares:

```text
Ingress
host = shop.example.com
```

Argo CD successfully synchronizes the Ingress.

But ALB is not created.

Investigate:

```text
Argo CD
   |
   | Synced?
   v
Ingress
   |
   v
AWS Load Balancer Controller
   |
   | Events / logs
   v
AWS ALB
```

Commands:

```bash
kubectl describe ingress roboshop -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
kubectl get pods -n kube-system
```

Then inspect the AWS Load Balancer Controller logs.

---

# 103. Production Example: HPA Failure

Git defines:

```yaml
minReplicas: 3
maxReplicas: 10
```

Argo CD sync succeeds.

But HPA does not scale.

Investigate:

```bash
kubectl get hpa cart -n roboshop
kubectl describe hpa cart -n roboshop
```

Possible cause:

```text
Metrics unavailable
```

This is a runtime platform issue rather than necessarily an Argo CD issue.

---

# 104. GitOps Command Set

Useful Kubernetes commands:

```bash
kubectl get namespaces
kubectl get deployments -A
kubectl get pods -A
kubectl get services -A
kubectl get ingress -A
kubectl get hpa -A
kubectl get configmaps -A
kubectl get secrets -A
kubectl get events -A
```

Inspect:

```bash
kubectl describe deployment <name> -n <namespace>
kubectl describe pod <name> -n <namespace>
kubectl describe service <name> -n <namespace>
kubectl describe ingress <name> -n <namespace>
kubectl describe hpa <name> -n <namespace>
```

Logs:

```bash
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

---

# 105. Useful Argo CD Commands

Later files will explain these in depth, but the core commands are:

```bash
argocd app list
argocd app get <application>
argocd app diff <application>
argocd app sync <application>
argocd app history <application>
argocd app rollback <application> <revision>
argocd cluster list
argocd repo list
```

Use the correct context and application name before executing changes.

---

# 106. Command: kubectl get applications

Argo CD Applications are Kubernetes custom resources.

Depending on installation/version, a common command is:

```bash
kubectl get applications -A
```

This helps verify:

```text
Application existence
Namespace
Sync/health information
```

The exact CRD group and output depend on the Argo CD installation.

---

# 107. Command: argocd app get

Example:

```bash
argocd app get roboshop-cart
```

This is one of the most useful commands for investigating:

- Sync status
- Health
- Repository
- Revision
- Destination
- Resources
- Conditions

---

# 108. Command: argocd app diff

Example:

```bash
argocd app diff roboshop-cart
```

Use it when:

```text
Application = OutOfSync
```

It helps answer:

> What exactly differs between desired and live state?

This should usually be checked before blindly synchronizing.

---

# 109. Command: argocd app sync

Example:

```bash
argocd app sync roboshop-cart
```

This triggers synchronization.

Production rule:

> Understand the pending difference before executing a production sync.

---

# 110. Command: argocd cluster list

Example:

```bash
argocd cluster list
```

Useful for multi-cluster troubleshooting.

It helps identify:

```text
Cluster
Server
Connection status
```

If a target cluster is unavailable, investigate the cluster registration and authentication path.

---

# 111. Command: argocd repo list

Example:

```bash
argocd repo list
```

Useful when investigating repository connectivity.

Possible issues:

```text
Authentication
SSH key
HTTPS token
Certificate
Network
Repository URL
Permissions
```

---

# 112. Production Kubernetes File Organization

For raw manifests:

```text
manifests/
└── cart/
    ├── namespace.yaml
    ├── configmap.yaml
    ├── deployment.yaml
    ├── service.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    └── ingress.yaml
```

For Helm:

```text
helm/
└── cart/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-qa.yaml
    ├── values-prod.yaml
    └── templates/
```

---

# 113. Environment-Specific Helm Values

Example:

```text
helm/cart/
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
└── values-prod.yaml
```

Common values:

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart
  tag: "2026.08.19-abc123"
```

Production can override:

```yaml
replicaCount: 5

resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
```

The exact values should come from workload requirements.

---

# 114. GitOps and Configuration Promotion

A promotion might change only:

```yaml
image:
  tag: "2026.08.19-abc123"
```

while leaving:

```text
resources
probes
service
ingress
security
```

unchanged.

This is useful because the artifact is promoted while environment configuration remains controlled.

---

# 115. GitOps and Production Configuration Review

A production PR might look like:

```diff
image:
- tag: "2026.08.18-def456"
+ tag: "2026.08.19-abc123"
```

This is easy to review.

A reviewer can answer:

```text
What changed?
```

without inspecting a large deployment script.

This is one of GitOps's practical strengths.

---

# 116. GitOps and Deployment Observability

After synchronization, validate:

```bash
argocd app get roboshop-cart
kubectl rollout status deployment/cart -n roboshop
kubectl get pods -n roboshop
```

Then inspect application metrics/logs.

For example:

```text
Argo CD = Synced
Deployment = Available
Pods = Ready
ALB = Healthy
Application = Serving traffic
```

A deployment is not complete until the runtime is healthy.

---

# 117. GitOps and Rollout Status

Kubernetes provides:

```bash
kubectl rollout status deployment/cart -n roboshop
```

This checks Deployment rollout progress.

You can also inspect:

```bash
kubectl rollout history deployment/cart -n roboshop
```

Argo CD provides its own application-level history and sync information.

Both perspectives are useful.

---

# 118. GitOps and Rollout Failure

Suppose Git changes:

```text
cart:1.9.0
```

Argo CD syncs successfully.

But:

```text
Pods never become Ready
```

The Deployment rollout may stall.

Check:

```bash
kubectl rollout status deployment/cart -n roboshop
kubectl describe deployment cart -n roboshop
kubectl get pods -n roboshop
```

Then inspect Pod failures.

This is a runtime rollout problem.

---

# 119. GitOps and Revision History

There are multiple histories:

```text
Git commit history
      +
Argo CD application history
      +
Kubernetes Deployment rollout history
```

Together they help reconstruct a production incident.

For example:

```text
Git commit abc123
       |
       v
Argo CD sync
       |
       v
Deployment revision
       |
       v
Pod failure
```

This is powerful for incident investigation.

---

# 120. Production Incident Investigation

When an application breaks after deployment:

```text
1. Check Argo CD
2. Identify Git revision
3. Identify deployment change
4. Check Kubernetes rollout
5. Check Pod status
6. Check logs/events
7. Check metrics
8. Check dependencies
9. Determine rollback or fix
```

Commands:

```bash
argocd app get <application>
argocd app history <application>
kubectl rollout history deployment/<name> -n <namespace>
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl get events -n <namespace>
```

---

# 121. GitOps and Disaster Recovery of Kubernetes Configuration

If EKS is recreated:

```text
New EKS
   |
   v
Argo CD
   |
   v
GitOps Repository
   |
   v
Kubernetes resources
```

The desired Kubernetes configuration can be recreated.

But verify separately:

```text
ECR
Secrets
RDS
S3
DNS
Certificates
Persistent volumes
External systems
```

---

# 122. Production DR Checklist for GitOps

- [ ] Git repository is recoverable.
- [ ] Argo CD installation method is documented.
- [ ] Argo CD configuration is recoverable.
- [ ] Cluster registration process is documented.
- [ ] EKS infrastructure can be recreated.
- [ ] ECR images are retained according to policy.
- [ ] Secrets can be recovered.
- [ ] Database backups are tested.
- [ ] DNS recovery is documented.
- [ ] ALB/certificate configuration is reproducible.
- [ ] Recovery owners are defined.
- [ ] Recovery time objectives are known.
- [ ] Recovery point objectives are known.
- [ ] DR exercises are performed.

---

# 123. GitOps and Production Security Checklist

- [ ] No plaintext production secrets in Git.
- [ ] Git branches are protected.
- [ ] Pull requests require appropriate approvals.
- [ ] CI uses least privilege.
- [ ] ECR access is restricted.
- [ ] Argo CD repository access is restricted.
- [ ] Argo CD cluster access is restricted.
- [ ] AppProjects define boundaries.
- [ ] Argo CD RBAC is configured.
- [ ] Kubernetes RBAC follows least privilege.
- [ ] Workloads use appropriate security contexts.
- [ ] Images are scanned.
- [ ] Immutable images are used.
- [ ] AWS workload identities avoid long-lived keys.
- [ ] Network boundaries are documented.

---

# 124. GitOps and Production Availability Checklist

- [ ] Kubernetes workloads have multiple replicas where required.
- [ ] Readiness probes are configured.
- [ ] Liveness/startup probes are appropriate.
- [ ] Resource requests/limits are defined.
- [ ] HPA is configured where needed.
- [ ] PDB is configured where appropriate.
- [ ] ALB health checks are understood.
- [ ] Argo CD HA is considered.
- [ ] Git repository availability is considered.
- [ ] ECR availability is considered.
- [ ] Application dependencies are monitored.
- [ ] Observability covers application and GitOps layers.

---

# 125. Interview: How Does Argo CD Interact With Kubernetes?

### Answer

> Argo CD communicates with the Kubernetes API server. It reads the desired application configuration from Git, renders the configuration when necessary, compares desired resources with the live resources returned by the Kubernetes API, and performs synchronization through the API. Kubernetes then handles the runtime reconciliation through its own controllers, such as the Deployment and HPA controllers.

---

# 126. Interview: Explain the Layered Control Loops

### Answer

> There are multiple reconciliation layers. GitOps reconciliation compares Git desired state with Kubernetes resources. Kubernetes controllers then reconcile those resources into runtime state. In EKS, cloud controllers can add another layer—for example, the AWS Load Balancer Controller watches Kubernetes Ingress resources and reconciles them into AWS ALBs. Therefore a GitOps deployment can involve Git, Argo CD, Kubernetes controllers, and AWS controllers.

---

# 127. Interview: What Happens If Argo CD Syncs Successfully but Pods Are Not Ready?

### Answer

> Successful synchronization means Argo CD was able to apply the desired Kubernetes resources. It does not mean the application is healthy. I would inspect Deployment rollout status, Pod status, events, logs, probes, resource limits, image availability, configuration, and dependencies. I would use commands such as `kubectl get pods`, `kubectl describe pod`, `kubectl logs`, and `kubectl get events`.

---

# 128. Interview: Why Should You Not Commit `kubectl get -o yaml` Directly?

### Answer

> A live Kubernetes object contains runtime-generated fields such as UID, resourceVersion, managedFields, timestamps, and status. Those fields are not normally part of the desired configuration. GitOps manifests should contain the fields the team intends to manage rather than blindly copying the entire live object.

---

# 129. Interview: How Does GitOps Work With HPA?

### Answer

> GitOps can manage the HPA specification, such as minimum replicas, maximum replicas, behavior, and metric targets. The Kubernetes HPA controller then dynamically changes the Deployment replica count at runtime. Therefore not every runtime replica change should be considered unwanted drift. The GitOps system must account for fields controlled by the HPA.

---

# 130. Interview: How Does GitOps Work With AWS ALB?

### Answer

> Argo CD manages the Kubernetes Ingress resource declared in Git. The AWS Load Balancer Controller watches that Ingress and reconciles it into an AWS ALB. So the chain is Git, Argo CD, Kubernetes Ingress, AWS Load Balancer Controller, and ALB. If ALB provisioning fails, I would troubleshoot each layer rather than assuming Argo CD is the root cause.

---

# 131. Interview: What Is the Difference Between Deployment and Pod Ownership?

### Answer

> The Deployment is the higher-level desired workload resource. It manages ReplicaSets, and ReplicaSets manage Pods. GitOps generally manages the Deployment. Kubernetes creates and manages the generated ReplicaSet and Pods. Understanding this ownership chain helps prevent manually managing generated resources.

---

# 132. Interview: What Is Resource Ownership in GitOps?

### Answer

> Resource ownership means defining which system is authoritative for a resource or field. For example, Terraform may own an EKS cluster, Argo CD may own a Deployment, Kubernetes may own ReplicaSets, and the AWS Load Balancer Controller may own the resulting ALB. Clear ownership prevents multiple controllers from repeatedly changing the same state.

---

# 133. Interview Scenario: GitOps Application Is Synced but ALB Is Missing

### Answer

I would investigate:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Then inspect the AWS Load Balancer Controller:

```bash
kubectl get pods -n kube-system
kubectl logs <aws-load-balancer-controller-pod> -n kube-system
```

I would check:

- IngressClass
- Annotations
- Subnet discovery
- Security groups
- IAM permissions
- Controller health
- Target service
- AWS account resources

The key is that:

```text
Argo CD Synced
```

only proves the Kubernetes desired resource was reconciled.

---

# 134. Interview Scenario: Service Has No Traffic

### Answer

I would check:

```bash
kubectl get service cart -n roboshop
kubectl get endpoints cart -n roboshop
kubectl get pods -n roboshop --show-labels
```

Then compare:

```text
Service selector
```

with:

```text
Pod labels
```

If the Service selector does not match Pod labels, the Service can have no endpoints even though the Pods are healthy.

---

# 135. Interview Scenario: Pods Are Pending

### Answer

I would run:

```bash
kubectl describe pod <pod> -n roboshop
```

and inspect scheduler events.

I would check:

- CPU/memory requests
- Node capacity
- Taints/tolerations
- Node selectors
- Affinity
- PVC availability
- Scheduling constraints

This is a Kubernetes scheduling problem, not automatically an Argo CD problem.

---

# 136. Interview Scenario: HPA Shows Unknown Metrics

### Answer

I would check:

```bash
kubectl describe hpa cart -n roboshop
```

and investigate the metrics provider.

I would verify:

- Metrics availability
- Resource requests
- HPA configuration
- Kubernetes metrics pipeline

If the HPA specification matches Git, Argo CD may correctly report it as synchronized even though runtime scaling is not functioning.

---

# 137. Practical Exercise: Build a GitOps Kubernetes Application

Create:

```text
gitops-kubernetes-lab/
└── cart/
    ├── namespace.yaml
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── hpa.yaml
```

The desired flow:

```text
Git
 |
 v
Argo CD
 |
 v
EKS
 |
 +--> Namespace
 +--> Deployment
 +--> Service
 +--> ConfigMap
 +--> HPA
```

Observe each resource after synchronization.

---

# 138. Practical Exercise: Test Drift

After synchronization:

```bash
kubectl scale deployment cart \
  --replicas=1 \
  -n roboshop
```

Then inspect:

```bash
argocd app get roboshop-cart
```

and:

```bash
argocd app diff roboshop-cart
```

Observe the difference between:

```text
Git desired state
```

and:

```text
Kubernetes live state
```

Later, repeat this with self-healing enabled.

---

# 139. Practical Exercise: Test Application Failure

Deploy an intentionally invalid application image:

```text
cart:does-not-exist
```

Observe:

```text
Argo CD
  |
  +--> Sync may succeed
  |
  v
Kubernetes
  |
  v
ImagePullBackOff
```

This demonstrates:

```text
Sync status != runtime health
```

Always perform such exercises in a test environment, never production.

---

# 140. Practical Exercise: Test Probe Failure

Configure an incorrect readiness path:

```yaml
readinessProbe:
  httpGet:
    path: /wrong-health
    port: 8080
```

Observe:

```text
Pod Running
Readiness = false
Service traffic = unavailable
```

Then fix the Git configuration and let GitOps reconcile the correction.

---

# 141. Practical Exercise: Test Resource Limits

Temporarily configure a low memory limit in a lab:

```yaml
resources:
  limits:
    memory: "64Mi"
```

Run a workload that exceeds it.

Observe:

```text
OOMKilled
```

Then update the GitOps configuration and reconcile.

This teaches how GitOps can manage operational tuning.

---

# 142. Practical Exercise: Test ALB Ingress

Create an Ingress:

```text
Ingress
 |
 v
AWS Load Balancer Controller
 |
 v
ALB
```

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Then verify ALB health and target registration in AWS.

---

# 143. Practical Exercise: Test HPA

Deploy a workload with:

```text
CPU request
HPA minReplicas
HPA maxReplicas
```

Generate controlled test load.

Observe:

```text
CPU increases
   |
   v
HPA scales replicas
```

Then remember:

```text
HPA runtime replicas
```

are dynamic and should not automatically be treated as unwanted Git drift.

---

# 144. Production Kubernetes GitOps Example Structure

A mature application can eventually use:

```text
cart/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   └── kustomization.yaml
│
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── qa/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

Or Helm:

```text
cart/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
├── values-prod.yaml
└── templates/
```

The choice depends on the team's deployment model.

---

# 145. Production Design Recommendation

For the RoboShop project, a strong learning architecture is:

```text
Application source
        |
        v
CI
        |
        v
ECR
        |
        v
GitOps repository
        |
        v
Helm
        |
        v
Argo CD
        |
        v
EKS
        |
        +--> Deployment
        +--> Service
        +--> ConfigMap
        +--> HPA
        +--> PDB
        +--> Ingress
        |
        v
AWS ALB
```

This integrates the user's existing Kubernetes, Helm, AWS, CI/CD, and GitOps skills.

---

# 146. Final Architecture Summary

The key architecture is:

```text
                         Git
                          |
                          v
                 Desired Kubernetes State
                          |
                          v
                      Argo CD
                          |
                  +-------+-------+
                  |               |
               Compare           Sync
                  |               |
                  +-------+-------+
                          |
                          v
                   Kubernetes API
                          |
             +------------+------------+
             |            |            |
             v            v            v
        Deployment      Service      Ingress
             |                         |
             v                         v
          ReplicaSet          AWS Load Balancer Controller
             |                         |
             v                         v
            Pods                      ALB
```

And the Kubernetes runtime flow is:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Service
    |
    v
Ingress
    |
    v
ALB
```

---

# 147. Final Key Takeaways

1. Kubernetes is naturally suited to GitOps because it is declarative and controller-driven.
2. Argo CD interacts with Kubernetes through the Kubernetes API.
3. Kubernetes itself contains multiple reconciliation loops.
4. GitOps adds a higher-level reconciliation loop between Git and Kubernetes.
5. Kubernetes controllers transform declared resources into runtime resources.
6. Deployment manages ReplicaSets, which manage Pods.
7. Services provide stable networking to Pods.
8. ConfigMaps provide non-sensitive configuration.
9. Secrets require a secure external management strategy for production.
10. Ingress can be managed through GitOps.
11. The AWS Load Balancer Controller translates Kubernetes Ingress intent into AWS ALB resources.
12. HPA dynamically changes replicas and must be treated as a controller-managed resource.
13. Resource requests and limits are important production configuration.
14. Readiness, liveness, and startup probes affect application health and rollout behavior.
15. SecurityContext helps enforce safer workload execution.
16. GitOps should manage desired Kubernetes configuration rather than runtime-generated fields.
17. Resource ownership must be explicit.
18. Terraform and Argo CD should not fight over the same resources.
19. Helm and Kustomize are configuration/rendering mechanisms; Argo CD provides GitOps reconciliation.
20. Successful Argo CD synchronization does not guarantee application health.
21. Production troubleshooting should move layer by layer from Git to Argo CD to Kubernetes to Pods to application.
22. Immutable ECR image versions improve reproducibility.
23. GitOps can manage both application resources and platform add-ons.
24. Kubernetes namespaces provide logical isolation but are not equivalent to cluster isolation.
25. EKS cluster/account separation can reduce production blast radius.
26. The GitOps architecture should be designed around ownership, trust boundaries, failure domains, and recoverability.

---

# 148. Interview Summary

A concise senior-level explanation is:

> Kubernetes is a natural platform for GitOps because it uses declarative resources and controller-based reconciliation. In our EKS architecture, CI builds and validates the RoboShop application and pushes an immutable image to ECR. The desired Kubernetes configuration is stored in a GitOps repository. Argo CD reads that configuration, renders Helm or Kustomize when required, compares the desired resources with the live resources in EKS through the Kubernetes API, and synchronizes differences. Kubernetes controllers then reconcile Deployments, Services, HPAs, and other resources into runtime state. For external traffic, Argo CD manages the Kubernetes Ingress and the AWS Load Balancer Controller reconciles that into an ALB. This creates layered control loops where GitOps manages Kubernetes desired state and Kubernetes/cloud controllers manage runtime state.

---