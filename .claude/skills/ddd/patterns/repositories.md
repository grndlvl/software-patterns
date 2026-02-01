# DDD Repository Pattern

## Definition

A **Repository** is an abstraction that encapsulates aggregate persistence logic. It presents a collection-like interface to the domain layer, hiding all details of data storage, retrieval, and query mechanisms. Repositories act as a bridge between the domain model and data mapping layers.

## Purpose

1. **Hide Storage Complexity** - Isolate domain logic from infrastructure concerns (databases, APIs, file systems)
2. **Collection-like Interface** - Present aggregates as if they exist in an in-memory collection
3. **Decouple Domain from Persistence** - Enable storage implementation to change without affecting domain code
4. **Enable Testing** - Facilitate easy mocking with in-memory repositories for unit tests
5. **Centralize Query Logic** - Aggregate all persistence operations in one place per aggregate root

## Repository Per Aggregate Root

**Golden Rule:** Create one repository for each aggregate root, never for entities or value objects.

```
Aggregate Root (Order)
├── Entity (LineItem)
├── Entity (Shipment)
└── Value Object (Money)

Repository: OrderRepository
  - Persists entire Order aggregate including all entities and values
  - Never: LineItemRepository, ShipmentRepository
  - Never: Create repositories for entities within the aggregate
```

**Why?**
- Aggregates must be persisted and loaded as atomic units
- Repository operations maintain aggregate invariants
- Single consistency boundary = single repository

## Pseudocode: Repository Interface

```pseudocode
interface Repository<T: AggregateRoot>
  
  // Add an aggregate to the repository
  add(aggregate: T): void
  
  // Remove an aggregate from the repository
  remove(aggregate: T): void
  
  // Find aggregate by its unique identifier
  findById(id: Identifier): T | null
  
  // Get all aggregates (use sparingly - can be expensive)
  findAll(): List<T>
  
  // Find aggregates matching a specification
  findBySpecification(spec: Specification<T>): List<T>
  
  // Query-specific method (example)
  findByEmail(email: String): T | null
  
  // Persistence hook (persist changes made to loaded aggregate)
  update(aggregate: T): void
  
  // Count aggregates (for pagination, validation)
  count(): Integer
  
  // Save (add or update) - convenience method
  save(aggregate: T): void
end interface
```

## Pseudocode: Implementation Example

```pseudocode
class InMemoryOrderRepository implements Repository<Order>
  
  private storage: Map<OrderId, Order> = new Map()
  
  function add(order: Order): void
    if storage.contains(order.getId())
      throw DuplicateAggregateException("Order already exists")
    storage.put(order.getId(), order)
  end function
  
  function remove(order: Order): void
    if not storage.contains(order.getId())
      throw AggregateNotFoundException("Order not found")
    storage.remove(order.getId())
  end function
  
  function findById(id: OrderId): Order | null
    return storage.get(id)
  end function
  
  function findAll(): List<Order>
    return storage.values().toList()
  end function
  
  function findBySpecification(spec: Specification<Order>): List<Order>
    results: List<Order> = []
    for order in storage.values()
      if spec.isSatisfiedBy(order)
        results.add(order)
    end for
    return results
  end function
  
  function findByCustomerId(customerId: CustomerId): List<Order>
    return findBySpecification(
      new CustomerIdSpecification(customerId)
    )
  end function
  
  function update(order: Order): void
    if not storage.contains(order.getId())
      throw AggregateNotFoundException("Order not found")
    storage.put(order.getId(), order)
  end function
  
  function count(): Integer
    return storage.size()
  end function
  
  function save(order: Order): void
    if storage.contains(order.getId())
      update(order)
    else
      add(order)
    end if
  end function
  
end class


class DatabaseOrderRepository implements Repository<Order>
  
  private database: Database
  private mapper: OrderMapper
  
  function add(order: Order): void
    sql: String = "INSERT INTO orders (...) VALUES (...)"
    parameters: Map = mapper.toPersistence(order)
    try
      database.execute(sql, parameters)
    catch DuplicateKeyException
      throw DuplicateAggregateException()
    end try
  end function
  
  function remove(order: Order): void
    sql: String = "DELETE FROM orders WHERE id = ?"
    rowsAffected: Integer = database.execute(sql, [order.getId()])
    if rowsAffected == 0
      throw AggregateNotFoundException()
    end if
  end function
  
  function findById(id: OrderId): Order | null
    sql: String = "SELECT * FROM orders WHERE id = ?"
    row: Row = database.queryOne(sql, [id])
    if row == null
      return null
    return mapper.toDomain(row)
  end function
  
  function findAll(): List<Order>
    sql: String = "SELECT * FROM orders"
    rows: List<Row> = database.queryAll(sql)
    return rows.map(row => mapper.toDomain(row))
  end function
  
  function findByCustomerId(customerId: CustomerId): List<Order>
    sql: String = "SELECT * FROM orders WHERE customer_id = ?"
    rows: List<Row> = database.queryAll(sql, [customerId])
    return rows.map(row => mapper.toDomain(row))
  end function
  
  function count(): Integer
    sql: String = "SELECT COUNT(*) FROM orders"
    result: Integer = database.queryScalar(sql)
    return result
  end function
  
end class
```

## Repository Methods

### Core Methods

| Method | Purpose | Returns | Notes |
|--------|---------|---------|-------|
| `add(aggregate)` | Persist new aggregate | void | Throw if already exists |
| `remove(aggregate)` | Delete aggregate | void | Throw if not found |
| `findById(id)` | Retrieve by primary key | Aggregate \| null | Fundamental query method |
| `update(aggregate)` | Persist changes to loaded aggregate | void | Only needed in some designs |
| `save(aggregate)` | Add or update (upsert) | void | Convenience method |
| `findAll()` | Retrieve all aggregates | List\<Aggregate\> | Use cautiously (can be expensive) |
| `count()` | Count total aggregates | Integer | For pagination, validation |

### Query Methods

| Method | Purpose | Returns | Notes |
|--------|---------|---------|-------|
| `findBySpecification(spec)` | Retrieve by specification | List\<Aggregate\> | Generic query pattern |
| `findByAttribute(value)` | Domain-specific queries | List\<Aggregate\> \| null | Examples: `findByEmail()`, `findByStatus()` |
| `findByDateRange(from, to)` | Temporal queries | List\<Aggregate\> | For time-based filtering |

### Best Practices

- **Minimize Query Methods** - Use Specifications for flexibility rather than proliferating `findByX()` methods
- **No Update Required in Some Designs** - If using Unit of Work pattern, `update()` is implicit
- **Aggregate Loading** - Load complete aggregates including child entities and value objects
- **Lazy Loading** - Avoid; load aggregates completely to maintain consistency boundaries
- **Batch Operations** - Keep outside repository; use application services for transactions

## Repository vs DAO Comparison

| Aspect | Repository | DAO (Data Access Object) |
|--------|-----------|------------------------|
| **Scope** | Domain aggregates only | Any persistent object |
| **Interface** | Collection-like | Generic CRUD (create, read, update, delete) |
| **Abstraction Level** | High-level, domain-oriented | Low-level, data-oriented |
| **Granularity** | One per aggregate root | Often one per table/entity |
| **Child Entities** | Loaded with aggregate | May have separate DAOs |
| **Design Pattern** | DDD pattern | Generic data access pattern |
| **Query Methods** | Domain-meaningful names | Generic query methods |
| **Consistency** | Maintains aggregate invariants | No knowledge of invariants |
| **When to Use** | Domain-driven projects | Data-centric or legacy systems |

**Example:**

DAO approach:
```pseudocode
orderDao.insert(order)
orderDao.delete(orderId)
orderDao.findById(orderId)

lineItemDao.findByOrderId(orderId)
shipmentDao.findByOrderId(orderId)
```

Repository approach:
```pseudocode
orderRepository.add(order)      // Includes all child entities
orderRepository.remove(order)
orderRepository.findById(orderId)
// LineItems and Shipments loaded automatically
```

## Summary

| Concept | Details |
|---------|---------|
| **What** | Abstraction for aggregate persistence with collection-like interface |
| **Where** | One per aggregate root (not per entity or value object) |
| **Why** | Hide storage complexity, decouple domain from infrastructure, enable testing |
| **How** | Implement with add(), remove(), findById(), findBySpecification(), query methods |
| **When** | Use in domain-driven designs; avoid in simple CRUD systems |
| **Key Rule** | One aggregate root = one repository; aggregates loaded atomically |
| **Query Strategy** | Prefer Specifications over proliferating `findByX()` methods |
| **Testing** | Replace with in-memory implementation for fast, isolated tests |

---

*Based on concepts from "Domain-Driven Design" by Eric Evans.*
