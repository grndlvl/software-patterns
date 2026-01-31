# Adjacency List

## Overview

An adjacency list represents a graph as a collection of lists where each vertex stores a list of its adjacent vertices (neighbors). It's the most common graph representation for sparse graphs, offering efficient storage and fast iteration over neighbors.

## Properties

- **Vertex-centric**: Each vertex maintains its own neighbor list
- **Space efficient**: Only stores existing edges
- **Dynamic**: Easy to add vertices and edges
- **Neighbor iteration**: Fast access to adjacent vertices
- **Edge lookup**: Slower than adjacency matrix for checking edge existence

## Time Complexity

| Operation           | Complexity      |
|---------------------|-----------------|
| Add vertex          | O(1)            |
| Add edge            | O(1)            |
| Remove vertex       | O(V + E)        |
| Remove edge         | O(degree)       |
| Check edge exists   | O(degree)       |
| Get neighbors       | O(1)            |
| Iterate neighbors   | O(degree)       |
| Get all edges       | O(V + E)        |

Where V = vertices, E = edges, degree = edges from vertex

## Space Complexity

O(V + E) for directed graphs
O(V + 2E) for undirected graphs (each edge stored twice)

Much more efficient than O(V²) adjacency matrix for sparse graphs.

## Operations

### Graph Structure

```pseudocode
class AdjacencyList {
    adjacencyList = {}  // Map: vertex -> list of neighbors
    directed = false

    function constructor(directed = false) {
        this.directed = directed
    }
}

// For weighted graphs
class WeightedAdjacencyList {
    adjacencyList = {}  // Map: vertex -> list of (neighbor, weight) pairs
}
```

### Add Vertex

```pseudocode
function addVertex(vertex) {
    if vertex not in this.adjacencyList {
        this.adjacencyList[vertex] = []
        return true
    }
    return false
}
```

### Add Edge

```pseudocode
function addEdge(source, destination) {
    // Ensure vertices exist
    if source not in this.adjacencyList {
        addVertex(source)
    }
    if destination not in this.adjacencyList {
        addVertex(destination)
    }

    // Add edge
    this.adjacencyList[source].append(destination)

    // For undirected graphs, add reverse edge
    if not this.directed {
        this.adjacencyList[destination].append(source)
    }
}

// Weighted version
function addWeightedEdge(source, destination, weight) {
    if source not in this.adjacencyList {
        addVertex(source)
    }
    if destination not in this.adjacencyList {
        addVertex(destination)
    }

    this.adjacencyList[source].append({ vertex: destination, weight: weight })

    if not this.directed {
        this.adjacencyList[destination].append({ vertex: source, weight: weight })
    }
}
```

### Remove Vertex

```pseudocode
function removeVertex(vertex) {
    if vertex not in this.adjacencyList {
        return false
    }

    // Remove all edges pointing to this vertex
    for v in this.adjacencyList.keys() {
        this.adjacencyList[v] = filter(this.adjacencyList[v],
                                       neighbor => neighbor != vertex)
    }

    // Remove the vertex itself
    delete this.adjacencyList[vertex]
    return true
}
```

### Remove Edge

```pseudocode
function removeEdge(source, destination) {
    if source not in this.adjacencyList {
        return false
    }

    // Remove destination from source's list
    originalLength = this.adjacencyList[source].length
    this.adjacencyList[source] = filter(this.adjacencyList[source],
                                        neighbor => neighbor != destination)

    if this.adjacencyList[source].length == originalLength {
        return false  // Edge didn't exist
    }

    // For undirected graphs, remove reverse edge
    if not this.directed {
        this.adjacencyList[destination] = filter(this.adjacencyList[destination],
                                                 neighbor => neighbor != source)
    }

    return true
}
```

### Check Edge Exists

```pseudocode
function hasEdge(source, destination) {
    if source not in this.adjacencyList {
        return false
    }

    return destination in this.adjacencyList[source]
}
```

### Get Neighbors

```pseudocode
function getNeighbors(vertex) {
    if vertex not in this.adjacencyList {
        return []
    }
    return this.adjacencyList[vertex]
}
```

## Implementation

### Basic Graph

```pseudocode
class Graph {
    adjacencyList = new Map()
    directed
    vertexCount = 0
    edgeCount = 0

    function constructor(directed = false) {
        this.directed = directed
    }

    function addVertex(vertex) {
        if not this.adjacencyList.has(vertex) {
            this.adjacencyList.set(vertex, [])
            this.vertexCount++
            return true
        }
        return false
    }

    function addEdge(source, destination) {
        addVertex(source)
        addVertex(destination)

        if not hasEdge(source, destination) {
            this.adjacencyList.get(source).push(destination)
            if not this.directed {
                this.adjacencyList.get(destination).push(source)
            }
            this.edgeCount++
            return true
        }
        return false
    }

    function removeVertex(vertex) {
        if not this.adjacencyList.has(vertex) {
            return false
        }

        // Count edges being removed
        edgesRemoved = this.adjacencyList.get(vertex).length

        // Remove incoming edges
        for [v, neighbors] in this.adjacencyList {
            index = neighbors.indexOf(vertex)
            if index != -1 {
                neighbors.splice(index, 1)
                if this.directed {
                    edgesRemoved++
                }
            }
        }

        this.adjacencyList.delete(vertex)
        this.vertexCount--
        this.edgeCount -= this.directed ? edgesRemoved : edgesRemoved / 2
        return true
    }

    function removeEdge(source, destination) {
        if not this.adjacencyList.has(source) {
            return false
        }

        neighbors = this.adjacencyList.get(source)
        index = neighbors.indexOf(destination)

        if index == -1 {
            return false
        }

        neighbors.splice(index, 1)

        if not this.directed {
            destNeighbors = this.adjacencyList.get(destination)
            destIndex = destNeighbors.indexOf(source)
            if destIndex != -1 {
                destNeighbors.splice(destIndex, 1)
            }
        }

        this.edgeCount--
        return true
    }

    function hasEdge(source, destination) {
        if not this.adjacencyList.has(source) {
            return false
        }
        return this.adjacencyList.get(source).includes(destination)
    }

    function getNeighbors(vertex) {
        return this.adjacencyList.get(vertex) or []
    }

    function getVertices() {
        return Array.from(this.adjacencyList.keys())
    }

    function getEdges() {
        edges = []
        for [vertex, neighbors] in this.adjacencyList {
            for neighbor in neighbors {
                if this.directed or vertex < neighbor {
                    edges.push([vertex, neighbor])
                }
            }
        }
        return edges
    }

    function getDegree(vertex) {
        if not this.adjacencyList.has(vertex) {
            return 0
        }
        return this.adjacencyList.get(vertex).length
    }

    function getInDegree(vertex) {
        if not this.directed {
            return getDegree(vertex)
        }

        count = 0
        for [v, neighbors] in this.adjacencyList {
            if neighbors.includes(vertex) {
                count++
            }
        }
        return count
    }

    function getOutDegree(vertex) {
        return getDegree(vertex)
    }
}
```

### Weighted Graph

```pseudocode
class WeightedGraph {
    adjacencyList = new Map()
    directed

    function constructor(directed = false) {
        this.directed = directed
    }

    function addVertex(vertex) {
        if not this.adjacencyList.has(vertex) {
            this.adjacencyList.set(vertex, [])
            return true
        }
        return false
    }

    function addEdge(source, destination, weight = 1) {
        addVertex(source)
        addVertex(destination)

        this.adjacencyList.get(source).push({
            vertex: destination,
            weight: weight
        })

        if not this.directed {
            this.adjacencyList.get(destination).push({
                vertex: source,
                weight: weight
            })
        }
    }

    function getWeight(source, destination) {
        if not this.adjacencyList.has(source) {
            return null
        }

        for edge in this.adjacencyList.get(source) {
            if edge.vertex == destination {
                return edge.weight
            }
        }

        return null
    }

    function getNeighborsWithWeights(vertex) {
        return this.adjacencyList.get(vertex) or []
    }

    function updateWeight(source, destination, newWeight) {
        if not this.adjacencyList.has(source) {
            return false
        }

        for edge in this.adjacencyList.get(source) {
            if edge.vertex == destination {
                edge.weight = newWeight

                if not this.directed {
                    for reverseEdge in this.adjacencyList.get(destination) {
                        if reverseEdge.vertex == source {
                            reverseEdge.weight = newWeight
                            break
                        }
                    }
                }
                return true
            }
        }

        return false
    }
}
```

## Array-Based Implementation

For graphs with integer vertices (0 to V-1):

```pseudocode
class ArrayAdjacencyList {
    lists[]         // Array of arrays
    vertexCount

    function constructor(vertexCount) {
        this.vertexCount = vertexCount
        this.lists = new Array(vertexCount)
        for i from 0 to vertexCount - 1 {
            this.lists[i] = []
        }
    }

    function addEdge(source, destination) {
        this.lists[source].push(destination)
    }

    function getNeighbors(vertex) {
        return this.lists[vertex]
    }
}
```

## Use Cases

- **Social networks**: Friend connections, followers
- **Web graphs**: Hyperlinks between pages
- **Road networks**: Intersections and roads
- **Dependency graphs**: Package dependencies, task ordering
- **Recommendation systems**: User-item relationships
- **Network topology**: Computer/device connections

## Advantages

- **Space efficient**: O(V + E) for sparse graphs
- **Fast neighbor iteration**: O(degree) to visit all neighbors
- **Dynamic**: Easy to add/remove vertices and edges
- **Natural for sparse graphs**: Most real-world graphs are sparse
- **Efficient for algorithms**: BFS, DFS traverse naturally

## Disadvantages

- **Slow edge lookup**: O(degree) to check if edge exists
- **No random access**: Must traverse list to find specific neighbor
- **More complex structure**: Than adjacency matrix
- **Poor for dense graphs**: Matrix more efficient when E ≈ V²

## Comparison with Alternatives

| Aspect              | Adjacency List | Adjacency Matrix | Edge List    |
|---------------------|----------------|------------------|--------------|
| Space               | O(V + E)       | O(V²)            | O(E)         |
| Add edge            | O(1)           | O(1)             | O(1)         |
| Remove edge         | O(degree)      | O(1)             | O(E)         |
| Check edge          | O(degree)      | O(1)             | O(E)         |
| Get neighbors       | O(1)           | O(V)             | O(E)         |
| Iterate neighbors   | O(degree)      | O(V)             | O(E)         |
| Best for            | Sparse graphs  | Dense graphs     | Edge-focused |

## Common Pitfalls

- **Duplicate edges**: Not checking before adding
- **Self-loops**: May or may not be allowed
- **Undirected edge storage**: Forgetting both directions
- **Memory leaks**: Orphaned vertex references after removal
- **Index bounds**: For array-based implementation
- **Inconsistent state**: One direction removed but not other

## Related Structures

- **Adjacency Matrix**: O(1) edge lookup, O(V²) space
- **Edge List**: Simple list of (source, destination) pairs
- **Incidence Matrix**: Vertex × Edge matrix
- **Compressed Sparse Row**: Memory-efficient for static graphs

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
