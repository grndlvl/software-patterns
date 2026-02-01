# Transaction Isolation Levels

## Definition and Purpose

**Transaction isolation** determines how and when the changes made by one transaction become visible to other concurrent transactions. Isolation is the "I" in ACID properties and addresses the fundamental challenge of concurrent database access.

### Why Isolation Matters

Without proper isolation, concurrent transactions can interfere with each other in subtle and dangerous ways:

- **Race Conditions**: Multiple transactions reading and writing the same data simultaneously
- **Data Inconsistency**: Transactions seeing partial results from other incomplete transactions
- **Lost Updates**: One transaction's changes overwriting another's
- **Integrity Violations**: Business rules being violated due to concurrent access

The isolation level you choose represents a **tradeoff between consistency and performance**. Stronger isolation provides more guarantees but reduces concurrency and throughput.

---

## Isolation Levels (Weakest to Strongest)

### 1. Read Uncommitted

**Definition**: Transactions can see uncommitted changes from other transactions.

**Characteristics**:
- Weakest isolation level
- Allows **dirty reads** (reading uncommitted data)
- No protection against any anomalies
- Highest concurrency, minimal locking overhead
- Rarely used in practice due to serious consistency issues

**When to Use**:
- Analytical queries where approximate results are acceptable
- Non-critical read-only operations
- Performance-critical scenarios where consistency doesn't matter

```pseudocode
// Transaction 1
BEGIN TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1
-- Not yet committed

// Transaction 2 (Read Uncommitted)
BEGIN TRANSACTION
SELECT balance FROM accounts WHERE id = 1
-- Sees the uncommitted -100 change (dirty read)
COMMIT

// Transaction 1
ROLLBACK  -- Transaction 2 saw data that never existed!
```

**Risks**: If Transaction 1 rolls back, Transaction 2 has read data that was never committed, potentially leading to incorrect business logic.

---

### 2. Read Committed

**Definition**: Transactions only see data that has been committed. This is the **default isolation level** in many databases (PostgreSQL, SQL Server, Oracle).

**Guarantees**:
- **No dirty reads**: Only see committed data
- **No dirty writes**: Only overwrite committed data

**How It Works**:
- Uses row-level locks for writes
- For reads, database keeps both old committed value and new uncommitted value
- Readers see old value until writer commits

```pseudocode
// Transaction 1
BEGIN TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1
-- Holds write lock on row

// Transaction 2 (Read Committed)
BEGIN TRANSACTION
SELECT balance FROM accounts WHERE id = 1
-- Sees OLD committed value (before -100)
-- Does NOT block, reads snapshot

// Transaction 1
COMMIT  -- Releases lock

// Transaction 2 (if it reads again)
SELECT balance FROM accounts WHERE id = 1
-- Now sees NEW committed value (after -100)
```

**Still Allows**:
- **Non-repeatable reads**: Reading the same row twice may return different values
- **Phantom reads**: Range queries may return different rows

**Common Use Cases**:
- General-purpose OLTP applications
- Web applications with mostly independent requests
- Systems where slight inconsistency between reads is acceptable

---

### 3. Repeatable Read / Snapshot Isolation

**Definition**: Each transaction reads from a consistent snapshot of the database as it existed at the start of the transaction.

**Guarantees**:
- No dirty reads
- No dirty writes
- **No non-repeatable reads**: Reading the same row twice returns the same value
- Each transaction sees a **frozen snapshot** of committed data

**Implementation** (Multi-Version Concurrency Control - MVCC):
- Database maintains multiple versions of each object
- Each transaction gets a unique transaction ID
- Readers never block writers, writers never block readers
- Reads use snapshot from transaction start time

```pseudocode
// Initial state: balance = 500

// Transaction 1 (starts at time T1)
BEGIN TRANSACTION
SELECT balance FROM accounts WHERE id = 1
-- Reads snapshot from T1: balance = 500

// Transaction 2 (starts at time T2 > T1)
BEGIN TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1
COMMIT
-- Creates new version with balance = 400

// Transaction 1 (still reading from T1 snapshot)
SELECT balance FROM accounts WHERE id = 1
-- Still sees balance = 500 (repeatable read!)
-- Does NOT see Transaction 2's committed change

COMMIT
```

**Visibility Rules**:

```pseudocode
transaction_can_see(object_version, reader_txid):
    // Created by later transaction? Not visible
    if object_version.created_by > reader_txid:
        return false

    // Deleted by earlier committed transaction? Not visible
    if object_version.deleted_by is not null
       and object_version.deleted_by < reader_txid
       and is_committed(object_version.deleted_by):
        return false

    // Otherwise visible
    return true
```

**Still Allows**:
- **Phantom reads** in some implementations (not PostgreSQL)
- **Write skew** and lost updates (see Anomalies section)

**Database Implementations**:
- PostgreSQL: "Repeatable Read" = True snapshot isolation
- MySQL InnoDB: "Repeatable Read" = Snapshot isolation + next-key locks (prevents phantoms)
- Oracle: "Serializable" = Actually snapshot isolation (not true serializability)

---

### 4. Serializable

**Definition**: The strongest isolation level. Guarantees that transactions execute as if they ran **serially** (one after another), even though they may run concurrently.

**Guarantees**:
- No dirty reads
- No dirty writes
- No non-repeatable reads
- **No phantom reads**
- **No write skew**
- **No lost updates**
- Complete isolation from all anomalies

**Implementation Techniques**:

#### A. Two-Phase Locking (2PL)

Traditional approach using locks:

```pseudocode
// Two-Phase Locking Protocol

// Phase 1: Growing (acquire locks)
// Phase 2: Shrinking (release locks after commit/abort)

read_with_2pl(object):
    acquire_shared_lock(object)  // Blocks if exclusive lock held
    value = read(object)
    // Lock held until transaction ends
    return value

write_with_2pl(object, value):
    acquire_exclusive_lock(object)  // Blocks if any lock held
    write(object, value)
    // Lock held until transaction ends

commit():
    release_all_locks()
```

**Types of 2PL**:
- **Shared locks**: Multiple readers can hold simultaneously
- **Exclusive locks**: Only one writer, blocks all readers
- **Predicate locks**: Lock ranges of data (e.g., "all accounts with balance > 1000")

**Downsides**:
- Significant performance overhead
- Reduced concurrency (readers block writers, writers block readers)
- Potential for deadlocks

#### B. Serializable Snapshot Isolation (SSI)

Modern optimistic approach (PostgreSQL, FoundationDB):

```pseudocode
// SSI Algorithm

transaction_execute():
    snapshot_version = get_current_version()

    // Execute reads/writes against snapshot
    execute_sql_statements()

    // Before commit: detect conflicts
    for each read in transaction.reads:
        if read.data_modified_since_snapshot():
            ABORT  // Another transaction modified our read set

    for each write in transaction.writes:
        if write.conflicts_with_concurrent_write():
            ABORT  // Write-write conflict detected

    // No conflicts detected
    COMMIT
```

**How SSI Works**:
1. Transactions execute against snapshots (like Repeatable Read)
2. Database tracks reads and writes
3. Before commit, detects if execution is equivalent to some serial ordering
4. If not, aborts one of the conflicting transactions

**Advantages over 2PL**:
- Better performance (optimistic, less locking)
- No deadlocks (transactions abort instead)
- Readers don't block writers

**Downsides**:
- Higher abort rate under high contention
- Application must retry aborted transactions

---

## Read Anomalies

### Dirty Reads

**Definition**: Reading uncommitted data from another transaction.

```pseudocode
// Transaction 1
BEGIN
UPDATE products SET stock = stock - 1 WHERE id = 42
-- Stock is now 0 (uncommitted)

// Transaction 2
BEGIN
SELECT stock FROM products WHERE id = 42
-- Reads stock = 0 (dirty read!)
if stock == 0:
    alert("Out of stock!")

// Transaction 1
ROLLBACK  -- Stock returns to 1
-- But Transaction 2 already sent alert!
```

**Prevented by**: Read Committed and stronger.

---

### Non-Repeatable Reads

**Definition**: Reading the same row twice within a transaction returns different values.

```pseudocode
// Initial: balance = 1000

// Transaction 1
BEGIN
SELECT balance FROM accounts WHERE id = 1  -- Reads 1000

// Transaction 2
BEGIN
UPDATE accounts SET balance = 500 WHERE id = 1
COMMIT

// Transaction 1
SELECT balance FROM accounts WHERE id = 1  -- Reads 500!
-- Same query, different result (non-repeatable)
```

**Prevented by**: Repeatable Read and stronger.

---

### Phantom Reads

**Definition**: Range query returns different rows when executed twice due to inserts/deletes by other transactions.

```pseudocode
// Transaction 1
BEGIN
SELECT COUNT(*) FROM orders WHERE customer_id = 42
-- Returns 10 orders

// Transaction 2
BEGIN
INSERT INTO orders (customer_id, ...) VALUES (42, ...)
COMMIT

// Transaction 1
SELECT COUNT(*) FROM orders WHERE customer_id = 42
-- Returns 11 orders (phantom row appeared!)
```

**Prevented by**: Serializable (and MySQL Repeatable Read with next-key locks).

---

## Write Anomalies

### Lost Updates

**Definition**: Two transactions read, modify, and write back the same value, causing one update to be lost.

```pseudocode
// Initial: counter = 100

// Transaction 1
BEGIN
value = SELECT counter FROM stats WHERE id = 1  -- Reads 100
value = value + 1  -- Computes 101

// Transaction 2
BEGIN
value = SELECT counter FROM stats WHERE id = 1  -- Reads 100
value = value + 1  -- Computes 101

// Transaction 1
UPDATE stats SET counter = 101 WHERE id = 1
COMMIT

// Transaction 2
UPDATE stats SET counter = 101 WHERE id = 1  -- Overwrites!
COMMIT

// Final: counter = 101 (should be 102)
```

**Solutions**:
- **Atomic operations**: `UPDATE stats SET counter = counter + 1`
- **Explicit locking**: `SELECT ... FOR UPDATE`
- **Compare-and-set**: `UPDATE ... WHERE counter = old_value`
- **Serializable isolation**

---

### Write Skew

**Definition**: Two transactions read overlapping data and make decisions based on it, but their combined writes violate a constraint.

**Classic Example**: Doctor On-Call Constraint

```pseudocode
// Constraint: At least 1 doctor must be on-call at all times
// Initial: Alice and Bob are both on-call

// Transaction 1 (Alice)
BEGIN
on_call_count = SELECT COUNT(*) FROM doctors WHERE on_call = true
-- Reads 2 doctors on call

if on_call_count >= 2:
    UPDATE doctors SET on_call = false WHERE name = 'Alice'
COMMIT

// Transaction 2 (Bob, concurrent with T1)
BEGIN
on_call_count = SELECT COUNT(*) FROM doctors WHERE on_call = true
-- Reads 2 doctors on call (same snapshot)

if on_call_count >= 2:
    UPDATE doctors SET on_call = false WHERE name = 'Bob'
COMMIT

// Result: 0 doctors on call (constraint violated!)
```

**Why This Happens**:
- Both transactions read the same snapshot (2 doctors)
- Both make valid decisions based on that snapshot
- But combined writes create invalid state
- Not prevented by Repeatable Read/Snapshot Isolation!

**Solutions**:
- **Explicit locking**: `SELECT ... FOR UPDATE`
- **Materialized conflicts**: Create explicit lock table
- **Serializable isolation**: SSI or 2PL detects the conflict

---

## Implementation Techniques

### Multi-Version Concurrency Control (MVCC)

**Core Idea**: Keep multiple versions of each database object to allow readers and writers to not block each other.

```pseudocode
// Object Version Structure
class ObjectVersion:
    data: value
    created_by: transaction_id  // Which txn created this version
    deleted_by: transaction_id  // Which txn deleted this version (nullable)

// Transaction Structure
class Transaction:
    txid: unique_id
    snapshot_version: version_id  // Which snapshot to read from

    read_object(object_id):
        versions = get_all_versions(object_id)
        visible_version = find_visible_version(versions, this.txid)
        return visible_version.data

    write_object(object_id, new_data):
        create_new_version(object_id, new_data, this.txid)

    delete_object(object_id):
        current_version = read_object(object_id)
        current_version.deleted_by = this.txid

// Garbage Collection
cleanup_old_versions():
    for each object_version in database:
        if all_active_transactions_newer_than(object_version):
            delete(object_version)  // No transaction can see it anymore
```

**Advantages**:
- Readers never block writers
- Writers never block readers
- High concurrency for read-heavy workloads

**Downsides**:
- Storage overhead (multiple versions)
- Garbage collection complexity
- Index maintenance overhead

---

### Two-Phase Locking (2PL)

**Core Idea**: Transactions acquire locks before accessing data and hold them until commit.

```pseudocode
// Lock Types
enum LockType:
    SHARED      // Read lock (compatible with other shared locks)
    EXCLUSIVE   // Write lock (incompatible with all locks)

// Lock Manager
class LockManager:
    locks: Map[object_id -> List[Lock]]

    acquire_lock(transaction, object_id, lock_type):
        while true:
            current_locks = locks[object_id]

            if is_compatible(current_locks, lock_type):
                grant_lock(transaction, object_id, lock_type)
                return
            else:
                wait_for_lock_release(object_id)

    is_compatible(current_locks, new_lock_type):
        if new_lock_type == SHARED:
            // Shared locks compatible with other shared locks
            return all(lock.type == SHARED for lock in current_locks)
        else:
            // Exclusive locks incompatible with any locks
            return len(current_locks) == 0

// Predicate Locks (for range queries)
acquire_predicate_lock(transaction, predicate, lock_type):
    // Lock all objects matching predicate
    // Prevents phantoms by locking gaps in index
    lock_range(predicate, lock_type)
```

**Deadlock Detection**:

```pseudocode
// Wait-For Graph
class DeadlockDetector:
    wait_for_graph: Map[transaction -> List[transaction]]

    add_wait_edge(waiting_txn, holding_txn):
        wait_for_graph[waiting_txn].append(holding_txn)

        if has_cycle(wait_for_graph):
            victim = choose_victim()  // E.g., youngest transaction
            abort(victim)
            remove_from_graph(victim)

    has_cycle(graph):
        // Run cycle detection algorithm (DFS)
        return detect_cycle_dfs(graph)
```

---

## Choosing the Right Isolation Level

### Decision Matrix

| Isolation Level | Use Case | Performance | Consistency |
|----------------|----------|-------------|-------------|
| **Read Uncommitted** | Analytics with approximate results | Highest | Lowest |
| **Read Committed** | General OLTP, web apps | High | Medium |
| **Repeatable Read** | Financial transactions, reports | Medium | High |
| **Serializable** | Critical operations, regulatory compliance | Lowest | Highest |

### Practical Guidelines

```pseudocode
// Decision Tree for Isolation Level

function choose_isolation_level(requirements):
    if requirements.regulatory_compliance:
        return SERIALIZABLE

    if requirements.financial_accuracy_required:
        return SERIALIZABLE

    if requirements.has_complex_constraints:
        // E.g., inventory management, booking systems
        return SERIALIZABLE

    if requirements.reads_must_be_consistent:
        // E.g., generating reports, multi-step workflows
        return REPEATABLE_READ

    if requirements.can_tolerate_slight_inconsistency:
        // E.g., typical web apps, content management
        return READ_COMMITTED

    if requirements.approximate_results_ok:
        // E.g., analytics, monitoring dashboards
        return READ_UNCOMMITTED

    // Default for most applications
    return READ_COMMITTED
```

### When to Use Serializable

**Must Use**:
- Financial transactions (payments, transfers)
- Inventory management with constraints
- Booking systems (seats, rooms)
- Healthcare records
- Regulatory/audit requirements

**Example Scenarios**:

```pseudocode
// Scenario 1: Meeting Room Booking
BEGIN SERIALIZABLE
bookings = SELECT * FROM bookings
           WHERE room_id = 42
           AND time_range OVERLAPS (start_time, end_time)

if len(bookings) == 0:
    INSERT INTO bookings (room_id, start_time, end_time)
    COMMIT
else:
    ROLLBACK  // Double booking prevented
```

```pseudocode
// Scenario 2: Bank Transfer with Overdraft Protection
BEGIN SERIALIZABLE
balance = SELECT balance FROM accounts WHERE id = sender
if balance >= amount:
    UPDATE accounts SET balance = balance - amount WHERE id = sender
    UPDATE accounts SET balance = balance + amount WHERE id = receiver
    COMMIT
else:
    ROLLBACK  // Insufficient funds
```

---

## Summary Table

| Anomaly | Read Uncommitted | Read Committed | Repeatable Read | Serializable |
|---------|-----------------|----------------|-----------------|--------------|
| **Dirty Reads** | Possible | ❌ Prevented | ❌ Prevented | ❌ Prevented |
| **Non-Repeatable Reads** | Possible | Possible | ❌ Prevented | ❌ Prevented |
| **Phantom Reads** | Possible | Possible | Possible* | ❌ Prevented |
| **Lost Updates** | Possible | Possible | Possible | ❌ Prevented |
| **Write Skew** | Possible | Possible | Possible | ❌ Prevented |

*Some implementations (PostgreSQL, MySQL with next-key locks) prevent phantoms at Repeatable Read.

---

## Key Takeaways

1. **Isolation is a spectrum**: Stronger isolation = better consistency but lower performance
2. **Read Committed is common default**: Good balance for most web applications
3. **Snapshot Isolation (Repeatable Read)**: Great for read-heavy workloads, prevents most anomalies
4. **Serializable is safest**: Use for critical operations where correctness matters more than performance
5. **MVCC vs 2PL**: Modern databases prefer MVCC for better concurrency
6. **Write anomalies are subtle**: Lost updates and write skew require careful handling even at Repeatable Read
7. **Test under load**: Isolation bugs often only appear under high concurrency
8. **Application-level solutions**: Sometimes explicit locking or atomic operations are simpler than Serializable isolation

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
