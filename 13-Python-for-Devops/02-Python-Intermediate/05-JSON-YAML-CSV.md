# JSON-YAML-CSV

## DevOps Focus

This file covers JSON, YAML, and CSV from a **Python-for-DevOps** perspective. The goal is not just syntax memorization. You should be able to use these formats in AWS, Kubernetes/EKS, Terraform, Helm, GitOps, CI/CD, DevSecOps, monitoring, logging, and production automation.

> Core principle: **parse structured data with a real parser; use regex for unstructured text.**


## 1. JSON Fundamentals

JSON is the most common machine-readable format in DevOps. Python's standard `json` module handles it without an external dependency.

```python
import json

text = '{"service": "payment", "replicas": 3}'
data = json.loads(text)

print(data["service"])
print(data["replicas"])
```

Use JSON for APIs, AWS/Kubernetes machine output, security reports, and automation metadata.

## 2. load vs loads / dump vs dumps

Remember the four core functions:

```text
json.loads(string) -> Python object
json.load(file)   -> Python object
json.dumps(object) -> JSON string
json.dump(object, file) -> JSON file
```

Example:

```python
with open("config.json", encoding="utf-8") as f:
    data = json.load(f)

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2)
```

## 3. Nested JSON

Cloud APIs commonly return deeply nested objects.

```python
replicas = (
    data["application"]
        ["deployment"]
        ["replicas"]
)
```

For optional fields, use `.get()` and validate required fields explicitly:

```python
status = data.get("status")
if status is None:
    raise ValueError("status is required")
```

## 4. JSON Arrays

JSON arrays become Python lists.

```python
for service in data.get("services", []):
    print(service["name"])
```

Always validate the expected item type before assuming every element is a dictionary.

## 5. JSON Error Handling

Invalid JSON raises `json.JSONDecodeError`.

```python
try:
    data = json.loads(text)
except json.JSONDecodeError as exc:
    raise ValueError("Invalid JSON") from exc
```

Do not silently continue with invalid deployment or security configuration.

## 6. JSON Validation

Parsing proves syntax, not correctness. Validate required fields and types.

```python
replicas = data.get("replicas")

if not isinstance(replicas, int) or replicas < 1:
    raise ValueError("replicas must be a positive integer")
```

For larger contracts, use JSON Schema or an approved typed validation library.

## 7. JSON Pretty Printing

Use deterministic, readable output:

```python
json.dumps(
    data,
    indent=2,
    sort_keys=True,
)
```

Stable serialization reduces noisy Git diffs and makes code review easier.

## 8. JSON Lines / NDJSON

For very large event streams, JSON Lines stores one JSON document per line.

```text
{"service":"payment","status":500}
{"service":"catalog","status":200}
```

Process it incrementally:

```python
for line in file:
    record = json.loads(line)
    process(record)
```

This avoids loading a huge document into memory.

## 9. AWS CLI JSON

When using AWS CLI from Python, prefer machine-readable JSON rather than human-readable tables.

```python
result = subprocess.run(
    ["aws", "ec2", "describe-instances", "--output", "json"],
    capture_output=True,
    text=True,
    check=True,
)
data = json.loads(result.stdout)
```

For long-lived Python automation, prefer `boto3` when it provides the required API.

## 10. Kubernetes JSON

Never build production automation around the formatting of `kubectl get pods` tables.

Prefer:

```bash
kubectl get pods -A -o json
```

Then:

```python
data = json.loads(result.stdout)

for pod in data["items"]:
    print(
        pod["metadata"]["namespace"],
        pod["metadata"]["name"],
        pod["status"].get("phase"),
    )
```

For robust applications, the Kubernetes API/client is preferable.

## 11. Kubernetes Container Status

Structured Kubernetes data lets you inspect restart counts, readiness, images, and waiting reasons.

```python
for pod in data["items"]:
    for container in pod["status"].get("containerStatuses", []):
        if container.get("restartCount", 0) > 5:
            print("High restart:", pod["metadata"]["name"])
```

This is much safer than regex against CLI tables.

## 12. Kubernetes Manifest YAML

YAML is the standard human-facing format for Kubernetes manifests.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
spec:
  replicas: 3
```

Python can read it into dictionaries and lists, but Kubernetes-specific validation still needs Kubernetes tooling.

## 13. PyYAML Safe Loading

Install the approved PyYAML dependency and use:

```python
import yaml

data = yaml.safe_load(text)
```

Avoid unsafe loaders for untrusted configuration. Safe parsing is only the first layer; validate schema, types, and platform rules afterward.

## 14. YAML File Operations

Read:

```python
with open("values.yaml", encoding="utf-8") as f:
    values = yaml.safe_load(f)
```

Write:

```python
with open("values.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(values, f, sort_keys=False)
```

Be aware that normal YAML serialization may not preserve comments or exact formatting.

## 15. Multi-Document YAML

Kubernetes files can contain multiple documents separated by `---`.

```python
with open("manifests.yaml", encoding="utf-8") as f:
    documents = list(yaml.safe_load_all(f))

for document in documents:
    if document:
        print(document.get("kind"))
```

Preserve document boundaries when writing multi-resource files back.

## 16. Updating Helm Values

A GitOps image update should parse YAML rather than perform blind text replacement.

```python
values["image"]["tag"] = new_tag
```

Then validate, compare old/new state, and write only if the desired state actually changed.

## 17. Updating a Kubernetes Image

For multi-container workloads, locate the container by name instead of assuming index zero.

```python
containers = (
    manifest["spec"]["template"]["spec"]["containers"]
)

for container in containers:
    if container["name"] == "payment":
        container["image"] = "payment:v42"
```

Then validate with Helm/Kubernetes tooling.

## 18. YAML and GitOps

A strong GitOps flow is:

```text
CI builds image
   -> update values.yaml
   -> validate
   -> commit / pull request
   -> merge
   -> ArgoCD reconciliation
   -> EKS
```

Do not bypass Git as the source of truth unless direct cluster changes are an intentional operational procedure.

## 19. YAML vs JSON

JSON is strict and common for APIs. YAML is more human-friendly and common for Kubernetes, Helm, CI/CD, and configuration.

Use:

```text
JSON -> machine interfaces
YAML -> human-maintained configuration
```

Both ultimately become Python dictionaries/lists when parsed.

## 20. YAML Security

Never assume a YAML file is safe because it came from Git. Treat external YAML as untrusted input.

Use `yaml.safe_load()`, validate expected fields, restrict file permissions for sensitive files, and never commit real secrets.

## 21. CSV Fundamentals

CSV is appropriate for tabular data such as inventories, exports, audit reports, and spreadsheet-compatible results.

Use Python's `csv` module instead of manually joining strings with commas.

## 22. CSV Reader

Basic reading:

```python
import csv

with open("services.csv", newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

For named columns, `DictReader` is usually easier for DevOps scripts.

## 23. CSV DictReader

Use:

```python
reader = csv.DictReader(file)

for row in reader:
    service = row["service"]
    environment = row["environment"]
```

CSV values are strings, so convert them explicitly according to the schema.

## 24. CSV DictWriter

Generate reports safely:

```python
writer = csv.DictWriter(
    file,
    fieldnames=["service", "status", "version"],
)
writer.writeheader()
writer.writerows(rows)
```

The CSV module correctly handles quoting, commas, and embedded newlines.

## 25. CSV Validation

Validate:

```text
required columns
missing values
numeric fields
allowed environments
duplicates
unexpected columns
```

Example:

```python
replicas = int(row["replicas"])
if replicas < 1:
    raise ValueError("invalid replicas")
```

## 26. CSV Large Files

Process large CSV files incrementally:

```python
with open("large.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        process(row)
```

Do not convert a huge file to `list(reader)` unless the memory cost is acceptable.

## 27. JSON to CSV

JSON often contains nested data while CSV is flat. Explicitly choose which fields to export.

```python
rows = [
    {
        "service": item["name"],
        "status": item["status"],
    }
    for item in services
]
```

Then use `DictWriter`. Do not assume every nested JSON structure maps naturally to a single CSV row.

## 28. CSV to JSON

`DictReader` produces strings:

```python
data = list(csv.DictReader(file))
```

If `replicas` must be numeric, explicitly convert it:

```python
record["replicas"] = int(record["replicas"])
```

Never use `eval()` for external data.

## 29. JSON to YAML

Both formats can represent dictionaries and lists.

```python
data = json.load(json_file)

yaml.safe_dump(
    data,
    yaml_file,
    sort_keys=False,
)
```

Be aware that YAML can represent features and types that do not map one-to-one to JSON.

## 30. YAML to JSON

Read YAML safely and serialize the resulting Python object:

```python
data = yaml.safe_load(file)

json.dump(
    data,
    output,
    indent=2,
)
```

Validate the resulting object before writing it into a production pipeline.

## 31. CSV to YAML

CSV is tabular, while YAML supports nesting. Define the target structure explicitly.

```python
rows = list(csv.DictReader(file))

yaml.safe_dump(
    rows,
    output,
    sort_keys=False,
)
```

Convert numeric/boolean fields before serialization if consumers expect typed values.

## 32. Data Type Conversion

CSV values are strings. Use explicit converters:

```python
def parse_bool(value):
    value = value.strip().lower()
    if value in {"true", "1", "yes"}:
        return True
    if value in {"false", "0", "no"}:
        return False
    raise ValueError("invalid boolean")
```

Use `int()`, `float()`, and datetime parsing for known fields.

## 33. Configuration Validation Pipeline

A production configuration workflow should be:

```text
read
 -> parse syntax
 -> validate schema
 -> validate business rules
 -> validate security
 -> transform
 -> serialize
 -> re-read/verify
 -> dry-run
 -> deploy
```

Do not jump directly from parsing to deployment.

## 34. Kubernetes Dry Run

After generating a manifest, use Kubernetes-native validation where appropriate:

```bash
kubectl apply --dry-run=server -f manifest.yaml
```

This catches cluster/API-level problems that a YAML parser cannot detect.

## 35. Helm Validation

For Helm-based deployments, useful checks include:

```bash
helm lint
helm template
```

A Python script can orchestrate these commands with `subprocess`, while Helm remains responsible for Helm-specific rendering and validation.

## 36. Configuration Drift

Compare desired state from Git with actual state from the platform.

```text
Git values
   |
   v
desired
   |
   +---- compare ---- actual
                       ^
                       |
                 Kubernetes API
```

Compare semantic fields such as image and replicas rather than raw YAML formatting.

## 37. Semantic Comparison

Parse before comparing:

```python
old = yaml.safe_load(old_text)
new = yaml.safe_load(new_text)

if old == new:
    print("No semantic change")
```

This avoids false differences caused by key ordering or formatting.

## 38. Idempotent Updates

A good automation script should not create changes when the desired state already exists.

```python
if current_tag != desired_tag:
    update()
```

In GitOps this prevents unnecessary commits and unnecessary reconciliations.

## 39. Atomic Configuration Writes

For critical files:

```text
generate temporary file
      -> validate
      -> atomic replace
```

Python provides `os.replace()` for atomic replacement under appropriate filesystem conditions. Add cleanup handling for temporary files.

## 40. Secret Handling

Never hard-code credentials in JSON/YAML/CSV.

Prefer:

```text
AWS IAM roles
Kubernetes Secrets
external secret managers
CI/CD secret stores
```

Do not print full API responses or configuration objects if they may contain secrets.

## 41. Structured Logging

Prefer JSON logs when you need searchable fields:

```json
{
  "service": "payment",
  "level": "ERROR",
  "status": 500,
  "request_id": "abc123"
}
```

Structured logs reduce downstream regex parsing and work well with ELK.

## 42. Security Scan Reports

Security tools often provide JSON output. Parse it directly and separate:

```text
parser
   ->
normalized findings
   ->
policy
   ->
CI exit code
```

For critical security gates, machine-readable output is preferable to scraping human-readable text.

## 43. AWS Inventory Project

A practical project is an AWS inventory exporter:

```text
boto3
 -> EC2/EBS/RDS/EKS data
 -> normalize fields
 -> inventory.json
 -> inventory.csv
```

Include resource ID, region, environment, state, and selected tags. Avoid exporting unnecessary sensitive metadata.

## 44. EKS Inventory Project

Collect:

```text
cluster
namespace
pod
container
image
phase
ready
restart_count
```

Prefer Kubernetes API/client data or `kubectl -o json` rather than table scraping. Export normalized JSON/CSV for operations reporting.

## 45. GitOps Image Updater

Build `update-image.py`:

```text
input service + tag
 -> load values.yaml
 -> locate exact service
 -> update image
 -> validate
 -> compare
 -> write if changed
```

Then let Git review/merge and ArgoCD reconcile the cluster.

## 46. Deployment Auditor

Build an auditor that compares expected YAML with actual Kubernetes state.

Check:

```text
namespace
image
desired replicas
available replicas
container readiness
restart count
```

Output a structured report and return a meaningful non-zero exit code when policy is violated.

## 47. CI/CD Integration

A Python validator can run in Jenkins, GitHub Actions, or GitLab CI:

```bash
python scripts/validate_config.py
```

The script must return `0` for success and a documented non-zero value for failure. Never print an error and then accidentally exit successfully.

## 48. Error Handling

Use specific exceptions:

```python
try:
    data = json.load(file)
except json.JSONDecodeError as exc:
    raise ValueError("Invalid JSON") from exc
```

For YAML catch `yaml.YAMLError`; for files handle `OSError` where recovery or a clearer message is useful.

## 49. Large Data Performance

For large operational data:

```text
API -> paginate
CSV -> stream
JSON events -> JSON Lines
Kubernetes -> structured API
filter -> before expensive transformation
```

Avoid unnecessary format conversions and loading multi-gigabyte data into memory.

## 50. Schema Evolution

External APIs and platform responses can gain fields or change optional data. Use `.get()` for optional fields, but explicitly validate required fields and types.

Do not silently accept a breaking change in a field that controls a deployment.

## 51. Testing

Create fixtures:

```text
valid.json
invalid.json
deployment.yaml
multi-doc.yaml
services.csv
malformed.csv
```

Test valid input, invalid syntax, missing fields, wrong types, duplicates, secrets, and format changes.

## 52. Production Best Practices

Follow these rules:

```text
1. Parse structured data with real parsers.
2. Validate syntax and semantics separately.
3. Use yaml.safe_load().
4. Never use eval() for external data.
5. Prefer APIs/SDKs over CLI text scraping.
6. Use JSON output when a CLI must be used.
7. Stream large CSV files.
8. Use JSON Lines for large event streams.
9. Validate required fields and types.
10. Never log secrets.
11. Keep parsing separate from policy.
12. Make updates idempotent.
13. Validate before deployment.
14. Use dry runs.
15. Compare semantic state.
16. Use atomic writes for critical files.
17. Test malformed input.
18. Verify production account/cluster context.
19. Return meaningful CI exit codes.
20. Keep generated output deterministic.
```

## 53. Interview Questions

### Why use JSON/YAML parsers instead of regex?

> JSON and YAML are structured formats. A parser understands nesting, types, escaping, and syntax; regex only matches text patterns. I use regex for unstructured logs and parsers for structured configuration.

### How do you parse Kubernetes output?

> I prefer the Kubernetes API/client. If I use kubectl, I use `-o json` and parse structured data rather than scraping the human-readable table.

### How do you safely parse YAML?

> I use `yaml.safe_load()` and then validate the resulting object against required fields, types, and platform-specific rules.

### How do you process a huge CSV?

> I use `csv.DictReader()` and stream rows instead of loading the entire file into memory.

### How would you update an image tag in GitOps?

> Parse the YAML, locate the exact workload/container, update only the intended field, validate it, compare old and new state, and commit only when the desired state changed.

## 54. Scenario-Based Troubleshooting

### YAML is valid but Kubernetes deployment fails

Check:

```text
apiVersion
kind
schema
metadata
namespace
image
RBAC
admission policies
resource configuration
```

Use `helm lint`, `helm template`, and Kubernetes server-side dry-run as appropriate.

### kubectl parser suddenly breaks

Likely cause: human-readable output changed. Replace table scraping with `-o json` or the Kubernetes API.

### CSV contains `replicas=three`

Reject the row with a clear validation error. Never silently convert invalid input to a default.

### Secret appears in a generated report

Remove it, rotate it if exposure occurred, identify the source, and add a regression test preventing future leakage.

## Final Mental Model

```text
AWS / Kubernetes / CI/CD / Security / APIs
                    |
                    v
              JSON / YAML / CSV
                    |
                    v
                Python
                    |
        +-----------+-----------+
        |           |           |
      Parse      Validate    Transform
        |           |           |
        +-----------+-----------+
                    |
                    v
             Verify / Dry Run
                    |
                    v
              Deploy / Report
```

Remember:

```text
JSON -> APIs and machine-readable data
YAML -> Kubernetes, Helm, CI/CD, configuration
CSV  -> inventories, reports, tabular exports
```

Python connects them to real DevOps workflows.