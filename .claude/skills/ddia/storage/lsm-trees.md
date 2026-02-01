# LSM-Trees (Log-Structured Merge Trees)

## Definition

**LSM-Tree (Log-Structured Merge Tree)** is a data structure optimized for write-heavy workloads that maintains data in multiple sorted levels, with periodic background merging (compaction) to consolidate levels.

**Key characteristics:**
- Sequential writes to in-memory structure (memtable)
- Periodic flushes to immutable on-disk sorted files (SSTables)
- Background compaction merges smaller files into larger ones
- Optimized for write throughput over read performance

## How LSM-Trees Work

### Core Components

```pseudocode
structure LSMTree:
    memtable: SortedMap           // In-memory write buffer
    immutable_memtables: Queue    // Memtables being flushed
    sstables: List<SSTable>       // On-disk sorted string tables
    write_ahead_log: WAL          // Durability guarantee
```

### SSTable (Sorted String Table)

```pseudocode
structure SSTable:
    data_blocks: List<Block>      // Sorted key-value pairs
    index_block: Block            // Sparse index: key → offset
    bloom_filter: BloomFilter     // Probabilistic membership test
    metadata: Metadata            // Min/max keys, level, timestamp
```

## Write Path

```pseudocode
function Write(key, value):
    // 1. Write to WAL for durability
    write_ahead_log.Append(key, value)

    // 2. Write to in-memory memtable
    memtable.Put(key, value)

    // 3. Check if memtable is full
    if memtable.Size() >= MEMTABLE_SIZE_THRESHOLD:
        FlushMemtable()

function FlushMemtable():
    // Mark current memtable as immutable
    immutable = memtable
    immutable_memtables.Enqueue(immutable)

    // Create new memtable for incoming writes
    memtable = NewSortedMap()

    // Asynchronously flush to SSTable
    BackgroundTask:
        sstable = ConvertToSSTable(immutable)
        sstables.Prepend(sstable)
        immutable_memtables.Dequeue(immutable)
        write_ahead_log.Truncate()
```

## Read Path

```pseudocode
function Read(key):
    // 1. Check memtable (most recent)
    value = memtable.Get(key)
    if value != null:
        return value

    // 2. Check immutable memtables
    for each immutable in immutable_memtables:
        value = immutable.Get(key)
        if value != null:
            return value

    // 3. Check SSTables (newest to oldest)
    for each sstable in sstables:
        // Quick negative check with bloom filter
        if !sstable.bloom_filter.MayContain(key):
            continue

        value = sstable.Get(key)
        if value != null:
            return value

    return null
```

## Compaction Strategies

### Size-Tiered Compaction

Merge SSTables of similar size together:

```pseudocode
function SizeTieredCompaction():
    buckets = GroupBySimilarSize(sstables)

    for each bucket in buckets:
        if bucket.Size() >= COMPACTION_THRESHOLD:
            merged = MergeSSTables(bucket)
            sstables.Remove(bucket)
            sstables.Add(merged)
```

**Characteristics:**
- Good write amplification
- Higher space amplification
- Read amplification can be high

### Leveled Compaction

Organize into levels with non-overlapping key ranges:

```pseudocode
function LeveledCompaction():
    // L0: overlapping SSTables from flushes
    // L1+: non-overlapping key ranges, 10x size each level

    for level from 0 to MAX_LEVEL - 1:
        if levels[level].Size() > levels[level].max_size:
            to_compact = SelectSSTable(level)
            overlapping = FindOverlapping(levels[level + 1], to_compact)

            merged = MergeSSTables(to_compact + overlapping)
            new_sstables = SplitNonOverlapping(merged)

            levels[level].Remove(to_compact)
            levels[level + 1].Remove(overlapping)
            levels[level + 1].Add(new_sstables)
```

**Characteristics:**
- Better read amplification (fewer files to check)
- Better space amplification
- Higher write amplification

## Write Amplification vs Read Amplification

| Metric | Size-Tiered | Leveled |
|--------|-------------|---------|
| Write Amplification | O(log n) | O(10 × levels) |
| Read Amplification | O(n files) | O(log n) |
| Space Amplification | High | Low (~10%) |

## When to Use LSM-Trees

### Ideal Use Cases

| Scenario | Why LSM-Trees Excel |
|----------|---------------------|
| Write-heavy workloads | Sequential writes → high throughput |
| Time-series data | Natural key ordering, append-only |
| Log aggregation | Continuous ingestion |
| Event sourcing | Immutable event log |

### Avoid When

| Scenario | Reason |
|----------|--------|
| Read-heavy workloads | B-trees faster for reads |
| Range scans critical | B-trees keep keys together |
| Predictable latency needed | Compaction causes spikes |

## Summary: LSM-Trees vs B-Trees

| Aspect | LSM-Trees | B-Trees |
|--------|-----------|---------|
| Write pattern | Sequential (fast) | Random (slower) |
| Write throughput | High | Moderate |
| Read throughput | Moderate | High |
| Write amplification | High (compaction) | Low |
| Read amplification | High (check levels) | Low |
| Point lookup | O(k × log n) | O(log n) |
| Examples | LevelDB, RocksDB, Cassandra | PostgreSQL, MySQL |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
