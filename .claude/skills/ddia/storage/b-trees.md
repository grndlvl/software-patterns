# B-Trees

## Overview

B-Trees are the most widely used indexing structure in databases, forming the foundation for storage engines in almost all relational databases and many non-relational ones. Unlike LSM-Trees which use sequential writes, B-Trees use in-place updates and provide strong read performance through a balanced tree structure.

## History and Context

B-Trees were introduced in 1970 by Rudolf Bayer and Edward McCreight at Boeing Research Labs. They have remained dominant in database systems for over 50 years because:

- **Predictable performance**: Balanced structure guarantees O(log n) operations
- **Strong read performance**: Direct access path to any key
- **Well-understood**: Decades of optimization research and production experience
- **Hardware-friendly**: Page-based design aligns with disk and OS caching

## How B-Trees Work

### Core Structure

B-Trees break the database into fixed-size **blocks** or **pages** (traditionally 4KB), matching the underlying hardware. Each page can be identified by an address/location on disk.

```pseudocode
// Basic B-Tree Node Structure
Node {
    isLeaf: boolean
    keys: Array<Key>           // Sorted keys
    children: Array<NodeRef>   // References to child pages (if internal node)
    values: Array<Value>       // Values (if leaf node)
}
```

**Key Properties:**

- **Balanced**: All leaf nodes are at the same depth
- **Branching factor**: Each page contains multiple keys (typically hundreds)
- **Self-balancing**: Tree maintains balance during insertions/deletions
- **Page-oriented**: Operations work at the granularity of full pages

### Tree Depth and Capacity

For a B-Tree with 4KB pages and a branching factor of 500:

```pseudocode
// Storage capacity by depth
Depth 1: 500 keys
Depth 2: 500 × 500 = 250,000 keys
Depth 3: 500 × 500 × 500 = 125,000,000 keys
Depth 4: 500^4 = 62.5 billion keys

// A 4-level B-Tree can store 256TB of data
// with each lookup requiring only 4 page reads
```

## Read Operations

### Point Lookup

```pseudocode
function btree_search(rootPage, searchKey):
    currentPage = loadPage(rootPage)

    while not currentPage.isLeaf:
        // Binary search within page
        index = binarySearch(currentPage.keys, searchKey)
        childRef = currentPage.children[index]
        currentPage = loadPage(childRef)

    // Found leaf page
    index = binarySearch(currentPage.keys, searchKey)
    if currentPage.keys[index] == searchKey:
        return currentPage.values[index]
    else:
        return NOT_FOUND
```

**Performance characteristics:**

- **Predictable**: Always log(n) page reads
- **Cache-friendly**: Pages align with OS page cache
- **No compaction overhead**: Unlike LSM-Trees, no need to check multiple structures

### Range Scan

```pseudocode
function btree_range_scan(rootPage, startKey, endKey):
    results = []

    // Navigate to start of range
    currentPage = navigateToLeaf(rootPage, startKey)

    // Scan leaf pages (often linked as doubly-linked list)
    while currentPage != null:
        for i = 0 to currentPage.keys.length:
            if currentPage.keys[i] >= startKey and currentPage.keys[i] <= endKey:
                results.append(currentPage.values[i])
            else if currentPage.keys[i] > endKey:
                return results

        currentPage = currentPage.nextLeafPage

    return results
```

## Write Operations

### Insertion Without Split

```pseudocode
function btree_insert(rootPage, key, value):
    // Find leaf page for insertion
    path = navigateToLeaf(rootPage, key, trackPath=true)
    leafPage = path.last()

    if leafPage.hasSpace():
        // Simple case: insert into existing page
        insertIntoPage(leafPage, key, value)
        writePage(leafPage)
    else:
        // Complex case: page is full, must split
        splitAndInsert(path, key, value)
```

### Page Splitting

When a page is full, it must be split:

```pseudocode
function splitAndInsert(path, key, value):
    leafPage = path.last()

    // 1. Create new page
    newPage = allocatePage()

    // 2. Distribute keys between old and new page
    midpoint = leafPage.keys.length / 2
    newPage.keys = leafPage.keys[midpoint:]
    newPage.values = leafPage.values[midpoint:]
    leafPage.keys = leafPage.keys[:midpoint]
    leafPage.values = leafPage.values[:midpoint]

    // 3. Insert key into appropriate page
    if key < leafPage.keys.last():
        insertIntoPage(leafPage, key, value)
    else:
        insertIntoPage(newPage, key, value)

    // 4. Add reference to new page in parent
    parentPage = path[path.length - 2]
    insertChildReference(parentPage, newPage.firstKey, newPage.pageRef)

    // 5. If parent is now full, recursively split
    if not parentPage.hasSpace():
        splitParent(path[:-1], newPage.firstKey, newPage.pageRef)
```

**Split implications:**

- **Write amplification**: One insertion can cause multiple page updates
- **Dangerous operation**: Corrupts index if interrupted
- **Requires WAL**: Must be recoverable from crashes

## Write-Ahead Log (WAL)

B-Trees use a write-ahead log to ensure crash recovery:

```pseudocode
// WAL-protected update
function wal_protected_update(page, operation):
    // 1. Write operation to log first
    walEntry = {
        timestamp: now(),
        operation: operation,
        pageId: page.id,
        oldData: page.serialize(),
        newData: operation.apply(page).serialize()
    }
    appendToWAL(walEntry)
    syncWAL()  // Force to disk

    // 2. Now safe to modify the page
    applyOperation(page, operation)
    writePage(page)

    // 3. Checkpoint: can truncate old WAL entries
    if shouldCheckpoint():
        checkpoint()
```

**WAL guarantees:**

- **Atomicity**: Operations are atomic even across page splits
- **Durability**: Committed writes survive crashes
- **Recovery**: Can replay log to restore consistent state

## Optimizations

### Copy-on-Write (LMDB, BoltDB)

Instead of overwriting pages in place, write modified pages to new locations:

```pseudocode
function cow_update(rootPage, key, value):
    // Clone path from root to leaf
    newLeaf = clone(findLeaf(rootPage, key))
    insertIntoPage(newLeaf, key, value)

    // Clone parent chain with updated references
    newParent = cloneParent(newLeaf)
    newRoot = updateParentChain(rootPage, newParent)

    // Atomic switch to new root
    atomicSwapRoot(rootPage, newRoot)

    // Old pages can be garbage collected
    markForGC(rootPage)
```

**Benefits:**

- No WAL needed (writes are atomic)
- Enables snapshot isolation (old roots remain valid)
- Better concurrency (readers don't block writers)

### Key Abbreviation

Store only enough of the key to determine boundaries:

```pseudocode
// Instead of storing full keys in internal nodes
InternalNode {
    keys: ["aardvark", "banana", "cucumber", "dragon"]
    children: [ref1, ref2, ref3, ref4, ref5]
}

// Store abbreviated keys
InternalNode {
    keys: ["b", "c", "d"]  // Only need to distinguish ranges
    children: [ref1, ref2, ref3, ref4, ref5]
}
```

**Benefits:**

- Higher branching factor (more keys per page)
- Shallower trees
- Fewer page reads per lookup

### B+ Tree Variant

Store values only in leaf nodes, internal nodes only store keys:

```pseudocode
// B+ Tree structure
InternalNode {
    keys: Array<Key>
    children: Array<NodeRef>  // No values
}

LeafNode {
    keys: Array<Key>
    values: Array<Value>
    nextLeaf: NodeRef  // Leaf pages form linked list
}
```

**Benefits:**

- More keys per internal node (higher branching factor)
- Efficient range scans (traverse linked leaf list)
- All values at same depth (predictable performance)

## Read Performance Characteristics

### Strengths

1. **Predictable latency**: Always O(log n) page reads
2. **No read amplification**: Single lookup path (vs. multiple levels in LSM)
3. **Cache-friendly**: Hot pages stay in memory
4. **Good for point queries**: Direct access path

### Weaknesses

1. **Random I/O**: Page updates scattered across disk
2. **Write amplification**: Page splits can cascade
3. **Fragmentation**: Over time, pages may become partially empty
4. **Space overhead**: Pages typically 50-70% full on average

## When to Use B-Tree Storage

**Best suited for:**

| Use Case | Why B-Tree Wins |
|----------|-----------------|
| Read-heavy workloads | No compaction overhead, single lookup path |
| Point queries | Direct access, no merge needed |
| Range scans with low write rate | Sequential leaf page reads |
| Strong consistency requirements | In-place updates, no compaction delays |
| Limited RAM | Smaller memory footprint than LSM bloom filters |
| Disk-based storage | Page-aligned I/O matches disk behavior |

**Examples:**

- Traditional relational databases (PostgreSQL, MySQL InnoDB)
- Embedded databases (SQLite, LMDB)
- File systems (Btrfs, ext4)
- Analytics with infrequent updates

## B-Tree vs. LSM-Tree Comparison

| Aspect | B-Tree | LSM-Tree |
|--------|--------|----------|
| **Write Pattern** | In-place updates, random I/O | Sequential writes only |
| **Read Pattern** | Single lookup path | Check multiple levels |
| **Write Amplification** | Lower (only updated pages) | Higher (repeated compaction) |
| **Read Amplification** | Lower (single path) | Higher (bloom + multiple levels) |
| **Space Amplification** | 50-70% page utilization | Lower (compaction removes tombstones) |
| **Write Throughput** | Lower (random I/O) | Higher (sequential) |
| **Read Latency** | Lower (predictable) | Higher (variable, depends on compaction) |
| **Crash Recovery** | WAL replay | Memtable recovery |
| **Best For** | Read-heavy, point queries | Write-heavy, range scans |

## Complete Example: Simple B-Tree Implementation

```pseudocode
// Simple B-Tree with order 3 (max 2 keys per node)
class BTree {
    root: Node
    order: int = 3

    function search(key):
        return search_recursive(root, key)

    function search_recursive(node, key):
        i = 0
        while i < node.keys.length and key > node.keys[i]:
            i += 1

        if i < node.keys.length and key == node.keys[i]:
            return node.values[i]

        if node.isLeaf:
            return NOT_FOUND

        return search_recursive(node.children[i], key)

    function insert(key, value):
        if root.isFull():
            newRoot = Node(isLeaf=false)
            newRoot.children[0] = root
            splitChild(newRoot, 0)
            root = newRoot

        insertNonFull(root, key, value)

    function insertNonFull(node, key, value):
        if node.isLeaf:
            // Find insertion position
            i = node.keys.length - 1
            while i >= 0 and key < node.keys[i]:
                node.keys[i+1] = node.keys[i]
                node.values[i+1] = node.values[i]
                i -= 1

            node.keys[i+1] = key
            node.values[i+1] = value
        else:
            // Find child to insert into
            i = node.keys.length - 1
            while i >= 0 and key < node.keys[i]:
                i -= 1
            i += 1

            if node.children[i].isFull():
                splitChild(node, i)
                if key > node.keys[i]:
                    i += 1

            insertNonFull(node.children[i], key, value)

    function splitChild(parent, index):
        fullChild = parent.children[index]
        newNode = Node(isLeaf=fullChild.isLeaf)

        // Move half of keys to new node
        mid = order / 2
        newNode.keys = fullChild.keys[mid+1:]
        fullChild.keys = fullChild.keys[:mid]

        if fullChild.isLeaf:
            newNode.values = fullChild.values[mid+1:]
            fullChild.values = fullChild.values[:mid]
        else:
            newNode.children = fullChild.children[mid+1:]
            fullChild.children = fullChild.children[:mid+1]

        // Insert middle key into parent
        parent.keys.insert(index, fullChild.keys[mid])
        parent.children.insert(index+1, newNode)
}
```

## Summary

**B-Trees provide:**

✅ Predictable O(log n) performance
✅ Strong read performance
✅ Mature, well-understood technology
✅ Decades of production hardening

**At the cost of:**

❌ Random I/O for writes
❌ Write amplification from splits
❌ Gradual fragmentation
❌ WAL overhead

**Use B-Trees when:** You need predictable latency, read-heavy workloads, or strong consistency guarantees without compaction delays.

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
