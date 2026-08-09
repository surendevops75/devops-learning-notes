# Java CI

Java Continuous Integration (CI) is the process of automatically building, testing, analyzing, and validating Java application changes whenever developers push code or create a Pull Request.

The main goal is to detect problems early and prevent broken code from reaching later stages of the delivery process.

A typical Java CI pipeline looks like:

```text
Developer
    |
    ↓
Git Push / Pull Request
    |
    ↓
CI Pipeline
    |
    +-- Checkout Code
    |
    +-- Setup JDK
    |
    +-- Dependency Download
    |
    +-- Compile
    |
    +-- Unit Tests
    |
    +-- Code Quality
    |
    +-- Security Scan
    |
    +-- Package
    |
    ↓
Build Artifact
```

---

# What Is Java CI?

Java CI automatically validates Java source code whenever a change is submitted.

Typical activities include:

```text
Source Code Checkout
JDK Setup
Dependency Resolution
Compilation
Unit Testing
Code Quality Analysis
Security Scanning
Packaging
Artifact Generation
```

The pipeline provides fast feedback to developers.

---

# Why Java CI Is Important

Without CI, developers may manually perform:

```text
Pull Code
Install Java
Install Dependencies
Compile
Run Tests
Check Code Quality
Build Artifact
```

This can lead to:

```text
Human Errors
Inconsistent Builds
Late Bug Detection
Integration Problems
Broken Releases
```

CI automates these activities.

---

# Java CI Workflow

A typical workflow is:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
CI Pipeline
    |
    +-- Checkout
    +-- JDK
    +-- Maven
    +-- Compile
    +-- Test
    +-- SonarQube
    +-- Trivy
    +-- Package
    |
    ↓
Build Success
    |
    ↓
Artifact
```

---

# Java Build Tools

Common Java build tools include:

```text
Maven
Gradle
Ant
```

In many enterprise Java projects, Maven is commonly used.

Example Maven project:

```text
project/
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    └── test/
        └── java/
```

---

# Maven

Maven is a build automation and dependency management tool.

The main Maven configuration file is:

```text
pom.xml
```

POM means:

```text
Project Object Model
```

The `pom.xml` can define:

```text
Project Information
Dependencies
Plugins
Build Configuration
Java Version
Repositories
Packaging
Profiles
```

---

# Basic Maven Project

Example:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>

    <dependencies>
        ...
    </dependencies>
</project>
```

The important coordinates are:

```text
groupId
artifactId
version
```

---

# Maven Coordinates

A Maven artifact is commonly identified using:

```text
groupId
artifactId
version
```

Example:

```text
com.example
myapp
1.0.0
```

Conceptually:

```text
com.example:myapp:1.0.0
```

---

# Java Version

The CI pipeline must use a supported JDK version.

Example:

```text
JDK 17
```

or:

```text
JDK 21
```

The exact version should match the application's requirements.

Pipeline flow:

```text
CI Runner
    |
    ↓
Install / Configure JDK
    |
    ↓
Maven
    |
    ↓
Build
```

---

# JDK vs JRE

JDK:

```text
Java Development Kit
```

JRE:

```text
Java Runtime Environment
```

For CI builds, the JDK is normally required because the source code needs to be compiled.

```text
Java Source Code
       |
       ↓
      JDK
       |
       ↓
Compiled Classes
```

---

# Checking Java Version

Use:

```bash
java -version
```

Check compiler:

```bash
javac -version
```

Example:

```text
java version "17"
javac 17
```

The CI environment should use the expected version.

---

# Maven Version

Check Maven:

```bash
mvn -version
```

This can show:

```text
Apache Maven Version
Java Version
Java Home
OS Information
```

Example:

```text
Apache Maven 3.9.x
Java version: 17
```

---

# Maven Lifecycle

Maven provides a standard build lifecycle.

Important phases include:

```text
validate
compile
test
package
verify
install
deploy
```

Simplified flow:

```text
validate
   |
   ↓
compile
   |
   ↓
test
   |
   ↓
package
   |
   ↓
verify
   |
   ↓
install
   |
   ↓
deploy
```

---

# Maven Validate

The `validate` phase checks whether the project is correctly structured and configured.

Command:

```bash
mvn validate
```

It is an early validation step.

---

# Maven Compile

Compile Java source code:

```bash
mvn compile
```

Flow:

```text
Java Source
    |
    ↓
Maven Compiler
    |
    ↓
Compiled Classes
```

If compilation fails, the CI pipeline should stop.

---

# Maven Test

Run tests:

```bash
mvn test
```

Typical flow:

```text
Source Code
    |
    ↓
Compile
    |
    ↓
Unit Tests
    |
    ↓
Test Result
```

If required tests fail:

```text
CI Pipeline
    |
    ↓
Test Failure
    |
    ↓
Pipeline Failed
```

---

# Maven Package

Create the application package:

```bash
mvn package
```

Depending on the project, this may create:

```text
JAR
WAR
```

Example:

```text
target/
└── myapp-1.0.0.jar
```

---

# Maven Verify

Run:

```bash
mvn verify
```

This runs the required verification steps after packaging.

It is commonly useful when additional checks or plugins are configured.

---

# Maven Install

Run:

```bash
mvn install
```

This installs the built artifact into the local Maven repository.

Typical location:

```text
~/.m2/repository
```

This is more commonly useful for local development or multi-module builds than for a simple CI artifact publication workflow.

---

# Maven Deploy

Run:

```bash
mvn deploy
```

This publishes artifacts to a configured remote Maven repository.

Examples of artifact repositories include:

```text
JFrog Artifactory
Nexus Repository
Maven Repository Manager
```

---

# Common Maven Commands

Compile:

```bash
mvn compile
```

Test:

```bash
mvn test
```

Package:

```bash
mvn package
```

Verify:

```bash
mvn verify
```

Install:

```bash
mvn install
```

Deploy:

```bash
mvn deploy
```

Clean:

```bash
mvn clean
```

Clean and package:

```bash
mvn clean package
```

Skip tests:

```bash
mvn package -DskipTests
```

Run a specific test:

```bash
mvn -Dtest=PaymentServiceTest test
```

---

# Maven Clean

Command:

```bash
mvn clean
```

This removes the previous build output, normally:

```text
target/
```

A common CI command is:

```bash
mvn clean package
```

This helps ensure the build starts from a clean state.

---

# Clean Build

A clean build looks like:

```text
Previous Build
      |
      ↓
mvn clean
      |
      ↓
Remove target/
      |
      ↓
Compile
      |
      ↓
Test
      |
      ↓
Package
```

This helps avoid problems caused by stale build output.

---

# Maven Dependency Management

Java applications commonly depend on external libraries.

Example:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>...</version>
</dependency>
```

Maven resolves these dependencies automatically.

Flow:

```text
pom.xml
   |
   ↓
Dependencies
   |
   ↓
Maven Repository
   |
   ↓
Local Cache
   |
   ↓
Build
```

---

# Maven Local Repository

Maven stores downloaded dependencies in:

```text
~/.m2/repository
```

Example:

```text
~/.m2/repository/
├── org/
├── com/
└── ...
```

This cache can make subsequent builds faster.

---

# Maven Dependency Tree

To inspect dependencies:

```bash
mvn dependency:tree
```

Example:

```text
com.example:myapp
 |
 +-- spring-core
 |
 +-- spring-web
 |
 +-- jackson
 |
 +-- junit
```

This is useful for troubleshooting dependency issues.

---

# Maven Dependency Resolution

When the pipeline runs:

```bash
mvn clean package
```

Maven may:

```text
Read pom.xml
    |
    ↓
Resolve Dependencies
    |
    ↓
Download Missing Dependencies
    |
    ↓
Compile
    |
    ↓
Test
    |
    ↓
Package
```

---

# Maven Settings

Maven can use:

```text
settings.xml
```

Common locations include:

```text
~/.m2/settings.xml
```

This can configure:

```text
Repositories
Credentials
Mirrors
Profiles
Proxy Settings
```

Sensitive credentials should not be hardcoded into source control.

---

# CI Secrets

Credentials used by the pipeline should be stored securely.

Examples:

```text
Maven Repository Credentials
AWS Credentials
Git Credentials
SonarQube Token
JFrog Credentials
```

Do not put secrets directly into:

```text
pom.xml
Jenkinsfile
GitHub Actions YAML
Source Code
```

Use the CI platform's secret management system.

---

# Java CI Pipeline Stages

A practical Java CI pipeline can contain:

```text
1. Checkout
2. Setup JDK
3. Cache Dependencies
4. Compile
5. Unit Test
6. Code Quality
7. Security Scan
8. Package
9. Publish Artifact
```

Example:

```text
Checkout
   |
   ↓
JDK Setup
   |
   ↓
Maven Dependencies
   |
   ↓
Compile
   |
   ↓
Unit Tests
   |
   ↓
SonarQube
   |
   ↓
Trivy
   |
   ↓
Package
   |
   ↓
Artifact
```

---

# Java CI Checkout Stage

The pipeline first retrieves the source code.

```text
Git Repository
      |
      ↓
CI Runner
      |
      ↓
Source Code
```

Example:

```bash
git checkout main
```

The exact checkout operation is normally handled by the CI system.

---

# Java CI JDK Setup

After checkout:

```text
Source Code
    |
    ↓
JDK Setup
    |
    ↓
Maven
```

Verify:

```bash
java -version
mvn -version
```

The pipeline should fail early if the required Java environment is unavailable.

---

# Java CI Compile Stage

Command:

```bash
mvn clean compile
```

Pipeline:

```text
Source Code
    |
    ↓
Maven
    |
    ↓
Compiler
    |
    ↓
Compiled Classes
```

Compile errors should fail the pipeline.

---

# Java CI Test Stage

Command:

```bash
mvn test
```

Pipeline:

```text
Compile
   |
   ↓
Unit Tests
   |
   ↓
Test Reports
```

If tests fail:

```text
Test Failure
     |
     ↓
CI Failed
```

---

# Unit Testing

Java projects commonly use:

```text
JUnit
TestNG
```

Tests can validate:

```text
Business Logic
Methods
Services
Controllers
Utilities
```

Example:

```text
PaymentService
      |
      ↓
PaymentServiceTest
```

---

# Unit Test Example

A simplified test:

```java
@Test
void shouldCalculateTotal() {
    int result = service.calculateTotal(100, 20);

    assertEquals(120, result);
}
```

The CI pipeline executes the test automatically.

---

# Test Reports

Maven commonly generates test results under:

```text
target/surefire-reports/
```

Example:

```text
target/
└── surefire-reports/
    ├── TEST-PaymentServiceTest.xml
    └── ...
```

CI systems can publish these reports for developers.

---

# Integration Tests

Integration tests verify interactions between components.

Example:

```text
Application
    |
    +-- Database
    |
    +-- API
    |
    +-- External Service
```

Unit test:

```text
Service → Mock
```

Integration test:

```text
Service → Real / Test Dependency
```

---

# Java CI Code Quality

After tests, code quality can be analyzed.

Common tool:

```text
SonarQube
```

Typical flow:

```text
Build
  |
  ↓
Tests
  |
  ↓
SonarQube
  |
  ↓
Quality Gate
```

---

# Java CI SonarQube

A Maven project can integrate with SonarQube.

Example:

```bash
mvn clean verify sonar:sonar
```

The exact command and configuration depend on the SonarQube setup.

The analysis can identify:

```text
Bugs
Vulnerabilities
Code Smells
Duplications
Coverage Information
```

---

# Java CI Security Scanning

Security checks can be added to the pipeline.

Example:

```text
Source Code
    |
    ↓
Build
    |
    ↓
Tests
    |
    ↓
SonarQube
    |
    ↓
Trivy
    |
    ↓
Veracode
    |
    ↓
Package
```

Security tools should be integrated according to the organization's policies.

---

# Trivy in Java CI

Trivy can be used for vulnerability scanning.

Depending on configuration, it can scan:

```text
Filesystem
Dependencies
Container Images
Configuration
```

Example:

```bash
trivy fs .
```

Container scanning:

```bash
trivy image myapp:1.0.0
```

---

# Java Packaging

After successful validation:

```bash
mvn package
```

The output can be:

```text
target/
└── myapp-1.0.0.jar
```

This artifact can then be:

```text
Stored
Published
Containerized
Deployed
```

---

# JAR File

JAR means:

```text
Java Archive
```

Example:

```text
myapp-1.0.0.jar
```

A JAR can contain:

```text
Compiled Classes
Resources
Metadata
Application Code
Dependencies
```

depending on how the project is packaged.

---

# WAR File

WAR means:

```text
Web Application Archive
```

Example:

```text
myapp-1.0.0.war
```

WAR files are commonly associated with Java web applications deployed to servlet containers.

Modern Spring Boot applications are often packaged as executable JARs.

---

# Java CI Artifact Flow

```text
Source Code
    |
    ↓
Maven Build
    |
    ↓
Tests
    |
    ↓
Quality
    |
    ↓
Security
    |
    ↓
Package
    |
    ↓
myapp-1.0.0.jar
    |
    ↓
Artifact Repository
```

---

# Artifact Repository

The generated artifact can be published to a repository.

Examples:

```text
JFrog Artifactory
Nexus
AWS CodeArtifact
```

Example:

```text
Maven Build
    |
    ↓
JAR
    |
    ↓
JFrog Artifactory
```

The next deployment stage can retrieve the exact artifact.

---

# Java CI with JFrog Artifactory

Example flow:

```text
Developer
    |
    ↓
Git
    |
    ↓
CI
    |
    ↓
Maven
    |
    ↓
Build
    |
    ↓
Tests
    |
    ↓
Package
    |
    ↓
JAR
    |
    ↓
JFrog Artifactory
```

Artifact versioning should be consistent.

Example:

```text
myapp-1.0.0.jar
myapp-1.0.1.jar
myapp-1.1.0.jar
```

---

# Java CI with Docker

A Java application can be packaged into a Docker image.

Flow:

```text
Java Source
    |
    ↓
Maven Build
    |
    ↓
JAR
    |
    ↓
Docker Build
    |
    ↓
Docker Image
    |
    ↓
ECR
```

Example:

```text
myapp:v1.0.0
```

---

# Example Dockerfile

A simple Java container:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY target/myapp.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The exact base image and Java version should match the application requirements.

---

# Java CI Docker Build

Pipeline:

```text
Maven
 |
 ↓
mvn clean package
 |
 ↓
target/myapp.jar
 |
 ↓
docker build
 |
 ↓
myapp:v1.0.0
```

Example:

```bash
docker build -t myapp:v1.0.0 .
```

---

# Docker Image Scanning

After building the image:

```bash
trivy image myapp:v1.0.0
```

Flow:

```text
JAR
 |
 ↓
Docker Build
 |
 ↓
Image
 |
 ↓
Trivy
 |
 ↓
Security Result
```

The organization can configure policies for vulnerabilities that should fail the pipeline.

---

# Java CI Pipeline Failure

A CI pipeline should fail when critical required checks fail.

Examples:

```text
Compilation Failure
Unit Test Failure
Quality Gate Failure
Security Gate Failure
Packaging Failure
Artifact Publishing Failure
```

Example:

```text
Compile ✓
   |
Test ✓
   |
SonarQube ✗
   |
Pipeline Failed
```

---

# Fail Fast

CI pipelines should detect obvious failures as early as practical.

Example:

```text
Checkout
   |
   ↓
Compile
   |
   ↓
Test
   |
   ↓
Quality
   |
   ↓
Security
   |
   ↓
Package
```

If compilation fails, there is no reason to continue with later stages.

---

# Java CI Pipeline Success

Successful pipeline:

```text
Checkout ✓
   |
JDK Setup ✓
   |
Dependencies ✓
   |
Compile ✓
   |
Unit Tests ✓
   |
SonarQube ✓
   |
Trivy ✓
   |
Package ✓
   |
Artifact ✓
```

Result:

```text
CI PASSED
```

---

# Java CI in Jenkins

A Jenkins pipeline can be used for Java CI.

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

In a real pipeline, additional stages can be added for quality, security, packaging, and publishing.

---

# Better Jenkins Java Pipeline

A more complete structure:

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Quality') {
            steps {
                sh 'mvn verify'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
```

Security and publishing stages can be added according to project requirements.

---

# Java CI in GitHub Actions

Example:

```yaml
name: Java CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build
        run: mvn clean package
```

This creates a basic Java CI workflow.

---

# GitHub Actions Java CI with Tests

Example:

```yaml
name: Java CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Build and Test
        run: mvn clean verify
```

The `cache: maven` configuration can help reuse Maven dependencies between workflow runs.

---

# GitHub Actions Java CI with Quality

A pipeline can be extended:

```text
Checkout
   |
   ↓
JDK
   |
   ↓
Maven Cache
   |
   ↓
Build
   |
   ↓
Test
   |
   ↓
SonarQube
   |
   ↓
Security
   |
   ↓
Package
```

---

# Java CI with GitLab

A basic GitLab CI pipeline can look like:

```yaml
stages:
  - build
  - test

build:
  image: maven:3.9-eclipse-temurin-17
  stage: build
  script:
    - mvn clean compile

test:
  image: maven:3.9-eclipse-temurin-17
  stage: test
  script:
    - mvn test
```

The exact image and versions should be selected according to the project requirements.

---

# Java CI Environment Variables

Environment variables can be used for configuration.

Examples:

```text
MAVEN_OPTS
JAVA_HOME
SONAR_HOST_URL
SONAR_TOKEN
ARTIFACTORY_URL
```

Sensitive values such as tokens should be stored as CI secrets rather than plain text.

---

# JAVA_HOME

`JAVA_HOME` identifies the Java installation used by tools.

Example:

```bash
export JAVA_HOME=/path/to/jdk
```

Verify:

```bash
echo $JAVA_HOME
```

Then:

```bash
java -version
```

CI runners should have the correct JDK configured.

---

# Maven Options

Maven behavior can be customized using:

```text
MAVEN_OPTS
```

Example:

```bash
export MAVEN_OPTS="-Xmx1024m"
```

This can be useful for builds requiring more JVM memory.

The actual memory configuration should be based on the CI runner capacity and build requirements.

---

# Maven Wrapper

Maven projects can use the Maven Wrapper.

Files include:

```text
mvnw
mvnw.cmd
.mvn/
```

Run:

```bash
./mvnw clean package
```

The Maven Wrapper helps ensure the project uses the expected Maven version.

---

# Why Use Maven Wrapper?

Without wrapper:

```text
CI Runner
   |
   ↓
Installed Maven Version
```

Different runners may have different versions.

With Maven Wrapper:

```text
Project
   |
   ↓
Maven Wrapper
   |
   ↓
Expected Maven Version
```

This can improve build consistency.

---

# Java CI Reproducibility

A reproducible build should use controlled versions of:

```text
JDK
Maven
Dependencies
Plugins
Build Configuration
```

Example:

```text
JDK 17
Maven 3.9.x
Pinned Dependencies
Controlled Plugins
```

This reduces differences between developer and CI environments.

---

# Dependency Version Management

Dependencies should use controlled versions.

Example:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>example-library</artifactId>
    <version>1.2.3</version>
</dependency>
```

Avoid uncontrolled dependency changes.

Dependency updates should be reviewed and tested.

---

# Dependency Vulnerabilities

Java dependencies can contain vulnerabilities.

Example:

```text
Application
   |
   +-- Dependency A
   |
   +-- Dependency B
   |
   +-- Dependency C
```

If Dependency B has a vulnerability:

```text
Dependency B
     |
     ↓
Security Scan
     |
     ↓
Vulnerability
```

The pipeline should detect it according to the organization's security policy.

---

# Java CI Security Pipeline

Example:

```text
Checkout
   |
   ↓
Maven Build
   |
   ↓
Unit Tests
   |
   ↓
SonarQube
   |
   ↓
Dependency / Security Scan
   |
   ↓
Docker Build
   |
   ↓
Trivy Image Scan
   |
   ↓
Package / Publish
```

---

# Java CI Quality Gates

A quality gate can enforce required conditions.

Example:

```text
Code
 |
 ↓
SonarQube
 |
 ↓
Quality Gate
 |
 +-- Pass → Continue
 |
 +-- Fail → Stop
```

This prevents low-quality code from moving forward when configured as a mandatory pipeline check.

---

# Java CI Security Gates

A security gate can work similarly.

```text
Build
 |
 ↓
Security Scan
 |
 ↓
Security Gate
 |
 +-- Pass → Continue
 |
 +-- Fail → Stop
```

The exact threshold should be defined by the organization.

---

# Java CI Artifact Versioning

Artifacts should have meaningful versions.

Example:

```text
myapp-1.0.0.jar
myapp-1.0.1.jar
myapp-1.1.0.jar
```

Avoid ambiguous names such as:

```text
myapp-final.jar
myapp-new.jar
myapp-latest.jar
```

Versioned artifacts improve traceability.

---

# Git Commit to Artifact Traceability

A strong CI pipeline should connect:

```text
Git Commit
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Docker Image
    |
    ↓
Deployment
```

Example:

```text
Commit:
8f3a91d

Artifact:
myapp-1.2.0.jar

Image:
myapp:v1.2.0
```

This makes troubleshooting and rollback easier.

---

# Java CI and Git Tags

A release can be triggered using a Git tag.

Example:

```text
Git Tag
v1.2.0
    |
    ↓
CI Pipeline
    |
    ↓
Maven Build
    |
    ↓
JAR
    |
    ↓
Docker Image
    |
    ↓
ECR
```

This provides a clear relationship between source version and build output.

---

# Java CI Production Flow

A mature Java CI/CD process can look like:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Checkout
    +-- JDK
    +-- Maven
    +-- Compile
    +-- Unit Tests
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    |
    ↓
Code Review
    |
    ↓
main
    |
    ↓
Package
    |
    ↓
JAR
    |
    ↓
Docker Build
    |
    ↓
Image Scan
    |
    ↓
ECR
```

---

# Java CI Best Practices

```text
Use a Supported JDK
Pin Important Versions
Use Maven Wrapper When Appropriate
Keep Dependencies Controlled
Run Tests Automatically
Use Clean Builds
Use Code Quality Checks
Use Security Scanning
Fail on Required Quality/Security Gates
Cache Dependencies Carefully
Publish Versioned Artifacts
Avoid Hardcoded Secrets
Keep CI Configuration in Git
Keep Builds Reproducible
Track Commit to Artifact Relationships
```

---

# Common Java CI Problems

## Compilation Failure

Example:

```text
cannot find symbol
```

Possible causes:

```text
Missing Dependency
Incorrect Java Version
Incorrect Import
Compilation Error
Incompatible Library
```

Check:

```bash
mvn clean compile
```

---

# Dependency Download Failure

Example:

```text
Could not resolve dependencies
```

Possible causes:

```text
Repository Unavailable
Incorrect Dependency Version
Network Problem
Authentication Failure
Proxy Problem
```

Useful command:

```bash
mvn dependency:tree
```

Check Maven repository configuration.

---

# Java Version Mismatch

Example:

```text
Unsupported class file major version
```

Possible cause:

```text
Build JDK ≠ Required JDK
```

Check:

```bash
java -version
mvn -version
```

Ensure the CI runner uses the required Java version.

---

# Test Failure

Example:

```text
Tests run: 100
Failures: 2
```

Pipeline should normally stop:

```text
Build
  |
  ↓
Test
  |
  ↓
Failure
  |
  ↓
Pipeline Failed
```

The developer should investigate the failing tests before merging.

---

# Out of Memory During Maven Build

Possible causes:

```text
Large Project
Large Dependency Graph
Insufficient CI Runner Memory
Heavy Tests
Large JVM Heap Requirement
```

Possible configuration:

```bash
export MAVEN_OPTS="-Xmx2048m"
```

The value should be chosen according to the runner's available memory.

---

# Slow Maven Build

Possible causes:

```text
Dependency Downloads
Large Test Suite
Large Project
Repeated Clean Builds
Slow Network
Heavy Plugins
Limited CI Resources
```

Possible improvements:

```text
Dependency Caching
Parallel Builds Where Appropriate
Build Optimization
Test Optimization
Better CI Runner Resources
```

---

# Maven Dependency Caching

Without cache:

```text
Every CI Run
    |
    ↓
Download Dependencies
    |
    ↓
Build
```

With cache:

```text
CI Run
   |
   ↓
Check Cache
   |
   +-- Hit → Reuse Dependencies
   |
   +-- Miss → Download
   |
   ↓
Build
```

Caching is covered in more detail in:

```text
09-Dependency-Caching.md
```

---

# Java CI Pipeline Optimization

A practical optimization order is:

```text
1. Measure Build Time
2. Identify Slow Stages
3. Cache Dependencies
4. Optimize Tests
5. Avoid Unnecessary Work
6. Parallelize Independent Checks
7. Optimize Docker Build
8. Use Appropriate CI Resources
```

Do not optimize blindly.

Measure first.

---

# Java CI Pipeline Example

Complete conceptual pipeline:

```text
                    Git Repository
                           |
                           ↓
                    Pull Request
                           |
                           ↓
                       CI Runner
                           |
                           ↓
                       Checkout
                           |
                           ↓
                       Setup JDK
                           |
                           ↓
                    Maven Dependency
                           |
                           ↓
                         Compile
                           |
                           ↓
                      Unit Tests
                           |
                           ↓
                       SonarQube
                           |
                           ↓
                         Trivy
                           |
                           ↓
                       Veracode
                           |
                           ↓
                         Package
                           |
                           ↓
                    myapp-1.0.0.jar
                           |
                           ↓
                     Docker Build
                           |
                           ↓
                  myapp:v1.0.0
                           |
                           ↓
                      Image Scan
                           |
                           ↓
                          ECR
```

---

# Java CI Interview Questions

## Basic

1. What is Continuous Integration?
2. What is Java CI?
3. What is Maven?
4. What is `pom.xml`?
5. What is the Maven lifecycle?
6. What is the difference between JDK and JRE?
7. How do you check the Java version?
8. How do you check the Maven version?
9. What is `mvn clean`?
10. What is `mvn compile`?
11. What is `mvn test`?
12. What is `mvn package`?
13. What is a JAR?
14. What is a WAR?
15. What is Maven dependency management?

---

# Intermediate Interview Questions

16. How would you design a Java CI pipeline?
17. What stages would you include in a Java CI pipeline?
18. How do you handle Maven dependencies in CI?
19. How do you cache Maven dependencies?
20. How do you run unit tests in Maven?
21. Where are Maven test reports generated?
22. How do you integrate SonarQube with Maven?
23. How do you integrate Trivy into a Java CI pipeline?
24. How do you publish a JAR to an artifact repository?
25. How do you build a Docker image from a Java application?
26. How do you handle Java version mismatches?
27. How do you troubleshoot Maven dependency failures?
28. How do you troubleshoot slow Maven builds?
29. What is Maven Wrapper?
30. Why is `mvn clean package` commonly used?

---

# Advanced Interview Questions

31. Design a production-grade Java CI pipeline.

32. How would you implement DevSecOps in a Java CI pipeline?

33. How would you integrate SonarQube, Trivy, and Veracode?

34. How would you implement quality gates?

35. How would you implement security gates?

36. How would you optimize a Java CI pipeline that takes 20 minutes?

37. How would you handle Maven dependency caching?

38. How would you make Java builds reproducible?

39. How would you securely provide Maven repository credentials?

40. How would you publish Java artifacts to JFrog Artifactory?

41. How would you build and scan a Java Docker image?

42. How would you trace a production Docker image back to a Git commit?

43. How would you handle a failed Maven build?

44. How would you handle a failed unit test?

45. How would you handle a vulnerable Java dependency?

---

# Scenario Question

## A Java CI pipeline suddenly starts failing during Maven dependency resolution. How would you troubleshoot it?

I would troubleshoot it systematically.

```text
Pipeline Failure
      |
      ↓
Check Maven Error
      |
      ↓
Check Dependency Name/Version
      |
      ↓
Check Repository Availability
      |
      ↓
Check Credentials
      |
      ↓
Check settings.xml
      |
      ↓
Check Network / Proxy
      |
      ↓
Check Dependency Cache
```

Useful commands:

```bash
mvn -version
mvn dependency:tree
mvn clean package
```

I would first determine whether the issue is caused by the dependency itself, repository configuration, authentication, network connectivity, or the CI environment.

---

# Scenario Question

## The Java build works locally but fails in CI. What would you check?

I would compare:

```text
Java Version
Maven Version
Environment Variables
Maven Settings
Dependencies
Operating System
Credentials
Network Access
Build Arguments
```

Commands:

```bash
java -version
mvn -version
echo $JAVA_HOME
```

I would also compare the local and CI Maven configuration and use the Maven Wrapper if appropriate to reduce Maven-version differences.

---

# Scenario Question

## The Java CI pipeline takes 15 minutes. How would you reduce it?

I would first measure each stage.

```text
Checkout       → 20 sec
Dependencies   → 4 min
Compile        → 2 min
Tests          → 6 min
SonarQube      → 1 min
Security       → 1 min
Package        → 1 min
```

Then optimize the slow stages.

Possible improvements:

```text
Maven Dependency Cache
Test Optimization
Parallel Independent Checks
Build Optimization
Docker Layer Caching
CI Runner Optimization
Avoid Unnecessary Rebuilds
```

The goal should be based on measured bottlenecks rather than assumptions.

---

# Scenario Question

## A developer says "It works on my machine." How would you reduce this problem?

I would make the CI environment reproducible.

Use:

```text
Pinned JDK
Maven Wrapper
Controlled Dependencies
Consistent Build Commands
Containerized Build Environment Where Appropriate
Automated CI
```

Example:

```text
Developer
    |
    ↓
Maven Wrapper
    |
    ↓
Expected Maven Version
    |
    ↓
Expected JDK
```

The same build process should be used locally and in CI as much as practical.

---

# Scenario Question

## A Java test fails in CI but passes locally. What would you investigate?

I would check:

```text
Java Version
Environment Variables
Timezone
Locale
Database / External Dependencies
Test Ordering
Shared State
File System Differences
Network Dependencies
Parallel Execution
Resource Availability
```

I would inspect the CI test logs and reproduce the failure using the same environment if possible.

---

# Scenario Question

## A security scan detects a critical vulnerability in a Java dependency. What would you do?

I would:

```text
1. Identify the Vulnerable Dependency
2. Identify the Vulnerable Version
3. Check the Recommended Fixed Version
4. Update the Dependency
5. Run Unit Tests
6. Run Integration Tests
7. Run Security Scan Again
8. Review the Result
9. Merge After Required Gates Pass
```

The pipeline should follow the organization's vulnerability severity and exception policies.

---

# Scenario Question

## How would you design a Java CI pipeline for a microservices application?

Example:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Checkout
    +-- JDK
    +-- Maven
    +-- Dependency Cache
    +-- Compile
    +-- Unit Tests
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    +-- Package
    |
    ↓
JAR
    |
    ↓
Docker Build
    |
    ↓
Image Scan
    |
    ↓
ECR
```

Each service can have an independent build and deployment pipeline where appropriate.

---

# Scenario Question

## How would you connect Java CI to Kubernetes deployment?

Example:

```text
Java Source
    |
    ↓
Maven
    |
    ↓
Tests
    |
    ↓
Security
    |
    ↓
JAR
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

The CI pipeline builds and validates the application, while the GitOps process handles the Kubernetes desired state and deployment.

---

# Scenario Question

## How would you implement rollback for a Java application?

I would use versioned artifacts and container images.

Example:

```text
myapp:v1.0.0
myapp:v1.1.0
myapp:v1.2.0
```

If `v1.2.0` causes a production problem:

```text
v1.2.0
   |
   ↓
Problem
   |
   ↓
Rollback
   |
   ↓
v1.1.0
```

The rollback should use the previously validated artifact or image.

---

# Java CI Best-Practice Checklist

```text
☐ Use a supported JDK
☐ Pin important tool versions
☐ Use Maven Wrapper where appropriate
☐ Keep dependencies controlled
☐ Run clean builds when required
☐ Run unit tests automatically
☐ Publish test reports
☐ Integrate SonarQube
☐ Integrate security scanning
☐ Enforce required quality gates
☐ Enforce required security gates
☐ Cache Maven dependencies
☐ Publish versioned artifacts
☐ Store credentials securely
☐ Build versioned Docker images
☐ Scan container images
☐ Maintain commit-to-artifact traceability
☐ Keep CI configuration in Git
☐ Monitor pipeline execution time
☐ Optimize based on measured bottlenecks
```

---

# Quick Revision

Java CI:

```text
Git
 ↓
Checkout
 ↓
JDK
 ↓
Maven
 ↓
Compile
 ↓
Test
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Package
 ↓
JAR
 ↓
Docker Build
 ↓
Image Scan
 ↓
ECR
```

Important Maven commands:

```bash
mvn clean
mvn compile
mvn test
mvn package
mvn verify
mvn install
mvn deploy
```

Important files:

```text
pom.xml
settings.xml
mvnw
mvnw.cmd
```

Important outputs:

```text
target/
target/surefire-reports/
target/myapp-1.0.0.jar
```

Core idea:

> Java CI automates the process of compiling, testing, analyzing, securing, packaging, and validating Java application changes so that problems are detected early and only validated artifacts move toward deployment.