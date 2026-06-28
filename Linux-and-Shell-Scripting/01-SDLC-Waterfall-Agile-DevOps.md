# SDLC, Waterfall, Agile and DevOps

## What is SDLC?

**SDLC (Software Development Life Cycle)** is a structured process used to develop, test, deploy, and maintain software applications efficiently and systematically.

### Why SDLC?

- Improves software quality
- Reduces project risks
- Ensures timely delivery
- Controls project costs
- Meets business requirements

---

# SDLC Phases

## 1. Requirement Gathering

The first phase where business requirements are collected and analyzed.

### Participants

- Business Analyst (BA)
- Product Owner
- Stakeholders
- Clients

### Deliverables

- Business Requirement Document (BRD)
- Software Requirement Specification (SRS)

---

## 2. Planning

The project team estimates:

- Budget
- Resources
- Timeline
- Risks

### Example

Project Cost: ₹100 Crore

Project Duration: 2 Years

---

## 3. Design

The system architecture is designed before development begins.

### Activities

- High-Level Design (HLD)
- Low-Level Design (LLD)
- Database Design
- UI/UX Design

---

## 4. Development

Developers write code based on approved requirements and designs.

### Technologies

- Java
- Python
- Node.js
- React
- Angular

---

## 5. Testing

The QA team validates the application.

### Types of Testing

- Functional Testing
- Integration Testing
- System Testing
- Security Testing
- Performance Testing
- Regression Testing

---

## 6. Deployment

The application is deployed to target environments.

### Examples

- AWS EC2
- Kubernetes
- AWS EKS
- Azure AKS

---

## 7. Maintenance

After production release, teams perform:

- Bug Fixes
- Enhancements
- Security Patches
- Monitoring and Support

---

# School Management System Example

To understand Waterfall, Agile, and DevOps, consider a School Management System.

## Stakeholders

Stakeholders are people directly or indirectly involved in the system.

### Stakeholders Include

- Students
- Teachers
- Parents
- Principal
- School Staff
- Management
- Investors
- Government

---

# Traditional Education Model

## One Final Exam (Waterfall Approach)

Students study throughout the year but are evaluated only once at the end.

### Problems

- Students study seriously only during exam time.
- Teachers don't know student progress regularly.
- Parents receive feedback very late.
- Problems are identified only at the end of the year.

### Result

- Success Rate: 20-30%
- Failure Rate: 70-80%

---

# Improved Model - Unit Tests (Agile Approach)

Instead of waiting for the final exam:

- Unit Test I
- Unit Test II
- Unit Test III
- Unit Test IV
- Quarterly Exam
- Half-Yearly Exam
- Pre-Final Exam

### Benefits

- Early feedback
- Early corrections
- Better student performance

### Result

- Success Rate improves to 70-80%

---

# Best Model - Daily Slip Tests (DevOps Approach)

Students are evaluated continuously.

### Example

- 365 Days in a Year
- 200 Working Days
- 150 Slip Tests

### Benefits

- Immediate feedback
- Immediate corrections
- Continuous improvement
- Better collaboration among students, teachers, and parents

### Result

- Success Rate: 95%

---

# Mapping School Example to Software Industry

| School System | Software Industry |
|---------------|-------------------|
| Final Exam | Waterfall |
| Unit Tests | Agile |
| Daily Slip Tests | DevOps |

---

# Waterfall Model

Waterfall is a sequential software development model where each phase starts only after the previous phase is completed.

## Flow

```text
Requirements
     ↓
Design
     ↓
Development
     ↓
Testing
     ↓
Deployment
     ↓
Maintenance
```

---

## Example Project

- Project Duration: 2 Years
- Budget: ₹100 Crore

### Timeline

- Requirement Gathering: 1 Year
- Development: 6 Months
- Testing: 6 Months
- Deployment: After Testing

---

## Problems in Waterfall

Suppose after 2 years:

- 100 Defects Found
- 10 Invalid Defects
- 90 Valid Defects

### Challenges

- Huge rework is required
- Cost increases significantly
- Delivery gets delayed
- Customer dissatisfaction increases

### Example

Customer requested a Ferrari.

After 2 years, the delivered product resembles a Maruti 800.

Customer rejects the product because feedback was not collected during development.

---

## Advantages of Waterfall

- Easy to understand
- Well documented
- Suitable for fixed requirements

## Disadvantages of Waterfall

- Late feedback
- High risk
- Expensive changes
- Slow delivery
- Minimal customer involvement

---

# Agile Methodology

Agile is an iterative development methodology where software is delivered in small increments called Sprints.

## Agile Flow

```text
Requirement
     ↓
Sprint Planning
     ↓
Development
     ↓
Testing
     ↓
Review
     ↓
Release
     ↓
Next Sprint
```

---

# Example E-Commerce Application

Features:

- Signup/Login
- Product Management
- Order Management
- Shipping
- Payment
- Reviews

---

## Sprint 1

### Feature

Signup and Login

### Duration

- Development: 15 Days
- Testing: 15 Days
- Deployment: End of Sprint

### Result

- 10 Defects
- 2 Invalid Defects

Problems identified much earlier compared to Waterfall.

---

## Sprint 2

### Feature

Product Management

### Activities

- Development
- Testing
- Bug Fixing
- Deployment

---

## Sprint 3

### Feature

Order Management

Same cycle repeats.

---

## Agile Advantages

- Faster feedback
- Better quality
- Lower risk
- Higher customer satisfaction
- Early bug detection

## Agile Disadvantages

- Frequent scope changes
- Requires customer participation
- Difficult long-term estimation

---

## Ferrari Example

### Waterfall

```text
Ferrari Request
        ↓
2 Years Development
        ↓
Maruti 800 Delivery
```

### Agile

```text
Ferrari Request
        ↓
Honda City
        ↓
BMW
        ↓
Ferrari
```

Continuous customer feedback leads to the expected outcome.

---

# DevOps

DevOps is a culture, practice, and set of tools that bring Development, Testing, Operations, and Security teams together to deliver software continuously and reliably.

---

# Traditional Process

```text
Development Team
        ↓
Testing Team
        ↓
Operations Team
        ↓
Production
```

### Problems

- Communication gaps
- Delayed releases
- Manual deployments
- Environment inconsistencies

---

# DevOps Process

```text
Developer Writes Code
        ↓
Git Commit
        ↓
Build
        ↓
Testing
        ↓
Security Scan
        ↓
Deployment
        ↓
Monitoring
        ↓
Feedback
```

---

## Goal of DevOps

> Whatever code is developed today should be tested and deployed on the same day.

---

# DevOps CI/CD Pipeline Example

Developer creates a Signup Form.

Fields:

- First Name
- Last Name

### Automated Pipeline

```text
Git Commit
    ↓
Build Application
    ↓
Unit Testing
    ↓
SonarQube Scan
    ↓
Trivy Scan
    ↓
Docker Build
    ↓
Push to Registry
    ↓
Deploy to DEV
    ↓
Deploy to SIT
    ↓
Integration Testing
    ↓
Approval
    ↓
Deploy to Production
```

---

# DevOps Lifecycle

```text
Plan
 ↓
Code
 ↓
Build
 ↓
Test
 ↓
Release
 ↓
Deploy
 ↓
Operate
 ↓
Monitor
 ↓
Feedback
 ↓
Plan Again
```

---

# Popular DevOps Tools

## Source Code Management

- Git
- GitHub
- GitLab
- Bitbucket

## CI/CD

- Jenkins
- GitHub Actions
- GitLab CI/CD

## Build Tools

- Maven
- Gradle
- NPM

## Configuration Management

- Ansible

## Containerization

- Docker

## Container Orchestration

- Kubernetes
- AWS EKS
- Azure AKS

## Infrastructure as Code

- Terraform

## GitOps

- ArgoCD

## Security

- SonarQube
- Trivy
- Veracode

## Monitoring

- Prometheus
- Grafana
- ELK Stack

---

# Interview Definitions

## What is Waterfall?

Waterfall is a traditional software development model where each phase must be completed before moving to the next phase.

---

## What is Agile?

Agile is an iterative software development methodology that delivers software in small increments called sprints and incorporates continuous customer feedback.

---

## What is DevOps?

DevOps is a culture, practice, and set of tools that improve collaboration between Development, Testing, Operations, and Security teams by automating software delivery and infrastructure management.

---

# Waterfall vs Agile vs DevOps

| Feature | Waterfall | Agile | DevOps |
|----------|-----------|--------|---------|
| Delivery Frequency | End of Project | End of Sprint | Continuous |
| Customer Feedback | Very Late | Sprint Based | Continuous |
| Automation | Minimal | Partial | Extensive |
| Risk | High | Medium | Low |
| Time to Market | Slow | Faster | Fastest |
| Quality | Medium | High | Very High |
| Collaboration | Low | Medium | High |
| Deployment | Manual | Semi-Automated | Automated |

---

# Conclusion

Software development has evolved significantly over time.

- **Waterfall** focuses on sequential execution.
- **Agile** focuses on iterative development and customer feedback.
- **DevOps** extends Agile by introducing automation, CI/CD, monitoring, and collaboration.

### Simple Analogy

| School System | IT Industry |
|---------------|-------------|
| Final Exam | Waterfall |
| Unit Tests | Agile |
| Daily Slip Tests | DevOps |

The faster the feedback cycle, the higher the quality and success rate.
```