# 16-Networking-for-DevOps
# 18-Nginx

## 1. Purpose

Nginx is one of the most important reverse-proxy and web-serving technologies for DevOps engineers. It is used in traditional VM environments, container platforms, Kubernetes, edge architectures, internal platforms, and production application stacks.

This file covers Nginx from fundamentals through production implementation.

Topics include:

- Nginx fundamentals
- architecture
- event-driven model
- master and worker processes
- installation
- package management
- configuration hierarchy
- nginx.conf
- contexts
- directives
- server blocks
- locations
- reverse proxy
- upstreams
- load balancing
- health checks
- proxy headers
- TLS
- certificates
- HTTP to HTTPS redirects
- HTTP/2 concepts
- WebSockets
- timeouts
- buffering
- connection handling
- keep-alive
- caching
- compression
- static files
- rate limiting
- request limits
- security headers
- access control
- authentication concepts
- logging
- log rotation
- metrics
- Prometheus
- Grafana
- ELK
- systemd
- Nginx on EC2
- Nginx in Docker
- Nginx in Kubernetes
- Nginx Ingress Controller
- EKS
- ALB vs Nginx
- Nginx vs Envoy
- production architecture
- GitOps
- RoboShop
- troubleshooting
- interview preparation

---

## 2. What Is Nginx?

Nginx is a high-performance web server and reverse proxy.

Common DevOps uses:

```text
web server
reverse proxy
load balancer
TLS termination
static file server
HTTP gateway
cache
```

---

## 3. Why Nginx Is Important

Nginx is useful because it can efficiently handle large numbers of concurrent connections while providing flexible HTTP routing and proxy controls.

---

## 4. Nginx Architecture

A simplified architecture is:

```text
                Nginx
                  |
        +---------+---------+
        |                   |
      Master             Workers
        |                   |
   configuration       event loop
                         |
                    connections
```

---

## 5. Master Process

The master process generally handles:

```text
configuration
worker lifecycle
signals
graceful reload
```

---

## 6. Worker Processes

Workers handle network connections and requests.

The event-driven model allows efficient handling of many concurrent connections.

---

## 7. Event-Driven Architecture

Nginx uses asynchronous/event-driven networking rather than creating a dedicated heavyweight process for every connection.

This is one reason it can handle high concurrency efficiently.

---

## 8. Worker Processes Configuration

Typical configuration:

```nginx
worker_processes auto;
```

This lets Nginx determine an appropriate worker count based on available CPU resources.

---

## 9. Worker Connections

Example:

```nginx
events {
    worker_connections 4096;
}
```

This controls the maximum number of simultaneous connections each worker can handle subject to OS and other limits.

---

## 10. File Descriptor Limits

High-concurrency Nginx systems also depend on operating-system file descriptor limits.

Check:

```bash
ulimit -n
```

Systemd-managed services may have separate limits.

---

## 11. Nginx Configuration File

Common location:

```text
/etc/nginx/nginx.conf
```

Actual locations vary by distribution.

---

## 12. Configuration Hierarchy

A typical configuration structure:

```text
main
 ├── events
 └── http
      ├── upstream
      ├── server
      │    └── location
      └── server
```

---

## 13. Main Context

Contains global directives such as:

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
```

The exact user/path depends on the OS package.

---

## 14. Events Context

Example:

```nginx
events {
    worker_connections 4096;
}
```

---

## 15. HTTP Context

The `http` context contains HTTP-specific configuration:

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    server {
        ...
    }
}
```

---

## 16. Server Context

A `server` block defines a virtual server.

Example:

```nginx
server {
    listen 80;
    server_name example.com;
}
```

---

## 17. Location Context

A `location` block controls how requests matching a URI are processed.

Example:

```nginx
location /api/ {
    proxy_pass http://backend;
}
```

---

## 18. Directives

Nginx configuration is built from directives.

Examples:

```nginx
listen
server_name
proxy_pass
proxy_set_header
root
index
return
rewrite
```

---

## 19. Include Files

Large configurations should be split into reusable files.

Example:

```nginx
include /etc/nginx/conf.d/*.conf;
```

---

## 20. Configuration Test

Always validate before reload:

```bash
nginx -t
```

A successful configuration test should be a prerequisite for production reloads.

---

## 21. Reload

Graceful reload:

```bash
sudo nginx -s reload
```

or through systemd:

```bash
sudo systemctl reload nginx
```

---

## 22. Restart vs Reload

Reload:

```text
read new configuration
keep existing workers/connections as appropriate
```

Restart:

```text
stop
start
```

Reload is usually preferred for routine configuration changes.

---

## 23. Systemd

Common commands:

```bash
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
```

---

## 24. Enable at Boot

```bash
sudo systemctl enable nginx
```

---

## 25. Installation on Amazon Linux

Package names and repositories vary by Amazon Linux release.

Typical approach:

```bash
sudo dnf install nginx
```

Then:

```bash
sudo systemctl enable --now nginx
```

Verify the package manager and repository for the exact AMI version.

---

## 26. Installation on Ubuntu

Typical:

```bash
sudo apt update
sudo apt install nginx
```

Then:

```bash
sudo systemctl enable --now nginx
```

---

## 27. Verify Installation

```bash
nginx -v
nginx -V
```

`nginx -V` provides build/configuration information.

---

## 28. Verify Listening Ports

```bash
ss -lntp | grep nginx
```

Typical:

```text
0.0.0.0:80
```

---

## 29. Local Test

```bash
curl -I http://localhost
```

---

## 30. Basic Web Server

Example:

```nginx
server {
    listen 80;
    server_name example.com;

    root /usr/share/nginx/html;
    index index.html;
}
```

---

## 31. Static File Serving

Nginx is highly effective for serving:

```text
HTML
CSS
JavaScript
images
fonts
```

---

## 32. Static File Architecture

```text
Client
 |
Nginx
 |
Static Files
```

---

## 33. Reverse Proxy Architecture

```text
Client
 |
Nginx
 |
Application
```

---

## 34. Basic Reverse Proxy

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

---

## 35. proxy_pass

`proxy_pass` specifies the upstream destination.

Example:

```nginx
proxy_pass http://127.0.0.1:8080;
```

---

## 36. URI Behavior of proxy_pass

The trailing slash behavior of `proxy_pass` can change how the URI is forwarded.

Example:

```nginx
location /api/ {
    proxy_pass http://backend;
}
```

versus:

```nginx
location /api/ {
    proxy_pass http://backend/;
}
```

This distinction is important in production routing.

---

## 37. Host Header

Set the original host:

```nginx
proxy_set_header Host $host;
```

---

## 38. Client IP

Common configuration:

```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

---

## 39. Original Scheme

```nginx
proxy_set_header X-Forwarded-Proto $scheme;
```

---

## 40. Complete Reverse Proxy Example

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://backend;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 41. Upstream Block

Example:

```nginx
upstream backend {
    server 10.0.1.10:8080;
    server 10.0.2.10:8080;
}
```

Then:

```nginx
proxy_pass http://backend;
```

---

## 42. Why Use Upstream Blocks?

They provide a clean abstraction for backend pools and support load-balancing configuration.

---

## 43. Round Robin

Default upstream behavior is commonly round robin when no alternative method is configured.

Concept:

```text
A → B → C → A
```

---

## 44. Weight

Example:

```nginx
upstream backend {
    server 10.0.1.10:8080 weight=3;
    server 10.0.2.10:8080 weight=1;
}
```

The weighted distribution depends on request patterns and connection behavior.

---

## 45. Backup Server

Example:

```nginx
upstream backend {
    server 10.0.1.10:8080;
    server 10.0.2.10:8080 backup;
}
```

The backup server is used according to upstream failure behavior.

---

## 46. Max Fails

Example:

```nginx
server 10.0.1.10:8080 max_fails=3 fail_timeout=30s;
```

This controls passive failure handling.

---

## 47. Passive vs Active Health Checks

Open-source Nginx commonly relies on passive upstream failure detection.

More advanced active health-check capabilities may require Nginx Plus or external mechanisms.

---

## 48. Important Production Point

Do not assume that open-source Nginx upstream configuration provides the same active health-check features as commercial Nginx Plus.

Choose the required edition/architecture deliberately.

---

## 49. Least Connections

Nginx can use:

```nginx
upstream backend {
    least_conn;
    server 10.0.1.10:8080;
    server 10.0.2.10:8080;
}
```

---

## 50. IP Hash

Example:

```nginx
upstream backend {
    ip_hash;
    server 10.0.1.10:8080;
    server 10.0.2.10:8080;
}
```

Use carefully because client-IP distribution may be uneven.

---

## 51. Session Persistence

Avoid using local sessions when possible.

Prefer:

```text
stateless app
+
Redis/database session store
```

---

## 52. Proxy Connect Timeout

Example:

```nginx
proxy_connect_timeout 5s;
```

---

## 53. Proxy Read Timeout

Example:

```nginx
proxy_read_timeout 60s;
```

---

## 54. Proxy Send Timeout

Example:

```nginx
proxy_send_timeout 60s;
```

---

## 55. Timeout Design

Do not blindly set all timeouts to very large values.

Large timeouts can keep failed connections around and consume resources.

---

## 56. Keep-Alive

Example:

```nginx
keepalive 32;
```

inside an upstream can enable connection reuse to upstream servers.

Tune based on workload.

---

## 57. Client Keep-Alive

Example:

```nginx
keepalive_timeout 65;
```

---

## 58. Worker Connections and Keep-Alive

A high number of idle keep-alive connections can consume connection capacity.

Monitor actual concurrency before increasing limits.

---

## 59. Proxy Buffering

Example:

```nginx
proxy_buffering on;
```

Buffering can improve normal HTTP performance.

---

## 60. Disable Buffering for Streaming

For streaming use cases, buffering may need to be disabled:

```nginx
proxy_buffering off;
```

Validate against the application's behavior.

---

## 61. WebSocket Proxy

Typical:

```nginx
location /socket/ {
    proxy_pass http://websocket_backend;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

---

## 62. WebSocket Timeout

Long-lived WebSockets may require a larger:

```nginx
proxy_read_timeout
```

---

## 63. Large Uploads

Example:

```nginx
client_max_body_size 50m;
```

Choose a value based on actual application requirements.

---

## 64. Request Body Limits

Limits protect Nginx and upstream services from unexpectedly large requests.

---

## 65. Header Limits

Nginx has directives controlling client header buffers.

Oversized headers can consume memory and should be handled deliberately.

---

## 66. Static Content Caching

Example:

```nginx
location /static/ {
    root /var/www/html;
    expires 7d;
    add_header Cache-Control "public";
}
```

---

## 67. Cache-Control

Use application-appropriate cache semantics.

Immutable versioned assets are good candidates for long cache lifetimes.

---

## 68. Reverse Proxy Cache

Nginx can cache upstream responses using proxy cache directives.

Use caching only where response semantics are safe.

---

## 69. Cache Key

A cache key can depend on request characteristics such as:

```text
scheme
host
URI
query string
```

Design cache keys carefully.

---

## 70. Do Not Cache Sensitive Responses

Avoid caching:

```text
private account data
authentication responses
payment data
personalized content
```

unless explicitly designed to be safe.

---

## 71. Compression

Example:

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

---

## 72. Compression Trade-Off

Benefits:

```text
lower bandwidth
```

Cost:

```text
CPU
```

Avoid compressing already compressed formats unnecessarily.

---

## 73. Security Headers

Example:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

Security headers must be validated against the application's requirements.

---

## 74. HSTS

Example:

```nginx
add_header Strict-Transport-Security "max-age=31536000" always;
```

Only enable long-lived HSTS policies after confirming HTTPS is correctly deployed for the relevant domain/subdomains.

---

## 75. Content Security Policy

CSP can mitigate certain browser-side injection risks.

Example policies must be designed for the application's actual scripts/resources.

Do not copy a restrictive CSP blindly.

---

## 76. Server Tokens

Nginx can reduce version disclosure:

```nginx
server_tokens off;
```

This is defense in depth, not a substitute for patching.

---

## 77. TLS Configuration

Typical structure:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/tls/fullchain.pem;
    ssl_certificate_key /etc/nginx/tls/privkey.pem;
}
```

Use a trusted certificate chain and secure TLS settings.

---

## 78. HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    server_name example.com;

    return 301 https://$host$request_uri;
}
```

---

## 79. TLS Private Key Permissions

Private keys should be readable only by the required service account/process.

Example:

```bash
sudo chmod 600 /etc/nginx/tls/privkey.pem
```

Use ownership appropriate to the installed service.

---

## 80. Certificate Renewal

Production certificate renewal should be automated.

Possible approaches:

```text
ACM
cert-manager
Let's Encrypt automation
enterprise PKI
```

depending on architecture.

---

## 81. Nginx on AWS EC2

Architecture:

```text
Internet
 |
Route 53
 |
ALB
 |
EC2 Nginx
 |
Application
```

Nginx may be unnecessary if ALB already satisfies all edge requirements, but can be useful for application-specific proxy behavior.

---

## 82. Nginx as EC2 Edge Proxy

Alternative:

```text
Internet
 |
Nginx
 |
private application
```

This requires you to operate:

```text
HA
patching
scaling
monitoring
certificate management
```

---

## 83. Nginx HA on EC2

Better:

```text
                ALB
               /   \
          Nginx-A  Nginx-B
             |       |
          backend backend
```

Deploy across multiple AZs when appropriate.

---

## 84. Nginx Security Group

Example conceptual model:

```text
Internet
   |
ALB-SG
   |
Nginx-SG
   |
Backend-SG
```

Allow only required ports between layers.

---

## 85. Nginx in Docker

Basic Dockerfile:

```dockerfile
FROM nginx:stable-alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html/
```

Pin a tested image version/digest in production.

---

## 86. Docker Run

Example:

```bash
docker run -d \
  --name nginx \
  -p 8080:80 \
  nginx:stable-alpine
```

---

## 87. Containerized Nginx Configuration

Prefer immutable image/config deployment:

```text
Git
→ CI
→ image/config validation
→ registry
→ deployment
```

rather than manual edits inside running containers.

---

## 88. Nginx in Kubernetes

Nginx can run as:

```text
Deployment
+
Service
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

Production should pin image versions and include resources/probes/security context.

---

## 89. Nginx Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

---

## 90. Nginx ConfigMap

Configuration can be mounted from a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {}

    http {
      server {
        listen 80;
        location / {
          return 200 "ok\n";
        }
      }
    }
```

For production, validate the configuration during CI and consider immutable configuration/versioning strategies.

---

## 91. Nginx Ingress Controller

Nginx can also act as a Kubernetes Ingress Controller.

Architecture:

```text
Internet
 |
Load Balancer
 |
Nginx Ingress Controller
 |
Ingress rules
 |
Services
 |
Pods
```

---

## 92. Ingress Resource

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop
spec:
  ingressClassName: nginx
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

## 93. Nginx Ingress vs AWS ALB Ingress

Nginx Ingress:

```text
Kubernetes
→ Nginx proxy
→ services
```

AWS ALB Ingress:

```text
Kubernetes
→ AWS Load Balancer Controller
→ ALB
→ services/Pods
```

---

## 94. When to Use AWS ALB Instead

For an AWS-native EKS application requiring:

```text
HTTP/S
host/path routing
managed load balancing
AWS integration
```

ALB can reduce the operational burden of running another proxy tier.

---

## 95. When Nginx Ingress Is Useful

Nginx Ingress can be useful when you need:

```text
Nginx-specific features
portable Kubernetes ingress behavior
custom proxy configuration
existing Nginx operational expertise
```

---

## 96. Nginx Ingress Architecture

```text
                    Internet
                       |
                    DNS/WAF
                       |
                  Load Balancer
                       |
                Nginx Ingress
                /     |      \
               /      |       \
          frontend   API     admin
              |       |        |
           Service  Service  Service
              |       |        |
             Pods    Pods      Pods
```

---

## 97. Nginx Ingress High Availability

Use multiple controller replicas:

```yaml
spec:
  replicas: 3
```

and spread across failure domains where possible.

---

## 98. Pod Anti-Affinity

For production ingress controllers, schedule replicas across nodes/AZs using:

```text
topology spread constraints
pod anti-affinity
```

according to cluster design.

---

## 99. Resource Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

Tune from observed workload.

---

## 100. Nginx Health Probes

Example:

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 10254
```

The exact health endpoint depends on the Nginx image/controller being used.

---

## 101. Nginx Security Context

Production containers should consider:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

but the complete settings must match the Nginx image's filesystem/runtime requirements.

---

## 102. Running Nginx Non-Root

Nginx can run non-root with an appropriate image/configuration.

Some standard configurations require privileged binding to ports below 1024 or special capabilities.

Do not simply change the user without validating all required permissions.

---

## 103. Read-Only Root Filesystem

A read-only root filesystem improves security but Nginx may need writable locations for:

```text
PID
temporary buffers
cache
logs
```

Use explicit writable volumes/tmpfs where needed.

---

## 104. Nginx Temporary Paths

Production containers may need temporary paths for:

```text
client body
proxy
fastcgi
uwsgi
scgi
```

Configure according to workload.

---

## 105. Nginx Logging in Kubernetes

Prefer writing logs to:

```text
stdout
stderr
```

so Kubernetes logging agents can collect them.

---

## 106. Container Log Collection

Typical:

```text
Nginx
 |
stdout/stderr
 |
Fluent Bit/agent
 |
Logstash/Elasticsearch
 |
Kibana
```

---

## 107. Nginx Access Log Format

Useful fields:

```text
time
remote address
request
status
bytes
request time
upstream response time
upstream status
user agent
request ID
```

---

## 108. Structured Logs

JSON logs can simplify downstream parsing.

Example conceptual output:

```json
{
  "status": 200,
  "request_time": 0.023,
  "upstream_time": 0.019
}
```

---

## 109. Nginx Error Log

Error logs are essential for diagnosing:

```text
upstream failures
TLS errors
configuration errors
connection failures
permission problems
```

---

## 110. Log Levels

Choose an appropriate production log level.

Excessively verbose logging can:

```text
increase cost
increase I/O
hide important signals
```

---

## 111. Log Rotation

On VM deployments, configure log rotation using the operating system's logrotate/systemd strategy.

In containers, prefer stdout/stderr with centralized retention.

---

## 112. Nginx Metrics

Open-source Nginx does not expose the same full metric set as every commercial/observability integration.

Use suitable:

```text
exporter
Nginx Plus API
Ingress controller metrics
log-based metrics
```

depending on deployment.

---

## 113. Prometheus

A Prometheus exporter can expose Nginx metrics.

Typical architecture:

```text
Nginx
 |
Exporter
 |
Prometheus
 |
Grafana
```

---

## 114. Grafana Panels

Recommended:

```text
requests
active connections
accepted connections
handled connections
5xx
4xx
latency
upstream failures
```

Exact metric names depend on exporter.

---

## 115. ELK

Nginx logs can be shipped to:

```text
Logstash
→ Elasticsearch
→ Kibana
```

Useful for request-level investigation.

---

## 116. Request Correlation

Use a request ID:

```text
X-Request-ID
```

and propagate it to upstream services.

---

## 117. OpenTelemetry

Modern applications can use OpenTelemetry to correlate proxy traffic with distributed traces.

Nginx integration depends on deployment/tooling.

---

## 118. Rate Limiting

Example:

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

Then:

```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    proxy_pass http://backend;
}
```

Tune values based on business traffic.

---

## 119. Connection Limiting

Nginx can limit concurrent connections.

Example:

```nginx
limit_conn_zone $binary_remote_addr zone=addr:10m;
```

Use carefully because NAT can cause many legitimate users to appear under one public IP.

---

## 120. Rate Limiting by API Key

Rate limiting can use a variable representing an authenticated identity/API key.

Ensure the identity is trusted and not client-spoofable.

---

## 121. DDoS Considerations

Nginx can provide some request-level controls but should not be treated as a complete DDoS defense.

Use appropriate upstream protections such as:

```text
AWS Shield
WAF
CloudFront
rate limiting
network controls
```

where applicable.

---

## 122. IP Allowlist

Example:

```nginx
location /admin/ {
    allow 10.0.0.0/8;
    deny all;
}
```

Do not use broad private ranges without understanding the actual network trust boundary.

---

## 123. Basic Authentication

Nginx can support basic authentication using an htpasswd file.

For production enterprise access, prefer a centralized identity provider where practical.

---

## 124. Reverse Proxy Authentication

Possible architecture:

```text
Client
 |
SSO/Auth Layer
 |
Nginx
 |
Application
```

Authentication requirements depend on application architecture.

---

## 125. CORS

Nginx can add CORS headers, but CORS is a browser security policy and should be designed with the application's origin model.

Avoid:

```text
Access-Control-Allow-Origin: *
```

for sensitive authenticated applications unless explicitly intended.

---

## 126. OPTIONS Requests

CORS preflight requests commonly use:

```text
OPTIONS
```

Ensure proxy routing handles them correctly.

---

## 127. URI Rewriting

Nginx can rewrite URLs.

Example:

```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;
```

Test rewrite rules carefully.

---

## 128. return Directive

Simple redirects/responses:

```nginx
return 301 https://$host$request_uri;
```

---

## 129. Location Matching

Nginx location matching has specific rules involving:

```text
prefix
exact =
regular expression ~
case-insensitive regex ~*
```

Understand matching precedence before using complex configurations.

---

## 130. Exact Match

Example:

```nginx
location = /health {
    return 200 "OK";
}
```

---

## 131. Prefix Match

Example:

```nginx
location /api/ {
    proxy_pass http://api;
}
```

---

## 132. Regex Match

Example:

```nginx
location ~ \.php$ {
    ...
}
```

Use regex locations only when necessary because they can complicate configuration behavior.

---

## 133. Location Troubleshooting

When routing is unexpected, determine:

```text
which location matched
```

then inspect:

```text
prefix
exact match
regex
nested locations
```

---

## 134. Server Name Matching

Nginx can select a `server` block using:

```text
listen address/port
Host header
server_name
```

---

## 135. Default Server

If no server name matches, Nginx may use the default server for that listen socket.

Configure explicit defaults for predictable behavior.

---

## 136. Host Header Security

Do not blindly use arbitrary Host headers for:

```text
redirects
absolute URLs
security decisions
```

Validate expected hostnames.

---

## 137. Request Smuggling

Proxy/backend parsing differences can create HTTP request-smuggling risks.

Keep Nginx and upstream components patched and avoid inconsistent HTTP parsing assumptions.

---

## 138. Open Redirect

Avoid constructing redirects from untrusted request headers without validation.

---

## 139. SSRF Considerations

If Nginx configuration or upstream selection can be influenced by user-controlled URLs, validate allowed destinations.

---

## 140. Upstream DNS

Dynamic upstream DNS requires deliberate resolver configuration.

Do not assume all runtime DNS changes are automatically re-resolved in every Nginx configuration pattern.

---

## 141. DNS Resolver

For dynamic resolution, Nginx can use a `resolver` directive.

Example:

```nginx
resolver 10.0.0.2 valid=30s;
```

Use the actual VPC/Kubernetes DNS resolver for the environment.

---

## 142. Nginx and Kubernetes DNS

A Pod can resolve services using cluster DNS:

```text
service.namespace.svc.cluster.local
```

Nginx configuration must account for DNS resolution behavior.

---

## 143. Nginx and Kubernetes Service

Example upstream concept:

```nginx
upstream backend {
    server cart.roboshop.svc.cluster.local:8080;
}
```

For dynamic Kubernetes endpoints, a Kubernetes-native Service/Ingress implementation is often preferable to manually maintaining Pod IPs.

---

## 144. Nginx and ConfigMap

Kubernetes configuration:

```text
ConfigMap
 |
volume mount
 |
/etc/nginx/nginx.conf
```

A configuration change may require a Pod reload/restart depending on how it is mounted and managed.

---

## 145. GitOps Nginx

Production pattern:

```text
Git
 |
CI validation
 |
Argo CD
 |
ConfigMap/Deployment
 |
EKS
```

---

## 146. GitOps Nginx Repository

Example:

```text
gitops-repo/
├── applications/
├── applicationsets/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── helm/
    └── nginx/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
```

---

## 147. Helm Values

Example:

```yaml
replicaCount: 3

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

---

## 148. Production Helm Strategy

Use:

```text
common values
environment values
secret references
versioned chart
```

Avoid copying entire configurations unnecessarily between environments.

---

## 149. Nginx and Argo CD

Argo CD can deploy:

```text
Nginx Deployment
Service
ConfigMap
Ingress
HPA
NetworkPolicy
```

from Git.

---

## 150. Production Nginx Deployment

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: edge
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          securityContext:
            allowPrivilegeEscalation: false
```

The exact port/configuration must match the selected image and Nginx configuration.

---

## 151. Production Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: edge
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

---

## 152. Nginx ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: edge
data:
  nginx.conf: |
    worker_processes auto;

    events {
      worker_connections 4096;
    }

    http {
      include /etc/nginx/mime.types;
      default_type application/octet-stream;

      server {
        listen 8080;

        location = /healthz {
          access_log off;
          return 200 "ok\n";
        }

        location / {
          proxy_pass http://frontend.roboshop.svc.cluster.local:80;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
        }
      }
    }
```

Validate DNS and upstream behavior in the target cluster before production use.

---

## 153. NetworkPolicy for Nginx

Example conceptual policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx
  namespace: edge
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: roboshop
      ports:
        - protocol: TCP
          port: 80
```

Adapt selectors and DNS egress rules to the actual CNI/network-policy implementation.

---

## 154. Nginx HPA

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: edge
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 3
  maxReplicas: 10
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
    scaleDown:
      stabilizationWindowSeconds: 300
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

Ensure Metrics Server/resource metrics are available.

---

## 155. Pod Disruption Budget

For critical Nginx replicas:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx
  namespace: edge
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: nginx
```

---

## 156. Topology Spread

Production replicas should be spread across nodes/AZs where possible.

Example concept:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: nginx
```

---

## 157. Nginx Ingress Production Considerations

For an Nginx Ingress Controller:

```text
multiple replicas
PDB
resource limits
topology spread
RBAC
Service
external LB
metrics
logs
TLS
```

---

## 158. Nginx vs AWS ALB Production Decision

Use ALB when:

```text
AWS-managed L7 ingress is sufficient
```

Use Nginx when:

```text
custom proxy behavior
existing Nginx standard
special routing/configuration
```

Do not add Nginx merely because it is familiar.

---

## 159. Nginx vs Envoy

```text
Nginx:
mature reverse proxy/web server

Envoy:
dynamic proxy for distributed systems
```

For service mesh architectures, Envoy often provides capabilities that are outside a basic Nginx deployment.

---

## 160. Nginx vs HAProxy

Both support:

```text
reverse proxy
load balancing
TLS
health/failure handling
```

Choose based on operational requirements.

---

## 161. Production Architecture: EC2

```text
                    Internet
                       |
                    Route 53
                       |
                      WAF
                       |
                      ALB
                     /   \
                    /     \
               Nginx-A   Nginx-B
                  |         |
                  +----+----+
                       |
                 Private App
```

---

## 162. Production Architecture: EKS

```text
                    Internet
                       |
                    Route 53
                       |
                      WAF
                       |
                      ALB
                       |
             AWS Load Balancer Controller
                       |
                    Ingress
                       |
                    Service
                       |
                 Nginx/Pods
                       |
               Internal Services
```

---

## 163. Production Architecture: Nginx Ingress

```text
Internet
   |
AWS LB
   |
Nginx Ingress
   |
+--+----------------+
|                   |
Service A        Service B
|                   |
Pods               Pods
```

---

## 164. Production Architecture: RoboShop

```text
Developer
   |
Git
   |
CI
   |
Build/Test/Security
   |
Docker Image
   |
ECR
   |
GitOps Repository
   |
Argo CD
   |
EKS
   |
AWS Load Balancer Controller
   |
ALB
   |
Nginx/Frontend
   |
Internal Kubernetes Services
   |
Microservices
```

---

## 165. RoboShop External Traffic

```text
Internet
 |
Route 53
 |
WAF
 |
ALB
 |
frontend
```

No API Gateway is required in this architecture.

---

## 166. RoboShop Internal Traffic

```text
frontend
 |
catalogue
 |
user
 |
cart
 |
payment
 |
shipping
```

Services should communicate through internal Kubernetes networking.

---

## 167. Nginx as RoboShop Frontend Proxy

Nginx can serve frontend assets and proxy selected API paths.

Concept:

```text
Nginx
 |
+-- /       → frontend files
|
+-- /api/   → backend services
```

---

## 168. RoboShop Nginx Configuration

Example conceptual configuration:

```nginx
server {
    listen 8080;

    root /usr/share/nginx/html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://api.roboshop.svc.cluster.local:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The actual service names/ports must match the deployed RoboShop application.

---

## 169. Production Nginx CI Validation

CI should validate:

```bash
nginx -t
```

inside the same or equivalent container/image used for deployment.

---

## 170. GitOps Validation

Recommended:

```text
Git commit
→ CI lint
→ nginx -t
→ YAML validation
→ security scanning
→ image scan
→ pull request
→ merge
→ Argo CD
```

---

## 171. Immutable Nginx Images

Prefer:

```text
nginx:<tested-version>
```

or an immutable digest.

Avoid:

```text
nginx:latest
```

in production.

---

## 172. Nginx Image Scanning

Use:

```text
Trivy
```

or an approved vulnerability scanner in CI.

---

## 173. Nginx Configuration Security Scanning

Validate:

```text
TLS
headers
weak ciphers
unnecessary exposure
dangerous directives
```

using appropriate security tooling.

---

## 174. Secret Management

Do not commit:

```text
private keys
passwords
API tokens
basic-auth credentials
```

to Git.

Use:

```text
AWS Secrets Manager
External Secrets
Kubernetes Secret with appropriate encryption/access controls
```

---

## 175. Certificate Management in EKS

For AWS ALB, prefer ACM where appropriate.

For Nginx itself, consider:

```text
cert-manager
enterprise PKI
managed certificate distribution
```

depending on architecture.

---

## 176. Disaster Recovery

Nginx configuration should be reproducible from:

```text
Git
container image
Helm
Terraform
Argo CD
```

Do not depend on manual server changes.

---

## 177. Nginx Backup

Back up:

```text
configuration
certificates where appropriate
deployment manifests
```

Secrets require separate secure backup and access controls.

---

## 178. Nginx Upgrade Strategy

Use:

```text
new image/version
CI validation
security scan
staging
canary/rolling deployment
production monitoring
```

---

## 179. Nginx Security Updates

Patch quickly for:

```text
critical CVEs
TLS vulnerabilities
dependency issues
base image vulnerabilities
```

Balance emergency patching with controlled rollout.

---

## 180. Nginx Troubleshooting Workflow

```text
1. Check process.
2. Check configuration.
3. Check listener.
4. Check DNS.
5. Check TLS.
6. Check location.
7. Check upstream.
8. Check network.
9. Check application.
10. Check logs/metrics.
```

---

## 181. Process Check

```bash
ps aux | grep nginx
systemctl status nginx
```

---

## 182. Configuration Check

```bash
nginx -t
```

---

## 183. Listening Port Check

```bash
ss -lntp | grep nginx
```

---

## 184. Local HTTP Test

```bash
curl -v http://127.0.0.1
```

---

## 185. Host Header Test

```bash
curl -v \
  -H 'Host: api.example.com' \
  http://127.0.0.1
```

Useful when multiple server blocks exist.

---

## 186. TLS Test

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

---

## 187. Upstream Connectivity Test

```bash
curl -v http://10.0.1.10:8080/health
```

or use the appropriate DNS/service name.

---

## 188. Nginx Access Logs

```bash
tail -f /var/log/nginx/access.log
```

---

## 189. Nginx Error Logs

```bash
tail -f /var/log/nginx/error.log
```

---

## 190. 502 Bad Gateway

Common causes:

```text
backend unavailable
wrong port
connection refused
protocol mismatch
upstream reset
DNS issue
```

---

## 191. 503 Service Unavailable

Possible causes:

```text
no available upstream
overload
upstream failure
configuration
```

---

## 192. 504 Gateway Timeout

Possible causes:

```text
slow backend
network problem
timeout too low
dependency failure
```

---

## 193. 404 Not Found

Check:

```text
location matching
root
proxy_pass
upstream route
application route
```

---

## 194. 403 Forbidden

Check:

```text
filesystem permissions
allow/deny
SELinux/AppArmor where applicable
application authorization
```

---

## 195. Permission Denied

Check:

```bash
ls -l /path
namei -l /path/to/file
```

Also inspect service user and mandatory access controls.

---

## 196. Nginx Won't Start

Run:

```bash
nginx -t
journalctl -u nginx -xe
```

Look for:

```text
syntax errors
port conflicts
permission errors
missing certificates
invalid includes
```

---

## 197. Port Already in Use

Check:

```bash
ss -lntp
```

Find the process using the desired port.

---

## 198. Certificate Error

Check:

```text
certificate chain
key match
hostname
expiration
permissions
TLS configuration
```

---

## 199. Upstream DNS Error

Check:

```bash
getent hosts backend.example.com
```

Then verify resolver configuration.

---

## 200. Kubernetes Nginx Troubleshooting

```bash
kubectl get pods -n edge
kubectl describe pod <pod> -n edge
kubectl logs <pod> -n edge
kubectl get svc -n edge
kubectl get endpointslice -n edge
```

---

## 201. ConfigMap Troubleshooting

Check:

```bash
kubectl get configmap nginx-config -n edge -o yaml
```

Then verify the mounted file inside the Pod.

---

## 202. Nginx Container Configuration Test

If the image supports it:

```bash
kubectl exec -n edge deploy/nginx -- nginx -t
```

---

## 203. Ingress Troubleshooting

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
```

Then inspect the controller logs.

---

## 204. Nginx Ingress Controller Troubleshooting

Check:

```bash
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <controller-pod>
kubectl get ingressclass
```

Actual namespace/name can differ.

---

## 205. NetworkPolicy Troubleshooting

If Nginx cannot reach backend:

```text
DNS
→ Service
→ EndpointSlice
→ NetworkPolicy
→ Pod listener
```

Test from inside the Nginx Pod.

---

## 206. DNS Test From Pod

```bash
kubectl exec -it <nginx-pod> -n edge -- \
  getent hosts frontend.roboshop.svc.cluster.local
```

---

## 207. Connectivity Test From Pod

```bash
kubectl exec -it <nginx-pod> -n edge -- \
  curl -v http://frontend.roboshop.svc.cluster.local:80
```

---

## 208. Nginx 502 in Kubernetes

Check:

```text
Service name
Service port
targetPort
EndpointSlice
Pod readiness
NetworkPolicy
Nginx proxy_pass
```

---

## 209. Nginx 504 in Kubernetes

Check:

```text
application latency
Service networking
DNS
timeouts
dependency
CPU/memory
```

---

## 210. Nginx Ingress 404

Check:

```text
IngressClass
host
path
Ingress rules
Service
controller logs
```

---

## 211. Nginx Ingress 503

Usually investigate:

```text
service
endpoints
Pod readiness
controller configuration
```

---

## 212. Nginx Ingress TLS Failure

Check:

```text
TLS Secret
certificate
host
Ingress
secret namespace
controller logs
```

---

## 213. Nginx Ingress and AWS Load Balancer

Typical:

```text
Internet
 |
AWS Load Balancer
 |
Nginx Ingress
 |
Service
```

This introduces two routing layers and should be justified operationally.

---

## 214. Avoid Unnecessary Proxy Layers

Bad:

```text
CloudFront
→ ALB
→ Nginx
→ another proxy
→ application
```

Every layer adds:

```text
latency
configuration
failure modes
cost
```

Add a proxy only when it provides required capabilities.

---

## 215. Production Nginx Layering Decision

For EKS web applications:

```text
Preferred AWS-native:
Route 53 → WAF → ALB → Service → Pods

Use Nginx when:
custom proxy behavior is actually required.
```

---

## 216. Nginx Production Checklist

```text
[ ] configuration in Git
[ ] nginx -t in CI
[ ] immutable image/version
[ ] TLS
[ ] certificate automation
[ ] security headers
[ ] request limits
[ ] rate limits where required
[ ] timeouts
[ ] health checks
[ ] readiness
[ ] graceful shutdown
[ ] logs
[ ] metrics
[ ] alerting
[ ] vulnerability scanning
[ ] HA
[ ] rollback
```

---

## 217. Interview: What Is Nginx?

Nginx is a high-performance web server and reverse proxy commonly used for HTTP serving, TLS termination, routing, load balancing, and caching.

---

## 218. Interview: How Does Nginx Work Internally?

A master process manages configuration and workers; worker processes use an event-driven architecture to handle connections.

---

## 219. Interview: What Is nginx.conf?

The primary Nginx configuration file containing global, events, HTTP, server, location, and related directives.

---

## 220. Interview: What Is a Server Block?

A virtual server configuration that defines how Nginx handles a hostname/listener combination.

---

## 221. Interview: What Is a Location Block?

A configuration block that determines how matching URI requests are processed.

---

## 222. Interview: What Is proxy_pass?

A directive that forwards requests to an upstream server or upstream group.

---

## 223. Interview: What Is an Upstream?

A logical group of backend servers used for proxying/load balancing.

---

## 224. Interview: Does Open-Source Nginx Have Active Health Checks?

Open-source Nginx primarily provides passive upstream failure handling. Active health-check capabilities differ by edition and integration.

---

## 225. Interview: What Is Least Connections?

A load-balancing method that favors upstreams with fewer active connections.

---

## 226. Interview: What Is IP Hash?

A method that selects upstreams based on client IP, providing a form of session affinity.

---

## 227. Interview: Why Is Statelessness Better Than Sticky Sessions?

It allows traffic to move freely between replicas and improves scalability and failure recovery.

---

## 228. Interview: What Is X-Forwarded-For?

An HTTP header used to propagate client IP information through proxies.

---

## 229. Interview: Can X-Forwarded-For Be Trusted?

Only when inserted/overwritten by a trusted proxy boundary.

---

## 230. Interview: What Is proxy_read_timeout?

The timeout controlling how long Nginx waits for data from an upstream after a request has been forwarded.

---

## 231. Interview: What Causes Nginx 502?

Common causes include upstream connection refusal, wrong port, invalid response, protocol mismatch, or upstream reset.

---

## 232. Interview: What Causes Nginx 504?

An upstream did not respond within the configured timeout or the network/backend path prevented a timely response.

---

## 233. Interview: How Do You Troubleshoot Nginx?

Use:

```bash
nginx -t
systemctl status nginx
ss -lntp
curl -v
tail -f /var/log/nginx/error.log
```

Then inspect upstream connectivity.

---

## 234. Interview: How Do You Troubleshoot Nginx in Kubernetes?

Check:

```text
Pod
ConfigMap
Service
EndpointSlice
DNS
NetworkPolicy
Ingress
controller logs
```

---

## 235. Interview: What Is Nginx Ingress Controller?

A Kubernetes controller that uses Nginx to implement Ingress resources and route external HTTP/S traffic to Kubernetes Services.

---

## 236. Interview: Nginx Ingress vs AWS ALB Ingress?

Nginx Ingress runs a proxy layer inside Kubernetes.

AWS ALB Ingress uses AWS Load Balancer Controller to configure an AWS ALB.

---

## 237. Interview: When Would You Choose ALB Over Nginx?

When AWS-managed HTTP/S load balancing and routing satisfy the requirements and you want to reduce proxy infrastructure operations.

---

## 238. Interview: When Would You Choose Nginx?

When you require Nginx-specific proxy features, custom behavior, portability, or an established Nginx operating model.

---

## 239. Interview: How Do You Make Nginx Highly Available?

Use:

```text
multiple replicas/instances
multiple AZs
external load balancer
health checks
graceful deployment
```

---

## 240. Interview: How Do You Secure Nginx?

Use:

```text
TLS
patched versions
least privilege
request limits
rate limiting
security headers
trusted forwarding headers
restricted admin access
logging
monitoring
```

---

## 241. Interview: How Do You Deploy Nginx Through GitOps?

```text
Git
→ CI validation
→ image/config
→ GitOps repo
→ Argo CD
→ EKS
```

---

## 242. Interview: Why Should Nginx Configuration Be in Git?

For:

```text
auditability
review
rollback
reproducibility
disaster recovery
```

---

## 243. Interview: How Do You Roll Back Nginx?

Revert the Git change/image version and let the deployment mechanism restore the known-good configuration.

---

## 244. Interview: What Is Graceful Reload?

Nginx loads new configuration while allowing existing workers/connections to finish according to its graceful shutdown behavior.

---

## 245. Interview: What Is Connection Pooling?

Reusing upstream connections to reduce repeated connection-establishment overhead.

---

## 246. Interview: What Is Proxy Buffering?

Temporarily buffering request/response data between the client and upstream.

---

## 247. Interview: When Should Proxy Buffering Be Disabled?

For appropriate streaming/real-time workloads where buffering delays data delivery.

---

## 248. Interview: How Do You Handle WebSockets?

Use HTTP/1.1 upgrade headers and suitable timeout/connection configuration.

---

## 249. Interview: What Is Nginx Rate Limiting?

A mechanism that limits request frequency or concurrent connections to protect Nginx/upstream services.

---

## 250. Interview: Why Can Rate Limiting by IP Be Problematic?

Many legitimate users may share one public IP through NAT, while a single attacker can use many IPs.

---

## 251. Interview: What Is Nginx Caching?

Storing eligible upstream responses so future requests can be served without contacting the backend every time.

---

## 252. Interview: What Should Not Be Cached?

Sensitive personalized/authenticated data unless caching is explicitly safe and correctly keyed.

---

## 253. Interview: How Do You Monitor Nginx?

Use:

```text
logs
exporter/metrics
Prometheus
Grafana
ELK
tracing
```

depending on the deployment.

---

## 254. Interview: What Is the Most Important Nginx Troubleshooting Command?

```bash
nginx -t
```

It quickly identifies configuration syntax/load problems before a reload.

---

## 255. Interview: Why Does Nginx Show 502 While Backend Works With Curl?

The curl test may originate from a different network location or use a different DNS/port/protocol. Test connectivity from the Nginx host/Pod itself and inspect Nginx error logs.

---

## 256. Interview: How Do You Debug a 504?

Trace:

```text
client
→ Nginx
→ upstream
→ dependency
```

and compare request time, upstream time, timeout values, and application logs.

---

## 257. Interview: Why Can a Kubernetes Service Work but Nginx Still Fail?

Possible causes:

```text
wrong DNS name
wrong port
proxy_pass URI behavior
NetworkPolicy
Nginx resolver behavior
application Host header
```

---

## 258. Interview: What Is the Recommended EKS Public Ingress for RoboShop?

A strong AWS-native pattern is:

```text
Route 53
→ WAF
→ ALB
→ frontend Service/Pods
```

with Nginx added only when there is a concrete requirement for it.

---

## 259. Final Nginx Mental Model

```text
                   Client
                      |
                   DNS/TLS
                      |
                 Nginx Proxy
                 /    |    \
                /     |     \
          Static     API    WebSocket
             |        |        |
          Files    Upstream  Upstream
                      |
                 Application
```

---

## 260. Final EKS Mental Model

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                           ALB
                            |
                  AWS Load Balancer
                       Controller
                            |
                         Ingress
                            |
                      Nginx (optional)
                            |
                         Service
                            |
                          Pods
                            |
                     Internal Services
```

---

## 261. Final Production Rules

```text
1. Use Nginx when it provides a required capability.
2. Prefer managed AWS ingress when it already satisfies the requirement.
3. Keep Nginx configuration in Git.
4. Run nginx -t in CI.
5. Pin production image versions.
6. Use TLS and automate certificate lifecycle.
7. Do not blindly trust forwarded headers.
8. Tune timeouts and connection limits.
9. Use readiness and graceful shutdown.
10. Monitor logs, metrics, and latency.
11. Test 502/503/504 scenarios.
12. Keep Nginx patched.
13. Avoid unnecessary proxy layers.
14. Design for multi-AZ availability.
15. Make rollback reproducible.
```

---

## 262. Next File

The next planned file is:

```text
19-AWS-VPC-Networking.md
```

It will cover:

```text
AWS VPC architecture
CIDR
subnets
route tables
internet gateway
NAT gateway
VPC endpoints
security groups
NACLs
DNS
DHCP options
ENIs
secondary IPs
AWS networking for EKS
public/private subnets
multi-AZ architecture
routing
VPC peering
Transit Gateway
PrivateLink concepts
VPC Flow Logs
network security
Terraform
production architecture
RoboShop
troubleshooting
interview preparation
```

# End of 18-Nginx.md
