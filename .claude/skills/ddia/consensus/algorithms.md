# Consensus Algorithms

## Definition

**Consensus** is the fundamental problem of getting multiple nodes in a distributed system to agree on a single value or decision, even in the presence of failures. Solving consensus is critical for:

- **Leader election**: Ensuring only one node acts as leader
- **Atomic commit**: All nodes commit or abort a transaction together
- **Total order broadcast**: Delivering messages in the same order to all nodes
- **State machine replication**: Keeping replicas in sync by applying operations in the same order

Consensus enables building fault-tolerant distributed systems where correct behavior is guaranteed despite node crashes, network partitions, and message delays.

---

## The FLP Impossibility Result

The **Fischer-Lynch-Paterson (FLP) impossibility theorem** (1985) proves that:

> In an asynchronous distributed system, there is no deterministic algorithm that can guarantee consensus in finite time if even a single process may fail.

**Key implications:**
- **Asynchronous network**: No bounds on message delays or processing time
- **Deterministic algorithms**: Cannot use randomization
- **Even one crash**: Just one node failing makes consensus impossible to guarantee

**Why this matters:**
- Real networks are asynchronous (no reliable timing guarantees)
- We must accept that consensus may not always terminate
- Practical algorithms use timeouts and leader election (which may be unstable)

**Workarounds in practice:**
- Use **partially synchronous** model (bounded delays *eventually*)
- Accept that progress may stall during network problems
- Use randomization (not covered by FLP)
- Trade liveness for safety (prefer correctness over availability)

---

## Paxos

**Paxos** is the original consensus algorithm, invented by Leslie Lamport in 1989. It's notoriously difficult to understand but forms the theoretical foundation for many modern systems.

### Core Concepts

**Roles:**
- **Proposers**: Propose values
- **Acceptors**: Vote on proposals
- **Learners**: Learn the chosen value

**Two phases:**

**Phase 1 (Prepare):**
1. Proposer selects unique proposal number `n`
2. Sends `PREPARE(n)` to majority of acceptors
3. Acceptors promise not to accept proposals < `n`
4. Acceptors respond with highest-numbered proposal they've accepted

**Phase 2 (Accept):**
1. If majority responds, proposer sends `ACCEPT(n, v)`
   - `v` = value from highest-numbered prior proposal, or proposer's value
2. Acceptors accept if they haven't promised a higher number
3. When majority accepts, value is chosen

**Safety guarantee:**
- Only one value can be chosen
- A node never learns that a value has been chosen unless it actually has been

**Liveness challenge:**
- Competing proposers can block each other (dueling proposers)
- Requires additional mechanisms (like leader election) to ensure progress

**Why Paxos is hard:**
- Complex protocol with many edge cases
- Difficult to implement correctly
- Multi-Paxos (for multiple decisions) adds more complexity

---

## Raft

**Raft** was designed as an understandable alternative to Paxos, published by Diego Ongaro and John Ousterhout in 2014. It explicitly optimizes for comprehensibility.

### Leader Election

**Three states:**
- **Follower**: Passive, responds to leader and candidates
- **Candidate**: Actively seeking election
- **Leader**: Handles all client requests, replicates log

**Election process:**

```pseudocode
# Follower timeout triggers election
function startElection():
    currentTerm++
    state = CANDIDATE
    votedFor = self
    votesReceived = 1

    for each server in cluster:
        send RequestVote(
            term: currentTerm,
            candidateId: self,
            lastLogIndex: log.lastIndex(),
            lastLogTerm: log.lastTerm()
        )

# Vote handling
function handleRequestVote(request):
    if request.term < currentTerm:
        return vote: false

    if votedFor is null or votedFor == request.candidateId:
        if request.lastLogIndex >= log.lastIndex():
            votedFor = request.candidateId
            return vote: true

    return vote: false

# Becoming leader
function onMajorityVotes():
    state = LEADER
    for each server:
        nextIndex[server] = log.lastIndex() + 1
        matchIndex[server] = 0

    sendHeartbeats()  # Establish authority
```

**Election properties:**
- **Election timeout**: Randomized (150-300ms) to reduce split votes
- **Safety**: At most one leader per term
- **Liveness**: Eventually elects a leader if majority available

### Log Replication

**Append entries process:**

```pseudocode
# Leader receives client command
function handleClientCommand(command):
    log.append(Entry(term: currentTerm, command: command))

    for each follower:
        send AppendEntries(
            term: currentTerm,
            leaderId: self,
            prevLogIndex: nextIndex[follower] - 1,
            prevLogTerm: log[prevLogIndex].term,
            entries: log[nextIndex[follower]...],
            leaderCommit: commitIndex
        )

# Follower receives append entries
function handleAppendEntries(request):
    if request.term < currentTerm:
        return success: false

    if log[request.prevLogIndex].term != request.prevLogTerm:
        # Log inconsistency - reject
        return success: false

    # Delete conflicting entries
    log.deleteFrom(request.prevLogIndex + 1)

    # Append new entries
    log.append(request.entries)

    # Update commit index
    if request.leaderCommit > commitIndex:
        commitIndex = min(request.leaderCommit, log.lastIndex())

    return success: true

# Leader tracks replication
function onAppendEntriesSuccess(follower, matchIndex):
    nextIndex[follower] = matchIndex + 1
    matchIndex[follower] = matchIndex

    # Check if we can commit
    for N from commitIndex + 1 to log.lastIndex():
        if log[N].term == currentTerm:
            if majority of matchIndex[i] >= N:
                commitIndex = N
                applyToStateMachine(log[commitIndex])
```

**Log properties:**
- **Log matching**: If two logs contain entry with same index and term, they're identical up to that point
- **Leader completeness**: If entry is committed, all future leaders have it
- **State machine safety**: If a server applies log entry at index i, no other server applies different entry at i

**Advantages:**
- **Understandability**: Clear separation of leader election and log replication
- **Strong leader**: All log entries flow from leader to followers
- **Membership changes**: Built-in support for adding/removing nodes

---

## Zab (ZooKeeper Atomic Broadcast)

**Zab** is the consensus protocol used by Apache ZooKeeper, designed for total order broadcast.

### Key Concepts

**Primary-backup architecture:**
- **Leader**: Accepts all write requests, assigns sequence numbers
- **Followers**: Replicate leader's state, serve reads

**Two modes:**
- **Recovery mode**: Electing a new leader, synchronizing state
- **Broadcast mode**: Normal operation, replicating updates

**Ordering guarantees:**
- **Total order**: All writes delivered in same order to all nodes
- **Causal order**: If write A happens before write B, A delivered first
- **Prefix property**: Earlier messages committed before later ones

**Epochs (similar to Raft terms):**
- Each leader has unique epoch number
- Epoch changes only during leader election
- Transaction IDs include epoch: `(epoch, counter)`

**Three-phase protocol:**

1. **Discovery**: Find most recent state among nodes
2. **Synchronization**: Leader brings followers up to date
3. **Broadcast**: Normal operation, two-phase commit for each transaction

**Comparison to Paxos/Raft:**
- Optimized for high-throughput atomic broadcast
- Simpler recovery (followers always sync from leader)
- Used in production by ZooKeeper, Yahoo!, and others

---

## Comparison of Algorithms

| Aspect | Paxos | Raft | Zab |
|--------|-------|------|-----|
| **Goal** | Single-value consensus | Replicated log | Total order broadcast |
| **Leader** | Optional (Multi-Paxos) | Strong, required | Strong, required |
| **Understandability** | Difficult | Designed for clarity | Moderate |
| **Performance** | High (fewer messages) | Moderate | High (optimized for broadcast) |
| **Production use** | Google Chubby, Spanner | etcd, Consul, CockroachDB | ZooKeeper |
| **Membership changes** | Complex | Built-in | Supported |
| **Implementation complexity** | Very high | Moderate | Moderate |

**When to use each:**
- **Paxos**: Theoretical foundation, use implementations (not raw Paxos)
- **Raft**: Building new distributed systems, want understandability
- **Zab**: Need high-throughput ordered broadcast (like ZooKeeper)

---

## Practical Systems

### Apache ZooKeeper

**Purpose**: Coordination service for distributed applications

**Use cases:**
- **Configuration management**: Centralized config store
- **Leader election**: Distributed lock for leader selection
- **Service discovery**: Registry of available services
- **Distributed coordination**: Barriers, queues, locks

**Guarantees:**
- **Sequential consistency**: Updates applied in order
- **Atomicity**: Updates succeed or fail completely
- **Single system image**: Same view regardless of server
- **Reliability**: Persists updates once applied
- **Timeliness**: Bounded staleness guarantees

**Implementation:**
- Uses Zab protocol
- Typically 3-5 node ensemble
- Tolerates (n-1)/2 failures

### etcd

**Purpose**: Distributed key-value store for configuration and coordination

**Use cases:**
- **Kubernetes**: Stores cluster state and configuration
- **Service discovery**: Service registration and health
- **Distributed locking**: Coordination primitives
- **Leader election**: Via TTL leases

**Features:**
- Uses Raft consensus
- gRPC API for high performance
- Watch mechanism for change notifications
- Multi-version concurrency control (MVCC)
- Transactions with compare-and-swap

### Consul

**Purpose**: Service mesh solution with consensus

**Use cases:**
- **Service discovery**: DNS and HTTP interfaces
- **Health checking**: Automatic failure detection
- **Key-value store**: Configuration and coordination
- **Multi-datacenter**: WAN gossip protocol

**Architecture:**
- Raft consensus within datacenter
- Gossip protocol (Serf) between datacenters
- Agent on every node
- Server nodes form consensus group

**Comparison:**

| Feature | ZooKeeper | etcd | Consul |
|---------|-----------|------|--------|
| **Protocol** | Zab | Raft | Raft + Gossip |
| **Data model** | Hierarchical znodes | Flat key-value | Key-value + service catalog |
| **API** | Custom | gRPC | HTTP + DNS |
| **Primary use** | Coordination | Config store | Service mesh |
| **Watches** | Yes | Yes (streams) | Yes (blocking queries) |
| **Multi-DC** | Via observers | Via proxies | Native support |

---

## When You Need Consensus

### Use consensus when:

**✓ Correctness is critical:**
- Financial transactions
- Inventory management
- Medical records
- Any system where inconsistency causes serious problems

**✓ Need strong consistency:**
- Leader election (exactly one leader)
- Distributed locks (mutual exclusion)
- Linearizable operations (appear atomic)
- Configuration management (everyone sees same config)

**✓ Coordination required:**
- Cluster membership changes
- Work distribution (exactly-once processing)
- Barriers and rendezvous points
- Service discovery with failover

### Avoid consensus when:

**✗ High availability needed:**
- Consensus requires majority (unavailable during partitions)
- Consider eventual consistency instead

**✗ Low latency required:**
- Consensus adds network round-trips
- Use caching, read replicas, or eventual consistency

**✗ High write throughput:**
- Single leader is bottleneck
- Consider sharding or leaderless replication

**✗ Geographically distributed:**
- Cross-datacenter latency expensive
- Use conflict-free data types (CRDTs) or application-level resolution

### Alternatives to consensus:

| Pattern | Use when | Trade-off |
|---------|----------|-----------|
| **Eventual consistency** | High availability > strong consistency | Temporary inconsistency acceptable |
| **Leaderless replication** | No coordination needed | Requires conflict resolution |
| **CRDTs** | Concurrent updates common | Limited operation types |
| **Client-side coordination** | Small number of clients | Clients must be trusted |
| **Uniqueness constraints** | Database can enforce | Single database becomes bottleneck |

---

## Pseudocode Example: Raft Leader Election

Complete implementation of Raft leader election:

```pseudocode
# Server state
state = FOLLOWER
currentTerm = 0
votedFor = null
log = []
commitIndex = 0
lastApplied = 0

# Leader state
nextIndex = {}
matchIndex = {}

# Timeouts
electionTimeout = randomRange(150, 300)  # milliseconds
heartbeatInterval = 50  # milliseconds

# Main loop
function main():
    while true:
        if state == FOLLOWER:
            followerLoop()
        else if state == CANDIDATE:
            candidateLoop()
        else if state == LEADER:
            leaderLoop()

# Follower behavior
function followerLoop():
    resetElectionTimer()

    while state == FOLLOWER:
        if electionTimedOut():
            becomeCandidate()
            break

        handleIncomingMessages()

# Candidate behavior
function candidateLoop():
    currentTerm++
    votedFor = self
    votesReceived = 1
    resetElectionTimer()

    sendRequestVoteToAll()

    while state == CANDIDATE:
        if electionTimedOut():
            # Start new election
            currentTerm++
            votedFor = self
            votesReceived = 1
            resetElectionTimer()
            sendRequestVoteToAll()

        message = receiveMessage()

        if message.type == VOTE_RESPONSE:
            if message.term > currentTerm:
                currentTerm = message.term
                becomeFollower()
                break

            if message.voteGranted:
                votesReceived++

                if votesReceived > cluster.size() / 2:
                    becomeLeader()
                    break

        else if message.type == APPEND_ENTRIES:
            if message.term >= currentTerm:
                currentTerm = message.term
                becomeFollower()
                break

# Leader behavior
function leaderLoop():
    initializeLeaderState()
    sendHeartbeatsToAll()

    while state == LEADER:
        if heartbeatIntervalElapsed():
            sendHeartbeatsToAll()

        message = receiveMessage()

        if message.type == APPEND_ENTRIES_RESPONSE:
            if message.term > currentTerm:
                currentTerm = message.term
                becomeFollower()
                break

            if message.success:
                updateFollowerProgress(message.from, message.matchIndex)
                tryCommit()
            else:
                # Retry with earlier index
                nextIndex[message.from]--
                sendAppendEntries(message.from)

        else if message.type == CLIENT_REQUEST:
            appendToLog(message.command)
            replicateToFollowers()

# Request vote RPC
function sendRequestVote(server):
    request = {
        type: REQUEST_VOTE,
        term: currentTerm,
        candidateId: self,
        lastLogIndex: log.length - 1,
        lastLogTerm: log[log.length - 1].term if log.length > 0 else 0
    }
    send(server, request)

function handleRequestVote(request):
    response = {
        type: VOTE_RESPONSE,
        term: currentTerm,
        voteGranted: false
    }

    # Reject if candidate's term is outdated
    if request.term < currentTerm:
        send(request.candidateId, response)
        return

    # Update term if candidate's is higher
    if request.term > currentTerm:
        currentTerm = request.term
        votedFor = null
        becomeFollower()

    # Vote if we haven't voted and candidate's log is up-to-date
    if votedFor is null or votedFor == request.candidateId:
        if isLogUpToDate(request.lastLogIndex, request.lastLogTerm):
            votedFor = request.candidateId
            response.voteGranted = true
            resetElectionTimer()

    response.term = currentTerm
    send(request.candidateId, response)

function isLogUpToDate(candidateIndex, candidateTerm):
    myLastIndex = log.length - 1
    myLastTerm = log[myLastIndex].term if log.length > 0 else 0

    # Candidate's log is up-to-date if:
    # - Last entry has higher term, OR
    # - Last entry has same term but longer/equal length
    if candidateTerm > myLastTerm:
        return true
    if candidateTerm == myLastTerm and candidateIndex >= myLastIndex:
        return true

    return false

# State transitions
function becomeFollower():
    state = FOLLOWER
    votedFor = null

function becomeCandidate():
    state = CANDIDATE

function becomeLeader():
    state = LEADER

function initializeLeaderState():
    for each server in cluster:
        nextIndex[server] = log.length
        matchIndex[server] = 0

# Helper functions
function randomRange(min, max):
    return min + random() * (max - min)

function resetElectionTimer():
    electionTimeout = randomRange(150, 300)
    lastHeartbeat = currentTime()

function electionTimedOut():
    return currentTime() - lastHeartbeat > electionTimeout

function heartbeatIntervalElapsed():
    return currentTime() - lastHeartbeat > heartbeatInterval
```

**Key implementation details:**

1. **Randomized timeouts**: Prevents split votes (150-300ms typical)
2. **Term comparison**: Always update to higher term, reject lower
3. **Log comparison**: Vote only if candidate's log ≥ yours
4. **Majority requirement**: >50% votes needed (not ≥50%)
5. **State transitions**: Any higher term → become follower
6. **Heartbeats**: Leader sends empty AppendEntries to maintain authority

---

## Summary Table

| Concept | Key Points | Practical Impact |
|---------|------------|------------------|
| **FLP impossibility** | Consensus impossible in async system with failures | Accept that progress may stall; use timeouts and partial synchrony |
| **Paxos** | Theoretical foundation; prepare/accept phases | Correct but complex; use in libraries, not from scratch |
| **Raft** | Understandable alternative; leader + log replication | Easier to implement; used in etcd, Consul, CockroachDB |
| **Zab** | Total order broadcast; leader + epochs | High throughput; ZooKeeper's foundation |
| **Leader election** | Ensures single authority | Prevents split-brain; requires majority |
| **Log replication** | Consistent state across nodes | Fault tolerance; deterministic state machines |
| **Quorum writes** | Majority must acknowledge | Tolerates minority failures; costs latency |
| **Trade-offs** | Consistency vs availability vs latency | Choose consensus when correctness > availability |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
