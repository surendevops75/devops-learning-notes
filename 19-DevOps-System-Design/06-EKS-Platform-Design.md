# 1. Purpose

This file is a senior-level production design guide for Amazon EKS platforms.
The focus is architecture, operational trade-offs, failure domains, security,
networking, scaling, identity, upgrades, observability, GitOps and disaster recovery.

A production EKS platform should provide:
```text
secure identity
private networking where appropriate
multi-AZ resilience
elastic compute
controlled ingress and egress
persistent storage
GitOps delivery
observability
policy
backup and recovery
cost governance
safe upgrades
```
The design principle is:
```text
AWS infrastructure -> EKS foundation -> platform addons -> workloads -> business service
```

# 2. Requirements First

Before choosing an EKS topology establish:
```text
number of clusters
number of accounts
regions
teams
applications
pod count
peak traffic
stateful workloads
availability target
RTO/RPO
compliance
network connectivity
security boundaries
upgrade windows
```
Do not start with a tool such as Karpenter or a CNI setting. Start with
requirements and derive the architecture.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 3. Reference Enterprise Architecture

```text
AWS Organization
 |
+-- Network / Shared Services
+-- Security / Audit
+-- Development Accounts
+-- Production Accounts
       |
       +-- VPC
            |
            +-- Public Edge Subnets
            +-- Private Worker Subnets
            +-- Private EKS API
            |
            +-- EKS
                 |
                 +-- Managed Control Plane
                 +-- System Nodes
                 +-- General Nodes
                 +-- Specialized / Spot Capacity
                 |
                 +-- CoreDNS
                 +-- VPC CNI
                 +-- EBS/EFS CSI
                 +-- AWS Load Balancer Controller
                 +-- EKS Pod Identity / IAM integration
                 +-- Metrics / Logs / Traces
                 +-- Argo CD
                 |
                 +-- Applications
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 4. EKS Responsibility Boundary

AWS manages the EKS managed control-plane service components.
The platform team remains responsible for the workload-facing platform:
```text
VPC and connectivity
node capacity
IAM configuration
Kubernetes RBAC
addons
ingress
storage integration
observability
security policies
GitOps
application platform
cost
upgrade orchestration
```
Always verify current AWS responsibility boundaries for the exact EKS
feature being used.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 5. EKS Cluster Strategy

Use one cluster when:
```text
teams can safely share
failure isolation is acceptable
resource scale is manageable
```
Use multiple clusters for:
```text
strong isolation
regional placement
compliance
independent upgrades
blast-radius reduction
large fleet scale
```
Use separate AWS accounts when the organization needs a stronger
security, billing or administrative boundary.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 6. AWS Account Strategy

A common model:
```text
Organization
 |
+-- Security
+-- Log Archive
+-- Network
+-- Shared Services
+-- Dev
+-- Stage
+-- Prod
```
Production EKS clusters should normally have account-level isolation
appropriate to business risk. Do not assume account separation alone
solves workload security.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 7. VPC Design

Typical production EKS:
```text
VPC
 |
+-- Public subnets
|    |
|    +-- internet-facing load balancers where required
|
+-- Private subnets
     |
     +-- worker nodes
     +-- internal load balancers
     +-- private endpoints
```
Use multiple Availability Zones. Plan CIDR ranges before cluster creation
because pod and node IP growth can become a hard scaling limit.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 8. Private EKS

A private EKS architecture may use:
```text
private EKS API endpoint
private worker subnets
VPC endpoints
controlled NAT egress
internal load balancers
```
Validate access paths for:
```text
developers
CI
Argo CD
AWS services
container registry
secrets
monitoring
```
A private cluster is not automatically more secure if egress and identity
are poorly controlled.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 9. EKS API Endpoint

Common endpoint modes include public, private,
or configurations that support controlled access to both.
Design around:
```text
who can reach API
from where
how authentication works
how authorization works
how emergency access works
```
For a private API, provide an intentional administrative network path
rather than opening broad public access during incidents.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 10. Subnet and IP Planning

Calculate:
```text
node IP demand
pod IP demand
load balancer addresses
future cluster growth
AZ distribution
```
An EKS cluster can have CPU capacity while still failing to schedule
new pods because the VPC has insufficient usable IP addresses.
Monitor subnet utilization and pod IP allocation.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 11. AWS VPC CNI

The AWS VPC CNI integrates Kubernetes pod networking
with AWS VPC networking. Design decisions include:
```text
pod IP allocation
ENI limits
prefix delegation where supported
custom networking where required
security groups for pods where required
```
Capacity planning must account for instance ENI/IP limits and subnet
availability, not only Kubernetes CPU and memory.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 12. Prefix Delegation

Prefix delegation can improve pod IP allocation
efficiency on supported configurations by allocating prefixes rather than
individual secondary IPs.
Evaluate:
```text
instance support
CNI version
subnet capacity
pod density
startup behavior
```
Do not enable a networking optimization without validating the complete
node and CNI compatibility matrix.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 13. Security Groups for Pods

For workloads needing AWS security-group
semantics, use the supported EKS capability to associate security groups
with pods.
This can provide:
```text
fine-grained AWS network controls
database access segmentation
legacy security-group integration
```
Use it selectively because it adds operational complexity and must be
understood alongside Kubernetes NetworkPolicy.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 14. NetworkPolicy

Use Kubernetes NetworkPolicy for pod-level
communication controls:
```text
default deny
 |
explicit application flows
```
Example:
```text
frontend -> api
api -> database
worker -> queue
```
Cloud security groups and NetworkPolicy operate at different layers and
should be designed together.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 15. DNS Architecture

EKS service discovery commonly depends on CoreDNS:
```text
application
 |
service DNS
 |
CoreDNS
 |
service endpoint
```
Monitor:
```text
DNS latency
errors
CPU
memory
restarts
```
DNS failures can create broad application outages because service discovery
is a foundational dependency.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 16. Load Balancer Architecture

Typical internet flow:
```text
Client
 |
Route 53
 |
WAF / CloudFront where required
 |
ALB
 |
Ingress
 |
Service
 |
Pods
```
Internal applications can use internal load balancers:
```text
VPC client -> internal LB -> service -> pods
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 17. AWS Load Balancer Controller

The AWS Load Balancer Controller
integrates Kubernetes resources with AWS load-balancing services.
Typical responsibilities:
```text
Ingress -> ALB
Service type LoadBalancer -> supported AWS load balancer behavior
```
Protect its IAM permissions and monitor reconciliation failures.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 18. ALB vs NLB

ALB is commonly chosen for:
```text
HTTP/HTTPS
host routing
path routing
application-layer features
```
NLB is commonly chosen for:
```text
TCP/UDP
high-performance network traffic
static IP requirements
certain TLS/networking use cases
```
Choose from traffic requirements rather than familiarity.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 19. IAM Architecture

Separate identities:
```text
human admin
CI
Argo
node
application workload
```
Avoid sharing credentials.
The target model is:
```text
workload -> Kubernetes ServiceAccount -> AWS identity -> least-privilege IAM role
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 20. EKS Pod Identity

EKS Pod Identity provides a supported mechanism for
associating Kubernetes service accounts with AWS IAM roles.
Design:
```text
Pod
 |
ServiceAccount
 |
Pod Identity association
 |
IAM role
 |
AWS API
```
Use it to avoid embedding long-lived AWS keys in containers.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 21. IRSA vs Pod Identity

Both solve workload-to-IAM association.
Compare based on:
```text
AWS account strategy
cluster management model
existing integrations
operational simplicity
organizational standards
```
Do not mix mechanisms randomly. Standardize one approach where practical.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 22. Node IAM

Node roles should contain only permissions required by
node-level agents and AWS integrations.
Do not give nodes broad application permissions.
Workload permissions should come from workload identity.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 23. Kubernetes RBAC

Separate:
```text
cluster administrators
platform operators
namespace administrators
developers
read-only users
service accounts
```
Avoid:
```text
developer -> cluster-admin
```
Use namespace-scoped permissions for normal application operations.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 24. Namespace Architecture

A practical model:
```text
team-a
team-b
team-c
platform-system
observability
security
ingress
```
Namespaces provide logical organization and policy scope, but stronger
isolation may require dedicated node pools, clusters or AWS accounts.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 25. Multi-Tenancy

Layer tenant controls:
```text
namespace
+
RBAC
+
NetworkPolicy
+
ResourceQuota
+
LimitRange
+
Pod security
+
workload identity
+
admission policy
```
For high-risk tenants add:
```text
dedicated nodes
dedicated cluster
dedicated account
```
based on requirements.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 26. Node Groups

Common pools:
```text
system
general
compute
memory
critical
spot
GPU
```
Keep system components away from unreliable capacity when required.
Use labels, taints and tolerations deliberately.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 27. Managed Node Groups

Managed Node Groups simplify:
```text
node lifecycle
AMI integration
scaling configuration
rolling replacement
```
They remain an important default for teams that prefer operational
simplicity over maximum scheduling flexibility.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 28. Karpenter

Karpenter can provision nodes dynamically based on
pending pod requirements.
Conceptually:
```text
Pending Pods
 |
Karpenter
 |
choose compatible instance
 |
launch node
 |
schedule pods
```
Use it when workload diversity and rapid capacity provisioning justify
the added platform complexity.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 29. Managed Node Groups vs Karpenter

Managed Node Groups:
```text
predictable
simple
stable pools
```
Karpenter:
```text
dynamic
instance-aware
fast capacity matching
```
A mature platform may use both:
```text
managed nodes -> baseline/system capacity
Karpenter -> elastic application capacity
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 30. Spot Instances

Good candidates:
```text
batch
CI
stateless workers
fault-tolerant processing
```
Requirements:
```text
multiple instance types
interruption handling
replicas
graceful termination
capacity fallback
```
Never treat spot as guaranteed capacity.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 31. Node Provisioning

Node lifecycle:
```text
select capacity
 |
launch
 |
bootstrap
 |
join cluster
 |
daemonsets
 |
ready
 |
schedule workload
```
Monitor failures in:
```text
IAM
network
user data
AMI
CNI
bootstrap
security groups
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 32. Bottlerocket and Node OS

Container-optimized operating systems can
reduce host maintenance and attack surface.
Choose node OS based on:
```text
application compatibility
debugging model
security
patching
team expertise
```
Do not standardize on an OS without validating operational tooling.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 33. Pod Density

Pod density depends on:
```text
instance limits
CNI IP capacity
CPU
memory
daemonsets
networking
```
The theoretical maximum pod count is not a safe production target.
Maintain headroom for system workloads and replacements.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 34. Resource Requests

Requests affect scheduling:
```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```
Under-requesting causes contention.
Over-requesting causes wasted capacity.
Use measured workload behavior.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 35. Resource Limits

Limits can protect nodes but poorly chosen limits
can cause:
```text
CPU throttling
OOMKilled
latency
restart loops
```
Tune them based on workload characteristics rather than copying one
global number to every service.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 36. ResourceQuota

Namespace quotas can limit:
```text
CPU
memory
pods
services
PVCs
```
This protects shared clusters from uncontrolled tenant growth.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 37. LimitRange

LimitRange can establish namespace defaults and
constraints for container resources.
It is useful as a guardrail but should not replace workload-specific
capacity planning.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 38. Scheduling

Scheduler considers:
```text
resource requests
node labels
taints
tolerations
affinity
anti-affinity
topology constraints
priority
```
Use explicit scheduling requirements for critical workloads.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 39. Topology Spread

For three replicas:
```text
replica 1 -> AZ-A
replica 2 -> AZ-B
replica 3 -> AZ-C
```
Use topology spread constraints to express desired distribution.
Do not rely on chance placement.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 40. Pod Disruption Budget

PDB protects against voluntary disruptions:
```text
3 replicas
minAvailable: 2
```
It does not protect against:
```text
hard node failure
application crash
AZ loss
```
Therefore combine PDB with replication and topology design.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 41. Probes

Readiness:
```text
should this pod receive traffic?
```
Liveness:
```text
should this container be restarted?
```
Startup:
```text
has this slow-starting application finished initialization?
```
Bad probes can create outages through restart storms.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 42. Graceful Shutdown

Production shutdown:
```text
termination signal
 |
stop new work
 |
connection drain
 |
finish in-flight requests
 |
exit
```
Coordinate:
```text
terminationGracePeriodSeconds
preStop behavior
load-balancer draining
application signal handling
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 43. HPA

Horizontal Pod Autoscaler changes replica count based on
metrics:
```text
load -> metric -> HPA -> replicas
```
Validate that downstream systems can handle the resulting concurrency.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 44. Cluster Autoscaling

A typical scaling chain:
```text
traffic
 |
HPA
 |
pending pods
 |
node autoscaling
 |
new nodes
 |
pods scheduled
```
Node startup latency must be included in capacity planning.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 45. Storage

EKS storage commonly uses CSI integrations:
```text
Pod
 |
PVC
 |
CSI
 |
EBS / EFS / other supported storage
```
Choose storage based on:
```text
access mode
latency
durability
backup
AZ behavior
throughput
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 46. EBS

EBS-backed volumes are commonly used for block storage.
Plan:
```text
AZ placement
volume type
IOPS
throughput
snapshot
restore
```
A volume's lifecycle and application data lifecycle must be understood
before automated deletion is enabled.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 47. EFS

EFS can provide shared file access where the workload requires
it.
Evaluate:
```text
performance
throughput
mount targets
cost
application locking
```
Do not use shared filesystem storage merely because it avoids
application redesign.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 48. Backup

Back up:
```text
persistent data
database state
cluster-specific non-Git state
secret recovery material
critical external configuration
```
Git-managed YAML is not a backup of application data.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 49. EKS Backup Strategy

A recovery design may include:
```text
infrastructure as code
GitOps
container registry retention
database backups
volume snapshots
object-storage backups
secret recovery
DNS recovery
```
Test restoration rather than only verifying that backups exist.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 50. Observability

Monitor:
```text
control-plane-facing signals available through AWS
node health
pod health
API metrics where exposed
DNS
CNI
load balancers
storage
applications
deployments
```
Use:
```text
metrics
logs
traces
events
audit logs
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 51. Logging

Centralize:
```text
application logs
container logs
node logs
ingress/load-balancer logs
security/audit logs
```
Design:
```text
collection -> transport -> storage -> retention -> search
```
Control retention and cost.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 52. Metrics

Core platform metrics:
```text
node CPU
node memory
pod CPU
pod memory
restarts
pending pods
scheduling failures
network errors
DNS failures
storage latency
deployment failures
```
Use SLOs for user-impacting services.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 53. Tracing

Distributed tracing:
```text
request
 |
frontend
 |
API
 |
worker
 |
database
```
helps identify latency propagation across microservices.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 54. EKS Addons

Common platform addons include:
```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI
EFS CSI where required
AWS Load Balancer Controller
metrics
secrets integration
GitOps
policy
observability
```
Maintain a version compatibility matrix.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 55. Addon Lifecycle

For each addon track:
```text
owner
version
compatibility
security advisories
upgrade procedure
rollback
monitoring
```
Treat addons as production dependencies, not background packages.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 56. GitOps Integration

A mature model:
```text
Application source
 |
CI
 |
immutable artifact
 |
GitOps repository
 |
Argo CD
 |
EKS
```
Separate:
```text
artifact creation
from
deployment intent
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 57. Argo CD in EKS

Argo CD may run inside a management cluster or
appropriate control-plane environment and reconcile target EKS clusters.
At fleet scale use:
```text
Applications
ApplicationSets
Projects
cluster labels
deployment waves
```
Control cluster credentials with least privilege.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 58. EKS Fleet Architecture

Example:
```text
Management
 |
Argo CD
 |
+-- dev-cluster
+-- stage-cluster
+-- prod-a
+-- prod-b
+-- dr-cluster
```
Avoid unrestricted global synchronization. Use environments and waves
to contain bad changes.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 59. Multi-Cluster

Reasons:
```text
failure isolation
regional deployment
compliance
upgrade independence
team isolation
scale
```
Costs:
```text
more addons
more operations
more observability
more networking
```
The correct number of clusters is a business and operational decision.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 60. Multi-Region

Example:
```text
Global traffic
 |
+-- Region A -> EKS A
|
+-- Region B -> EKS B
```
Multi-region requires:
```text
data strategy
DNS/traffic strategy
secrets
artifact availability
observability
operational runbooks
```
A second cluster alone is not DR.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 61. Deployment Strategy

Use:
```text
rolling
blue-green
canary
progressive delivery
```
For critical releases:
```text
5%
 |
health
 |
20%
 |
health
 |
50%
 |
health
 |
100%
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 62. Database Compatibility

Use expand/migrate/contract:
```text
expand schema
 |
deploy compatible application
 |
migrate data
 |
validate
 |
contract old schema
```
Do not assume Kubernetes rollback reverses database changes.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 63. EKS Security Architecture

Layer controls:
```text
AWS IAM
 |
EKS authentication
 |
Kubernetes RBAC
 |
namespace
 |
NetworkPolicy
 |
Pod security
 |
admission
 |
image security
 |
runtime monitoring
```
No single layer is sufficient.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 64. Image Security

Production image policy can require:
```text
approved registry
immutable digest
vulnerability scanning
SBOM
signature
provenance
```
Admission controls can enforce selected requirements.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 65. Secrets

Preferred flow:
```text
AWS Secrets Manager / approved secret store
 |
secret integration
 |
Kubernetes Secret
 |
Pod
```
Avoid:
```text
plaintext Git
Dockerfile secrets
pipeline logs
```
Rotate credentials and monitor rotation failures.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 66. Certificate Management

Automate:
```text
request
issue
store
attach
renew
validate
```
Certificate expiration should be an alertable condition.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 67. Egress

Unrestricted pod egress can enable:
```text
data exfiltration
malware callbacks
unapproved downloads
```
Controls may include:
```text
NetworkPolicy
NAT
firewall
proxy
VPC endpoints
DNS controls
```
Balance security with application dependency requirements.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 68. VPC Endpoints

For private clusters, endpoints can reduce dependency
on public internet paths for AWS services.
Evaluate endpoints for:
```text
ECR
S3
STS
CloudWatch
Secrets Manager
other required AWS APIs
```
Exact endpoint requirements depend on the platform architecture.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 69. NAT Gateway

Private workloads often require controlled egress.
Consider:
```text
NAT per AZ
centralized NAT
egress proxy
VPC endpoints
```
Trade-offs include:
```text
resilience
cost
routing complexity
inspection
```
Avoid a single NAT dependency for critical multi-AZ production unless
the risk is explicitly accepted.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 70. Private Connectivity

Enterprise EKS may need:
```text
Transit Gateway
VPC peering
Direct Connect
VPN
PrivateLink
```
Design routes and security boundaries deliberately.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 71. CI to EKS

Prefer:
```text
CI -> artifact
CI -> GitOps change
Argo -> EKS
```
rather than giving every CI job direct unrestricted production cluster
credentials.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 72. Developer Access

Provide controlled paths:
```text
SSO
short-lived identity
RBAC
audit
break-glass
```
Developers should not require permanent administrator credentials.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 73. Break-Glass

Emergency process:
```text
incident authorization
 |
temporary privileged access
 |
mitigation
 |
audit
 |
restore Git desired state
 |
remove emergency access
```
Break-glass should be tested before an incident.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 74. Production Upgrade

A safe sequence:
```text
compatibility review
 |
test cluster
 |
non-production
 |
production canary
 |
observe
 |
remaining clusters
```
Check:
```text
Kubernetes version
CNI
CoreDNS
CSI
Ingress
Argo
operators
admission policies
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 75. Node Upgrade

Controlled node replacement:
```text
new capacity
 |
cordon/drain old nodes
 |
PDB respected
 |
workloads rescheduled
 |
old nodes removed
```
Maintain enough spare capacity for the migration.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 76. Cluster Blue-Green Migration

For high-risk changes:
```text
old EKS cluster
 |
new EKS cluster
 |
bootstrap platform
 |
deploy workloads
 |
validate
 |
shift traffic
 |
retire old
```
This can reduce in-place upgrade risk at the cost of temporary duplicate
capacity and migration complexity.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 77. Failure Domain

Design across:
```text
container
pod
node
AZ
cluster
region
account
```
Critical services should survive the failure domains specified by their
SLO and business continuity requirements.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 78. Node Failure

Expected recovery:
```text
node fails
 |
pods become unavailable
 |
controller creates replacements
 |
scheduler places pods
 |
autoscaler adds capacity if needed
```
Validate that topology and capacity allow recovery.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 79. AZ Failure

If an AZ is lost:
```text
remaining AZs
 |
load balancer
 |
replicas
 |
capacity
```
must be sufficient.
Multi-AZ labels alone do not guarantee resilience.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 80. Cluster Failure

For services requiring cluster-level recovery:
```text
traffic manager
 |
secondary EKS cluster
 |
GitOps reconciliation
 |
data recovery
```
Data replication and recovery must be independently tested.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 81. CNI Failure

Symptoms:
```text
pods cannot obtain networking
new pods fail
network errors
```
Investigate:
```text
CNI daemonsets
IAM
ENI/IP availability
subnets
security groups
instance limits
```
Do not immediately delete healthy nodes without understanding the cause.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 82. DNS Failure

Symptoms:
```text
service-to-service calls fail
external hostname resolution fails
timeouts
```
Check:
```text
CoreDNS pods
node connectivity
VPC DNS
upstream resolution
NetworkPolicy
```
Protect CoreDNS with appropriate replicas and scheduling.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 83. Storage Failure

Symptoms:
```text
PVC pending
mount failure
IO errors
pod startup failure
```
Check:
```text
CSI controller
node plugin
IAM
AZ placement
volume state
security groups
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 84. Ingress Failure

Check:
```text
DNS
WAF
load balancer
controller
Ingress object
target health
service
pod readiness
security groups
```
Trace the request path from client to pod rather than changing random
Kubernetes objects.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 85. Autoscaling Failure

If HPA scales but nodes do not:
```text
pending pods
 |
scheduler reason
 |
node provisioning
 |
IAM/network
 |
instance capacity
```
If nodes scale but pods remain pending, inspect:
```text
taints
labels
resource requests
IP capacity
volume constraints
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 86. Cost Architecture

Major EKS costs:
```text
worker compute
NAT
load balancers
storage
data transfer
observability
idle capacity
```
Optimize:
```text
right-sizing
autoscaling
spot where safe
VPC endpoints
log retention
node consolidation
```
Never remove required HA solely to save cost.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 87. Capacity Planning

Track:
```text
CPU growth
memory growth
pod growth
IP growth
storage growth
node growth
API load
```
Plan for:
```text
normal
peak
node failure
AZ failure
deployment surge
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 88. SLOs

Define platform SLOs such as:
```text
cluster availability
application availability
deployment success
deployment latency
DNS availability
ingress availability
```
Do not confuse platform SLOs with application SLOs.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 89. Platform Ownership

Example:
```text
Cloud/IaC -> Platform
EKS -> Platform
CNI -> Platform
Ingress -> Platform
Argo -> Platform
Application -> App Team
Business SLO -> App Team
Shared Observability -> SRE/Platform
```
Document exceptions.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 90. Onboarding

New team automation:
```text
team request
 |
namespace
 |
Argo Project
 |
RBAC
 |
quota
 |
NetworkPolicy
 |
service template
 |
CI
 |
observability
```
Self-service reduces manual configuration drift.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 91. Golden Path

Developer experience:
```text
service template
 |
Git repository
 |
CI
 |
artifact
 |
GitOps
 |
Argo
 |
EKS
 |
dashboard
 |
alerts
```
The platform should provide safe defaults without preventing justified
advanced use cases.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 92. Policy as Code

Policies can require:
```text
approved images
non-root containers
resource requests
owner labels
no privileged mode
approved namespaces
```
Validate at:
```text
PR
CI
admission
runtime monitoring
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 93. Auditability

A production deployment should answer:
```text
who changed code
who reviewed it
what artifact was built
what digest was deployed
what Git revision was applied
which cluster changed
when it changed
what health signals followed
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 94. Disaster Recovery

A rebuild path:
```text
AWS infrastructure as code
 |
VPC
 |
EKS
 |
addons
 |
Argo
 |
GitOps
 |
applications
 |
data restore
```
Recovery time depends heavily on data and external dependencies.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 95. DR Test

Test:
```text
cluster loss
region loss where required
secret recovery
registry access
GitOps bootstrap
database restore
DNS/traffic failover
observability recovery
```
Measure actual RTO and RPO.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 96. Production Reference

```text
Developer
 |
Git
 |
CI
 |  |  security
 |
Immutable Image
 |
Registry
 |
GitOps PR
 |
Argo CD
 |
Private EKS API
 |
+-----------------------------+
| Multi-AZ EKS                |
|                             |
| system nodes                |
| general nodes               |
| Karpenter/elastic capacity  |
|                             |
| ingress -> services -> pods |
|                             |
| CNI / CoreDNS / CSI         |
|                             |
| observability               |
+-----------------------------+
 |
AWS services via private paths
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 97. Senior Design Framework

When asked to design EKS:
```text
1. Clarify workload and traffic.
2. Clarify availability.
3. Clarify scale.
4. Clarify account and region boundaries.
5. Design VPC.
6. Design EKS endpoint.
7. Design node capacity.
8. Choose MNG/Karpenter/spot.
9. Design pod networking.
10. Design ingress/egress.
11. Design identity.
12. Design tenancy.
13. Design storage.
14. Design GitOps.
15. Design observability.
16. Design upgrades.
17. Design DR.
18. Design cost.
19. Explain failure modes.
20. Explain trade-offs.
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 98. Scenario: 1,000 Services

Use:
```text
multiple clusters
standard node pools
Karpenter where justified
ApplicationSets
GitOps
resource quotas
central observability
standard addons
deployment waves
```
Do not put every workload into one unrestricted cluster without capacity,
security and blast-radius analysis.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 99. Scenario: 100,000 Pods

Primary constraints may become:
```text
API scale
scheduler load
CNI/IP capacity
node density
observability ingestion
controller reconciliation
```
At this scale, cluster partitioning and workload distribution become
architectural decisions rather than simple configuration changes.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 100. Scenario: Private Production EKS

Design:
```text
private API
private worker nodes
VPC endpoints
controlled NAT/egress
central CI/GitOps access
SSO
workload identity
internal load balancers
```
Then validate every dependency path before closing public access.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 101. Scenario: Bad Platform Upgrade

Contain:
```text
stop rollout
 |
freeze additional upgrades
 |
identify affected component
 |
rollback or replace capacity
 |
restore application health
 |
validate compatibility
 |
resume canary
```
Platform changes should never be rolled across the fleet blindly.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 102. Scenario: Pod IP Exhaustion

Symptoms:
```text
new pods fail networking
pending pods
CNI errors
```
Response:
```text
confirm subnet utilization
 |
confirm ENI/IP limits
 |
check CNI allocation
 |
add capacity / address space according to architecture
 |
prevent recurrence with forecasting
```
CPU dashboards alone will not reveal this failure.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 103. Scenario: Karpenter Cannot Launch Nodes

Check:
```text
NodePool/NodeClass configuration
IAM
subnet discovery
security groups
instance availability
taints
requirements
quotas
EC2 capacity
```
Then inspect why the pending pod cannot be satisfied.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 104. Scenario: Entire AZ Lost

Validate:
```text
replica distribution
remaining node capacity
load balancer behavior
storage recovery
database resilience
DNS
```
If the workload cannot survive the loss, document the accepted risk or
redesign the failure-domain architecture.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 105. Scenario: Security Incident

For compromised workload:
```text
isolate
 |
deny network
 |
revoke identity
 |
preserve evidence
 |
replace image
 |
redeploy
 |
audit
```
Do not assume deleting one pod removes persistence or compromise.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 106. Anti-Patterns

Avoid:
```text
single-AZ production
single node for critical service
cluster-admin for developers
long-lived AWS keys in pods
latest image
unbounded egress
unplanned CIDR
no PDB
bad liveness probes
one giant shared namespace
Terraform and GitOps fighting
manual cluster changes
no upgrade matrix
no DR test
no capacity headroom
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 107. Production Checklist

```text
[ ] multi-AZ
[ ] subnet capacity
[ ] pod IP capacity
[ ] private worker nodes where appropriate
[ ] endpoint access designed
[ ] IAM least privilege
[ ] workload identity
[ ] Kubernetes RBAC
[ ] NetworkPolicy
[ ] quotas
[ ] probes
[ ] PDB
[ ] topology spread
[ ] autoscaling
[ ] ingress
[ ] DNS
[ ] storage
[ ] secrets
[ ] observability
[ ] GitOps
[ ] backup
[ ] restore test
[ ] upgrade test
[ ] cost controls
[ ] runbooks
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 108. Golden Rules 1–50

```text
1. Start with requirements.
2. Treat EKS as a platform, not merely a cluster.
3. Design failure domains explicitly.
4. Use multiple AZs for production.
5. Maintain capacity headroom.
6. Plan IP space before deployment.
7. Monitor pod IP consumption.
8. Keep workers private where appropriate.
9. Design the EKS API access path.
10. Never solve an incident by opening unrestricted API access.
11. Separate human and workload identity.
12. Use least-privilege IAM.
13. Prefer workload identity over static keys.
14. Separate CI identity from runtime identity.
15. Separate Argo identity from developer identity.
16. Avoid cluster-admin defaults.
17. Use namespace boundaries.
18. Use NetworkPolicy.
19. Use resource quotas.
20. Use realistic resource requests.
21. Tune limits intentionally.
22. Use topology spread for critical replicas.
23. Use PDB for voluntary disruption.
24. Use readiness correctly.
25. Use liveness conservatively.
26. Use startup probes for slow applications.
27. Implement graceful shutdown.
28. Use autoscaling based on real demand.
29. Protect downstream dependencies from scale-out.
30. Use spot only for interruption-tolerant workloads.
31. Choose managed nodes or Karpenter from requirements.
32. Do not use Karpenter merely because it is popular.
33. Standardize node pools.
34. Isolate critical system capacity.
35. Monitor node pressure.
36. Monitor pending pods.
37. Monitor scheduling failures.
38. Monitor DNS.
39. Monitor CNI.
40. Monitor ingress.
41. Monitor storage.
42. Monitor certificates.
43. Centralize logs.
44. Collect metrics.
45. Use traces where valuable.
46. Correlate deployments with telemetry.
47. Keep addons versioned.
48. Maintain compatibility matrices.
49. Test upgrades before production.
50. Never assume managed control plane means zero operational responsibility.

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 109. Golden Rules 51–100

```text
51. Treat cluster addons as production dependencies.
52. Protect addon IAM.
53. Keep Argo and GitOps protected.
54. Separate infrastructure from application ownership.
55. Avoid Terraform/Argo ownership conflicts.
56. Use immutable artifacts.
57. Prefer image digests in production.
58. Scan images.
59. Track provenance.
60. Use SBOM where required.
61. Verify signatures where required.
62. Keep secrets out of plaintext Git.
63. Automate secret rotation.
64. Monitor secret integration.
65. Automate certificates.
66. Monitor renewal.
67. Control egress.
68. Use VPC endpoints where beneficial.
69. Avoid unnecessary NAT cost.
70. Do not introduce a single-AZ egress dependency.
71. Use internal load balancers for private services.
72. Choose ALB/NLB by traffic requirements.
73. Protect load-balancer controller permissions.
74. Use deployment waves.
75. Use canary for risky changes.
76. Test rollback.
77. Design database migrations for compatibility.
78. Do not equate Git rollback with data rollback.
79. Back up persistent data.
80. Test restore.
81. Define RTO.
82. Define RPO.
83. Rehearse DR.
84. Make cluster rebuild repeatable.
85. Keep infrastructure as code.
86. Keep platform configuration as code.
87. Keep deployment desired state in Git where appropriate.
88. Use break-glass access.
89. Audit break-glass access.
90. Remove emergency permissions after use.
91. Reduce blast radius.
92. Separate critical workloads.
93. Use dedicated clusters when isolation requires it.
94. Use dedicated accounts when stronger boundaries require it.
95. Forecast capacity.
96. Forecast IP usage.
97. Forecast storage.
98. Forecast observability cost.
99. Measure platform SLOs.
100. Test the architecture's failure assumptions.
```

### Production Review Questions

```text
- What is the failure domain?
- What is the owner?
- What is the dependency?
- What is the scaling limit?
- What is the security boundary?
- What happens during failure?
- How is the change rolled back?
- How is the component restored?
- What metric proves health?
- What test validates the design?
```

# 110. Final Architecture Principle

A production EKS platform should make
application delivery predictable while making infrastructure failure
recoverable.

The complete chain is:

```text
AWS Organization
      |
Account / Security Boundary
      |
VPC / Routing / Egress
      |
EKS Control Plane
      |
Node Capacity
      |
CNI / DNS / Storage / Ingress
      |
IAM / RBAC / Policy
      |
GitOps
      |
Applications
      |
Metrics / Logs / Traces
      |
SLO / Incident / DR
```

The senior engineer must be able to explain not only how the happy path
works, but also:

```text
What happens when a node fails?
What happens when an AZ fails?
What happens when pod IPs run out?
What happens when CoreDNS fails?
What happens when CNI fails?
What happens when EKS API access is lost?
What happens when CI fails?
What happens when Argo fails?
What happens when the registry fails?
What happens when an image is compromised?
What happens when a secret is exposed?
What happens when a cluster is lost?
What happens when an entire region is lost?
How do we recover?
How do we prove recovery works?
```

That is the standard for a production EKS platform design.
