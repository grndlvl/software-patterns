# Column-Oriented Storage

## Definition

**Column-oriented storage** stores all values from each column together, rather than all columns for each row together. Optimized for analytics workloads where queries access many rows but few columns.

## Row vs Column Storage

### Row-Oriented (Traditional)

```pseudocode
// Data stored by complete rows
ROW 1: [user_id=123, name="Alice", age=30, country="USA"]
ROW 2: [user_id=124, name="Bob", age=25, country="UK"]

// Query "average age" reads ALL columns for ALL rows
```

### Column-Oriented

```pseudocode
// Data stored by complete columns
COLUMN user_id:  [123, 124, 125, ...]
COLUMN name:     ["Alice", "Bob", "Carol", ...]
COLUMN age:      [30, 25, 35, ...]
COLUMN country:  ["USA", "UK", "USA", ...]

// Query "average age" reads ONLY the age column
```

## Compression Benefits

Column storage enables exceptional compression because similar data is stored together.

### Run-Length Encoding

```pseudocode
// Original: ["USA", "USA", "USA", "UK", "UK", "USA"]
// Encoded: [(value="USA", count=3), (value="UK", count=2), (value="USA", count=1)]
```

### Bitmap Encoding

```pseudocode
// Low cardinality column (country)
USA_bitmap:    [1, 0, 1, 0, 1, 0]
UK_bitmap:     [0, 1, 0, 0, 0, 1]
Canada_bitmap: [0, 0, 0, 1, 0, 0]

// COUNT users in USA = count 1s in USA_bitmap
```

### Dictionary Encoding

```pseudocode
// Original: ["Alice", "Bob", "Alice", "Carol", "Bob"]
// Dictionary: {0: "Alice", 1: "Bob", 2: "Carol"}
// Encoded: [0, 1, 0, 2, 1]
```

## Vectorized Processing

Modern CPUs operate on compressed column data using SIMD instructions:

```pseudocode
// Process 8 integers at once
CHUNK = [30, 30, 30, 30, 25, 25, 35, 35]
SIMD_SUM(chunk) = 240  // Single CPU instruction
```

## Column Families

Hybrid approach grouping related columns:

```pseudocode
TABLE users:
    COLUMN FAMILY identity: user_id, name, email
    COLUMN FAMILY activity: last_login, purchases, page_views

// Query user names reads only 'identity' family
```

## Sort Order

Column stores can maintain multiple sort orders:

```pseudocode
// Sort by date (time-range queries)
SORT_ORDER_1: date, product, sales

// Sort by product (product aggregations)
SORT_ORDER_2: product, date, sales
```

## When to Use Column Storage

### Good For

| Workload | Why |
|----------|-----|
| Analytics queries | Scan millions of rows, aggregate few columns |
| Data warehouses | OLAP workloads, batch writes |
| Time-series | Query by time range, aggregate metrics |
| Reporting | Calculate summaries across large datasets |

### Not Good For

| Workload | Why |
|----------|-----|
| OLTP | Need full row access, frequent updates |
| Small datasets | Overhead not worth it |
| Random row access | Optimizes for scanning |
| Write-heavy | Updates expensive |

## Summary Table

| Aspect | Row-Oriented | Column-Oriented |
|--------|--------------|-----------------|
| Storage layout | Columns for one row together | Rows for one column together |
| Optimal for | OLTP (transactions) | OLAP (analytics) |
| Compression | Limited | Excellent (10-100x) |
| Query types | Point lookups, updates | Scans, aggregations |
| Write performance | Fast | Slow |
| Examples | PostgreSQL, MySQL | Redshift, BigQuery, Parquet |

---

*Based on concepts from "Designing Data-Intensive Applications" by Martin Kleppmann.*
