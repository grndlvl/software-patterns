# Document Data Model

## Definition

The **Document Data Model** is a data modeling approach that stores data as self-contained documents (typically JSON, XML, or BSON). Unlike the relational model, which spreads data across multiple tables with foreign key relationships, the document model keeps related data together in nested structures. This model emerged prominently in the late 2000s with the rise of NoSQL databases like MongoDB, CouchDB, and RethinkDB.

## Core Concepts

### Documents

A **document** is a self-describing, hierarchical tree structure composed of key-value pairs. Documents are typically encoded as JSON (or similar formats like BSON, XML). Each document can have a different structure, even within the same collection.

```pseudocode
// Example document representing a user profile
{
  "user_id": "user123",
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z",
  "address": {
    "street": "123 Main St",
    "city": "Springfield",
    "country": "USA",
    "postal_code": "12345"
  },
  "phone_numbers": [
    "+1-555-0100",
    "+1-555-0101"
  ],
  "preferences": {
    "theme": "dark",
    "notifications_enabled": true
  }
}
```

### Collections

Documents are organized into **collections** (analogous to tables in relational databases). Collections group similar documents together, though there's no enforced schema requiring all documents to have the same structure.

```pseudocode
// Users collection
users = [
  { "user_id": "user123", "name": "Alice", ... },
  { "user_id": "user456", "name": "Bob", ... }
]

// Posts collection
posts = [
  { "post_id": "post001", "author_id": "user123", "title": "...", ... },
  { "post_id": "post002", "author_id": "user456", "title": "...", ... }
]
```

### Embedded Data

One of the document model's key features is **data locality**: related data can be embedded directly within a document rather than spread across multiple tables.

```pseudocode
// Embedded approach: author details within each post
{
  "post_id": "post001",
  "title": "Understanding Document Databases",
  "content": "...",
  "author": {
    "user_id": "user123",
    "name": "Alice Johnson",
    "email": "alice@example.com"
  },
  "comments": [
    {
      "comment_id": "c1",
      "user": { "user_id": "user456", "name": "Bob" },
      "text": "Great post!",
      "timestamp": "2024-01-16T14:00:00Z"
    }
  ],
  "tags": ["databases", "nosql", "architecture"]
}
```

## Schema Flexibility

### Schemaless vs Schema-on-Read

Document databases are often described as "schemaless," but more accurately they use **schema-on-read** rather than **schema-on-write**:

- **Schema-on-write** (relational): The database enforces a schema when data is written. All rows must conform to the table's structure.
- **Schema-on-read** (document): The schema is implicit and only enforced when the application reads data. Documents can have varying structures.

```pseudocode
// Same collection can contain documents with different structures

// Older document format
{
  "user_id": "user001",
  "full_name": "Charlie Davis"
}

// Newer document format (name split into components)
{
  "user_id": "user002",
  "name": {
    "first": "Diana",
    "last": "Evans"
  }
}

// Application handles both formats when reading
function getUserName(user_document):
    if user_document has "full_name":
        return user_document.full_name
    else if user_document has "name":
        return user_document.name.first + " " + user_document.name.last
    else:
        return "Unknown"
```

### Benefits of Schema Flexibility

1. **Easier schema evolution**: Add new fields without migrations
2. **Heterogeneous data**: Store varied document structures in the same collection
3. **Rapid development**: No need to define schema upfront
4. **Natural object mapping**: Documents often map directly to application objects

## Data Locality Benefits

When an application frequently needs to access an entire document (or a large portion of it), the document model's **data locality** provides significant performance advantages:

```pseudocode
// Single query retrieves entire user profile with embedded data
user_profile = database.users.findOne({ "user_id": "user123" })

// Immediately accessible without joins:
// - user_profile.name
// - user_profile.address.city
// - user_profile.phone_numbers[0]
// - user_profile.preferences.theme

// Contrast with relational model requiring multiple queries/joins:
// 1. SELECT * FROM users WHERE user_id = 'user123'
// 2. SELECT * FROM addresses WHERE user_id = 'user123'
// 3. SELECT * FROM phone_numbers WHERE user_id = 'user123'
// 4. SELECT * FROM preferences WHERE user_id = 'user123'
```

**Locality advantage**: The entire document is stored as a contiguous sequence on disk, requiring only one seek operation rather than multiple seeks across different tables.

## When Documents Excel vs Struggle

### Documents Excel When:

1. **Data has a tree-like structure** (one-to-many relationships)
2. **Entire document is frequently accessed together**
3. **Schema varies between records** or evolves frequently
4. **Application needs are document-centric** (e.g., content management, user profiles)

```pseudocode
// Resume/CV example (perfect for document model)
{
  "person_id": "person789",
  "name": "Elena Foster",
  "positions": [
    {
      "title": "Senior Engineer",
      "organization": "Tech Corp",
      "start_date": "2020-01-01",
      "end_date": "2023-12-31"
    },
    {
      "title": "Lead Architect",
      "organization": "Startup Inc",
      "start_date": "2024-01-01",
      "end_date": null
    }
  ],
  "education": [
    {
      "degree": "BS Computer Science",
      "institution": "State University",
      "year": 2015
    }
  ],
  "skills": ["Python", "JavaScript", "System Design"]
}
```

### Documents Struggle When:

1. **Many-to-many relationships** are common
2. **Data is highly interconnected** (graph-like)
3. **Joins across collections** are frequently needed
4. **Updates affect deeply nested fields**
5. **Document size grows unbounded** (large arrays)

```pseudocode
// Social network example (poor fit for document model)
// Problem: Many-to-many relationships between users

// User A follows users B, C, D
{
  "user_id": "A",
  "following": ["B", "C", "D"]
}

// User B is followed by users A, E, F
// How to efficiently query "Who follows user B?"
// Requires scanning all user documents or maintaining redundant data
```

## Document References vs Embedding

The document model offers two approaches for representing relationships:

### 1. Embedding (Denormalization)

Store related data directly within the document.

**Pros:**
- Better read performance (data locality)
- Single query retrieves all data
- Atomic updates to embedded data

**Cons:**
- Data duplication if same data embedded in multiple documents
- Document size can grow large
- Updates require modifying multiple documents

```pseudocode
// Embedding approach: Blog post with embedded comments
{
  "post_id": "post001",
  "title": "Document Databases Explained",
  "author_id": "user123",
  "comments": [
    {
      "comment_id": "c1",
      "author_id": "user456",
      "author_name": "Bob",
      "text": "Excellent article!",
      "timestamp": "2024-01-20T10:00:00Z"
    },
    {
      "comment_id": "c2",
      "author_id": "user789",
      "author_name": "Carol",
      "text": "Very informative.",
      "timestamp": "2024-01-20T11:00:00Z"
    }
  ]
}

// Query: Get post with all comments
post = database.posts.findOne({ "post_id": "post001" })
// All comments immediately available: post.comments
```

### 2. References (Normalization)

Store references (IDs) to related documents in other collections.

**Pros:**
- No data duplication
- Easier to maintain consistency
- Smaller document sizes

**Cons:**
- Requires multiple queries (no joins in many document databases)
- Slower read performance for related data
- Application must resolve references

```pseudocode
// Reference approach: Posts reference comments in separate collection

// Posts collection
{
  "post_id": "post001",
  "title": "Document Databases Explained",
  "author_id": "user123",
  "comment_ids": ["c1", "c2"]
}

// Comments collection
{
  "comment_id": "c1",
  "post_id": "post001",
  "author_id": "user456",
  "text": "Excellent article!",
  "timestamp": "2024-01-20T10:00:00Z"
}

{
  "comment_id": "c2",
  "post_id": "post001",
  "author_id": "user789",
  "text": "Very informative.",
  "timestamp": "2024-01-20T11:00:00Z"
}

// Query: Get post and resolve comments (requires two queries)
post = database.posts.findOne({ "post_id": "post001" })
comments = database.comments.find({ "comment_id": in post.comment_ids })
```

### Decision Criteria

| Factor | Prefer Embedding | Prefer References |
|--------|------------------|-------------------|
| Relationship cardinality | One-to-few | One-to-many, Many-to-many |
| Data update frequency | Rarely updated | Frequently updated |
| Data size | Small, bounded | Large or unbounded |
| Query patterns | Access together | Access independently |
| Data consistency needs | Eventual consistency OK | Strong consistency needed |

## Comparison with Relational Model

| Aspect | Relational Model | Document Model |
|--------|------------------|----------------|
| **Data structure** | Tables with rows and columns | Collections with nested documents |
| **Schema** | Schema-on-write (enforced) | Schema-on-read (flexible) |
| **Relationships** | Foreign keys with joins | Embedded data or references |
| **Data locality** | Spread across tables | Localized in documents |
| **Query language** | SQL (standardized) | Database-specific (e.g., MongoDB query language) |
| **Strengths** | Complex queries, many-to-many, ACID | Tree-like data, flexibility, scalability |
| **Weaknesses** | Schema rigidity, join overhead | Limited join support, data duplication |
| **Best for** | Transactional systems, complex analytics | Content management, catalogs, real-time web |

### Conversion Example

**Relational approach:**

```pseudocode
// Users table
users: [
  { user_id: 1, first_name: "Alice", last_name: "Johnson" }
]

// Addresses table
addresses: [
  { address_id: 101, user_id: 1, street: "123 Main St", city: "Springfield" }
]

// Phone_numbers table
phone_numbers: [
  { phone_id: 201, user_id: 1, number: "+1-555-0100" },
  { phone_id: 202, user_id: 1, number: "+1-555-0101" }
]

// Query requires joins:
// SELECT u.*, a.*, p.number
// FROM users u
// LEFT JOIN addresses a ON u.user_id = a.user_id
// LEFT JOIN phone_numbers p ON u.user_id = p.user_id
// WHERE u.user_id = 1
```

**Document approach:**

```pseudocode
// Users collection
{
  "user_id": 1,
  "name": {
    "first": "Alice",
    "last": "Johnson"
  },
  "address": {
    "street": "123 Main St",
    "city": "Springfield"
  },
  "phone_numbers": [
    "+1-555-0100",
    "+1-555-0101"
  ]
}

// Single query, no joins:
// database.users.findOne({ "user_id": 1 })
```

## Pseudocode Examples

### Basic CRUD Operations

```pseudocode
// CREATE: Insert a new document
new_user = {
  "user_id": "user999",
  "name": "Grace Hopper",
  "email": "grace@example.com",
  "joined_date": "2024-02-01T00:00:00Z"
}
database.users.insertOne(new_user)

// READ: Query documents
user = database.users.findOne({ "user_id": "user999" })
all_users = database.users.find({ "joined_date": { $gte: "2024-01-01" } })

// UPDATE: Modify a document
database.users.updateOne(
  { "user_id": "user999" },
  { $set: { "email": "grace.hopper@example.com" } }
)

// DELETE: Remove a document
database.users.deleteOne({ "user_id": "user999" })
```

### Querying Nested Fields

```pseudocode
// Find users in a specific city
users_in_springfield = database.users.find({
  "address.city": "Springfield"
})

// Find users with specific preference
dark_theme_users = database.users.find({
  "preferences.theme": "dark"
})
```

### Working with Arrays

```pseudocode
// Find posts with a specific tag
database.posts.find({
  "tags": "nosql"
})

// Add an element to an array
database.posts.updateOne(
  { "post_id": "post001" },
  { $push: { "tags": "data-modeling" } }
)

// Remove an element from an array
database.posts.updateOne(
  { "post_id": "post001" },
  { $pull: { "tags": "nosql" } }
)
```

### Handling Schema Evolution

```pseudocode
// Old document format
{
  "product_id": "prod001",
  "name": "Widget",
  "price": 19.99
}

// New document format (price now includes currency)
{
  "product_id": "prod002",
  "name": "Gadget",
  "price": {
    "amount": 29.99,
    "currency": "USD"
  }
}

// Application code handles both formats
function getProductPrice(product):
    if typeof(product.price) == "number":
        // Old format
        return { "amount": product.price, "currency": "USD" }
    else:
        // New format
        return product.price
```

## Summary Table

| Feature | Description | Benefit | Trade-off |
|---------|-------------|---------|-----------|
| **Self-contained documents** | Related data stored together | Simple queries, data locality | Potential duplication |
| **Schema flexibility** | Schema-on-read, not schema-on-write | Easy evolution, rapid development | No enforcement at database level |
| **Nested structures** | Arrays and objects within documents | Natural object mapping | Limited join capabilities |
| **Data locality** | Contiguous storage of document | Fast read performance | Updates may rewrite entire document |
| **Collection-based** | Documents grouped in collections | Flexible organization | Less structure than relational tables |
| **Denormalization-friendly** | Embedding encouraged for performance | Reduced query complexity | Data consistency challenges |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
