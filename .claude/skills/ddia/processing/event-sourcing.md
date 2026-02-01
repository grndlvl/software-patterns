# Event Sourcing and CQRS

## Definition

**Event Sourcing** stores state changes as a sequence of immutable events rather than current state. The event log is the source of truth.

| Traditional | Event Sourcing |
|-------------|---------------|
| Store current state | Store all events |
| UPDATE modifies data | Append-only |
| History lost | History is primary data |

## Deriving Current State

```pseudocode
function getCurrentState(eventLog):
    state = initialState()
    for event in eventLog:
        state = applyEvent(state, event)
    return state

function applyEvent(state, event):
    switch event.type:
        case "AccountCreated":
            return {id: event.accountId, balance: 0}
        case "MoneyDeposited":
            state.balance += event.amount
            return state
        case "MoneyWithdrawn":
            state.balance -= event.amount
            return state
```

## CQRS (Command Query Responsibility Segregation)

Separate write model (commands) from read model (queries):

```pseudocode
// WRITE SIDE
function handleCommand(command):
    validate(command)
    events = commandToEvents(command)
    eventStore.append(events)

// READ SIDE
function handleQuery(query):
    return readModel.query(query)

// READ MODEL UPDATER (async)
eventStore.subscribe(event => readModel.update(event))
```

## Event Store Design

```pseudocode
class EventStore:
    function append(streamId, event):
        // Optimistic concurrency
        expectedVersion = getCurrentVersion(streamId)
        if streamId.version != expectedVersion:
            throw ConcurrencyException()

        event.version = expectedVersion + 1
        event.timestamp = now()
        database.insert(streamId, event)
        publishToSubscribers(event)

    function getEvents(streamId, fromVersion = 0):
        return database.query(
            "SELECT * FROM events WHERE stream_id = ?
             AND version >= ? ORDER BY version",
            streamId, fromVersion
        )
```

## Snapshots

Avoid replaying millions of events:

```pseudocode
function getCurrentState(streamId):
    snapshot = getLatestSnapshot(streamId)

    if snapshot:
        state = snapshot.state
        fromVersion = snapshot.version + 1
    else:
        state = initialState()
        fromVersion = 0

    events = getEvents(streamId, fromVersion)
    for event in events:
        state = applyEvent(state, event)

    return state

// Create snapshot every N events
function maybeSnapshot(streamId):
    if getCurrentVersion(streamId) % 100 == 0:
        createSnapshot(streamId)
```

## Bank Account Example

```pseudocode
// Events
class AccountCreated: accountId, owner, timestamp
class MoneyDeposited: accountId, amount, timestamp
class MoneyWithdrawn: accountId, amount, timestamp

// Command Handler
function deposit(accountId, amount):
    account = getCurrentState(accountId)
    if account.isClosed:
        throw "Account closed"
    if amount <= 0:
        throw "Invalid amount"

    eventStore.append(accountId, MoneyDeposited(accountId, amount, now()))

// Read Model
function handleEvent(event):
    switch event.type:
        case MoneyDeposited:
            db.execute("UPDATE accounts SET balance = balance + ? WHERE id = ?",
                       event.amount, event.accountId)
            db.insert("transactions", {type: "deposit", ...event})
```

## Benefits and Drawbacks

| Benefits | Drawbacks |
|----------|-----------|
| Complete audit trail | Complexity |
| Temporal queries (state at any time) | Eventual consistency |
| Replay for debugging | Schema evolution challenges |
| Multiple read models | Query complexity |
| Natural for event-driven | Learning curve |

## When to Use

### Good Fit
- Audit requirements (financial, medical, legal)
- Temporal queries needed
- Event-driven domains (orders, workflows)
- Need to replay events

### Poor Fit
- Simple CRUD
- No history requirements
- Strong consistency required
- Team unfamiliar with pattern

## Summary

| Aspect | Traditional | Event Sourcing |
|--------|-------------|---------------|
| What is stored | Current state | All events |
| Operations | CRUD | Append, read, project |
| Audit trail | Separate | Built-in |
| Temporal queries | Difficult | Natural |
| Complexity | Lower | Higher |
| Best for | Simple CRUD | Complex domains, audit |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
