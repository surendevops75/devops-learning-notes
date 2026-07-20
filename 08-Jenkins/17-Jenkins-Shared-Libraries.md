# Jenkins Shared Libraries

## Introduction

As the number of projects grows, maintaining individual Jenkinsfiles becomes difficult.

Many pipelines contain the same stages such as Checkout, Build, SonarQube Scan, Docker Build, Trivy Scan, and Deployment.

Instead of copying the same code into every Jenkinsfile, Jenkins provides **Shared Libraries** to store reusable pipeline code.

A Shared Library allows multiple Jenkins pipelines to use common functions, reducing duplication and improving maintainability.

---

Without Shared Libraries

```text
Project A
│
├── Jenkinsfile (200 Lines)

Project B
│
├── Jenkinsfile (210 Lines)

Project C
│
├── Jenkinsfile (205 Lines)

Same Code Repeated
```

---

With Shared Libraries

```text
                Shared Library
              ┌───────────────┐
              │ Build()       │
              │ SonarScan()   │
              │ DockerBuild() │
              │ TrivyScan()   │
              │ Deploy()      │
              └───────────────┘
                     ▲
                     │
      ┌──────────────┼──────────────┐
      │              │              │
      ▼              ▼              ▼
 Project A      Project B      Project C
```

---

# Why Do We Need Shared Libraries?

Imagine an organization with 100 microservices.

Every Jenkinsfile contains:

```text
Checkout

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Push Image

↓

Deploy
```

Updating all Jenkinsfiles individually is difficult.

With Shared Libraries, changes are made once and automatically used by all pipelines.

---

Benefits

```text
Reusable Code

Less Duplication

Centralized Logic

Easy Maintenance

Standardized Pipelines

Faster Development
```

---

# Shared Library Architecture

```text
               Git Repository
          (Shared Library Code)
                    │
                    ▼
              Jenkins Server
                    │
         Loads Shared Library
                    │
                    ▼
            Jenkins Pipeline
                    │
                    ▼
         Executes Shared Functions
```

---

# Folder Structure

A Shared Library follows a standard directory structure.

```text
shared-library/
│
├── vars/
│
├── src/
│
├── resources/
│
└── README.md
```

---

# vars Directory

The **vars** directory contains reusable pipeline functions.

Example

```text
vars/

buildApp.groovy

dockerBuild.groovy

sonarScan.groovy

deployApp.groovy
```

These functions can be called directly from the Jenkinsfile.

---

# src Directory

The **src** directory contains reusable Groovy classes.

Example

```text
src/

com/company/Build.groovy

com/company/Docker.groovy

com/company/Deploy.groovy
```

---

# resources Directory

Stores configuration files used by the Shared Library.

Example

```text
resources/

deployment.yaml

application.properties

config.json
```

---

# Global Shared Library

Configured under

```text
Manage Jenkins

↓

System

↓

Global Trusted Pipeline Libraries
```

---

Required Details

```text
Library Name

Git Repository URL

Branch

Credentials

Default Version
```

---

# Loading a Shared Library

Example

```groovy
@Library('shared-library') _
```

Jenkins downloads the library before executing the pipeline.

---

# Calling Shared Functions

Example

```groovy
buildApp()

sonarScan()

dockerBuild()

deployApp()
```

Instead of writing all the implementation inside the Jenkinsfile.

---

# Shared Library Flow

```text
Pipeline

↓

Load Library

↓

Call Function

↓

Execute Function

↓

Continue Pipeline
```

---

# Production Pipeline Flow

## Scenario

An organization has 50 microservices.

Instead of maintaining 50 different Jenkinsfiles, all common pipeline logic is stored in a Shared Library.

Whenever a project starts a pipeline, Jenkins first downloads the Shared Library and executes the reusable functions.

---

Pipeline Flow

```text
                    Developer
                         │
                         ▼
                   Git Push
                         │
                         ▼
                 GitHub Webhook
                         │
                         ▼
                    Jenkins
                         │
                         ▼
         Load Shared Library from Git
                         │
                         ▼
                  Checkout Source
                         │
                         ▼
                   buildApp()
                         │
                         ▼
                 unitTest()
                         │
                         ▼
                 sonarScan()
                         │
                         ▼
         dependencyCheckScan()
                         │
                         ▼
                dockerBuild()
                         │
                         ▼
                 trivyScan()
                         │
                         ▼
                  pushImage()
                         │
                         ▼
                  deployApp()
                         │
                         ▼
                   Post Actions
```

---

# Enterprise Shared Library Repository

Example Repository

```text
shared-library/
│
├── vars/
│   ├── buildApp.groovy
│   ├── unitTest.groovy
│   ├── sonarScan.groovy
│   ├── dependencyCheckScan.groovy
│   ├── dockerBuild.groovy
│   ├── trivyScan.groovy
│   ├── pushImage.groovy
│   └── deployApp.groovy
│
├── src/
│   └── com/company/
│       ├── Docker.groovy
│       ├── Build.groovy
│       └── Deploy.groovy
│
├── resources/
│   ├── deployment.yaml
│   └── config.properties
│
└── README.md
```

---

# Sample Shared Library Functions

## vars/buildApp.groovy

```groovy
def call() {

    sh "mvn clean package"

}
```

---

## vars/unitTest.groovy

```groovy
def call() {

    sh "mvn test"

}
```

---

## vars/sonarScan.groovy

```groovy
def call() {

    withCredentials([string(
        credentialsId: 'sonarqube-token',
        variable: 'TOKEN'
    )]) {

        sh """
        mvn sonar:sonar \
        -Dsonar.login=$TOKEN
        """

    }

}
```

---

## vars/dependencyCheckScan.groovy

```groovy
def call() {

    dependencyCheck(
        additionalArguments: '--scan .',
        odcInstallation: 'OWASP'
    )

}
```

---

## vars/dockerBuild.groovy

```groovy
def call(imageName) {

    sh "docker build -t ${imageName} ."

}
```

---

## vars/trivyScan.groovy

```groovy
def call(imageName) {

    sh "trivy image ${imageName}"

}
```

---

## vars/pushImage.groovy

```groovy
def call(imageName) {

    sh "docker push ${imageName}"

}
```

---

## vars/deployApp.groovy

```groovy
def call() {

    sh "kubectl apply -f k8s/deployment.yaml"

}
```

---

# Complete Production Jenkinsfile

```groovy
@Library('shared-library') _

pipeline {

    agent any

    environment {

        IMAGE = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart:${BUILD_NUMBER}"

    }

    triggers {

        githubPush()

    }

    stages {

        stage("Checkout") {

            steps {

                git branch: "main",
                    credentialsId: "github-creds",
                    url: "https://github.com/company/cart.git"

            }

        }

        stage("Build") {

            steps {

                buildApp()

            }

        }

        stage("Unit Test") {

            steps {

                unitTest()

            }

        }

        stage("SonarQube") {

            steps {

                sonarScan()

            }

        }

        stage("Dependency Check") {

            steps {

                dependencyCheckScan()

            }

        }

        stage("Docker Build") {

            steps {

                dockerBuild(IMAGE)

            }

        }

        stage("Trivy Scan") {

            steps {

                trivyScan(IMAGE)

            }

        }

        stage("Push Image") {

            steps {

                pushImage(IMAGE)

            }

        }

        stage("Deploy") {

            steps {

                deployApp()

            }

        }

    }

    post {

        always {

            junit "target/surefire-reports/*.xml"

            archiveArtifacts artifacts: "target/*.jar"

        }

        success {

            echo "Deployment Successful"

        }

        failure {

            echo "Deployment Failed"

        }

        cleanup {

            cleanWs()

        }

    }

}
```

---

# How Shared Libraries Work

```text
Pipeline

↓

@Library('shared-library')

↓

Download Shared Library

↓

Load Functions

↓

Execute Pipeline

↓

Build

↓

Deploy
```

---

# Benefits of Shared Libraries

```text
One Place to Maintain Code

↓

Reusable Functions

↓

Standard Pipelines

↓

Easy Updates

↓

Reduced Jenkinsfile Size
```

---

# Before and After

Without Shared Library

```text
Project A

↓

250 Lines

-----------------

Project B

↓

260 Lines

-----------------

Project C

↓

240 Lines
```

---

With Shared Library

```text
Project A

↓

40 Lines

-----------------

Project B

↓

42 Lines

-----------------

Project C

↓

38 Lines
```

---

# Best Practices

## Keep Business Logic Inside Shared Libraries

Instead of

```groovy
pipeline {

    ...

    steps {

        sh "mvn clean package"

        sh "docker build ..."

        sh "kubectl apply ..."

    }

}
```

Move the logic to reusable functions.

---

## Keep Jenkinsfiles Small

A Jenkinsfile should mainly orchestrate the workflow.

```text
Load Library

↓

Call Functions

↓

Finish
```

---

## Version Shared Libraries

Use Git branches or tags.

Example

```text
main

v1.0

v2.0

release
```

This allows different projects to use different library versions if needed.

---

## Store Libraries in Git

Treat Shared Libraries like application code.

Benefits

- Version Control
- Code Review
- Rollback
- Collaboration

---

## Write Reusable Functions

Avoid project-specific logic.

Good

```groovy
dockerBuild(image)
```

Bad

```groovy
dockerBuildCartApplication()
```

---

## Secure Shared Libraries

Only trusted users should modify Shared Library repositories because changes affect multiple pipelines.

---

# Troubleshooting

## Library Not Found

Verify

```text
Library Name

Git Repository URL

Branch
```

---

## Function Not Found

Check

```text
vars/

Function Name

call() Method
```

---

## Compilation Error

Verify

```text
Groovy Syntax

Missing Braces

Incorrect Method Name
```

---

## Git Authentication Failed

Verify

```text
Git Credentials

Repository Access

Credential ID
```

---

## Changes Not Reflected

Possible Causes

```text
Old Library Version

Wrong Branch

Pipeline Cache
```

---

## Jenkins Cannot Load Library

Verify

```text
Manage Jenkins

↓

Global Trusted Pipeline Libraries
```

---

# Common Interview Questions

## 1. What is a Jenkins Shared Library?

### Short Answer

A Jenkins Shared Library is a reusable collection of Groovy scripts and pipeline functions that can be shared across multiple Jenkins pipelines.

### Detailed Explanation

As organizations grow, maintaining the same pipeline code in every repository becomes difficult. Many projects have identical stages such as Checkout, Build, Unit Test, SonarQube Scan, Docker Build, Trivy Scan, and Deployment.

Instead of duplicating this logic in every Jenkinsfile, Jenkins provides Shared Libraries. These libraries are stored in a separate Git repository and loaded dynamically during pipeline execution.

A Shared Library promotes code reuse, standardization, and easier maintenance across multiple projects.

### Production Example

Suppose an organization has 100 microservices.

Without Shared Libraries

```text
100 Jenkinsfiles

↓

Each contains Build

↓

Each contains Docker Build

↓

Each contains Sonar Scan

↓

100 Files to Maintain
```

With Shared Libraries

```text
Shared Library

↓

buildApp()

↓

dockerBuild()

↓

sonarScan()

↓

100 Pipelines Reuse Same Functions
```

If the Docker build process changes, only the Shared Library needs to be updated.

### Follow-Up Questions

- Why are Shared Libraries important?
- Where are Shared Libraries stored?
- How are they loaded into a Jenkins pipeline?

---

## 2. Why do we use Shared Libraries?

### Short Answer

Shared Libraries eliminate duplicate pipeline code and help standardize CI/CD pipelines.

### Detailed Explanation

In enterprise environments, multiple teams develop different applications, but the CI/CD workflow is often very similar.

For example, every project may need to:

- Build the application
- Run unit tests
- Perform SonarQube analysis
- Scan dependencies
- Build Docker images
- Scan images with Trivy
- Push images to ECR
- Deploy to Kubernetes

Instead of rewriting these stages for every application, they are implemented once in a Shared Library and reused across all pipelines.

This reduces maintenance effort and ensures every project follows the same deployment standards.

### Production Example

Before Shared Library

```text
Cart Service

↓

250-Line Jenkinsfile

--------------------

User Service

↓

240-Line Jenkinsfile

--------------------

Payment Service

↓

245-Line Jenkinsfile
```

After Shared Library

```text
Cart Service

↓

35-Line Jenkinsfile

--------------------

User Service

↓

36-Line Jenkinsfile

--------------------

Payment Service

↓

34-Line Jenkinsfile
```

Most of the implementation resides inside the Shared Library.

### Follow-Up Questions

- What problems do Shared Libraries solve?
- How do they improve maintainability?
- Can multiple teams use the same Shared Library?

---

## 3. What is the purpose of the `vars` directory?

### Short Answer

The `vars` directory contains reusable pipeline functions that can be called directly from a Jenkinsfile.

### Detailed Explanation

Every Groovy file inside the `vars` directory represents a global pipeline function.

When Jenkins loads a Shared Library, these functions become available to every pipeline.

Each file usually contains a `call()` method, allowing it to be invoked like a normal pipeline step.

### Production Example

Repository

```text
vars/

↓

buildApp.groovy

↓

dockerBuild.groovy

↓

deployApp.groovy
```

buildApp.groovy

```groovy
def call() {

    sh "mvn clean package"

}
```

Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                buildApp()

            }

        }

    }

}
```

Instead of writing Maven commands in every Jenkinsfile, the reusable function is called.

### Follow-Up Questions

- Why is the `call()` method used?
- Can multiple functions exist inside `vars`?
- When should we use `vars` instead of `src`?

---

## 4. What is the purpose of the `src` directory?

### Short Answer

The `src` directory stores reusable Groovy classes and complex business logic.

### Detailed Explanation

Unlike the `vars` directory, which provides simple pipeline functions, the `src` directory is designed for object-oriented programming.

It is commonly used when pipelines require classes, helper methods, utility functions, or complex processing logic.

The directory structure follows Java package conventions.

### Production Example

Repository

```text
src/

↓

com/company/

↓

Docker.groovy

↓

Deploy.groovy

↓

Notification.groovy
```

Pipeline

```text
Pipeline

↓

Import Class

↓

Create Object

↓

Execute Methods
```

This approach keeps pipelines clean while separating complex logic into reusable classes.

### Follow-Up Questions

- What is the difference between `vars` and `src`?
- Why does `src` follow Java package conventions?
- When should you move code from `vars` to `src`?

---

## 5. What is the purpose of the `resources` directory?

### Short Answer

The `resources` directory stores static files required by Shared Libraries.

### Detailed Explanation

Sometimes a Shared Library needs configuration files, templates, Kubernetes manifests, or JSON files.

Instead of embedding these directly into Groovy scripts, they are stored inside the `resources` directory.

During pipeline execution, these files can be loaded and used by Shared Library functions.

### Production Example

Repository

```text
resources/

↓

deployment.yaml

↓

application.properties

↓

config.json

↓

email-template.html
```

Pipeline Flow

```text
Pipeline

↓

Shared Library

↓

Load Resource File

↓

Deploy Application
```

For example, a reusable Kubernetes deployment template can be stored once inside the Shared Library and used by multiple projects.

### Follow-Up Questions

- What types of files belong in `resources`?
- Can Kubernetes manifests be stored there?
- Why shouldn't configuration files be hardcoded in Groovy scripts?

## 6. How do you load a Shared Library in Jenkins?

### Short Answer

A Shared Library is loaded using the `@Library` annotation at the beginning of the Jenkinsfile.

### Detailed Explanation

Before a pipeline can use reusable functions, Jenkins must first load the Shared Library. The library is configured globally (or at the folder level) in Jenkins and referenced using its library name.

When Jenkins encounters the `@Library` annotation, it clones the Shared Library repository, loads the Groovy files, and makes all functions available to the pipeline.

This process happens before pipeline execution starts.

### Production Example

Global Shared Library Configuration

```text
Manage Jenkins

↓

System

↓

Global Trusted Pipeline Libraries

↓

Library Name : shared-library

↓

Git Repository

↓

https://github.com/company/shared-library.git
```

Jenkinsfile

```groovy
@Library('shared-library') _

pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                buildApp()

            }

        }

    }

}
```

Execution Flow

```text
Pipeline Starts

↓

Load Shared Library

↓

Download Library

↓

Load Functions

↓

Execute Pipeline
```

### Follow-Up Questions

- Where are Shared Libraries configured?
- Can multiple libraries be loaded?
- Can a specific version of a library be loaded?

---

## 7. Can Shared Libraries be version controlled?

### Short Answer

Yes. Shared Libraries are stored in Git repositories, allowing them to be versioned using branches, tags, or commits.

### Detailed Explanation

Since a Shared Library is a Git repository, all changes are tracked through version control.

Organizations commonly maintain different versions of the library, such as:

- Development
- QA
- Production

This prevents unstable changes from affecting production pipelines.

Projects can also use specific branches or Git tags when loading the library.

### Production Example

```text
Shared Library Repository

│

├── main

├── develop

├── release-v1.0

└── release-v2.0
```

Project A

```groovy
@Library('shared-library@release-v1.0') _
```

Project B

```groovy
@Library('shared-library@release-v2.0') _
```

Both projects use different library versions without impacting each other.

### Follow-Up Questions

- Why should Shared Libraries be versioned?
- Can different projects use different versions?
- What happens if the main branch changes unexpectedly?

---

## 8. What is the difference between Global and Folder Shared Libraries?

### Short Answer

Global Shared Libraries are available to all Jenkins jobs, while Folder Shared Libraries are accessible only to jobs within a specific folder.

### Detailed Explanation

Large organizations often organize Jenkins into folders based on teams or business units.

Instead of exposing every Shared Library to every project, Jenkins allows libraries to be scoped at the folder level.

This improves security and enables different teams to maintain their own reusable pipeline code.

### Production Example

```text
Jenkins

│

├── Banking Team

│      │

│      └── banking-library

│

├── Retail Team

│      │

│      └── retail-library

│

└── Platform Team

       │

       └── platform-library
```

Each team can update its own library without affecting other teams.

### Follow-Up Questions

- When should Folder Libraries be used?
- Which type is commonly used in enterprises?
- Can both Global and Folder Libraries exist together?

---

## 9. Can Shared Libraries use Jenkins Credentials?

### Short Answer

Yes. Shared Libraries can access Jenkins Credentials using `withCredentials()` or `credentials()`.

### Detailed Explanation

A Shared Library executes inside the Jenkins pipeline context.

This means it has access to all pipeline steps, installed plugins, and Jenkins Credentials.

Instead of exposing passwords or tokens in every Jenkinsfile, authentication logic is implemented once inside the Shared Library.

This approach improves security and reduces code duplication.

### Production Example

Shared Library Function

```groovy
def call() {

    withCredentials([string(

        credentialsId: 'sonarqube-token',

        variable: 'TOKEN'

    )]) {

        sh """

        mvn sonar:sonar \
        -Dsonar.login=$TOKEN

        """

    }

}
```

Pipeline

```groovy
stage("SonarQube") {

    steps {

        sonarScan()

    }

}
```

Execution Flow

```text
Pipeline

↓

sonarScan()

↓

Read Credential

↓

Authenticate

↓

Execute Scan
```

### Follow-Up Questions

- Which credential types can be used?
- Is it safe to hardcode credentials?
- Can AWS credentials also be accessed this way?

---

## 10. How do Shared Libraries improve CI/CD pipelines?

### Short Answer

Shared Libraries make pipelines reusable, standardized, and easier to maintain.

### Detailed Explanation

Without Shared Libraries, every application team maintains its own Jenkinsfile.

Over time, these Jenkinsfiles become inconsistent, making upgrades difficult.

Shared Libraries centralize common logic so every project follows the same CI/CD process.

This improves consistency, reduces maintenance effort, and accelerates onboarding for new projects.

### Production Example

Enterprise Pipeline

```text
Project A

↓

Shared Library

↓

Checkout

↓

Build

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Deploy
```

Every project follows the same standardized pipeline.

If a new Trivy scanning step is introduced, it is added once to the Shared Library, and all projects automatically benefit after updating to the new library version.

### Follow-Up Questions

- How do Shared Libraries improve consistency?
- Why do enterprises prefer standardized pipelines?
- How do Shared Libraries reduce maintenance costs?

## 11. Scenario: Your organization has 100 microservices. How would you design a Jenkins Shared Library?

### Short Answer

I would separate reusable pipeline logic into modular functions, store them in a Shared Library Git repository, version the library, and keep Jenkinsfiles minimal.

### Detailed Explanation

Large organizations usually have dozens or even hundreds of microservices. Although the applications differ, most CI/CD pipelines follow the same workflow:

- Checkout Source Code
- Build
- Unit Test
- SonarQube Analysis
- OWASP Dependency Check
- Docker Build
- Trivy Scan
- Push Image
- Deploy

Instead of implementing these stages in every repository, each stage is converted into a reusable function inside the Shared Library.

Application repositories only orchestrate the workflow by calling these functions.

This approach provides consistency, reduces duplication, and allows platform teams to improve the pipeline in one place.

### Production Example

Shared Library

```text
shared-library/

↓

buildApp()

↓

unitTest()

↓

sonarScan()

↓

dependencyCheck()

↓

dockerBuild()

↓

trivyScan()

↓

pushImage()

↓

deployApp()
```

Application Jenkinsfile

```groovy
@Library('shared-library') _

pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                buildApp()

            }

        }

        stage("Test") {

            steps {

                unitTest()

            }

        }

        stage("Deploy") {

            steps {

                deployApp()

            }

        }

    }

}
```

Execution Flow

```text
100 Projects

↓

Same Shared Library

↓

Standard CI/CD Pipeline
```

### Follow-Up Questions

- How would you organize the Shared Library repository?
- Should each team have its own library?
- How would you handle library version upgrades?

---

## 12. Scenario: A Shared Library update breaks multiple pipelines. How would you troubleshoot it?

### Short Answer

Identify the affected library version, reproduce the issue, review recent changes, and roll back to the previous stable version if necessary.

### Detailed Explanation

Since multiple projects share the same library, a faulty change can impact every pipeline.

Troubleshooting should begin by determining:

- Which library version is in use?
- What changed recently?
- Is the issue reproducible?

If the latest version introduced the problem, pipelines should temporarily use the previous stable release while the issue is fixed.

Proper versioning significantly reduces the impact of such failures.

### Production Example

```text
Library v1.0

↓

Working

-------------------

Library v2.0

↓

New Docker Logic

↓

Multiple Pipeline Failures

↓

Rollback to v1.0

↓

Fix Issue

↓

Release v2.1
```

Pipeline

```groovy
@Library('shared-library@v1.0') _
```

### Follow-Up Questions

- Why should Shared Libraries be versioned?
- How do you roll back a Shared Library?
- How do you test changes before production?

---

## 13. How do enterprises organize Jenkins Shared Libraries?

### Short Answer

Enterprises organize Shared Libraries into reusable modules with separate functions for build, testing, security, containerization, deployment, and notifications.

### Detailed Explanation

Instead of creating one large Groovy file, enterprises divide the library into logical components.

Each function performs a single responsibility.

This modular design improves readability, testing, maintenance, and reuse.

Platform teams maintain the library while application teams simply consume it.

### Production Example

```text
vars/

↓

buildApp.groovy

↓

unitTest.groovy

↓

sonarScan.groovy

↓

dependencyCheck.groovy

↓

dockerBuild.groovy

↓

trivyScan.groovy

↓

pushImage.groovy

↓

deployApp.groovy

↓

notifySlack.groovy
```

Pipeline

```text
Application

↓

Calls Shared Functions

↓

No Business Logic
```

### Follow-Up Questions

- Why should functions have a single responsibility?
- Should deployment logic be separated from build logic?
- How do platform teams maintain Shared Libraries?

---

## 14. How do you test Shared Library changes before deploying them to production?

### Short Answer

Test changes in a development branch, validate them with non-production pipelines, perform code reviews, and release a version only after successful testing.

### Detailed Explanation

Since Shared Libraries affect many applications, changes should never be committed directly to production.

A typical workflow is:

- Create a feature branch.
- Implement the change.
- Test using a sample Jenkins pipeline.
- Perform peer review.
- Merge into the main branch.
- Create a version tag.

This minimizes the risk of breaking production pipelines.

### Production Example

```text
Feature Branch

↓

Implement Change

↓

Test Pipeline

↓

Pull Request

↓

Code Review

↓

Merge

↓

Release v2.0
```

### Follow-Up Questions

- Why should Shared Libraries have pull requests?
- Why is testing important before release?
- Should production pipelines use the latest commit or a version tag?

---

## 15. Design a reusable DevSecOps Shared Library for GitHub → SonarQube → OWASP → Trivy → ECR → EKS.

### Short Answer

Create separate reusable functions for each CI/CD stage and orchestrate them from a lightweight Jenkinsfile.

### Detailed Explanation

In an enterprise DevSecOps environment, the Shared Library should encapsulate every major pipeline activity.

Each function is responsible for one task and can be reused across multiple applications.

This enables consistent security scanning and deployment standards throughout the organization.

### Production Example

Shared Library

```text
buildApp()

↓

unitTest()

↓

sonarScan()

↓

dependencyCheck()

↓

dockerBuild()

↓

trivyScan()

↓

pushImage()

↓

deployApp()

↓

notifyTeams()
```

Pipeline

```groovy
@Library('shared-library') _

pipeline {

    agent any

    stages {

        stage("Build") {

            steps {

                buildApp()

            }

        }

        stage("Security") {

            steps {

                sonarScan()

                dependencyCheck()

                trivyScan()

            }

        }

        stage("Deployment") {

            steps {

                deployApp()

            }

        }

    }

}
```

Execution Flow

```text
Developer Push

↓

GitHub

↓

Jenkins

↓

Shared Library

↓

Build

↓

Unit Test

↓

SonarQube

↓

OWASP Dependency Check

↓

Docker Build

↓

Trivy Scan

↓

Push Image to ECR

↓

Deploy to EKS

↓

Post Actions
```

### Follow-Up Questions

- Why separate each pipeline stage into different functions?
- Which stages should be mandatory in a DevSecOps pipeline?
- How would you add Slack or Teams notifications without modifying every Jenkinsfile?

---

# Key Takeaways

```text
Shared Libraries eliminate duplicate Jenkins pipeline code.

They centralize reusable functions such as Build, Test, Security Scan, Docker Build, and Deployment.

The standard structure consists of vars, src, and resources directories.

Load a library using @Library('library-name').

Store Shared Libraries in Git for version control.

Keep Jenkinsfiles focused on orchestration and move implementation details into reusable library functions.

Shared Libraries improve consistency, scalability, and maintainability across enterprise CI/CD pipelines.
```