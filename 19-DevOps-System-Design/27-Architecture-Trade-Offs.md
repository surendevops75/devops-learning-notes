# Architecture-Trade-Offs

## How to Use These Notes

For every architecture discussion, explicitly state the requirement, compare at least two viable designs, identify the dominant trade-off, quantify where possible, and define a validation or rollback strategy.

---

## 1. Purpose

Architecture trade-offs are the deliberate choices made when no design can maximize every desirable property at the same time.

Typical competing goals are:

```text
availability
consistency
latency
cost
security
simplicity
scalability
operability
developer velocity
```

Senior DevOps engineers must explain not only what architecture they choose,
but why they choose it, what they sacrifice, and how they control the risk.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 2. The Trade-Off Mindset

A production architecture is a set of constraints and decisions.

Do not ask only:

```text
What is the best technology?
```

Ask:

```text
What problem are we solving?
What constraints exist?
What failure must we tolerate?
What scale is required?
What latency is required?
What recovery target exists?
What is the operational burden?
What is the cost?
What happens if our assumption is wrong?
```


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 3. Decision Framework

Use this sequence:

```text
Business requirements
        |
Non-functional requirements
        |
Constraints
        |
Candidate architectures
        |
Trade-off analysis
        |
Risk controls
        |
Decision
        |
Validation
```

A strong design decision is traceable to requirements rather than personal
technology preference.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 4. Functional vs Non-Functional Requirements

Functional requirements describe what the system does.

Non-functional requirements describe how the system must behave.

Examples:

```text
Functional:
  process payment

Non-functional:
  99.99% availability
  P99 latency < target
  RPO near zero
  RTO < target
  encrypted data
  defined cost envelope
```


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 5. Availability vs Cost

Increasing redundancy normally increases cost.

Example:

```text
single instance
    |
lower cost
    |
higher failure risk
```

versus:

```text
multi-AZ replicas
    |
higher cost
    |
better availability
```

Choose redundancy according to business impact.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 6. Availability vs Complexity

Every additional failover path adds:

```text
configuration
monitoring
testing
operational procedures
failure modes
```

Do not add multi-region architecture merely because it sounds highly
available.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 7. Availability vs Consistency

Strong consistency can simplify correctness but may increase latency or reduce
availability during partitions.

Eventual consistency can improve scalability and availability but requires
application-level reconciliation and user-visible consistency handling.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 8. Latency vs Consistency

Cross-region synchronous writes can improve durability and consistency but add
network round-trip latency.

A latency-sensitive application may prefer asynchronous replication with an
explicit RPO.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 9. Scalability vs Simplicity

A simple architecture can handle moderate scale with lower operational overhead.

Premature distribution creates:

```text
network calls
distributed failures
coordination
observability complexity
```

Scale only when requirements justify it.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 10. Horizontal vs Vertical Scaling

Vertical scaling:

```text
bigger machine
```

Advantages:

```text
simple
few moving parts
```

Disadvantages:

```text
hard upper limit
larger failure unit
potentially expensive
```

Horizontal scaling:

```text
more instances
```

Advantages:

```text
elasticity
failure distribution
```

Disadvantages:

```text
statelessness requirements
coordination
load balancing
```

Use both when appropriate.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 11. Stateless vs Stateful

Stateless services are generally easier to scale and replace.

Stateful systems require decisions about:

```text
replication
consistency
storage
backup
failover
recovery
```

Do not force every workload into a stateless pattern when state semantics are
the actual business requirement.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 12. Managed vs Self-Managed

Managed services reduce:

```text
patching
hardware operations
control-plane maintenance
```

Self-managed services can provide:

```text
greater control
customization
portability
```

But the organization inherits operational responsibility.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 13. AWS Managed Services

Prefer managed services when they materially reduce undifferentiated operational work
and satisfy requirements.

Evaluate:

```text
availability
feature fit
limits
pricing
lock-in
recovery
security
```

rather than choosing managed services automatically.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 14. Kubernetes vs EC2

Kubernetes provides:

```text
scheduling
service discovery
declarative deployment
autoscaling primitives
```

But adds platform complexity.

EC2-based deployment may be simpler for a small system.

Choose Kubernetes when its orchestration capabilities justify the platform
investment.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 15. Kubernetes vs ECS

EKS/Kubernetes provides a broad ecosystem and portability.

ECS can reduce Kubernetes-specific operational complexity.

Evaluate:

```text
team expertise
existing platform
workload requirements
ecosystem
operational cost
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 16. Containers vs Virtual Machines

Containers provide efficient application packaging and density.

VMs provide stronger isolation boundaries and can simplify legacy workloads.

Many production platforms use both.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 17. Serverless vs Containers

Serverless can reduce infrastructure management.

Containers provide more control over:

```text
runtime
networking
resource behavior
long-running processes
```

Serverless introduces its own considerations:

```text
cold starts
execution limits
platform coupling
observability
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 18. Build vs Buy

Build when:

```text
capability is strategic
requirements are unique
existing products cannot meet constraints
```

Buy when:

```text
capability is commodity
maintenance would be expensive
managed expertise has significant value
```

The real comparison is total ownership cost, not license price alone.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 19. Open Source vs Commercial

Open source may reduce licensing costs but still requires:

```text
operations
upgrades
security
support
expertise
```

Commercial products may reduce operational burden but introduce licensing and
vendor dependency.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 20. Vendor Lock-In

Lock-in is not automatically bad.

A managed service can provide significant value.

Evaluate:

```text
migration cost
data portability
API dependency
skills
contract terms
business value
```

Accept lock-in when its benefits outweigh migration risk.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 21. Portability vs Optimization

Portable designs may avoid provider-specific features but sacrifice useful cloud
capabilities.

Example:

```text
generic abstraction
    -> portability
AWS-native service
    -> optimization
```

Do not sacrifice major operational benefits solely for theoretical portability.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 22. Multi-Cloud

Multi-cloud can reduce provider concentration but increases:

```text
platform complexity
networking
IAM differences
observability
skills
deployment complexity
```

Use multi-cloud when driven by a real requirement such as regulatory, business,
or provider-risk constraints.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 23. Single Cloud vs Multi-Cloud

Single cloud:

```text
simpler
deeper integration
lower operational overhead
```

Multi-cloud:

```text
provider diversity
greater independence
higher complexity
```

A well-designed single-cloud architecture can be more resilient than a poorly
implemented multi-cloud architecture.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 24. Multi-AZ vs Multi-Region

Multi-AZ protects against many localized infrastructure failures.

Multi-region addresses larger regional failure scenarios.

Typical progression:

```text
single AZ
   ->
multi-AZ
   ->
regional DR
   ->
multi-region active/passive
   ->
multi-region active/active
```

Do not jump to the final stage without business justification.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 25. Active-Passive vs Active-Active

Active-passive:

```text
Primary -> traffic
Secondary -> standby
```

Advantages:

```text
simpler
lower cost
easier consistency
```

Disadvantages:

```text
failover time
standby capacity
```

Active-active:

```text
Primary A -> traffic
Primary B -> traffic
```

Advantages:

```text
regional resilience
resource utilization
```

Disadvantages:

```text
data consistency
routing
deployment
testing
cost
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 26. Warm vs Cold DR

Cold DR:

```text
restore infrastructure after disaster
```

Lower cost, higher recovery time.

Warm DR:

```text
partially running recovery environment
```

Higher cost, faster recovery.

Choose according to RTO.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 27. Backup vs Replication

Replication improves availability or recovery readiness.

Backup protects against:

```text
accidental deletion
corruption
historical recovery
```

Replication can replicate bad data.

Backups and replication solve different problems.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 28. RTO vs Cost

Lower RTO generally requires:

```text
pre-provisioned capacity
automation
replication
tested failover
```

Therefore:

```text
lower RTO -> higher investment
```

Set RTO from business requirements.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 29. RPO vs Cost

Near-zero RPO generally requires more frequent or synchronous replication.

Higher tolerated data loss can permit:

```text
asynchronous replication
periodic backups
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 30. Synchronous vs Asynchronous Replication

Synchronous:

```text
write
 |
remote acknowledgement
 |
success
```

Better consistency/durability, greater latency dependency.

Asynchronous:

```text
write
 |
local success
 |
replicate later
```

Lower latency, potential replication lag.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 31. Strong vs Eventual Consistency

Strong consistency simplifies reads but can reduce availability or increase
coordination cost.

Eventual consistency scales well but requires handling:

```text
stale reads
ordering
reconciliation
duplicate events
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 32. SQL vs NoSQL

SQL is often preferred when:

```text
transactions
relational integrity
complex queries
```

are important.

NoSQL can be useful for:

```text
specific access patterns
massive scale
flexible schemas
high write distribution
```

Choose based on data access requirements rather than popularity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 33. RDS vs Self-Managed Database

RDS reduces infrastructure operations.

Self-managed databases provide greater control.

Evaluate:

```text
extensions
version requirements
performance tuning
backup
HA
team expertise
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 34. Database per Service vs Shared Database

Database-per-service improves ownership and isolation.

Shared database can simplify:

```text
transactions
reporting
legacy integration
```

but creates coupling and larger blast radius.

Use service boundaries intentionally.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 35. Cache vs Database

Caching reduces database load and latency but introduces:

```text
staleness
invalidation
memory cost
failure behavior
```

A cache should have an intentional consistency model.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 36. Redis vs Application Cache

Distributed Redis provides shared caching and coordination capabilities.

Local application cache is faster and simpler but has per-instance state and
staleness challenges.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 37. SQS vs Kafka

Queue-oriented systems are useful for asynchronous task processing.

Kafka-style event streaming is useful when requirements include:

```text
durable event streams
replay
partitioned throughput
consumer groups
```

Choose based on messaging semantics, not raw throughput claims.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 38. Synchronous vs Asynchronous Architecture

Synchronous calls simplify immediate request/response workflows.

Asynchronous messaging improves decoupling and resilience but introduces:

```text
eventual consistency
retries
duplicate delivery
observability complexity
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 39. REST vs Event-Driven

REST is straightforward for request/response APIs.

Events are useful when multiple consumers need decoupled reactions.

A hybrid architecture is often appropriate.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 40. API Gateway vs Load Balancer

A load balancer primarily distributes network/application traffic.

An API gateway can provide:

```text
authentication
rate limiting
API policies
routing
transformation
```

Do not add gateway functionality that the workload does not require.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 41. ALB vs NLB

ALB is appropriate for HTTP-aware routing.

NLB is appropriate for high-performance Layer 4 traffic and use cases requiring
specific transport behavior.

Choose based on protocol and routing requirements.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 42. CDN vs Direct Origin

CDN improves:

```text
latency
origin protection
static delivery
edge caching
```

but adds cache invalidation and edge configuration complexity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 43. CloudFront vs S3 Direct Access

For public/static delivery, CDN plus object storage can reduce origin load and
improve global latency.

Direct object access may be simpler for internal or low-scale use cases.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 44. NAT Gateway vs VPC Endpoints

NAT provides outbound connectivity for private resources.

VPC endpoints can provide private connectivity to supported AWS services and
can reduce unnecessary NAT dependency and traffic.

Evaluate both architecture and cost.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 45. Centralized vs Distributed Networking

Centralized networking can improve governance.

Distributed networking can reduce dependency on a single central component.

Use hub-and-spoke or decentralized patterns according to organizational
requirements.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 46. Centralized vs Distributed Logging

Centralized logging simplifies search and correlation.

Distributed logging can improve isolation and reduce a single ingestion
failure domain.

A mature platform often centralizes storage while maintaining resilient
collection paths.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 47. Centralized vs Distributed Monitoring

Central monitoring provides unified visibility.

Independent monitoring can remain useful when the primary platform fails.

Critical services should have monitoring that does not disappear with the
same failure being observed.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 48. Prometheus vs Managed Metrics

Prometheus provides flexibility and Kubernetes-native monitoring.

Managed metrics can reduce operational burden.

Evaluate:

```text
retention
query requirements
federation
cost
operational ownership
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 49. ELK vs Managed Logging

Self-managed Elasticsearch/OpenSearch-style stacks offer control but require
substantial capacity and lifecycle management.

Managed logging reduces operations but may have pricing and feature constraints.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 50. OpenTelemetry vs Vendor Agent

OpenTelemetry can improve instrumentation portability.

Vendor agents may provide deeper provider-specific integrations.

Use OpenTelemetry where standardization and portability matter, while using
specialized integrations when they materially improve operations.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 51. Logs vs Metrics vs Traces

Metrics answer:

```text
Is something wrong?
```

Logs answer:

```text
What happened?
```

Traces answer:

```text
Where did the request spend time?
```

Use all three where operational complexity requires them.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 52. High Cardinality vs Detailed Observability

More labels can improve investigation but explode storage and query cost.

Use controlled dimensions and avoid unbounded identifiers.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 53. Sampling vs Full Tracing

100% tracing provides more evidence but can be expensive.

Sampling reduces cost.

Tail-based sampling can retain traces matching important conditions such as
errors or high latency.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 54. Security vs Usability

Strong controls can increase developer friction.

Poor controls create production risk.

Prefer:

```text
automated policy
short-lived access
self-service with guardrails
audited break-glass
```

over unrestricted permanent access.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 55. Least Privilege vs Operational Speed

Least privilege may initially slow debugging.

Use:

```text
JIT access
break-glass roles
audited escalation
predefined permissions
```

to preserve security without making incidents impossible to resolve.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 56. Secrets Manager vs Kubernetes Secrets

Kubernetes Secrets are integrated with workloads but require careful encryption,
RBAC and cluster security.

External secret systems can provide stronger centralized lifecycle management.

Evaluate operational integration and security requirements.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 57. IAM Roles vs Static Keys

Prefer short-lived role-based credentials.

Static keys create:

```text
rotation burden
leak risk
long-lived privilege
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 58. Security Scanning Depth vs Developer Velocity

More scanning can increase security confidence but slow pipelines.

Use risk-based gates:

```text
critical vulnerability -> block
accepted low-risk finding -> track
```

Avoid making every informational finding a deployment blocker.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 59. Shift Left vs Runtime Security

Shift-left controls catch issues before deployment.

Runtime controls catch behavior that static analysis cannot predict.

Use defense in depth rather than choosing one.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 60. Build Once vs Rebuild Per Environment

Build once:

```text
artifact
 |
scan
 |
promote
 |
deploy
```

improves artifact consistency.

Rebuilding per environment can introduce environment-specific changes but
increases reproducibility risk.

Prefer immutable promotion where practical.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 61. Immutable vs Mutable Infrastructure

Immutable:

```text
replace
```

Advantages:

```text
reproducibility
drift reduction
rollback
```

Mutable:

```text
modify existing
```

can be useful for legacy systems but increases drift risk.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 62. Golden Images vs Dynamic Configuration

Golden images reduce startup work and configuration variability.

Dynamic configuration improves flexibility.

A balanced model often uses immutable base images plus controlled runtime
configuration.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 63. GitOps vs Imperative Operations

GitOps provides:

```text
auditability
desired state
review
reconciliation
```

Imperative operations are useful for emergencies and diagnostics.

Emergency changes should be reconciled back into desired state.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 64. Automation vs Human Control

Automate repetitive, deterministic and reversible work.

Keep human approval for:

```text
destructive changes
large blast-radius changes
uncertain recovery
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 65. Auto-Remediation vs Safety

Automatic remediation can reduce MTTR.

Poor automation can multiply failures.

Use:

```text
preconditions
scope limits
cooldowns
validation
abort conditions
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 66. Autoscaling vs Predictability

Autoscaling improves elasticity.

But it can create:

```text
cost spikes
connection storms
cold capacity
scaling oscillation
```

Use bounded scaling and appropriate stabilization windows.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 67. HPA vs Karpenter

HPA changes workload replica count.

Karpenter-style node provisioning changes compute capacity.

They solve different layers:

```text
HPA -> pods
node autoscaling -> nodes
```

Use them together when appropriate.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 68. Spot vs On-Demand

Spot reduces cost but introduces interruption risk.

Use Spot for:

```text
fault-tolerant
interruptible
distributed
stateless
batch
```

workloads.

Keep critical capacity appropriately protected.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 69. Reserved Capacity vs Elasticity

Reserved/Savings-style commitments can reduce cost for predictable usage.

On-demand/elastic capacity preserves flexibility.

Use a blended capacity strategy.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 70. Overprovisioning vs Efficiency

Extra capacity improves resilience and reduces saturation risk.

Too much idle capacity wastes money.

Capacity planning should consider:

```text
normal load
peak load
failure load
recovery load
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 71. Cost vs Availability

Higher availability often requires:

```text
more replicas
more AZs
DR capacity
replication
monitoring
```

Cost optimization must not accidentally remove the redundancy required by
business-critical SLOs.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 72. Cost vs Performance

Faster infrastructure can reduce latency but increase cost.

Optimize based on:

```text
customer value
SLO
unit economics
```

not infrastructure prestige.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 73. Network Cost vs Resilience

Cross-AZ and cross-region traffic can improve architecture resilience but increase
cost.

Choose traffic topology deliberately.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 74. Data Transfer vs Centralization

Centralizing workloads may simplify operations but create cross-region or
cross-account transfer costs.

Place data and compute close together when latency and transfer cost matter.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 75. Reliability vs Developer Velocity

Strict controls can slow delivery.

Use automated guardrails and progressive delivery to achieve both:

```text
fast development
+
safe production
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 76. Standardization vs Team Autonomy

Central platform standards improve reliability.

Too much standardization blocks legitimate workload differences.

Define:

```text
mandatory guardrails
recommended defaults
optional extensions
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 77. Platform Abstraction vs Direct Cloud APIs

Abstraction can simplify developer experience.

Too much abstraction hides important cloud behavior.

Expose escape hatches for advanced users while maintaining safe defaults.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 78. Monolith vs Microservices

Monolith advantages:

```text
simple deployment
local calls
simple transactions
```

Microservice advantages:

```text
independent scaling
team ownership
failure isolation
deployment independence
```

Microservices also introduce distributed-system complexity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 79. Modular Monolith vs Microservices

A modular monolith can provide strong internal boundaries without immediate
network distribution.

Use it when team or scale requirements do not justify distributed services.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 80. Service Count vs Operational Burden

Every service adds:

```text
deployment
monitoring
on-call
security
dependencies
```

Do not create services solely to make architecture diagrams look modern.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 81. Synchronous Transactions vs Saga

Distributed transactions are difficult.

Saga-style workflows use:

```text
local transactions
events
compensation
```

This improves distributed scalability but makes business workflow handling
more complex.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 82. Exactly Once vs At Least Once

Exactly-once processing is difficult to guarantee end to end.

At-least-once delivery plus:

```text
idempotency
deduplication
```

is often a practical production strategy.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 83. Idempotency vs Complexity

Idempotency adds implementation complexity but dramatically improves safe retry
behavior.

Use idempotency for operations where duplicate execution is dangerous.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 84. Queue Durability vs Latency

More durable messaging can introduce additional persistence or replication
overhead.

Choose based on data-loss tolerance.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 85. Event Retention vs Cost

Long event retention improves replay and audit capabilities but increases storage
cost.

Define retention by business value.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 86. Schema Flexibility vs Governance

Flexible schemas speed evolution but can create integration inconsistency.

Use schema contracts for critical event and API boundaries.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 87. API Gateway Policies vs Service Autonomy

Central gateway policies simplify common controls.

Too much central policy creates a bottleneck.

Keep universal policies centralized and business-specific behavior with
services where appropriate.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 88. Centralized Identity vs Local Authorization

Central identity improves consistency.

Service-level authorization is still required because identity alone does not
define every resource permission.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 89. Network Security vs Connectivity

Overly permissive networks increase risk.

Overly restrictive networks create operational failures.

Use explicit service-to-service requirements and automated policy testing.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 90. Zero Trust vs Network Perimeter

Traditional perimeter controls are insufficient for modern distributed platforms.

Identity-aware authorization and workload-level controls reduce lateral
movement.

Network boundaries still remain useful defense layers.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 91. Deployment Frequency vs Change Risk

Frequent small changes can reduce individual change size.

Large infrequent releases increase change blast radius.

Use small, observable deployments.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 92. Feature Flags vs Configuration Complexity

Feature flags reduce deployment risk but create lifecycle debt.

Every flag should have:

```text
owner
purpose
expiry
default
rollback
```

Remove obsolete flags.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 93. Canary Size vs Detection Confidence

A very small canary minimizes impact but may produce insufficient statistical
evidence.

A larger canary provides more signal but increases exposure.

Choose size based on traffic and risk.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 94. Blue-Green Cost vs Rollback Speed

Blue-green requires duplicate capacity but enables fast traffic switching.

Use it where rollback speed justifies the additional cost.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 95. Rolling Deployment vs Isolation

Rolling deployments reduce duplicate capacity requirements.

However, old and new versions coexist and can interact.

Ensure backward compatibility.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 96. Database Migration Safety vs Speed

Fast destructive migrations are risky.

Expand/contract migration takes longer but supports:

```text
backward compatibility
progressive rollout
rollback
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 97. Read Replicas vs Strong Read Semantics

Read replicas improve scaling but may return stale data.

Do not route consistency-sensitive reads to replicas without understanding
replication behavior.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 98. CQRS vs Simplicity

CQRS can optimize read/write models independently.

It introduces:

```text
multiple models
synchronization
eventual consistency
```

Use it when access patterns justify the complexity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 99. Search Engine vs Database Queries

Search systems are optimized for search and indexing.

Do not use a search engine as a transactional database without understanding
its consistency and durability semantics.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 100. Object Storage vs Block Storage

Object storage is ideal for:

```text
artifacts
backups
static objects
data lakes
```

Block storage is appropriate for:

```text
filesystem workloads
databases
low-level disk access
```

Choose according to access semantics.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 101. EBS vs EFS

EBS provides block storage with workload-specific performance characteristics.

EFS provides shared file access.

Shared file semantics can simplify some workloads but may have different
latency and cost characteristics.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 102. S3 Lifecycle vs Immediate Storage

Lifecycle policies reduce storage cost by moving older data to appropriate classes.

Trade storage savings against retrieval latency and retrieval charges.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 103. Compression vs CPU

Compression reduces:

```text
storage
network transfer
```

but increases CPU.

Choose based on workload bottlenecks.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 104. Caching vs Freshness

Caching improves latency and reduces backend load.

Trade-offs include:

```text
TTL
invalidation
stale data
memory cost
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 105. CDN Cache vs Dynamic Data

Aggressive caching is excellent for immutable content.

Dynamic personalized content requires careful cache-key and invalidation design.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 106. Security Controls vs Pipeline Time

Security gates should prioritize high-risk findings.

Use parallel scanning where possible to reduce pipeline latency without
removing controls.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 107. Dependency Pinning vs Updates

Pinning improves reproducibility.

But never updating dependencies creates security and maintenance risk.

Use controlled automated update workflows.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 108. Release Cadence vs Stability

Frequent releases reduce change size.

Too much release frequency without adequate testing can increase operational
noise.

Optimize for safe delivery rather than raw deployment count.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 109. Environment Parity vs Cost

Production-like staging improves confidence.

A full production replica may be expensive.

Use representative scale and behavior where exact duplication is unnecessary.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 110. Ephemeral Environments vs Resource Cost

Ephemeral environments improve developer testing and isolation.

They can create significant idle resource cost.

Use automatic expiration and quotas.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 111. Test Depth vs Pipeline Speed

More tests increase confidence but lengthen pipelines.

Use:

```text
fast unit tests
parallel integration tests
targeted expensive tests
pre-production validation
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 112. Shift Testing Left vs Production Validation

Pre-production testing catches defects early.

Production canary testing catches real environment interactions.

Use both.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 113. Synthetic Monitoring vs Real User Monitoring

Synthetic tests provide controlled signals.

Real-user telemetry captures actual customer behavior.

Use both when customer experience is critical.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 114. SLO Strictness vs Development Velocity

Aggressive SLOs require more reliability investment.

Set SLOs based on customer expectations rather than arbitrary perfection.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 115. Error Budget vs Feature Delivery

When the error budget is exhausted, prioritize reliability work.

This creates a measurable balance between velocity and reliability.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 116. HA vs DR

HA keeps a service available through certain failures.

DR restores service after larger failures.

They are complementary, not interchangeable.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 117. Backup Frequency vs Cost

Frequent backups reduce potential data loss but increase:

```text
storage
I/O
management
```

Choose frequency from RPO.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 118. Immutable Backup vs Operational Flexibility

Immutable backups improve ransomware and accidental deletion protection.

They can make operational cleanup less flexible.

Use appropriate retention and approval controls.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 119. Security Isolation vs Shared Services

Shared services reduce duplication.

Strong isolation reduces blast radius.

Use shared services for low-risk common functions and isolate high-risk
boundaries where necessary.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 120. Account Count vs Governance

More AWS accounts can improve isolation.

Too many accounts increase:

```text
governance
networking
billing
access
```

complexity.

Use account boundaries intentionally.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 121. Cluster Count vs Isolation

Multiple clusters improve isolation for certain failure and security boundaries.

They increase:

```text
upgrades
observability
platform operations
```

cost.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 122. Namespace vs Cluster Isolation

Namespaces are lightweight logical boundaries.

Clusters provide stronger operational isolation.

Use clusters when namespace-level isolation is insufficient.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 123. Node Pool Isolation vs Utilization

Dedicated node pools isolate workloads.

But they can leave capacity idle.

Use dedicated pools for meaningful requirements, not every application.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 124. Resource Limits vs Performance

Limits prevent noisy-neighbor behavior.

Poor limits can cause throttling or OOM kills.

Set limits from measured workload behavior.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 125. Requests vs Bin Packing

Higher requests improve scheduling guarantees but reduce packing efficiency.

Lower requests improve utilization but increase contention risk.

Use observed resource usage and appropriate headroom.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 126. CPU Limit vs Predictability

CPU limits can protect shared capacity but may cause throttling.

Evaluate whether the workload needs a hard CPU ceiling.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 127. Memory Limit vs Stability

Memory limits protect nodes from runaway workloads.

Too-low limits cause OOM kills.

Use profiling and production telemetry.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 128. HPA Responsiveness vs Stability

Fast scaling reacts quickly but can oscillate.

Stabilization windows and sensible thresholds reduce scaling thrash.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 129. Autoscaling Metric Choice

CPU may be a poor proxy for application load.

Better signals can include:

```text
requests per second
queue depth
latency
business load
```

Choose metrics that represent capacity pressure.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 130. Karpenter Flexibility vs Governance

Dynamic node provisioning improves capacity flexibility.

Use constraints to control:

```text
instance families
AZs
capacity type
architecture
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 131. Spot Diversification vs Complexity

Diversifying Spot capacity reduces correlated interruption risk.

But it increases scheduling and capacity planning complexity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 132. ARM vs x86

ARM can provide attractive price/performance for compatible workloads.

Migration may require:

```text
multi-architecture images
dependency compatibility
testing
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 133. Graviton vs x86

Evaluate real application performance, library support and operational effort.

Do not choose architecture solely from instance pricing.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 134. Container Density vs Isolation

Higher density improves utilization.

Lower density can improve:

```text
failure isolation
performance predictability
security boundaries
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 135. Shared Node vs Dedicated Node

Shared nodes improve utilization.

Dedicated nodes are useful for:

```text
high-risk workloads
special hardware
strict performance
compliance
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 136. Security Group Granularity

Fine-grained security groups improve isolation.

Too many rules and groups increase management complexity.

Use automation and clear ownership.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 137. NetworkPolicy Granularity

Fine-grained policies reduce lateral movement.

Poorly maintained policies can break service communication.

Test policies before broad enforcement.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 138. Egress Control vs Developer Freedom

Restricting outbound traffic reduces exfiltration and dependency risk.

Allow approved destinations through controlled mechanisms.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 139. WAF Strictness vs False Positives

Aggressive WAF rules can block legitimate traffic.

Tune rules using observed traffic and staged enforcement.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 140. DDoS Protection vs Cost

Advanced edge protections may increase cost.

For internet-facing critical systems, compare protection cost with expected
business impact.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 141. Encryption vs Performance

Modern encryption overhead is often manageable, but encryption can still add CPU,
latency or operational complexity in specific workloads.

Protect sensitive data by default and measure performance-sensitive paths.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 142. KMS Centralization vs Independence

Centralized key management simplifies governance.

Excessive central dependency can increase failure coupling.

Design recovery and access procedures carefully.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 143. Audit Logging vs Storage Cost

Detailed audit data is valuable for security and compliance.

Use retention and tiering to control cost.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 144. Compliance vs Architecture Simplicity

Compliance requirements may require:

```text
segmentation
audit
retention
encryption
access controls
```

Do not treat compliance as an afterthought.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 145. Human Approval vs Automation

Human approval is valuable for high-risk changes.

It becomes a bottleneck when applied to every low-risk action.

Use risk-based approvals.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 146. On-Call Coverage vs Cost

More coverage reduces response gaps but increases staffing cost.

Automate detection and escalation before adding unnecessary human layers.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 147. Runbook Detail vs Maintenance

Detailed runbooks improve incident response.

Outdated runbooks are dangerous.

Automate validation and assign ownership for maintenance.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 148. Central Platform vs Embedded Teams

A central platform can standardize:

```text
security
observability
deployment
networking
```

Teams should still own application behavior and service reliability.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 149. Platform Team vs Product Team

Platform teams provide reusable capabilities.

Product teams remain accountable for their service's business behavior,
performance and operational health.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 150. Golden Path vs Escape Hatch

Golden paths reduce cognitive load.

Escape hatches allow legitimate exceptions.

Every escape hatch should have appropriate ownership and security controls.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 151. Architecture Documentation vs Agility

Documentation improves shared understanding.

Do not document every trivial implementation detail.

Focus on:

```text
decisions
dependencies
failure modes
operational procedures
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 152. ADRs

Architecture Decision Records should capture:

```text
context
decision
alternatives
trade-offs
consequences
```

This prevents repeated architectural debates.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 153. Proof of Concept vs Production Readiness

A POC proves technical feasibility.

Production readiness requires:

```text
security
HA
observability
backup
operations
cost
failure handling
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 154. Benchmark vs Real Workload

Synthetic benchmarks can mislead.

Validate using production-like workload patterns and business metrics.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 155. Scale Testing vs Cost

Load tests consume resources.

Run tests at the level required to validate architecture and capacity rather
than maximizing test size without a question to answer.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 156. Chaos Testing vs Operational Risk

Chaos testing validates resilience but can cause real impact.

Use:

```text
small scope
abort conditions
observability
approved windows
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 157. Failure Testing vs Availability

Planned failure exercises temporarily reduce availability.

The objective is to increase confidence in recovery.

Test progressively.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 158. DR Testing vs Business Disruption

DR tests can affect real traffic.

Use controlled failover or isolated simulations where appropriate, and verify
that recovery assumptions are real.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 159. Single Region vs Multi Region

Single-region multi-AZ is often sufficient for many applications.

Multi-region is appropriate when regional outage impact exceeds its operational
cost and complexity.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 160. Provider Independence vs Deep Integration

Deep cloud integration can produce strong operational outcomes.

Provider independence can reduce strategic concentration.

Choose based on business risk, not ideology.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 161. Architecture Review Criteria

Evaluate each candidate using:

```text
availability
latency
scalability
security
operability
cost
complexity
portability
recovery
blast radius
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 162. Weighted Decision Matrix

Example:

| Criterion | Weight | Option A | Option B |
|---|---:|---:|---:|
| Availability | 25 | 4 | 5 |
| Cost | 20 | 5 | 3 |
| Complexity | 15 | 5 | 2 |
| Scalability | 20 | 3 | 5 |
| Security | 10 | 4 | 4 |
| Recovery | 10 | 3 | 5 |

Multiply score by weight and compare totals.

The numbers do not replace engineering judgment; they make assumptions visible.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 163. Reversibility

Prefer decisions that can be reversed cheaply when uncertainty is high.

Irreversible decisions require stronger evidence.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 164. Two-Way Door Decisions

A reversible decision can be made quickly with reasonable evidence.

An irreversible decision deserves deeper review.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 165. One-Way Door Decisions

Examples:

```text
data model migration
irreversible vendor dependency
large-scale network redesign
```

Use stronger validation before committing.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 166. Risk Matrix

Classify decisions by:

```text
probability
impact
detectability
reversibility
```

High-impact, low-reversibility decisions deserve the most review.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 167. Technical Debt Trade-Off

Technical debt is acceptable when intentional and managed.

Bad debt is:

```text
unknown
unowned
unmeasured
```

Document why a shortcut was taken and when it should be revisited.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 168. Complexity Budget

Every distributed component consumes an operational complexity budget.

Ask:

```text
Does this component provide enough business value to justify its operational
cost?
```


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 169. Operational Load

Architecture should account for:

```text
on-call
patching
upgrades
incidents
capacity
security
observability
```

A theoretically elegant design can be operationally poor.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 170. Cognitive Load

Prefer architectures that engineers can understand during an incident.

Reduce unnecessary technologies and hidden dependencies.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 171. Standardization

Standardize where repetition creates value:

```text
CI/CD
observability
security
networking
identity
deployment
```

Allow variation where workload requirements truly differ.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 172. Technology Sprawl

Technology sprawl increases:

```text
skills required
patching
monitoring
incident complexity
```

Prefer a small set of well-supported platform technologies.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 173. Team Topology

Architecture should align with team ownership.

If a service requires constant coordination between five teams, the boundary
may be poorly designed.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 174. Conway's Law

System structure tends to reflect communication structure.

Use team boundaries intentionally when designing service ownership.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 175. Ownership Trade-Off

Every production component should have:

```text
owner
on-call
documentation
SLO
```

Unowned infrastructure becomes operational debt.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 176. Reliability Ownership

Platform teams can provide reliability capabilities, but application teams should
remain responsible for service-specific behavior and SLOs.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 177. Cost Ownership

Assign cost to teams or products through:

```text
tags
accounts
namespaces
chargeback
showback
```

Visibility improves optimization decisions.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 178. Unit Economics

Measure cost per business unit such as:

```text
cost per order
cost per active user
cost per API request
cost per transaction
```

This is more useful than total infrastructure cost alone.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 179. Performance Budget

Define explicit budgets for:

```text
latency
CPU
memory
network
database
```

Optimize the bottleneck that affects customer outcomes.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 180. Availability Budget

Use SLOs and error budgets to decide how much unreliability is acceptable.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 181. Latency Budget

Break request latency into:

```text
edge
service
database
dependency
serialization
network
```

Then optimize the dominant contributor.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 182. Capacity Headroom

Capacity should support:

```text
normal peak
failure of one domain
deployment overlap
traffic surge
```

Do not plan only for average utilization.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 183. Scale-Out Threshold

Scale before resource exhaustion.

Use leading indicators where possible:

```text
queue growth
latency
concurrency
```

instead of waiting for CPU saturation.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 184. Queue Depth vs CPU

For asynchronous workers, queue depth may be a better scaling signal than CPU.

Use workload-specific signals.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 185. Database Scaling

Scale database resources only after identifying the bottleneck:

```text
CPU
IOPS
connections
locks
queries
storage
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 186. Read Scaling

Read replicas can scale reads but do not solve write bottlenecks.

Separate read and write scaling decisions.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 187. Sharding

Sharding can scale data horizontally but adds:

```text
routing
rebalancing
cross-shard queries
operational complexity
```

Use only when simpler scaling options are insufficient.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 188. Partitioning

Partitioning can improve manageability and query performance without the full
operational complexity of distributed sharding.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 189. Search Index Trade-Off

Denormalized search indexes improve query performance but require synchronization
and reindexing strategies.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 190. Eventual Consistency User Experience

If data may be stale, design the UI/API to communicate state appropriately.

Do not hide consistency limitations behind misleading interfaces.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 191. Resilience vs User Experience

Graceful degradation should preserve the most valuable user journey.

Example:

```text
recommendations unavailable
checkout still works
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 192. Feature Priority During Failure

Classify features:

```text
critical
important
optional
```

and shed lower-priority functionality first during overload.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 193. Dependency Criticality

Classify dependencies by:

```text
critical
degraded
optional
```

and design fallback behavior accordingly.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 194. Retry vs Fail Fast

Retry transient failures.

Fail fast when retrying would amplify an outage or when the operation cannot
succeed within the user's latency budget.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 195. Retry Budget

Treat retries as additional traffic.

Bound retry volume so recovery attempts do not overwhelm an already unhealthy
dependency.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 196. Timeout Budget

Nested services should have timeouts compatible with the caller's overall latency
budget.

Do not allow downstream timeouts to exceed the request deadline.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 197. Circuit Breaker Trade-Off

Circuit breakers protect dependencies but can hide temporary recovery if thresholds
are poorly configured.

Monitor open/close behavior and tune using real traffic.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 198. Bulkhead Trade-Off

Bulkheads reduce cross-workload failure but may leave reserved capacity unused.

Use them for critical isolation boundaries.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 199. Graceful Degradation

Design degraded modes before incidents.

Examples:

```text
cached response
read-only mode
queue request
disable optional feature
```




### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


## 200. Architecture Decision Summary

A senior DevOps architecture decision should conclude with:

```text
Chosen architecture
Why it satisfies requirements
What trade-offs it accepts
What risks remain
How risks are mitigated
How success is measured
When the decision should be revisited
```

Never present architecture as universally correct.

The correct production architecture is the one that best satisfies the current
business, reliability, security, scale, cost and operational constraints.


### Production Decision Checklist

```text
[ ] requirement identified
[ ] scale identified
[ ] SLO identified
[ ] failure modes identified
[ ] security impact considered
[ ] cost impact considered
[ ] operational burden considered
[ ] blast radius considered
[ ] recovery strategy defined
[ ] rollback/reversibility considered
[ ] measurable validation defined
```


### Senior Interview Angle

```text
I would not choose this architecture based on technology preference alone.
I would start with business and non-functional requirements, compare viable
options, identify the dominant trade-off, quantify important consequences,
and then choose the simplest design that meets the required SLO and recovery
objectives. I would document the decision and define conditions that would
trigger a future redesign.
```


# PART II — TRADE-OFF DECISION TABLES

## 201. Quick Reference

| Decision | Prefer A When | Prefer B When | Main Risk |
|---|---|---|---|
| Multi-AZ / Multi-region | regional failure is not in scope | regional outage is unacceptable | cost/complexity |
| Active-passive / Active-active | simplicity and consistency dominate | regional continuity dominates | data consistency |
| Managed / self-managed | operations should be minimized | deep control is required | lock-in/operations |
| Kubernetes / VM | many services and orchestration needed | workload is simple/legacy | platform complexity |
| SQL / NoSQL | transactions and relational queries dominate | access patterns require distributed scale | wrong data model |
| Sync / async | immediate consistency is required | decoupling and scale dominate | stale state |
| REST / events | direct request/response | multiple decoupled consumers | event complexity |
| Cache / direct DB | latency/backend load matters | freshness and simplicity dominate | invalidation |
| Canary / blue-green | gradual exposure is valuable | instant switching is valuable | capacity |
| Rolling / replacement | capacity is constrained | isolation is more important | version coexistence |
| Spot / on-demand | workload is interruptible | workload is critical | interruptions |
| Central / distributed | governance dominates | isolation dominates | central bottleneck |
| Build / buy | capability is strategic | capability is commodity | ownership cost |
| Multi-cloud / single cloud | provider concentration is unacceptable | simplicity dominates | operational complexity |

## 202. Decision Questions

Before finalizing a design:

```text
1. What business outcome are we protecting?
2. What is the required availability?
3. What is the latency target?
4. What scale is expected?
5. What is the acceptable RTO?
6. What is the acceptable RPO?
7. Which failures must be tolerated?
8. What is the maximum blast radius?
9. What data consistency is required?
10. What security boundary is required?
11. What is the budget?
12. What skills are available?
13. What operational burden is acceptable?
14. What is the rollback strategy?
15. What is the migration strategy?
16. What is the testing strategy?
17. What metrics prove the design works?
18. Which assumption is most likely to be wrong?
19. Which decision is difficult to reverse?
20. When should the architecture be revisited?
```

# PART III — REAL PRODUCTION SCENARIOS

## 203. Global E-Commerce Platform

Requirements:

```text
global users
high checkout availability
regional resilience
strict payment correctness
```

Recommended reasoning:

```text
multi-AZ application
+
regional DR
+
strong consistency for payment state
+
asynchronous noncritical workflows
+
progressive delivery
```

Do not force every subsystem into active-active strong consistency.

## 204. Internal Developer Platform

Requirements:

```text
hundreds of services
self-service deployment
security guardrails
low developer friction
```

Trade-off:

```text
central platform standards
vs
team autonomy
```

Use golden paths, policy-as-code and controlled escape hatches.

## 205. Financial Transaction System

Prioritize:

```text
correctness
auditability
idempotency
data integrity
```

over theoretical maximum throughput.

Use asynchronous processing for noncritical side effects while keeping the
transactional core strongly controlled.

## 206. Media Platform

Prioritize:

```text
high read throughput
global delivery
cacheability
cost efficiency
```

CDN/object storage can carry much of the delivery workload while origin
services remain protected.

## 207. Batch Processing Platform

Prioritize:

```text
cost
throughput
fault tolerance
```

Spot capacity may be appropriate if jobs are checkpointed and retryable.

## 208. Enterprise API Platform

Use:

```text
gateway
authentication
rate limiting
observability
service-level authorization
```

but avoid turning the gateway into a universal business-logic bottleneck.

# PART IV — 250 GOLDEN RULES

## 209. Rules 1–50

```text
1. Start with requirements, not technology.
2. Separate functional from non-functional requirements.
3. State the dominant trade-off explicitly.
4. Prefer the simplest architecture that meets requirements.
5. Do not optimize for hypothetical scale.
6. Do not optimize only for current scale.
7. Define availability targets.
8. Define latency targets.
9. Define RTO.
10. Define RPO.
11. Define security boundaries.
12. Define cost constraints.
13. Define operational ownership.
14. Define expected growth.
15. Identify critical dependencies.
16. Identify shared failure domains.
17. Identify common-mode failures.
18. Identify irreversible decisions.
19. Prefer reversible decisions under uncertainty.
20. Quantify important trade-offs.
21. Compare at least two viable designs.
22. Document rejected alternatives.
23. Record architectural decisions.
24. Revisit decisions when assumptions change.
25. Treat complexity as a real cost.
26. Treat operational effort as a real cost.
27. Treat developer friction as a real cost.
28. Treat security risk as a real cost.
29. Treat outage impact as a real cost.
30. Treat vendor lock-in as a risk, not an automatic failure.
31. Managed services can be strategically valuable.
32. Self-managed services require ownership.
33. Portability is not free.
34. Multi-cloud is not automatically resilient.
35. Multi-region is not automatically required.
36. Multi-AZ is not the same as DR.
37. Backups are not the same as replication.
38. Replication is not the same as backup.
39. Read replicas are not automatically HA.
40. More replicas do not guarantee independence.
41. More services do not guarantee scalability.
42. More automation does not guarantee safety.
43. More monitoring does not guarantee observability.
44. More security rules do not guarantee security.
45. More capacity does not guarantee resilience.
46. More abstraction does not guarantee simplicity.
47. More standardization does not guarantee productivity.
48. More tests do not guarantee correctness.
49. More approvals do not guarantee safe change.
50. Architecture quality is requirement-dependent.
```

## 210. Rules 51–100

```text
51. Use multi-AZ for appropriate critical workloads.
52. Use multi-region only when justified.
53. Match redundancy to business impact.
54. Match DR investment to RTO/RPO.
55. Match consistency to business correctness.
56. Match latency to customer expectations.
57. Match scaling strategy to workload shape.
58. Prefer horizontal scaling for suitable stateless services.
59. Use vertical scaling when it is simpler and sufficient.
60. Design stateful systems deliberately.
61. Keep critical paths short.
62. Isolate optional dependencies.
63. Use asynchronous processing where appropriate.
64. Use queues to absorb bursts.
65. Bound retries.
66. Use exponential backoff.
67. Use jitter.
68. Use timeouts.
69. Use circuit breakers.
70. Use bulkheads.
71. Use rate limiting.
72. Use load shedding.
73. Use graceful degradation.
74. Protect database connections.
75. Protect thread pools.
76. Protect node resources.
77. Protect network capacity.
78. Protect storage capacity.
79. Protect IP capacity.
80. Protect observability capacity.
81. Protect CI/CD capacity.
82. Protect artifact availability.
83. Protect backup availability.
84. Protect recovery credentials.
85. Design for failure domains.
86. Design for blast-radius limits.
87. Use progressive delivery.
88. Use canaries for risky changes.
89. Use blue-green when fast switching justifies cost.
90. Use rolling deployments when capacity is constrained.
91. Preserve backward compatibility.
92. Use expand/contract migrations.
93. Avoid destructive schema changes during peak traffic.
94. Build once and promote immutable artifacts where practical.
95. Scan artifacts before promotion.
96. Sign trusted artifacts where appropriate.
97. Control dependency updates.
98. Pin versions appropriately.
99. Test upgrades progressively.
100. Make application versions observable.
```

## 211. Rules 101–150

```text
101. Use least privilege.
102. Prefer short-lived credentials.
103. Avoid shared production credentials.
104. Separate environment permissions.
105. Use workload identity.
106. Use network segmentation.
107. Restrict east-west traffic where appropriate.
108. Restrict egress where justified.
109. Protect metadata endpoints.
110. Protect audit logs.
111. Separate security administration.
112. Protect break-glass access.
113. Audit emergency actions.
114. Use policy-as-code.
115. Validate infrastructure changes.
116. Review Terraform plans.
117. Protect Terraform state.
118. Review GitOps changes.
119. Protect production branches.
120. Stage IAM changes.
121. Stage network changes.
122. Stage configuration changes.
123. Stage certificate changes.
124. Stage secret rotation.
125. Keep emergency changes reversible.
126. Reconcile emergency changes into desired state.
127. Monitor configuration drift.
128. Monitor policy drift.
129. Monitor identity drift.
130. Monitor infrastructure drift.
131. Use resource quotas.
132. Use PDBs appropriately.
133. Use topology spread where required.
134. Use anti-affinity where useful.
135. Avoid unnecessary dedicated node pools.
136. Use dedicated pools when isolation justifies them.
137. Bound autoscaling.
138. Monitor scaling behavior.
139. Prevent scaling oscillation.
140. Choose scaling metrics that represent workload pressure.
141. Use queue depth for queue-driven workers when appropriate.
142. Use business metrics where technical metrics are insufficient.
143. Diversify Spot capacity.
144. Keep critical capacity on appropriate reliable capacity.
145. Use ARM only after compatibility validation.
146. Measure actual workload performance.
147. Use benchmarks carefully.
148. Validate with production-like traffic.
149. Test degraded modes.
150. Test failure recovery.
```

## 212. Rules 151–200

```text
151. Test backup restoration.
152. Test regional failover.
153. Test database failover.
154. Test certificate renewal.
155. Test credential rotation.
156. Test node failure.
157. Test AZ failure.
158. Test cluster failure.
159. Test observability failure.
160. Test CI/CD failure.
161. Test incident escalation.
162. Test runbooks.
163. Conduct game days.
164. Conduct controlled chaos experiments.
165. Define abort criteria.
166. Measure recovery time.
167. Measure customer impact.
168. Measure blast radius.
169. Measure change failure rate.
170. Measure SLO performance.
171. Use error budgets.
172. Do not chase 100% availability without business justification.
173. Do not treat CPU as the only performance metric.
174. Do not treat infrastructure health as customer health.
175. Use synthetic checks.
176. Use real-user signals where appropriate.
177. Use metrics, logs and traces together.
178. Control telemetry cardinality.
179. Control log volume.
180. Control trace sampling.
181. Protect monitoring from monitored-system failure.
182. Keep critical external health checks.
183. Use managed logging when operational savings justify it.
184. Use self-managed observability when control requirements justify it.
185. Avoid technology sprawl.
186. Prefer platform standards.
187. Provide escape hatches.
188. Keep platform interfaces versioned.
189. Avoid breaking developer workflows unnecessarily.
190. Automate safe repetitive tasks.
191. Require approval for destructive high-risk tasks.
192. Bound auto-remediation.
193. Validate automated actions.
194. Stop automation on abnormal conditions.
195. Keep incident roles clear.
196. Maintain incident runbooks.
197. Maintain architecture diagrams.
198. Maintain dependency maps.
199. Assign owners.
200. Treat unowned systems as architectural risk.
```

## 213. Rules 201–250

```text
201. Evaluate cost per business unit.
202. Evaluate cost per transaction.
203. Evaluate cost per request.
204. Optimize customer value, not raw infrastructure cost.
205. Do not remove redundancy blindly to save money.
206. Do not overprovision blindly for safety.
207. Maintain failure headroom.
208. Include failure load in capacity planning.
209. Include deployment overlap in capacity planning.
210. Include traffic spikes in capacity planning.
211. Consider cross-AZ traffic cost.
212. Consider cross-region traffic cost.
213. Consider storage retrieval cost.
214. Consider observability retention cost.
215. Consider CI/CD compute cost.
216. Consider platform team cost.
217. Consider on-call cost.
218. Consider migration cost.
219. Consider vendor exit cost.
220. Consider security incident cost.
221. Prefer reversible choices when uncertainty is high.
222. Review one-way-door decisions carefully.
223. Use weighted decision matrices when useful.
224. Make assumptions visible.
225. Challenge assumptions with experiments.
226. Use proof-of-concepts for uncertain technical risks.
227. Do not confuse a POC with production readiness.
228. Validate operational readiness.
229. Validate security readiness.
230. Validate recovery readiness.
231. Validate cost behavior.
232. Validate scaling behavior.
233. Validate failure behavior.
234. Validate upgrade behavior.
235. Validate rollback behavior.
236. Validate data migration behavior.
237. Validate dependency failure behavior.
238. Validate degraded user experience.
239. Keep optional features detachable.
240. Keep critical dependencies explicit.
241. Use idempotency for retry-sensitive operations.
242. Prefer at-least-once plus idempotency when appropriate.
243. Use schemas for critical integrations.
244. Version APIs intentionally.
245. Version platform interfaces.
246. Remove obsolete feature flags.
247. Remove obsolete architecture components.
248. Revisit architecture when scale, SLO, team or business constraints change.
249. A senior engineer explains consequences, not just technology choices.
250. The best architecture is the simplest design that satisfies the required
     business outcomes, reliability, security, scale and operational
     constraints while making its accepted trade-offs explicit.
```

# PART V — SENIOR INTERVIEW ANSWER FRAMEWORK

## 214. Standard Answer

```text
First, I clarify the business and non-functional requirements.

Then I identify availability, latency, scale, security, RTO/RPO, consistency
and cost constraints.

I compare at least two viable architectures and explain the trade-offs.

I choose the simplest architecture that satisfies the requirements, identify
the risks I am accepting, add appropriate controls, and define how I would
validate the design in production.

If the assumptions change, I revisit the decision rather than treating the
architecture as permanent.
```

## 215. When the Interviewer Says "Why AWS?"

```text
I would not choose AWS simply because it is popular.

I would evaluate the required managed services, team expertise, availability
requirements, security model, operational burden, cost and existing
organizational standards.

If AWS provides the strongest fit under those constraints, I would use its
managed capabilities while accepting the relevant provider dependency.
```

## 216. When Asked "Why Kubernetes?"

```text
I would use Kubernetes when workload count, deployment frequency, scheduling,
service discovery, autoscaling and platform standardization justify the
additional operational complexity.

For a small application with simple deployment requirements, a simpler
platform may be preferable.
```

## 217. When Asked "Why Multi-Region?"

```text
I would first establish whether a regional outage is within the required
failure model.

If the business RTO/RPO cannot be met with single-region multi-AZ recovery,
then I would evaluate multi-region.

I would explicitly account for replication, consistency, traffic management,
operational complexity and cost.
```

## 218. When Asked "How Do You Reduce Cost?"

```text
I first identify the business workload and reliability constraints.

Then I optimize utilization, rightsizing, autoscaling, storage lifecycle,
network transfer, observability retention and committed-use economics without
violating SLOs.

I measure cost per business transaction rather than optimizing the invoice
blindly.
```

## 219. When Asked "How Do You Choose Between Two Technologies?"

```text
I compare them against the actual workload:

protocol
scale
latency
consistency
availability
security
operational burden
cost
team expertise
migration risk
vendor dependency

Then I choose based on weighted requirements rather than familiarity.
```

---