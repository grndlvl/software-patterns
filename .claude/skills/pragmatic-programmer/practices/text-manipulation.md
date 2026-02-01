# Text Manipulation

> "Pragmatic Programmers manipulate text the same way woodworkers shape wood."
> — David Thomas & Andrew Hunt

## Why Text Manipulation Skills Matter

Text manipulation is a foundational skill that amplifies a programmer's productivity across nearly every task. Whether you're parsing logs, transforming data formats, generating code, or cleaning input, the ability to efficiently process text streams separates pragmatic programmers from those who manually edit data line by line.

**Core Benefits:**

- **Automation Over Repetition** – One well-crafted regex or script can replace hours of manual editing
- **Universal Applicability** – Text processing applies to configuration files, logs, data formats, code generation, and more
- **Composability** – Text tools can be chained together to solve complex problems
- **Speed** – Mastery of text manipulation lets you solve problems in seconds that might take others hours

## Regular Expressions Basics

Regular expressions (regex) are pattern-matching languages for text. They're essential for searching, validating, and transforming strings.

### Core Regex Components

| Pattern | Meaning | Example Match |
|---------|---------|---------------|
| `.` | Any single character | `a`, `7`, `@` |
| `*` | Zero or more of preceding | `ab*` matches `a`, `ab`, `abbb` |
| `+` | One or more of preceding | `ab+` matches `ab`, `abbb` (not `a`) |
| `?` | Zero or one of preceding | `ab?` matches `a`, `ab` |
| `^` | Start of line | `^Error` matches lines starting with "Error" |
| `$` | End of line | `failed$` matches lines ending with "failed" |
| `[abc]` | Character class (any of a, b, c) | `[0-9]` matches any digit |
| `[^abc]` | Negated class (not a, b, or c) | `[^0-9]` matches any non-digit |
| `\d` | Digit (0-9) | Matches `3` in `abc3def` |
| `\w` | Word character (alphanumeric + _) | Matches `a`, `Z`, `5`, `_` |
| `\s` | Whitespace (space, tab, newline) | Matches spaces and tabs |
| `(group)` | Capture group | `(error\|warning)` captures error or warning |
| `\|` | Alternation (OR) | `cat\|dog` matches "cat" or "dog" |

### Common Patterns

```pseudocode
// Email validation (simplified)
pattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

// Extract IP addresses
pattern = /\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/

// Match ISO date format (YYYY-MM-DD)
pattern = /^\d{4}-\d{2}-\d{2}$/

// Find function calls in code (simplified)
pattern = /\w+\s*\([^)]*\)/
```

## Text Processing Patterns

### Pattern 1: Filter Lines by Criteria

```pseudocode
// Find all error lines in a log file
filter lines in logfile where line matches /ERROR/

// Exclude comment lines from a config file
filter lines in configfile where line does not match /^\s*#/
```

### Pattern 2: Extract Specific Fields

```pseudocode
// Extract usernames from Apache access logs
for each line in access_log:
    match = extract pattern /^(\S+)/ from line
    print match.group(1)

// Parse CSV columns
for each line in csv_file:
    fields = split line by ','
    print fields[2]  // third column
```

### Pattern 3: Transform Text In-Place

```pseudocode
// Replace all occurrences of "old_function" with "new_function"
for each line in source_code:
    line = replace /old_function/ with "new_function" in line
    print line

// Convert snake_case to camelCase
for each identifier in code:
    identifier = replace /_([a-z])/ with uppercase($1) in identifier
```

### Pattern 4: Aggregate and Summarize

```pseudocode
// Count HTTP status codes in logs
counts = {}
for each line in access_log:
    status = extract pattern /HTTP\/\d\.\d" (\d{3})/ from line
    counts[status] += 1

for status, count in counts:
    print status, count
```

## Stream Editing Concepts

Stream editors process text line by line without loading the entire file into memory.

### Key Principles

1. **Input → Transform → Output** – Read from input stream, apply transformations, write to output stream
2. **Composability** – Chain multiple stream editors together using pipes
3. **No Side Effects** – Original data remains unchanged unless explicitly redirected
4. **Efficiency** – Process gigabyte-sized files with constant memory usage

### Common Stream Editing Operations

| Operation | Description |
|-----------|-------------|
| **Search** | Filter lines matching pattern |
| **Substitute** | Replace text globally |
| **Column extraction** | Print specific field |
| **Line numbering** | Add line numbers |
| **Deduplication** | Remove duplicate lines |
| **Counting** | Count lines |

### Pipeline Composition

```pseudocode
// Count unique IP addresses hitting an endpoint
input = read access_log
filtered = filter lines matching "/api/users"
ips = extract first field from each line
sorted = sort ips
unique = deduplicate sorted
counted = count lines in unique
print counted
```

## Practical Applications

### 1. Log Parsing

```pseudocode
// Find top 10 most frequent error messages
errors = filter lines from logfile matching /ERROR/
messages = extract pattern /ERROR: (.+)$/ from errors
sorted = sort messages
counted = count unique occurrences in sorted
top10 = take first 10 from counted
print top10
```

### 2. Data Extraction

```pseudocode
// Extract email addresses from a text file
emails = extract all matches of /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/ from document
deduplicated = unique entries in emails
print each email in deduplicated
```

### 3. Code Generation

```pseudocode
// Generate database model from schema definition
for each table in schema:
    print "class " + capitalize(table.name) + " {"
    for each column in table.columns:
        print "    " + column.type + " " + column.name
    print "}"
```

### 4. Data Cleaning

```pseudocode
// Standardize phone numbers
for each line in contacts:
    phone = extract pattern /\d+/ from line
    digits = join all matches
    if length(digits) == 10:
        formatted = format as "(###) ###-####"
        print formatted
```

## Summary Table

| Concept | Description | When to Use |
|---------|-------------|-------------|
| **Regular Expressions** | Pattern language for matching/extracting text | Validation, search, extraction |
| **Filtering** | Select lines matching criteria | Log analysis, data cleanup |
| **Extraction** | Pull specific fields from structured text | Parsing logs, CSVs, configs |
| **Transformation** | Replace or modify text according to rules | Refactoring, normalization |
| **Aggregation** | Count, sum, or summarize data | Statistics, reporting |
| **Code Generation** | Create boilerplate from templates | Reduce manual coding |
| **Stream Editing** | Process text line-by-line without full load | Large files, pipelines |
| **Pipeline Composition** | Chain simple tools for complex tasks | Unix philosophy, modularity |

## Key Takeaways

1. **Master Regex** – Regular expressions are the Swiss Army knife of text manipulation
2. **Think in Pipelines** – Compose small, single-purpose transformations
3. **Automate Early** – If you're doing it twice, script it
4. **Test on Real Data** – Validate patterns against actual inputs
5. **Know Your Tools** – Familiarity with text processing tools pays dividends daily

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
