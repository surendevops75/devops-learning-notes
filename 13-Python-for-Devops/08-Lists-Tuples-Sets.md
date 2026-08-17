# Lists-Tuples-Sets

## 1. Overview

Python provides several collection types for storing multiple values.

The three important collection types covered here are:

```text
List   -> ordered, mutable, duplicates allowed
Tuple  -> ordered, immutable, duplicates allowed
Set    -> unordered collection of unique values
```

DevOps automation uses collections constantly for:

- AWS resources
- EC2 instances
- EKS nodes
- Kubernetes pods
- Docker images
- CI/CD stages
- deployment targets
- security groups
- file names
- services
- environments
- log records
- configuration values

---

# 2. Lists

A list stores multiple values in an ordered collection.

```python
services = ["user", "product", "cart", "payment"]
```

Lists are:

```text
ordered
mutable
indexable
sliceable
duplicates allowed
```

---

# 3. Creating a List

```python
services = ["user", "product", "cart"]
```

Empty list:

```python
services = []
```

Using the constructor:

```python
services = list()
```

---

# 4. List Indexing

Python uses zero-based indexing.

```python
services = ["user", "product", "cart"]
```

Indexes:

```text
user     -> 0
product  -> 1
cart     -> 2
```

Example:

```python
print(services[0])
```

Output:

```text
user
```

---

# 5. Negative Indexing

```python
services = ["user", "product", "cart"]

print(services[-1])
```

Output:

```text
cart
```

Useful when you need the last item.

---

# 6. List Slicing

```python
services = ["user", "product", "cart", "payment"]

print(services[0:2])
```

Output:

```text
['user', 'product']
```

The ending index is excluded.

---

# 7. List Slice With Step

```python
numbers = [1, 2, 3, 4, 5, 6]

print(numbers[::2])
```

Output:

```text
[1, 3, 5]
```

---

# 8. Reverse a List

```python
services = ["user", "product", "cart"]

print(services[::-1])
```

This creates a reversed list.

For in-place reversal:

```python
services.reverse()
```

---

# 9. Lists Are Mutable

Unlike strings and tuples, lists can be changed.

```python
services = ["user", "product"]

services[1] = "payment"

print(services)
```

Output:

```text
['user', 'payment']
```

---

# 10. Why List Mutability Matters

A list passed to a function can be modified by that function.

```python
services = ["user", "payment"]

def add_service(items):
    items.append("cart")

add_service(services)

print(services)
```

The original list changes.

This can be useful but can also create unexpected side effects.

---

# 11. `append()`

Adds one item to the end.

```python
services = ["user", "product"]

services.append("payment")

print(services)
```

Output:

```text
['user', 'product', 'payment']
```

---

# 12. `extend()`

Adds multiple items.

```python
services = ["user", "product"]

services.extend(["cart", "payment"])

print(services)
```

Output:

```text
['user', 'product', 'cart', 'payment']
```

---

# 13. `append()` vs `extend()`

Important interview question.

```python
items = ["a", "b"]

items.append(["c", "d"])
```

Result:

```text
['a', 'b', ['c', 'd']]
```

But:

```python
items = ["a", "b"]

items.extend(["c", "d"])
```

Result:

```text
['a', 'b', 'c', 'd']
```

Think:

```text
append -> one object
extend -> elements from an iterable
```

---

# 14. `insert()`

Insert at a specific position.

```python
services = ["user", "payment"]

services.insert(1, "product")

print(services)
```

Output:

```text
['user', 'product', 'payment']
```

---

# 15. `remove()`

Remove the first matching value.

```python
services = ["user", "product", "payment"]

services.remove("product")
```

If the value does not exist, `remove()` raises `ValueError`.

Safer:

```python
if "product" in services:
    services.remove("product")
```

---

# 16. `pop()`

Remove and return an item.

```python
services = ["user", "product", "payment"]

service = services.pop()

print(service)
```

Output:

```text
payment
```

---

# 17. `pop(index)`

```python
services = ["user", "product", "payment"]

service = services.pop(1)

print(service)
```

Output:

```text
product
```

---

# 18. `clear()`

Remove everything.

```python
services = ["user", "product"]

services.clear()

print(services)
```

Output:

```text
[]
```

---

# 19. `len()`

```python
services = ["user", "product", "payment"]

print(len(services))
```

Output:

```text
3
```

Useful for:

- replica lists
- server lists
- deployment targets
- inventory counts

---

# 20. Membership Testing

```python
services = ["user", "product", "payment"]

if "payment" in services:
    print("Payment service exists")
```

---

# 21. `not in`

```python
services = ["user", "product"]

if "payment" not in services:
    print("Payment service missing")
```

---

# 22. Count Values

```python
services = ["user", "payment", "payment"]

print(services.count("payment"))
```

Output:

```text
2
```

---

# 23. Find Index

```python
services = ["user", "product", "payment"]

print(services.index("payment"))
```

Output:

```text
2
```

If the item is missing, `index()` raises `ValueError`.

---

# 24. Sorting

```python
services = ["payment", "cart", "user", "product"]

services.sort()

print(services)
```

Output:

```text
['cart', 'payment', 'product', 'user']
```

---

# 25. Reverse Sorting

```python
numbers = [10, 5, 20, 3]

numbers.sort(reverse=True)

print(numbers)
```

---

# 26. `sorted()` vs `sort()`

`sort()` modifies the original list:

```python
services.sort()
```

`sorted()` creates a new sorted list:

```python
new_services = sorted(services)
```

This distinction matters when the original ordering must be preserved.

---

# 27. Sorting With a Key

```python
services = ["api", "payment-service", "user"]

services.sort(key=len)

print(services)
```

The list is sorted by string length.

---

# 28. Copying a List

This does not create an independent copy:

```python
services2 = services
```

Both variables reference the same list.

---

# 29. Shallow Copy

Use:

```python
services2 = services.copy()
```

or:

```python
services2 = list(services)
```

or:

```python
services2 = services[:]
```

For a flat list, these create a separate list object.

---

# 30. Why Assignment Can Be Dangerous

```python
services = ["user", "payment"]

backup = services

backup.append("cart")

print(services)
```

The original list also changes.

This happens because both variables reference the same object.

---

# 31. Shallow Copy With Nested Data

A shallow copy only copies the outer collection.

Example:

```python
data = [["user", 2], ["payment", 3]]

copy_data = data.copy()

copy_data[0][1] = 10
```

The nested list is still shared.

For complex nested structures, understand shallow vs deep copying.

---

# 32. Deep Copy

```python
import copy

data = [["user", 2], ["payment", 3]]

copy_data = copy.deepcopy(data)
```

Now nested objects are recursively copied.

Use deep copies intentionally because they can be more expensive.

---

# 33. Iterating Through Lists

```python
services = ["user", "product", "payment"]

for service in services:
    print(service)
```

This is one of the most common patterns in DevOps automation.

---

# 34. Iterating With Index

Use `enumerate()`:

```python
services = ["user", "product", "payment"]

for index, service in enumerate(services):
    print(index, service)
```

Output:

```text
0 user
1 product
2 payment
```

---

# 35. Starting `enumerate()` From 1

```python
for number, service in enumerate(services, start=1):
    print(number, service)
```

---

# 36. Loop Through Infrastructure Resources

```python
instances = [
    "i-012345",
    "i-067890",
    "i-098765"
]

for instance in instances:
    print(f"Checking {instance}")
```

---

# 37. Nested Lists

```python
servers = [
    ["web-01", "10.0.1.10"],
    ["web-02", "10.0.1.11"],
]
```

Access:

```python
print(servers[0][0])
```

Output:

```text
web-01
```

For complex infrastructure data, dictionaries are often clearer than nested lists.

---

# 38. List Comprehension

A concise way to create lists.

```python
services = ["user", "product", "payment"]

upper_services = [
    service.upper()
    for service in services
]
```

Result:

```text
['USER', 'PRODUCT', 'PAYMENT']
```

---

# 39. List Comprehension With Condition

```python
services = ["user", "payment", "product", "payment"]

payment_services = [
    service
    for service in services
    if service == "payment"
]
```

---

# 40. Filtering Infrastructure Resources

```python
instances = ["web-01", "db-01", "web-02", "cache-01"]

web_servers = [
    instance
    for instance in instances
    if instance.startswith("web-")
]
```

---

# 41. Transforming Resource Names

```python
services = ["user", "product", "payment"]

names = [
    f"roboshop-{service}"
    for service in services
]
```

Result:

```text
['roboshop-user', 'roboshop-product', 'roboshop-payment']
```

---

# 42. Avoid Overly Complex Comprehensions

This is technically possible:

```python
result = [
    f"{service}-{env}"
    for service in services
    for env in environments
    if ...
]
```

But if the logic becomes difficult to read, use normal loops or functions.

Production code should prioritize clarity.

---

# 43. Conditional Expression in Comprehension

```python
statuses = ["running", "stopped", "running"]

result = [
    "UP" if status == "running" else "DOWN"
    for status in statuses
]
```

---

# 44. `filter()`

```python
services = ["user", "payment", "product"]

result = filter(
    lambda service: service.startswith("p"),
    services
)

print(list(result))
```

List comprehensions are often easier to read for simple transformations.

---

# 45. `map()`

```python
services = ["user", "payment"]

result = map(str.upper, services)

print(list(result))
```

---

# 46. `map()` vs List Comprehension

Both can transform values.

```python
list(map(str.upper, services))
```

or:

```python
[service.upper() for service in services]
```

Choose the clearer form.

---

# 47. `any()`

Check whether at least one condition is true.

```python
statuses = ["running", "running", "failed"]

if any(status == "failed" for status in statuses):
    print("Deployment has failures")
```

Very useful in monitoring and deployment validation.

---

# 48. `all()`

Check whether all conditions are true.

```python
statuses = ["running", "running", "running"]

if all(status == "running" for status in statuses):
    print("All services healthy")
```

---

# 49. `min()` and `max()`

```python
numbers = [10, 20, 5, 30]

print(min(numbers))
print(max(numbers))
```

Useful for metrics and thresholds.

---

# 50. `sum()`

```python
replicas = [2, 3, 4]

print(sum(replicas))
```

Output:

```text
9
```

---

# 51. `zip()`

Combine corresponding values.

```python
services = ["user", "payment", "cart"]
ports = [8080, 8081, 8082]

for service, port in zip(services, ports):
    print(service, port)
```

Useful for processing related collections.

---

# 52. `zip()` for Configuration

```python
services = ["user", "payment", "cart"]
replicas = [2, 3, 2]

configuration = list(zip(services, replicas))
```

Result:

```text
[
    ('user', 2),
    ('payment', 3),
    ('cart', 2)
]
```

For long-term configuration, dictionaries or dataclasses are usually clearer.

---

# 53. Unpacking Lists

```python
services = ["user", "payment", "cart"]

user, payment, cart = services
```

The number of variables must match the number of elements.

---

# 54. Extended Unpacking

```python
services = ["user", "product", "cart", "payment"]

first, *middle, last = services
```

Now:

```text
first = user
middle = ['product', 'cart']
last = payment
```

---

# 55. List Comprehension for File Processing

```python
lines = [
    line.strip()
    for line in file
    if line.strip()
]
```

This removes empty lines and surrounding whitespace.

For very large files, prefer streaming rather than storing everything in a list.

---

# 56. List of Files

```python
files = [
    "deployment.yaml",
    "service.yaml",
    "configmap.yaml"
]
```

Validate:

```python
yaml_files = [
    file
    for file in files
    if file.endswith((".yaml", ".yml"))
]
```

---

# 57. List of Kubernetes Namespaces

```python
namespaces = [
    "dev",
    "staging",
    "production"
]
```

Loop:

```python
for namespace in namespaces:
    print(f"Checking namespace {namespace}")
```

---

# 58. List of Deployment Targets

```python
targets = [
    "api",
    "worker",
    "frontend"
]
```

Deployment:

```python
for target in targets:
    deploy(target)
```

Always add appropriate failure handling for production automation.

---

# 59. Handling List Failures

Bad:

```python
for service in services:
    deploy(service)
```

If one deployment raises an exception, the entire loop may stop.

Depending on the workflow, you may need:

```python
for service in services:
    try:
        deploy(service)
    except Exception as exc:
        print(f"{service} failed: {exc}")
```

For real production systems, use structured logging and controlled failure policies.

---

# 60. Fail-Fast vs Continue-on-Error

Two valid strategies:

```text
Fail-fast:
service failure -> stop deployment
```

or:

```text
Continue:
service failure -> record failure -> continue
```

The correct approach depends on dependency relationships and deployment policy.

---

# 61. List of Dependencies

```python
services = [
    "user",
    "product",
    "cart",
    "payment"
]
```

If:

```text
payment depends on cart
cart depends on product
```

a simple list is not enough to represent the dependency graph.

Use a suitable graph/data structure instead.

---

# 62. Lists From Command Output

```python
output = """
user
product
payment
"""

services = output.splitlines()
```

Clean:

```python
services = [
    line.strip()
    for line in output.splitlines()
    if line.strip()
]
```

---

# 63. Lists From JSON

Suppose an API returns:

```json
{
  "services": ["user", "product", "payment"]
}
```

Python:

```python
services = data["services"]
```

Always validate external JSON before assuming the field exists.

---

# 64. Safe JSON List Access

```python
services = data.get("services", [])
```

If the field is optional, this can provide a safe default.

But if the field is mandatory, silently defaulting to an empty list may hide configuration errors.

---

# 65. Tuple Basics

A tuple is an ordered immutable collection.

```python
regions = ("ap-south-1", "us-east-1")
```

Tuples are:

```text
ordered
immutable
indexable
sliceable
duplicates allowed
```

---

# 66. Creating a Tuple

```python
values = (1, 2, 3)
```

You can also write:

```python
values = 1, 2, 3
```

The comma is what makes it a tuple.

---

# 67. Single-Element Tuple

This is not a tuple:

```python
value = ("production")
```

This is a tuple:

```python
value = ("production",)
```

The comma matters.

---

# 68. Tuple Indexing

```python
regions = ("ap-south-1", "us-east-1")

print(regions[0])
```

---

# 69. Tuple Slicing

```python
regions = ("ap-south-1", "us-east-1", "eu-west-1")

print(regions[:2])
```

---

# 70. Tuple Immutability

This is invalid:

```python
regions[0] = "ap-northeast-1"
```

It raises:

```text
TypeError
```

---

# 71. Why Use Tuples?

Tuples are useful when the collection should not change.

Examples:

```python
DEFAULT_REGIONS = ("ap-south-1", "us-east-1")
```

or:

```python
coordinates = (10.2, 20.4)
```

---

# 72. Tuple Unpacking

```python
server = ("web-01", "10.0.1.10")

name, ip = server
```

This is common and readable.

---

# 73. Function Return Values

Python functions commonly return tuples:

```python
def get_server():
    return "web-01", "10.0.1.10"

name, ip = get_server()
```

This is an important Python pattern.

---

# 74. Tuple as Dictionary Key

Tuples containing hashable values can be used as dictionary keys:

```python
resource = {
    ("production", "payment"): 3
}
```

Lists cannot be dictionary keys because lists are mutable and unhashable.

---

# 75. Tuple Methods

Tuples provide fewer methods because they are immutable.

Important methods:

```python
count()
index()
```

Example:

```python
values = ("prod", "dev", "prod")

print(values.count("prod"))
```

---

# 76. Tuple vs List

| Feature | List | Tuple |
|---|---|---|
| Ordered | Yes | Yes |
| Mutable | Yes | No |
| Duplicates | Yes | Yes |
| Indexing | Yes | Yes |
| Slicing | Yes | Yes |
| Methods | Many | Few |
| Use case | Dynamic collection | Fixed collection |

---

# 77. Production Decision

Use a list when:

```text
items will change
```

Use a tuple when:

```text
items represent a fixed group
```

Do not choose tuples merely because they are immutable; choose based on semantics and API design.

---

# 78. Sets

A set stores unique values.

```python
services = {"user", "payment", "cart"}
```

Sets are:

```text
unique
mutable
unordered
not indexable
```

---

# 79. Creating an Empty Set

This is a dictionary:

```python
items = {}
```

Correct empty set:

```python
items = set()
```

---

# 80. Set Automatically Removes Duplicates

```python
services = {"user", "payment", "user"}

print(services)
```

Only unique values remain.

---

# 81. Convert List to Set

```python
services = ["user", "payment", "user", "cart"]

unique_services = set(services)
```

Useful for deduplication.

Remember that converting to a set does not preserve list ordering semantics.

---

# 82. Add to Set

```python
services = {"user", "payment"}

services.add("cart")
```

---

# 83. Remove From Set

```python
services.remove("payment")
```

Raises `KeyError` if the value does not exist.

---

# 84. `discard()`

```python
services.discard("payment")
```

Does not raise an error if the value is absent.

---

# 85. Set Membership

```python
allowed = {"dev", "staging", "production"}

if environment in allowed:
    print("Valid environment")
```

This is a very common and efficient validation pattern.

---

# 86. Set Union

```python
dev = {"user", "product"}
prod = {"user", "payment"}

all_services = dev | prod
```

Result contains unique values from both sets.

---

# 87. `union()`

Equivalent:

```python
all_services = dev.union(prod)
```

---

# 88. Set Intersection

```python
common = dev & prod
```

Finds values present in both.

Useful for:

```text
common services
common permissions
common resources
```

---

# 89. `intersection()`

```python
common = dev.intersection(prod)
```

---

# 90. Set Difference

```python
only_dev = dev - prod
```

Finds values present in `dev` but not `prod`.

Useful for identifying:

```text
services missing from production
```

---

# 91. Symmetric Difference

```python
difference = dev ^ prod
```

Returns values that exist in one set but not both.

---

# 92. Subset

```python
required = {"user", "product"}

available = {"user", "product", "payment"}

print(required.issubset(available))
```

Output:

```text
True
```

---

# 93. Superset

```python
print(available.issuperset(required))
```

---

# 94. Disjoint Sets

```python
a = {"user", "product"}
b = {"payment", "cart"}

print(a.isdisjoint(b))
```

Returns:

```text
True
```

---

# 95. Set Operations in DevOps

Suppose:

```python
expected = {"user", "product", "cart", "payment"}
running = {"user", "product", "payment"}
```

Find missing services:

```python
missing = expected - running
```

Result:

```text
{'cart'}
```

This is an excellent production validation pattern.

---

# 96. Find Unexpected Resources

```python
expected = {"user", "product", "payment"}
actual = {"user", "product", "payment", "debug"}

unexpected = actual - expected
```

This can identify unexpected deployments or resources.

---

# 97. Environment Drift Detection

```python
staging = {"user", "product", "cart", "payment"}
production = {"user", "product", "payment"}

missing_in_prod = staging - production
extra_in_prod = production - staging
```

This gives a simple configuration drift comparison.

---

# 98. Kubernetes Drift Example

Expected:

```python
expected_deployments = {
    "user",
    "product",
    "cart",
    "payment",
}
```

Actual:

```python
actual_deployments = {
    "user",
    "product",
    "payment",
}
```

Then:

```python
missing = expected_deployments - actual_deployments
```

---

# 99. Set Performance

Set membership is generally very efficient:

```python
if service in allowed_services:
    ...
```

For large collections where membership checking dominates, a set is often preferable to a list.

---

# 100. Why Not Use Set Everywhere?

Sets:

```text
do not provide list-style indexing
do not represent ordered business sequences
remove duplicates
```

If order or duplicates matter, use a list.

---

# 101. List vs Set Example

Deployment order:

```python
deployment_order = [
    "database",
    "backend",
    "frontend"
]
```

Use a list because order matters.

Allowed environments:

```python
allowed = {
    "dev",
    "staging",
    "production"
}
```

Use a set because uniqueness and membership matter.

---

# 102. Tuple vs Set Example

Fixed AWS regions:

```python
regions = (
    "ap-south-1",
    "us-east-1",
)
```

Unique allowed regions:

```python
allowed_regions = {
    "ap-south-1",
    "us-east-1",
}
```

The choice depends on whether ordering/immutability or uniqueness/membership is the main requirement.

---

# 103. Frozen Set

A `frozenset` is an immutable set.

```python
regions = frozenset({
    "ap-south-1",
    "us-east-1"
})
```

Useful when you need set semantics but do not want the collection modified.

A `frozenset` can also be used where a hashable set-like object is required.

---

# 104. Set Cannot Contain Mutable Values

This is invalid:

```python
items = {[1, 2], [3, 4]}
```

because lists are unhashable.

Use tuples:

```python
items = {(1, 2), (3, 4)}
```

---

# 105. Hashability

Dictionary keys and set members generally need hashable objects.

Common hashable types:

```text
str
int
float
bool
tuple of hashable values
frozenset
```

Common unhashable types:

```text
list
dict
set
```

---

# 106. Nested Collections

Example:

```python
services = {
    "frontend": ["web", "nginx"],
    "backend": ["user", "payment"],
}
```

This is a dictionary containing lists.

For complex configurations, dictionaries often provide more readable structure than nested lists.

---

# 107. List of Dictionaries

Very common in automation:

```python
instances = [
    {"name": "web-01", "ip": "10.0.1.10"},
    {"name": "web-02", "ip": "10.0.1.11"},
]
```

Process:

```python
for instance in instances:
    print(instance["name"])
```

This structure is often returned by APIs.

---

# 108. Filtering List of Dictionaries

```python
production = [
    item
    for item in instances
    if item["environment"] == "production"
]
```

Always validate external API fields if the schema is not guaranteed.

---

# 109. Sorting List of Dictionaries

```python
instances.sort(key=lambda item: item["name"])
```

Or:

```python
sorted_instances = sorted(
    instances,
    key=lambda item: item["name"]
)
```

---

# 110. `key` Functions

For:

```python
instances = [
    {"name": "web-01", "cpu": 80},
    {"name": "web-02", "cpu": 40},
]
```

Sort by CPU:

```python
instances.sort(key=lambda item: item["cpu"])
```

---

# 111. `operator.itemgetter`

For repeated sorting patterns:

```python
from operator import itemgetter

instances.sort(key=itemgetter("cpu"))
```

This can improve readability.

---

# 112. List Deduplication While Preserving Order

Simple:

```python
items = ["a", "b", "a", "c"]

unique = list(dict.fromkeys(items))
```

Result:

```text
['a', 'b', 'c']
```

This works for hashable values.

---

# 113. Set Deduplication Tradeoff

```python
unique = list(set(items))
```

This removes duplicates but does not preserve the original ordering semantics.

Use the approach appropriate to your requirements.

---

# 114. `dict.fromkeys()` for Unique Values

```python
services = [
    "user",
    "payment",
    "user",
    "cart",
]

unique_services = list(dict.fromkeys(services))
```

Useful when both uniqueness and original order matter.

---

# 115. List Memory Considerations

Large lists keep all elements in memory.

If processing millions of records, consider:

```text
generators
iterators
streaming
pagination
batch processing
```

rather than building one enormous list.

---

# 116. Generator vs List

List:

```python
values = [x * 2 for x in range(1000000)]
```

creates all values immediately.

Generator:

```python
values = (x * 2 for x in range(1000000))
```

produces values lazily.

This is useful for large data processing.

---

# 117. Generator in DevOps

For a large log:

```python
def errors(file):
    for line in file:
        if "ERROR" in line:
            yield line
```

This avoids storing all matching lines in memory.

Generators are covered more deeply later in the Python curriculum.

---

# 118. Avoid Modifying a List While Iterating

Dangerous pattern:

```python
for service in services:
    if service == "deprecated":
        services.remove(service)
```

Items can be skipped.

---

# 119. Safe Filtering

Prefer:

```python
services = [
    service
    for service in services
    if service != "deprecated"
]
```

Or build a new collection.

---

# 120. Removing Multiple Values

```python
blocked = {"deprecated", "test"}

services = [
    service
    for service in services
    if service not in blocked
]
```

Set membership makes the intent clear.

---

# 121. List Mutation During Function Calls

Be careful with:

```python
def process(items):
    items.append("new")
```

If the caller expects the list to remain unchanged, this creates a hidden side effect.

Prefer returning a new list when immutability is desirable:

```python
def process(items):
    return [*items, "new"]
```

---

# 122. Mutable Default Argument Trap

Avoid:

```python
def deploy(services=[]):
    services.append("payment")
    return services
```

The same default list is reused across calls.

Use:

```python
def deploy(services=None):
    if services is None:
        services = []

    services.append("payment")
    return services
```

This is a common Python interview question.

---

# 123. List as Function Argument

```python
def deploy_services(services):
    for service in services:
        deploy(service)
```

Call:

```python
deploy_services([
    "user",
    "product",
    "payment",
])
```

---

# 124. Tuple as Function Contract

```python
def get_region():
    return "ap-south-1", "production"
```

Caller:

```python
region, environment = get_region()
```

A tuple is useful for a fixed number of related return values.

---

# 125. Set for Validation

```python
VALID_ENVIRONMENTS = {
    "dev",
    "staging",
    "production",
}
```

Then:

```python
if environment not in VALID_ENVIRONMENTS:
    raise ValueError("Invalid environment")
```

This is cleaner than a long chain of `or` conditions.

---

# 126. Production CI/CD Example

Suppose a pipeline receives:

```python
changed_services = [
    "user",
    "payment",
    "payment",
    "cart",
]
```

Deduplicate while preserving order:

```python
changed_services = list(dict.fromkeys(changed_services))
```

Then:

```python
for service in changed_services:
    deploy(service)
```

---

# 127. Production Deployment Target Selection

```python
targets = {
    "dev": ["user", "product", "payment"],
    "staging": ["user", "product", "payment", "cart"],
    "production": ["user", "product", "payment"],
}
```

Select:

```python
services = targets["production"]
```

For more complex configuration, consider typed configuration models or dedicated config files.

---

# 128. Production Health Validation

```python
expected = {"user", "product", "payment"}

healthy = {
    "user",
    "payment",
}

missing = expected - healthy

if missing:
    print(f"Unhealthy services: {missing}")
```

This pattern is useful in deployment verification.

---

# 129. Production Node Validation

```python
expected_nodes = {
    "ip-10-0-1-10",
    "ip-10-0-1-11",
    "ip-10-0-1-12",
}

actual_nodes = {
    "ip-10-0-1-10",
    "ip-10-0-1-11",
}

missing = expected_nodes - actual_nodes
```

---

# 130. Production Resource Drift

Compare:

```python
desired = {"alb", "eks", "rds", "s3"}
actual = {"alb", "eks", "rds", "s3", "test-bucket"}
```

Unexpected:

```python
actual - desired
```

Missing:

```python
desired - actual
```

This is a simplified drift-detection concept.

---

# 131. Production Inventory

A list can represent an ordered inventory:

```python
inventory = [
    "web-01",
    "web-02",
    "worker-01",
]
```

A set can represent unique inventory:

```python
unique_inventory = set(inventory)
```

A dictionary can represent detailed inventory:

```python
inventory = {
    "web-01": {"ip": "10.0.1.10", "role": "web"},
}
```

Choose the data structure based on the information you need.

---

# 132. Common Mistake — `{}`

Remember:

```python
{}
```

is an empty dictionary.

Use:

```python
set()
```

for an empty set.

---

# 133. Common Mistake — Tuple Comma

Incorrect:

```python
value = ("prod")
```

Correct:

```python
value = ("prod",)
```

---

# 134. Common Mistake — `append()` vs `extend()`

```python
append(["a", "b"])
```

adds one list.

```python
extend(["a", "b"])
```

adds two elements.

---

# 135. Common Mistake — List Assignment

```python
backup = original
```

does not copy the list.

Use:

```python
backup = original.copy()
```

for a shallow copy.

---

# 136. Common Mistake — Set Ordering

Do not rely on a set for deployment order.

Use:

```python
list
```

when order matters.

---

# 137. Common Mistake — Set Deduplication

Converting:

```python
list -> set -> list
```

can change ordering.

If order matters:

```python
list(dict.fromkeys(values))
```

---

# 138. Common Mistake — Mutating During Iteration

Do not remove elements directly from the list being iterated.

Build a filtered list instead.

---

# 139. Common Mistake — Mutable Default Arguments

Avoid:

```python
def function(items=[]):
```

Use:

```python
def function(items=None):
```

and initialize inside.

---

# 140. Common Mistake — Huge Lists

Do not load millions of records into a list unless required.

Use:

```text
streaming
generators
pagination
batching
iterators
```

---

# 141. Common Mistake — Overusing Comprehensions

Readable:

```python
names = [x["name"] for x in instances]
```

Potentially unreadable:

```python
result = [complex_expression(x) for x in items if condition(x) and another_condition(x)]
```

Use a normal loop when the logic becomes difficult to understand.

---

# 142. Common Mistake — Wrong Data Structure

If you need:

```text
unique membership -> set
ordered mutable collection -> list
fixed immutable collection -> tuple
key-value lookup -> dictionary
```

Do not force every problem into a list.

---

# 143. Interview Question — List vs Tuple

Answer:

> A list is mutable while a tuple is immutable. I use lists when the collection needs to change and tuples when the values represent a fixed ordered group or fixed return structure.

---

# 144. Interview Question — List vs Set

Answer:

> Lists preserve ordering and allow duplicates. Sets contain unique values and are optimized for membership-style operations. I use lists for deployment sequences and sets for validation, deduplication, and drift comparison.

---

# 145. Interview Question — `append()` vs `extend()`

Answer:

> `append()` adds one object as a single element. `extend()` iterates over another iterable and adds its elements individually.

---

# 146. Interview Question — `sort()` vs `sorted()`

Answer:

> `sort()` modifies the existing list in place and returns `None`. `sorted()` returns a new sorted collection and leaves the original unchanged.

---

# 147. Interview Question — Why Is a Set Useful in DevOps?

Answer:

> Sets are useful for unique resource inventories, allowed values, membership validation, deduplication, and comparing desired versus actual infrastructure. For example, I can calculate missing Kubernetes deployments using `expected - actual`.

---

# 148. Interview Question — What Is List Comprehension?

Answer:

> It is a concise syntax for constructing a list from an iterable, optionally applying a transformation and filtering condition.

Example:

```python
running = [
    pod
    for pod in pods
    if pod["status"] == "Running"
]
```

---

# 149. Interview Question — What Is a Shallow Copy?

Answer:

> A shallow copy creates a new outer collection but does not recursively copy nested objects. Therefore, nested mutable objects can still be shared.

---

# 150. Interview Question — What Is a Mutable Default Argument?

Answer:

> Python evaluates default arguments once when the function is defined. A mutable default such as `[]` can therefore be shared across function calls. I use `None` and create a new list inside the function.

---

# 151. Interview Question — How Would You Detect Missing Services?

Answer:

```python
expected = {"user", "product", "cart"}
actual = {"user", "product"}

missing = expected - actual
```

Then:

```python
if missing:
    raise RuntimeError(f"Missing services: {missing}")
```

---

# 152. Scenario — Production Has an Unexpected Deployment

Expected:

```python
expected = {"user", "product", "payment"}
```

Actual:

```python
actual = {"user", "product", "payment", "debug"}
```

Find unexpected resources:

```python
unexpected = actual - expected
```

Then investigate before automatically deleting anything.

---

# 153. Scenario — Production Is Missing a Service

```python
missing = expected - actual
```

Then:

```text
check deployment
check namespace
check Git desired state
check ArgoCD sync status
check image
check scheduling
check rollout
```

Python only identifies the difference; operational investigation still requires Kubernetes/observability tools.

---

# 154. Scenario — Duplicate Services in CI Input

Input:

```python
services = [
    "payment",
    "user",
    "payment",
]
```

If deployment should happen once:

```python
services = list(dict.fromkeys(services))
```

This preserves first-seen ordering.

---

# 155. Scenario — Deployment Order Matters

Do not use:

```python
set(services)
```

because order is not the business contract.

Use:

```python
deployment_order = [
    "database",
    "backend",
    "frontend",
]
```

---

# 156. Scenario — Large AWS Inventory

If an AWS API returns thousands of resources:

```text
do not unnecessarily build huge transformed lists
```

Prefer:

```text
pagination
streaming where available
batch processing
generators
```

This reduces memory pressure.

---

# 157. Scenario — Kubernetes Pod Health

Given:

```python
pods = [
    {"name": "user-1", "status": "Running"},
    {"name": "payment-1", "status": "CrashLoopBackOff"},
]
```

Find unhealthy pods:

```python
unhealthy = [
    pod
    for pod in pods
    if pod["status"] != "Running"
]
```

---

# 158. Scenario — Environment Comparison

```python
staging = {"user", "product", "cart", "payment"}
production = {"user", "product", "payment"}
```

Find missing:

```python
staging - production
```

Find extra:

```python
production - staging
```

This is a useful simplified configuration-drift technique.

---

# 159. Scenario — Service Availability

```python
services = {
    "user": "healthy",
    "product": "healthy",
    "payment": "failed",
}
```

A dictionary is better than a list here because each service has a key and state.

This demonstrates an important principle:

> Choose the data structure based on the relationship between the data.

---

# 160. Production Collection Design

A practical DevOps automation might use all three:

```python
deployment_order = (
    "database",
    "backend",
    "frontend",
)

services = [
    "user",
    "product",
    "payment",
]

allowed_environments = {
    "dev",
    "staging",
    "production",
}
```

Each collection communicates a different intent.

---

# 161. Production Architecture Example

```text
CI/CD Variables
      |
      v
Python Configuration
      |
      +--> tuple: fixed deployment stages
      |
      +--> list: ordered changed services
      |
      +--> set: allowed environments
      |
      v
Validation
      |
      v
AWS / Kubernetes / Docker
      |
      v
Verification
```

---

# 162. Collection Processing Pipeline

```text
Raw API / CLI Data
       |
       v
Parse
       |
       v
List / Tuple / Set
       |
       v
Filter
       |
       v
Transform
       |
       v
Compare
       |
       v
Validate
       |
       v
Take Action
```

---

# 163. Practical Exercise — List Operations

Create:

```python
services = ["user", "product", "cart"]
```

Perform:

```text
append payment
insert notification
remove cart
sort
reverse
```

Print the result after every operation.

---

# 164. Practical Exercise — Deduplicate Services

Input:

```python
services = [
    "user",
    "payment",
    "user",
    "cart",
    "payment",
]
```

Create a unique ordered list.

Expected:

```text
['user', 'payment', 'cart']
```

---

# 165. Practical Exercise — Environment Validation

Create:

```python
allowed = {"dev", "staging", "production"}
```

Validate an environment supplied by the user.

Reject invalid values.

---

# 166. Practical Exercise — Kubernetes Drift

Create:

```python
expected = {"user", "product", "cart", "payment"}
actual = {"user", "product", "payment", "debug"}
```

Print:

```text
missing
unexpected
```

---

# 167. Practical Exercise — Pod Filtering

Given:

```python
pods = [
    {"name": "user-1", "status": "Running"},
    {"name": "payment-1", "status": "CrashLoopBackOff"},
    {"name": "cart-1", "status": "Running"},
]
```

Find unhealthy pods.

---

# 168. Practical Exercise — Deployment Ordering

Create:

```python
deployment_order = (
    "database",
    "backend",
    "frontend",
)
```

Loop through it and print:

```text
Deploying database
Deploying backend
Deploying frontend
```

---

# 169. Practical Exercise — Large Log Processing

Write a function that yields only lines containing:

```text
ERROR
```

Do not load the entire file into memory.

---

# 170. Practical Exercise — AWS Inventory

Given:

```python
expected = {"web-01", "web-02", "web-03"}
actual = {"web-01", "web-03", "web-04"}
```

Find:

```text
missing instances
unexpected instances
```

---

# 171. Final Collection Cheat Sheet

```python
# List
items = []
items.append(x)
items.extend(values)
items.insert(i, x)
items.remove(x)
items.pop()
items.sort()
items.reverse()

# Tuple
items = (1, 2, 3)
a, b, c = items

# Set
items = set()
items.add(x)
items.discard(x)

a | b       # union
a & b       # intersection
a - b       # difference
a ^ b       # symmetric difference

# Common
len(items)
x in items
enumerate(items)
zip(a, b)
sorted(items)
any(...)
all(...)
```

---

# 172. Final Decision Guide

```text
Need ordered mutable data?
        -> List

Need fixed ordered data?
        -> Tuple

Need unique values?
        -> Set

Need key/value lookup?
        -> Dictionary

Need ordered deployment sequence?
        -> List / Tuple

Need allowed-value validation?
        -> Set

Need immutable configuration constants?
        -> Tuple

Need desired-vs-actual comparison?
        -> Set

Need API records with named fields?
        -> List of dictionaries or typed objects
```

---

# 173. Final DevOps Mental Model

The important lesson is not memorizing every method.

Think:

```text
What does my data represent?
          |
          v
Does order matter?
          |
          +-- Yes --> List / Tuple
          |
          No
          |
          v
Do duplicates matter?
          |
          +-- Yes --> List
          |
          No
          |
          v
Use Set
```

And for infrastructure automation:

```text
Lists  -> ordered deployment/resources
Tuples -> fixed configuration
Sets   -> uniqueness / validation / drift
Dicts  -> structured resource information
```

The right data structure makes DevOps automation simpler, safer, and easier to maintain.

---

# 174. Next File

```text
09-Dictionaries.md
```

The next file will cover dictionaries deeply, including:

```text
creation
keys and values
get()
setdefault()
update()
pop()
popitem()
keys()
values()
items()
nested dictionaries
dictionary comprehensions
list of dictionaries
JSON/API responses
AWS resource data
Kubernetes resource data
configuration management
inventory automation
safe access patterns
copying
merging
validation
performance
common mistakes
production DevOps patterns
interview questions
scenario-based questions
practical exercises
```
