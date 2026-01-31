# Disjoint Set (Union-Find)

## Overview

A disjoint set (also called Union-Find) is a data structure that tracks a set of elements partitioned into non-overlapping subsets. It supports two primary operations: finding which set an element belongs to (Find) and merging two sets (Union). With path compression and union by rank, both operations achieve near-constant amortized time.

## Properties

- **Disjoint partitions**: Each element belongs to exactly one set
- **Representative**: Each set has a unique representative element
- **Forest representation**: Sets represented as trees
- **Dynamic equivalence**: Can merge sets but not split them
- **Near O(1) operations**: With optimizations, almost constant time

### Inverse Ackermann Function

With both optimizations, operations take O(α(n)) time where α is the inverse Ackermann function:
- α(n) ≤ 4 for any practical n (up to 10^80)
- Effectively constant time

## Time Complexity

| Operation    | Without Optimization | With Path Compression | With Both* |
|--------------|----------------------|-----------------------|------------|
| MakeSet      | O(1)                 | O(1)                  | O(1)       |
| Find         | O(n)                 | O(log n) amortized    | O(α(n))*   |
| Union        | O(n)                 | O(log n) amortized    | O(α(n))*   |

*α(n) is inverse Ackermann function, effectively constant

## Space Complexity

O(n) for n elements. Each element stores a parent pointer and optionally a rank/size.

## Operations

### Basic Structure

```pseudocode
class DisjointSet {
    parent[]    // parent[i] = parent of element i
    rank[]      // rank[i] = upper bound on height of subtree rooted at i

    function constructor(n) {
        this.parent = new Array(n)
        this.rank = new Array(n).fill(0)

        // Initially, each element is its own parent (separate set)
        for i from 0 to n - 1 {
            this.parent[i] = i
        }
    }
}
```

### Find (without optimization)

```pseudocode
function findSimple(x) {
    while this.parent[x] != x {
        x = this.parent[x]
    }
    return x
}
```

### Find with Path Compression

Flattens the tree during find for future speedup:

```pseudocode
function find(x) {
    if this.parent[x] != x {
        this.parent[x] = find(this.parent[x])  // Path compression
    }
    return this.parent[x]
}

// Iterative version with path compression
function findIterative(x) {
    root = x

    // Find root
    while this.parent[root] != root {
        root = this.parent[root]
    }

    // Compress path
    while this.parent[x] != root {
        next = this.parent[x]
        this.parent[x] = root
        x = next
    }

    return root
}
```

### Union (without optimization)

```pseudocode
function unionSimple(x, y) {
    rootX = find(x)
    rootY = find(y)

    if rootX != rootY {
        this.parent[rootX] = rootY
    }
}
```

### Union by Rank

Attach smaller tree under larger tree:

```pseudocode
function union(x, y) {
    rootX = find(x)
    rootY = find(y)

    if rootX == rootY {
        return false  // Already in same set
    }

    // Union by rank
    if this.rank[rootX] < this.rank[rootY] {
        this.parent[rootX] = rootY
    } else if this.rank[rootX] > this.rank[rootY] {
        this.parent[rootY] = rootX
    } else {
        this.parent[rootY] = rootX
        this.rank[rootX] = this.rank[rootX] + 1
    }

    return true
}
```

### Union by Size

Alternative: attach smaller set under larger set:

```pseudocode
class DisjointSetBySize {
    parent[]
    size[]      // size[i] = size of set if i is root

    function constructor(n) {
        this.parent = new Array(n)
        this.size = new Array(n).fill(1)

        for i from 0 to n - 1 {
            this.parent[i] = i
        }
    }

    function find(x) {
        if this.parent[x] != x {
            this.parent[x] = find(this.parent[x])
        }
        return this.parent[x]
    }

    function union(x, y) {
        rootX = find(x)
        rootY = find(y)

        if rootX == rootY {
            return false
        }

        // Union by size
        if this.size[rootX] < this.size[rootY] {
            this.parent[rootX] = rootY
            this.size[rootY] = this.size[rootY] + this.size[rootX]
        } else {
            this.parent[rootY] = rootX
            this.size[rootX] = this.size[rootX] + this.size[rootY]
        }

        return true
    }

    function getSize(x) {
        return this.size[find(x)]
    }
}
```

## Implementation

### Complete Union-Find

```pseudocode
class UnionFind {
    parent[]
    rank[]
    count       // Number of disjoint sets

    function constructor(n) {
        this.parent = new Array(n)
        this.rank = new Array(n).fill(0)
        this.count = n

        for i from 0 to n - 1 {
            this.parent[i] = i
        }
    }

    function find(x) {
        if this.parent[x] != x {
            this.parent[x] = find(this.parent[x])
        }
        return this.parent[x]
    }

    function union(x, y) {
        rootX = find(x)
        rootY = find(y)

        if rootX == rootY {
            return false
        }

        if this.rank[rootX] < this.rank[rootY] {
            this.parent[rootX] = rootY
        } else if this.rank[rootX] > this.rank[rootY] {
            this.parent[rootY] = rootX
        } else {
            this.parent[rootY] = rootX
            this.rank[rootX]++
        }

        this.count--
        return true
    }

    function connected(x, y) {
        return find(x) == find(y)
    }

    function getCount() {
        return this.count
    }

    function getSetSize(x) {
        // Requires union by size variant
    }
}
```

### Map-Based Union-Find

For non-integer elements:

```pseudocode
class MapUnionFind {
    parent = new Map()
    rank = new Map()

    function makeSet(x) {
        if not this.parent.has(x) {
            this.parent.set(x, x)
            this.rank.set(x, 0)
        }
    }

    function find(x) {
        if not this.parent.has(x) {
            makeSet(x)
        }

        if this.parent.get(x) != x {
            this.parent.set(x, find(this.parent.get(x)))
        }
        return this.parent.get(x)
    }

    function union(x, y) {
        rootX = find(x)
        rootY = find(y)

        if rootX == rootY {
            return false
        }

        rankX = this.rank.get(rootX)
        rankY = this.rank.get(rootY)

        if rankX < rankY {
            this.parent.set(rootX, rootY)
        } else if rankX > rankY {
            this.parent.set(rootY, rootX)
        } else {
            this.parent.set(rootY, rootX)
            this.rank.set(rootX, rankX + 1)
        }

        return true
    }

    function connected(x, y) {
        return find(x) == find(y)
    }
}
```

## Classic Applications

### Kruskal's MST Algorithm

```pseudocode
function kruskal(graph) {
    edges = graph.getAllEdges()
    edges.sortByWeight()

    uf = new UnionFind(graph.vertexCount)
    mst = []

    for (u, v, weight) in edges {
        if uf.find(u) != uf.find(v) {
            uf.union(u, v)
            mst.append((u, v, weight))

            if mst.length == graph.vertexCount - 1 {
                break
            }
        }
    }

    return mst
}
```

### Connected Components

```pseudocode
function countConnectedComponents(n, edges) {
    uf = new UnionFind(n)

    for (u, v) in edges {
        uf.union(u, v)
    }

    return uf.getCount()
}
```

### Cycle Detection in Undirected Graph

```pseudocode
function hasCycle(n, edges) {
    uf = new UnionFind(n)

    for (u, v) in edges {
        if uf.find(u) == uf.find(v) {
            return true  // Adding this edge creates cycle
        }
        uf.union(u, v)
    }

    return false
}
```

### Detect Redundant Connection

```pseudocode
function findRedundantConnection(edges) {
    n = edges.length
    uf = new UnionFind(n + 1)

    for (u, v) in edges {
        if uf.find(u) == uf.find(v) {
            return [u, v]  // This edge creates cycle
        }
        uf.union(u, v)
    }

    return null
}
```

### Number of Islands (2D Grid)

```pseudocode
function numIslands(grid) {
    if grid.isEmpty() {
        return 0
    }

    rows = grid.length
    cols = grid[0].length
    uf = new UnionFind(rows * cols)
    waterCount = 0

    function index(r, c) {
        return r * cols + c
    }

    directions = [[0, 1], [1, 0]]  // Right and down only

    for r from 0 to rows - 1 {
        for c from 0 to cols - 1 {
            if grid[r][c] == '0' {
                waterCount++
                continue
            }

            for (dr, dc) in directions {
                nr = r + dr
                nc = c + dc
                if nr < rows and nc < cols and grid[nr][nc] == '1' {
                    uf.union(index(r, c), index(nr, nc))
                }
            }
        }
    }

    return uf.getCount() - waterCount
}
```

## Use Cases

- **Network connectivity**: Check if nodes are connected
- **Kruskal's algorithm**: Minimum spanning tree
- **Percolation**: Physics simulation
- **Image processing**: Connected component labeling
- **Social networks**: Friend groups, communities
- **Equivalence classes**: Compiler type checking
- **Game development**: Territory control

## Advantages

- **Near O(1) operations**: With both optimizations
- **Simple implementation**: Easy to code correctly
- **Space efficient**: Just two arrays
- **Dynamic**: Supports incremental additions
- **No complex rebalancing**: Unlike trees

## Disadvantages

- **No split operation**: Cannot separate merged sets
- **No enumeration**: Cannot list all elements in a set efficiently
- **Fixed element set**: Elements determined at initialization
- **One-directional**: Can only merge, not undo

## Comparison with Alternatives

| Aspect          | Union-Find  | DFS/BFS     | Adjacency List |
|-----------------|-------------|-------------|----------------|
| Find connected  | O(α(n))     | O(V + E)    | O(V + E)       |
| Union/Add edge  | O(α(n))     | O(1)        | O(1)           |
| Query connected | O(α(n))     | O(V + E)    | O(V + E)       |
| Dynamic queries | Excellent   | Poor        | Poor           |
| Space           | O(n)        | O(V + E)    | O(V + E)       |
| List components | O(n)        | O(V + E)    | O(V + E)       |

## Common Pitfalls

- **Forgetting path compression**: Much slower without it
- **Wrong initialization**: Each element must be its own parent initially
- **Off-by-one**: Array indices vs element values
- **Redundant finds**: Find returns root, no need to find again
- **Not checking same set**: Union should check if already connected
- **Assuming split exists**: Cannot undo unions

## Related Structures

- **Persistent Union-Find**: Supports undo operations
- **Weighted Quick Union**: Track size for weighted merging
- **Link-Cut Trees**: Supports dynamic graph operations
- **Euler Tour Trees**: Alternative for dynamic connectivity

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
