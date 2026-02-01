# Multi-Leader Replication

## Definition

**Multi-leader replication** allows multiple nodes to accept writes, with changes replicated to all other leaders. Used for multi-datacenter operations and offline clients.

## Use Cases

### Multi-Datacenter Operations

```pseudocode
// Leader in each datacenter
Datacenter A: Leader A ←→ Leader B :Datacenter B
                ↓                      ↓
            Followers              Followers

// Benefits:
// - Low latency (writes to local datacenter)
// - Datacenter fault tolerance
// - Network partition tolerance
```

### Offline Clients

Each device acts as a leader that syncs when online:

```pseudocode
// Calendar app on multiple devices
Phone: Local DB (leader) ←→ Cloud ←→ Laptop: Local DB (leader)

// Can write offline, sync later, resolve conflicts
```

## Conflict Handling

### Last Write Wins (LWW)

```pseudocode
function resolve_lww(value1, value2):
    if value1.timestamp > value2.timestamp:
        return value1
    return value2

// Problem: Data loss! One write is discarded
```

### Custom Merge Resolution

```pseudocode
// Shopping cart: union merge
function merge_carts(cart1, cart2):
    merged = new Set()
    merged.addAll(cart1.items)
    merged.addAll(cart2.items)
    return merged
```

### Version Vectors

```pseudocode
class VersionVector:
    counters: Map<NodeID, Integer>

    function concurrent_with(other):
        // Neither dominates → conflict detected
        return not (this.dominates(other) or other.dominates(this))
```

### CRDTs (Conflict-free Replicated Data Types)

```pseudocode
// G-Set: Grow-only set, always converges
class GrowOnlySet:
    function merge(other):
        return this.elements.union(other.elements)
```

## Replication Topologies

| Topology | Description | Fault Tolerance |
|----------|-------------|-----------------|
| Circular | Each leader → next in ring | One failure breaks chain |
| Star | Central leader distributes | Central is bottleneck/SPOF |
| All-to-All | Every leader → all others | Most fault tolerant |

```pseudocode
// All-to-All with causality
function replicate(key, value, version_vector):
    for peer in peer_leaders:
        peer.receive_write(key, value, version_vector)
```

## When to Use

### Good Fit

| Scenario | Reason |
|----------|--------|
| Multi-datacenter | Low latency local writes |
| Offline capable | Devices as independent leaders |
| Geographic distribution | Regional write autonomy |

### Poor Fit

| Scenario | Reason |
|----------|--------|
| Strong consistency needed | Conflicts make it impossible |
| Simple data model | Added complexity not worth it |
| Critical financial data | Conflict resolution may lose data |

## Summary

| Aspect | Multi-Leader | Single-Leader |
|--------|--------------|---------------|
| Write latency | Low (local) | Higher (to leader) |
| Consistency | Eventual | Strong possible |
| Conflicts | Must handle | None |
| Complexity | High | Lower |
| Availability | Higher | Medium |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
