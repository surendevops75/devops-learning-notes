# Terraform Automation with Python

## 1. Overview

Terraform is the primary Infrastructure as Code tool in many DevOps and cloud environments.

Python can automate Terraform workflows such as:

- Terraform configuration generation
- Variable file generation
- Workspace/environment management
- Terraform initialization
- Provider validation
- Formatting
- Validation
- Planning
- Applying approved changes
- Destroy workflows with safeguards
- Plan output inspection
- State inspection
- Drift analysis
- Backend validation
- AWS resource discovery
- CI/CD integration
- Terraform policy checks
- Multi-environment orchestration
- Infrastructure reporting

The production principle is:

> **Python should orchestrate and validate Terraform rather than bypass Terraform's state management and declarative model.**

A strong architecture is:

```text
Python Automation
       |
       v
Terraform Configuration
       |
       v
terraform init
       |
       v
terraform validate
       |
       v
terraform plan
       |
       v
Policy / Review
       |
       v
terraform apply
       |
       v
AWS / Azure / Cloud
```

---

# 2. Terraform + Python Architecture

```text
                         Python
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
   File Generation    Terraform CLI      Cloud APIs
          |                |                |
          v                v                v
    .tf / .tfvars       Terraform       boto3/Azure
                           |
                           v
                    State / Backend
                           |
                           v
                      Cloud Provider
```

Python has three common roles:

```text
1. Terraform orchestrator
2. Configuration generator
3. Cloud inventory / validation tool
```

---

# 3. Why Use Python with Terraform?

Terraform already provides an automation language.

So why Python?

Python is useful when the workflow requires:

```text
External APIs
Complex logic
Repository automation
Data transformation
Environment discovery
CI/CD integration
Validation
Reporting
Cross-system orchestration
```

Example:

```text
AWS API
   |
   v
Python
   |
   v
Generate tfvars
   |
   v
Terraform plan
```

---

# 4. When Not to Use Python

Do not use Python simply to replace:

```text
resource "aws_instance" ...
```

with generated strings unless there is a real requirement.

Prefer native Terraform for:

```text
Resource lifecycle
Dependencies
State
Modules
Providers
Outputs
Resource relationships
```

Use Python around Terraform for orchestration and integration.

---

# 5. Terraform Installation

Verify:

```bash
terraform version
```

Terraform should be installed through your organization's approved package/repository process.

Python:

```bash
python3 --version
```

Create virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install required Python libraries:

```bash
pip install boto3
```

For Terraform-specific Python wrappers, evaluate the library's maintenance and feature coverage before adopting it. Direct Terraform CLI invocation is often easier to reason about in production.

---

# 6. Terraform CLI vs Python Terraform Libraries

Common approaches:

```text
Python
 |
 +-- subprocess -> terraform CLI
 |
 +-- Terraform-specific library
 |
 +-- HCL parser/generator
```

For production orchestration, `subprocess` with a controlled Terraform CLI is often straightforward because:

```text
Terraform CLI is authoritative
Exit codes are clear
All Terraform features are available
Logs are familiar
```

---

# 7. Safe Terraform subprocess execution

Good:

```python
import subprocess


result = subprocess.run(
    [
        "terraform",
        "version"
    ],
    text=True,
    capture_output=True,
    check=True
)

print(result.stdout)
```

Avoid:

```python
subprocess.run(
    f"terraform {user_input}",
    shell=True
)
```

Use argument lists and validate external inputs.

---

# 8. Terraform Working Directory

Terraform commands must run against the correct configuration.

Example:

```python
from pathlib import Path


terraform_dir = Path(
    "/workspace/infrastructure"
)

if not (
    terraform_dir / "main.tf"
).exists():
    raise RuntimeError(
        "Terraform configuration not found"
    )
```

A production tool should verify the expected directory before executing Terraform.

---

# 9. Terraform Initialization

CLI:

```bash
terraform init
```

Python:

```python
subprocess.run(
    [
        "terraform",
        "init",
        "-input=false"
    ],
    cwd=terraform_dir,
    check=True,
    text=True
)
```

Initialization configures:

```text
Backend
Providers
Modules
Dependency lock
```

---

# 10. Why `-input=false` Matters

In CI/CD, interactive prompts can cause a pipeline to hang.

Use:

```bash
terraform init -input=false
```

Similarly:

```bash
terraform plan -input=false
```

and:

```bash
terraform apply -input=false
```

where appropriate.

CI should be non-interactive.

---

# 11. Terraform Backend

Your infrastructure architecture uses remote state.

Example:

```hcl
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "eks/prod/terraform.tfstate"
    region = "ap-south-1"
  }
}
```

The backend provides:

```text
Remote state storage
State locking where supported/configured
Centralized state
Team collaboration
```

Python should validate backend configuration but should not manipulate the state file directly.

---

# 12. S3 Backend Architecture

```text
Developer / CI
      |
      v
Terraform
      |
      v
S3 Backend
      |
      v
terraform.tfstate
      |
      v
AWS Infrastructure
```

Your production workflow should use the organization's approved state bucket, key structure, encryption, and access controls.

---

# 13. State Is Critical

Terraform state contains infrastructure mapping information.

It may also contain sensitive values depending on resource/provider behavior.

Therefore:

```text
Never commit terraform.tfstate
Never print state blindly
Never edit state manually without a controlled procedure
```

Python should treat Terraform state as sensitive infrastructure data.

---

# 14. Terraform Provider Lock File

Terraform commonly uses:

```text
.terraform.lock.hcl
```

Commit this file to version control for consistent provider selection.

Python automation should not casually delete it.

---

# 15. Terraform Format

Run:

```bash
terraform fmt -check -recursive
```

Python:

```python
subprocess.run(
    [
        "terraform",
        "fmt",
        "-check",
        "-recursive"
    ],
    cwd=terraform_dir,
    check=True,
    text=True
)
```

This is useful as a CI validation stage.

---

# 16. Terraform Validation

Run:

```bash
terraform validate
```

Python:

```python
subprocess.run(
    [
        "terraform",
        "validate",
        "-no-color"
    ],
    cwd=terraform_dir,
    check=True,
    text=True
)
```

Validation checks configuration consistency.

It does not replace:

```text
terraform plan
security scanning
policy validation
```

---

# 17. Terraform Plan

Plan:

```bash
terraform plan \
  -input=false \
  -out=tfplan
```

Python:

```python
subprocess.run(
    [
        "terraform",
        "plan",
        "-input=false",
        "-out=tfplan"
    ],
    cwd=terraform_dir,
    check=True,
    text=True
)
```

The plan answers:

```text
What will Terraform change?
```

---

# 18. Why Plan Is Important

Terraform plan may show:

```text
Create
Update
Destroy
Replace
No changes
```

Production automation should inspect the plan before applying.

Never assume:

```text
terraform plan succeeded
```

means:

```text
change is safe
```

---

# 19. Terraform Apply

Preferred production pattern:

```bash
terraform apply tfplan
```

after a reviewed plan.

Python:

```python
subprocess.run(
    [
        "terraform",
        "apply",
        "-input=false",
        "tfplan"
    ],
    cwd=terraform_dir,
    check=True,
    text=True
)
```

This is safer than allowing Terraform to generate a new plan during apply.

---

# 20. Plan Artifact Workflow

```text
terraform plan
      |
      v
tfplan
      |
      v
Review / Policy
      |
      v
terraform apply tfplan
```

This creates a clear relationship between:

```text
Approved plan
```

and:

```text
Applied changes
```

---

# 21. Convert Plan to JSON

Terraform supports:

```bash
terraform show -json tfplan
```

Python:

```python
result = subprocess.run(
    [
        "terraform",
        "show",
        "-json",
        "tfplan"
    ],
    cwd=terraform_dir,
    check=True,
    capture_output=True,
    text=True
)

plan_json = result.stdout
```

This can be parsed for policy and reporting.

---

# 22. Plan JSON Architecture

```text
Terraform Plan
      |
      v
terraform show -json
      |
      v
Python
      |
      +-- Resource changes
      +-- Actions
      +-- Risk checks
      +-- Reporting
      |
      v
Approval
```

This is a powerful use case for Python.

---

# 23. Detect Resource Actions

Plan JSON contains resource change information.

Conceptually:

```python
import json

plan = json.loads(
    plan_json
)

for resource in plan.get(
    "resource_changes",
    []
):
    actions = (
        resource["change"]["actions"]
    )

    print(
        resource["address"],
        actions
    )
```

Possible actions include:

```text
create
update
delete
replace
```

Exact plan structure should be handled according to the Terraform version being used.

---

# 24. Dangerous Change Detection

Python can inspect:

```text
delete
replace
large resource count
production resources
network changes
IAM changes
database changes
```

Example policy:

```text
If production plan contains
database replacement:
    require manual approval
```

This is much stronger than simply checking whether `terraform plan` exits successfully.

---

# 25. Plan Summary

Python can produce:

```text
Create: 5
Update: 3
Delete: 0
Replace: 1
```

Example:

```python
summary = {
    "create": 0,
    "update": 0,
    "delete": 0,
    "replace": 0
}
```

Then classify each resource action.

---

# 26. Resource-Level Policy

Example:

```text
aws_db_instance
```

may be considered high risk.

Python policy:

```python
HIGH_RISK = {
    "aws_db_instance",
    "aws_rds_cluster",
    "aws_vpc",
    "aws_iam_role"
}
```

If such resources are changed:

```text
Require approval
```

The exact policy must match the organization's infrastructure risk model.

---

# 27. Production Infrastructure Policy

A practical policy engine:

```text
Plan
 |
 +-- Destroy? --------> Approval
 |
 +-- Replace? --------> Approval
 |
 +-- IAM change? -----> Security review
 |
 +-- Network change? -> Network review
 |
 +-- DB change? ------> DBA/platform review
 |
 v
Normal change
```

Python can implement this decision layer.

---

# 28. Terraform Variables

Example:

```hcl
variable "environment" {
  type = string
}
```

Python can generate:

```text
terraform.tfvars
```

Example:

```python
variables = {
    "environment": "prod",
    "instance_type": "t3.medium"
}
```

---

# 29. Generating tfvars

```python
from pathlib import Path


content = 