# Terraform Security Groups and Security Group Referencing

## Revisiting Load Balancer Analogy

To understand AWS Load Balancer easily:

```text
Delivery Manager → Load Balancer

Manager → Listener

Lead → Rule

Team → Target Group

Team Member → Instance

Availability Check → Health Check
```

---

# Listener Analogy

Listener means:

```text
Whom Are We Listening To?
```

Example Organization Flow:

```text
Business Analyst
      ↓
Delivery Manager
```

The Delivery Manager listens to Business Analyst requests.

AWS Equivalent:

```text
Client Request
      ↓
Listener
```

Example:

```text
80

443

8080
```

---

# Rule Analogy

Suppose Business Analyst requests:

```text
Frontend Change
```

Manager decides:

```text
Assign To Frontend Team
```

AWS Equivalent:

```text
Rule
```

Example:

```text
catalogue.daws86s.fun

↓

Catalogue Target Group
```

---

```text
shipping.daws86s.fun

↓

Shipping Target Group
```

---

# Target Group Analogy

A team contains multiple members.

Example:

```text
Frontend Team

Employee-1

Employee-2

Employee-3
```

AWS Equivalent:

```text
Target Group

Instance-1

Instance-2

Instance-3
```

---

# Health Check Analogy

Manager periodically checks:

```text
Are Team Members Available?
```

AWS Equivalent:

```text
Health Check
```

Example:

```text
http://catalogue.daws86s.fun:8080/health
```

---

If response:

```text
200 OK
```

Target becomes:

```text
Healthy
```

---

If continuous failures occur:

```text
Unhealthy
```

Traffic is stopped.

---

# Load Balancer Flow

```text
User
  ↓
ALB
  ↓
Listener
  ↓
Rule
  ↓
Target Group
  ↓
Healthy Instance
```

---

# Example

```text
http://ALB-URL:80
```

Request Flow:

```text
ALB
 ↓
Listener 80
 ↓
Rule
 ↓
Target Group
 ↓
Instance
```

---

# DNS Mapping

Instead of:

```text
http://alb-123456.us-east-1.elb.amazonaws.com
```

we use:

```text
frontend-alb.daws86s.fun
```

Route53 record points to:

```text
ALB DNS Name
```

---

# What is a Security Group?

Security Group acts as:

```text
Virtual Firewall
```

for AWS resources.

Attached to:

```text
EC2

ALB

RDS

EKS Nodes
```

---

# Responsibilities

```text
Allow Traffic

Deny By Default

Control Access
```

---

# Security Group Components

```text
Inbound Rules

Outbound Rules
```

---

# Inbound Rules

Inbound means:

```text
Traffic Entering Resource
```

Example:

```text
Internet
   ↓
Frontend EC2
```

---

# Outbound Rules

Outbound means:

```text
Traffic Leaving Resource
```

Example:

```text
Frontend
    ↓
Catalogue
```

---

# Real Scenario

Suppose:

```text
Instance-A

10.0.11.25
```

and

```text
Instance-B

10.0.12.30
```

---

Requirement

```text
Instance-B

↓

Instance-A
```

communication.

Question:

Which Security Group should allow traffic?

Answer:

```text
Instance-A Security Group
```

because A is receiving traffic.

---

# Important Rule

If:

```text
A Accepting Traffic From B
```

Then:

```text
Ingress Rule Of A

Must Allow B
```

---

# Traditional Method

Allow IP Address:

```text
10.0.12.30/32
```

Example:

```text
Instance-A SG

Ingress

Port 8080

Source:

10.0.12.30/32
```

---

# Problem

If:

```text
Instance-B Recreated
```

New IP:

```text
10.0.12.45
```

Rule breaks.

---

# Better Solution

Use:

```text
Security Group Referencing
```

---

# Security Group Referencing

Instead of:

```text
IP Address
```

allow:

```text
Security Group ID
```

Example:

```text
sg-123456
```

---

# How It Works

Instance-B attached to:

```text
SG-B
```

Instance-A attached to:

```text
SG-A
```

---

Requirement:

```text
B → A
```

Rule:

```text
Ingress Rule Of SG-A

Allow SG-B
```

---

# Diagram

```text
Instance-B
    ↓
 SG-B
    ↓
 SG-A
    ↓
Instance-A
```

---

# Benefits

```text
No Dependency On IP

Works After Instance Recreation

Easy Maintenance

Auto Scaling Friendly
```

---

# Production Example

Catalogue Service:

```text
Catalogue-SG
```

MongoDB:

```text
MongoDB-SG
```

Requirement:

```text
Catalogue

↓

MongoDB
```

Rule:

```text
MongoDB-SG

Ingress

27017

Source:

Catalogue-SG
```

---

# Communication Rule

If:

```text
A Accepting From B
```

Then:

```text
Ingress Of A

Must Allow B
```

---

Examples

```text
Catalogue

↓

MongoDB
```

Rule:

```text
MongoDB SG

Ingress

Allow Catalogue SG
```

---

```text
Shipping

↓

MySQL
```

Rule:

```text
MySQL SG

Ingress

Allow Shipping SG
```

---

```text
Payment

↓

RabbitMQ
```

Rule:

```text
RabbitMQ SG

Ingress

Allow Payment SG
```

---

# Bidirectional Example

Suppose:

```text
A Accepting From B
```

Rule:

```text
Ingress Of A

Allow SG-B
```

---

Suppose:

```text
B Accepting From A
```

Rule:

```text
Ingress Of B

Allow SG-A
```

---

# ALB Security Group Flow

```text
Internet
   ↓
ALB-SG
   ↓
Frontend-SG
```

Rule:

```text
Frontend SG

Ingress

Allow ALB SG
```

---

# Complete Roboshop Flow

```text
Internet
   ↓
ALB
   ↓
Frontend
   ↓
Catalogue
   ↓
MongoDB
```

Security Groups:

```text
Frontend SG

Allow ALB SG
```

```text
Catalogue SG

Allow Frontend SG
```

```text
MongoDB SG

Allow Catalogue SG
```

---

# Terraform Example

```hcl
resource "aws_security_group_rule" "mongodb" {

  type = "ingress"

  from_port = 27017
  to_port   = 27017

  protocol = "tcp"

  security_group_id = aws_security_group.mongodb.id

  source_security_group_id =
  aws_security_group.catalogue.id
}
```

---

# Common Interview Questions

## What is a Security Group?

### Short Answer

A Security Group is a stateful virtual firewall attached to AWS resources.

### Detailed Explanation

Security Groups control inbound and outbound traffic for AWS resources such as EC2, ALB, and RDS.

### Production Example

Frontend Security Group allowing traffic only from the ALB Security Group.

### Follow-Up Questions

- Is Security Group stateful or stateless?
- Can a Security Group deny traffic?

---

## Why Use Security Group Referencing?

### Short Answer

To allow traffic based on Security Groups instead of IP addresses.

### Detailed Explanation

Security Group referencing avoids dependency on changing IP addresses and works well with Auto Scaling environments.

### Production Example

MongoDB SG allowing Catalogue SG instead of allowing specific EC2 IPs.

### Follow-Up Questions

- Why is SG referencing preferred?
- What happens if EC2 IP changes?

---

## If A Accepts Traffic From B, Which Security Group Needs Rule?

### Short Answer

Ingress rule of A should allow B.

### Detailed Explanation

The receiving resource always controls inbound traffic.

### Production Example

MySQL accepting connections from Shipping service.

Rule:

```text
MySQL SG

Ingress

Allow Shipping SG
```

---

# Key Takeaways

```text
Security Groups are stateful firewalls.

Inbound controls incoming traffic.

Outbound controls outgoing traffic.

Always configure ingress on the receiving resource.

Security Group referencing is preferred over IP-based rules.

ALB forwards traffic using listeners, rules, and target groups.

Health checks determine target availability.

SG referencing is heavily used in production environments.
```