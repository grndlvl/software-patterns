# Data Structure Selection Guide

A practical guide for choosing the right data structure for your problem.

## Quick Decision Tree

```
START: What operations do you need?

├─ Need fast random access by index?
│  ├─ Fixed size? → Array
│  ├─ Need to grow/shrink? → Dynamic Array (List)
│  └─ Both ends matter? → Deque
│
├─ Need fast lookup by key?
│  ├─ Order doesn't matter? → Hash Table (Map/Dict)
│  ├─ Need sorted order? → Tree Map (BST/Red-Black)
│  └─ Need ranges? → Tree Map or Skip List
│
├─ Need fast insert/delete?
│  ├─ At ends only? → Deque or Linked List
│  ├─ In middle? → Linked List
│  ├─ By value? → Set (Hash or Tree)
│  └─ Priority matters? → Heap
│
├─ Need to maintain order?
│  ├─ Sorted automatically? → Tree Set or Heap
│  ├─ Custom order? → Linked List or Array
│  └─ Insertion order? → Linked Hash Map
│
├─ Need hierarchical structure?
│  ├─ Parent-child relationships? → Tree
│  ├─ Network relationships? → Graph
│  └─ Categories/tags? → Tree or Graph
│
└─ Need specialized behavior?
   ├─ Last-in-first-out? → Stack
   ├─ First-in-first-out? → Queue
   ├─ Always get min/max? → Heap
   ├─ Unique items only? → Set
   ├─ Need fast prefix search? → Trie
   └─ Need to find in text? → Suffix Tree/Array
```

## "I Need Fast..." Scenarios

### Fast Lookup

| Requirement | Best Choice | Why | Time | Space |
|------------|-------------|-----|------|-------|
| Lookup by key | Hash Table | O(1) average access | O(1) | O(n) |
| Lookup + sorted order | Tree Map | O(log n) access + sorted | O(log n) | O(n) |
| Lookup by index | Array/List | Direct index access | O(1) | O(n) |
| Lookup by range | Tree Map or Segment Tree | Efficient range queries | O(log n) | O(n) |
| Lookup by prefix | Trie | Common prefix matching | O(m) | O(n×m) |
| Lookup in text | Suffix Tree/Array | Pattern matching | O(m) | O(n²) |

*m = key/pattern length, n = number of items*

### Fast Insert

| Requirement | Best Choice | Why | Time | Space |
|------------|-------------|-----|------|-------|
| Insert at end | Dynamic Array | Amortized constant time | O(1)* | O(n) |
| Insert at beginning | Linked List or Deque | No shifting needed | O(1) | O(n) |
| Insert in middle | Linked List | Just pointer updates | O(1)** | O(n) |
| Insert maintaining order | Tree Set | Self-balancing | O(log n) | O(n) |
| Insert with priority | Heap | Efficient heapify | O(log n) | O(n) |
| Insert unique only | Hash Set | Duplicate detection | O(1) | O(n) |

*Amortized, **After finding position*

### Fast Delete

| Requirement | Best Choice | Why | Time | Space |
|------------|-------------|-----|------|-------|
| Delete at end | Dynamic Array | Just decrement size | O(1) | O(n) |
| Delete at beginning | Linked List or Deque | No shifting | O(1) | O(n) |
| Delete in middle | Linked List | Pointer updates | O(1)* | O(n) |
| Delete by key | Hash Table | Direct access | O(1) | O(n) |
| Delete by value | Hash Set or Tree Set | Efficient search + delete | O(1) or O(log n) | O(n) |
| Delete min/max | Heap | Root extraction | O(log n) | O(n) |

*After finding node*

### Fast Traversal

| Requirement | Best Choice | Why | Time | Space |
|------------|-------------|-----|------|-------|
| Sequential access | Array or Linked List | Cache-friendly / Simple | O(n) | O(n) |
| Sorted traversal | Tree Set | In-order traversal | O(n) | O(n) |
| Level-order | Tree (BFS) | Queue-based traversal | O(n) | O(w)** |
| Depth-first | Tree/Graph (DFS) | Stack-based traversal | O(n) | O(h)*** |
| All paths | Graph | Backtracking | O(V+E) | O(V+E) |

**w = max width, ***h = height*

## Operation-Based Comparison

### Core Operations

| Data Structure | Access | Search | Insert | Delete | Space |
|---------------|--------|--------|--------|--------|-------|
| **Array** | O(1) | O(n) | O(n) | O(n) | O(n) |
| **Dynamic Array** | O(1) | O(n) | O(1)* | O(n) | O(n) |
| **Linked List** | O(n) | O(n) | O(1)** | O(1)** | O(n) |
| **Doubly Linked** | O(n) | O(n) | O(1)** | O(1)** | O(n) |
| **Skip List** | O(log n) | O(log n) | O(log n) | O(log n) | O(n log n) |
| **Hash Table** | - | O(1)*** | O(1)*** | O(1)*** | O(n) |
| **Binary Search Tree** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| **AVL Tree** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| **Red-Black Tree** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| **B-Tree** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| **Stack** | O(n) | O(n) | O(1) | O(1) | O(n) |
| **Queue** | O(n) | O(n) | O(1) | O(1) | O(n) |
| **Deque** | O(1)**** | O(n) | O(1) | O(1) | O(n) |
| **Priority Queue (Heap)** | O(1)***** | O(n) | O(log n) | O(log n) | O(n) |
| **Trie** | O(m) | O(m) | O(m) | O(m) | O(ALPHABET×n×m) |
| **Suffix Tree** | O(m) | O(m) | - | - | O(n²) |

*Amortized at end, **After finding position, ***Average case, ****At ends, *****Access min/max only*

### Specialized Operations

| Data Structure | Special Operation | Time | Use Case |
|---------------|-------------------|------|----------|
| **Heap** | Get min/max | O(1) | Priority queues, scheduling |
| **Heap** | Extract min/max | O(log n) | Event processing |
| **Hash Table** | Check membership | O(1) | Deduplication, caching |
| **Tree Map** | Range query | O(log n + k) | Date ranges, scores |
| **Tree Set** | Floor/ceiling | O(log n) | Nearest neighbor |
| **Trie** | Prefix search | O(m) | Autocomplete, spell check |
| **Suffix Tree** | Substring match | O(m) | Text search, bioinformatics |
| **Disjoint Set** | Union | O(α(n))* | Connected components |
| **Disjoint Set** | Find | O(α(n))* | Cycle detection |
| **Segment Tree** | Range query | O(log n) | Sum/min/max in range |
| **Fenwick Tree** | Prefix sum | O(log n) | Cumulative frequency |
| **Bloom Filter** | Membership test | O(k) | Fast probabilistic lookup |

*α(n) = inverse Ackermann function, effectively O(1)*

## Use Case Mapping

### Caching

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Simple cache | Hash Table | Fast lookup by key |
| LRU cache | Hash Table + Doubly Linked List | O(1) access + eviction |
| LFU cache | Hash Table + Min Heap | O(log n) eviction by frequency |
| TTL cache | Hash Table + Priority Queue | Expire by timestamp |

### Collections

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Shopping cart | Dynamic Array or Linked List | Easy add/remove |
| Unique tags | Hash Set | No duplicates, fast lookup |
| Sorted leaderboard | Tree Set or Heap | Maintain order |
| Recent items | Deque | FIFO with fixed size |
| Undo history | Stack | LIFO operations |

### Search & Filtering

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Autocomplete | Trie | Prefix matching |
| Full-text search | Inverted Index (Hash Table) | Fast keyword lookup |
| Range queries | Segment Tree or Tree Map | Efficient range operations |
| Nearest neighbor | K-D Tree or Ball Tree | Spatial queries |
| Spell checker | Trie + Edit Distance | Prefix + fuzzy matching |

### Scheduling & Events

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Task queue | Queue | FIFO processing |
| Priority tasks | Priority Queue (Heap) | Always process highest priority |
| Delayed tasks | Priority Queue by time | Execute when ready |
| Event log | Dynamic Array | Append-only, indexed access |
| Rate limiting | Circular Buffer or Deque | Sliding window |

### Graph Problems

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Social network | Adjacency List | Sparse graphs |
| Road network | Adjacency List | Weighted edges |
| Dependencies | Directed Acyclic Graph (DAG) | Topological sort |
| Shortest path | Priority Queue + Graph | Dijkstra's algorithm |
| Connected components | Disjoint Set | Union-find operations |

### Database Indexing

| Requirement | Data Structure | Why |
|------------|---------------|-----|
| Primary key | Hash Table or B-Tree | Fast exact match |
| Range queries | B-Tree or B+ Tree | Sorted range access |
| Full-text index | Inverted Index | Keyword search |
| Geospatial | R-Tree or Quad Tree | Spatial indexing |
| Time series | Log-Structured Merge Tree | Append-optimized |

## Memory vs Speed Tradeoffs

### Space-Efficient Choices

| Scenario | Compact Choice | Spacious Alternative | Savings |
|----------|---------------|---------------------|---------|
| Small collections | Array | Hash Table | Avoid hash overhead |
| Sequential access | Array | Linked List | No pointer overhead |
| Dense keys | Array | Hash Table | Direct indexing |
| Sorted data | Sorted Array | Tree | No node overhead |
| Unique items | Bit Set | Hash Set | 1 bit per item |
| Large text | Suffix Array | Suffix Tree | O(n) vs O(n²) |

### Speed-Optimized Choices

| Scenario | Fast Choice | Slower Alternative | Speedup |
|----------|------------|-------------------|---------|
| Random access | Array | Linked List | O(1) vs O(n) |
| Frequent inserts | Linked List | Array | O(1) vs O(n) |
| Key lookups | Hash Table | Array search | O(1) vs O(n) |
| Sorted access | Tree | Sorted Array | O(log n) insert vs O(n) |
| Priority operations | Heap | Sorted Array | O(log n) vs O(n) |
| String matching | Suffix Tree | Naive search | O(m) vs O(n×m) |

### Balanced Tradeoffs

| Use Case | Balanced Choice | Why |
|----------|----------------|-----|
| General collection | Dynamic Array | Good at most things |
| Sorted collection | Tree Set | Balanced insert/search |
| Key-value store | Hash Table | Fast with reasonable memory |
| Text processing | Trie | Good space/speed for strings |
| Graph representation | Adjacency List | Works for sparse/dense |

## Language-Specific Implementations

### JavaScript/TypeScript

| Data Structure | Built-in | Library |
|---------------|----------|---------|
| Dynamic Array | `Array` | - |
| Hash Table | `Map`, `Object` | - |
| Hash Set | `Set` | - |
| Linked List | - | Custom |
| Tree | - | Custom |
| Heap | - | Custom |
| Trie | - | Custom |

### Python

| Data Structure | Built-in | Library |
|---------------|----------|---------|
| Dynamic Array | `list` | - |
| Hash Table | `dict` | - |
| Hash Set | `set` | - |
| Linked List | - | `collections.deque` |
| Tree | - | Custom or `bintrees` |
| Heap | - | `heapq` |
| Trie | - | Custom |

### Java

| Data Structure | Built-in |
|---------------|----------|
| Dynamic Array | `ArrayList` |
| Linked List | `LinkedList` |
| Hash Table | `HashMap` |
| Hash Set | `HashSet` |
| Tree Map | `TreeMap` |
| Tree Set | `TreeSet` |
| Stack | `Stack` |
| Queue | `Queue` interface |
| Deque | `ArrayDeque` |
| Heap | `PriorityQueue` |

### PHP

| Data Structure | Built-in | SPL |
|---------------|----------|-----|
| Dynamic Array | `array` | - |
| Hash Table | `array` | - |
| Linked List | - | `SplDoublyLinkedList` |
| Stack | - | `SplStack` |
| Queue | - | `SplQueue` |
| Heap | - | `SplHeap`, `SplMinHeap`, `SplMaxHeap` |
| Priority Queue | - | `SplPriorityQueue` |

## Common Mistakes

### Using the Wrong Structure

| Mistake | Problem | Better Choice |
|---------|---------|---------------|
| Array for frequent inserts | O(n) insertion | Linked List |
| Linked List for random access | O(n) access | Array |
| Array for membership tests | O(n) lookup | Hash Set |
| Unsorted for searches | O(n) search | Hash Table or Tree |
| Hash Table for ordered data | No order guarantee | Tree Map |
| Tree for simple key-value | Overhead | Hash Table |

### Premature Optimization

| Scenario | Common Mistake | When It's Actually Fine |
|----------|---------------|------------------------|
| Small data | Using complex structures | Array/List works fine |
| Rare operations | Optimizing infrequent ops | Focus on common case |
| Known max size | Dynamic structure | Fixed Array is simpler |
| Simple iteration | Complex traversal | Linear scan is clear |

### Ignoring Hidden Costs

| Structure | Hidden Cost | Impact |
|-----------|-------------|--------|
| Hash Table | Hash computation | Can be expensive for complex keys |
| Linked List | Cache misses | Poor CPU cache utilization |
| Tree | Recursion overhead | Stack space in traversals |
| Dynamic Array | Reallocation | Periodic O(n) copy |
| Trie | Memory per node | Can be huge for large alphabets |

## Rules of Thumb

### When O(n²) is Actually Fine

- **n < 100**: Quadratic is fast enough
- **Once-only**: Initialization or setup
- **Simple code**: Much easier to understand
- **Hot path**: Not in critical loops

### When to Use Array Despite Inefficiency

- **Small collections**: <100 items
- **Append-mostly**: Rare inserts in middle
- **Sequential access**: Iterating often
- **Cache matters**: Performance critical

### When to Use Linked List

- **Many inserts**: Frequently add/remove
- **Unknown size**: Can't predict count
- **Mid-list ops**: Insert/delete in middle
- **Memory steady**: No growth/shrink

### When to Use Hash Table

- **Fast lookup**: Need O(1) access
- **Unique keys**: Natural key-value pairing
- **Order doesn't matter**: Iteration order irrelevant
- **Have space**: Can afford memory overhead

### When to Use Tree

- **Sorted order**: Need ordered iteration
- **Range queries**: Min/max or ranges
- **Balanced access**: Good read/write balance
- **Predictable**: Guaranteed O(log n)

## Performance Patterns

### The Access Pattern Matters

| Access Pattern | Best Choice |
|---------------|-------------|
| Sequential (front to back) | Array |
| Random (any order) | Hash Table |
| Sorted order | Tree |
| Recent items | Deque or Ring Buffer |
| Priority order | Heap |

### The Size Matters

| Size | General Advice |
|------|---------------|
| < 10 items | Use Array, simple is best |
| 10-1000 items | Hash Table or Array based on ops |
| 1000-100K items | Consider space/speed tradeoff |
| 100K-1M items | Hash Table or Tree, avoid O(n²) |
| > 1M items | External storage, B-Trees, indices |

### The Change Rate Matters

| Change Rate | Best Choice |
|------------|-------------|
| Read-heavy | Array or Hash Table |
| Write-heavy | Linked List or Tree |
| Balanced | Tree or Hash Table |
| Append-only | Dynamic Array |
| Insert-anywhere | Linked List |

## Quick Reference Cards

### I Need to Store...

```
Numbers/primitives → Array or Hash Set
Key-value pairs → Hash Table
Sorted numbers → Tree Set or Heap
Unique items → Hash Set
Ordered items → Linked List or Array
Hierarchical data → Tree
Network data → Graph
Text/strings → Trie (if prefix matching)
```

### I Need to Optimize for...

```
Speed → Hash Table
Memory → Array or Bit Set
Sorted access → Tree
Fast insert → Linked List
Fast delete → Linked List or Hash Table
Fast search → Hash Table or Tree
All operations → Tree (balanced tradeoff)
```

---

**Remember**: The "best" data structure depends on your specific use case. Profile real usage patterns and optimize bottlenecks, not hypotheticals.
