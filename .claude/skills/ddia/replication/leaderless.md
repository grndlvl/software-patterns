# Leaderless Replication

## Definition

**Leaderless replication** (Dynamo-style) allows any node to accept writes directly. Clients send writes to multiple replicas in parallel, and reads query multiple nodes.

## Quorum Consensus

**Quorum condition**: `w + r > n`

- **n** = total replicas
- **w** = nodes that must acknowledge write
- **r** = nodes queried on read

Common: n=3, w=2, r=2

### Write Process

```pseudocode
function write(key, value):
    replicas = get_replicas(key)
    successful = 0

    for replica in replicas:
        send_async(replica, WRITE, key, value)

    wait_for(timeout)

    if successful_responses >= w:
        return SUCCESS
    return FAILURE
```

### Read Process

```pseudocode
function read(key):
    replicas = get_replicas(key)
    responses = query_replicas(key, count=r)

    if responses.length < r:
        return ERROR

    return select_latest_version(responses)
```

## Read Repair and Anti-Entropy

### Read Repair

Fix stale data during reads:

```pseudocode
function read_with_repair(key):
    responses = read_from_nodes(key, count=r)
    latest = find_latest_version(responses)

    for response in responses:
        if response.version < latest.version:
            // Repair stale node asynchronously
            send_async(response.node, WRITE, key, latest.value)

    return latest.value
```

### Anti-Entropy

Background process comparing replicas:

```pseudocode
function anti_entropy():
    for partition in partitions:
        peer = select_random_peer()
        local_tree = build_merkle_tree(partition)
        peer_tree = get_merkle_tree(peer, partition)

        differences = compare_trees(local_tree, peer_tree)
        sync_differences(differences)
```

## Sloppy Quorums and Hinted Handoff

### Sloppy Quorum

Accept writes to any available nodes when preferred nodes are down:

```pseudocode
function write_sloppy(key, value):
    preferred = get_replicas(key)
    available = filter_available(preferred)

    if available.length < w:
        // Add temporary nodes
        available.extend(get_healthy_nodes())

    for node in available.take(w):
        if node in preferred:
            write(node, key, value)
        else:
            write_with_hint(node, key, value, intended_for=preferred)
```

### Hinted Handoff

Return data to correct nodes when they recover:

```pseudocode
function hinted_handoff():
    for hint in pending_hints:
        if is_available(hint.intended_node):
            transfer_data(hint.intended_node, hint.key, hint.value)
            delete_hint(hint)
```

## Detecting Concurrent Writes

### Version Vectors

```pseudocode
class VersionVector:
    counters: Map<NodeID, Integer>

    function happens_before(other):
        // All our counters <= other's, at least one <
        return all_less_or_equal AND at_least_one_less

    function concurrent_with(other):
        return not (happens_before(other) or other.happens_before(this))
```

## Last-Write-Wins Problems

```pseudocode
// Clock skew causes data loss
Client A (clock ahead):  write("x", "new", timestamp=1000)
Client B (clock behind): write("x", "old", timestamp=999)
// "new" wins even though "old" was written later!
```

## When to Use Leaderless

### Good Fit

| Scenario | Why |
|----------|-----|
| High availability | No single point of failure |
| Low latency writes | No leader bottleneck |
| Multi-datacenter | Write to nearest nodes |
| Shopping carts, sessions | Conflicts rare |

### Poor Fit

| Scenario | Why |
|----------|-----|
| Strong consistency | Only eventual consistency |
| Ordered operations | No ordering guarantees |
| Complex transactions | No coordination |

## Summary Table

| Aspect | Leaderless | Leader-Based |
|--------|------------|--------------|
| Write latency | Low | Medium |
| Availability | High | Medium |
| Consistency | Eventual | Strong possible |
| Conflict handling | Required | None |
| Single point of failure | None | Leader |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
