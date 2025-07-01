---
date: 2025-07-01
tags:
  - course
hubs:
  - "[[gcp]]"
  - "[[system-design]]"
  - "[[devops-concept]]"
urls:
  -
---

# cloud native design patterns

## Load Balancing

### Objective

Enable systems to handle massive scale (billions of requests/day, petabytes of
data) in a cost-effective, cloud-native way by distributing work across many
instances.

### Problem Statement

- A single server cannot sustain high traffic: CPU, memory, or network become
  bottlenecks.
- Vertical scaling (bigger VM) only postpones capacity limits.
- Need horizontal distribution of requests to avoid crashes and performance
  degradation.

### Pattern Overview

- **Dispatcher** sits between clients and backend workers.
- Each incoming request is routed to one of the available worker instances.
- Adding or removing workers happens transparently to clients.
- Supports fault tolerance: if an instance fails, dispatcher reroutes to healthy
  ones.

### Common Use Cases

1. **HTTP Frontend → Backend**
   - Client (browser/mobile) → Load balancer → Identical backend instances
2. **Microservices**
   - Each service group behind its own load balancer → independent scaling per
     service

### Implementation Techniques

#### 1. Cloud Load Balancing Service

- Fully managed by cloud provider (e.g., AWS ELB, GCP Cloud Load Balancing)
- Automatically scales across zones
- Supports health checks and auto-restart of balancer instances
- Can route external (client) and internal (service-to-service) traffic

#### 2. Message Broker / Distributed Queue

- Publishers enqueue messages; consumers dequeue and process
- Best for **asynchronous**, one-way communication
- Use internally between services, not for client-facing request routing
- Scales by adjusting the number of consumer workers

### Routing Algorithms

| Algorithm             | Description                                                           | When to Use                                   |
| --------------------- | --------------------------------------------------------------------- | --------------------------------------------- |
| **Round Robin**       | Distributes requests sequentially across instances                    | Stateless applications                        |
| **Session Affinity**  | Routes a client’s requests to the same instance (via cookie or IP)    | Applications that maintain server-side state  |
| **Least Connections** | Sends new requests to the instance with the fewest active connections | Long-lived connections (e.g., database, LDAP) |

### Auto Scaling Integration

- **Agents** on each instance collect CPU, memory, network metrics.
- **Auto scaling policies** adjust instance count based on thresholds.
- Load balancer stays in sync with dynamic pool of instances.
- Ensures elastic scale-out (on demand) and scale-in (to save cost).

### Summary

- Load balancing is the foundational scalability pattern for cloud systems.
- Two primary implementations: managed load-balancer service and message broker.
- Choice of routing algorithm depends on application statefulness and connection
  patterns.
- Combining load balancing with auto scaling yields highly resilient,
  cost-efficient architectures.
