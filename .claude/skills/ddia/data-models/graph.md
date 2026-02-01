# Graph Data Model

## Definition

A **graph data model** stores data as vertices (nodes) and edges (connections), optimized for representing and querying highly interconnected data where relationships are as important as the entities themselves.

Unlike relational databases that use foreign keys to represent relationships, graph databases make relationships first-class citizens, enabling efficient traversal of complex relationship patterns.

## Use Cases

| Use Case | Why Graph Excels |
|----------|------------------|
| **Social Networks** | "Friends of friends", mutual connections, influence paths |
| **Recommendation Engines** | "People who bought X also bought Y", collaborative filtering |
| **Knowledge Graphs** | Semantic relationships, entity linking, inference |
| **Fraud Detection** | Pattern matching across transactions, identifying suspicious networks |
| **Network/IT Operations** | Dependency mapping, root cause analysis, impact analysis |
| **Supply Chain** | Multi-hop traceability, bottleneck identification |
| **Access Control** | Role hierarchies, permission inheritance |

## Core Concepts

### Vertices (Nodes)

Vertices represent entities in the system. Each vertex has:
- **ID**: Unique identifier
- **Labels**: Type classification (e.g., "Person", "Product", "Location")
- **Properties**: Key-value pairs describing the entity

```pseudocode
Vertex {
  id: "person:123"
  labels: ["Person", "Customer"]
  properties: {
    name: "Alice",
    email: "alice@example.com",
    age: 28
  }
}
```

### Edges (Relationships)

Edges represent connections between vertices. Each edge has:
- **Source vertex**: The starting point
- **Target vertex**: The ending point
- **Label**: Type of relationship (e.g., "KNOWS", "PURCHASED", "LOCATED_IN")
- **Properties**: Metadata about the relationship (timestamps, weights, etc.)
- **Direction**: Edges can be directed or bidirectional

```pseudocode
Edge {
  source: "person:123"
  target: "person:456"
  label: "KNOWS"
  properties: {
    since: "2020-01-15",
    relationship_type: "colleague"
  }
}
```

### Properties

Both vertices and edges can store arbitrary key-value properties:

```pseudocode
# Vertex properties
Person {
  name: "Bob"
  occupation: "Engineer"
  skills: ["Python", "Rust", "Go"]
}

# Edge properties
WORKS_WITH {
  since: "2022-03-01"
  project: "Database Engine"
  hours_per_week: 40
}
```

## Graph Models: Property Graphs vs Triple Stores

### Property Graph Model

**Structure**: Vertices and edges with properties

**Characteristics**:
- Flexible schema (properties can vary between vertices of same type)
- Rich metadata on both entities and relationships
- Intuitive for object-oriented thinking
- Common in Neo4j, JanusGraph, Amazon Neptune

```pseudocode
# Property graph example
(alice:Person {name: "Alice", age: 28})
  -[:KNOWS {since: 2020}]->
(bob:Person {name: "Bob", age: 32})
  -[:WORKS_AT {role: "Engineer"}]->
(acme:Company {name: "ACME Corp", founded: 2015})
```

### Triple Store (RDF) Model

**Structure**: Subject-Predicate-Object triples

**Characteristics**:
- Uniform data representation (everything is a triple)
- Built for semantic web standards (SPARQL, OWL, RDFS)
- Strong typing and ontology support
- Common in Stardog, Virtuoso, RDF4J

```pseudocode
# Triple store example (subject, predicate, object)
<alice>  rdf:type          <Person>
<alice>  name              "Alice"
<alice>  age               28
<alice>  knows             <bob>
<bob>    rdf:type          <Person>
<bob>    name              "Bob"
<bob>    works_at          <acme>
<acme>   rdf:type          <Company>
<acme>   name              "ACME Corp"
```

**Key Difference**: Property graphs store properties directly on entities/edges. Triple stores decompose everything into atomic subject-predicate-object statements.

## Graph Query Patterns

### 1. Direct Relationships

Finding immediate connections:

```pseudocode
# Find all friends of Alice
MATCH (alice:Person {name: "Alice"})-[:KNOWS]->(friend)
RETURN friend.name
```

### 2. Multi-Hop Traversals

Following paths of varying depth:

```pseudocode
# Find friends of friends (2 hops)
MATCH (alice:Person {name: "Alice"})-[:KNOWS]->(friend)-[:KNOWS]->(fof)
WHERE fof <> alice
RETURN DISTINCT fof.name

# Variable length paths (up to 5 hops)
MATCH (alice:Person {name: "Alice"})-[:KNOWS*1..5]->(connection)
RETURN connection.name, length(path) AS degrees_of_separation
```

### 3. Path Finding

Finding shortest or optimal paths:

```pseudocode
# Shortest path between two people
MATCH path = shortestPath(
  (alice:Person {name: "Alice"})-[:KNOWS*]-(bob:Person {name: "Bob"})
)
RETURN path, length(path) AS hops

# All paths with conditions
MATCH path = (start:Location {name: "NYC"})-[:ROUTE*]->(end:Location {name: "LAX"})
WHERE ALL(edge IN relationships(path) WHERE edge.cost < 500)
RETURN path
ORDER BY length(path)
LIMIT 5
```

### 4. Pattern Matching

Finding complex structural patterns:

```pseudocode
# Find triangles (mutual friend groups)
MATCH (a:Person)-[:KNOWS]->(b:Person)-[:KNOWS]->(c:Person)-[:KNOWS]->(a)
RETURN a.name, b.name, c.name

# Find influencers (highly connected nodes)
MATCH (person:Person)-[:KNOWS]->(friend)
WITH person, COUNT(friend) AS friend_count
WHERE friend_count > 100
RETURN person.name, friend_count
ORDER BY friend_count DESC
```

### 5. Aggregation on Paths

Analyzing graph metrics:

```pseudocode
# Average distance in social network
MATCH (a:Person)-[:KNOWS*]-(b:Person)
WHERE id(a) < id(b)  # Avoid counting pairs twice
RETURN AVG(length(path)) AS average_separation

# Clustering coefficient
MATCH (person:Person)-[:KNOWS]->(friend)-[:KNOWS]->(fof)-[:KNOWS]->(person)
WITH person, COUNT(DISTINCT friend) AS friends, COUNT(*) AS triangles
RETURN person.name, (triangles * 2.0) / (friends * (friends - 1)) AS clustering_coeff
```

### 6. Recommendation Queries

Collaborative filtering patterns:

```pseudocode
# Recommend products based on similar users' purchases
MATCH (user:Person {name: "Alice"})-[:PURCHASED]->(product:Product)
       <-[:PURCHASED]-(other:Person)-[:PURCHASED]->(recommendation:Product)
WHERE NOT (user)-[:PURCHASED]->(recommendation)
WITH recommendation, COUNT(DISTINCT other) AS common_buyers
RETURN recommendation.name, common_buyers
ORDER BY common_buyers DESC
LIMIT 10
```

## When Graphs Excel

### Strengths of Graph Databases

| Scenario | Graph Advantage |
|----------|-----------------|
| **Many-to-many relationships** | No expensive JOINs required; edges are materialized |
| **Variable-depth traversals** | "Friends of friends of friends..." queries are natural |
| **Path-dependent queries** | Shortest path, all paths, path conditions |
| **Evolving schema** | Adding new relationship types doesn't require migrations |
| **Highly connected data** | Performance stays consistent with graph complexity |
| **Pattern matching** | Finding structural patterns (triangles, communities) |

### Performance Characteristics

```pseudocode
# Graph traversal complexity
O(1) - Find vertex by ID
O(deg(v)) - Find neighbors of vertex v (where deg(v) = number of edges)
O(V + E) - Traverse entire graph (V vertices, E edges)

# Relational equivalent grows exponentially with depth
Depth 1: 1 JOIN
Depth 2: 2 JOINs
Depth 3: 3 JOINs (performance degrades rapidly)
Depth N: N JOINs (often impractical beyond depth 3-4)
```

## Comparison with Relational Model

### Representing Relationships: Graph vs Relational

**Relational approach** (modeling friends):

```pseudocode
# Tables
Person (id, name, email, age)
Friendship (person1_id, person2_id, since)

# Query: Friends of friends
SELECT p3.name
FROM Person p1
JOIN Friendship f1 ON p1.id = f1.person1_id
JOIN Person p2 ON f1.person2_id = p2.id
JOIN Friendship f2 ON p2.id = f2.person1_id
JOIN Person p3 ON f2.person2_id = p3.id
WHERE p1.name = 'Alice' AND p3.id != p1.id
```

**Graph approach**:

```pseudocode
# Query: Friends of friends
MATCH (alice:Person {name: "Alice"})-[:KNOWS*2]->(fof)
WHERE fof <> alice
RETURN fof.name
```

### Trade-offs

| Aspect | Relational Database | Graph Database |
|--------|---------------------|----------------|
| **Schema rigidity** | Strict schema enforced | Flexible, schema-optional |
| **Relationship queries** | Multiple JOINs (slow for deep traversals) | Direct traversal (fast at any depth) |
| **Simple queries** | Excellent (indexed lookups) | Good |
| **Complex joins** | Possible but expensive | Natural and efficient |
| **Aggregations** | Excellent (GROUP BY, SUM, etc.) | Possible but less optimized |
| **Transactions** | ACID, mature | ACID, varies by implementation |
| **Tooling ecosystem** | Extensive (BI, reporting, ORMs) | Growing but smaller |
| **Storage efficiency** | Compact for tabular data | More overhead per relationship |

### When to Choose Each

**Use Relational when**:
- Data is naturally tabular
- Queries primarily aggregate or filter records
- Relationship depth is fixed and shallow
- Strong schema validation is critical
- Extensive reporting/BI tooling needed

**Use Graph when**:
- Data is highly interconnected
- Queries traverse variable-depth paths
- Relationship types evolve frequently
- Pattern matching is important
- Real-time traversal performance is critical

## Practical Examples

### Example 1: Social Network Recommendations

```pseudocode
# Find friend suggestions (friends of friends who aren't already friends)
MATCH (user:Person {id: $userId})-[:KNOWS]->(friend)-[:KNOWS]->(suggestion)
WHERE NOT (user)-[:KNOWS]->(suggestion)
  AND suggestion <> user
WITH suggestion, COUNT(DISTINCT friend) AS mutual_friends
WHERE mutual_friends >= 3
RETURN suggestion.name, suggestion.id, mutual_friends
ORDER BY mutual_friends DESC
LIMIT 10
```

### Example 2: Fraud Detection

```pseudocode
# Find suspicious transaction patterns (circular money flow)
MATCH path = (account1:Account)-[:TRANSFER*3..5]->(account1)
WHERE ALL(txn IN relationships(path)
    WHERE txn.timestamp > $last_24_hours
    AND txn.amount > 1000)
WITH path, REDUCE(total = 0, txn IN relationships(path) | total + txn.amount) AS total_flow
WHERE total_flow > 50000
RETURN path, total_flow
ORDER BY total_flow DESC
```

### Example 3: Supply Chain Traceability

```pseudocode
# Trace product origin through supply chain
MATCH path = (product:Product {batch_id: $batchId})
             -[:MANUFACTURED_FROM*]->(raw_material:RawMaterial)
WHERE raw_material.supplier_country = "Country X"
RETURN path,
       [node IN nodes(path) | node.name] AS supply_chain,
       length(path) AS chain_depth
```

### Example 4: Knowledge Graph Inference

```pseudocode
# Find experts on a topic through multi-hop reasoning
MATCH (topic:Topic {name: "Machine Learning"})
      <-[:SUBTOPIC_OF*0..3]-(related_topic:Topic)
      <-[:PUBLISHED_ON]-(paper:Paper)
      <-[:AUTHORED]-(expert:Person)
WITH expert,
     COUNT(DISTINCT paper) AS papers_count,
     COLLECT(DISTINCT related_topic.name) AS topics
WHERE papers_count >= 5
RETURN expert.name, papers_count, topics
ORDER BY papers_count DESC
LIMIT 20
```

## Summary Table

| Aspect | Description |
|--------|-------------|
| **Primary Abstraction** | Vertices (nodes) and edges (relationships) |
| **Key Strength** | Efficient traversal of highly connected data |
| **Query Model** | Pattern matching and path traversal |
| **Schema** | Flexible, schema-optional (property graphs) or schema-enforced (RDF) |
| **Best For** | Social networks, recommendations, fraud detection, knowledge graphs |
| **Weakness** | Less efficient for tabular aggregations and simple filtering |
| **Popular Implementations** | Neo4j, JanusGraph, Amazon Neptune, Stardog (RDF) |
| **Query Languages** | Cypher (property graphs), SPARQL (RDF/triple stores) |
| **Scaling Approach** | Often sharded by graph partitioning (minimize edge cuts) |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
