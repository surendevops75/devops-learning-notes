# 04 — Service and Ingress Automation with Python

## 1. Overview

Kubernetes Services and Ingress resources provide stable network access to applications running inside a Kubernetes cluster.

In real DevOps environments, Python automation can be used to:

- Create and manage Kubernetes Services
- Expose Deployments and Pods
- Change Service types
- Update ports and selectors
- Create and manage Ingress resources
- Configure host-based and path-based routing
- Automate TLS configuration
- Integrate with AWS Load Balancer Controller
- Validate Service and Ingress health
- Detect configuration drift
- Build self-service deployment automation
- Automate operational workflows

The Python Kubernetes client communicates with the Kubernetes API Server.

Typical flow:

```text
Python Automation
       |
       v
Kubernetes Python Client
       |
       v
Kubernetes API Server
       |
       +--------------------+
       |                    |
       v                    v
   Service              Ingress
       |                    |
       v                    v
  Kubernetes         Ingress Controller
  Endpoints                 |
                            v
                    ALB / NGINX / Gateway
                            |
                            v
                         Users
```

---

# 2. Service Fundamentals

A Pod is ephemeral.

Its IP address can change when the Pod is recreated.

A Service provides a stable virtual endpoint for a group of Pods.

```text
                 Service
              10.100.20.10
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      Pod-1       Pod-2       Pod-3
    10.0.1.10    10.0.2.20    10.0.3.30
```

The Service identifies Pods using labels.

Example:

```yaml
selector:
  app: payment
```

Any Pod with:

```yaml
labels:
  app: payment
```

can become a Service backend.

---

# 3. Why Services Matter in DevOps

Without a Service:

- Pod IPs are unstable
- Clients cannot reliably connect to application instances
- Load balancing becomes difficult
- Service discovery is harder
- Scaling becomes operationally complex

With a Service:

- Stable virtual IP
- DNS-based discovery
- Load balancing across matching Pods
- Decoupling between clients and Pods
- Easier rolling deployments
- Better automation

Example DNS:

```text
payment-service.production.svc.cluster.local
```

Applications can use:

```text
http://payment-service:8080
```

when communicating within the same namespace.

---

# 4. Kubernetes Service Types

The main Service types are:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

## ClusterIP

Default Service type.

Used for internal cluster communication.

```text
Application
     |
     v
ClusterIP Service
     |
     v
Pods
```

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  type: ClusterIP
  selector:
    app: payment
  ports:
    - port: 8080
      targetPort: 8080
```

Use it for:

- Internal microservices
- Database access
- Internal APIs
- Backend services

---

# 5. NodePort

NodePort exposes the Service on a port on each Kubernetes node.

Example:

```yaml
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

Traffic:

```text
Client
  |
  v
NodeIP:30080
  |
  v
Service
  |
  v
Pod:8080
```

NodePort is useful for:

- Development
- Testing
- Simple external exposure

For production AWS EKS architectures, LoadBalancer/Ingress is generally more appropriate.

---

# 6. LoadBalancer

LoadBalancer integrates the Service with an external load balancer supported by the cloud provider.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
```

On AWS, this can result in an AWS load-balancing resource depending on the configured controller and Service annotations.

---

# 7. Service Ports

A Service commonly has:

```text
port
targetPort
nodePort
```

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Meaning:

```text
Client
  |
  | :80
  v
Service :80
  |
  | forwards to
  v
Pod :8080
```

`port` is the Service port.

`targetPort` is the container/backend port.

`nodePort` is used when the Service type is NodePort or a compatible LoadBalancer configuration.

---

# 8. Installing the Python Kubernetes Client

Install the Kubernetes Python client:

```bash
python3 -m pip install kubernetes
```

Verify:

```bash
python3 -c "import kubernetes; print(kubernetes.__version__)"
```

Recommended approach:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install kubernetes
```

Create:

```text
requirements.txt
```

with:

```text
kubernetes
```

Install:

```bash
pip install -r requirements.txt
```

For production, pin a tested version rather than allowing uncontrolled upgrades.

---

# 9. Kubernetes Client Configuration

For local development:

```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CoreV1Api()
```

This normally uses:

```text
~/.kube/config
```

For code running inside a Pod:

```python
from kubernetes import client, config

config.load_incluster_config()

v1 = client.CoreV1Api()
```

For production automation running inside Kubernetes, prefer:

```text
ServiceAccount
+
RBAC
+
in-cluster authentication
```

instead of copying administrator kubeconfig files into containers.

---

# 10. Basic Service Object Structure

A Kubernetes Service consists of:

```text
metadata
spec
```

Example Python object:

```python
service = client.V1Service(
    metadata=client.V1ObjectMeta(
        name="payment-service",
        namespace="production"
    ),
    spec=client.V1ServiceSpec(
        selector={
            "app": "payment"
        },
        ports=[
            client.V1ServicePort(
                port=8080,
                target_port=8080
            )
        ],
        type="ClusterIP"
    )
)
```

---

# 11. Create a ClusterIP Service

Complete example:

```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException

config.load_kube_config()

v1 = client.CoreV1Api()

service = client.V1Service(
    metadata=client.V1ObjectMeta(
        name="payment-service",
        namespace="production",
        labels={
            "app": "payment"
        }
    ),
    spec=client.V1ServiceSpec(
        type="ClusterIP",
        selector={
            "app": "payment"
        },
        ports=[
            client.V1ServicePort(
                name="http",
                protocol="TCP",
                port=8080,
                target_port=8080
            )
        ]
    )
)

try:
    response = v1.create_namespaced_service(
        namespace="production",
        body=service
    )

    print(f"Service created: {response.metadata.name}")

except ApiException as e:
    print(f"Kubernetes API error: {e}")
```

---

# 12. Verify the Service

Using kubectl:

```bash
kubectl get svc -n production
```

Detailed:

```bash
kubectl describe svc payment-service -n production
```

Check endpoints:

```bash
kubectl get endpoints payment-service -n production
```

Modern clusters can also use EndpointSlices:

```bash
kubectl get endpointslice -n production
```

---

# 13. Important Service Troubleshooting Concept

A Service can exist successfully while having no working backend.

Example:

```text
Service
   |
   X
No matching Pods
```

The most common cause is selector mismatch.

Service:

```yaml
selector:
  app: payment
```

Pod:

```yaml
labels:
  app: payments
```

These do not match.

Therefore:

```text
Service -> No endpoints
```

Always verify:

```bash
kubectl get svc -n production
kubectl describe svc payment-service -n production
kubectl get endpoints payment-service -n production
kubectl get endpointslice -n production
kubectl get pods -n production --show-labels
```

---

# 14. Create a NodePort Service

Python:

```python
service = client.V1Service(
    metadata=client.V1ObjectMeta(
        name="frontend-nodeport",
        namespace="production"
    ),
    spec=client.V1ServiceSpec(
        type="NodePort",
        selector={
            "app": "frontend"
        },
        ports=[
            client.V1ServicePort(
                name="http",
                port=80,
                target_port=8080,
                node_port=30080
            )
        ]
    )
)

v1.create_namespaced_service(
    namespace="production",
    body=service
)
```

The NodePort must be within the cluster's configured NodePort range.

The default Kubernetes range is commonly:

```text
30000-32767
```

but cluster configuration can differ.

---

# 15. Create a LoadBalancer Service

```python
service = client.V1Service(
    metadata=client.V1ObjectMeta(
        name="frontend-lb",
        namespace="production"
    ),
    spec=client.V1ServiceSpec(
        type="LoadBalancer",
        selector={
            "app": "frontend"
        },
        ports=[
            client.V1ServicePort(
                name="http",
                port=80,
                target_port=8080
            )
        ]
    )
)

v1.create_namespaced_service(
    namespace="production",
    body=service
)
```

Check:

```bash
kubectl get svc frontend-lb -n production
```

The external address may take time to become available.

---

# 16. Service Annotations

Cloud providers and controllers commonly use annotations to customize behavior.

Example:

```python
metadata=client.V1ObjectMeta(
    name="frontend",
    namespace="production",
    annotations={
        "service.beta.kubernetes.io/aws-load-balancer-type": "external"
    }
)
```

Annotations should be controller-specific.

Do not blindly copy annotations from old clusters because supported annotations and controller behavior can change.

---

# 17. Updating a Service

Read the current object:

```python
service = v1.read_namespaced_service(
    name="payment-service",
    namespace="production"
)
```

Modify:

```python
service.spec.ports[0].target_port = 9090
```

Then replace:

```python
v1.replace_namespaced_service(
    name="payment-service",
    namespace="production",
    body=service
)
```

For production automation, patching only the required fields is often safer than replacing the complete resource.

---

# 18. Patching a Service

Example:

```python
patch = {
    "spec": {
        "ports": [
            {
                "name": "http",
                "port": 80,
                "targetPort": 8080
            }
        ]
    }
}

v1.patch_namespaced_service(
    name="frontend",
    namespace="production",
    body=patch
)
```

Patch operations reduce the chance of accidentally overwriting unrelated fields.

---

# 19. Delete a Service

```python
from kubernetes.client import V1DeleteOptions

v1.delete_namespaced_service(
    name="frontend",
    namespace="production",
    body=V1DeleteOptions()
)
```

Production automation should normally validate:

```text
Environment
Namespace
Resource name
Ownership
Dependency impact
```

before deleting a Service.

---

# 20. Idempotent Service Automation

A production automation script should be safe to run repeatedly.

Bad approach:

```python
v1.create_namespaced_service(...)
```

If the Service already exists, the API returns:

```text
409 Conflict
```

Better pattern:

```text
Check
 |
 +-- Does Service exist?
       |
       +-- No -> Create
       |
       +-- Yes -> Compare desired state
                    |
                    +-- Same -> Do nothing
                    |
                    +-- Different -> Patch/Update
```

Example:

```python
from kubernetes.client.rest import ApiException

def ensure_service(v1, namespace, body):
    name = body.metadata.name

    try:
        existing = v1.read_namespaced_service(
            name=name,
            namespace=namespace
        )

        print(f"Service {name} already exists")

        # Compare desired and current configuration here.
        return existing

    except ApiException as e:
        if e.status == 404:
            return v1.create_namespaced_service(
                namespace=namespace,
                body=body
            )

        raise
```

---

# 21. Why Idempotency Matters

Imagine Jenkins runs:

```text
Deploy pipeline
     |
     v
Python Service automation
```

The pipeline may run multiple times.

If every execution blindly creates resources:

```text
Run 1 -> Create
Run 2 -> Conflict
Run 3 -> Conflict
```

With idempotent logic:

```text
Run 1 -> Create
Run 2 -> Verify
Run 3 -> Verify
```

This makes automation predictable.

---

# 22. Service Discovery

Kubernetes provides DNS-based service discovery.

Example:

```text
payment-service.production.svc.cluster.local
```

Within the same namespace:

```text
payment-service
```

From another namespace:

```text
payment-service.production
```

Full DNS:

```text
payment-service.production.svc.cluster.local
```

Python automation can use these names when generating application configuration.

---

# 23. Headless Services

A headless Service uses:

```yaml
clusterIP: None
```

Python:

```python
spec=client.V1ServiceSpec(
    cluster_ip="None",
    selector={
        "app": "database"
    },
    ports=[
        client.V1ServicePort(
            port=5432,
            target_port=5432
        )
    ]
)
```

Headless Services are useful when applications need direct discovery of individual Pods.

Common examples include:

- Stateful applications
- Database clusters
- Distributed systems

---

# 24. Service Without a Selector

A Service can be created without a selector and connected to manually managed EndpointSlices.

This is useful for advanced cases such as:

```text
Kubernetes Service
       |
       v
External backend
```

However, this should not be used unnecessarily.

For normal microservices, selector-based Services are simpler and safer.

---

# 25. Ingress Fundamentals

A Service provides network access to Pods.

Ingress provides HTTP/HTTPS routing into the cluster.

Example:

```text
Internet
   |
   v
Load Balancer
   |
   v
Ingress Controller
   |
   +------------------+
   |                  |
   v                  v
/api/users        /api/orders
   |                  |
   v                  v
user-service      order-service
```

Ingress can provide:

- Host-based routing
- Path-based routing
- TLS termination
- HTTP/HTTPS routing
- Load balancing
- Controller-specific features

Ingress is an API resource.

It does not route traffic by itself.

An Ingress Controller watches the Ingress resource and configures the actual load-balancing system.

---

# 26. Ingress vs Service

Think of them as different layers.

```text
Ingress
   |
   | HTTP/HTTPS routing
   v
Service
   |
   | Stable backend endpoint
   v
Pods
```

Example:

```text
app.example.com
      |
      v
Ingress
      |
      +---- /users  ---> user-service
      |
      +---- /orders ---> order-service
```

---

# 27. IngressClass

IngressClass identifies which controller should handle an Ingress.

Example:

```yaml
spec:
  ingressClassName: alb
```

For AWS Load Balancer Controller, the class is commonly:

```text
alb
```

For NGINX deployments, it may be:

```text
nginx
```

The actual class name depends on cluster configuration.

Always verify:

```bash
kubectl get ingressclass
```

---

# 28. Kubernetes Python Client and Ingress

Ingress resources are part of the networking API.

Use:

```python
client.NetworkingV1Api()
```

Example:

```python
networking = client.NetworkingV1Api()
```

This API manages:

- Ingress
- IngressClass
- NetworkPolicy

---

# 29. Basic Ingress Object

```python
from kubernetes import client

ingress = client.V1Ingress(
    metadata=client.V1ObjectMeta(
        name="frontend-ingress",
        namespace="production"
    ),
    spec=client.V1IngressSpec(
        ingress_class_name="nginx"
    )
)
```

---

# 30. Ingress Backend Structure

An Ingress rule contains:

```text
Host
 |
 +-- Path
       |
       +-- Service
              |
              +-- Service Port
```

Example:

```text
app.example.com
     |
     +-- /users
     |      |
     |      v
     |  user-service:8080
     |
     +-- /orders
            |
            v
        order-service:8080
```

---

# 31. Create Path-Based Ingress with Python

```python
from kubernetes import client, config

config.load_kube_config()

networking = client.NetworkingV1Api()

user_backend = client.V1IngressBackend(
    service=client.V1IngressServiceBackend(
        name="user-service",
        port=client.V1ServiceBackendPort(
            number=8080
        )
    )
)

order_backend = client.V1IngressBackend(
    service=client.V1IngressServiceBackend(
        name="order-service",
        port=client.V1ServiceBackendPort(
            number=8080
        )
    )
)

user_path = client.V1HTTPIngressPath(
    path="/users",
    path_type="Prefix",
    backend=user_backend
)

order_path = client.V1HTTPIngressPath(
    path="/orders",
    path_type="Prefix",
    backend=order_backend
)

rule = client.V1IngressRule(
    host="app.example.com",
    http=client.V1HTTPIngressRuleValue(
        paths=[
            user_path,
            order_path
        ]
    )
)

ingress = client.V1Ingress(
    metadata=client.V1ObjectMeta(
        name="application-ingress",
        namespace="production"
    ),
    spec=client.V1IngressSpec(
        ingress_class_name="nginx",
        rules=[rule]
    )
)

networking.create_namespaced_ingress(
    namespace="production",
    body=ingress
)

print("Ingress created")
```

---

# 32. Path Types

Kubernetes supports path matching types such as:

```text
Exact
Prefix
ImplementationSpecific
```

Example:

```python
client.V1HTTPIngressPath(
    path="/api",
    path_type="Prefix",
    backend=backend
)
```

`Prefix` is commonly useful for API routing.

---

# 33. Host-Based Routing

Example:

```text
api.example.com
      |
      v
api-service

app.example.com
      |
      v
frontend-service
```

Python:

```python
api_rule = client.V1IngressRule(
    host="api.example.com",
    http=client.V1HTTPIngressRuleValue(
        paths=[
            client.V1HTTPIngressPath(
                path="/",
                path_type="Prefix",
                backend=client.V1IngressBackend(
                    service=client.V1IngressServiceBackend(
                        name="api-service",
                        port=client.V1ServiceBackendPort(
                            number=8080
                        )
                    )
                )
            )
        ]
    )
)
```

---

# 34. Multiple Ingress Rules

```python
rules = [
    client.V1IngressRule(
        host="api.example.com",
        http=client.V1HTTPIngressRuleValue(
            paths=[
                client.V1HTTPIngressPath(
                    path="/",
                    path_type="Prefix",
                    backend=api_backend
                )
            ]
        )
    ),
    client.V1IngressRule(
        host="app.example.com",
        http=client.V1HTTPIngressRuleValue(
            paths=[
                client.V1HTTPIngressPath(
                    path="/",
                    path_type="Prefix",
                    backend=frontend_backend
                )
            ]
        )
    )
]
```

Then:

```python
spec = client.V1IngressSpec(
    ingress_class_name="alb",
    rules=rules
)
```

---

# 35. TLS Configuration

Ingress can terminate HTTPS.

Example:

```python
tls = client.V1IngressTLS(
    hosts=["app.example.com"],
    secret_name="app-tls"
)
```

Then:

```python
spec = client.V1IngressSpec(
    ingress_class_name="nginx",
    tls=[tls],
    rules=[rule]
)
```

Traffic:

```text
Client
  |
 HTTPS
  |
  v
Ingress Controller
  |
 TLS termination
  |
  v
Service
  |
  v
Pods
```

---

# 36. AWS EKS Architecture with ALB Ingress

In an EKS environment using AWS Load Balancer Controller:

```text
                         Internet
                            |
                            v
                     Route 53 DNS
                            |
                            v
                     AWS ALB
                            |
                            v
              AWS Load Balancer Controller
                            |
                            v
                       Kubernetes
                         Ingress
                            |
              +-------------+-------------+
              |                           |
              v                           v
       frontend-service             api-service
              |                           |
              v                           v
          Frontend Pods                API Pods
```

The controller watches Kubernetes resources and configures AWS load-balancing resources.

---

# 37. ALB Ingress Configuration

A typical ALB Ingress may contain annotations such as:

```python
annotations = {
    "alb.ingress.kubernetes.io/scheme": "internet-facing",
    "alb.ingress.kubernetes.io/target-type": "ip",
    "alb.ingress.kubernetes.io/listen-ports": '[{"HTTP":80}]'
}
```

Python:

```python
metadata = client.V1ObjectMeta(
    name="frontend-alb",
    namespace="production",
    annotations=annotations
)
```

For HTTPS, an ACM certificate can be referenced through the appropriate AWS Load Balancer Controller annotation.

Example:

```python
annotations = {
    "alb.ingress.kubernetes.io/scheme": "internet-facing",
    "alb.ingress.kubernetes.io/target-type": "ip",
    "alb.ingress.kubernetes.io/listen-ports": '[{"HTTP":80},{"HTTPS":443}]',
    "alb.ingress.kubernetes.io/certificate-arn": "<ACM-CERTIFICATE-ARN>",
    "alb.ingress.kubernetes.io/ssl-redirect": "443"
}
```

Use the controller version and AWS documentation applicable to the cluster before standardizing annotations.

---

# 38. Why target-type IP Is Important in EKS

With:

```text
target-type: ip
```

the ALB can route directly to Pod IPs.

Architecture:

```text
ALB
 |
 +---- Pod IP
 +---- Pod IP
 +---- Pod IP
```

This is particularly useful with AWS Load Balancer Controller and EKS workloads.

The exact networking requirements depend on the EKS/VPC configuration.

---

# 39. Python Automation for ALB Ingress

Example:

```python
from kubernetes import client, config

config.load_kube_config()

networking = client.NetworkingV1Api()

annotations = {
    "alb.ingress.kubernetes.io/scheme": "internet-facing",
    "alb.ingress.kubernetes.io/target-type": "ip",
    "alb.ingress.kubernetes.io/listen-ports": '[{"HTTP":80}]'
}

backend = client.V1IngressBackend(
    service=client.V1IngressServiceBackend(
        name="frontend-service",
        port=client.V1ServiceBackendPort(
            number=80
        )
    )
)

path = client.V1HTTPIngressPath(
    path="/",
    path_type="Prefix",
    backend=backend
)

rule = client.V1IngressRule(
    host="app.example.com",
    http=client.V1HTTPIngressRuleValue(
        paths=[path]
    )
)

ingress = client.V1Ingress(
    metadata=client.V1ObjectMeta(
        name="frontend-alb",
        namespace="production",
        annotations=annotations
    ),
    spec=client.V1IngressSpec(
        ingress_class_name="alb",
        rules=[rule]
    )
)

networking.create_namespaced_ingress(
    namespace="production",
    body=ingress
)
```

---

# 40. Ingress Status

Creating an Ingress does not mean the external endpoint is immediately ready.

Check:

```bash
kubectl get ingress -n production
```

Detailed:

```bash
kubectl describe ingress frontend-alb -n production
```

Python:

```python
ingress = networking.read_namespaced_ingress(
    name="frontend-alb",
    namespace="production"
)

print(ingress.status)
```

Depending on the controller, the status may eventually contain an address or hostname.

---

# 41. Wait for ALB/Ingress Readiness

Production automation should not assume immediate provisioning.

A better workflow:

```text
Create Ingress
      |
      v
Wait
      |
      v
Read Ingress status
      |
      +-- Address exists? -- No --> Retry
      |
     Yes
      |
      v
Validate endpoint
```

Example:

```python
import time

def wait_for_ingress_address(networking, name, namespace, timeout=600):
    start = time.time()

    while time.time() - start < timeout:
        ingress = networking.read_namespaced_ingress(
            name=name,
            namespace=namespace
        )

        status = ingress.status

        if (
            status
            and status.load_balancer
            and status.load_balancer.ingress
        ):
            entry = status.load_balancer.ingress[0]

            address = entry.hostname or entry.ip

            if address:
                return address

        print("Waiting for Ingress address...")
        time.sleep(10)

    raise TimeoutError(
        f"Ingress {name} did not receive an address"
    )
```

This is a useful production automation pattern.

---

# 42. Service and Ingress Automation Workflow

A realistic deployment automation can follow:

```text
Application Deployment
        |
        v
Check Deployment readiness
        |
        v
Create/Update Service
        |
        v
Validate Service endpoints
        |
        v
Create/Update Ingress
        |
        v
Wait for Ingress status
        |
        v
Get external address
        |
        v
DNS / Application validation
        |
        v
Health check
```

---

# 43. Complete Service Automation Script

```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException


NAMESPACE = "production"
SERVICE_NAME = "frontend-service"


def build_service():
    return client.V1Service(
        metadata=client.V1ObjectMeta(
            name=SERVICE_NAME,
            namespace=NAMESPACE,
            labels={
                "app": "frontend"
            }
        ),
        spec=client.V1ServiceSpec(
            type="ClusterIP",
            selector={
                "app": "frontend"
            },
            ports=[
                client.V1ServicePort(
                    name="http",
                    protocol="TCP",
                    port=80,
                    target_port=8080
                )
            ]
        )
    )


def ensure_service(v1):
    desired = build_service()

    try:
        existing = v1.read_namespaced_service(
            name=SERVICE_NAME,
            namespace=NAMESPACE
        )

        print(
            f"Service already exists: "
            f"{existing.metadata.name}"
        )

        return existing

    except ApiException as e:
        if e.status == 404:
            created = v1.create_namespaced_service(
                namespace=NAMESPACE,
                body=desired
            )

            print(
                f"Service created: "
                f"{created.metadata.name}"
            )

            return created

        raise


def main():
    config.load_kube_config()

    v1 = client.CoreV1Api()

    ensure_service(v1)


if __name__ == "__main__":
    main()
```

---

# 44. Complete Ingress Automation Script

```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException


NAMESPACE = "production"
INGRESS_NAME = "frontend-ingress"


def build_ingress():
    backend = client.V1IngressBackend(
        service=client.V1IngressServiceBackend(
            name="frontend-service",
            port=client.V1ServiceBackendPort(
                number=80
            )
        )
    )

    path = client.V1HTTPIngressPath(
        path="/",
        path_type="Prefix",
        backend=backend
    )

    rule = client.V1IngressRule(
        host="app.example.com",
        http=client.V1HTTPIngressRuleValue(
            paths=[path]
        )
    )

    return client.V1Ingress(
        metadata=client.V1ObjectMeta(
            name=INGRESS_NAME,
            namespace=NAMESPACE
        ),
        spec=client.V1IngressSpec(
            ingress_class_name="alb",
            rules=[rule]
        )
    )


def ensure_ingress(networking):
    desired = build_ingress()

    try:
        existing = networking.read_namespaced_ingress(
            name=INGRESS_NAME,
            namespace=NAMESPACE
        )

        print(
            f"Ingress already exists: "
            f"{existing.metadata.name}"
        )

        return existing

    except ApiException as e:
        if e.status == 404:
            created = networking.create_namespaced_ingress(
                namespace=NAMESPACE,
                body=desired
            )

            print(
                f"Ingress created: "
                f"{created.metadata.name}"
            )

            return created

        raise


def main():
    config.load_kube_config()

    networking = client.NetworkingV1Api()

    ensure_ingress(networking)


if __name__ == "__main__":
    main()
```

---

# 45. Production Design: Separate Desired State from Code

Avoid hardcoding every value directly into Python.

Instead:

```text
Python automation
      |
      +-- configuration
      |
      +-- templates
      |
      +-- Kubernetes API
```

Example:

```python
APP_NAME = "frontend"
NAMESPACE = "production"
SERVICE_PORT = 80
TARGET_PORT = 8080
HOST = "app.example.com"
INGRESS_CLASS = "alb"
```

For larger systems, configuration can come from:

- YAML
- JSON
- Environment variables
- CI/CD parameters
- Secret managers
- Git repositories

Do not put passwords or private keys directly in source code.

---

# 46. Environment-Based Configuration

Example:

```python
import os

namespace = os.getenv(
    "K8S_NAMESPACE",
    "default"
)

environment = os.getenv(
    "ENVIRONMENT",
    "dev"
)

ingress_class = os.getenv(
    "INGRESS_CLASS",
    "alb"
)
```

This allows the same automation to run across:

```text
dev
staging
production
```

without modifying the code.

---

# 47. Production Architecture

A mature Python Kubernetes automation platform can look like:

```text
                  Git Repository
                       |
                       v
                  CI/CD Pipeline
                       |
                       v
              Python Automation
                       |
              +--------+--------+
              |                 |
              v                 v
       Kubernetes API       AWS APIs
              |                 |
              v                 v
        EKS Resources       AWS Resources
              |
       +------+------+
       |             |
       v             v
    Service       Ingress
       |             |
       v             v
      Pods           ALB
```

In a GitOps environment, Python should be used carefully.

If ArgoCD is the source of truth for Kubernetes manifests, imperative Python changes can create drift.

A better architecture may be:

```text
Python
  |
  v
Generate/update Git configuration
  |
  v
Git
  |
  v
ArgoCD
  |
  v
Kubernetes
```

This preserves GitOps principles.

---

# 48. Python + Jenkins + Kubernetes

A CI/CD pipeline can execute Python automation:

```text
Developer
   |
   v
Git
   |
   v
Jenkins
   |
   +-- Build
   |
   +-- Test
   |
   +-- Security Scan
   |
   +-- Build Image
   |
   +-- Push Image
   |
   v
Python Kubernetes Automation
   |
   v
Kubernetes
```

However, if ArgoCD manages deployment, Jenkins should generally update the Git desired state rather than directly modifying live Kubernetes resources.

---

# 49. Python + ArgoCD

GitOps-friendly approach:

```text
Developer
   |
   v
Git
   |
   v
Jenkins / GitHub Actions
   |
   +-- Build image
   +-- Security scan
   |
   v
Update image tag / manifest
   |
   v
Git repository
   |
   v
ArgoCD
   |
   v
EKS
```

Python can help automate:

- Manifest generation
- Environment-specific configuration
- Image tag updates
- Validation
- Repository changes
- Pre-deployment checks

This avoids creating unmanaged Kubernetes resources.

---

# 50. Service Troubleshooting with Python

Python can automate health checks.

Example:

```python
def check_service(v1, name, namespace):
    service = v1.read_namespaced_service(
        name=name,
        namespace=namespace
    )

    print("Service:", service.metadata.name)
    print("Type:", service.spec.type)
    print("Cluster IP:", service.spec.cluster_ip)
    print("Selector:", service.spec.selector)

    return service
```

Check endpoints:

```python
def check_endpoints(v1, service_name, namespace):
    endpoints = v1.read_namespaced_endpoints(
        name=service_name,
        namespace=namespace
    )

    if not endpoints.subsets:
        print("WARNING: Service has no endpoints")
        return False

    for subset in endpoints.subsets:
        addresses = subset.addresses or []

        print(
            f"Ready endpoints: {len(addresses)}"
        )

    return True
```

---

# 51. Common Service Problems

## Problem 1: No endpoints

Check:

```bash
kubectl get endpoints <service> -n <namespace>
```

Likely causes:

- Selector mismatch
- Pods not Ready
- Wrong namespace
- Pod labels incorrect

---

## Problem 2: Connection refused

Check:

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get pods
```

Possible causes:

- Wrong targetPort
- Application not listening on expected port
- Container port mismatch
- NetworkPolicy
- Application failure

---

## Problem 3: Service works internally but not externally

Check:

```bash
kubectl get svc
kubectl describe svc
```

For LoadBalancer:

- Cloud controller
- Security groups
- Subnets
- Target health
- Controller logs

---

# 52. Ingress Troubleshooting

Start with:

```bash
kubectl get ingress -n production
```

Then:

```bash
kubectl describe ingress frontend-ingress -n production
```

Check:

```bash
kubectl get ingressclass
```

Then verify:

```bash
kubectl get svc -n production
kubectl get endpoints -n production
kubectl get pods -n production
```

For AWS Load Balancer Controller:

```bash
kubectl get pods -n kube-system
```

Find the controller:

```bash
kubectl get deployment -n kube-system
```

Check logs:

```bash
kubectl logs -n kube-system \
  deployment/aws-load-balancer-controller
```

The deployment name can differ by installation.

---

# 53. Ingress Troubleshooting Decision Tree

```text
Ingress not working
       |
       v
Does Ingress exist?
       |
      No
       |
   Create/fix it
       |
      Yes
       |
       v
Correct IngressClass?
       |
      No
       |
   Fix class
       |
      Yes
       |
       v
Does controller see it?
       |
      No
       |
 Check controller
       |
      Yes
       |
       v
External address exists?
       |
      No
       |
 Check controller events/logs
       |
      Yes
       |
       v
Does Service have endpoints?
       |
      No
       |
 Fix selector/readiness
       |
      Yes
       |
       v
Does backend respond?
       |
      No
       |
 Check targetPort/application
       |
      Yes
       |
       v
Check DNS/TLS/security/network
```

---

# 54. Common Ingress Problems

### Wrong IngressClass

Ingress:

```yaml
ingressClassName: nginx
```

but cluster uses:

```text
alb
```

Result:

```text
Ingress not processed by expected controller
```

---

### Service does not exist

Ingress points to:

```text
frontend-service
```

but Service is named:

```text
frontend
```

Result:

```text
Backend unavailable
```

---

### Service has no endpoints

Ingress is correct but:

```text
Service -> No Pods
```

Fix:

```text
labels
selector
readiness
```

---

### Wrong service port

Ingress:

```text
service port: 80
```

but Service exposes:

```text
443
```

Result:

```text
Routing failure
```

---

### DNS issue

ALB exists, but:

```text
app.example.com
```

does not resolve correctly.

Check:

```bash
dig app.example.com
nslookup app.example.com
```

---

# 55. API Exception Handling

Production automation must distinguish common Kubernetes API errors.

```python
try:
    service = v1.read_namespaced_service(
        name="frontend",
        namespace="production"
    )

except ApiException as e:
    if e.status == 404:
        print("Service does not exist")

    elif e.status == 403:
        print("RBAC permission denied")

    elif e.status == 409:
        print("Resource conflict")

    else:
        print(
            f"Kubernetes API error: "
            f"{e.status} {e.reason}"
        )
```

Important statuses:

```text
404 -> Not Found
403 -> Forbidden
409 -> Conflict
429 -> Too Many Requests
500 -> Internal Server Error
```

Do not blindly retry every error.

---

# 56. Retry Strategy

Transient failures can occur because:

- API server temporarily unavailable
- Network interruption
- Controller reconciliation delay
- Cloud resource provisioning delay

Use controlled retries.

Example:

```python
import time

def retry_operation(operation, retries=5, delay=5):
    for attempt in range(1, retries + 1):
        try:
            return operation()

        except Exception as e:
            if attempt == retries:
                raise

            print(
                f"Attempt {attempt} failed: {e}"
            )

            time.sleep(delay)
```

Production systems should use bounded retries and exponential backoff where appropriate.

---

# 57. RBAC for Service and Ingress Automation

Do not give automation:

```text
cluster-admin
```

unless absolutely necessary.

Example ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: service-ingress-automation
  namespace: automation
```

Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: service-ingress-manager
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["services", "endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
```

Bind the Role to the ServiceAccount.

Use the minimum permissions required.

---

# 58. Security Best Practices

Never hardcode:

```text
AWS credentials
Kubernetes tokens
Private keys
Passwords
ACM private material
```

Use:

- IAM roles
- EKS Pod Identity / IRSA where applicable
- Kubernetes ServiceAccounts
- RBAC
- Secrets
- External secret management
- Environment-specific configuration

Validate user-supplied values before creating Kubernetes resources.

---

# 59. Production Validation

Before creating a Service:

```text
Namespace exists?
Selector valid?
Target Pods exist?
Port valid?
Service type allowed?
```

Before creating an Ingress:

```text
IngressClass exists?
Backend Service exists?
Backend port exists?
Host valid?
TLS Secret exists if required?
Controller installed?
```

This prevents many avoidable deployment failures.

---

# 60. Observability for Automation

Python automation should produce useful logs.

Bad:

```python
print("Failed")
```

Better:

```python
print(
    "Failed to create Ingress "
    "name=frontend-ingress "
    "namespace=production"
)
```

Production systems should preferably use Python logging:

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(__name__)

logger.info(
    "Creating Service: %s",
    SERVICE_NAME
)
```

Useful fields include:

```text
environment
namespace
resource
resource type
operation
status
error
duration
```

---

# 61. Real-World DevOps Use Case

Suppose a team deploys a new microservice:

```text
inventory
```

The automation needs to:

```text
1. Verify Deployment
2. Create inventory-service
3. Validate endpoints
4. Create/update Ingress
5. Configure /inventory route
6. Wait for ALB
7. Validate DNS
8. Perform HTTP health check
9. Report deployment status
```

Python can orchestrate these checks.

---

# 62. Example Production Workflow

```text
New Application
      |
      v
Deployment created
      |
      v
Pods Ready?
   |       |
  No      Yes
   |       |
 Troubleshoot
           |
           v
   Ensure Service
           |
           v
   Endpoints Ready?
           |
           v
   Ensure Ingress
           |
           v
   Controller Reconciles
           |
           v
   ALB Address Available
           |
           v
   DNS Validation
           |
           v
   HTTP Health Check
           |
           v
       Success
```

---

# 63. Service and Ingress Health Validation

Example:

```python
import requests

def validate_http(url):
    try:
        response = requests.get(
            url,
            timeout=10
        )

        print(
            f"HTTP status: {response.status_code}"
        )

        return response.ok

    except requests.RequestException as e:
        print(f"HTTP validation failed: {e}")
        return False
```

Install:

```bash
pip install requests
```

For production, pin the dependency.

---

# 64. End-to-End Automation Architecture

```text
                 Developer
                     |
                     v
                  Git Push
                     |
                     v
             CI/CD Pipeline
                     |
          +----------+----------+
          |                     |
          v                     v
      Security              Build Image
       Scans                    |
          |                     v
          +----------> Image Registry
                              |
                              v
                        Deployment
                              |
                              v
                     Python Validation
                              |
                +-------------+-------------+
                |                           |
                v                           v
             Service                    Ingress
                |                           |
                v                           v
             Endpoints                    ALB
                |                           |
                +-------------+-------------+
                              |
                              v
                        Health Check
```

---

# 65. Best Practices

1. Prefer idempotent automation.
2. Use `NetworkingV1Api` for Ingress.
3. Use `CoreV1Api` for Services.
4. Validate selectors carefully.
5. Validate Service endpoints after creation.
6. Do not assume cloud load balancers are immediately ready.
7. Poll asynchronous resources with bounded timeouts.
8. Use RBAC least privilege.
9. Keep configuration outside application logic.
10. Never hardcode credentials.
11. Log resource names and namespaces.
12. Handle API exceptions explicitly.
13. Use retries only for transient failures.
14. Avoid blind full-resource replacement.
15. Prefer patches for small changes.
16. Keep GitOps environments declarative.
17. Avoid modifying ArgoCD-managed resources directly unless that is intentionally part of the design.
18. Validate IngressClass.
19. Validate backend Service ports.
20. Validate DNS and TLS separately from Kubernetes resource creation.
21. Use health checks after provisioning.
22. Keep production automation observable.
23. Test scripts against non-production clusters first.
24. Pin Python dependencies.
25. Review Kubernetes API/client compatibility during upgrades.

---

# 66. Common Mistakes

### Mistake 1

Using the wrong API:

```python
client.CoreV1Api()
```

for Ingress.

Correct:

```python
client.NetworkingV1Api()
```

---

### Mistake 2

Assuming Service creation means backend is healthy.

Always check:

```bash
kubectl get endpoints
```

---

### Mistake 3

Hardcoding `nodePort` values unnecessarily.

Prefer automatic allocation unless a fixed port is required.

---

### Mistake 4

Using the wrong `targetPort`.

Service:

```text
port: 80
targetPort: 8080
```

The application must actually listen on the expected backend port.

---

### Mistake 5

Creating an Ingress without checking the controller.

An Ingress resource alone does not provide routing.

---

### Mistake 6

Giving the Python automation `cluster-admin`.

Use namespace-scoped RBAC where possible.

---

### Mistake 7

Blindly creating resources.

Use ensure/create/update logic.

---

### Mistake 8

Ignoring asynchronous provisioning.

AWS ALBs can take time to become ready.

---

### Mistake 9

Creating direct Kubernetes changes in a GitOps-managed cluster without considering drift.

If ArgoCD is the source of truth, prefer changing Git.

---

# 67. Interview Questions

## Q1. Why do we need a Kubernetes Service?

A Service provides a stable network endpoint for a dynamic set of Pods and distributes traffic across matching backend Pods.

---

## Q2. What is the difference between `port` and `targetPort`?

`port` is the port exposed by the Service.

`targetPort` is the port on the backend Pod/container receiving the traffic.

---

## Q3. How does a Service find Pods?

It uses label selectors.

Example:

```yaml
selector:
  app: payment
```

Pods must have matching labels.

---

## Q4. How do you troubleshoot a Service with no endpoints?

I would check:

```bash
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslice
kubectl get pods --show-labels
```

Then I would compare the Service selector with Pod labels and verify that Pods are Ready.

---

## Q5. Which Python API manages Services?

```python
client.CoreV1Api()
```

---

## Q6. Which Python API manages Ingress?

```python
client.NetworkingV1Api()
```

---

## Q7. Does creating an Ingress automatically create routing?

No.

An Ingress Controller must watch the Ingress resource and implement the routing.

---

## Q8. What is IngressClass?

IngressClass identifies the controller intended to process an Ingress resource.

---

## Q9. How would you automate an ALB Ingress in EKS?

I would use the Kubernetes Python client to create/update the Ingress, configure the appropriate `alb` IngressClass and controller-specific annotations, then wait for the Ingress status to receive the load-balancer address and validate the endpoint.

---

## Q10. How do you make Python Kubernetes automation idempotent?

I first read the resource.

If it does not exist, I create it.

If it exists, I compare the desired state with the current state and patch only the required differences.

---

# 68. Scenario-Based Interview Questions

## Scenario 1

### Interviewer

You created a Service successfully, but requests are failing. What do you check?

### Strong Answer

I would first verify the Service:

```bash
kubectl describe svc <service>
```

Then check endpoints:

```bash
kubectl get endpoints <service>
```

If there are no endpoints, I would compare the Service selector with Pod labels and check Pod readiness.

If endpoints exist, I would verify the Service `targetPort`, application listening port, NetworkPolicies, and application logs.

---

## Scenario 2

### Interviewer

Your Ingress exists but no ALB is created.

### Strong Answer

I would verify the IngressClass and confirm that the AWS Load Balancer Controller is installed and healthy.

Then I would inspect:

```bash
kubectl describe ingress <name>
```

and controller logs.

I would also validate:

- IAM permissions
- Subnet discovery
- Security groups
- Ingress annotations
- Controller events
- EKS/VPC configuration

I would not assume that an Ingress object automatically creates an ALB.

---

## Scenario 3

### Interviewer

Your ALB exists, but requests return 503.

### Strong Answer

I would determine whether the ALB target group has healthy targets.

Then I would trace:

```text
ALB
 -> target
 -> Service
 -> Pod
```

I would verify:

- Service selector
- Endpoints
- Pod readiness
- TargetPort
- Application listening port
- Security groups
- Health check configuration

A 503 often indicates that the load balancer cannot reach a healthy backend.

---

## Scenario 4

### Interviewer

Your Python script works once but fails when executed again.

### Strong Answer

The script is probably not idempotent.

If it blindly calls `create`, the second execution can receive HTTP 409 Conflict.

I would implement an ensure pattern:

```text
Read
 |
 +-- 404 -> Create
 |
 +-- Exists -> Compare/Patch
```

This makes repeated CI/CD executions safe.

---

## Scenario 5

### Interviewer

ArgoCD manages the application, but your Python script modifies the Service directly.

What problem can occur?

### Strong Answer

The direct change can create GitOps drift.

ArgoCD may detect the difference and reconcile the Service back to the Git-defined state.

If ArgoCD is the source of truth, I would prefer updating the desired configuration in Git and allowing ArgoCD to reconcile it.

---

# 69. Production Checklist

Before deploying Service/Ingress automation:

```text
[ ] Kubernetes client version tested
[ ] Authentication configured
[ ] Namespace validated
[ ] RBAC configured
[ ] Service selector validated
[ ] Service ports validated
[ ] targetPort validated
[ ] IngressClass validated
[ ] Backend Services exist
[ ] Backend endpoints validated
[ ] TLS configuration validated
[ ] Controller health checked
[ ] AWS permissions validated
[ ] Subnet/network requirements validated
[ ] Idempotency implemented
[ ] Retry strategy implemented
[ ] Timeout implemented
[ ] Logging implemented
[ ] Health checks implemented
[ ] GitOps ownership understood
[ ] Secrets protected
[ ] Non-production testing completed
```

---

# 70. Key Takeaways

The important mental model is:

```text
Pod
 |
 | dynamic
 v
Service
 |
 | stable endpoint
 v
Ingress
 |
 | HTTP/HTTPS routing
 v
Load Balancer
 |
 v
Users
```

For Python:

```text
CoreV1Api
   |
   +-- Services

NetworkingV1Api
   |
   +-- Ingress
   +-- IngressClass
   +-- NetworkPolicy
```

For production automation:

```text
Desired State
     |
     v
Idempotent Python
     |
     v
Kubernetes API
     |
     v
Controller Reconciliation
     |
     v
Infrastructure
     |
     v
Health Validation
```

The key DevOps principle is not simply:

> "Use Python to create Kubernetes resources."

It is:

> **Use Python to build reliable, repeatable, observable, secure, and idempotent automation around Kubernetes APIs—while respecting GitOps ownership when ArgoCD is the source of truth.**
