# Designing Data-Intensive Applications Skill

Reference for distributed systems and data architecture concepts from Martin Kleppmann's "Designing Data-Intensive Applications."

## Activation Triggers

Use this skill when discussing:
- Database selection and data modeling
- Replication and high availability
- Partitioning/sharding strategies
- Distributed transactions
- Consistency models and guarantees
- Stream vs batch processing
- Event sourcing and CQRS

## Quick Reference

### Data Models

| Model | Best For | Trade-offs |
|-------|----------|------------|
| Relational | Complex queries, joins, ACID | Schema rigidity, scaling writes |
| Document | Hierarchical data, flexibility | Poor joins, denormalization |
| Graph | Highly connected data | Specialized queries, complexity |
| Wide-Column | Time series, analytics | Limited query patterns |

### Storage Engines

| Engine | Optimized For | Examples |
|--------|---------------|----------|
| B-Tree | Read-heavy, random access | PostgreSQL, MySQL |
| LSM-Tree | Write-heavy, sequential | Cassandra, RocksDB, LevelDB |
| Column Store | Analytics, aggregations | ClickHouse, Parquet |

### Replication Strategies

| Strategy | Consistency | Availability | Use Case |
|----------|-------------|--------------|----------|
| Single Leader | Strong | Medium | Traditional RDBMS |
| Multi-Leader | Eventual | High | Multi-datacenter |
| Leaderless | Eventual | Highest | High availability |

### Partitioning Strategies

| Strategy | Description | Pros | Cons |
|----------|-------------|------|------|
| Range | Partition by key ranges | Efficient range queries | Hot spots |
| Hash | Partition by hash of key | Even distribution | No range queries |
| Composite | Combine range + hash | Balanced | Complexity |

### Consistency Models

| Model | Guarantee | Performance |
|-------|-----------|-------------|
| Linearizable | Strongest (appears sequential) | Slowest |
| Sequential | Operations ordered per client | Medium |
| Causal | Cause-effect preserved | Good |
| Eventual | Will converge eventually | Fastest |

### Transaction Isolation Levels

| Level | Dirty Read | Non-Repeatable | Phantom |
|-------|------------|----------------|---------|
| Read Uncommitted | ✗ | ✗ | ✗ |
| Read Committed | ✓ | ✗ | ✗ |
| Repeatable Read | ✓ | ✓ | ✗ |
| Serializable | ✓ | ✓ | ✓ |

### CAP Theorem

> "In the presence of a network partition, choose Consistency OR Availability."

| Choice | Behavior | Examples |
|--------|----------|----------|
| CP | Reject requests if can't guarantee consistency | ZooKeeper, HBase |
| AP | Accept requests, allow inconsistency | Cassandra, DynamoDB |

### Batch vs Stream Processing

| Aspect | Batch | Stream |
|--------|-------|--------|
| Latency | High (hours/days) | Low (seconds/minutes) |
| Data | Bounded, complete | Unbounded, continuous |
| Processing | MapReduce, Spark | Kafka, Flink, Storm |
| Use Case | Analytics, ETL | Real-time alerts, dashboards |

## Directory Structure

```
ddia/
├── SKILL.md
├── data-models/
│   ├── relational.md
│   ├── document.md
│   └── graph.md
├── storage/
│   ├── b-trees.md
│   ├── lsm-trees.md
│   └── column-storage.md
├── replication/
│   ├── leader-follower.md
│   ├── multi-leader.md
│   └── leaderless.md
├── partitioning/
│   ├── strategies.md
│   └── rebalancing.md
├── transactions/
│   ├── acid.md
│   ├── isolation-levels.md
│   └── distributed-transactions.md
├── consistency/
│   ├── models.md
│   └── linearizability.md
├── consensus/
│   └── algorithms.md
└── processing/
    ├── batch.md
    ├── stream.md
    └── event-sourcing.md
```

## Usage Examples

### Choosing a Database

```
Question: "Should I use PostgreSQL or MongoDB?"

Consider:
- Data relationships → See data-models/
- Query patterns → See storage/
- Scale requirements → See partitioning/
- Consistency needs → See consistency/
```

### Designing for Scale

```
Question: "How do I handle millions of users?"

Consider:
- Read scaling → See replication/leader-follower.md
- Write scaling → See partitioning/strategies.md
- Geographic distribution → See replication/multi-leader.md
```

### Handling Failures

```
Question: "What happens when a node fails?"

Consider:
- Data durability → See replication/
- Consistency trade-offs → See consistency/models.md
- Recovery → See consensus/algorithms.md
```

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
