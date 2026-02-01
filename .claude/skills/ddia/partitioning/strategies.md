# Partitioning/Sharding Strategies

## Definition

**Partitioning** (sharding) splits a large dataset across multiple nodes. Each partition is a small database of its own.

**Why partition?**
- Scalability: Distribute query load
- Storage: More data than one machine
- Performance: Parallelize queries

## Range Partitioning

Assign continuous key ranges to partitions:

```pseudocode
function partition_by_range(key, boundaries):
    // boundaries: ["D", "H", "M", "S"]
    // Partitions: [A-D), [D-H), [H-M), [M-S), [S-Z]
    for i in range(len(boundaries)):
        if key < boundaries[i]:
            return partition[i]
    return partition[len(boundaries)]
```

**Pros:** Efficient range queries, natural ordering
**Cons:** Hot spots with predictable keys

## Hash Partitioning

Apply hash function to distribute evenly:

```pseudocode
function partition_by_hash(key, num_partitions):
    return hash(key) mod num_partitions
```

**Pros:** Uniform distribution, no hot spots
**Cons:** Range queries scatter across all partitions

### Consistent Hashing

```pseudocode
class ConsistentHash:
    ring: Map<hash, node>

    function get_node(key):
        hash_value = hash(key)
        // Find first node >= hash_value on ring
        return ring.ceiling(hash_value) or ring.first()

    function add_node(node):
        for i in range(VIRTUAL_NODES):
            ring[hash(node + ":" + i)] = node
```

## Compound Partitioning

Combine strategies:

```pseudocode
// Hash for distribution, range within partition
function partition_compound(partition_key, sort_key):
    hash_partition = hash(partition_key) mod N
    range_partition = find_range(sort_key)
    return (hash_partition, range_partition)

// Used by Cassandra: (partition_key, clustering_key)
```

## Hot Spots

### Causes
- Celebrity problem: Popular key gets massive traffic
- Temporal patterns: Today's date concentrates writes
- Sequential IDs as partition keys

### Mitigation

```pseudocode
// Salting: Split hot key across multiple partitions
function write_salted(hot_key, value):
    salt = random(0, 99)
    actual_key = hot_key + ":" + salt
    write(actual_key, value)

function read_salted(hot_key):
    results = []
    for salt in range(100):
        results.append(read(hot_key + ":" + salt))
    return merge(results)
```

## Secondary Indexes

### Local (Document-Partitioned)

Each partition indexes its own documents:

```pseudocode
// Write: fast (single partition)
write_document(doc):
    partition = hash(doc.id)
    partitions[partition].store(doc)
    partitions[partition].index(doc.category, doc.id)

// Read by secondary: scatter-gather (slow)
query_by_category(category):
    results = []
    for partition in partitions:
        results.extend(partition.query_index(category))
    return results
```

### Global (Term-Partitioned)

Index partitioned separately from data:

```pseudocode
// Write: slower (may update multiple index partitions)
// Read: fast (query single index partition)
query_by_category(category):
    index_partition = hash(category) mod N
    return index_partitions[index_partition].query(category)
```

## Request Routing

| Strategy | Description |
|----------|-------------|
| Client-side | Client has partition logic |
| Routing tier | Proxy forwards requests |
| Gossip | Nodes share partition info |
| Coordination service | ZooKeeper tracks assignments |

```pseudocode
// Coordination service approach
class Router:
    function route(key):
        partition_map = zookeeper.get("/partitions")
        node = partition_map.find_node(key)
        return forward_to_node(node, key)
```

## Summary Table

| Strategy | Distribution | Range Queries | Hot Spot Risk |
|----------|--------------|---------------|---------------|
| Range | Can be uneven | Efficient | High |
| Hash | Uniform | Inefficient | Low |
| Consistent hash | Uniform | Inefficient | Low |
| Compound | Balanced | Efficient within partition | Medium |

| Index Type | Write Cost | Read Cost |
|------------|------------|-----------|
| Local secondary | Low | High (scatter-gather) |
| Global secondary | High | Low |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
