# Distributed Transactions

## Definition and Challenges

A **distributed transaction** coordinates changes across multiple independent databases or services, ensuring atomicity even when participants are on different nodes.

### Key Challenges:
- **Partial failures**: Some nodes may succeed while others fail
- **Network partitions**: Nodes may be unreachable during commit
- **Coordinator failures**: The coordinating node may crash mid-transaction
- **Resource locking**: Holding locks while waiting for remote nodes
- **Latency**: Network round-trips add significant overhead

Distributed transactions sacrifice availability and performance for consistency across services.

---

## Two-Phase Commit (2PC)

The most common protocol for distributed transactions. Ensures all participants commit or all abort.

### Phase 1: Prepare
1. **Coordinator** sends PREPARE request to all participants
2. Each **participant**:
   - Executes transaction locally
   - Writes to local transaction log
   - Acquires necessary locks
   - Responds YES (ready to commit) or NO (cannot commit)

### Phase 2: Commit
3. **Coordinator** collects all votes:
   - If ALL voted YES → send COMMIT to all participants
   - If ANY voted NO → send ABORT to all participants
4. Each **participant**:
   - Commits or aborts based on coordinator's decision
   - Releases locks
   - Acknowledges completion

```pseudocode
// Coordinator perspective
function twoPhaseCommit(transaction, participants):
    // Phase 1: Prepare
    votes = []
    for participant in participants:
        response = participant.prepare(transaction)
        votes.append(response)

    // Phase 2: Commit/Abort
    if all(vote == YES for vote in votes):
        decision = COMMIT
    else:
        decision = ABORT

    for participant in participants:
        participant.finalize(decision)

    return decision

// Participant perspective
function prepare(transaction):
    try:
        executeLocally(transaction)
        writeToTransactionLog(transaction, PREPARED)
        acquireLocks(transaction)
        return YES
    catch error:
        rollback(transaction)
        return NO

function finalize(decision):
    if decision == COMMIT:
        commitLocally()
    else:
        rollbackLocally()
    releaseLocks()
    writeToTransactionLog(decision)
```

---

## 2PC Failure Modes and Blocking

### The Blocking Problem

2PC can **block indefinitely** if the coordinator crashes after participants vote YES but before sending the final decision.

**Scenario:**
1. Participants vote YES and enter prepared state
2. Coordinator crashes before sending COMMIT/ABORT
3. Participants hold locks, waiting for decision
4. System is blocked until coordinator recovers

```pseudocode
// Participant stuck in prepared state
state = PREPARED
locksHeld = true
while not receivedDecision():
    wait()  // Indefinite blocking!
    // Cannot commit (don't know if others committed)
    // Cannot abort (don't know if others committed)
```

### Recovery from Coordinator Failure

**Coordinator log**: Must persist prepare and commit decisions to stable storage before sending messages.

```pseudocode
// Coordinator recovery
function recoverCoordinator():
    transactions = readTransactionLog()

    for txn in transactions:
        if txn.state == PREPARING:
            // Never sent prepare, safe to abort
            sendAbortToAll(txn)

        elif txn.state == PREPARED:
            // Sent prepare, must ask participants
            decisions = queryParticipants(txn)
            if all(d == YES for d in decisions):
                sendCommitToAll(txn)
            else:
                sendAbortToAll(txn)

        elif txn.state == COMMITTED:
            // Ensure all participants committed
            sendCommitToAll(txn)
```

### Participant Timeout Handling

Participants cannot unilaterally abort after voting YES (other participants may have committed).

```pseudocode
// Participant with timeout
function handlePrepareTimeout():
    if state == WAITING_FOR_PREPARE:
        // Safe to abort, haven't voted yet
        abort()

    elif state == PREPARED:
        // CANNOT abort unilaterally
        // Must wait for coordinator or contact other participants
        tryContactCoordinator()
        if coordinatorDead():
            tryContactOtherParticipants()
```

---

## Three-Phase Commit (3PC)

Attempts to avoid blocking by adding a pre-commit phase.

### Phases:
1. **CanCommit**: Coordinator asks if participants can commit
2. **PreCommit**: If all agree, coordinator sends preCommit (participants know decision is commit)
3. **DoCommit**: Coordinator sends final commit

**Advantage**: Participants know others have agreed before entering blocking state.

**Limitation**: Still fails under network partitions (doesn't solve FLP impossibility).

```pseudocode
function threePhaseCommit(transaction, participants):
    // Phase 1: CanCommit
    votes = []
    for participant in participants:
        vote = participant.canCommit(transaction)
        votes.append(vote)

    if not all(vote == YES for vote in votes):
        sendAbortToAll(participants)
        return ABORT

    // Phase 2: PreCommit
    for participant in participants:
        participant.preCommit(transaction)

    // Phase 3: DoCommit
    for participant in participants:
        participant.doCommit(transaction)

    return COMMIT
```

---

## Coordinator Failure Handling

### Transaction Log Requirements

The coordinator must write to its transaction log **before** sending each phase message:

```pseudocode
// Coordinator durability
function coordinatorPreparePhase(txn, participants):
    writeLog(txn.id, PREPARE_SENT)
    sync()  // Ensure durable before proceeding

    for participant in participants:
        participant.prepare(txn)

function coordinatorCommitPhase(txn, participants):
    writeLog(txn.id, COMMIT_DECISION)
    sync()  // Critical: must be durable before commit

    for participant in participants:
        participant.commit(txn)
```

### Participant Log Requirements

Participants must also log their state transitions:

```pseudocode
// Participant durability
function participantPrepare(txn):
    executeTransaction(txn)
    writeLog(txn.id, PREPARED)
    sync()

    sendVoteYes(coordinator)

function participantCommit(txn):
    commitLocalTransaction(txn)
    writeLog(txn.id, COMMITTED)
    sync()

    releaseLocks(txn)
```

---

## XA Transactions

**XA** is a standard API for distributed transactions (X/Open XA specification).

### Architecture:
- **Transaction Manager (TM)**: Coordinator
- **Resource Managers (RM)**: Databases, message queues
- **Application**: Initiates transaction

```pseudocode
// XA Transaction Example
function performXATransaction():
    tm = TransactionManager()
    txn = tm.begin()

    try:
        // Enlist resource managers
        db1 = ResourceManager("database1")
        db2 = ResourceManager("database2")
        queue = ResourceManager("messageQueue")

        txn.enlist(db1)
        txn.enlist(db2)
        txn.enlist(queue)

        // Perform operations
        db1.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
        db2.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
        queue.send("Transfer completed")

        // Two-phase commit handled by TM
        tm.commit(txn)

    catch error:
        tm.rollback(txn)
        throw error
```

### XA Limitations:
- **Performance**: Multiple round-trips, blocking locks
- **Deadlocks**: Cross-database deadlocks hard to detect
- **Incompatibility**: Not all systems support XA
- **Operational complexity**: Difficult to debug and monitor

---

## Saga Pattern as Alternative

**Sagas** provide distributed consistency without locking, using compensating transactions.

### Approach:
- Break distributed transaction into local transactions
- Execute sequentially
- If one fails, run compensating transactions to undo previous steps

```pseudocode
// Saga for booking trip
function bookTrip(customerId, flightId, hotelId):
    saga = Saga("book-trip")

    // Step 1: Book flight
    saga.addStep(
        action: lambda: flightService.reserve(flightId, customerId),
        compensation: lambda: flightService.cancel(flightId, customerId)
    )

    // Step 2: Book hotel
    saga.addStep(
        action: lambda: hotelService.reserve(hotelId, customerId),
        compensation: lambda: hotelService.cancel(hotelId, customerId)
    )

    // Step 3: Charge payment
    saga.addStep(
        action: lambda: paymentService.charge(customerId, totalAmount),
        compensation: lambda: paymentService.refund(customerId, totalAmount)
    )

    // Execute saga
    try:
        saga.execute()
        return SUCCESS
    catch error:
        saga.compensate()  // Run compensations in reverse order
        return FAILURE
```

### Saga Orchestration vs Choreography:

**Orchestration** (centralized):
```pseudocode
// Orchestrator controls entire flow
class TripBookingOrchestrator:
    function execute(customerId, flightId, hotelId):
        // Step 1
        flightReservation = flightService.reserve(flightId)
        if flightReservation.failed:
            return FAILURE

        // Step 2
        hotelReservation = hotelService.reserve(hotelId)
        if hotelReservation.failed:
            flightService.cancel(flightReservation.id)
            return FAILURE

        // Step 3
        payment = paymentService.charge(customerId)
        if payment.failed:
            hotelService.cancel(hotelReservation.id)
            flightService.cancel(flightReservation.id)
            return FAILURE

        return SUCCESS
```

**Choreography** (event-driven):
```pseudocode
// Each service reacts to events

// Flight Service
function onBookingRequested(event):
    reservation = reserve(event.flightId)
    publish(FlightReserved(reservation.id))

function onPaymentFailed(event):
    cancel(event.flightReservationId)

// Hotel Service
function onFlightReserved(event):
    reservation = reserve(event.hotelId)
    if reservation.success:
        publish(HotelReserved(reservation.id))
    else:
        publish(HotelReservationFailed(event.flightReservationId))

// Payment Service
function onHotelReserved(event):
    payment = charge(event.customerId)
    if payment.success:
        publish(PaymentSucceeded())
    else:
        publish(PaymentFailed(event.flightReservationId, event.hotelReservationId))
```

---

## When to Use Distributed Transactions

### Use 2PC/XA When:
- **Strong consistency** required (e.g., financial transactions)
- **Small number** of participants (2-3)
- **Short duration** transactions (milliseconds, not seconds)
- All systems support XA/2PC
- **Low scale** (not high-throughput)

### Use Sagas When:
- **Long-running** business processes
- **Many participants** or microservices
- **High availability** more important than immediate consistency
- Need to maintain **business invariants** rather than ACID guarantees
- Compensation logic is acceptable

### Avoid Distributed Transactions When:
- **High throughput** requirements
- **Cross-organization** boundaries (unreliable networks)
- Need for **high availability**
- Participants don't support 2PC

```pseudocode
// Decision matrix
function chooseApproach(requirements):
    if requirements.strongConsistency and
       requirements.participants <= 3 and
       requirements.durationMs < 100 and
       allSupportXA():
        return TWO_PHASE_COMMIT

    elif requirements.longRunning or
         requirements.participants > 3 or
         requirements.availabilityFirst:
        return SAGA_PATTERN

    elif requirements.crossOrganization:
        return EVENTUAL_CONSISTENCY

    else:
        return REDESIGN_BOUNDARIES  // Avoid distributed txn
```

---

## Summary Table

| Aspect | 2PC | 3PC | Saga |
|--------|-----|-----|------|
| **Consistency** | Strong (ACID) | Strong (ACID) | Eventual |
| **Availability** | Low (blocking) | Low (blocking) | High |
| **Latency** | High (2 RTT) | Higher (3 RTT) | Lower (async) |
| **Failure Mode** | Blocking | Blocking (network partition) | Non-blocking |
| **Compensations** | No | No | Yes |
| **Isolation** | Full (locks) | Full (locks) | None (dirty reads) |
| **Scalability** | Poor | Poor | Good |
| **Use Case** | Small, critical | Theoretical | Long-running processes |

### Key Takeaways:

1. **2PC blocks** if coordinator fails after prepare phase
2. **3PC reduces blocking** but still vulnerable to network partitions
3. **Sagas trade consistency** for availability and scalability
4. **Coordinator log** must be durable before sending commit
5. **Participants cannot abort** unilaterally after voting YES
6. **XA is standardized 2PC** with transaction manager coordination
7. **Choose based on requirements**: consistency vs availability vs scale

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
