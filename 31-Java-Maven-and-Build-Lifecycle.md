# Java, Maven and Build Lifecycle

## Introduction

Many backend applications in enterprises are developed using Java.

Popular examples:

- Banking Applications
- Insurance Applications
- E-Commerce Platforms
- ERP Systems
- Government Applications

In RoboShop:

```text
Shipping Service
```

is developed using Java.

---

# Java Basics

Java source files use:

```text
.java
```

extension.

Example:

```text
HelloWorld.java
Shipping.java
Payment.java
```

---

# Source Code

Source code is the code written by developers.

Example:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

Humans can read source code.

Computers cannot execute source code directly.

---

# Compilation

Compilation is the process of converting source code into machine-understandable format.

```text
Source Code
      │
      ▼
Compilation
      │
      ▼
Byte Code
```

---

# Why Compilation?

Computers understand:

```text
0
1
```

only.

Developers write:

```java
System.out.println("Hello World");
```

Compiler converts it into bytecode.

---

# Java Compilation

Compile:

```bash
javac HelloWorld.java
```

Output:

```text
HelloWorld.class
```

---

# Byte Code

Generated file:

```text
HelloWorld.class
```

contains:

```text
Java Bytecode
```

Bytecode can run on any system having JVM.

---

# Java Execution Flow

```text
HelloWorld.java
        │
        ▼
      javac
        │
        ▼
HelloWorld.class
        │
        ▼
       JVM
        │
        ▼
     Output
```

---

# What is Maven?

Maven is the most popular build tool for Java projects.

Responsibilities:

- Dependency Management
- Compilation
- Testing
- Packaging
- Deployment

---

# Build Tools

| Technology | Build Tool |
|------------|------------|
| NodeJS | npm |
| Java | Maven |
| Python | pip |
| Golang | go modules |

---

# Java Project Structure

```text
shipping/
│
├── src/
├── pom.xml
└── target/
```

---

# What is pom.xml?

POM stands for:

```text
Project Object Model
```

File:

```text
pom.xml
```

is the build file of Java projects.

---

# Information Stored in pom.xml

Contains:

- Project Name
- Description
- Version
- Dependencies
- Build Configuration

Example:

```xml
<project>
</project>
```

---

# Maven Coordinates

Every Maven project is uniquely identified using:

```text
GroupId
ArtifactId
Version
```

---

# GroupId

Represents:

```text
Organization
Project
```

Example:

```text
in.amazon
com.hdfcbank
com.joindevops
```

---

# ArtifactId

Represents:

```text
Module
Component
```

Examples:

```text
cart
payment
shipping
savingbanking
loan
```

---

# Version

Represents application version.

Examples:

```text
1.0
2.0
3.0
```

---

# Example

Amazon Cart Service:

```text
GroupId    : in.amazon
ArtifactId : cart
Version    : 1.0
```

Unique Identity:

```text
in.amazon:cart:1.0
```

---

# Banking Example

Project:

```text
HDFC Bank
```

Modules:

```text
savingbanking
currentbanking
loan
debitcards
creditcards
demataccounts
fd
```

Example:

```text
com.hdfcbank:savingbanking:1.0
```

---

# Dependencies

Applications often require external libraries.

Example:

```text
HTTP Client
Logging Framework
Database Drivers
```

These are called:

```text
Dependencies
```

---

# Dependency Example

```text
com.joindevops.httpClient-1.0
```

or

```text
in.cloudcamp.httpClient-2.0
```

Application downloads and uses these libraries.

---

# Dependency Repositories

Dependencies are stored in repositories.

---

## NodeJS

Dependencies come from:

```text
npm Repository
```

Command:

```bash
npm install
```

---

## Java

Dependencies come from:

```text
Maven Repository
```

Command:

```bash
mvn package
```

---

# Maven Lifecycle

Maven follows a lifecycle.

```text
Validate
   │
   ▼
Compile
   │
   ▼
Test
   │
   ▼
Package
   │
   ▼
Install
   │
   ▼
Deploy
```

---

# Validate Phase

Checks project structure.

Command:

```bash
mvn validate
```

Purpose:

```text
Validate project configuration
```

---

# Compile Phase

Compiles source code into bytecode.

Command:

```bash
mvn compile
```

Flow:

```text
.java
   │
   ▼
.class
```

---

# Test Phase

Runs test cases written by developers.

Command:

```bash
mvn test
```

Purpose:

```text
Verify application functionality
```

---

# Package Phase

Creates deployable application.

Command:

```bash
mvn package
```

Flow:

```text
Validate
Compile
Test
Package
```

All previous phases execute automatically.

---

# Packaging Formats

## JAR

Java Archive File

```text
shipping.jar
```

Commonly used for:

```text
Spring Boot Applications
```

---

## WAR

Web Archive File

```text
shipping.war
```

Used for:

```text
Web Applications
```

---

## EAR

Enterprise Archive File

```text
shipping.ear
```

Used for:

```text
Enterprise Java Applications
```

---

# Understanding Packaging

Example:

```text
.zip
```

appears as a single file.

Internally:

```text
Many Files
```

are packaged together.

Similarly:

```text
.jar
.war
.ear
```

contain multiple files bundled into one deployable package.

---

# Maven Clean

Command:

```bash
mvn clean
```

Purpose:

```text
Remove old builds
Remove old compilation files
```

Deletes:

```text
target/
```

directory.

---

# Common Maven Commands

## Validate

```bash
mvn validate
```

---

## Compile

```bash
mvn compile
```

---

## Test

```bash
mvn test
```

---

## Package

```bash
mvn package
```

---

## Clean

```bash
mvn clean
```

---

## Clean and Package

```bash
mvn clean package
```

Most commonly used command.

---

# Maven Build Flow

```text
Source Code
      │
      ▼
mvn validate
      │
      ▼
mvn compile
      │
      ▼
mvn test
      │
      ▼
mvn package
      │
      ▼
shipping.jar
```

---

# Shipping Service Deployment Flow

```text
Java Source Code
        │
        ▼
      Maven
        │
        ▼
   shipping.jar
        │
        ▼
   Application Server
```

---

# Frequently Asked Interview Questions

## What is Maven?

A build and dependency management tool for Java projects.

---

## What is pom.xml?

Project Object Model file used to define project configuration and dependencies.

---

## What are Maven Coordinates?

```text
GroupId
ArtifactId
Version
```

---

## What is GroupId?

Identifies organization or project.

---

## What is ArtifactId?

Identifies module or component.

---

## What is Version?

Identifies release version of the application.

---

## What is Compilation?

Converting source code into bytecode.

---

## Which Command Compiles Java Code?

```bash
javac HelloWorld.java
```

---

## What Does mvn package Do?

Runs:

```text
Validate
Compile
Test
Package
```

and creates a deployable artifact.

---

## What Does mvn clean Do?

Removes old build artifacts.

---

## What are Common Packaging Formats?

```text
.jar
.war
.ear
```

---

# Key Takeaways

- Java source files use the `.java` extension.
- Java code is compiled into `.class` bytecode files.
- Maven is the standard build tool for Java applications.
- `pom.xml` contains project configuration and dependencies.
- Maven uses GroupId, ArtifactId, and Version to uniquely identify projects.
- Dependencies are downloaded from Maven repositories.
- Maven lifecycle includes Validate, Compile, Test, Package, Install, and Deploy.
- `mvn clean package` is one of the most commonly used Maven commands.
- Java applications are commonly packaged as `.jar`, `.war`, or `.ear` files.
- Shipping Service in RoboShop uses Java and Maven.