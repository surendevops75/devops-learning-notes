# 18 --- ALB Ingress --- Production DevOps Capstone

> Deep production guide for AWS Application Load Balancer, AWS Load
> Balancer Controller, Kubernetes Ingress, Route 53, ACM, WAF, security
> groups, target groups, health checks, TLS, Helm, GitOps, multi-cluster
> traffic, troubleshooting, disaster recovery, production YAMLs, and
> senior DevOps interviews.

## Chapter Objective

This chapter explains the complete production request path from DNS to
ALB to Kubernetes Services and pods, including security, reliability,
deployment behavior, observability, and failure recovery.

## 1. ALB and Ingress Overview

In EKS, an Ingress object is Kubernetes desired state; the AWS Load
Balancer Controller watches that state and provisions or configures AWS
Application Load Balancers. The production design must cover DNS, TLS,
security groups, target groups, health checks, routing, controller
permissions, subnet discovery, and failure behavior.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 2. AWS Load Balancer Controller

The AWS Load Balancer Controller is the key integration between
Kubernetes resources and AWS load-balancing services. It can provision
ALBs for Ingress resources and target groups for Services depending on
configuration. Treat the controller as a critical platform component
with controlled IAM permissions, HA, monitoring, and a tested upgrade
process.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 3. Ingress vs Service

A Service provides stable Kubernetes networking to workloads. An Ingress
describes HTTP/HTTPS routing rules. The AWS controller translates those
rules into AWS ALB listeners, listener rules, target groups, and related
resources. Do not treat an Ingress as a replacement for a Service.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 4. ALB Architecture

``` text
Internet / Internal Client
          |
       Route 53
          |
     ALB :443
          |
   Listener + Rules
      /         \
 /catalogue   /cart
     |           |
 Target Group  Target Group
     |           |
 Kubernetes Service
     |
     Pods
```

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 5. Internet-Facing vs Internal ALB

Use an internet-facing ALB for public applications. Use an internal ALB
for private enterprise or service traffic. Subnet selection, DNS,
security groups, and routing must match the intended exposure. Never
make an internal application public simply to simplify connectivity.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 6. Subnet Design

Production ALBs should use appropriate subnets across multiple
Availability Zones. Public ALBs require suitable public subnets and
routing; internal ALBs require private subnets and appropriate VPC
routing. Tagging and subnet discovery must be correct for the
controller.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 7. Availability Zones

An ALB should span multiple Availability Zones to avoid a single-AZ
failure. Backend EKS nodes should also be distributed appropriately. An
ALB with healthy targets in only one AZ can still represent a hidden
availability weakness.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 8. Subnet Tags

The controller commonly relies on subnet tags and/or explicit
configuration to determine eligible subnets. Verify tags rather than
assuming discovery is correct. Wrong tags can cause provisioning
failures or place resources in unintended networks.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 9. Security Groups

ALB security groups control traffic reaching the load balancer. Backend
security groups and Kubernetes networking must permit the ALB-to-target
path. Use the narrowest source and destination rules that support the
application.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 10. Security Group for Pods

Where the platform uses security groups for pods, the target security
model can be more granular than node-level security groups. Validate how
the selected target mode and CNI configuration affect traffic
enforcement.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 11. Target Types

ALB target groups can use instance or IP targets depending on controller
configuration and Kubernetes service design. IP targets can route
directly to pod IPs and are common in EKS designs. Choose deliberately
and verify health checks and networking.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 12. IngressClass

Use an explicit IngressClass such as an AWS ALB class so ownership is
unambiguous. Avoid relying on implicit defaults in a multi-controller
cluster. This becomes especially important when NGINX or other ingress
controllers coexist.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 13. IngressGroup

AWS Load Balancer Controller supports grouping multiple Ingress
resources onto a shared ALB through group configuration. This can reduce
load-balancer count but increases shared blast radius. Only use it when
ownership, listener rules, and security boundaries are clearly defined.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 14. Shared ALB Risk

If several teams share one ALB, a configuration change in one Ingress
can affect listener rules or routing for another service. Apply RBAC,
namespace ownership, group conventions, and review controls to prevent
cross-team interference.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 15. Host-Based Routing

Host rules route requests such as api.example.com or shop.example.com to
different target groups. DNS must point the intended hostname to the
ALB. TLS certificates must cover the hostnames clients use.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 16. Path-Based Routing

Path rules can route /catalogue and /cart to different services.
Normalize path behavior and understand how rewrite or prefix semantics
work in the selected controller/version. Test trailing slashes, encoded
paths, and unexpected paths.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 17. Listener Ports

Typical public production configuration uses HTTP 80 for redirect and
HTTPS 443 for application traffic. Avoid serving sensitive applications
over plain HTTP. Listener behavior should be defined declaratively.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 18. HTTP to HTTPS Redirect

A production ALB can redirect HTTP to HTTPS. Test that the redirect
preserves the expected host and URI and does not create loops. Health
checks should target a suitable backend path rather than assuming the
public redirect behavior is healthy.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 19. TLS Termination

TLS can terminate at the ALB, reducing certificate-management complexity
for application pods. The backend connection can remain HTTP when
network trust and application requirements permit it, or HTTPS can be
used for end-to-end encryption.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 20. ACM Certificates

AWS Certificate Manager can provide certificates for ALB listeners.
Certificates should be provisioned and renewed through an approved
lifecycle. Monitor certificate expiration and ensure DNS validation or
the chosen validation process remains operational.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 21. Certificate Selection

Ingress annotations or controller configuration can associate the
appropriate ACM certificate with the ALB listener. Verify certificate
ARN, region, account, and domain coverage. An ACM certificate in the
wrong region cannot be used by an ALB in another region.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 22. TLS Policy

Use an approved modern TLS security policy. The exact policy name should
be selected according to the organization's security standard and AWS
support for the deployed ALB. Test client compatibility before
tightening policy.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 23. TLS Backend Encryption

If TLS is required from ALB to pod, configure the backend protocol and
certificates appropriately. Validate certificate trust, SNI behavior,
health checks, and application listener configuration.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 24. Route 53

Route 53 can map application hostnames to ALB endpoints using alias
records. Production DNS should be managed declaratively where possible
and should have clear ownership so multiple controllers do not conflict.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 25. DNS and ALB Lifecycle

An ALB can be recreated during configuration changes or cluster
lifecycle events depending on the design. Use stable DNS aliases rather
than hardcoding ALB DNS names into application configuration.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 26. ExternalDNS

ExternalDNS can automatically manage DNS records from Kubernetes
resources. If used, restrict which zones and records it can modify and
prevent untrusted namespaces from creating records for sensitive
production domains.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 27. WAF Integration

AWS WAF can protect an ALB against common web attack patterns, malicious
request rates, and application-specific rules. WAF rules should be
tested in a safe mode before enforcing aggressive blocks.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 28. WAF Rule Design

Use managed protections where appropriate, supplemented by
application-specific rules such as rate limiting and known malicious
patterns. Avoid broad IP blocks that accidentally deny legitimate
customers.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 29. WAF Observability

Monitor blocked requests, rule IDs, source patterns, and false
positives. During an incident, determine whether customer impact comes
from the application or a WAF rule before disabling protections
globally.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 30. ALB Access Logs

ALB access logs provide request-level evidence useful for
troubleshooting latency, status codes, target behavior, and suspicious
traffic. Store logs in an appropriately protected S3 location and define
retention.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 31. CloudWatch Metrics

Monitor ALB request count, target response time, HTTP 4xx/5xx counts,
rejected connections, healthy/unhealthy target counts, and other
relevant metrics. Correlate ALB metrics with Kubernetes and application
telemetry.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 32. Prometheus Integration

AWS infrastructure metrics can be collected through the organization's
monitoring architecture, while Kubernetes controller metrics and
application metrics can be scraped by Prometheus. Use consistent
service, cluster, namespace, and environment labels.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 33. Health Checks

ALB target health checks determine whether traffic is sent to a target.
The endpoint should be lightweight, deterministic, and representative of
whether the process can serve requests. Do not make a health check
depend on a slow business transaction.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 34. Readiness vs ALB Health

Kubernetes readiness and ALB health checks solve related but different
problems. Readiness controls Kubernetes endpoint eligibility; ALB health
checks independently determine target health. Configure them so they do
not contradict the application's lifecycle.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 35. Health Check Path

Choose a path such as /health or /ready that returns a fast expected
status. If authentication is required for normal endpoints, create a
safe health endpoint rather than embedding credentials into health
checks.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 36. Health Check Interval

Tune interval, timeout, healthy threshold, and unhealthy threshold based
on application startup and failure characteristics. Aggressive settings
can cause flapping; slow settings delay failure detection.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 37. Deregistration Delay

ALB target deregistration delay gives existing connections time to
complete. Align this with Kubernetes terminationGracePeriodSeconds and
application shutdown behavior. A mismatch can cause dropped requests
during rolling deployments.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 38. Connection Draining

Graceful shutdown should stop accepting new work, allow active requests
to finish, and then terminate. Test long-running requests because
defaults that work for normal HTTP traffic may be insufficient for
streaming or slow operations.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 39. Pod Termination

When a pod terminates, Kubernetes lifecycle, endpoint updates,
application preStop behavior, and ALB target deregistration interact.
Use sufficient termination grace and verify actual traffic behavior
under load.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 40. Rolling Deployment with ALB

A safe rollout maintains healthy old targets while new targets pass
readiness and ALB health checks. Configure maxUnavailable and maxSurge
to preserve capacity. Test rollouts with real traffic rather than only
checking Deployment status.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 41. Pod Readiness Gates

AWS Load Balancer Controller can use readiness-gate behavior so
Kubernetes readiness reflects target registration health in supported
configurations. This can prevent a pod from being considered fully ready
before the ALB target is healthy.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 42. Readiness Gate Troubleshooting

If pods remain unready, inspect target registration, target health
reason codes, security groups, service selectors, health-check path, and
controller events. Do not remove readiness behavior simply to force a
deployment green.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 43. Ingress Annotations

Controller annotations influence ALB scheme, target type, health checks,
listener behavior, grouping, certificates, attributes, and other
features. Treat annotations as production configuration and validate
them against the installed controller version.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 44. Annotation Version Compatibility

Do not copy annotations from an old blog or unrelated cluster.
Controller features and annotation names can change. Verify against the
exact AWS Load Balancer Controller version deployed in the cluster.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 45. Production Ingress Example

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: catalogue
  namespace: catalogue
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/certificate-arn: <acm-certificate-arn>
spec:
  ingressClassName: alb
  rules:
    - host: catalogue.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: catalogue
                port:
                  number: 8080
```

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 46. Production Service Example

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: catalogue
spec:
  selector:
    app: catalogue
  ports:
    - port: 8080
      targetPort: 8080
```

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 47. AWS Controller Installation Strategy

Install the AWS Load Balancer Controller through the platform's approved
Helm/Terraform/GitOps mechanism. Configure its service account identity
and IAM permissions using the chosen EKS workload identity approach. Do
not manually create broad IAM credentials inside the cluster.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 48. Controller IAM

The controller requires AWS permissions to discover networking resources
and create or modify load-balancer-related resources. Use the official
policy appropriate to the controller release and restrict its identity
to the intended AWS account. Review policy changes during upgrades.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 49. Controller Service Account

Use a dedicated Kubernetes service account for the controller. Do not
reuse an application service account. The controller's IAM role is
highly privileged compared with ordinary application roles.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 50. Controller High Availability

Run the controller with production-appropriate replicas and scheduling.
Protect it with resource requests, disruption budgets where appropriate,
and anti-affinity or topology spread so one node failure does not remove
all controller capacity.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 51. Controller Upgrades

Upgrade the controller in staging before production. Verify CRDs,
annotations, IAM policy requirements, Kubernetes compatibility, existing
ALBs, and reconciliation behavior. Keep a tested rollback path.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 52. Controller Failure

If the controller becomes unavailable, existing ALBs and workloads may
continue serving traffic, but new Ingress changes and reconciliation can
be delayed. Monitor controller health separately from application
availability.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 53. Ingress Drift

If an ALB differs from Git desired state, determine whether the
controller has reconciled the latest manifest, whether another tool
modified the AWS resource, or whether the controller is failing. Avoid
manual AWS-console changes that GitOps will immediately overwrite.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 54. AWS Console Changes

Manual console changes are useful for investigation but should not
become the normal configuration path. If a change is required, encode
the desired final state in Kubernetes/GitOps and let the controller
reconcile it.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 55. ALB Naming

Use controller-supported naming or grouping mechanisms rather than
assuming an ALB name is a stable application identifier. Record the
relationship between Ingress, ALB, listener, target group, and DNS name.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 56. Target Group Mapping

When troubleshooting, map Kubernetes Ingress -\> Service -\>
endpoints/pods -\> target group -\> ALB listener rule. This chain
identifies where requests are being lost.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 57. Ingress to Service Selector

A common failure is a Service selector that matches zero pods. Verify
Service endpoints before troubleshooting AWS. If there are no endpoints,
the ALB cannot successfully route traffic.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 58. Endpoints and EndpointSlices

Inspect EndpointSlices to confirm the Service has expected pod IPs and
ports. Compare endpoint readiness with pod readiness. A healthy
Deployment does not guarantee a healthy Service endpoint set.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 59. Port Mismatch

A mismatch among containerPort, Service targetPort, Service port, and
Ingress backend port can produce connection failures or unhealthy
targets. Trace the complete port path rather than checking only the
Ingress.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 60. Target Health Reason Codes

ALB target health reason codes help distinguish timeout, connection
failure, response-code mismatch, target deregistration, and other
conditions. Use the AWS load-balancer target-group diagnostics together
with Kubernetes events.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 61. 503 Troubleshooting

An ALB 503 commonly indicates that no healthy targets are available.
Check target health, Service endpoints, readiness, health-check path,
ports, security groups, and application listener behavior.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 62. 502 Troubleshooting

A 502 can indicate an upstream connection or protocol problem. Check
target response, backend protocol, connection resets, application
crashes, TLS configuration, and target health.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 63. 404 Troubleshooting

A 404 may be generated by the ALB rule set or the application. Confirm
the Host header, path, listener rule priority, default action, and
backend application routes.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 64. 403 Troubleshooting

A 403 may originate from WAF, ALB authentication/rules, an upstream
proxy, or the application. Use access logs and application logs to
locate the source.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 65. 504 Troubleshooting

A 504 often indicates a backend response timeout. Check application
latency, dependency calls, ALB idle timeout, target health, network
paths, and long-running request behavior.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 66. DNS Troubleshooting

Verify the hostname resolves to the intended ALB, DNS record is current,
resolver caches are considered, and there are no conflicting records.
Use the exact hostname clients report rather than testing only the ALB
DNS name.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 67. TLS Troubleshooting

For certificate failures, check certificate domain coverage, ACM status,
listener certificate association, DNS, SNI behavior, certificate chain,
and client compatibility. Confirm that the certificate belongs to the
correct AWS region/account.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 68. Security Group Troubleshooting

For connection timeouts, inspect ALB security group ingress and target
security group rules. Verify the source identity used by the target
path. A permissive security group may hide the actual design flaw.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 69. Network ACL Troubleshooting

If security groups look correct but traffic fails, inspect subnet
routing and network ACLs. Ensure return traffic is permitted. Avoid
modifying multiple network controls at once during incident response.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 70. NAT and Egress

ALB-to-pod traffic does not normally require NAT because both endpoints
are inside the VPC, but applications may require outbound internet
access for dependencies. Keep ingress troubleshooting separate from
application egress troubleshooting.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 71. Private ALB

Internal ALBs require private DNS resolution and network reachability
from clients. Validate Route 53 private hosted zones, VPC association,
security groups, and routing from the caller's network.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 72. Public ALB

Internet-facing ALBs require public subnet placement and appropriate
internet gateway routing. Restrict backend exposure so clients cannot
bypass the ALB unless direct access is explicitly intended.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 73. WAF and Private ALB

WAF can protect applicable ALB traffic regardless of whether the ALB is
internet-facing or internal, subject to AWS service capabilities. Use
WAF where the threat model requires it, not as a substitute for private
networking.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 74. Authentication at ALB

ALB supports integration patterns for authentication in appropriate
architectures. If used, document where authentication occurs, what
identity reaches the application, and how health checks bypass or
satisfy authentication safely.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 75. Header Handling

Applications should not blindly trust client-supplied forwarding
headers. Understand which headers the ALB adds and how the
application/framework interprets them. Incorrect proxy configuration can
cause wrong redirects, scheme detection, or client IP reporting.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 76. Client IP

If the application requires source IP information for auditing or rate
limiting, verify the forwarded-header behavior and ensure trusted proxy
boundaries are configured correctly. Do not accept arbitrary forwarding
headers from untrusted clients as authoritative.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 77. ALB Idle Timeout

Tune the ALB idle timeout for application behavior such as long polling,
streaming, or slow requests. Increasing the timeout without controlling
application resource usage can increase connection pressure.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 78. Request Size

ALB and application request-size limits should be considered for uploads
and large payloads. Large request support should be deliberate and
monitored rather than discovered through production failures.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 79. WebSockets

If the application uses WebSockets, verify ALB support and timeout
behavior and test reconnect handling. Load balancing does not eliminate
the need for application-level session and reconnection design.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 80. Sticky Sessions

Avoid sticky sessions unless the application genuinely requires them.
Stateless applications are easier to scale and fail over. If stickiness
is enabled, understand how it affects rolling deployments and target
distribution.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 81. Autoscaling Interaction

When HPA adds pods, Service endpoints and ALB target registration must
converge before the new pods receive production traffic. Scale-down
should similarly allow graceful deregistration.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 82. HPA and ALB Metrics

CPU-based HPA alone may not represent user-facing load. Combine
application metrics such as request rate and latency with resource
metrics where appropriate. ALB request metrics can provide another
signal for capacity planning.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 83. Cluster Autoscaler / Karpenter

If new pods cannot schedule, ALB target health may fall as existing
capacity is exhausted. Investigate node capacity, pending pods, and node
provisioning alongside load-balancer symptoms.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 84. Blue-Green with ALB

Blue-green deployment can use separate target groups or Services and
controlled routing. The exact mechanism should be implemented through
supported Kubernetes/controller features. Validate both environments
before shifting traffic.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 85. Canary with ALB

Canary routing can split traffic between versions using appropriate ALB
rule or weighted-routing capabilities where supported. Keep the routing
model observable and reversible. A canary is useful only when metrics
can distinguish versions.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 86. Weighted Traffic

When using weighted target groups or equivalent capabilities, define
safe weights and health requirements. Start with a small percentage,
observe errors and latency, then increase gradually.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 87. Multi-Cluster ALB

Each EKS cluster normally has its own load-balancer resources.
Multi-cluster traffic can be implemented above the cluster layer using
DNS or another global routing mechanism. Avoid assuming one ALB can
transparently replace the entire multi-cluster architecture.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 88. Cross-Region Routing

For multi-region clusters, Route 53 routing, health checks, or another
traffic layer can select a regional ALB. Validate health at the
application level and consider DNS propagation and client caching.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 89. ALB and Disaster Recovery

A DR cluster should have a functional ingress path, certificate, DNS
strategy, target health, security groups, and capacity. A standby
cluster with no working ALB path is not production-ready DR.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 90. Ingress Security

Restrict who can create or modify production Ingress resources. Ingress
can expose services publicly, alter TLS, attach WAF settings, and
potentially join shared ALBs. Treat it as a security-sensitive API.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 91. RBAC for Ingress

Application teams should normally manage only their namespaces.
Cluster-wide IngressClass, shared ALB groups, certificates, WAF
associations, and DNS capabilities should have controlled ownership.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 92. Namespace Isolation

Do not allow one team's Ingress configuration to route traffic to
another team's service without authorization. Shared ALB grouping
increases the importance of namespace and RBAC boundaries.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 93. Admission Policies

Use admission policy tooling where appropriate to enforce rules such as
approved ingress classes, required TLS, approved hostnames, prohibited
public exposure, and required annotations. Policy should fail safely and
provide actionable messages.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 94. Public Hostname Governance

Production public hostnames should come from an approved domain
inventory. Prevent arbitrary namespaces from creating sensitive
hostnames. DNS automation should enforce similar boundaries.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 95. TLS Governance

Require HTTPS for production applications containing credentials or
sensitive data. Validate that certificates are approved, current, and
cover the intended host.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 96. Ingress Manifest Validation

Validate manifests with schema checks, Kubernetes dry-run, policy
checks, Helm rendering, and controller-compatible tests before
production promotion.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 97. GitOps Workflow

A production ingress change should flow through Git, CI validation,
security/policy checks, review, Argo CD synchronization, and
post-deployment verification. Avoid manual production edits as the
primary workflow.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 98. Helm Ingress Values

Keep hostnames, certificate identifiers, ALB scheme, and approved
routing settings configurable through environment-specific values. Do
not place account credentials or secret values in Helm values.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 99. Helm Example

``` yaml
ingress:
  enabled: true
  className: alb
  host: catalogue.example.com
  tls:
    certificateArn: <approved-acm-arn>
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
```

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 100. CI Validation

CI should render Helm templates, validate Kubernetes schemas, run policy
checks, inspect dangerous ingress annotations, and optionally perform
server-side dry-run against a controlled cluster. The pipeline should
reject accidental public exposure where policy forbids it.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 101. Production Deployment Checklist

-   [ ] Correct AWS account and region
-   [ ] Correct subnets and subnet tags
-   [ ] Controller healthy
-   [ ] Controller IAM correct
-   [ ] IngressClass correct
-   [ ] ALB scheme intentional
-   [ ] TLS certificate valid
-   [ ] DNS record correct
-   [ ] Security groups reviewed
-   [ ] Service has endpoints
-   [ ] Target health is healthy
-   [ ] Health-check path tested
-   [ ] HTTP/HTTPS behavior verified
-   [ ] WAF rules validated where required
-   [ ] Access logs and metrics available
-   [ ] Rollback path tested

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 102. Production Troubleshooting Flow

Start at the client symptom and trace downward: DNS -\> ALB listener -\>
listener rule -\> target group -\> Service -\> EndpointSlice -\> Pod -\>
application -\> dependency. At each layer, prove the state before
changing anything.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 103. Troubleshooting: DNS Works but 503

DNS and ALB reachability are proven. Move directly to target health,
Service endpoints, readiness, health-check path, ports, and security
groups. Do not spend time changing DNS when the ALB is already receiving
requests.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 104. Troubleshooting: Pods Healthy but Targets Unhealthy

Kubernetes pod readiness does not guarantee ALB target health. Check
target registration mode, target port, health-check response, security
groups, route tables, and application binding address.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 105. Troubleshooting: New Pods Not Receiving Traffic

Check readiness gates, target registration delay, Service
EndpointSlices, ALB target health, rollout state, and deregistration
behavior. Compare old and new pod configuration.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 106. Troubleshooting: HTTPS Certificate Wrong

Verify the requested hostname, DNS destination, ALB listener, ACM
certificate association, SAN coverage, and SNI behavior. Check whether
multiple certificates or listeners are involved.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 107. Troubleshooting: Ingress Not Creating ALB

Check IngressClass, controller logs, Kubernetes events, AWS
credentials/IAM, subnet discovery, security-group configuration,
resource quotas, and AWS API errors.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 108. Troubleshooting: Controller Logs

Controller logs should be correlated with the Ingress name, namespace,
reconciliation event, AWS resource, and error message. Avoid treating
every warning as an incident; identify whether reconciliation is
actually failing.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 109. Troubleshooting: AWS API Errors

For AWS authorization errors, identify the exact API action and
resource. Compare it with the controller IAM policy for the deployed
version. Do not attach AdministratorAccess as a shortcut.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 110. Troubleshooting: ALB Rule Priority

When multiple host/path rules overlap, inspect actual listener rule
priorities and conditions. A more specific application rule may still
behave unexpectedly if grouping or generated configuration changes the
final rule set.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 111. Troubleshooting: Shared ALB

If one application breaks after another team's Ingress change, inspect
shared IngressGroup configuration, rule conditions, group ownership, and
recent Git changes. Shared infrastructure requires shared change
controls.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 112. Incident: ALB Public Exposure

If an internal application becomes publicly reachable, immediately
restrict the exposure path according to incident procedures, inspect the
Ingress and generated ALB configuration, identify the Git change, verify
DNS, and audit access logs. Then add policy controls to prevent
recurrence.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 113. Incident: Certificate Expiry

If a certificate is near expiry or expired, validate ACM status and
listener association, restore the valid certificate, verify TLS from
multiple clients, and investigate why renewal monitoring failed.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 114. Incident: WAF False Positive

Identify the blocking rule and affected requests, confirm the
application behavior, adjust the narrowest rule or exception, and
monitor after the change. Avoid disabling the entire WAF without a
documented risk decision.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 115. Incident: Regional ALB Failure

If one region's ALB path is degraded, compare regional target health and
application metrics, validate the alternate region, and execute the
documented traffic failover. After recovery, determine whether the root
cause was ALB, networking, EKS, or application.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 116. Incident: Ingress Controller Compromise

Treat controller IAM and Kubernetes permissions as high-impact. Isolate
according to security procedures, rotate affected credentials, inspect
controller and AWS audit logs, review generated load-balancer changes,
and validate all production ingress resources.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 117. DR Runbook

1.  Validate secondary EKS cluster. 2. Confirm controller and IAM. 3.
    Confirm ALB exists or can be provisioned. 4. Confirm ACM
    certificate. 5. Confirm DNS/routing. 6. Confirm Service endpoints
    and target health. 7. Shift traffic according to the approved
    mechanism. 8. Run application smoke tests. 9. Monitor errors and
    latency.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 118. Rollback Runbook

1.  Identify the last known-good Git commit. 2. Revert ingress or Helm
    configuration. 3. Sync through Argo CD. 4. Verify ALB listener/rules
    and target health. 5. Validate DNS/TLS. 6. Run smoke tests. 7.
    Monitor. 8. Record the incident and prevention action.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 119. Observability Dashboard

A useful dashboard combines ALB request count, 4xx, 5xx, target response
time, healthy targets, unhealthy targets, Kubernetes pod readiness,
deployment rollout state, node capacity, application error rate, and
dependency latency. Add cluster, namespace, service, region, and
environment dimensions.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 120. SLOs for Ingress

Define user-facing SLOs such as availability and latency at the
ALB/application boundary. Infrastructure health alone is insufficient:
an ALB can be healthy while every application request returns an error.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 121. Capacity Planning

Estimate peak requests, concurrent connections, target capacity, pod
resources, node capacity, and scaling time. Test traffic spikes and
target-registration delays. Ensure the system has sufficient headroom
for one-AZ or one-node failures.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 122. Cost Optimization

Avoid unnecessary ALBs because each load balancer and associated
resources add cost. Shared ALBs can reduce cost but increase blast
radius. Choose based on ownership, security, and operational boundaries
rather than cost alone.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 123. ALB Attributes

ALB attributes can influence behavior such as idle timeout and access
logging. Manage them declaratively through supported controller
configuration and review changes because they can have production-wide
impact.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 124. Deletion Protection

Where supported and appropriate, load-balancer deletion protection can
reduce accidental deletion risk. Understand how this interacts with
GitOps deletion and cluster decommissioning before enabling it.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 125. Final Architecture

``` text
                           Route 53
                              |
                     +--------+--------+
                     |                 |
               Region A ALB       Region B ALB
               /   |   \          /   |   \
          WAF/TLS Rules           WAF/TLS Rules
                |                     |
             EKS A                  EKS B
                |                     |
        AWS LB Controller      AWS LB Controller
                |                     |
       Services / Pods         Services / Pods
                \_____________________/
                         |
                 Central Observability

Git -> CI/Policy -> Argo CD -> Kubernetes Ingress
                         |
                 AWS Load Balancer
                    Controller
                         |
                    AWS ALB
```

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 126. Senior Interview: Explain ALB Ingress

In EKS, Kubernetes Ingress expresses HTTP routing intent, while AWS Load
Balancer Controller reconciles that intent into AWS ALB resources. I
design the complete chain from Route 53 and ACM through listeners and
target groups to Services and pods, with IAM, security groups, health
checks, observability, and GitOps controls.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 127. Senior Interview: 503 From ALB

I first confirm DNS and listener reachability, then inspect target-group
health. If targets are unhealthy, I trace target registration, Service
endpoints, pod readiness, ports, health-check path, security groups, and
application binding. I avoid changing multiple layers simultaneously.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 128. Senior Interview: Why IP Target Type?

IP targeting can route traffic directly to pod IPs and can simplify
scaling behavior in EKS when the networking model supports it. I choose
it based on the controller, CNI, security model, and operational
requirements rather than treating it as universally superior.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 129. Senior Interview: ALB vs NGINX Ingress

ALB integrates directly with AWS networking and managed load-balancing
capabilities, while NGINX provides a different feature and operational
model. In an AWS environment I choose based on requirements such as
AWS-native integration, routing features, security controls,
portability, and existing platform standards.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 130. Senior Interview: How Do You Secure Ingress?

I restrict who can create Ingress resources, enforce approved
IngressClasses and hostnames, require TLS, use least-privilege
controller IAM, restrict security groups, integrate WAF where
appropriate, protect DNS automation, and monitor ALB access and
application traffic.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 131. Senior Interview: How Do You Handle Multi-Cluster Ingress?

Each cluster has its own ingress path, usually an ALB. A higher-level
DNS or traffic-management layer selects the cluster or region. The
design must include health-based failover, certificates, capacity,
application state, and tested recovery rather than relying only on
Kubernetes replication.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 132. Senior Interview: Zero-Downtime Deployment

I coordinate Kubernetes readiness, ALB target health, graceful
termination, deregistration delay, and rollout capacity. New pods must
become genuinely healthy before old capacity is removed, and
long-running requests must be tested.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 133. Senior Interview: Controller Failure

Existing ALB traffic can continue if the generated AWS resources and
workloads remain healthy, but reconciliation of new changes can stop. I
run the controller HA, monitor it separately, and maintain a tested
recovery and upgrade procedure.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 134. Senior Interview: Terraform vs Argo CD for ALB

Terraform can provision foundational AWS networking and cluster
infrastructure. Argo CD should manage Kubernetes Ingress and application
desired state. The AWS Load Balancer Controller then reconciles Ingress
into ALB resources. I avoid having Terraform and the controller fight
over the same dynamically managed ALB configuration.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?

## 135. Final Production Rules

1.  Treat Ingress as security-sensitive configuration. 2. Use explicit
    IngressClass. 3. Use least-privilege controller IAM. 4. Keep ALBs
    multi-AZ. 5. Use ACM and HTTPS for production. 6. Protect DNS
    automation. 7. Monitor target health. 8. Align readiness and
    graceful shutdown. 9. Validate controller-version compatibility. 10.
    Test rollback and DR.

### Production validation

-   Verify the intended AWS account, region, VPC, subnets, and cluster.
-   Confirm the controller is healthy and authorized only for required
    AWS resources.
-   Trace traffic through DNS -\> ALB -\> target group -\> Service -\>
    EndpointSlice -\> Pod.
-   Validate TLS, security groups, health checks, and graceful
    termination.
-   Keep the final configuration in GitOps and test rollback.

### Operator questions

1.  Where does the request enter?
2.  Which component owns the next hop?
3.  What proves the target is healthy?
4.  What happens during deployment or failure?
5.  How do we recover without introducing manual configuration drift?
