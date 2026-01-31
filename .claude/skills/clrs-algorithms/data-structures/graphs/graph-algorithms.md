# Graph Algorithms

## Overview

This document covers fundamental graph algorithms: traversal (BFS, DFS), shortest paths (Dijkstra, Bellman-Ford, Floyd-Warshall), minimum spanning trees (Prim, Kruskal), and topological sorting. These algorithms form the foundation for solving complex problems in networks, scheduling, and optimization.

## Traversal Algorithms

### Breadth-First Search (BFS)

Explores vertices level by level, finding shortest paths in unweighted graphs.

```pseudocode
function bfs(graph, start) {
    visited = new Set()
    queue = new Queue()
    parent = new Map()      // For path reconstruction
    distance = new Map()    // Distance from start

    queue.enqueue(start)
    visited.add(start)
    distance.set(start, 0)

    while not queue.isEmpty() {
        vertex = queue.dequeue()
        process(vertex)

        for neighbor in graph.getNeighbors(vertex) {
            if neighbor not in visited {
                visited.add(neighbor)
                parent.set(neighbor, vertex)
                distance.set(neighbor, distance.get(vertex) + 1)
                queue.enqueue(neighbor)
            }
        }
    }

    return { visited, parent, distance }
}

// Reconstruct path from start to end
function reconstructPath(parent, start, end) {
    if end not in parent and end != start {
        return null  // No path exists
    }

    path = []
    current = end

    while current != null {
        path.prepend(current)
        current = parent.get(current)
    }

    return path
}
```

**Time Complexity**: O(V + E)
**Space Complexity**: O(V)

**Use Cases**:
- Shortest path in unweighted graphs
- Level-order traversal
- Finding connected components
- Checking bipartiteness

### Depth-First Search (DFS)

Explores as far as possible before backtracking.

```pseudocode
// Recursive DFS
function dfs(graph, start) {
    visited = new Set()
    parent = new Map()
    discovery = new Map()   // Discovery time
    finish = new Map()      // Finish time
    time = 0

    function dfsVisit(vertex) {
        time++
        discovery.set(vertex, time)
        visited.add(vertex)

        for neighbor in graph.getNeighbors(vertex) {
            if neighbor not in visited {
                parent.set(neighbor, vertex)
                dfsVisit(neighbor)
            }
        }

        time++
        finish.set(vertex, time)
    }

    dfsVisit(start)

    return { visited, parent, discovery, finish }
}

// Iterative DFS
function dfsIterative(graph, start) {
    visited = new Set()
    stack = new Stack()

    stack.push(start)

    while not stack.isEmpty() {
        vertex = stack.pop()

        if vertex not in visited {
            visited.add(vertex)
            process(vertex)

            for neighbor in graph.getNeighbors(vertex) {
                if neighbor not in visited {
                    stack.push(neighbor)
                }
            }
        }
    }

    return visited
}

// DFS for entire graph (handles disconnected components)
function dfsComplete(graph) {
    visited = new Set()
    components = []

    for vertex in graph.getVertices() {
        if vertex not in visited {
            component = []
            dfsComponent(graph, vertex, visited, component)
            components.append(component)
        }
    }

    return components
}
```

**Time Complexity**: O(V + E)
**Space Complexity**: O(V)

**Use Cases**:
- Topological sorting
- Cycle detection
- Finding strongly connected components
- Maze solving
- Path finding

## Shortest Path Algorithms

### Dijkstra's Algorithm

Finds shortest paths from a single source in graphs with non-negative weights.

```pseudocode
function dijkstra(graph, source) {
    distance = new Map()
    parent = new Map()
    visited = new Set()
    pq = new MinPriorityQueue()  // (distance, vertex)

    // Initialize distances
    for vertex in graph.getVertices() {
        distance.set(vertex, INFINITY)
    }
    distance.set(source, 0)

    pq.enqueue(0, source)

    while not pq.isEmpty() {
        (dist, current) = pq.dequeue()

        if current in visited {
            continue
        }
        visited.add(current)

        for (neighbor, weight) in graph.getNeighborsWithWeights(current) {
            if neighbor not in visited {
                newDist = distance.get(current) + weight

                if newDist < distance.get(neighbor) {
                    distance.set(neighbor, newDist)
                    parent.set(neighbor, current)
                    pq.enqueue(newDist, neighbor)
                }
            }
        }
    }

    return { distance, parent }
}
```

**Time Complexity**: O((V + E) log V) with binary heap
**Space Complexity**: O(V)

**Limitations**: Does not work with negative edge weights.

### Bellman-Ford Algorithm

Handles negative edge weights and detects negative cycles.

```pseudocode
function bellmanFord(graph, source) {
    distance = new Map()
    parent = new Map()

    // Initialize
    for vertex in graph.getVertices() {
        distance.set(vertex, INFINITY)
    }
    distance.set(source, 0)

    vertices = graph.getVertices()
    edges = graph.getAllEdges()

    // Relax all edges V-1 times
    for i from 1 to vertices.length - 1 {
        for (u, v, weight) in edges {
            if distance.get(u) + weight < distance.get(v) {
                distance.set(v, distance.get(u) + weight)
                parent.set(v, u)
            }
        }
    }

    // Check for negative cycles
    for (u, v, weight) in edges {
        if distance.get(u) + weight < distance.get(v) {
            throw NegativeCycleError("Graph contains negative cycle")
        }
    }

    return { distance, parent }
}
```

**Time Complexity**: O(V × E)
**Space Complexity**: O(V)

**Use Cases**: Graphs with negative weights, detecting arbitrage opportunities.

### Floyd-Warshall Algorithm

Finds shortest paths between all pairs of vertices.

```pseudocode
function floydWarshall(graph) {
    V = graph.vertexCount
    dist = copy(graph.matrix)  // Initialize with edge weights
    next = new Array(V, V)     // For path reconstruction

    // Initialize next pointers
    for i from 0 to V - 1 {
        for j from 0 to V - 1 {
            if dist[i][j] != INFINITY and i != j {
                next[i][j] = j
            }
        }
    }

    // Main algorithm
    for k from 0 to V - 1 {
        for i from 0 to V - 1 {
            for j from 0 to V - 1 {
                if dist[i][k] + dist[k][j] < dist[i][j] {
                    dist[i][j] = dist[i][k] + dist[k][j]
                    next[i][j] = next[i][k]
                }
            }
        }
    }

    // Check for negative cycles
    for i from 0 to V - 1 {
        if dist[i][i] < 0 {
            throw NegativeCycleError("Graph contains negative cycle")
        }
    }

    return { dist, next }
}

function reconstructPath(next, start, end) {
    if next[start][end] == null {
        return null
    }

    path = [start]
    while start != end {
        start = next[start][end]
        path.append(start)
    }
    return path
}
```

**Time Complexity**: O(V³)
**Space Complexity**: O(V²)

**Use Cases**: All-pairs shortest paths, transitive closure, detecting negative cycles.

## Minimum Spanning Tree Algorithms

### Prim's Algorithm

Grows MST from a starting vertex by adding minimum weight edges.

```pseudocode
function prim(graph) {
    V = graph.vertexCount
    inMST = new Array(V).fill(false)
    key = new Array(V).fill(INFINITY)    // Minimum weight to connect
    parent = new Array(V).fill(-1)
    pq = new MinPriorityQueue()

    // Start from vertex 0
    key[0] = 0
    pq.enqueue(0, 0)  // (weight, vertex)

    while not pq.isEmpty() {
        (weight, u) = pq.dequeue()

        if inMST[u] {
            continue
        }
        inMST[u] = true

        for (v, edgeWeight) in graph.getNeighborsWithWeights(u) {
            if not inMST[v] and edgeWeight < key[v] {
                key[v] = edgeWeight
                parent[v] = u
                pq.enqueue(edgeWeight, v)
            }
        }
    }

    // Build MST edges
    mstEdges = []
    totalWeight = 0
    for v from 1 to V - 1 {
        if parent[v] != -1 {
            mstEdges.append((parent[v], v, key[v]))
            totalWeight += key[v]
        }
    }

    return { edges: mstEdges, totalWeight: totalWeight }
}
```

**Time Complexity**: O((V + E) log V) with binary heap
**Space Complexity**: O(V)

### Kruskal's Algorithm

Builds MST by adding edges in order of increasing weight, using Union-Find.

```pseudocode
function kruskal(graph) {
    edges = graph.getAllEdgesWithWeights()
    edges.sortByWeight()  // Sort by weight ascending

    uf = new UnionFind(graph.vertexCount)
    mstEdges = []
    totalWeight = 0

    for (u, v, weight) in edges {
        if uf.find(u) != uf.find(v) {
            uf.union(u, v)
            mstEdges.append((u, v, weight))
            totalWeight += weight

            if mstEdges.length == graph.vertexCount - 1 {
                break  // MST complete
            }
        }
    }

    return { edges: mstEdges, totalWeight: totalWeight }
}

class UnionFind {
    parent[]
    rank[]

    function constructor(n) {
        this.parent = new Array(n)
        this.rank = new Array(n).fill(0)
        for i from 0 to n - 1 {
            this.parent[i] = i
        }
    }

    function find(x) {
        if this.parent[x] != x {
            this.parent[x] = find(this.parent[x])  // Path compression
        }
        return this.parent[x]
    }

    function union(x, y) {
        rootX = find(x)
        rootY = find(y)

        if rootX == rootY {
            return false
        }

        // Union by rank
        if this.rank[rootX] < this.rank[rootY] {
            this.parent[rootX] = rootY
        } else if this.rank[rootX] > this.rank[rootY] {
            this.parent[rootY] = rootX
        } else {
            this.parent[rootY] = rootX
            this.rank[rootX]++
        }

        return true
    }
}
```

**Time Complexity**: O(E log E) for sorting
**Space Complexity**: O(V) for Union-Find

## Topological Sort

Orders vertices so that for every edge (u, v), u comes before v. Only works on DAGs.

### DFS-Based Topological Sort

```pseudocode
function topologicalSort(graph) {
    visited = new Set()
    stack = []

    function dfs(vertex) {
        visited.add(vertex)

        for neighbor in graph.getNeighbors(vertex) {
            if neighbor not in visited {
                dfs(neighbor)
            }
        }

        stack.push(vertex)  // Add after all descendants processed
    }

    for vertex in graph.getVertices() {
        if vertex not in visited {
            dfs(vertex)
        }
    }

    return stack.reverse()
}
```

### Kahn's Algorithm (BFS-Based)

```pseudocode
function topologicalSortKahn(graph) {
    inDegree = new Map()
    result = []

    // Calculate in-degrees
    for vertex in graph.getVertices() {
        inDegree.set(vertex, 0)
    }
    for vertex in graph.getVertices() {
        for neighbor in graph.getNeighbors(vertex) {
            inDegree.set(neighbor, inDegree.get(neighbor) + 1)
        }
    }

    // Queue vertices with in-degree 0
    queue = new Queue()
    for vertex in graph.getVertices() {
        if inDegree.get(vertex) == 0 {
            queue.enqueue(vertex)
        }
    }

    while not queue.isEmpty() {
        vertex = queue.dequeue()
        result.append(vertex)

        for neighbor in graph.getNeighbors(vertex) {
            inDegree.set(neighbor, inDegree.get(neighbor) - 1)
            if inDegree.get(neighbor) == 0 {
                queue.enqueue(neighbor)
            }
        }
    }

    if result.length != graph.vertexCount {
        throw CycleError("Graph has a cycle")
    }

    return result
}
```

**Time Complexity**: O(V + E)
**Space Complexity**: O(V)

**Use Cases**: Build systems, course prerequisites, task scheduling.

## Cycle Detection

### Undirected Graph

```pseudocode
function hasCycleUndirected(graph) {
    visited = new Set()

    function dfs(vertex, parent) {
        visited.add(vertex)

        for neighbor in graph.getNeighbors(vertex) {
            if neighbor not in visited {
                if dfs(neighbor, vertex) {
                    return true
                }
            } else if neighbor != parent {
                return true  // Found cycle
            }
        }

        return false
    }

    for vertex in graph.getVertices() {
        if vertex not in visited {
            if dfs(vertex, null) {
                return true
            }
        }
    }

    return false
}
```

### Directed Graph

```pseudocode
function hasCycleDirected(graph) {
    WHITE = 0  // Unvisited
    GRAY = 1   // In current path
    BLACK = 2  // Completed

    color = new Map()
    for vertex in graph.getVertices() {
        color.set(vertex, WHITE)
    }

    function dfs(vertex) {
        color.set(vertex, GRAY)

        for neighbor in graph.getNeighbors(vertex) {
            if color.get(neighbor) == GRAY {
                return true  // Back edge found
            }
            if color.get(neighbor) == WHITE {
                if dfs(neighbor) {
                    return true
                }
            }
        }

        color.set(vertex, BLACK)
        return false
    }

    for vertex in graph.getVertices() {
        if color.get(vertex) == WHITE {
            if dfs(vertex) {
                return true
            }
        }
    }

    return false
}
```

## Algorithm Comparison

| Algorithm       | Time         | Space   | Negative Weights | Use Case                  |
|-----------------|--------------|---------|------------------|---------------------------|
| BFS             | O(V + E)     | O(V)    | N/A              | Unweighted shortest path  |
| DFS             | O(V + E)     | O(V)    | N/A              | Traversal, cycle detection|
| Dijkstra        | O((V+E)logV) | O(V)    | No               | Single-source shortest    |
| Bellman-Ford    | O(VE)        | O(V)    | Yes              | Negative weights          |
| Floyd-Warshall  | O(V³)        | O(V²)   | Yes              | All-pairs shortest        |
| Prim            | O((V+E)logV) | O(V)    | N/A              | MST (dense graphs)        |
| Kruskal         | O(E log E)   | O(V)    | N/A              | MST (sparse graphs)       |
| Topological     | O(V + E)     | O(V)    | N/A              | DAG ordering              |

## Common Pitfalls

- **Wrong algorithm for negative weights**: Dijkstra fails with negative edges
- **Forgetting to handle disconnected graphs**: Process all components
- **Cycle in topological sort**: Only works on DAGs
- **Priority queue updates**: Dijkstra may add duplicate entries
- **Off-by-one in Floyd-Warshall**: Loop bounds matter
- **Undirected edge handling**: Add both directions or handle properly

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
