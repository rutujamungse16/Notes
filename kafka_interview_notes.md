# Apache Kafka - Interview Preparation Notes for Java Backend Engineers
## 3.5+ Years Experience | Product Companies & MNC Backend Interviews

---

## Table of Contents
1. [High-Level Summary](#high-level-summary)
2. [Core Concepts (In Depth)](#core-concepts-in-depth)
3. [Frequently Asked Interview Questions](#frequently-asked-interview-questions)
4. [Practical Code Examples](#practical-code-examples)
5. [Mini Interview Scenarios](#mini-interview-scenarios)
6. [Optimization, Debugging & Trade-Offs](#optimization-debugging--trade-offs)
7. [Revision Cheat Sheet](#revision-cheat-sheet)

---

## High-Level Summary

### What is Apache Kafka?
Apache Kafka is a distributed event streaming platform built for real-time data pipelines and streaming applications. It acts as a publish-subscribe messaging system with persistence, allowing producers to write data to topics and consumers to read from them independently.

### Why It Matters for Backend Interviews
- **Scalability**: Fundamental to microservices and real-time data systems at scale
- **Fault Tolerance**: Demonstrates understanding of distributed systems resilience
- **Asynchronous Processing**: Core pattern in modern backend architectures
- **Interview Signal**: Knowledge of Kafka separates mid-level engineers from seniors
- **Industry Standard**: Used across fintech, e-commerce, logistics, and analytics companies

### Where It's Used in Real Projects

| Use Case | Example | Why Kafka? |
|----------|---------|-----------|
| **Event Sourcing** | Order processing → Payment → Inventory | Durability; ordering guarantees per partition |
| **Real-time Analytics** | Log aggregation, metrics streaming | High throughput; persistence for replay |
| **Microservice Integration** | Service A → Topic → Service B & C | Decoupling; fan-out without tight coupling |
| **Stream Processing** | Real-time recommendations, fraud detection | Kafka Streams; Flink integration |
| **Data Pipelines** | Database → Kafka → Data warehouse | Reliable data movement; schema evolution |

### 5-8 Key Bullet Points
- **Pub-Sub Model**: Producers publish to topics; consumers subscribe independently—no tight coupling
- **Partitioning**: Topics split into partitions for parallelism; messages with same key go to same partition (ordering guarantee)
- **Durability & Replication**: Data replicated across brokers (ISR); persisted to disk for fault tolerance and replay
- **Consumer Groups**: Multiple consumers share partitions for scalability; offset tracking enables exactly-once semantics
- **Low Latency, High Throughput**: Optimized for millions of messages/sec via batching, compression, and sequential I/O
- **Distributed Architecture**: Brokers, KRaft controller (replaces ZooKeeper), no single point of failure
- **Exactly-Once Semantics**: Transactions + idempotent producers + offset management ensure no duplicates or loss
- **Ecosystem Integration**: Spring Kafka, Kafka Streams, Schema Registry, MirrorMaker for enterprise use

---

## Core Concepts (In Depth)

### 1. Kafka Architecture & Components

#### Brokers & Cluster
- **Broker**: A Kafka server that stores topic partitions and serves producers/consumers
- **Cluster**: Multiple brokers work together; data replicated across brokers for HA
- **KRaft Controller**: Modern Kafka (4.0+) replaces ZooKeeper with Raft-based metadata management
  - Handles leader election, partition assignment, topic creation
  - No external dependency; simpler deployment

#### Topics & Partitions
- **Topic**: Logical channel for a stream of related events (e.g., `orders`, `payments`)
- **Partition**: Independent, ordered log within a topic—unit of parallelism
  - Each partition replicated across brokers (replication factor typically 3)
  - Only one partition can be assigned to one consumer per consumer group
  - Message ordering **guaranteed within a partition only**, not across partitions

#### Replication & ISR (In-Sync Replicas)
- **Replication Factor**: Number of copies of each partition (e.g., 3 means leader + 2 followers)
- **ISR**: Set of replicas that are "caught up" with the leader (in sync)
  - Leader: receives all writes; serves all reads
  - Followers: replicate from leader; can become leader if leader fails
  - Removed from ISR if it falls behind leader by `replica.lag.time.max.ms` (default: 10s)
- **Min.insync.replicas**: Minimum ISR size required for acks=all; ensures durability
  - If min.insync.replicas=2 and replication.factor=3, leader + 1 follower must ack producer
  - Prevents data loss even if one broker fails

#### Leader Election
- When a leader fails, Kafka elects a new leader from the ISR (ensures data is not lost)
- If ISR is empty and `unclean.leader.election.enable=true`, a non-ISR replica becomes leader (data loss risk)
- **Best practice**: Set `unclean.leader.election.enable=false` in production

### 2. Producers: Sending Data

#### Producer Configuration
```
acks = 0|1|all
  - 0: Fire-and-forget; no guarantee
  - 1: Leader ack; broker crash after ack = data loss (default)
  - all: All ISR replicas ack; highest durability but latency impact
  - Best practice for critical data: acks=all + retries > 1

retries = N
  - Automatic retry on transient failures (broker unavailable, timeout)
  - Only applies to retriable exceptions (not permanent errors like validation)
  - Combine with retry.backoff.ms to avoid hammering broker

linger.ms = N (milliseconds)
  - Producer waits up to N ms before sending batch
  - Allows more records to accumulate → fewer requests → higher throughput
  - Trade-off: increases end-to-end latency
  - Typical: 0-100 ms

batch.size = N (bytes)
  - Max bytes to batch before sending
  - Larger batches = higher throughput but higher latency/memory
  - Default: 16 KB; tune to 64-256 KB for throughput

compression.type = none|snappy|lz4|zstd
  - Reduces network bandwidth and storage
  - lz4 and zstd recommended for throughput; snappy for low CPU
  - Compression adds CPU overhead but saves network I/O
```

#### Partition Selection (Key-Based Routing)
- **With Key**: Hash of key determines partition → all messages with same key go to same partition (ordering guarantee)
- **Without Key**: Round-robin or default partitioner (no ordering)
- **Use Key for**:
  - Ordering: All user events to same partition
  - Aggregation: Events for same entity grouped
  - Routing: Ensure specific consumers process related data
- **Partition Key Pitfall**: Skewed keys (e.g., 80% traffic to one user) cause hot partitions

#### Error Handling in Producer
- **Retriable Errors**: Network timeouts, broker temporarily unavailable → automatic retry
- **Non-Retriable Errors**: Serialization failure, invalid topic → immediate failure
- **Callback Handling**: Use `send(record, callback)` to handle success/failure asynchronously

### 3. Consumers: Reading Data

#### Consumer Groups & Partition Assignment
- **Consumer Group**: Set of consumers that collaborate to consume a topic
  - Each partition assigned to exactly one consumer in a group
  - Enables horizontal scaling: add consumer → fewer partitions per consumer → lower latency
- **Partition Assignment Strategies**:
  - **Cooperative Sticky** (default Kafka 2.4+): Minimal partition movement during rebalancing
  - **Range Assignor**: Assigns partition ranges (can cause skew with multiple topics)
  - **Round-robin**: Even distribution
- **Rebalancing**: Triggered when consumer joins/leaves or fails → brief pause in processing
  - New strategy (Kafka 2.4+) allows some consumers to continue while others rebalance

#### Offsets & Offset Management
- **Offset**: Position in a partition's log (e.g., 0, 1, 2, ...)
- **Committed Offset**: Last offset consumer acknowledges as processed
- **Consumer Lag**: Difference between latest offset in topic and committed offset
  - High lag = consumers falling behind producers
  - Monitor via Prometheus/Grafana or Kafka CLI tools

**Offset Commit Strategies**:
```
1. Auto Commit (enable.auto.commit=true)
   - Simplest; offsets committed every 5 seconds (auto.commit.interval.ms)
   - Risk: Consumer fails after processing but before commit → message reprocessed
   - Use for: Non-critical data with tolerance for duplicates

2. Manual Sync Commit (commitSync())
   - Blocking; waits for broker acknowledgement before continuing
   - Safest but impacts throughput
   - Use for: Critical business logic where latency acceptable

3. Manual Async Commit (commitAsync())
   - Non-blocking; offsets committed in background
   - Better throughput; handle callback for errors
   - Use for: Balance between safety and performance

4. Commit Specific Offset (commitSync(offsets))
   - Commit only after processing batch of messages
   - Granular control; prevents reprocessing on failure
   - Use for: Batch processing with error handling
```

#### Isolation Levels
- **read_uncommitted** (default): Consumer reads all messages, including aborted transactions
  - Lower latency; may see uncommitted data
- **read_committed**: Consumer reads only committed messages (blocks on open transactions)
  - Safer for critical systems; small latency impact
  - Consumer waits until transaction commits before reading message

### 4. Message Delivery Semantics

| Semantic | Guarantee | Failure Scenario | When to Use |
|----------|-----------|------------------|------------|
| **At-Most-Once** | 0 or 1 delivery | Producer doesn't retry; message may be lost | Analytics, non-critical metrics |
| **At-Least-Once** | 1 or more deliveries | Producer retries; duplicates possible | Default; most use cases |
| **Exactly-Once** | Exactly 1 delivery | Idempotent producer + transactions + offset management | Financial transactions, inventory |

**Exactly-Once Implementation**:
1. **Producer**: Enable idempotence (`enable.idempotence=true`)
   - Broker deduplicates via producer ID + sequence number
   - Handles producer retries transparently
2. **Transactions** (`transactional.id`):
   - Producer batches multiple partitions into atomic transaction
   - All writes succeed or all rollback
3. **Consumer**: Manual offset commit after processing
   - Commit offset only after message successfully processed
   - If consumer crashes, unprocessed message reprocessed

### 5. Transactions (Transactional Guarantees)

#### Exactly-Once Producer-Consumer Flow
```
Producer Writes:
  - Idempotent producer enabled
  - Writes to partition with sequence number
  - Broker deduplicates retries

Consumer Reads:
  - isolation.level = read_committed (waits for transaction commit)
  - Processes message
  - Commits offset manually

If consumer crashes:
  - Offset not committed → message reprocessed
  - Idempotent processing (e.g., upsert instead of insert) handles duplicates
```

#### Transactional State
- **Ongoing**: Transaction open; writes buffered in broker
- **PrepareCommit**: Transaction coordinator writes commit marker
- **CompleteCommit**: Marker replicated to ISR; visible to consumers
- **PrepareAbort**: Consumer filters out aborted messages when reading

### 6. Kafka Streams & Stream Processing

#### When to Use Kafka Streams
- Real-time transformations, filtering, aggregations
- Stateless vs. stateful processing
- Scalable alternative to writing custom consumer code

#### KTable vs. KStream
- **KStream**: Immutable log of events; each record is independent
- **KTable**: Materialized state; each key has latest value (like a table/changelog topic)
- KTable used for joins, lookups, stateful operations

### 7. Schema Management (Schema Registry & Avro)

#### Problem: Schema Evolution
- Producer adds new field → old consumers don't know how to handle it
- Message format changes break deserialization

#### Solution: Confluent Schema Registry
- Central repository for schemas (Avro, JSON, Protobuf)
- Versioning and compatibility checking
- Producer/consumer libraries fetch schema by ID

#### Avro Example
```
Avro Schema (stored in registry):
{
  "type": "record",
  "name": "Order",
  "fields": [
    {"name": "orderId", "type": "string"},
    {"name": "amount", "type": "double"},
    {"name": "status", "type": "string"}  // New field added
  ]
}

Backward Compatibility: New field must be optional
Forward Compatibility: Old consumer can ignore new fields
```

#### Best Practices
- Define schema before producing → versioning built in
- Make new fields optional (with defaults) for backward compatibility
- Use Schema Registry for governance

### 8. Common Pitfalls & Best Practices

#### Pitfall 1: Wrong `request.timeout.ms`
- If timeout too low, producer/consumer may time out on valid requests
- Default 30 seconds; increase if network latency high
- **Fix**: Tune based on network latency + broker processing time

#### Pitfall 2: Misunderstanding Producer Retries
- `retries=0` (fire-and-forget) risks data loss
- `retries=N` without `acks=all` can still lose data on broker crash
- **Fix**: Use `acks=all` + `retries > 1` + `min.insync.replicas > 1` for critical data

#### Pitfall 3: Too Many Partitions
- Each partition = extra overhead (files, memory, rebalancing time)
- More partitions than consumers = idle consumers
- Scaling consumers beyond partitions wastes resources
- **Fix**: Calculate: `partitions >= max_expected_concurrent_consumers`
- Start conservative; increase only if throughput is bottleneck

#### Pitfall 4: Committing Offsets Before Processing
- Offset committed → consumer crashes during processing → message never reprocessed
- **Fix**: Always commit offset AFTER successful processing
- Use `enable.auto.commit=false` + manual commit in error handler

#### Pitfall 5: Not Being Idempotent
- Consumer reprocesses message (rebalance, crash, retry) → duplicate processing
- **Fix**: Design consumers to be idempotent (e.g., upsert instead of insert; check before deletion)

#### Pitfall 6: No DLT (Dead Letter Topic) Strategy
- Messages fail consistently; stuck in reprocessing loop
- **Fix**: Implement DLT for failed messages; separate monitoring/alerting

#### Pitfall 7: Ignoring Consumer Lag
- No visibility into whether consumers keep up with producers
- **Fix**: Monitor `consumer_lag_sum` metrics; alert when lag > threshold

---

## Frequently Asked Interview Questions

### 1. **What is Apache Kafka and how does it differ from traditional message queues like RabbitMQ?**

**Expected Answer (for 3.5+ years):**
- Kafka is a distributed log-based streaming platform; messages are persisted durably
- Consumers pull data (not pushed); can replay messages from any offset
- RabbitMQ: Messages deleted after consumption; push model
- Kafka: Built for high throughput, low latency, and stream processing at scale

**Key Points:**
- Kafka can handle millions of messages/sec; RabbitMQ optimized for traditional request-reply
- Kafka replicates across brokers; RabbitMQ has active-passive setup
- Kafka suited for analytics, event sourcing, log streaming; RabbitMQ for task queues

**Deep Dive:**
- Kafka uses sequential disk I/O (fast); RabbitMQ uses RAM and disk selectively
- Kafka partitions enable horizontal scaling; RabbitMQ uses queue clustering
- Kafka message retention configurable (retention.ms, retention.bytes); RabbitMQ deletes immediately

---

### 2. **Explain how Kafka achieves fault tolerance and high availability.**

**Expected Answer:**
- **Replication**: Each partition replicated across multiple brokers (default replication.factor=3)
- **ISR (In-Sync Replicas)**: Set of replicas caught up with leader
- **Leader Election**: New leader elected from ISR if current leader fails → no data loss
- **Distributed Coordination**: KRaft controller (or ZooKeeper) manages broker health and leader election

**Key Points:**
- `min.insync.replicas`: Ensures N replicas acknowledge write before acks returned to producer
- If `min.insync.replicas=2` and one broker fails, still meet durability requirement
- Producers can retry on broker failure; consumers resume from last committed offset

**Deep Dive:**
- **Scenario**: Broker holds leader partition fails
  - Broker marked unhealthy by KRaft after 6 seconds of missed heartbeats
  - New leader elected from ISR
  - Producer/consumer reconnect to new leader within seconds
  - No message loss if replicated to follower before crash

- **Rack-Aware Replication**: Replicas distributed across racks/AZs → single rack failure doesn't cause complete data loss

- **Cross-Cluster Replication**: Use MirrorMaker for disaster recovery across data centers

---

### 3. **What is a partition and why is partitioning important?**

**Expected Answer:**
- Partition is an immutable ordered log of messages within a topic
- Each partition can only be consumed by one consumer in a consumer group
- Partitioning enables parallelism: multiple consumers process different partitions concurrently
- Partitioning key determines which partition a message goes to → guarantees ordering per key

**Key Points:**
- Number of partitions = max parallelism for consumption
- Message ordering guaranteed within partition; not across partitions
- Skewed keys (e.g., 90% traffic for one key) cause hot partitions

**Deep Dive:**
- **Partition Selection**:
  - If message has key: hash(key) % num_partitions → same partition
  - If no key: round-robin or random partitioner
  - Custom partitioner can implement business logic
  
- **Scaling Decision**:
  - Increase partitions to increase throughput (more parallel consumers)
  - But: More partitions = more rebalancing overhead, more files on disk
  - Start with max(expected_consumers, 10); increase if bottleneck confirmed

---

### 4. **How do offsets work and why are they important?**

**Expected Answer:**
- Offset is a position in partition's log (0, 1, 2, ...)
- Committed offset = last message acknowledged by consumer
- Consumer can resume from last committed offset on failure or rebalance
- Enables exactly-once semantics when combined with idempotent processing

**Key Points:**
- Offset stored in internal __consumer_offsets topic (or external store)
- Consumer lag = max_offset - committed_offset
- High lag indicates consumer can't keep up with producer

**Deep Dive:**
- **Offset Management Options**:
  1. Kafka stores offsets (default) → simple but less flexible
  2. External store (database) → more control, enable read-your-own-writes semantics
  3. Manual offset management → fine-grained control for batch processing
  
- **Offset Reset**:
  - `auto.offset.reset=earliest`: Start from beginning of topic (for new consumer group)
  - `auto.offset.reset=latest`: Start from end (skip existing messages)
  - Default: throw exception if offset invalid (data loss scenario)

---

### 5. **What are consumer groups and how do they provide scalability?**

**Expected Answer:**
- Consumer group: Set of consumers collaborating to consume a topic
- Each partition assigned to one consumer per group
- Adding consumers → partitions redistributed → lower per-consumer latency
- Enables horizontal scaling without duplicating data processing

**Key Points:**
- Partition assignment strategy determines which consumer gets which partition
- Rebalancing triggered on join/leave/failure → brief pause
- Different consumer groups can consume same topic independently

**Deep Dive:**
- **Cooperative Sticky Rebalancing** (Kafka 2.4+):
  - Minimizes partition movement
  - Some consumers continue processing during rebalance (others handle rebalanced partitions)
  - Lower "stop-the-world" pause time
  - Recommended for all new applications

- **Scenario**: Scaling from 2 → 4 consumers
  - Rebalance triggered
  - Each consumer gets 1-2 partitions instead of 2-3
  - Partitions reassigned but data persists in topic
  - Consumers resume from last committed offset

---

### 6. **Explain exactly-once semantics. How is it achieved in Kafka?**

**Expected Answer:**
- Exactly-once: Each message delivered and processed exactly once; no loss, no duplicates
- Achieved via: Idempotent producer + transactions + manual offset commit
- Producer deduplicates retries; consumer commits offset only after processing

**Key Points:**
- Idempotence (`enable.idempotence=true`): Broker deduplicates via producer ID + sequence number
- Transactions: Atomically write to multiple partitions; all-or-nothing semantics
- Manual offset commit: Offset committed only after message successfully processed

**Deep Dive:**
- **Idempotent Producer Flow**:
  ```
  Producer sends msg with seq=1
  Broker writes to partition; acks
  Network fails; producer retries with seq=1
  Broker recognizes seq=1 already written → returns success without rewriting
  ```

- **Exactly-Once Transaction Flow**:
  ```
  Producer.beginTransaction()
  Producer.send(partition_A, msg1)
  Producer.send(partition_B, msg2)
  Producer.commitTransaction()
  → Both writes atomic; both visible to consumers at same time or both rolled back
  ```

- **Consumer-Side**:
  ```
  Consumer polls message
  Message processed (e.g., database update)
  Consumer manually commits offset
  If crash between processing and commit → message reprocessed
  Processing logic must be idempotent (upsert not insert)
  ```

---

### 7. **What are ISR and why does min.insync.replicas matter?**

**Expected Answer:**
- ISR: In-Sync Replicas; set of replicas caught up with leader
- Replica falls out of ISR if lag > replica.lag.time.max.ms
- min.insync.replicas: Minimum ISR size required for write to succeed (acks=all)
- Higher value = more durability; lower throughput

**Key Points:**
- Leader always part of ISR
- `min.insync.replicas=2` with `replication.factor=3` means leader + 1 follower must ack
- If `min.insync.replicas=3` and only 2 replicas in ISR → write rejected

**Deep Dive:**
- **Example Configuration** (production):
  ```
  replication.factor=3
  min.insync.replicas=2
  acks=all
  
  → Write succeeds if leader + 1 follower ack (2 out of 3)
  → Even if 1 broker dies, other 2 can serve reads/writes
  → Data persisted to at least 2 brokers before ack
  ```

- **Failure Scenario**:
  - 3 brokers; replication.factor=3; min.insync.replicas=2
  - Broker 1 (leader) fails
  - Broker 2 & 3 still in ISR; one elected new leader
  - Producer continues writing; consumers continue reading
  - Broker 1 rejoins → data automatically replicated

---

### 8. **What is a Dead Letter Topic (DLT) and how do you handle failed messages?**

**Expected Answer:**
- DLT: Separate topic for messages that fail processing repeatedly
- Messages failing all retries sent to DLT for manual review/recovery
- Prevents message loss and enables monitoring of failure patterns

**Key Points:**
- Retry policy: Configure max retries + backoff (fixed or exponential)
- DLT naming convention: `<original_topic>-dlt`
- DLT consumer reads failed messages with failure headers (exception, attempt count)

**Deep Dive:**
- **Spring Kafka Implementation**:
  ```
  @RetryableTopic(
    attempts = "3",
    backoff = @Backoff(delay = 1000, multiplier = 2),
    autoCreateTopics = "true",
    dltStrategy = DltStrategy.FAIL_ON_ERROR,
    dltTopicSuffix = "-dlt"
  )
  @KafkaListener(topics = "orders")
  public void processOrder(Order order) {
    // Process order; throw exception to trigger retry
  }
  
  @DltHandler
  public void handleOrderDlt(Order order, @Header(KafkaHeaders.EXCEPTION_MESSAGE) String msg) {
    // Log to monitoring; alert ops
  }
  ```

- **Failure Scenarios**:
  1. Network timeout on database write → retry on next attempt (recoverable)
  2. Validation error (bad data) → send to DLT (non-recoverable)
  3. Permanent database error → retry with backoff; after N attempts → DLT

---

### 9. **How do you handle schema evolution in Kafka?**

**Expected Answer:**
- Use Confluent Schema Registry to manage schemas centrally
- Producer/consumer libraries fetch schema by ID; broker stores schema metadata
- Versioning enables backward/forward compatibility
- New fields must be optional; old fields shouldn't be removed

**Key Points:**
- Avro schema with schema ID embedded in message
- Compatibility checking prevents incompatible schema changes
- Old consumer can ignore new fields (forward compatible)

**Deep Dive:**
- **Scenario**: Add new field to message
  ```
  v1 Schema: {orderId, amount, status}
  v2 Schema: {orderId, amount, status, trackingId}
  
  Old consumer (v1 reader) gets v2 message:
    - Reads orderId, amount, status
    - Ignores trackingId
    - Continues processing
  
  New consumer (v2 reader) gets v1 message:
    - Reads orderId, amount, status
    - trackingId not present; uses default value
    - Continues processing
  ```

- **Breaking Changes** (prevent):
  - Removing field without default
  - Changing field type (string → int)
  - Solution: Create new topic; migrate gradually

---

### 10. **Explain message delivery guarantees and when to use each.**

**Expected Answer (see Table in Core Concepts)**

- **At-Most-Once**: Best effort; acceptable data loss
- **At-Least-Once**: Default; risk of duplicates
- **Exactly-Once**: Idempotent producer + transactions

**Key Points:**
- Delivery guarantee trade-off: throughput vs. durability
- Most use cases: at-least-once + idempotent consumer logic

**Deep Dive:**
- **At-Most-Once Configuration**:
  ```
  acks=1, retries=0, enable.idempotence=false
  → Publish immediately; don't retry
  → If broker crashes after receiving but before acking, message lost
  ```

- **At-Least-Once Configuration**:
  ```
  acks=all, retries>1, min.insync.replicas=2, enable.idempotence=true
  → Wait for ISR to ack
  → Retry on timeout
  → Idempotence handles producer retries
  → Consumer may see duplicates on rebalance
  ```

- **Exactly-Once Configuration**:
  ```
  Producer: enable.idempotence=true, transactional.id="..."
  Consumer: isolation.level=read_committed, manual offset commit
  → Transactions ensure atomic writes
  → Manual offset commit after processing
  → Idempotent processing handles reprocessing
  ```

---

### 11. **How do you optimize Kafka for high throughput?**

**Expected Answer:**
- Increase `batch.size` (64-256 KB) to batch more records
- Increase `linger.ms` (10-100 ms) to wait for more records before sending
- Enable compression (`lz4` or `zstd`)
- Increase partitions to enable more parallel consumers
- Tune broker settings: increase `num.network.threads` and `num.io.threads`

**Key Points:**
- Throughput = messages per second
- Trade-off: latency increases with larger batches and longer linger time
- Compression trades CPU for network bandwidth

**Deep Dive:**
- **Producer Tuning**:
  ```
  batch.size=102400         # 100 KB default 16 KB
  linger.ms=50              # Wait 50 ms for batch
  compression.type=lz4      # CPU: low; compression: medium
  buffer.memory=33554432    # Increase if many partitions
  
  Result: Fewer requests; more data per request → higher throughput
  Trade-off: 50 ms added latency per message (acceptable for batch processing)
  ```

- **Consumer Tuning**:
  ```
  fetch.min.bytes=1024      # Wait for at least 1 KB before returning
  fetch.max.wait.ms=500     # Wait max 500 ms
  max.partition.fetch.bytes=1048576  # Fetch up to 1 MB per partition
  
  → Fetch fewer, larger batches → higher throughput
  ```

- **Broker Tuning**:
  ```
  num.network.threads=8     # Handle more concurrent connections
  num.io.threads=8          # More parallel disk I/O
  log.segment.bytes=1073741824  # 1 GB segments (default 1 GB; smaller = more overhead)
  ```

---

### 12. **How do you monitor and debug Kafka consumer lag?**

**Expected Answer:**
- Consumer lag = max offset in partition - committed offset by consumer group
- Monitor via Kafka CLI (`kafka-consumer-groups.sh`), Prometheus metrics, or managed services
- High lag indicates consumers can't keep up; scale up consumers or optimize processing

**Key Points:**
- Lag per partition shows which partitions are backlogged
- Time-based lag (milliseconds behind) more intuitive than offset-based lag
- Alert when lag exceeds SLA (e.g., lag > 100,000 messages or > 5 minutes behind)

**Deep Dive:**
- **Debugging Slow Consumers**:
  1. Check lag by partition: `kafka-consumer-groups.sh --group <group> --describe`
  2. Is lag skewed to one partition? → Hot partition (skewed key)
  3. Is lag uniform across partitions? → Consumer processing slow; check logs, CPU, GC
  4. Is lag increasing? → Producer throughput exceeds consumer throughput; need more consumers

- **Monitoring Setup**:
  ```
  Prometheus metrics (JMX):
  - kafka_consumer_fetch_total_bytes_consumed_total
  - kafka_consumer_records_lag_max
  - kafka_consumer_records_lag_sum
  
  Alert rules:
  - records_lag_max > 100000 for 5 minutes
  - (max_offset - committed_offset) > threshold
  
  Grafana dashboard:
  - Lag trend (over time)
  - Lag by partition
  - Consumer throughput (records/sec)
  ```

---

### 13. **What happens during consumer group rebalancing and how can you minimize its impact?**

**Expected Answer:**
- Rebalancing: Triggered when consumer joins/leaves/fails; partitions reassigned
- During rebalance: Consumers stop processing → "stop-the-world" pause
- Minimize impact: Use cooperative sticky assignment; design idempotent handlers

**Key Points:**
- Rebalance time depends on rebalance timeout + processing time
- Each rebalance = brief spike in latency; potential data loss if not careful
- Consumer can pause consumption before rebalance (graceful shutdown)

**Deep Dive:**
- **Rebalancing Scenarios**:
  1. Consumer joins:
     - Group coordinator notifies members
     - All consumers stop, submit partition assignments
     - Group leader computes new assignments (using strategy)
     - Consumers resume with new partitions
  
  2. Consumer crashes:
     - Session timeout (6 seconds default) expires
     - Coordinator removes consumer from group
     - Rebalance triggered; remaining consumers reallocate partitions
  
  3. Consumer leaves gracefully:
     - Consumer sends LeaveGroup message
     - Immediate rebalance; minimal impact

- **Cooperative Sticky Strategy (Kafka 2.4+)**:
  ```
  Old strategy (Eager):
  - All consumers revoke all partitions
  - Compute new assignment
  - All consumers take new partitions
  - Downtime: sum of all pause times
  
  Cooperative Sticky:
  - Only affected consumers revoke partitions
  - Unaffected consumers continue processing
  - Compute new assignment (minimal movement)
  - Affected consumers take new partitions
  - Downtime: only for affected consumers + coordination time
  
  Result: ~50% reduction in rebalance pause
  ```

- **Best Practices**:
  - `session.timeout.ms=45000` (increase from 10s to avoid false positives)
  - `heartbeat.interval.ms=3000` (heartbeat more frequently)
  - Use cooperative sticky assignment (Spring Kafka default)
  - Implement `onPartitionsRevoked()` to gracefully flush state

---

### 14. **How do you ensure no message loss in a Kafka pipeline?**

**Expected Answer:**
- **Producer**: `acks=all` + `retries > 1` + `min.insync.replicas > 1` + `enable.idempotence=true`
- **Broker**: Replication factor ≥ 3; enable `unclean.leader.election.enable=false`
- **Consumer**: Manual offset commit after processing; idempotent message handling

**Key Points:**
- Message loss can occur at producer, broker, or consumer stage
- No single configuration; requires all three layers to align

**Deep Dive:**
- **Producer-Side Protection**:
  ```
  acks=all → Wait for all ISR replicas to ack
  retries=3 → Retry on transient failures
  max.in.flight.requests.per.connection=5 → Allow pipelining (with enable.idempotence)
  enable.idempotence=true → Deduplicate producer retries
  
  Scenario: Producer sends msg; broker acks; network fails before client receives ack
  → Producer retries (thinks it failed)
  → Idempotence deduplicates on broker
  → Message reaches topic once
  ```

- **Broker-Side Protection**:
  ```
  replication.factor=3
  min.insync.replicas=2
  unclean.leader.election.enable=false
  
  Scenario: Leader + 1 follower have message; leader crashes
  → Non-ISR replica cannot become leader (even if only survivor)
  → ISR replica (has message) becomes leader
  → Message not lost
  ```

- **Consumer-Side Protection**:
  ```
  enable.auto.commit=false
  Manual commitSync() after processing
  
  Scenario: Consumer processes message; crashes before commit
  → Offset not committed
  → Consumer resumes from last committed offset
  → Message reprocessed
  → Processing logic must be idempotent (upsert; check before delete)
  ```

---

### 15. **What is the relationship between `replication.factor` and `min.insync.replicas`?**

**Expected Answer:**
- `replication.factor`: Number of copies of each partition
- `min.insync.replicas`: Minimum replicas that must ack for producer to consider write successful
- `min.insync.replicas <= replication.factor`; typically: min.insync.replicas = replication.factor / 2 + 1

**Key Points:**
- Higher replication.factor = more durability + more storage + more replication lag
- Higher min.insync.replicas = more durability + lower throughput (wait for more acks)
- Balance: durability vs. performance

**Deep Dive:**
- **Example Configurations**:
  ```
  Config 1 (High Availability):
  replication.factor=3, min.insync.replicas=2
  → 3 copies; writes ack'd when 2 replicas have it
  → Can lose 1 broker; still meet durability guarantee
  → Can handle 1 broker failure + continue writes
  
  Config 2 (Maximum Durability):
  replication.factor=3, min.insync.replicas=3
  → 3 copies; writes ack'd only when all 3 have it
  → Any broker failure stops writes (until rejoin)
  → Rarely used (too low throughput)
  
  Config 3 (Low Durability, High Throughput):
  replication.factor=3, min.insync.replicas=1
  → 3 copies; writes ack'd when leader has it
  → Follower replication happens asynchronously
  → Broker crash = potential data loss (if crash before followers replicate)
  → Used for non-critical data
  ```

- **Recommendation**: `replication.factor >= 3` + `min.insync.replicas = 2` for production critical data

---

## Practical Code Examples

### Example 1: Spring Boot Kafka Producer with Error Handling

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, Order> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        
        // Durability configuration
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        config.put(ProducerConfig.RETRIES_CONFIG, 3);
        config.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);
        config.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        
        // Performance tuning
        config.put(ProducerConfig.BATCH_SIZE_CONFIG, 102400); // 100 KB
        config.put(ProducerConfig.LINGER_MS_CONFIG, 50);
        config.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
        
        // Timeouts
        config.put(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG, 30000);
        
        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, Order> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

@Component
public class OrderProducer {
    
    private final KafkaTemplate<String, Order> kafkaTemplate;
    
    @Autowired
    public OrderProducer(KafkaTemplate<String, Order> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
    
    public void publishOrder(Order order) {
        String topic = "orders";
        String key = order.getCustomerId(); // Partition by customer ID
        
        ListenableFuture<SendResult<String, Order>> future = 
            kafkaTemplate.send(topic, key, order);
        
        // Async callback for error handling
        future.addCallback(
            result -> {
                // Success
                SendResult<String, Order> sendResult = result;
                logger.info("Order sent successfully: topic={}, partition={}, offset={}",
                    sendResult.getRecordMetadata().topic(),
                    sendResult.getRecordMetadata().partition(),
                    sendResult.getRecordMetadata().offset());
            },
            ex -> {
                // Failure
                logger.error("Failed to send order: {}", order, ex);
                // Implement retry logic, DLT routing, or alerting
                handlePublishingError(order, ex);
            }
        );
    }
    
    private void handlePublishingError(Order order, Throwable ex) {
        // Implement custom error handling:
        // - Retry with exponential backoff
        // - Send to DLT
        // - Alert monitoring system
        // - Store in database for manual review
    }
}
```

**What This Demonstrates:**
- Producer configuration for durability (`acks=all`, idempotence enabled)
- Batching for throughput (`batch.size`, `linger.ms`, compression)
- Async callback pattern for error handling
- Partition key selection (customer ID ensures ordering per customer)

**Interview Questions:**
- Why use `enable.idempotence=true`?
  → Deduplicates retries; ensures exactly-once semantics at producer level
- What happens if the callback's error handler itself throws exception?
  → Implement circuit breaker; log and move to DLT to prevent message loss
- How would you implement retry with exponential backoff?
  → Use Spring Retry (`@Retryable`) or manual backoff in callback

---

### Example 2: Spring Kafka Consumer with Manual Offset Commit & DLT

```java
@Configuration
@EnableKafka
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, Order> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processing-group");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        config.put(JsonDeserializer.VALUE_DEFAULT_TYPE, Order.class.getName());
        
        // Offset management
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // Manual commit
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        
        // Isolation level for exactly-once
        config.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
        
        // Rebalancing
        config.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 45000);
        config.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000);
        config.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG, 
            "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
        
        return new DefaultKafkaConsumerFactory<>(config);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Order> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Order> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 3 concurrent listeners
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
        
        // Error handler with DLT
        DefaultErrorHandler errorHandler = new DefaultErrorHandler(
            deadLetterPublishingRecoverer(),
            new FixedBackOff(1000, 3) // 1 sec delay, 3 retries
        );
        errorHandler.addRetryableExceptions(TemporaryDataAccessException.class);
        errorHandler.addNotRetryableExceptions(DataIntegrityViolationException.class);
        
        factory.setCommonErrorHandler(errorHandler);
        return factory;
    }

    @Bean
    public DeadLetterPublishingRecoverer deadLetterPublishingRecoverer() {
        KafkaTemplate<String, Order> kafkaTemplate = new KafkaTemplate<>(
            new DefaultKafkaProducerFactory<>(Map.of(
                ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
                ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class,
                ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class
            ))
        );
        
        return new DeadLetterPublishingRecoverer(
            kafkaTemplate,
            (record, ex) -> new TopicPartition(record.topic() + "-dlt", record.partition())
        );
    }
}

@Component
public class OrderProcessor {
    
    private final OrderService orderService;
    private final KafkaTemplate<String, Order> kafkaTemplate;
    
    @KafkaListener(
        id = "order-processing-listener",
        topics = "orders",
        groupId = "order-processing-group",
        containerFactory = "kafkaListenerContainerFactory",
        concurrency = "3"
    )
    public void processOrder(
            Order order,
            @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment ack) {
        
        try {
            logger.info("Processing order: id={}, partition={}, offset={}",
                order.getId(), partition, offset);
            
            // Business logic: process order (e.g., payment, inventory)
            orderService.processPayment(order);
            orderService.updateInventory(order);
            
            // Commit offset only after successful processing
            ack.acknowledge();
            logger.info("Order processed and offset committed: id={}, offset={}",
                order.getId(), offset);
            
        } catch (TemporaryDataAccessException ex) {
            // Retriable error (e.g., database connection timeout)
            logger.warn("Temporary error processing order: {}, will retry", order.getId(), ex);
            throw ex; // Re-throw to trigger retry
            
        } catch (DataIntegrityViolationException ex) {
            // Non-retriable error (e.g., duplicate order ID)
            logger.error("Non-retriable error processing order: {}", order.getId(), ex);
            // Move to DLT via error handler; don't re-throw
            
        } catch (Exception ex) {
            logger.error("Unexpected error processing order: {}", order.getId(), ex);
            throw new RuntimeException("Order processing failed", ex);
        }
    }
    
    @KafkaListener(topics = "orders-dlt", groupId = "order-dlt-group")
    public void handleOrderDlt(
            Order order,
            @Header(KafkaHeaders.EXCEPTION_MESSAGE) String exceptionMessage,
            @Header(KafkaHeaders.EXCEPTION_STACKTRACE) String stacktrace) {
        
        logger.error("Order failed and sent to DLT: id={}, exception={}", 
            order.getId(), exceptionMessage);
        
        // Implement DLT handling:
        // - Alert ops team
        // - Store in monitoring database
        // - Trigger manual review workflow
        // - Optionally retry after investigation
    }
}
```

**What This Demonstrates:**
- Manual offset commit after processing (exactly-once semantics)
- Error handler with DLT routing
- Retry logic with fixed backoff and retriable/non-retriable exceptions
- Cooperative sticky assignment for minimal rebalance impact
- Isolation level for transactional consistency

**Interview Questions:**
- Why use `ENABLE_AUTO_COMMIT_CONFIG=false`?
  → Ensures offset committed only after processing succeeds; prevents message loss
- What's the difference between retriable and non-retriable exceptions?
  → Retriable: temporary network/DB issues; automatically retried. Non-retriable: validation errors; sent to DLT immediately
- How does manual offset commit ensure exactly-once?
  → Consumer crashes before commit → offset not persisted → message reprocessed. Processing logic must be idempotent
- What happens if consumer takes > 45 seconds to process a message?
  → Session timeout expires; consumer considered dead; rebalance triggered; partition reassigned to other consumer

---

### Example 3: Idempotent Message Processing

```java
@Component
public class IdempotentOrderProcessor {
    
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    
    /**
     * Idempotent order processing using database upsert pattern.
     * If message reprocessed (due to rebalance/crash), database naturally handles duplicate.
     */
    @KafkaListener(topics = "orders", groupId = "idempotent-group")
    public void processOrderIdempotent(Order order, Acknowledgment ack) {
        try {
            // Step 1: Check if order already processed (idempotency key)
            Optional<OrderProcessingRecord> existing = 
                orderRepository.findByOrderIdAndStatus(order.getId(), "PAID");
            
            if (existing.isPresent()) {
                logger.info("Order already processed: {}, skipping", order.getId());
                ack.acknowledge();
                return;
            }
            
            // Step 2: Process payment (idempotent: can call multiple times safely)
            PaymentResult paymentResult = paymentService.chargeCard(
                order.getCustomerId(),
                order.getAmount(),
                order.getId() // Idempotency key: same order ID → same charge
            );
            
            // Step 3: Upsert order processing record (insert or update)
            OrderProcessingRecord record = new OrderProcessingRecord();
            record.setOrderId(order.getId());
            record.setStatus("PAID");
            record.setPaymentId(paymentResult.getPaymentId());
            record.setProcessedAt(LocalDateTime.now());
            
            // Database constraint: UNIQUE on (order_id) ensures only one successful processing
            orderRepository.saveIfNotExists(record);
            
            // Step 4: Commit offset only after database write
            ack.acknowledge();
            logger.info("Order processed and persisted: {}", order.getId());
            
        } catch (Exception ex) {
            logger.error("Error processing order: {}", order.getId(), ex);
            throw new RuntimeException("Processing failed; will retry", ex);
        }
    }
}

/**
 * Repository with saveIfNotExists logic (idempotent).
 * Tries to insert; if duplicate key, updates instead.
 */
@Repository
public class OrderRepository extends JpaRepository<OrderProcessingRecord, Long> {
    
    @Transactional
    public void saveIfNotExists(OrderProcessingRecord record) {
        try {
            // Try to insert
            this.save(record);
        } catch (DataIntegrityViolationException ex) {
            // Duplicate key: order already processed
            // No-op: record unchanged; idempotency preserved
            logger.info("Order already exists in DB: {}", record.getOrderId());
        }
    }
}
```

**What This Demonstrates:**
- Idempotency pattern: check if already processed before executing
- Database constraint (UNIQUE) ensures only one successful processing
- Upsert pattern for idempotent data updates
- Offset commit only after database persistence
- Handling duplicate messages transparently

**Interview Questions:**
- Why is idempotency important for exactly-once semantics?
  → Messages can be reprocessed (consumer restart, rebalance); logic must handle duplicates transparently
- How does database UNIQUE constraint help with idempotency?
  → Ensures only one processing succeeds; other duplicate attempts fail/skip gracefully
- What if payment service API is not idempotent?
  → Charge same order twice; use order ID as idempotency key on payment service side too

---

### Example 4: Spring Kafka Configuration with Properties

```java
// application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
      properties:
        linger.ms: 50
        batch.size: 102400
        compression.type: lz4
        max.in.flight.requests.per.connection: 5
        enable.idempotence: true
    consumer:
      bootstrap-servers: localhost:9092
      group-id: order-processing-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false
      properties:
        session.timeout.ms: 45000
        heartbeat.interval.ms: 3000
        isolation.level: read_committed
        partition.assignment.strategy: org.apache.kafka.clients.consumer.CooperativeStickyAssignor
    listener:
      ack-mode: manual
      concurrency: 3

// ConfigurationProperties class
@Configuration
@ConfigurationProperties(prefix = "spring.kafka")
public class KafkaCustomConfig {
    // Auto-loaded from application.yml
}
```

**What This Demonstrates:**
- Spring Boot externalized configuration
- Producer/consumer separation
- Durability, performance, and offset commit settings
- Easy environment-specific tuning

---

### Example 5: Integration Test with Embedded Kafka

```java
@SpringBootTest
@EmbeddedKafka(
    partitions = 1,
    topics = {"orders", "orders-dlt"},
    brokerProperties = {
        "log.dir=/tmp/kafka-logs"
    }
)
class OrderProcessorIntegrationTest {
    
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;
    
    @Autowired
    private OrderRepository orderRepository;
    
    private BlockingQueue<Order> receivedOrders = new LinkedBlockingQueue<>();
    
    @Test
    void testOrderProcessingEndToEnd() throws InterruptedException {
        // Arrange
        Order order = new Order();
        order.setId(UUID.randomUUID().toString());
        order.setCustomerId("customer-123");
        order.setAmount(BigDecimal.valueOf(99.99));
        
        // Act: Publish message to Kafka
        kafkaTemplate.send("orders", order.getCustomerId(), order);
        
        // Assert: Wait for message to be processed
        boolean processed = receivedOrders.poll(5, TimeUnit.SECONDS) != null;
        assertTrue(processed, "Order not processed within 5 seconds");
        
        // Verify database state
        Optional<OrderProcessingRecord> record = 
            orderRepository.findByOrderId(order.getId());
        assertTrue(record.isPresent(), "Order not persisted");
        assertEquals("PAID", record.get().getStatus());
    }
    
    @Test
    void testOrderProcessingWithRetry() throws InterruptedException {
        // Test that failed messages are retried
        Order order = new Order();
        order.setId("retry-test-order");
        order.setAmount(BigDecimal.valueOf(0)); // Trigger error
        
        kafkaTemplate.send("orders", order.getCustomerId(), order);
        
        // Expect message in DLT after retries exhausted
        boolean failedMessage = receivedOrders.poll(10, TimeUnit.SECONDS) != null;
        // assertTrue(failedMessage, "Message not sent to DLT");
    }
}
```

**What This Demonstrates:**
- Using `@EmbeddedKafka` for lightweight testing (no Docker)
- Publishing and asserting message processing
- Verifying database state after Kafka message consumption
- Testing retry and DLT behavior

---

## Mini Interview Scenarios

### Scenario 1: Real-Time Payment Processing with Exactly-Once Semantics

**Setting:**
Your team builds a payment service. Orders come in via REST API and must be processed exactly once. Each order contains: orderId, customerId, amount. You need to:
1. Charge the customer's credit card once
2. Update inventory
3. Send confirmation email

**Problem:** Network failures, service crashes, and Kafka rebalancing are common. How do you ensure payment charged exactly once?

**Strong 3.5-year Engineer's Answer:**

"I'd use exactly-once semantics with Kafka:

**Producer Side (API):**
- Publish order to `orders` topic with `enable.idempotence=true`
- Use `acks=all` to ensure broker replication before returning to user
- Set `min.insync.replicas=2` so at least 2 brokers have the order

**Consumer Side (Payment Service):**
- Enable `enable.auto.commit=false` (manual offset commit)
- Set `isolation.level=read_committed` to only read committed messages
- Process order: charge payment using orderId as idempotency key on payment gateway
  - Payment gateway stores idempotency key; if same orderId sent twice, returns same payment result
- Update inventory with upsert (insert if not exists, update if exists) to handle reprocessing
- Commit offset only after database writes succeed

**Failure Scenarios:**
1. Consumer crashes after charging but before offset commit:
   → Kafka rebalance; message reprocessed
   → Payment gateway recognizes same orderId; returns cached result (no double charge)
   → Inventory update is idempotent upsert; no duplicate record

2. Network failure between broker and consumer:
   → Broker keeps message in partition
   → Consumer resumes from last committed offset
   → Message reprocessed (handled by payment gateway idempotency)

3. Payment gateway fails during charge:
   → Exception thrown; offset not committed
   → Consumer retries after backoff
   → Eventually succeeds or moved to DLT

**Configuration:**
```
Producer: acks=all, retries=3, enable.idempotence=true, min.insync.replicas=2
Consumer: enable.auto.commit=false, isolation.level=read_committed, manual offset commit
```

This ensures that even with any number of failures, each payment is charged exactly once."

**Deep Dive Questions the Interviewer Might Ask:**
- Q: What if the payment gateway itself crashes after charging but before returning idempotency key?
  → A: Payment is still charged (money debited). When service recovers, check payment status via inquiry API before retrying. Handle reconciliation.

- Q: What if consumer processes 100 messages, crashes after 50 before offset commit?
  → A: All 100 reprocessed; first 50 have payment gateway recognize idempotency key (cached result). Second 50 processed normally. No duplicate charges.

- Q: How do you avoid long reprocessing if consumer stuck?
  → A: Implement timeout; if processing takes > 5 minutes, move to DLT. Alert ops. Add circuit breaker on payment gateway calls.

---

### Scenario 2: Real-Time Analytics Pipeline with Consumer Lag Spike

**Setting:**
Your analytics team has a Kafka pipeline: `events` topic → Consumer → Elasticsearch → Dashboard.

One morning, alerts trigger: consumer lag jumped from 10,000 messages to 500,000 messages in 1 minute. The pipeline is now 5 minutes behind real-time (SLA: < 1 minute).

**Problem:** Identify root cause and recover quickly.

**Strong 3.5-year Engineer's Answer:**

"I'd debug systematically:

**Step 1: Verify Consumer Health**
- Check consumer group status: `kafka-consumer-groups.sh --group analytics-consumer --describe`
- Is lag uniform across all partitions or skewed to one?
  - If skewed: hot partition (skewed event key) → hard to scale
  - If uniform: consumer likely slow → CPU/memory/GC issue

**Step 2: Check Producer Throughput**
- Compare producer throughput (events/sec) vs. consumer throughput
- If producer throughput spiked (e.g., marketing campaign triggered event storm):
  → Consumer can't keep up; need to scale
- If producer normal: consumer bottleneck

**Step 3: Diagnose Consumer Bottleneck**
- Check consumer logs for errors (database timeout, API call hanging)
- Check consumer JVM GC logs: long GC pauses? (> 100ms)
- Check downstream dependency (Elasticsearch): index taking long? Search slow?

**Immediate Actions to Recover:**
1. Increase consumer concurrency if not already at partition count:
   `concurrency = min(current_concurrency * 2, num_partitions)`
   
2. Add more consumer instances (if concurrency limited by partitions):
   → New instances trigger rebalance
   → Partitions redistributed
   → More parallel processing

3. Reduce processing load temporarily:
   → Batch messages (increase fetch.min.bytes, fetch.max.wait.ms)
   → Reduce transformations (skip enrichment if necessary)
   → Cache Elasticsearch lookups

4. Scale downstream (Elasticsearch):
   → Add replicas to index
   → Increase shard count
   → Check query performance

**Example Configuration Change:**
```java
factory.setConcurrency(6); // Increase from 3
containerProperties.setPollTimeout(5000); // Increase batch size
```

**Root Cause Analysis (After Recovery):**
- If event volume genuinely increased: adjust baseline consumer count upward
- If consumer code changed: profile with Async Profiler; identify bottleneck
- If downstream dependency slow: optimize queries or add caching
- If garbage collection issue: tune JVM heap, use low-latency GC (ZGC, Shenandoah)

**Prevention:**
- Set up proactive lag alerting: lag > 50,000 for 2 minutes → alert before SLA breach
- Auto-scaling: trigger new consumer instance when lag increases
- Load testing: simulate traffic spikes; verify pipeline handles peak
"

**Deep Dive Questions:**
- Q: What if you add consumers but lag doesn't decrease?
  → A: Likely downstream bottleneck (Elasticsearch). Check if index can ingest faster. Or consumer code single-threaded (blocked on I/O); optimize with async calls.

- Q: How do you know if hot partition is the issue?
  → A: Compare lag by partition: `kafka-consumer-groups.sh --describe`. If one partition lag >> others, that's hot partition. Solution: repartition on different key or increase partition count (may require topic recreation).

---

### Scenario 3: Data Consistency Issue with Transactional Writes

**Setting:**
Your e-commerce platform has:
1. Order service: publishes `OrderCreated` event to `orders` topic
2. Inventory service: subscribes to `orders` and decrements stock via database

**Problem:** A customer creates order, event published to Kafka, but inventory service crashes before updating inventory. Order goes through; stock not decremented. Customer gets item that doesn't exist.

**Question:** How do you ensure inventory updated atomically with order creation?

**Strong 3.5-year Engineer's Answer:**

"There are two patterns depending on architecture:

**Pattern 1: Transactional Outbox (Recommended)**
Order service uses two transactions:
1. Within database transaction:
   - Insert order record
   - Insert row in `outbox` table with event data (same DB transaction)
   - Commit both atomically

2. Separate async process:
   - Polls outbox table
   - Publishes event to Kafka
   - Only deletes outbox record after successful publish
   - If crash, outbox record persists; retry on restart

Guarantees: Kafka always publishes event if order created; never publishes without order in DB.

**Pattern 2: Kafka Transactions (Producer-Side)**
Order service uses Kafka producer transactions:
```java
producer.beginTransaction();
producer.send(ordersTopicRecord);
producer.send(inventoryTopicRecord); // Both go to Kafka atomically
producer.commitTransaction();
```

All-or-nothing: both messages visible to consumers or neither.

**Inventory Service Side:**
- Set `isolation.level=read_committed`
- Consumer only reads committed messages
- Processes atomically: if crash during processing, message reprocessed
- Makes inventory update idempotent (upsert or check-before-update)

**Example:**
```java
// Order Service
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order); // DB transaction
    outboxRepository.save(new OutboxEvent(\"OrderCreated\", order)); // Same transaction
    // On commit: event visible to outbox poller
}

// Inventory Service
@KafkaListener(topics = \"orders\", groupId = \"inventory-group\")
public void handleOrderCreated(Order order) {
    // Idempotent update: use ON CONFLICT DO UPDATE
    inventoryRepository.decrementStock(order.getItemId(), order.getQuantity());
    // If reprocessed: stock unchanged (already decremented)
}
```

**Guarantees:**
- Order created → event eventually published to Kafka
- Event published → inventory service eventually processes
- No data loss; no skipped updates

**Failure Scenarios:**
1. Network fails between order DB commit and Kafka publish:
   → Outbox poller retries → event published → inventory updated ✓

2. Inventory service crashes during stock update:
   → Stock not updated (transaction rolled back)
   → Consumer resumes from last committed offset (no offset change if manual commit)
   → Message reprocessed → stock updated ✓

3. Kafka broker fails after order event published but before inventory service reads:
   → Event replicated to other brokers (replication.factor=3)
   → Inventory service reconnects → reads event from new broker ✓
"

**Deep Dive:**
- Q: What if outbox poller itself crashes repeatedly?
  → A: Implement exponential backoff; DLT for events that fail to publish after N attempts. Alert ops.

- Q: What if customer simultaneously creates order and views inventory?
  → A: Read-your-own-writes: cache stock in session; or read from database with lock if critical. Kafka update async; GUI shows stale data briefly (acceptable).

---

## Optimization, Debugging & Trade-Offs

### Performance Optimization

#### Throughput vs. Latency Trade-off

| Optimization | Throughput Impact | Latency Impact | When to Use |
|---|---|---|---|
| Increase `batch.size` (16KB → 100KB) | ↑ 3-5x | ↑ 50-100ms | Bulk analytics, log aggregation |
| Increase `linger.ms` (0 → 100ms) | ↑ 2-3x | ↑ 100ms | Batch processing |
| Enable compression (none → lz4) | ↑ 1.5-2x | ↑ 5-10ms CPU | Network-bound workloads |
| Increase `fetch.min.bytes` (1B → 1KB) | ↑ throughput | ↑ latency per fetch | Real-time not critical |
| Increase consumer concurrency | ↑ parallelism | ← stable | Bottleneck in per-partition processing |

**Tuning Strategy:**
1. Profile: Identify bottleneck (CPU, network, disk, downstream system)
2. Target optimization to bottleneck
3. Measure: Track throughput, latency, resource utilization
4. Iterate

#### Common Bottlenecks & Solutions

| Bottleneck | Symptom | Solution |
|---|---|---|
| **Network I/O** | High bytes/sec; low msgs/sec; network util > 80% | Enable compression; increase batch size; add network capacity |
| **Disk I/O** | Broker disk util high; fsync slow | Use SSD; tune `log.flush.interval.ms`; increase replication factor carefully |
| **CPU (Broker)** | Broker CPU > 80%; serialization slow | Add brokers; optimize serialization; reduce client connections |
| **CPU (Consumer)** | Consumer CPU high; processing slow | Optimize processing logic; parallelize; reduce complex transformations |
| **Memory** | GC pauses > 100ms; heap util > 90% | Tune JVM GC (ZGC, Shenandoah for low-latency); increase heap carefully |
| **Downstream** | Lag increasing; broker metrics normal | Scale downstream system (Elasticsearch, DB, API); add caching; batch writes |

### Debugging Techniques

#### Consumer Not Processing Messages

```bash
# Step 1: Check if messages exist
kafka-console-consumer --bootstrap-servers localhost:9092 --topic orders --from-beginning

# Step 2: Check consumer group
kafka-consumer-groups --bootstrap-servers localhost:9092 --group order-processor --describe
# Output: Should show all partitions assigned to consumers

# Step 3: Check lag
# If LAG is high and increasing: consumer can't keep up
# If LAG is high but stable: consumer paused or stuck

# Step 4: Check consumer logs
tail -f application.log | grep "order-processor"
# Look for errors, exceptions, slow processing

# Step 5: Check offset position
kafka-consumer-groups --bootstrap-servers localhost:9092 --group order-processor --describe
# CURRENT-OFFSET vs LOG-END-OFFSET shows progress

# Step 6: Reset consumer group (if needed; careful!)
kafka-consumer-groups --bootstrap-servers localhost:9092 --group order-processor --reset-offsets --to-earliest --execute
```

#### Producer Messages Not Reaching Topic

```bash
# Step 1: Verify producer configuration
# Check: bootstrap-servers, acks, retries

# Step 2: Check if topic exists
kafka-topics --bootstrap-servers localhost:9092 --list | grep orders

# Step 3: Monitor producer metrics (JMX)
# batch.size, bytes.sent, records.sent, errors

# Step 4: Enable debug logging
log4j.logger.org.apache.kafka.clients.producer=DEBUG

# Step 5: Check broker logs
tail -f /var/log/kafka/server.log | grep -i error
```

#### High Consumer Lag Diagnosis

```bash
# Check lag by partition
kafka-consumer-groups --bootstrap-servers localhost:9092 --group order-processor --describe

# Example output:
# TOPIC      PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# orders     0          100             1000            900
# orders     1          500             500             0
# orders     2          300             2000            1700

# Analysis:
# Partition 0: Lag 900 (catching up)
# Partition 1: No lag (up-to-date)
# Partition 2: Lag 1700 (falling behind)
# → Hot partition (more data) or one consumer slow

# Solution:
# 1. Check if specific partition has larger key distribution
# 2. Add more consumers (up to num_partitions)
# 3. Optimize consumer processing logic
# 4. Check downstream system performance
```

### Reliability & Data Integrity

#### Monitoring Kafka Health

```java
@Component
public class KafkaHealthMonitor {
    
    @Bean
    public Micrometer metrics() {
        // Track key metrics via Prometheus
        return MeterRegistry;
    }
    
    // Key metrics to monitor
    /*
    kafka.consumer.fetch.total_bytes_consumed_total
    kafka.consumer.records_lag_max
    kafka.consumer.records_lag_sum
    kafka.consumer_records_consumed_total
    
    kafka.producer.batch.size.avg
    kafka.producer.record_send_latency_avg
    kafka.producer_records_sent_total
    
    kafka.broker.active_controller_count
    kafka.broker.broker_metrics_is_leader_count
    */
}
```

#### Disaster Recovery Strategy

**RPO (Recovery Point Objective):** Time to last successful backup
**RTO (Recovery Time Objective):** Time to restore service

| Strategy | RPO | RTO | Cost | Complexity |
|---|---|---|---|---|
| **Single Cluster** (base HA) | Hours (replication.factor=3) | Minutes (failover) | Low | Low |
| **Active-Active Replication** (MirrorMaker) | Seconds | Seconds | Medium | Medium |
| **Active-Passive (Standby)** | Minutes | Minutes | Medium | Medium |
| **Backup & Restore** | Hours | Hours | High | High |

**Recommended for Production:** Active-Active with MirrorMaker + cross-region replication

---

## Revision Cheat Sheet

### Must-Remember Concepts

1. **Kafka is a distributed log:** Durable, replicated, partitioned
2. **Partitions are units of parallelism:** One partition → one consumer per group
3. **Offset is position in partition:** Consumer tracks offset; can replay from any offset
4. **Replication factor ≥ 3:** Protection against broker failures
5. **`min.insync.replicas`:** Minimum ISR required for write ack (durability guarantee)
6. **Consumer group:** Set of consumers sharing workload; enables horizontal scaling
7. **Rebalancing:** Partitions reassigned when consumer joins/leaves; causes pause
8. **Exactly-once:** Idempotent producer + transactions + manual offset commit + idempotent processing
9. **DLT:** Dead Letter Topic for failed messages after retries
10. **Isolation level:** `read_committed` waits for transaction commit; safer but slower

### Configuration Must-Knows

```
Producer Durability:
- acks=all, retries>1, min.insync.replicas>1, enable.idempotence=true

Producer Throughput:
- batch.size=100KB, linger.ms=50, compression.type=lz4

Consumer Reliability:
- enable.auto.commit=false, isolation.level=read_committed, manual commitSync()

Consumer Scalability:
- partitions >= expected_concurrent_consumers
- concurrency = min(partition_count, available_cores)

Topic Durability:
- replication.factor=3, min.insync.replicas=2, unclean.leader.election.enable=false
```

### Interview Traps & How to Answer

#### Trap 1: "Kafka ensures exactly-once out of the box"
**Wrong:** Yes, Kafka guarantees exactly-once.
**Right:** Kafka supports exactly-once semantics (EOS) but requires explicit configuration:
- Producer: idempotence enabled + retries configured
- Consumer: manual offset commit after processing
- Processing: must be idempotent (design required)
→ Kafka provides the building blocks; application must use them correctly

#### Trap 2: "Use auto-commit for best performance"
**Wrong:** Auto-commit is fastest.
**Right:** Auto-commit risks message loss if consumer crashes after processing but before offset commit.
→ Trade-off: Manual commit safer but slower. Choose based on requirements.
→ For exactly-once: must use manual commit.

#### Trap 3: "More partitions = better performance"
**Wrong:** More partitions always faster.
**Right:** More partitions increase parallelism up to consumer count; beyond that:
- More rebalancing overhead
- More files on disk
- More memory on broker
→ Calculate: partitions = max(expected_consumers, throughput_per_partition)

#### Trap 4: "Kafka transactions guarantee end-to-end exactly-once"
**Wrong:** Kafka transactions alone ensure exactly-once.
**Right:** Kafka transactions ensure exactly-once within Kafka (atomic writes).
→ For end-to-end: add idempotent processing logic in consumer.
→ Idempotency handles duplicates if consumer reprocesses (crash, rebalance).

#### Trap 5: "Consumer lag of 10,000 messages is always bad"
**Wrong:** High lag always means problem.
**Right:** Lag depends on message size and throughput.
→ 10,000 small messages (1KB each) = 10MB; might be seconds behind.
→ 10,000 large messages (1MB each) = 10GB; might be hours behind.
→ Use time-based lag (milliseconds behind) not offset-based lag for meaningful alerts.

#### Trap 6: "Set request.timeout.ms very high for reliability"
**Wrong:** Higher timeout = more reliable.
**Right:** Too-high timeout masks real problems and delays failure detection.
→ Recommendation: timeout = p99 latency + buffer
→ Too low: legitimate requests timeout unnecessarily
→ Too high: slow detection of broker failures; extended rebalancing

---

## Common Real-World Patterns

### Pattern 1: Event Sourcing with Kafka
```
Write all events to Kafka (single source of truth)
Consumers project events into materialized views (ElasticSearch, cache, DB)
On app restart: replay events from offset 0 to rebuild state
→ Full audit trail; time-travel debugging
```

### Pattern 2: Saga Pattern for Distributed Transactions
```
Order Service publishes OrderCreated
Payment Service subscribes; publishes PaymentProcessed
Inventory Service subscribes; publishes StockDecremented
If any step fails: publish Compensating transaction (refund, restock)
→ Eventual consistency without 2PC
```

### Pattern 3: CQRS (Command Query Responsibility Segregation)
```
Commands: Write requests → Kafka → Event Store (source of truth)
Queries: Read from materialized views (fast, denormalized)
Eventual consistency between write and read models
```

### Pattern 4: Stream Processing with Kafka Streams
```
KStream<userId, Event>: Immutable stream
.groupByKey().windowedBy(TimeWindows.ofSizeAndGrace(1h, 5m))
.aggregate(Aggregator) → Compute per-user metrics
.toStream().to("user-metrics-topic")
→ Real-time aggregations; sessionization; anomaly detection
```

---

## Quick Study Guide (60 minutes)

**Read (15 min):**
1. What is Apache Kafka? (Architecture, Brokers, Topics, Partitions)
2. Producer vs. Consumer vs. Broker (3 minute overview each)

**Understand (20 min):**
1. Exactly-once semantics (idempotence + transactions + offset commit)
2. Consumer groups and rebalancing (how scaling works)
3. ISR and durability guarantees

**Practice (15 min):**
1. Code Example 1: Spring Boot producer with error handling
2. Code Example 2: Spring Boot consumer with manual offset commit & DLT
3. Scenario 1: Payment processing with exactly-once

**Review (10 min):**
1. 10 must-remember concepts
2. 3 interview traps and correct answers
3. Key configuration values (acks, batch.size, linger.ms, min.insync.replicas)

---

## References & Further Reading

**Official Documentation:**
- Apache Kafka Docs: https://kafka.apache.org/documentation/
- Confluent Platform: https://docs.confluent.io/

**Key Paper:**
- Kafka: A Distributed Streaming Platform (NSDI 2015)

**Learning Resources:**
- Spring Kafka Documentation: https://spring.io/projects/spring-kafka
- Confluent Kafka Tutorials: https://developer.confluent.io/tutorials/

**Monitoring & Operations:**
- Prometheus JMX Exporter for Kafka
- Confluent Control Center
- LinkedIn Burrow (Kafka consumer lag monitoring)

---

**Last Updated:** January 2026
**Target Audience:** Java Backend Engineers with 3-5 years of experience preparing for product company and MNC interviews
