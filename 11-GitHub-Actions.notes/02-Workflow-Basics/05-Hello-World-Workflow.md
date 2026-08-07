# Hello World Workflow

The **Hello World Workflow** is the simplest GitHub Actions workflow.

It demonstrates the fundamental concepts of GitHub Actions, including:

- Workflow
- Event
- Runner
- Job
- Step
- Action
- Shell Command

Although this example is simple, it introduces the workflow execution model that is used in enterprise CI/CD pipelines.

---

# Objective

Create a workflow that:

- Executes automatically when code is pushed.
- Allocates a GitHub-hosted runner.
- Prints a message.
- Completes successfully.

---

# Repository Structure

```text
sample-project/

│

├── src/

├── README.md

└── .github/

      └── workflows/

            hello-world.yml
```

GitHub automatically detects the workflow because it is stored inside the `.github/workflows` directory.

---

# Hello World Workflow

```yaml
name: Hello World Workflow

on:
  push:

jobs:

  hello:

    runs-on: ubuntu-latest

    steps:

      - name: Print Hello World

        run: echo "Hello, World!"
```

---

# Workflow Breakdown

The workflow consists of five sections.

```text
Workflow

↓

Trigger

↓

Runner

↓

Job

↓

Step
```

Let's understand each section.

---

# Workflow Name

```yaml
name: Hello World Workflow
```

The **name** field provides a friendly name displayed in the GitHub Actions UI.

Example:

```text
Actions

↓

Hello World Workflow
```

---

# Trigger

```yaml
on:

  push:
```

This workflow starts whenever code is pushed to the repository.

Example:

```text
Developer

↓

Git Push

↓

Workflow Starts
```

---

# Job

```yaml
jobs:

  hello:
```

The workflow contains one job named **hello**.

A workflow may contain multiple jobs, but this example uses only one.

---

# Runner

```yaml
runs-on: ubuntu-latest
```

GitHub allocates an Ubuntu virtual machine to execute the workflow.

Execution flow:

```text
Workflow

↓

Ubuntu Runner

↓

Execute Job
```

---

# Step

```yaml
steps:

  - name: Print Hello World
```

The job contains one step.

The step simply prints a message.

---

# Shell Command

```yaml
run: echo "Hello, World!"
```

GitHub executes the command on the runner.

Output:

```text
Hello, World!
```

---

# Complete Execution Flow

```text
Developer

↓

Push Code

↓

GitHub Event

↓

Workflow Detected

↓

Ubuntu Runner

↓

Hello Job

↓

Print Hello World

↓

Workflow Completed
```

This is the complete execution lifecycle of the Hello World workflow.

---

# Viewing Workflow Results

After pushing the workflow file:

1. Open the GitHub repository.
2. Click **Actions**.
3. Select **Hello World Workflow**.
4. Open the latest workflow run.
5. View the execution logs.

GitHub displays:

- Workflow status
- Job status
- Step logs
- Execution duration

---

# Expected Output

```text
Hello, World!
```

The workflow should complete successfully.

---

# Adding Multiple Steps

A workflow can contain multiple steps.

Example:

```yaml
steps:

  - name: Print Greeting

    run: echo "Hello"

  - name: Print Repository

    run: echo ${{ github.repository }}

  - name: Print Branch

    run: echo ${{ github.ref }}
```

Execution:

```text
Step 1

↓

Step 2

↓

Step 3
```

Steps execute sequentially.

---

# Running Multiple Commands

A single step can execute multiple commands.

Example:

```yaml
- name: Multiple Commands

  run: |

    pwd

    ls -la

    echo "Workflow Running"
```

The commands execute one after another.

---

# First Enterprise Workflow

Although "Hello World" is used for learning, enterprise teams quickly extend it into a basic CI pipeline.

```text
Developer

↓

Push Code

↓

Checkout Repository

↓

Display Repository Information

↓

Validate Environment

↓

Complete
```

This verifies that the runner is correctly configured before adding build and deployment stages.

---

# Enterprise Starter Workflow

Many organizations begin new repositories with a validation workflow.

```text
Developer Push

↓

Checkout Code

↓

Display Runner Details

↓

Display Branch

↓

Display Commit SHA

↓

Verify Required Files

↓

Success
```

Purpose:

- Validate GitHub Actions setup.
- Verify runner configuration.
- Confirm repository structure.
- Ensure workflows execute correctly.

This is often the first workflow committed to a new project.

---

# Production Validation Workflow

Before implementing a full CI/CD pipeline, DevOps teams usually verify the execution environment.

```text
Developer Push

↓

Workflow Starts

↓

Checkout Repository

↓

Print Branch Name

↓

Print Commit SHA

↓

Print Runner OS

↓

Verify Docker Installed

↓

Verify Java Installed

↓

Verify Terraform Installed

↓

Workflow Success
```

This simple validation avoids troubleshooting larger pipelines later.

---

# Best Practices

- Start with a simple workflow before building complex pipelines.
- Keep workflow names descriptive.
- Verify workflows in feature branches before merging to `main`.
- Use GitHub-hosted runners during initial development.
- Add one feature at a time.
- Review workflow logs after every execution.

---

# Common Mistakes

- Placing the workflow outside `.github/workflows/`.
- Incorrect YAML indentation.
- Using tabs instead of spaces.
- Forgetting the `on` trigger.
- Omitting `runs-on`.
- Assuming steps execute in parallel.

---

# Summary

The Hello World workflow is the foundation of GitHub Actions.

It introduces the essential workflow components:

- Workflow
- Trigger
- Runner
- Job
- Step
- Shell Command

Once this workflow executes successfully, additional stages such as building, testing, scanning, packaging, and deployment can be added to create complete enterprise CI/CD pipelines.

---

# Interview Questions

## Basic

1. What is the purpose of a Hello World workflow?
2. Where should a workflow file be stored?
3. What triggers this workflow?
4. Which runner is used in this example?
5. What does the `run` keyword do?

---

## Intermediate

1. Explain the execution flow of the Hello World workflow.
2. How can you add multiple steps to a job?
3. Where can you view workflow execution logs?
4. What happens after a developer pushes code?
5. Why do enterprise teams create validation workflows before implementing full CI/CD pipelines?

---

## Advanced

1. Extend the Hello World workflow into a production-ready validation workflow that verifies repository structure, runner configuration, required tools, and environment readiness before allowing developers to build full CI/CD pipelines.
2. Design a reusable starter workflow that every new repository in an organization can use to validate GitHub Actions setup before application-specific workflows are added.
3. Your team's first GitHub Actions workflow is not executing after a push. Describe your troubleshooting approach, including repository structure, workflow location, event configuration, runner allocation, and log analysis.