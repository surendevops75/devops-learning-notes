# Section 5 - Service Troubleshooting Scenarios

---

## Scenario 51

### Interview Question

Your Pods are running successfully, but the application is not accessible through the Kubernetes Service. How would you troubleshoot it?

### Production-Level Answer

If the Pods are healthy but the Service is not working, the issue usually lies in the Service configuration, selectors, endpoints, or network routing.

Always verify whether the Service is forwarding traffic to the correct Pods.

### Approach

Check Services.

```bash
kubectl get svc
```

Describe the Service.

```bash
kubectl describe svc <service-name>
```

Verify Endpoints.

```bash
kubectl get endpoints <service-name>
```

Check

- Service selector
- Pod labels
- TargetPort
- Container Port
- Pod readiness

### Common Root Causes

- Selector mismatch
- Wrong TargetPort
- Pods not Ready
- Incorrect labels
- Endpoint not created

### Best Practices

- Standardize labels
- Verify selectors before deployment
- Test Service connectivity
- Monitor endpoint health

---

## Scenario 52

### Interview Question

A Service has **no Endpoints**. How would you investigate?

### Production-Level Answer

A Service without Endpoints has no Pods available to receive traffic.

This is almost always caused by a selector mismatch or unhealthy Pods.

### Approach

Check Endpoints.

```bash
kubectl get endpoints
```

Check Service selector.

```bash
kubectl describe svc <service-name>
```

Verify Pod labels.

```bash
kubectl get pods --show-labels
```

Compare

- Service selector
- Pod labels

### Common Root Causes

- Wrong selector
- Label typo
- Pods failed readiness
- Pods in different namespace

### Best Practices

- Keep labels consistent
- Use label standards
- Validate selectors during CI
- Monitor endpoint count

---

## Scenario 53

### Interview Question

Your Service is exposing the wrong port. Users receive connection failures. How would you troubleshoot it?

### Production-Level Answer

A Service must forward traffic from the Service Port to the correct TargetPort inside the container.

Incorrect port mappings prevent application connectivity.

### Approach

Review Service.

```bash
kubectl describe svc <service-name>
```

Verify

- Port
- TargetPort
- Container Port
- Application listening port

Check Deployment.

```bash
kubectl describe deployment <deployment-name>
```

### Common Root Causes

- Wrong TargetPort
- Wrong container port
- Application listening on different port
- Manifest error

### Best Practices

- Keep port naming consistent
- Test connectivity after deployment
- Validate manifests before release

---

## Scenario 54

### Interview Question

A ClusterIP Service is unreachable from another Pod in the same namespace. How would you investigate?

### Production-Level Answer

ClusterIP Services should be accessible from any Pod within the cluster.

If connectivity fails, investigate DNS, Endpoints, and Network Policies.

### Approach

Verify Service.

```bash
kubectl get svc
```

Test DNS.

```bash
nslookup <service-name>
```

Check Endpoints.

```bash
kubectl get endpoints
```

Verify

- Network Policies
- CoreDNS
- Service selector

### Common Root Causes

- DNS failure
- Endpoint missing
- Network Policy blocking traffic
- Selector mismatch

### Best Practices

- Monitor CoreDNS
- Validate Service discovery
- Test internal connectivity regularly

---

## Scenario 55

### Interview Question

A NodePort Service works from one Worker Node but not another. How would you troubleshoot it?

### Production-Level Answer

NodePort should expose the application on every Worker Node.

If only some nodes work, investigate kube-proxy, networking, and firewall rules.

### Approach

Check

- kube-proxy
- Node health
- Firewall
- Security Groups
- Node routing

Verify Service.

```bash
kubectl describe svc <service-name>
```

### Common Root Causes

- kube-proxy failure
- Firewall rule
- Node issue
- Network routing problem
- Security Group restriction

### Best Practices

- Monitor kube-proxy
- Standardize firewall rules
- Validate NodePort connectivity
- Prefer LoadBalancer in production

---

## Scenario 56

### Interview Question

Your LoadBalancer Service remains in the **Pending** state. What would you investigate?

### Production-Level Answer

A LoadBalancer Service depends on the cloud provider.

If it remains Pending, Kubernetes cannot provision the external load balancer.

### Approach

Check Service.

```bash
kubectl describe svc <service-name>
```

Review Events.

Verify

- Cloud Controller Manager
- IAM permissions
- Cloud provider integration
- Subnet configuration

### Common Root Causes

- Cloud integration issue
- Missing IAM permissions
- Incorrect subnet tags
- Cloud API failure

### Best Practices

- Monitor cloud controllers
- Validate IAM permissions
- Tag subnets correctly
- Review Service Events

---

## Scenario 57

### Interview Question

Users report intermittent connectivity to a Service. Some requests succeed while others fail. How would you investigate?

### Production-Level Answer

Intermittent failures usually indicate that some backend Pods are unhealthy while others continue serving requests.

### Approach

Verify

- Endpoints
- Ready Pods
- Service selector
- Application logs
- Readiness Probes

Check Endpoint list.

```bash
kubectl get endpoints <service-name>
```

### Common Root Causes

- One unhealthy Pod
- Readiness failure
- Pod restart loop
- Node issue
- Application crash

### Best Practices

- Enable readiness probes
- Monitor endpoint health
- Remove unhealthy Pods quickly
- Configure rolling updates

---

## Scenario 58

### Interview Question

Your application cannot resolve another Service using its DNS name. How would you troubleshoot it?

### Production-Level Answer

Kubernetes relies on CoreDNS for internal Service discovery.

If DNS resolution fails, investigate CoreDNS before the application.

### Approach

Test DNS.

```bash
nslookup <service-name>
```

Check CoreDNS.

```bash
kubectl get pods -n kube-system
```

Review CoreDNS logs.

```bash
kubectl logs -n kube-system <coredns-pod>
```

### Common Root Causes

- CoreDNS failure
- Wrong Service name
- Wrong namespace
- Network Policy
- DNS configuration issue

### Best Practices

- Monitor CoreDNS
- Use FQDN when required
- Validate DNS after deployment

---

## Scenario 59

### Interview Question

Two applications in different namespaces cannot communicate using Service names. How would you investigate?

### Production-Level Answer

A Service name alone only resolves within the same namespace.

Cross-namespace communication requires the fully qualified DNS name.

### Approach

Verify

- Namespace
- Service name
- DNS resolution
- Network Policies

Test

```bash
nslookup <service>.<namespace>.svc.cluster.local
```

### Common Root Causes

- Wrong namespace
- Incorrect DNS name
- Network Policy restriction
- Service missing

### Best Practices

- Use FQDN for cross-namespace communication
- Document namespace architecture
- Validate service discovery

---

## Scenario 60

### Interview Question

A Service is correctly configured, Endpoints exist, Pods are healthy, but users still cannot access the application. How would you troubleshoot it?

### Production-Level Answer

When the Service appears healthy, investigate the complete request path from the client to the application.

The issue may exist outside Kubernetes itself.

### Approach

Verify

- Client connectivity
- DNS
- Service
- Endpoints
- Pod readiness
- Application logs
- Ingress
- Load Balancer
- Firewall
- Security Groups

Test connectivity step by step until the failure point is identified.

### Common Root Causes

- External Load Balancer issue
- Ingress misconfiguration
- Firewall restriction
- Security Group blocking traffic
- Application listening on wrong port
- External DNS problem

### Best Practices

- Troubleshoot layer by layer
- Monitor Service latency
- Validate network path after deployment
- Use observability tools for traffic analysis

---

# Section 6 - Ingress Troubleshooting Scenarios

---

## Scenario 61

### Interview Question

Your Ingress resource is created successfully, but users receive **404 Not Found**. How would you troubleshoot it?

### Production-Level Answer

A 404 error from an Ingress usually indicates that the request reached the Ingress Controller, but no routing rule matched the request.

Focus on verifying the Ingress rules before investigating the application.

### Approach

Check Ingress.

```bash
kubectl get ingress
```

Describe it.

```bash
kubectl describe ingress <ingress-name>
```

Verify

- Host
- Path
- Backend Service
- Service Port
- DNS

Check Ingress Controller logs.

### Common Root Causes

- Wrong Host
- Wrong Path
- Service not found
- DNS pointing incorrectly
- Missing rewrite rule

### Best Practices

- Validate routing rules
- Test with curl
- Keep path rules simple
- Monitor Ingress logs

---

## Scenario 62

### Interview Question

Users receive **502 Bad Gateway** from the Ingress. How would you troubleshoot it?

### Production-Level Answer

A 502 error usually means the Ingress Controller cannot communicate with the backend Service or Pods.

The issue is generally between the Ingress and the application.

### Approach

Verify

- Service
- Endpoints
- Pod health
- TargetPort
- Container Port

Check

```bash
kubectl get endpoints
```

Review Ingress Controller logs.

### Common Root Causes

- Service unavailable
- No Endpoints
- Wrong TargetPort
- Application crashed
- Network issue

### Best Practices

- Validate backend Services
- Monitor endpoint availability
- Configure readiness probes
- Monitor Ingress metrics

---

## Scenario 63

### Interview Question

Users receive **503 Service Unavailable** after deployment. What would you investigate?

### Production-Level Answer

A 503 response indicates that the Ingress Controller has no healthy backend available to serve requests.

### Approach

Verify

- Pods
- Readiness Probes
- Endpoints
- Service
- Deployment

Run

```bash
kubectl get endpoints
```

Check

```bash
kubectl describe ingress <ingress-name>
```

### Common Root Causes

- No Ready Pods
- Readiness Probe failure
- Selector mismatch
- Deployment issue
- Application startup failure

### Best Practices

- Configure readiness probes
- Monitor endpoint health
- Validate rollouts
- Test Services before exposing them

---

## Scenario 64

### Interview Question

Your Ingress is created successfully, but no external IP or ALB is provisioned. How would you troubleshoot it?

### Production-Level Answer

If no external Load Balancer is created, investigate the Ingress Controller and cloud integration rather than the application.

### Approach

Check

```bash
kubectl describe ingress <ingress-name>
```

Verify

- Ingress Controller
- IngressClass
- Controller logs
- IAM permissions
- Cloud Controller

### Common Root Causes

- Ingress Controller not running
- Wrong IngressClass
- IAM permission issue
- Missing controller
- Cloud integration failure

### Best Practices

- Install one supported Ingress Controller
- Validate IAM permissions
- Monitor controller logs
- Verify IngressClass

---

## Scenario 65

### Interview Question

An AWS ALB is not created after deploying an Ingress on Amazon EKS. How would you investigate?

### Production-Level Answer

In EKS, the AWS Load Balancer Controller is responsible for creating the ALB.

If the ALB is not created, verify the controller before troubleshooting Kubernetes resources.

### Approach

Check

- AWS Load Balancer Controller
- Controller logs
- IAM Role
- IRSA
- Subnet tags
- Security Groups

Describe the Ingress.

```bash
kubectl describe ingress <ingress-name>
```

### Common Root Causes

- Controller not installed
- IRSA issue
- Missing subnet tags
- IAM permission denied
- Invalid annotations

### Best Practices

- Monitor AWS Load Balancer Controller
- Validate IRSA
- Tag subnets correctly
- Use official annotations

---

## Scenario 66

### Interview Question

HTTPS is not working even though the Ingress is configured with TLS. What would you investigate?

### Production-Level Answer

TLS failures generally occur because of certificate issues, incorrect Secrets, or Ingress configuration problems.

### Approach

Verify

- TLS Secret
- Certificate
- Host
- DNS
- Ingress TLS configuration

Describe Ingress.

```bash
kubectl describe ingress <ingress-name>
```

### Common Root Causes

- Missing Secret
- Expired certificate
- Wrong hostname
- Certificate mismatch
- DNS issue

### Best Practices

- Monitor certificate expiration
- Automate certificate renewal
- Validate TLS before production
- Secure Secrets

---

## Scenario 67

### Interview Question

Traffic reaches the ALB, but requests never reach the application Pods. How would you troubleshoot it?

### Production-Level Answer

If traffic reaches the Load Balancer but not the Pods, investigate the Service and Endpoints before the application.

### Approach

Verify

- ALB Target Groups
- Service
- Endpoints
- Pod readiness
- Health checks

Check

```bash
kubectl get endpoints
```

### Common Root Causes

- No Endpoints
- Health check failure
- Wrong Service selector
- TargetPort mismatch
- Readiness Probe failure

### Best Practices

- Monitor Target Groups
- Configure health checks
- Test Services independently
- Monitor endpoint health

---

## Scenario 68

### Interview Question

Your Ingress works for one hostname but fails for another. How would you investigate?

### Production-Level Answer

Multiple hostnames require matching DNS records and Ingress rules.

A mismatch causes routing failures.

### Approach

Verify

- DNS records
- Host rules
- TLS configuration
- Backend Service

Describe Ingress.

```bash
kubectl describe ingress <ingress-name>
```

### Common Root Causes

- Missing DNS record
- Wrong hostname
- Certificate mismatch
- Typographical error
- Missing rule

### Best Practices

- Standardize hostnames
- Validate DNS propagation
- Test every host individually

---

## Scenario 69

### Interview Question

The AWS ALB health checks continuously fail even though the Pods are running. What would you investigate?

### Production-Level Answer

Running Pods do not guarantee successful health checks.

The ALB health endpoint must return a successful response.

### Approach

Verify

- Health check path
- Readiness Probe
- Application endpoint
- Security Groups
- Target Groups

Review application logs.

### Common Root Causes

- Wrong health path
- Application not Ready
- Security Group restriction
- Wrong TargetPort
- HTTP 500 responses

### Best Practices

- Use dedicated health endpoints
- Keep health checks lightweight
- Monitor ALB Target Groups
- Validate health paths

---

## Scenario 70

### Interview Question

Everything appears healthy—Pods, Services, Ingress, and ALB—but users still cannot access the application. How would you troubleshoot it?

### Production-Level Answer

When every Kubernetes resource appears healthy, investigate the complete request path from the user's browser to the application.

Troubleshoot one layer at a time instead of making assumptions.

### Approach

Verify

- DNS
- ALB
- Security Groups
- Ingress
- Service
- Endpoints
- Pod
- Application logs

Test connectivity at every layer.

### Common Root Causes

- DNS propagation delay
- Security Group blocking traffic
- Firewall
- Wrong ALB listener
- Backend application issue
- External network problem

### Best Practices

- Troubleshoot layer by layer
- Validate external connectivity after deployment
- Monitor every networking component
- Maintain end-to-end observability

---

# Section 7 - Storage Troubleshooting Scenarios

---

## Scenario 71

### Interview Question

A Pod remains in the **Pending** state because its PersistentVolumeClaim (PVC) is also Pending. How would you troubleshoot it?

### Production-Level Answer

A Pod cannot start until its required PersistentVolumeClaim is successfully bound to a PersistentVolume.

Investigate the storage layer before troubleshooting the application.

### Approach

Check PVC status.

```bash
kubectl get pvc
```

Describe the PVC.

```bash
kubectl describe pvc <pvc-name>
```

Verify

- StorageClass
- Access Mode
- Requested Storage
- Available PersistentVolumes

### Common Root Causes

- No matching PersistentVolume
- StorageClass missing
- Wrong AccessMode
- Insufficient storage
- CSI Driver issue

### Best Practices

- Use dynamic provisioning
- Monitor PVC status
- Standardize StorageClasses
- Validate storage before deployment

---

## Scenario 72

### Interview Question

A PersistentVolumeClaim is created, but it never binds to a PersistentVolume. What would you investigate?

### Production-Level Answer

PVC binding depends on matching storage requirements.

If no suitable PersistentVolume exists, the PVC remains Pending.

### Approach

Check PVC.

```bash
kubectl describe pvc <pvc-name>
```

Check PVs.

```bash
kubectl get pv
```

Compare

- StorageClass
- Capacity
- AccessMode
- ReclaimPolicy

### Common Root Causes

- StorageClass mismatch
- Capacity mismatch
- AccessMode mismatch
- PV unavailable
- CSI failure

### Best Practices

- Use dynamic provisioning
- Maintain consistent StorageClasses
- Monitor storage capacity

---

## Scenario 73

### Interview Question

Your Pod reports **FailedMount** events. How would you troubleshoot it?

### Production-Level Answer

FailedMount indicates Kubernetes could not attach or mount the requested storage.

Investigate the volume rather than the application.

### Approach

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

Review Events.

Verify

- PVC
- PV
- StorageClass
- CSI Driver
- Node

### Common Root Causes

- PVC Pending
- CSI Driver unavailable
- Volume attachment failure
- Permission issue
- Storage unavailable

### Best Practices

- Monitor CSI Drivers
- Validate storage before deployment
- Alert on mount failures

---

## Scenario 74

### Interview Question

An Amazon EBS volume cannot attach to a Worker Node in Amazon EKS. How would you troubleshoot it?

### Production-Level Answer

EBS volumes are AZ-specific and require the EBS CSI Driver.

Verify both AWS and Kubernetes resources.

### Approach

Check

- PVC
- PV
- EBS CSI Driver
- Worker Node AZ
- Volume AZ
- IAM permissions

Describe PVC.

```bash
kubectl describe pvc <pvc-name>
```

### Common Root Causes

- EBS CSI Driver failure
- AZ mismatch
- IAM permission denied
- Volume unavailable
- Attachment timeout

### Best Practices

- Deploy EBS CSI Driver
- Schedule Pods within the correct AZ
- Monitor CSI Driver logs

---

## Scenario 75

### Interview Question

Your application reports **Permission Denied** while writing to a mounted volume. How would you investigate?

### Production-Level Answer

The volume is mounted successfully, but the application lacks permission to write.

Investigate file ownership and security context.

### Approach

Verify

- SecurityContext
- fsGroup
- runAsUser
- File permissions
- Mounted volume

Inspect inside the container.

### Common Root Causes

- Incorrect UID
- Missing fsGroup
- Read-only mount
- Volume ownership mismatch

### Best Practices

- Configure SecurityContext
- Avoid running containers as root
- Validate permissions during testing

---

## Scenario 76

### Interview Question

A PersistentVolume shows **Released** state after deleting the PVC. What does this mean?

### Production-Level Answer

Released means the PVC has been removed, but Kubernetes has not yet reclaimed or reused the PersistentVolume.

The next action depends on the Reclaim Policy.

### Approach

Check PV.

```bash
kubectl get pv
```

Describe it.

```bash
kubectl describe pv <pv-name>
```

Verify

- ReclaimPolicy
- StorageClass
- ClaimRef

### Common Root Causes

- Retain policy
- Manual cleanup required
- Storage administrator action pending

### Best Practices

- Understand Reclaim Policies
- Clean released volumes
- Monitor orphaned storage

---

## Scenario 77

### Interview Question

A StatefulSet Pod cannot start because its PersistentVolumeClaim is missing. How would you troubleshoot it?

### Production-Level Answer

Each StatefulSet Pod requires its own dedicated PersistentVolumeClaim.

Without storage, the Pod cannot start.

### Approach

Check StatefulSet.

```bash
kubectl get statefulset
```

Verify PVCs.

```bash
kubectl get pvc
```

Review Events.

### Common Root Causes

- Missing VolumeClaimTemplate
- StorageClass issue
- Dynamic provisioning failure
- CSI Driver problem

### Best Practices

- Validate StatefulSet templates
- Monitor PVC creation
- Use dynamic provisioning

---

## Scenario 78

### Interview Question

A Pod using Amazon EFS cannot access shared files across multiple Worker Nodes. How would you investigate?

### Production-Level Answer

Amazon EFS supports ReadWriteMany access.

Investigate mount configuration, networking, and the EFS CSI Driver.

### Approach

Verify

- EFS CSI Driver
- Mount Targets
- Security Groups
- NFS connectivity
- PVC

### Common Root Causes

- Security Group blocking NFS
- Missing Mount Target
- CSI Driver failure
- Incorrect filesystem ID

### Best Practices

- Deploy EFS CSI Driver
- Monitor NFS connectivity
- Validate mount targets

---

## Scenario 79

### Interview Question

Your application reports **Read-only file system** while writing to a PersistentVolume. How would you troubleshoot it?

### Production-Level Answer

This indicates the mounted filesystem is read-only or the volume has entered a protective state.

Investigate both Kubernetes configuration and the storage backend.

### Approach

Verify

- Mount options
- PersistentVolume
- PersistentVolumeClaim
- Storage backend
- Application configuration

Inspect mount information inside the container.

### Common Root Causes

- ReadOnly mount
- Storage backend issue
- Filesystem corruption
- Incorrect manifest

### Best Practices

- Verify mount options
- Monitor storage health
- Validate application write permissions

---

## Scenario 80

### Interview Question

Your PersistentVolumes, PersistentVolumeClaims, and Pods all appear healthy, but the application still cannot read or write data. How would you troubleshoot it?

### Production-Level Answer

Healthy Kubernetes resources do not guarantee storage functionality.

Investigate connectivity from the application to the underlying storage.

### Approach

Verify

- Volume mounts
- File permissions
- SecurityContext
- Storage backend
- CSI Driver
- Application logs

Test file operations inside the container.

### Common Root Causes

- Application permission issue
- Incorrect mount path
- Storage backend failure
- SecurityContext configuration
- Filesystem corruption

### Best Practices

- Test storage after deployment
- Monitor storage latency
- Validate permissions
- Perform regular storage health checks

---

# Section 8 - ConfigMaps & Secrets Troubleshooting Scenarios

---

## Scenario 81

### Interview Question

A Pod fails to start because the required **ConfigMap** is missing. How would you troubleshoot it?

### Production-Level Answer

When a Pod references a ConfigMap that does not exist, Kubernetes cannot inject the required configuration into the container.

The Pod will fail before the application starts.

### Approach

Check Pod Events.

```bash
kubectl describe pod <pod-name>
```

Verify ConfigMaps.

```bash
kubectl get configmap
```

Describe the ConfigMap.

```bash
kubectl describe configmap <configmap-name>
```

Verify

- ConfigMap name
- Namespace
- Volume mounts
- Environment variables

### Common Root Causes

- ConfigMap deleted
- Wrong namespace
- Typographical error
- Deployment before ConfigMap creation

### Best Practices

- Store ConfigMaps in Git
- Deploy ConfigMaps before applications
- Validate manifests in CI/CD
- Monitor missing resources

---

## Scenario 82

### Interview Question

Your Pod reports that a **Secret** cannot be found. How would you troubleshoot it?

### Production-Level Answer

Secrets are required before the container starts.

If the Secret does not exist or is in the wrong namespace, the Pod cannot initialize.

### Approach

Check Pod Events.

```bash
kubectl describe pod <pod-name>
```

Verify Secrets.

```bash
kubectl get secrets
```

Describe the Secret.

```bash
kubectl describe secret <secret-name>
```

Verify

- Secret name
- Namespace
- Secret type
- Deployment manifest

### Common Root Causes

- Secret deleted
- Wrong namespace
- Typographical error
- Incorrect Secret name

### Best Practices

- Store Secrets securely
- Avoid manual Secret creation
- Use GitOps
- Rotate Secrets regularly

---

## Scenario 83

### Interview Question

Your application starts successfully, but an environment variable loaded from a ConfigMap is empty. How would you investigate?

### Production-Level Answer

An empty environment variable usually indicates a mapping issue rather than an application issue.

Verify how the ConfigMap is referenced inside the Pod.

### Approach

Inspect the Pod.

```bash
kubectl describe pod <pod-name>
```

Review

- Environment Variables
- ConfigMap keys
- Pod manifest

Check ConfigMap.

```bash
kubectl get configmap <configmap-name> -o yaml
```

### Common Root Causes

- Wrong key
- Missing key
- Typographical error
- Incorrect environment mapping

### Best Practices

- Use meaningful ConfigMap keys
- Validate environment variables
- Keep ConfigMaps version-controlled

---

## Scenario 84

### Interview Question

A Secret is mounted successfully, but the application cannot read the files. How would you troubleshoot it?

### Production-Level Answer

The Secret exists and is mounted, but the application may lack permissions or be reading the wrong path.

### Approach

Verify

- Mount path
- File permissions
- SecurityContext
- Application configuration

Inspect the mounted directory inside the container.

### Common Root Causes

- Wrong mount path
- Incorrect file permissions
- Application reading wrong location
- SecurityContext issue

### Best Practices

- Use standard mount locations
- Validate file permissions
- Avoid hardcoded paths
- Test Secret mounts before deployment

---

## Scenario 85

### Interview Question

A Secret was updated, but the application continues using the old values. Why?

### Production-Level Answer

Updating a Secret does not automatically restart Pods.

Many applications only read Secret values during startup.

### Approach

Verify

- Secret update
- Pod creation time
- Mounted Secret
- Application behavior

Restart the Deployment if necessary.

```bash
kubectl rollout restart deployment <deployment-name>
```

### Common Root Causes

- Application caches configuration
- Pod not restarted
- Secret volume refresh delay
- Static environment variables

### Best Practices

- Restart workloads after Secret updates
- Automate Secret rotation
- Document Secret refresh behavior

---

## Scenario 86

### Interview Question

Your application reports authentication failures even though the Secret exists. How would you investigate?

### Production-Level Answer

The Secret may exist, but its contents could be incorrect.

Always verify the Secret values before blaming the application.

### Approach

Check

- Secret
- Deployment
- Environment variables
- External service credentials

Verify Secret contents carefully.

### Common Root Causes

- Incorrect password
- Expired credentials
- Wrong Secret referenced
- Incorrect encoding

### Best Practices

- Rotate credentials regularly
- Validate Secrets during deployment
- Use external Secret managers

---

## Scenario 87

### Interview Question

Multiple Pods fail after a ConfigMap update. How would you troubleshoot it?

### Production-Level Answer

A ConfigMap update can impact every workload using it.

Determine whether the update introduced invalid configuration.

### Approach

Review

- ConfigMap history
- Deployment rollout
- Application logs
- Pod Events

Compare old and new configuration.

### Common Root Causes

- Invalid configuration
- Missing keys
- Typographical error
- Application incompatibility

### Best Practices

- Version ConfigMaps
- Review configuration changes
- Test configuration in staging
- Implement change approvals

---

## Scenario 88

### Interview Question

Your team accidentally deleted a production Secret. What would you do?

### Production-Level Answer

A deleted Secret can immediately impact production workloads.

Restore the Secret as quickly as possible while minimizing application downtime.

### Approach

Identify

- Affected applications
- Backup
- GitOps repository
- Secret Manager

Restore the Secret.

Restart affected workloads if necessary.

### Common Root Causes

- Human error
- GitOps synchronization
- Incorrect cleanup script
- Unauthorized change

### Best Practices

- Use external Secret Managers
- Enable RBAC
- Audit Secret changes
- Maintain secure backups

---

## Scenario 89

### Interview Question

You need to rotate production database credentials without causing downtime. How would you approach it?

### Production-Level Answer

Credential rotation should be performed gradually to prevent application failures.

Coordinate Secret updates with application rollout.

### Approach

Update

- Database credentials
- Kubernetes Secret
- Application Deployment

Perform

```bash
kubectl rollout restart deployment <deployment-name>
```

Verify application connectivity.

### Common Root Causes

- Secret updated before database
- Application cache
- Old Pods still running
- Credential mismatch

### Best Practices

- Automate Secret rotation
- Use rolling deployments
- Validate connectivity after rotation
- Document credential management

---

## Scenario 90

### Interview Question

Everything appears healthy—ConfigMaps, Secrets, Pods, and Deployments—but the application still fails because of configuration issues. How would you troubleshoot it?

### Production-Level Answer

Healthy Kubernetes resources do not guarantee that the application is using the expected configuration.

Trace the configuration from Kubernetes to the application.

### Approach

Verify

- ConfigMap
- Secret
- Deployment
- Environment variables
- Mounted files
- Application logs

Compare the running configuration with the expected configuration.

### Common Root Causes

- Wrong environment variable
- Incorrect ConfigMap key
- Secret mismatch
- Application cache
- Configuration not reloaded

### Best Practices

- Validate configuration after deployment
- Monitor configuration changes
- Use GitOps for configuration management
- Automate configuration testing

---