# Complexity Cheat Sheet

Quick reference for algorithm and data structure complexity analysis.

## Big-O Notation Explained

### What It Means

Big-O describes how performance scales as input size grows.

```
O(1)       → Constant      → Same time regardless of input size
O(log n)   → Logarithmic   → Doubles input, adds constant time
O(n)       → Linear        → Doubles input, doubles time
O(n log n) → Linearithmic  → Efficient sorting algorithms
O(n²)      → Quadratic     → Doubles input, 4x time
O(n³)      → Cubic         → Doubles input, 8x time
O(2ⁿ)      → Exponential   → Adds one item, doubles time
O(n!)      → Factorial     → Unusable for n > 10
```

### Visual Scale

For n = 100:

```
O(1)       → 1 operation
O(log n)   → 7 operations
O(n)       → 100 operations
O(n log n) → 700 operations
O(n²)      → 10,000 operations
O(n³)      → 1,000,000 operations
O(2ⁿ)      → 1.27 × 10³⁰ operations (universe age in nanoseconds)
O(n!)      → 9.33 × 10¹⁵⁷ operations (more than atoms in universe)
```

### Growth Comparison

| n | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) |
|---|----------|------|------------|-------|-------|
| 10 | 3 | 10 | 33 | 100 | 1,024 |
| 100 | 7 | 100 | 664 | 10,000 | 1.27×10³⁰ |
| 1,000 | 10 | 1,000 | 9,966 | 1,000,000 | ∞ |
| 10,000 | 13 | 10,000 | 132,877 | 100,000,000 | ∞ |
| 100,000 | 17 | 100,000 | 1,660,964 | 10,000,000,000 | ∞ |

### Common Complexities Ranked

```
Excellent: O(1) < O(log n)
Good:      O(n) < O(n log n)
Fair:      O(n²) < O(n³)
Bad:       O(2ⁿ) < O(n!)
```

## Data Structure Operations

### Master Comparison Table

| Data Structure | Access | Search | Insert | Delete | Space | Notes |
|---------------|--------|--------|--------|--------|-------|-------|
| **Array** | O(1) | O(n) | O(n) | O(n) | O(n) | Fast access, slow modify |
| **Dynamic Array** | O(1) | O(n) | O(1)* | O(n) | O(n) | Amortized append |
| **Singly Linked List** | O(n) | O(n) | O(1)** | O(1)** | O(n) | Fast insert at position |
| **Doubly Linked List** | O(n) | O(n) | O(1)** | O(1)** | O(n) | Bidirectional traversal |
| **Skip List** | O(log n) | O(log n) | O(log n) | O(log n) | O(n log n) | Probabilistic balancing |
| **Hash Table** | - | O(1)*** | O(1)*** | O(1)*** | O(n) | Best for key lookup |
| **BST (unbalanced)** | O(n)**** | O(n)**** | O(n)**** | O(n)**** | O(n) | Can degrade to list |
| **BST (balanced)** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | AVL, Red-Black trees |
| **B-Tree** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | Database indexes |
| **Stack** | O(n) | O(n) | O(1) | O(1) | O(n) | LIFO access |
| **Queue** | O(n) | O(n) | O(1) | O(1) | O(n) | FIFO access |
| **Deque** | O(1)***** | O(n) | O(1) | O(1) | O(n) | Double-ended queue |
| **Binary Heap** | O(1)****** | O(n) | O(log n) | O(log n) | O(n) | Priority queue |
| **Fibonacci Heap** | O(1) | O(n) | O(1) | O(log n)* | O(n) | Amortized operations |
| **Trie** | O(m) | O(m) | O(m) | O(m) | O(ALPHABET×n×m) | m = key length |
| **Suffix Tree** | O(m) | O(m) | O(n) | - | O(n²) | Text search |
| **Suffix Array** | O(m log n) | O(m log n) | - | - | O(n) | Compact suffix tree |
| **Disjoint Set** | - | O(α(n)) | O(α(n)) | - | O(n) | Union-find |
| **Bloom Filter** | - | O(k) | O(k) | - | O(m) | Probabilistic |
| **LRU Cache** | O(1) | O(1) | O(1) | O(1) | O(n) | Hash + Linked List |

*Amortized, **After finding position, ***Average case, ****Worst case, *****At ends only, ******Min/max access only*

### Specialized Structure Operations

| Data Structure | Special Operation | Complexity | Use Case |
|---------------|-------------------|-----------|----------|
| **Segment Tree** | Range query | O(log n) | Sum/min/max in range |
| **Segment Tree** | Point update | O(log n) | Update single element |
| **Fenwick Tree** | Prefix sum | O(log n) | Cumulative frequency |
| **Fenwick Tree** | Range update | O(log n) | Add to range |
| **K-D Tree** | Nearest neighbor | O(log n) avg | Spatial queries |
| **R-Tree** | Range query | O(log n + k) | Geospatial search |
| **Quad Tree** | Point query | O(log n) | 2D spatial indexing |
| **Interval Tree** | Overlapping intervals | O(log n + k) | Scheduling |
| **Splay Tree** | Amortized access | O(log n)* | Self-adjusting |
| **Treap** | Expected operations | O(log n) | Randomized BST |

*Amortized over sequence of operations*

## Algorithm Complexities

### Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable | Notes |
|-----------|------|---------|-------|-------|--------|-------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Educational only |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Good for small/nearly sorted |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No | Simple, always same time |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Predictable, stable |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Fast average case |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No | In-place, predictable |
| **Counting Sort** | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | Limited range |
| **Radix Sort** | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes | Fixed-length keys |
| **Bucket Sort** | O(n+k) | O(n+k) | O(n²) | O(n+k) | Yes | Uniformly distributed |
| **Tim Sort** | O(n) | O(n log n) | O(n log n) | O(n) | Yes | Python/Java default |

*k = range of values, d = number of digits*

### Search Algorithms

| Algorithm | Best | Average | Worst | Space | Requirements |
|-----------|------|---------|-------|-------|--------------|
| **Linear Search** | O(1) | O(n) | O(n) | O(1) | None |
| **Binary Search** | O(1) | O(log n) | O(log n) | O(1) | Sorted array |
| **Jump Search** | O(1) | O(√n) | O(√n) | O(1) | Sorted array |
| **Interpolation Search** | O(1) | O(log log n) | O(n) | O(1) | Sorted, uniform |
| **Exponential Search** | O(1) | O(log n) | O(log n) | O(1) | Sorted, unbounded |
| **Ternary Search** | O(1) | O(log₃ n) | O(log₃ n) | O(1) | Sorted array |
| **Fibonacci Search** | O(1) | O(log n) | O(log n) | O(1) | Sorted array |

### Graph Algorithms

| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| **BFS** | O(V+E) | O(V) | Shortest path (unweighted) |
| **DFS** | O(V+E) | O(V) | Traversal, cycle detection |
| **Dijkstra** | O((V+E) log V) | O(V) | Shortest path (positive weights) |
| **Bellman-Ford** | O(VE) | O(V) | Shortest path (negative weights) |
| **Floyd-Warshall** | O(V³) | O(V²) | All-pairs shortest path |
| **Kruskal** | O(E log E) | O(V) | Minimum spanning tree |
| **Prim** | O(E log V) | O(V) | Minimum spanning tree |
| **Topological Sort** | O(V+E) | O(V) | DAG ordering |
| **Strongly Connected** | O(V+E) | O(V) | Tarjan's/Kosaraju's |
| **A\*** | O(E) | O(V) | Heuristic shortest path |

*V = vertices, E = edges*

### String Algorithms

| Algorithm | Preprocessing | Matching | Space | Use Case |
|-----------|--------------|----------|-------|----------|
| **Naive** | O(1) | O(nm) | O(1) | Simple patterns |
| **KMP** | O(m) | O(n) | O(m) | Single pattern |
| **Boyer-Moore** | O(m+σ) | O(n/m) best | O(m+σ) | Single pattern |
| **Rabin-Karp** | O(m) | O(nm) worst | O(1) | Multiple patterns |
| **Aho-Corasick** | O(Σm) | O(n+z) | O(Σm) | Multiple patterns |
| **Suffix Array** | O(n log n) | O(m log n) | O(n) | Repeated searches |
| **Suffix Tree** | O(n) | O(m) | O(n²) | Many operations |

*n = text length, m = pattern length, σ = alphabet size, z = matches*

### Dynamic Programming

| Problem Type | Time | Space | Notes |
|-------------|------|-------|-------|
| **Fibonacci** | O(n) | O(1) | Bottom-up |
| **Coin Change** | O(n×m) | O(n) | n = amount, m = coins |
| **Knapsack** | O(n×W) | O(n×W) | W = capacity |
| **LCS** | O(nm) | O(nm) | Longest common subsequence |
| **Edit Distance** | O(nm) | O(nm) | Levenshtein distance |
| **Matrix Chain** | O(n³) | O(n²) | Optimal parenthesization |
| **Longest Increasing** | O(n log n) | O(n) | Binary search optimization |

## Space Complexity

### Memory Usage by Data Structure

| Data Structure | Space per Item | Overhead | Total |
|---------------|----------------|----------|-------|
| **Array** | 1× | Minimal | O(n) |
| **Dynamic Array** | 1-2× | Capacity | O(n) |
| **Singly Linked List** | 1× + 1 pointer | High | O(n) |
| **Doubly Linked List** | 1× + 2 pointers | Higher | O(n) |
| **Hash Table** | 1× + pointer | Load factor | O(n) |
| **BST** | 1× + 2 pointers | Node overhead | O(n) |
| **Heap** | 1× | Minimal | O(n) |
| **Trie** | 1 char + children | Very high | O(ALPHABET×n×m) |

### Typical Pointer Sizes

| System | Pointer Size |
|--------|-------------|
| 32-bit | 4 bytes |
| 64-bit | 8 bytes |

### Example: 1M Integers

| Structure | Memory (64-bit) | Calculation |
|-----------|----------------|-------------|
| Array | 4 MB | 1M × 4 bytes |
| Linked List | 20 MB | 1M × (4 + 8 + 8) bytes |
| Hash Table | 8 MB (ideal) | 1M × 4 bytes × 2 (load factor) |
| BST | 20 MB | 1M × (4 + 8 + 8) bytes |

## Recursion Complexity

### Stack Space

| Recursion Type | Space Complexity | Example |
|---------------|------------------|---------|
| **Linear** | O(n) | Factorial, sum |
| **Binary** | O(log n) | Binary search |
| **Tree** | O(h) | Tree traversal (h = height) |
| **Memoized** | O(n) | Fibonacci with cache |
| **Tail recursive** | O(1)* | Optimized by compiler |

*If compiler optimizes tail calls*

### Tail Call Optimization

```javascript
// NOT tail recursive - O(n) space
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);  // Operation after recursive call
}

// Tail recursive - O(1) space (if optimized)
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc);  // Recursive call is last operation
}
```

## When Complexity Doesn't Matter

### Small Input Sizes

| Size | O(n²) Time | O(n³) Time | Verdict |
|------|-----------|-----------|---------|
| n < 10 | < 1 ms | < 1 ms | Any algorithm works |
| n < 100 | < 10 ms | < 1 sec | O(n²) acceptable |
| n < 1,000 | < 1 sec | Too slow | Need O(n log n) |
| n < 10,000 | Too slow | Way too slow | Need O(n) or O(log n) |

### One-Time Operations

- **Initialization**: O(n²) setup is fine if O(1) queries
- **Data loading**: Slow load + fast access = acceptable
- **Pre-computation**: Build index once, use many times

### Dominated Terms

When one operation is much more expensive:

```
O(n² + n log n) → O(n²)   // n² dominates
O(n log n + n) → O(n log n)   // n log n dominates
O(2ⁿ + n³) → O(2ⁿ)   // exponential dominates
```

### Hidden Constants

```
O(1000n) is slower than O(n log n) for n < 100
O(n) with expensive operations can be slower than O(n log n) with cheap ones
```

## Rule of Thumb Guidelines

### Performance Targets

| Operations per Second | Algorithm Class |
|----------------------|-----------------|
| 10⁸ - 10⁹ | O(1), O(log n) |
| 10⁷ - 10⁸ | O(n) |
| 10⁶ - 10⁷ | O(n log n) |
| 10⁴ - 10⁵ | O(n²) |
| 10² - 10³ | O(n³) |
| < 20 items | O(2ⁿ) |
| < 10 items | O(n!) |

### Modern CPU Performance

Rough estimates (varies by CPU, operation, cache hits):

- **Simple operation**: 1 nanosecond
- **Cache hit**: 1-10 nanoseconds
- **RAM access**: 100 nanoseconds
- **SSD access**: 10-100 microseconds
- **Network**: 1-100 milliseconds

### When O(n²) is Fine

1. **n < 100**: Usually runs in milliseconds
2. **Inner loop is simple**: Just comparisons/swaps
3. **Not in hot path**: Runs rarely
4. **Code clarity**: Much simpler than O(n log n) version
5. **One-time**: Initialization or setup

### When O(n²) is NOT Fine

1. **n > 1,000**: Noticeable delays
2. **Frequent operation**: In critical path
3. **User-facing**: Affects responsiveness
4. **Scalability**: Will grow with users
5. **Real-time**: Strict time constraints

### Optimization Priority

```
1. Correctness (it works)
2. Clarity (you can maintain it)
3. Performance (it's fast enough)
4. Optimization (make critical parts faster)
```

Only optimize when:
- [ ] Profiling shows this is a bottleneck
- [ ] Users are affected by the slowness
- [ ] Better algorithm exists without sacrificing clarity
- [ ] The optimization justifies added complexity

## Common Complexity Mistakes

### Mistake: Ignoring Inner Loops

```javascript
// Looks like O(n), actually O(n²)
for (let i = 0; i < n; i++) {
  array.includes(i);  // O(n) operation!
}

// Fix: Use Set for O(n)
const set = new Set(array);
for (let i = 0; i < n; i++) {
  set.has(i);  // O(1) operation
}
```

### Mistake: Hidden String Operations

```javascript
// Looks like O(n), actually O(n²)
let result = "";
for (let i = 0; i < n; i++) {
  result += array[i];  // String concatenation is O(n)!
}

// Fix: Use array join for O(n)
const parts = [];
for (let i = 0; i < n; i++) {
  parts.push(array[i]);
}
const result = parts.join("");
```

### Mistake: Forgetting Sorting Cost

```javascript
// Looks like O(n), actually O(n log n)
array.sort();
for (let item of array) {
  process(item);
}

// Total: O(n log n) + O(n) = O(n log n)
```

### Mistake: Underestimating Space

```javascript
// Time: O(n), but Space: O(2ⁿ) due to recursion branches
function allSubsets(array) {
  if (array.length === 0) return [[]];
  const first = array[0];
  const rest = array.slice(1);
  const subsetsWithoutFirst = allSubsets(rest);
  const subsetsWithFirst = subsetsWithoutFirst.map(s => [first, ...s]);
  return [...subsetsWithoutFirst, ...subsetsWithFirst];
}
```

## Quick Lookup

### Need Algorithm With...

```
Time O(1)? → Hash Table
Time O(log n)? → Binary Search, Balanced Tree
Time O(n)? → Linear scan, Hash Table building
Time O(n log n)? → Merge Sort, Quick Sort, Heap Sort
Space O(1)? → In-place algorithms
Space O(log n)? → Recursive with balanced tree
Space O(n)? → Hash Table, Merge Sort
```

### Common Complexity Formulas

```
Sum: 1 + 2 + ... + n = n(n+1)/2 = O(n²)
Powers of 2: 1 + 2 + 4 + ... + 2ⁿ = 2ⁿ⁺¹ - 1 = O(2ⁿ)
Log sum: log 1 + log 2 + ... + log n = log(n!) = O(n log n)
Nested loops: outer × inner = O(mn) or O(n²) if equal
Binary tree height: log₂(n+1) for complete tree
```

---

**Remember**: Big-O is about growth rate, not actual speed. Profile real code with real data to find actual bottlenecks.
