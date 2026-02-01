# Linearizability

## Definition

**Linearizability** (also called atomic consistency or strong consistency) is the strongest consistency guarantee for concurrent systems. It makes a distributed system appear as if there is only a single copy of the data, and all operations on it are atomic.

**Core guarantee:** Once a read returns a value, all subsequent reads must return that value or a newer value, even if they happen on different nodes.

## What Makes a System Linearizable

A system is linearizable if it satisfies these properties:

### 1. **Total Order**
All operations appear to execute atomically at some point between their invocation and completion (the linearization point).

### 2. **Real-Time Ordering**
If operation A completes before operation B begins, then A must appear before B in the total order.

### 3. **Recency Guarantee**
Once a write completes, all subsequent reads must see that write or a newer value.

## Visual Example

```pseudocode
Timeline (time flows left to right):

Client 1: write(x=1) [====]             read(x) → 1
Client 2:                  read(x) → 0     read(x) → 1
Client 3:                     read(x) → 1

NOT LINEARIZABLE: Client 2's first read returned 0 after Client 1's write completed
```

```pseudocode
Timeline (linearizable):

Client 1: write(x=1) [====]             read(x) → 1
Client 2:                  read(x) → 1     read(x) → 1
Client 3:                     read(x) → 1

LINEARIZABLE: All reads after write completion see the new value
```

## Linearizability vs. Serializability

**IMPORTANT:** These are different concepts often confused!

| Aspect | Linearizability | Serializability |
|--------|----------------|-----------------|
| **Scope** | Individual operations (reads/writes) | Transactions (groups of operations) |
| **Ordering** | Real-time order of operations | Equivalent to some serial execution |
| **Recency** | Guarantees reading latest value | No recency guarantee |
| **Use case** | Coordination, single-object ops | Multi-object transactions |
| **Example** | Read-your-writes for single key | Bank transfer (debit + credit) |

### Key Difference Illustrated

```pseudocode
// Serializable but NOT linearizable:
Thread 1: write(x=1) completes at T1
Thread 2: read(x) at T2 (T2 > T1) returns 0
// Still serializable if equivalent to Thread 2 → Thread 1 order

// Linearizable implies recency:
Thread 1: write(x=1) completes at T1
Thread 2: read(x) at T2 (T2 > T1) MUST return 1 or newer
```

**You can have both:** Strict serializability = Serializability + Linearizability

## Use Cases for Linearizability

### 1. **Leader Election**
Only one node should believe it's the leader.

```pseudocode
function electLeader(nodeId):
    if compareAndSet(leaderKey, null, nodeId):
        return "I am leader"
    else:
        return "Someone else is leader"

// Linearizability ensures only one CAS succeeds
```

### 2. **Distributed Locks**
Ensure mutual exclusion across nodes.

```pseudocode
function acquireLock(lockName, clientId):
    success = compareAndSet(lockName, null, clientId)
    return success

function releaseLock(lockName, clientId):
    compareAndSet(lockName, clientId, null)
```

### 3. **Uniqueness Constraints**
Guarantee unique usernames, account numbers, etc.

```pseudocode
function registerUsername(username, userId):
    if read(username) == null:
        success = compareAndSet(username, null, userId)
        if success:
            return "Registered"
    return "Username taken"

// Without linearizability, two clients could both see null
// and both believe they successfully registered
```

### 4. **Cross-Channel Timing Dependencies**
When writes in one channel affect reads in another.

```pseudocode
// User uploads image, then sends message referencing it
write(image, imageData)           // Channel 1: Storage
write(message, "See image.jpg")   // Channel 2: Messages

// Without linearizability:
read(message) → "See image.jpg"
read(image) → null  // Race condition!
```

## Cost of Linearizability

### CAP Theorem Implications

**CAP Theorem:** In a distributed system with network partitions, you must choose between:
- **C**onsistency (linearizability)
- **A**vailability (every request gets a response)

You cannot have both during a partition.

### Network Partition Scenario

```pseudocode
                Network Partition
                        |
    [Node A]            |            [Node B]
    (isolated)          |          (isolated)
        |               |               |
        v               |               v
    Option 1:           |           Option 1:
    Return error        |           Return error
    (CP - consistent    |           (CP - consistent
     but unavailable)   |            but unavailable)
        |               |               |
        v               |               v
    Option 2:           |           Option 2:
    Serve stale data    |           Accept writes
    (AP - available     |           (AP - available
     but inconsistent)  |            but inconsistent)
```

### Performance Cost

Linearizability requires coordination:

| Implementation | Cost |
|----------------|------|
| **Single-leader replication** | All writes go through leader (slow if leader is far) |
| **Consensus (Raft/Paxos)** | Requires majority quorum (network round-trips) |
| **Multi-leader** | Cannot guarantee linearizability |
| **Leaderless** | Cannot guarantee linearizability (even with quorum) |

## Implementing Linearizability

### Approach 1: Single-Leader Replication

```pseudocode
class SingleLeaderStorage:
    def __init__(self):
        self.leader = self
        self.data = {}
        self.replicas = []

    def write(self, key, value):
        if self != leader:
            return leader.write(key, value)

        self.data[key] = value

        // Wait for majority of replicas to acknowledge
        acks = 0
        for replica in self.replicas:
            if replica.replicate(key, value):
                acks += 1

        if acks >= len(self.replicas) / 2:
            return SUCCESS
        else:
            return FAILURE

    def read(self, key):
        if self != leader:
            return leader.read(key)

        return self.data.get(key)

// Linearizable if:
// 1. All reads go to leader
// 2. Leader waits for replication before acknowledging writes
```

### Approach 2: Consensus Algorithm (Raft)

```pseudocode
class RaftNode:
    def __init__(self):
        self.state = {}
        self.log = []
        self.commitIndex = 0

    def write(self, key, value):
        logEntry = LogEntry(key, value, len(self.log))
        self.log.append(logEntry)

        // Replicate to majority
        acks = self.replicateToMajority(logEntry)

        if acks >= (clusterSize / 2) + 1:
            self.commitIndex += 1
            self.state[key] = value
            return SUCCESS
        else:
            return FAILURE

    def read(self, key):
        // Option 1: Read from leader (linearizable)
        if self.isLeader():
            return self.state.get(key)
        else:
            return self.forwardToLeader(key)

        // Option 2: Read from replica (NOT linearizable without lease)

// Linearizable because:
// - Writes require majority consensus
// - Reads go through leader who has latest committed state
```

### Approach 3: Compare-and-Set Operations

```pseudocode
class LinearizableRegister:
    def __init__(self):
        self.value = null
        self.version = 0
        self.lock = Lock()

    def compareAndSet(self, expectedValue, newValue):
        with self.lock:
            if self.value == expectedValue:
                self.value = newValue
                self.version += 1
                return True
            return False

    def read(self):
        with self.lock:
            return (self.value, self.version)

// Atomic operations ensure linearizability
```

## Testing for Linearizability

### Technique 1: Linearization Point Detection

```pseudocode
class LinearizabilityChecker:
    def __init__(self):
        self.history = []  // List of (operation, start_time, end_time, result)

    def recordOperation(self, op, start, end, result):
        self.history.append((op, start, end, result))

    def verify(self):
        // Try to find a linearization point for each operation
        // such that:
        // 1. Point is between start and end
        // 2. Operations ordered by linearization points
        //    produce observed results

        for permutation in allPermutations(self.history):
            if self.isValidLinearization(permutation):
                return True

        return False

    def isValidLinearization(self, order):
        state = {}

        for (op, start, end, result) in order:
            if op.type == WRITE:
                state[op.key] = op.value
            elif op.type == READ:
                if state.get(op.key) != result:
                    return False

            // Check real-time ordering
            for (otherOp, otherStart, otherEnd, otherResult) in order:
                if otherEnd < start and not appearsBeforeInOrder(otherOp, op):
                    return False

        return True
```

### Technique 2: Jepsen-Style Testing

```pseudocode
class JepsenTest:
    def __init__(self, system):
        self.system = system
        self.clients = []

    def runTest(self):
        // 1. Start multiple concurrent clients
        for i in range(10):
            client = Client(self.system)
            client.startRandomOperations()
            self.clients.append(client)

        // 2. Induce network partitions
        self.inducePartition()
        time.sleep(10)
        self.healPartition()

        // 3. Collect operation history
        history = []
        for client in self.clients:
            history.extend(client.getHistory())

        // 4. Check linearizability
        checker = LinearizabilityChecker()
        for op in history:
            checker.recordOperation(op)

        if checker.verify():
            print("System is linearizable")
        else:
            print("Linearizability violation detected!")
            print(checker.getViolationExample())
```

## Practical Pseudocode Examples

### Example 1: Linearizable Counter

```pseudocode
class LinearizableCounter:
    def __init__(self, consensusService):
        self.consensus = consensusService
        self.key = "counter"

    def increment(self):
        while True:
            (currentValue, version) = self.consensus.read(self.key)
            newValue = currentValue + 1

            if self.consensus.compareAndSet(self.key, currentValue, newValue, version):
                return newValue

            // CAS failed, retry

    def get(self):
        (value, version) = self.consensus.read(self.key)
        return value

// Usage:
counter = LinearizableCounter(raftCluster)

// From different threads/nodes:
Thread 1: counter.increment()  // Returns 1
Thread 2: counter.increment()  // Returns 2
Thread 3: counter.get()        // Returns 2 or higher, never less
```

### Example 2: Linearizable Queue

```pseudocode
class LinearizableQueue:
    def __init__(self, consensusService):
        self.consensus = consensusService
        self.headKey = "queue_head"
        self.tailKey = "queue_tail"

    def enqueue(self, item):
        while True:
            (tail, version) = self.consensus.read(self.tailKey)

            // Write item at tail position
            self.consensus.write(f"queue_item_{tail}", item)

            // Try to advance tail
            if self.consensus.compareAndSet(self.tailKey, tail, tail + 1, version):
                return SUCCESS

    def dequeue(self):
        while True:
            (head, headVersion) = self.consensus.read(self.headKey)
            (tail, tailVersion) = self.consensus.read(self.tailKey)

            if head >= tail:
                return null  // Queue empty

            item = self.consensus.read(f"queue_item_{head}")

            if self.consensus.compareAndSet(self.headKey, head, head + 1, headVersion):
                return item
```

### Example 3: Detecting Linearizability Violation

```pseudocode
class NonLinearizableExample:
    // This demonstrates a violation

    def __init__(self):
        self.nodeA_value = 0
        self.nodeB_value = 0

    def write_to_nodeA(self, value):
        self.nodeA_value = value
        // Async replication to B (not waited)
        asyncReplicate(nodeB, value)

    def read_from_nodeB(self):
        return self.nodeB_value

// Execution timeline:
Client 1: write_to_nodeA(42) completes at T1
Client 2: read_from_nodeB() at T2 (T2 > T1) returns 0

// VIOLATION: Client 2's read should see 42 or newer
// This system is NOT linearizable
```

## Summary Table

| Property | Description | Example |
|----------|-------------|---------|
| **Guarantee** | System behaves like single copy | Read after write sees new value |
| **Ordering** | Total order + real-time respect | If A completes before B starts, A < B |
| **Recency** | Reads see latest or newer | No stale reads after write completes |
| **Cost** | Requires coordination | Cannot be available during partition |
| **Use cases** | Locks, leaders, constraints | compareAndSet, elect leader |
| **vs. Serializability** | Different concepts | Can have one without the other |
| **Implementation** | Consensus or single leader | Raft, ZooKeeper, etcd |
| **CAP tradeoff** | Consistency over availability | Return error if no majority |

## When to Use Linearizability

✅ **USE when:**
- Need locks or leader election
- Enforcing uniqueness constraints
- Cross-channel dependencies
- Strong coordination required
- Small-scale critical data

❌ **AVOID when:**
- High availability is critical
- Geographically distributed
- Performance is priority
- Eventual consistency is acceptable
- Large-scale data storage

## Key Takeaways

1. **Linearizability = strongest single-object consistency**
2. **Different from serializability** (transactions vs operations)
3. **Costs availability** during network partitions (CAP)
4. **Requires coordination** (consensus, single leader)
5. **Essential for** locks, leaders, constraints
6. **Not always necessary** - eventual consistency often sufficient
7. **Test carefully** using history analysis tools

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
