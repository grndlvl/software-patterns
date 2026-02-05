# Data Structure Cheatsheet

## By Operation Complexity

| Need | Structure | Lookup | Insert | Delete | Space | Ordered |
|------|-----------|--------|--------|--------|-------|---------|
| Fast key lookup | Hash Table | O(1)* | O(1)* | O(1)* | O(n) | No |
| Sorted + lookup | Red-Black Tree | O(log n) | O(log n) | O(log n) | O(n) | Yes |
| Priority access | Binary Heap | O(1) peek | O(log n) | O(log n) | O(n) | Partial |
| FIFO | Queue | O(1) front | O(1) | O(1) | O(n) | FIFO |
| LIFO | Stack | O(1) top | O(1) | O(1) | O(n) | LIFO |
| Range queries | Segment Tree | O(log n) | O(log n) | - | O(n) | Yes |
| Prefix search | Trie | O(k) | O(k) | O(k) | O(ALPHABET*n*k) | Lexical |
| Dynamic array | Dynamic Array | O(1) | O(1)* | O(n) | O(n) | Index |
| Fixed capacity | Array | O(1) | - | - | O(n) | Index |
| Frequent insert/delete | Linked List | O(n) | O(1)** | O(1)** | O(n) | No |
| Membership test | Hash Set | O(1)* | O(1)* | O(1)* | O(n) | No |
| Range sum/update | Fenwick Tree | O(log n) | O(log n) | O(log n) | O(n) | Yes |
| Prefix sum | Prefix Sum Array | O(1) | - | - | O(n) | Yes |
| Union/Find | Disjoint Set (Union-Find) | O(α(n))*** | O(α(n))*** | - | O(n) | No |
| LRU eviction | LRU Cache (Hash + DLL) | O(1) | O(1) | O(1) | O(n) | Access order |
| Spatial queries | KD-Tree | O(log n)† | O(log n)† | O(log n)† | O(n) | Spatial |
| Graph sparse | Adjacency List | O(V+E) traverse | O(1) edge | O(E) edge | O(V+E) | No |
| Graph dense | Adjacency Matrix | O(V²) traverse | O(1) edge | O(1) edge | O(V²) | No |
| Probabilistic membership | Bloom Filter | O(k) | O(k) | No | O(m) | No |
| 2D range query | 2D Segment Tree | O(log² n) | O(log² n) | O(log² n) | O(n²) | Yes |

*Amortized/average case (worst case O(n) for hash collisions)
**At known position (O(n) to find position)
***α(n) is inverse Ackermann function, effectively constant
†Average case (worst case O(n) with balanced tree)

---

## Quick Decision Tree

### **Start Here: What's your primary operation?**

#### **1. Key-Value Lookup**
```
Do you need order?
├─ NO  → Hash Table/HashMap/dict
└─ YES → Do you need range queries?
    ├─ NO  → Red-Black Tree (TreeMap)
    └─ YES → Segment Tree or Interval Tree
```

#### **2. Collection Access**
```
How do you access elements?
├─ By index     → Array or Dynamic Array
├─ First/Last   → Deque (double-ended queue)
├─ First only   → Queue
├─ Last only    → Stack
└─ By priority  → Heap (PriorityQueue)
```

#### **3. Membership Testing**
```
How many elements?
├─ Small (<100)          → Array with linear search
├─ Medium                → Hash Set
├─ Large + exact needed  → Hash Set
└─ Large + "maybe" ok    → Bloom Filter (space-efficient)
```

#### **4. String/Prefix Operations**
```
What operation?
├─ Exact string lookup  → Hash Set
├─ Prefix search        → Trie
├─ Substring search     → Suffix Tree/Array
├─ Autocomplete         → Trie
└─ Pattern matching     → KMP/Boyer-Moore (algorithm, not DS)
```

#### **5. Range Operations**
```
What do you need?
├─ Static range sum           → Prefix Sum Array
├─ Range sum + updates        → Fenwick Tree (BIT)
├─ Range min/max/sum + update → Segment Tree
├─ 2D range sum               → 2D Prefix Sum or 2D Segment Tree
└─ Range distinct count       → Mo's Algorithm or Wavelet Tree
```

#### **6. Graph Representation**
```
How dense is your graph?
├─ Sparse (E << V²)  → Adjacency List
├─ Dense (E ≈ V²)    → Adjacency Matrix
└─ Spatial/geometric → KD-Tree or Quadtree
```

#### **7. Dynamic Set Operations**
```
What do you need?
├─ Find which set contains element → Disjoint Set (Union-Find)
├─ Find min/max dynamically        → Balanced BST or Heap
├─ Find median dynamically         → Two Heaps
└─ Find k-th smallest dynamically  → Order Statistic Tree
```

---

## By Use Case

### **Caching**
| Eviction Policy | Data Structure | Complexity |
|-----------------|----------------|------------|
| LRU (Least Recently Used) | Hash + Doubly Linked List | O(1) all ops |
| LFU (Least Frequently Used) | Hash + Min Heap or DLL per frequency | O(log n) or O(1) |
| FIFO | Queue + Hash | O(1) all ops |
| Random | Array + Hash | O(1) all ops |

### **String Algorithms**
| Problem | Data Structure | Time | Space |
|---------|----------------|------|-------|
| Autocomplete | Trie | O(k) search | O(ALPHABET*n*k) |
| Pattern matching | KMP/Boyer-Moore | O(n+m) | O(m) |
| All suffixes | Suffix Array | O(n log n) build | O(n) |
| Substring search | Suffix Tree | O(m) search | O(n) |
| Dictionary lookup | Hash Set | O(k) | O(n*k) |
| Lexicographic order | Trie or Sorted Array | O(k) or O(log n) | varies |

### **Range Query Problems**
| Query Type | Update Type | Data Structure | Query Time | Update Time |
|------------|-------------|----------------|------------|-------------|
| Range sum | None | Prefix Sum Array | O(1) | - |
| Range sum | Point update | Fenwick Tree | O(log n) | O(log n) |
| Range sum | Range update | Segment Tree w/ lazy | O(log n) | O(log n) |
| Range min/max | Point update | Segment Tree | O(log n) | O(log n) |
| Range min/max | Range update | Segment Tree w/ lazy | O(log n) | O(log n) |
| Range GCD/LCM | None | Sparse Table | O(1) | O(n log n) preprocess |

### **Priority/Scheduling**
| Need | Data Structure | Operations |
|------|----------------|------------|
| Always need min/max | Binary Heap | O(log n) insert/delete, O(1) peek |
| Need both min and max | Two heaps or Min-Max Heap | O(log n) ops |
| Need k-th smallest | Order Statistic Tree | O(log n) all ops |
| Dynamic median | Two heaps (max + min) | O(log n) insert, O(1) median |
| Decrease key operation | Fibonacci Heap | O(1) amortized decrease-key |

### **Graph Problems**
| Problem Type | Data Structure | Notes |
|--------------|----------------|-------|
| Sparse graph traversal | Adjacency List | Most common, O(V+E) space |
| Dense graph | Adjacency Matrix | O(V²) space, O(1) edge check |
| Shortest path (Dijkstra) | Min Heap + Adjacency List | O((V+E) log V) |
| Minimum spanning tree | Union-Find + Edges | Kruskal's algorithm |
| Strongly connected components | Adjacency List + Stack | Tarjan's or Kosaraju's |
| Spatial nearest neighbor | KD-Tree | O(log n) average |

### **Geometric Problems**
| Problem | Data Structure | Complexity |
|---------|----------------|------------|
| Nearest neighbor | KD-Tree | O(log n) average |
| Range search (2D) | Quadtree or KD-Tree | O(log n + k) |
| Point in polygon | Ray casting or BST | O(log n) or O(n) |
| Interval overlap | Interval Tree | O(log n + k) |
| Segment intersection | Sweep Line + BST | O((n+k) log n) |

### **Specialized Caches**
| Use Case | Implementation | Why |
|----------|----------------|-----|
| Database query cache | LRU (Hash + DLL) | Recent queries likely to repeat |
| CDN cache | LRU or LFU | Popular content stays hot |
| CPU cache | LRU approximation | Temporal locality |
| Browser cache | Hybrid (LRU + size limit) | Balance frequency and size |
| DNS cache | TTL-based expiry | Data has inherent lifetime |

---

## Complexity Reference

### Array-Based Structures

| Structure | Access | Search | Insert | Delete | Space |
|-----------|--------|--------|--------|--------|-------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Dynamic Array | O(1) | O(n) | O(1)* | O(n) | O(n) |
| Sorted Array | O(1) | O(log n) | O(n) | O(n) | O(n) |

*Amortized O(1) at end, O(n) at arbitrary position

### Linked Structures

| Structure | Access | Search | Insert | Delete | Space |
|-----------|--------|--------|--------|--------|-------|
| Singly Linked List | O(n) | O(n) | O(1)** | O(1)** | O(n) |
| Doubly Linked List | O(n) | O(n) | O(1)** | O(1)** | O(n) |
| Skip List | O(log n) | O(log n) | O(log n) | O(log n) | O(n log n) |

**At known position

### Hash-Based Structures

| Structure | Lookup | Insert | Delete | Space | Ordered |
|-----------|--------|--------|--------|-------|---------|
| Hash Table | O(1)* | O(1)* | O(1)* | O(n) | No |
| Hash Set | O(1)* | O(1)* | O(1)* | O(n) | No |

*Average case, O(n) worst case with bad hash function

### Tree-Based Structures

| Structure | Search | Insert | Delete | Space | Balanced |
|-----------|--------|--------|--------|-------|----------|
| Binary Search Tree | O(log n)† | O(log n)† | O(log n)† | O(n) | No |
| AVL Tree | O(log n) | O(log n) | O(log n) | O(n) | Yes |
| Red-Black Tree | O(log n) | O(log n) | O(log n) | O(n) | Yes |
| B-Tree | O(log n) | O(log n) | O(log n) | O(n) | Yes |
| Splay Tree | O(log n)* | O(log n)* | O(log n)* | O(n) | No (self-adjusting) |

†Average case, O(n) worst case unbalanced

### Heap Structures

| Structure | Find Min | Insert | Delete Min | Decrease Key | Space |
|-----------|----------|--------|------------|--------------|-------|
| Binary Heap | O(1) | O(log n) | O(log n) | O(log n) | O(n) |
| Binomial Heap | O(log n) | O(1)* | O(log n) | O(log n) | O(n) |
| Fibonacci Heap | O(1) | O(1)* | O(log n)* | O(1)* | O(n) |

*Amortized

### Specialized Structures

| Structure | Primary Operation | Complexity | Space |
|-----------|-------------------|------------|-------|
| Trie | Prefix search | O(k) | O(ALPHABET*n*k) |
| Segment Tree | Range query | O(log n) | O(n) |
| Fenwick Tree | Prefix sum | O(log n) | O(n) |
| Disjoint Set | Union/Find | O(α(n)) | O(n) |
| Bloom Filter | Membership test | O(k) | O(m) |
| Suffix Array | Substring search | O(m log n) | O(n) |
| Suffix Tree | Substring search | O(m) | O(n) |

---

## Common Interview Patterns

### **Two Pointers**
- Data structures: Array, String
- Use when: Sorted array, palindrome, pairs with target sum

### **Sliding Window**
- Data structures: Array, Deque
- Use when: Subarray/substring problems, max/min in window

### **Fast & Slow Pointers**
- Data structures: Linked List
- Use when: Cycle detection, middle of list

### **Monotonic Stack**
- Data structures: Stack
- Use when: Next greater element, histogram area

### **Monotonic Queue**
- Data structures: Deque
- Use when: Sliding window max/min

### **Prefix Sum**
- Data structures: Array
- Use when: Range sum queries, subarray sum

### **Union-Find**
- Data structures: Disjoint Set
- Use when: Dynamic connectivity, graph components

### **Trie**
- Data structures: Trie
- Use when: Prefix problems, word search, autocomplete

### **Heap**
- Data structures: Priority Queue
- Use when: k-th largest/smallest, merge k sorted

### **Two Heaps**
- Data structures: Max Heap + Min Heap
- Use when: Median of stream, balance problems

---

## Space-Time Tradeoffs

### More Space → Faster Operations

| Problem | Time-Optimized (More Space) | Space-Optimized (Slower) |
|---------|----------------------------|--------------------------|
| Fibonacci | O(1) with lookup table | O(n) with recursion + memo |
| Range sum | O(1) with prefix sum array | O(n) recalculate each time |
| Pathfinding | O(1) with distance matrix | O(E) with adjacency list |
| Membership | O(1) with hash set | O(n) with array linear search |
| Duplicate detection | O(n) with hash set | O(n²) with nested loops |

### Approximate vs Exact

| Structure | Exact | Approximate (Less Space) |
|-----------|-------|--------------------------|
| Set membership | Hash Set O(n) space | Bloom Filter O(m) space, false positives |
| Distinct count | Hash Set O(n) space | HyperLogLog O(log log n) space, estimate |
| Frequency count | Hash Map O(n) space | Count-Min Sketch O(log n) space, estimate |
| Heavy hitters | Hash Map O(n) space | Count Sketch O(k) space for top-k |

---

## Language-Specific Implementations

### Python

| Data Structure | Python Implementation | Import |
|----------------|----------------------|--------|
| Dynamic Array | `list` | Built-in |
| Hash Table | `dict` | Built-in |
| Hash Set | `set` | Built-in |
| Ordered Map | `OrderedDict` | `collections` |
| Deque | `deque` | `collections` |
| Heap | `heapq` (min heap) | `heapq` |
| Counter | `Counter` | `collections` |
| Default Dict | `defaultdict` | `collections` |
| Sorted List | `SortedList` | `sortedcontainers` |

### JavaScript/TypeScript

| Data Structure | JS/TS Implementation | Notes |
|----------------|---------------------|-------|
| Dynamic Array | `Array` | Built-in |
| Hash Table | `Map` | Built-in (ES6+) |
| Hash Set | `Set` | Built-in (ES6+) |
| Ordered Map | `Map` | Maintains insertion order |
| Deque | Manual or library | No built-in |
| Heap | Library (e.g., `heap-js`) | No built-in |
| Queue | `Array` with push/shift | Or library |
| Stack | `Array` with push/pop | Built-in |

### Java

| Data Structure | Java Implementation | Package |
|----------------|---------------------|---------|
| Dynamic Array | `ArrayList` | `java.util` |
| Linked List | `LinkedList` | `java.util` |
| Hash Table | `HashMap` | `java.util` |
| Hash Set | `HashSet` | `java.util` |
| Sorted Map | `TreeMap` (Red-Black) | `java.util` |
| Sorted Set | `TreeSet` (Red-Black) | `java.util` |
| Priority Queue | `PriorityQueue` (Heap) | `java.util` |
| Deque | `ArrayDeque` | `java.util` |
| Stack | `Stack` or `ArrayDeque` | `java.util` |

### C++

| Data Structure | C++ Implementation | Header |
|----------------|-------------------|--------|
| Dynamic Array | `vector` | `<vector>` |
| Linked List | `list` (doubly) | `<list>` |
| Forward List | `forward_list` (singly) | `<forward_list>` |
| Hash Table | `unordered_map` | `<unordered_map>` |
| Hash Set | `unordered_set` | `<unordered_set>` |
| Sorted Map | `map` (Red-Black) | `<map>` |
| Sorted Set | `set` (Red-Black) | `<set>` |
| Multimap | `multimap` | `<map>` |
| Priority Queue | `priority_queue` (Heap) | `<queue>` |
| Deque | `deque` | `<deque>` |
| Stack | `stack` | `<stack>` |
| Queue | `queue` | `<queue>` |

### Go

| Data Structure | Go Implementation | Notes |
|----------------|------------------|-------|
| Dynamic Array | `[]T` (slice) | Built-in |
| Hash Table | `map[K]V` | Built-in |
| Hash Set | `map[K]struct{}` | Use map with empty value |
| Heap | `container/heap` | Interface-based |
| List | `container/list` (doubly) | Standard library |
| Ring | `container/ring` (circular) | Standard library |

### Rust

| Data Structure | Rust Implementation | Module |
|----------------|---------------------|--------|
| Dynamic Array | `Vec<T>` | `std::vec` |
| Linked List | `LinkedList<T>` | `std::collections` |
| Hash Table | `HashMap<K, V>` | `std::collections` |
| Hash Set | `HashSet<T>` | `std::collections` |
| Sorted Map | `BTreeMap<K, V>` | `std::collections` |
| Sorted Set | `BTreeSet<T>` | `std::collections` |
| Heap | `BinaryHeap<T>` | `std::collections` |
| Deque | `VecDeque<T>` | `std::collections` |

---

## Anti-Patterns

### **Don't Use This** → **Use This Instead**

| Anti-Pattern | Better Choice | Why |
|--------------|---------------|-----|
| Unsorted array for frequent search | Hash Set or Sorted Array + Binary Search | O(1) or O(log n) vs O(n) |
| Array for frequent insert/delete at start | Linked List or Deque | O(1) vs O(n) |
| Linked List for random access | Array or Dynamic Array | O(1) vs O(n) |
| Multiple searches for same membership | Hash Set lookup once | O(1) vs repeated O(n) |
| Recalculating range sums | Prefix Sum Array | O(1) query vs O(n) |
| Nested loops for duplicate detection | Hash Set | O(n) vs O(n²) |
| Linear search in sorted data | Binary Search | O(log n) vs O(n) |
| Array for priority operations | Heap | O(log n) vs O(n) |
| Rebuilding entire data structure | Incremental updates | Avoid O(n) rebuilds |

---

## When to Use Each Structure

### **Use Array/Dynamic Array when:**
- Need random access by index
- Size is mostly fixed or grows at end
- Iteration is frequent
- Cache locality is important

### **Use Linked List when:**
- Frequent insert/delete at known positions
- Don't need random access
- Size changes frequently at multiple points

### **Use Hash Table/Set when:**
- Need fast lookup by key
- Order doesn't matter
- Keys are hashable
- Average case O(1) is acceptable

### **Use Balanced Tree (Red-Black, AVL) when:**
- Need sorted order
- Need range queries
- Guaranteed O(log n) operations
- Order statistics (k-th element)

### **Use Heap when:**
- Only need min/max element
- Priority queue operations
- Don't need arbitrary access
- O(log n) insert/delete acceptable

### **Use Trie when:**
- Prefix-based operations
- Autocomplete features
- Dictionary lookups
- String-based collections

### **Use Segment Tree when:**
- Range queries with updates
- Intervals on a line
- Both point and range updates

### **Use Union-Find when:**
- Dynamic connectivity
- Grouping/clustering elements
- Minimum spanning tree
- Connected components

### **Use Bloom Filter when:**
- Space is critical
- False positives are acceptable
- Membership testing only (no deletion)
- Very large datasets

---

## Decision Framework Summary

1. **Start with the operation**: What do you do most?
2. **Consider order requirements**: Do you need sorted data?
3. **Analyze access patterns**: Random, sequential, by key?
4. **Check space constraints**: Can you afford O(n) extra space?
5. **Evaluate update frequency**: Static, occasional, or frequent updates?
6. **Think about edge cases**: Empty, single element, very large?
7. **Profile if uncertain**: Measure actual performance

**Default choices (80% of cases):**
- Key-value with no order: **Hash Table**
- Key-value with order: **Red-Black Tree** (TreeMap)
- Sequential access: **Array** or **Dynamic Array**
- Priority operations: **Binary Heap**
- Membership testing: **Hash Set**
- Queue/Stack: **Deque** (most flexible)

**When in doubt:** Start simple (Array, Hash Table), optimize later with profiling.
