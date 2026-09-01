# 19 --- Application Autoscaling --- Production DevOps Capstone

> Deep production guide for Kubernetes HPA, VPA, KEDA, Prometheus
> metrics, Prometheus Adapter, EKS node autoscaling, Karpenter, Cluster
> Autoscaler, resource requests/limits, scheduling, scaling behavior,
> dependency protection, cost optimization, failure scenarios,
> production YAMLs, runbooks, and senior DevOps interviews.

## Chapter Objective

This chapter treats autoscaling as a complete production feedback system
rather than simply enabling HPA.

## 1. Autoscaling Objectives

Production autoscaling should maintain availability and latency while
using infrastructure efficiently. Scaling is a feedback system: metrics
indicate demand, controllers change replica counts or capacity,
scheduling places workloads, and the system waits for new capacity to
become usable. Good designs account for metric delay, startup time,
workload behavior, and failure conditions.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 2. The EKS Scaling Layers

``` text
User Traffic
    |
    v
ALB / Service
    |
    v
Pods <---- HPA / KEDA
    |
    v
Kubernetes Scheduler
    |
    v
Nodes <---- Cluster Autoscaler / Karpenter
    |
    v
EC2 / Capacity Providers

VPA can recommend or modify pod resources.
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 3. HPA Fundamentals

Horizontal Pod Autoscaler changes the number of pod replicas based on
observed metrics. It is appropriate for stateless workloads and many
horizontally scalable services. HPA should be paired with realistic
resource requests, reliable readiness behavior, and enough cluster
capacity to schedule the desired replicas.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 4. HPA Control Loop

The HPA controller periodically reads metrics, calculates desired
replicas, applies scaling limits and behavior rules, and repeats.
Scaling is not instantaneous. Metrics collection, controller evaluation,
scheduling, image pulling, application startup, readiness, and ALB
registration all introduce delay.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 5. Replica Calculation

For resource utilization, HPA compares observed utilization against the
target and derives a desired replica count. The practical result depends
on missing metrics, readiness, tolerance, stabilization, and behavior
settings. Never design capacity around the assumption that HPA reacts
instantly to a traffic spike.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 6. Resource Requests Matter

CPU and memory utilization percentages are calculated relative to
resource requests. A workload with unrealistic requests can produce
misleading HPA behavior. Requests should represent realistic scheduling
needs, while limits should be chosen deliberately rather than copied
blindly.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 7. CPU-Based HPA

CPU-based HPA is useful when CPU correlates with request load. It is
less effective when a service is I/O-bound, waiting on databases, or
limited by an external API. Use application-level metrics when CPU does
not represent user demand.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 8. Memory-Based HPA

Memory-based scaling can help workloads whose memory pressure grows with
concurrency or data volume, but memory is often less elastic than CPU.
Scaling on memory can also hide leaks: replicas keep increasing while
the underlying leak remains.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 9. Custom Metrics

Custom metrics allow scaling on business or application signals such as
requests per second, queue depth, active sessions, or latency-related
indicators. Metric quality is critical; noisy or delayed metrics can
cause oscillation.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 10. External Metrics

External metrics can represent signals outside Kubernetes, such as cloud
queue depth. Use them when workload demand is naturally expressed
outside pod CPU/memory. Protect metric access and validate behavior
during metric-server outages.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 11. Prometheus Adapter

Prometheus Adapter can expose selected Prometheus metrics through
Kubernetes custom or external metrics APIs for HPA. Keep the query
efficient, stable, and labeled so the metric maps unambiguously to the
intended workload.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 12. Metric Design

A scaling metric should be proportional to work, respond quickly enough
for the workload, be available during normal operation, and have
predictable units. Avoid metrics that disappear during load exactly when
the autoscaler needs them most.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 13. Requests Per Second

RPS is often a strong metric for stateless HTTP services. A target can
represent sustainable request load per replica. Test the relationship
between RPS and latency because identical request rates can have very
different computational costs.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 14. Queue Depth

Queue depth is a natural scaling signal for asynchronous workers. A
rising queue means demand exceeds processing capacity. Combine queue
depth with processing rate and maximum worker capacity to avoid
uncontrolled scaling.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 15. Latency-Based Scaling

Latency can be useful but is often noisy because it reflects
dependencies and network conditions. Prefer a stable service-level
metric or combine latency with request rate rather than scaling
aggressively on every transient latency spike.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 16. HPA v2

The autoscaling/v2 API supports multiple metrics and behavior controls.
Production manifests should explicitly define minReplicas, maxReplicas,
metrics, and appropriate scale-up/scale-down behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 17. Production HPA Example

``` yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalogue
  namespace: catalogue
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalogue
  minReplicas: 3
  maxReplicas: 20
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 18. MinReplicas

minReplicas defines the lower bound. Production services should maintain
enough warm capacity for normal traffic, availability-zone failures,
rolling updates, and expected bursts. Setting minReplicas to one is
rarely sufficient for a critical production service.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 19. MaxReplicas

maxReplicas is a safety boundary as well as a capacity setting. It
prevents runaway scaling from exhausting cluster resources or external
dependencies. Choose it based on tested application capacity, downstream
limits, and business demand.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 20. Scale-Up Behavior

Scale-up should generally be faster than scale-down. Slow scale-up
causes user-visible latency while waiting for capacity. Fast scale-up
can be expensive, so test startup time and set policies according to
workload characteristics.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 21. Scale-Down Behavior

Scale-down should be conservative. Rapid scale-down can cause
oscillation and remove capacity just before another burst. Stabilization
windows and controlled percentage policies help avoid flapping.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 22. Stabilization Windows

A stabilization window prevents the autoscaler from reacting immediately
to every metric fluctuation. Use longer windows for scale-down when
traffic is bursty, while avoiding so much delay that unused capacity
persists unnecessarily.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 23. Multiple HPA Metrics

When multiple metrics are configured, HPA calculates desired capacity
from each and uses the recommendation that requires the greatest number
of replicas. This is useful for protecting both compute capacity and
workload-specific demand.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 24. HPA and Readiness

New pods that are not ready should not be treated as equivalent to
healthy serving capacity. Application startup and readiness design
directly influence effective scaling speed.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 25. Startup Time

A five-second HPA decision is not useful if a pod takes two minutes to
pull its image, start, load data, and become ready. Capacity planning
must include image size, node provisioning, scheduling, init containers,
startup probes, and application warm-up.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 26. Startup Probes

Startup probes prevent liveness behavior from killing slow-starting
applications. They should reflect realistic worst-case startup times
without hiding genuine startup failures.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 27. Readiness Probes

Readiness determines when a pod can receive traffic. It should represent
actual serving capability, including required initialization, while
avoiding unnecessarily expensive dependency checks.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 28. Liveness Probes

Liveness should detect unrecoverable application states, not temporary
dependency failures. A poorly designed liveness probe can turn a
downstream outage into a restart storm.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 29. HPA and ALB

HPA increases pod count, but the new pod becomes useful only after
scheduling, startup, readiness, Service endpoint propagation, and ALB
target registration/health checks. Measure this complete
time-to-capacity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 30. HPA and Rolling Updates

A Deployment rollout and HPA can interact. During a rollout, temporary
extra replicas may consume capacity. Set maxSurge, maxUnavailable, HPA
limits, and node capacity so the cluster can accommodate the rollout.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 31. PodDisruptionBudget

PDBs protect against voluntary disruptions but do not guarantee capacity
during every failure. Ensure PDB settings are compatible with
autoscaling and node maintenance; an overly strict PDB can block node
drains.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 32. Topology Spread

Use topology spread constraints or appropriate anti-affinity to
distribute replicas across failure domains. Autoscaling should not
simply create many replicas on one node or one Availability Zone.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 33. Node Autoscaling

HPA creates pods, but those pods may remain Pending when nodes lack
capacity. A node autoscaler such as Karpenter or Cluster Autoscaler is
therefore often required for elastic EKS workloads.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 34. HPA + Karpenter Flow

``` text
Traffic increases
      |
      v
HPA increases replicas
      |
      v
Pods become Pending
      |
      v
Karpenter observes unschedulable demand
      |
      v
EC2 capacity launched
      |
      v
Pods scheduled
      |
      v
Readiness -> Service -> ALB
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 35. Karpenter Overview

Karpenter dynamically provisions compute capacity based on unschedulable
pod requirements and node configuration. It can respond quickly to
diverse workload shapes, but production configuration must constrain
instance families, zones, architectures, capacity types, budgets, and
disruption behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 36. Cluster Autoscaler Overview

Cluster Autoscaler adjusts node-group size based on pending pods and
node utilization/availability characteristics. It is predictable when
node groups are well designed, but multiple node groups and complex
scheduling rules can increase decision complexity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 37. Karpenter vs Cluster Autoscaler

Karpenter is often attractive for flexible EKS capacity because it can
provision nodes from scheduling requirements rather than only resizing
predefined node groups. Cluster Autoscaler remains useful for
organizations standardized on managed node groups or fixed node pools.
Choose one architecture deliberately and avoid overlapping controllers
that fight over capacity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 38. NodePool Design

Define NodePools or equivalent capacity policies by workload
requirements. Separate general-purpose, compute-optimized,
memory-optimized, and special workloads when their requirements differ
materially. Avoid dozens of nearly identical pools that make scheduling
and cost analysis difficult.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 39. Capacity Types

Use on-demand capacity for critical baseline workloads and consider Spot
capacity for interruption-tolerant workloads. Stateful,
latency-critical, or disruption-sensitive workloads may need stronger
capacity guarantees.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 40. Spot Strategy

Spot can reduce cost but introduces interruption risk. Use multiple
instance types and Availability Zones, maintain replicas across failure
domains, and ensure workloads tolerate node termination. Never put the
entire production baseline on a single Spot capacity type.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 41. Instance Selection

Node capacity should match pod CPU, memory, architecture, storage,
networking, and daemon overhead. Oversized nodes can waste money;
undersized nodes increase scheduling pressure and provisioning churn.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 42. DaemonSet Overhead

Node autoscaling must account for DaemonSets such as monitoring,
networking, security, and logging agents. The usable capacity of a node
is lower than its raw instance CPU and memory.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 43. Requests and Bin Packing

Schedulers place pods using resource requests. Accurate requests improve
bin packing and autoscaling decisions. Inflated requests create
unnecessary nodes; underestimated requests create contention and poor
performance.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 44. Limits

Limits protect nodes from unbounded resource usage but can also cause
throttling or OOM kills when set poorly. CPU throttling and memory
limits should be tested against application behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 45. QoS Classes

Kubernetes QoS classes influence resource behavior during node pressure.
Production workloads should have deliberate requests and limits rather
than inheriting arbitrary defaults.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 46. Vertical Pod Autoscaler

VPA analyzes workload resource usage and can recommend or apply resource
changes depending on mode. It is valuable for rightsizing requests, but
automatic changes can restart pods and interact with HPA.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 47. VPA + HPA Conflict

Do not independently autoscale the same resource dimension with HPA and
VPA without a deliberate design. If HPA uses CPU utilization while VPA
changes CPU requests, the denominator changes and can destabilize
scaling.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 48. VPA Recommendation Mode

Recommendation-only VPA is a safe starting point for production. Use
observed recommendations to adjust requests through controlled GitOps
changes before enabling automatic updates.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 49. KEDA

KEDA can scale workloads from event-driven metrics such as queues,
streams, and cloud services. It is particularly useful when work is
naturally represented by an event source rather than CPU utilization.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 50. HPA vs KEDA

HPA is a general Kubernetes autoscaling mechanism. KEDA adds
event-driven scaling and can expose event-source metrics to HPA. Choose
the model that matches the workload instead of adding KEDA everywhere.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 51. Scale to Zero

Scale-to-zero can reduce cost for non-critical or sporadic workloads but
introduces cold-start latency and dependency on the scaler/event source.
It is usually inappropriate for a latency-sensitive production API
without a deliberate warm-up strategy.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 52. Event-Driven Worker Example

``` yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: order-worker
  namespace: order
spec:
  scaleTargetRef:
    name: order-worker
  minReplicaCount: 2
  maxReplicaCount: 30
  triggers:
    - type: rabbitmq
      metadata:
        queueName: orders
        mode: QueueLength
        value: "20"
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 53. Queue Scaling

For queue workers, define a target backlog per replica based on
processing time and required drain time. If one worker processes 10
messages per second and backlog grows to 10,000, merely adding one
worker will not meet a strict recovery objective.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 54. Kafka Consumer Scaling

Kafka scaling must respect partitions and consumer-group semantics.
Adding consumers beyond available partitions does not necessarily
increase parallelism. Partition count, consumer lag, processing rate,
and ordering requirements must be considered together.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 55. Scaling Stateful Applications

Stateful workloads require more caution because replicas may represent
shards, leaders, storage attachments, or quorum members. HPA should not
be applied simply because CPU rises. Use application-specific scaling
semantics and verify storage and consistency behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 56. Database Autoscaling

Do not blindly scale database clients when application traffic
increases. More application replicas can create more database
connections and overload the database. Set connection pools, database
capacity limits, and backpressure.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 57. Downstream Protection

Autoscaling a service without scaling or protecting its dependencies can
amplify an outage. Apply concurrency limits, queueing, circuit breakers,
connection-pool limits, and rate limits where appropriate.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 58. Thundering Herd

When many replicas start simultaneously, they may all initialize caches,
open database connections, or call external APIs. Staggered rollout and
controlled scale-up can prevent a scaling event from becoming a
dependency outage.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 59. Connection Pool Scaling

Total database connections can roughly grow with replica count
multiplied by per-pod pool size. Calculate the maximum and ensure it
remains below database and proxy limits.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 60. Cache Warm-Up

If new replicas require expensive cache warm-up, scaling too quickly can
increase latency. Consider lazy loading, bounded concurrency, shared
caches, or prewarming mechanisms.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 61. Image Pull Time

Large container images slow scale-out. Use small production images,
efficient layers, regional registries, and pre-pulling only where
justified. ECR should be used with immutable versioned image references.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 62. Node Startup Time

Karpenter or another node provisioner adds capacity only after demand
appears. Estimate EC2 launch, bootstrap, CNI initialization, daemon
startup, image pull, and application readiness time when setting
autoscaling policies.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 63. Warm Capacity

Maintain baseline nodes or replicas for workloads that require immediate
response. Pure scale-from-zero designs can be operationally fragile for
critical services.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 64. Predictive Scaling

If traffic follows predictable patterns, scheduled scaling or capacity
planning can provide warm capacity before known peaks. Combine
predictable schedules with reactive autoscaling rather than relying on
HPA alone.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 65. Scheduled Scaling

Scheduled replica changes can prepare for business hours or known
events. Keep the schedule in GitOps where practical and ensure HPA
min/max bounds do not conflict with the desired behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 66. Autoscaling During Deployment

Deployment automation should not fight autoscaling. Avoid hardcoding
replica changes after HPA is installed unless the change is part of the
intended HPA baseline configuration.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 67. GitOps Ownership

Define ownership clearly: Git declares HPA and scaling policies; the HPA
controller changes the live replica count. Do not let Argo CD repeatedly
overwrite the controller's desired replica count.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 68. Argo CD and HPA Replicas

When HPA controls a Deployment, avoid configuring Argo CD to constantly
enforce a fixed replicas field unless the GitOps strategy explicitly
ignores that live-field difference. Otherwise, GitOps can scale the
application down immediately after HPA scales it up.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 69. Sync Options

Use appropriate Argo CD diff/ignore configuration only for fields that
are genuinely controller-managed. Broad ignore rules hide drift and can
conceal security or availability changes.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 70. Production HPA GitOps Pattern

Store the HPA manifest with the application. Environment-specific
min/max values should be managed through Helm values or overlays. Keep
production limits conservative and review them like application capacity
configuration.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 71. Autoscaling Observability

Monitor current replicas, desired replicas, metric values, HPA
conditions, Pending pods, node count, node provisioning latency, pod
startup time, target health, request latency, errors, and downstream
saturation.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 72. Key HPA Metrics

Useful signals include desired versus current replicas, CPU/memory
utilization, custom metric values, scaling events, and time spent
waiting for capacity. Correlate them with user-facing SLOs.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 73. Node Autoscaling Metrics

Track node count, provisioning latency, pending pod age, unschedulable
reasons, instance types, Spot interruptions, node utilization, and
consolidation/disruption events.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 74. Scaling Saturation

An application at maxReplicas is a critical operational state when
demand remains high. Alert on sustained maxReplicas, not just the moment
the value is reached.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 75. Pending Pod Alert

Alert on pods remaining Pending beyond a short expected scheduling
window. Investigate insufficient CPU/memory, topology constraints,
taints, node selectors, affinity, quota, image pull, and
node-provisioner failures.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 76. No Metrics Scenario

If metrics disappear, HPA may stop making useful scaling decisions.
Monitor metrics-server, Prometheus Adapter, KEDA, and their dependencies
separately. Define a safe baseline replica count.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 77. Metrics API Troubleshooting

Check Kubernetes metrics APIs, adapter logs, Prometheus query results,
service endpoints, RBAC, TLS, and metric labels. Confirm the exact
metric returned for the target rather than assuming the query is
correct.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 78. HPA Troubleshooting

Use kubectl describe hpa to inspect current/desired replicas, metrics,
conditions, and events. Then inspect the Deployment, pods, metrics
provider, and scheduler state. Scaling failures often occur outside the
HPA object itself.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 79. Why HPA Is Not Scaling Up

Check whether the metric exceeds target, maxReplicas has been reached,
metrics are available, the target reference is correct, pods have
appropriate requests, and the HPA controller reports a condition or
event explaining the decision.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 80. Why HPA Is Not Scaling Down

Check stabilization windows, current metric values, minReplicas,
multiple metrics, missing metrics, and recent scale-up activity. A
delayed scale-down can be intentional protection against oscillation.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 81. HPA Oscillation

Oscillation usually indicates noisy metrics, aggressive policies,
workload startup effects, or a target that sits near the natural
operating point. Increase stabilization, smooth the metric, or choose a
more representative signal.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 82. Runaway Scaling

Runaway scaling can come from a bad metric, retry storm, dependency
failure, or an application that creates work faster as replicas
increase. Cap maxReplicas and investigate the feedback loop rather than
continuously increasing capacity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 83. Scaling Amplifies Failure

If a database is failing, HPA may add replicas because requests become
slower and CPU rises. More replicas can create more database connections
and worsen the outage. Autoscaling must be combined with dependency
protection.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 84. Circuit Breakers

Circuit breakers can prevent unhealthy downstream calls from consuming
all application resources. They complement autoscaling by controlling
work rather than merely adding more consumers.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 85. Rate Limiting

Rate limits protect services from traffic spikes and misbehaving
clients. Autoscaling should increase capacity within safe limits rather
than attempting to serve unlimited demand.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 86. Backpressure

Backpressure prevents upstream producers from overwhelming consumers.
Queues, bounded worker pools, concurrency limits, and HTTP 429/503
responses can be safer than unrestricted scale-out.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 87. Graceful Scale-In

Scale-in should allow in-flight requests to finish and workers to stop
accepting new work. For queue consumers, stop consumption, finish or
safely requeue work, then terminate.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 88. PreStop Hooks

A preStop hook can initiate application shutdown behavior before process
termination. Do not rely on it alone: termination grace period,
readiness transitions, endpoint updates, and load-balancer
deregistration must be aligned.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 89. Termination Grace

terminationGracePeriodSeconds should cover realistic request completion
and shutdown time. Setting it too low causes dropped work; setting it
extremely high can slow node maintenance.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 90. PDB and Scale-In

PDB does not directly control HPA scale-down. It protects voluntary
disruptions. Test how application scaling, node consolidation, and
maintenance interact so availability remains acceptable.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 91. Cluster Capacity Baseline

Keep enough baseline capacity for system DaemonSets, critical platform
components, and minimum application replicas. Node autoscaling should
provide burst capacity, not be required for every normal operation.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 92. Reserved Capacity

For predictable production workloads, committed or reserved capacity can
lower cost while maintaining a baseline. Use flexible on-demand or Spot
capacity for bursts where appropriate.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 93. Cost-Aware Autoscaling

Track cost per request, cost per successful job, and idle capacity.
Scaling to maximum performance at all times is not automatically
optimal; scaling too aggressively can also overload downstream systems.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 94. Rightsizing

Use VPA recommendations, historical usage, and load tests to tune
requests. Rightsizing improves scheduling density and reduces the number
of nodes required by both Cluster Autoscaler and Karpenter.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 95. Overprovisioning

A small pool of low-priority placeholder pods can reserve scheduling
capacity for latency-sensitive workloads in some architectures. This is
an advanced pattern and should be used only when the operational benefit
justifies complexity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 96. Priority Classes

Critical workloads can use appropriate PriorityClasses so essential pods
receive scheduling preference during resource pressure. Avoid giving
every application highest priority.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 97. Resource Quotas

Namespace ResourceQuota can prevent one team from consuming all cluster
capacity. Quotas should account for HPA maxReplicas so autoscaling does
not fail unexpectedly because the namespace hits its quota.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 98. LimitRange

LimitRange can provide default or minimum/maximum resource constraints.
Defaults should be carefully chosen because they directly influence
scheduling and HPA behavior.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 99. Node Taints and Tolerations

Special node pools can be protected with taints and accessed only by
workloads with matching tolerations. This is useful for GPU,
memory-heavy, or isolated workloads but can create unschedulable pods if
configured incorrectly.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 100. Node Affinity

Use node affinity to express legitimate placement requirements. Overly
restrictive affinity reduces autoscaling flexibility and can leave pods
Pending even when total cluster capacity appears sufficient.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 101. Topology Constraints

Topology spread constraints improve resilience but may increase capacity
requirements. Test them with HPA maxReplicas and node provisioning so
scale-out remains feasible.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 102. Architecture Awareness

If workloads use ARM and x86 nodes, container images must support the
required architectures and scheduling rules must be explicit. A
scale-out failure can occur if the only compatible node capacity is
unavailable.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 103. GPU Scaling

GPU workloads need specialized node provisioning, resource requests,
taints, and capacity planning. GPU scarcity can make autoscaling slow or
expensive, so queueing and batching may be more effective than unlimited
replica growth.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 104. Batch Jobs

Kubernetes Jobs should not be managed by HPA. Use Job parallelism,
completions, queue-driven workers, or an event-driven scheduler
appropriate to the workload.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 105. CronJobs

CronJobs create scheduled workloads and can produce synchronized load
spikes. Set concurrencyPolicy, starting deadlines, resource requests,
and job history limits deliberately.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 106. Autoscaling and DR

DR clusters should have a tested baseline capacity and scaling
configuration. Do not assume the secondary region can instantly
provision enough capacity for the full production workload.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 107. Regional Capacity Planning

Validate that required EC2 instance types, Spot capacity, quotas, and
networking are available in the recovery region. A mathematically
correct HPA maxReplicas is meaningless if AWS service quotas prevent
provisioning.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 108. AWS Service Quotas

Check EC2 and related quotas for the instance families used by node
provisioning. Capacity limits can appear as Kubernetes Pending pods even
though manifests and autoscaling logic are correct.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 109. Load Testing

Test autoscaling with realistic traffic profiles: gradual increase,
sudden spike, sustained peak, sudden drop, dependency degradation, node
loss, and rollout during peak load. Measure time-to-capacity and SLO
impact.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 110. Chaos Testing

Simulate pod failures, node termination, metrics-provider outage,
image-pull delay, downstream slowness, and Spot interruption. The goal
is to verify the feedback loop under failure rather than only normal
load.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 111. Production Failure Scenario: Traffic Spike

1.  ALB request rate rises. 2. HPA metric crosses target. 3. Replica
    count increases. 4. Scheduler evaluates placement. 5. Node
    autoscaler adds capacity if necessary. 6. Pods start and become
    ready. 7. ALB registers healthy targets. 8. Latency returns to SLO.
    Monitor every transition.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 112. Production Failure Scenario: Pods Pending

First inspect scheduling events and pod requests. Then check node
capacity, taints, affinity, topology constraints, quota, architecture,
and node-provisioner status. Do not increase HPA maxReplicas when the
actual issue is lack of schedulable capacity.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 113. Production Failure Scenario: MaxReplicas Reached

Determine whether demand is legitimate or caused by dependency failure.
Check request rate, latency, error rate, CPU, memory, queue depth,
database saturation, connection count, and downstream rate limits.
Increase maxReplicas only after confirming the rest of the system can
support it.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 114. Production Failure Scenario: Scale-Out Causes DB Outage

Reduce or cap application concurrency, protect the database, and
stabilize traffic. Review connection-pool sizing and HPA behavior. Do
not respond by adding even more application replicas.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 115. Production Failure Scenario: Karpenter Not Provisioning

Check unschedulable pod events, NodePool requirements, EC2 instance
availability, subnet/security configuration, IAM permissions, quotas,
architecture constraints, and controller logs. Identify the first failed
dependency.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 116. Production Failure Scenario: HPA Metrics Missing

Verify the metrics provider, API registration, RBAC, Prometheus/adapter
query, labels, and controller conditions. Maintain a safe baseline
replica count so a metrics outage does not immediately remove
availability.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 117. Production Failure Scenario: Rapid Scale-Up/Down

Inspect metric noise, policy percentages, stabilization windows,
workload startup behavior, and traffic pattern. Smooth the metric or
tune behavior rather than manually changing replicas every time.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 118. Production Failure Scenario: Node Consolidation Disrupts Service

Verify replicas are distributed, PDB is appropriate, termination
behavior is graceful, and disruption budgets are compatible with
capacity. Protect critical workloads with appropriate scheduling and
capacity policies.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 119. Production Runbook: HPA Incident

1.  Check user SLOs. 2. Check HPA desired/current replicas. 3. Check
    metrics. 4. Check maxReplicas. 5. Check Pending pods. 6. Check node
    capacity/provisioning. 7. Check downstream saturation. 8. Stabilize
    traffic if necessary. 9. Make the smallest safe change. 10. Record
    root cause.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 120. Production Runbook: Capacity Exhaustion

1.  Identify exhausted resource. 2. Determine whether demand or
    misconfiguration caused it. 3. Check node autoscaler. 4. Check AWS
    quotas/capacity. 5. Protect critical workloads. 6. Reduce
    non-critical load if needed. 7. Restore capacity. 8. Validate
    scheduling and SLOs.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 121. Production Runbook: Scaling Policy Change

1.  Review historical metrics. 2. Load-test the proposed values. 3.
    Verify downstream limits. 4. Update Git. 5. Run CI validation. 6.
    Deploy through Argo CD. 7. Observe scale behavior. 8. Roll back if
    SLO or dependency health degrades.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 122. Production YAML: Resource Requests

``` yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 123. Production YAML: Deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 25%
  template:
    spec:
      containers:
        - name: catalogue
          image: <immutable-image>
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
          startupProbe:
            httpGet:
              path: /health
              port: 8080
            failureThreshold: 30
            periodSeconds: 5
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 124. Production YAML: PDB

``` yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: catalogue
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: catalogue
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 125. Production YAML: Topology Spread

``` yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: catalogue
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 126. Production YAML: ResourceQuota

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: catalogue-quota
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    pods: "50"
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 127. HPA Alerting

Alert on sustained maxReplicas, sustained Pending pods, missing metrics,
HPA unable-to-fetch-metrics conditions, repeated scale events, and
prolonged mismatch between desired and ready replicas. Pair these with
service-level alerts so autoscaling is treated as a means, not the
objective.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 128. Autoscaling SLOs

Define measurable goals such as time from demand increase to sufficient
ready capacity, maximum acceptable latency during scale-out, and
recovery time after a node failure. These are more meaningful than
simply saying that HPA is enabled.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 129. Scaling Budget

Define how much temporary overcapacity is acceptable during rollout and
scale-up. maxSurge, HPA maxReplicas, node pool limits, and budget
constraints should be evaluated together.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 130. Security of Autoscaling

Autoscaling controllers have powerful permissions. Protect service
accounts, IAM roles, admission policies, and configuration repositories.
An attacker who can manipulate HPA, NodePool, or provisioning policies
can cause cost spikes or capacity exhaustion.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 131. Cost Attack Scenario

A malicious or broken metric can cause replicas and nodes to scale
aggressively. Limit maxReplicas, node provisioning limits, AWS quotas,
and IAM capabilities. Monitor unusual cost and resource growth.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 132. Observability Architecture

``` text
Application Metrics ---> Prometheus
          |                 |
          |          Prometheus Adapter
          |                 |
          +-----------> HPA
                            |
                       Deployment
                            |
                     Pending Pods
                            |
                 Karpenter / CA
                            |
                      AWS EC2 Nodes
                            |
                       CloudWatch
```

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 133. Golden Signals and Autoscaling

Monitor latency, traffic, errors, and saturation. Traffic explains
demand, saturation explains capacity pressure, latency shows user
impact, and errors reveal whether scaling is actually helping.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 134. Scaling Dashboard

Create panels for request rate, p95/p99 latency, error rate,
current/desired replicas, HPA metric values, Pending pods, node count,
node CPU/memory, provisioning latency, ALB target health, and downstream
database saturation.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 135. Capacity Forecasting

Use historical traffic and resource usage to forecast normal and peak
demand. Compare forecast demand with tested per-replica capacity and
node capacity. This prevents HPA from becoming a substitute for capacity
planning.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 136. Production Architecture Decision

For a typical stateless EKS application, use HPA for pod-level
elasticity, Karpenter or Cluster Autoscaler for node elasticity,
accurate requests, readiness/startup probes, topology-aware placement,
PDBs, and observability. Use VPA mainly for rightsizing and KEDA for
event-driven workloads.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 137. Final Autoscaling Checklist

-   [ ] HPA uses autoscaling/v2
-   [ ] minReplicas supports HA
-   [ ] maxReplicas protects dependencies
-   [ ] Resource requests are realistic
-   [ ] Metrics provider is monitored
-   [ ] Custom metrics are validated
-   [ ] Startup/readiness are correct
-   [ ] Node autoscaling is configured
-   [ ] AWS quotas are verified
-   [ ] Replicas span failure domains
-   [ ] PDB is compatible with operations
-   [ ] Scale-up is tested
-   [ ] Scale-down is tested
-   [ ] Dependency saturation is monitored
-   [ ] MaxReplicas alert exists
-   [ ] Pending-pod alert exists
-   [ ] DR scaling is tested
-   [ ] GitOps ownership is clear

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 138. Senior Interview: Explain EKS Autoscaling

I use layered autoscaling. HPA adjusts pod replicas from resource or
application metrics, while Karpenter or Cluster Autoscaler provides
nodes when scheduling capacity is insufficient. Accurate requests,
readiness, topology, PDBs, downstream protection, and observability make
the loop production-safe.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 139. Senior Interview: Why Is HPA Not Enough?

HPA can request more pods, but those pods still need schedulable nodes.
Without node autoscaling or spare capacity, replicas can remain Pending.
I therefore design pod and node elasticity together.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 140. Senior Interview: HPA MaxReplicas

maxReplicas is both a capacity limit and a safety boundary. I set it
from load tests, application capacity, dependency limits, database
connections, and cost constraints rather than choosing an arbitrary high
value.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 141. Senior Interview: HPA CPU vs Custom Metrics

CPU works when CPU correlates with workload demand. For queue workers I
prefer queue depth; for HTTP services RPS can be better; for
event-driven workloads KEDA may be appropriate. The metric must
represent work and react predictably.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 142. Senior Interview: HPA and VPA

I avoid independently changing the same resource dimension with HPA and
VPA because VPA changes requests and can alter utilization-based HPA
behavior. I commonly use VPA recommendations for rightsizing and HPA for
horizontal scaling.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 143. Senior Interview: HPA Reaches Maximum

I check whether the load is legitimate and whether the dependency chain
can handle more capacity. I inspect ALB traffic, latency, errors, CPU,
queue depth, database saturation, and connection counts. I do not simply
increase maxReplicas.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 144. Senior Interview: Pods Pending After HPA

I inspect scheduling events, node capacity, Karpenter or Cluster
Autoscaler, taints, affinity, topology constraints, quotas,
architecture, and AWS capacity. Pending pods mean the pod scaling
request was made but infrastructure could not satisfy it.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 145. Senior Interview: Avoiding Autoscaling Outage

I use conservative scale-down, fast but bounded scale-up, accurate
requests, dependency protection, maxReplicas, connection-pool limits,
readiness gates, and load testing. Autoscaling should never be allowed
to amplify a downstream failure.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 146. Senior Interview: Production Example

For a high-traffic stateless API, I might maintain three replicas across
Availability Zones, scale between three and twenty based on CPU plus an
application metric, provision nodes dynamically, protect disruption with
a PDB, and monitor request latency, target health, Pending pods, and
dependency saturation. Exact values come from load testing rather than a
universal template.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?

## 147. Final Rules

1.  Scale on meaningful demand. 2. Scale pods and nodes as one
    system. 3. Keep accurate resource requests. 4. Make scale-up faster
    than scale-down. 5. Protect downstream dependencies. 6. Cap maximum
    capacity. 7. Distribute replicas across failure domains. 8. Monitor
    the complete time-to-capacity path. 9. Test failure and peak-load
    scenarios. 10. Keep scaling configuration under controlled GitOps
    ownership.

### Production validation

-   Verify the metric represents real workload demand.
-   Verify the target workload can scale safely.
-   Verify Kubernetes has or can obtain sufficient node capacity.
-   Verify dependencies can handle the scaled workload.
-   Verify scale-up, scale-down, rollout, and failure behavior.

### Operator questions

1.  What signal causes scaling?
2.  How quickly can new capacity become ready?
3.  What limits scaling?
4.  What dependency could fail when replicas increase?
5.  How do we observe and recover the scaling system?
