# Multi-Cluster Deployment

> Deep production guide for deploying and operating applications across
> multiple AWS EKS clusters using Argo CD, ApplicationSets, GitOps,
> multi-region architecture, progressive fleet rollout, disaster
> recovery, security boundaries, and production troubleshooting.

## Chapter Objective

This chapter extends the multi-environment model into a real EKS fleet.
The objective is to make cluster placement deterministic, secure,
observable, recoverable, and safe to operate at production scale.

## 1. Multi-Cluster Strategy

A production multi-cluster platform must make cluster identity,
ownership, purpose, region, AWS account, Kubernetes version, and
workload placement explicit. Argo CD can manage several EKS clusters
from a central control plane, or separate Argo CD instances can provide
stronger isolation. The choice is driven by blast radius, compliance,
network connectivity, operational scale, and recovery requirements.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 2. Why Multiple EKS Clusters

Multiple clusters can provide environment isolation, regional
resilience, workload isolation, independent upgrade windows, compliance
boundaries, and blast-radius reduction. They also introduce additional
operational overhead: cluster lifecycle management, add-ons,
credentials, observability, networking, and GitOps targeting.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 3. Cluster Taxonomy

Define clusters using a predictable identity such as
prod-primary-ap-south-1, prod-secondary-ap-southeast-1,
staging-ap-south-1, and nonprod-ap-south-1. Identity should communicate
environment, role, and region. Avoid ambiguous names such as cluster1
and cluster2 in production automation.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 4. Cluster Inventory

Maintain an inventory containing cluster name, AWS account, region, EKS
version, VPC, Argo CD ownership, environment, critical workloads, node
groups, ingress endpoint, secret-management boundary, and
disaster-recovery role. This inventory should be generated or reconciled
where possible rather than maintained manually.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 5. Central Argo CD Model

A central Argo CD control plane can manage multiple EKS clusters. This
provides a single deployment interface and consistent policy model. Its
downside is control-plane blast radius: an Argo CD failure or compromise
can affect every registered cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 6. Dedicated Argo CD Model

Separate Argo CD instances can isolate production from non-production or
isolate business domains. This increases operational overhead but
reduces the number of clusters affected by one control-plane failure.
Highly sensitive production environments often justify stronger
isolation.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 7. Hub-and-Spoke

In a hub-and-spoke model, one Argo CD control plane acts as the hub and
EKS clusters are managed spokes. Cluster credentials and project
restrictions determine which Applications can target each spoke. The hub
must be highly protected because it represents a broad administrative
boundary.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 8. Multi-Cluster Security Boundary

Cluster credentials stored by Argo CD are high-value secrets. A
compromise of a central controller can potentially affect all managed
clusters. Use least privilege, project destination restrictions, network
controls, credential rotation, and strong administrative authentication.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 9. Cluster Registration

Register each target EKS cluster with an explicit identity. Verify that
the Kubernetes endpoint, certificate, credentials, and AWS
authentication mechanism are correct. Do not assume that a successful
registration means the cluster is correctly authorized for every
production Application.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 10. AWS Authentication

Where supported by the chosen Argo CD integration, use AWS IAM-backed
authentication mechanisms rather than long-lived static credentials. The
exact implementation depends on the Argo CD version and AWS/EKS
authentication model. Keep the identity scoped to the resources Argo CD
actually manages.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 11. Cluster Credentials

If static or generated Kubernetes credentials are used, store them only
in the supported Argo CD secret mechanism, restrict access, rotate them,
and audit changes. Never commit cluster credentials into GitOps
repositories.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 12. Cluster Labels

Apply labels such as environment=prod, region=ap-south-1, tier=primary,
and business-unit=platform to registered clusters. ApplicationSet can
use these labels for controlled placement. Protect label changes because
they can alter which workloads are deployed to a cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 13. ApplicationSet Cluster Generator

ApplicationSet can generate Applications for clusters matching labels. A
conceptual pattern is to select clusters with environment=prod and
deploy only the applications appropriate to that fleet. Test selector
changes carefully because a broad selector can suddenly target many
clusters.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 14. Multi-Cluster Repository Layout

A practical layout can combine environment and cluster overlays:
clusters/prod-primary, clusters/prod-secondary, environments/prod, and
applications/`<service>`{=html}. Keep common configuration reusable
while making cluster-specific differences explicit.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 15. Cluster-Specific Configuration

Cluster-specific configuration can include ingress hostnames,
region-specific endpoints, storage classes, node architecture, capacity
settings, external DNS zones, and disaster-recovery role. Avoid
duplicating the entire application configuration just to change a few
values.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 16. Primary and Secondary Clusters

A primary cluster serves normal production traffic while a secondary
cluster is maintained for disaster recovery or regional resilience. The
secondary may run reduced capacity or selected workloads. Its exact
operating model must be documented and tested.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 17. Active-Active

Active-active clusters serve production traffic simultaneously. This
improves regional resilience but requires globally appropriate routing,
data replication, application statelessness, session strategy, and
consistency design. Kubernetes-level redundancy alone does not make the
system active-active.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 18. Active-Passive

Active-passive keeps one cluster primary while another is prepared for
failover. It is simpler operationally but requires a tested promotion
process. Capacity, images, secrets, DNS, certificates, and dependencies
must all be available in the standby environment.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 19. Regional Deployment

Regional clusters reduce dependence on a single AWS region. Applications
must account for data locality, cross-region latency, DNS routing,
replicated data, regional service dependencies, and regulatory
requirements. Deploying the same image digest to two regions preserves
artifact consistency.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 20. Global Traffic Routing

Route53 or another approved traffic-management layer can direct users to
regional ingress endpoints. Health checks and routing policies must be
designed around real application availability rather than only ALB
reachability.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 21. DNS Failover

DNS failover can move traffic from a failed primary region to a
secondary region. DNS TTL and client caching mean failover is not
instantaneous. Test actual client behavior and document expected
recovery time.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 22. Load Balancer Per Cluster

Each EKS cluster normally has its own ingress or load-balancer
resources. Keep DNS records and certificates mapped clearly to the
cluster. During failover, ensure the secondary cluster's ingress
endpoint is already valid and reachable.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 23. AWS Account Isolation

Production clusters may live in separate AWS accounts. Argo CD must be
able to authenticate to each account according to the chosen
architecture. Cross-account permissions should be narrowly scoped and
audited.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 24. Network Connectivity

A central Argo CD control plane needs reliable connectivity to each
managed Kubernetes API endpoint. Private EKS endpoints require
appropriate VPC routing, security groups, DNS, and network connectivity.
If connectivity is lost, reconciliation pauses even though the target
cluster may continue running existing workloads.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 25. Private EKS Endpoint

Private Kubernetes API endpoints reduce public exposure but require the
GitOps control plane to reach the VPC. Design routing before deploying
Argo CD. A common failure is an Argo CD controller that is healthy but
cannot reach a private target cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 26. Public EKS Endpoint

If public API access is required, restrict allowed CIDRs and use strong
authentication and network controls. Do not expose a production
Kubernetes API broadly just because Argo CD needs access.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 27. Cluster NetworkPolicy

NetworkPolicies inside each cluster should control application traffic.
Argo CD's ability to deploy resources does not remove the need for
runtime network isolation. Platform and application namespaces should
have explicit communication rules.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 28. Cluster Add-ons

Every production EKS cluster needs a defined add-on baseline such as VPC
CNI, CoreDNS, kube-proxy, EBS CSI where required, AWS Load Balancer
Controller, metrics components, logging agents, and security tooling.
GitOps should manage compatible application-layer add-ons where
ownership is appropriate.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 29. Add-on Version Alignment

Multi-cluster fleets should track Kubernetes and add-on compatibility.
Do not assume an ApplicationSet can safely deploy identical platform
components to clusters running different Kubernetes versions without
testing.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 30. Cluster Upgrade Strategy

Upgrade clusters progressively: lower environment first, then staging,
then one production cluster, then the remaining production fleet. Argo
CD helps redeploy workloads, but cluster upgrades still require node,
add-on, API compatibility, and workload disruption planning.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 31. Version Skew

Kubernetes components have supported version-skew boundaries. During
multi-cluster operations, document which clusters run which versions and
avoid introducing manifests that depend on APIs unavailable in older
clusters.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 32. Fleet Standardization

Standardize security policies, namespaces, labels, observability,
ingress, and baseline add-ons across the fleet. Allow only deliberate
exceptions. Standardization reduces operational complexity and makes a
failure in one cluster easier to compare against healthy clusters.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 33. Cluster Bootstrap

Bootstrap each cluster with Terraform and a controlled platform layer,
then install Argo CD and its required configuration. The bootstrap
should be repeatable from a clean cluster so disaster recovery does not
depend on undocumented manual actions.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 34. Bootstrap Ordering

A reliable order is AWS networking -\> EKS -\> core EKS add-ons -\>
ingress and storage controllers -\> secrets integration -\> Argo CD -\>
Projects and repositories -\> Applications/ApplicationSets -\>
workloads. Exact ordering depends on the platform architecture.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 35. Terraform and Argo CD Boundary

Terraform should own infrastructure such as AWS accounts, VPCs, EKS
clusters, node groups, IAM, and foundational resources. Argo CD should
generally own Kubernetes application and platform manifests. Avoid two
systems fighting over the same resource.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 36. Shared Resource Ownership

Define ownership for cluster-scoped resources such as CRDs,
ClusterRoles, namespaces, ingress classes, and admission policies. A
platform Application can own shared resources while application
Applications own namespace-scoped resources.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 37. Application Ownership

One Argo CD Application should normally be the clear owner of a
resource. Multiple Applications managing the same object can produce
reconciliation conflicts. Shared objects should have a deliberate owner.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 38. Cluster-Scoped Resource Risk

Cluster-scoped resources can affect every namespace. Changes to
ClusterRoles, CRDs, admission webhooks, NetworkPolicies at the cluster
level, and storage classes deserve higher review than an ordinary
Deployment.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 39. Production Project Restrictions

Use Argo CD Projects to restrict production Applications to approved
repositories, clusters, namespaces, and resource kinds. This prevents an
application team from changing an Application definition to deploy into
an unrelated cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 40. Application Destination Restrictions

A production Project can allow only known production cluster
destinations and approved namespaces. Test this boundary by attempting
an unauthorized destination in a controlled environment.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 41. Multi-Tenant Argo CD

If multiple teams share Argo CD, assign Applications to Projects based
on ownership. Teams should manage their application scope without
receiving unrestricted control of the Argo CD installation or other
teams' production workloads.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 42. Team RBAC

Map enterprise identity groups to Argo CD roles. Read-only users can
inspect status, operators can sync approved applications, and platform
administrators manage projects and clusters. Keep wildcard
administrative permissions exceptional.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 43. Cluster Admin Avoidance

Do not automatically grant every Argo CD-managed application
cluster-admin. Model the actual resources required. Cluster-admin should
be limited to platform-level Applications that genuinely need it.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 44. ApplicationSet Blast Radius

An ApplicationSet is effectively a deployment multiplier. A small change
to a generator can affect dozens of Applications. Require stronger
review and validation for ApplicationSets than for an individual service
value change.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 45. ApplicationSet Git Generator

A Git generator can create Applications from directories or files. The
directory structure becomes deployment input, so path conventions must
be protected. A renamed directory can create or remove applications
depending on the generator design.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 46. ApplicationSet Matrix Generator

Matrix-style generation can combine environments and clusters. This is
powerful for fleet deployment but can multiply mistakes. Validate the
generated Application list before merging production changes.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 47. Previewing Generated Applications

Before merging an ApplicationSet change, render or inspect the expected
generated Applications. Verify names, source paths, destinations,
namespaces, and project assignments.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 48. Progressive Fleet Rollout

Do not necessarily update every production cluster simultaneously.
Deploy to one canary cluster first, observe health, then advance to the
remaining fleet. This reduces the probability that a bad configuration
affects every region.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 49. Cluster Wave Strategy

Fleet rollout can use explicit cluster groups such as canary, primary,
secondary, and remaining. GitOps configuration can encode which cluster
receives a release first. Promotion should stop automatically or
operationally when the canary fails.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 50. Regional Canary

A regional canary can receive a new application version before other
regions. Monitor application error rate, latency, dependency health, and
business KPIs. Only after the canary is stable should the release
advance.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 51. Cross-Cluster Consistency

After deployment, compare desired and live image digests across
clusters. A fleet inventory should identify whether every cluster is
running the intended release or whether a controlled rollout is still in
progress.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 52. Fleet Inventory Example

Maintain a machine-readable inventory such as service, environment,
cluster, region, namespace, desired digest, live digest, chart version,
and rollout status. This becomes valuable during incident response and
audits.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 53. Observability Across Clusters

Centralize metrics and logs with cluster identity labels. Dashboards
should allow operators to compare the same service across regions and
clusters. A problem isolated to one cluster is often visible only when
the fleet is compared.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 54. Alerting Across Clusters

Alert on cluster-specific failures and fleet-wide failures separately. A
single regional outage should not be confused with a global deployment
failure. Include cluster and region in alert labels.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 55. Incident Detection

When one cluster shows increased errors, determine whether the issue is
regional infrastructure, cluster configuration, workload version,
dependency access, or data locality. Compare against the healthy cluster
before changing production configuration.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 56. Deployment Failure in One Cluster

If a fleet deployment succeeds in cluster A but fails in cluster B,
compare Kubernetes versions, add-on versions, admission policies, node
architecture, secrets, network routes, and environment-specific values.
Do not assume the artifact is the cause merely because the timing
matches.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 57. Rollback One Cluster

A multi-cluster rollout can be partially rolled back. If only the canary
cluster is unhealthy, revert that cluster's desired state while keeping
the stable version elsewhere. Document the fleet's mixed-version state
until the incident is resolved.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 58. Global Rollback

If the same release causes customer-impacting behavior across multiple
clusters, revert the production desired state to the previous digest
across the fleet. Use controlled automation rather than manually
changing each cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 59. Data Plane vs Control Plane

Argo CD is part of the deployment control plane. Existing Kubernetes
workloads are the data plane. A temporary Argo CD outage does not
necessarily stop running workloads, but it prevents or delays new
reconciliation. Design monitoring and recovery accordingly.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 60. Argo CD Outage

If central Argo CD is unavailable, existing applications can continue
serving traffic. New releases, drift correction, and some operational
actions are unavailable until recovery. This is why Argo CD itself
requires HA, monitoring, and disaster recovery.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 61. Central Control Plane Failure

A central Argo CD failure can affect deployment operations for every
cluster. Separate instances or a highly available architecture can
reduce this risk. Existing workloads should remain independently
available as much as possible.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 62. Cluster Failure

If one EKS cluster fails while another remains healthy, traffic can be
shifted according to the DR strategy. Argo CD should continue managing
the healthy cluster and should be able to bootstrap or recover the
failed cluster when infrastructure is restored.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 63. Argo CD Cluster Credential Compromise

Treat compromised cluster credentials as a security incident. Restrict
or revoke access, rotate credentials, inspect audit logs, identify
unauthorized Applications or changes, and validate every affected
cluster. Do not only rotate the credential without investigating
potential use.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 64. Repository Compromise

If the GitOps repository is compromised, an attacker may alter desired
state. Protect branches, require reviews, enable security controls,
monitor unexpected changes, and maintain an independent recovery path.
Validate production state against trusted commits during an incident.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 65. Supply Chain and Multi-Cluster

A malicious or vulnerable image promoted across a fleet can affect every
cluster. Immutable digests, image scanning, SBOMs, signing, admission
verification, and controlled promotion reduce the blast radius.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 66. Image Consistency

Use the same digest when the same release is intended across clusters.
If a region requires a different build, document why and treat it as a
separate release candidate rather than silently diverging.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 67. Cluster-Specific Image Exceptions

Architecture differences such as amd64 versus arm64 may require
multi-architecture images or separate artifacts. Prefer a properly built
multi-architecture image when practical, and verify that the digest and
manifest list behavior match the deployment strategy.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 68. Node Architecture

If clusters contain different CPU architectures, verify that images
support both. Helm configuration can express architecture-specific
scheduling only when necessary. Avoid hardcoding architecture
assumptions into generic application configuration.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 69. Storage Differences

EBS-backed storage is regional and cluster-specific. Stateful workloads
require explicit storage-class and volume recovery planning. GitOps can
recreate Kubernetes objects, but it cannot magically recreate lost data.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 70. Stateful Workloads

Multi-cluster deployment is easier for stateless services. Stateful
systems require replication, backup, recovery, consistency, and failover
design. Never claim multi-cluster redundancy for a database without
validating the database's replication model.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 71. Secret Replication

Secrets required by a secondary cluster must be available through an
approved cross-region or cross-account secret strategy. Do not copy
production secrets manually into every cluster. Workload identity and
secret-store policies should be environment and cluster aware.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 72. Certificate Management

Each cluster's ingress may use regional certificates or a shared
certificate strategy. Renewal must work even when one cluster is
unavailable. Monitor certificate expiration across the fleet.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 73. External DNS

If DNS records are managed automatically, define which controller owns
them and which cluster is authoritative for each hostname. Multiple
controllers must not fight over the same DNS record without a deliberate
design.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 74. Failover Readiness

A secondary cluster is not DR-ready merely because Argo CD shows it as
Synced. Validate traffic routing, secrets, storage, dependencies,
capacity, certificates, monitoring, and application behavior.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 75. DR Exercise

Perform scheduled failover exercises. Record recovery time objective,
recovery point objective, traffic-switch duration, dependency failures,
manual steps, and gaps. Update GitOps and infrastructure automation
based on findings.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 76. RTO and RPO

RTO defines how quickly service should be restored; RPO defines
acceptable data loss. GitOps primarily helps configuration recovery.
Data RPO depends on the application's database and storage replication
strategy.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 77. Cluster Rebuild

A cluster rebuild should use Terraform for AWS infrastructure and GitOps
for Kubernetes desired state. Once the cluster and Argo CD are ready,
Applications can reconcile workloads. Validate external dependencies
before declaring recovery complete.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 78. Cluster Decommissioning

Before deleting a cluster, identify Applications, traffic routes,
persistent volumes, secrets, DNS records, monitoring targets, and
dependencies. Remove Argo CD registration only after workload migration
and data validation are complete.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 79. Cluster Onboarding Runbook

Onboard a new cluster by creating infrastructure, installing required
add-ons, validating networking, registering it with Argo CD, applying
the appropriate Project restrictions, syncing a low-risk application,
validating observability, and only then enabling the production fleet
selector.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 80. Cluster Offboarding Runbook

Drain traffic, stop new promotions, migrate or recover stateful
workloads, validate the replacement cluster, remove Applications from
the old destination, revoke cluster credentials, remove monitoring
targets, and finally decommission infrastructure.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 81. Fleet Upgrade Runbook

Upgrade one non-production cluster, validate workloads and add-ons,
upgrade staging, then one production canary cluster. Monitor before
proceeding to the remaining production fleet. Keep a documented rollback
or recovery path.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 82. Fleet Security Review

Review cluster credentials, Project destinations, repository
permissions, admission policies, service accounts, AWS IAM roles,
network paths, and cluster API exposure. Repeat after major topology
changes.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 83. Fleet Capacity

Plan aggregate capacity across clusters. During failover, the surviving
cluster may need to handle more traffic. A standby cluster without
sufficient node and pod capacity is not a credible failover target.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 84. Overprovisioning for DR

Balance standby capacity against cost. Critical workloads may need warm
capacity while less critical workloads can scale during failover.
Document the trade-off rather than assuming every service needs full
duplicate capacity.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 85. Cost Optimization

Multi-cluster platforms increase baseline cost through control planes,
nodes, load balancers, NAT gateways, observability, and duplicated
platform services. Use workload criticality to determine which services
require multi-region or multi-cluster redundancy.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 86. Production Multi-Cluster Architecture

``` text
                         GitOps Repository
                               |
                        Protected Main
                               |
                         GitOps CI
                               |
                        +------+------+
                        |   Argo CD   |
                        | Control     |
                        +------+------+
                               |
              +----------------+----------------+
              |                                 |
       Production Cluster A              Production Cluster B
       Primary / Region A                Secondary / Region B
              |                                 |
        ALB / Ingress                       ALB / Ingress
              |                                 |
        Application Pods                   Application Pods
              |                                 |
        Metrics / Logs                     Metrics / Logs
              \_________________ ______________/
                                |
                         Central Observability
```

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 87. Production Promotion Across Clusters

For a fleet release, promote the tested digest to a canary cluster
first, verify health, then promote to the remaining production clusters.
Keep the GitOps commit or generated ApplicationSet configuration
explicit about which clusters are in each rollout wave.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 88. Cluster-Level Rollback

Rollback can be represented as a Git change to the affected cluster or
fleet configuration. Argo CD reconciles the old digest. For fleet-wide
failures, automate the rollback carefully and verify that all target
clusters converge to the known-good state.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 89. Failure Scenario: Wrong Selector

If an ApplicationSet selector accidentally matches every production
cluster, pause synchronization, inspect generated Applications, remove
the incorrect selector, and restore the intended cluster labels or
generator rules. This scenario demonstrates why ApplicationSet changes
require elevated review.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 90. Failure Scenario: Private API Unreachable

If Argo CD cannot reach a private EKS endpoint, check VPC routing,
security groups, DNS, network ACLs, proxy configuration, and Argo CD
egress. Existing workloads may remain healthy while deployment
operations fail.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 91. Failure Scenario: One Region Degraded

Compare the affected region with the healthy region. Inspect AWS service
health, cluster nodes, ingress, dependencies, application metrics, and
configuration. If the DR strategy is active-passive, shift traffic only
after validating the secondary's readiness.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 92. Failure Scenario: Partial Fleet Rollout

If the canary fails, stop promotion and keep the remaining clusters on
the previous version. Fix or rollback the canary. Do not automatically
continue the rollout simply because some clusters succeeded.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 93. Failure Scenario: Credential Expiration

If one cluster stops reconciling because its credential expired,
identify the exact credential and affected Applications, rotate through
the approved process, verify connectivity, and confirm that the cluster
converges to the desired state.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 94. Failure Scenario: Cluster Version Mismatch

If a manifest works on one cluster but fails on another, compare
Kubernetes API versions and admission policies. Use version-compatible
manifests or complete the cluster upgrade before promoting the release.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 95. Senior Interview: Why Multiple Clusters?

I use multiple EKS clusters when isolation, regional resilience,
compliance, independent lifecycle, or blast-radius reduction justifies
the operational cost. I don't create clusters merely for complexity; the
topology should solve a concrete reliability or security requirement.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 96. Senior Interview: Central Argo CD

A central Argo CD simplifies fleet management and gives consistent
GitOps operations, but it becomes a high-value control plane. I protect
it with HA, strong authentication, Project restrictions, least-privilege
cluster access, monitoring, and disaster recovery. For high-isolation
environments I consider separate Argo CD instances.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 97. Senior Interview: How Do You Deploy to Multiple Clusters?

I label registered clusters with environment and role, then use Argo CD
Projects and ApplicationSets to generate only the Applications that
belong on each cluster. For production, I use progressive fleet rollout:
canary cluster first, observe, then expand.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 98. Senior Interview: What If One Cluster Fails?

I separate workload availability from deployment control. If one cluster
fails, traffic is moved according to the DR design while the healthy
cluster continues serving. I rebuild the failed EKS infrastructure with
Terraform, bootstrap Argo CD, reconcile workloads, restore secrets and
dependencies, and validate before returning traffic.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 99. Senior Interview: How Do You Prevent Cross-Cluster Deployment?

I use explicit Application destinations, Project destination
restrictions, protected cluster labels, least-privilege credentials, and
review controls around ApplicationSet generators. A service should not
be able to change its destination to an unrelated production cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 100. Senior Interview: Active-Active vs Active-Passive

Active-active improves utilization and can reduce failover time but
requires strong data and traffic architecture. Active-passive is simpler
but requires tested failover and sufficient standby capacity. The choice
depends on RTO, RPO, data consistency, cost, and application
architecture.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 101. Senior Interview: DR Readiness

I don't consider a cluster DR-ready because it is merely running Argo
CD. I validate traffic routing, secrets, certificates, storage,
dependencies, capacity, observability, and actual application
transactions. I run scheduled failover exercises and measure RTO and
RPO.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 102. Senior Interview: Terraform vs Argo CD

Terraform owns AWS infrastructure and foundational cluster resources.
Argo CD owns Kubernetes application and platform desired state. I avoid
having both systems manage the same resource because that creates
competing sources of truth.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 103. Senior Interview: Fleet Rollback

If a canary fails, I stop the rollout and keep other clusters on the
previous version. If the release is globally harmful, I revert the
fleet's desired digest to the last known-good version and verify
convergence across every cluster.

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

## 104. Final Multi-Cluster Checklist

-   [ ] Cluster identity and ownership are explicit
-   [ ] Environment and region labels are protected
-   [ ] Cluster inventory exists
-   [ ] Argo CD architecture has a defined blast radius
-   [ ] Cluster authentication is least privileged
-   [ ] Private API connectivity is tested
-   [ ] Projects restrict destinations
-   [ ] Application ownership is clear
-   [ ] ApplicationSet selectors are reviewed
-   [ ] Fleet rollout is progressive
-   [ ] Canary cluster is defined
-   [ ] Observability includes cluster and region dimensions
-   [ ] Secrets are available securely in every target cluster
-   [ ] Certificates and DNS are covered by DR
-   [ ] Stateful data replication is explicitly designed
-   [ ] Capacity supports failover
-   [ ] Cluster upgrades are progressive
-   [ ] Cluster rebuild is automated
-   [ ] Cluster offboarding is documented
-   [ ] Credential rotation is tested
-   [ ] Rollback is tested at cluster and fleet levels
-   [ ] DR exercises measure RTO/RPO
-   [ ] Production architecture is documented

### Production implementation checks

-   Make the target cluster explicit.
-   Protect every mechanism capable of changing deployment placement.
-   Separate infrastructure ownership from Kubernetes application
    ownership.
-   Test failure of individual clusters and the GitOps control plane.
-   Keep disaster recovery executable rather than theoretical.

### Operational questions

1.  Which AWS account, region, cluster, and namespace are targeted?
2.  What prevents an application from deploying to another cluster?
3.  What happens if this cluster or region fails?
4.  How is the workload restored?
5.  How do we prove all clusters converged to the intended release?

---