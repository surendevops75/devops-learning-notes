# Terraform Versioning

## Introduction

Terraform itself evolves continuously.

New versions introduce:

```text
New Features

Bug Fixes

Performance Improvements

Security Enhancements
```

---

Sometimes new versions also:

```text
Deprecate Old Features

Remove Old Syntax

Change Behaviors
```

---

Because of this, Terraform projects should explicitly define:

```text
Supported Terraform Version
```

---

# What is Terraform Version?

Terraform Version refers to:

```text
Terraform CLI Version
```

installed on your system.

---

Check version:

```bash
terraform version
```

---

Example:

```text
Terraform v1.8.5
```

---

# Why Versioning Matters?

Suppose project was developed using:

```text
Terraform 1.8
```

---

New engineer installs:

```text
Terraform 0.12
```

---

Result:

```text
Syntax Errors

Unsupported Features

Failed Deployments
```

---

# Required Version

Terraform allows us to specify:

```text
Minimum

Maximum

Allowed
```

versions.

---

Example:

```hcl
terraform {

  required_version = ">= 1.5.0"

}
```

---

Meaning:

```text
Terraform Version Must Be

1.5.0 Or Higher
```

---

# Exact Version

Example:

```hcl
terraform {

  required_version = "= 1.8.5"

}
```

---

Meaning:

```text
Only Version 1.8.5 Allowed
```

---

# Greater Than

Example:

```hcl
required_version = "> 1.5.0"
```

---

Meaning:

```text
Any Version Above 1.5.0
```

---

# Less Than

Example:

```hcl
required_version = "< 2.0.0"
```

---

Meaning:

```text
Terraform 2.x Not Allowed
```

---

# Range Constraint

Example:

```hcl
required_version = ">= 1.5.0, < 2.0.0"
```

---

Meaning:

```text
Allow Terraform 1.x

Reject Terraform 2.x
```

---

# Semantic Versioning

Terraform follows:

```text
Major.Minor.Patch
```

---

Example:

```text
1.8.5
```

---

Breakdown:

```text
1 → Major

8 → Minor

5 → Patch
```

---

# Major Version

Example:

```text
1.x → 2.x
```

---

May introduce:

```text
Breaking Changes
```

---

# Minor Version

Example:

```text
1.7 → 1.8
```

---

Usually adds:

```text
New Features
```

---

# Patch Version

Example:

```text
1.8.4 → 1.8.5
```

---

Typically includes:

```text
Bug Fixes

Security Fixes
```

---

# Pessimistic Constraint (~>)

Most commonly used.

---

Example:

```hcl
required_version = "~> 1.8"
```

---

Meaning:

```text
Allow

1.8.x

Reject

1.9.x
```

---

Example:

```hcl
required_version = "~> 1.8.0"
```

---

Meaning:

```text
1.8.0

1.8.1

1.8.2

1.8.10
```

Allowed.

---

Not Allowed:

```text
1.9.0
```

---

# Version Validation

Suppose project requires:

```hcl
required_version = ">= 1.8.0"
```

---

Engineer runs:

```text
Terraform 1.4
```

---

Result:

```text
terraform init fails
```

---

Example Error

```text
Unsupported Terraform Version
```

---

# Why Organizations Pin Versions?

Benefits:

```text
Predictability

Stability

Consistency

Repeatable Deployments
```

---

Without version pinning:

```text
Different Engineers

Different Versions

Different Results
```

---

# Production Example

DevOps Team:

```text
10 Engineers
```

---

Without version constraints:

```text
Terraform 1.5

Terraform 1.6

Terraform 1.7

Terraform 1.8
```

---

Different behavior possible.

---

Version pinning ensures:

```text
Same Experience For Everyone
```

---

# Terraform Upgrade Strategy

Current:

```text
Terraform 1.5
```

---

Target:

```text
Terraform 1.8
```

---

Steps:

```text
Upgrade DEV

Run Plan

Validate

Upgrade QA

Validate

Upgrade PROD
```

---

Never directly upgrade:

```text
Production First
```

---

# Architecture

```text
Terraform Code
       ↓
Required Version Check
       ↓
Version Compatible
       ↓
Execution
```

---

# Common Interview Questions

## What is required_version?

### Short Answer

required_version defines the Terraform CLI versions allowed to execute the configuration.

### Detailed Explanation

Terraform validates the installed version before executing commands.

### Production Example

Restricting execution to Terraform versions between 1.5 and 2.0.

### Follow-Up Questions

- Why is version pinning important?
- What happens if versions mismatch?

---

## What is Semantic Versioning?

### Short Answer

Semantic Versioning uses Major.Minor.Patch format.

### Detailed Explanation

Major versions may introduce breaking changes, minor versions add features, and patch versions fix bugs.

### Production Example

Terraform 1.8.5.

### Follow-Up Questions

- What does a major version change mean?
- What is a patch release?

---

## What Does ~> Mean?

### Short Answer

It is a pessimistic version constraint.

### Detailed Explanation

It allows compatible updates while preventing unexpected upgrades.

### Production Example

~> 1.8 allows 1.8.x but blocks 1.9.x.

### Follow-Up Questions

- Why is ~> commonly used?
- How is it different from >=?

---

# Key Takeaways

```text
Terraform version refers to the Terraform CLI version.

required_version controls which versions can execute a project.

Terraform follows Semantic Versioning.

Major versions may introduce breaking changes.

Minor versions usually introduce new features.

Patch versions typically contain bug fixes.

~> is the most commonly used version constraint.

Version pinning improves stability and consistency.

Terraform validates version compatibility before execution.

Production environments should always define required_version.
```