# Section 5 - Service Troubleshooting Scenarios

---

## Scenario 51

### Interview Question

Your Pods are running successfully, but the application is not accessible through the Kubernetes Service. How would you troubleshoot it?

### Production-Level Answer

If the Pods are healthy but the Service is not working, the issue usually lies in the Service configuration, selectors, endpoints, or network routing.

Always verify whether the Service is forwarding traffic to the correct Pods.

### Approach

Check Services.

```bash
kubectl get svc
```

Describe the Service.

```bash
kubectl describe svc <service-name>
```

Verify Endpoints.

```bash
kubectl get endpoints <service-name>
```

Check

- Service selector
- Pod labels
- TargetPort
- Container Port
- Pod readiness

### Common Root Causes

- Selector mismatch
- Wrong TargetPort
- Pods not Ready
- Incorrect labels
- Endpoint not created

### Best Practices

- Standardize labels
- Verify selectors before deployment
- Test Service connectivity
- Monitor endpoint health

---

## Scenario 52

### Interview Question

A Service has **no Endpoints**. How would you investigate?

### Production-Level Answer

A Service without Endpoints has no Pods available to receive traffic.

This is almost always caused by a selector mismatch or unhealthy Pods.

### Approach

Check Endpoints.

```bash
kubectl get endpoints
```

Check Service selector.

```bash
kubectl describe svc <service-name>
```

Verify Pod labels.

```bash
kubectl get pods --show-labels
```

Compare

- Service selector
- Pod labels

### Common Root Causes

- Wrong selector
- Label typo
- Pods failed readiness
- Pods in different namespace

### Best Practices

- Keep labels consistent
- Use label standards
- Validate selectors during CI
- Monitor endpoint count

---

## Scenario 53

### Interview Question

Your Service is exposing the wrong port. Users receive connection failures. How would you troubleshoot it?

### Production-Level Answer

A Service must forward traffic from the Service Port to the correct TargetPort inside the container.

Incorrect port mappings prevent application connectivity.

### Approach

Review Service.

```bash
kubectl describe svc <service-name>
```

Verify

- Port
- TargetPort
- Container Port
- Application listening port

Check Deployment.

```bash
kubectl describe deployment <deployment-name>
```

### Common Root Causes

- Wrong TargetPort
- Wrong container port
- Application listening on different port
- Manifest error

### Best Practices

- Keep port naming consistent
- Test connectivity after deployment
- Validate manifests before release

---

## Scenario 54

### Interview Question

A ClusterIP Service is unreachable from another Pod in the same namespace. How would you investigate?

### Production-Level Answer

ClusterIP Services should be accessible from any Pod within the cluster.

If connectivity fails, investigate DNS, Endpoints, and Network Policies.

### Approach

Verify Service.

```bash
kubectl get svc
```

Test DNS.

```bash
nslookup <service-name>
```

Check Endpoints.

```bash
kubectl get endpoints
```

Verify

- Network Policies
- CoreDNS
- Service selector

### Common Root Causes

- DNS failure
- Endpoint missing
- Network Policy blocking traffic
- Selector mismatch

### Best Practices

- Monitor CoreDNS
- Validate Service discovery
- Test internal connectivity regularly

---

## Scenario 55

### Interview Question

A NodePort Service works from one Worker Node but not another. How would you troubleshoot it?

### Production-Level Answer

NodePort should expose the application on every Worker Node.

If only some nodes work, investigate kube-proxy, networking, and firewall rules.

### Approach

Check

- kube-proxy
- Node health
- Firewall
- Security Groups
- Node routing

Verify Service.

```bash
kubectl describe svc <service-name>
```

### Common Root Causes

- kube-proxy failure
- Firewall rule
- Node issue
- Network routing problem
- Security Group restriction

### Best Practices

- Monitor kube-proxy
- Standardize firewall rules
- Validate NodePort connectivity
- Prefer LoadBalancer in production

---

## Scenario 56

### Interview Question

Your LoadBalancer Service remains in the **Pending** state. What would you investigate?

### Production-Level Answer

A LoadBalancer Service depends on the cloud provider.

If it remains Pending, Kubernetes cannot provision the external load balancer.

### Approach

Check Service.

```bash
kubectl describe svc <service-name>
```

Review Events.

Verify

- Cloud Controller Manager
- IAM permissions
- Cloud provider integration
- Subnet configuration

### Common Root Causes

- Cloud integration issue
- Missing IAM permissions
- Incorrect subnet tags
- Cloud API failure

### Best Practices

- Monitor cloud controllers
- Validate IAM permissions
- Tag subnets correctly
- Review Service Events

---

## Scenario 57

### Interview Question

Users report intermittent connectivity to a Service. Some requests succeed while others fail. How would you investigate?

### Production-Level Answer

Intermittent failures usually indicate that some backend Pods are unhealthy while others continue serving requests.

### Approach

Verify

- Endpoints
- Ready Pods
- Service selector
- Application logs
- Readiness Probes

Check Endpoint list.

```bash
kubectl get endpoints <service-name>
```

### Common Root Causes

- One unhealthy Pod
- Readiness failure
- Pod restart loop
- Node issue
- Application crash

### Best Practices

- Enable readiness probes
- Monitor endpoint health
- Remove unhealthy Pods quickly
- Configure rolling updates

---

## Scenario 58

### Interview Question

Your application cannot resolve another Service using its DNS name. How would you troubleshoot it?

### Production-Level Answer

Kubernetes relies on CoreDNS for internal Service discovery.

If DNS resolution fails, investigate CoreDNS before the application.

### Approach

Test DNS.

```bash
nslookup <service-name>
```

Check CoreDNS.

```bash
kubectl get pods -n kube-system
```

Review CoreDNS logs.

```bash
kubectl logs -n kube-system <coredns-pod>
```

### Common Root Causes

- CoreDNS failure
- Wrong Service name
- Wrong namespace
- Network Policy
- DNS configuration issue

### Best Practices

- Monitor CoreDNS
- Use FQDN when required
- Validate DNS after deployment

---

## Scenario 59

### Interview Question

Two applications in different namespaces cannot communicate using Service names. How would you investigate?

### Production-Level Answer

A Service name alone only resolves within the same namespace.

Cross-namespace communication requires the fully qualified DNS name.

### Approach

Verify

- Namespace
- Service name
- DNS resolution
- Network Policies

Test

```bash
nslookup <service>.<namespace>.svc.cluster.local
```

### Common Root Causes

- Wrong namespace
- Incorrect DNS name
- Network Policy restriction
- Service missing

### Best Practices

- Use FQDN for cross-namespace communication
- Document namespace architecture
- Validate service discovery

---

## Scenario 60

### Interview Question

A Service is correctly configured, Endpoints exist, Pods are healthy, but users still cannot access the application. How would you troubleshoot it?

### Production-Level Answer

When the Service appears healthy, investigate the complete request path from the client to the application.

The issue may exist outside Kubernetes itself.

### Approach

Verify

- Client connectivity
- DNS
- Service
- Endpoints
- Pod readiness
- Application logs
- Ingress
- Load Balancer
- Firewall
- Security Groups

Test connectivity step by step until the failure point is identified.

### Common Root Causes

- External Load Balancer issue
- Ingress misconfiguration
- Firewall restriction
- Security Group blocking traffic
- Application listening on wrong port
- External DNS problem

### Best Practices

- Troubleshoot layer by layer
- Monitor Service latency
- Validate network path after deployment
- Use observability tools for traffic analysis

---

