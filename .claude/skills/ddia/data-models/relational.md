# Relational Data Model

## Definition and History

The **relational data model** was introduced by Edgar F. Codd in 1970 in his seminal paper "A Relational Model of Data for Large Shared Data Banks." It revolutionized database design by providing a mathematical foundation based on set theory and predicate logic.

**Core Principles:**
- Data is organized into **relations** (tables)
- Each relation consists of **tuples** (rows) and **attributes** (columns)
- Data is accessed through declarative queries (SQL)
- Physical storage details are hidden from the application (data independence)

The relational model dominated database systems for decades due to its simplicity, mathematical rigor, and powerful query capabilities.

---

## Core Concepts

### Tables (Relations)
A table represents an entity type in the domain (e.g., Users, Orders, Products).

### Rows (Tuples)
Each row represents a single instance of the entity with specific attribute values.

### Columns (Attributes)
Columns define the properties of the entity. Each column has a name and a data type.

### Schemas
A schema defines the structure of tables, including:
- Column names and types
- Constraints (NOT NULL, UNIQUE, CHECK)
- Primary keys and foreign keys
- Indexes

```pseudocode
SCHEMA Users {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  username: VARCHAR(50) UNIQUE NOT NULL
  email: VARCHAR(100) UNIQUE NOT NULL
  created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
}

SCHEMA Orders {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  user_id: INTEGER FOREIGN KEY REFERENCES Users(id)
  total_amount: DECIMAL(10, 2) NOT NULL
  order_date: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
}
```

---

## Normalization

Normalization is the process of organizing data to reduce redundancy and improve data integrity. It involves decomposing tables into smaller tables and defining relationships between them.

### First Normal Form (1NF)
- Each column contains atomic (indivisible) values
- Each row is unique
- No repeating groups

**Violation Example:**
```pseudocode
TABLE Orders_Denormalized {
  order_id: 1
  customer_name: "Alice"
  items: "Laptop, Mouse, Keyboard"  // NOT atomic!
}
```

**1NF Compliant:**
```pseudocode
TABLE Order_Items {
  order_id: 1, item: "Laptop"
  order_id: 1, item: "Mouse"
  order_id: 1, item: "Keyboard"
}
```

### Second Normal Form (2NF)
- Must be in 1NF
- All non-key attributes are fully functionally dependent on the entire primary key (no partial dependencies)

**Violation Example:**
```pseudocode
TABLE Order_Items {
  order_id, product_id → PRIMARY KEY
  product_id → product_name  // Partial dependency!
  order_id, product_id → quantity
}
```

**2NF Compliant:**
```pseudocode
TABLE Products {
  product_id → PRIMARY KEY
  product_name
}

TABLE Order_Items {
  order_id, product_id → PRIMARY KEY
  quantity
}
```

### Third Normal Form (3NF)
- Must be in 2NF
- No transitive dependencies (non-key attributes depend only on the primary key)

**Violation Example:**
```pseudocode
TABLE Employees {
  employee_id → PRIMARY KEY
  department_id
  department_id → department_name  // Transitive dependency!
}
```

**3NF Compliant:**
```pseudocode
TABLE Departments {
  department_id → PRIMARY KEY
  department_name
}

TABLE Employees {
  employee_id → PRIMARY KEY
  department_id → FOREIGN KEY
}
```

### Boyce-Codd Normal Form (BCNF)
- Must be in 3NF
- For every functional dependency X → Y, X must be a superkey

BCNF is a stricter version of 3NF that eliminates certain anomalies. Most 3NF tables are also in BCNF.

---

## Relationships

### One-to-One (1:1)
One row in Table A corresponds to exactly one row in Table B.

```pseudocode
TABLE Users {
  user_id: PRIMARY KEY
  username
}

TABLE UserProfiles {
  profile_id: PRIMARY KEY
  user_id: UNIQUE FOREIGN KEY REFERENCES Users(user_id)
  bio
  avatar_url
}
```

**Use cases:** Separating frequently accessed from rarely accessed data, security isolation.

### One-to-Many (1:N)
One row in Table A can correspond to many rows in Table B.

```pseudocode
TABLE Authors {
  author_id: PRIMARY KEY
  name
}

TABLE Books {
  book_id: PRIMARY KEY
  author_id: FOREIGN KEY REFERENCES Authors(author_id)
  title
}
```

**Implementation:** Foreign key in the "many" side pointing to the "one" side.

### Many-to-Many (M:N)
Many rows in Table A can correspond to many rows in Table B.

```pseudocode
TABLE Students {
  student_id: PRIMARY KEY
  name
}

TABLE Courses {
  course_id: PRIMARY KEY
  course_name
}

TABLE Enrollments {
  student_id: FOREIGN KEY REFERENCES Students(student_id)
  course_id: FOREIGN KEY REFERENCES Courses(course_id)
  enrollment_date
  PRIMARY KEY (student_id, course_id)
}
```

**Implementation:** Requires a junction/bridge table with foreign keys to both tables.

---

## Joins and Performance Implications

### Join Types

**INNER JOIN:** Returns only matching rows from both tables.
```pseudocode
SELECT users.username, orders.total_amount
FROM users
INNER JOIN orders ON users.id = orders.user_id
```

**LEFT JOIN:** Returns all rows from the left table, with matching rows from the right (or NULL).
```pseudocode
SELECT users.username, orders.total_amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id
```

**RIGHT JOIN:** Returns all rows from the right table, with matching rows from the left (or NULL).

**FULL OUTER JOIN:** Returns all rows from both tables, with NULLs where no match exists.

### Performance Implications

| Factor | Impact |
|--------|--------|
| **Index on join columns** | Dramatically improves join performance (log time vs linear scan) |
| **Number of joins** | Each join adds computational overhead; 3+ joins can be expensive |
| **Table size** | Joining large tables without proper indexes can cause performance issues |
| **Join order** | Query optimizer determines optimal join order, but hints may be needed |
| **Cardinality** | High cardinality (many distinct values) generally performs better |

**Optimization strategies:**
- Index foreign key columns
- Use covering indexes for frequently joined columns
- Denormalize for read-heavy workloads
- Consider materialized views for complex joins
- Partition large tables

---

## When to Use Relational Databases

### Ideal Use Cases

| Scenario | Why Relational Fits |
|----------|---------------------|
| **Transactional systems (OLTP)** | ACID guarantees, strong consistency, complex queries |
| **Structured data with relationships** | Foreign keys, joins, referential integrity |
| **Ad-hoc queries** | SQL's flexibility for unknown query patterns |
| **Multi-record transactions** | Atomicity across multiple tables |
| **Data integrity critical** | Constraints, triggers, stored procedures |
| **Moderate scale (up to TBs)** | Proven scalability with vertical and horizontal partitioning |

### Strong Suits

- **ACID transactions:** Atomicity, Consistency, Isolation, Durability
- **Declarative queries:** SQL is powerful and well-understood
- **Schema enforcement:** Prevents invalid data at write time
- **Rich tooling:** Mature ecosystem of ORMs, migration tools, monitoring
- **Join operations:** Efficient querying across multiple related tables

---

## Limitations and Trade-offs

### Scalability Challenges

| Challenge | Description |
|-----------|-------------|
| **Vertical scaling limits** | Single-server architecture has physical hardware limits |
| **Sharding complexity** | Distributing data across multiple servers is difficult with joins and transactions |
| **Replication lag** | Read replicas may be inconsistent during high write loads |
| **Schema migrations** | Changing schemas on large tables can cause downtime |

### Impedance Mismatch

Object-oriented applications represent data as objects with methods and inheritance, while relational databases use tables with rows and columns. This mismatch requires:
- ORMs (Object-Relational Mappers) to bridge the gap
- Additional code to handle joins and nested structures
- Performance overhead from translation layers

```pseudocode
// Application object
CLASS User {
  id
  username
  profile: Profile  // Nested object
  orders: List<Order>  // Collection
}

// Must be decomposed into:
TABLE users (id, username)
TABLE profiles (id, user_id, ...)
TABLE orders (id, user_id, ...)
```

### Poor Fit for Certain Data Models

- **Hierarchical/nested data:** JSON documents, XML trees (document databases are better)
- **Graph relationships:** Social networks, recommendation engines (graph databases are better)
- **Sparse data:** Tables with many NULL values waste space
- **Schema-less data:** Rapidly evolving schemas require frequent migrations

### Write Performance at Scale

- High write throughput can overwhelm single-server databases
- Multi-master replication introduces conflict resolution complexity
- Write-ahead logging (WAL) becomes a bottleneck

---

## Pseudocode Examples

### Schema Design: E-commerce System

```pseudocode
// Users and authentication
SCHEMA users {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  email: VARCHAR(255) UNIQUE NOT NULL
  password_hash: VARCHAR(255) NOT NULL
  created_at: TIMESTAMP DEFAULT NOW()
  INDEX idx_email (email)
}

// Product catalog
SCHEMA products {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  name: VARCHAR(255) NOT NULL
  description: TEXT
  price: DECIMAL(10, 2) NOT NULL
  stock_quantity: INTEGER DEFAULT 0
  category_id: INTEGER FOREIGN KEY REFERENCES categories(id)
  INDEX idx_category (category_id)
  INDEX idx_price (price)
}

SCHEMA categories {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  name: VARCHAR(100) UNIQUE NOT NULL
  parent_category_id: INTEGER FOREIGN KEY REFERENCES categories(id)
}

// Orders and order items (1:N relationship)
SCHEMA orders {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  user_id: INTEGER FOREIGN KEY REFERENCES users(id)
  total_amount: DECIMAL(10, 2) NOT NULL
  status: ENUM('pending', 'shipped', 'delivered', 'cancelled')
  created_at: TIMESTAMP DEFAULT NOW()
  INDEX idx_user (user_id)
  INDEX idx_status_created (status, created_at)
}

SCHEMA order_items {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  order_id: INTEGER FOREIGN KEY REFERENCES orders(id) ON DELETE CASCADE
  product_id: INTEGER FOREIGN KEY REFERENCES products(id)
  quantity: INTEGER NOT NULL CHECK (quantity > 0)
  price_at_purchase: DECIMAL(10, 2) NOT NULL
  INDEX idx_order (order_id)
}

// Many-to-many: products and tags
SCHEMA tags {
  id: INTEGER PRIMARY KEY AUTO_INCREMENT
  name: VARCHAR(50) UNIQUE NOT NULL
}

SCHEMA product_tags {
  product_id: INTEGER FOREIGN KEY REFERENCES products(id) ON DELETE CASCADE
  tag_id: INTEGER FOREIGN KEY REFERENCES tags(id) ON DELETE CASCADE
  PRIMARY KEY (product_id, tag_id)
}
```

### Common Query Patterns

```pseudocode
// Find all orders for a user with product details
QUERY get_user_orders(user_id) {
  SELECT
    orders.id AS order_id,
    orders.created_at,
    orders.total_amount,
    order_items.quantity,
    products.name AS product_name,
    order_items.price_at_purchase
  FROM orders
  INNER JOIN order_items ON orders.id = order_items.order_id
  INNER JOIN products ON order_items.product_id = products.id
  WHERE orders.user_id = user_id
  ORDER BY orders.created_at DESC
}

// Get product inventory by category
QUERY inventory_by_category() {
  SELECT
    categories.name AS category,
    COUNT(products.id) AS product_count,
    SUM(products.stock_quantity) AS total_stock,
    AVG(products.price) AS avg_price
  FROM categories
  LEFT JOIN products ON categories.id = products.category_id
  GROUP BY categories.id, categories.name
  ORDER BY total_stock DESC
}

// Find products by tag (many-to-many)
QUERY products_by_tag(tag_name) {
  SELECT products.*
  FROM products
  INNER JOIN product_tags ON products.id = product_tags.product_id
  INNER JOIN tags ON product_tags.tag_id = tags.id
  WHERE tags.name = tag_name
}
```

### Transaction Example

```pseudocode
TRANSACTION process_order(user_id, cart_items) {
  BEGIN TRANSACTION

  // Create order
  order_id = INSERT INTO orders (user_id, total_amount, status)
             VALUES (user_id, calculate_total(cart_items), 'pending')

  // Add order items and update inventory
  FOR EACH item IN cart_items {
    // Check stock
    SELECT stock_quantity INTO current_stock
    FROM products
    WHERE id = item.product_id
    FOR UPDATE  // Lock row for update

    IF current_stock < item.quantity {
      ROLLBACK
      THROW ERROR "Insufficient stock for product " + item.product_id
    }

    // Insert order item
    INSERT INTO order_items (order_id, product_id, quantity, price_at_purchase)
    VALUES (order_id, item.product_id, item.quantity, item.price)

    // Decrease inventory
    UPDATE products
    SET stock_quantity = stock_quantity - item.quantity
    WHERE id = item.product_id
  }

  COMMIT
  RETURN order_id
}
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| **Data Model** | Tables with rows and columns; relations defined by foreign keys |
| **Query Language** | SQL (declarative, set-based) |
| **Schema** | Rigid schema defined upfront; migrations required for changes |
| **Relationships** | 1:1, 1:N, M:N via foreign keys and junction tables |
| **Normalization** | 1NF, 2NF, 3NF, BCNF to reduce redundancy |
| **Joins** | INNER, LEFT, RIGHT, FULL OUTER; performance depends on indexes |
| **Transactions** | ACID guarantees (Atomicity, Consistency, Isolation, Durability) |
| **Strengths** | Strong consistency, complex queries, data integrity, mature tooling |
| **Weaknesses** | Impedance mismatch, horizontal scaling difficulty, rigid schema |
| **Best For** | OLTP systems, structured data with complex relationships, ad-hoc queries |
| **Poor Fit For** | Document/hierarchical data, graph data, massive scale, schema-less data |
| **Scalability** | Vertical scaling (strong); horizontal scaling (challenging) |
| **Examples** | PostgreSQL, MySQL, Oracle, SQL Server, SQLite |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
