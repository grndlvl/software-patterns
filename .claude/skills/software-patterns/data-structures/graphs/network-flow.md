# Network Flow Algorithms

## Overview

Network flow algorithms solve the maximum flow problem: given a directed graph with edge capacities, find the maximum amount of "flow" that can be sent from a source node s to a sink node t. These algorithms have applications in transportation, resource allocation, bipartite matching, image segmentation, and many combinatorial optimization problems.

The fundamental insight is the Max-Flow Min-Cut Theorem: the maximum flow equals the minimum cut capacity separating source from sink.

## Problem Definition

```pseudocode
// Flow network: directed graph G = (V, E)
// Each edge (u, v) has capacity c(u, v) > 0
// Source s and sink t
// Flow f: E → R satisfying:
//   1. Capacity constraint: 0 ≤ f(u, v) ≤ c(u, v)
//   2. Flow conservation: ∑f(u,v) = ∑f(v,w) for all v ≠ s,t
// Goal: maximize total flow from s to t
```

## Ford-Fulkerson Method - O(E × max_flow)

The foundation of flow algorithms. Repeatedly finds augmenting paths.

```pseudocode
function fordFulkerson(graph, source, sink) {
    // Initialize flow to 0
    flow = new Map()  // (u, v) -> flow value
    for (u, v) in graph.edges {
        flow[(u, v)] = 0
    }

    maxFlow = 0

    // While there's an augmenting path
    while true {
        // Find path using DFS or BFS
        path = findAugmentingPath(graph, source, sink, flow)

        if path == null {
            break  // No more augmenting paths
        }

        // Find bottleneck capacity
        bottleneck = infinity
        for (u, v) in path {
            residual = getResidualCapacity(graph, flow, u, v)
            bottleneck = min(bottleneck, residual)
        }

        // Augment flow along path
        for (u, v) in path {
            if (u, v) in graph.edges {
                flow[(u, v)] += bottleneck
            } else {
                flow[(v, u)] -= bottleneck
            }
        }

        maxFlow += bottleneck
    }

    return maxFlow
}

function getResidualCapacity(graph, flow, u, v) {
    if (u, v) in graph.edges {
        // Forward edge: capacity - flow
        return graph.capacity(u, v) - flow[(u, v)]
    } else {
        // Backward edge: current flow
        return flow[(v, u)]
    }
}

function findAugmentingPath(graph, source, sink, flow) {
    // DFS to find any path with positive residual capacity
    visited = new Set()
    parent = new Map()

    function dfs(u) {
        if u == sink {
            return true
        }
        visited.add(u)

        for v in graph.neighbors(u) {
            if v not in visited and getResidualCapacity(graph, flow, u, v) > 0 {
                parent[v] = u
                if dfs(v) {
                    return true
                }
            }
        }
        return false
    }

    if not dfs(source) {
        return null
    }

    // Reconstruct path
    path = []
    v = sink
    while v != source {
        u = parent[v]
        path.prepend((u, v))
        v = u
    }
    return path
}

// Time: O(E × max_flow) - each augmentation increases flow by ≥ 1
// Space: O(V + E)
```

## Edmonds-Karp Algorithm - O(VE²)

Ford-Fulkerson with BFS to find shortest augmenting path.

```pseudocode
function edmondsKarp(graph, source, sink) {
    flow = initializeFlow(graph)
    maxFlow = 0

    while true {
        // BFS to find shortest augmenting path
        parent = new Map()
        visited = new Set([source])
        queue = new Queue([source])

        while not queue.isEmpty() and sink not in visited {
            u = queue.dequeue()

            for v in graph.allNeighbors(u) {
                if v not in visited and getResidualCapacity(graph, flow, u, v) > 0 {
                    visited.add(v)
                    parent[v] = u
                    queue.enqueue(v)
                }
            }
        }

        if sink not in visited {
            break  // No augmenting path
        }

        // Find bottleneck
        bottleneck = infinity
        v = sink
        while v != source {
            u = parent[v]
            bottleneck = min(bottleneck, getResidualCapacity(graph, flow, u, v))
            v = u
        }

        // Augment flow
        v = sink
        while v != source {
            u = parent[v]
            augmentFlow(flow, u, v, bottleneck)
            v = u
        }

        maxFlow += bottleneck
    }

    return maxFlow
}

// Time: O(VE²) - at most O(VE) augmentations, each O(E)
// Key insight: shortest path length never decreases
```

## Dinic's Algorithm - O(V²E)

Uses level graph and blocking flows for faster augmentation.

```pseudocode
function dinic(graph, source, sink) {
    maxFlow = 0

    while true {
        // Build level graph using BFS
        level = buildLevelGraph(graph, source, sink)

        if level[sink] == -1 {
            break  // Sink not reachable
        }

        // Find blocking flow using DFS
        while true {
            blocked = findBlockingFlow(graph, source, sink, infinity, level)
            if blocked == 0 {
                break
            }
            maxFlow += blocked
        }
    }

    return maxFlow
}

function buildLevelGraph(graph, source, sink) {
    level = new Map()
    for v in graph.vertices {
        level[v] = -1
    }
    level[source] = 0

    queue = new Queue([source])

    while not queue.isEmpty() {
        u = queue.dequeue()

        for v in graph.allNeighbors(u) {
            if level[v] == -1 and getResidualCapacity(graph, flow, u, v) > 0 {
                level[v] = level[u] + 1
                queue.enqueue(v)
            }
        }
    }

    return level
}

function findBlockingFlow(graph, u, sink, pushed, level) {
    if u == sink {
        return pushed
    }

    for v in graph.allNeighbors(u) {
        if level[v] == level[u] + 1 {
            residual = getResidualCapacity(graph, flow, u, v)
            if residual > 0 {
                result = findBlockingFlow(graph, v, sink, min(pushed, residual), level)
                if result > 0 {
                    augmentFlow(flow, u, v, result)
                    return result
                }
            }
        }
    }

    return 0
}

// Time: O(V²E) general, O(E√V) for unit capacity graphs
// Space: O(V + E)
```

## Push-Relabel Algorithm - O(V²E)

Works locally by pushing excess flow and relabeling vertices.

```pseudocode
function pushRelabel(graph, source, sink) {
    n = graph.vertexCount
    height = array of size n, initialized to 0
    excess = array of size n, initialized to 0
    flow = initializeFlow(graph)

    // Initialize: source height = n, saturate edges from source
    height[source] = n

    for v in graph.neighbors(source) {
        cap = graph.capacity(source, v)
        flow[(source, v)] = cap
        excess[v] = cap
        excess[source] -= cap
    }

    // Active vertices (excess > 0, not source or sink)
    active = new Queue()
    for v in graph.vertices {
        if v != source and v != sink and excess[v] > 0 {
            active.enqueue(v)
        }
    }

    while not active.isEmpty() {
        u = active.dequeue()

        // Try to push or relabel
        while excess[u] > 0 {
            pushed = false

            for v in graph.allNeighbors(u) {
                if canPush(graph, flow, height, u, v) {
                    push(graph, flow, excess, u, v)
                    pushed = true

                    if v != source and v != sink and excess[v] > 0 {
                        active.enqueue(v)
                    }

                    if excess[u] == 0 {
                        break
                    }
                }
            }

            if not pushed {
                relabel(graph, flow, height, u)
            }
        }
    }

    return excess[sink]
}

function canPush(graph, flow, height, u, v) {
    return height[u] == height[v] + 1 and getResidualCapacity(graph, flow, u, v) > 0
}

function push(graph, flow, excess, u, v) {
    delta = min(excess[u], getResidualCapacity(graph, flow, u, v))
    augmentFlow(flow, u, v, delta)
    excess[u] -= delta
    excess[v] += delta
}

function relabel(graph, flow, height, u) {
    minHeight = infinity

    for v in graph.allNeighbors(u) {
        if getResidualCapacity(graph, flow, u, v) > 0 {
            minHeight = min(minHeight, height[v])
        }
    }

    height[u] = minHeight + 1
}

// Time: O(V²E) basic, O(V³) with highest-label selection
// Space: O(V + E)
```

## Comparison

| Algorithm      | Time          | Best For                        |
|----------------|---------------|--------------------------------|
| Ford-Fulkerson | O(E × max_flow) | Small integer capacities      |
| Edmonds-Karp   | O(VE²)        | General graphs                 |
| Dinic          | O(V²E)        | Unit capacity, bipartite       |
| Push-Relabel   | O(V²E)        | Dense graphs                   |

## Applications

### Bipartite Matching

```pseudocode
function maxBipartiteMatching(leftVertices, rightVertices, edges) {
    // Create flow network
    source = newVertex()
    sink = newVertex()

    // Source to left vertices (capacity 1)
    for v in leftVertices {
        addEdge(source, v, 1)
    }

    // Left to right edges (capacity 1)
    for (u, v) in edges {
        addEdge(u, v, 1)
    }

    // Right vertices to sink (capacity 1)
    for v in rightVertices {
        addEdge(v, sink, 1)
    }

    return maxFlow(source, sink)
}
```

### Minimum Cut

```pseudocode
function findMinCut(graph, source, sink) {
    // Run max flow
    maxFlow = edmondsKarp(graph, source, sink)

    // BFS from source in residual graph
    reachable = bfsReachable(graph, source, flow)

    // Min cut = edges from reachable to unreachable
    cut = []
    for (u, v) in graph.edges {
        if u in reachable and v not in reachable {
            cut.append((u, v))
        }
    }

    return (maxFlow, cut)
}
```

### Image Segmentation

```pseudocode
function segmentImage(pixels, foregroundSeeds, backgroundSeeds) {
    // Create graph: each pixel is a vertex
    // Source connects to foreground seeds
    // Sink connects to background seeds
    // Adjacent pixels have edges based on similarity

    graph = buildSegmentationGraph(pixels, foregroundSeeds, backgroundSeeds)
    (minCut, cutEdges) = findMinCut(graph, source, sink)

    // Pixels reachable from source are foreground
    foreground = bfsReachable(graph, source)
    return foreground
}
```

## Common Pitfalls

- **Forgetting reverse edges**: Residual graph needs backward edges
- **Integer overflow**: Use long integers for large capacities
- **Floating point precision**: Prefer integer capacities
- **Infinite capacity edges**: Handle carefully to avoid overflow
- **Self-loops**: Usually not allowed in flow networks

## Related Topics

- **Min-Cost Max-Flow**: Flow with edge costs
- **Multi-Commodity Flow**: Multiple source-sink pairs
- **Circulation Problems**: Flow with demands
- **Assignment Problem**: Special case of bipartite matching

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
