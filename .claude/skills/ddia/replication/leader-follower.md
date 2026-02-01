# Leader-Follower (Single-Leader) Replication

## Definition

**Leader-Follower Replication** (also called **Master-Slave** or **Primary-Secondary** replication) is the most common replication architecture where:

- **One node** is designated as the **leader** (master/primary)
- **All writes** go to the leader only
- **Multiple followers** (slaves/read replicas/secondaries) receive changes from the leader
- **Reads** can be served by leader or followers

This pattern is used by relational databases (PostgreSQL, MySQL, Oracle) and NoSQL systems (MongoDB, RethinkDB).

---

## How It Works

```pseudocode
# Leader node
function handleWrite(data):
    applyToLocalStorage(data)
    replicationLog.append(data)
    for each follower in followers:
        sendReplicationLog(follower, replicationLog)
    return SUCCESS

# Follower node
function receiveReplicationLog(logEntry):
    applyToLocalStorage(logEntry)
    updateReplicationPosition(logEntry.sequenceNumber)
```

**Flow:**

1. Client sends write request to leader
2. Leader writes to local storage
3. Leader sends change to all followers via **replication log** (change stream)
4. Each follower applies changes in the **same order** as the leader
5. Clients can read from leader or any follower

---

## Synchronous vs Asynchronous Replication

### Synchronous Replication

**Leader waits** for follower acknowledgment before confirming write to client.

```pseudocode
function synchronousWrite(data):
    applyToLocalStorage(data)

    # Wait for at least one follower to confirm
    follower = selectSynchronousFollower()
    sendToFollower(follower, data)
    waitForAcknowledgment(follower, timeout=5000)

    # Asynchronous to other followers
    for each otherFollower in followers:
        if otherFollower != follower:
            sendToFollowerAsync(otherFollower, data)

    return SUCCESS
```

**Advantages:**
- Guaranteed up-to-date copy on at least one follower
- If leader fails, follower has all data

**Disadvantages:**
- Higher write latency (must wait for network + follower processing)
- If synchronous follower crashes, writes block until it recovers or new follower designated
- Impractical to make **all** followers synchronous (any single node failure blocks writes)

### Asynchronous Replication

**Leader does not wait** for follower acknowledgment.

```pseudocode
function asynchronousWrite(data):
    applyToLocalStorage(data)

    # Fire-and-forget to all followers
    for each follower in followers:
        sendToFollowerAsync(follower, data)

    # Immediately return to client
    return SUCCESS
```

**Advantages:**
- Low write latency
- Leader can continue even if all followers are down
- Better throughput

**Disadvantages:**
- Followers may lag behind leader
- If leader fails, recent writes may be lost
- Reads from followers may return stale data

### Semi-Synchronous (Hybrid)

**Common practice:** Make **one** follower synchronous, rest asynchronous.

- Guarantees up-to-date copy on at least two nodes (leader + one follower)
- If synchronous follower becomes unavailable, make another follower synchronous

---

## Setting Up New Followers

**Challenge:** How to create a new follower without locking the database?

### Process

```pseudocode
function setupNewFollower():
    # Step 1: Take consistent snapshot of leader
    snapshot = leader.takeConsistentSnapshot()
    snapshotPosition = snapshot.replicationLogPosition

    # Step 2: Copy snapshot to new follower
    follower.restoreSnapshot(snapshot)

    # Step 3: Connect to leader and request changes since snapshot
    follower.replicationPosition = snapshotPosition
    follower.connectToLeader()
    follower.requestChangesSince(snapshotPosition)

    # Step 4: Process backlog until caught up
    while not follower.isCaughtUp():
        logEntry = leader.getNextLogEntry()
        follower.apply(logEntry)

    # Step 5: Now ready to serve reads
    follower.status = READY
```

**Key points:**

- Snapshot must be associated with exact position in replication log (sequence number, binlog coordinates, etc.)
- Leader continues accepting writes during snapshot + catchup
- Follower processes backlog of changes that occurred since snapshot
- Once caught up, follower enters normal replication mode

---

## Handling Node Outages

### Follower Failure: Catch-up Recovery

**Scenario:** Follower crashes or network interruption.

```pseudocode
function followerRecovery():
    # Step 1: Read last processed position from local log
    lastPosition = localLog.getLastReplicationPosition()

    # Step 2: Reconnect to leader
    connectToLeader()

    # Step 3: Request all changes since last position
    requestChangesSince(lastPosition)

    # Step 4: Apply changes until caught up
    while not isCaughtUp():
        logEntry = leader.getNextLogEntry()
        apply(logEntry)
        updateReplicationPosition(logEntry.sequenceNumber)

    # Step 5: Resume normal replication
    status = READY
```

**Simple because:** Each follower maintains its position in the replication log.

### Leader Failure: Failover

**Much more complex.** Must promote a follower to new leader.

```pseudocode
function failover():
    # Step 1: Detect leader failure
    if not leader.isResponding(timeout=30_000):
        # Step 2: Choose new leader from followers
        newLeader = selectMostUpToDateFollower()

        # Step 3: Reconfigure system
        newLeader.promoteToLeader()

        # Step 4: Point all followers to new leader
        for each follower in followers:
            if follower != newLeader:
                follower.setLeader(newLeader)

        # Step 5: Point clients to new leader
        updateClientConfiguration(newLeader)

function selectMostUpToDateFollower():
    candidates = followers.filter(f => f.isHealthy())
    return candidates.maxBy(f => f.replicationPosition)
```

**Challenges:**

1. **Determining leader failure:** Timeout-based detection may have false positives
2. **Data loss:** Asynchronous followers may not have all writes
3. **Split brain:** Two nodes both think they're leader (violates single-leader guarantee)
4. **Replication position conflict:** Must discard unreplicated writes from old leader
5. **Timeout tuning:** Too short = unnecessary failovers; too long = longer recovery time

**Example conflict scenario:**

```pseudocode
# Before failover
Leader: [write1, write2, write3]  # write3 not yet replicated
Follower1: [write1, write2]       # Most up-to-date follower
Follower2: [write1]

# Leader crashes, Follower1 promoted to new leader
NewLeader: [write1, write2, write4]  # New write from client

# Old leader recovers
OldLeader: [write1, write2, write3]  # Has write3 that conflicts with write4

# Resolution: Discard write3 (data loss!)
```

---

## Replication Log Implementations

### 1. Statement-Based Replication

**Leader logs every write statement** (SQL INSERT, UPDATE, DELETE) and sends to followers.

```pseudocode
# Leader
function handleWrite(sqlStatement):
    executeStatement(sqlStatement)
    replicationLog.append(sqlStatement)
    broadcastToFollowers(sqlStatement)

# Follower
function receiveStatement(sqlStatement):
    executeStatement(sqlStatement)
```

**Problems:**

- **Nondeterministic functions:** `NOW()`, `RAND()` produce different values on each node
- **Auto-incrementing columns:** If statement uses `INSERT INTO ... SELECT`, depends on existing data
- **Side effects:** Triggers, stored procedures may behave differently on each node
- **Order dependencies:** Statements with same timestamp may execute in different order

**Solution:** Leader must replace nondeterministic functions with fixed values, but complex and error-prone.

### 2. Write-Ahead Log (WAL) Shipping

**Leader sends the raw write-ahead log** (disk-level data structure changes).

```pseudocode
# Leader
function handleWrite(data):
    walEntry = createWALEntry(data)  # Low-level: "write bytes X to page Y at offset Z"
    wal.append(walEntry)
    applyToStorage(walEntry)
    broadcastToFollowers(walEntry)

# Follower
function receiveWALEntry(walEntry):
    wal.append(walEntry)
    applyToStorage(walEntry)
```

**Used by:** PostgreSQL, Oracle

**Problems:**

- **Tightly coupled to storage engine:** WAL describes changes at byte-level
- **Version incompatibility:** Cannot run different database versions on leader vs followers
- **Zero-downtime upgrades impossible:** Must take database offline to upgrade

### 3. Logical (Row-Based) Log Replication

**Decoupled from storage engine.** Logs **logical changes** to rows.

```pseudocode
# Leader
function handleWrite(operation, tableName, row):
    logEntry = createLogicalLogEntry({
        operation: operation,    # INSERT, UPDATE, DELETE
        table: tableName,
        row: row,                # New values for INSERT/UPDATE
        oldRow: oldRow           # Old values for UPDATE (for identification)
    })
    logicalLog.append(logEntry)
    applyToStorage(logEntry)
    broadcastToFollowers(logEntry)

# Example logical log entries
{operation: "INSERT", table: "users", row: {id: 42, name: "Alice"}}
{operation: "UPDATE", table: "users", oldRow: {id: 42}, row: {id: 42, name: "Alice Smith"}}
{operation: "DELETE", table: "users", row: {id: 42}}
```

**Advantages:**

- **Version independent:** Can run different database versions
- **Easier to parse by external systems:** Change Data Capture (CDC) tools can consume
- **Zero-downtime upgrades:** Followers can run newer version than leader

**Used by:** MySQL binlog (when configured for row-based replication), PostgreSQL logical decoding

### 4. Trigger-Based Replication

**Application-level replication** using database triggers.

```pseudocode
# Setup: Create trigger that logs changes to special table
CREATE TRIGGER replicate_users_insert
AFTER INSERT ON users
FOR EACH ROW
BEGIN
    INSERT INTO replication_log (table, operation, data)
    VALUES ('users', 'INSERT', NEW.*);
END;

# Application reads replication_log and sends to followers
function replicationDaemon():
    while true:
        entries = SELECT * FROM replication_log WHERE processed = false
        for entry in entries:
            broadcastToFollowers(entry)
            markProcessed(entry)
        sleep(100)
```

**Advantages:**

- **Flexible:** Can replicate subset of data, add custom transformations
- **Application-controlled:** Can implement complex logic

**Disadvantages:**

- **High overhead:** More complex than built-in replication
- **Bugs and edge cases:** Custom code is error-prone
- **Performance:** Slower than native replication

**Used by:** Oracle GoldenGate, Databus for Oracle

---

## Read Scaling Benefits

```pseudocode
# Simple load balancing across followers
function handleRead(query):
    if query.requiresLatestData():
        # Must read from leader for consistency
        return leader.execute(query)
    else:
        # Can read from any follower
        follower = selectRandomFollower()
        return follower.execute(query)

function selectRandomFollower():
    return followers[random(0, followers.length - 1)]
```

**Benefits:**

- Distribute read load across multiple nodes
- Improve read throughput
- Reduce latency by placing followers geographically closer to users

**Limitations:**

- Only works for **read-heavy** workloads
- **Write throughput** limited by single leader
- Asynchronous replication creates consistency challenges (see below)

---

## Consistency Challenges

### Replication Lag

**Eventual consistency:** Followers will catch up **eventually**, but may lag behind.

```pseudocode
# Replication lag example
t=0:  Leader writes X=1
      Follower1 has X=0  (lag: 0.5 seconds)
      Follower2 has X=0  (lag: 2.0 seconds)

t=1:  User writes X=1 to leader, then reads from Follower2
      Returns X=0  (stale read!)

t=3:  All followers caught up, X=1 everywhere
```

**Normally negligible** (< 1 second), but can grow due to:
- Network issues
- Follower overload
- Leader writing faster than follower can process

### Read-Your-Writes Consistency

**Problem:** User writes data, then reads from follower that hasn't caught up yet.

```pseudocode
# Bad: User may not see their own write
function updateProfile(userId, newData):
    leader.write(userId, newData)
    return SUCCESS

function getProfile(userId):
    follower = selectRandomFollower()
    return follower.read(userId)  # May return old data!

# User flow
updateProfile(42, {name: "Alice"})
profile = getProfile(42)  # Might return old name if read from lagging follower
```

**Solutions:**

```pseudocode
# Solution 1: Read user's own data from leader
function getProfile(userId):
    if userId == currentUser.id:
        return leader.read(userId)  # Always fresh
    else:
        return selectRandomFollower().read(userId)  # Others can be stale

# Solution 2: Track timestamp of last write
function updateProfile(userId, newData):
    timestamp = leader.write(userId, newData)
    currentUser.lastWriteTimestamp = timestamp

function getProfile(userId):
    if userId == currentUser.id:
        # Only read from followers caught up past user's last write
        follower = selectFollowerWithReplicationTime(currentUser.lastWriteTimestamp)
        return follower.read(userId)
    else:
        return selectRandomFollower().read(userId)

# Solution 3: Read from leader for N seconds after write
function getProfile(userId):
    if userId == currentUser.id and (now() - currentUser.lastWriteTime) < 60_000:
        return leader.read(userId)
    else:
        return selectRandomFollower().read(userId)
```

### Monotonic Reads

**Problem:** User reads from different followers, sees data "go backward in time."

```pseudocode
# Bad: User sees time travel
t=0:  User reads comment from Follower1 (caught up): sees comment
t=1:  User reads again from Follower2 (lagging): comment disappears!

# Example
query1 = follower1.execute("SELECT * FROM comments WHERE post_id = 42")
# Returns: [{id: 1, text: "Great post!"}]

query2 = follower2.execute("SELECT * FROM comments WHERE post_id = 42")
# Returns: []  (Follower2 hasn't received the comment yet)
```

**Solution:** Always read from **same follower** for a given user.

```pseudocode
function selectFollowerForUser(userId):
    # Hash user ID to consistently route to same follower
    followerIndex = hash(userId) % followers.length
    return followers[followerIndex]

function handleRead(userId, query):
    follower = selectFollowerForUser(userId)
    return follower.execute(query)
```

### Consistent Prefix Reads

**Problem:** Reads see data in wrong **causal order** (violates happens-before relationship).

```pseudocode
# Example: Conversation between two users
User1 writes: "What's the capital of France?"
User2 writes: "Paris"

# Observer reading from different partitions might see
"Paris"  (from Follower1, caught up)
"What's the capital of France?"  (from Follower2, lagging)

# Appears as if User2 answered before question was asked!
```

**Solution:** Ensure **causally related writes** go to same partition or use **version vectors**.

```pseudocode
# Ensure related data goes to same partition
function writeMessage(conversationId, message):
    partition = hash(conversationId) % numPartitions
    leader.writeToPartition(partition, message)

# Or use version vectors to track causality
function writeWithCausality(message):
    message.vectorClock = currentVectorClock
    leader.write(message)
    updateVectorClock()
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Write Path** | All writes to leader only |
| **Read Path** | Leader or any follower |
| **Replication** | Leader sends log to followers; followers apply in same order |
| **Synchronous Replication** | Leader waits for follower ACK; guarantees durability but higher latency |
| **Asynchronous Replication** | Leader doesn't wait; lower latency but potential data loss on leader failure |
| **New Follower Setup** | Snapshot + apply changes since snapshot position |
| **Follower Recovery** | Reconnect + catch up from last known position |
| **Leader Failover** | Complex: detect failure, promote follower, reconfigure clients |
| **Log Types** | Statement-based, WAL, logical (row-based), trigger-based |
| **Read Scaling** | Works well for read-heavy workloads |
| **Consistency Challenges** | Replication lag → read-your-writes, monotonic reads, consistent prefix |
| **Use Cases** | PostgreSQL, MySQL, MongoDB, Oracle, most relational databases |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
