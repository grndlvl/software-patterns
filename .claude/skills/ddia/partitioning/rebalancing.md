# Partition Rebalancing

## Overview

**Partition rebalancing** is the process of moving data from one node to another when the cluster configuration changes. It ensures that the load remains evenly distributed as nodes are added or removed.

## When Rebalancing is Needed

Rebalancing becomes necessary in several scenarios:

| Scenario | Reason |
|----------|--------|
| **Adding nodes** | New hardware added to increase capacity and share load |
| **Removing nodes** | Failed nodes or planned decommissioning |
| **Load changes** | Some partitions become hot and need splitting |
| **Hardware changes** | Nodes with different capacities require adjustment |

## Goals of Rebalancing

1. **Even distribution**: Load should be shared fairly after rebalancing
2. **Minimal movement**: Only necessary data should move to reduce I/O
3. **Availability**: Database should continue accepting reads/writes during rebalancing
4. **Automation**: Should happen automatically with minimal manual intervention

## Rebalancing Strategies

### 1. Fixed Number of Partitions

Create many more partitions than nodes at the start, then assign multiple partitions to each node.

**How it works:**
- Database created with fixed number of partitions (e.g., 1000)
- Each node gets multiple partitions (e.g., 10 nodes × 100 partitions each)
- When node added: steal partitions from existing nodes
- When node removed: distribute its partitions to remaining nodes

**Advantages:**
- Simple to implement
- Predictable partition assignment
- Easy to move entire partitions

**Disadvantages:**
- Must choose partition count at setup (hard to change later)
- Too few partitions: can't utilize many nodes
- Too many partitions: overhead per partition

**Example configuration:**

```pseudocode
// Initial setup with 1000 partitions, 10 nodes
partitions = createPartitions(1000)
nodes = [Node1, Node2, ..., Node10]

for partition in partitions:
    node = nodes[partition.id % nodes.length]
    assignPartition(partition, node)

// Adding an 11th node
newNode = Node11
nodes.append(newNode)

// Rebalance: move ~91 partitions to new node
partitionsToMove = selectPartitions(count=91, strategy="even_distribution")
for partition in partitionsToMove:
    movePartition(partition, from=currentOwner(partition), to=newNode)
```

### 2. Dynamic Partitioning

Partitions split when they grow too large and merge when they shrink too small.

**How it works:**
- Start with single partition or small number
- When partition exceeds threshold: split into two
- When partition shrinks below threshold: merge with neighbor
- Transfer partitions between nodes to balance load

**Advantages:**
- Adapts to data volume automatically
- Works well with both small and large datasets
- No need to estimate partition count upfront

**Disadvantages:**
- Empty database starts with single partition (cold start problem)
- More complex to implement
- Split/merge operations add overhead

**Example split/merge logic:**

```pseudocode
class DynamicPartition:
    threshold_max = 10_000_000  // 10 GB
    threshold_min = 1_000_000   // 1 GB

    function checkAndRebalance():
        if this.size > threshold_max:
            this.split()
        elif this.size < threshold_min:
            this.mergeWithNeighbor()

    function split():
        midpoint = this.range.start + (this.range.end - this.range.start) / 2

        partition1 = Partition(
            range=(this.range.start, midpoint),
            node=this.node
        )

        partition2 = Partition(
            range=(midpoint, this.range.end),
            node=selectLeastLoadedNode()
        )

        // Transfer data to new partition
        for key, value in this.data:
            if hashKey(key) < midpoint:
                partition1.insert(key, value)
            else:
                partition2.insert(key, value)
                transferToNode(partition2.node, key, value)

        replaceSelfWith([partition1, partition2])

    function mergeWithNeighbor():
        neighbor = findAdjacentPartition(this)

        if neighbor.size + this.size <= threshold_max:
            merged = Partition(
                range=(this.range.start, neighbor.range.end),
                node=this.node
            )

            // Transfer neighbor's data to this node
            for key, value in neighbor.data:
                transferToNode(this.node, key, value)
                merged.insert(key, value)

            for key, value in this.data:
                merged.insert(key, value)

            replaceSelfWith([merged])
```

**Pre-splitting optimization:**

```pseudocode
// Solve cold start problem by pre-creating partitions
function initializeDatabase(expectedDataSize):
    if expectedDataSize > 0:
        initialPartitions = max(10, expectedDataSize / threshold_max)
        return createPartitions(initialPartitions)
    else:
        return createPartitions(1)
```

### 3. Partitioning Proportional to Nodes

Make the number of partitions proportional to the number of nodes (fixed partitions per node).

**How it works:**
- Each node has fixed number of partitions (e.g., 256)
- When node added: creates new partitions by splitting existing ones
- When node removed: merge its partitions into remaining nodes
- Partition size grows proportionally to dataset size

**Advantages:**
- Balances partition count with cluster size
- Partition size stays relatively constant
- Works well for hash partitioning

**Disadvantages:**
- Partition splits during node addition (more complex)
- Less suitable for range partitioning
- Can be unfair if data distribution is skewed

**Example implementation:**

```pseudocode
class ProportionalPartitioning:
    partitions_per_node = 256

    function addNode(newNode):
        nodes.append(newNode)

        // Each existing node donates partitions
        partitionsNeeded = partitions_per_node
        partitionsPerExistingNode = partitionsNeeded / (nodes.length - 1)

        for existingNode in nodes[:-1]:  // exclude new node
            partitionsToSplit = selectRandom(
                existingNode.partitions,
                count=partitionsPerExistingNode
            )

            for partition in partitionsToSplit:
                splitAndTransfer(partition, existingNode, newNode)

    function splitAndTransfer(partition, fromNode, toNode):
        // Random split point for hash-based partitioning
        splitHash = randomBetween(partition.hashStart, partition.hashEnd)

        partition1 = Partition(
            hashRange=(partition.hashStart, splitHash),
            node=fromNode
        )

        partition2 = Partition(
            hashRange=(splitHash, partition.hashEnd),
            node=toNode
        )

        // Transfer data in partition2's range
        for key, value in partition.data:
            if hashKey(key) >= splitHash:
                transferToNode(toNode, key, value)
                partition2.insert(key, value)
            else:
                partition1.insert(key, value)

        fromNode.replace(partition, partition1)
        toNode.add(partition2)
```

## Automatic vs Manual Rebalancing

### Automatic Rebalancing

System decides when and how to rebalance without human intervention.

**Advantages:**
- Less operational burden
- Responds quickly to changes
- Consistent behavior

**Risks:**
- Can overload network during peak times
- May cascade failures if not careful
- Hard to predict timing

**Example automatic rebalancing:**

```pseudocode
class AutomaticRebalancer:
    check_interval = 60  // seconds
    imbalance_threshold = 0.15  // 15% difference

    function run():
        while true:
            sleep(check_interval)

            if shouldRebalance():
                performRebalance()

    function shouldRebalance():
        avgLoad = totalLoad() / nodes.length

        for node in nodes:
            deviation = abs(node.load - avgLoad) / avgLoad
            if deviation > imbalance_threshold:
                return true

        return false

    function performRebalance():
        plan = calculateRebalancePlan()

        // Throttle to avoid overload
        for move in plan.moves:
            if networkUtilization() < 0.8:  // 80% threshold
                executeMove(move)
                sleep(1)  // Rate limiting
            else:
                sleep(10)  // Back off when busy
```

### Manual Rebalancing

Administrator generates rebalancing plan and approves execution.

**Advantages:**
- Control over timing (schedule during off-peak)
- Human oversight prevents mistakes
- Can coordinate with maintenance windows

**Disadvantages:**
- Slower response to changes
- Requires operational expertise
- Can forget to rebalance

**Example manual workflow:**

```pseudocode
class ManualRebalancer:
    function generatePlan():
        currentState = snapshot()
        targetState = calculateOptimalDistribution()

        plan = Plan()
        for partition in allPartitions:
            currentNode = currentState.owner(partition)
            targetNode = targetState.owner(partition)

            if currentNode != targetNode:
                plan.addMove(partition, from=currentNode, to=targetNode)

        plan.estimateTime()
        plan.estimateNetworkImpact()
        return plan

    function reviewAndApprove(plan):
        // Human reviews the plan
        print("Rebalancing Plan:")
        print(f"  Total moves: {plan.moveCount}")
        print(f"  Estimated time: {plan.estimatedTime}")
        print(f"  Network impact: {plan.networkImpact}")
        print(f"  Partition moves: {plan.moves}")

        approval = getUserInput("Approve? (yes/no)")
        return approval == "yes"

    function execute(plan):
        if not reviewAndApprove(plan):
            print("Plan rejected")
            return

        for move in plan.moves:
            try:
                executeMove(move)
                logSuccess(move)
            except Exception as e:
                logFailure(move, e)
                if shouldAbort(e):
                    rollback(plan)
                    return

        print("Rebalancing complete")
```

## Minimizing Data Movement

Moving data is expensive. Good rebalancing algorithms minimize unnecessary transfers.

### Only Move What's Necessary

```pseudocode
function efficientRebalance(nodes, targetDistribution):
    // Calculate minimum moves needed
    moves = []

    for partition in allPartitions:
        currentOwner = partition.node
        targetOwner = targetDistribution[partition.id]

        if currentOwner != targetOwner:
            // Only add move if ownership changes
            moves.append(Move(partition, currentOwner, targetOwner))

    // Sort by partition size (move small ones first)
    moves.sortBy(partition.size)

    return moves
```

### Incremental Transfer

Transfer data gradually instead of all at once.

```pseudocode
function incrementalMove(partition, fromNode, toNode):
    // Phase 1: Copy data to new node while still serving from old
    copyPhase(partition, fromNode, toNode)

    // Phase 2: Sync changes that happened during copy
    syncDelta(partition, fromNode, toNode)

    // Phase 3: Brief pause for final sync
    pauseWrites(partition)
    finalSync(partition, fromNode, toNode)

    // Phase 4: Switch ownership
    updateRoutingTable(partition, newOwner=toNode)
    resumeWrites(partition)

    // Phase 5: Clean up old data
    deleteOldCopy(partition, fromNode)

function copyPhase(partition, fromNode, toNode):
    // Read from source, write to destination
    startTimestamp = now()

    for key, value in fromNode.scan(partition):
        toNode.write(key, value)

        // Yield periodically to avoid blocking
        if operationCount % 1000 == 0:
            sleep(0.01)

    return startTimestamp

function syncDelta(partition, fromNode, toNode, since):
    // Copy writes that happened during initial copy
    changes = fromNode.getChangesSince(partition, since)

    for change in changes:
        toNode.apply(change)
```

### Routing During Rebalancing

Handle requests correctly while data is moving.

```pseudocode
class RoutingDuringRebalance:
    function routeRequest(partition, operation):
        state = partition.migrationState

        if state == "STABLE":
            // Normal case: route to owner
            return partition.owner

        elif state == "COPYING":
            // Still copying: old node is authoritative
            if operation.isWrite():
                // Write to both old and new
                return [partition.oldOwner, partition.newOwner]
            else:
                // Read from old (complete data)
                return partition.oldOwner

        elif state == "SYNCING":
            // Final sync: new node is almost ready
            if operation.isWrite():
                // Write to both
                return [partition.oldOwner, partition.newOwner]
            else:
                // Can read from either
                return partition.newOwner

        elif state == "FINALIZING":
            // Brief pause before switch
            waitForMigrationComplete(partition)
            return partition.newOwner
```

## Rebalancing During Operation

Rebalancing must not disrupt normal database operations.

### Maintaining Availability

```pseudocode
class OnlineRebalancer:
    function rebalanceWithoutDowntime():
        // 1. Lock-free snapshot of current state
        snapshot = captureState()

        // 2. Calculate plan
        plan = generatePlan(snapshot)

        // 3. Execute moves without blocking
        for move in plan.moves:
            // Dual-write phase: writes go to both nodes
            enableDualWrite(move.partition)

            // Copy data in background
            backgroundCopy(move.partition, move.from, move.to)

            // Wait for copy to complete
            waitForCopyComplete(move.partition)

            // Atomic switch
            atomicSwitchOwner(move.partition, move.to)

            // Clean up old data
            asyncDelete(move.partition, move.from)

    function atomicSwitchOwner(partition, newOwner):
        // Use version number or compare-and-swap
        routingTable.compareAndSwap(
            partition.id,
            expected=partition.currentOwner,
            newValue=newOwner
        )

        // Broadcast update to all nodes
        broadcastUpdate(partition.id, newOwner)
```

### Handling Failures During Rebalancing

```pseudocode
class RebalanceWithFailureRecovery:
    function safeRebalance():
        checkpoint = saveCheckpoint()

        try:
            for move in rebalancePlan.moves:
                // Mark move as in-progress
                checkpoint.markInProgress(move)

                executeMove(move)

                // Mark move as complete
                checkpoint.markComplete(move)

        except NodeFailure as e:
            // Abort in-progress moves
            rollbackInProgressMoves(checkpoint)

            // Remove failed node from plan
            replan(excluding=e.failedNode)

            // Retry
            safeRebalance()

        finally:
            clearCheckpoint()

    function rollbackInProgressMoves(checkpoint):
        for move in checkpoint.inProgressMoves():
            // Revert ownership back to original
            routingTable.update(move.partition, owner=move.from)

            // Delete partial copy
            move.to.delete(move.partition)
```

## Problems and Trade-offs

### Common Challenges

| Problem | Description | Mitigation |
|---------|-------------|------------|
| **Network saturation** | Data transfer overwhelms network | Throttle transfer rate, schedule off-peak |
| **Cascading failures** | Node failure during rebalance triggers more failures | Pause rebalance on failures, reduce concurrency |
| **Hotspot migration** | Hot partition moves to overloaded node | Monitor load, avoid moving to busy nodes |
| **Thundering herd** | Many clients update routing simultaneously | Stagger routing updates, use version numbers |
| **Consistency issues** | Dual-write phase creates anomalies | Use transactions or careful ordering |

### Trade-off Matrix

| Approach | Speed | Safety | Complexity | Automation |
|----------|-------|--------|------------|------------|
| **Fixed partitions** | Fast | High | Low | Easy |
| **Dynamic partitions** | Medium | Medium | High | Hard |
| **Proportional** | Medium | Medium | Medium | Medium |
| **Automatic** | Fast | Medium | Medium | Full |
| **Manual** | Slow | High | Low | None |

## Rebalancing Decision Guide

```pseudocode
function chooseRebalancingStrategy(database):
    if database.workload == "predictable" and database.growth == "slow":
        return "fixed_partitions"

    elif database.workload == "variable" or database.size == "unknown":
        return "dynamic_partitions"

    elif database.isHashPartitioned() and database.nodes.change_frequently:
        return "proportional_partitions"

    else:
        // Default to fixed with generous partition count
        return "fixed_partitions", partitions=1024

function chooseAutomation(operationalCapacity):
    if operationalCapacity == "high" and database.critical:
        return "manual_with_automation_assist"

    elif operationalCapacity == "low":
        return "fully_automatic"

    else:
        return "automatic_with_approval"
```

## Summary Table

| Strategy | Best For | Partition Count | Adapts to Growth | Complexity |
|----------|----------|-----------------|------------------|------------|
| **Fixed** | Predictable workloads | Fixed at setup | No | Low |
| **Dynamic** | Variable data size | Grows/shrinks | Yes | High |
| **Proportional** | Hash partitioning | Fixed per node | No | Medium |

| Automation Level | Response Time | Safety | Ops Burden |
|------------------|---------------|--------|------------|
| **Fully Automatic** | Fast | Medium | Low |
| **Manual** | Slow | High | High |
| **Hybrid** | Medium | High | Medium |

## Key Takeaways

1. **Choose strategy based on workload characteristics**: Fixed for predictable, dynamic for variable
2. **Minimize data movement**: Only move necessary partitions
3. **Maintain availability**: Use dual-write and incremental transfer
4. **Monitor and throttle**: Prevent network saturation
5. **Plan for failures**: Checkpoint progress and support rollback
6. **Balance automation vs control**: Consider operational capacity
7. **Test rebalancing**: Ensure it works before production need

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
