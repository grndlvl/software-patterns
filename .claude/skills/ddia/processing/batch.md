# Batch Processing

## Definition

**Batch processing** processes large, bounded datasets in scheduled jobs. Jobs read input, transform data, and write output without user interaction.

**Characteristics:**
- Bounded input (finite dataset)
- High latency (minutes to hours)
- High throughput (petabytes/day)
- Scheduled execution
- Immutable inputs

## Unix Philosophy

```pseudocode
// Composable tools with pipes
cat access.log |
  awk '{print $3}' |    // Extract user ID
  sort | uniq |         // Deduplicate
  wc -l                 // Count
```

**Principles:**
- Simple tools doing one thing well
- Text streams as universal interface
- Composability via pipes

## MapReduce

```pseudocode
function MapReduce(input, map_fn, reduce_fn):
    // PHASE 1: MAP
    for record in input:
        key_value_pairs = map_fn(record)
        emit to intermediate

    // PHASE 2: SHUFFLE (framework groups by key)
    grouped = group_by_key(intermediate)

    // PHASE 3: REDUCE
    for (key, values) in grouped:
        result = reduce_fn(key, values)
        emit to output
```

### Word Count Example

```pseudocode
function map(document):
    for word in document.split():
        emit (word, 1)

function reduce(word, counts):
    emit (word, sum(counts))

// Input: ["hello world", "hello"]
// Map output: [(hello,1), (world,1), (hello,1)]
// Shuffle: {hello: [1,1], world: [1]}
// Reduce output: [(hello,2), (world,1)]
```

## Joins in Batch Processing

### Sort-Merge Join (Reduce-Side)

```pseudocode
// Join users with events
function map_users(user):
    emit (user.id, ("user", user))

function map_events(event):
    emit (event.user_id, ("event", event))

function reduce_join(user_id, records):
    user = find_by_tag(records, "user")
    events = filter_by_tag(records, "event")
    for event in events:
        emit merge(user, event)
```

### Broadcast Hash Join (Map-Side)

```pseudocode
// Small table fits in memory
function setup():
    users_map = load_into_memory(users_table)

function map(event):
    user = users_map[event.user_id]
    emit merge(user, event)

// No shuffle needed!
```

## Beyond MapReduce (Spark, Flink)

| Feature | MapReduce | Dataflow Engines |
|---------|-----------|------------------|
| Operators | Map, Reduce | Many (filter, join, window) |
| Intermediate | Disk (HDFS) | Memory |
| Iteration | Multiple jobs | Native loops |
| Fault tolerance | Task retry | Lineage + recompute |

```pseudocode
// Spark: single DAG, pipelined
rdd = input
    .map(transform)
    .filter(predicate)
    .reduceByKey(aggregate)
    .collect()
```

## Fault Tolerance

### Task-Level Retry

```pseudocode
function execute_task(task):
    for attempt in range(MAX_ATTEMPTS):
        try:
            output = run_task(task)
            atomic_commit(output)
            return SUCCESS
        catch:
            retry on different machine
    fail_job()
```

### Lineage-Based Recovery (Spark)

```pseudocode
class RDD:
    dependencies: List<RDD>
    compute_function: Function

    function recompute(partition):
        if cached:
            return cache[partition]
        parent_data = dependencies.map(d => d.recompute(partition))
        return compute_function(parent_data)
```

## When to Use Batch

| Use Case | Why Batch |
|----------|-----------|
| Daily reports | Scheduled, not real-time |
| ML training | High throughput on large data |
| Log aggregation | Process accumulated logs |
| ETL pipelines | Nightly data warehouse loads |
| Search indexing | Build index from snapshot |

| Requirement | Batch | Stream | Database |
|-------------|-------|--------|----------|
| Latency | Hours | Seconds | Milliseconds |
| Data volume | Petabytes | Unbounded | Terabytes |
| Query pattern | Full scans | Per-event | Point queries |

## Summary

| Aspect | Details |
|--------|---------|
| Definition | Process bounded datasets in scheduled jobs |
| Latency | Minutes to hours |
| Throughput | Very high |
| Execution | MapReduce, Spark, Flink |
| Fault tolerance | Task retry, lineage |
| Best for | Analytics, ETL, ML training |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
