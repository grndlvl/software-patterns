# Stream Processing

## Definition

**Stream processing** is the continuous processing of unbounded data as it arrives, rather than waiting to collect a batch. Streams are infinite sequences of events ordered in time, where each event represents something that happened at a particular point in time.

**Core characteristics:**
- **Unbounded data**: No predetermined end to the input
- **Low latency**: Events processed shortly after they occur
- **Continuous computation**: Processing runs indefinitely
- **Event-driven**: Reacts to incoming events rather than polling

## Message Brokers vs Log-Based Storage

### Message Brokers (Traditional)

Examples: RabbitMQ, ActiveMQ, AWS SQS

**Characteristics:**
- Transient storage: Messages deleted after acknowledgment
- Multiple consumers via queues or topics
- Push-based delivery to consumers
- Load balancing across consumers in a group

**Use cases:**
- Asynchronous task queues
- Simple pub/sub messaging
- When message history is not needed

### Log-Based Storage

Examples: Apache Kafka, Amazon Kinesis

**Characteristics:**
- Durable storage: Messages retained for configured period (e.g., days/weeks)
- Append-only log structure
- Pull-based consumption: consumers read at their own pace
- Partition-based parallelism
- Replay capability: consumers can re-read historical data

**Use cases:**
- Event sourcing
- Stream processing pipelines
- Multiple independent consumers needing same data
- Auditing and debugging with message replay

## Stream Processing Patterns

### 1. Windowing

Grouping events into finite time-based buckets for aggregation.

**Window types:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Tumbling** | Fixed-size, non-overlapping windows | Hourly summaries |
| **Hopping** | Fixed-size, overlapping windows | Moving averages |
| **Sliding** | Window moves with each event | Recent N items |
| **Session** | Dynamic windows based on activity gaps | User sessions |

### 2. Stream Joins

Combining events from multiple streams based on correlation.

**Join types:**

| Type | Description | State Requirements |
|------|-------------|-------------------|
| **Stream-Stream** | Join events from two streams | Both streams buffered in window |
| **Stream-Table** | Enrich stream events with table data | Table data cached/indexed |
| **Table-Table** | Materialized view maintenance | Both tables' state maintained |

### 3. Stream Aggregations

Computing running aggregates over event streams.

**Common aggregations:**
- Counts, sums, averages
- Top-K elements
- Distinct count (approximate via HyperLogLog)
- Percentiles (approximate via t-digest)

## Time Handling

### Event Time vs Processing Time

| Aspect | Event Time | Processing Time |
|--------|------------|-----------------|
| **Definition** | When event actually occurred | When event is processed |
| **Source** | Embedded in event | System clock |
| **Accuracy** | Reflects reality | May have delays |
| **Complexity** | Requires watermarks | Simple |
| **Use when** | Correctness critical | Low-latency approximations OK |

### Watermarks

Watermarks indicate "all events with timestamp < T have been seen." They handle out-of-order events and late arrivals.

**Strategies:**
- **Perfect watermark**: Exact knowledge of event times (rare)
- **Heuristic watermark**: Best estimate based on observed patterns
- **Late data handling**: Allowed lateness period, then discard or update

## State Management

Stream processors need to maintain state for:
- Window contents
- Join buffers
- Aggregation results
- Session data

**State storage options:**

| Approach | Pros | Cons |
|----------|------|------|
| **In-memory** | Fast access | Limited by RAM, lost on failure |
| **Local disk** | Larger capacity | Slower, still lost on failure |
| **Remote database** | Durable, scalable | Network latency |
| **Embedded store** | Fast + durable (RocksDB) | Complexity of replication |

**State partitioning**: Co-locate state with processing by partitioning stream and state by same key.

## Exactly-Once Semantics and Checkpointing

### Processing Guarantees

| Guarantee | Meaning | Implementation |
|-----------|---------|----------------|
| **At-most-once** | May lose messages | No retries |
| **At-least-once** | May duplicate messages | Retry on failure |
| **Exactly-once** | Each message processed once | Idempotent processing + checkpointing |

### Checkpointing

Periodically save consistent snapshots of state and stream position.

**Mechanisms:**
- **Chandy-Lamport algorithm**: Distributed snapshot using barrier messages
- **Microbatching**: Process small batches atomically (Spark Streaming)
- **Transaction log**: Kafka transactions for exactly-once across multiple topics

**Recovery**: Restore from last checkpoint and replay messages from that point.

## Stream vs Batch Comparison

| Aspect | Batch Processing | Stream Processing |
|--------|------------------|-------------------|
| **Data** | Bounded, finite | Unbounded, infinite |
| **Latency** | Minutes to hours | Milliseconds to seconds |
| **State** | Recomputed each batch | Continuously maintained |
| **Completeness** | Knows when done | Never "done" |
| **Use case** | Periodic reports, ETL | Real-time dashboards, monitoring |
| **Complexity** | Simpler | Requires time/state handling |
| **Cost** | Can schedule off-peak | Continuous resource usage |

**Lambda Architecture**: Run both batch (for accuracy) and stream (for speed), merge results.

**Kappa Architecture**: Stream-only with replay capability; batch is just reprocessing the stream.

## When to Use Stream Processing

**Good fit:**
- Real-time monitoring and alerting
- Live dashboards and metrics
- Fraud detection and anomaly detection
- Recommendations based on recent behavior
- Event-driven microservices
- Complex event processing (CEP)
- Real-time ETL and data pipelines

**Poor fit:**
- Complete historical analysis (use batch)
- Computations requiring unbounded history
- Perfect accuracy required without late data handling
- Simple request/response APIs (use synchronous RPC)

## Pseudocode Examples

### Windowed Aggregation (Tumbling Windows)

```pseudocode
// Count page views per URL in 1-hour tumbling windows

function processPageView(event):
    key = (event.url, windowForTimestamp(event.timestamp))
    state.increment(key)

    if windowComplete(key.window):
        output((key.url, key.window, state.get(key)))
        state.remove(key)

function windowForTimestamp(timestamp):
    windowDuration = 1_HOUR
    return floor(timestamp / windowDuration) * windowDuration

function windowComplete(window):
    return currentWatermark() > window.end
```

### Stream-Stream Join

```pseudocode
// Join click events with impression events within 1-hour window

function processImpression(impression):
    state.impressions.put(impression.adId, impression)
    expireTime = impression.timestamp + 1_HOUR
    scheduler.scheduleExpiry(impression.adId, expireTime)

function processClick(click):
    impression = state.impressions.get(click.adId)
    if impression != null:
        output(JoinedEvent(impression, click))

function handleExpiry(adId):
    state.impressions.remove(adId)
```

### Maintaining Aggregate State

```pseudocode
// Running average of sensor readings per sensor

function processReading(event):
    key = event.sensorId
    agg = state.get(key, default={count: 0, sum: 0})

    agg.count += 1
    agg.sum += event.value
    state.put(key, agg)

    output((key, agg.sum / agg.count))
```

### Handling Late Data with Watermarks

```pseudocode
// Process events with allowed lateness

config:
    allowedLateness = 5_MINUTES

function processEvent(event):
    window = windowForTimestamp(event.timestamp)

    if event.timestamp < currentWatermark() - allowedLateness:
        // Too late, discard
        metrics.increment("late_events")
        return

    if event.timestamp < currentWatermark():
        // Late but within allowed lateness
        updateExistingWindow(window, event)
    else:
        // On-time event
        addToWindow(window, event)

function onWatermark(watermark):
    for window in windows.where(end < watermark):
        finalizeWindow(window)
        windows.remove(window)
```

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Definition** | Continuous processing of unbounded data streams |
| **Message Storage** | Brokers (transient) vs Logs (durable, replayable) |
| **Windowing** | Tumbling, hopping, sliding, session windows |
| **Time** | Event time (accurate) vs processing time (simple) |
| **State** | In-memory, disk, remote DB, or embedded store |
| **Guarantees** | At-most-once, at-least-once, exactly-once |
| **Checkpointing** | Periodic snapshots for fault tolerance |
| **vs Batch** | Lower latency, unbounded data, continuous state |
| **Use When** | Real-time analytics, monitoring, event-driven systems |
| **Challenges** | Late data, state management, exactly-once semantics |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
