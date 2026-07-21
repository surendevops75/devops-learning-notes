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

---

# Workspace Structure

A Jenkins workspace contains all the files required during pipeline execution.

It includes the application source code, configuration files, dependencies, test reports, security scan reports, and generated artifacts.

---

# Typical Workspace Structure

Java Application

```text
workspace/

├── Jenkinsfile
├── pom.xml
├── Dockerfile
├── src/
├── target/
├── reports/
├── dependency-check-report/
├── trivy-report/
├── .git/
└── README.md
```

---

Node.js Application

```text
workspace/

├── Jenkinsfile
├── package.json
├── package-lock.json
├── Dockerfile
├── src/
├── dist/
├── node_modules/
├── reports/
└── .git/
```

---

Python Application

```text
workspace/

├── Jenkinsfile
├── requirements.txt
├── app.py
├── Dockerfile
├── tests/
├── reports/
├── venv/
└── .git/
```

---

# Workspace Lifecycle

A workspace goes through multiple stages during a pipeline.

```text
Pipeline Triggered

↓

Allocate Agent

↓

Create Workspace

↓

Checkout Repository

↓

Download Dependencies

↓

Compile

↓

Execute Tests

↓

Run Security Scans

↓

Build Docker Image

↓

Generate Reports

↓

Archive Artifacts

↓

Clean Workspace
```

---

# Source Code Checkout

The first step inside the workspace is cloning the repository.

```text
GitHub

↓

Clone Repository

↓

Workspace
```

Example

```groovy
checkout scm
```

After checkout

```text
workspace/

├── Jenkinsfile
├── src/
├── pom.xml
└── Dockerfile
```

---

# Working Directory

Every shell command executes inside the workspace.

Example

```groovy
sh 'pwd'
```

Output

```text
/var/lib/jenkins/workspace/catalog-service
```

Every relative path starts from this location.

---

# pwd() Step

The `pwd()` step returns the current workspace directory.

Example

```groovy
script {

    def workspace = pwd()

    echo workspace

}
```

Output

```text
/var/lib/jenkins/workspace/catalog-service
```

Useful for:

- Logging
- Debugging
- Creating custom file paths

---

# dir() Step

The `dir()` step changes the working directory temporarily.

Example

```groovy
dir('frontend') {

    sh 'npm install'
    sh 'npm run build'

}
```

Pipeline flow

```text
Workspace

↓

frontend/

↓

npm install

↓

npm build
```

Once the block finishes, Jenkins automatically returns to the original workspace.

---

# Multiple Directories

Example

```groovy
stage('Backend') {

    steps {

        dir('backend') {

            sh 'mvn clean package'

        }

    }

}

stage('Frontend') {

    steps {

        dir('frontend') {

            sh 'npm install'
            sh 'npm run build'

        }

    }

}
```

Execution

```text
Workspace

├── backend/

│      Build Java

│

└── frontend/

       Build UI
```

---

# Custom Workspace

Sometimes projects require a different workspace location.

Example

```groovy
pipeline {

    agent {

        node {

            label 'linux'

            customWorkspace '/opt/builds/catalog'

        }

    }

}
```

Instead of

```text
/var/lib/jenkins/workspace/catalog
```

Jenkins uses

```text
/opt/builds/catalog
```

---

# Workspace Reuse

By default,

Jenkins reuses the existing workspace.

```text
Build #10

↓

Workspace Exists

↓

Reuse Workspace

↓

Pull Latest Changes
```

Advantages

- Faster builds
- Dependency cache reuse

Potential issue

Old files may remain and affect new builds.

---

# Workspace Isolation

Each job receives its own workspace.

```text
Order Service

↓

workspace/order-service

-------------------------

Payment Service

↓

workspace/payment-service

-------------------------

Frontend

↓

workspace/frontend
```

This prevents conflicts between projects.

---

# Workspace on Kubernetes Agents

With Kubernetes Plugin,

each build gets a new Pod and a fresh workspace.

```text
Pipeline

↓

Create Kubernetes Pod

↓

Create Workspace

↓

Execute Pipeline

↓

Delete Pod

↓

Workspace Removed
```

This ensures clean and isolated builds for every execution.

---

# Production Workspace Flow

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Kubernetes Agent

↓

Create Workspace

↓

Checkout Code

↓

Compile

↓

Unit Tests

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

Archive Reports

↓

Clean Workspace

↓

Delete Kubernetes Pod
```

---

# Workspace Structure

A Jenkins workspace contains all the files required during pipeline execution.

It includes the application source code, configuration files, dependencies, test reports, security scan reports, and generated artifacts.

---

# Typical Workspace Structure

Java Application

```text
workspace/

├── Jenkinsfile
├── pom.xml
├── Dockerfile
├── src/
├── target/
├── reports/
├── dependency-check-report/
├── trivy-report/
├── .git/
└── README.md
```

---

Node.js Application

```text
workspace/

├── Jenkinsfile
├── package.json
├── package-lock.json
├── Dockerfile
├── src/
├── dist/
├── node_modules/
├── reports/
└── .git/
```

---

Python Application

```text
workspace/

├── Jenkinsfile
├── requirements.txt
├── app.py
├── Dockerfile
├── tests/
├── reports/
├── venv/
└── .git/
```

---

# Workspace Lifecycle

A workspace goes through multiple stages during a pipeline.

```text
Pipeline Triggered

↓

Allocate Agent

↓

Create Workspace

↓

Checkout Repository

↓

Download Dependencies

↓

Compile

↓

Execute Tests

↓

Run Security Scans

↓

Build Docker Image

↓

Generate Reports

↓

Archive Artifacts

↓

Clean Workspace
```

---

# Source Code Checkout

The first step inside the workspace is cloning the repository.

```text
GitHub

↓

Clone Repository

↓

Workspace
```

Example

```groovy
checkout scm
```

After checkout

```text
workspace/

├── Jenkinsfile
├── src/
├── pom.xml
└── Dockerfile
```

---

# Working Directory

Every shell command executes inside the workspace.

Example

```groovy
sh 'pwd'
```

Output

```text
/var/lib/jenkins/workspace/catalog-service
```

Every relative path starts from this location.

---

# pwd() Step

The `pwd()` step returns the current workspace directory.

Example

```groovy
script {

    def workspace = pwd()

    echo workspace

}
```

Output

```text
/var/lib/jenkins/workspace/catalog-service
```

Useful for:

- Logging
- Debugging
- Creating custom file paths

---

# dir() Step

The `dir()` step changes the working directory temporarily.

Example

```groovy
dir('frontend') {

    sh 'npm install'
    sh 'npm run build'

}
```

Pipeline flow

```text
Workspace

↓

frontend/

↓

npm install

↓

npm build
```

Once the block finishes, Jenkins automatically returns to the original workspace.

---

# Multiple Directories

Example

```groovy
stage('Backend') {

    steps {

        dir('backend') {

            sh 'mvn clean package'

        }

    }

}

stage('Frontend') {

    steps {

        dir('frontend') {

            sh 'npm install'
            sh 'npm run build'

        }

    }

}
```

Execution

```text
Workspace

├── backend/

│      Build Java

│

└── frontend/

       Build UI
```

---

# Custom Workspace

Sometimes projects require a different workspace location.

Example

```groovy
pipeline {

    agent {

        node {

            label 'linux'

            customWorkspace '/opt/builds/catalog'

        }

    }

}
```

Instead of

```text
/var/lib/jenkins/workspace/catalog
```

Jenkins uses

```text
/opt/builds/catalog
```

---

# Workspace Reuse

By default,

Jenkins reuses the existing workspace.

```text
Build #10

↓

Workspace Exists

↓

Reuse Workspace

↓

Pull Latest Changes
```

Advantages

- Faster builds
- Dependency cache reuse

Potential issue

Old files may remain and affect new builds.

---

# Workspace Isolation

Each job receives its own workspace.

```text
Order Service

↓

workspace/order-service

-------------------------

Payment Service

↓

workspace/payment-service

-------------------------

Frontend

↓

workspace/frontend
```

This prevents conflicts between projects.

---

# Workspace on Kubernetes Agents

With Kubernetes Plugin,

each build gets a new Pod and a fresh workspace.

```text
Pipeline

↓

Create Kubernetes Pod

↓

Create Workspace

↓

Execute Pipeline

↓

Delete Pod

↓

Workspace Removed
```

This ensures clean and isolated builds for every execution.

---

# Production Workspace Flow

```text
Developer Push

↓

GitHub Webhook

↓

Jenkins

↓

Kubernetes Agent

↓

Create Workspace

↓

Checkout Code

↓

Compile

↓

Unit Tests

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

Archive Reports

↓

Clean Workspace

↓

Delete Kubernetes Pod
```

---

