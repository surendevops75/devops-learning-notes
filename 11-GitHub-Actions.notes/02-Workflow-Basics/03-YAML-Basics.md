# YAML Basics

GitHub Actions workflows are written in **YAML (Yet Another Markup Language)**.

YAML is a human-readable data serialization language that uses **indentation** instead of brackets or braces to represent structure.

Before learning GitHub Actions, it is essential to understand YAML because every workflow is written using YAML syntax.

---

# Why YAML?

GitHub uses YAML because it is:

- Easy to read
- Easy to write
- Human friendly
- Lightweight
- Widely supported

Instead of writing complex XML or JSON, GitHub workflows are written using simple indentation.

---

# YAML vs JSON

## JSON

```json
{
  "name": "CI Pipeline"
}
```

---

## YAML

```yaml
name: CI Pipeline
```

YAML is cleaner and easier to understand.

---

# YAML Rules

YAML follows a few important rules.

- Indentation is mandatory.
- Spaces are used instead of tabs.
- Keys end with a colon (`:`).
- Lists begin with a hyphen (`-`).
- Parent-child relationships are defined using indentation.

Incorrect indentation is one of the most common causes of workflow failures.

---

# Indentation

Correct:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

Incorrect:

```yaml
jobs:
build:
runs-on: ubuntu-latest
```

GitHub will fail to parse the workflow.

---

# Key-Value Pairs

A key stores a value.

Example:

```yaml
name: Java CI Pipeline
```

Another example:

```yaml
runs-on: ubuntu-latest
```

General format:

```yaml
key: value
```

---

# Nested Keys

YAML allows keys inside other keys.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest
```

Hierarchy:

```text
jobs

↓

build

↓

runs-on
```

---

# Lists

Lists are represented using a hyphen (`-`).

Example:

```yaml
branches:

  - main

  - develop

  - release
```

This creates a list of branches.

---

# Maps (Objects)

Maps contain multiple key-value pairs.

Example:

```yaml
env:

  APP_NAME: catalogue

  ENV: qa

  AWS_REGION: us-east-1
```

The **env** object contains three variables.

---

# Strings

YAML supports strings with or without quotes.

Examples:

```yaml
name: CI Pipeline
```

```yaml
name: "CI Pipeline"
```

```yaml
name: 'CI Pipeline'
```

All are valid.

---

# Numbers

Example:

```yaml
timeout-minutes: 20
```

---

# Boolean Values

Example:

```yaml
continue-on-error: true
```

or

```yaml
continue-on-error: false
```

---

# Comments

Comments begin with `#`.

Example:

```yaml
# Build Stage

jobs:
```

Comments improve readability but are ignored during execution.

---

# Multi-Line Strings

Example:

```yaml
run: |
  echo "Build Started"

  mvn clean package

  echo "Build Completed"
```

The `|` preserves line breaks.

---

# Folded Strings

Example:

```yaml
run: >

  echo "This command

  becomes one line"
```

The `>` folds multiple lines into one line.

---

# Variables

Variables are declared as key-value pairs.

Example:

```yaml
env:

  JAVA_VERSION: 21

  APP_NAME: catalogue
```

---

# Arrays

Example:

```yaml
branches:

  - main

  - develop

  - release
```

This is an array of branch names.

---

# Complete YAML Example

```yaml
name: Java CI

on:

  push:

    branches:

      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout

        uses: actions/checkout@v4

      - name: Build

        run: mvn clean package
```

This demonstrates the basic YAML structure used in GitHub Actions.

---

# YAML Hierarchy

```text
Workflow

↓

Jobs

↓

Build

↓

Steps

↓

Checkout

↓

Build

↓

Test
```

Every child element is identified by its indentation.

---

# Common YAML Mistakes

## Using Tabs

❌ Incorrect

```yaml
jobs:
	build:
```

Always use spaces.

---

## Incorrect Indentation

❌ Incorrect

```yaml
jobs:

build:
```

The parser cannot determine the parent-child relationship.

---

## Missing Colon

❌ Incorrect

```yaml
name Java CI
```

Correct:

```yaml
name: Java CI
```

---

## Wrong List Format

❌ Incorrect

```yaml
branches:

main

develop
```

Correct:

```yaml
branches:

  - main

  - develop
```

---

# YAML Validation

Before committing workflows:

- Check indentation.
- Use spaces instead of tabs.
- Verify list formatting.
- Validate workflow syntax.
- Test the workflow in a non-production branch.

---

# Production Example

A simplified production deployment workflow.

```yaml
name: Production Deployment

on:

  workflow_dispatch:

jobs:

  deploy:

    runs-on: self-hosted

    steps:

      - name: Checkout

        uses: actions/checkout@v4

      - name: Deploy

        run: ./deploy.sh
```

This example shows how YAML defines the workflow structure for a production deployment.

---

# Production Workflow

```text
Developer

↓

workflow_dispatch

↓

Read YAML

↓

Allocate Runner

↓

Execute Jobs

↓

Execute Steps

↓

Deploy Production

↓

Health Check

↓

Complete
```

GitHub interprets the YAML file to execute each stage of the deployment.

---

# Best Practices

- Always use spaces for indentation.
- Keep indentation consistent.
- Use meaningful key names.
- Comment complex sections.
- Keep workflows readable.
- Validate YAML before merging.
- Store common values in environment variables.
- Avoid deeply nested structures.

---

# Common Mistakes

- Using tabs instead of spaces.
- Incorrect indentation.
- Missing colons.
- Incorrect list formatting.
- Mixing tabs and spaces.
- Writing large, unreadable YAML files.
- Hardcoding values throughout the workflow.

---

# Summary

YAML is the language used to define GitHub Actions workflows.

Understanding indentation, key-value pairs, lists, maps, and comments is essential for creating reliable workflows.

Since GitHub Actions depends entirely on YAML, mastering its syntax is the first step toward writing production-ready automation.

---

# Interview Questions

## Basic

1. What is YAML?
2. Why does GitHub Actions use YAML?
3. What is the difference between YAML and JSON?
4. How are lists represented in YAML?
5. Why is indentation important?

---

## Intermediate

1. Explain YAML key-value pairs.
2. What is the difference between `|` and `>` in YAML?
3. Why should tabs never be used in YAML?
4. What are common YAML syntax errors?
5. How do comments work in YAML?

---

## Advanced

1. Explain how GitHub parses YAML workflows.
2. Design a readable YAML structure for a large enterprise CI/CD workflow.
3. A production deployment workflow fails immediately due to a YAML parsing error. Describe your troubleshooting process and the common syntax mistakes you would check first.