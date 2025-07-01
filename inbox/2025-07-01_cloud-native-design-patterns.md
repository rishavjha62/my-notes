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

## Scatter-Gather Pattern

### Objective

Process a single client request by dispatching it to multiple workers in
parallel, then aggregating the results into a unified response.

### Pattern Structure

- **Sender (Client)**: Initiates the request.
- **Dispatcher**: Sends the request to multiple workers.
- **Workers**: Handle independent processing tasks (internal or external).
- **Aggregator**: Collects and combines responses from all workers into a single
  response.

### Key Characteristics

- Unlike Load Balancing, each worker receives the same request.
- Workers are typically **not identical** – they may vary in logic, data, or
  ownership.
- Each request is **independent and parallel**, enabling constant-time
  aggregation regardless of the number of workers.

### Common Use Cases

1. **Distributed Search Services**

   - Query is sent to multiple search workers, each responsible for a subset of
     data.
   - Aggregated and ranked results are returned to the user.

2. **External Service Integration (e.g., Hotel Booking)**

   - A single search query is sent to multiple external vendors.
   - Quotes are collected, sorted, and shown to the user.

3. **Big Data Analysis or Reporting**
   - Workers process subsets of data in parallel for long-running analysis.
   - Aggregator collects results when processing is complete.

### Benefits

- High parallelism and scalability
- Works well with internal and external systems
- User remains unaware of number or type of workers
- Constant response time despite system size

### Implementation Considerations

1. **Timeouts and Fault Tolerance**

   - Dispatcher should set a timeout for responses.
   - If some workers fail or delay, partial results can still be used.

2. **Decoupling via Message Broker**

   - Use pub/sub model to reduce tight coupling.
   - Dispatcher publishes requests; workers subscribe to topics.
   - Workers publish results to another topic; aggregator subscribes and
     compiles responses.

3. **Support for Long-Running Jobs**
   - Separate dispatcher and aggregator.
   - Assign a unique identifier to each request.
   - Workers tag results with the ID.
   - Aggregator stores result using the ID; user can poll with ID to check
     status or retrieve results.

### Summary

- Scatter-Gather enables concurrent processing of a single request across
  multiple components.
- Effective for search, aggregation, and integration tasks.
- Enhances scalability, supports partial failure, and enables both real-time and
  batch-style workflows.

## Execution Orchestration Pattern

### Objective

Coordinate complex workflows involving multiple services by introducing a
centralized orchestrator that manages the execution flow.

### Pattern Structure

- **Execution Orchestrator**:
  - Directs the flow of operations across multiple services.
  - Maintains context and state.
  - Handles retries, failures, and recovery logic.
- **Services (Microservices)**:
  - Perform domain-specific business logic.
  - Remain stateless and unaware of overall orchestration flow.
  - Can be implemented as Functions (e.g., FaaS) for cost efficiency.

### Key Characteristics

- Centralized controller (Orchestrator) that doesn’t perform business logic, but
  invokes it.
- Sequential and parallel execution of services is supported.
- Scalable and resilient via distributed deployments and state persistence.
- Especially useful in **microservices** and **serverless** environments.

### Common Use Case: User Registration in a Video Streaming Platform

1. **User Form Submission** → Orchestrator receives request.
2. **User Service** → Validates username/password.
3. **Payment Service** → Authorizes credit card.
4. **Location Service** → Associates user with a region for content access.
5. **Recommendation Service** → Prepares initial recommendation data.
6. **Email Service** → Sends welcome email with relevant details.

- All steps are coordinated by the **Execution Orchestrator**.
- Only the Orchestrator is aware of the full process.

### Benefits

- Centralized flow control in distributed architecture.
- Supports scalable, decoupled microservices.
- Easier debugging and traceability via orchestrator logs.
- Simplifies updates (add/remove services without impacting others).

### Failure & Recovery Strategies

- **Service-Level Failures**:
  - Orchestrator retries on failure.
  - Sends appropriate error messages to the client.
- **Orchestrator Failure**:

  - State persisted in a database allows recovery.
  - New orchestrator instance resumes the flow using saved state.

- **Duplicate Requests**:
  - Orchestrator handles idempotency (e.g., checking if a step has already been
    completed).

### Implementation Considerations

1. **Avoid Business Logic in Orchestrator**

   - Orchestrator should manage flow, not logic.
   - Prevents monolithic growth of the orchestrator itself.

2. **State Management**

   - Persist orchestration state to enable retries and recovery.

3. **Scalability**

   - Orchestrator can be deployed behind a load balancer like other services.

4. **Parallelism**
   - Some steps can be executed in parallel for efficiency.

### Summary

- Execution Orchestration Pattern is vital for coordinating flows in
  microservices.
- Enables independent service scaling and easy change management.
- Centralized state management allows resilience and observability.

## Choreography Pattern

### Objective

Enable microservices to collaborate on a business process without centralized
control, using asynchronous event-based communication.

### Problem Context

- System consists of **independent microservices**, each owning specific
  domains.
- Need to complete a multi-step **business transaction** across several
  microservices.
- Alternative to **Execution Orchestrator Pattern**, which introduces tight
  coupling.

### Key Characteristics

- **No central orchestrator**: Flow is driven by services emitting and
  responding to events.
- **Event-driven communication** via a **message broker** or **distributed
  message queue**.
- Services **subscribe** to relevant events and **publish** their outcomes as
  new events.
- Promotes **loose coupling**, **autonomous teams**, and **independent
  deployments**.

### Real-World Analogy

- Like a **choreographed dance**: each dancer (microservice) performs their
  steps based on music (events) and not direct instructions.

---

### Example: Job Search Platform – User Signup Flow

**Flow Steps:**

1. **Candidate fills form and uploads resume** → data sent to **Candidate
   Service**.
2. **Candidate Service** stores info → emits **"CandidateRegistered"** event.
3. **Email Function** subscribes to this event → sends confirmation email.
4. **Skills Parser Service** also consumes this event → extracts skills → emits
   **"SkillsParsed"** event.
5. **Job Search Service** consumes **"SkillsParsed"** event → finds job matches
   → emits **"JobsFound"** event.
6. **Candidate Service** updates user record with jobs.  
   **Email Service** optionally sends job list email (immediate or scheduled
   digest).

**Key Points:**

- No service knows the full flow—only their responsibilities.
- Events trigger a cascade of actions asynchronously.
- Services can come and go without impacting others.

---

### Advantages

- **Loose coupling**: Add or remove services without affecting existing ones.
- **High scalability**: Independent services scale per need.
- **Cost-effective**: Serverless-friendly—pay only on execution.
- **Flexibility**: New functionality added via subscribing to events.
- **Fault-tolerant**: Temporary service downtimes don’t lose messages (message
  broker ensures delivery when service is back).

---

### Drawbacks

- **Debugging is difficult**: No central view of the execution flow.
- **Hard to trace failures**: Asynchronous events complicate root cause
  analysis.
- **Complex integration tests** required to ensure event chains behave as
  expected.
- **Not ideal for highly interdependent, transactional flows.**

---

### Implementation Considerations

- Use **message broker** (e.g., Pub/Sub, Kafka, RabbitMQ) for reliable event
  delivery.
- Consider **dead letter queues** and **event versioning** for production
  resilience.
- Include **event correlation IDs** for better tracing.
- Start with **simpler flows** before scaling to complex chains.

---

### Summary

- **Choreography Pattern** solves the drawbacks of orchestration by removing
  central control.
- Services **collaborate through events**, enabling autonomy and agility.
- Ideal for **simple to moderate** workflows with **high scalability
  requirements**.
- Comes with **observability and testing challenges** that must be managed
  carefully.
