# Variables-and-Data-Types

> Deep Python data-model notes for DevOps engineers, focused on configuration, APIs, AWS/Kubernetes automation, CI/CD, validation, troubleshooting, production reliability, and interviews.

---

# 1. Why Variables and Data Types Matter in DevOps

Almost every automation script moves data through:

```text
Environment / File / API / CLI
          |
          v
      Python value
          |
          v
       Validation
          |
          v
     Transformation
          |
          v
       Automation
          |
          v
        Result
```

Typical examples:

```text
Environment variable -> str
Replica count         -> int
CPU percentage        -> float
Health status         -> bool
Missing API field     -> None
AWS API response      -> dict/list
Kubernetes data       -> nested structures
```

Understanding types prevents subtle production failures.

---

# 2. What Is a Variable?

A variable is a name that refers to an object.

```python
environment = "production"
replicas = 3
```

Think:

```text
environment -> "production"
replicas    -> 3
```

Python does not require a separate type declaration.

---

# 3. Dynamic and Strong Typing

Python is dynamically typed:

```python
value = 100
value = "production"
value = True
```

It is also strongly typed.

This fails:

```python
message = "servers: "
count = 3

print(message + count)
```

because Python does not silently convert the integer to a string.

Prefer:

```python
print(f"{message}{count}")
```

Dynamic typing is convenient, but production automation should still validate external data.

---

# 4. Variable Naming

Good:

```python
cluster_name = "prod-eks"
namespace = "payments"
replica_count = 5
cpu_threshold = 80
```

Avoid unclear names:

```python
x = "prod-eks"
r = 5
```

Use `snake_case` for variables and functions:

```python
deployment_status
retry_count
check_health()
```

Constants conventionally use uppercase:

```python
DEFAULT_TIMEOUT = 10
MAX_RETRIES = 3
CPU_THRESHOLD = 80
```

---

# 5. Assignment and Reassignment

```python
replicas = 3
replicas = 5
```

The name now refers to the new value.

For operational code, meaningful names are better:

```python
desired_replicas = 5
current_replicas = 3
```

This makes troubleshooting easier.

Multiple assignment:

```python
environment, region = "production", "ap-south-1"
```

Use it only when readability improves.

---

# 6. Type Inspection

Use:

```python
type(value)
```

Example:

```python
replicas = 3
print(type(replicas))
```

Use `isinstance()` for validation:

```python
if isinstance(replicas, int):
    print("Valid integer")
```

Multiple accepted types:

```python
if isinstance(value, (int, float)):
    print("Numeric value")
```

---

# 7. Important Python Data Types

Common types:

```text
str
int
float
bool
NoneType
list
tuple
set
dict
range
bytes
```

The most important for DevOps are:

```text
str
int
float
bool
None
list
dict
tuple
set
```

---

# 8. Strings

Strings represent text:

```python
service_name = "payment"
environment = "production"
```

Both are valid:

```python
'payment'
"payment"
```

Useful methods:

```python
text.lower()
text.upper()
text.strip()
text.replace()
text.split()
text.startswith()
text.endswith()
```

---

# 9. String Length, Indexing and Slicing

Length:

```python
service = "payment"
print(len(service))
```

Indexing:

```python
print(service[0])
print(service[-1])
```

Slicing:

```python
print(service[0:4])
```

Remember:

```text
Index starts at 0
Ending slice index is excluded
```

---

# 10. String Cleaning

Very common when processing CLI output or files:

```python
status = " RUNNING "

status = status.strip().lower()

print(status)
```

Result:

```text
running
```

Other methods:

```python
lstrip()
rstrip()
```

---

# 11. split() and join()

Split:

```python
services = "user,payment,order"

service_list = services.split(",")
```

Result:

```python
["user", "payment", "order"]
```

Join:

```python
result = ",".join(service_list)
```

Result:

```text
user,payment,order
```

Useful for environment variables, CLI output, configuration, and reports.

---

# 12. f-Strings

Preferred formatting:

```python
service = "payment"
replicas = 3

message = f"{service} has {replicas} replicas"
print(message)
```

Format numbers:

```python
cpu = 72.5678
print(f"CPU: {cpu:.2f}%")
```

f-strings are useful for logs, reports, alerts, and CLI output.

---

# 13. Integers

Integers represent whole numbers:

```python
replicas = 5
port = 8080
retry_count = 3
```

Common DevOps uses:

```text
Replica counts
Ports
Retries
Timeout values
HTTP status codes
Resource counts
```

Arithmetic:

```python
total = 10
failed = 2
successful = total - failed
```

---

# 14. Float

Floats represent decimal values:

```python
cpu_usage = 72.5
memory_usage = 81.3
latency_ms = 125.4
```

Used for:

```text
CPU percentage
Memory percentage
Latency
Ratios
Throughput
```

Be aware that floating-point arithmetic can have precision limitations.

---

# 15. Boolean

Boolean values:

```python
True
False
```

Example:

```python
deployment_healthy = True

if deployment_healthy:
    print("Deployment is healthy")
```

Comparisons also produce booleans:

```python
cpu = 85
is_high = cpu > 80
```

---

# 16. None

`None` represents the absence of a value.

```python
error_message = None
```

Typical API example:

```python
private_ip = response.get("private_ip")

if private_ip is None:
    print("Private IP unavailable")
```

Use:

```python
is None
is not None
```

for `None` checks.

---

# 17. Truthy and Falsy Values

Common falsy values:

```text
False
None
0
0.0
""
[]
{}
set()
```

Example:

```python
services = []

if not services:
    print("No services found")
```

Be careful: an empty collection and a missing value can have different operational meanings.

---

# 18. Lists

Lists are ordered and mutable:

```python
pods = [
    "payment-1",
    "payment-2",
    "payment-3"
]
```

Access:

```python
pods[0]
```

Common operations:

```python
pods.append("payment-4")
pods.remove("payment-4")
len(pods)
```

Lists are ideal for collections such as servers, pods, services, namespaces, files, and regions.

---

# 19. List Slicing and Mutation

```python
pods = ["p1", "p2", "p3", "p4"]

print(pods[1:3])
```

Copy:

```python
copy = pods.copy()
```

Sort:

```python
pods.sort()
```

Reverse:

```python
pods.reverse()
```

Remember that these operations can mutate the original list.

---

# 20. Tuples

Tuples are ordered collections generally treated as immutable:

```python
server = ("web-01", "10.0.1.10")
```

Access:

```python
name = server[0]
ip = server[1]
```

Tuple unpacking:

```python
name, ip = server
```

Useful for fixed groups of values and function return values.

---

# 21. Sets

Sets contain unique values:

```python
regions = {
    "ap-south-1",
    "us-east-1",
    "ap-south-1"
}
```

The duplicate is removed.

Operations:

```python
a | b   # union
a & b   # intersection
a - b   # difference
```

Useful for unique IPs, services, error codes, and namespaces.

---

# 22. Dictionaries

Dictionaries map keys to values:

```python
instance = {
    "id": "i-123",
    "state": "running",
    "region": "ap-south-1"
}
```

Access:

```python
instance["state"]
```

Safer lookup:

```python
instance.get("state")
```

Dictionaries are fundamental because JSON objects and many API responses naturally map to Python dictionaries.

---

# 23. Dictionary Operations

Update:

```python
instance["state"] = "stopped"
```

Add:

```python
instance["environment"] = "production"
```

Safe default:

```python
status = instance.get("status", "unknown")
```

Iterate:

```python
for key, value in instance.items():
    print(key, value)
```

Other views:

```python
instance.keys()
instance.values()
instance.items()
```

---

# 24. Nested Dictionaries

Example:

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
name = deployment["metadata"]["name"]
replicas = deployment["spec"]["replicas"]
```

This pattern appears frequently in structured infrastructure data.

---

# 25. List of Dictionaries — Critical DevOps Pattern

Example:

```python
instances = [
    {"id": "i-001", "state": "running"},
    {"id": "i-002", "state": "stopped"}
]
```

Process:

```python
for instance in instances:
    print(instance["id"], instance["state"])
```

This resembles AWS inventory and many REST API responses.

---

# 26. JSON to Python Type Mapping

JSON:

```text
object
array
string
number
true
false
null
```

Python:

```text
dict
list
str
int/float
True
False
None
```

Example JSON:

```json
{
  "service": "payment",
  "replicas": 3,
  "healthy": true,
  "error": null
}
```

Python representation:

```python
{
    "service": "payment",
    "replicas": 3,
    "healthy": True,
    "error": None
}
```

---

# 27. json.loads() and json.dumps()

Parse JSON text:

```python
import json

text = '{"replicas": 3, "healthy": true}'

data = json.loads(text)

print(data["replicas"])
```

Convert Python data to JSON:

```python
data = {
    "service": "payment",
    "replicas": 3
}

text = json.dumps(data, indent=2)
print(text)
```

Remember:

```text
loads -> string to Python
dumps -> Python to string
load  -> file to Python
dump  -> Python to file
```

---

# 28. YAML and Python Types

YAML commonly maps to:

```text
mapping  -> dict
sequence -> list
string   -> str
number   -> int/float
boolean  -> bool
null     -> None
```

Example:

```yaml
replicas: 3
enabled: true
service: payment
```

Conceptual Python:

```python
{
    "replicas": 3,
    "enabled": True,
    "service": "payment"
}
```

Use safe YAML loading for untrusted input.

---

# 29. Mutable vs Immutable

Common immutable types:

```text
str
int
float
bool
tuple
```

Common mutable types:

```text
list
dict
set
```

Mutable objects can change in place. Immutable objects cannot.

This matters when the same object is referenced by multiple parts of a script.

---

# 30. Assignment Does Not Copy

Example:

```python
config1 = {"replicas": 3}
config2 = config1

config2["replicas"] = 5
```

Now:

```python
print(config1["replicas"])
```

returns:

```text
5
```

Both variables refer to the same dictionary.

---

# 31. Shallow and Deep Copy

Shallow copy:

```python
config2 = config1.copy()
```

For nested mutable data:

```python
import copy

config2 = copy.deepcopy(config1)
```

Use copying deliberately. Deep copying large structures can consume unnecessary memory.

---

# 32. == vs is

`==` compares values:

```python
a = "production"
b = "production"

print(a == b)
```

`is` checks object identity.

Correct use:

```python
if value is None:
    ...
```

Do not normally use `is` to compare strings or numbers.

---

# 33. Type Conversion

String to integer:

```python
replicas = int("5")
```

Integer to string:

```python
port_text = str(8080)
```

String to float:

```python
cpu = float("75.5")
```

Conversion is especially important because environment variables are strings.

---

# 34. Safe Integer Conversion

External input can be invalid:

```python
value = "five"
```

Handle it:

```python
try:
    replicas = int(value)
except ValueError:
    raise ValueError("Replica count must be an integer")
```

Do not silently replace invalid production configuration with an arbitrary default unless that behavior is intentional.

---

# 35. Float Validation

```python
threshold = float("80.0")

if not 0 <= threshold <= 100:
    raise ValueError("Threshold must be between 0 and 100")
```

This pattern is useful for:

```text
CPU thresholds
Memory thresholds
Latency limits
Error percentages
```

---

# 36. Boolean Configuration Pitfall

This surprises many beginners:

```python
bool("false")
```

returns:

```text
True
```

because a non-empty string is truthy.

Do not parse environment booleans with plain `bool()`.

---

# 37. Safe Boolean Parsing

Example:

```python
value = "false"

enabled = value.strip().lower() in {
    "true",
    "1",
    "yes",
    "on"
}
```

For strict production configuration, explicitly validate allowed values and reject unknown values such as:

```text
maybe
unknown
sometimes
```

---

# 38. Type Hints

Python supports type hints:

```python
def check_cpu(cpu: float) -> bool:
    return cpu <= 80
```

Variables:

```python
environment: str = "production"
replicas: int = 3
healthy: bool = True
cpu: float = 75.5
```

Type hints improve readability, IDE support, static analysis, and maintenance.

They do not automatically enforce types at runtime.

---

# 39. Collection Type Hints

Modern Python:

```python
services: list[str] = [
    "user",
    "payment",
    "order"
]
```

Dictionary:

```python
config: dict[str, str] = {
    "environment": "production"
}
```

For mixed or complex structures, use a more appropriate model rather than forcing an inaccurate annotation.

---

# 40. Optional Values

A value can explicitly be absent:

```python
from typing import Optional

private_ip: Optional[str] = None
```

Modern Python:

```python
private_ip: str | None = None
```

This is useful for optional API fields and configuration.

---

# 41. Type Hints Are Not Runtime Validation

This:

```python
replicas: int = "three"
```

can still execute.

Type hints communicate intent; they do not automatically validate runtime input.

Reliable automation needs:

```text
Type hints
+
Validation
+
Tests
```

---

# 42. Enum for Fixed States

For fixed operational states:

```python
from enum import Enum

class DeploymentStatus(Enum):
    SUCCESS = "success"
    FAILED = "failed"
    UNKNOWN = "unknown"
```

Use:

```python
if status == DeploymentStatus.FAILED:
    print("Deployment failed")
```

Enums can make state-heavy automation clearer.

---

# 43. Dataclasses

For known internal structures:

```python
from dataclasses import dataclass

@dataclass
class Server:
    name: str
    ip: str
    environment: str
```

Create:

```python
server = Server(
    name="web-01",
    ip="10.0.1.10",
    environment="production"
)
```

Dataclasses are useful when dictionaries become difficult to manage.

---

# 44. Dictionaries vs Dataclasses

Use dictionaries when:

```text
Data comes directly from JSON/API
Schema is dynamic
Generic key/value access is useful
```

Use dataclasses when:

```text
Internal structure is known
Fields should be explicit
Typed objects improve maintainability
```

Do not introduce classes just to make a tiny script look complicated.

---

# 45. List Comprehensions

Example:

```python
services = ["user", "payment", "order"]

upper_names = [
    service.upper()
    for service in services
]
```

Filtering:

```python
running = [
    instance
    for instance in instances
    if instance["state"] == "running"
]
```

Use comprehensions when they remain readable.

---

# 46. Dictionary Comprehensions

```python
services = ["user", "payment", "order"]

service_status = {
    service: "unknown"
    for service in services
}
```

Useful for building lookup maps.

---

# 47. Sorting Resource Data

```python
servers = [
    {"name": "web-02", "cpu": 80},
    {"name": "web-01", "cpu": 65}
]
```

Sort:

```python
servers.sort(
    key=lambda server: server["cpu"]
)
```

This is useful when generating infrastructure reports.

---

# 48. Filtering Resources

```python
high_cpu = [
    server
    for server in servers
    if server["cpu"] > 80
]
```

Production pattern:

```text
AWS inventory
    |
    v
Collect resource data
    |
    v
Filter by environment
    |
    v
Filter by threshold
    |
    v
Report/action
```

---

# 49. Common Exceptions With Data Types

KeyError:

```python
server["missing"]
```

IndexError:

```python
servers = []
servers[0]
```

TypeError:

```python
"3" + 1
```

ValueError:

```python
int("abc")
```

AttributeError:

```python
value = None
value.get("status")
```

Understanding the exception often points directly to the data problem.

---

# 50. Configuration Validation Pattern

```python
def validate_config(config):
    if not isinstance(config.get("environment"), str):
        raise ValueError("environment must be a string")

    replicas = config.get("replicas")

    if not isinstance(replicas, int):
        raise ValueError("replicas must be an integer")

    if replicas < 1:
        raise ValueError("replicas must be >= 1")
```

The important sequence is:

```text
Receive
 |
Validate type
 |
Validate value
 |
Validate business rule
 |
Perform action
```

---

# 51. Environment Variable Example

```python
import os

environment = os.getenv("ENVIRONMENT", "dev")
region = os.getenv("AWS_REGION", "ap-south-1")
replicas_raw = os.getenv("REPLICAS", "3")

try:
    replicas = int(replicas_raw)
except ValueError:
    raise ValueError("REPLICAS must be an integer")

if replicas < 1:
    raise ValueError("REPLICAS must be at least 1")

print(
    f"Environment={environment}, "
    f"Region={region}, "
    f"Replicas={replicas}"
)
```

This is directly applicable to CI/CD jobs and containers.

---

# 52. Kubernetes Data Example

Conceptual structured data:

```python
pod = {
    "metadata": {
        "name": "payment-7c8d"
    },
    "status": {
        "phase": "Running"
    }
}
```

Access:

```python
pod_name = pod["metadata"]["name"]
phase = pod["status"]["phase"]

if phase != "Running":
    print(f"{pod_name} is not running")
```

Real Kubernetes clients often expose objects rather than raw dictionaries, but the same data concepts apply.

---

# 53. AWS Data Example

Conceptual EC2 response:

```python
instance = {
    "InstanceId": "i-123456",
    "State": {
        "Name": "running"
    },
    "Tags": [
        {
            "Key": "Environment",
            "Value": "production"
        }
    ]
}
```

Access:

```python
instance_id = instance["InstanceId"]
state = instance["State"]["Name"]
```

---

# 54. Safely Reading AWS Tags

```python
tags = instance.get("Tags", [])

for tag in tags:
    if tag.get("Key") == "Environment":
        environment = tag.get("Value")
```

If the environment tag is required for an enterprise workflow, validate its presence rather than silently assuming a default.

---

# 55. Data Transformation

Input:

```python
instances = [
    {"id": "i-1", "state": "running"},
    {"id": "i-2", "state": "stopped"}
]
```

Extract IDs:

```python
instance_ids = [
    instance["id"]
    for instance in instances
]
```

Result:

```python
["i-1", "i-2"]
```

This pattern is common when one API response becomes another API request's input.

---

# 56. Bytes vs Strings

Subprocess and network operations can produce bytes.

Prefer:

```python
subprocess.run(
    command,
    capture_output=True,
    text=True
)
```

Then stdout/stderr are strings.

Without text mode, you may need to decode bytes explicitly.

---

# 57. Encoding

Use explicit encoding for files:

```python
with open("config.txt", encoding="utf-8") as file:
    content = file.read()
```

UTF-8 is the common default for modern text processing.

Explicit encoding improves portability and predictability.

---

# 58. Date and Time Types

DevOps automation often processes:

```text
Deployment timestamps
Log timestamps
Certificate expiry
Backup age
Resource age
Retention periods
```

Use timezone-aware timestamps:

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
print(now)
```

Avoid ambiguous naive timestamps in distributed production systems.

---

# 59. Timedelta

Calculate age:

```python
from datetime import datetime, timedelta, timezone

now = datetime.now(timezone.utc)
old_time = now - timedelta(days=7)
```

Useful for:

```text
Files older than 7 days
Stale resources
Retention checks
Certificate monitoring
Backup policies
```

---

# 60. Explicit Units

Avoid:

```python
timeout = 10
```

Prefer:

```python
timeout_seconds = 10
```

Other examples:

```python
memory_bytes
latency_ms
retention_days
retry_count
```

Unit-aware names prevent operational mistakes.

---

# 61. Constants for Thresholds

Instead of:

```python
if cpu > 80:
```

define:

```python
CPU_WARNING_THRESHOLD = 80
```

Then:

```python
if cpu > CPU_WARNING_THRESHOLD:
    ...
```

Benefits:

```text
Readability
Consistency
Easy changes
Testing
```

---

# 62. Avoid Global Mutable State

Avoid:

```python
servers = []

def add_server(server):
    servers.append(server)
```

Global mutable state makes automation harder to reason about and test.

Prefer:

```python
def add_server(servers, server):
    servers.append(server)
```

Or return a new result when that better fits the design.

---

# 63. Data Validation Before Action

Production pattern:

```text
Input
 |
 v
Type validation
 |
 v
Value/range validation
 |
 v
Business validation
 |
 v
Action
```

Example:

```python
if not isinstance(replicas, int):
    raise ValueError("replicas must be int")

if replicas < 1:
    raise ValueError("replicas must be >= 1")

if environment not in {"dev", "stage", "prod"}:
    raise ValueError("invalid environment")
```

---

# 64. Production Configuration Example

```python
config = {
    "environment": "production",
    "replicas": 5,
    "cpu_threshold": 80.0,
    "enabled": True
}
```

Validate:

```python
if config["environment"] not in {"dev", "stage", "prod"}:
    raise ValueError("Invalid environment")

if not isinstance(config["replicas"], int):
    raise ValueError("replicas must be integer")

if config["replicas"] < 1:
    raise ValueError("replicas must be >= 1")

if not 0 <= config["cpu_threshold"] <= 100:
    raise ValueError("cpu threshold must be 0-100")

if not isinstance(config["enabled"], bool):
    raise ValueError("enabled must be boolean")
```

---

# 65. Production Health Result

```python
health = {
    "service": "payment",
    "healthy": True,
    "latency_ms": 125.4,
    "status_code": 200,
    "errors": 0
}
```

Then:

```python
if not health["healthy"]:
    print(f"{health['service']} is unhealthy")
```

This kind of structured result can later be sent to monitoring or reporting systems.

---

# 66. Production Resource Inventory

```python
resources = [
    {
        "id": "i-001",
        "type": "ec2",
        "environment": "production",
        "state": "running"
    },
    {
        "id": "i-002",
        "type": "ec2",
        "environment": "development",
        "state": "stopped"
    }
]
```

Filter:

```python
production = [
    resource
    for resource in resources
    if resource["environment"] == "production"
]
```

Tag-based filtering is a common enterprise automation pattern.

---

# 67. Production Error Classification

```python
errors = [
    {"code": 500, "service": "payment"},
    {"code": 404, "service": "catalog"},
    {"code": 500, "service": "payment"}
]
```

Unique codes:

```python
codes = {error["code"] for error in errors}
```

Server errors:

```python
server_errors = [
    error
    for error in errors
    if error["code"] >= 500
]
```

---

# 68. Common Mistake — Comparing Different Types

Bad:

```python
replicas = 3

if replicas == "3":
    print("Three replicas")
```

`3` and `"3"` are different values/types.

Correct:

```python
if replicas == 3:
    ...
```

Or convert external input first.

---

# 69. Common Mistake — String Boolean

Bad:

```python
enabled = "false"

if enabled:
    print("Enabled")
```

This executes because a non-empty string is truthy.

Parse the value explicitly into a real boolean.

---

# 70. Common Mistake — Mutable Default Argument

Avoid:

```python
def collect_errors(errors=[]):
    errors.append("error")
    return errors
```

The same default list can persist between calls.

Use:

```python
def collect_errors(errors=None):
    if errors is None:
        errors = []

    errors.append("error")
    return errors
```

This is a frequent Python interview question.

---

# 71. Common Mistake — Modifying a List While Iterating

Avoid:

```python
for server in servers:
    if server["state"] == "stopped":
        servers.remove(server)
```

Prefer:

```python
servers = [
    server
    for server in servers
    if server["state"] != "stopped"
]
```

This avoids skipped elements.

---

# 72. Common Mistake — Silent Defaults

Potentially dangerous:

```python
replicas = config.get("replicas", 1)
```

If `replicas` is required, silently defaulting may hide a deployment configuration error.

Defaults should be intentional, documented, and safe.

---

# 73. Common Mistake — Assuming API Fields Exist

Risky:

```python
status = response["status"]["state"]
```

For required fields, validate:

```python
if "status" not in response:
    raise ValueError("Missing status in API response")
```

For optional fields, use safe access and an intentional default.

---

# 74. Interview — What Is Dynamic Typing?

Strong answer:

> Python is dynamically typed, so I don't need to declare a variable's type before assigning a value. A variable can later reference another type. Python is also strongly typed, so incompatible operations such as adding a string and an integer do not silently convert the values.

---

# 75. Interview — Mutable vs Immutable

Strong answer:

> Mutable objects such as lists, dictionaries and sets can be changed in place. Immutable objects such as strings, integers and tuples cannot be changed in place. This matters in automation because shared mutable configuration can be modified unexpectedly by another part of a script.

---

# 76. Interview — List vs Tuple

Strong answer:

> Both are ordered collections, but lists are mutable while tuples are generally immutable. I use lists when the collection needs to change and tuples for fixed groups of values or when immutability is useful.

---

# 77. Interview — List vs Set

Strong answer:

> A list preserves order and allows duplicates. A set stores unique values and is useful when uniqueness and membership checks matter, such as finding unique IP addresses or error codes.

---

# 78. Interview — Dictionary vs List

Strong answer:

> A list is useful for an ordered collection of items, while a dictionary maps keys to values. In DevOps, dictionaries are especially useful for structured resource data and JSON API responses, while lists commonly contain multiple resource dictionaries.

---

# 79. Interview — == vs is

Strong answer:

> `==` compares values, while `is` checks object identity. I use `is None` when checking for `None`, but I don't normally use `is` to compare strings or numbers.

---

# 80. Interview — Why Are Environment Variables Strings?

Operating-system environment variables are exposed as text values.

Therefore:

```python
replicas = int(os.getenv("REPLICAS", "3"))
```

is required when the application needs a numeric value.

Boolean environment variables also require explicit parsing.

---

# 81. Interview — How Do You Validate Configuration?

Strong answer:

```text
Check required fields
Check Python types
Check allowed values
Check numeric ranges
Check relationships between fields
Reject invalid configuration before changes
```

Example:

```python
if environment not in {"dev", "stage", "prod"}:
    raise ValueError("Invalid environment")

if not isinstance(replicas, int) or replicas < 1:
    raise ValueError("Invalid replica count")
```

---

# 82. Interview — What Is a Type Hint?

Strong answer:

> A type hint documents the expected type and helps IDEs, static analyzers and maintainers. It does not automatically enforce the type at runtime.

Example:

```python
def check_health(cpu: float) -> bool:
    return cpu <= 80
```

---

# 83. Interview — Why Are Dictionaries Important in DevOps?

Strong answer:

> Many DevOps systems expose structured JSON data. JSON objects naturally map to Python dictionaries and JSON arrays map to lists. AWS APIs, REST APIs and automation workflows therefore require strong understanding of nested dictionaries and lists.

---

# 84. Interview — How Do You Handle Missing API Fields?

Strong answer:

> First I determine whether the field is required or optional. For optional fields I can use `get()` with an intentional default. For required fields I validate the response and fail clearly rather than silently continuing with incorrect data.

---

# 85. Interview — TypeError vs ValueError

```text
TypeError
    -> inappropriate object/type for an operation

ValueError
    -> correct general type, invalid value
```

Examples:

```python
"3" + 1
```

causes `TypeError`.

```python
int("abc")
```

causes `ValueError`.

---

# 86. Interview — Why Is bool("false") True?

Because `"false"` is a non-empty string and non-empty strings are truthy.

Therefore:

```python
bool("false")
```

returns:

```text
True
```

Environment variables representing booleans must be explicitly parsed.

---

# 87. Interview — How Would You Process AWS Inventory?

Strong workflow:

```text
Call AWS API
 |
 v
Receive list/dictionary data
 |
 v
Validate response
 |
 v
Extract fields
 |
 v
Normalize data
 |
 v
Filter by tags/environment
 |
 v
Generate report/action
```

This directly uses Python's list and dictionary capabilities.

---

# 88. Interview — How Would You Process Kubernetes Data?

Conceptually:

```text
Kubernetes client
 |
 v
Resource objects
 |
 v
Extract metadata/status
 |
 v
Validate state
 |
 v
Filter unhealthy resources
 |
 v
Log/report/remediate
```

The exact Python client representation may be object-based rather than raw dictionaries, but the same data-model reasoning applies.

---

# 89. Practical Exercise 1 — Environment Configuration

Read:

```text
ENVIRONMENT
AWS_REGION
REPLICAS
CPU_THRESHOLD
```

Requirements:

```text
ENVIRONMENT -> string
AWS_REGION -> string
REPLICAS -> integer >= 1
CPU_THRESHOLD -> float from 0 to 100
```

Reject invalid values.

Expected:

```text
Environment: production
Region: ap-south-1
Replicas: 5
CPU threshold: 80.0
```

---

# 90. Practical Exercise 2 — AWS Inventory

Given:

```python
instances = [
    {"id": "i-1", "environment": "prod", "state": "running"},
    {"id": "i-2", "environment": "dev", "state": "running"},
    {"id": "i-3", "environment": "prod", "state": "stopped"}
]
```

Find:

```text
Production instances
Running production instances
Their IDs
```

Expected:

```python
["i-1"]
```

---

# 91. Practical Exercise 3 — Kubernetes Health Data

Given:

```python
pods = [
    {"name": "payment-1", "phase": "Running"},
    {"name": "payment-2", "phase": "Pending"},
    {"name": "payment-3", "phase": "Running"}
]
```

Find unhealthy pods.

Expected:

```text
payment-2
```

Then extend the code to handle a missing `phase`.

---

# 92. Practical Exercise 4 — Disk Threshold

Given:

```python
disk_usage = 91.5
```

Return:

```text
NORMAL   < 80
WARNING  80-89.99
CRITICAL >= 90
```

Use named constants rather than repeating magic numbers.

---

# 93. Practical Exercise 5 — Safe Boolean Parsing

Accept:

```text
true
false
1
0
yes
no
on
off
```

Convert to Python booleans.

Reject:

```text
maybe
unknown
```

This is a common production configuration problem.

---

# 94. Practical Exercise 6 — Resource Report

Given:

```python
resources = [
    {"name": "web-01", "cpu": 45.5, "environment": "prod"},
    {"name": "web-02", "cpu": 91.2, "environment": "prod"},
    {"name": "test-01", "cpu": 95.0, "environment": "dev"}
]
```

Generate a report containing only production resources above 80% CPU.

Expected:

```text
web-02 -> 91.2%
```

---

# 95. Production Checklist

Before deploying Python automation:

```text
[ ] Variable names are descriptive
[ ] Units are explicit
[ ] External strings are converted correctly
[ ] Boolean configuration is parsed correctly
[ ] Required fields are validated
[ ] Optional fields have intentional defaults
[ ] API structures are not blindly assumed
[ ] Mutable data is handled deliberately
[ ] Type hints are used where useful
[ ] Sensitive values are not logged
[ ] Configuration is separated from logic
[ ] Invalid input fails clearly
[ ] Data structures are appropriate for the problem
```

---

# 96. Final Mental Model

Think about operational data in layers:

```text
External world
    |
    +--> Environment variables
    +--> CLI arguments
    +--> Files
    +--> JSON
    +--> YAML
    +--> AWS APIs
    +--> Kubernetes APIs
    |
    v
Python types
    |
    +--> str
    +--> int
    +--> float
    +--> bool
    +--> None
    +--> list
    +--> dict
    +--> tuple
    +--> set
    |
    v
Validation
    |
    v
Transformation
    |
    v
Automation action
    |
    v
Result / logs / exit code
```

The goal is not to memorize every Python type. The goal is to recognize the shape of operational data, validate it, transform it safely, and use it to make reliable automation decisions.

---

# 97. Next File

```text
22-Python-for-DevOps/
└── 01-Python-Fundamentals/
    ├── 01-Python-Introduction.md
    ├── 02-Variables-and-Data-Types.md
    └── 03-Operators.md
```

Next topic:

```text
Arithmetic operators
Comparison operators
Logical operators
Assignment operators
Bitwise operators
Membership operators
Identity operators
Operator precedence
Short-circuit evaluation
DevOps threshold logic
Retry/counter logic
Conditional expressions
Common mistakes
Production examples
Interview questions
Practical exercises
```
