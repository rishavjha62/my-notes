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

## -> Scalability Patterns

### 1. Load Balancing

#### Objective

Enable systems to handle massive scale (billions of requests/day, petabytes of
data) in a cost-effective, cloud-native way by distributing work across many
instances.

#### Problem Statement

- A single server cannot sustain high traffic: CPU, memory, or network become
  bottlenecks.
- Vertical scaling (bigger VM) only postpones capacity limits.
- Need horizontal distribution of requests to avoid crashes and performance
  degradation.

#### Pattern Overview

- **Dispatcher** sits between clients and backend workers.
- Each incoming request is routed to one of the available worker instances.
- Adding or removing workers happens transparently to clients.
- Supports fault tolerance: if an instance fails, dispatcher reroutes to healthy
  ones.

#### Common Use Cases

1. **HTTP Frontend → Backend**
   - Client (browser/mobile) → Load balancer → Identical backend instances
2. **Microservices**
   - Each service group behind its own load balancer → independent scaling per
     service

#### Implementation Techniques

1. Cloud Load Balancing Service

- Fully managed by cloud provider (e.g., AWS ELB, GCP Cloud Load Balancing)
- Automatically scales across zones
- Supports health checks and auto-restart of balancer instances
- Can route external (client) and internal (service-to-service) traffic

2. Message Broker / Distributed Queue

- Publishers enqueue messages; consumers dequeue and process
- Best for **asynchronous**, one-way communication
- Use internally between services, not for client-facing request routing
- Scales by adjusting the number of consumer workers

#### Routing Algorithms

| Algorithm             | Description                                                           | When to Use                                   |
| --------------------- | --------------------------------------------------------------------- | --------------------------------------------- |
| **Round Robin**       | Distributes requests sequentially across instances                    | Stateless applications                        |
| **Session Affinity**  | Routes a client’s requests to the same instance (via cookie or IP)    | Applications that maintain server-side state  |
| **Least Connections** | Sends new requests to the instance with the fewest active connections | Long-lived connections (e.g., database, LDAP) |

#### Auto Scaling Integration

- **Agents** on each instance collect CPU, memory, network metrics.
- **Auto scaling policies** adjust instance count based on thresholds.
- Load balancer stays in sync with dynamic pool of instances.
- Ensures elastic scale-out (on demand) and scale-in (to save cost).

#### Summary

- Load balancing is the foundational scalability pattern for cloud systems.
- Two primary implementations: managed load-balancer service and message broker.
- Choice of routing algorithm depends on application statefulness and connection
  patterns.
- Combining load balancing with auto scaling yields highly resilient,
  cost-efficient architectures.

### 2. Pipes and Filters

#### Objective

Decompose data processing into independent stages (filters) connected by streams
(pipes) to enable flexibility, parallelism, and optimal resource utilization.

#### Pattern Overview

- **Filters**: Stateless components that perform a single transformation on
  data.
- **Pipes**: Channels (e.g., message queues, distributed logs, notifications to
  storage) that carry data from one filter to the next.
- **Data Source**: Origin of input (e.g., HTTP service, IoT sensor).
- **Data Sink**: Final destination after all transformations (e.g., database,
  external API).

#### Problems Addressed

- **Tight Coupling**: Monolithic pipelines force all stages into one codebase
  and language.
- **Heterogeneous Requirements**: Different stages may need specialized hardware
  or runtime (GPU for ML, high-CPU, high-memory).
- **Independent Scaling**: Each stage can scale separately based on its
  throughput needs.

#### Benefits

- Flexibility to choose best language/library per stage
- Optimal hardware provisioning per filter
- Independent scaling of each filter
- Parallel execution of stages for high throughput

#### Typical Use Cases

- **Stream Processing** (e.g., user activity, clickstream)
- **IoT Data Ingestion**
- **Media Pipelines** (image/video/audio processing)

#### Example: Video Processing Pipeline

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

#### Implementation Considerations

- **Granularity vs. Overhead**
  - Too many fine-grained filters increase operational complexity.
- **Statelessness**
  - Filters should be stateless and receive all required context in input.
- **Transactional Boundaries**
  - Not suitable for workflows requiring a single atomic transaction across
    stages.

#### Summary

- Pipes and Filters split complex processing into modular, independent
  components.
- Ideal for heterogeneous workloads and high-throughput data streams.
- Careful design needed to balance modularity with system complexity and
  transactional requirements.

### 3. Scatter-Gather Pattern

#### Objective

Process a single client request by dispatching it to multiple workers in
parallel, then aggregating the results into a unified response.

#### Pattern Structure

- **Sender (Client)**: Initiates the request.
- **Dispatcher**: Sends the request to multiple workers.
- **Workers**: Handle independent processing tasks (internal or external).
- **Aggregator**: Collects and combines responses from all workers into a single
  response.

#### Key Characteristics

- Unlike Load Balancing, each worker receives the same request.
- Workers are typically **not identical** – they may vary in logic, data, or
  ownership.
- Each request is **independent and parallel**, enabling constant-time
  aggregation regardless of the number of workers.

#### Common Use Cases

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

#### Benefits

- High parallelism and scalability
- Works well with internal and external systems
- User remains unaware of number or type of workers
- Constant response time despite system size

#### Implementation Considerations

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

#### Summary

- Scatter-Gather enables concurrent processing of a single request across
  multiple components.
- Effective for search, aggregation, and integration tasks.
- Enhances scalability, supports partial failure, and enables both real-time and
  batch-style workflows.

### 4. Execution Orchestration Pattern

#### Objective

Coordinate complex workflows involving multiple services by introducing a
centralized orchestrator that manages the execution flow.

#### Pattern Structure

- **Execution Orchestrator**:
  - Directs the flow of operations across multiple services.
  - Maintains context and state.
  - Handles retries, failures, and recovery logic.
- **Services (Microservices)**:
  - Perform domain-specific business logic.
  - Remain stateless and unaware of overall orchestration flow.
  - Can be implemented as Functions (e.g., FaaS) for cost efficiency.

#### Key Characteristics

- Centralized controller (Orchestrator) that doesn’t perform business logic, but
  invokes it.
- Sequential and parallel execution of services is supported.
- Scalable and resilient via distributed deployments and state persistence.
- Especially useful in **microservices** and **serverless** environments.

#### Common Use Case: User Registration in a Video Streaming Platform

1. **User Form Submission** → Orchestrator receives request.
2. **User Service** → Validates username/password.
3. **Payment Service** → Authorizes credit card.
4. **Location Service** → Associates user with a region for content access.
5. **Recommendation Service** → Prepares initial recommendation data.
6. **Email Service** → Sends welcome email with relevant details.

- All steps are coordinated by the **Execution Orchestrator**.
- Only the Orchestrator is aware of the full process.

#### Benefits

- Centralized flow control in distributed architecture.
- Supports scalable, decoupled microservices.
- Easier debugging and traceability via orchestrator logs.
- Simplifies updates (add/remove services without impacting others).

#### Failure & Recovery Strategies

- **Service-Level Failures**:
  - Orchestrator retries on failure.
  - Sends appropriate error messages to the client.
- **Orchestrator Failure**:

  - State persisted in a database allows recovery.
  - New orchestrator instance resumes the flow using saved state.

- **Duplicate Requests**:
  - Orchestrator handles idempotency (e.g., checking if a step has already been
    completed).

#### Implementation Considerations

1. **Avoid Business Logic in Orchestrator**

   - Orchestrator should manage flow, not logic.
   - Prevents monolithic growth of the orchestrator itself.

2. **State Management**

   - Persist orchestration state to enable retries and recovery.

3. **Scalability**

   - Orchestrator can be deployed behind a load balancer like other services.

4. **Parallelism**
   - Some steps can be executed in parallel for efficiency.

#### Summary

- Execution Orchestration Pattern is vital for coordinating flows in
  microservices.
- Enables independent service scaling and easy change management.
- Centralized state management allows resilience and observability.

### 5. Choreography Pattern

#### Objective

Enable microservices to collaborate on a business process without centralized
control, using asynchronous event-based communication.

#### Problem Context

- System consists of **independent microservices**, each owning specific
  domains.
- Need to complete a multi-step **business transaction** across several
  microservices.
- Alternative to **Execution Orchestrator Pattern**, which introduces tight
  coupling.

#### Key Characteristics

- **No central orchestrator**: Flow is driven by services emitting and
  responding to events.
- **Event-driven communication** via a **message broker** or **distributed
  message queue**.
- Services **subscribe** to relevant events and **publish** their outcomes as
  new events.
- Promotes **loose coupling**, **autonomous teams**, and **independent
  deployments**.

#### Real-World Analogy

- Like a **choreographed dance**: each dancer (microservice) performs their
  steps based on music (events) and not direct instructions.

#### Example: Job Search Platform – User Signup Flow

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

#### Advantages

- **Loose coupling**: Add or remove services without affecting existing ones.
- **High scalability**: Independent services scale per need.
- **Cost-effective**: Serverless-friendly—pay only on execution.
- **Flexibility**: New functionality added via subscribing to events.
- **Fault-tolerant**: Temporary service downtimes don’t lose messages (message
  broker ensures delivery when service is back).

#### Drawbacks

- **Debugging is difficult**: No central view of the execution flow.
- **Hard to trace failures**: Asynchronous events complicate root cause
  analysis.
- **Complex integration tests** required to ensure event chains behave as
  expected.
- **Not ideal for highly interdependent, transactional flows.**

#### Implementation Considerations

- Use **message broker** (e.g., Pub/Sub, Kafka, RabbitMQ) for reliable event
  delivery.
- Consider **dead letter queues** and **event versioning** for production
  resilience.
- Include **event correlation IDs** for better tracing.
- Start with **simpler flows** before scaling to complex chains.

#### Summary

- **Choreography Pattern** solves the drawbacks of orchestration by removing
  central control.
- Services **collaborate through events**, enabling autonomy and agility.
- Ideal for **simple to moderate** workflows with **high scalability
  requirements**.
- Comes with **observability and testing challenges** that must be managed
  carefully.

## -> Performance Patterns for Data Intensive Systems

### 1. MapReduce Architecture Pattern

#### Introduction & Motivation

- Solves the challenge of running computations on massive data by abstracting
  away:

  - Data distribution
  - Task scheduling
  - Failure recovery
  - Result aggregation

- The goal: define every computation in terms of a common model → map and reduce
  functions.
- The framework handles execution, parallelism, recovery, and output collection.

##### Use Cases

- Used for:
  - Machine learning
  - Log filtering and analysis
  - Index construction
  - Graph traversal
  - Distributed sort/search
  - Any big data batch processing

#### MapReduce Programming Model

##### Step-by-Step Flow

1. **Input:** Represented as key-value pairs (e.g., filename → file content).
2. **Map Function:**
   - Processes each key-value pair.
   - Emits intermediate key-value pairs.
3. **Shuffle:**
   - Framework groups and sorts all intermediate data by key.
4. **Reduce Function:**
   - Processes each key with a list/iterator of values.
   - Emits final output key-value pair (usually one per key).

##### Example: Word Count

- **Input:** Files with text
- **Map:** Emit (word, 1) for every word in the file
- **Shuffle:** Group all (word, 1) pairs by word
- **Reduce:** Sum values for each word → (word, total count)

#### MapReduce Software Architecture

##### Components

1. **Master:**
   - Schedules all map and reduce tasks
   - Monitors progress
   - Handles worker failures
2. **Workers:**
   - Map workers execute map() on their assigned input chunks
   - Reduce workers handle reduce() tasks post-shuffle

##### Execution Steps

- Master splits input into chunks
- Map workers process and partition their output into `R` partitions
- Reduce workers pull relevant partitions, sort/shuffle, and run reduce()

##### Parallelism

- All map tasks run in parallel
- Reduce workers start as soon as at least one map worker finishes
- All reduce tasks also run in parallel

#### Failure Handling

- **Worker Failure:**

  - Master pings workers regularly
  - Failed tasks are reassigned
  - If a map worker fails, reduce workers are informed of new intermediate data
    locations

- **Master Failure:**
  - Options:
    - Restart from scratch
    - Log snapshots and resume with new master
    - Run a backup master in parallel

#### Cloud Environment Advantages

- **Elastic compute:** Instantly scale to hundreds/thousands of machines
- **Cost-effective:** Pay only when computation runs
- **Batch pattern:** Ideal for scheduled jobs over large datasets
- **Separation of storage vs compute:** Store cheap, compute expensive

#### Real-World Usage

- Cloud vendors and open-source projects implement MapReduce
- Users only write map() and reduce() logic
- Commonly tuned for:
  - Chunk size
  - Partition function
  - Number of reducers
  - Fault tolerance

#### Summary

- MapReduce simplifies big data processing via a reusable pattern
- The architecture includes a master and many worker machines
- Code logic is limited to defining `map` and `reduce`
- Framework handles:
  - Data partitioning
  - Parallelism
  - Task orchestration
  - Failure recovery
- Ideal for batch-processing workloads in cloud environments

### 2. Saga Pattern

##### Introduction

- Microservices architecture encourages one database per microservice.
- Sharing a database between services tightly couples them and leads to a
  distributed monolith.
- Isolated databases allow independent schema changes without breaking other
  services.
- **Problem:** We lose ACID transactions across services.
- **Solution:** Saga Pattern — ensures **data consistency** in **distributed
  transactions**.

#### What is the Saga Pattern?

- A saga is a **sequence of local transactions**, each updating one
  service/database.
- After each successful operation, the next step is triggered.
- If a step fails:
  - Perform **compensating operations** to undo previous steps.
  - Either retry the failed step or abort the entire saga.

#### Saga Implementations

##### Execution Orchestrator Pattern

- A centralized **orchestration service** controls the flow.
- Calls services in order, waits for responses.
- On failure, it triggers compensating operations to undo past actions.

##### Choreography Pattern

- No centralized service.
- Services **listen to and emit events** via a message broker.
- Each service:
  - Listens to incoming events.
  - Performs its action.
  - Emits either success or compensating events depending on the result.

#### Real-Life Example: Ticketing System

##### Business Requirements

- Sell assigned seats for events.
- Ensure:
  - No blacklisted/bot users buy tickets.
  - Customer is charged.
  - Seat is reserved only once.

##### Services Involved

- `Order Service`
- `Security Service`
- `Billing Service`
- `Reservation Service`
- `Email Service`
- Optional: `Orchestration Service` (if using execution orchestrator pattern)

#### Flow: Execution Orchestrator Saga

1. **User Request → Orchestration Service**
2. **Orchestration → Order Service**

   - Create a pending order record.
   - If seat is taken, either:
     - Retry (if still pending)
     - Abort (if already purchased)

3. **Orchestration → Security Service**

   - Validate user is not a bot or blacklisted.
   - If invalid: abort saga and delete order.

4. **Orchestration → Billing Service**

   - Authorize pending transaction on user's card.
   - If insufficient funds or invalid card:
     - Abort saga and delete order.
   - On success, save pending transaction.

5. **Orchestration → Reservation Service**

   - Check if seat is already reserved.
   - If reserved:
     - Compensate:
       - Cancel billing transaction
       - Delete order
   - On success:
     - Reserve ticket
     - Update reservation DB

6. **Orchestration → Finalization Steps**
   - Bill user.
   - Mark order as purchased.
   - Send confirmation email.

#### Flow: Choreography Saga

- Same services, **no orchestration service**.
- Services communicate using **asynchronous events**:
  - e.g. `order.created`, `security.validated`, `billing.pending`, etc.
- Each service is responsible for:
  - Triggering next step.
  - Triggering compensating events if failure occurs.

#### Summary

- Saga Pattern helps maintain **data consistency** in microservices with
  separate databases.
- Supports distributed transactions using local transactions + compensations.
- Two main implementations:
  - Execution Orchestrator (central controller)
  - Choreography (event-driven flow)
- Ensures reliable rollback in case of failure and enables smooth forward flow
  on success.

### 3. Transactional Outbox Pattern

#### Objective

Guarantee that every critical database update is followed by a corresponding
event being sent to a message broker, even when services crash mid-operation.

#### Problem Statement

- In an event-driven architecture, services often:
  - Update a local database.
  - Emit an event to trigger further processing by other services.
- These two actions are **not atomic**—there’s no transaction across database
  and message broker.
- If a crash happens in between:
  - **User saved, no event sent** → rest of the system unaware.
  - **Event sent, user not saved** → system acts on invalid/incomplete data.

#### Pattern Overview

- Introduce an **Outbox table** in the same database.
- Any change to the core business table (e.g., `users`) also adds an event to
  the **Outbox**.
- Both inserts are part of the **same transaction** → guarantees atomicity.
- A separate service called the **Message Relay (or Sender)**:
  - Polls the outbox.
  - Sends events to the message broker.
  - Marks them as sent (or deletes them).

#### Benefits

- Ensures **exactly-once business logic execution** even in face of service
  crashes.
- Solves the **dual-write problem** (write to DB + send message).
- Can support **multiple message broker types** (Kafka, Pub/Sub, RabbitMQ,
  etc.).
- Can be implemented using relational or non-relational databases.

#### Potential Issues & Solutions

##### 1. Duplicate Events

- Scenario:
  - Message is sent to broker.
  - Crash happens before it's marked as "sent".
- Result: **Same event might be sent again.**

###### Solution:

- Use **at-least-once** delivery semantics.
- Add a **message ID** to each outbox entry.
- Consumers track processed IDs → ignore duplicates.
- Ensure downstream consumers are **idempotent**.

##### 2. Lack of Transaction Support

- Some NoSQL databases (e.g., document stores) **don’t support multi-table
  transactions**.

###### Solution:

- Store outbox message as a **field inside the main document**.
- Sender scans for documents with unsent outbox messages.
- After sending, remove the field or mark it as sent.

##### 3. Event Ordering

- E.g., `user_registered` followed by `user_canceled`.
- If `user_canceled` is sent before `user_registered`, consumers may reject the
  cancel event.

###### Solution:

- Add **sequence numbers** to Outbox entries.
- Sender service **sorts by sequence** before sending events.
- Guarantees **correct temporal order** of events.

#### Example Use Case

**Job Platform: New User Registration**

1. User signs up → `User Service`:
   - Inserts user into `users` table.
   - Inserts signup event into `outbox` table.
2. `Message Relay` service:
   - Sends event to message broker.
3. Downstream services (Search, Scheduler, Email):
   - Subscribe to event and take appropriate actions.

#### Summary

- **Transactional Outbox Pattern** bridges the gap between database writes and
  event emission.
- Key ideas:
  - Use a single DB transaction to write both data and message.
  - Delegate message sending to an external relay.
- Ensures strong consistency across distributed systems.
- Must handle:
  - **Duplicates** with message IDs and idempotency.
  - **Lack of DB transactions** via document-embedded messages.
  - **Ordering** via sequencing of outbox records.

### 4. Materialized View Pattern

#### Introduction

- Solves performance and cost issues when querying large datasets repeatedly.
- Complex queries (joins, aggregations) are expensive and slow.
- Increases latency for end users and leads to high cloud resource costs.
- Solution: Precompute and store results in a separate, read-optimized table
  called a **Materialized View**.

#### Pattern Overview

- Create a **read-only** table populated with the result of a complex query.
- When needed, query the materialized view instead of computing results live.
- Updates to base tables can trigger:
  - Immediate regeneration
  - Scheduled refresh (batch mode)

#### Real-World Example: Online Education Platform

- Tables: `courses`, `reviews`
- Goal: Show top courses per topic based on average rating.
- Original query: Join `courses` + `reviews`, group by course ID, compute
  average rating, filter by topic.
- This query is:
  - Costly to run repeatedly
  - Slow at scale (thousands of courses, millions of reviews)
- Solution:
  - Create a materialized view with course ID, title, topic, and average rating.
  - Optional: Create one view per topic (e.g., `programming_mv`, `art_mv`) for
    ultra-fast queries.

#### Benefits

- **Low Latency**: Read-optimized views return results faster.
- **Cost-Efficient**: Avoid recomputation of the same queries.
- **Scalability**: Supports massive reads without hitting core DB.
- **Flexibility**: Can support multiple use cases (e.g., dashboards, APIs).

#### Implementation Considerations

1. **Storage Trade-Off**

   - Uses extra space to store precomputed data.
   - Trade-off between storage cost vs. read efficiency.
   - Use selectively for high-impact queries.

2. **Where to Store**

   - **Same Database**:
     - Simple, native support in modern RDBMS (e.g., PostgreSQL, Oracle).
     - Auto-refresh on data changes using incremental updates.
   - **Separate Store** (e.g., Redis, BigQuery, Elasticsearch):
     - Optimized for reads.
     - Must manually sync from source tables.
     - Suitable for analytics or caching.

3. **Refresh Strategy**

   - **Real-time Refresh**: For highly dynamic data.
   - **Scheduled Refresh**: Lower cost, good for slowly changing datasets.
   - **Manual Refresh**: Triggered only when needed.

4. **Query Optimization**

   - Materialized views can store data:
     - Without filters (global view)
     - With filters (e.g., topic-specific views)
   - Index or pre-sort the view to support common access patterns (e.g.,
     ranking)

5. **Recovery and Redundancy**

   - Reconstructible from base tables — backup is not critical.
   - Ideal candidate for in-memory systems where durability isn’t mandatory.

#### Summary

- **Materialized View** is a performance pattern for optimizing frequent,
  complex queries.
- Saves time and cost in high-read environments.
- Works well for analytics, dashboards, and recommendation systems.
- Implementation should consider storage cost, update frequency, and view
  placement.
- Widely supported in modern databases and easily integrated into microservice
  architectures.

### 5. CQRS (Command and Query Responsibility Segregation)

#### Terminology

- **Command**: Operations that **mutate** data (insert, update, delete).
  - Examples: Add user, post review, delete comment.
- **Query**: Operations that **read** data.
  - Examples: Fetch all reviews for a product, show product details.

#### Motivation

- Traditional systems mix commands and queries in the same service and database.
- This leads to trade-offs in:
  - **Schema design**
  - **Performance tuning**
  - **Business logic complexity**
- CQRS **separates** read and write operations into:
  - Different **services**
  - Different **databases**
- Each side is optimized independently:
  - Write-optimized DB (relational, normalized)
  - Read-optimized DB (NoSQL, denormalized, precomputed)

#### Benefits

- **Separation of Concerns**:
  - Command Service: Business logic, validations, user permissions.
  - Query Service: Read-only, fast access, no business logic.
- **Independent Evolution**:
  - Each service can evolve, scale, and deploy independently.
- **Optimized Storage Models**:
  - Use schemas and storage patterns tailored to specific access patterns.
- **Scalability**:
  - Scale services and databases based on their individual workloads.
- **Improved Performance**:
  - Queries become faster and cheaper when isolated.

#### Synchronization Strategy

- Command DB → Event → Query DB
- Methods:
  1. **Message Broker**
     - Command service publishes event after DB write.
     - Query service consumes event and updates read DB.
     - Use **Transactional Outbox** pattern to guarantee atomicity.
  2. **Function as a Service**
     - Triggered on DB change.
     - Updates the read database accordingly.
     - Cost-effective for infrequent writes.

#### Consistency Model

- CQRS offers **eventual consistency** between command and query stores.
- Not suitable for cases requiring **strong consistency**.
- Make trade-offs based on:
  - Business use case
  - Tolerance for stale reads

#### Real-World Example: Online Store Reviews

##### Command Side

- Users add/edit/delete reviews.
- Relational DB:
  - Columns: user_id, product_id, order_id, review, rating
- Business logic ensures:
  - User purchased product
  - No duplicate reviews per order
  - Content filtering for inappropriate reviews

##### Query Side

- Users read:
  - Reviews
  - Average product ratings
- Read-optimized NoSQL DB:
  - Reviews collection includes:
    - user_name, review_text, rating, store_location, product_id
  - Ratings collection includes:
    - product_id, average_rating
- Querying is simple:
  - Filter by product ID
  - Data is pre-aggregated and denormalized
- Update strategies:
  - **Low review volume**: Recompute and update rating immediately.
  - **High review volume**: Defer to scheduled FaaS that recomputes ratings in
    batch.

##### Scaling

- Query service and DB can be scaled independently.
- Use autoscaling based on traffic patterns.

##### Expansion

- Add more read models:
  - Product inventory
  - Description
  - Trending products
  - Recommendations
- Each optimized and decoupled from command logic.

#### Trade-Offs & Complexity

- **Overhead**:
  - Two services
  - Two databases
  - Synchronization mechanisms
- **Maintenance**:
  - Higher dev/test/deploy complexity
- **Best used when**:
  - High performance requirements
  - Heavy read/write loads
  - Complex business logic on write path

#### Summary

- CQRS = Separation of Command and Query responsibilities.
- Write and Read paths have their own optimized:
  - Codebases
  - Databases
  - Scaling and consistency models
- Gains:
  - Performance
  - Maintainability
  - Scalability
- Costs:
  - Complexity
  - Eventual consistency
  - DevOps effort

### 6. Combining CQRS and Materialized View in Microservices

#### The Problem

- In monolithic systems, a **single relational database** allows JOINs across
  multiple tables.
- In microservices architecture:
  - Data is **split across multiple services**, each with its **own database**.
  - Cross-service JOINs are no longer possible.
  - To aggregate data:
    - Multiple **network calls** → microservices
    - **Programmatic JOIN** → aggregation in service code
  - Leads to **high latency**, **increased cost**, and **poor user experience**.

#### The Solution

- Combine:
  - **CQRS** (Command Query Responsibility Segregation)
  - **Materialized View** pattern

#### Architecture Strategy

- Create a **new query microservice**:
  - Owns a **read-optimized database**
  - Stores a **materialized view** built from data in other microservices
- Updates to the materialized view are driven by **events** published by the
  source services:
  - Use **Message Broker** (e.g., Kafka, Pub/Sub)
  - Or **Function-as-a-Service (FaaS)** triggers
- Results in a **pre-joined**, low-latency data model available for
  high-performance reads

#### Consistency Model

- Consistency is **eventual**, not strong.
- Updates from source services eventually reflect in the materialized view.

#### Real-World Example: Online Education Platform

##### System Components

- **API Gateway**: Routes external API calls.
- **Courses Microservice**:
  - Database includes course name, description, price.
- **Reviews Microservice**:
  - Database includes review text, star rating, user details.
- **Course Search Microservice**:
  - New microservice with **read-optimized DB**
  - Contains a **materialized view** with:
    - Course title, subtitle, price
    - Review count and average rating

##### Flow of Data

1. **User performs a course search**
   - API Gateway → Course Search Service
   - Returns results from **materialized view**
2. **Course updated**
   - Course Service publishes event
   - Course Search Service consumes it → updates view
3. **New review added**
   - Review Service publishes event
   - Course Search Service consumes it → updates view

##### Update Strategies

- Updates can be:
  - **Immediate**: On each data change
  - **Scheduled**: To reduce write pressure (e.g., every X minutes)

##### Benefits

- Fast, unified queries across services
- Reduced latency for UI-facing endpoints
- Cost-efficient: avoids redundant network/database operations

#### Summary

- This pattern combines **CQRS** and **Materialized View** for
  **microservices**.
- Solves the challenge of **joining distributed data** across services.
- Involves:
  - Creating a **read-optimized service**
  - Building a **pre-joined materialized view**
  - Keeping it updated via **events**
- Enables **low-latency, cost-effective** reads in a microservices environment.

### 7. Event Sourcing Pattern

#### What is Event Sourcing?

- **Event sourcing** is an architecture pattern where **state is not stored
  directly**.
- Instead of storing current values, we **store a sequence of events** that
  describe state transitions over time.
- Each **event** represents:
  - A fact or change about the system
  - **Immutability**: Once written, cannot be changed
  - **Append-only**: New events are added to the log

#### Why Not Just Store Current State?

- For most CRUD-based apps (e.g., product catalog), current state is enough.
- But in systems like **Banking**, **Inventory tracking**, **Audit trails**, etc
  just knowing the current state isn't enough. We need the **entire history** of
  how that state came to be.

#### Real-World Use Cases

- **Bank Account**:
  - Current balance = sum of all deposits and withdrawals.
  - Clients expect a **transaction history**, not just a balance.
- **Online Store Inventory**:
  - 100 items left: Was it 200 sales + 100 returns? Or 1 bulk sale?
  - Knowing the sequence matters.

#### Benefits of Event Sourcing

1. **Complete History & Auditability**

   - Every state change is captured.
   - Enables:
     - Auditing
     - Analytics
     - Fraud detection
     - Debugging issues post-fact

2. **High Write Performance**

   - Append-only events = No contention, no locking
   - Avoids conflicts in write-intensive scenarios (e.g., inventory updates)

#### How to Store Events

1. **Database Table**

   - Store each event as a row.
   - Schema:
     - `event_type`, `entity_id`, `payload`, `timestamp`
   - Benefits:
     - Easy to **query/analyze**
     - Can filter by time, type, user, etc.
   - Tradeoff:
     - Replay logic is needed to reconstruct state

2. **Message Broker** (Kafka, Pub/Sub)

   - Events are **published**, not just stored
   - Consumers **subscribe** and act (e.g., update materialized views)
   - Benefits:
     - Scalable, high-throughput
     - Good ordering guarantees
   - Tradeoff:
     - Complex querying; best used for streaming/event processing

#### Reconstructing State

##### Problem:

Replaying all events from day one is **inefficient**.

##### Solutions:

1. **Snapshots**

   - Periodically persist the current state
   - Future reads only need to replay from the **last snapshot**

2. **CQRS + Event Sourcing**

   - Combine with CQRS
   - **Command Service**:
     - Appends events to event log (e.g., broker or DB)
   - **Query Service**:
     - Listens to event stream
     - Maintains **read-optimized view** (e.g., balance, order status)

#### Bonus: In-memory caching

- Use in-memory databases for reads (e.g., Redis)

#### Eventual Consistency

- Reads might **lag behind** writes
- Suitable for most use cases, **not** real-time-critical systems

#### Summary

- **Event Sourcing** captures every change as an event.
- Enables:
  - Full history
  - Write performance
  - Analytics
- Events can be stored in:
  - Traditional DBs (easier querying)
  - Message brokers (scalable processing)
- Combine with **CQRS** for optimized reads.
- Apply **snapshots** or materialized views for fast access.
- Great fit for audit-focused and write-heavy systems.
