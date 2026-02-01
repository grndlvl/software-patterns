# ACID Transactions

## Definition

ACID is a set of properties that guarantee reliable processing of database transactions, ensuring data validity despite errors, power failures, and other mishaps.

**ACID Acronym:**
- **A**tomicity
- **C**onsistency
- **I**solation
- **D**urability

These properties work together to ensure that database transactions are processed reliably and maintain data integrity.

---

## Atomicity

**Definition:** A transaction is an indivisible unit of work - it either completes entirely or has no effect at all.

### Key Concepts

- **All-or-Nothing:** If any part of a transaction fails, the entire transaction is rolled back
- **No Partial State:** The database never ends up in a state where only some operations from a transaction have been applied
- **Rollback Capability:** The system can undo changes if a transaction fails partway through

### Example

```pseudocode
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
  // System crashes here
  UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'
COMMIT

// Result: Neither update is applied (atomicity ensures rollback)
```

### Without Atomicity

```pseudocode
// Money disappears if system crashes between operations
UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'  // Completes
[CRASH]
UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'   // Never executes
```

---

## Consistency

**Definition:** A transaction brings the database from one valid state to another valid state, preserving all defined rules and constraints.

### Key Concepts

- **Invariant Preservation:** Application-specific constraints remain true
- **Data Integrity:** Foreign keys, uniqueness constraints, check constraints are maintained
- **Application Responsibility:** The application defines what "valid state" means

### Example

```pseudocode
// Invariant: Total money in system must remain constant
BEGIN TRANSACTION
  current_alice = SELECT balance FROM accounts WHERE id = 'Alice'
  current_bob = SELECT balance FROM accounts WHERE id = 'Bob'

  IF current_alice >= 100 THEN
    UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
    UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'
  ELSE
    ROLLBACK  // Maintain invariant: don't allow negative balance
  END IF
COMMIT

// Total money before = Total money after (consistency maintained)
```

### Constraint Violations

```pseudocode
// Database enforces constraints
BEGIN TRANSACTION
  INSERT INTO users (id, email) VALUES (1, 'alice@example.com')
  INSERT INTO users (id, email) VALUES (2, 'alice@example.com')  // Violates UNIQUE constraint
COMMIT
// Result: Transaction aborted, consistency preserved
```

---

## Isolation

**Definition:** Concurrent transactions execute as if they were run sequentially, preventing interference between transactions.

### Key Concepts

- **Concurrency Control:** Prevents race conditions and conflicts
- **Isolation Levels:** Different guarantees about what concurrent transactions can see
- **Serializability:** The gold standard - results identical to serial execution

### Example: Read Committed Isolation

```pseudocode
// Transaction 1: Transfer money
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
  // Long-running computation here...
  UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'
COMMIT

// Transaction 2: Read balances (concurrent)
BEGIN TRANSACTION
  alice_bal = SELECT balance FROM accounts WHERE id = 'Alice'
  bob_bal = SELECT balance FROM accounts WHERE id = 'Bob'
  total = alice_bal + bob_bal
COMMIT

// With isolation: Transaction 2 sees either old values or new values, never partial
```

### Without Isolation: Dirty Read

```pseudocode
// Transaction 1
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
  // Not yet committed...

// Transaction 2 (concurrent)
BEGIN TRANSACTION
  alice_bal = SELECT balance FROM accounts WHERE id = 'Alice'  // Sees uncommitted value!
  // Transaction 1 rolls back - Transaction 2 read invalid data
COMMIT
```

---

## Durability

**Definition:** Once a transaction is committed, the changes are permanent - even if the system crashes immediately afterward.

### Key Concepts

- **Persistent Storage:** Data written to non-volatile storage (disk, SSD)
- **Write-Ahead Logging (WAL):** Log changes before applying them
- **Replication:** Copies on multiple machines for redundancy

### Example

```pseudocode
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
  UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'

  // Write to WAL first
  WRITE_LOG("UPDATE accounts Alice balance-100")
  WRITE_LOG("UPDATE accounts Bob balance+100")
  FLUSH_LOG_TO_DISK()

COMMIT  // Returns success only after data is durable

[SYSTEM CRASH]
[SYSTEM RESTART]
// Changes are still present - durability ensured
```

### Durability Techniques

```pseudocode
// Single-node durability
FUNCTION commit_transaction(txn):
  FOR EACH operation IN txn.operations:
    APPEND operation TO write_ahead_log
  END FOR

  CALL fsync(write_ahead_log)  // Force to disk

  APPLY operations TO database
  RETURN success

// Multi-node durability (replication)
FUNCTION commit_with_replication(txn):
  WRITE_LOG(txn) ON primary
  REPLICATE_TO(secondary_nodes, txn)
  WAIT_FOR(quorum_acks)
  RETURN success
```

---

## Single-Object vs Multi-Object Transactions

### Single-Object Operations

Operations on a single database object (row, document):

```pseudocode
// Atomic increment (single-object)
UPDATE counters SET value = value + 1 WHERE id = 'page_views'

// Compare-and-set (single-object)
UPDATE session
SET data = 'new_value'
WHERE id = 'session_123' AND version = 5

// Single document update (MongoDB-style)
db.users.updateOne(
  { _id: 'alice' },
  { $inc: { balance: -100 }, $set: { last_modified: NOW() } }
)
```

### Multi-Object Transactions

Operations spanning multiple objects requiring coordinated commits:

```pseudocode
// Multi-object transaction
BEGIN TRANSACTION
  // Read from one table
  alice_balance = SELECT balance FROM accounts WHERE id = 'Alice'

  IF alice_balance >= 100 THEN
    // Update multiple tables
    UPDATE accounts SET balance = balance - 100 WHERE id = 'Alice'
    UPDATE accounts SET balance = balance + 100 WHERE id = 'Bob'
    INSERT INTO transfers (from, to, amount) VALUES ('Alice', 'Bob', 100)
  ELSE
    ROLLBACK
  END IF
COMMIT

// All succeed together or all fail together
```

### Why Multi-Object Matters

```pseudocode
// Without multi-object transactions, foreign key constraints fail
BEGIN TRANSACTION
  user_id = INSERT INTO users (email) VALUES ('alice@example.com') RETURNING id
  INSERT INTO profiles (user_id, name) VALUES (user_id, 'Alice')
COMMIT

// If second insert fails, orphaned user record exists without transaction
```

---

## Weak vs Strong Isolation Guarantees

### Isolation Levels (Weakest to Strongest)

#### 1. Read Uncommitted
**Guarantees:** None - can see uncommitted changes from other transactions

```pseudocode
// Transaction 1
BEGIN TRANSACTION
  UPDATE accounts SET balance = 500 WHERE id = 'Alice'
  // Not committed yet...

// Transaction 2 (concurrent)
BEGIN TRANSACTION
  balance = SELECT balance FROM accounts WHERE id = 'Alice'  // May see 500 (dirty read)
COMMIT
```

#### 2. Read Committed
**Guarantees:** No dirty reads, no dirty writes

```pseudocode
// Transaction 1
BEGIN TRANSACTION
  UPDATE accounts SET balance = 500 WHERE id = 'Alice'
  // Long operation...
  COMMIT

// Transaction 2 (concurrent)
BEGIN TRANSACTION
  balance = SELECT balance FROM accounts WHERE id = 'Alice'  // Sees old value until T1 commits
  COMMIT
```

#### 3. Repeatable Read (Snapshot Isolation)
**Guarantees:** Consistent snapshot - all reads see the same version

```pseudocode
// Transaction 1
BEGIN TRANSACTION
  balance1 = SELECT balance FROM accounts WHERE id = 'Alice'  // Reads 1000

  // Transaction 2 commits a change here

  balance2 = SELECT balance FROM accounts WHERE id = 'Alice'  // Still reads 1000 (repeatable)
COMMIT
```

#### 4. Serializable
**Guarantees:** Equivalent to serial execution - no anomalies

```pseudocode
// Transaction 1: Check if room is available, then book
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE
  available = SELECT COUNT(*) FROM bookings
              WHERE room_id = 101 AND date = '2026-02-15'

  IF available = 0 THEN
    INSERT INTO bookings (room_id, date, guest)
    VALUES (101, '2026-02-15', 'Alice')
  END IF
COMMIT

// Transaction 2: Same operation (concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE
  available = SELECT COUNT(*) FROM bookings
              WHERE room_id = 101 AND date = '2026-02-15'

  IF available = 0 THEN
    INSERT INTO bookings (room_id, date, guest)
    VALUES (101, '2026-02-15', 'Bob')
  END IF
COMMIT

// Result: Only one succeeds (no double-booking)
```

---

## Complete Example: Bank Transfer

### Full ACID Implementation

```pseudocode
FUNCTION transfer_money(from_account, to_account, amount):
  BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE

    // CONSISTENCY: Check business rules
    from_balance = SELECT balance FROM accounts
                   WHERE id = from_account FOR UPDATE

    IF from_balance < amount THEN
      ROLLBACK  // Consistency: prevent negative balance
      RETURN error("Insufficient funds")
    END IF

    // ATOMICITY: Both updates succeed or both fail
    UPDATE accounts
    SET balance = balance - amount,
        last_modified = NOW()
    WHERE id = from_account

    UPDATE accounts
    SET balance = balance + amount,
        last_modified = NOW()
    WHERE id = to_account

    // Audit trail
    INSERT INTO transfer_log (from, to, amount, timestamp)
    VALUES (from_account, to_account, amount, NOW())

    // DURABILITY: Write-ahead log flushed to disk
    FLUSH_WAL_TO_DISK()

  COMMIT  // ISOLATION: Changes become visible atomically

  RETURN success
END FUNCTION

// Usage
result = transfer_money('Alice', 'Bob', 100)

// ACID guarantees:
// A: All updates applied together or none at all
// C: Balance constraints maintained
// I: Other transactions see before-state or after-state, never partial
// D: Once committed, survives crashes
```

---

## Summary Table

| Property | Question Answered | Failure Scenario Prevented | Implementation Technique |
|----------|-------------------|---------------------------|-------------------------|
| **Atomicity** | What happens if the transaction fails partway through? | Partial updates, orphaned records, inconsistent state | Write-ahead logging, rollback journals, undo logs |
| **Consistency** | Does the transaction preserve database invariants? | Constraint violations, invalid states, broken business rules | Application logic, database constraints, triggers |
| **Isolation** | Can concurrent transactions interfere with each other? | Dirty reads, lost updates, write skew, race conditions | Locking, MVCC, timestamp ordering, serialization |
| **Durability** | What happens if the system crashes after commit? | Lost committed data, data loss | Persistent storage, WAL, fsync, replication |

### Trade-offs

| Aspect | Stronger Guarantees | Weaker Guarantees |
|--------|-------------------|-------------------|
| **Correctness** | Higher | Lower |
| **Performance** | Lower (more overhead) | Higher (less overhead) |
| **Complexity** | Higher (complex protocols) | Lower (simpler protocols) |
| **Use Cases** | Financial systems, inventory | Analytics, caching, social feeds |

### When to Use What

```pseudocode
// High consistency required: Use full ACID
IF application IN [banking, inventory, reservations] THEN
  USE SERIALIZABLE isolation
  USE multi-object transactions
END IF

// High throughput required: Relax some guarantees
IF application IN [analytics, logging, social_media] THEN
  USE READ COMMITTED isolation
  USE single-object operations
  ACCEPT eventual consistency
END IF

// Middle ground: Most applications
IF application IN [e-commerce, SaaS, web_apps] THEN
  USE REPEATABLE READ isolation
  USE multi-object transactions for critical paths
  USE optimistic locking for conflicts
END IF
```

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
