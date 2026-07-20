# Jenkins Credentials

## Introduction

Production CI/CD pipelines interact with multiple external systems such as GitHub, Docker Hub, AWS, Kubernetes, SonarQube, and Artifactory.

These systems require authentication. Instead of hardcoding usernames, passwords, or API keys inside the Jenkinsfile, Jenkins provides a secure Credentials Management system.

Jenkins Credentials allow pipelines to securely access external resources without exposing sensitive information.

---

Without Credentials

```text
Jenkinsfile

↓

Username = admin

Password = admin123

↓

Security Risk
```

---

With Jenkins Credentials

```text
Jenkins

↓

Credentials Store

↓

Secure Authentication

↓

Pipeline Uses Credentials
```

---

# Why Do We Need Jenkins Credentials?

Imagine a production pipeline.

```text
GitHub

↓

Clone Repository

↓

Docker Hub

↓

Push Image

↓

AWS

↓

Deploy to EKS

↓

SonarQube

↓

Upload Reports
```

Every system requires authentication.

Instead of writing passwords inside Jenkinsfiles,

Jenkins retrieves them securely from its Credentials Store.

---

Benefits

```text
Secure

Centralized

Reusable

Encrypted

Easy to Manage
```

---

# Where Are Credentials Used?

Almost every enterprise pipeline uses credentials.

Example

```text
GitHub PAT

↓

Clone Repository

---------------------

Docker Hub

↓

Push Images

---------------------

AWS Access Keys

↓

Deploy Infrastructure

---------------------

Kubernetes Config

↓

Deploy Applications

---------------------

SonarQube Token

↓

Code Analysis
```

---

# Jenkins Credentials Architecture

```text
Developer

↓

Jenkins Pipeline

↓

Credentials Store

↓

Authentication

↓

External Service
```

---

# Types of Jenkins Credentials

Jenkins supports multiple credential types.

```text
Username & Password

Secret Text

SSH Username with Private Key

Secret File

Certificate
```

---

# Username with Password

Used when authentication requires both a username and password.

Examples

```text
GitHub

Docker Hub

JFrog Artifactory

Nexus Repository
```

---

Example

```text
Username

↓

surendra

Password

↓

********
```

---

# Secret Text

Stores a single secret value.

Examples

```text
GitHub PAT

SonarQube Token

Slack Token

API Keys
```

---

Example

```text
Token

↓

ghp_xxxxxxxxxxxxx
```

---

# SSH Username with Private Key

Used for Git repositories accessed using SSH.

```text
GitHub

↓

SSH Key

↓

Clone Repository
```

---

Examples

```text
GitHub SSH

GitLab SSH

Bitbucket SSH
```

---

# Secret File

Stores files securely.

Examples

```text
Kubeconfig

AWS Credentials File

JSON Keys

License Files
```

---

Example

```text
kubeconfig

↓

Deploy to Kubernetes
```

---

# Certificate

Stores certificates required for secure communication.

Examples

```text
SSL Certificates

PKCS12

Java Keystore
```

---

# Credential Scope

Jenkins supports two scopes.

```text
Global

System
```

---

Global

```text
Accessible

↓

All Pipelines
```

---

System

```text
Only Jenkins Internal Usage
```

---

# Adding Credentials

Navigate to

```text
Manage Jenkins

↓

Credentials

↓

Global

↓

Add Credentials
```

---

Required Information

```text
Type

ID

Username

Password / Secret

Description
```

---

# Credential ID

Each credential has a unique identifier.

Example

```text
github-creds

aws-creds

dockerhub-creds

sonarqube-token
```

---

Pipelines reference the Credential ID instead of passwords.

```groovy
credentialsId: "github-creds"
```

---

# Using Credentials in Pipeline

Example

```groovy
git branch: "main",
    credentialsId: "github-creds",
    url: "https://github.com/company/cart.git"
```

---

Jenkins automatically retrieves the stored credentials.

The password is never exposed in the Jenkinsfile.

---

# Environment Variables with Credentials

Jenkins can expose credentials as environment variables.

Example

```groovy
environment {

    SONAR_TOKEN = credentials('sonarqube-token')

}
```

---

Pipeline

↓

Credential Store

↓

Environment Variable

↓

Pipeline Uses Secret

---

# Using Secret Text

Example

```groovy
withCredentials([string(credentialsId: 'sonarqube-token',
variable: 'TOKEN')]) {

    sh "mvn sonar:sonar -Dsonar.login=$TOKEN"

}
```

---

# Using Username and Password

Example

```groovy
withCredentials([usernamePassword(

credentialsId: 'dockerhub-creds',

usernameVariable: 'USER',

passwordVariable: 'PASS'

)]) {

    sh "docker login -u $USER -p $PASS"

}
```

---

# Using SSH Credentials

Example

```groovy
git branch: "main",

credentialsId: "github-ssh",

url: "git@github.com:company/cart.git"
```

---

# Credential Flow

```text
Pipeline

↓

Credential ID

↓

Credentials Store

↓

Authentication

↓

External Service
```

---

# Production Pipeline Flow

## Scenario

A developer pushes code to GitHub. The Jenkins pipeline securely accesses GitHub, SonarQube, AWS, and Kubernetes using Jenkins Credentials.

No passwords or tokens are stored inside the Jenkinsfile.

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
Jenkins Pipeline
     │
     ▼
Retrieve GitHub Credentials
     │
     ▼
Checkout Source Code
     │
     ▼
Build Application
     │
     ▼
Unit Tests
     │
     ▼
Retrieve SonarQube Token
     │
     ▼
SonarQube Analysis
     │
     ▼
OWASP Dependency Check
     │
     ▼
Build Docker Image
     │
     ▼
Trivy Image Scan
     │
     ▼
Retrieve AWS Credentials
     │
     ▼
Login to AWS ECR
     │
     ▼
Push Docker Image
     │
     ▼
Retrieve Kubernetes Credentials
     │
     ▼
Deploy to AWS EKS
     │
     ▼
Post Actions
```

---

# Complete Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    tools {

        jdk "jdk17"
        maven "maven3"

    }

    environment {

        AWS_REGION = "ap-south-1"

        ECR_REPOSITORY = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart"

        IMAGE_TAG = "${BUILD_NUMBER}"

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

                sh "mvn clean package"

            }

        }

        stage("Unit Test") {

            steps {

                sh "mvn test"

            }

        }

        stage("SonarQube Analysis") {

            environment {

                SONAR_TOKEN = credentials('sonarqube-token')

            }

            steps {

                sh """

                mvn sonar:sonar \
                -Dsonar.login=$SONAR_TOKEN

                """

            }

        }

        stage("OWASP Dependency Check") {

            steps {

                dependencyCheck additionalArguments: '--scan .',
                                odcInstallation: 'OWASP'

            }

        }

        stage("Docker Build") {

            steps {

                sh """

                docker build \
                -t ${ECR_REPOSITORY}:${IMAGE_TAG} .

                """

            }

        }

        stage("Trivy Scan") {

            steps {

                sh """

                trivy image \
                ${ECR_REPOSITORY}:${IMAGE_TAG}

                """

            }

        }

        stage("Login to AWS ECR") {

            steps {

                withCredentials([usernamePassword(

                    credentialsId: 'aws-creds',

                    usernameVariable: 'AWS_ACCESS_KEY_ID',

                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'

                )]) {

                    sh """

                    aws ecr get-login-password \
                    --region ${AWS_REGION} \
                    | docker login \
                    --username AWS \
                    --password-stdin \
                    ${ECR_REPOSITORY}

                    """

                }

            }

        }

        stage("Push Docker Image") {

            steps {

                sh """

                docker push \
                ${ECR_REPOSITORY}:${IMAGE_TAG}

                """

            }

        }

        stage("Deploy to Kubernetes") {

            steps {

                withCredentials([file(

                    credentialsId: 'kubeconfig',

                    variable: 'KUBECONFIG'

                )]) {

                    sh """

                    kubectl apply -f k8s/deployment.yaml

                    kubectl rollout status deployment/cart

                    """

                }

            }

        }

    }

    post {

        always {

            junit "target/surefire-reports/*.xml"

            archiveArtifacts artifacts: "target/*.jar"

        }

        success {

            echo "Application Deployed Successfully"

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

# How Credentials Are Used

```text
GitHub Credentials

↓

Clone Repository

-------------------------

SonarQube Token

↓

Upload Code Analysis

-------------------------

AWS Credentials

↓

Login to AWS ECR

↓

Push Docker Image

-------------------------

Kubernetes Credentials

↓

Deploy to EKS
```

---

# Best Practices

## Never Hardcode Secrets

❌ Bad

```groovy
password = "admin123"
```

---

✅ Good

```groovy
credentialsId: "github-creds"
```

---

## Use Meaningful Credential IDs

Good Examples

```text
github-creds

aws-creds

dockerhub-creds

sonarqube-token

kubeconfig
```

Avoid IDs like

```text
cred1

test123

password
```

---

## Grant Minimum Required Permissions

Example

```text
GitHub

↓

Read Repository

Only
```

---

```text
AWS

↓

Push Images

Deploy

Only
```

Never grant Administrator permissions unless required.

---

## Rotate Credentials Regularly

Replace

```text
PAT

AWS Keys

SSH Keys

API Tokens
```

at regular intervals.

---

## Use Secret Text for Tokens

Instead of

```text
Username

Password
```

use

```text
Secret Text
```

for

- GitHub PAT
- SonarQube Token
- Slack Token
- API Keys

---

## Store Secrets Only in Jenkins

```text
Git Repository

↓

No Passwords

↓

Jenkins Credentials

↓

Secure Access
```

---

## Use Folder-Level Credentials

Large organizations use

```text
Folder

↓

Own Credentials

↓

Project Isolation
```

instead of sharing one credential across all projects.

---

# Troubleshooting

## Authentication Failed

Verify

```text
Credential ID

Username

Password

Token
```

---

## Credential Not Found

Check

```text
credentialsId

↓

Matches Jenkins Credential
```

---

## Git Clone Failed

Verify

```text
Repository URL

PAT

SSH Key

Permissions
```

---

## SonarQube Authentication Failed

Verify

```text
Secret Text

Token

Credential ID
```

---

## Docker Login Failed

Verify

```text
AWS Credentials

Docker Credentials

Registry URL
```

---

## Kubernetes Deployment Failed

Check

```text
Kubeconfig

Namespace

Cluster Access
```

---

# Common Interview Questions

## 1. What are Jenkins Credentials?

### Short Answer

Jenkins Credentials securely store authentication details used by pipelines.

### Detailed Explanation

Instead of hardcoding usernames, passwords, tokens, or SSH keys inside Jenkinsfiles, Jenkins retrieves them securely from its Credentials Store during pipeline execution.

### Production Example

GitHub PAT used to clone a private repository.

### Follow-Up Questions

- Where are credentials stored?
- Are they encrypted?

---

## 2. Why should we use Jenkins Credentials?

### Short Answer

To securely authenticate with external systems without exposing secrets.

### Detailed Explanation

Credentials improve security, centralize secret management, and prevent accidental exposure in source code.

### Production Example

AWS credentials used to push Docker images to Amazon ECR.

---

## 3. What credential types does Jenkins support?

### Short Answer

- Username & Password
- Secret Text
- SSH Key
- Secret File
- Certificate

### Follow-Up Questions

- Which type is used for SonarQube?
- Which type is used for Kubernetes?

---

## 4. What is Secret Text?

### Short Answer

A credential type used to store a single secret such as an API token.

### Production Example

SonarQube authentication token.

---

## 5. What is the difference between Secret Text and Username/Password?

### Short Answer

Secret Text stores one value, while Username/Password stores two values.

---

## 6. What is a Credential ID?

### Short Answer

A unique identifier used by Jenkins pipelines to retrieve credentials.

### Example

```groovy
credentialsId: "github-creds"
```

---

## 7. How do you use credentials inside a Pipeline?

### Short Answer

Using `credentials()` or `withCredentials()`.

---

## 8. Where should secrets be stored?

### Short Answer

Inside Jenkins Credentials Store, never inside Git repositories or Jenkinsfiles.

---

## 9. How do you deploy to Kubernetes securely?

### Short Answer

Store the kubeconfig as a Secret File credential and access it using `withCredentials(file(...))`.

---

## 10. How do enterprises manage credentials?

### Short Answer

Using separate credentials for GitHub, SonarQube, Docker, AWS, Kubernetes, and other services with least-privilege access.

---

## 11. Scenario: Jenkins cannot clone a private repository.

### Answer

Check

```text
PAT

Credential ID

Repository Permissions

Repository URL
```

---

## 12. Scenario: AWS ECR login fails.

### Answer

Verify

```text
AWS Credentials

IAM Permissions

AWS Region

ECR Repository
```

---

## 13. Scenario: Kubernetes deployment fails due to authentication.

### Answer

Verify

```text
Kubeconfig

Credential ID

Cluster Access

Namespace
```

---

## 14. What are Folder Credentials?

### Short Answer

Credentials scoped to a specific Jenkins folder, allowing project isolation and better access control.

---

## 15. What is the best practice for managing credentials?

### Short Answer

Store secrets in Jenkins Credentials, use least-privilege access, rotate them regularly, and never hardcode them in source code.

---

# Key Takeaways

```text
Jenkins Credentials securely manage sensitive information.

Never hardcode usernames, passwords, or tokens in Jenkinsfiles.

Use Secret Text for API tokens.

Use Username & Password for services requiring both values.

Use Secret File for Kubernetes kubeconfig.

Use SSH Keys for Git repositories over SSH.

Reference credentials using credentialsId.

Use withCredentials() to securely access secrets during pipeline execution.

Rotate credentials regularly and grant only the minimum required permissions.

Enterprise pipelines use credentials to securely access GitHub, SonarQube, AWS, Docker registries, Kubernetes, and other external services.
```