# Jenkins Workspace

## Introduction

A **Jenkins Workspace** is a directory on the Jenkins Controller or Agent where a pipeline executes.

When a build starts, Jenkins checks out the source code into the workspace, performs compilation, testing, packaging, security scans, and generates build artifacts.

Every job gets its own workspace to isolate builds and avoid conflicts.

---

# Why Do We Need a Workspace?

Without a workspace, Jenkins would have no location to:

- Clone source code
- Store dependencies
- Execute build commands
- Generate reports
- Create artifacts

The workspace acts as the working directory for the entire pipeline.

---

# Build Without Workspace

```text
Developer Push

↓

Jenkins

↓

No Working Directory

↓

Cannot Clone Code

↓

Build Failed
```

---

# Build With Workspace

```text
Developer Push

↓

Jenkins

↓

Create Workspace

↓

Clone Repository

↓

Build

↓

Test

↓

Package

↓

Deploy
```

---

# What Does a Workspace Contain?

A typical workspace includes:

```text
Workspace

├── Source Code
├── pom.xml
├── package.json
├── Dockerfile
├── Jenkinsfile
├── target/
├── node_modules/
├── reports/
├── test-results/
└── build-artifacts/
```

---

# Workspace Creation

When a build starts:

```text
Pipeline Triggered

↓

Allocate Agent

↓

Create Workspace

↓

Checkout Source Code

↓

Execute Pipeline
```

If the workspace already exists, Jenkins reuses it unless configured otherwise.

---

# Default Workspace Location

Typical Linux location:

```text
/var/lib/jenkins/workspace/
```

Example:

```text
/var/lib/jenkins/workspace/catalog-service
```

Each Jenkins job has its own workspace directory.

---

# Workspace Architecture

```text
                   Jenkins Controller
                          │
              Schedule Pipeline
                          │
                          ▼
                  Jenkins Agent
                          │
                          ▼
                 Create Workspace
                          │
                          ▼
             Checkout Source Code
                          │
                          ▼
        Compile → Test → Scan → Package
                          │
                          ▼
                  Generate Artifacts
```

---

# Workspace During Pipeline

```text
Workspace

↓

Git Checkout

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP

↓

Docker Build

↓

Trivy Scan

↓

Deployment
```

Everything happens inside the workspace.

---

# Workspace on Different Agents

Each agent maintains its own workspace.

```text
Controller

│

├── Linux Agent
│      └── Workspace A
│
├── Docker Agent
│      └── Workspace B
│
└── Kubernetes Agent
       └── Workspace C
```

A workspace is **local to the agent** executing the build.

---

# Multiple Builds

Example:

```text
Build #120

↓

Workspace

↓

Compile

---------------------

Build #121

↓

Same Workspace

↓

Updated Source Code
```

Jenkins may reuse the workspace between builds unless it is cleaned.

---

# Workspace Cleanup

Old files can cause build failures.

Cleaning the workspace removes:

- Old binaries
- Temporary files
- Cached reports
- Previous artifacts

Example:

```text
Workspace

↓

cleanWs()

↓

Empty Workspace

↓

Fresh Build
```

---

# Enterprise CI/CD Flow

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Allocate Kubernetes Agent

↓

Create Workspace

↓

Git Checkout

↓

Maven Build

↓

JUnit

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Push Image to Amazon ECR

↓

Deploy to Amazon EKS

↓

Archive Artifacts

↓

Clean Workspace
```

---

# Workspace vs Repository

| Repository | Workspace |
|------------|-----------|
| Stores source code | Working directory |
| Located in GitHub/GitLab | Located on Jenkins Agent |
| Permanent | Temporary/Reused |
| Shared by developers | Used by Jenkins builds |

---

# Production Example

Java Microservice Pipeline

```text
GitHub

↓

Clone Repository

↓

Workspace

↓

mvn clean package

↓

target/app.jar

↓

Docker Build

↓

Image

↓

Amazon ECR

↓

Amazon EKS
```