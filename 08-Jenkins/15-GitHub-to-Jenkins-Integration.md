# GitHub to Jenkins Integration

## Introduction

A CI/CD pipeline starts when developers push code to GitHub.

Jenkins automatically detects the change, builds the application, performs security scans, and deploys it.

---

Without Integration

```text
Developer Push

↓

Login to Jenkins

↓

Click Build Now
```

---

With Integration

```text
Developer Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline Starts Automatically
```

---

# Why Integrate GitHub with Jenkins?

GitHub integration enables Continuous Integration.

Every code change is automatically validated.

---

Benefits

```text
Automatic Builds

Faster Feedback

Reduced Manual Work

Continuous Integration

Early Bug Detection
```

---

# Integration Architecture

```text
Developer
     │
     ▼
Git Commit
     │
     ▼
Git Push
     │
     ▼
GitHub Repository
     │
     ▼
Webhook
     │
     ▼
Jenkins
     │
     ▼
Pipeline Execution
```

---

# Prerequisites

Before integrating GitHub with Jenkins, ensure the following are available.

```text
GitHub Account

Git Repository

Jenkins Server

Git Plugin

GitHub Plugin

Network Connectivity
```

---

# Integration Workflow

```text
Developer

↓

Commit Code

↓

Push Code

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout Source

↓

Execute Pipeline
```

---

# Authentication Methods

Jenkins can authenticate with GitHub using:

```text
Username & Password (Deprecated)

Personal Access Token (Recommended)

SSH Key
```

---

For production environments,

```text
Personal Access Token (PAT)
```

is recommended.

---

# Personal Access Token (PAT)

A Personal Access Token is used instead of a GitHub password.

---

Benefits

```text
More Secure

Easy to Revoke

Granular Permissions
```

---

Workflow

```text
GitHub

↓

Generate PAT

↓

Store in Jenkins Credentials

↓

Access Repository
```

---

# Configure Jenkins Credentials

Navigate to

```text
Manage Jenkins

↓

Credentials

↓

Global Credentials

↓

Add Credentials
```

---

Store

```text
GitHub Username

Personal Access Token
```

---

Result

```text
Jenkins Can Access
Private Repository
```

---

# Configure GitHub Repository

Repository

↓

Settings

↓

Webhooks

↓

Add Webhook

---

Payload URL

```text
http://jenkins-ip:8080/github-webhook/
```

---

Content Type

```text
application/json
```

---

Events

```text
Just the push event
```

---

# Webhook Flow

```text
Developer Push

↓

GitHub

↓

Webhook Payload

↓

Jenkins

↓

Pipeline Triggered
```

---

# Configure Jenkins Job

Pipeline Job

↓

Source Code Management

↓

Git

↓

Repository URL

↓

Credentials

↓

Branch

---

Example

```text
Repository

↓

https://github.com/company/cart.git
```

---

Branch

```text
main
```

---

# Build Triggers

Jenkins supports multiple build triggers.

```text
GitHub Webhook

Poll SCM

Manual Build

Cron Schedule
```

---

Recommended

```text
GitHub Webhook
```

---

# Poll SCM vs Webhook

| Feature | Poll SCM | Webhook |
|----------|-----------|----------|
| Trigger | Periodic | Instant |
| Performance | Lower | Better |
| Resource Usage | High | Low |
| Recommended | No | Yes |

---

Workflow

Poll SCM

```text
Every 5 Minutes

↓

Check Repository

↓

Build If Changed
```

---

Webhook

```text
Push Code

↓

Immediate Build
```

---

# Webhook Payload

GitHub sends an HTTP POST request.

---

Payload Contains

```text
Repository

Branch

Commit ID

Author

Modified Files
```

---

Jenkins uses this information to start the pipeline.

---

# Branch-Based Build

```text
feature/login

↓

Build
```

---

```text
main

↓

Build

↓

Deploy
```

---

Branch filtering can be implemented using the Jenkins `when` directive.

---

# Production Pipeline Flow

## Scenario

A developer pushes code to the **main** branch of the Cart Service.

Jenkins automatically builds, scans, packages, and deploys the application to AWS EKS.

---

Pipeline Flow

```text
Developer
     │
     ▼
Git Commit
     │
     ▼
Git Push
     │
     ▼
GitHub Repository
     │
     ▼
GitHub Webhook
     │
     ▼
Jenkins Pipeline
     │
     ▼
Checkout Source Code
     │
     ▼
Compile Application
     │
     ▼
Unit Tests
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
Push Image to AWS ECR
     │
     ▼
Update Kubernetes Manifest
     │
     ▼
Deploy to AWS EKS
     │
     ▼
Verify Deployment
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

        ECR_REPO = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart"

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

            steps {

                sh "mvn sonar:sonar"

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
                -t ${ECR_REPO}:${IMAGE_TAG} .
                """

            }

        }

        stage("Trivy Image Scan") {

            steps {

                sh """
                trivy image \
                ${ECR_REPO}:${IMAGE_TAG}
                """

            }

        }

        stage("Login to AWS ECR") {

            steps {

                sh """
                aws ecr get-login-password \
                --region ${AWS_REGION} \
                | docker login \
                --username AWS \
                --password-stdin \
                ${ECR_REPO}
                """

            }

        }

        stage("Push Docker Image") {

            steps {

                sh """
                docker push \
                ${ECR_REPO}:${IMAGE_TAG}
                """

            }

        }

        stage("Update Kubernetes Manifest") {

            steps {

                sh """
                sed -i \
                's#IMAGE_PLACEHOLDER#${ECR_REPO}:${IMAGE_TAG}#g' \
                k8s/deployment.yaml
                """

            }

        }

        stage("Deploy to AWS EKS") {

            steps {

                sh """

                kubectl apply -f k8s/deployment.yaml

                kubectl rollout status \
                deployment/cart

                """

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

# How GitHub Triggers This Pipeline

```text
Developer Push
      │
      ▼
GitHub Repository
      │
      ▼
Webhook Trigger
      │
      ▼
githubPush()
      │
      ▼
Jenkins Starts Pipeline
```

---

# Best Practices

## Use GitHub Webhooks

Avoid Poll SCM.

```text
Push Code

↓

Immediate Build
```

---

## Store Credentials Securely

Store GitHub PAT inside Jenkins Credentials.

Never hardcode

```text
GitHub Token

Username

Password
```

inside Jenkinsfiles.

---

## Protect Main Branch

```text
feature/*

↓

Build
```

---

```text
main

↓

Build

↓

Deploy
```

---

## Trigger Deployment Only From Main

Use

```groovy
when {

    branch "main"

}
```

before deployment.

---

## Keep Jenkinsfile in Git

```text
Application

+

Jenkinsfile

↓

Same Repository
```

This enables version control for the pipeline.

---

## Use Webhooks Instead of Manual Builds

```text
Developer Push

↓

Automatic Pipeline
```

instead of

```text
Developer Push

↓

Login Jenkins

↓

Build Now
```

---

# Troubleshooting

## Webhook Not Triggering

Verify

```text
Webhook URL

Jenkins Reachability

GitHub Delivery Logs
```

---

## Authentication Failed

Verify

```text
GitHub PAT

Credentials ID

Repository Access
```

---

## Repository Checkout Failed

Check

```text
Repository URL

Branch Name

Git Credentials
```

---

## Pipeline Not Triggered

Verify

```text
githubPush()

Webhook

GitHub Plugin
```

---

## Jenkins Cannot Clone Private Repository

Verify

```text
PAT Permissions

Repository Access

Stored Credentials
```

---

## Webhook Returns 404

Verify webhook URL.

Correct

```text
http://jenkins-server:8080/github-webhook/
```

---

# Common Interview Questions

## 1. Why do we integrate GitHub with Jenkins?

### Short Answer

To automatically trigger CI/CD pipelines whenever code is pushed to GitHub.

### Detailed Explanation

GitHub integration enables Continuous Integration by eliminating manual builds. Jenkins immediately receives webhook events and starts the pipeline.

### Production Example

Developer pushes code → GitHub Webhook → Jenkins Build.

### Follow-Up Questions

- Which trigger is commonly used?
- What are the benefits over manual builds?

---

## 2. What is a GitHub Webhook?

### Short Answer

A webhook is an HTTP callback sent by GitHub to Jenkins when repository events occur.

### Detailed Explanation

GitHub sends an HTTP POST request containing repository and commit information. Jenkins receives it and starts the configured job.

### Production Example

Push to the **main** branch automatically starts the deployment pipeline.

### Follow-Up Questions

- Which HTTP method is used?
- What payload does GitHub send?

---

## 3. Poll SCM vs Webhook?

### Short Answer

Webhook is event-driven, while Poll SCM checks the repository periodically.

### Detailed Explanation

Webhooks provide instant builds with lower resource usage. Poll SCM continuously checks for changes, consuming additional resources.

### Production Example

Most enterprises use Webhooks instead of Poll SCM.

### Follow-Up Questions

- Which one is recommended?
- Why is Poll SCM less efficient?

---

## 4. What is a Personal Access Token?

### Short Answer

A PAT is a secure authentication token used instead of a GitHub password.

### Detailed Explanation

PATs provide granular permissions and are stored securely in Jenkins Credentials.

### Production Example

Jenkins uses a PAT to clone a private GitHub repository.

### Follow-Up Questions

- Why are passwords deprecated?
- Where should PATs be stored?

---

## 5. How does Jenkins authenticate with GitHub?

### Short Answer

Using Jenkins Credentials such as a Personal Access Token or SSH key.

### Detailed Explanation

Credentials are referenced in the Jenkins job and used during Git operations.

### Production Example

A private repository is cloned using a stored GitHub PAT.

### Follow-Up Questions

- Can SSH keys be used?
- Where are credentials managed?

---

## 6. Explain the GitHub to Jenkins workflow.

### Short Answer

Developer Push → GitHub → Webhook → Jenkins → Pipeline Execution.

### Follow-Up Questions

- What triggers the pipeline?
- What happens after the webhook?

---

## 7. Why are Webhooks preferred in Production?

### Short Answer

They provide immediate, event-driven pipeline execution with minimal resource usage.

---

## 8. Where should the Jenkinsfile be stored?

### Short Answer

Inside the application's Git repository.

---

## 9. Can Jenkins build private GitHub repositories?

### Short Answer

Yes, using Jenkins Credentials.

---

## 10. What happens if the webhook fails?

### Short Answer

The pipeline will not start automatically. Check GitHub webhook delivery logs and Jenkins accessibility.

---

## 11. Scenario: Git Push does not trigger Jenkins.

### Answer

Check

```text
Webhook

GitHub Plugin

Firewall

Jenkins URL

Webhook Delivery Logs
```

---

## 12. Scenario: Jenkins cannot clone a private repository.

### Answer

Verify

```text
PAT

Credentials

Repository Permissions
```

---

# Key Takeaways

```text
GitHub and Jenkins together enable Continuous Integration.

Webhooks automatically trigger Jenkins pipelines.

Personal Access Tokens are recommended for authentication.

Store credentials securely in Jenkins Credentials.

Webhook is preferred over Poll SCM.

Keep the Jenkinsfile inside the Git repository.

Production pipelines should automatically build, scan, package, and deploy after every successful push.

Enterprise CI/CD commonly follows the flow:
GitHub → Jenkins → SonarQube → OWASP → Docker → Trivy → ECR → EKS.
```