# Strongly Connected Components

## Overview

A Strongly Connected Component (SCC) of a directed graph is a maximal set of vertices such that there is a path from every vertex to every other vertex within the set. Finding SCCs is fundamental for understanding the structure of directed graphs and has applications in compiler optimization, social network analysis, and solving 2-SAT problems.

The condensation of a graph (replacing each SCC with a single vertex) is always a DAG (Directed Acyclic Graph), which enables topological analysis.

## Properties

- **Maximal**: Cannot add more vertices while maintaining strong connectivity
- **Partition**: Every vertex belongs to exactly one SCC
- **Condensation DAG**: Graph of SCCs has no cycles
- **Reachability**: Within an SCC, all vertices can reach all others

## Time Complexity

| Algorithm   | Time    | Space   | Notes                        |
|-------------|---------|---------|------------------------------|
| Kosaraju    | O(V + E)| O(V)    | Two DFS passes               |
| Tarjan      | O(V + E)| O(V)    | Single DFS, uses lowlink     |
| Path-based  | O(V + E)| O(V)    | Similar to Tarjan            |

## Kosaraju's Algorithm - O(V + E)

Two-pass algorithm: DFS to get finish order, then DFS on transpose graph.

```pseudocode
function kosarajuSCC(graph) {
    n = graph.vertexCount
    visited = array of false, size n
    finishOrder = []  // Stack of vertices by finish time

    // First DFS: compute finish order
    for v from 0 to n - 1 {
        if not visited[v] {
            dfsFirst(graph, v, visited, finishOrder)
        }
    }

    // Build transpose graph
    transpose = buildTranspose(graph)

    // Second DFS: find SCCs in reverse finish order
    visited = array of false, size n
    sccs = []

    while not finishOrder.isEmpty() {
        v = finishOrder.pop()
        if not visited[v] {
            component = []
            dfsSecond(transpose, v, visited, component)
            sccs.append(component)
        }
    }

    return sccs
}

function dfsFirst(graph, v, visited, finishOrder) {
    visited[v] = true

    for u in graph.neighbors(v) {
        if not visited[u] {
            dfsFirst(graph, u, visited, finishOrder)
        }
    }

    finishOrder.push(v)  // Add after all descendants processed
}

function dfsSecond(graph, v, visited, component) {
    visited[v] = true
    component.append(v)

    for u in graph.neighbors(v) {
        if not visited[u] {
            dfsSecond(graph, u, visited, component)
        }
    }
}

function buildTranspose(graph) {
    transpose = new Graph(graph.vertexCount)

    for u from 0 to graph.vertexCount - 1 {
        for v in graph.neighbors(u) {
            transpose.addEdge(v, u)  // Reverse direction
        }
    }

    return transpose
}

// Key insight: If there's a path u → v in original graph,
// there's a path v → u in transpose.
// Processing by reverse finish order ensures we start from "sink" SCCs.
```

## Tarjan's Algorithm - O(V + E)

Single-pass algorithm using DFS discovery times and lowlink values.

```pseudocode
function tarjanSCC(graph) {
    n = graph.vertexCount
    index = 0              // Discovery time counter
    indices = array of -1, size n    // Discovery time of each vertex
    lowlink = array of -1, size n    // Lowest reachable index
    onStack = array of false, size n
    stack = []
    sccs = []

    for v from 0 to n - 1 {
        if indices[v] == -1 {
            strongConnect(graph, v, index, indices, lowlink, onStack, stack, sccs)
        }
    }

    return sccs
}

function strongConnect(graph, v, index, indices, lowlink, onStack, stack, sccs) {
    // Set discovery index and lowlink
    indices[v] = index
    lowlink[v] = index
    index++

    stack.push(v)
    onStack[v] = true

    // Consider successors
    for w in graph.neighbors(v) {
        if indices[w] == -1 {
            // w has not been visited; recurse
            strongConnect(graph, w, index, indices, lowlink, onStack, stack, sccs)
            lowlink[v] = min(lowlink[v], lowlink[w])
        } else if onStack[w] {
            // w is in stack, hence in current SCC
            lowlink[v] = min(lowlink[v], indices[w])
        }
        // If w is visited but not on stack, it's in a finished SCC
    }

    // If v is a root node, pop stack and generate SCC
    if lowlink[v] == indices[v] {
        component = []
        while true {
            w = stack.pop()
            onStack[w] = false
            component.append(w)
            if w == v {
                break
            }
        }
        sccs.append(component)
    }
}

// Key insight: lowlink[v] = smallest index reachable from v
// v is SCC root if lowlink[v] == indices[v]
```

### Lowlink Explanation

```
lowlink[v] = minimum of:
  1. indices[v]                         (own discovery time)
  2. lowlink[w] for tree edges v → w    (reachable through subtree)
  3. indices[w] for back edges v → w    (direct back edge in stack)

SCC root: vertex where lowlink[v] == indices[v]
          (cannot reach any vertex discovered earlier)
```

## Path-Based Algorithm - O(V + E)

Uses two stacks: one for vertices, one for SCC boundaries.

```pseudocode
function pathBasedSCC(graph) {
    n = graph.vertexCount
    preorder = array of -1, size n
    count = 0
    S = []      // Stack of vertices in current path
    P = []      // Stack of SCC root candidates
    sccs = []
    assigned = array of false, size n

    for v from 0 to n - 1 {
        if preorder[v] == -1 {
            dfs(graph, v, preorder, count, S, P, sccs, assigned)
        }
    }

    return sccs
}

function dfs(graph, v, preorder, count, S, P, sccs, assigned) {
    preorder[v] = count
    count++

    S.push(v)
    P.push(v)

    for w in graph.neighbors(v) {
        if preorder[w] == -1 {
            dfs(graph, w, preorder, count, S, P, sccs, assigned)
        } else if not assigned[w] {
            // Back edge to vertex in current path
            while preorder[P.top()] > preorder[w] {
                P.pop()
            }
        }
    }

    // If v is root of SCC
    if P.top() == v {
        P.pop()
        component = []
        while true {
            w = S.pop()
            assigned[w] = true
            component.append(w)
            if w == v {
                break
            }
        }
        sccs.append(component)
    }
}
```

## Building the Condensation DAG

```pseudocode
function buildCondensation(graph, sccs) {
    n = graph.vertexCount
    sccId = array of size n  // Which SCC each vertex belongs to

    // Assign SCC IDs
    for (id, component) in enumerate(sccs) {
        for v in component {
            sccId[v] = id
        }
    }

    // Build condensation graph
    condensation = new Graph(length(sccs))
    edges = new Set()

    for u from 0 to n - 1 {
        for v in graph.neighbors(u) {
            if sccId[u] != sccId[v] {
                edge = (sccId[u], sccId[v])
                if edge not in edges {
                    edges.add(edge)
                    condensation.addEdge(sccId[u], sccId[v])
                }
            }
        }
    }

    return condensation
}

// Condensation is always a DAG
// Can topologically sort to process SCCs in dependency order
```

## Use Cases

### 2-SAT Problem

```pseudocode
function solve2SAT(clauses, numVariables) {
    // Build implication graph
    // Clause (a OR b) becomes: (¬a → b) and (¬b → a)
    graph = buildImplicationGraph(clauses, numVariables)

    // Find SCCs
    sccs = tarjanSCC(graph)

    // Check satisfiability: no variable and its negation in same SCC
    sccId = computeSCCIds(sccs)

    for i from 1 to numVariables {
        if sccId[i] == sccId[-i] {  // Variable and negation
            return null  // Unsatisfiable
        }
    }

    // Assign values based on SCC order
    // If SCC(x) comes after SCC(¬x) in reverse topological order, x = true
    assignment = array of size numVariables + 1

    for i from 1 to numVariables {
        if sccId[i] > sccId[-i] {
            assignment[i] = true
        } else {
            assignment[i] = false
        }
    }

    return assignment
}
```

### Dead Code Elimination

```pseudocode
function findLiveCode(callGraph, entryPoints) {
    // Find all SCCs
    sccs = tarjanSCC(callGraph)

    // Build condensation
    condensation = buildCondensation(callGraph, sccs)

    // Find SCCs reachable from entry points
    entrySccs = new Set()
    for entry in entryPoints {
        entrySccs.add(sccId[entry])
    }

    reachable = bfsReachable(condensation, entrySccs)

    // Live code = all vertices in reachable SCCs
    liveCode = new Set()
    for sccIndex in reachable {
        for v in sccs[sccIndex] {
            liveCode.add(v)
        }
    }

    return liveCode
}
```

### Social Network Analysis

```pseudocode
function findCommunities(socialGraph) {
    // In directed graphs (follow relationships), SCCs represent
    // groups where everyone can reach everyone through follows

    sccs = tarjanSCC(socialGraph)

    communities = []
    for component in sccs {
        if length(component) > 1 {  // Non-trivial community
            communities.append(component)
        }
    }

    // Sort by size
    sort(communities by size, descending)

    return communities
}
```

### Dependency Resolution

```pseudocode
function resolveInOrder(dependencyGraph) {
    // Find SCCs (circular dependencies)
    sccs = tarjanSCC(dependencyGraph)

    // Build condensation DAG
    condensation = buildCondensation(dependencyGraph, sccs)

    // Topological sort of condensation
    order = topologicalSort(condensation)

    // Process SCCs in topological order
    result = []
    for sccIndex in order {
        // All items in an SCC must be processed together
        // (they have circular dependencies)
        result.append(sccs[sccIndex])
    }

    return result
}
```

## Comparison of Algorithms

| Aspect        | Kosaraju         | Tarjan          | Path-Based     |
|---------------|------------------|-----------------|----------------|
| DFS passes    | 2                | 1               | 1              |
| Extra storage | Transpose graph  | Stack + arrays  | Two stacks     |
| Simplicity    | Easier to understand | Moderate     | Moderate       |
| Constants     | Higher (2 DFS)   | Lower           | Lower          |
| Output order  | Reverse topo     | Reverse topo    | Reverse topo   |

## Advantages

- **Linear time**: O(V + E) optimal complexity
- **Structural insight**: Reveals graph structure
- **Enables other algorithms**: 2-SAT, dead code elimination
- **DAG transformation**: Condensation simplifies analysis

## Disadvantages

- **Directed graphs only**: Undefined for undirected graphs
- **Memory overhead**: Need stack and various arrays
- **Not incremental**: Must recompute from scratch on changes
- **Implementation subtlety**: Tarjan's lowlink can be tricky

## Common Pitfalls

- **Confusing lowlink update**: Use indices[w] for back edges, not lowlink[w]
- **Forgetting onStack check**: Only update from stack vertices
- **Wrong finish order**: Kosaraju needs reverse finish order
- **Transpose errors**: All edges must be reversed exactly once
- **Off-by-one in indices**: Be consistent with 0 or 1 indexing

## Related Algorithms

- **Connected Components**: For undirected graphs (simpler)
- **Biconnected Components**: 2-vertex-connected components
- **Bridge Finding**: Edges whose removal disconnects graph
- **Articulation Points**: Vertices whose removal disconnects graph
- **Topological Sort**: Order vertices in DAG

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
