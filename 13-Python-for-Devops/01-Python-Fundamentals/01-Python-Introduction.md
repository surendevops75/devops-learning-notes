# Python Fundamentals

## Python Introduction

Python is a high-level, general-purpose programming language widely used in DevOps for automation, infrastructure operations, cloud APIs, Kubernetes, CI/CD, monitoring, testing, reporting and operational tooling.

This curriculum treats Python as a **DevOps automation language**, not as a generic software-development course.

Core DevOps uses:

```text
Linux automation
AWS automation
Kubernetes/EKS automation
CI/CD integration
REST API automation
Configuration processing
Log processing
Health checks
Backup automation
Cleanup automation
Monitoring
Testing
Security automation
```


---

# 2. Why Python Matters for DevOps

DevOps engineers repeatedly perform operational tasks that are easy to automate.

Manual:

```text
Find servers
 -> connect
 -> inspect health
 -> collect results
 -> identify failures
 -> create report
```

Automated:

```text
Python
 -> discover resources
 -> collect data
 -> process results
 -> apply thresholds
 -> log findings
 -> return success/failure
```

Python becomes especially valuable when a Bash script starts requiring complex branching, JSON processing, API calls, reusable functions, testing or sophisticated error handling.


---

# 3. Python vs Bash

Use Bash for simple operating-system tasks:

```bash
df -h
systemctl status nginx
kubectl get pods
```

Use Python when you need:

```text
Complex logic
JSON/YAML processing
AWS SDKs
Kubernetes SDK
REST APIs
Reusable modules
Testing
Structured error handling
Data processing
```

A practical rule:

> Use the simplest tool that makes the automation reliable and maintainable.


---

# 4. Installing and Verifying Python

Check the installed version:

```bash
python3 --version
which python3
```

Start the interactive interpreter:

```bash
python3
```

Then:

```python
print("Hello DevOps")
```

Exit with:

```python
exit()
```

or `Ctrl+D`.

Create a script:

```bash
touch hello.py
```

```python
print("Hello DevOps")
```

Run:

```bash
python3 hello.py
```


---

# 5. Python Syntax and Indentation

Python uses indentation to define blocks.

Correct:

```python
if disk_usage > 80:
    print("Disk usage is high")
```

Incorrect:

```python
if disk_usage > 80:
print("Disk usage is high")
```

Use four spaces consistently. Do not mix tabs and spaces.

Readable DevOps automation is important because scripts are frequently maintained during incidents.


---

# 6. Comments and Documentation

Single-line comment:

```python
# Check disk utilization
```

Prefer comments that explain **why**:

```python
# Avoid deploying when production disk utilization is above the safety threshold.
if disk_usage > 85:
    ...
```

For reusable functions, use docstrings:

```python
def check_disk(path):
    """Return disk utilization for the supplied filesystem."""
    ...
```


---

# 7. Variables

Variables hold values:

```python
environment = "production"
region = "ap-south-1"
replicas = 3
deployment_status = "SUCCESS"
```

Prefer descriptive names:

```python
deployment_status = "FAILED"
```

instead of:

```python
x = "FAILED"
```

Typical DevOps variables represent:

```text
environment
cluster_name
namespace
service_name
region
replica_count
timeout
retry_count
```


---

# 8. Python Data Types

Important built-in types:

```text
str
int
float
bool
list
tuple
set
dict
None
```

Examples:

```python
service = "payment"
replicas = 3
latency = 1.25
healthy = True
error = None
```

Check a type:

```python
print(type(replicas))
```


---

# 9. Strings

Strings contain text:

```python
service = "payment-service"
environment = "production"
```

Useful operations:

```python
service.upper()
service.lower()
service.startswith("payment")
service.endswith("service")
```

String formatting:

```python
print(f"{service} is running in {environment}")
```

Strings are heavily used for:

```text
Logs
API responses
Commands
Configuration
File paths
Resource names
```


---

# 10. Numbers and Booleans

Integers:

```python
replicas = 3
port = 8080
```

Floats:

```python
cpu_usage = 72.5
latency = 1.42
```

Booleans:

```python
deployment_successful = True
```

Arithmetic:

```python
success = total - failed
percentage = (failed / total) * 100
```

Comparison:

```python
if cpu_usage > 80:
    print("High CPU")
```


---

# 11. Type Conversion

Environment variables and many API inputs arrive as strings.

```python
import os

retry_count = int(os.getenv("RETRY_COUNT", "3"))
timeout = float(os.getenv("TIMEOUT", "10"))
```

Other conversions:

```python
str(8080)
int("8080")
float("75.5")
bool(value)
```

Always validate external input before converting when failure is possible.


---

# 12. Operators

Arithmetic:

```text
+  -  *  /  %  //  **
```

Comparison:

```text
==  !=  >  <  >=  <=
```

Logical:

```text
and
or
not
```

Membership:

```text
in
not in
```

Identity:

```text
is
is not
```

Example:

```python
if cpu > 80 and memory > 80:
    print("Resource pressure detected")
```

Use `is None` for checking `None`:

```python
if result is None:
    ...
```


---

# 13. Conditional Statements

Basic:

```python
if status == "FAILED":
    print("Deployment failed")
```

With alternatives:

```python
if health == "healthy":
    print("Healthy")
else:
    print("Unhealthy")
```

Multiple thresholds:

```python
if cpu >= 90:
    print("CRITICAL")
elif cpu >= 80:
    print("WARNING")
else:
    print("NORMAL")
```

This pattern is common in health-check automation.


---

# 14. DevOps Threshold Example

```python
disk_usage = 92

if disk_usage >= 90:
    print("CRITICAL: Disk usage above 90%")
elif disk_usage >= 80:
    print("WARNING: Disk usage above 80%")
else:
    print("Disk usage is normal")
```

A production version should additionally use logging, configuration, exception handling and a meaningful exit code.


---

# 15. Lists

A list stores an ordered collection:

```python
services = [
    "user",
    "product",
    "order",
    "payment"
]
```

Access:

```python
print(services[0])
```

Common operations:

```python
services.append("inventory")
services.remove("inventory")
len(services)
```

Lists are useful for:

```text
Services
Servers
Namespaces
Regions
Files
Packages
```


---

# 16. Iterating Over Lists

```python
services = ["user", "order", "payment"]

for service in services:
    print(f"Checking {service}")
```

A common automation pattern is:

```text
Get resources
 -> loop over resources
 -> perform check
 -> collect result
```

Avoid duplicating the same code for every server or service.


---

# 17. Tuples and Sets

Tuple:

```python
server = ("web-01", "10.0.1.10")
```

Tuples are useful for fixed collections.

Set:

```python
environments = {"dev", "stage", "prod"}
```

Sets contain unique values and are useful for:

```text
Unique IP addresses
Unique service names
Unique error types
Unique namespaces
```


---

# 18. Dictionaries

Dictionaries store key-value data:

```python
server = {
    "name": "web-01",
    "ip": "10.0.1.10",
    "environment": "production"
}
```

Access:

```python
server["name"]
```

Safer lookup:

```python
server.get("status")
```

Update:

```python
server["status"] = "healthy"
```

Dictionaries are extremely important because JSON and API responses map naturally to Python dictionaries.


---

# 19. Nested Dictionaries

Cloud and Kubernetes APIs often return nested structures:

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
deployment["metadata"]["name"]
deployment["spec"]["replicas"]
```

For production API code, use safe access patterns when fields may be absent.


---

# 20. Loops

For loop:

```python
for server in servers:
    print(server)
```

While loop:

```python
count = 0

while count < 3:
    print(count)
    count += 1
```

DevOps automation normally uses `for` loops for resources returned by APIs.

`break` stops a loop:

```python
for server in servers:
    if server == "web-02":
        break
```

`continue` skips the current iteration:

```python
for server in servers:
    if server == "web-02":
        continue
    print(server)
```


---

# 21. Functions

Functions make automation reusable:

```python
def check_service(service, status):
    if status == "healthy":
        return f"{service} is healthy"
    return f"{service} requires investigation"
```

Call:

```python
print(check_service("payment", "healthy"))
```

Prefer small functions with one responsibility:

```text
get_instances()
check_health()
send_report()
validate_config()
```

Avoid a single `do_everything()` function.


---

# 22. Return Values

Functions can return data:

```python
def check_cpu(cpu):
    return cpu <= 80

healthy = check_cpu(75)

if healthy:
    print("CPU is healthy")
```

Returning values is preferable to hiding all logic inside print statements because callers can make decisions, test the function and reuse the result.


---

# 23. Exceptions

DevOps scripts interact with systems that can fail:

```text
Network
AWS API
Kubernetes API
Files
Commands
Databases
```

Use exceptions:

```python
try:
    value = int("abc")
except ValueError:
    print("Invalid number")
```

Prefer specific exceptions where possible.

A broad:

```python
except Exception:
```

should be used deliberately, normally at an application/script boundary where the failure can be logged and handled safely.


---

# 24. finally and Cleanup

`finally` executes regardless of success or failure:

```python
try:
    print("Running operation")
except Exception as error:
    print(error)
finally:
    print("Cleanup")
```

Useful for:

```text
Closing files
Closing connections
Removing temporary files
Releasing resources
```


---

# 25. Modules and Imports

A module is reusable Python code.

```python
import os
import json
import logging
```

Or:

```python
from pathlib import Path
```

Important standard-library modules for DevOps include:

```text
os
sys
subprocess
pathlib
shutil
json
csv
re
datetime
time
logging
argparse
socket
```

External libraries later in this curriculum include:

```text
boto3
requests
PyYAML
kubernetes
pytest
```


---

# 26. os Module and Environment Variables

The `os` module provides operating-system functionality.

```python
import os

print(os.getcwd())
```

Environment variables:

```bash
export ENVIRONMENT=production
```

Python:

```python
environment = os.getenv("ENVIRONMENT")
print(environment)
```

Default:

```python
environment = os.getenv("ENVIRONMENT", "development")
```

Environment variables are useful for non-secret configuration. Sensitive values should be handled through appropriate secret-management mechanisms.


---

# 27. sys Module and Exit Codes

`sys` provides runtime and process functionality.

```python
import sys

print(sys.version)
```

Exit codes are critical in CI/CD:

```python
if deployment_failed:
    sys.exit(1)

sys.exit(0)
```

Convention:

```text
0     success
nonzero failure
```

Pipeline:

```text
Python validation
    |
    +--> exit 0 --> continue
    |
    +--> exit 1 --> fail
```


---

# 28. subprocess

Use `subprocess` to execute external commands.

```python
import subprocess

result = subprocess.run(
    ["df", "-h"],
    capture_output=True,
    text=True,
    check=True
)

print(result.stdout)
```

It can automate:

```text
git
docker
kubectl
terraform
systemctl
aws CLI
Linux commands
```

Prefer argument lists over shell command strings when input can be influenced externally.


---

# 29. pathlib

`pathlib` is the preferred modern filesystem interface.

```python
from pathlib import Path

log_file = Path("/var/log/application.log")

if log_file.exists():
    print("Log exists")
```

Create directories safely:

```python
Path("/opt/app").mkdir(parents=True, exist_ok=True)
```

The `exist_ok=True` behavior makes this operation safe to repeat.


---

# 30. JSON

JSON is heavily used in DevOps APIs.

Convert Python data to JSON:

```python
import json

data = {
    "service": "payment",
    "replicas": 3
}

print(json.dumps(data, indent=2))
```

Parse JSON:

```python
text = '{"service": "payment", "replicas": 3}'

data = json.loads(text)

print(data["service"])
```

JSON handling becomes essential for AWS, REST APIs and automation reports.


---

# 31. YAML

YAML is common in:

```text
Kubernetes
Helm
GitHub Actions
GitLab CI
Docker Compose
Ansible
```

With PyYAML:

```python
import yaml

with open("deployment.yaml") as file:
    data = yaml.safe_load(file)
```

Use safe loaders for untrusted YAML. Never deserialize untrusted YAML with unsafe loading mechanisms.


---

# 32. Regular Expressions

Regex is useful for:

```text
Log parsing
Version extraction
IP extraction
Error detection
Validation
```

Example:

```python
import re

text = "server returned HTTP 500"

match = re.search(r"HTTP (\d+)", text)

if match:
    print(match.group(1))
```

Do not use complex regex when a simple string operation is clearer.


---

# 33. Logging

Production automation should use the `logging` module instead of relying only on `print()`.

```python
import logging

logging.basicConfig(level=logging.INFO)

logging.info("Deployment started")
logging.warning("High CPU detected")
logging.error("Deployment failed")
```

Useful levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Production logs should contain enough context to identify what happened, where, and why.


---

# 34. Virtual Environments and pip

Create an isolated environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install packages:

```bash
pip install boto3 requests PyYAML
```

Deactivate:

```bash
deactivate
```

A `requirements.txt` can declare dependencies:

```text
boto3
requests
PyYAML
pytest
```

Install:

```bash
pip install -r requirements.txt
```

Pin versions when reproducible builds require it.


---

# 35. Command-Line Arguments

Operational scripts should avoid hardcoding every input.

Example:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--environment",
    required=True
)

args = parser.parse_args()

print(f"Deploying to {args.environment}")
```

Run:

```bash
python deploy.py --environment production
```

`argparse` will be covered more deeply later.


---

# 36. Configuration Strategy

Prefer configuration through:

```text
Environment variables
Command-line arguments
Configuration files
Secret managers
```

Example:

```python
import os

region = os.getenv("AWS_REGION", "ap-south-1")
```

Do not hardcode:

```python
password = "MyPassword123"
```

For production credentials, use an appropriate secret manager or workload identity mechanism.


---

# 37. Idempotency

Idempotency is essential for reliable DevOps automation.

Example:

```python
from pathlib import Path

Path("/opt/app").mkdir(parents=True, exist_ok=True)
```

Running the operation repeatedly still results in the desired directory state.

Non-idempotent behavior can cause:

```text
Duplicate configuration
Repeated resources
Duplicate records
Unexpected side effects
```

Always ask:

> What happens if this script runs twice?


---

# 38. Dry Run and Safety

Destructive automation should support a dry-run where practical.

Example:

```bash
python cleanup.py --dry-run
```

Output:

```text
Would delete:
- old-image-1
- old-image-2
```

No destructive action occurs.

Other safeguards:

```text
Validate environment
Confirm resource ownership
Use allowlists
Use least privilege
Log planned actions
Require explicit destructive mode
```


---

# 39. Retries and Timeouts

Transient failures are normal in distributed systems:

```text
Network timeout
API throttling
Temporary unavailable service
Connection reset
```

A reliable script may use:

```text
Timeout
 -> retry
 -> backoff
 -> retry
 -> fail safely
```

Do not retry everything. Authentication failures, invalid requests and permanent configuration errors usually should not be blindly retried.

Later topics will cover exponential backoff in detail.


---

# 40. Security

Production Python automation should:

```text
Never hardcode secrets
Validate external input
Use least privilege
Avoid shell injection
Protect sensitive logs
Use TLS
Handle credentials safely
Limit permissions
```

Dangerous:

```python
import os
os.system(f"kubectl get pods {user_input}")
```

Prefer structured arguments:

```python
import subprocess

subprocess.run(
    ["kubectl", "get", "pods", user_input],
    check=True
)
```

Also validate `user_input` against expected values.


---

# 41. Python in CI/CD

Python can perform:

```text
Pre-deployment validation
Configuration validation
Security checks
Artifact verification
Deployment orchestration
Post-deployment health checks
```

Example:

```text
Git
 |
 v
CI
 |
 +--> Python validation
 +--> Tests
 +--> Security scan
 +--> Build
 +--> Deploy
 +--> Python health check
```

The script's exit code determines whether the pipeline continues.


---

# 42. Python and Terraform

Terraform should remain the primary declarative infrastructure tool when it fits the problem.

Python can complement Terraform for:

```text
Pre-validation
Post-deployment checks
AWS API queries
Custom reporting
Operational workflows
```

Avoid replacing declarative infrastructure management with a large custom Python provisioning script without a strong reason.


---

# 43. Python and Kubernetes

Python can interact with Kubernetes in two broad ways:

```text
kubectl through subprocess
Kubernetes Python client
```

For simple command orchestration, subprocess can be sufficient.

For structured, reusable applications, prefer the Kubernetes client because it provides programmatic access to Kubernetes resources.

Later files will cover:

```text
Pods
Deployments
Services
Ingress
ConfigMaps
Secrets
Troubleshooting
EKS automation
```


---

# 44. Python and AWS

The primary AWS SDK for Python is:

```text
boto3
```

Future automation examples include:

```text
List EC2 instances
Inspect EKS
Create S3 backups
Generate resource reports
Find unused resources
Inspect RDS
Manage snapshots
```

Use IAM least privilege and avoid embedding AWS access keys in code.


---

# 45. Python and REST APIs

Typical API automation:

```text
Python
 |
 v
HTTP request
 |
 v
REST API
 |
 v
JSON response
 |
 v
Python processing
```

A common library is:

```text
requests
```

Production API clients should consider:

```text
Timeouts
Status codes
Retries
Authentication
Rate limits
Logging
Response validation
```


---

# 46. Python and Monitoring

Python can support monitoring by:

```text
Calling monitoring APIs
Running health checks
Processing metrics
Parsing logs
Generating reports
Triggering controlled remediation
```

Example:

```text
Disk usage > threshold
       |
       v
Python health checker
       |
       v
Identify server
       |
       v
Log/report/notify
```

Automatic remediation should be introduced only when the condition and corrective action are well understood.


---

# 47. Python and Backups

A production backup workflow can be:

```text
Create backup
 |
 v
Validate backup
 |
 v
Compress if appropriate
 |
 v
Upload to S3
 |
 v
Verify upload
 |
 v
Log result
 |
 v
Notify
```

A backup script should also consider:

```text
Retention
Encryption
Integrity
Permissions
Failure handling
Restore testing
```


---

# 48. Python and Cleanup Automation

Typical cleanup tasks:

```text
Old files
Old logs
Unused artifacts
Old Docker images
Unused cloud resources
Expired temporary objects
```

Destructive cleanup must use:

```text
Dry run
Allowlist/filters
Age threshold
Environment checks
Ownership checks
Logging
```

Never assume that an old resource is automatically safe to delete.


---

# 49. Production Script Lifecycle

A useful production pattern is:

```text
Configuration
     |
     v
Validation
     |
     v
Input collection
     |
     v
Processing
     |
     v
Action
     |
     v
Logging
     |
     v
Result
     |
     v
Exit code
```

This structure works for:

```text
Deployment validation
Health checks
AWS reporting
Kubernetes automation
Backup scripts
Cleanup jobs
```


---

# 50. Practical Project — Server Health Checker

Goal:

```text
Check CPU
Check memory
Check disk
Return health status
```

Concept:

```text
Start
 |
 v
Collect metrics
 |
 v
Compare thresholds
 |
 v
Log results
 |
 v
Exit 0/1
```

Later this project can use:

```text
psutil
logging
argparse
```

The important design is that the script should be usable by a CI/CD job or scheduler, not only by a developer at a terminal.


---

# 51. Practical Project — Kubernetes Pod Checker

Goal:

```text
Find unhealthy pods
```

Workflow:

```text
Connect to cluster
 |
 v
List pods
 |
 v
Inspect status
 |
 +--> Running/Ready
 |
 +--> Pending
 |
 +--> CrashLoopBackOff
 |
 +--> ImagePullBackOff
 |
 v
Report
```

A production implementation should avoid treating every non-Running phase as an incident without understanding the workload type.


---

# 52. Practical Project — AWS Resource Reporter

Workflow:

```text
Python
 |
 v
Boto3
 |
 v
AWS API
 |
 v
Collect resources
 |
 v
Filter by tags/environment
 |
 v
Generate report
```

The report can include:

```text
Resource ID
Name
Region
Environment
State
Owner
```

Tag-based filtering is essential in enterprise environments.


---

# 53. Practical Project — CI/CD Deployment Validator

After a deployment:

```text
Python validator
 |
 +--> Desired replicas?
 +--> Ready replicas?
 +--> Pods healthy?
 +--> Service reachable?
 +--> Error rate acceptable?
 |
 v
exit 0 / exit 1
```

This provides a post-deployment gate and can prevent a pipeline from declaring success when the application is not actually healthy.


---

# 54. Common Beginner Mistakes

Avoid:

```text
Hardcoded secrets
Huge scripts
One giant function
No logging
No exception handling
No input validation
No timeout
Blind retries
Non-idempotent operations
Ignoring exit codes
Using shell strings with untrusted input
Over-engineering tiny scripts
```

The goal is not maximum code. The goal is reliable automation.


---

# 55. DevOps Python Mental Model

Remember this pattern:

```text
Read configuration
       |
       v
Validate input
       |
       v
Call system/API
       |
       v
Parse response
       |
       v
Apply logic
       |
       v
Perform safe action
       |
       v
Log result
       |
       v
Return exit code
```

This pattern appears repeatedly in real DevOps automation.


---

# 56. Interview — Why Python for DevOps?

Strong answer:

> "I use Python mainly as an automation and integration language. It is useful when shell scripting becomes difficult to maintain or when I need to interact with APIs such as AWS and Kubernetes. I can use Python for health checks, CI/CD validation, AWS automation, log processing and operational tooling. I also use practices such as logging, exception handling, input validation, idempotency, retries and proper exit codes so the automation is reliable in production."


---

# 57. Interview — Python vs Bash

Strong answer:

> "I use Bash for straightforward Linux commands and simple system tasks. I prefer Python when the automation involves complex logic, structured data such as JSON, API calls, AWS or Kubernetes SDKs, reusable functions, testing or more sophisticated error handling."


---

# 58. Interview — How Does Python Fail a CI/CD Pipeline?

A Python script returns a non-zero exit code:

```python
import sys

sys.exit(1)
```

Pipeline behavior:

```text
Validation
 |
 +--> exit 0 --> continue
 |
 +--> exit 1 --> fail
```

This makes Python useful as a deployment validation or quality gate.


---

# 59. Interview — How Do You Handle Secrets?

Strong answer:

> "I don't hardcode secrets in Python source code. I use environment variables for suitable runtime configuration and preferably a secret-management mechanism such as AWS Secrets Manager or Kubernetes secret facilities with appropriate access controls. I also make sure secrets are not written into logs and that the workload has only the permissions it needs."


---

# 60. Interview — How Do You Make Automation Safe?

Answer:

```text
Validate input
Use least privilege
Use dry-run where appropriate
Make operations idempotent
Use timeouts
Handle exceptions
Log actions
Return correct exit codes
Test before production
```

Also ask:

> What happens if the script is interrupted halfway through?


---

# 61. Interview — What Is Idempotency?

Strong answer:

> "Idempotency means repeated execution results in the same desired state without unintended duplicate side effects. For example, creating a directory with `exist_ok=True` is safe to repeat. Blindly appending configuration on every execution may not be idempotent."


---

# 62. Interview — How Do You Debug Python Automation?

Use:

```text
Read traceback
Check logs
Validate inputs
Inspect API responses
Check permissions
Check environment variables
Check dependency versions
Reproduce a smaller case
```

For production automation, structured logging is preferable to temporary print statements.


---

# 63. Interview — What Makes a Python Script Production Ready?

Answer:

```text
Clear configuration
Input validation
Logging
Exception handling
Timeouts
Retries where appropriate
Idempotency
Security
Testing
Documentation
Monitoring
Correct exit codes
```

Not every script needs a large framework. The implementation should match operational risk.


---

# 64. Interview — Important Python Modules for DevOps

Core standard-library modules:

```text
os
sys
subprocess
pathlib
shutil
json
csv
re
datetime
time
logging
argparse
socket
```

Common external libraries:

```text
boto3
requests
PyYAML
kubernetes
pytest
```


---

# 65. Interview — What Would You Automate With Python?

Good examples from a DevOps environment:

```text
EC2 inventory/reporting
S3 backup validation
EKS pod health checks
Deployment validation
Log analysis
API integrations
Configuration validation
Resource cleanup
Infrastructure health checks
CI/CD quality gates
```

Choose examples that demonstrate measurable operational value rather than automation for its own sake.


---

# 66. Final Takeaway

You do not need to learn Python as if your immediate goal were to become a backend Python developer.

For DevOps, focus on:

```text
Python syntax
+
Linux automation
+
AWS/Boto3
+
Kubernetes
+
APIs
+
CI/CD
+
Testing
+
Reliability
+
Security
```

The DevOps Python mindset is:

```text
Can this be automated?
       |
       v
What inputs are required?
       |
       v
What API/command should I use?
       |
       v
What can fail?
       |
       v
How do I handle failure safely?
       |
       v
How do I verify the result?
       |
       v
How do I integrate it into operations?
```

