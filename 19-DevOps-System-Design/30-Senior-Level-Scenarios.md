# Senior-Level-Scenarios

## Senior DevOps Production Scenarios

This chapter focuses on the reasoning expected from senior DevOps engineers:
production incidents, architecture decisions, migrations, security events,
scaling crises, reliability engineering, platform engineering and interview
scenarios.

## Universal Senior Incident Method

```text
Customer impact
 |
Timeline
 |
Evidence
 |
Hypotheses
 |
Containment
 |
Mitigation
 |
Recovery
 |
Validation
 |
Root cause
 |
Prevention
```

## 1. Production Incident — API 5xx Spike

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 2. Production Incident — Database CPU 100%

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 3. Production Incident — Kubernetes Nodes Exhausted

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 4. Production Incident — Kubernetes IP Exhaustion

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 5. Production Incident — ALB 5xx

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 6. Production Incident — DNS Failure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 7. Production Incident — Redis Failure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 8. Production Incident — Queue Backlog

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 9. Production Incident — Retry Storm

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 10. Production Incident — Cascading Failure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 11. Production Incident — Memory Leak

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 12. Production Incident — Disk Full

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 13. Production Incident — Certificate Expiry

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 14. Production Incident — Secret Rotation Breaks Production

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 15. Production Incident — Bad Terraform Apply

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 16. Production Incident — Bad GitOps Change

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 17. Production Incident — Argo CD Unavailable

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 18. Production Incident — Container Registry Outage

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 19. Production Incident — CI Platform Outage

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 20. Security Incident — Compromised Container Image

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 21. Security Incident — Leaked AWS Credentials

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 22. Security Incident — Compromised Pod

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 23. Security Incident — Ransomware Scenario

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 24. Security Incident — Supply Chain Dependency

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 25. Security Incident — Over-Permissioned IAM Role

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 26. Security Incident — Public Object Storage Exposure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 27. Security Incident — WAF False Positive

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 28. Architecture Decision — Single Cluster vs Multiple Clusters

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 29. Architecture Decision — EKS vs ECS vs EC2

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 30. Architecture Decision — Serverless vs Containers

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 31. Architecture Decision — Monolith vs Microservices

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 32. Architecture Decision — Multi-AZ vs Multi-Region

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 33. Architecture Decision — Active-Active vs Active-Passive

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 34. Architecture Decision — SQL vs NoSQL

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 35. Architecture Decision — Redis vs Database

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 36. Architecture Decision — Queue vs Synchronous Call

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 37. Architecture Decision — Build vs Buy

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 38. Architecture Decision — Managed vs Self-Managed

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 39. Architecture Decision — Centralized vs Distributed Observability

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 40. Architecture Decision — Centralized vs Team-Owned CI/CD

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 41. Architecture Decision — GitOps vs Imperative Deployment

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 42. Architecture Decision — Blue-Green vs Canary

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 43. Architecture Decision — Spot vs On-Demand

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 44. Architecture Decision — NAT Gateway vs VPC Endpoints

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 45. Architecture Decision — Large vs Small EKS Clusters

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 46. Architecture Decision — Shared vs Dedicated Node Pools

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 47. Architecture Decision — Centralized vs Team AWS Accounts

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 48. Architecture Decision — One Region vs Global

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 49. Architecture Decision — Strong vs Eventual Consistency

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 50. Architecture Decision — Synchronous vs Asynchronous Integration

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 51. Architecture Decision — Logging Retention

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 52. Architecture Decision — Full Tracing vs Sampling

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 53. Architecture Decision — Platform Standardization vs Autonomy

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 54. Architecture Decision — Automation vs Human Approval

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 55. Architecture Decision — Rebuild vs Repair

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 56. Architecture Decision — Rollback vs Roll Forward

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 57. Senior Scenario — Latency Doubles Without Errors

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 58. Senior Scenario — CPU Normal but Application Is Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 59. Senior Scenario — Deployment Succeeds but Users See Old Version

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 60. Senior Scenario — Pods Restart During Node Scaling

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 61. Senior Scenario — One Tenant Consumes All Capacity

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 62. Senior Scenario — Cost Doubles

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 63. Senior Scenario — Observability Costs Explode

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 64. Senior Scenario — Deployment During Incident

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 65. Senior Scenario — Dependency Down for Two Hours

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 66. Senior Scenario — Traffic Suddenly 10x

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 67. Senior Scenario — Region Degraded

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 68. Senior Scenario — Database Replication Lag

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 69. Senior Scenario — Duplicate Queue Messages

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 70. Senior Scenario — Dead-Letter Queue Growing

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 71. Senior Scenario — Kubernetes API Is Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 72. Senior Scenario — Admission Webhook Failure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 73. Senior Scenario — Secret Store Unavailable

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 74. Senior Scenario — Monitoring Healthy but Customers Complain

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 75. Senior Scenario — No Alerts Fired

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 76. Senior Scenario — Alert Fatigue

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 77. Senior Scenario — Direct SSH Requested

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 78. Senior Scenario — Emergency Manual Production Change

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 79. Senior Scenario — New Team Requests Own Cluster

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 80. Senior Scenario — Platform Team Is Bottleneck

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 81. Senior Scenario — Team Rejects Standards

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 82. Senior Scenario — Four-Nines on Limited Budget

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 83. Senior Scenario — Five-Nines Requested

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 84. Senior Scenario — Five-Minute RTO

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 85. Senior Scenario — Near-Zero RPO

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 86. Senior Scenario — DR Test Fails

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 87. Senior Scenario — Backup Restore Too Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 88. Senior Scenario — Corruption Replicated Everywhere

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 89. Senior Scenario — AZ Failure During Deployment

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 90. Senior Scenario — Spot Capacity Disappears

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 91. Senior Scenario — Node Autoscaler Cannot Launch

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 92. Senior Scenario — Deployment Doubles Cost

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 93. Senior Scenario — Zero-Downtime Database Migration

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 94. Senior Scenario — API Version Migration

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 95. Senior Scenario — Upgrade Across 20 Clusters

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 96. Senior Scenario — Urgent Security Patch

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 97. Senior Scenario — Compliance Requires Auditability

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 98. Senior Scenario — Too Many Architecture Components

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 99. Senior Scenario — Multi-Cloud Availability Request

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 100. Senior Scenario — Vendor Lock-In Concern

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 101. Senior Scenario — No Clear Ownership

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 102. Senior Scenario — SLO Constantly Violated

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 103. Senior Scenario — Error Budget Exhausted

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 104. Senior Scenario — Every Service Wants Its Own Database

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 105. Senior Scenario — Shared Database Bottleneck

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 106. Senior Scenario — Single NAT Gateway

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 107. Senior Scenario — Network CIDR Exhaustion

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 108. Senior Scenario — Cross-Region Transfer Too Expensive

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 109. Senior Scenario — Logging Platform Affects Apps

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 110. Senior Scenario — Extreme Trace Cardinality

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 111. Senior Scenario — CI Queue Takes Hours

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 112. Senior Scenario — Maven Build Is Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 113. Senior Scenario — Docker Build Is Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 114. Senior Scenario — Artifact Repository Grows Rapidly

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 115. Senior Scenario — Mutable Image Tag

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 116. Senior Scenario — Direct Production Deploy Requested

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 117. Senior Scenario — Production Access Too Broad

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 118. Senior Scenario — Shared Platform Upgrade Breaks Teams

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 119. Senior Scenario — One Standard for Everything

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 120. Senior Scenario — Complete Team Freedom

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 121. Senior Scenario — Rollback Is Slow

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 122. Senior Scenario — Feature Flag Service Fails

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 123. Senior Scenario — Global Configuration Failure

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 124. Senior Scenario — Shared DNS Change Causes Outage

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 125. Senior Scenario — 24/7 Deployments

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 126. Senior Scenario — 100% Test Coverage but Incidents

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 127. Senior Scenario — Load Test Passes but Production Fails

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 128. Senior Scenario — Architecture at 10x Traffic

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 129. Senior Scenario — Business Wants 100x Growth

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 130. Senior Scenario — Recovery Depends on One Person

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 131. Senior Scenario — Runbook Is Too Long

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 132. Senior Scenario — Incident Has No Timeline

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 133. Senior Scenario — Teams Blame Each Other

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 134. Senior Scenario — Blaming Postmortem

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 135. Senior Scenario — Repeated Incident

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 136. Senior Scenario — Excellent but Too Expensive Architecture

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 137. Senior Scenario — Cheap but Unreliable Architecture

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 138. Senior Scenario — Critical Service Has No SLO

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 139. Senior Scenario — Architecture Approval

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 140. Senior Scenario — Interviewer Says Design It Better

### Situation

A production system has entered the condition described by this scenario.

### Senior Investigation

Start with customer impact, timeline, recent changes, metrics, logs, traces,
dependencies, capacity and the exact failure domain. Avoid assuming the first
symptom is the root cause.

### Response

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
investigate
 |
prevent
```

During an active outage, restoration of customer service takes priority over
deep root-cause analysis.

### Production Considerations

Discuss:

```text
blast radius
failure domains
timeouts
retries
capacity
security
observability
rollback
DR
ownership
cost
```

### Interview Answer

"I would first establish customer impact and scope, correlate the timeline
with changes and dependency health, then contain the blast radius and apply
the safest reversible mitigation. After recovery I would validate using both
technical and business signals, identify the root cause, and implement a
specific preventive control."

### Senior Checklist

```text
[ ] customer impact
[ ] scope
[ ] timeline
[ ] recent changes
[ ] metrics
[ ] logs
[ ] traces
[ ] dependencies
[ ] containment
[ ] recovery
[ ] validation
[ ] prevention
[ ] ownership
```


## 151. Universal Architecture Framework

```text
Business requirement
 |
Scale
 |
SLO / SLA
 |
RTO / RPO
 |
Failure model
 |
Network
 |
Compute
 |
Data
 |
Delivery
 |
Security
 |
Observability
 |
Recovery
 |
Cost
 |
Trade-offs
```

## 152. Production Readiness Gate

```text
[ ] owner
[ ] on-call
[ ] SLO
[ ] dashboard
[ ] actionable alerts
[ ] logs
[ ] traces where useful
[ ] health checks
[ ] resource requests
[ ] autoscaling
[ ] security controls
[ ] secrets strategy
[ ] backup
[ ] restore test
[ ] deployment strategy
[ ] rollback
[ ] dependency map
[ ] runbook
[ ] incident procedure
[ ] cost visibility
```

## 153. Senior Interview Signals

A strong senior answer demonstrates:

```text
customer-first prioritization
evidence-driven troubleshooting
failure-domain awareness
blast-radius reduction
safe automation
rollback thinking
security awareness
cost awareness
operational ownership
measurable outcomes
explicit trade-offs
```

Avoid product-first answers. Explain why a component exists and what failure
it prevents or contains.

## 154. 100 Senior Follow-Up Questions

```text
1. Why is this component necessary?
2. What is the first bottleneck?
3. What happens if it fails?
4. What is the blast radius?
5. What is the SLO?
6. What is the RTO?
7. What is the RPO?
8. How do you test DR?
9. How do you rollback?
10. How do you prevent a retry storm?
11. How do you protect the database?
12. How do you handle cache failure?
13. How do you handle queue backlog?
14. How do you handle poison messages?
15. How do you handle duplicate events?
16. How do you handle region failure?
17. How do you handle AZ failure?
18. How do you handle node failure?
19. How do you handle pod failure?
20. How do you handle IP exhaustion?
21. How do you handle storage exhaustion?
22. How do you handle certificate expiry?
23. How do you rotate secrets?
24. How do you limit IAM?
25. How do you prevent lateral movement?
26. How do you secure CI runners?
27. How do you secure artifacts?
28. How do you generate an SBOM?
29. How do you verify an image?
30. How do you stop a vulnerable release?
31. How do you stage platform changes?
32. How do you stage cluster upgrades?
33. How do you test Terraform?
34. How do you protect Terraform state?
35. How do you protect GitOps?
36. How do you recover from bad GitOps?
37. How do you handle Argo CD failure?
38. How do you handle registry failure?
39. How do you handle CI failure?
40. How do you handle observability failure?
41. How do you control telemetry cost?
42. How do you control log retention?
43. How do you control trace volume?
44. How do you handle noisy neighbors?
45. How do you isolate tenants?
46. Why EKS?
47. Why ECS?
48. Why EC2?
49. Why serverless?
50. Why multi-region?
51. Why not multi-cloud?
52. Why active-active?
53. Why active-passive?
54. Why SQL?
55. Why NoSQL?
56. Why Redis?
57. Why queues?
58. Why synchronous calls?
59. Why asynchronous processing?
60. Why GitOps?
61. Why blue-green?
62. Why canary?
63. How do you define canary success?
64. Which metrics stop deployment?
65. How do you protect error budget?
66. How do you handle schema migration?
67. How do you handle API versioning?
68. How do you migrate a monolith?
69. How do you migrate clusters?
70. How do you migrate regions?
71. How do you migrate clouds?
72. How do you prove backup recovery?
73. How do you recover corrupted data?
74. How do you handle ransomware?
75. How do you respond to leaked credentials?
76. How do you respond to compromised images?
77. How do you respond to a compromised pod?
78. How do you investigate an IAM incident?
79. How do you respond to WAF false positives?
80. How do you handle DNS outage?
81. How do you handle load-balancer outage?
82. How do you handle database connection exhaustion?
83. How do you handle API latency?
84. How do you handle CPU saturation?
85. How do you handle memory leaks?
86. How do you handle disk-full incidents?
87. How do you handle a failed deployment?
88. How do you handle a failed rollback?
89. How do you choose rollback versus roll-forward?
90. How do you protect shared infrastructure?
91. How do you reduce platform blast radius?
92. How do you balance standardization and autonomy?
93. How do you handle teams that reject platform standards?
94. How do you prevent platform bottlenecks?
95. How do you design self-service?
96. How do you define ownership?
97. How do you design runbooks?
98. How do you run a game day?
99. How do you run a postmortem?
100. How do you ensure corrective actions happen?
101. How do you measure platform success?
102. How do you optimize cost without damaging SLOs?
103. How does the design change at 10x scale?
104. How does it change at 100x scale?
105. Which decision is hardest to reverse?
106. Which component is the biggest operational risk?
```

## 155. Final Senior Mental Model

The senior DevOps mindset is not simply:

```text
How do I deploy this?
```

It is:

```text
How does it fail?
How quickly can we detect it?
How do we contain it?
How do we recover?
How do we prove recovery?
How do we prevent recurrence?
Who owns it?
What does it cost?
What happens at 10x scale?
What happens when a dependency fails?
What happens when credentials are compromised?
```

A production architecture is complete only when deployment, security,
observability, failure handling, recovery and ownership are designed together.

---