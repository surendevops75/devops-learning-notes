# Docker Network Types - Bridge, Host and None

## Introduction

Docker provides multiple network drivers to enable communication between:

```text
Containers

Host Machine

External Systems

Internet
```

---

The three most common network types are:

```text
Bridge

Host

None
```

---

Understanding these networks is important for:

```text
Docker

Docker Compose

Kubernetes

Microservices

Production Troubleshooting
```

---

# Docker Network Drivers

List Available Networks

```bash
docker network ls
```

---

Example Output

```text
NETWORK ID     NAME      DRIVER

xxxxx          bridge    bridge

xxxxx          host      host

xxxxx          none      null
```

---

Docker creates:

```text
bridge

host

none
```

automatically during installation.

---

# Bridge Network

## What is Bridge Network?

Bridge network is:

```text
Docker's Default Network
```

---

Docker creates:

```text
docker0
```

virtual bridge interface.

---

Verify

```bash
ip addr
```

---

Output

```text
docker0
```

---

# Bridge Architecture

```text
Host Server
       ↓

docker0 Bridge
       ↓

Container A
172.17.0.2

Container B
172.17.0.3
```

---

Each container receives:

```text
Private IP Address
```

---

Docker performs:

```text
Routing

NAT

Forwarding
```

---

# Running Container in Bridge Network

```bash
docker run -d nginx
```

---

Container automatically joins:

```text
bridge
```

network.

---

Verify

```bash
docker inspect container-id
```

---

Network Section

```text
bridge
```

---

# Bridge Network Communication

Containers communicate using:

```text
IP Address
```

or

```text
Container Name
```

inside custom bridge networks.

---

Example

```text
172.17.0.2

172.17.0.3
```

---

# Bridge Network Advantages

```text
Container Isolation

Separate IPs

Safe Default Option

Most Common Deployment Model
```

---

# Bridge Network Disadvantages

```text
Additional NAT Layer

Slight Performance Overhead

Requires Port Mapping
```

---

# Host Network

## What is Host Network?

Host network means:

```text
Container Uses
Host Network Directly
```

---

No:

```text
docker0

Virtual Interface

NAT
```

---

Container shares:

```text
Host IP

Host Ports

Host Network Stack
```

---

# Host Network Architecture

```text
Container
      ↓

Host Network
      ↓

Internet
```

---

No bridge involved.

---

# Running Host Network Container

```bash
docker run --network host nginx
```

---

Container uses:

```text
Host Port 80
```

directly.

---

No need for:

```bash
-p 80:80
```

---

# Host Network Example

Bridge Network

```bash
docker run -p 80:80 nginx
```

---

Host Network

```bash
docker run --network host nginx
```

---

Application immediately available on:

```text
Host IP
```

---

# Host Network Benefits

```text
Better Performance

No NAT

Lower Latency

Direct Host Access
```

---

Useful for:

```text
Monitoring Agents

High Performance Applications

Network Appliances
```

---

# Host Network Limitations

Suppose

```text
Container A

Container B
```

both run Nginx.

---

Host Network

```text
Port 80
```

already used.

---

Second container fails.

---

Reason

```text
Both Share Host Ports
```

---

# Host Network Security Risk

Container gains visibility into:

```text
Host Networking
```

---

Less isolation compared to:

```text
Bridge Network
```

---

Production teams usually use:

```text
Bridge Networks
```

unless performance requires host mode.

---

# None Network

## What is None Network?

None means:

```text
No Networking
```

---

Container receives:

```text
Loopback Interface Only
```

---

No:

```text
Internet

External Access

Container Access
```

---

# None Network Architecture

```text
Container
     ↓

lo (Loopback)
```

---

Nothing else.

---

# Running None Network

```bash
docker run --network none nginx
```

---

Verify

```bash
docker exec -it container-id bash
```

---

Check Interfaces

```bash
ip addr
```

---

Output

```text
lo
```

only.

---

# None Network Use Cases

```text
Security Testing

Offline Processing

Batch Jobs

Isolated Containers
```

---

# Compare Network Types

| Feature | Bridge | Host | None |
|----------|---------|---------|---------|
| IP Address | Separate IP | Host IP | No IP |
| Isolation | High | Low | Highest |
| Port Mapping Needed | Yes | No | N/A |
| Internet Access | Yes | Yes | No |
| Container Communication | Yes | Limited By Ports | No |
| Performance | Good | Best | N/A |

---

# Real Production Example

Roboshop

Most services use:

```text
Bridge Network
```

---

Example

```text
Frontend

Catalogue

Cart

User

Shipping

RabbitMQ

MongoDB
```

---

Reason

```text
Service Discovery

Isolation

DNS Resolution
```

---

Monitoring Agents

Sometimes use:

```text
Host Network
```

---

Examples

```text
Node Exporter

Datadog Agent

Network Monitoring Tools
```

---

Highly Secure Containers

Sometimes use:

```text
None Network
```

---

# Troubleshooting

List Networks

```bash
docker network ls
```

---

Inspect Network

```bash
docker network inspect bridge
```

---

Verify Container Network

```bash
docker inspect container-id
```

---

Check Interfaces

```bash
ip addr
```

inside container.

---

# Interview Scenario

Question:

```text
Why Does Host Network
Not Need Port Mapping?
```

---

Answer:

```text
Container Directly Uses
Host Network Stack
```

---

Question:

```text
Why Is Bridge Network
More Common?
```

---

Answer:

```text
Better Isolation

Container Separation

Safer For Production
```

---

Question:

```text
Can Containers Communicate
In None Network?
```

---

Answer:

```text
No

Networking Disabled
```

---

# Common Interview Questions

## What is Bridge Network?

### Short Answer

Docker's default virtual network that provides isolated networking for containers.

### Detailed Explanation

Containers receive private IPs and communicate through the docker0 bridge interface.

### Production Example

Roboshop containers connected through a custom bridge network.

### Follow-Up Questions

- What is docker0?
- Why use custom bridge networks?

---

## What is Host Network?

### Short Answer

Container directly uses the host's network stack.

### Detailed Explanation

No NAT or bridge network is involved. Container ports are directly exposed on the host.

### Production Example

Running Node Exporter with host networking.

### Follow-Up Questions

- Why is port mapping not needed?
- What are the risks?

---

## What is None Network?

### Short Answer

A network mode with networking disabled.

### Detailed Explanation

Container only has loopback access and cannot communicate externally.

### Production Example

Security-sensitive batch processing containers.

### Follow-Up Questions

- Does it have internet access?
- When would you use it?

---

# Key Takeaways

```text
Bridge is Docker's default network.

Bridge networks provide isolation and private IP addresses.

Host network shares the host's networking stack.

Host network does not require port mapping.

Host network provides better performance but less isolation.

None network disables networking completely.

Bridge is the most commonly used network type in production.

Host is used for performance-sensitive workloads.

None is used for isolated workloads and security scenarios.

Understanding these network types is essential for Docker and Kubernetes networking.
```