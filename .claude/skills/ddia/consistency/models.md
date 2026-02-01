# Consistency Models

## Definition

**Consistency** defines the guarantees about the ordering and visibility of operations in a distributed system. It answers the question: "When multiple replicas hold the same data, what guarantees can we make about what a client reads after a write?"

Consistency models exist on a spectrum from **strong** (expensive, slow, but simple to reason about) to **weak** (fast, available, but complex application logic).

## Strong Consistency

### Definition
Every read sees the most recent write or a consistent snapshot. The system appears as if there is only one copy of the data.

### Characteristics
- **Linearizability**: Operations appear to occur instantaneously at some point between their invocation and response
- **Strongest guarantee**: System behaves like a single-threaded program
- **Cost**: High latency, limited availability (can't tolerate network partitions)

### Use Cases
- Financial transactions
- Inventory management
- Leader election
- Distributed locks

### Pseudocode Example
```pseudocode
// Linearizable register implementation
class LinearizableRegister:
    value = null
    version = 0
    lock = new Lock()

    function write(new_value):
        lock.acquire()
        version = version + 1
        value = new_value
        replicate_to_all_nodes(value, version)
        wait_for_majority_ack()
        lock.release()

    function read():
        lock.acquire()
        ensure_latest_version()  // Read repair if needed
        result = value
        lock.release()
        return result
```

## Eventual Consistency

### Definition
If no new updates are made, eventually all replicas will converge to the same value. No guarantee about *when* convergence happens.

### Characteristics
- **High availability**: Reads and writes succeed even during network partitions
- **Low latency**: No coordination required
- **Anomalies possible**: Stale reads, conflicting writes, read-your-writes violations

### Use Cases
- DNS
- Caching layers
- Social media feeds
- Analytics dashboards

### Pseudocode Example
```pseudocode
// Eventually consistent key-value store
class EventuallyConsistentKV:
    local_data = {}
    replica_peers = []

    function write(key, value):
        timestamp = current_time()
        local_data[key] = {value: value, timestamp: timestamp}

        // Asynchronously propagate to replicas
        for peer in replica_peers:
            async_send(peer, {key: key, value: value, timestamp: timestamp})

        return success  // Don't wait for acks

    function read(key):
        if key in local_data:
            return local_data[key].value
        else:
            return null  // Or read from random replica

    function on_receive_update(key, value, timestamp):
        if key not in local_data or timestamp > local_data[key].timestamp:
            local_data[key] = {value: value, timestamp: timestamp}
```

## Causal Consistency

### Definition
Causally related operations are seen in the same order by all nodes. Concurrent operations may be seen in different orders.

### Characteristics
- **Preserves happens-before**: If A caused B, all nodes see A before B
- **Concurrent writes**: Can be reordered differently on different replicas
- **Weaker than linearizability**, stronger than eventual consistency

### Use Cases
- Collaborative editing
- Comment threads
- Distributed logs

### Pseudocode Example
```pseudocode
// Causal consistency using vector clocks
class CausalKV:
    data = {}
    vector_clock = {}  // node_id -> counter

    function write(key, value):
        // Increment local clock
        vector_clock[this_node_id] += 1

        entry = {
            value: value,
            causal_timestamp: copy(vector_clock)
        }
        data[key] = entry

        // Propagate with causal metadata
        broadcast_to_replicas(key, entry)

    function read(key):
        return data[key].value

    function on_receive_update(key, entry):
        // Only apply if we've seen all causal dependencies
        if can_apply(entry.causal_timestamp):
            if key not in data or entry.causal_timestamp > data[key].causal_timestamp:
                data[key] = entry
        else:
            buffer_for_later(key, entry)

    function can_apply(remote_clock):
        for node_id, counter in remote_clock:
            if counter > vector_clock.get(node_id, 0):
                return false
        return true
```

## Sequential Consistency

### Definition
All operations appear to execute in some sequential order, and operations from each individual process appear in the order specified by its program.

### Characteristics
- **Total order**: Single global order that all nodes agree on
- **Per-process order**: Each process's operations stay in order
- **Weaker than linearizability**: No real-time guarantees

### Use Cases
- Replicated state machines
- Distributed databases with total order broadcast

### Pseudocode Example
```pseudocode
// Sequential consistency via total order broadcast
class SequentiallyConsistentLog:
    log = []
    sequence_number = 0

    function append(operation):
        // Assign sequence number via coordination
        seq = atomic_increment_global_counter()

        entry = {seq: seq, op: operation}
        broadcast_to_all_replicas(entry)

        wait_for_delivery_confirmation(seq)

    function on_receive_entry(entry):
        // Buffer out-of-order entries
        buffer.add(entry)

        // Apply entries in sequence number order
        while buffer.has_next_sequence(sequence_number + 1):
            next_entry = buffer.remove_next()
            log.append(next_entry)
            apply(next_entry.op)
            sequence_number += 1
```

## Read-Your-Writes Consistency

### Definition
After a client writes a value, all subsequent reads by that same client see the written value (or later values).

### Characteristics
- **Session guarantee**: Per-client consistency
- **No global guarantee**: Different clients may see different states
- **Practical**: Prevents confusing user experiences

### Use Cases
- User profile updates
- Shopping cart operations
- Any user-facing application where users modify their own data

### Pseudocode Example
```pseudocode
// Read-your-writes via sticky sessions
class ReadYourWritesKV:
    data = {}
    last_write_timestamp = {}  // client_id -> timestamp

    function write(client_id, key, value):
        timestamp = current_time()
        data[key] = {value: value, timestamp: timestamp}
        last_write_timestamp[client_id] = timestamp

        async_replicate(key, value, timestamp)

    function read(client_id, key):
        min_timestamp = last_write_timestamp.get(client_id, 0)

        // Ensure we read from a replica that has client's writes
        replica = select_replica_with_timestamp(min_timestamp)

        return replica.get(key)

    function select_replica_with_timestamp(min_ts):
        for replica in replicas:
            if replica.last_applied_timestamp >= min_ts:
                return replica

        // Fallback: wait for replication or read from primary
        return primary_replica
```

## Monotonic Reads

### Definition
If a client reads a value v1, any subsequent reads by that client will return v1 or a later value (never an earlier value).

### Characteristics
- **No time travel**: Prevents reading older versions after newer ones
- **Session guarantee**: Per-client property
- **Implementation**: Route client's reads to same replica or use version tracking

### Use Cases
- Timeline consistency
- Preventing confusing rollbacks in UI
- Distributed caches

### Pseudocode Example
```pseudocode
// Monotonic reads via session tracking
class MonotonicReadKV:
    data = {}
    client_sessions = {}  // client_id -> max_version_seen

    function read(client_id, key):
        max_version_seen = client_sessions.get(client_id, 0)

        // Find a replica that is at least as up-to-date
        replica = select_replica_with_min_version(max_version_seen)

        value, version = replica.get_with_version(key)

        // Update session tracking
        client_sessions[client_id] = max(version, max_version_seen)

        return value

    function write(key, value):
        version = atomic_increment_version()
        data[key] = {value: value, version: version}
        replicate(key, value, version)
```

## CAP Theorem and PACELC

### CAP Theorem
In the presence of a network **Partition**, you must choose between **Availability** and **Consistency**.

```pseudocode
// Visualization of CAP trade-off
if network_partition_occurs():
    choose_one_of:

        option CP:  // Consistency + Partition tolerance
            refuse_requests_to_minority_partition()
            maintain_strong_consistency()
            // Examples: HBase, MongoDB (with majority reads)

        option AP:  // Availability + Partition tolerance
            accept_all_requests()
            allow_divergence()
            reconcile_later()
            // Examples: Cassandra, DynamoDB (with relaxed consistency)
```

### PACELC
Extends CAP: **If Partition**, choose between **Availability** and **Consistency**; **Else** (no partition), choose between **Latency** and **Consistency**.

```pseudocode
// PACELC decision tree
if network_partition:
    if prioritize_availability:
        accept_stale_reads  // AP
    else:
        reject_requests_to_minority  // CP
else:  // Normal operation
    if prioritize_low_latency:
        read_from_nearest_replica  // May be stale (EL)
    else:
        read_from_primary_or_quorum  // Consistent but slower (EC)

// System classifications:
// PA/EL: Cassandra, DynamoDB - Always prioritize availability and latency
// PC/EC: HBase, MongoDB - Always prioritize consistency
// PA/EC: Cosmos DB - Available during partition, consistent otherwise
// PC/EL: Many caches - Consistent when healthy, low latency normally
```

## Choosing the Right Model

### Decision Framework

```pseudocode
function choose_consistency_model(requirements):

    // Strong consistency needed?
    if requires_linearizability():
        // Banking, inventory, coordination
        return STRONG_CONSISTENCY
        note: "High latency, limited availability"

    // Can tolerate any staleness?
    if can_tolerate_stale_reads() and no_write_conflicts():
        // Analytics, caching, read-heavy
        return EVENTUAL_CONSISTENCY
        note: "Best performance and availability"

    // Need causal ordering?
    if must_preserve_causality():
        // Collaborative editing, social feeds
        return CAUSAL_CONSISTENCY
        note: "Good balance for interactive apps"

    // Single-user consistency enough?
    if single_user_view_important():
        return READ_YOUR_WRITES + MONOTONIC_READS
        note: "Good UX without global coordination"

    // Default for unknown requirements
    return EVENTUAL_CONSISTENCY + session_guarantees
```

### Practical Guidelines

| Requirement | Recommended Model | Trade-offs |
|-------------|-------------------|------------|
| Money transfers | Strong (Linearizable) | Slow, less available |
| User profile | Read-your-writes | Fast, good UX |
| Social feed | Eventual or Causal | Very fast, minor inconsistencies |
| Comment threads | Causal | Preserves reply order |
| Analytics dashboard | Eventual | Fast, slightly stale okay |
| Leader election | Strong | Safety critical |
| Shopping cart | Read-your-writes + Monotonic | User sees own changes |
| Real-time collaboration | Causal | Balances performance and correctness |

### Hybrid Approaches

```pseudocode
// Different consistency for different data types
class HybridStore:

    function write(key, value, consistency_level):
        if consistency_level == STRONG:
            return linearizable_write(key, value)
        elif consistency_level == CAUSAL:
            return causal_write(key, value)
        else:
            return eventual_write(key, value)

    function read(key, consistency_level):
        if consistency_level == STRONG:
            return quorum_read(key)
        elif consistency_level == SESSION:
            return session_consistent_read(client_id, key)
        else:
            return any_replica_read(key)

// Example: E-commerce system
class EcommerceSystem:
    inventory = HybridStore()  // Use STRONG for inventory
    user_cart = HybridStore()  // Use SESSION for cart
    recommendations = HybridStore()  // Use EVENTUAL for recommendations

    function checkout(user_id, items):
        // Strong consistency for inventory deduction
        for item in items:
            inventory.decrement(item.sku, item.quantity, STRONG)

        // Session consistency for user's order history
        user_cart.clear(user_id, SESSION)
```

## Pseudocode Examples

### Complete Consistency Model Implementations

#### 1. Quorum-Based Tunable Consistency

```pseudocode
class QuorumKV:
    replicas = 3
    data = {}

    function write(key, value, W):
        // W = write quorum (how many replicas must ack)
        timestamp = current_time()
        entry = {value: value, timestamp: timestamp}

        acks = parallel_write_to_replicas(key, entry, replicas)

        if acks >= W:
            return success
        else:
            return failure  // Couldn't reach quorum

    function read(key, R):
        // R = read quorum (how many replicas must respond)
        responses = parallel_read_from_replicas(key, R)

        // Return value with highest timestamp
        latest = responses.max_by(r => r.timestamp)

        // Read repair: propagate latest value
        for response in responses:
            if response.timestamp < latest.timestamp:
                async_write(response.replica, key, latest)

        return latest.value

    // Consistency guarantees:
    // - R + W > N: Strong consistency (quorum overlap)
    // - R = 1, W = N: Fast reads, slow writes
    // - R = N, W = 1: Fast writes, slow reads
    // - R = W = N/2 + 1: Balanced
```

#### 2. Conflict-Free Replicated Data Type (CRDT)

```pseudocode
// Last-Write-Wins Register (LWW-Register)
class LWWRegister:
    value = null
    timestamp = 0
    node_id = get_node_id()

    function set(new_value):
        timestamp = current_time()
        value = new_value
        broadcast({value: value, timestamp: timestamp, node: node_id})

    function get():
        return value

    function merge(remote_value, remote_timestamp, remote_node):
        // Deterministic conflict resolution
        if remote_timestamp > timestamp:
            value = remote_value
            timestamp = remote_timestamp
        elif remote_timestamp == timestamp and remote_node > node_id:
            // Tie-breaker using node ID
            value = remote_value
            timestamp = remote_timestamp

// Grow-Only Counter (G-Counter)
class GCounter:
    counts = {}  // node_id -> local_count

    function increment():
        node_id = get_node_id()
        counts[node_id] = counts.get(node_id, 0) + 1
        broadcast(counts)

    function value():
        return sum(counts.values())

    function merge(remote_counts):
        for node_id, remote_count in remote_counts:
            counts[node_id] = max(counts.get(node_id, 0), remote_count)
```

#### 3. Version Vector for Conflict Detection

```pseudocode
class VersionedKV:
    data = {}

    function write(client_id, key, value, client_version_vector):
        // Check for conflicts
        if key in data:
            existing = data[key]

            if version_vectors_concurrent(client_version_vector, existing.version):
                // Concurrent write - record conflict
                data[key] = {
                    values: [existing.value, value],
                    versions: [existing.version, client_version_vector],
                    conflict: true
                }
                return CONFLICT

        // No conflict - normal write
        new_version = merge_version_vectors(client_version_vector, get_local_version())
        data[key] = {
            value: value,
            version: new_version,
            conflict: false
        }
        return SUCCESS

    function read(key):
        entry = data[key]
        if entry.conflict:
            return {
                status: CONFLICT,
                conflicting_values: entry.values,
                versions: entry.versions
            }
        else:
            return {status: OK, value: entry.value, version: entry.version}

    function resolve_conflict(key, resolved_value):
        // Application provides conflict resolution
        new_version = increment_version_vector()
        data[key] = {
            value: resolved_value,
            version: new_version,
            conflict: false
        }

function version_vectors_concurrent(v1, v2):
    // Returns true if neither v1 <= v2 nor v2 <= v1
    v1_less = false
    v2_less = false

    all_nodes = set(v1.keys()).union(set(v2.keys()))
    for node in all_nodes:
        if v1.get(node, 0) < v2.get(node, 0):
            v2_less = true
        if v1.get(node, 0) > v2.get(node, 0):
            v1_less = true

    return v1_less and v2_less  // Concurrent if both have some higher values
```

## Summary Table

| Model | Guarantee | Visibility | Ordering | Availability | Latency | Complexity |
|-------|-----------|------------|----------|--------------|---------|------------|
| **Strong (Linearizable)** | See all writes | Immediate | Total order | Low (CP) | High | Low (simple) |
| **Sequential** | See all writes in some order | Immediate | Per-process + global | Medium | Medium-High | Low |
| **Causal** | See causally-related writes in order | Immediate (causal) | Partial order | High | Low-Medium | Medium |
| **Read-Your-Writes** | See own writes | Immediate (own) | None (global) | High | Low | Medium |
| **Monotonic Reads** | No backwards time travel | May lag | Per-client forward | High | Low | Low |
| **Eventual** | Eventually see all writes | Delayed | None | Highest (AP) | Lowest | High (app) |

### Consistency vs. Performance

```
Strong ────────────────────────────────────────── Eventual
  ↑                                                  ↑
  │                                                  │
Consistency                                    Performance
& Simplicity                                   & Availability
  │                                                  │
  ↓                                                  ↓
Slow, Limited Availability              Fast, Highly Available
Single-threaded semantics              Complex conflict handling
```

### When to Use What

```pseudocode
// Quick reference guide
consistency_requirements = {
    "financial_transaction": LINEARIZABLE,
    "inventory_decrement": LINEARIZABLE,
    "distributed_lock": LINEARIZABLE,

    "user_profile_update": READ_YOUR_WRITES,
    "shopping_cart": READ_YOUR_WRITES + MONOTONIC_READS,
    "user_preferences": SESSION_CONSISTENCY,

    "comment_thread": CAUSAL,
    "chat_messages": CAUSAL,
    "collaborative_doc": CAUSAL,

    "social_feed": EVENTUAL,
    "recommendation_engine": EVENTUAL,
    "analytics_dashboard": EVENTUAL,
    "dns_lookup": EVENTUAL,

    "replicated_state_machine": SEQUENTIAL,
    "total_order_broadcast": SEQUENTIAL
}
```

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
