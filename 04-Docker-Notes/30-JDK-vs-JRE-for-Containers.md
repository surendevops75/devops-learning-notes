# JDK vs JRE for Containers

## Introduction

When containerizing Java applications, one common question is:

```text
Should We Use

JDK

or

JRE
```

---

Choosing the correct runtime affects:

```text
Image Size

Security

Performance

Build Speed

Production Readiness
```

---

Understanding the difference is important for:

```text
Docker

Kubernetes

EKS

Java Applications
```

---

# What is JDK?

JDK stands for:

```text
Java Development Kit
```

---

JDK contains:

```text
JRE

Compiler

Debugger

Development Tools

Build Utilities
```

---

Architecture

```text
JDK
 │
 ├── JRE
 │
 ├── javac
 ├── jdb
 ├── javadoc
 ├── jar
 └── Other Development Tools
```

---

# Purpose of JDK

JDK is used for:

```text
Application Development

Compilation

Testing

Packaging
```

---

Example

```bash
javac App.java
```

---

Compiler

```text
javac
```

exists only in:

```text
JDK
```

---

# What is JRE?

JRE stands for:

```text
Java Runtime Environment
```

---

JRE contains:

```text
Java Virtual Machine (JVM)

Runtime Libraries

Java Runtime Components
```

---

Architecture

```text
JRE
 │
 ├── JVM
 ├── Runtime Libraries
 └── Supporting Components
```

---

# Purpose of JRE

JRE is used for:

```text
Running Applications
```

---

Example

```bash
java -jar app.jar
```

---

No compilation.

Only execution.

---

# JDK vs JRE

| Feature | JDK | JRE |
|----------|----------|----------|
| Run Java Applications | Yes | Yes |
| Compile Java Code | Yes | No |
| Development Tools | Yes | No |
| JVM Included | Yes | Yes |
| Runtime Libraries | Yes | Yes |
| Image Size | Larger | Smaller |
| Production Runtime | Rarely | Preferred |

---

# Why Does This Matter in Docker?

Suppose application:

```text
catalogue.jar
```

---

Needs:

```text
Build Stage

Runtime Stage
```

---

Build Stage requires:

```text
JDK
```

---

Runtime requires:

```text
JRE
```

---

# Traditional Dockerfile

```dockerfile
FROM openjdk:17

COPY catalogue.jar .

CMD ["java","-jar","catalogue.jar"]
```

---

Problem

```text
JDK Included

Compiler Included

Debugger Included

Unused Tools Included
```

---

Image Size Larger.

---

# Better Approach

Use:

```text
JDK For Build

JRE For Runtime
```

---

# Multi-Stage Build Example

## Builder Stage

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package
```

---

Builder Contains

```text
Maven

JDK

Compiler

Build Tools
```

---

# Runtime Stage

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=builder \
/app/target/catalogue.jar .

CMD ["java","-jar","catalogue.jar"]
```

---

Runtime Contains

```text
JRE Only
```

---

Result

```text
Smaller Image

Less Attack Surface

Production Ready
```

---

# Image Size Comparison

Example

```text
JDK Image
```

Size

```text
400-700 MB
```

---

JRE Image

```text
150-250 MB
```

---

Savings

```text
Hundreds Of MB
```

---

# Security Benefits

JDK Contains

```text
Compiler

Debugging Tools

Development Utilities
```

---

Production applications:

```text
Do Not Need Them
```

---

Using JRE reduces:

```text
Attack Surface

Unused Software

Security Risk
```

---

# Build and Runtime Separation

Modern container strategy:

```text
Build Stage
        ↓
JDK
        ↓
Compile
        ↓
Jar File
        ↓
JRE Runtime
```

---

This is the foundation of:

```text
Multi-Stage Builds
```

---

# Spring Boot Example

Application

```text
catalogue.jar
```

---

Build

```bash
mvn clean package
```

---

Requires

```text
JDK
```

---

Run

```bash
java -jar catalogue.jar
```

---

Requires

```text
JRE
```

---

# Common Base Images

## JDK Images

```text
openjdk

eclipse-temurin

amazoncorretto
```

---

## JRE Images

```text
eclipse-temurin:17-jre

amazoncorretto:17-alpine
```

---

# JDK in Development

Developer Machine

```text
JDK
```

---

Needed for:

```text
Compilation

Debugging

Testing
```

---

# JRE in Production

Container

```text
JRE
```

---

Needed for:

```text
Application Execution
```

---

# Real Production Example

Roboshop Catalogue Service

Build Pipeline

```text
GitHub

Jenkins

Maven

JDK
```

---

Artifact

```text
catalogue.jar
```

---

Container Runtime

```text
JRE
```

---

Benefits

```text
Smaller Images

Faster Pull Time

Lower Security Risk
```

---

# Common Mistakes

## Using JDK in Production

Example

```dockerfile
FROM openjdk:17
```

---

Works.

---

But

```text
Unnecessary Components

Larger Image

Higher Security Exposure
```

---

# Keeping Build Tools in Runtime Image

Example

```text
Maven

Gradle

Compiler
```

---

Not Required.

---

Should remain only in:

```text
Builder Stage
```

---

# Interview Scenario

Question:

```text
Why Use JRE
Instead Of JDK
In Production Containers?
```

---

Answer:

```text
Application Only Needs Runtime.

Compilation Already Completed.

JRE Is Smaller And More Secure.
```

---

# Common Interview Questions

## What is JDK?

### Short Answer

Java Development Kit used to develop and compile Java applications.

### Detailed Explanation

JDK includes JRE plus development tools such as javac, debugger, and packaging utilities.

### Production Example

Building Spring Boot applications using Maven.

### Follow-Up Questions

- Does JDK include JRE?
- Can JDK run applications?

---

## What is JRE?

### Short Answer

Java Runtime Environment used to run Java applications.

### Detailed Explanation

JRE contains JVM and runtime libraries required for application execution.

### Production Example

Running catalogue.jar inside a container.

### Follow-Up Questions

- Does JRE include a compiler?
- Why is JRE smaller?

---

## Why Use JRE in Production Containers?

### Short Answer

Smaller image size and reduced security risk.

### Detailed Explanation

Applications only need runtime components after compilation is complete.

### Production Example

Running Spring Boot applications using eclipse-temurin:17-jre.

### Follow-Up Questions

- Why avoid JDK in production?
- What are the image size benefits?

---

## How Do Multi-Stage Builds Use JDK and JRE?

### Short Answer

JDK is used in the builder stage and JRE is used in the runtime stage.

### Detailed Explanation

Applications are compiled using JDK and only the generated artifacts are copied into a smaller JRE image.

### Production Example

Maven builder image and JRE runtime image.

### Follow-Up Questions

- What gets copied between stages?
- Why does this reduce image size?

---

# Key Takeaways

```text
JDK stands for Java Development Kit.

JDK = JRE + Development Tools.

JRE stands for Java Runtime Environment.

JRE contains JVM and runtime libraries.

JDK is used for building applications.

JRE is used for running applications.

Production containers should prefer JRE over JDK.

JRE images are smaller and more secure.

Multi-stage builds commonly use JDK for build and JRE for runtime.

Using JRE improves image size, security, and deployment efficiency.
```