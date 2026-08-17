# Data-Structures

> Data structures are the foundation of DevOps automation. AWS responses, Kubernetes objects, JSON/YAML configuration, CI/CD metadata, logs, metrics, inventories, deployment state, and incident evidence are all represented as structured data. The goal is not only to know Python syntax, but to choose the right structure, process data safely, and scale the automation.

---

# 1. Core Data Structures

The four most important built-in structures are:

```text
list       -> ordered, mutable collection
tuple      -> ordered, immutable collection
set        -> unique values
dictionary -> key/value mapping
```

Common supporting structures:

```text
deque
defaultdict
Counter
frozenset
generators
dataclasses
TypedDict
```

---

# 2. How to Choose

```text
Need ordering?
    -> list

Need a fixed/immutable group?
    -> tuple

Need uniqueness or fast membership?
    -> set

Need lookup by key?
    -> dictionary

Need grouping?
    -> defaultdict

Need frequency counting?
    -> Counter

Need queue/recent bounded history?
    -> deque

Need large lazy processing?
    -> generator
```

---

# 3. Lists

```python
services = [
    "user",
    "catalog",
    "cart",
    "payment"
]
```

Lists are:

```text
ordered
mutable
indexable
sliceable
allow duplicates
```

DevOps examples:

```python
namespaces = ["dev", "stage", "production"]
ports = [80, 443, 8080]
services = ["user", "cart", "payment"]
```

---

# 4. List Indexing

```python
services[0]
services[1]
services[-1]
```

Negative indexing:

```text
-1 -> last
-2 -> second last
```

Useful when processing ordered operational data.

---

# 5. List Slicing

```python
services[1:3]
```

returns indexes 1 and 2.

Other forms:

```python
services[:3]
services[2:]
services[::2]
services[::-1]
```

Remember that the end index is excluded.

---

# 6. Add and Remove

```python
services.append("inventory")
services.extend(["notification", "orders"])
services.insert(1, "auth")
```

Remove:

```python
services.remove("cart")
service = services.pop()
service = services.pop(1)
del services[1]
services.clear()
```

`remove()` raises `ValueError` if the value does not exist.

`pop()` returns the removed value.

---

# 7. `append()` vs `extend()`

```python
items = ["a"]

items.append(["b", "c"])
```

produces:

```text
["a", ["b", "c"]]
```

Whereas:

```python
items = ["a"]
items.extend(["b", "c"])
```

produces:

```text
["a", "b", "c"]
```

This is a common interview question.

---

# 8. Membership

```python
if "payment" in services:
    print("Found")
```

For large collections with frequent membership checks, consider a set.

---

# 9. Iterating Lists

```python
for service in services:
    print(service)
```

With index:

```python
for index, service in enumerate(
    services,
    start=1
):
    print(index, service)
```

Prefer `enumerate()` over manually using `range(len(...))`.

---

# 10. Sorting

In-place:

```python
services.sort()
```

New list:

```python
sorted_services = sorted(
    services
)
```

Reverse:

```python
services.sort(reverse=True)
```

Sort by a field:

```python
pods.sort(
    key=lambda pod:
        pod["restart_count"],
    reverse=True
)
```

---

# 11. `.sort()` vs `sorted()`

`.sort()`:

```python
items.sort()
```

changes the original list and returns `None`.

`sorted()`:

```python
new_items = sorted(items)
```

returns a new list and leaves the original unchanged.

---

# 12. Lists of Dictionaries

This is extremely common in DevOps:

```python
pods = [
    {
        "name": "user-abc",
        "namespace": "production",
        "status": "Running"
    },
    {
        "name": "payment-xyz",
        "namespace": "production",
        "status": "Pending"
    }
]
```

Access:

```python
pods[0]["name"]
```

---

# 13. Filtering

```python
failed = [
    pod
    for pod in pods
    if pod["status"] != "Running"
]
```

Production examples:

```text
unhealthy pods
stopped instances
failed deployments
resources without tags
services with high latency
```

---

# 14. Transforming

```python
names = [
    pod["name"]
    for pod in pods
]
```

Conditional transformation:

```python
statuses = [
    "healthy"
    if pod["status"] == "Running"
    else "unhealthy"
    for pod in pods
]
```

---

# 15. Nested Lists

```python
clusters = [
    ["node-1", "node-2"],
    ["node-3", "node-4"]
]
```

Access:

```python
clusters[0][1]
```

Flatten:

```python
nodes = [
    node
    for cluster in clusters
    for node in cluster
]
```

---

# 16. List Copying

This does not create an independent list:

```python
backup = services
```

Shallow copy:

```python
backup = services.copy()
```

or:

```python
backup = services[:]
```

For deeply nested mutable structures:

```python
import copy

backup = copy.deepcopy(
    services
)
```

Avoid unnecessary deep copies for large AWS/Kubernetes responses because they can consume significant memory.

---

# 17. Tuples

```python
server = (
    "web-01",
    "10.0.1.10",
    8080
)
```

Tuples are:

```text
ordered
immutable
indexable
sliceable
```

Useful for fixed values and composite dictionary keys.

---

# 18. Tuple Unpacking

```python
status_code, message = (
    200,
    "OK"
)
```

A function can return:

```python
def health():
    return 200, "OK"
```

Then:

```python
code, message = health()
```

---

# 19. Tuple as Composite Key

```python
deployments = {
    ("production", "payment"): "1.5.0",
    ("stage", "payment"): "1.6.0"
}
```

Useful for:

```text
environment + service
account + region
cluster + namespace
region + resource ID
```

---

# 20. Sets

```python
services = {
    "user",
    "cart",
    "payment"
}
```

Sets are:

```text
unique
mutable
not indexable
optimized for membership
```

Use them when order is not the primary requirement.

---

# 21. Remove Duplicates

```python
items = [
    "user",
    "cart",
    "user",
    "payment"
]

unique = set(items)
```

If order matters:

```python
unique = list(
    dict.fromkeys(items)
)
```

Do not rely on set iteration order for deterministic reports.

---

# 22. Set Operations

Union:

```python
a | b
```

Intersection:

```python
a & b
```

Difference:

```python
a - b
```

Symmetric difference:

```python
a ^ b
```

---

# 23. Drift Detection With Sets

Desired:

```python
desired = {
    "user",
    "cart",
    "payment"
}
```

Actual:

```python
actual = {
    "user",
    "cart",
    "inventory"
}
```

Missing:

```python
desired - actual
```

Unexpected:

```python
actual - desired
```

Common:

```python
desired & actual
```

This is a fundamental DevOps reconciliation pattern.

---

# 24. Dictionary

```python
service = {
    "name": "payment",
    "port": 8080,
    "environment": "production",
    "replicas": 3
}
```

A dictionary maps keys to values.

Keys must be hashable.

---

# 25. Dictionary Access

```python
service["name"]
```

If the key is absent:

```text
KeyError
```

Safer for optional values:

```python
service.get("description")
```

With default:

```python
service.get(
    "replicas",
    1
)
```

---

# 26. Required vs Optional Fields

Required:

```python
name = service["name"]
```

Optional:

```python
description = service.get(
    "description"
)
```

This distinction is important when processing external API data.

---

# 27. Dictionary Mutation

```python
service["replicas"] = 4
service["version"] = "1.5.0"
```

Multiple:

```python
service.update({
    "replicas": 4,
    "version": "1.5.0"
})
```

Delete:

```python
del service["version"]
```

Safer removal:

```python
service.pop(
    "version",
    None
)
```

---

# 28. Dictionary Iteration

Keys:

```python
for key in service:
    print(key)
```

Values:

```python
for value in service.values():
    print(value)
```

Both:

```python
for key, value in service.items():
    print(key, value)
```

Use `items()` when both key and value are required.

---

# 29. Dictionary Comprehension

```python
service_ports = {
    service["name"]: service["port"]
    for service in services
}
```

This is one of the most useful DevOps patterns.

It creates an index:

```text
service name -> port
```

---

# 30. Build Resource Index

```python
resources_by_id = {
    resource["id"]: resource
    for resource in resources
}
```

Now:

```python
resources_by_id["i-123"]
```

is a direct lookup instead of scanning the whole list.

---

# 31. Duplicate Key Warning

If two resources have the same ID:

```python
{
    resource["id"]: resource
    for resource in resources
}
```

the later value silently overwrites the earlier value.

Validate uniqueness when IDs are expected to be unique:

```python
ids = [
    resource["id"]
    for resource in resources
]

if len(ids) != len(set(ids)):
    raise ValueError(
        "Duplicate resource IDs"
    )
```

---

# 32. Nested Dictionaries

Kubernetes-style example:

```python
deployment = {
    "metadata": {
        "name": "payment"
    },
    "spec": {
        "replicas": 3
    }
}
```

Access:

```python
deployment["spec"]["replicas"]
```

For optional nested data:

```python
replicas = (
    deployment
    .get("spec", {})
    .get("replicas")
)
```

---

# 33. Normalize Deep Data

Avoid passing deeply nested API structures everywhere.

Instead:

```python
def normalize_pod(pod):
    return {
        "name": pod["metadata"]["name"],
        "namespace": pod["metadata"].get(
            "namespace"
        ),
        "phase": pod.get(
            "status",
            {}
        ).get("phase")
    }
```

Downstream code now uses:

```python
pod["name"]
pod["namespace"]
pod["phase"]
```

This reduces coupling.

---

# 34. AWS Response Structures

AWS APIs commonly return dictionaries containing lists containing dictionaries.

Example:

```python
response = {
    "Reservations": [
        {
            "Instances": [
                {
                    "InstanceId": "i-123",
                    "State": {
                        "Name": "running"
                    }
                }
            ]
        }
    ]
}
```

Process:

```python
for reservation in response.get(
    "Reservations",
    []
):
    for instance in reservation.get(
        "Instances",
        []
    ):
        print(
            instance["InstanceId"]
        )
```

---

# 35. Normalize AWS Resources

```python
def normalize_instance(raw):
    return {
        "id": raw.get("InstanceId"),
        "state": raw.get(
            "State",
            {}
        ).get("Name")
    }
```

This gives the rest of the application a stable structure.

---

# 36. Kubernetes Objects

Example:

```python
pod = {
    "metadata": {
        "name": "payment-abc",
        "namespace": "production",
        "labels": {
            "app": "payment"
        }
    },
    "status": {
        "phase": "Running"
    }
}
```

Access:

```python
pod["metadata"]["labels"]["app"]
```

For production automation, validate external structures instead of assuming every field exists.

---

# 37. JSON

JSON maps naturally to Python:

```text
object -> dict
array  -> list
string -> str
number -> int/float
true   -> True
false  -> False
null   -> None
```

Parse a string:

```python
import json

data = json.loads(
    '{"environment": "production"}'
)
```

---

# 38. JSON Files

Read:

```python
with open(
    "config.json"
) as file:
    config = json.load(file)
```

Write:

```python
with open(
    "report.json",
    "w"
) as file:
    json.dump(
        report,
        file,
        indent=2
    )
```

Serialize:

```python
payload = json.dumps(data)
```

---

# 39. YAML

Typical DevOps YAML:

```yaml
environment: production
replicas: 3
services:
  - user
  - payment
```

With PyYAML:

```python
import yaml

with open(
    "config.yaml"
) as file:
    config = yaml.safe_load(file)
```

Use safe loading for normal configuration data. Do not use unsafe YAML loaders for untrusted content.

---

# 40. YAML to Python

```yaml
services:
  payment:
    port: 8080
    replicas: 3
```

becomes approximately:

```python
{
    "services": {
        "payment": {
            "port": 8080,
            "replicas": 3
        }
    }
}
```

Access:

```python
config[
    "services"
]["payment"]["port"]
```

---

# 41. Configuration Structure

A practical DevOps configuration:

```python
config = {
    "environment": "production",
    "region": "ap-south-1",
    "cluster": "prod-eks",
    "services": [
        {
            "name": "user",
            "replicas": 3
        },
        {
            "name": "payment",
            "replicas": 4
        }
    ]
}
```

This is a dictionary containing a list of dictionaries.

---

# 42. Validate Configuration

```python
required = [
    "environment",
    "region",
    "cluster"
]

missing = [
    key
    for key in required
    if key not in config
]

if missing:
    raise ValueError(
        f"Missing: {missing}"
    )
```

Fail before infrastructure mutation.

---

# 43. Configuration Index

```python
replicas = {
    service["name"]:
        service["replicas"]
    for service in config[
        "services"
    ]
}
```

Then:

```python
replicas["payment"]
```

returns the desired replica count.

---

# 44. List Search vs Dictionary Lookup

Repeated search:

```python
for service in services:
    if service["name"] == "payment":
        ...
```

For many lookups, build:

```python
services_by_name = {
    service["name"]: service
    for service in services
}
```

Then:

```python
services_by_name["payment"]
```

This is an important scalability optimization.

---

# 45. `defaultdict`

Grouping example:

```python
from collections import defaultdict

pods_by_namespace = defaultdict(list)

for pod in pods:
    namespace = pod["namespace"]

    pods_by_namespace[
        namespace
    ].append(pod)
```

No need to manually create the list for each new namespace.

---

# 46. `Counter`

```python
from collections import Counter

states = Counter(
    pod["status"]
    for pod in pods
)
```

Useful for:

```text
pod states
HTTP statuses
log levels
error types
instance states
deployment results
```

Top values:

```python
states.most_common(5)
```

---

# 47. `deque`

```python
from collections import deque

queue = deque()

queue.append("job-1")
queue.append("job-2")

job = queue.popleft()
```

A `deque` is appropriate for efficient queue operations.

For distributed production queues, use a durable queue system such as SQS/RabbitMQ/Kafka when required.

---

# 48. Bounded Recent History

```python
recent_events = deque(
    maxlen=100
)

for event in events:
    recent_events.append(event)
```

Memory remains bounded.

Useful for incident diagnostics.

---

# 49. `frozenset`

```python
permissions = frozenset({
    "read",
    "list"
})
```

It is an immutable set and can be used as a dictionary key.

---

# 50. Permission Analysis

```python
required = {
    "read",
    "list",
    "watch"
}

granted = {
    "read",
    "list"
}

missing = required - granted
extra = granted - required
```

This is a useful RBAC analysis pattern.

Actual authorization must still be enforced by IAM/RBAC, not by Python data structures alone.

---

# 51. `zip()`

```python
services = [
    "user",
    "cart",
    "payment"
]

replicas = [2, 3, 4]

for service, count in zip(
    services,
    replicas
):
    print(service, count)
```

By default, `zip()` stops at the shortest input.

If equal lengths are required:

```python
zip(
    services,
    replicas,
    strict=True
)
```

---

# 52. `any()` and `all()`

At least one unhealthy:

```python
has_failure = any(
    pod["status"] != "Running"
    for pod in pods
)
```

All healthy:

```python
healthy = all(
    pod["status"] == "Running"
    for pod in pods
)
```

These are useful for deployment verification and health checks.

---

# 53. `min()` and `max()`

Highest restart count:

```python
highest = max(
    pods,
    key=lambda pod:
        pod["restart_count"]
)
```

Lowest latency:

```python
lowest = min(
    results,
    key=lambda item:
        item["latency_ms"]
)
```

Handle empty collections explicitly because `min([])` and `max([])` raise `ValueError`.

---

# 54. Generator Expressions

For aggregation:

```python
total_cost = sum(
    resource["monthly_cost"]
    for resource in resources
)
```

This avoids creating a separate list of costs.

---

# 55. Generators for Large Data

Instead of:

```python
def get_instances():
    return all_instances
```

use:

```python
def iter_instances(client):
    paginator = client.get_paginator(
        "describe_instances"
    )

    for page in paginator.paginate():
        for reservation in page.get(
            "Reservations",
            []
        ):
            for instance in reservation.get(
                "Instances",
                []
            ):
                yield instance
```

Then:

```python
for instance in iter_instances(client):
    process(instance)
```

This supports incremental processing.

---

# 56. Data Transformation Pipeline

A common production pattern:

```text
raw API response
      |
      v
extract
      |
      v
normalize
      |
      v
validate
      |
      v
filter
      |
      v
group/index
      |
      v
aggregate
      |
      v
report/action
```

This is far easier to maintain than passing raw API objects everywhere.

---

# 57. AWS Inventory Architecture

```text
AWS paginator
      |
      v
page
      |
      v
normalize instance
      |
      v
filter environment
      |
      v
index by ID
      |
      v
aggregate state
      |
      v
report
```

For large inventories, process incrementally and retain only data required for the report.

---

# 58. Kubernetes Inventory Architecture

```text
Kubernetes API
      |
      v
pods/nodes/deployments
      |
      v
normalize
      |
      v
filter unhealthy
      |
      v
group by namespace
      |
      v
count states
      |
      v
diagnostic report
```

---

# 59. Log Processing Architecture

```text
log stream
    |
    v
parse
    |
    v
normalize
    |
    v
classify
    |
    +--> error
    +--> warning
    +--> normal
    |
    v
Counter / aggregation
    |
    v
report
```

Do not load huge logs into memory unnecessarily.

---

# 60. Monitoring Data

A normalized metric record:

```python
metric = {
    "service": "payment",
    "metric": "error_rate",
    "value": 0.04
}
```

Multiple metrics:

```python
metrics = [
    {...},
    {...}
]
```

Be careful with metric semantics; not every value should simply be summed or averaged.

---

# 61. Alert Data

```python
alerts = [
    {
        "service": "payment",
        "type": "high_error_rate",
        "severity": "high"
    },
    {
        "service": "payment",
        "type": "high_latency",
        "severity": "high"
    }
]
```

Group by service:

```python
grouped = defaultdict(list)

for alert in alerts:
    grouped[
        alert["service"]
    ].append(alert)
```

This supports alert correlation.

---

# 62. Incident Evidence

```python
incident = {
    "service": "payment",
    "severity": "high",
    "metrics": {},
    "logs": [],
    "events": [],
    "deployment": {},
    "actions": []
}
```

This gives an incident tool a predictable output structure.

---

# 63. Error Aggregation

```python
errors = Counter(
    log["service"]
    for log in logs
    if log["level"] == "ERROR"
)
```

Top failing services:

```python
errors.most_common(5)
```

For very large environments, perform aggregation in the logging platform where practical rather than moving all raw logs into Python.

---

# 64. Normalize High-Cardinality Errors

Raw messages:

```text
Connection timeout for user 123
Connection timeout for user 456
```

should often be classified as:

```text
connection_timeout
```

before aggregation.

This prevents unique IDs from producing meaningless high-cardinality categories.

---

# 65. Kubernetes Restart Analysis

```python
top = sorted(
    pods,
    key=lambda pod:
        pod["restart_count"],
    reverse=True
)[:10]
```

This can quickly identify pods needing investigation.

---

# 66. Kubernetes Pod State Counting

```python
counts = Counter(
    pod.get(
        "phase",
        "Unknown"
    )
    for pod in pods
)
```

Report:

```text
Running
Pending
Failed
Unknown
```

---

# 67. Kubernetes Namespace Grouping

```python
by_namespace = defaultdict(list)

for pod in pods:
    namespace = pod.get(
        "namespace",
        "unknown"
    )

    by_namespace[
        namespace
    ].append(pod)
```

This is useful for cluster-level health reports.

---

# 68. AWS Tag Normalization

Raw:

```python
tags = [
    {
        "Key": "Environment",
        "Value": "production"
    },
    {
        "Key": "Owner",
        "Value": "platform"
    }
]
```

Normalize:

```python
tag_map = {
    tag["Key"]: tag["Value"]
    for tag in tags
}
```

Now:

```python
tag_map.get(
    "Environment"
)
```

---

# 69. Required Tag Validation

```python
required = {
    "Environment",
    "Owner",
    "Application"
}

actual = set(
    tag_map.keys()
)

missing = required - actual
```

This is a practical cloud governance check.

---

# 70. Environment Drift

```python
stage = {
    "user",
    "cart",
    "payment"
}

production = {
    "user",
    "cart"
}

missing = stage - production
```

Then classify whether the difference is expected or actual drift.

---

# 71. Configuration Drift

Desired:

```python
desired = {
    "replicas": 3,
    "image": "payment:1.5.0"
}
```

Current:

```python
current = {
    "replicas": 2,
    "image": "payment:1.4.0"
}
```

Diff:

```python
changes = {
    key: {
        "current": current.get(key),
        "desired": desired.get(key)
    }
    for key in desired
    if current.get(key)
    != desired.get(key)
}
```

---

# 72. Change Plan

Represent changes explicitly:

```python
changes = [
    {
        "resource": "payment",
        "field": "replicas",
        "current": 2,
        "desired": 3
    },
    {
        "resource": "payment",
        "field": "image",
        "current": "1.4.0",
        "desired": "1.5.0"
    }
]
```

This supports:

```text
dry-run
review
approval
audit
execution
```

---

# 73. Dry-Run Design

```python
def apply_change(
    change,
    *,
    dry_run=True
):
    if dry_run:
        return {
            "changed": False,
            "action": "would_change"
        }

    perform_change(change)

    return {
        "changed": True,
        "action": "changed"
    }
```

Destructive operations should have explicit safety controls.

---

# 74. Configuration Precedence

Represent layers:

```python
defaults = {
    "replicas": 2,
    "timeout": 300
}

environment = {
    "replicas": 3
}

runtime = {
    "timeout": 600
}
```

Merge:

```python
config = (
    defaults
    | environment
    | runtime
)
```

Later dictionaries override earlier top-level keys.

Document the precedence.

---

# 75. Shallow Merge Warning

If:

```python
base = {
    "service": {
        "replicas": 2,
        "timeout": 300
    }
}

override = {
    "service": {
        "replicas": 4
    }
}
```

then:

```python
base | override
```

replaces the entire nested `service` dictionary.

It does not recursively merge nested keys.

---

# 76. Data Classes

For stable configuration contracts:

```python
from dataclasses import dataclass

@dataclass
class ServiceConfig:
    name: str
    replicas: int
    port: int
```

Use:

```python
service = ServiceConfig(
    name="payment",
    replicas=3,
    port=8080
)
```

Dataclasses can be clearer than deeply nested dictionaries when the schema is stable.

---

# 77. TypedDict

For dictionary-shaped data:

```python
from typing import TypedDict

class Service(TypedDict):
    name: str
    replicas: int
    port: int
```

This gives type checkers an expected structure while the runtime object remains a dictionary.

---

# 78. Dictionary vs Dataclass

Use a dictionary when:

```text
data is dynamic
JSON serialization is important
provider response is variable
```

Consider a dataclass/TypedDict when:

```text
schema is stable
many functions use the same fields
validation/readability matter
```

---

# 79. Environment Variables

Environment variables are strings:

```python
value = os.environ.get(
    "REPLICAS"
)
```

If:

```text
REPLICAS=3
```

then `value` is:

```text
"3"
```

Convert:

```python
replicas = int(value)
```

Do not compare `"3"` to `3`.

---

# 80. CLI Data

CLI values often arrive as strings. Use `argparse` types:

```python
parser.add_argument(
    "--replicas",
    type=int,
    required=True
)
```

Then the application receives an integer.

---

# 81. Secrets and Data Structures

Never blindly log:

```python
print(config)
```

because it may contain:

```text
password
token
API key
private key
secret
```

Use redaction or log only safe fields.

---

# 82. Simple Redaction

```python
SENSITIVE_KEYS = {
    "password",
    "token",
    "secret",
    "api_key"
}

def redact(data):
    return {
        key: (
            "***"
            if key.lower()
            in SENSITIVE_KEYS
            else value
        )
        for key, value in data.items()
    }
```

For nested structures, redaction must recurse through dictionaries and lists.

---

# 83. Recursive Data

Configuration can look like:

```python
config = {
    "services": [
        {
            "name": "payment",
            "secrets": {
                "token": "secret"
            }
        }
    ]
}
```

A top-level redaction function will not protect the nested token.

Production redaction should recursively handle:

```text
dict
list
tuple
nested values
```

---

# 84. File/Log Streaming

Bad for huge files:

```python
lines = file.readlines()
```

Better:

```python
with open(path) as file:
    for line in file:
        process(line)
```

This keeps memory usage bounded.

---

# 85. Batch Processing

```python
def batches(items, size):
    for start in range(
        0,
        len(items),
        size
    ):
        yield items[
            start:start + size
        ]
```

Use:

```python
for batch in batches(
    resources,
    100
):
    process_batch(batch)
```

Useful for APIs and cloud resources.

---

# 86. Pagination

Do not assume an API response contains every resource.

Concept:

```text
page
 |
 +--> process items
 |
 +--> next token?
        |
       yes
        |
        v
     next page
```

Use SDK paginators where available.

---

# 87. Pagination With a Generator

```python
def iter_resources(client):
    paginator = client.get_paginator(
        "describe_resources"
    )

    for page in paginator.paginate():
        for item in page.get(
            "Resources",
            []
        ):
            yield item
```

Then process incrementally.

---

# 88. Memory-Safe Processing

Avoid:

```python
raw = get_all()
normalized = [
    normalize(x)
    for x in raw
]
filtered = [
    x for x in normalized
    if condition(x)
]
```

This can create several large structures.

Prefer a pipeline:

```python
for raw in iter_resources():
    item = normalize(raw)

    if condition(item):
        process(item)
```

---

# 89. Large-Scale Lookup

Bad:

```python
for service in services:
    for resource in resources:
        if resource["service"] == service:
            ...
```

This can become expensive.

Build an index:

```python
resources_by_service = defaultdict(list)

for resource in resources:
    resources_by_service[
        resource["service"]
    ].append(resource)
```

Then:

```python
resources_by_service["payment"]
```

---

# 90. Complexity

Typical conceptual costs:

```text
list index          O(1)
list search         O(n)
list append         amortized O(1)

dict lookup         average O(1)
set membership      average O(1)

sorting             O(n log n)
```

The exact behavior depends on implementation and workload, but this model is useful for interview reasoning.

---

# 91. Why Complexity Matters

A script that scans:

```text
500 resources
```

may be acceptable.

The same design against:

```text
1,000,000 resources
```

can become a major bottleneck.

Choose structures based on expected scale.

---

# 92. Top-N Instead of Full Sort

If you only need the top 10:

```python
from heapq import nlargest

top = nlargest(
    10,
    resources,
    key=lambda x:
        x["cpu"]
)
```

This may avoid sorting the entire dataset.

---

# 93. Bounded Aggregation

Instead of storing every error:

```python
errors = []
```

use:

```python
from collections import Counter

errors = Counter()

for log in logs:
    errors[
        log["error_type"]
    ] += 1
```

This can dramatically reduce memory.

---

# 94. Recent Events

```python
recent = deque(
    maxlen=100
)

for event in events:
    recent.append(event)
```

Only the latest 100 are retained.

---

# 95. Data Structures and Concurrency

Do not assume that shared mutable structures are safe just because Python provides them.

Prefer:

```text
workers return results
main coordinator aggregates
```

instead of every worker mutating shared global state.

For distributed systems, use proper queues/storage.

---

# 96. Async Processing

```python
results = await asyncio.gather(
    check_service("user"),
    check_service("cart"),
    check_service("payment")
)
```

For many services, use bounded concurrency.

Do not create unlimited tasks against production APIs.

---

# 97. Data Structure and Rate Limits

Represent policy:

```python
policy = {
    "max_concurrency": 10,
    "timeout": 5,
    "max_retries": 3
}
```

Use it to bound API operations.

Respect:

```text
AWS throttling
Kubernetes API capacity
database limits
external service rate limits
```

---

# 98. Data Structure and Caching

A dictionary can implement a simple cache:

```python
cache = {}

cache["config"] = config
```

But production caches need:

```text
TTL
invalidation
maximum size
refresh policy
concurrency considerations
```

Do not cache live infrastructure state indefinitely.

---

# 99. Data Structure and Queue

For a local worker:

```python
queue = deque(
    ["job-1", "job-2"]
)

while queue:
    job = queue.popleft()
    process(job)
```

For durable distributed work, use a real queue system.

---

# 100. Data Structure and Incident Correlation

```python
alerts_by_service = defaultdict(list)

for alert in alerts:
    alerts_by_service[
        alert["service"]
    ].append(alert)
```

Now a single service can be correlated across:

```text
alerts
metrics
logs
deployments
events
```

---

# 101. Data Structure and SLO

```python
slo = {
    "service": "payment",
    "target": 0.999,
    "availability": 0.9985,
    "error_budget_remaining": 0.0005
}
```

A list can hold multiple service SLOs.

For actual SLO computation, use correct time-series aggregation rather than arbitrary local samples.

---

# 102. Data Structure and CI/CD

```python
pipeline = {
    "build": True,
    "tests": True,
    "sonarqube": True,
    "trivy": True,
    "veracode": True,
    "deployment": True
}
```

Gate:

```python
if not all(
    pipeline.values()
):
    raise SystemExit(1)
```

This is a simple DevSecOps gate representation.

---

# 103. Data Structure and Docker

```python
image = {
    "repository": "payment",
    "tag": "1.5.0",
    "digest": "sha256:...",
    "scan_status": "passed"
}
```

Validation:

```python
if image["scan_status"] != "passed":
    raise RuntimeError(
        "Security gate failed"
    )
```

---

# 104. Data Structure and Terraform

Terraform-related data can be normalized into:

```python
resource = {
    "address": "aws_instance.web",
    "type": "aws_instance",
    "action": "update"
}
```

Then:

```python
if resource["action"] == "delete":
    review(resource)
```

Python should normally consume Terraform output/state information rather than manually editing Terraform state.

---

# 105. Data Structure and GitOps

Desired state:

```python
desired = {
    "service": "payment",
    "image": "payment:1.5.0",
    "replicas": 3
}
```

A GitOps workflow should update the declarative source of truth rather than bypassing it with direct live mutation when Git is the chosen authority.

---

# 106. Data Structure and Deployment Matrix

```python
matrix = {
    ("production", "payment"): {
        "version": "1.5.0",
        "replicas": 4
    },
    ("stage", "payment"): {
        "version": "1.6.0",
        "replicas": 2
    }
}
```

Tuple keys are useful for composite identities.

---

# 107. Data Structure and Resource Ownership

```python
resource = {
    "id": "i-123",
    "environment": "production",
    "owner": "platform",
    "managed_by": "terraform"
}
```

A cleanup utility can use this metadata for policy decisions.

Actual authorization must remain enforced by cloud/IAM controls.

---

# 108. Data Structure and Approval

```python
action = {
    "name": "scale",
    "environment": "production",
    "approved": True,
    "dry_run": False
}
```

Before mutation:

```python
if (
    action["environment"]
    == "production"
    and not action["approved"]
):
    raise PermissionError(
        "Approval required"
    )
```

The real authorization boundary should be CI/CD/cloud identity, not just this dictionary.

---

# 109. Data Structure and Audit

```python
audit = {
    "action": "scale",
    "resource": "payment",
    "previous": 2,
    "requested": 3,
    "result": "success"
}
```

Do not place:

```text
passwords
tokens
private keys
secret payloads
```

into audit records.

---

# 110. Data Structure and Rollback

```python
release = {
    "current": "1.5.0",
    "previous": "1.4.0",
    "candidate": "1.6.0"
}
```

Rollback target:

```python
release["previous"]
```

But application/database compatibility must be considered before rollback.

---

# 111. Data Structure and Blue/Green

```python
deployment = {
    "blue": {
        "version": "1.4.0",
        "healthy": True
    },
    "green": {
        "version": "1.5.0",
        "healthy": True
    }
}
```

Traffic routing is handled separately.

---

# 112. Data Structure and Canary

```python
canary = {
    "version": "1.5.0",
    "traffic_percent": 10,
    "error_rate": 0.01,
    "latency_ms": 120
}
```

A policy function can decide whether traffic should increase.

---

# 113. Data Structure and HPA Diagnostics

```python
hpa = {
    "min_replicas": 2,
    "max_replicas": 10,
    "current_replicas": 4,
    "target_cpu": 70,
    "current_cpu": 82
}
```

This supports troubleshooting and reporting.

---

# 114. Data Structure and OOMKilled

```python
container = {
    "last_state": {
        "terminated": {
            "reason": "OOMKilled",
            "exit_code": 137
        }
    }
}
```

Check:

```python
reason = (
    container
    .get("last_state", {})
    .get("terminated", {})
    .get("reason")
)

if reason == "OOMKilled":
    investigate_memory()
```

---

# 115. Data Structure and Disk Troubleshooting

```python
filesystem = {
    "/": 72,
    "/var": 91,
    "/var/log": 95
}

critical = {
    mount: usage
    for mount, usage
    in filesystem.items()
    if usage >= 90
}
```

This mirrors the type of structure a Python troubleshooting tool can create from command/API output.

---

# 116. Data Structure and Network Troubleshooting

```python
connectivity = {
    "dns": True,
    "tcp": True,
    "tls": False,
    "http": False
}
```

This represents staged connectivity testing:

```text
DNS
 ->
TCP
 ->
TLS
 ->
HTTP
```

---

# 117. Data Structure and Error Classification

```python
error_types = {
    400: "client_error",
    401: "unauthorized",
    403: "forbidden",
    404: "not_found",
    429: "rate_limited",
    500: "server_error",
    502: "bad_gateway",
    503: "unavailable"
}
```

Lookup:

```python
classification = error_types.get(
    status_code,
    "unknown"
)
```

---

# 118. Data Structure and Retry Statuses

```python
retryable_statuses = {
    429,
    500,
    502,
    503,
    504
}
```

Then:

```python
if status_code in retryable_statuses:
    retry()
```

Real retry logic must also consider request idempotency and server guidance.

---

# 119. Data Structure and Kubernetes Conditions

```python
conditions = [
    {
        "type": "Ready",
        "status": "True"
    },
    {
        "type": "Available",
        "status": "True"
    }
]
```

Find one:

```python
ready = next(
    (
        condition
        for condition in conditions
        if condition["type"]
        == "Ready"
    ),
    None
)
```

---

# 120. `next()` for Searches

```python
payment = next(
    (
        service
        for service in services
        if service["name"] == "payment"
    ),
    None
)
```

For repeated searches, build a dictionary index instead.

---

# 121. Duplicate Detection

Before building an index:

```python
ids = [
    item["id"]
    for item in items
]

if len(ids) != len(set(ids)):
    raise ValueError(
        "Duplicate identifiers"
    )
```

This prevents silent data loss from dictionary overwrites.

---

# 122. Data Structure Invariants

Useful production checks:

```text
Filtering should never add records.
Grouping should preserve record count.
Deduplication should never increase count.
Normalization should preserve identity.
Sorting should not change count.
```

These are useful unit-test properties.

---

# 123. Grouping Invariant

If each resource belongs to exactly one group:

```python
grouped_count = sum(
    len(group)
    for group in grouped.values()
)

assert grouped_count == len(resources)
```

This can catch accidental data loss.

---

# 124. Data Structure and Partial Failure

For batch operations, represent outcomes explicitly:

```python
result = {
    "succeeded": [],
    "failed": [],
    "skipped": []
}
```

This is clearer than mixing:

```python
True
None
{"error": ...}
```

in one list.

---

# 125. Data Structure and Result Contracts

For complex operations:

```python
from dataclasses import dataclass

@dataclass
class OperationResult:
    success: bool
    data: object | None = None
    error: str | None = None
```

This gives callers a consistent result model.

---

# 126. Data Structure and Function Boundaries

Document whether a function:

```text
reads input
mutates input
returns a new structure
changes external state
```

Prefer:

```python
def normalize(raw):
    return normalized
```

over silently modifying `raw`.

---

# 127. Data Structure and Memory

A large:

```python
list[dict]
```

can consume substantial memory.

For large AWS/Kubernetes/log datasets:

```text
pagination
streaming
generators
batching
incremental aggregation
bounded buffers
```

are preferred.

---

# 128. Avoid Raw + Normalized Copies

Bad:

```python
raw = get_all()
normalized = [
    normalize(x)
    for x in raw
]
```

If both remain in memory, memory usage can grow substantially.

Better:

```python
for raw in iter_resources():
    item = normalize(raw)
    process(item)
```

---

# 129. Avoid Unbounded Lists

Bad:

```python
events = []

while True:
    events.append(get_event())
```

This can grow indefinitely.

Use:

```python
deque(maxlen=1000)
```

if only recent history is needed, or stream to durable storage.

---

# 130. Data Structure and Caches

A cache must define:

```text
what is cached
TTL
invalidation
maximum size
refresh strategy
```

A plain dictionary is not automatically a production-ready cache.

---

# 131. Data Structure and Concurrency

When multiple workers process data, prefer:

```text
worker -> result
worker -> result
worker -> result
        |
        v
coordinator -> aggregate
```

rather than uncontrolled shared mutation.

For distributed processing use external coordination/storage.

---

# 132. Data Structure and Asyncio

```python
results = await asyncio.gather(
    check_service("user"),
    check_service("cart")
)
```

For many services, add bounded concurrency using a semaphore or worker pool.

---

# 133. Data Structure and Performance

Choose based on access pattern:

```text
ordered iteration -> list
membership        -> set
key lookup        -> dict
queue              -> deque
frequency          -> Counter
grouping           -> defaultdict
streaming          -> generator
```

This is more important than memorizing every Python method.

---

# 134. Data Structure Anti-Pattern — Everything as a List

Bad:

```python
services = [
    "payment",
    "cart"
]
```

and repeatedly scanning it for every lookup.

Better:

```python
services_by_name = {
    "payment": {...},
    "cart": {...}
}
```

when keyed access is the dominant requirement.

---

# 135. Anti-Pattern — Everything as a Dictionary

A dictionary is not ideal for:

```text
ordered event streams
simple unique membership
queue operations
fixed tuples
```

Choose based on the operation.

---

# 136. Anti-Pattern — Giant Nested Dictionary

Avoid one enormous object containing:

```text
AWS
Kubernetes
logs
metrics
deployments
incidents
configuration
```

if every function uses a different subset.

Use focused structures and clear contracts.

---

# 137. Anti-Pattern — Mixed Types

Bad:

```python
results = [
    True,
    {"error": "failed"},
    None
]
```

Prefer:

```python
OperationResult(...)
```

or a consistent dictionary structure.

---

# 138. Anti-Pattern — Duplicate Sources of Truth

Avoid independently maintaining:

```python
services_list
services_set
services_dict
```

Instead maintain one source and derive indexes:

```python
services_by_name = {
    service["name"]: service
    for service in services
}
```

---

# 139. Anti-Pattern — Unbounded API Data

Bad:

```python
all_resources.extend(
    every_page
)
```

for a huge account.

Prefer:

```text
page
 -> normalize
 -> process
 -> discard
```

unless a complete in-memory dataset is genuinely required.

---

# 140. Anti-Pattern — Logging Full Structures

Bad:

```python
logger.info(
    "config=%s",
    config
)
```

because nested secrets may leak.

Use redacted/safe fields.

---

# 141. Production Architecture — Python Inventory Tool

```text
AWS/Kubernetes API
        |
        v
Pagination
        |
        v
Normalization
        |
        v
Validation
        |
        v
Filtering
        |
        v
Index / Group
        |
        v
Aggregation
        |
        v
Report
```

This structure scales better than one giant function.

---

# 142. Production Architecture — Incident Tool

```text
Alert
  |
  v
identify service
  |
  +--> metrics
  +--> logs
  +--> Kubernetes
  +--> deployment
  |
  v
normalize evidence
  |
  v
correlate
  |
  v
incident report
```

Keep raw evidence bounded.

---

# 143. Production Architecture — Reconciliation

```text
desired state
      |
      v
normalize
      |
      +----------------+
                       |
current state          |
      |                |
      v                |
normalize              |
      |                |
      +------> diff <---+
                |
                v
          change plan
                |
                v
             dry-run
                |
                v
             approval
                |
                v
             execute
                |
                v
             verify
```

This is a strong production automation model.

---

# 144. Production Architecture — CI/CD

```text
CI parameters
      |
      v
parse
      |
      v
typed configuration
      |
      v
validate
      |
      v
security gates
      |
      v
deployment
      |
      v
verification
      |
      v
structured result
      |
      v
exit code
```

The Python utility should integrate with the pipeline's existing authorization and GitOps controls.

---

# 145. Production Security Checklist

```text
[ ] Validate external input
[ ] Never hardcode credentials
[ ] Never log secrets
[ ] Recursively redact sensitive structures
[ ] Use safe YAML loading
[ ] Avoid unsafe deserialization
[ ] Use IAM/RBAC
[ ] Keep authorization outside simple booleans
[ ] Use dry-run for destructive operations
[ ] Audit mutations
```

---

# 146. Production Performance Checklist

```text
[ ] Use pagination
[ ] Stream large files
[ ] Use generators
[ ] Avoid unnecessary copies
[ ] Build indexes for repeated lookups
[ ] Use sets for membership
[ ] Batch API operations
[ ] Bound concurrency
[ ] Bound queues/caches
[ ] Aggregate instead of retaining raw data
[ ] Measure before optimizing
```

---

# 147. Production Reliability Checklist

```text
[ ] Explicit timeouts
[ ] Bounded retries
[ ] Idempotent operations
[ ] Clear result structures
[ ] Verify mutations
[ ] Handle partial failures
[ ] Preserve useful error context
[ ] Return correct CI exit codes
[ ] Keep desired/current state separate
```

---

# 148. Interview — List vs Tuple

Strong answer:

> "Both are ordered sequences, but lists are mutable and tuples are immutable. I use lists for changing collections such as resource inventories and tuples for fixed values or composite keys."

---

# 149. Interview — List vs Set

Strong answer:

> "A list preserves ordering and allows duplicates. A set stores unique values and is optimized for membership checks. I use a set for resource/tag/permission membership and a list when order matters."

---

# 150. Interview — Dictionary vs List

Strong answer:

> "A list is good for sequential processing, while a dictionary is better for key-based lookup. If I repeatedly need a Kubernetes pod or AWS resource by ID, I build a dictionary index instead of scanning the list repeatedly."

---

# 151. Interview — Why Normalize API Responses?

Strong answer:

> "AWS and Kubernetes responses can be deeply nested and provider-specific. I normalize them into a smaller internal structure so business logic is simpler, testing is easier, and the rest of the code is less coupled to provider response formats."

---

# 152. Interview — How Process Millions of Records?

Strong answer:

> "I use pagination, generators, streaming, batching, bounded concurrency, and incremental aggregation. I avoid retaining complete raw API responses and transformed copies in memory."

---

# 153. Interview — How Remove Duplicates?

Strong answer:

> "If order does not matter, I use a set. If first-seen order matters, I use `dict.fromkeys()` or a `seen` set."

---

# 154. Interview — `defaultdict` and `Counter`

Strong answer:

> "`defaultdict` is useful for grouping, such as pods by namespace. `Counter` is useful for frequency analysis, such as error counts, pod states, or HTTP status classes."

---

# 155. Interview — Why `deque`?

Strong answer:

> "`deque` is designed for efficient insertion and removal from both ends. I use it for local queues and bounded recent-event buffers."

---

# 156. Interview — Generator

Strong answer:

> "A generator produces values lazily. It is useful for large AWS inventories, paginated APIs, and log processing because records can be processed incrementally without storing everything in memory."

---

# 157. Interview — Shallow vs Deep Copy

Strong answer:

> "A shallow copy creates a new outer container but nested mutable objects may still be shared. A deep copy recursively copies nested objects. I avoid unnecessary deep copies of large infrastructure data because of memory and CPU cost."

---

# 158. Interview — How Handle Missing API Fields?

Strong answer:

> "I distinguish required and optional fields. Required fields are validated explicitly, while optional fields can use `.get()`. I normalize the response before business logic."

---

# 159. Interview — How Compare Two Environments?

Strong answer:

> "I normalize both environments, use sets for membership differences, dictionaries for keyed configuration comparisons, and produce a structured diff instead of comparing raw API responses."

---

# 160. Interview — High Cardinality

Strong answer:

> "High cardinality means a dimension has many unique values. In observability, using request IDs, unique pod IDs, or timestamps as metric labels can create excessive time series. I use stable dimensions and normalize data before aggregation."

---

# 161. Interview — How Avoid Memory Growth?

Strong answer:

> "I stream data, use pagination and generators, process in batches, use bounded queues/caches, and aggregate incrementally. I avoid keeping raw responses, transformed copies, and unnecessary historical records."

---

# 162. Interview — Configuration Representation

Strong answer:

> "For dynamic API/configuration data I use dictionaries and lists. For stable contracts I may use dataclasses or TypedDict. I validate and normalize external data before passing it to operational logic."

---

# 163. Interview — Data Structures in Kubernetes

Strong answer:

> "Kubernetes API objects are usually nested dictionaries/lists when represented as JSON. I extract stable fields such as name, namespace, phase, readiness, labels, and conditions into a normalized structure before running health or policy logic."

---

# 164. Interview — Data Structures in AWS

Strong answer:

> "AWS SDK responses commonly contain nested dictionaries and lists. I use paginators, iterate through pages, normalize only required fields, and process resources incrementally when the account is large."

---

# 165. Interview — Data Structures in CI/CD

Strong answer:

> "I normalize pipeline inputs such as environment, image tag, commit, and branch into a configuration object, validate types and allowed values, then pass that object through the deployment workflow."

---

# 166. Interview — Data Structures in DevSecOps

Strong answer:

> "I can represent security gate results as a dictionary and use `all()` to determine whether required gates passed. I keep detailed results for reporting but fail the pipeline with a non-zero exit code when mandatory gates fail."

---

# 167. Scenario — AWS Script Out of Memory

Question:

> A script works for 500 resources but crashes for 100,000.

Answer:

```text
Use paginator
  |
  v
Process page
  |
  v
Normalize only required fields
  |
  v
Filter early
  |
  v
Aggregate incrementally
  |
  v
Discard page
```

Use an index only if repeated lookup genuinely requires it.

---

# 168. Scenario — Kubernetes Script Too Slow

Investigate:

```text
repeated API calls
repeated list scans
unnecessary log collection
serial external calls
large response parsing
```

Possible fixes:

```text
normalize once
build indexes
filter early
bound concurrency
collect expensive diagnostics only for failures
```

---

# 169. Scenario — Duplicate Resources

Check:

```text
pagination
multiple regions
multiple accounts
duplicate API discovery paths
resource identity
```

For multi-account/multi-region inventories, a composite key may be:

```python
key = (
    account_id,
    region,
    resource_id
)
```

---

# 170. Scenario — Secret Leaked Through Config

Root causes:

```text
print(config)
debug dump
logger.info(config)
exception contains secret
HTTP payload logging
```

Fix:

```text
redaction
safe logging
secret rotation
restricted debugging
```

---

# 171. Scenario — API Schema Changes

Use:

```text
provider API
    |
    v
adapter/normalizer
    |
    v
stable internal model
    |
    v
business logic
```

Then API changes are isolated to the adapter.

---

# 172. Scenario — Alert Storm

Group:

```python
alerts_by_service = defaultdict(list)

for alert in alerts:
    alerts_by_service[
        alert["service"]
    ].append(alert)
```

Then correlate:

```text
high CPU
high latency
high errors
restarts
deployment changes
```

into a service-level incident.

---

# 173. Scenario — Need Latest 100 Events

Use:

```python
recent = deque(
    maxlen=100
)
```

This avoids an unbounded list.

---

# 174. Scenario — Need Top 5 Error Services

```python
from collections import Counter

counts = Counter(
    log["service"]
    for log in logs
    if log["level"] == "ERROR"
)

top_five = counts.most_common(5)
```

At very large scale, aggregate in ELK or the log backend where possible.

---

# 175. Scenario — Need Fast Lookup

Input:

```python
resources = [...]
```

Build:

```python
by_id = {
    resource["id"]: resource
    for resource in resources
}
```

Validate duplicate IDs first when uniqueness is expected.

---

# 176. Scenario — Need Drift Detection

Use:

```python
desired - actual
```

for missing resources and:

```python
actual - desired
```

for unexpected resources.

For full configuration drift, use dictionaries and structured diffs.

---

# 177. Scenario — Need Permission Comparison

Use:

```python
missing = required - granted
extra = granted - required
```

Sets are ideal because ordering is irrelevant and uniqueness matters.

---

# 178. Scenario — Production Cleanup

Represent candidate:

```python
candidate = {
    "id": "i-123",
    "environment": "dev",
    "protected": False,
    "managed_by": "terraform"
}
```

Flow:

```text
discover
  |
  v
validate
  |
  v
classify
  |
  v
dry-run
  |
  v
approval
  |
  v
action
  |
  v
verify
  |
  v
audit
```

Never make discovery itself destructive.

---

# 179. Scenario — Deployment Verification

Represent:

```python
deployment = {
    "desired_replicas": 3,
    "ready_replicas": 2,
    "error_rate": 0.08
}
```

A verification function can evaluate:

```text
replica readiness
error rate
latency
application health
```

before declaring success.

---

# 180. Scenario — Partial Batch Failure

Represent:

```python
result = {
    "succeeded": [...],
    "failed": [...],
    "skipped": [...]
}
```

Then decide:

```text
continue?
retry?
rollback?
alert?
```

based on the operation's semantics.

---

# 181. Practical Exercise 1

Create a service list.

Tasks:

```text
add inventory
remove cart
print first
print last
sort
membership check
```

---

# 182. Practical Exercise 2

Create a list of service dictionaries:

```python
[
    {"name": "user", "port": 8080},
    {"name": "payment", "port": 8081}
]
```

Tasks:

```text
extract names
create name -> port index
find payment
sort by port
```

---

# 183. Practical Exercise 3

Create 10 pod dictionaries with:

```text
name
namespace
phase
restart_count
```

Tasks:

```text
find unhealthy pods
find highest restart pod
group by namespace
count phases
create pod index
```

---

# 184. Practical Exercise 4

Create AWS-style instance dictionaries.

Tasks:

```text
filter production
count states
group by region
find stopped instances
index by instance ID
```

---

# 185. Practical Exercise 5

Create desired and actual service sets.

Calculate:

```text
missing
extra
common
```

Generate a drift report.

---

# 186. Practical Exercise 6

Create resources with cloud tags.

Required:

```text
Environment
Owner
Application
```

Find resources with missing tags.

---

# 187. Practical Exercise 7

Create log records:

```text
timestamp
service
level
message
```

Calculate:

```text
errors per service
top errors
affected services
```

Use:

```text
Counter
defaultdict
set
```

---

# 188. Practical Exercise 8

Create an incident structure containing:

```text
alerts
metrics
logs
events
deployment
```

Generate:

```text
severity
affected services
top errors
current health
```

---

# 189. Practical Exercise 9

Process a simulated million-line log without storing all lines.

Use:

```text
streaming
Counter
deque(maxlen=100)
```

Report:

```text
top errors
latest events
error count
```

---

# 190. Practical Exercise 10

Create:

```text
defaults
environment
service
runtime
```

Merge them in precedence order and test nested configuration behavior.

---

# 191. Practical Exercise 11

Create 10,000 resources and compare:

```text
list scan
dictionary index lookup
```

Measure the difference.

---

# 192. Practical Exercise 12

Create:

```python
required_permissions
granted_permissions
```

Calculate:

```text
missing
extra
```

using set operations.

---

# 193. Practical Exercise 13

Create resource cost records and calculate:

```text
total
production total
highest cost
top 5
```

---

# 194. Practical Exercise 14

Create a deployment matrix using tuple keys:

```python
(environment, service)
```

Find version differences between stage and production.

---

# 195. Practical Exercise 15

Implement a local queue with:

```python
deque
```

Tasks:

```text
enqueue
dequeue
retry
bounded queue
```

---

# 196. Practical Exercise 16

Implement a recent-event buffer:

```python
deque(maxlen=100)
```

Then generate an incident snapshot.

---

# 197. Practical Exercise 17

Validate a configuration containing:

```text
environment
region
cluster
services
replicas
```

Return all validation errors.

---

# 198. Practical Exercise 18

Create a fake AWS response and implement:

```python
normalize_instance()
```

Return:

```text
id
state
environment
```

Test missing fields.

---

# 199. Practical Exercise 19

Create a fake Kubernetes pod response and normalize:

```text
name
namespace
phase
ready
restart_count
```

Then run health checks.

---

# 200. Practical Exercise 20 — End-to-End Inventory Tool

Build:

```text
load config
    |
    v
fetch resources
    |
    v
paginate
    |
    v
normalize
    |
    v
validate
    |
    v
index/group
    |
    v
aggregate
    |
    v
report
```

This exercise combines the core Python data-structure skills needed for DevOps automation.

---

# 201. Production Mental Model

Remember:

```text
LIST
  -> ordered mutable data

TUPLE
  -> fixed immutable data

SET
  -> uniqueness/membership

DICT
  -> keyed lookup/state

DEFAULTDICT
  -> grouping

COUNTER
  -> frequency aggregation

DEQUE
  -> queues/bounded recent history

GENERATOR
  -> lazy large-data processing
```

---

# 202. Final DevOps Pattern

```text
External data
      |
      v
Validate
      |
      v
Normalize
      |
      v
Choose structure
      |
      v
Process efficiently
      |
      v
Aggregate
      |
      v
Verify
      |
      v
Report safely
```

The correct data structure improves:

```text
correctness
performance
memory usage
maintainability
testability
operational safety
```

---

# 203. Final Interview Takeaway

Do not answer Python questions only with syntax.

Connect the structure to production:

> "For an EKS inventory, I would process paginated API results incrementally, normalize each resource into a small dictionary, use a set for membership/drift checks, build a dictionary index for repeated resource-ID lookups, use `Counter` for state aggregation, and avoid retaining raw API responses unnecessarily."

That demonstrates actual DevOps engineering judgment.

---