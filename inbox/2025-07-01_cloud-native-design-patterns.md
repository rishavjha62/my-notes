---
date: 2025-07-01
tags:
  - course
hubs:
  - "[[gcp]]"
  - "[[system-design]]"
  - "[[devops-concept]]"
urls:
  - udemy.com/course/the-complete-cloud-computing-software-architecture-patterns
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

## Pipes and Filters

### Objective

Decompose data processing into independent stages (filters) connected by streams
(pipes) to enable flexibility, parallelism, and optimal resource utilization.

### Pattern Overview

- **Filters**: Stateless components that perform a single transformation on
  data.
- **Pipes**: Channels (e.g., message queues, distributed logs, notifications to
  storage) that carry data from one filter to the next.
- **Data Source**: Origin of input (e.g., HTTP service, IoT sensor).
- **Data Sink**: Final destination after all transformations (e.g., database,
  external API).

### Problems Addressed

- **Tight Coupling**: Monolithic pipelines force all stages into one codebase
  and language.
- **Heterogeneous Requirements**: Different stages may need specialized hardware
  or runtime (GPU for ML, high-CPU, high-memory).
- **Independent Scaling**: Each stage can scale separately based on its
  throughput needs.

### Benefits

- Flexibility to choose best language/library per stage
- Optimal hardware provisioning per filter
- Independent scaling of each filter
- Parallel execution of stages for high throughput

### Typical Use Cases

- **Stream Processing** (e.g., user activity, clickstream)
- **IoT Data Ingestion**
- **Media Pipelines** (image/video/audio processing)

### Example: Video Processing Pipeline

1. **Chunking**
   - Split large video into smaller segments for streaming.
2. **Thumbnail Extraction**
   - Select a representative frame per segment.
3. **Transcoding**
   - Resize segments to multiple resolutions/bitrates for adaptive streaming.
4. **Format Encoding**
   - Encode segments into various container formats for device compatibility.
5. **Audio Processing (parallel branch)**
   - Transcribe speech → Translate captions → Generate subtitle files.
6. **Compliance Check (parallel branch)**
   - Run content-moderation algorithms → Flag or reject
     inappropriate/copyrighted content.

### Implementation Considerations

- **Granularity vs. Overhead**
  - Too many fine-grained filters increase operational complexity.
- **Statelessness**
  - Filters should be stateless and receive all required context in input.
- **Transactional Boundaries**
  - Not suitable for workflows requiring a single atomic transaction across
    stages.

### Summary

- Pipes and Filters split complex processing into modular, independent
  components.
- Ideal for heterogeneous workloads and high-throughput data streams.
- Careful design needed to balance modularity with system complexity and
  transactional requirements.
