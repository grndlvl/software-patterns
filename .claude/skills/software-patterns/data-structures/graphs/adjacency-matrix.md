# Adjacency Matrix

## Overview

An adjacency matrix represents a graph as a 2D array where entry [i][j] indicates whether an edge exists from vertex i to vertex j (and its weight for weighted graphs). It provides O(1) edge lookup at the cost of O(V²) space, making it ideal for dense graphs or algorithms requiring frequent edge queries.

## Properties

- **Fixed size**: V × V matrix for V vertices
- **Symmetric**: For undirected graphs, matrix[i][j] = matrix[j][i]
- **Direct access**: O(1) edge existence check
- **Space intensive**: O(V²) regardless of edge count
- **Vertex indices**: Vertices typically numbered 0 to V-1

## Time Complexity

| Operation           | Complexity |
|---------------------|------------|
| Add edge            | O(1)       |
| Remove edge         | O(1)       |
| Check edge exists   | O(1)       |
| Get neighbors       | O(V)       |
| Iterate neighbors   | O(V)       |
| Add vertex          | O(V²)      |
| Remove vertex       | O(V²)      |
| Get edge weight     | O(1)       |
| Get all edges       | O(V²)      |

## Space Complexity

O(V²) always, regardless of the number of edges. This makes it inefficient for sparse graphs but acceptable for dense graphs where E ≈ V².

## Operations

### Matrix Structure

```pseudocode
// Unweighted graph: 0 = no edge, 1 = edge exists
// Weighted graph: 0 or INF = no edge, value = weight

class AdjacencyMatrix {
    matrix[][]
    vertexCount
    directed

    function constructor(vertexCount, directed = false) {
        this.vertexCount = vertexCount
        this.directed = directed
        this.matrix = new Array(vertexCount)

        for i from 0 to vertexCount - 1 {
            this.matrix[i] = new Array(vertexCount).fill(0)
        }
    }
}
```

### Add Edge

```pseudocode
function addEdge(source, destination, weight = 1) {
    if source < 0 or source >= this.vertexCount {
        throw InvalidVertexError
    }
    if destination < 0 or destination >= this.vertexCount {
        throw InvalidVertexError
    }

    this.matrix[source][destination] = weight

    if not this.directed {
        this.matrix[destination][source] = weight
    }
}
```

### Remove Edge

```pseudocode
function removeEdge(source, destination) {
    if source < 0 or source >= this.vertexCount {
        return false
    }
    if destination < 0 or destination >= this.vertexCount {
        return false
    }

    existed = this.matrix[source][destination] != 0
    this.matrix[source][destination] = 0

    if not this.directed {
        this.matrix[destination][source] = 0
    }

    return existed
}
```

### Check Edge Exists

```pseudocode
function hasEdge(source, destination) {
    if source < 0 or source >= this.vertexCount {
        return false
    }
    if destination < 0 or destination >= this.vertexCount {
        return false
    }

    return this.matrix[source][destination] != 0
}
```

### Get Neighbors

```pseudocode
function getNeighbors(vertex) {
    if vertex < 0 or vertex >= this.vertexCount {
        return []
    }

    neighbors = []
    for i from 0 to this.vertexCount - 1 {
        if this.matrix[vertex][i] != 0 {
            neighbors.append(i)
        }
    }
    return neighbors
}
```

### Get Edge Weight

```pseudocode
function getWeight(source, destination) {
    if not hasEdge(source, destination) {
        return null  // or INFINITY for weighted shortest path
    }
    return this.matrix[source][destination]
}
```

## Implementation

### Basic Unweighted Graph

```pseudocode
class Graph {
    matrix[][]
    vertexCount
    directed
    edgeCount = 0

    function constructor(vertexCount, directed = false) {
        this.vertexCount = vertexCount
        this.directed = directed
        this.matrix = createMatrix(vertexCount, 0)
    }

    function createMatrix(size, defaultValue) {
        matrix = new Array(size)
        for i from 0 to size - 1 {
            matrix[i] = new Array(size).fill(defaultValue)
        }
        return matrix
    }

    function addEdge(source, destination) {
        validateVertex(source)
        validateVertex(destination)

        if this.matrix[source][destination] == 0 {
            this.matrix[source][destination] = 1
            this.edgeCount++

            if not this.directed {
                this.matrix[destination][source] = 1
            }
        }
    }

    function removeEdge(source, destination) {
        validateVertex(source)
        validateVertex(destination)

        if this.matrix[source][destination] != 0 {
            this.matrix[source][destination] = 0
            this.edgeCount--

            if not this.directed {
                this.matrix[destination][source] = 0
            }
            return true
        }
        return false
    }

    function hasEdge(source, destination) {
        validateVertex(source)
        validateVertex(destination)
        return this.matrix[source][destination] != 0
    }

    function getNeighbors(vertex) {
        validateVertex(vertex)

        neighbors = []
        for i from 0 to this.vertexCount - 1 {
            if this.matrix[vertex][i] != 0 {
                neighbors.append(i)
            }
        }
        return neighbors
    }

    function getDegree(vertex) {
        validateVertex(vertex)

        degree = 0
        for i from 0 to this.vertexCount - 1 {
            if this.matrix[vertex][i] != 0 {
                degree++
            }
        }
        return degree
    }

    function getInDegree(vertex) {
        validateVertex(vertex)

        if not this.directed {
            return getDegree(vertex)
        }

        inDegree = 0
        for i from 0 to this.vertexCount - 1 {
            if this.matrix[i][vertex] != 0 {
                inDegree++
            }
        }
        return inDegree
    }

    function getOutDegree(vertex) {
        return getDegree(vertex)
    }

    function getAllEdges() {
        edges = []
        for i from 0 to this.vertexCount - 1 {
            startJ = this.directed ? 0 : i
            for j from startJ to this.vertexCount - 1 {
                if this.matrix[i][j] != 0 {
                    edges.append([i, j])
                }
            }
        }
        return edges
    }

    function validateVertex(vertex) {
        if vertex < 0 or vertex >= this.vertexCount {
            throw InvalidVertexError("Vertex " + vertex + " out of bounds")
        }
    }

    function print() {
        for i from 0 to this.vertexCount - 1 {
            row = ""
            for j from 0 to this.vertexCount - 1 {
                row = row + this.matrix[i][j] + " "
            }
            output(row)
        }
    }
}
```

### Weighted Graph

```pseudocode
class WeightedGraph {
    matrix[][]
    vertexCount
    directed
    INF = Infinity

    function constructor(vertexCount, directed = false) {
        this.vertexCount = vertexCount
        this.directed = directed
        this.matrix = createMatrix(vertexCount, this.INF)

        // Diagonal is 0 (distance to self)
        for i from 0 to vertexCount - 1 {
            this.matrix[i][i] = 0
        }
    }

    function addEdge(source, destination, weight) {
        validateVertex(source)
        validateVertex(destination)

        this.matrix[source][destination] = weight

        if not this.directed {
            this.matrix[destination][source] = weight
        }
    }

    function getWeight(source, destination) {
        validateVertex(source)
        validateVertex(destination)
        return this.matrix[source][destination]
    }

    function hasEdge(source, destination) {
        weight = getWeight(source, destination)
        return weight != this.INF and source != destination
    }

    function getNeighborsWithWeights(vertex) {
        validateVertex(vertex)

        neighbors = []
        for i from 0 to this.vertexCount - 1 {
            if i != vertex and this.matrix[vertex][i] != this.INF {
                neighbors.append({ vertex: i, weight: this.matrix[vertex][i] })
            }
        }
        return neighbors
    }
}
```

### Dynamic Vertex Addition

```pseudocode
class DynamicAdjacencyMatrix {
    matrix[][]
    vertexCount
    capacity

    function constructor(initialCapacity = 16) {
        this.capacity = initialCapacity
        this.vertexCount = 0
        this.matrix = createMatrix(initialCapacity, 0)
    }

    function addVertex() {
        if this.vertexCount >= this.capacity {
            resize(this.capacity * 2)
        }

        newVertex = this.vertexCount
        this.vertexCount++
        return newVertex
    }

    function resize(newCapacity) {
        newMatrix = createMatrix(newCapacity, 0)

        // Copy existing data
        for i from 0 to this.vertexCount - 1 {
            for j from 0 to this.vertexCount - 1 {
                newMatrix[i][j] = this.matrix[i][j]
            }
        }

        this.matrix = newMatrix
        this.capacity = newCapacity
    }

    function removeVertex(vertex) {
        // Shift rows and columns
        for i from vertex to this.vertexCount - 2 {
            for j from 0 to this.vertexCount - 1 {
                this.matrix[i][j] = this.matrix[i + 1][j]
            }
        }

        for j from vertex to this.vertexCount - 2 {
            for i from 0 to this.vertexCount - 1 {
                this.matrix[i][j] = this.matrix[i][j + 1]
            }
        }

        this.vertexCount--
    }
}
```

## Matrix Properties for Graph Analysis

```pseudocode
// Number of paths of length k from i to j = matrix^k[i][j]
function countPaths(matrix, source, destination, length) {
    result = matrixPower(matrix, length)
    return result[source][destination]
}

// Transitive closure using matrix multiplication
function transitiveClosure(matrix) {
    n = matrix.length
    closure = copy(matrix)

    for k from 0 to n - 1 {
        for i from 0 to n - 1 {
            for j from 0 to n - 1 {
                closure[i][j] = closure[i][j] or (closure[i][k] and closure[k][j])
            }
        }
    }

    return closure
}
```

## Use Cases

- **Dense graphs**: When E ≈ V²
- **Edge-heavy algorithms**: Frequent edge existence checks
- **Floyd-Warshall**: All-pairs shortest paths
- **Graph properties**: Easy to check connectivity patterns
- **Small graphs**: When V is small, V² is acceptable
- **Matrix operations**: When graph algorithms use linear algebra

## Advantages

- **O(1) edge lookup**: Instant check if edge exists
- **O(1) edge modification**: Add or remove in constant time
- **Simple implementation**: Just a 2D array
- **Good for dense graphs**: Minimal overhead when mostly full
- **Matrix operations**: Powers, transitive closure, etc.

## Disadvantages

- **O(V²) space**: Inefficient for sparse graphs
- **O(V²) initialization**: Must initialize all cells
- **O(V) neighbor iteration**: Must scan entire row
- **O(V²) to add/remove vertex**: Must resize matrix
- **Wastes space**: Many zeros in sparse graphs

## Comparison with Alternatives

| Aspect              | Adjacency Matrix | Adjacency List | Edge List    |
|---------------------|------------------|----------------|--------------|
| Space               | O(V²)            | O(V + E)       | O(E)         |
| Add edge            | O(1)             | O(1)           | O(1)         |
| Remove edge         | O(1)             | O(degree)      | O(E)         |
| Check edge          | O(1)             | O(degree)      | O(E)         |
| Get neighbors       | O(V)             | O(1)           | O(E)         |
| Iterate neighbors   | O(V)             | O(degree)      | O(E)         |
| Best for            | Dense graphs     | Sparse graphs  | Edge-focused |

### When to Use Adjacency Matrix

| Condition                      | Use Matrix? |
|--------------------------------|-------------|
| E > V² / 2 (dense)             | Yes         |
| Frequent edge existence checks | Yes         |
| Floyd-Warshall algorithm       | Yes         |
| Small V (< 1000)               | Often yes   |
| E << V² (sparse)               | No          |
| Large V, sparse                | No          |
| Dynamic vertex add/remove      | No          |

## Common Pitfalls

- **Index out of bounds**: Always validate vertex indices
- **Symmetric updates**: Forgetting undirected edge symmetry
- **Self-loops**: Diagonal interpretation (0 or weight?)
- **Memory usage**: Underestimating V² growth
- **Sparse graphs**: Using matrix for sparse data
- **Initialization**: Not setting INF for weighted shortest paths

## Related Structures

- **Adjacency List**: O(V + E) space, O(degree) edge check
- **Edge List**: Simple list of edges
- **Incidence Matrix**: V × E matrix for edge-vertex relations
- **Laplacian Matrix**: Degree matrix - Adjacency matrix
- **Sparse Matrix**: Compressed storage for sparse graphs

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
