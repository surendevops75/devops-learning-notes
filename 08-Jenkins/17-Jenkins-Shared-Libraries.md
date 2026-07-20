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

A Jenkins Shared Library is a reusable collection of pipeline code that can be shared across multiple Jenkins pipelines.

### Production Example

One Shared Library is used by 50 microservices.

---

## 2. Why do we use Shared Libraries?

### Short Answer

To eliminate duplicate pipeline code and standardize CI/CD pipelines.

---

## 3. What are the main directories in a Shared Library?

### Answer

```text
vars/

src/

resources/
```

---

## 4. What is the purpose of the `vars` directory?

### Short Answer

It stores reusable pipeline functions that can be called directly from a Jenkinsfile.

---

## 5. What is the purpose of the `src` directory?

### Short Answer

It contains reusable Groovy classes and business logic.

---

## 6. What is the purpose of the `resources` directory?

### Short Answer

It stores configuration files and other resources required by the Shared Library.

---

## 7. How do you load a Shared Library?

### Answer

```groovy
@Library('shared-library') _
```

---

## 8. How are Shared Library functions called?

### Example

```groovy
buildApp()

dockerBuild()

deployApp()
```

---

## 9. Where are Shared Libraries configured?

### Answer

```text
Manage Jenkins

↓

System

↓

Global Trusted Pipeline Libraries
```

---

## 10. Can Shared Libraries be versioned?

### Short Answer

Yes. They are stored in Git, so projects can use branches, tags, or specific versions.

---

## 11. Scenario: Multiple teams use the same build process. How would you avoid duplicating Jenkinsfiles?

### Answer

Create a Shared Library containing reusable build, scan, and deployment functions, then load it into each Jenkinsfile.

---

## 12. Scenario: A build function changes for all projects. What should you do?

### Answer

Update the Shared Library once. All projects using the updated version will benefit from the change.

---

## 13. What are the benefits of Shared Libraries in enterprise environments?

### Short Answer

- Reusable code
- Standardized pipelines
- Easier maintenance
- Reduced duplication
- Centralized updates

---

## 14. Can a Shared Library access Jenkins Credentials?

### Short Answer

Yes. Shared Library functions can use `withCredentials()` or `credentials()` just like a regular Jenkinsfile.

---

## 15. What is the best practice for designing Shared Libraries?

### Short Answer

Keep functions generic, reusable, version-controlled, and independent of any single application.

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